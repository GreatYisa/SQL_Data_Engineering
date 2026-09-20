# Week 1, Day 2 — Tasks
## Load Your First Real Data Into the Raw Layer (6-Hour Session)

Read today's Concepts doc first. **Pace:** this session is planned for up to 6 hours so you can work slowly and carefully. A confident learner can finish in about 3 hours. The time note beside each block shows both.

Today has **four blocks**. There are fewer tasks than yesterday, but each one goes deeper: you download real data, load it, prove that the load worked, and save your script.

Today's data: **Capital Bikeshare trips for May 2025** (a real file with hundreds of thousands of rows) and the **Capital Bikeshare station list**.

**Before you start:** open VS Code on your `bikeshare-pipeline` folder, and open DBeaver.

---

## Block 1 — Download Your Source Files (Hour 1 )

### Step 1.1 — Download the trips file
1. In your browser, go to this address. The download starts on its own:
   `https://s3.amazonaws.com/capitalbikeshare-data/202505-capitalbikeshare-tripdata.zip`
2. Open your **Downloads** folder and **unzip** the file:
   - **Windows:** right-click the zip → **Extract All** → **Extract**
   - **Mac:** double-click the zip
3. Inside is a `.csv` file. **Move that CSV** into your project's `data/raw` folder. You can drag it in the VS Code Explorer panel, or move it in your file manager.

✅ **Checkpoint:** In VS Code Explorer, `data/raw` contains `202505-capitalbikeshare-tripdata.csv` (it appears grey because Git ignores it, which is what you want).

> **If the CSV has a different name:** use the name you actually see wherever this guide shows the file name.

### Step 1.2 — Download the stations file
1. In your browser, open:
   `https://gbfs.capitalbikeshare.com/gbfs/en/station_information.json`
   You will see a page full of text with curly braces. That is JSON.
2. Save it: press **Ctrl + S** (Mac: **Cmd + S**). Name the file exactly `station_information.json`, and save it inside your `data/raw` folder.

✅ **Checkpoint:** `data/raw` now contains `station_information.json`. The name must end in `.json`, not `.txt` or `.html`.

> **If it saved with the wrong ending, or saving is awkward:** open the terminal in VS Code (**Terminal → New Terminal**) and run one of these from your project folder:
> - Windows: `curl.exe -L -o data/raw/station_information.json https://gbfs.capitalbikeshare.com/gbfs/en/station_information.json`
> - Mac: `curl -L -o data/raw/station_information.json https://gbfs.capitalbikeshare.com/gbfs/en/station_information.json`

### Step 1.3 — Get the full path of each file
DuckDB running inside DBeaver needs the **full path** to each file.
1. In VS Code Explorer, right-click `202505-capitalbikeshare-tripdata.csv` → **Copy Path**. Paste it into a notepad file and label it `TRIPS_PATH`.
2. Do the same for `station_information.json` and label it `STATIONS_PATH`.
3. If you are on Windows, change every backslash `\` to a forward slash `/`. For example, `C:\Users\Sam\...` becomes `C:/Users/Sam/...`.

You will paste these paths into the SQL below wherever you see `TRIPS_PATH` or `STATIONS_PATH`. Keep the quotation marks around the path.

---

## Block 2 — Load the Trips Into the Raw Layer (Hours 2–3 )

In DBeaver, right-click your DuckDB connection → **SQL Editor → New SQL script**. Run each statement below **one at a time**: click inside it and press **Ctrl + Enter** (Mac: **Cmd + Enter**).

### Step 2.1 — Peek at the file without loading it
```sql
SELECT *
FROM read_csv('TRIPS_PATH', sample_size = -1)
LIMIT 5;
```
`read_csv` lets DuckDB read the file as if it were a table. `sample_size = -1` means "read the whole file before guessing column types".

✅ **Checkpoint:** 5 rows appear. The columns include `ride_id`, `rideable_type`, `started_at`, `ended_at`, `start_station_name`, `end_station_name`, and `member_casual`, plus station id and latitude/longitude columns. Some cells may show `[NULL]`, which means no value.

### Step 2.2 — See the schema DuckDB guessed
```sql
DESCRIBE SELECT *
FROM read_csv('TRIPS_PATH', sample_size = -1);
```
✅ **Checkpoint:** A list of all the columns with a type for each. `started_at` and `ended_at` should be `TIMESTAMP`, and latitude/longitude columns should be `DOUBLE`.

📝 **Write down** (in a notepad, for your README later) the types DuckDB chose for `started_at`, `start_station_id`, and `start_lat`.

### Step 2.3 — Count the rows in the source file
```sql
SELECT COUNT(*) AS source_rows
FROM read_csv('TRIPS_PATH', sample_size = -1);
```
📝 **Write down the number.** This is the number of trips in the source file. It will be in the hundreds of thousands.

### Step 2.4 — Load the raw table
```sql
CREATE OR REPLACE TABLE raw_trips AS
SELECT *,
       '202505-capitalbikeshare-tripdata.csv' AS source_file,
       current_timestamp                      AS loaded_at
