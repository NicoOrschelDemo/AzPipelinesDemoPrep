# CI migration archive

This directory stores Azure DevOps pipeline definitions after they have been replaced by GitHub Actions workflows.

## Migrated pipelines

### Tailwind Traders build
- Archived pipeline: `tailwindtraders-build.yml`
- Replacement workflow: `.github/workflows/tailwindtraders-build.yml`
- Notes: preserves the main-branch Windows build, installs Node.js 10.16.3, restores and builds the .NET solution, and conditionally runs matching test projects when present.
