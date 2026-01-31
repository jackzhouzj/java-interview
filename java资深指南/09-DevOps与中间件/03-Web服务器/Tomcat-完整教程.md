# Tomcat 完整教程

## 📋 目录
- 技术概述
- 学习目标
- 基础概念
- 核心架构
- 安装与配置
- 部署应用
- 性能优化
- 集群与高可用
- 监控与调优
- 最佳实践
- 常见问题
- 相关资源
- 学习检查清单

## 📚 技术概述
- **版本**: Apache Tomcat 10.1.x / 9.0.x
- **官方文档**: https://tomcat.apache.org/
- **学习难度**: ⭐⭐⭐ (1-5星)
- **重要程度**: ⭐⭐⭐⭐ (1-5星)
- **前置知识**: Java基础、Servlet规范、HTTP协议
- **文档来源**: Context7 - Apache Tomcat官方文档
- **更新时间**: 2024-01-04

## 🎯 学习目标
- [ ] 理解Tomcat的核心架构和组件
- [ ] 掌握Tomcat的安装、配置和部署
- [ ] 熟悉server.xml等核心配置文件
- [ ] 掌握性能调优和JVM参数配置
- [ ] 了解集群部署和高可用方案
- [ ] 掌握生产环境的最佳实践

## 📖 基础概念

### 1.1 什么是Tomcat

Apache Tomcat是一个开源的Java Servlet容器和Web服务器，实现了Jakarta EE（原Java EE）规范中的Servlet、JSP、WebSocket等核心技术。Tomcat由Apache软件基金会维护，是目前最流行的Java Web应用服务器之一。

**核心特点**:
- **轻量级**: 相比完整的应用服务器（如WebLogic、WebSphere），Tomcat更轻量，启动快速
- **开源免费**: Apache License 2.0许可，完全免费
- **标准兼容**: 完全实现Servlet和JSP规范
- **易于集成**: 可以嵌入到Java应用中，也可以独立部署
- **广泛应用**: 被Spring Boot等主流框架作为默认嵌入式容器

### 1.2 核心概念

- **Servlet容器**: 负责管理Servlet的生命周期，处理HTTP请求和响应
- **Connector**: 连接器，负责接收客户端请求并返回响应
- **Container**: 容器，处理请求的核心组件（Engine、Host、Context、Wrapper）
- **Catalina**: Tomcat的Servlet容器实现
- **Coyote**: Tomcat的HTTP连接器实现
- **Jasper**: Tomcat的JSP引擎

### 1.3 应用场景

- **Web应用部署**: 部署传统的WAR包应用
- **微服务容器**: 作为Spring Boot等微服务的嵌入式容器
- **API服务**: 提供RESTful API服务
- **静态资源服务**: 托管静态HTML、CSS、JavaScript文件
- **反向代理后端**: 配合Nginx、Apache HTTP Server使用

### 1.4 版本选择

| 版本 | Servlet规范 | JSP规范 | Java版本 | 说明 |
|------|------------|---------|---------|------|
| Tomcat 10.1.x | 6.0 | 3.1 | Java 11+ | Jakarta EE 10，推荐新项目使用 |
| Tomcat 10.0.x | 5.0 | 3.0 | Java 8+ | Jakarta EE 9，命名空间从javax改为jakarta |
| Tomcat 9.0.x | 4.0 | 2.3 | Java 8+ | Java EE 8，稳定版本，广泛使用 |
| Tomcat 8.5.x | 3.1 | 2.3 | Java 7+ | 维护模式，不推荐新项目 |

**选择建议**:
- 新项目：优先选择Tomcat 10.1.x
- 现有项目：Tomcat 9.0.x（稳定且兼容性好）
- 遗留系统：根据Java版本选择对应Tomcat版本

## 🔥 核心架构 (重点)

### 2.1 整体架构

Tomcat采用分层的容器架构，从外到内依次为：

```
Server (服务器)
  └── Service (服务)
        ├── Connector (连接器) - 处理网络请求
        └── Engine (引擎)
              └── Host (虚拟主机)
                    └── Context (Web应用上下文)
                          └── Wrapper (Servlet包装器)
```

**架构说明**:
- **Server**: Tomcat实例的顶层容器，一个JVM只有一个Server
- **Service**: 将多个Connector与一个Engine关联，通常只有一个Service（Catalina）
- **Connector**: 监听端口，接收请求，支持HTTP/1.1、HTTP/2、AJP协议
- **Engine**: 处理所有请求的引擎，包含多个虚拟主机
- **Host**: 虚拟主机，代表一个域名，可以部署多个Web应用
- **Context**: 代表一个Web应用（WAR包或目录）
- **Wrapper**: 代表一个Servlet

### 2.2 核心组件

#### 2.2.1 Catalina (Servlet容器)

Catalina是Tomcat的核心，实现了Servlet规范，负责：
- 管理Servlet生命周期
- 处理HTTP请求和响应
- 会话管理
- 安全认证和授权

#### 2.2.2 Coyote (连接器)

Coyote负责底层网络通信，支持多种协议：
- **HTTP/1.1**: 标准HTTP协议
- **HTTP/2**: 支持多路复用、服务器推送
- **AJP**: Apache JServ Protocol，用于与Apache HTTP Server集成

#### 2.2.3 Jasper (JSP引擎)

Jasper负责JSP页面的编译和执行：
- 将JSP编译为Servlet
- 支持JSP标签库
- 提供JSP调试功能

### 2.3 请求处理流程 (⚠️ 难点)

```
客户端请求
  ↓
Connector (接收请求)
  ↓
Engine (路由到Host)
  ↓
Host (路由到Context)
  ↓
Context (路由到Servlet)
  ↓
Wrapper (执行Servlet)
  ↓
Filter Chain (过滤器链)
  ↓
Servlet (业务处理)
  ↓
响应返回客户端
```

**关键点**:
1. **Connector线程模型**: 使用NIO或APR模式处理并发连接
2. **请求路由**: 根据域名、路径匹配到具体的Servlet
3. **过滤器链**: 在Servlet执行前后进行拦截处理
4. **线程池管理**: 使用线程池处理请求，避免频繁创建销毁线程

## 💻 安装与配置

### 3.1 环境准备

**系统要求**:
- Java JDK 11+ (Tomcat 10.1.x)
- 操作系统：Windows、Linux、macOS
- 内存：至少512MB，推荐2GB+
- 磁盘：至少100MB

