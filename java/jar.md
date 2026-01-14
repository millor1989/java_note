#### JAR 和 WAR

### 一、基本定义

| 类型    | 全称                    | 用途                                                         |
| ------- | ----------------------- | ------------------------------------------------------------ |
| **JAR** | Java Archive            | 用于打包 Java 类、资源文件、元数据等，通常作为**库**或**可执行程序**使用 |
| **WAR** | Web Application Archive | 用于打包完整的 **Java Web 应用程序**，供 Web 容器（如 Tomcat、Jetty）部署 |

---

### 二、主要区别

#### 1. **用途不同**
- **JAR**：
  - 用于普通 Java 应用（如命令行工具、后台服务）。
  - 可以是：
    - **库 JAR**：被其他项目依赖（如 `commons-lang3.jar`）。
    - **可执行 JAR**：jar 包还可以包含一个特殊的 `/META-INF/MANIFEST.MF` 文件，`MANIFEST.MF`是纯文本，可以指定 `Main-Class` 和其它信息，可通过 `java -jar app.jar` 直接运行。
- **WAR**：
  - 专为 **Servlet/JSP Web 应用**设计。
  - 必须部署到 **Web 容器**（如 Tomcat、WildFly）中才能运行，不能直接通过 `java -jar` 启动。

#### 2. **内部目录结构不同**

##### JAR 结构（典型）：
```
my-app.jar
├── com/
│   └── example/
│       └── MyClass.class
├── META-INF/
│   └── MANIFEST.MF
└── config.properties
```

##### WAR 结构（标准）：
```
my-webapp.war
├── WEB-INF/
│   ├── web.xml                 ← 部署描述符（可选，Servlet 3.0+ 可省略）
│   ├── classes/                ← 编译后的 .class 文件
│   │   └── com/example/MyServlet.class
│   └── lib/                    ← 依赖的 JAR 包
│       ├── gson.jar
│       └── mysql-connector.jar
├── index.html
├── css/
└── js/
```

> ✅ 关键点：WAR 包必须包含 `WEB-INF/` 目录，其中 `classes/` 和 `lib/` 是 Web 应用类路径的核心。

#### 3. **部署方式不同**
- **JAR**：
  - 独立运行：`java -jar myapp.jar`
  - 或作为依赖引入其他项目。
- **WAR**：
  - 必须部署到 Web 容器（如将 `.war` 文件复制到 Tomcat 的 `webapps/` 目录）。
  - 容器会自动解压并启动 Web 应用（如访问 `http://localhost:8080/my-webapp`）。

#### 4. **是否包含 Web 组件**
- **WAR** 包含：
  - Servlet、Filter、Listener
  - JSP、HTML、CSS、JS 等静态资源
  - `web.xml`（传统配置）或基于注解的配置
- **JAR** 一般不包含这些 Web 特定内容（除非是 Spring Boot 的“可执行 JAR”，见下文）。

---

### 三、特殊说明：Spring Boot 的“Fat JAR”

现代框架（如 Spring Boot）模糊了 JAR 和 WAR 的界限：
- **Spring Boot 默认打成可执行 JAR**（内嵌 Tomcat/Jetty），虽然扩展名是 `.jar`，但实际包含 Web 应用所需的一切。
- 也可以配置为生成 **WAR 包**，以便部署到外部 Web 容器。

> ⚠️ 注意：这种“可执行 JAR”不是传统意义上的 JAR，而是一种自包含的部署格式。
