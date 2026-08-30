---
title: "Java - Set - HashSet TreeSet LinkedHashSet"
phase: "Phase 1 - Java Core and OOP"
language: "java"
tags:
  - backend
  - java
  - oop
  - collections
  - set
  - hashset
  - treeset
  - linkedhashset
status: "not-started"
---

# Java - Set - HashSet TreeSet LinkedHashSet

> [!abstract] Overview
> The `Set` interface represents a collection that contains no duplicate elements. It models the mathematical concept of a set and is the go-to data structure whenever you need to enforce uniqueness, perform membership testing, or eliminate duplicates from a collection. The three primary implementations are `HashSet` (backed by a hash table, O(1) operations, no ordering), `LinkedHashSet` (hash table plus a linked list, O(1) operations, maintains insertion order), and `TreeSet` (backed by a red-black tree, O(log n) operations, maintains sorted order). In backend development, sets are used for deduplication, permission checks, tag management, caching unique identifiers, and implementing mathematical set operations like union, intersection, and difference.

---

## Prerequisites

> [!info] Before reading this note, you should understand:
> - [[Java - Collections Framework Overview]]
> - [[Java - List - ArrayList LinkedList]]
> - [[Java - Abstraction - Abstract Classes and Interfaces]]
> - [[Java - Encapsulation - Getters Setters Access Modifiers]]

---

## Theory

### The `Set` Interface

The `Set` interface extends `Collection` and adds one critical constraint: **no duplicate elements**. Formally, a set contains no pair of elements `e1` and `e2` such that `e1.equals(e2)`. This means:

1. **No duplicates**: Adding an element that already exists in the set has no effect. The `add()` method returns `false` to indicate that the set was not modified.
2. **At most one null**: Most set implementations allow at most one `null` element (because `null.equals(null)` would be true).
3. **No positional access**: Unlike `List`, sets do not support `get(index)` because sets have no concept of position. You cannot ask "what is the third element?" because the ordering depends on the implementation.

**Core `Set` methods:**

| Method | Description | Return |
|--------|-------------|--------|
| `add(E e)` | Adds the element if not already present | `boolean` (false if duplicate) |
| `remove(Object o)` | Removes the element if present | `boolean` |
| `contains(Object o)` | Checks if the element exists | `boolean` |
| `size()` | Returns the number of elements | `int` |
| `isEmpty()` | Returns true if the set is empty | `boolean` |
| `clear()` | Removes all elements | `void` |
| `addAll(Collection)` | Union: adds all elements from another collection | `boolean` |
| `removeAll(Collection)` | Difference: removes all elements found in another collection | `boolean` |
| `retainAll(Collection)` | Intersection: keeps only elements found in another collection | `boolean` |

The last three methods (`addAll`, `removeAll`, `retainAll`) correspond directly to mathematical set operations, which makes sets the natural choice for any logic involving uniqueness or membership.

### `HashSet` Internals

`HashSet` is backed by a `HashMap`. When you add an element to a `HashSet`, it is stored as a **key** in the internal `HashMap` with a dummy value (`PRESENT`, a static final `Object`). This means all the performance characteristics of `HashSet` are inherited from `HashMap`.

**How hashing works:**

1. **Hash code computation**: When you call `set.add(element)`, the set calls `element.hashCode()` to compute an integer hash code. This hash code is then processed (XOR-shifted) to reduce collisions.

2. **Bucket selection**: The processed hash code is mapped to a bucket index using `hash & (capacity - 1)`. The capacity is always a power of 2, so this bitwise AND operation is equivalent to `hash % capacity` but much faster.

3. **Collision handling**: If two elements hash to the same bucket, they are stored in a linked list (or a balanced tree if the list grows beyond 8 elements, a Java 8 optimization called "treeification"). The set traverses the list and uses `equals()` to check if the element already exists.

4. **Resizing**: When the number of elements exceeds `capacity * loadFactor` (default load factor is 0.75), the hash table doubles in size and all elements are rehashed into new buckets. This is an O(n) operation but happens infrequently.

**Memory layout:**

```text
HashSet<String> set = new HashSet<>(4);
set.add("Dhaka");
set.add("Sylhet");
set.add("Rajshahi");

Internal HashMap:
  capacity = 4, loadFactor = 0.75, size = 3
  threshold = 3 (resize when size > 3)

  Bucket 0: null
  Bucket 1: ["Rajshahi" -> PRESENT] -> null
  Bucket 2: ["Dhaka" -> PRESENT] -> ["Sylhet" -> PRESENT] -> null  (collision!)
  Bucket 3: null
```

**Performance:**

| Operation | Average | Worst Case |
|-----------|---------|------------|
| `add()` | O(1) | O(n) (all elements in one bucket) |
| `remove()` | O(1) | O(n) |
| `contains()` | O(1) | O(n) |

The worst case occurs when all elements hash to the same bucket, degenerating the hash table into a linked list. This is extremely unlikely with a good `hashCode()` implementation but can be triggered deliberately in a hash collision attack (a real security concern in backend systems).

**Critical requirement: `hashCode()` and `equals()` contract:**

For `HashSet` to work correctly, the elements stored in it must obey the following contract:

1. If `a.equals(b)` is true, then `a.hashCode() == b.hashCode()` must also be true.
2. If `a.hashCode() == b.hashCode()`, then `a.equals(b)` is NOT necessarily true (hash collisions are allowed).
3. `hashCode()` must return the same value for the same object as long as the fields used in `equals()` have not changed.

If you violate this contract (e.g., by overriding `equals()` without overriding `hashCode()`), the `HashSet` will behave incorrectly: it may store duplicate elements, fail to find existing elements, or lose elements after resizing.

### `LinkedHashSet` Internals

`LinkedHashSet` extends `HashSet` and adds a **doubly-linked list** that maintains the insertion order of elements. Internally, it uses a `LinkedHashMap` instead of a `HashMap`.

**How it works:**

- Each entry in the hash table also has `before` and `after` pointers that link it to the previously and subsequently inserted entries.
- Iteration follows the linked list, so elements are returned in the order they were inserted.
- The hash table still provides O(1) `add()`, `remove()`, and `contains()`.
- The linked list adds a small memory overhead (two extra references per entry) but does not affect the time complexity of any operation.

**When to use `LinkedHashSet`:**

- When you need uniqueness AND insertion order. For example, maintaining a list of unique tags in the order they were added.
- When you need predictable iteration order for testing or debugging.
- When you are building a cache that needs to evict the oldest entry (LRU cache pattern, though `LinkedHashMap` is more commonly used for this).

### `TreeSet` Internals

`TreeSet` is backed by a `TreeMap`, which uses a **red-black tree** (a self-balancing binary search tree). Elements are stored in sorted order according to their natural ordering (defined by `Comparable`) or a custom `Comparator`.

**How it works:**

1. **Insertion**: When you add an element, the tree compares it with existing nodes using `compareTo()` (or the `Comparator`) and places it in the correct position to maintain the sorted order. The tree then rebalances itself to ensure that the height remains O(log n).

2. **Search**: The tree traverses from the root, comparing the target with each node and going left or right based on the comparison. This takes O(log n) time because the tree is balanced.

3. **Deletion**: The tree finds the node, removes it, and rebalances. O(log n).

4. **Range queries**: `TreeSet` supports efficient range operations like `subSet()`, `headSet()`, and `tailSet()`, which return views of the elements within a specified range. This is impossible with `HashSet`.

**Performance:**

| Operation | Time |
|-----------|------|
| `add()` | O(log n) |
| `remove()` | O(log n) |
| `contains()` | O(log n) |
| `first()` / `last()` | O(log n) |
| `subSet()` / `headSet()` / `tailSet()` | O(log n) to create the view |

**Critical requirement: `Comparable` or `Comparator`:**

Elements in a `TreeSet` must be mutually comparable. Either the element class implements `Comparable<T>` (natural ordering), or you provide a `Comparator<T>` when creating the `TreeSet`. If elements are not comparable, a `ClassCastException` is thrown at runtime.

```java
import java.util.Comparator;
import java.util.Set;
import java.util.TreeSet;

// Natural ordering (String implements Comparable<String>)
Set<String> sorted = new TreeSet<>();
sorted.add("Banana");
sorted.add("Apple");
sorted.add("Cherry");
// Iteration order: Apple, Banana, Cherry

// Custom comparator (reverse order)
Set<String> reversed = new TreeSet<>(Comparator.reverseOrder());
reversed.add("Banana");
reversed.add("Apple");
reversed.add("Cherry");
// Iteration order: Cherry, Banana, Apple
```

### Comparison of All Three Implementations

| Feature | HashSet | LinkedHashSet | TreeSet |
|---------|---------|---------------|---------|
| Backing structure | HashMap | LinkedHashMap | TreeMap (red-black tree) |
| Ordering | None (unpredictable) | Insertion order | Sorted (natural or custom) |
| `add()` | O(1) | O(1) | O(log n) |
| `remove()` | O(1) | O(1) | O(log n) |
| `contains()` | O(1) | O(1) | O(log n) |
| Null elements | One null allowed | One null allowed | No nulls (throws NPE) |
| Memory overhead | Low | Medium (linked list pointers) | High (tree node overhead) |
| Range queries | No | No | Yes (`subSet`, `headSet`, `tailSet`) |
| Use case | Fast membership testing | Unique + ordered | Sorted unique elements |

**Decision guide:**

- Need fast lookups and do not care about order? Use `HashSet`.
- Need fast lookups and insertion order? Use `LinkedHashSet`.
- Need sorted order or range queries? Use `TreeSet`.
- Need thread safety? Use `ConcurrentHashMap.newKeySet()` (for hash-based) or `Collections.synchronizedSet(new TreeSet<>())` (for sorted).

### How Sets Determine Equality

This is the most important concept for using sets correctly. When you call `set.add(element)`, the set determines whether the element is a duplicate by:

1. **For `HashSet` and `LinkedHashSet`**: First comparing hash codes (`hashCode()`). If the hash codes differ, the elements are definitely different. If the hash codes are the same, it calls `equals()` to confirm. This two-step process makes the lookup O(1) on average.

