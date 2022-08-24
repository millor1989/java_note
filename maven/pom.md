#### 1、POM

pom 文件 `pom.xml` 包含了项目的基本信息，用于描述项目如何构建，声明项目依赖，等等。执行任务或目标时，Maven 会在当前目录中查找 POM。它读取 POM，获取所需的配置信息，然后执行目标。

所有的 pom 文件中都必须有 `project`、`groupId`、`artifactId`、`version`。

##### 1.1、父（Super）POM

父 POM 是 Maven 默认的 POM。在 Maven 目录的 `lib` 目录下有一个 `maven-model-builder-3.x.x.jar`，其中的 `pom-4.0.0.xml` 文件就是父 POM。所有的 POM 都继承自父 POM（无论是否显式定义了这个父 POM）。父 POM 包含了一些可以被继承的默认设置。因此，当 Maven 发现需要下载 POM 中的依赖时，它会到 Super POM 中配置的默认仓库（repo1.maven.org）去下载。

Maven 使用 effective pom（Super pom 加上工程自己的配置）来执行相关的目标，它帮助开发者在 pom.xml 中做尽可能少的配置，当然这些配置可以被重写。

#### 2、依赖管理

##### 2.1、可传递性依赖发现（Transitive Dependencies）

 A 依赖于其他库 B。如果，另外一个项目 C 想要使用 A ，那么 C 项目也需要使用库 B。Maven 通过读取项目文件（pom.xml），找出项目之间的依赖关系。当 C 项目 pom 中只配置了对 A 的依赖后，Maven 会自动地（隐式地）引入对项目 B 的依赖。

maven 有对依赖可传递性的控制：

1. 依赖调节：两个依赖版本在依赖树里的深度是一样的时候，第一个被声明的依赖将会被使用。
2. 依赖排除：使用 `exclusion` 排除传递的依赖
3. 可选依赖：使用 `optional` 将可传递依赖标记为可选。

##### 2.2、依赖关系（scope）

1. compile：编译时需要用到的依赖（默认），maven 会将这类依赖直接放入 classpath。
2. test：只在测试时用到的依赖。比如 JUnit。
3. runtime：在编译阶段不是必须的，但是在执行阶段是必须的。比如 jdbc 驱动。
4. provided：编译时需要，但是运行时不需要。比如 Servlet API，编译时需要，但是运行时由于 Servlet 服务器内置了相关依赖，就不需要了。**p.s.** 打包项目时，如果 jar A 标记为 `provided`，但是同一个 pom 中依赖的 jar B 依赖了 jar A，而 jar B 不是 `provided` 的，那么还是会将 jar A 打包。

##### 2.3、Maven 仓库

Maven 维护了一个中央仓库（repo1.maven.org）。项目构建时，会从中央仓库下载所需的依赖。Maven 不会每次都从中央仓库下载依赖，下载的依赖会缓存在本地 `.m2` 目录。

如果 Maven 默认的中央仓库比较慢，可以使用速度较快 Maven 镜像库。修改 `.m2` 目录的 Maven 配置文件 `setting.xml` ：

```xml
<settings>
    <mirrors>
        <mirror>
            <id>aliyun</id>
            <name>aliyun</name>
            <mirrorOf>central</mirrorOf>
            <!-- 阿里云的Maven镜像 -->
            <url>https://maven.aliyun.com/repository/central</url>
        </mirror>
    </mirrors>
</settings>
```

#### 3、构建生命周期

Maven 构建生命周期（lifecycle）定义了一个项目构建和发布的过程。

Maven 有以下三个标准的生命周期：

- **clean**：项目清理
- **default/build**：项目部署
- **site**：项目站点文档创建

一个典型的 Maven 构建（build）生命周期是由以下几个阶段（phase）的序列组成的：

```
validate
initialize
generate-sources
process-sources
generate-resources
process-resources
compile
process-classes
generate-test-sources
process-test-sources
generate-test-resources
process-test-resources
test-compile
process-test-classes
test
prepare-package
package
pre-integration-test
integration-test
post-integration-test
verify
install
deploy
```

部分阶段的描述：

| 阶段     | 处理     | 描述                                                     |
| -------- | -------- | -------------------------------------------------------- |
| validate | 验证项目 | 验证项目是否正确且所有必须信息是可用的                   |
| compile  | 执行编译 | 源代码编译在此阶段完成                                   |
| test     | 测试     | 使用适当的单元测试框架（例如JUnit）运行测试。            |
| package  | 打包     | 创建JAR/WAR包，如在 pom.xml 中定义提及的包               |
| verify   | 检查     | 对集成测试的结果进行检查，以保证质量达标                 |
| install  | 安装     | 安装打包的项目到本地仓库，以供其他项目使用               |
| deploy   | 部署     | 拷贝最终的工程包到远程仓库中，以共享给其他开发人员和工程 |

