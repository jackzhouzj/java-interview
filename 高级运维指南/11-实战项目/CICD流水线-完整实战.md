# CI/CD流水线-完整实战

> @author erik.zhou

## 📋 目录
- [项目概述](#项目概述)
- [架构设计](#架构设计)
- [环境准备](#环境准备)
- [Jenkins配置](#jenkins配置)
- [流水线实现](#流水线实现)
- [最佳实践](#最佳实践)

## 📚 项目概述

### 项目目标
构建一套完整的CI/CD流水线，实现：
- 代码提交自动触发构建
- 自动化测试
- Docker镜像构建和推送
- 自动部署到Kubernetes

### 技术栈
- **代码仓库**：GitLab
- **CI工具**：Jenkins
- **容器**：Docker
- **编排**：Kubernetes
- **镜像仓库**：Harbor


---

## 🏗️ 架构设计

### 流水线架构 🔥

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   GitLab    │───►│   Jenkins   │───►│   Harbor    │
│  (代码仓库) │    │  (CI/CD)    │    │ (镜像仓库)  │
└─────────────┘    └─────────────┘    └─────────────┘
                          │
                          ▼
                   ┌─────────────┐
                   │ Kubernetes  │
                   │  (部署)     │
                   └─────────────┘
```

### 流水线阶段

```
代码提交 → 代码检出 → 代码扫描 → 单元测试 → 构建镜像 → 推送镜像 → 部署应用 → 通知
```

---

## 🔧 环境准备

### Docker Compose部署Jenkins

```yaml
# docker-compose.yml
version: '3.8'

services:
  jenkins:
    image: jenkins/jenkins:lts
    container_name: jenkins
    user: root
    ports:
      - "8080:8080"
      - "50000:50000"
    volumes:
      - jenkins_data:/var/jenkins_home
      - /var/run/docker.sock:/var/run/docker.sock
      - /usr/bin/docker:/usr/bin/docker
    environment:
      - JAVA_OPTS=-Djenkins.install.runSetupWizard=false
    restart: always

volumes:
  jenkins_data:
```

### 安装必要插件

```
必装插件：
- Git
- Pipeline
- Docker Pipeline
- Kubernetes
- Blue Ocean
- GitLab
- SonarQube Scanner
- Credentials Binding
```

---

## ⚙️ Jenkins配置

### 凭据配置

```groovy
// 需要配置的凭据
// 1. gitlab-credentials - GitLab访问凭据
// 2. harbor-credentials - Harbor仓库凭据
// 3. kubeconfig - Kubernetes配置文件
// 4. sonar-token - SonarQube Token
```

### 全局工具配置

```groovy
// Jenkins系统配置 -> 全局工具配置
// 1. JDK
// 2. Maven
// 3. Docker
// 4. kubectl
```

---

## 📝 流水线实现

### Jenkinsfile 🔥

```groovy
// Jenkinsfile
pipeline {
    agent any
    
    environment {
        // 项目信息
        APP_NAME = 'myapp'
        GIT_REPO = 'https://gitlab.example.com/team/myapp.git'
        
        // Harbor配置
        HARBOR_URL = 'harbor.example.com'
        HARBOR_PROJECT = 'myproject'
        IMAGE_NAME = "${HARBOR_URL}/${HARBOR_PROJECT}/${APP_NAME}"
        
        // Kubernetes配置
        K8S_NAMESPACE = 'production'
        
        // 凭据
        HARBOR_CREDENTIALS = credentials('harbor-credentials')
        KUBECONFIG = credentials('kubeconfig')
    }
    
    options {
        timeout(time: 30, unit: 'MINUTES')
        disableConcurrentBuilds()
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }
    
    stages {
        stage('Checkout') {
            steps {
                echo "=== 代码检出 ==="
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[
                        url: "${GIT_REPO}",
                        credentialsId: 'gitlab-credentials'
                    ]]
                ])
                script {
                    env.GIT_COMMIT_SHORT = sh(
                        script: 'git rev-parse --short HEAD',
                        returnStdout: true
                    ).trim()
                    env.IMAGE_TAG = "${BUILD_NUMBER}-${GIT_COMMIT_SHORT}"
                }
            }
        }
        
        stage('Code Analysis') {
            steps {
                echo "=== 代码扫描 ==="
                withSonarQubeEnv('SonarQube') {
                    sh '''
                        mvn sonar:sonar \
                            -Dsonar.projectKey=${APP_NAME} \
                            -Dsonar.projectName=${APP_NAME}
                    '''
                }
            }
        }
        
        stage('Unit Test') {
            steps {
                echo "=== 单元测试 ==="
                sh 'mvn test'
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }
        
        stage('Build') {
            steps {
                echo "=== 构建应用 ==="
                sh 'mvn clean package -DskipTests'
            }
        }
        
        stage('Build Image') {
            steps {
                echo "=== 构建镜像 ==="
                sh """
                    docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                    docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${IMAGE_NAME}:latest
                """
            }
        }
        
        stage('Push Image') {
            steps {
                echo "=== 推送镜像 ==="
                sh """
                    echo ${HARBOR_CREDENTIALS_PSW} | docker login ${HARBOR_URL} -u ${HARBOR_CREDENTIALS_USR} --password-stdin
                    docker push ${IMAGE_NAME}:${IMAGE_TAG}
                    docker push ${IMAGE_NAME}:latest
                """
            }
        }
        
        stage('Deploy to K8s') {
            steps {
                echo "=== 部署到Kubernetes ==="
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    sh """
                        # 更新deployment镜像
                        kubectl --kubeconfig=${KUBECONFIG} -n ${K8S_NAMESPACE} \
                            set image deployment/${APP_NAME} ${APP_NAME}=${IMAGE_NAME}:${IMAGE_TAG}
                        
                        # 等待部署完成
                        kubectl --kubeconfig=${KUBECONFIG} -n ${K8S_NAMESPACE} \
                            rollout status deployment/${APP_NAME} --timeout=300s
                    """
                }
            }
        }
    }
    
    post {
        success {
            echo "=== 构建成功 ==="
            // 发送成功通知
            script {
                sendNotification('SUCCESS')
            }
        }
        failure {
            echo "=== 构建失败 ==="
            // 发送失败通知
            script {
                sendNotification('FAILURE')
            }
        }
        always {
            // 清理工作空间
            cleanWs()
            // 清理Docker镜像
            sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG} || true"
        }
    }
}

// 通知函数
def sendNotification(String status) {
    def color = status == 'SUCCESS' ? 'good' : 'danger'
    def message = """
        *${status}*: Job `${env.JOB_NAME}` build `${env.BUILD_NUMBER}`
        Branch: `${env.GIT_BRANCH}`
        Commit: `${env.GIT_COMMIT_SHORT}`
        Duration: ${currentBuild.durationString}
        <${env.BUILD_URL}|View Build>
    """
    
    // 发送到钉钉/企业微信/Slack
    // slackSend(color: color, message: message)
}
```

### Dockerfile

```dockerfile
# Dockerfile
FROM openjdk:17-slim

LABEL maintainer="erik.zhou"

WORKDIR /app

COPY target/*.jar app.jar

EXPOSE 8080

HEALTHCHECK --interval=30s --timeout=3s \
    CMD curl -f http://localhost:8080/actuator/health || exit 1

ENTRYPOINT ["java", "-jar", "app.jar"]
```

### Kubernetes部署文件

```yaml
# k8s/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
  namespace: production
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: harbor.example.com/myproject/myapp:latest
        ports:
        - containerPort: 8080
        resources:
          requests:
            memory: "256Mi"
            cpu: "100m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /actuator/health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /actuator/health
            port: 8080
          initialDelaySeconds: 10
          periodSeconds: 5
        env:
        - name: SPRING_PROFILES_ACTIVE
          value: "prod"
---
apiVersion: v1
kind: Service
metadata:
  name: myapp
  namespace: production
spec:
  selector:
    app: myapp
  ports:
  - port: 80
    targetPort: 8080
  type: ClusterIP
```

---

## 🔄 多环境流水线

### 多分支流水线

```groovy
// Jenkinsfile - 多环境
pipeline {
    agent any
    
    environment {
        APP_NAME = 'myapp'
    }
    
    stages {
        stage('Determine Environment') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'main') {
                        env.DEPLOY_ENV = 'production'
                        env.K8S_NAMESPACE = 'production'
                    } else if (env.BRANCH_NAME == 'develop') {
                        env.DEPLOY_ENV = 'staging'
                        env.K8S_NAMESPACE = 'staging'
                    } else if (env.BRANCH_NAME.startsWith('feature/')) {
                        env.DEPLOY_ENV = 'development'
                        env.K8S_NAMESPACE = 'development'
                    }
                }
            }
        }
        
        stage('Build & Test') {
            steps {
                sh 'mvn clean test package'
            }
        }
        
        stage('Deploy to Dev') {
            when {
                branch 'feature/*'
            }
            steps {
                echo "Deploying to Development"
                // 部署到开发环境
            }
        }
        
        stage('Deploy to Staging') {
            when {
                branch 'develop'
            }
            steps {
                echo "Deploying to Staging"
                // 部署到测试环境
            }
        }
        
        stage('Deploy to Production') {
            when {
                branch 'main'
            }
            steps {
                // 生产环境需要人工确认
                input message: '确认部署到生产环境?', ok: '确认部署'
                echo "Deploying to Production"
                // 部署到生产环境
            }
        }
    }
}
```

---

## 💡 最佳实践

### 流水线设计原则

1. **快速反馈**：尽早发现问题
2. **自动化**：减少人工干预
3. **可重复**：每次构建结果一致
4. **可追溯**：记录所有变更
5. **安全**：凭据加密存储

### 安全建议

1. **凭据管理**：使用Jenkins Credentials
2. **镜像扫描**：集成Trivy扫描
3. **代码扫描**：集成SonarQube
4. **权限控制**：最小权限原则

### 优化建议

1. **并行执行**：加快构建速度
2. **缓存利用**：Maven/npm缓存
3. **增量构建**：只构建变更部分
4. **资源限制**：避免资源争抢

---

## 📝 学习检查清单

- [ ] 能够搭建Jenkins环境
- [ ] 能够编写Jenkinsfile
- [ ] 能够配置多环境流水线
- [ ] 能够集成代码扫描
- [ ] 能够部署到Kubernetes
- [ ] 理解CI/CD最佳实践

---

@author erik.zhou
