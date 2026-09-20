# Week 1, Day 2 — Concepts
## Loading Data: From Files to Raw Tables

---

## Question of the Day
**"How does a file become a table, and how do I prove every row arrived?"**

Yesterday you built the workspace. Today the real work of a data engineer starts: **bringing data in**. You will take two real files, a month of Capital Bikeshare trips and a list of stations, and load them into your database. Then you will do the thing that separates a data engineer from a casual user: you will *prove* that everything arrived.

```
 SOURCE FILES -> INGEST -> RAW LAYER -> STAGING -> MARTS -> DASHBOARD
 (CSV, JSON)     ^^^^^^^^^^^^^^^^^^^^
                 Today
```

---

## 1. What Is Ingestion?
**Ingestion** means bringing data from a source (a file, a website feed, another database) into the system where you will work with it. In the pipeline you are building, that is the **Extract and Load** part.

Think of a delivery truck arriving at a restaurant. Ingestion is unloading the boxes and putting them on the shelves. You do not open them, cook them, or throw anything away yet. You just make sure everything that was on the truck is now in the building.

Today's ingestion is **batch ingestion from files**: a whole file is loaded in one go. (Other pipelines load data continuously as it arrives, but batch files are the most common place to begin.)

---

## 2. The Raw Layer
Data engineers keep data in **layers**. The first layer is the **raw layer**: an exact copy of the source data, exactly as it arrived.

The rules of the raw layer:
- **Do not clean it.** No fixing, no deleting, no renaming.
- **Do not edit it.** If something looks wrong, leave it and make a note.
- **Keep a record of where it came from.** Extra columns such as `source_file` (which file it came from) and `loaded_at` (when it was loaded) are added so you can always trace a row back to its origin.

**Why?** If something goes wrong later in the pipeline, you can always return to the raw layer and rebuild everything from it. If you had already changed the raw data, you would have nothing reliable to go back to. The raw layer is your safety net and your single source of truth.

By convention, raw tables often start with `raw_`, so today's tables are `raw_trips` and `raw_stations`.

---

## 3. File Formats: CSV, JSON, and Parquet
Data comes in different **file formats**. You will meet three today.

| Format | What it looks like | Good for | Watch out for |
|---|---|---|---|
| **CSV** | Plain text, one row per line, values separated by commas | Simple, universal, easy to open | Files are large, and there is no built-in type information, so every value is just text until something guesses |
| **JSON** | Text with curly braces `{}` and labeled values, and it can nest lists inside lists | Data from websites and feeds (APIs) | Nested structure has to be flattened into rows and columns before it is a table |
| **Parquet** | A binary file (not readable as text) that stores data by *column* | Analytics: small, fast, and it remembers each column's type | You cannot open it in a text editor |

**Columnar storage, simply:** a CSV stores data row by row, like a list of index cards. Parquet stores it column by column, like keeping all the first names together, all the dates together, and so on. Questions like "what is the average trip length?" only need one column, so a columnar file can skip everything else. It also compresses very well because similar values sit together.

**Why data engineers care:** files moving between systems in a real pipeline are often Parquet, because they are smaller and faster. Today you will see the size difference with your own eyes.

---

## 4. Schema and Data Types
A **schema** is the structure of a table: the column names and the **data type** of each column.

| Data type (DuckDB) | Holds | Example |
|---|---|---|
| `VARCHAR` | Text | `'Union Station'` |
| `BIGINT` | Whole numbers | `31623` |
| `DOUBLE` | Decimal numbers | `38.897` |
| `TIMESTAMP` | Date and time | `2025-05-01 08:05:00` |

**Why types matter:** the database treats each type differently. As text, the values `100` and `20` sort with `100` first (because `1` comes before `2`). As numbers, `20` comes first. A date stored as text cannot be filtered by month or subtracted from another date. A wrong type quietly gives wrong answers.

**Where do types come from?** A CSV file has none. When DuckDB reads a CSV, it **infers** the schema, which means it reads some rows and guesses each column's type. This is convenient, but a guess can be wrong, especially if the first rows look different from the rest. So a data engineer always **checks the schema after loading** (with `DESCRIBE`). Today you will also tell DuckDB to read the *whole* file before guessing (`sample_size = -1`), which makes the guess more reliable.

