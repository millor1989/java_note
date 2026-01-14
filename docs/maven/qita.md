- maven 仓库网站 https://mvnrepository.com，可以从该网站检索需要的 jar 包。

  如果，无法直接通过 maven 仓库下载 jar 包文件，可以通过 url [
  https://repo1.maven.org/maven2/](https://repo1.maven.org/maven2/) 直接下载需要的 jar 文件。

- `mvn dependency:tree` 查看 Maven 的依赖树

  **局限性**：标准的 `mvn dependency:tree` 命令主要分析 `dependency` 标签中声明的依赖及其传递依赖（**IDEA 的 Dependency Analyzer 插件也是这个原理**），对于 `type` 为 `war` 的依赖（maven overlay 项目中），它无法直接展开其内部的库（传递依赖）。

  - 确认最终 jar / war 包是否包含某个库，最靠谱的还是打开包检查：

  - **maven overlay**

    主依赖是一个 type=war 的 Overlay 包。Maven 在打包时，会先解压这个 war 包内的所有库到 target 目录，然后再用项目代码和配置进行覆盖。`dependencyManagement` 标签也无法覆盖或替换 overlay 的 war 包内部的库。排除 overlay war 中的包，只能通过 `maven-war-plugin` 实现：

    ```xml
    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-war-plugin</artifactId>
        <version>2.6</version>
        <configuration>
            <warName>cas-server</warName>
            <failOnMissingWebXml>false</failOnMissingWebXml>
            <recompressZippedFiles>false</recompressZippedFiles>
            <archive>
                <compress>false</compress>
                <manifestFile>${manifestFileToUse}</manifestFile>
            </archive>
            <overlays>
                <overlay>
                    <groupId>org.apereo.cas</groupId>
                    <artifactId>cas-server-webapp${app.server}</artifactId>
                    <!-- 排除cas-server-webapp 中的 Log4j 包 -->
                    <excludes>
                        <exclude>**/log4j-*.jar</exclude>
                        <exclude>**/log4j-web-*.jar</exclude>
                    </excludes>
                </overlay>
            </overlays>
        </configuration>
    </plugin>
    ```

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

- `deploy`  jar 文件到 maven 中央仓库

  ```bash
  mvn deploy:deploy-file -DgroupId=com.bytedance.bytehouse-ce -DartifactId=clickhouse-spark-runtime-3.5_2.12 -Dversion=0.8.1.1 -Dpackaging=jar -Dfile=F:\Download\clickhouse-spark-runtime-3.5_2.12-0.8.1.1.jar -Durl=http://maven.remote.com/repository/maven-releases/ -DrepositoryId=releases
  ```

  `status code: 401, reason phrase: Unauthorized (401)` 错误表明 Maven 部署时未通过 Nexus 服务器的身份验证，需重确保 settings.xml 中配置的 &lt;server&gt; 的 id 与命令中的 `-DrepositoryId=nexus-snapshot` 完全一致，且用户名密码正确。
  
  `status code: 405, reason phrase: PUT (405) ` 表明 Maven 尝试向 Nexus 仓库执行 PUT 操作时被拒绝，通常由仓库类型配置错误导致：要确保目标仓库为 `hosted` 类型而非 `proxy` 或 `group` 类型。Nexus 默认的 maven-public 是仓库组（`group` 类型），直接向其部署会触发 405 错误，应改用具体的 maven-releases 或 maven-snapshots 等 `hosted` 仓库。

- maven overlay

  