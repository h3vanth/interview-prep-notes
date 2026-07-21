# Spring Boot Conditional Annotations

Spring Boot uses **conditional annotations** to decide whether a bean or configuration should be created.

Instead of creating every possible bean, Spring Boot checks conditions:

```
Condition matches
        |
        v
Create Bean

Condition fails
        |
        v
Skip Configuration
```

This is the core mechanism behind Spring Boot Auto-Configuration.

## Why Conditional Annotations Are Needed

Example:

Spring Boot provides a default `DataSource`:

```java
@Bean
public DataSource dataSource() {
    return new HikariDataSource();
}
```

But if the developer already provides:

```java
@Bean
public DataSource customDataSource() {
    return new CustomDataSource();
}
```

Spring Boot should not create another one.

This is handled by:

```java
@ConditionalOnMissingBean(DataSource.class)
```

Meaning:

```
Create default DataSource only if no DataSource exists
```

## @ConditionalOnClass

Creates configuration only when a specific class exists in the classpath.

Example:

```java
@Configuration
@ConditionalOnClass(ObjectMapper.class)
public class JacksonAutoConfiguration {

}
```

Meaning:

```
Jackson library present
        |
        v
Enable Jackson configuration
```

Real example:

If this dependency exists:

```xml
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
</dependency>
```

Spring Boot activates Jackson auto-configuration.

## @ConditionalOnMissingClass

Opposite of `@ConditionalOnClass`.

Creates configuration only when a class is NOT available.

Example:

```java
@Configuration
@ConditionalOnMissingClass("com.example.LegacyService")
public class DefaultConfiguration {

}
```

Meaning:

```
LegacyService class does not exist
        |
        v
Create default configuration
```

## @ConditionalOnBean

Creates a bean only if another bean already exists.

Example:

```java
@Configuration
@ConditionalOnBean(DataSource.class)
public class JdbcConfiguration {

}
```

Meaning:

```
DataSource bean exists
        |
        v
Enable JDBC configuration
```

## @ConditionalOnMissingBean

Creates a bean only when the bean does not already exist.

Example:

```java
@Bean
@ConditionalOnMissingBean
public ObjectMapper objectMapper() {
    return new ObjectMapper();
}
```

Flow:

```
User ObjectMapper exists?
          |
     Yes  |  No
          |
     Skip | Create default bean
```

This allows customization.

## @ConditionalOnProperty

Creates a bean based on application properties.

Example:

```java
@Configuration
@ConditionalOnProperty(
    name = "payment.enabled",
    havingValue = "true"
)
public class PaymentConfiguration {

}
```

application.properties:

```properties
payment.enabled=true
```

Result:

```
PaymentConfiguration created
```

If:

```properties
payment.enabled=false
```

Result:

```
Configuration skipped
```

Common attributes:

```java
name
havingValue
matchIfMissing
```

Example:

```java
@ConditionalOnProperty(
    name = "feature.enabled",
    havingValue = "true",
    matchIfMissing = true
)
```

Meaning:

```
Property missing
        |
        v
Still create bean
```

## @ConditionalOnWebApplication

Creates configuration only for web applications.

Example:

```java
@Configuration
@ConditionalOnWebApplication
public class WebConfiguration {

}
```

Matches:

```
Spring MVC application
        OR
Reactive Web application
```

## @ConditionalOnNotWebApplication

Opposite of `@ConditionalOnWebApplication`.

Used when configuration should run only for non-web applications.

Example:

```java
@Configuration
@ConditionalOnNotWebApplication
public class BatchConfiguration {

}
```

## @ConditionalOnResource

Creates configuration if a resource exists.

Example:

```java
@Configuration
@ConditionalOnResource(
    resources = "classpath:config.properties"
)
public class CustomConfig {

}
```

Checks:

```
src/main/resources/config.properties exists
```

## @ConditionalOnExpression

Uses a SpEL expression.

Example:

```java
@Configuration
@ConditionalOnExpression(
    "${feature.enabled:false}"
)
public class FeatureConfiguration {

}
```

Bean is created when expression evaluates to true.

## @ConditionalOnJava

Checks Java version.

Example:

```java
@Configuration
@ConditionalOnJava(
    JavaVersion.EIGHT
)
public class Java8Configuration {

}
```

## @Conditional

The base annotation for creating custom conditions.

Example:

```java
@Configuration
@Conditional(MyCustomCondition.class)
public class CustomConfiguration {

}
```

Custom condition:

```java
public class MyCustomCondition 
        implements Condition {

    @Override
    public boolean matches(
        ConditionContext context,
        AnnotatedTypeMetadata metadata) {

        return true;
    }
}
```

## Multiple Conditions

Multiple conditions can be combined.

Example:

```java
@Configuration
@ConditionalOnClass(DataSource.class)
@ConditionalOnMissingBean(DataSource.class)
public class DataSourceConfiguration {

}
```

Meaning:

```
DataSource class exists
        AND
No DataSource bean exists
        |
        v
Create DataSource bean
```

## Common Conditional Annotations Used in Spring Boot

| Annotation | Purpose |
|---|---|
| `@ConditionalOnClass` | Create bean if class exists |
| `@ConditionalOnMissingClass` | Create bean if class does not exist |
| `@ConditionalOnBean` | Create bean if another bean exists |
| `@ConditionalOnMissingBean` | Create bean if another bean does not exist |
| `@ConditionalOnProperty` | Create bean based on property value |
| `@ConditionalOnResource` | Create bean if resource exists |
| `@ConditionalOnWebApplication` | Create bean for web applications |
| `@ConditionalOnNotWebApplication` | Create bean for non-web applications |
| `@ConditionalOnExpression` | Create bean based on SpEL |
| `@ConditionalOnJava` | Create bean based on Java version |

## Auto Configuration Example

Simplified:

```java
@Configuration
@ConditionalOnClass(DataSource.class)
@ConditionalOnProperty(
    name = "spring.datasource.url"
)
@ConditionalOnMissingBean(DataSource.class)
public class DataSourceAutoConfiguration {

    @Bean
    public DataSource dataSource() {
        return new HikariDataSource();
    }
}
```

Spring Boot checks:

```
1. Is DataSource class available?
             |
             v
2. Is datasource property configured?
             |
             v
3. Does DataSource bean already exist?
             |
             v
Create DataSource
```

## Interview Summary

> Spring Boot conditional annotations control whether a configuration or bean should be created. Auto-configuration classes use these annotations to apply configuration only when required conditions are satisfied, allowing Spring Boot to provide defaults while still allowing developers to override them.
