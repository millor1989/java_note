### volatile

Java `volatile` 关键字是用于标记 Java 变量为“被保存在主内存（main memory）中的”。即，每次读 `volatile` 变量都会从主内存读取，而不是 CPU 缓存，每次写 `volatile` 变量都会写到主内存，而不仅是 CPU 缓存。事实上，`volatile` 关键字的功能不止于此。

#### 1、变量可见性问题

![Threads may hold copies of variables from main memory in CPU caches.](/assets/java-volatile-1.png)

在多个线程操作非 `volatile` 变量的应用中，为了性能，每个线程会从主内存将变量复制到它们运行的 CPU 缓存中，如果计算机有多个 CPU，每个线程可能会运行在不同 CPU 中，即线程可能会把一个变量复制到不同的 CPU 缓存中。

对于非 `volatile` 变量，不能保证 JVM 何时将变量从主内存读取到 CPU 缓存以及何时将 CPU 缓存的变量写到主内存。这可能回导致问题，比如：

```java
public class SharedObject {

    public int counter = 0;

}
```

多个线程访问 `counter` 时，不同线程所在 CPU 中的 `counter` 值可能会与主内存中的 `counter` 值不同。

![The CPU cache used by Thread 1 and main memory contains different values for the counter variable.](/assets/java-volatile-2.png)

#### 2、 `volatile` 可见性保证

对 `volatile` 变量，会直接从主内存读，写也会立即反映到主内存。事实上，`volatile` 的可见性保证，超过了 `volatile` 变量本身：

- 如果线程 A 写一个 `volatile` 变量，线程 B 接着读相同的 `volatile` 变量，那么线程 A 写 `volatile` 变量前所有对线程 A 可见的变量，会在线程 B 读取过 `volatile` 变量后对线程 B 可见。
- 如果线程 A 读一个 `volatile` 变量，那么线程 A 读取 `volative` 变量时对线程 A 可见的所有变量也都会被从主内存中重新读取（re-read）

比如：

```java
public class MyClass {
    private int years;
    private int months
    private volatile int days;


    public void update(int years, int months, int days){
        this.years  = years;
        this.months = months;
        this.days   = days;
    }
}
```

`upate()` 方法写了三个变量，只有 `days` 是 `volatile` 的，但是，写 `days` 时，对线程可见的所有变量（`years`、`months`）也会被写到主内存。

读取变量时，如果首先读取 `days`，那么可以保证 `months` 和 `years` 也是从主内存读取的，从而可以保证 `days` 、 `months`、`years` 读取的都是最新的值；代码应该如下：

```java
public class MyClass {
    private int years;
    private int months
    private volatile int days;

    public int totalDays() {
        int total = this.days;
        total += months * 30;
        total += years * 365;
        return total;
    }

    public void update(int years, int months, int days){
        this.years  = years;
        this.months = months;
        this.days   = days;
    }
}
```

#### 3、volatile 的 Happens-Before 保证

面对指令重排（Instruction Reordering）时，`volatile` 关键字提供了 “Happens-Before” 保证：

- 如果对其它变量的读写发生在写 `volatile` 变量之前，那么对其它变量的读写不能重排到写 `volatile` 变量之后。即，保证对其它变量的读写发生在写 `volatile` 变量之前。但是，有可能写 `volatile` 变量之后的对其它变量的读写被重排到写 `volatile` 变量之前。
- 如果对其它变量的读写发生在读 `volatile` 变量之后，那么对其它变了读写不能被重排到读 `volatile` 变量之前。但是，在读 `volatile` 变量之前的对其它变量的读写可以被重排到读 `volatile` 变量之后。

happens-before 保证增强了 `volatile` 关键字的可见性保证。

#### 4、只有 `volatile` 可能还不够

如果只有一个线程去修改 `volatile` 变量的值，或者多个线程写 `volatile` 变量的值，但是 `volatile` 变量的值与它被更新之前的值无关，那么只有 `volatile` 就够了。

如果多个线程写 `volatile` 变量的值，并且 `volatile` 变量的更新值与之前的值相关，那么只用 `volatile` 就不能保证结果的正确了。单个线程读 `volatile` 变量和写 `volatile` 变量之间会有一个时间间隔，多个线程可能会读到 `volatile` 变量的相同值造成了一个**比赛状况（race condition）**，每个线程都会产生一个新的值，并且将新的值写到主内存时，会出现相互覆盖。比如，两个线程并发修改 `volatile counter` 正确结果应该是 `2`，但是并发状态下写到主内存的是 `1`，如下图：

