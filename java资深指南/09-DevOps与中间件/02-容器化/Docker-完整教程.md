# Docker 完整教程

## 📋 目录
- 基础概念
- 核心特性
- 实战应用
- 最佳实践
- 常见问题

## 📚 技术概述
- **版本**: Docker 24.x
- **官方文档**: https://docs.docker.com/
- **学习难度**: ⭐⭐⭐ (1-5星)
- **重要程度**: ⭐⭐⭐⭐⭐ (1-5星)
- **前置知识**: Linux基础、网络基础、虚拟化概念
- **文档来源**: Context7 - Docker官方文档
- **更新时间**: 2024-01-04
- **作者**: @author erik.zhou

## 🎯 学习目标
- [ ] 理解Docker的核心概念和架构
- [ ] 掌握镜像的构建和管理
- [ ] 熟练使用容器的生命周期管理
- [ ] 理解Docker网络模型和存储机制
- [ ] 能够编写高效的Dockerfile
- [ ] 掌握Docker Compose多容器编排

## 📖 基础概念

### 1.1 什么是Docker

Docker是一个开源的容器化平台，它允许开发者将应用程序及其依赖打包到一个轻量级、可移植的容器中，然后可以在任何支持Docker的环境中运行。

**核心价值**：
- **一致性**：开发、测试、生产环境完全一致
- **轻量级**：相比虚拟机，容器启动快、资源占用少
- **可移植性**：一次构建，到处运行
- **隔离性**：应用之间相互隔离，互不影响

### 1.2 核心概念

- **Image（镜像）**：只读模板，包含创建容器所需的指令和文件系统
- **Container（容器）**：镜像的运行实例，可以被创建、启动、停止、删除
- **Dockerfile**：包含构建镜像所需所有命令的文本文件
- **Registry（仓库）**：存储和分发Docker镜像的服务，Docker Hub是默认的公共仓库
- **Volume（卷）**：用于持久化容器数据的机制
- **Network（网络）**：容器之间通信的网络层

### 1.3 应用场景

- **微服务架构**：每个服务独立容器化部署
- **持续集成/持续部署**：快速构建、测试、部署
- **开发环境标准化**：团队成员使用相同的开发环境
- **应用隔离**：在同一主机上运行多个应用而不冲突
- **快速扩展**：根据负载快速增加或减少容器实例

### 1.4 Docker vs 虚拟机

| 特性 | Docker容器 | 虚拟机 |
|------|-----------|--------|
| 启动速度 | 秒级 | 分钟级 |
| 资源占用 | MB级 | GB级 |
| 性能 | 接近原生 | 有性能损耗 |
| 隔离级别 | 进程级 | 操作系统级 |
| 可移植性 | 高 | 中 |

## 🔥 核心特性 (重点)

### 2.1 镜像分层机制 🔥 (⚠️ 难点)

Docker镜像采用分层存储架构，每一层代表Dockerfile中的一条指令。

**分层原理**：
- 每个镜像由多个只读层组成
- 容器在镜像顶部添加一个可写层
- 多个容器可以共享相同的镜像层
- 只有修改的内容会写入容器层

**查看镜像层**：
```bash
# 查看镜像的历史层
docker image history nginx:alpine

# 输出示例
IMAGE          CREATED        CREATED BY                                      SIZE
a6eb2a334a9f   2 weeks ago    CMD ["nginx" "-g" "daemon off;"]                0B
<missing>      2 weeks ago    EXPOSE 80                                       0B
<missing>      2 weeks ago    COPY nginx.conf /etc/nginx/nginx.conf           1.2kB
<missing>      2 weeks ago    RUN apk add --no-cache nginx                    8.5MB
<missing>      2 weeks ago    FROM alpine:3.18                                7.3MB
```

**分层优势**：
1. **节省存储空间**：相同的层只存储一次
2. **加速镜像构建**：利用缓存机制，未修改的层不重新构建
3. **快速分发**：只传输变化的层

**Dockerfile示例**：

