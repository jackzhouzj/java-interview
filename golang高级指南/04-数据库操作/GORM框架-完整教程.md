# GORM框架 - 完整教程

> @author erik.zhou

## 📚 技术概述

### 版本信息
- **GORM版本**：1.25+
- **Go版本**：1.21+
- **数据库**：MySQL、PostgreSQL、SQLite、SQL Server

### 学习难度
- **难度等级**：⭐⭐⭐ (中等)
- **预计学习时间**：15-20小时
- **重要程度**：⭐⭐⭐⭐⭐ (必学)

### 前置知识
- Go语言基础
- SQL基础
- 数据库基本概念
- 结构体和标签

## 🎯 学习目标

完成本教程后，你将能够：

- [ ] 掌握GORM的基本使用
- [ ] 理解GORM的模型定义
- [ ] 能够进行CRUD操作
- [ ] 掌握关联查询
- [ ] 理解事务处理
- [ ] 能够使用钩子函数
- [ ] 掌握GORM的高级特性

## 📖 目录

1. [GORM简介](#1-gorm简介)
2. [连接数据库](#2-连接数据库)
3. [模型定义](#3-模型定义)
4. [CRUD操作](#4-crud操作)
5. [查询](#5-查询)
6. [关联](#6-关联)
7. [事务](#7-事务)
8. [钩子](#8-钩子)
9. [最佳实践](#9-最佳实践)

---

## 1. GORM简介

### 1.1 什么是GORM

GORM是Go语言的ORM（Object-Relational Mapping）库，提供了完整的数据库操作功能。

**核心特点**：
- 🔥 **全功能ORM**：支持CRUD、关联、事务等
- 🔥 **自动迁移**：自动创建和更新表结构
- 🔥 **钩子支持**：Before/After钩子
- 🔥 **预加载**：支持Preload和Joins
- 🔥 **事务支持**：完整的事务支持

### 1.2 安装GORM

```bash
# 🔥 安装GORM
go get -u gorm.io/gorm

# 安装MySQL驱动
go get -u gorm.io/driver/mysql

# 安装PostgreSQL驱动
go get -u gorm.io/driver/postgres

# 安装SQLite驱动
go get -u gorm.io/driver/sqlite
```

---

## 2. 连接数据库

### 2.1 连接MySQL

```go
package main

import (
    "gorm.io/driver/mysql"
    "gorm.io/gorm"
    "log"
)

func main() {
    // 🔥 MySQL连接字符串
    dsn := "user:password@tcp(127.0.0.1:3306)/dbname?charset=utf8mb4&parseTime=True&loc=Local"
    
    db, err := gorm.Open(mysql.Open(dsn), &gorm.Config{})
    if err != nil {
        log.Fatal("连接数据库失败:", err)
    }
    
    // 获取底层sql.DB
    sqlDB, err := db.DB()
    if err != nil {
        log.Fatal(err)
    }
    
    // 🔥 设置连接池
    sqlDB.SetMaxIdleConns(10)           // 最大空闲连接数
    sqlDB.SetMaxOpenConns(100)          // 最大打开连接数
    sqlDB.SetConnMaxLifetime(time.Hour) // 连接最大生命周期
    
    log.Println("数据库连接成功")
}
```

### 2.2 GORM配置

```go
package main

import (
    "gorm.io/driver/mysql"
    "gorm.io/gorm"
    "gorm.io/gorm/logger"
    "log"
    "os"
    "time"
)

func main() {
    // 🔥 自定义日志
    newLogger := logger.New(
        log.New(os.Stdout, "\r\n", log.LstdFlags),
        logger.Config{
            SlowThreshold:             time.Second,   // 慢SQL阈值
            LogLevel:                  logger.Info,   // 日志级别
            IgnoreRecordNotFoundError: true,          // 忽略记录未找到错误
            Colorful:                  true,          // 彩色打印
        },
    )
    
    dsn := "user:password@tcp(127.0.0.1:3306)/dbname?charset=utf8mb4&parseTime=True&loc=Local"
    
    db, err := gorm.Open(mysql.Open(dsn), &gorm.Config{
        Logger:                                   newLogger,
        SkipDefaultTransaction:                   true,  // 跳过默认事务
        DisableForeignKeyConstraintWhenMigrating: true,  // 禁用外键约束
        PrepareStmt:                              true,  // 预编译语句
    })
    
    if err != nil {
        log.Fatal(err)
    }
    
    log.Println("数据库连接成功")
}
```

---

## 3. 模型定义

### 3.1 基本模型

```go
package main

import (
    "gorm.io/gorm"
    "time"
)

// 🔥 基本模型
type User struct {
    ID        uint           `gorm:"primaryKey"`
    Name      string         `gorm:"size:100;not null"`
    Email     string         `gorm:"size:100;uniqueIndex"`
    Age       int            `gorm:"default:0"`
    Birthday  *time.Time
    CreatedAt time.Time
    UpdatedAt time.Time
    DeletedAt gorm.DeletedAt `gorm:"index"`
}

// 🔥 使用gorm.Model（包含ID、CreatedAt、UpdatedAt、DeletedAt）
type Product struct {
    gorm.Model
    Code  string `gorm:"size:100;uniqueIndex"`
    Price uint
}

// 🔥 自定义表名
func (User) TableName() string {
    return "users"
}
```

### 3.2 字段标签

```go
package main

import (
    "gorm.io/gorm"
    "time"
)

type User struct {
    // 🔥 常用标签
    ID        uint   `gorm:"primaryKey"`                    // 主键
    Name      string `gorm:"size:100;not null"`            // 大小和非空
    Email     string `gorm:"size:100;uniqueIndex"`         // 唯一索引
    Age       int    `gorm:"default:0"`                    // 默认值
    Role      string `gorm:"size:50;default:'user'"`       // 默认值
    Active    bool   `gorm:"default:true"`                 // 布尔默认值
    Score     float64 `gorm:"type:decimal(10,2)"`          // 自定义类型
    Avatar    string `gorm:"size:255;comment:'用户头像'"`   // 注释
    Password  string `gorm:"size:100;not null;-:migration"` // 迁移时忽略
    Token     string `gorm:"-"`                            // 忽略字段
    CreatedAt time.Time
    UpdatedAt time.Time
    DeletedAt gorm.DeletedAt `gorm:"index"`
}
```

### 3.3 自动迁移

```go
package main

import (
    "gorm.io/driver/mysql"
    "gorm.io/gorm"
    "log"
)

func main() {
    dsn := "user:password@tcp(127.0.0.1:3306)/dbname?charset=utf8mb4&parseTime=True&loc=Local"
    db, err := gorm.Open(mysql.Open(dsn), &gorm.Config{})
    if err != nil {
        log.Fatal(err)
    }
    
    // 🔥 自动迁移（创建表）
    err = db.AutoMigrate(&User{}, &Product{})
    if err != nil {
        log.Fatal("迁移失败:", err)
    }
    
    log.Println("迁移成功")
}
```

---

## 4. CRUD操作

### 4.1 创建记录

```go
package main

import (
    "gorm.io/driver/mysql"
    "gorm.io/gorm"
    "log"
)

func main() {
    dsn := "user:password@tcp(127.0.0.1:3306)/dbname?charset=utf8mb4&parseTime=True&loc=Local"
    db, err := gorm.Open(mysql.Open(dsn), &gorm.Config{})
    if err != nil {
        log.Fatal(err)
    }
    
    // 🔥 创建单条记录
    user := User{
        Name:  "张三",
        Email: "zhangsan@example.com",
        Age:   25,
    }
    
    result := db.Create(&user)
    if result.Error != nil {
        log.Fatal("创建失败:", result.Error)
    }
    
    log.Printf("创建成功，ID: %d, 影响行数: %d\n", user.ID, result.RowsAffected)
    
    // 🔥 批量创建
    users := []User{
        {Name: "李四", Email: "lisi@example.com", Age: 30},
        {Name: "王五", Email: "wangwu@example.com", Age: 28},
    }
    
    db.Create(&users)
    
    // 🔥 使用Map创建
    db.Model(&User{}).Create(map[string]interface{}{
        "Name":  "赵六",
        "Email": "zhaoliu@example.com",
        "Age":   35,
    })
    
    // 🔥 批量插入（分批）
    db.CreateInBatches(users, 100) // 每批100条
}
```

### 4.2 查询记录

```go
package main

import (
    "gorm.io/driver/mysql"
    "gorm.io/gorm"
    "log"
)

func main() {
    dsn := "user:password@tcp(127.0.0.1:3306)/dbname?charset=utf8mb4&parseTime=True&loc=Local"
    db, err := gorm.Open(mysql.Open(dsn), &gorm.Config{})
    if err != nil {
        log.Fatal(err)
    }
    
    // 🔥 查询单条记录
    var user User
    
    // 根据主键查询
    db.First(&user, 1)  // SELECT * FROM users WHERE id = 1;
    
    // 根据条件查询第一条
    db.Where("name = ?", "张三").First(&user)
    
    // 🔥 查询所有记录
    var users []User
    db.Find(&users)
    
    // 🔥 条件查询
    // WHERE条件
    db.Where("name = ?", "张三").Find(&users)
    db.Where("name <> ?", "张三").Find(&users)
    db.Where("name IN ?", []string{"张三", "李四"}).Find(&users)
    db.Where("name LIKE ?", "%张%").Find(&users)
    db.Where("age > ?", 25).Find(&users)
    db.Where("age BETWEEN ? AND ?", 20, 30).Find(&users)
    
    // 多个条件
    db.Where("name = ? AND age > ?", "张三", 20).Find(&users)
    
    // Struct条件
    db.Where(&User{Name: "张三", Age: 25}).Find(&users)
    
    // Map条件
    db.Where(map[string]interface{}{"name": "张三", "age": 25}).Find(&users)
    
    // 🔥 排序
    db.Order("age desc").Find(&users)
    db.Order("age desc, name").Find(&users)
    
    // 🔥 限制和偏移
    db.Limit(10).Find(&users)                    // LIMIT 10
    db.Limit(10).Offset(5).Find(&users)          // LIMIT 10 OFFSET 5
    
    // 🔥 分组和聚合
    type Result struct {
        Age   int
        Count int
    }
    var results []Result
    db.Model(&User{}).Select("age, count(*) as count").Group("age").Find(&results)
    
    // 🔥 去重
    db.Distinct("name").Find(&users)
    
    // 🔥 选择字段
    db.Select("name", "age").Find(&users)
    db.Select([]string{"name", "age"}).Find(&users)
    
    // 🔥 Pluck（获取单列）
    var names []string
    db.Model(&User{}).Pluck("name", &names)
    
    // 🔥 Count
    var count int64
    db.Model(&User{}).Where("age > ?", 20).Count(&count)
}
```

### 4.3 更新记录

```go
package main

import (
    "gorm.io/driver/mysql"
    "gorm.io/gorm"
    "log"
)

func main() {
    dsn := "user:password@tcp(127.0.0.1:3306)/dbname?charset=utf8mb4&parseTime=True&loc=Local"
    db, err := gorm.Open(mysql.Open(dsn), &gorm.Config{})
    if err != nil {
        log.Fatal(err)
    }
    
    // 🔥 更新单个字段
    db.Model(&User{}).Where("id = ?", 1).Update("name", "新名字")
    
    // 🔥 更新多个字段（Struct）
    db.Model(&User{}).Where("id = ?", 1).Updates(User{
        Name: "新名字",
        Age:  30,
    })
    
    // 🔥 更新多个字段（Map）
    db.Model(&User{}).Where("id = ?", 1).Updates(map[string]interface{}{
        "name": "新名字",
        "age":  30,
    })
    
    // 🔥 更新选定字段
    db.Model(&User{}).Where("id = ?", 1).Select("name", "age").Updates(User{
        Name: "新名字",
        Age:  30,
    })
    
    // 🔥 批量更新
    db.Model(&User{}).Where("age > ?", 20).Updates(map[string]interface{}{
        "active": true,
    })
    
    // 🔥 Save（保存所有字段）
    var user User
    db.First(&user, 1)
    user.Name = "新名字"
    user.Age = 30
    db.Save(&user)
}
```

### 4.4 删除记录

```go
package main

import (
    "gorm.io/driver/mysql"
    "gorm.io/gorm"
    "log"
)

func main() {
    dsn := "user:password@tcp(127.0.0.1:3306)/dbname?charset=utf8mb4&parseTime=True&loc=Local"
    db, err := gorm.Open(mysql.Open(dsn), &gorm.Config{})
    if err != nil {
        log.Fatal(err)
    }
    
    // 🔥 软删除（DeletedAt不为NULL）
    db.Delete(&User{}, 1)  // UPDATE users SET deleted_at = NOW() WHERE id = 1;
    
    // 🔥 根据条件删除
    db.Where("age < ?", 18).Delete(&User{})
    
    // 🔥 批量删除
    db.Delete(&User{}, []int{1, 2, 3})
    
    // 🔥 永久删除
    db.Unscoped().Delete(&User{}, 1)  // DELETE FROM users WHERE id = 1;
    
    // 🔥 查询包含软删除的记录
    var users []User
    db.Unscoped().Find(&users)
}
```

---

## 5. 查询

### 5.1 高级查询

```go
package main

import (
    "gorm.io/driver/mysql"
    "gorm.io/gorm"
    "log"
)

func main() {
    dsn := "user:password@tcp(127.0.0.1:3306)/dbname?charset=utf8mb4&parseTime=True&loc=Local"
    db, err := gorm.Open(mysql.Open(dsn), &gorm.Config{})
    if err != nil {
        log.Fatal(err)
    }
    
    var users []User
    
    // 🔥 链式查询
    db.Where("age > ?", 20).
        Where("name LIKE ?", "%张%").
        Order("age desc").
        Limit(10).
        Find(&users)
    
    // 🔥 Or条件
    db.Where("name = ?", "张三").Or("name = ?", "李四").Find(&users)
    
    // 🔥 Not条件
    db.Not("name = ?", "张三").Find(&users)
    
    // 🔥 FirstOrInit（找不到则初始化）
    var user User
    db.FirstOrInit(&user, User{Name: "张三"})
    
    // 🔥 FirstOrCreate（找不到则创建）
    db.FirstOrCreate(&user, User{Name: "张三"})
    
    // 🔥 原生SQL
    db.Raw("SELECT * FROM users WHERE age > ?", 20).Scan(&users)
    
    // 🔥 Exec执行SQL
    db.Exec("UPDATE users SET age = age + 1 WHERE id = ?", 1)
    
    // 🔥 子查询
    subQuery := db.Model(&User{}).Select("AVG(age)").Where("active = ?", true)
    db.Where("age > (?)", subQuery).Find(&users)
}
```

---

## 6. 关联

### 6.1 一对一关联

```go
package main

import "gorm.io/gorm"

// 🔥 一对一：用户和个人资料
type User struct {
    gorm.Model
    Name    string
    Profile Profile
}

type Profile struct {
    gorm.Model
    UserID  uint
    Bio     string
    Website string
}

func main() {
    // 创建用户和资料
    user := User{
        Name: "张三",
        Profile: Profile{
            Bio:     "这是我的简介",
            Website: "https://example.com",
        },
    }
    db.Create(&user)
    
    // 🔥 预加载
    var u User
    db.Preload("Profile").First(&u, 1)
}
```

### 6.2 一对多关联

```go
package main

import "gorm.io/gorm"

// 🔥 一对多：用户和文章
type User struct {
    gorm.Model
    Name  string
    Posts []Post
}

type Post struct {
    gorm.Model
    UserID uint
    Title  string
    Content string
}

func main() {
    // 创建用户和文章
    user := User{
        Name: "张三",
        Posts: []Post{
            {Title: "文章1", Content: "内容1"},
            {Title: "文章2", Content: "内容2"},
        },
    }
    db.Create(&user)
    
    // 🔥 预加载
    var u User
    db.Preload("Posts").First(&u, 1)
    
    // 🔥 条件预加载
    db.Preload("Posts", "title LIKE ?", "%Go%").First(&u, 1)
}
```

### 6.3 多对多关联

```go
package main

import "gorm.io/gorm"

// 🔥 多对多：用户和角色
type User struct {
    gorm.Model
    Name  string
    Roles []Role `gorm:"many2many:user_roles;"`
}

type Role struct {
    gorm.Model
    Name  string
    Users []User `gorm:"many2many:user_roles;"`
}

func main() {
    // 创建用户和角色
    user := User{
        Name: "张三",
        Roles: []Role{
            {Name: "管理员"},
            {Name: "编辑"},
        },
    }
    db.Create(&user)
    
    // 🔥 预加载
    var u User
    db.Preload("Roles").First(&u, 1)
    
    // 🔥 添加关联
    var role Role
    db.First(&role, 1)
    db.Model(&u).Association("Roles").Append(&role)
    
    // 🔥 删除关联
    db.Model(&u).Association("Roles").Delete(&role)
    
    // 🔥 清空关联
    db.Model(&u).Association("Roles").Clear()
}
```

---

## 7. 事务

### 7.1 基本事务

```go
package main

import (
    "gorm.io/driver/mysql"
    "gorm.io/gorm"
    "log"
)

func main() {
    dsn := "user:password@tcp(127.0.0.1:3306)/dbname?charset=utf8mb4&parseTime=True&loc=Local"
    db, err := gorm.Open(mysql.Open(dsn), &gorm.Config{})
    if err != nil {
        log.Fatal(err)
    }
    
    // 🔥 手动事务
    tx := db.Begin()
    
    // 创建用户
    user := User{Name: "张三", Email: "zhangsan@example.com"}
    if err := tx.Create(&user).Error; err != nil {
        tx.Rollback()
        log.Fatal(err)
    }
    
    // 创建资料
    profile := Profile{UserID: user.ID, Bio: "简介"}
    if err := tx.Create(&profile).Error; err != nil {
        tx.Rollback()
        log.Fatal(err)
    }
    
    // 提交事务
    tx.Commit()
}
```

### 7.2 事务回调

```go
package main

import (
    "gorm.io/driver/mysql"
    "gorm.io/gorm"
    "log"
)

func main() {
    dsn := "user:password@tcp(127.0.0.1:3306)/dbname?charset=utf8mb4&parseTime=True&loc=Local"
    db, err := gorm.Open(mysql.Open(dsn), &gorm.Config{})
    if err != nil {
        log.Fatal(err)
    }
    
    // 🔥 事务回调
    err = db.Transaction(func(tx *gorm.DB) error {
        // 创建用户
        user := User{Name: "张三", Email: "zhangsan@example.com"}
        if err := tx.Create(&user).Error; err != nil {
            return err
        }
        
        // 创建资料
        profile := Profile{UserID: user.ID, Bio: "简介"}
        if err := tx.Create(&profile).Error; err != nil {
            return err
        }
        
        // 返回nil提交事务
        return nil
    })
    
    if err != nil {
        log.Fatal("事务失败:", err)
    }
}
```

---

## 8. 钩子

### 8.1 创建钩子

```go
package main

import (
    "gorm.io/gorm"
    "log"
)

type User struct {
    gorm.Model
    Name  string
    Email string
}

// 🔥 BeforeCreate钩子
func (u *User) BeforeCreate(tx *gorm.DB) error {
    log.Println("创建前:", u.Name)
    // 可以在这里进行验证、设置默认值等
    return nil
}

// 🔥 AfterCreate钩子
func (u *User) AfterCreate(tx *gorm.DB) error {
    log.Println("创建后:", u.Name)
    // 可以在这里发送通知、记录日志等
    return nil
}
```

### 8.2 更新钩子

```go
package main

import (
    "gorm.io/gorm"
    "log"
)

type User struct {
    gorm.Model
    Name  string
    Email string
}

// 🔥 BeforeUpdate钩子
func (u *User) BeforeUpdate(tx *gorm.DB) error {
    log.Println("更新前:", u.Name)
    return nil
}

// 🔥 AfterUpdate钩子
func (u *User) AfterUpdate(tx *gorm.DB) error {
    log.Println("更新后:", u.Name)
    return nil
}
```

---

## 9. 最佳实践

### 9.1 使用Context

```go
package main

import (
    "context"
    "gorm.io/driver/mysql"
    "gorm.io/gorm"
    "time"
)

func main() {
    dsn := "user:password@tcp(127.0.0.1:3306)/dbname?charset=utf8mb4&parseTime=True&loc=Local"
    db, _ := gorm.Open(mysql.Open(dsn), &gorm.Config{})
    
    // 🔥 使用Context
    ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
    defer cancel()
    
    var users []User
    db.WithContext(ctx).Find(&users)
}
```

### 9.2 错误处理

```go
package main

import (
    "errors"
    "gorm.io/gorm"
    "log"
)

func main() {
    var user User
    
    // 🔥 检查记录是否存在
    result := db.First(&user, 1)
    if errors.Is(result.Error, gorm.ErrRecordNotFound) {
        log.Println("记录不存在")
    }
    
    // 🔥 检查其他错误
    if result.Error != nil {
        log.Fatal("查询失败:", result.Error)
    }
}
```

---

## 📝 学习检查清单

- [ ] 掌握GORM的基本使用
- [ ] 理解模型定义和标签
- [ ] 能够进行CRUD操作
- [ ] 掌握高级查询
- [ ] 理解关联关系
- [ ] 掌握事务处理
- [ ] 了解钩子函数
- [ ] 掌握GORM最佳实践

---

## 🔗 相关资源

- [GORM官方文档](https://gorm.io/zh_CN/docs/)
- [GORM GitHub](https://github.com/go-gorm/gorm)
- [GORM示例](https://github.com/go-gorm/gorm/tree/master/examples)
- [GORM插件](https://gorm.io/zh_CN/docs/write_plugins.html)

---

@author erik.zhou
