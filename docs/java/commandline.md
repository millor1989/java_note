### 命令行工具

- #### `java`

  - `java -jar <jar 文件路径>`

    执行指定 jar 文件的主类（jar 文件 `MANIFEST.MF` 文件中的 `Main-Class` 指定的类）。

  - `java -classpath <jar \ class 文件路径> <全限定名>`

    在指定路径中 查找 class 文件并执行。class 路径可以是相对路径，比如 `.` （当前所在目录）。可以指定多个 jar 文件，jar 文件直接使用 `:` 分隔。

    缩写形式 `java -cp`

    `java <全限定名>` 形式：如果配置了 `CLASSPATH` 环境变量，则可以省略 `-classpath`，即，此时在 `CLASSPATH` 环境变量中查找指定的 class 文件。如果没有配置 `CLASSPATH` 环境变量，则是在默认的当前所在目录中查找 class 文件。

- #### `javac`

  编译工具

- #### `jmap`

  查看指定 Java 进程的共享对象内存映射或堆内存细节

- #### `jps`

  Java 虚拟机进程状态工具（Java Virtual Machine Process Status Tool）；列出当前正在运行的 JVM 进程信息（只能显示权限范围内 JVM 进程信息）。

  - ##### 显示 JVM 进程：

    ```bash
    # jps
    15729 jar
    92153 Jps
    90267 Jstat
    ```

  - ##### 显示主类的全限定名或 JAR 文件名：

    ```bash
    # jps -l
    15729 one-more-1.0.0.RELEASE.jar
    112054 sun.tools.jps.Jps
    90267 sun.tools.jstat.Jstat
    ```

  - ##### 显示主类的全限定名或 JAR 文件名，并显示 JVM 参数：

    ```bash
    # jps -lv
    15729 one-more-1.0.0.RELEASE.jar -Xmx1g -Xms1g -Xmn512m 
    9043 sun.tools.jps.Jps -Denv.class.path=.:/usr/local/java/jdk1.8.0_251/lib:/usr/local/java/jdk1.8.0_251/jre/lib -Dapplication.home=/usr/local/java/jdk1.8.0_251 -Xms8m
    ```

  - ##### 显示主类的全限定名或 JAR 文件名，并显示传递给 `main()` 方法的参数：

    ```bash
    # jps -lm
    15729 one-more-1.0.0.RELEASE.jar
    59014 sun.tools.jps.Jps -lm
    90267 sun.tools.jstat.Jstat -gc 15729 1000
    ```

- #### `jstack`

  查看 JVM 线程快照