# Claude Code ↔ Power BI: Complete Setup & Workflow Guide
**Education Dashboard Analytics Project**

---

## What You're Building

By the end of this guide you will have:
- Claude Code connected to Power BI Desktop via the MCP server
- A sample 7-sheet education dataset loaded into Power BI
- A fully built star-schema data model with relationships
- 20+ DAX measures covering attendance, assessment, enrollment, and lesson delivery
- A ready-to-use workflow prompt you paste into Claude Code to automate the entire thing

---

## Prerequisites

| Tool | Where to get it |
|---|---|
| **Power BI Desktop** (latest) | microsoft.com/en-us/power-bi/downloads |
| **VS Code** | code.visualstudio.com |
| **Claude Code** (CLI) | `npm install -g @anthropic-ai/claude-code` |
| **Node.js 18+** | nodejs.org |
| Anthropic API key | console.anthropic.com |

---

## Part 1 — Connect Claude Code to Power BI (Step-by-Step)

### Step 1: Enable Preview Features in Power BI Desktop

Open Power BI Desktop → **File → Options and Settings → Options → Preview Features**

Enable all four of these:
- ✅ Power BI Project (.pbip) save option
- ✅ Store semantic model using TMDL format
- ✅ Store reports using enhanced metadata format (PBIR)
- ✅ Store PBIX reports using enhanced metadata format (PBIR)

Click **OK** and restart Power BI Desktop.

---

### Step 2: Save Your File as a .pbip Project

1. Open Power BI Desktop → **File → Open** → load the `Education_Sample_Dataset.xlsx` file
2. Once loaded → **File → Save As**
3. In the file type dropdown, choose **Power BI Project files (.pbip)**
4. Save it to a project folder, e.g.: `C:\Projects\EducationDashboard\`

> The `.pbip` format is required. The MCP server communicates with Power BI Desktop through a local port — it does not read the file directly.

---

### Step 3: Install the Power BI Modeling MCP Extension in VS Code

1. Open VS Code
2. Go to **Extensions** (Ctrl+Shift+X)
3. Search: **Power BI Modeling MCP Server**
4. Publisher: **Microsoft (analysis-services)**
5. Click **Install**

After installation, find the executable on your disk at:
```
C:\Users\<YourUsername>\.vscode\extensions\
  analysis-services.powerbi-modeling-mcp-<VERSION>-win32-x64\
    server\
      powerbi-modeling-mcp.exe
