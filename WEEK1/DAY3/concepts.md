# Week 1, Day 3 — Concepts
## Filtering and Sorting: Finding the Rows You Need, and the Rows That Look Wrong

---

## Question of the Day
**"How do I find the rows I need, and the rows that look wrong?"**

Yesterday you loaded the raw data and proved it arrived. Today you start to **look inside it**. A data engineer never builds on data they have not inspected. The SQL you learn today does two jobs: it finds the rows you want, and it finds the rows that might cause trouble later.

```
 SOURCE FILES -> INGEST -> RAW LAYER -> STAGING -> MARTS -> DASHBOARD
                            ^^^^^^^^^
                            Today: inspect it (but do not change it)
```

The rule for today: **find it, note it, do not fix it.** The raw layer stays untouched. You will record what you find, and in Week 2 you will decide how to handle it in the staging layer.

---

## 1. `WHERE`: Keeping Only the Rows That Pass a Test
`WHERE` filters rows. Think of a sieve: every row is tested against your condition, and only the rows that pass come through.

```sql
SELECT ride_id, member_casual
FROM raw_trips
WHERE member_casual = 'casual';
```

Rules to remember:
- **Text goes in single quotes:** `'casual'`. Numbers do not: `60`.
- **Text matching is exact.** `'casual'` and `'Casual'` are different values.
- **Comparison operators:** `=` (equal), `<>` or `!=` (not equal), `>`, `<`, `>=`, `<=`.

**Why data engineers use it:** pipelines constantly need "just the rows for this month", "just the valid records", or "just the new data". `WHERE` is how you say that.

---

## 2. Combining Conditions: `AND`, `OR`, `IN`, `LIKE`
| Tool | Meaning | Example |
|---|---|---|
| `AND` | Both conditions must be true | `member_casual = 'casual' AND rideable_type = 'electric_bike'` |
| `OR` | At least one must be true | `rideable_type = 'classic_bike' OR rideable_type = 'electric_bike'` |
| `NOT` | Reverses a condition | `NOT member_casual = 'member'` |
| `IN (...)` | Matches any value in a list | `rideable_type IN ('classic_bike', 'electric_bike')` |
| `LIKE` | Matches a text pattern, where `%` means "anything" | `start_station_name LIKE '%Union Station%'` |

**Use parentheses when mixing `AND` with `OR`.** SQL works out `AND` before `OR`, which can give a surprising result. Parentheses make your meaning clear:

```sql
WHERE member_casual = 'casual'
  AND (rideable_type = 'electric_bike' OR start_station_name IS NULL)
```

`LIKE` is case-sensitive in most databases. `ILIKE` is the case-insensitive version.

---

## 3. Sorting and Limiting: `ORDER BY` and `LIMIT`
- `ORDER BY column` sorts the results. Add `DESC` for largest first (`ASC`, smallest first, is the default).
- You can sort by more than one column: `ORDER BY member_casual, started_at DESC`.
- `LIMIT n` keeps only the first *n* rows.

**Important:** without `ORDER BY`, the database returns rows in **no guaranteed order**. So `LIMIT 10` on its own gives you *some* 10 rows, not the "top" 10. To get a true "top 10", you need `ORDER BY` first, then `LIMIT`.

**Why data engineers care:** pipelines must give the same result every time they run. Results that depend on an accidental row order are a hidden source of bugs.

---

## 4. Computed Columns
`SELECT` can calculate new columns from existing ones. The bikeshare file has a start time and an end time, but no duration column, so you can work it out:

```sql
SELECT ride_id,
       ROUND(datediff('second', started_at, ended_at) / 60.0, 1) AS trip_minutes
FROM raw_trips;
```

- `datediff('second', start, end)` counts the seconds between two timestamps
- dividing by `60.0` turns seconds into minutes
- `ROUND(..., 1)` keeps one decimal place
- `AS trip_minutes` gives the new column a name (an **alias**)

Creating new columns from raw ones is a core data engineering job. Tomorrow's summaries and next week's staging layer are built on it.

---

## 5. Dates and Timestamps
A **timestamp** stores a date *and* a time. Dates in queries are written as text in the form `'YYYY-MM-DD'`, and the database reads `'2025-05-10'` as **midnight at the start of that day**.

### Filtering a whole day the safe way
To select every trip on May 10, use a window that starts at the beginning of the day and stops just before the next one:

```sql
WHERE started_at >= '2025-05-10'
  AND started_at <  '2025-05-11'
```

This is a **half-open range**: it includes the start and excludes the end.

### The `BETWEEN` trap
It is tempting to write `BETWEEN '2025-05-10' AND '2025-05-10'`. But both dates mean midnight, so that window has zero length and matches almost nothing. The query runs without an error and quietly returns the wrong answer. Data engineers avoid this by using half-open ranges for time.

