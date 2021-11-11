### Maven

Maven是一个Java项目管理和构建工具，它可以定义项目结构、项目依赖，并使用统一的方式进行自动化构建。

主要功能：

- 提供了一套标准化的项目结构；
- 提供了一套标准化的构建流程（编译，测试，打包，发布……）；
- 提供了一套依赖管理机制。

Maven使用`pom.xml`定义项目内容，并使用预设的目录结构；在Maven中声明一个依赖项可以自动下载并导入classpath；Maven使用`groupId`，`artifactId`和`version`唯一定位一个依赖。





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

