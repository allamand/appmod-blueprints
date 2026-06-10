# CLAUDE.md — Fleet Management (gitops/fleet)

> Nested override for this directory. Read the root `CLAUDE.md` first for repo-wide context.

## What this is

**Hub-spoke Fleet Management** for multi-cluster ArgoCD on EKS. A *control plane* cluster hosts ArgoCD and registers *member* (workload) clusters; every member's addons, platform services, and team apps are driven from Git on the hub.

Editorial angle: this directory is the concrete backbone of the *"Fleet Management GitOps"* blog drafts in `allamand/peeks-veille`. Keep it consistent with that narrative.

## Layout

| Path | Role |
|---|---|
| `bootstrap/clusters.yaml` | ApplicationSet that generates one ArgoCD `Cluster` per registered member (pulls from secret store / External Secrets). |
| `bootstrap/addons.yaml` | ApplicationSet fanout for addons onto each member (Karpenter, Crossplane, Istio, observability…). |
| `bootstrap/fleet-secrets.yaml` | ExternalSecret wiring for cluster credentials. |
| `bootstrap/excluded/` | Templates kept out of the default bootstrap — enable case by case (app-specific ApplicationSets). |
| `charts/fleet-secret/` | Helm chart packaging the ExternalSecret used to inject member-cluster kubeconfigs into the hub. |
| `kro-values/tenants/control-plane/kro-clusters/` | **kro** `ResourceGraphDefinition` values for declarative cluster provisioning from the hub. |
| `members/` | Per-member overlays (empty skeleton — populated at workshop runtime). |

## Working rules

- **Hub is source of truth.** Never register clusters by calling `argocd cluster add` manually; add a secret and let `clusters.yaml` reconcile.
- **ApplicationSets only.** Don't hand-craft `Application` resources in this tree.
- **kro over raw YAML** when the surface is stable. kro `ResourceGraphDefinition` hides the glue (IAM, secret copy, cluster registration) behind one claim.
- **`excluded/` is intentional.** Entries here are NOT missing; they're held back to keep the default workshop path short. Move out of `excluded/` only with a corresponding workshop step update.
- **Secrets: always via ExternalSecrets / SSM / Secrets Manager.** No kubeconfig or token in Git, even base64.

## Verify before commit

- `task test-applicationsets` — validates every ApplicationSet in the repo, including this tree.
- For kro-values changes: `task test-kro-dryrun` then `task test-kro-integration`.
- For Helm chart changes under `charts/fleet-secret/`: `helm template charts/fleet-secret/ | kubeval -` (or equivalent policy check).

## When extending

Typical safe changes:
- Add an addon to `bootstrap/addons.yaml`' generator list.
- Add a new member profile under `members/<name>/` with overlays.
- Extend `kro-values/tenants/control-plane/kro-clusters/values.yaml` with a new cluster shape.

Riskier changes — discuss plan first:
- Restructuring generators in `clusters.yaml` (breaks every member at once).
- Modifying the `fleet-secret` chart contract (every ExternalSecret consumer breaks).
- Promoting anything out of `excluded/` (workshop impact).

## Anti-patterns

- Don't duplicate member-cluster identity in multiple places — single source is the secret backend, surfaced through `fleet-secrets.yaml`.
- Don't mix workshop-specific hacks with fleet primitives. Workshop hacks belong under `solutions/module1/`.
- Don't reference a member cluster by hardcoded name in `bootstrap/` — use the ApplicationSet generator fields.
