# Week 1, Day 3 — Tasks
## Find the Rows You Need, and the Rows That Look Wrong (6-Hour Session)

Read today's Concepts doc first. **Pace:** this session is planned for up to 6 hours so you can work slowly and carefully. A confident learner can finish in about 3 hours. The time note beside each block shows both.

Today has **four blocks**. Each one goes deep on one idea, and everything runs against the `raw_trips` table you loaded yesterday. **You will not change any data today.** You only read it, count things, and write down what you find.

**Before you start:** open DBeaver, make sure your DuckDB connection is active, and open a new SQL script (right-click the connection → **SQL Editor → New SQL script**). Run each query **one at a time**: click inside it and press **Ctrl + Enter** (Mac: **Cmd + Enter**).

📝 **Keep a notepad open.** You will write down several numbers today. They go into your README at the end.

---

## Block 1 — Find the Rows You Need (Hours 1–2 · confident pace: about 50 minutes)

### Step 1.1 — Discover the exact values in a column
Before you filter on text, find out exactly which values exist:
```sql
SELECT DISTINCT rideable_type FROM raw_trips;
SELECT DISTINCT member_casual FROM raw_trips;
```
`DISTINCT` shows each different value once.

✅ **Checkpoint:** `member_casual` shows 2 values (`member` and `casual`). `rideable_type` shows 2 or 3 values (for example `classic_bike` and `electric_bike`). 📝 **Write them down exactly.** If yours are spelled differently from this guide, use *your* spelling in all the queries below.

### Step 1.2 — Filter by text
```sql
SELECT ride_id, rideable_type, member_casual, start_station_name
FROM raw_trips
WHERE member_casual = 'casual'
LIMIT 10;
```
✅ **Checkpoint:** 10 rows, and every row shows `casual` in the `member_casual` column.

Now add a second condition. Change the `WHERE` part to:
```sql
WHERE member_casual = 'casual'
  AND rideable_type = 'electric_bike'
```
✅ **Checkpoint:** 10 rows, all `casual` **and** `electric_bike`.

### Step 1.3 — Count the filtered rows
```sql
SELECT COUNT(*) AS casual_electric_trips
FROM raw_trips
WHERE member_casual = 'casual'
  AND rideable_type = 'electric_bike';
```
✅ **Checkpoint:** one row with one number. 📝 **Write it down.** (`COUNT(*)` with a `WHERE` counts only the rows that pass the filter.)

### Step 1.4 — Match part of a text value
```sql
SELECT ride_id, start_station_name
FROM raw_trips
WHERE start_station_name LIKE '%Union Station%'
LIMIT 10;
```
The `%` means "anything can be here". So this finds every station name that *contains* `Union Station`.

✅ **Checkpoint:** 10 rows, each with `Union Station` somewhere in the name. If you get no rows, `LIKE` is case-sensitive and the name may be spelled differently. Try `LIKE '%Union%'` instead.

### Step 1.5 — Calculate trip duration and sort
```sql
SELECT ride_id,
       started_at,
       ended_at,
       ROUND(datediff('second', started_at, ended_at) / 60.0, 1) AS trip_minutes
FROM raw_trips
ORDER BY trip_minutes DESC
LIMIT 10;
```
✅ **Checkpoint:** 10 rows with a new `trip_minutes` column, sorted from the **longest** trip down. 📝 **Write down** the longest trip length in minutes. If it is thousands of minutes, that is many hours or days. Keep that in mind for Block 3.

### Step 1.6 — Your turn
Write a query that shows the **10 longest trips** made by **casual riders on electric bikes** that lasted **more than 60 minutes**. Show `ride_id` and `trip_minutes`, longest first.

*Hint:* in `WHERE`, you cannot use the alias `trip_minutes`, because `WHERE` runs before the alias exists. Repeat the calculation instead: `datediff('minute', started_at, ended_at) > 60`.

✅ **Checkpoint:** up to 10 rows, all with `trip_minutes` above 60, sorted largest first.

<details>
<summary>Check your answer</summary>

```sql
SELECT ride_id,
       ROUND(datediff('second', started_at, ended_at) / 60.0, 1) AS trip_minutes
FROM raw_trips
WHERE member_casual = 'casual'
  AND rideable_type = 'electric_bike'
  AND datediff('minute', started_at, ended_at) > 60
ORDER BY trip_minutes DESC
LIMIT 10;
```
</details>

---

## Block 2 — Dates and Timestamps (Hours 3–4 · confident pace: about 50 minutes)

### Step 2.1 — Select one full day the safe way
```sql
SELECT COUNT(*) AS trips_on_may_10
FROM raw_trips
WHERE started_at >= '2025-05-10'
  AND started_at <  '2025-05-11';
```
✅ **Checkpoint:** one number, probably in the thousands. 📝 **Write it down.**

