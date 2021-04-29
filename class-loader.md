### Java虚拟机类加载机制

Java 虚拟机的类加载是指：虚拟机把描述类的数据从 class 文件加载到内存，对数据进行校验、转化解析和初始化，最终形成虚拟机可以直接使用的 Java 类型。

#### 1、类的加载过程

类的生命周期如下图：

![class-load](/assets/1240.png)

为了支持运行时绑定，解析过程再某些情况下可以在初始化之后再开始，除解析过程外的其它加载过程均严格按照上图的顺序进行。

##### 1.1、加载

1. 通过全限定类名来获取定义此类的的二进制字节流。
2. 将这个字节流所代表的静态存储结构转化为方法区的运行时数据结构。
3. 在内存中生成一个代表这个类的 `java.lang.Class` 对象，这个对象作为方法区这个类的各种数据访问的入口。

##### 1.2、验证

验证是连接阶段的第一步，目的是为了确保 class 文件的字节流包含的信息符合当前虚拟机的要求，并且不会危害虚拟机本身的安全。

1. **文件格式验证**：比如，是否以魔法数 `0xCAFEBABE` 开头，主、次版本号是否在当前虚拟机处理范围之内，常量合理性验证等。
2. **元数据验证**：是否存在父类，父类的继承链是否正确，抽象类是否实现了其父类或接口中要求实现的所有方法，字段、方法是否与父类产生矛盾等。
3. **字节码验证**：通过数据流和控制流分析，确定程序预计是合法的、符合逻辑的。例如，保证跳转指令不会跳转到方法体以外的字节码指令上。
4. **符号引用验证**：在解析阶段发生，保证可以将符号引用转化为直接引用。

可以考虑使用 `-Xverify:none` 参数来关闭大部分的类验证措施，这样可以缩短虚拟机类加载的时间。

##### 1.3、准备

为类变量分配内存并设置**类变量**的初始值，这些变量所使用的内存都将在方法区中进行分配。

##### 1.4、解析

虚拟机将常量池内的符号引用替换为直接引用的过程。

解析主要针对类或接口、字段、类方法、接口方法、方法类型、方法句柄和调用点限定符 7 类符号引用进行。

##### 1.5、初始化

初始化阶段才真正开始执行类中定义的 Java 代码，此阶段是执行 `<clinit>()` 方法的过程。

`<clinit>()` 方法是由编译器按语句在源文件中出现的顺序，依次自动收集类中的所有**类变量**的赋值动作和静态代码快中的语句合并产生的（不包括构造器中的语句，构造器是初始化对象的，类加载完成后，创建对象时候将调用 `<init>()` 方法来初始化对象）。

静态语句块只能访问到定义在静态语句块之前的变量；静态语句块可以为定义在它之后的变量赋值，但是不能访问，比如：

```java
public class Test {
    static {
        // 给变量赋值可以正常编译通过
        i = 0;
        // 这句编译器会提示"非法向前引用"
        System.out.println(i);
    }

    static int i = 1;
}
```

`<clinit>()` 不需要显式调用父类的初始化方法 `<clinit>()`（接口除外，接口不需要调用父接口的初始化方法，只有使用到富姐口中的静态变量时才需要调用），虚拟机会保证父类的 `<clinit>()` 方法会在子类的 `<clinit>()` 方法执行之前执行完毕，这也就意味着弗雷中定义的静态语句块要优先于字累的变量赋值操作。

`<clinit>()` 方法对于类或接口来说并不是必需的，如果一个类没有静态语句块，也没有对变量的赋值操作，那么编译器可以不为这个类生成 `<clinit>()` 方法。

虚拟机会保证一个类的 `<clinit>()` 方法在多线程环境中被正确地加锁、同步，如果多个线程同时去初始化一个类，那么只会有一个线程去执行这个类的 `<clinit>()` 方法，其它线程都会阻塞等待，直到活动线程执行 `<clinit>()` 方法完毕。

#### 2、类加载的时机

对于初始化阶段，虚拟机规范规定了有且仅有 5 种情况必须立即对类进行“初始化”（对应的加载、验证、准备当然要在次之前开始）：

