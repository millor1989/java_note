### 单例模式

[原文链接](https://blog.51cto.com/devbean/203501)

单例模式——类在整个应用中只有一个实例存在。

#### 1、最简单实现

```java
public class SingletonClass {

  private static final SingletonClass instance = new SingletonClass();
    
  public static SingletonClass getInstance() {
    return instance;
  }
    
  private SingletonClass() {
    
  }
    
}
```

私有构造函数，提供一个私有 `final` 的静态属性，如果要获取类实例只能通过 `getInstance()` 方法获取，从而保证了只有一个实例存在。

存在的问题是——无论类是否被使用，都会创建一个实例（如果实例的创建开销很大，但是没有用到，显然是资源的浪费）。

#### 2、懒加载——Lazy loaded

```java
public class SingletonClass {

  private static SingletonClass instance = null;
    
  public static SingletonClass getInstance() {
    if(instance == null) {
      instance = new SingletonClass();
    }
    return instance;
  }
    
  private SingletonClass() {
    
  }
    
}
```

`instance` 变量初始化为 `null`，第一次使用的时候判断为 `null` 并创建对象，再次使用时，`instance` 变量为静态变量，已经被赋值不为 `null`，则直接返回。

存在的问题是——多线程情况下，可能会创建多个实例，从而单例失败。

#### 3、同步

```java
public class SingletonClass {

  private static SingletonClass instance = null;
    
  public synchronized static SingletonClass getInstance() {
    if(instance == null) {
      instance = new SingletonClass();
    }
    return instance;
  }
    
  private SingletonClass() {
    
  }
    
}
```

获取实例的方法经过同步后，只允许一个线程访问，从而保证了实例的唯一性。

存在的问题——性能！！同时多次调用 `getInstance()` 方法，性能会很差。

#### 4、锁改进

```java
public class SingletonClass {

  private static SingletonClass instance = null;
    
  public static SingletonClass getInstance() {
    synchronized (SingletonClass.class) {
      if(instance == null) {
        instance = new SingletonClass();
      }
    }    
    return instance;
  }
    
  private SingletonClass() {
    
  }
    
}
```

仅将检测 `null` 和创建实例的代码进行同步，也可以保证单例；但是，这样每次调用 `getInstance()` 方法时，还是要经过同步的代码块，性能问题亦然严重。

```java
public class SingletonClass {

  private static SingletonClass instance = null;

  public static SingletonClass getInstance() {
    if (instance == null) {
      synchronized (SingletonClass.class) {
        if (instance == null) {
          instance = new SingletonClass();
        }
      }
    }
    return instance;
  }

  private SingletonClass() {

  }

}
```

首先判断 `instance` 是否为 `null`，仅在首次 `instance` 为 `null` 时才经过同步代码块。

这就是双重验证加锁（double-checked locking）设计实现的单例模式。

#### 5、从编译原理说来开去

编译，把源代码解释为目标代码（通常是机器码）的过程。对 Java 来说，编译的目标代码不是机器代码，而是虚拟机代码。编译原理有一个重要内容——编译器优化，在不改变原来语义的情况下，通过调整语句顺序，来让程序运行更快，这个过程称为 reorder（instruction reorder）。

JVM 只是一个标准，并没有规定编译器优化的内容，所以说，JVM 实现可以自由地进行编译器优化。

创建变量的步骤，其一、要申请内存，调用构造方法进行初始化；其二、分配指针指向该内存。JVM 规范中没有规定该操作的顺序，所以，存在先将指针指向内存，再进行初始化（instruction reorder）的可能。那么，上面的双重验证加锁设计就有问题了——线程 1 开始创建 `SingletonClass` 实例时，线程 2 调用了 `getInstance()` 方法，判断 `instance` 是否为 `null`，如前所述，如果指针已经指向了一块内存，而实例初始化没有完成，线程 2 会得到一个没有初始化完成的 `instance`，虽然 `instance` 不为 `null`，但是不能正常使用，会引起程序错误。

为了避免错误，可以：

```java
public class SingletonClass {

  private static SingletonClass instance = null;

  public static SingletonClass getInstance() {
    if (instance == null) {
      SingletonClass sc;
      synchronized (SingletonClass.class) {
        sc = instance;
        if (sc == null) {
          synchronized (SingletonClass.class) {
            if(sc == null) {
              sc = new SingletonClass();
            }
          }
          instance = sc;
        }
      }
    }
    return instance;
  }

  private SingletonClass() {

  }
    
}
```

其中创建一个局部变量，通过局部变量进行代码创建，最后把 `instance` 指向局部变量的内存空间。

这个代码的思想基础是，`synchronized` 会起到一个代码屏蔽的作用，同步块内部代码和外部代码没有联系；因此，外部同步块里面对局部变量 `sc` 进行操作，并不影响 `instance`，所以外部类在 `instance=sc` 之前检测 `instance` 的时候，结果 `instance` 依然是 `null`。

但是，这种想法是**错误**的！！同步块只保证释放前同步块里的操作必须完成，当不保证同步块之后的操作不能因编译器优化而调换到同步块结束之前进行。因此，编译器完全可以把`instance=sc` 这句移动到内部同步块执行（instruction reorder），这样还是**错误**的。

#### 6、解决方案

对于JDK 5 之后，可以使用 `volatile` 关键字，`volatile` 关键字修饰的变量不会进行 instruction reorder，从而保证指向内存空间之前对象初始化完成。

```java
public class SingletonClass {

  private volatile static SingletonClass instance = null;

  public static SingletonClass getInstance() {
    if (instance == null) {
      synchronized (SingletonClass.class) {
        if(instance == null) {
          instance = new SingletonClass();
        }
      }
    }
    return instance;
  }

  private SingletonClass() {

  }
    
}
```

还有一种不受 Java 版本影响的方案（JDK 1.5 之前也适用，JDK 1.5 之前 `volatile` 是保留字，但是没有明确用途）：

```
public class SingletonClass {
    
  private static class SingletonClassInstance {
    private static final SingletonClass instance = new SingletonClass();
  }

  public static SingletonClass getInstance() {
    return SingletonClassInstance.instance;
  }

  private SingletonClass() {

  }
    
}
```

这里用到了静态内部类（Java 不支持静态类，但是支持内部类是静态的），是 JVM 明确说明了的，不存在任何歧义。这段代码中没有任何的 `staic` 属性，不会被初始化，调用 `getInstance()` 的时候，会首先加载 `SingletonClassInstance` 类，并调用 `SingletonClass` 的构造器初始化内部类的 `instance` 属性。并且，`instance` 属性是 `static` 的所以不会多次构造。`SingletonClassInstance` 是私有的静态内部类，对其它类不可见，`static` 语义表示他也不会有多个实例存在。JSL 规范定义，类的构造必须是原子性的、非并发的，也不需要加同步块，`getInstance()` 也就不需要加同步。

《Effective Java》 推荐使用第二种单例方案。