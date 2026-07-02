---
name: sql-linq-converter
description: Convert raw SQL to EF Core LINQ and LINQ to tuned SQL, both directions, preserving semantics. Use this skill whenever the user wants to turn a SQL query into LINQ, convert LINQ to SQL, migrate stored procedures or ADO.NET queries to EF Core, or asks "how do I write this SQL in EF / what SQL does this LINQ produce / convert this query". Always use this skill for SQL<->LINQ conversion; it preserves behavior and flags anything that won't translate.
category: EF Core / Database
version: 1.0.0
---

# SQL ⇄ LINQ Converter

Translate between raw SQL and EF Core LINQ in either direction, keeping the result set identical and flagging constructs that don't translate cleanly.

## Input — point it at your code or paste the query
Works on a target (a **file**, **folder**, **project**, **GitHub URL**) or a pasted query. For SQL→LINQ, you may need the entity definitions — find them with `grep -rn "DbSet<\|class .* :" --include=*.cs <target>`.

## SQL → LINQ
```sql
SELECT c.Name, COUNT(o.Id) AS Orders
FROM Customers c LEFT JOIN Orders o ON o.CustomerId = c.Id
WHERE c.IsActive = 1
GROUP BY c.Name HAVING COUNT(o.Id) > 5
ORDER BY Orders DESC;
```
```csharp
var rows = await db.Customers
    .Where(c => c.IsActive)
    .Select(c => new { c.Name, Orders = c.Orders.Count() })
    .Where(x => x.Orders > 5)
    .OrderByDescending(x => x.Orders)
    .ToListAsync(ct);
```

## LINQ → SQL
Show the translation EF will produce (or use `query.ToQueryString()`), so the user can read the actual SQL and spot N+1 or client evaluation.

## Won't-translate flags
- Window functions / CTEs / `PIVOT` — often need `FromSql` or a mapped view; say so.
- Vendor-specific functions — map to `EF.Functions.*` where available, else raw SQL.
- Set operations and recursive queries — note EF support limits.
- Stored procedures — wrap with `FromSql`/`SqlQuery`, or port the logic to LINQ if simple.

## Principles
- **Preserve semantics first** (joins, null handling, grouping), performance second.
- **Parameterize** — never concatenate user input; use interpolated `FromSql($"...{p}")` (safe) when raw SQL is needed.
- When a query genuinely belongs in SQL (heavy analytics), say so instead of forcing awkward LINQ.
- Always offer to show the generated SQL so the translation is verifiable.

## How to use it & best prompts
"Convert this SQL to EF Core LINQ", "what SQL does this LINQ generate", "port this stored procedure to LINQ", "rewrite this ADO.NET query with EF". Pairs with `ef-core-query-optimizer`.
