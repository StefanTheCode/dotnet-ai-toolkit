---
name: ddd-aggregate-generator
description: Generate Domain-Driven Design building blocks in C# — aggregates, entities, value objects, domain events, and invariant-protecting factory methods. Use this skill whenever the user wants a DDD aggregate, value object, domain event, rich domain model, bounded context structure, encapsulated invariants, or asks "how do I model this domain / build an aggregate root". Always use this skill for DDD modeling requests; it generates a rich, encapsulated model rather than anemic setters.
category: Architecture
version: 1.0.0
---

# DDD Aggregate Generator

Model a domain with proper DDD building blocks: an aggregate root that protects its invariants, value objects for concepts, and domain events — no anemic bags of public setters.

## Input — point it at your project
Works on a target: a **file**, **folder**, **project/solution**, or **GitHub URL**. If a domain layer exists, match its base types (`Entity`, `AggregateRoot`, `ValueObject`). Detect with: `grep -rn "AggregateRoot\|ValueObject\|: Entity\|IDomainEvent" --include=*.cs <target>`.

## Aggregate root (invariants protected)
```csharp
public sealed class Order : AggregateRoot
{
    private readonly List<OrderLine> _lines = new();
    public IReadOnlyList<OrderLine> Lines => _lines.AsReadOnly();
    public OrderStatus Status { get; private set; }
    public Money Total { get; private set; }

    private Order() { }                              // EF

    public static Result<Order> Create(CustomerId customer, IEnumerable<OrderLine> lines)
    {
        var list = lines.ToList();
        if (list.Count == 0) return Error.Validation("Order needs at least one line");
        var order = new Order { Status = OrderStatus.Pending };
        order._lines.AddRange(list);
        order.Total = Money.Sum(list.Select(l => l.LineTotal));
        order.Raise(new OrderCreated(order.Id));     // domain event
        return order;
    }

    public Result Confirm()
    {
        if (Status != OrderStatus.Pending) return Error.Conflict("Only pending orders can be confirmed");
        Status = OrderStatus.Confirmed;
        Raise(new OrderConfirmed(Id));
        return Result.Success();
    }
}
```

## Value object
```csharp
public sealed record Money(decimal Amount, string Currency)
{
    public static Money Sum(IEnumerable<Money> items) { /* guard same currency */ }
    public static implicit operator decimal(Money m) => m.Amount;
}
```
Records give value equality for free; add guards in the constructor.

## Domain events
Raised inside the aggregate (`Raise(...)`), collected on the base type, dispatched after `SaveChanges` (outbox or MediatR). Keep events immutable and past-tense.

## Principles
- **Encapsulate invariants in the aggregate** — no public setters that let outsiders break rules. Mutate through intention-revealing methods.
- **The aggregate is the consistency boundary** — reference other aggregates by id, not by navigation.
- **Value objects for concepts** (Money, Email, DateRange), not primitives everywhere.
- Don't model anemically (logic in services, data in entities) — but also don't over-DDD a simple CRUD table.

## How to use it & best prompts
"Model an Order aggregate with these rules", "turn this anemic entity into a rich aggregate", "extract Money/Email value objects", "add domain events for order lifecycle".
