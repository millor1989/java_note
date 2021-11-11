### synchronized

对同一个对象的所有 `synchronized` 代码块，同时只能有一个线程在同步代码块中执行。其它尝试进入同步代码块的线程都会被阻塞，直到同步代码块中的线程退出代码块。

`synchronized` 关键字可以修饰 4 种不同类型的代码块：

1. 实例方法
2. 静态方法
3. 实例方法中的代码块
4. 静态方法中的代码块

#### 1、同步实例方法

```java
public class MyCounter {

  private int count = 0;

  public synchronized void add(int value){
      this.count += value;
  }
}
```

Java 中同步实例方法是对拥有该方法的实例（对象）同步的。每个实例只能有一个线程可以在同步实例方法中执行。对于同一个实例（对象）的所有同步实例方法来说，也是一样的，即，整个实例中只能有一个线程执行。

```java
public class MyCounter {

  private int count = 0;

  public synchronized void add(int value){
      this.count += value;
  }
  public synchronized void subtract(int value){
      this.count -= value;
  }

}
```

#### 2、同步静态方法

```java
public class MyStaticCounter{

  private static int count = 0;

  public static synchronized void add(int value){
      count += value;
  }

}
```

同步静态方法是类级别的同步。由于 JVM 中每个类只能有一个，JVM 中只能有一个线程能在同步静态方法中执行。如果类有多个同步静态方法，整个类也只能有一个线程在同步静态方法中执行。

```java
public class MyStaticCounter{

  private static int count = 0;

  public static synchronized void add(int value){
    count += value;
  }

  public static synchronized void subtract(int value){
    count -= value;
  }
}
```

#### 3、实例方法中的静态代码块

多数时间，不用同步整个方法，只用同步方法的一部分即可。

```java
  public void add(int value){

    synchronized(this){
       this.count += value;   
    }
  }
```

本例中，同步代码结构使用了 `this`，即 `add` 方法被调用的实例。同步结构的括号中使用的对象叫作**监控对象**（monitor object）。同步代码块会基于监控对象进行同步。同步实例方法使用它所属的对象作为监控对象。

如下代码中，两个同步结构都是对它们被调用的实例同步的：

```java
  public class MyClass {
  
    public synchronized void log1(String msg1, String msg2){
       log.writeln(msg1);
       log.writeln(msg2);
    }

  
    public void log2(String msg1, String msg2){
       synchronized(this){
          log.writeln(msg1);
          log.writeln(msg2);
       }
    }
  }
```

同一时间，对于同一个实例，只能有一个线程在这两个同步结构中执行。

如果，第二个同步块对不同的对象同步，而不是`this`，那么这两个同步块可以同时执行。

#### 4、静态方法中的同步代码块

```java
  public class MyClass {

    public static synchronized void log1(String msg1, String msg2){
       log.writeln(msg1);
       log.writeln(msg2);
    }

  
    public static void log2(String msg1, String msg2){
       synchronized(MyClass.class){
          log.writeln(msg1);
          log.writeln(msg2);  
       }
    }
  }
```

同步是针对 `MyClass` 类的，同时只能有一个线程在这两个同步结构中执行，如果第二个代码块的监控对象不是 `MyClass.class`， 则互不影响。

#### 5、Lambda 表达式中的同步块

在匿名类和 Java Lambda 表达式中也能使用同步块。

```java
import java.util.function.Consumer;

public class SynchronizedExample {

  public static void main(String[] args) {

    Consumer<String> func = (String param) -> {

      synchronized(SynchronizedExample.class) {

        System.out.println(
            Thread.currentThread().getName() +
                    " step 1: " + param);

        try {
          Thread.sleep( (long) (Math.random() * 1000));
        } catch (InterruptedException e) {
          e.printStackTrace();
        }

        System.out.println(
            Thread.currentThread().getName() +
                    " step 2: " + param);
      }

    };

    Thread thread1 = new Thread(() -> {
        func.accept("Parameter");
    }, "Thread 1");

    Thread thread2 = new Thread(() -> {
        func.accept("Parameter");
    }, "Thread 2");

    thread1.start();
    thread2.start();
  }
}
```

