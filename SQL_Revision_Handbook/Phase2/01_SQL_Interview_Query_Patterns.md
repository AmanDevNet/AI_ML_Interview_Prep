# Phase 2: SQL Interview Query Patterns

This reference file contains the 20 most frequently asked SQL query patterns in technical screening rounds and backend/data engineering interviews.

---

## 1. Find Duplicates

### 1. Problem Statement
Find all duplicate email addresses in a `users` table.

### 2. Sample Table (`users`)
| id (INT) | email (VARCHAR) |
|---|---|
| 1 | alice@test.com |
| 2 | bob@test.com |
| 3 | alice@test.com |
| 4 | charlie@test.com |

### 3. Expected Output
| email |
|---|
| alice@test.com |

### 4. Step-by-Step Thinking
1. To find duplicates, we need to group the table rows by the `email` column.
2. For each group of duplicate emails, we count how many times it appears.
3. If the count is greater than 1, it means the email is a duplicate. We filter this using `HAVING` because `COUNT` is an aggregate function.

### 5. SQL Query
```sql
SELECT email
FROM users
GROUP BY email
HAVING COUNT(email) > 1;
```

### 6. Explanation
* `GROUP BY email` merges all identical email rows into individual buckets.
* `HAVING COUNT(email) > 1` filters out any email buckets that only contain a single row, leaving only duplicate addresses.

### 7. Interview Follow-up
* *How do you delete all duplicate rows while keeping only the one with the lowest ID?*
  ```sql
  DELETE FROM users
  WHERE id NOT IN (
      SELECT MIN(id)
      FROM users
      GROUP BY email
  );
  ```

### 8. Common Wrong Answer
```sql
-- ❌ WRONG: WHERE cannot filter aggregates!
SELECT email FROM users WHERE COUNT(email) > 1 GROUP BY email;
```

---

## 2. Find Second Highest Salary

### 1. Problem Statement
Find the second highest salary from the `employees` table. If there is no second highest, return `NULL`.

### 2. Sample Table (`employees`)
| id (INT) | salary (DECIMAL) |
|---|---|
| 1 | 10000 |
| 2 | 12000 |
| 3 | 12000 |
| 4 | 9000 |

### 3. Expected Output
| SecondHighestSalary |
|---|
| 10000 |

### 4. Step-by-Step Thinking
1. Sort unique salaries descending.
2. The second highest salary is at offset index 1.
3. If we query `LIMIT 1 OFFSET 1`, we get 10000.
4. To return `NULL` if no second highest exists (e.g. only 1 employee exists), wrap the query in a scalar select block.

### 5. SQL Query
```sql
SELECT (
    SELECT DISTINCT salary
    FROM employees
    ORDER BY salary DESC
    LIMIT 1 OFFSET 1
) AS SecondHighestSalary;
```

### 6. Explanation
* `SELECT DISTINCT salary` deduplicates values (so 12000 tie is resolved).
* `ORDER BY salary DESC LIMIT 1 OFFSET 1` skips the first value (12000) and returns the second one (10000).
* Wrapping it in a main `SELECT (...)` returns `NULL` automatically if the subquery returns an empty set.

### 7. Interview Follow-up
* *How do you write this without using LIMIT/OFFSET for database engines that do not support them?*
  ```sql
  SELECT MAX(salary)
  FROM employees
  WHERE salary < (SELECT MAX(salary) FROM employees);
  ```

### 8. Common Wrong Answer
```sql
-- ❌ WRONG: Fails to return NULL if there are less than 2 distinct salaries!
SELECT DISTINCT salary FROM employees ORDER BY salary DESC LIMIT 1 OFFSET 1;
```

---

## 3. Top N Records

### 1. Problem Statement
Find the top 3 highest-earning employees in the company.

### 2. Sample Table (`employees`)
| name (VARCHAR) | salary (DECIMAL) |
|---|---|
| Alice | 12000 |
| Bob | 11000 |
| Charlie | 9000 |
| David | 8000 |

### 3. Expected Output
| name | salary |
|---|---|
| Alice | 12000 |
| Bob | 11000 |
| Charlie | 9000 |

### 4. Step-by-Step Thinking
1. We sort all employees by salary in descending order.
2. We restrict the output size using the `LIMIT` clause set to 3.

### 5. SQL Query
```sql
SELECT name, salary
FROM employees
ORDER BY salary DESC
LIMIT 3;
```

