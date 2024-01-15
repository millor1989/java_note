### Tomcat

Tomcat 是 Jakarta EE（Java EE 演变而来，2017 年 Oracle 将 Java EE 交给 Eclipse 基金会，但是不能使用 Java 的名称）的一个开源实现，包含了 Jakarat Servlet、Jakarat WebSocket 等标准的实现。

#### 1、组件

- **Catalina**：Tomcat 的核心组件，是 Servlet 容器。
- **Bootstrap**：Bootstrap 类是 Tomcat 的入口类，Tomcat 的 main 方法在这个类中，启动脚本往往就是调用这个类。
- **Server**：指的是一个启动的 Tomcat 实例，server 在操作系统中占用一个进程浩，提供服务。
- **Service**：一个 Server 可以有多个 Service，一个 Service 可以有多个 Connector 和一个 Container
- **Connecor**：接收客户端请求，返回结果

Server 代表整个 Tomcat 容器，可以有一个或多个 Service；Service 包括了 Connector 和 Engine，将它们组装在一起对外提供服务，一个 Service 可以包含多个 Connector，但只能包含一个 Engine，Connector 接受请求，Engine 处理请求；Engine、Host、Context 都是容器，Engine 包含 Host，Host 包含 Context，Host 代表 Engine 中的一个虚拟主机，每个 Context 组件代表特定 Host 上运行的一个 Web 应用。

#### 2、Tomcat 服务部署

##### 2.1、CATALINA_HOME 与 CATALINA_BASE

tomcat 启动时，日志会包含以下信息：

```bash
Using CATALINA_BASE: “C:\apache-tomcat-8.0.42”
Using CATALINA_HOME: “C:\apache-tomcat-8.0.42”
Using CATALINA_TMPDIR: “C:\apache-tomcat-8.0.42\temp”
Using JRE_HOME: “C:\Java\jdk1.8.0_101”
```

其中，**CATALINA_HOME** 是 Tomcat 的安装目录，**CATALINA_BASE** 是 Tomcat 的工作目录—— Tomcat 实例的目录。

Tomcat 安装目录包含以下子目录：

```bash
 bin/
 conf/
 lib/
 logs/
 temp/
 webapps/
 work/
```

其中，子目录 `bin`、`lib` 是可以在多个 Tomcat 实例之间共用的。Tomcat 实例目录只用包含以下子目录：

```

 conf/
 logs/
 temp/
 webapps/
 work/
```

运行 Tomcat 时指定 **CATALINA_BASE** 为 Tomcat 实例目录即可运行对应的 Tomcat 实例。

##### 2.2、`server.xml`

一个 `server.xml` 对应一个 Server 实例，即 Tomcat 实例。`server.xml` 的整体结构如下：

```xml
<Server>
    <Service>
        <Connector />
        <Connector />
        <Engine>
            <Host>
                <Context />
            </Host>
        </Engine>
    </Service>
</Server>
```

- 对于 `Server` 标签：`<Server port="8005" shutdown="SHUTDOWN">`  `shutdown` 属性表示关闭 server 的指令，`port` 是接收关闭指令的端口号，端口号设置为 `-1` 表示禁止该端口。

- `Service` 标签可以包含多个 `Connector`，但只能包含一个 `Engine`。

- `Connector` 默认有两个 ：

  ```xml
  <Connector port="8080" protocol="HTTP/1.1" connectionTimeout="20000" redirectPort="8443" />
  <Connector port="8009" protocol="AJP/1.3" redirectPort="8443" />
  ```

  第一个 `Connector` 使用 HTTP 协议建立连接访问 Tomcat 服务器的 Web 应用，第二个 Connector 使用 AJP 协议访问其它 HTTP 服务器。Tomcat 与其它 HTTP 服务器集成时，使用 AJP 协议和其它的 HTTP 服务器（比如 Apache HTTP Server）建立连接，此时使用第二个 `Connector`。

  Tomcat 作为 servlet 容器，处理静态资源性能较差，不如 Apache HTTP 服务器（只能处理静态资源，不能处理动态资源，比如 servlet）。

- `Engine` 是一个容器，是 `Service` 中处理请求的组件。

- `Host` 是 `Engine` 的子容器，可以有多个，表示 Engine 中的一个虚拟主机（host）。

- `Context` 是 `Host` 的子容器，表示一个 Web 应用。

`Host` 的 `name` 属性是主机名称，Tomcat 从 HTTP 头中提取主机名，寻找匹配的主机，与所有 `Host` 名称不匹配的请求发送给默认主机。默认主机通过 `Engine` 的 `defaultHost` 属性配置： 

```xml
<Engine name="Catalina" defaultHost="localhost">
```

##### 2.3、自动部署

当 `Host` 开启了自动部署时，不需要配置 `Context`：

```xml
<Host name="localhost" appBase="webapps" unpackWARs="true" autoDeploy="true">
```