2. **For `TreeSet`**: Using `compareTo()` (or the `Comparator`). If `compareTo()` returns 0, the elements are considered equal. `TreeSet` does NOT use `equals()` or `hashCode()`. This means a `TreeSet` can consider two objects equal even if `equals()` returns false, as long as `compareTo()` returns 0. This inconsistency can cause subtle bugs.

> [!tip] Key Insight
> The most common bug with sets in backend development is storing mutable objects whose `hashCode()` or `compareTo()` fields change after insertion. If you add an `Order` object to a `HashSet` and then modify its `orderId` field (which is used in `hashCode()`), the set will no longer be able to find the object. The object is still in the set, but it is in the wrong bucket. This is why immutable objects work best as set elements, and why you should never modify fields that participate in `hashCode()` or `equals()` after adding an object to a set.

---

## Syntax and Basic Examples

### Example 1: HashSet basics

```java
import java.util.*;

public class HashSetDemo {
    public static void main(String[] args) {
        Set<String> tags = new HashSet<>();

        // Adding elements
        System.out.println("Add 'java': " + tags.add("java"));       // true (new)
        System.out.println("Add 'spring': " + tags.add("spring"));   // true (new)
        System.out.println("Add 'backend': " + tags.add("backend")); // true (new)
        System.out.println("Add 'java': " + tags.add("java"));       // false (duplicate!)

        System.out.println("Tags: " + tags);        // [java, backend, spring] (order unpredictable)
        System.out.println("Size: " + tags.size()); // 3 (duplicate not counted)

        // Membership testing (O(1))
        System.out.println("Contains 'java': " + tags.contains("java"));     // true
        System.out.println("Contains 'python': " + tags.contains("python")); // false

        // Removing
        tags.remove("backend");
        System.out.println("After remove: " + tags);  // [java, spring]

        // Null handling
        tags.add(null);
        System.out.println("Contains null: " + tags.contains(null));  // true
        tags.add(null);  // Second null is a duplicate, ignored

        // Iteration (order is unpredictable)
        System.out.println("\nIterating:");
        for (String tag : tags) {
            System.out.println("  " + tag);
        }
    }
}
```

### Example 2: LinkedHashSet preserves insertion order

```java
import java.util.*;

public class LinkedHashSetDemo {
    public static void main(String[] args) {
        // LinkedHashSet: unique elements in insertion order
        Set<String> visitedPages = new LinkedHashSet<>();
        visitedPages.add("/home");
        visitedPages.add("/products");
        visitedPages.add("/cart");
        visitedPages.add("/products");  // Duplicate, ignored
        visitedPages.add("/checkout");
        visitedPages.add("/home");      // Duplicate, ignored

        System.out.println("Browsing history:");
        for (String page : visitedPages) {
            System.out.println("  " + page);
        }
        // Output (insertion order preserved):
        //   /home
        //   /products
        //   /cart
        //   /checkout
    }
}
```

### Example 3: TreeSet with natural and custom ordering

```java
import java.util.*;

public class TreeSetDemo {
    public static void main(String[] args) {
        // Natural ordering (String implements Comparable)
        Set<String> sortedNames = new TreeSet<>();
        sortedNames.add("Saad");
        sortedNames.add("Rahim");
        sortedNames.add("Karim");
        sortedNames.add("Arif");
        sortedNames.add("Nila");

        System.out.println("Sorted names: " + sortedNames);
        // [Arif, Karim, Nila, Rahim, Saad]

        // First and last
        System.out.println("First: " + ((TreeSet<String>) sortedNames).first());  // Arif
        System.out.println("Last: " + ((TreeSet<String>) sortedNames).last());    // Saad

        // Range queries
        TreeSet<String> tree = (TreeSet<String>) sortedNames;
        System.out.println("SubSet (K-R): " + tree.subSet("Karim", "Saad"));
        // [Karim, Nila, Rahim] (from inclusive, to exclusive)

        System.out.println("HeadSet (before N): " + tree.headSet("Nila"));
        // [Arif, Karim]

        System.out.println("TailSet (from N): " + tree.tailSet("Nila"));
        // [Nila, Rahim, Saad]

        // Custom comparator: reverse alphabetical
        Set<String> reverseNames = new TreeSet<>(Comparator.reverseOrder());
        reverseNames.addAll(sortedNames);
        System.out.println("\nReverse: " + reverseNames);
        // [Saad, Rahim, Nila, Karim, Arif]

        // Custom comparator: by string length, then alphabetical
        Set<String> byLength = new TreeSet<>(
            Comparator.comparingInt(String::length).thenComparing(Comparator.naturalOrder())
        );
        byLength.addAll(List.of("Saad", "Rahim", "Karim", "Arif", "Nila", "Abdullah"));
        System.out.println("By length: " + byLength);
        // [Saad, Arif, Nila, Rahim, Karim, Abdullah]

        // TreeSet does NOT allow null
        try {
            sortedNames.add(null);  // NullPointerException!
        } catch (NullPointerException e) {
            System.out.println("\nTreeSet does not allow null elements");
        }
    }
}
```

### Example 4: Set operations (union, intersection, difference)

