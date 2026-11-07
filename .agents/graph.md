# graph.md — Math Test Generator (MT)

Knowledge graph shared between the Master and the workers. Only the Master writes this file.
Created: 2026-11-07, at the Stage 5 gate.

## How to read this file (workers)

1. Find your milestone node in section 1. Note its `depends_on`.
2. Read every node whose `tags` include your milestone ID, the ID of a milestone in your `depends_on`, or `all`.
3. Ignore the other nodes.
4. A node with `status: frozen` cannot change without an escalation.

Status values:
- milestone: `planned` | `active` | `done`
- decision: `settled` | `superseded`
- artifact: `frozen` | `accepted` | `planned`
- risk: `open` | `mitigated` | `closed`

Node fields: `type`, `status`, `tags`, `depends_on`, `produced_by`, `verified_by`, `notes`.

---

## 1. Milestones

### MT-S1 — Repo setup
- type: milestone
- status: active
- tags: [MT-S1]
- depends_on: none
- produced_by: Giorgos
- verified_by: Master
- notes: Public repo, `docs/` and `.agents/` committed, branch protection on `main`.

### MT-M1 — Repo skeleton, domain and CI
- type: milestone
- status: planned
- tags: [MT-M1]
- depends_on: MT-S1
- produced_by: worker
- verified_by: Master
- notes: TB-1. Estimate 10-14 h.

### MT-M2 — Bank persistence, import and selector
- type: milestone
- status: planned
- tags: [MT-M2]
- depends_on: MT-M1
- produced_by: worker
- verified_by: Master
- notes: TB-1. Estimate 12-16 h.

### MT-M3 — LaTeX worker and builder
- type: milestone
- status: planned
- tags: [MT-M3]
- depends_on: MT-M2
- produced_by: worker
- verified_by: Master
- notes: TB-1. Estimate 12-16 h.

### MT-C1 — Real seed content
- type: milestone
- status: planned
- tags: [MT-C1, MT-M4]
- depends_on: MT-M1
- produced_by: Giorgos
- verified_by: Master
- notes: 4 proofs, 20 Σ-Λ, 6 "Θέμα 2", 4 "Θέμα 4" as item folders. Bank items stay private. Estimate 6-8 h.

### MT-M4 — End-to-end generation slice
- type: milestone
- status: planned
- tags: [MT-M4]
- depends_on: MT-M3, MT-C1
- produced_by: worker
- verified_by: Master
- notes: Closes TB-1. Estimate 8-12 h.

### MT-M5 — Single-user auth, archive and observability
- type: milestone
- status: planned
- tags: [MT-M5]
- depends_on: MT-M4
- produced_by: worker
- verified_by: Master
- notes: TB-2. Estimate 10-14 h.

### MT-G1 — Deployment rights gate
- type: milestone
- status: planned
- tags: [MT-G1, MT-M6]
- depends_on: MT-M5
- produced_by: Master
- verified_by: Giorgos
- notes: Decide how bank data may be used on a deployed server. Output: a decision node.

### MT-M6 — Hetzner deployment, CD and backups
- type: milestone
- status: planned
- tags: [MT-M6]
- depends_on: MT-G1
- produced_by: worker
- verified_by: Master
- notes: Closes TB-2. Estimate 12-18 h. Terraform state location is escalated, not decided by the worker.

### MT-M7 — LLM client, cost cap and eval harness
- type: milestone
- status: planned
- tags: [MT-M7]
- depends_on: MT-M6
- produced_by: worker
- verified_by: Master
- notes: TB-3. Estimate 10-14 h.

### MT-C2 — Instruction eval set
- type: milestone
- status: planned
- tags: [MT-C2, MT-M8]
- depends_on: MT-M7
- produced_by: Giorgos
- verified_by: Master
- notes: 100 instructions (60 own, 40 from colleagues), about 70 Greek and 30 English, labeled, split 60 dev / 40 held-out. Estimate 4-5 h.

