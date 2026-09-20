# Week 1, Day 1 — Concepts
## Your Data Engineer Workspace: Databases, SQL, and Version Control

---

## Question of the Day
**"What does a data engineer's workspace look like, and how do I know mine works?"**

Today you do not analyze any data yet. Today you build the workspace you will use for the next six weeks, and you prove that it works. Data engineers care a lot about this: a pipeline is only as reliable as the setup it is built and run in. By the end of today you will have a database, a tool to talk to it, a place to write SQL files, and an online repository that stores your work.

---

## 1. What Is Data Engineering?
**Data engineering** is the work of moving data from where it is created to where it is useful, and making sure it is clean, correct, and on time when it arrives. The system that does this is called a **data pipeline**.

Think of a restaurant kitchen:
- Ingredients arrive at the back door → **ingestion** (bringing data in)
- They are washed and prepared → **cleaning**
- They are cooked into dishes → **transformation**
- The dish is checked before it leaves the kitchen → **data quality testing**
- It is served to the customer → **dashboards and reports**

Data engineers run that kitchen. Over six weeks you will build every part of it. Today you set up the kitchen itself.

```
 SOURCE FILES -> INGEST -> RAW LAYER -> STAGING -> MARTS -> DASHBOARD
                 ^^^^^^^^^^^^^^^^^^^^
                 Week 1 (Day 1 = setting up the workspace)
```

---

## 2. What Is a Database?
A **database** is an organized place to store data so it can be searched and combined quickly.

The basic building block is a **table**. A table looks like one tab in a spreadsheet:
- Each **column** holds one kind of information (for example, `start_station_name`)
- Each **row** holds one record (for example, one bike trip)

A database is like a workbook with many tabs, but with stricter rules than a spreadsheet:
- Every column has a **data type** (text, number, date and time). You cannot put text into a number column.
- It handles millions of rows quickly, far beyond what a spreadsheet can manage.
- You ask it questions using a language (SQL) instead of scrolling and clicking.

**Why data engineers use databases:** pipelines move data from file to file and system to system, and the database is where data lives while it is being checked, cleaned, and combined.

---

## 3. What Is SQL?
**SQL** (Structured Query Language, often said "S-Q-L" or "sequel") is the language you use to talk to a database. A piece of SQL that asks a question is called a **query**.

The most common query looks like this:

```sql
SELECT start_station_name, member_casual
FROM practice_trips
LIMIT 3;
```

In plain English: *"From the table `practice_trips`, show me the columns `start_station_name` and `member_casual`, and only the first 3 rows."*

A few rules to know from day one:
- A statement ends with a semicolon (`;`)
- SQL keywords such as `SELECT` and `FROM` are usually written in capital letters, which is a convention that makes queries easier to read
- You can spread a query over several lines, and SQL does not care about the line breaks

**Why data engineers use SQL:** it is the shared language of data work. The same SQL skills carry across almost every database and cloud warehouse you will meet, including Snowflake in Week 3.

---

## 4. Your Database and Your SQL Editor
Two tools work together:

| Tool | What it is | Simple picture |
|---|---|---|
| **DuckDB** | A database that runs on your own computer and keeps everything in one file ending in `.duckdb` | The filing cabinet that holds your tables |
| **DBeaver** | A free program with buttons and menus for writing SQL and viewing results | The desk where you ask questions and read the answers |

DBeaver connects to DuckDB through a **connection**. A connection is the saved link between the two. You create it once, and DBeaver remembers it.

**Why DuckDB for a data engineer:** it is built for analysis on large amounts of data, and it needs no server or complicated setup. It gives you a small, fast practice warehouse on your own laptop.

---

## 5. Why We Save SQL in Files
In DBeaver you can type SQL and run it. But data engineers do not leave their work only in the editor. They save SQL in **script files** (ending in `.sql`).

Why?
- **Repeatable:** you can run the same script again tomorrow and get the same result
- **Shareable:** a teammate can read it, run it, and improve it
- **Reviewable:** you can see what changed and when
- **The pipeline is the code:** in real projects, the SQL files *are* the pipeline

Think of a script as a written recipe. Anyone with the recipe can cook the same dish.

---

## 6. Version Control: Git and GitHub
When you edit a file, the old version is gone. **Version control** keeps every version, like save points in a video game. You can go back to any of them.

| Term | Meaning |
|---|---|
| **Git** | A tool on your computer that tracks changes to your files |
| **GitHub** | A website that stores your Git projects online |
| **Repository (repo)** | A project folder that Git is tracking |
| **Clone** | Make a copy of an online repository on your computer |
| **Commit** | Save a snapshot of your changes, with a short message describing them |
| **Push** | Send your commits from your computer up to GitHub |

The habit you will build every day: **write, save, commit, push.**

**Why data engineers use it:** pipelines are code, and code changes constantly. Version control lets a team work on the same project without overwriting each other, and it lets you undo mistakes. For you, your GitHub repository also becomes a **portfolio** that employers can read.

---

## 7. What We Do Not Put in Git: `.gitignore`
A file called `.gitignore` lists things Git should skip. In this course it will skip:
- **Data files and the `.duckdb` file.** They are big and they can be rebuilt by running your scripts.
- Small system files your computer creates automatically.

The rule data engineers follow: **store the code, not the data.** If your code is good, anyone can recreate the data by running it. That is called **reproducibility**, and it is one of the most important ideas in data engineering.

---

## 8. Your Project Folder
A tidy project is easier to understand and to re-run. Yours will look like this:

```
bikeshare-pipeline/
  data/
    raw/             <- downloaded data files go here (from Day 2)
    bikeshare.duckdb <- your database file (created today)
  sql/               <- all your SQL scripts
  .gitignore         <- tells Git what to skip
  README.md          <- describes your project
```

Everything has a place. This is a habit worth building early.

---

## 9. Practice Data Today
Today's queries use a tiny table called `practice_trips` with 8 rows. **It is made-up sample data, not real trips**, shaped like the real bikeshare data you will load tomorrow. Starting with a tiny table lets you learn the tools without waiting for large downloads.

---

## Vocabulary for Day 1
| Term | Meaning |
|---|---|
| Data pipeline | Automated steps that move and prepare data |
| Ingestion | Bringing data in from a source |
| Database | An organized store of data |
| Table, row, column | A grid of data: rows are records, columns are fields |
| Data type | The kind of value a column holds (text, number, timestamp) |
| SQL query | A written question to a database |
| Connection | The saved link between DBeaver and a database |
| Script | A file of SQL that can be re-run |
| Repository, commit, push, clone | Git and GitHub words for storing and saving your project |
| Reproducibility | Anyone can recreate your results by running your code |

---

## What Success Looks Like Today
By the end of Day 1, you should be able to:
- Explain, in your own words, what a data pipeline is and what a database table is
- Describe what SQL is used for, and read a simple `SELECT` query
- Explain why data engineers save SQL in files and use Git and GitHub
- Have a working workspace: DBeaver connected to a DuckDB database, VS Code open on your project, and your project stored on GitHub
- Run your first SQL queries and know how to check that they worked