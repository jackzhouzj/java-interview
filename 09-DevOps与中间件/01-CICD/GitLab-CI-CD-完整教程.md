# GitLab CI/CD 完整教程

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

- **版本**: GitLab 16.x+
- **官方文档**: https://docs.gitlab.com/ee/ci/
- **学习难度**: ⭐⭐⭐ (1-5星)
- **重要程度**: ⭐⭐⭐⭐⭐ (1-5星)
- **前置知识**: 
  - Git版本控制基础
  - YAML语法
  - Linux基础命令
  - Docker基础（可选）
  - Kubernetes基础（可选）

### 什么是GitLab CI/CD

GitLab CI/CD是GitLab内置的持续集成和持续交付工具，通过`.gitlab-ci.yml`配置文件定义自动化流水线。它与GitLab代码仓库深度集成，提供从代码提交到生产部署的完整自动化流程。

### 核心价值

1. **原生集成**: 与GitLab无缝集成，无需额外安装
2. **配置即代码**: 通过YAML文件定义Pipeline
3. **并行执行**: 支持Job并行执行，提高构建速度
4. **多环境部署**: 支持多环境管理和部署
5. **容器化构建**: 原生支持Docker和Kubernetes

### 应用场景

- 自动化构建和测试Java/Python/Node.js等项目
- 实现CI/CD流水线，自动部署到各种环境
- 代码质量检查和安全扫描
- 容器镜像构建和推送
- 多环境部署管理（开发、测试、生产）

## 🎯 学习目标

- [ ] 理解GitLab CI/CD的架构和工作原理
- [ ] 掌握.gitlab-ci.yml的编写
- [ ] 熟练使用stages、jobs、scripts
- [ ] 理解GitLab Runner的配置和使用
- [ ] 掌握变量和缓存的使用
- [ ] 了解环境和部署管理
- [ ] 掌握CI/CD最佳实践


## 📖 基础概念

### 1.1 GitLab CI/CD架构

```
┌─────────────────────────────────────────────────────────┐
│                    GitLab Server                         │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │  Git仓库     │  │  CI/CD引擎   │  │  Web界面     │  │
│  └──────────────┘  └──────────────┘  └──────────────┘  │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │  Pipeline    │  │  Job队列     │  │  制品仓库    │  │
│  └──────────────┘  └──────────────┘  └──────────────┘  │
└─────────────────────────────────────────────────────────┘
                          │
        ┌─────────────────┼─────────────────┐
        │                 │                 │
┌───────▼──────┐  ┌───────▼──────┐  ┌───────▼──────┐
│  Runner 1    │  │  Runner 2    │  │  Runner 3    │
│  (Docker)    │  │  (Shell)     │  │  (K8s)       │
│  Executor×2  │  │  Executor×1  │  │  Executor×5  │
└──────────────┘  └──────────────┘  └──────────────┘
```

**核心组件**:
- **GitLab Server**: 管理代码仓库和CI/CD配置
- **Pipeline**: 完整的CI/CD流程
- **Stage**: Pipeline中的阶段（如build、test、deploy）
- **Job**: Stage中的具体任务
- **Runner**: 执行Job的服务
- **Executor**: Runner的执行方式（Docker、Shell、Kubernetes等）

### 1.2 核心概念

**Pipeline结构**:
```yaml
# .gitlab-ci.yml
stages:           # 定义阶段
  - build
  - test
  - deploy

build-job:        # Job名称
  stage: build    # 所属阶段
  script:         # 执行脚本
    - echo "Building..."
    - mvn clean package
```

**Job执行流程**:
1. Runner从GitLab Server获取Job
2. 准备执行环境（拉取Docker镜像或准备Shell环境）
3. 克隆代码仓库
4. 执行before_script
5. 执行script
6. 执行after_script
7. 上传制品和缓存
8. 清理环境

**Runner类型**:
- **Shared Runner**: 所有项目共享
- **Group Runner**: 组内项目共享
- **Specific Runner**: 特定项目专用

**Executor类型**:
- **Docker**: 在Docker容器中执行（推荐）
- **Shell**: 直接在Runner主机上执行
- **Kubernetes**: 在K8s Pod中执行
- **Docker Machine**: 动态创建Docker主机
- **SSH**: 通过SSH连接到远程主机执行

### 1.3 .gitlab-ci.yml基础语法

**基本结构**:
```yaml
# 全局配置
image: maven:3.8-jdk-11          # 默认Docker镜像
variables:                        # 全局变量
  MAVEN_OPTS: "-Dmaven.repo.local=.m2/repository"

# 定义阶段
stages:
  - build
  - test
  - deploy

# 定义Job
build-job:
  stage: build
  script:
    - mvn clean package
  artifacts:                      # 制品
    paths:
      - target/*.jar
    expire_in: 1 week
  cache:                          # 缓存
    paths:
      - .m2/repository
  only:                           # 执行条件
    - main
    - develop
  tags:                           # Runner标签
    - docker
```


## 🔥 核心特性

### 2.1 Pipeline配置 🔥

