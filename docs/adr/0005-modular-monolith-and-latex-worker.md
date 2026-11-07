# ADR-0005: Modular monolith with one isolated LaTeX worker

- Status: accepted (2026-11-07)

## Context
One developer works 5-10 hours per week on one VM. The design must show clear boundaries and testability without microservice operations cost. LaTeX compilation needs a security boundary.

## Decision
- One FastAPI application (`app`) with modules split into a pure core (`domain`, `selector`, `builder`) and I/O adapters (`bank`, `archive`, `render`, `storage`, `auth`, `llm`, `parser`, `cli`, `web`).
- The pure core imports no I/O module. A check in CI enforces this.
- The LaTeX worker is a separate container: a small HTTP service on an internal network with no internet, called synchronously by `app`.
- No job queue until a bullet needs one.

## Consequences
- Selector and builder tests are fast and need no services.
- A future split into services stays possible along the adapter interfaces.
- Long compiles block a request; acceptable at seconds per compile.

## Alternatives considered
- `app` runs `docker run` per compile: needs the Docker socket, which is root-equivalent.
- A job queue now: speculative work against rule 4 of the method.
- Microservices: operations cost without benefit at this size.
