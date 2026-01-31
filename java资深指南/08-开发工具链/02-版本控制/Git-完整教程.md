# Git 完整教程

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
- **版本**: 2.43+
- **官方文档**: https://git-scm.com/
- **学习难度**: ⭐⭐⭐ (1-5星)
- **重要程度**: ⭐⭐⭐⭐⭐ (1-5星)
- **前置知识**: 命令行基础
- **文档来源**: Git官方文档
- **更新时间**: 2024-01-04

## 🎯 学习目标
- [ ] 理解Git的核心概念和工作原理
- [ ] 掌握Git基本命令的使用
- [ ] 熟练进行分支管理和合并操作
- [ ] 能够解决代码冲突
- [ ] 掌握团队协作工作流
- [ ] 了解Git高级特性和最佳实践

## 📖 基础概念

### 1.1 什么是Git

Git是一个分布式版本控制系统，用于跟踪文件的变化，协调多人协作开发。由Linus Torvalds于2005年创建，最初用于Linux内核开发。

**核心特点**：
- **分布式**: 每个开发者都有完整的代码仓库
- **快速**: 大部分操作在本地完成
- **数据完整性**: 使用SHA-1哈希保证数据完整性
- **分支管理**: 轻量级分支，支持快速切换

### 1.2 核心概念

#### 三个工作区域

```
工作区 (Working Directory)
    ↓ git add
暂存区 (Staging Area / Index)
    ↓ git commit
本地仓库 (Local Repository)
    ↓ git push
远程仓库 (Remote Repository)
```

- **工作区**: 实际编辑文件的目录
- **暂存区**: 临时存储即将提交的修改
- **本地仓库**: 本地的Git数据库
- **远程仓库**: 托管在服务器上的仓库（如GitHub、GitLab）

#### 文件状态

```
未跟踪 (Untracked)
    ↓ git add
已暂存 (Staged)
    ↓ git commit
已提交 (Committed)
    ↓ 修改文件
已修改 (Modified)
```

#### Git对象模型

- **Blob**: 文件内容
- **Tree**: 目录结构
- **Commit**: 提交记录
- **Tag**: 标签引用

### 1.3 应用场景
- 代码版本管理
- 团队协作开发
- 代码审查和合并
- 版本回退和恢复
- 分支开发和实验
- 开源项目贡献

## 🔥 核心特性 (重点)

### 2.1 基础命令 🔥

#### 仓库初始化

```bash
# 初始化新仓库
git init

# 克隆远程仓库
git clone https://github.com/user/repo.git

# 克隆指定分支
git clone -b develop https://github.com/user/repo.git
```

#### 配置Git

```bash
# 配置用户信息
git config --global user.name "Erik Zhou"
git config --global user.email "erik.zhou@example.com"

# 配置默认编辑器
git config --global core.editor "vim"

# 配置别名
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit

# 查看配置
git config --list
git config user.name
```

#### 基本操作

```bash
# 查看状态
git status

# 添加文件到暂存区
git add file.txt              # 添加单个文件
git add *.java                # 添加所有Java文件
git add .                     # 添加所有修改

# 提交更改
git commit -m "提交说明"
git commit -am "添加并提交"   # 跳过暂存区直接提交已跟踪文件

# 查看提交历史
git log
git log --oneline             # 简洁模式
git log --graph               # 图形化显示
git log --author="Erik"       # 按作者过滤
git log --since="2 weeks ago" # 按时间过滤

# 查看文件差异
git diff                      # 工作区 vs 暂存区
git diff --staged             # 暂存区 vs 本地仓库
git diff HEAD                 # 工作区 vs 本地仓库
git diff commit1 commit2      # 两个提交之间的差异
```

### 2.2 分支管理 🔥

#### 分支基础

```bash
# 查看分支
git branch                    # 查看本地分支
git branch -r                 # 查看远程分支
git branch -a                 # 查看所有分支

# 创建分支
git branch feature-login      # 创建分支
git checkout -b feature-login # 创建并切换到分支
git switch -c feature-login   # 新语法：创建并切换

# 切换分支
git checkout main
git switch main               # 新语法

# 删除分支
git branch -d feature-login   # 删除已合并的分支
git branch -D feature-login   # 强制删除分支

# 重命名分支
git branch -m old-name new-name
```

#### 分支合并

```bash
# 合并分支（Fast-forward）
git checkout main
git merge feature-login

# 合并分支（创建合并提交）
git merge --no-ff feature-login

# 查看已合并的分支
git branch --merged

# 查看未合并的分支
git branch --no-merged
```

