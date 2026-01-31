# Go语言基础 - 完整教程

> @author erik.zhou

## 📚 技术概述

### 版本信息
- **Go版本**：1.21+
- **最新稳定版**：1.21.x
- **推荐版本**：1.21+

### 学习难度
- **难度等级**：⭐⭐ (简单)
- **预计学习时间**：15-20小时
- **重要程度**：⭐⭐⭐⭐⭐ (必学)

### 前置知识
- 基本的编程概念
- 命令行基础
- 了解其他编程语言（推荐）

## 🎯 学习目标

完成本教程后，你将能够：

- [ ] 掌握Go语言基础语法
- [ ] 理解Go的类型系统
- [ ] 掌握控制流程
- [ ] 理解包和模块
- [ ] 能够编写基本的Go程序

## 📖 目录

1. [Go语言简介](#1-go语言简介)
2. [环境搭建](#2-环境搭建)
3. [Hello World](#3-hello-world)
4. [变量和常量](#4-变量和常量)
5. [基本数据类型](#5-基本数据类型)
6. [运算符](#6-运算符)
7. [控制流程](#7-控制流程)
8. [包和导入](#8-包和导入)
9. [输入输出](#9-输入输出)
10. [最佳实践](#10-最佳实践)

---

## 1. Go语言简介

### 1.1 什么是Go

Go（又称Golang）是Google开发的开源编程语言，于2009年发布。

**核心特点**：
- 🔥 **简洁高效**：语法简单，编译快速
- 🔥 **并发支持**：原生支持并发编程
- 🔥 **静态类型**：编译时类型检查
- 🔥 **垃圾回收**：自动内存管理
- 🔥 **跨平台**：支持多平台编译

### 1.2 Go的优势

```go
// 🔥 简洁的语法
func add(a, b int) int {
    return a + b
}

// 🔥 强大的并发
go func() {
    fmt.Println("并发执行")
}()

// 🔥 快速编译
// go build main.go  # 秒级编译

// 🔥 单一二进制文件
// 编译后只有一个可执行文件，部署简单
```

---

## 2. 环境搭建

### 2.1 安装Go

```bash
# 🔥 下载Go
# 访问 https://go.dev/dl/ 下载对应平台的安装包

# Linux/Mac安装
wget https://go.dev/dl/go1.21.0.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.21.0.linux-amd64.tar.gz

# 配置环境变量
export PATH=$PATH:/usr/local/go/bin
export GOPATH=$HOME/go
export PATH=$PATH:$GOPATH/bin

# 验证安装
go version
```

### 2.2 配置Go环境

```bash
# 🔥 查看Go环境
go env

# 设置Go代理（国内推荐）
go env -w GOPROXY=https://goproxy.cn,direct

# 启用Go Modules
go env -w GO111MODULE=on

# 设置私有仓库（可选）
go env -w GOPRIVATE=github.com/yourcompany
```

### 2.3 IDE推荐

```bash
# 🔥 推荐IDE
# 1. VS Code + Go插件（推荐）
# 2. GoLand（JetBrains）
# 3. Vim/Neovim + vim-go

# VS Code安装Go插件
# 1. 安装VS Code
# 2. 安装Go扩展
# 3. 安装Go工具：Ctrl+Shift+P -> Go: Install/Update Tools
```

---

## 3. Hello World

### 3.1 第一个Go程序

```go
// main.go
package main

import "fmt"

func main() {
    fmt.Println("Hello, World!")
}
```

### 3.2 运行程序

```bash
# 🔥 直接运行
go run main.go

# 🔥 编译后运行
go build main.go
./main

# 🔥 编译并指定输出文件名
go build -o hello main.go
./hello
```

### 3.3 程序结构

```go
// 🔥 包声明（必须）
package main

// 🔥 导入包
import (
    "fmt"
    "time"
)

// 🔥 常量定义
const PI = 3.14159

// 🔥 变量定义
var name string = "Go"

// 🔥 init函数（可选，在main之前执行）
func init() {
    fmt.Println("初始化")
}

// 🔥 main函数（程序入口）
func main() {
    fmt.Println("Hello, Go!")
}
```

---

## 4. 变量和常量

### 4.1 变量声明

```go
package main

import "fmt"

func main() {
    // 🔥 方式1：var关键字
    var name string = "Go"
    var age int = 10
    
    // 🔥 方式2：类型推导
    var language = "Golang"
    
    // 🔥 方式3：短变量声明（只能在函数内使用）
    version := "1.21"
    
    // 🔥 方式4：批量声明
    var (
        x int = 1
        y int = 2
        z int = 3
    )
    
    // 🔥 方式5：多变量声明
    var a, b, c int = 1, 2, 3
    m, n := 10, 20
    
    fmt.Println(name, age, language, version)
    fmt.Println(x, y, z)
    fmt.Println(a, b, c, m, n)
}
```

### 4.2 变量作用域

```go
package main

import "fmt"

// 🔥 包级别变量（全局变量）
var globalVar = "全局变量"

func main() {
    // 🔥 函数级别变量
    var localVar = "局部变量"
    
    // 🔥 块级别变量
    if true {
        var blockVar = "块变量"
        fmt.Println(blockVar)
    }
    // fmt.Println(blockVar) // 错误！无法访问
    
    fmt.Println(globalVar, localVar)
}
```

### 4.3 常量

```go
package main

import "fmt"

func main() {
    // 🔥 常量声明
    const PI = 3.14159
    const Name = "Go"
    
    // 🔥 批量声明常量
    const (
        StatusOK = 200
        StatusNotFound = 404
        StatusError = 500
    )
    
    // 🔥 iota枚举
    const (
        Sunday = iota    // 0
        Monday           // 1
        Tuesday          // 2
        Wednesday        // 3
        Thursday         // 4
        Friday           // 5
        Saturday         // 6
    )
    
    // 🔥 iota高级用法
    const (
        _  = iota             // 0（忽略）
        KB = 1 << (10 * iota) // 1 << 10 = 1024
        MB                     // 1 << 20 = 1048576
        GB                     // 1 << 30 = 1073741824
    )
    
    fmt.Println(PI, Name)
    fmt.Println(StatusOK, StatusNotFound)
    fmt.Println(Monday, Friday)
    fmt.Println(KB, MB, GB)
}
```

---

## 5. 基本数据类型

### 5.1 数值类型

```go
package main

import "fmt"

func main() {
    // 🔥 整数类型
    var i8 int8 = 127           // -128 到 127
    var i16 int16 = 32767       // -32768 到 32767
    var i32 int32 = 2147483647  // -2^31 到 2^31-1
    var i64 int64 = 9223372036854775807 // -2^63 到 2^63-1
    
    var ui8 uint8 = 255         // 0 到 255
    var ui16 uint16 = 65535     // 0 到 65535
    var ui32 uint32 = 4294967295 // 0 到 2^32-1
    var ui64 uint64 = 18446744073709551615 // 0 到 2^64-1
    
    // int和uint根据平台自动选择32位或64位
    var i int = 100
    var ui uint = 200
    
    // 🔥 浮点类型
    var f32 float32 = 3.14
    var f64 float64 = 3.141592653589793
    
    // 🔥 复数类型
    var c64 complex64 = 1 + 2i
    var c128 complex128 = 2 + 3i
    
    // 🔥 字节类型
    var b byte = 'A'  // byte是uint8的别名
    var r rune = '中'  // rune是int32的别名，用于Unicode字符
    
    fmt.Println(i8, i16, i32, i64)
    fmt.Println(ui8, ui16, ui32, ui64)
    fmt.Println(i, ui)
    fmt.Println(f32, f64)
    fmt.Println(c64, c128)
    fmt.Println(b, r)
}
```

### 5.2 字符串类型

```go
package main

import "fmt"

func main() {
    // 🔥 字符串声明
    var str1 string = "Hello"
    str2 := "World"
    
    // 🔥 多行字符串
    str3 := `这是一个
多行
字符串`
    
    // 🔥 字符串拼接
    result := str1 + " " + str2
    
    // 🔥 字符串长度
    length := len(str1)
    
    // 🔥 访问字符
    char := str1[0]  // 'H'
    
    // 🔥 字符串遍历
    for i := 0; i < len(str1); i++ {
        fmt.Printf("%c ", str1[i])
    }
    fmt.Println()
    
    // 🔥 按Unicode字符遍历
    for _, r := range "Hello世界" {
        fmt.Printf("%c ", r)
    }
    fmt.Println()
    
    fmt.Println(result, length, char)
    fmt.Println(str3)
}
```

### 5.3 布尔类型

```go
package main

import "fmt"

func main() {
    // 🔥 布尔类型
    var isTrue bool = true
    var isFalse bool = false
    
    // 🔥 布尔运算
    result1 := isTrue && isFalse  // false（与）
    result2 := isTrue || isFalse  // true（或）
    result3 := !isTrue            // false（非）
    
    // 🔥 比较运算
    result4 := 5 > 3   // true
    result5 := 5 == 5  // true
    result6 := 5 != 3  // true
    
    fmt.Println(isTrue, isFalse)
    fmt.Println(result1, result2, result3)
    fmt.Println(result4, result5, result6)
}
```

### 5.4 类型转换

```go
package main

import (
    "fmt"
    "strconv"
)

func main() {
    // 🔥 数值类型转换
    var i int = 42
    var f float64 = float64(i)
    var u uint = uint(i)
    
    // 🔥 字符串转数值
    str := "123"
    num, err := strconv.Atoi(str)
    if err != nil {
        fmt.Println("转换失败:", err)
    }
    
    // 🔥 数值转字符串
    numStr := strconv.Itoa(42)
    
    // 🔥 字符串转浮点数
    floatStr := "3.14"
    floatNum, _ := strconv.ParseFloat(floatStr, 64)
    
    // 🔥 浮点数转字符串
    floatToStr := strconv.FormatFloat(3.14, 'f', 2, 64)
    
    fmt.Println(i, f, u)
    fmt.Println(num, numStr)
    fmt.Println(floatNum, floatToStr)
}
```

---

## 6. 运算符

### 6.1 算术运算符

```go
package main

import "fmt"

func main() {
    a, b := 10, 3
    
    // 🔥 基本运算
    fmt.Println(a + b)  // 13 加法
    fmt.Println(a - b)  // 7  减法
    fmt.Println(a * b)  // 30 乘法
    fmt.Println(a / b)  // 3  除法（整数除法）
    fmt.Println(a % b)  // 1  取余
    
    // 🔥 自增自减
    a++  // a = a + 1
    b--  // b = b - 1
    fmt.Println(a, b)  // 11 2
}
```

### 6.2 比较运算符

```go
package main

import "fmt"

func main() {
    a, b := 5, 3
    
    // 🔥 比较运算
    fmt.Println(a == b)  // false 等于
    fmt.Println(a != b)  // true  不等于
    fmt.Println(a > b)   // true  大于
    fmt.Println(a < b)   // false 小于
    fmt.Println(a >= b)  // true  大于等于
    fmt.Println(a <= b)  // false 小于等于
}
```

### 6.3 逻辑运算符

```go
package main

import "fmt"

func main() {
    a, b := true, false
    
    // 🔥 逻辑运算
    fmt.Println(a && b)  // false 逻辑与
    fmt.Println(a || b)  // true  逻辑或
    fmt.Println(!a)      // false 逻辑非
}
```

### 6.4 位运算符

```go
package main

import "fmt"

func main() {
    a, b := 60, 13  // 60 = 0011 1100, 13 = 0000 1101
    
    // 🔥 位运算
    fmt.Println(a & b)   // 12  = 0000 1100 按位与
    fmt.Println(a | b)   // 61  = 0011 1101 按位或
    fmt.Println(a ^ b)   // 49  = 0011 0001 按位异或
    fmt.Println(a << 2)  // 240 = 1111 0000 左移
    fmt.Println(a >> 2)  // 15  = 0000 1111 右移
}
```

---

## 7. 控制流程

### 7.1 if语句

```go
package main

import "fmt"

func main() {
    // 🔥 基本if语句
    age := 18
    if age >= 18 {
        fmt.Println("成年人")
    }
    
    // 🔥 if-else
    score := 85
    if score >= 90 {
        fmt.Println("优秀")
    } else if score >= 80 {
        fmt.Println("良好")
    } else if score >= 60 {
        fmt.Println("及格")
    } else {
        fmt.Println("不及格")
    }
    
    // 🔥 if语句的初始化
    if num := 10; num > 5 {
        fmt.Println("num大于5")
    }
}
```

### 7.2 switch语句

```go
package main

import "fmt"

func main() {
    // 🔥 基本switch
    day := 3
    switch day {
    case 1:
        fmt.Println("星期一")
    case 2:
        fmt.Println("星期二")
    case 3:
        fmt.Println("星期三")
    case 4:
        fmt.Println("星期四")
    case 5:
        fmt.Println("星期五")
    case 6, 7:
        fmt.Println("周末")
    default:
        fmt.Println("无效的日期")
    }
    
    // 🔥 switch的初始化
    switch num := 10; {
    case num > 0:
        fmt.Println("正数")
    case num < 0:
        fmt.Println("负数")
    default:
        fmt.Println("零")
    }
    
    // 🔥 类型switch
    var x interface{} = "hello"
    switch v := x.(type) {
    case string:
        fmt.Println("字符串:", v)
    case int:
        fmt.Println("整数:", v)
    default:
        fmt.Println("未知类型")
    }
}
```

### 7.3 for循环

```go
package main

import "fmt"

func main() {
    // 🔥 基本for循环
    for i := 0; i < 5; i++ {
        fmt.Println(i)
    }
    
    // 🔥 while风格的for循环
    j := 0
    for j < 5 {
        fmt.Println(j)
        j++
    }
    
    // 🔥 无限循环
    k := 0
    for {
        if k >= 5 {
            break
        }
        fmt.Println(k)
        k++
    }
    
    // 🔥 for-range遍历
    nums := []int{1, 2, 3, 4, 5}
    for index, value := range nums {
        fmt.Printf("索引:%d, 值:%d\n", index, value)
    }
    
    // 🔥 只要值
    for _, value := range nums {
        fmt.Println(value)
    }
    
    // 🔥 只要索引
    for index := range nums {
        fmt.Println(index)
    }
    
    // 🔥 遍历字符串
    for index, char := range "Hello" {
        fmt.Printf("%d: %c\n", index, char)
    }
}
```

### 7.4 break和continue

```go
package main

import "fmt"

func main() {
    // 🔥 break跳出循环
    for i := 0; i < 10; i++ {
        if i == 5 {
            break
        }
        fmt.Println(i)
    }
    
    // 🔥 continue跳过本次循环
    for i := 0; i < 10; i++ {
        if i%2 == 0 {
            continue
        }
        fmt.Println(i)  // 只打印奇数
    }
    
    // 🔥 标签跳转
outer:
    for i := 0; i < 3; i++ {
        for j := 0; j < 3; j++ {
            if i == 1 && j == 1 {
                break outer  // 跳出外层循环
            }
            fmt.Printf("i=%d, j=%d\n", i, j)
        }
    }
}
```

---

## 8. 包和导入

### 8.1 包的概念

```go
// 🔥 包声明
package main  // main包是程序入口

// 🔥 导入单个包
import "fmt"

// 🔥 导入多个包
import (
    "fmt"
    "time"
    "math"
)

// 🔥 包别名
import (
    f "fmt"
    t "time"
)

// 🔥 匿名导入（只执行init函数）
import _ "database/sql"

// 🔥 点导入（不推荐）
import . "fmt"
```

### 8.2 自定义包

```go
// mypackage/utils.go
package mypackage

// 🔥 导出函数（首字母大写）
func Add(a, b int) int {
    return a + b
}

// 🔥 未导出函数（首字母小写）
func subtract(a, b int) int {
    return a - b
}

// 🔥 导出变量
var Version = "1.0.0"

// 🔥 未导出变量
var internal = "内部变量"
```

```go
// main.go
package main

import (
    "fmt"
    "myproject/mypackage"
)

func main() {
    result := mypackage.Add(1, 2)
    fmt.Println(result)
    fmt.Println(mypackage.Version)
    
    // fmt.Println(mypackage.subtract(1, 2))  // 错误！无法访问
    // fmt.Println(mypackage.internal)        // 错误！无法访问
}
```

---

## 9. 输入输出

### 9.1 格式化输出

```go
package main

import "fmt"

func main() {
    name := "Go"
    age := 10
    
    // 🔥 Println（自动换行）
    fmt.Println("Hello", name)
    
    // 🔥 Print（不换行）
    fmt.Print("Hello ")
    fmt.Print(name)
    fmt.Println()
    
    // 🔥 Printf（格式化输出）
    fmt.Printf("Name: %s, Age: %d\n", name, age)
    
    // 🔥 常用格式化动词
    fmt.Printf("%v\n", name)   // 默认格式
    fmt.Printf("%+v\n", name)  // 带字段名
    fmt.Printf("%#v\n", name)  // Go语法表示
    fmt.Printf("%T\n", name)   // 类型
    fmt.Printf("%t\n", true)   // 布尔值
    fmt.Printf("%d\n", 123)    // 十进制整数
    fmt.Printf("%b\n", 123)    // 二进制
    fmt.Printf("%o\n", 123)    // 八进制
    fmt.Printf("%x\n", 123)    // 十六进制
    fmt.Printf("%f\n", 3.14)   // 浮点数
    fmt.Printf("%.2f\n", 3.14159)  // 保留2位小数
    fmt.Printf("%s\n", "hello")    // 字符串
    fmt.Printf("%q\n", "hello")    // 带引号的字符串
    fmt.Printf("%p\n", &name)      // 指针
}
```

### 9.2 输入

```go
package main

import "fmt"

func main() {
    // 🔥 Scan（空格分隔）
    var name string
    var age int
    fmt.Print("请输入姓名和年龄: ")
    fmt.Scan(&name, &age)
    fmt.Printf("姓名: %s, 年龄: %d\n", name, age)
    
    // 🔥 Scanf（格式化输入）
    var x, y int
    fmt.Print("请输入两个数字(用空格分隔): ")
    fmt.Scanf("%d %d", &x, &y)
    fmt.Printf("x=%d, y=%d\n", x, y)
    
    // 🔥 Scanln（读取一行）
    var line string
    fmt.Print("请输入一行文本: ")
    fmt.Scanln(&line)
    fmt.Println("你输入的是:", line)
}
```

---

## 10. 最佳实践

### 10.1 命名规范

```go
// ✅ 好的命名
var userName string
var userAge int
func getUserInfo() {}
type UserInfo struct {}

// ❌ 不好的命名
var user_name string  // 不使用下划线
var UserAge int       // 局部变量不大写
func get_user_info() {}  // 不使用下划线
```

### 10.2 代码格式

```go
// ✅ 使用gofmt格式化代码
// go fmt main.go

// ✅ 使用goimports管理导入
// goimports -w main.go

// ✅ 使用golint检查代码
// golint main.go
```

### 10.3 错误处理

```go
// ✅ 显式处理错误
result, err := someFunction()
if err != nil {
    // 处理错误
    return err
}

// ❌ 忽略错误
result, _ := someFunction()  // 不推荐
```

---

## 📝 学习检查清单

- [ ] 掌握Go语言基础语法
- [ ] 理解变量和常量
- [ ] 掌握基本数据类型
- [ ] 理解运算符
- [ ] 掌握控制流程
- [ ] 理解包和导入
- [ ] 能够进行输入输出
- [ ] 了解Go的最佳实践

---

## 🔗 相关资源

- [Go官方文档](https://go.dev/doc/)
- [Go Tour](https://go.dev/tour/)
- [Effective Go](https://go.dev/doc/effective_go)
- [Go标准库](https://pkg.go.dev/std)
- [Go by Example](https://gobyexample.com/)

---

@author erik.zhou
