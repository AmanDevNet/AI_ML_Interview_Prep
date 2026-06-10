# Chapter 1: Relational Database Concepts

---

## 1. What is SQL?

### 1. Definition
SQL (Structured Query Language) is the standard programming language used to manage, query, and manipulate data stored in relational databases (RDBMS).

### 2. Why It Exists
Before SQL, querying databases required writing complex, low-level procedural code to tell the database *how* to search files on disk. SQL exists to provide a declarative, high-level language where you only define *what* data you want, leaving it to the database engine to find the most efficient way to retrieve it.

### 3. Where It Is Used
Everywhere data is stored: backing up backend applications (PostgreSQL/MySQL), data warehouses (Snowflake/BigQuery), ML data pipelines, logs ingestion, and analytics.

### 4. Syntax
```sql
SELECT column_name 
FROM table_name 
WHERE condition;
```

### 5. Simple Example
```sql
-- Fetch name of the user with ID 5
SELECT name 
FROM users 
WHERE id = 5;
```

### 6. Interview Explanation
> "Think of SQL as a declarative negotiator. I tell the database, 'Hey, give me these columns from this table where these conditions are met.' I don't need to specify how to navigate index trees or scan memory blocks; the database's query optimizer handles the execution details under the hood, making data fetching extremely clean and standardized."

### 7. Common Mistakes
* **Thinking SQL is procedural:** Beginners write queries expecting SQL to run line-by-line like a Python script. SQL runs declaratively as sets of data, and the execution engine re-arranges operations for speed.
* **Confusing SQL with a Database:** SQL is the *language*; PostgreSQL, MySQL, and SQLite are the *database engines* that interpret the language.

### 8. Follow-up Questions
* *What is the difference between DDL (Data Definition Language) and DML (Data Manipulation Language)?*
* *Is SQL a good fit for unstructured data like social media posts or raw video files?*

---

## 2. Tables, Rows, and Columns

### 1. Definition
* **Table:** A structured collection of related data organized in a grid.
* **Row (Record / Tuple):** A single, horizontal entry representing a unique item.
* **Column (Attribute / Field):** A vertical classification representing a specific property of the data.

### 2. Why It Exists
They provide a clean, two-dimensional structure to represent real-world entities (like users, orders, or products) and enforce consistent datatypes for each property.

### 3. Where It Is Used
Designing database schemas. Every table in a database defines the structure of a specific business object.

### 4. Syntax
```sql
CREATE TABLE table_name (
    column_1 datatype,
    column_2 datatype
);
```

### 5. Simple Example
```sql
-- Creating an employees table
CREATE TABLE employees (
    employee_id INT,
    first_name VARCHAR(50),
    hire_date DATE
);
```

### 6. Interview Explanation
> "A table represents a specific entity, like 'Customers'. Columns define the blueprint—the attributes that every customer must have, like an email address or signup date, with strict datatypes. A row is a concrete instance of that blueprint—one actual customer's records."

### 7. Common Mistakes
* **Using bad datatypes:** Storing phone numbers or zip codes as integers. If a zip code starts with `0` (e.g. `02111`), storing it as an `INT` will strip the leading zero, and you cannot format phone numbers easily. Always use `VARCHAR` for numbers you won't perform arithmetic on.
* **Bloated tables:** Storing too many unrelated attributes in a single table, rather than splitting them logically using normalization.

### 8. Follow-up Questions
* *What is the difference between a NULL value and an empty string in a column?*
* *How do you choose between `CHAR` and `VARCHAR` datatypes for text columns?*

---

## 3. Primary Key

### 1. Definition
A Primary Key is a column (or combination of columns) that uniquely identifies each row in a table. It must be unique and cannot contain `NULL` values.

### 2. Why It Exists
Without a primary key, you could have duplicate rows in a table with no way to reference or update a specific record uniquely. It also automatically builds a clustered index, making lookups by ID extremely fast.

### 3. Where It Is Used
Every table should have a primary key (usually an auto-incrementing integer `id` or a random UUID).

### 4. Syntax
```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50)
);
```

### 5. Simple Example
```sql
-- Creating a table with a Composite Primary Key (multiple columns combined)
CREATE TABLE order_items (
    order_id INT,
    product_id INT,
    quantity INT,
    PRIMARY KEY (order_id, product_id) -- Combined unique identifier
);
```

