# Spring WebSocket 完整教程

## 📋 目录
- [技术概述](#技术概述)
- [学习目标](#学习目标)
- [基础概念](#基础概念)
- [核心特性](#核心特性)
- [实战应用](#实战应用)
- [最佳实践](#最佳实践)
- [常见问题](#常见问题)
- [相关资源](#相关资源)
- [学习检查清单](#学习检查清单)

## 📚 技术概述
- **版本**: Spring Framework 6.1.x / Spring Boot 3.2.x
- **官方文档**: https://docs.spring.io/spring-framework/reference/web/websocket.html
- **学习难度**: ⭐⭐⭐⭐ (1-5星)
- **重要程度**: ⭐⭐⭐⭐ (1-5星)
- **前置知识**: 
  - Spring Framework核心知识
  - Spring MVC基础
  - WebSocket协议基础
  - JavaScript/前端基础
  - 消息队列基本概念
- **文档来源**: Context7 + Spring官方文档
- **更新时间**: 2025-01-05
- **作者**: @author erik.zhou

## 🎯 学习目标
- [ ] 理解WebSocket协议和应用场景
- [ ] 掌握Spring WebSocket配置和使用
- [ ] 理解STOMP协议和消息代理
- [ ] 掌握@MessageMapping注解的使用
- [ ] 理解SockJS降级方案
- [ ] 掌握WebSocket安全认证
- [ ] 能够实现实时聊天、推送等功能
- [ ] 理解WebSocket性能优化

## 📖 基础概念

### 1.1 什么是WebSocket

WebSocket是一种在单个TCP连接上进行全双工通信的协议。与HTTP不同，WebSocket允许服务器主动向客户端推送数据，实现真正的实时双向通信。

**WebSocket的特点**：
- 全双工通信：客户端和服务器可以同时发送和接收数据
- 持久连接：建立连接后保持长期连接
- 低延迟：无需频繁建立连接，减少握手开销
- 轻量级：相比HTTP轮询，减少数据传输量
- 跨域支持：支持跨域通信

**WebSocket vs HTTP**：

| 特性 | HTTP | WebSocket |
|------|------|-----------|
| 通信方式 | 请求-响应（单向） | 全双工（双向） |
| 连接 | 短连接 | 长连接 |
| 服务器推送 | 不支持（需轮询） | 原生支持 |
| 开销 | 每次请求都有HTTP头 | 握手后数据帧很小 |
| 实时性 | 差（需轮询） | 优秀 |
| 适用场景 | 普通Web应用 | 实时应用（聊天、推送） |

### 1.2 STOMP协议

STOMP（Simple Text Oriented Messaging Protocol）是一个简单的文本消息协议，为WebSocket提供了更高级的消息传递语义。

**STOMP的优势**：
- 提供消息路由和订阅机制
- 支持消息代理（Broker）
- 简化客户端和服务器的消息处理
- 支持多种消息模式（点对点、发布订阅）
- 与Spring消息抽象无缝集成

**STOMP帧结构**：

```
COMMAND
header1:value1
header2:value2

Body^@
```

**常用STOMP命令**：
- CONNECT：建立连接
- SUBSCRIBE：订阅目的地
- SEND：发送消息
- MESSAGE：接收消息
- DISCONNECT：断开连接

### 1.3 应用场景

- **实时聊天系统**：在线客服、即时通讯
- **实时通知推送**：系统通知、消息提醒
- **协同编辑**：多人在线文档编辑
- **实时数据展示**：股票行情、监控大屏
- **在线游戏**：多人在线游戏
- **物联网**：设备状态实时监控

### 1.4 SockJS降级方案

SockJS是一个JavaScript库，为不支持WebSocket的浏览器提供降级方案，确保应用在各种环境下都能正常工作。

**SockJS传输方式**（按优先级）：
1. WebSocket
2. HTTP Streaming
3. HTTP Long Polling


## 🔥 核心特性

### 2.1 WebSocket配置 🔥

#### 2.1.1 基础配置

**添加依赖**：

```xml
<dependencies>
    <!-- Spring Boot WebSocket Starter -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-websocket</artifactId>
    </dependency>
    
    <!-- Spring Messaging -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-messaging</artifactId>
    </dependency>
</dependencies>
```

**WebSocket配置类**：

```java
import org.springframework.context.annotation.Configuration;
import org.springframework.messaging.simp.config.MessageBrokerRegistry;
import org.springframework.web.socket.config.annotation.EnableWebSocketMessageBroker;
import org.springframework.web.socket.config.annotation.StompEndpointRegistry;
import org.springframework.web.socket.config.annotation.WebSocketMessageBrokerConfigurer;

/**
 * WebSocket配置类
 * 
 * @author erik.zhou
 */
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfiguration implements WebSocketMessageBrokerConfigurer {

    @Override
    public void configureMessageBroker(MessageBrokerRegistry config) {
        // 启用简单消息代理，处理以/topic和/queue开头的消息
        config.enableSimpleBroker("/topic", "/queue");
        // 设置应用目的地前缀，客户端发送消息的目的地前缀
        config.setApplicationDestinationPrefixes("/app");
        // 设置用户目的地前缀（点对点消息）
        config.setUserDestinationPrefix("/user");
    }

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        // 注册STOMP端点，并启用SockJS降级方案
        registry.addEndpoint("/ws")
                .setAllowedOriginPatterns("*")
                .withSockJS();
    }
}
```


#### 2.1.2 配置说明

**消息代理配置**：

```java
@Override
public void configureMessageBroker(MessageBrokerRegistry config) {
    // 1. 启用简单内存消息代理
    config.enableSimpleBroker("/topic", "/queue");
    
    // 2. 配置应用目的地前缀
    config.setApplicationDestinationPrefixes("/app");
    
    // 3. 配置用户目的地前缀（点对点消息）
    config.setUserDestinationPrefix("/user");
}
```

**目的地前缀说明**：
- `/topic`：用于发布-订阅模式（广播消息）
- `/queue`：用于点对点模式（单播消息）
- `/app`：客户端发送消息到服务器的前缀
- `/user`：发送给特定用户的消息前缀

**端点注册配置**：

```java
@Override
public void registerStompEndpoints(StompEndpointRegistry registry) {
    registry.addEndpoint("/ws")
            .setAllowedOriginPatterns("*")  // 允许跨域
            .withSockJS();  // 启用SockJS降级
}
```

### 2.2 消息处理 🔥

#### 2.2.1 @MessageMapping注解

`@MessageMapping`用于处理客户端发送的消息，类似于Spring MVC的`@RequestMapping`。

**基础消息处理**：

```java
import org.springframework.messaging.handler.annotation.MessageMapping;
import org.springframework.messaging.handler.annotation.SendTo;
import org.springframework.stereotype.Controller;

/**
 * WebSocket消息控制器
 * 
 * @author erik.zhou
 */
@Controller
public class ChatController {

    /**
     * 处理客户端发送到/app/chat的消息
     * 并将返回结果广播到/topic/messages
     */
    @MessageMapping("/chat")
    @SendTo("/topic/messages")
    public ChatMessage sendMessage(ChatMessage message) {
        message.setTimestamp(System.currentTimeMillis());
        return message;
    }
}
```

**消息模型**：

```java
public class ChatMessage {
    
    private String content;
    private String sender;
    private MessageType type;
    private Long timestamp;

    public enum MessageType {
        CHAT,
        JOIN,
        LEAVE
    }

    // Getter和Setter方法
    public String getContent() {
        return content;
    }

    public void setContent(String content) {
        this.content = content;
    }

    public String getSender() {
        return sender;
    }

    public void setSender(String sender) {
        this.sender = sender;
    }

    public MessageType getType() {
        return type;
    }

    public void setType(MessageType type) {
        this.type = type;
    }

    public Long getTimestamp() {
        return timestamp;
    }

    public void setTimestamp(Long timestamp) {
        this.timestamp = timestamp;
    }
}
```


#### 2.2.2 使用SimpMessagingTemplate发送消息

`SimpMessagingTemplate`允许在任何地方主动向客户端推送消息。

```java
import org.springframework.messaging.simp.SimpMessagingTemplate;
import org.springframework.stereotype.Service;

/**
 * 消息推送服务
 * 
 * @author erik.zhou
 */
@Service
public class MessageService {

    private final SimpMessagingTemplate messagingTemplate;

    public MessageService(SimpMessagingTemplate messagingTemplate) {
        this.messagingTemplate = messagingTemplate;
    }

    /**
     * 广播消息到所有订阅者
     */
    public void broadcastMessage(ChatMessage message) {
        messagingTemplate.convertAndSend("/topic/messages", message);
    }

    /**
     * 发送消息给特定用户
     */
    public void sendToUser(String username, ChatMessage message) {
        messagingTemplate.convertAndSendToUser(
            username, 
            "/queue/private", 
            message
        );
    }

    /**
     * 发送系统通知
     */
    public void sendNotification(String destination, String content) {
        ChatMessage notification = new ChatMessage();
        notification.setContent(content);
        notification.setType(ChatMessage.MessageType.JOIN);
        notification.setTimestamp(System.currentTimeMillis());
        messagingTemplate.convertAndSend(destination, notification);
    }
}
```

#### 2.2.3 消息参数绑定

```java
import org.springframework.messaging.handler.annotation.DestinationVariable;
import org.springframework.messaging.handler.annotation.Header;
import org.springframework.messaging.handler.annotation.MessageMapping;
import org.springframework.messaging.handler.annotation.Payload;
import org.springframework.messaging.simp.annotation.SendToUser;
import org.springframework.stereotype.Controller;

import java.security.Principal;

@Controller
public class AdvancedChatController {

    /**
     * 使用@Payload绑定消息体
     * 使用Principal获取当前用户
     */
    @MessageMapping("/send")
    @SendTo("/topic/messages")
    public ChatMessage handleMessage(@Payload ChatMessage message, 
                                     Principal principal) {
        message.setSender(principal.getName());
        message.setTimestamp(System.currentTimeMillis());
        return message;
    }

    /**
     * 使用@DestinationVariable绑定路径变量
     */
    @MessageMapping("/chat/{roomId}")
    @SendTo("/topic/room/{roomId}")
    public ChatMessage sendToRoom(@DestinationVariable String roomId,
                                  ChatMessage message) {
        message.setTimestamp(System.currentTimeMillis());
        return message;
    }

    /**
     * 使用@Header获取消息头
     */
    @MessageMapping("/private")
    @SendToUser("/queue/reply")
    public ChatMessage sendPrivateMessage(@Payload ChatMessage message,
                                         @Header("simpSessionId") String sessionId) {
        message.setTimestamp(System.currentTimeMillis());
        return message;
    }
}
```

### 2.3 WebSocket安全认证 🔥 (⚠️ 难点)

#### 2.3.1 基于HTTP Session的认证

WebSocket握手是HTTP请求，可以利用现有的HTTP认证机制。

```java
import org.springframework.context.annotation.Configuration;
import org.springframework.messaging.Message;
import org.springframework.messaging.MessageChannel;
import org.springframework.messaging.simp.config.ChannelRegistration;
import org.springframework.messaging.simp.config.MessageBrokerRegistry;
import org.springframework.messaging.simp.stomp.StompCommand;
import org.springframework.messaging.simp.stomp.StompHeaderAccessor;
import org.springframework.messaging.support.ChannelInterceptor;
import org.springframework.messaging.support.MessageHeaderAccessor;
import org.springframework.web.socket.config.annotation.EnableWebSocketMessageBroker;
import org.springframework.web.socket.config.annotation.StompEndpointRegistry;
import org.springframework.web.socket.config.annotation.WebSocketMessageBrokerConfigurer;

/**
 * WebSocket安全配置
 * 
 * @author erik.zhou
 */
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketSecurityConfiguration implements WebSocketMessageBrokerConfigurer {

    @Override
    public void configureClientInboundChannel(ChannelRegistration registration) {
        registration.interceptors(new ChannelInterceptor() {
            @Override
            public Message<?> preSend(Message<?> message, MessageChannel channel) {
                StompHeaderAccessor accessor = 
                    MessageHeaderAccessor.getAccessor(message, StompHeaderAccessor.class);
                
                if (StompCommand.CONNECT.equals(accessor.getCommand())) {
                    // 从HTTP Session获取认证信息
                    Principal principal = accessor.getUser();
                    if (principal == null) {
                        throw new IllegalArgumentException("未认证的连接");
                    }
                }
                return message;
            }
        });
    }

    @Override
    public void configureMessageBroker(MessageBrokerRegistry config) {
        config.enableSimpleBroker("/topic", "/queue");
        config.setApplicationDestinationPrefixes("/app");
        config.setUserDestinationPrefix("/user");
    }

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry.addEndpoint("/ws")
                .setAllowedOriginPatterns("*")
                .withSockJS();
    }
}
```


#### 2.3.2 基于Token的认证 (⚠️ 难点)

对于无状态应用，可以使用Token进行认证。

```java
import org.springframework.context.annotation.Configuration;
import org.springframework.core.Ordered;
import org.springframework.core.annotation.Order;
import org.springframework.messaging.Message;
import org.springframework.messaging.MessageChannel;
import org.springframework.messaging.simp.config.ChannelRegistration;
import org.springframework.messaging.simp.config.MessageBrokerRegistry;
import org.springframework.messaging.simp.stomp.StompCommand;
import org.springframework.messaging.simp.stomp.StompHeaderAccessor;
import org.springframework.messaging.support.ChannelInterceptor;
import org.springframework.messaging.support.MessageHeaderAccessor;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.web.socket.config.annotation.EnableWebSocketMessageBroker;
import org.springframework.web.socket.config.annotation.StompEndpointRegistry;
import org.springframework.web.socket.config.annotation.WebSocketMessageBrokerConfigurer;

import java.util.ArrayList;
import java.util.List;

/**
 * WebSocket Token认证配置
 * 
 * @author erik.zhou
 */
@Configuration
@Order(Ordered.HIGHEST_PRECEDENCE + 99)
@EnableWebSocketMessageBroker
public class WebSocketTokenAuthConfiguration implements WebSocketMessageBrokerConfigurer {

    @Override
    public void configureClientInboundChannel(ChannelRegistration registration) {
        registration.interceptors(new ChannelInterceptor() {
            @Override
            public Message<?> preSend(Message<?> message, MessageChannel channel) {
                StompHeaderAccessor accessor = 
                    MessageHeaderAccessor.getAccessor(message, StompHeaderAccessor.class);
                
                if (StompCommand.CONNECT.equals(accessor.getCommand())) {
                    // 从消息头获取Token
                    String authToken = accessor.getFirstNativeHeader("Authorization");
                    
                    if (authToken != null && authToken.startsWith("Bearer ")) {
                        String token = authToken.substring(7);
                        // 验证Token并创建认证对象
                        Authentication authentication = validateAndCreateAuthentication(token);
                        accessor.setUser(authentication);
                    } else {
                        throw new IllegalArgumentException("缺少认证Token");
                    }
                }
                return message;
            }
        });
    }

    /**
     * 验证Token并创建认证对象
     */
    private Authentication validateAndCreateAuthentication(String token) {
        // 实际项目中应该验证JWT Token
        // 这里简化处理
        try {
            // 解析Token获取用户信息
            String username = parseTokenToUsername(token);
            List<SimpleGrantedAuthority> authorities = new ArrayList<>();
            authorities.add(new SimpleGrantedAuthority("ROLE_USER"));
            
            return new UsernamePasswordAuthenticationToken(
                username, 
                null, 
                authorities
            );
        } catch (Exception e) {
            throw new IllegalArgumentException("无效的Token", e);
        }
    }

    /**
     * 从Token解析用户名（示例）
     */
    private String parseTokenToUsername(String token) {
        // 实际项目中使用JWT库解析
        return "user_" + token.hashCode();
    }

    @Override
    public void configureMessageBroker(MessageBrokerRegistry registry) {
        // 配置消息代理
    }

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        // 注册端点
    }
}
```

#### 2.3.3 访问会话属性

```java
import org.springframework.messaging.handler.annotation.MessageMapping;
import org.springframework.messaging.simp.SimpMessageHeaderAccessor;
import org.springframework.stereotype.Controller;

import java.util.Map;

/**
 * 会话属性访问示例
 * 
 * @author erik.zhou
 */
@Controller
public class SessionController {

    @MessageMapping("/action")
    public void handleAction(SimpMessageHeaderAccessor headerAccessor) {
        // 获取会话属性
        Map<String, Object> sessionAttributes = headerAccessor.getSessionAttributes();
        
        // 读取属性
        String userId = (String) sessionAttributes.get("userId");
        
        // 设置属性
        sessionAttributes.put("lastAction", System.currentTimeMillis());
    }
}
```

### 2.4 连接事件监听

#### 2.4.1 监听连接和断开事件

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.context.event.EventListener;
import org.springframework.messaging.simp.SimpMessagingTemplate;
import org.springframework.messaging.simp.stomp.StompHeaderAccessor;
import org.springframework.stereotype.Component;
import org.springframework.web.socket.messaging.SessionConnectedEvent;
import org.springframework.web.socket.messaging.SessionDisconnectEvent;
import org.springframework.web.socket.messaging.SessionSubscribeEvent;

/**
 * WebSocket事件监听器
 * 
 * @author erik.zhou
 */
@Component
public class WebSocketEventListener {

    private static final Logger logger = LoggerFactory.getLogger(WebSocketEventListener.class);

    private final SimpMessagingTemplate messagingTemplate;

    public WebSocketEventListener(SimpMessagingTemplate messagingTemplate) {
        this.messagingTemplate = messagingTemplate;
    }

    /**
     * 监听连接建立事件
     */
    @EventListener
    public void handleWebSocketConnectListener(SessionConnectedEvent event) {
        StompHeaderAccessor headerAccessor = StompHeaderAccessor.wrap(event.getMessage());
        String sessionId = headerAccessor.getSessionId();
        logger.info("新的WebSocket连接建立: sessionId={}", sessionId);
    }

    /**
     * 监听连接断开事件
     */
    @EventListener
    public void handleWebSocketDisconnectListener(SessionDisconnectEvent event) {
        StompHeaderAccessor headerAccessor = StompHeaderAccessor.wrap(event.getMessage());
        String username = (String) headerAccessor.getSessionAttributes().get("username");
        
        if (username != null) {
            logger.info("用户断开连接: username={}", username);
            
            // 广播用户离开消息
            ChatMessage leaveMessage = new ChatMessage();
            leaveMessage.setType(ChatMessage.MessageType.LEAVE);
            leaveMessage.setSender(username);
            leaveMessage.setTimestamp(System.currentTimeMillis());
            
            messagingTemplate.convertAndSend("/topic/messages", leaveMessage);
        }
    }

    /**
     * 监听订阅事件
     */
    @EventListener
    public void handleSubscribeEvent(SessionSubscribeEvent event) {
        StompHeaderAccessor headerAccessor = StompHeaderAccessor.wrap(event.getMessage());
        String destination = headerAccessor.getDestination();
        logger.info("用户订阅目的地: destination={}", destination);
    }
}
```


### 2.5 外部消息代理集成

#### 2.5.1 集成RabbitMQ

对于生产环境，建议使用外部消息代理（如RabbitMQ、ActiveMQ）来处理消息。

**添加依赖**：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

**配置RabbitMQ消息代理**：

```java
import org.springframework.context.annotation.Configuration;
import org.springframework.messaging.simp.config.MessageBrokerRegistry;
import org.springframework.web.socket.config.annotation.EnableWebSocketMessageBroker;
import org.springframework.web.socket.config.annotation.StompEndpointRegistry;
import org.springframework.web.socket.config.annotation.WebSocketMessageBrokerConfigurer;

/**
 * RabbitMQ消息代理配置
 * 
 * @author erik.zhou
 */
@Configuration
@EnableWebSocketMessageBroker
public class RabbitMQWebSocketConfiguration implements WebSocketMessageBrokerConfigurer {

    @Override
    public void configureMessageBroker(MessageBrokerRegistry config) {
        // 配置RabbitMQ作为消息代理
        config.enableStompBrokerRelay("/topic", "/queue")
                .setRelayHost("localhost")
                .setRelayPort(61613)
                .setClientLogin("guest")
                .setClientPasscode("guest")
                .setSystemLogin("guest")
                .setSystemPasscode("guest");
        
        config.setApplicationDestinationPrefixes("/app");
        config.setUserDestinationPrefix("/user");
    }

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry.addEndpoint("/ws")
                .setAllowedOriginPatterns("*")
                .withSockJS();
    }
}
```

**配置文件**：

```yaml
spring:
  rabbitmq:
    host: localhost
    port: 5672
    username: guest
    password: guest
```

## 💻 实战应用

### 3.1 实时聊天室应用

#### 3.1.1 后端实现

**聊天控制器**：

```java
import org.springframework.messaging.handler.annotation.MessageMapping;
import org.springframework.messaging.handler.annotation.Payload;
import org.springframework.messaging.handler.annotation.SendTo;
import org.springframework.messaging.simp.SimpMessageHeaderAccessor;
import org.springframework.stereotype.Controller;

/**
 * 聊天室控制器
 * 
 * @author erik.zhou
 */
@Controller
public class ChatRoomController {

    /**
     * 处理聊天消息
     */
    @MessageMapping("/chat.send")
    @SendTo("/topic/public")
    public ChatMessage sendMessage(@Payload ChatMessage chatMessage) {
        chatMessage.setTimestamp(System.currentTimeMillis());
        return chatMessage;
    }

    /**
     * 处理用户加入
     */
    @MessageMapping("/chat.join")
    @SendTo("/topic/public")
    public ChatMessage addUser(@Payload ChatMessage chatMessage,
                               SimpMessageHeaderAccessor headerAccessor) {
        // 在WebSocket会话中添加用户名
        headerAccessor.getSessionAttributes().put("username", chatMessage.getSender());
        chatMessage.setType(ChatMessage.MessageType.JOIN);
        chatMessage.setTimestamp(System.currentTimeMillis());
        return chatMessage;
    }
}
```

**用户管理服务**：

```java
import org.springframework.stereotype.Service;

import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;

/**
 * 在线用户管理服务
 * 
 * @author erik.zhou
 */
@Service
public class OnlineUserService {

    private final Map<String, String> onlineUsers = new ConcurrentHashMap<>();

    /**
     * 用户上线
     */
    public void userOnline(String sessionId, String username) {
        onlineUsers.put(sessionId, username);
    }

    /**
     * 用户下线
     */
    public String userOffline(String sessionId) {
        return onlineUsers.remove(sessionId);
    }

    /**
     * 获取在线用户数
     */
    public int getOnlineUserCount() {
        return onlineUsers.size();
    }

    /**
     * 获取所有在线用户
     */
    public Map<String, String> getAllOnlineUsers() {
        return new ConcurrentHashMap<>(onlineUsers);
    }
}
```

#### 3.1.2 前端实现（JavaScript）

**引入SockJS和STOMP库**：

```html
<!DOCTYPE html>
<html>
<head>
    <title>聊天室</title>
    <script src="https://cdn.jsdelivr.net/npm/sockjs-client@1/dist/sockjs.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/stompjs@2.3.3/lib/stomp.min.js"></script>
</head>
<body>
    <div id="chat-container">
        <div id="message-area"></div>
        <input type="text" id="message-input" placeholder="输入消息...">
        <button onclick="sendMessage()">发送</button>
    </div>

    <script>
        let stompClient = null;
        let username = prompt("请输入用户名:");

        function connect() {
            const socket = new SockJS('/ws');
            stompClient = Stomp.over(socket);

            // 连接到WebSocket服务器
            stompClient.connect({}, function(frame) {
                console.log('Connected: ' + frame);

                // 订阅公共频道
                stompClient.subscribe('/topic/public', function(message) {
                    showMessage(JSON.parse(message.body));
                });

                // 发送加入消息
                stompClient.send("/app/chat.join", {}, JSON.stringify({
                    sender: username,
                    type: 'JOIN'
                }));
            }, function(error) {
                console.error('连接错误:', error);
            });
        }

        function sendMessage() {
            const messageInput = document.getElementById('message-input');
            const messageContent = messageInput.value.trim();

            if (messageContent && stompClient) {
                const chatMessage = {
                    sender: username,
                    content: messageContent,
                    type: 'CHAT'
                };

                stompClient.send("/app/chat.send", {}, JSON.stringify(chatMessage));
                messageInput.value = '';
            }
        }

        function showMessage(message) {
            const messageArea = document.getElementById('message-area');
            const messageElement = document.createElement('div');

            if (message.type === 'JOIN') {
                messageElement.textContent = message.sender + ' 加入了聊天室';
            } else if (message.type === 'LEAVE') {
                messageElement.textContent = message.sender + ' 离开了聊天室';
            } else {
                messageElement.textContent = message.sender + ': ' + message.content;
            }

            messageArea.appendChild(messageElement);
        }

        function disconnect() {
            if (stompClient !== null) {
                stompClient.disconnect();
            }
            console.log("Disconnected");
        }

        // 页面加载时连接
        window.onload = connect;

        // 页面关闭时断开连接
        window.onbeforeunload = disconnect;
    </script>
</body>
</html>
```


### 3.2 实时通知推送系统

#### 3.2.1 通知服务实现

```java
import org.springframework.messaging.simp.SimpMessagingTemplate;
import org.springframework.stereotype.Service;

/**
 * 通知推送服务
 * 
 * @author erik.zhou
 */
@Service
public class NotificationService {

    private final SimpMessagingTemplate messagingTemplate;

    public NotificationService(SimpMessagingTemplate messagingTemplate) {
        this.messagingTemplate = messagingTemplate;
    }

    /**
     * 发送系统通知给所有用户
     */
    public void sendSystemNotification(String message) {
        Notification notification = new Notification();
        notification.setType(NotificationType.SYSTEM);
        notification.setMessage(message);
        notification.setTimestamp(System.currentTimeMillis());
        
        messagingTemplate.convertAndSend("/topic/notifications", notification);
    }

    /**
     * 发送通知给特定用户
     */
    public void sendUserNotification(String username, String message) {
        Notification notification = new Notification();
        notification.setType(NotificationType.PERSONAL);
        notification.setMessage(message);
        notification.setTimestamp(System.currentTimeMillis());
        
        messagingTemplate.convertAndSendToUser(
            username,
            "/queue/notifications",
            notification
        );
    }

    /**
     * 发送订单状态更新通知
     */
    public void sendOrderStatusNotification(String username, Long orderId, String status) {
        Notification notification = new Notification();
        notification.setType(NotificationType.ORDER);
        notification.setMessage("订单 " + orderId + " 状态更新为: " + status);
        notification.setTimestamp(System.currentTimeMillis());
        
        messagingTemplate.convertAndSendToUser(
            username,
            "/queue/orders",
            notification
        );
    }
}

/**
 * 通知消息模型
 */
class Notification {
    private NotificationType type;
    private String message;
    private Long timestamp;

    // Getter和Setter方法
    public NotificationType getType() {
        return type;
    }

    public void setType(NotificationType type) {
        this.type = type;
    }

    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }

    public Long getTimestamp() {
        return timestamp;
    }

    public void setTimestamp(Long timestamp) {
        this.timestamp = timestamp;
    }
}

enum NotificationType {
    SYSTEM,
    PERSONAL,
    ORDER
}
```

#### 3.2.2 REST接口触发推送

```java
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * 通知推送REST接口
 * 
 * @author erik.zhou
 */
@RestController
@RequestMapping("/api/notifications")
public class NotificationController {

    private final NotificationService notificationService;

    public NotificationController(NotificationService notificationService) {
        this.notificationService = notificationService;
    }

    /**
     * 发送系统通知
     */
    @PostMapping("/system")
    public void sendSystemNotification(@RequestBody NotificationRequest request) {
        notificationService.sendSystemNotification(request.getMessage());
    }

    /**
     * 发送用户通知
     */
    @PostMapping("/user")
    public void sendUserNotification(@RequestBody UserNotificationRequest request) {
        notificationService.sendUserNotification(
            request.getUsername(),
            request.getMessage()
        );
    }
}

class NotificationRequest {
    private String message;

    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }
}

class UserNotificationRequest {
    private String username;
    private String message;

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }
}
```

### 3.3 实时数据监控大屏

#### 3.3.1 定时推送数据

```java
import org.springframework.messaging.simp.SimpMessagingTemplate;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Service;

import java.util.Random;

/**
 * 实时数据推送服务
 * 
 * @author erik.zhou
 */
@Service
public class MonitorDataService {

    private final SimpMessagingTemplate messagingTemplate;
    private final Random random = new Random();

    public MonitorDataService(SimpMessagingTemplate messagingTemplate) {
        this.messagingTemplate = messagingTemplate;
    }

    /**
     * 每秒推送系统监控数据
     */
    @Scheduled(fixedRate = 1000)
    public void pushSystemMetrics() {
        SystemMetrics metrics = new SystemMetrics();
        metrics.setCpuUsage(random.nextDouble() * 100);
        metrics.setMemoryUsage(random.nextDouble() * 100);
        metrics.setDiskUsage(random.nextDouble() * 100);
        metrics.setTimestamp(System.currentTimeMillis());
        
        messagingTemplate.convertAndSend("/topic/metrics", metrics);
    }

    /**
     * 每5秒推送业务数据
     */
    @Scheduled(fixedRate = 5000)
    public void pushBusinessData() {
        BusinessData data = new BusinessData();
        data.setOrderCount(random.nextInt(1000));
        data.setRevenue(random.nextDouble() * 100000);
        data.setActiveUsers(random.nextInt(5000));
        data.setTimestamp(System.currentTimeMillis());
        
        messagingTemplate.convertAndSend("/topic/business", data);
    }
}

class SystemMetrics {
    private Double cpuUsage;
    private Double memoryUsage;
    private Double diskUsage;
    private Long timestamp;

    // Getter和Setter方法
    public Double getCpuUsage() {
        return cpuUsage;
    }

    public void setCpuUsage(Double cpuUsage) {
        this.cpuUsage = cpuUsage;
    }

    public Double getMemoryUsage() {
        return memoryUsage;
    }

    public void setMemoryUsage(Double memoryUsage) {
        this.memoryUsage = memoryUsage;
    }

    public Double getDiskUsage() {
        return diskUsage;
    }

    public void setDiskUsage(Double diskUsage) {
        this.diskUsage = diskUsage;
    }

    public Long getTimestamp() {
        return timestamp;
    }

    public void setTimestamp(Long timestamp) {
        this.timestamp = timestamp;
    }
}

class BusinessData {
    private Integer orderCount;
    private Double revenue;
    private Integer activeUsers;
    private Long timestamp;

    // Getter和Setter方法
    public Integer getOrderCount() {
        return orderCount;
    }

    public void setOrderCount(Integer orderCount) {
        this.orderCount = orderCount;
    }

    public Double getRevenue() {
        return revenue;
    }

    public void setRevenue(Double revenue) {
        this.revenue = revenue;
    }

    public Integer getActiveUsers() {
        return activeUsers;
    }

    public void setActiveUsers(Integer activeUsers) {
        this.activeUsers = activeUsers;
    }

    public Long getTimestamp() {
        return timestamp;
    }

    public void setTimestamp(Long timestamp) {
        this.timestamp = timestamp;
    }
}
```

**启用定时任务**：

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.scheduling.annotation.EnableScheduling;

@SpringBootApplication
@EnableScheduling
public class WebSocketApplication {

    public static void main(String[] args) {
        SpringApplication.run(WebSocketApplication.class, args);
    }
}
```


## ✨ 最佳实践

### 4.1 配置优化

#### 4.1.1 连接池配置

```yaml
spring:
  websocket:
    # 消息缓冲区大小
    message-size-limit: 65536
    # 发送超时时间（毫秒）
    send-timeout: 20000
    # 发送缓冲区大小
    send-buffer-size: 524288
    # 接收缓冲区大小
    receive-buffer-size: 524288
```

#### 4.1.2 线程池配置

```java
import org.springframework.context.annotation.Configuration;
import org.springframework.messaging.simp.config.MessageBrokerRegistry;
import org.springframework.scheduling.concurrent.ThreadPoolTaskScheduler;
import org.springframework.web.socket.config.annotation.EnableWebSocketMessageBroker;
import org.springframework.web.socket.config.annotation.StompEndpointRegistry;
import org.springframework.web.socket.config.annotation.WebSocketMessageBrokerConfigurer;
import org.springframework.web.socket.config.annotation.WebSocketTransportRegistration;

/**
 * WebSocket性能优化配置
 * 
 * @author erik.zhou
 */
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketPerformanceConfiguration implements WebSocketMessageBrokerConfigurer {

    @Override
    public void configureMessageBroker(MessageBrokerRegistry config) {
        // 配置心跳
        ThreadPoolTaskScheduler scheduler = new ThreadPoolTaskScheduler();
        scheduler.setPoolSize(10);
        scheduler.setThreadNamePrefix("websocket-heartbeat-");
        scheduler.initialize();
        
        config.enableSimpleBroker("/topic", "/queue")
                .setHeartbeatValue(new long[]{10000, 10000})
                .setTaskScheduler(scheduler);
        
        config.setApplicationDestinationPrefixes("/app");
    }

    @Override
    public void configureWebSocketTransport(WebSocketTransportRegistration registration) {
        // 配置消息大小限制
        registration.setMessageSizeLimit(128 * 1024);  // 128KB
        registration.setSendBufferSizeLimit(512 * 1024);  // 512KB
        registration.setSendTimeLimit(20 * 1000);  // 20秒
    }

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry.addEndpoint("/ws")
                .setAllowedOriginPatterns("*")
                .withSockJS()
                .setStreamBytesLimit(512 * 1024)  // SockJS流字节限制
                .setHttpMessageCacheSize(1000)  // HTTP消息缓存大小
                .setDisconnectDelay(30 * 1000);  // 断开延迟
    }
}
```

### 4.2 异常处理

#### 4.2.1 全局异常处理

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.messaging.handler.annotation.MessageExceptionHandler;
import org.springframework.messaging.simp.annotation.SendToUser;
import org.springframework.web.bind.annotation.ControllerAdvice;

/**
 * WebSocket全局异常处理
 * 
 * @author erik.zhou
 */
@ControllerAdvice
public class WebSocketExceptionHandler {

    private static final Logger logger = LoggerFactory.getLogger(WebSocketExceptionHandler.class);

    /**
     * 处理消息处理异常
     */
    @MessageExceptionHandler
    @SendToUser("/queue/errors")
    public ErrorMessage handleException(Exception exception) {
        logger.error("WebSocket消息处理异常", exception);
        
        ErrorMessage errorMessage = new ErrorMessage();
        errorMessage.setMessage(exception.getMessage());
        errorMessage.setTimestamp(System.currentTimeMillis());
        return errorMessage;
    }

    /**
     * 处理非法参数异常
     */
    @MessageExceptionHandler(IllegalArgumentException.class)
    @SendToUser("/queue/errors")
    public ErrorMessage handleIllegalArgumentException(IllegalArgumentException exception) {
        logger.warn("非法参数: {}", exception.getMessage());
        
        ErrorMessage errorMessage = new ErrorMessage();
        errorMessage.setMessage("参数错误: " + exception.getMessage());
        errorMessage.setTimestamp(System.currentTimeMillis());
        return errorMessage;
    }
}

class ErrorMessage {
    private String message;
    private Long timestamp;

    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }

    public Long getTimestamp() {
        return timestamp;
    }

    public void setTimestamp(Long timestamp) {
        this.timestamp = timestamp;
    }
}
```

### 4.3 安全最佳实践

#### 4.3.1 CSRF保护

```java
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.web.SecurityFilterChain;
import org.springframework.context.annotation.Bean;

/**
 * WebSocket安全配置
 * 
 * @author erik.zhou
 */
@Configuration
@EnableWebSecurity
public class WebSocketSecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf()
                .ignoringRequestMatchers("/ws/**")  // WebSocket端点忽略CSRF
            .and()
            .authorizeHttpRequests()
                .requestMatchers("/ws/**").permitAll()
                .anyRequest().authenticated();
        
        return http.build();
    }
}
```

#### 4.3.2 消息权限控制

```java
import org.springframework.context.annotation.Configuration;
import org.springframework.messaging.simp.SimpMessageType;
import org.springframework.security.config.annotation.web.messaging.MessageSecurityMetadataSourceRegistry;
import org.springframework.security.config.annotation.web.socket.AbstractSecurityWebSocketMessageBrokerConfigurer;

/**
 * WebSocket消息安全配置
 * 
 * @author erik.zhou
 */
@Configuration
public class WebSocketMessageSecurityConfig 
        extends AbstractSecurityWebSocketMessageBrokerConfigurer {

    @Override
    protected void configureInbound(MessageSecurityMetadataSourceRegistry messages) {
        messages
            // 允许任何人连接
            .simpTypeMatchers(SimpMessageType.CONNECT).permitAll()
            // 订阅需要认证
            .simpSubscribeDestMatchers("/user/queue/**").authenticated()
            .simpSubscribeDestMatchers("/topic/**").authenticated()
            // 发送消息需要认证
            .simpDestMatchers("/app/**").authenticated()
            // 其他消息拒绝
            .anyMessage().denyAll();
    }

    @Override
    protected boolean sameOriginDisabled() {
        // 禁用同源策略检查（开发环境）
        return true;
    }
}
```

### 4.4 性能优化建议

#### 4.4.1 消息压缩

```java
import org.springframework.context.annotation.Configuration;
import org.springframework.web.socket.config.annotation.EnableWebSocketMessageBroker;
import org.springframework.web.socket.config.annotation.StompEndpointRegistry;
import org.springframework.web.socket.config.annotation.WebSocketMessageBrokerConfigurer;

@Configuration
@EnableWebSocketMessageBroker
public class WebSocketCompressionConfig implements WebSocketMessageBrokerConfigurer {

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry.addEndpoint("/ws")
                .setAllowedOriginPatterns("*")
                .withSockJS()
                .setSupressCors(true);  // 启用压缩
    }
}
```

#### 4.4.2 消息批量处理

```java
import org.springframework.messaging.simp.SimpMessagingTemplate;
import org.springframework.stereotype.Service;

import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.Executors;
import java.util.concurrent.ScheduledExecutorService;
import java.util.concurrent.TimeUnit;

/**
 * 批量消息推送服务
 * 
 * @author erik.zhou
 */
@Service
public class BatchMessageService {

    private final SimpMessagingTemplate messagingTemplate;
    private final List<ChatMessage> messageBuffer = new ArrayList<>();
    private final ScheduledExecutorService scheduler = Executors.newScheduledThreadPool(1);

    public BatchMessageService(SimpMessagingTemplate messagingTemplate) {
        this.messagingTemplate = messagingTemplate;
        
        // 每秒批量发送一次消息
        scheduler.scheduleAtFixedRate(this::flushMessages, 1, 1, TimeUnit.SECONDS);
    }

    /**
     * 添加消息到缓冲区
     */
    public synchronized void addMessage(ChatMessage message) {
        messageBuffer.add(message);
        
        // 如果缓冲区达到阈值，立即发送
        if (messageBuffer.size() >= 100) {
            flushMessages();
        }
    }

    /**
     * 批量发送消息
     */
    private synchronized void flushMessages() {
        if (!messageBuffer.isEmpty()) {
            messagingTemplate.convertAndSend("/topic/messages/batch", new ArrayList<>(messageBuffer));
            messageBuffer.clear();
        }
    }
}
```

### 4.5 监控和日志

#### 4.5.1 连接监控

```java
import org.springframework.stereotype.Component;

import java.util.concurrent.atomic.AtomicInteger;

/**
 * WebSocket连接监控
 * 
 * @author erik.zhou
 */
@Component
public class WebSocketConnectionMonitor {

    private final AtomicInteger activeConnections = new AtomicInteger(0);
    private final AtomicInteger totalConnections = new AtomicInteger(0);

    /**
     * 连接建立
     */
    public void onConnect() {
        activeConnections.incrementAndGet();
        totalConnections.incrementAndGet();
    }

    /**
     * 连接断开
     */
    public void onDisconnect() {
        activeConnections.decrementAndGet();
    }

    /**
     * 获取活跃连接数
     */
    public int getActiveConnections() {
        return activeConnections.get();
    }

    /**
     * 获取总连接数
     */
    public int getTotalConnections() {
        return totalConnections.get();
    }
}
```

#### 4.5.2 消息日志记录

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.context.annotation.Configuration;
import org.springframework.messaging.Message;
import org.springframework.messaging.MessageChannel;
import org.springframework.messaging.simp.config.ChannelRegistration;
import org.springframework.messaging.support.ChannelInterceptor;
import org.springframework.web.socket.config.annotation.EnableWebSocketMessageBroker;
import org.springframework.web.socket.config.annotation.WebSocketMessageBrokerConfigurer;

/**
 * WebSocket消息日志配置
 * 
 * @author erik.zhou
 */
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketLoggingConfig implements WebSocketMessageBrokerConfigurer {

    private static final Logger logger = LoggerFactory.getLogger(WebSocketLoggingConfig.class);

    @Override
    public void configureClientInboundChannel(ChannelRegistration registration) {
        registration.interceptors(new ChannelInterceptor() {
            @Override
            public Message<?> preSend(Message<?> message, MessageChannel channel) {
                logger.debug("接收到消息: {}", message);
                return message;
            }
        });
    }

    @Override
    public void configureClientOutboundChannel(ChannelRegistration registration) {
        registration.interceptors(new ChannelInterceptor() {
            @Override
            public Message<?> preSend(Message<?> message, MessageChannel channel) {
                logger.debug("发送消息: {}", message);
                return message;
            }
        });
    }
}
```


## ❓ 常见问题

### Q1: WebSocket和HTTP轮询的区别？

**A**: 
- **WebSocket**：建立持久连接，服务器可主动推送，低延迟，适合实时应用
- **HTTP轮询**：客户端定时请求，服务器被动响应，延迟高，资源消耗大
- **长轮询**：客户端请求后服务器保持连接直到有数据，比短轮询好但仍不如WebSocket

### Q2: 如何处理WebSocket连接断开重连？

**A**: 
客户端实现自动重连机制：

```javascript
let reconnectAttempts = 0;
const maxReconnectAttempts = 5;

function connect() {
    const socket = new SockJS('/ws');
    stompClient = Stomp.over(socket);

    stompClient.connect({}, function(frame) {
        console.log('Connected');
        reconnectAttempts = 0;  // 重置重连次数
    }, function(error) {
        console.error('连接失败:', error);
        
        // 自动重连
        if (reconnectAttempts < maxReconnectAttempts) {
            reconnectAttempts++;
            setTimeout(connect, 5000);  // 5秒后重连
        }
    });
}
```

### Q3: 如何实现点对点消息？

**A**: 
使用`convertAndSendToUser`方法：

```java
// 服务端
messagingTemplate.convertAndSendToUser(
    "username",  // 目标用户
    "/queue/private",  // 目的地
    message
);

// 客户端订阅
stompClient.subscribe('/user/queue/private', function(message) {
    console.log('收到私信:', message.body);
});
```

### Q4: 如何限制WebSocket连接数？

**A**: 
使用拦截器限制连接：

```java
@Component
public class ConnectionLimitInterceptor implements ChannelInterceptor {
    
    private final AtomicInteger connectionCount = new AtomicInteger(0);
    private static final int MAX_CONNECTIONS = 1000;

    @Override
    public Message<?> preSend(Message<?> message, MessageChannel channel) {
        StompHeaderAccessor accessor = 
            MessageHeaderAccessor.getAccessor(message, StompHeaderAccessor.class);
        
        if (StompCommand.CONNECT.equals(accessor.getCommand())) {
            if (connectionCount.get() >= MAX_CONNECTIONS) {
                throw new IllegalStateException("连接数已达上限");
            }
            connectionCount.incrementAndGet();
        } else if (StompCommand.DISCONNECT.equals(accessor.getCommand())) {
            connectionCount.decrementAndGet();
        }
        
        return message;
    }
}
```

### Q5: 如何测试WebSocket？

**A**: 
使用Spring Boot Test进行测试：

```java
import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.web.server.LocalServerPort;
import org.springframework.messaging.converter.MappingJackson2MessageConverter;
import org.springframework.messaging.simp.stomp.StompSession;
import org.springframework.messaging.simp.stomp.StompSessionHandlerAdapter;
import org.springframework.web.socket.client.standard.StandardWebSocketClient;
import org.springframework.web.socket.messaging.WebSocketStompClient;

import java.util.concurrent.TimeUnit;

@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class WebSocketTest {

    @LocalServerPort
    private int port;

    @Test
    public void testWebSocketConnection() throws Exception {
        WebSocketStompClient stompClient = new WebSocketStompClient(
            new StandardWebSocketClient()
        );
        stompClient.setMessageConverter(new MappingJackson2MessageConverter());

        String url = "ws://localhost:" + port + "/ws";
        StompSession session = stompClient.connect(
            url,
            new StompSessionHandlerAdapter() {}
        ).get(1, TimeUnit.SECONDS);

        // 测试订阅和发送消息
        session.subscribe("/topic/messages", new StompSessionHandlerAdapter() {});
        session.send("/app/chat", new ChatMessage());
    }
}
```

### Q6: SockJS和原生WebSocket如何选择？

**A**: 
- **原生WebSocket**：性能更好，适合现代浏览器
- **SockJS**：提供降级方案，兼容性更好，适合需要支持老旧浏览器的场景
- **建议**：生产环境使用SockJS，确保最大兼容性

### Q7: 如何处理大量并发连接？

**A**: 
1. 使用外部消息代理（RabbitMQ、Redis）
2. 配置合理的线程池大小
3. 启用消息压缩
4. 实现消息批量处理
5. 使用负载均衡和集群部署
6. 限制单个连接的消息频率

### Q8: WebSocket如何实现集群部署？

**A**: 
使用外部消息代理实现集群：

```java
@Configuration
@EnableWebSocketMessageBroker
public class ClusterWebSocketConfig implements WebSocketMessageBrokerConfigurer {

    @Override
    public void configureMessageBroker(MessageBrokerRegistry config) {
        // 使用Redis作为消息代理
        config.enableStompBrokerRelay("/topic", "/queue")
                .setRelayHost("redis-server")
                .setRelayPort(6379);
    }
}
```

## 🔗 相关资源

### 官方文档
- [Spring WebSocket官方文档](https://docs.spring.io/spring-framework/reference/web/websocket.html)
- [STOMP协议规范](https://stomp.github.io/)
- [SockJS官网](https://github.com/sockjs/sockjs-client)
- [Spring Messaging文档](https://docs.spring.io/spring-framework/reference/web/websocket/stomp.html)

### 推荐文章
- [WebSocket协议详解](https://datatracker.ietf.org/doc/html/rfc6455)
- [Spring WebSocket最佳实践](https://www.baeldung.com/websockets-spring)
- [STOMP over WebSocket](https://www.baeldung.com/spring-websockets-stomp)

### 视频教程
- [Spring WebSocket实战教程](https://spring.io/guides/gs/messaging-stomp-websocket/)
- [构建实时聊天应用](https://www.youtube.com/springdevelopers)

### 推荐书籍
- 《Spring实战》（第6版）- WebSocket章节
- 《WebSocket权威指南》
- 《Spring微服务实战》

### 相关技术
- [Spring MVC](./03-Spring-MVC-完整教程.md) - Web开发基础
- [Spring Security](https://spring.io/projects/spring-security) - 安全认证
- [RabbitMQ](../../05-消息队列/RabbitMQ-完整教程.md) - 消息代理
- [Redis](../../04-数据库/Redis-完整教程.md) - 缓存和消息

## 📝 学习检查清单

### 基础知识
- [ ] 理解WebSocket协议和工作原理
- [ ] 掌握STOMP协议基础
- [ ] 理解SockJS降级方案
- [ ] 了解WebSocket应用场景
- [ ] 理解消息代理的作用

### 核心特性
- [ ] 掌握Spring WebSocket配置
- [ ] 理解@MessageMapping注解使用
- [ ] 掌握@SendTo和@SendToUser
- [ ] 理解消息路由机制
- [ ] 掌握SimpMessagingTemplate使用
- [ ] 理解连接事件监听

### 安全认证
- [ ] 掌握基于HTTP Session的认证
- [ ] 理解基于Token的认证
- [ ] 掌握消息权限控制
- [ ] 理解CSRF保护
- [ ] 掌握会话属性访问

### 实战能力
- [ ] 能够实现实时聊天室
- [ ] 能够实现通知推送系统
- [ ] 能够实现实时数据监控
- [ ] 能够处理连接断开重连
- [ ] 能够实现点对点消息
- [ ] 能够集成外部消息代理

### 最佳实践
- [ ] 掌握性能优化配置
- [ ] 理解异常处理机制
- [ ] 掌握安全最佳实践
- [ ] 理解监控和日志记录
- [ ] 掌握集群部署方案
- [ ] 理解消息批量处理

---

**学习建议**：
1. 先理解WebSocket和STOMP协议基础
2. 掌握Spring WebSocket基本配置
3. 实践简单的聊天室应用
4. 学习安全认证机制（重点）
5. 掌握性能优化技巧
6. 在实际项目中应用所学知识

**预计学习时长**: 20-30小时（基础学习）+ 40-60小时（进阶学习）

**下一步学习**：
- [Spring Security](https://spring.io/projects/spring-security)：深入学习安全认证
- [RabbitMQ](../../05-消息队列/RabbitMQ-完整教程.md)：学习消息队列
- [Redis](../../04-数据库/Redis-完整教程.md)：学习缓存和发布订阅
- [Spring Cloud](./06-Spring-Cloud-完整教程.md)：学习微服务架构

**实战项目推荐**：
1. 在线客服系统
2. 实时协同编辑工具
3. 股票行情监控系统
4. 多人在线游戏
5. IoT设备监控平台