```java
import java.util.*;

public class SetOperations {
    public static void main(String[] args) {
        Set<String> javaSkills = new HashSet<>(Set.of("Spring", "Hibernate", "Maven", "JPA"));
        Set<String> pythonSkills = new HashSet<>(Set.of("Django", "FastAPI", "Pandas", "Spring"));

        System.out.println("Java skills: " + javaSkills);
        System.out.println("Python skills: " + pythonSkills);

        // Union: all skills from both sets
        Set<String> union = new HashSet<>(javaSkills);
        union.addAll(pythonSkills);
        System.out.println("\nUnion (all skills): " + union);
        // [Spring, Hibernate, Maven, JPA, Django, FastAPI, Pandas]

        // Intersection: skills common to both
        Set<String> intersection = new HashSet<>(javaSkills);
        intersection.retainAll(pythonSkills);
        System.out.println("Intersection (common): " + intersection);
        // [Spring]

        // Difference: Java skills not in Python
        Set<String> javaOnly = new HashSet<>(javaSkills);
        javaOnly.removeAll(pythonSkills);
        System.out.println("Java only: " + javaOnly);
        // [Hibernate, Maven, JPA]

        // Symmetric difference: skills in one but not both
        Set<String> symmetricDiff = new HashSet<>(javaSkills);
        symmetricDiff.addAll(pythonSkills);
        Set<String> common = new HashSet<>(javaSkills);
        common.retainAll(pythonSkills);
        symmetricDiff.removeAll(common);
        System.out.println("Symmetric difference: " + symmetricDiff);
        // [Hibernate, Maven, JPA, Django, FastAPI, Pandas]
    }
}
```

### Example 5: Custom objects in sets (hashCode and equals)

```java
import java.util.*;

public class CustomObjectSet {

    // A class with properly overridden equals() and hashCode()
    static class Product {
        private final long id;
        private final String name;
        private double price;  // Mutable, but NOT used in equals/hashCode

        Product(long id, String name, double price) {
            this.id = id;
            this.name = name;
            this.price = price;
        }

        // Two products are equal if they have the same ID.
        // Price changes do not affect equality.
        @Override
        public boolean equals(Object o) {
            if (this == o) return true;
            if (!(o instanceof Product p)) return false;
            return id == p.id;
        }

        // hashCode must be consistent with equals.
        // Since equals uses only 'id', hashCode must use only 'id'.
        @Override
        public int hashCode() {
            return Long.hashCode(id);
        }

        @Override
        public String toString() {
            return name + " ($" + price + ")";
        }
    }

    public static void main(String[] args) {
        Set<Product> catalog = new HashSet<>();

        Product laptop = new Product(1, "Laptop", 85000);
        Product mouse = new Product(2, "Mouse", 1500);
        Product laptopDuplicate = new Product(1, "Laptop Pro", 90000);
        // Same ID as laptop, different name and price.
        // equals() says they are the same product.

        catalog.add(laptop);
        catalog.add(mouse);
        catalog.add(laptopDuplicate);  // Duplicate! Not added.

        System.out.println("Catalog size: " + catalog.size());  // 2
        System.out.println("Catalog: " + catalog);

        // Changing the price does not affect set membership
        // because price is not used in hashCode/equals.
        laptop.price = 79999;
        System.out.println("Contains laptop: " + catalog.contains(laptop));  // true
    }
}
```

---

## Real Backend Code

> [!example] How this appears in actual backend projects
> Sets are used extensively in backend systems for deduplication, authorization, and data processing. Here are three realistic scenarios.

### Scenario 1: Role-based access control with sets

In Spring Security, user permissions and roles are stored as sets for fast membership testing.

```java
package com.company.orderservice.security;

import org.springframework.stereotype.Service;
import java.util.Collections;
import java.util.HashSet;
import java.util.Set;

@Service
public class AuthorizationService {

    // Roles and permissions stored as sets for O(1) lookup
    private static final Set<String> ADMIN_PERMISSIONS = Set.of(
        "ORDER_CREATE", "ORDER_READ", "ORDER_UPDATE", "ORDER_DELETE",
        "USER_CREATE", "USER_READ", "USER_UPDATE", "USER_DELETE",
        "REPORT_VIEW", "SETTINGS_MANAGE"
    );

    private static final Set<String> MANAGER_PERMISSIONS = Set.of(
        "ORDER_CREATE", "ORDER_READ", "ORDER_UPDATE",
        "USER_READ", "REPORT_VIEW"
    );

    private static final Set<String> CUSTOMER_PERMISSIONS = Set.of(
        "ORDER_CREATE", "ORDER_READ"
    );

    public boolean hasPermission(String userRole, String requiredPermission) {
        Set<String> permissions = getPermissionsForRole(userRole);
        return permissions.contains(requiredPermission);  // O(1) lookup
    }

    public Set<String> getEffectivePermissions(String userRole, Set<String> extraPermissions) {
        Set<String> basePermissions = new HashSet<>(getPermissionsForRole(userRole));
        basePermissions.addAll(extraPermissions);  // Union with extra permissions
        return Collections.unmodifiableSet(basePermissions);
    }

    public boolean hasAllPermissions(String userRole, Set<String> requiredPermissions) {
        Set<String> userPermissions = getPermissionsForRole(userRole);
        return userPermissions.containsAll(requiredPermissions);  // Checks if userPermissions is a superset
    }

    private Set<String> getPermissionsForRole(String role) {
        return switch (role.toUpperCase()) {
            case "ADMIN" -> ADMIN_PERMISSIONS;
            case "MANAGER" -> MANAGER_PERMISSIONS;
            case "CUSTOMER" -> CUSTOMER_PERMISSIONS;
            default -> Set.of();  // Empty immutable set for unknown roles
        };
    }
}
```

