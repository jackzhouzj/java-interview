# Goroutine - 完整教程

> @author erik.zhou

## 📚 技术概述

### 版本信息
- **Go版本**：1.21+
- **并发模型**：CSP（Communicating Sequential Processes）
- **调度器**：GMP模型

### 学习难度
- **难度等级**：⭐⭐⭐⭐ (较难)
- **预计学习时间**：10-15小时
- **重要程度**：⭐⭐⭐⭐⭐ (必学)

### 前置知识
- Go语言基础
- 函数和闭包
- 基本的并发概念

## 🎯 学习目标

完成本教程后，你将能够：

- [ ] 理解Goroutine的工作原理
- [ ] 掌握Goroutine的创建和管理
- [ ] 理解GMP调度模型
- [ ] 能够避免常见的并发陷阱
- [ ] 掌握Goroutine的性能优化

## 📖 目录

1. [Goroutine简介](#1-goroutine简介)
2. [创建Goroutine](#2-创建goroutine)
3. [Goroutine调度](#3-goroutine调度)
4. [Goroutine通信](#4-goroutine通信)
5. [并发安全](#5-并发安全)
6. [性能优化](#6-性能优化)
7. [常见陷阱](#7-常见陷阱)
8. [最佳实践](#8-最佳实践)

---

## 1. Goroutine简介

### 1.1 什么是Goroutine

Goroutine是Go语言的轻量级线程，由Go运行时管理。

**核心特点**：
- 🔥 **轻量级**：初始栈只有2KB，可动态增长
- 🔥 **高效调度**：M:N调度模型，用户态调度
- 🔥 **简单易用**：只需go关键字即可创建
- 🔥 **成本低**：可以轻松创建成千上万个Goroutine

### 1.2 Goroutine vs 线程

```go
package main

import (
    "fmt"
    "runtime"
    "time"
)

func main() {
    // 🔥 线程（操作系统线程）
    // - 栈大小：1-2MB
    // - 创建成本：高
    // - 切换成本：高（内核态切换）
    // - 数量限制：受系统资源限制
    
    // 🔥 Goroutine（用户态线程）
    // - 栈大小：2KB起，可动态增长
    // - 创建成本：低
    // - 切换成本：低（用户态切换）
    // - 数量限制：理论上可创建百万级
    
    fmt.Printf("CPU核心数: %d\n", runtime.NumCPU())
    fmt.Printf("Goroutine数量: %d\n", runtime.NumGoroutine())
}
```

---

## 2. 创建Goroutine

### 2.1 基本用法

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    // 🔥 创建Goroutine
    go sayHello()
    
    // 🔥 匿名函数Goroutine
    go func() {
        fmt.Println("匿名函数Goroutine")
    }()
    
    // 🔥 带参数的Goroutine
    go printNumber(42)
    
    // 等待Goroutine执行完成
    time.Sleep(time.Second)
}

func sayHello() {
    fmt.Println("Hello from Goroutine")
}

func printNumber(n int) {
    fmt.Printf("Number: %d\n", n)
}
```

### 2.2 Goroutine闭包陷阱

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    // ❌ 错误：闭包变量问题
    for i := 0; i < 5; i++ {
        go func() {
            fmt.Println(i)  // 可能全部打印5
        }()
    }
    
    // ✅ 正确：传递参数
    for i := 0; i < 5; i++ {
        go func(n int) {
            fmt.Println(n)  // 正确打印0-4
        }(i)
    }
    
    // ✅ 正确：使用局部变量
    for i := 0; i < 5; i++ {
        i := i  // 创建新变量
        go func() {
            fmt.Println(i)  // 正确打印0-4
        }()
    }
    
    time.Sleep(time.Second)
}
```

### 2.3 等待Goroutine完成

```go
package main

import (
    "fmt"
    "sync"
)

func main() {
    // 🔥 使用WaitGroup等待
    var wg sync.WaitGroup
    
    for i := 0; i < 5; i++ {
        wg.Add(1)  // 增加计数
        go func(n int) {
            defer wg.Done()  // 完成时减少计数
            fmt.Printf("Goroutine %d\n", n)
        }(i)
    }
    
    wg.Wait()  // 等待所有Goroutine完成
    fmt.Println("所有Goroutine完成")
}
```

---

## 3. Goroutine调度

### 3.1 GMP模型

```go
package main

import (
    "fmt"
    "runtime"
)

func main() {
    // 🔥 GMP模型
    // G (Goroutine): 用户态线程
    // M (Machine): 操作系统线程
    // P (Processor): 处理器，调度上下文
    
    // 设置最大P数量（默认为CPU核心数）
    runtime.GOMAXPROCS(runtime.NumCPU())
    
    fmt.Printf("GOMAXPROCS: %d\n", runtime.GOMAXPROCS(0))
    fmt.Printf("NumCPU: %d\n", runtime.NumCPU())
    fmt.Printf("NumGoroutine: %d\n", runtime.NumGoroutine())
}
```

### 3.2 Goroutine调度时机

```go
package main

import (
    "fmt"
    "runtime"
    "time"
)

func main() {
    // 🔥 Goroutine调度时机：
    // 1. 主动让出CPU：runtime.Gosched()
    // 2. 系统调用：文件I/O、网络I/O
    // 3. Channel操作：发送/接收阻塞
    // 4. 锁操作：获取锁阻塞
    // 5. 垃圾回收：GC时
    
    go func() {
        for i := 0; i < 5; i++ {
            fmt.Println("Goroutine 1:", i)
            runtime.Gosched()  // 主动让出CPU
        }
    }()
    
    go func() {
        for i := 0; i < 5; i++ {
            fmt.Println("Goroutine 2:", i)
            runtime.Gosched()
        }
    }()
    
    time.Sleep(time.Second)
}
```

### 3.3 Goroutine状态

```go
package main

import (
    "fmt"
    "runtime"
    "time"
)

func main() {
    // 🔥 Goroutine状态：
    // - Runnable: 可运行，等待被调度
    // - Running: 正在运行
    // - Waiting: 等待（I/O、Channel、锁等）
    // - Dead: 已结束
    
    // 查看Goroutine栈信息
    go func() {
        time.Sleep(100 * time.Millisecond)
    }()
    
    time.Sleep(50 * time.Millisecond)
    
    // 打印所有Goroutine的栈信息
    buf := make([]byte, 1<<16)
    n := runtime.Stack(buf, true)
    fmt.Printf("Goroutine栈信息:\n%s\n", buf[:n])
}
```

---

## 4. Goroutine通信

### 4.1 使用Channel通信

```go
package main

import "fmt"

func main() {
    // 🔥 通过Channel通信
    ch := make(chan int)
    
    go func() {
        ch <- 42  // 发送数据
    }()
    
    result := <-ch  // 接收数据
    fmt.Println("接收到:", result)
}
```

### 4.2 使用共享内存通信

```go
package main

import (
    "fmt"
    "sync"
)

func main() {
    // ⚠️ 使用共享内存需要加锁
    var (
        counter int
        mu      sync.Mutex
    )
    
    var wg sync.WaitGroup
    
    for i := 0; i < 1000; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            mu.Lock()
            counter++
            mu.Unlock()
        }()
    }
    
    wg.Wait()
    fmt.Println("Counter:", counter)
}
```

---

## 5. 并发安全

### 5.1 数据竞争

```go
package main

import (
    "fmt"
    "sync"
)

func main() {
    // ❌ 数据竞争示例
    counter := 0
    var wg sync.WaitGroup
    
    for i := 0; i < 1000; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            counter++  // 数据竞争！
        }()
    }
    
    wg.Wait()
    fmt.Println("Counter:", counter)  // 结果不确定
    
    // 🔥 使用 go run -race main.go 检测数据竞争
}
```

### 5.2 避免数据竞争

```go
package main

import (
    "fmt"
    "sync"
    "sync/atomic"
)

func main() {
    // ✅ 方法1：使用Mutex
    var (
        counter1 int
        mu       sync.Mutex
        wg       sync.WaitGroup
    )
    
    for i := 0; i < 1000; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            mu.Lock()
            counter1++
            mu.Unlock()
        }()
    }
    wg.Wait()
    fmt.Println("Counter1:", counter1)
    
    // ✅ 方法2：使用原子操作
    var counter2 int64
    wg = sync.WaitGroup{}
    
    for i := 0; i < 1000; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            atomic.AddInt64(&counter2, 1)
        }()
    }
    wg.Wait()
    fmt.Println("Counter2:", counter2)
    
    // ✅ 方法3：使用Channel
    counter3 := 0
    ch := make(chan int, 1000)
    
    for i := 0; i < 1000; i++ {
        go func() {
            ch <- 1
        }()
    }
    
    for i := 0; i < 1000; i++ {
        counter3 += <-ch
    }
    fmt.Println("Counter3:", counter3)
}
```

---

## 6. 性能优化

### 6.1 控制Goroutine数量

```go
package main

import (
    "fmt"
    "sync"
)

func main() {
    // 🔥 使用Worker Pool限制并发数
    const maxWorkers = 10
    jobs := make(chan int, 100)
    results := make(chan int, 100)
    
    var wg sync.WaitGroup
    
    // 创建Worker Pool
    for w := 0; w < maxWorkers; w++ {
        wg.Add(1)
        go worker(w, jobs, results, &wg)
    }
    
    // 发送任务
    for j := 1; j <= 100; j++ {
        jobs <- j
    }
    close(jobs)
    
    // 等待所有Worker完成
    wg.Wait()
    close(results)
    
    // 收集结果
    for result := range results {
        fmt.Println("Result:", result)
    }
}

func worker(id int, jobs <-chan int, results chan<- int, wg *sync.WaitGroup) {
    defer wg.Done()
    for j := range jobs {
        fmt.Printf("Worker %d processing job %d\n", id, j)
        results <- j * 2
    }
}
```

### 6.2 复用Goroutine

```go
package main

import (
    "fmt"
    "sync"
)

// 🔥 Goroutine池
type GoroutinePool struct {
    workers int
    jobs    chan func()
    wg      sync.WaitGroup
}

func NewGoroutinePool(workers int) *GoroutinePool {
    pool := &GoroutinePool{
        workers: workers,
        jobs:    make(chan func(), workers*2),
    }
    
    // 启动Worker
    for i := 0; i < workers; i++ {
        pool.wg.Add(1)
        go pool.worker()
    }
    
    return pool
}

func (p *GoroutinePool) worker() {
    defer p.wg.Done()
    for job := range p.jobs {
        job()
    }
}

func (p *GoroutinePool) Submit(job func()) {
    p.jobs <- job
}

func (p *GoroutinePool) Close() {
    close(p.jobs)
    p.wg.Wait()
}

func main() {
    pool := NewGoroutinePool(5)
    
    for i := 0; i < 20; i++ {
        i := i
        pool.Submit(func() {
            fmt.Printf("Task %d\n", i)
        })
    }
    
    pool.Close()
}
```

---

## 7. 常见陷阱

### 7.1 Goroutine泄漏

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    // ❌ Goroutine泄漏：永远阻塞
    ch := make(chan int)
    go func() {
        val := <-ch  // 永远等待
        fmt.Println(val)
    }()
    
    // 没有发送数据，Goroutine永远不会结束
    time.Sleep(time.Second)
    
    // ✅ 正确：确保Goroutine能够退出
    ch2 := make(chan int)
    done := make(chan bool)
    
    go func() {
        select {
        case val := <-ch2:
            fmt.Println(val)
        case <-done:
            return  // 收到退出信号
        }
    }()
    
    time.Sleep(100 * time.Millisecond)
    close(done)  // 发送退出信号
}
```

### 7.2 循环变量捕获

```go
package main

import (
    "fmt"
    "sync"
)

func main() {
    var wg sync.WaitGroup
    
    // ❌ 错误：所有Goroutine共享同一个i
    for i := 0; i < 5; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            fmt.Println(i)  // 可能全部打印5
        }()
    }
    wg.Wait()
    
    // ✅ 正确：传递参数
    for i := 0; i < 5; i++ {
        wg.Add(1)
        go func(n int) {
            defer wg.Done()
            fmt.Println(n)
        }(i)
    }
    wg.Wait()
}
```

---

## 8. 最佳实践

### 8.1 使用Context控制Goroutine

```go
package main

import (
    "context"
    "fmt"
    "time"
)

func main() {
    // 🔥 使用Context控制Goroutine生命周期
    ctx, cancel := context.WithTimeout(context.Background(), 2*time.Second)
    defer cancel()
    
    go worker(ctx)
    
    time.Sleep(3 * time.Second)
}

func worker(ctx context.Context) {
    for {
        select {
        case <-ctx.Done():
            fmt.Println("Worker退出:", ctx.Err())
            return
        default:
            fmt.Println("Worker工作中...")
            time.Sleep(500 * time.Millisecond)
        }
    }
}
```

### 8.2 优雅关闭

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

func main() {
    // 🔥 优雅关闭Goroutine
    done := make(chan struct{})
    var wg sync.WaitGroup
    
    wg.Add(1)
    go func() {
        defer wg.Done()
        ticker := time.NewTicker(500 * time.Millisecond)
        defer ticker.Stop()
        
        for {
            select {
            case <-done:
                fmt.Println("收到关闭信号")
                return
            case <-ticker.C:
                fmt.Println("执行任务...")
            }
        }
    }()
    
    time.Sleep(2 * time.Second)
    close(done)  // 发送关闭信号
    wg.Wait()    // 等待Goroutine退出
    fmt.Println("程序退出")
}
```

---

## 📝 学习检查清单

- [ ] 理解Goroutine的工作原理
- [ ] 掌握Goroutine的创建和管理
- [ ] 理解GMP调度模型
- [ ] 能够避免数据竞争
- [ ] 掌握Goroutine性能优化
- [ ] 能够避免Goroutine泄漏
- [ ] 理解并发安全的重要性

---

## 🔗 相关资源

- [Go并发编程](https://go.dev/doc/effective_go#concurrency)
- [Go调度器设计](https://golang.org/s/go11sched)
- [Go内存模型](https://go.dev/ref/mem)
- [并发编程最佳实践](https://go.dev/blog/pipelines)

---

@author erik.zhou
