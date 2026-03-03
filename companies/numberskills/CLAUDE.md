# Fabric & Power BI Toolkit — Agent Instructions

This workspace has 3 MCP servers configured:
1. **fabric-core** — 138+ tools for Microsoft Fabric management (workspaces, lakehouses, SQL, DAX, notebooks, pipelines, OneLake, Graph, Git, CI/CD, environments, connections, admin, item definitions, Spark jobs, raw API)
2. **powerbi-modeling** — Microsoft's Power BI Modeling MCP for live semantic model editing in Power BI Desktop
3. **powerbi-translation-audit** — Translation validation tools (scan for untranslated content, PASS/FAIL verdict)

To re-run setup: Command Palette > **Fabric & Power BI: Full Setup**

---

## fabric-core: Context Flow

- Always `set_workspace` before other operations
- Always `set_lakehouse` or `set_warehouse` before SQL/table operations
- Use `list_tables` or `SELECT TOP 1 *` to discover schema before writing queries

## fabric-core: SQL Rules

- Always pass `type` ("lakehouse" or "warehouse") to sql_query, sql_explain, sql_export
- Fabric SQL endpoints are read-only — no INSERT/UPDATE/DELETE/DDL
- New delta tables take 5-10 min to appear in SQL endpoint
- Use T-SQL dialect with FORMAT() for readable numbers
- NEVER guess table names, schemas, or column names. When querying a lakehouse for the first time:
  1. Run `SELECT TABLE_SCHEMA, TABLE_NAME FROM INFORMATION_SCHEMA.TABLES` to discover what exists
  2. Run `SELECT TOP 1 * FROM <schema>.<table>` on relevant tables to get actual column names
  3. Then write the real query using the discovered names
  Column conventions vary wildly (PascalCase, snake_case, etc). Always discover, never assume.

## fabric-core: Data Questions

- Translate natural language to SQL automatically — don't ask, just query
- When querying unfamiliar data, silently discover tables and columns first, then write the query. Don't show the discovery steps to the user unless they ask.
- Format results as markdown tables
- Default to TOP 20 for ranked queries
- Suggest follow-up analyses when relevant

## fabric-core: Semantic Models & DAX

- Add descriptions when creating measures
- Use format strings (e.g., "#,0.00", "0.0%") on measures
- When gold tables are created, suggest relevant DAX measures

## fabric-core: OneLake

- `onelake_ls` browses Files path by default — use `path: "Tables"` for delta tables
- Shortcuts reference data without copying — prefer over duplication

## fabric-core: Notebooks

- Use `create_pyspark_notebook` with templates (basic, etl, analytics, ml)
- `run_notebook_job` + `get_run_status` to execute and poll
- `update_notebook_cell` caches the original notebook before writing — use `restore_notebook` to undo if needed
- If a notebook read fails during update, the operation aborts to prevent data loss

## fabric-core: Naming Conventions

- Bronze: `{source}_{entity}_raw`
- Silver: `{entity}_clean`
- Gold: `fact_{entity}`, `dim_{entity}`
- Measures: `Total {Metric}`, `{Metric} YTD`, `{Metric} Growth %`

---

## powerbi-modeling: Usage

Use ToolSearch for `powerbi-modeling` to discover available tools.
Common operations: translate semantic model captions, manage cultures, edit model metadata, create/edit measures, batch operations.
This server talks to a **running Power BI Desktop instance** — the report must be open in Desktop.

---

## Context Discipline (CRITICAL — read on every task)

These rules prevent hallucination and context loss during long multi-step Fabric operations.

### Rule 1: Track State with Task Lists
For any task requiring 3+ tool calls, create a task list BEFORE starting. Mark tasks in_progress/completed as you go. This is your source of truth — if you forget where you are, check the task list.

### Rule 2: Anchor to the Objective
Before every major step, re-state the goal in one sentence. Example: "Goal: load CSV into bronze table and create silver view. Currently on step 3 of 5: running uv sync." If you cannot state the goal, STOP and re-read the conversation.

### Rule 3: Verify, Don't Assume
- Never assume a previous tool call succeeded — check the result
- Never assume workspace/lakehouse context is still set — if in doubt, call `set_workspace`/`set_lakehouse` again (they're cheap)
- Never fabricate table names, column names, or tool output
- If a tool returns an error, report it. Don't silently retry with made-up data.

### Rule 4: Limit Output to Save Context
- `sql_query`: Use `max_rows: 10` unless user needs more. Don't dump 100 rows.
- `list_tables`: Read the result, summarize for user. Don't echo the full JSON.
- `table_preview`: Use `limit: 5` for schema discovery, not 50.
- `dax_query`: Limit results. Summarize aggregates, don't paste raw rows.
- When presenting results, use compact markdown tables. Truncate after 20 rows with "... and N more rows."

### Rule 5: Checkpoint After Each Phase
After completing a logical phase (e.g., "schema discovered", "data loaded", "measures created"), output a brief status line:
```
✓ Phase 2 complete: 3 tables loaded to bronze. Next: create silver views.
```
This anchors both you and the user.

### Rule 6: Fail Explicitly
If a step fails and you can't recover:
1. State what failed and why
2. State what completed successfully so far
3. Ask the user how to proceed
Never silently skip a failed step and continue as if it worked.

### Rule 7: One Domain at a Time
For cross-domain tasks (e.g., "load data AND create DAX measures"), split into phases. Complete data engineering fully, verify results, THEN switch to DAX. Don't interleave — it causes context confusion.

---

## Agent Routing

**Preferred: Use `/deploy-agents` slash command.** It decomposes the task into domains, launches parallel subagents via the Agent tool, and synthesizes results. Each subagent gets its own context window with domain-specific expertise — prevents the context bloat and hallucination that happens when one agent tries to do everything.

Example: `/deploy-agents load CSV into lakehouse and create DAX measures`
→ Launches data-engineer subagent (load data), waits, then launches dax subagent (create measures with discovered table names)

Example: `/deploy-agents load data and connect to git`
→ Launches data-engineer + cicd-engineer in parallel (independent work)

**Manual routing** (if not using the slash command — for simple single-domain tasks):

- **Any multi-step task** → ALWAYS read `.claude/agents/_operational-discipline.md` first
- **Lakehouses, ETL, delta tables, data loading, notebooks** → then read `.claude/agents/data-engineer.md`
- **DAX measures, semantic models, model optimization** → then read `.claude/agents/dax-analyst.md`
- **Translating Power BI reports** → then read `.claude/agents/translator.md`
- **SQL queries, data questions, analytics** → then read `.claude/agents/sql-analyst.md`
- **Git integration, deployment pipelines, CI/CD** → then read `.claude/agents/cicd-engineer.md`

---

## Known Limitations

| Tool | Issue |
|------|-------|
| `install_requirements` / `install_wheel` | Non-functional — returns guidance to use Environments API or Fabric portal instead. |
| `create_measure` / `update_measure` / `delete_measure` / `get_model_schema` | Only works with user-created semantic models. Auto-generated lakehouse default models don't support `getDefinition`. |
| `set_permissions` | Workspace-level only. Fabric REST API doesn't support item-level permissions. |
| Lakehouse SQL | Read-only. No INSERT/UPDATE/DELETE/DDL. New delta tables take 5-10 min to appear. |
