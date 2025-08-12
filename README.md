# Schedule Data Generator

**פרויקט סופי – ניהול מערכת שעות לסטודנט**
**השוואה בין אפשרויות מימוש**

Automated pipeline to generate and consolidate course scheduling data. Instead of adding dozens of CSV files by hand, this repo uses two small Python scripts to (1) auto‑create lecture/tutorial/lab CSVs for each course and (2) convert *all* CSVs into a single, clean YAML file you can feed into your project.

> TL;DR: Put `courses.csv` in `data/`, run `generate_course_schedules.py` to create all per‑course CSVs, then run `generate_yaml_data.py` to produce `complete_schedule_data.yaml`.

---

## Why this exists

Manual creation of schedule files is slow and error‑prone. This pipeline:

* **Saves time** by auto‑generating plausible lecture/tutorial/lab options for each course.
* **Centralizes data** into one YAML to simplify loading in your app.
* **Supports comparison** between multiple schedule implementations (השוואה בין אפשרויות מימוש).

---

## Repository structure

```plaintext
.
├── data/
│   ├── courses.csv              # Input: master list of courses (you provide)
│   ├── *_lectures.csv           # Auto-generated (per course)
│   ├── *_tutorials.csv          # Auto-generated (per course)
│   └── *_labs.csv               # Auto-generated (per course)
│   # Optional (if you use named schedules):
│   ├── schedules.csv            # List of scheduleIds
│   └── schedule_<ID>.csv        # Contents per schedule
├── generate_course_schedules.py # Step 1: create *_lectures/tutorials/labs.csv
├── generate_yaml_data.py        # Step 2: aggregate all CSVs → YAML
└── complete_schedule_data.yaml  # Final output (generated)
```

---

## Quick start

### Prerequisites

* Python 3.9+
* Dependencies: `PyYAML`

```bash
pip install pyyaml
```

### 1) Prepare `data/courses.csv`

Create a `data/` folder and add `courses.csv` with **UTF‑8** encoding and headers:

| courseId | name       | credits | examDateA  | examDateB  | lecturer  |
| -------- | ---------- | ------- | ---------- | ---------- | --------- |
| 11002    | Calculus I | 5       | 2025-02-10 | 2025-03-10 | Dr. Smith |

> The generator uses `name` to pick a default building (e.g., courses containing “Physics” → building `P`). It also collects `lecturer` names to assign randomly across groups.

### 2) Generate per‑course CSVs

```bash
python generate_course_schedules.py
```

This will create CSVs like `11002_lectures.csv`, `11002_tutorials.csv`, and `11002_labs.csv` under `data/`, with reasonable day/time patterns per semester and randomized groups/teachers.

### 3) Convert everything to YAML

```bash
python generate_yaml_data.py
```

This scans every `*.csv` in `data/` and writes a single `complete_schedule_data.yaml` with:

* `courses`, `lectures`, `tutorials`, `labs`
* (Optional) `schedules` and `schedule_contents` if you include `schedules.csv` and `schedule_<ID>.csv` files

The converter prints a processing summary and **validates** required fields before saving.

---

## How it works

### `generate_course_schedules.py` (Step 1)

* Reads `data/courses.csv`.
* For each course (grouped by semester), creates *N* alternatives for lectures/tutorials/labs.
* Patterns by semester (days and time windows) emulate realistic timetabling.
* Outputs CSVs with columns: `courseId, day, startTime, duration, classroom, building, teacher, groupId`.

### `generate_yaml_data.py` (Step 2)

* Walks all CSVs in `data/`.
* Builds a unified data structure: `courses`, `lectures`, `tutorials`, `labs`, `schedules`, `schedule_contents`.
* Validates presence of key fields and writes **readable** YAML (`width=80`, sorted keys, Unicode safe).

---

## Example: custom schedules (optional)

If you want named schedules to compare alternatives, add:

**`data/schedules.csv`**

```csv
scheduleId
1
2
```

**`data/schedule_1.csv`**

```csv
scheduleId,courseId,groupId,day,startTime,duration,classroom,building,teacher,type
1,11002,L1,Sunday,08:00,2,Rm1,EM,Dr. Smith,lecture
1,11002,T1,Tuesday,14:00,1,Rm11,M,Dr. Lee,tutorial
```

`generate_yaml_data.py` will include these under `schedules` and `schedule_contents` so your app can render each full option.

---

## Data model (YAML keys)

* **courses**: `{courseId → {courseId, name, credits, examDateA, examDateB, lecturer}}`
* **lectures/tutorials/labs**: `{"<courseId>_<groupId>" → {courseId, day, startTime, duration, classroom, building, teacher, groupId}}`
* **schedules** *(optional)*: `{scheduleId → {scheduleId}}`
* **schedule\_contents** *(optional)*: `{scheduleId → [ {scheduleId, courseId, groupId, day, startTime, duration, classroom, building, teacher, type} ]}`

---

## Rationale (עברית)

במקום ליצור ידנית קבצי CSV רבים לכל קורס ולכל קבוצת לימוד, הסקריפטים האלו מייצרים את הנתונים אוטומטית ומאגדים הכול לקובץ YAML יחיד. כך אפשר להשוות במהירות בין אפשרויות מימוש של **ניהול מערכת שעות לסטודנט**, לעדכן נתונים בבת אחת, ולהזין את הפרויקט ב־dataset עקבי ונקי.

---

## Tips & troubleshooting

* Make sure `data/courses.csv` exists **before** step 1.
* Use **UTF‑8** everywhere (especially Hebrew names).
* If validation fails, the script prints exactly which field is missing.
* Want deterministic output? Set an env var before step 1: `PYTHONHASHSEED=0` and add `random.seed(42)` inside the generator if you need strict reproducibility.

---

## Contributing

PRs welcome. Please run both scripts locally and commit only the YAML and any canonical CSVs you actually use in the app.

## License

MIT

## Authors

**Francis Aboud**
**Bshara Habib**