### MT-M8 — Instruction parser and UI integration
- type: milestone
- status: planned
- tags: [MT-M8]
- depends_on: MT-M7, MT-C2
- produced_by: worker
- verified_by: Master
- notes: Closes TB-3 and the MVP. Estimate 12-16 h. Model choice is escalated with numbers.

### MT-M9 — Keycloak locally, roles and token validation
- type: milestone
- status: planned
- tags: [MT-M9]
- depends_on: MT-M8
- produced_by: worker
- verified_by: Master
- notes: TB-4. Estimate 10-16 h.

### MT-G2 — Colleague access gate
- type: milestone
- status: planned
- tags: [MT-G2, MT-M10]
- depends_on: MT-M9
- produced_by: Master
- verified_by: Giorgos
- notes: Confirm the rights position before 2-3 colleagues get accounts.

### MT-M10 — Keycloak in production
- type: milestone
- status: planned
- tags: [MT-M10]
- depends_on: MT-G2
- produced_by: worker
- verified_by: Master
- notes: Closes TB-4. Estimate 8-12 h. Escalate if peak memory is above 85% of RAM.

### MT-M11 — Draft, preview, verify, publish
- type: milestone
- status: planned
- tags: [MT-M11]
- depends_on: MT-M10
- produced_by: worker
- verified_by: Master
- notes: Closes TB-5. Estimate 12-16 h.

### MT-C3 — Difficulty labels
- type: milestone
- status: planned
- tags: [MT-C3, MT-M12]
- depends_on: MT-M11
- produced_by: Giorgos
- verified_by: Master
- notes: Difficulty labels for about 80 bank items. Estimate 2-3 h.

### MT-M12 — Tagger model and evaluation
- type: milestone
- status: planned
- tags: [MT-M12]
- depends_on: MT-M11, MT-C3
- produced_by: worker
- verified_by: Master
- notes: TB-6. Estimate 10-16 h.

### MT-M13 — Tag suggestions in the admin form
- type: milestone
- status: planned
- tags: [MT-M13]
- depends_on: MT-M12
- produced_by: worker
- verified_by: Master
- notes: Closes TB-6. Estimate 6-10 h.

### MT-C4 — OCR evaluation pages
- type: milestone
- status: planned
- tags: [MT-C4, MT-M14]
- depends_on: MT-M13
- produced_by: Giorgos
- verified_by: Master
- notes: 30 PDF pages from licensed or own sources, with corrected LaTeX. Estimate 4-6 h.

### MT-M14 — OCR comparison
- type: milestone
- status: planned
- tags: [MT-M14]
- depends_on: MT-M13, MT-C4
- produced_by: worker
- verified_by: Master
- notes: TB-7. Estimate 12-16 h. The approaches are chosen with the Master at the start.

### MT-M15 — Agent pipeline with traces
- type: milestone
- status: planned
- tags: [MT-M15]
- depends_on: MT-M14
- produced_by: worker
- verified_by: Master
- notes: TB-7. Estimate 14-20 h.

### MT-M16 — Duplicates, auto-tag and the correction loop
- type: milestone
- status: planned
- tags: [MT-M16]
- depends_on: MT-M15
- produced_by: worker
- verified_by: Master
- notes: Closes TB-7. Estimate 10-14 h.

### MT-C5 — Hand-checked solution set
- type: milestone
- status: planned
- tags: [MT-C5, MT-M17]
- depends_on: MT-M16
- produced_by: Giorgos
- verified_by: Master
- notes: Correct and wrong final answers for the SymPy verifier evaluation. Estimate 2-3 h.

### MT-M17 — SymPy verifier
- type: milestone
- status: planned
- tags: [MT-M17]
- depends_on: MT-M16, MT-C5
- produced_by: worker
- verified_by: Master
- notes: TB-8. Estimate 10-14 h.

