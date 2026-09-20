# SQL for Data Engineering

At Dataskools, we build data skills on real data, not toy examples. This six-week course teaches **SQL the way data engineers actually use it**: to load raw data, clean it, model it, test it, and deliver it to the people who need it. You start from zero. By Week 6, you will have built and run a complete, working **data pipeline** on a real cloud data warehouse.

Every week follows the same simple idea: **you do the work step by step, and every step is explained.** No prior SQL or programming experience is needed.

---

## What Is Data Engineering, and Why Does It Need SQL?

**Data engineering** is the work of moving data from where it is created to where it is useful, and making sure it is clean, correct, and on time when it gets there. Data engineers build the **pipelines** that do this.

SQL is the main language of that work. A data engineer uses SQL to:
- **Check** raw data when it arrives (does it look right? did every row load?)
- **Clean** it (fix missing values, duplicates, and inconsistent formats)
- **Transform** it (join, aggregate, and reshape it into tables people can use)
- **Test** it (catch bad data before it reaches a dashboard)
- **Serve** it (publish tables that analysts, dashboards, and data scientists rely on)

Think of a restaurant kitchen. Raw ingredients arrive at the back door (**ingestion**), get washed and prepped (**cleaning**), get cooked into dishes (**transformation**), get checked before leaving the kitchen (**data quality testing**), and get served to customers (**dashboards and reports**). Data engineers run that kitchen, and SQL is the main knife.

### The Pipeline You Will Build

```
 SOURCES          INGEST         RAW LAYER      STAGING LAYER      MARTS LAYER        SERVE
 (CSV files)  ->  (load data) -> (untouched  -> (cleaned,       -> (business-ready -> (dashboard,
 trips, weather                   copy)          standardized)      star schema)       report)

          ---------- data quality checks, version control, and automation across every step ----------
```

Each week adds one part of this picture. By the end you will have built all of it.

---

## How This Course Is Taught (Built for Beginners)

This course is designed for learners who find technical material hard, so:

1. **Start small, then grow.** You begin with one month of data (a few hundred thousand rows), then a full year, then the full seven years. Each jump in size teaches you why data engineers use the tools they do.
2. **Click-by-click instructions.** Every hands-on task tells you exactly where to click, what to type, and what you should see on screen afterwards.
3. **Checkpoints with expected results.** After each step, you get a number or output to compare against (for example, "you should see 412,000 rows"). If yours matches, you move on. If not, a troubleshooting note tells you what to check.
4. **Learn by doing.** Concepts guides are short. Task guides are long and practical. The ratio is roughly one part explanation to two parts hands-on work.
5. **Everyday analogies for every new idea,** followed by the technical term, so the data engineering vocabulary sticks.
6. **The same workflow, every week:** load, clean, transform, check, save to Git. Repetition builds habits.
7. **No programming background needed.** You will copy, run, read, and then modify SQL. Any command-line steps are given in full.

---

## Your Toolkit

| Tool | What it is | What it is used for in data engineering | Introduced |
|---|---|---|---|
| **DuckDB** | A fast analytical database that runs on your own computer (no server to set up) | Practising SQL and building the raw, staging, and mart tables on millions of rows | Week 1 |
| **DBeaver** | A free SQL editor with a point-and-click interface | Writing and running queries, browsing tables and schemas | Week 1 |
| **VS Code** | A code editor | Writing and organizing `.sql` scripts and project files | Week 1 |
| **Terminal (command line)** | A text window for running commands | Running SQL scripts and pipeline commands without clicking | Week 1 |
| **Git and GitHub** | Version control and online code storage | Saving every version of your pipeline code; building a portfolio | Week 1 |
| **CSV and Parquet files** | Two common data file formats | Understanding how data is stored and moved between systems | Week 1 |
| **Snowflake** | A cloud data warehouse | Storing and processing large data in the cloud; running the production pipeline | Week 3 |
| **dbt** | A tool that organizes SQL into tested, connected models | Building the staging and marts layers as a proper, testable pipeline | Week 4 |
| **BI dashboard tool (Power BI or a free browser-based alternative)** | A dashboard builder | Serving your finished tables to end users | Week 6 |

Weeks 1 and 2 use only local tools, so nothing needs a cloud account. **Snowflake enters in Week 3**, once you are comfortable with SQL and the pipeline idea.

---

## The Dataset

All six weeks use one real dataset, so each week builds on the last:

- **Capital Bikeshare trip data (2019–2026):** millions of rides, one row per trip
- **A station reference table:** names and locations of the stations
- **NOAA weather data:** daily weather for the same area and period

It is real data, which means it has real problems: missing values, duplicate rows, odd outliers, and a file layout that changed over the years (called **schema drift**, a very common data engineering headache). Learning to handle this is the point.

---

## Course at a Glance

