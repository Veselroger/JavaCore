# [←](../README.md) <a id="home"></a> Java Generics

## Table of Contents:
- [Java Generics](#generics)
- [Covariance](#сovariance)
- [Wildcards](#wildcards)
- [Generic classes](#class)
- [Generic methods](#methods)

----

## [↑](#home) <a id="generics"></a> Generics
**[Generics](https://docs.oracle.com/javase/tutorial/java/generics/why.html)** is a special mechanism introduced in Java 5.0.\
It enables stronger type checking at compile time, allows reuse of the same code but with different inputs, and reduces the number of explicit casts in code.

Before Generics, we would write it like this:
```java
List numbers = new ArrayList();
numbers.add(2);
numbers.add("2");
System.out.println((Integer) numbers.get(0) + 2);
```
The problem with this code is that it uses an explicit cast, which can lead to a runtime error.\
The zeroth element will work fine in this case, but the first element will fail.

Starting with Java 5, this type usage was called **Raw Types**, and qualified types were called **Generic Types**:
```java
List<Integer> numbers = new ArrayList<Integer>();
numbers.add(2);
System.out.println(numbers.get(0) + 2);
```

In addition, the generics mechanism uses the **"[type inference](https://docs.oracle.com/javase/tutorial/java/generics/genTypeInference.html)"** mechanism, also known as Type Inference.\
Thanks to this, generics do not need to specify the type if the compiler can infer it automatically (from the passed parameters or the variable type).\
This specification is called the **"Diamond operator"**:
```java
List<Integer> numbers = new ArrayList<>();
```

Since generics didn't exist previously and compatibility with legacy code is required, type erasure (**[Type Erasure](https://docs.oracle.com/javase/tutorial/java/generics/erasure.html)**) occurs when compiling Java code to bytecode.\
This process replaces generics with types that define bounds. If such bounds are not specified, the bounds will be defined by the Object type.\
For example:

![](../img/generics/erasure.png)

It's also worth noting that, although type erasure does occur, some information is still available at runtime in some cases.\
For example, for anonymous classes:
```java
public static void main(String[] args) {
	List<String> list = new ArrayList<String>() {};
	ParameterizedType t = (ParameterizedType) list.getClass().getGenericSuperclass();
	System.out.println("Generic:" + t.getActualTypeArguments()[0]);
}
```

Thanks to this fact, frameworks like Spring can use this information for their own purposes.\
You can read more in the articles "[Spring Framework 4.0 and Java Generics](https://spring.io/blog/2013/12/03/spring-framework-4-0-and-java-generics)", which explains that Spring uses this information to do the following:
```java
@Autowired
private Store<String> s1;
```

Generics can be specified for methods, as well as classes and interfaces.

----

## [↑](#home) <a id="covariance"></a> Ковариантность и инвариантность
There are two approaches to working with types: **Covariance** and **invariance**.

Initially, Java chose the covariance approach. This is how arrays were constructed:
```java
String[] strings = new String[2];
Object[] objects = strings;
objects[0] = 12;
```
Line 2 contains no errors, since the Object type is the parent of String.\
From the compiler's perspective, line 3 is correct, since the objects array accepts any objects, so you can put an Integer there.\
However, in reality, our array must contain strings.\
As a result, everything will fail on line 3 with a **java.lang.ArrayStoreException** error.\
The worst part is that this error won't be caught at compile time, but will occur somewhere and at some point in the running application.

Since generics were designed for stricter typing, generics are invariant.\
This means that generics do not preserve hierarchy.\
For example, the following line simply won't compile:
```java
List<Number> numbers = new ArrayList<Integer>();
```

The only problem is that the generics mechanism breaks when mixing generics and raw types.\
This situation is called **"[Heap Pollution](https://itsobes.ru/JavaSobes/chto-takoe-heap-pollution/)"**:
```java
List<Integer> numbers = new ArrayList<Integer>();
List rawList = numbers;
rawList.add("test");
System.out.println(numbers.get(0) + 2);
```
This code will fail at runtime because For the compiler, the last line does not contain an error.

----

## [↑](#home) <a id="wildcards"></a> Wildcards
**[Wildcards](https://docs.oracle.com/javase/tutorial/java/generics/wildcards.html)** is a special mechanism for indicating that a type is unknown.\
According to Oracle's Java Tutorial, a wildcard is a question mark (?), which implies that an unknown type is specified where it is used.

A good explanation is given in the Oracle guide: **"[Generics: Wildcards](https://docs.oracle.com/javase/tutorial/extra/generics/wildcards.html)"**.\
As mentioned above, generics are invariant, and therefore a problem arises with the following method:
```java
static void printCollection(Collection<Object> c) {
	for (Object e : c) {
		System.out.println(e);
	}
}
```
That is, you can't call the method: ``printCollection(new ArrayList<Integer>());``\
But you can use a wildcard and change the method signature:
```java
static void printCollection(Collection<?> c) ​​{
```

Furthermore, it is possible to specify an upper bound:
```java
static int sum(Collection<? extends Number> c) {
	int sum = 0;
	for (Number e : c) {
		sum = sum + e.intValue();
	}
	return sum;
}
```

It's worth remembering that by specifying extends, we lose the ability to add values:
```java
static void addZero(Collection<? extends Number> c) {
	c.add(0);
}
```
As you might guess, there's no guarantee of what kind of Number collection we're receiving.\
Therefore, we can't add anything without adding an Integer to some Double.

If a wildcard is used in a consumer definition (i.e., where we want to add a value), we must use ``<? super Number>``:
```java
static void addZero(Collection<? super Number> c) {
	c.add(0);
	c.add(0.0);
}
```

Therefore, when using wildcards, remember about **PECS**.
**PECS** (**Producer Extends Consumer Super**) is a rule that states that if the typed object is a data source (i.e., we receive data from it),\
then **extends** is used in the generic, and if the object is a data consumer (i.e., we add data to it), then **super** is used.

The **PECS** rule is best illustrated with an example:
```java
public void addFirst(List<? extends T> source) {
	this.entry = source.get(0);
}
```
In this case, **source** acts as a producer and therefore must specify **extends**.\
Specifying **super** here will result in a compilation error.

The topic of wildcards in generics is well covered in the lecture by **"[Tagir Valeev - Generics](https://www.youtube.com/watch?v=usiKCn7SwxI&t=2234s)"**.

----

## [↑](#home) <a id="class"></a> Generic classes
Generics can be used when defining a class:
```java
private static class Box<T> {
	private T entry;
	public Box(){}
	public Box(T entry) { this.entry = entry; }
	public void add(T obj) { this.entry = obj; }
	public T get() { return this.entry; }
}
```

Thanks to the generic, the compiler can infer the type:
```java
new Box<>("string").get().length();
```
This is possible because since we received a string in the constructor, the type will be string.

Additionally, generics in class definitions can be qualified with the **extends** keyword, for example:
```java
public static class Box<T extends Number> {
```

This will only allow us to create Box instances with types that inherit from Number:
```java
Box<Integer> t = new Box<>();
t.add(1);
```

Furthermore, generics can specify which interfaces must be implemented in addition to the primary requirement.\
For example:
```java
private static class Box <T extends Number & Comparable<T>> {
```

----

## [↑](#home) <a id="methods"></a> Generic methods
Generics can be used not only in class declarations but also in method declarations: **"[Generic Methods](https://docs.oracle.com/javase/tutorial/java/generics/methods.html)"**.

A generic is declared last in a method, but before its first use.\
Since a generic can be used as a method result, generics are declared BEFORE the return type for methods.

For example:
```java
public static <K extends Number> Box<K> create(K obj) {
	return new Box<K>(obj);
}
```
This method will attempt to infer K from the argument specified as the method parameter.\
However, if inference fails (for example, ``Box.create(null)``), the bound will be set to the specified bound, which in this case is Number.

To specify a type in a method, you must specify the generic BEFORE the method:
```java
Box.<Integer>create(null)
```

----