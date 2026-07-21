## Sort Ascending

```java
List<Integer> result = list.stream()
        .sorted()
        .toList();
```

## Sort Descending

```java
List<Integer> result = list.stream()
        .sorted(Comparator.reverseOrder())
        .toList();
```

## Find Maximum Number

```java
int max = list.stream()
        .max(Integer::compare)
        .orElse(0);
```

## Find Minimum Number

```java
int min = list.stream()
        .min(Integer::compare)
        .orElse(0);
```

## Sum of Numbers

```java
int sum = list.stream()
        .mapToInt(Integer::intValue)
        .sum();
```

## Average of Numbers

```java
double avg = list.stream()
        .mapToInt(Integer::intValue)
        .average()
        .orElse(0);
```

## Count Elements

```java
long count = list.stream().count();
```

## Find First Element

```java
Integer first = list.stream()
        .findFirst()
        .orElse(null);
```

## Find Any Element

```java
Integer any = list.stream()
        .findAny()
        .orElse(null);
```

## Check If All Numbers Are Positive

```java
boolean result = list.stream()
        .allMatch(n -> n > 0);
```

## Check If Any Number Is Negative

```java
boolean result = list.stream()
        .anyMatch(n -> n < 0);
```

## Check If No Number Is Zero

```java
boolean result = list.stream()
        .noneMatch(n -> n == 0);
```

## Convert List to Set

```java
Set<Integer> set = list.stream()
        .collect(Collectors.toSet());
```

## Join Strings

```java
String result = names.stream()
        .collect(Collectors.joining(", "));
```

## Convert Strings to Uppercase

```java
List<String> result = names.stream()
        .map(String::toUpperCase)
        .toList();
```

## Convert Strings to Lowercase

```java
List<String> result = names.stream()
        .map(String::toLowerCase)
        .toList();
```

## Filter Names Starting with "A"

```java
List<String> result = names.stream()
        .filter(name -> name.startsWith("A"))
        .toList();
```

## Find Longest String

```java
String longest = names.stream()
        .max(Comparator.comparing(String::length))
        .orElse("");
```

## Find Shortest String

```java
String shortest = names.stream()
        .min(Comparator.comparing(String::length))
        .orElse("");
```

## Count Frequency of Elements

```java
Map<Integer, Long> frequency = list.stream()
        .collect(Collectors.groupingBy(
                Function.identity(),
                Collectors.counting()));
```

## Find Duplicate Elements

```java
Set<Integer> duplicates = list.stream()
        .filter(n -> Collections.frequency(list, n) > 1)
        .collect(Collectors.toSet());
```

## Find Unique Elements

```java
List<Integer> result = list.stream()
        .distinct()
        .toList();
```

## Find Second Highest Number

```java
Integer secondHighest = list.stream()
        .distinct()
        .sorted(Comparator.reverseOrder())
        .skip(1)
        .findFirst()
        .orElse(null);
```

## Find Second Smallest Number

```java
Integer secondSmallest = list.stream()
        .distinct()
        .sorted()
        .skip(1)
        .findFirst()
        .orElse(null);
```

## Reverse Sort

```java
List<Integer> result = list.stream()
        .sorted((a, b) -> b - a)
        .toList();
```

## Partition Even and Odd

```java
Map<Boolean, List<Integer>> map = list.stream()
        .collect(Collectors.partitioningBy(n -> n % 2 == 0));
```

## Group Employees by Department

```java
Map<String, List<Employee>> result = employees.stream()
        .collect(Collectors.groupingBy(Employee::getDepartment));
```

## Highest Salary Employee

```java
Employee emp = employees.stream()
        .max(Comparator.comparing(Employee::getSalary))
        .orElse(null);
```

## Average Salary

```java
double avg = employees.stream()
        .collect(
            Collectors.averagingDouble(Employee::getSalary)
        );
```

## Total Salary

```java
double total = employees.stream()
        .mapToDouble(Employee::getSalary)
        .sum();
```

