---
name: caching-strategy-setup
description: Add the right caching to a .NET app — IMemoryCache, distributed cache, or HybridCache (.NET 9+) — with correct keys, expiration, stampede protection, and invalidation. Use this skill whenever the user wants caching, to cache a query/response, IMemoryCache, Redis/distributed cache, HybridCache, output caching, or asks "how do I cache this / add caching / avoid cache stampede". Always use this skill for caching requests; it picks the right layer and wires expiration + invalidation correctly.
category: Performance
version: 1.0.0
---

# Caching Strategy Setup

Add caching at the right layer with the parts people forget: sane keys, expiration policy, stampede protection, and an invalidation plan.

## Input — point it at your project
Works on a target: a **file**, **folder**, **project**, or **GitHub URL**. Find hot reads to cache and existing cache usage: `grep -rn "IMemoryCache\|IDistributedCache\|HybridCache\|GetOrCreate" --include=*.cs <target>`.

## Pick the layer
- **`IMemoryCache`** — single instance, fastest, not shared across nodes. Good for per-node hot data.
- **`IDistributedCache` (Redis)** — shared across instances, survives restarts, network hop. Good for multi-node.
- **`HybridCache` (.NET 9+)** — combines L1 (memory) + L2 (distributed) with **built-in stampede protection**; prefer it when available.
- **Output caching** — cache whole responses at the endpoint when appropriate.

## HybridCache (recommended where available)
```csharp
builder.Services.AddHybridCache();

public class ProductService(HybridCache cache, IProductRepo repo)
{
    public ValueTask<Product> GetAsync(int id, CancellationToken ct) =>
        cache.GetOrCreateAsync($"product:{id}",
            factory: async c => await repo.GetAsync(id, c),
            options: new() { Expiration = TimeSpan.FromMinutes(10), LocalCacheExpiration = TimeSpan.FromMinutes(2) },
            cancellationToken: ct);
}
```

## The parts people forget
- **Stampede / cache miss storm** — without protection, many requests recompute on expiry. HybridCache handles it; with `IMemoryCache` add a per-key lock (`SemaphoreSlim`) or `LazyCache`.
- **Keys** — stable, namespaced, include all inputs that change the result (`product:{id}:{culture}`).
- **Expiration** — absolute + sliding deliberately chosen; don't cache forever.
- **Invalidation** — the hard part. On write, evict/refresh affected keys (`cache.RemoveAsync`). Tag-based eviction for groups.
- **Don't cache user-specific/secret data in a shared cache** without scoping the key.

## Principles
- Cache to cut **expensive, repeated, tolerably-stale** reads — not everything.
- Every cache entry needs an expiration and an invalidation story before it ships.
- Measure hit rate; a low-hit cache is just overhead.

## How to use it & best prompts
"Cache this expensive query", "add Redis caching", "use HybridCache here", "I'm getting a cache stampede on expiry — fix it", "how do I invalidate this cache on update".