```dockerfile
# syntax=docker/dockerfile:1

# 基础层
FROM ubuntu:22.04

# 元数据层（不创建新层）
LABEL org.opencontainers.image.authors="erik.zhou@example.com"

# 文件层
COPY .. /app

# 构建层
RUN make /app

# 清理层
RUN rm -r $HOME/.cache

# 元数据层（不创建新层）
CMD python /app/app.py
```

### 2.2 容器生命周期管理 🔥

容器的完整生命周期包括：创建 → 启动 → 运行 → 暂停 → 停止 → 删除

**基本操作**：
```bash
# 创建并启动容器
docker run -d --name my-nginx nginx:alpine

# 查看运行中的容器
docker ps

# 查看所有容器（包括停止的）
docker ps -a

# 停止容器
docker stop my-nginx

# 启动已停止的容器
docker start my-nginx

# 重启容器
docker restart my-nginx

# 暂停容器
docker pause my-nginx

# 恢复暂停的容器
docker unpause my-nginx

# 删除容器
docker rm my-nginx

# 强制删除运行中的容器
docker rm -f my-nginx
```

**容器日志和监控**：
```bash
# 查看容器日志
docker logs my-nginx

# 实时查看日志
docker logs -f my-nginx

# 查看容器资源使用情况
docker stats my-nginx

# 进入容器内部
docker exec -it my-nginx /bin/sh
```

### 2.3 Docker网络模式 🔥 (⚠️ 难点)

Docker提供多种网络驱动，适用于不同场景。


#### 2.3.1 Bridge网络（默认）

**特点**：
- 容器连接到同一bridge网络可以通信
- 与其他bridge网络的容器隔离
- 适用于单主机上的容器通信

**使用示例**：
```bash
# 创建自定义bridge网络
docker network create my-net

# 在自定义网络中运行容器
docker run -d --name web --network my-net nginx:alpine

# 在同一网络中运行另一个容器并测试连通性
docker run --rm -it --network my-net busybox
/ # ping web
PING web (172.18.0.2): 56 data bytes
64 bytes from 172.18.0.2: seq=0 ttl=64 time=0.326 ms
```

#### 2.3.2 Host网络

**特点**：
- 容器直接使用主机网络栈
- 无网络隔离，性能最好
- 端口冲突风险高

**使用示例**：
```bash
# 使用host网络运行容器
docker run -d --network host nginx:alpine
```

#### 2.3.3 Overlay网络

**特点**：
- 跨多个Docker主机的分布式网络
- 用于Swarm服务或跨主机容器通信
- 支持加密传输

**使用示例**：
```bash
# 创建overlay网络（需要Swarm模式）
docker network create \
  --driver overlay \
  --attachable \
  my-overlay-network

# 创建加密的overlay网络
docker network create \
  --opt encrypted \
  --driver overlay \
  --attachable \
  my-secure-network
```

#### 2.3.4 Macvlan网络

**特点**：
- 为容器分配MAC地址，使其在网络上显示为物理设备
- 适用于需要直接连接物理网络的场景

**使用示例**：
```bash
# 创建macvlan网络
docker network create -d macvlan \
  --subnet=172.16.86.0/24 \
  --gateway=172.16.86.1 \
  -o parent=eth0 \
  my-macvlan-net
```

### 2.4 数据持久化 🔥

Docker提供三种数据持久化方式：Volumes、Bind Mounts、tmpfs。


#### 2.4.1 Volumes（推荐）

**特点**：
- 由Docker管理，存储在主机文件系统的特定位置
- 独立于容器生命周期
- 可以在多个容器间共享
- 支持远程存储驱动

**使用示例**：
```bash
# 创建volume
docker volume create my_volume

# 使用volume运行容器
docker run --rm --mount source=my_volume,target=/foo busybox \
  sh -c "echo 'hello, volume!' > /foo/hello.txt"

# 在另一个容器中读取数据
docker run --mount source=my_volume,target=/bar busybox \
  cat /bar/hello.txt
# 输出: hello, volume!

# 查看所有volumes
docker volume ls

# 删除volume
docker volume rm my_volume
```

