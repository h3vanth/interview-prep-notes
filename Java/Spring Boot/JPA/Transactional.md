# @Transactional Annotation in Spring Boot (PostgreSQL)

`@Transactional` is a Spring annotation used to manage database transactions.

It ensures that a group of database operations execute as a **single atomic unit**.

Example:

```java
@Transactional
public void transferMoney(
        Long fromAccount,
        Long toAccount,
        double amount
) {

    debit(fromAccount, amount);

    credit(toAccount, amount);

}
```

If `credit()` fails:

```
Debit
  |
  v
Credit (fails)
  |
  v
Rollback debit
```

The entire transaction is rolled back.

## How @Transactional Works Internally

Spring uses AOP proxies.

Flow:

```
Client
  |
  v
@Transactional Proxy
  |
  v
Begin Transaction
  |
  v
Service Method
  |
  v
Commit / Rollback
```

The proxy:

1. Opens a transaction before method execution.
2. Executes the method.
3. Commits if successful.
4. Rolls back if an exception occurs.

## Common @Transactional Attributes

```java
@Transactional(
    propagation = Propagation.REQUIRED,
    isolation = Isolation.READ_COMMITTED,
    timeout = 30,
    readOnly = false,
    rollbackFor = Exception.class
)
```

## 1. propagation

Defines how a transaction behaves when another transaction already exists.

Attribute:

```java
propagation = Propagation.REQUIRED
```

### REQUIRED (Default)

Uses existing transaction if available.

Otherwise creates a new one.

Example:

```
Transaction A
     |
     v
Service Method
     |
     v
Uses Transaction A
```

Code:

```java
@Transactional(
    propagation = Propagation.REQUIRED
)
```

Most commonly used.

### REQUIRES_NEW

Always creates a new transaction.

Existing transaction is suspended.

Example:

```
Transaction A
     |
     |
     +---- Service Method
              |
              v
        Transaction B
```

Use case:

Saving audit logs even if the main transaction fails.

```java
@Transactional(
    propagation = Propagation.REQUIRES_NEW
)
public void saveAudit() {

}
```

### SUPPORTS

Uses existing transaction if available.

Otherwise executes without a transaction.

```java
@Transactional(
    propagation = Propagation.SUPPORTS
)
```

### NOT_SUPPORTED

Executes without a transaction.

Suspends existing transaction.

```java
@Transactional(
    propagation = Propagation.NOT_SUPPORTED
)
```

### MANDATORY

Requires an existing transaction.

Throws exception if none exists.

```java
@Transactional(
    propagation = Propagation.MANDATORY
)
```

### NEVER

Must execute without a transaction.

Throws exception if a transaction exists.

```java
@Transactional(
    propagation = Propagation.NEVER
)
```

### NESTED

Creates a nested transaction using savepoints.

Example:

```
Main Transaction
       |
       v
   Savepoint
       |
       v
 Nested Transaction
```

Supported by databases that support savepoints.

PostgreSQL supports savepoints.

```java
@Transactional(
    propagation = Propagation.NESTED
)
```

## 2. isolation

Controls how one transaction sees changes made by another transaction.

Example:

```java
@Transactional(
    isolation = Isolation.READ_COMMITTED
)
```

## PostgreSQL Supported Isolation Levels

PostgreSQL supports:

| Spring Isolation | PostgreSQL Support |
|---|---|
| DEFAULT | Yes |
| READ_COMMITTED | Yes (Default PostgreSQL isolation) |
| REPEATABLE_READ | Yes |
| SERIALIZABLE | Yes |

PostgreSQL does not support:

```
READ_UNCOMMITTED
```

It treats it as:

```
READ_COMMITTED
```

## Isolation Levels

### READ_COMMITTED

Default in PostgreSQL.

Prevents:

```
Dirty Reads
```

Example:

Transaction A:

```
Update balance = 500
```

Transaction B cannot see it until commit.

### REPEATABLE_READ

Same query returns the same result within a transaction.

