# SQL Cheatsheet for Backend Beginners

A practical guide for people learning backend development with Python.  
Goal: be able to **design a simple database** and write the queries you need day-to-day.

---

## 1. How to Think About a Database

A database is just a collection of **tables**.  
Each table stores one kind of thing (users, movies, orders, scans...).

**Good simple design rules:**
- One table = one main concept
- Each row has a unique `id`
- Related data is linked with foreign keys (not copied)
- Prefer many small tables over one huge table

Example of a simple design:

```
users
  id, name, email

orders
  id, user_id → users.id, total, created_at

order_items
  id, order_id → orders.id, product_name, price
```

This is called a **relational** design.

---

## 2. Schema (Creating Tables)

This is how you define the structure of your database.

### Basic CREATE TABLE

```sql
CREATE TABLE users (
    id INTEGER PRIMARY KEY,
    email TEXT NOT NULL UNIQUE,
    name TEXT NOT NULL,
    created_at TEXT NOT NULL
);
```

### Important column rules

| Rule              | Meaning                              | Example |
|-------------------|--------------------------------------|---------|
| `PRIMARY KEY`     | Unique ID of the row                 | `id INTEGER PRIMARY KEY` |
| `NOT NULL`        | Value is required                    | `email TEXT NOT NULL` |
| `UNIQUE`          | No duplicates allowed                | `email TEXT UNIQUE` |
| `DEFAULT`         | Value used when you don't provide one| `DEFAULT 'pending'` |
| `CHECK (...)`     | Only allow certain values            | `CHECK (status IN ('a', 'b'))` |

### Linking tables (Foreign Keys)

```sql
CREATE TABLE orders (
    id INTEGER PRIMARY KEY,
    user_id INTEGER NOT NULL REFERENCES users(id),
    total REAL NOT NULL
);
```

`REFERENCES users(id)` means:  
“This `user_id` must exist in the `users` table.”

You can also say what happens when the parent is deleted:

```sql
user_id INTEGER REFERENCES users(id) ON DELETE CASCADE
```

| Action                | What happens when parent is deleted |
|-----------------------|-------------------------------------|
| `ON DELETE CASCADE`   | Also delete the child rows          |
| `ON DELETE SET NULL`  | Set the foreign key to NULL         |
| `ON DELETE RESTRICT`  | Block the delete if children exist  |

### Auto-incrementing IDs

**SQLite:**
```sql
id INTEGER PRIMARY KEY AUTOINCREMENT
```

**PostgreSQL:**
```sql
id SERIAL PRIMARY KEY
-- or modern way:
id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY
```

### Indexes (make queries faster)

```sql
CREATE INDEX idx_orders_user_id ON orders(user_id);
CREATE INDEX idx_users_email ON users(email);
```

Create indexes on:
- Foreign keys
- Columns you often filter or sort by

---

## 3. Relationships (One-to-One, One-to-Many, Many-to-Many)

This is one of the most important parts of database design.

### One-to-Many (1:N) — the most common

One record on the left side can be linked to **many** records on the right side.

**Example:** One user can have many orders.

```sql
CREATE TABLE users (
    id INTEGER PRIMARY KEY,
    name TEXT NOT NULL
);

CREATE TABLE orders (
    id INTEGER PRIMARY KEY,
    user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    total REAL NOT NULL
);
```

- One user → many orders
- Each order belongs to only **one** user
- The foreign key (`user_id`) goes on the “many” side

**Real examples:**
- User → Orders
- Show → Episodes
- Scan → Media Files
- Category → Products

### One-to-One (1:1)

One record is linked to **exactly one** other record.

**Example:** Each user has one profile.

```sql
CREATE TABLE users (
    id INTEGER PRIMARY KEY,
    email TEXT NOT NULL UNIQUE
);

CREATE TABLE user_profiles (
    id INTEGER PRIMARY KEY,
    user_id INTEGER NOT NULL UNIQUE REFERENCES users(id) ON DELETE CASCADE,
    bio TEXT,
    avatar_url TEXT
);
```

