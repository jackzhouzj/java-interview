# Ansible-完整教程

> @author erik.zhou

## 📋 目录
- [技术概述](#技术概述)
- [Ansible基础](#ansible基础)
- [Inventory管理](#inventory管理)
- [Playbook编写](#playbook编写)
- [Role开发](#role开发)
- [常用模块](#常用模块)
- [实战案例](#实战案例)

## 📚 技术概述

### 基本信息
- **重要程度**：⭐⭐⭐⭐⭐ (P0必学)
- **难度级别**：⭐⭐⭐
- **前置知识**：Linux、SSH、YAML
- **学习时长**：25-35小时
- **官方文档**：https://docs.ansible.com/

### 学习目标
- [ ] 理解Ansible架构
- [ ] 掌握Inventory配置
- [ ] 能够编写Playbook
- [ ] 能够开发Role


---

## 🏗️ Ansible基础

### 架构概览 🔥

```
┌─────────────────────────────────────────────────────────────┐
│                    Ansible Control Node                      │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐  │
│  │  Inventory  │  │  Playbooks  │  │      Modules        │  │
│  │  (主机清单) │  │  (剧本)     │  │  (执行模块)         │  │
│  └─────────────┘  └─────────────┘  └─────────────────────┘  │
└─────────────────────────────┬───────────────────────────────┘
                              │ SSH
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                      Managed Nodes                           │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐         │
│  │   Server1   │  │   Server2   │  │   Server3   │         │
│  └─────────────┘  └─────────────┘  └─────────────┘         │
└─────────────────────────────────────────────────────────────┘
```

### 安装配置

```bash
# 安装Ansible
# CentOS/RHEL
yum install -y epel-release
yum install -y ansible

# Ubuntu/Debian
apt-get update
apt-get install -y ansible

# pip安装
pip install ansible

# 验证安装
ansible --version

# 配置文件位置（优先级从高到低）
# 1. ANSIBLE_CONFIG环境变量
# 2. ./ansible.cfg（当前目录）
# 3. ~/.ansible.cfg（用户目录）
# 4. /etc/ansible/ansible.cfg（系统目录）
```

### 配置文件 🔥

```ini
# ansible.cfg
[defaults]
inventory = ./inventory
remote_user = root
host_key_checking = False
timeout = 30
forks = 10
log_path = ./ansible.log
roles_path = ./roles

[privilege_escalation]
become = True
become_method = sudo
become_user = root
become_ask_pass = False

[ssh_connection]
ssh_args = -o ControlMaster=auto -o ControlPersist=60s
pipelining = True
```

### Ad-Hoc命令

```bash
# 基本语法
ansible <host-pattern> -m <module> -a "<arguments>"

# 测试连通性
ansible all -m ping

# 执行命令
ansible webservers -m shell -a "uptime"
ansible webservers -m command -a "df -h"

# 复制文件
ansible webservers -m copy -a "src=/etc/hosts dest=/tmp/hosts"

# 安装软件
ansible webservers -m yum -a "name=nginx state=present"

# 管理服务
ansible webservers -m service -a "name=nginx state=started enabled=yes"

# 用户管理
ansible webservers -m user -a "name=deploy state=present"

# 指定inventory
ansible -i inventory.ini all -m ping

# 指定用户和密码
ansible all -m ping -u admin -k
ansible all -m ping -u admin --become -K
```

---

## 📋 Inventory管理

### INI格式 🔥

```ini
# inventory.ini
# 单个主机
192.168.1.10
server1.example.com

# 主机组
[webservers]
web1 ansible_host=192.168.1.11
web2 ansible_host=192.168.1.12
web[3:5] ansible_host=192.168.1.1[3:5]  # web3, web4, web5

[dbservers]
db1 ansible_host=192.168.1.21 ansible_port=22
db2 ansible_host=192.168.1.22

# 组变量
[webservers:vars]
http_port=80
ansible_user=deploy

[dbservers:vars]
mysql_port=3306

# 子组
[production:children]
webservers
dbservers

# 全局变量
[all:vars]
ansible_python_interpreter=/usr/bin/python3
```

### YAML格式

```yaml
# inventory.yml
all:
  hosts:
    server1:
      ansible_host: 192.168.1.10
  children:
    webservers:
      hosts:
        web1:
          ansible_host: 192.168.1.11
          http_port: 80
        web2:
          ansible_host: 192.168.1.12
      vars:
        ansible_user: deploy
    dbservers:
      hosts:
        db1:
          ansible_host: 192.168.1.21
        db2:
          ansible_host: 192.168.1.22
      vars:
        mysql_port: 3306
    production:
      children:
        webservers:
        dbservers:
```

### 动态Inventory

```python
#!/usr/bin/env python3
# dynamic_inventory.py
import json

inventory = {
    "webservers": {
        "hosts": ["web1", "web2"],
        "vars": {
            "http_port": 80
        }
    },
    "_meta": {
        "hostvars": {
            "web1": {"ansible_host": "192.168.1.11"},
            "web2": {"ansible_host": "192.168.1.12"}
        }
    }
}

print(json.dumps(inventory))
```

---

## 📖 Playbook编写

### 基本结构 🔥

```yaml
# playbook.yml
---
- name: Configure web servers
  hosts: webservers
  become: yes
  vars:
    http_port: 80
    app_name: myapp
  
  tasks:
    - name: Install nginx
      yum:
        name: nginx
        state: present
      tags: install

    - name: Copy nginx config
      template:
        src: nginx.conf.j2
        dest: /etc/nginx/nginx.conf
      notify: Restart nginx
      tags: config

    - name: Start nginx
      service:
        name: nginx
        state: started
        enabled: yes
      tags: service

  handlers:
    - name: Restart nginx
      service:
        name: nginx
        state: restarted
```

### 变量使用 🔥

```yaml
---
- name: Variable examples
  hosts: all
  vars:
    # 简单变量
    app_name: myapp
    app_port: 8080
    
    # 列表
    packages:
      - nginx
      - vim
      - curl
    
    # 字典
    user:
      name: deploy
      uid: 1001
      shell: /bin/bash

  vars_files:
    - vars/common.yml
    - vars/{{ env }}.yml

  tasks:
    - name: Install packages
      yum:
        name: "{{ item }}"
        state: present
      loop: "{{ packages }}"

    - name: Create user
      user:
        name: "{{ user.name }}"
        uid: "{{ user.uid }}"
        shell: "{{ user.shell }}"

    - name: Debug variable
      debug:
        msg: "App {{ app_name }} running on port {{ app_port }}"

    # 注册变量
    - name: Get hostname
      command: hostname
      register: hostname_result

    - name: Show hostname
      debug:
        var: hostname_result.stdout

    # 条件变量
    - name: Set fact based on OS
      set_fact:
        package_manager: "{{ 'yum' if ansible_os_family == 'RedHat' else 'apt' }}"
```

### 条件判断

```yaml
tasks:
  # when条件
  - name: Install on RedHat
    yum:
      name: nginx
      state: present
    when: ansible_os_family == "RedHat"

  - name: Install on Debian
    apt:
      name: nginx
      state: present
    when: ansible_os_family == "Debian"

  # 多条件
  - name: Complex condition
    debug:
      msg: "Production web server"
    when:
      - env == "production"
      - "'webservers' in group_names"

  # 或条件
  - name: Or condition
    debug:
      msg: "Web or DB server"
    when: "'webservers' in group_names or 'dbservers' in group_names"

  # 基于变量
  - name: Based on variable
    debug:
      msg: "Feature enabled"
    when: feature_enabled | default(false) | bool
```

### 循环

```yaml
tasks:
  # 简单循环
  - name: Install packages
    yum:
      name: "{{ item }}"
      state: present
    loop:
      - nginx
      - vim
      - curl

  # 字典循环
  - name: Create users
    user:
      name: "{{ item.name }}"
      uid: "{{ item.uid }}"
      state: present
    loop:
      - { name: 'user1', uid: 1001 }
      - { name: 'user2', uid: 1002 }

  # with_items（旧语法）
  - name: Copy files
    copy:
      src: "{{ item.src }}"
      dest: "{{ item.dest }}"
    with_items:
      - { src: 'file1.conf', dest: '/etc/app/file1.conf' }
      - { src: 'file2.conf', dest: '/etc/app/file2.conf' }

  # 嵌套循环
  - name: Nested loop
    debug:
      msg: "{{ item[0] }} - {{ item[1] }}"
    loop: "{{ ['a', 'b'] | product(['1', '2']) | list }}"
```

### 模板 (Jinja2) 🔥

```jinja2
{# templates/nginx.conf.j2 #}
# Nginx configuration
# Generated by Ansible

user {{ nginx_user | default('nginx') }};
worker_processes {{ ansible_processor_vcpus }};

events {
    worker_connections {{ worker_connections | default(1024) }};
}

http {
    server {
        listen {{ http_port }};
        server_name {{ server_name }};

        {% for location in locations %}
        location {{ location.path }} {
            proxy_pass {{ location.backend }};
        }
        {% endfor %}

        {% if ssl_enabled | default(false) %}
        listen 443 ssl;
        ssl_certificate {{ ssl_cert }};
        ssl_certificate_key {{ ssl_key }};
        {% endif %}
    }
}
```

---

## 📦 Role开发

### Role结构 🔥

```
roles/
└── nginx/
    ├── defaults/
    │   └── main.yml      # 默认变量
    ├── files/
    │   └── nginx.conf    # 静态文件
    ├── handlers/
    │   └── main.yml      # 处理器
    ├── meta/
    │   └── main.yml      # 元数据和依赖
    ├── tasks/
    │   └── main.yml      # 主任务
    ├── templates/
    │   └── nginx.conf.j2 # 模板文件
    ├── vars/
    │   └── main.yml      # 变量
    └── README.md
```

### Role示例

```yaml
# roles/nginx/defaults/main.yml
---
nginx_user: nginx
nginx_port: 80
nginx_worker_processes: auto
nginx_worker_connections: 1024

# roles/nginx/tasks/main.yml
---
- name: Install nginx
  yum:
    name: nginx
    state: present
  tags: install

- name: Copy nginx config
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
    owner: root
    group: root
    mode: '0644'
  notify: Restart nginx
  tags: config

- name: Ensure nginx is running
  service:
    name: nginx
    state: started
    enabled: yes
  tags: service

# roles/nginx/handlers/main.yml
---
- name: Restart nginx
  service:
    name: nginx
    state: restarted

- name: Reload nginx
  service:
    name: nginx
    state: reloaded

# roles/nginx/meta/main.yml
---
dependencies:
  - role: common
```

### 使用Role

```yaml
# site.yml
---
- name: Configure web servers
  hosts: webservers
  become: yes
  roles:
    - common
    - nginx
    - { role: app, app_port: 8080 }
    - role: monitoring
      vars:
        monitor_port: 9100
      tags: monitoring
```

---

## 🔧 常用模块

### 文件模块

```yaml
# copy - 复制文件
- name: Copy file
  copy:
    src: files/app.conf
    dest: /etc/app/app.conf
    owner: root
    group: root
    mode: '0644'
    backup: yes

# template - 模板
- name: Deploy config
  template:
    src: app.conf.j2
    dest: /etc/app/app.conf
    owner: root
    mode: '0644'

# file - 文件/目录管理
- name: Create directory
  file:
    path: /data/app
    state: directory
    owner: app
    group: app
    mode: '0755'

- name: Create symlink
  file:
    src: /data/app/current
    dest: /opt/app
    state: link

# lineinfile - 行编辑
- name: Add line to file
  lineinfile:
    path: /etc/hosts
    line: "192.168.1.10 server1"
    state: present

# blockinfile - 块编辑
- name: Add block to file
  blockinfile:
    path: /etc/ssh/sshd_config
    block: |
      Match User deploy
        PasswordAuthentication no
```

### 系统模块

```yaml
# yum/apt - 包管理
- name: Install packages
  yum:
    name:
      - nginx
      - vim
    state: present

# service/systemd - 服务管理
- name: Manage service
  systemd:
    name: nginx
    state: started
    enabled: yes
    daemon_reload: yes

# user - 用户管理
- name: Create user
  user:
    name: deploy
    uid: 1001
    group: deploy
    shell: /bin/bash
    create_home: yes

# cron - 定时任务
- name: Add cron job
  cron:
    name: "Backup"
    minute: "0"
    hour: "2"
    job: "/opt/scripts/backup.sh"
```

### 命令模块

```yaml
# command - 执行命令
- name: Run command
  command: /opt/app/start.sh
  args:
    chdir: /opt/app
    creates: /opt/app/app.pid

# shell - 执行shell命令
- name: Run shell
  shell: |
    cd /opt/app
    ./start.sh > /var/log/app.log 2>&1
  args:
    executable: /bin/bash

# script - 执行脚本
- name: Run script
  script: scripts/setup.sh
  args:
    creates: /opt/app/.installed
```

---

## 💻 实战案例

### 部署Web应用

```yaml
# deploy_app.yml
---
- name: Deploy web application
  hosts: webservers
  become: yes
  vars:
    app_name: myapp
    app_version: "1.0.0"
    app_user: deploy
    app_dir: /opt/{{ app_name }}

  tasks:
    - name: Create app user
      user:
        name: "{{ app_user }}"
        system: yes
        create_home: no

    - name: Create app directory
      file:
        path: "{{ app_dir }}"
        state: directory
        owner: "{{ app_user }}"
        mode: '0755'

    - name: Download application
      get_url:
        url: "https://releases.example.com/{{ app_name }}-{{ app_version }}.tar.gz"
        dest: "/tmp/{{ app_name }}-{{ app_version }}.tar.gz"

    - name: Extract application
      unarchive:
        src: "/tmp/{{ app_name }}-{{ app_version }}.tar.gz"
        dest: "{{ app_dir }}"
        remote_src: yes
        owner: "{{ app_user }}"

    - name: Deploy config
      template:
        src: app.conf.j2
        dest: "{{ app_dir }}/config/app.conf"
        owner: "{{ app_user }}"
      notify: Restart app

    - name: Deploy systemd service
      template:
        src: app.service.j2
        dest: /etc/systemd/system/{{ app_name }}.service
      notify:
        - Reload systemd
        - Restart app

    - name: Start application
      systemd:
        name: "{{ app_name }}"
        state: started
        enabled: yes

  handlers:
    - name: Reload systemd
      systemd:
        daemon_reload: yes

    - name: Restart app
      systemd:
        name: "{{ app_name }}"
        state: restarted
```

---

## 💡 最佳实践

1. **使用Role组织代码**：模块化、可复用
2. **变量分层管理**：group_vars、host_vars
3. **使用Vault加密敏感信息**
4. **幂等性设计**：多次执行结果一致
5. **使用tags标记任务**：便于部分执行
6. **测试先行**：使用--check和--diff

---

## 📝 学习检查清单

- [ ] 理解Ansible架构
- [ ] 能够配置Inventory
- [ ] 能够编写Playbook
- [ ] 能够开发Role
- [ ] 掌握常用模块
- [ ] 能够使用Vault加密

---

@author erik.zhou
