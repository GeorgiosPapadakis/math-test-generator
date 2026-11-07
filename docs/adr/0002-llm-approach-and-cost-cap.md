# ADR-0002: Hosted LLM behind an interface, with a hard cost cap

- Status: accepted (2026-11-07)
- Decisions: D3, D13

## Context
The instruction parser (TB-3), the ingestion agent (TB-7) and verified solutions (TB-8) need an LLM. The project is non-profit, with a development budget of $10 per month. Every AI component must have a baseline and a measured evaluation.

## Decision
- Use a hosted LLM API behind an `LLMClient` interface.
- `LLMClient` records `model`, `tokens_in`, `tokens_out`, `cost_usd` and `latency_ms` for every call in the `llm_calls` table.
- `LLMClient` refuses calls when the month's total reaches the cap. Development cap: $10 per month.
- A small open model is used only as an evaluation baseline, not on the critical path.
- Unit tests use a stub. Real calls happen only in the eval job and in the running app.
- LLM output never reaches a published exam without schema validation, SymPy verification, or admin approval.

## Consequences
- The provider can change without touching `parser` or later components.
- The cap can block development in a heavy month. That is intended.
- The specific provider and model are chosen in TB-3, against the eval set.

## Alternatives considered
- Self-hosted open model: needs a GPU or a larger VM, beyond the hosting cap.
- No cap, alerts only: alerts do not stop a loop that keeps calling.
