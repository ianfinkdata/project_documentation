# claude_powerbi

project_identifier: [p-20001](https://github.com/ianfinkdata/project_documentation)

## Use claude as a power bi agent.

---

| field | value |
|---|---|
| status | active |
| opened | 2026-05-12 |
| closed | — |
| owner | Ian Fink |

## Summary

Explore using Claude as an agent layer on top of Power BI workflows — likely covering query generation, report interpretation, and/or conversational access to data without requiring the user to write DAX or navigate the Power BI interface directly.

## Scope

- Identify where Claude can reduce manual effort in the current Power BI workflow
- Determine whether the integration point is at the data layer (SQL/DAX generation) or the report layer (interpreting visuals, answering questions about outputs)
- Document a repeatable pattern that can be applied to future projects

## Open questions

- What is the primary input — natural language prompts, existing reports, raw data?
- Is the goal automation, assistance, or both?

## MCP Server integration

Microsoft released a Power BI MCP Server (preview, Nov 2025) that allows Claude to interact directly with Power BI semantic models without opening the application. This is the primary integration path being adopted.

**What it enables:**
- **DAX generation & optimization** — translate plain English into DAX measures, benchmark and refactor existing queries
- **Semantic model documentation** — bulk update table/field descriptions, generate business-language explanations of measures
- **Power Query / M code** — write transformation logic from natural language descriptions
- **Conversational analytics** — query live Fabric semantic models and get answers without building a report
- **Report development** — handle technical syntax while the user focuses on the business question

The integration point is the semantic model layer, not the report or UI layer.

## Folder structure

```
p-20001_claude_powerbi/
├── p-20001_claude_powerbi_readme.md
└── pbip_sales_pipeline_demo/
    ├── p-20001_model_audit_workflow.md
    ├── Sales Pipeline Demo.pbip
    ├── Sales Pipeline Demo.Report/
    └── Sales Pipeline Demo.SemanticModel/
```

Ticket folders under `active/`, `paused/`, and `complete/` always start with `p-` (e.g., `p-20001_claude_powerbi/`). The main readme uses the `_readme` suffix to distinguish it from artifact-specific docs. Artifact-specific docs (such as model audit workflows) live inside their own artifact subfolder, not at the ticket root.

Power BI project folders are named `pbip_[filename]` using the `.pbip` filename (without extension). This keeps the report identity visible in the directory listing and supports multiple `.pbip` projects within the same ticket folder.

## Reference links

- [Use Claude to Talk to Your Power BI XMLA Data via CData Connect AI](https://www.cdata.com/kb/tech/powerbixmla-cloud-claude.rst)
- [AI as a Thinking Partner: Using Claude + Power BI MCP in Campaign Analysis](https://medium.com/@tomato.fang/ai-as-a-thinking-partner-using-claude-power-bi-mcp-in-campaign-analysis-f7d3d4ffcd25)
- [Claude + Power BI END-USER AI Analytics (Fabric Models + MCP) — YouTube](https://www.youtube.com/watch?v=OTZVL-leV38)
- [Claude + Power BI Integration — MASSIVE Breakthrough via MCP (Nov 2025) — YouTube](https://www.youtube.com/watch?v=jDSoSJz4ams)
- [How to Use Claude AI to Build Power BI Reports 10x Faster in 2026](https://pcwebinars.com/how-to-use-claude-ai-to-build-power-bi-reports-10x-faster-in-2026/)
- [My Gen-BI Experiment #1: Claude Meets Power BI](https://medium.com/@tomato.fang/my-gen-bi-experiment-1-claude-meets-power-bi-4ecd1844cd50)
- [Connect Claude AI to Power BI and SAVE HOURS (MCP) — YouTube](https://www.youtube.com/watch?v=NITlilK5LPE)
- [AI for PE finance teams: Claude in Excel, MCP for Power BI](https://hollandmountain.com/ai-for-finance-teams-in-private-equity/)
- [How to connect Claude AI to Power BI with MCP Server — LinkedIn](https://www.linkedin.com/posts/datataleau_powerbi-dax-ai-activity-7397406055519088640-RITb)