### 6. Explanation
* `ORDER BY salary DESC` puts the highest earners at the top.
* `LIMIT 3` discards everything below the first 3 rows.

### 7. Interview Follow-up
* *What is the standard SQL syntax for database engines that don't support LIMIT?*
  ```sql
  SELECT name, salary
  FROM employees
  ORDER BY salary DESC
  FETCH FIRST 3 ROWS ONLY;
  ```

### 8. Common Wrong Answer
```sql
-- ❌ WRONG: Omiting ORDER BY means you get 3 arbitrary records, not top earners.
SELECT name, salary FROM employees LIMIT 3;
```

---

## 4. Top N Per Group

### 1. Problem Statement
Find the top 2 highest-paid employees in *each* department.

### 2. Sample Table (`employees`)
| name | dept_id | salary |
|---|---|---|
| Alice | 1 | 10000 |
| Bob | 1 | 12000 |
| Charlie | 1 | 9000 |
| David | 2 | 8000 |
| Eva | 2 | 9500 |

### 3. Expected Output
| name | dept_id | salary |
|---|---|---|
| Bob | 1 | 12000 |
| Alice | 1 | 10000 |
| Eva | 2 | 9500 |
| David | 2 | 8000 |

### 4. Step-by-Step Thinking
1. To rank within groups, we must use a window function: `DENSE_RANK()` or `ROW_NUMBER()`.
2. We partition the rows by `dept_id` and order by `salary DESC`.
3. Wrap this calculation inside a CTE, and query where the rank is `<= 2`.

### 5. SQL Query
```sql
WITH RankedEmployees AS (
    SELECT name, dept_id, salary,
           DENSE_RANK() OVER (
               PARTITION BY dept_id 
               ORDER BY salary DESC
           ) as rnk
    FROM employees
)
SELECT name, dept_id, salary
FROM RankedEmployees
WHERE rnk <= 2;
```

### 6. Explanation
* `DENSE_RANK() OVER (PARTITION BY dept_id ORDER BY salary DESC)` ranks salaries inside each department.
* The CTE outputs this temporary table, and the outer query filters out any employees with a rank of `3` or lower.

### 7. Interview Follow-up
* *What happens if there is a salary tie? Which ranking function should you choose?*
  * If two employees tie for second place, `DENSE_RANK` returns both (returning 3 rows total). If the interviewer wants *exactly* 2 rows, swap it with `ROW_NUMBER()`.

### 8. Common Wrong Answer
```sql
-- ❌ WRONG: Cannot use window functions in WHERE clauses!
SELECT name, dept_id, salary
FROM employees
WHERE DENSE_RANK() OVER (PARTITION BY dept_id ORDER BY salary DESC) <= 2;
```

---

## 5. Employees Without Managers

### 1. Problem Statement
Find all employees who do not report to any manager.

### 2. Sample Table (`employees`)
| id | name | manager_id |
|---|---|---|
| 1 | Alice | NULL |
| 2 | Bob | 1 |
| 3 | Charlie | 1 |

### 3. Expected Output
| name |
|---|
| Alice |

### 4. Step-by-Step Thinking
1. A manager relation is stored in the `manager_id` column pointing to another employee ID.
2. If `manager_id` is missing/empty, it means they have no manager.
3. We filter using `IS NULL`.

### 5. SQL Query
```sql
SELECT name
FROM employees
WHERE manager_id IS NULL;
```

### 6. Explanation
* `WHERE manager_id IS NULL` filters the table rows to match only records where the manager relation is missing.

### 7. Interview Follow-up
* *How do you write a query to fetch the manager's name next to the employee's name for all employees?*
  ```sql
  SELECT e.name AS employee, COALESCE(m.name, 'CEO') AS manager
  FROM employees e
  LEFT JOIN employees m ON e.manager_id = m.id;
  ```

### 8. Common Wrong Answer
```sql
-- ❌ WRONG: NULL cannot be compared with '='!
SELECT name FROM employees WHERE manager_id = NULL;
```

---

## 6. Customers With No Orders

### 1. Problem Statement
Find all customers who have never placed an order.

### 2. Sample Tables
**`customers`**
| id | name |
|---|---|
| 1 | Alice |
| 2 | Bob |

**`orders`**
| order_id | customer_id |
|---|---|
| 101 | 1 |

