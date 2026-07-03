# AGENTS.md

This file provides guidance to Codex CLI and other AI coding agents when working with code in this repository.

## What This Repo Is

GitHub template repository (`is_template: true`) for Sky Haven Azure infrastructure projects. When creating a new `infra-*` repo, generate it from this template. The `infra/` scaffold and all pipelines come pre-wired.

## Template Bootstrap

When generating a repo from this template, update these placeholders:

- `infra/vars/globals.tfvars` — set `workload` to the project name (e.g. `sky-haven`)
- `infra/vars/dev.tfvars` / `prd.tfvars` — add environment-specific variable values

## Common Commands

```bash
# Initialise — substitute <env> with dev or prd
terraform -chdir=infra init \
  -backend-config="resource_group_name=rg-tfs-platform-<env>-uks-01" \
  -backend-config="storage_account_name=sttfsplatform<env>uks01" \
  -backend-config="container_name=<repo-name>" \
  -backend-config="key=terraform.tfstate" \
  -backend-config="subscription_id=<platform-subscription-id>"

# Plan / Apply / Destroy
terraform -chdir=infra plan    -var-file="vars/globals.tfvars" -var-file="vars/<env>.tfvars"
terraform -chdir=infra apply   -var-file="vars/globals.tfvars" -var-file="vars/<env>.tfvars"
terraform -chdir=infra destroy -var-file="vars/globals.tfvars" -var-file="vars/<env>.tfvars"
```

## Pipeline Behaviour

Pipelines consume shared reusable workflows and composite actions from `skyhaven-ltd/pipeline-engineering-github-actions`, SHA-pinned to a released tag.

**`lint.yml`** — runs on every PR to `main`. Calls the shared `reusable-lint.yml` (MegaLinter, `terraform` flavour); tune it via `.mega-linter.yml`. Terraform IaC scanning lives in `pr-validation.yml`, not here.

**`pr-validation.yml`** — runs on PRs to `main` (excluding `**/*.md` and `docs/**` changes). Calls the shared `reusable-terraform.yml`, which runs Terraform hygiene (`fmt`, backendless `init`, `validate`, TFLint) and zizmor workflow-security scanning, then for both `dev` and `prd` (GitHub Environments, Azure OIDC, remote state, `fail-fast: false`) runs a real Terraform plan and Checkov plan-aware deep analysis, and posts per-environment sticky Infracost comments after Checkov passes. Checkov findings fail the run unless centrally suppressed.

**`terraform.yml`** — deploy workflow. Triggers on push to `major/**`, `minor/**`, `patch/**` branches that touch `infra/**`, or via `workflow_dispatch` (env: dev/prd, action: plan/apply/destroy). Defaults to `dev` + `plan`. Authenticates to Azure via OIDC (no client secret). State plumbing — ensure container, backend init, break lease — is delegated to the shared composite actions.

**`tag.yml`** — auto-tags on PR merge via the shared `reusable-tag.yml`. Branch prefix drives semver bump: `major/**` → major, `minor/**` → minor, `patch/**` → patch. Other prefixes produce no tag.

## Terraform Structure

All `.tf` files live under `infra/`. Underscore-prefixed files hold specific block types only:

| File            | Block type     |
| --------------- | -------------- |
| `_terraform.tf` | `terraform {}` |
| `_providers.tf` | `provider`     |
| `_variables.tf` | `variable`     |
| `_locals.tf`    | `locals`       |

`infra/vars/globals.tfvars` holds shared values (`workload`). Environment files (`dev.tfvars`, `prd.tfvars`) hold values that differ per env. Both are always passed together to Terraform.

### Resource Naming

`local.resource_suffix` and `local.resource_suffix_flat` are the single source of truth for resource naming — all resource `name` arguments must reference one of these.

```
resource_suffix      = "{workload}-{environment}-{location_short}-{instance}"   # e.g. sky-haven-dev-uks-01
resource_suffix_flat = "{workload}{environment}{location_short}{instance}"       # e.g. skyhavendevuks01
```

All resources also inherit `local.tags` (`managed-by = "terraform"`).

## Azure State Backend

State backend is environment-specific:

| Env | Resource Group               | Storage Account         |
| --- | ---------------------------- | ----------------------- |
| dev | `rg-tfs-platform-dev-uks-01` | `sttfsplatformdevuks01` |
| prd | `rg-tfs-platform-prd-uks-01` | `sttfsplatformprduks01` |

Each repo gets its own container named after the repository. The `ensure-tfstate-container` action creates it on first run; `break-tfstate-lease` cleans up stuck leases.

## Required Secrets

GitHub environment secrets needed by the pipelines:

| Secret                           | Used for                                                |
| -------------------------------- | ------------------------------------------------------- |
| `AZURE_CLIENT_ID`                | OIDC workload identity federated credential             |
| `AZURE_SUBSCRIPTION_ID`          | Target subscription for ARM provider                    |
| `AZURE_TENANT_ID`                | OIDC tenant                                             |
| `AZURE_PLATFORM_SUBSCRIPTION_ID` | Platform subscription hosting the state storage account |
| `INFRACOST_API_KEY`              | Infracost PR cost-visibility comments                   |

## Validation Config

Validation tool configuration is centralised in `.github/validation/`:

| File           | Tool    |
| -------------- | ------- |
| `checkov.yaml` | Checkov |
| `tflint.hcl`   | TFLint  |
| `zizmor.yml`   | zizmor  |

## Checkov Skips

Skips in `.github/validation/checkov.yaml` are intentional suppressions for controls that conflict with Sky Haven's design decisions (e.g. Key Vault soft-delete managed externally, `workflow_dispatch` inputs by design). Do not remove them without understanding the rationale.
