# PLAN — Math Test Generator (MT)

Status: approved by Giorgos on 2026-11-07. Stage 5 (implementation) is open.
Prefix: `MT`. Method: tracer bullets, one worker at a time, strictly linear.

## 0. Paths and rules for every worker

These paths override the folder instructions for this repo.

| What | Path |
|---|---|
| This plan | `.agents/PLAN.md` |
| Knowledge graph | `.agents/graph.md` |
| Worker reports | `.agents/reports/<ID>-report.md` |
| Scope (Stage 2) | `docs/scope.md` |
| Architecture (Stage 3) | `docs/architecture.md` |
| ADRs | `docs/adr/` |

Worker rules:
1. Read this plan, then the `graph.md` nodes tagged with your milestone and its dependencies, then the architecture sections your milestone names.
2. Stay inside your milestone. Build backend first, then API and UI, with tests alongside.
3. Do not change a frozen interface (architecture section 4), the architecture, the scope, or your verification check. Stop and escalate instead.
4. If the work passes 1.5x the estimate, stop and escalate.
5. Never commit Τράπεζα Θεμάτων text (ADR-0003).
6. Write only your report file. Never tick a checkbox. Never edit `.agents/PLAN.md` or `.agents/graph.md`.
7. Your report has six fields: milestone ID, what was done, verification evidence (commands and their output), deviations, escalations and open questions, files changed.

A milestone starts only after the previous one is ticked by the Master.

ID note: `D<n>` in this plan and in `docs/` means the graph node `MT-D<n>`.

## 1. Current milestone

**Setup (Giorgos)**, then `MT-M1`.

## 2. Setup (Giorgos, before MT-M1)

- [ ] MT-S1 — Create the public GitHub repo. Commit `docs/` (scope, architecture, ADRs) and `.agents/` (PLAN, graph). Turn on branch protection for `main` (pull request plus green CI required).
  Verify: the repo URL is sent to the Master; `main` rejects a direct push.

## 3. Milestones

Estimates are in hours. At 5-10 hours per week, one milestone takes about 1-3 weeks.

### TB-1 — Walking skeleton

- [ ] **MT-M1 — Repo skeleton, domain and CI** (10-14 h)
  Depends on: MT-S1. Architecture: sections 3, 4.1, 8, 9, 12.
  Build: `uv` project and the repo layout; `ruff`, `mypy` (strict on `src/`), `pytest`, `pre-commit`; GitHub Actions on every pull request; Compose file with `postgres` and `seaweedfs` for local and CI; `domain` models for Item schema v1 and `Constraints` v1 (JSON Schema file); item-folder loader and validator; `mt check-licenses` with its pre-commit hook; the module dependency check; 3-4 synthetic fixtures per slot (clearly fake, license `own`).
  Verify: CI is green on a pull request. One CI test proves that a fixture with license `iep-terms` fails `check-licenses`. One test proves that `selector` cannot import an I/O module. `uv run pytest` passes locally with Compose up.

- [ ] **MT-M2 — Bank persistence, import and selector** (12-16 h)
  Depends on: MT-M1. Architecture: sections 3, 4.1, 4.2, 5, 6.2.
  Build: SQLAlchemy models and Alembic migrations for `items` and `item_status_events`; `bank` repository; `storage` (`ObjectStore` over SeaweedFS); `mt import-bank` from a local path or an S3 prefix, idempotent, items loaded as `published`; `selector` implementing the exam template (D1), deterministic for a given seed, raising `UnfillableSlotError`.
  Verify: importing the fixtures twice gives the same row count. Property tests (Hypothesis) prove three things: the same seed gives the same selection; every selected item is `published` and inside the chosen chapters; an empty slot raises the named error.

- [ ] **MT-M3 — LaTeX worker and builder** (12-16 h)
  Depends on: MT-M2. Architecture: sections 4.3, 7; ADR-0001, ADR-0005.
  Build: `latex-worker` image (minimal TeX Live, XeLaTeX, Libertinus) and its sandbox exactly as in section 7; `POST /compile` per the Renderer contract; `builder` with the exam and key templates (D1, D11); `render` client; one fixture with a figure (D10).
  Verify: the golden test on a Greek-and-math stress document passes (extracted text and page count). The security tests prove that `\write18` and reading `/etc/passwd` fail. A test proves the worker has no internet access. The figure fixture compiles.

