---
name: ef-index-advisor
description: Recommend database indexes for EF Core entities based on how they're actually queried — filters, joins, ordering, and uniqueness — and generate the HasIndex configuration. Use this skill whenever the user wants index recommendations, asks why a query is slow due to a missing index, wants to add indexes, composite/covering/unique indexes, or asks "what indexes do I need / how do I index this". Always use this skill for indexing requests; it maps query patterns to concrete index definitions.
category: EF Core / Database
version: 1.0.0
---

# EF Core Index Advisor

Find the indexes a model is missing by looking at how its entities are queried, and emit the `HasIndex` configuration (plus column order and includes) — without over-indexing.

## Input — point it at your code
Works on a target: a **folder**, **project/solution**, or **GitHub URL**. Find query patterns and current indexes with:
```
grep -rn "\.Where(\|\.OrderBy\|\.Join(\|\.FirstOrDefault\|\.Single" --include=*.cs <target>
grep -rn "HasIndex\|IsUnique" --include=*.cs <target>
```

## How it reasons
For each frequently-queried entity, map predicates → index:
- **Equality filters** → index the filtered column(s). Multiple `AND` equality filters → **composite** index (most selective / most common first).
- **Range + equality** → equality columns first, range column last.
- **`OrderBy` after a filter** → include the sort column in the composite to avoid a sort.
- **Frequently selected extra columns** → consider a **covering** index (`.IncludeProperties(...)`).
- **Natural keys / dedupe** → **unique** index.
- **Foreign keys used in joins/filters** → index them (EF indexes FKs by default, but verify composite needs).

## Output
```csharp
modelBuilder.Entity<Order>()
    .HasIndex(o => new { o.CustomerId, o.Status, o.CreatedAt })   // filter CustomerId+Status, sort CreatedAt
    .HasDatabaseName("IX_Orders_Customer_Status_Created");

modelBuilder.Entity<Order>()
    .HasIndex(o => o.ExternalRef).IsUnique();
```

## Principles
- **Index for the queries you run, not every column.** Every index speeds reads but slows writes and costs storage.
- **Column order in composites matters** — equality before range, common filters first.
- **Confirm with the query plan.** Recommend `EXPLAIN ANALYZE` (Postgres) / actual execution plan (SQL Server) and `ToQueryString()` to verify the index is used.
- Watch for redundant indexes (a composite often makes a single-column one unnecessary).

## How to use it & best prompts
"What indexes does this project need?", "why is this query slow — missing index?", "design a composite index for filtering by customer+status and sorting by date". Pairs with `ef-core-query-optimizer`.
