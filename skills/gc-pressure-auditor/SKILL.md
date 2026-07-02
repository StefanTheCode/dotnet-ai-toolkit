---
name: gc-pressure-auditor
description: Diagnose garbage-collection pressure in a .NET service — Large Object Heap usage, Gen2 churn, allocation rate, server vs workstation GC, and config — and recommend fixes. Use this skill whenever the user reports GC pauses, high CPU in GC, memory growth, LOH fragmentation, Gen2 collections, or asks "why is the GC running so much / reduce GC pauses / tune GC". Always use this skill for GC-pressure diagnosis; it covers config, allocation sources, and pooling.
category: Performance
version: 1.0.0
---

# GC Pressure Auditor

Diagnose why the garbage collector is working hard in a .NET service and recommend concrete fixes — config, allocation reduction, and pooling — based on the symptoms.

## Input — point it at your project / paste the symptoms
Works on a target (a **project** or **GitHub URL**) and/or counters the user shares (Gen0/1/2 counts, `% Time in GC`, allocation rate, LOH size). Check config: `grep -rn "ServerGarbageCollection\|ConcurrentGarbageCollection\|GCHeapHardLimit" <target>` and the `.csproj`/`runtimeconfig`.

## What to check
1. **Server vs Workstation GC** — a throughput server should use **Server GC** (`<ServerGarbageCollection>true</ServerGarbageCollection>`). Workstation GC on a busy server = avoidable pauses.
2. **Concurrent GC** — keep background GC on for latency-sensitive services.
3. **Allocation rate** — high Gen0 churn means too many short-lived allocations. Trace the sources (`memory-allocation-analyzer`).
4. **Gen2 churn** — objects surviving to Gen2 then dying = mid-lifetime objects; often caching done wrong or large graphs held too long.
5. **Large Object Heap (LOH)** — arrays/strings ≥ 85 KB go to the LOH, which fragments and is collected with Gen2. Pool large buffers (`ArrayPool<T>`), stream instead of buffering, chunk large payloads.
6. **Pinning / fragmentation** — long-lived pinned buffers fragment the heap.
7. **Hard limits in containers** — set `GCHeapHardLimit`/honor cgroup limits so the GC sizes correctly in k8s.

## Common fixes
- Switch to Server GC + concurrent for throughput services.
- Pool large/short-lived buffers with `ArrayPool<T>.Shared` / `RecyclableMemoryStream`.
- Cut steady-state allocations on hot paths.
- Avoid LOH: stream large files, chunk, reuse buffers.

## Principles
- **GC pressure is an allocation problem first.** Config tuning helps, but reducing what you allocate helps more.
- Diagnose with data (`dotnet-counters`, `dotnet-trace`, GC stats) — don't tune blind.
- LOH is the usual culprit for big, periodic pauses.

## How to use it & best prompts
"Why is GC eating CPU", "reduce GC pauses in this service", "LOH keeps growing", "should I use Server GC", "tune GC for our container limits". Pairs with `memory-allocation-analyzer`.
