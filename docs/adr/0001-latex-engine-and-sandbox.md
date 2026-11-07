# ADR-0001: LaTeX engine, fonts and compile sandbox

- Status: accepted (2026-11-07)
- Decisions: D2, D10

## Context
Exams mix Greek text and mathematics. LaTeX from the bank, from the admin, and later from an LLM is untrusted input. Compiling untrusted LaTeX can run shell commands or read files when the engine is not locked down.

## Decision
- Engine: XeLaTeX with `fontspec`, `polyglossia` (Greek) and `unicode-math`.
- Fonts: Libertinus Serif and Libertinus Math (OFL).
- Image: TeX Live with a minimal scheme plus an explicit package list.
- Sandbox: `-no-shell-escape`; `openin_any` and `openout_any` set to `p`; non-root user; read-only root filesystem; tmpfs work directory per job; CPU, memory, process and time limits; internal network with no internet; only `app` can reach it.
- The worker is a small HTTP service called synchronously by `app` (see ADR-0005).
- Figures are PNG or PDF files copied into the job bundle. No TikZ or pgfplots in TB-1.
- Tests: golden tests on extracted PDF text and page count; security tests proving that `\write18` and reading `/etc/passwd` fail.

## Consequences
- One engine and one font family to support.
- XeLaTeX is slower than pdfLaTeX. A compile still takes seconds, which is acceptable.
- Byte-level PDF comparison is not used, because PDFs embed timestamps and ids.

## Alternatives considered
- pdfLaTeX with `babel` greek: weaker Unicode input handling for Greek.
- LuaLaTeX: viable; slower start and no clear gain for this use.
- Compiling inside `app`: no isolation boundary.