**Why it matters:** pipelines constantly select "all data for yesterday" or "all data for this month". A boundary mistake means missing rows or double-counted rows, and nobody gets an error message.

### Pulling out parts of a timestamp
| Expression | Gives you |
|---|---|
| `EXTRACT(hour FROM started_at)` | The hour, 0 to 23 |
| `dayname(started_at)` | The weekday name, such as `Monday` |
| `CAST(started_at AS DATE)` | Just the date, without the time |

These let you ask questions like "how many trips start during the morning commute?"

---

## 6. `NULL`: The Value That Means "Unknown"
`NULL` means **no value**. It is not zero, and it is not an empty piece of text. It is missing or unknown.

Two rules that trip everyone up:
1. **You cannot test for `NULL` with `=`.** `WHERE start_station_name = NULL` never matches anything. The correct test is `IS NULL` (or `IS NOT NULL`).
2. **A condition involving `NULL` is never true, so the row silently drops out.** For example, `WHERE start_lat NOT BETWEEN 38.5 AND 39.5` will *not* return rows where `start_lat` is `NULL`. You have to ask for those separately.

**Is a `NULL` always a mistake?** No. In the trips data, a missing station name may simply mean the trip did not start or end at a station. The `NULL` might be meaningful. Your job today is to find out how many there are and what they look like, and to leave the decision about what to do to Week 2.

---

## 7. Data Quality Checks
A **data quality check** tests whether data follows the rules it should. Looking through data for problems like this is called **data profiling**. Today's checks are all the same shape: write a `WHERE` that describes the *bad* case, and count how many rows match.

| Check | The rule | The suspicious rows |
|---|---|---|
| Valid duration | A trip must end after it starts | `ended_at <= started_at` |
| Reasonable duration | Trips longer than a day are suspicious | duration over 1,440 minutes |
| Date range | The file should only hold May 2025 | `started_at` outside May 2025 |
| Required fields | Every trip needs an id | `ride_id IS NULL` |
| Sensible location | Coordinates should fall in the Washington area | latitude or longitude outside the region, or missing |

For each check, record: the **count**, and **what you think should happen** (drop the rows? fix them? keep them and flag them?). A count of **zero is a good result**: it means the rule holds.

Why not just delete the bad rows? Because the raw layer is your untouched copy, and because a "bad" row is not always a real error. Decisions about cleaning are made deliberately, in the next layer, and written down.

---

## 8. The Order SQL Really Runs In
You write `SELECT` first, but the database does not run it first. The logical order is:

1. `FROM`: pick the table
2. `WHERE`: filter the rows
3. `SELECT`: calculate the columns
4. `ORDER BY`: sort
5. `LIMIT`: keep the first rows

This explains why an alias created in `SELECT` (like `trip_minutes`) often **cannot be used inside `WHERE`**: when `WHERE` runs, the alias does not exist yet. The safe habit is to repeat the calculation in `WHERE`. (`ORDER BY` runs after `SELECT`, so it can use the alias.) Tomorrow you will add `GROUP BY` and `HAVING` to this order.

---

## 9. A Note About SQL "Dialects"
The core of SQL (`SELECT`, `WHERE`, `ORDER BY`) works the same everywhere. But some functions have different names or behave slightly differently between databases. For example, the function that returns a weekday name can give `Monday` in one database and `Mon` in another. When you move to Snowflake in Week 3, you will see a few of these differences. It is a normal part of data engineering: always check the documentation for the tool you are using.

---

## Vocabulary for Day 3
| Term | Meaning |
|---|---|
| Filter | Keeping only the rows that meet a condition |
| Operator | A symbol like `=`, `>`, `AND`, `IN` used to build conditions |
| Wildcard (`%`) | Stands for "anything" in a `LIKE` pattern |
| Alias | A name you give a column or calculation with `AS` |
| Timestamp | A value holding both a date and a time |
| Half-open range | A time window that includes its start and excludes its end |
| `NULL` | A missing or unknown value |
| Data profiling | Inspecting data to learn what it contains and where the problems are |
| Data quality check | A test of whether data follows its expected rules |
| Dialect | The small differences in SQL between database products |

---

## What Success Looks Like Today
By the end of Day 3, you should be able to:
- Filter rows by text, numbers, and dates, and combine conditions safely with `AND`, `OR`, `IN`, and parentheses
- Sort results with `ORDER BY` and explain why `LIMIT` needs it for a real "top N"
- Calculate a new column, such as trip duration, and give it an alias
- Filter a full day correctly, and explain why `BETWEEN` on dates is a trap
- Test for `NULL` with `IS NULL`, and explain why `= NULL` fails
- Run data quality checks by counting suspicious rows, and log what you found without changing the raw data