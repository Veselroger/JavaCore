# [←](../README.md) <a id="home"></a> Immutability 

## Table of Contents:
- [Immutability](#immutability)
- [Strings](#strings)
- [Other examples](#examples)

----

## [↑](#home) <a id="immutability"></a> Immutability
**Immutability** is a very important concept.

When object state can be changed it means that object is mutable.\
Regular approach to chage object state: setters.

To make object immutable the **final** keyword can be used for fields.\
Initial values can be set by constructors.

Collections - one of the main categories of objects for which immutability may be important.\
For example, it can be quite strange/dangerous to change the user roles collections.\

There are specific util **Collections** methods to make collections immutable wrappers:
```java
List<String> collection = new ArrayList<>();
List<String> unmodifiable = Collections.unmodifiableList(collection);
```

Also, some util methods returns immutable collections.\
For example:
```java
String[] data = {"a", "b", "c"};
List<String> immutable = Arrays.asList(data);
immutable.add("d");
```
It throws ``UnsupportedOperationException`` when you try to add new elements.

But in some cases immutability can be an illusion:
```java
String[] data = {"a", "b", "c"};
List<String> immutable = Arrays.asList(data);
data[0] = "o";
System.out.println(immutable); // Returns 'o', not 'a'
```

The same thing works with **unmodifiableList**.\
Because it creates an immutable wrapper, but original object is still mutable.

Such aspects need to be taken into account when creating copies.\
Such issue is called **Shallow immutability**. The reference can be copied, not the content.\
The **deep copy** should be used to avoid issues.\
The correct deep copy approach depends on the classes structure.

Also, **[java records](https://docs.oracle.com/en/java/javase/17/language/records.html)** can be used as **immutable** objects:
```java
record Rectangle(double length, double width) { }
```

Another well-known immutable objects: Strings

----

## [↑](#home) <a id="strings"></a> Strings
Strings are immutable in Java.\
They store an array of characters and any changes (like concatenation) create a new instance.\
It can be a problem in some cases like concatenation in cycles.

To avoid issues with new instances creation the **Builder** pattern is used.\
There are two implementations: **StringBuffer** and **[StringBuilder](https://docs.oracle.com/javase/tutorial/java/data/buffers.html)**.

Mutability can be a problem in multithreaded environment.\
That's why Java provides a thread-safe **StringBuffer** and not thread-safe **StringBuilder**.\
Because thread safety adds overhead. We don't need it every time.

How builder pattern safe us? We get string as a final result only when we need.\
For example:
```java
StringBuilder sb = new StringBuilder();
for (int i = 'a'; i < 'a' + 7; i++) {
	sb.append((char)i);
}
System.out.println(sb.toString());
```

----

## [↑](#home) <a id="examples"></a> Other examples
Another immutable object example is: **wrapper types** (like Integer, Long, Double, Boolean, etc.)\
Any operations creates a new object.

Some classes provide methods for new object creation (with a new state):
```java
LocalDate d = LocalDate.now();
LocalDate d2 = d.plusDays(1);
```
or 
```java
BigDecimal a = new BigDecimal("10.0");
BigDecimal b = a.add(new BigDecimal("2.0"));
```

Another immutable object example is **[Optional](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Optional.html)**.

----