> In production pipelines, engineers often write the schema out explicitly instead of trusting a guess. You will do this later in the course.

---

## 5. Reading Files Directly With SQL
DuckDB can treat a file as if it were a table, without loading it first:

```sql
SELECT * FROM read_csv('some_file.csv') LIMIT 5;
```

There are matching functions for other formats: `read_json_auto(...)` for JSON and `read_parquet(...)` for Parquet. This is very handy for a **peek** before you commit to loading anything.

To save the result as a real table, you use **CREATE TABLE AS SELECT** (often shortened to **CTAS**). It means "run this query and store the result as a new table":

```sql
CREATE OR REPLACE TABLE raw_trips AS
SELECT * FROM read_csv('some_file.csv');
```

---

## 6. Idempotent Loads: `CREATE OR REPLACE`
A load is **idempotent** if running it once, twice, or ten times gives the same result. Why does this matter? Pipelines get re-run all the time (after a failure, after a fix, on a schedule). If every re-run adds the rows again, you end up with duplicates and wrong numbers.

`CREATE OR REPLACE TABLE` is idempotent: it rebuilds the table from scratch each time. An `INSERT INTO` run twice would add the rows twice. Today you will prove this by running your load twice and checking that the row count does not change.

---

## 7. Semi-Structured Data: Flattening JSON
The station list arrives as JSON with a nested structure. Instead of one row per station, the file has a small header (like when it was updated) and one big list of stations inside a field called `data`. It is like a single box with all the stations packed inside.

To turn that into a table with one row per station, you **flatten** it using `unnest`, which takes a list and spreads it into separate rows. Flattening nested data is a very common data engineering task, because APIs and web feeds mostly deliver JSON.

---

## 8. Row-Count Validation
After loading, you must answer the question: **"Did everything arrive?"** The simplest test is to compare the number of rows in the **source** with the number of rows in your **target** table.

```
 rows in the source file  =?=  rows in the raw table
```

- If the numbers match, the load is at least the right size.
- If the target has fewer rows, some data was lost.
- If the target has more, something was duplicated.

This is called **reconciliation**, and it is the most basic **data quality check**. Later in the course, checks like this are automated so a failed check stops the pipeline. Today you do it by hand, which makes you understand what the automation is doing.

---

## 9. Peeking at Big Tables Safely: `LIMIT`
Your trips table has hundreds of thousands of rows. Asking for all of them just to look is slow and pointless. Always add `LIMIT` when you are exploring:

```sql
SELECT * FROM raw_trips LIMIT 10;
```

On a very large table in a cloud warehouse, an unrestricted `SELECT *` can also cost real money, so this is a habit worth building from the start.

---

## 10. A Note About Real Data
Today you load **one month** of trips in the newer file layout. In earlier years the trip files used different columns, and the layout changed over time. That is called **schema drift**, and you will meet it in Week 2. For now, notice how your columns are named.

---

## Vocabulary for Day 2
| Term | Meaning |
|---|---|
| Ingestion | Bringing data from a source into your system |
| Raw layer | An untouched copy of source data, with traceability columns |
| Batch load | Loading a whole file in one go |
| CSV, JSON, Parquet | Three common file formats |
| Columnar storage | Storing data column by column, as Parquet does |
| Schema | The column names and types of a table |
| Data type | The kind of value a column holds |
| Schema inference | The database guessing types from the data |
| CTAS | `CREATE TABLE AS SELECT`: save a query result as a table |
| Idempotent | Safe to run again with the same result |
| Flatten / `unnest` | Turn a nested list into separate rows |
| Row-count validation | Comparing source and target row counts to check a load |
| Reconciliation | Checking that two things agree |

---

## What Success Looks Like Today
By the end of Day 2, you should be able to:
- Explain what ingestion is and why the raw layer is never edited
- Describe the difference between CSV, JSON, and Parquet, and why Parquet is popular in data engineering
- Explain what a schema and a data type are, and why a guessed schema must be checked
- Load a CSV and a JSON file into DuckDB tables with SQL
- Prove a load is complete using a row-count check, and prove it is idempotent
- Use `LIMIT` to peek at large tables