**完整的Pipeline示例**:
```yaml
# .gitlab-ci.yml

# 全局配置
image: maven:3.8-jdk-11

variables:
  MAVEN_OPTS: "-Dmaven.repo.local=$CI_PROJECT_DIR/.m2/repository"
  DOCKER_REGISTRY: "registry.example.com"
  APP_NAME: "spring-boot-app"

# 定义阶段
stages:
  - build
  - test
  - quality
  - package
  - deploy

# 全局before_script
before_script:
  - echo "开始执行Job: $CI_JOB_NAME"
  - echo "当前分支: $CI_COMMIT_REF_NAME"

# 构建阶段
build:
  stage: build
  script:
    - mvn clean compile
  artifacts:
    paths:
      - target/classes
    expire_in: 1 hour
  cache:
    key: ${CI_COMMIT_REF_SLUG}
    paths:
      - .m2/repository
  only:
    - branches
  tags:
    - docker

# 单元测试
unit-test:
  stage: test
  script:
    - mvn test
  artifacts:
    when: always
    reports:
      junit: target/surefire-reports/TEST-*.xml
    paths:
      - target/surefire-reports
  coverage: '/Total.*?([0-9]{1,3})%/'
  only:
    - branches
  tags:
    - docker

# 集成测试
integration-test:
  stage: test
  services:
    - mysql:8.0
    - redis:7
  variables:
    MYSQL_ROOT_PASSWORD: "root"
    MYSQL_DATABASE: "testdb"
  script:
    - mvn verify -P integration-test
  artifacts:
    reports:
      junit: target/failsafe-reports/TEST-*.xml
  only:
    - main
    - develop
  tags:
    - docker

# 代码质量检查
code-quality:
  stage: quality
  image: sonarsource/sonar-scanner-cli:latest
  variables:
    SONAR_HOST_URL: "http://sonarqube:9000"
    SONAR_TOKEN: $SONAR_TOKEN
  script:
    - sonar-scanner
      -Dsonar.projectKey=$CI_PROJECT_NAME
      -Dsonar.sources=src/main/java
      -Dsonar.java.binaries=target/classes
  allow_failure: true
  only:
    - main
    - develop
  tags:
    - docker

# 安全扫描
security-scan:
  stage: quality
  image: aquasec/trivy:latest
  script:
    - trivy fs --severity HIGH,CRITICAL .
  allow_failure: true
  only:
    - main
  tags:
    - docker

# 打包
package:
  stage: package
  script:
    - mvn package -DskipTests
  artifacts:
    paths:
      - target/*.jar
    expire_in: 1 week
  only:
    - main
    - develop
  tags:
    - docker

# Docker镜像构建
docker-build:
  stage: package
  image: docker:latest
  services:
    - docker:dind
  variables:
    DOCKER_TLS_CERTDIR: "/certs"
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $DOCKER_REGISTRY
  script:
    - docker build -t $DOCKER_REGISTRY/$APP_NAME:$CI_COMMIT_SHORT_SHA .
    - docker build -t $DOCKER_REGISTRY/$APP_NAME:latest .
    - docker push $DOCKER_REGISTRY/$APP_NAME:$CI_COMMIT_SHORT_SHA
    - docker push $DOCKER_REGISTRY/$APP_NAME:latest
  only:
    - main
  tags:
    - docker

# 部署到开发环境
deploy-dev:
  stage: deploy
  environment:
    name: development
    url: https://dev.example.com
  script:
    - kubectl config use-context dev-cluster
    - kubectl set image deployment/$APP_NAME 
      $APP_NAME=$DOCKER_REGISTRY/$APP_NAME:$CI_COMMIT_SHORT_SHA
      -n development
    - kubectl rollout status deployment/$APP_NAME -n development
  only:
    - develop
  tags:
    - kubernetes

# 部署到生产环境
deploy-prod:
  stage: deploy
  environment:
    name: production
    url: https://prod.example.com
  script:
    - kubectl config use-context prod-cluster
    - kubectl set image deployment/$APP_NAME 
      $APP_NAME=$DOCKER_REGISTRY/$APP_NAME:$CI_COMMIT_SHORT_SHA
      -n production
    - kubectl rollout status deployment/$APP_NAME -n production
  when: manual                    # 手动触发
  only:
    - main
  tags:
    - kubernetes

# 全局after_script
after_script:
  - echo "Job执行完成: $CI_JOB_NAME"
  - echo "状态: $CI_JOB_STATUS"
```


### 2.2 变量和密钥管理 🔥

**变量类型**:
1. **预定义变量**: GitLab自动提供
2. **自定义变量**: 在.gitlab-ci.yml中定义
3. **项目变量**: 在项目设置中定义
4. **组变量**: 在组设置中定义

**常用预定义变量**:
```yaml
variables:
  # CI/CD相关
  CI_COMMIT_SHA: "完整的commit SHA"
  CI_COMMIT_SHORT_SHA: "短commit SHA"
  CI_COMMIT_REF_NAME: "分支或标签名"
  CI_COMMIT_REF_SLUG: "分支名（小写，适合URL）"
  CI_COMMIT_MESSAGE: "commit消息"
  CI_COMMIT_AUTHOR: "commit作者"
  
  # 项目相关
  CI_PROJECT_ID: "项目ID"
  CI_PROJECT_NAME: "项目名称"
  CI_PROJECT_PATH: "项目路径"
  CI_PROJECT_DIR: "项目目录"
  
  # Pipeline相关
  CI_PIPELINE_ID: "Pipeline ID"
  CI_PIPELINE_IID: "Pipeline内部ID"
  CI_PIPELINE_SOURCE: "触发源"
  
  # Job相关
  CI_JOB_ID: "Job ID"
  CI_JOB_NAME: "Job名称"
  CI_JOB_STAGE: "Job所属阶段"
  CI_JOB_STATUS: "Job状态"
  
  # Runner相关
  CI_RUNNER_ID: "Runner ID"
  CI_RUNNER_TAGS: "Runner标签"
```

**使用变量**:
```yaml
variables:
  # 定义全局变量
  DEPLOY_ENV: "production"
  APP_VERSION: "1.0.0"

build-job:
  variables:
    # Job级别变量
    BUILD_TYPE: "release"
  script:
    # 使用变量
    - echo "部署环境: $DEPLOY_ENV"
    - echo "应用版本: $APP_VERSION"
    - echo "构建类型: $BUILD_TYPE"
    - echo "Commit SHA: $CI_COMMIT_SHORT_SHA"
    
    # 变量替换
    - sed -i "s/VERSION/$APP_VERSION/g" config.yaml
```