- [ ] **MT-M4 — End-to-end generation slice** (8-12 h)
  Depends on: MT-M3 and content task MT-C1. Architecture: sections 5, 6.1.
  Build: `generations` table; `archive`; the generation service (select, build, render, store); `POST /api/exams`; minimal UI: chapter picker, then the exam PDF and the key PDF.
  Verify: a CI integration test runs the full flow on the fixtures and checks the PDF text. Giorgos imports the private seed bank and generates one real exam locally; the report records the command and a screenshot.
  **Closes TB-1.**

### TB-2 — Persistent service, deployed

- [ ] **MT-M5 — Single-user auth, archive and observability** (10-14 h)
  Depends on: MT-M4. Architecture: sections 4.4, 10.
  Build: `AuthProvider` with `SingleUserAuth` (configured credentials, session cookie); every route requires login; archive page with past generations; JSON logs with request id; `/healthz`, `/metrics`; admin usage page (D12).
  Verify: tests prove that an anonymous request is rejected on every route except `/healthz`. The archive lists a generation after it is created. Log lines are valid JSON with a request id.

- [ ] **MT-G1 — Deployment rights gate (Master and Giorgos)**
  Depends on: MT-M5. Decide how the bank data may be used on a deployed server (ADR-0003). Publish the decision as a node.

- [ ] **MT-M6 — Hetzner deployment, CD and backups** (12-18 h)
  Depends on: MT-G1. Architecture: sections 2, 9; ADR-0004.
  Build: Terraform (`hcloud`) for the VM, firewall and DNS; decide where the Terraform state lives (escalate the choice); production Compose with `caddy`; image build and push to GHCR; deploy workflow through a protected environment; nightly backup of Postgres and buckets to an off-site target; `docs/runbook.md` with a restore procedure.
  Verify: a merge to `main` deploys without manual steps. The HTTPS site generates and archives an exam. The report shows one full restore into a fresh VM or local Compose. The monthly cost estimate is at or under €25.
  **Closes TB-2.**

### TB-3 — Instruction parser (component A) and evals in CI (component E)

- [ ] **MT-M7 — LLM client, cost cap and eval harness** (10-14 h)
  Depends on: MT-M6. Architecture: sections 4.4, 5, 11; ADR-0002.
  Build: `LLMClient` with per-call logging to `llm_calls` and the monthly hard cap; a stub for tests; the eval harness (MLflow file store, JSON baselines in `evals/baselines/`, CI eval job with a regression gate); a rule-based baseline parser (keywords to `Constraints`).
  Verify: a test proves that a call over the cap is refused. The CI eval job runs the baseline on the dev split and fails when a metric drops below the committed baseline.

- [ ] **MT-M8 — Instruction parser and UI integration** (12-16 h)
  Depends on: MT-M7 and content task MT-C2. Architecture: sections 6.1, 11.
  Build: the LLM parser that returns schema-valid `Constraints`; the fallback to chapters only, with the error shown; the free-text field in the UI; model choice against the eval set (escalate the choice with numbers).
  Verify: the report shows parse accuracy on the 40-instruction held-out set for the LLM parser and for the baseline, the cost per call, and the latency. The CI gate passes. A malformed LLM response falls back without a server error (test).
  **Closes TB-3. MVP complete.**

### TB-4 — Keycloak

- [ ] **MT-M9 — Keycloak locally, roles and token validation** (10-16 h)
  Depends on: MT-M8. Architecture: section 4.4; ADR-0004.
  Build: Keycloak in Compose with its own database and role; realm, clients and roles as code (committed realm export, no secrets); `KeycloakAuth` (OIDC login, token validation); `admin` and `teacher` roles on routes.
  Verify: tests prove that a teacher cannot reach admin routes and that an expired or forged token is rejected. Swapping `SingleUserAuth` for `KeycloakAuth` changes no table.

- [ ] **MT-G2 — Colleague access gate (Master and Giorgos)**
  Depends on: MT-M9. Confirm the rights position for 2-3 colleagues (D12).

- [ ] **MT-M10 — Keycloak in production** (8-12 h)
  Depends on: MT-G2.
  Build: Keycloak on the VM behind `caddy`; Keycloak's database included in the backups; memory measured under load.
  Verify: a second user signs in as a teacher in production and cannot reach admin routes. The report shows peak VM memory. If it is above 85% of RAM, escalate.
  **Closes TB-4.**

### TB-5 — Admin form for typed items

- [ ] **MT-M11 — Draft, preview, verify, publish** (12-16 h)
  Depends on: MT-M10. Architecture: sections 5, 6.3.
  Build: admin forms for (a) Θέμα Α and Σωστό-Λάθος items and (b) bank items, with images; preview through the worker; status changes per D9; `corrections` table.
  Verify: Giorgos adds one real item through the UI and it appears in a generated exam. Tests cover every allowed and every forbidden status change.
  **Closes TB-5.**

