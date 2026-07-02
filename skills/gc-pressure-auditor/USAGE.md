# How to use — GC Pressure Auditor

**What it is.** Diagnoses GC pressure (LOH growth, Gen2 churn, allocation rate, Server vs Workstation GC, container limits) and recommends config + allocation + pooling fixes.

**When to reach for it.** GC pauses, high CPU in GC, steady memory growth, or LOH fragmentation.

**How to use it.** Point it at the project (path or GitHub URL) and/or paste GC counters. Example prompts:
- "Why is the GC eating CPU in this service?"
- "Our LOH keeps growing — what's wrong?"
- "Should we switch to Server GC? Tune for our k8s limits."

**Get the best out of it.** Share `dotnet-counters`/GC stats if you have them — it tunes to real data. Pairs with `memory-allocation-analyzer` for the allocation sources behind the pressure.

**Won't do.** It won't tune blind — for allocation sources it points you to profile/trace rather than guessing.