### 6. Interview Explanation
> "A Primary Key is the unique fingerprint for a row. It does two things: it guarantees that no two records can have the same identifier, and it automatically creates a physical index in memory so searching for `id = 450` takes constant time instead of scanning the whole table."

### 7. Common Mistakes
* **Using mutable values as primary keys:** Using an email address as a primary key. If the user changes their email, you have to cascade updates across all tables referencing that key, which degrades database performance. Always use immutable, surrogate keys like auto-incrementing integers or UUIDs.
* **Creating multiple primary keys:** A table can only have **one** primary key (though it can be a composite key containing multiple columns).

### 8. Follow-up Questions
* *What is the difference between a Natural Key and a Surrogate Key?*
* *When would you choose a UUID over an auto-incrementing integer as a primary key?*

---

## 4. Foreign Key

### 1. Definition
A Foreign Key is a column (or columns) in one table that points to the Primary Key of another table, establishing a relationship between them.

### 2. Why It Exists
To enforce **Referential Integrity**. It prevents orphaned records (for example, inserting an order for a customer ID that doesn't exist, or deleting a customer while their orders are still on file).

### 3. Where It Is Used
Linking tables together (e.g., matching `orders.customer_id` to `customers.id`).

### 4. Syntax
```sql
CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    customer_id INT REFERENCES customers(id) ON DELETE CASCADE
);
```

### 5. Simple Example
```sql
-- Creating an orders table linked to customers
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    FOREIGN KEY (customer_id) REFERENCES customers(id)
);
```

### 6. Interview Explanation
> "Foreign keys are the glue of relational databases. They enforce rules at the database level so the application code doesn't accidentally orphan data. If I declare `orders.customer_id` as a foreign key referencing `customers.id`, the database will block me from inserting an order for a customer that doesn't exist, and can automatically clean up orders if a customer is deleted."

### 7. Common Mistakes
* **Forgetting to index foreign keys:** Databases do not automatically index foreign keys. Because you join tables on foreign keys constantly, neglecting to index them manually leads to slow table scans during joins.
* **Not configuring deletion rules:** If you delete a parent record and have no `ON DELETE` rule configured, the database will block the deletion, which can crash your application flows unless handled.

### 8. Follow-up Questions
* *What is the difference between `ON DELETE CASCADE` and `ON DELETE SET NULL`?*
* *How do foreign keys impact the write performance (insert/update) of a table?*

---

## 5. Constraints

### 1. Definition
Constraints are rules enforced on data columns in a table. They limit the type of data that can go into a table.

### 2. Why It Exists
To maintain data quality and consistency directly inside the database, preventing invalid data from being saved even if validation fails in your application code.

### 3. Where It Is Used
Restricting fields like age to be positive, checking email format, making columns mandatory, and enforcing unique usernames.

### 4. Syntax
```sql
CREATE TABLE users (
    id INT PRIMARY KEY,
    age INT CHECK (age >= 18),
    email VARCHAR(100) UNIQUE NOT NULL
);
```

### 5. Simple Example
```sql
-- Adding a constraint to an existing table
ALTER TABLE employees
ADD CONSTRAINT chk_salary CHECK (salary > 0);
```

### 6. Interview Explanation
> "Constraints are the database's defense mechanism. By defining constraints like `NOT NULL`, `UNIQUE`, or `CHECK(price > 0)` directly on the table schema, we ensure database integrity. Even if a bug in the backend code bypasses validation, the database will raise an error and roll back the transaction before writing bad data."

### 7. Common Mistakes
* **Ignoring constraints and relying solely on backend validation:** If someone writes a direct script to insert values or accesses the database via an admin panel, they can easily insert invalid data if there are no database-level constraints.
* **Overcomplicating check constraints:** Putting highly complex, volatile business logic in constraints. Business rules change often, and altering constraints in a massive production table requires locks that can cause downtime.

### 8. Follow-up Questions
* *Can a column have multiple constraints?*
* *What is the difference between a `UNIQUE` constraint and a `PRIMARY KEY`?*

---

*← [Previous: Phase 3 Chapter 10 (Interview Handbook)](../Phase3/10_Phase3_Interview_Handbook.md) | [Next: Chapter 2 (Basic Queries & Filtering) →](02_Basic_Queries.md)*