## Sort Employees by Salary

```java
List<Employee> result = employees.stream()
        .sorted(Comparator.comparing(Employee::getSalary))
        .toList();
```

## Sort Employees by Salary Descending

```java
List<Employee> result = employees.stream()
        .sorted(
            Comparator.comparing(Employee::getSalary)
            .reversed()
        )
        .toList();
```

## Convert List to Map

```java
Map<Integer, Employee> map = employees.stream()
        .collect(Collectors.toMap(
                Employee::getId,
                Function.identity()));
```

## Flatten Nested Lists

```java
List<Integer> result = nestedList.stream()
        .flatMap(List::stream)
        .toList();
```

## Find First Non-Repeated Character

```java
Character c = str.chars()
        .mapToObj(ch -> (char) ch)
        .collect(Collectors.groupingBy(
                Function.identity(),
                LinkedHashMap::new,
                Collectors.counting()))
        .entrySet()
        .stream()
        .filter(e -> e.getValue() == 1)
        .map(Map.Entry::getKey)
        .findFirst()
        .orElse(null);
```

## Find Common Elements Between Two Lists

```java
List<Integer> common = list1.stream()
        .filter(list2::contains)
        .toList();
```

## Merge Two Lists

```java
List<Integer> merged = 
    Stream.concat(list1.stream(), list2.stream()).toList();
```

## Remove Null Values

```java
List<String> result = list.stream()
        .filter(Objects::nonNull)
        .toList();
```

## Count Occurrences of Each Character

```java
Map<Character, Long> map = str.chars()
        .mapToObj(c -> (char) c)
        .collect(Collectors.groupingBy(
                Function.identity(),
                Collectors.counting()));
```

## Find Least Frequent Element

```java
Map.Entry<Integer, Long> result = list.stream()
        .collect(Collectors.groupingBy(
                Function.identity(),
                Collectors.counting()))
        .entrySet()
        .stream()
        .min(Map.Entry.comparingByValue())
        .orElse(null);
```

## Handle Duplicate Keys in `toMap`

```java
Map<String, Employee> result = employees.stream()
        .collect(Collectors.toMap(
                Employee::getDepartment,
                Function.identity(),
                (e1, e2) -> e1));
```

## Create Immutable Set

```java
Set<String> result = employees.stream()
        .map(Employee::getDepartment)
        .collect(Collectors.collectingAndThen(
                Collectors.toSet(),
                Set::copyOf)); // List::copyOf for List
```

## Find Highest Salary Among High Earners

```java
Map<Boolean, Optional<Employee>> result = employees.stream()
        .collect(Collectors.partitioningBy(
                e -> e.getSalary() > 50000,
                Collectors.maxBy(
                        Comparator.comparing(Employee::getSalary))));
```

## Get Distinct Departments

```java
Set<String> departments = employees.stream()
        .collect(Collectors.mapping(
                Employee::getDepartment,
                Collectors.toSet()));
```

## Join Employee Names Department-wise

```java
Map<String, String> result = employees.stream()
        .collect(Collectors.groupingBy(
                Employee::getDepartment,
                Collectors.mapping(
                        Employee::getName,
                        Collectors.joining(", "))));
```

## Salary Statistics by Department

```java
Map<String, DoubleSummaryStatistics> result = employees.stream()
        .collect(Collectors.groupingBy(
                Employee::getDepartment,
                Collectors.summarizingDouble(Employee::getSalary)));
```

## Find Lowest Paid Employee in Each Department

```java
Map<String, Optional<Employee>> result = employees.stream()
        .collect(Collectors.groupingBy(
                Employee::getDepartment,
                Collectors.minBy( // maxBy to get Highest Paid Employee
                        Comparator.comparing(Employee::getSalary))));
```

## Average Salary by Department

```java
Map<String, Double> result = employees.stream()
        .collect(Collectors.groupingBy(
                Employee::getDepartment,
                Collectors.averagingDouble(Employee::getSalary)));
```