### 3. Expected Output
| name |
|---|
| Bob |

### 4. Step-by-Step Thinking
1. Run a `LEFT JOIN` from `customers` to `orders` using `customer_id`.
2. For customers who haven't ordered anything, the matching `order_id` will be `NULL`.
3. Filter where `orders.order_id IS NULL`.

### 5. SQL Query
```sql
SELECT c.name
FROM customers c
LEFT JOIN orders o ON c.id = o.customer_id
WHERE o.order_id IS NULL;
```

### 6. Explanation
* The `LEFT JOIN` retains both customers. For Bob, since no order matches, `o.order_id` returns `NULL`.
* `WHERE o.order_id IS NULL` discards Alice and returns Bob.

### 7. Interview Follow-up
* *What is the alternative way to write this without using JOINS, and which is faster?*
  ```sql
  SELECT name FROM customers c
  WHERE NOT EXISTS (
      SELECT 1 FROM orders o WHERE o.customer_id = c.id
  );
  ```
  `NOT EXISTS` is usually faster or equal because the query optimizer stops scanning the orders table as soon as it finds a single match, whereas `LEFT JOIN` compiles the whole join dataset first.

### 8. Common Wrong Answer
```sql
-- ❌ WRONG: Fails if orders has NULL customer_ids!
SELECT name FROM customers WHERE id NOT IN (SELECT customer_id FROM orders);
```

---

## 7. Monthly Revenue

### 1. Problem Statement
Calculate the total revenue generated in each month.

### 2. Sample Table (`orders`)
| order_date (DATE) | amount (DECIMAL) |
|---|---|
| 2026-01-15 | 100 |
| 2026-01-20 | 150 |
| 2026-02-10 | 200 |

### 3. Expected Output
| month | total_revenue |
|---|---|
| 2026-01 | 250 |
| 2026-02 | 200 |

### 4. Step-by-Step Thinking
1. We need to extract the year and month from `order_date`.
2. Group by this month code.
3. Compute the `SUM(amount)` for each month.

### 5. SQL Query
```sql
SELECT TO_CHAR(order_date, 'YYYY-MM') AS month,
       SUM(amount) AS total_revenue
FROM orders
GROUP BY TO_CHAR(order_date, 'YYYY-MM')
ORDER BY month ASC;
```

### 6. Explanation
* `TO_CHAR(order_date, 'YYYY-MM')` converts a date like `2026-01-15` to string `"2026-01"`.
* `GROUP BY` bundles all transactions matching that month, and `SUM` computes the totals.

### 7. Interview Follow-up
* *What is the MySQL equivalent of `TO_CHAR`?*
  * In MySQL, you use `DATE_FORMAT(order_date, '%Y-%m')`.

### 8. Common Wrong Answer
```sql
-- ❌ WRONG: Grouping by date directly returns daily revenue, not monthly!
SELECT order_date, SUM(amount) FROM orders GROUP BY order_date;
```

---

## 8. Running Total

### 1. Problem Statement
Calculate the running total of transactions ordered by transaction date.

### 2. Sample Table (`transactions`)
| tx_date (DATE) | amount (DECIMAL) |
|---|---|
| 2026-01-01 | 100 |
| 2026-01-02 | 50 |
| 2026-01-03 | 200 |

### 3. Expected Output
| tx_date | amount | running_total |
|---|---|---|
| 2026-01-01 | 100 | 100 |
| 2026-01-02 | 50 | 150 |
| 2026-01-03 | 200 | 350 |

### 4. Step-by-Step Thinking
1. We need to aggregate the `amount` column cumulatively.
2. We use the window function `SUM(amount) OVER()`.
3. To make it a running calculation, sort the window partition: `ORDER BY tx_date ASC`.

### 5. SQL Query
```sql
SELECT tx_date, amount,
       SUM(amount) OVER (
           ORDER BY tx_date ASC
       ) AS running_total
FROM transactions;
```

### 6. Explanation
* `SUM(amount) OVER (ORDER BY tx_date ASC)` tells Python to calculate a running sum from the beginning of the table up to the current row's date.

### 7. Interview Follow-up
* *How do you restart the running total for different accounts?*
  * Add a `PARTITION BY` clause: `SUM(amount) OVER (PARTITION BY account_id ORDER BY tx_date)`.