**保护变量和密钥**:
```yaml
# 在GitLab UI中设置保护变量
# Settings > CI/CD > Variables

# 使用密钥变量
deploy-job:
  script:
    # 使用密钥登录
    - echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin
    
    # 使用API Token
    - curl -H "Authorization: Bearer $API_TOKEN" https://api.example.com
    
    # 使用SSH密钥
    - echo "$SSH_PRIVATE_KEY" > ~/.ssh/id_rsa
    - chmod 600 ~/.ssh/id_rsa
    - ssh user@server 'bash deploy.sh'
```

**变量优先级**（从高到低）:
1. 触发器变量
2. 计划Pipeline变量
3. Job变量
4. 全局变量
5. 项目变量
6. 组变量
7. 实例变量

### 2.3 缓存和制品 (⚠️ 难点)

**缓存（Cache）**:
用于加速构建，在不同Pipeline之间共享。

```yaml
# 全局缓存配置
cache:
  key: ${CI_COMMIT_REF_SLUG}      # 缓存键
  paths:
    - .m2/repository               # Maven本地仓库
    - node_modules/                # Node.js依赖
  policy: pull-push                # 拉取并推送

build-job:
  cache:
    key: ${CI_COMMIT_REF_SLUG}-build
    paths:
      - target/
    policy: push                   # 只推送

test-job:
  cache:
    key: ${CI_COMMIT_REF_SLUG}-build
    paths:
      - target/
    policy: pull                   # 只拉取
```

**制品（Artifacts）**:
用于在同一Pipeline的不同Job之间传递文件。

```yaml
build-job:
  script:
    - mvn clean package
  artifacts:
    name: "$CI_JOB_NAME-$CI_COMMIT_REF_NAME"
    paths:
      - target/*.jar               # 制品路径
      - target/classes/
    exclude:
      - target/**/*.log            # 排除文件
    expire_in: 1 week              # 过期时间
    when: on_success               # 成功时保存
    reports:                       # 报告
      junit: target/surefire-reports/TEST-*.xml
      coverage_report:
        coverage_format: cobertura
        path: target/site/cobertura/coverage.xml

test-job:
  dependencies:
    - build-job                    # 依赖build-job的制品
  script:
    - java -jar target/*.jar --test
```

**缓存 vs 制品对比**:

| 特性 | 缓存（Cache） | 制品（Artifacts） |
|------|--------------|------------------|
| 用途 | 加速构建 | 传递文件 |
| 范围 | 跨Pipeline | 同一Pipeline |
| 可靠性 | 不保证存在 | 保证存在 |
| 下载 | 自动 | 需要dependencies |
| 存储位置 | Runner本地 | GitLab Server |
| 典型用例 | 依赖包 | 构建产物 |


### 2.4 条件执行和规则 🔥 (⚠️ 难点)

**使用only/except（旧语法）**:
```yaml
# 只在特定分支执行
deploy-job:
  script:
    - echo "Deploying..."
  only:
    - main
    - develop
  except:
    - tags

# 只在特定变更时执行
test-job:
  script:
    - npm test
  only:
    changes:
      - "src/**/*.js"
      - "package.json"
```

**使用rules（推荐）**:
```yaml
# 基于条件执行
deploy-job:
  script:
    - echo "Deploying..."
  rules:
    # 主分支且非MR时执行
    - if: '$CI_COMMIT_BRANCH == "main" && $CI_PIPELINE_SOURCE != "merge_request_event"'
      when: always
    # 开发分支手动执行
    - if: '$CI_COMMIT_BRANCH == "develop"'
      when: manual
    # 其他情况不执行
    - when: never

# 基于文件变更
test-frontend:
  script:
    - npm test
  rules:
    - changes:
        - "frontend/**/*"
        - "package.json"
      when: always
    - when: never

# 复杂规则组合
build-job:
  script:
    - mvn clean package
  rules:
    # 主分支或标签
    - if: '$CI_COMMIT_BRANCH == "main" || $CI_COMMIT_TAG'
      when: always
    # MR且有Java文件变更
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
      changes:
        - "src/**/*.java"
      when: always
    # 其他情况不执行
    - when: never
```

**workflow规则（控制整个Pipeline）**:
```yaml
workflow:
  rules:
    # 不为MR创建Pipeline
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
      when: never
    # 为分支创建Pipeline
    - if: '$CI_COMMIT_BRANCH'
      when: always
    # 为标签创建Pipeline
    - if: '$CI_COMMIT_TAG'
      when: always

stages:
  - build
  - test
  - deploy

# 后续Job定义...
```

### 2.5 并行和矩阵构建

**并行执行**:
```yaml
# 并行执行多个相同Job
test-job:
  script:
    - npm test -- --shard=$CI_NODE_INDEX/$CI_NODE_TOTAL
  parallel: 5                      # 创建5个并行Job

# 并行矩阵构建
test-matrix:
  script:
    - mvn test -Djava.version=$JAVA_VERSION
  parallel:
    matrix:
      - JAVA_VERSION: ["11", "17", "21"]
        OS: ["ubuntu", "alpine"]
  # 生成6个Job: 3个Java版本 × 2个OS
```

**需要等待的Job**:
```yaml
build-job:
  stage: build
  script:
    - mvn clean package

test-job-1:
  stage: test
  needs: [build-job]               # 等待build-job完成
  script:
    - mvn test

test-job-2:
  stage: test
  needs: [build-job]               # 等待build-job完成
  script:
    - mvn verify

deploy-job:
  stage: deploy
  needs:                           # 等待多个Job
    - build-job
    - test-job-1
    - test-job-2
  script:
    - kubectl apply -f k8s/
```

### 2.6 环境和部署

**定义环境**:
```yaml
deploy-staging:
  stage: deploy
  environment:
    name: staging
    url: https://staging.example.com
    on_stop: stop-staging          # 停止环境的Job
    auto_stop_in: 1 week           # 自动停止时间
  script:
    - kubectl apply -f k8s/staging/
  only:
    - develop

stop-staging:
  stage: deploy
  environment:
    name: staging
    action: stop
  script:
    - kubectl delete -f k8s/staging/
  when: manual
  only:
    - develop

deploy-production:
  stage: deploy
  environment:
    name: production
    url: https://prod.example.com
    deployment_tier: production    # 部署层级
  script:
    - kubectl apply -f k8s/production/
  when: manual                     # 手动触发
  only:
    - main
```

