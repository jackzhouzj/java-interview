# Kubernetes 完整教程

## 📋 目录
- 基础概念
- 核心特性
- 实战应用
- 最佳实践
- 常见问题

## 📚 技术概述
- **版本**: Kubernetes 1.28+
- **官方文档**: https://kubernetes.io/docs/
- **学习难度**: ⭐⭐⭐⭐⭐ (1-5星)
- **重要程度**: ⭐⭐⭐⭐⭐ (1-5星)
- **前置知识**: Docker容器、Linux基础、网络基础、分布式系统概念
- **文档来源**: Context7 - Kubernetes官方文档
- **更新时间**: 2024-01-04
- **作者**: @author erik.zhou

## 🎯 学习目标
- [ ] 理解Kubernetes的架构和核心概念
- [ ] 掌握Pod、Service、Deployment等核心资源
- [ ] 熟练使用kubectl命令行工具
- [ ] 理解Kubernetes的调度策略和网络模型
- [ ] 掌握ConfigMap和Secret的配置管理
- [ ] 理解持久化存储机制（PV、PVC、StorageClass）
- [ ] 能够部署和管理有状态应用（StatefulSet）
- [ ] 掌握服务发现和负载均衡
- [ ] 了解Kubernetes的安全机制

## 📖 基础概念

### 1.1 什么是Kubernetes

Kubernetes（简称K8s）是一个开源的容器编排平台，用于自动化部署、扩展和管理容器化应用程序。它最初由Google设计，现在由Cloud Native Computing Foundation（CNCF）维护。

**核心价值**：
- **自动化部署**：自动化容器的部署和回滚
- **服务发现和负载均衡**：自动分配IP和DNS名称，并进行负载均衡
- **存储编排**：自动挂载存储系统
- **自我修复**：自动重启失败的容器，替换和重新调度容器
- **水平扩展**：根据CPU使用率或其他指标自动扩展应用
- **密钥和配置管理**：安全地存储和管理敏感信息

### 1.2 核心概念

- **Cluster（集群）**：一组运行容器化应用的节点（物理机或虚拟机）
- **Node（节点）**：集群中的工作机器，可以是物理机或虚拟机
- **Pod**：Kubernetes中最小的部署单元，包含一个或多个容器
- **Service**：定义一组Pod的访问策略，提供稳定的网络端点
- **Deployment**：声明式地管理Pod和ReplicaSet
- **ConfigMap**：存储非敏感配置数据
- **Secret**：存储敏感信息（如密码、令牌）
- **Namespace**：虚拟集群，用于资源隔离

### 1.3 Kubernetes架构


#### 1.3.1 控制平面组件（Master节点）

- **kube-apiserver**：API服务器，集群的统一入口
- **etcd**：分布式键值存储，保存集群所有数据
- **kube-scheduler**：调度器，负责Pod的调度
- **kube-controller-manager**：控制器管理器，运行各种控制器
- **cloud-controller-manager**：云控制器管理器，与云提供商交互

#### 1.3.2 节点组件（Worker节点）

- **kubelet**：节点代理，确保容器在Pod中运行
- **kube-proxy**：网络代理，维护节点上的网络规则
- **Container Runtime**：容器运行时（如Docker、containerd）

```
┌─────────────────────────────────────────────────────────┐
│                    Control Plane                         │
│  ┌──────────┐  ┌──────┐  ┌───────────┐  ┌────────────┐ │
│  │ API      │  │ etcd │  │ Scheduler │  │ Controller │ │
│  │ Server   │  │      │  │           │  │ Manager    │ │
│  └──────────┘  └──────┘  └───────────┘  └────────────┘ │
└─────────────────────────────────────────────────────────┘
                          │
        ┌─────────────────┼─────────────────┐
        │                 │                 │
┌───────▼────────┐ ┌──────▼────────┐ ┌─────▼─────────┐
│  Worker Node 1 │ │ Worker Node 2 │ │ Worker Node 3 │
│  ┌──────────┐  │ │  ┌──────────┐ │ │  ┌──────────┐ │
│  │ kubelet  │  │ │  │ kubelet  │ │ │  │ kubelet  │ │
│  │ kube-    │  │ │  │ kube-    │ │ │  │ kube-    │ │
│  │ proxy    │  │ │  │ proxy    │ │ │  │ proxy    │ │
│  │ Pods     │  │ │  │ Pods     │ │ │  │ Pods     │ │
│  └──────────┘  │ │  └──────────┘ │ │  └──────────┘ │
└────────────────┘ └───────────────┘ └───────────────┘
```