1. 遇到`new` 、`getstatic`、`putstatic`、`invokestatic` 这 4 条字节码指令时，如果类没有进行过初始化，则需要先触发其初始化。对应场景是：使用 `new` 实例化对象、读取或设置一个类的静态字段（被 `final` 修饰、已在编译期把结果放入常量池的静态字段除外）、以及调用一个类的静态方法。
2. 对类进行反射调用的时候，如果类没有进行过初始化，则需要先触发其初始化。
3. 当初始化类的父类还没有进行过初始化，则需要先触发其父类的初始化。（接口在初始化时，不要求其父接口都完成了初始化）
4. 虚拟机启动时，用户需要指定一个要执行的主类（包含 `main()` 方法的类），虚拟机会首先初始化这个主类。
5. 使用 JDK 1.7 的动态语言支持时，如果一个 `java.lang.invoke.MethodHandle` 实例最后的解析结果是 `REF_getStatic`、`REF_putStatic`、`REF_invokeStatic` 的方法句柄，并且这个方法所对应的类没有进行过初始化，则需要先触发其初始化。

以上 5 种情况称为对一个类的**主动引用**。除此之外，所有引用类的方式都不会触发初始化，称为**被动引用**，比如：

1. 通过子类引用父类的静态字段，不会导致子类的初始化。
2. 通过数组定义来引用类，不会触发此类的初始化。
3. 常量在边一阶段会存入调用类的常量池中，本质上并没有直接引用到定义常量的类，也不会触发定义常量的类的初始化。

#### 3、类加载器

实现类加载阶段中“通过一个类的全限定名来获取描述此类的二进制字节流”这一行为的代码模块被称为**类加载器**。

将 class 文件二进制数据放入方法区内，然后在堆（heap）内创建一个 `java.lang.Class` 对象，Class 对象封装了类在方法区内的数据结构，并且向开发者提供了访问方法区内的数据结构的接口。

类加载器在类层次划分、OSGi、热部署、代码加密等领域非常重要，运行任意一个 Java 程序都涉及到类加载器。

##### 3.1、类的唯一性和类加载器

类本身和加载它的类加载器一起确立了类在 Java 虚拟机中的唯一性。

同一个 class 文件，被同一个 JVM 加载，如果是不同的类加载器进行加载，结果类也是不相等的（此处，“相等”包括代表类的 Class 对象的`equals()`方法、`isAssignableFrom()` 方法、`isInstance()` 方法的返回结果，也包括使用 `instanceof` 关键字做对象所属关系判定等情况）。

##### 3.2、双亲委派模型

类加载器收到了类的加载请求后，首先将请求委派给父加载器去加载，并且每一层次的类加载都是如此，这样所有的加载请求最终都应该传送到顶层的**启动类加载器**（Bootstrap ClassLoader）中，只有父加载器反馈无法完成请求（搜索范围内没有找到所需的类）时，子加载器才会尝试自己去加载。

![类加载模型](/assets/bulallaldfajsdakdskfj.png)

类加载器之间的父子关系不是以继承（Inheritance）来实现的，而是使用组合（Composition）关系来复用父类加载器的代码。

**Bootstrap 类加载器**是用 C++ 实现的，它是虚拟机自身的一部分，获取它的对象会返回 `null`；**扩展类加载器**和**应用类加载器**独立于虚拟机，是由 Java 语言实现，均继承子抽象类 `java.lang.ClassLoader`，可以直接使用这两个类加载器。

**Application 类加载器**可以通过 `ClassLoader.getSystemClassLoader()` 方法获取，一般也称它为**系统类加载器**（System ClassLoader）。它负责加载用户**类路径**（ClassPath）上指定的类库，如果应用程序没有自定义自己的类加载器，那么它就是程序默认的类加载器。

双亲委派模型对于保证 Java 程序的稳定运作很重要，比如，类 `java.lang.Object` 存放在 `rt.jar` 中，无论哪一个类加载器都要加载这个类，最终都是模型最顶层的 Bootstrap 类加载器进行加载，从而 `Object` 类在程序的各种类加载环境中都是同一个类。

双亲委派模型加载类的逻辑示例：

```java
    // 代码摘自《深入理解Java虚拟机》
    protected synchronized Class<?> loadClass(String name, boolean resolve) throws ClassNotFoundException {
        // 首先，检查请求的类是否已经被加载过了
        Class c = findLoadedClass(name);
        if (c == null) {
            try {
                if (parent != null) {
                    c = parent.loadClass(name, false);
                } else {
                    c = findBootstrapClassOrNull(name);
                }
            } catch (ClassNotFoundException e) {
            // 如果父类加载器抛出ClassNotFoundException
            // 说明父类加载器无法完成加载请求
            }
            if (c == null) {
                // 在父类加载器无法加载的时候
                // 再调用本身的findClass方法来进行类加载
                c = findClass(name);
            }
        }
        if (resolve) {
            resolveClass(c);
        }
        return c;
    }
```

##### 3.3、破坏双亲委派模型