```

> **Important:** Note the exact version number in the folder name (e.g. `0.4.0`). You'll need it in the next step.

---

### Step 4: Configure Claude Code to Use the MCP Server

Claude Code uses a `.claude.json` file to register MCP servers.

**For global scope** (works across all projects), edit:
`C:\Users\<YourUsername>\.claude.json`

**For project-only scope**, create a `.claude.json` in your project root:
`C:\Projects\EducationDashboard\.claude.json`

Add this configuration (replace `<YourUsername>` and `<VERSION>`):

```json
{
  "mcpServers": {
    "powerbi-modeling-mcp": {
      "type": "stdio",
      "command": "C:\\Users\\<YourUsername>\\.vscode\\extensions\\analysis-services.powerbi-modeling-mcp-<VERSION>-win32-x64\\server\\powerbi-modeling-mcp.exe",
      "args": ["--start"],
      "env": {}
    }
  }
}
```

**Key rules:**
- Every backslash must be **doubled** (`\\`) in JSON
- The version string in the path must match the folder name exactly
- If VS Code auto-updates the extension, update the version number here too

---

### Step 5: Verify the Connection

1. Open Power BI Desktop and make sure your `.pbip` file is open
2. Open a terminal in your project folder
3. Run: `claude` to start Claude Code
4. Type: `List the tables in my Power BI model`

If connected, Claude Code will return a list of your tables. If it fails, check:
- Is Power BI Desktop open with a `.pbip` file loaded?
- Does the path in `.claude.json` exactly match the executable location?
- Did you use double backslashes in the path?
- Try fully restarting both Power BI Desktop and your terminal

---

### Step 6: Understanding What MCP Can Do

Once connected, Claude Code can:
- Read your model schema (tables, columns, relationships)
- Create and edit DAX measures
- Add calculated columns
- Build relationships between tables
- Organize measures into display folders
- Query your model with DAX queries
- Validate DAX syntax before applying changes

> Claude Code **cannot** build visuals or reports directly — you still design the report canvas in Power BI Desktop. But it handles all the hard modeling work.

---

## Part 2 — The Sample Dataset

The file `Education_Sample_Dataset.xlsx` contains **7 sheets** designed as a star schema:

| Sheet | Type | Rows | Description |
|---|---|---|---|
| `dim_Schools` | Dimension | 10 | School master list — state, type, capacity |
| `dim_Students` | Dimension | 500 | Student roster — gender, grade, enrollment date |
| `dim_Calendar` | Dimension | 731 | Date table — year, month, quarter, term, school day flag |
| `fact_Attendance` | Fact | ~36,000 | Daily attendance records — present/absent/late |
| `fact_Assessment` | Fact | ~24,000 | Assessment scores by subject, term, type |
| `fact_Enrollment` | Fact | ~1,440 | Enrollment counts vs targets by school/grade/term |
| `fact_TeacherActivity` | Fact | ~2,880 | Weekly lessons planned vs delivered |

---

## Part 3 — The Claude Code Workflow Prompt

Copy the entire block below and paste it into Claude Code after your MCP connection is live. Run it once — Claude will build the full data model and all DAX measures automatically.

---

```
## EDUCATION DASHBOARD — FULL MODELING WORKFLOW

You are a Power BI expert working on an education analytics dashboard. 
The Power BI Desktop file currently has 7 sheets loaded as tables from Excel:
  dim_Schools, dim_Students, dim_Calendar,
  fact_Attendance, fact_Assessment, fact_Enrollment, fact_TeacherActivity

Work through all phases below in sequence. Confirm completion of each phase 
before moving to the next. Validate all DAX before applying it.

─────────────────────────────────────────────────────────────────────────────
PHASE 1 — DATA MODEL: BUILD THE STAR SCHEMA
─────────────────────────────────────────────────────────────────────────────

Create the following relationships (all Many-to-One, single direction unless noted):

1. fact_Attendance[SchoolID]        → dim_Schools[SchoolID]
2. fact_Attendance[StudentID]       → dim_Students[StudentID]
3. fact_Attendance[AttendanceDate]  → dim_Calendar[Date]   (use DateKey or Date)
4. fact_Assessment[SchoolID]        → dim_Schools[SchoolID]
5. fact_Assessment[StudentID]       → dim_Students[StudentID]
6. fact_Enrollment[SchoolID]        → dim_Schools[SchoolID]
7. fact_TeacherActivity[SchoolID]   → dim_Schools[SchoolID]

Set dim_Calendar as the central date dimension. Mark it as a Date Table using 
the [Date] column.

Hide all foreign key columns (SchoolID, StudentID, DateKey) from fact tables 
in Report View to keep the field list clean.

─────────────────────────────────────────────────────────────────────────────
PHASE 2 — DAX MEASURES: ATTENDANCE ANALYTICS
─────────────────────────────────────────────────────────────────────────────

Create a dedicated measures table called [_Measures_Attendance].
Create the following measures inside it:

1. Total Attendance Records
   = COUNTROWS(fact_Attendance)

2. Days Present
   = CALCULATE(COUNTROWS(fact_Attendance), fact_Attendance[Status] = "Present")

3. Days Absent
   = CALCULATE(COUNTROWS(fact_Attendance), fact_Attendance[Status] = "Absent")

4. Days Late
   = CALCULATE(COUNTROWS(fact_Attendance), fact_Attendance[Status] = "Late")

5. Attendance Rate %
   = DIVIDE([Days Present], [Total Attendance Records], 0)
   Format as percentage, 1 decimal place.

