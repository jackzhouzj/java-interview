# Jenkins 完整教程

## 📋 目录
- [技术概述](#技术概述)
- [学习目标](#学习目标)
- [基础概念](#基础概念)
- [核心特性](#核心特性)
- [实战应用](#实战应用)
- [最佳实践](#最佳实践)
- [常见问题](#常见问题)
- [相关资源](#相关资源)

## 📚 技术概述

- **版本**: Jenkins 2.440+ (LTS)
- **官方文档**: https://www.jenkins.io/doc/
- **GitHub**: https://github.com/jenkinsci/jenkins
- **学习难度**: ⭐⭐⭐ (1-5星)
- **重要程度**: ⭐⭐⭐⭐⭐ (1-5星)
- **前置知识**: 
  - Java基础
  - Linux基础命令
  - Git版本控制
  - Maven/Gradle构建工具
  - Shell脚本基础

### 什么是Jenkins

Jenkins是领先的开源自动化服务器，使用Java构建。它提供超过2000个插件来支持几乎任何自动化任务，让人类可以专注于机器无法完成的工作。Jenkins是实现持续集成(CI)和持续交付(CD)的核心工具。

### 核心价值

1. **自动化构建**: 自动编译、测试、打包应用程序
2. **持续集成**: 频繁集成代码变更，快速发现问题
3. **持续交付**: 自动化部署流程，快速交付价值
4. **可扩展性**: 丰富的插件生态系统
5. **分布式构建**: 支持主从架构，横向扩展

### 应用场景

- 自动化构建和测试Java/Python/Node.js等项目
- 实现CI/CD流水线，自动部署到各种环境
- 定时任务执行（数据备份、报表生成等）
- 代码质量检查和安全扫描
- 多环境部署管理（开发、测试、生产）

## 🎯 学习目标

- [ ] 理解Jenkins的架构和工作原理
- [ ] 掌握Jenkins的安装和基础配置
- [ ] 熟练使用Freestyle Job和Pipeline
- [ ] 掌握Jenkinsfile的编写
- [ ] 理解分布式构建和Agent管理
- [ ] 掌握常用插件的使用
- [ ] 了解Jenkins的安全配置
- [ ] 掌握CI/CD最佳实践


## 📖 基础概念

### 1.1 Jenkins架构

```
┌─────────────────────────────────────────────────────────┐
│                    Jenkins Master                        │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │  Web UI      │  │  Job管理     │  │  插件系统    │  │
│  └──────────────┘  └──────────────┘  └──────────────┘  │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │  调度器      │  │  构建队列    │  │  安全管理    │  │
│  └──────────────┘  └──────────────┘  └──────────────┘  │
└─────────────────────────────────────────────────────────┘
                          │
        ┌─────────────────┼─────────────────┐
        │                 │                 │
┌───────▼──────┐  ┌───────▼──────┐  ┌───────▼──────┐
│  Agent 1     │  │  Agent 2     │  │  Agent 3     │
│  (Linux)     │  │  (Windows)   │  │  (Docker)    │
│  Executor×2  │  │  Executor×4  │  │  Executor×2  │
└──────────────┘  └──────────────┘  └──────────────┘
```

**核心组件**:
- **Master**: 调度构建、分发任务、监控Agent、提供Web界面
- **Agent/Slave**: 执行Master分配的构建任务
- **Executor**: Agent上的执行槽位，决定并发构建数
- **Job/Project**: 构建任务的配置单元
- **Build**: Job的一次执行实例
- **Workspace**: 构建时的工作目录

### 1.2 核心概念

**Job类型**:
- **Freestyle Project**: 传统的自由风格项目，通过UI配置
- **Pipeline**: 使用代码定义的流水线项目
- **Multi-configuration Project**: 多配置项目，用于矩阵构建
- **Folder**: 组织和管理Job的文件夹
- **Multibranch Pipeline**: 自动为每个分支创建Pipeline

**构建触发器**:
- **SCM轮询**: 定期检查代码仓库变更
- **Webhook**: Git仓库推送时触发
- **定时构建**: 按Cron表达式定时执行
- **上游项目触发**: 依赖项目构建完成后触发
- **手动触发**: 用户手动点击构建

**构建参数**:
- **String Parameter**: 字符串参数
- **Boolean Parameter**: 布尔参数
- **Choice Parameter**: 选择参数
- **File Parameter**: 文件参数
- **Password Parameter**: 密码参数

### 1.3 Pipeline概念

**Pipeline类型**:
1. **Declarative Pipeline**: 声明式，结构化，推荐使用
2. **Scripted Pipeline**: 脚本式，更灵活，基于Groovy

**Pipeline结构**:
```groovy
pipeline {
    agent any              // 在哪里执行
    stages {              // 阶段定义
        stage('Build') {  // 单个阶段
            steps {       // 步骤
                // 执行命令
            }
        }
    }
}
```


## 🔥 核心特性

### 2.1 Pipeline即代码 🔥

Pipeline即代码是Jenkins最核心的特性，通过Jenkinsfile将构建流程代码化。

**Declarative Pipeline示例**:
```groovy
pipeline {
    agent any
    
    environment {
        MAVEN_HOME = '/usr/local/maven'
        JAVA_HOME = '/usr/local/jdk11'
    }
    
    parameters {
        string(name: 'BRANCH', defaultValue: 'main', description: 'Git分支')
        choice(name: 'ENVIRONMENT', choices: ['dev', 'test', 'prod'], description: '部署环境')
        booleanParam(name: 'SKIP_TESTS', defaultValue: false, description: '跳过测试')
    }
    
    stages {
        stage('Checkout') {
            steps {
                git branch: "${params.BRANCH}", 
                    url: 'https://github.com/myorg/myrepo.git',
                    credentialsId: 'github-credentials'
            }
        }
        
        stage('Build') {
            steps {
                sh """
                    mvn clean package \
                    -DskipTests=${params.SKIP_TESTS} \
                    -Dmaven.test.failure.ignore=true
                """
            }
        }
        
        stage('Test') {
            when {
                expression { params.SKIP_TESTS == false }
            }
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }
        
        stage('Deploy') {
            steps {
                script {
                    if (params.ENVIRONMENT == 'prod') {
                        input message: '确认部署到生产环境?', ok: '部署'
                    }
                }
                sh """
                    scp target/*.jar user@${params.ENVIRONMENT}-server:/opt/app/
                    ssh user@${params.ENVIRONMENT}-server 'systemctl restart myapp'
                """
            }
        }
    }
    
    post {
        success {
            echo '构建成功！'
            emailext subject: "构建成功: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                     body: "构建成功，查看详情: ${env.BUILD_URL}",
                     to: 'team@example.com'
        }
        failure {
            echo '构建失败！'
            emailext subject: "构建失败: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                     body: "构建失败，查看详情: ${env.BUILD_URL}",
                     to: 'team@example.com'
        }
    }
}
```

**Scripted Pipeline示例**:
```groovy
node {
    try {
        stage('Checkout') {
            checkout scm
        }
        
        stage('Build') {
            sh 'mvn clean package'
        }
        
        stage('Test') {
            sh 'mvn test'
        }
        
        stage('Deploy') {
            if (env.BRANCH_NAME == 'main') {
                sh 'kubectl apply -f k8s/'
            }
        }
        
        currentBuild.result = 'SUCCESS'
    } catch (Exception e) {
        currentBuild.result = 'FAILURE'
        throw e
    } finally {
        // 清理工作
        cleanWs()
    }
}
```


### 2.2 分布式构建 🔥 (⚠️ 难点)

分布式构建是Jenkins扩展能力的关键，通过Master-Agent架构实现负载分担。

**Agent配置方式**:

1. **SSH Agent** (推荐):
```java
import hudson.model.*;
import hudson.slaves.*;
import jenkins.model.Jenkins;

public class AgentManager {
    public void createSSHAgent() throws IOException {
        Jenkins jenkins = Jenkins.get();
        
        // 创建SSH连接的Agent
        DumbSlave sshAgent = new DumbSlave(
            "linux-agent-01",           // Agent名称
            "/opt/jenkins",             // 远程工作目录
            new SSHLauncher(
                "192.168.1.100",        // 主机地址
                22,                     // SSH端口
                "jenkins-ssh-key",      // 凭据ID
                null,                   // JVM选项
                null,                   // Java路径
                null,                   // 启动前缀命令
                null                    // 启动后缀命令
            )
        );
        
        sshAgent.setNumExecutors(4);                    // 执行器数量
        sshAgent.setLabelString("linux maven java11");  // 标签
        sshAgent.setMode(Node.Mode.NORMAL);             // 使用模式
        
        jenkins.addNode(sshAgent);
        jenkins.save();
    }
}
```

2. **JNLP Agent** (适用于Docker):
```java
public void createJNLPAgent() throws IOException {
    Jenkins jenkins = Jenkins.get();
    
    DumbSlave agent = new DumbSlave(
        "docker-agent-01",
        "/home/jenkins/agent",
        new JNLPLauncher(true)  // Java Web Start启动器
    );
    
    agent.setNumExecutors(2);
    agent.setLabelString("docker linux x64");
    agent.setMode(Node.Mode.NORMAL);
    agent.setRetentionStrategy(new RetentionStrategy.Always());
    
    // 添加环境变量
    EnvironmentVariablesNodeProperty envProp = 
        new EnvironmentVariablesNodeProperty(
            new EnvironmentVariablesNodeProperty.Entry(
                "DOCKER_HOST", "unix:///var/run/docker.sock"
            )
        );
    agent.getNodeProperties().add(envProp);
    
    jenkins.addNode(agent);
    jenkins.save();
}
```

**在Pipeline中使用Agent**:
```groovy
pipeline {
    agent none  // 不使用默认agent
    
    stages {
        stage('Build on Linux') {
            agent {
                label 'linux && maven'  // 使用标签选择agent
            }
            steps {
                sh 'mvn clean package'
            }
        }
        
        stage('Build on Docker') {
            agent {
                docker {
                    image 'maven:3.8-jdk-11'
                    label 'docker'
                    args '-v /root/.m2:/root/.m2'
                }
            }
            steps {
                sh 'mvn clean package'
            }
        }
        
        stage('Build on Kubernetes') {
            agent {
                kubernetes {
                    yaml '''
                        apiVersion: v1
                        kind: Pod
                        spec:
                          containers:
                          - name: maven
                            image: maven:3.8-jdk-11
                            command: ['cat']
                            tty: true
                    '''
                }
            }
            steps {
                container('maven') {
                    sh 'mvn clean package'
                }
            }
        }
    }
}
```

**Agent管理**:
```java
public void manageAgents() {
    Jenkins jenkins = Jenkins.get();
    
    // 列出所有节点
    List<Node> nodes = jenkins.getNodes();
    for (Node node : nodes) {
        Computer computer = node.toComputer();
        
        if (computer != null) {
            String name = node.getNodeName();
            boolean online = computer.isOnline();
            boolean idle = computer.isIdle();
            int executors = computer.getExecutors().size();
            
            System.out.println(String.format(
                "Node: %s, Online: %s, Idle: %s, Executors: %d",
                name, online, idle, executors
            ));
            
            // 临时下线节点（维护）
            if (online) {
                computer.setTemporarilyOffline(true, 
                    new OfflineCause.UserCause(User.current(), "维护中"));
            }
            
            // 恢复在线
            computer.setTemporarilyOffline(false, null);
        }
    }
    
    // 删除节点
    Node nodeToRemove = jenkins.getNode("old-agent");
    if (nodeToRemove != null) {
        jenkins.removeNode(nodeToRemove);
    }
}
```


### 2.3 插件系统 🔥

Jenkins拥有超过2000个插件，极大扩展了其功能。

**常用插件分类**:

1. **源码管理插件**:
   - Git Plugin: Git集成
   - GitHub Plugin: GitHub集成
   - GitLab Plugin: GitLab集成
   - Subversion Plugin: SVN集成

2. **构建工具插件**:
   - Maven Integration: Maven项目构建
   - Gradle Plugin: Gradle项目构建
   - NodeJS Plugin: Node.js项目构建
   - Docker Plugin: Docker镜像构建

3. **部署插件**:
   - Deploy to container: 部署到Tomcat等容器
   - Kubernetes Plugin: 部署到K8s
   - SSH Plugin: SSH远程部署
   - Ansible Plugin: Ansible自动化部署

4. **通知插件**:
   - Email Extension: 邮件通知
   - Slack Notification: Slack通知
   - DingTalk Plugin: 钉钉通知
   - WeChat Plugin: 企业微信通知

5. **代码质量插件**:
   - SonarQube Scanner: 代码质量分析
   - Checkstyle Plugin: 代码风格检查
   - FindBugs Plugin: Bug检测
   - JaCoCo Plugin: 代码覆盖率

6. **安全插件**:
   - Role-based Authorization: 基于角色的权限控制
   - LDAP Plugin: LDAP认证
   - Active Directory Plugin: AD域认证
   - OWASP Dependency-Check: 依赖安全检查

**插件使用示例**:
```groovy
pipeline {
    agent any
    
    stages {
        stage('Code Quality') {
            steps {
                // SonarQube代码扫描
                withSonarQubeEnv('SonarQube') {
                    sh 'mvn sonar:sonar'
                }
                
                // 等待质量门检查
                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        
        stage('Security Scan') {
            steps {
                // OWASP依赖检查
                dependencyCheck additionalArguments: '--format HTML --format XML',
                                odcInstallation: 'OWASP-DC'
                
                // 发布报告
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
        stage('Deploy to K8s') {
            steps {
                // Kubernetes部署
                kubernetesDeploy(
                    configs: 'k8s/*.yaml',
                    kubeconfigId: 'k8s-config',
                    enableConfigSubstitution: true
                )
            }
        }
        
        stage('Notify') {
            steps {
                // 钉钉通知
                dingtalk(
                    robot: 'jenkins-robot',
                    type: 'MARKDOWN',
                    title: "构建通知: ${env.JOB_NAME}",
                    text: [
                        "### 构建信息",
                        "- 项目: ${env.JOB_NAME}",
                        "- 构建号: ${env.BUILD_NUMBER}",
                        "- 状态: ${currentBuild.result}",
                        "- 详情: [查看](${env.BUILD_URL})"
                    ]
                )
            }
        }
    }
}
```

### 2.4 凭据管理

Jenkins提供安全的凭据管理机制，避免在代码中硬编码敏感信息。

**凭据类型**:
- **Username with password**: 用户名密码
- **SSH Username with private key**: SSH私钥
- **Secret text**: 密钥文本
- **Secret file**: 密钥文件
- **Certificate**: 证书

**在Pipeline中使用凭据**:
```groovy
pipeline {
    agent any
    
    stages {
        stage('Use Credentials') {
            steps {
                // 使用用户名密码
                withCredentials([
                    usernamePassword(
                        credentialsId: 'docker-hub',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh '''
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker push myimage:latest
                    '''
                }
                
                // 使用SSH密钥
                withCredentials([
                    sshUserPrivateKey(
                        credentialsId: 'deploy-key',
                        keyFileVariable: 'SSH_KEY',
                        usernameVariable: 'SSH_USER'
                    )
                ]) {
                    sh '''
                        ssh -i $SSH_KEY $SSH_USER@server 'bash deploy.sh'
                    '''
                }
                
                // 使用密钥文本
                withCredentials([
                    string(credentialsId: 'api-token', variable: 'API_TOKEN')
                ]) {
                    sh '''
                        curl -H "Authorization: Bearer $API_TOKEN" https://api.example.com
                    '''
                }
            }
        }
    }
}
```


### 2.5 Multibranch Pipeline (⚠️ 难点)

Multibranch Pipeline自动为Git仓库的每个分支创建独立的Pipeline。

**配置Multibranch Pipeline**:
```groovy
// Jenkinsfile在代码仓库根目录
pipeline {
    agent any
    
    stages {
        stage('Branch Info') {
            steps {
                script {
                    echo "当前分支: ${env.BRANCH_NAME}"
                    echo "变更作者: ${env.CHANGE_AUTHOR}"
                    echo "变更标题: ${env.CHANGE_TITLE}"
                }
            }
        }
        
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
        
        stage('Deploy') {
            when {
                anyOf {
                    branch 'main'
                    branch 'develop'
                }
            }
            steps {
                script {
                    def env = (env.BRANCH_NAME == 'main') ? 'prod' : 'dev'
                    sh "kubectl apply -f k8s/${env}/"
                }
            }
        }
    }
}
```

**分支策略配置**:
```groovy
// 在Jenkins UI中配置或使用Job DSL
multibranchPipelineJob('my-multibranch-pipeline') {
    branchSources {
        git {
            id('my-repo')
            remote('https://github.com/myorg/myrepo.git')
            credentialsId('github-credentials')
            
            // 分支发现策略
            traits {
                // 发现所有分支
                gitBranchDiscovery()
                
                // 发现PR
                gitHubPullRequestDiscovery {
                    strategyId(1) // 合并后的代码
                }
                
                // 忽略特定分支
                headWildcardFilter {
                    includes('main develop feature/* release/*')
                    excludes('hotfix/*')
                }
            }
        }
    }
    
    // 触发器
    triggers {
        periodic(1) // 每分钟扫描一次
    }
    
    // 孤儿项目策略
    orphanedItemStrategy {
        discardOldItems {
            numToKeep(10)
        }
    }
}
```

### 2.6 共享库 (⚠️ 难点)

共享库允许在多个Pipeline之间复用代码。

**共享库结构**:
```
jenkins-shared-library/
├── vars/                    # 全局变量（可直接在Pipeline中调用）
│   ├── buildJava.groovy
│   ├── deployK8s.groovy
│   └── notifyDingTalk.groovy
├── src/                     # 类库（需要import）
│   └── org/
│       └── mycompany/
│           └── jenkins/
│               ├── Builder.groovy
│               └── Deployer.groovy
└── resources/               # 资源文件
    └── templates/
        └── Dockerfile
```

**vars/buildJava.groovy**:
```groovy
// 全局变量，可直接调用
def call(Map config) {
    pipeline {
        agent any
        
        stages {
            stage('Checkout') {
                steps {
                    git url: config.gitUrl, branch: config.branch
                }
            }
            
            stage('Build') {
                steps {
                    sh """
                        mvn clean package \
                        -DskipTests=${config.skipTests ?: false}
                    """
                }
            }
            
            stage('Test') {
                when {
                    expression { !config.skipTests }
                }
                steps {
                    sh 'mvn test'
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }
    }
}
```

**使用共享库**:
```groovy
@Library('jenkins-shared-library@main') _

// 直接调用vars中的方法
buildJava(
    gitUrl: 'https://github.com/myorg/myrepo.git',
    branch: 'main',
    skipTests: false
)

// 或者在Pipeline中使用
pipeline {
    agent any
    
    stages {
        stage('Build') {
            steps {
                script {
                    // 使用共享库中的类
                    def builder = new org.mycompany.jenkins.Builder()
                    builder.build('maven')
                }
            }
        }
        
        stage('Deploy') {
            steps {
                // 使用vars中的方法
                deployK8s(
                    namespace: 'production',
                    deployment: 'myapp'
                )
            }
        }
        
        stage('Notify') {
            steps {
                notifyDingTalk(
                    robot: 'jenkins-robot',
                    message: "部署完成: ${env.JOB_NAME}"
                )
            }
        }
    }
}
```


## 💻 实战应用

### 3.1 环境搭建

**Docker方式安装（推荐）**:
```bash
# 拉取Jenkins镜像
docker pull jenkins/jenkins:lts

# 创建数据卷
docker volume create jenkins_home

# 启动Jenkins容器
docker run -d \
  --name jenkins \
  -p 8080:8080 \
  -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  jenkins/jenkins:lts

# 查看初始管理员密码
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

**Linux系统安装**:
```bash
# CentOS/RHEL
sudo wget -O /etc/yum.repos.d/jenkins.repo \
    https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
sudo yum install jenkins java-11-openjdk-devel
sudo systemctl start jenkins
sudo systemctl enable jenkins

# Ubuntu/Debian
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > \
    /etc/apt/sources.list.d/jenkins.list'
sudo apt-get update
sudo apt-get install jenkins
sudo systemctl start jenkins
sudo systemctl enable jenkins
```

**初始化配置**:
1. 访问 http://localhost:8080
2. 输入初始管理员密码
3. 安装推荐插件或自选插件
4. 创建管理员用户
5. 配置Jenkins URL

### 3.2 快速开始 - Freestyle Job

**创建简单的Maven构建任务**:

1. 新建任务 → Freestyle project
2. 源码管理 → Git
   - Repository URL: https://github.com/myorg/myrepo.git
   - Credentials: 添加GitHub凭据
   - Branch: */main

3. 构建触发器
   - ☑ GitHub hook trigger for GITScm polling

4. 构建环境
   - ☑ Delete workspace before build starts

5. 构建步骤 → Invoke top-level Maven targets
   - Goals: clean package
   - Properties: -DskipTests=false

6. 构建后操作
   - Archive the artifacts: target/*.jar
   - Publish JUnit test result report: **/target/surefire-reports/*.xml
   - E-mail Notification: team@example.com

### 3.3 进阶案例 - 完整CI/CD Pipeline

**Spring Boot应用的完整CI/CD流程**:

```groovy
@Library('jenkins-shared-library') _

pipeline {
    agent any
    
    environment {
        // 环境变量
        DOCKER_REGISTRY = 'registry.example.com'
        APP_NAME = 'spring-boot-app'
        K8S_NAMESPACE = 'production'
        SONAR_HOST = 'http://sonarqube:9000'
    }
    
    parameters {
        choice(name: 'DEPLOY_ENV', choices: ['dev', 'test', 'prod'], description: '部署环境')
        booleanParam(name: 'SKIP_TESTS', defaultValue: false, description: '跳过测试')
        booleanParam(name: 'DEPLOY_ENABLED', defaultValue: true, description: '是否部署')
    }
    
    options {
        // 保留最近10次构建
        buildDiscarder(logRotator(numToKeepStr: '10'))
        // 禁止并发构建
        disableConcurrentBuilds()
        // 超时时间
        timeout(time: 30, unit: 'MINUTES')
        // 添加时间戳
        timestamps()
    }
    
    stages {
        stage('Checkout') {
            steps {
                script {
                    echo "检出代码: ${env.GIT_BRANCH}"
                }
                checkout scm
            }
        }
        
        stage('Build') {
            steps {
                script {
                    echo "开始构建..."
                }
                sh '''
                    mvn clean package \
                    -DskipTests=${SKIP_TESTS} \
                    -Dmaven.test.failure.ignore=true
                '''
            }
        }
        
        stage('Unit Test') {
            when {
                expression { params.SKIP_TESTS == false }
            }
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                    jacoco(
                        execPattern: '**/target/jacoco.exec',
                        classPattern: '**/target/classes',
                        sourcePattern: '**/src/main/java'
                    )
                }
            }
        }
        
        stage('Code Quality') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh '''
                        mvn sonar:sonar \
                        -Dsonar.projectKey=${APP_NAME} \
                        -Dsonar.host.url=${SONAR_HOST}
                    '''
                }
                
                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        
        stage('Security Scan') {
            parallel {
                stage('Dependency Check') {
                    steps {
                        dependencyCheck additionalArguments: '--format HTML --format XML',
                                        odcInstallation: 'OWASP-DC'
                        dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
                    }
                }
                
                stage('Docker Image Scan') {
                    steps {
                        sh '''
                            trivy image --severity HIGH,CRITICAL \
                            ${DOCKER_REGISTRY}/${APP_NAME}:${BUILD_NUMBER}
                        '''
                    }
                }
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    docker.withRegistry("https://${DOCKER_REGISTRY}", 'docker-registry-credentials') {
                        def customImage = docker.build("${APP_NAME}:${BUILD_NUMBER}")
                        customImage.push()
                        customImage.push('latest')
                    }
                }
            }
        }
        
        stage('Deploy') {
            when {
                expression { params.DEPLOY_ENABLED == true }
            }
            steps {
                script {
                    if (params.DEPLOY_ENV == 'prod') {
                        input message: '确认部署到生产环境?', ok: '部署', submitter: 'admin'
                    }
                    
                    // 使用Kubernetes插件部署
                    kubernetesDeploy(
                        configs: "k8s/${params.DEPLOY_ENV}/*.yaml",
                        kubeconfigId: 'k8s-config',
                        enableConfigSubstitution: true
                    )
                    
                    // 等待部署完成
                    sh """
                        kubectl rollout status deployment/${APP_NAME} \
                        -n ${K8S_NAMESPACE} \
                        --timeout=5m
                    """
                }
            }
        }
        
        stage('Smoke Test') {
            when {
                expression { params.DEPLOY_ENABLED == true }
            }
            steps {
                script {
                    def serviceUrl = sh(
                        script: "kubectl get svc ${APP_NAME} -n ${K8S_NAMESPACE} -o jsonpath='{.status.loadBalancer.ingress[0].ip}'",
                        returnStdout: true
                    ).trim()
                    
                    sh """
                        curl -f http://${serviceUrl}/actuator/health || exit 1
                    """
                }
            }
        }
    }
    
    post {
        success {
            script {
                notifyDingTalk(
                    robot: 'jenkins-robot',
                    type: 'MARKDOWN',
                    title: "✅ 构建成功",
                    text: [
                        "### 构建信息",
                        "- 项目: ${env.JOB_NAME}",
                        "- 构建号: ${env.BUILD_NUMBER}",
                        "- 分支: ${env.GIT_BRANCH}",
                        "- 环境: ${params.DEPLOY_ENV}",
                        "- 状态: 成功 ✅",
                        "- 详情: [查看](${env.BUILD_URL})"
                    ]
                )
            }
        }
        
        failure {
            script {
                notifyDingTalk(
                    robot: 'jenkins-robot',
                    type: 'MARKDOWN',
                    title: "❌ 构建失败",
                    text: [
                        "### 构建信息",
                        "- 项目: ${env.JOB_NAME}",
                        "- 构建号: ${env.BUILD_NUMBER}",
                        "- 分支: ${env.GIT_BRANCH}",
                        "- 状态: 失败 ❌",
                        "- 详情: [查看](${env.BUILD_URL}console)"
                    ]
                )
            }
        }
        
        always {
            // 清理工作空间
            cleanWs()
        }
    }
}
```


### 3.4 REST API使用

Jenkins提供完整的REST API用于自动化管理。

**触发构建**:
```bash
# 触发无参数构建
curl -X POST -u username:token \
  http://jenkins.example.com/job/my-project/build

# 触发带参数构建
curl -X POST -u username:token \
  --data "BRANCH=main&ENVIRONMENT=prod" \
  http://jenkins.example.com/job/my-project/buildWithParameters

# 使用JSON参数
curl -X POST -u username:token \
  --data-urlencode 'json={"parameter": [{"name":"BRANCH", "value":"main"}]}' \
  http://jenkins.example.com/job/my-project/buildWithParameters
```

**查询构建状态**:
```bash
# 获取最后一次构建状态
curl -s -u username:token \
  "http://jenkins.example.com/job/my-project/lastBuild/api/json?tree=building,result"

# 获取特定构建的详细信息
curl -s -u username:token \
  "http://jenkins.example.com/job/my-project/123/api/json"

# 获取构建日志
curl -s -u username:token \
  "http://jenkins.example.com/job/my-project/123/consoleText"
```

**停止构建**:
```bash
curl -X POST -u username:token \
  http://jenkins.example.com/job/my-project/123/stop
```

**CLI工具使用**:
```bash
# 下载CLI客户端
wget http://jenkins.example.com/jnlpJars/jenkins-cli.jar

# 构建任务
java -jar jenkins-cli.jar -s http://jenkins.example.com/ \
  -auth username:token build my-project -s -v

# 带参数构建
java -jar jenkins-cli.jar -s http://jenkins.example.com/ \
  -auth username:token build my-project -p BRANCH=main -p ENV=prod

# 创建任务
cat job-config.xml | java -jar jenkins-cli.jar -s http://jenkins.example.com/ \
  -auth username:token create-job new-project

# 列出所有任务
java -jar jenkins-cli.jar -s http://jenkins.example.com/ \
  -auth username:token list-jobs
```


## ✨ 最佳实践

### 4.1 Pipeline最佳实践

**1. 使用Declarative Pipeline**:
```groovy
// ✅ 推荐：声明式Pipeline，结构清晰
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
    }
}

// ❌ 不推荐：脚本式Pipeline，除非需要复杂逻辑
node {
    stage('Build') {
        sh 'mvn clean package'
    }
}
```

**2. 使用共享库复用代码**:
```groovy
// ✅ 推荐：使用共享库
@Library('jenkins-shared-library') _
buildJava(gitUrl: 'https://github.com/myorg/myrepo.git')

// ❌ 不推荐：在每个Pipeline中重复代码
pipeline {
    // 大量重复的构建逻辑...
}
```

**3. 合理使用并行构建**:
```groovy
// ✅ 推荐：独立任务并行执行
stage('Tests') {
    parallel {
        stage('Unit Tests') {
            steps { sh 'mvn test' }
        }
        stage('Integration Tests') {
            steps { sh 'mvn verify' }
        }
        stage('Security Scan') {
            steps { sh 'trivy scan' }
        }
    }
}
```

**4. 使用when条件控制执行**:
```groovy
// ✅ 推荐：使用when条件
stage('Deploy to Prod') {
    when {
        allOf {
            branch 'main'
            expression { currentBuild.result == 'SUCCESS' }
        }
    }
    steps {
        sh 'kubectl apply -f k8s/prod/'
    }
}
```

**5. 合理设置超时**:
```groovy
// ✅ 推荐：设置合理的超时时间
options {
    timeout(time: 30, unit: 'MINUTES')
}

stage('Long Running Task') {
    options {
        timeout(time: 10, unit: 'MINUTES')
    }
    steps {
        sh 'long-running-command'
    }
}
```

### 4.2 性能优化

**1. 使用增量构建**:
```groovy
// Maven增量构建
sh 'mvn clean package -T 4 -Dmaven.test.skip=true'

// Gradle增量构建
sh './gradlew build --build-cache --parallel'
```

**2. 缓存依赖**:
```groovy
// Docker构建缓存
docker.build("myapp:${BUILD_NUMBER}", "--cache-from myapp:latest .")

// Maven本地仓库缓存
agent {
    docker {
        image 'maven:3.8-jdk-11'
        args '-v /root/.m2:/root/.m2'
    }
}
```

**3. 合理分配Agent资源**:
```groovy
// ✅ 推荐：根据任务类型选择Agent
stage('Build') {
    agent {
        label 'linux && maven && high-memory'
    }
    steps {
        sh 'mvn clean package'
    }
}

// ❌ 不推荐：所有任务都在Master上执行
agent { label 'master' }
```

**4. 清理工作空间**:
```groovy
// ✅ 推荐：构建后清理
post {
    always {
        cleanWs()
    }
}

// 或者构建前清理
options {
    skipDefaultCheckout()
}
stages {
    stage('Checkout') {
        steps {
            cleanWs()
            checkout scm
        }
    }
}
```

### 4.3 安全最佳实践

**1. 使用凭据管理**:
```groovy
// ✅ 推荐：使用凭据管理
withCredentials([
    usernamePassword(credentialsId: 'docker-hub', 
                     usernameVariable: 'USER', 
                     passwordVariable: 'PASS')
]) {
    sh 'echo $PASS | docker login -u $USER --password-stdin'
}

// ❌ 不推荐：硬编码密码
sh 'docker login -u myuser -p mypassword'
```

**2. 限制Pipeline权限**:
```groovy
// 使用@NonCPS注解限制Groovy代码执行
@NonCPS
def parseJson(String json) {
    return new groovy.json.JsonSlurper().parseText(json)
}

// 使用Script Security插件审批脚本
```

**3. 审计和日志**:
```groovy
// 记录关键操作
stage('Deploy') {
    steps {
        script {
            echo "部署操作由 ${env.BUILD_USER} 触发"
            echo "部署环境: ${params.ENVIRONMENT}"
            echo "部署时间: ${new Date()}"
        }
        sh 'kubectl apply -f k8s/'
    }
}
```

**4. 使用RBAC权限控制**:
- 安装Role-based Authorization Strategy插件
- 为不同团队创建不同角色
- 最小权限原则：只授予必要的权限
- 定期审查权限配置

### 4.4 常见陷阱

**⚠️ 陷阱1：在Pipeline中使用不可序列化的对象**
```groovy
// ❌ 错误：不可序列化的对象
def myObject = new SomeNonSerializableClass()
stage('Use Object') {
    steps {
        script {
            myObject.doSomething()  // 可能导致序列化错误
        }
    }
}

// ✅ 正确：使用@NonCPS或避免跨stage使用
@NonCPS
def processData(data) {
    def obj = new SomeNonSerializableClass()
    return obj.process(data)
}
```

**⚠️ 陷阱2：忘记清理工作空间**
```groovy
// ❌ 错误：不清理工作空间，磁盘空间耗尽
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
    }
}