1. 引入双亲委派模型之前，破坏它的代码已经存在了。

   双亲委派模型在 JDK 1.2 之后才引入，而类加载器和抽象类 `java.lang.ClassLoader` 则在 JDK 1.0 时代就存在，JDK 1.2 之后，其添加了一个新的`protected findClass()` 方法，在此之前，用去继承 `ClassLoader` 类的唯一目的就是重写它的`loadClass()` 方法，而双亲委派的具体逻辑就实现在这个方法之中，JDK 1.2 之后不提倡用户去重写`loadClass()` 方法，而应该将自己的类加载逻辑写到 `finaClass()` 方法中，这样可以保证新写的类加载器是符合双亲委派规则的。

2. 基础类无法调用类加载器加载用户提供的代码。

   双亲委派很好地解决了各个类加载器的基础类的统一问题（越基础的类由越上层的类加载器进行加载），但是如果基础类要调用用户的代码，比如 JNDI 服务（JNDI 是Java的标准服务，代码由启动类加载器加载），但是JNDI 的目的是对资源进行集中管理和查找，他需要条调用由独立厂商实现并不输在应用程序的 ClassPath 下的 JNDI 接口提供者（SPI，Service Provider Interface，例如，JDBC 驱动就是 MySQL 等接口提供者提供的）的代码，但启动类加载器只能加载基础类，不能加载用户类。

   为此，Java 引入了线程上下文加载器（Thread Context ClassLoader）。它可以通过`java.lang.Thread.setContextClassLoader()` 方法进行设置，如果创建线程时还没有设置，它会从父线程中继承一个，如果在应用程序的全局范围内都没有设置过的话，那么这个类加载器默认是应用程序类加载器。JNDI 服务使用这个线程上下文加载器去加载所需要的 SPI 代码，也就是父类加载器请求子类加载器去完成类加载的动作，这其实是打通了双亲委派模型的层次结构来你想使用类加载器，违背了双亲委派模型的一般原则，但是也是无奈之举。Java 中所有涉及 SPI 的加载动作级别都是采用这种方式，比如 JNDI、JDBC、JCE、JAXB和JBI等。

3. 用户对程序动态性的追求

   代码热替换（HotSwap）、模块热部署（Hot Deployment）等，OSGi（Open Service Gateway initiative，开放服务网关协议，OSGi 技术是指一系列用于定义 Java 动态化组件系统的的标准） 实现模块化热部署的关键则是它自定义的类加载器机制的实现。每一个程序模块（Bundle）都有一个自己的类加载器，需要更换 Bundle 时，就把 Bundle 联通类加载器一起替换掉，以实现代码的热替换。

   在 OSGi 环境下，类加载器不再是双亲委派模型中的树状结构，而是进一步发展为更加复杂的网状结构，当收到类加载请求时，OSGi 将按照下面的顺序进行搜索：

   - 将以 java.* 开头的类委派给父类加载器加载
   - 否则，将委派列表名单内的类委派给父类加载器加载
   - 否则，将 import 列表中的类委派给 Export 这个类的 Bundle 的类加载器加载
   - 否则，查找当前 Bundle 的 ClassPath，使用自己的类加载器加载
   - 否则，查找类是否在自给的 Fragment Bundle 中，如果在，则委派给 Fragment Bundle 的类加载器加载
   - 否则，查找 Dynamic Import 列表的 Bundle，委派给对应的 Bundle 的类加载器加载
   - 否则，类查找失败

   上面的查找顺序只有开头两点符合双亲委派规则，其余的查找都是在平级的类加载器中进行的。OSGi 的 Bundle 类加载器之间只有规则，没有对应的委派关系。

##### 3.4、自定义类加载器

Java 默认 ClassLoader 只加载指定目录下的 class，如果要动态加载类到内存，比如加载远程网络下载的类并调用这个类中的方法实现业务逻辑，就需要定义 ClassLoader。自定义类加载器需要——继承`java.lang.ClassLoader` 并重写`findClass()`方法。

之所以继承`ClassLoader`这个抽象类而不是继承`AppClassLoader`，是因为`AppClassLoader`和`ExtClassLoader`都是 Launcher 的静态内部类，其访问权限是默认的包访问权限。

```java
static class AppClassLoader extends URLClassLoader {...}
```

JDK 的`loadClass()` 方法在所有父类加载器无法加载的时候，会调用本身的`findClass()` 方法进行类加载，因此只用重写 `findClass()` 方法找到类的二进制数据即可。

一个简单的类加载器的例子：

