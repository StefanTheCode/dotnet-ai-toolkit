---
name: memory-allocation-analyzer
description: Find unnecessary heap allocations in C# hot paths — LINQ in loops, boxing, closures, string concatenation, params arrays, async state machines — and rewrite to reduce GC pressure. Use this skill whenever the user wants to reduce allocations, cut GC pressure, find boxing/closures, optimize a hot path's memory, or asks "why does this allocate so much / reduce garbage". Always use this skill for allocation-reduction requests; it runs a fixed checklist and rewrites the code.
category: Performance
version: 1.0.0
---

# Memory Allocation Analyzer

Spot avoidable heap allocations on hot paths and rewrite them to cut GC pressure — without sacrificing readability where it doesn't matter.

## Input — point it at your code
Works on a target: a **file**, **folder**, **project**, or **GitHub URL**. Focus on hot paths (request handlers, loops, serialization). Find candidates: `grep -rn "\.Select(\|\.Where(\|\.ToList()\|+ \"\|string.Format\|new \[\]" --include=*.cs <target>`.

## Allocation checklist
1. **LINQ in hot loops** — `Where/Select/ToList` allocate iterators + lists per call. In a tight loop, a plain `for` with no intermediate collection avoids it.
2. **Boxing** — value type → `object`/interface (e.g. `int` into a non-generic API, struct enumerator boxing). Use generics; avoid `object`.
3. **Closures** — lambdas capturing locals allocate a display class. Hoist captured state or use static lambdas (`static () => ...`).
4. **String building** — concatenation in loops → `StringBuilder` or `string.Create`/interpolation handler.
5. **`params` arrays** — hidden array allocation per call; provide non-params overloads on hot APIs.
6. **Unnecessary `async`** — a method that often completes synchronously → `ValueTask` to avoid the `Task` allocation.
7. **Defensive copies** — large structs passed by value; use `in`/`ref readonly`.
8. **Collections sized wrong** — pre-size `List`/`Dictionary` capacity to avoid re-allocation.

## Rewrite example
```csharp
// allocates: closure + iterator + list, every call
var ids = items.Where(i => i.IsActive).Select(i => i.Id).ToList();

// hot path: no intermediate allocations
var ids = new List<int>(items.Count);
foreach (var i in items) if (i.IsActive) ids.Add(i.Id);
```

## Principles
- **Only optimize hot paths.** LINQ readability wins everywhere else — don't uglify cold code to save nanoseconds.
- **Measure with `[MemoryDiagnoser]`** (`benchmarkdotnet-setup`) before and after — guesses mislead.
- Allocation reduction is about *steady-state GC pressure*, not one-off startup cost.

## Output
```
## Findings (hot paths only)
- <file:line> [closure] ... → ...
## Rewritten
<code>
## Verify
Benchmark with [MemoryDiagnoser]; compare Allocated/Gen0.
```

## How to use it & best prompts
"Why does this allocate so much", "reduce GC pressure in this handler", "find boxing/closures here", "cut allocations in this hot loop". Pairs with `benchmarkdotnet-setup` and `gc-pressure-auditor`.
