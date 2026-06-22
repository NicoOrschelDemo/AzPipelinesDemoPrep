# CI/CD Migration: Azure DevOps → GitHub Actions

## Migration Summary

| Field | Value |
|-------|-------|
| **Source System** | Azure DevOps Pipelines |
| **Target System** | GitHub Actions |
| **Migration Date** | 2026-06-22 |
| **Migrated Pipeline** | Renovate |

---

## Source Files (Archived)

| Original File | Archived Location |
|---------------|------------------|
| `azure-pipelines_renovate.yml` | `.github/ci-archive/azure-pipelines_renovate.yml` |
| `Renovate.yml` | `.github/ci-archive/Renovate.yml` |

## Target Workflow

| GitHub Actions Workflow | Location |
|------------------------|----------|
| `renovate.yml` | `.github/workflows/renovate.yml` |

---

## Pipeline Mapping

### Triggers

| Azure DevOps | GitHub Actions |
|-------------|----------------|
| `trigger: [main]` | `on: push: branches: [main]` |
| *(manual/scheduled not defined)* | `on: schedule: cron: '0 2 * * *'` (added) |
| *(manual not defined)* | `on: workflow_dispatch` (added) |

### Runner / Pool

| Azure DevOps | GitHub Actions |
|-------------|----------------|
| `pool: vmImage: ubuntu-latest` | `runs-on: ubuntu-latest` |

### Variables

| Azure DevOps Variable | GitHub Actions Equivalent |
|----------------------|--------------------------|
| `buildConfiguration: 'Release'` | `env: BUILD_CONFIGURATION: 'Release'` |
| `solutionPath: 'TailwindTraders.Website/Source'` | `env: SOLUTION_PATH: 'TailwindTraders.Website/Source'` |
| `system_accesstoken: $(System.AccessToken)` | `secrets.RENOVATE_TOKEN` (see Secrets section) |

### Steps

| Azure DevOps Step | GitHub Actions Equivalent | Notes |
|------------------|--------------------------|-------|
| `script: dotnet build $(solutionPath) --configuration $(buildConfiguration)` | `run: dotnet build ${{ env.SOLUTION_PATH }} --configuration ${{ env.BUILD_CONFIGURATION }}` | Direct equivalent |
| `AdvancedSecurity-Dependency-Scanning@1` | GitHub Dependabot (native) | GitHub handles dependency vulnerability scanning natively; `actions/dependency-review-action` can be added to a PR workflow for inline PR checks |
| `BuildQualityChecks@9` (warnings threshold) | *(no direct equivalent)* | Warning thresholds can be enforced via MSBuild properties (`TreatWarningsAsErrors`, `WarningsAsErrors`) or custom scripts |
| `NodeTool@0 (versionSpec: 22.x)` | `actions/setup-node@v4 (node-version: '22')` | Direct equivalent |
| `RenovateMe@1 (renovateOptionsVersion: latest)` | `renovatebot/github-action@v40` | Direct equivalent |

---

## Required Secrets & Variables

The following secrets must be configured in the GitHub repository settings (**Settings → Secrets and variables → Actions**):

| Secret Name | Description | Required |
|-------------|-------------|----------|
| `RENOVATE_TOKEN` | Personal Access Token (PAT) with `repo` scope for Renovate to create PRs and update dependencies. Use `GITHUB_TOKEN` for same-repository operations only. | **Required** |

---

## Manual Actions Required After Migration

1. **Configure `RENOVATE_TOKEN` secret**: Create a GitHub PAT with `repo` scope (and `workflow` scope if Renovate should update workflow files) and store it as `RENOVATE_TOKEN` in repository secrets.

2. **Enable Dependabot**: Navigate to **Settings → Security → Code security and analysis** and enable:
   - Dependency graph
   - Dependabot alerts
   - Dependabot security updates
   
   This replaces the `AdvancedSecurity-Dependency-Scanning@1` task.

3. **Review build warning policy** (optional): If warning threshold enforcement from `BuildQualityChecks@9` is required, add `/warnaserror` or `-p:TreatWarningsAsErrors=true` to the `dotnet build` command, or configure it in the project's `.csproj` file.

4. **Add Renovate configuration file** (optional): If not already present, create a `renovate.json` at the repository root to configure Renovate behavior.

---

## Validation

Actionlint syntax validation was executed against `.github/workflows/renovate.yml` prior to merge.

---

## Notes on Task Equivalence

- **`AdvancedSecurity-Dependency-Scanning@1`**: This is an Azure DevOps Advanced Security task. GitHub's equivalent is Dependabot, which runs automatically at the repository level without requiring a workflow step. For PR-gating on dependency vulnerabilities, add `actions/dependency-review-action` to a separate `pull_request` workflow.

- **`BuildQualityChecks@9`**: This Azure DevOps Marketplace extension enforces warning thresholds from prior pipeline tasks. GitHub Actions does not have a native equivalent. The functionality can be replicated by treating compiler warnings as errors or writing a custom shell script that parses build output.

- **`RenovateMe@1`**: This Azure DevOps Marketplace task wraps Renovate bot execution. The direct GitHub Actions equivalent is `renovatebot/github-action`, which is the official Renovate GitHub Action.

- **`system_accesstoken`**: Azure DevOps uses `$(System.AccessToken)` for pipeline authentication. In GitHub Actions, this is replaced by `${{ secrets.GITHUB_TOKEN }}` (for same-repo operations) or a PAT stored as `RENOVATE_TOKEN`.
