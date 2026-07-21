# Frequently Asked SQL Problems in PostgreSQL (Interview Questions)

## 1. Find Second Highest Salary

### Table

```sql
employee

id | name | salary
-------------------
1  | John | 5000
2  | Alex | 7000
3  | Mike | 9000
4  | Sam  | 7000
```

## Solution 1: Using LIMIT and OFFSET

```sql
SELECT DISTINCT salary
FROM employee
ORDER BY salary DESC
LIMIT 1 OFFSET 1;
```

Output:

```
7000
```

Explanation:

```
ORDER BY DESC
9000
7000
5000

OFFSET 1 skips highest salary
LIMIT 1 gets second highest
```

## Solution 2: Using DENSE_RANK

```sql
SELECT salary
FROM (
    SELECT 
        salary,
        DENSE_RANK() OVER(
            ORDER BY salary DESC
        ) AS rank
    FROM employee
) e
WHERE rank = 2;
```

Best for interviews because it handles duplicates.

## 2. Find Nth Highest Salary

Example: Find 3rd highest salary.

```sql
SELECT salary
FROM (
    SELECT
        salary,
        DENSE_RANK() OVER(
            ORDER BY salary DESC
        ) AS rank
    FROM employee
) e
WHERE rank = 3;
```

## 3. Find Employees With Duplicate Records

Example:

Find duplicate emails.

```sql
SELECT 
    email,
    COUNT(*)
FROM employee
GROUP BY email
HAVING COUNT(*) > 1;
```

Output:

```
john@test.com | 2
```

## 4. Delete Duplicate Records

Example:

Keep one record and delete duplicates.

```sql
DELETE FROM employee
WHERE id NOT IN (

    SELECT MIN(id)
    FROM employee
    GROUP BY email

);
```

Using PostgreSQL `ROW_NUMBER()`:

```sql
DELETE FROM employee
WHERE id IN (

    SELECT id
    FROM (
        SELECT
            id,
            ROW_NUMBER() OVER(
                PARTITION BY email
                ORDER BY id
            ) AS rn
        FROM employee
    ) x
    WHERE rn > 1

);
```

## 5. Find Employees With Highest Salary Per Department

### Table

```
employee

id | name | dept | salary
--------------------------
1  | John | IT   | 5000
2  | Alex | IT   | 7000
3  | Sam  | HR   | 6000
```

Query:

```sql
SELECT *
FROM (
    SELECT
        e.*,
        DENSE_RANK() OVER(
            PARTITION BY dept
            ORDER BY salary DESC
        ) rank
    FROM employee e
) x
WHERE rank = 1;
```

Output:

```
Alex IT 7000
Sam  HR 6000
```

## 6. Find Top 3 Salaries in Each Department

```sql
SELECT *
FROM (
    SELECT
        e.*,
        DENSE_RANK() OVER(
            PARTITION BY dept
            ORDER BY salary DESC
        ) rank
    FROM employee e
) x
WHERE rank <= 3;
```

## 7. Find Employees Who Do Not Have a Department

Tables:

```
employee

id | name | dept_id


department

id | name
```

Query:

```sql
SELECT e.*
FROM employee e
LEFT JOIN department d
ON e.dept_id = d.id
WHERE d.id IS NULL;
```

## 8. Find Department With Maximum Employees

```sql
SELECT
    department_id,
    COUNT(*) AS employee_count
FROM employee
GROUP BY department_id
ORDER BY employee_count DESC
LIMIT 1;
```

## 9. Find Count of Employees Per Department

```sql
SELECT
    department_id,
    COUNT(*) AS total
FROM employee
GROUP BY department_id;
```

## 10. Find Employees Joined in Last 30 Days

PostgreSQL:

```sql
SELECT *
FROM employee
WHERE joining_date >= CURRENT_DATE - INTERVAL '30 days';
```

## 11. Find Running Total

Table:

```
sales

date       amount
-----------------
Jan-01     100
Jan-02     200
Jan-03     300
```

Query:

```sql
SELECT
    date,
    amount,
    SUM(amount) OVER(
        ORDER BY date
    ) AS running_total
FROM sales;
```

Output:

```
Jan-01 100 100
Jan-02 200 300
Jan-03 300 600
```

## 12. Difference Between Current and Previous Row

Using `LAG()`:

```sql
SELECT
    date,
    amount,
    amount -
    LAG(amount) OVER(
        ORDER BY date
    ) AS difference
FROM sales;
```

## 13. Find Duplicate Consecutive Records

Example:

```
status_log

id | status
-----------
1  | OPEN
2  | OPEN
3  | CLOSED
```

Query:

```sql
SELECT *
FROM (
    SELECT
        *,
        LAG(status) OVER(
            ORDER BY id
        ) previous_status
    FROM status_log
) x
WHERE status = previous_status;
```

## 14. Find Missing Numbers in Sequence

Table:

```
orders

id
---
1
2
4
5
```

Generate series:

```sql
SELECT generate_series(
    1,
    (SELECT MAX(id) FROM orders)
) AS missing_id

EXCEPT

SELECT id
FROM orders;
```

Output:

```
3
```

## 15. Find Employees Having Same Salary

```sql
SELECT
    salary,
    STRING_AGG(
        name,
        ', '
    )
FROM employee
GROUP BY salary
HAVING COUNT(*) > 1;
```

Output:

```
7000 | Alex, Sam
```

## 16. Find Third Highest Salary Without LIMIT

Using window functions:

```sql
SELECT salary
FROM (
    SELECT
        salary,
        ROW_NUMBER() OVER(
            ORDER BY salary DESC
        ) rn
    FROM employee
) x
WHERE rn = 3;
```

## 17. Inner Join vs Left Join Problem

Tables:

```
customer

id | name


orders

id | customer_id
```

Find customers without orders:

```sql
SELECT c.*
FROM customer c
LEFT JOIN orders o
ON c.id = o.customer_id
WHERE o.id IS NULL;
```

## 18. Find Latest Record Per User

Example:

```
login_history

user_id | login_time
```

Query:

```sql
SELECT *
FROM (
    SELECT
        *,
        ROW_NUMBER() OVER(
            PARTITION BY user_id
            ORDER BY login_time DESC
        ) rn
    FROM login_history
) x
WHERE rn = 1;
```

## 19. Find Longest Consecutive Streak

Example:

```
user_activity

user_id | activity_date
```

Using window functions:

```sql
SELECT
    user_id,
    COUNT(*) AS streak
FROM (
    SELECT
        user_id,
        activity_date,
        activity_date -
        ROW_NUMBER() OVER(
            PARTITION BY user_id
            ORDER BY activity_date
        )::int AS grp
    FROM user_activity
) x
GROUP BY user_id, grp;
```

## 20. Find Employees Reporting to Managers

Self Join:

Table:

```
employee

id | name | manager_id
```

Query:

```sql
SELECT
    e.name employee,
    m.name manager
FROM employee e
JOIN employee m
ON e.manager_id = m.id;
```

## 21. Find nth Highest Salary Using PostgreSQL Function

```sql
CREATE FUNCTION nth_salary(n INT)
RETURNS INT
AS $$

SELECT salary
FROM employee
ORDER BY salary DESC
OFFSET n-1
LIMIT 1;

$$ LANGUAGE SQL;
```

Usage:

```sql
SELECT nth_salary(3);
```

## 22. JSON Query in PostgreSQL

Table:

```
users

id | details(jsonb)
```

Example JSON:

```json
{
 "city":"Chennai",
 "age":30
}
```

Query:

```sql
SELECT
    details->>'city'
FROM users;
```

Output:

```
Chennai
```

## 23. Find Records Created Today

```sql
SELECT *
FROM orders
WHERE created_at::date = CURRENT_DATE;
```

## 24. Pagination Query

Page 1:

```sql
SELECT *
FROM employee
ORDER BY id
LIMIT 10
OFFSET 0;
```

Page 2:

```sql
SELECT *
FROM employee
ORDER BY id
LIMIT 10
OFFSET 10;
```

## 25. Find Most Recent 5 Records

```sql
SELECT *
FROM employee
ORDER BY created_at DESC
LIMIT 5;
```

# Important PostgreSQL Interview Topics

## Window Functions

Frequently asked:

- ROW_NUMBER()
- RANK()
- DENSE_RANK()
- LAG()
- LEAD()
- SUM() OVER()

## Joins

Must know:

- INNER JOIN
- LEFT JOIN
- RIGHT JOIN
- FULL JOIN
- SELF JOIN

## Aggregations

Must know:

- GROUP BY
- HAVING
- COUNT
- SUM
- AVG
- MAX
- MIN

## PostgreSQL Specific Features

Frequently asked:

- JSONB queries
- ARRAY operations
- generate_series()
- CTE (`WITH`)
- Recursive queries
- Indexes
- EXPLAIN ANALYZE

# Interview Tip

For senior Java + Spring Boot roles, the most frequently asked SQL problems are:

1. Nth highest salary
2. Duplicate records
3. Top N per group
4. Latest record per group
5. Pagination
6. Joins
7. Window functions
8. Running totals
9. Query optimization using indexes
10. CTE and recursive queries
