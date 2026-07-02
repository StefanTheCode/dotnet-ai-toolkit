# How to use — Temporal Tables / Auditing Setup

**What it is.** Adds change history to entities — SQL Server temporal tables (automatic point-in-time) or an interceptor-based application audit log (any DB, captures who/what).

**When to reach for it.** You need record history, point-in-time queries, or a who-changed-what audit trail.

**How to use it.** Point it at your project (path or GitHub URL). Example prompts:
- "Add temporal-table history to my Order entity."
- "Track who changed what across my entities."
- "Point-in-time query for this record as of last week."

**Get the best out of it.** Tell it your database (temporal = SQL Server; audit log = any). Say whether you need actor/reason ("who") — that pushes toward the audit-log option. Plan retention for history growth.

**Won't do.** It won't decide your PII policy — be explicit about which field values may be stored in audit rows.
