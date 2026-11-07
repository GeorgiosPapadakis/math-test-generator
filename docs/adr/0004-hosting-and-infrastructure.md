# ADR-0004: Hetzner VM with Docker Compose, provisioned by Terraform

- Status: accepted (2026-11-07)
- Decisions: D4, D7, D8

## Context
The app needs Postgres, object storage, a LaTeX worker and, from TB-4, Keycloak. The hosting budget is €25 per month. Kubernetes is a non-goal. As of October 2026, Hetzner's cheapest cost-optimized plans could not be ordered; the nearest 4 GB plan costs about €20 per month. The MinIO community edition is archived.

## Decision
- One Hetzner Cloud VM with about 4 GB RAM, provisioned with Terraform (`hcloud` provider).
- Docker Compose runs `caddy`, `app`, `latex-worker`, `postgres`, `seaweedfs` and, from TB-4, `keycloak`.
- SeaweedFS provides the S3 API locally, in CI and in production. Hetzner Object Storage is used only when needed (about €6.49 per month base price).
- Keycloak uses its own database and role on the same Postgres server.
- GitHub Actions builds images, pushes them to GHCR, and deploys through a protected environment.
- Hosting cap: €25 per month from TB-2, separate from the $10 LLM development cap.
- Terraform state location and the off-site backup target are decided in TB-2.

## Consequences
- One VM is a single point of failure; nightly backups and a tested restore are required.
- The same Compose definition runs everywhere, which keeps local, CI and production close.
- The README must explain why Hetzner and not AWS: cost, and more hands-on operations work.

## Alternatives considered
- AWS EC2 or ECS: about the same design at higher cost for 4 GB.
- Serverless (for example Cloud Run): fits Keycloak and the sandboxed worker badly.
- Kubernetes: a non-goal.
