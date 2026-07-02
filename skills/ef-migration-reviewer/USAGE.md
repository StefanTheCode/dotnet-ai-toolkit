# How to use — EF Core Migration Reviewer

**What it is.** Reviews an EF Core migration for data-loss and downtime risks (drops, non-nullable adds, drop+add renames, type changes, locking ops) and suggests a safer rewrite — often expand/contract.

**When to reach for it.** Before applying any migration to a populated/production database.

**How to use it.** Point it at a migration file, the `Migrations/` folder, or the project (path or GitHub URL). Example prompts:
- "Review this migration before I deploy it."
- "Will this migration lose data?"
- "Make this column rename safe on a live table."

**Get the best out of it.** Run it on the newest unapplied migration. Ask for the expand/contract split when the change is risky on a live system. Mention table size — it changes the locking advice.

**Won't do.** It can't see your data, so backfill/cast-safety items come marked "verify".
