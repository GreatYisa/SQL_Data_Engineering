# Week 1, Day 5 — Concepts
## Joining Tables Safely, and Explaining Your Work Like a Data Engineer

---

## Question of the Day
**"How do I combine two tables safely, and how do I package my work so someone else can trust and re-run it?"**

Today is the last day of Week 1, and it is a thinking day. There is a small amount of SQL, and a lot of writing. That is on purpose. A data engineer's SQL is only half the job. The other half is **explaining what you did, what you checked, and what you are unsure about**, in words that a colleague can follow. Today you practice both.

```
 SOURCE FILES -> INGEST -> RAW LAYER -> STAGING -> MARTS -> DASHBOARD
   (Day 2)        (Day 2)    (Day 2-3)    (Week 2)   (Day 4-5)    (Week 6)
```

---

## 1. Why We Join Tables
Data is usually stored in separate tables so that each fact is written down once.

- The **trips** table records what happened: one row per ride
- The **stations** table describes places: one row per station, with its location and size

If every trip row also carried every detail about its station, the same station information would be repeated hundreds of thousands of times. Instead, the trip stores a short station id, and a **join** brings the station details in when you need them.

A table that records **events** (trips) is often called a **fact table**. A table that describes **things** (stations) is called a **reference table** or **dimension table**. You will study these properly in Week 4. For now, notice the pattern: events on one side, descriptions on the other.

---

## 2. The Join Key
A join matches rows from two tables using a **join key**: a column in each table that holds the same kind of value.

```sql
SELECT t.ride_id, s.name
FROM raw_trips AS t
LEFT JOIN raw_stations AS s
       ON CAST(t.start_station_id AS VARCHAR) = CAST(s.short_name AS VARCHAR);
```

Reading this:
- `raw_trips AS t` and `raw_stations AS s` give each table a short nickname (an **alias**), so you can say which table a column comes from (`t.ride_id`, `s.name`)
- `ON ... = ...` is the matching rule: a trip and a station are paired when the ids are equal
- `CAST(... AS VARCHAR)` turns both sides into text, so the two ids can be compared even if one was stored as a number and the other as text

**A good join key:**
- Means the same thing in both tables (a station id matches a station id, not a bike id)
- Has compatible types (text with text, number with number)
- Is **unique on the reference side**: each station appears only once in the stations table

Choosing a key is a decision, not a guess. A data engineer checks candidate keys against the data before joining. Today you will test two candidates and use the evidence to choose.

---

## 3. Join Types
The join type decides what happens to rows that **do not** find a match.

A tiny example. Three trips and two stations:

| Trips | station id |     | Stations | id | name |
|---|---|---|---|---|---|
| A | 100 | | | 100 | Union Station |
| B | 200 | | | 300 | Dupont Circle |
| C | *(missing)* | | | | |

| Join type | Keeps | Result for this example | Rows |
|---|---|---|---|
| `INNER JOIN` | Only rows that match on both sides | A + Union Station | **1** |
| `LEFT JOIN` | **All** rows from the left table (trips), plus matches from the right | A + Union Station; B + nothing; C + nothing | **3** |
| `RIGHT JOIN` | **All** rows from the right table (stations), plus matches from the left | A + Union Station; nothing + Dupont Circle | **2** |
| `FULL OUTER JOIN` | Everything from both sides | All four rows above, combined | **4** |

Where there is no match, the columns from the other table are filled with `NULL`.

**Which one should you use?** It depends on the question, but a very common pattern is: use `LEFT JOIN` from your main table (trips) so you never lose a trip just because its station is missing from the reference table.

---

## 4. `NULL` in Joins
Two important facts:

1. **`NULL` never equals anything, not even another `NULL`.** A trip with no station id cannot match any station, because `NULL = 100` is never true.
2. **An `INNER JOIN` silently drops unmatched rows.** If 20% of your trips have no station, an `INNER JOIN` gives you a table 20% smaller, and it shows no warning. Any total you calculate from it is now too low.

This is one of the most common ways for real reports to go wrong without anyone noticing.

