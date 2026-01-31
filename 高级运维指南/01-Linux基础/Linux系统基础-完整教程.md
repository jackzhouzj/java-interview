# Linux系统基础-完整教程

> @author erik.zhou

## 📋 目录
- [技术概述](#技术概述)
- [文件系统管理](#文件系统管理)
- [用户与权限管理](#用户与权限管理)
- [进程管理](#进程管理)
- [软件包管理](#软件包管理)
- [系统服务管理](#系统服务管理)
- [磁盘管理](#磁盘管理)
- [常用命令速查](#常用命令速查)

## 📚 技术概述

### 基本信息
- **重要程度**：⭐⭐⭐⭐⭐ (P0必学)
- **难度级别**：⭐⭐⭐
- **前置知识**：计算机基础
- **学习时长**：30-40小时

### 学习目标
- [ ] 理解Linux目录结构
- [ ] 掌握文件和目录操作
- [ ] 能够管理用户和权限
- [ ] 掌握进程管理
- [ ] 能够安装和管理软件包

---

## 📁 文件系统管理

### Linux目录结构 🔥

```
/                   # 根目录
├── bin/            # 基本命令（所有用户可用）
├── sbin/           # 系统管理命令（root用户）
├── etc/            # 配置文件目录
├── home/           # 用户主目录
├── root/           # root用户主目录
├── var/            # 可变数据（日志、缓存等）
│   ├── log/        # 系统日志
│   ├── cache/      # 缓存文件
│   └── lib/        # 程序数据
├── tmp/            # 临时文件
├── usr/            # 用户程序
│   ├── bin/        # 用户命令
│   ├── sbin/       # 系统管理命令
│   ├── lib/        # 库文件
│   ├── local/      # 本地安装的程序
│   └── share/      # 共享数据
├── opt/            # 第三方软件
├── proc/           # 进程信息（虚拟文件系统）
├── sys/            # 系统信息（虚拟文件系统）
├── dev/            # 设备文件
├── mnt/            # 临时挂载点
├── media/          # 可移动设备挂载点
└── boot/           # 启动文件
```

### 文件操作命令 🔥

#### 基本操作
```bash
# 列出文件
ls                  # 列出当前目录文件
ls -l               # 详细信息（长格式）
ls -la              # 包含隐藏文件
ls -lh              # 人性化显示大小
ls -lt              # 按时间排序
ls -lS              # 按大小排序

# 切换目录
cd /path/to/dir     # 切换到指定目录
cd ~                # 切换到用户主目录
cd -                # 切换到上一个目录
cd ..               # 切换到上级目录

# 查看当前目录
pwd                 # 显示当前工作目录

# 创建目录
mkdir dir1          # 创建目录
mkdir -p a/b/c      # 递归创建目录

# 创建文件
touch file.txt      # 创建空文件或更新时间戳
echo "content" > file.txt    # 创建并写入内容

# 复制文件
cp file1 file2      # 复制文件
cp -r dir1 dir2     # 递归复制目录
cp -p file1 file2   # 保留属性复制
cp -a dir1 dir2     # 归档复制（保留所有属性）

# 移动/重命名
mv file1 file2      # 重命名
mv file1 /path/     # 移动文件

# 删除文件
rm file.txt         # 删除文件
rm -r dir/          # 递归删除目录
rm -f file.txt      # 强制删除
rm -rf dir/         # 强制递归删除（危险！）

# 查看文件内容
cat file.txt        # 显示全部内容
head -n 20 file.txt # 显示前20行
tail -n 20 file.txt # 显示后20行
tail -f file.txt    # 实时跟踪文件变化
less file.txt       # 分页查看
more file.txt       # 分页查看

# 查找文件
find /path -name "*.txt"           # 按名称查找
find /path -type f -size +100M     # 查找大于100M的文件
find /path -mtime -7               # 查找7天内修改的文件
find /path -user root              # 查找属于root的文件
find /path -perm 755               # 按权限查找

# 文件类型
file filename       # 查看文件类型

# 链接
ln -s source link   # 创建软链接
ln source link      # 创建硬链接
```

#### 文件权限 🔥

```bash
# 权限说明
# r(4) - 读权限
# w(2) - 写权限
# x(1) - 执行权限

# 权限格式：-rwxrwxrwx
# 第1位：文件类型（- 普通文件，d 目录，l 链接）
# 第2-4位：所有者权限
# 第5-7位：所属组权限
# 第8-10位：其他用户权限

# 修改权限
chmod 755 file.txt          # 数字方式
chmod u+x file.txt          # 符号方式（给所有者添加执行权限）
chmod g-w file.txt          # 移除组的写权限
chmod o=r file.txt          # 设置其他用户只读
chmod -R 755 dir/           # 递归修改

# 修改所有者
chown user file.txt         # 修改所有者
chown user:group file.txt   # 修改所有者和组
chown -R user:group dir/    # 递归修改

# 修改所属组
chgrp group file.txt        # 修改所属组

# 特殊权限
# SUID(4) - 执行时以文件所有者身份运行
# SGID(2) - 执行时以文件所属组身份运行
# Sticky(1) - 只有所有者可以删除文件

chmod 4755 file             # 设置SUID
chmod 2755 dir              # 设置SGID
chmod 1777 dir              # 设置Sticky bit
```

### 文本处理工具 🔥

#### grep - 文本搜索
```bash
grep "pattern" file.txt           # 搜索包含pattern的行
grep -i "pattern" file.txt        # 忽略大小写
grep -v "pattern" file.txt        # 反向匹配（不包含）
grep -n "pattern" file.txt        # 显示行号
grep -r "pattern" dir/            # 递归搜索目录
grep -E "regex" file.txt          # 使用扩展正则
grep -c "pattern" file.txt        # 统计匹配行数
grep -l "pattern" *.txt           # 只显示文件名
grep -A 3 "pattern" file.txt      # 显示匹配行及后3行
grep -B 3 "pattern" file.txt      # 显示匹配行及前3行
grep -C 3 "pattern" file.txt      # 显示匹配行及前后3行
```

#### sed - 流编辑器
```bash
sed 's/old/new/' file.txt         # 替换每行第一个匹配
sed 's/old/new/g' file.txt        # 替换所有匹配
sed -i 's/old/new/g' file.txt     # 直接修改文件
sed -n '5,10p' file.txt           # 打印5-10行
sed '5d' file.txt                 # 删除第5行
sed '/pattern/d' file.txt         # 删除匹配行
sed '1i\new line' file.txt        # 在第1行前插入
sed '1a\new line' file.txt        # 在第1行后追加
```

#### awk - 文本分析
```bash
awk '{print $1}' file.txt                    # 打印第一列
awk '{print $1, $3}' file.txt                # 打印第1和第3列
awk -F: '{print $1}' /etc/passwd             # 指定分隔符
awk '{sum+=$1} END {print sum}' file.txt     # 求和
awk 'NR==5' file.txt                         # 打印第5行
awk 'NR>=5 && NR<=10' file.txt               # 打印5-10行
awk '$3>100 {print $0}' file.txt             # 条件过滤
awk '{print NR, $0}' file.txt                # 添加行号
awk 'BEGIN{OFS=","} {print $1,$2}' file.txt  # 设置输出分隔符
```

---

## 👤 用户与权限管理

### 用户管理 🔥

```bash
# 查看用户信息
whoami                      # 当前用户
id                          # 当前用户详细信息
id username                 # 指定用户信息
who                         # 当前登录用户
w                           # 登录用户详细信息
last                        # 登录历史
lastlog                     # 所有用户最后登录时间

# 用户文件
/etc/passwd                 # 用户信息
/etc/shadow                 # 用户密码（加密）
/etc/group                  # 组信息

# 创建用户
useradd username            # 创建用户
useradd -m username         # 创建用户并创建主目录
useradd -s /bin/bash username   # 指定shell
useradd -g group username   # 指定主组
useradd -G group1,group2 username  # 指定附加组

# 修改用户
usermod -s /bin/zsh username    # 修改shell
usermod -g newgroup username    # 修改主组
usermod -aG group username      # 添加到附加组
usermod -L username             # 锁定用户
usermod -U username             # 解锁用户

# 删除用户
userdel username            # 删除用户
userdel -r username         # 删除用户及主目录

# 密码管理
passwd                      # 修改当前用户密码
passwd username             # 修改指定用户密码
passwd -l username          # 锁定用户
passwd -u username          # 解锁用户
passwd -e username          # 强制下次登录修改密码

# 切换用户
su - username               # 切换用户（加载环境变量）
sudo command                # 以root权限执行命令
sudo -i                     # 切换到root
```

### 组管理

```bash
# 创建组
groupadd groupname          # 创建组
groupadd -g 1001 groupname  # 指定GID

# 修改组
groupmod -n newname oldname # 重命名组
groupmod -g 1002 groupname  # 修改GID

# 删除组
groupdel groupname          # 删除组

# 组成员管理
gpasswd -a user group       # 添加用户到组
gpasswd -d user group       # 从组中删除用户
```

### sudo配置 🔥

```bash
# 编辑sudoers文件
visudo                      # 安全编辑sudoers

# sudoers配置示例
# 用户权限
username ALL=(ALL) ALL                    # 用户可执行所有命令
username ALL=(ALL) NOPASSWD: ALL          # 无需密码
username ALL=(ALL) /usr/bin/systemctl     # 只能执行特定命令

# 组权限
%groupname ALL=(ALL) ALL                  # 组成员可执行所有命令

# 命令别名
Cmnd_Alias NETWORKING = /sbin/route, /sbin/ifconfig
username ALL=(ALL) NETWORKING
```

---

## ⚙️ 进程管理

### 查看进程 🔥

```bash
# ps命令
ps                          # 当前终端进程
ps aux                      # 所有进程（BSD风格）
ps -ef                      # 所有进程（System V风格）
ps -ef | grep nginx         # 查找特定进程
ps aux --sort=-%mem         # 按内存排序
ps aux --sort=-%cpu         # 按CPU排序
ps -p PID                   # 查看特定PID进程
ps -u username              # 查看用户进程
ps -C nginx                 # 按命令名查找

# top命令（实时监控）
top                         # 实时进程监控
top -p PID                  # 监控特定进程
top -u username             # 监控用户进程
# top交互命令：
# P - 按CPU排序
# M - 按内存排序
# k - 杀死进程
# q - 退出

# htop（增强版top）
htop                        # 更友好的进程监控

# 进程树
pstree                      # 显示进程树
pstree -p                   # 显示PID
pstree -u                   # 显示用户

# 查看进程详情
cat /proc/PID/status        # 进程状态
cat /proc/PID/cmdline       # 启动命令
ls -l /proc/PID/fd          # 打开的文件描述符
ls -l /proc/PID/cwd         # 工作目录
```

### 进程控制 🔥

```bash
# 前后台控制
command &                   # 后台运行
Ctrl+Z                      # 暂停当前进程
bg                          # 将暂停的进程放到后台运行
fg                          # 将后台进程放到前台
jobs                        # 查看后台任务
nohup command &             # 后台运行（退出终端不停止）

# 杀死进程
kill PID                    # 发送SIGTERM信号
kill -9 PID                 # 强制杀死（SIGKILL）
kill -15 PID                # 优雅终止（SIGTERM）
kill -HUP PID               # 重新加载配置
killall nginx               # 按名称杀死所有进程
pkill -f "pattern"          # 按模式杀死进程

# 信号列表
kill -l                     # 列出所有信号
# 常用信号：
# 1  SIGHUP   - 挂起/重新加载配置
# 2  SIGINT   - 中断（Ctrl+C）
# 9  SIGKILL  - 强制终止
# 15 SIGTERM  - 优雅终止
# 18 SIGCONT  - 继续
# 19 SIGSTOP  - 停止

# 进程优先级
nice -n 10 command          # 以较低优先级运行
renice -n 5 -p PID          # 修改运行中进程优先级
```

---

## 📦 软件包管理

### YUM/DNF (RHEL/CentOS) 🔥

```bash
# 查询
yum list                    # 列出所有包
yum list installed          # 已安装的包
yum search keyword          # 搜索包
yum info package            # 包信息
yum provides /path/file     # 查找文件属于哪个包

# 安装
yum install package         # 安装包
yum install -y package      # 自动确认安装
yum localinstall file.rpm   # 安装本地rpm包
yum groupinstall "Group"    # 安装包组

# 更新
yum update                  # 更新所有包
yum update package          # 更新指定包
yum check-update            # 检查可用更新

# 删除
yum remove package          # 删除包
yum autoremove              # 删除不需要的依赖

# 清理
yum clean all               # 清理缓存
yum makecache               # 重建缓存

# 仓库管理
yum repolist                # 列出仓库
yum-config-manager --add-repo URL   # 添加仓库
yum-config-manager --enable repo    # 启用仓库
yum-config-manager --disable repo   # 禁用仓库

# DNF (CentOS 8+)
dnf install package         # 安装包
dnf module list             # 列出模块
dnf module enable module    # 启用模块
```

### APT (Debian/Ubuntu) 🔥

```bash
# 更新源
apt update                  # 更新包列表
apt upgrade                 # 升级所有包
apt full-upgrade            # 完整升级

# 查询
apt list                    # 列出所有包
apt list --installed        # 已安装的包
apt search keyword          # 搜索包
apt show package            # 包信息

# 安装
apt install package         # 安装包
apt install -y package      # 自动确认
apt install ./file.deb      # 安装本地deb包

# 删除
apt remove package          # 删除包
apt purge package           # 删除包及配置
apt autoremove              # 删除不需要的依赖

# 清理
apt clean                   # 清理缓存
apt autoclean               # 清理旧缓存

# 源管理
# 编辑 /etc/apt/sources.list
add-apt-repository ppa:xxx  # 添加PPA源
```

### RPM基础

```bash
rpm -ivh package.rpm        # 安装
rpm -Uvh package.rpm        # 升级
rpm -e package              # 删除
rpm -qa                     # 列出所有已安装包
rpm -qi package             # 包信息
rpm -ql package             # 包文件列表
rpm -qf /path/file          # 查找文件属于哪个包
```

---

## 🔧 系统服务管理

### Systemd 🔥

```bash
# 服务管理
systemctl start service     # 启动服务
systemctl stop service      # 停止服务
systemctl restart service   # 重启服务
systemctl reload service    # 重新加载配置
systemctl status service    # 查看状态
systemctl enable service    # 开机自启
systemctl disable service   # 禁止开机自启
systemctl is-enabled service    # 检查是否自启
systemctl is-active service     # 检查是否运行

# 查看服务
systemctl list-units        # 列出所有单元
systemctl list-units --type=service     # 列出服务
systemctl list-unit-files   # 列出所有单元文件
systemctl list-dependencies service     # 查看依赖

# 系统管理
systemctl poweroff          # 关机
systemctl reboot            # 重启
systemctl suspend           # 挂起

# 日志查看
journalctl                  # 查看所有日志
journalctl -u service       # 查看服务日志
journalctl -f               # 实时跟踪日志
journalctl --since "1 hour ago"     # 最近1小时日志
journalctl -p err           # 只看错误日志
journalctl -b               # 本次启动的日志
```

### 自定义服务

```bash
# 创建服务文件 /etc/systemd/system/myapp.service
[Unit]
Description=My Application
After=network.target

[Service]
Type=simple
User=myuser
WorkingDirectory=/opt/myapp
ExecStart=/opt/myapp/start.sh
ExecStop=/opt/myapp/stop.sh
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target

# 重新加载配置
systemctl daemon-reload

# 启动服务
systemctl start myapp
systemctl enable myapp
```

---

## 💾 磁盘管理

### 磁盘查看 🔥

```bash
# 磁盘空间
df -h                       # 查看磁盘使用情况
df -i                       # 查看inode使用情况
df -T                       # 显示文件系统类型

# 目录大小
du -sh /path                # 目录总大小
du -sh *                    # 当前目录下各项大小
du -h --max-depth=1         # 一级子目录大小
du -ah | sort -rh | head -20    # 最大的20个文件/目录

# 磁盘信息
lsblk                       # 列出块设备
fdisk -l                    # 列出磁盘分区
blkid                       # 显示块设备UUID
```

### 磁盘分区

```bash
# 分区工具
fdisk /dev/sdb              # MBR分区
gdisk /dev/sdb              # GPT分区
parted /dev/sdb             # 通用分区工具

# fdisk交互命令
# n - 新建分区
# d - 删除分区
# p - 打印分区表
# w - 保存退出
# q - 不保存退出

# 格式化
mkfs.ext4 /dev/sdb1         # 格式化为ext4
mkfs.xfs /dev/sdb1          # 格式化为xfs
```

### 挂载管理

```bash
# 临时挂载
mount /dev/sdb1 /mnt/data   # 挂载
umount /mnt/data            # 卸载
mount -o remount,rw /       # 重新挂载

# 永久挂载 /etc/fstab
/dev/sdb1  /mnt/data  ext4  defaults  0  0
UUID=xxx   /mnt/data  ext4  defaults  0  0

# 挂载所有
mount -a                    # 挂载fstab中所有设备
```

### LVM管理

```bash
# 物理卷
pvcreate /dev/sdb1          # 创建PV
pvs                         # 查看PV
pvdisplay                   # PV详情

# 卷组
vgcreate vg01 /dev/sdb1     # 创建VG
vgextend vg01 /dev/sdc1     # 扩展VG
vgs                         # 查看VG
vgdisplay                   # VG详情

# 逻辑卷
lvcreate -L 10G -n lv01 vg01    # 创建LV
lvextend -L +5G /dev/vg01/lv01  # 扩展LV
lvs                         # 查看LV
lvdisplay                   # LV详情

# 扩展文件系统
resize2fs /dev/vg01/lv01    # ext4扩展
xfs_growfs /dev/vg01/lv01   # xfs扩展
```

---

## 📋 常用命令速查

### 系统信息

```bash
uname -a                    # 系统信息
hostname                    # 主机名
uptime                      # 运行时间
date                        # 当前时间
cal                         # 日历
free -h                     # 内存使用
lscpu                       # CPU信息
lsmem                       # 内存信息
dmidecode                   # 硬件信息
```

### 网络命令

```bash
ip addr                     # IP地址
ip route                    # 路由表
ss -tunlp                   # 端口监听
netstat -tunlp              # 端口监听（旧）
ping host                   # 连通性测试
traceroute host             # 路由追踪
curl url                    # HTTP请求
wget url                    # 下载文件
scp file user@host:/path    # 远程复制
rsync -avz src dest         # 同步文件
```

### 压缩解压

```bash
# tar
tar -cvf file.tar dir/      # 打包
tar -xvf file.tar           # 解包
tar -czvf file.tar.gz dir/  # 打包压缩
tar -xzvf file.tar.gz       # 解压
tar -cjvf file.tar.bz2 dir/ # bz2压缩
tar -xjvf file.tar.bz2      # bz2解压

# zip
zip -r file.zip dir/        # 压缩
unzip file.zip              # 解压

# gzip
gzip file                   # 压缩
gunzip file.gz              # 解压
```

### 定时任务

```bash
# crontab
crontab -e                  # 编辑定时任务
crontab -l                  # 列出定时任务
crontab -r                  # 删除所有任务

# cron格式
# 分 时 日 月 周 命令
# *  *  *  *  *  command
0 2 * * * /path/backup.sh   # 每天2点执行
*/5 * * * * /path/check.sh  # 每5分钟执行
0 0 * * 0 /path/weekly.sh   # 每周日0点执行
```

---

## 💡 最佳实践

### 安全建议

1. **最小权限原则**：只给必要的权限
2. **禁用root远程登录**：使用普通用户+sudo
3. **定期更新系统**：及时修复安全漏洞
4. **配置防火墙**：只开放必要端口
5. **定期备份**：重要数据定期备份

### 运维习惯

1. **操作前备份**：修改配置前先备份
2. **记录操作日志**：重要操作记录在案
3. **使用版本控制**：配置文件纳入Git管理
4. **自动化脚本**：重复操作编写脚本
5. **监控告警**：关键指标设置告警

---

## 📝 学习检查清单

- [ ] 能够熟练使用文件操作命令
- [ ] 理解Linux权限模型
- [ ] 能够管理用户和组
- [ ] 掌握进程查看和控制
- [ ] 能够安装和管理软件包
- [ ] 能够配置systemd服务
- [ ] 掌握磁盘分区和挂载
- [ ] 能够使用grep、sed、awk处理文本

---

@author erik.zhou
