# Python运维开发-完整教程

> @author erik.zhou

## 📋 目录
- [技术概述](#技术概述)
- [Python基础](#python基础)
- [运维常用库](#运维常用库)
- [系统管理](#系统管理)
- [网络编程](#网络编程)
- [Web开发](#web开发)
- [实战案例](#实战案例)

## 📚 技术概述

### 基本信息
- **重要程度**：⭐⭐⭐⭐⭐ (P0必学)
- **难度级别**：⭐⭐⭐
- **前置知识**：Linux基础
- **学习时长**：40-50小时

### 学习目标
- [ ] 掌握Python基础语法
- [ ] 能够使用运维常用库
- [ ] 能够开发运维工具
- [ ] 能够开发Web管理平台


---

## 🐍 Python基础

### 数据类型 🔥

```python
# 字符串
name = "server01"
ip = '192.168.1.10'
multiline = """
这是多行
字符串
"""

# 字符串操作
print(name.upper())           # SERVER01
print(ip.split('.'))          # ['192', '168', '1', '10']
print(f"Host: {name}, IP: {ip}")  # f-string格式化

# 列表
servers = ['web01', 'web02', 'db01']
servers.append('cache01')
servers.extend(['mq01', 'mq02'])
print(servers[0])             # web01
print(servers[-1])            # mq02
print(servers[1:3])           # ['web02', 'db01']

# 列表推导式
ips = [f"192.168.1.{i}" for i in range(1, 11)]
web_servers = [s for s in servers if s.startswith('web')]

# 字典
server_info = {
    'hostname': 'web01',
    'ip': '192.168.1.10',
    'port': 80,
    'status': 'running'
}
print(server_info['ip'])
print(server_info.get('memory', 'N/A'))

# 字典推导式
server_status = {s: 'running' for s in servers}

# 集合
ports = {80, 443, 8080}
ports.add(3306)
```

### 流程控制

```python
# 条件判断
status = 'running'
if status == 'running':
    print("服务正常")
elif status == 'stopped':
    print("服务已停止")
else:
    print("状态未知")

# 循环
for server in servers:
    print(f"Checking {server}...")

for i, server in enumerate(servers, 1):
    print(f"{i}. {server}")

# while循环
retry = 0
while retry < 3:
    print(f"尝试第 {retry + 1} 次")
    retry += 1

# 异常处理
try:
    result = 10 / 0
except ZeroDivisionError as e:
    print(f"错误: {e}")
except Exception as e:
    print(f"未知错误: {e}")
finally:
    print("清理资源")
```

### 函数

```python
# 基本函数
def check_server(hostname, port=22):
    """检查服务器连通性"""
    print(f"Checking {hostname}:{port}")
    return True

# 可变参数
def deploy(*servers, **options):
    for server in servers:
        print(f"Deploying to {server}")
    print(f"Options: {options}")

deploy('web01', 'web02', version='1.0', env='prod')

# Lambda函数
get_ip = lambda host: f"192.168.1.{host}"

# 装饰器
import time
import functools

def timer(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        print(f"{func.__name__} 耗时: {time.time() - start:.2f}s")
        return result
    return wrapper

@timer
def slow_function():
    time.sleep(1)
    return "Done"
```

### 面向对象

```python
class Server:
    """服务器类"""
    
    def __init__(self, hostname, ip, port=22):
        self.hostname = hostname
        self.ip = ip
        self.port = port
        self.status = 'unknown'
    
    def connect(self):
        """连接服务器"""
        print(f"Connecting to {self.hostname}({self.ip})...")
        self.status = 'connected'
        return True
    
    def execute(self, command):
        """执行命令"""
        if self.status != 'connected':
            raise Exception("Not connected")
        print(f"Executing: {command}")
        return "output"
    
    def __str__(self):
        return f"Server({self.hostname}, {self.ip})"
    
    def __repr__(self):
        return self.__str__()

# 使用
server = Server('web01', '192.168.1.10')
server.connect()
output = server.execute('uptime')
```

---

## 📚 运维常用库

### os和sys模块 🔥

```python
import os
import sys

# 环境变量
print(os.environ.get('HOME'))
os.environ['MY_VAR'] = 'value'

# 路径操作
print(os.path.exists('/etc/hosts'))
print(os.path.join('/var', 'log', 'nginx'))
print(os.path.dirname('/var/log/nginx/access.log'))
print(os.path.basename('/var/log/nginx/access.log'))

# 目录操作
os.makedirs('/tmp/test/subdir', exist_ok=True)
os.listdir('/var/log')
os.walk('/var/log')  # 递归遍历

# 文件操作
os.rename('old.txt', 'new.txt')
os.remove('file.txt')
os.chmod('/tmp/script.sh', 0o755)

# 执行命令
os.system('ls -la')
exit_code = os.system('echo hello')

# sys模块
print(sys.argv)           # 命令行参数
print(sys.path)           # Python路径
print(sys.version)        # Python版本
sys.exit(0)               # 退出程序
```

### subprocess模块 🔥

```python
import subprocess

# 简单执行
result = subprocess.run(['ls', '-la'], capture_output=True, text=True)
print(result.stdout)
print(result.returncode)

# 执行shell命令
result = subprocess.run('ps aux | grep nginx', shell=True, 
                       capture_output=True, text=True)

# 获取输出
output = subprocess.check_output(['hostname'], text=True)

# 管道
p1 = subprocess.Popen(['cat', '/etc/passwd'], stdout=subprocess.PIPE)
p2 = subprocess.Popen(['grep', 'root'], stdin=p1.stdout, stdout=subprocess.PIPE)
output = p2.communicate()[0]

# 超时控制
try:
    result = subprocess.run(['sleep', '10'], timeout=5)
except subprocess.TimeoutExpired:
    print("命令超时")

# 封装执行函数
def run_command(cmd, timeout=30):
    """执行命令并返回结果"""
    try:
        result = subprocess.run(
            cmd, shell=True, capture_output=True, 
            text=True, timeout=timeout
        )
        return {
            'success': result.returncode == 0,
            'stdout': result.stdout,
            'stderr': result.stderr,
            'returncode': result.returncode
        }
    except subprocess.TimeoutExpired:
        return {'success': False, 'error': 'Timeout'}
    except Exception as e:
        return {'success': False, 'error': str(e)}
```

### paramiko - SSH连接 🔥

```python
import paramiko

# SSH连接
def ssh_connect(host, username, password=None, key_file=None):
    """建立SSH连接"""
    client = paramiko.SSHClient()
    client.set_missing_host_key_policy(paramiko.AutoAddPolicy())
    
    if key_file:
        client.connect(host, username=username, key_filename=key_file)
    else:
        client.connect(host, username=username, password=password)
    
    return client

# 执行命令
def ssh_exec(client, command):
    """执行SSH命令"""
    stdin, stdout, stderr = client.exec_command(command)
    return {
        'stdout': stdout.read().decode(),
        'stderr': stderr.read().decode(),
        'exit_code': stdout.channel.recv_exit_status()
    }

# SFTP文件传输
def sftp_upload(client, local_path, remote_path):
    """上传文件"""
    sftp = client.open_sftp()
    sftp.put(local_path, remote_path)
    sftp.close()

def sftp_download(client, remote_path, local_path):
    """下载文件"""
    sftp = client.open_sftp()
    sftp.get(remote_path, local_path)
    sftp.close()

# 使用示例
client = ssh_connect('192.168.1.10', 'root', password='password')
result = ssh_exec(client, 'uptime')
print(result['stdout'])
client.close()
```

### psutil - 系统监控 🔥

```python
import psutil

# CPU信息
print(f"CPU核心数: {psutil.cpu_count()}")
print(f"CPU使用率: {psutil.cpu_percent(interval=1)}%")
print(f"每核使用率: {psutil.cpu_percent(interval=1, percpu=True)}")

# 内存信息
mem = psutil.virtual_memory()
print(f"总内存: {mem.total / 1024 / 1024 / 1024:.2f} GB")
print(f"已用内存: {mem.used / 1024 / 1024 / 1024:.2f} GB")
print(f"内存使用率: {mem.percent}%")

# 磁盘信息
for partition in psutil.disk_partitions():
    usage = psutil.disk_usage(partition.mountpoint)
    print(f"{partition.mountpoint}: {usage.percent}%")

# 网络信息
net = psutil.net_io_counters()
print(f"发送: {net.bytes_sent / 1024 / 1024:.2f} MB")
print(f"接收: {net.bytes_recv / 1024 / 1024:.2f} MB")

# 进程信息
for proc in psutil.process_iter(['pid', 'name', 'cpu_percent', 'memory_percent']):
    if proc.info['cpu_percent'] > 10:
        print(proc.info)

# 系统监控类
class SystemMonitor:
    @staticmethod
    def get_system_info():
        return {
            'cpu_percent': psutil.cpu_percent(interval=1),
            'memory_percent': psutil.virtual_memory().percent,
            'disk_percent': psutil.disk_usage('/').percent,
            'boot_time': psutil.boot_time()
        }
```

### requests - HTTP请求

```python
import requests

# GET请求
response = requests.get('https://api.example.com/servers')
print(response.status_code)
print(response.json())

# POST请求
data = {'hostname': 'web01', 'ip': '192.168.1.10'}
response = requests.post('https://api.example.com/servers', json=data)

# 带认证
response = requests.get(
    'https://api.example.com/servers',
    headers={'Authorization': 'Bearer token'},
    timeout=30
)

# 会话保持
session = requests.Session()
session.headers.update({'Authorization': 'Bearer token'})
response = session.get('https://api.example.com/servers')

# 健康检查函数
def health_check(url, timeout=5):
    """检查服务健康状态"""
    try:
        response = requests.get(url, timeout=timeout)
        return response.status_code == 200
    except requests.RequestException:
        return False
```

---

## 💻 实战案例

### 服务器批量管理工具 🔥

```python
#!/usr/bin/env python3
"""
服务器批量管理工具
@author erik.zhou
"""

import paramiko
import concurrent.futures
from dataclasses import dataclass
from typing import List, Dict

@dataclass
class Server:
    hostname: str
    ip: str
    username: str = 'root'
    password: str = None
    key_file: str = None

class ServerManager:
    def __init__(self, servers: List[Server]):
        self.servers = servers
    
    def _connect(self, server: Server) -> paramiko.SSHClient:
        """建立SSH连接"""
        client = paramiko.SSHClient()
        client.set_missing_host_key_policy(paramiko.AutoAddPolicy())
        
        if server.key_file:
            client.connect(server.ip, username=server.username, 
                          key_filename=server.key_file)
        else:
            client.connect(server.ip, username=server.username, 
                          password=server.password)
        return client
    
    def _exec_command(self, server: Server, command: str) -> Dict:
        """在单台服务器执行命令"""
        try:
            client = self._connect(server)
            stdin, stdout, stderr = client.exec_command(command)
            result = {
                'server': server.hostname,
                'success': True,
                'stdout': stdout.read().decode(),
                'stderr': stderr.read().decode(),
                'exit_code': stdout.channel.recv_exit_status()
            }
            client.close()
            return result
        except Exception as e:
            return {
                'server': server.hostname,
                'success': False,
                'error': str(e)
            }
    
    def batch_exec(self, command: str, max_workers: int = 10) -> List[Dict]:
        """批量执行命令"""
        results = []
        with concurrent.futures.ThreadPoolExecutor(max_workers=max_workers) as executor:
            futures = {
                executor.submit(self._exec_command, server, command): server 
                for server in self.servers
            }
            for future in concurrent.futures.as_completed(futures):
                results.append(future.result())
        return results
    
    def batch_upload(self, local_path: str, remote_path: str) -> List[Dict]:
        """批量上传文件"""
        results = []
        for server in self.servers:
            try:
                client = self._connect(server)
                sftp = client.open_sftp()
                sftp.put(local_path, remote_path)
                sftp.close()
                client.close()
                results.append({'server': server.hostname, 'success': True})
            except Exception as e:
                results.append({'server': server.hostname, 'success': False, 'error': str(e)})
        return results

# 使用示例
if __name__ == '__main__':
    servers = [
        Server('web01', '192.168.1.10', password='password'),
        Server('web02', '192.168.1.11', password='password'),
    ]
    
    manager = ServerManager(servers)
    
    # 批量执行命令
    results = manager.batch_exec('uptime')
    for r in results:
        print(f"{r['server']}: {r.get('stdout', r.get('error'))}")
```

### 系统监控脚本

```python
#!/usr/bin/env python3
"""
系统监控脚本
@author erik.zhou
"""

import psutil
import time
import json
from datetime import datetime

class SystemMonitor:
    def __init__(self, interval=60):
        self.interval = interval
        self.thresholds = {
            'cpu': 80,
            'memory': 85,
            'disk': 90
        }
    
    def collect_metrics(self):
        """收集系统指标"""
        return {
            'timestamp': datetime.now().isoformat(),
            'cpu': {
                'percent': psutil.cpu_percent(interval=1),
                'count': psutil.cpu_count()
            },
            'memory': {
                'total': psutil.virtual_memory().total,
                'used': psutil.virtual_memory().used,
                'percent': psutil.virtual_memory().percent
            },
            'disk': {
                'total': psutil.disk_usage('/').total,
                'used': psutil.disk_usage('/').used,
                'percent': psutil.disk_usage('/').percent
            },
            'network': {
                'bytes_sent': psutil.net_io_counters().bytes_sent,
                'bytes_recv': psutil.net_io_counters().bytes_recv
            }
        }
    
    def check_alerts(self, metrics):
        """检查告警"""
        alerts = []
        if metrics['cpu']['percent'] > self.thresholds['cpu']:
            alerts.append(f"CPU使用率过高: {metrics['cpu']['percent']}%")
        if metrics['memory']['percent'] > self.thresholds['memory']:
            alerts.append(f"内存使用率过高: {metrics['memory']['percent']}%")
        if metrics['disk']['percent'] > self.thresholds['disk']:
            alerts.append(f"磁盘使用率过高: {metrics['disk']['percent']}%")
        return alerts
    
    def run(self):
        """运行监控"""
        print("系统监控启动...")
        while True:
            metrics = self.collect_metrics()
            alerts = self.check_alerts(metrics)
            
            print(json.dumps(metrics, indent=2))
            
            if alerts:
                for alert in alerts:
                    print(f"[ALERT] {alert}")
            
            time.sleep(self.interval)

if __name__ == '__main__':
    monitor = SystemMonitor(interval=10)
    monitor.run()
```

### 日志分析工具

```python
#!/usr/bin/env python3
"""
Nginx日志分析工具
@author erik.zhou
"""

import re
from collections import Counter, defaultdict
from datetime import datetime

class NginxLogAnalyzer:
    # Nginx日志正则
    LOG_PATTERN = re.compile(
        r'(?P<ip>\d+\.\d+\.\d+\.\d+) - - '
        r'\[(?P<time>[^\]]+)\] '
        r'"(?P<method>\w+) (?P<url>[^ ]+) [^"]*" '
        r'(?P<status>\d+) (?P<size>\d+)'
    )
    
    def __init__(self, log_file):
        self.log_file = log_file
        self.entries = []
    
    def parse(self):
        """解析日志文件"""
        with open(self.log_file, 'r') as f:
            for line in f:
                match = self.LOG_PATTERN.match(line)
                if match:
                    self.entries.append(match.groupdict())
        return self
    
    def top_ips(self, n=10):
        """访问量最高的IP"""
        ips = Counter(e['ip'] for e in self.entries)
        return ips.most_common(n)
    
    def top_urls(self, n=10):
        """访问量最高的URL"""
        urls = Counter(e['url'] for e in self.entries)
        return urls.most_common(n)
    
    def status_distribution(self):
        """状态码分布"""
        status = Counter(e['status'] for e in self.entries)
        return dict(status)
    
    def error_urls(self):
        """错误URL统计"""
        errors = [e for e in self.entries if e['status'].startswith(('4', '5'))]
        return Counter(e['url'] for e in errors).most_common(10)
    
    def report(self):
        """生成报告"""
        print("=" * 50)
        print("Nginx日志分析报告")
        print("=" * 50)
        
        print(f"\n总请求数: {len(self.entries)}")
        
        print("\n--- Top 10 IP ---")
        for ip, count in self.top_ips():
            print(f"  {ip}: {count}")
        
        print("\n--- Top 10 URL ---")
        for url, count in self.top_urls():
            print(f"  {url}: {count}")
        
        print("\n--- 状态码分布 ---")
        for status, count in sorted(self.status_distribution().items()):
            print(f"  {status}: {count}")
        
        print("\n--- 错误URL ---")
        for url, count in self.error_urls():
            print(f"  {url}: {count}")

if __name__ == '__main__':
    analyzer = NginxLogAnalyzer('/var/log/nginx/access.log')
    analyzer.parse().report()
```

---

## 🌐 Web开发

### Flask基础

```python
from flask import Flask, request, jsonify

app = Flask(__name__)

# 服务器列表（模拟数据库）
servers = {}

@app.route('/api/servers', methods=['GET'])
def list_servers():
    """获取服务器列表"""
    return jsonify(list(servers.values()))

@app.route('/api/servers', methods=['POST'])
def add_server():
    """添加服务器"""
    data = request.json
    hostname = data.get('hostname')
    servers[hostname] = data
    return jsonify({'message': 'Server added', 'server': data}), 201

@app.route('/api/servers/<hostname>', methods=['GET'])
def get_server(hostname):
    """获取服务器详情"""
    server = servers.get(hostname)
    if server:
        return jsonify(server)
    return jsonify({'error': 'Not found'}), 404

@app.route('/api/servers/<hostname>', methods=['DELETE'])
def delete_server(hostname):
    """删除服务器"""
    if hostname in servers:
        del servers[hostname]
        return jsonify({'message': 'Server deleted'})
    return jsonify({'error': 'Not found'}), 404

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000, debug=True)
```

---

## 📝 学习检查清单

- [ ] 掌握Python基础语法
- [ ] 能够使用os、subprocess模块
- [ ] 能够使用paramiko进行SSH操作
- [ ] 能够使用psutil进行系统监控
- [ ] 能够开发运维自动化工具
- [ ] 能够开发简单的Web API

---

@author erik.zhou
