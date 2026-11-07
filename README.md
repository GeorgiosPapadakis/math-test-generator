# Math Test Generator

A web platform that builds practice math exams for Greek lyceum teachers, as LaTeX and PDF, from structured exercise banks. First scope: Α΄ Λυκείου, Άλγεβρα.

> Status: in development. See `.agents/PLAN.md` for the current milestone.

## What it is for

Practice tests, διαγωνίσματα and ωριαία τετραμήνου. It does **not** produce the official end-of-year exam: there, Θέμα Β and Θέμα Δ are drawn by lottery from the Τράπεζα Θεμάτων on exam day.

## Bank data

This repository contains no Τράπεζα Θεμάτων text. A self-hoster imports their own copy with the `import-bank` command. Tests use exercises with open licenses. See `docs/adr/0003-bank-data-policy.md`.

## Documentation

- Scope: `docs/scope.md`
- Architecture: `docs/architecture.md`
- Decisions: `docs/adr/`
- Plan and knowledge graph (agent coordination): `.agents/`

Setup, usage and results sections arrive with the milestones that produce them.

## License

To be decided before the first public release.