---

## 5. Join Fan-Out: When Rows Multiply
Suppose the stations table accidentally contained station 100 **twice**. Then trip A would match **both** rows, and appear **twice** in the result. This is called **fan-out**. Every count and every average built on top of it is now wrong, and again there is no error message.

The defence is a habit, not a trick:
- **Before joining:** check that the key is unique on the reference side (`GROUP BY key HAVING COUNT(*) > 1` should return no rows, as on Day 4)
- **After a `LEFT JOIN`:** the row count should be **exactly the same** as the left table's row count. If it is bigger, rows have multiplied.

---

## 6. The Data Engineer's Join Checklist
Run through this every time you join:

1. **Is the key unique** on the reference side?
2. **What is the match rate?** How many rows found a partner? (For an important join, anything much below 100% needs an explanation.)
3. **Do the row counts make sense?** After a `LEFT JOIN`, the count should equal the left table.
4. **What do the unmatched rows look like?** Look at examples. Are they meaningful or broken?

Notice that this is the same idea as Day 2's row-count check: *trust, but verify.*

---

## 7. Reference Data Changes Over Time
Your trips are from **May 2025**. Your station list was downloaded **later**, and stations get added, moved, renamed, and removed. So some trips may refer to a station that no longer exists in today's list, or that has a new name.

This is a real and very common data engineering problem. The reference data describes the world *now*, but the events happened *then*. Professional pipelines deal with this by keeping dated versions (snapshots) of reference data. You will not solve it this week, but you should be able to **notice it and describe it**, which is what today's questions ask you to do.

---

## 8. Putting It Together: The Mini-Project
Your Week 1 mini-project is a **top stations summary table**. It brings together everything from the week:

| Step in the query | Skill | Day |
|---|---|---|
| Read from `raw_trips` | `SELECT`, the raw layer | 1–2 |
| Filter to sensible trips (dates in May, positive duration) | `WHERE`, data quality checks | 3 |
| Join to station details | `LEFT JOIN` | 5 |
| Count trips and average duration per station | `GROUP BY`, aggregates | 4 |
| Keep only stations with enough trips | `HAVING` | 4 |
| Show the top 10 | `ORDER BY`, `LIMIT` | 3 |
| Save it as a table and as a script | `CREATE OR REPLACE TABLE`, Git | 2, 4 |

**Grain** of the result: one row per start station. **Layer:** it is a small summary built on the raw data, a first taste of a mart.

---

## 9. Writing Is Part of the Job
Data engineers write constantly: README files, comments in SQL, notes on data quality problems, explanations of why a decision was made. Why?
- **Someone else will run your work** (or you, in six months, having forgotten everything)
- **Decisions need reasons.** "I filtered out trips shorter than one minute" is only useful if it also says why.
- **Writing exposes gaps in understanding.** If you cannot explain a step, you do not fully understand it yet.

Today's questions have no trick answers. They ask for your reasoning. A short, honest answer that shows how you thought is worth more than a long answer that sounds impressive.

---

## Vocabulary for Day 5
| Term | Meaning |
|---|---|
| Join | Combining rows from two tables based on a matching rule |
| Join key | The column in each table used to match rows |
| Alias | A short nickname for a table or column |
| `INNER` / `LEFT` / `RIGHT` / `FULL OUTER` join | Different rules for what happens to unmatched rows |
| Match rate | The share of rows that found a partner in the join |
| Fan-out | Rows multiplying because the join key is not unique |
| Fact table | A table of events (trips) |
| Reference (dimension) table | A table that describes things (stations) |
| Snapshot | A dated copy of data, used to track how it changes |

---

## What Success Looks Like Today
By the end of Day 5, you should be able to:
- Explain what a join key is, and choose one using evidence
- Describe how `INNER`, `LEFT`, and `RIGHT` joins differ, and predict which gives more rows
- Explain how `NULL` values and duplicate keys can silently produce wrong numbers
- Run the four-step join checklist
- Build and save the top-stations summary as a script
- Explain your choices and your doubts in clear, honest writing