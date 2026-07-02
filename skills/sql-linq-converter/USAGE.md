# How to use — SQL ⇄ LINQ Converter

**What it is.** Converts raw SQL to EF Core LINQ and LINQ to its generated SQL, both directions, preserving the result set and flagging constructs that won't translate.

**When to reach for it.** Migrating SQL/stored procs/ADO.NET to EF, or checking what SQL your LINQ produces.

**How to use it.** Paste the query, or point it at a project (path or GitHub URL) for the entity context. Example prompts:
- "Convert this SQL to EF Core LINQ."
- "What SQL does this LINQ generate?"
- "Port this stored procedure to LINQ."

**Get the best out of it.** Give it the entity definitions for SQL→LINQ. Ask to see the generated SQL to verify. Heavy analytics? It'll tell you when to keep it in SQL via `FromSql`.

**Won't do.** It won't force awkward LINQ for queries that genuinely belong in SQL — it flags those.