6. Absence Rate %
   = DIVIDE([Days Absent], [Total Attendance Records], 0)
   Format as percentage.

7. Chronic Absenteeism Rate
   Definition: students absent more than 10% of school days
   = 
   VAR StudentAbsencePct =
       ADDCOLUMNS(
           VALUES(dim_Students[StudentID]),
           "AbsPct", DIVIDE(
               CALCULATE([Days Absent]),
               CALCULATE([Total Attendance Records]),
               0
           )
       )
   RETURN
       DIVIDE(
           COUNTROWS(FILTER(StudentAbsencePct, [AbsPct] > 0.10)),
           COUNTROWS(StudentAbsencePct),
           0
       )
   Format as percentage.

8. Attendance Rate vs Prior Term (Period-over-Period)
   =
   VAR CurrentRate = [Attendance Rate %]
   VAR PriorRate =
       CALCULATE(
           [Attendance Rate %],
           DATEADD(dim_Calendar[Date], -1, QUARTER)
       )
   RETURN CurrentRate - PriorRate
   Format as percentage.

─────────────────────────────────────────────────────────────────────────────
PHASE 3 — DAX MEASURES: ASSESSMENT & ACADEMIC PERFORMANCE
─────────────────────────────────────────────────────────────────────────────

Create table [_Measures_Assessment]. Add:

9. Average Score
   = AVERAGE(fact_Assessment[Score])
   Format: 1 decimal place.

10. Pass Rate %
    = DIVIDE(
          CALCULATE(COUNTROWS(fact_Assessment), 
                    fact_Assessment[Score] >= fact_Assessment[PassMark]),
          COUNTROWS(fact_Assessment),
          0
      )
    Format as percentage.

11. Fail Rate %
    = 1 - [Pass Rate %]
    Format as percentage.

12. Top Performers (Score >= 80%)
    = CALCULATE(
          COUNTROWS(fact_Assessment),
          fact_Assessment[Score] >= 80
      )

13. At-Risk Students (Score < 40%)
    = CALCULATE(
          COUNTROWS(fact_Assessment),
          fact_Assessment[Score] < 40
      )

14. Subject Average Score (using SELECTEDVALUE for dynamic context)
    =
    VAR SelectedSubject = SELECTEDVALUE(fact_Assessment[Subject])
    RETURN
        IF(
            ISBLANK(SelectedSubject),
            [Average Score],
            CALCULATE([Average Score], fact_Assessment[Subject] = SelectedSubject)
        )

15. Score Band Distribution — High (>=75)
    = CALCULATE(COUNTROWS(fact_Assessment), fact_Assessment[Score] >= 75)

16. Score Band Distribution — Mid (50–74)
    = CALCULATE(COUNTROWS(fact_Assessment), 
                fact_Assessment[Score] >= 50 && fact_Assessment[Score] < 75)

17. Score Band Distribution — Low (<50)
    = CALCULATE(COUNTROWS(fact_Assessment), fact_Assessment[Score] < 50)

18. Term-on-Term Score Growth
    =
    VAR Current = [Average Score]
    VAR Prior =
        CALCULATE(
            [Average Score],
            DATEADD(dim_Calendar[Date], -1, QUARTER)
        )
    RETURN
        DIVIDE(Current - Prior, Prior, BLANK())
    Format as percentage.

─────────────────────────────────────────────────────────────────────────────
PHASE 4 — DAX MEASURES: ENROLLMENT ANALYTICS
─────────────────────────────────────────────────────────────────────────────

Create table [_Measures_Enrollment]. Add:

19. Total Enrolled
    = SUM(fact_Enrollment[EnrolledCount])

20. Total Target Enrollment
    = SUM(fact_Enrollment[TargetEnrollment])

21. Enrollment Achievement Rate %
    = DIVIDE([Total Enrolled], [Total Target Enrollment], 0)
    Format as percentage.

22. Enrollment Gap
    = [Total Target Enrollment] - [Total Enrolled]

