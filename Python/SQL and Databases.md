# SQL and Databases

SQL looks like lots of keywords, but once you understand **table → filter rows → join tables → group**, you can answer most interview queries.

The core idea:

```text
Database holds tables
    ↓
SQL asks questions
    ↓
returns rows
```

This file is **SQL theory + SQLite practice**. ORM mapping with SQLAlchemy is the next file.

Setup used here (stdlib only):

```python
import sqlite3

db = sqlite3.connect(":memory:")
```

---

# 1. SQL Fundamentals

SQL = Structured Query Language. Tables hold rows and columns.

```text
users table
id | name   | age
1  | Kakade | 25
2  | Asha   | 30
```

Key statements:

```text
SELECT → read
INSERT → create
UPDATE → modify
DELETE → remove
```

Python setup for all examples:

```python
import sqlite3

db = sqlite3.connect(":memory:")
cur = db.cursor()

cur.execute("CREATE TABLE users (id INTEGER PRIMARY KEY, name TEXT, age INTEGER)")
cur.execute("INSERT INTO users VALUES (1, 'Kakade', 25)")
cur.execute("INSERT INTO users VALUES (2, 'Asha', 30)")
db.commit()
```

---

# 2. SELECT

Read columns.

```sql
SELECT name FROM users;
```

```python
print(cur.execute("SELECT name FROM users").fetchall())
```

Output:

```text
[('Kakade',), ('Asha',)]
```

All columns:

```sql
SELECT * FROM users;
```

```python
print(cur.execute("SELECT * FROM users").fetchall())
```

Output:

```text
[(1, 'Kakade', 25), (2, 'Asha', 30)]
```

Think:

```text
SELECT <columns> FROM <table>
```

---

# 3. INSERT

Add rows.

```sql
INSERT INTO users (name, age) VALUES ('Rohan', 22);
```

```python
cur.execute("INSERT INTO users (name, age) VALUES (?, ?)", ("Rohan", 22))
db.commit()
```

Always use `?` placeholders, never f-strings. Prevents SQL injection.

Bad:

```python
cur.execute(f"INSERT INTO users VALUES ({user_input})")
```

---

# 4. UPDATE

Modify existing rows. Always use `WHERE`, or you update everything.

```sql
UPDATE users SET age = 26 WHERE name = 'Kakade';
```

```python
cur.execute("UPDATE users SET age = ? WHERE name = ?", (26, "Kakade"))
db.commit()

print(cur.execute("SELECT * FROM users WHERE name='Kakade'").fetchall())
```

Output:

```text
[(1, 'Kakade', 26)]
```

Without `WHERE`:

```text
UPDATE users SET age = 0
    ↓
every user now age 0
```

Classic interview trap.

---

# 5. DELETE

Remove rows. Same warning: always `WHERE`.

```sql
DELETE FROM users WHERE name = 'Rohan';
```

```python
cur.execute("DELETE FROM users WHERE name = ?", ("Rohan",))
db.commit()
```

---

# 6. WHERE

Filter rows.

```sql
SELECT * FROM users WHERE age > 25;
```

Operators:

```text
=  equal
!= not equal
> < >= <=
AND OR NOT
LIKE, IN, BETWEEN, IS NULL
```

```python
print(cur.execute("SELECT name FROM users WHERE age >= 25 AND name != 'Asha'").fetchall())
```

Output:

```text
[('Kakade',)]
```

LIKE for patterns:

```sql
SELECT * FROM users WHERE name LIKE 'K%';
```

`%` means anything. `'K%'` = starts with K.

---

# 7. ORDER BY

Sort results.

```sql
SELECT * FROM users ORDER BY age DESC;
```

```python
print(cur.execute("SELECT name, age FROM users ORDER BY age DESC").fetchall())
```

Output:

```text
[('Asha', 30), ('Kakade', 26)]
```

```text
ASC → small to large (default)
DESC → large to small
```

---

# 8. GROUP BY

Group rows, then aggregate each group.

Setup:

