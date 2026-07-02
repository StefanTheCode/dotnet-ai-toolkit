# How to use — Serilog Structured Logging Setup

**What it is.** Wires Serilog in ASP.NET Core for structured JSON logs — enrichers, request logging, levels, sinks (Seq/Elastic/console) — correlated with traces.

**When to reach for it.** Moving off `Console.WriteLine`/default logging, or you need queryable structured logs.

**How to use it.** Point it at your project (path or GitHub URL). Example prompts:
- "Set up Serilog with JSON logs and a Seq sink."
- "Add request logging and trace correlation."
- "Fix my logging so it's actually structured."

**Get the best out of it.** Let it detect OTel and wire log/trace correlation. Ask for `UseSerilogRequestLogging` to cut default noise. Pairs with `opentelemetry-setup` and `correlation-id-middleware`.

**Won't do.** It won't decide your PII policy — it flags not to log secrets, you set what's allowed.