### 8. Common Wrong Answer
```sql
-- ❌ WRONG: Omitting ORDER BY returns the total sum on every row, not a running total!
SELECT tx_date, SUM(amount) OVER () FROM transactions;
```

---

## 9. Moving Average

### 1. Problem Statement
Calculate a 3-day moving average of stock prices ordered by date.

### 2. Sample Table (`stock_prices`)
| price_date | price |
|---|---|
| 2026-01-01 | 10 |
| 2026-01-02 | 20 |
| 2026-01-03 | 30 |
| 2026-01-04 | 40 |

### 3. Expected Output
| price_date | price | moving_avg |
|---|---|---|
| 2026-01-01 | 10 | 10.0 |
| 2026-01-02 | 20 | 15.0 |
| 2026-01-03 | 30 | 20.0 |
| 2026-01-04 | 40 | 30.0 |

### 4. Step-by-Step Thinking
1. A moving average computes the mean of a sliding window.
2. For a 3-day average, we calculate: the current row and the two preceding rows.
3. We write: `AVG(price) OVER(...)`.
4. Define the window frame: `ROWS BETWEEN 2 PRECEDING AND CURRENT ROW`.

### 5. SQL Query
```sql
SELECT price_date, price,
       AVG(price) OVER (
           ORDER BY price_date ASC
           ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
       ) AS moving_avg
FROM stock_prices;
```

### 6. Explanation
* `ROWS BETWEEN 2 PRECEDING AND CURRENT ROW` restricts the average window calculation to the current day and the previous 2 days. On day 2, it averages 10 and 20 (15.0). On day 4, it averages 20, 30, and 40 (30.0).

### 7. Interview Follow-up
* *What is the difference between `ROWS` and `RANGE` inside window frames?*
  * `ROWS` counts the physical number of rows in the partition. `RANGE` checks the actual values of the sorted column (e.g. range of dates, handling missing dates in a sequence).

### 8. Common Wrong Answer
```sql
-- ❌ WRONG: Average of the whole table, not a moving window!
SELECT price_date, AVG(price) OVER (ORDER BY price_date) FROM stock_prices;
```

---

## 10. Retention / Cohort Style Query

### 1. Problem Statement
Calculate the percentage of users who signed up in Month 1 and returned to perform an action in Month 2.

### 2. Sample Tables
**`users`**
| user_id | signup_date |
|---|---|
| 1 | 2026-01-01 |
| 2 | 2026-01-15 |

**`events`**
| event_id | user_id | event_date |
|---|---|---|
| 100 | 1 | 2026-02-05 |

### 3. Expected Output
| cohort_month | total_users | retained_users | retention_rate |
|---|---|---|---|
| 2026-01 | 2 | 1 | 50.0 |

### 4. Step-by-Step Thinking
1. Define a cohort of users based on their signup month.
2. Join this cohort to the `events` table to see if they logged in during the subsequent calendar month (e.g. difference of 1 month).
3. Compute the ratio: `retained_users / total_users * 100`.

### 5. SQL Query
```sql
WITH Cohort AS (
    SELECT user_id, 
           TO_CHAR(signup_date, 'YYYY-MM') AS cohort_month
    FROM users
),
Retention AS (
    SELECT c.cohort_month,
           COUNT(DISTINCT c.user_id) AS total_users,
           COUNT(DISTINCT e.user_id) AS retained_users
    FROM Cohort c
    LEFT JOIN events e ON c.user_id = e.user_id 
        AND TO_CHAR(e.event_date, 'YYYY-MM') = TO_CHAR(c.signup_date + INTERVAL '1 month', 'YYYY-MM')
    GROUP BY c.cohort_month
)
SELECT cohort_month, total_users, retained_users,
       (retained_users::DECIMAL / total_users) * 100 AS retention_rate
FROM Retention;
```

### 6. Explanation
* The `Cohort` CTE finds the signup month for each user.
* The `Retention` CTE left-joins events matching the next month (`signup_date + 1 month`). If they didn't return, `e.user_id` is null.
* The outer query calculates the rate, converting integers to decimals to prevent integer division truncating to 0.

### 7. Interview Follow-up
* *How do you write this for daily retention (Day 1 retention)?*
  * Change the join condition to match events where `e.event_date = c.signup_date + INTERVAL '1 day'`.

