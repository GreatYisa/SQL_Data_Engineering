# Week 1 — SQL Foundations and Your First Data Load

## Week Goal
This week you set up a data engineer's workspace, bring real raw data into a database, and learn the core SQL used to inspect, filter, summarize, and combine data. By Friday you will have done the first three steps of every data pipeline: **ingest** the data, **check** it, and **summarize** it into a table someone could use.

You will not just learn SQL commands. Every command is tied to a data engineering job: `SELECT` is how you inspect raw data, `WHERE` is how you find bad rows, `GROUP BY` is how you build summary tables, and `JOIN` is how you combine sources. Each day shows you what the command does and why a data engineer needs it.

**Key theme to keep in mind throughout the week:** *"Load it, look at it, check it, then use it. A data engineer never trusts data until they have checked it."*

---

## Where This Week Fits in the Pipeline

```
 SOURCE FILES  ->  INGEST  ->  RAW LAYER  ->  (Week 2: STAGING)  ->  (Week 4: MARTS)  ->  (Week 6: DASHBOARD)
   CSV files       load into    untouched
                   a database   copy of data
        ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
               THIS WEEK (Week 1)
```

Week 1 covers the front of the pipeline: getting data in, and understanding it. Later weeks clean it, model it, test it, and serve it.

---

## Dataset Used
**Capital Bikeshare trip data**, the same real dataset you will use for all six weeks:
- **Trips table:** one row per bike ride, with start and end time, start and end station, and the type of rider
- **Station reference table:** the names and locations of the stations

This week you work with **one month of trips** (a few hundred thousand rows). That is big enough to feel like real data and small enough to run fast on any laptop. In later weeks the amount of data grows, and you will see why data engineers use bigger tools.

The data is real, so expect some messiness. Finding it, not fixing it, is this week's job. (Fixing comes in Week 2.)

---

## Tools This Week

| Tool | What it does | Data engineering use |
|---|---|---|
| **DuckDB** | A database that runs on your own computer, with nothing to install on a server | Stores your raw tables and runs your SQL |
| **DBeaver** | A free SQL editor with a point-and-click interface | Writing queries, browsing tables, viewing results |
| **VS Code** | A code editor | Writing and saving `.sql` script files |
| **Terminal** | A window where you type commands | Running scripts (light use this week) |
| **Git and GitHub** | Version control and online storage for code | Saving every version of your work and building your portfolio |

---

## How Data Engineering Uses This Week's SQL

| SQL you learn | What you do with it | Why a data engineer needs it |
|---|---|---|
| `SELECT`, `FROM`, `LIMIT` | Look at columns and sample rows | First step on any new data source: inspect before you build |
| `WHERE`, `ORDER BY` | Find specific rows and unusual values | Spot bad or suspicious data early (for example, a trip that ends before it starts) |
| Dates and timestamps | Filter by day, month, or hour | Pipelines almost always work with time-based data |
| `GROUP BY` + `COUNT`, `SUM`, `AVG`, `MIN`, `MAX` | Turn many rows into a short summary | Builds the **summary tables** that dashboards run on |
| `HAVING` | Filter after grouping | Removes noise from summaries |
| `JOIN` | Combine trips with stations | Brings separate data sources together |
| `NULL` handling | See what is missing or unmatched | Missing data is the most common data quality problem |
| Row counts | Compare counts before and after each step | The simplest **data quality check**: did every row make it? |

---

## Week Structure

| Day | Focus | Data Engineering Idea | Real-World Question It Answers |
|-----|-------|-----------------------|---------------------------------|
| 1 | Set up your workspace: DuckDB, DBeaver, VS Code, Git, GitHub; what SQL and relational data are | Workspace, database, version control | "What does a data engineer's workspace look like, and how do I know mine works?" |
| 2 | Loading data: files to tables, `SELECT` and `LIMIT`, data types, CSV vs. Parquet | Ingestion, raw layer, schema, row-count validation | "How does a file become a table, and how do I prove every row arrived?" |
| 3 | Filtering and sorting: `WHERE`, `ORDER BY`, dates and timestamps, `NULL` checks | Inspecting raw data, spotting bad rows | "How do I find the rows I need, and the rows that look wrong?" |
| 4 | Summarizing: `GROUP BY`, aggregate functions, `HAVING` vs. `WHERE`, aliases, rounding | Summary (aggregate) tables | "How do I turn thousands of rows into a few rows that answer a question?" |
| 5 | Joining tables: `INNER`, `LEFT`, `RIGHT` joins, `NULL`s in joins; mini-project | Combining sources, join checks, reusable scripts | "How do I combine two tables safely, and package my work so it can be re-run?" |

---

## Approach This Week (Beginner-Friendly)
1. **Small data first:** one month of trips, so everything runs fast and mistakes are cheap
2. **Click-by-click instructions:** every task says exactly where to click, what to type, and what you should see on screen
3. **Checkpoints with expected results:** after each step you compare a number or screenshot description to yours; if it matches, you move on, and if not, a troubleshooting note tells you what to check
4. **Copy, run, read, then change:** you first run a working example, then read it line by line, then modify it yourself
5. **One idea at a time:** each new SQL command is introduced with a real data engineering reason and an everyday analogy
6. **Check your row counts after every step:** this habit is the foundation of data quality work
7. **Save your work daily:** at the end of each day you save your SQL files and commit them to GitHub, so your repository grows into a portfolio
8. **Vocabulary a little at a time:** each day adds a few data engineering terms, with plain-language meanings

---

## Your Project Folder

By the end of Week 1, your repository will look like this:

```
bikeshare-pipeline/
  data/
    raw/                  <- downloaded files (not uploaded to GitHub)
  sql/
    01_load_raw.sql       <- loads files into tables and checks row counts
    02_explore.sql        <- filtering and sorting practice
    03_summaries.sql      <- GROUP BY practice
    04_top_stations.sql   <- the mini-project
  .gitignore              <- tells Git to skip big data files
  README.md               <- describes your project
```

Keeping data files out of Git and only saving the code is standard data engineering practice.

---

## 📌 Mini-Project (Week 1): Top Stations Summary
Build a **joined, aggregated summary table** that answers: *which start stations are the busiest, and what do we know about them?*

**Requirements:**
1. Raw trips and stations tables loaded into DuckDB, with a row-count check proving every row arrived
2. Trips filtered to sensible rows (for example, a valid date range and no obviously broken records)
3. Trips joined to the station reference table, with a check of how many trips did not match a station
4. Trips grouped by station, with a trip count and an average trip duration
5. `HAVING` used to leave out stations with very few trips
6. Results sorted, showing the top 10 stations
7. Everything saved as a script that runs from start to finish, committed to GitHub

---

## Deliverable by End of Week 1
1. A GitHub repository with the folder structure above and your SQL scripts committed
2. A DuckDB database containing your raw trips and stations tables
3. A row-count check showing that the number of rows loaded matches the source file
4. A top-stations summary table, created by a saved SQL script
5. A short written note (a few sentences) on what you noticed about the data, including anything that looked wrong or missing

This mirrors real data engineering work: you got data in, checked that it arrived intact, turned it into something useful, and saved your work in a way that anyone can re-run.

---

## Week 1 Vocabulary
**Database, table, row, column, SQL query, schema, data type, ingestion, raw layer, CSV, Parquet, row count, aggregate, join, NULL, version control, repository, commit.**

Each term is explained in the daily Concepts guides.

---
*See `Week1_Day1_Concepts.md` / `Week1_Day1_Tasks.md` through `Week1_Day5_Concepts.md` / `Week1_Day5_Tasks.md` for detailed daily breakdowns.*