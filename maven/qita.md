- maven 仓库网站 https://mvnrepository.com，可以从该网站检索需要的 jar 包。

  如果，无法直接通过 maven 仓库下载 jar 包文件，可以通过 url [
  https://repo1.maven.org/maven2/](https://repo1.maven.org/maven2/) 直接下载需要的 jar 文件。

- `mvn dependency:tree` 查看 Maven 的依赖树

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

- `install` 安装 jar 文件到本地的 maven 仓库

  ```bash
  $ mvn install:install-file -DgroupId=org.apache.hadoop -DartifactId=hadoop-minicluster -Dversion=2.6.0 -Dpackaging=jar -Dfile=/f/Download/hadoop-minicluster-2.6.0.jar
  ```

  