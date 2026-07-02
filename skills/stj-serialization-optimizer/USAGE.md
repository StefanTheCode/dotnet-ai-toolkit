# How to use — System.Text.Json Optimizer

**What it is.** Optimizes System.Text.Json usage — source-generated contexts, reused options, streaming, and the small mistakes (new options per call) that cost a lot.

**When to reach for it.** Slow or allocation-heavy JSON, high-throughput APIs, or AOT/startup-sensitive apps.

**How to use it.** Point it at your project (path or GitHub URL). Example prompts:
- "Speed up my JSON serialization."
- "Add System.Text.Json source generation."
- "Why does serializing this allocate so much?"

**Get the best out of it.** The biggest wins are reusing `JsonSerializerOptions` and source generation — ask for those first. Stream large payloads. Verify with `benchmarkdotnet-setup` `[MemoryDiagnoser]`.

**Won't do.** It won't hand-roll converters the serializer already handles, and it won't micro-optimize a cold serialization path.
