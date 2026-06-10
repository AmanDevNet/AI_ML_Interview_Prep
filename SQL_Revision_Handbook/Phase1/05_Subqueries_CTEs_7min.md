# Chapter 5: Subqueries & CTEs

---

## 17. Subqueries

### 1. Definition
A Subquery is a query nested inside another SQL query (inside `SELECT`, `FROM`, `WHERE`, or `HAVING` statements).

### 2. Why It Exists
Sometimes a query requires a value that is calculated dynamically by another query (for example, finding employees who earn more than the average salary). You calculate the average first in a subquery, then feed it to the main query.

### 3. Where It Is Used
Dynamic filtering, checking existence using `IN` or `EXISTS`, and calculated column values.

### 4. Syntax
```sql
SELECT columns 
FROM table_name 
WHERE column_name > (SELECT AGGREGATE(col) FROM other_table);
```

### 5. Simple Example
```sql
-- Find employees earning more than the company average
SELECT name, salary 
FROM employees 
WHERE salary > (SELECT AVG(salary) FROM employees);
```

### 6. Interview Explanation
> "A subquery is just a nested query. The database engine executes the inner query first, stores the resulting value or set of values in memory, and then runs the outer query using that inner result as a filter or parameter."

### 7. Common Mistakes
* **Single-row subquery returning multiple values:** If you write `WHERE salary = (SELECT salary FROM employees WHERE dept = 1)` and the inner query returns 3 salaries, the query will crash with an error: `Subquery returned more than 1 row`. Use `IN` instead of `=` for multi-value subqueries.

### 8. Follow-up Questions
* *What is the difference between a subquery inside the `WHERE` clause and one in the `FROM` clause? (A subquery in FROM acts as a temporary inline view table and must be aliased).*
* *How does `IN` differ from `EXISTS` when using subqueries?*

---

## 18. Correlated Subqueries

### 1. Definition
A Correlated Subquery is a nested query that references one or more columns from the **outer query**.

### 2. Why It Exists
Standard subqueries run once and return a fixed value. A correlated subquery must run **once for every single row** evaluated by the outer query because its conditions depend on that specific row's data.

### 3. Where It Is Used
Finding the highest-paid employee *per department*, or identifying customers who ordered above *their own* average order values.

### 4. Syntax
```sql
SELECT o.col1, o.col2
FROM outer_table o
WHERE o.val > (
    SELECT AGGREGATE(i.val)
    FROM inner_table i
    WHERE i.join_id = o.join_id -- Reference to outer query o!
);
```

### 5. Simple Example
```sql
-- Find employees who earn more than the average salary of THEIR department
SELECT outer_emp.name, outer_emp.salary, outer_emp.department_id
FROM employees outer_emp
WHERE outer_emp.salary > (
    SELECT AVG(inner_emp.salary)
    FROM employees inner_emp
    WHERE inner_emp.department_id = outer_emp.department_id -- Correlation link
);
```

### 6. Interview Explanation
> "In a correlated subquery, the inner query cannot run independently of the outer query. It acts like a nested `for` loop in Python: for each row in the outer table, Python passes that row's IDs into the inner query, executes the inner query, and uses the result to evaluate the outer row."

### 7. Common Mistakes
* **Assuming good performance on large tables:** Because a correlated subquery evaluates for every row, running it on a table with 1,000,000 rows will trigger 1,000,000 inner queries, slowing down performance. Rewrite them using joins or window functions.

### 8. Follow-up Questions
* *How can you optimize a correlated subquery? (By converting it to an INNER JOIN on a grouped subquery or using a CTE).*
* *Does `EXISTS` use correlated subqueries? (Yes, almost always, because it checks for matching criteria between the tables).*

---

## 19. CTEs (Common Table Expressions)

### 1. Definition
A CTE (Common Table Expression) is a temporary, named result set that you define before running your main query. It is created using the `WITH` keyword.

### 2. Why It Exists
Subqueries inside `FROM` clauses are hard to read and cannot be easily reused in the same query. CTEs make complex queries clean, structured, self-documenting, and allow referencing the same set multiple times.

### 3. Where It Is Used
Breaking down massive queries into sequential steps, calculating intermediate aggregates, and writing recursive queries (like parsing hierarchy charts).

### 4. Syntax
```sql
WITH cte_name AS (
    SELECT columns FROM table_name
)
SELECT * FROM cte_name;
```

### 5. Simple Example
```sql
-- Calculate avg department salary first, then filter departments
WITH DeptAverage AS (
    SELECT department_id, AVG(salary) AS avg_sal
    FROM employees
    GROUP BY department_id
)
SELECT d.department_id, d.avg_sal
FROM DeptAverage d
WHERE d.avg_sal > 60000;
```

### 6. Interview Explanation
> "A CTE acts like a named temporary variable in SQL. It makes code highly readable. Instead of writing deeply nested subqueries, I declare my intermediate steps as CTE blocks at the top, and then join or filter them in the main query below."

### 7. Common Mistakes
* **Assuming CTEs are stored on disk:** A standard CTE is in-memory and temporary; it only exists for the duration of that single query run.
* **Recursive CTE loop traps:** Writing a recursive CTE without a solid termination condition will run infinitely and crash the server due to memory exhaustion.

### 8. Follow-up Questions
* *What is a Recursive CTE and when do you use it? (It references itself, useful for traversing org charts or tree networks).*
* *Does a CTE run faster than a Subquery? (In modern database engines, the query planner optimizes both into the same execution plan, so performance is usually identical).*

---

*← [Previous: Chapter 4 (Joins in Depth)](04_Joins.md) | [Next: Chapter 6 (Window Functions) →](06_Window_Functions.md)*