### 1.4 应用场景

- **微服务架构**：管理大量微服务容器
- **持续集成/持续部署**：自动化应用部署和更新
- **混合云和多云部署**：跨云平台统一管理
- **大数据和机器学习**：管理计算密集型任务
- **边缘计算**：在边缘节点部署应用

## 🔥 核心特性 (重点)

### 2.1 Pod - 最小部署单元 🔥

Pod是Kubernetes中最小的可部署单元，包含一个或多个紧密耦合的容器。

**Pod特点**：
- 共享网络命名空间（同一Pod内容器可通过localhost通信）
- 共享存储卷
- 共享生命周期
- 通常一个Pod运行一个容器（单容器模式）

**创建Pod示例**：
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.21
    ports:
    - containerPort: 80
      protocol: TCP
    resources:
      limits:
        cpu: "500m"
        memory: "512Mi"
      requests:
        cpu: "250m"
        memory: "256Mi"
```

**使用kubectl创建Pod**：
```bash
# 从YAML文件创建
kubectl apply -f nginx-pod.yaml

# 查看Pod
kubectl get pods

# 查看Pod详细信息
kubectl describe pod nginx-pod

# 查看Pod日志
kubectl logs nginx-pod

# 进入Pod容器
kubectl exec -it nginx-pod -- /bin/bash

# 删除Pod
kubectl delete pod nginx-pod
```


### 2.2 Deployment - 声明式部署 🔥

Deployment提供Pod和ReplicaSet的声明式更新能力。

**Deployment特点**：
- 声明式管理Pod副本数量
- 支持滚动更新和回滚
- 自动创建和管理ReplicaSet
- 支持暂停和恢复部署

**Deployment示例**：
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        ports:
        - containerPort: 80
        resources:
          limits:
            cpu: "500m"
            memory: "512Mi"
          requests:
            cpu: "250m"
            memory: "256Mi"
```

**Deployment操作**：
```bash
# 创建Deployment
kubectl apply -f nginx-deployment.yaml

# 查看Deployment
kubectl get deployments

# 查看ReplicaSet
kubectl get rs

# 查看Pod
kubectl get pods

# 扩容/缩容
kubectl scale deployment nginx-deployment --replicas=5

# 更新镜像（滚动更新）
kubectl set image deployment/nginx-deployment nginx=nginx:1.22

# 查看更新状态
kubectl rollout status deployment/nginx-deployment

# 查看更新历史
kubectl rollout history deployment/nginx-deployment

# 回滚到上一个版本
kubectl rollout undo deployment/nginx-deployment

# 回滚到指定版本
kubectl rollout undo deployment/nginx-deployment --to-revision=2

# 暂停部署
kubectl rollout pause deployment/nginx-deployment

# 恢复部署
kubectl rollout resume deployment/nginx-deployment

# 删除Deployment
kubectl delete deployment nginx-deployment
```

### 2.3 Service - 服务发现与负载均衡 🔥

Service为一组Pod提供稳定的网络端点和负载均衡。

**Service类型**：
1. **ClusterIP**（默认）：集群内部访问
2. **NodePort**：通过节点端口暴露服务
3. **LoadBalancer**：使用云提供商的负载均衡器
4. **ExternalName**：映射到外部DNS名称

