# Chapter 7: Database Internals & Performance

---

## 21. Indexing Basics

### 1. Definition
An Index is a physical data structure (usually a B-Tree) that the database engine maintains to speed up row retrieval times.

### 2. Why It Exists
Without an index, querying a row (like `WHERE email = 'user@test.com'`) forces the database to scan every single record in the table on disk (`O(n)` time). An index organizes the search keys into a sorted tree, reducing lookups to logarithmic time (`O(log n)`).

### 3. Where It Is Used
Excluding primary keys (which are indexed automatically), you manually add indexes to columns that appear frequently in `WHERE` filters, `JOIN` ON clauses, or `ORDER BY` sorting fields.

### 4. Syntax
```sql
CREATE INDEX index_name ON table_name (column_name);
```

### 5. Simple Example
```sql
-- Speed up customer lookups by email
CREATE UNIQUE INDEX idx_customers_email ON customers (email);
```

### 6. Interview Explanation
> "Think of a database index like the index at the back of a textbook. If you want to find where 'ACID transactions' are mentioned, you don't scan the book page-by-page. You go to the index, find the term, get the page number (row pointer), and jump straight there. While it speeds up reads significantly, it adds a small write penalty because every time you insert or delete rows, the database has to update the index tree."

### 7. Common Mistakes
* **Indexing every column:** Every index takes up disk space and slows down write operations (`INSERT`, `UPDATE`, `DELETE`) because the database has to update the tree. Only index fields that are frequently queried.
* **Writing queries that bypass indexes:** Applying functions to indexed columns in filters, like `WHERE YEAR(signup_date) = 2026`. This forces the database to evaluate the function on every row, disabling the index lookup. Use range queries instead: `WHERE signup_date >= '2026-01-01'`.

### 8. Follow-up Questions
* *What is the difference between a Clustered and a Non-Clustered index? (Clustered physically dictates the order of rows on disk, while Non-Clustered is a separate table of pointers).*
* *What is a Composite Index, and how does the Left-Prefix rule affect it?*

---

## 22. Query Execution Order

### 1. Definition
Query Execution Order defines the logical sequence in which the database engine processes SQL clauses. This differs from the order in which you write them.

