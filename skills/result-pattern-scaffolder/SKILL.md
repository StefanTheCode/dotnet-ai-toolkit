---
name: result-pattern-scaffolder
description: Add a Result/Error pattern to a .NET codebase so expected failures stop using exceptions for control flow. Use this skill whenever the user wants a Result type, Result<T>, railway-oriented programming, OneOf, error handling without exceptions, mapping domain errors to HTTP responses, or asks "how do I return errors cleanly / stop throwing for validation". Always use this skill for Result-pattern requests; it generates the type, helpers, and endpoint mapping.
category: Architecture
version: 1.0.0
---

# Result Pattern Scaffolder

Introduce a Result/Error type so expected failures (validation, not-found, conflicts) are returned as values, not thrown — and map them cleanly to HTTP responses at the edge.

## Input — point it at your code
Works on a target: a **file**, **folder**, **project/solution**, or **GitHub URL** (`git clone --depth 1 <url>`). If a Result type or `OneOf` is already used, match it instead of adding a second. Detect with: `grep -rn "OneOf\|Result<\|IResult\|throw new .*Exception" --include=*.cs <target>`.

## The type
```csharp
public readonly record struct Error(string Code, string Message)
{
    public static readonly Error None = new(string.Empty, string.Empty);
    public static Error NotFound(string msg) => new("not_found", msg);
    public static Error Validation(string msg) => new("validation", msg);
    public static Error Conflict(string msg) => new("conflict", msg);
}

public readonly struct Result<T>
{
    public bool IsSuccess { get; }
    public T? Value { get; }
    public Error Error { get; }
    private Result(T v){ IsSuccess=true; Value=v; Error=Error.None; }
    private Result(Error e){ IsSuccess=false; Value=default; Error=e; }
    public static Result<T> Success(T v) => new(v);
    public static Result<T> Failure(Error e) => new(e);
    public static implicit operator Result<T>(T v) => Success(v);
    public static implicit operator Result<T>(Error e) => Failure(e);
}
```

## Using it in a handler
```csharp
public async Task<Result<OrderDto>> Handle(GetOrder q, CancellationToken ct)
{
    var order = await _db.Orders.AsNoTracking()
        .FirstOrDefaultAsync(o => o.Id == q.Id, ct);
    if (order is null) return Error.NotFound($"Order {q.Id} not found");
    return _mapper.ToDto(order);
}
```

## Mapping to HTTP at the edge (one place)
```csharp
public static IResult ToHttp<T>(this Result<T> r) => r.IsSuccess
    ? Results.Ok(r.Value)
    : r.Error.Code switch
    {
        "not_found"  => Results.NotFound(r.Error.Message),
        "validation" => Results.BadRequest(r.Error.Message),
        "conflict"   => Results.Conflict(r.Error.Message),
        _            => Results.Problem(r.Error.Message)
    };
```

## Principles
- **Exceptions are for the exceptional** (bugs, infra failures) — not for "user typed a bad value".
- Map errors to HTTP in exactly one place, not in every handler.
- Don't swallow real exceptions into a Result — only expected, domain-level outcomes become Results.
- If the codebase already uses `OneOf<T, Error>`, extend that rather than introducing a competing type.

## How to use it & best prompts
"Add a Result pattern to this project", "stop throwing for validation in these handlers", "map domain errors to HTTP in one place". Pairs with `cqrs-mediatr-setup` (handler return types) and the `dotnet-code-reviewer` agent.