| Week | Title | Data Engineering Focus | Main Tools | Deliverable |
|---|---|---|---|---|
| 1 | SQL Foundations and Your First Data Load | Ingestion, raw layer, joins, aggregation | DuckDB, DBeaver, VS Code, Git | Top-stations summary table as a saved, reusable SQL script |
| 2 | Cleaning and Transforming Data | Staging layer, data quality checks, reusable scripts | DuckDB, SQL scripts, terminal | Clean staging tables, a data quality report, and a trend analysis |
| 3 | Advanced SQL on Snowflake | Cloud data warehouse, bulk loading, window functions | Snowflake | Raw and staging data loaded into Snowflake; a window-function query |
| 4 | Data Modeling and Your First dbt Project | Star schema, marts, dbt models, lineage | dbt, Snowflake | A star schema built with dbt |
| 5 | Reliable Pipelines | Data tests, documentation, incremental loads, cost and performance | dbt, Snowflake | A tested, documented, incremental pipeline |
| 6 | Capstone: End-to-End Pipeline and Dashboard | Serving layer, full pipeline run, portfolio | Snowflake, dbt, BI tool, GitHub | Working pipeline, dashboard, written report, portfolio repo |

---

## Week-by-Week Details

### Week 1: SQL Foundations and Your First Data Load

**Goal:** Set up a data engineer's workspace, load real raw data into a database, and answer questions with SQL.

**SQL you will learn**
- What SQL is and how relational data is organized into tables, rows, and columns
- `SELECT`, `FROM`, `WHERE`, `ORDER BY`, `LIMIT`
- Filtering by text, number, and date; working with timestamps
- `INNER JOIN`, `LEFT JOIN`, `RIGHT JOIN`, and what `NULL` does in a join
- `GROUP BY` with `COUNT`, `SUM`, `AVG`, `MIN`, `MAX`; `HAVING` vs. `WHERE`; aliases and rounding

**Data engineering ideas you will learn**
- Data sources and the **raw layer** (an untouched copy of the incoming data)
- **Ingestion:** loading a file into a database table
- **Schema** and **data types**
- CSV vs. Parquet file formats
- **Row-count validation:** checking that every row arrived (your first data quality test)
- Saving work in **version control**

**How data engineering benefits**
Every pipeline starts with looking at raw data, and `SELECT` with filters is how you inspect it. Joins bring separate sources together. Aggregation builds the summary tables that reports run on. Row counts are the simplest and most common check that a load worked.

**What you will do:** install DuckDB, DBeaver, VS Code, and Git; create your first GitHub repository; load one month of trip data; explore it; join it to the station table; build a top-stations summary and save it as a script.

**Mini-project:** A joined, aggregated top-stations summary table, saved as a reusable SQL script in GitHub, with a row-count check proving the load worked.

---

### Week 2: Cleaning and Transforming Data

**Goal:** Turn messy raw data into a clean, trustworthy **staging layer**, and learn to prove it is trustworthy.

**SQL you will learn**
- Subqueries in `SELECT`, `WHERE`, and `FROM`
- Common Table Expressions (CTEs)
- `NULL` handling, `COALESCE`, `CASE WHEN`
- Finding and flagging unusual periods in a time series

**Data engineering ideas you will learn**
- The **staging layer** (a cleaned, standardized version of raw data)
- Cleaning tasks: fixing data types, handling missing values, removing duplicates
- **Schema drift:** when the same data arrives in different layouts over time
- **Data quality checks:** null counts, duplicate checks, out-of-range values
- **Idempotent scripts:** scripts you can safely run again and get the same result
- Running SQL from a script file in the terminal (your first step toward **automation**)
- Scaling from one month to a full year

**How data engineering benefits**
Cleaning is where data engineers spend much of their time. CTEs break a big transformation into small, readable steps, which is exactly how professional pipeline code is written. Quality checks catch bad data early, before anyone builds a dashboard on it.

**What you will do:** build a clean staging table from raw trips; write quality checks and a small quality report; flag anomalous periods; save the whole flow as a script that runs from start to finish with one command.

**Mini-project:** Identify a meaningful trend in the data and separate genuine change from data gaps and one-off anomalies, backed by a clean staging table and a data quality report.

---

### Week 3: Advanced SQL on Snowflake

**Goal:** Move your pipeline into a **cloud data warehouse** and learn the advanced SQL that data engineers use every day.

**SQL you will learn**
- Window functions: `ROW_NUMBER`, `RANK`, `DENSE_RANK`, `LAG`, `LEAD`
- Running totals, moving averages, and period-over-period comparisons
- Pivoting and unpivoting data
- Recursive CTEs (optional stretch topic)

**Data engineering ideas you will learn**
- What a cloud data warehouse is and why teams use one
- Snowflake basics: **storage and compute are separate**, and **virtual warehouses** (the compute that runs your queries)
- Databases, schemas, and tables in Snowflake; organizing `RAW`, `STAGING`, and `MARTS` schemas
- **Stages** and **bulk loading** with `COPY INTO`
- Roles and permissions basics
- Credits and **cost awareness** (why you suspend a warehouse when you are done)
- Loading the full seven years of data, and why the cloud helps at this scale

**How data engineering benefits**
Window functions are how data engineers calculate running metrics and find the "latest record per key", which is the standard way to remove duplicates in real pipelines. Snowflake is where production pipelines usually run, and learning to load data into it is a core data engineering skill.

