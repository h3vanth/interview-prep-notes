# Pagination in Spring Boot with Spring Data JPA

## What is Pagination?

Pagination is the process of fetching data in smaller chunks instead of loading the entire dataset at once.

Example:

Instead of:

```sql
SELECT * FROM employee;
```

which returns 1 million records,

we fetch:

```
Page 1 → 20 records
Page 2 → 20 records
Page 3 → 20 records
```

Benefits:

- Reduces memory usage
- Improves API response time
- Reduces database load
- Improves user experience

# Spring Data JPA Pagination

Spring Data JPA provides pagination using:

```java
Pageable
```

and returns:

```java
Page<T>
```

## Repository Setup

Entity:

```java
@Entity
public class Employee {

    @Id
    private Long id;

    private String name;

    private double salary;

}
```

Repository:

```java
@Repository
public interface EmployeeRepository 
        extends JpaRepository<Employee, Long> {

}
```

`JpaRepository` already provides:

```java
Page<T> findAll(Pageable pageable);
```

# Basic Pagination Example

## Service Layer

```java
@Service
public class EmployeeService {


    private final EmployeeRepository repository;


    public EmployeeService(
            EmployeeRepository repository
    ) {
        this.repository = repository;
    }


    public Page<Employee> getEmployees(
            int page,
            int size
    ) {

        Pageable pageable =
            PageRequest.of(page, size);


        return repository.findAll(pageable);
    }
}
```

## Controller

```java
@RestController
@RequestMapping("/employees")
public class EmployeeController {


    private final EmployeeService service;


    @GetMapping
    public Page<Employee> getEmployees(
            @RequestParam(defaultValue = "0")
            int page,

            @RequestParam(defaultValue = "10")
            int size
    ) {

        return service.getEmployees(
            page,
            size
        );
    }
}
```

Request:

```
GET /employees?page=0&size=10
```

Response:

```json
{
  "content": [
    {
      "id":1,
      "name":"John"
    }
  ],
  "pageable": {
    "pageNumber":0,
    "pageSize":10
  },
  "totalElements":100,
  "totalPages":10
}
```

# Page vs Slice

Spring Data provides:

- Page
- Slice

## Page

`Page` executes an additional count query.

Example:

Data query:

```sql
SELECT *
FROM employee
LIMIT 10
OFFSET 0;
```

Count query:

```sql
SELECT COUNT(*)
FROM employee;
```

Response:

```json
{
 "content": [],
 "totalElements":100,
 "totalPages":10
}
```

Use when you need:

- Total records
- Total pages
- Page numbers

## Slice

`Slice` does not execute count query.

Example:

```java
Slice<Employee> findAll(Pageable pageable);
```

Response:

```json
{
 "content": [],
 "hasNext":true
}
```

Use when:

- Infinite scrolling
- Large datasets
- Count is expensive

# Sorting with Pagination

Example:

```java
Pageable pageable =
    PageRequest.of(
        0,
        10,
        Sort.by("salary")
            .descending()
    );
```

Generated SQL:

```sql
SELECT *
FROM employee
ORDER BY salary DESC
LIMIT 10
OFFSET 0;
```

# Multiple Sort Fields

Example:

```java
Sort sort =
    Sort.by(
        Sort.Order.desc("salary"),
        Sort.Order.asc("name")
    );


Pageable pageable =
    PageRequest.of(
        0,
        10,
        sort
    );
```

SQL:

```sql
ORDER BY salary DESC, name ASC
```

# Pagination with Custom Query

Example:

```java
@Query("""
SELECT e
FROM Employee e
WHERE e.department = :dept
""")
Page<Employee> findByDepartment(
        String dept,
        Pageable pageable
);
```

Usage:

```java
repository.findByDepartment(
    "IT",
    PageRequest.of(0,10)
);
```

Spring automatically adds:

```sql
LIMIT
OFFSET
```

# Pagination with Native Query

Example:

```java
@Query(
    value = """
    SELECT *
    FROM employee
    WHERE department = :dept
    """,

    countQuery = """
    SELECT COUNT(*)
    FROM employee
    WHERE department = :dept
    """,

    nativeQuery = true
)
Page<Employee> findEmployees(
    String dept,
    Pageable pageable
);
```

## Why countQuery is Needed?

For `Page<T>`, Spring needs:

```
Total records
```

The count query provides:

```sql
SELECT COUNT(*)
```

Without it, complex native queries may fail.

# Pagination with DTO Projection

For REST APIs, returning entities is often not recommended.

Example DTO:

```java
public record EmployeeDTO(
    Long id,
    String name,
    double salary
) {}
```

Repository:

```java
@Query("""
SELECT new com.example.EmployeeDTO(
    e.id,
    e.name,
    e.salary
)
FROM Employee e
""")
Page<EmployeeDTO> findEmployees(
        Pageable pageable
);
```

Benefits:

- Fetch only required columns
- Less memory usage
- Avoid lazy loading issues

# Pagination and N+1 Problem

Problem:

```java
Page<Order> orders =
    repository.findAll(pageable);


orders.forEach(order ->
    order.getCustomer().getName()
);
```

May generate:

```
1 query for orders
+
N queries for customers
```

Solutions:

## EntityGraph

```java
@EntityGraph(
    attributePaths = {
        "customer"
    }
)
Page<Order> findAll(
    Pageable pageable
);
```

## DTO Projection

```java
SELECT new OrderDTO(
    o.id,
    c.name
)
FROM Order o
JOIN o.customer c
```

# Offset Pagination

Default Spring pagination uses:

```sql
LIMIT
OFFSET
```

Example:

Page 100:

```sql
LIMIT 20 OFFSET 2000;
```

Problem:

Large offsets become slow.

Database must scan and skip rows.

# Keyset Pagination (Cursor Pagination)

Better for large datasets.

Instead of:

```
OFFSET 100000
```

Use:

```
WHERE id > last_seen_id
```

Example:

First request:

```sql
SELECT *
FROM employee
ORDER BY id
LIMIT 20;
```

Last record:

```
id = 500
```

Next request:

```sql
SELECT *
FROM employee
WHERE id > 500
ORDER BY id
LIMIT 20;
```

Advantages:

- Faster for large tables
- More stable results

# PostgreSQL Pagination Optimization

## Add Index

Bad:

```sql
SELECT *
FROM employee
ORDER BY created_at
LIMIT 20;
```

Better:

```sql
CREATE INDEX idx_employee_created_at
ON employee(created_at);
```

Now PostgreSQL can use the index.

## Avoid Fetching Huge Pages

Bad:

```
page=10000
size=100
```

Better:

```
Cursor pagination
```

# Common Interview Questions

## 1. Difference between Page and Slice?

```
Page:
- Executes count query
- Gives total pages
- Used for page navigation

Slice:
- No count query
- Only tells hasNext
- Used for infinite scrolling
```

## 2. How does Spring Data implement pagination?

```
Controller
    |
    v
Pageable
    |
    v
Repository
    |
    v
Hibernate
    |
    v
SQL LIMIT + OFFSET
```

## 3. How to avoid N+1 during pagination?

Use:

- EntityGraph
- JOIN FETCH
- DTO Projection
- Batch fetching

## 4. Why is OFFSET pagination slow?

Because database must:

```
Read rows
Skip OFFSET rows
Return requested rows
```

For large offsets this becomes expensive.

# Interview Summary

```
Spring Boot pagination is implemented using Pageable and Page.

PageRequest creates pagination information:

PageRequest.of(page, size)

Spring Data JPA converts it into SQL LIMIT and OFFSET.

Page provides total count information but executes an additional count query.

Slice avoids the count query and is better for infinite scrolling.

For large datasets, cursor/keyset pagination is preferred over OFFSET pagination.
```

## One-Line Interview Answer

> Spring Data JPA pagination uses Pageable and PageRequest to generate LIMIT/OFFSET queries. Page provides total record information using a count query, while Slice avoids the count query. For large datasets, keyset pagination is preferred for better performance.