### MT-M18 — LLM solutions in the teacher key
- type: milestone
- status: planned
- tags: [MT-M18]
- depends_on: MT-M17
- produced_by: worker
- verified_by: Master
- notes: Closes TB-8. Estimate 10-14 h.

### MT-M19 — Docs, write-up, usage numbers
- type: milestone
- status: planned
- tags: [MT-M19]
- depends_on: MT-M18
- produced_by: worker
- verified_by: Master
- notes: Closes TB-9. Estimate 10-14 h.

---

## 2. Decisions

### MT-D0 — Practice-test framing
- type: decision
- status: settled
- tags: [MT-M4, MT-M19]
- depends_on: none
- produced_by: Master
- verified_by: Giorgos
- notes: The tool makes διαγωνίσματα, practice tests and ωριαία τετραμήνου. Not the official end-of-year exam, where Θέμα Β and Δ are drawn by lottery. The README says so.

### MT-D1 — TB-1 content and exam template
- type: decision
- status: settled
- tags: [MT-M1, MT-M2, MT-M3, MT-M4, MT-C1]
- depends_on: MT-D0
- produced_by: Master
- verified_by: Giorgos
- notes: Κεφ. 2 and 3. Θέμα Α = 5 objective questions (Σ-Λ) + 1 proof. Θέμα Β = one `theme_2` item. Θέμα Γ = one `theme_4` item with difficulty 1. Θέμα Δ = one `theme_4` item. Difficulty 1-3 is hand-set.

### MT-D2 — LaTeX engine, fonts and sandbox
- type: decision
- status: settled
- tags: [MT-M3]
- depends_on: none
- produced_by: Master
- verified_by: Giorgos
- notes: See ADR-0001 (MT-A4).

### MT-D3 — Hosted LLM behind an interface, hard cost cap
- type: decision
- status: settled
- tags: [MT-M7, MT-M8, MT-M15, MT-M18]
- depends_on: none
- produced_by: Master
- verified_by: Giorgos
- notes: Development cap $10 per month, enforced in `LLMClient`. Open model only as an eval baseline. See ADR-0002 (MT-A5).

### MT-D4 — No Kubernetes
- type: decision
- status: settled
- tags: [MT-M6]
- depends_on: none
- produced_by: Master
- verified_by: Giorgos
- notes: Non-goal.

### MT-D5 — Bank text never committed
- type: decision
- status: settled
- tags: [MT-M1, MT-M2, MT-M4, MT-G1, MT-M6]
- depends_on: none
- produced_by: Master
- verified_by: Giorgos
- notes: `mt import-bank` loads from local files or the private `bank-source` bucket. Rights for deployment are decided at MT-G1. See ADR-0003 (MT-A6).

### MT-D6 — Fixture licenses
- type: decision
- status: settled
- tags: [MT-M1]
- depends_on: MT-D5
- produced_by: Master
- verified_by: Giorgos
- notes: Committed items need `own`, `CC0-1.0`, `CC-BY-4.0` or `CC-BY-SA-4.0`. CI and a pre-commit hook fail on anything else.

### MT-D7 — Stack and hosting
- type: decision
- status: settled
- tags: [MT-M1, MT-M6, MT-M10]
- depends_on: MT-D4
- produced_by: Master
- verified_by: Giorgos
- notes: FastAPI, PostgreSQL, SQLAlchemy, Alembic, Jinja + HTMX. Hetzner VM via Terraform (`hcloud`), Docker Compose, SeaweedFS. Hosting cap €25 per month from TB-2. See ADR-0004 (MT-A7).

### MT-D8 — Local and CI services
- type: decision
- status: settled
- tags: [MT-M1, MT-M2, MT-M3]
- depends_on: MT-D7
- produced_by: Master
- verified_by: Giorgos
- notes: Compose with Postgres 17, SeaweedFS and the LaTeX worker in local and CI. No S3 or Postgres mocks. LLM stubbed in unit tests.

