# SPEC: Pain Diary (Schmerz-Tagebuch)

> Homework "Dynamic website related to you" – MTPR WS2026
> Author: Christian Pinna · Status: Draft v1 · Date: 2026-11-20

**When to read this file:** before planning or implementing anything for the pain diary page.

---

## 1. Purpose

I work in nursing (Pflege). Pain is assessed several times per shift with the
**NRS** (Numeric Rating Scale: patient rates pain from 0 = no pain to 10 = worst pain imaginable).

On paper it is hard to see at one glance:
- how the pain changed over the day,
- whether a measure (e.g. medication) helped,
- which values are too high.

The page lets **one nurse** enter pain ratings for **one patient** and
generates a table with small coloured bars, so trends and outliers are visible immediately.

It is a **learning project**, not a medical product.

---

## 2. Input data (user input via form)

| Field   | Type (HTML)          | Required | Allowed values                                                              | Default        |
|---------|----------------------|----------|-----------------------------------------------------------------------------|----------------|
| Date    | `<input type=date>`  | yes      | any valid date                                                              | today          |
| Time    | `<input type=time>`  | yes      | `HH:MM`, 00:00–23:59                                                        | empty          |
| NRS     | `<select>`           | yes      | integers 0, 1, 2 … 10                                                       | "– choose –"   |
| Measure | `<select>`           | yes      | `none`, `medication`, `non-drug` (e.g. positioning, cold pack), `other`     | `none`         |
| Note    | `<input type=text>`  | no       | free text, max. 100 characters                                              | empty          |

Example data (for testing, no real patient data):

```
date,time,nrs,measure,note
2026-11-20,06:00,3,none,
2026-11-20,08:30,7,medication,after mobilisation
2026-11-20,09:30,4,none,reassessment
2026-11-20,12:00,2,non-drug,positioning
2026-11-20,15:00,8,medication,
```

---

## 3. Output format

### 3.1 Table (generated with JavaScript)

Columns: **Date | Time | NRS | Bar | Measure | Note | (Delete button)**

- One row per entry, sorted by date + time, **oldest first**.
- **Bar:** a `<div>` whose width is `NRS × 10 %` of the bar cell (NRS 0 → no bar, NRS 10 → full cell).
- **Colour** of the bar:

| NRS     | Colour | Meaning (for this page) |
|---------|--------|-------------------------|
| 0 – 3   | green  | low                     |
| 4 – 6   | orange | medium                  |
| 7 – 10  | red    | high                    |

- **Threshold:** rows with `NRS ≥ THRESHOLD` get a ⚠ symbol in the NRS cell.
  `THRESHOLD` is one constant in `script.js`, value **4**.
  *(Not verified: value taken from my memory of the German "Expertenstandard Schmerzmanagement" (NRS > 3 at rest). Must be checked against the standard of my ward.)*

### 3.2 Summary line (below the table)

`Entries: n · Min: x · Max: y · Average: z`
- Average rounded to **1 decimal** (e.g. `4.8`).

### 3.3 Empty state

If there are no entries, the page shows the text **"No entries yet."** instead of table and summary.

---

## 4. User interface (scribble)

```
+-------------------------------------------------------------+
|  PAIN DIARY                                                 |
|                                                             |
|  Date [2026-11-20]  Time [__:__]  NRS [– choose – v]        |
|  Measure [none v]   Note [______________________]           |
|  [ Add entry ]          <- error messages appear here (red) |
|-------------------------------------------------------------|
| Date       | Time  | NRS  | Bar          | Measure    | Note|  |
| 2026-11-20 | 06:00 |  3   | ###  (green) | none       |     | x|
| 2026-11-20 | 08:30 | 7 ⚠  | #######(red) | medication | ... | x|
| 2026-11-20 | 09:30 | 4 ⚠  | #### (orange)| none       | ... | x|
| 2026-11-20 | 12:00 |  2   | ##   (green) | non-drug   | ... | x|
|-------------------------------------------------------------|
| Entries: 4 · Min: 2 · Max: 7 · Average: 4.0                 |
+-------------------------------------------------------------+
```

