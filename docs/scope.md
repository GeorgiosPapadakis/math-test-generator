# Scope / MVP (Stage 2) — Math Test Generator (MT)

Status: confirmed by Giorgos on 2026-11-07. Stage 1 score: 22/30 (feasibility 3).
Decision IDs `D<n>` here mean the graph nodes `MT-D<n>` in `.agents/graph.md`.

## Facts found
- ΓΕΛ Α΄/Β΄ Άλγεβρα: Θέμα 2 and Θέμα 4 are drawn by lottery from the Τράπεζα Θεμάτων Διαβαθμισμένης Δυσκολίας on exam day. Θέμα 1 and 3 are set by teachers. Θέμα 1 = 5 objective questions (10 points) + 1 proof (15 points).
- The tool therefore serves διαγωνίσματα, practice tests and ωριαία τετραμήνου, not the official end-of-year exam.
- The IEP website terms forbid copying, republishing and commercial use. No open license for the Τράπεζα Θεμάτων was found.
- Hetzner cost-optimized plans (CX/CAX) were not orderable in October 2026; the nearest 4 GB plan costs about €20 per month. The MinIO community edition is archived.

## Settled decisions
- D0: Product framing: practice tests and διαγωνίσματα, stated in the README.
- D1: TB-1 covers Κεφ. 2 and 3. Seed: 4 proofs, 20 Σ-Λ, 6 "Θέμα 2", 4 "Θέμα 4". Θέμα Α = 5 objective questions + 1 proof. Θέμα Γ = a "Θέμα 4" item with difficulty 1 (difficulty 1-3, hand-set).
- D2: XeLaTeX + polyglossia + unicode-math, Libertinus Serif and Math, minimal TeX Live, sandbox (no shell escape, no network, limits), golden-file tests on extracted text.
- D3: Hosted LLM behind `LLMClient` with per-call cost logging; open model only as an eval baseline. Development cap $10 per month, enforced in code.
- D4: No Kubernetes.
- D5: Bank text never committed. CLI `import-bank` loads from local files or S3. Rights decided at the deployment gate.
- D6: Fixtures from open-licensed or own exercises. `source` and `license` on every item; CI fails on a committed item without an allowed license.
- D7: FastAPI, PostgreSQL, SQLAlchemy, Alembic, Jinja + HTMX. Hetzner via Terraform (`hcloud`), one VM with Docker Compose, SeaweedFS for objects. Hosting cap €25 per month from TB-2.
- D8: Local and CI: Compose with Postgres 17, SeaweedFS and the LaTeX worker. No S3 or Postgres mocks. LLM stubbed in unit tests.
- D9: Lifecycle `draft → verified → published → retired`; the selector reads `published` only; changes are logged; import loads as `published`.
- D10: Figures as PNG or PDF in S3, copied into the sandbox; no TikZ in TB-1; one fixture with a figure.
- D11: Two PDFs per generation: exam and teacher key. The key holds own content only until TB-8; bank items get solutions only after SymPy verification.
- D12: Users: Giorgos at TB-2; 2-3 colleagues at TB-4, subject to the rights decision. Usage from generation records; time saved from a timed baseline (3 exams by hand, 3 with the tool).
- D13: Eval sets built inside their bullets: parser 100 (about 70 Greek / 30 English, 60/40 split), tagger difficulty labels for about 80 items, OCR 30 pages. About 10-14 hours of labeling.

## MVP and release 1
- MVP = TB-1 to TB-3: a deployed generator for Giorgos, with the instruction parser and its CI eval gate.
- Release 1 = TB-1 to TB-9.

## Non-goals
Payments; other grades or Γεωμετρία; Laya; βοήθημα content; verbatim textbook text; Kubernetes; committed Τράπεζα Θεμάτων text; official end-of-year exam generation; TikZ or generated plots; a learned ranker; a fine-tuned OCR model; a variant generator; surveys or tracking scripts.

## Assumptions
- The ΓΕΛ 2026-27 paragraph list for Άλγεβρα matches Giorgos's syllabus (the ΦΕΚ text was not yet checked).
- Libertinus fonts render all needed Greek and math symbols (checked in TB-1).
- A 4 GB Hetzner VM fits the app, Postgres and Keycloak (checked at TB-4).
- The OCR approaches to compare are chosen inside TB-7.
