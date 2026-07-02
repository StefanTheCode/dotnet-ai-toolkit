---
name: authorization-policy-generator
description: Design ASP.NET Core authorization — policy-based, role-based, and claims/requirement-based with custom authorization handlers and resource-based checks. Use this skill whenever the user wants authorization policies, role/claim checks, custom AuthorizationHandler, resource-based authorization, permission checks, or asks "how do I authorize this / policy vs role / restrict this endpoint to X". Always use this skill for authorization requests; it generates policies, handlers, and endpoint wiring.
category: Security
version: 1.0.0
---

# Authorization Policy Generator

Design ASP.NET Core authorization the right way: named policies over scattered role strings, custom requirements/handlers for real rules, and resource-based checks for "can this user act on this object".

## Input — point it at your project
Works on a target: a **project** or **GitHub URL**. Inspect current setup: `grep -rn "AddAuthorization\|RequireRole\|RequireClaim\|\[Authorize\|RequireAuthorization\|IAuthorizationHandler" --include=*.cs <target>`.

## Named policies (not role strings everywhere)
```csharp
builder.Services.AddAuthorizationBuilder()
    .AddPolicy("CanManageOrders", p => p.RequireClaim("permission", "orders:manage"))
    .AddPolicy("AdminOnly", p => p.RequireRole("Admin"))
    .AddPolicy("Over18", p => p.AddRequirements(new MinimumAgeRequirement(18)));
```

## Custom requirement + handler (real business rules)
```csharp
public record MinimumAgeRequirement(int Age) : IAuthorizationRequirement;

public class MinimumAgeHandler : AuthorizationHandler<MinimumAgeRequirement>
{
    protected override Task HandleRequirementAsync(AuthorizationHandlerContext ctx, MinimumAgeRequirement req)
    {
        var dob = ctx.User.FindFirst("dob")?.Value;
        if (DateTime.TryParse(dob, out var d) && d.AddYears(req.Age) <= DateTime.Today)
            ctx.Succeed(req);
        return Task.CompletedTask;
    }
}
builder.Services.AddSingleton<IAuthorizationHandler, MinimumAgeHandler>();
```

## Resource-based (can THIS user act on THIS object)
```csharp
var auth = await _authService.AuthorizeAsync(User, order, "OrderOwnerPolicy");
if (!auth.Succeeded) return Results.Forbid();
```
This is how you stop IDOR — ownership checked against the actual resource, not just a role.

## Wiring endpoints
```csharp
app.MapPost("/orders", ...).RequireAuthorization("CanManageOrders");
group.RequireAuthorization("AdminOnly");
// Consider a FallbackPolicy so endpoints are authenticated by default.
```

## Principles
- **Policies over inline role strings** — one named policy, reused, easy to change.
- **Requirements/handlers for business rules**, not giant `if (User.IsInRole(...))` blocks.
- **Resource-based authorization** prevents IDOR — check ownership, not just role.
- Consider a **fallback policy** so new endpoints aren't accidentally anonymous.

## How to use it & best prompts
"Design authorization policies for my API", "restrict this endpoint to managers", "add a custom authorization rule", "stop users from accessing others' records" (resource-based). Pairs with `jwt-auth-setup` and the `aspnetcore-security-auditor` agent.
