# [←](../README.md) <a id="home"></a> JVM, Bytecode and JIT

## Table of Contents:
- [Bytecode](#bytecode)
- [JVM and JIT](#jit)
- [Binding](#bindings)
- [Resources](#resources)

----

## [↑](#home) <a id="bytecode"></a> Bytecode
Исходный код программы на языке Java хранится в текстовых файлах с расширением ``*.java``.\
Его можно написать в любом редакторе. Например, можно воспользоваться **"[VS Code](https://code.visualstudio.com)"**.

Например, так выглядит исходный код классического "[Hello World](https://docs.oracle.com/javase/tutorial/getStarted/application/index.html)":
```java
class HelloWorldApp {
    public static void main(String[] args) {
        System.out.println("Hello World!");
    }
}
```

Исходный код в Java состоит из описаний классов. Каждый public класс должен хранится в одноимённом файле.\
Например, класс **HelloWorldApp** должен хранится в файле **HelloWorldApp.java**.\
В одном файле не может быть больше одного **[Top Level](https://docs.oracle.com/javase/specs/jls/se8/html/jls-8.html)** класса.

Исходный код - это просто текст. Чтобы исходный код стал программой его нужно скомпилировать при помощи компилятора Java кода.
Для этого в JDK есть утилита **[javac](https://docs.oracle.com/en/java/javase/11/tools/javac.html)** - **java compiler**:
> javac HelloWorldApp.java

Вызов данной команды скомпилирует исходный java код в файлы с расширением ***.class**.\
Class-файлы хранят не исходный код, а набор инструкций, называемых **байт-кодом**. Код каждой инструкции - один байт, отсюда и такое название формата.

Чтобы увидеть байт-код можно воспользоваться дизассемблером - утилитой **[javap](https://docs.oracle.com/en/java/javase/11/tools/javap.html)**.\
Если вызвать **javap** без аргументов, то мы увидим сигнатуры методов из кода.\
Например, если выполнить ``javap HelloWorldApp.class`` то мы увидим:
```
Compiled from "HelloWorldApp.java"
class HelloWorldApp {
  HelloWorldApp();
  public static void main(java.lang.String[]);
}
```
Если после javap добавить флаг **-v** (v = verbose, т.е. "подробнее"), то мы увидим уже байт-код, в который был скомпилирован наш java код.

Утилита javap может быть использована в том числе и на **jar** файлах.\
Например, мы можем создать jar (вспомним как в "[Creating a JAR File](https://docs.oracle.com/javase/tutorial/deployment/jar/build.html)") выполнив:
> jar cf test.jar HelloWorldApp.class

Чтобы посмотреть содержимое class-файла из jar файла выполним ``javap -classpath test.jar HelloWorldApp``.\
Это может быть полезно в том случае, если нужно узнать, есть ли нужный метод внутри **jar** архива какой-нибудь библиотеки.

Подробнее про байт-код можно прочитать в статье **"[JRebel: Java Bytecode Tutorial](https://www.jrebel.com/blog/java-bytecode-tutorial)"**.

**Байт-код** — это это набор инструкций, которые будут выполнены Виртуальной Машиной Java (**JVM**).


## [↑](#home) <a id="jit"></a> JVM and JIT
Выше было сказано, что исходный код программы на Java компилируется в байт-код, который состоит из инструкций для виртуальной Java машины. Именно **Виртуальная Java машина** или **Java Virtual Machine** (**JVM**) являвется средой выполнения Java кода. 

JVM входит в cостав **JRE** - Java Runtime Environment, она же среда выполнения Java. JRE состоит из JVM и библиотеки стандартных классов (например, пакет java.lang или java.util). До Java 9 было разделение на JRE и JDK (Java Development Kit - набор для разработки, который включает в себя JRE и утилиты вроде javac, javap и других). Начиная с Java 9 распространяется только JDK, а JRE отдельно может быть собрано при помощи **[jlink](https://docs.oracle.com/en/java/javase/11/tools/jlink.html)**.

Запуск java программы выполняется при помощи специальной утилиты **[java](https://docs.oracle.com/en/java/javase/11/tools/java.html)**, которая запускает новый процесс экземпляра виртуальной машины и запускает на ней java программу. Чтобы посмотреть список запущенных процессов виртуальных машин Java следует использовать утилиту **[jps](https://docs.oracle.com/en/java/javase/11/tools/jps.html)**.

JVM контролирует выполнение кода и собирает различные метрики, связанные с выполнением кода. Например, JVM знает, какие участки кода выполняются гораздо чаще, чем другие. Такие участки кода называют **"горячими"**. Кроме того, JVM умеет выполнять большое количество оптимизаций, связанных с исполнением кода. JVM для выполнения программы использует два инструмента: интерпритатор и JIT компилятор.

Изначально весь байт-код выполняется **интепритатором**.\
**Интерпритатор** - это часть JVM, которая выполняет байт-код инструкция за инструкцией, а точнее выполняет машинные инструкции, которые соответствуют этому байт-коду. Однако, это не оптимально и скомпилированный код выполнялся бы быстрее. Именно по этой причине когда JVM понимает, что код используется довольно часто, JVM для данного кода использует **JIT компилятор**.

**JIT компилятор** "горячие участки" для оптимизации компилирует в машинные инструкции. Компиляцию горячих участков можно увидеть, передав при запуске JVM флаг ``-XX:+PrintCompilation``. Скомпилированный код будет помещён в область памяти JVM, которая называется **Codecache**.\
Подробнее можно прочитать в статье **"[Java Codecache](http://blog.andresteingress.com/2016/10/19/java-codecache)"**.

На тему JIT компилятора есть хороший доклад: **"[JIT компилятор в JVM глазами Java программиста](https://www.youtube.com/watch?v=Pc-aB5U7RBI)"**.

Кроме того, кроме JIT в Java мире существует **Ahead-Of-Time Compilation**, когда компиляция выполняется не во время исполнения, а до запуска программы. Подробнее можно посмотреть в докладе **"[Никита Липский — Ahead-of-time компиляция](https://www.youtube.com/watch?v=KbbSGg-PK70)"**.

Про JIT ещё стоит знать, что JIT - это не просто компилятор. Т.к. у JVM есть статистика по исполнению программы, то при помощи JIT компиляции могут быть сделаны различные оптимизации, которые помогут уменьшить потребление ресурсов. Подробнее см. статьи:
- [Введение в JIT компиляцию](http://www.polovko.me/blog/2012/10/04/vvedenie-v-jit-kompilyaciyu/)
- [JIT-компилятор оптимизирует не круто, а очень круто](https://habr.com/ru/post/305894/)


## [↑](#home) <a id="binding"></a> Bindings
В байткоде в местах вызова методов можно увидеть, что в Java существуют разные типы вызовов, начинающихся со слова **invoke**.\
На эту тему есть отличные материалы: 
- **[Java Bytecode: Using Objects and Calling Methods](https://www.jrebel.com/blog/using-objects-and-calling-methods-in-java-bytecode)**
- **"[Methods invocation inside of a Java Virtual Machine](https://www.lohika.com/methods-invocation-inside-of-a-java-virtual-machine)"**.

Код методов хранится в области памяти JVM, называмой **Metaspace** (до Java 8 она называлась Permanent Generation). Но при исполнении JVM нужно будет понять, что именно этот метод нужно выполнить. Для этого JVM нужно **связать (bind)** вызов метода с его "телом".\
Связывания бывают двух видов:
- **Раннее связывание** выполняется на этапе компиляции (Compile time).
- **Позднее связывание** выполняется в момент выполнения (Runtime).

Позднее связывание использует так называемую **Virtual Method Table (VMT)** для поиска конкретного метода, который должен быть вызван.\
Благодаря позднему связыванию в Java работает такой принцип ООП, как полиморфизм. Ведь он именно про то, что поведение (т.е. вызов конкретного метода) меняется от того, на каком объекте он вызван.

Если упростить, то можно взять утверждение из **"[How does dynamic binding happens in JVM?](https://www.thetopsites.net/article/52733201.shtml)"**:
> All non-final non-private, not-static methods in Java are virtual

------------

## [↑](#home) <a id="resources"></a> Resources
Дополнительные ресурсы про теме:
- [JVM Internals](https://blog.jamesdbloom.com/JVMInternals.html)
- [Ahead Of Time VS Just In Time in Java](https://medium.com/@sauravomar01/ahead-of-time-vs-just-in-time-in-java-8456f5a77e00)
- [OpenJDK wiki: InterfaceCalls](https://wiki.openjdk.java.net/display/HotSpot/InterfaceCalls)
- [Methods invocation inside of a Java Virtual Machine](https://www.lohika.com/methods-invocation-inside-of-a-java-virtual-machine)
- [Invoke Dynamic](https://www.baeldung.com/java-invoke-dynamic)
- [Hexlet: Байт-код Java](https://ru.hexlet.io/courses/bytecode)