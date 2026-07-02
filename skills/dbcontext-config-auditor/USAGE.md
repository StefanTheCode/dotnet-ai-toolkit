# How to use — DbContext Configuration Auditor

**What it is.** Audits your `DbContext`/entity configuration for the settings that quietly cause perf problems, data leaks, or odd behavior — tracking, lazy loading, sensitive logging, pooling, conventions, delete behavior.

**When to reach for it.** Reviewing EF setup, or when EF "behaves oddly" (extra queries, surprising cascades).

**How to use it.** Point it at your project (path or GitHub URL). Example prompts:
- "Review my DbContext and entity configuration."
- "Why is EF issuing extra queries?"
- "Split OnModelCreating into per-entity config classes."

**Get the best out of it.** Include `AddDbContext` registration and `OnModelCreating`/config classes. Pairs with `ef-core-query-optimizer` (query-level) and `db-resiliency-setup` (retry).

**Won't do.** It reviews configuration, not individual query bodies — use the query optimizer for those.
