# LangChain4j 完整教程

> **作者**: erik.zhou  
> **创建时间**: 2025-01-31  
> **技术栈**: LangChain4j 0.36+, Spring Boot 3.x, OpenAI/Azure OpenAI

## 📋 目录

- [1. LangChain4j简介](#1-langchain4j简介)
- [2. 核心概念](#2-核心概念)
- [3. 快速开始](#3-快速开始)
- [4. 核心功能详解](#4-核心功能详解)
- [5. 高级特性](#5-高级特性)
- [6. 实战案例](#6-实战案例)
- [7. 最佳实践](#7-最佳实践)
- [8. 性能优化](#8-性能优化)

---

## 1. LangChain4j简介

### 1.1 什么是LangChain4j

LangChain4j是LangChain的Java实现，专为Java开发者设计的大语言模型(LLM)应用开发框架。

**核心特性**:
- 🚀 原生Java实现，无需Python依赖
- 🔌 支持多种LLM提供商(OpenAI、Azure、Hugging Face等)
- 🧠 内置RAG(检索增强生成)支持
- 💾 多种向量数据库集成
- 🔄 流式响应支持
- 🛠️ Spring Boot无缝集成

### 1.2 为什么选择LangChain4j

**对比Python版LangChain的优势**:

| 特性 | LangChain4j | LangChain(Python) |
|------|-------------|-------------------|
| 类型安全 | ✅ 编译时检查 | ❌ 运行时检查 |
| 性能 | ✅ JVM优化 | ⚠️ 解释执行 |
| 企业集成 | ✅ Spring生态 | ⚠️ 需要额外工作 |
| 部署 | ✅ JAR/容器化 | ⚠️ 依赖管理复杂 |
| 并发 | ✅ 多线程优势 | ⚠️ GIL限制 |

### 1.3 应用场景

- **智能客服**: 基于知识库的问答系统
- **文档分析**: PDF/Word文档智能解析
- **代码助手**: 代码生成和解释
- **数据分析**: 自然语言查询数据库
- **内容生成**: 营销文案、报告生成

---

## 2. 核心概念

### 2.1 架构图

```
┌─────────────────────────────────────────────────────────┐
│                    Application Layer                     │
│              (Spring Boot / Java Application)            │
└────────────────────┬────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────┐
│                   LangChain4j Core                       │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌─────────┐ │
│  │  Chain   │  │ Memory   │  │  Agent   │  │  Tools  │ │
│  └──────────┘  └──────────┘  └──────────┘  └─────────┘ │
└────────────────────┬────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────┐
│                  Integration Layer                       │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌─────────┐ │
│  │   LLM    │  │ Embedding│  │  Vector  │  │Document │ │
│  │ Provider │  │  Model   │  │   Store  │  │ Loader  │ │
│  └──────────┘  └──────────┘  └──────────┘  └─────────┘ │
└─────────────────────────────────────────────────────────┘
```

### 2.2 核心组件

#### 2.2.1 Language Model (语言模型)
```java
// 与LLM交互的接口
ChatLanguageModel model = OpenAiChatModel.builder()
    .apiKey(System.getenv("OPENAI_API_KEY"))
    .modelName("gpt-4")
    .build();
```

#### 2.2.2 Embedding Model (嵌入模型)
```java
// 将文本转换为向量
EmbeddingModel embeddingModel = OpenAiEmbeddingModel.builder()
    .apiKey(System.getenv("OPENAI_API_KEY"))
    .modelName("text-embedding-ada-002")
    .build();
```

#### 2.2.3 Vector Store (向量存储)
```java
// 存储和检索向量
EmbeddingStore<TextSegment> embeddingStore = 
    new InMemoryEmbeddingStore<>();
```

#### 2.2.4 Document Loader (文档加载器)
```java
// 加载各种格式的文档
Document document = FileSystemDocumentLoader
    .loadDocument("/path/to/document.pdf");
```

---

## 3. 快速开始

### 3.1 Maven依赖

```xml
<dependencies>
    <!-- LangChain4j核心 -->
    <dependency>
        <groupId>dev.langchain4j</groupId>
        <artifactId>langchain4j</artifactId>
        <version>0.36.0</version>
    </dependency>
    
    <!-- OpenAI集成 -->
    <dependency>
        <groupId>dev.langchain4j</groupId>
        <artifactId>langchain4j-open-ai</artifactId>
        <version>0.36.0</version>
    </dependency>
    
    <!-- Spring Boot集成 -->
    <dependency>
        <groupId>dev.langchain4j</groupId>
        <artifactId>langchain4j-spring-boot-starter</artifactId>
        <version>0.36.0</version>
    </dependency>
    
    <!-- 文档解析 -->
    <dependency>
        <groupId>dev.langchain4j</groupId>
        <artifactId>langchain4j-document-parser-apache-pdfbox</artifactId>
        <version>0.36.0</version>
    </dependency>
    
    <!-- 向量数据库 - Pinecone -->
    <dependency>
        <groupId>dev.langchain4j</groupId>
        <artifactId>langchain4j-pinecone</artifactId>
        <version>0.36.0</version>
    </dependency>
</dependencies>
```

### 3.2 第一个LangChain4j应用

```java
package com.example.langchain.quickstart;

import dev.langchain4j.model.chat.ChatLanguageModel;
import dev.langchain4j.model.openai.OpenAiChatModel;

/**
 * LangChain4j快速入门示例
 * 
 * @author erik.zhou
 */
public class QuickStartExample {
    
    public static void main(String[] args) {
        // 1. 创建语言模型
        ChatLanguageModel model = OpenAiChatModel.builder()
            .apiKey(System.getenv("OPENAI_API_KEY"))
            .modelName("gpt-4")
            .temperature(0.7)
            .build();
        
        // 2. 发送消息
        String response = model.generate("什么是LangChain4j?");
        
        // 3. 输出响应
        System.out.println(response);
    }
}
```

### 3.3 Spring Boot集成

#### 3.3.1 配置文件
```yaml
# application.yml
langchain4j:
  open-ai:
    chat-model:
      api-key: ${OPENAI_API_KEY}
      model-name: gpt-4
      temperature: 0.7
      max-tokens: 2000
    embedding-model:
      api-key: ${OPENAI_API_KEY}
      model-name: text-embedding-ada-002
```

#### 3.3.2 服务类
```java
package com.example.langchain.service;

import dev.langchain4j.model.chat.ChatLanguageModel;
import dev.langchain4j.service.AiServices;
import org.springframework.stereotype.Service;

/**
 * AI服务接口
 * 
 * @author erik.zhou
 */
interface Assistant {
    String chat(String message);
}

/**
 * AI服务实现
 * 
 * @author erik.zhou
 */
@Service
public class AiService {
    
    private final Assistant assistant;
    
    public AiService(ChatLanguageModel chatLanguageModel) {
        // 使用AiServices创建代理
        this.assistant = AiServices.create(Assistant.class, chatLanguageModel);
    }
    
    public String chat(String message) {
        return assistant.chat(message);
    }
}
```

#### 3.3.3 控制器
```java
package com.example.langchain.controller;

import com.example.langchain.service.AiService;
import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.*;

/**
 * AI聊天控制器
 * 
 * @author erik.zhou
 */
@RestController
@RequestMapping("/api/chat")
@RequiredArgsConstructor
public class ChatController {
    
    private final AiService aiService;
    
    @PostMapping
    public String chat(@RequestBody String message) {
        return aiService.chat(message);
    }
}
```

---

## 4. 核心功能详解

### 4.1 对话管理 (Chat Memory)

#### 4.1.1 消息历史管理
```java
package com.example.langchain.memory;

import dev.langchain4j.memory.ChatMemory;
import dev.langchain4j.memory.chat.MessageWindowChatMemory;
import dev.langchain4j.model.chat.ChatLanguageModel;
import dev.langchain4j.service.AiServices;

/**
 * 对话记忆示例
 * 
 * @author erik.zhou
 */
public class ChatMemoryExample {
    
    interface ChatBot {
        String chat(String message);
    }
    
    public static void main(String[] args) {
        ChatLanguageModel model = OpenAiChatModel.builder()
            .apiKey(System.getenv("OPENAI_API_KEY"))
            .build();
        
        // 创建消息窗口记忆(保留最近10条消息)
        ChatMemory chatMemory = MessageWindowChatMemory.withMaxMessages(10);
        
        // 创建带记忆的聊天机器人
        ChatBot bot = AiServices.builder(ChatBot.class)
            .chatLanguageModel(model)
            .chatMemory(chatMemory)
            .build();
        
        // 多轮对话
        System.out.println(bot.chat("我叫张三"));
        System.out.println(bot.chat("我的名字是什么?")); // 会记住之前的对话
    }
}
```


#### 4.1.2 持久化记忆
```java
package com.example.langchain.memory;

import dev.langchain4j.data.message.ChatMessage;
import dev.langchain4j.memory.ChatMemory;
import dev.langchain4j.store.memory.chat.ChatMemoryStore;
import lombok.RequiredArgsConstructor;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.stereotype.Component;

import java.util.List;

/**
 * Redis持久化聊天记忆
 * 
 * @author erik.zhou
 */
@Component
@RequiredArgsConstructor
public class RedisChatMemoryStore implements ChatMemoryStore {
    
    private final RedisTemplate<String, List<ChatMessage>> redisTemplate;
    private static final String KEY_PREFIX = "chat:memory:";
    
    @Override
    public List<ChatMessage> getMessages(Object memoryId) {
        String key = KEY_PREFIX + memoryId;
        return redisTemplate.opsForValue().get(key);
    }
    
    @Override
    public void updateMessages(Object memoryId, List<ChatMessage> messages) {
        String key = KEY_PREFIX + memoryId;
        redisTemplate.opsForValue().set(key, messages);
    }
    
    @Override
    public void deleteMessages(Object memoryId) {
        String key = KEY_PREFIX + memoryId;
        redisTemplate.delete(key);
    }
}
```

### 4.2 RAG (检索增强生成)

#### 4.2.1 基础RAG实现
```java
package com.example.langchain.rag;

import dev.langchain4j.data.document.Document;
import dev.langchain4j.data.document.DocumentSplitter;
import dev.langchain4j.data.document.splitter.DocumentSplitters;
import dev.langchain4j.data.embedding.Embedding;
import dev.langchain4j.data.segment.TextSegment;
import dev.langchain4j.model.embedding.EmbeddingModel;
import dev.langchain4j.model.openai.OpenAiEmbeddingModel;
import dev.langchain4j.store.embedding.EmbeddingStore;
import dev.langchain4j.store.embedding.inmemory.InMemoryEmbeddingStore;

import java.util.List;

/**
 * RAG基础示例
 * 
 * @author erik.zhou
 */
public class BasicRAGExample {
    
    public static void main(String[] args) {
        // 1. 创建嵌入模型
        EmbeddingModel embeddingModel = OpenAiEmbeddingModel.builder()
            .apiKey(System.getenv("OPENAI_API_KEY"))
            .modelName("text-embedding-ada-002")
            .build();
        
        // 2. 创建向量存储
        EmbeddingStore<TextSegment> embeddingStore = new InMemoryEmbeddingStore<>();
        
        // 3. 加载文档
        Document document = Document.from("LangChain4j是一个Java版本的LangChain框架...");
        
        // 4. 文档分割
        DocumentSplitter splitter = DocumentSplitters.recursive(
            300,  // 每个片段最大字符数
            50    // 片段之间的重叠字符数
        );
        List<TextSegment> segments = splitter.split(document);
        
        // 5. 生成嵌入并存储
        for (TextSegment segment : segments) {
            Embedding embedding = embeddingModel.embed(segment).content();
            embeddingStore.add(embedding, segment);
        }
        
        System.out.println("文档已成功索引!");
    }
}
```

#### 4.2.2 完整RAG服务
```java
package com.example.langchain.rag;

import dev.langchain4j.data.document.Document;
import dev.langchain4j.data.document.loader.FileSystemDocumentLoader;
import dev.langchain4j.data.document.parser.apache.pdfbox.ApachePdfBoxDocumentParser;
import dev.langchain4j.data.document.splitter.DocumentSplitters;
import dev.langchain4j.data.segment.TextSegment;
import dev.langchain4j.model.chat.ChatLanguageModel;
import dev.langchain4j.model.embedding.EmbeddingModel;
import dev.langchain4j.rag.content.retriever.ContentRetriever;
import dev.langchain4j.rag.content.retriever.EmbeddingStoreContentRetriever;
import dev.langchain4j.service.AiServices;
import dev.langchain4j.store.embedding.EmbeddingStore;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;

import java.nio.file.Path;
import java.util.List;

/**
 * RAG服务实现
 * 
 * @author erik.zhou
 */
@Service
@RequiredArgsConstructor
public class RAGService {
    
    private final ChatLanguageModel chatModel;
    private final EmbeddingModel embeddingModel;
    private final EmbeddingStore<TextSegment> embeddingStore;
    
    /**
     * 索引PDF文档
     */
    public void indexDocument(Path filePath) {
        // 1. 加载PDF文档
        Document document = FileSystemDocumentLoader.loadDocument(
            filePath,
            new ApachePdfBoxDocumentParser()
        );
        
        // 2. 分割文档
        List<TextSegment> segments = DocumentSplitters.recursive(500, 100)
            .split(document);
        
        // 3. 生成嵌入并存储
        for (TextSegment segment : segments) {
            embeddingStore.add(
                embeddingModel.embed(segment).content(),
                segment
            );
        }
    }
    
    /**
     * 基于文档的问答
     */
    public String query(String question) {
        // 创建内容检索器
        ContentRetriever retriever = EmbeddingStoreContentRetriever.builder()
            .embeddingStore(embeddingStore)
            .embeddingModel(embeddingModel)
            .maxResults(3)  // 检索最相关的3个片段
            .minScore(0.7)  // 最小相似度阈值
            .build();
        
        // 创建RAG助手
        interface RAGAssistant {
            String answer(String question);
        }
        
        RAGAssistant assistant = AiServices.builder(RAGAssistant.class)
            .chatLanguageModel(chatModel)
            .contentRetriever(retriever)
            .build();
        
        return assistant.answer(question);
    }
}
```

### 4.3 AI Services (AI服务)

#### 4.3.1 声明式AI服务
```java
package com.example.langchain.service;

import dev.langchain4j.service.SystemMessage;
import dev.langchain4j.service.UserMessage;
import dev.langchain4j.service.V;

/**
 * 声明式AI服务接口
 * 
 * @author erik.zhou
 */
public interface CustomerSupportAgent {
    
    /**
     * 系统提示词
     */
    @SystemMessage("""
        你是一个专业的客服助手。
        你的任务是帮助用户解决问题。
        请保持礼貌和专业。
        """)
    String chat(@UserMessage String userMessage);
    
    /**
     * 带参数的提示词
     */
    @UserMessage("将以下文本翻译成{{language}}: {{text}}")
    String translate(@V("language") String language, @V("text") String text);
    
    /**
     * 结构化输出
     */
    @UserMessage("从以下文本中提取客户信息: {{text}}")
    CustomerInfo extractCustomerInfo(@V("text") String text);
}

/**
 * 客户信息实体
 */
record CustomerInfo(
    String name,
    String email,
    String phone
) {}
```

#### 4.3.2 使用AI服务
```java
package com.example.langchain.service;

import dev.langchain4j.model.chat.ChatLanguageModel;
import dev.langchain4j.service.AiServices;
import org.springframework.stereotype.Service;

/**
 * 客服服务实现
 * 
 * @author erik.zhou
 */
@Service
public class CustomerService {
    
    private final CustomerSupportAgent agent;
    
    public CustomerService(ChatLanguageModel chatModel) {
        this.agent = AiServices.create(CustomerSupportAgent.class, chatModel);
    }
    
    public String handleCustomerQuery(String query) {
        return agent.chat(query);
    }
    
    public String translateText(String text, String targetLanguage) {
        return agent.translate(targetLanguage, text);
    }
    
    public CustomerInfo extractInfo(String text) {
        return agent.extractCustomerInfo(text);
    }
}
```

### 4.4 Tools (工具调用)

#### 4.4.1 定义工具
```java
package com.example.langchain.tools;

import dev.langchain4j.agent.tool.Tool;
import org.springframework.stereotype.Component;

import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;

/**
 * 工具类示例
 * 
 * @author erik.zhou
 */
@Component
public class DateTimeTools {
    
    @Tool("获取当前日期和时间")
    public String getCurrentDateTime() {
        return LocalDateTime.now()
            .format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"));
    }
    
    @Tool("计算两个日期之间的天数差")
    public long daysBetween(String date1, String date2) {
        LocalDateTime dt1 = LocalDateTime.parse(date1);
        LocalDateTime dt2 = LocalDateTime.parse(date2);
        return java.time.Duration.between(dt1, dt2).toDays();
    }
}

/**
 * 天气工具
 */
@Component
public class WeatherTools {
    
    @Tool("获取指定城市的天气信息")
    public String getWeather(String city) {
        // 实际应用中应该调用真实的天气API
        return String.format("%s的天气: 晴天, 温度25°C", city);
    }
}
```

#### 4.4.2 使用工具的Agent
```java
package com.example.langchain.agent;

import com.example.langchain.tools.DateTimeTools;
import com.example.langchain.tools.WeatherTools;
import dev.langchain4j.model.chat.ChatLanguageModel;
import dev.langchain4j.service.AiServices;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;

/**
 * 带工具的AI Agent
 * 
 * @author erik.zhou
 */
@Service
@RequiredArgsConstructor
public class AgentService {
    
    private final ChatLanguageModel chatModel;
    private final DateTimeTools dateTimeTools;
    private final WeatherTools weatherTools;
    
    interface Assistant {
        String chat(String message);
    }
    
    public String chat(String message) {
        Assistant assistant = AiServices.builder(Assistant.class)
            .chatLanguageModel(chatModel)
            .tools(dateTimeTools, weatherTools)  // 注册工具
            .build();
        
        return assistant.chat(message);
    }
}
```

---

## 5. 高级特性

### 5.1 流式响应

```java
package com.example.langchain.streaming;

import dev.langchain4j.model.chat.StreamingChatLanguageModel;
import dev.langchain4j.model.openai.OpenAiStreamingChatModel;
import dev.langchain4j.service.AiServices;
import dev.langchain4j.service.TokenStream;
import org.springframework.stereotype.Service;
import reactor.core.publisher.Flux;

/**
 * 流式响应服务
 * 
 * @author erik.zhou
 */
@Service
public class StreamingService {
    
    private final StreamingChatLanguageModel streamingModel;
    
    public StreamingService() {
        this.streamingModel = OpenAiStreamingChatModel.builder()
            .apiKey(System.getenv("OPENAI_API_KEY"))
            .modelName("gpt-4")
            .build();
    }
    
    interface StreamingAssistant {
        TokenStream chat(String message);
    }
    
    /**
     * 流式聊天
     */
    public Flux<String> streamChat(String message) {
        StreamingAssistant assistant = AiServices.create(
            StreamingAssistant.class,
            streamingModel
        );
        
        return Flux.create(sink -> {
            assistant.chat(message)
                .onNext(sink::next)
                .onComplete(response -> sink.complete())
                .onError(sink::error)
                .start();
        });
    }
}
```


### 5.2 多模型支持

#### 5.2.1 Azure OpenAI
```java
package com.example.langchain.model;

import dev.langchain4j.model.azure.AzureOpenAiChatModel;
import dev.langchain4j.model.chat.ChatLanguageModel;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * Azure OpenAI配置
 * 
 * @author erik.zhou
 */
@Configuration
public class AzureOpenAiConfig {
    
    @Bean
    public ChatLanguageModel azureChatModel() {
        return AzureOpenAiChatModel.builder()
            .endpoint(System.getenv("AZURE_OPENAI_ENDPOINT"))
            .apiKey(System.getenv("AZURE_OPENAI_KEY"))
            .deploymentName("gpt-4")
            .temperature(0.7)
            .build();
    }
}
```

#### 5.2.2 本地模型 (Ollama)
```java
package com.example.langchain.model;

import dev.langchain4j.model.chat.ChatLanguageModel;
import dev.langchain4j.model.ollama.OllamaChatModel;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * Ollama本地模型配置
 * 
 * @author erik.zhou
 */
@Configuration
public class OllamaConfig {
    
    @Bean
    public ChatLanguageModel ollamaChatModel() {
        return OllamaChatModel.builder()
            .baseUrl("http://localhost:11434")
            .modelName("llama2")
            .temperature(0.7)
            .build();
    }
}
```

### 5.3 提示词工程

#### 5.3.1 提示词模板
```java
package com.example.langchain.prompt;

import dev.langchain4j.model.chat.ChatLanguageModel;
import dev.langchain4j.model.input.Prompt;
import dev.langchain4j.model.input.PromptTemplate;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;

import java.util.Map;

/**
 * 提示词模板服务
 * 
 * @author erik.zhou
 */
@Service
@RequiredArgsConstructor
public class PromptService {
    
    private final ChatLanguageModel chatModel;
    
    /**
     * 使用提示词模板
     */
    public String generateProductDescription(String productName, String features) {
        PromptTemplate template = PromptTemplate.from("""
            你是一个专业的产品文案撰写专家。
            
            请为以下产品撰写一段吸引人的描述:
            产品名称: {{productName}}
            产品特点: {{features}}
            
            要求:
            1. 突出产品优势
            2. 语言生动有趣
            3. 长度控制在100字以内
            """);
        
        Prompt prompt = template.apply(Map.of(
            "productName", productName,
            "features", features
        ));
        
        return chatModel.generate(prompt.text());
    }
}
```

#### 5.3.2 Few-Shot Learning
```java
package com.example.langchain.prompt;

import dev.langchain4j.data.message.AiMessage;
import dev.langchain4j.data.message.ChatMessage;
import dev.langchain4j.data.message.SystemMessage;
import dev.langchain4j.data.message.UserMessage;
import dev.langchain4j.model.chat.ChatLanguageModel;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;

import java.util.List;

/**
 * Few-Shot学习示例
 * 
 * @author erik.zhou
 */
@Service
@RequiredArgsConstructor
public class FewShotService {
    
    private final ChatLanguageModel chatModel;
    
    /**
     * 情感分析 - Few-Shot示例
     */
    public String analyzeSentiment(String text) {
        List<ChatMessage> messages = List.of(
            SystemMessage.from("你是一个情感分析专家，分析文本的情感倾向(正面/负面/中性)"),
            
            // 示例1
            UserMessage.from("这个产品太棒了，我非常喜欢！"),
            AiMessage.from("正面"),
            
            // 示例2
            UserMessage.from("质量太差了，完全不值这个价格。"),
            AiMessage.from("负面"),
            
            // 示例3
            UserMessage.from("还可以，没有特别的感觉。"),
            AiMessage.from("中性"),
            
            // 实际查询
            UserMessage.from(text)
        );
        
        return chatModel.generate(messages).content().text();
    }
}
```

### 5.4 错误处理和重试

```java
package com.example.langchain.retry;

import dev.langchain4j.model.chat.ChatLanguageModel;
import io.github.resilience4j.retry.Retry;
import io.github.resilience4j.retry.RetryConfig;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Service;

import java.time.Duration;

/**
 * 带重试机制的AI服务
 * 
 * @author erik.zhou
 */
@Slf4j
@Service
public class ResilientAiService {
    
    private final ChatLanguageModel chatModel;
    private final Retry retry;
    
    public ResilientAiService(ChatLanguageModel chatModel) {
        this.chatModel = chatModel;
        
        // 配置重试策略
        RetryConfig config = RetryConfig.custom()
            .maxAttempts(3)
            .waitDuration(Duration.ofSeconds(2))
            .retryExceptions(Exception.class)
            .build();
        
        this.retry = Retry.of("aiService", config);
    }
    
    /**
     * 带重试的聊天
     */
    public String chatWithRetry(String message) {
        return Retry.decorateSupplier(retry, () -> {
            try {
                return chatModel.generate(message);
            } catch (Exception e) {
                log.error("AI调用失败，准备重试: {}", e.getMessage());
                throw e;
            }
        }).get();
    }
}
```

---

## 6. 实战案例

### 6.1 智能文档问答系统

```java
package com.example.langchain.qa;

import dev.langchain4j.data.document.Document;
import dev.langchain4j.data.document.loader.FileSystemDocumentLoader;
import dev.langchain4j.data.document.parser.apache.pdfbox.ApachePdfBoxDocumentParser;
import dev.langchain4j.data.document.splitter.DocumentSplitters;
import dev.langchain4j.data.segment.TextSegment;
import dev.langchain4j.model.chat.ChatLanguageModel;
import dev.langchain4j.model.embedding.EmbeddingModel;
import dev.langchain4j.rag.content.retriever.ContentRetriever;
import dev.langchain4j.rag.content.retriever.EmbeddingStoreContentRetriever;
import dev.langchain4j.service.AiServices;
import dev.langchain4j.store.embedding.EmbeddingStore;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Service;

import java.nio.file.Path;
import java.util.List;

/**
 * 智能文档问答系统
 * 
 * 功能:
 * 1. 支持PDF文档上传和索引
 * 2. 基于文档内容的智能问答
 * 3. 相关性评分和答案来源追踪
 * 
 * @author erik.zhou
 */
@Slf4j
@Service
@RequiredArgsConstructor
public class DocumentQAService {
    
    private final ChatLanguageModel chatModel;
    private final EmbeddingModel embeddingModel;
    private final EmbeddingStore<TextSegment> embeddingStore;
    
    /**
     * 索引文档
     */
    public void indexDocument(Path filePath, String documentId) {
        log.info("开始索引文档: {}", filePath);
        
        try {
            // 1. 加载文档
            Document document = FileSystemDocumentLoader.loadDocument(
                filePath,
                new ApachePdfBoxDocumentParser()
            );
            
            // 2. 分割文档 (每段500字符，重叠100字符)
            List<TextSegment> segments = DocumentSplitters
                .recursive(500, 100)
                .split(document);
            
            log.info("文档已分割为 {} 个片段", segments.size());
            
            // 3. 为每个片段添加元数据
            for (int i = 0; i < segments.size(); i++) {
                TextSegment segment = segments.get(i);
                segment.metadata().put("documentId", documentId);
                segment.metadata().put("segmentIndex", i);
                segment.metadata().put("source", filePath.toString());
                
                // 4. 生成嵌入并存储
                embeddingStore.add(
                    embeddingModel.embed(segment).content(),
                    segment
                );
            }
            
            log.info("文档索引完成: {}", documentId);
            
        } catch (Exception e) {
            log.error("文档索引失败: {}", e.getMessage(), e);
            throw new RuntimeException("文档索引失败", e);
        }
    }
    
    /**
     * 问答接口
     */
    interface QAAssistant {
        String answer(String question);
    }
    
    /**
     * 基于文档的问答
     */
    public QAResponse query(String question) {
        log.info("收到问题: {}", question);
        
        // 创建内容检索器
        ContentRetriever retriever = EmbeddingStoreContentRetriever.builder()
            .embeddingStore(embeddingStore)
            .embeddingModel(embeddingModel)
            .maxResults(5)      // 检索最相关的5个片段
            .minScore(0.6)      // 最小相似度阈值
            .build();
        
        // 创建QA助手
        QAAssistant assistant = AiServices.builder(QAAssistant.class)
            .chatLanguageModel(chatModel)
            .contentRetriever(retriever)
            .build();
        
        // 获取答案
        String answer = assistant.answer(question);
        
        return new QAResponse(question, answer);
    }
    
    /**
     * 问答响应
     */
    public record QAResponse(
        String question,
        String answer
    ) {}
}
```

### 6.2 智能代码审查助手

```java
package com.example.langchain.codereview;

import dev.langchain4j.model.chat.ChatLanguageModel;
import dev.langchain4j.service.AiServices;
import dev.langchain4j.service.SystemMessage;
import dev.langchain4j.service.UserMessage;
import dev.langchain4j.service.V;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;

import java.util.List;

/**
 * 智能代码审查助手
 * 
 * @author erik.zhou
 */
@Service
@RequiredArgsConstructor
public class CodeReviewService {
    
    private final ChatLanguageModel chatModel;
    
    /**
     * 代码审查助手接口
     */
    interface CodeReviewer {
        
        @SystemMessage("""
            你是一个资深的Java代码审查专家。
            你的任务是审查代码并提供建设性的反馈。
            
            审查重点:
            1. 代码质量和可读性
            2. 潜在的bug和安全问题
            3. 性能优化建议
            4. 最佳实践和设计模式
            
            请以专业、友好的方式提供反馈。
            """)
        @UserMessage("请审查以下代码:\n\n```java\n{{code}}\n```")
        CodeReviewResult review(@V("code") String code);
    }
    
    /**
     * 审查代码
     */
    public CodeReviewResult reviewCode(String code) {
        CodeReviewer reviewer = AiServices.create(CodeReviewer.class, chatModel);
        return reviewer.review(code);
    }
    
    /**
     * 代码审查结果
     */
    public record CodeReviewResult(
        String summary,              // 总体评价
        List<Issue> issues,          // 发现的问题
        List<Suggestion> suggestions // 改进建议
    ) {}
    
    public record Issue(
        String severity,  // 严重程度: HIGH/MEDIUM/LOW
        String line,      // 行号
        String description // 问题描述
    ) {}
    
    public record Suggestion(
        String category,   // 类别: PERFORMANCE/READABILITY/SECURITY
        String description // 建议描述
    ) {}
}
```

### 6.3 多语言翻译服务

```java
package com.example.langchain.translation;

import dev.langchain4j.model.chat.ChatLanguageModel;
import dev.langchain4j.service.AiServices;
import dev.langchain4j.service.SystemMessage;
import dev.langchain4j.service.UserMessage;
import dev.langchain4j.service.V;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;

/**
 * 多语言翻译服务
 * 
 * @author erik.zhou
 */
@Service
@RequiredArgsConstructor
public class TranslationService {
    
    private final ChatLanguageModel chatModel;
    
    interface Translator {
        
        @SystemMessage("""
            你是一个专业的翻译专家。
            你的任务是将文本准确地翻译成目标语言。
            
            翻译要求:
            1. 保持原文的语气和风格
            2. 确保专业术语的准确性
            3. 符合目标语言的表达习惯
            4. 只返回翻译结果，不要添加额外说明
            """)
        @UserMessage("将以下{{sourceLanguage}}文本翻译成{{targetLanguage}}:\n\n{{text}}")
        String translate(
            @V("sourceLanguage") String sourceLanguage,
            @V("targetLanguage") String targetLanguage,
            @V("text") String text
        );
    }
    
    private final Translator translator;
    
    public TranslationService(ChatLanguageModel chatModel) {
        this.chatModel = chatModel;
        this.translator = AiServices.create(Translator.class, chatModel);
    }
    
    /**
     * 翻译文本
     */
    public String translate(String text, String sourceLanguage, String targetLanguage) {
        return translator.translate(sourceLanguage, targetLanguage, text);
    }
    
    /**
     * 批量翻译
     */
    public List<String> batchTranslate(
        List<String> texts,
        String sourceLanguage,
        String targetLanguage
    ) {
        return texts.stream()
            .map(text -> translate(text, sourceLanguage, targetLanguage))
            .toList();
    }
}
```

---

## 7. 最佳实践

### 7.1 提示词优化

```java
/**
 * 提示词最佳实践
 * 
 * @author erik.zhou
 */
public class PromptBestPractices {
    
    // ❌ 不好的提示词
    String badPrompt = "翻译这个";
    
    // ✅ 好的提示词
    String goodPrompt = """
        你是一个专业的技术文档翻译专家。
        
        任务: 将以下英文技术文档翻译成中文
        要求:
        1. 保持技术术语的准确性
        2. 使用专业的技术表达
        3. 保持原文的格式和结构
        
        原文:
        {{text}}
        """;
}
```

### 7.2 成本优化

```java
package com.example.langchain.optimization;

import dev.langchain4j.model.chat.ChatLanguageModel;
import dev.langchain4j.model.openai.OpenAiChatModel;
import org.springframework.cache.annotation.Cacheable;
import org.springframework.stereotype.Service;

/**
 * 成本优化策略
 * 
 * @author erik.zhou
 */
@Service
public class CostOptimizationService {
    
    private final ChatLanguageModel cheapModel;   // 便宜的模型
    private final ChatLanguageModel expensiveModel; // 昂贵但更强大的模型
    
    public CostOptimizationService() {
        // GPT-3.5 - 便宜快速
        this.cheapModel = OpenAiChatModel.builder()
            .apiKey(System.getenv("OPENAI_API_KEY"))
            .modelName("gpt-3.5-turbo")
            .build();
        
        // GPT-4 - 昂贵但更强大
        this.expensiveModel = OpenAiChatModel.builder()
            .apiKey(System.getenv("OPENAI_API_KEY"))
            .modelName("gpt-4")
            .build();
    }
    
    /**
     * 根据任务复杂度选择模型
     */
    public String smartGenerate(String prompt, boolean isComplexTask) {
        ChatLanguageModel model = isComplexTask ? expensiveModel : cheapModel;
        return model.generate(prompt);
    }
    
    /**
     * 缓存常见问题的答案
     */
    @Cacheable(value = "aiResponses", key = "#question")
    public String cachedGenerate(String question) {
        return cheapModel.generate(question);
    }
}
```

### 7.3 安全性

```java
package com.example.langchain.security;

import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Component;

import java.util.List;
import java.util.regex.Pattern;

/**
 * 输入验证和安全过滤
 * 
 * @author erik.zhou
 */
@Slf4j
@Component
public class InputValidator {
    
    private static final int MAX_INPUT_LENGTH = 10000;
    private static final List<Pattern> DANGEROUS_PATTERNS = List.of(
        Pattern.compile("(?i)ignore.*previous.*instructions"),
        Pattern.compile("(?i)system.*prompt"),
        Pattern.compile("(?i)jailbreak")
    );
    
    /**
     * 验证用户输入
     */
    public void validateInput(String input) {
        // 1. 长度检查
        if (input == null || input.isBlank()) {
            throw new IllegalArgumentException("输入不能为空");
        }
        
        if (input.length() > MAX_INPUT_LENGTH) {
            throw new IllegalArgumentException("输入长度超过限制");
        }
        
        // 2. 危险模式检查
        for (Pattern pattern : DANGEROUS_PATTERNS) {
            if (pattern.matcher(input).find()) {
                log.warn("检测到可疑输入: {}", input);
                throw new SecurityException("输入包含不安全内容");
            }
        }
    }
    
    /**
     * 清理输出
     */
    public String sanitizeOutput(String output) {
        // 移除敏感信息
        return output
            .replaceAll("API[_\\s]?KEY[:\\s]+[\\w-]+", "API_KEY: [REDACTED]")
            .replaceAll("PASSWORD[:\\s]+[\\w-]+", "PASSWORD: [REDACTED]");
    }
}
```

---

## 8. 性能优化

### 8.1 批处理

```java
package com.example.langchain.performance;

import dev.langchain4j.model.embedding.EmbeddingModel;
import dev.langchain4j.data.segment.TextSegment;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;

import java.util.List;
import java.util.concurrent.CompletableFuture;
import java.util.stream.Collectors;

/**
 * 批处理优化
 * 
 * @author erik.zhou
 */
@Service
@RequiredArgsConstructor
public class BatchProcessingService {
    
    private final EmbeddingModel embeddingModel;
    
    /**
     * 批量生成嵌入
     */
    public List<Embedding> batchEmbed(List<String> texts) {
        // 将文本分批处理 (每批100个)
        int batchSize = 100;
        List<CompletableFuture<List<Embedding>>> futures = new ArrayList<>();
        
        for (int i = 0; i < texts.size(); i += batchSize) {
            int end = Math.min(i + batchSize, texts.size());
            List<String> batch = texts.subList(i, end);
            
            CompletableFuture<List<Embedding>> future = CompletableFuture
                .supplyAsync(() -> embeddingModel.embedAll(batch).content());
            
            futures.add(future);
        }
        
        // 等待所有批次完成
        return futures.stream()
            .map(CompletableFuture::join)
            .flatMap(List::stream)
            .collect(Collectors.toList());
    }
}
```

### 8.2 连接池配置

```yaml
# application.yml
langchain4j:
  open-ai:
    chat-model:
      timeout: 60s
      max-retries: 3
      log-requests: false
      log-responses: false
    
spring:
  task:
    execution:
      pool:
        core-size: 10
        max-size: 50
        queue-capacity: 100
```

---

## 9. 总结

LangChain4j为Java开发者提供了强大的LLM应用开发能力:

### 核心优势
- ✅ 类型安全的Java实现
- ✅ Spring Boot无缝集成
- ✅ 丰富的模型和向量数据库支持
- ✅ 完善的RAG实现
- ✅ 生产级性能和稳定性

### 适用场景
- 智能客服和问答系统
- 文档分析和知识管理
- 代码生成和审查
- 内容创作和翻译
- 数据分析和洞察

### 学习建议
1. 从简单的聊天开始
2. 掌握RAG核心概念
3. 实践工具调用和Agent
4. 关注成本和性能优化
5. 重视安全性和错误处理

---

**作者**: erik.zhou  
**最后更新**: 2025-01-31  
**版本**: 1.0.0
