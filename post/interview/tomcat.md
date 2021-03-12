## 容器

通常来说，tomcat被称为一个servlet容器，但事实上tomcat是一个嵌套的容器，我们看server.xml就能理解容器的含义

```xml
<Server port="8005" shutdown="SHUTDOWN"> // 顶层组件，可包含多个 Service，代表一个 Tomcat 实例

  <Service name="Catalina">  // 顶层组件，包含一个 Engine ，多个连接器
    <Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" />

    <!-- Define an AJP 1.3 Connector on port 8009 -->
    <Connector port="8009" protocol="AJP/1.3" redirectPort="8443" />  // 连接器

    // 容器组件：一个 Engine 处理 Service 所有请求，包含多个 Host
    <Engine name="Catalina" defaultHost="localhost">
      // 容器组件：处理指定Host下的客户端请求， 可包含多个 Context
      <Host name="localhost"  appBase="webapps"
            unpackWARs="true" autoDeploy="true">
            // 容器组件：处理特定 Context Web应用的所有客户端请求
            <Context></Context>
      </Host>
    </Engine>
  </Service>
</Server>
```

t

![容器](https://tva1.sinaimg.cn/large/008eGmZEly1gogwr715uej30wk0jyk0c.jpg)

`Wrapper` 表示一个 `Servlet` ，`Context` 表示一个 Web 应用程序，而一个 Web 程序可能有多个 `Servlet` ；`Host` 表示一个虚拟主机，或者说一个站点，一个 Tomcat 可以配置多个站点（Host）；一个站点（ Host） 可以部署多个 Web 应用；`Engine` 代表 引擎，用于管理多个站点（Host），一个 Service 只能有 一个 `Engine`。

## 请求如何定位Servelt

![img](https://tva1.sinaimg.cn/large/008eGmZEly1gogwsaudizj31560u04d7.jpg)

## tomcat的执行流程





## 参考

[Tomcat 架构原理解析到架构设计借鉴](https://segmentfault.com/a/1190000023475177)