Key point: the foreign key has a **UNIQUE** constraint so one user can only have one profile.

**Real examples:**
- User ↔ Profile
- User ↔ Settings
- Movie ↔ Detailed metadata (sometimes)

### Many-to-Many (M:N)

Many records on both sides can be linked to each other.

**Example:** A student can enroll in many courses, and a course can have many students.

You need a **third table** (called a junction / bridge / join table):

```sql
CREATE TABLE students (
    id INTEGER PRIMARY KEY,
    name TEXT NOT NULL
);

CREATE TABLE courses (
    id INTEGER PRIMARY KEY,
    title TEXT NOT NULL
);

-- Junction table
CREATE TABLE enrollments (
    student_id INTEGER NOT NULL REFERENCES students(id) ON DELETE CASCADE,
    course_id  INTEGER NOT NULL REFERENCES courses(id)  ON DELETE CASCADE,
    enrolled_at TEXT NOT NULL,
    PRIMARY KEY (student_id, course_id)   -- prevents the same enrollment twice
);
```

**Real examples:**
- Students ↔ Courses
- Posts ↔ Tags
- Movies ↔ Genres
- Scans ↔ Scan Roots (like in your MediaVault script)

### Quick summary

| Relationship   | How to implement                          | Foreign key location      |
|----------------|-------------------------------------------|---------------------------|
| One-to-Many    | Simple foreign key                        | On the “many” side        |
| One-to-One     | Foreign key + `UNIQUE`                    | On either side (usually the dependent one) |
| Many-to-Many   | Extra junction table with two foreign keys| In the junction table     |

---

## 4. Basic Queries

### Get data
```sql
SELECT id, name, email FROM users;
```

### Filter rows
```sql
SELECT * FROM users
WHERE status = 'active'
  AND created_at > '2024-01-01';
```

### Sort and limit (pagination)
```sql
SELECT * FROM users
ORDER BY created_at DESC
LIMIT 20 OFFSET 40;          -- page 3 (20 items per page)
```

- `ORDER BY ... DESC` = newest first
- `LIMIT 20` = only 20 rows
- `OFFSET 40` = skip the first 40 rows

### Check for NULL
```sql
WHERE deleted_at IS NULL          -- not deleted
WHERE deleted_at IS NOT NULL      -- is deleted
```
Never use `= NULL`.

---

## 5. JOINs (Connecting Tables)

Most real queries need data from more than one table.

### INNER JOIN
Only rows that have a match in both tables.

```sql
SELECT users.name, orders.total
FROM users
INNER JOIN orders ON orders.user_id = users.id;
```

### LEFT JOIN
Keep all rows from the left table, even if there is no match.

```sql
SELECT users.name, COUNT(orders.id) AS order_count
FROM users
LEFT JOIN orders ON orders.user_id = users.id
GROUP BY users.id, users.name;
```

**When to use which:**
- Need only matching data → `INNER JOIN`
- Need all records from one side → `LEFT JOIN`

---

## 6. Aggregation (Counting & Summarizing)

```sql
SELECT 
    status,
    COUNT(*) AS total,
    SUM(amount) AS revenue
FROM orders
GROUP BY status
HAVING COUNT(*) > 10;
```

- `GROUP BY` = group rows together
- `HAVING` = filter the groups (like WHERE, but after grouping)

Common functions: `COUNT`, `SUM`, `AVG`, `MIN`, `MAX`

---

## 7. Insert, Update, Delete

### Insert
```sql
INSERT INTO users (name, email)
VALUES ('Alice', 'alice@example.com')
RETURNING id;                  -- get the new id back
```

### Update
```sql
UPDATE users
SET status = 'inactive'
WHERE last_login < '2023-01-01';
```

### Delete
```sql
DELETE FROM sessions WHERE expires_at < date('now');
```

In real apps we often do **soft delete** instead:
```sql
UPDATE users SET deleted_at = date('now') WHERE id = 42;
```

---

## 8. Triggers

