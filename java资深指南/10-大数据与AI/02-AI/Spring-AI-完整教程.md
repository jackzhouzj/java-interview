# Spring AI 完整教程

## 📋 目录
- [技术概述](#技术概述)
- [学习目标](#学习目标)
- [基础概念](#基础概念)
- [核心特性](#核心特性)
- [实战应用](#实战应用)
- [最佳实践](#最佳实践)
- [常见问题](#常见问题)
- [相关资源](#相关资源)

## 📚 技术概述

- **版本**: 1.0.0-M4 (最新里程碑版本)
- **官方文档**: https://spring.io/projects/spring-ai
- **学习难度**: ⭐⭐⭐⭐ (4/5星)
- **重要程度**: ⭐⭐⭐⭐ (4/5星)
- **前置知识**: 
  - Spring Boot基础
  - RESTful API设计
  - AI/ML基本概念
  - 向量数据库概念

**文档来源**: Spring AI官方文档 (Context7)

## 🎯 学习目标

- [ ] 理解Spring AI的核心架构
- [ ] 掌握ChatClient API的使用
- [ ] 理解RAG (检索增强生成) 架构
- [ ] 掌握向量数据库的集成
- [ ] 能够构建AI驱动的Java应用
- [ ] 了解AI应用的最佳实践

## 📖 基础概念

### 1.1 什么是Spring AI

Spring AI是Spring生态系统中用于构建AI应用的框架，它提供了：

- **统一的AI接口**: 支持多种AI模型 (OpenAI, Azure OpenAI, Anthropic等)
- **RAG支持**: 检索增强生成，结合知识库和AI
- **向量数据库集成**: 支持多种向量数据库
- **Spring Boot集成**: 自动配置和依赖注入
- **类型安全**: 强类型API，编译时检查

### 1.2 核心组件

```
ChatClient (对话客户端)
  ↓
ChatModel (AI模型)
  ↓
VectorStore (向量存储)
  ↓
DocumentRetriever (文档检索器)
  ↓
RAG Advisor (RAG顾问)
```

### 1.3 应用场景

- **智能客服**: AI驱动的客户支持
- **知识问答**: 基于企业知识库的问答系统
- **内容生成**: 自动生成文档、报告
- **代码助手**: AI辅助编程
- **数据分析**: 自然语言查询数据

## 🔥 核心特性 (重点)

### 2.1 ChatClient API 🔥

ChatClient是Spring AI的核心API，用于与AI模型交互。

#### 2.1.1 基本使用

**Maven依赖**:
```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-openai-spring-boot-starter</artifactId>
    <version>1.0.0-M4</version>
</dependency>
```

**配置文件** (application.yml):
```yaml
spring:
  ai:
    openai:
      api-key: ${OPENAI_API_KEY}
      chat:
        options:
          model: gpt-4
          temperature: 0.7
```

**基本示例**:
```java
import org.springframework.ai.chat.client.ChatClient;
import org.springframework.ai.chat.model.ChatModel;
import org.springframework.stereotype.Service;

/**
 * ChatClient基本使用
 * 
 * @author erik.zhou
 */
@Service
public class AIChatService {
    
    private final ChatClient chatClient;
    
    public AIChatService(ChatModel chatModel) {
        this.chatClient = ChatClient.builder(chatModel).build();
    }
    
    /**
     * 简单对话
     */
    public String chat(String userMessage) {
        return chatClient.prompt()
            .user(userMessage)
            .call()
            .content();
    }
    
    /**
     * 带系统提示的对话
     */
    public String chatWithSystem(String userMessage) {
        return chatClient.prompt()
            .system("你是一个专业的Java开发助手")
            .user(userMessage)
            .call()
            .content();
    }
}
```

#### 2.1.2 结构化输出

```java
import org.springframework.ai.chat.client.ChatClient;
import org.springframework.ai.chat.model.ChatModel;
import org.springframework.stereotype.Service;

/**
 * 结构化输出示例
 * 
 * @author erik.zhou
 */
@Service
public class StructuredOutputService {
    
    private final ChatClient chatClient;
    
    public StructuredOutputService(ChatModel chatModel) {
        this.chatClient = ChatClient.builder(chatModel).build();
    }
    
    // 定义输出结构
    public record UserInfo(String name, int age, String city) {}
    
    /**
     * 提取结构化信息
     */
    public UserInfo extractUserInfo(String text) {
        return chatClient.prompt()
            .user("从以下文本中提取用户信息: " + text)
            .call()
            .entity(UserInfo.class);
    }
}
```

### 2.2 RAG (检索增强生成) 🔥

RAG是Spring AI的核心特性，结合知识库和AI模型生成答案。

#### 2.2.1 RAG架构

```
用户问题
  ↓
查询转换 (Query Transformation)
  ↓
文档检索 (Document Retrieval)
  ↓
上下文增强 (Context Augmentation)
  ↓
AI生成答案
  ↓
返回结果
```

#### 2.2.2 基本RAG实现

```java
import org.springframework.ai.chat.client.ChatClient;
import org.springframework.ai.chat.model.ChatModel;
import org.springframework.ai.rag.advisor.RetrievalAugmentationAdvisor;
import org.springframework.ai.rag.retrieval.search.VectorStoreDocumentRetriever;
import org.springframework.ai.vectorstore.VectorStore;
import org.springframework.stereotype.Service;

/**
 * 基本RAG服务
 * 
 * @author erik.zhou
 */
@Service
public class BasicRAGService {
    
    private final ChatClient chatClient;
    
    public BasicRAGService(ChatModel chatModel, VectorStore vectorStore) {
        // 创建文档检索器
        VectorStoreDocumentRetriever retriever = VectorStoreDocumentRetriever.builder()
            .vectorStore(vectorStore)
            .similarityThreshold(0.75)  // 相似度阈值
            .topK(5)  // 检索前5个最相关文档
            .build();
        
        // 创建RAG顾问
        RetrievalAugmentationAdvisor ragAdvisor = RetrievalAugmentationAdvisor.builder()
            .documentRetriever(retriever)
            .build();
        
        // 构建ChatClient
        this.chatClient = ChatClient.builder(chatModel)
            .defaultAdvisors(ragAdvisor)
            .build();
    }
    
    /**
     * 基于知识库的问答
     */
    public String query(String question) {
        return chatClient.prompt()
            .user(question)
            .call()
            .content();
    }
}
```

#### 2.2.3 高级RAG实现 (⚠️ 难点)

```java
import org.springframework.ai.rag.preretrieval.query.transformation.RewriteQueryTransformer;
import org.springframework.ai.rag.preretrieval.query.transformation.TranslationQueryTransformer;
import org.springframework.ai.rag.preretrieval.query.expansion.MultiQueryExpander;

/**
 * 高级RAG服务 - 包含查询转换和扩展
 * 
 * @author erik.zhou
 */
@Service
public class AdvancedRAGService {
    
    private final ChatClient chatClient;
    
    public AdvancedRAGService(ChatModel chatModel, VectorStore vectorStore) {
        // 查询重写器
        RewriteQueryTransformer queryRewriter = RewriteQueryTransformer.builder()
            .chatClientBuilder(ChatClient.builder(chatModel))
            .targetSearchSystem("技术文档向量数据库")
            .build();
        
        // 查询翻译器
        TranslationQueryTransformer queryTranslator = TranslationQueryTransformer.builder()
            .chatClientBuilder(ChatClient.builder(chatModel))
            .targetLanguage("en")
            .build();
        
        // 查询扩展器
        MultiQueryExpander queryExpander = MultiQueryExpander.builder()
            .chatClientBuilder(ChatClient.builder(chatModel))
            .build();
        
        // 文档检索器
        VectorStoreDocumentRetriever retriever = VectorStoreDocumentRetriever.builder()
            .vectorStore(vectorStore)
            .topK(5)
            .similarityThreshold(0.7)
            .build();
        
        // RAG顾问
        RetrievalAugmentationAdvisor ragAdvisor = RetrievalAugmentationAdvisor.builder()
            .queryTransformers(queryRewriter, queryTranslator)
            .queryExpander(queryExpander)
            .documentRetriever(retriever)
            .build();
        
        this.chatClient = ChatClient.builder(chatModel)
            .defaultAdvisors(ragAdvisor)
            .build();
    }
    
    /**
     * 高级RAG查询
     */
    public String queryWithAdvancedRAG(String userQuestion) {
        return chatClient.prompt()
            .user(userQuestion)
            .call()
            .content();
    }
}
```

### 2.3 向量数据库集成

Spring AI支持多种向量数据库。

#### 2.3.1 文档加载和存储

```java
import org.springframework.ai.document.Document;
import org.springframework.ai.reader.pdf.PagePdfDocumentReader;
import org.springframework.ai.transformer.splitter.TextSplitter;
import org.springframework.ai.transformer.splitter.TokenTextSplitter;
import org.springframework.ai.vectorstore.VectorStore;
import org.springframework.stereotype.Service;
import org.springframework.web.multipart.MultipartFile;

import java.util.List;

/**
 * 文档管理服务
 * 
 * @author erik.zhou
 */
@Service
public class DocumentService {
    
    private final VectorStore vectorStore;
    private final TextSplitter textSplitter;
    
    public DocumentService(VectorStore vectorStore) {
        this.vectorStore = vectorStore;
        this.textSplitter = new TokenTextSplitter();
    }
    
    /**
     * 上传PDF文档到向量数据库
     */
    public void uploadPDF(MultipartFile file, String category) throws Exception {
        // 读取PDF
        PagePdfDocumentReader pdfReader = new PagePdfDocumentReader(file.getResource());
        List<Document> documents = pdfReader.get();
        
        // 添加元数据
        documents.forEach(doc -> {
            doc.getMetadata().put("category", category);
            doc.getMetadata().put("filename", file.getOriginalFilename());
            doc.getMetadata().put("uploadTime", System.currentTimeMillis());
        });
        
        // 分割文档
        List<Document> chunks = textSplitter.apply(documents);
        
        // 存储到向量数据库
        vectorStore.add(chunks);
    }
    
    /**
     * 搜索相关文档
     */
    public List<Document> searchDocuments(String query, int topK) {
        return vectorStore.similaritySearch(query, topK);
    }
}
```

## 💻 实战应用

### 3.1 构建知识问答系统

```java
import org.springframework.web.bind.annotation.*;

/**
 * 知识问答API
 * 
 * @author erik.zhou
 */
@RestController
@RequestMapping("/api/qa")
public class QAController {
    
    private final BasicRAGService ragService;
    private final DocumentService documentService;
    
    public QAController(BasicRAGService ragService, DocumentService documentService) {
        this.ragService = ragService;
        this.documentService = documentService;
    }
    
    /**
     * 上传文档
     */
    @PostMapping("/documents")
    public String uploadDocument(
            @RequestParam("file") MultipartFile file,
            @RequestParam("category") String category) throws Exception {
        documentService.uploadPDF(file, category);
        return "文档上传成功";
    }
    
    /**
     * 提问
     */
    @PostMapping("/ask")
    public AnswerResponse ask(@RequestBody QuestionRequest request) {
        long startTime = System.currentTimeMillis();
        String answer = ragService.query(request.question());
        long responseTime = System.currentTimeMillis() - startTime;
        
        return new AnswerResponse(answer, responseTime);
    }
    
    public record QuestionRequest(String question, String category) {}
    public record AnswerResponse(String answer, long responseTimeMs) {}
}
```

## ✨ 最佳实践

### 4.1 性能优化

1. **缓存AI响应**: 对常见问题缓存答案
2. **异步处理**: 使用异步API处理长时间请求
3. **批量处理**: 批量上传文档到向量数据库
4. **合理设置topK**: 平衡准确性和性能

### 4.2 安全考虑

1. **API密钥管理**: 使用环境变量存储密钥
2. **输入验证**: 验证用户输入，防止注入攻击
3. **速率限制**: 限制API调用频率
4. **内容过滤**: 过滤敏感内容

## ❓ 常见问题

### Q1: Spring AI支持哪些AI模型？

**A**: 
- OpenAI (GPT-3.5, GPT-4)
- Azure OpenAI
- Anthropic (Claude)
- Google Vertex AI
- Ollama (本地模型)

### Q2: 如何选择向量数据库？

**A**:
- **Chroma**: 轻量级，适合开发测试
- **Pinecone**: 云服务，易于扩展
- **Weaviate**: 功能丰富，支持多种模型
- **Milvus**: 高性能，适合大规模部署

### Q3: RAG的相似度阈值如何设置？

**A**:
- 0.6-0.7: 宽松，返回更多文档
- 0.7-0.8: 平衡，推荐值
- 0.8-0.9: 严格，只返回高度相关文档

## 🔗 相关资源

- [Spring AI官网](https://spring.io/projects/spring-ai)
- [Spring AI文档](https://docs.spring.io/spring-ai/reference/)
- [Spring AI GitHub](https://github.com/spring-projects/spring-ai)
- [Spring AI示例](https://github.com/spring-projects/spring-ai-examples)

## 📝 学习检查清单

- [ ] 理解Spring AI的核心架构
- [ ] 掌握ChatClient API的使用
- [ ] 理解RAG架构和实现
- [ ] 掌握向量数据库的集成
- [ ] 能够构建AI驱动的应用
- [ ] 了解AI应用的最佳实践

---

**@author** erik.zhou
**最后更新**: 2024-01-04
**文档版本**: 1.0
**文档来源**: Spring AI官方文档 (Context7)
