---
name: modular-monolith-generator
description: Structure a .NET app as a modular monolith — independent modules with enforced boundaries, public contracts, and in-process messaging between modules. Use this skill whenever the user wants a modular monolith, module boundaries, to split a monolith into modules, internal-by-default modules, module contracts, or asks "how do I structure a modular monolith / keep modules decoupled". Always use this skill for modular-monolith requests; it sets up modules, boundaries, and a guardrail test.
category: Architecture
version: 1.0.0
---

# Modular Monolith Generator

Set up (or tighten) a modular monolith: each module owns its domain, exposes a small public contract, keeps everything else `internal`, and talks to other modules through contracts or an in-process bus — so you get module independence without microservice overhead.

## Input — point it at your project
Works on a target: a **file**, **folder**, **project/solution**, or **GitHub URL**. If modules exist, audit their boundaries; if not, propose a split. Detect with: `find <target> -name "*.csproj"` and `grep -rn "public class\|internal class" --include=*.cs <target> | head`.

## Module layout
```
src/
├── Modules/
│   ├── Orders/
│   │   ├── Orders.Public/        # contracts other modules may use (DTOs, integration events, a façade interface)
│   │   └── Orders/               # internal: domain, handlers, EF — internal by default
│   ├── Billing/
│   │   ├── Billing.Public/
│   │   └── Billing/
├── Shared/                       # truly shared kernel only (Result, base types)
└── Host/                         # composition root: wires every module
```

## Module registration (self-contained)
```csharp
public interface IModule
{
    IServiceCollection RegisterModule(IServiceCollection services, IConfiguration config);
    void MapEndpoints(IEndpointRouteBuilder app);
}
// Host discovers and calls every IModule — no module references another module's internals.
```

## Boundaries — enforce them, don't just hope
- A module references only other modules' **`.Public`** projects, never their internals.
- Cross-module calls go through a public façade interface or **integration events** on an in-process bus (MediatR notifications, or a channel) — not direct DB access into another module's tables.
- One schema per module (or clearly owned tables); no foreign keys across module boundaries.

## Guardrail test (NetArchTest)
```csharp
[Fact]
public void Orders_internals_are_not_referenced_by_other_modules()
{
    var result = Types.InAssembly(typeof(BillingModule).Assembly)
        .Should().NotHaveDependencyOn("Company.Modules.Orders") // internal assembly
        .GetResult();
    Assert.True(result.IsSuccessful);
}
```

## Principles
- **Internal by default.** If it's not in `.Public`, no other module can touch it.
- **Boundaries enforced by tests**, not documentation — otherwise they erode.
- **A modular monolith is a migration path:** clean module boundaries make a later extraction to services cheap. Don't pay the distributed-systems tax until you need it.
- Keep the shared kernel tiny — it's a coupling magnet.

## How to use it & best prompts
"Structure this app as a modular monolith", "audit my module boundaries", "split this monolith into Orders/Billing/Catalog modules with contracts", "add a guardrail test so modules can't reach into each other".
