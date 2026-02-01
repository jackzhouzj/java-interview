# CrewAI - 完整教程

> @author erik.zhou

## 📚 技术概述

### 版本信息
- **CrewAI版本**：0.175.0+
- **最新稳定版**：0.175.0+
- **推荐版本**：0.175.0+（2024-2025最新版本）
- **Python要求**：>=3.10 and <3.14

### 学习难度
- **难度等级**：⭐⭐⭐⭐ (较难)
- **预计学习时间**：20-25小时
- **重要程度**：⭐⭐⭐⭐⭐ (必学)

### 前置知识
- Python 3.10+基础
- LangChain 0.3+基础
- Agent开发经验
- 面向对象编程
- 异步编程基础

## 🎯 学习目标

完成本教程后，你将能够：

- [ ] 理解CrewAI的核心概念
- [ ] 掌握Agent和Task的定义
- [ ] 能够构建多Agent团队
- [ ] 理解Agent协作机制
- [ ] 掌握工具集成方法
- [ ] 能够设计复杂的工作流
- [ ] 了解最佳实践和优化技巧

## 📖 目录

1. [CrewAI简介](#1-crewai简介)
2. [环境搭建](#2-环境搭建)
3. [核心概念](#3-核心概念)
4. [创建Agent](#4-创建agent)
5. [定义Task](#5-定义task)
6. [构建Crew](#6-构建crew)
7. [工具集成](#7-工具集成)
8. [协作模式](#8-协作模式)
9. [实战案例](#9-实战案例)
10. [最佳实践](#10-最佳实践)

---

## 1. CrewAI简介

### 1.1 什么是CrewAI

CrewAI是一个用于编排角色扮演、自主AI Agent的框架。它使Agent能够无缝协作，处理复杂任务。

**核心特性**：
- 🔥 **角色扮演**：每个Agent有明确的角色和目标
- 🔥 **自主协作**：Agent自主决策和协作
- 🔥 **任务分配**：智能任务分配和执行
- 🔥 **工具使用**：丰富的工具集成
- 🔥 **灵活编排**：多种协作模式

### 1.2 CrewAI vs 其他框架

| 特性 | CrewAI | AutoGPT | LangGraph |
|------|--------|---------|-----------|
| 多Agent | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ |
| 角色定义 | ⭐⭐⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐ |
| 易用性 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ |
| 灵活性 | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| 协作能力 | ⭐⭐⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐⭐ |

### 1.3 应用场景

- **内容创作**：多Agent协作创作内容
- **研究分析**：研究团队协作分析
- **项目管理**：任务分配和执行
- **客户服务**：多角色客服系统
- **数据处理**：复杂数据处理流程

---

## 2. 环境搭建

### 2.1 安装CrewAI

```bash
# 🔥 检查Python版本（必须 >=3.10 and <3.14）
python3 --version

# 安装CrewAI
pip install crewai

# 安装工具包
pip install 'crewai[tools]'

# 🔥 安装相关依赖（CrewAI 0.175.0+ 需要 openai >= 1.13.3）
pip install "openai>=1.13.3"
pip install langchain-openai

# 验证安装
crewai version
crewai version --tools  # 查看工具版本
```

### 2.2 配置环境

```python
import os
from dotenv import load_dotenv

# 加载环境变量
load_dotenv()

# 配置API密钥
os.environ["OPENAI_API_KEY"] = "your-api-key"
```

---

## 3. 核心概念

### 3.1 CrewAI架构

```
CrewAI核心组件
├── Agent (智能体)
│   ├── Role (角色)
│   ├── Goal (目标)
│   ├── Backstory (背景故事)
│   └── Tools (工具)
├── Task (任务)
│   ├── Description (描述)
│   ├── Agent (执行者)
│   └── Expected Output (期望输出)
├── Crew (团队)
│   ├── Agents (Agent列表)
│   ├── Tasks (任务列表)
│   └── Process (执行流程)
└── Tools (工具)
    ├── Built-in Tools (内置工具)
    └── Custom Tools (自定义工具)
```

### 3.2 核心概念解释

#### Agent（智能体）
- 具有特定角色和目标的AI实体
- 可以使用工具完成任务
- 能够与其他Agent协作

#### Task（任务）
- 需要完成的具体工作
- 分配给特定Agent
- 有明确的期望输出

#### Crew（团队）
- Agent的集合
- 管理任务执行流程
- 协调Agent协作

---

## 4. 创建Agent

### 4.1 基础Agent

```python
from crewai import Agent
from langchain_openai import ChatOpenAI

# 🔥 创建LLM
llm = ChatOpenAI(model="gpt-4", temperature=0.7)

# 创建Agent
researcher = Agent(
    role="研究员",
    goal="深入研究给定的主题，提供详细的分析报告",
    backstory="""你是一位经验丰富的研究员，擅长收集和分析信息。
    你总是能够找到最相关和最新的信息，并以清晰的方式呈现。""",
    llm=llm,
    verbose=True,
    allow_delegation=False
)

# Agent属性
print(f"角色: {researcher.role}")
print(f"目标: {researcher.goal}")
```

### 4.2 带工具的Agent

```python
from crewai_tools import SerperDevTool, WebsiteSearchTool

# 创建工具
search_tool = SerperDevTool()
web_tool = WebsiteSearchTool()

# 创建带工具的Agent
researcher = Agent(
    role="研究员",
    goal="研究AI领域的最新进展",
    backstory="你是AI领域的专家研究员",
    tools=[search_tool, web_tool],
    llm=llm,
    verbose=True
)
```

### 4.3 Agent配置选项

```python
agent = Agent(
    role="角色名称",
    goal="Agent的目标",
    backstory="Agent的背景故事",
    llm=llm,
    tools=[],  # 工具列表
    verbose=True,  # 是否输出详细信息
    allow_delegation=True,  # 是否允许委托任务
    max_iter=15,  # 最大迭代次数
    max_rpm=10,  # 每分钟最大请求数
    memory=True  # 是否启用记忆
)
```

---

## 5. 定义Task

### 5.1 基础Task

```python
from crewai import Task

# 🔥 创建任务
research_task = Task(
    description="""
    研究2024年AI领域的最新进展，重点关注：
    1. 大语言模型的发展
    2. 多模态AI的应用
    3. AI Agent的进展
    
    提供详细的分析报告，包括关键发现和未来趋势。
    """,
    agent=researcher,
    expected_output="一份详细的研究报告，包含关键发现和趋势分析"
)
```

### 5.2 任务依赖

```python
# 任务1：研究
research_task = Task(
    description="研究AI领域的最新进展",
    agent=researcher,
    expected_output="研究报告"
)

# 任务2：写作（依赖任务1）
writing_task = Task(
    description="""
    基于研究报告，撰写一篇技术博客文章。
    文章应该：
    - 通俗易懂
    - 结构清晰
    - 包含实例
    """,
    agent=writer,
    expected_output="一篇技术博客文章",
    context=[research_task]  # 依赖research_task的输出
)
```

### 5.3 异步任务

```python
# 异步任务
async_task = Task(
    description="执行耗时的数据分析",
    agent=analyst,
    expected_output="分析结果",
    async_execution=True  # 异步执行
)
```

---

## 6. 构建Crew

### 6.1 基础Crew

```python
from crewai import Crew, Process

# 🔥 创建Crew
crew = Crew(
    agents=[researcher, writer],
    tasks=[research_task, writing_task],
    process=Process.sequential,  # 顺序执行
    verbose=True
)

# 执行Crew
result = crew.kickoff()
print(result)
```

### 6.2 执行流程

```python
# 顺序执行（Sequential）
crew_sequential = Crew(
    agents=[agent1, agent2, agent3],
    tasks=[task1, task2, task3],
    process=Process.sequential
)

# 层级执行（Hierarchical）
crew_hierarchical = Crew(
    agents=[agent1, agent2, agent3],
    tasks=[task1, task2, task3],
    process=Process.hierarchical,
    manager_llm=ChatOpenAI(model="gpt-4")  # 需要管理者LLM
)
```

### 6.3 Crew配置

```python
crew = Crew(
    agents=[researcher, writer, reviewer],
    tasks=[research_task, writing_task, review_task],
    process=Process.sequential,
    verbose=True,
    memory=True,  # 启用记忆
    cache=True,  # 启用缓存
    max_rpm=10,  # 每分钟最大请求数
    share_crew=False  # 是否共享Crew信息
)
```

---

## 7. 工具集成

### 7.1 内置工具

```python
from crewai_tools import (
    SerperDevTool,  # 搜索工具
    WebsiteSearchTool,  # 网站搜索
    FileReadTool,  # 文件读取
    DirectoryReadTool,  # 目录读取
    CodeInterpreterTool  # 代码解释器
)

# 使用内置工具
search_tool = SerperDevTool()
file_tool = FileReadTool()

agent = Agent(
    role="研究员",
    goal="研究和分析",
    backstory="专业研究员",
    tools=[search_tool, file_tool],
    llm=llm
)
```

### 7.2 自定义工具

```python
from crewai_tools import tool

# 🔥 使用装饰器创建工具
@tool("数据库查询工具")
def query_database(query: str) -> str:
    """
    在数据库中执行查询
    
    Args:
        query: SQL查询语句
    
    Returns:
        查询结果
    """
    # 实际的数据库查询逻辑
    result = f"查询结果: {query}"
    return result

# 使用自定义工具
agent = Agent(
    role="数据分析师",
    goal="分析数据",
    backstory="专业数据分析师",
    tools=[query_database],
    llm=llm
)
```

### 7.3 LangChain工具集成

```python
from langchain.tools import Tool
from langchain_community.utilities import WikipediaAPIWrapper

# 创建LangChain工具
wikipedia = WikipediaAPIWrapper()
wiki_tool = Tool(
    name="Wikipedia",
    func=wikipedia.run,
    description="用于搜索Wikipedia的工具"
)

# 在CrewAI中使用
agent = Agent(
    role="研究员",
    goal="研究历史事件",
    backstory="历史研究专家",
    tools=[wiki_tool],
    llm=llm
)
```

---

## 8. 协作模式

### 8.1 顺序协作

```python
# 🔥 顺序执行：一个接一个
crew = Crew(
    agents=[researcher, writer, editor],
    tasks=[research_task, write_task, edit_task],
    process=Process.sequential
)

# 执行流程：
# 1. researcher完成research_task
# 2. writer完成write_task（使用research_task的输出）
# 3. editor完成edit_task（使用write_task的输出）
```

### 8.2 层级协作

```python
# 层级执行：有管理者协调
crew = Crew(
    agents=[researcher, writer, analyst],
    tasks=[research_task, write_task, analyze_task],
    process=Process.hierarchical,
    manager_llm=ChatOpenAI(model="gpt-4")
)

# 执行流程：
# 1. 管理者分析任务
# 2. 管理者分配任务给合适的Agent
# 3. Agent执行任务
# 4. 管理者协调和整合结果
```

### 8.3 委托机制

```python
# Agent可以委托任务给其他Agent
senior_researcher = Agent(
    role="高级研究员",
    goal="领导研究项目",
    backstory="经验丰富的研究领导者",
    allow_delegation=True,  # 允许委托
    llm=llm
)

junior_researcher = Agent(
    role="初级研究员",
    goal="执行具体研究任务",
    backstory="勤奋的初级研究员",
    allow_delegation=False,
    llm=llm
)

# senior_researcher可以将子任务委托给junior_researcher
```

---

## 9. 实战案例

### 9.1 内容创作团队

```python
from crewai import Agent, Task, Crew, Process
from langchain_openai import ChatOpenAI

# 创建LLM
llm = ChatOpenAI(model="gpt-4", temperature=0.7)

# 🔥 定义Agent
researcher = Agent(
    role="内容研究员",
    goal="研究给定主题，收集相关信息和数据",
    backstory="""你是一位专业的内容研究员，擅长快速找到
    高质量的信息源，并提取关键要点。""",
    llm=llm,
    verbose=True
)

writer = Agent(
    role="内容作者",
    goal="基于研究结果创作引人入胜的文章",
    backstory="""你是一位才华横溢的作家，能够将复杂的
    信息转化为易于理解且有趣的内容。""",
    llm=llm,
    verbose=True
)

editor = Agent(
    role="编辑",
    goal="审核和优化文章，确保质量",
    backstory="""你是一位经验丰富的编辑，对细节有敏锐的
    洞察力，能够显著提升内容质量。""",
    llm=llm,
    verbose=True
)

# 定义任务
research_task = Task(
    description="""
    研究主题：人工智能在医疗领域的应用
    
    要求：
    1. 收集最新的研究和案例
    2. 分析主要应用场景
    3. 总结关键发现和趋势
    """,
    agent=researcher,
    expected_output="详细的研究报告，包含数据和案例"
)

write_task = Task(
    description="""
    基于研究报告，撰写一篇2000字的技术文章。
    
    要求：
    1. 结构清晰，逻辑连贯
    2. 包含具体案例和数据
    3. 语言通俗易懂
    4. 有吸引力的标题和开头
    """,
    agent=writer,
    expected_output="一篇完整的技术文章",
    context=[research_task]
)

edit_task = Task(
    description="""
    审核和优化文章。
    
    要求：
    1. 检查语法和拼写
    2. 优化句子结构
    3. 确保逻辑清晰
    4. 提升可读性
    """,
    agent=editor,
    expected_output="经过编辑的最终文章",
    context=[write_task]
)

# 创建Crew
content_crew = Crew(
    agents=[researcher, writer, editor],
    tasks=[research_task, write_task, edit_task],
    process=Process.sequential,
    verbose=True
)

# 执行
result = content_crew.kickoff()
print("\n最终文章：")
print(result)
```

### 9.2 客户服务团队

```python
# 客服Agent
support_agent = Agent(
    role="客服代表",
    goal="解决客户问题，提供优质服务",
    backstory="友好且专业的客服代表",
    llm=llm
)

# 技术支持Agent
tech_agent = Agent(
    role="技术支持",
    goal="解决技术问题",
    backstory="经验丰富的技术专家",
    llm=llm
)

# 任务
handle_inquiry = Task(
    description="处理客户咨询：{customer_inquiry}",
    agent=support_agent,
    expected_output="客户问题的解决方案"
)

# Crew
support_crew = Crew(
    agents=[support_agent, tech_agent],
    tasks=[handle_inquiry],
    process=Process.sequential
)

# 执行
result = support_crew.kickoff(inputs={
    "customer_inquiry": "我的账户无法登录"
})
```

---

## 10. 最佳实践

### 10.1 Agent设计

```python
# ✅ 好的Agent设计
good_agent = Agent(
    role="数据分析师",  # 明确的角色
    goal="分析销售数据，发现增长机会",  # 具体的目标
    backstory="""你是一位经验丰富的数据分析师，
    擅长从数据中发现洞察，曾帮助多家公司提升业绩。""",  # 详细的背景
    tools=[data_tool, chart_tool],  # 相关工具
    llm=llm,
    verbose=True
)

# ❌ 不好的Agent设计
bad_agent = Agent(
    role="助手",  # 角色太模糊
    goal="帮助用户",  # 目标不具体
    backstory="一个助手",  # 背景太简单
    llm=llm
)
```

### 10.2 任务设计

```python
# ✅ 好的任务设计
good_task = Task(
    description="""
    分析2024年Q1的销售数据。
    
    具体要求：
    1. 计算总销售额和增长率
    2. 识别top 10产品
    3. 分析地区分布
    4. 提供改进建议
    
    输出格式：结构化的分析报告
    """,
    agent=analyst,
    expected_output="包含数据、图表和建议的完整报告"
)

# ❌ 不好的任务设计
bad_task = Task(
    description="分析数据",  # 太模糊
    agent=analyst,
    expected_output="报告"  # 不具体
)
```

### 10.3 性能优化

```python
# 使用缓存
crew = Crew(
    agents=[agent1, agent2],
    tasks=[task1, task2],
    cache=True  # 启用缓存
)

# 控制请求频率
agent = Agent(
    role="研究员",
    goal="研究",
    backstory="研究员",
    max_rpm=10,  # 限制每分钟请求数
    llm=llm
)

# 使用更快的模型
fast_llm = ChatOpenAI(model="gpt-3.5-turbo")
agent = Agent(
    role="助手",
    goal="快速响应",
    backstory="助手",
    llm=fast_llm
)
```

---

## 📝 学习检查清单

- [ ] 理解CrewAI的核心概念
- [ ] 能够创建和配置Agent
- [ ] 掌握Task的定义和依赖
- [ ] 能够构建和执行Crew
- [ ] 掌握工具集成方法
- [ ] 理解不同的协作模式
- [ ] 能够设计复杂的多Agent系统

---

## 🔗 相关资源

- [CrewAI官方文档](https://docs.crewai.com/)
- [CrewAI GitHub](https://github.com/joaomdmoura/crewAI)
- [CrewAI示例](https://github.com/joaomdmoura/crewAI-examples)
- [CrewAI工具](https://github.com/joaomdmoura/crewAI-tools)

---

@author erik.zhou
