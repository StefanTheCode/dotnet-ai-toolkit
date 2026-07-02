---
name: ef-migration-reviewer
description: Review EF Core migrations for destructive or risky operations before they hit production — dropped columns/tables, non-nullable adds without defaults, renames that drop data, type changes, and long-locking operations. Use this skill whenever the user has an EF Core migration, a generated Migration.cs, a pending schema change, or asks "is this migration safe / will this migration lose data / review my migration". Always use this skill for migration-safety questions; it runs a fixed risk checklist.
category: EF Core / Database
version: 1.0.0
---

# EF Core Migration Reviewer

Catch data-loss and downtime risks in an EF Core migration before it runs in production, and suggest a safer rewrite (often an expand/contract split).

## Input — point it at your migrations
Works on a target: a **migration file**, the **Migrations/ folder**, a **project**, or a **GitHub URL**. Find them with: `find <target> -path "*Migrations*" -name "*.cs" | grep -vi Designer`. Review the most recent unapplied migration(s) first; read `Up()` and `Down()`.

## Risk checklist
1. 🔴 **Dropped column/table** (`DropColumn`, `DropTable`) — permanent data loss. Confirm it's intended and backed up.
2. 🔴 **Add non-nullable column without default** to a populated table — fails or forces a bad default. Add a default or backfill first.
3. 🔴 **Rename via drop+add** — EF sometimes models a rename as drop old + add new = data loss. Use `RenameColumn`/`RenameTable`.
4. 🟡 **Type change** (e.g. `string`→`int`, widening/narrowing) — can truncate or fail on existing rows. Verify cast safety.
5. 🟡 **Long-locking op on a big table** — adding an index or `NOT NULL` can lock writes. Prefer `CREATE INDEX CONCURRENTLY` (Postgres) / online index (SQL Server) via raw SQL.
6. 🟡 **No `Down()`** or a `Down()` that can't restore — migrations should be reversible where feasible.
7. 🟢 **Multiple unrelated changes** in one migration — harder to roll back; suggest splitting.

## Safer pattern: expand / contract
For risky renames/type changes on live systems, split across deploys:
1. **Expand** — add the new column (nullable), backfill, dual-write.
2. **Migrate** — switch reads to the new column.
3. **Contract** — drop the old column in a later migration once nothing uses it.

## Output
```
## Verdict: safe / risky / data-loss
## Findings
🔴 <op> at <file> — <risk> → <safer approach>
## Suggested rewrite
<migration code or expand/contract plan>
## Before you run
<backup / backfill / off-peak notes>
```

## Principles
- A migration that loses data should be a deliberate, backed-up decision — never a surprise.
- On a live DB, schema changes are deploys with timing and locking implications, not just code.
- Reversibility matters: prefer changes you can roll back.

## How to use it & best prompts
"Review this migration before I deploy", "will this migration lose data", "make this column rename safe on a live table", "split this risky migration into expand/contract".
