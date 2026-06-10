# CLAUDE.md — Platform Engineering on EKS (PeEKS)

> Read this first at the start of every session in this repo. Keep it short on purpose. Nested `CLAUDE.md` in subdirectories override/augment on demand.

## What this repo is

Upstream: **aws-samples/appmod-blueprints** — code backing the AWS workshop *Platform Engineering on Amazon EKS*: https://catalog.workshops.aws/platform-engineering-on-eks

Stack: EKS + ArgoCD (GitOps) + Backstage (IDP) + kro (resource orchestration) + Karpenter + Crossplane + OpenTofu/Terraform. Workshop tested in `us-west-2` and `eu-central-1`.

## Working with this codebase

Task runner is **Taskfile.yml** (not Makefile). Always prefer `task <target>` over ad-hoc shell.

Common targets:
- `task build-helm-dependencies` / `check-helm-dependencies` / `clean-helm-dependencies`
- `task test-applicationsets` — validates ArgoCD ApplicationSets
- `task test-kro-all` / `test-kro-complete` — full kro validation suite
- `task test-kro-unit` / `test-kro-integration` / `test-kro-dryrun` — granular kro checks
- `task backstage-validate`, `task test-backstage-kro*` — Backstage templates
- `task test-airflow-chart`, `task test-kubeflow-chart` — data workloads
- `task default` — quick CI-equivalent smoke

## Architecture map

| Dir | Purpose |
|---|---|
| `platform/infra/` | Terraform/OpenTofu for EKS cluster, VPC, addons |
| `platform/backstage/` | Backstage portal config + kro-backed templates |
| `platform/validation/` | Shared validation scripts (kro, helm) |
| `gitops/addons/` | Cluster addons via ArgoCD (Karpenter, Crossplane, Istio, etc.) |
| `gitops/apps/` | Team apps (ApplicationSets) |
| `gitops/fleet/` | **Fleet management** — multi-cluster hub/spoke ArgoCD |
| `gitops/platform/` | Platform-level GitOps (Backstage, observability) |
| `gitops/workloads/` | Stateful/ML workloads (Airflow, Kubeflow) |
| `applications/` | Sample apps (dotnet, golang, java, node, python, rust, next-js, mono-a2c) |
| `solutions/module1/` | Workshop module source |
| `backstage/` | Backstage source (forked customisations) |

## Pivot files to read when onboarding

- `README.md` — workshop entry, CloudFormation bootstrap (Cloud9 / us-west-2 or eu-central-1)
- `Taskfile.yml` — canonical command surface
- `gitops/fleet/` — Fleet Management reference (angle éditorial du draft *Fleet Management GitOps*)
- `platform/backstage/` + `gitops/platform/` — Backstage ↔ kro integration
- `.kiro/` — Kiro CLI spec files (separate IDE context, ignore unless touching Kiro)

## Conventions & constraints

- **GitOps is the source of truth.** Never `kubectl apply` against workshop clusters; commit to `gitops/**` and let ArgoCD reconcile.
- **Regions**: tested in `us-west-2` and `eu-central-1`. Don't introduce region-coupled code without parametrisation.
- **Workshop-first**: every change must keep `https://catalog.workshops.aws/peeks` scenarios working. Check `solutions/module1/` before refactoring platform code.
- **PRs**: upstream is `aws-samples/appmod-blueprints` — respect `CONTRIBUTING.md`, DCO sign-off, CLA.
- **Renovate** is active (`renovate.json`) — don't bump pinned versions manually.

## Before writing code

1. Start with **codebase Q&A**: explore relevant dir, read the local README/docs if any, check how it's referenced from `gitops/` or `Taskfile.yml`.
2. For changes that touch >30% of a file or cross directories, **brainstorm a plan and wait for approval** before editing.
3. Prefer small, tight commits that map to a single workshop step or a single ApplicationSet.
4. When in doubt about workshop impact, check `docs/` and `RELEASE_NOTES.md`.

## Feedback loops (always give yourself a way to verify)

- Kubernetes changes → `kubectl apply --dry-run=server -f …` or `task test-kro-dryrun`
- Helm changes → `task check-helm-dependencies` + `helm template` diff
- Terraform/OpenTofu → `tofu plan` in `platform/infra/` (never `apply` from an agent without explicit user confirmation)
- ArgoCD manifests → `task test-applicationsets`
- Backstage templates → `task backstage-validate`

## Anti-patterns (don't)

- Don't `terraform apply` / `tofu apply` / `kubectl delete` without explicit confirmation in the current message.
- Don't expand this file past ~100 lines — it eats context. Push depth into nested `CLAUDE.md` in the subdirectory you're working in.
- Don't mirror `AGENTS.md`/Kiro/Cursor rules here; reference them if needed, don't duplicate.
- Don't add a MEMORY.md or ERRORS.md at the repo root — those are agent-personal, not project-shared.
