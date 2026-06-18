# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Not a software codebase — a **Power BI Desktop project (PBIP)** for education analytics (schools, students, attendance, assessments, enrollment, teacher activity).

- `Education_Sample.pbip` — project entry point (open in Power BI Desktop)
- `Education_Sample.SemanticModel/` — data model in **TMDL** (`definition/tables/*.tmdl`, `relationships.tmdl`, `model.tmdl`)
- `Education_Sample.Report/` — report in **PBIR** (`definition/pages/<pageId>/page.json` + `visuals/<id>/visual.json`, ordered by `pages/pages.json`)
- `Education_Sample_Dataset.xlsx` — source data (7 sheets) imported into the model
- `Build_Education_Model.csx` — Tabular Editor C# script that builds the **entire** model in one run (source of truth for the model)
- `Education_Model_Report.md` — **technical** deliverable (objects created · DAX fixes · structural insights · data findings)
- `Education_Analysis_Report.md` — **non-technical / business** deliverable (project requirement · questions · EDA · findings · recommendations · conclusion)
- `ClaudeCode_PowerBI_Guide.md` — the original spec this project implements

## Editing the model — do NOT use the MCP for writes

The `powerbi-modeling` MCP server allows **reads** (connect, list/get tables/columns/measures/relationships) but its consent gate **auto-declines every write and every DAX `EVALUATE`**. This is not a `settings.json` problem (the allowlist is already complete) and cannot be cleared from here. Apply all model changes by editing `Build_Education_Model.csx` and running it in **Tabular Editor 2/3**:

1. Tabular Editor → File ▸ Open ▸ From DB → `localhost:<port>`
2. Paste the script into the C# Script tab → **F5** → **Ctrl+S** to commit back to Power BI Desktop.
3. In Power BI Desktop, **Ctrl+S** to persist the model to the `.SemanticModel` TMDL files on disk.

The local Analysis Services **port is dynamic per Power BI session** — get it from MCP `connection_operations` → `ListLocalInstances` (or re-check in Power BI). The script is idempotent (checks before creating), so re-running is safe.

## Editing the report (visuals) — close Power BI first

Report visuals are plain JSON files. You can author them directly, but **Power BI Desktop does not hot-reload, and saving in Power BI overwrites external file changes** (its in-memory report wins). Safe sequence:

1. **Close** Power BI Desktop — confirm with `ListLocalInstances` (0 instances).
2. Write/edit the `page.json` / `visual.json` files (and `pages/pages.json` `pageOrder` for new pages).
3. Reopen `Education_Sample.pbip`.

Back up `Education_Sample.Report/` before bulk-authoring (a backup already exists at `Education_Sample.Report.backup`). Validate every file parses (`Get-Content … | ConvertFrom-Json`) before reopening — one malformed JSON can block the whole report from loading.

**PBIR `visual.json` shape that renders here:** top-level `name` (matches its folder), `position {x,y,z,width,height,tabOrder}`, and `visual {visualType, query.queryState.<role>.projections[]}`. Each projection `field` is `{Measure|Column: {Expression:{SourceRef:{Entity:"<table>"}}, Property:"<name>"}}` with `queryRef` = `"<table>.<name>"`. Roles by visual type: `card` / `slicer` / `tableEx` → `Values`; `clusteredColumnChart` / `clusteredBarChart` / `lineChart` → `Category` + `Y` (multiple `Y` projections allowed). Both `visualContainer/1.4.0` and `2.9.0` schema versions load; Power BI rewrites to `2.9.0` on save.

## Reading data values when DAX is gated

Because DAX `EVALUATE` is blocked, compute aggregates straight from `Education_Sample_Dataset.xlsx` — the model imports from it, so results equal the measures. Notes: the workbook has **no `sharedStrings.xml`** (strings are inline `<is><t>…</t>`, numerics are `<v>…</v>`); sheets map in declared order `sheet1`=dim_Schools, `sheet2`=dim_Students, `sheet3`=fact_Attendance, `sheet4`=fact_Assessment, `sheet5`=fact_Enrollment, `sheet6`=fact_TeacherActivity, `sheet7`=dim_Calendar; and the file is often **locked open in Excel**, so copy it to a temp path first, then read via `System.IO.Compression.ZipFile`.

## Model architecture

Star schema: 4 fact tables → 3 dimensions, with measures isolated in 4 `_Measures_*` tables (each a 1-row calculated table that holds only measures, grouped by display folder).

- **Dimensions:** `dim_Schools`, `dim_Students`, `dim_Calendar`.
- **Facts:** `fact_Attendance` (daily; Present/Absent/Late), `fact_Assessment` (per assessment event — Score/MaxScore/PassMark, Subject, Term; ~24k rows), `fact_Enrollment` (aggregate counts), `fact_TeacherActivity` (weekly per teacher).
- **Every fact joins `dim_Schools` on `SchoolID`** — school is the only dimension that filters the whole model. The snowflake `dim_Students[SchoolID] → dim_Schools` is intentionally **deactivated** so facts reach schools directly (no path ambiguity).
- **`StudentID` relationships** (`fact_Attendance` and `fact_Assessment` → `dim_Students`) are **active**, enabling student-grain analysis.
- **Calendar is connected to attendance only:** `fact_Attendance[AttendanceDate] → dim_Calendar[Date]`. Assessment, Enrollment, and Teacher Activity have **no date relationship** — trend them on their own period columns (`Term`, `AcademicYear`, `Week`), not `dim_Calendar`. This is why the YoY-enrollment, rolling-completion, and term-growth measures are keyed off those columns instead of `DATEADD`.
- `MaxScore = 100` across all rows (verified), so the raw-`Score` threshold measures (Top ≥80, At-Risk <40, Score Bands) are valid; `fact_Assessment[ScorePercent]` is the normalization hook if scales ever vary.
- Auto date/time is on, leaving `LocalDateTable_*` / `DateTableTemplate_*` tables — ignore them; disable Options ▸ Data Load ▸ Auto date/time to drop them.

## DAX / PBIR gotchas (already fixed — don't reintroduce)

- A `CALCULATE` boolean filter **cannot compare two columns** (`[Score] >= [PassMark]`) — wrap in `FILTER(table, …)`.
- **Don't sort a column by another column whose formula references it** → circular dependency that Power BI refuses to load. `ScoreBandOrder` is derived from `[Score]`, not `[ScoreBand]`, for exactly this reason.
- **`Current` is a reserved DAX identifier** — never name a `VAR` `Current` (use `CurrentScore`, etc.). Tabular Editor writes measures without validating, so such errors stay latent until Power BI opens/uses the measure.
- A measure-holder table from `AddCalculatedTable(name, "{BLANK()}")` does **not** materialize its `Value` column in Tabular Editor 2 offline — guard `if (t.Columns.Contains("Value"))` before referencing it.
