---
name: specification-pattern-generator
description: Implement the Specification pattern for EF Core so query criteria (filters, includes, ordering, paging) become reusable, testable objects instead of duplicated LINQ. Use this skill whenever the user wants the specification pattern, reusable query criteria, to dedupe repeated Where/Include logic, a generic repository with specs, or asks "how do I reuse query logic / specification pattern in EF". Always use this skill for specification-pattern requests; it generates the base spec, evaluator, and an example.
category: EF Core / Database
version: 1.0.0
---

# Specification Pattern Generator

Turn repeated query criteria into reusable, unit-testable Specification objects, with an evaluator that applies them to an EF `IQueryable`.

## Input — point it at your code
Works on a target: a **folder**, **project**, or **GitHub URL**. Spot duplicated query logic to consolidate: `grep -rn "\.Where(\|\.Include(\|\.OrderBy" --include=*.cs <target>`.

## Base specification
```csharp
public abstract class Specification<T>
{
    public Expression<Func<T, bool>>? Criteria { get; protected set; }
    public List<Expression<Func<T, object>>> Includes { get; } = new();
    public Expression<Func<T, object>>? OrderBy { get; private set; }
    public int? Take { get; private set; }
    public int? Skip { get; private set; }
    public bool AsNoTracking { get; private set; } = true;

    protected void Where(Expression<Func<T, bool>> c) => Criteria = c;
    protected void Include(Expression<Func<T, object>> i) => Includes.Add(i);
    protected void Order(Expression<Func<T, object>> o) => OrderBy = o;
    protected void Page(int skip, int take){ Skip = skip; Take = take; }
    protected void Tracked() => AsNoTracking = false;
}
```

## Evaluator
```csharp
public static class SpecificationEvaluator
{
    public static IQueryable<T> Apply<T>(IQueryable<T> q, Specification<T> spec) where T : class
    {
        if (spec.AsNoTracking) q = q.AsNoTracking();
        if (spec.Criteria is not null) q = q.Where(spec.Criteria);
        q = spec.Includes.Aggregate(q, (cur, inc) => cur.Include(inc));
        if (spec.OrderBy is not null) q = q.OrderBy(spec.OrderBy);
        if (spec.Skip is not null) q = q.Skip(spec.Skip.Value);
        if (spec.Take is not null) q = q.Take(spec.Take.Value);
        return q;
    }
}
```

## Example spec + usage
```csharp
public class ActiveOrdersForCustomer : Specification<Order>
{
    public ActiveOrdersForCustomer(Guid customerId)
    {
        Where(o => o.CustomerId == customerId && o.Status == OrderStatus.Active);
        Include(o => o.Lines);
        Order(o => o.CreatedAt);
    }
}

var orders = await SpecificationEvaluator
    .Apply(db.Orders, new ActiveOrdersForCustomer(id)).ToListAsync(ct);
```

## Principles
- **Use specs to dedupe real repetition** — don't wrap a one-off query in ceremony.
- Specs are unit-testable (assert `Criteria` compiles to the right predicate) and composable.
- Keep `AsNoTracking` the default for read specs (opt into tracking explicitly).
- Don't let specs hide N+1 — they still need sensible includes/projection.

## How to use it & best prompts
"Add the specification pattern", "dedupe these repeated EF queries into specs", "make my query criteria reusable and testable". Pairs with `ef-core-query-optimizer`.
