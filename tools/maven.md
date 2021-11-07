# [←](../README.md) <a id="home"></a> Apache Maven

## Table of content:
- [Intro](#intro)
- [Life cycle](#lifecycle)
- [Maven Plugins](#plugins)
- [Packaging](#packaging)
- [Dependencies](#dependencies)
- [Dependency management](#management)
- [Dependency Scope](#scope)
- [Project Inheritance](#inheritance)
- [Maven Profiles](#profiles)
- [Maven Properties](#properties)
- [Additional Resources](#resources)


## [↑](#Home) <a name="intro"></a> Intro
**Apache Maven** - это система автоматизации сборки проектов. Каждый Maven проект описывается в отдельном файле при помощи специальной структуры, которая называется **Project Object Model**. Каждая модель сохраняется в отдельном файле с названием **pom.xml**.

Project - основная единица, с которой работает Maven. Проект определяется при помощи так называемых "**[Maven Coordinates](https://maven.apache.org/pom.html#Maven_Coordinates)**" они же **GAV**:
- **g**roupId: идентификатор (имя) группы (что-то вроде пакета или каталога)
- **a**rtifactId: идентификатор (имя) проекта
- **v**ersion: версия проекта

Кроме этого, описание любого Maven проекта обязано иметь указание того, какая версия модели Maven используется для описания проекта. Разные версии Maven могут использовать разные версии модели.\
Текущая версия модели - "4.0.0".

Таким образом для описания проекта необходимо создать файл **pom.xml**, который может выглядеть следующим образом:
```xml
<project>
	<modelVersion>4.0.0</modelVersion>
    <groupId>com.github.veselroger</groupId>
	<artifactId>maven-notes</artifactId>
	<version>1</version>
</project>
```
Это - минимальное описание Maven проекта.\
Подробнее можно найти в документации Maven в разделе **"[Minimal POM](https://maven.apache.org/guides/introduction/introduction-to-the-pom.html#minimal-pom)"**.


## [↑](#Home) <a name="lifecycle"></a> Life cycle
У любого проекта в Maven есть свой жизненный цикл (**"[Maven Lifecycle](https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html)"**). Жизненный цикл состоит из Phase.

**Phase** - это фаза или некоторый этап жизни проекта. Например, этапами жизни проекта могут быть "компиляция исходного кода проекта" или "упаковывание проекта в jar архив". То есть фаза - это некоторый этап жизни проекта с логической точки зрения.

Фазы объединены в некоторые последовательности, которые и называются жизненными циклами. По умолчанию есть 3 цикла: **default**, **clean** и **site**.\
Для выполнения любой фазы из последовательности необходимомо выполнить все фазы, которые идут в этой последовательности ДО этой фазы.

Например, жизненный цикл по умолчанию (**default lifecycle**) состоит из следующих фаз: ``validate``, ``compile``, ``test``, ``package``, ``verify``, ``install``, ``deploy``. Это означает, что при выполнении фазы **test** будут выполнены фазы начиная от **validate**.


## [↑](#Home) <a name="plugins"></a> Maven Plugins
Функциональность в Maven предоставляется благодаря плагинам. Есть встроенные плагины, а так же могут быть использованы сторонние плагины.\
Плагины описываются в файле pom.xml в блоке ``<build> ``.

У каждого плагина может быть одна или несколько целей (**Goal**). Цель - это некоторое действие, которое плагин выполняет.

Плагины могут быть вызваны как в процессе выполнения фазы жизненного цикла Maven проекта так и отдельно.

Например, мы можем вызвать goal "effective-pom" у встроенного плагина "maven-help-plugin". Мы можем сделать это двумя способами:
- через полное название плагина и название goal
``mvn org.apache.maven.plugins:maven-help-plugin:3.2.0:effective-pom``
- через короткое название плагина и название goal
``mvn help:effective-pom``

**Effective pom** - это полный pom, по которому Maven в конечном итоге будет выполнять сборку проекта. Этот pom отличается от того, что указано в **pom.xml**, потому что любой pom.xml неявным образом наследуется от так называемого **Super POM**, если не указан родительский другой родительский проект. Про наследование проектов речь пойдёт дальше, поэтому пока вернёмся к плагинам.

В этом effective pom нас интересует описание плагинов. Например:
```xml
<plugin>
	<artifactId>maven-jar-plugin</artifactId>
	<version>2.4</version>
	<executions>
		<execution>
			<id>default-jar</id>
			<phase>package</phase>
			<goals>
        		<goal>jar</goal>
        	</goals>
		</execution>
	</executions>
</plugin>
```

Таким образом, в pom в описании плагина может быть указана привязка **Goal** плагина к одной из **phase**. В данном примере, при выполнении фазы **package** будет выполнен goal "jar" плагина maven-jar-plugin. Кроме того, плагины для Maven - тоже проэкты, а следовательно определяются при помощи Maven coordinates.


## [↑](#Home) <a name="packaging"></a> Packaging
Ещу одной немаловажной частью понимания Maven является атрибут **Packaging**.

**Packaging** - это тип "запаковки" проекта или то, в каком виде ожидается конечный артефакт, который будет получен на фазе **package**. Данный атрибут влияет на то, какие плагины и какие их Goal с какими фазами будут связаны по умолчанию. Всё это будет отражено на effective pom проекта.

Стоит выделить два особенных типа паковки:
- **pom** : проект в виде метаданных, проект представляет из себя просто pom файл
- **maven-plugin** : представляет из себя проект, являющийся maven плагином

А так же стоит знать основные паковки: **jar**, **war**, **ear**. По умолчанию используется типа паковки **jar**.


## [↑](#Home) <a name="dependencies"></a> Dependencies
Одной из самых главных задач систем сборок проекта является управление зависимостями. Такие зависимости описываются в блоке ``<dependencies>`` в pom файле.

Зависимости - это другие Maven проекты, а следовательно указываются так же при помощи Maven координат. Часто официальные сайты различных проектов указывают то, как описать зависимость от их проектов. Например, база данных H2 у себя в "[H2 Database Cheat Sheet](https://www.h2database.com/html/cheatSheet.html)" говорит, как добавить H2 в Maven проект как зависимость:
```xml
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <version>1.4.200</version>
</dependency>
```

Зависимости должны быть откуда-то получены. Место, откуда могут быть получены зависимости, называется **"репозиторий"**. Более подробную информацию можно найти в руководстве **"[Introduction to Repositories](https://maven.apache.org/guides/introduction/introduction-to-repositories.html)"**.

Репозитории делятся на два типа: **local** и **remote**. Сначала Maven ищет зависимости в локальном репозитории и если не находит, то тогда скачивает зависимость из remote репозитория в локальный.

По умолчанию, Maven скачивает артифакты из центрального репозитория (**central repository**). Чтобы найти зависимость используются те самые Maven Coordinates (GOV). Например, можно искать через **[search.maven.org](https://search.maven.org/)** или через "[mvnrepository.com](https://mvnrepository.com/repos/central)".

Чтобы узнать, где Maven хранит скачанные зависимости можно при помощи Maven плагина узнать, где хранится **localRepository**:
> mvn help:effective-settings

Кроме того, из одних зависимостей могут быть исключены другие зависимости. Например:
```xml
<dependency>
	<groupId>group-a</groupId>
	<artifactId>artifact-a</artifactId>
    <version>1.0</version>
    <exclusions>
		<exclusion>
			<groupId>group-c</groupId>
          	<artifactId>excluded-artifact</artifactId>
        </exclusion>
	</exclusions>
</dependency>
```
Это позволяет исключать **[транзитивные зависимости](https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html#Transitive_Dependencies)** чтобы избежать конфликты версий.
Кроме того, транзитивные зависимости могут быть исключены при помощи настройки **"[Optional](https://www.baeldung.com/maven-optional-dependency)"**.

Чтобы увидеть граф зависимостей можно построить **Dependency Tree**:
> mvn dependency:tree -Dverbose=true


## [↑](#Home) <a name="management"></a> Dependency management
Важной частью работы с зависимостями является **Dependency Management**. Подробнее про этот механизм можно прочитать в руководстве Maven про **"[Dependency Management](https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html#dependency-management)"**.

В данном блоке описываются конфигурации зависимостей (какие версии использовать и нужно ли исключать что-то из зависимостей). Это позволяет использовать этот механизм в качестве **[Bill of Materials (BOM)](https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html#bill-of-materials-bom-poms)**, когда нужно добиться того, чтобы разные версии разных библиотек не конфликтовали друг с другом. Например, при использовании Spring Framework.


## [↑](#Home) <a name="scope"></a> Dependency Scope
Говоря про зависимости стоит так же помнить про **[Dependency Scope](https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html#Dependency_Scope)**.

Существуют следующие типы dependency scope:
- **compile** (используется по умолчанию)
Зависимости с таким типом доступны во всех classpath: компиляции, тестирования и выполнения. Кроме того, зависимости будут включены в package.
- **provided**
Зависимости с таким типом будут доступны на classpath только во время компиляции, но не будут включены в package. Это означаете, что во время выполнения их должен будет предоставить (provide) кто-то другой. Например, сервер приложений, на котором будет развёрнут проект
- **runtime**
Такие зависимости не будут включены в classpath при компиляции, но будут использоваться в classpath при тестировании и при выполнении. Соответственно, они будут в том числе включены в package
- **test**
Такие зависимости будут использоваться только при тестировании и в classpath при компиляции и в package не попадут
- **import**
Позволяет указать зависимости типа **pom**, которые добавляют в проект описание того, какие версии зависимостей нужно использовать (при помощи секции ```<dependencyManagement>```).


## [↑](#Home) <a name="inheritance"></a> Project Inheritance
Как ранее было сказано, по умолчанию все Maven проекты унаследованы от **Super Pom** до тех пор, пока не будет указан родительский проект. Подробнее про это можно прочитать в документации Maven в разделе "[Project Inheritance](https://maven.apache.org/guides/introduction/introduction-to-the-pom.html#Project_Inheritance)" или "[Maven Complete Reference: 3.5.2. Project Inheritance](https://books.sonatype.com/mvnref-book/reference/pom-relationships-sect-project-relationships.html#pom-relationships-sect-project-inheritance)".

Некоторые фрэймворки предлагают наследоваться от своего parent-pom, например так делает Spring Framework: "[3.1. Inheriting the Starter Parent POM](https://docs.spring.io/spring-boot/docs/current/maven-plugin/reference/htmlsingle/#using-parent-pom)".

Вместо наследования Maven предлагает другой механизм - **[Aggregation](https://maven.apache.org/guides/introduction/introduction-to-the-pom.html#Project_Aggregation)**. В таком случае проекты считают модулями и объявляют в некотором "управляющем проекте". Вызов фазы на управляющем проекте приводит к вызову этой фазы у всех указанных модулей.

Подробнее можно прочитать в Sonatype Maven Book в разделе **"[3.6.2. Multi-module vs. Inheritance](https://books.sonatype.com/mvnref-book/reference/pom-relationships-sect-pom-best-practice.html#pom-relationships-sect-multi-vs-inherit)"**.

Говоря про наследование и Effective Pom очень важно помнить про то, как Maven "складывает" описания pom. Подробнее про это хорошо написано в документации Maven, в разделе **"[Plugins: default configuration inheritance](https://maven.apache.org/pom.html#plugins)"**.


## [↑](#Home) <a name="profiles"></a> Maven Profiles
Maven предоставляет так называемые профили сборки. Каждый такой профиль позволяет при условии включённости профиля добавлять какую-то часть в pom, например какие-то другие зависимости или property.

Более подробно стоит ознакомиться в официальной документации Maven в разделе **"[Introduction to Build Profiles](https://maven.apache.org/guides/introduction/introduction-to-profiles.html)"**.

Кроме того, профили можно удобно использовать из современных IDE. Например можно ознакомиться с руководством от IntelliJ IDEA: **"[Maven profiles](https://www.jetbrains.com/help/idea/work-with-maven-profiles.html#activate_maven_profiles)"**.


## [↑](#Home) <a name="properties"></a> Maven Properties
Maven позволяет определять различные именованные настройки. Более того, эти настройки наследуются из родительского проекта в дочерние.
Пример указания пользовательских properties:
```xml
<properties>
	<hibernate.version>3.6.0.Final</hibernate.version>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>
</properties>
```

Maven properties довольно гибкий инструмент. Например, их можно передавать через командуню строку при выполнении maven команд. Например, можно переписать значение property с именем finalName:
> mvn -DfinalName=build clean package

Более подробно про механизм property:
- [Maven The Complete Reference: 9.2. Maven Properties](https://books.sonatype.com/mvnref-book/reference/resource-filtering-sect-properties.html)
- [Maven Pom Reference: Properties](https://maven.apache.org/pom.html#Properties)
- [Maven Model Builder](https://maven.apache.org/ref/3.2.3/maven-model-builder/)


## [↑](#Home) <a name="resources"></a> Additional Resources
Maven - это огромная тема. Более подробно с Maven можно ознакомиться по следующим ссылкам:
- [Maven: The Complete Reference](https://books.sonatype.com/mvnref-book/reference/index.html)
- [Maven Guides](https://maven.apache.org/guides/index.html)