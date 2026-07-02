---
name: db-resiliency-setup
description: Add database resiliency to EF Core — connection retry on transient failures, command timeouts, and safe retry with transactions/execution strategy. Use this skill whenever the user wants connection resiliency, retry on failure, transient fault handling, EnableRetryOnFailure, execution strategy, or asks "how do I handle transient DB errors / make my DB calls resilient". Always use this skill for DB resiliency requests; it wires retry correctly including the transaction caveat.
category: EF Core / Database
version: 1.0.0
---

# Database Resiliency Setup

Make EF Core survive transient database failures (timeouts, failovers, throttling) with the built-in execution strategy — and do it correctly, including the manual-transaction caveat that trips people up.

## Input — point it at your project
Works on a target: a **file**, **project**, or **GitHub URL**. Find DB setup: `grep -rn "UseNpgsql\|UseSqlServer\|EnableRetryOnFailure\|BeginTransaction" --include=*.cs <target>`.

## Enable retry
```csharp
builder.Services.AddDbContext<AppDbContext>(o =>
    o.UseNpgsql(conn, npg => npg.EnableRetryOnFailure(
        maxRetryCount: 5,
        maxRetryDelay: TimeSpan.FromSeconds(10),
        errorCodesToAdd: null)));      // SQL Server: o.UseSqlServer(conn, s => s.EnableRetryOnFailure());
```
Also set a sensible command timeout: `npg.CommandTimeout(30)`.

## The transaction caveat (important)
With a retrying execution strategy, you **cannot** wrap multiple `SaveChanges`/commands in a manual `BeginTransaction` directly — the strategy needs to own the retry boundary. Use the execution strategy explicitly:
```csharp
var strategy = db.Database.CreateExecutionStrategy();
await strategy.ExecuteAsync(async () =>
{
    await using var tx = await db.Database.BeginTransactionAsync(ct);
    // ... multiple operations ...
    await db.SaveChangesAsync(ct);
    await tx.CommitAsync(ct);
});
```

## Idempotency
Retries can re-run a unit of work. Ensure operations are **idempotent** (or guarded by a unique key / outbox) so a retry after a partial success doesn't double-apply.

## Principles
- **Retry transient faults, not logic errors** — the execution strategy targets known transient codes; don't retry a constraint violation.
- **Own the retry boundary:** manual transactions must go inside `CreateExecutionStrategy().ExecuteAsync(...)`.
- **Cap retries and back off** — infinite retries turn a blip into an outage.
- Combine with app-level resiliency (Polly) for non-DB dependencies, not for the DB call EF already guards.

## How to use it & best prompts
"Add connection retry to my DbContext", "handle transient DB failures", "my multi-step transaction breaks with retry enabled — fix it", "make these DB writes resilient and idempotent". Pairs with `dbcontext-config-auditor`.
