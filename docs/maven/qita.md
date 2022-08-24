
`mvn dependency:tree` 查看 Maven 的依赖树



- 页面出现乱码，可以在pom文件中设置属性，统一使用UTF-8编码（也能解决maven build时，提示 `xxxxx 编码GBK的不可映射字符` 的警告）：

```xml
    <properties>
		<!-- 文件拷贝时的编码 -->
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <!-- 编译时的编码 -->
        <maven.compiler.encoding>UTF-8</maven.compiler.encoding>
	</properties>
```