### 8. Common Wrong Answer
```sql
-- ❌ WRONG: Simple division of two integers in PostgreSQL yields 0!
SELECT (retained_users / total_users) * 100 FROM Retention;
```

---

## 11. Consecutive Login Days

### 1. Problem Statement
Identify users who have logged in on 3 or more consecutive days.

### 2. Sample Table (`logins`)
| user_id | login_date |
|---|---|
| 1 | 2026-01-01 |
| 1 | 2026-01-02 |
| 1 | 2026-01-03 |
| 2 | 2026-01-01 |

### 3. Expected Output
| user_id |
|---|
| 1 |

### 4. Step-by-Step Thinking
1. We group consecutive dates together using the **Gaps and Islands** pattern.
2. If we assign a row number to each user's login dates: `ROW_NUMBER() OVER(PARTITION BY user_id ORDER BY login_date)`.
3. If dates are consecutive, the difference between the `login_date` and the `row_number` will yield the same base date.
4. We group by `user_id` and the difference date, and filter groups with `COUNT(*) >= 3`.

### 5. SQL Query
```sql
WITH RankedLogins AS (
    SELECT DISTINCT user_id, login_date, -- DISTINCT prevents duplicate logins on same day
           ROW_NUMBER() OVER (
               PARTITION BY user_id 
               ORDER BY login_date ASC
           ) as rn
    FROM logins
),
Groups AS (
    SELECT user_id,
           login_date - CAST(rn || ' days' AS INTERVAL) as base_group
    FROM RankedLogins
)
SELECT user_id
FROM Groups
GROUP BY user_id, base_group
HAVING COUNT(*) >= 3;
```

### 6. Explanation
* If user logs on `Jan-01 (rn=1)`, `Jan-02 (rn=2)`, `Jan-03 (rn=3)`:
  * `Jan-01 - 1 day` = `Dec-31`
  * `Jan-02 - 2 days` = `Dec-31`
  * `Jan-03 - 3 days` = `Dec-31`
* Because the base date is identical (`Dec-31`), they group together. Counting this group returns `3`.

### 7. Interview Follow-up
* *What is the alternative way using LEAD/LAG if you only check for exactly 3 days?*
  ```sql
  SELECT DISTINCT user_id
  FROM (
      SELECT user_id, login_date,
             LEAD(login_date, 1) OVER (PARTITION BY user_id ORDER BY login_date) as date_1,
             LEAD(login_date, 2) OVER (PARTITION BY user_id ORDER BY login_date) as date_2
      FROM logins
  ) t
  WHERE date_1 = login_date + 1 AND date_2 = login_date + 2;
  ```

### 8. Common Wrong Answer
```sql
-- ❌ WRONG: Simple count checks total logins, not consecutive login days!
SELECT user_id FROM logins GROUP BY user_id HAVING COUNT(login_date) >= 3;
```

---

## 12. Difference Between WHERE and HAVING

### 1. Problem Statement
Write a query to demonstrate the difference: find departments with more than 5 high-earning employees (earning > 80,000).

### 2. Sample Table (`employees`)
| name | dept_id | salary |
|---|---|---|
| Alice | 1 | 90000 |
| Bob | 1 | 85000 |

### 3. Expected Output
| dept_id | employee_count |
|---|---|
| 1 | 2 |

### 4. Step-by-Step Thinking
1. Filter individual rows *before* aggregation to keep only employees earning > 80,000 using `WHERE`.
2. Group by `dept_id`.
3. Filter the resulting department groups to keep only those with a count > 5 using `HAVING`.

### 5. SQL Query
```sql
SELECT dept_id, COUNT(*) AS employee_count
FROM employees
WHERE salary > 80000 -- 1. Row-level filter
GROUP BY dept_id
HAVING COUNT(*) > 5; -- 2. Group-level filter
```

### 6. Explanation
* `WHERE salary > 80000` runs first. It discards low-income rows.
* `GROUP BY` partitions the remaining rows.
* `HAVING COUNT(*) > 5` runs last, filtering out departments that don't have enough qualifying employees.

### 7. Interview Follow-up
* *Can we write `HAVING salary > 80000` instead of WHERE?*
  * No, this will crash because `salary` is not aggregated or grouped, and the database doesn't know which individual salary to check in the department bucket.

### 8. Common Wrong Answer
```sql
-- ❌ WRONG: Crashes because WHERE cannot hold aggregate functions!
SELECT dept_id FROM employees WHERE COUNT(*) > 5 GROUP BY dept_id;
```

