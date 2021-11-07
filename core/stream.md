# [←](../README.md) <a id="home"></a> Stream API

## Table of Content:
* [Anonymous classes](#anonymous)
* [Lambda](#lambda)
* [Functional Interface](#functional)
* [Stream](#stream)
* [Collectors](#collectors)
* [Method Reference](#methodref)
* [Resources](#resources)

## [↑](#home) <a id="anonymous"></a> Anonymous classes
**[Анонимные классы](https://docs.oracle.com/javase/tutorial/java/javaOO/anonymousclasses.html)** - это внутренние классы, которые не имеют имени (что следует из их названия). Они объявляются и создаётся в одно время:
```java
Comparator<Integer> myComp = new Comparator<Integer>() {
    @Override
    public int compare(Integer o1, Integer o2) {
        return Integer.compare(o2.intValue(), o1.intValue());
    }
};
```
Компилятор создаёт отдельный файл класса для анонимных классов. Например: **Generics$1.class**.

Особенностью анонимных классов в том числе является то, что у них есть свой собственный scope. Это значит, что анонимные классы могут замещать переменные снаружи. Кроме того, так как анонимные классы являются версией внутреннего класса, то они имеют возможность обратиться к классу снаружи через название класса снаружи. Например: ``Main.super.superFieldName``.

Кроме того, использование переменных из метода, где исползуется анонимный класс, разрешено только в том случае, если эта переменная final или effectively final (т.е. компилятор понимает, что её никто не изменяет).


## [↑](#home) <a id="lambda"></a> Lambda
Начиная с Java 1.8 у нас появились Lambda выражения. Lambda выражения основаны на **FunctionalInterface** (функциональных интерфейсах). Это такие интерфейсы, у которых есть ТОЛЬКО ОДИН абстрактный метод (т.е. метод без тела). Количество default методов не учитывается, т.к. у этих методов есть тело.

Фунукциональные интерфейсы могут быть помечены аннотацией **@FunctionalInterface**, что включит доп. проверки при компиляции. Однако, если интерфейс не помечен, но удовлетворяет требованиям - он всё равно будет считаться функциональным. Одним из таких интерфейсов является интерфейс **Comparator**. Это позволяет написать его реализацию так:
```java
Comparator<Integer> myComp = (o1, o2) -> {
    return Integer.compare(o2.intValue(), o1.intValue());
};
```
Или даже так:
```java
Comparator<Integer> myComp = (o1, o2) -> Integer.compare(o2.intValue(), o1.intValue());
```

Особенностью работы лямбд является то, что лямбды не компилируются в отдельный класс, как это делают анонимные классы. Кроме того, scope лямбд такой же, как и scope их окружения. То есть из лямбды доступен **this** и **super**, а shadowing переменных не работает.

Кроме того, хотелось бы отметить, что хотя лямбды и похожи на анонимные классы, но у них есть ещё два очень важных отличия:
- байткод лямбд генерируется "на лету", а не компилятором javac\
Подробнее см. **"[Behind the scenes: How do lambda expressions really work in Java?](https://blogs.oracle.com/javamagazine/post/behind-the-scenes-how-do-lambda-expressions-really-work-in-java)"**
- исполнение лямбд "ленивое", т.е. не при объявлении, а при обращении


## [↑](#home) <a id="functional"></a> Functional Interface
**Functional Interface** - это интерфейс, у которого есть ТОЛЬКО ОДИН абстрактный метод (т.е. метод без тела).

Важно помнить, что есть несколько базовых функциональных интерфейсов, которые активно используются в Java API.

Есть потребители. Потребители выражены как **Consumer**:
```java
Consumer<String> consumer = (text) -> System.out.println(text);
consumer.accept("Hello, World!");
```
Как ранее было замечено, лябда выражения ленивые. А это значит, что код в consumer будет выполнен только при вызове accept.

Есть поставщики. Поставщики выражены как **Supplier**:
```java
Supplier<String> supplier = () -> "Hello, World!";
System.out.println(supplier.get());
```

Есть предикаты, т.е. некоторое условие, выраженное как **Predicate**:
```java
Predicate<String> predicate = (text) -> text.equals("test");
System.out.println(predicate.test("test"));
```

Кроме того, раз у нас есть функциональные интерфейсы, то есть и функции:
```java
Function<String, Integer> func = (text) -> Integer.valueOf(text);
System.out.println(func.apply("2") + 2);
```

А есть BiFunction:
```java
BiFunction<Integer, Integer, String> func = (a, b) -> String.valueOf(Math.max(a, b));
System.out.println(func.apply(2, 4));
```

## [↑](#home) <a id="stream"></a> Stream
Stream API - это специальный API, появившийся начиная с Java 1.8, который позволяет потоково обрабатывать данные. От коллекций он отличается тем, что элементы могут быть обработаны только как единый поток.

Stream'ы могут быть созданы разным способом. Например, мы можем сделать это через методы коллекций:
```java
List<Integer> src = Arrays.asList(1, 2, 3, 4, 5);
long result = src.stream().count();
System.out.println(result);
```

Стримы работают с объектами, поэтому примитивы будут boxed в объекты: ```Stream.of(1, 2, 3, 4)```.

Существуют специальные стримы для примитивов: IntStream, LongStream, DoubleStream.

Важно понимать, что над стримами можно выполнять различные операции.\
Эти операции можно разделить на 2 категории: **конвейрные** и **терминальные**. Терминальная операция завершает работу со стримом возвращая какой-то результат.

Для примера можно рассмотреть такую задачу: "Найти сумму чётных элементов и количество чётных элементов". Можно её выразить следующим образом:
```java
Random gen = new Random();
AtomicInteger cnt = new AtomicInteger();
long result = gen.ints(0, 11)
        .limit(10)
        .filter(num -> num % 2 == 0)
        .peek(num -> cnt.incrementAndGet())
        .summaryStatistics().getSum();
```
Так как внутри стрима нельзя использовать не final и не effective final, то мы использовали AtomicInteger.


## [↑](#home) <a id="collectors"></a> Collectors
Коллекторы - очень полезная возможность стримов, т.к. она позволяет стримы собирать (collect) в некоторую коллекцию. Например:
```java
List<String> collect = Stream.of("a", "b", "c")
    .collect(Collectors.toList());
```
Интересно, что в этом случае мы получим "какой-то" List, без гарантии типа этого листа. То есть нет никакой гарантии, что мы получим **java.util.ArrayList**.

Чтобы гарантированно получить нужную коллекцию нужно использовать специальный коллектор:
```java
List<String> collect = Stream.of("a", "b", "c")
    .collect(Collectors.toCollection(LinkedList::new));
```

Кроме того, интересны коллекторы в Map. Помимо обычного toMap есть группировка в Map:
```java
Map<String, List<Integer>> map = Stream.of(1, 2, 3, 7)
    .collect(Collectors.groupingBy(num -> num % 2 == 0 ? "even" : "odd"));
System.out.println(map.get("even"));
```


## [↑](#home) <a id="methodref"></a> Method Reference
Важно ещё помнить про такую вещь, как method reference. Этот механизм позволяет использовать следующий синтаксис:
```java
List<Integer> collection = new ArrayList<>();
Stream.of(1, 2, 3, 7).forEach(collection::add);
```

Интересно, что по вызываемым методам определяется к какому типу можно привести method reference. Например, если метод на вход принимает аргументы и не возвращает результат (т.е. void), то method reference будет использован в качестве consumer, а если метод возвращает что-то, то в качестве supplier.


## [↑](#home) <a id="resources"></a> Resources
Дополнительные материалы на тему Stream API:
- [Тагир Валеев : Stream API](https://www.youtube.com/watch?v=FCT2jbAs_uA)
- [Шпаргалка по Stream API](https://habr.com/ru/company/luxoft/blog/270383/)
- [Infinite streams](https://www.baeldung.com/java-inifinite-streams)
- [Вопросы на собеседование](https://jsehelper.blogspot.com/2016/05/java-8-2.html)
- [Code Formatting in IntelliJ IDEA](https://youtu.be/vjVWjocENLg?t=274)
- [15 Practical Exercises Help You Master Java Stream API](https://blog.devgenius.io/15-practical-exercises-help-you-master-java-stream-api-3f9c86b1cf82)