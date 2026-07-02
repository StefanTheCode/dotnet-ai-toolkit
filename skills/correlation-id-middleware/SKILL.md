---
name: correlation-id-middleware
description: Add correlation-ID middleware to ASP.NET Core so every request has a traceable id flowing through logs, downstream HTTP calls, and messaging. Use this skill whenever the user wants a correlation id, request id, to trace a request across services in logs, propagate an id to downstream calls, or asks "how do I add a correlation id / track a request through logs". Always use this skill for correlation-id requests; it wires middleware, log enrichment, and outbound propagation.
category: Observability
version: 1.0.0
---

# Correlation-ID Middleware

Give every request a correlation id that shows up in every log line, propagates to downstream HTTP/messaging, and ties a user's request together across services — even where full tracing isn't set up.

## Input — point it at your project
Works on a target: a **project** or **GitHub URL**. Check for existing setup and the HTTP/logging stack: `grep -rn "CorrelationId\|X-Correlation\|HttpContext.TraceIdentifier\|UseSerilog\|HttpClient" --include=*.cs <target>`.

## Middleware
```csharp
public class CorrelationIdMiddleware(RequestDelegate next)
{
    public const string Header = "X-Correlation-ID";
    public async Task Invoke(HttpContext ctx)
    {
        var id = ctx.Request.Headers.TryGetValue(Header, out var v) && !string.IsNullOrEmpty(v)
            ? v.ToString() : Activity.Current?.TraceId.ToString() ?? Guid.NewGuid().ToString();

        ctx.Items[Header] = id;
        ctx.Response.Headers[Header] = id;                       // echo back to caller
        using (Serilog.Context.LogContext.PushProperty("CorrelationId", id))  // enrich all logs
            await next(ctx);
    }
}
app.UseMiddleware<CorrelationIdMiddleware>();   // early in the pipeline
```

## Propagate to downstream calls
A `DelegatingHandler` that copies the id onto outbound `HttpClient` requests:
```csharp
public class CorrelationHandler(IHttpContextAccessor http) : DelegatingHandler
{
    protected override Task<HttpResponseMessage> SendAsync(HttpRequestMessage req, CancellationToken ct)
    {
        if (http.HttpContext?.Items[CorrelationIdMiddleware.Header] is string id)
            req.Headers.TryAddWithoutValidation(CorrelationIdMiddleware.Header, id);
        return base.SendAsync(req, ct);
    }
}
// builder.Services.AddHttpClient<T>().AddHttpMessageHandler<CorrelationHandler>();
```
For messaging, put the id in message headers (publish) and restore it (consume).

## Principles
- **Reuse the W3C trace id when present** — don't invent a parallel id that competes with real tracing; correlation id complements it for log-grep-ability.
- Accept an inbound id (from a gateway/caller) before generating a new one.
- Enrich logs once via `LogContext` so every line carries it automatically.
- Echo it in the response so clients can report it in bug tickets.

## How to use it & best prompts
"Add a correlation id to my requests", "track a request through my logs", "propagate the correlation id to downstream services", "echo a request id in responses". Pairs with `serilog-logging-setup` and `distributed-tracing-diagnostics`.