**动态环境**:
```yaml
# 为每个分支创建独立环境
deploy-review:
  stage: deploy
  environment:
    name: review/$CI_COMMIT_REF_SLUG
    url: https://$CI_COMMIT_REF_SLUG.review.example.com
    on_stop: stop-review
    auto_stop_in: 3 days
  script:
    - kubectl create namespace review-$CI_COMMIT_REF_SLUG || true
    - kubectl apply -f k8s/ -n review-$CI_COMMIT_REF_SLUG
  only:
    - branches
  except:
    - main
    - develop

stop-review:
  stage: deploy
  environment:
    name: review/$CI_COMMIT_REF_SLUG
    action: stop
  script:
    - kubectl delete namespace review-$CI_COMMIT_REF_SLUG
  when: manual
  only:
    - branches
  except:
    - main
    - develop
```


### 2.7 GitLab Runner配置 (⚠️ 难点)

**安装GitLab Runner**:
```bash
# Linux安装
curl -L "https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.deb.sh" | sudo bash
sudo apt-get install gitlab-runner

# 或使用Docker
docker run -d --name gitlab-runner --restart always \
  -v /srv/gitlab-runner/config:/etc/gitlab-runner \
  -v /var/run/docker.sock:/var/run/docker.sock \
  gitlab/gitlab-runner:latest
```

**注册Runner**:
```bash
# 交互式注册
sudo gitlab-runner register

# 非交互式注册
sudo gitlab-runner register \
  --non-interactive \
  --url "https://gitlab.example.com/" \
  --registration-token "PROJECT_REGISTRATION_TOKEN" \
  --executor "docker" \
  --docker-image "alpine:latest" \
  --description "docker-runner" \
  --tag-list "docker,linux" \
  --run-untagged="true" \
  --locked="false" \
  --access-level="not_protected"
```

**Runner配置文件**:
```toml
# /etc/gitlab-runner/config.toml

concurrent = 4                     # 并发Job数量
check_interval = 0

[session_server]
  session_timeout = 1800

[[runners]]
  name = "docker-runner"
  url = "https://gitlab.example.com/"
  token = "RUNNER_TOKEN"
  executor = "docker"
  
  [runners.custom_build_dir]
  
  [runners.cache]
    [runners.cache.s3]
    [runners.cache.gcs]
    [runners.cache.azure]
  
  [runners.docker]
    tls_verify = false
    image = "alpine:latest"
    privileged = true              # 允许特权模式（Docker in Docker）
    disable_entrypoint_overwrite = false
    oom_kill_disable = false
    disable_cache = false
    volumes = ["/cache", "/var/run/docker.sock:/var/run/docker.sock"]
    shm_size = 0
    pull_policy = "if-not-present"
```

**Kubernetes Executor配置**:
```toml
[[runners]]
  name = "kubernetes-runner"
  url = "https://gitlab.example.com/"
  token = "RUNNER_TOKEN"
  executor = "kubernetes"
  
  [runners.kubernetes]
    host = "https://kubernetes.example.com"
    namespace = "gitlab-runner"
    privileged = true
    cpu_limit = "2"
    memory_limit = "4Gi"
    service_cpu_limit = "1"
    service_memory_limit = "2Gi"
    helper_cpu_limit = "500m"
    helper_memory_limit = "512Mi"
    poll_interval = 5
    poll_timeout = 360
    
    [runners.kubernetes.pod_labels]
      "app" = "gitlab-runner"
      "environment" = "production"
```

**Runner管理命令**:
```bash
# 查看Runner状态
sudo gitlab-runner status

# 启动Runner
sudo gitlab-runner start

# 停止Runner
sudo gitlab-runner stop

# 重启Runner
sudo gitlab-runner restart

# 验证配置
sudo gitlab-runner verify

# 查看Runner列表
sudo gitlab-runner list

# 注销Runner
sudo gitlab-runner unregister --name docker-runner
```


## 💻 实战应用

### 3.1 快速开始 - 简单Pipeline

**创建第一个Pipeline**:

1. 在项目根目录创建`.gitlab-ci.yml`:
```yaml
# .gitlab-ci.yml

stages:
  - build
  - test

build-job:
  stage: build
  script:
    - echo "开始构建..."
    - mvn clean compile
    - echo "构建完成！"

test-job:
  stage: test
  script:
    - echo "开始测试..."
    - mvn test
    - echo "测试完成！"
```

2. 提交并推送到GitLab:
```bash
git add .gitlab-ci.yml
git commit -m "Add CI/CD pipeline"
git push origin main
```

3. 在GitLab UI中查看Pipeline:
   - 进入项目 → CI/CD → Pipelines
   - 查看Pipeline执行状态和日志

### 3.2 进阶案例 - Spring Boot应用CI/CD

**完整的Spring Boot应用Pipeline**:

