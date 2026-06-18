# Education Performance — Data Analysis Report

*Prepared for: District Education Leadership · Scope: 10 schools, 67,786 enrollments, 2023 Term 1 – 2024 Term 1*

---

## 1. Project Requirement

The district oversees **10 schools** and roughly **200 students**, and tracks four areas of day‑to‑day operation in separate records: student **attendance**, **assessment** results, **enrollment** against targets, and **teacher lesson delivery**. Because these records lived in disconnected spreadsheets, leadership could not easily answer cross‑cutting questions such as *"which schools are struggling, and in what way?"* or *"do students who attend more actually perform better?"*

This project was commissioned to **bring the four data sources together into a single analytical model and an interactive dashboard** that reports the district's standard performance indicators, makes it easy to compare schools, and surfaces the students and institutions most in need of support — so that decisions for the academic year are based on evidence rather than anecdote.

---

## 2. Questions the Analysis Set Out to Answer

1. **Attendance** — How regularly are students attending, and how many are *chronically* absent (a known early‑warning sign)?
2. **Academic performance** — What are pass rates and average scores, and do they differ by subject, grade, or term? Who are the top performers and who is at risk?
3. **Enrollment** — Are schools meeting their enrollment targets, and is intake gender‑balanced?
4. **Teaching delivery** — Are teachers delivering the lessons they plan, and which schools are falling behind?
5. **Relationships** — Does better attendance translate into better academic results?
6. **Prioritisation** — Taken together, which schools and which areas need the most attention?

---

## 3. Assumptions

The following assumptions underpin how the figures in this report were calculated, and should be kept in mind when interpreting them:

1. **Grading is out of 100.** Every assessment was confirmed to use a maximum score of 100, so all score‑based thresholds below are expressed on that scale.
2. **A "pass"** is any result where the student's score meets or exceeds that assessment's own pass mark — not a single fixed cut‑off applied to every assessment.
3. **Performance thresholds.** "Top performers" score **80 or above**; "at‑risk" students score **below 40**. Score bands are Distinction ≥75, Credit ≥60, Pass ≥50, Near Pass ≥40, Fail <40.
4. **Attendance rate counts only days marked *Present*.** Days marked *Late* and *Absent* both count against it, so the attendance, late, and absence percentages add up to 100%.
5. **"Chronically absent"** means a student missed **more than 10%** of their recorded school days — a per‑student measure, deliberately distinct from the district‑wide absence rate.
6. **Lesson completion** is lessons delivered ÷ lessons planned; schools below **80%** completion are flagged as underperforming.
7. **Gender parity** is the ratio of female to male **enrolled** students, where 1.0 represents perfect balance.
8. **Time is handled differently by area.** Only attendance is recorded against calendar dates; assessments, enrollment, and teaching activity are compared using their own period labels (term, academic year, teaching week). The four terms are treated in chronological order (2023 Term 1 → 2024 Term 1).
9. **Records are linked by school and student identifiers** that are assumed to be consistent and correct across all four datasets.
10. **The source data is taken as complete and accurate as supplied** — no extra cleansing, de‑duplication, or correction was applied beyond combining the datasets.
11. **The report reflects a fixed snapshot** covering 2023 Term 1 through 2024 Term 1.

---

## 4. Analysis Performed (EDA & Data Modelling)