---

## 13. Find Users Who Purchased Multiple Products

### 1. Problem Statement
Find all users who have bought more than one unique product.

### 2. Sample Table (`orders`)
| user_id | product_id |
|---|---|
| 1 | 101 |
| 1 | 101 |
| 2 | 101 |
| 2 | 102 |

### 3. Expected Output
| user_id |
|---|
| 2 |

### 4. Step-by-Step Thinking
1. Group the orders by `user_id`.
2. Count the number of unique products they bought using `COUNT(DISTINCT product_id)`.
3. Filter groups where this count is greater than 1 using `HAVING`.

### 5. SQL Query
```sql
SELECT user_id
FROM orders
GROUP BY user_id
HAVING COUNT(DISTINCT product_id) > 1;
```

### 6. Explanation
* User 1 purchased product 101 twice. `COUNT(product_id)` is 2, but `COUNT(DISTINCT product_id)` is 1. They are filtered out.
* User 2 purchased 101 and 102. Their distinct count is 2, so they are returned.

### 7. Interview Follow-up
* *How do you modify this to find users who purchased *all* products from a specific catalog table?*
  ```sql
  SELECT user_id FROM orders
  GROUP BY user_id
  HAVING COUNT(DISTINCT product_id) = (SELECT COUNT(*) FROM catalog);
  ```

### 8. Common Wrong Answer
```sql
-- ❌ WRONG: Fails to detect duplicates of the same product!
SELECT user_id FROM orders GROUP BY user_id HAVING COUNT(product_id) > 1;
```

---

## 14. Department-Wise Highest Salary

### 1. Problem Statement
Find the name and salary of the employee earning the highest salary in each department.

### 2. Sample Table (`employees`)
| name | dept_id | salary |
|---|---|---|
| Alice | 1 | 10000 |
| Bob | 1 | 12000 |
| Charlie | 2 | 9000 |

### 3. Expected Output
| name | dept_id | salary |
|---|---|---|
| Bob | 1 | 12000 |
| Charlie | 2 | 9000 |

### 4. Step-by-Step Thinking
1. Rank salaries inside departments: `RANK() OVER (PARTITION BY dept_id ORDER BY salary DESC)`.
2. Wrap this query in a CTE.
3. Select rows from the CTE where the rank is exactly `1`.

### 5. SQL Query
```sql
WITH RankedSales AS (
    SELECT name, dept_id, salary,
           RANK() OVER (
               PARTITION BY dept_id 
               ORDER BY salary DESC
           ) as rnk
    FROM employees
)
SELECT name, dept_id, salary
FROM RankedSales
WHERE rnk = 1;
```

### 6. Explanation
* The window function ranks employees inside each department. Bob gets rank 1 in department 1, and Charlie gets rank 1 in department 2. The outer query filters for rank 1.

### 7. Interview Follow-up
* *How do you write this using a correlated subquery instead of a window function?*
  ```sql
  SELECT name, dept_id, salary
  FROM employees e1
  WHERE salary = (
      SELECT MAX(salary)
      FROM employees e2
      WHERE e2.dept_id = e1.dept_id
  );
  ```

### 8. Common Wrong Answer
```sql
-- ❌ WRONG: SQL will crash because 'name' is not grouped!
SELECT name, dept_id, MAX(salary) FROM employees GROUP BY dept_id;
```

---

## 15. First and Last Event Per User

### 1. Problem Statement
Find the timestamp of the first and the last event logged for each user.

### 2. Sample Table (`events`)
| user_id | event_time (TIMESTAMP) | event_name |
|---|---|---|
| 1 | 2026-01-01 10:00:00 | login |
| 1 | 2026-01-01 12:00:00 | logout |
| 2 | 2026-01-01 11:00:00 | login |

### 3. Expected Output
| user_id | first_event | last_event |
|---|---|---|
| 1 | 2026-01-01 10:00:00 | 2026-01-01 12:00:00 |
| 2 | 2026-01-01 11:00:00 | 2026-01-01 11:00:00 |

### 4. Step-by-Step Thinking
1. Group the logs by `user_id`.
2. Find the minimum timestamp using `MIN(event_time)` (first event).
3. Find the maximum timestamp using `MAX(event_time)` (last event).

