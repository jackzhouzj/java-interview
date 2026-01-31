# Netty 源码解析 - 完整教程

> 深入理解 Netty 的 Reactor 模型和高性能网络编程
> 
> @author erik.zhou

## 📚 技术概述

| 项目 | 说明 |
|------|------|
| **框架名称** | Netty |
| **当前版本** | 4.1.x |
| **源码地址** | https://github.com/netty/netty |
| **学习难度** | ⭐⭐⭐⭐⭐ |
| **重要程度** | ⭐⭐⭐⭐ |
| **预计时长** | 35-50 小时 |
| **前置知识** | Java NIO、多线程、网络编程 |

## 🎯 学习目标

- [ ] 理解 Netty 的整体架构
- [ ] 掌握 Reactor 线程模型
- [ ] 深入理解 EventLoop 事件循环
- [ ] 掌握 Channel 和 Pipeline 机制
- [ ] 理解 ByteBuf 内存管理
- [ ] 掌握编解码器的实现
- [ ] 能够使用 Netty 开发高性能网络应用

## 📖 目录

1. [Netty 整体架构](#1-netty-整体架构)
2. [Reactor 线程模型](#2-reactor-线程模型)
3. [EventLoop 事件循环](#3-eventloop-事件循环)
4. [Channel 和 Pipeline](#4-channel-和-pipeline)
5. [ByteBuf 内存管理](#5-bytebuf-内存管理)
6. [编解码器实现](#6-编解码器实现)

---

## 1. Netty 整体架构

### 1.1 核心组件

```
netty/
├── Bootstrap/ServerBootstrap  # 启动引导类
├── EventLoopGroup             # 事件循环组
├── EventLoop                  # 事件循环
├── Channel                    # 网络通道
├── ChannelPipeline            # 处理器链
├── ChannelHandler             # 处理器
├── ByteBuf                    # 字节缓冲区
└── ChannelFuture              # 异步结果
```

### 1.2 Netty 服务端示例

```java
public class NettyServer {
    public static void main(String[] args) throws Exception {
        // 1. 创建 BossGroup 和 WorkerGroup
        EventLoopGroup bossGroup = new NioEventLoopGroup(1);
        EventLoopGroup workerGroup = new NioEventLoopGroup();
        
        try {
            // 2. 创建服务端启动引导类
            ServerBootstrap bootstrap = new ServerBootstrap();
            bootstrap.group(bossGroup, workerGroup)
                .channel(NioServerSocketChannel.class)
                .option(ChannelOption.SO_BACKLOG, 128)
                .childOption(ChannelOption.SO_KEEPALIVE, true)
                .childHandler(new ChannelInitializer<SocketChannel>() {
                    @Override
                    protected void initChannel(SocketChannel ch) {
                        ChannelPipeline pipeline = ch.pipeline();
                        pipeline.addLast(new StringDecoder());
                        pipeline.addLast(new StringEncoder());
                        pipeline.addLast(new ServerHandler());
                    }
                });
            
            // 3. 绑定端口，启动服务
            ChannelFuture future = bootstrap.bind(8080).sync();
            System.out.println("服务器启动成功，监听端口：8080");
            
            // 4. 等待服务端监听端口关闭
            future.channel().closeFuture().sync();
        } finally {
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }
    }
}

// 自定义处理器
class ServerHandler extends ChannelInboundHandlerAdapter {
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) {
        System.out.println("收到消息：" + msg);
        ctx.writeAndFlush("服务器收到：" + msg);
    }
    
    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) {
        cause.printStackTrace();
        ctx.close();
    }
}
```

---

## 2. Reactor 线程模型 🔥

### 2.1 三种 Reactor 模型

**单 Reactor 单线程**
```
Reactor (单线程)
  ↓
Accept + Read + Decode + Process + Encode + Send
```
- 优点：简单
- 缺点：性能差，无法利用多核

**单 Reactor 多线程**
```
Reactor (单线程)
  ↓
Accept + Read + Decode
  ↓
Thread Pool (Process)
  ↓
Encode + Send
```
- 优点：充分利用多核
- 缺点：Reactor 单线程成为瓶颈

**主从 Reactor 多线程（Netty 使用）** 🔥
```
Main Reactor (BossGroup)
  ↓
Accept
  ↓
Sub Reactor (WorkerGroup)
  ↓
Read + Decode + Process + Encode + Send
```
- 优点：高性能、高并发
- 缺点：实现复杂

### 2.2 Netty 的 Reactor 实现

```java
// 主从 Reactor 模型
EventLoopGroup bossGroup = new NioEventLoopGroup(1);      // 主 Reactor
EventLoopGroup workerGroup = new NioEventLoopGroup(8);    // 从 Reactor

ServerBootstrap bootstrap = new ServerBootstrap();
bootstrap.group(bossGroup, workerGroup)  // 设置主从 Reactor
    .channel(NioServerSocketChannel.class);

// BossGroup 职责：
// 1. 接收客户端连接（Accept）
// 2. 将连接注册到 WorkerGroup

// WorkerGroup 职责：
// 1. 处理 I/O 读写
// 2. 执行业务逻辑
// 3. 编解码
```

### 2.3 EventLoopGroup 创建

```java
// NioEventLoopGroup 构造器
public NioEventLoopGroup(int nThreads) {
    this(nThreads, (Executor) null);
}

public NioEventLoopGroup(int nThreads, Executor executor) {
    this(nThreads, executor, SelectorProvider.provider());
}

public NioEventLoopGroup(int nThreads, Executor executor, 
        final SelectorProvider selectorProvider) {
    this(nThreads, executor, selectorProvider, 
        DefaultSelectStrategyFactory.INSTANCE);
}

// 最终调用父类构造器
protected MultithreadEventLoopGroup(int nThreads, Executor executor, 
        Object... args) {
    super(nThreads == 0 ? DEFAULT_EVENT_LOOP_THREADS : nThreads, 
        executor, args);
}

// 创建 EventLoop 数组
protected MultithreadEventExecutorGroup(int nThreads, 
        Executor executor, Object... args) {
    // 创建 EventLoop 数组
    children = new EventExecutor[nThreads];
    
    for (int i = 0; i < nThreads; i++) {
        boolean success = false;
        try {
            // 创建 NioEventLoop
            children[i] = newChild(executor, args);
            success = true;
        } catch (Exception e) {
            throw new IllegalStateException("failed to create a child event loop", e);
        } finally {
            if (!success) {
                // 创建失败，关闭已创建的 EventLoop
                for (int j = 0; j < i; j++) {
                    children[j].shutdownGracefully();
                }
            }
        }
    }
    
    // 创建 EventLoop 选择器（轮询策略）
    chooser = chooserFactory.newChooser(children);
}
```

---

## 3. EventLoop 事件循环 🔥

### 3.1 EventLoop 核心概念

```java
// EventLoop 继承关系
EventLoop extends EventLoopGroup, EventExecutor

// EventLoop 特点：
// 1. 一个 EventLoop 对应一个线程
// 2. 一个 EventLoop 可以处理多个 Channel
// 3. 一个 Channel 只能注册到一个 EventLoop
// 4. EventLoop 负责处理 I/O 事件和任务
```

### 3.2 EventLoop 运行流程

```java
// NioEventLoop.run() - 事件循环主逻辑
@Override
protected void run() {
    int selectCnt = 0;
    for (;;) {
        try {
            int strategy;
            try {
                // 1. 选择策略（是否需要 select）
                strategy = selectStrategy.calculateStrategy(selectNowSupplier, hasTasks());
                switch (strategy) {
                case SelectStrategy.CONTINUE:
                    continue;
                case SelectStrategy.BUSY_WAIT:
                    // NIO 不支持 busy-wait
                case SelectStrategy.SELECT:
                    // 执行 select 操作
                    long curDeadlineNanos = nextScheduledTaskDeadlineNanos();
                    if (curDeadlineNanos == -1L) {
                        curDeadlineNanos = NONE;
                    }
                    nextWakeupNanos.set(curDeadlineNanos);
                    try {
                        if (!hasTasks()) {
                            strategy = select(curDeadlineNanos);
                        }
                    } finally {
                        nextWakeupNanos.lazySet(AWAKE);
                    }
                default:
                }
            } catch (IOException e) {
                rebuildSelector0();
                selectCnt = 0;
                handleLoopException(e);
                continue;
            }
            
            selectCnt++;
            cancelledKeys = 0;
            needsToSelectAgain = false;
            final int ioRatio = this.ioRatio;
            boolean ranTasks;
            if (ioRatio == 100) {
                try {
                    if (strategy > 0) {
                        // 2. 处理 I/O 事件
                        processSelectedKeys();
                    }
                } finally {
                    // 3. 执行所有任务
                    ranTasks = runAllTasks();
                }
            } else if (strategy > 0) {
                final long ioStartTime = System.nanoTime();
                try {
                    // 2. 处理 I/O 事件
                    processSelectedKeys();
                } finally {
                    // 3. 根据 ioRatio 执行任务
                    final long ioTime = System.nanoTime() - ioStartTime;
                    ranTasks = runAllTasks(ioTime * (100 - ioRatio) / ioRatio);
                }
            } else {
                // 3. 执行最少数量的任务
                ranTasks = runAllTasks(0);
            }
            
            // 4. 检查是否需要重新 select
            if (ranTasks || strategy > 0) {
                if (selectCnt > MIN_PREMATURE_SELECTOR_RETURNS && logger.isDebugEnabled()) {
                    logger.debug("Selector.select() returned prematurely {} times in a row for Selector {}.",
                            selectCnt - 1, selector);
                }
                selectCnt = 0;
            } else if (unexpectedSelectorWakeup(selectCnt)) {
                selectCnt = 0;
            }
        } catch (CancelledKeyException e) {
            // Harmless exception - log anyway
            if (logger.isDebugEnabled()) {
                logger.debug(CancelledKeyException.class.getSimpleName() + " raised by a Selector {} - JDK bug?",
                        selector, e);
            }
        } catch (Error e) {
            throw e;
        } catch (Throwable t) {
            handleLoopException(t);
        } finally {
            // Always handle shutdown even if the loop processing threw an exception.
            try {
                if (isShuttingDown()) {
                    closeAll();
                    if (confirmShutdown()) {
                        return;
                    }
                }
            } catch (Error e) {
                throw e;
            } catch (Throwable t) {
                handleLoopException(t);
            }
        }
    }
}
```

### 3.3 处理 I/O 事件

```java
// 处理选中的 Key
private void processSelectedKeys() {
    if (selectedKeys != null) {
        // 优化的 SelectedKeys 处理
        processSelectedKeysOptimized();
    } else {
        // 普通的 SelectedKeys 处理
        processSelectedKeysPlain(selector.selectedKeys());
    }
}

// 优化的处理方式
private void processSelectedKeysOptimized() {
    for (int i = 0; i < selectedKeys.size; ++i) {
        final SelectionKey k = selectedKeys.keys[i];
        selectedKeys.keys[i] = null;
        
        final Object a = k.attachment();
        
        if (a instanceof AbstractNioChannel) {
            // 处理 Channel 的 I/O 事件
            processSelectedKey(k, (AbstractNioChannel) a);
        } else {
            @SuppressWarnings("unchecked")
            NioTask<SelectableChannel> task = (NioTask<SelectableChannel>) a;
            processSelectedKey(k, task);
        }
        
        if (needsToSelectAgain) {
            selectedKeys.reset(i + 1);
            selectAgain();
            i = -1;
        }
    }
}

// 处理单个 Channel 的事件
private void processSelectedKey(SelectionKey k, AbstractNioChannel ch) {
    final AbstractNioChannel.NioUnsafe unsafe = ch.unsafe();
    
    try {
        int readyOps = k.readyOps();
        
        // 处理 OP_CONNECT 事件
        if ((readyOps & SelectionKey.OP_CONNECT) != 0) {
            int ops = k.interestOps();
            ops &= ~SelectionKey.OP_CONNECT;
            k.interestOps(ops);
            unsafe.finishConnect();
        }
        
        // 处理 OP_WRITE 事件
        if ((readyOps & SelectionKey.OP_WRITE) != 0) {
            ch.unsafe().forceFlush();
        }
        
        // 处理 OP_READ 或 OP_ACCEPT 事件
        if ((readyOps & (SelectionKey.OP_READ | SelectionKey.OP_ACCEPT)) != 0 || readyOps == 0) {
            unsafe.read();
        }
    } catch (CancelledKeyException ignored) {
        unsafe.close(unsafe.voidPromise());
    }
}
```

---

## 4. Channel 和 Pipeline 🔥

### 4.1 Channel 核心概念

```java
// Channel 接口
public interface Channel extends AttributeMap, ChannelOutboundInvoker, Comparable<Channel> {
    EventLoop eventLoop();              // 获取 EventLoop
    ChannelPipeline pipeline();         // 获取 Pipeline
    ChannelConfig config();             // 获取配置
    boolean isOpen();                   // 是否打开
    boolean isActive();                 // 是否激活
    ChannelMetadata metadata();         // 获取元数据
    SocketAddress localAddress();       // 本地地址
    SocketAddress remoteAddress();      // 远程地址
}

// Channel 生命周期
channelRegistered    → Channel 注册到 EventLoop
channelActive        → Channel 激活（连接建立）
channelRead          → Channel 读取数据
channelReadComplete  → Channel 读取完成
channelInactive      → Channel 失活（连接断开）
channelUnregistered  → Channel 从 EventLoop 注销
```

