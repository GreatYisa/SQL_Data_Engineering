# Week 1, Day 4 — Concepts
## Summarizing Data: From Thousands of Rows to a Few Rows That Answer a Question

---

## Question of the Day
**"How do I turn thousands of rows into a few rows that answer a question?"**

Your `raw_trips` table has hundreds of thousands of rows. Nobody can read that many rows, and a dashboard should not have to process them every time someone opens it. Today you learn to **summarize**: to collapse many rows into a small table that answers a business question. In data engineering, these are called **aggregate tables** or **summary tables**, and building them is one of the most common jobs in a pipeline.

```
 SOURCE FILES -> INGEST -> RAW LAYER -> STAGING -> MARTS -> DASHBOARD
                                                    ^^^^^
                          Today: build a small summary table (a first taste of a mart)
```

---

## 1. Aggregate Functions
An **aggregate function** takes many rows and returns one value.

| Function | What it does | Example |
|---|---|---|
| `COUNT(*)` | Counts rows | Number of trips |
| `COUNT(column)` | Counts rows where that column is **not** `NULL` | Trips that have a start station |
| `COUNT(DISTINCT column)` | Counts different values | Number of different stations |
| `SUM(column)` | Adds up the values | Total minutes ridden |
| `AVG(column)` | The average | Average trip length |
| `MIN(column)`, `MAX(column)` | Smallest and largest value | Shortest and longest trip |
| `median(column)` | The middle value | Typical trip length |

### Aggregates and `NULL`
Aggregate functions **skip `NULL` values**, with one exception: `COUNT(*)` counts every row. That means:

```
 COUNT(*) - COUNT(start_station_name)  =  the number of rows where start_station_name is NULL
```

You can use this as a quick check. It should give the same number you found yesterday with `IS NULL`.

### Average vs. median
An **average** is easily pulled up by a few extreme values. If a handful of trips lasted several days (you may have seen some on Day 3), the average trip length will be much larger than what a normal trip looks like. The **median** (the middle value when everything is sorted) is not affected much by extremes. Comparing the two is a quick way to spot outliers. It also shows why data quality matters: dirty data quietly changes your summary numbers.

---

## 2. `GROUP BY`: One Result Row per Group
Without `GROUP BY`, an aggregate summarizes the whole table into a single row. With `GROUP BY`, the database first sorts the rows into groups, then summarizes each group separately.

Think of sorting a pile of mail into bins by city, then counting each bin.

```sql
SELECT member_casual, COUNT(*) AS trips
FROM raw_trips
GROUP BY member_casual;
```

You get one result row per different `member_casual` value.

**The golden rule:** every column in your `SELECT` must either appear in `GROUP BY` or be inside an aggregate function. If you ask for `start_station_name` and `COUNT(*)` but do not group by the station, the database cannot know which station name to show for the count, so it gives an error.

You can group by:
- **More than one column:** `GROUP BY member_casual, rideable_type` gives one row for each combination
- **A calculation:** `GROUP BY EXTRACT(hour FROM started_at)` gives one row per hour of the day, and `GROUP BY CAST(started_at AS DATE)` gives one row per day

**`NULL` values form their own group.** Trips with no start station will all land together in one group whose name shows `NULL`.

---

## 3. Grain: What Does One Row Mean?
**Grain** is one of the most important ideas in data engineering. It is the answer to the question *"what does one row in this table represent?"*

- `raw_trips` has a grain of **one trip**
- A summary grouped by `member_casual` has a grain of **one rider type**
- A summary grouped by date, rider type, and bike type has a grain of **one day, for one rider type, for one bike type**

`GROUP BY` **defines the grain** of your result. Always be able to say the grain of a table in one sentence. Confusion about grain is the source of a huge number of wrong numbers in real data work, so data engineers state it clearly and write it in documentation.

---

## 4. `WHERE` vs. `HAVING`
Both filter, but at different moments:

| | `WHERE` | `HAVING` |
|---|---|---|
| Filters | Individual **rows** | **Groups** (after they are summarized) |
| Runs | **Before** grouping | **After** grouping |
| Can use aggregates like `COUNT(*)`? | **No** | **Yes** |

