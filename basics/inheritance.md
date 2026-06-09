# [←](../README.md) <a id="home"></a> Inheritance

## Table of Contents:
- [Inheritance](#inheritance)
- [Overriding and overloading](#overriding)
- [Initialization blocks](#initblock)
- [Sealed classes](#sealed)
- [Inner vs nested classes](#nested)

----

## [↑](#home) <a id="inheritance"></a> Inheritance
**Inheritance** - one of the main principles or concepts of Object-Oriented Design.

The idea in general is quite simple - one class can be based on another class.\
It allows to reduce repetable code.\
Also, inheritance is a base for the **polymorphism** concept.

Every class in Java implicitly inherits from **java.lang.Object**.\
It provides basic behavior such as equals, hashcode, toString.\
Also, it's very important in terms of multithreading because it provides common mechanisms such as wait, notify,

In Java, there is **NO** multi-inheritance. It means that class can have ONLY ONE parent class:
```Java
public final class StringBuilder extends AbstractStringBuilder
```
As we can see, the **extends** keyword is used for inheritance.

To replace the **multi-inheritance** different approaches can be used.

The most popular one is to use **composition**.\
In that case class just contains different parts that are expressed by other classes.\
For example, **class OrderService** can have fields **AuditService** and **NotificationService**.

Also, interfaces can be used:
```
class MetricsReporter
        implements Runnable,
                   AutoCloseable,
                   Comparable<MetricsReporter> {
```

Since java version 8 it's possible to provide **default** methods in interfaces.\
Such methods have a default implementation. For example, let's take a look at Iterable#forEach:
```java
default void forEach(Consumer<? super T> action) {
	Objects.requireNonNull(action);
	for (T t : this) {
		action.accept(t);
	}
}
```

The interesting thing is that interface can inherit (i.e. extend) from multiple interfaces:
```java
public interface List<E> extends Collection<E>, Iterable<E> {
```

----

## [↑](#home) <a id="overriding"></a> Overriding and overloading
The main idea is that child class can extend, override or overload the base (super) class.

The first thing that we should consider - **constructor**.\
Child class should always call the parent constructor.

For example:
```java
public static class Parent {
	public Parent(){
		System.out.println("Parent constuctor");
	}
}

public static class Child extends Parent {
	public Child(){
		System.out.println("test");
	}
}
``` 
On Child creation we will see the Parent constructor print text.

The **super** keyword can be used to refer to the super class members.\
For example, we can call the super class constructor:
```java
public static class Child extends Parent {
	public Child(){
		super("test");
	}
}
```

The keyword **this** can be used to refer to the current class.

It's useful to use **this** and **super** when we overload methods:
```java
public static class Parent {
	public void print(String text) {
		System.out.println(text);
	}
}

public static class Child extends Parent {
	public void print(String text, String prefix) {
		this.print(prefix + text);
	}
}
```
Overload means to add method with the same name **BUT** with different **signature** (i.e. with different arguments).

Overriding fully replaces the behavior:
```java
public static class Child extends Parent {
	@Override
	public void print(String text) {
		super.print("[" + text + "]");
	}
}
```
It's interesting that in child method we can call the method version from super.\
But the Child instance users can't get an access to it.

When we are talking about **this** keyword we have an interesting syntax also:
```java 
public static class Parent {
	private int ID = 1;
	public class Part {
		public void test() {
			System.out.println(ID);
			System.out.println(Parent.this.ID);
		}
	}
}

public static void main(String[] args) {
	Parent child = new Parent();
	child.new Part().test();
}
```
We can refer to the outer class from the nested class.

----

## [↑](#home) <a id="initblock"></a> Initialization blocks
Each class can have initialization blocks.\
They can belong to the class itself (static block) or to the instance.

Initialization blocks are called **BEFORE** constructors.

Static initialization block is called **ONLY once** on class loading.\
Usually they are used to perform some expensive operations such as environment analysis.\
For example, the **Netty** framework (that is a base for the reactive Spring WebFlux) is used it for **PlatformDependent** utility class.

In old days, static init blocks were used to load JDBC drivers with ``Class.forName("org.postgresql.Driver");``.\
But now the **SPI** (**Service Provider Interface**) is used for such cases.\
The JDBC **DriverManager** has static block to use SPI to search and load JDBC drivers (with ``java.util.ServiceLoader``).

Also, initialization blocks can be non-static.\
In that case they will be executed before any constructors.\
The original idea was to provide a way to share some logic across all constructors.\
Also, non-static initialization blocks can be used for anonymous clasess.

----

## [↑](#home) <a id="sealed"></a> Sealed classes
Originally, we couldn't partially restrict the inheritance mechanism.

Since Java 17 the **sealed** keyword was introduced:
```java
public abstract sealed class Vehicle permits Car, Truck {

    protected final String registrationNumber;

    public Vehicle(String registrationNumber) {
        this.registrationNumber = registrationNumber;
    }

    public String getRegistrationNumber() {
        return registrationNumber;
    }

}
```
In that case we can list specific classes to whom we **permit** the inheritance.\

The same approach can be used for interfaces:
```java
public sealed interface Service permits Car, Truck {

    int getMaxServiceIntervalInMonths();

    default int getMaxDistanceBetweenServicesInKilometers() {
        return 100000;
    }

}
```

----

## [↑](#home) <a id="nested"></a> Inner vs nested classes
Classes can be nested into another classes.\
They can be **nested** (i.e. part of already created class) OR **inner** (do not require outer class).

The **inner class** example:
```java
public static class Parent {
	public static class Inner {
		
	}
}

public static void main(String[] args) {
	Parent.Inner instance = new Parent.Inner();
```

The **nested class** example:
```java
public static class Parent {
	public class Nested {

	}
}

public static void main(String[] args) {
	Parent.Nested instance = new Parent().new Nested();
```
As we can see, the Parent class context is needed to create a Nested class.\
Also, nested class has an access to the outer class (even to private members).

----