**What you will do:** create a Snowflake account; create a warehouse, database, and schemas; upload files to a stage; load them with `COPY INTO`; run your Week 2 cleaning SQL on Snowflake; write window-function queries.

**Mini-project:** Rewrite an earlier query using window functions to solve something `GROUP BY` alone could not, and use `ROW_NUMBER` to remove duplicate rows, running on Snowflake.

---

### Week 4: Data Modeling and Your First dbt Project

**Goal:** Organize your tables into a proper **data model**, and build it as a real pipeline with dbt.

**SQL and modeling you will learn**
- **Dimensional modeling:** fact tables and dimension tables, the **star schema**, and **grain** (what one row means)
- Keys and relationships between tables
- Building a **data mart** (a business-ready table set) from the staging layer

**Data engineering ideas you will learn**
- What **dbt** is and how it fits an **ELT** workflow (load first, transform inside the warehouse)
- dbt project structure: **sources**, **staging models**, **marts models**
- Using `ref()` to connect models, and **materializations** (views vs. tables)
- `dbt run`, and reading the **lineage graph** (DAG)

**How data engineering benefits**
A good model makes data easy to use and gives everyone the same definitions. dbt turns loose SQL scripts into version-controlled, connected models that a whole team can maintain, which is why it is the standard tool for the transformation step.

**What you will do:** install dbt and connect it to Snowflake; turn your staging SQL into dbt models; build `fct_trips`, `dim_stations`, `dim_date`, and `dim_weather`; view the lineage graph.

**Mini-project:** A star schema for bikeshare trips, built as a dbt project on Snowflake.

---

### Week 5: Reliable Pipelines

**Goal:** Make the pipeline **trustworthy, documented, and efficient**.

**What you will learn**
- **dbt tests:** `not_null`, `unique`, `relationships`, `accepted_values`
- **Documentation generation** and lineage
- **Incremental models:** loading only new data instead of reprocessing years of history every time
- Source **freshness** (is the data up to date?)
- Performance and cost basics: reading a query plan, warehouse sizing, and clustering concepts
- Materialized views, and why they speed up dashboards
- Everyday Git workflow: branches and pull requests
- **Orchestration** concepts: how pipelines run on a schedule (explained, not built)

**How data engineering benefits**
Tests are how data engineers earn trust: a failing test stops bad data before it spreads. Incremental models keep cost and run time low as data grows. Documentation and lineage let other people understand and safely change your work.

**What you will do:** add tests to your models; break the data on purpose and watch tests catch it; convert a large model to incremental; generate documentation; check query cost and performance.

**Mini-project:** A tested, documented, incremental version of your bikeshare pipeline.

---

### Week 6: Capstone: End-to-End Pipeline and Dashboard

**Goal:** Run the whole pipeline from raw to dashboard, and package your work as a portfolio.

**What you will do**
- Run the full pipeline end to end: raw, staging, and marts on Snowflake
- Connect a BI tool to your marts and build a dashboard (bar and line charts, slicers)
- Write a short report: **three data-backed observations and one recommendation**
- Set up your GitHub portfolio with a clear README and a pipeline diagram
- Review data and analytics career tracks: data engineer, analytics engineer, and BI or data analyst
- Course completion review

**How data engineering benefits**
A pipeline is only useful when its output reaches people. The **serving layer** (dashboards and reports) is where data engineering work becomes visible, and a documented, working project is what employers want to see.

**Deliverables**
- GitHub repository with all SQL and the dbt project
- A pipeline diagram
- Dashboard file and a screenshot
- Written report

---

## Data Engineering Words You Will Learn

| Term | Simple meaning |
|---|---|
| **Data pipeline** | A series of automated steps that moves and prepares data |
| **Ingestion** | Bringing data from a source into your system |
| **Raw / staging / marts layers** | Untouched data, then cleaned data, then business-ready data |
| **ETL / ELT** | Extract, Transform, Load, or Extract, Load, Transform (transforming after loading) |
| **Schema** | The structure of a table: its columns and their types |
| **Schema drift** | When a data source changes its layout over time |
| **Data warehouse** | A database built for analysis on large data |
| **Data quality test** | An automated check that data meets expectations |
| **Idempotent** | Safe to run many times with the same result |
| **Incremental load** | Processing only new or changed data |
| **Lineage / DAG** | A map showing which tables are built from which |
| **Grain** | What one row in a table represents |
| **Orchestration** | Scheduling and coordinating pipeline steps |

---

## What You Will Have Built by the End

A complete, working pipeline: raw data loaded into a cloud data warehouse, cleaned into a staging layer, modeled into a star schema with dbt, tested and documented, and served through a dashboard. All of the code lives in a GitHub repository you can show to employers.

You will also be able to explain, in plain language, what each layer does and why data engineers build pipelines this way.

---

## How Each Week's Materials Are Organized

Each week has two guides:
- **Concepts:** short explanations with everyday analogies and the data engineering vocabulary for the week
- **Tasks:** long, step-by-step, hands-on instructions with screen-by-screen directions, expected results at each checkpoint, and troubleshooting tips

*See `Week1_Concepts.md` and `Week1_Tasks.md` onward for the detailed weekly guides.*