// ✅ 正确：定期清理
post {
    always {
        cleanWs()
    }
}
```

**⚠️ 陷阱3：Agent标签配置错误**
```groovy
// ❌ 错误：标签不存在或拼写错误
agent {
    label 'linuxx'  // 拼写错误
}

// ✅ 正确：使用正确的标签
agent {
    label 'linux && docker'
}
```

**⚠️ 陷阱4：忽略构建失败**
```groovy
// ❌ 错误：忽略错误继续执行
sh 'mvn test || true'

// ✅ 正确：正确处理错误
script {
    try {
        sh 'mvn test'
    } catch (Exception e) {
        currentBuild.result = 'UNSTABLE'
        echo "测试失败: ${e.message}"
    }
}
```


## ❓ 常见问题

### Q1: Jenkins构建速度慢怎么优化？

**A**: 多方面优化策略：

1. **使用分布式构建**:
   - 添加多个Agent节点
   - 合理分配构建任务到不同Agent
   - 使用标签管理Agent

2. **启用构建缓存**:
   ```groovy
   // Maven本地仓库缓存
   agent {
       docker {
           image 'maven:3.8-jdk-11'
           args '-v $HOME/.m2:/root/.m2'
       }
   }
   
   // Docker镜像缓存
   sh 'docker build --cache-from myapp:latest -t myapp:${BUILD_NUMBER} .'
   ```

3. **并行构建**:
   ```groovy
   stage('Tests') {
       parallel {
           stage('Unit') { steps { sh 'mvn test' } }
           stage('Integration') { steps { sh 'mvn verify' } }
       }
   }
   ```

4. **增量构建**:
   ```bash
   # Maven多线程构建
   mvn clean package -T 4
   
   # Gradle增量构建
   ./gradlew build --build-cache
   ```

### Q2: 如何实现Jenkins的高可用？

**A**: 高可用方案：

1. **主备模式**:
   - 使用共享存储（NFS/EFS）存储JENKINS_HOME
   - 配置主备Jenkins实例
   - 使用负载均衡器切换

2. **使用Jenkins Configuration as Code (JCasC)**:
   ```yaml
   # jenkins.yaml
   jenkins:
     systemMessage: "Jenkins configured automatically by JCasC"
     numExecutors: 2
     securityRealm:
       local:
         allowsSignup: false
         users:
           - id: "admin"
             password: "${ADMIN_PASSWORD}"
     authorizationStrategy:
       globalMatrix:
         permissions:
           - "Overall/Administer:admin"
   ```

3. **定期备份**:
   ```bash
   # 备份脚本
   #!/bin/bash
   JENKINS_HOME=/var/lib/jenkins
   BACKUP_DIR=/backup/jenkins
   DATE=$(date +%Y%m%d_%H%M%S)
   
   # 停止Jenkins
   systemctl stop jenkins
   
   # 备份
   tar -czf ${BACKUP_DIR}/jenkins_${DATE}.tar.gz ${JENKINS_HOME}
   
   # 启动Jenkins
   systemctl start jenkins
   
   # 保留最近7天的备份
   find ${BACKUP_DIR} -name "jenkins_*.tar.gz" -mtime +7 -delete
   ```

### Q3: Pipeline中如何处理敏感信息？

**A**: 使用凭据管理：

```groovy
pipeline {
    agent any
    
    stages {
        stage('Deploy') {
            steps {
                // 方式1：使用withCredentials
                withCredentials([
                    string(credentialsId: 'api-key', variable: 'API_KEY'),
                    usernamePassword(
                        credentialsId: 'db-credentials',
                        usernameVariable: 'DB_USER',
                        passwordVariable: 'DB_PASS'
                    )
                ]) {
                    sh '''
                        curl -H "Authorization: Bearer $API_KEY" https://api.example.com
                        mysql -u $DB_USER -p$DB_PASS < schema.sql
                    '''
                }
                
                // 方式2：使用environment
                environment {
                    API_TOKEN = credentials('api-token')
                }
            }
        }
    }
}
```

### Q4: 如何调试Pipeline脚本？

**A**: 调试技巧：

1. **使用echo输出调试信息**:
   ```groovy
   script {
       echo "当前分支: ${env.BRANCH_NAME}"
       echo "构建号: ${env.BUILD_NUMBER}"
       echo "参数值: ${params.ENVIRONMENT}"
   }
   ```

2. **使用Replay功能**:
   - 在构建历史中点击"Replay"
   - 修改Pipeline脚本
   - 重新运行测试

3. **使用Blue Ocean查看可视化流程**:
   - 安装Blue Ocean插件
   - 更直观地查看Pipeline执行流程

4. **使用try-catch捕获错误**:
   ```groovy
   script {
       try {
           sh 'some-command'
       } catch (Exception e) {
           echo "错误: ${e.toString()}"
           echo "错误消息: ${e.getMessage()}"
           currentBuild.result = 'FAILURE'
       }
   }
   ```

### Q5: Jenkins磁盘空间不足怎么办？

**A**: 清理策略：

1. **配置构建保留策略**:
   ```groovy
   options {
       buildDiscarder(logRotator(
           numToKeepStr: '10',        // 保留最近10次构建
           artifactNumToKeepStr: '5'  // 保留最近5次构建的制品
       ))
   }
   ```

2. **定期清理工作空间**:
   ```groovy
   post {
       always {
           cleanWs()
       }
   }
   ```

3. **清理Docker镜像**:
   ```bash
   # 清理未使用的镜像
   docker system prune -a -f
   
   # 定期清理（添加到crontab）
   0 2 * * * docker system prune -a -f
   ```

4. **清理旧的构建记录**:
   ```bash
   # 清理30天前的构建
   find /var/lib/jenkins/jobs/*/builds/ -type d -mtime +30 -exec rm -rf {} \;
   ```

### Q6: 如何实现多环境部署？

**A**: 多环境部署方案：

```groovy
pipeline {
    agent any
    
    parameters {
        choice(name: 'ENVIRONMENT', 
               choices: ['dev', 'test', 'staging', 'prod'], 
               description: '部署环境')
    }
    
    stages {
        stage('Deploy') {
            steps {
                script {
                    // 根据环境加载不同配置
                    def config = [
                        'dev': [
                            'namespace': 'dev',
                            'replicas': 1,
                            'resources': 'small'
                        ],
                        'test': [
                            'namespace': 'test',
                            'replicas': 2,
                            'resources': 'medium'
                        ],
                        'prod': [
                            'namespace': 'production',
                            'replicas': 5,
                            'resources': 'large'
                        ]
                    ]
                    
                    def envConfig = config[params.ENVIRONMENT]
                    
                    // 生产环境需要审批
                    if (params.ENVIRONMENT == 'prod') {
                        input message: '确认部署到生产环境?', 
                              ok: '部署',
                              submitter: 'admin,ops'
                    }
                    
                    // 部署
                    sh """
                        kubectl apply -f k8s/${params.ENVIRONMENT}/ \
                        --namespace=${envConfig.namespace}
                        
                        kubectl scale deployment myapp \
                        --replicas=${envConfig.replicas} \
                        --namespace=${envConfig.namespace}
                    """
                }
            }
        }
    }
}
```

### Q7: Pipeline执行超时怎么办？

**A**: 超时处理：

```groovy
pipeline {
    agent any
    
    options {
        // 全局超时
        timeout(time: 1, unit: 'HOURS')
    }
    
    stages {
        stage('Long Running Task') {
            options {
                // 单个stage超时
                timeout(time: 30, unit: 'MINUTES')
            }
            steps {
                script {
                    // 单个步骤超时
                    timeout(time: 10, unit: 'MINUTES') {
                        sh 'long-running-command'
                    }
                }
            }
        }
    }
}
```

## 🔗 相关资源

### 官方资源
- **官方文档**: https://www.jenkins.io/doc/
- **Pipeline语法**: https://www.jenkins.io/doc/book/pipeline/syntax/
- **插件中心**: https://plugins.jenkins.io/
- **GitHub仓库**: https://github.com/jenkinsci/jenkins
- **社区论坛**: https://community.jenkins.io/

### 推荐插件
- **Blue Ocean**: 现代化的Pipeline可视化界面
- **Pipeline**: Pipeline核心插件
- **Git Plugin**: Git集成
- **Docker Plugin**: Docker集成
- **Kubernetes Plugin**: Kubernetes集成
- **SonarQube Scanner**: 代码质量分析
- **Email Extension**: 邮件通知
- **Role-based Authorization**: 基于角色的权限控制

### 学习资源
- **Jenkins官方教程**: https://www.jenkins.io/doc/tutorials/
- **Pipeline示例**: https://www.jenkins.io/doc/pipeline/examples/
- **最佳实践**: https://www.jenkins.io/doc/book/pipeline/pipeline-best-practices/

### 相关技术
- [GitLab CI/CD完整教程](GitLab-CI-CD-完整教程.md)
- [Docker完整教程](../02-容器化/Docker-完整教程.md)
- [Kubernetes完整教程](../02-容器化/Kubernetes-完整教程.md)
- [Maven完整教程](../../08-开发工具链/01-构建工具/Maven-完整教程.md)
- [Git完整教程](../../08-开发工具链/02-版本控制/Git-完整教程.md)

## 📝 学习检查清单

- [ ] 理解Jenkins的Master-Agent架构
- [ ] 掌握Freestyle Job的创建和配置
- [ ] 熟练编写Declarative Pipeline
- [ ] 理解Pipeline的各个组成部分（agent、stages、steps、post）
- [ ] 掌握常用插件的使用
- [ ] 理解分布式构建的配置和使用
- [ ] 掌握凭据管理
- [ ] 了解共享库的使用
- [ ] 掌握Multibranch Pipeline
- [ ] 理解Jenkins的安全配置
- [ ] 掌握CI/CD最佳实践
- [ ] 能够排查常见问题

---

**文档版本**: v1.0  
**最后更新**: 2024-01  
**文档来源**: Jenkins官方文档 + Context7  
**@author** erik.zhou