**ClusterIP Service示例**：
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  type: ClusterIP
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
```

**NodePort Service示例**：
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-nodeport
spec:
  selector:
    app: nginx
  type: NodePort
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
    nodePort: 30080  # 30000-32767范围
```

**LoadBalancer Service示例**：
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-lb
spec:
  selector:
    app: nginx
  type: LoadBalancer
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
```

**Service操作**：
```bash
# 创建Service
kubectl apply -f nginx-service.yaml

# 查看Service
kubectl get services
kubectl get svc

# 查看Service详细信息
kubectl describe service nginx-service

# 查看Service的Endpoints
kubectl get endpoints nginx-service

# 删除Service
kubectl delete service nginx-service
```


### 2.4 ConfigMap和Secret - 配置管理 🔥

#### 2.4.1 ConfigMap

ConfigMap用于存储非敏感的配置数据。

**创建ConfigMap**：
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  database.url: "postgres://db:5432/myapp"
  cache.enabled: "true"
  logging.level: "info"
  app.properties: |
    key1=value1
    key2=value2
```

**使用ConfigMap**：
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
  - name: app
    image: myapp:latest
    # 方式1：环境变量
    env:
    - name: DATABASE_URL
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: database.url
    # 方式2：挂载为文件
    volumeMounts:
    - name: config
      mountPath: /etc/config
      readOnly: true
  volumes:
  - name: config
    configMap:
      name: app-config
```

**ConfigMap操作**：
```bash
# 从文件创建
kubectl create configmap app-config --from-file=config.properties

# 从字面值创建
kubectl create configmap app-config \
  --from-literal=database.url=postgres://db:5432/myapp \
  --from-literal=cache.enabled=true

# 查看ConfigMap
kubectl get configmaps
kubectl describe configmap app-config

# 编辑ConfigMap
kubectl edit configmap app-config

# 删除ConfigMap
kubectl delete configmap app-config
```

#### 2.4.2 Secret

Secret用于存储敏感信息（如密码、令牌、密钥）。

**创建Secret**：
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secrets
type: Opaque
stringData:
  db-password: "super-secret-password"
  api-key: "abc123xyz789"
```

**使用Secret**：
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
  - name: app
    image: myapp:latest
    env:
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: app-secrets
          key: db-password
    volumeMounts:
    - name: secrets
      mountPath: /etc/secrets
      readOnly: true
  volumes:
  - name: secrets
    secret:
      secretName: app-secrets
```

**Secret操作**：
```bash
# 从文件创建
kubectl create secret generic app-secrets --from-file=password.txt

# 从字面值创建
kubectl create secret generic app-secrets \
  --from-literal=db-password=mypassword \
  --from-literal=api-key=myapikey

# 创建TLS Secret
kubectl create secret tls tls-secret \
  --cert=path/to/tls.crt \
  --key=path/to/tls.key

# 查看Secret
kubectl get secrets
kubectl describe secret app-secrets

# 查看Secret内容（base64编码）
kubectl get secret app-secrets -o yaml

# 删除Secret
kubectl delete secret app-secrets
```

### 2.5 持久化存储 🔥 (⚠️ 难点)

#### 2.5.1 PersistentVolume (PV) 和 PersistentVolumeClaim (PVC)

**PersistentVolume**：集群级别的存储资源
**PersistentVolumeClaim**：用户对存储的请求

**PV示例**：
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-data
spec:
  capacity:
    storage: 10Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: fast-ssd
  hostPath:
    path: /mnt/data
    type: DirectoryOrCreate
```


**PVC示例**：
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-data
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests:
      storage: 5Gi
  storageClassName: fast-ssd
```

**在Pod中使用PVC**：
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-with-storage
spec:
  containers:
  - name: app
    image: myapp:latest
    volumeMounts:
    - name: data
      mountPath: /data
  volumes:
  - name: data
    persistentVolumeClaim:
      claimName: pvc-data
```

