---
name: cqrs-mediatr-setup
description: Set up CQRS with MediatR (or a thin custom dispatcher) in an ASP.NET Core project — commands, queries, handlers, and pipeline behaviors for validation, logging, and transactions. Use this skill whenever the user wants CQRS, MediatR, command/query handlers, a request pipeline, MediatR behaviors, or asks "how do I add CQRS / wire up MediatR / separate reads and writes". Always use this skill for CQRS/MediatR requests; it wires a working pipeline rather than a snippet.
category: Architecture
version: 1.0.0
---

# CQRS + MediatR Setup

Wire up CQRS in an ASP.NET Core app: command/query separation, handlers, and the cross-cutting pipeline behaviors that make CQRS pay off (validation, logging, transactions).

## Input — point it at your project
Works on a target, not pasted snippets: a **file**, **folder**, **project/solution**, or **GitHub URL** (`git clone --depth 1 <url>` then scan). If the project exists, read one feature first and match its conventions. If MediatR is being phased out in your stack, offer the thin custom-dispatcher variant.

## Packages
```
dotnet add package MediatR
dotnet add package FluentValidation.DependencyInjectionExtensions
```
```csharp
builder.Services.AddMediatR(c => c.RegisterServicesFromAssembly(typeof(Program).Assembly));
builder.Services.AddValidatorsFromAssembly(typeof(Program).Assembly);
```

## Command / query + handler
```csharp
public record CreateOrderCommand(Guid CustomerId, List<OrderLine> Lines) : IRequest<Result<Guid>>;

public class CreateOrderHandler(IOrderRepository repo, IUnitOfWork uow)
    : IRequestHandler<CreateOrderCommand, Result<Guid>>
{
    public async Task<Result<Guid>> Handle(CreateOrderCommand cmd, CancellationToken ct)
    {
        var order = Order.Create(cmd.CustomerId, cmd.Lines);
        await repo.AddAsync(order, ct);
        await uow.SaveChangesAsync(ct);
        return Result.Success(order.Id);
    }
}
```
Queries are the same shape with `IRequest<TDto>` and a read-only handler (`AsNoTracking`, projection).

## Pipeline behaviors (where CQRS earns its keep)
```csharp
public class ValidationBehavior<TReq, TRes>(IEnumerable<IValidator<TReq>> validators)
    : IPipelineBehavior<TReq, TRes> where TReq : notnull
{
    public async Task<TRes> Handle(TReq req, RequestHandlerDelegate<TRes> next, CancellationToken ct)
    {
        foreach (var v in validators)
        {
            var r = await v.ValidateAsync(req, ct);
            if (!r.IsValid) throw new ValidationException(r.Errors); // or map to a Result
        }
        return await next();
    }
}
builder.Services.AddTransient(typeof(IPipelineBehavior<,>), typeof(ValidationBehavior<,>));
```
Add `LoggingBehavior` and a `TransactionBehavior` (open transaction around commands only) the same way.

## Principles
- **Commands change state and return little; queries read and never mutate.** Keep them in separate folders.
- **Behaviors over base classes.** Cross-cutting concerns belong in the pipeline, not copy-pasted into handlers.
- **Don't add MediatR to a tiny CRUD app** — for a few endpoints a direct handler call is simpler. Say so.
- One command/query per file; name by intent (`CreateOrder`, `GetOrderById`).

## How to use it & best prompts
Point it at a project and say what you want: "add CQRS with MediatR + validation/transaction behaviors", "convert these controllers to commands/queries", "set up a query pipeline with logging". For new feature slices, pair with `clean-architecture-scaffolder`.
