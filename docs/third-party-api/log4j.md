### Log4j

Log4j是Java编程语言中广泛使用的一个日志记录（Logging）工具。它是Apache软件基金会的一个开源项目，旨在帮助开发者在应用程序中方便地实现灵活的日志记录功能。

Log4j 1.x 已经不再维护，取而代之的是 Log4j 2.x。

配置文件默认为 classpath 中的 `log4j.properties` 或 `log4j.xml`。可通过属性：`log4j.configuration`（Log4j 1.x ）或 `log4j.configurationFile`（Log4j 2.x）另行指定。

#### Spring web.xml 中动态指定 Log4j 配置文件

```xml
<context-param>
    <param-name>log4jConfigLocation</param-name>
    <param-value>classpath:/log4j-${ENV}.properties</param-value>
</context-param>
<listener>
    <listener-class>org.springframework.web.util.Log4jConfigListener</listener-class>
</listener>
```

`${ENV}` 可以获取系统环境变量，根据变量的值加载不同的配置文件。

Spring 官方不提倡这种配置方式。`org.springframework.web.util.Log4jConfigListener` 已被标注为 `@Deprecated`。

除了这种**运行时**动态加载配置文件的方式，还可以不同运行时设置不同的 `log4j.configuration` 来加载不同的 Log4j 配置文件。

另外，还可以通过 Maven profile 在**编译时**，打包不同的配置文件。

### Logback

Log4j 创始人开发，旨在取代 Log4j 1.x。

### JUL（java.util.logging）

JDK 自带的日志工具类，配置文件 `logging.properties`，可以在代码中指定配置文件，也可以通过 `Djava.util.logging.config.file=logging.properties` 指定。

### Commons-logging

日志接口，自带了 `SimpleLog` 实现类。

### SLF4j

日志接口

### PlumeLog