# Architecture — Math Test Generator (MT)

Status: Stage 3, agreed with Giorgos on 2026-11-07. Changes to sections 3 and 6 need a decision node in `graph.md`.
Scope source: `docs/scope.md` (Stage 2, decisions D0-D13).

## 1. Purpose

A web platform where a math teacher picks chapters (and, from TB-3, free-text instructions) and gets a practice exam for Α΄ Λυκείου Άλγεβρα as LaTeX and PDF, plus a teacher key. It is for διαγωνίσματα, practice tests and ωριαία τετραμήνου. It does not produce the official end-of-year exam, where Θέμα Β and Δ are drawn by lottery (D0).

## 2. Containers

| Container | Role | From |
|---|---|---|
| `app` | FastAPI. JSON API under `/api`, server-rendered UI (Jinja + HTMX), CLI. | TB-1 |
| `latex-worker` | The only place XeLaTeX runs. Small HTTP service in a sandbox (section 7). | TB-1 |
| `postgres` | PostgreSQL 17. App data. From TB-4 also a separate database and role for Keycloak. | TB-1 |
| `seaweedfs` | S3-compatible object store. Buckets: `assets`, `bank-source` (private), `generations`. | TB-1 |
| `caddy` | Reverse proxy, automatic TLS. | TB-2 |
| `keycloak` | OIDC login, roles `admin` and `teacher`. | TB-4 |

The same Compose definition runs locally, in CI, and on the Hetzner VM (D7, D8). Only `caddy` is exposed to the internet in production.

**Why a modular monolith:** one VM, one developer, 5-10 hours per week. Clear module boundaries give the design signal without microservice operations cost. The LaTeX worker is the only split, and it exists for security isolation, not for scale. See ADR-0005.

## 3. Modules in `app` (`src/mt/`)

| Module | Responsibility | Kind |
|---|---|---|
| `domain` | Models and enums: `Item`, `Constraints`, `ExamSpec`, `Selection`, `Theme` (A-D), `ItemKind`, `ItemCategory`, `ItemStatus`, `Difficulty`. | Pure |
| `selector` | `select(constraints, candidates, seed) -> Selection`. Deterministic for a given seed. Raises `UnfillableSlotError` naming the slot. | Pure |
| `builder` | Fills LaTeX templates. Returns `exam.tex`, `key.tex`, and the asset list. Escapes every inserted plain-text value. | Pure |
| `bank` | Repository for items, assets, status events (D9). | I/O |
| `archive` | Generation records. | I/O |
| `render` | `Renderer` client for the worker. | I/O |
| `storage` | `ObjectStore` over the S3 API. | I/O |
| `auth` | `AuthProvider`: `SingleUserAuth` (TB-2), `KeycloakAuth` (TB-4). | I/O |
| `parser` | TB-3. Instruction parser over `LLMClient`, validated against the `Constraints` schema. | I/O |
| `llm` | TB-3. `LLMClient`, cost logging, monthly hard cap (D3). | I/O |
| `cli` | `import-bank`, `check-licenses`, `export`. | I/O |
| `web` | Routes and templates. No business logic. | I/O |

**Dependency rule:** `domain`, `selector` and `builder` import no I/O module. `web` and `cli` are the only entry points and wire the rest together. A lint check (import-linter or an equivalent test) enforces this from TB-1.

## 4. Frozen interfaces (v1)

A worker escalates any change to these. The Master publishes the change as a decision node.

### 4.1 Item schema v1

| Field | Type | Notes |
|---|---|---|
| `id` | UUID | Internal. |
| `external_id` | str, nullable | Τράπεζα Θεμάτων code for bank items. |
| `kind` | `proof` \| `true_false` \| `objective` \| `exercise` | |
| `category` | `theme_2` \| `theme_4` \| `own` | Bank label, or `own` for Giorgos's items. |
| `sections` | list[str] | Textbook paragraph codes, for example `"3.2"`. The chapter is derived from the prefix. |
| `difficulty` | 1-3, nullable | Hand-set (D1). Required for `theme_4` items used as Θέμα Γ. |
| `body_latex` | str | LaTeX fragment, no preamble. |
| `answer_latex` | str, nullable | Own items only (D11). |
| `assets` | list[str] | Object keys in the `assets` bucket (D10). |
| `source` | str | Origin, for example a URL or "own". |
| `license` | str | SPDX-style id or `own` or `iep-terms`. CI rule in section 9. |
| `status` | `draft` \| `verified` \| `published` \| `retired` | D9. |