**What to notice:**

- `Set.of()` creates immutable sets for constant permission definitions. These sets are created once at class loading and shared across all threads safely.
- `permissions.contains(requiredPermission)` is O(1) with `HashSet`, making permission checks extremely fast even with thousands of permissions.
- `containsAll()` checks if one set is a superset of another. This is a single method call that replaces a loop with individual `contains()` checks.
- The `getEffectivePermissions()` method uses `addAll()` to compute the union of base permissions and extra permissions. This is the mathematical set union operation applied to authorization.

### Scenario 2: Deduplication in data processing

Backend systems frequently receive data with duplicates that must be eliminated before processing.

```java
package com.company.orderservice.service;

import org.springframework.stereotype.Service;
import java.util.HashSet;
import java.util.LinkedHashSet;
import java.util.List;
import java.util.Set;

@Service
public class EmailNotificationService {

    private final UserRepository userRepository;
    private final EmailClient emailClient;

    public EmailNotificationService(UserRepository userRepository, EmailClient emailClient) {
        this.userRepository = userRepository;
        this.emailClient = emailClient;
    }

    public int sendPromotionalEmail(String campaignId, List<Long> userIds) {
        // Step 1: Remove duplicate user IDs using a Set
        // A user might appear multiple times in the list due to
        // overlapping segments in the marketing database.
        Set<Long> uniqueUserIds = new LinkedHashSet<>(userIds);
        // LinkedHashSet preserves the original order for consistent processing.

        if (uniqueUserIds.size() < userIds.size()) {
            // logger.info("Removed {} duplicate user IDs from campaign {}",
            //     userIds.size() - uniqueUserIds.size(), campaignId);
        }

        // Step 2: Exclude users who have already received this campaign
        Set<Long> alreadySent = getAlreadySentUserIds(campaignId);
        Set<Long> toSend = new HashSet<>(uniqueUserIds);
        toSend.removeAll(alreadySent);  // Set difference: unique minus already sent

        // Step 3: Send emails
        int sentCount = 0;
        for (Long userId : toSend) {
            try {
                User user = userRepository.findById(userId).orElse(null);
                if (user != null && user.isActive() && user.isEmailOptIn()) {
                    emailClient.send(user.getEmail(), campaignId);
                    sentCount++;
                }
            } catch (Exception e) {
                // logger.error("Failed to send email to user {}", userId, e);
            }
        }

        return sentCount;
    }

    private Set<Long> getAlreadySentUserIds(String campaignId) {
        // Returns a Set of user IDs who already received this campaign.
        // Using a Set here makes the removeAll() operation O(n) instead of O(n*m).
        List<CampaignLog> logs = campaignLogRepository.findByCampaignId(campaignId);
        Set<Long> sentIds = new HashSet<>();
        for (CampaignLog log : logs) {
            sentIds.add(log.getUserId());
        }
        return sentIds;
    }
}
```

**What to notice:**

- `new LinkedHashSet<>(userIds)` eliminates duplicates from the input list in O(n) time while preserving the original order. This is the most efficient deduplication technique in Java.
- `toSend.removeAll(alreadySent)` computes the set difference in O(n) time. If `alreadySent` were a `List` instead of a `Set`, this operation would be O(n * m), which could take minutes for large campaigns.
- The combination of `LinkedHashSet` for deduplication and `HashSet` for membership testing is a common pattern in backend data processing pipelines.

### Scenario 3: Tag management with TreeSet for sorted display

```java
package com.company.orderservice.model;

import jakarta.persistence.CollectionTable;
import jakarta.persistence.Column;
import jakarta.persistence.ElementCollection;
import jakarta.persistence.Entity;
import jakarta.persistence.FetchType;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;
import jakarta.persistence.JoinColumn;
import jakarta.persistence.Table;
import org.hibernate.annotations.SortNatural;
import java.util.Collections;
import java.util.HashSet;
import java.util.Set;
import java.util.SortedSet;
import java.util.TreeSet;

@Entity
@Table(name = "products")
public class Product {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    // Tags stored as a sorted set for consistent ordering in API responses.
    // The @ElementCollection annotation tells JPA to store the tags
    // in a separate table (product_tags) with a foreign key to the product.
    @ElementCollection(fetch = FetchType.EAGER)
    @CollectionTable(name = "product_tags", joinColumns = @JoinColumn(name = "product_id"))
    @Column(name = "tag")
    @SortNatural  // Tells Hibernate to use a TreeSet with natural ordering
    private SortedSet<String> tags = new TreeSet<>();

    public void addTag(String tag) {
        if (tag == null || tag.isBlank()) {
            throw new IllegalArgumentException("Tag cannot be empty");
        }
        this.tags.add(tag.toLowerCase().strip());
        // TreeSet automatically prevents duplicates and maintains sorted order.
    }

    public void removeTag(String tag) {
        this.tags.remove(tag.toLowerCase().strip());
    }

    public Set<String> getTags() {
        return Collections.unmodifiableSet(tags);
        // Return an unmodifiable view to prevent external modification.
    }

    public boolean hasTag(String tag) {
        return tags.contains(tag.toLowerCase().strip());  // O(log n) for TreeSet
    }

    public boolean hasAnyTag(Set<String> requiredTags) {
        // Check if the product has at least one of the required tags.
        // This is a set intersection check.
        Set<String> intersection = new HashSet<>(tags);
        intersection.retainAll(requiredTags);
        return !intersection.isEmpty();
    }

    // Getters and setters for id, name...
}
```