Example: *"Which start stations had more than 500 trips?"*

```sql
SELECT start_station_name, COUNT(*) AS trips
FROM raw_trips
WHERE start_station_name IS NOT NULL   -- filter rows first
GROUP BY start_station_name
HAVING COUNT(*) > 500                  -- then filter the groups
ORDER BY trips DESC;
```

- `WHERE` decides **which rows go into the groups**
- `HAVING` decides **which groups make it into the result**

If you try `WHERE COUNT(*) > 500`, you get an error, because when `WHERE` runs, the counting has not happened yet. Reading that error message is part of today's work.

### The full order SQL runs in
Yesterday's order gets two more steps:

1. `FROM`: pick the table
2. `WHERE`: filter rows
3. `GROUP BY`: make groups
4. `HAVING`: filter groups
5. `SELECT`: calculate the columns (and aggregates)
6. `ORDER BY`: sort
7. `LIMIT`: keep the first rows

Keep this order in mind. It explains almost every "why does this not work?" question.

---

## 5. Aliases and Rounding
- Give every calculated column a clear name with `AS`: `COUNT(*) AS trips`, `ROUND(AVG(...), 1) AS avg_minutes`
- `ROUND(value, 1)` keeps one decimal place. Averages have many decimal places, and rounding them makes the result readable.
- Column names in a summary table are read by other people and by dashboards. Good names (`trips`, `avg_trip_minutes`) are part of professional work.

---

## 6. Summary Tables in Data Engineering
Why do data engineers build summary tables instead of letting every report query the raw data?

- **Speed:** a dashboard reading 150 summary rows loads instantly. Reading hundreds of thousands of raw rows every time is slow.
- **Cost:** in cloud warehouses, less data scanned means less money spent. You will see this with Snowflake in Week 3.
- **Consistency:** everyone uses the same definition of "trips per day", because it is calculated once, in one place.
- **Reproducibility:** if the summary is created with `CREATE OR REPLACE TABLE ... AS SELECT ...`, you can rebuild it any time by running the script.

A summary table like this is a first small example of a **data mart**, the final layer of the pipeline that dashboards read from. You will build proper marts in Week 4.

---

## 7. Using Aggregates to Check Your Data
`GROUP BY` is one of the best data quality tools you have.

**Finding duplicates.** Every trip should have its own `ride_id`. To check:
```sql
SELECT ride_id, COUNT(*) AS copies
FROM raw_trips
GROUP BY ride_id
HAVING COUNT(*) > 1;
```
If the query returns **no rows**, there are no duplicates. If it returns rows, some ids appear more than once, and you have found a problem to handle in Week 2.

**Reconciling summaries.** If you summarize a table, the group totals should add back up to the total of the original. For example, the trips per rider type should add up to the total number of trips. If a summary table's `SUM(trips)` does not match the raw row count, rows were lost or duplicated somewhere. This is the same idea as Day 2's row-count check, applied to a summary.

---

## Vocabulary for Day 4
| Term | Meaning |
|---|---|
| Aggregate function | A function that turns many rows into one value (`COUNT`, `SUM`, `AVG`, `MIN`, `MAX`) |
| `GROUP BY` | Sorts rows into groups and summarizes each group |
| `HAVING` | Filters groups after they are summarized |
| Median | The middle value; less affected by extremes than the average |
| Grain | What one row in a table represents |
| Aggregate (summary) table | A small table of summarized results |
| Data mart | A business-ready table that dashboards and reports read from |
| Duplicate | The same record appearing more than once |
| Reconciliation | Checking that two numbers that should agree do agree |

---

## What Success Looks Like Today
By the end of Day 4, you should be able to:
- Use `COUNT`, `SUM`, `AVG`, `MIN`, `MAX`, and `median`, and explain how they treat `NULL`
- Write a `GROUP BY` query, and state the grain of its result
- Group by columns and by calculated values such as date, hour, and weekday
- Explain the difference between `WHERE` and `HAVING`, and use both in one query
- Compare an average with a median to spot outliers
- Build a summary table with `CREATE OR REPLACE TABLE ... AS SELECT`, and check that its totals reconcile with the raw data
- Use `GROUP BY` and `HAVING` to look for duplicate records