### 4.2 Constraints schema v1

The JSON the UI and the parser produce and the selector consumes. Published as a JSON Schema file in the repo.
- `chapters`: list of chapter codes (required, at least one).
- `exclude_sections`: optional list of paragraph codes.
- `difficulty`: optional per-theme min and max.
- `pin_items`: optional list of item ids per theme.
- `seed`: optional integer. The app sets one when it is missing and stores it.

### 4.3 Renderer contract

`POST /compile` on the worker. Request: a tar archive with `main.tex` and asset files. Response: the PDF and the compile log, or a structured error (`timeout`, `compile_error`, `limit_exceeded`) with the log.

### 4.4 Other interfaces
- `AuthProvider.current_user() -> User(subject: str, roles: set[str])`. Records store `subject` as text, not a foreign key, so the TB-4 swap needs no data migration.
- `ObjectStore`: `put`, `get`, `exists`, `presign_get`.
- `LLMClient`: one call method that returns the result plus `tokens_in`, `tokens_out`, `cost_usd`, `latency_ms`. It refuses calls when the monthly cap is reached.

## 5. Data model (PostgreSQL)

| Table | Holds | From |
|---|---|---|
| `items` | Item schema v1. | TB-1 |
| `item_status_events` | `item_id`, `from_status`, `to_status`, `subject`, `at`. | TB-1 |
| `generations` | `id`, `subject`, `created_at`, `request` (jsonb), `constraints` (jsonb), `selection` (jsonb), `seed`, object keys for the tex and PDF files, `render_ms`, `status`. | TB-1 |
| `llm_calls` | `id`, `generation_id` (nullable), `purpose`, `model`, `tokens_in`, `tokens_out`, `cost_usd`, `latency_ms`, `at`. The monthly cap is computed from this table. | TB-3 |
| `corrections` | Admin edits as labeled examples. | TB-5 |

Migrations are managed with Alembic from TB-1.

## 6. Data flows

### 6.1 Generation
1. The teacher selects chapters. From TB-3, also free-text instructions.
2. From TB-3: `parser` turns the instructions into `Constraints`. On failure the UI shows the error and falls back to chapters only.
3. `selector` picks from `published` items only, with a stored seed.
4. `builder` produces `exam.tex` and `key.tex`.
5. `render` sends each to the worker, with assets fetched from `ObjectStore`.
6. The PDFs and tex files go to the `generations` bucket. The record goes to `generations`.
7. The UI shows the exam PDF and offers the key.

### 6.2 Bank import (TB-1)
`mt import-bank <path-or-s3-prefix>` reads item folders (section 8), validates them against schema v1, uploads assets, and upserts items as `published` with Giorgos as verifier. Bank source files are never in git (D5).

### 6.3 Admin ingestion (TB-5, TB-7)
Form or PDF → `draft` → preview through the same worker → `verified` → `published`. Every admin edit is stored in `corrections`.

## 7. LaTeX sandbox (ADR-0001)

- XeLaTeX with `-no-shell-escape`. `openin_any` and `openout_any` set to `p` (paranoid).
- Non-root user. Read-only root filesystem. Work directory on tmpfs, new per job, deleted after.
- Limits on CPU, memory, process count and wall time.
- Internal Docker network with no internet. Only `app` can reach the worker.
- Security tests: `\write18` and reading `/etc/passwd` both fail.

## 8. Item folder format

One folder per item, used by `import-bank` and by `fixtures/items/`:

```
<item-slug>/
  item.yaml      # metadata: every schema v1 field except body and answer
  body.tex       # body_latex
  answer.tex     # optional
  <asset files>  # PNG or PDF
```