### 2. Why It Exists
Understanding execution order is necessary to debug syntax errors (like why you can't use alias names in a `WHERE` clause) and to optimize query speed.

### 3. Where It Is Used
Writing efficient aggregations, debugging filters, and scoping window functions.

### 4. Code Order vs Execution Order:

| Code Order | Execution Order | Clause | Details |
|---|---|---|---|
| 1 | **3** | `SELECT` | Projections are defined, aliases are created. |
| 2 | **1** | `FROM` / `JOIN` | Source tables are identified and combined. |
| 3 | **2** | `WHERE` | Row-level filtering is executed. |
| 4 | **4** | `GROUP BY` | Rows are grouped into buckets. |
| 5 | **5** | `HAVING` | Group-level aggregates are filtered. |
| 6 | **6** | `ORDER BY` | Result rows are sorted. |
| 7 | **7** | `LIMIT` / `OFFSET` | Output size is capped. |

### 5. Simple Example
```sql
-- Why does this fail?
SELECT country, COUNT(*) as cnt
FROM users
WHERE cnt > 10 -- ❌ FAIL: 'cnt' alias doesn't exist yet!
GROUP BY country;
```
It fails because `WHERE` runs in Step 2, but the alias `cnt` is only defined in Step 3 (`SELECT`).

### 6. Interview Explanation
> "Although we write `SELECT` first, the database engine executes `FROM` and `JOIN`s first to build the working dataset. It then applies `WHERE` filters, performs `GROUP BY` divisions, runs `HAVING` filters, projects columns in `SELECT` (assigning aliases), sorts the results in `ORDER BY`, and finally cuts off the output using `LIMIT`."

### 7. Common Mistakes
* **Trying to use a SELECT alias in WHERE:** As shown above, this will cause a compile error. You must repeat the column logic or wrap the query in a CTE.

### 8. Follow-up Questions
* *Why can you use SELECT aliases inside the `ORDER BY` clause? (Because ORDER BY runs after SELECT).*
* *Does the execution order change if the query optimizer decides to re-order tasks? (The logical order remains strict, but physical execution may vary for optimization).*

---

## 23. Normalization Basics

### 1. Definition
Normalization is the process of organizing database columns and tables to reduce data redundancy and improve data integrity.
* **1NF (First Normal Form):** Atomic values (no nested lists in columns) and unique rows.
* **2NF (Second Normal Form):** Meets 1NF, and all non-key columns depend fully on the primary key (no partial dependencies on composite keys).
* **3NF (Third Normal Form):** Meets 2NF, and no non-key columns depend on other non-key columns (no transitive dependencies).

### 2. Why It Exists
If customer addresses are stored directly in every row of an `orders` table, updating a customer's address requires updating thousands of rows. Normalization splits this into separate tables, reducing redundancy.

### 3. Where It Is Used
Designing transactional system schemas (OLTP).

### 4. Syntax
Normalization is a design philosophy, not a code syntax. It involves designing tables with foreign keys:
```sql
CREATE TABLE customers (id INT PRIMARY KEY, name VARCHAR(50), address VARCHAR(100));
CREATE TABLE orders (id INT PRIMARY KEY, customer_id INT REFERENCES customers(id), amount DECIMAL);
```

### 5. Simple Example
* **Non-Normalized (Violating 3NF):** A table storing `employee_id`, `department_name`, and `department_manager`. The manager depends on the department name, not the employee ID.
* **Normalized:** Split into `employees` (pointing to a `department_id`) and `departments` (storing `id`, `name`, and `manager`).

### 6. Interview Explanation
> "Normalization is about splitting data to ensure that 'every fact is stored in exactly one place.' 1NF gets rid of comma-separated values in columns. 2NF removes redundancy on composite keys. 3NF ensures that non-key attributes only depend on the primary key, preventing update anomalies where updating a single value requires modifying multiple rows."

### 7. Common Mistakes
* **Over-normalizing OLAP systems:** Normalizing analytical data warehouses causes too many joins, slowing down read queries. Analytical databases are often denormalized for speed.

### 8. Follow-up Questions
* *What is Denormalization and when do you use it?*
* *What is the difference between OLTP (Online Transaction Processing) and OLAP (Online Analytical Processing)?*

---

## 24. Transactions Basics (COMMIT, ROLLBACK, ACID)

### 1. Definition
A Transaction is a single logical unit of database work containing one or more SQL queries.
* **COMMIT:** Saves the changes permanently.
* **ROLLBACK:** Discards all changes made in the active transaction block.
* **ACID:** The four properties of transactions: **Atomicity** (all-or-nothing), **Consistency** (valid state), **Isolation** (concurrent runs don't interfere), and **Durability** (persisted after crash).

### 2. Why It Exists
If a banking server transfers $100 from Account A to Account B, it must execute two queries: deduct $100 from A, and add $100 to B. If the server crashes between the queries, money vanishes. A transaction ensures both queries succeed, or both fail.

### 3. Where It Is Used
Financial ledger entries, booking seats, processing shopping cart checkouts.

### 4. Syntax
```sql
BEGIN TRANSACTION; -- or START TRANSACTION
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT; -- Save! (Or ROLLBACK if an error occurs)
```

### 5. Simple Example
```sql
BEGIN;
INSERT INTO orders (id, amount) VALUES (1, 99.99);
-- If the insert succeeds, we write to inventory
UPDATE inventory SET stock = stock - 1 WHERE item_id = 45;
COMMIT;
```

### 6. Interview Explanation
> "Transactions guarantee database reliability using ACID. Atomicity ensures that if any single query inside a block fails, the whole block is rolled back. Isolation ensures concurrent users don't see half-written states, and Durability guarantees committed data survives a power outage."

### 7. Common Mistakes
* **Keeping transactions open too long:** Holding transactions open while making external HTTP API calls. If the API is slow, the database keeps locks on rows, blocking other database updates and freezing your application. Always run API calls *before* starting a transaction.

### 8. Follow-up Questions
* *What are Transaction Isolation Levels? (Read Uncommitted, Read Committed, Repeatable Read, Serializable).*
* *What is a Dirty Read?*

---

## 25. SQL Interview Traps

### 1. Definition
Interview traps are common trick scenarios interviewers use to test if you truly understand database mechanics rather than having memorized basic queries.

### 2. The Trap Scenarios:

#### Trap A: NULL comparison trap
* **The trick:** The interviewer asks you to query users where `country != 'US'`.
* **The trap:** If a user has `NULL` in the `country` column, they will be **excluded** by `!= 'US'`. Because comparing anything to `NULL` yields `UNKNOWN`, those rows are dropped.
* **The fix:** Always explicitly check for NULLs: `WHERE country != 'US' OR country IS NULL`.

#### Trap B: COUNT(column_name) on left joins
* **The trick:** Count the number of orders per customer, including those with 0 orders.
* **The trap:** Writing `COUNT(*)` with a left join:
  ```sql
  SELECT c.name, COUNT(*) -- ❌ WRONG: Counts 1 for customers with 0 orders!
  FROM customers c
  LEFT JOIN orders o ON c.id = o.customer_id
  GROUP BY c.name;
  ```
  If a customer has 0 orders, the left join still returns 1 row (with NULL order columns). `COUNT(*)` counts that row, returning `1`.
* **The fix:** Count the foreign key column: `COUNT(o.order_id)`. Since it is `NULL` for customers with no orders, it will correctly return `0`.

#### Trap C: Aggregate filters in WHERE
* **The trick:** Find the employee with the highest salary.
* **The trap:** Writing `WHERE salary = MAX(salary)`.
* **The fix:** Use a subquery: `WHERE salary = (SELECT MAX(salary) FROM employees)`.

---

*← [Previous: Chapter 6 (Window Functions)](06_Window_Functions.md) | [Home (README) →](../README.md)*
