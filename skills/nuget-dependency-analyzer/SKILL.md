---
name: nuget-dependency-analyzer
description: Analyze a .NET solution's NuGet dependencies — version conflicts, duplicate/inconsistent versions across projects, unused packages, and central package management opportunities. Use this skill whenever the user wants to analyze dependencies, fix version conflicts, consolidate package versions, set up Central Package Management, find unused packages, or asks "why do I have version conflicts / clean up my NuGet / consolidate versions". Always use this skill for dependency-hygiene requests; it maps the graph and recommends consolidation.
category: DevOps
version: 1.0.0
---

# NuGet Dependency Analyzer

Bring order to a multi-project solution's dependencies: spot version conflicts and drift, find unused/duplicate packages, and consolidate with Central Package Management.

## Input — point it at your solution
Works on a target: a **solution** or **GitHub URL**. Map versions across projects:
```
find <target> -name "*.csproj" | xargs grep -h "PackageReference" | sort | uniq -c | sort -rn
dotnet list <target> package                       # per-project versions
dotnet list <target> package --outdated
```

## What it finds
1. **Version drift** — the same package pinned to different versions across projects (a top source of runtime binding surprises). Consolidate to one.
2. **Transitive conflicts** — diamond dependencies resolved to a version no one intended.
3. **Unused packages** — referenced but not used (check namespaces actually imported).
4. **Inconsistent meta-packages** — mixing framework/package versions.

## Central Package Management (CPM)
Recommend and scaffold `Directory.Packages.props` so versions live in one place:
```xml
<!-- Directory.Packages.props (repo root) -->
<Project>
  <PropertyGroup><ManagePackageVersionsCentrally>true</ManagePackageVersionsCentrally></PropertyGroup>
  <ItemGroup>
    <PackageVersion Include="Serilog.AspNetCore" Version="10.0.0" />
    <PackageVersion Include="FluentValidation" Version="11.11.0" />
  </ItemGroup>
</Project>
```
Then `.csproj` files use `<PackageReference Include="..." />` with no version — one source of truth.

## Principles
- **One version per package across the solution** — drift causes the worst, hardest-to-repro bugs.
- **CPM is the cure for version sprawl** in multi-project repos.
- Remove what you don't use — fewer packages, smaller attack surface and faster restores.
- Pair with the vuln scanner: consolidation makes patching one place, not ten.

## How to use it & best prompts
"Analyze my solution's NuGet dependencies", "fix version conflicts across projects", "set up Central Package Management", "find unused packages". Pairs with `dependency-vuln-scanner`.