## 9. CI/CD

**Every pull request:** `ruff`, `mypy`, unit and property tests, integration tests against the Compose services, golden-PDF tests (extracted text and page count), the module dependency check, the license check, and a coverage gate.

**License check (D6):** every item under `fixtures/` must have a `license` of `own`, `CC0-1.0`, `CC-BY-4.0` or `CC-BY-SA-4.0`. Anything else fails CI. A pre-commit hook runs the same check.

**Merge to main (from TB-2):** build images, push to GHCR, deploy to the VM through a protected GitHub environment.

**From TB-3:** an eval job with a regression gate (section 11).

## 10. Observability (TB-2)

- Structured JSON logs with a request id on every line.
- `/healthz` and `/metrics`.
- An admin-only usage page built from `generations` (D12).
- A free external uptime check.
- No Prometheus or Grafana server on the VM.

## 11. Evaluation (TB-3 onward)

The CI eval job writes MLflow runs to a file store and uploads it as a CI artifact. JSON baselines in `evals/baselines/` are committed and drive the regression gate. No MLflow server on the VM. Each eval set is built inside its own bullet (D13).

## 12. Repository layout

```
src/mt/            app package (section 3)
worker/            LaTeX worker service and Dockerfile
templates/latex/   exam and key templates
fixtures/items/    open-licensed test items (section 8)
infra/compose/     Compose files for local, CI and production
infra/terraform/   Hetzner infrastructure (TB-2)
evals/             eval sets, baselines, runners (TB-3 onward)
tests/             unit, property, integration, golden
docs/              human-facing docs: architecture.md, scope.md, adr/, runbook.md (TB-2)
.agents/           agent coordination: PLAN.md, graph.md, reports/
```

Milestone prefix: `MT`.

## 13. Risks

| ID | Risk | Mitigation |
|---|---|---|
| R1 | Untrusted LaTeX escapes the sandbox. | Section 7 and its security tests. |
| R2 | Missing Greek or math glyphs. | TB-1 golden test on a Greek-and-math stress document. |
| R3 | Bank text leaks into git. | `.gitignore`, license check in CI, pre-commit hook. |
| R4 | Selector cannot fill a slot. | `UnfillableSlotError` naming the slot; property tests over random banks. |
| R5 | Keycloak does not fit in 4 GB. | Measure at TB-4. `AuthProvider` keeps the swap small. |
| R6 | LLM cost passes the cap. | Hard monthly limit in `LLMClient` (D3). |
| R7 | The single VM loses data. | Nightly `pg_dump` and bucket sync off-site; restore test in the runbook. |
| R8 | Hetzner prices or availability change again. | Terraform makes a rebuild elsewhere cheap. |
| R9 | Scope creep against 5-10 hours per week. | One bullet at a time; backlog stays out (scope.md). |

## 14. Deferred to their bullets (rule 4)

- Terraform state location and backup target: TB-2.
- Keycloak realm design and memory check: TB-4.
- OCR approaches to compare: TB-7.
- SymPy verification design: TB-8.

## 15. Decision map

| Decision | Where it lives |
|---|---|
| D0 practice-test framing | README, section 1 |
| D1 TB-1 content and template | `fixtures/`, `templates/latex/`, `selector` |
| D2 LaTeX engine and sandbox | `latex-worker`, section 7, ADR-0001 |
| D3 LLM and cost cap | `llm`, `llm_calls`, ADR-0002 |
| D4 no Kubernetes | section 2, ADR-0004 |
| D5 bank text never committed | `cli import-bank`, `bank-source` bucket, ADR-0003 |
| D6 fixture licenses | section 9, ADR-0003 |
| D7 stack and Hetzner | sections 2 and 3, ADR-0004 |
| D8 local and CI services | section 2, section 9 |
| D9 item lifecycle | `bank`, `item_status_events` |
| D10 figures | `assets` bucket, item `assets`, worker bundle |
| D11 answer keys | `builder` (`key.tex`), `answer_latex` |
| D12 users and usage | `auth`, `generations`, usage page |
| D13 eval sets | `evals/`, section 11 |