### MT-D9 — Item lifecycle
- type: decision
- status: settled
- tags: [MT-M2, MT-M11]
- depends_on: none
- produced_by: Master
- verified_by: Giorgos
- notes: `draft → verified → published → retired`. Selector reads `published` only. Every change logged in `item_status_events`. Import loads as `published` with Giorgos as verifier.

### MT-D10 — Figures
- type: decision
- status: settled
- tags: [MT-M3]
- depends_on: MT-D2
- produced_by: Master
- verified_by: Giorgos
- notes: PNG or PDF in the `assets` bucket, copied into the job bundle. No TikZ or pgfplots in TB-1. One fixture with a figure.

### MT-D11 — Answer keys
- type: decision
- status: settled
- tags: [MT-M3, MT-M4, MT-M18]
- depends_on: none
- produced_by: Master
- verified_by: Giorgos
- notes: Two PDFs per generation: exam and teacher key. Until TB-8 the key holds only own content. Bank items get a solution only after SymPy verification.

### MT-D12 — Users and usage numbers
- type: decision
- status: settled
- tags: [MT-M5, MT-M9, MT-G2, MT-M10, MT-M19]
- depends_on: none
- produced_by: Master
- verified_by: Giorgos
- notes: Giorgos only until TB-4, then 2-3 colleagues after MT-G2. Usage from `generations`. Time saved from 3 exams by hand against 3 with the tool. No surveys or tracking scripts.

### MT-D13 — Eval sets
- type: decision
- status: settled
- tags: [MT-M7, MT-M8, MT-M12, MT-M14, MT-M17, MT-C2, MT-C3, MT-C4, MT-C5]
- depends_on: none
- produced_by: Master
- verified_by: Giorgos
- notes: Each set is built inside its bullet. Parser 100 (60/40), tagger about 80 difficulty labels, OCR 30 pages. Every held-out set includes Greek text.

### MT-D14 — Items store paragraph codes
- type: decision
- status: settled
- tags: [MT-M1, MT-M2]
- depends_on: none
- produced_by: Master
- verified_by: Giorgos
- notes: Field `sections` holds codes such as `3.2`. The chapter is the prefix. Selection is by chapter, with optional section exclusions.

### MT-D15 — Item kind `objective`
- type: decision
- status: settled
- tags: [MT-M1]
- depends_on: none
- produced_by: Master
- verified_by: Giorgos
- notes: Kinds are `proof`, `true_false`, `objective`, `exercise`. TB-1 fills Θέμα Α's objective part with `true_false` items only.

### MT-D16 — License value `iep-terms`
- type: decision
- status: settled
- tags: [MT-M1, MT-M2]
- depends_on: MT-D5, MT-D6
- produced_by: Master
- verified_by: Giorgos
- notes: Used for imported bank items in the database. Never allowed under `fixtures/`.

### MT-D17 — Repo paths for agents and docs
- type: decision
- status: settled
- tags: [all]
- depends_on: none
- produced_by: Master
- verified_by: Giorgos
- notes: `.agents/` holds PLAN.md, graph.md and reports/. `docs/` holds scope.md, architecture.md, adr/ and runbook.md. PLAN.md section 0 overrides the folder paths.

### MT-D18 — Worker called over internal HTTP
- type: decision
- status: settled
- tags: [MT-M3, MT-M4]
- depends_on: MT-D2
- produced_by: Master
- verified_by: Giorgos
- notes: Synchronous HTTP on an internal network with no internet. No Docker socket. No job queue. See ADR-0005 (MT-A8).

### MT-D19 — Item folder format
- type: decision
- status: settled
- tags: [MT-M1, MT-M2, MT-C1]
- depends_on: MT-D14
- produced_by: Master
- verified_by: Giorgos
- notes: One folder per item: `item.yaml`, `body.tex`, optional `answer.tex`, asset files. Architecture section 8.

