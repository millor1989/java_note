### 日期、时间、时间戳

#### 1、LocalDate、LocalTime、LocalDateTime

`LocalDate`、`LocalTime`、`LocalDateTime` 类的实例是不可变得对象，分别表示使用 `ISO-8601` 日历系统的日期、时间、日期和时间。它们提供了简单的日期或时间，**并不包含当前的时间信息和时区信息**。

| 方法                                               | 描述                                                         |
| -------------------------------------------------- | ------------------------------------------------------------ |
| now()                                              | 静态方法，根据当前时间创建对象                               |
| of()                                               | 静态方法，根据指定日期/时间创建 对象                         |
| plusDays, plusWeeks, plusMonths, plusYears         | 向当前 `LocalDate` 对象添加几天、 几周、 几个月、 几年       |
| minusDays, minusWeeks, minusMonths, minusYears     | 从当前 `LocalDate` 对象减去几天、 几周、 几个月、 几年       |
| plus, minus                                        | 添加或减少一个 `Duration` 或 `Period`                        |
| withDayOfMonth, withDayOfYear, withMonth, withYear | 将月份天数、年份天数、月份、年 份 修 改 为 指 定的值并 返回新的 LocalDate 对象 |
| getDayOfMonth                                      | 获得月份第 (1-31)天                                          |
| getDayOfYear                                       | 获得年份第(1-366)天                                          |
| getDayOfWeek                                       | 获得星期几(返回一个 `DayOfWeek` 枚举值)                      |
| getMonth                                           | 获得月份, 返回一个 `Month` 枚举值                            |
| getMonthValue                                      | 获得月份(1-12)                                               |
| getYear                                            | 获得年份                                                     |
| until                                              | 获得两个日期之间的 `Period` 对象， 或者指定 `ChronoUnits` 的数字 |
| isBefore, isAfter                                  | 比较两个 `LocalDate`                                         |
| isLeapYear                                         | 判断是否是闰年                                               |

#### 2、`Instant` 时间戳

`Instant` 用于时间戳运算。是以 Unix 元年（传统的设定为 UTC 时区 1970 年 1 月 1 日午夜时分）开始。

#### 3、`Duration` 和 `Period`

它们两个都有 `between` 方法来计算间隔。

`Duation`：用于计算两个“时间”间隔

`Period`：用于计算两个“日期”间隔

#### 4、日期操作

`TemporalAdjuster`：时间校正器。可以执行注入：将日期调整为“下个周日”之类的操作。

`TemporalAdjusters`：通过静态方法提供了大量常用的 `TemporalAdjuster` 实现。

例如：

```java
// 获取下个周日
LocalDate.now().with(TemporalAdjusters.next(DayOfWeek.SUNDAY));

// 获取当月第一天
LocalDate.now().with(TemporalAdjusters.firstDayOfMonth());
```

#### 5、解析和格式化

`java.time.format.DateTimeFormatter` 类，提供了格式化方法，可以根据标准格式（ISO）、语言环境格式（EN）、自定义格式进行格式化。

#### 6、时区处理

带时区的时间分别为：`ZonedDate`、`ZonedTime`、`ZonedDateTime`。

`ZoneId`类包含了所有的时区信息：

- `getAvailableZoneIds()`：可以获取所有时区信息
- `of(id)`：用指定的时区信息获取 `ZoneId` 对象。

#### 7、与传统日期转换

有`from` 或者 `to` 方法，来与传统的日期进行转换。

比如，：

```java
// java.time.Instant 与 java.util.Date
Date.from(instant);

date.toInstant();

// java.time.LocalDate 与 java.sql.Date
Date.valueOf(localDate);

date.toLocalDate();

// ...
```