### 2.3 Rebase vs Merge (⚠️ 难点)

#### Merge（合并）

```bash
# 创建合并提交
git checkout main
git merge feature-branch

# 优点：保留完整的历史记录
# 缺点：历史记录可能变得复杂
```

**Merge示意图**：
```
main:     A---B---C---F
                   /
feature:          D---E
```

#### Rebase（变基）

```bash
# 将feature分支变基到main
git checkout feature-branch
git rebase main

# 或者
git rebase main feature-branch

# 优点：保持线性的提交历史
# 缺点：改写历史，不适合已推送的分支
```

**Rebase示意图**：
```
main:     A---B---C
                   \
feature:            D'---E'
```

#### 何时使用Rebase vs Merge

| 场景 | 推荐 | 原因 |
|------|------|------|
| 本地分支整理 | Rebase | 保持历史清晰 |
| 公共分支合并 | Merge | 保留完整历史 |
| 功能分支合并到主分支 | Merge | 明确功能边界 |
| 同步远程更新 | Rebase | 避免无意义的合并提交 |

**黄金法则**：永远不要rebase已经推送到公共仓库的提交！

### 2.4 冲突解决 (⚠️ 难点)

#### 冲突产生

当两个分支修改了同一文件的同一部分时，合并会产生冲突。

```bash
# 合并时出现冲突
git merge feature-branch
# Auto-merging file.txt
# CONFLICT (content): Merge conflict in file.txt
# Automatic merge failed; fix conflicts and then commit the result.
```

#### 冲突标记

```java
<<<<<<< HEAD
// 当前分支的代码
public void method() {
    System.out.println("Main branch");
}
=======
// 要合并分支的代码
public void method() {
    System.out.println("Feature branch");
}
>>>>>>> feature-branch
```

#### 解决冲突

```bash
# 1. 查看冲突文件
git status

# 2. 手动编辑文件，解决冲突
# 删除冲突标记，保留需要的代码

# 3. 标记为已解决
git add file.txt

# 4. 完成合并
git commit

# 或者放弃合并
git merge --abort
```

#### 使用工具解决冲突

```bash
# 使用mergetool
git mergetool

# 配置mergetool
git config --global merge.tool vimdiff
git config --global merge.tool meld
```

### 2.5 远程仓库操作 🔥

#### 远程仓库管理

```bash
# 查看远程仓库
git remote -v

# 添加远程仓库
git remote add origin https://github.com/user/repo.git

# 修改远程仓库URL
git remote set-url origin https://github.com/user/new-repo.git

# 删除远程仓库
git remote remove origin

# 重命名远程仓库
git remote rename origin upstream
```

#### 推送和拉取

```bash
# 推送到远程仓库
git push origin main
git push -u origin main       # 设置上游分支

# 推送所有分支
git push --all origin

# 推送标签
git push --tags

# 拉取远程更新
git pull origin main          # fetch + merge
git pull --rebase origin main # fetch + rebase

# 仅获取远程更新
git fetch origin

# 查看远程分支
git branch -r

# 跟踪远程分支
git checkout -b local-branch origin/remote-branch
git checkout --track origin/remote-branch
```

## 💻 实战应用

### 3.1 环境搭建

#### 安装Git

**Linux (Ubuntu/Debian)**
```bash
sudo apt-get update
sudo apt-get install git
```

**macOS**
```bash
# 使用Homebrew
brew install git

# 或使用Xcode Command Line Tools
xcode-select --install
```

**Windows**
- 下载Git for Windows: https://git-scm.com/download/win
- 安装时选择"Git Bash"

#### 初始配置

```bash
# 配置用户信息
git config --global user.name "Erik Zhou"
git config --global user.email "erik.zhou@example.com"

# 配置默认分支名
git config --global init.defaultBranch main

# 配置换行符处理
git config --global core.autocrlf input  # Linux/macOS
git config --global core.autocrlf true   # Windows

# 配置颜色输出
git config --global color.ui auto

# 配置中文文件名显示
git config --global core.quotepath false
```

### 3.2 快速开始

#### 创建新项目

```bash
# 1. 创建项目目录
mkdir my-project
cd my-project

# 2. 初始化Git仓库
git init

# 3. 创建README文件
echo "# My Project" > README.md

# 4. 添加并提交
git add README.md
git commit -m "Initial commit"

# 5. 关联远程仓库
git remote add origin https://github.com/user/my-project.git

# 6. 推送到远程
git push -u origin main
```