#### 2.4.2 Bind Mounts

**特点**：
- 直接挂载主机文件系统的目录或文件
- 性能好，但依赖主机文件系统结构
- 适用于开发环境

**使用示例**：
```bash
# 挂载主机目录到容器
docker run -d \
  --name nginx-dev \
  -v /path/on/host:/usr/share/nginx/html:ro \
  nginx:alpine
```

#### 2.4.3 tmpfs Mounts

**特点**：
- 数据存储在主机内存中
- 容器停止后数据丢失
- 适用于临时数据、敏感信息

**使用示例**：
```bash
# 使用tmpfs
docker run -d \
  --name tmpfs-test \
  --tmpfs /app:rw,size=100m \
  nginx:alpine
```

## 💻 实战应用

### 3.1 环境搭建

#### 3.1.1 安装Docker

**Ubuntu/Debian**：
```bash
# 更新包索引
sudo apt-get update

# 安装依赖
sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release

# 添加Docker官方GPG密钥
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
  sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# 设置仓库
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# 安装Docker Engine
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin

# 验证安装
docker --version
```


**CentOS/RHEL**：
```bash
# 安装yum-utils
sudo yum install -y yum-utils

# 添加Docker仓库
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo

# 安装Docker Engine
sudo yum install docker-ce docker-ce-cli containerd.io docker-compose-plugin

# 启动Docker
sudo systemctl start docker
sudo systemctl enable docker
```

**配置非root用户**：
```bash
# 创建docker组
sudo groupadd docker

# 将当前用户添加到docker组
sudo usermod -aG docker $USER

# 重新登录或执行
newgrp docker

# 验证
docker run hello-world
```

### 3.2 快速开始

#### 3.2.1 运行第一个容器

```bash
# 运行nginx容器
docker run -d -p 8080:80 --name my-nginx nginx:alpine

# 访问 http://localhost:8080 查看nginx欢迎页面

# 查看容器日志
docker logs my-nginx

# 停止并删除容器
docker stop my-nginx
docker rm my-nginx
```

#### 3.2.2 构建自定义镜像

**创建Dockerfile**：
```dockerfile
# syntax=docker/dockerfile:1

# 使用官方Java运行时作为基础镜像
FROM openjdk:17-jdk-slim

# 设置工作目录
WORKDIR /app

# 复制jar文件
COPY target/myapp.jar app.jar

# 暴露端口
EXPOSE 8080

# 运行应用
ENTRYPOINT ["java", "-jar", "app.jar"]
```

**构建镜像**：
```bash
# 构建镜像
docker build -t myapp:1.0 .

# 查看镜像
docker images

# 运行容器
docker run -d -p 8080:8080 --name myapp myapp:1.0
```

### 3.3 进阶案例

#### 3.3.1 多容器应用（Spring Boot + MySQL）

**docker-compose.yml**：
```yaml
version: '3.8'

services:
  mysql:
    image: mysql:8.0
    container_name: mysql-db
    environment:
      MYSQL_ROOT_PASSWORD: root123
      MYSQL_DATABASE: myapp
      MYSQL_USER: appuser
      MYSQL_PASSWORD: apppass
    ports:
      - "3306:3306"
    volumes:
      - mysql-data:/var/lib/mysql
    networks:
      - app-network

  app:
    build: .
    container_name: spring-app
    depends_on:
      - mysql
    environment:
      SPRING_DATASOURCE_URL: jdbc:mysql://mysql:3306/myapp
      SPRING_DATASOURCE_USERNAME: appuser
      SPRING_DATASOURCE_PASSWORD: apppass
    ports:
      - "8080:8080"
    networks:
      - app-network

volumes:
  mysql-data:

networks:
  app-network:
    driver: bridge
```

**启动应用**：
```bash
# 启动所有服务
docker-compose up -d

# 查看服务状态
docker-compose ps

# 查看日志
docker-compose logs -f

# 停止所有服务
docker-compose down

# 停止并删除volumes
docker-compose down -v
```


