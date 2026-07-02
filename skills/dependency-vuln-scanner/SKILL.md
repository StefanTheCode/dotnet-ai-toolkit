---
name: dependency-vuln-scanner
description: Scan a .NET project's NuGet dependencies for known vulnerabilities, deprecated/outdated packages, and transitive risks, then recommend safe upgrades. Use this skill whenever the user wants to check dependencies for CVEs, vulnerable packages, outdated NuGet, transitive vulnerabilities, supply-chain risk, or asks "are my packages safe / any vulnerable dependencies / dotnet list package vulnerable". Always use this skill for dependency-security requests; it runs the scan and prioritizes fixes.
category: Security
version: 1.0.0
---

# Dependency Vulnerability Scanner

Check NuGet dependencies (direct and transitive) for known CVEs, deprecation, and staleness, then recommend the safest upgrade path.

## Input — point it at your project
Works on a target: a **project/solution** or **GitHub URL**. Run the built-in scanners:
```
dotnet restore <target>
dotnet list <target> package --vulnerable --include-transitive
dotnet list <target> package --deprecated
dotnet list <target> package --outdated
```
If you can't restore (offline), inspect versions in `.csproj`/`packages.lock.json` and flag known-risky ranges, marking results "verify".

## What it reports
1. 🔴 **Known vulnerabilities** — package, installed version, severity, advisory, and the fixed version to move to. Transitive ones include the path that pulls them in.
2. 🟠 **Deprecated packages** — with the recommended replacement.
3. 🟡 **Significantly outdated** — major versions behind (security fixes you're missing even without a listed CVE).
4. **Transitive pulls** — a safe direct package dragging in a vulnerable transitive one → pin or update the parent, or add a direct reference to force the fixed version.

## Upgrade guidance
- Prefer the **minimal version bump** that clears the advisory (lower regression risk).
- For transitive vulns, update the parent if it ships a fixed transitive; otherwise add a direct `PackageReference` to the patched version.
- Use `packages.lock.json` for reproducible, auditable restores.
- Note breaking-change risk for major bumps and suggest testing.

## Principles
- **Transitive dependencies are your attack surface too** — scan `--include-transitive`.
- Patch the advisory with the smallest safe bump; don't blindly jump majors.
- Make it repeatable — wire `dotnet list package --vulnerable` into CI so new CVEs surface automatically.

## How to use it & best prompts
"Scan my project for vulnerable packages", "any CVEs in my dependencies", "what NuGet packages should I update for security", "this transitive package is flagged — how do I fix it". Pairs with `nuget-dependency-analyzer` and the `aspnetcore-security-auditor` agent.
