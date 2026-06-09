# [←](../README.md) <a id="home"></a> Object-Oriented Programming

Java is an object-oriented language based on the following OOP (or Object-Oriented Design) principles:
- Encapsulation
- Inheritance
- Abstraction
- Polymorphism

## Table of Contents:
- [Incapsulation](#incapsulation)
- [Inheritance](#inheritance)
- [Abstraction](#abstraction)
- [Polymorphism](#polymorphism)
- [Inner Classes](#inner)
- [SOLID](#solid)

----

## [↑](#home) <a id="incapsulation"></a> Encapsulation
The principle of encapsulation describes the hiding of an object's implementation and accessibility only through special public fields.

To ensure accessibility, classes are grouped into packages (**[packages](https://docs.oracle.com/javase/tutorial/java/package/packages.html)**).
If a package is not specified, the class is placed in **Unnamed Packages**, which is provided by the JVM. However, such packages cannot have subpackages, since subpackages imply the presence of a name for the parent package.

The principle of encapsulation is implemented using **Access Modifiers**:
- **private**: visible only within the current class and nowhere beyond it
- **package private** (no modifier specified): visible within the class and its package
- **protected**: visible within the class and its descendants
- **public**: visible to everyone

## [↑](#home) <a id="inheritance"></a> Inheritance
Inheritance is a very important principle.\
Classes can inherit from other classes, adopting their behavior.

Only classes that are visible to the current class and that are not marked with the **final** keyword can be inherited.

Subclasses can override their parent's methods, and their visibility can be expanded, but not narrowed:
```java
public class Example {
	@Override
	public Object clone() throws CloneNotSupportedException {
		return super.clone();
	}
}
```
Subclasses can access their immediate parent using the **super** keyword. Because the parent is a **Superclass**,\
Accessing a class from within itself is done using the **this** keyword.

When talking about inheritance, it's important to remember that when creating a subclass, the parent's static method is initialized first, then the subclass's static method. Then the parent's non-static method, then the parent's constructor, and then the subclass's non-static method. The subclass's constructor completes the process. It's also important to remember that constructors are not inherited.

There is not multi-inheritance.\
In that case the **Composition** approach can be used.\
In that case classes can be composed of others (as modules).


## [↑](#home) <a id="abstraction"></a> Abstraction
Abstraction is one of the important principles that links all other principles.

In addition to classes, Java can declare abstract classes (classes with the abstract keyword) and interfaces.
Abstract classes and interfaces can have methods without bodies. These methods must be implemented in their descendants.

According to Oracle documentation, [interfaces](https://docs.oracle.com/javase/tutorial/java/concepts/interface.html) describe the behavior of an object.

For example, Java has the **java.lang.Iterable** interface, which indicates that an iterator can be requested from such an object to traverse its contents. Since an interface is about behavior, it can only contain static variables (they are auxiliary and do not relate to the state of the object).

An interesting feature of interfaces is that they can inherit from multiple interfaces:
```java
interface MainInterface extends inter1, inter2, inter3 {
	// methods
}
```

Starting with Java 8, interfaces can create methods with a body (**default methods**) and static methods.

Starting with Java 9, interfaces can also contain private methods.

Unlike interfaces, abstract classes don't just define behavior, so they're almost like full-fledged classes, except that they can't be instantiated (since an abstract class assumes the possibility of having methods that aren't defined).

## [↑](#home) <a id="polymorphism"></a> Polymorphism
Polymorphism allows for program execution flexibility.

Polymorphism uses late binding and the principle of abstraction for its implementation.

The simplest example of polymorphism is when a method takes a type as input, and then different implementations are passed to it. For example:
```java
interface Messenger {
	public void send(String msg);
}

public void sendToMessenger(Messenger messenger) {
	messenger.send("Error");
}
```

## [↑](#home) <a id="inner"></a> Inner Classes
Inner classes are classes within classes.

The special feature of inner classes is that they fall within the scope of the outer class.

This means we can have a private inner class.

Inner classes can be static (**Nested classes**) or not (**Inner classes**).

The special feature of inner classes is that they know and see their outer class. This allows inner class methods to access the method through which they were created:
```java
public class T1 {
	private String name = "name";
	public class Messenger {
		public void send(String msg) {
		System.out.println(T1.this.name);;
	}
}
```

## [↑](#home) <a id="solid"></a> SOLID
When talking about object-oriented programming, it's worth remembering the acronym SOLID.

**Single Responsibility Principle**\
The single responsibility principle states that each class should have only one purpose. For example, an XML parser shouldn't be responsible for exporting the result.

**Open-closed principle**\
This principle states that code should be open for extension but closed for modification.\
It's a vague description, but in practice, it's easy to violate. The most obvious example is a method that performs a specific action based on the type of the argument it receives. This means that when adding a new class, a new if clause must be added.

**Liskov substitution principle**\
This principle states:
> Objects in a program should be replaceable with instances of their subtypes without changing the correctness of the program.

**Interface segregation principle**\
The single responsibility principle applied to interfaces. The idea is that there shouldn't be one giant interface for unrelated actions, but rather several interfaces describing specific aspects of a class's behavior.

**Dependency inversion principle**\
An important principle that states that lower levels should not depend on higher ones. Abstractions should not depend on details.\
If this principle is violated, changing the implementation details of lower-level modules may require changes to higher-level modules. This is a clear sign of a SOLID violation.

----