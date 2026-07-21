# Java Collections Framework Hierarchy

```text
Iterable
│
└── Collection
    │
    ├── List
    │   ├── ArrayList
    │   ├── LinkedList
    │   ├── Vector
    │   │   └── Stack
    │   └── CopyOnWriteArrayList (Concurrent)
    │
    ├── Set
    │   ├── HashSet
    │   │   └── LinkedHashSet
    │   ├── SortedSet
    │   │   └── NavigableSet
    │   │       └── TreeSet
    │   ├── EnumSet
    │   └── CopyOnWriteArraySet (Concurrent)
    │
    └── Queue
        ├── PriorityQueue
        ├── Deque
        │   ├── ArrayDeque
        │   └── LinkedList
        └── BlockingQueue (Concurrent)
            ├── ArrayBlockingQueue
            ├── LinkedBlockingQueue
            ├── PriorityBlockingQueue
            ├── DelayQueue
            └── SynchronousQueue


Map (Not part of Collection)
│
├── HashMap
│   └── LinkedHashMap
├── SortedMap
│   └── NavigableMap
│       └── TreeMap
├── Hashtable
│
├── WeakHashMap
├── IdentityHashMap
├── EnumMap
└── ConcurrentHashMap
```

# Most Used Implementations

| Interface | Common Implementations |
|-----------|-------------------------|
| List | ArrayList, LinkedList |
| Set | HashSet, LinkedHashSet, TreeSet |
| Queue | PriorityQueue, ArrayDeque, LinkedList |
| Deque | ArrayDeque, LinkedList |
| Map | HashMap, LinkedHashMap, TreeMap, ConcurrentHashMap |


# When to Use

| Collection | Best Use Case |
|------------|---------------|
| ArrayList | Fast random access, frequent reads |
| LinkedList | Frequent insertions/deletions in the middle |
| HashSet | Fast uniqueness lookup |
| LinkedHashSet | Unique elements while preserving insertion order |
| TreeSet | Sorted unique elements |
| PriorityQueue | Processing by priority (min-heap by default) |
| ArrayDeque | Stack or Queue replacement |
| HashMap | Fast key-value lookup |
| LinkedHashMap | Predictable insertion/access order |
| TreeMap | Sorted key-value storage |
| ConcurrentHashMap | Thread-safe, high-performance concurrent access |


# Time Complexity (Most Asked)

| Collection | Get | Add | Remove | Search |
|------------|-----|-----|--------|--------|
| ArrayList | O(1) | O(1)* | O(n) | O(n) |
| LinkedList | O(n) | O(1) | O(1) | O(n) |
| HashSet | - | O(1) | O(1) | O(1) |
| TreeSet | - | O(log n) | O(log n) | O(log n) |
| HashMap | O(1) | O(1) | O(1) | O(1) |
| TreeMap | O(log n) | O(log n) | O(log n) | O(log n) |
| PriorityQueue | Peek: O(1) | O(log n) | O(log n) | O(n) |
| ArrayDeque | O(1) | O(1) | O(1) | O(n) |

> **Note:** `ArrayList` append is **amortized O(1)**; resizing occasionally takes **O(n)**.

# Fail-Fast vs Fail-Safe Iterator (Java)

| Feature | Fail-Fast Iterator | Fail-Safe Iterator |
|---------|--------------------|--------------------|
| Behavior | Throws `ConcurrentModificationException` if the collection is structurally modified during iteration | Iterates over a copy (or snapshot), so no exception is thrown |
| Underlying Mechanism | Uses `modCount` to detect structural modifications | Uses a cloned/snapshot collection |
| Reflects Changes | ❌ No | ❌ No (iterates over the snapshot) |
| Thread Safe | ❌ No | ✅ Yes (designed for concurrent use) |
| Performance | Faster, less memory overhead | Slower, extra memory for snapshot |
| Package | `java.util` | `java.util.concurrent` |


## Fail-Fast Example

```java
List<String> list = new ArrayList<>();
list.add("A");
list.add("B");
list.add("C");

for (String s : list) {
    if (s.equals("B")) {
        list.remove(s);   // ConcurrentModificationException
    }
}
```

### Correct Way Using Iterator

```java
Iterator<String> iterator = list.iterator();

while (iterator.hasNext()) {
    String s = iterator.next();

    if (s.equals("B")) {
        iterator.remove();   // Safe
    }
}
```

## Fail-Safe Example (`CopyOnWriteArrayList`)

```java
CopyOnWriteArrayList<String> list = new CopyOnWriteArrayList<>();

list.add("A");
list.add("B");
list.add("C");

for (String s : list) {
    if (s.equals("B")) {
        list.remove(s);   // No exception
    }
}

System.out.println(list);
```

**Output**

```
[A, C]
```

The iterator continues over the original snapshot, even though the list has been modified.


## Common Fail-Fast Collections

- `ArrayList`
- `LinkedList`
- `HashSet`
- `LinkedHashSet`
- `TreeSet`
- `HashMap`
- `LinkedHashMap`
- `TreeMap`
- `Vector` (its iterators are fail-fast)


## Common Fail-Safe Collections

- `CopyOnWriteArrayList`
- `CopyOnWriteArraySet`
- `ConcurrentHashMap`
- `ConcurrentLinkedQueue`
- `ConcurrentLinkedDeque`


## Interview Points

- **Fail-Fast**
  - Detects structural modifications using `modCount`.
  - Throws `ConcurrentModificationException` on a best-effort basis.
  - Does **not** guarantee thread safety.

- **Fail-Safe**
  - Iterates over a snapshot or otherwise tolerates concurrent modifications.
  - Does not throw `ConcurrentModificationException`.
  - Changes made after iterator creation may not be visible during that iteration.


## One-Line Difference

> **Fail-Fast**: Detects concurrent structural modifications and throws `ConcurrentModificationException`.

> **Fail-Safe**: Iterates over a snapshot (or concurrent view), allowing modifications without throwing `ConcurrentModificationException`.
