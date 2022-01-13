### 队列（Queue）

先进先出，只能在末尾追加数据，从头部取出数据，不能再中间插入或提取数据，就是典型的队列（queue）结构。

队列的两个基本操作：入队（enqueue()，将数据加到队列尾部）、出队（dequeue()，从队列头部取数据）。

![img](/assets/9eca53f9b557b1213c5d94b94e9dce3e.jpg)

队列与栈类似，也是一种**操作受限的线性表数据结构**。

队列作为非常基础的数据结构，应用非常广泛，尤其是那些具有额外特性的队列，比如循环队列、阻塞队列、并发队列。在很多偏低层系统、框架、中间件的开发中，起着关键性的作用。比如高性能队列 Disruptor、Linux 环形缓存，都用到了循环并发队列；Java concurrent 并发包利用 ArrayBlockingQueue 来实现公平锁等。

#### 1、队列的实现

队列可以用数组来实现，也可以用链表来实现。用数组实现的队列叫作**顺序队列**，用链表实现的队列叫作**链式队列**。

顺序队列的 Java 实现：

```java
// 用数组实现的队列
public class ArrayQueue {
  // 数组：items，数组大小：n
  private String[] items;
  private int n = 0;
  // head表示队头下标，tail表示队尾下标
  private int head = 0;
  private int tail = 0;

  // 申请一个大小为capacity的数组
  public ArrayQueue(int capacity) {
    items = new String[capacity];
    n = capacity;
  }

  // 入队
  public boolean enqueue(String item) {
    // 如果tail == n 表示队列已经满了
    if (tail == n) return false;
    items[tail] = item;
    ++tail;
    return true;
  }

  // 出队
  public String dequeue() {
    // 如果head == tail 表示队列为空
    if (head == tail) return null;
    // 为了让其他语言的同学看的更加明确，把--操作放到单独一行来写了
    String ret = items[head];
    ++head;
    return ret;
  }
}
```

与顺序栈的实现（只需要一个栈顶指针）相比，队列需要两个指针：一个是 head 指针，指向队头；一个是 tail 指针，指向队尾。

当 a、b、c、d 依次入队之后，队列中的 head 指针指向下标为 0 的位置，tail 指针指向下标为 4 的位置。

![img](/assets/5c0ec42eb797e8a7d48c9dbe89dc93cb.jpg)

当调用两次出队操作之后，队列中 head 指针指向下标为 2 的位置，tail 指针仍然指向下标为 4 的位置。

![img](/assets/dea27f2c505dd8d0b6b86e262d03430d.jpg)

可以发现，随着不停地进行入队、出队操作，head 和 tail 都会持续往后移动。当 tail 移动到最右边，即使数组中还有空闲空间，也无法继续往队列中添加数据了。要解决这个问题，最直接的方案就是数据搬移。但是，如果每次进行出队操作（相当于删除数组下标为 0 的数据）都搬移整个队列中的数据，这样出队操作的时间复杂度就会从原来的 O(1) 变为 O(n)。其实不需要每次出队的时候都搬移数据，只有在入队时没有了空闲空间的时候才需要触发数据搬移操作，其代码实现为：

```java
   // 入队操作，将item放入队尾
  public boolean enqueue(String item) {
    // tail == n表示队列末尾没有空间了
    if (tail == n) {
      // tail ==n && head==0，表示整个队列都占满了
      if (head == 0) return false;
      // 数据搬移
      for (int i = head; i < tail; ++i) {
        items[i-head] = items[i];
      }
      // 搬移完之后重新更新head和tail
      tail -= head;
      head = 0;
    }
    
    items[tail] = item;
    ++tail;
    return true;
  }
```

这种实现思路，出队操作的时间复杂度为 O(1)，入队的时间复杂度最坏情况下为 O(n)，最后情况下为 O(1)，入队的均摊时间复杂度为 O(1)。

基于链表的队列实现，同样需要两个指针：head 指针和 tail 指针。它们分别指向链表的第一个结点和最后一个结点。

![img](/assets/c916fe2212f8f543ddf539296444d393.jpg)

入队时，`tail->next= new_node`, `tail = tail->next`；出队时，`head = head->next`。

#### 2、循环队列

用数组来实现队列的时候，在 `tail==n` 时，会有数据搬移操作，这样入队操作性能就会受到影响。使用循环队列可以避免数据搬移。

循环队列，逻辑上是一个环：

![img](/assets/58ba37bb4102b87d66dffe7148b0f990.jpg)

图中这个队列的大小为 8，当前 head=4，tail=7。当有一个新的元素 a 入队时，放入下标为 7 的位置。但这个时候并不把 tail 更新为 8，而是将其在环中后移一位，到下标为 0 的位置。当再有一个元素 b 入队时，将 b 放入下标为 0 的位置，然后 tail 加 1 更新为 1。所以，在 a，b 依次入队之后，循环队列中的元素就变成了下面的样子：

