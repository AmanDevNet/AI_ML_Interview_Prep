# Chapter 3: Aggregations & Grouping

---

## 13. Aggregate Functions (COUNT, SUM, AVG, MIN, MAX)

### 1. Definition
Aggregate functions perform a calculation on a set of values in a column and return a single summary value.
* `COUNT`: Counts the number of rows.
* `SUM`: Adds all numeric values together.
* `AVG`: Computes the arithmetic mean.
* `MIN`: Finds the smallest value.
* `MAX`: Finds the largest value.

### 2. Why It Exists
Databases contain raw transactions. We rarely want to fetch millions of rows to sum them up on our application server; aggregate functions allow the database engine to compute totals quickly at the storage level.

### 3. Where It Is Used
Calculating total monthly sales, average order sizes, counting signed-up users, or finding the highest/lowest price in a catalog.

### 4. Syntax
```sql
SELECT COUNT(column), SUM(column), AVG(column) FROM table_name;
```

### 5. Simple Example
```sql
SELECT COUNT(*) AS total_employees, 
       AVG(salary) AS average_salary 
FROM employees;
```

### 6. Interview Explanation
> "Aggregate functions take a vertical list of numbers and collapse them into a single metric. CPython or Node servers would take seconds to download and parse millions of records to compute an average; a relational database performs this calculation in compiled C/C++ directly on the storage disk block, returning only the final answer."

### 7. Common Mistakes
* **Assuming `COUNT(column)` and `COUNT(*)` behave the same:** `COUNT(*)` counts every row, including rows containing `NULL` values. `COUNT(column)` ignores `NULL`s. If a table has 5 rows and 2 are `NULL` in the `phone` column, `COUNT(*)` returns `5` but `COUNT(phone)` returns `3`.
* **Aggregating non-numeric types:** Calling `SUM()` on a string date column will throw a syntax/type error.

### 8. Follow-up Questions
* *How do you count only the unique values in a column? (Using `COUNT(DISTINCT column)`).*
* *What does `AVG(column)` do if there are NULL values in that column? (It completely ignores NULLs, meaning the divisor is the count of non-null values, not the total rows count).*

---

## 14. GROUP BY

### 1. Definition
The `GROUP BY` clause is used to arrange identical row data into groups. It is almost always used in combination with aggregate functions to summarize properties per group.

### 2. Why It Exists
We don't just want the average salary of the entire company; we want the average salary per department, or total sales per customer. `GROUP BY` partitions the data into buckets before executing the aggregate functions.

### 3. Where It Is Used
Revenue reports categorized by region, tracking transaction volume per user, or count of events grouped by date.

### 4. Syntax
```sql
SELECT grouping_column, AGGREGATE(value_column)
FROM table_name
GROUP BY grouping_column;
```

### 5. Simple Example
```sql
-- Calculate average salary for each department
SELECT department_id, AVG(salary) AS avg_sal
FROM employees
GROUP BY department_id;
```

### 6. Interview Explanation
> "Think of `GROUP BY` as sorting data into separate physical buckets. If I group by `department_id`, the database gathers all rows for department 1, computes the aggregate (like `AVG`), then moves to department 2, and so on. Any column in the `SELECT` list that isn't wrapped in an aggregate function **must** be listed in the `GROUP BY` clause."

### 7. Common Mistakes
* **The "Select Non-Grouped Column" Trap:** Trying to run a query like:
  ```sql
  SELECT department_id, name, AVG(salary) -- ❌ WRONG: 'name' is not in GROUP BY!
  FROM employees
  GROUP BY department_id;
  ```
  This crashes because for any department, there are multiple names. Python doesn't know which name to display alongside the single average salary. Every column in your SELECT list must either be inside an aggregate function or listed in the `GROUP BY` clause.

### 8. Follow-up Questions
* *What is the execution order of `GROUP BY` relative to `WHERE`? (WHERE executes BEFORE GROUP BY. You filter rows first, then group the remaining rows).*
* *Can you group by multiple columns? (Yes, e.g., `GROUP BY country, city` will create a bucket for every unique country-city combination).*

---

## 15. HAVING

### 1. Definition
The `HAVING` clause is used to filter groups created by the `GROUP BY` clause based on aggregate conditions.

### 2. Why It Exists
The `WHERE` clause cannot filter by aggregate results because it runs *before* grouping occurs. `HAVING` exists to filter *after* the groups have been built and aggregated.

### 3. Where It Is Used
Finding departments that have more than 10 employees, or customers whose total order values exceed $1,000.

### 4. Syntax
```sql
SELECT grouping_column, AGGREGATE(value_column)
FROM table_name
GROUP BY grouping_column
HAVING AGGREGATE(value_column) > threshold;
```

### 5. Simple Example
```sql
-- Find departments with an average salary greater than 80,000
SELECT department_id, AVG(salary) AS avg_sal
FROM employees
GROUP BY department_id
HAVING AVG(salary) > 80000;
```

### 6. Interview Explanation
> "The easiest way to contrast them is: `WHERE` is a row-level filter, whereas `HAVING` is a group-level filter. If you want to filter out low-earning employees, you use `WHERE salary > 30000` before grouping. If you want to filter out departments whose average salary is low, you must use `HAVING AVG(salary) > 30000` after grouping."

### 7. Common Mistakes
* **Using `HAVING` for conditions that belong in `WHERE`:** Writing:
  ```sql
  SELECT department_id, COUNT(*)
  FROM employees
  GROUP BY department_id
  HAVING department_id = 5; -- ❌ POOR PERFORMANCE
  ```
  This is slow because the database groups *all* departments first and then discards all except department 5. Use `WHERE department_id = 5` to discard irrelevant rows before the grouping execution begins.

### 8. Follow-up Questions
* *Can you use `HAVING` without a `GROUP BY` clause? (Yes, it behaves like a filter on the single implicit group, but it's rarely used this way).*
* *Does the database engine evaluate alias names inside HAVING? (In standard SQL, no, because HAVING evaluates before SELECT. You must repeat the aggregate formula, e.g. `HAVING AVG(salary) > 50000` rather than using the alias `HAVING avg_sal > 50000`).*

---

*← [Previous: Chapter 2 (Basic Queries & Filtering)](02_Basic_Queries.md) | [Next: Chapter 4 (Joins in Depth) →](04_Joins.md)*
