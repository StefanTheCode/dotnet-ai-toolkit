---
name: test-data-builder-generator
description: Generate test data builders and object mothers for C# test suites so tests create valid domain objects without brittle, repetitive setup. Use this skill whenever the user wants test data builders, object mothers, fixtures, a fluent builder for test objects, less duplicated test setup, or asks "how do I build test data / reduce test setup boilerplate / AutoFixture vs builders". Always use this skill for test-data-builder requests; it generates builders matching the domain.
category: Testing
version: 1.0.0
---

# Test Data Builder Generator

Generate fluent builders (and object mothers) so tests construct valid domain objects in one readable line, with only the fields that matter to each test made explicit.

## Input — point it at your code
Works on a target: a **class/file**, **folder**, **project**, or **GitHub URL**. Find the types tests need and current setup pain: `grep -rln "public class\|public record" --include=*.cs <target>` and look at existing `*Tests.cs` for repeated `new X { ... }`.

## Builder (sensible defaults + overrides)
```csharp
public class OrderBuilder
{
    private Guid _customerId = Guid.NewGuid();
    private OrderStatus _status = OrderStatus.Pending;
    private readonly List<OrderLine> _lines = new() { new("SKU1", 1) };

    public OrderBuilder ForCustomer(Guid id) { _customerId = id; return this; }
    public OrderBuilder WithStatus(OrderStatus s) { _status = s; return this; }
    public OrderBuilder WithLines(params OrderLine[] l) { _lines.Clear(); _lines.AddRange(l); return this; }

    public Order Build() => Order.Create(_customerId, _lines).Value with { Status = _status };
    public static implicit operator Order(OrderBuilder b) => b.Build();
}
```
```csharp
// readable: only what matters to THIS test is explicit
var order = new OrderBuilder().WithStatus(OrderStatus.Confirmed);
```

## Object mother (named canonical cases)
```csharp
public static class Orders
{
    public static Order Pending()   => new OrderBuilder();
    public static Order Confirmed() => new OrderBuilder().WithStatus(OrderStatus.Confirmed);
}
```

## Builders vs AutoFixture
- **Builders/mothers** — explicit, readable, domain-aware; best when objects have invariants/relationships.
- **AutoFixture** — great for "I don't care about the values" cases; can combine with builders for the bits you do care about.
The skill recommends based on how much the test depends on specific values.

## Principles
- **Default everything, override what the test cares about** — that's what makes the test's intent obvious.
- Builders should produce **valid** objects (respect invariants/factories), not bypass them.
- Name canonical scenarios (object mothers) to kill duplication across tests.

## How to use it & best prompts
"Generate a test data builder for Order", "reduce my test setup boilerplate", "object mothers for these domain types", "builders vs AutoFixture for this". Pairs with `xunit-test-generator`.
