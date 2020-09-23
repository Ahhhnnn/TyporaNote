# 用好Java 8的日期时间类，少踩一些“老三样”的坑

https://time.geekbang.org/column/article/224240

## 初始化日期时间

年应该是和 1900 的差值，月应该是从 0 到 11 而不是从 1 到 12。

```java

Date date = new Date(2019 - 1900, 11, 31, 11, 12, 13);
```

使用 Calendar 改造之后，初始化时年参数直接使用当前年即可，不过月需要注意是从 0 到 11。当然，你也可以直接使用 Calendar.DECEMBER 来初始化月份，更不容易犯错。为了说明时区的问题，我分别使用当前时区和纽约时区初始化了两次相同的日期

```java

Calendar calendar = Calendar.getInstance();
calendar.set(2019, 11, 31, 11, 12, 13);
System.out.println(calendar.getTime());
Calendar calendar2 = Calendar.getInstance(TimeZone.getTimeZone("America/New_York"));
calendar2.set(2019, Calendar.DECEMBER, 31, 11, 12, 13);
System.out.println(calendar2.getTime());
```

```java

Tue Dec 31 11:12:13 CST 2019
Wed Jan 01 00:12:13 CST 2020
```

## “恼人”的时区问题

我们知道，全球有 24 个时区，同一个时刻不同时区（比如中国上海和美国纽约）的时间是不一样的。对于需要全球化的项目，如果初始化时间时没有提供时区，那就不是一个真正意义上的时间，只能认为是我看到的当前时间的一个表示。

关于 Date 类，我们要有两点认识：一是，Date 并无时区问题，世界上任何一台计算机使用 new Date() 初始化得到的时间都一样。因为，Date 中保存的是 UTC 时间，UTC 是以原子钟为基础的统一时间，不以太阳参照计时，并无时区划分。二是，Date 中保存的是一个时间戳，代表的是从 1970 年 1 月 1 日 0 点（Epoch 时间）到现在的毫秒数。尝试输出 Date(0)：

```java

System.out.println(new Date(0));
System.out.println(TimeZone.getDefault().getID() + ":" + TimeZone.getDefault().getRawOffset()/3600000);
```

我得到的是 1970 年 1 月 1 日 8 点。因为我机器当前的时区是中国上海，相比 UTC 时差 +8 小时：

```java
Thu Jan 01 08:00:00 CST 1970Asia/Shanghai:8
```

所有人同一时间点使用new Date(0)得到的一定是同一个时间戳，打印出来不一样是因为，System.out.println默认使用了机器的默认时区进行转换

对于国际化（世界各国的人都在使用）的项目，处理好时间和时区问题首先就是要正确保存日期时间。这里有两种保存方式：

- 方式一，以 UTC 保存，保存的时间没有时区属性，是不涉及时区时间差问题的世界统一时间。我们通常说的时间戳，或 Java 中的 Date 类就是用的这种方式，这也是推荐的方式。
- 方式二，以字面量保存，比如年 / 月 / 日 时: 分: 秒，一定要同时保存时区信息。只有有了时区信息，我们才能知道这个字面量时间真正的时间点，否则它只是一个给人看的时间表示，只在当前时区有意义。Calendar 是有时区概念的，所以我们通过不同的时区初始化 Calendar，得到了不同的时间

正确保存日期时间之后，就是正确展示，即我们要使用正确的时区，把时间点展示为符合当前时区的时间表示。到这里，我们就能理解为什么会有所谓的“时间错乱”问题了。接下来，我再通过实际案例分析一下，从字面量解析成时间和从时间格式化为字面量这两类问题。

第一类是，对于同一个时间表示，比如 2020-01-02 22:00:00，不同时区的人转换成 Date 会得到不同的时间（时间戳）：

```java

String stringDate = "2020-01-02 22:00:00";
SimpleDateFormat inputFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
//默认时区解析时间表示
Date date1 = inputFormat.parse(stringDate);
System.out.println(date1 + ":" + date1.getTime());
//纽约时区解析时间表示
inputFormat.setTimeZone(TimeZone.getTimeZone("America/New_York"));
Date date2 = inputFormat.parse(stringDate);
System.out.println(date2 + ":" + date2.getTime());
```

