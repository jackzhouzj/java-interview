# LangChain - 完整教程

> @author erik.zhou

## 📚 技术概述

### 版本信息
- **LangChain版本**：0.1+
- **最新稳定版**：0.1.x
- **推荐版本**：0.1.0+

### 学习难度
- **难度等级**：⭐⭐⭐ (中等)
- **预计学习时间**：30-40小时
- **重要程度**：⭐⭐⭐⭐⭐ (必学)

### 前置知识
- Python基础
- 异步编程基础
- LLM基本概念
- API调用经验

## 🎯 学习目标

完成本教程后，你将能够：

- [ ] 理解LangChain的核心概念和架构
- [ ] 掌握LLM集成和调用方法
- [ ] 熟练使用Prompt模板
- [ ] 能够构建复杂的Chain
- [ ] 掌握Agent开发技巧
- [ ] 理解Memory管理机制
- [ ] 能够集成外部工具
- [ ] 掌握回调和监控方法

## 📖 目录

1. [LangChain简介](#1-langchain简介)
2. [环境搭建](#2-环境搭建)
3. [核心概念](#3-核心概念)
4. [LLM集成](#4-llm集成)
5. [Prompt模板](#5-prompt模板)
6. [Chain链式调用](#6-chain链式调用)
7. [Agent智能体](#7-agent智能体)
8. [Memory记忆](#8-memory记忆)
9. [工具集成](#9-工具集成)
10. [回调和监控](#10-回调和监控)
11. [最佳实践](#11-最佳实践)
12. [实战案例](#12-实战案例)

---

## 1. LangChain简介

### 1.1 什么是LangChain

LangChain是一个用于开发由大语言模型（LLM）驱动的应用程序的框架。它提供了一套标准化的接口和工具，简化了LLM应用的开发流程。

**核心特性**：
- 🔥 **模块化设计**：组件可独立使用或组合
- 🔥 **多模型支持**：支持OpenAI、Anthropic、HuggingFace等
- 🔥 **链式调用**：将多个组件串联成复杂流程
- 🔥 **Agent框架**：构建自主决策的智能体
- 🔥 **丰富的工具**：内置大量实用工具和集成

### 1.2 LangChain生态系统

```
LangChain生态系统
├── LangChain Core      # 核心框架
├── LangGraph          # 状态图和工作流
├── LangSmith          # 调试和监控
├── LangServe          # API部署
└── LangChain Hub      # 模板和资源共享
```

### 1.3 应用场景

- **问答系统**：基于文档的智能问答
- **聊天机器人**：对话式AI助手
- **代码助手**：代码生成和解释
- **数据分析**：自然语言查询数据
- **内容生成**：文章、报告自动生成
- **工作流自动化**：复杂任务自动化

---

## 2. 环境搭建

### 2.1 安装LangChain

```bash
# 安装核心包
pip install langchain

# 安装OpenAI集成
pip install langchain-openai

# 安装社区集成
pip install langchain-community

# 安装完整版（包含所有依赖）
pip install langchain[all]
```

### 2.2 配置API密钥

```python
import os

# 设置OpenAI API密钥
os.environ["OPENAI_API_KEY"] = "your-api-key"

# 或使用.env文件
# 创建.env文件，添加：
# OPENAI_API_KEY=your-api-key

# 使用python-dotenv加载
from dotenv import load_dotenv
load_dotenv()
```

### 2.3 验证安装

```python
from langchain_openai import ChatOpenAI

# 创建LLM实例
llm = ChatOpenAI(model="gpt-3.5-turbo")

# 测试调用
response = llm.invoke("Hello, LangChain!")
print(response.content)
```

---

## 3. 核心概念

### 3.1 LangChain架构

```
LangChain核心组件
├── Models (模型)
│   ├── LLMs (大语言模型)
│   ├── Chat Models (聊天模型)
│   └── Embeddings (嵌入模型)
├── Prompts (提示词)
│   ├── Prompt Templates (模板)
│   └── Example Selectors (示例选择器)
├── Chains (链)
│   ├── LLMChain (基础链)
│   ├── Sequential Chain (顺序链)
│   └── Router Chain (路由链)
├── Agents (智能体)
│   ├── Agent Types (智能体类型)
│   └── Tools (工具)
├── Memory (记忆)
│   ├── Buffer Memory (缓冲记忆)
│   └── Vector Store Memory (向量存储记忆)
└── Callbacks (回调)
    ├── Logging (日志)
    └── Tracing (追踪)
```

### 3.2 核心概念解释

#### Models（模型）
- **LLMs**：文本生成模型（如GPT-3）
- **Chat Models**：对话模型（如ChatGPT）
- **Embeddings**：文本向量化模型

#### Prompts（提示词）
- 结构化的输入模板
- 支持变量替换
- 可复用的提示词设计

#### Chains（链）
- 将多个组件串联
- 实现复杂的处理流程
- 支持条件分支和循环

#### Agents（智能体）
- 自主决策的AI系统
- 可以使用工具
- 动态规划执行步骤

#### Memory（记忆）
- 保存对话历史
- 上下文管理
- 长期记忆存储

---

## 4. LLM集成

### 4.1 OpenAI集成

```python
from langchain_openai import ChatOpenAI, OpenAI

# 🔥 Chat模型（推荐）
chat_model = ChatOpenAI(
    model="gpt-3.5-turbo",
    temperature=0.7,
    max_tokens=1000
)

# 基础LLM模型
llm = OpenAI(
    model="gpt-3.5-turbo-instruct",
    temperature=0.7
)

# 调用模型
from langchain_core.messages import HumanMessage, SystemMessage

messages = [
    SystemMessage(content="你是一个有帮助的AI助手"),
    HumanMessage(content="什么是LangChain？")
]

response = chat_model.invoke(messages)
print(response.content)
```

### 4.2 其他模型集成

```python
# Anthropic Claude
from langchain_anthropic import ChatAnthropic
claude = ChatAnthropic(model="claude-3-opus-20240229")

# Google Gemini
from langchain_google_genai import ChatGoogleGenerativeAI
gemini = ChatGoogleGenerativeAI(model="gemini-pro")

# HuggingFace
from langchain_community.llms import HuggingFaceHub
hf_llm = HuggingFaceHub(
    repo_id="google/flan-t5-large",
    model_kwargs={"temperature": 0.7}
)

# 本地模型（Ollama）
from langchain_community.llms import Ollama
local_llm = Ollama(model="llama2")
```

### 4.3 流式输出

```python
# 🔥 流式响应
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(model="gpt-3.5-turbo", streaming=True)

for chunk in llm.stream("讲一个笑话"):
    print(chunk.content, end="", flush=True)
```

---

## 5. Prompt模板

### 5.1 基础模板

```python
from langchain_core.prompts import PromptTemplate

# 🔥 创建Prompt模板
template = """
你是一个{role}。
请回答以下问题：{question}
"""

prompt = PromptTemplate(
    input_variables=["role", "question"],
    template=template
)

# 格式化Prompt
formatted_prompt = prompt.format(
    role="Python专家",
    question="什么是装饰器？"
)
print(formatted_prompt)
```

### 5.2 Chat Prompt模板

```python
from langchain_core.prompts import ChatPromptTemplate

# 🔥 Chat模板
chat_template = ChatPromptTemplate.from_messages([
    ("system", "你是一个{role}"),
    ("human", "{question}"),
])

# 格式化
messages = chat_template.format_messages(
    role="Python专家",
    question="什么是装饰器？"
)

# 调用模型
response = chat_model.invoke(messages)
print(response.content)
```

### 5.3 Few-shot Prompt

```python
from langchain_core.prompts import FewShotPromptTemplate

# 示例
examples = [
    {"input": "happy", "output": "sad"},
    {"input": "tall", "output": "short"},
    {"input": "hot", "output": "cold"},
]

# 示例模板
example_template = """
Input: {input}
Output: {output}
"""

example_prompt = PromptTemplate(
    input_variables=["input", "output"],
    template=example_template
)

# Few-shot模板
few_shot_prompt = FewShotPromptTemplate(
    examples=examples,
    example_prompt=example_prompt,
    prefix="给出以下词的反义词：",
    suffix="Input: {input}\nOutput:",
    input_variables=["input"]
)

# 使用
print(few_shot_prompt.format(input="big"))
```

---

## 6. Chain链式调用

### 6.1 LLMChain

```python
from langchain.chains import LLMChain
from langchain_openai import ChatOpenAI
from langchain_core.prompts import PromptTemplate

# 🔥 创建LLMChain
llm = ChatOpenAI(model="gpt-3.5-turbo")

prompt = PromptTemplate(
    input_variables=["product"],
    template="为{product}写一个广告语"
)

chain = LLMChain(llm=llm, prompt=prompt)

# 运行Chain
result = chain.run(product="智能手表")
print(result)
```

### 6.2 Sequential Chain

```python
from langchain.chains import SequentialChain

# 第一个Chain：生成剧本大纲
synopsis_chain = LLMChain(
    llm=llm,
    prompt=PromptTemplate(
        input_variables=["title"],
        template="为电影《{title}》写一个剧本大纲"
    ),
    output_key="synopsis"
)

# 第二个Chain：生成评论
review_chain = LLMChain(
    llm=llm,
    prompt=PromptTemplate(
        input_variables=["synopsis"],
        template="为以下剧本写一个评论：\n{synopsis}"
    ),
    output_key="review"
)

# 🔥 组合Chain
overall_chain = SequentialChain(
    chains=[synopsis_chain, review_chain],
    input_variables=["title"],
    output_variables=["synopsis", "review"]
)

# 运行
result = overall_chain({"title": "AI的未来"})
print(result["review"])
```

### 6.3 LCEL (LangChain Expression Language)

```python
# 🔥 现代化的Chain写法（推荐）
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import ChatPromptTemplate
from langchain_openai import ChatOpenAI

# 定义组件
prompt = ChatPromptTemplate.from_template("讲一个关于{topic}的笑话")
model = ChatOpenAI(model="gpt-3.5-turbo")
output_parser = StrOutputParser()

# 使用管道操作符组合
chain = prompt | model | output_parser

# 调用
result = chain.invoke({"topic": "程序员"})
print(result)

# 流式调用
for chunk in chain.stream({"topic": "程序员"}):
    print(chunk, end="", flush=True)
```

---

## 7. Agent智能体

### 7.1 Agent基础

```python
from langchain.agents import AgentExecutor, create_openai_functions_agent
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder
from langchain.tools import Tool

# 定义工具
def get_weather(location: str) -> str:
    """获取天气信息"""
    return f"{location}的天气是晴天，温度25度"

def calculate(expression: str) -> str:
    """计算数学表达式"""
    try:
        return str(eval(expression))
    except:
        return "计算错误"

tools = [
    Tool(
        name="Weather",
        func=get_weather,
        description="获取指定地点的天气信息。输入应该是地点名称。"
    ),
    Tool(
        name="Calculator",
        func=calculate,
        description="计算数学表达式。输入应该是数学表达式。"
    )
]

# 🔥 创建Agent
llm = ChatOpenAI(model="gpt-3.5-turbo", temperature=0)

prompt = ChatPromptTemplate.from_messages([
    ("system", "你是一个有帮助的AI助手"),
    ("human", "{input}"),
    MessagesPlaceholder(variable_name="agent_scratchpad"),
])

agent = create_openai_functions_agent(llm, tools, prompt)
agent_executor = AgentExecutor(agent=agent, tools=tools, verbose=True)

# 运行Agent
result = agent_executor.invoke({"input": "北京的天气怎么样？"})
print(result["output"])

result = agent_executor.invoke({"input": "计算 25 * 4 + 10"})
print(result["output"])
```

### 7.2 ReAct Agent

```python
from langchain.agents import create_react_agent, AgentExecutor
from langchain import hub

# 使用Hub中的ReAct prompt
prompt = hub.pull("hwchase17/react")

# 创建ReAct Agent
agent = create_react_agent(llm, tools, prompt)
agent_executor = AgentExecutor(
    agent=agent,
    tools=tools,
    verbose=True,
    handle_parsing_errors=True
)

# 运行
result = agent_executor.invoke({
    "input": "北京的天气怎么样？如果温度超过20度，计算20*2"
})
```

---

## 8. Memory记忆

### 8.1 ConversationBufferMemory

```python
from langchain.memory import ConversationBufferMemory
from langchain.chains import ConversationChain
from langchain_openai import ChatOpenAI

# 🔥 创建记忆
memory = ConversationBufferMemory()

# 创建对话链
llm = ChatOpenAI(model="gpt-3.5-turbo")
conversation = ConversationChain(
    llm=llm,
    memory=memory,
    verbose=True
)

# 多轮对话
print(conversation.predict(input="你好，我叫Alice"))
print(conversation.predict(input="我喜欢Python编程"))
print(conversation.predict(input="我叫什么名字？"))  # 会记住之前的对话
```

### 8.2 ConversationBufferWindowMemory

```python
from langchain.memory import ConversationBufferWindowMemory

# 只保留最近k轮对话
memory = ConversationBufferWindowMemory(k=2)

conversation = ConversationChain(
    llm=llm,
    memory=memory,
    verbose=True
)
```

### 8.3 ConversationSummaryMemory

```python
from langchain.memory import ConversationSummaryMemory

# 🔥 自动总结对话历史
memory = ConversationSummaryMemory(llm=llm)

conversation = ConversationChain(
    llm=llm,
    memory=memory,
    verbose=True
)
```

---

## 9. 工具集成

### 9.1 内置工具

```python
from langchain.tools import WikipediaQueryRun
from langchain_community.utilities import WikipediaAPIWrapper

# Wikipedia工具
wikipedia = WikipediaQueryRun(api_wrapper=WikipediaAPIWrapper())

# 使用工具
result = wikipedia.run("Python programming language")
print(result)
```

### 9.2 自定义工具

```python
from langchain.tools import tool

# 🔥 使用装饰器创建工具
@tool
def search_database(query: str) -> str:
    """在数据库中搜索信息"""
    # 实际的数据库查询逻辑
    return f"搜索结果：{query}"

@tool
def send_email(to: str, subject: str, body: str) -> str:
    """发送邮件"""
    return f"邮件已发送到{to}"

# 将工具添加到Agent
tools = [search_database, send_email]
```

---

## 10. 回调和监控

### 10.1 回调处理器

```python
from langchain.callbacks import StdOutCallbackHandler
from langchain_openai import ChatOpenAI

# 🔥 使用回调
llm = ChatOpenAI(
    model="gpt-3.5-turbo",
    callbacks=[StdOutCallbackHandler()]
)

response = llm.invoke("Hello")
```

### 10.2 自定义回调

```python
from langchain.callbacks.base import BaseCallbackHandler

class MyCallbackHandler(BaseCallbackHandler):
    def on_llm_start(self, serialized, prompts, **kwargs):
        print(f"LLM开始: {prompts}")
    
    def on_llm_end(self, response, **kwargs):
        print(f"LLM结束: {response}")
    
    def on_llm_error(self, error, **kwargs):
        print(f"LLM错误: {error}")

# 使用自定义回调
llm = ChatOpenAI(callbacks=[MyCallbackHandler()])
```

---

## 11. 最佳实践

### 11.1 错误处理

```python
from langchain.schema import OutputParserException

try:
    result = chain.invoke({"input": "test"})
except OutputParserException as e:
    print(f"解析错误: {e}")
except Exception as e:
    print(f"其他错误: {e}")
```

### 11.2 性能优化

```python
# 使用缓存
from langchain.cache import InMemoryCache
from langchain.globals import set_llm_cache

set_llm_cache(InMemoryCache())

# 批量处理
results = llm.batch([
    "问题1",
    "问题2",
    "问题3"
])
```

---

## 12. 实战案例

### 12.1 简单问答系统

```python
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser

# 构建Chain
prompt = ChatPromptTemplate.from_template("回答问题：{question}")
model = ChatOpenAI(model="gpt-3.5-turbo")
output_parser = StrOutputParser()

chain = prompt | model | output_parser

# 使用
answer = chain.invoke({"question": "什么是LangChain？"})
print(answer)
```

---

## 📝 学习检查清单

- [ ] 理解LangChain的核心概念
- [ ] 能够集成不同的LLM
- [ ] 掌握Prompt模板的使用
- [ ] 能够构建复杂的Chain
- [ ] 理解Agent的工作原理
- [ ] 掌握Memory管理
- [ ] 能够集成和创建工具
- [ ] 了解回调和监控机制

---

## 🔗 相关资源

- [LangChain官方文档](https://python.langchain.com/)
- [LangChain GitHub](https://github.com/langchain-ai/langchain)
- [LangChain教程](https://python.langchain.com/docs/get_started/introduction)

---

@author erik.zhou
