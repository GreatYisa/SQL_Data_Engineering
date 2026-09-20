# Week 1, Day 4 — Tasks
## Summarize the Trips and Build Your First Summary Table (6-Hour Session)

Read today's Concepts doc first. **Pace:** this session is planned for up to 6 hours so you can work slowly and carefully. A confident learner can finish in about 3 hours. The time note beside each block shows both.

Today has **four blocks**. Each one goes deep on one idea. Everything runs against your `raw_trips` table. The only new thing you create is a small summary table at the very end, and you will prove that it agrees with the raw data.

**Before you start:** open DBeaver, make sure your DuckDB connection is active, and open a new SQL script (right-click the connection → **SQL Editor → New SQL script**). Run each query **one at a time**: click inside it and press **Ctrl + Enter** (Mac: **Cmd + Enter**).

📝 **Keep a notepad open** to write down numbers as you go. They go into your README at the end.

---

## Block 1 — Summarize the Whole Table (Hour 1 · confident pace: about 30 minutes)

### Step 1.1 — Count in three ways
```sql
SELECT COUNT(*)                        AS total_trips,
       COUNT(start_station_name)       AS trips_with_start_station,
       COUNT(DISTINCT start_station_name) AS different_start_stations
FROM raw_trips;
```
✅ **Checkpoints:**
- `total_trips` equals the row count you found on Day 2 (`table_rows`).
- `trips_with_start_station` is **smaller** than `total_trips`, because `COUNT(column)` skips `NULL` values.
- `different_start_stations` is in the hundreds.

📝 **Write down all three numbers.** Then work out `total_trips` minus `trips_with_start_station`. It should equal the "right way" `IS NULL` count from Day 3, Step 3.1. If it does, two different methods agree, and that is a reconciliation.

### Step 1.2 — Summarize trip length
```sql
SELECT ROUND(AVG(datediff('second', started_at, ended_at) / 60.0), 1)    AS avg_minutes,
       ROUND(median(datediff('second', started_at, ended_at) / 60.0), 1) AS median_minutes,
       ROUND(MIN(datediff('second', started_at, ended_at) / 60.0), 1)    AS shortest_minutes,
       ROUND(MAX(datediff('second', started_at, ended_at) / 60.0), 1)    AS longest_minutes
FROM raw_trips;
```
✅ **Checkpoints:**
- The **average** is noticeably larger than the **median**. A few very long trips pull the average up, while the median stays close to a normal trip.
- `longest_minutes` matches the longest trip you found on Day 3, Step 1.5.
- `shortest_minutes` may be zero or below. This links to your Day 3 quality check 1.

📝 **Write down** the average and the median. In your notes, write one sentence explaining which of the two you would show on a dashboard, and why.

---

## Block 2 — `GROUP BY`: One Row per Group (Hours 2–3 · confident pace: about 60 minutes)

### Step 2.1 — Trips by rider type
```sql
SELECT member_casual,
       COUNT(*) AS trips,
       ROUND(AVG(datediff('second', started_at, ended_at) / 60.0), 1)    AS avg_minutes,
       ROUND(median(datediff('second', started_at, ended_at) / 60.0), 1) AS median_minutes
FROM raw_trips
GROUP BY member_casual
ORDER BY trips DESC;
```
✅ **Checkpoint:** 2 rows (`member` and `casual`), busiest first. 📝 **Write down** both trip counts. **Add them together.** The total must equal `total_trips` from Step 1.1. If it does, no rows were lost by grouping.

*Grain check:* one row here means "one rider type". Write that in your notes.

### Step 2.2 — Trips by bike type
```sql
SELECT rideable_type,
       COUNT(*) AS trips
FROM raw_trips
GROUP BY rideable_type
ORDER BY trips DESC;
```
✅ **Checkpoint:** 2 or 3 rows, one per bike type.

### Step 2.3 — Group by two columns
```sql
SELECT member_casual,
       rideable_type,
       COUNT(*) AS trips
FROM raw_trips
GROUP BY member_casual, rideable_type
ORDER BY member_casual, rideable_type;
```
✅ **Checkpoint:** one row per combination (4 to 6 rows).

### Step 2.4 — Trips per day
```sql
SELECT CAST(started_at AS DATE) AS trip_date,
       COUNT(*)                 AS trips
FROM raw_trips
GROUP BY CAST(started_at AS DATE)
ORDER BY trip_date;
```
✅ **Checkpoint:** about 31 rows, one per day of May (there may be one or two extra rows if your Day 3 check 3 found trips outside May). 📝 **Write down** the date with the most trips.

