---
title: LocalDate
date: 2021-07-31 13:10:52
tags:
---
# LocalDate、LocalDateTime、LocalTime相关类使用指南

> 由于JDK1.8之前的时间相关处理类设计的不合理及不易用，jdk的大佬们在吸收了jodaTime 等优秀的时间框架设计理念,在Java8中提出了关于时间新的使用姿势。使用Java8新的时间特性会让你像从eclipse换到IntellJ IDEA那样丝滑顺畅。

## 一、起源（为替代Date、Calendar而生）

从 Java 1.0 开始，就提供了对日期与时间处理的 `java.util.Date` 类，它允许把日期解释为年、月、日、小时、分钟和秒值，也允许格式化和解析日期字符串。不过，这些函数的 API 不易于实现国际化。在升级版本到 Java 1.1 前，Java 维护者认为 Date 类很难被重新构造，由于这个原因，Java 1.1 增加了一个新的 `java.util.Calendar` 类。`Calendar` 类实现日期和时间字段之间转换，使用 `DateFormat` 类来格式化和解析日期字符串。可是在开发者使用过程中感受到，`Calendar` 类并不比 `Date` 类好用，它们面临的部分问题是：

- 可变性：像时间和日期这样的类应该是不可变的。
- 偏移性：Date 中的年份是从 1900 开始的，而月份是从 0 开始的，不太符合常识习惯。
- 类命名：Date 并不表示处理"日期"，而`Calendar`类也不全是表示"日历"，类命名比较不合理。
- 格式化：时间日期格式化只对 `Date` 有用，`Calendar` 则不行，且时间格式化对象存在线程安全问题。

自 2001 年起 Joda-Time 项目发布，它提供了简单易用且线程安全的时间类库，很快在 Java 社区中流行并广泛使用。Java 维护人员考虑到 JDK 中也需要一个这样的库，于是就与巴西的 Michael Nascimento Santos 合作，Java 官方 JDK 新增了的时间/日期 API的进程（JSR-310）。

直观性方面举个例子:

```java
  Date date = new Date(121, 7, 18);
  //输出: Wed Aug 18 00:00:00 CST 2021
```

很难辨认这个构造方法

## 二、奥德赛（使用指南）

### 2.1 新的时间API组成

新增的时间 API 由 5 个包组成，如下：

