# infra-engineering-template

Template repository for Sky Haven Azure infrastructure projects. It provides a standard Terraform project scaffold and repository automation so new infrastructure repos start with consistent naming, validation, deployment, and versioning conventions.

## PR validation

Pull requests to `main` run `.github/workflows/pr-validation.yml`, which replaces Super-Linter with Terraform-focused validation:

- Terraform formatting, backendless init, validate, and TFLint.
- zizmor workflow security scanning.
- Remote Terraform plans for both `dev` and `prd` using Azure OIDC and environment-scoped secrets.
- Checkov plan-aware deep analysis against Terraform plan JSON and source Terraform.
- Infracost sticky PR comments for each environment after Checkov passes.

Docs-only and README-only PRs are ignored, forked PRs are skipped, and Terraform plan artifacts are not uploaded.
