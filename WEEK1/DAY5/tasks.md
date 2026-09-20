# Week 1, Day 5 — Tasks
## Joins, the Mini-Project, and Thinking Like a Data Engineer (6-Hour Session)

Read today's Concepts doc first. **Pace:** this session is planned for up to 6 hours so you can think and write slowly. A confident learner can finish in about 3 hours. The time note beside each block shows both.

Today is different from the other days. There is **very little SQL** (three short blocks of queries and one script), and most of the work is **writing and thinking**. Wherever you see ✍️, stop and write your answer in your notepad **before** you run the next query. Predicting first, then checking, is how you learn what you really understand.

**There are no trick questions.** A short, honest answer that shows your reasoning is better than a long answer that sounds impressive. If you are unsure, say so and explain why.

**Before you start:** open DBeaver, make sure your DuckDB connection is active, and open a new SQL script (right-click the connection → **SQL Editor → New SQL script**). Run each query **one at a time**: click inside it and press **Ctrl + Enter** (Mac: **Cmd + Enter**).

---

## Block 1 — Predict, Then Run: Joining Trips to Stations (Hours 1–2 · confident pace: about 50 minutes)

### Step 1.1 — Choose the join key
The trips table and the stations table each describe stations, but in different columns. Look at Day 2, Step 3.5, where you compared them. Run these two queries again:
```sql
SELECT DISTINCT start_station_id, start_station_name
FROM raw_trips
WHERE start_station_id IS NOT NULL
LIMIT 5;

SELECT station_id, short_name, name
FROM raw_stations
LIMIT 5;
```
✍️ **Write:** Which column in `raw_trips` and which column in `raw_stations` do you think hold the same kind of value? Give **two candidate keys** (for example, an id-to-id match and a name-to-name match). If your stations table has no `short_name`, list the columns you do have and pick your best candidates.

### Step 1.2 — Is the key unique on the stations side?
✍️ **Predict:** Do you expect any station to appear twice in `raw_stations`? Why does it matter?

Run this check for each stations-side candidate (change `short_name` to `name` for the second one):
```sql
SELECT short_name, COUNT(*) AS copies
FROM raw_stations
GROUP BY short_name
HAVING COUNT(*) > 1;
```
✅ **Checkpoint:** **no rows** means each value appears once, so the key is unique. If you see rows, 📝 write down how many. A key with duplicates is dangerous, because it causes fan-out.

### Step 1.3 — Test both candidate keys
Each test counts how many trips find a partner in the stations table.

**Test A: match on the station id** (it turns both sides into text so they can be compared):
```sql
SELECT COUNT(*)           AS trips_checked,
       COUNT(s.short_name) AS matched_to_a_station
FROM raw_trips AS t
LEFT JOIN raw_stations AS s
       ON CAST(t.start_station_id AS VARCHAR) = CAST(s.short_name AS VARCHAR)
WHERE t.start_station_id IS NOT NULL;
```

**Test B: match on the station name:**
```sql
SELECT COUNT(*)      AS trips_checked,
       COUNT(s.name) AS matched_to_a_station
FROM raw_trips AS t
LEFT JOIN raw_stations AS s
       ON t.start_station_name = s.name
WHERE t.start_station_name IS NOT NULL;
```
✅ **Checkpoint:** each test returns one row with two numbers. `matched_to_a_station` is never larger than `trips_checked`. (If Test A gives a column error, your stations table has no `short_name`. Skip Test A and note that.)

📝 **Work out the match rate** for each: `matched_to_a_station ÷ trips_checked × 100`.

✍️ **Write (3 to 4 sentences):** Which key did you choose, and why? Use your two match rates as evidence. Also say what you think explains the trips that did **not** match. (Hint: think about when the trips happened, and when you downloaded the station list.)

> **Use your chosen join condition below.** Wherever you see `YOUR_JOIN_CONDITION`, paste one of these:
> - Id match: `CAST(t.start_station_id AS VARCHAR) = CAST(s.short_name AS VARCHAR)`
> - Name match: `t.start_station_name = s.name`

### Step 1.4 — `INNER`, `LEFT`, and `RIGHT`: predict first
Here are three versions of the same join.

✍️ **Predict before running:** Which will give the **most** rows, and which the **fewest**? Will the `LEFT JOIN` give exactly the same number as the total number of trips? What do you think the `RIGHT JOIN` count is made of? Write your guesses.

```sql
SELECT COUNT(*) AS inner_rows
FROM raw_trips AS t
INNER JOIN raw_stations AS s ON YOUR_JOIN_CONDITION;

SELECT COUNT(*) AS left_rows
FROM raw_trips AS t
LEFT JOIN raw_stations AS s ON YOUR_JOIN_CONDITION;

SELECT COUNT(*) AS right_rows
FROM raw_trips AS t
RIGHT JOIN raw_stations AS s ON YOUR_JOIN_CONDITION;
```
✅ **Checkpoints:**
- `left_rows` is **exactly equal** to your total number of trips (`total_trips` from Day 4, Step 1.1). If it is bigger, rows multiplied, and something is wrong with the key.
- `inner_rows` is smaller than `left_rows`.
- `right_rows` is a little larger than `inner_rows`, by roughly the number of stations that had no trips.