![Two threads have read a shared counter variable into their local CPU caches and incremented it.](/assets/java-volatile-3.png)

此时，需要使用 `synchronized` 来保证对 `counter` 的读和写是原子性的。

除了 `synchronized` 关键字，还可以使用 `java.util.concurrent` 包中提供的原子性数据类型，比如 `AtomicLong` 等等。

对于 32 bit 和 64 bit 变量， `volatile` 关键字被保证是有效的。`long` 和 `double` 都是 64 bit的，默认情况下它们的写操作不是原子性的，并且因平台而异，某些平台分两次进行写，每次写 32 位数据，使用 `volatile` 则避免了这种情况。

#### 5、`volatile` 对性能的影响

只从主内存读写变量的值，相对于访问 CPU 缓存中的值当然是要慢的，访问 `volatile` 变量也会禁止指令重排，也会影响性能。只应在多线程且必要的时候才使用 `volatile` 变量。

#### 6、其它

JDK 5 才明确了 `volatile` 的语义。

JDK 5 的改变还有自动拆装包（Autoboxing），枚举类（Enum），泛型（Generic）和可变参数（Variable arguments）。

#### 7、`volatile` 和 `synchronized` 的区别

- `volatile` 关键字只能用于变量。

- `synchronized` 要获取和释放监控对象锁， `volatile` 不需要
- 使用 `synchronized` 时，线程会被阻塞以等待锁， `volatile` 不会阻塞
- `synchronized` 对性能影响更大
- 不能对 `null` 对象使用 `synchronized` ，但是可以对 `null` 对象使用 `volatile`
- 写 `volatile` 变量与锁释放有相同的内存影响，读 `volatile` 变量与锁获取有相同的内存影响。

#### 8、再看个例子

共享的多处理器架构（Shared Multiprocessor Architecture）——处理器负责执行程序指令。因此，它们需要从 RAM 中读取程序指令和必要的数据。CPUs 每秒能够处理大量地指令，所以从 RAM 直接获取指令是低效的。处理器会通过乱序执行（Out of Order Execution）、Branch Prediction，Speculative Execution，和缓存（Caching）来提升性能。但是，多线程中线程更新缓存的值时，会带来一些问题。

```java
public class TaskRunner {

    private static int number;
    private static boolean ready;

    private static class Reader extends Thread {

        @Override
        public void run() {
            while (!ready) {
                Thread.yield();
            }

            System.out.println(number);
        }
    }

    public static void main(String[] args) {
        new Reader().start();
        number = 42;
        ready = true;
    }
}
```

乍一看，程序应该在一段延时后输出 42。但是，事实上，延迟可能很长，或者程序一直挂起，或者输出 0。

这个例子中，有两个线程，主线程和 `Reader` 线程。如果 OS 在两个不同的 CPU 核心上运行这两个线程：

- 主线程的core cache 中会有一个 `ready` 和 `number` 的副本
- `reader` 线程的core cache 中也会有一个 `ready` 和 `number` 的副本
- 主线程更新缓存的值

对于最新的处理器，发起写操作后并不会立即实施。事实上，处理器倾向于将这些写操作加入到特殊的写缓冲队列中。过一段时间后，这些写操作会被一次性的写到主内存。主线程更新 `ready` 和 `number`时，不能保证 `reader`线程能看到。

另外，`reader` 线程可能看到的写`ready` 和 `number`的顺序与实际的编码顺序不同。因为，为了提升性能可能会有指令重排（reordering）。

为了保证对变量更新的传播对其它线程可预测，可以使用 `volatile`，这样可以告诉编译器和处理器不重排调用 `volatile` 变量的指令，处理器也会立即将更新的值刷到主内存。

多线程要确保：

- 互斥（Mutual Exclusion）——同一时间只有一个线程执行关键部分（critical section）
- 可见性（Visibility）——一个线程对共享数据的修改对其他线程可见，以保证数据的一致性

`synchronized` 方法和代码块，以应用性能为代价，提供了上述的保证。

`volatile` 可以确保数据改变的可见性方面，但是不提供互斥性。所以，它用在多线程可以并行执行一块代码，但是需要保证可见性的地方是合适的。

另外 `volatile` 还保证任何对`volatile` 变量的写操作都发生在后续对该变量的读操作之前（Happens-before）。