---

## 5. Requirements (EARS notation)

**Adding**
- R1: When the user clicks "Add entry" and all required fields are valid, the system shall add the entry to the table and re-sort the table.
- R2: When an entry was added, the system shall clear Time, NRS and Note, and keep Date and Measure.
- R3: When the table changes, the system shall recalculate the summary line.

**Validation**
- R4: If Time or NRS is empty when "Add entry" is clicked, then the system shall not add an entry and shall show "Please fill in time and NRS."
- R5: If an entry with the same date **and** time already exists, then the system shall not add the entry and shall show "An entry for this date and time already exists."
- R6: If the note is longer than 100 characters, then the system shall prevent further typing (`maxlength="100"`).
- R7: When a valid entry is added, the system shall remove any error message.

**Display**
- R8: The system shall colour each bar according to the table in 3.1.
- R9: While `NRS ≥ THRESHOLD`, the system shall show ⚠ next to the NRS value of that row.
- R10: While no entries exist, the system shall show "No entries yet." and hide table and summary.

**Deleting**
- R11: When the user clicks the delete button (x) of a row, the system shall ask "Delete this entry?" and remove the entry only after confirmation.

---

## 6. Non-requirements (explicitly NOT part of v1 / MVP)

- No saving: data is lost on page reload (no file, no `localStorage`, no server).
- No patient name or ID on the page (privacy, GDPR / DSGVO).
- No editing of existing entries (delete + add again instead).
- No line chart, no printing, no export.
- No login, no multiple patients.
- No frameworks or libraries: plain HTML, CSS, JavaScript only.
- No medical advice or dosage logic.

---

## 7. Acceptance tests

| #  | Given                                   | When                                       | Then                                                         |
|----|-----------------------------------------|--------------------------------------------|--------------------------------------------------------------|
| T1 | empty page                              | page loads                                 | "No entries yet." visible, no table                          |
| T2 | empty page                              | add 20.11., 06:00, NRS 3, none             | 1 row, green bar 30 % wide, no ⚠, summary `1 · 3 · 3 · 3.0`  |
| T3 | T2                                      | add 20.11., 08:30, NRS 7, medication       | 2nd row, red bar 70 %, ⚠ shown, average `5.0`                |
| T4 | T3                                      | add 20.11., **07:00**, NRS 4               | new row is placed **between** 06:00 and 08:30, orange bar, ⚠ |
| T5 | any                                     | click "Add entry" with NRS empty           | no new row, error message from R4                            |
| T6 | T3                                      | add again 20.11., 06:00, NRS 5             | no new row, error message from R5                            |
| T7 | T3                                      | click x on 06:00, confirm                  | row removed, summary recalculated                            |
| T8 | T3                                      | click x on 06:00, cancel                   | nothing changes                                              |
| T9 | only one entry                          | delete it and confirm                      | "No entries yet." visible again (R10)                        |
| T10| empty page                              | add NRS 0                                  | row shown, bar width 0 %, summary min 0                      |

---

## 8. DOM elements I expect to use (link to Learn_DOM exercises)

`form`, `input` (date, time, text), `select`/`option`, `button`, `table` (rows created with
`document.createElement("tr")`), `div` (bars, via `style.width` and `style.backgroundColor`).

---

## 9. Open questions (for discussion with buddy / spec-reviewer AI)

- Q1: Is THRESHOLD = 4 correct for my ward? (see 3.1, not verified)
- Q2: Should a reassessment reminder be shown after a "medication" entry (e.g. after 30–60 min)? → not in v1.
- Q3: Should NRS be split into "at rest" and "during movement"? → not in v1.
- Q4: Is "oldest first" the order nurses prefer, or "newest first"?

---

## 10. Review log

| Date | Reviewer | Result |
|------|----------|--------|
|      | buddy    |        |
|      | AI (spec-reviewer.agent.md) |  |
