# How to use — Distributed Tracing Diagnostics

**What it is.** Finds where a distributed trace breaks (missing spans, dropped context across HTTP/messaging, unregistered ActivitySource, sampling) and fixes propagation so a request is one connected trace.

**When to reach for it.** Traces that don't link across services, missing spans, or orphaned traces.

**How to use it.** Point it at the service(s) (path or GitHub URL). Example prompts:
- "My traces don't connect across services — why?"
- "Trace context is lost over NATS/RabbitMQ."
- "Spans from my own code don't show up."

**Get the best out of it.** Give it all the services involved if you can — breaks happen at boundaries. For messaging, ask for the inject/extract code. Pairs with `opentelemetry-setup`.

**Won't do.** It can't read your live backend — confirm the fix in Jaeger/Tempo after applying.
