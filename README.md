# Fabric & Power BI Company Configs

Company-specific knowledge bases for the [Fabric & Power BI Toolkit](https://github.com/Jasuni69/autonomy_mcp).

## Structure

```
companies/
├── numberskills/
│   ├── CLAUDE.md          # Project instructions
│   ├── agents/            # Agent .md files
│   └── skills/            # Skill .md files
├── projektportalen/
│   ├── CLAUDE.md
│   ├── agents/
│   └── skills/
└── <your-company>/
    ├── CLAUDE.md
    ├── agents/
    └── skills/
```

## How it works

1. User runs the Fabric & Power BI Toolkit CLI
2. Picks **Company Config** from the menu
3. CLI clones this repo, lists companies
4. User picks their company
5. Files sync to their local setup

## Adding a company

1. Create a folder under `companies/<company-name>/`
2. Add `CLAUDE.md` with project-specific instructions
3. Add agent files in `agents/` (e.g. `data-engineer.md`, `dax-analyst.md`)
4. Add skill files in `skills/` if needed
5. Commit and push