**访问模式**：
- **ReadWriteOnce (RWO)**：单节点读写
- **ReadOnlyMany (ROX)**：多节点只读
- **ReadWriteMany (RWX)**：多节点读写

**回收策略**：
- **Retain**：保留数据，需手动清理
- **Delete**：删除PVC时自动删除PV和存储
- **Recycle**：基本清理后重新可用（已废弃）

#### 2.5.2 StorageClass - 动态存储供应

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-ssd
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp3
  iopsPerGB: "10"
  fsType: ext4
reclaimPolicy: Delete
allowVolumeExpansion: true
volumeBindingMode: WaitForFirstConsumer
```

### 2.6 StatefulSet - 有状态应用 🔥 (⚠️ 难点)

StatefulSet用于管理有状态应用，提供稳定的网络标识和持久化存储。

**StatefulSet特点**：
- 稳定的网络标识（Pod名称固定）
- 稳定的持久化存储
- 有序的部署和扩展
- 有序的删除和终止

**StatefulSet示例**：
```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql-headless
spec:
  clusterIP: None
  selector:
    app: mysql
  ports:
  - port: 3306
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
spec:
  serviceName: mysql-headless
  replicas: 3
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:8.0
        ports:
        - containerPort: 3306
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: password
        volumeMounts:
        - name: data
          mountPath: /var/lib/mysql
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: fast-ssd
      resources:
        requests:
          storage: 10Gi
```

**StatefulSet Pod命名**：
- mysql-0
- mysql-1
- mysql-2

**访问StatefulSet Pod**：
```bash
# 通过Headless Service访问
mysql-0.mysql-headless.default.svc.cluster.local
mysql-1.mysql-headless.default.svc.cluster.local
mysql-2.mysql-headless.default.svc.cluster.local
```

### 2.7 调度策略 (⚠️ 难点)

#### 2.7.1 节点选择器（NodeSelector）

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  nodeSelector:
    disktype: ssd
  containers:
  - name: nginx
    image: nginx
```

#### 2.7.2 节点亲和性（Node Affinity）

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: kubernetes.io/zone
            operator: In
            values:
            - us-east-1a
            - us-east-1b
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
          - key: disktype
            operator: In
            values:
            - ssd
  containers:
  - name: nginx
    image: nginx
```


#### 2.7.3 Pod亲和性和反亲和性

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web
spec:
  affinity:
    # Pod亲和性：与cache Pod部署在同一节点
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: app
            operator: In
            values:
            - cache
        topologyKey: kubernetes.io/hostname
    # Pod反亲和性：不与其他web Pod部署在同一节点
    podAntiAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        podAffinityTerm:
          labelSelector:
            matchExpressions:
            - key: app
              operator: In
              values:
              - web
          topologyKey: kubernetes.io/hostname
  containers:
  - name: web
    image: nginx
```

#### 2.7.4 污点（Taint）和容忍（Toleration）

**添加污点**：
```bash
# 添加污点
kubectl taint nodes node1 key=value:NoSchedule

# 查看节点污点
kubectl describe node node1 | grep Taints

# 删除污点
kubectl taint nodes node1 key:NoSchedule-
```

**容忍污点**：
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  tolerations:
  - key: "key"
    operator: "Equal"
    value: "value"
    effect: "NoSchedule"
  containers:
  - name: nginx
    image: nginx
```

### 2.8 网络模型 (⚠️ 难点)

Kubernetes网络模型要求：
1. 所有Pod可以在不使用NAT的情况下与其他Pod通信
2. 所有节点可以在不使用NAT的情况下与所有Pod通信
3. Pod看到的自己的IP与其他Pod看到的IP相同

**常见网络插件**：
- **Calico**：支持网络策略，性能好
- **Flannel**：简单易用，适合小规模集群
- **Weave Net**：自动配置，易于部署
- **Cilium**：基于eBPF，高性能

**网络策略示例**：
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-from-frontend
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - protocol: TCP
      port: 8080
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: database
    ports:
    - protocol: TCP
      port: 3306
```

