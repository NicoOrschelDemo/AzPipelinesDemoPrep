# CI/CD Migration Report – Renovate Pipeline

## Summary

| Field | Value |
|---|---|
| **Source System** | Azure DevOps Pipelines |
| **Target System** | GitHub Actions |
| **Original File** | `azure-pipelines_renovate.yml` |
| **Migrated Workflow** | `.github/workflows/renovate.yml` |
| **Migration Date** | 2026-06-22 |
| **Status** | ✅ Complete |

---

## Pipeline Analysis

### Original Pipeline: `azure-pipelines_renovate.yml`

**Trigger:** Push to `main` branch  
**Agent Pool:** `ubuntu-latest`  
**Variables:**
- `buildConfiguration` = `Release`
- `solutionPath` = `TailwindTraders.Website/Source`
- `system_accesstoken` = `$(System.AccessToken)` *(Azure DevOps service identity token)*

**Steps:**
1. `dotnet build $(solutionPath) --configuration $(buildConfiguration)`
2. `AdvancedSecurity-Dependency-Scanning@1`
3. `BuildQualityChecks@9` (warning threshold enforcement)
4. `NodeTool@0` – Node.js 22.x
5. `RenovateMe@1` – Run Renovate

---

## Conversion Details

### Triggers

| Azure DevOps | GitHub Actions |
|---|---|
| `trigger: [main]` | `on: push: branches: [main]` |
| *(none)* | `on: schedule: cron: '0 3 * * *'` *(added – Renovate typically runs on a schedule)* |
| *(none)* | `on: workflow_dispatch` *(added for manual runs)* |

### Runner

| Azure DevOps | GitHub Actions |
|---|---|
| `pool: vmImage: ubuntu-latest` | `runs-on: ubuntu-latest` |

### Variables / Environment

| Azure DevOps Variable | GitHub Actions |
|---|---|
| `buildConfiguration: Release` | `env.BUILD_CONFIGURATION: Release` |
| `solutionPath: TailwindTraders.Website/Source` | `env.SOLUTION_PATH: TailwindTraders.Website/Source` |
| `system_accesstoken: $(System.AccessToken)` | `${{ secrets.RENOVATE_TOKEN \|\| secrets.GITHUB_TOKEN }}` *(see Secrets section)* |

### Steps

| # | Azure DevOps Task | GitHub Actions Equivalent |
|---|---|---|
| 1 | `script: dotnet build ...` | `run: dotnet build ...` |
| 2 | `AdvancedSecurity-Dependency-Scanning@1` | ⚠️ Not directly migrated – see Notes |
| 3 | `BuildQualityChecks@9` | ⚠️ Not directly migrated – see Notes |
| 4 | `NodeTool@0` (22.x) | `actions/setup-node@v4` with `node-version: '22.x'` |
| 5 | `RenovateMe@1` | `renovatebot/github-action@v41` |

---

## Required Secrets & Variables

| Name | Type | Description |
|---|---|---|
| `RENOVATE_TOKEN` | Repository Secret *(recommended)* | Personal Access Token or GitHub App token for Renovate. Needs `repo` and `workflow` scopes. Falls back to `GITHUB_TOKEN` if not set. |
| `GITHUB_TOKEN` | Built-in | Automatically provided by GitHub Actions. Used as fallback if `RENOVATE_TOKEN` is not configured. |

---

## Manual Setup Required

### 1. Dependabot / Dependency Scanning
`AdvancedSecurity-Dependency-Scanning@1` is an Azure DevOps Advanced Security task. GitHub handles dependency vulnerability scanning natively:

- **Dependabot Alerts**: Enable in repository **Settings → Security & analysis → Dependabot alerts**
- **Dependabot Security Updates**: Enable in repository **Settings → Security & analysis → Dependabot security updates**
- **Dependency Review** (for PRs): Available via `actions/dependency-review-action` in a PR-triggered workflow

### 2. BuildQualityChecks
`BuildQualityChecks@9` is an Azure Pipelines extension with no direct GitHub Actions equivalent. Equivalent functionality options:

- Use compiler warnings-as-errors (`--warnaserror`) in the `dotnet build` command
- Add a custom script to fail the job when MSBuild warning counts exceed a threshold
- Use branch protection rules to enforce check requirements

### 3. Renovate Token
For full Renovate functionality (creating/merging PRs), create a `RENOVATE_TOKEN` repository secret with a PAT that has `repo` and `workflow` scopes. Without it, the workflow falls back to `GITHUB_TOKEN` which may have limited permissions for some Renovate operations.

---

## Validation

- ✅ Workflow validated with **actionlint v1.7.12** – no errors

---

## Archived Files

| File | Location |
|---|---|
| Original Azure Pipeline | `.github/ci-archive/azure-pipelines_renovate.yml` |
| This report | `.github/ci-archive/MIGRATION-README.md` |
