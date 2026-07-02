# How to use — OpenTelemetry Setup

**What it is.** A skill that wires up OpenTelemetry observability — traces, metrics, and logs — in an ASP.NET Core or .NET worker app, exported to whatever backend you use (OTLP collector, Jaeger, Prometheus/Grafana, Azure Monitor). It generates production-shaped `Program.cs` wiring, custom span/metric examples, and a ready-to-run docker-compose stack.

**When to reach for it.** Adding observability to a service, setting up distributed tracing across microservices, wiring metrics to Prometheus, or troubleshooting "I can't see what my app is doing in production".

**How to use it.** Point it at your project (a path, solution, or GitHub URL) and it inspects the existing `Program.cs`/OTel wiring first — extending what's there rather than starting over. Then tell it three things: **app type** (API / worker / library), **backend** (OTLP, Jaeger, Prometheus, Azure Monitor, or "console for now"), and **.NET version**. Example prompts:
- "Add OpenTelemetry tracing and metrics to my ASP.NET Core API, exporting to Jaeger + Prometheus."
- "Set up OTel logs and traces for my worker service, Azure Monitor backend."
- "I want a local observability stack to test traces — give me the compose file too."

**How to get the best out of it.**
- **Name your service and domain.** It generates custom spans/metrics around *your* operations (e.g. `PlaceOrder`, `orders.placed`) instead of generic examples if you tell it what the app does.
- **Ask for the local stack** (`references/otel-collector-compose.md`) when you want to see traces in Jaeger before touching production.
- **Mention scale.** It defaults to safe production settings (sampling, resource attributes) — tell it your traffic and it tunes sampling ratio.
- **Pair it with the architecture/observability work** — it'll add correlation IDs and cross-service trace propagation if you ask.

**What it won't do.** It won't deploy or configure your real backend credentials — it shows you the wiring and env vars; you set the secrets. Sampling ratios are starting points, not tuned to your traffic.