- **Data integration.** The four operational datasets were connected to shared reference tables for **schools**, **students**, and a **calendar**, so that any metric can be sliced consistently by school, grade, gender, subject, or term. School became the common thread linking every dataset.
- **Standardised indicators.** Around **29 reusable performance measures** were defined (attendance rate, absence rate, chronic absenteeism, pass rate, top/at‑risk counts, enrollment achievement, gender parity, lesson completion, and more), so every chart reports numbers the same way.
- **Exploratory data analysis (EDA).** Distributions, ranges, and completeness were reviewed across ~**24,000 assessment records** and ~**36,000 attendance records**. The grading scale was confirmed to be **out of 100** throughout, the four academic terms were verified, and the attendance–performance relationship was tested directly.
- **Interactive dashboard.** A **four‑page report** was built for self‑service exploration:
  1. **Executive Summary** — headline KPIs and a school scorecard
    ![Executive Summary](https://github.com/mchenryspagg/district-education-performance-analysis/blob/main/education%20ex%20summary.png)

  3. **Attendance Deep Dive** — rates, daily/weekday patterns, absence reasons, chronic cases
    ![Attendance Deep dive](https://github.com/mchenryspagg/district-education-performance-analysis/blob/main/attendance%20deepdive.png)

  5. **Academic Performance** — scores and pass rates by subject, grade, and term
    ![Academic Performance](https://github.com/mchenryspagg/district-education-performance-analysis/blob/main/acdemic%20perf.png)

  7. **Enrollment & Teacher Activity** — targets vs. actuals, gender balance, and lesson delivery
   ![Enrollment & Teacher Activity](https://github.com/mchenryspagg/district-education-performance-analysis/blob/main/enrollment%26teacher%20activity.png)
---

## 5. Key Findings & Insights

### Headline indicators

| Area | Indicator | Result |
|---|---|---|
| Attendance | Attendance rate | **82.0%** (10.0% absent, 8.0% late) |
| Attendance | Chronically absent students | **48%** (96 of 200) |
| Academics | Pass rate | **90.4%** (average score 60.8 / 100) |
| Academics | At‑risk students (score < 40) | **9.6%** |
| Enrollment | Target achievement | **84.3%** (12,623 places short) |
| Enrollment | Gender parity (F : M) | **1.02** (balanced) |
| Teaching | Lesson completion | **79.1%** (8 of 10 schools below 80%) |

### What the dashboard shows

1. **Teaching delivery is the weakest area.** Only **79.1%** of planned lessons were delivered, and **8 of the 10 schools fall below the 80% benchmark**. This is the single clearest operational shortfall in the district.

2. **A strong headline attendance rate hides a widespread problem.** Overall attendance looks healthy at **82%**, but nearly **half of all students (48%) are chronically absent** — i.e., they miss more than 10% of their school days. The district‑wide average masks how many individual students are slipping.

3. **Enrollment is consistently below target — across the board.** The district filled only **84.3%** of its target places (a shortfall of **12,623**), and every school sits in a narrow **83–85%** band. The uniformity points to a **system‑wide** cause (target setting, demand, or capacity) rather than a few weak schools. Intake is **gender‑balanced** (parity index 1.02).

4. **Academic results are strong and remarkably consistent.** The pass rate is **90.4%** with an average of **60.8/100**. Performance barely moves across subjects (Mathematics highest at 61.2; Science lowest at 60.3 — under a one‑point spread), across the four terms (essentially flat), or across schools. Still, roughly **1 in 10 students is at risk** (scoring under 40), a group worth targeted support.

5. **Attendance does not predict performance in this data.** Contrary to the usual expectation, students who attend more do **not** score higher here — the relationship is statistically negligible, and chronically‑absent students average essentially the same score (60.9) as their peers (60.6). Performance is being driven by something other than attendance.

---

### 5b. Data Source 
- [Raw Data](Education_Sample_Dataset.xlsx)

---

### 5c. Solution Power BI file 
- [Open Power BI Project](Education_Sample.pbip)

---

## 6. Recommendations

1. **Make lesson delivery the top operational priority.** With 8 of 10 schools under 80% completion, investigate the causes (staffing, timetabling, teacher absence) and set a recovery target. This is the area with both the largest gap and the most direct lever for improvement.
2. **Manage chronic absenteeism at the student level, not the headline level.** The 82% average is reassuring and misleading; track and intervene with the 48% of students who individually exceed the 10% absence threshold.
3. **Review enrollment targets and demand together.** Because the shortfall is uniform across schools, examine whether targets are set too high relative to catchment demand/capacity before treating it as a recruitment failure.
4. **Sustain academic strengths while protecting the at‑risk tail.** Pass rates are healthy; focus targeted academic support on the ~10% scoring below 40 (and watch Science, the lowest‑performing subject) rather than broad interventions.
5. **Look beyond attendance for performance drivers.** Since attendance shows no measurable link to scores here, direct future analysis toward teaching quality, curriculum, or assessment design to explain and lift outcomes.

---

## 7. Conclusion

Consolidating the district's attendance, assessment, enrollment, and teacher‑activity records into one model and dashboard turned scattered spreadsheets into a clear, comparable picture of performance. The data tells a consistent story: **academic outcomes are solid and stable, but operational delivery is lagging** — lessons are under‑delivered, almost half of students are frequently absent, and enrollment is system‑wide below target. Encouragingly, intake is gender‑balanced. The most valuable next steps are operational (lesson delivery and absenteeism) rather than academic, and the dashboard now gives leadership the means to monitor each of these indicators by school, term, and student group on an ongoing basis.

---

> **Data note.** The figures are exact for the dataset analysed, but the distributions are unusually uniform (subjects within ~1 point, schools within ~2 points), indicating this is a **generated sample** intended to demonstrate the reporting approach. The percentages and patterns should be read as illustrative of the method rather than as real‑world effect sizes.
