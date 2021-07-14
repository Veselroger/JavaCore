# [←](./README.md) <a id="home"></a> JVM, Bytecode and JIT

## Table of Contents:
- [Bytecode](#bytecode)
- [Binding](#binding)
- [JIT](#jit)
- [Resources](#resources)

----

## [↑](#home) <a id="bytecode"></a> Bytecode
Исходный код Java программ хранится в файлах с расширением *.java. Например, можно взять классический "[Hello World](https://docs.oracle.com/javase/tutorial/getStarted/application/index.html)":
```java
class HelloWorldApp {
    public static void main(String[] args) {
        System.out.println("Hello World!");
    }
}
```
Такой код можно в любом редакторе (например, в "[VS Code](https://code.visualstudio.com)") сохранить в файл **HelloWorldApp.java**. Чтобы код можно было выполнить он должен быть скомпилирован. Для этого в JDK есть утилита **javac**, он же **java compiler**.

Вызов ``javac HelloWorldApp.java`` скомпилирует java код в файл с расширением ***.class**. Данный файл хранит не исходный код, а так называемый **байт-код**. Чтобы его увидеть можно воспользоваться дизассемблером **javap**.

Если вызвать javap без аргументов, то мы увидим сигнатуры методов из кода. Например, если выполнить ``javap HelloWorldApp.class`` то мы увидим:
```
Compiled from "HelloWorldApp.java"
class HelloWorldApp {
  HelloWorldApp();
  public static void main(java.lang.String[]);
}
```
Если после javap добавить флаг **-v** (v = verbose, т.е. "подробнее"), то мы увидим уже байт-код, в который был скомпилирован наш java код.

Утилита javap может быть полезна в том числе и в том случае, если нужно узнать, например, есть ли нужные методы внутри **jar** архива.

Например, мы можем создать jar (всопним как в "[Creating a JAR File](https://docs.oracle.com/javase/tutorial/deployment/jar/build.html)") при помощи команды ``jar cf test.jar HelloWorldApp.class`` и теперь мы можем посмотреть доступные методы при помощи ``javap -classpath test.jar HelloWorldApp``.

Подробнее про байт-код можно прочитать в статье "[Java Bytecode: Using Objects and Calling Methods](https://www.jrebel.com/blog/using-objects-and-calling-methods-in-java-bytecode)".


## [↑](#home) <a id="binding"></a> Binding
В байткоде в местах вызова методов можно увидеть, что в Java существуют разные типы вызовов. Все они начинаются с **invoke**, на заканчиваются по-разному. На эту тему есть отличный материал: **"[Methods invocation inside of a Java Virtual Machine](https://www.lohika.com/methods-invocation-inside-of-a-java-virtual-machine)"**.

В Java есть так называемые "связывания", когда вызов метода связывается непосредственно с телом (т.е. с кодом) метода, который нужно выполнить. Связывания бывают двух видов:
- **Раннее связывание** выполняется на этапе компиляции (Compile time).
- **Позднее связывание** выполняется в момент выполнения (Runtime).

Благодаря позднему связыванию в Java работает такой принцип ООП, как полиморфизм. Ведь он именно про то, что поведение (т.е. вызов конкретного метода) меняется от того, на каком объекте он вызван.

Если кратко подитожить, то:
> All non-final non-private, not-static methods in Java are virtual


## [↑](#home) <a id="jit"></a> JIT
Чтобы выполнить Java код ему нужно окружение. Этим окружением является так называемая **Виртуальная Java машина** или **Java Virtual Machine** (**JVM**).

JVM контролирует выполнение кода и собирает различные метрики, связанные с выполнением кода. Например, JVM знает, какие участки кода выполняются гораздо чаще, чем другие. Такие участки кода называют **"горячими"**. Кроме того, JVM умеет выполнять большое количество оптимизаций, связанных с исполнением кода.

Изначально весь байт-код выполняется **интепритатором**.
**Интерпритатор** - это часть JVM, которая выполняет байт-код инструкция за инструкцией, а точнее выполняет машинные инструкции, которые соответствуют этому байт-коду. Но понятно, что это не очень оптимально.

Говоря о таких горячих методах стоит помнить про такой механизм, как **JIT компилятор**, который такие горячие участки для оптимизации сразу компилирует в машинные инструкции для ускорения. Компиляцию горячих участков можно увидеть, передав при запуске JVM флаг ``-XX:+PrintCompilation``.

Скомпилированный код лежит в специальной отдельной области памяти, которая называется **Codecache**. Подробнее можно прочитать в статье "[Java Codecache](http://blog.andresteingress.com/2016/10/19/java-codecache)".

На тему JIT компилятора есть хороший доклад: **"[JIT компилятор в JVM глазами Java программиста](https://www.youtube.com/watch?v=Pc-aB5U7RBI)"**.

Кроме того, кроме JIT в Java мире существует Ahead-Of-Time Compilation. Подробнее можно посмотреть в докладе "[Никита Липский — Ahead-of-time компиляция](https://www.youtube.com/watch?v=KbbSGg-PK70)".



## [↑](#home) <a id="resources"></a> Resources
Дополнительные ресурсы про байт-код и компиляторы:
- [Введение в JIT компиляцию](http://www.polovko.me/blog/2012/10/04/vvedenie-v-jit-kompilyaciyu/)
- [JIT-компилятор оптимизирует не круто, а очень круто](https://habr.com/ru/post/305894/)
- [JVM Internals](https://blog.jamesdbloom.com/JVMInternals.html)
- [Ahead Of Time VS Just In Time in Java](https://medium.com/@sauravomar01/ahead-of-time-vs-just-in-time-in-java-8456f5a77e00)
- [OpenJDK wiki: InterfaceCalls](https://wiki.openjdk.java.net/display/HotSpot/InterfaceCalls)
- [Methods invocation inside of a Java Virtual Machine](https://www.lohika.com/methods-invocation-inside-of-a-java-virtual-machine)
- [Invoke Dynamic](https://www.baeldung.com/java-invoke-dynamic)
- [Hexlet: Байт-код Java](https://ru.hexlet.io/courses/bytecode)

