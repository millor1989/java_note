### Stream

Java8 的两大重要改变：Lambda 和 Stream API（`java.util.stream.*`）。

#### 1、关于Stream

Stream 是 Java8 出来集合的关键抽象概念。使用 Stream API 可以对集合数据进行**并行的**、**流式**处理。Stream API 是一种**高效**且**易于使用**的处理数据方式。

流失数据渠道，用于操作数据源（集合、数组等）所生成的元素序列。集合对应的是数据，流对应的是计算。

##### 注意：

1. Stream 本身不会存储元素
2. Stream **不会改变源对象**；而是返回一个持有结果的新的 Stream
3. Stream 操作是**延迟执行**的，在需要结果的时候才会执行。

##### Stream 操作的步骤：

1. 创建 Stream：从数据源（数组、集合）获取流
2. 中间操作：一系列中间操作链
3. 终止操作（终端操作）：执行中间操作链，产生结果

#### 2、创建 Stream

##### 2.1、由集合创建 Stream

Java8 的 Collection 接口被扩展，提供了两个获取流的方法：

- `default Stream stream()`：返回一个顺序流（sequential stream）
- `default Stream parallelStream()`：返回一个并行流（parallel stream）

##### 2.2、由数组创建 Stream

Java8 的 `Arrays` 提供了可以获取顺序流的静态方法：

```java
public static <T> Stream<T> stream(T[] array)

public static <T> Stream<T> stream(T[] array, int startInclusive, int endExclusive)

public static IntStream stream(int[] array)

public static LongStream stream(long[] array)

// ...
```

##### 2.3、由值创建流

静态方法 `Stream.of(T... valuse)`，接收任意数量的参数，显式创建一个流。

`Stream.empty()` 创建空流。

##### 2.4、由函数创建流

可以使用函数创建无限流：

```java
// 迭代
public static Stream iterate(final T seed, final UnaryOperator f)
// 例：初始值 0，每次迭代加 2
Stream.iterate(0, (x) -> x + 2);


// 生成
public static Stream generate(Supplier s)
// 例：使用随机数生成
Stream.generate(() -> Math.random())
```

#### 3、中间操作（intermediate operation）

多个中间操作可以连接起来构成一个流水线，除非流水线由终止操作触发，否则中间操作不会
执行任何的处理。终止操作触发时中间操作一次性全部执行，称为“惰性求值”。

##### 3.1、筛选与切片

- `filter(Predicate p)`——通过 Lambda 从流中排除某些元素
- `limit(long maxSize)`——截断流
- `skip(long n)`——返回跳过前 n 个元素的流，数据流中元素不足 n 时，返回空流。
- `distinct()`——通过流中元素的 `hashCode()` 和 `equals()` 进行元素去重

##### 3.2、mapping

- `map(Function f)`——接收函数作为参数，生成包含与流中元素一一对应的新元素的流；类似有 `mapToInt`、`mapToLong`、`mapToDouble`
- `flatMap(Function f)`—— 接收函数作为参数，将流中的每个值都换成另一个流，然后把所有流链接构成一个流；类似有 `flatMapToInt`、`flatMapToLong`、`flatMapToDouble`

##### 3.3、排序

- `sorted()`——自然排序
- `sorted(Comparator com)`——用比较器定制排序

##### 3.4、窥视

`peek(Consumer<? super T> action)`——返回由流中元素构成的流，额外对流中的元素进行 `action` 操作。主要用于 debug ——查看流中元素。

#### 4、终止操作（terminal operation）

**流进行了终止操作之后，就不能再次使用了**！！！

##### 4.1、迭代

Stream API 使用**内部迭代**（使用 `Collection` 接口需要用户去迭代——外部迭代）

- `forEach(Consumer<? super T> action)`——迭代流中的元素
- `forEachOrdered(Consumer<? super T> action)`——如果流有序，按序迭代

##### 4.2、查找与匹配

- `allMatch(Predicate p)`——检查是否匹配所有元素
- `anyMatch(Predicate p)`——检查是否至少匹配一个元素
- `noneMatch(Predicate p)`——检查是否没有匹配的元素
- `findFirst()`——返回第一个元素
- `findAny()`——返回当前流中的任意一个元素

##### 4.3、统计

- `count()`——返回流中元素个数
- `max(Comparator c)`——返回流中最大值
- `min(Comparator c)`——返回流中最小值

##### 4.4、规约操作

- `reduce(T iden, BinaryOperator b)`——将流中元素结合，得到一个 T 类型值（iden 为初始值）
- `reduce(BinaryOperator b)`——将流中元素结合，得到一个 `Optional` 值

