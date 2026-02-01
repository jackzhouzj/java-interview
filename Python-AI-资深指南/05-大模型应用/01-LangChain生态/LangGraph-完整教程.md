# LangGraph - 完整教程

> @author erik.zhou

## 📚 技术概述

### 版本信息
- **LangGraph版本**：1.0+
- **最新稳定版**：1.0.x
- **推荐版本**：1.0.0+（2024-2025最新版本）

### 学习难度
- **难度等级**：⭐⭐⭐⭐ (较难)
- **预计学习时间**：15-20小时
- **重要程度**：⭐⭐⭐⭐⭐ (必学)

### 前置知识
- LangChain 0.3+基础
- Python 3.9+异步编程
- 状态机概念
- Agent开发经验
- TypedDict和类型注解

## 🎯 学习目标

完成本教程后，你将能够：

- [ ] 理解LangGraph的核心概念
- [ ] 掌握状态图的构建方法
- [ ] 能够实现循环和条件流程
- [ ] 掌握多Agent协作开发
- [ ] 理解人机交互流程设计
- [ ] 能够构建复杂的工作流
- [ ] 掌握状态持久化和检查点

## 📖 目录

1. [LangGraph简介](#1-langgraph简介)
2. [环境搭建](#2-环境搭建)
3. [核心概念](#3-核心概念)
4. [状态图基础](#4-状态图基础)
5. [节点和边](#5-节点和边)
6. [条件路由](#6-条件路由)
7. [循环和迭代](#7-循环和迭代)
8. [多Agent协作](#8-多agent协作)
9. [人机交互](#9-人机交互)
10. [持久化和检查点](#10-持久化和检查点)
11. [实战案例](#11-实战案例)

---

## 1. LangGraph简介

### 1.1 什么是LangGraph

LangGraph是LangChain生态系统中用于构建有状态、多参与者应用的库。它扩展了LangChain的能力，使得可以创建具有循环和条件的复杂工作流。

**核心特性**：
- 🔥 **状态管理**：内置状态管理机制
- 🔥 **循环支持**：支持循环和迭代流程
- 🔥 **条件分支**：基于状态的条件路由
- 🔥 **多Agent**：多个Agent协作
- 🔥 **人机交互**：支持人工介入
- 🔥 **持久化**：状态持久化和恢复

### 1.2 LangGraph vs LangChain

| 特性 | LangChain | LangGraph |
|------|-----------|-----------|
| 流程类型 | 线性/顺序 | 图状/循环 |
| 状态管理 | 有限 | 强大 |
| 条件分支 | 基础 | 高级 |
| 循环支持 | 不支持 | 支持 |
| 多Agent | 基础 | 高级 |
| 人机交互 | 有限 | 原生支持 |

### 1.3 应用场景

- **复杂工作流**：需要循环和条件的流程
- **多Agent系统**：多个Agent协作完成任务
- **人机协作**：需要人工审核或介入
- **状态机应用**：基于状态的决策系统
- **游戏AI**：复杂的游戏逻辑
- **自动化流程**：企业级工作流自动化

---

## 2. 环境搭建

### 2.1 安装LangGraph

```bash
# 🔥 安装LangGraph（最新版本）
pip install langgraph

# 安装相关依赖
pip install langchain-core langchain-openai

# 可选：安装检查点存储
pip install langgraph-checkpoint-sqlite  # SQLite存储
pip install langgraph-checkpoint-postgres  # PostgreSQL存储
```

### 2.2 基础配置

```python
import os
from dotenv import load_dotenv

# 加载环境变量
load_dotenv()

# 设置API密钥
os.environ["OPENAI_API_KEY"] = "your-api-key"
```

---

## 3. 核心概念

### 3.1 LangGraph架构

```
LangGraph核心组件
├── StateGraph (状态图)
│   ├── State (状态)
│   ├── Nodes (节点)
│   └── Edges (边)
├── Conditional Edges (条件边)
├── Checkpointer (检查点)
└── Human-in-the-Loop (人机交互)
```

### 3.2 核心概念解释

#### State（状态）
- 图中共享的数据结构
- 在节点间传递
- 可以被节点修改

#### Node（节点）
- 执行具体操作的函数
- 接收状态作为输入
- 返回状态更新

#### Edge（边）
- 连接节点的路径
- 定义执行顺序
- 可以是条件的

#### Conditional Edge（条件边）
- 基于状态的动态路由
- 实现条件分支
- 支持复杂决策逻辑

---

## 4. 状态图基础

### 4.1 创建简单状态图

```python
from typing import TypedDict, Annotated
from langgraph.graph import StateGraph, START, END
import operator

# 🔥 定义状态（使用TypedDict）
class State(TypedDict):
    messages: Annotated[list[str], operator.add]  # 使用operator.add自动合并列表
    count: int

# 定义节点函数
def node_1(state: State) -> dict:
    """第一个节点"""
    print("执行节点1")
    return {
        "messages": ["节点1执行"],
        "count": state["count"] + 1
    }

def node_2(state: State) -> dict:
    """第二个节点"""
    print("执行节点2")
    return {
        "messages": ["节点2执行"],
        "count": state["count"] + 1
    }

# 🔥 创建状态图（新API）
workflow = StateGraph(State)

# 添加节点
workflow.add_node("node_1", node_1)
workflow.add_node("node_2", node_2)

# 🔥 添加边（使用START和END常量）
workflow.add_edge(START, "node_1")
workflow.add_edge("node_1", "node_2")
workflow.add_edge("node_2", END)

# 编译图
app = workflow.compile()

# 运行
initial_state = {"messages": [], "count": 0}
result = app.invoke(initial_state)
print(result)
# 输出：{'messages': ['节点1执行', '节点2执行'], 'count': 2}
```

### 4.2 可视化状态图

```python
from IPython.display import Image, display

# 生成图的可视化
try:
    display(Image(app.get_graph().draw_mermaid_png()))
except Exception:
    # 如果无法显示，打印Mermaid代码
    print(app.get_graph().draw_mermaid())
```

---

## 5. 节点和边

### 5.1 节点类型

```python
from langchain_openai import ChatOpenAI
from langchain_core.messages import HumanMessage, AIMessage
from typing import TypedDict

class State(TypedDict):
    messages: list
    input: str
    output: str
    count: int
    should_continue: bool

# 🔥 LLM节点
def llm_node(state: State) -> dict:
    """使用LLM的节点"""
    llm = ChatOpenAI(model="gpt-4o-mini")
    
    messages = [HumanMessage(content=state["input"])]
    response = llm.invoke(messages)
    
    return {
        "messages": [response],
        "output": response.content
    }

# 🔥 工具节点
def tool_node(state: State) -> dict:
    """使用工具的节点"""
    # 调用外部工具或API
    result = f"处理结果：{state['input']}"
    return {"output": result}

# 🔥 决策节点
def decision_node(state: State) -> dict:
    """做出决策的节点"""
    should_continue = state["count"] < 5
    return {"should_continue": should_continue}

# 🔥 带重试策略的节点
from langgraph.types import RetryPolicy

workflow.add_node(
    "api_call",
    api_call_function,
    retry_policy=RetryPolicy(
        max_attempts=3,  # 最多重试3次
        backoff_factor=2.0  # 指数退避
    )
)
```

### 5.2 边的类型

```python
from langgraph.graph import START, END

# 🔥 普通边（固定路由）
workflow.add_edge("node_a", "node_b")

# 🔥 入口边（使用START常量）
workflow.add_edge(START, "start_node")

# 🔥 结束边（使用END常量）
workflow.add_edge("final_node", END)

# 🔥 条件边（动态路由）
workflow.add_conditional_edges(
    "decision_node",
    lambda state: "continue" if state["should_continue"] else "end",
    {
        "continue": "next_node",
        "end": END
    }
)

# 🔥 多路条件边
def route_by_type(state: State) -> str:
    """根据类型路由"""
    return state["type"]

workflow.add_conditional_edges(
    "classifier",
    route_by_type,
    {
        "type_a": "handler_a",
        "type_b": "handler_b",
        "type_c": "handler_c",
    }
)
```

---

## 6. 条件路由

### 6.1 基于状态的路由

```python
from typing import Literal

# 🔥 定义路由函数
def route_based_on_count(state: State) -> Literal["continue", "end"]:
    """基于计数器的路由"""
    if state["count"] < 3:
        return "continue"
    return "end"

# 添加条件边
workflow.add_conditional_edges(
    "counter_node",
    route_based_on_count,
    {
        "continue": "process_node",
        "end": END
    }
)
```

### 6.2 复杂条件路由

```python
def complex_router(state: State) -> str:
    """复杂的路由逻辑"""
    if state["error"]:
        return "error_handler"
    elif state["needs_review"]:
        return "human_review"
    elif state["is_complete"]:
        return "finalize"
    else:
        return "continue_processing"

workflow.add_conditional_edges(
    "decision_node",
    complex_router,
    {
        "error_handler": "error_node",
        "human_review": "review_node",
        "finalize": "final_node",
        "continue_processing": "process_node"
    }
)
```

---

## 7. 循环和迭代

### 7.1 简单循环

```python
from typing import TypedDict

class LoopState(TypedDict):
    count: int
    max_iterations: int
    result: str

def loop_node(state: LoopState) -> LoopState:
    """循环处理节点"""
    print(f"迭代 {state['count']}")
    return {
        "count": state["count"] + 1,
        "result": state["result"] + f"迭代{state['count']} "
    }

def should_continue(state: LoopState) -> Literal["continue", "end"]:
    """判断是否继续循环"""
    if state["count"] < state["max_iterations"]:
        return "continue"
    return "end"

# 🔥 构建循环图
workflow = StateGraph(LoopState)
workflow.add_node("loop", loop_node)

workflow.add_conditional_edges(
    "loop",
    should_continue,
    {
        "continue": "loop",  # 循环回自己
        "end": END
    }
)

workflow.set_entry_point("loop")
app = workflow.compile()

# 运行
result = app.invoke({
    "count": 0,
    "max_iterations": 5,
    "result": ""
})
print(result)
```

### 7.2 Agent循环

```python
from langchain_openai import ChatOpenAI
from langchain.agents import AgentExecutor, create_openai_functions_agent
from langchain_core.messages import HumanMessage

class AgentState(TypedDict):
    messages: list
    iterations: int

def agent_node(state: AgentState) -> AgentState:
    """Agent执行节点"""
    # 创建Agent
    llm = ChatOpenAI(model="gpt-3.5-turbo")
    # ... Agent逻辑
    
    return {
        "messages": state["messages"] + [response],
        "iterations": state["iterations"] + 1
    }

def should_continue_agent(state: AgentState) -> str:
    """判断Agent是否应该继续"""
    last_message = state["messages"][-1]
    
    # 如果是最终答案，结束
    if "FINAL ANSWER" in last_message.content:
        return "end"
    
    # 如果迭代次数过多，结束
    if state["iterations"] > 10:
        return "end"
    
    return "continue"

# 构建Agent循环
workflow = StateGraph(AgentState)
workflow.add_node("agent", agent_node)

workflow.add_conditional_edges(
    "agent",
    should_continue_agent,
    {
        "continue": "agent",
        "end": END
    }
)

workflow.set_entry_point("agent")
app = workflow.compile()
```

---

## 8. 多Agent协作

### 8.1 顺序协作

```python
from typing import TypedDict

class MultiAgentState(TypedDict):
    task: str
    research_result: str
    writing_result: str
    review_result: str

def research_agent(state: MultiAgentState) -> MultiAgentState:
    """研究Agent"""
    llm = ChatOpenAI(model="gpt-3.5-turbo")
    prompt = f"研究以下主题：{state['task']}"
    result = llm.invoke(prompt)
    
    return {"research_result": result.content}

def writing_agent(state: MultiAgentState) -> MultiAgentState:
    """写作Agent"""
    llm = ChatOpenAI(model="gpt-3.5-turbo")
    prompt = f"基于以下研究结果写一篇文章：\n{state['research_result']}"
    result = llm.invoke(prompt)
    
    return {"writing_result": result.content}

def review_agent(state: MultiAgentState) -> MultiAgentState:
    """审核Agent"""
    llm = ChatOpenAI(model="gpt-3.5-turbo")
    prompt = f"审核以下文章：\n{state['writing_result']}"
    result = llm.invoke(prompt)
    
    return {"review_result": result.content}

# 🔥 构建多Agent工作流
workflow = StateGraph(MultiAgentState)

workflow.add_node("research", research_agent)
workflow.add_node("writing", writing_agent)
workflow.add_node("review", review_agent)

workflow.add_edge("research", "writing")
workflow.add_edge("writing", "review")
workflow.add_edge("review", END)

workflow.set_entry_point("research")
app = workflow.compile()

# 运行
result = app.invoke({"task": "人工智能的未来"})
print(result["review_result"])
```

### 8.2 并行协作

```python
from typing import TypedDict, Annotated
import operator

class ParallelState(TypedDict):
    task: str
    results: Annotated[list, operator.add]  # 合并多个结果

def agent_1(state: ParallelState) -> ParallelState:
    """Agent 1"""
    result = f"Agent1处理: {state['task']}"
    return {"results": [result]}

def agent_2(state: ParallelState) -> ParallelState:
    """Agent 2"""
    result = f"Agent2处理: {state['task']}"
    return {"results": [result]}

def aggregator(state: ParallelState) -> ParallelState:
    """聚合结果"""
    combined = "\n".join(state["results"])
    return {"final_result": combined}

# 构建并行工作流
workflow = StateGraph(ParallelState)

workflow.add_node("agent_1", agent_1)
workflow.add_node("agent_2", agent_2)
workflow.add_node("aggregator", aggregator)

# 并行执行
workflow.add_edge("agent_1", "aggregator")
workflow.add_edge("agent_2", "aggregator")
workflow.add_edge("aggregator", END)

# 设置多个入口点（并行）
workflow.set_entry_point("agent_1")
workflow.set_entry_point("agent_2")

app = workflow.compile()
```

---

## 9. 人机交互

### 9.1 人工审核节点（最新API）

```python
from langgraph.checkpoint.memory import MemorySaver
from langgraph.graph import StateGraph, START, END
from typing import TypedDict

class ReviewState(TypedDict):
    content: str
    approved: bool
    feedback: str

def generate_content(state: ReviewState) -> dict:
    """生成内容"""
    from langchain_openai import ChatOpenAI
    llm = ChatOpenAI(model="gpt-4o-mini")
    content = llm.invoke("写一篇关于AI的文章").content
    return {"content": content}

def human_review(state: ReviewState) -> dict:
    """人工审核（这里会暂停等待人工输入）"""
    print(f"请审核以下内容：\n{state['content']}")
    # 在实际应用中，这里会等待外部输入
    # 通过interrupt()暂停执行
    return state

def finalize(state: ReviewState) -> dict:
    """最终处理"""
    if state.get("approved"):
        return {"result": "内容已发布"}
    else:
        return {"result": f"内容被拒绝，原因：{state.get('feedback', '未提供')}"}

# 🔥 使用检查点支持人机交互
memory = MemorySaver()

workflow = StateGraph(ReviewState)
workflow.add_node("generate", generate_content)
workflow.add_node("review", human_review)
workflow.add_node("finalize", finalize)

workflow.add_edge(START, "generate")
workflow.add_edge("generate", "review")

# 🔥 条件边：根据审核结果决定下一步
def should_finalize(state: ReviewState) -> str:
    if state.get("approved") is not None:
        return "finalize"
    return END

workflow.add_conditional_edges(
    "review",
    should_finalize,
    {
        "finalize": "finalize",
        END: END
    }
)
workflow.add_edge("finalize", END)

# 使用检查点编译
app = workflow.compile(checkpointer=memory)

# 🔥 第一次运行（会在review节点暂停）
config = {"configurable": {"thread_id": "review-1"}}
result = app.invoke({"content": "", "approved": None, "feedback": ""}, config)

# 🔥 人工审核后继续（更新状态）
# 方式1：批准
app.update_state(config, {"approved": True})
result = app.invoke(None, config)

# 方式2：拒绝
app.update_state(config, {"approved": False, "feedback": "内容需要改进"})
result = app.invoke(None, config)
```

---

## 10. 持久化和检查点

### 10.1 使用检查点

```python
from langgraph.checkpoint.memory import MemorySaver
from langgraph.graph import StateGraph, START, END

# 🔥 创建检查点（内存存储）
checkpointer = MemorySaver()

# 编译时添加检查点
app = workflow.compile(checkpointer=checkpointer)

# 🔥 使用配置运行（必须提供thread_id）
config = {"configurable": {"thread_id": "conversation-1"}}
result = app.invoke(initial_state, config)

# 继续之前的会话（使用相同的thread_id）
result = app.invoke(new_input, config)

# 🔥 使用SQLite持久化存储（推荐生产环境）
from langgraph.checkpoint.sqlite import SqliteSaver

# 创建SQLite检查点
checkpointer = SqliteSaver.from_conn_string("checkpoints.db")
app = workflow.compile(checkpointer=checkpointer)
```

### 10.2 获取状态历史和时间旅行

```python
# 🔥 获取所有检查点（状态历史）
for checkpoint in app.get_state_history(config):
    print(f"步骤: {checkpoint.metadata.get('step')}")
    print(f"状态: {checkpoint.values}")
    print("---")

# 🔥 获取当前状态
current_state = app.get_state(config)
print(f"当前状态: {current_state.values}")
print(f"下一个节点: {current_state.next}")

# 🔥 时间旅行：回到之前的检查点
# 获取特定检查点ID
checkpoint_id = "specific-checkpoint-id"
config_with_checkpoint = {
    "configurable": {
        "thread_id": "conversation-1",
        "checkpoint_id": checkpoint_id
    }
}
result = app.invoke(new_input, config_with_checkpoint)
```

---

## 11. 实战案例

### 11.1 客服机器人

```python
from typing import TypedDict, Literal
from langchain_openai import ChatOpenAI
from langchain_core.messages import HumanMessage, AIMessage

class CustomerServiceState(TypedDict):
    messages: list
    category: str
    resolved: bool

def classify_query(state: CustomerServiceState) -> CustomerServiceState:
    """分类用户查询"""
    llm = ChatOpenAI(model="gpt-3.5-turbo")
    
    prompt = f"""
    分类以下用户查询到以下类别之一：
    - technical: 技术问题
    - billing: 账单问题
    - general: 一般咨询
    
    用户查询: {state['messages'][-1].content}
    
    只返回类别名称。
    """
    
    category = llm.invoke(prompt).content.strip().lower()
    return {"category": category}

def handle_technical(state: CustomerServiceState) -> CustomerServiceState:
    """处理技术问题"""
    llm = ChatOpenAI(model="gpt-3.5-turbo")
    response = llm.invoke(f"技术支持: {state['messages'][-1].content}")
    return {
        "messages": state["messages"] + [AIMessage(content=response.content)],
        "resolved": True
    }

def handle_billing(state: CustomerServiceState) -> CustomerServiceState:
    """处理账单问题"""
    llm = ChatOpenAI(model="gpt-3.5-turbo")
    response = llm.invoke(f"账单咨询: {state['messages'][-1].content}")
    return {
        "messages": state["messages"] + [AIMessage(content=response.content)],
        "resolved": True
    }

def handle_general(state: CustomerServiceState) -> CustomerServiceState:
    """处理一般咨询"""
    llm = ChatOpenAI(model="gpt-3.5-turbo")
    response = llm.invoke(f"一般咨询: {state['messages'][-1].content}")
    return {
        "messages": state["messages"] + [AIMessage(content=response.content)],
        "resolved": True
    }

def route_by_category(state: CustomerServiceState) -> str:
    """根据类别路由"""
    return state["category"]

# 🔥 构建客服机器人
workflow = StateGraph(CustomerServiceState)

workflow.add_node("classify", classify_query)
workflow.add_node("technical", handle_technical)
workflow.add_node("billing", handle_billing)
workflow.add_node("general", handle_general)

workflow.set_entry_point("classify")

workflow.add_conditional_edges(
    "classify",
    route_by_category,
    {
        "technical": "technical",
        "billing": "billing",
        "general": "general"
    }
)

workflow.add_edge("technical", END)
workflow.add_edge("billing", END)
workflow.add_edge("general", END)

app = workflow.compile()

# 使用
result = app.invoke({
    "messages": [HumanMessage(content="我的账单有问题")],
    "category": "",
    "resolved": False
})

print(result["messages"][-1].content)
```

---

## 📝 学习检查清单

- [ ] 理解LangGraph的核心概念
- [ ] 能够构建状态图
- [ ] 掌握条件路由的使用
- [ ] 能够实现循环流程
- [ ] 理解多Agent协作模式
- [ ] 掌握人机交互实现
- [ ] 了解状态持久化机制

---

## 🔗 相关资源

- [LangGraph官方文档](https://langchain-ai.github.io/langgraph/)
- [LangGraph GitHub](https://github.com/langchain-ai/langgraph)
- [LangGraph示例](https://github.com/langchain-ai/langgraph/tree/main/examples)

---

@author erik.zhou