```python
cur.execute("CREATE TABLE orders (id INTEGER, user_id INTEGER, amount INTEGER)")
cur.executemany("INSERT INTO orders VALUES (?, ?, ?)", [
    (1, 1, 100), (2, 1, 200), (3, 2, 50),
])
```

Query:

```sql
SELECT user_id, COUNT(*), SUM(amount) FROM orders GROUP BY user_id;
```

```python
print(cur.execute(
    "SELECT user_id, COUNT(*), SUM(amount) FROM orders GROUP BY user_id"
).fetchall())
```

Output:

```text
[(1, 2, 300), (2, 1, 50)]
```

Think:

```text
GROUP BY user_id
    ↓
bucket rows per user
    ↓
COUNT/SUM per bucket
```

---

# 9. JOIN

Combine two tables.

Setup:

```python
print(cur.execute("SELECT * FROM users").fetchall())
print(cur.execute("SELECT * FROM orders").fetchall())
```

Query:

```sql
SELECT users.name, orders.amount
FROM users
JOIN orders ON users.id = orders.user_id;
```

```python
print(cur.execute(
    "SELECT users.name, orders.amount FROM users JOIN orders ON users.id = orders.user_id"
).fetchall())
```

Output:

```text
[('Kakade', 100), ('Kakade', 200), ('Asha', 50)]
```

Think:

```text
users.id = orders.user_id
    ↓
match rows across tables
```

---

# 10. INNER JOIN

`JOIN` alone means `INNER JOIN`. Only matching rows.

```text
users: Kakade(1), Asha(2), Meera(3, no orders)
orders: for user 1, 1, 2

INNER JOIN
    ↓
Kakade x2, Asha x1
Meera excluded (no match)
```

Use when you want only rows with matches on both sides.

---

# 11. LEFT JOIN

Keep all left rows, even without match.

```sql
SELECT users.name, orders.amount
FROM users
LEFT JOIN orders ON users.id = orders.user_id;
```

Output concept:

```text
Kakade | 100
Kakade | 200
Asha   | 50
Meera  | NULL
```

Meera appears with `NULL` amount. Use when you want "all users, plus their orders if any".

```text
INNER → only matches
LEFT → all left + matches or NULL
```

---

# 12. Aggregations

Functions over many rows → one value.

```sql
SELECT COUNT(*), SUM(amount), AVG(amount), MAX(amount), MIN(amount) FROM orders;
```

```python
print(cur.execute(
    "SELECT COUNT(*), SUM(amount), AVG(amount), MAX(amount), MIN(amount) FROM orders"
).fetchone())
```

Output:

```text
(3, 350, 116.66, 200, 50)
```

| Function | Returns |
| -------- | ------- |
| `COUNT(*)` | Row count |
| `SUM()` | Total |
| `AVG()` | Average |
| `MAX()` / `MIN()` | Extremes |

Combine with `GROUP BY` for per-group stats.

---

# 13. Primary Keys

Uniquely identifies each row. Never null, never duplicate.

```sql
CREATE TABLE users (
  id INTEGER PRIMARY KEY,
  name TEXT
);
```

```text
id 1 → Kakade
id 2 → Asha
```

Think: row's Aadhaar number. Used in JOINs and URLs (`/users/5`).

---

# 14. Foreign Keys

Links one table to another's primary key.

```sql
CREATE TABLE orders (
  id INTEGER PRIMARY KEY,
  user_id INTEGER,
  FOREIGN KEY (user_id) REFERENCES users(id)
);
```

```text
orders.user_id = 1
    ↓
points to users.id = 1 (Kakade)
```

Enforces integrity: cannot order for non-existent user (if enforced).

---

# 15. Constraints

Rules on data.

```sql
CREATE TABLE users (
  id INTEGER PRIMARY KEY,
  email TEXT UNIQUE NOT NULL,
  age INTEGER CHECK (age >= 0),
  role TEXT DEFAULT 'user'
);
```

