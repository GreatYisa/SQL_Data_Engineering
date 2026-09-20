# Week 1, Day 1 — Tasks
## Set Up Your Data Engineer Workspace (6-Hour Session)

Read today's Concepts doc first. **Pace:** this session is planned for up to 6 hours so you can work slowly and carefully. A confident learner can finish in about 3 hours. The time note beside each block shows both.

Today you install your tools, create your GitHub project, build a database, run your first queries, and make your first commit. You do not download real data yet.

**Before you start, you need:** a computer where you can install programs, an internet connection, and an email address for GitHub.

**Note:** program screens change slightly between versions. If a button looks a little different, look for the same *words* from the instructions.

---

## Block 1 — Install Your Tools (Hours 1–2 · confident pace: about 45 minutes)

### Step 1.1 — Install DBeaver (your SQL editor)
1. Open your web browser and go to **dbeaver.io/download**.
2. Click the download for your system (Windows installer, macOS, or Linux). Choose **DBeaver Community**, which is the free one.
3. Open the downloaded file and follow the installer. Click **Next** and accept the defaults until it finishes.
4. Open DBeaver. If it asks whether to create a sample database, click **No** or **Skip**.

✅ **Checkpoint:** You see the DBeaver window with a **Database Navigator** panel on the left.

### Step 1.2 — Install VS Code (your file editor)
1. Go to **code.visualstudio.com** and click the download button for your system.
2. Run the installer and accept the defaults.
3. Open VS Code.

✅ **Checkpoint:** VS Code opens to a Welcome tab.

### Step 1.3 — Install Git (your version control tool)
- **Windows:** go to **git-scm.com/downloads**, click **Windows**, run the installer, and click **Next** through every screen to accept the defaults.
- **Mac:** open **Terminal** (press Cmd + Space, type `Terminal`, press Enter). Type `git --version` and press Enter. If a pop-up offers to install developer tools, click **Install** and wait for it to finish.

Now check it worked. In VS Code, click **Terminal** in the top menu, then **New Terminal**. A panel opens at the bottom. Type this and press Enter:

```
git --version
```

✅ **Checkpoint:** It prints a line starting with `git version` followed by numbers.

> **If VS Code says git is not found:** close VS Code completely, open it again, and retry.

---

## Block 2 — Create Your GitHub Project (Hour 3 · confident pace: about 30 minutes)

### Step 2.1 — Create a GitHub account
1. Go to **github.com** and click **Sign up** (skip this if you already have an account and just sign in).
2. Enter your email, a password, and a username. Choose a username you would be happy to show an employer, because your projects will be public.
3. Verify your email when GitHub asks.

### Step 2.2 — Tell Git who you are
Git needs your name and email to label your commits. In the VS Code terminal (**Terminal → New Terminal**), type these two lines, one at a time, using your own name and the email you used for GitHub. Keep the quotation marks.

```
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```

✅ **Checkpoint:** Type `git config --global --list`. You see your name and email in the output.

### Step 2.3 — Create the repository on GitHub
1. On GitHub, click the **+** at the top right, then **New repository**.
2. **Repository name:** `bikeshare-pipeline`
3. Choose **Public**.
4. Tick **Add a README file**. Leave everything else as it is.
5. Click **Create repository**.

✅ **Checkpoint:** You see your new repository page with a file called `README.md`.

### Step 2.4 — Copy the repository to your computer (clone)
1. On your computer, create a folder where you will keep course work, for example `Documents/dataskools`.
2. In VS Code, click the **Source Control** icon in the left bar (it looks like a branch with dots). Click **Clone Repository**, then **Clone from GitHub**.
3. VS Code asks you to sign in to GitHub. Click **Allow** or **Sign in**, and your browser opens. Click **Authorize** in the browser, then return to VS Code. (You sign in through the browser, so you do not type a password into VS Code.)
4. Pick `your-username/bikeshare-pipeline` from the list.
5. Choose the folder you created, then click **Select as Repository Destination**.
6. When VS Code asks if you want to open the cloned repository, click **Open**.

✅ **Checkpoint:** The Explorer panel on the left shows a folder named `bikeshare-pipeline` containing `README.md`.

> **If "Clone from GitHub" does not appear:** click **Clone Repository** and paste this address instead, using your username: `https://github.com/YOUR-USERNAME/bikeshare-pipeline.git`

### Step 2.5 — Create your folders and `.gitignore`
1. In the Explorer panel, hover over the `bikeshare-pipeline` heading. Click the **New Folder** icon and name the folder `sql`.
2. Click **New Folder** again and name it `data`. Then right-click `data`, choose **New Folder**, and name it `raw`.
3. Click the **New File** icon and name the file exactly `.gitignore` (with the dot at the front).
4. Paste this into the file and save with **Ctrl + S** (Cmd + S on Mac):

```
# Data files: code goes to GitHub, data does not
data/raw/
*.duckdb
*.duckdb.wal
.DS_Store
```

