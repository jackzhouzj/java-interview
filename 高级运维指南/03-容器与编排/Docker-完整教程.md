# Docker-完整教程

> @author erik.zhou

## 📋 目录
- [技术概述](#技术概述)
- [Docker基础](#docker基础)
- [镜像管理](#镜像管理)
- [容器管理](#容器管理)
- [网络管理](#网络管理)
- [存储管理](#存储管理)
- [Docker Compose](#docker-compose)
- [最佳实践](#最佳实践)

## 📚 技术概述

### 基本信息
- **重要程度**：⭐⭐⭐⭐⭐ (P0必学)
- **难度级别**：⭐⭐⭐
- **前置知识**：Linux基础
- **学习时长**：30-40小时
- **官方文档**：https://docs.docker.com/

### 学习目标
- [ ] 理解Docker架构和原理
- [ ] 掌握镜像构建和管理
- [ ] 掌握容器生命周期管理
- [ ] 理解Docker网络和存储
- [ ] 能够使用Docker Compose编排多容器应用


---

## 🐳 Docker基础

### Docker架构 🔥

```
┌─────────────────────────────────────────────────────┐
│                    Docker Client                     │
│              (docker build, run, pull)              │
└─────────────────────┬───────────────────────────────┘
                      │ REST API
┌─────────────────────▼───────────────────────────────┐
│                   Docker Daemon                      │
│                    (dockerd)                         │
├─────────────┬─────────────┬─────────────────────────┤
│   Images    │  Containers │      Networks           │
├─────────────┴─────────────┴─────────────────────────┤
│                  containerd                          │
├─────────────────────────────────────────────────────┤
│                     runc                             │
├─────────────────────────────────────────────────────┤
│              Linux Kernel (cgroups, namespaces)     │
└─────────────────────────────────────────────────────┘
```

### 核心概念

- **镜像(Image)**：只读模板，包含运行应用所需的一切
- **容器(Container)**：镜像的运行实例
- **仓库(Registry)**：存储和分发镜像的服务
- **Dockerfile**：构建镜像的脚本文件

### 安装Docker

```bash
# CentOS/RHEL
yum install -y yum-utils
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum install -y docker-ce docker-ce-cli containerd.io

# Ubuntu/Debian
apt-get update
apt-get install -y ca-certificates curl gnupg
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list
apt-get update
apt-get install -y docker-ce docker-ce-cli containerd.io

# 启动Docker
systemctl start docker
systemctl enable docker

# 验证安装
docker version
docker info
docker run hello-world
```

### 配置Docker

```bash
# 配置文件 /etc/docker/daemon.json
{
  "registry-mirrors": [
    "https://mirror.ccs.tencentyun.com",
    "https://registry.docker-cn.com"
  ],
  "insecure-registries": ["192.168.1.100:5000"],
  "data-root": "/data/docker",
  "storage-driver": "overlay2",
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m",
    "max-file": "3"
  },
  "live-restore": true,
  "default-ulimits": {
    "nofile": {
      "Name": "nofile",
      "Hard": 65536,
      "Soft": 65536
    }
  }
}

# 重新加载配置
systemctl daemon-reload
systemctl restart docker
```

---

## 📦 镜像管理

### 镜像操作 🔥

```bash
# 搜索镜像
docker search nginx

# 拉取镜像
docker pull nginx                   # 默认latest
docker pull nginx:1.24              # 指定版本
docker pull registry.example.com/nginx:1.24  # 私有仓库

# 查看镜像
docker images                       # 列出所有镜像
docker images -a                    # 包含中间层镜像
docker image ls                     # 同上
docker image inspect nginx          # 镜像详情

# 删除镜像
docker rmi nginx                    # 删除镜像
docker rmi -f nginx                 # 强制删除
docker image prune                  # 删除悬空镜像
docker image prune -a               # 删除所有未使用镜像

# 镜像标签
docker tag nginx:latest myregistry/nginx:v1

# 推送镜像
docker login registry.example.com
docker push myregistry/nginx:v1

# 导出导入
docker save nginx:latest > nginx.tar
docker save nginx:latest | gzip > nginx.tar.gz
docker load < nginx.tar
docker load -i nginx.tar.gz

# 查看镜像历史
docker history nginx
```

### Dockerfile 🔥

```dockerfile
# 基础镜像
FROM ubuntu:22.04

# 维护者信息
LABEL maintainer="erik.zhou"
LABEL version="1.0"
LABEL description="My Application"

# 设置环境变量
ENV APP_HOME=/app \
    APP_USER=appuser

# 设置工作目录
WORKDIR $APP_HOME

# 安装依赖
RUN apt-get update && apt-get install -y \
    curl \
    vim \
    && rm -rf /var/lib/apt/lists/* \
    && useradd -m $APP_USER

# 复制文件
COPY --chown=$APP_USER:$APP_USER . .
ADD https://example.com/file.tar.gz /tmp/

# 暴露端口
EXPOSE 8080

# 挂载点
VOLUME ["/data"]

# 切换用户
USER $APP_USER

# 健康检查
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:8080/health || exit 1

# 启动命令
ENTRYPOINT ["java", "-jar"]
CMD ["app.jar"]
```

### Dockerfile最佳实践 🔥

```dockerfile
# 1. 使用多阶段构建减小镜像体积
FROM golang:1.21 AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -o main .

FROM alpine:3.18
RUN apk --no-cache add ca-certificates
WORKDIR /root/
COPY --from=builder /app/main .
CMD ["./main"]

# 2. 合理利用缓存
# 将不常变化的指令放在前面
COPY package.json package-lock.json ./
RUN npm install
COPY . .

# 3. 使用.dockerignore
# .dockerignore文件内容
.git
node_modules
*.log
Dockerfile
.dockerignore

# 4. 使用非root用户
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser

# 5. 最小化层数
RUN apt-get update && apt-get install -y \
    package1 \
    package2 \
    && rm -rf /var/lib/apt/lists/*
```

### 构建镜像

```bash
# 基本构建
docker build -t myapp:v1 .

# 指定Dockerfile
docker build -f Dockerfile.prod -t myapp:prod .

# 构建参数
docker build --build-arg VERSION=1.0 -t myapp:v1 .

# 不使用缓存
docker build --no-cache -t myapp:v1 .

# 多平台构建
docker buildx build --platform linux/amd64,linux/arm64 -t myapp:v1 .
```

---

## 🚀 容器管理

### 容器生命周期 🔥

```bash
# 创建并启动容器
docker run nginx                    # 前台运行
docker run -d nginx                 # 后台运行
docker run -it ubuntu bash          # 交互式运行
docker run --name mynginx nginx     # 指定名称
docker run --rm nginx               # 退出后自动删除
docker run -p 8080:80 nginx         # 端口映射
docker run -P nginx                 # 随机端口映射
docker run -v /host:/container nginx    # 挂载目录
docker run -e "ENV_VAR=value" nginx     # 环境变量
docker run --restart=always nginx       # 重启策略

# 完整示例
docker run -d \
    --name mynginx \
    -p 80:80 \
    -v /data/nginx/html:/usr/share/nginx/html \
    -v /data/nginx/conf:/etc/nginx/conf.d \
    -e TZ=Asia/Shanghai \
    --restart=always \
    nginx:1.24

# 查看容器
docker ps                           # 运行中的容器
docker ps -a                        # 所有容器
docker ps -q                        # 只显示ID
docker inspect container_name       # 容器详情

# 容器操作
docker start container_name         # 启动
docker stop container_name          # 停止
docker restart container_name       # 重启
docker pause container_name         # 暂停
docker unpause container_name       # 恢复
docker kill container_name          # 强制停止

# 删除容器
docker rm container_name            # 删除已停止的容器
docker rm -f container_name         # 强制删除
docker container prune              # 删除所有已停止容器

# 进入容器
docker exec -it container_name bash
docker exec -it container_name sh
docker attach container_name        # 附加到容器（不推荐）

# 查看日志
docker logs container_name          # 查看日志
docker logs -f container_name       # 实时跟踪
docker logs --tail 100 container_name   # 最后100行
docker logs --since 1h container_name   # 最近1小时

# 复制文件
docker cp container_name:/path/file ./local/
docker cp ./local/file container_name:/path/

# 查看资源使用
docker stats                        # 实时资源使用
docker top container_name           # 容器内进程

# 导出导入容器
docker export container_name > container.tar
docker import container.tar myimage:v1
```

### 资源限制 🔥

```bash
# CPU限制
docker run -d --cpus=2 nginx                    # 限制使用2个CPU
docker run -d --cpu-shares=512 nginx            # CPU权重
docker run -d --cpuset-cpus="0,1" nginx         # 绑定CPU核心

# 内存限制
docker run -d -m 512m nginx                     # 限制内存512MB
docker run -d --memory=512m --memory-swap=1g nginx  # 内存+swap

# 完整示例
docker run -d \
    --name myapp \
    --cpus=2 \
    --memory=1g \
    --memory-swap=2g \
    --restart=always \
    myapp:v1
```

---

## 🌐 网络管理

### Docker网络模式 🔥

```bash
# 网络模式
# bridge  - 默认模式，容器有独立网络命名空间
# host    - 使用宿主机网络
# none    - 无网络
# container - 共享其他容器的网络

# 查看网络
docker network ls
docker network inspect bridge

# 创建网络
docker network create mynet
docker network create --driver bridge --subnet 172.20.0.0/16 mynet

# 使用网络
docker run -d --network mynet --name app1 nginx
docker run -d --network mynet --name app2 nginx
# app1和app2可以通过容器名互相访问

# 连接/断开网络
docker network connect mynet container_name
docker network disconnect mynet container_name

# 删除网络
docker network rm mynet
docker network prune                # 删除未使用的网络
```

### 端口映射

```bash
# 端口映射
docker run -p 8080:80 nginx         # 映射到所有接口
docker run -p 127.0.0.1:8080:80 nginx   # 映射到指定IP
docker run -p 8080-8090:80-90 nginx     # 端口范围
docker run -P nginx                 # 随机端口

# 查看端口映射
docker port container_name
```

---

## 💾 存储管理

### 数据卷 🔥

```bash
# 创建数据卷
docker volume create myvolume

# 查看数据卷
docker volume ls
docker volume inspect myvolume

# 使用数据卷
docker run -v myvolume:/data nginx
docker run --mount source=myvolume,target=/data nginx

# 绑定挂载
docker run -v /host/path:/container/path nginx
docker run -v /host/path:/container/path:ro nginx  # 只读

# 删除数据卷
docker volume rm myvolume
docker volume prune                 # 删除未使用的卷

# 备份数据卷
docker run --rm -v myvolume:/data -v $(pwd):/backup alpine \
    tar cvf /backup/backup.tar /data
```

---

## 🎼 Docker Compose

### docker-compose.yml 🔥

```yaml
version: '3.8'

services:
  web:
    image: nginx:1.24
    container_name: web
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/conf:/etc/nginx/conf.d
      - ./nginx/html:/usr/share/nginx/html
      - ./nginx/logs:/var/log/nginx
    environment:
      - TZ=Asia/Shanghai
    depends_on:
      - app
    networks:
      - frontend
    restart: always

  app:
    build:
      context: ./app
      dockerfile: Dockerfile
    container_name: app
    ports:
      - "8080:8080"
    environment:
      - SPRING_PROFILES_ACTIVE=prod
      - DB_HOST=db
      - REDIS_HOST=redis
    depends_on:
      - db
      - redis
    networks:
      - frontend
      - backend
    restart: always
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  db:
    image: mysql:8.0
    container_name: db
    ports:
      - "3306:3306"
    volumes:
      - mysql_data:/var/lib/mysql
      - ./mysql/conf:/etc/mysql/conf.d
      - ./mysql/init:/docker-entrypoint-initdb.d
    environment:
      - MYSQL_ROOT_PASSWORD=rootpassword
      - MYSQL_DATABASE=myapp
      - MYSQL_USER=appuser
      - MYSQL_PASSWORD=apppassword
    networks:
      - backend
    restart: always

  redis:
    image: redis:7.2
    container_name: redis
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    command: redis-server --appendonly yes
    networks:
      - backend
    restart: always

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge

volumes:
  mysql_data:
  redis_data:
```

### Compose命令 🔥

```bash
# 启动服务
docker-compose up                   # 前台启动
docker-compose up -d                # 后台启动
docker-compose up --build           # 重新构建
docker-compose up -d --scale web=3  # 扩展实例

# 停止服务
docker-compose down                 # 停止并删除容器
docker-compose down -v              # 同时删除卷
docker-compose stop                 # 仅停止

# 查看状态
docker-compose ps
docker-compose logs
docker-compose logs -f web

# 执行命令
docker-compose exec web bash
docker-compose run web bash

# 其他命令
docker-compose build                # 构建镜像
docker-compose pull                 # 拉取镜像
docker-compose restart              # 重启服务
docker-compose config               # 验证配置
```

---

## 💡 最佳实践

### 镜像优化

1. **使用小基础镜像**：alpine、distroless
2. **多阶段构建**：减小最终镜像体积
3. **合理利用缓存**：不常变化的指令放前面
4. **清理不必要文件**：删除缓存、临时文件
5. **使用.dockerignore**：排除不需要的文件

### 安全建议

1. **使用非root用户**运行容器
2. **只读文件系统**：`--read-only`
3. **限制资源**：CPU、内存限制
4. **扫描镜像漏洞**：Trivy、Clair
5. **使用可信镜像**：官方镜像或自建

### 运维建议

1. **配置日志轮转**：避免日志占满磁盘
2. **设置重启策略**：`--restart=always`
3. **健康检查**：HEALTHCHECK指令
4. **资源监控**：docker stats
5. **定期清理**：prune命令

---

## 📝 学习检查清单

- [ ] 理解Docker架构和核心概念
- [ ] 能够编写高效的Dockerfile
- [ ] 掌握镜像构建和管理
- [ ] 掌握容器生命周期管理
- [ ] 理解Docker网络模式
- [ ] 能够使用Docker Compose编排应用
- [ ] 了解Docker安全最佳实践

---

@author erik.zhou
