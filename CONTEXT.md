# Infra Engineering Template

Language for the Sky Haven Terraform infrastructure template and its CI validation model.

## Language

**PR validation**:
The required checks that run when a pull request targets `main`; these checks decide whether an infrastructure change is safe enough to merge.
_Avoid_: CI linting, merge checks

**Docs-only change**:
A pull request change limited to documentation files that does not alter Terraform, CI workflow behaviour, validation configuration, or executable scripts.
_Avoid_: README change, markdown change

**Validation config directory**:
The central `.github/validation/` directory that stores configuration for PR validation tools such as Checkov, TFLint, and zizmor.
_Avoid_: Linter directory, Super-Linter config

**Split validation jobs**:
A PR validation shape where fast Terraform hygiene checks and GitHub Actions security checks run separately from Azure-authenticated plan-aware Checkov checks.
_Avoid_: Combined pipeline, single lint job

**Workflow security scanning**:
Explicit GitHub Actions security analysis with zizmor, kept after removing Super-Linter because this template relies on OIDC and privileged CI workflows.
_Avoid_: YAML linting, Actions linting

**SHA-pinned action**:
A third-party GitHub Action referenced by full commit SHA rather than a mutable tag to reduce CI supply-chain risk.
_Avoid_: Version-pinned action, tagged action

**Pinned CLI tool**:
A command-line validation tool installed at an explicit version rather than `latest` so PR validation remains deterministic.
_Avoid_: Latest tool, floating tool version

**Job-scoped OIDC permission**:
A least-privilege workflow permission model where only the Azure-authenticated validation job receives `id-token: write`.
_Avoid_: Workflow-wide OIDC, global OIDC permission

**Trusted PR**:
A same-repository pull request that is allowed to run the full PR validation workflow.
_Avoid_: Internal PR, safe PR

**Forked PR**:
A pull request from a repository fork that is excluded from PR validation for this template.
_Avoid_: External PR, contributor PR

**Fail-fast disabled matrix**:
An environment matrix setting where `dev` and `prd` validation continue independently so all environment failures are reported in one PR run.
_Avoid_: Parallel matrix, matrix strategy

**Ephemeral plan file**:
A Terraform plan or plan JSON generated inside a CI runner for scanning and cost analysis, then discarded instead of uploaded as a workflow artifact.
_Avoid_: Plan artifact, retained plan

**Merge-blocking finding**:
A validation result that fails PR validation and prevents a pull request from being merged into `main` until it is fixed or explicitly suppressed.
_Avoid_: Error, violation

**Environment matrix**:
The set of Terraform environments that PR validation evaluates independently before merge; for this template the matrix is `dev` and `prd`.
_Avoid_: Deployment matrix, build matrix

**Environment cost comment**:
A sticky Infracost pull-request summary for one Terraform environment, posted only after Checkov passes and updated on later runs instead of duplicated.
_Avoid_: Cost gate, combined cost report, one-off comment

**Environment secret scope**:
The GitHub Environment boundary, `dev` or `prd`, used to provide environment-specific Azure and Infracost secrets to PR validation.
_Avoid_: Repository secret scope, global secret scope

**State helper action**:
An existing local GitHub Action reused by PR validation to create the Terraform state container before planning or break a leftover state lease afterwards.
_Avoid_: Backend helper, storage helper

**Remote state backend**:
The Azure Storage-backed Terraform state location used by CI and operators for an environment-specific infrastructure workspace.
_Avoid_: Backend, state storage

**OIDC workload identity**:
The GitHub Actions authentication path that lets PR validation request Azure access without storing a client secret.
_Avoid_: SPN secret, service principal password

**Terraform hygiene checks**:
Fast pre-plan checks that verify Terraform formatting, syntax, provider-aware validation, and lint rules before any security or cost analysis runs.
_Avoid_: Generic linting, code quality sweep

**Configuration scanning**:
Static infrastructure-as-code security analysis performed directly against Terraform source files before a Terraform plan exists.
_Avoid_: File scanning, static scanning

**Plan-aware deep analysis**:
Infrastructure-as-code security analysis performed by Checkov using both Terraform plan JSON and source Terraform so evaluated resource changes can be enriched with source-level context.
_Avoid_: Dynamic scanning, graph scanning, plan-aware scanning

**Central suppression file**:
The repository-level Checkov configuration file that records accepted Checkov finding suppressions.
_Avoid_: Skip file, ignore list

**Finding suppression**:
An explicit exception for a Checkov finding that is allowed to pass PR validation because the central suppression file lists it as an accepted exception.
_Avoid_: Skip, ignore, false positive

**Workflow-log finding**:
A Checkov finding reported by failing the GitHub Actions workflow output rather than being uploaded to GitHub code scanning as SARIF.
_Avoid_: SARIF finding, Security tab finding

**Compliance mapping**:
The documented relationship between Checkov policy checks and external or client-specific frameworks such as CIS, NIST, FedRAMP, PCI DSS, SOC 2, HIPAA, or ISO 27001.
_Avoid_: Compliance, certification

**Cost visibility**:
Comment-only pull-request feedback that estimates the cost impact of Terraform changes before merge or apply.
_Avoid_: Cost control, FinOps enforcement
