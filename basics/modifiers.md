# [←](../README.md) <a id="home"></a> Access modifiers

## Table of Contents:
- [Access modifiers](#modifiers)
- [Polymorphism](#polymorphism)

----

## [↑](#home) <a id="modifiers"></a> Access modifiers
Classes and class members have access modifiers to control/restrict access.

Access modifiers:
- **public**: accessible to everyone, from anywhere
- **private**: only available from the same class
- **default (package private)**: available from the same package (like private, but private for package)
- **protected**: available from the same package + accessible through inheritance to anyone

It's important that package is defined with **package** keyword on top of the class file.\
Classes can be loaded from different soure roots (like ``src/test/java`` and ``src/main/java``).\
That's why we can test package private members.


## [↑](#home) <a id="polymorphism"></a> Polymorphism
There are several interesting aspect which are worth taking into account.

The main rule: **DO NOT TIGHTEN (reduce) access**.

Because **private** is the most severe restriction, we can say that **private** members do not participate in polymorphism at all.\
Heirs can't see them and that's why they can't change them or use them somehow.

**protected** methods are visible by heirs.\
That's why they can be made **public** in the heirs of the original class.

**package private (default)** members can be made **public** or **protected**.\
But such members should be visible (i.e. can be changed only inside the same package).

This behavior allows us to maintaint the polymorphic behavior.\
For example, we are using ``Parent parent`` variable.\
If something is visible when we work with parent it should be still visible.

Access is checked by compiler and at runtime.\
But runtime check can be disabled with reflection:
```java
field.setAccessible(true);
```

Also, it's interesting that **Inner classes** can have an access to private members:
```java
public class MyClass {
    
    private void test() {
        System.out.println("test");
    }
    
    public class Inner {
        public void innerMethod() {
            test();
        }
    }
    
}
```
But it's logical because inner class is a part of the same member.\
If inner class is static (i.e. nested class) then private member will NOT be available.

----