## 💻 实战应用

### 3.1 环境搭建

#### 3.1.1 使用Minikube（本地开发）

```bash
# 安装Minikube
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube

# 启动Minikube
minikube start --driver=docker --cpus=4 --memory=8192

# 查看状态
minikube status

# 访问Dashboard
minikube dashboard

# 停止Minikube
minikube stop

# 删除Minikube
minikube delete
```

#### 3.1.2 使用kubeadm（生产环境）

**Master节点初始化**：
```bash
# 初始化集群
sudo kubeadm init --pod-network-cidr=10.244.0.0/16

# 配置kubectl
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# 安装网络插件（Flannel）
kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml
```

**Worker节点加入**：
```bash
# 在Worker节点执行（token从master节点获取）
sudo kubeadm join <master-ip>:6443 --token <token> \
  --discovery-token-ca-cert-hash sha256:<hash>
```


### 3.2 快速开始

#### 3.2.1 部署第一个应用

```bash
# 创建Deployment
kubectl create deployment nginx --image=nginx:1.21

# 暴露服务
kubectl expose deployment nginx --port=80 --type=NodePort

# 查看服务
kubectl get services

# 访问应用（Minikube）
minikube service nginx --url

# 扩容
kubectl scale deployment nginx --replicas=3

# 查看Pod
kubectl get pods -o wide

# 删除资源
kubectl delete deployment nginx
kubectl delete service nginx
```

#### 3.2.2 使用YAML文件部署

**创建完整应用**：
```yaml
# app.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: myapp
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
  namespace: myapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        ports:
        - containerPort: 80
        resources:
          limits:
            cpu: "500m"
            memory: "512Mi"
          requests:
            cpu: "250m"
            memory: "256Mi"
---
apiVersion: v1
kind: Service
metadata:
  name: web-service
  namespace: myapp
spec:
  selector:
    app: web
  type: LoadBalancer
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
```

**部署应用**：
```bash
# 应用配置
kubectl apply -f app.yaml

# 查看资源
kubectl get all -n myapp

# 查看详细信息
kubectl describe deployment web -n myapp

# 删除应用
kubectl delete -f app.yaml
```

### 3.3 进阶案例

#### 3.3.1 部署Spring Boot微服务

**完整部署文件**：
```yaml
# spring-boot-app.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: default
data:
  application.yml: |
    spring:
      datasource:
        url: jdbc:mysql://mysql:3306/myapp
        username: root
      redis:
        host: redis
        port: 6379
---
apiVersion: v1
kind: Secret
metadata:
  name: app-secrets
  namespace: default
type: Opaque
stringData:
  db-password: "mypassword"
  redis-password: "redispass"
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: spring-boot-app
  namespace: default
spec:
  replicas: 3
  selector:
    matchLabels:
      app: spring-boot-app
  template:
    metadata:
      labels:
        app: spring-boot-app
    spec:
      containers:
      - name: app
        image: myapp:1.0
        ports:
        - containerPort: 8080
        env:
        - name: SPRING_DATASOURCE_PASSWORD
          valueFrom:
            secretKeyRef:
              name: app-secrets
              key: db-password
        - name: SPRING_REDIS_PASSWORD
          valueFrom:
            secretKeyRef:
              name: app-secrets
              key: redis-password
        volumeMounts:
        - name: config
          mountPath: /config
          readOnly: true
        livenessProbe:
          httpGet:
            path: /actuator/health/liveness
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /actuator/health/readiness
            port: 8080
          initialDelaySeconds: 20
          periodSeconds: 5
        resources:
          limits:
            cpu: "1"
            memory: "1Gi"
          requests:
            cpu: "500m"
            memory: "512Mi"
      volumes:
      - name: config
        configMap:
          name: app-config
---
apiVersion: v1
kind: Service
metadata:
  name: spring-boot-service
  namespace: default
spec:
  selector:
    app: spring-boot-app
  type: ClusterIP
  ports:
  - protocol: TCP
    port: 8080
    targetPort: 8080
---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: spring-boot-hpa
  namespace: default
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: spring-boot-app
  minReplicas: 3
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
```


