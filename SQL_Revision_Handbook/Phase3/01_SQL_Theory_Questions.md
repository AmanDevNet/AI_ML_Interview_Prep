# Phase 3: SQL Theory Questions
### *Total Questions: 50*
### *The Definitive Concept Guide for Data and Backend Interviews*

---

## 📂 Topic Directory
1. [SELECT & WHERE (Q1 - Q5)](#1-select--where-q1---q5)
2. [GROUP BY & HAVING (Q6 - Q10)](#2-group-by--having-q6---q10)
3. [JOINs (Q11 - Q16)](#3-joins-q11---q16)
4. [NULL Handling (Q17 - Q21)](#4-null-handling-q17---q21)
5. [Subqueries & CTEs (Q22 - Q27)](#5-subqueries--ctes-q22---q27)
6. [Window Functions (Q28 - Q33)](#6-window-functions-q28---q33)
7. [Indexes & Query Optimization (Q34 - Q40)](#7-indexes--query-optimization-q34---q40)
8. [Normalization & Database Design (Q41 - Q44)](#8-normalization--database-design-q41---q44)
9. [Transactions & ACID Properties (Q45 - Q48)](#9-transactions--acid-properties-q45---q48)
10. [SQL for AI/ML Data Pipelines (Q49 - Q50)](#10-sql-for-aiml-data-pipelines-q49---q50)

---

## 1. SELECT & WHERE (Q1 - Q5)

### Q1: What is the main difference between `SELECT` list projections and row-level filtering?
> **Definition:** `SELECT` projects (selects) specific vertical columns of a dataset, whereas row-level filtering (via `WHERE`) extracts horizontal records based on conditional logic.
> 
> **Example:**
> ```sql
> SELECT name, email FROM users WHERE signup_date >= '2026-01-01';
> ```
> 
> **Common mistake:** Putting columns you want to filter in the `SELECT` list but omitting them from the `WHERE` clause, or trying to filter row-by-row on the application side.
> 
> **Interview-level explanation:** *"Think of it as slicing a grid. The `WHERE` clause slices the grid horizontally, discarding rows that don't match the condition early in the process. The `SELECT` list slices it vertically, discarding columns we don't need, which minimizes the memory footprint and network load."*

### Q2: Why is the `WHERE` clause evaluated before the `SELECT` clause?
> **Definition:** The database engine needs to filter and reduce the rows of the source dataset first before it performs calculations or assigns aliases in the SELECT list.
> 
> **Example:**
> ```sql
> SELECT first_name AS name FROM users WHERE signup_date > '2026-01-01';
> -- WHERE evaluates signup_date before name is projected.
> ```
> 
> **Common mistake:** Trying to reference a column alias created in the `SELECT` list inside the `WHERE` clause (e.g., `WHERE name = 'Alice'`).
> 
> **Interview-level explanation:** *"This is down to the logical execution order of SQL. The database engine executes `FROM` and `WHERE` first to build and filter the dataset. It executes `SELECT` much later, which is why aliases defined in the select projection aren't available to the `WHERE` clause filter."*

### Q3: When should you use `LIKE` versus `IN` in a filter?
> **Definition:** `LIKE` is used for pattern matching with wildcards (`%` or `_`) on string columns. `IN` matches values against a explicit list or a subquery result set.
> 
> **Example:**
> ```sql
> SELECT * FROM users WHERE email LIKE '%@gmail.com';
> SELECT * FROM users WHERE status IN ('active', 'pending');
> ```
> 
> **Common mistake:** Using `LIKE` when you have a list of exact matches (e.g. `LIKE 'active'`), which has higher string parsing overhead.
> 
> **Interview-level explanation:** *"I use `LIKE` for partial text searches—for example, finding Gmail addresses by matching `%@gmail.com`. I use `IN` when I have a distinct set of discrete values to match against, which is cleaner and allows the query planner to optimize checks using indexes."*

### Q4: How does the database execute a query containing `LIMIT`?
> **Definition:** `LIMIT` caps the maximum number of rows returned to the caller by stopping the query execution scan once the limit count is reached.
> 
> **Example:**
> ```sql
> SELECT name FROM users ORDER BY id ASC LIMIT 10;
> ```
> 
> **Common mistake:** Running `LIMIT` without an `ORDER BY` clause, which yields non-deterministic results that can vary between runs.
> 
> **Interview-level explanation:** *"The query planner uses a Sort-Limit or Scan-Limit operation. If there is no sorting, it scans and returns the first matching rows it hits and immediately stops. If there is an `ORDER BY`, it sorts the matching rows first and then discards everything after the threshold count."*

### Q5: What is the difference between `BETWEEN` and comparison operators?
> **Definition:** `BETWEEN` is a shorthand operator to filter values within an inclusive range (lower to upper limit). It is equivalent to `>=` and `<=`.
> 
> **Example:**
> ```sql
> SELECT * FROM orders WHERE amount BETWEEN 100 AND 200;
> -- Equivalent to: amount >= 100 AND amount <= 200
> ```
> 
> **Common mistake:** Forgetting that `BETWEEN` is inclusive on both ends, which can lead to off-by-one errors when filtering date ranges.
> 
> **Interview-level explanation:** *"I treat `BETWEEN` as syntactic sugar for matching inclusive boundaries. When filtering dates, however, I prefer using explicit comparison operators (`>=` and `<`) because standard datetimes include timestamps, and `BETWEEN` might drop transactions that happen on the final day's evening."*

---

## 2. GROUP BY & HAVING (Q6 - Q10)

### Q6: Can you explain why aggregate functions require a `GROUP BY` clause?
> **Definition:** Aggregate functions collapse multiple rows into a single value. If you SELECT both a raw column and an aggregate, you must define the grouping buckets using `GROUP BY`.
> 
> **Example:**
> ```sql
> SELECT country, COUNT(*) FROM users GROUP BY country;
> ```
> 
> **Common mistake:** Selecting a non-aggregated column that is missing from the `GROUP BY` clause.
> 
> **Interview-level explanation:** *"If you write `SELECT country, MAX(age)`, you're asking the database to perform two conflicting tasks: show individual countries and a single max age. The engine doesn't know which country to align with that single max age. Adding `GROUP BY country` divides the data into country buckets first, resolving the conflict."*

### Q7: Why can't you use `WHERE` to filter group aggregates?
> **Definition:** The `WHERE` clause filters individual rows before they are grouped. Aggregates are computed after grouping, meaning they do not exist yet when the `WHERE` clause executes.
> 
> **Example:**
> ```sql
> SELECT dept_id, SUM(salary) FROM employees GROUP BY dept_id HAVING SUM(salary) > 100000;
> ```
> 
> **Common mistake:** Trying to write `WHERE SUM(salary) > 100000`.
> 
> **Interview-level explanation:** *"This is a pipeline sequence rule. Row filtering happens early. Grouping and aggregation happen later. Since aggregate values don't exist during the row-filtering phase, you must use `HAVING` to filter after the groups have been aggregated."*

### Q8: What does the database engine do under the hood when executing `GROUP BY`?
> **Definition:** The engine partitions the dataset using either a Hash Grouping (building an in-memory hash table of keys) or a Sort Grouping (sorting the rows by key first) to calculate aggregates.
> 
> **Example:**
> ```sql
> SELECT category, AVG(price) FROM products GROUP BY category;
> ```
> 
> **Common mistake:** Grouping by columns with high cardinality (like primary keys or timestamps) which consumes massive RAM for hash tables.
> 
> **Interview-level explanation:** *"The query planner usually builds a hash table where the hash keys are the unique values of the grouped columns. As it scans the table, it routes each row to its matching key bucket, updating the running count, sum, or mean. If the memory budget is exceeded, it falls back to sorting the table by key on disk."*

### Q9: Can you use a `HAVING` clause without a `GROUP BY` clause?
> **Definition:** Yes. In this case, the entire table is treated as a single implicit group, and the `HAVING` clause filters that single output row.
> 
> **Example:**
> ```sql
> SELECT AVG(salary) FROM employees HAVING AVG(salary) > 50000;
> ```
> 
> **Common mistake:** Confusing this with a normal row filter. If the aggregate doesn't match the condition, it returns an empty result set (0 rows), not individual filtered rows.
> 
> **Interview-level explanation:** *"Yes, you can write `HAVING` without `GROUP BY`. The database treats the table as a single group. If the group aggregate satisfies the condition, you get the aggregate value; otherwise, you get nothing. However, this is rarely used in production since subqueries or CTEs are much easier to read."*

### Q10: How do you group by multiple columns, and what does it mean?
> **Definition:** Grouping by multiple columns partitions the dataset into unique combinations of those columns.
> 
> **Example:**
> ```sql
> SELECT department_id, job_title, COUNT(*) 
> FROM employees 
> GROUP BY department_id, job_title;
> ```
> 
> **Common mistake:** Assuming that grouping by `A, B` is the same as grouping by `A` then `B` separately.
> 
> **Interview-level explanation:** *"Grouping by multiple columns creates buckets for every unique combination. For example, grouping by `department` and `role` will create a bucket for 'IT - Engineer', another for 'IT - Manager', and another for 'HR - Recruiter', calculating aggregates for each combination."*

---

## 3. JOINs (Q11 - Q16)

### Q11: What is the difference between a `LEFT JOIN` and an `INNER JOIN`?
> **Definition:** `INNER JOIN` returns only rows that have matches in both tables. `LEFT JOIN` returns all rows from the left table and matches from the right table (filling missing columns with `NULL`s).
> 
> **Example:**
> ```sql
> SELECT u.name, o.id FROM users u INNER JOIN orders o ON u.id = o.user_id;
> SELECT u.name, o.id FROM users u LEFT JOIN orders o ON u.id = o.user_id;
> ```
> 
> **Common mistake:** Using `LEFT JOIN` but filtering the right table inside `WHERE`, which implicitly converts it into an `INNER JOIN`.
> 
> **Interview-level explanation:** *"I use `INNER JOIN` when I only want records that have active links on both sides. I use `LEFT JOIN` when I need a complete list of records from my primary table, even if they don't have any matching detail records in the joined table."*

### Q12: What is a `FULL OUTER JOIN`, and when is it useful?
> **Definition:** Returns all rows from both tables, matching them where possible and filling unmatched values on either side with `NULL`.
> 
> **Example:**
> ```sql
> SELECT e.name, p.name 
> FROM employees e 
> FULL OUTER JOIN projects p ON e.project_id = p.id;
> ```
> 
> **Common mistake:** Using it to join tables that have a strict parent-child relationship (use `LEFT` or `INNER` instead).
> 
> **Interview-level explanation:** *"A `FULL OUTER JOIN` acts as a union of a left join and a right join. I use it for data reconciliations—for example, comparing inventory records against sales logs to identify items present in one but missing in the other."*

### Q13: How does a `CROSS JOIN` differ from other join types?
> **Definition:** A `CROSS JOIN` creates a Cartesian product, joining every row of the first table with every row of the second table without any matching condition.
> 
> **Example:**
> ```sql
> SELECT colors.name, sizes.name FROM colors CROSS JOIN sizes;
> ```
> 
> **Common mistake:** Running a `CROSS JOIN` on large tables, which causes the output size to blow up exponentially ($N \times M$).
> 
> **Interview-level explanation:** *"Unlike standard joins that filter rows based on keys, `CROSS JOIN` multiplies the tables. If you have 10 products and 3 sizes, it generates all 30 unique combinations. It is the only join type that doesn't use an `ON` clause."*

### Q14: Explain what a `SELF JOIN` is and provide a real-world scenario.
> **Definition:** A `SELF JOIN` is a query that joins a table to itself by referencing it with distinct aliases.
> 
> **Example:**
> ```sql
> SELECT emp.name AS employee, mgr.name AS manager
> FROM employees emp
> LEFT JOIN employees mgr ON emp.manager_id = mgr.id;
> ```
> 
> **Common mistake:** Forgetting to use distinct aliases for the table, which causes a name collision syntax error.
> 
> **Interview-level explanation:** *"I use a self-join to query hierarchical data stored in a single table. For example, in an organizational chart, employees and their managers are both in the `employees` table. Joining the table to itself using `emp` and `mgr` aliases allows us to align the hierarchy."*

### Q15: How does the database execute Joins (Nested Loop, Hash, Sort-Merge)?
> **Definition:** These are the three physical algorithms used by the query optimizer:
> * **Nested Loop:** Iterates through Table A row-by-row and searches Table B.
> * **Hash Join:** Builds an in-memory hash table of Table A, then scans Table B.
> * **Sort-Merge:** Sorts both tables by join keys, then merges them in a single sweep.
> 
> **Example:** Shown in the query planner output (`EXPLAIN SELECT...`).
> 
> **Common mistake:** Assuming the database always uses nested loops, which is slow for large tables.
> 
> **Interview-level explanation:** *"The query optimizer chooses based on table size and index availability. For small datasets, it runs a nested loop. For large, unindexed tables, it builds an in-memory hash join for speed. If the dataset is massive and already indexed, it runs a sort-merge join."*

### Q16: What is a Non-Equi Join, and when would you use it?
> **Definition:** A join that combines tables using comparison operators other than equality (like `<`, `>`, or `BETWEEN`).
> 
> **Example:**
> ```sql
> SELECT e.name, b.grade
> FROM employees e
> INNER JOIN brackets b ON e.salary BETWEEN b.min_salary AND b.max_salary;
> ```
> 
> **Common mistake:** Assuming joins can only match exact primary-to-foreign keys.
> 
> **Interview-level explanation:** *"I use non-equi joins when matching ranges rather than exact IDs—for example, mapping sales timestamps to active promotions, or mapping employee salaries to salary grade brackets using `BETWEEN`."*

---

## 4. NULL Handling (Q17 - Q21)

### Q17: What is Three-Valued Logic in SQL?
> **Definition:** SQL evaluations can return three states: `TRUE`, `FALSE`, or `UNKNOWN` (NULL). 
> 
> **Example:**
> ```sql
> SELECT * FROM users WHERE age > 20; -- If age is NULL, returns UNKNOWN (row dropped)
> ```
> 
> **Common mistake:** Assuming that `NOT (column = 'value')` will return rows where the column is `NULL`.
> 
> **Interview-level explanation:** *"In SQL, `NULL` represents the unknown. Because it's unknown, comparing it to anything yields `UNKNOWN`. If a column is `NULL`, both `col = 5` and `col != 5` evaluate to `UNKNOWN` and are discarded. I must use `IS NULL` to check for missing data."*

### Q18: Why does `column = NULL` always return zero rows?
> **Definition:** `NULL` represents an unknown value, and you cannot check if an unknown is equal to another unknown. The expression evaluates to `UNKNOWN` (neither true nor false), which fails filtering.
> 
> **Example:**
> ```sql
> SELECT * FROM users WHERE status IS NULL; -- Correct
> -- SELECT * FROM users WHERE status = NULL; -- ❌ Incorrect (returns 0 rows)
> ```
> 
> **Common mistake:** Using `=` or `!=` instead of `IS NULL` or `IS NOT NULL`.
> 
> **Interview-level explanation:** *"Because `NULL` is a state of missing data, not a concrete value. In SQL logic, `unknown = unknown` is still unknown. Therefore, checking `col = NULL` evaluates to `UNKNOWN`, which is discarded by the filter. You must write `IS NULL`."*

### Q19: How does the `COALESCE` function work?
> **Definition:** `COALESCE` accepts a list of arguments and returns the first non-null value it encounters.
> 
> **Example:**
> ```sql
> SELECT name, COALESCE(phone, alt_phone, 'N/A') FROM users;
> ```
> 
> **Common mistake:** Passing mismatched datatypes into `COALESCE` (e.g. string and integer), which can cause type conversion errors.
> 
> **Interview-level explanation:** *"I use `COALESCE` to define fallback options. For instance, if a user has no primary phone, it checks their alternative phone. If that is also `NULL`, it falls back to a default value like 'N/A'. It keeps the API output clean."*

### Q20: What is the difference between `COALESCE` and `NULLIF`?
> **Definition:** `COALESCE` returns the first non-null argument. `NULLIF(arg1, arg2)` returns `NULL` if `arg1` equals `arg2`; otherwise, it returns `arg1`.
> 
> **Example:**
> ```sql
> SELECT 100 / NULLIF(clicks, 0) FROM ad_campaigns;
> ```
> 
> **Common mistake:** Confusing their roles—using `COALESCE` to prevent division-by-zero crashes.
> 
> **Interview-level explanation:** *"They serve opposite roles. `COALESCE` replaces `NULL` values with default values. `NULLIF` turns specific values (like `0`) into `NULL`. I use `NULLIF` to prevent division-by-zero errors by turning zero divisors into `NULL`."*

### Q21: How do aggregate functions handle `NULL` values?
> **Definition:** Aggregate functions (like `SUM`, `AVG`, `COUNT`) completely ignore `NULL` values, with the exception of `COUNT(*)`.
> 
> **Example:**
> ```sql
> SELECT AVG(val) FROM (SELECT 10 AS val UNION ALL SELECT NULL); -- returns 10
> ```
> 
> **Common mistake:** Assuming `COUNT(column)` returns the total number of table rows when some values in that column are `NULL`.
> 
> **Interview-level explanation:** *"Aggregates skip `NULL`s. If you have 5 rows and 2 are `NULL` in the `salary` column, `AVG(salary)` divides the sum by 3, not 5. `COUNT(salary)` will return 3. However, `COUNT(*)` counts the row structures, returning 5."*

---

## 5. Subqueries & CTEs (Q22 - Q27)

### Q22: What is the difference between a subquery and a CTE?
> **Definition:** A subquery is an inline query nested inside brackets. A CTE (Common Table Expression) is a named temporary result set defined at the top of a query using `WITH`.
> 
> **Example:**
> ```sql
> WITH CTE AS (SELECT id FROM users) SELECT * FROM CTE;
> ```
> 
> **Common mistake:** Writing massive nested subqueries, which makes code unreadable and hard to maintain.
> 
> **Interview-level explanation:** *"CTEs are cleaner. They let you write top-down, self-documenting code by assigning temporary variables to queries. Standard subqueries are nested inside clauses, which can become messy. While they are written differently, database engines optimize and execute both similarly."*

### Q23: How does a Correlated Subquery differ from a standard Subquery?
> **Definition:** A standard subquery is independent and runs once. A correlated subquery references columns from the outer query, meaning it must run for every row evaluated by the outer query.
> 
> **Example:**
> ```sql
> SELECT * FROM employees e WHERE salary > (
>     SELECT AVG(salary) FROM employees WHERE dept_id = e.dept_id
> );
> ```
> 
> **Common mistake:** Running correlated subqueries on large tables, which is slow because it behaves like a nested loop ($O(N^2)$).
> 
> **Interview-level explanation:** *"A standard subquery is computed once and its result is reused. A correlated subquery is dependent on the outer row. It behaves like a nested loop in programming: for each outer row, the inner query runs using that row's variables."*

### Q24: When should you use `EXISTS` instead of `IN`?
> **Definition:** `IN` compares values against a list in memory. `EXISTS` returns `TRUE` as soon as it finds a matching row in a subquery, stopping the scan immediately.
> 
> **Example:**
> ```sql
> SELECT * FROM customers c WHERE EXISTS (
>     SELECT 1 FROM orders o WHERE o.customer_id = c.id
> );
> ```
> 
> **Common mistake:** Using `NOT IN` with subqueries that return `NULL` values, which causes the outer query to return zero rows.
> 
> **Interview-level explanation:** *"I prefer `EXISTS` for checking relationships. It stops scanning as soon as it finds a match, which is fast for large tables. I use `IN` only for small list comparisons. Furthermore, `NOT IN` fails if the subquery returns any `NULL`s, whereas `NOT EXISTS` is safe."*

### Q25: Why does `NOT IN` fail if the subquery contains a `NULL` value?
> **Definition:** If a subquery returns any `NULL`, `NOT IN` compares the outer value using `!= NULL`, which yields `UNKNOWN`. Because `UNKNOWN` is discarded, the query returns zero rows.
> 
> **Example:**
> ```sql
> -- If orders.customer_id has a NULL, this query returns nothing:
> SELECT * FROM customers WHERE id NOT IN (SELECT customer_id FROM orders);
> ```
> 
> **Common mistake:** Assuming `NOT IN` will ignore `NULL` values automatically.
> 
> **Interview-level explanation:** *"This is a Three-Valued Logic trap. `NOT IN (1, 2, NULL)` translates to `id != 1 AND id != 2 AND id != NULL`. Since `id != NULL` is `UNKNOWN`, the entire `AND` expression becomes `UNKNOWN`. The query returns nothing. You must filter out `NULL`s in the subquery or use `NOT EXISTS` instead."*

### Q26: What is a Recursive CTE, and when would you use it?
> **Definition:** A Recursive CTE is a CTE that references itself, containing an anchor query joined to a recursive query using `UNION` or `UNION ALL`.
> 
> **Example:**
> ```sql
> WITH RECURSIVE OrgChart AS (
>     SELECT id, manager_id FROM employees WHERE manager_id IS NULL -- Anchor
>     UNION ALL
>     SELECT e.id, e.manager_id FROM employees e JOIN OrgChart o ON e.manager_id = o.id -- Recursive
> )
> SELECT * FROM OrgChart;
> ```
> 
> **Common mistake:** Forgetting a termination condition, which causes infinite loop crashes.
> 
> **Interview-level explanation:** *"I use recursive CTEs to query hierarchical or graph data structures—like organizational reporting structures, file directories, or network paths. The anchor query sets the starting point, and the recursive query joins back to the anchor's output repeatedly until no matches remain."*

### Q27: Does using a CTE improve query performance?
> **Definition:** No, in modern databases, CTEs are generally not materialized in memory. The query planner merges the CTE into the main query, optimizing it the same way as subqueries.
> 
> **Example:** Shown in execution plans (`EXPLAIN`).
> 
> **Common mistake:** Assuming that splitting a query into 5 CTEs makes it faster. It only improves code readability.
> 
> **Interview-level explanation:** *"Generally, no. CTEs are logical wrappers for readability. The database optimizer compiles the CTEs and main query into a single execution plan. However, in Postgres pre-12, CTEs acted as optimization fences (materializing in memory), which could slow things down. Modern Postgres allows configuring this using `AS MATERIALIZED` or `NOT MATERIALIZED`."*

---

## 6. Window Functions (Q28 - Q33)

### Q28: How do Window Functions differ from `GROUP BY` aggregations?
> **Definition:** `GROUP BY` collapses input rows into a single summary row. A Window Function calculates values over a partition of rows but preserves each individual row in the output.
> 
> **Example:**
> ```sql
> SELECT name, salary, AVG(salary) OVER(PARTITION BY dept_id) FROM employees;
> ```
> 
> **Common mistake:** Trying to reference a window function calculation in the `WHERE` clause.
> 
> **Interview-level explanation:** *"Both perform aggregations, but `GROUP BY` collapses the dataset. Window functions let you calculate averages or rankings over a group while keeping the detailed rows intact. The aggregated value is simply appended as a column to each raw row."*

### Q29: What is the difference between `RANK()`, `DENSE_RANK()`, and `ROW_NUMBER()`?
> **Definition:** 
> * `ROW_NUMBER()`: Assigns unique sequential numbers (1, 2, 3, 4).
> * `RANK()`: Assigns identical ranks to ties and **skips** the next numbers (1, 2, 2, 4).
> * `DENSE_RANK()`: Assigns identical ranks to ties and **does not skip** numbers (1, 2, 2, 3).
> 
> **Example:**
> ```sql
> SELECT val,
>        ROW_NUMBER() OVER(ORDER BY val),
>        RANK() OVER(ORDER BY val),
>        DENSE_RANK() OVER(ORDER BY val)
> FROM (VALUES (10), (10), (20)) AS t(val);
> ```
> 
> **Common mistake:** Using `ROW_NUMBER` to rank items where you need to handle ties fairly.
> 
> **Interview-level explanation:** *"It comes down to how they handle ties. If two elements share the same value, `ROW_NUMBER` arbitrarily assigns them 1 and 2. `RANK` gives both 1, and skips to 3. `DENSE_RANK` gives both 1, and assigns 2 to the next item, leaving no gaps."*

### Q30: What is the purpose of `LAG()` and `LEAD()`, and how are they used in trend analysis?
> **Definition:** `LAG` reads values from a previous row at a specific offset. `LEAD` reads values from a subsequent row.
> 
> **Example:**
> ```sql
> SELECT price, LAG(price, 1) OVER(ORDER BY date) FROM stock_prices;
> ```
> 
> **Common mistake:** Forgetting to sort using `ORDER BY` inside `OVER()`, which yields values based on unordered records.
> 
> **Interview-level explanation:** *"I use them to calculate trends—like month-over-month growth. By using `LAG()`, I can compare today's sales with yesterday's sales on the same row, making it easy to calculate percentage differences without running self-joins."*

### Q31: What is a Window Frame, and what is the difference between `ROWS` and `RANGE`?
> **Definition:** A Window Frame defines the boundaries of the subset of rows within the partition used for the current row's calculation.
> * `ROWS` defines the boundary by counting physical rows.
> * `RANGE` defines the boundary by comparing values in the sorted column.
> 
> **Example:**
> ```sql
> SUM(revenue) OVER(ORDER BY date ROWS BETWEEN 2 PRECEDING AND CURRENT ROW)
> ```
> 
> **Common mistake:** Omitting boundaries, which defaults to `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`, potentially causing slow performance on large datasets.
> 
> **Interview-level explanation:** *"Frames define the sliding window. `ROWS` counts rows, so `3 PRECEDING` is exactly 3 rows back. `RANGE` checks values, which is useful when dealing with dates: if a date is missing in the sequence, `RANGE` adjusts accordingly based on the values rather than physical row counts."*

### Q32: How do you optimize a query that has multiple window functions?
> **Definition:** You can define a shared window specification at the bottom of the query using the `WINDOW` clause to prevent duplicate definitions.
> 
> **Example:**
> ```sql
> SELECT name,
>        RANK() OVER w,
>        DENSE_RANK() OVER w
> FROM employees
> WINDOW w AS (PARTITION BY dept_id ORDER BY salary DESC);
> ```
> 
> **Common mistake:** Repeating the same `OVER (PARTITION BY ...)` statement multiple times, which makes editing prone to errors.
> 
> **Interview-level explanation:** *"I use the `WINDOW` clause to declare a shared window specification. It makes the code cleaner. More importantly, it signals the query optimizer to sort the dataset once and reuse that sorted state, rather than performing duplicate sorts."*

### Q33: How does the database engine sort data for window functions?
> **Definition:** The engine uses a Sort operator to organize rows by the `PARTITION BY` and `ORDER BY` columns before scanning the rows sequentially to calculate the window aggregates.
> 
> **Example:** Appears as a `Sort` step in `EXPLAIN` execution plans.
> 
> **Common mistake:** Not indexing the columns used in `PARTITION BY` and `ORDER BY`, which forces the engine to run an expensive in-memory sort.
> 
> **Interview-level explanation:** *"The engine must group and sort the partition. If there's an index matching the partition and sort keys, it reads the rows in order (`O(1)` sorting). If not, it executes a sort in memory. Once sorted, it calculates values by sliding a pointer frame across the rows."*

---

## 7. Indexes & Query Optimization (Q34 - Q40)

### Q34: How does a B-Tree index speed up database lookups?
> **Definition:** A B-Tree (Balanced Tree) index is a self-balancing search tree that organizes column keys in a sorted hierarchy, allowing searches in logarithmic time.
> 
> **Example:** Created using `CREATE INDEX`.
> 
> **Common mistake:** Thinking indexes speed up every query. They actually slow down writes (`INSERT`/`UPDATE`/`DELETE`) because the database has to update the tree.
> 
> **Interview-level explanation:** *"A B-Tree index keeps data sorted in a tree structure. When you search for `id = 500`, the engine does a binary search through the tree nodes instead of scanning the table sequentially. Lookups drop from $O(N)$ to $O(\log N)$ time, which turns table scans into fast index lookups."*

### Q35: What is a Covered Index, and why is it highly performant?
> **Definition:** A Covered Index is an index that contains all the columns requested in the `SELECT` list, allowing the engine to return the data directly from the index structure without fetching the actual table row.
> 
> **Example:**
> ```sql
> CREATE INDEX idx_users_name_email ON users (name, email);
> -- SELECT email FROM users WHERE name = 'Alice'; -- Covered!
> ```
> 
> **Common mistake:** Selecting unnecessary columns (like `SELECT *`) which prevents the index from covering the query.
> 
> **Interview-level explanation:** *"Usually, when an index matches a filter, the engine finds the row pointer and does a second read (a Heap Lookup) to get the other columns. A covered index has all the requested columns inside the index tree itself. The engine skips the heap lookup entirely, reducing disk reads."*

### Q36: Explain the Left-Prefix Rule (Most-Selective-First) in Composite Indexes.
> **Definition:** A Composite Index (on multiple columns) is sorted from left to right. The engine can only use the index if your query filters include the leftmost column.
> 
> **Example:**
> ```sql
> CREATE INDEX idx_test ON users (country, city);
> -- WHERE country = 'US' AND city = 'NY' -- ✅ Uses index
> -- WHERE city = 'NY'                  -- ❌ Bypasses index!
> ```
> 
> **Common mistake:** Creating a composite index on `(A, B)` but writing queries that only filter by `B`.
> 
> **Interview-level explanation:** *"A composite index is like a phone book sorted by Last Name then First Name. You can't search by First Name alone without flipping page-by-page. The engine must have the leftmost column (`country`) to navigate the tree structure."*

### Q37: What is the difference between a Clustered and a Non-Clustered index?
> **Definition:** A Clustered Index dictates the physical sorting order of the actual rows on disk. A Non-Clustered Index is a separate structure containing copy keys and pointers back to the rows.
> 
> **Example:** In Postgres, primary keys are clustered using the `CLUSTER` command manually; in MySQL (InnoDB), primary keys are clustered automatically.
> 
> **Common mistake:** Trying to create multiple clustered indexes on a single table (you can only have one because rows can only be sorted on disk in one order).
> 
> **Interview-level explanation:** *"A clustered index is the table itself sorted by that key (usually the primary key). A non-clustered index is a separate index book that contains pointers to the actual rows. Because clustered indexes store raw data at leaf nodes, they are faster for range scans."*

### Q38: How do you read an `EXPLAIN` execution plan to spot slow queries?
> **Answer:**
> **Definition:** `EXPLAIN` outputs the execution plan chosen by the query planner, showing step-by-step physical operations.
> 
> **Example:**
> ```sql
> EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'test@test.com';
> ```
> 
> **Common mistake:** Reading `EXPLAIN` without `ANALYZE` (which only shows estimates, not actual execution times).
> 
> **Interview-level explanation:** *"I look for two main red flags: 'Seq Scan' (indicating a full table scan on a large table) and 'Temp Write' (sorting on disk). I want to see 'Index Scan' or 'Index Only Scan'. Running it with `ANALYZE` displays actual runtime and loops, letting me see which join or filter is the bottleneck."*

### Q39: Why does applying functions on indexed columns in a `WHERE` clause disable the index?
> **Definition:** The database index is sorted by the raw column values, not the results of functions. Applying a function forces the engine to run the function on every row, triggering a full table scan.
> 
> **Example:**
> ```sql
> -- ❌ Bypasses index:
> SELECT * FROM users WHERE UPPER(email) = 'TEST@TEST.COM';
> -- ✅ Uses index:
> SELECT * FROM users WHERE email = 'test@test.com';
> ```
> 
> **Common mistake:** Writing `WHERE DATE(created_at) = '2026-01-01'` instead of range filters.
> 
> **Interview-level explanation:** *"Because the index tree contains raw values (like 'alice@test.com'). If you write `WHERE UPPER(email) = '...'`, the engine cannot search the index tree because the keys in the index are lowercase. It has to scan the whole table and run the function on every row. To fix this, you either avoid functions in filters or create a Function-Based Index."*

### Q40: What is a Function-Based (Expression) Index?
> **Definition:** An index built on the result of an expression or function, rather than a raw column.
> 
> **Example:**
> ```sql
> CREATE INDEX idx_users_lower_email ON users (LOWER(email));
> ```
> 
> **Common mistake:** Creating a function index but writing a query that doesn't match the function expression exactly (e.g. indexing `LOWER(email)` but filtering by `UPPER(email)`).
> 
> **Interview-level explanation:** *"If you must query using a function like `LOWER(email)`, you can build a function-based index. The database pre-computes the function values for every row and stores them in the index tree, allowing index scans for queries like `WHERE LOWER(email) = '...'`."*

---

## 8. Normalization & Database Design (Q41 - Q44)

### Q41: Can you define 1NF, 2NF, and 3NF in simple terms?
> **Definition:** 
> * **1NF:** Atomic values (no nested lists in cells) and unique rows.
> * **2NF:** Meets 1NF, and non-key columns must depend on the *whole* primary key (no partial dependencies on composite keys).
> * **3NF:** Meets 2NF, and non-key columns must not depend on other non-key columns (no transitive dependencies).
> 
> **Example:** Splitting a nested JSON or list-column into a separate child table satisfies 1NF.
> 
> **Common mistake:** Assuming 3NF is always best for every database system (it can slow down analytics queries due to excessive joins).
> 
> **Interview-level explanation:** *"1NF means 'no lists inside cells.' 2NF means 'if you have a composite primary key, non-key columns cannot depend on only half of the key.' 3NF means 'non-key columns must only depend on the primary key, not on other non-key columns.' It eliminates update anomalies."*

### Q42: What is the difference between OLTP and OLAP database design?
> **Definition:** 
> * **OLTP (Transactional):** Optimized for fast writes, updates, and integrity. Highly normalized (3NF) to avoid redundancy.
> * **OLAP (Analytical):** Optimized for fast read aggregation queries. Denormalized (using Star or Snowflake schemas) to minimize joins.
> 
> **Example:** PostgreSQL/MySQL for OLTP; Snowflake/BigQuery for OLAP.
> 
> **Common mistake:** Using a highly normalized OLTP structure for analytical dashboards, causing slow joins.
> 
> **Interview-level explanation:** *"OLTP is for running the app—it needs fast, safe writes with zero redundancy, so we normalize it. OLAP is for analytics—we denormalize the data into massive tables or star schemas because joining 20 normalized tables across billions of rows for a dashboard is too slow."*

### Q43: What is a Star Schema, and how does it differ from a Snowflake Schema?
> **Definition:** A Star Schema organizes data into a central large **Fact Table** joined directly to single-level **Dimension Tables**. A Snowflake Schema normalizes the dimension tables further, splitting them into sub-tables.
> 
> **Example:**
> * Star: `fact_sales` joins directly to `dim_customers` (which contains city and country names).
> * Snowflake: `fact_sales` joins to `dim_customers`, which joins to `dim_cities`, which joins to `dim_countries`.
> 
> **Common mistake:** Designing a star schema with nested dimensions in OLTP databases.
> 
> **Interview-level explanation:** *"Star schemas are designed for query simplicity—one join takes you from fact to any dimension. Snowflake schemas normalize those dimensions to save disk space, but they require more joins, which increases query complexity and reduces analytical performance."*

### Q44: What are the risks of Denormalization?
> **Definition:** Denormalization is the deliberate introduction of redundant data to speed up reads. The risk is data inconsistency (update anomalies).
> 
> **Example:** Storing `customer_name` directly inside the `orders` table.
> 
> **Common mistake:** Denormalizing data without implementing sync mechanisms (like database triggers or transaction safeguards) to update duplicate fields when the parent changes.
> 
> **Interview-level explanation:** *"The primary risk is data drift. If I store the customer's name in both the `customers` and `orders` tables to avoid a join, and the customer changes their name, I must update it in both places. If a write fails, my database enters an inconsistent state. Writes also become heavier."*

---

## 9. Transactions & ACID Properties (Q45 - Q48)

### Q45: Explain the ACID properties with a banking transfer example.
> **Definition:** ACID defines transaction reliability:
> * **Atomicity:** All-or-nothing execution.
> * **Consistency:** Enforces constraints before and after writes.
> * **Isolation:** Concurrent transactions don't interfere.
> * **Durability:** Committed data is safe after crashes.
> 
> **Example:** Transferring $100: deduct $100 from Account A, add $100 to Account B.
> 
> **Common mistake:** Thinking ACID properties are automatically active across multiple microservices without distributed transaction protocols.
> 
> **Interview-level explanation:** *"ACID ensures safety. In a transfer, if the server crashes after deducting from A but before adding to B, **Atomicity** rolls back the entire transaction. **Consistency** ensures balances don't go negative if check constraints exist. **Isolation** prevents other users from seeing the transient state, and **Durability** guarantees that once committed, the bank records survive a power failure."*

### Q46: What are Transaction Isolation Levels, and what problems do they solve?
> **Definition:** Isolation levels define how changes made by one transaction are visible to other concurrent transactions. They are: Read Uncommitted, Read Committed, Repeatable Read, and Serializable.
> 
> **Example:** Configured using `SET TRANSACTION ISOLATION LEVEL`.
> 
> **Common mistake:** Using `Serializable` isolation level everywhere, which causes high lock contention and database timeouts.
> 
> **Interview-level explanation:** *"It's a tradeoff between consistency and speed. **Read Committed** (the default in Postgres) prevents dirty reads—you can't see uncommitted writes. **Serializable** is the highest level, executing transactions as if they ran sequentially. This prevents all anomalies, but it causes transactions to block each other, slowing down writes."*

### Q47: What is a Dirty Read, a Non-Repeatable Read, and a Phantom Read?
> **Definition:** 
> * **Dirty Read:** Transaction A reads data modified by Transaction B before B commits. If B rolls back, A has read fake data.
> * **Non-Repeatable Read:** Transaction A reads a row, Transaction B updates that row and commits. Transaction A reads it again and sees different values.
> * **Phantom Read:** Transaction A reads a set of rows matching a query, Transaction B inserts new rows and commits. Transaction A runs the query again and sees new 'phantom' rows.
> 
> **Example:** Checked using concurrent connections.
> 
> **Common mistake:** Assuming that `Read Committed` protects against phantom reads (it only protects against dirty reads).
> 
> **Interview-level explanation:** *"A dirty read is reading uncommitted draft data. A non-repeatable read is when a single row's value changes mid-transaction because someone updated it. A phantom read is when new rows appear in a range query because someone inserted them. Higher isolation levels are required to prevent these anomalies."*

### Q48: What is a Deadlock in database transactions, and how do databases resolve it?
> **Definition:** A Deadlock occurs when Transaction A locks Row 1 and waits for Row 2, while concurrent Transaction B locks Row 2 and waits for Row 1. Neither can proceed.
> 
> **Example:** Happens during concurrent updates on shared rows.
> 
> **Common mistake:** Not handling transaction retries in application code when a deadlock aborts a query.
> 
> **Interview-level explanation:** *"When a deadlock occurs, the database engine's deadlock detector spots the dependency cycle in its lock graph. It immediately aborts and rolls back one of the transactions (the 'victim'), releasing its locks so the other transaction can complete. The aborted transaction must be retried by the application."*

---

## 10. SQL for AI/ML Data Pipelines (Q49 - Q50)

### Q49: How do you prevent Data Leakage when preparing model training datasets in SQL?
> **Definition:** Data Leakage occurs when information from the future or test set accidentally leaks into the training dataset, leading to artificially high model performance during testing but poor performance in production.
> 
> **Example:**
> ```sql
> -- ❌ WRONG: Leaks future events into features!
> SELECT user_id, AVG(amount) FROM transactions GROUP BY user_id;
> -- ✅ CORRECT: Limit aggregation to events prior to the prediction cutoff date
> SELECT user_id, AVG(amount) 
> FROM transactions 
> WHERE tx_date < '2026-06-01' -- Prediction cutoff
> GROUP BY user_id;
> ```
> 
> **Common mistake:** Aggregating user features over the entire history of a table without restricting the time window to events before the prediction cutoff timestamp.
> 
> **Interview-level explanation:** *"When building training features, I always define a strict 'Cutoff Timestamp' for each record. All aggregations and feature calculations must only use data that was available *before* that cutoff. If I include any data from after the cutoff, the model will train on future information that won't be available during real-time production inference."*

### Q50: How do you build Time-Based Features (e.g. rolling averages) for ML models in SQL?
> **Definition:** Time-based features capture temporal trends (like transaction counts or average spend over the last 7, 30, or 90 days) using window functions.
> 
> **Example:**
> ```sql
> SELECT user_id, tx_date,
>        COUNT(*) OVER(
>            PARTITION BY user_id 
>            ORDER BY tx_date 
>            RANGE BETWEEN INTERVAL '7 days' PRECEDING AND '1 day' PRECEDING
>        ) AS tx_count_last_7d
> FROM transactions;
> ```
> 
> **Common mistake:** Using physical `ROWS` instead of `RANGE` time intervals, which fails to calculate values correctly if a user has days with no transactions.
> 
> **Interview-level explanation:** *"To train predictive models, we need features that represent recent user behavior. I build these in SQL using window functions with temporal range frames—for example, calculating the sum of purchases over a range of `7 days preceding`. Using `RANGE` ensures that if a user has days with no activity, the calculation stays accurate to the actual time delta, which is a robust way to engineer ML features directly inside the database."*

---

*← [Previous: Phase 2 Chapter 1 (SQL Query Patterns)](../Phase2/01_SQL_Interview_Query_Patterns.md) | [Home (README) →](../README.md)*