```yaml
# .gitlab-ci.yml

# 全局配置
image: maven:3.8-jdk-11

variables:
  MAVEN_OPTS: "-Dmaven.repo.local=$CI_PROJECT_DIR/.m2/repository"
  DOCKER_REGISTRY: "registry.gitlab.com"
  DOCKER_IMAGE: "$CI_REGISTRY_IMAGE"
  KUBERNETES_NAMESPACE: "production"

# 定义阶段
stages:
  - build
  - test
  - quality
  - package
  - deploy

# 缓存配置
cache:
  key: ${CI_COMMIT_REF_SLUG}
  paths:
    - .m2/repository

# 构建
build:
  stage: build
  script:
    - echo "开始构建..."
    - mvn clean compile
  artifacts:
    paths:
      - target/classes
    expire_in: 1 hour
  only:
    - branches
    - tags

# 单元测试
unit-test:
  stage: test
  script:
    - echo "执行单元测试..."
    - mvn test
  artifacts:
    when: always
    reports:
      junit:
        - target/surefire-reports/TEST-*.xml
      coverage_report:
        coverage_format: cobertura
        path: target/site/cobertura/coverage.xml
  coverage: '/Total.*?([0-9]{1,3})%/'
  only:
    - branches
    - tags

# 集成测试
integration-test:
  stage: test
  services:
    - name: mysql:8.0
      alias: mysql
    - name: redis:7
      alias: redis
  variables:
    MYSQL_ROOT_PASSWORD: "root"
    MYSQL_DATABASE: "testdb"
    SPRING_DATASOURCE_URL: "jdbc:mysql://mysql:3306/testdb"
    SPRING_REDIS_HOST: "redis"
  script:
    - echo "执行集成测试..."
    - mvn verify -P integration-test
  artifacts:
    reports:
      junit:
        - target/failsafe-reports/TEST-*.xml
  only:
    - main
    - develop

# 代码质量
code-quality:
  stage: quality
  image: sonarsource/sonar-scanner-cli:latest
  variables:
    SONAR_HOST_URL: $SONAR_HOST_URL
    SONAR_TOKEN: $SONAR_TOKEN
  script:
    - sonar-scanner
      -Dsonar.projectKey=$CI_PROJECT_NAME
      -Dsonar.sources=src/main/java
      -Dsonar.java.binaries=target/classes
      -Dsonar.coverage.jacoco.xmlReportPaths=target/site/jacoco/jacoco.xml
  allow_failure: true
  only:
    - main
    - develop

# 安全扫描
dependency-scan:
  stage: quality
  script:
    - mvn dependency-check:check
  artifacts:
    paths:
      - target/dependency-check-report.html
    expire_in: 1 week
  allow_failure: true
  only:
    - main

# 打包
package:
  stage: package
  script:
    - echo "打包应用..."
    - mvn package -DskipTests
  artifacts:
    paths:
      - target/*.jar
    expire_in: 1 week
  only:
    - main
    - develop
    - tags

# Docker镜像构建
docker-build:
  stage: package
  image: docker:latest
  services:
    - docker:dind
  variables:
    DOCKER_TLS_CERTDIR: "/certs"
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
  script:
    - echo "构建Docker镜像..."
    - |
      if [ "$CI_COMMIT_TAG" ]; then
        TAG=$CI_COMMIT_TAG
      else
        TAG=$CI_COMMIT_SHORT_SHA
      fi
    - docker build -t $DOCKER_IMAGE:$TAG .
    - docker build -t $DOCKER_IMAGE:latest .
    - docker push $DOCKER_IMAGE:$TAG
    - docker push $DOCKER_IMAGE:latest
  only:
    - main
    - tags

# 部署到开发环境
deploy-dev:
  stage: deploy
  image: bitnami/kubectl:latest
  environment:
    name: development
    url: https://dev.example.com
    kubernetes:
      namespace: development
  script:
    - echo "部署到开发环境..."
    - kubectl config use-context dev-cluster
    - kubectl set image deployment/myapp 
      myapp=$DOCKER_IMAGE:$CI_COMMIT_SHORT_SHA 
      -n development
    - kubectl rollout status deployment/myapp -n development --timeout=5m
  only:
    - develop

# 部署到测试环境
deploy-test:
  stage: deploy
  image: bitnami/kubectl:latest
  environment:
    name: testing
    url: https://test.example.com
    kubernetes:
      namespace: testing
  script:
    - echo "部署到测试环境..."
    - kubectl config use-context test-cluster
    - kubectl set image deployment/myapp 
      myapp=$DOCKER_IMAGE:$CI_COMMIT_SHORT_SHA 
      -n testing
    - kubectl rollout status deployment/myapp -n testing --timeout=5m
  when: manual
  only:
    - main

# 部署到生产环境
deploy-prod:
  stage: deploy
  image: bitnami/kubectl:latest
  environment:
    name: production
    url: https://prod.example.com
    kubernetes:
      namespace: production
  script:
    - echo "部署到生产环境..."
    - kubectl config use-context prod-cluster
    - |
      if [ "$CI_COMMIT_TAG" ]; then
        TAG=$CI_COMMIT_TAG
      else
        TAG=$CI_COMMIT_SHORT_SHA
      fi
    - kubectl set image deployment/myapp 
      myapp=$DOCKER_IMAGE:$TAG 
      -n production
    - kubectl rollout status deployment/myapp -n production --timeout=10m
    
    # 健康检查
    - sleep 30
    - kubectl get pods -n production -l app=myapp
    - |
      HEALTH_URL=$(kubectl get svc myapp -n production -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
      curl -f http://$HEALTH_URL/actuator/health || exit 1
  when: manual
  only:
    - main
    - tags
  allow_failure: false

# 回滚生产环境
rollback-prod:
  stage: deploy
  image: bitnami/kubectl:latest
  environment:
    name: production
    action: rollback
  script:
    - echo "回滚生产环境..."
    - kubectl config use-context prod-cluster
    - kubectl rollout undo deployment/myapp -n production
    - kubectl rollout status deployment/myapp -n production --timeout=5m
  when: manual
  only:
    - main
    - tags
```

**Dockerfile示例**:
```dockerfile
# Dockerfile
FROM openjdk:11-jre-slim

WORKDIR /app

COPY target/*.jar app.jar

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "app.jar"]
```


### 3.3 模板和继承

**使用include引入外部配置**:
```yaml
# .gitlab-ci.yml

include:
  # 引入本地文件
  - local: '/templates/build.yml'
  
  # 引入同一项目的其他文件
  - project: 'my-group/my-project'
    ref: main
    file: '/templates/deploy.yml'
  
  # 引入远程文件
  - remote: 'https://example.com/ci-templates/test.yml'
  
  # 引入GitLab模板
  - template: Security/SAST.gitlab-ci.yml

stages:
  - build
  - test
  - deploy
```