为了完成 default 生命周期，这些阶段将被按顺序地执行。

运行 `mvn package`，Maven default 生命周期就会从开始执行到 `package` 阶段；运行 `mvn compile`，Maven default 生命周期就会从开始执行到 `compile` 阶段；运行 `mvn clean`，Maven `clean` 生命周期就会执行 clean 的 3 个阶段：

```
pre-clean
clean
post-clean
```

`mvn` 命令后面跟的参数其实就是阶段（phase）。一个 `mvn` 命令可以跟多个阶段参数，会依次执行。比如，`mvn clean package` 限制性 clean 生命周期至 clean 阶段，在执行 default 生命周期至 package 阶段。

执行阶段实际是调用目标（goal）的执行，比如，compile 阶段对应 `compiler:compile` 目标，test 阶段对应 `compiler:testCompile`、`surefire:test` 目标。一般，使用 `mvn` 命令指定阶段来执行就行，也可以指定目标来执行，比如启动 Tomcat 服务器：`mvn tomcat:run`。

总结，使用 Maven 构建项目就是执行 lifecycle，执行到指定的 phase，每个 phase 会执行自己默认的一个或者多个 goal。goal 时最小任务单元。以 `compile` 阶段为例，执行 `mvn compile`，会调用 `compiler` 插件执行关联的 `compiler:compile` 这个目标。

##### 3.1、插件

执行每个阶段都是通过插件（plugin）执行的。Maven 内置的标准插件有：

| 插件     | 对应阶段 |
| -------- | -------- |
| clean    | clean    |
| compiler | compile  |
| surefire | test     |
| jar      | package  |

标准插件无需声明即可使用。标准插件无法满足需求时，可以声明使用自定义插件。使用自定义插件需要使用 `configuration` 标签声明一些配置，比如 `maven-shade-plugin`，需要指定 Java 程序入口：

```xml
<project>
	...
	<build>
		<plugins>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-shade-plugin</artifactId>
				<version>3.2.1</version>
				<executions>
					<execution>
						<phase>package</phase>
						<goals>
							<goal>shade</goal>
						</goals>
					<configuration>
						<transformers>
							<transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
								<mainClass>com.itranswarp.learnjava.Main</mainClass>
							</transformer>
						</transformers>
					</configuration>
					</execution>
				</executions>
			</plugin>
		</plugins>
	</build>
</project>
```

#### 4、resources 标签

项目构建时，Maven 会按照标准的目录结构将 `src/main/resources` 目录下的资源文件打包到 jar 或 war 中。对于不在 Maven 默认资源目录下的资源文件，需要使用 `resources` 标签将其打包。

```xml
<build>
 ...
    <resources>
        <resource>
            <!-- 将文件 src/main/java/com/bulala/ConfigFile/file1.properties 打包 -->
            <directory>src/main/java/com/bulala/ConfigFile</directory>
            <includes>
                <include>file1.properties</include>
            </includes>
        </resource>
        ...
    </resources>
    ...
</build>
```

由于 `includes` 标签仅指明了打包 `file1.properites` 文件，`src/main/java/com/bulala/ConfigFile/` 目录下的其它文件不会被打包。

类似地，`excludes -> exclude` 可以指定相应目录下需要排除的文件。

##### 4.1、filtering 标签

`file1.properties` 文件内容：

```properties
database.username=@db.name@
database.password=@db.pwd@
```

通过如下 `pom.xml` 文件可知，`properties` 标签定义了 `file1.properties` 资源文件引用的变量：

```xml
...
<properties>
	<db.name>utopia</db.name>
	<db.pwd>utopia999</db.pwd>
	...
</properties>
...
<build>
 ...
    <resources>
        <resource>
            <directory>src/main/java/com/bulala/ConfigFile</directory>
            <!-- 默认值 false -->
            <filtering>true</filtering>
            <includes>
                <include>file1.properties</include>
            </includes>
        </resource>
        ...
    </resources>
    ...
</build>
```

当对应资源文件的 `resource -> filtering` 为 true 时，资源文件的引用变量在编译之后会被替换为对应的属性值，比如，编译后 `target` 目录下的 `file1.propertes` 内容为：

```properties
database.username=utopia
database.password=utopia999
```

#### 5、Profile

构建 profile 是一系列用于 Maven 构建的配置值。使用构建 profile 可以为不同环境进行个性化配置。

profile 是使用 `activeProfiles` 或 `profiles` 元素在 `pom.xml` 文件中配置的。在项目 build 时 profiles 会修改 POM，将配置的参数值应用到目标环境。