### 5. SQL Query
```sql
SELECT user_id,
       MIN(event_time) AS first_event,
       MAX(event_time) AS last_event
FROM events
GROUP BY user_id;
```

### 6. Explanation
* `GROUP BY user_id` groups events. `MIN` grabs the earliest timestamp in the group, and `MAX` grabs the latest.

### 7. Interview Follow-up
* *What if you need to fetch the actual event_name of the first and last event, not just the timestamps?*
  * Use `ROW_NUMBER` windows: one ascending, one descending, and join them or filter where rank is 1.

### 8. Common Wrong Answer
```sql
-- ❌ WRONG: Trying to write complex window functions when simple aggregates work!
SELECT user_id, FIRST_VALUE(event_time) OVER() FROM events;
```

---

## 16. Ranking Records Within Groups

### 1. Problem Statement
Rank products by their price inside each product category. If prices match, they must get the same rank and the rank numbers must not have gaps.

### 2. Sample Table (`products`)
| name | category | price |
|---|---|---|
| Phone A | Tech | 500 |
| Phone B | Tech | 500 |
| Phone C | Tech | 400 |

### 3. Expected Output
| name | category | price | price_rank |
|---|---|---|---|
| Phone A | Tech | 500 | 1 |
| Phone B | Tech | 500 | 1 |
| Phone C | Tech | 400 | 2 |

### 4. Step-by-Step Thinking
1. We need to rank within groups: partition by `category` and order by `price DESC`.
2. Because matching prices must get the same rank with **no gaps** (e.g. 1, 1, 2), we choose `DENSE_RANK()`.

### 5. SQL Query
```sql
SELECT name, category, price,
       DENSE_RANK() OVER (
           PARTITION BY category
           ORDER BY price DESC
       ) AS price_rank
FROM products;
```

### 6. Explanation
* `DENSE_RANK()` assigns rank 1 to both 500-price phones. It assigns rank 2 (no gap) to the 400-price phone.

### 7. Interview Follow-up
* *What is the rank of Phone C if we change this to `RANK()`?*
  * The rank of Phone C would become `3`, because `RANK()` skips rank 2 due to the duplicate rank 1 tie.

### 8. Common Wrong Answer
```sql
-- ❌ WRONG: ROW_NUMBER will assign 1 and 2 to the tied items, violating ties rule!
SELECT name, ROW_NUMBER() OVER(ORDER BY price DESC) FROM products;
```

---

## 17. Join Multiple Tables

### 1. Problem Statement
Retrieve the customer name, their order ID, and the total price of products ordered.

### 2. Sample Tables
**`customers`** (id, name)  
**`orders`** (id, customer_id)  
**`order_items`** (order_id, product_id, price)  

### 3. Expected Output
| customer_name | order_id | total_price |
|---|---|---|
| Alice | 100 | 1500 |

### 4. Step-by-Step Thinking
1. We must combine three tables.
2. Join `customers` to `orders` using `customer_id`.
3. Join `orders` to `order_items` using `order_id`.
4. Group by customer name and order ID, and sum the price column.

### 5. SQL Query
```sql
SELECT c.name AS customer_name,
       o.id AS order_id,
       SUM(oi.price) AS total_price
FROM customers c
INNER JOIN orders o ON c.id = o.customer_id
INNER JOIN order_items oi ON o.id = oi.order_id
GROUP BY c.name, o.id;
```

### 6. Explanation
* Multiple tables are chained sequentially. First, the engine joins `c` and `o`, then joins the result to `oi`.
* `SUM(oi.price)` aggregates item totals per group.

### 7. Interview Follow-up
* *How would you change this to include customers who haven't ordered anything?*
  * Swap both `INNER JOIN`s with `LEFT JOIN`s.

### 8. Common Wrong Answer
```sql
-- ❌ WRONG: Joining order_items directly to customers without the middle order bridge table!
SELECT * FROM customers c INNER JOIN order_items oi ON c.id = oi.order_id;
```

---

## 18. Handling NULLs in Joins

### 1. Problem Statement
Join employees with their departments, but if an employee has a `NULL` department ID, display their department as `'Unassigned'`.

### 2. Sample Tables
**`employees`** (name, dept_id)  
**`departments`** (id, name)  

### 3. Expected Output
| employee_name | department_name |
|---|---|
| Alice | Tech |
| Bob | Unassigned |