**安装JDK**:
```bash
# 检查Java版本
java -version

# 设置JAVA_HOME环境变量（Linux/macOS）
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk
export PATH=$JAVA_HOME/bin:$PATH

# Windows设置环境变量
set JAVA_HOME=C:\Program Files\Java\jdk-11
set PATH=%JAVA_HOME%\bin;%PATH%
```

### 3.2 下载与安装

**下载Tomcat**:
```bash
# 访问官网下载
https://tomcat.apache.org/download-10.cgi

# 或使用wget下载（Linux）
wget https://dlcdn.apache.org/tomcat/tomcat-10/v10.1.17/bin/apache-tomcat-10.1.17.tar.gz

# 解压
tar -xzf apache-tomcat-10.1.17.tar.gz
cd apache-tomcat-10.1.17
```

**目录结构**:
```
apache-tomcat-10.1.17/
├── bin/              # 启动脚本和工具
│   ├── startup.sh    # 启动脚本（Linux/macOS）
│   ├── shutdown.sh   # 停止脚本
│   ├── catalina.sh   # 核心启动脚本
│   └── startup.bat   # Windows启动脚本
├── conf/             # 配置文件
│   ├── server.xml    # 核心配置文件
│   ├── web.xml       # 全局Web应用配置
│   ├── context.xml   # 全局Context配置
│   ├── tomcat-users.xml  # 用户权限配置
│   └── logging.properties # 日志配置
├── lib/              # Tomcat核心库
├── logs/             # 日志文件
├── temp/             # 临时文件
├── webapps/          # Web应用部署目录
│   ├── ROOT/         # 根应用
│   ├── docs/         # 文档
│   ├── examples/     # 示例应用
│   └── manager/      # 管理应用
└── work/             # JSP编译后的文件
```

### 3.3 启动与停止

**启动Tomcat**:
```bash
# Linux/macOS
cd $TOMCAT_HOME/bin
./startup.sh

# Windows
cd %TOMCAT_HOME%\bin
startup.bat

# 查看启动日志
tail -f ../logs/catalina.out
```

**停止Tomcat**:
```bash
# Linux/macOS
./shutdown.sh

# Windows
shutdown.bat

# 强制停止（如果shutdown失败）
ps -ef | grep tomcat
kill -9 <pid>
```

**验证启动**:
```bash
# 访问默认页面
http://localhost:8080

# 检查端口占用
netstat -an | grep 8080
lsof -i:8080
```

### 3.4 核心配置文件 (🔥 重点)

#### 3.4.1 server.xml - 服务器配置

这是Tomcat最核心的配置文件，定义了Server、Service、Connector、Engine等组件。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Server port="8005" shutdown="SHUTDOWN">
  <!-- 监听器：用于初始化和清理资源 -->
  <Listener className="org.apache.catalina.startup.VersionLoggerListener" />
  <Listener className="org.apache.catalina.core.JreMemoryLeakPreventionListener" />
  
  <!-- 全局JNDI资源 -->
  <GlobalNamingResources>
    <Resource name="UserDatabase"
              auth="Container"
              type="org.apache.catalina.UserDatabase"
              factory="org.apache.catalina.users.MemoryUserDatabaseFactory"
              pathname="conf/tomcat-users.xml" />
  </GlobalNamingResources>
  
  <!-- 服务：关联Connector和Engine -->
  <Service name="Catalina">
    
    <!-- HTTP/1.1 Connector - 默认8080端口 -->
    <Connector port="8080"
               protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443"
               maxThreads="200"
               minSpareThreads="10"
               acceptCount="100"
               enableLookups="false"
               compression="on"
               compressionMinSize="2048"
               compressibleMimeType="text/html,text/xml,text/plain,text/css,text/javascript,application/javascript,application/json" />
    
    <!-- HTTPS Connector - 支持HTTP/2 -->
    <Connector port="8443"
               protocol="org.apache.coyote.http11.Http11NioProtocol"
               maxThreads="150"
               SSLEnabled="true">
      <UpgradeProtocol className="org.apache.coyote.http2.Http2Protocol" />
      <SSLHostConfig>
        <Certificate certificateKeystoreFile="conf/keystore.jks"
                     certificateKeystorePassword="changeit"
                     type="RSA" />
      </SSLHostConfig>
    </Connector>
    
    <!-- AJP Connector - 用于与Apache HTTP Server集成 -->
    <Connector protocol="AJP/1.3"
               address="::1"
               port="8009"
               redirectPort="8443"
               secretRequired="false" />
    
    <!-- Engine：处理请求的引擎 -->
    <Engine name="Catalina" defaultHost="localhost">
      
      <!-- Realm：安全认证 -->
      <Realm className="org.apache.catalina.realm.LockOutRealm">
        <Realm className="org.apache.catalina.realm.UserDatabaseRealm"
               resourceName="UserDatabase"/>
      </Realm>
      
      <!-- Host：虚拟主机 -->
      <Host name="localhost"
            appBase="webapps"
            unpackWARs="true"
            autoDeploy="true">
        
        <!-- Valve：访问日志 -->
        <Valve className="org.apache.catalina.valves.AccessLogValve"
               directory="logs"
               prefix="localhost_access_log"
               suffix=".txt"
               pattern="%h %l %u %t &quot;%r&quot; %s %b" />
      </Host>
    </Engine>
  </Service>
</Server>
```

**关键配置说明**:
- **port="8005"**: 关闭端口，用于shutdown命令
- **maxThreads**: 最大工作线程数，默认200
- **minSpareThreads**: 最小空闲线程数
- **acceptCount**: 等待队列长度，超过则拒绝连接
- **connectionTimeout**: 连接超时时间（毫秒）
- **compression**: 启用响应压缩

#### 3.4.2 context.xml - 应用上下文配置

配置Web应用的上下文参数，如数据源、资源引用等。

```xml
<!-- 全局context.xml：$TOMCAT_HOME/conf/context.xml -->
<Context>
  <!-- 数据库连接池配置 -->
  <Resource name="jdbc/MyDB"
            auth="Container"
            type="javax.sql.DataSource"
            maxTotal="100"
            maxIdle="30"
            maxWaitMillis="10000"
            username="dbuser"
            password="dbpass"
            driverClassName="com.mysql.cj.jdbc.Driver"
            url="jdbc:mysql://localhost:3306/mydb?useSSL=false&amp;serverTimezone=UTC"/>
  
  <!-- 禁用会话持久化 -->
  <Manager pathname="" />