**使用extends继承配置**:
```yaml
# 定义基础模板
.deploy-template:
  image: bitnami/kubectl:latest
  before_script:
    - kubectl config use-context $CLUSTER_CONTEXT
  script:
    - kubectl set image deployment/$APP_NAME 
      $APP_NAME=$DOCKER_IMAGE:$TAG 
      -n $NAMESPACE
    - kubectl rollout status deployment/$APP_NAME -n $NAMESPACE

# 继承模板
deploy-dev:
  extends: .deploy-template
  stage: deploy
  environment:
    name: development
  variables:
    CLUSTER_CONTEXT: "dev-cluster"
    NAMESPACE: "development"
    TAG: $CI_COMMIT_SHORT_SHA
  only:
    - develop

deploy-prod:
  extends: .deploy-template
  stage: deploy
  environment:
    name: production
  variables:
    CLUSTER_CONTEXT: "prod-cluster"
    NAMESPACE: "production"
    TAG: $CI_COMMIT_TAG
  when: manual
  only:
    - tags
```

**使用anchor和alias（YAML特性）**:
```yaml
# 定义anchor
.common-config: &common-config
  image: maven:3.8-jdk-11
  cache:
    key: ${CI_COMMIT_REF_SLUG}
    paths:
      - .m2/repository
  before_script:
    - echo "开始执行..."

# 使用alias
build-job:
  <<: *common-config
  stage: build
  script:
    - mvn clean compile

test-job:
  <<: *common-config
  stage: test
  script:
    - mvn test
```

### 3.4 触发器和Pipeline调度

**触发其他项目的Pipeline**:
```yaml
# 项目A的.gitlab-ci.yml
trigger-downstream:
  stage: deploy
  trigger:
    project: my-group/downstream-project
    branch: main
    strategy: depend              # 等待下游Pipeline完成
  only:
    - main
```

**多项目Pipeline**:
```yaml
# 父项目
trigger-child-pipelines:
  stage: deploy
  trigger:
    include:
      - project: 'my-group/frontend'
        file: '.gitlab-ci.yml'
        ref: main
      - project: 'my-group/backend'
        file: '.gitlab-ci.yml'
        ref: main
  only:
    - main
```

**定时Pipeline**:
在GitLab UI中配置:
1. 进入项目 → CI/CD → Schedules
2. 点击"New schedule"
3. 设置Cron表达式和变量
4. 保存

```yaml
# 在.gitlab-ci.yml中使用定时变量
nightly-build:
  script:
    - mvn clean package
  only:
    variables:
      - $CI_PIPELINE_SOURCE == "schedule"
      - $SCHEDULE_TYPE == "nightly"
```


## ✨ 最佳实践

### 4.1 Pipeline设计最佳实践

**1. 使用stages组织Pipeline**:
```yaml
# ✅ 推荐：清晰的阶段划分
stages:
  - build
  - test
  - quality
  - package
  - deploy

# ❌ 不推荐：所有Job在同一阶段
stages:
  - all
```

**2. 合理使用缓存和制品**:
```yaml
# ✅ 推荐：缓存依赖，制品传递构建产物
cache:
  key: ${CI_COMMIT_REF_SLUG}
  paths:
    - .m2/repository

build:
  script:
    - mvn clean package
  artifacts:
    paths:
      - target/*.jar

# ❌ 不推荐：每次都重新下载依赖
build:
  script:
    - mvn clean package
```

**3. 使用rules替代only/except**:
```yaml
# ✅ 推荐：使用rules
deploy:
  script:
    - kubectl apply -f k8s/
  rules:
    - if: '$CI_COMMIT_BRANCH == "main"'
      when: always
    - when: never

# ❌ 不推荐：使用only/except（已过时）
deploy:
  script:
    - kubectl apply -f k8s/
  only:
    - main
```

**4. 使用extends复用配置**:
```yaml
# ✅ 推荐：使用extends
.deploy-template:
  image: kubectl:latest
  script:
    - kubectl apply -f k8s/

deploy-dev:
  extends: .deploy-template
  environment: development

deploy-prod:
  extends: .deploy-template
  environment: production

# ❌ 不推荐：重复配置
deploy-dev:
  image: kubectl:latest
  script:
    - kubectl apply -f k8s/
  environment: development

deploy-prod:
  image: kubectl:latest
  script:
    - kubectl apply -f k8s/
  environment: production
```

**5. 合理设置超时和重试**:
```yaml
# ✅ 推荐：设置超时和重试
deploy:
  script:
    - kubectl apply -f k8s/
  timeout: 10m
  retry:
    max: 2
    when:
      - runner_system_failure
      - stuck_or_timeout_failure
```

### 4.2 性能优化

**1. 使用并行执行**:
```yaml
# 并行测试
test:
  script:
    - npm test -- --shard=$CI_NODE_INDEX/$CI_NODE_TOTAL
  parallel: 5
```

**2. 使用needs跳过不必要的等待**:
```yaml
# ✅ 推荐：使用needs
stages:
  - build
  - test
  - deploy

build:
  stage: build
  script:
    - mvn clean package

test:
  stage: test
  needs: [build]                   # 不等待其他build阶段Job
  script:
    - mvn test

deploy:
  stage: deploy
  needs: [build, test]             # 只等待必要的Job
  script:
    - kubectl apply -f k8s/
```

**3. 优化Docker镜像**:
```yaml
# ✅ 推荐：使用轻量级镜像
image: maven:3.8-jdk-11-slim

# ❌ 不推荐：使用完整镜像
image: maven:3.8-jdk-11
```

**4. 合理配置缓存**:
```yaml
# ✅ 推荐：按分支缓存
cache:
  key: ${CI_COMMIT_REF_SLUG}
  paths:
    - .m2/repository
  policy: pull-push

# 构建Job推送缓存
build:
  cache:
    policy: push

# 测试Job拉取缓存
test:
  cache:
    policy: pull
```

### 4.3 安全最佳实践