##### 4.5、收集

- `collect(Collector c)`——接收 `Collector` 接口实现，将流转换为其他形式。

`Collector` 接口中方法的实现决定了如何对流执行收集操作。

`Collectors` 提供了很多静态方法，可以很方便地创建常见的收集器实例：

| 方法              | 返回类型             | 作用                                             |
| ----------------- | -------------------- | ------------------------------------------------ |
| toList            | List                 | 将流中元素收集到 List                            |
| toSet             | Set                  | 将流中元素收集到 Set                             |
| toCollection      | Collection           | 将流中元素收集到创建的集合                       |
| counting          | Long                 | 计算流中元素的个数                               |
| sumingInt         | Integer              | 计算流中元素某个整数属性的和                     |
| averagingInt      | Double               | 计算流中元素某个整数属性的平均值                 |
| summarizingInt    | IntSummaryStatistics | 计算流中元素某个整数属性的统计值（比如，平均值） |
| joining           | String               | 连接流中的每个字符串                             |
| maxBy             | Optional             | 根据比较器选择最大值                             |
| minBy             | Optional             | 根据比较器选择最小值                             |
| reducing          | 规约产生的类型       | 使用提供的 `BinaryOperator` 进行 reduce          |
| collectingAndThen | 转换函数返回的类型   | 包裹另一个收集器，转换器结果数据                 |
| groupingBy        | Map<K, List>         | 根据流中元素的某个属性 K 进行 group              |
| partitioningBy    | Map<Boolean, List>   | 根据 true 或 false 进行分区                      |

##### 4.6、并行流与串行流

并行流就是把一个内容分成多个数据块，并用不同的线程分别处理每个数据块的流。

Java 8 中对并行进行了优化，可以很容易的对数据进行并行操作。 Stream API 可以声
明性地通过 `parallel()` 与 `sequential()` 在并行流与顺序流之间进行切换。

##### 4.7、Fork / Join 框架

Stream 并行流使用了 Fork / join 框架。

[使用并行流要注意线程安全的问题？高并发场合下不能使用并发流？](https://cloud.tencent.com/developer/article/1544929)

Fork / Join 框架，就是在必要的情况下，将一个大任务拆分（fork）成若干个小任务（拆到不可再拆时），再将一个个的小任务运算的结果进行 join 汇总，类似 Hadoop 中的 MapReduce 框架。

与传统线程池的不同的是，Fork / Join 框架采用 “工作窃取”模式（work-stealing）： 当执行新的任务时可以将其拆分成更小的任务执行，并将小任务加到线程队列中，然后再从一个随机线程的队列中偷一个并把它放在自己的队列中。在 fork/join 框架实现中，如果某个子问题由于等待另外一个子问题的完成而无法继续运行，那么处理该子问题的线程会主动寻找其他尚未运行的子问题来执行，只有队列中没有任务线程才是空闲的。这种方式减少了线程的等待时间，提高了性能。

计算累加和的例子：

```java
import java.util.concurrent.ForkJoinPool;
import java.util.concurrent.Future;
import java.util.concurrent.RecursiveTask;

public class ForkJoinTaskExample extends RecursiveTask<Integer> {
    public static final int threshold = 2;
    private int start;
    private int end;

    public ForkJoinTaskExample(int start, int end) {
        this.start = start;
        this.end = end;
    }

    @Override
    protected Integer compute() {
        int sum = 0;
        // 如果任务足够小就计算任务
        boolean canCompute = (end - start) <= threshold;
        if (canCompute) {
            for (int i = start; i <= end; i++) {
                sum += i;
            }
        } else {
            // 如果任务大于阈值，就分裂成两个子任务计算
            int middle = (start + end) / 2;
            ForkJoinTaskExample leftTask = new ForkJoinTaskExample(start, middle);
            ForkJoinTaskExample rightTask = new ForkJoinTaskExample(middle + 1, end);
            // 执行子任务
            leftTask.fork();
            rightTask.fork();
            // 等待任务执行结束合并其结果
            int leftResult = leftTask.join();
            int rightResult = rightTask.join();
            // 合并子任务
            sum = leftResult + rightResult;
        }
        return sum;
    }

    public static void main(String[] args) {
        ForkJoinPool forkjoinPool = new ForkJoinPool();
        // 生成一个计算任务，计算 1+2+3+4+...
        ForkJoinTaskExample task = new ForkJoinTaskExample(1, 100);
        // 执行一个任务
        Future<Integer> result = forkjoinPool.submit(task);
        try {
            System.out.println(result.get());
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