可以看到，把 2020-01-02 22:00:00 这样的时间表示，对于当前的上海时区和纽约时区，转化为 UTC 时间戳是不同的时间：

```java

Thu Jan 02 22:00:00 CST 2020:1577973600000
Fri Jan 03 11:00:00 CST 2020:1578020400000
```

这正是 UTC 的意义，并不是时间错乱。**对于同一个本地时间的表示，不同时区的人解析得到的 UTC 时间一定是不同的，反过来不同的本地时间可能对应同一个 UTC**

第二类问题是，格式化后出现的错乱，即同一个 Date，在不同的时区下格式化得到不同的时间表示。比如，在我的当前时区和纽约时区格式化 2020-01-02 22:00:00：

```java

String stringDate = "2020-01-02 22:00:00";
SimpleDateFormat inputFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
//同一Date
Date date = inputFormat.parse(stringDate);
//默认时区格式化输出：
System.out.println(new SimpleDateFormat("[yyyy-MM-dd HH:mm:ss Z]").format(date));
//纽约时区格式化输出
TimeZone.setDefault(TimeZone.getTimeZone("America/New_York"));
System.out.println(new SimpleDateFormat("[yyyy-MM-dd HH:mm:ss Z]").format(date));
```

输出如下，我当前时区的 Offset（时差）是 +8 小时，对于 -5 小时的纽约，晚上 10 点对应早上 9 点

```java

[2020-01-02 22:00:00 +0800]
[2020-01-02 09:00:00 -0500]
```

**因此，有些时候数据库中相同的时间，由于服务器的时区设置不同，读取到的时间表示不同。这，不是时间错乱，正是时区发挥了作用，因为 UTC 时间需要根据当前时区解析为正确的本地时间。**

所以，**要正确处理时区，在于存进去和读出来两方面**：**存的时候，需要使用正确的当前时区来保存，这样 UTC 时间才会正确；读的时候，也只有正确设置本地时区，才能把 UTC 时间转换为正确的当地时间**

Java 8 推出了新的时间日期类 ZoneId、ZoneOffset、LocalDateTime、ZonedDateTime 和 DateTimeFormatter，处理时区问题更简单清晰

- 首先初始化上海、纽约和东京三个时区。我们可以使用 ZoneId.of 来初始化一个标准的时区，也可以使用 ZoneOffset.ofHours 通过一个 offset，来初始化一个具有指定时间差的自定义时区。
- 对于日期时间表示，LocalDateTime 不带有时区属性，所以命名为本地时区的日期时间；而 ZonedDateTime=LocalDateTime+ZoneId，具有时区属性。因此，LocalDateTime 只能认为是一个时间表示，ZonedDateTime 才是一个有效的时间。在这里我们把 2020-01-02 22:00:00 这个时间表示，使用东京时区来解析得到一个 ZonedDateTime。
- 使用 DateTimeFormatter 格式化时间的时候，可以直接通过 withZone 方法直接设置格式化使用的时区。最后，分别以上海、纽约和东京三个时区来格式化这个时间输出

```java

//一个时间表示
String stringDate = "2020-01-02 22:00:00";
//初始化三个时区
ZoneId timeZoneSH = ZoneId.of("Asia/Shanghai");
ZoneId timeZoneNY = ZoneId.of("America/New_York");
ZoneId timeZoneJST = ZoneOffset.ofHours(9);
//格式化器
DateTimeFormatter dateTimeFormatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
// 使用的是timeZoneJST时区 相当于存的是timeZoneJST的时间表示
ZonedDateTime date = ZonedDateTime.of(LocalDateTime.parse(stringDate, dateTimeFormatter), timeZoneJST);
//使用DateTimeFormatter格式化时间，可以通过withZone方法直接设置格式化使用的时区
DateTimeFormatter outputFormat = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss Z");
System.out.println(timeZoneSH.getId() + outputFormat.withZone(timeZoneSH).format(date));
System.out.println(timeZoneNY.getId() + outputFormat.withZone(timeZoneNY).format(date));
System.out.println(timeZoneJST.getId() + outputFormat.withZone(timeZoneJST).format(date));
```

