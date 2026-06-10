# Chapter 6: Window Functions — "Inline Analytical Analytics"

---

## 20. Window Functions

### 1. Definition
A Window Function performs a calculation across a set of table rows that are related to the current row. Unlike regular group-by aggregates, window functions do **not** collapse rows; they preserve the individual row identity in the output.

### 2. Why It Exists
Before window functions, calculating running totals, rankings, or comparing values between rows required writing complex, slow self-joins or correlated subqueries. Window functions calculate these metrics efficiently in a single pass over partitions.

### 3. Where It Is Used
Financial reports (running totals), user behavior (time delta between consecutive logins), leaderboards (ranking records inside categories), and calculating moving averages.

### 4. Syntax
```sql
SELECT column_name,
       WINDOW_FUNCTION() OVER (
           PARTITION BY partition_column
           ORDER BY sort_column
       ) AS alias_name
FROM table_name;
```

---

## Window Functions Breakdown

### 1. ROW_NUMBER(), RANK(), and DENSE_RANK()
* **`ROW_NUMBER()`:** Assigns a sequential integer starting at 1. If there's a tie, it assigns numbers arbitrarily.
* **`RANK()`:** Assigns ranking numbers. If there's a tie, they get the same rank, and the next rank is **skipped** (e.g. 1, 2, 2, 4).
* **`DENSE_RANK()`:** Assigns ranking numbers. If there's a tie, they get the same rank, and the next rank is **not skipped** (e.g. 1, 2, 2, 3).

#### Example:
```sql
SELECT employee_id, department_id, salary,
       ROW_NUMBER() OVER(PARTITION BY department_id ORDER BY salary DESC) as row_num,
       RANK() OVER(PARTITION BY department_id ORDER BY salary DESC) as rnk,
       DENSE_RANK() OVER(PARTITION BY department_id ORDER BY salary DESC) as dense_rnk
FROM employees;
```

---

### 2. LAG() and LEAD()
* **`LAG(column, offset)`:** Accesses data from a previous row in the partition at the specified offset (default offset is 1).
* **`LEAD(column, offset)`:** Accesses data from a subsequent row in the partition.

#### Example:
```sql
-- Find previous and next order dates per customer
SELECT customer_id, order_date,
       LAG(order_date, 1) OVER(PARTITION BY customer_id ORDER BY order_date ASC) as prev_order,
       LEAD(order_date, 1) OVER(PARTITION BY customer_id ORDER BY order_date ASC) as next_order
FROM orders;
```

---

### 3. SUM() OVER and AVG() OVER
* **`SUM(column) OVER()`:** Calculates running totals.
* **`AVG(column) OVER()`:** Calculates running averages or partition averages.

#### Example:
```sql
-- Calculate running revenue total per day
SELECT sales_date, revenue,
       SUM(revenue) OVER(ORDER BY sales_date ASC) as running_total
FROM sales;
```

---

## Simple Example (All Combined)

Given an `employees` table:

| id | dept | salary |
|---|---|---|
| 1 | IT | 10000 |
| 2 | IT | 10000 |
| 3 | HR | 8000 |

```sql
SELECT dept, salary,
       RANK() OVER (PARTITION BY dept ORDER BY salary DESC) as rnk,
       SUM(salary) OVER (PARTITION BY dept) as dept_total
FROM employees;
```

Output:
| dept | salary | rnk | dept_total |
|---|---|---|---|
| IT | 10000 | 1 | 20000 |
| IT | 10000 | 1 | 20000 |
| HR | 8000 | 1 | 8000 |

---

## Interview Explanation
> "Window functions are like overlaying a magnifying glass on partitions of data. The `OVER` clause defines the 'window' of rows. `PARTITION BY` divides the rows into groups, `ORDER BY` sorts the rows within each group, and the function calculates a value for each row. The unique feature is that it leaves the row structure unchanged, returning the aggregate next to the raw details."

---

## Common Mistakes & Edge Cases

### 1. Trying to filter window function outputs inside `WHERE`
This is a classic SQL mistake. If you write:
```sql
-- ❌ WRONG: Will fail because window functions run AFTER WHERE!
SELECT employee_id, 
       RANK() OVER(ORDER BY salary DESC) as rnk
FROM employees
WHERE RANK() OVER(ORDER BY salary DESC) = 1;
```
Because `WHERE` is executed *before* window functions are calculated, the column `rnk` doesn't exist yet when `WHERE` runs.
* **Fix:** Use a Subquery or a CTE:
```sql
-- ✅ CORRECT
WITH RankedEmployees AS (
    SELECT employee_id, 
           RANK() OVER(ORDER BY salary DESC) as rnk
    FROM employees
)
SELECT employee_id
FROM RankedEmployees
WHERE rnk = 1;
```

### 2. Forgetting `ORDER BY` inside running totals
If you write `SUM(val) OVER (PARTITION BY id)`, it will sum the *entire* partition and display that total on every row. You must include `ORDER BY` inside `OVER` to tell the engine to calculate a **running** cumulative total row-by-row.

---

## Interview Questions (Top 5)

**Q1: What is the difference between ROW_NUMBER(), RANK(), and DENSE_RANK()?**
> * `ROW_NUMBER()` assigns a unique, sequential number to each row, regardless of duplicate values in the sort column.
> * `RANK()` assigns identical numbers to duplicate values. If a tie occurs, it skips the next ranking numbers (e.g. 1, 2, 2, 4).
> * `DENSE_RANK()` also assigns identical numbers to duplicates, but does not skip any ranks (e.g. 1, 2, 2, 3).

**Q2: How does `LAG()` work, and how do you handle cases where there is no previous row?**
> `LAG(col, offset)` pulls data from a row that sits a specific offset behind the current row. If the current row is the first row in the partition, there is no previous row, so `LAG()` returns `NULL`. You can override this default behavior by passing a third default argument: `LAG(col, 1, 'No Previous')`.

**Q3: Why can't you write a window function calculation directly inside the WHERE or HAVING clauses?**
> Relational databases execute SQL clauses in a strict sequence. `WHERE` (filtering rows) and `HAVING` (filtering groups) are evaluated early in the pipeline. Window functions require the final sorted set, so they are evaluated very late—just before the final `SELECT` is returned. Therefore, their values do not exist yet when `WHERE` or `HAVING` are running.

**Q4: How do you calculate a moving average (e.g. 3-day average) using window functions?**
> By specifying the frame boundaries using the `ROWS` or `RANGE` sub-clause inside `OVER()`:
> ```sql
> AVG(revenue) OVER(
>     ORDER BY sales_date ASC 
>     ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
> )
> ```
> This tells the engine to calculate the average of the current row and the two preceding rows.

**Q5: What is the default window frame boundary if you write `ORDER BY` inside the `OVER` clause but omit the frame boundaries?**
> The default is `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`. This calculates the aggregate from the start of the partition up to the current row's value, which behaves as a running total.

---

*← [Previous: Chapter 5 (Subqueries & CTEs)](05_Subqueries_CTEs.md) | [Next: Chapter 7 (Database Internals & Performance) →](07_Database_Internals.md)*