✍️ **Write (3 to 4 sentences):** Compare your predictions with the results. Where were you right or wrong? Which trips are "missing" from the `INNER JOIN`, and why could that be dangerous in a report?

### Step 1.5 — Look at the trips that did not match
```sql
SELECT t.start_station_name,
       COUNT(*) AS unmatched_trips
FROM raw_trips AS t
LEFT JOIN raw_stations AS s ON YOUR_JOIN_CONDITION
WHERE t.start_station_name IS NOT NULL
  AND s.station_id IS NULL
GROUP BY t.start_station_name
ORDER BY unmatched_trips DESC
LIMIT 10;
```
(`s.station_id IS NULL` picks the trips where no station was found. If your stations table has no `station_id` column, use `s.name IS NULL` instead.)

✅ **Checkpoint:** up to 10 station names with counts. If you get no rows, everything matched, which is a fine result. Write that down.

✍️ **Write (2 to 3 sentences):** What do you notice about the unmatched stations? Give one reason a trip in May 2025 might refer to a station that is not in today's list.

---

## Block 2 — The Mini-Project: Top Stations (Hour 3 · confident pace: about 35 minutes)

### Step 2.1 — Build the summary table
Paste this into DBeaver and replace `YOUR_JOIN_CONDITION` with your chosen condition. If a column such as `lat`, `lon`, or `capacity` has a different name in your stations table, use the names you saw in `DESCRIBE raw_stations`.

```sql
CREATE OR REPLACE TABLE top_stations AS
SELECT t.start_station_name AS station_name,
       s.lat,
       s.lon,
       s.capacity,
       COUNT(*) AS trips,
       ROUND(AVG(datediff('second', t.started_at, t.ended_at) / 60.0), 1) AS avg_trip_minutes
FROM raw_trips AS t
LEFT JOIN raw_stations AS s
       ON YOUR_JOIN_CONDITION
WHERE t.start_station_name IS NOT NULL      -- we need a station to summarize by
  AND t.ended_at > t.started_at             -- leave out zero or negative durations (Day 3, check 1)
  AND t.started_at >= '2025-05-01'          -- keep only May 2025 (Day 3, check 3)
  AND t.started_at <  '2025-06-01'
GROUP BY t.start_station_name, s.lat, s.lon, s.capacity
HAVING COUNT(*) >= 100                      -- leave out stations with very few trips
ORDER BY trips DESC
LIMIT 10;
```
**Grain of this table:** one row per start station.

Then look at it:
```sql
SELECT * FROM top_stations ORDER BY trips DESC;
```
✅ **Checkpoints:**
- 10 rows (or fewer, if fewer stations qualify), sorted by `trips`, busiest first
- The busiest station's `trips` is **equal to or slightly smaller than** the number you wrote down on Day 4, Step 3.2. It can be smaller because today's query also removes bad-duration and out-of-range trips. It must **never be larger**. If it is larger, rows multiplied in the join.
- Some stations may show `NULL` for `lat`, `lon`, and `capacity`. Those are stations that did not match the stations table.

✍️ **Write (2 to 3 sentences):** Explain in your own words why we used a `LEFT JOIN` here instead of an `INNER JOIN`.

### Step 2.2 — Record how many trips did not match
```sql
SELECT COUNT(*) AS trips_without_station_details
FROM raw_trips AS t
LEFT JOIN raw_stations AS s ON YOUR_JOIN_CONDITION
WHERE t.start_station_name IS NOT NULL
  AND s.station_id IS NULL;
```
📝 **Write down the number.** It goes in your README and in your script as a comment. (Use `s.name IS NULL` if you have no `station_id` column.)

### Step 2.3 — Save the script
1. In VS Code, right-click the `sql` folder → **New File** → name it `04_top_stations.sql`.
2. Paste in three things, in this order, with your real join condition in place of `YOUR_JOIN_CONDITION`:
   - A comment block at the top: `-- Week 1 mini-project: top start stations, May 2025` plus one line naming the join key you chose and why
   - The `CREATE OR REPLACE TABLE top_stations ...` statement
   - The unmatched-trips query from Step 2.2, with a comment above it: `-- Join check: trips with no matching station`
3. Save with **Ctrl + S** (Cmd + S on Mac).

---

## Block 3 — Thoughtful Questions (Hours 4–6 · confident pace: about 75 minutes)

This is the main part of today. Open `README.md` in VS Code, add a section called `## Week 1 Reflection`, and answer the questions below in your own words. Aim for **2 to 4 sentences per question**. You do not need perfect grammar. You need honest reasoning.

If you get stuck starting, use a sentence starter: *"I think ... because ..."*, *"The evidence I used was ..."*, *"If this went wrong, ..."*, *"I am not sure about ..., because ..."*.

