# How HashMap Works Internally in Java

`HashMap` stores data as **key-value pairs** and uses **hashing** to provide fast lookup.

Example:

```java
Map<String, Integer> map = new HashMap<>();

map.put("John", 100);
map.get("John");
```


# Internal Data Structure

A `HashMap` internally uses an **array of buckets**.

Each bucket contains:

- A linked list of nodes (in case of collisions)
- A Red-Black Tree when collisions become large (Java 8+)

```
HashMap
 |
 |-- Bucket Array
 |
 |--[0] → null
 |
 |--[1] → Node("John",100) → Node("Alex",200)
 |
 |--[2] → null
 |
 |--[3] → Node("Mike",300)
```

Each node internally looks like:

```java
static class Node<K,V> {
    final int hash;
    final K key;
    V value;
    Node<K,V> next;
}
```

# How put() Works Internally

Example:

```java
map.put("John", 100);
```

## Step 1: Calculate hashCode()

HashMap calls:

```java
key.hashCode()
```

Example:

```java
"John".hashCode()
```

Returns an integer hash value.

## Step 2: Hash Spreading

Java improves the hash distribution:

```java
hash = h ^ (h >>> 16);
```

This reduces collisions.

## Step 3: Calculate Bucket Index

HashMap calculates the bucket position:

```java
index = (n - 1) & hash;
```

Where:

- `n` = array size

Example:

```
Array size = 16

index = (16 - 1) & hash
```

The object is stored in that bucket.

## Step 4: Store Node

Example:

```
Bucket[10]

Node
 |
 |-- key = "John"
 |-- value = 100
```


# How get() Works Internally

Example:

```java
map.get("John");
```

## Step 1: Calculate Hash

HashMap again calculates:

```java
"John".hashCode()
```

## Step 2: Find Bucket

The same index is calculated:

```
Bucket[10]
```

## Step 3: Compare Keys

Inside the bucket:

```
Bucket[10]

Node("John",100)
       |
       next
       |
Node("Alex",200)
```

HashMap checks:

```java
node.hash == hash 
&& node.key.equals(key)
```

If matched:

```java
return value;
```

# Collision Handling

A collision occurs when multiple keys map to the same bucket.

Example:

```
Bucket[5]

John → Jane → null
```

Before Java 8:

```
LinkedList
```

was used.

# Java 8 Collision Handling

When a bucket contains too many entries:

```
TREEIFY_THRESHOLD = 8
```

The linked list is converted into a:

```
Red-Black Tree
```

Before:

```
Search = O(n)
```

After conversion:

```
Search = O(log n)
```

# HashMap Resizing

Default HashMap capacity:

```
16 buckets
```

Default load factor:

```
0.75
```

Resize threshold:

```
capacity * loadFactor
```

Example:

```
16 * 0.75 = 12
```

When entries exceed 12:

```
16 buckets
      |
      v
32 buckets
```

The internal array is expanded and entries are redistributed.

# Why HashMap Allows One Null Key?

HashMap supports:

```java
map.put(null, 100);
```

Because:

```java
null.hashCode()
```

cannot be called.

Java handles it specially:

```java
if(key == null)
    hash = 0;
```

The null key is stored in bucket 0.

# Why HashMap Keys Should Be Immutable

Example:

```java
map.put(employee, "John");
```

If the key changes:

```java
employee.id = 10;
```

The hash value changes.

Original location:

```
Old hash → Bucket 5
```

New lookup:

```
New hash → Bucket 12
```

The object cannot be found.

Good HashMap keys should have:

- Immutable fields
- Proper `equals()`
- Proper `hashCode()`

Examples:

- String
- Integer
- UUID

# equals() and hashCode() Relationship

HashMap uses:

1. `hashCode()` → Find bucket
2. `equals()` → Find exact key

Rule:

```java
If a.equals(b) is true

then

a.hashCode() == b.hashCode()
```

But:

```
Same hashCode does not mean equals() is true
```

# HashMap Time Complexity

| Operation | Average | Worst Case |
|-----------|---------|------------|
| put() | O(1) | O(log n) |
| get() | O(1) | O(log n) |
| remove() | O(1) | O(log n) |

---

# HashMap vs Hashtable

| Feature | HashMap | Hashtable |
|---------|---------|-----------|
| Thread safe | No | Yes |
| Allows null key | Yes | No |
| Allows null values | Yes | No |
| Performance | Faster | Slower |
| Introduced | Java 1.2 | Java 1.0 |

# Interview Summary

