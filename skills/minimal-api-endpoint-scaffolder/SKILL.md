---
name: minimal-api-endpoint-scaffolder
description: Scaffold ASP.NET Core Minimal API endpoints using the REPR pattern (Request-Endpoint-Response) with validation, typed results, and route grouping. Use this skill whenever the user wants to add a minimal API endpoint, convert controllers to minimal APIs, set up endpoint groups, TypedResults, route handlers, or asks "how do I add an endpoint / structure minimal APIs". Always use this skill for minimal-API requests; it generates a complete, validated endpoint that matches the project's conventions.
category: Architecture
version: 1.0.0
---

# Minimal API Endpoint Scaffolder

Generate clean ASP.NET Core Minimal API endpoints — one feature per endpoint (REPR), validated, with `TypedResults`, grouped routes, and OpenAPI metadata.

## Input — point it at your project
Works on a target: a **file**, **folder**, **project/solution**, or **GitHub URL**. Read one existing endpoint and match its style (Carter module? plain `MapGroup`? MediatR dispatch?). Detect with: `grep -rn "MapGet\|MapPost\|MapGroup\|ICarterModule\|: ControllerBase" --include=*.cs <target>`.

## Endpoint (REPR + TypedResults)
```csharp
public static class CreateOrderEndpoint
{
    public static RouteGroupBuilder MapCreateOrder(this RouteGroupBuilder group)
    {
        group.MapPost("/", Handle)
             .WithName("CreateOrder")
             .WithSummary("Creates an order")
             .Produces<OrderResponse>(StatusCodes.Status201Created)
             .ProducesValidationProblem();
        return group;
    }

    public record Request(Guid CustomerId, List<OrderLine> Lines);
    public record OrderResponse(Guid Id);

    private static async Task<Results<Created<OrderResponse>, ValidationProblem>> Handle(
        Request req, IValidator<Request> validator, ISender sender, CancellationToken ct)
    {
        var v = await validator.ValidateAsync(req, ct);
        if (!v.IsValid) return TypedResults.ValidationProblem(v.ToDictionary());

        var id = await sender.Send(new CreateOrderCommand(req.CustomerId, req.Lines), ct);
        return TypedResults.Created($"/orders/{id}", new OrderResponse(id));
    }
}
```

## Route grouping (Program.cs)
```csharp
var orders = app.MapGroup("/orders").WithTags("Orders").RequireAuthorization();
orders.MapCreateOrder();
orders.MapGetOrderById();
```

## Principles
- **One endpoint per feature** — don't pile unrelated routes into one class.
- **`TypedResults` over `Results`** — they give you compile-time response types and accurate OpenAPI.
- **Validate at the edge**, return `ValidationProblem` (RFC 7807), keep business rules in the handler/domain.
- **Group routes** for shared prefix, tags, and auth instead of repeating per endpoint.
- Don't hand-roll what `MapGroup` + filters already do.

## How to use it & best prompts
"Add a minimal API endpoint for creating X", "convert this controller to minimal APIs", "group these routes under /orders with auth". Matches your existing endpoint style automatically.