#### 3.3.2 微服务架构部署

**docker-compose.yml**：
```yaml
version: '3.8'

services:
  # 服务注册中心
  nacos:
    image: nacos/nacos-server:v2.2.3
    container_name: nacos
    environment:
      MODE: standalone
    ports:
      - "8848:8848"
    networks:
      - microservice-network

  # 用户服务
  user-service:
    build: ./user-service
    container_name: user-service
    depends_on:
      - nacos
      - mysql
    environment:
      NACOS_SERVER_ADDR: nacos:8848
    ports:
      - "8081:8081"
    networks:
      - microservice-network

  # 订单服务
  order-service:
    build: ./order-service
    container_name: order-service
    depends_on:
      - nacos
      - mysql
    environment:
      NACOS_SERVER_ADDR: nacos:8848
    ports:
      - "8082:8082"
    networks:
      - microservice-network

  # 网关
  gateway:
    build: ./gateway
    container_name: gateway
    depends_on:
      - nacos
    environment:
      NACOS_SERVER_ADDR: nacos:8848
    ports:
      - "8080:8080"
    networks:
      - microservice-network

  # MySQL数据库
  mysql:
    image: mysql:8.0
    container_name: mysql
    environment:
      MYSQL_ROOT_PASSWORD: root123
    volumes:
      - mysql-data:/var/lib/mysql
    networks:
      - microservice-network

volumes:
  mysql-data:

networks:
  microservice-network:
    driver: bridge
```

## ✨ 最佳实践

### 4.1 Dockerfile最佳实践 🔥

#### 4.1.1 使用多阶段构建

**优势**：减小最终镜像大小，分离构建环境和运行环境

```dockerfile
# syntax=docker/dockerfile:1

# 构建阶段
FROM maven:3.8-openjdk-17 AS builder
WORKDIR /app
COPY pom.xml .
COPY src ./src
RUN mvn clean package -DskipTests

# 运行阶段
FROM openjdk:17-jdk-slim
WORKDIR /app
COPY --from=builder /app/target/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

#### 4.1.2 优化层缓存

**原则**：将变化频率低的指令放在前面

```dockerfile
# ❌ 不好的做法
FROM openjdk:17-jdk-slim
COPY . /app
RUN apt-get update && apt-get install -y curl
WORKDIR /app

# ✅ 好的做法
FROM openjdk:17-jdk-slim
RUN apt-get update && apt-get install -y curl
WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline
COPY src ./src
RUN mvn package
```


#### 4.1.3 最小化镜像大小

**技巧**：
1. 使用精简基础镜像（alpine、slim）
2. 合并RUN命令减少层数
3. 清理不必要的文件

```dockerfile
# ✅ 优化示例
FROM openjdk:17-jdk-alpine

