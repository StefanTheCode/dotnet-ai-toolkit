---
name: dbcontext-config-auditor
description: Audit EF Core DbContext and entity configuration for correctness and performance pitfalls — tracking defaults, lazy loading, missing configuration, connection setup, sensitive data logging, and pooling. Use this skill whenever the user shares a DbContext, OnModelCreating, IEntityTypeConfiguration, AddDbContext setup, or asks "review my DbContext / is my EF configuration right / why is EF behaving oddly". Always use this skill for DbContext configuration reviews; it runs a fixed checklist.
category: EF Core / Database
version: 1.0.0
---

# DbContext Configuration Auditor

Review how the `DbContext` and entity configurations are set up — the settings that quietly cause perf problems, data leaks, or surprising behavior.

## Input — point it at your project
Works on a target: a **file**, **folder**, **project**, or **GitHub URL**. Find config with:
```
grep -rln "DbContext\|OnModelCreating\|IEntityTypeConfiguration\|AddDbContext\|UseNpgsql\|UseSqlServer" --include=*.cs <target>
```

## Checklist
1. 🔴 **`EnableSensitiveDataLogging()` unconditional** — logs parameter values (PII/secrets). Gate behind `IsDevelopment()`.
2. 🟡 **Lazy loading enabled** (`UseLazyLoadingProxies`) — silent N+1. Prefer explicit `Include`/projection.
3. 🟡 **Tracking default for a read-heavy context** — consider `QueryTrackingBehavior.NoTracking` as default, opt into tracking for writes.
4. 🟡 **No pooling for a high-traffic API** — `AddDbContextPool` reduces allocation cost (mind no request state in the context).
5. 🟡 **Configuration in `OnModelCreating` vs `IEntityTypeConfiguration`** — large `OnModelCreating` should be split into per-entity config classes (`ApplyConfigurationsFromAssembly`).
6. 🟡 **Missing/over-broad conventions** — string lengths, decimal precision, required/optional, delete behavior (cascade surprises).
7. 🟢 **Connection resiliency not enabled** — `EnableRetryOnFailure` for transient faults (see `db-resiliency-setup`).
8. 🟢 **DbContext registered with wrong lifetime** — scoped is correct; singleton/transient cause bugs.

## Output
```
## Findings (ranked)
🔴 [Sensitive logging] <file> — <risk> → <fix>
## Suggested configuration
<corrected AddDbContext + a sample IEntityTypeConfiguration>
```

## Principles
- Defaults you didn't set are still decisions — make tracking, lazy loading, and logging explicit.
- Entity config belongs in `IEntityTypeConfiguration<T>` classes once `OnModelCreating` grows.
- Never leak parameter values into logs in production.
- Cascade-delete behavior should be intentional, not whatever EF inferred.

## How to use it & best prompts
"Review my DbContext setup", "is my EF configuration sane", "why is EF doing extra queries", "split OnModelCreating into per-entity config". Pairs with `ef-core-query-optimizer` and `db-resiliency-setup`.
