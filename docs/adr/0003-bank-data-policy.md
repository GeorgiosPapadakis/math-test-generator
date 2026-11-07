# ADR-0003: Bank data stays out of git; fixtures are open-licensed

- Status: accepted (2026-11-07)
- Decisions: D5, D6

## Context
The repository is public. No open license for the Τράπεζα Θεμάτων Διαβαθμισμένης Δυσκολίας was found. The IEP website terms forbid copying, republishing and commercial use of site content. Text committed to a public repo cannot be fully removed later, because clones and forks keep the history.

## Decision
- The Τράπεζα Θεμάτων text and images are never committed.
- `mt import-bank` loads item folders from local disk or from the private `bank-source` bucket into the database.
- The repo may contain an item's id, sections, category and Giorgos's labels, never its text.
- CI and demos use fixtures: exercises Giorgos owns or that carry an open license.
- Every item has `source` and `license` fields. Allowed licenses for committed items: `own`, `CC0-1.0`, `CC-BY-4.0`, `CC-BY-SA-4.0`. CI and a pre-commit hook fail on anything else.
- The rights question for deployment and for sharing with colleagues is decided at the deployment gate, before TB-2 goes public and before TB-4 adds users.

## Consequences
- A self-hoster must obtain the bank files on their own and run the import.
- Tests never depend on the real bank.
- If IEP grants written permission later, a new decision node can relax this policy.

## Alternatives considered
- Commit the bank with attribution: no license supports this.
- Private repo: removes the portfolio value.
