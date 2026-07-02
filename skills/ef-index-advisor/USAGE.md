# How to use — EF Core Index Advisor

**What it is.** Recommends indexes based on how your entities are actually queried (filters, joins, ordering, uniqueness) and generates the `HasIndex` config with correct column order — without over-indexing.

**When to reach for it.** A slow query you suspect needs an index, or designing indexes for a model.

**How to use it.** Point it at your project (path or GitHub URL). Example prompts:
- "What indexes does this project need?"
- "Design a composite index for filtering by customer + status, sorting by date."
- "Why is this query slow — am I missing an index?"

**Get the best out of it.** Let it scan the query sites so recommendations match real usage. Ask it to explain column order. Verify with the query plan it suggests. Pairs with `ef-core-query-optimizer`.

**Won't do.** It can't read your live query plan/stats, so it marks recommendations to verify against `EXPLAIN`/execution plan.
