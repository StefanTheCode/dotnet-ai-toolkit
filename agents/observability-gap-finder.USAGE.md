# How to use — Observability Gap Finder (agent)

**What it is.** An agent that audits a .NET service across traces, metrics, logs, health checks, and correlation, then gives a prioritized plan to make it properly observable.

**When to reach for it.** "Can I actually debug this in production", a pre-prod readiness check, or finding what telemetry is missing.

**How to use it.** Point it at the service repo (path or GitHub URL). Example prompts:
- "Review this service's observability."
- "Can we operate and debug this in production?"
- "What telemetry am I missing?"

**Get the best out of it.** Give it the whole service (and related services for cross-service tracing gaps). It hands fixes to `opentelemetry-setup`, `serilog-logging-setup`, `healthchecks-setup`, `correlation-id-middleware`, and `metrics-dashboard-generator`.

**Won't do.** It reviews wiring, not live dashboards — confirm signals flow in your backend after applying fixes.