### MT-D20 — MLflow without a server
- type: decision
- status: settled
- tags: [MT-M7, MT-M12, MT-M15]
- depends_on: MT-D7
- produced_by: Master
- verified_by: Giorgos
- notes: File store uploaded as a CI artifact. JSON baselines in `evals/baselines/` drive the regression gate.

### MT-D21 — Minimum observability
- type: decision
- status: settled
- tags: [MT-M5, MT-M6]
- depends_on: MT-D7
- produced_by: Master
- verified_by: Giorgos
- notes: JSON logs with request id, `/healthz`, `/metrics`, admin usage page, free external uptime check. No Prometheus or Grafana server.

### MT-D22 — Pure core dependency rule
- type: decision
- status: settled
- tags: [MT-M1, MT-M2, MT-M3]
- depends_on: none
- produced_by: Master
- verified_by: Giorgos
- notes: `domain`, `selector` and `builder` import no I/O module. Enforced by a check in CI from MT-M1.

---

## 3. Artifacts

### MT-A1 — Scope (Stage 2)
- type: artifact
- status: accepted
- tags: [all]
- depends_on: none
- produced_by: Master
- verified_by: Giorgos
- notes: `docs/scope.md`. Decisions D0-D13, MVP, non-goals.

### MT-A2 — Architecture (Stage 3)
- type: artifact
- status: accepted
- tags: [all]
- depends_on: MT-A1
- produced_by: Master
- verified_by: Giorgos
- notes: `docs/architecture.md`.

### MT-A3 — PLAN
- type: artifact
- status: accepted
- tags: [all]
- depends_on: MT-A2
- produced_by: Master
- verified_by: Giorgos
- notes: `.agents/PLAN.md`. Approved 2026-11-07.

### MT-A4 — ADR-0001 LaTeX engine and sandbox
- type: artifact
- status: accepted
- tags: [MT-M3]
- depends_on: MT-D2
- produced_by: Master
- verified_by: Giorgos
- notes: `docs/adr/0001-latex-engine-and-sandbox.md`.

### MT-A5 — ADR-0002 LLM approach and cost cap
- type: artifact
- status: accepted
- tags: [MT-M7, MT-M8]
- depends_on: MT-D3
- produced_by: Master
- verified_by: Giorgos
- notes: `docs/adr/0002-llm-approach-and-cost-cap.md`.

### MT-A6 — ADR-0003 Bank data policy
- type: artifact
- status: accepted
- tags: [MT-M1, MT-M2, MT-G1, MT-G2]
- depends_on: MT-D5, MT-D6
- produced_by: Master
- verified_by: Giorgos
- notes: `docs/adr/0003-bank-data-policy.md`.

### MT-A7 — ADR-0004 Hosting and infrastructure
- type: artifact
- status: accepted
- tags: [MT-M6, MT-M9, MT-M10]
- depends_on: MT-D7
- produced_by: Master
- verified_by: Giorgos
- notes: `docs/adr/0004-hosting-and-infrastructure.md`.

### MT-A8 — ADR-0005 Modular monolith and LaTeX worker
- type: artifact
- status: accepted
- tags: [MT-M1, MT-M3]
- depends_on: MT-D18, MT-D22
- produced_by: Master
- verified_by: Giorgos
- notes: `docs/adr/0005-modular-monolith-and-latex-worker.md`.

### MT-I1 — Item schema v1
- type: artifact
- status: frozen
- tags: [MT-M1, MT-M2, MT-M3, MT-M11]
- depends_on: MT-D14, MT-D15, MT-D16
- produced_by: Master (design), MT-M1 (implementation)
- verified_by: Master
- notes: Architecture section 4.1.

### MT-I2 — Constraints schema v1
- type: artifact
- status: frozen
- tags: [MT-M1, MT-M2, MT-M7, MT-M8]
- depends_on: MT-D1, MT-D14
- produced_by: Master (design), MT-M1 (JSON Schema file)
- verified_by: Master
- notes: Architecture section 4.2.