| Constraint | Meaning |
| ---------- | ------- |
| `UNIQUE` | No duplicates (emails) |
| `NOT NULL` | Must have value |
| `CHECK` | Condition must hold |
| `DEFAULT` | Value if not given |

Violate and DB rejects write. Better than manual `if` checks.

---

# 16. Indexes

Speed up `WHERE` / `JOIN` lookups. Tradeoff: slower writes, faster reads.

```sql
CREATE INDEX idx_users_email ON users(email);
```

```text
No index → scan every row
With index → jump directly
```

Add indexes on filtered/joined columns (`email`, `user_id`). Don't index everything — each index costs write speed and space.

---

# 17. Transactions

Group writes so all succeed or all fail. Money transfer must not half-complete.

```python
try:
    cur.execute("UPDATE accounts SET balance = balance - 100 WHERE id = 1")
    cur.execute("UPDATE accounts SET balance = balance + 100 WHERE id = 2")
    db.commit()
except Exception:
    db.rollback()
```

Think:

```text
BEGIN
    ↓
write 1, write 2
    ↓
COMMIT → both saved
or ROLLBACK → both undone
```

ACID in one line: Atomic, Consistent, Isolated, Durable.

---

# 18. SQLite

File-based DB, zero setup, stdlib `sqlite3`. Perfect for learning, tests, small tools.

```python
import sqlite3

db = sqlite3.connect("app.db")
```

Pros: no server, single file. Cons: weak concurrency, no advanced types. Interview answer: "SQLite for dev/tests, PostgreSQL for production."

---

# 19. PostgreSQL

Production open-source DB. Handles concurrency, JSON, full-text, strict types.

Same SQL, different connection (needs `psycopg`):

```python
import psycopg

db = psycopg.connect("dbname=prep user=kakade")
```

Differences to mention:

```text
SQLite → file, one writer, lenient types
Postgres → server, many connections, SERIAL/UUID, RETURNING, JSONB
```

`RETURNING id` after INSERT is a Postgres favorite in interviews.

---

# Real-World Uses

```python
User signup → INSERT with UNIQUE email
Feed → SELECT with WHERE + ORDER BY + LIMIT
Dashboard → GROUP BY + SUM/COUNT
Transfer → transaction + rollback
```

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **SELECT** | `SELECT cols FROM t WHERE ...` |
| **INSERT** | With `?` placeholders |
| **UPDATE/DELETE** | Always `WHERE` |
| **WHERE** | `AND/OR/LIKE/IN/BETWEEN` |
| **ORDER BY** | `ASC/DESC` |
| **GROUP BY** | Bucket + aggregate |
| **JOIN/INNER** | Only matches |
| **LEFT JOIN** | All left + NULL |
| **Aggregates** | `COUNT SUM AVG MAX MIN` |
| **PK** | Unique row id |
| **FK** | Points to PK |
| **Constraints** | `UNIQUE NOT NULL CHECK DEFAULT` |
| **Indexes** | Fast reads, slow writes |
| **Transactions** | `commit` all or `rollback` |
| **SQLite** | File, dev/tests |
| **Postgres** | Server, production |

## The Mental Model

```text
        DATABASE
            │
            ↓
   Tables + PK/FK + Constraints
            │
  ┌─────────┼─────────┐
  ↓         ↓         ↓
SELECT    INSERT   UPDATE/DELETE
WHERE     values    WHERE!
  │         │         │
  ↓         ↓         ↓
ORDER BY  Indexes  Transactions
JOIN      commit/rollback
GROUP BY
  │
  ↓
Aggregates per group
```

### The 3 things to remember

**1. Read shape:**

```sql
SELECT cols FROM t WHERE x ORDER BY y;
```

**2. Relate tables:**

```sql
SELECT u.name, o.amount FROM users u
LEFT JOIN orders o ON o.user_id = u.id;
```

**3. Write safely:**

```python
cur.execute("INSERT INTO users VALUES (?, ?)", (name, age))
db.commit()
```

> **SQL asks tables for rows: filter with WHERE, combine with JOIN, summarize with GROUP BY, and protect writes with constraints and transactions.**