FROM read_csv('TRIPS_PATH', sample_size = -1);
```
What the extra lines do: `source_file` records which file each row came from, and `loaded_at` records when it was loaded. Those two columns are what makes the raw layer traceable. Everything else is loaded exactly as it came.

✅ **Checkpoint:** No red error. Right-click your connection → **Refresh** (or press **F5**), expand it, and you see `raw_trips` under **Tables**.

### Step 2.5 — Validate the row count
```sql
SELECT COUNT(*) AS table_rows
FROM raw_trips;
```
✅ **Checkpoint:** `table_rows` is **exactly the same** as the `source_rows` number you wrote down in Step 2.3. This is your first real reconciliation.

### Step 2.6 — Prove the load is idempotent
1. Run the `CREATE OR REPLACE TABLE raw_trips ...` statement from Step 2.4 **a second time**.
2. Run the `SELECT COUNT(*) ...` statement from Step 2.5 again.

✅ **Checkpoint:** The number is **still the same**. Running the load twice did not double the rows. That is idempotency.

### Step 2.7 — Your turn
Write a query that shows **10 rows** from `raw_trips` with only these four columns: `ride_id`, `started_at`, `start_station_name`, `member_casual`.

✅ **Checkpoint:** 10 rows and 4 columns.

<details>
<summary>Check your answer</summary>

```sql
SELECT ride_id, started_at, start_station_name, member_casual
FROM raw_trips
LIMIT 10;
```
</details>

---

## Block 3 — Load the Stations Into the Raw Layer (Hour 4 )

### Step 3.1 — Peek at the JSON file
```sql
SELECT *
FROM read_json_auto('STATIONS_PATH');
```
✅ **Checkpoint:** Only **1 row** appears, with columns such as `last_updated`, `ttl`, `version`, and `data`. The `data` cell holds a long piece of text in curly braces. That one cell contains *all* the stations packed together. Click the cell to see it.

### Step 3.2 — Flatten the stations into a table
```sql
CREATE OR REPLACE TABLE raw_stations AS
SELECT s.*,
       'station_information.json' AS source_file,
       current_timestamp          AS loaded_at
FROM (
    SELECT unnest(data.stations) AS s
    FROM read_json_auto('STATIONS_PATH')
);
```
How to read it, from the inside out:
- `data.stations` is the list of stations inside the `data` field
- `unnest(...)` spreads that list into one row per station
- `s.*` turns each station's details into separate columns

✅ **Checkpoint:** No red error. After **F5** refresh, `raw_stations` appears under **Tables**.

### Step 3.3 — Check the stations table
Run these two queries:
```sql
SELECT COUNT(*) AS table_rows
FROM raw_stations;

SELECT * FROM raw_stations LIMIT 5;
```
✅ **Checkpoint:** The count is in the **hundreds** (around 800 is typical, and the exact number changes as stations are added). The 5 rows show station details such as an id, a name, and a location. A few columns may hold lists (shown in square brackets). You can ignore those for now.

### Step 3.4 — Validate against the source
```sql
SELECT len(data.stations) AS source_stations
FROM read_json_auto('STATIONS_PATH');
```
✅ **Checkpoint:** `source_stations` equals the `table_rows` count from Step 3.3. Another reconciliation passed.

### Step 3.5 — Look ahead (no answer needed yet)
Run these two queries and compare the results:
```sql
SELECT DISTINCT start_station_id, start_station_name
FROM raw_trips
WHERE start_station_id IS NOT NULL
LIMIT 5;

SELECT station_id, short_name, name
FROM raw_stations
LIMIT 5;
```
📝 **Notice and write down:** the trips table refers to a station by an id. Which column in `raw_stations` looks like the same kind of value? On Day 5 you will use this to join the two tables. If your stations table has no `short_name` column, write down which columns it has instead.

---

## Block 4 — CSV vs. Parquet, Then Save and Commit (Hours 5–6 · confident pace: about 40 minutes)

### Step 4.1 — Export the raw table to Parquet
Use the same folder as your CSV file. Take your `TRIPS_PATH` and replace the file name at the end with `raw_trips.parquet`.
```sql
COPY raw_trips TO 'PATH_TO_RAW_FOLDER/raw_trips.parquet' (FORMAT PARQUET);
```
For example: `'C:/Users/Sam/Documents/dataskools/bikeshare-pipeline/data/raw/raw_trips.parquet'`

✅ **Checkpoint:** A new file `raw_trips.parquet` appears in `data/raw` in VS Code Explorer (click the refresh icon if needed).

### Step 4.2 — Compare file sizes
1. Find the two files in your file manager. **Windows:** right-click → **Properties**. **Mac:** right-click → **Get Info**.
2. Write down the size of the CSV and the size of the Parquet file.

✅ **Checkpoint:** The Parquet file is much smaller than the CSV (often several times smaller). Same data, less space.

### Step 4.3 — Check the Parquet file has every row
```sql
SELECT COUNT(*) AS parquet_rows
FROM read_parquet('PATH_TO_RAW_FOLDER/raw_trips.parquet');
```
✅ **Checkpoint:** The number matches `table_rows` from Step 2.5. Rows in, rows out.

### Step 4.4 — Save your work as a script
1. In VS Code, right-click the `sql` folder → **New File** → name it `01_load_raw.sql`.
2. Paste in these sections, using your real paths in place of `TRIPS_PATH`, `STATIONS_PATH`, and `PATH_TO_RAW_FOLDER`:

```sql
-- Day 2: load raw data (raw layer)
-- Note: these paths belong to my computer. We will make scripts portable later in the course.

