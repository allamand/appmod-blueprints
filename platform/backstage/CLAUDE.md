# CLAUDE.md — Backstage Templates (platform/backstage)

> Nested override for this directory. Read the root `CLAUDE.md` first for repo-wide context.

## What this is

Backstage **software templates** (scaffolder) packaged for the PeEKS IDP. Each template is a ready-to-scaffold pattern the developer portal exposes, usually backed by **kro** `ResourceGraphDefinition`s, **Crossplane** compositions, **KubeVela** applications, or **ACK** controllers on the target cluster.

Two roots:
- `templates/` — core templates shipped with the workshop (deploy, pipeline, RDS, S3, Spark, Ray, EKS cluster, etc.).
- `customtemplates/` — extra templates maintained here for demo/extension (e.g. DynamoDB via kro).

`catalog-info.yaml` at each level registers the templates with Backstage.

## Layout

| Path | Role |
|---|---|
| `catalog-info.yaml` | Top-level catalog pointing at template dirs. |
| `templates/catalog-info.yaml` | Aggregator for all core templates. |
| `templates/app-deploy/` + `app-deploy-without-repo/` | Generic microservice deploy scaffolds (ArgoCD + namespace + RBAC). |
| `templates/cicd-pipeline/` | Argo Workflows / EventBridge pipeline template with `GITOPS_MIGRATION.md` and `WEBHOOK_INTEGRATION.md` notes. |
| `templates/create-dev-and-prod-env/` | Two-cluster (dev + prod) scaffold via kro. |
| `templates/eks-cluster-template/` | Single-cluster kro template. |
| `templates/rds-cluster/`, `s3-bucket/`, `s3-bucket-ack-kro/`, `s3-bucket-ack-kubevela/` | Data-plane claim templates (Crossplane / kro / KubeVela / ACK flavors). |
| `templates/ray-serve-{cpu,gpu,trainium}/` | ML inference scaffolds. |
| `templates/spark-job/` | Batch template. |
| `templates/old_legacy/` | **Deprecated** — do not extend, reference for history only. |
| `customtemplates/ddb-table/` | Example of a kro-backed Backstage template you can copy from. |

## Working rules

- **Template contract**: every template is a `template.yaml` + `skeleton/` tree. `skeleton/` contains what ends up in the user's new repo; `template.yaml` describes inputs + steps.
- **Catalog registration**: any new template MUST be referenced from `templates/catalog-info.yaml` (or `customtemplates/catalog-info.yaml`). Unregistered templates are invisible to Backstage.
- **kro > Crossplane > raw YAML** when multiple options exist. kro gives the cleanest claim-API surface for workshop attendees. Use Crossplane when you need an existing managed resource that kro doesn't yet wrap.
- **ACK vs kro vs KubeVela**: the `s3-bucket-*` family illustrates the three idioms side by side. Keep them in sync — fixing a bug in one means reviewing the other two for the same issue.
- **Don't scaffold AWS resources via Terraform inside a Backstage template.** If you need terraform, use the `tofu-controller` or the `cicd-pipeline` path, not the scaffolder directly.

## Verify before commit

- `task backstage-validate` — validates Backstage catalog + template YAML.
- `task test-backstage-kro` (+ `-frontend` / `-backend`) — integration for kro-backed templates.
- Lint the embedded manifests: `kubectl apply --dry-run=client -f templates/<name>/skeleton/manifests/` when a template writes K8s manifests.

## When extending

Adding a template:
1. Copy an existing one close to your use case (`customtemplates/ddb-table/` is a good kro baseline).
2. Adjust `template.yaml` inputs and `skeleton/` contents.
3. Register in the relevant `catalog-info.yaml`.
4. `task backstage-validate`, then `task test-backstage-kro*` if kro-backed.
5. Document the new template in `README.md` (keep the bullet list alphabetical by template name).

Modifying an existing template:
- Bump / note backward-incompatible changes — users already scaffolded from this template will not be auto-migrated.
- If the template is referenced from workshop docs (`solutions/module1/`, `docs/`), update the workshop step in the same PR.

## Anti-patterns

- Don't inline long shell scripts in `template.yaml` steps. Put them under `skeleton/scripts/` and invoke them.
- Don't hard-code cluster names, account IDs, or regions — use template parameters.
- Don't add new templates to `old_legacy/`. Create a new directory.
- Don't branch from `main` directly to edit `template.yaml` — always scaffold a test repo first to verify the rendered output.