#### 3.3.2 部署MySQL有状态应用

```yaml
# mysql-statefulset.yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysql-secret
type: Opaque
stringData:
  password: "root123"
---
apiVersion: v1
kind: Service
metadata:
  name: mysql-headless
spec:
  clusterIP: None
  selector:
    app: mysql
  ports:
  - port: 3306
    name: mysql
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
spec:
  serviceName: mysql-headless
  replicas: 3
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:8.0
        ports:
        - containerPort: 3306
          name: mysql
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: password
        volumeMounts:
        - name: data
          mountPath: /var/lib/mysql
        - name: config
          mountPath: /etc/mysql/conf.d
        livenessProbe:
          exec:
            command:
            - mysqladmin
            - ping
            - -h
            - localhost
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          exec:
            command:
            - mysql
            - -h
            - localhost
            - -e
            - SELECT 1
          initialDelaySeconds: 10
          periodSeconds: 5
      volumes:
      - name: config
        configMap:
          name: mysql-config
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: fast-ssd
      resources:
        requests:
          storage: 10Gi
```

## ✨ 最佳实践

### 4.1 资源管理

#### 4.1.1 设置资源请求和限制

```yaml
resources:
  requests:
    cpu: "250m"      # 最小保证
    memory: "256Mi"
  limits:
    cpu: "500m"      # 最大使用
    memory: "512Mi"
```

**建议**：
- 始终设置requests，确保Pod能被调度
- 设置合理的limits，防止资源耗尽
- CPU是可压缩资源，内存是不可压缩资源
- 生产环境建议 limits = 2 * requests

#### 4.1.2 使用命名空间隔离

```bash
# 创建命名空间
kubectl create namespace dev
kubectl create namespace prod

# 在特定命名空间部署
kubectl apply -f app.yaml -n dev

# 设置默认命名空间
kubectl config set-context --current --namespace=dev
```

#### 4.1.3 使用ResourceQuota限制资源

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: dev-quota
  namespace: dev
spec:
  hard:
    requests.cpu: "10"
    requests.memory: "20Gi"
    limits.cpu: "20"
    limits.memory: "40Gi"
    pods: "50"
    services: "10"
    persistentvolumeclaims: "10"
```

### 4.2 健康检查

#### 4.2.1 存活探针（Liveness Probe）

检查容器是否运行，失败则重启容器。

```yaml
livenessProbe:
  httpGet:
    path: /actuator/health/liveness
    port: 8080
  initialDelaySeconds: 30
  periodSeconds: 10
  timeoutSeconds: 5
  failureThreshold: 3
```

#### 4.2.2 就绪探针（Readiness Probe）

检查容器是否准备好接收流量，失败则从Service中移除。

```yaml
readinessProbe:
  httpGet:
    path: /actuator/health/readiness
    port: 8080
  initialDelaySeconds: 20
  periodSeconds: 5
  timeoutSeconds: 3
  failureThreshold: 3
```

#### 4.2.3 启动探针（Startup Probe）

检查容器是否已启动，适用于启动慢的应用。

```yaml
startupProbe:
  httpGet:
    path: /actuator/health
    port: 8080
  initialDelaySeconds: 0
  periodSeconds: 10
  timeoutSeconds: 3
  failureThreshold: 30  # 最多等待300秒
```


### 4.3 滚动更新策略

```yaml
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1        # 最多可以超出期望副本数1个
      maxUnavailable: 0  # 最多不可用副本数为0（零停机）
```

**更新流程**：
1. 创建1个新版本Pod（maxSurge=1）
2. 等待新Pod就绪
3. 删除1个旧版本Pod
4. 重复直到所有Pod更新完成

### 4.4 安全最佳实践

#### 4.4.1 使用非root用户运行容器

```yaml
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    fsGroup: 1000
  containers:
  - name: app
    image: myapp:1.0
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities:
        drop:
        - ALL