23. Gender Parity Index
    =
    VAR GirlsEnrolled =
        CALCULATE([Total Enrolled], fact_Enrollment[Gender] = "Female")
    VAR BoysEnrolled =
        CALCULATE([Total Enrolled], fact_Enrollment[Gender] = "Male")
    RETURN
        DIVIDE(GirlsEnrolled, BoysEnrolled, BLANK())
    Format: 2 decimal places. Value of 1.0 = perfect parity.

24. YoY Enrollment Growth %
    =
    VAR CY = CALCULATE([Total Enrolled], dim_Calendar[Year] = YEAR(TODAY()))
    VAR PY = CALCULATE([Total Enrolled], dim_Calendar[Year] = YEAR(TODAY())-1)
    RETURN DIVIDE(CY - PY, PY, BLANK())
    Format as percentage.

─────────────────────────────────────────────────────────────────────────────
PHASE 5 — DAX MEASURES: TEACHER & LESSON DELIVERY
─────────────────────────────────────────────────────────────────────────────

Create table [_Measures_TeacherActivity]. Add:

25. Lessons Planned
    = SUM(fact_TeacherActivity[LessonsPlanned])

26. Lessons Delivered
    = SUM(fact_TeacherActivity[LessonsDelivered])

27. Lesson Completion Rate %
    = DIVIDE([Lessons Delivered], [Lessons Planned], 0)
    Format as percentage.

28. Lesson Completion Rate — Rolling 4 Weeks
    =
    VAR Last4Weeks =
        DATESINPERIOD(dim_Calendar[Date], LASTDATE(dim_Calendar[Date]), -28, DAY)
    RETURN
        CALCULATE([Lesson Completion Rate %], Last4Weeks)
    Format as percentage.

29. Schools Below 80% Lesson Completion
    =
    VAR SchoolRates =
        ADDCOLUMNS(
            VALUES(dim_Schools[SchoolID]),
            "Rate", CALCULATE([Lesson Completion Rate %])
        )
    RETURN
        COUNTROWS(FILTER(SchoolRates, [Rate] < 0.80))

─────────────────────────────────────────────────────────────────────────────
PHASE 6 — CALCULATED COLUMNS
─────────────────────────────────────────────────────────────────────────────

Add these calculated columns to their respective tables:

In dim_Students:
  - Age = DATEDIFF(dim_Students[DateOfBirth], TODAY(), YEAR)
  - AgeGroup = 
      SWITCH(TRUE(),
          dim_Students[Age] <= 9,  "Under 10",
          dim_Students[Age] <= 12, "10–12",
          dim_Students[Age] <= 15, "13–15",
          dim_Students[Age] <= 18, "16–18",
          "Over 18"
      )

In fact_Assessment:
  - ScoreBand =
      SWITCH(TRUE(),
          fact_Assessment[Score] >= 75, "Distinction",
          fact_Assessment[Score] >= 60, "Credit",
          fact_Assessment[Score] >= 50, "Pass",
          fact_Assessment[Score] >= 40, "Near Pass",
          "Fail"
      )
  - ScorePercent = DIVIDE(fact_Assessment[Score], fact_Assessment[MaxScore], 0)

In dim_Calendar:
  - AcademicYear =
      IF(
          dim_Calendar[Month] >= 9,
          dim_Calendar[Year] & "/" & (dim_Calendar[Year] + 1),
          (dim_Calendar[Year] - 1) & "/" & dim_Calendar[Year]
      )

─────────────────────────────────────────────────────────────────────────────
PHASE 7 — MODEL HOUSEKEEPING
─────────────────────────────────────────────────────────────────────────────

1. Organize all measures into display folders within each measures table:
   - Attendance measures → folder: "Attendance KPIs"
   - Assessment measures → folders: "Volume", "Performance", "Trends"
   - Enrollment measures → folder: "Enrollment KPIs"
   - Teacher measures → folder: "Lesson Delivery"

