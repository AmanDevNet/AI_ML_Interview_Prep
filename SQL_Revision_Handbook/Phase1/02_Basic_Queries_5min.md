# Chapter 2: Basic Queries & Filtering

---

## 6. SELECT

### 1. Definition
The `SELECT` statement is used to retrieve columns of data from a database table.

### 2. Why It Exists
Tables store raw data. `SELECT` allows you to fetch only the specific properties you need, reducing network bandwidth and query execution time.

### 3. Where It Is Used
Virtually every SQL query starts with `SELECT`.

### 4. Syntax
```sql
SELECT column_1, column_2 FROM table_name;
```

### 5. Simple Example
```sql
SELECT name, price FROM products;
```

### 6. Interview Explanation
> "The `SELECT` statement acts as the projections filter of a database query. It tells the engine, 'Out of the dozens of columns in this table, only retrieve and format these specific ones for the result set.'"

### 7. Common Mistakes
* **Using `SELECT *` in production:** `SELECT *` fetches all columns. If someone adds a large binary or text column to the table later, your query will slow down and load redundant data. Always list columns explicitly.

### 8. Follow-up Questions
* *How does the query optimizer handle `SELECT *` differently than named columns?*
* *Can we rename columns in the select statement output? (Yes, using the `AS` alias keyword).*

---

## 7. WHERE

### 1. Definition
The `WHERE` clause is used to filter records, extracting only the rows that fulfill a specified condition.

### 2. Why It Exists
We rarely want to fetch millions of table rows. `WHERE` filters out irrelevant data at the database level before loading it into memory.

### 3. Where It Is Used
Filtering by dates, status checks, user IDs, and text search matches.

### 4. Syntax
```sql
SELECT columns 
FROM table_name 
WHERE column_name = value;
```

### 5. Simple Example
```sql
SELECT name 
FROM users 
WHERE country = 'US' AND status = 'active';
```

### 6. Interview Explanation
> "The `WHERE` clause is the row-level filter. It executes early in the database engine's lifecycle, matching every row against a condition. If a row returns `TRUE`, it moves to the next phase; if it returns `FALSE` or `UNKNOWN` (NULL), it gets discarded."

### 7. Common Mistakes
* **Filtering on unindexed columns:** If you filter a massive table by a column that has no index, the database engine has to perform a slow "Full Table Scan" (checking every row from top to bottom).
* **Using aggregate functions in WHERE:** Writing `WHERE salary > AVG(salary)` will crash. Aggregate functions are evaluated *after* `WHERE`. To filter by aggregates, you must use `HAVING` or a subquery.

### 8. Follow-up Questions
* *What is the difference between `AND` and `OR` precedence in a WHERE clause?*
* *How do you write a pattern-matching filter in a WHERE clause? (Using `LIKE` or `ILIKE` with wildcards like `%`).*

---

## 8. ORDER BY

### 1. Definition
The `ORDER BY` clause is used to sort the result set in ascending (`ASC`) or descending (`DESC`) order.

### 2. Why It Exists
By default, relational databases return data in no guaranteed order (it depends on how rows are laid out on disk). `ORDER BY` ensures deterministic sorting.

### 3. Where It Is Used
Sorting leaderboards, timelines, prices from low to high, or recent transactions first.

### 4. Syntax
```sql
SELECT columns 
FROM table 
ORDER BY column_name DESC;
```

### 5. Simple Example
```sql
SELECT name, score 
FROM players 
ORDER BY score DESC, name ASC; -- Sorts by score first, then alphabetically
```

### 6. Interview Explanation
> "Since relational databases return rows as an unordered set, `ORDER BY` is necessary to guarantee sorting. It runs late in the query execution order because sorting can be expensive if the data isn't indexed, forcing the engine to run an in-memory sort."

### 7. Common Mistakes
* **Sorting large datasets without indexes:** Sorting millions of rows on a non-indexed column forces the database to write temporary files to disk to perform the sort, slowing down operations.
* **Confusing column positional numbers:** Writing `ORDER BY 2` works (sorts by the second column in the SELECT list), but it is a bad practice because if you change the SELECT list order later, your query will silently sort by the wrong data.

### 8. Follow-up Questions
* *Is sorting ascending or descending by default? (Ascending).*
* *How are `NULL` values sorted in an ORDER BY clause? (Usually NULLs go first in ASC and last in DESC, but you can control this using `NULLS FIRST` or `NULLS LAST` in Postgres).*

---

## 9. LIMIT / TOP

### 1. Definition
`LIMIT` (or `TOP` in SQL Server) restricts the maximum number of rows returned by a query.

### 2. Why It Exists
It prevents queries from returning millions of rows to the caller, saving network bandwidth and memory.

### 3. Where It Is Used
Pagination (using `LIMIT` and `OFFSET`), and fetching the single highest or lowest value (e.g. `ORDER BY salary DESC LIMIT 1`).

### 4. Syntax
```sql
-- PostgreSQL / MySQL
SELECT columns FROM table LIMIT count;

-- SQL Server
SELECT TOP count columns FROM table;
```

### 5. Simple Example
```sql
-- Fetch the 5 most recent orders
SELECT order_id, order_date 
FROM orders 
ORDER BY order_date DESC 
LIMIT 5;
```

### 6. Interview Explanation
> "I use `LIMIT` to cap the output size. It's especially useful for API pagination when combined with `OFFSET` (e.g., `LIMIT 10 OFFSET 20` to fetch page 3). However, I always make sure to use `LIMIT` alongside `ORDER BY` to guarantee that the results are deterministic."

