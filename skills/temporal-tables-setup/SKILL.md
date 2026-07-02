---
name: temporal-tables-setup
description: Set up auditing and history in EF Core using SQL Server temporal tables or an application-level audit trail. Use this skill whenever the user wants temporal tables, row history, audit trail, point-in-time queries, who-changed-what, soft history, or asks "how do I track changes / keep history of records / audit my entities". Always use this skill for EF Core auditing/history requests; it wires temporal tables or an interceptor-based audit log.
category: EF Core / Database
version: 1.0.0
---

# Temporal Tables / Auditing Setup

Add change history to entities — either SQL Server **temporal tables** (automatic, queryable point-in-time) or an **application-level audit log** (DB-agnostic, captures who/what) — depending on the database and requirements.

## Input — point it at your project
Works on a target: a **file**, **folder**, **project**, or **GitHub URL**. Detect provider with `grep -rn "UseSqlServer\|UseNpgsql" --include=*.cs <target>`. SQL Server → temporal tables are the cleanest; Postgres/others → interceptor-based audit log.

## Option A — SQL Server temporal tables
```csharp
modelBuilder.Entity<Order>().ToTable("Orders", t => t.IsTemporal(tt =>
{
    tt.HasPeriodStart("ValidFrom");
    tt.HasPeriodEnd("ValidTo");
    tt.UseHistoryTable("OrdersHistory");
}));
```
Query history / point-in-time:
```csharp
var asOf = await db.Orders.TemporalAsOf(timestamp).ToListAsync();
var changes = await db.Orders.TemporalAll().Where(o => o.Id == id)
    .OrderBy(o => EF.Property<DateTime>(o, "ValidFrom")).ToListAsync();
```

## Option B — application audit log (DB-agnostic, captures "who")
A `SaveChangesInterceptor` that writes an `AuditEntry` per changed entity:
```csharp
public class AuditInterceptor(ICurrentUser user) : SaveChangesInterceptor
{
    public override ValueTask<InterceptionResult<int>> SavingChangesAsync(
        DbContextEventData e, InterceptionResult<int> r, CancellationToken ct)
    {
        var ctx = e.Context!;
        foreach (var entry in ctx.ChangeTracker.Entries()
                     .Where(x => x.State is EntityState.Added or EntityState.Modified or EntityState.Deleted))
        {
            ctx.Add(new AuditEntry {
                Entity = entry.Metadata.GetTableName(), KeyValues = /* pk */,
                Action = entry.State.ToString(), Changes = /* changed props */,
                User = user.Id, At = DateTime.UtcNow });
        }
        return base.SavingChangesAsync(e, r, ct);
    }
}
```

## Choosing
- **Temporal tables:** automatic, perfect point-in-time, SQL Server only, doesn't record *who*.
- **Audit log:** any DB, records who/why, but you maintain it and it's queryable as data, not as table state.
- You can combine: temporal for state history + a light audit log for actor/reason.

## Principles
- Decide what you need: **point-in-time state** (temporal) vs **who-changed-what** (audit log).
- Don't log sensitive field *values* into audit rows without considering PII.
- History tables grow — plan retention/cleanup.

## How to use it & best prompts
"Add history to my Order entity", "track who changed what", "set up temporal tables for auditing", "point-in-time query for this record". (Stefan has a blog post on EF Core temporal tables — this complements it.)
