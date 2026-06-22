# Migration Report: azure-pipelines_renovate.yml → GitHub Actions

## Summary

| Field | Value |
|---|---|
| **Source File** | `azure-pipelines_renovate.yml` |
| **Target Workflow** | `.github/workflows/renovate.yml` |
| **Migration Date** | 2026-06-22 |
| **Migrated By** | GitHub Copilot Azure DevOps Migration Agent |
| **Validation** | ✅ actionlint passed — no errors |

---

## Pipeline Analysis

The original pipeline triggered on pushes to `main` and performed four operations:
1. Build the .NET solution in Release configuration
2. Run Azure Advanced Security dependency scanning
3. Check build warning counts (via BuildQualityChecks)
4. Set up Node.js and run the Renovate dependency-update bot

---

## Task Mapping

| Azure DevOps Task | GitHub Actions Equivalent | Notes |
|---|---|---|
| `trigger: [main]` | `on: push: branches: [main]` | Direct equivalent |
| `pool: vmImage: ubuntu-latest` | `runs-on: ubuntu-latest` | Direct equivalent |
| `variables: buildConfiguration / solutionPath` | `env: BUILD_CONFIGURATION / SOLUTION_PATH` | Workflow-level env vars |
| `system_accesstoken: $(System.AccessToken)` | `GITHUB_TOKEN` (implicit) | Not explicitly needed; permissions block covers required access |
| `script: dotnet build …` | `run: dotnet build …` | Direct equivalent |
| `AdvancedSecurity-Dependency-Scanning@1` | `actions/dependency-review-action@v4` | See note below |
| `BuildQualityChecks@9` | ⚠️ No direct equivalent | See note below |
| `NodeTool@0` (22.x) | `actions/setup-node@v4` (22.x) | Direct equivalent |
| `RenovateMe@1` | `renovatebot/github-action@v41` | See note below |

---

## Migration Notes

### AdvancedSecurity-Dependency-Scanning@1

GitHub Advanced Security dependency scanning is a native GitHub platform feature enabled at the repository/organisation level. It does not require a workflow step to activate. The `actions/dependency-review-action@v4` step provides complementary push-time dependency review with configurable severity thresholds.

**Recommended follow-up:** Ensure Dependabot and the GitHub dependency graph are enabled in *Settings → Code security and analysis*. Add a `.github/dependabot.yml` configuration to automate dependency-update PRs independently of this workflow if desired.

### BuildQualityChecks@9

This Azure DevOps Marketplace extension enforced a maximum warning count for prior build steps. **There is no direct GitHub Actions equivalent.** The step has been removed from the migrated workflow.

**Recommended follow-up:** Implement a custom warning-threshold check using `dotnet build --warnaserror` or by parsing MSBuild output if this gate is required.

### RenovateMe@1 → renovatebot/github-action@v41

The Renovate task requires an authentication token with repository read/write access to open dependency-update PRs.

**Required secret:** Create a repository (or organisation) secret named **`RENOVATE_TOKEN`** containing either:
- A **Personal Access Token (classic)** with `repo` scope, or
- A **GitHub App** installation token via the `tibdex/github-app-token` action.

The `GITHUB_TOKEN` automatically available to workflows has limited permissions and **cannot** open PRs targeting the default branch without branch protection bypass rules, so a dedicated token is strongly recommended.

**Recommended follow-up:** Add a `renovate.json` configuration file at the repository root if one does not already exist. Renovate will error if `configurationFile: renovate.json` is set but the file is missing.

---

## Required Secrets / Variables

| Name | Type | Description |
|---|---|---|
| `RENOVATE_TOKEN` | Secret | PAT or GitHub App token for Renovate to open dependency-update PRs |

---

## Files Changed

| Action | Path |
|---|---|
| Created | `.github/workflows/renovate.yml` |
| Archived | `.github/ci-archive/azure-pipelines_renovate.yml` |
| Deleted | `azure-pipelines_renovate.yml` (root) |
