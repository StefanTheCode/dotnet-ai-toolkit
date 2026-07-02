---
name: hotpath-profiler-assistant
description: Guide profiling a .NET app to find the real hot paths before optimizing — which tool to use (dotnet-trace, dotnet-counters, PerfView, VS Profiler), how to capture, and how to read the results. Use this skill whenever the user wants to profile, find bottlenecks, figure out what's slow, capture a trace, read a flame graph, or asks "what's slow in my app / how do I profile this / where's the bottleneck". Always use this skill before micro-optimizing; it finds where time actually goes.
category: Performance
version: 1.0.0
---

# Hot-Path Profiler Assistant

Find where time and allocations actually go before optimizing anything — pick the right tool, capture a useful trace, and interpret it. Stops premature optimization of cold code.

## Input — point it at your project / describe the symptom
Works on a target (a **project** or **GitHub URL**) for context, plus the symptom (slow endpoint, high CPU, latency spikes). If they have a trace already, go to interpretation.

## Pick the tool
- **`dotnet-counters`** — live, low-overhead metrics (CPU, GC, thread pool, requests). Start here to see *what kind* of problem it is.
- **`dotnet-trace`** — sampling/event trace of CPU time; cross-platform; open in Speedscope/PerfView. Best for "where is CPU spent".
- **`dotnet-gcdump` / `dotnet-dump`** — heap snapshots for memory leaks/retention.
- **PerfView** (Windows) — deep ETW analysis, allocation tick, GC.
- **VS / Rider profiler** — IDE-integrated, friendly flame graphs.

## Capture (examples)
```
dotnet-counters monitor -p <pid>
dotnet-trace collect -p <pid> --duration 00:00:30        # then open the .nettrace in speedscope.app
dotnet-gcdump collect -p <pid>                           # heap snapshot
```
For an endpoint, drive load while capturing (e.g. `bombardier`/`k6`) so the hot path is exercised.

## Reading it
- **Flame graph width = time** — widest frames are where you optimize. Ignore narrow ones.
- Separate **CPU-bound** (busy frames) from **wait** (async/IO) — different fixes.
- Check **allocations** alongside CPU — GC time can masquerade as slowness.
- Confirm the suspected method is actually hot before touching it.

## Principles
- **Profile first, optimize second.** Most "obvious" bottlenecks are wrong guesses.
- Measure under realistic load, not a single cold request.
- Optimize the widest frame, re-measure, repeat — don't shotgun.

## How to use it & best prompts
"What's slow in my app", "where's the bottleneck in this endpoint", "how do I capture a CPU trace", "read this flame graph / trace". Feeds into `benchmarkdotnet-setup`, `memory-allocation-analyzer`, `gc-pressure-auditor`.