- **[java.time](http://docs.oracle.com/javase/8/docs/api/java/time/package-summary.html)：** 包含值对象的基础包
- **[java.time.zone](http://docs.oracle.com/javase/8/docs/api/java/time/zone/package-summary.html)：** 包含时区支持的类
- **[java.time.chrono](http://docs.oracle.com/javase/8/docs/api/java/time/chrono/package-summary.html)：** 提供对不同的日历系统的访问
- **[java.time.format](http://docs.oracle.com/javase/8/docs/api/java/time/format/package-summary.html)：** 格式化和解析时间和日期
- **[java.time.temporal](http://docs.oracle.com/javase/8/docs/api/java/time/temporal/package-summary.html)：** 包括底层框架和扩展特性

### 2.2 一些时间概念

#### 2.2.1 时间与时刻

- **时间：** 可以表示准确的时刻或日期，也可以表示一个时间段。

- **时刻：** 指某一瞬bai间，表示具体的某一时间点，只是时间中的一个点。

#### 2.2.2 时间戳和时区

- **时间戳：** 时间戳是指格林尼治时间1970年01月01日00时00分00秒到现在的总秒数（毫秒数），可以理解成绝对时间，它与时区无关，不同时区对同一时间戳的解读不一样
- **时区：** 同一时刻（时间戳），世界上各地区的时间可能是不一样的，具体时间与时区相关，按经度一共分为24个时区，英国格林尼治是0时区，中国北京是东8区

#### 2.2.3 时间输出格式类型

| 输出类型                                     | 描述                                                         |
| :------------------------------------------- | :----------------------------------------------------------- |
| 2019-06-10T03:48:20.847Z                     | 世界标准时间，T：日期和时间分隔，Z：世界标准时间             |
| 2019-06-10T11:51:48.872                      | 不含时区信息的时间                                           |
| 2019-06-10T11:55:04.421+08:00[Asia/Shanghai] | 包含时区信息的时间，+08:00 表示相对于0时区加 8 小时，[Asia/Shanghai]：时区 |

### 2.3 LocalDate、LocalDateTime、Instant、Duration以及Period

#### 2.3.1使用LocalDate和LocalTime

使用新日期和时间API我们时，最先碰到的可能事`LocalDate`类。该类实例是一个不可变对象，它只提供简单的日期，并不含当天的时间信息。另外它也不含当天的时间信息。另外，它也不附带任何与时区相关的信息。

通过静态工厂方法`of`创建一个`LocalDate`实例。LocalDate实例提供了多种方法来读取常用的值，比如年、月、日 星期几等如下所示：

```java
LocalDate date = LocalDate.of(2021, 7, 25);//2021-07-25
int year = date.getYear();//2021
Month month = date.getMonth();//JULY
int dayOfMonth = date.getDayOfMonth();//25
int i = date.lengthOfMonth();//31
boolean leapYear = date.isLeapYear();//false
```

还可以使用工厂方法从系统时钟获取当前日期：

```java
LocalDate now = LocalDate.now();
```
本文剩余部分讨论的时日期-时间类都提供了类似相应的工厂方法。你还可以通过传递一个`TemporalField` 参数给`get`方法拿到同样的信息。`TemporalField` 是一个接口，他定义了如何访问`Temporal`对象某个字段的值。`ChronField`枚举实现了这一接口，所以你可以很方便的使用get方法得到枚举元素的值，如下所示。

```java
int year = date.get(ChronoField.YEAR);
int month = date.get(ChronoField.MONTH_OF_YEAR);
int day = date.get(ChronoField.DAY_OF_MONTH);
```

类似地，一天中的时间，比如17:30:20，可以使用`LocalTime`类来表示，它有两个重载的工厂方法，一个是接受小时和分钟，第二个同时还接受秒，同`LocalDate`一样，`LocalTime`类也提供了一些`getter`方法来访问这些变量的值，如下所示：

```java
LocalTime time = LocalTime.of(17, 30, 20);//17:30:20
int hour = time.getHour();//17
int minute = time.getMinute();//30
int second = time.getSecond();//20
```

`LocalDate` 和`LocalDateTime` 都可以通过解析代表他们的字符串创建，使用静态方法`parse`，你可以实现这一目的：

```java
LocalDate date = LocalDate.parse("2021-07-27");
LocalTime time = LocalTime.parse("17:30:20");
```

你可以向`parse`方法传递一个`DateTimeFormatter`。该类实例定义了如何格式化一个日期或者时间对象。正如上文所说，它是为了替换老版本的`java.util.DateFormat`推荐替代品。我们会在下文详细介绍。同时，如果那你传入了一个无法解析的字符串，将会抛出继承自`RuntimeException`的`DateTimeParseException`异常。

#### 2.3.2 合并日期和时间

`LocalDateTime`是`LocalDate`和`LocalTime`的复合体。它同时表示了日期和时间。但是不带有时区信息，你可以直接创建，也可以通过合并日期和时间对象构造。如下所示：

```java
LocalDate date = LocalDate.parse("2021-07-27");
LocalTime time = LocalTime.parse("17:30:20");
LocalDateTime dt1 = LocalDateTime.of(date, time);
LocalDateTime dt2 = LocalDateTime.of(2021, Month.JULY, 25, 18, 10, 20);
LocalDateTime dt3 = date.atTime(18, 10, 10);
LocalDateTime dt4 = date.atTime(time);
LocalDateTime dt5 = time.atDate(date);
```

它们通过各自的`atTime`或`atDate`方法，向`LocalDate`传递一个`LocalTime`，或者向`LocalTime`传递一个`LocalDate`。同样的你可以使用`toLocalDate`或者同`LocalTime`方法，从`LocalDateTime`中提取`LocalDate`或者`LocalTime`对象：

```java
 LocalDate date1 = dt1.toLocalDate();
 LocalTime time1 = dt1.toLocalTime();
```

#### 2.3.4 机器的日期和时间格式（Instance类的使用）

作为人，我们习惯于以星期几、几号、几点、几分这样的方式理解日期和时间。但是这种方式对于计算机而言并不容易理解。从计算机角度来看，建模时间最自然的格式是表示一个持续时间段上某个点的单一最大整型。这也是新的`java.time.Instant`类对时间的建模方式，基本上它是已Unix元年时间（传统设定为UTC时区1970年1月1日）开始所有的秒数进行计算。

```java
long currentTimeMillis = instant.getEpochSecond();
```

你可以通过向静态工厂方法`ofEpochSecond`传递一个代表描述的值来创建一个该类的实例。静态工厂方法还有一个增强版本，它接受第二个以纳秒为单位的参数值，对传入作为秒数的参数进行调整。重载的版本会对纳秒进行调整确保它是在0~999 999 999之间如下：

```java
Instant.ofEpochSecond(3);
Instant.ofEpochSecond(3,0);
Instant.ofEpochSecond(2,1_000_000_000);//2秒+10亿纳秒（1秒）
Instant.ofEpochSecond(4,-1_000_000_000);//4秒-10亿纳秒（1秒）
```

正如上文`LocalDate`类一样`Instant`类也支持静态工厂方法`now`，他能帮你获取当前时间戳。在此，再次强调一遍，`Instant`类的设计初衷是为了便于机器使用。它包含的是由秒及纳秒所构成的数字。所以，它无法处理那些人所理解的时间单位。如下：

```java
int day = Instant.now().get(ChronoField.DAY_OF_MONTH);
```

它会抛出这样的异常：

```java
java.time.temporal.UnsupportedTemporalTypeException: Unsupported field: DayOfMonth
```

但是你可以使用`Duration`和`Period`类使用`Instant`，接下来我们会对这部分内容进行介绍。

#### 2.3.5 定义Duration和Period

到目前为止，你看到的所有类，，`Temporal`接口定义了如何读取和操纵为时间建模的对象值。之前的介绍中，我们已经了解了创建Temporal实例的几种方法。很自然的你会想到，我们需要创建两个Temporal时间对象之间的间隔。`Duration`类的静态工厂方法`between`就是为了这个目的而设计的。你可以创建两个`LocalTime`，两个`LocalDate`，或者两个`Instant`对象之间的duration，如下所示：

```java
Duration d1 = Duration.between(time1, time2);
Duration d2 = Duration.between(dateTime1, dateTime2);
Duration d3 = Duration.between(instant1, instant2);
```

由于`LocalDateTime`和`Instant`是为不同目的而设计的，一个是为了人阅读使用，另外一个是便于机器处理，所以二者不能混用。如果你试图，在这两个类对象直接创建duration，会触发一个`DateTimeException`异常。此外，由于duration类主要用于衡量秒和纳秒时间的长短，你不能向`between`方法传递`LocalDate`对象做为参数。也就是说不能传日期进去。

如果你需要以年、月、日的方式对多个时间单位建模，可以使用`Period`类。使用该类的工厂方法`between`，你可以使用得到两个`LocalDate`之间的时长，如下所示：

```java
Period between = Period.between(LocalDate.of(2021, 6, 1), 
                                LocalDate.of(2021, 7, 1));
```

最后，`Duration`和`Period`类都提供了很多非常方便的工厂类，直接创建对应的实例；换句话说，可以不只以来两个时间差来创建定义他们如下：

```java
Duration threeMinutes = Duration.ofMinutes(3);
Duration threeMinutes = Duration.ofMinutes(3, ChronoUnit.MINUTES);

Period tenDays = Period.ofDays(10);
Period threeWeeks = Period.ofWeeks(3);
Period twoYearsSixMonthsOneDay = Period.of(2, 6, 1);
```

`Duration`类和`Period`类**共享了很多相似的方法**，参见下表

| 方法名       | 是否是静态方法 | 方法描述                                           |
| ------------ | :------------: | -------------------------------------------------- |
| between      |       是       | 创建两个时间点之间的时间间隔                       |
| from         |       是       | 由一个临时时间节点创建的时间间隔                   |
| of           |       是       | 由它的组成部分创建时间间隔实例                     |
| parse        |       是       | 由字符串创建时间间隔实例                           |
| addTo        |       否       | 创建该时间间隔的副本，并加到某个指定的时间间隔对象 |
| get          |       否       | 读取该时间间隔的状态                               |
| isNegative   |       否       | 检查该时间间隔是否为负值，不包含零                 |
| isZero       |       否       | 检查该时间间隔的时长是否为零                       |
| minus        |       否       | 通过减去一定的时间创建该时间间隔的副本             |
| multipliedBy |       否       | 将该时间间隔乘某个值创建出新的时间间隔             |
| negated      |       否       | 以忽略某个时长的方式创建该时间间隔副本             |
| plus         |       否       | 将该时间间隔增加某个时长来创建时间间隔副本         |
| subtractFrom |       否       | 从指定的对象减去该时间间隔                         |

截至目前为，介绍的这些日期-时间都是不可修改的，这是为了更好的支持函数式编程，**确保了线程安全**，保持领域模式的一致性而做出的重大设计决定。当然，新的日期和时间API也提供了一些便利的方法来创建这些对象的可变版本。比如你可能向在已有的`LocalDate`对象上增加3天。在接下来还将继续介绍对时间格式化的一些操作。

### 2.4 操纵、解析和格式化日期

如果你已经有一个`LocalDate`对象，想要创建它的一个修改版，最直接也最简单的方法是是用`withAttribute`方法。`withAttribute`方法会创建一个对象，并按照需要赋值他的属性。注意，下面这段代码的所有方法都返回了一个新的对象。它们都不会修改原来的对象！

```java
//以比较直观的方式操纵LocalDate的属性
LocalDate date1 = LocalDate.of(2021, 7, 25);
LocalDate date2 = date1.withYear(2022);
LocalDate date3 = date2.withDayOfMonth(25);
LocalDate date4 = date3.with(ChronoField.MONTH_OF_YEAR, 9);
```

`with`方法和`get`方法类似都声明于**`Temporal`**接口，**所有的日期和时间类API都实现了这两个方法**，他们定义了单点的时间，比如`LocalDate`、`LocalTime`、`LocalDateTime`以及`Instant`。更确切地说，使用`get`和`with`方法能将`Temporal`进行读写分离，它也可以声明式的操纵`LocalDate`对象，比如，可以像下面这段代码那样加上或者减去一段时间：

```java
LocalDate date1 = LocalDate.of(2021, 7, 25);
LocalDate date2 = date1.plusWeeks(1);
LocalDate date3 = date2.minusYears(3);
LocalDate date4 = date3.plus(6, ChronoUnit.MONTHS);
```

与刚才介绍的get和with方法类似，**`plus`方法也是通用方法**，它和`minus`方法都声明于`Temporal`接口中，通过这些方法，对`TemporalUnit`对象加上或者减去一个数字，我们就能非常方面的将`Temporal`对象前进或者回滚到某个时间段，通过`ChronUnit`枚举我们可以非常方便的实现Temporal接口。

**`LocalDate`**、**`LocalTime`**、**`LocalDateTime`**以及**`Instant`类提供了大量的通用方法**，下表进行了总结。

| 方法名   | 是否是静态方法 | 方法描述                                                     |
| -------- | :------------: | ------------------------------------------------------------ |
| from     |       是       | 根据传入的Temporal对象创建新的对象                           |
| now      |       是       | 根据系统时钟来创建当前时间的对象                             |
| of       |       是       | 由Temporal对象的某个部分创建该对象的实例                     |
| parse    |       是       | 由字符串创建时间对象                                         |
| atOffset |       否       | 创建Temporal对象和某个时区偏移相结合                         |
| atZone   |       否       | 将Temporal对象和某个时区相结合                               |
| format   |       否       | 使用某个指定的格式将Temporal对象转换为字符串（Instant类不提供该方法） |
| get      |       否       | 读取Temporal对象的一部分的值                                 |
| minus    |       否       | 通过将当前Temporal对象减去一段时间返回新创建一个对象         |
| plus     |       否       | 通过将当前Temporal对象加上一段时间返回新创建一个对象         |
| with     |       否       | 以当前时间对象为准，改变某些状态来创建一个新的时间对象       |

#### 2.4.1 使用TemporalAdjuster

截至目前，你所看到的所有日期操作相对比较直接的。有的时候，你需要进行一些更加复杂的操作，比如将日期调整到下个周日，下个工作日，或者本月的最后一天。这时你可以使用重载版本的with方法，向其传递一个提供了更多定制化选择的`TemporalAdjuster`对象，更加灵活地处理日期。对于最常见的用例，日期和时间API已经提供了大量预定义的`TemporalAdjuster`。你可以通过静态工厂访问他们，如下所示：

```java
LocalDate date1 = LocalDate.of(2021, 7, 25);
LocalDate date2 = date1.with(TemporalAdjusters.nextOrSame(DayOfWeek.SUNDAY));
LocalDate date3 = date2.with(TemporalAdjusters.lastDayOfMonth());
```

下表提供了`TemporalAdjusters`中包含的工厂方法列表。

| 方法名                    | 描述                                                         |
| ------------------------- | :----------------------------------------------------------- |
| dayOfWeekInMonth          | 创建一个新的日期，它的值为同一个月中每一周的第几天           |
| firstDayOfMonth           | 创建一个新的日期，它的值为当月的第一天                       |
| firstDayOfNextMonth       | 创建一个新的日期，它的值为下月的第一天                       |
| firstDayOfNextYear        | 创建一个新的日期，它的值为明年的第一天                       |
| firstDayOfYear            | 创建一个新的日期，它的值为当年的第一天                       |
| firstInMonth              | 创建一个新的日期，它的值为同一个月中，第一个符合星期几要求的值 |
| lastDayOfMonth            | 创建一个新的日期，它的值为当月的最后一天                     |
| lastDayOfYear             | 创建一个新的日期，它的值为今年的最后一天                     |
| lastInMonth               | 创建一个新的日期，它的值为同一个月中，最后一个符合星期几要求的值 |
| next/previous             | 创建一个新的日期，并将其设定为日期调整后或调整前，第一个符合指定星期几要求的日期 |
| nextOrSame/previousOrSame | 创建一个新的日期，并将其值设定为日期调整后或调整前，第一个符合指定星期几要求的日期，如果该日期已经符合要求，直接返回对象 |

正如上面所列出的方法，使用`TemporalAdjuster`我们可以进行更加复杂的日期操作，而且这些方法的名称也非常直观，方法名基本就是问题陈述。此外，即使你没有找到符合你要求的预定义的`TemporalAdjuster`，创建你自己的`TemporalAdjuster`也并非难事。`TemporalAdjusters`接口只声明了单一的一个方法（这使得它成为了一个函数式接口），定义入下：

```java
@FunctionalInterface
public interface TemporalAdjuster {
    Temporal adjustInto(Temporal temporal);
}
```

这意味着`TemporalAdjuster`接口的实现需要定义如何将一个`Temporal`对象转换为另一个`Temporal`对象。你可以把它看成一个`UnaryOperator<Temporal>`。比如我们想实现设计一个NextWorkingDay的的方法，能计算下个工作日的日期，并且过滤掉周六周日。如果当天介于周一至周五，日期向后移动一天，如果当天是周六或周日则返回下周一代码如下：

```java
TemporalAdjuster nextWorkingDay = TemporalAdjusters.ofDateAdjuster(temporal -> {
    DayOfWeek dow = DayOfWeek.of(temporal.get(ChronoField.DAY_OF_WEEK));
    int dayToAdd = 1;
    if (dow == DayOfWeek.FRIDAY) dayToAdd = 3;
    if (dow == DayOfWeek.SATURDAY) dayToAdd = 2;
    return temporal.plus(dayToAdd, ChronoUnit.DAYS);
});
```

接下来，我们继续讨论关于日期的序列化和反序列化，根据新的日期和时间API来满足自己的业务要求。

#### 2.4.2 格式化和解析日期-时间对象

处理日期和时间对象时，格式化和解析日期-时间对象是另外一个非常重要的功能，新的`java.time.format`包就是特别为这个目的而设计的。这个包中最重要的类是`DateTimeFormatter`。创建格式器最简单的房价是同规它的静态工厂方法及常量。像`BASIC_ISO_DATE`和`ISO_LOCAL_DATE`这样的常量都是`DateTimeFormatter`预定义实例。所有的`DateTimeFormatter`实例都能用于以一定的格式创建代表特定日期或时间的字符串。比如下面我们使用了两个不同格式器序列化生成了字符串：

```java
 LocalDate date1 = LocalDate.of(2021, 7, 25);
 String s1 = date1.format(DateTimeFormatter.BASIC_ISO_DATE);//20210725
 String s2 = date1.format(DateTimeFormatter.ISO_LOCAL_DATE);//2021-07-25
```

你也可以通过解析代表日期或之间的字符串重新创建该日期对象。所有的日期和时间API都提供了表示时间点或者时间段的工厂方法，你可以使用工厂方法`parse`达到重创该日期对象的目的：

```java
 LocalDate date1 = LocalDate.parse("20210725", DateTimeFormatter.BASIC_ISO_DATE);
 LocalDate date2 = LocalDate.parse("2021-07-25", DateTimeFormatter.ISO_LOCAL_DATE);
```

和老的`java.util.DateFormat`相比较，所有的`DateTimeFormatter`**实例都是线程安全的**。所以，你能够以单例模式创建格式器实例，就像`DateTimeFormatter`所定义的那些常量，并能在***多个线程间共享这些实例***，`DateTimeFormatter`还支持一个静态的工厂方法，它可以按照某个特定的模式创建格式器，代码如下

```java
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd");
LocalDate date1 = LocalDate.of(2021, 7, 25);
String formatDate = date1.format(formatter);
LocalDate date2 = LocalDate.parse(formatDate, formatter);
```

`parse`方法用来解析字符串反序列化生成`LocalDate`对象，`format`用来序列化对象，`ofPattern`方法也提供了一个重载的版本，使用它你可以创建某个Locale的格式器，如下:

```java
DateTimeFormatter italianFormatter = DateTimeFormatter.ofPattern("d.MMMM yyyy", Locale.ITALIAN);//25.luglio 2021
        DateTimeFormatter chFormatter = DateTimeFormatter.ofPattern("yyyy MMMM dd ", Locale.CHINA);//2021 七月 25
LocalDate date1 = LocalDate.of(2021, 7, 25);
String format = date1.format(chFormatter);
LocalDate date2 = LocalDate.parse(format, italianFormatter);
```

最后，如果你还需要更细粒度的控制，`DateTimeFormatterBuilder`类还提供了更加复杂的格式器，你可以选择恰当的方法，一步一步构造自己的格式器，另外它还提供了非常强大的解析功能，比如区分大小写的解析、柔性解析（允许解析器使用启发式的机制去解析输入，不精确的匹配指定的模式）、填充，以及在格式器中指定可选接。通过`DateTimeFormatterBuilder`我们可以是实现自己的需求的解释器。如下：

```java
LocalDate date1 = LocalDate.of(2021, 7, 25);
DateTimeFormatter twFormatter = new DateTimeFormatterBuilder()
	.appendText(ChronoField.YEAR)
	.appendLiteral(" ")
	.appendText(ChronoField.MONTH_OF_YEAR)
	.appendLiteral(" ")               			
    .appendText(ChronoField.DAY_OF_MONTH)
    .parseCaseInsensitive()
    .toFormatter(Locale.TRADITIONAL_CHINESE);	
String format1 = date1.format(twFormatter);//2021 七月 25

```

目前为止，你已经学习了如何创建、操纵、格式化以及解析时间点和时间段，但是你还不了解如何处理日期和时间之间的微妙关系。比如你可能需要处理不同时区，或者由于不同历法系统带来的差异，接下来我们再探究如果使用新的日期和时间API解决这些问题。

### 2.5 处理不同的时区和历法

之前你看到的日期和时间都不包含时区信息。时区的处理时新版日期和时间API新增加的重要功能，使用新版日期和时间API时区的处理被极大的简化了。新的`java.time.ZoneId`类时老版本`java.util.TimeZone`的替代平，它的设计目标就是让你无需为时区处理的复杂和繁琐操心。跟其他的日期和时间类一样，`ZoneId`类也是无法被修改的。

在`ZoneRules`这个类中包含了40个这样的实例。你可以简单的通过调用`ZoneId`的`getRules`方法得到指定时区的规则。每隔特定的`ZoneId`对象都有一个地区ID表示，比如：

```java
ZoneId shanghai = ZoneId.of("Asia/Shanghai");
```

地区ID都为“{区域}/{城市}”的格式，这些地区集合的设定都有intnet编号分配机构（IANA）的时区数据库提供。你可以通过Java8的新方法`toZoneId` 将一个老时区对象转换为ZoneId：

```java
ZoneId zoneId = TimeZone.getDefault().toZoneId();
```

一旦得到这个`ZoneId`对象，你就可以将它与`LocalDate`、`LocalDateTime`或者`Instant`对象整合起来，构造一个为一个`ZoneDateTime`对象，他代表了相对于指定时区的时间点，代码如下：

```java
ZoneId shanghai = ZoneId.of("Asia/Shanghai");
LocalDate date = LocalDate.of(2021, 7, 25);
ZonedDateTime zdt1 = date.atStartOfDay(shanghai);
LocalDateTime zdt2 = LocalDateTime.of(2021, Month.JULY, 25, 18, 13, 45);
Instant instant = Instant.now();
ZonedDateTime zdt3 = instant.atZone(shanghai);
```

下图对`ZoneDateTime`组成部分进行了说明：

![LocalDateTime相关类关系](https://cdn.jsdelivr.net/gh/cchenjc/image/20210731140414.png)

通过`ZoneID`,你还可以将`LocalDateTime`转换为Instant

```java
LocalDateTime localDateTime = LocalDateTime.now();
Instant instant = localDateTime.toInstant(ZoneOffset.of("+08:00"));
```

还可以通过反向方式获得`Instant`

```java
LocalDateTime localDateTime1 = LocalDateTime.ofInstant(instant, ZoneId.of("Asia/Shanghai"));
```

#### 2.5.1 利用和UTC/格林尼治时间的固定偏差计算时区

另一种表达时区的方式时利用当前时区和UTC/格林尼治的固定偏差。比如，基于这个理论，表示上海时间为东八区你可以使用`ZoneOffset`类，它时一个`ZoneId`的一个子类，表示当前时间和格林尼治时间的差异：

```java
 ZoneOffset shanghaiOffset = ZoneOffset.of("+08:00");
```

新版的日期和时间API孩提供了另一个高级特性，即对非ISO历法系统的支持。

#### 2.5.2 使用别的日历系统图

ISO-8601日历系统时世界标准日历系统，但是java8中还另外提供了4种其他的日历胸痛，这些日历系统中每一个都有一个对应的日历类，分别是`ThaiBuddhistDate`、`HijrahDate`、`JapaneseDate`、`MinguoDate`。所有这些类以及`LocalDate`都实现了`ChronoLocalDate`接口，能够对公历进行建模。利用`LocalDate`对象，你可以创建这些类的实例如下：

```java
LocalDateTime now = LocalDateTime.now();
MinguoDate minguoDate = MinguoDate.from(now);//Minguo ROC 110-07-28
```

或者，你还可以显示的为某个`Locale`传教日历系统，接着创建该`Locale`对应的日期的实例。新的日期和时间API中，`Chronology`接口建模了一个日历系统，使用它的静态工厂方法`ofLocale`，可以得到它的一个实例，如下：

```java
Chronology chronology = Chronology.ofLocale(Locale.CHINESE);
ChronoLocalDate chronoLocalDate = chronology.dateNow();
```

日期及时间API得设计者建议我们使用`LocalDate`，尽量避免使用`ChronoLocalDate`，原因时开发者会在代码中做一些假设，比如一个月不会超过31天，一年12个月，或者月份数量时固定得，出于这些原因都应该尽量使用`LocalDate`，包括存储、操作、业务规则解读等，不过当你需要输入或输出本地化时，你应该使用`ChronoLocalDate`类。

### 2.6 小结

- Java8之前老版的`java.util.Date`类以及其他用于建模日期时间类有很多不一致及设计上的缺陷，包括易变性以及糟糕的偏移性、默认值和命名。
- 新版的日期和时间API中，日期和时间对象是不可变的（如：String）
- 新的API提供了两种不同的时间表示方式，有效地区分了运行时任何机器的不同需求。
- 你可以使用绝对或者相对的方式操纵日期和时间，操作的结果总是返回一个新的对象，老的对象不会发生变化
- `TemporalAdjuster`让你能够用更精细的方式操纵日期，不再局限于一次只能改变它的一个值，并且你还可以按照需求定义自己的日期转换器。
- 你现在可以按照特定的格式需求，定义自己的格式器，打印输出或者解析日期-时间对象。这些格式器可以通过模板创建，也可以自己编程创建，并且它们都是线程安全。
- 你可以用相对于某个地区/位置的方式，或者以与UTC/格林尼治时间的绝对偏差的方式表示时区，并将其应用到日期-时间的对象上，对其本地化。
- 你可以使用不同于ISO-8601标准的系统的其他日历系统了。
- 下面为相关类转换关系

![LocalDateTime相关类转换图](https://cdn.jsdelivr.net/gh/cchenjc/image/20210802085406.png)

## 三、英灵殿（踩坑记）

### 3.1 SpringMVC中时间类型的转换和序列化问题

### 3.2 FastJson配置全局LocalDateTime序列化