当检测到新的 Web 应用（war）或 Web 应用更新时，会触发应用的部署（重新部署）。

`Host` 有一个 `deployOnStartup`，设置为 `true` 时，Tomcat 在启动时检查 Web 应用（新的或者有更新）；`autoDeploy` 为 `ture` 时，Tomcat 定期检查 Web 应用（新的或更新）。

`Host` 的 `appBase` 属性和 `xmlBase` 属性限定了 Tomcat 检查应用更新的目录。`appBase` 对应 Web 应用所在目录，默认值 `webapps`；`xmlBase` 对于 Web 应用 XML 配置文件目录，默认值 `conf/<engine_name>/<host_name>`。

##### 2.4、`Context` 配置

```xml
<Context path="/app1" docBase="D:\wars\app1.war" reloadable="true"/>
```

`docBase` 属性指定了 Web 应用使用的 War 包路径，自动部署场景下，war 包路径不在 appBase 目录下才需要指定。

`path` 属性指定访问该 Web 应用的 URL 路径。自动部署场景下，URL 路径通过 XML 配置文件的文件名、WAR 文件名或 Web 应用目录名自动推导得出，不能指定 `path` 属性。

`reloadable` 属性，用于调试时设置为 `true`，生产环境应该设置为 `false`。

##### 2.5、静态部署 Web 应用

可以在 `server.xml` 文件中通过 `Context` 元素静态部署 Web 应用。

静态部署可以与自动部署共存。

实际应用中，**不推荐使用静态部署**，因为 `server.xml` 是不可动态重加载的资源，服务器一旦启动后，修改 `server.xml` 就需要重启服务器才能重新加载。而自动部署时，Tomcat 可以通过定期扫描实现，不用重启服务器。

静态部署时，可以显式的指定 `Context` 元素的 `path` 属性。但是，只有当自动部署完全关闭（`deployOnStartup` 和 `autoDeploy` 都是 `false`）或 `docBase` 不在 `appBase` 中时，才可以设置 `path`。

1. 

#### 3、[IDEA 中 Tomcat 部署原理](https://www.cnblogs.com/wl-blog/p/13701071.html)

IDEA 会为应用创建 Tomcat 实例目录，并使用对应的脚本启动应用，脚本中会设置变量 `CATALINA_BASE` 为当前实例目录。

Context 可以在 `${TOMCAT-BASE}/conf/Catalina/localhost` 目录下，以 xml 文件的方式配置。

- 如果要以 `http://localhost:8080/` 访问应用，xml 文件名必须为 `ROOT.xml`，内容为：

  ```xml
  <?xml version="1.0" encoding="UTF-8">
  <Context path="" docBase="AppPath">
  ```

- 如果要以 `http://localhost:8080/dir1` 访问应用，xml 文件名必须为 `dir1.xml`，内容为：

  ```xml
  <?xml version="1.0" encoding="UTF-8">
  <Context path="dir1" docBase="AppPath">
  ```

- 如果要以 `http://localhost:8080/dir1/dir2` 访问应用，xml 文件名必须为 `dir1#dir2.xml`，内容为：

  ```xml
  <?xml version="1.0" encoding="UTF-8">
  <Context path="dir1/dir2" docBase="AppPath">
  ```

IDEA Tomcat 应用部署就是使用 Context xml 配置文件的方式。

##### IDEA 部署 WAR 和部署 WAR exploded 的区别

- 部署 war：直接部署 War 包，`Context` 的 docBase 指定为，打好的 war 包，不支持热部署。
- 部署 war exploded：部署的是 `target` 目录下的编译后的项目，支持热部署。

#### 4、请求处理

##### 4.1、根据协议和端口号选定 Service 和 Engine

Tomcat 根据协议和端口号（Connector 的协议和端口号）选择处理请求的 Service，Service 选定之后，Engine 就确定了。Server 中可以配置多个 Service，可以通过不同的端口号访问同一机器上部署的不同应用。

##### 4.2、根据域名（或 IP）选定 Host

在 Service 中寻找名称与域名（IP）匹配的 Host，没有找到时就使用 Engine 中指定的 defaultHost 处理该请求。

##### 4.3、根据 URI 选定 Context

根据应用 path 属性与 URI 匹配，选择 Web 应用。

Tomcat 处理请求的时候，**优先**查找 `server.xml` 中的 Context 配置，**其次**查找 `catalina\localhost` 中的 `xxx.xml` 配置文件，**最后**才会查找默认的 `appBase`（webapps）。

#### 5、配置多个服务

1. 配置多个 Service
2. 每个 Service 的 Connector 端口号设置为不同值
3. 每个 Service 的 name 属性不能相同， Service 和它所包含的 Engine 的 name 属性应该相同。
4. 每个 Service 中 Host 的 appBase 不同，启用自动部署。
5. web 应用放在不同的 appBase 目录中。





`webapps` 目录下

`webapps` 目录下 `ROOT` 目录

`conf/Catalina/localhost`  目录下 xml 文件