### Step 2.5 — Trips per hour of the day
```sql
SELECT EXTRACT(hour FROM started_at) AS start_hour,
       COUNT(*)                      AS trips
FROM raw_trips
GROUP BY EXTRACT(hour FROM started_at)
ORDER BY start_hour;
```
✅ **Checkpoint:** 24 rows (hours 0 to 23). Look at the pattern. You will probably see a morning peak and an evening peak, which looks like commuting. 📝 **Write down** which hours are busiest.

### Step 2.6 — Your turn
Write a query that shows, for each **weekday** (`dayname(started_at)`), the number of trips and the average trip length in minutes (rounded to 1 decimal). Show the busiest day first.

✅ **Checkpoint:** 7 rows, one per weekday.

<details>
<summary>Check your answer</summary>

```sql
SELECT dayname(started_at) AS weekday,
       COUNT(*)            AS trips,
       ROUND(AVG(datediff('second', started_at, ended_at) / 60.0), 1) AS avg_minutes
FROM raw_trips
GROUP BY dayname(started_at)
ORDER BY trips DESC;
```
</details>

---

## Block 3 — `WHERE` vs. `HAVING`, and Checking Your Data (Hours 4–5 · confident pace: about 50 minutes)

### Step 3.1 — `WHERE` filters rows before grouping
```sql
SELECT member_casual,
       COUNT(*) AS electric_trips
FROM raw_trips
WHERE rideable_type = 'electric_bike'
GROUP BY member_casual;
```
✅ **Checkpoint:** 2 rows. Their numbers add up to the electric bike trip count from Step 2.2.

### Step 3.2 — `HAVING` filters groups after grouping
```sql
SELECT start_station_name,
       COUNT(*) AS trips
FROM raw_trips
WHERE start_station_name IS NOT NULL
GROUP BY start_station_name
HAVING COUNT(*) > 500
ORDER BY trips DESC
LIMIT 10;
```
✅ **Checkpoint:** up to 10 rows, the busiest start stations, each with more than 500 trips, sorted highest first. If you get **no rows**, change `500` to `100` and run it again. 📝 **Write down** the busiest station and its number of trips. (You will use this exact query as the base of your mini-project on Day 5.)

### Step 3.3 — See the classic error
Run this **wrong** query on purpose:
```sql
SELECT start_station_name,
       COUNT(*) AS trips
FROM raw_trips
WHERE COUNT(*) > 500
GROUP BY start_station_name;
```
✅ **Checkpoint:** a red error message saying that `WHERE` cannot contain aggregates. 📝 **Write one sentence** in your notes explaining why, using the order SQL runs in (`WHERE` runs before the groups exist).

### Step 3.4 — Look for duplicate trips
```sql
SELECT ride_id,
       COUNT(*) AS copies
FROM raw_trips
GROUP BY ride_id
HAVING COUNT(*) > 1
LIMIT 10;
```
✅ **Checkpoint:** most likely **no rows**, which means every `ride_id` appears once (no duplicates). If you do get rows, 📝 write down how many, and what you think should happen to them. You will handle that in Week 2.

### Step 3.5 — Your turn
Find the **5 end stations with the longest average trip**, but only look at stations with **at least 200 trips**, and leave out trips with no end station. Show the station name, the number of trips, and the average trip length in minutes (1 decimal).

✅ **Checkpoint:** up to 5 rows, sorted by average length, longest first.

<details>
<summary>Check your answer</summary>

```sql
SELECT end_station_name,
       COUNT(*) AS trips,
       ROUND(AVG(datediff('second', started_at, ended_at) / 60.0), 1) AS avg_minutes
FROM raw_trips
WHERE end_station_name IS NOT NULL
GROUP BY end_station_name
HAVING COUNT(*) >= 200
ORDER BY avg_minutes DESC
LIMIT 5;
```
</details>

---

## Block 4 — Build a Summary Table, Then Save and Commit (Hour 6 · confident pace: about 40 minutes)

### Step 4.1 — Create your first summary table
```sql
CREATE OR REPLACE TABLE daily_trip_summary AS
SELECT CAST(started_at AS DATE) AS trip_date,
       member_casual,
       rideable_type,
       COUNT(*) AS trips,
       ROUND(AVG(datediff('second', started_at, ended_at) / 60.0), 1) AS avg_trip_minutes
FROM raw_trips
GROUP BY CAST(started_at AS DATE), member_casual, rideable_type;
```
**Grain of this table:** one row per **day**, per **rider type**, per **bike type**.

✅ **Checkpoint:** no error. Right-click your connection → **Refresh** (or press **F5**) and `daily_trip_summary` appears under **Tables**.