### 4. Step-by-Step Thinking
1. If we use `INNER JOIN`, employees with `NULL` department IDs are discarded. We must use `LEFT JOIN` to keep them.
2. If `departments.name` is null, we replace it with `'Unassigned'` using `COALESCE()`.

### 5. SQL Query
```sql
SELECT e.name AS employee_name,
       COALESCE(d.name, 'Unassigned') AS department_name
FROM employees e
LEFT JOIN departments d ON e.dept_id = d.id;
```

### 6. Explanation
* `LEFT JOIN` retains Bob. Since his `dept_id` is null, `d.name` returns `NULL`.
* `COALESCE(d.name, 'Unassigned')` intercepts the `NULL` and replaces it with the default string.

### 7. Interview Follow-up
* *Can we join tables on keys that are NULL? (No, writing `ON a.key = b.key` fails if keys are null because NULL = NULL is UNKNOWN. You must write `ON a.key = b.key OR (a.key IS NULL AND b.key IS NULL)`).*

### 8. Common Wrong Answer
```sql
-- ❌ WRONG: Discards employees with NULL department IDs!
SELECT e.name, d.name FROM employees e INNER JOIN departments d ON e.dept_id = d.id;
```

---

## 19. Percentage Calculation

### 1. Problem Statement
Calculate the percentage of total company sales contributed by each product.

### 2. Sample Table (`sales`)
| product_id | revenue |
|---|---|
| 101 | 300 |
| 102 | 700 |

### 3. Expected Output
| product_id | revenue | percentage |
|---|---|---|
| 101 | 300 | 30.0 |
| 102 | 700 | 70.0 |

### 4. Step-by-Step Thinking
1. To calculate the percentage, we divide each row's `revenue` by the total revenue of the entire table.
2. We find the total revenue using a window function with an empty over clause: `SUM(revenue) OVER()`.
3. Division must use float/decimal types to prevent integer truncation.

### 5. SQL Query
```sql
SELECT product_id, revenue,
       (revenue * 100.0) / SUM(revenue) OVER () AS percentage
FROM sales;
```

### 6. Explanation
* `SUM(revenue) OVER ()` sums the entire table (1000).
* For product 101, `(300 * 100.0) / 1000` evaluates to `30.0`.

### 7. Interview Follow-up
* *How do you round the percentage to two decimal places?*
  * Wrap it in `ROUND(val, 2)`.

### 8. Common Wrong Answer
```sql
-- ❌ WRONG: Will fail because of integer division or missing OVER block!
SELECT product_id, (revenue / SUM(revenue)) * 100 FROM sales GROUP BY product_id;
```

---

## 20. Conversion Rate Calculation

### 1. Problem Statement
Calculate the conversion rate of visits to purchases (percentage of page views that resulted in a purchase).

### 2. Sample Table (`events`)
| user_id | event_type |
|---|---|
| 1 | view |
| 1 | purchase |
| 2 | view |

### 3. Expected Output
| conversion_rate |
|---|
| 50.0 |

### 4. Step-by-Step Thinking
1. Count the number of unique purchase sessions.
2. Count the total number of unique view sessions.
3. Compute the ratio using conditional aggregation: `COUNT(case when purchase) / COUNT(case when view) * 100`.

### 5. SQL Query
```sql
SELECT (
    COUNT(CASE WHEN event_type = 'purchase' THEN 1 END) * 100.0
) / NULLIF(COUNT(CASE WHEN event_type = 'view' THEN 1 END), 0) AS conversion_rate
FROM events;
```

### 6. Explanation
* `COUNT(CASE WHEN event_type = 'purchase' THEN 1 END)` counts how many times the status is purchase.
* `NULLIF(..., 0)` turns the divisor to `NULL` if view count is `0`, preventing division-by-zero errors.

### 7. Interview Follow-up
* *How does this protect against division-by-zero errors?*
  * By using `NULLIF(count, 0)`. In SQL, dividing any number by `NULL` returns `NULL` instead of crashing.

### 8. Common Wrong Answer
```sql
-- ❌ WRONG: Will throw Division by Zero error if there are no view events!
SELECT (COUNT(purchase) / COUNT(view)) * 100 FROM events;
```

---

*← [Previous: Phase 1 Chapter 7 (Database Internals)](../Phase1/07_Database_Internals.md) | [Home (README) →](../README.md)*
