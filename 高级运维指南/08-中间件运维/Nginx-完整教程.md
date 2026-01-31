# Nginx-完整教程

> @author erik.zhou

## 📋 目录
- [技术概述](#技术概述)
- [安装配置](#安装配置)
- [核心配置](#核心配置)
- [反向代理](#反向代理)
- [负载均衡](#负载均衡)
- [HTTPS配置](#https配置)
- [性能优化](#性能优化)
- [常见问题](#常见问题)

## 📚 技术概述

### 基本信息
- **重要程度**：⭐⭐⭐⭐⭐ (P0必学)
- **难度级别**：⭐⭐⭐
- **前置知识**：Linux、网络基础
- **学习时长**：25-35小时
- **官方文档**：https://nginx.org/en/docs/

### 学习目标
- [ ] 掌握Nginx配置
- [ ] 配置反向代理
- [ ] 配置负载均衡
- [ ] 进行性能优化


---

## 🔧 安装配置

### 安装方式

```bash
# CentOS/RHEL - yum安装
yum install -y epel-release
yum install -y nginx

# Ubuntu/Debian - apt安装
apt-get update
apt-get install -y nginx

# 编译安装
wget http://nginx.org/download/nginx-1.24.0.tar.gz
tar -xzf nginx-1.24.0.tar.gz
cd nginx-1.24.0

./configure \
    --prefix=/usr/local/nginx \
    --with-http_ssl_module \
    --with-http_v2_module \
    --with-http_realip_module \
    --with-http_stub_status_module \
    --with-http_gzip_static_module \
    --with-pcre \
    --with-stream \
    --with-stream_ssl_module

make && make install

# 启动服务
systemctl start nginx
systemctl enable nginx

# 验证
nginx -v
nginx -t
curl http://localhost
```

### 目录结构

```
/etc/nginx/
├── nginx.conf              # 主配置文件
├── conf.d/                 # 自定义配置目录
│   └── default.conf
├── sites-available/        # 可用站点（Debian系）
├── sites-enabled/          # 启用站点
├── mime.types              # MIME类型
└── fastcgi_params          # FastCGI参数

/var/log/nginx/
├── access.log              # 访问日志
└── error.log               # 错误日志

/usr/share/nginx/html/      # 默认网站目录
```

---

## ⚙️ 核心配置

### 配置结构 🔥

```nginx
# nginx.conf
user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log warn;
pid /run/nginx.pid;

events {
    worker_connections 10240;
    use epoll;
    multi_accept on;
}

http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    # 日志格式
    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for" '
                    '$request_time $upstream_response_time';

    access_log /var/log/nginx/access.log main;

    # 基础优化
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    types_hash_max_size 2048;

    # Gzip压缩
    gzip on;
    gzip_vary on;
    gzip_proxied any;
    gzip_comp_level 6;
    gzip_types text/plain text/css text/xml application/json 
               application/javascript application/xml;

    # 包含其他配置
    include /etc/nginx/conf.d/*.conf;
}
```

### Server配置 🔥

```nginx
# /etc/nginx/conf.d/example.conf
server {
    listen 80;
    listen [::]:80;
    server_name example.com www.example.com;
    root /var/www/html;
    index index.html index.htm;

    # 访问日志
    access_log /var/log/nginx/example.access.log main;
    error_log /var/log/nginx/example.error.log;

    # 静态文件
    location / {
        try_files $uri $uri/ =404;
    }

    # 静态资源缓存
    location ~* \.(jpg|jpeg|png|gif|ico|css|js)$ {
        expires 30d;
        add_header Cache-Control "public, immutable";
    }

    # 禁止访问隐藏文件
    location ~ /\. {
        deny all;
    }

    # 错误页面
    error_page 404 /404.html;
    error_page 500 502 503 504 /50x.html;
    location = /50x.html {
        root /usr/share/nginx/html;
    }
}
```

---

## 🔄 反向代理

### 基本反向代理 🔥

```nginx
server {
    listen 80;
    server_name api.example.com;

    location / {
        proxy_pass http://127.0.0.1:8080;
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

### WebSocket代理

```nginx
server {
    listen 80;
    server_name ws.example.com;

    location /ws {
        proxy_pass http://127.0.0.1:8080;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_read_timeout 3600s;
    }
}
```

---

## ⚖️ 负载均衡

### 负载均衡配置 🔥

```nginx
# 上游服务器组
upstream backend {
    # 负载均衡算法
    # 默认轮询
    # least_conn;      # 最少连接
    # ip_hash;         # IP哈希
    # hash $request_uri consistent;  # 一致性哈希

    server 192.168.1.10:8080 weight=5;
    server 192.168.1.11:8080 weight=3;
    server 192.168.1.12:8080 weight=2;
    server 192.168.1.13:8080 backup;      # 备用服务器
    server 192.168.1.14:8080 down;        # 下线服务器

    # 健康检查（商业版）
    # health_check interval=5s fails=3 passes=2;

    # 连接保持
    keepalive 32;
}

server {
    listen 80;
    server_name app.example.com;

    location / {
        proxy_pass http://backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_http_version 1.1;
        proxy_set_header Connection "";
    }
}
```

### 会话保持

```nginx
upstream backend {
    ip_hash;  # 基于IP的会话保持
    server 192.168.1.10:8080;
    server 192.168.1.11:8080;
}

# 或使用cookie
upstream backend {
    hash $cookie_sessionid consistent;
    server 192.168.1.10:8080;
    server 192.168.1.11:8080;
}
```

---

## 🔒 HTTPS配置

### SSL配置 🔥

```nginx
server {
    listen 80;
    server_name example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name example.com;

    # SSL证书
    ssl_certificate /etc/nginx/ssl/example.com.crt;
    ssl_certificate_key /etc/nginx/ssl/example.com.key;

    # SSL优化
    ssl_session_timeout 1d;
    ssl_session_cache shared:SSL:50m;
    ssl_session_tickets off;

    # 协议和加密套件
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256;
    ssl_prefer_server_ciphers off;

    # HSTS
    add_header Strict-Transport-Security "max-age=63072000" always;

    # OCSP Stapling
    ssl_stapling on;
    ssl_stapling_verify on;
    resolver 8.8.8.8 8.8.4.4 valid=300s;

    location / {
        root /var/www/html;
        index index.html;
    }
}
```

---

## 🚀 性能优化

### 系统优化

```bash
# /etc/sysctl.conf
net.core.somaxconn = 65535
net.core.netdev_max_backlog = 65535
net.ipv4.tcp_max_syn_backlog = 65535
net.ipv4.tcp_fin_timeout = 10
net.ipv4.tcp_tw_reuse = 1
net.ipv4.ip_local_port_range = 1024 65535

# 应用配置
sysctl -p
```

### Nginx优化配置 🔥

```nginx
# 工作进程
worker_processes auto;
worker_rlimit_nofile 65535;

events {
    worker_connections 65535;
    use epoll;
    multi_accept on;
}

http {
    # 文件传输优化
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;

    # 连接优化
    keepalive_timeout 65;
    keepalive_requests 10000;

    # 缓冲优化
    client_body_buffer_size 10K;
    client_header_buffer_size 1k;
    client_max_body_size 100m;
    large_client_header_buffers 4 32k;

    # 超时优化
    client_body_timeout 12;
    client_header_timeout 12;
    send_timeout 10;

    # Gzip压缩
    gzip on;
    gzip_min_length 1k;
    gzip_comp_level 4;
    gzip_types text/plain application/javascript text/css application/json;
    gzip_vary on;
    gzip_disable "MSIE [1-6]\.";

    # 静态文件缓存
    open_file_cache max=10000 inactive=20s;
    open_file_cache_valid 30s;
    open_file_cache_min_uses 2;
    open_file_cache_errors on;
}
```

---

## ❓ 常见问题

### 问题1：502 Bad Gateway

**原因**：后端服务不可用

**排查**：
```bash
# 检查后端服务
curl http://backend:port

# 查看错误日志
tail -f /var/log/nginx/error.log

# 检查upstream配置
nginx -t
```

### 问题2：504 Gateway Timeout

**原因**：后端响应超时

**解决**：
```nginx
proxy_connect_timeout 300s;
proxy_send_timeout 300s;
proxy_read_timeout 300s;
```

### 问题3：413 Request Entity Too Large

**原因**：请求体过大

**解决**：
```nginx
client_max_body_size 100m;
```

---

## 📝 学习检查清单

- [ ] 掌握Nginx配置结构
- [ ] 能够配置反向代理
- [ ] 能够配置负载均衡
- [ ] 能够配置HTTPS
- [ ] 能够进行性能优化
- [ ] 能够排查常见问题

---

@author erik.zhou
