# Spring Boot AOP (Aspect-Oriented Programming)

## What is AOP?

AOP (Aspect-Oriented Programming) is a programming technique used to separate **cross-cutting concerns** from business logic.

Cross-cutting concerns are functionalities that are needed in many places:

- Logging
- Security
- Transaction management
- Auditing
- Performance monitoring
- Exception handling

Without AOP:

```java
public void createOrder() {

    log.info("Create order started");

    // Business logic

    log.info("Create order completed");
}
```

The logging code gets repeated in many methods.

With AOP:

```java
public void createOrder() {

    // Only business logic
}
```

Logging is handled separately by an Aspect.

## AOP Terminology

```
Aspect
 |
 |-- Pointcut
 |
 |-- Advice
 |
 |-- Join Point
```

## 1. Aspect

An Aspect contains cross-cutting logic.

Example:

```java
@Aspect
@Component
public class LoggingAspect {

}
```

It defines:

- What should run
- When it should run

## 2. Join Point

A Join Point is a point during program execution where an aspect can be applied.

In Spring AOP:

```
Only method execution join points are supported
```

Example:

```java
public void saveUser()
```

The execution of this method is a Join Point.

## 3. Pointcut

A Pointcut defines **where** the advice should execute.

Example:

```java
@Pointcut("execution(* com.example.service.*.*(..))")
public void serviceMethods() {

}
```

Meaning:

```
Any method
inside service package
with any arguments
```

## 4. Advice

Advice defines **when** the aspect logic runs.

Types:

| Advice | When it Executes |
|---|---|
| @Before | Before method execution |
| @After | After method execution (finally block style) |
| @AfterReturning | After successful execution |
| @AfterThrowing | When exception occurs |
| @Around | Before and after execution |

# Example Project

## Service Class

```java
@Service
public class OrderService {

    public String createOrder(String product) {

        System.out.println("Creating order");

        return "Order created";
    }
}
```

## Logging Aspect

```java
@Aspect
@Component
public class LoggingAspect {


    @Before(
      "execution(* com.example.service.*.*(..))"
    )
    public void logBefore() {

        System.out.println(
            "Method execution started"
        );
    }
}
```

Output:

```
Method execution started
Creating order
```

The service class does not contain logging code.

# @Before Advice

Runs before the method.

Example:

```java
@Before("execution(* com.example.service.*.*(..))")
public void beforeMethod() {

    System.out.println("Before method");
}
```

Flow:

```
Aspect
  |
  v
Before Advice
  |
  v
Service Method
```

# @After Advice

Runs after method execution regardless of success or failure.

Example:

```java
@After("execution(* com.example.service.*.*(..))")
public void afterMethod() {

    System.out.println("Cleanup");
}
```

Similar to:

```java
finally {
    cleanup();
}
```

# @AfterReturning Advice

Runs only when method completes successfully.

Example:

```java
@AfterReturning(
    pointcut = "execution(* com.example.service.*.*(..))",
    returning = "result"
)
public void afterSuccess(Object result) {

    System.out.println(
        "Result: " + result
    );
}
```

Execution:

```
Method success
       |
       v
AfterReturning runs
```

# @AfterThrowing Advice

Runs when an exception occurs.

Example:

```java
@AfterThrowing(
    pointcut = "execution(* com.example.service.*.*(..))",
    throwing = "exception"
)
public void handleException(Exception exception) {

    System.out.println(
        exception.getMessage()
    );
}
```

Flow:

```
Method
  |
Exception
  |
AfterThrowing
```

# @Around Advice

Most powerful advice.

It can:

- Execute before method
- Execute after method
- Modify input
- Modify output
- Prevent execution

Example:

```java
@Around(
    "execution(* com.example.service.*.*(..))"
)
public Object measureTime(
        ProceedingJoinPoint joinPoint
) throws Throwable {


    long start = System.currentTimeMillis();


    Object result = joinPoint.proceed();


    long end = System.currentTimeMillis();


    System.out.println(
        "Time taken: " +
        (end - start)
    );


    return result;
}
```

Flow:

```
Around Advice
      |
      v
Before logic
      |
      v
Actual Method
      |
      v
After logic
```

# Real-Time Example: Performance Monitoring

Service:

```java
@Service
public class PaymentService {

    public void processPayment() {

        // Payment logic

    }
}
```

Aspect:

```java
@Aspect
@Component
public class PerformanceAspect {


    @Around(
      "execution(* com.example.service.*.*(..))"
    )
    public Object monitor(
        ProceedingJoinPoint joinPoint
    ) throws Throwable {


        long start =
            System.currentTimeMillis();


        Object response =
            joinPoint.proceed();


        long end =
            System.currentTimeMillis();


        System.out.println(
            joinPoint.getSignature()
            + " took "
            + (end - start)
            + " ms"
        );


        return response;
    }
}
```

Output:

```
PaymentService.processPayment took 25 ms
```

# Real-Time Example: Authorization Check

Controller:

```java
@RestController
public class UserController {


    @GetMapping("/users")
    public List<User> getUsers() {

        return userService.findAll();
    }
}
```

Security Aspect:

```java
@Aspect
@Component
public class SecurityAspect {


    @Before(
      "@annotation(AdminOnly)"
    )
    public void checkPermission() {

        // Check user role

    }
}
```

Before executing the method:

```
Request
 |
 v
Security Aspect
 |
 v
Controller Method
```

# AOP Proxy Mechanism

Spring AOP works using proxies.

Two types:

## JDK Dynamic Proxy

Used when class implements an interface.

Example:

```
Interface
    |
    v
Proxy Object
    |
    v
Actual Object
```

## CGLIB Proxy

Used when there is no interface.

Spring creates a subclass:

```
Original Class
      |
      v
CGLIB Proxy
```

# Enabling AOP

Spring Boot usually enables it automatically when adding:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>
        spring-boot-starter-aop
    </artifactId>
</dependency>
```

You can also use:

```java
@EnableAspectJAutoProxy
```

# Common AOP Use Cases

| Use Case | Advice Type |
|---|---|
| Logging | @Before / @Around |
| Performance monitoring | @Around |
| Transactions | @Around |
| Security checks | @Before |
| Audit logging | @AfterReturning |
| Exception tracking | @AfterThrowing |

# AOP Limitations

## Self Invocation Problem

Example:

```java
@Service
class UserService {

    public void methodA() {
        methodB();
    }

    @Transactional
    public void methodB() {

    }
}
```

`methodB()` may not trigger AOP because the call happens inside the same object.

Reason:

```
Object
 |
 methodA()
 |
 methodB()

(no proxy involved)
```

## Private Methods

Spring AOP does not intercept:

```java
private void calculate()
```

because proxy works only with public method calls.

# Interview Summary

```
Spring AOP separates cross-cutting concerns from business logic.

It works using proxies and applies additional behavior using aspects.

Aspect contains advice and pointcuts.

Advice defines when the logic executes:
@Before
@After
@AfterReturning
@AfterThrowing
@Around

Spring AOP is commonly used for logging, transactions, security,
auditing, and performance monitoring.
```

**One-line interview answer:**

> Spring AOP allows us to modularize cross-cutting concerns by applying additional behavior to methods using proxies, aspects, pointcuts, and advice without changing the actual business logic.
