<div align="center">

<img src="docs/readme/hero.svg" width="100%" alt="Wiecon Time Tracker: a Windows desktop app that turns raw punch-clock rows into monthly hours-worked timesheets, ready to file">

<br><br>

<a href="https://www.dropbox.com/scl/fi/k7jliixl9v4mlwhamy5m1/video-rev6.mp4?rlkey=gnzw56te49ozo5q1dmv385i6t&raw=1"><img src="docs/video/promo_thumbnail_play.jpg" alt="Play the Wiecon Time Tracker promo video" width="840"></a>

<sub>A 120-second tour of what the program does, from raw punch file to filed monthly hours. Click the image to play.<br>Every name and punch in this README is invented — no real employee data appears anywhere.</sub>

</div>

> [!NOTE]
> **This is a showcase repository.** It holds the README, screenshots, diagrams, the user guide and a sample report so you can see what the program does. The source code and prebuilt builds are private.

## About this project

| | |
| :--- | :--- |
| **What it is** | A Windows desktop app that turns a punch clock's raw export into the monthly hours-worked figures Wiecon Consultants Taiwan files with the government |
| **Who uses it** | The office secretary, every month. The monthly report went from 1 h 20 min to 20 min |
| **Timeline** | 14 releases in 8 weeks, from 0.1.2 on 6 August 2026 to 0.9.0 on 1 October 2026 |
| **Scale** | About 3,400 lines of Python in three packages, a three-table SQLite schema, four kinds of PDF report |
| **Stack** | Python 3.13 · NiceGUI · SQLModel over SQLite · pendulum · fpdf2 · LangChain with Gemini · PyInstaller |

