---
name: distributed-tracing-diagnostics
description: Diagnose and fix broken distributed traces in .NET — missing spans, broken context propagation across HTTP/messaging, orphaned traces, and missing instrumentation. Use this skill whenever the user has traces that don't connect across services, missing spans, broken trace context, propagation issues over HTTP/queues, or asks "why is my trace broken / spans not linking / context not propagating". Always use this skill for tracing diagnostics; it finds where the trace chain breaks.
category: Observability
version: 1.0.0
---

# Distributed Tracing Diagnostics

Find where a distributed trace breaks — the missing span, the dropped context, the uninstrumented hop — and fix propagation so one request shows as one connected trace across services.

## Input — point it at your project(s)
Works on a target: a **project**, multiple services, or a **GitHub URL**. Inspect instrumentation: `grep -rn "ActivitySource\|AddSource\|StartActivity\|Propagator\|AddOpenTelemetry\|WithTracing" --include=*.cs <target>`.

## Where traces break (checklist)
1. **No tracing registered** — `.WithTracing(...)` missing entirely (metrics-only). Add it (`opentelemetry-setup`).
2. **Custom `ActivitySource` not registered** — you `StartActivity` but didn't `AddSource("name")`, so spans are dropped.
3. **HTTP context not propagated** — usually automatic with `AddHttpClientInstrumentation`, but a hand-rolled `HttpClient` or a non-standard header strips `traceparent`. Ensure W3C `TraceContext` propagation.
4. **Messaging hops lose context** — queues/EventBus (NATS, RabbitMQ, Kafka) don't auto-propagate. **Inject** trace context into message headers on publish, **extract** on consume:
```csharp
// publish
var props = new Dictionary<string,string>();
Propagators.DefaultTextMapPropagator.Inject(
    new PropagationContext(activity.Context, Baggage.Current), props, (c,k,v) => c[k]=v);
// consume
var parent = Propagators.DefaultTextMapPropagator.Extract(default, props, (c,k) => /* get header */);
using var act = source.StartActivity("consume", ActivityKind.Consumer, parent.ActivityContext);
```
5. **Sampling drops the parent** — a parent-based sampler with a low ratio can drop spans; verify sampling config.
6. **Async context lost** — `Activity.Current` not flowing across a custom thread/`Task.Run` boundary.

## Principles
- A broken trace is almost always a **propagation** problem at a boundary (HTTP, queue, thread).
- Register every custom `ActivitySource`; an unregistered source is invisible.
- Messaging needs **manual inject/extract** — it won't propagate itself.
- Verify in your backend (Jaeger/Tempo) that parent/child span ids line up.

## How to use it & best prompts
"My traces don't connect across services", "spans from my code don't show up", "trace context is lost over NATS/RabbitMQ", "why is this trace orphaned". Pairs with `opentelemetry-setup`.