### Part A: About your joins
1. **Join key:** In your own words, what is a join key? What makes one key better than another? Which did you use, and what evidence supports your choice?
2. **Silent loss:** Your `INNER JOIN` had fewer rows than your `LEFT JOIN`. Imagine a manager reads a report built on the `INNER JOIN`. What might they wrongly believe, and how could you avoid that?
3. **Fan-out:** A colleague says, *"The top-stations table shows 10,000 trips for one station, but the raw table only has 5,000 trips for it."* What is the most likely cause? Describe the two checks you would run to find out.

### Part B: About your data
4. **Time mismatch:** The trips are from May 2025, but the station list was downloaded later. What problems could that cause? How might a professional pipeline handle it?
5. **Your quality log:** Look at your Day 3 quality-check table. Which findings affect the top-stations table? Which ones did you filter out in today's query, and which did you leave alone? Would you make the same choice in real life? Why or why not?

### Part C: About the pipeline
6. **Where does it belong?** Is `top_stations` part of the raw layer, a staging layer, or a mart? Explain your reasoning, and state its grain in one sentence.
7. **Reproducibility:** Suppose a teammate clones your GitHub repository tomorrow. Could they rebuild your tables? List what they would need. What might stop them from doing it, and what would you improve?
8. **Why keep raw data untouched?** You have found bad data (durations, missing stations). Why did we only *record* these problems in the raw layer instead of deleting the rows? Give a real situation where editing the raw data would cause a problem.

### Part D: Looking back at Week 1
9. **Three big ideas:** List three ideas from this week that you now understand better. For each, use a data engineering word (for example, *ingestion*, *raw layer*, *reconciliation*, *grain*, *idempotent*) and explain it in one sentence.
10. **Your traps:** This week showed several silent traps: `LIMIT` without `ORDER BY`, `BETWEEN` on dates, `= NULL`, `WHERE COUNT(*)`, and join fan-out. Which one do you think you would be most likely to fall into in real work? What habit would help you catch it?
11. **A stuck moment:** Describe one thing that confused you or went wrong this week, and how you got past it (or what you would do differently next time).
12. **Looking ahead:** In Week 2 you will clean the data. From your notes and quality log, list **three problems** you expect to have to deal with, and say how you might treat each one.

### Part E: Explain it to a friend
13. In **6 to 8 sentences**, explain to a friend who has never heard of data engineering what a **data pipeline** is. Use one example from your bikeshare project, and mention at least three of these words: *ingestion*, *raw layer*, *data quality check*, *join*, *summary table*, *version control*.

---

## Block 4 — Finish and Commit (Hour 6 · confident pace: about 15 minutes)
1. In `README.md`, update **Progress** with:
   `- Day 5: joined trips to stations; built top_stations; Week 1 reflection written`
2. Add a short **Week 1 Results** section above your reflection with:
   - The join key you chose and its match rate
   - The number of trips with no matching station (from Step 2.2)
   - The name of your busiest station and its number of trips
3. In **Source Control**, check that you see `04_top_stations.sql` and `README.md`, and **no** data files. Commit with the message `Add top stations mini-project and Week 1 reflection`, then click **Sync Changes**.

✅ **Checkpoint:** On GitHub, your repository shows `sql/01_load_raw.sql`, `02_explore.sql`, `03_summaries.sql`, `04_top_stations.sql`, your `.gitignore`, and a README with your notes, quality log, results, and reflection. There are no `.duckdb`, `.csv`, `.json`, or `.parquet` files.

---

## If You Get Stuck

| Problem | What to try |
|---|---|
| A column such as `short_name`, `station_id`, `lat`, or `capacity` is not found | Run `DESCRIBE raw_stations;` to see the real column names, and use those. |
| Both match rates are very low | That is a finding, not a failure. Use the better key, note the low match rate in your README, and explain what you think causes it. |
| `left_rows` is bigger than your total trips | The key is not unique on the stations side. Recheck Step 1.2 and choose a different key. |
| `top_stations` has no rows | Lower `HAVING COUNT(*) >= 100` to `>= 20` and run again. |
| The type comparison fails in the join | Make sure both sides use `CAST(... AS VARCHAR)`, as in Test A. |
| You are not sure how to answer a question | Write what you *do* understand, then write what you are unsure about. That is a good answer. |
| Git shows a data file in Source Control | Check your `.gitignore` contains `data/raw/` and `*.duckdb`, and save it. |

If you are stuck on one step for more than 15 minutes, take a screenshot and ask your instructor.

---

## End-of-Week Checklist
- [ ] Compared two candidate join keys and chose one with evidence
- [ ] Checked the stations-side key is unique
- [ ] Compared `INNER`, `LEFT`, and `RIGHT` row counts against written predictions
- [ ] Built `top_stations` and checked its busiest count against Day 4
- [ ] Recorded the number of trips with no matching station
- [ ] `04_top_stations.sql` saved, committed, and pushed
- [ ] All 13 reflection questions answered in the README
- [ ] Repository contains all four SQL scripts, `.gitignore`, and README, with no data files
- [ ] `raw_trips` and `raw_stations` still untouched

### Week 1 Deliverable Recap
- A GitHub repository with your SQL scripts committed
- A DuckDB database with raw trips and stations tables
- Row-count checks showing every row loaded
- A top-stations summary table, built by a saved script
- A written record of what you noticed about the data, including problems found