### 7. Common Mistakes
* **Using `LIMIT` without `ORDER BY`:** If you do not sort, the database will return *any* arbitrary rows that it finds first, which will vary between query runs.
* **High OFFSET performance issues:** Using high offsets (e.g. `LIMIT 10 OFFSET 1000000`) is slow because the database engine still has to read and discard 1,000,000 rows before returning the 10 rows.

### 8. Follow-up Questions
* *How do you optimize pagination on massive tables when OFFSET becomes too slow? (Using Keyset Pagination / Cursor-based Pagination).*
* *What is the difference between LIMIT and FETCH FIRST in SQL standards?*

---

## 10. DISTINCT

### 1. Definition
The `DISTINCT` keyword is used to remove duplicate rows from the query output, returning only unique values.

### 2. Why It Exists
When querying columns that contain duplicate values (like user countries), `DISTINCT` collapses duplicates to show you only the unique list of values.

### 3. Where It Is Used
Finding unique visitor IDs, list of countries where you have customers, or unique transaction types.

### 4. Syntax
```sql
SELECT DISTINCT column_name FROM table_name;
```

### 5. Simple Example
```sql
-- Find all unique departments that have employees
SELECT DISTINCT department_id FROM employees;
```

### 6. Interview Explanation
> "The `DISTINCT` keyword is a set operator that collapses duplicates. It does this by sorting the rows or building a hash table in memory to filter out repeats. Because of this, using it on large datasets can add CPU and memory overhead."

### 7. Common Mistakes
* **Using DISTINCT blindly to fix bad joins:** If a query returns duplicate rows because of a bad join condition, beginners often slap `DISTINCT` on top to hide the duplicates. This degrades database speed. Fix the join condition instead.
* **Assuming `DISTINCT` applies to only one column:** In `SELECT DISTINCT col1, col2`, it checks for unique *combinations* of both columns, not just `col1`.

### 8. Follow-up Questions
* *What is the difference between `SELECT DISTINCT` and `GROUP BY`?*
* *How does DISTINCT handle `NULL` values? (It returns a single `NULL` row).*

---

## 11. NULL Handling

### 1. Definition
`NULL` represents a missing or unknown value. You cannot compare `NULL` using standard operators (`=`, `!=`, `<`). You must use `IS NULL`, `IS NOT NULL`, or functions like `COALESCE()`.

### 2. Why It Exists
To differentiate between a known value of zero (or an empty string) and data that is completely missing or unknown.

### 3. Where It Is Used
Checking for incomplete profiles, optional fields, or handling unmatched outer join results.

### 4. Syntax
```sql
SELECT columns FROM table WHERE column IS NULL;

-- COALESCE returns the first non-null argument
SELECT COALESCE(column, default_value) FROM table;
```

### 5. Simple Example
```sql
-- Find users who haven't set their phone number, default to 'N/A'
SELECT name, COALESCE(phone, 'N/A') 
FROM users 
WHERE phone IS NULL;
```

### 6. Interview Explanation
> "In SQL, `NULL` represents the unknown. It behaves according to Three-Valued Logic (True, False, Unknown). Because `NULL` is not a value, writing `column = NULL` is always `UNKNOWN` and returns zero rows. I must use `IS NULL` to check for missing values, or `COALESCE` to swap `NULL`s with defaults."

### 7. Common Mistakes
* **Writing `WHERE column = NULL`:** This is the most common SQL rookie mistake. It will return an empty set. Always write `WHERE column IS NULL`.
* **Not accounting for NULLs in math calculations:** In SQL, any arithmetic operation with a `NULL` returns `NULL` (e.g. `10 + NULL` is `NULL`). Use `COALESCE(col, 0)` before doing addition.

### 8. Follow-up Questions
* *What is the difference between `COALESCE` and `NULLIF`?*
* *How does `COUNT(column)` handle NULLs compared to `COUNT(*)`? (COUNT(col) ignores NULLs, COUNT(*) counts everything).*

---

## 12. CASE WHEN (Conditional Logic)

### 1. Definition
The `CASE` statement is SQL's way of executing conditional logic (`if-then-else`) within a query.

### 2. Why It Exists
To categorize data, calculate conditional flags, or transform values directly in the SELECT statement without needing to process the values in backend application loops.

### 3. Where It Is Used
Transforming score metrics to grades, dividing age into brackets, or pivoting columns.

### 4. Syntax
```sql
SELECT 
    CASE 
        WHEN condition THEN result_1
        ELSE result_2
    END AS alias_name
FROM table_name;
```

### 5. Simple Example
```sql
-- Classify employee salaries
SELECT name,
       CASE 
           WHEN salary >= 100000 THEN 'High'
           WHEN salary >= 50000 THEN 'Medium'
           ELSE 'Low'
       END AS salary_bracket
FROM employees;
```

### 6. Interview Explanation
> "The `CASE WHEN` statement runs sequential conditional checks. It is evaluated for every row. As soon as a condition is met, it returns the value and stops evaluating the remaining branches, acting exactly like an `if-elif-else` block in Python."

### 7. Common Mistakes
* **Forgetting the `END` keyword:** This will trigger a syntax error.
* **Mismatched datatypes in THEN branches:** Returning a string in the first `WHEN` and an integer in the `ELSE` will cause a type collision error in most databases. All branches must return compatible datatypes.

### 8. Follow-up Questions
* *What happens if no conditions are met and there is no `ELSE` clause? (It returns `NULL`).*
* *Can you use `CASE WHEN` inside aggregate functions like `SUM(CASE WHEN...)`? (Yes, this is a common pattern to pivot table counts).*

---

*← [Previous: Chapter 1 (Relational Database Concepts)](01_SQL_Foundations.md) | [Next: Chapter 3 (Aggregations & Grouping) →](03_Aggregations_Groups.md)*