```
HashMap uses an array of buckets.

1. Calculate hashCode() of key.
2. Apply hash function.
3. Find bucket index.
4. Store key-value as Node.
5. Handle collisions using LinkedList.
6. Convert large buckets into Red-Black Trees.
7. Use equals() to find the exact key.
8. Resize when threshold is exceeded.
```

**One-line interview answer:**

> HashMap internally uses an array of buckets. It uses hashCode() to locate the bucket and equals() to identify the exact key. Collisions are handled using linked lists and converted to balanced trees in Java 8 when required.

# HashSet Internal Implementation in Java

`HashSet` internally uses a **HashMap** to store elements.

The elements added to a `HashSet` are stored as **keys** in the internal `HashMap`, and a dummy object called `PRESENT` is stored as the value.

## Internal Structure

Simplified implementation:

```java
public class HashSet<E> {

    private transient HashMap<E, Object> map;

    // Dummy value
    private static final Object PRESENT = new Object();

    public HashSet() {
        map = new HashMap<>();
    }

    public boolean add(E e) {
        return map.put(e, PRESENT) == null;
    }

    public boolean remove(Object o) {
        return map.remove(o) == PRESENT;
    }

    public boolean contains(Object o) {
        return map.containsKey(o);
    }
}
```

## Why Does HashSet Use a Dummy Object?

A `HashMap` stores data as:

```
Key → Value
```

But a `Set` only needs:

```
Unique Values
```

So Java uses:

```
HashSet element → HashMap key
PRESENT object → HashMap value
```

Example:

```java
Set<String> set = new HashSet<>();

set.add("Apple");
set.add("Banana");
```

Internally:

```
HashMap

Key          Value
----------------------
Apple        PRESENT
Banana       PRESENT
```

The value is never used. Only the keys matter.

## How add() Works

Example:

```java
set.add("Apple");
```

Internally:

```java
map.put("Apple", PRESENT);
```

### First insertion

HashMap does not find the key:

```
Apple → PRESENT
```

`put()` returns:

```
null
```

Therefore:

```java
add() returns true
```

### Duplicate insertion

```java
set.add("Apple");
```

HashMap finds the existing key using:

1. `hashCode()`
2. `equals()`

Existing entry:

```
Apple → PRESENT
```

The old value is returned:

```
PRESENT
```

Therefore:

```java
add() returns false
```

This is how `HashSet` prevents duplicates.

## How contains() Works

Example:

```java
set.contains("Apple");
```

Internally:

```java
map.containsKey("Apple");
```

HashMap:

1. Calculates hash using `hashCode()`
2. Finds the bucket
3. Uses `equals()` to compare keys
4. Returns true if found

## HashSet Internal Flow

```
HashSet

add("Apple")
        |
        v
HashMap.put("Apple", PRESENT)
        |
        v
hashCode()
        |
        v
Find bucket
        |
        v
Store Node
```

## Collision Handling

Since HashSet uses HashMap internally, collisions are handled the same way.

Before Java 8:

```
Bucket

Apple
 |
Orange
 |
Mango
```

A linked list was used.

Java 8+:

```
Bucket

LinkedList
     |
     |
     v
Red-Black Tree
```

When collisions exceed the threshold:

```
TREEIFY_THRESHOLD = 8
```

the linked list is converted into a balanced tree.

## Time Complexity

| Operation | Average Complexity |
|-----------|-------------------|
| add() | O(1) |
| remove() | O(1) |
| contains() | O(1) |

Worst case:

```
O(log n)
```

when a bucket becomes a tree.

## LinkedHashSet Internal Implementation

`LinkedHashSet` uses `LinkedHashMap` internally.

Structure:

```
LinkedHashSet

Element → PRESENT
```

Difference:

- Maintains insertion order
- Uses a doubly linked list internally

Example:

```java
Set<String> set = new LinkedHashSet<>();

set.add("A");
set.add("B");
set.add("C");
```

Iteration order:

```
A B C
```

## TreeSet Internal Implementation

`TreeSet` does not use HashMap.

It uses:

```
TreeMap
```

Internally:

```
TreeMap

Element → PRESENT
```

Difference:

- Maintains sorted order
- Uses Red-Black Tree

Example:

```java
Set<Integer> set = new TreeSet<>();

set.add(30);
set.add(10);
set.add(20);
```

Output:

```
10 20 30
```

## Interview Summary

> HashSet internally uses a HashMap. The elements added to HashSet become keys in the HashMap, and a static dummy object called PRESENT is stored as the value. HashSet uses HashMap's hashing mechanism, collision handling, and equals/hashCode logic to maintain uniqueness.
