# Model Audit Workflow

project_identifier: [p-20001](https://github.com/ianfinkdata/project_documentation)

## Overview

A repeatable process for using Claude to audit Power BI semantic models stored as `.pbip` TMDL files in git. The audit reads the plain-text TMDL source and produces a structured findings report without requiring a live Power BI connection.

---

## Phases

### Phase 1 — Git/TMDL audit (POC)

**Trigger:** Manual or on pull request when any `.tmdl` file changes.

**How it works:**
1. Claude reads all `.tmdl` files in the semantic model's `definition/` folder
2. Claude evaluates each file against the checklist below
3. Output is a structured findings report (see format below)

**What's needed:** The `.pbip` project checked into git. No credentials, no MCP server, no live connection.

**Limitation:** Audits the committed state only — does not reach models that aren't in git.

---

### Phase 2 — Live MCP audit (extended)

**Trigger:** On-demand via Claude conversation connected to the Power BI MCP server.

**How it works:**
1. Claude connects to the Fabric workspace via the Power BI MCP server
2. Claude pulls the semantic model schema directly from the live service
3. Same checklist and output format as Phase 1

**What's needed:** Power BI MCP server configured and authenticated. Reaches models that aren't in git.

**Note:** The Power BI MCP server was in preview as of Nov 2025. Confirm current GA status before recommending for production use.

---

## Audit Checklist

### Category A — Summarization
| Check | What to look for |
|---|---|
| ID columns summing | FK/PK columns with `summarizeBy: sum` — summing identifiers is meaningless |
| Numeric dimensions summing | Columns like `employee_count`, `annual_revenue` on dimension tables — assess whether `none` is intentional or an oversight |

### Category B — Data types and consistency
| Check | What to look for |
|---|---|
| Boolean flag consistency | Flag columns (named `is_*`) should use a consistent type — either all `boolean` or all `int64` (0/1), not mixed |
| Date format consistency | Date columns should use a consistent format string across tables |
| Naming convention | All columns and tables should follow the same case convention (snake_case vs. PascalCase) |

### Category C — Relationships
| Check | What to look for |
|---|---|
| AutoDetected relationships | Flag for manual review — verify cardinality and cross-filter direction are correct |
| Relationship completeness | All FK columns in fact/bridge tables should have a corresponding relationship |

### Category D — Measures
| Check | What to look for |
|---|---|
| No measures defined | A model with zero DAX measures relies entirely on implicit aggregation — flag and suggest foundational measures |
| Missing descriptions | Measures without `///` description annotations |

### Category E — M query concerns
| Check | What to look for |
|---|---|
| Hardcoded date windows | Filters like "previous N years" that could silently drop historical data |
| Computed columns in M | Values calculated in M that depend on the current date (e.g., tenure, age, days open) — these go stale between refreshes and are better as DAX measures |
| Format string bugs | Date format strings should be verified (e.g., `yyy` vs `yyyy`) |

### Category F — Documentation
| Check | What to look for |
|---|---|
| Missing column descriptions | Columns without `///` annotation |
| Missing table descriptions | Tables without a `///` annotation above the `table` declaration |

---

## Output Format

Each finding includes:
- **Severity:** `ERROR` / `WARNING` / `INFO`
- **Table:** which table
- **Column / object:** specific column, relationship, or measure (if applicable)
- **Finding:** what the issue is
- **Recommendation:** what to change and why
- **TMDL fix:** the corrected TMDL snippet where applicable

---

## Sample Audit — Sales Pipeline Demo

Model: `Sales Pipeline Demo.SemanticModel`
Tables audited: `fact_opportunities`, `dim_accounts`, `dim_sales_reps`, `dim_date`, `bridge_opportunity_date`
Relationships audited: 4
Measures audited: 0

---

### Findings

---

**[ERROR] fact_opportunities — column product_id — Invalid summarization**

`product_id` has `summarizeBy: sum`. This is a foreign key column; summing it produces a meaningless number and will appear as a default aggregation in any visual that uses this field.

Recommendation: Change to `summarizeBy: none`.

```
// before
column product_id
    summarizeBy: sum

// after
column product_id
    summarizeBy: none
```

---

**[WARNING] fact_opportunities — Inconsistent boolean flag types**

Four flag columns use two different data types:

| Column | Type |
|---|---|
| `is_closed` | `boolean` |
| `is_won` | `boolean` |
| `is_open` | `int64` (0/1) |
| `is_lost` | `int64` (0/1) |

This inconsistency causes different default behavior in visuals and DAX. Pick one convention and apply it across all four.

Recommendation: Convert `is_open` and `is_lost` to `boolean` to match `is_closed` and `is_won`.

---

**[WARNING] dim_sales_reps — column tenure_days — Stale computed value**

`tenure_days` is imported as a column calculated at source (days since `hire_date`). It goes stale between refreshes — if the model refreshes weekly, tenure can be off by up to 7 days for all active reps.

Recommendation: Remove `tenure_days` from the source query and replace with a DAX measure: `Tenure Days = DATEDIFF(MAX(dim_sales_reps[hire_date]), TODAY(), DAY)`. This always reflects the current date.

---

**[WARNING] bridge_opportunity_date — M query — Possible date format bug**

The `DateKey` computed column uses:
```
Int64.From(Date.ToText([date], "yyyMMdd"))
```

The format string has three `y` characters (`yyy`) instead of four (`yyyy`). In some locales Power Query treats `yyy` as equivalent to `yyyy`, but this is locale-dependent and not guaranteed. If DateKey values come out as 7-digit numbers instead of 8-digit, date relationships will silently break.

Recommendation: Change to `"yyyyMMdd"` to be explicit.

---

**[WARNING] Relationships — 2 of 4 are AutoDetected**

| Relationship | Status |
|---|---|
| `bridge_opportunity_date.opportunity_id` → `fact_opportunities.opportunity_id` | AutoDetected |
| `fact_opportunities.sales_rep_id` → `dim_sales_reps.sales_rep_id` | AutoDetected |
| `fact_opportunities.account_id` → `dim_accounts.account_id` | Manual |
| `bridge_opportunity_date.DateKey` → `dim_date.DateKey` | AutoDetected |

AutoDetected relationships default to many-to-many with single cross-filter direction. For a star schema these should be many-to-one. Verify in Power BI Desktop that cardinality is correctly set for all four relationships.

---

**[WARNING] dim_date — Hardcoded date window in M query**

The dim_date partition filters to:
```
Date.IsInPreviousNYears([Date], 2) or Date.IsInCurrentYear([Date])
```

This means the date table only covers approximately 3 years. Any historical analysis beyond that range will return blanks without an obvious error. If historical reporting is a requirement, this window needs to be widened or made configurable.

---

**[WARNING] dim_date — Naming convention mismatch**

`dim_date` columns use PascalCase (`DateKey`, `Year`, `Month`, `MonthNum`, `WeekDay`, `WeekDayName`, `IsWeekend`, `WeekStart`, `ISOWeekStart`). All other tables use snake_case. This inconsistency creates a visible seam in the field list and complicates DAX that references columns across tables.

Recommendation: Rename dim_date columns to snake_case (`date_key`, `year`, `month`, `month_num`, etc.) to match the rest of the model. Note this is a breaking change if any reports already reference these columns by name.

---

**[INFO] No DAX measures defined**

The model contains zero explicit DAX measures. All aggregation relies on implicit column-level `summarizeBy` settings. This works for basic visuals but makes it difficult to enforce consistent business logic (e.g., win rate definition, weighted pipeline calculation).

Recommended foundational measures to add:

| Measure | DAX sketch |
|---|---|
| Total Revenue | `SUM(fact_opportunities[amount])` |
| Weighted Pipeline | `SUMX(fact_opportunities, fact_opportunities[amount] * fact_opportunities[probability])` |
| Win Rate | `DIVIDE(COUNTROWS(FILTER(fact_opportunities, fact_opportunities[is_won])), COUNTROWS(fact_opportunities))` |
| Open Opportunity Count | `SUMX(fact_opportunities, fact_opportunities[is_open])` |
| Avg Deal Size | `AVERAGE(fact_opportunities[amount])` |
| Avg Actual Cycle Days | `AVERAGE(fact_opportunities[actual_cycle_days])` |

---

### Summary scorecard

| Severity | Count |
|---|---|
| ERROR | 1 |
| WARNING | 6 |
| INFO | 1 |
| **Total** | **8** |

| Category | Issues |
|---|---|
| Summarization | 1 |
| Data types / consistency | 2 |
| Relationships | 1 |
| M query | 2 |
| Measures | 1 |
| Documentation | 0 (all columns have descriptions) |

---

## Action Items — Sales Pipeline Demo

| # | Priority | Item | Status |
|---|---|---|---|
| 1 | Bug | `fact_opportunities.product_id` — change `summarizeBy` from `sum` to `none` | ✓ done |
| 2 | Bug | `bridge_opportunity_date` M query — fix date format string `"yyyMMdd"` → `"yyyyMMdd"` | ✓ done |
| 3 | High | Define six foundational DAX measures (see Findings section) | open — requires Desktop |
| 4 | Medium | Convert `is_open` and `is_lost` from `int64` to `boolean` | open |
| 5 | Medium | Replace `dim_sales_reps.tenure_days` stored column with a DAX measure | open |
| 6 | Medium | Widen (or parameterize) the `dim_date` 3-year filter | open |
| 7 | Low | Rename `dim_date` columns to snake_case — breaking change, do before reports proliferate | open |

---

## Implementation Notes

**To run this audit on a new model:**
1. Ensure the `.pbip` project is committed to git with TMDL format enabled (File → Settings → Preview Features → Store semantic model in TMDL format)
2. Point Claude at the `definition/` folder of the semantic model
3. Ask Claude to run the audit checklist above and produce findings in the format above

**To automate via CI:**
- Add a GitHub Actions workflow that triggers on changes to `**/*.tmdl`
- Use Claude API with the TMDL file contents as context
- Post the findings report as a PR comment

**Extending to multiple models:**
- Each `.pbip` project in the repo is independently auditable
- A wrapper script can loop over all semantic model `definition/` folders and concatenate findings into a single report
- With the MCP server, the same audit can reach models that aren't in git