Built by [**@ThunderIW**](https://github.com/ThunderIW).

### Engineering highlights

- **Bare timestamps into working days.** The punch clock records a time and nothing else: no in/out flag. The app groups each employee's punches by day and measures from first to last, so a stray midday punch can't split one day into two. [See it on sample data →](#06-example-workflow-getting-monthly-hourly-report)
- **One query layer for everything.** The screen and the AI assistant both read through a single class, `InteractWithDatabase`, so they can never quote different numbers. [Architecture →](#04-architecture)
- **An AI assistant that can't invent figures.** A supervisor agent hands each question to two tool-calling specialists, whose prompts forbid stating any number they haven't fetched through a read-only database tool. [AI assistant →](#ai-assistant)
- **Integrity enforced by the database.** Foreign keys are switched on for every SQLite connection, so punches for unknown badge numbers are rejected and removing an employee cascades to their punches.
- **Shipped like a product.** A single Windows executable built with PyInstaller, a release-notes PDF with every version, a step-by-step user guide and a two-minute promo video.

<br>

<div align="center">

**User guide:** **[PDF](docs/user-guide/Wiecon%20Time%20Tracker%20-%20User%20Guide.pdf)** &nbsp;·&nbsp; **[HTML file](docs/user-guide/Wiecon%20Time%20Tracker%20-%20User%20Guide.html)**

<sub>A step-by-step guide for the people using the app: numbered screenshots for every screen, from uploading the punch file to downloading the timesheets.<br>The <b>PDF</b> opens right here. GitHub can't preview the <b>HTML file</b>; to use it offline, click the download button next to <i>Raw</i> and open it in Edge or Chrome.</sub>

</div>


<br>

| # | Section | What's in it |
| :---: | --- | --- |
| 01 | [Why this exists](#01-why-this-exists) | The two-program manual routine this replaces, and the figure it produces |
| 02 | [Testimonials](#02-testimonials) | What the people using it every month say |
| 03 | [Features](#03-features) | The six things the program actually gives you |
| 04 | [Architecture](#04-architecture) | Front end, back end, the three tables, and what is derived rather than stored |
| 05 | [Design decisions and lessons](#05-design-decisions-and-lessons) | What went wrong in real use, and how the design changed because of it |
| 06 | [Example workflow (Getting monthly hourly report)](#06-example-workflow-getting-monthly-hourly-report) | A complete run on invented data, from uploaded CSV to the filed total |
| 07 | [Scope and known issues](#07-scope-and-known-issues) | What it doesn't do, and three behaviours worth knowing |
| 08 | [Release history](#08-release-history) | 14 releases in eight weeks, newest first |
| 09 | [Copyright and license](#09-copyright-and-license) | Proprietary software |

<br>

---

## 01. Why this exists

<div align="center">
<img src="docs/readme/impact.svg" width="100%" alt="Monthly reporting took 1 hour 20 minutes across two programs; it now takes 20 minutes in one: 75% less time, four times faster">
</div>

In the current setup, employees check in and check out using a punch system that lets them use either their fingerprint or a code. Before this program, the employee in charge of keeping tabs on this, the secretary, had to perform the following steps every month:

1. Download the plaintext file that holds all the punch entries.
2. Upload it to another piece of software to extract the required information: the total number of hours each employee worked that month, as required by the government.

As a result of introducing this system, the task now takes the secretary 20 minutes instead of 1 hour and 20 minutes. That cuts the time spent by 75%, making the job four times faster.

<div align="center">
<img src="docs/diagrams/01-before-after.svg" alt="The manual routine before, and the same job now" width="900">
</div>

---

## 02. Testimonials

![From the people using it](https://img.shields.io/badge/from-the_people_using_it-16235c?style=flat-square)

> *"Monthly hours used to be a chore. Now they’re done before my coffee gets cold."*
>
> — Office secretary, Wiecon

---

## 03. Features
Six things the program does:

**1.** ![Bulk CSV / Plain-Text Import](https://img.shields.io/badge/Bulk%20CSV%20%2F%20Plain--Text%20Import-b45309?style=flat-square)<br>
Several punch files can be uploaded at once. Bad rows — entries for people who don't work for the company — are reported back to you and left out, while everything else is stored.

**2.** ![Bare timestamps into usable data](https://img.shields.io/badge/Bare%20timestamps%20into%20usable%20data-2f6fd0?style=flat-square)<br>
The punch system records only a time, with no indication of a check-in or a check-out. The program resolves each day into a date, a start time and an end time, so it is clear when an employee began and finished work.

**3.** ![Problems flagged on sight](https://img.shields.io/badge/Problems%20flagged%20on%20sight-dc2626?style=flat-square)<br>
Rows with a missed punch-out are coloured red, short days under 8 hours are coloured amber, and everything else stays white.

**4.** ![Four PDF exports](https://img.shields.io/badge/Four%20PDF%20exports-0f766e?style=flat-square)<br>
A full monthly timesheet for all employees, a single employee's selected days, a zero-hours sheet to be signed by hand, and a timesheet for an individual employee for a selection of months and year with an added summary page.

**5.** ![AI Agent](https://img.shields.io/badge/AI%20Agent-7c3aed?style=flat-square)<br>
A multi-agent AI assistant (LangChain + Google Gemini) that answers natural-language questions about employee attendance. A supervisor agent delegates each question to specialist agents, which autonomously select and invoke database query tools — ask it about an employee's details, the hours they worked on a given day, their total hours for a month, or who worked zero hours.

**6.** ![Deletion of data](https://img.shields.io/badge/Deletion%20of%20data-16235c?style=flat-square)<br>
A UI interface that lets users delete unwanted attendance data or resigned employees from the system, with a confirmation pop-up to prevent accidental or unwanted deletion.

---

## 04. Architecture

![NiceGUI](https://img.shields.io/badge/front_end-NiceGUI-2f6fd0?style=flat-square) ![SQLModel](https://img.shields.io/badge/ORM-SQLModel-16235c?style=flat-square) ![SQLite](https://img.shields.io/badge/store-SQLite-0b1437?style=flat-square) ![LangChain](https://img.shields.io/badge/assistant-LangChain-7c3aed?style=flat-square) [![fpdf2](https://img.shields.io/badge/reports-fpdf2_and_pypdf-0f766e?style=flat-square)](https://py-pdf.github.io/fpdf2/)

A NiceGUI front end over a SQLite database accessed through SQLModel ([SQLAlchemy](https://www.sqlalchemy.org/)), on Python 3.13.

<div align="center">
<img src="docs/diagrams/06-techstack.png" alt="Tech stack: NiceGUI UI, pywebview native window, SQLModel over SQLite, fpdf2 PDF export, pendulum dates, PyInstaller packaging, uv for env and deps, Python 3.13 or newer." width="880">
</div>

<div align="center">
<img src="docs/diagrams/02-architecture.svg" alt="Architecture: what you touch, what it does, where it lives" width="880">
</div>

Slanted boxes are files entering or leaving. The hexagon is the single query layer — nothing reaches the database except through it, which is why the screen and the assistant can never quote different numbers.

### Front end

![NiceGUI · one page · no routing](https://img.shields.io/badge/NiceGUI%20%C2%B7%20one%20page%20%C2%B7%20no%20routing-2f6fd0?style=flat-square)

NiceGUI 3.15. The app uses no routing: rather than declaring pages with `@ui.page`, `main.py` passes a single function to `ui.run(root=index)`, and `index()` builds the entire interface — every other screen is a dialog opened from the header or the Tools tab.

The parts that change are `@ui.refreshable` functions, re-rendered on demand instead of by reloading the page. They cover whether the assistant's API key is set, the year and month navigation buttons so only months with data are offered, the metric cards showing the selected employee's average and total hours, and the attendance table itself, rendered with [AG Grid](https://www.ag-grid.com/) via `ui.aggrid` for the chosen employee and month. The yearly chart is a [pyecharts](https://pyecharts.org/) bar chart passed through `ui.echart.from_pyecharts`.

Run normally it serves in the browser over [uvicorn](https://www.uvicorn.org/); the packaged build wraps the same page in a [pywebview](https://pywebview.flowrl.com/) desktop window.

### Back end

![SQLite · SQLModel · one query layer](https://img.shields.io/badge/SQLite%20%C2%B7%20SQLModel%20%C2%B7%20one%20query%20layer-16235c?style=flat-square)

The backend uses SQLite, with SQLModel providing the schema definition and query interface. Tables are declared as SQLModel classes in `Database/create_database.py` and initialized on import, so no migration step is required. Every query the program runs goes through a single dedicated Python class, `InteractWithDatabase` in `Database/QueryDatabase.py`, and both the UI and the AI assistant share it. This eliminates any chance of the two diverging — they are always reading the same numbers from the same database.


Each row's date and time are read from the uploaded plain text and parsed with [pendulum](https://pendulum.eustace.io/), so `05/01/26 08:57:00` becomes `05 January 2026` and `08:57 AM`, both on screen and in the exported PDFs.


<div align="center">
<img src="docs/diagrams/03-schema.svg" alt="Database schema: employees, punches, cached monthly hours" width="600">
</div>

> [!NOTE]
> `PRAGMA foreign_keys=ON` is set on every connection. SQLite leaves foreign keys unenforced by default, and both the cascade from an employee to their punches and the rejection of punches for unknown badge numbers depend on it.

### AI assistant

![LangChain · Gemini · supervisor and two specialists](https://img.shields.io/badge/LangChain%20%C2%B7%20Gemini%20%C2%B7%20supervisor%20%2B%20two%20specialists-7c3aed?style=flat-square)

A multi-agent system built with [LangChain](https://www.langchain.com/)'s `create_agent` and [Google's Gemini](https://ai.google.dev/gemini-api/docs). It answers natural-language questions about employee attendance using a supervisor and two specialist agents, all running on the same model:

<div align="center">
<img src="docs/diagrams/09-ai-assistant.drawio.svg" alt="You talk to the supervisor, which delegates to Attendance_agent and hours_agent; both reach attendance.db only through InteractWithDatabase" width="760">
</div>

<sub>Editable in draw.io: open <code>docs/diagrams/09-ai-assistant.drawio.svg</code> at <a href="https://app.diagrams.net">app.diagrams.net</a>.</sub>

- **The supervisor** talks to you. It never answers from its own knowledge: it turns your question into self-contained requests, sends them to one or both specialists, and combines what they report.
- **The specialists** are tool-calling agents. Each one autonomously selects and invokes its database query tools, reads the results, and reports the facts back. Their prompts forbid stating any figure they haven't retrieved through a tool, and require them to report "no data" rather than zero hours.

| Agent | Tool | What it returns |
| --- | --- | --- |
| Both | `find_employee` | Full employee names containing the text given, so "amy" resolves to "林佳蓉 Amy" |
| `Attendance_agent` | `get_person` | An employee's badge number and their per-day start and end times |
| `Attendance_agent` | `get_hours_worked_on_particular_day` | Start time, end time and hours worked for one employee on one date, given in any common date format |
| `hours_agent` | `get_total_hours_worked_month_and_year` | An employee's total hours for a given month and year, summed live from their punches |
| `hours_agent` | `get_zero_hours_employees` | Every day in a month where an employee's hours come out at zero |
| `hours_agent` | `get_available_years` | The years that have calculated monthly hours on record |

Only the supervisor's reply is streamed into the chat. Each message is answered on its own; the assistant doesn't carry earlier messages forward, and every tool is read-only.


<div align="center">
<img src="docs/screenshots/06_ai_assistant.jpg" alt="The AI assistant chat window inside the app" width="760">
</div>




### Project layout

![3,400 lines · three packages](https://img.shields.io/badge/3%2C400%20lines%20%C2%B7%20three%20packages-0f766e?style=flat-square)

About 3,400 lines of Python across three packages. Two thirds of that is `main.py`, because the entire interface is built in one function.

Four modules, and everything funnels through one of them:

<div align="center">
<img src="docs/diagrams/04-modules.svg" alt="Module dependency graph" width="760">
</div>

<table>
<tr><th align="left">Module</th><th align="right">Lines</th><th align="left">Share of the Python in this repo</th></tr>
<tr><td><code>main.py</code></td><td align="right">2334</td><td><code>████████████████████</code></td></tr>
<tr><td><code>Database/QueryDatabase.py</code></td><td align="right">573</td><td><code>█████</code></td></tr>
<tr><td><code>Chat_bot/bot_setup.py</code></td><td align="right">420</td><td><code>████</code></td></tr>
<tr><td><code>Database/create_database.py</code></td><td align="right">64</td><td><code>█</code></td></tr>
</table>

The full tree, including what is not code:

<img src="docs/diagrams/07-tree.svg" alt="The project file tree, including what is not code" width="760">

Where to start reading, depending on what you are looking for:

| To understand | Read | Start at |
| --- | --- | --- |
| The whole application | `main.py` | `index()` — it builds every screen in one pass |
| How hours are derived from punches | `Database/QueryDatabase.py` | `group_punches_by_day` |
| The schema | `Database/create_database.py` | the three SQLModel classes |
| What the assistant can and cannot do | `Chat_bot/bot_setup.py` | `SUPERVISOR_PROMPT` and the `agent` class, then the two specialist agents and their `@tool` functions |
| How a release is cut | `release/build_release_notes.py` | `VERSION` and `CHANGELOG` at the top |

---

## 05. Design decisions and lessons

![From real use](https://img.shields.io/badge/from-real_use-16235c?style=flat-square)

Most of the design came from problems found in real use. These are the changes that shaped it most:

| Problem in real use | Decision | Why |
| --- | --- | --- |
| Importing a whole year of punch files at once crashed the app (0.7.0). | Limit uploads to three files at a time. | A clear limit is better than a crash halfway through an import. |
| Building a large timesheet PDF for every employee froze the app (0.1.6). | Build the PDF off the main thread, with a loading indicator. | The window stays responsive while a big report is made. |
| Hours had to be recalculated by hand after every upload (0.1.3). | Recalculate on every upload; keep the manual recompute only for restores and deletions. | The figures on screen are never stale after an import. |
| The zero-hours sheet put afternoon punches under Start Time, so a forgotten punch-in looked like a forgotten punch-out (0.6.0). | Write an afternoon punch under End Time and leave Start Time blank. | The secretary has to know which punch to ask the employee about. |
| The AI assistant failed silently without a key or internet, and the key had to be typed into `.env` by hand (0.5.0). | Set the key from inside the app, and say plainly when a request fails. | The person using it isn't a developer and has to see what went wrong. |
| Removing an employee or a month of data can't be undone (0.8.0). | One shared confirmation dialog that names exactly what will be removed. | Every destructive action asks the same way, so nothing is deleted by accident. |

---

## 06. Example workflow (Getting monthly hourly report)

![Invented data](https://img.shields.io/badge/data-invented-1f8a70?style=flat-square) ![Verified](https://img.shields.io/badge/output-measured_from_a_real_run-16235c?style=flat-square)

A complete run on invented data, from an uploaded punch file to the total the firm files.

<div align="center">
<img src="docs/diagrams/08-example-workflow.svg" alt="The four steps of the example workflow: add employees, drop the punch file, read the month, export the timesheet" width="880">
<br><sub>The whole run at a glance. Each step is walked through below.</sub>
</div>

<details>
<summary><b>Watch the whole run, animated</b> (a 26-second loop of the four steps below)</summary>
<br>

<div align="center">
<img src="docs/diagrams/example-workflow-animated.svg" alt="Animated walk-through of the four steps: add the employees, drop the punch file, read the month, then export the monthly timesheet PDF" width="780">
</div>

</details>

### Step one: add the employees

Through <kbd>Employees</kbd> → <kbd>Employee Tools</kbd> → <kbd>Manage</kbd> → <kbd>Add Employee</kbd>. Employees must exist before any of their punches will import, so this comes first.

| ID Number   | English Name    | Chinese Name       | 
|-------------|-----------------|--------------------|
| 20401       | Alice Bennett   | 王雅婷               |
| 20402       | Marco Reyes     | 李承翰             |
| 20403       | Ingrid Halvorsen |                    |

The Chinese name is optional. The software was built for an international company in Taiwan, where some employees go by a Chinese name only and others by an English one.

### Step two: drop the punch file

Through <kbd>Timesheet</kbd> → <kbd>Upload Timesheet</kbd>. 
The file itself has no header row; 
the columns below are labeled for readability only.

| ID Number | Date | Time | Status |
| --- | :---: | :---: | :---: |
| 020401 | 05/01/26 | 08:57:00 | 00 |
| 020401 | 05/01/26 | 17:32:00 | 00 |
| 020402 | 05/01/26 | 09:14:00 | 00 |
| 020402 | 05/01/26 | 18:03:00 | 00 |
| 020403 | 05/01/26 | 08:45:00 | 00 |
| 020403 | 05/01/26 | 17:10:00 | 00 |
| 020401 | 06/01/26 | 08:59:00 | 00 |
| 020401 | 06/01/26 | 17:28:00 | 00 |
| 020402 | 06/01/26 | 09:02:00 | 00 |
| 020402 | 06/01/26 | 12:30:00 | 00 |
| 020402 | 06/01/26 | 18:11:00 | 00 |
| 020404 | 06/01/26 | 09:00:00 | 00 |

The real file is **plain text** with one row per punch. Nothing in it says which punch is an arrival and which a departure, or groups an employee's punches by day; working that out is the program's job.




<div align="center">
<img src="docs/diagrams/05-import-flow.svg" alt="What happens to each row of an uploaded punch file" width="820">
</div>

Eleven rows import. Badge 20404 belongs to nobody, so its punch is dropped and reported as a skipped ID. The malformed last row is counted separately. The import then recalculates hours for the three employees it touched and moves the date picker to 6 January 2026, the newest punch it saw.

### Step three: read the month

| Employee | Date | Start | End | Hours |
| --- | --- | --- | --- | --- |
| Alice Bennett | 05 January 2026 | 08:57 AM | 05:32 PM | 8.58 |
| Alice Bennett | 06 January 2026 | 08:59 AM | 05:28 PM | 8.48 |
| Marco Reyes | 05 January 2026 | 09:14 AM | 06:03 PM | 8.82 |
| Marco Reyes | 06 January 2026 | 09:02 AM | 06:11 PM | 9.15 |
| Ingrid Halvorsen | 05 January 2026 | 08:45 AM | 05:10 PM | 8.42 |

Marco punched three times on 6 January. The middle punch at 12:30 is discarded, and the day is measured from first to last, giving 9.15 hours rather than two separate spans. Alice's stored monthly average comes out at 8.53 hours.

The month's average and total sit under the table, with the year's totals charted beneath them:

<div align="center">
<img src="docs/screenshots/02_hours_chart.jpg" alt="Monthly average and total, and the yearly chart" width="720">
<br><sub>The total is summed from punches every time it is displayed, not read from the cache.</sub>
</div>

Selecting rows opens them for review and export. A day with one punch shows plainly as zero hours:

<div align="center">
<img src="docs/screenshots/03_export_selection.jpg" alt="Selected days, with print and PDF export" width="620">
</div>

### Step four: export the timesheet

<kbd>Employees</kbd> → <kbd>Employee Tools</kbd> → <kbd>Download</kbd> → <kbd>Timesheet</kbd>, then <kbd>Download timesheet for 2026 January</kbd>, builds the whole month, one A4 landscape page per employee, merged into a single file named `Employee timesheet 2026_January.pdf`:

<div align="center">
<img src="docs/screenshots/15_download_timesheet.jpg" alt="Download Timesheet dialog, under Employee Tools" width="620">
</div>

Each page repeats the table above for one person and closes with their average and total:

<div align="center">

**Attendance Report - 王雅婷 Alice Bennett**

| Date | Start Time | End Time | Hours Worked |
| :--- | :---: | :---: | ---: |
| 05 January 2026 | 08:57 AM | 05:32 PM | 8.58 |
| 06 January 2026 | 08:59 AM | 05:28 PM | 8.48 |
| **Average Hours Worked** | | | **8.53 h** |
| **Total Hours Worked** | | | **17.06 h** |

<sub>The real file: **[attendance-report-alice-bennett.pdf](docs/samples/attendance-report-alice-bennett.pdf)** — produced by the run above, not transcribed.</sub>

</div>

> [!TIP]
> That total is the figure the firm files.


---

## 07. Scope and known issues

### What it doesn't do

| Not included | Status                                                                                                                             |
| --- |------------------------------------------------------------------------------------------------------------------------------------|
| Payroll calculation | Out of scope                                                                                                                       |
| Leave and vacation tracking | Out of scope                                                                                                                       |
| Editing records after import | Out of scope                                                                                                                       |
| Any form of login | Not implemented — anyone who opens the app sees every employee's records under <br/>the idea that only one person is using the software |



### Known issues

**Employees must exist before their punches will import.** A punch whose badge number has no matching employee violates a foreign key and is dropped. The upload notification lists the skipped IDs, which is easy to miss on a large file.

**There is no bulk employee import in the UI.** A function that reads an employee CSV exists in the database layer, but nothing calls it. Employees are added one at a time through the Add tab, so a fresh database means entering everyone by hand before the first import will do anything.

**Re-uploading the same file duplicates rows.** There is no unique constraint on a punch, and nothing checks whether one is already stored. Importing the same file twice was measured to take the table from 6 rows to 12. Derived hours are unaffected, since a day is reduced to its first and last punch, but the table grows on every re-import.

---

## 08. Release history

![Latest 0.9.0](https://img.shields.io/badge/latest-0.9.0-16235c?style=flat-square) ![14 releases](https://img.shields.io/badge/releases-14-2f6fd0?style=flat-square) ![Notes PDF per release](https://img.shields.io/badge/every_release_ships-a_notes_PDF-0f766e?style=flat-square)

14 releases between 6 August and 1 October 2026, newest first.

| Version | Released | Headline |
| :--- | :--- | :--- |
| **0.9.0** | 1 Oct 2026 | Timesheet and Employees menus in the header |
| **0.8.1** | 26 Aug 2026 | Long multi-month reports and clearer file names |
| **0.8.0** | 24 Aug 2026 | Removing an employee, with a confirmation for every destructive action |
| **0.7.0** | 21 Aug 2026 | One employee across as many months as you like |
| **0.6.0** | 20 Aug 2026 | Both timesheet downloads moved into one dialog |
| **0.5.0** | 19 Aug 2026 | The assistant's API key, set from inside the app |
| **0.4.0** | 13 Aug 2026 | The zero-hours sheet for missed punch-outs |
| **0.3.0** | 13 Aug 2026 | Employees can be added without an external SQLite tool |
| **0.2.0** | 11 Aug 2026 | The AI assistant arrives |
| **0.1.6 and earlier** | 6–10 Aug 2026 | Five maintenance releases |

<details>
<summary><b>Full release notes</b>: open for every version's details</summary>

<br>

<details>
<summary><b>0.9.0</b> &nbsp;·&nbsp; 1 October 2026 &nbsp;·&nbsp; <i>Timesheet and Employees menus in the header</i></summary>

<br>

![New menu bar](https://img.shields.io/badge/feature-new_menu_bar-2f6fd0?style=flat-square) ![Database: keep](https://img.shields.io/badge/database-keep-15803d?style=flat-square)

![New](https://img.shields.io/badge/new-2f6fd0?style=flat-square)
- The blue header has two drop-down buttons: **Timesheet** (Upload Timesheet, Delete Timesheet) and **Employees** (Employee Tools). The everyday tasks are one click from the main screen instead of being buried inside Settings.
- The AI assistant runs as a small team of specialised agents behind one supervisor, and can also list every employee on record.

![Changed](https://img.shields.io/badge/changed-b45309?style=flat-square)
- Uploading moved out of Settings into its own dialog, <kbd>Timesheet</kbd> → <kbd>Upload Timesheet</kbd>. It accepts `.csv` attendance sheets, up to three at a time, with a spinner while they are processed.
- Deleting a month of attendance moved to <kbd>Timesheet</kbd> → <kbd>Delete Timesheet</kbd>. The red button names exactly what will be removed before the usual *"This cannot be undone"* confirmation.
- Settings holds only the two occasional tools, Calculate hours and AI setup, and opens straight onto them.
- The Download sub-tabs in Employee Tools are labelled **Timesheet**, **Zero Hours** and **Individual Employee**, and every panel uses the same blue title bar.
- The unused `Attendance/connect.py` was removed.

![Fixed](https://img.shields.io/badge/fixed-0f766e?style=flat-square)
- Dialogs that could only be dismissed by clicking outside them have a proper Close button.
- Valid files were being rejected by the upload with a message asking for `.csv` files.

<div align="center">
<img src="docs/screenshots/16_timesheet_menu.jpg" alt="The Timesheet menu open in the header, with Upload Timesheet and Delete Timesheet" width="620">
<br><sub>The Timesheet menu in the new header, next to Employees.</sub>
</div>

</details>

<details>
<summary><b>0.8.1</b> &nbsp;·&nbsp; 26 August 2026 &nbsp;·&nbsp; <i>long multi-month reports and clearer file names</i></summary>

<br>

![Report fixes](https://img.shields.io/badge/fix-individual_employee_report-dc2626?style=flat-square) ![Database: keep](https://img.shields.io/badge/database-keep-15803d?style=flat-square)

![Changed](https://img.shields.io/badge/changed-b45309?style=flat-square)
- The individual employee report is named after what it holds: the employee, the year and the months chosen, or *full year* when all twelve are selected.

![Fixed](https://img.shields.io/badge/fixed-0f766e?style=flat-square)
- A report covering many months ran its list of month names off the page instead of wrapping onto a new line.

</details>

<details>
<summary><b>0.8.0</b> &nbsp;·&nbsp; 24 August 2026 &nbsp;·&nbsp; <i>removing an employee, with a confirmation for every destructive action</i></summary>

<br>

![Remove Employee, with confirmation](https://img.shields.io/badge/feature-remove_employee-dc2626?style=flat-square) ![Database: keep](https://img.shields.io/badge/database-keep-15803d?style=flat-square)

![New](https://img.shields.io/badge/new-2f6fd0?style=flat-square)
- A **Remove Employee** sub-tab sits alongside View and Add, grouped under a **Manage** tab inside Employee Tools (Download stays a tab of its own). Pick an employee and the button says exactly what it will do — *"Remove employee (name) from system"* — before you press it.
- Removing an employee asks for confirmation first: a dialog names the employee and warns **"This cannot be undone"**. The same dialog is reused everywhere the app removes data, so every destructive action asks the same way.
- Once confirmed, the employee is removed together with their attendance data, and the employee list refreshes everywhere it appears — the main picker, View Employee, and the Download sub-tabs.

![Changed](https://img.shields.io/badge/changed-b45309?style=flat-square)
- The **Delete data** tool (Settings, Tools tab) for clearing a whole month/year of attendance is back, with the same blue header and confirmation dialog as the rest of the app.
- Employee Tools has two tabs, **Manage** (View, Add and Remove Employee as a segmented control) and **Download**, instead of three side by side. A shared header component draws the blue title bar on every panel in Employee Tools and Settings.

<div align="center">
<img src="docs/screenshots/07_remove_employee.jpg" alt="Remove Employee sub-tab, picking who to remove" width="620">
<br><sub>Manage tab, Remove Employee. The button names the employee before you press it.</sub>
</div>

<div align="center">
<img src="docs/screenshots/08_remove_confirm.jpg" alt="Confirmation dialog before an employee is removed" width="620">
<br><sub>The same confirmation dialog every destructive action in the app now uses.</sub>
</div>

</details>

<details>
<summary><b>0.7.0</b> &nbsp;·&nbsp; 21 August 2026 &nbsp;·&nbsp; <i>one employee across as many months as you like</i></summary>

<br>

![Per-employee multi-month report](https://img.shields.io/badge/feature-per--employee_multi--month_report-b45309?style=flat-square) ![Database: keep](https://img.shields.io/badge/database-keep-15803d?style=flat-square)

![New](https://img.shields.io/badge/new-2f6fd0?style=flat-square)
- A **Download individual Employee** sub-tab at <kbd>Settings</kbd> → <kbd>Tools</kbd> → <kbd>Employee Tools</kbd> → <kbd>Download</kbd>. Select an employee and any months of the chosen year, and the app builds a single PDF of just those months. Previously the only per-employee export was the one month on the main screen.
- Each month gets its own page listing every logged day with its date, start time, end time and hours worked, ending with the number of recorded days, total hours and that month's average.
- A summary page closes the report: a bar chart of hours by month, a table of each month's days and hours with a Total row, the total across the full period, and the employee, year and date the report was created.

![Changed](https://img.shields.io/badge/changed-b45309?style=flat-square)
- The download button says what it will create — *"Download individual data for (employee) - January, February"* — and stays hidden until an employee and at least one month are chosen.
- The yearly chart on the main screen shows each month's **average** hours alongside its total.
- Uploads are limited to **three files at a time**; importing a whole year of punch files at once was found to crash the app.

![Fixed](https://img.shields.io/badge/fixed-0f766e?style=flat-square)
- Adding an employee reports the right result: an already-registered badge number is rejected, each case has its own notification, and a successful addition is confirmed instead of happening silently.
- Newly created employees appear in the employee picker straight away.

<div align="center">
<img src="docs/screenshots/09_individual_download.jpg" alt="Download individual Employee, months selected" width="620">
<br><sub>One employee, any combination of months, in a single PDF.</sub>
</div>

</details>

<details>
<summary><b>0.6.0</b> &nbsp;·&nbsp; 20 August 2026 &nbsp;·&nbsp; <i>both timesheet downloads moved into one dialog</i></summary>

<br>

![Timesheet downloads consolidated](https://img.shields.io/badge/change-timesheet_downloads_consolidated-2f6fd0?style=flat-square) ![Database: keep](https://img.shields.io/badge/database-keep-15803d?style=flat-square)

![Changed](https://img.shields.io/badge/changed-b45309?style=flat-square)
- The two timesheet downloads live in one place: <kbd>Employee Tools</kbd> → <kbd>Download</kbd>, with **Download Timesheet** (the monthly sheet for every employee) and **Download Timesheet with zero hours** as sub-tabs. The separate *Timesheets* card on the Tools tab was removed.
- The zero-hours panel names the month and year taken from the main screen, and hides the download button when that month has nothing to report.
- Settings uses the same blue title bar as every other dialog, the attendance summary popup was widened so long dates and times no longer wrap, and Employee Tools has a single Close button shared by all its tabs.
- The *Delete Data* tool was removed (it returned in 0.8.0).

![Fixed](https://img.shields.io/badge/fixed-0f766e?style=flat-square)
- The zero-hours PDF put afternoon punches in the wrong column. An afternoon punch is now written under End Time with Start Time left blank, so someone who forgot to punch **in** no longer looks like someone who forgot to punch **out**.

<div align="center">
<img src="docs/screenshots/11_download_tabs.jpg" alt="Download Timesheet and Download Timesheet with zero hours as sub-tabs" width="620">
<br><sub>Both timesheet downloads now live in one dialog.</sub>
</div>

</details>

<details>
<summary><b>0.5.0</b> &nbsp;·&nbsp; 19 August 2026 &nbsp;·&nbsp; <i>the assistant's API key, set from inside the app</i></summary>

<br>

![API key set from inside the app](https://img.shields.io/badge/feature-API_key_set_in--app-7c3aed?style=flat-square) ![Database: keep](https://img.shields.io/badge/database-keep-15803d?style=flat-square)

![New](https://img.shields.io/badge/new-2f6fd0?style=flat-square)
- The assistant's Google API key can be set from <kbd>Settings</kbd> → <kbd>Tools</kbd> → <kbd>AI setup</kbd>. It takes effect immediately with no restart, is remembered next time, is masked as you type, and the dialog shows whether a key is already set. Previously it had to be written into `.env` by hand.

![Changed](https://img.shields.io/badge/changed-b45309?style=flat-square)
- The message box and send button are disabled while a reply is in flight, so a second question cannot be sent on top of the first.
- The main screen was reorganised: date and employee pickers side by side under a titled header, the two "currently selected" lines condensed into one, and the month's average and total shown as figures rather than a sentence. Every Settings dialog shares the same header and spacing.

![Fixed](https://img.shields.io/badge/fixed-0f766e?style=flat-square)
- The assistant no longer fails silently: with no key set it says so and points at AI setup, and a request that fails part way — no internet, an expired key — shows a short apology instead of a loading animation that never ends.

<div align="center">
<img src="docs/screenshots/13_ai_setup.jpg" alt="AI setup dialog, API key entry" width="620">
<br><sub>The assistant's Google API key, set from inside the app.</sub>
</div>

</details>

<details>
<summary><b>0.4.0</b> &nbsp;·&nbsp; 13 August 2026 &nbsp;·&nbsp; <i>the zero-hours sheet for missed punch-outs</i></summary>

<br>

![Zero-hours sheet](https://img.shields.io/badge/feature-zero--hours_sheet-dc2626?style=flat-square)

![New](https://img.shields.io/badge/new-2f6fd0?style=flat-square)
- A zero-hours attendance sheet under <kbd>Employee Tools</kbd> → <kbd>Download</kbd>: a PDF of every employee who punched in on a day but recorded no hours, normally a forgotten punch-out. Each row has the name, the date and the recorded punch, with blank End Time and Signature columns to fill in and sign by hand.
- The sheet covers the month and year selected on the main screen, and says so when that month has no such days instead of producing an empty file.

![Changed](https://img.shields.io/badge/changed-b45309?style=flat-square)
- Employee Tools has three evenly spaced tabs — View, Add and Download — each with an icon, and no longer changes width when switching between them.

</details>

<details>
<summary><b>0.3.0</b> &nbsp;·&nbsp; 13 August 2026 &nbsp;·&nbsp; <i>employees can be added without an external SQLite tool</i></summary>

<br>

![Add employees in-app](https://img.shields.io/badge/feature-add_employees_in--app-b45309?style=flat-square)

![New](https://img.shields.io/badge/new-2f6fd0?style=flat-square)
- Employees can be added from inside the app: <kbd>Employee Tools</kbd> → <kbd>Add</kbd>, with an English name, an employee ID and optionally a Chinese name, and are selectable on the main screen straight away. Previously they had to be added to `attendance.db` by hand.

![Changed](https://img.shields.io/badge/changed-b45309?style=flat-square)
- The year chart shows the **total** hours worked in each month rather than the monthly average, calculated straight from the punch records.

![Fixed](https://img.shields.io/badge/fixed-0f766e?style=flat-square)
- The year chart showed "No data available" for an employee with punches that year, its legend overlapped the rotated month names, and a duplicate "No data available" pop-up reappeared on every refresh.

</details>

<details>
<summary><b>0.2.0</b> &nbsp;·&nbsp; 11 August 2026 &nbsp;·&nbsp; <i>the AI assistant arrives</i></summary>

<br>

![AI assistant](https://img.shields.io/badge/feature-AI_assistant-7c3aed?style=flat-square)

![New](https://img.shields.io/badge/new-2f6fd0?style=flat-square)
- The AI assistant, on the chat icon at the bottom right of the main screen. Ask it *"How many hours did [employee] work on [date]?"* or *"What's the total hours [employee] worked in [month] [year]?"* and it looks up the real attendance data.
- The assistant dialog can be resized by dragging its bottom-right corner, and has a "Delete all chats" button to clear the conversation.

![Fixed](https://img.shields.io/badge/fixed-0f766e?style=flat-square)
- A month-name mismatch that made month/year totals silently return 0, a database lookup with its arguments swapped that broke single-day lookups, and "Delete all chats" not updating when a message was sent with <kbd>Enter</kbd>.

<div align="center">
<img src="docs/screenshots/14_ai_assistant_arrives.jpg" alt="The AI assistant chat dialog, its first appearance" width="620">
<br><sub>The chat icon at the bottom right, and the assistant it opens.</sub>
</div>

</details>

<details>
<summary><b>0.1.6 and earlier</b> &nbsp;·&nbsp; 6–10 August 2026 &nbsp;·&nbsp; <i>five maintenance releases</i></summary>

<br>

| Version | Date | What changed |
| :--- | :--- | :--- |
| **0.1.6** | 10 Aug 2026 | ![Fixed](https://img.shields.io/badge/fixed-0f766e?style=flat-square) The timesheet dialog's year and month pickers no longer read *"...you want to delete"*, left over from the Delete Data wording. ![Changed](https://img.shields.io/badge/changed-b45309?style=flat-square) The multi-employee timesheet PDF is built off the main thread with a loading indicator, so a large timesheet no longer freezes the app for everyone. The logo's right-click menu gained a working **Exit**, replacing a *Reset* entry that did nothing. |
| **0.1.5** | 10 Aug 2026 | ![Changed](https://img.shields.io/badge/changed-b45309?style=flat-square) A partially failed upload still refreshes the table, chart and hours for the rows that did import. Uploading jumps the calendar to the month of the newest punch, with a *"Processing upload..."* spinner while a file is read. ![Fixed](https://img.shields.io/badge/fixed-0f766e?style=flat-square) The Delete Data month picker no longer offers a month after its data was deleted. |
| **0.1.4** | 8 Aug 2026 | ![Fixed](https://img.shields.io/badge/fixed-0f766e?style=flat-square) The Delete Data year and month picker listed every month ever recorded across **all** years, instead of only the months of the year picked. |
| **0.1.3** | 8 Aug 2026 | ![Changed](https://img.shields.io/badge/changed-b45309?style=flat-square) Uploads recalculate hours automatically, so *"Calculate employees working hours by month"* is only needed for a full recompute — after restoring a backup, deleting data, or editing `attendance.db` directly. |
| **0.1.2** | 6 Aug 2026 | The earliest archived release. Its notes PDF ships in `release/archives/`; the changelog source has no itemised entry for it. |

</details>

</details>

<br>

## 09. Copyright and license

![Proprietary](https://img.shields.io/badge/license-proprietary-64748b?style=flat-square)

Copyright © 2026 Immanuel Wiessler and Wiecon Consultants Taiwan. All rights reserved.

This is proprietary software, not open source. It is shared here so that it can be read and reviewed; it may not be copied, modified, distributed or used elsewhere without written permission. The full terms are in the [LICENSE](LICENSE) file.

For permission or questions: [im.wiessler@gmail.com](mailto:im.wiessler@gmail.com) · GitHub [@ThunderIW](https://github.com/ThunderIW)
