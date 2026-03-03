# Fabric & Power BI — Company Configs

Company-specific knowledge bases for the [Fabric & Power BI Toolkit](https://marketplace.visualstudio.com/items?itemName=JasonNicolini.fabric-powerbi-toolkit).

---

## Folder structure

```
companies/
└── <your-company>/
    ├── CLAUDE.md                  # Required — project-level instructions
    ├── agents/                    # Required — one .md per role
    │   ├── data-engineer.md
    │   ├── dax-analyst.md
    │   ├── sql-analyst.md
    │   ├── translator.md
    │   └── cicd-engineer.md
    └── skills/                    # Optional — custom slash commands
        └── my-skill.md
```

Create a folder under `companies/` matching your company name. The CLI shows this name to users.

---

## File formats

### CLAUDE.md

This is the top-level file Claude reads on every conversation. Put your company's global rules here.

```markdown
# Company Name — Agent Instructions

## Naming Conventions
- Bronze tables: `{source}_{entity}_raw`
- Silver tables: `{entity}_clean`
- Gold tables: `fact_{entity}`, `dim_{entity}`

## SQL Rules
- Always use schema-qualified names: `dbo.table_name`
- Default to TOP 20 for ranked queries
- Format numbers with FORMAT()

## Workspaces
- Production: `PROD-Analytics`
- Development: `DEV-Analytics`

## DAX Standards
- Measures go in a `_Measures` table
- Use format strings: "#,0.00" for currency, "0.0%" for percentages
- Prefix KPIs with "KPI — "
```

**Keep it concise.** Claude reads this every time. Rules, not essays.

---

### Agent files (`agents/*.md`)

Each file defines one role. Start with a one-line identity, then list rules and workflows.

**Filename must match one of:**
- `data-engineer.md`
- `dax-analyst.md`
- `sql-analyst.md`
- `translator.md`
- `cicd-engineer.md`

Example `agents/data-engineer.md`:

```markdown
# Data Engineer

You are a senior Fabric data engineer at <Company>.

## Core Rules

1. Always discover schema before writing queries — run `list_tables` then `SELECT TOP 1 *`
2. Follow medallion architecture: Bronze → Silver → Gold
3. Run `lakehouse_table_maintenance` after bulk loads
4. Use `enable_schemas=True` on new lakehouses

## Company-Specific

- Our bronze lakehouse is named `raw_data_lh`
- Silver lakehouse: `clean_data_lh`
- All ETL notebooks live in workspace `ETL-Pipelines`
- Source systems: SAP (daily), Salesforce (hourly), flat files (ad-hoc)

## Naming

| Layer  | Pattern                    | Example              |
|--------|----------------------------|----------------------|
| Bronze | `{source}_{entity}_raw`    | `sap_customers_raw`  |
| Silver | `{entity}_clean`           | `customers_clean`    |
| Gold   | `fact_{entity}`            | `fact_sales`         |
```

Example `agents/dax-analyst.md`:

```markdown
# DAX Analyst

You are a senior DAX developer at <Company>.

## Measure Standards

- All measures in a `_Measures` table
- Format strings: "#,0" for integers, "#,0.00" for currency, "0.0%" for pct
- Always add descriptions when creating measures
- Prefix time intelligence with period: "Sales YTD", "Sales MTD"

## Our Semantic Models

- `Sales Analytics` — fact_sales, dim_customer, dim_product, dim_date
- `Finance Model` — fact_gl, dim_account, dim_cost_center

## Common Measures We Use

- Total Revenue = SUM(fact_sales[Revenue])
- Margin % = DIVIDE([Total Revenue] - [Total Cost], [Total Revenue])
```

---

### Skill files (`skills/*.md`) — Optional

Custom slash commands for company-specific workflows. Only add these if you have repeatable processes.

```markdown
# /deploy-report

Deploy a Power BI report from DEV to PROD.

## Steps
1. Run `git_get_status` on DEV workspace
2. Commit all changes with `git_commit_to_git`
3. Deploy via `deploy_stage_content` from Dev → Test → Prod
4. Refresh semantic model with `semantic_model_refresh`
```

---

## Writing guidelines

| Do | Don't |
|----|-------|
| Write instructions in imperative form ("Always use...", "Never assume...") | Write long explanations of why |
| Keep files under 300 lines | Put everything in CLAUDE.md |
| One domain per agent file | Mix SQL rules into the DAX analyst |
| Include real table/workspace names from your environment | Use placeholder names like `my_table` |
| List your actual semantic models and lakehouses | Leave agents generic |

---

## Submitting a PR

1. Fork this repo
2. Create `companies/<your-company>/` with the files above
3. Test locally: run **Company Config** in the toolkit CLI, pick your company, verify files sync
4. Open a PR with a brief description of your company setup
