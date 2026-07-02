---
name: stj-serialization-optimizer
description: Optimize System.Text.Json usage in .NET — source-generated contexts, options reuse, streaming, and avoiding reflection/allocation overhead. Use this skill whenever the user works with System.Text.Json, JsonSerializer, slow or allocation-heavy serialization, source generation, custom converters, or asks "how do I speed up JSON / optimize serialization / reduce JSON allocations". Always use this skill for System.Text.Json performance; it applies source-gen and the common fixes.
category: Performance
version: 1.0.0
---

# System.Text.Json Optimizer

Make JSON serialization faster and leaner with System.Text.Json — source generation, reused options, streaming, and the small mistakes that cost a lot.

## Input — point it at your code
Works on a target: a **file**, **folder**, **project**, or **GitHub URL**. Find usage: `grep -rn "JsonSerializer\.\|JsonSerializerOptions\|System.Text.Json" --include=*.cs <target>`.

## Top fixes
1. **Reuse `JsonSerializerOptions`** — building a new options object per call is expensive (it caches metadata). Create one static instance and reuse it. This alone is a common big win.
2. **Source generation** — avoid reflection at runtime with a `JsonSerializerContext`:
```csharp
[JsonSerializable(typeof(Order))]
[JsonSerializable(typeof(List<Order>))]
public partial class AppJsonContext : JsonSerializerContext { }

var json = JsonSerializer.Serialize(order, AppJsonContext.Default.Order);
```
Faster startup, lower allocations, AOT-friendly.
3. **Stream, don't buffer** — `JsonSerializer.SerializeAsync(stream, ...)` / `DeserializeAsync` for large payloads and HTTP bodies instead of `Serialize` to a string then write.
4. **`Utf8JsonWriter`/`Utf8JsonReader`** for the hottest custom paths — work on `byte`/UTF-8 directly, skip the string.
5. **Trim the payload** — `[JsonIgnore]`, `DefaultIgnoreCondition = WhenWritingNull`, shorter property names where appropriate.
6. **Custom converters** only when needed — and make them allocation-light.

## Principles
- **Options reuse + source generation** are 80% of the wins — do those first.
- Stream large payloads; don't materialize giant strings.
- Measure with `[MemoryDiagnoser]` — serialization is often an allocation story.
- Don't hand-roll converters the built-in serializer already handles.

## How to use it & best prompts
"Speed up my JSON serialization", "add System.Text.Json source generation", "why does serializing this allocate so much", "stream this large JSON response". Pairs with `benchmarkdotnet-setup`.