### Step 4.2 — Prove the summary agrees with the raw data
Run these two queries:
```sql
SELECT COUNT(*) AS summary_rows FROM daily_trip_summary;
SELECT SUM(trips) AS total_in_summary FROM daily_trip_summary;
```
✅ **Checkpoints:**
- `summary_rows` is small, roughly 120 to 190 rows, compared with hundreds of thousands of raw rows.
- `total_in_summary` is **exactly equal** to `total_trips` from Step 1.1.

📝 **Write down** the summary row count and the raw row count. This is your **reconciliation**: no trip was lost or double-counted on the way from raw data to summary.

### Step 4.3 — Use it
```sql
SELECT *
FROM daily_trip_summary
ORDER BY trip_date, member_casual, rideable_type
LIMIT 10;
```
✅ **Checkpoint:** 10 rows showing daily figures. A dashboard would read this small table instead of the huge raw one.

### Step 4.4 — Save your queries as a script
1. In DBeaver, click in the editor, press **Ctrl + A**, then **Ctrl + C** (Cmd on Mac).
2. In VS Code, right-click the `sql` folder → **New File** → name it `03_summaries.sql`.
3. Paste the queries in and save.
4. Add this as the first line: `-- Day 4: aggregates, GROUP BY, HAVING, and the daily_trip_summary table`

> **Tip for reproducibility:** because the summary is created with `CREATE OR REPLACE TABLE ... AS SELECT`, anyone can recreate it by running this script. The table lives in your `.duckdb` file, which Git ignores, but the *code* that builds it is saved on GitHub.

### Step 4.5 — Write your notes and commit
1. Open `README.md`. Under **Progress**, add:
   `- Day 4: aggregated trips with GROUP BY; built daily_trip_summary and reconciled it with raw_trips`
2. Add a section called `## Day 4 Notes` with a short list of your findings:
   - Trips by rider type (both numbers)
   - Average vs. median trip length
   - The busiest day, the busiest hours, and the busiest start station
   - Raw row count vs. summary total (they should match)
3. Below the list, write 3 to 4 sentences answering:
   - What does `GROUP BY` do, and what is the **grain** of `daily_trip_summary`?
   - What is the difference between `WHERE` and `HAVING`?
   - Why was the average trip length so much bigger than the median?
   - Why would a data engineer build a summary table instead of letting a dashboard read the raw data?
4. In **Source Control**, check you see `03_summaries.sql` and `README.md` (and no data files). Commit with the message `Add Day 4 summary queries and daily_trip_summary table`, then click **Sync Changes**.

✅ **Checkpoint:** On GitHub, `sql/03_summaries.sql` is there and your README shows the Day 4 notes.

---

## If You Get Stuck

| Problem | What to try |
|---|---|
| Error: column must appear in the `GROUP BY` or be used in an aggregate | Every column in `SELECT` must be in `GROUP BY` or inside `COUNT`, `AVG`, and so on. Add the missing column to `GROUP BY`, or wrap it in an aggregate. |
| The `HAVING` query returns no rows | The threshold may be too high for your data. Lower the number (for example, from 500 to 100). |
| Your bike type or rider type spelling differs | Use exactly the values from Day 3, Step 1.1. |
| `median` is not recognized | Use `quantile_cont(datediff('second', started_at, ended_at) / 60.0, 0.5)` instead, which gives the same middle value. |
| The summary total does not match the raw row count | Check that `GROUP BY` lists every non-aggregate column in your `SELECT`, that you did not add a `WHERE` to the summary, and rerun the `CREATE OR REPLACE TABLE`. |
| The wrong query runs when you press Ctrl + Enter | Click *inside* the query you want, and make sure each query ends with a semicolon and has a blank line between it and the next. |
| A query takes more than a minute | Press the red **Stop** button in DBeaver and ask your instructor. |
| Git shows a data file in Source Control | Check your `.gitignore` contains `data/raw/` and `*.duckdb`, and save it. |

If you are stuck on one step for more than 15 minutes, take a screenshot of the error and ask your instructor.

---

## End-of-Day Checklist
- [ ] Counted rows in three ways and checked `COUNT(*) − COUNT(column)` against the `NULL` count
- [ ] Compared average and median trip length
- [ ] Wrote `GROUP BY` queries by rider type, bike type, day, hour, and weekday
- [ ] Checked that group totals add back up to the raw row count
- [ ] Used `WHERE` and `HAVING` correctly, and read the "aggregates not allowed in `WHERE`" error
- [ ] Looked for duplicate `ride_id` values
- [ ] Built `daily_trip_summary` and reconciled its total with `raw_trips`
- [ ] `03_summaries.sql` saved, committed, and pushed
- [ ] README updated with Day 4 findings and notes