2. Set the following column descriptions (Descriptions property):
   - dim_Schools[Capacity] → "Maximum student intake capacity of the school"
   - fact_Attendance[Status] → "Present | Absent | Late"
   - fact_Assessment[ScoreBand] → "Distinction ≥75 | Credit ≥60 | Pass ≥50 | Near Pass ≥40 | Fail <40"
   - [Gender Parity Index] → "Female:Male enrollment ratio. 1.0 = equal. <1.0 = more boys."

3. Set sort order:
   - dim_Calendar[MonthName] → sort by [Month]
   - dim_Calendar[DayName] → sort by [DayOfWeek]
   - fact_Assessment[ScoreBand] → sort by [ScorePercent] descending

4. Confirm the model has:
   - No unresolved relationship errors
   - No circular dependency warnings
   - All measures returning non-blank values when tested with EVALUATE in DAX query view

─────────────────────────────────────────────────────────────────────────────
PHASE 8 — DASHBOARD PAGE LAYOUT RECOMMENDATIONS
─────────────────────────────────────────────────────────────────────────────

Suggest the following 4-page dashboard structure. For each page, list which 
visuals to use and which measures/fields to assign to them.

Page 1 — Executive Summary
  - Card visuals: Attendance Rate %, Pass Rate %, Enrollment Achievement Rate %, 
    Lesson Completion Rate %
  - KPI visual: Attendance Rate % vs prior term target
  - Clustered bar: Average Score by Subject
  - Line chart: Attendance Rate % over time by Term

Page 2 — Attendance Deep Dive
  - Matrix: School × Term with Attendance Rate %, Absence Rate %
  - Stacked bar: Attendance Status breakdown by School
  - Line chart: Daily attendance trend with slicer for Term
  - Map visual: Schools by State, bubble size = Chronic Absenteeism Rate

Page 3 — Academic Performance
  - Donut chart: Score Band Distribution (High/Mid/Low)
  - Clustered column: Pass Rate by Subject × Term
  - Scatter chart: Avg Score vs Attendance Rate (school level)
  - Table: Bottom 10 students by average score with Grade, School

Page 4 — Enrollment & Teacher Activity
  - Gauge: Enrollment Achievement Rate % vs 100% target
  - Bar chart: Gender Parity Index by School
  - Line chart: Lesson Completion Rate rolling 4 weeks
  - Card: Schools Below 80% Lesson Completion

─────────────────────────────────────────────────────────────────────────────
END OF WORKFLOW
─────────────────────────────────────────────────────────────────────────────

After completing all phases, provide:
1. A summary of all objects created (tables, relationships, measures, columns)
2. Any DAX measures that returned warnings or required adjustment
3. Three analytical insights you can already draw from the model structure
```

---

## Part 4 — Running the Workflow

1. Ensure Power BI Desktop is open with your `.pbip` file loaded
2. Navigate to your project folder in a terminal: `cd C:\Projects\EducationDashboard`
3. Start Claude Code: `claude`
4. Paste the entire workflow prompt from Part 3
5. Claude Code will work through all 8 phases sequentially — confirm each phase as it completes
6. Once done, switch to Power BI Desktop and build the report visuals on the canvas

---

## Troubleshooting Quick Reference

| Symptom | Fix |
|---|---|
| "MCP server not found" | Check `.claude.json` path has double backslashes and correct version |
| "Power BI not connected" | Ensure PBI Desktop is open with a `.pbip` file, not `.pbix` |
| DAX measure returns BLANK | Check relationship direction; add `CROSSFILTER` if needed |
| Version mismatch error after VS Code update | Update version string in `.claude.json` to match new extension folder |
| Claude Code doesn't list tables | Run `List all MCP servers` in Claude Code to confirm the server is active |

---

## Going Further

- **Fabric integration:** The same MCP server works against Power BI semantic models published to Microsoft Fabric
- **Remote MCP:** Microsoft has a remote MCP server variant — see `aka.ms/powerbi-modeling-mcp-remote` — that doesn't require a local VS Code install
- **Automation:** You can script `.claude.json` updates with a PowerShell script that auto-detects the latest extension version to avoid manual path maintenance after updates