**1. 使用保护变量**:
```yaml
# 在GitLab UI中设置保护变量
# Settings > CI/CD > Variables
# ✅ 勾选 "Protect variable"
# ✅ 勾选 "Mask variable"

deploy:
  script:
    - echo $DEPLOY_TOKEN | docker login -u $DEPLOY_USER --password-stdin
```

**2. 限制敏感Job的执行**:
```yaml
deploy-prod:
  script:
    - kubectl apply -f k8s/
  rules:
    - if: '$CI_COMMIT_BRANCH == "main"'
      when: manual
  environment:
    name: production
  only:
    refs:
      - main
    variables:
      - $CI_COMMIT_REF_PROTECTED == "true"
```

**3. 使用安全扫描**:
```yaml
include:
  - template: Security/SAST.gitlab-ci.yml
  - template: Security/Dependency-Scanning.gitlab-ci.yml
  - template: Security/Container-Scanning.gitlab-ci.yml
  - template: Security/Secret-Detection.gitlab-ci.yml
```

**4. 审计和日志**:
```yaml
deploy:
  before_script:
    - echo "部署操作由 $GITLAB_USER_LOGIN 触发"
    - echo "部署时间: $(date)"
    - echo "Commit: $CI_COMMIT_SHORT_SHA"
  script:
    - kubectl apply -f k8s/
  after_script:
    - echo "部署完成，状态: $CI_JOB_STATUS"
```

### 4.4 常见陷阱

**⚠️ 陷阱1：缓存和制品混淆**
```yaml
# ❌ 错误：使用缓存传递构建产物
build:
  script:
    - mvn clean package
  cache:
    paths:
      - target/*.jar              # 不可靠！

# ✅ 正确：使用制品传递构建产物
build:
  script:
    - mvn clean package
  artifacts:
    paths:
      - target/*.jar
```

**⚠️ 陷阱2：忘记设置dependencies**
```yaml
# ❌ 错误：没有指定dependencies，会下载所有制品
deploy:
  script:
    - kubectl apply -f k8s/

# ✅ 正确：只下载需要的制品
deploy:
  dependencies:
    - build
  script:
    - kubectl apply -f k8s/
```

**⚠️ 陷阱3：rules规则顺序错误**
```yaml
# ❌ 错误：when: never在前面，后续规则无效
deploy:
  rules:
    - when: never
    - if: '$CI_COMMIT_BRANCH == "main"'
      when: always

# ✅ 正确：when: never在最后
deploy:
  rules:
    - if: '$CI_COMMIT_BRANCH == "main"'
      when: always
    - when: never
```

**⚠️ 陷阱4：Docker in Docker权限问题**
```yaml
# ❌ 错误：没有设置privileged
docker-build:
  image: docker:latest
  services:
    - docker:dind
  script:
    - docker build -t myapp .

# ✅ 正确：设置privileged
docker-build:
  image: docker:latest
  services:
    - docker:dind
  variables:
    DOCKER_TLS_CERTDIR: "/certs"
  script:
    - docker build -t myapp .
```


## ❓ 常见问题

### Q1: Pipeline执行很慢怎么优化？

**A**: 多方面优化策略：

1. **使用并行执行**:
```yaml
test:
  script:
    - npm test
  parallel: 5
```

2. **使用needs跳过等待**:
```yaml
deploy:
  needs: [build]                   # 不等待其他阶段
  script:
    - kubectl apply -f k8s/
```

3. **优化缓存配置**:
```yaml
cache:
  key: ${CI_COMMIT_REF_SLUG}
  paths:
    - .m2/repository
    - node_modules/
  policy: pull-push
```

4. **使用轻量级Docker镜像**:
```yaml
image: maven:3.8-jdk-11-slim      # 使用slim版本
```

### Q2: 如何调试Pipeline失败？

**A**: 调试技巧：

1. **查看Job日志**:
   - 在GitLab UI中点击失败的Job
   - 查看详细的执行日志

2. **使用echo输出调试信息**:
```yaml
build:
  script:
    - echo "当前目录: $(pwd)"
    - echo "文件列表: $(ls -la)"
    - echo "环境变量: $CI_COMMIT_SHA"
    - mvn clean package
```

3. **使用artifacts保存调试信息**:
```yaml
build:
  script:
    - mvn clean package
  artifacts:
    when: on_failure
    paths:
      - target/
      - logs/
```

4. **本地测试**:
```bash
# 使用gitlab-runner本地执行
gitlab-runner exec docker build-job
```

### Q3: 如何实现多环境部署？

**A**: 多环境部署方案：

```yaml
# 定义环境模板
.deploy-template:
  image: bitnami/kubectl:latest
  script:
    - kubectl config use-context $CLUSTER
    - kubectl set image deployment/$APP_NAME 
      $APP_NAME=$IMAGE:$TAG 
      -n $NAMESPACE
    - kubectl rollout status deployment/$APP_NAME -n $NAMESPACE

# 开发环境
deploy-dev:
  extends: .deploy-template
  stage: deploy
  environment:
    name: development
    url: https://dev.example.com
  variables:
    CLUSTER: "dev-cluster"
    NAMESPACE: "development"
    TAG: $CI_COMMIT_SHORT_SHA
  only:
    - develop

# 测试环境
deploy-test:
  extends: .deploy-template
  stage: deploy
  environment:
    name: testing
    url: https://test.example.com
  variables:
    CLUSTER: "test-cluster"
    NAMESPACE: "testing"
    TAG: $CI_COMMIT_SHORT_SHA
  when: manual
  only:
    - main

# 生产环境
deploy-prod:
  extends: .deploy-template
  stage: deploy
  environment:
    name: production
    url: https://prod.example.com
  variables:
    CLUSTER: "prod-cluster"
    NAMESPACE: "production"
    TAG: $CI_COMMIT_TAG
  when: manual
  only:
    - tags
```

### Q4: 如何处理敏感信息？

**A**: 使用GitLab CI/CD变量：

1. **在GitLab UI中设置变量**:
   - Settings > CI/CD > Variables
   - 添加变量并勾选"Protect variable"和"Mask variable"

