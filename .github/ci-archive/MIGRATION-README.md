# Migration Report: azure-pipelines-codeql.yml → GitHub Actions

## Summary

| Field | Value |
|---|---|
| **Source** | `azure-pipelines-codeql.yml` |
| **Target** | `.github/workflows/codeql.yml` |
| **Migration Date** | 2026-06-22 |
| **Status** | ✅ Complete |

## Source Pipeline Analysis

The original Azure DevOps pipeline performed CodeQL security scanning using GitHub Advanced Security tasks for Azure DevOps:

- **Trigger:** Push and PR to `main`
- **Runner:** `windows-latest`
- **Languages:** `csharp`, `javascript`
- **Query suite:** `security-extended`
- **Node version:** `10.16.3`
- **Features:** Auto-build, wait for processing, submit to Advanced Security

## Conversion Mapping

| Azure DevOps | GitHub Actions |
|---|---|
| `trigger: - main` | `on: push: branches: [main]` |
| `pr: - main` | `on: pull_request: branches: [main]` |
| `pool: vmImage: windows-latest` | `runs-on: windows-latest` |
| `NodeTool@0` (versionSpec: 10.16.3) | `actions/setup-node@v4` (node-version: 10.16.3) |
| `AdvancedSecurity-Codeql-Init@1` | `github/codeql-action/init@v3` |
| `AdvancedSecurity-Codeql-Autobuild@1` | `github/codeql-action/autobuild@v3` |
| `AdvancedSecurity-Codeql-Analyze@1` (WaitForProcessing: true) | `github/codeql-action/analyze@v3` (wait-for-processing: true) |
| `querysuite: security-extended` | `queries: security-extended` |
| `advancedsecurity.submittoadvancedsecurity: true` | Automatic (GitHub GHAS uploads results automatically) |
| `languages: 'csharp, javascript'` | `matrix.language: [csharp, javascript]` (parallel jobs) |

## Design Decisions

1. **Matrix strategy:** The two CodeQL languages (`csharp` and `javascript`) are split into a matrix so they run as parallel jobs, which is the idiomatic GitHub Actions approach for CodeQL and reduces total scan time.

2. **Permissions:** Added `security-events: write`, `actions: read`, and `contents: read` permissions as required by `github/codeql-action`.

3. **GHAS submission:** The `advancedsecurity.submittoadvancedsecurity: true` variable is not needed in GitHub Actions — `codeql-action/analyze` automatically uploads SARIF results to GitHub Advanced Security.

4. **Build variables:** `BuildConfiguration`, `BuildPlatform`, `Parameters.RestoreBuildProjects`, and `Parameters.TestProjects` were ADO pipeline variables not actively used in any pipeline steps, so they have not been migrated.

5. **Node version notice:** Node.js 10.16.3 is end-of-life and may not be available on current runners. Consider upgrading to a supported LTS version (e.g., 20.x) after migration.

## Required Secrets / Variables

None required. CodeQL results are uploaded automatically using the built-in `GITHUB_TOKEN`.

## Post-Migration Steps

1. Ensure **GitHub Advanced Security** is enabled for this repository (Settings → Security → Code security and analysis).
2. Consider upgrading the Node.js version from `10.16.3` to a supported LTS version.
3. Delete or disable the original Azure DevOps pipeline in Azure DevOps.

## Archived Files

| File | Location |
|---|---|
| Original pipeline | `.github/ci-archive/azure-pipelines-codeql.yml` |