Prevents:

```
Dirty Reads
Non-repeatable Reads
```

### SERIALIZABLE

Highest isolation.

Transactions behave as if executed sequentially.

Prevents:

```
Dirty Reads
Non-repeatable Reads
Phantom Reads
```

## 3. timeout

Maximum transaction execution time.

Example:

```java
@Transactional(
    timeout = 10
)
```

Meaning:

```
Transaction must complete within 10 seconds
```

Otherwise:

```
TransactionRollbackException
```

## 4. readOnly

Marks transaction as read-only.

Example:

```java
@Transactional(
    readOnly = true
)
public List<User> findUsers() {

}
```

Useful for:

- SELECT operations
- Performance optimization

Example:

```java
@Transactional(readOnly = true)
public User findById(Long id) {
    return repository.findById(id)
                     .orElseThrow();
}
```

For PostgreSQL:

Spring sends:

```sql
SET TRANSACTION READ ONLY
```

This prevents write operations inside the transaction.

## 5. rollbackFor

Defines exceptions that should trigger rollback.

Example:

```java
@Transactional(
    rollbackFor = IOException.class
)
```

By default:

Rollback happens for:

```
RuntimeException
Error
```

Not for:

```
Checked Exceptions
```

Example:

```java
@Transactional
public void processFile()
        throws IOException {

}
```

IOException will not rollback by default.

Use:

```java
@Transactional(
    rollbackFor = IOException.class
)
```

## 6. noRollbackFor

Defines exceptions that should NOT cause rollback.

Example:

```java
@Transactional(
    noRollbackFor = ValidationException.class
)
```

Even if:

```
ValidationException occurs
```

transaction commits.

## PostgreSQL Example

Entity:

```java
@Entity
public class Account {

    @Id
    private Long id;

    private double balance;

}
```

Service:

```java
@Service
public class TransferService {


    @Transactional(
        isolation = Isolation.READ_COMMITTED,
        propagation = Propagation.REQUIRED,
        timeout = 20,
        rollbackFor = Exception.class
    )
    public void transfer(
            Long from,
            Long to,
            double amount
    ) {

        accountRepository.debit(from, amount);

        accountRepository.credit(to, amount);
    }
}
```

## Transaction Rollback Example

```java
@Transactional
public void createOrder() {

    orderRepository.save(order);

    paymentService.pay();

    throw new RuntimeException(
        "Payment failed"
    );
}
```

Result:

```
Order insert
     |
Payment failed
     |
Rollback
     |
Order removed
```

## Important Interview Points

### Runtime vs Checked Exception

Default:

```
RuntimeException
        |
        v
Rollback
```

Checked exception:

```
Exception
        |
        v
Commit
```

unless configured:

```java
rollbackFor = Exception.class
```

## Self Invocation Problem

This does NOT trigger transaction:

```java
@Service
public class OrderService {

    public void methodA() {
        methodB();
    }

    @Transactional
    public void methodB() {

    }
}
```

Reason:

```
methodA()
   |
   v
same object call
   |
   v
Proxy bypassed
```

## PostgreSQL Recommended Usage

For most Spring Boot applications:

```java
@Transactional(
    propagation = Propagation.REQUIRED,
    isolation = Isolation.READ_COMMITTED
)
```

For read-only queries:

```java
@Transactional(readOnly = true)
```

For critical financial operations:

```java
@Transactional(
    isolation = Isolation.SERIALIZABLE,
    rollbackFor = Exception.class
)
```

## Interview Summary

```
@Transactional manages database transactions in Spring.

It uses AOP proxies to begin, commit, and rollback transactions.

Important attributes:

propagation  -> transaction behavior
isolation    -> data visibility between transactions
timeout      -> maximum execution time
readOnly     -> optimize read operations
rollbackFor  -> exceptions causing rollback
noRollbackFor -> exceptions ignored for rollback

PostgreSQL supports:
READ_COMMITTED
REPEATABLE_READ
SERIALIZABLE
```
