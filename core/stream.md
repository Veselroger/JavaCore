# [←](../README.md) <a id="home"></a> Stream API

## Table of Contents:
- [Anonymous classes](#anonymous)
- [Lambda](#lambda)
- [Functional Interface](#functional)
- [Method Reference](#methodref)
- [Stream](#stream)
- [Collectors](#collectors)

----

## [↑](#home) <a id="anonymous"></a> Anonymous classes
**[Anonymous classes](https://docs.oracle.com/javase/tutorial/java/javaOO/anonymousclasses.html)** are inner classes that don't have a name (as their name suggests).\
They are declared and instantiated at the same time:
```java
Comparator<Integer> myComp = new Comparator<Integer>() {
	@Override
	public int compare(Integer o1, Integer o2) {
		return Integer.compare(o2.intValue(), o1.intValue());
	}
};
```
The compiler creates a separate class file for anonymous classes.\
For example: **Generics$1.class**.

One of the features of anonymous classes is that they have their own scope.\
This means anonymous classes can override variables from outside the class.\
Furthermore, since anonymous classes are a version of an inner class, they can be accessed externally via the class name.\
For example:
```java 
public class Main {
	private int ID = 0;
	private Runnable task = new Runnable() {
		@Override
		public void run() {
			System.out.println(Main.this.ID);
		}
	};
```

Furthermore, using variables from a method that uses an anonymous class is only permitted if the variable is final or effectively final (i.e., the compiler assumes it won't be modified).

----

## [↑](#home) <a id="lambda"></a> Lambda
Starting with Java 1.8, we introduced Lambda expressions.\
**Lambda expressions** are based on **FunctionalInterface** (functional interfaces).\
These are interfaces that have ONLY ONE abstract method (i.e., a method without a body).\
The number of default methods is not counted, since these methods have a body.

Functional interfaces can be marked with the **@FunctionalInterface** annotation, which enables additional compile-time checks.\
However, if an interface is not marked but meets the requirements, it will still be considered functional.\
One such interface is the **Comparator** interface. This allows its implementation to be written like this:
```java
Comparator<Integer> myComp = (o1, o2) -> {
	return Integer.compare(o2.intValue(), o1.intValue());
};
```

Or even like this:
```java
Comparator<Integer> myComp = (o1, o2) -> Integer.compare(o2.intValue(), o1.intValue());
```

A unique feature of lambdas is that they are not compiled into a separate class, as anonymous classes are.\
Furthermore, the scope of lambdas is the same as the scope of their environment.\
This means that **this** and **super** are accessible from within a lambda, and variable shadowing does not work.

It's also worth noting that while lambdas are similar to anonymous classes, they have two other important differences:
- Lambda bytecode is generated on the fly, not by the javac compiler.\
For more information, see **"[Behind the scenes: How do lambda expressions really work in Java?](https://blogs.oracle.com/javamagazine/post/behind-the-scenes-how-do-lambda-expressions-really-work-in-java)"**
- Lambda execution is lazy, meaning it occurs when the lambda is accessed, not when it's declared.

So, **"what happens with lambda after compile time"**?\
Lambdas are created by **LambdaMetafactory** at runtime.\
So, we don't have a separate class file as for anonymous classes.

Also, an interesting detail:
```java
public class Main {

    public void test() {
        //method context is here. this in lambda means Main class instance
        Runnable task = () -> {System.out.println(this.hashCode());};
    }

    public static void main(String[] args) throws InterruptedException {
        Runnable task = () -> {
            // Can't use this in static context, because static context doesn't have this
        };
    }
}
```
lambda uses the same context where it's defined.


**[default](https://www.youtube.com/watch?v=njx4t5faYQ0)** methods in interfaces allow us to provide interfaces methods that do not require implementation because they have default implementation:
```java
@FunctionalInterface
public interface MyDemo {
    public void test();
    
    default void print() {
        System.out.println("print");
    }
}
```
They can be used **ONLY** in interfaces.

----

## [↑](#home) <a id="functional"></a> Functional Interface
A **[Functional Interface](https://www.youtube.com/watch?v=ZsiFNxiqCcw)** is an interface that has ONLY ONE abstract method (i.e., a method without a body).

It's important to remember that there are several basic functional interfaces that are actively used in the Java API.

There are consumers.\
Consumers are expressed as **Consumer**:
```java
Consumer<String> consumer = (text) -> System.out.println(text);
consumer.accept("Hello, World!");
```
As noted earlier, java expressions are lazy.\
This means that the code in the consumer will only be executed when accept is called.

There are suppliers.\
Suppliers are expressed as **Supplier**:
```java
Supplier<String> supplier = () -> "Hello, World!";
System.out.println(supplier.get());
```

There are predicates, i.e. Some condition, expressed as a **Predicate**:
```java
Predicate<String> predicate = (text) -> text.equals("test");
System.out.println(predicate.test("test"));
```

Furthermore, since we have functional interfaces, we also have functions:
```java
Function<String, Integer> func = (text) -> Integer.valueOf(text);
System.out.println(func.apply("2") + 2);
```

And there is BiFunction:
```java
BiFunction<Integer, Integer, String> func = (a, b) -> String.valueOf(Math.max(a, b));
System.out.println(func.apply(2, 4));
```

----

## [↑](#home) <a id="methodref"></a> Method Reference
It's also important to remember something called a **[method reference](https://www.youtube.com/watch?v=n1EAf759DwA)**.\
This mechanism allows the following syntax:
```java
List<Integer> collection = new ArrayList<>();
Stream.of(1, 2, 3, 7).forEach(collection::add);
```

Interestingly, the type of the method reference can be cast to is determined by the methods being called.\
For example, if a method takes arguments and does not return a result (i.e., void), the method reference will be used as a consumer, and if the method returns something, it will be used as a supplier.

----

## [↑](#home) <a id="stream"></a> Stream
The Stream API is a special API introduced in Java 1.8 that enables stream processing.\
It differs from collections in that elements can only be processed as a single stream.

Streams can be created in various ways. For example, we can do this using collection methods:
```java
List<Integer> src = Arrays.asList(1, 2, 3, 4, 5);
long result = src.stream().count();
System.out.println(result);
```

Streams work with objects, so primitives will be boxed into objects: ```Stream.of(1, 2, 3, 4)```.

There are special streams for primitives: IntStream, LongStream, DoubleStream.

It's important to understand that various operations can be performed on streams.\
These operations can be divided into two categories: **pipelined** and **terminal**.\
A terminal operation terminates the stream by returning some result.

For example, consider the following problem: "Find the sum of even elements and the number of even elements." It can be expressed as follows:
```java
Random gen = new Random();
AtomicInteger cnt = new AtomicInteger();
long result = gen.ints(0, 11)
	.limit(10)
	.filter(num -> num % 2 == 0)
	.peek(num -> cnt.incrementAndGet())
	.summaryStatistics().getSum();
```
Since non-final and non-effectively final operations cannot be used within a stream, we used AtomicInteger.

----

## [↑](#home) <a id="collectors"></a> Collectors
Collectors are a very useful feature of streams, as they allow streams to be collected into a specific collection.\
For example:
```java
List<String> collect = Stream.of("a", "b", "c")
	.collect(Collectors.toList());
```
Interestingly, in this case, we'll get "some" List, with no guarantee of the type of that list.\
That is, there's no guarantee that we'll get a **java.util.ArrayList**.

To guarantee the desired collection, you need to use a special collector:
```java
List<String> collect = Stream.of("a", "b", "c")
.collect(Collectors.toCollection(LinkedList::new));
```

Also, collectors in Map are interesting.\
In addition to the standard toMap, there is grouping in Map:
```java
Map<String, List<Integer>> map = Stream.of(1, 2, 3, 7)
.collect(Collectors.groupingBy(num -> num % 2 == 0 ? "even" : "odd"));
System.out.println(map.get("even"));
```

----