```java
// 要被加载的类，存放于D盘根目录
public class Test {

    public static void test() {
        System.out.println("Test类已成功加载运行！");
        ClassLoader classLoader = Test.class.getClassLoader();
        System.out.println("加载我的classLoader：" + classLoader);
        System.out.println("classLoader.parent：" + classLoader.getParent());
    }
}

// 自定义的类加载器
import java.io.*;

public class MyClassLoader extends ClassLoader {
    @Override
    protected Class<?> findClass(String name) throws ClassNotFoundException {
        // 加载D盘根目录下指定类名的class
        String clzDir = "D:\\" + File.separatorChar
                + name.replace('.', File.separatorChar) + ".class";
        byte[] classData = getClassData(clzDir);

        if (classData == null) {
            throw new ClassNotFoundException();
        } else {
            return defineClass(name, classData, 0, classData.length);
        }
    }

    private byte[] getClassData(String path) {
        try (InputStream ins = new FileInputStream(path);
             ByteArrayOutputStream baos = new ByteArrayOutputStream()
        ) {

            int bufferSize = 4096;
            byte[] buffer = new byte[bufferSize];
            int bytesNumRead = 0;
            while ((bytesNumRead = ins.read(buffer)) != -1) {
                baos.write(buffer, 0, bytesNumRead);
            }
            return baos.toByteArray();
        } catch (IOException e) {
            e.printStackTrace();
        }
        return null;
    }
}
```

使用 `javac -encoding utf8 Test.java` 编译为 class 文件`Test.class`，使用自定义的类加载器进行加载：

```java
public class MyClassLoaderTest {
    public static void main(String[] args) throws Exception {
        // 指定类加载器加载调用
        MyClassLoader classLoader = new MyClassLoader();
        classLoader.loadClass("Test").getMethod("test").invoke(null);
    }
}
```

运行结果：

```
Test.test()已成功加载运行！
加载我的classLoader：class MyClassLoader@73846619
classLoader.parent：class sun.misc.Launcher$AppClassLoader@18b4aac2
```

##### 3.5、线程上下文加载器

默认的线程上下文加载器是 Application 类加载器，可以通过 `java.lang.Thread.setContextClassLoader()`方法进行设置：

```java
// Now create the class loader to use to launch the application
try {
    loader = AppClassLoader.getAppClassLoader(extcl);
} catch (IOException e) {
    throw new InternalError(
"Could not create application class loader" );
}
 
// Also set the context class loader for the primordial thread.
Thread.currentThread().setContextClassLoader(loader);
```

虽然，使用 `ClassLoader.getSystemClassLoader()` 方法也能获取 AppClassLoader，但是有时需要使用自定义类加载器去加载某个位置的类，比如 Tomcat 使用的线程上下文加载器并非 AppClassLoader，而是 Tomcat 自定义的类加载器。Tomcat 的每个 Web 应用都有一个对应的类加载器，该类加载器使用代理模式，首先尝试去加载某个类，如果找不到才交给父类加载器，这与一般类加载器的顺序是相反的。这也是 Java Servlet 规范中的推荐做法，目的是使得Web 应用自己的类的优先级高于 Web 容器提供的类。

#### 4、`new`一个对象

`new` 一个对象发生了如下过程：

1. 确认类元信息是否存在。

   当 JVM 收到`new` 指令，首先在 metaspace 内检查需要创建的类元信息是否存在。若不存在，则在双亲委派模型下，使用当前类加载器以 ClassLoader + 包名 + 类名 为 Key 进行查找对应的 class 文件。如果没有找到文件，则抛出 `ClassNotFoundException` 异常，如果找到，则进行类加载（加载 - 验证 - 准备 - 解析 - 初始化），并生成对应的 Class 类对象。

2. 分配对象内存。

   首先计算对象占用空间大小，如果实例成员变量是引用变量，仅分配引用变量空间即可（即 4 字节大小），接着在堆中划分一块内存给新对象。在分配内存空间时，需要进行同步操作，比如采用 CAS （Compare And Swap）失败重试、区域加锁等方式保证分配操作的原子性。

3. 设置默认值

   成员变量都需要设定默认值，即各种不同形式的零值。

4. 设置对象头

   设置新对象的哈希码、GC 信息、锁信息、对象所属的类元信息等。这个过程的具体设置方法取决于 JVM的实现。

5. 执行 init 方法

   初始化成员变量，执行实例化代码块，调用类的构造方法，并把堆内对象的首地址赋值给引用变量。

#### 5、Spring 中的类加载机制

Spring 使用的类加载器是自定义的类加载器 `OverridingClassLoader`，默认会自己先加载（ `excludedPackages` 或 `excludedClasses` 例外），只有加载不到才会委托给双亲加载，实际上是破坏了 JDK 的双亲委派模式。