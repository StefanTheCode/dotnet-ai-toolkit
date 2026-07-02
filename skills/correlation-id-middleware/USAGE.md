# How to use — Correlation-ID Middleware

**What it is.** Adds correlation-ID middleware so every request has an id flowing through all log lines, echoed to the caller, and propagated to downstream HTTP and messaging.

**When to reach for it.** Tracing a request through logs across services, especially before full distributed tracing is in place.

**How to use it.** Point it at your project (path or GitHub URL). Example prompts:
- "Add a correlation id to my requests and logs."
- "Propagate the correlation id to downstream services."
- "Echo a request id back in responses."

**Get the best out of it.** Let it reuse the W3C trace id when present so it complements (not competes with) tracing. Accept an inbound id from your gateway first. Pairs with `serilog-logging-setup`.

**Won't do.** It won't replace distributed tracing — it's the log-grep companion to it.
