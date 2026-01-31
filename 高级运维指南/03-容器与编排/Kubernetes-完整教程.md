# Kubernetes-完整教程

> @author erik.zhou

## 📋 目录
- [技术概述](#技术概述)
- [K8s架构](#k8s架构)
- [核心概念](#核心概念)
- [工作负载](#工作负载)
- [服务与网络](#服务与网络)
- [存储管理](#存储管理)
- [配置管理](#配置管理)
- [集群管理](#集群管理)
- [故障排查](#故障排查)

## 📚 技术概述

### 基本信息
- **重要程度**：⭐⭐⭐⭐⭐ (P0必学)
- **难度级别**：⭐⭐⭐⭐⭐
- **前置知识**：Docker、Linux、网络基础
- **学习时长**：50-60小时
- **官方文档**：https://kubernetes.io/docs/

### 学习目标
- [ ] 理解K8s架构和核心概念
- [ ] 掌握Pod、Deployment、Service等资源
- [ ] 能够搭建和管理K8s集群
- [ ] 掌握应用部署和运维
- [ ] 能够排查K8s常见问题


---

## 🏗️ K8s架构

### 整体架构 🔥

```
┌─────────────────────────────────────────────────────────────────┐
│                        Control Plane                             │
├─────────────┬─────────────┬─────────────┬─────────────┬─────────┤
│ kube-apiserver│ etcd      │ kube-scheduler│ controller-manager│ │
│             │             │             │ (replication,       │ │
│             │             │             │  node, endpoint...) │ │
└─────────────┴─────────────┴─────────────┴─────────────┴─────────┘
                              │
                              │ API
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                         Worker Nodes                             │
├─────────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐              │
│  │   kubelet   │  │ kube-proxy  │  │  Container  │              │
│  │             │  │             │  │   Runtime   │              │
│  └─────────────┘  └─────────────┘  └─────────────┘              │
│  ┌─────────────────────────────────────────────────┐            │
│  │                     Pods                         │            │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐         │            │
│  │  │Container│  │Container│  │Container│         │            │
│  │  └─────────┘  └─────────┘  └─────────┘         │            │
│  └─────────────────────────────────────────────────┘            │
└─────────────────────────────────────────────────────────────────┘
```

### 核心组件

**Control Plane（控制平面）**
- **kube-apiserver**：API服务器，集群的入口
- **etcd**：分布式键值存储，保存集群状态
- **kube-scheduler**：调度器，决定Pod运行在哪个节点
- **kube-controller-manager**：控制器管理器，维护集群状态

**Node组件**
- **kubelet**：节点代理，管理Pod生命周期
- **kube-proxy**：网络代理，实现Service
- **Container Runtime**：容器运行时（containerd、CRI-O）

---

## 📦 核心概念

### Pod 🔥

Pod是K8s最小的部署单元，包含一个或多个容器。

```yaml
# pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
    env: prod
  annotations:
    description: "Nginx web server"
spec:
  containers:
  - name: nginx
    image: nginx:1.24
    ports:
    - containerPort: 80
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
    livenessProbe:
      httpGet:
        path: /healthz
        port: 80
      initialDelaySeconds: 3
      periodSeconds: 10
    readinessProbe:
      httpGet:
        path: /ready
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 5
    volumeMounts:
    - name: html
      mountPath: /usr/share/nginx/html
  volumes:
  - name: html
    configMap:
      name: nginx-html
  restartPolicy: Always
```

### Namespace

命名空间用于隔离资源。

```bash
# 查看命名空间
kubectl get namespaces
kubectl get ns

# 创建命名空间
kubectl create namespace dev

# 在指定命名空间操作
kubectl get pods -n dev
kubectl apply -f pod.yaml -n dev

# 设置默认命名空间
kubectl config set-context --current --namespace=dev
```

---

## 🚀 工作负载

### Deployment 🔥

Deployment用于管理无状态应用的部署和更新。

```yaml
# deployment.yaml
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
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.24
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "64Mi"
            cpu: "100m"
          limits:
            memory: "128Mi"
            cpu: "200m"
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 10
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
```

```bash
# Deployment操作
kubectl apply -f deployment.yaml
kubectl get deployments
kubectl describe deployment nginx-deployment

# 扩缩容
kubectl scale deployment nginx-deployment --replicas=5

# 更新镜像
kubectl set image deployment/nginx-deployment nginx=nginx:1.25

# 查看更新状态
kubectl rollout status deployment/nginx-deployment

# 查看历史版本
kubectl rollout history deployment/nginx-deployment

# 回滚
kubectl rollout undo deployment/nginx-deployment
kubectl rollout undo deployment/nginx-deployment --to-revision=2

# 暂停/恢复更新
kubectl rollout pause deployment/nginx-deployment
kubectl rollout resume deployment/nginx-deployment
```

### StatefulSet

StatefulSet用于管理有状态应用。

```yaml
# statefulset.yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
spec:
  serviceName: mysql
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
      storageClassName: standard
      resources:
        requests:
          storage: 10Gi
```

### DaemonSet

DaemonSet确保每个节点运行一个Pod副本。

```yaml
# daemonset.yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd
spec:
  selector:
    matchLabels:
      name: fluentd
  template:
    metadata:
      labels:
        name: fluentd
    spec:
      tolerations:
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
      containers:
      - name: fluentd
        image: fluentd:v1.16
        volumeMounts:
        - name: varlog
          mountPath: /var/log
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
```

### Job和CronJob

```yaml
# job.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: backup-job
spec:
  completions: 1
  parallelism: 1
  backoffLimit: 3
  template:
    spec:
      containers:
      - name: backup
        image: backup-tool:v1
        command: ["/bin/sh", "-c", "backup.sh"]
      restartPolicy: Never

---
# cronjob.yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: backup-cronjob
spec:
  schedule: "0 2 * * *"
  concurrencyPolicy: Forbid
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 1
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: backup
            image: backup-tool:v1
            command: ["/bin/sh", "-c", "backup.sh"]
          restartPolicy: OnFailure
```

---

## 🌐 服务与网络

### Service 🔥

Service为Pod提供稳定的访问入口。

```yaml
# ClusterIP Service（默认，集群内部访问）
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: ClusterIP
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80

---
# NodePort Service（节点端口访问）
apiVersion: v1
kind: Service
metadata:
  name: nginx-nodeport
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30080

---
# LoadBalancer Service（云平台负载均衡）
apiVersion: v1
kind: Service
metadata:
  name: nginx-lb
spec:
  type: LoadBalancer
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80

---
# Headless Service（用于StatefulSet）
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
```

### Ingress 🔥

Ingress提供HTTP/HTTPS路由。

```yaml
# ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - app.example.com
    secretName: app-tls-secret
  rules:
  - host: app.example.com
    http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 8080
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-service
            port:
              number: 80
```

---

## 💾 存储管理

### PV和PVC 🔥

```yaml
# PersistentVolume
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nfs-pv
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  storageClassName: nfs
  nfs:
    server: 192.168.1.100
    path: /data/nfs

---
# PersistentVolumeClaim
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nfs-pvc
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: nfs
  resources:
    requests:
      storage: 5Gi

---
# 使用PVC
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
  - name: app
    image: myapp:v1
    volumeMounts:
    - name: data
      mountPath: /data
  volumes:
  - name: data
    persistentVolumeClaim:
      claimName: nfs-pvc
```

### StorageClass

```yaml
# storageclass.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp3
  iopsPerGB: "10"
reclaimPolicy: Delete
allowVolumeExpansion: true
volumeBindingMode: WaitForFirstConsumer
```

---

## ⚙️ 配置管理

### ConfigMap 🔥

```yaml
# configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  # 简单键值对
  database_host: "mysql.default.svc.cluster.local"
  database_port: "3306"
  
  # 配置文件
  app.properties: |
    server.port=8080
    spring.profiles.active=prod
    logging.level.root=INFO

---
# 使用ConfigMap
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
  - name: app
    image: myapp:v1
    env:
    - name: DB_HOST
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: database_host
    envFrom:
    - configMapRef:
        name: app-config
    volumeMounts:
    - name: config
      mountPath: /etc/config
  volumes:
  - name: config
    configMap:
      name: app-config
```

### Secret 🔥

```yaml
# secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
data:
  # base64编码
  username: YWRtaW4=
  password: cGFzc3dvcmQ=

---
# 使用Secret
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
  - name: app
    image: myapp:v1
    env:
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: app-secret
          key: password
    volumeMounts:
    - name: secret
      mountPath: /etc/secret
      readOnly: true
  volumes:
  - name: secret
    secret:
      secretName: app-secret
```

```bash
# 创建Secret
kubectl create secret generic db-secret \
    --from-literal=username=admin \
    --from-literal=password=secret123

# 创建TLS Secret
kubectl create secret tls tls-secret \
    --cert=tls.crt \
    --key=tls.key

# 创建Docker Registry Secret
kubectl create secret docker-registry regcred \
    --docker-server=registry.example.com \
    --docker-username=user \
    --docker-password=password
```

---

## 🔧 集群管理

### kubectl常用命令 🔥

```bash
# 资源查看
kubectl get pods                    # 查看Pod
kubectl get pods -o wide            # 详细信息
kubectl get pods -o yaml            # YAML格式
kubectl get all                     # 所有资源
kubectl get pods -l app=nginx       # 按标签过滤
kubectl get pods --all-namespaces   # 所有命名空间

# 资源详情
kubectl describe pod nginx-pod
kubectl describe node node1

# 创建/更新资源
kubectl apply -f manifest.yaml
kubectl create -f manifest.yaml
kubectl replace -f manifest.yaml

# 删除资源
kubectl delete pod nginx-pod
kubectl delete -f manifest.yaml
kubectl delete pods --all

# 进入容器
kubectl exec -it pod-name -- bash
kubectl exec -it pod-name -c container-name -- bash

# 查看日志
kubectl logs pod-name
kubectl logs pod-name -c container-name
kubectl logs -f pod-name            # 实时跟踪
kubectl logs --previous pod-name    # 上一个容器的日志

# 端口转发
kubectl port-forward pod-name 8080:80
kubectl port-forward svc/service-name 8080:80

# 复制文件
kubectl cp pod-name:/path/file ./local/
kubectl cp ./local/file pod-name:/path/

# 资源编辑
kubectl edit deployment nginx-deployment

# 标签操作
kubectl label pod nginx-pod env=prod
kubectl label pod nginx-pod env-               # 删除标签

# 污点和容忍
kubectl taint nodes node1 key=value:NoSchedule
kubectl taint nodes node1 key-                 # 删除污点

# 节点管理
kubectl cordon node1                # 标记不可调度
kubectl uncordon node1              # 取消标记
kubectl drain node1                 # 驱逐Pod
```

### RBAC权限控制 🔥

```yaml
# ServiceAccount
apiVersion: v1
kind: ServiceAccount
metadata:
  name: app-sa
  namespace: default

---
# Role（命名空间级别）
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: default
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]

---
# RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
- kind: ServiceAccount
  name: app-sa
  namespace: default
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io

---
# ClusterRole（集群级别）
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-admin-role
rules:
- apiGroups: ["*"]
  resources: ["*"]
  verbs: ["*"]

---
# ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cluster-admin-binding
subjects:
- kind: User
  name: admin
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: cluster-admin-role
  apiGroup: rbac.authorization.k8s.io
```

---

## 🔍 故障排查

### 常见问题排查 🔥

```bash
# Pod状态异常
kubectl describe pod pod-name       # 查看事件
kubectl logs pod-name               # 查看日志
kubectl logs pod-name --previous    # 上一个容器日志

# Pod处于Pending状态
# 可能原因：资源不足、调度失败、PVC未绑定
kubectl describe pod pod-name | grep -A 10 Events

# Pod处于CrashLoopBackOff状态
# 可能原因：应用启动失败、配置错误
kubectl logs pod-name
kubectl describe pod pod-name

# ImagePullBackOff
# 可能原因：镜像不存在、认证失败
kubectl describe pod pod-name | grep -A 5 "Failed"

# 网络问题排查
kubectl run debug --image=busybox --rm -it -- sh
# 在debug容器中测试
nslookup kubernetes
wget -qO- http://service-name

# 节点问题
kubectl describe node node-name
kubectl get events --field-selector involvedObject.name=node-name

# 查看集群事件
kubectl get events --sort-by='.lastTimestamp'
kubectl get events -w               # 实时监控
```

### 调试技巧

```bash
# 临时调试容器
kubectl debug pod-name -it --image=busybox

# 复制Pod进行调试
kubectl debug pod-name -it --copy-to=debug-pod --container=app

# 查看资源使用
kubectl top nodes
kubectl top pods

# API调试
kubectl get --raw /api/v1/namespaces/default/pods
kubectl proxy                       # 启动代理
```

---

## 💡 最佳实践

### 资源管理

1. **设置资源限制**：requests和limits
2. **使用命名空间**：隔离不同环境
3. **标签规范**：app、env、version等
4. **健康检查**：liveness和readiness探针

### 安全建议

1. **最小权限原则**：RBAC精细控制
2. **使用Secret**：敏感信息加密存储
3. **网络策略**：限制Pod间通信
4. **镜像安全**：使用可信镜像，定期扫描

### 运维建议

1. **GitOps**：配置文件版本控制
2. **监控告警**：Prometheus + Grafana
3. **日志收集**：EFK/ELK
4. **备份恢复**：etcd定期备份

---

## 📝 学习检查清单

- [ ] 理解K8s架构和核心组件
- [ ] 掌握Pod、Deployment、Service等资源
- [ ] 能够编写K8s YAML配置文件
- [ ] 掌握kubectl常用命令
- [ ] 理解K8s网络和存储
- [ ] 能够配置RBAC权限
- [ ] 能够排查K8s常见问题

---

@author erik.zhou
