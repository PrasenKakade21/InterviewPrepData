# SQL and Databases — Revision Notes (Intermediate)
> One-line mental model: Filter rows with WHERE → combine with JOIN → summarize with GROUP BY, protect writes.
## Core recall
- CRUD: `SELECT` read, `INSERT` create with `?` placeholders, `UPDATE/DELETE` always with `WHERE`.
- `WHERE`: `= != > < AND OR NOT LIKE IN BETWEEN IS NULL`; `'K%'` starts-with-K.
- `ORDER BY age DESC` (`ASC` default); `GROUP BY user_id` buckets + `COUNT/SUM/AVG`.
- `JOIN` = `INNER` = only matches; `LEFT JOIN` keeps all left + `NULL` (e.g. Meera no orders).
- Aggregates: `COUNT(*) SUM AVG MAX MIN`; combine with GROUP BY per-group stats.
- Keys: PK unique row id; FK `user_id REFERENCES users(id)` integrity.
- Constraints: `UNIQUE NOT NULL CHECK(age>=0) DEFAULT 'user'` — DB rejects bad writes.
- Indexes: `CREATE INDEX idx_users_email ON users(email)` fast reads, slow writes; index WHERE/JOIN cols.
- Transactions: `commit` all or `rollback`; ACID atomic; money-transfer pattern.
- SQLite: file, stdlib `sqlite3`, dev/tests; Postgres: server, concurrency, `RETURNING id`, JSONB.
## Must-remember code
```python
import sqlite3
db = sqlite3.connect(":memory:"); cur = db.cursor()
cur.execute("CREATE TABLE users (id INTEGER PRIMARY KEY, name TEXT, age INTEGER)")
cur.execute("INSERT INTO users (name, age) VALUES (?, ?)", ("Kakade", 25)); db.commit()
print(cur.execute("SELECT * FROM users WHERE age > 20 ORDER BY age DESC").fetchall())
```
```sql
SELECT users.name, orders.amount FROM users
LEFT JOIN orders ON users.id = orders.user_id;
```
```sql
SELECT user_id, COUNT(*), SUM(amount) FROM orders GROUP BY user_id;
```
```python
try:
    cur.execute("UPDATE accounts SET balance=balance-100 WHERE id=1")
    cur.execute("UPDATE accounts SET balance=balance+100 WHERE id=2")
    db.commit()
except Exception: db.rollback()
```
## Interview traps
- Missing `WHERE` on UPDATE/DELETE wipes whole table → always filter.
- f-string SQL → injection; always `?` placeholders.
- `INNER` drops non-matching rows (Meera vanishes) → use `LEFT` for "all + optional".
- Indexing everything slows writes → index only filtered/joined cols.
- SQLite for prod concurrency fails → answer "SQLite dev/tests, Postgres prod".
## 3 things to remember
1. Read shape: `SELECT cols FROM t WHERE x ORDER BY y`.
2. Relate: `JOIN` matches only, `LEFT JOIN` keeps left + NULL.
3. Write safely: `?` placeholders + transactions + constraints.