### MT-I3 — Renderer contract
- type: artifact
- status: frozen
- tags: [MT-M3, MT-M4, MT-M11]
- depends_on: MT-D18
- produced_by: Master (design), MT-M3 (implementation)
- verified_by: Master
- notes: Architecture section 4.3.

### MT-I4 — AuthProvider
- type: artifact
- status: frozen
- tags: [MT-M5, MT-M9]
- depends_on: MT-D12
- produced_by: Master (design), MT-M5 (implementation)
- verified_by: Master
- notes: `current_user() -> User(subject, roles)`. Records store `subject` as text.

### MT-I5 — ObjectStore
- type: artifact
- status: frozen
- tags: [MT-M2, MT-M3, MT-M4]
- depends_on: MT-D7
- produced_by: Master (design), MT-M2 (implementation)
- verified_by: Master
- notes: `put`, `get`, `exists`, `presign_get`.

### MT-I6 — LLMClient
- type: artifact
- status: frozen
- tags: [MT-M7, MT-M8, MT-M15, MT-M18]
- depends_on: MT-D3
- produced_by: Master (design), MT-M7 (implementation)
- verified_by: Master
- notes: Returns result plus `tokens_in`, `tokens_out`, `cost_usd`, `latency_ms`. Refuses calls over the monthly cap.

---

## 4. Risks

### MT-R1 — Untrusted LaTeX escapes the sandbox
- type: risk
- status: open
- tags: [MT-M3, MT-M11, MT-M15]
- depends_on: MT-D2
- produced_by: Master
- verified_by: Master
- notes: Mitigation: architecture section 7 and its security tests.

### MT-R2 — Missing Greek or math glyphs
- type: risk
- status: open
- tags: [MT-M3]
- depends_on: MT-D2
- produced_by: Master
- verified_by: Master
- notes: Mitigation: golden test on a Greek-and-math stress document.

### MT-R3 — Bank text leaks into git
- type: risk
- status: open
- tags: [MT-M1, MT-M2, MT-M4]
- depends_on: MT-D5
- produced_by: Master
- verified_by: Master
- notes: Mitigation: `.gitignore`, license check in CI, pre-commit hook.

### MT-R4 — Selector cannot fill a slot
- type: risk
- status: open
- tags: [MT-M2]
- depends_on: MT-D1
- produced_by: Master
- verified_by: Master
- notes: Mitigation: `UnfillableSlotError` naming the slot; property tests over random banks.

### MT-R5 — Keycloak does not fit in 4 GB
- type: risk
- status: open
- tags: [MT-M9, MT-M10]
- depends_on: MT-D7
- produced_by: Master
- verified_by: Master
- notes: Mitigation: memory measured at MT-M10; `AuthProvider` keeps a swap small.

### MT-R6 — LLM cost passes the cap
- type: risk
- status: open
- tags: [MT-M7, MT-M8]
- depends_on: MT-D3
- produced_by: Master
- verified_by: Master
- notes: Mitigation: hard monthly limit in `LLMClient`.

### MT-R7 — The single VM loses data
- type: risk
- status: open
- tags: [MT-M6]
- depends_on: MT-D7
- produced_by: Master
- verified_by: Master
- notes: Mitigation: nightly backups off-site and a tested restore in the runbook.

### MT-R8 — Hetzner prices or availability change again
- type: risk
- status: open
- tags: [MT-M6]
- depends_on: MT-D7
- produced_by: Master
- verified_by: Master
- notes: Mitigation: Terraform makes a rebuild elsewhere cheap.

### MT-R9 — Scope creep against 5-10 hours per week
- type: risk
- status: open
- tags: [all]
- depends_on: none
- produced_by: Master
- verified_by: Master
- notes: Mitigation: one milestone at a time; escalate at 1.5x the estimate; backlog stays out.