```

#### 4.4.2 使用Pod Security Standards

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: myapp
  labels:
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/audit: restricted
    pod-security.kubernetes.io/warn: restricted
```

#### 4.4.3 使用Network Policy限制网络访问

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
  namespace: default
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
```

### 4.5 常见陷阱

#### ⚠️ 陷阱1：未设置资源限制

**问题**：Pod可能耗尽节点资源，影响其他Pod

**解决方案**：始终设置resources.requests和resources.limits

#### ⚠️ 陷阱2：使用latest标签

**问题**：无法追踪版本，难以回滚

**解决方案**：使用具体版本标签
```yaml
# ❌ 不好
image: nginx:latest

# ✅ 好
image: nginx:1.21.6
```

#### ⚠️ 陷阱3：忽略健康检查

**问题**：无法及时发现和处理故障

**解决方案**：配置livenessProbe和readinessProbe

#### ⚠️ 陷阱4：在容器内存储数据

**问题**：容器重启后数据丢失

**解决方案**：使用PersistentVolume持久化数据

#### ⚠️ 陷阱5：单点故障

**问题**：只运行一个副本，服务不可用

**解决方案**：设置合理的副本数（至少2个）
```yaml
spec:
  replicas: 3  # 生产环境建议至少3个
```

## ❓ 常见问题

### Q1: 如何查看Pod的详细日志？

**A**: 使用kubectl logs命令：
```bash
# 查看Pod日志
kubectl logs pod-name

# 查看特定容器的日志
kubectl logs pod-name -c container-name

# 实时查看日志
kubectl logs -f pod-name

# 查看最近100行日志
kubectl logs --tail=100 pod-name

# 查看最近1小时的日志
kubectl logs --since=1h pod-name

# 查看之前容器的日志（容器重启后）
kubectl logs pod-name --previous
```

### Q2: 如何调试Pod启动失败的问题？

**A**: 使用以下方法排查：
```bash
# 查看Pod状态
kubectl get pods

# 查看Pod详细信息
kubectl describe pod pod-name

# 查看Pod事件
kubectl get events --sort-by='.lastTimestamp'

# 查看Pod日志
kubectl logs pod-name

# 进入容器调试
kubectl exec -it pod-name -- /bin/sh

# 查看容器退出状态
kubectl get pod pod-name -o jsonpath='{.status.containerStatuses[0].state}'
```

**常见错误状态**：
- **ImagePullBackOff**: 镜像拉取失败
- **CrashLoopBackOff**: 容器启动后立即崩溃
- **Pending**: 无法调度（资源不足或节点选择器不匹配）
- **Error**: 容器启动失败

### Q3: 如何在不停机的情况下更新应用？

**A**: 使用滚动更新：
```bash
# 更新镜像
kubectl set image deployment/myapp app=myapp:2.0

# 查看更新状态
kubectl rollout status deployment/myapp

# 如果有问题，立即回滚
kubectl rollout undo deployment/myapp
```


### Q4: 如何扩容和缩容应用？

**A**: 使用kubectl scale或HPA：
```bash
# 手动扩容
kubectl scale deployment myapp --replicas=5

# 使用HPA自动扩容
kubectl autoscale deployment myapp --min=3 --max=10 --cpu-percent=70

# 查看HPA状态
kubectl get hpa

# 删除HPA
kubectl delete hpa myapp
```

### Q5: 如何在集群外访问服务？

**A**: 有多种方式：
```bash
# 方式1：使用NodePort
kubectl expose deployment myapp --type=NodePort --port=8080

# 方式2：使用LoadBalancer（需要云提供商支持）
kubectl expose deployment myapp --type=LoadBalancer --port=8080

# 方式3：使用Ingress
kubectl apply -f ingress.yaml