**What to notice:**

- `TreeSet` (via `SortedSet`) maintains tags in alphabetical order automatically. When the API returns the product's tags, they are always sorted without any explicit sorting step.
- `addTag()` normalizes the tag to lowercase before adding. Since `TreeSet` uses `compareTo()` for equality, "Java" and "java" would be considered the same tag and only one would be stored.
- `hasAnyTag()` computes the intersection of the product's tags and the required tags. If the intersection is non-empty, the product matches at least one required tag. This is a common pattern for tag-based filtering in e-commerce backends.

---

## Common Mistakes

> [!warning] Mistakes beginners make with this concept

### Mistake 1: Not overriding `hashCode()` when overriding `equals()`

**Wrong:**

```java
public class User {
    private String email;

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof User u)) return false;
        return email.equals(u.email);
    }

    // hashCode() is NOT overridden! The default Object.hashCode() uses the
    // memory address, so two User objects with the same email will have
    // different hash codes.

    public static void main(String[] args) {
        Set<User> users = new HashSet<>();
        users.add(new User("saad@example.com"));
        users.add(new User("saad@example.com"));  // Should be a duplicate, but...

        System.out.println("Size: " + users.size());  // 2! Both users are in the set!
        // HashSet first checks hashCode(). The two objects have different hash codes
        // (different memory addresses), so HashSet assumes they are different
        // and never calls equals().
    }
}
```

**Right:**

```java
public class User {
    private String email;

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof User u)) return false;
        return email.equals(u.email);
    }

    @Override
    public int hashCode() {
        return email.hashCode();  // Consistent with equals(): uses the same field
    }
}
```

**Why it is wrong:** The `HashSet` contract requires that equal objects have equal hash codes. If you override `equals()` without overriding `hashCode()`, the set will use the default `Object.hashCode()` (based on memory address), which is different for every object. The set will place equal objects in different buckets and never detect the duplicate. This is the single most common set-related bug in Java.

### Mistake 2: Modifying an object's hash code fields after adding it to a set

**Wrong:**

```java
Set<Product> catalog = new HashSet<>();
Product laptop = new Product(1, "Laptop", 85000);
catalog.add(laptop);

// Modifying the field used in hashCode() AFTER adding to the set
laptop.setId(2);  // id is used in hashCode()!

System.out.println("Contains laptop: " + catalog.contains(laptop));  // false!
// The set looks in the bucket for hash code 2, but the object is in the bucket
// for hash code 1. The object is lost inside the set.

catalog.remove(laptop);  // Returns false! Cannot remove it either.
// The set is now in a corrupted state: it contains an object it cannot find.
```

**Right:**

```java
// Option 1: Use immutable objects as set elements (best practice)
public final class Product {
    private final long id;  // final: cannot change after construction
    // ...
}

// Option 2: Do not modify hash code fields after adding to the set
Product laptop = new Product(1, "Laptop", 85000);
catalog.add(laptop);
// Do NOT call laptop.setId(2) while laptop is in the set.
// Remove it first, modify it, then re-add it.
catalog.remove(laptop);
laptop.setId(2);
catalog.add(laptop);
```

**Why it is wrong:** When you add an object to a `HashSet`, the set computes its hash code and places it in the corresponding bucket. If you later change a field that participates in `hashCode()`, the object's hash code changes, but the object remains in the old bucket. The set can no longer find it because it looks in the new bucket. This creates a "ghost" element that cannot be found, removed, or detected by `contains()`.

### Mistake 3: Using `TreeSet` with inconsistent `compareTo()` and `equals()`

**Wrong:**

```java
public class Order implements Comparable<Order> {
    private Long id;
    private BigDecimal total;

    @Override
    public int compareTo(Order other) {
        return this.total.compareTo(other.total);  // Sort by total amount
    }

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof Order other)) return false;
        return this.id.equals(other.id);  // Equal by ID
    }

    @Override
    public int hashCode() {
        return id.hashCode();
    }
}

// Problem: two orders with the same total but different IDs
Set<Order> orders = new TreeSet<>();
orders.add(new Order(1L, new BigDecimal("100")));
orders.add(new Order(2L, new BigDecimal("100")));  // compareTo returns 0!
// TreeSet considers them equal (compareTo == 0) and does NOT add the second one.
// But equals() says they are different objects (different IDs).
System.out.println("Size: " + orders.size());  // 1! Order 2 was silently dropped.
```

