# How to use — Caching Strategy Setup

**What it is.** Adds caching at the right layer (IMemoryCache / distributed / HybridCache / output caching) with the parts people forget: keys, expiration, stampede protection, and invalidation.

**When to reach for it.** Expensive repeated reads, scaling reads across nodes, or fixing a cache stampede.

**How to use it.** Point it at your project (path or GitHub URL). Example prompts:
- "Cache this expensive query."
- "Use HybridCache here with a 10-minute expiry."
- "I get a cache stampede on expiry — fix it."

**Get the best out of it.** Tell it single-node vs multi-node (drives memory vs distributed vs hybrid) and how stale data can be. Always ask for the invalidation plan — that's the hard part. Verify hit rate after.

**Won't do.** It won't cache user-specific/secret data into a shared cache without scoped keys — and it'll push back on caching things that shouldn't be cached.
