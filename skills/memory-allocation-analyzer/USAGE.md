# How to use — Memory Allocation Analyzer

**What it is.** Finds avoidable heap allocations on hot paths (LINQ in loops, boxing, closures, string concat, params, async state machines) and rewrites them to cut GC pressure.

**When to reach for it.** A hot path allocates heavily, GC pressure is high, or you're tuning a throughput-critical method.

**How to use it.** Point it at the code (path or GitHub URL). Example prompts:
- "Why does this handler allocate so much?"
- "Reduce GC pressure in this hot loop."
- "Find boxing and closures in this file."

**Get the best out of it.** Tell it which paths are actually hot — it deliberately leaves cold code's LINQ alone for readability. Verify before/after with `benchmarkdotnet-setup` `[MemoryDiagnoser]`.

**Won't do.** It won't uglify cold code to save nanoseconds, and it won't guess at hotness — confirm with a profiler (`hotpath-profiler-assistant`).