**Right:**

```java
@Override
public int compareTo(Order other) {
    int totalComparison = this.total.compareTo(other.total);
    if (totalComparison != 0) return totalComparison;
    return this.id.compareTo(other.id);  // Break ties with ID to ensure consistency
}
```

**Why it is wrong:** `TreeSet` uses `compareTo()` (not `equals()`) to determine equality. If `compareTo()` returns 0 for two objects that `equals()` considers different, the `TreeSet` will silently drop one of them. The general contract is that `compareTo()` should be **consistent with equals**: `a.compareTo(b) == 0` if and only if `a.equals(b)`. When sorting by a non-unique field, always add a tiebreaker (like the ID) to ensure that `compareTo()` returns 0 only for truly equal objects.

### Mistake 4: Using a `List` when a `Set` is more appropriate

**Wrong:**

```java
// Checking if a user has a specific permission using a List
List<String> permissions = getUserPermissions(userId);  // 500 permissions
if (permissions.contains("ORDER_DELETE")) {  // O(n) search through 500 elements
    deleteOrder(orderId);
}
```

**Right:**

```java
// Using a Set for O(1) membership testing
Set<String> permissions = new HashSet<>(getUserPermissions(userId));
if (permissions.contains("ORDER_DELETE")) {  // O(1) lookup
    deleteOrder(orderId);
}
```

**Why it is wrong:** `List.contains()` is O(n). `Set.contains()` is O(1) for `HashSet`. When you are checking membership (does this element exist in the collection?), a `Set` is the correct data structure. Using a `List` for membership testing is one of the most common performance mistakes in backend code, especially when the check happens inside a loop.

---

## Key Takeaways

> [!tip] Remember these points
> 1. The `Set` interface guarantees **no duplicate elements**. It is the correct data structure for uniqueness enforcement, membership testing, and mathematical set operations (union, intersection, difference).
> 2. **`HashSet`** provides O(1) `add()`, `remove()`, and `contains()` but has no ordering. It is the fastest set implementation and the default choice for most backend use cases. It requires correctly implemented `hashCode()` and `equals()` on the element type.
> 3. **`LinkedHashSet`** provides O(1) operations AND maintains insertion order. Use it when you need both uniqueness and predictable iteration order (e.g., deduplication while preserving the original sequence).
> 4. **`TreeSet`** provides O(log n) operations and maintains sorted order. Use it when you need sorted elements or range queries (`subSet`, `headSet`, `tailSet`). Elements must implement `Comparable` or you must provide a `Comparator`. `TreeSet` does not allow `null` elements.
> 5. The **`hashCode()`/`equals()` contract** is critical for `HashSet` and `LinkedHashSet`: equal objects must have equal hash codes. Never modify fields that participate in `hashCode()` after adding an object to a set. For `TreeSet`, ensure that `compareTo()` is consistent with `equals()` to avoid silently dropping elements.

---

## Exercise

> [!abstract] Practice before moving to the next note

### Exercise 1: Set Operations on Student Courses (Easy)

Create two sets of course codes: one for a CSE student and one for an EEE student. Use set operations to find:

1. All unique courses taken by either student (union).
2. Courses taken by both students (intersection).
3. Courses taken only by the CSE student (difference).
4. Courses taken by exactly one student (symmetric difference).

Print all results with descriptive labels.

> **Hint:** Use `addAll()` for union, `retainAll()` for intersection, and `removeAll()` for difference. For symmetric difference, compute the union minus the intersection.

### Exercise 2: Custom Object in HashSet (Medium)

Create a `Student` class with fields `studentId` (String), `name` (String), and `cgpa` (double). Override `equals()` and `hashCode()` so that two students are considered equal if they have the same `studentId` (ignore name and CGPA). Then:

1. Create a `HashSet<Student>` and add 5 students.
2. Try to add a student with the same ID but different name and CGPA. Verify that the set rejects the duplicate.
3. Verify that `contains()` works correctly with a newly created `Student` object that has the same ID.
4. Change the `hashCode()` implementation to use `name` instead of `studentId` and observe how the set breaks.

> **Hint:** Use `Objects.hash(studentId)` for the hash code. The broken version demonstrates why the `hashCode()`/`equals()` contract is critical.

### Exercise 3: TreeSet with Custom Comparator (Medium)

Create an `Order` record with fields `orderId` (long), `totalAmount` (double), and `createdAt` (LocalDateTime). Create a `TreeSet<Order>` with a custom comparator that sorts orders by `totalAmount` descending, then by `createdAt` ascending (for ties). Add 8-10 orders with varying amounts and dates. Then:

1. Print all orders in sorted order.
2. Use `first()` and `last()` to find the highest and lowest orders.
3. Use `headSet()` to find all orders above 5000 BDT.
4. Use `subSet()` to find all orders between 1000 and 5000 BDT.

