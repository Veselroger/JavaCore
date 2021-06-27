# <a id="home"></a> Java Exceptions

## Table of Contents:
- [Exceptions](#exceptions)
- [Exception Handling](#handling)
- [Chained exceptions](#chain) 
- [Try with resources](#withResources) 
- [Inheritance](#inheritance)
- [UncaughtExceptionHandler](#uncaughtExceptionHandler)
- [NoClassDefFoundError](#noClassDefFoundError)
- [Resources](#resources)

----

## [↑](#home) <a id="exceptions"></a> Exceptions
В процессе выполнения программы могут возникать события, которые нарушают (**disrupts**) нормальное выполнение инструкций. Эти события в Java называются **[Exceptions](https://docs.oracle.com/javase/tutorial/essential/exceptions/definition.html)** (**исключения**).\
Как сказано в документации Oracle:
> The term exception is shorthand for the phrase "exceptional event."

Чтобы использовать исключение необходимо выполнить выражение **[throw](https://docs.oracle.com/javase/tutorial/essential/exceptions/throwing.html)**, т.е. бросить/пробросить исключение.

Всё, что может быть выброшено выражением throw наследуется от класса **Throwable**, т.е. бросаемое/выбрасываемое. Таким образом иерархия классов может выглядеть следующим образом:

![](./img/exceptions/throwable.gif)

Подробнее про эти три категории исключений можно прочитать в официальной документации Oracle: **"[The Catch or Specify Requirement](https://docs.oracle.com/javase/tutorial/essential/exceptions/catchOrDeclare.html)"**.

Исключения, которые **НЕ** Error и **НЕ** RuntimeException требуют **обязательной** обработки и называются **сhecked**. Остальные исключения могут быть обработаны, но это необязательно. Обработка исключений выполняется при помощи конструкции **try-catch**.


## [↑](#home) <a id="handling"></a> Exception Handling
Если участок кода требует особой обработки исключительных ситуаций, то такой участок кода помещается в блок **try**.\
Выполнение блока try выполняется до тех пор, пока не произойдёт исключение или до тех пор пока не выполнятся все инструкции.

Блок **try** не может быть использован сам по себе. Блок try должен быть использован с блоком **catch** и/или **finally**.

Блок **catch** ВСЕГДА выполняется после блока try. Данный блок предназначен для "отлова" исключений:
```java
public static void main(String[] args) {
    try {
        File.createTempFile("123", ".tmp");
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

Блоков catch может быть несколько. В таком случае, блоки catch должны идти от более конкретного исключения к более общему, если исключения внутри одной иерархии. Также исключения можно перечислять через символ "|":
```java
catch (IOException|SQLException ex) {
    logger.log(ex);
    throw ex;
}
```
Подробнее см. **"[Exceptions: The catch Blocks](https://docs.oracle.com/javase/tutorial/essential/exceptions/catch.html)"**.

Блок **finally** выполняется самым последним (т.е. после catch), но выполняется гарантировано, даже если в блоке catch есть return.

Использование return в блоке catch стоит избегать, т.к. выполнение return из catch при использовании переменных приведёт к тому, что изменение локальных переменных в блоке final будут проигнорировано. Например:
```java
public static int sendMessage() {
    int msg = 0;
    try {
        throw new IllegalStateException("Error");
    } catch (Exception e) {
        msg++;
        return msg;
    } finally {
        msg++;
    }
}
```
Данный код вернёт 1, хотя блок finally на самом деле выполнится.
Подробнее про блок finally см. **"[Exceptions: The finally Block](https://docs.oracle.com/javase/tutorial/essential/exceptions/finally.html)"**.

Кроме того, мы можем использовать ключевое слово **throws**, чтобы делегировать обработку исключения тому, кто нас будет вызывать:
```java
public static File createTempFile() throws IOException {
    File tmp = File.createTempFile("1", ".tmp");
    return tmp;
}
```
Подробнее см. **"[Specifying the Exceptions Thrown by a Method](https://docs.oracle.com/javase/tutorial/essential/exceptions/declaring.html)"**.


## [↑](#home) <a id="chain"></a> Chained exceptions
Иногда бывает, что приходится работать сразу с несколькими исключениями. В этом случае есть несколько вариантов.

Исключения могут быть объединены в цепочку (**chain**):
```java
public static File createTempFile() {
    try {
        File tmp = File.createTempFile("1", ".tmp");
        return tmp;
    } catch (Exception e) {
        throw new IllegalStateException("tmp file error", e);
    }
}
```

Исключение, которое используется в качестве аргумента при создании другого исключения называется **Cause** (т.е. причина). В этом случае при выводе стэктрэйса мы увидим следующее:
```
Exception in thread "main" java.lang.IllegalStateException: tmp file error
	at TestClass.createTempFile(TestClass.java:33)
	at TestClass.main(TestClass.java:38)
Caused by: java.lang.IllegalArgumentException: Prefix string "1" too short: length must be at least 3
	at java.base/java.io.File.createTempFile(File.java:2083)
	at java.base/java.io.File.createTempFile(File.java:2154)
	at TestClass.createTempFile(TestClass.java:30)
	... 1 more
```
Подробнее см. **"[Chained Exceptions](https://docs.oracle.com/javase/tutorial/essential/exceptions/chained.html)"**.


## [↑](#home) <a id="withResources"></a> Try with resources
При работе с исключениями у нас есть конструкция, которая называется **[try-with-resources](https://docs.oracle.com/javase/tutorial/essential/exceptions/tryResourceClose.html)**. Назначение данной конструкции - обрабатывать исключения при работе с ресурсами и автоматически закрывать ресурс без написания избыточного кода.

Особенностью данной конструкции является то, что блоки catch и finally в этом случае необязательны:
```java
public static String readFirstLineFromFile(String path) throws IOException {
    try (BufferedReader br = new BufferedReader(new FileReader(path))) {
        return br.readLine();
    }
}
```

В качестве ресурса может быть использован любой класс, реализующий интерфейс **java.lang.AutoCloseable**:
```java
public static void main(String[] args) {
    try (AutoCloseable closeable = () -> {throw new IllegalStateException("close");}) {
        throw new IllegalStateException("try");
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```
Тут важно запомнить, что ресурс закрывается СРАЗУ как только мы выходим из блока try по какой либо из причин. То есть закрытие ресурса происходит ПРЕЖДЕ чем выполнится даже catch.

Как видно из примера, у нас всегда может возникнуть 2 ошибки: при выполнении try и при выполнении закрытия ресурса. Ошибка, которая возникает в блоке try всегда самая важная, а это значит что ошибка, возникшая при закрытии ресурса "подавляется" и учитывается как **"supressed"**:
```
java.lang.IllegalStateException: try
	at TestClass.main(TestClass.java:49)
	Suppressed: java.lang.IllegalStateException: close
		at TestClass.lambda$main$0(TestClass.java:48)
		at TestClass.main(TestClass.java:48)
```

Интересно, что если блок try отработает, но при закрытии произойдёт исключение, то это исключение тоже будет обработано блоком catch. А это значит, что любая конструкция try-with-resources обязана иметь блок catch или метод с такой конструкцией должен быть объявлен с throws.

Кроме того стоит знать, что начиная с Java 9 блок try-with-resources умеет работать с ресурсами, которые были созданы вне блока при выполнении требования, что такие ресурсы **final** или **effective final**.


## [↑](#home) <a id="inheritance"></a> Inheritance
Говоря про наследование, стоит упоминуть что наследники могут переопределять методы:
```java
AutoCloseable acl = new AutoCloseable() {
    @Override
    public void close() throws IOException {
        System.out.println("close");
    }
};
```
Как видно из примера, наследники могут сужать тип (т.е. уточнять его). На основе этой особенности, например, старый интерфейс **Closeable** был адаптирован под механизм try-with-resources. Более подробно см. **"[Чем отличается Closeable от AutoCloseable?](https://itsobes.ru/JavaSobes/chem-otlichaetsia-closeable-ot-autocloseable/)"**.


## [↑](#home) <a id="uncaughtExceptionHandler"></a> UncaughtExceptionHandler
В Java существует возможность установить свой обработик необработанных исключений на уровен потока:
```java
Thread.UncaughtExceptionHandler handler = new Thread.UncaughtExceptionHandler() {
    @Override
    public void uncaughtException(Thread t, Throwable e) {
        e.printStackTrace();
        System.err.println("Handled");
    }
};
Thread.currentThread().setUncaughtExceptionHandler(handler);
```
Если такой обработчик не задан, то у потока, в котором получено необработанное исключение, будет получена группа и у неё будет вызван метод **[uncaughtException()](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/ThreadGroup.html#uncaughtException(java.lang.Thread,java.lang.Throwable))**.


## [↑](#home) <a id="noClassDefFoundError"></a> NoClassDefFoundError
Исключение - нормальный механизм до тех пор, пока он не мешает другим механизмам.
Например, если при статической инициализации класса произойдёт ошибка, то это закончится тем, что класс не будет загружен и при обращении к нему в дальнейшем мы получим ошибку **java.lang.NoClassDefFoundError**:
```java
public static void main(String[] args) {
    try {
        Tezt tezt = new Tezt();
    } catch (Throwable ex) {
        // save the thread - skip ex
    }
    Tezt tezt = new Tezt();
}
```
На этом примере хорошо можно понять отличие Error от Exception. 

**ClassNotFoundException** - исключение, потому что не может быть гарантии, что какой-то класс доступен, т.к. во всём мире миллиарды классов. Но если класс доступен, он обязан быть загружен. Если это не так, то мы получим **ExceptionInInitializerError**. А если потом обратимся к такому классу, то получим **NoClassDefFoundError**.


## [↑](#home) <a id="resources"></a> Resources
- [Обработка исключений в контроллерах Spring](https://habr.com/ru/post/528116/)
- [Баранцев: как в JUnit проверять ожидаемые исключения?](http://barancev.github.io/junit-catch-throwable/)
- [CompletableFuture - Exception Handling](https://www.logicbig.com/tutorials/core-java-tutorial/java-multi-threading/completion-stages-exception-handling.html)
- [Тагир Валеев: Исключения, try-catch](https://www.youtube.com/watch?v=usiKCn7SwxI)
- [Spring Validation & JSR-303](https://habr.com/ru/post/424819/)