# 🚀 Azure DevOps to GitHub Actions Migration Report

## 📊 Migration Overview

| Metric          | Before (Azure DevOps)          | After (GitHub Actions)          |
| --------------- | ------------------------------ | ------------------------------- |
| Pipeline Files  | 1 file (tailwindtraders-build.yml) | 1 workflow (.github/workflows/tailwindtraders-build.yml) |
| Pipeline Stages | 1 stage (Build)                | 1 job (build)                   |
| Pipeline Jobs   | 1 job (Build on Windows)       | 1 job / 9 steps                 |
| Templates       | 0 templates                    | N/A                             |

## 🔄 Conversion Diagram

```mermaid
graph LR
    A[Azure DevOps Pipeline] --> B[GitHub Actions Workflow]

    subgraph "Azure DevOps Structure"
        D1[Stage: Build]
        D2[Job: Build on Windows]
        D3[NodeTool@0]
        D4[UseDotNet@2]
        D5[Npm@1]
        D6[DotNetCoreCLI@2 restore]
        D7[PowerShell generate AssemblyInfo]
        D8[VersionAssemblies@2]
        D9[DotNetCoreCLI@2 build]
        D10[AdvancedSecurity-Dependency-Scanning@1]
        D11[AdvancedSecurity-Publish@1]
        D12[DotNetCoreCLI@2 test]
        D13[script echo publishKey]
    end

    subgraph "GitHub Actions Structure"
        G1[Job: build]
        G2[actions/checkout]
        G3[actions/setup-node]
        G4[actions/setup-dotnet]
        G5[npm install]
        G6[dotnet restore]
        G7[pwsh: Generate AssemblyInfo.cs]
        G8[mingjun97/file-regex-replace]
        G9[dotnet build]
        G10[GitHub GHAS - native Dependabot]
        G11[dotnet test]
        G12[echo PUBLISH_KEY secret]
    end

    D1 --> G1
    D2 --> G1
    D3 --> G3
    D4 --> G4
    D5 --> G5
    D6 --> G6
    D7 --> G7
    D8 --> G8
    D9 --> G9
    D10 --> G10
    D11 --> G10
    D12 --> G11
    D13 --> G12
```

## 🔧 Key Transformations

### Stage/Job Conversions

- Azure DevOps `stages:` → GitHub Actions `jobs:` (single stage = single job)
- Azure DevOps `pool: vmImage: windows-latest` → `runs-on: windows-latest`
- Azure DevOps `variables:` block → `env:` at workflow level

### Task and Variable Mappings

| Azure DevOps Task | GitHub Actions Equivalent | Notes |
|---|---|---|
| `NodeTool@0` | `actions/setup-node@v6.4.0` | Node 10.16.3 |
| `UseDotNet@2` | `actions/setup-dotnet@v5.4.0` | .NET SDK 5.x |
| `Npm@1` | `run: npm install` | npm install with working-directory |
| `DotNetCoreCLI@2 restore` | `run: dotnet restore` | |
| `PowerShell` | `run:` with `shell: pwsh` | AssemblyInfo generation preserved as-is |
| `VersionAssemblies@2` | `mingjun97/file-regex-replace@v1` | Regex version replacement in AssemblyInfo.cs |
| `DotNetCoreCLI@2 build` | `run: dotnet build` | |
| `AdvancedSecurity-Dependency-Scanning@1` | GitHub GHAS (native) | No explicit step needed; handled by Dependabot |
| `AdvancedSecurity-Publish@1` | GitHub GHAS (native) | Results automatically processed by GitHub GHAS |
| `DotNetCoreCLI@2 test` | `run: dotnet test` | |
| `script: echo $(publishKey)` | `run: echo "${{ secrets.PUBLISH_KEY }}"` | Secret variable reference migrated |

### Build Number / Version Strategy

Azure DevOps pipeline name `$(MajorVersion).$(MinorVersion).$(Rev:r)` → GitHub Actions uses `${{ github.run_number }}` as the revision:
- `MAJOR_VERSION=2`, `MINOR_VERSION=0` defined as `env:` variables
- Version at runtime: `2.0.<run_number>`

### Structural Changes

- Added explicit `actions/checkout` step (implicit in Azure DevOps)
- Added `permissions: contents: read` for least-privilege security
- `AdvancedSecurity-Dependency-Scanning@1` / `AdvancedSecurity-Publish@1` removed from steps — GitHub Advanced Security handles dependency scanning natively via Dependabot (no explicit workflow step needed when GHAS is enabled on the repository)
- `advancedsecurity.submittoadvancedsecurity` variable is an Azure DevOps-specific setting with no GitHub equivalent