-- 1. Trips (CSV)
CREATE OR REPLACE TABLE raw_trips AS
SELECT *,
       '202505-capitalbikeshare-tripdata.csv' AS source_file,
       current_timestamp                      AS loaded_at
FROM read_csv('TRIPS_PATH', sample_size = -1);

-- 2. Stations (JSON, flattened)
CREATE OR REPLACE TABLE raw_stations AS
SELECT s.*,
       'station_information.json' AS source_file,
       current_timestamp          AS loaded_at
FROM (
    SELECT unnest(data.stations) AS s
    FROM read_json_auto('STATIONS_PATH')
);

-- 3. Row-count checks: source vs. table (each pair should match)
SELECT COUNT(*) AS source_rows
FROM read_csv('TRIPS_PATH', sample_size = -1);
SELECT COUNT(*) AS table_rows FROM raw_trips;

SELECT len(data.stations) AS source_stations
FROM read_json_auto('STATIONS_PATH');
SELECT COUNT(*) AS table_rows FROM raw_stations;

-- 4. Export raw trips to Parquet
COPY raw_trips TO 'PATH_TO_RAW_FOLDER/raw_trips.parquet' (FORMAT PARQUET);
```
3. Save with **Ctrl + S** (Cmd + S on Mac).

### Step 4.5 — Write your notes and commit
1. Open `README.md`. Under **Progress**, add this line:
   `- Day 2: loaded May 2025 trips and station data into raw tables; row counts validated`
2. Add a section called `## Day 2 Notes` and write 3 to 4 sentences answering:
   - What is the raw layer, and why do we not edit it?
   - Did your row counts match? Write both numbers.
   - What are two differences you saw between CSV and Parquet?
   - What is one thing in the data that looked odd or missing?
3. In **Source Control**, check that you see `01_load_raw.sql` and `README.md`, and **no** `.csv`, `.json`, `.parquet`, or `.duckdb` files. (They are ignored, which is correct.)
4. Commit with the message `Add Day 2 raw layer load script` and click **Sync Changes**.

✅ **Checkpoint:** On GitHub, refresh your repository. `sql/01_load_raw.sql` is there and your README shows the Day 2 notes. No data files appear.

---

## If You Get Stuck

| Problem | What to try |
|---|---|
| "No files found" or a file error | Check the path: forward slashes, quotation marks on both sides, the exact file name, and that it ends in `.csv` (not `.zip`). |
| Error like "Could not convert string ... to BIGINT" | DuckDB guessed a wrong type. Change `sample_size = -1` to `all_varchar = true` in the `read_csv(...)` call, so every column is loaded as text. Write a note about it in your README. |
| Row counts do not match | Check both counts use the same `read_csv` options, run the load again, and recount. If they still differ, take a screenshot and ask your instructor. |
| `station_information.json` has the wrong ending | Rename it to end with `.json`, or use the `curl` command from Step 1.2. |
| Error while flattening stations | Open the JSON file in VS Code and check it looks like `{ ... "data": { "stations": [ ... ] } }`. If it does not, download it again. |
| "Database is locked" | Only one program can use the `.duckdb` file at a time. Close any other window connected to it. |
| DBeaver freezes on a big result | Always use `LIMIT` when looking at `raw_trips`. |
| Git shows a data file in Source Control | Open `.gitignore`, check it contains `data/raw/` and `*.duckdb`, save, and look again. |

If you are stuck on one step for more than 15 minutes, take a screenshot of the error and ask your instructor.

---

## End-of-Day Checklist
- [ ] Downloaded the May 2025 trips CSV and the stations JSON into `data/raw`
- [ ] Peeked at the CSV with `read_csv` and checked the schema with `DESCRIBE`
- [ ] Loaded `raw_trips` with `source_file` and `loaded_at` columns
- [ ] Trips row count matches the source, and stays the same after running the load twice
- [ ] Flattened the JSON into `raw_stations` and its row count matches the source
- [ ] Exported `raw_trips` to Parquet and compared file sizes
- [ ] `01_load_raw.sql` saved, committed, and pushed (no data files on GitHub)
- [ ] README updated with Day 2 progress and notes