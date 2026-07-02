---
name: cicd-pipeline-generator
description: Generate a CI/CD pipeline for a .NET app — GitHub Actions or Azure DevOps — with restore, build, test, coverage, vulnerability scan, container build, and deploy stages. Use this skill whenever the user wants a CI/CD pipeline, GitHub Actions workflow, Azure DevOps YAML, build/test automation, a release pipeline, or asks "how do I set up CI / pipeline for .NET / automate build and deploy". Always use this skill for CI/CD requests; it generates a complete, sensible pipeline.
category: DevOps
version: 1.0.0
---

# CI/CD Pipeline Generator

Generate a complete CI/CD pipeline for a .NET app — GitHub Actions or Azure DevOps — covering build, test, coverage, security scan, container image, and deploy.

## Input — point it at your project
Works on a target: a **repo** or **GitHub URL**. Detect platform (existing `.github/workflows` or `azure-pipelines.yml`), TFM, test projects, and whether there's a Dockerfile: `find <target> -name "*.csproj" -o -name "Dockerfile" -o -name "*.sln"`.

## GitHub Actions (CI)
```yaml
name: ci
on:
  push: { branches: [main] }
  pull_request: { branches: [main] }
jobs:
  build-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-dotnet@v4
        with: { dotnet-version: '9.0.x' }
      - run: dotnet restore
      - run: dotnet build --no-restore -c Release
      - run: dotnet test --no-build -c Release --collect:"XPlat Code Coverage" --logger trx
      - run: dotnet list package --vulnerable --include-transitive   # fail the build on CVEs
      - uses: actions/upload-artifact@v4
        with: { name: coverage, path: '**/coverage.cobertura.xml' }
```
Add a container job (build + push to a registry) and a deploy job (environment-gated) when a Dockerfile/target exists.

## Azure DevOps variant
Equivalent stages in `azure-pipelines.yml` (`DotNetCoreCLI@2` tasks for restore/build/test/publish, `Docker@2` for image, environments for approvals).

## What it wires
- Restore → build → test (with coverage) → **vulnerability scan** → publish/container → deploy.
- PR triggers for fast feedback; `main` for release.
- Secrets via the platform's secret store (never inline).
- Caching for `~/.nuget/packages` to speed restores.

## Principles
- **Fail fast on quality + security** — tests and `--vulnerable` should break the build.
- Keep CI fast (cache NuGet, `--no-restore`/`--no-build` between steps).
- Secrets come from the platform, never YAML literals.
- Gate deploys with environments/approvals; don't auto-ship to prod on every commit unless that's intended.

## How to use it & best prompts
"Set up GitHub Actions for my .NET app", "Azure DevOps pipeline with tests and coverage", "add a vulnerability scan to CI", "build and push a container, then deploy". Pairs with `dockerfile-generator` and `dependency-vuln-scanner`.
