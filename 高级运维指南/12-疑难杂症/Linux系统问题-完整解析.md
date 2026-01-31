# Linux系统问题-完整解析

> @author erik.zhou

## 📋 目录
- [CPU问题](#cpu问题)
- [内存问题](#内存问题)
- [磁盘问题](#磁盘问题)
- [网络问题](#网络问题)
- [进程问题](#进程问题)
- [系统启动问题](#系统启动问题)

---

## 🔥 CPU问题

### 问题1：CPU使用率100%

**现象**：系统响应缓慢，CPU使用率持续100%

**排查步骤**：
```bash
# 1. 查看整体CPU使用情况
top
htop

# 2. 找出CPU占用最高的进程
ps aux --sort=-%cpu | head -20

# 3. 查看进程详情
top -p PID
pidstat -p PID 1

# 4. 查看进程的线程
top -H -p PID
ps -T -p PID

# 5. 如果是Java进程，查看线程堆栈
jstack PID > thread_dump.txt

# 6. 使用perf分析
perf top -p PID
perf record -p PID -g -- sleep 30
perf report
```

**常见原因**：
- 死循环代码
- 正则表达式回溯
- 频繁GC
- 挖矿病毒

**解决方案**：
- 优化代码逻辑
- 限制进程CPU使用（cgroups）
- 杀死异常进程
- 安全扫描

---

### 问题2：系统负载高但CPU使用率不高

**现象**：load average很高，但CPU使用率正常

**排查步骤**：
```bash
# 1. 查看负载
uptime
cat /proc/loadavg

# 2. 查看CPU等待
vmstat 1
# 关注wa列（IO等待）

# 3. 查看不可中断进程（D状态）
ps aux | awk '$8 ~ /D/'

# 4. 查看IO情况
iostat -x 1
iotop

# 5. 查看是否有大量进程
ps aux | wc -l
```

**常见原因**：
- 磁盘IO瓶颈
- 网络IO等待
- 进程数过多
- NFS挂载问题

---

## 💾 内存问题

### 问题1：内存不足（OOM）

**现象**：进程被OOM Killer杀死

**排查步骤**：
```bash
# 1. 查看OOM日志
dmesg | grep -i "out of memory"
grep -i "killed process" /var/log/messages

# 2. 查看内存使用
free -h
cat /proc/meminfo

# 3. 查看进程内存使用
ps aux --sort=-%mem | head -20
top -o %MEM

# 4. 查看内存详细分布
cat /proc/PID/status | grep -i mem
cat /proc/PID/smaps

# 5. 查看slab缓存
slabtop
cat /proc/slabinfo
```

**常见原因**：
- 内存泄漏
- 缓存占用过多
- 配置的内存限制过小

**解决方案**：
```bash
# 清理缓存（谨慎使用）
sync; echo 3 > /proc/sys/vm/drop_caches

# 调整OOM优先级
echo -17 > /proc/PID/oom_adj

# 调整swappiness
sysctl vm.swappiness=10
```

---

### 问题2：内存泄漏

**现象**：内存使用持续增长，不释放

**排查步骤**：
```bash
# 1. 监控内存变化
watch -n 1 'free -h'

# 2. 查看进程内存变化
pidstat -r -p PID 1

# 3. 使用valgrind检测（开发环境）
valgrind --leak-check=full ./program

# 4. Java应用使用jmap
jmap -heap PID
jmap -histo PID | head -30
jmap -dump:format=b,file=heap.bin PID
```

---

## 💿 磁盘问题

### 问题1：磁盘空间不足

**现象**：No space left on device

**排查步骤**：
```bash
# 1. 查看磁盘使用
df -h
df -i  # inode使用

# 2. 找出大文件
du -sh /* | sort -rh | head -20
find / -type f -size +100M -exec ls -lh {} \;

# 3. 查看已删除但未释放的文件
lsof | grep deleted

# 4. 查看日志文件
du -sh /var/log/*
```

**解决方案**：
```bash
# 清理日志
> /var/log/large.log
logrotate -f /etc/logrotate.conf

# 清理包缓存
yum clean all
apt-get clean

# 清理Docker
docker system prune -a

# 重启进程释放已删除文件
systemctl restart service_name
```

---

### 问题2：磁盘IO高

**现象**：系统响应慢，iowait高

**排查步骤**：
```bash
# 1. 查看IO情况
iostat -x 1
# 关注：%util, await, svctm

# 2. 找出IO高的进程
iotop -o
pidstat -d 1

# 3. 查看进程IO详情
cat /proc/PID/io

# 4. 使用blktrace分析
blktrace -d /dev/sda -o - | blkparse -i -
```

**常见原因**：
- 大量日志写入
- 数据库查询慢
- 备份任务
- 磁盘故障

---

## 🌐 网络问题

### 问题1：网络不通

**排查步骤**：
```bash
# 1. 检查网络配置
ip addr
ip route
cat /etc/resolv.conf

# 2. 测试连通性
ping gateway_ip
ping 8.8.8.8
ping domain.com

# 3. 检查端口
telnet host port
nc -zv host port
curl -v http://host:port

# 4. 检查防火墙
iptables -L -n
firewall-cmd --list-all

# 5. 路由追踪
traceroute host
mtr host

# 6. DNS解析
nslookup domain.com
dig domain.com
```

---

### 问题2：网络延迟高

**排查步骤**：
```bash
# 1. 测试延迟
ping -c 100 host
mtr host

# 2. 抓包分析
tcpdump -i eth0 host target_ip
tcpdump -i eth0 port 80 -w capture.pcap

# 3. 查看网络统计
netstat -s
ss -s

# 4. 查看网卡状态
ethtool eth0
cat /proc/net/dev
```

---

### 问题3：TIME_WAIT过多

**现象**：大量TIME_WAIT连接

**排查步骤**：
```bash
# 查看连接状态统计
ss -s
netstat -an | awk '/tcp/ {print $6}' | sort | uniq -c
```

**解决方案**：
```bash
# 调整内核参数
cat >> /etc/sysctl.conf << EOF
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_fin_timeout = 30
net.ipv4.tcp_max_tw_buckets = 5000
EOF
sysctl -p
```

---

## ⚙️ 进程问题

### 问题1：僵尸进程

**现象**：出现Z状态进程

**排查步骤**：
```bash
# 1. 查找僵尸进程
ps aux | grep Z
ps -ef | grep defunct

# 2. 找到父进程
ps -ef | grep PPID

# 3. 查看进程树
pstree -p
```

**解决方案**：
```bash
# 杀死父进程（僵尸进程会被init回收）
kill -9 PPID

# 或者让父进程正确处理SIGCHLD信号
```

---

### 问题2：进程无法启动

**排查步骤**：
```bash
# 1. 查看错误信息
systemctl status service_name
journalctl -u service_name

# 2. 手动启动查看错误
/path/to/program

# 3. 检查依赖
ldd /path/to/program

# 4. 检查权限
ls -la /path/to/program
namei -l /path/to/program

# 5. 检查端口占用
ss -tlnp | grep port
lsof -i :port

# 6. 检查资源限制
ulimit -a
cat /proc/PID/limits
```

---

## 🔧 系统启动问题

### 问题1：系统无法启动

**排查步骤**：
```bash
# 1. 进入救援模式
# 启动时按e编辑grub，添加 init=/bin/bash 或 rd.break

# 2. 检查文件系统
fsck /dev/sda1

# 3. 检查fstab
cat /etc/fstab

# 4. 查看启动日志
journalctl -xb
dmesg
```

---

### 问题2：服务启动失败

**排查步骤**：
```bash
# 1. 查看服务状态
systemctl status service_name

# 2. 查看详细日志
journalctl -u service_name -f

# 3. 检查配置文件
nginx -t
httpd -t

# 4. 检查依赖服务
systemctl list-dependencies service_name
```

---

## 💡 排查工具速查

| 问题类型 | 常用工具 |
|---------|---------|
| CPU | top, htop, pidstat, perf |
| 内存 | free, vmstat, pmap, valgrind |
| 磁盘 | df, du, iostat, iotop |
| 网络 | ping, traceroute, tcpdump, ss |
| 进程 | ps, pstree, strace, lsof |
| 系统 | dmesg, journalctl, sar |

---

## 📝 学习检查清单

- [ ] 能够排查CPU问题
- [ ] 能够排查内存问题
- [ ] 能够排查磁盘问题
- [ ] 能够排查网络问题
- [ ] 能够排查进程问题
- [ ] 掌握常用排查工具

---

@author erik.zhou
