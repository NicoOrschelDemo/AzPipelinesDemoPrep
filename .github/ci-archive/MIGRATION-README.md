# 🚀 Azure DevOps to GitHub Actions Migration Report

## 📊 Migration Overview

| Metric          | Before (Azure DevOps)         | After (GitHub Actions)          |
| --------------- | ----------------------------- | ------------------------------- |
| Pipeline Files  | 1 file (azure-pipelines-codeql.yml) | 1 workflow (codeql.yml)    |
| Pipeline Stages | 1 stage (Build)               | 1 job (analyze)                 |
| Pipeline Jobs   | 1 job (Build on Windows)      | 1 job / 5 steps                 |
| Templates       | None                          | N/A                             |

## 🔄 Conversion Diagram

```mermaid
graph LR
    A[Azure DevOps Pipeline] --> B[GitHub Actions Workflow]

    subgraph "Azure DevOps Structure"
        D1[Stage: Build]
        D2[Job: Build on Windows]
        D3[NodeTool@0]
        D4[AdvancedSecurity-Codeql-Init@1]
        D5[AdvancedSecurity-Codeql-Autobuild@1]
        D6[AdvancedSecurity-Codeql-Analyze@1]
    end

    subgraph "GitHub Actions Structure"
        G1[Job: analyze - windows-latest]
        G2[actions/checkout@v7]
        G3[actions/setup-node@v6]
        G4[github/codeql-action/init@v4]
        G5[github/codeql-action/autobuild@v4]
        G6[github/codeql-action/analyze@v4]
    end

    D1 --> G1
    D2 --> G1
    D3 --> G3
    D4 --> G4
    D5 --> G5
    D6 --> G6
```

## 🔧 Key Transformations

### Stage/Job Conversions

- Azure DevOps single stage + single job → single GitHub Actions job
- `pool: vmImage: windows-latest` → `runs-on: windows-latest`
- `trigger: [main]` + `pr: [main]` → `on: push/pull_request: branches: [main]`
- Pipeline-level `variables:` → job-level `env:` (BuildConfiguration, BuildPlatform)

### Task Mappings

| Azure DevOps Task | GitHub Actions Equivalent | Version |
|---|---|---|
| `NodeTool@0` (versionSpec: 10.16.3) | `actions/setup-node` | v6.4.0 (SHA-pinned) |
| `AdvancedSecurity-Codeql-Init@1` | `github/codeql-action/init` | v4.36.2 (SHA-pinned) |
| `AdvancedSecurity-Codeql-Autobuild@1` | `github/codeql-action/autobuild` | v4.36.2 (SHA-pinned) |
| `AdvancedSecurity-Codeql-Analyze@1` | `github/codeql-action/analyze` | v4.36.2 (SHA-pinned) |

### Parameters Handled

| Azure DevOps Input | GitHub Actions Equivalent | Notes |
|---|---|---|
| `enableAutomaticCodeQLInstall: true` | (not needed) | CodeQL is automatically available in `github/codeql-action` |
| `languages: 'csharp, javascript'` | `languages: csharp, javascript` | Direct equivalent |
| `querysuite: 'security-extended'` | `queries: security-extended` | Parameter renamed |
| `WaitForProcessing: true` | (not needed) | GitHub processes SARIF asynchronously; no polling required |
| `advancedsecurity.submittoadvancedsecurity: true` | (implicit) | GHAS always receives results from `github/codeql-action/analyze` |

## ✅ Validation Results

### Linting Results (actionlint v1.7.12)

```
$ actionlint .github/workflows/codeql.yml
(no output)
Exit code: 0
```

✅ No errors, no warnings.

### Manual Verification Checklist

- [x] YAML syntax validated (actionlint clean)
- [x] All actions properly SHA-pinned with version comments
- [x] Job dependencies verified (single job, no dependencies needed)
- [x] Environment variables migrated (BuildConfiguration, BuildPlatform)
- [x] Secrets and variables: none required for this workflow
- [x] Triggers match original behavior (push + PR to main)
- [x] Runner preserved (windows-latest)

## 🔐 Security Improvements

- All actions pinned to commit SHAs (never mutable tags) per security guardrails
- Minimal `permissions:` block added: `security-events: write`, `actions: read`, `contents: read`
- Original Azure DevOps pipeline had no explicit permission scoping; GitHub Actions enforces least-privilege

## 🔗 Variable and Secret Requirements

### Required GitHub Secrets

_None required._ This CodeQL workflow uses only the built-in `GITHUB_TOKEN` (automatically provided).

### Required GitHub Variables

_None required._ The `BuildConfiguration` and `BuildPlatform` values are hardcoded in the workflow `env:` block (matching the original pipeline values of `Release` and `any cpu`).

### GitHub Advanced Security

- Ensure **GitHub Advanced Security** is enabled on the repository so CodeQL SARIF results are uploaded and displayed in the Security tab.
- Results from `github/codeql-action/analyze` are automatically submitted — no additional configuration needed.

## 🎯 Next Steps

1. **Enable GitHub Advanced Security** on the repository (Settings → Security → Code security and analysis)
2. **Push the branch / merge to main** to trigger the first CodeQL run
3. **Review Code Scanning alerts** in the Security tab after the first run
4. **(Optional) Upgrade Node.js version** — Node 10.16.3 is EOL; consider bumping to a supported LTS version once the team is ready (preserved faithfully from the original pipeline)

## 📁 Original Azure DevOps File

The original Azure DevOps pipeline has been moved to `.github/ci-archive/` for reference:

- `azure-pipelines-codeql.yml` → [`.github/ci-archive/azure-pipelines-codeql.yml`](azure-pipelines-codeql.yml)

---
*Migration completed by GitHub Copilot Azure DevOps Migration Agent*