A **trigger** is a piece of SQL that runs **automatically** when something happens to a table (INSERT, UPDATE, or DELETE).

### Why use triggers?

- Keep related data in sync
- Create audit logs
- Prevent invalid changes
- Automatically update timestamps
- Implement complex cleanup logic

### Simple example (SQLite)

```sql
CREATE TRIGGER update_updated_at
AFTER UPDATE ON users
FOR EACH ROW
BEGIN
    UPDATE users SET updated_at = datetime('now') WHERE id = NEW.id;
END;
```

This automatically sets `updated_at` whenever a user row is changed.

### Example: Clean up related data

Even though foreign keys can do `ON DELETE CASCADE`, sometimes you need more control:

```sql
CREATE TRIGGER delete_user_orders
AFTER DELETE ON users
FOR EACH ROW
BEGIN
    DELETE FROM orders WHERE user_id = OLD.id;
END;
```

- `OLD` = the row that was deleted (or the old version on UPDATE)
- `NEW` = the new version of the row (on INSERT or UPDATE)

### Common trigger events

| Event            | When it runs                  |
|------------------|-------------------------------|
| `AFTER INSERT`   | After a new row is added      |
| `AFTER UPDATE`   | After a row is changed        |
| `AFTER DELETE`   | After a row is removed        |
| `BEFORE ...`     | Before the change happens     |

**Advice for beginners:**  
Prefer foreign keys with `ON DELETE CASCADE` when possible.  
Use triggers only when you need extra logic that foreign keys cannot do.

---

## 9. Transactions

A transaction makes sure a group of changes either **all succeed** or **all fail**.

```sql
BEGIN;

UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;

COMMIT;        -- save everything
-- ROLLBACK;   -- cancel everything if something went wrong
```

This is critical when money or important data is involved.

---

## 10. SQLite vs PostgreSQL

These two databases are the most common for Python backends.  
They speak almost the same SQL, but some details are different.

### 10.1 IDs (Auto increment)

**SQLite**
```sql
id INTEGER PRIMARY KEY AUTOINCREMENT
```

**PostgreSQL**
```sql
id SERIAL PRIMARY KEY
-- modern recommended way:
id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY
```

### 10.2 Boolean

**SQLite**  
No real boolean type. People use `INTEGER` (0 or 1) or the word `BOOLEAN` (still stored as 0/1).

**PostgreSQL**  
Has a real `BOOLEAN` type (`true` / `false`).

### 10.3 Current time

**SQLite**
```sql
datetime('now')
strftime('%Y-%m-%dT%H:%M:%fZ', 'now')
```

**PostgreSQL**
```sql
NOW()
CURRENT_TIMESTAMP
```

### 10.4 Upsert (insert or update)

**SQLite**
```sql
INSERT INTO settings (key, value) VALUES ('theme', 'dark')
ON CONFLICT(key) DO UPDATE SET value = excluded.value;
```

**PostgreSQL**
```sql
INSERT INTO settings (key, value) VALUES ('theme', 'dark')
ON CONFLICT (key) DO UPDATE SET value = EXCLUDED.value;
```
(Almost the same)

### 10.5 Foreign keys

**SQLite**  
Foreign keys are **disabled by default**. You must turn them on:
```sql
PRAGMA foreign_keys = ON;
```

**PostgreSQL**  
Foreign keys are always enabled.

### 10.6 Strictness

- **SQLite** is flexible (you can put text in an integer column).
- **PostgreSQL** is strict (it will reject wrong types).

### Quick recommendation

| Situation                              | Better choice   |
|----------------------------------------|-----------------|
| Local development, simple apps, mobile | SQLite          |
| Real production backend, multi-user    | PostgreSQL      |
| Learning SQL                           | Either (start with SQLite) |

Most Python frameworks (Django, SQLAlchemy, FastAPI) work with both.

---

## 11. Useful Patterns

### Soft delete
```sql
UPDATE users SET deleted_at = datetime('now') WHERE id = 42;
SELECT * FROM users WHERE deleted_at IS NULL;
```