可以看到，相同的时区，经过解析存进去和读出来的时间表示是一样的（比如最后一行）；而对于不同的时区，比如上海和纽约，最后输出的本地时间不同。+9 小时时区的晚上 10 点，对于上海是 +8 小时，所以上海本地时间是晚上 9 点；而对于纽约是 -5 小时，差 14 小时，所以是早上 8 点：

```java
Asia/Shanghai2020-01-02 21:00:00 +0800
America/New_York2020-01-02 08:00:00 -0500
+09:002020-01-02 22:00:00 +0900
```

要正确处理国际化时间问题，我推荐使用 Java 8 的日期时间类，即使用 ZonedDateTime 保存时间，然后使用设置了 ZoneId 的 DateTimeFormatter 配合 ZonedDateTime 进行时间格式化得到本地时间表示。这样的划分十分清晰、细化，也不容易出错。



## 日期时间格式化和解析

### SimpleDateFormat 格式化格式

Y 表示week year，也就是所在的周属于哪一年

y 表示年

没有特殊需求情况下，一般都是用y表示年



### 定义的 static 的 SimpleDateFormat 可能会出现线程安全问题

```java

ExecutorService threadPool = Executors.newFixedThreadPool(100);
for (int i = 0; i < 20; i++) {
    //提交20个并发解析时间的任务到线程池，模拟并发环境
    threadPool.execute(() -> {
        for (int j = 0; j < 10; j++) {
            try {
                System.out.println(simpleDateFormat.parse("2020-01-01 11:12:13"));
            } catch (ParseException e) {
                e.printStackTrace();
            }
        }
    });
}
threadPool.shutdown();
threadPool.awaitTermination(1, TimeUnit.HOURS);
```

原因是：

- SimpleDateFormat 继承了 DateFormat，DateFormat 有一个字段 Calendar；

- SimpleDateFormat 的 parse 方法调用 CalendarBuilder 的 establish 方法，来构建 Calendar；

- establish 方法内部先清空 Calendar 再构建 Calendar，整个操作没有加锁。

显然，如果多线程池调用 parse 方法，也就意味着多线程在并发操作一个 Calendar，可能会产生一个线程还没来得及处理 Calendar 就被另一个线程清空了的情况



### 当需要解析的字符串和格式不匹配的时候，SimpleDateFormat 表现得很宽容

```java

String dateString = "20160901";
SimpleDateFormat dateFormat = new SimpleDateFormat("yyyyMM");
System.out.println("result:" + dateFormat.parse(dateString));
```

结果

```java
result:Mon Jan 01 00:00:00 CST 2091
```

对于 SimpleDateFormat 的这三个坑，我们使用 Java 8 中的 DateTimeFormatter 就可以避过去。首先，使用 DateTimeFormatterBuilder 来定义格式化字符串，不用去记忆使用大写的 Y 还是小写的 Y，大写的 M 还是小写的 m

**定义一个静态的DateTimeFormatter**

```java

private static DateTimeFormatter dateTimeFormatter = new DateTimeFormatterBuilder()
        .appendValue(ChronoField.YEAR) //年
        .appendLiteral("/")
        .appendValue(ChronoField.MONTH_OF_YEAR) //月
        .appendLiteral("/")
        .appendValue(ChronoField.DAY_OF_MONTH) //日
        .appendLiteral(" ")
        .appendValue(ChronoField.HOUR_OF_DAY) //时
        .appendLiteral(":")
        .appendValue(ChronoField.MINUTE_OF_HOUR) //分
        .appendLiteral(":")
        .appendValue(ChronoField.SECOND_OF_MINUTE) //秒
        .appendLiteral(".")
        .appendValue(ChronoField.MILLI_OF_SECOND) //毫秒
        .toFormatter();
```

