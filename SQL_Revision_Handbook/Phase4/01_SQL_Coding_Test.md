# 📊 Phase 4 — Ultimate Progressive SQL Coding Practice

This guide is structured topic-by-topic. If you are weak in any specific area (like Joins or Window Functions), start from the first question of that section and work your way down. The difficulty increases gradually so you can build muscle memory.

Each question features the **Scenario**, **Schema/Data**, **SQL Query**, and a **1-2 line explanation** of the query logic immediately below it for instant verification.

---

## 📋 Table of Contents
1. [Basic Selection & Filtering (Q1 - Q4)](#1-basic-selection--filtering-q1---q4)
2. [Aggregations & Grouping (Q5 - Q8)](#2-aggregations--grouping-q5---q8)
3. [Joins in Depth (Q9 - Q14)](#3-joins-in-depth-q9---q14)
4. [Subqueries & CTEs (Q15 - Q18)](#4-subqueries--ctes-q15---q18)
5. [Window Functions (Q19 - Q23)](#5-window-functions-q19---q23)
6. [Date & Time Operations (Q24 - Q26)](#6-date--time-operations-q24---q26)
7. [Advanced Interview Challenges (Q27 - Q30)](#7-advanced-interview-challenges-q27---q30)

---

## 1. Basic Selection & Filtering (Q1 - Q4)

### Q1: Simple Filtering (Easiest)
**Scenario:** Find all employees working in Department 10 who make more than $60,000.
* **Table:** `employees` (`employee_id` INT, `name` VARCHAR, `department_id` INT, `salary` DECIMAL)
* **SQL Query:**
```sql
SELECT employee_id, name, salary
FROM employees
WHERE department_id = 10 
  AND salary > 60000;
```
* **Explanation:** The `WHERE` clause filters rows down so that only records satisfying both conditions (`department_id` match and `salary` threshold) are returned.

---

### Q2: Pattern Matching & Range Filters (Easy)
**Scenario:** Find all products whose names start with the letter 'S' and whose price is between $10 and $50 (inclusive).
* **Table:** `products` (`product_id` INT, `product_name` VARCHAR, `price` DECIMAL)
* **SQL Query:**
```sql
SELECT product_id, product_name, price
FROM products
WHERE product_name LIKE 'S%'
  AND price BETWEEN 10.00 AND 50.00;
```
* **Explanation:** We use `LIKE 'S%'` to match names starting with 'S' (where `%` matches any trailing characters) and `BETWEEN` to check for prices in the inclusive range.

---

### Q3: Handling NULL Values (Easy)
**Scenario:** List all customers who do not have a phone number registered in the system.
* **Table:** `customers` (`customer_id` INT, `customer_name` VARCHAR, `phone` VARCHAR)
* **SQL Query:**
```sql
SELECT customer_id, customer_name
FROM customers
WHERE phone IS NULL;
```
* **Explanation:** NULL represents missing data, so we must use the `IS NULL` comparison operator. Writing `phone = NULL` will not return any rows because comparing values to NULL is undefined.

---

### Q4: Complex Boolean logic (Easy-Medium)
**Scenario:** Find all orders that were placed in 2024 and are either high priority (`priority = 'High'`) OR have an order value strictly greater than $500.
* **Table:** `orders` (`order_id` INT, `order_date` DATE, `priority` VARCHAR, `total_value` DECIMAL)
* **SQL Query:**
```sql
SELECT order_id, order_date, priority, total_value
FROM orders
WHERE order_date BETWEEN '2024-01-01' AND '2024-12-31'
  AND (priority = 'High' OR total_value > 500.00);
```
* **Explanation:** Parentheses are required around the `OR` conditions to ensure they are evaluated together before checking the date filter constraint.

---

## 2. Aggregations & Grouping (Q5 - Q8)

### Q5: Basic Counting & Grouping (Easiest)
**Scenario:** Find the total number of items sold and the total revenue for each product ID.
* **Table:** `sales` (`sale_id` INT, `product_id` INT, `quantity` INT, `revenue` DECIMAL)
* **SQL Query:**
```sql
SELECT product_id, SUM(quantity) AS total_sold, SUM(revenue) AS total_revenue
FROM sales
GROUP BY product_id;
```
* **Explanation:** `GROUP BY product_id` groups the rows by each unique product, and `SUM` aggregates the quantity and revenue columns for each group.

---

### Q6: Filtering Groups with HAVING (Easy)
**Scenario:** Find all product IDs that have earned more than $5,000 in total revenue.
* **Table:** `sales` (`sale_id` INT, `product_id` INT, `quantity` INT, `revenue` DECIMAL)
* **SQL Query:**
```sql
SELECT product_id, SUM(revenue) AS total_revenue
FROM sales
GROUP BY product_id
HAVING SUM(revenue) > 5000.00;
```
* **Explanation:** The `HAVING` clause is used to filter groups *after* the aggregation (`SUM`) takes place; we cannot use `WHERE` to filter aggregated column values.

---

### Q7: Multi-Column Grouping (Medium)
**Scenario:** Calculate the total amount spent by each customer in each store location.
* **Table:** `transactions` (`transaction_id` INT, `customer_id` INT, `store_location` VARCHAR, `amount` DECIMAL)
* **SQL Query:**
```sql
SELECT customer_id, store_location, SUM(amount) AS total_spent
FROM transactions
GROUP BY customer_id, store_location;
```
* **Explanation:** Grouping by multiple columns generates a separate aggregated row for every unique combination of `customer_id` and `store_location`.

---

### Q8: Conditional Aggregation (Medium-Hard)
**Scenario:** Count the number of high-value orders (value > $100) and low-value orders (value <= $100) placed by each user.
* **Table:** `orders` (`order_id` INT, `user_id` INT, `total_value` DECIMAL)
* **SQL Query:**
```sql
SELECT 
  user_id,
  COUNT(CASE WHEN total_value > 100.00 THEN 1 END) AS high_value_count,
  COUNT(CASE WHEN total_value <= 100.00 THEN 1 END) AS low_value_count
FROM orders
GROUP BY user_id;
```
* **Explanation:** The `CASE` statement inside the `COUNT` function evaluates to `1` (or any non-null value) when the condition matches, and `NULL` when it doesn't, causing `COUNT` to only count the matching cases.

---

## 3. Joins in Depth (Q9 - Q14)

### Q9: Basic INNER JOIN (Easiest)
**Scenario:** Get a list of all order IDs, order dates, and the name of the customer who placed each order.
* **Table:** `customers` (`customer_id` INT, `customer_name` VARCHAR)
* **Table:** `orders` (`order_id` INT, `customer_id` INT, `order_date` DATE)
* **SQL Query:**
```sql
SELECT o.order_id, o.order_date, c.customer_name
FROM orders o
INNER JOIN customers c ON o.customer_id = c.customer_id;
```
* **Explanation:** `INNER JOIN` matches and returns rows only when the join key `customer_id` exists in both tables.

---

### Q10: LEFT JOIN for Missing Values (Easy)
**Scenario:** Find all customers who have never placed an order.
* **Table:** `customers` (`customer_id` INT, `customer_name` VARCHAR)
* **Table:** `orders` (`order_id` INT, `customer_id` INT, `order_date` DATE)
* **SQL Query:**
```sql
SELECT c.customer_id, c.customer_name
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
WHERE o.order_id IS NULL;
```
* **Explanation:** A `LEFT JOIN` returns all customers. Any customer without an order will have NULL values in the `orders` columns, which we filter using `WHERE o.order_id IS NULL`.

---

### Q11: Joining Multiple Tables (Easy-Medium)
**Scenario:** List all students, the courses they are enrolled in, and the names of the teachers leading those courses.
* **Table:** `students` (`student_id` INT, `student_name` VARCHAR)
* **Table:** `enrollments` (`student_id` INT, `course_id` INT)
* **Table:** `courses` (`course_id` INT, `course_name` VARCHAR, `teacher_name` VARCHAR)
* **SQL Query:**
```sql
SELECT s.student_name, c.course_name, c.teacher_name
FROM students s
INNER JOIN enrollments e ON s.student_id = e.student_id
INNER JOIN courses c ON e.course_id = c.course_id;
```
* **Explanation:** We chain multiple `INNER JOIN` clauses sequentially, using intermediate tables (`enrollments`) to establish the connection path.

---

### Q12: SELF JOIN (Medium)
**Scenario:** Find all employees and print their names alongside their manager's name. (Employees with no manager should still appear in the output).
* **Table:** `employees` (`employee_id` INT, `name` VARCHAR, `manager_id` INT)
* **SQL Query:**
```sql
SELECT e.name AS employee_name, m.name AS manager_name
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.employee_id;
```
* **Explanation:** We join the `employees` table to itself using two different aliases (`e` for employees and `m` for managers), matching the employee's `manager_id` to the manager's `employee_id`.

---

### Q13: FULL OUTER JOIN (Medium-Hard)
**Scenario:** You have lists of active mobile users and active web users. Generate a consolidated list of all unique users, indicating if they are active on mobile (`is_mobile_active`) and/or web (`is_web_active`).
* **Table:** `mobile_users` (`user_id` INT, `last_active_mobile` TIMESTAMP)
* **Table:** `web_users` (`user_id` INT, `last_active_web` TIMESTAMP)
* **SQL Query:**
```sql
SELECT 
  COALESCE(m.user_id, w.user_id) AS user_id,
  CASE WHEN m.user_id IS NOT NULL THEN TRUE ELSE FALSE END AS is_mobile_active,
  CASE WHEN w.user_id IS NOT NULL THEN TRUE ELSE FALSE END AS is_web_active
FROM mobile_users m
FULL OUTER JOIN web_users w ON m.user_id = w.user_id;
```
* **Explanation:** `FULL OUTER JOIN` retains records from both tables even if they don't match, and `COALESCE` selects the first non-null `user_id` available.

---

### Q14: CROSS JOIN (Medium-Hard)
**Scenario:** The retail team wants to print labels for every possible product color and size combination. Find all possible pairings of colors and sizes.
* **Table:** `sizes` (`size_code` VARCHAR)
* **Table:** `colors` (`color_name` VARCHAR)
* **SQL Query:**
```sql
SELECT s.size_code, c.color_name
FROM sizes s
CROSS JOIN colors c;
```
* **Explanation:** `CROSS JOIN` produces a Cartesian product, pairing every single row in the first table with every single row in the second table.

---

## 4. Subqueries & CTEs (Q15 - Q18)

### Q15: Subquery in WHERE Clause (Easy)
**Scenario:** Find all employees who earn strictly more than the company's average salary.
* **Table:** `employees` (`employee_id` INT, `name` VARCHAR, `salary` DECIMAL)
* **SQL Query:**
```sql
SELECT employee_id, name, salary
FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);
```
* **Explanation:** The subquery `(SELECT AVG(salary)...)` runs first to compute a single scalar value, which the outer query then uses to filter employee records.

---

### Q16: Correlated Subquery (Medium)
**Scenario:** Find employees who earn more than the average salary of their specific department.
* **Table:** `employees` (`employee_id` INT, `name` VARCHAR, `salary` DECIMAL, `department_id` INT)
* **SQL Query:**
```sql
SELECT e.employee_id, e.name, e.salary, e.department_id
FROM employees e
WHERE e.salary > (
  SELECT AVG(sub.salary) 
  FROM employees sub 
  WHERE sub.department_id = e.department_id
);
```
* **Explanation:** This is a correlated subquery because the inner query references the outer query's row alias (`e.department_id`), executing the subquery calculation for every evaluated department.

---

### Q17: EXISTS vs IN Check (Medium)
**Scenario:** Find all customers who have placed at least one order worth more than $1,000.
* **Table:** `customers` (`customer_id` INT, `customer_name` VARCHAR)
* **Table:** `orders` (`order_id` INT, `customer_id` INT, `total_value` DECIMAL)
* **SQL Query:**
```sql
SELECT c.customer_id, c.customer_name
FROM customers c
WHERE EXISTS (
  SELECT 1 
  FROM orders o 
  WHERE o.customer_id = c.customer_id 
    AND o.total_value > 1000.00
);
```
* **Explanation:** `EXISTS` is highly efficient because the database stops scanning the `orders` table for that customer the instant it finds a single match.

---

### Q18: CTE for Modular Queries (Medium-Hard)
**Scenario:** Identify departments whose total salary expense is higher than the average departmental salary expense across the company.
* **Table:** `employees` (`employee_id` INT, `department_id` INT, `salary` DECIMAL)
* **SQL Query:**
```sql
WITH DeptExpenses AS (
  SELECT department_id, SUM(salary) AS total_expense
  FROM employees
  GROUP BY department_id
)
SELECT department_id, total_expense
FROM DeptExpenses
WHERE total_expense > (SELECT AVG(total_expense) FROM DeptExpenses);
```
* **Explanation:** The CTE `DeptExpenses` calculates the salary sum per department. We then reference this temporary result set twice in the main query—once in the `FROM` and once in the subquery.

---

## 5. Window Functions (Q19 - Q23)

### Q19: Ranking Rows with ROW_NUMBER & DENSE_RANK (Easy-Medium)
**Scenario:** Find the top-priced product in each product category. (If there are price ties, return them all).
* **Table:** `products` (`product_id` INT, `product_name` VARCHAR, `category_id` INT, `price` DECIMAL)
* **SQL Query:**
```sql
WITH RankedProducts AS (
  SELECT 
    product_name, category_id, price,
    DENSE_RANK() OVER (PARTITION BY category_id ORDER BY price DESC) AS rnk
  FROM products
)
SELECT product_name, category_id, price
FROM RankedProducts
WHERE rnk = 1;
```
* **Explanation:** `DENSE_RANK()` ranks products by price descending per category. If there is a tie for the highest price, both get rank `1` and are returned.

---

### Q20: Cumulative Running Total (Medium)
**Scenario:** Compute a running total of deposits for each bank account, ordered chronologically.
* **Table:** `bank_transactions` (`transaction_id` INT, `account_id` INT, `amount` DECIMAL, `transaction_time` TIMESTAMP)
* **SQL Query:**
```sql
SELECT 
  account_id, transaction_time, amount,
  SUM(amount) OVER (
    PARTITION BY account_id 
    ORDER BY transaction_time
    ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
  ) AS running_balance
FROM bank_transactions
ORDER BY account_id, transaction_time;
```
* **Explanation:** `SUM(amount) OVER (...)` sums values from the start of the account's partition up to the current row index, creating a running ledger.

---

### Q21: Accessing Lead/Lag Rows (Medium-Hard)
**Scenario:** Write a query that shows the daily close price of a stock along with yesterday's close price and the daily price change.
* **Table:** `stock_prices` (`trade_date` DATE, `close_price` DECIMAL)
* **SQL Query:**
```sql
SELECT 
  trade_date,
  close_price,
  LAG(close_price, 1) OVER (ORDER BY trade_date) AS yesterday_close,
  close_price - LAG(close_price, 1) OVER (ORDER BY trade_date) AS daily_change
FROM stock_prices;
```
* **Explanation:** `LAG(close_price, 1)` pulls the value of `close_price` from the row immediately preceding the current row based on chronological `trade_date` sorting.

---

### Q22: Moving Average Calculation (Hard)
**Scenario:** Calculate a 3-day rolling average of sales revenue for the company (current day + 2 preceding days).
* **Table:** `daily_revenue` (`sale_date` DATE, `revenue` DECIMAL)
* **SQL Query:**
```sql
SELECT 
  sale_date,
  revenue,
  AVG(revenue) OVER (
    ORDER BY sale_date
    ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
  ) AS rolling_3_day_avg
FROM daily_revenue;
```
* **Explanation:** The `ROWS BETWEEN` frame limits the input values to the average function to the last 2 preceding rows and the current active row.

---

### Q23: Advanced Window Partitioning (Hard)
**Scenario:** For each customer, find their second-ever order details. If they have only placed one order, ignore them.
* **Table:** `orders` (`order_id` INT, `customer_id` INT, `order_date` DATE, `amount` DECIMAL)
* **SQL Query:**
```sql
WITH SequencedOrders AS (
  SELECT 
    order_id, customer_id, order_date, amount,
    ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY order_date ASC) as order_seq
  FROM orders
)
SELECT order_id, customer_id, order_date, amount
FROM SequencedOrders
WHERE order_seq = 2;
```
* **Explanation:** `ROW_NUMBER()` orders each customer's purchases from oldest to newest. We filter for rank `2` in the outer query to extract the second order.

---

## 6. Date & Time Operations (Q24 - Q26)

### Q24: Extracting Date Parts & Grouping (Easy-Medium)
**Scenario:** Calculate the total monthly sales revenue for the year 2024. Format the output month as 'YYYY-MM'.
* **Table:** `sales` (`sale_id` INT, `sale_date` DATE, `amount` DECIMAL)
* **SQL Query:**
```sql
SELECT 
  TO_CHAR(sale_date, 'YYYY-MM') AS sale_month,
  SUM(amount) AS monthly_revenue
FROM sales
WHERE sale_date BETWEEN '2024-01-01' AND '2024-12-31'
GROUP BY TO_CHAR(sale_date, 'YYYY-MM')
ORDER BY sale_month;
```
* **Explanation:** `TO_CHAR` (PostgreSQL) format parses the date into a monthly string. We group by this formatted text string to calculate aggregated monthly totals.

---

### Q25: Date Arithmetic & Intervals (Medium)
**Scenario:** Find all user accounts that were upgraded to 'VIP' status and then downgraded or canceled within 30 days of the upgrade.
* **Table:** `membership_log` (`user_id` INT, `action` VARCHAR, `action_date` DATE)
* **SQL Query:**
```sql
SELECT upgrade.user_id, upgrade.action_date AS upgraded_at, downgrade.action_date AS downgraded_at
FROM membership_log upgrade
INNER JOIN membership_log downgrade 
   ON upgrade.user_id = downgrade.user_id
  AND upgrade.action = 'Upgrade'
  AND downgrade.action = 'Downgrade'
WHERE downgrade.action_date >= upgrade.action_date
  AND downgrade.action_date <= upgrade.action_date + INTERVAL '30 days';
```
* **Explanation:** We self-join the log to line up upgrade events with downgrade events and use date addition `+ INTERVAL '30 days'` to evaluate the timeframe.

---

### Q26: Dynamic Rolling Windows (Hard)
**Scenario:** Find the count of active users who logged in within the 7 days prior to any given target date (e.g. '2024-05-15').
* **Table:** `user_logins` (`user_id` INT, `login_date` DATE)
* **SQL Query:**
```sql
SELECT COUNT(DISTINCT user_id) AS active_users_count
FROM user_logins
WHERE login_date BETWEEN '2024-05-15'::DATE - INTERVAL '7 days' AND '2024-05-15'::DATE;
```
* **Explanation:** We cast the input parameter string into a date and subtract an interval of 7 days to dynamically bound the evaluation window.

---

## 7. Advanced Interview Challenges (Q27 - Q30)

### Q27: Consecutive Days Active (Gaps & Islands) (Hard)
**Scenario:** Find all users who logged in for 3 or more consecutive days.
* **Table:** `user_logins` (`user_id` INT, `login_date` DATE)
* **SQL Query:**
```sql
WITH Deduped AS (
  SELECT DISTINCT user_id, login_date FROM user_logins
),
Groups AS (
  SELECT 
    user_id, login_date,
    login_date - CAST(ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY login_date) AS INT) AS group_key
  FROM Deduped
)
SELECT user_id, MIN(login_date) AS start_date, MAX(login_date) AS end_date, COUNT(*) AS consecutive_days
FROM Groups
GROUP BY user_id, group_key
HAVING COUNT(*) >= 3;
```
* **Explanation:** Subtracting a sequential row number from consecutive dates yields the same starting anchor date (`group_key`), grouping consecutive login streaks together.

---

### Q28: Cohort Retention (Hard)
**Scenario:** Find the percentage of users signing up in Jan 2024 who made a purchase in Feb 2024.
* **Table:** `users` (`user_id` INT, `signup_date` DATE)
* **Table:** `purchases` (`purchase_id` INT, `user_id` INT, `purchase_date` DATE)
* **SQL Query:**
```sql
WITH Cohort AS (
  SELECT user_id FROM users WHERE signup_date BETWEEN '2024-01-01' AND '2024-01-31'
),
ActiveFeb AS (
  SELECT DISTINCT user_id FROM purchases WHERE purchase_date BETWEEN '2024-02-01' AND '2024-02-29'
)
SELECT 
  COUNT(c.user_id) AS cohort_size,
  ROUND(COUNT(f.user_id) * 100.0 / COUNT(c.user_id), 2) AS pct_retained
FROM Cohort c
LEFT JOIN ActiveFeb f ON c.user_id = f.user_id;
```
* **Explanation:** A `LEFT JOIN` preserves all users in the January cohort. The `COUNT(f.user_id)` sums active users, while `COUNT(c.user_id)` gives the cohort denominator.

---

### Q29: Sessionization (Hard)
**Scenario:** Group user event clicks into distinct sessions. A session ends if a user is inactive for 30 minutes or more.
* **Table:** `clicks` (`user_id` INT, `click_time` TIMESTAMP)
* **SQL Query:**
```sql
WITH Gaps AS (
  SELECT 
    user_id, click_time,
    CASE WHEN click_time - LAG(click_time, 1) OVER (PARTITION BY user_id ORDER BY click_time) >= INTERVAL '30 minutes' THEN 1 ELSE 0 END AS new_session
  FROM clicks
),
Sessions AS (
  SELECT 
    user_id, click_time,
    SUM(new_session) OVER (PARTITION BY user_id ORDER BY click_time) AS session_id
  FROM Gaps
)
SELECT user_id, session_id, MIN(click_time) AS start_time, MAX(click_time) AS end_time
FROM Sessions
GROUP BY user_id, session_id;
```
* **Explanation:** We flag any row with a gap of 30+ minutes as `1`, and compute a running total of these flags to create stable session grouping IDs.

---

### Q30: Database-Agnostic Median (Hard)
**Scenario:** Calculate the median salary for each department without using proprietary database-specific functions.
* **Table:** `employees` (`employee_id` INT, `department_id` INT, `salary` DECIMAL)
* **SQL Query:**
```sql
WITH Ranked AS (
  SELECT 
    department_id, salary,
    ROW_NUMBER() OVER (PARTITION BY department_id ORDER BY salary) AS row_num,
    COUNT(*) OVER (PARTITION BY department_id) AS total_count
  FROM employees
)
SELECT department_id, AVG(salary) AS median_salary
FROM Ranked
WHERE row_num IN (total_count / 2 + 1, (total_count + 1) / 2)
GROUP BY department_id;
```
* **Explanation:** We fetch the middle row index (or average the two middle row indexes for even counts) dynamically using integer-division offsets.