✅ **Checkpoint:** Explorer shows `data`, `sql`, `.gitignore`, and `README.md`.

---

## Block 3 — Create Your Database and Run Your First Queries (Hours 4–5 · confident pace: about 60 minutes)

### Step 3.1 — Get the path for your database file
1. In VS Code Explorer, right-click the `data` folder and choose **Copy Path**.
2. Paste it into any notepad or text file, and add the file name at the end. It should look like this (yours will differ):
   - Windows: `C:\Users\Sam\Documents\dataskools\bikeshare-pipeline\data\bikeshare.duckdb`
   - Mac: `/Users/sam/Documents/dataskools/bikeshare-pipeline/data/bikeshare.duckdb`

This is the address where your new database file will be created.

### Step 3.2 — Connect DBeaver to DuckDB
1. In DBeaver, click **Database** in the top menu, then **New Database Connection**.
2. Type `DuckDB` in the search box, click **DuckDB**, and click **Next**.
3. In the **Path** box, paste the full path from Step 3.1.
4. Click **Test Connection**. DBeaver will ask to download the DuckDB driver. Click **Download** and wait for it to finish.
5. When the test says it connected, click **OK**, then **Finish**.

✅ **Checkpoint 1:** A new DuckDB connection appears in the Database Navigator on the left.
✅ **Checkpoint 2:** In VS Code, look inside the `data` folder. You see `bikeshare.duckdb` shown in **grey** (grey means Git is ignoring it, so your `.gitignore` works). If you do not see it, click the refresh icon in the Explorer panel.

> **If the driver download fails:** in DBeaver click **Database → Driver Manager**, select **DuckDB**, click **Edit**, open the **Libraries** tab, and click **Download/Update**. If you are on a school or work network, ask your instructor about firewall settings.

### Step 3.3 — Open a SQL editor
Right-click your DuckDB connection, choose **SQL Editor**, then **New SQL script**. A blank editor opens.

### Step 3.4 — Create the practice table
Paste the whole script below into the editor. Then run it as a script: press **Alt + X** (Mac: **Option + X**), or use the top menu **SQL Editor → Execute SQL Script**.

```sql
-- Day 1: practice table (made-up sample data, not real trips)
CREATE OR REPLACE TABLE practice_trips (
    ride_id            VARCHAR,
    rideable_type      VARCHAR,
    started_at         TIMESTAMP,
    ended_at           TIMESTAMP,
    start_station_name VARCHAR,
    end_station_name   VARCHAR,
    member_casual      VARCHAR
);

INSERT INTO practice_trips VALUES
    ('PRAC0001', 'classic_bike',  '2025-05-01 08:05:00', '2025-05-01 08:19:00', 'Union Station',    'Dupont Circle',    'member'),
    ('PRAC0002', 'electric_bike', '2025-05-01 08:12:00', '2025-05-01 08:24:00', 'Lincoln Memorial', 'Union Station',    'casual'),
    ('PRAC0003', 'classic_bike',  '2025-05-01 09:30:00', '2025-05-01 09:52:00', 'Dupont Circle',    'Lincoln Memorial', 'member'),
    ('PRAC0004', 'electric_bike', '2025-05-01 12:15:00', '2025-05-01 12:31:00', 'Union Station',    'Lincoln Memorial', 'casual'),
    ('PRAC0005', 'classic_bike',  '2025-05-01 17:40:00', '2025-05-01 18:02:00', 'Dupont Circle',    'Union Station',    'member'),
    ('PRAC0006', 'classic_bike',  '2025-05-02 07:55:00', '2025-05-02 08:10:00', 'Union Station',    'Dupont Circle',    'member'),
    ('PRAC0007', 'electric_bike', '2025-05-02 18:20:00', '2025-05-02 18:47:00', 'Lincoln Memorial', NULL,               'casual'),
    ('PRAC0008', 'classic_bike',  '2025-05-02 18:35:00', '2025-05-02 19:05:00', 'Lincoln Memorial', 'Union Station',    'casual');
```

✅ **Checkpoint:** No red error message appears. Right-click your connection and choose **Refresh** (or press **F5**), then expand it (click the small arrows) until you see `practice_trips` under **Tables**.

### Step 3.5 — Run your first five queries
Click at the end of the script, press Enter twice, and paste the queries below. To run one query, **click anywhere inside it** and press **Ctrl + Enter** (Mac: **Cmd + Enter**). The result appears in the panel at the bottom. Run them one at a time and compare each result with the checkpoint.

```sql
-- Query 1: look at everything
SELECT * FROM practice_trips;

-- Query 2: look at only the first 3 rows
SELECT * FROM practice_trips LIMIT 3;

-- Query 3: pick only some columns
SELECT ride_id, start_station_name, member_casual FROM practice_trips;

-- Query 4: count the rows
SELECT COUNT(*) AS trip_count FROM practice_trips;

-- Query 5: see the structure of the table
DESCRIBE practice_trips;
```