### Check if something exists
```sql
SELECT EXISTS (
    SELECT 1 FROM orders WHERE user_id = 42
);
```

### Keyset pagination (better than OFFSET)
```sql
SELECT * FROM posts
WHERE id < 12345          -- last id you saw
ORDER BY id DESC
LIMIT 20;
```

---

## 12. Keywords (Quick Reference)

| Keyword            | What it does                     |
|--------------------|----------------------------------|
| `SELECT`           | Get data                         |
| `FROM`             | Which table                      |
| `WHERE`            | Filter rows                      |
| `JOIN` / `LEFT JOIN`| Combine tables                  |
| `ORDER BY`         | Sort                             |
| `LIMIT` / `OFFSET` | Pagination                       |
| `GROUP BY`         | Group for counting/summing       |
| `HAVING`           | Filter groups                    |
| `INSERT INTO`      | Add rows                         |
| `UPDATE`           | Change rows                      |
| `DELETE FROM`      | Remove rows                      |
| `CREATE TABLE`     | Make a new table                 |
| `CREATE INDEX`     | Make queries faster              |
| `CREATE TRIGGER`   | Run code automatically           |
| `PRIMARY KEY`      | Unique row id                    |
| `REFERENCES`       | Link to another table            |
| `CHECK`            | Restrict allowed values          |
| `DEFAULT`          | Fallback value                   |
| `BEGIN` / `COMMIT` | Transactions                     |

---

## 13. Operators & Functions

### Operators
| Operator       | Meaning                  | Example |
|----------------|--------------------------|---------|
| `=` `!=`       | Equal / not equal        | `status = 'active'` |
| `>` `<` `>=` `<=` | Compare numbers/dates | `price > 100` |
| `AND` `OR`     | Combine conditions       | `a = 1 AND b = 2` |
| `IN`           | Match any in a list      | `status IN ('a', 'b')` |
| `LIKE`         | Pattern matching         | `email LIKE '%@gmail.com'` |
| `IS NULL`      | Is empty                 | `deleted_at IS NULL` |
| `BETWEEN`      | Range                    | `price BETWEEN 10 AND 50` |

### Common functions
| Function            | Purpose                     |
|---------------------|-----------------------------|
| `COUNT(*)`          | How many rows               |
| `SUM(x)`            | Add numbers                 |
| `AVG(x)`            | Average                     |
| `MAX(x)` / `MIN(x)` | Highest / lowest            |
| `COALESCE(a, b)`    | First value that is not NULL|
| `LOWER(x)` / `UPPER(x)` | Change case             |

---

## 14. How to Design a Simple Database (Step by step)

1. **List the main things** you need to store  
   (User, Movie, Scan, Subtitle...)

2. **Make one table per thing**

3. **Add an `id`** as primary key to every table

4. **Decide the relationships**  
   - One-to-Many?  
   - One-to-One?  
   - Many-to-Many? (needs a junction table)

5. **Add foreign keys** to show the relationships

6. **Decide what happens on delete**  
   (`ON DELETE CASCADE` or `SET NULL`)

7. **Add useful constraints**  
   (`NOT NULL`, `UNIQUE`, `CHECK`)

8. **Create indexes** on foreign keys and search columns

9. **Test with a few INSERT + SELECT statements**

That’s enough to design most small-to-medium backend databases.

---

## Final Mental Model

- **Schema** = how you structure data (`CREATE TABLE`, constraints, foreign keys)
- **Relationships** = One-to-One, One-to-Many, Many-to-Many
- **Queries** = how you read data (`SELECT` + `JOIN` + `WHERE`)
- **Changes** = how you write data (`INSERT` / `UPDATE` / `DELETE`)
- **Triggers** = automatic reactions to changes
- **Transactions** = make a group of changes safe
- **SQLite** = great for learning and simple apps
- **PostgreSQL** = better for real production backends

Master the sections above and you will be able to design a simple database and work with it confidently as a backend developer.
