# Migration Report: azure-pipelines_renovate.yml → GitHub Actions

## Summary

| Field | Value |
|-------|-------|
| **Source file** | `azure-pipelines_renovate.yml` |
| **Target file** | `.github/workflows/renovate.yml` |
| **Migration date** | 2026-06-22 |
| **Status** | ✅ Complete |

---

## Original Pipeline Analysis

The source pipeline (`azure-pipelines_renovate.yml`) was a single-stage, single-job Azure Pipeline that:

1. Triggered on pushes to `main`
2. Ran on `ubuntu-latest`
3. Built a .NET solution
4. Ran Azure DevOps Advanced Security dependency scanning
5. Applied build quality checks (warning thresholds)
6. Set up Node.js 22.x
7. Ran the Renovate dependency-update bot via `RenovateMe@1`

---

## Step-by-Step Mapping

| Azure DevOps Step | GitHub Actions Equivalent | Notes |
|---|---|---|
| `trigger: - main` | `on: push: branches: [main]` + `schedule` + `workflow_dispatch` | Added scheduled run and manual trigger — standard Renovate best practice |
| `pool: vmImage: ubuntu-latest` | `runs-on: ubuntu-latest` | Direct equivalent |
| `variables: buildConfiguration / solutionPath` | `env: BUILD_CONFIGURATION / SOLUTION_PATH` | Workflow-level `env` block |
| `variables: system_accesstoken: $(System.AccessToken)` | `secrets.RENOVATE_TOKEN` | See **Required Secrets** below |
| `script: dotnet build ...` | `run: dotnet build ...` | Direct shell equivalent |
| `AdvancedSecurity-Dependency-Scanning@1` | *(Dependabot — no workflow step)* | See **Manual Actions Required** below |
| `BuildQualityChecks@9` | *(no direct equivalent)* | See **Manual Actions Required** below |
| `NodeTool@0` (versionSpec: 22.x) | `actions/setup-node@v4` (node-version: 22.x) | Direct equivalent |
| `RenovateMe@1` (renovateOptionsVersion: latest) | `renovatebot/github-action@v40` (renovate-version: latest) | Direct equivalent |

---

## Actions Used

| Action | Version pinned to SHA | Purpose |
|--------|----------------------|---------|
| `actions/checkout` | `11bd71901bbe5b1630ceea73d27597364c9af683` (v4.2.2) | Repository checkout |
| `actions/setup-dotnet` | `3e891b0cb619bf60e2c25674b222b8940e2c1c25` (v4.1.0) | .NET 8 SDK setup |
| `actions/setup-node` | `49933ea5288caeca8642d1e84afbd3f7d6820020` (v4.4.0) | Node.js 22.x setup |
| `renovatebot/github-action` | `a4b7e8c10ef5e5614c6ceac36b2a29e62de2f11b` (v40.3.5) | Run Renovate bot (version pinned to 40.3.5) |

---

## Required Secrets

| Secret name | Description | Required |
|-------------|-------------|----------|
| `RENOVATE_TOKEN` | GitHub PAT with `repo` and `workflow` scopes, used by Renovate to create dependency-update PRs. Replaces Azure DevOps `System.AccessToken`. | ✅ Required |

**How to create:**
1. Go to **GitHub → Settings → Developer settings → Personal access tokens**
2. Create a token with `repo` and `workflow` scopes
3. Add it to the repository (or organisation) as a secret named `RENOVATE_TOKEN`

---

## Manual Actions Required

### 1. Enable GitHub Dependabot (replaces `AdvancedSecurity-Dependency-Scanning@1`)

Azure DevOps Advanced Security dependency scanning is replaced by GitHub's native Dependabot feature.

**Steps:**
1. Go to **Repository Settings → Security → Code security and analysis**
2. Enable **Dependency graph**, **Dependabot alerts**, and **Dependabot security updates**
3. Optionally add a `.github/dependabot.yml` configuration file for fine-grained control

No explicit workflow step is required — Dependabot runs automatically.

### 2. Build warning threshold enforcement (replaces `BuildQualityChecks@9`)

The original pipeline used `BuildQualityChecks@9` with `warningFailOption: fixed` and `warningThreshold: 10` to fail the build when warnings exceeded 10. There is no direct GitHub Actions equivalent.

**Options:**
- Add `-p:TreatWarningsAsErrors=true` to the `dotnet build` command to fail on any warning
- Add `-maxwarns 10` (MSBuild) to fail if warnings exceed 10
- Implement a custom shell step that counts warning lines in `dotnet build --verbosity normal` output

### 3. Renovate configuration file

Ensure a `renovate.json` (or `renovate.json5`) configuration file exists in the repository root or `.github/` directory. If migrating from Azure DevOps Renovate, the configuration syntax is the same.

---

## Archived Files

| Original location | Archived to |
|---|---|
| `azure-pipelines_renovate.yml` | `.github/ci-archive/azure-pipelines_renovate.yml` |