```xml
<profiles>
    <!-- 开发环境 -->
    <profile>
        <id>dev</id>
        <activation>
            <!--指定该环境为默认配置-->
            <activeByDefault>true</activeByDefault>
        </activation>
        <properties>
            <current.env>dev</current.env>
        </properties>
    </profile>

    <!-- 测试环境 -->
    <profile>
        <id>test</id>
        <properties>
            <current.env>test</current.env>
        </properties>
    </profile>
</profiles>
```

其中 `profile -> id` 定义了 profile 的 id；`profile -> propreties` 标签下的元素为配置参数。

通过配置 `profile -> activation -> activeByDefault` 为 `true` 激活当前的 `profile`。也可以在命令行构建项目时，使用 `-P` 选项选择使用的 `profile` 的 `id`，比如 `mvn clean install -Pdev`。

##### profile 类型

- 项目（project）：项目 pom.xml 文件中定义
- 用户（user）：Maven 用户 `%USER_HOME%/.m2/settings.xml` 文件中定义
- 全局（global）：Maven 全局 `%M2_HOME%/conf/settings.xml` 文件中定义

虽然，可以在 `profile -> properties` 标签下定义许多的参数，但是为了使得 `pom.xml` 看起来简洁，可以结合 `resources` 标签，使用多个 `*.properties` 文件来配置不同环境参数。

##### 5.1、根据环境打包资源文件

目录 `src/main/profile-resources` 下，有不同环境配置文件 `profile-dev.properties`、`profile-test.properties`。使用 `profile` 和 `resource` 标签可以实现根据不同环境打包资源文件：

```xml
<profiles>
    <!-- 开发环境 -->
    <profile>
        <id>dev</id>
        <activation>
            <!--指定该环境为默认配置-->
            <activeByDefault>true</activeByDefault>
        </activation>
        <properties>
            <current.env>dev</current.env>
        </properties>
    </profile>

    <!-- 测试环境 -->
    <profile>
        <id>test</id>
        <properties>
            <current.env>test</current.env>
        </properties>
    </profile>
</profiles>

<build>
    <resources>

        <resource>
            <directory>src/main/profile-resources</directory>
            <includes>
                <include>profile-${current.env}.properties</include>
            </includes>
        </resource>
        ...
    </resources>
</build>
```

不同的 `profile` 对应不同的 `current.env` 值，对应的 `profile-${current.env}.properties` 文件会被打包。

如果有不同环境有多个资源文件，且需要资源文件名相同，可以将不同内容的配置文件置于不同环境目录下，比如 `src/main/profile-resources` 目录下：

```
profile-resource /
	dev /
		db.properties
		other.properties
	test /
		db.properties
		other.properties
```

只需更改 `resource` 标签：

```xml
<profiles>
    <!-- 开发环境 -->
    <profile>
        <id>dev</id>
        <activation>
            <!--指定该环境为默认配置-->
            <activeByDefault>true</activeByDefault>
        </activation>
        <properties>
            <current.env>dev</current.env>
        </properties>
    </profile>

    <!-- 测试环境 -->
    <profile>
        <id>test</id>
        <properties>
            <current.env>test</current.env>
        </properties>
    </profile>
</profiles>

<build>
    <resources>
        <resource>
            <directory>src/main/profile-resources/${current.env}</directory>
            <includes>
                <include>*.properties</include>
            </includes>
        </resource>
        ...
    </resources>
</build>
```

##### 5.2、filters 标签

使用 `profile -> filters` 标签也能实现不同环境使用不同文件的参数。

目录 `src/main/profile-resources`：

```
profile-resources/
	db.properties
	db-dev.properties
	db-test.properties
```

文件 `db.properties`：

```properties
username=@db.name@
password=@db.pwd@
```

文件 `db-dev.properties`：

```properties
db.name=dev
db.pwd=123
```

文件 `db-test.properties`：

```properties
db.name=test
db.pwd=456
```

对应的 `pom.xml` 文件内容：

```xml
<profiles>
    <!-- 开发环境 -->
    <profile>
        <id>dev</id>
        <activation>
            <!--指定该环境为默认配置-->
            <activeByDefault>true</activeByDefault>
        </activation>
        <build>
            <filters>
                <filter>src/main/profile-resources/db-dev.properties</filter>
            </filters>
        </build>
    </profile>

    <!-- 测试环境 -->
    <profile>
        <id>test</id>
        <build>
            <filters>
                <filter>src/main/profile-resources/db-test.properties</filter>
            </filters>
        </build>
    </profile>

</profiles>

<build>
    <resources>
        <resource>
            <directory>src/main/profile-resources</directory>
            <filtering>true</filtering>
            <includes>
                <include>db.properties</include>
            </includes>
        </resource>
        ...
    </resources>
</build>
```

其中 `resource -> filtering` 必须是 `true`。

这种方式下，打包的都是 `db.properties` ，只是内容会被替换为不同环境文件的内容。[参考文章](https://zhuanlan.zhihu.com/p/305816099)