# 合并命令，减少层数
RUN apk add --no-cache curl && \
    rm -rf /var/cache/apk/*

WORKDIR /app
COPY app.jar .

# 使用非root用户运行
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser

EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

#### 4.1.4 使用.dockerignore

**作用**：排除不需要的文件，加快构建速度

**.dockerignore示例**：
```
# 版本控制
.git
.gitignore

# IDE
.idea
.vscode
*.iml

# 构建产物
target/
build/
*.class

# 日志
*.log

# 临时文件
*.tmp
*.swp

# 文档
README.md
docs/
```

### 4.2 性能优化

#### 4.2.1 资源限制

```bash
# 限制CPU和内存
docker run -d \
  --name myapp \
  --cpus="1.5" \
  --memory="512m" \
  --memory-swap="1g" \
  myapp:1.0

# 使用docker-compose
services:
  app:
    image: myapp:1.0
    deploy:
      resources:
        limits:
          cpus: '1.5'
          memory: 512M
        reservations:
          cpus: '0.5'
          memory: 256M
```

#### 4.2.2 健康检查

```dockerfile
# Dockerfile中添加健康检查
HEALTHCHECK --interval=30s --timeout=3s --start-period=40s --retries=3 \
  CMD curl -f http://localhost:8080/actuator/health || exit 1
```

```yaml
# docker-compose.yml中配置
services:
  app:
    image: myapp:1.0
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/actuator/health"]
      interval: 30s
      timeout: 3s
      retries: 3
      start_period: 40s
```

### 4.3 安全最佳实践

#### 4.3.1 使用非root用户

```dockerfile
# 创建并切换到非root用户
FROM openjdk:17-jdk-slim

RUN groupadd -r appgroup && useradd -r -g appgroup appuser

WORKDIR /app
COPY --chown=appuser:appgroup app.jar .

USER appuser

ENTRYPOINT ["java", "-jar", "app.jar"]
```


#### 4.3.2 扫描镜像漏洞

```bash
# 使用Docker Scout扫描镜像
docker scout cves myapp:1.0

# 使用Trivy扫描
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
  aquasec/trivy image myapp:1.0
```

#### 4.3.3 使用secrets管理敏感信息

```bash
# 创建secret
echo "my_secret_password" | docker secret create db_password -

# 在服务中使用secret
docker service create \
  --name myapp \
  --secret db_password \
  myapp:1.0
```

### 4.4 常见陷阱

#### ⚠️ 陷阱1：容器内时区问题

**问题**：容器默认使用UTC时区

**解决方案**：
```dockerfile
# 方案1：设置环境变量
ENV TZ=Asia/Shanghai

# 方案2：挂载时区文件
docker run -v /etc/localtime:/etc/localtime:ro myapp:1.0

# 方案3：在Dockerfile中安装时区数据
RUN apt-get update && apt-get install -y tzdata && \
    ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \
    echo "Asia/Shanghai" > /etc/timezone
```

#### ⚠️ 陷阱2：容器日志过大

**问题**：容器日志无限增长，占满磁盘

**解决方案**：
```bash
# 配置日志驱动和大小限制
docker run -d \
  --log-driver json-file \
  --log-opt max-size=10m \
  --log-opt max-file=3 \
  myapp:1.0
```

```json
// 全局配置 /etc/docker/daemon.json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  }
}
```

#### ⚠️ 陷阱3：数据丢失

**问题**：容器删除后数据丢失

**解决方案**：使用volumes持久化数据
```bash
# 使用命名volume
docker run -d \
  -v mysql-data:/var/lib/mysql \
  mysql:8.0

# 备份volume
docker run --rm \
  -v mysql-data:/data \
  -v $(pwd):/backup \
  busybox tar czf /backup/mysql-backup.tar.gz /data
```

#### ⚠️ 陷阱4：网络连接问题

**问题**：容器之间无法通信

**解决方案**：
```bash
# 确保容器在同一网络
docker network create mynet
docker run -d --name app1 --network mynet myapp:1.0
docker run -d --name app2 --network mynet myapp:1.0

# 使用容器名称进行通信
# app1可以通过 http://app2:8080 访问app2
```

## ❓ 常见问题

### Q1: 如何清理Docker占用的磁盘空间？

**A**: 使用以下命令清理：
```bash
# 清理未使用的镜像、容器、网络、volumes
docker system prune -a --volumes

# 只清理未使用的镜像
docker image prune -a

# 只清理停止的容器
docker container prune

# 只清理未使用的volumes
docker volume prune

# 查看磁盘使用情况
docker system df
```

### Q2: 如何进入正在运行的容器？

**A**: 使用exec命令：
```bash
# 进入容器的bash shell
docker exec -it container_name /bin/bash

# 如果容器没有bash，使用sh
docker exec -it container_name /bin/sh

# 执行单个命令
docker exec container_name ls -la /app
```


### Q3: 如何将容器保存为镜像？

**A**: 使用commit命令：
```bash
# 将容器保存为新镜像
docker commit container_name new_image:tag

# 导出镜像为tar文件
docker save -o myimage.tar myimage:1.0

# 从tar文件导入镜像
docker load -i myimage.tar
```

### Q4: 如何限制容器的资源使用？

**A**: 在运行时指定资源限制：
```bash
# 限制CPU和内存
docker run -d \
  --cpus="2" \
  --memory="1g" \
  --memory-swap="2g" \
  --pids-limit=100 \
  myapp:1.0

# 限制IO
docker run -d \
  --device-read-bps /dev/sda:1mb \
  --device-write-bps /dev/sda:1mb \
  myapp:1.0
```

### Q5: 如何调试容器启动失败的问题？

**A**: 使用以下方法排查：
```bash
# 查看容器日志
docker logs container_name

# 查看容器详细信息
docker inspect container_name

# 以交互模式运行容器
docker run -it --entrypoint /bin/sh myapp:1.0

# 查看容器退出状态码
docker ps -a
# 状态码含义：
# 0: 正常退出
# 1: 应用错误
# 137: 被SIGKILL信号终止（OOM）
# 139: 段错误
```

### Q6: 如何在容器间共享数据？

**A**: 使用volumes或bind mounts：
```bash
# 方案1：使用命名volume
docker volume create shared-data
docker run -d --name app1 -v shared-data:/data myapp:1.0
docker run -d --name app2 -v shared-data:/data myapp:1.0

# 方案2：使用volumes-from
docker run -d --name data-container -v /data busybox
docker run -d --name app1 --volumes-from data-container myapp:1.0
docker run -d --name app2 --volumes-from data-container myapp:1.0
```

### Q7: 如何更新运行中的容器？

**A**: 推荐的更新流程：
```bash
# 1. 拉取新镜像
docker pull myapp:2.0

# 2. 停止旧容器
docker stop myapp

# 3. 备份旧容器（可选）
docker commit myapp myapp:1.0-backup

# 4. 删除旧容器
docker rm myapp

# 5. 使用新镜像启动容器
docker run -d --name myapp myapp:2.0

# 或使用docker-compose滚动更新
docker-compose up -d --no-deps --build app
```

## 🔗 相关资源

### 官方资源
- **官方文档**: https://docs.docker.com/
- **Docker Hub**: https://hub.docker.com/
- **Docker GitHub**: https://github.com/docker
- **Docker Blog**: https://www.docker.com/blog/

### 推荐学习资源
- **书籍**:
  - 《Docker技术入门与实战》
  - 《Docker容器与容器云》
  - 《深入浅出Docker》

- **在线教程**:
  - Docker官方教程: https://docs.docker.com/get-started/
  - Play with Docker: https://labs.play-with-docker.com/

- **视频课程**:
  - Docker从入门到实践
  - Docker容器化部署实战

### 相关技术
- **Kubernetes**: 容器编排平台
- **Docker Compose**: 多容器应用编排
- **Docker Swarm**: Docker原生集群管理
- **Podman**: Docker的替代方案

## 📝 学习检查清单

- [ ] 理解Docker的核心概念（镜像、容器、仓库）
- [ ] 掌握Docker的安装和基本配置
- [ ] 能够编写Dockerfile构建自定义镜像
- [ ] 理解镜像分层机制和缓存原理
- [ ] 掌握容器的生命周期管理
- [ ] 理解Docker的四种网络模式及应用场景
- [ ] 掌握数据持久化的三种方式（Volumes、Bind Mounts、tmpfs）
- [ ] 能够使用Docker Compose编排多容器应用
- [ ] 掌握Dockerfile最佳实践（多阶段构建、层缓存优化）
- [ ] 了解Docker安全最佳实践
- [ ] 能够进行容器性能监控和资源限制
- [ ] 掌握常见问题的排查和解决方法

## 🎓 进阶学习路径

1. **容器编排**: 学习Kubernetes进行大规模容器管理
2. **CI/CD集成**: 将Docker集成到Jenkins、GitLab CI等CI/CD流程
3. **微服务架构**: 使用Docker部署微服务应用
4. **服务网格**: 学习Istio等服务网格技术
5. **云原生**: 深入学习云原生应用开发和部署

---

**最后更新**: 2024-01-04  
**文档版本**: v1.0  
**作者**: @author erik.zhou