![img](/assets/71a41effb54ccea9dd463bde1b6abe80.jpg)

这样就成功避免了数据搬移操作。

在用数组实现的非循环队列中，队满的判断条件是 tail == n，队空的判断条件是 head == tail。对于循环队列，队列为空的判断条件仍然是 head == tail，而队满的时候情况就复杂了：

![img](/assets/3d81a44f8c42b3ceee55605f9aeedcec.jpg)

经过分析，会发现，队满的时候 `(tail+1)%n=head`。

可以发现，这种方式实现的循环队列在队满的时候， tail 指向的位置实际上是没有存储数据的。即这种循环队列实现会浪费数组的一个存储空间。

循环队列 Java 代码实现：

```java
public class CircularQueue {
  // 数组：items，数组大小：n
  private String[] items;
  private int n = 0;
  // head表示队头下标，tail表示队尾下标
  private int head = 0;
  private int tail = 0;

  // 申请一个大小为capacity的数组
  public CircularQueue(int capacity) {
    items = new String[capacity];
    n = capacity;
  }

  // 入队
  public boolean enqueue(String item) {
    // 队列满了
    if ((tail + 1) % n == head) return false;
    items[tail] = item;
    tail = (tail + 1) % n;
    return true;
  }

  // 出队
  public String dequeue() {
    // 如果head == tail 表示队列为空
    if (head == tail) return null;
    String ret = items[head];
    head = (head + 1) % n;
    return ret;
  }
}
```

#### 3、阻塞队列

阻塞队列在队列基础上增加了阻塞操作。简单来说，就是在队列为空的时候，从队头取数据会被阻塞。因为此时还没有数据可取，直到队列中有了数据才能返回；如果队列已经满了，那么插入数据的操作就会被阻塞，直到队列中有空闲位置后再插入数据，然后再返回。

![img](/assets/5ef3326181907dea0964f612890185eb.jpg)

使用阻塞队列，可以轻松实现一个“生产者 - 消费者模型”，这种基于阻塞队列实现的“生产者 - 消费者模型”，可以有效地协调生产和消费的速度。当“生产者”生产数据的速度过快，“消费者”来不及消费时，存储数据的队列很快就会满了。这时，生产者就阻塞等待，直到“消费者”消费了数据，“生产者”才会被唤醒继续“生产”。而且不仅如此，基于阻塞队列，还可以通过协调“生产者”和“消费者”的个数，来提高数据的处理效率。比如，可以多配置几个“消费者”，来应对一个高效的“生产者”。

![img](/assets/9f539cc0f1edc20e7fa6559193898067.jpg)

#### 4、并发队列

线程安全的队列我们叫作并发队列。最简单直接的实现方式是直接在 enqueue()、dequeue() 方法上加锁，但是锁粒度大并发度会比较低，同一时刻仅允许一个存或者取操作。实际上，基于数组的循环队列，利用 CAS（Compare And Swap）原子操作，可以实现非常高效的并发队列。这也是循环队列比链式队列应用更加广泛的原因。

#### 5、线程池调度策略

线程池没有空闲线程时，新的任务请求线程资源时，一般有两种处理策略。第一种是非阻塞的处理方式，直接拒绝任务请求；另一种是阻塞的处理方式，将请求排队，等到有空闲线程时，取出排队的请求继续处理。

存储排队的请求时，如果希望公平地处理每个排队的请求，先进者先服务，那么队列这种数据结构很适合来存储排队请求。队列有基于链表和基于数组这两种实现方式。

基于链表的实现方式，可以实现一个支持无限排队的无界队列（unbounded queue），但是可能会导致过多的请求排队等待，请求处理的响应时间过长。所以，针对响应时间比较敏感的系统，基于链表实现的无限排队的线程池是不合适的。

而基于数组实现的有界队列（bounded queue），队列的大小有限，所以线程池中排队的请求超过队列大小时，接下来的请求就会被拒绝，这种方式对响应时间敏感的系统来说，就相对更加合理。不过，设置一个合理的队列大小，也是非常有讲究的。队列太大导致等待的请求太多，队列太小会导致无法充分利用系统资源、发挥最大性能。

队列除了应用在线程池请求排队的场景，还可以应用在任何有限资源池中，用于排队请求，比如数据库连接池等。**实际上，对于大部分资源有限的场景，当没有空闲资源时，基本上都可以通过“队列”这种数据结构来实现请求排队。**

#### 6、无锁并发队列

head 和 tail 使用 `AtomicInteger` 类型，利用 CAS 原理实现无锁的原子性操作。