#### 克隆现有项目

```bash
# 克隆项目
git clone https://github.com/user/project.git

# 进入项目目录
cd project

# 查看远程仓库
git remote -v

# 创建新分支开发
git checkout -b feature-new-feature

# 开发完成后提交
git add .
git commit -m "Add new feature"

# 推送到远程
git push origin feature-new-feature
```

### 3.3 进阶案例

#### Git Flow工作流

```bash
# 1. 初始化Git Flow
git flow init

# 2. 开始新功能开发
git flow feature start user-login

# 3. 开发功能...
git add .
git commit -m "Implement user login"

# 4. 完成功能开发
git flow feature finish user-login

# 5. 开始发布
git flow release start 1.0.0

# 6. 完成发布
git flow release finish 1.0.0

# 7. 紧急修复
git flow hotfix start critical-bug
# 修复bug...
git flow hotfix finish critical-bug
```

#### 交互式Rebase

```bash
# 交互式rebase最近3个提交
git rebase -i HEAD~3

# 在编辑器中可以：
# pick: 保留提交
# reword: 修改提交信息
# edit: 修改提交内容
# squash: 合并到前一个提交
# fixup: 合并到前一个提交，丢弃提交信息
# drop: 删除提交
```

#### Cherry-pick

```bash
# 将特定提交应用到当前分支
git cherry-pick commit-hash

# 应用多个提交
git cherry-pick commit1 commit2 commit3

# 应用提交但不自动提交
git cherry-pick -n commit-hash
```

#### Stash（储藏）

```bash
# 储藏当前修改
git stash

# 储藏并添加说明
git stash save "Work in progress"

# 查看储藏列表
git stash list

# 应用最近的储藏
git stash apply

# 应用并删除储藏
git stash pop

# 应用特定储藏
git stash apply stash@{2}

# 删除储藏
git stash drop stash@{0}

# 清空所有储藏
git stash clear
```

## ✨ 最佳实践

### 4.1 提交规范

#### Commit Message格式

```
<type>(<scope>): <subject>

<body>

<footer>
```

**Type类型**：
- `feat`: 新功能
- `fix`: 修复bug
- `docs`: 文档更新
- `style`: 代码格式调整
- `refactor`: 重构
- `test`: 测试相关
- `chore`: 构建/工具相关

**示例**：
```bash
git commit -m "feat(user): add user login functionality"

git commit -m "fix(api): resolve null pointer exception in user service

- Add null check before accessing user object
- Add unit tests for edge cases

Closes #123"
```

#### 提交频率

- ✅ 小步提交，每个提交只做一件事
- ✅ 提交可编译、可运行的代码
- ✅ 提交前运行测试
- ❌ 避免提交大量未相关的修改
- ❌ 避免提交调试代码和临时文件

### 4.2 分支策略

#### Git Flow

```
main (生产环境)
    ↓
release/* (预发布)
    ↓
develop (开发环境)
    ↓
feature/* (功能开发)

hotfix/* (紧急修复) → main
```

#### GitHub Flow（简化版）

```
main (生产环境)
    ↓
feature/* (功能开发)
```

**流程**：
1. 从main创建功能分支
2. 开发并提交
3. 创建Pull Request
4. 代码审查
5. 合并到main
6. 部署

#### Trunk-Based Development

```
main (主干)
    ↓
short-lived branches (短期分支，1-2天)
```

### 4.3 .gitignore配置

#### Java项目示例

```gitignore
# 编译输出
*.class
*.jar
*.war
*.ear
target/
build/
out/

# IDE
.idea/
*.iml
.vscode/
.eclipse/
.settings/

# 日志
*.log

# 临时文件
*.tmp
*.bak
*.swp
*~

# 操作系统
.DS_Store
Thumbs.db

# Maven
.mvn/
mvnw
mvnw.cmd

# Gradle
.gradle/
gradle/
gradlew
gradlew.bat

# 环境配置
.env
application-local.yml
```

### 4.4 常见陷阱

#### ⚠️ 陷阱1：直接在main分支开发

**问题**：污染主分支，难以回滚

**解决方案**：
```bash
# ✅ 始终在功能分支开发
git checkout -b feature-new-feature
# 开发...
git push origin feature-new-feature
# 通过PR合并到main
```

#### ⚠️ 陷阱2：提交敏感信息

**问题**：密码、密钥等敏感信息被提交到仓库