### TB-6 — Auto-tagger (component D)

- [ ] **MT-M12 — Tagger model and evaluation** (10-16 h)
  Depends on: MT-M11 and content task MT-C3.
  Build: embeddings plus a classifier for sections and difficulty; a majority-class baseline; F1 and a confusion matrix logged to MLflow; the CI regression gate.
  Verify: the report shows F1 per class for the model and the baseline on the held-out split, and a short error analysis.

- [ ] **MT-M13 — Tag suggestions in the admin form** (6-10 h)
  Depends on: MT-M12.
  Build: suggested tags in the admin form; the admin's accepted or changed tags are stored as labels.
  Verify: a test proves that a suggestion never publishes an item without admin action.
  **Closes TB-6.**

### TB-7 — Ingestion agent (component B)

- [ ] **MT-M14 — OCR comparison** (12-16 h)
  Depends on: MT-M13 and content task MT-C4.
  Build: compare 2-3 OCR approaches (chosen with the Master at the start) on the 30-page set; metric: edit distance against the corrected LaTeX, plus compile rate.
  Verify: the report has a results table per approach and a recommendation. The Master records the choice as a decision node.

- [ ] **MT-M15 — Agent pipeline with traces** (14-20 h)
  Depends on: MT-M14.
  Build: PDF, OCR, LaTeX, compile, fix with bounded retries, all as a traced pipeline; a no-retry baseline; output lands as a `draft`.
  Verify: the report shows success rate per step and compile rate, with and without retries, on the held-out pages.

- [ ] **MT-M16 — Duplicates, auto-tag and the correction loop** (10-14 h)
  Depends on: MT-M15.
  Build: duplicate flagging; auto-tagging of drafts with the MT-M12 model; admin corrections turned into versioned dataset files; CI re-evaluation on the new version.
  Verify: one corrected item produces a new dataset version, and CI re-runs the eval on it. The report shows the admin correction time on 5 items.
  **Closes TB-7.**

### TB-8 — Verified solutions (component C)

- [ ] **MT-M17 — SymPy verifier** (10-14 h)
  Depends on: MT-M16 and content task MT-C5.
  Build: a final-answer checker with SymPy for the supported item types; a hand-checked set of correct and wrong answers.
  Verify: the report shows the pass rate and the false-accept rate on the hand-checked set.

- [ ] **MT-M18 — LLM solutions in the teacher key** (10-14 h)
  Depends on: MT-M17.
  Build: LLM-drafted solutions; only SymPy-verified solutions enter the key; the others show "no verified solution" (D11).
  Verify: a test proves that an unverified solution never appears in the key. The report shows the verification pass rate on the bank.
  **Closes TB-8.**

### TB-9 — Polish and publish (Stages 7 and 8)

- [ ] **MT-M19 — Docs, write-up, usage numbers** (10-14 h)
  Depends on: MT-M18.
  Build: README (purpose, setup, usage, results, why Hetzner); the technical write-up with results and ablations; usage numbers from D12; the timed baseline (3 exams by hand, 3 with the tool).
  Verify: the Stage 8 quality checklist passes. The Master drafts the CV bullet and the retro.
  **Closes TB-9.**

## 4. Content tasks (Giorgos, not workers)

| ID | Task | Needed before | Estimate |
|---|---|---|---|
| MT-C1 | Real seed: 4 proofs, 20 Σ-Λ, 6 "Θέμα 2", 4 "Θέμα 4" in item folders; the bank items stay private | MT-M4 | 6-8 h |
| MT-C2 | 100 instructions (60 own, 40 from colleagues), labeled, split 60/40 | MT-M8 | 4-5 h |
| MT-C3 | Difficulty labels for about 80 bank items | MT-M12 | 2-3 h |
| MT-C4 | 30 PDF pages with corrected LaTeX | MT-M14 | 4-6 h |
| MT-C5 | Hand-checked solution set | MT-M17 | 2-3 h |

Do each task in a week when its milestone is not running, so the weekly budget holds.

## 5. Totals

- Worker milestones (19): about 198-282 hours.
- Content tasks: about 18-25 hours.
- Release 1 in total: about 216-307 hours, so about 5-14 months at 5-10 hours per week.
- MVP (MT-M1 to MT-M8 plus MT-C1 and MT-C2): about 96-133 hours, so about 2-6 months.

## 6. Status notes

- 2026-11-07: Stages 2 and 3 confirmed. Stage 4 draft written.
- 2026-11-07: PLAN approved. `graph.md` created. Stage 5 open. Current: MT-S1 (Giorgos).