```java

//使用刚才定义的DateTimeFormatterBuilder构建的DateTimeFormatter来解析这个时间
LocalDateTime localDateTime = LocalDateTime.parse("2020/1/2 12:34:56.789", dateTimeFormatter);
//解析成功
System.out.println(localDateTime.format(dateTimeFormatter));
//使用yyyyMM格式解析20160901是否可以成功呢？
String dt = "20160901";
DateTimeFormatter dateTimeFormatter = DateTimeFormatter.ofPattern("yyyyMM");
System.out.println("result:" + dateTimeFormatter.parse(dt));
```

## 日期时间的计算

1.8之前

**注意使用30L 转换为long  而不是直接使用30，会发生整形溢出**

```java

Date today = new Date();
Date nextMonth = new Date(today.getTime() + 30L * 1000 * 60 * 60 * 24);
System.out.println(today);
System.out.println(nextMonth);
```

Calendar 加30天

```java

Calendar c = Calendar.getInstance();
c.setTime(new Date());
c.add(Calendar.DAY_OF_MONTH, 30);
System.out.println(c.getTime());
```

1.8 操作日期

**减一天和加一天，以及减一个月和加一个月**

```java

System.out.println("//测试操作日期");
System.out.println(LocalDate.now()
        .minus(Period.ofDays(1))
        .plus(1, ChronoUnit.DAYS)
        .minusMonths(1)
        .plus(Period.ofMonths(1)));
```



### 还可以通过 with 方法进行快捷时间调节

- 使用 TemporalAdjusters.firstDayOfMonth 得到当前月的第一天；
- 使用 TemporalAdjusters.firstDayOfYear() 得到当前年的第一天；
- 使用 TemporalAdjusters.previous(DayOfWeek.SATURDAY) 得到上一个周六；
- 使用 TemporalAdjusters.lastInMonth(DayOfWeek.FRIDAY) 得到本月最后一个周五。



```java

System.out.println("//本月的第一天");
System.out.println(LocalDate.now().with(TemporalAdjusters.firstDayOfMonth()));

System.out.println("//今年的程序员日");
System.out.println(LocalDate.now().with(TemporalAdjusters.firstDayOfYear()).plusDays(255));

System.out.println("//今天之前的一个周六");
System.out.println(LocalDate.now().with(TemporalAdjusters.previous(DayOfWeek.SATURDAY)));

System.out.println("//本月最后一个工作日");
System.out.println(LocalDate.now().with(TemporalAdjusters.lastInMonth(DayOfWeek.FRIDAY)));
```

### 可以直接使用 lambda 表达式进行自定义的时间调整

为当前时间增加 100 天以内的随机天数：

```java

System.out.println(LocalDate.now().with(temporal -> temporal.plus(ThreadLocalRandom.current().nextInt(100), ChronoUnit.DAYS)));
```

除了计算外，还可以判断日期是否符合某个条件。比如，自定义函数，判断指定日期是否是家庭成员的生日：

**使用 Java 8 操作和计算日期时间虽然方便，但计算两个日期差时可能会踩坑：Java 8 中有一个专门的类 Period 定义了日期间隔，通过 Period.between 得到了两个 LocalDate 的差，返回的是两个日期差几年零几月零几天。如果希望得知两个日期之间差几天，直接调用 Period 的 getDays() 方法得到的只是最后的“零几天”，而不是算总的间隔天数**

比如，计算 2019 年 12 月 12 日和 2019 年 10 月 1 日的日期间隔，很明显日期差是 2 个月零 11 天，但获取 getDays 方法得到的结果只是 11 天，而不是 72 天：

```java

System.out.println("//计算日期差");
LocalDate today = LocalDate.of(2019, 12, 12);
LocalDate specifyDate = LocalDate.of(2019, 10, 1);
System.out.println(Period.between(specifyDate, today).getDays());
System.out.println(Period.between(specifyDate, today));
System.out.println(ChronoUnit.DAYS.between(specifyDate, today));
```

可以使用 ChronoUnit.DAYS.between 解决这个问题：

```java

//计算日期差
11
P2M11D
72
```

