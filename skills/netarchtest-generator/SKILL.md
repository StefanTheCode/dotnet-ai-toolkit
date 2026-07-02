---
name: netarchtest-generator
description: Generate architecture tests (NetArchTest or ArchUnitNET) that fail the build when architectural rules are violated — layer dependencies, naming, sealed/internal conventions, and namespace boundaries. Use this skill whenever the user wants architecture tests, to enforce layering, dependency rules as tests, naming conventions, fitness functions, or asks "how do I stop people breaking the architecture / enforce my layers". Always use this skill for architecture-test requests; it generates rules matching the project's structure.
category: Testing
version: 1.0.0
---

# Architecture Tests Generator (NetArchTest)

Turn architectural rules into automated tests so violations fail the build instead of slowly rotting the codebase — layering, dependencies, naming, and boundary rules tailored to the project.

## Input — point it at your project
Works on a target: a **solution/project** or **GitHub URL**. Detect the structure to enforce: `find <target> -name "*.csproj"`, and the style (Clean layers? Vertical Slice? modules?) via folder names and references.

## Setup
```
dotnet add package NetArchTest.Rules
```

## Rules (tailored to the detected architecture)
**Clean Architecture dependency rule:**
```csharp
[Fact]
public void Domain_has_no_outward_dependencies()
{
    var r = Types.InAssembly(typeof(Order).Assembly)
        .Should().NotHaveDependencyOnAny("MyApp.Application", "MyApp.Infrastructure", "MyApp.Api")
        .GetResult();
    Assert.True(r.IsSuccessful, string.Join("\n", r.FailingTypeNames ?? new()));
}
```
**Other common fitness functions:**
```csharp
// Handlers must be sealed
Types.InAssembly(asm).That().HaveNameEndingWith("Handler").Should().BeSealed();
// Controllers/endpoints don't reference DbContext directly
Types.InAssembly(asm).That().HaveNameEndingWith("Controller").ShouldNot().HaveDependencyOn("MyApp.Infrastructure.Data");
// Modules don't reference each other's internals (modular monolith)
Types.InAssembly(ordersAsm).ShouldNot().HaveDependencyOn("MyApp.Modules.Billing");
// Async methods end with "Async"
Types.InAssembly(asm).That().HaveNameMatching(".*").Should()...; // naming conventions
```

## Principles
- **Encode the rules you actually care about** — every test should map to a real architectural decision.
- Architecture tests are **fitness functions**: cheap, fast, run in CI, fail loudly.
- Match the rules to the real structure (don't impose Clean rules on a Vertical Slice app).
- A handful of high-value rules beats dozens of pedantic ones.

## How to use it & best prompts
"Add architecture tests to enforce my layers", "stop controllers from using DbContext", "enforce that handlers are sealed", "guardrail tests for my modular monolith". Pairs with `clean-architecture-scaffolder`, `modular-monolith-generator`, and the `dotnet-architecture-reviewer` agent.