2. **在Pipeline中使用**:
```yaml
deploy:
  script:
    - echo $DEPLOY_TOKEN | docker login -u $DEPLOY_USER --password-stdin
    - kubectl create secret generic app-secret 
      --from-literal=api-key=$API_KEY 
      -n production
```

3. **使用文件变量**:
```yaml
deploy:
  script:
    - kubectl apply -f $KUBECONFIG_FILE
```

### Q5: Runner磁盘空间不足怎么办？

**A**: 清理策略：

1. **配置制品过期时间**:
```yaml
build:
  artifacts:
    paths:
      - target/*.jar
    expire_in: 1 week              # 1周后自动删除
```

2. **清理Docker资源**:
```bash
# 在Runner主机上定期执行
docker system prune -a -f

# 添加到crontab
0 2 * * * docker system prune -a -f
```

3. **配置Runner清理策略**:
```toml
# /etc/gitlab-runner/config.toml
[[runners]]
  [runners.docker]
    volumes = ["/cache"]
    
  [runners.cache]
    Type = "s3"
    Shared = true
    [runners.cache.s3]
      ServerAddress = "s3.amazonaws.com"
      BucketName = "gitlab-runner-cache"
```

### Q6: 如何实现蓝绿部署？

**A**: 蓝绿部署示例：

```yaml
deploy-blue:
  stage: deploy
  environment:
    name: production-blue
  script:
    # 部署到蓝环境
    - kubectl apply -f k8s/blue/
    - kubectl rollout status deployment/myapp-blue -n production
    
    # 健康检查
    - sleep 30
    - curl -f http://blue.example.com/health || exit 1
  when: manual
  only:
    - main

switch-to-blue:
  stage: deploy
  environment:
    name: production
  script:
    # 切换流量到蓝环境
    - kubectl patch service myapp -n production 
      -p '{"spec":{"selector":{"version":"blue"}}}'
  when: manual
  needs: [deploy-blue]
  only:
    - main

deploy-green:
  stage: deploy
  environment:
    name: production-green
  script:
    # 部署到绿环境
    - kubectl apply -f k8s/green/
    - kubectl rollout status deployment/myapp-green -n production
    
    # 健康检查
    - sleep 30
    - curl -f http://green.example.com/health || exit 1
  when: manual
  only:
    - main

switch-to-green:
  stage: deploy
  environment:
    name: production
  script:
    # 切换流量到绿环境
    - kubectl patch service myapp -n production 
      -p '{"spec":{"selector":{"version":"green"}}}'
  when: manual
  needs: [deploy-green]
  only:
    - main
```

### Q7: 如何实现金丝雀发布？

**A**: 金丝雀发布示例：

```yaml
deploy-canary:
  stage: deploy
  environment:
    name: production-canary
  script:
    # 部署金丝雀版本（10%流量）
    - kubectl apply -f k8s/canary/
    - kubectl scale deployment/myapp-canary --replicas=1 -n production
    - kubectl scale deployment/myapp-stable --replicas=9 -n production
    
    # 等待并监控
    - sleep 300
    
    # 检查错误率
    - |
      ERROR_RATE=$(curl -s http://prometheus/api/v1/query?query=error_rate | jq '.data.result[0].value[1]')
      if [ $(echo "$ERROR_RATE > 0.01" | bc) -eq 1 ]; then
        echo "错误率过高，回滚"
        exit 1
      fi
  when: manual
  only:
    - main

promote-canary:
  stage: deploy
  environment:
    name: production
  script:
    # 金丝雀验证通过，全量发布
    - kubectl scale deployment/myapp-canary --replicas=10 -n production
    - kubectl scale deployment/myapp-stable --replicas=0 -n production
    - kubectl delete deployment/myapp-stable -n production
    - kubectl apply -f k8s/production/
  when: manual
  needs: [deploy-canary]
  only:
    - main
```

## 🔗 相关资源

### 官方资源
- **官方文档**: https://docs.gitlab.com/ee/ci/
- **Pipeline语法**: https://docs.gitlab.com/ee/ci/yaml/
- **GitLab Runner**: https://docs.gitlab.com/runner/
- **CI/CD示例**: https://docs.gitlab.com/ee/ci/examples/
- **社区论坛**: https://forum.gitlab.com/

### 推荐模板
- **SAST**: 静态应用安全测试
- **Dependency Scanning**: 依赖扫描
- **Container Scanning**: 容器扫描
- **Secret Detection**: 密钥检测
- **License Scanning**: 许可证扫描

### 学习资源
- **GitLab CI/CD教程**: https://docs.gitlab.com/ee/ci/quick_start/
- **Pipeline最佳实践**: https://docs.gitlab.com/ee/ci/pipelines/pipeline_efficiency.html
- **Runner配置指南**: https://docs.gitlab.com/runner/configuration/

### 相关技术
- [Jenkins完整教程](Jenkins-完整教程.md)
- [Docker完整教程](../02-容器化/Docker-完整教程.md)
- [Kubernetes完整教程](../02-容器化/Kubernetes-完整教程.md)
- [Git完整教程](../../08-开发工具链/02-版本控制/Git-完整教程.md)

## 📝 学习检查清单

- [ ] 理解GitLab CI/CD的架构和工作原理
- [ ] 掌握.gitlab-ci.yml的基本语法
- [ ] 熟练使用stages、jobs、scripts
- [ ] 理解缓存和制品的区别和使用
- [ ] 掌握变量和密钥管理
- [ ] 理解rules和条件执行
- [ ] 掌握环境和部署管理
- [ ] 了解GitLab Runner的配置
- [ ] 掌握并行和矩阵构建
- [ ] 理解模板和继承
- [ ] 掌握CI/CD最佳实践
- [ ] 能够排查常见问题

---

**文档版本**: v1.0  
**最后更新**: 2024-01  
**文档来源**: GitLab官方文档  
**@author** erik.zhou
