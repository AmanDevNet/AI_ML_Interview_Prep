# Chapter 4: Joins in Depth

---

## 16. Joins

### 1. Definition
A Join is an operation used to combine rows from two or more tables based on a related column (usually a foreign key pointing to a primary key).

### 2. Why It Exists
In relational databases, data is normalized—meaning it is split across multiple tables to avoid duplication. Joins exist to reconstruct the relationships, allowing you to fetch combined records on-demand.

---

## Join Types Breakdown

### 1. INNER JOIN
* **Definition:** Returns only the rows that have matching values in **both** tables.
* **Syntax:**
  ```sql
  SELECT employees.name, departments.name
  FROM employees
  INNER JOIN departments ON employees.department_id = departments.id;
  ```
* **Example:** Find employees who belong to a department. If an employee has a `NULL` department, or a department has no employees, they are excluded from the output.
* **Interview Explanation:** *"Inner join is an intersection. It evaluates the ON condition and only returns rows where the matching column values exist in both the left and right tables."*

---

### 2. LEFT JOIN (LEFT OUTER JOIN)
* **Definition:** Returns **all** rows from the left table, and the matched rows from the right table. If there is no match, the right side returns `NULL` values.
* **Syntax:**
  ```sql
  SELECT customers.name, orders.id
  FROM customers
  LEFT JOIN orders ON customers.id = orders.customer_id;
  ```
* **Example:** Find all customers and their orders. Customers who haven't ordered anything will still appear in the output, with `NULL` in the order ID column.
* **Interview Explanation:** *"Left join preserves the left-hand table. It is crucial when you need a complete list of parent entities (like users) regardless of whether they have child records (like comments or purchases)."*

---

### 3. RIGHT JOIN (RIGHT OUTER JOIN)
* **Definition:** Returns **all** rows from the right table, and the matched rows from the left table. If there is no match, the left side returns `NULL` values.
* **Syntax:**
  ```sql
  SELECT employees.name, departments.name
  FROM employees
  RIGHT JOIN departments ON employees.department_id = departments.id;
  ```
* **Example:** Find all departments and their employees. Departments with no employees will appear with `NULL` in the employee name.
* **Interview Explanation:** *"Right join is structurally the same as a Left join, just with the tables swapped. In practice, most data engineers avoid Right joins and stick to Left joins because reading left-to-right matches standard writing and is easier to read."*

---

### 4. FULL OUTER JOIN
* **Definition:** Returns rows when there is a match in **either** the left or the right table. If there is no match on one side, it fills the missing values with `NULL`.
* **Syntax:**
  ```sql
  SELECT employees.name, projects.name
  FROM employees
  FULL OUTER JOIN projects ON employees.project_id = projects.id;
  ```
* **Example:** Get a list of all employees and all projects, including employees not assigned to any project, and projects that have no assigned employees.
* **Interview Explanation:** *"Full outer join is a union of left and right joins. It is useful for reconciliations—like identifying discrepancies between two transactional tables (e.g. tracking items present in inventory but not sales, and vice versa)."*

---

### 5. CROSS JOIN (Cartesian Product)
* **Definition:** Returns the Cartesian product of the two tables—every row from the first table is joined with every row from the second table.
* **Syntax:**
  ```sql
  SELECT products.name, sizes.name
  FROM products
  CROSS JOIN sizes;
  -- Or implicitly: SELECT * FROM products, sizes;
  ```
* **Example:** Generating all possible shirt size/color combinations. If you have 10 products and 3 sizes, you get 30 rows.
* **Interview Explanation:** *"Cross join creates a matrix. It takes the N rows of Table A and multiplies them by the M rows of Table B, returning N * M rows. We use it to generate combinations or to multiply tables by config sets."*

---

### 6. SELF JOIN
* **Definition:** A Self Join is a regular join, but the table is joined **with itself**.
* **Syntax:**
  ```sql
  SELECT emp.name AS employee, mgr.name AS manager
  FROM employees emp
  LEFT JOIN employees mgr ON emp.manager_id = mgr.employee_id;
  ```
* **Example:** Querying hierarchical data, like finding which employee reports to which manager, when both employees and managers are stored in the same `employees` table.
* **Interview Explanation:** *"A self-join is just a normal join where the table plays two roles. To write it, you must use table aliases (like `e` for employee and `m` for manager) so the database engine can differentiate between the two instances of the table in memory."*

---

## Common Mistakes & Edge Cases

### 1. Filtering Left Joined tables in the `WHERE` clause
This is a very common bug. If you write:
```sql
-- ❌ WRONG: Converts LEFT JOIN to INNER JOIN!
SELECT customers.name, orders.amount
FROM customers
LEFT JOIN orders ON customers.id = orders.customer_id
WHERE orders.status = 'shipped';
```
If a customer has no orders, the `orders.status` is `NULL`. But `NULL = 'shipped'` is false, so those rows are discarded. You've silently turned your `LEFT JOIN` into an `INNER JOIN`.
* **Fix:** Put the condition inside the `ON` clause, or add a NULL check in `WHERE`:
```sql
-- ✅ CORRECT
SELECT customers.name, orders.amount
FROM customers
LEFT JOIN orders ON customers.id = orders.customer_id AND orders.status = 'shipped';
```

### 2. Mismatched Join Key datatypes
Attempting to join tables where one key is a `VARCHAR` and the other is an `INT` will fail in strict databases like PostgreSQL, or cause slow query execution times in MySQL due to dynamic runtime casting.

---

## Interview Questions (Top 5)

**Q1: What is the difference between an ON clause and a WHERE clause in a LEFT JOIN?**
> The `ON` clause defines the logic of how the tables are matched. If a row on the right table fails the `ON` condition, it is still returned in the output but populated with `NULL`s. 
> The `WHERE` clause filters the final output rows *after* the join has been completed. If a row fails the `WHERE` condition, it is completely removed from the result set.

**Q2: What is a Cartesian Product and how do you accidentally trigger it?**
> A Cartesian Product (N * M rows) is generated when you run a `CROSS JOIN`. 
> You accidentally trigger it by forgetting to include an `ON` clause in an implicit join (e.g. writing `SELECT * FROM table_a, table_b` without a matching condition), or by writing a join condition that has no common join keys.

**Q3: How do database engines execute Joins under the hood?**
> They use three main algorithms:
> * **Nested Loop Join:** For each row in Table A, scan Table B. Good for small tables.
> * **Hash Join:** Hash the join keys of Table A in memory, then scan Table B and look up keys. Very fast for large tables.
> * **Sort-Merge Join:** Sort both tables by the join key first, then merge them in a single sweep. Best when data is already sorted by indexes.

**Q4: How do you identify customers who have NEVER placed an order?**
> By running a `LEFT JOIN` from customers to orders and filtering where the order key `IS NULL`:
> ```sql
> SELECT customers.id, customers.name
> FROM customers
> LEFT JOIN orders ON customers.id = orders.customer_id
> WHERE orders.order_id IS NULL;
> ```

**Q5: What is the benefit of a SELF JOIN when handling hierarchy trees?**
> In relational databases, parent-child hierarchies (like employees reporting to managers, or category subfolders) are often stored in a single table using a parent ID column pointing back to the primary ID. A self join allows you to query this tree structure dynamically by referencing the table twice with separate aliases.

---

*← [Previous: Chapter 3 (Aggregations & Grouping)](03_Aggregations_Groups.md) | [Next: Chapter 5 (Subqueries & CTEs) →](05_Subqueries_CTEs.md)*