**解决方案**：
```bash
# 1. 使用.gitignore忽略敏感文件
echo "application-prod.yml" >> .gitignore

# 2. 如果已提交，从历史中删除
git filter-branch --force --index-filter \
  "git rm --cached --ignore-unmatch path/to/sensitive/file" \
  --prune-empty --tag-name-filter cat -- --all

# 3. 使用环境变量或配置管理工具
```

#### ⚠️ 陷阱3：强制推送覆盖他人代码

**问题**：`git push -f`会覆盖远程分支

**解决方案**：
```bash
# ❌ 避免在公共分支使用
git push -f origin main

# ✅ 使用--force-with-lease（更安全）
git push --force-with-lease origin feature-branch

# ✅ 或者先拉取再推送
git pull --rebase origin main
git push origin main
```

## ❓ 常见问题

### Q1: 如何撤销修改？

**A**: 根据不同场景选择

```bash
# 撤销工作区修改（未add）
git checkout -- file.txt
git restore file.txt          # 新语法

# 撤销暂存区修改（已add，未commit）
git reset HEAD file.txt
git restore --staged file.txt # 新语法

# 撤销提交（已commit，未push）
git reset --soft HEAD~1       # 保留修改
git reset --mixed HEAD~1      # 默认，保留工作区修改
git reset --hard HEAD~1       # 丢弃所有修改

# 撤销已推送的提交
git revert commit-hash        # 创建新提交来撤销
```

### Q2: 如何修改最后一次提交？

**A**: 使用--amend

```bash
# 修改提交信息
git commit --amend -m "New commit message"

# 添加遗漏的文件
git add forgotten-file.txt
git commit --amend --no-edit
```

### Q3: 如何查看某个文件的修改历史？

**A**: 使用git log和git blame

```bash
# 查看文件的提交历史
git log -- file.txt

# 查看文件每一行的修改者
git blame file.txt

# 查看文件在某个提交时的内容
git show commit-hash:file.txt
```

### Q4: 如何合并多个提交？

**A**: 使用交互式rebase

```bash
# 合并最近3个提交
git rebase -i HEAD~3

# 在编辑器中将后面的提交改为squash
# pick commit1
# squash commit2
# squash commit3
```

### Q5: 如何找回删除的提交？

**A**: 使用reflog

```bash
# 查看所有操作历史
git reflog

# 恢复到特定提交
git reset --hard commit-hash

# 或创建新分支
git checkout -b recovery-branch commit-hash
```

### Q6: 如何配置SSH密钥？

**A**: 生成并添加SSH密钥

```bash
# 1. 生成SSH密钥
ssh-keygen -t ed25519 -C "erik.zhou@example.com"

# 2. 启动ssh-agent
eval "$(ssh-agent -s)"

# 3. 添加私钥
ssh-add ~/.ssh/id_ed25519

# 4. 复制公钥
cat ~/.ssh/id_ed25519.pub

# 5. 添加到GitHub/GitLab的SSH Keys设置中

# 6. 测试连接
ssh -T git@github.com
```

## 🔗 相关资源

### 官方文档
- [Git官方网站](https://git-scm.com/)
- [Git Book（中文版）](https://git-scm.com/book/zh/v2)
- [GitHub文档](https://docs.github.com/)

### 推荐阅读
- 《Pro Git》- Scott Chacon
- [Git飞行规则](https://github.com/k88hudson/git-flight-rules)
- [Learn Git Branching](https://learngitbranching.js.org/) - 交互式学习

### 常用工具
- [GitKraken](https://www.gitkraken.com/) - Git GUI客户端
- [SourceTree](https://www.sourcetreeapp.com/) - 免费Git GUI
- [GitHub Desktop](https://desktop.github.com/) - GitHub官方客户端
- [Git Extensions](https://gitextensions.github.io/) - Windows Git工具

### 在线练习
- [Git Exercises](https://gitexercises.fracz.com/)
- [Oh My Git!](https://ohmygit.org/) - Git学习游戏

## 📝 学习检查清单
- [ ] 理解Git的核心概念（工作区、暂存区、仓库）
- [ ] 掌握基本命令（add、commit、push、pull）
- [ ] 熟练进行分支操作（创建、切换、合并、删除）
- [ ] 理解rebase和merge的区别
- [ ] 能够解决代码冲突
- [ ] 掌握远程仓库操作
- [ ] 了解Git工作流（Git Flow、GitHub Flow）
- [ ] 能够使用高级特性（stash、cherry-pick、reflog）

---

**@author** erik.zhou
