# How to use — Hot-Path Profiler Assistant

**What it is.** Guides profiling a .NET app to find the real hot paths before you optimize — which tool to use (dotnet-trace/counters, PerfView, VS/Rider), how to capture, and how to read the results.

**When to reach for it.** Something's slow or CPU/memory is high and you don't yet know where the time goes.

**How to use it.** Point it at the project (path or GitHub URL) and describe the symptom; or paste a trace to interpret. Example prompts:
- "What's slow in my app — how do I find it?"
- "Capture a CPU trace for this endpoint."
- "Read this flame graph and tell me what to optimize."

**Get the best out of it.** Run it before any micro-optimization — it stops you tuning cold code. Capture under realistic load. It hands off to `benchmarkdotnet-setup` / `memory-allocation-analyzer` / `gc-pressure-auditor` once you know the hot spot.

**Won't do.** It guides profiling; it can't run a profiler on your machine for you — it tells you exactly what to run.
