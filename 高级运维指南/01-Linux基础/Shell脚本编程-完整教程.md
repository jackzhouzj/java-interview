# Shell脚本编程-完整教程

> @author erik.zhou

## 📋 目录
- [技术概述](#技术概述)
- [基础语法](#基础语法)
- [变量与数组](#变量与数组)
- [流程控制](#流程控制)
- [函数](#函数)
- [文本处理](#文本处理)
- [实战脚本](#实战脚本)

## 📚 技术概述

### 基本信息
- **重要程度**：⭐⭐⭐⭐⭐ (P0必学)
- **难度级别**：⭐⭐⭐
- **前置知识**：Linux基础
- **学习时长**：25-35小时

### 学习目标
- [ ] 掌握Shell基础语法
- [ ] 能够编写自动化脚本
- [ ] 掌握文本处理工具
- [ ] 能够进行日志分析


---

## 🔤 基础语法

### 脚本结构 🔥

```bash
#!/bin/bash
# 脚本说明：这是一个示例脚本
# 作者：erik.zhou
# 日期：2024-01-01

# 变量定义
NAME="World"

# 主逻辑
echo "Hello, $NAME!"

# 退出码
exit 0
```

### 执行脚本

```bash
# 方式1：直接执行（需要执行权限）
chmod +x script.sh
./script.sh

# 方式2：指定解释器
bash script.sh
sh script.sh

# 方式3：source执行（在当前shell执行）
source script.sh
. script.sh

# 调试模式
bash -x script.sh           # 显示执行过程
bash -n script.sh           # 语法检查
set -x                      # 脚本内开启调试
set +x                      # 关闭调试
set -e                      # 遇错即退出
set -u                      # 使用未定义变量报错
```

### 注释

```bash
# 单行注释

: '
多行注释
可以写很多行
'

<<'COMMENT'
另一种多行注释方式
COMMENT
```

---

## 📦 变量与数组

### 变量定义 🔥

```bash
# 变量赋值（等号两边不能有空格）
name="John"
age=25
readonly PI=3.14            # 只读变量

# 使用变量
echo $name
echo ${name}                # 推荐使用花括号
echo "${name}_suffix"       # 拼接字符串

# 删除变量
unset name

# 变量类型
declare -i num=10           # 整数
declare -r const="value"    # 只读
declare -a arr              # 数组
declare -A map              # 关联数组
declare -x var              # 环境变量
```

### 特殊变量 🔥

```bash
$0          # 脚本名称
$1 - $9     # 位置参数
${10}       # 第10个及以后的参数
$#          # 参数个数
$*          # 所有参数（作为一个字符串）
$@          # 所有参数（作为独立字符串）
$?          # 上一个命令的退出码
$$          # 当前脚本PID
$!          # 最后一个后台进程PID
$_          # 上一个命令的最后一个参数
```

### 字符串操作 🔥

```bash
str="Hello World"

# 长度
echo ${#str}                # 11

# 截取
echo ${str:0:5}             # Hello（从0开始取5个）
echo ${str:6}               # World（从6开始到结尾）
echo ${str: -5}             # World（从右边取5个，注意空格）

# 替换
echo ${str/World/Shell}     # Hello Shell（替换第一个）
echo ${str//o/O}            # HellO WOrld（替换所有）

# 删除
echo ${str#Hello }          # World（从左删除最短匹配）
echo ${str##*o}             # rld（从左删除最长匹配）
echo ${str%World}           # Hello （从右删除最短匹配）
echo ${str%%o*}             # Hell（从右删除最长匹配）

# 默认值
echo ${var:-default}        # var为空则返回default
echo ${var:=default}        # var为空则赋值并返回default
echo ${var:+value}          # var不为空则返回value
echo ${var:?error}          # var为空则报错退出
```

### 数组 🔥

```bash
# 定义数组
arr=(a b c d e)
arr[0]="first"
arr[5]="sixth"

# 访问元素
echo ${arr[0]}              # 第一个元素
echo ${arr[@]}              # 所有元素
echo ${arr[*]}              # 所有元素
echo ${#arr[@]}             # 数组长度
echo ${!arr[@]}             # 所有索引

# 切片
echo ${arr[@]:1:3}          # 从索引1开始取3个

# 遍历数组
for item in "${arr[@]}"; do
    echo "$item"
done

# 关联数组（字典）
declare -A map
map["name"]="John"
map["age"]=25
echo ${map["name"]}
echo ${!map[@]}             # 所有键
echo ${map[@]}              # 所有值
```

---

## 🔀 流程控制

### 条件判断 🔥

```bash
# if语句
if [ condition ]; then
    commands
elif [ condition ]; then
    commands
else
    commands
fi

# 单行写法
[ condition ] && command    # 条件为真执行
[ condition ] || command    # 条件为假执行

# 数值比较
[ $a -eq $b ]   # 等于
[ $a -ne $b ]   # 不等于
[ $a -gt $b ]   # 大于
[ $a -ge $b ]   # 大于等于
[ $a -lt $b ]   # 小于
[ $a -le $b ]   # 小于等于

# 字符串比较
[ "$str1" = "$str2" ]       # 相等
[ "$str1" != "$str2" ]      # 不相等
[ -z "$str" ]               # 为空
[ -n "$str" ]               # 不为空

# 文件测试
[ -e file ]     # 存在
[ -f file ]     # 是普通文件
[ -d file ]     # 是目录
[ -r file ]     # 可读
[ -w file ]     # 可写
[ -x file ]     # 可执行
[ -s file ]     # 大小不为0
[ -L file ]     # 是符号链接

# 逻辑运算
[ cond1 ] && [ cond2 ]      # 与
[ cond1 ] || [ cond2 ]      # 或
[ ! condition ]             # 非

# [[ ]] 扩展测试（推荐）
[[ $str =~ regex ]]         # 正则匹配
[[ $str == pattern* ]]      # 模式匹配
[[ $a -gt 5 && $a -lt 10 ]] # 支持&&和||
```

### case语句

```bash
case $var in
    pattern1)
        commands
        ;;
    pattern2|pattern3)
        commands
        ;;
    *)
        default commands
        ;;
esac

# 示例
case $1 in
    start)
        echo "Starting..."
        ;;
    stop)
        echo "Stopping..."
        ;;
    restart)
        echo "Restarting..."
        ;;
    *)
        echo "Usage: $0 {start|stop|restart}"
        exit 1
        ;;
esac
```

### 循环 🔥

```bash
# for循环
for i in 1 2 3 4 5; do
    echo $i
done

for i in {1..10}; do
    echo $i
done

for i in {1..10..2}; do     # 步长为2
    echo $i
done

for ((i=0; i<10; i++)); do
    echo $i
done

for file in *.txt; do
    echo $file
done

# while循环
while [ condition ]; do
    commands
done

# 读取文件
while read line; do
    echo $line
done < file.txt

# 无限循环
while true; do
    commands
    sleep 1
done

# until循环（条件为假时执行）
until [ condition ]; do
    commands
done

# 循环控制
break               # 跳出循环
continue            # 跳过本次循环
break 2             # 跳出2层循环
```

---

## 🔧 函数

### 函数定义 🔥

```bash
# 方式1
function func_name {
    commands
}

# 方式2（推荐）
func_name() {
    commands
}

# 带参数的函数
greet() {
    local name=$1           # 局部变量
    echo "Hello, $name!"
}
greet "World"

# 返回值
add() {
    local result=$(($1 + $2))
    echo $result            # 通过echo返回
}
sum=$(add 3 5)
echo $sum                   # 8

# return只能返回0-255的整数（退出码）
check() {
    if [ -f "$1" ]; then
        return 0
    else
        return 1
    fi
}
check "/etc/passwd" && echo "File exists"
```

### 常用函数示例

```bash
# 日志函数
log() {
    local level=$1
    shift
    local message="$@"
    local timestamp=$(date '+%Y-%m-%d %H:%M:%S')
    echo "[$timestamp] [$level] $message"
}
log "INFO" "Script started"
log "ERROR" "Something went wrong"

# 检查命令是否存在
command_exists() {
    command -v "$1" &> /dev/null
}
if command_exists docker; then
    echo "Docker is installed"
fi

# 确认函数
confirm() {
    read -p "$1 [y/N] " response
    case $response in
        [yY][eE][sS]|[yY])
            return 0
            ;;
        *)
            return 1
            ;;
    esac
}
if confirm "Continue?"; then
    echo "Proceeding..."
fi
```

---

## 📝 文本处理

### 输入输出 🔥

```bash
# 输出
echo "Hello"                # 输出并换行
echo -n "Hello"             # 不换行
echo -e "Hello\tWorld"      # 解释转义字符
printf "Name: %s, Age: %d\n" "John" 25

# 输入
read var                    # 读取输入
read -p "Enter name: " name # 带提示
read -s password            # 不显示输入（密码）
read -t 5 var               # 超时5秒
read -n 1 char              # 只读1个字符

# 重定向
command > file              # 覆盖写入
command >> file             # 追加写入
command < file              # 从文件读取
command 2> file             # 错误输出重定向
command &> file             # 标准输出和错误都重定向
command 2>&1                # 错误重定向到标准输出

# Here Document
cat << EOF
多行文本
可以包含变量 $var
EOF

cat << 'EOF'                # 不解释变量
$var 保持原样
EOF

# 管道
command1 | command2         # 管道
command1 | tee file | command2  # 同时输出到文件和管道
```

### 命令替换

```bash
# 方式1：反引号（不推荐）
result=`command`

# 方式2：$()（推荐）
result=$(command)

# 嵌套
result=$(echo $(date +%Y))

# 算术运算
result=$((1 + 2))
result=$((a + b))
result=$((a++))
let result=a+b
```

---

## 💻 实战脚本

### 系统监控脚本 🔥

```bash
#!/bin/bash
# 系统资源监控脚本
# @author erik.zhou

LOG_FILE="/var/log/system_monitor.log"

log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" >> "$LOG_FILE"
}

# CPU使用率
get_cpu_usage() {
    local cpu_usage=$(top -bn1 | grep "Cpu(s)" | awk '{print $2}' | cut -d'%' -f1)
    echo "$cpu_usage"
}

# 内存使用率
get_mem_usage() {
    local mem_usage=$(free | grep Mem | awk '{printf "%.1f", $3/$2 * 100}')
    echo "$mem_usage"
}

# 磁盘使用率
get_disk_usage() {
    local disk_usage=$(df -h / | tail -1 | awk '{print $5}' | tr -d '%')
    echo "$disk_usage"
}

# 主函数
main() {
    local cpu=$(get_cpu_usage)
    local mem=$(get_mem_usage)
    local disk=$(get_disk_usage)
    
    log "CPU: ${cpu}%, Memory: ${mem}%, Disk: ${disk}%"
    
    # 告警阈值检查
    if (( $(echo "$cpu > 80" | bc -l) )); then
        log "WARNING: CPU usage is high!"
    fi
    
    if (( $(echo "$mem > 80" | bc -l) )); then
        log "WARNING: Memory usage is high!"
    fi
    
    if [ "$disk" -gt 80 ]; then
        log "WARNING: Disk usage is high!"
    fi
}

main
```

### 日志分析脚本

```bash
#!/bin/bash
# Nginx日志分析脚本
# @author erik.zhou

LOG_FILE="/var/log/nginx/access.log"

# 统计访问量最高的10个IP
echo "=== Top 10 IPs ==="
awk '{print $1}' "$LOG_FILE" | sort | uniq -c | sort -rn | head -10

# 统计访问量最高的10个URL
echo -e "\n=== Top 10 URLs ==="
awk '{print $7}' "$LOG_FILE" | sort | uniq -c | sort -rn | head -10

# 统计HTTP状态码分布
echo -e "\n=== HTTP Status Codes ==="
awk '{print $9}' "$LOG_FILE" | sort | uniq -c | sort -rn

# 统计每小时访问量
echo -e "\n=== Hourly Traffic ==="
awk '{print substr($4,14,2)}' "$LOG_FILE" | sort | uniq -c

# 统计404错误
echo -e "\n=== 404 Errors ==="
awk '$9==404 {print $7}' "$LOG_FILE" | sort | uniq -c | sort -rn | head -10
```

### 自动备份脚本

```bash
#!/bin/bash
# 数据库自动备份脚本
# @author erik.zhou

# 配置
DB_HOST="localhost"
DB_USER="root"
DB_PASS="password"
DB_NAME="mydb"
BACKUP_DIR="/backup/mysql"
KEEP_DAYS=7

# 创建备份目录
mkdir -p "$BACKUP_DIR"

# 备份文件名
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="${BACKUP_DIR}/${DB_NAME}_${DATE}.sql.gz"

# 执行备份
echo "Starting backup..."
mysqldump -h"$DB_HOST" -u"$DB_USER" -p"$DB_PASS" "$DB_NAME" | gzip > "$BACKUP_FILE"

if [ $? -eq 0 ]; then
    echo "Backup successful: $BACKUP_FILE"
    # 删除旧备份
    find "$BACKUP_DIR" -name "*.sql.gz" -mtime +$KEEP_DAYS -delete
    echo "Old backups cleaned"
else
    echo "Backup failed!"
    exit 1
fi
```

### 服务健康检查脚本

```bash
#!/bin/bash
# 服务健康检查脚本
# @author erik.zhou

# 配置
SERVICES=("nginx" "mysql" "redis")
WEBHOOK_URL="https://hooks.example.com/alert"

check_service() {
    local service=$1
    if systemctl is-active --quiet "$service"; then
        return 0
    else
        return 1
    fi
}

send_alert() {
    local message=$1
    curl -s -X POST "$WEBHOOK_URL" \
        -H "Content-Type: application/json" \
        -d "{\"text\": \"$message\"}"
}

main() {
    for service in "${SERVICES[@]}"; do
        if ! check_service "$service"; then
            echo "Service $service is down!"
            send_alert "⚠️ Service $service is down on $(hostname)"
            
            # 尝试重启
            systemctl restart "$service"
            sleep 5
            
            if check_service "$service"; then
                send_alert "✅ Service $service has been restarted"
            else
                send_alert "❌ Failed to restart $service"
            fi
        fi
    done
}

main
```

---

## 💡 最佳实践

### 脚本规范

1. **添加shebang**：`#!/bin/bash`
2. **添加注释**：说明脚本用途、作者、日期
3. **使用set命令**：`set -euo pipefail`
4. **变量用引号**：`"$var"` 防止空格问题
5. **使用函数**：模块化代码
6. **错误处理**：检查命令返回值
7. **日志记录**：记录关键操作

### 调试技巧

```bash
# 开启调试
set -x              # 显示执行的命令
set -v              # 显示读取的行
set -e              # 遇错退出
set -u              # 未定义变量报错
set -o pipefail     # 管道中任一命令失败则失败

# 推荐组合
set -euo pipefail

# 错误处理
trap 'echo "Error on line $LINENO"' ERR
```

---

## 📝 学习检查清单

- [ ] 掌握变量定义和使用
- [ ] 能够使用条件判断和循环
- [ ] 能够定义和使用函数
- [ ] 掌握文本处理工具（grep、sed、awk）
- [ ] 能够编写系统监控脚本
- [ ] 能够编写自动化部署脚本
- [ ] 掌握脚本调试技巧

---

@author erik.zhou