## ✅ Validation Results

### Linting Results

```
$ actionlint .github/workflows/tailwindtraders-build.yml
(no output — all checks passed)

Exit code: 0
```

### Manual Verification Checklist

- [x] YAML syntax validated
- [x] All actions pinned to commit SHAs with version comments
- [x] Job dependencies verified (single job, no dependencies needed)
- [x] Environment variables migrated
- [x] Secrets and variables properly referenced
- [x] Triggers match original behavior (push to main)

## 🔐 Security Improvements

- Added explicit `permissions: contents: read` (least-privilege GITHUB_TOKEN)
- All actions pinned to commit SHAs instead of floating tags for supply-chain security
- `$(publishKey)` secret variable migrated to GitHub Secrets (`PUBLISH_KEY`)
- Azure DevOps variable groups replaced with GitHub Secrets / Variables

## 📈 Performance Enhancements

- Dependency caching can be added via `actions/setup-dotnet` built-in cache or `actions/cache@v4` targeting NuGet and npm caches for faster builds

## 🔗 Variable and Secret Requirements

### Required GitHub Secrets

| Secret Name | Description | Source |
|---|---|---|
| `PUBLISH_KEY` | Secret used in the deployment echo step | Azure DevOps `$(publishKey)` variable |

### Required GitHub Variables (optional, currently hardcoded)

| Variable Name | Value | Description |
|---|---|---|
| `MAJOR_VERSION` | `2` | Major version component (currently hardcoded in `env:`) |
| `MINOR_VERSION` | `0` | Minor version component (currently hardcoded in `env:`) |

## 🎯 Next Steps

1. **Enable GitHub Advanced Security** on the repository to activate Dependabot dependency scanning (replaces `AdvancedSecurity-Dependency-Scanning@1` + `AdvancedSecurity-Publish@1`)
2. **Configure secret** `PUBLISH_KEY` in repository Settings → Secrets and variables → Actions
3. **Test the workflow** by pushing to the `main` branch
4. **Optionally configure** `MAJOR_VERSION` / `MINOR_VERSION` as repository variables (currently hardcoded in `env:`)
5. **Upgrade Node.js** from EOL version 10.16.3 to an actively maintained LTS (18.x or 20.x)
6. **Upgrade .NET** from EOL version 5.x to a supported LTS version (.NET 8 or later)
7. **Add dependency caching** for NuGet and npm to speed up builds (optional optimisation)
8. **Update team documentation** with new workflow information

## 📁 Original Azure DevOps Files

The original Azure DevOps pipeline file has been moved to `.github/ci-archive/` for reference:

- `tailwindtraders-build.yml` → [`.github/ci-archive/tailwindtraders-build.yml`](.github/ci-archive/tailwindtraders-build.yml)

## 📚 Migration Notes

- **EOL versions preserved**: Node.js 10.16.3 (EOL April 2021) and .NET 5.x (EOL May 2022) are faithfully preserved from the original pipeline. Both versions receive no security updates. Upgrading to Node.js 18.x/20.x and .NET 8 (LTS) is strongly recommended as a follow-up task.
- **VersionAssemblies@2**: Converted to `mingjun97/file-regex-replace@v1` (org-approved action per knowledge base). The original used `versionSource: buildNumber` with `threeParts` format — the GitHub Actions equivalent uses `github.run_number` as the revision counter and applies a regex replacement on `**/AssemblyInfo.cs` files. Note: the original `failIfNoMatchFound: true` behaviour is not replicated by the replacement action.
- **Advanced Security tasks**: `AdvancedSecurity-Dependency-Scanning@1` and `AdvancedSecurity-Publish@1` are Azure DevOps Advanced Security tasks. On GitHub, these capabilities are provided natively by GitHub Advanced Security (Dependabot alerts, dependency graph). No explicit workflow steps are needed when GHAS is enabled.
- **Build number format**: Azure DevOps `$(MajorVersion).$(MinorVersion).$(Rev:r)` maps to `2.0.<github.run_number>`. The run number resets per repository, not per workflow, so initial values will differ from the Azure DevOps counter.
- **`resource-group` and `webapp_name` variables**: Present in the original pipeline but unused in any build step. They are noted as comments in the workflow and can be configured as repository variables if deployment steps are added later.

---
*Migration completed by GitHub Copilot Azure DevOps Migration Agent*