### Step 2.2 — See the `BETWEEN` trap for yourself
```sql
SELECT COUNT(*) AS between_version
FROM raw_trips
WHERE started_at BETWEEN '2025-05-10' AND '2025-05-10';
```
✅ **Checkpoint:** the number is **0**, or tiny (only trips that started at exactly midnight). Compare it with Step 2.1. Both queries were meant to ask about May 10, but only one gave a correct answer, and neither showed an error. 📝 **Write down** both numbers.

### Step 2.3 — Pull parts out of a timestamp
```sql
SELECT ride_id,
       started_at,
       EXTRACT(hour FROM started_at) AS start_hour,
       dayname(started_at)           AS start_day
FROM raw_trips
LIMIT 10;
```
✅ **Checkpoint:** 10 rows. `start_hour` is a number from 0 to 23, and `start_day` is a weekday name such as `Monday`.

### Step 2.4 — Combine time filters with other filters
Count the weekday morning-commute trips (starting in the 7, 8, or 9 o'clock hours, Monday to Friday):
```sql
SELECT COUNT(*) AS weekday_morning_trips
FROM raw_trips
WHERE EXTRACT(hour FROM started_at) BETWEEN 7 AND 9
  AND dayname(started_at) NOT IN ('Saturday', 'Sunday');
```
✅ **Checkpoint:** one number. 📝 **Write it down.**

*(Using `BETWEEN` on whole hours like 7 and 9 is fine, because hours are whole numbers. The trap only appears with dates and timestamps.)*

### Step 2.5 — Your turn
Count the trips by **casual** riders that happened on a **Saturday or Sunday** and started between **12:00 and 15:59** (hours 12 to 15).

✅ **Checkpoint:** one number. 📝 **Write it down.**

<details>
<summary>Check your answer</summary>

```sql
SELECT COUNT(*) AS casual_weekend_afternoon_trips
FROM raw_trips
WHERE member_casual = 'casual'
  AND dayname(started_at) IN ('Saturday', 'Sunday')
  AND EXTRACT(hour FROM started_at) BETWEEN 12 AND 15;
```
</details>

---

## Block 3 — `NULL` and Data Quality Checks (Hour 5 · confident pace: about 50 minutes)

### Step 3.1 — The `NULL` trap
Run both queries and compare:
```sql
SELECT COUNT(*) AS wrong_way FROM raw_trips WHERE start_station_name = NULL;
SELECT COUNT(*) AS right_way FROM raw_trips WHERE start_station_name IS NULL;
```
✅ **Checkpoint:** `wrong_way` is **0**. `right_way` is a bigger number, probably many thousands. 📝 **Write both down.** The first query ran without an error and gave a wrong answer.

### Step 3.2 — Is the missing data random?
Count the trips with no start station, split by bike type. Use your exact `rideable_type` values from Step 1.1:
```sql
SELECT COUNT(*) AS electric_no_station
FROM raw_trips
WHERE start_station_name IS NULL
  AND rideable_type = 'electric_bike';

SELECT COUNT(*) AS classic_no_station
FROM raw_trips
WHERE start_station_name IS NULL
  AND rideable_type = 'classic_bike';
```
✅ **Checkpoint:** two numbers. You will probably see that almost all the missing stations belong to one bike type. 📝 **Write down what you see, and what you think it means.** A `NULL` here may not be an error. It may just mean the trip did not start at a station.

### Step 3.3 — Run five data quality checks
Each check counts the rows that break a rule. Run all five and fill in the log table below.

```sql
-- Check 1: trips that do not end after they start
SELECT COUNT(*) AS check1_bad_duration
FROM raw_trips
WHERE ended_at <= started_at;

-- Check 2: trips longer than 24 hours (1,440 minutes)
SELECT COUNT(*) AS check2_very_long
FROM raw_trips
WHERE datediff('minute', started_at, ended_at) > 1440;

-- Check 3: trips that start outside May 2025
SELECT COUNT(*) AS check3_outside_may
FROM raw_trips
WHERE started_at <  '2025-05-01'
   OR started_at >= '2025-06-01';

-- Check 4: trips with no ride_id
SELECT COUNT(*) AS check4_missing_id
FROM raw_trips
WHERE ride_id IS NULL;

-- Check 5: missing coordinates, or outside the Washington area
SELECT COUNT(*) AS check5_bad_location
FROM raw_trips
WHERE start_lat IS NULL
   OR start_lng IS NULL
   OR start_lat NOT BETWEEN 38.5  AND 39.5
   OR start_lng NOT BETWEEN -77.6 AND -76.6;
```
✅ **Checkpoint:** five numbers. Some may be **0**, which is a good result, because it means the rule holds. Do not worry if some are not zero. Real data usually is not perfect.

For any check with a count above 0, look at some examples by changing `COUNT(*) AS ...` to `*` and adding `LIMIT 10`. For example:
```sql
SELECT *
FROM raw_trips
WHERE ended_at <= started_at
LIMIT 10;
```

📝 **Copy this log table into your notepad and fill it in:**

| # | Check | Rows found | What should we do about it? (a guess is fine) |
|---|---|---|---|
| 1 | Duration zero or negative | | |
| 2 | Trip longer than 24 hours | | |
| 3 | Start date outside May 2025 | | |
| 4 | Missing ride_id | | |
| 5 | Missing or out-of-area coordinates | | |
| 6 | Missing start station | | |

(For row 6, use your number from Step 3.1.)

### Step 3.4 — Check the date range of the file
```sql
SELECT MIN(started_at) AS first_trip,
       MAX(started_at) AS last_trip
FROM raw_trips;
```
`MIN` and `MAX` find the smallest and largest value. You will study functions like these properly tomorrow.

✅ **Checkpoint:** `first_trip` is on or close to May 1, 2025, and `last_trip` is on or close to May 31, 2025. This should agree with your Check 3 result.

---

## Block 4 — Save Your Work and Write Your Notes (Hour 6 · confident pace: about 25 minutes)

### Step 4.1 — Save your queries as a script
1. In the DBeaver editor, click in the text, press **Ctrl + A**, then **Ctrl + C** (Cmd on Mac) to copy everything.
2. In VS Code, right-click the `sql` folder → **New File** → name it `02_explore.sql`.
3. Paste the queries in and save with **Ctrl + S**.
4. Add this comment on the first line: `-- Day 3: filtering, dates, NULLs and data quality checks on raw_trips (read-only)`

### Step 4.2 — Write your notes in the README
1. Open `README.md`. Under **Progress**, add:
   `- Day 3: filtered raw_trips, ran data quality checks (no data changed)`
2. Add a section called `## Day 3 Notes`. Paste your completed **data quality log table** from Step 3.3 into it.
3. Below the table, write 3 to 4 sentences answering:
   - What is the difference between `= NULL` and `IS NULL`, and what happened when you used the wrong one?
   - What happened when you used `BETWEEN` on dates, and how do you avoid it?
   - What was the most interesting or surprising thing you found in the data?
   - Why do we only *record* problems in the raw layer instead of fixing them?

### Step 4.3 — Commit
In **Source Control**, check that you see `02_explore.sql` and `README.md` (and no data files). Commit with the message `Add Day 3 exploration queries and data quality log`, then click **Sync Changes**.

✅ **Checkpoint:** On GitHub, `sql/02_explore.sql` is there and the README shows your quality log.

---

## If You Get Stuck

| Problem | What to try |
|---|---|
| A `LIKE` query returns no rows | `LIKE` is case-sensitive and the name may differ. Try a shorter pattern such as `'%Union%'`. |
| "Column not found" | Run `DESCRIBE raw_trips;` to see the exact column names, and use those. |
| Your bike type or rider type values look different | Use exactly the spelling you wrote down in Step 1.1. |
| Error when using `trip_minutes` in `WHERE` | Aliases do not work in `WHERE`. Repeat the calculation there, as in Step 1.6. |
| The wrong query runs when you press Ctrl + Enter | Click *inside* the query you want, and make sure each query ends with a semicolon and has a blank line between it and the next. |
| A query takes more than a minute | Press the red **Stop** button in DBeaver and ask your instructor. Queries on this table should finish in seconds. |
| Only some rows show in the results panel | This is normal. DBeaver shows part of a large result. Use `COUNT(*)` to see totals. |
| Git shows a data file in Source Control | Check your `.gitignore` contains `data/raw/` and `*.duckdb`, and save it. |

If you are stuck on one step for more than 15 minutes, take a screenshot of the error and ask your instructor.

---

## End-of-Day Checklist
- [ ] Found the exact values of `rideable_type` and `member_casual` with `DISTINCT`
- [ ] Filtered by text with `=`, `AND`, and `LIKE`, and counted filtered rows
- [ ] Calculated `trip_minutes`, sorted with `ORDER BY`, and used it with `LIMIT`
- [ ] Selected one full day with a half-open range, and compared it with the `BETWEEN` trap
- [ ] Used `EXTRACT` and `dayname` to filter by hour and weekday
- [ ] Compared `= NULL` with `IS NULL`, and looked at where missing stations occur
- [ ] Ran five data quality checks and filled in the log table
- [ ] Checked the first and last trip date
- [ ] `02_explore.sql` saved, committed, and pushed
- [ ] README updated with the quality log and Day 3 notes
- [ ] No data changed in `raw_trips`