</Context>
```

**应用级context.xml**:
```xml
<!-- 应用级：$TOMCAT_HOME/webapps/myapp/META-INF/context.xml -->
<Context path="/myapp" docBase="myapp" reloadable="true">
  <!-- 应用特定的资源配置 -->
  <Resource name="jdbc/AppDB"
            auth="Container"
            type="javax.sql.DataSource"
            maxTotal="50"
            maxIdle="20"
            maxWaitMillis="5000"
            username="appuser"
            password="apppass"
            driverClassName="com.mysql.cj.jdbc.Driver"
            url="jdbc:mysql://localhost:3306/appdb"/>
</Context>
```

#### 3.4.3 web.xml - Web应用配置

全局的Servlet和MIME类型配置。

```xml
<!-- $TOMCAT_HOME/conf/web.xml -->
<web-app>
  <!-- 默认Servlet配置 -->
  <servlet>
    <servlet-name>default</servlet-name>
    <servlet-class>org.apache.catalina.servlets.DefaultServlet</servlet-class>
    <init-param>
      <param-name>listings</param-name>
      <param-value>false</param-value>  <!-- 禁止目录列表 -->
    </init-param>
  </servlet>
  
  <!-- JSP Servlet配置 -->
  <servlet>
    <servlet-name>jsp</servlet-name>
    <servlet-class>org.apache.jasper.servlet.JspServlet</servlet-class>
    <init-param>
      <param-name>development</param-name>
      <param-value>false</param-value>  <!-- 生产环境设为false -->
    </init-param>
  </servlet>
  
  <!-- 会话超时（分钟） -->
  <session-config>
    <session-timeout>30</session-timeout>
  </session-config>
  
  <!-- 欢迎页面 -->
  <welcome-file-list>
    <welcome-file>index.html</welcome-file>
    <welcome-file>index.jsp</welcome-file>
  </welcome-file-list>
</web-app>
```

#### 3.4.4 tomcat-users.xml - 用户权限配置

配置管理界面的用户和角色。

```xml
<!-- $TOMCAT_HOME/conf/tomcat-users.xml -->
<tomcat-users>
  <!-- 定义角色 -->
  <role rolename="manager-gui"/>
  <role rolename="manager-script"/>
  <role rolename="admin-gui"/>
  
  <!-- 定义用户 -->
  <user username="admin" 
        password="admin123" 
        roles="manager-gui,admin-gui"/>
  <user username="deployer" 
        password="deploy123" 
        roles="manager-script"/>
</tomcat-users>
```

**角色说明**:
- **manager-gui**: 访问Manager Web界面
- **manager-script**: 使用脚本部署应用
- **admin-gui**: 访问Host Manager界面
- **manager-jmx**: JMX代理访问

## 🚀 部署应用

### 4.1 部署方式

#### 4.1.1 WAR包部署（推荐）

```bash
# 1. 将WAR包复制到webapps目录
cp myapp.war $TOMCAT_HOME/webapps/

# 2. Tomcat会自动解压并部署
# 访问：http://localhost:8080/myapp

# 3. 热部署：替换WAR包后自动重新部署
cp myapp-v2.war $TOMCAT_HOME/webapps/myapp.war
```

#### 4.1.2 目录部署

```bash
# 1. 创建应用目录
mkdir -p $TOMCAT_HOME/webapps/myapp

# 2. 复制应用文件
cp -r /path/to/myapp/* $TOMCAT_HOME/webapps/myapp/

# 3. 目录结构
myapp/
├── WEB-INF/
│   ├── web.xml
│   ├── classes/
│   └── lib/
├── META-INF/
│   └── context.xml
├── index.html
└── static/
```

#### 4.1.3 外部目录部署

在`conf/Catalina/localhost/`下创建XML配置文件：

```xml
<!-- $TOMCAT_HOME/conf/Catalina/localhost/myapp.xml -->
<Context docBase="/opt/apps/myapp" reloadable="true">
  <!-- 应用配置 -->
</Context>
```

#### 4.1.4 Manager应用部署

通过Web界面部署：
```
1. 访问：http://localhost:8080/manager/html
2. 登录（使用tomcat-users.xml中配置的用户）
3. 选择WAR文件上传
4. 点击"Deploy"按钮
```

### 4.2 嵌入式Tomcat（Spring Boot）

```java
// Spring Boot自动配置嵌入式Tomcat
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}

// application.properties配置
server.port=8080
server.tomcat.max-threads=200
server.tomcat.min-spare-threads=10
server.tomcat.connection-timeout=20000
server.tomcat.accept-count=100
```

### 4.3 编程式启动Tomcat

```java
import org.apache.catalina.Context;
import org.apache.catalina.LifecycleException;
import org.apache.catalina.startup.Tomcat;

import java.io.File;

/**
 * 嵌入式Tomcat启动示例
 * 
 * @author erik.zhou
 */
public class EmbeddedTomcat {
    
    public static void main(String[] args) throws LifecycleException {
        // 创建Tomcat实例
        Tomcat tomcat = new Tomcat();
        
        // 设置端口
        tomcat.setPort(8080);
        
        // 设置基础目录
        tomcat.setBaseDir("temp");
        
        // 添加Web应用
        String contextPath = "";
        String docBase = new File("src/main/webapp").getAbsolutePath();
        Context context = tomcat.addWebapp(contextPath, docBase);
        
        // 启动Tomcat
        tomcat.start();
        
        // 等待请求
        tomcat.getServer().await();
    }
}
```

## ⚡ 性能优化 (🔥 重点)

### 5.1 JVM参数优化 (⚠️ 难点)

在`catalina.sh`（Linux）或`catalina.bat`（Windows）中配置JVM参数：

```bash
# Linux: $TOMCAT_HOME/bin/setenv.sh
export JAVA_OPTS="-server \
  -Xms2048m \
  -Xmx2048m \
  -Xmn1024m \
  -XX:MetaspaceSize=256m \
  -XX:MaxMetaspaceSize=512m \
  -XX:+UseG1GC \
  -XX:MaxGCPauseMillis=200 \
  -XX:+HeapDumpOnOutOfMemoryError \
  -XX:HeapDumpPath=/var/log/tomcat/heapdump.hprof \
  -Djava.awt.headless=true \
  -Dfile.encoding=UTF-8 \
  -Duser.timezone=Asia/Shanghai"
