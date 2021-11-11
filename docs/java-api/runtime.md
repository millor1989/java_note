### Java.lang.Runtime

每个Java应用都有一个`Runtime`类的实例，通过它应用可以和所运行在的环境进行交互。

应用当前的runtime可以通过`getRuntime`方法获取。

#### 1、`public static Runtiem getRuntime()`

返回与当前Java应用相关的`Runtime`对象。

```java
public class GFG 
{ 
    public static void main(String[] args) 
    { 
        // get the current runtime assosiated with this process 
        Runtime run = Runtime.getRuntime(); 
        // print the current free memory for this runtime 
        System.out.println("" + run.freeMemory()); 
    } 
} 
```

其中`Runtime`实例的`public long freeMemory()`方法返回 JVM 中空闲内存的大小，`public long totalMemory()`方法返回 JVM 总内存大小。

#### 2、`public Process exec(String command) thorws IOException`

在独立的进程中执行指定命令。

```java
public class GFG 
{ 
    public static void main(String[] args) 
    { 
        try
        { 
            // create a process and execute google-chrome 
            Process process = Runtime.getRuntime().exec("google-chrome"); 
            System.out.println("Google Chrome successfully started"); 
        } 
        catch (Exception e) 
        { 
            e.printStackTrace(); 
        } 
    } 
}
```

这段代码，是Linux环境下打开google-chrome。Windows系统下，通过执行`Runtime.getRuntime().exec("notepad")`可以打开记事本。

#### 3、public void addShutdownHook(Thread hook)

这个方法注册一个新的虚拟机[shutdown hook](https://www.geeksforgeeks.org/jvm-shutdown-hook-java/)线程。

shutdown hooks是一个特殊的结构（construct），通过它可以插入一些当JVM关闭时执行的代码。使用它可以在VM关闭时执行一些特殊的清理操作。一般情况下，可以在应用退出之前（调用`System.exit(0)`）调用特殊的处理，但是这种方式在外部原因（比如，系统的kill请求——一般地`kill`命令，没有`-9`）或者资源问题（比如，内存溢出）导致关闭的情况下将不起作用。shutdown hooks很容易解决这种问题