# 方式4：使用kubectl port-forward（临时）
kubectl port-forward service/myapp 8080:8080
```

### Q6: 如何备份和恢复Kubernetes资源？

**A**: 使用kubectl和etcd备份：
```bash
# 导出所有资源
kubectl get all --all-namespaces -o yaml > all-resources.yaml

# 导出特定命名空间的资源
kubectl get all -n myapp -o yaml > myapp-resources.yaml

# 恢复资源
kubectl apply -f all-resources.yaml

# 备份etcd（在master节点）
ETCDCTL_API=3 etcdctl snapshot save snapshot.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key

# 恢复etcd
ETCDCTL_API=3 etcdctl snapshot restore snapshot.db
```

### Q7: 如何限制Pod的网络访问？

**A**: 使用NetworkPolicy：
```yaml
# 只允许来自frontend的流量
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: backend-policy
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - protocol: TCP
      port: 8080
```

### Q8: 如何监控Kubernetes集群？

**A**: 使用Prometheus + Grafana：
```bash
# 安装Prometheus Operator
kubectl apply -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/main/bundle.yaml

# 安装kube-prometheus-stack（包含Prometheus、Grafana、Alertmanager）
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install kube-prometheus prometheus-community/kube-prometheus-stack

# 访问Grafana
kubectl port-forward svc/kube-prometheus-grafana 3000:80

# 默认用户名: admin
# 默认密码: prom-operator
```

## 🔗 相关资源

### 官方资源
- **官方文档**: https://kubernetes.io/docs/
- **GitHub**: https://github.com/kubernetes/kubernetes
- **Kubernetes Blog**: https://kubernetes.io/blog/
- **CNCF**: https://www.cncf.io/

### 推荐学习资源
- **书籍**:
  - 《Kubernetes权威指南》
  - 《Kubernetes in Action》
  - 《深入剖析Kubernetes》

- **在线教程**:
  - Kubernetes官方教程: https://kubernetes.io/docs/tutorials/
  - Katacoda交互式教程: https://www.katacoda.com/courses/kubernetes

- **视频课程**:
  - Kubernetes从入门到实践
  - Kubernetes生产实战

### 相关技术
- **Helm**: Kubernetes包管理器
- **Istio**: 服务网格
- **Prometheus**: 监控系统
- **ArgoCD**: GitOps持续部署
- **Rancher**: Kubernetes管理平台

## 📝 学习检查清单

- [ ] 理解Kubernetes的架构和核心组件
- [ ] 掌握kubectl命令行工具的使用
- [ ] 能够创建和管理Pod、Deployment、Service
- [ ] 理解ConfigMap和Secret的使用场景
- [ ] 掌握持久化存储（PV、PVC、StorageClass）
- [ ] 能够部署和管理有状态应用（StatefulSet）
- [ ] 理解Kubernetes的调度策略（亲和性、污点、容忍）
- [ ] 掌握网络模型和NetworkPolicy
- [ ] 能够配置健康检查（Liveness、Readiness、Startup）
- [ ] 理解滚动更新和回滚机制
- [ ] 掌握HPA自动扩缩容
- [ ] 了解Kubernetes的安全最佳实践
- [ ] 能够排查和解决常见问题

## 🎓 进阶学习路径

1. **Helm包管理**: 学习使用Helm管理Kubernetes应用
2. **服务网格**: 学习Istio进行微服务治理
3. **GitOps**: 学习ArgoCD实现声明式部署
4. **监控告警**: 深入学习Prometheus和Grafana
5. **日志管理**: 学习EFK/ELK日志收集方案
6. **安全加固**: 学习RBAC、Pod Security、Network Policy
7. **多集群管理**: 学习Rancher、KubeFed等多集群管理工具
8. **云原生架构**: 深入学习云原生应用设计模式

---

**最后更新**: 2024-01-04  
**文档版本**: v1.0  
**作者**: @author erik.zhou