```

**参数说明**:
- **-server**: 使用Server模式JVM（生产环境必须）
- **-Xms/-Xmx**: 初始/最大堆内存，建议设置相同值避免动态扩容
- **-Xmn**: 新生代大小，一般设置为堆内存的1/3到1/2
- **-XX:MetaspaceSize**: 元空间初始大小
- **-XX:+UseG1GC**: 使用G1垃圾收集器（推荐）
- **-XX:MaxGCPauseMillis**: GC最大暂停时间目标
- **-XX:+HeapDumpOnOutOfMemoryError**: OOM时自动生成堆转储

**不同场景的JVM配置**:

```bash
# 小型应用（1-2GB内存）
JAVA_OPTS="-Xms512m -Xmx1024m -Xmn512m -XX:MetaspaceSize=128m"

# 中型应用（4-8GB内存）
JAVA_OPTS="-Xms2048m -Xmx4096m -Xmn2048m -XX:MetaspaceSize=256m"

# 大型应用（16GB+内存）
JAVA_OPTS="-Xms8192m -Xmx8192m -Xmn4096m -XX:MetaspaceSize=512m"
```

### 5.2 Connector优化

#### 5.2.1 线程池配置

```xml
<Connector port="8080"
           protocol="org.apache.coyote.http11.Http11NioProtocol"
           maxThreads="500"
           minSpareThreads="50"
           maxConnections="10000"
           acceptCount="200"
           connectionTimeout="20000"
           keepAliveTimeout="60000"
           maxKeepAliveRequests="100"
           enableLookups="false"
           compression="on"
           compressionMinSize="2048"
           compressibleMimeType="text/html,text/xml,text/plain,text/css,text/javascript,application/javascript,application/json,application/xml"
           URIEncoding="UTF-8" />
```

**参数调优建议**:
- **maxThreads**: 根据CPU核心数设置，一般为核心数 × 50-100
- **minSpareThreads**: 最小空闲线程，保持一定数量避免频繁创建
- **maxConnections**: 最大连接数，NIO模式下可设置较大值（10000+）
- **acceptCount**: 等待队列长度，超过则拒绝连接
- **keepAliveTimeout**: Keep-Alive超时时间，避免长时间占用连接
- **compression**: 启用压缩，减少网络传输

#### 5.2.2 协议选择

```xml
<!-- NIO模式（推荐） -->
<Connector protocol="org.apache.coyote.http11.Http11NioProtocol" ... />

<!-- NIO2模式（异步I/O） -->
<Connector protocol="org.apache.coyote.http11.Http11Nio2Protocol" ... />

<!-- APR模式（需要安装APR库，性能最好） -->
<Connector protocol="org.apache.coyote.http11.Http11AprProtocol" ... />
```

### 5.3 数据库连接池优化

```xml
<Resource name="jdbc/MyDB"
          auth="Container"
          type="javax.sql.DataSource"
          factory="org.apache.tomcat.jdbc.pool.DataSourceFactory"
          driverClassName="com.mysql.cj.jdbc.Driver"
          url="jdbc:mysql://localhost:3306/mydb?useSSL=false&amp;serverTimezone=UTC"
          username="dbuser"
          password="dbpass"
          
          <!-- 连接池配置 -->
          initialSize="10"
          maxActive="100"
          maxIdle="50"
          minIdle="10"
          maxWait="10000"
          
          <!-- 连接验证 -->
          testOnBorrow="true"
          testWhileIdle="true"
          validationQuery="SELECT 1"
          validationInterval="30000"
          
          <!-- 连接泄漏检测 -->
          removeAbandoned="true"
          removeAbandonedTimeout="60"
          logAbandoned="true"
          
          <!-- 性能优化 -->
          timeBetweenEvictionRunsMillis="30000"
          minEvictableIdleTimeMillis="60000"
          jdbcInterceptors="org.apache.tomcat.jdbc.pool.interceptor.ConnectionState;
                           org.apache.tomcat.jdbc.pool.interceptor.StatementFinalizer" />
```

### 5.4 静态资源优化

#### 5.4.1 启用缓存

```xml
<!-- 在web.xml中配置DefaultServlet -->
<servlet>
  <servlet-name>default</servlet-name>
  <servlet-class>org.apache.catalina.servlets.DefaultServlet</servlet-class>
  <init-param>
    <param-name>listings</param-name>
    <param-value>false</param-value>
  </init-param>
  <init-param>
    <param-name>cacheMaxSize</param-name>
    <param-value>102400</param-value>  <!-- 100MB缓存 -->
  </init-param>
  <init-param>
    <param-name>cacheTTL</param-name>
    <param-value>60000</param-value>  <!-- 缓存60秒 -->
  </init-param>
</servlet>
```

#### 5.4.2 使用CDN或反向代理

```nginx
# Nginx反向代理配置
upstream tomcat_backend {
    server 127.0.0.1:8080;
    server 127.0.0.1:8081;
}

