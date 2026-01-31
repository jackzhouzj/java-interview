# LangSmith - 完整教程

> @author erik.zhou

## 📚 技术概述

### 版本信息
- **LangSmith版本**：最新版
- **平台类型**：云服务
- **定价**：免费层 + 付费层

### 学习难度
- **难度等级**：⭐⭐⭐ (中等)
- **预计学习时间**：5-10小时
- **重要程度**：⭐⭐⭐⭐ (重要)

### 前置知识
- LangChain基础
- Python基础
- API调用经验

## 🎯 学习目标

完成本教程后，你将能够：

- [ ] 理解LangSmith的核心功能
- [ ] 掌握调试和追踪技巧
- [ ] 能够进行性能监控
- [ ] 掌握数据集管理
- [ ] 能够进行模型评估
- [ ] 了解生产环境监控
- [ ] 掌握团队协作功能

## 📖 目录

1. [LangSmith简介](#1-langsmith简介)
2. [环境搭建](#2-环境搭建)
3. [追踪和调试](#3-追踪和调试)
4. [数据集管理](#4-数据集管理)
5. [评估和测试](#5-评估和测试)
6. [监控和分析](#6-监控和分析)
7. [团队协作](#7-团队协作)
8. [最佳实践](#8-最佳实践)

---

## 1. LangSmith简介

### 1.1 什么是LangSmith

LangSmith是LangChain官方提供的开发者平台，用于调试、测试、评估和监控LLM应用。

**核心功能**：
- 🔥 **追踪调试**：详细的执行追踪
- 🔥 **性能监控**：实时性能指标
- 🔥 **数据集管理**：测试数据集管理
- 🔥 **评估测试**：自动化评估
- 🔥 **生产监控**：生产环境监控
- 🔥 **团队协作**：多人协作开发

### 1.2 为什么需要LangSmith

**开发阶段**：
- 调试复杂的Chain和Agent
- 追踪每一步的执行
- 快速定位问题

**测试阶段**：
- 管理测试数据集
- 自动化评估
- 回归测试

**生产阶段**：
- 监控应用性能
- 追踪用户反馈
- 持续优化

---

## 2. 环境搭建

### 2.1 注册LangSmith

1. 访问 https://smith.langchain.com/
2. 使用GitHub或Google账号注册
3. 创建组织和项目

### 2.2 获取API密钥

```python
# 在LangSmith平台获取API密钥
# Settings -> API Keys -> Create API Key
```

### 2.3 配置环境变量

```python
import os

# 🔥 配置LangSmith
os.environ["LANGCHAIN_TRACING_V2"] = "true"
os.environ["LANGCHAIN_API_KEY"] = "your-langsmith-api-key"
os.environ["LANGCHAIN_PROJECT"] = "your-project-name"

# 配置OpenAI
os.environ["OPENAI_API_KEY"] = "your-openai-api-key"
```

### 2.4 安装依赖

```bash
pip install langsmith langchain langchain-openai
```

---

## 3. 追踪和调试

### 3.1 基础追踪

```python
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser

# 🔥 自动追踪（配置环境变量后自动启用）
llm = ChatOpenAI(model="gpt-3.5-turbo")
prompt = ChatPromptTemplate.from_template("讲一个关于{topic}的笑话")
output_parser = StrOutputParser()

chain = prompt | llm | output_parser

# 运行（会自动追踪到LangSmith）
result = chain.invoke({"topic": "程序员"})
print(result)

# 在LangSmith平台查看追踪详情
```

### 3.2 自定义追踪

```python
from langsmith import traceable

# 🔥 使用装饰器追踪自定义函数
@traceable
def my_custom_function(input_text: str) -> str:
    """自定义函数"""
    # 处理逻辑
    result = input_text.upper()
    return result

# 调用会被追踪
result = my_custom_function("hello")
```

### 3.3 追踪Agent

```python
from langchain.agents import AgentExecutor, create_openai_functions_agent
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder
from langchain.tools import tool

@tool
def get_weather(location: str) -> str:
    """获取天气信息"""
    return f"{location}的天气是晴天"

tools = [get_weather]

llm = ChatOpenAI(model="gpt-3.5-turbo")
prompt = ChatPromptTemplate.from_messages([
    ("system", "你是一个有帮助的助手"),
    ("human", "{input}"),
    MessagesPlaceholder(variable_name="agent_scratchpad"),
])

agent = create_openai_functions_agent(llm, tools, prompt)
agent_executor = AgentExecutor(agent=agent, tools=tools, verbose=True)

# 🔥 运行Agent（自动追踪所有步骤）
result = agent_executor.invoke({"input": "北京的天气怎么样？"})

# 在LangSmith查看：
# - Agent的思考过程
# - 工具调用
# - 每一步的输入输出
```

### 3.4 查看追踪详情

在LangSmith平台可以看到：
- **执行时间**：每个步骤的耗时
- **输入输出**：每个组件的输入输出
- **Token使用**：LLM调用的token数
- **错误信息**：如果有错误，详细的堆栈信息
- **元数据**：自定义的元数据

---

## 4. 数据集管理

### 4.1 创建数据集

```python
from langsmith import Client

client = Client()

# 🔥 创建数据集
dataset_name = "qa-dataset"
dataset = client.create_dataset(
    dataset_name=dataset_name,
    description="问答测试数据集"
)

# 添加示例
examples = [
    {
        "inputs": {"question": "什么是Python？"},
        "outputs": {"answer": "Python是一种编程语言"}
    },
    {
        "inputs": {"question": "什么是LangChain？"},
        "outputs": {"answer": "LangChain是一个LLM应用开发框架"}
    },
]

for example in examples:
    client.create_example(
        inputs=example["inputs"],
        outputs=example["outputs"],
        dataset_id=dataset.id
    )
```

### 4.2 从CSV导入数据集

```python
import pandas as pd

# 读取CSV
df = pd.read_csv("qa_data.csv")

# 创建数据集
dataset = client.create_dataset(
    dataset_name="qa-from-csv",
    description="从CSV导入的数据集"
)

# 批量添加
for _, row in df.iterrows():
    client.create_example(
        inputs={"question": row["question"]},
        outputs={"answer": row["answer"]},
        dataset_id=dataset.id
    )
```

### 4.3 管理数据集

```python
# 列出所有数据集
datasets = client.list_datasets()
for ds in datasets:
    print(f"{ds.name}: {ds.description}")

# 获取数据集
dataset = client.read_dataset(dataset_name="qa-dataset")

# 获取示例
examples = client.list_examples(dataset_id=dataset.id)
for example in examples:
    print(example.inputs, example.outputs)

# 删除数据集
client.delete_dataset(dataset_id=dataset.id)
```

---

## 5. 评估和测试

### 5.1 运行评估

```python
from langsmith.evaluation import evaluate

# 定义要评估的函数
def qa_chain(inputs: dict) -> dict:
    """问答Chain"""
    llm = ChatOpenAI(model="gpt-3.5-turbo")
    prompt = ChatPromptTemplate.from_template("回答问题：{question}")
    chain = prompt | llm | StrOutputParser()
    
    answer = chain.invoke({"question": inputs["question"]})
    return {"answer": answer}

# 🔥 运行评估
results = evaluate(
    qa_chain,
    data="qa-dataset",  # 数据集名称
    evaluators=[],  # 评估器（稍后添加）
    experiment_prefix="qa-experiment",
    metadata={
        "model": "gpt-3.5-turbo",
        "version": "1.0"
    }
)

# 查看结果
print(results)
```

### 5.2 自定义评估器

```python
from langsmith.evaluation import EvaluationResult

# 🔥 自定义评估器
def accuracy_evaluator(run, example) -> EvaluationResult:
    """准确性评估器"""
    predicted = run.outputs["answer"]
    expected = example.outputs["answer"]
    
    # 简单的字符串匹配
    score = 1.0 if predicted.lower() == expected.lower() else 0.0
    
    return EvaluationResult(
        key="accuracy",
        score=score,
        comment=f"预测: {predicted}, 期望: {expected}"
    )

# 使用自定义评估器
results = evaluate(
    qa_chain,
    data="qa-dataset",
    evaluators=[accuracy_evaluator],
    experiment_prefix="qa-with-accuracy"
)
```

### 5.3 使用内置评估器

```python
from langsmith.evaluation import LangChainStringEvaluator

# 🔥 使用LangChain的评估器
qa_evaluator = LangChainStringEvaluator(
    "qa",
    config={
        "llm": ChatOpenAI(model="gpt-4"),
        "criteria": "correctness"
    }
)

results = evaluate(
    qa_chain,
    data="qa-dataset",
    evaluators=[qa_evaluator],
    experiment_prefix="qa-with-llm-eval"
)
```

### 5.4 比较实验

```python
# 运行多个实验
experiment_1 = evaluate(
    qa_chain_v1,
    data="qa-dataset",
    experiment_prefix="v1"
)

experiment_2 = evaluate(
    qa_chain_v2,
    data="qa-dataset",
    experiment_prefix="v2"
)

# 在LangSmith平台比较结果
# Projects -> Experiments -> Compare
```

---

## 6. 监控和分析

### 6.1 实时监控

```python
# 🔥 生产环境监控
os.environ["LANGCHAIN_TRACING_V2"] = "true"
os.environ["LANGCHAIN_PROJECT"] = "production"

# 所有调用都会被监控
chain = prompt | llm | output_parser
result = chain.invoke({"topic": "AI"})

# 在LangSmith查看：
# - 请求量
# - 响应时间
# - 错误率
# - Token使用量
```

### 6.2 添加元数据

```python
from langsmith import traceable

@traceable(
    run_type="chain",
    name="custom-qa-chain",
    metadata={
        "version": "1.0",
        "environment": "production"
    },
    tags=["qa", "production"]
)
def qa_with_metadata(question: str) -> str:
    """带元数据的问答"""
    llm = ChatOpenAI(model="gpt-3.5-turbo")
    return llm.invoke(question).content

# 调用
result = qa_with_metadata("什么是AI？")
```

### 6.3 用户反馈

```python
from langsmith import Client

client = Client()

# 🔥 收集用户反馈
def collect_feedback(run_id: str, score: float, comment: str):
    """收集用户反馈"""
    client.create_feedback(
        run_id=run_id,
        key="user-score",
        score=score,
        comment=comment
    )

# 在应用中使用
# 1. 获取run_id（从追踪中）
# 2. 用户评分后调用
collect_feedback(
    run_id="run-id-from-trace",
    score=5.0,
    comment="回答很好"
)
```

### 6.4 分析仪表板

在LangSmith平台查看：
- **性能指标**：平均响应时间、P95、P99
- **成本分析**：Token使用和成本
- **错误分析**：错误类型和频率
- **用户反馈**：评分分布和趋势

---

## 7. 团队协作

### 7.1 组织管理

```python
# 在LangSmith平台：
# 1. 创建组织
# 2. 邀请团队成员
# 3. 分配角色和权限
```

### 7.2 项目共享

```python
# 共享项目
# Settings -> Sharing -> Add members

# 设置权限
# - Viewer: 只读
# - Editor: 可编辑
# - Admin: 管理员
```

### 7.3 协作工作流

```python
# 开发流程
# 1. 开发环境：LANGCHAIN_PROJECT="dev"
# 2. 测试环境：LANGCHAIN_PROJECT="test"
# 3. 生产环境：LANGCHAIN_PROJECT="prod"

# 开发
os.environ["LANGCHAIN_PROJECT"] = "dev"
# ... 开发和测试

# 测试通过后部署到生产
os.environ["LANGCHAIN_PROJECT"] = "prod"
```

---

## 8. 最佳实践

### 8.1 开发阶段

```python
# ✅ 启用详细追踪
os.environ["LANGCHAIN_TRACING_V2"] = "true"
os.environ["LANGCHAIN_PROJECT"] = "development"

# ✅ 使用有意义的项目名称
# ✅ 添加元数据和标签
# ✅ 及时查看追踪，快速定位问题
```

### 8.2 测试阶段

```python
# ✅ 创建全面的测试数据集
# ✅ 使用多个评估器
# ✅ 运行回归测试
# ✅ 比较不同版本的性能
```

### 8.3 生产阶段

```python
# ✅ 监控关键指标
# ✅ 设置告警
# ✅ 收集用户反馈
# ✅ 定期分析和优化
```

### 8.4 成本优化

```python
# ✅ 监控Token使用
# ✅ 优化Prompt长度
# ✅ 使用缓存
# ✅ 选择合适的模型
```

---

## 📝 学习检查清单

- [ ] 能够配置和使用LangSmith
- [ ] 掌握追踪和调试技巧
- [ ] 能够管理测试数据集
- [ ] 掌握评估和测试方法
- [ ] 了解生产环境监控
- [ ] 能够收集和分析用户反馈
- [ ] 了解团队协作功能

---

## 🔗 相关资源

- [LangSmith官方文档](https://docs.smith.langchain.com/)
- [LangSmith平台](https://smith.langchain.com/)
- [LangSmith Python SDK](https://github.com/langchain-ai/langsmith-sdk)

---

@author erik.zhou