> **Hint:** The comparator should use `Comparator.comparingDouble(Order::totalAmount).reversed().thenComparing(Order::createdAt)`. Be careful with the tiebreaker: if two orders have the same amount AND the same timestamp, add `orderId` as a final tiebreaker to prevent the TreeSet from treating them as duplicates.

### Exercise 4: Duplicate Detection in a Log File (Hard, Optional)

Simulate a log processing system that detects duplicate error messages:

1. Create a list of 50 log entries (strings) with some duplicates. Include timestamps, log levels, and messages.
2. Use a `LinkedHashSet` to extract unique log messages (ignoring timestamps) while preserving the order of first occurrence.
3. Use a `HashMap<String, Integer>` to count the frequency of each unique message.
4. Use a `TreeSet` to display the unique messages sorted alphabetically.
5. Print a report showing: total entries, unique entries, most frequent message, and the sorted unique messages.

> **Hint:** To ignore timestamps, extract the message part of each log entry using `substring()` or `split()` before adding to the set. For the frequency count, use `map.merge(message, 1, Integer::sum)`.

<details>
<summary>Solution for Exercise 1</summary>

```java
import java.util.*;

public class SetOperationsDemo {
    public static void main(String[] args) {
        Set<String> cseCourses = new HashSet<>(Set.of("CSE101", "CSE201", "CSE301", "MATH101", "ENG101"));
        Set<String> eeeCourses = new HashSet<>(Set.of("EEE101", "EEE201", "CSE101", "MATH101", "PHY101"));

        // Union
        Set<String> union = new HashSet<>(cseCourses);
        union.addAll(eeeCourses);
        System.out.println("Union: " + union);

        // Intersection
        Set<String> intersection = new HashSet<>(cseCourses);
        intersection.retainAll(eeeCourses);
        System.out.println("Intersection: " + intersection);

        // Difference (CSE only)
        Set<String> cseOnly = new HashSet<>(cseCourses);
        cseOnly.removeAll(eeeCourses);
        System.out.println("CSE only: " + cseOnly);

        // Symmetric difference
        Set<String> symDiff = new HashSet<>(union);
        symDiff.removeAll(intersection);
        System.out.println("Symmetric difference: " + symDiff);
    }
}
```

</details>

<details>
<summary>Solution for Exercise 3</summary>

```java
import java.time.LocalDateTime;
import java.util.*;

record Order(long orderId, double totalAmount, LocalDateTime createdAt) {}

public class Main {
    public static void main(String[] args) {
        Comparator<Order> comparator = Comparator
            .comparingDouble(Order::totalAmount).reversed()
            .thenComparing(Order::createdAt)
            .thenComparingLong(Order::orderId);  // Final tiebreaker to prevent lost entries

        TreeSet<Order> orders = new TreeSet<>(comparator);
        orders.add(new Order(1, 5000, LocalDateTime.of(2025, 7, 1, 10, 0)));
        orders.add(new Order(2, 12000, LocalDateTime.of(2025, 7, 2, 14, 30)));
        orders.add(new Order(3, 3500, LocalDateTime.of(2025, 7, 1, 9, 0)));
        orders.add(new Order(4, 8000, LocalDateTime.of(2025, 7, 3, 11, 0)));
        orders.add(new Order(5, 5000, LocalDateTime.of(2025, 7, 2, 8, 0)));
        orders.add(new Order(6, 1500, LocalDateTime.of(2025, 7, 4, 16, 0)));
        orders.add(new Order(7, 25000, LocalDateTime.of(2025, 7, 1, 12, 0)));
        orders.add(new Order(8, 8000, LocalDateTime.of(2025, 7, 5, 10, 0)));

        System.out.println("All orders (sorted):");
        orders.forEach(o -> System.out.printf("  #%d: %.2f BDT%n", o.orderId(), o.totalAmount()));

        System.out.println("\nHighest: #" + orders.first().orderId());
        System.out.println("Lowest: #" + orders.last().orderId());
    }
}
```

</details>

---

## Related Notes

- [[Java - List - ArrayList LinkedList]]
- [[Java - Map - HashMap TreeMap LinkedHashMap]] (next note)
- [[Java - Comparable and Comparator]]
- [[Java - Java 8 Streams API]]

---

## Resources

- [Oracle Java Tutorials: The Set Interface](https://docs.oracle.com/javase/tutorial/collections/interfaces/set.html) - Official documentation covering the Set contract and operations.
- [Oracle Java Documentation: HashSet](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/HashSet.html) - Complete API reference.
- [Baeldung: Java Set Guide](https://www.baeldung.com/java-set-operations) - Comprehensive guide covering all Set implementations and operations.
- [Baeldung: HashSet vs TreeSet vs LinkedHashSet](https://www.baeldung.com/java-hashset-vs-treeset-vs-linkedhashset) - Detailed comparison with performance benchmarks.
- [Baeldung: Java hashCode and equals](https://www.baeldung.com/java-equals-hashcode-contracts) - Essential reading on the hashCode/equals contract. Read this before using any hash-based collection.
- [Effective Java by Joshua Bloch - Item 11: Always Override hashCode When You Override equals](https://www.oreilly.com/library/view/effective-java/9780134686097/) - The definitive explanation of why the hashCode/equals contract matters and how to implement it correctly.