server {
    listen 80;
    server_name example.com;
    
    # 静态资源直接由Nginx处理
    location ~* \.(jpg|jpeg|png|gif|css|js|ico)$ {
        root /var/www/static;
        expires 30d;
        add_header Cache-Control "public, immutable";
    }
    
    # 动态请求转发到Tomcat
    location / {
        proxy_pass http://tomcat_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

### 5.5 应用优化

#### 5.5.1 异步Servlet

```java
import jakarta.servlet.AsyncContext;
import jakarta.servlet.ServletException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;

import java.io.IOException;
import java.io.PrintWriter;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

/**
 * 异步Servlet示例
 * 
 * @author erik.zhou
 */
@WebServlet(urlPatterns = "/async", asyncSupported = true)
public class AsyncServlet extends HttpServlet {
    
    private static final ExecutorService executorService = 
        Executors.newFixedThreadPool(10);
    
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        
        // 启动异步处理
        final AsyncContext asyncContext = request.startAsync();
        asyncContext.setTimeout(30000); // 30秒超时
        
        // 提交任务到线程池
        executorService.submit(() -> {
            try {
                // 模拟长时间操作
                Thread.sleep(5000);
                
                // 写入响应
                HttpServletResponse asyncResponse = 
                    (HttpServletResponse) asyncContext.getResponse();
                asyncResponse.setContentType("text/html;charset=UTF-8");
                PrintWriter out = asyncResponse.getWriter();
                out.println("<html><body>");
                out.println("<h1>异步处理完成</h1>");
                out.println("<p>处理时间: " + System.currentTimeMillis() + "</p>");
                out.println("</body></html>");
                
                // 完成异步处理
                asyncContext.complete();
            } catch (Exception e) {
                e.printStackTrace();
            }
        });
        
        // 请求线程立即释放
    }
    
    @Override
    public void destroy() {
        executorService.shutdown();
    }
}
```

#### 5.5.2 禁用不必要的功能

```xml
<!-- server.xml -->
<Host name="localhost"
      appBase="webapps"
      unpackWARs="true"
      autoDeploy="false"  <!-- 生产环境禁用自动部署 -->
      deployOnStartup="true">
</Host>

<!-- context.xml -->
<Context>
  <!-- 禁用会话持久化 -->
  <Manager pathname="" />
</Context>
```

## 🌐 集群与高可用

### 6.1 集群架构

```
                    [负载均衡器]
                    (Nginx/HAProxy)
                          |
        +----------------+----------------+
        |                |                |
   [Tomcat1]        [Tomcat2]        [Tomcat3]
   (8080)           (8080)           (8080)
        |                |                |
        +----------------+----------------+
                          |
                  [共享Session存储]
                  (Redis/Memcached)
```

### 6.2 会话复制配置

#### 6.2.1 集群配置

```xml
<!-- server.xml -->
<Engine name="Catalina" defaultHost="localhost" jvmRoute="tomcat1">
  
  <!-- 集群配置 -->
  <Cluster className="org.apache.catalina.ha.tcp.SimpleTcpCluster"
           channelSendOptions="8">
    
    <!-- 会话管理器 -->
    <Manager className="org.apache.catalina.ha.session.DeltaManager"
             expireSessionsOnShutdown="false"
             notifyListenersOnReplication="true"/>
    
    <!-- 集群通信通道 -->
    <Channel className="org.apache.catalina.tribes.group.GroupChannel">
      <Membership className="org.apache.catalina.tribes.membership.McastService"
                  address="228.0.0.4"
                  port="45564"
                  frequency="500"
                  dropTime="3000"/>
      <Receiver className="org.apache.catalina.tribes.transport.nio.NioReceiver"
                address="auto"
                port="4000"
                autoBind="100"
                selectorTimeout="5000"
                maxThreads="6"/>
      <Sender className="org.apache.catalina.tribes.transport.ReplicationTransmitter">
        <Transport className="org.apache.catalina.tribes.transport.nio.PooledParallelSender"/>
      </Sender>
      <Interceptor className="org.apache.catalina.tribes.group.interceptors.TcpFailureDetector"/>
      <Interceptor className="org.apache.catalina.tribes.group.interceptors.MessageDispatchInterceptor"/>
    </Channel>
    
    <!-- 集群部署器 -->
    <Valve className="org.apache.catalina.ha.tcp.ReplicationValve"
           filter=""/>
    <Valve className="org.apache.catalina.ha.session.JvmRouteBinderValve"/>
    
    <ClusterListener className="org.apache.catalina.ha.session.ClusterSessionListener"/>
  </Cluster>
  
  <Host name="localhost" appBase="webapps">
    <!-- Host配置 -->
  </Host>
</Engine>
```

#### 6.2.2 应用配置

```xml
<!-- web.xml -->
<web-app>
  <!-- 启用会话复制 -->
  <distributable/>
</web-app>
```

### 6.3 Redis会话共享（推荐）

使用Redis存储会话，避免Tomcat集群间的会话复制开销。

**添加依赖**:
```xml
<!-- pom.xml -->
<dependency>
    <groupId>org.redisson</groupId>
    <artifactId>redisson-tomcat-10</artifactId>
    <version>3.24.3</version>
</dependency>
```

**配置**:
```xml
<!-- context.xml -->
<Context>
  <Manager className="org.redisson.tomcat.RedissonSessionManager"
           configPath="${catalina.base}/conf/redisson.yaml"
           readMode="REDIS"
           updateMode="DEFAULT"/>
</Context>
```

**Redisson配置**:
```yaml
# redisson.yaml
singleServerConfig:
  address: "redis://127.0.0.1:6379"
  password: null
  database: 0
  connectionPoolSize: 64
  connectionMinimumIdleSize: 10
  idleConnectionTimeout: 10000
  connectTimeout: 10000
  timeout: 3000
```

### 6.4 负载均衡配置

#### 6.4.1 Nginx负载均衡

```nginx
upstream tomcat_cluster {
    # 负载均衡策略
    ip_hash;  # 基于IP的会话保持
    
    server 192.168.1.101:8080 weight=1 max_fails=2 fail_timeout=30s;
    server 192.168.1.102:8080 weight=1 max_fails=2 fail_timeout=30s;
    server 192.168.1.103:8080 weight=1 max_fails=2 fail_timeout=30s;
}

server {
    listen 80;
    server_name example.com;
    
    location / {
        proxy_pass http://tomcat_cluster;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # 超时设置
        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
        
        # 缓冲设置
        proxy_buffering on;
        proxy_buffer_size 4k;
        proxy_buffers 8 4k;
    }
}
```

#### 6.4.2 Apache HTTP Server + AJP

```apache
# httpd.conf
LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_ajp_module modules/mod_proxy_ajp.so
LoadModule proxy_balancer_module modules/mod_proxy_balancer.so
LoadModule lbmethod_byrequests_module modules/mod_lbmethod_byrequests.so

<Proxy balancer://tomcat_cluster>
    BalancerMember ajp://192.168.1.101:8009 route=tomcat1
    BalancerMember ajp://192.168.1.102:8009 route=tomcat2
    BalancerMember ajp://192.168.1.103:8009 route=tomcat3
    ProxySet stickysession=JSESSIONID
</Proxy>

<VirtualHost *:80>
    ServerName example.com
    ProxyPass / balancer://tomcat_cluster/
    ProxyPassReverse / balancer://tomcat_cluster/
</VirtualHost>
```

## 📊 监控与调优

### 7.1 JMX监控

**启用JMX**:
```bash
# setenv.sh
export CATALINA_OPTS="$CATALINA_OPTS \
  -Dcom.sun.management.jmxremote \
  -Dcom.sun.management.jmxremote.port=9999 \
  -Dcom.sun.management.jmxremote.ssl=false \
  -Dcom.sun.management.jmxremote.authenticate=false"
```

**使用JConsole连接**:
```bash
jconsole localhost:9999
```

### 7.2 Manager应用监控

访问Manager应用查看运行状态：
```
http://localhost:8080/manager/status
http://localhost:8080/manager/html
```

**监控指标**:
- 线程池使用情况
- 内存使用情况
- 会话数量
- 请求处理时间
- 应用部署状态

### 7.3 日志分析

**访问日志分析**:
```bash
# 统计访问量
cat localhost_access_log.2024-01-04.txt | wc -l

# 统计状态码分布
awk '{print $9}' localhost_access_log.2024-01-04.txt | sort | uniq -c

# 统计响应时间
awk '{sum+=$NF; count++} END {print "平均响应时间:", sum/count, "ms"}' localhost_access_log.2024-01-04.txt

# 统计访问最多的URL
awk '{print $7}' localhost_access_log.2024-01-04.txt | sort | uniq -c | sort -rn | head -10
```

**Catalina日志分析**:
```bash
# 查看错误日志
grep -i "error\|exception" catalina.out

# 查看OOM错误
grep -i "OutOfMemoryError" catalina.out

# 实时监控日志
tail -f catalina.out
```

### 7.4 性能分析工具

**JProfiler**:
- 连接到Tomcat JVM
- 分析CPU使用率
- 分析内存分配
- 分析线程状态

**VisualVM**:
```bash
# 启动VisualVM
jvisualvm

# 连接到Tomcat进程
# 查看堆内存、线程、CPU使用情况
```

**Arthas**:
```bash
# 下载Arthas
wget https://arthas.aliyun.com/arthas-boot.jar

# 启动Arthas
java -jar arthas-boot.jar

# 选择Tomcat进程
# 执行诊断命令
dashboard  # 查看实时数据
thread     # 查看线程信息
jvm        # 查看JVM信息
```

## ✨ 最佳实践

### 8.1 生产环境配置清单

**安全配置**:
```xml
<!-- 1. 删除默认应用 -->
rm -rf $TOMCAT_HOME/webapps/docs
rm -rf $TOMCAT_HOME/webapps/examples
rm -rf $TOMCAT_HOME/webapps/host-manager
rm -rf $TOMCAT_HOME/webapps/manager  # 或限制访问IP

<!-- 2. 禁用目录列表 -->
<servlet>
  <servlet-name>default</servlet-name>
  <init-param>
    <param-name>listings</param-name>
    <param-value>false</param-value>
  </init-param>
</servlet>

<!-- 3. 隐藏版本信息 -->
<Connector port="8080" 
           server="Apache" />  <!-- 不显示Tomcat版本 -->

<!-- 4. 配置访问控制 -->
<Valve className="org.apache.catalina.valves.RemoteAddrValve"
       allow="192\.168\.1\.\d+|127\.0\.0\.1"/>

<!-- 5. 启用HTTPS -->
<Connector port="8443"
           protocol="org.apache.coyote.http11.Http11NioProtocol"
           SSLEnabled="true"
           scheme="https"
           secure="true">
  <SSLHostConfig protocols="TLSv1.2,TLSv1.3">
    <Certificate certificateKeystoreFile="conf/keystore.jks"
                 certificateKeystorePassword="changeit"/>
  </SSLHostConfig>
</Connector>
```

**性能配置**:
```bash
# JVM参数
JAVA_OPTS="-server \
  -Xms4096m -Xmx4096m \
  -XX:+UseG1GC \
  -XX:MaxGCPauseMillis=200 \
  -XX:+HeapDumpOnOutOfMemoryError \
  -XX:HeapDumpPath=/var/log/tomcat/heapdump.hprof"

# Connector配置
maxThreads="500"
minSpareThreads="50"
maxConnections="10000"
acceptCount="200"
compression="on"
```

**日志配置**:
```properties
# logging.properties
# 日志级别
.level = INFO
org.apache.catalina.level = INFO
org.apache.coyote.level = INFO

# 日志文件
1catalina.org.apache.juli.AsyncFileHandler.level = FINE
1catalina.org.apache.juli.AsyncFileHandler.directory = ${catalina.base}/logs
1catalina.org.apache.juli.AsyncFileHandler.prefix = catalina.
1catalina.org.apache.juli.AsyncFileHandler.maxDays = 30
1catalina.org.apache.juli.AsyncFileHandler.encoding = UTF-8

# 访问日志
<Valve className="org.apache.catalina.valves.AccessLogValve"
       directory="logs"
       prefix="access_log"
       suffix=".txt"
       pattern="%h %l %u %t &quot;%r&quot; %s %b %D"
       rotatable="true"
       maxDays="30"/>
```

### 8.2 部署最佳实践

**1. 使用外部配置**:
```bash
# 将配置文件放在外部目录
export CATALINA_BASE=/opt/tomcat-instance1
export CATALINA_HOME=/opt/tomcat

# 目录结构
/opt/tomcat-instance1/
├── conf/
├── logs/
├── temp/
├── webapps/
└── work/
```

**2. 自动化部署脚本**:
```bash
#!/bin/bash
# deploy.sh

APP_NAME="myapp"
WAR_FILE="$APP_NAME.war"
TOMCAT_HOME="/opt/tomcat"
WEBAPPS_DIR="$TOMCAT_HOME/webapps"
BACKUP_DIR="/opt/backups"

# 备份旧版本
if [ -d "$WEBAPPS_DIR/$APP_NAME" ]; then
    echo "备份旧版本..."
    tar -czf "$BACKUP_DIR/$APP_NAME-$(date +%Y%m%d%H%M%S).tar.gz" \
        -C "$WEBAPPS_DIR" "$APP_NAME"
    rm -rf "$WEBAPPS_DIR/$APP_NAME"
fi

# 停止Tomcat
echo "停止Tomcat..."
$TOMCAT_HOME/bin/shutdown.sh
sleep 5

# 部署新版本
echo "部署新版本..."
cp "$WAR_FILE" "$WEBAPPS_DIR/"

# 启动Tomcat
echo "启动Tomcat..."
$TOMCAT_HOME/bin/startup.sh

# 检查启动状态
echo "检查启动状态..."
sleep 10
if curl -s http://localhost:8080/$APP_NAME > /dev/null; then
    echo "部署成功！"
else
    echo "部署失败，回滚..."
    # 回滚逻辑
fi
```

**3. 健康检查**:
```java
/**
 * 健康检查Servlet
 * 
 * @author erik.zhou
 */
@WebServlet("/health")
public class HealthCheckServlet extends HttpServlet {
    
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        
        response.setContentType("application/json;charset=UTF-8");
        PrintWriter out = response.getWriter();
        
        try {
            // 检查数据库连接
            boolean dbOk = checkDatabase();
            
            // 检查Redis连接
            boolean redisOk = checkRedis();
            
            if (dbOk && redisOk) {
                response.setStatus(HttpServletResponse.SC_OK);
                out.println("{\"status\":\"UP\",\"database\":\"UP\",\"redis\":\"UP\"}");
            } else {
                response.setStatus(HttpServletResponse.SC_SERVICE_UNAVAILABLE);
                out.println("{\"status\":\"DOWN\",\"database\":\"" + 
                    (dbOk ? "UP" : "DOWN") + "\",\"redis\":\"" + 
                    (redisOk ? "UP" : "DOWN") + "\"}");
            }
        } catch (Exception e) {
            response.setStatus(HttpServletResponse.SC_INTERNAL_SERVER_ERROR);
            out.println("{\"status\":\"ERROR\",\"message\":\"" + 
                e.getMessage() + "\"}");
        }
    }
    
    private boolean checkDatabase() {
        // 数据库连接检查逻辑
        return true;
    }
    
    private boolean checkRedis() {
        // Redis连接检查逻辑
        return true;
    }
}
```

### 8.3 常见陷阱

**⚠️ 陷阱1: 内存泄漏**
```java
// 错误：ThreadLocal未清理
public class UserContext {
    private static ThreadLocal<User> userThreadLocal = new ThreadLocal<>();
    
    public static void setUser(User user) {
        userThreadLocal.set(user);
        // 忘记remove()，导致内存泄漏
    }
}

// 正确：使用后清理
public class UserContext {
    private static ThreadLocal<User> userThreadLocal = new ThreadLocal<>();
    
    public static void setUser(User user) {
        userThreadLocal.set(user);
    }
    
    public static void clear() {
        userThreadLocal.remove();  // 必须清理
    }
}

// 在Filter中清理
public class UserContextFilter implements Filter {
    @Override
    public void doFilter(ServletRequest request, ServletResponse response, 
                        FilterChain chain) throws IOException, ServletException {
        try {
            chain.doFilter(request, response);
        } finally {
            UserContext.clear();  // 确保清理
        }
    }
}
```

**⚠️ 陷阱2: 类加载器泄漏**
```xml
<!-- 避免类加载器泄漏 -->
<Context>
  <!-- 禁用热部署（生产环境） -->
  <Loader delegate="false" reloadable="false"/>
  
  <!-- 清理JDBC驱动 -->
  <Listener className="org.apache.catalina.core.JreMemoryLeakPreventionListener"/>
</Context>
```

**⚠️ 陷阱3: 会话过多**
```xml
<!-- 限制会话数量和超时时间 -->
<Context>
  <Manager className="org.apache.catalina.session.StandardManager"
           maxActiveSessions="1000"
           sessionIdLength="32"/>
</Context>

<!-- web.xml -->
<session-config>
  <session-timeout>30</session-timeout>  <!-- 30分钟超时 -->
</session-config>
```

**⚠️ 陷阱4: 文件描述符耗尽**
```bash
# 增加文件描述符限制
ulimit -n 65535

# 在/etc/security/limits.conf中永久设置
* soft nofile 65535
* hard nofile 65535
```

### 8.4 故障排查

**问题1: 启动失败**
```bash
# 检查端口占用
netstat -an | grep 8080
lsof -i:8080

# 检查日志
tail -f logs/catalina.out

# 检查JDK版本
java -version

# 检查权限
ls -la bin/*.sh
chmod +x bin/*.sh
```

**问题2: 内存溢出**
```bash
# 分析堆转储文件
jmap -dump:format=b,file=heapdump.hprof <pid>

# 使用MAT分析
# 下载Eclipse Memory Analyzer
# 打开heapdump.hprof文件
# 查看Leak Suspects报告
```

**问题3: 响应慢**
```bash
# 查看线程状态
jstack <pid> > thread_dump.txt

# 分析线程堆栈
# 查找BLOCKED、WAITING状态的线程
# 定位死锁或资源竞争

# 使用Arthas实时分析
trace com.example.MyService myMethod
```

**问题4: CPU占用高**
```bash
# 找出占用CPU最高的线程
top -H -p <pid>

# 转换线程ID为16进制
printf "%x\n" <thread_id>

# 在线程堆栈中查找对应线程
jstack <pid> | grep <hex_thread_id>
```

## ❓ 常见问题

### Q1: Tomcat与Jetty、Undertow的区别？

**A**: 
- **Tomcat**: 最成熟稳定，社区活跃，文档丰富，适合大多数场景
- **Jetty**: 更轻量，启动快，适合嵌入式场景和微服务
- **Undertow**: 性能最好，内存占用小，Spring Boot 2.x+默认支持

**选择建议**:
- 传统Web应用：Tomcat
- 微服务/嵌入式：Jetty或Undertow
- 高性能要求：Undertow

### Q2: 如何实现零停机部署？

**A**: 
1. **使用负载均衡器**:
   - 从负载均衡器摘除节点
   - 部署新版本
   - 健康检查通过后加入负载均衡器
   - 重复以上步骤部署其他节点

2. **使用Manager应用**:
   ```bash
   # 部署新版本到不同路径
   curl -u admin:password \
     "http://localhost:8080/manager/text/deploy?path=/myapp-v2&war=file:/path/to/myapp-v2.war"
   
   # 切换流量
   # 卸载旧版本
   curl -u admin:password \
     "http://localhost:8080/manager/text/undeploy?path=/myapp"
   ```

3. **使用蓝绿部署**:
   - 部署新版本到蓝环境
   - 测试通过后切换流量到蓝环境
   - 保留绿环境作为回滚备份

### Q3: 如何优化Tomcat启动速度？

**A**:
```xml
<!-- 1. 禁用不必要的功能 -->
<Host name="localhost"
      appBase="webapps"
      unpackWARs="true"
      autoDeploy="false"
      deployOnStartup="true">
</Host>

<!-- 2. 跳过JAR扫描 -->
<Context>
  <JarScanner>
    <JarScanFilter defaultPluggabilityScan="false"
                   defaultTldScan="false"/>
  </JarScanner>
</Context>

<!-- 3. 使用并行部署 -->
<Host startStopThreads="0">  <!-- 0表示使用CPU核心数 -->
</Host>
```

```bash
# 4. 优化JVM参数
JAVA_OPTS="-XX:+TieredCompilation \
  -XX:TieredStopAtLevel=1 \
  -Djava.security.egd=file:/dev/./urandom"
```

### Q4: 如何处理大文件上传？

**A**:
```xml
<!-- 1. 配置Connector -->
<Connector port="8080"
           maxPostSize="104857600"  <!-- 100MB -->
           maxSwallowSize="104857600"/>

<!-- 2. 配置应用 -->
<!-- web.xml -->
<servlet>
  <servlet-name>FileUploadServlet</servlet-name>
  <servlet-class>com.example.FileUploadServlet</servlet-class>
  <multipart-config>
    <max-file-size>104857600</max-file-size>  <!-- 100MB -->
    <max-request-size>104857600</max-request-size>
    <file-size-threshold>1048576</file-size-threshold>  <!-- 1MB -->
  </multipart-config>
</servlet>
```

```java
// 3. 使用流式处理
@WebServlet("/upload")
@MultipartConfig(
    maxFileSize = 100 * 1024 * 1024,  // 100MB
    maxRequestSize = 100 * 1024 * 1024,
    fileSizeThreshold = 1024 * 1024  // 1MB
)
public class FileUploadServlet extends HttpServlet {
    
    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        
        Part filePart = request.getPart("file");
        String fileName = getFileName(filePart);
        
        // 流式写入文件
        try (InputStream input = filePart.getInputStream();
             OutputStream output = new FileOutputStream("/upload/" + fileName)) {
            
            byte[] buffer = new byte[8192];
            int bytesRead;
            while ((bytesRead = input.read(buffer)) != -1) {
                output.write(buffer, 0, bytesRead);
            }
        }
        
        response.getWriter().println("上传成功");
    }
    
    private String getFileName(Part part) {
        String contentDisposition = part.getHeader("content-disposition");
        String[] tokens = contentDisposition.split(";");
        for (String token : tokens) {
            if (token.trim().startsWith("filename")) {
                return token.substring(token.indexOf("=") + 2, token.length() - 1);
            }
        }
        return "";
    }
}
```

### Q5: 如何配置HTTPS？

**A**:
```bash
# 1. 生成证书
keytool -genkey -alias tomcat -keyalg RSA -keysize 2048 \
  -keystore /opt/tomcat/conf/keystore.jks \
  -validity 365

# 2. 配置Connector
```

```xml
<Connector port="8443"
           protocol="org.apache.coyote.http11.Http11NioProtocol"
           maxThreads="150"
           SSLEnabled="true"
           scheme="https"
           secure="true"
           clientAuth="false"
           sslProtocol="TLS">
  <SSLHostConfig protocols="TLSv1.2,TLSv1.3"
                 ciphers="TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384">
    <Certificate certificateKeystoreFile="conf/keystore.jks"
                 certificateKeystorePassword="changeit"
                 type="RSA" />
  </SSLHostConfig>
</Connector>

<!-- 3. 强制HTTPS -->
<security-constraint>
  <web-resource-collection>
    <web-resource-name>Entire Application</web-resource-name>
    <url-pattern>/*</url-pattern>
  </web-resource-collection>
  <user-data-constraint>
    <transport-guarantee>CONFIDENTIAL</transport-guarantee>
  </user-data-constraint>
</security-constraint>
```

### Q6: 如何监控Tomcat性能？

**A**:
1. **使用JMX**:
   ```bash
   # 启用JMX
   CATALINA_OPTS="-Dcom.sun.management.jmxremote \
     -Dcom.sun.management.jmxremote.port=9999 \
     -Dcom.sun.management.jmxremote.ssl=false \
     -Dcom.sun.management.jmxremote.authenticate=false"
   ```

2. **使用Prometheus + Grafana**:
   ```xml
   <!-- 添加JMX Exporter -->
   CATALINA_OPTS="$CATALINA_OPTS -javaagent:/opt/jmx_exporter/jmx_prometheus_javaagent.jar=8081:/opt/jmx_exporter/config.yaml"
   ```

3. **使用APM工具**:
   - SkyWalking
   - Pinpoint
   - New Relic
   - Datadog

## 🔗 相关资源

### 官方资源
- **官方网站**: https://tomcat.apache.org/
- **官方文档**: https://tomcat.apache.org/tomcat-10.1-doc/index.html
- **GitHub仓库**: https://github.com/apache/tomcat
- **邮件列表**: https://tomcat.apache.org/lists.html

### 推荐书籍
- 《Tomcat权威指南》
- 《深入剖析Tomcat》
- 《How Tomcat Works》

### 推荐文章
- [Tomcat架构解析](https://tomcat.apache.org/tomcat-10.1-doc/architecture/overview.html)
- [Tomcat性能调优指南](https://tomcat.apache.org/tomcat-10.1-doc/performance.html)
- [Tomcat安全配置](https://tomcat.apache.org/tomcat-10.1-doc/security-howto.html)

### 相关技术
- **Servlet规范**: Jakarta Servlet Specification
- **JSP规范**: Jakarta Server Pages Specification
- **Spring Boot**: 嵌入式Tomcat集成
- **Nginx**: 反向代理和负载均衡
- **Docker**: 容器化部署

## 📝 学习检查清单

### 基础知识
- [ ] 理解Tomcat的核心架构（Server、Service、Connector、Engine、Host、Context）
- [ ] 掌握Tomcat的安装和启动
- [ ] 熟悉目录结构和核心配置文件
- [ ] 了解Servlet容器的工作原理

### 配置管理
- [ ] 掌握server.xml的核心配置
- [ ] 掌握Connector的配置和优化
- [ ] 掌握数据库连接池配置
- [ ] 掌握HTTPS配置

### 应用部署
- [ ] 掌握WAR包部署方式
- [ ] 掌握Manager应用的使用
- [ ] 了解嵌入式Tomcat的使用
- [ ] 掌握自动化部署脚本编写

### 性能优化
- [ ] 掌握JVM参数调优
- [ ] 掌握线程池配置优化
- [ ] 掌握静态资源优化
- [ ] 了解异步Servlet的使用

### 集群与高可用
- [ ] 了解Tomcat集群架构
- [ ] 掌握会话共享方案（Redis）
- [ ] 掌握负载均衡配置（Nginx）
- [ ] 了解零停机部署方案

### 监控与调优
- [ ] 掌握JMX监控配置
- [ ] 掌握日志分析方法
- [ ] 了解性能分析工具的使用
- [ ] 掌握常见问题的排查方法

### 生产实践
- [ ] 掌握生产环境安全配置
- [ ] 掌握故障排查方法
- [ ] 了解常见陷阱和最佳实践
- [ ] 掌握健康检查和监控方案

---

**文档版本**: v1.0  
**最后更新**: 2024-01-04  
**文档来源**: Context7 - Apache Tomcat官方文档  
**作者**: @author erik.zhou