此处的同步块是对类同步的，也可以对其它对象同步。

#### 6、`synchronized` 和数据可见性

不使用 `synchronized` 关键字（或者 Java `volatile` 关键字）将无法保证一个线程修改了与其它线程共享的变量（比如，一个所有线程都能访问的对象）的值之后其它的线程可以看到改变后的值。无法保证何时保存在某个线程的 CPU 寄存器（register）中的变量值被“提交”到主内存（main memory）中，并且无法保证何时其它线程从主内存“刷新”它们 CPU 寄存器中的变量值。

`synchronized` 关键字改变了这种情况，当线程进入到同步块中时，它会刷新对该线程可见的所有变量的值；当一个线程从同步块退出，对这个线程可见的所有的变量的值都会被提交到主内存。

#### 7、`synchronized` 和 指令重排（Instruction Reordering）

Java 编译器和 JVM 允许通过重排代码指令来使代码运行更快，但是多线程执行时，指令重排可能会导致问题。比如，一个在同步块中发生的写变量指令被重排到同步块的外面。

为了修正这个问题，Java `synchronized` 关键字对同步块前、中、后的指令的重排有一些限制。所以，可以确定代码会正常运行——代码重排不会导致代码运行与预期不同。

#### 8、对什么对象同步？

同步块必须是基于某些对象的，事实上，可以选择对任何对象同步，但是最好不要对 `String` 对象同步，也不要对任何基本类型的包装（wrapper）类对象同步，因为编译器可能会进行优化，最终导致在代码中的本以为使用的是不同实例的不同地方使用的是相同的实例。

```java
synchronized("Hey") {
   //do something in here.
}
```

在多处不同代码块中使用字符串 `Hey`，其实是基于同一个对象同步的。使用基本类型包装类对象也是一样的。

#### 9、同步块的限制和其它选择

Java `Synchronized` 代码块有一些限制，比如，一个 `Synchronized` 代码块同时只允许一个线程进入。但是，如果想要两个线程都读取一个共享变量，并且不会更新这个共享变量呢？此时，多个线程同时进入也是安全的。此时，可以使用 Read / Write Lock，Java 有一个内置的 `ReadWriteLock` 类。

如果要允许多个线程进入同步块，还可以使用 `Semaphore` 类。

`synchronzied` 块不保证线程访问的顺序，如果要保证线程访问的顺序，可以实现[ `Fairness` ](http://tutorials.jenkov.com/java-concurrency/starvation-and-fairness.html)。

如果只有一个线程写共享变量，其它线程只会读这个变量，那么可以使用 `volatile` 变量，而不用任何同步块。

#### 10、同步块性能开销

Java 中进出同步代码块有些性能开销。随着 Java 的发展，该开销在降低，但是还是存在。

不要使用大型的同步块，除非必要。同步块中仅保留必需的指令。

#### 11、同步块的重入（Reentrance）

当线程进入一个同步块后，线程被称为持有（同步块的监控对象的）锁（hold the lock）。如果线程调用了另一方法，该方法又调用了先前调用的同步方法，那么线程持有的锁可以重入（reenter）同步块。线程不会因为自身持有锁而被阻塞，只有不同线程会被阻塞。

```java
public class MyClass {
    
  List<String> elements = new ArrayList<String>();
    
  public void count() {
    if(elements.size() == 0) {
        return 0;
    }
    synchronized(this) {
       elements.remove();
       return 1 + count();  
    }
  }
    
}
```

递归调用同步块，一个线程重入同步块多次，这也是允许的。

要记住，设计一个线程进入多个同步块可能导致 [Nested Monitor Lockout](http://tutorials.jenkov.com/java-concurrency/nested-monitor-lockout.html)。

#### 12、集群中的同步块

同步块只阻塞同一个 JVM 中的线程。