✅ **Checkpoints:**
| Query | You should see |
|---|---|
| 1 | 8 rows and 7 columns. One cell in the `end_station_name` column (ride `PRAC0007`) shows `[NULL]`. That means "no value", and you will learn about it on Day 3. |
| 2 | 3 rows |
| 3 | 8 rows and only 3 columns |
| 4 | 1 row: `trip_count` = **8** |
| 5 | 7 rows listing each column name and its type (`VARCHAR` for text, `TIMESTAMP` for date and time) |

Query 4 is your first **row-count check**: it confirms the number of rows in the table. You will use this habit every day of the course.

### Step 3.6 — Your turn
Write one query that shows **only** `start_station_name` and `end_station_name`, and only the **first 5 rows**. Run it.

✅ **Checkpoint:** 5 rows and 2 columns.

<details>
<summary>Check your answer</summary>

```sql
SELECT start_station_name, end_station_name
FROM practice_trips
LIMIT 5;
```
</details>

---

## Block 4 — Save Your Work and Make Your First Commit (Hour 6, first half · confident pace: about 20 minutes)

### Step 4.1 — Save your SQL as a file
1. In the DBeaver editor, click inside the text, press **Ctrl + A** to select everything, then **Ctrl + C** to copy (Cmd on Mac).
2. In VS Code, right-click the `sql` folder, choose **New File**, and name it `00_day1_practice.sql`.
3. Paste the SQL into it and save with **Ctrl + S**.

### Step 4.2 — Commit and push
1. In VS Code, click the **Source Control** icon in the left bar. Under **Changes** you should see `.gitignore` and `00_day1_practice.sql`.
2. **Important check:** `bikeshare.duckdb` should **not** be in the list. If it is, your `.gitignore` is not working (see the help table below).
3. Click in the **Message** box at the top and type: `Add Day 1 practice SQL and .gitignore`
4. Click **Commit**. If VS Code asks whether to stage all changes and commit them directly, click **Yes**.
5. Click **Sync Changes** (or **Push**). If a pop-up asks you to confirm, click **OK**.

✅ **Checkpoint:** On GitHub, refresh your repository page. You see `.gitignore` and the `sql` folder, and your commit message is shown. There is no `.duckdb` file.

---

## Block 5 — Write-Up (Hour 6, second half · confident pace: about 15 minutes)

1. In VS Code, open `README.md` and replace everything with the text below. Then add your notes under **Day 1 Notes**.

```
# Bikeshare Data Pipeline

A data engineering project built in the Dataskools SQL for Data Engineering course.

- Data: Capital Bikeshare trips
- Tools: DuckDB, DBeaver, VS Code, Git, GitHub

## Progress
- Day 1: workspace set up, first queries run

## Day 1 Notes
(Write 3 to 4 sentences here.)
```

2. Under **Day 1 Notes**, write 3 to 4 sentences in your own words answering:
   - What is a table, and what does one row in `practice_trips` represent?
   - Why do we save SQL in files instead of only typing it in DBeaver?
   - Why do we not commit the `.duckdb` file to GitHub?
   - What was one thing that looked new or strange today?
3. Save the file, then commit it with the message `Update README with Day 1 notes` and click **Sync Changes** again.

✅ **Checkpoint:** Your GitHub repository page shows the new README text.

---

## If You Get Stuck

| Problem | What to try |
|---|---|
| `git` is not found in the terminal | Close VS Code fully and reopen it. If it still fails, reinstall Git and accept the defaults. |
| Commit fails with "tell me who you are" | Repeat Step 2.2 with your name and email, then commit again. |
| VS Code asks for a password when cloning | Do not type your password. Use the browser sign-in option, or the paste-the-address method in Step 2.4. |
| The table `practice_trips` is not found | Check that you ran the create script with **Alt + X** (Option + X on Mac), refresh with **F5**, and make sure the editor is attached to your DuckDB connection. |
| A query shows a red error | Check for missing semicolons, missing quotation marks, or a typo, then copy the query again from this guide. |
| `bikeshare.duckdb` shows up in Source Control | Open `.gitignore`. Check the spelling, that it contains `*.duckdb`, and that you saved it. Then reopen the Source Control panel. |
| DBeaver says the database is locked | Only one program can use a DuckDB file at a time. Close any other window connected to the same file. |

If you are still stuck after 15 minutes on one step, take a screenshot of the error and ask your instructor. Getting stuck is a normal part of data engineering.

---

## End-of-Day Checklist
- [ ] DBeaver, VS Code, and Git installed, and `git --version` works
- [ ] GitHub account created and Git identity set
- [ ] `bikeshare-pipeline` repository created and cloned to your computer
- [ ] `sql` and `data/raw` folders and `.gitignore` created
- [ ] DBeaver connected to a new DuckDB database file
- [ ] `practice_trips` table created, with a row count of 8
- [ ] Five practice queries run, plus your own 5-row, 2-column query
- [ ] `00_day1_practice.sql` saved, committed, and pushed (no `.duckdb` file on GitHub)
- [ ] README updated with your Day 1 notes and pushed