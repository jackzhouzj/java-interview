# vLLM - 完整教程

> @author erik.zhou

## 📚 技术概述

### 版本信息
- **vLLM版本**：0.2+
- **最新稳定版**：0.2.x
- **推荐版本**：最新版

### 学习难度
- **难度等级**：⭐⭐⭐⭐ (较难)
- **预计学习时间**：15-20小时
- **重要程度**：⭐⭐⭐⭐⭐ (必学)

### 前置知识
- Python基础
- PyTorch基础
- Transformer原理
- GPU编程基础
- LLM基础知识

## 🎯 学习目标

完成本教程后，你将能够：

- [ ] 理解vLLM的核心原理
- [ ] 掌握PagedAttention机制
- [ ] 能够部署和配置vLLM服务
- [ ] 理解连续批处理技术
- [ ] 掌握性能优化技巧
- [ ] 能够监控和调试vLLM
- [ ] 了解生产环境最佳实践

## 📖 目录

1. [vLLM简介](#1-vllm简介)
2. [环境搭建](#2-环境搭建)
3. [核心原理](#3-核心原理)
4. [基础使用](#4-基础使用)
5. [API服务](#5-api服务)
6. [性能优化](#6-性能优化)
7. [高级特性](#7-高级特性)
8. [生产部署](#8-生产部署)
9. [监控和调试](#9-监控和调试)
10. [最佳实践](#10-最佳实践)

---

## 1. vLLM简介

### 1.1 什么是vLLM

vLLM是一个快速且易用的LLM推理和服务库，专注于高吞吐量和内存效率。

**核心特性**：
- 🔥 **PagedAttention**：高效的注意力机制
- 🔥 **连续批处理**：动态批处理优化
- 🔥 **高吞吐量**：比传统方法快24倍
- 🔥 **内存优化**：减少内存浪费
- 🔥 **易于使用**：兼容HuggingFace模型

### 1.2 vLLM vs 其他推理引擎

| 特性 | vLLM | TGI | llama.cpp |
|------|------|-----|-----------|
| 吞吐量 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ |
| 内存效率 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| GPU支持 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |
| CPU支持 | ⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| 易用性 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ |
| 模型支持 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |

### 1.3 应用场景

- **高并发服务**：处理大量并发请求
- **批量推理**：批量处理大量文本
- **实时应用**：低延迟要求的应用
- **资源受限**：GPU内存有限的场景
- **生产环境**：企业级LLM服务

---

## 2. 环境搭建

### 2.1 系统要求

```bash
# GPU要求
- NVIDIA GPU (计算能力 >= 7.0)
- CUDA 11.8 或更高
- 至少 16GB GPU内存（推荐 24GB+）

# 系统要求
- Linux (推荐 Ubuntu 20.04+)
- Python 3.8+
```

### 2.2 安装vLLM

```bash
# 🔥 使用pip安装
pip install vllm

# 从源码安装（最新版本）
git clone https://github.com/vllm-project/vllm.git
cd vllm
pip install -e .

# 验证安装
python -c "import vllm; print(vllm.__version__)"
```

### 2.3 安装依赖

```bash
# 安装CUDA工具包
# 访问 https://developer.nvidia.com/cuda-downloads

# 安装PyTorch
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu118

# 验证GPU
python -c "import torch; print(torch.cuda.is_available())"
```

---

## 3. 核心原理

### 3.1 PagedAttention

PagedAttention是vLLM的核心创新，借鉴了操作系统的虚拟内存和分页技术。

**传统Attention的问题**：
```python
# 传统方法：预分配固定大小的KV缓存
# 问题：
# 1. 内存浪费（实际长度 < 最大长度）
# 2. 内存碎片
# 3. 无法动态调整

max_length = 2048
kv_cache = torch.zeros(batch_size, max_length, hidden_size)
# 大量内存被浪费
```

**PagedAttention的解决方案**：
```python
# PagedAttention：按需分配内存块
# 优势：
# 1. 减少内存浪费（接近零浪费）
# 2. 支持更大的批处理
# 3. 动态内存管理

# 内存被分成固定大小的块（pages）
# 只在需要时分配新块
# 类似操作系统的虚拟内存
```

### 3.2 连续批处理

```python
# 🔥 连续批处理（Continuous Batching）
# 传统批处理：等待所有请求完成
# 问题：慢请求拖累快请求

# vLLM连续批处理：
# 1. 请求完成后立即移除
# 2. 新请求立即加入批次
# 3. 最大化GPU利用率

# 示例：
# 批次1: [req1, req2, req3, req4]
# req2完成 -> 立即移除
# 批次2: [req1, req3, req4, req5]  # req5加入
```

### 3.3 内存优化

```python
# 内存优化技术
# 1. KV缓存共享（用于beam search）
# 2. 内存池管理
# 3. 预取和缓存
# 4. 零拷贝技术
```

---

## 4. 基础使用

### 4.1 离线推理

```python
from vllm import LLM, SamplingParams

# 🔥 创建LLM实例
llm = LLM(
    model="meta-llama/Llama-2-7b-chat-hf",
    tensor_parallel_size=1,  # GPU数量
    dtype="float16"
)

# 设置采样参数
sampling_params = SamplingParams(
    temperature=0.8,
    top_p=0.95,
    max_tokens=256
)

# 单个prompt
prompts = ["Hello, my name is"]
outputs = llm.generate(prompts, sampling_params)

for output in outputs:
    prompt = output.prompt
    generated_text = output.outputs[0].text
    print(f"Prompt: {prompt}")
    print(f"Generated: {generated_text}")
```

### 4.2 批量推理

```python
# 🔥 批量处理多个prompt
prompts = [
    "The future of AI is",
    "Machine learning is",
    "Deep learning enables",
    "Natural language processing helps"
]

# 批量生成
outputs = llm.generate(prompts, sampling_params)

for output in outputs:
    print(f"Prompt: {output.prompt}")
    print(f"Generated: {output.outputs[0].text}")
    print("-" * 50)
```

### 4.3 流式生成

```python
# 流式输出
from vllm import LLM, SamplingParams

llm = LLM(model="meta-llama/Llama-2-7b-chat-hf")
sampling_params = SamplingParams(
    temperature=0.8,
    max_tokens=256
)

# 使用stream参数
prompts = ["Tell me a story about"]

for output in llm.generate(prompts, sampling_params, use_tqdm=True):
    print(output.outputs[0].text, end="", flush=True)
```

---

## 5. API服务

### 5.1 启动OpenAI兼容服务

```bash
# 🔥 启动vLLM服务器
python -m vllm.entrypoints.openai.api_server \
    --model meta-llama/Llama-2-7b-chat-hf \
    --host 0.0.0.0 \
    --port 8000 \
    --tensor-parallel-size 1

# 高级配置
python -m vllm.entrypoints.openai.api_server \
    --model meta-llama/Llama-2-7b-chat-hf \
    --host 0.0.0.0 \
    --port 8000 \
    --tensor-parallel-size 2 \
    --dtype float16 \
    --max-model-len 4096 \
    --gpu-memory-utilization 0.9
```

### 5.2 使用OpenAI客户端

```python
from openai import OpenAI

# 🔥 连接vLLM服务
client = OpenAI(
    base_url="http://localhost:8000/v1",
    api_key="EMPTY"  # vLLM不需要API密钥
)

# Chat Completions
response = client.chat.completions.create(
    model="meta-llama/Llama-2-7b-chat-hf",
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "What is machine learning?"}
    ],
    temperature=0.7,
    max_tokens=256
)

print(response.choices[0].message.content)
```

### 5.3 流式响应

```python
# 流式Chat Completions
response = client.chat.completions.create(
    model="meta-llama/Llama-2-7b-chat-hf",
    messages=[
        {"role": "user", "content": "Tell me a story"}
    ],
    stream=True
)

for chunk in response:
    if chunk.choices[0].delta.content:
        print(chunk.choices[0].delta.content, end="", flush=True)
```

---

## 6. 性能优化

### 6.1 GPU配置

```python
from vllm import LLM

# 🔥 优化GPU使用
llm = LLM(
    model="meta-llama/Llama-2-7b-chat-hf",
    tensor_parallel_size=2,  # 使用2个GPU
    gpu_memory_utilization=0.9,  # 使用90%的GPU内存
    dtype="float16",  # 使用半精度
    max_model_len=4096,  # 最大序列长度
    swap_space=4,  # CPU交换空间（GB）
)
```

### 6.2 批处理优化

```python
# 优化批处理大小
llm = LLM(
    model="meta-llama/Llama-2-7b-chat-hf",
    max_num_batched_tokens=8192,  # 批次最大token数
    max_num_seqs=256,  # 批次最大序列数
)
```

### 6.3 采样参数优化

```python
from vllm import SamplingParams

# 🔥 优化采样参数
sampling_params = SamplingParams(
    temperature=0.8,
    top_p=0.95,
    top_k=50,
    max_tokens=256,
    presence_penalty=0.0,
    frequency_penalty=0.0,
    repetition_penalty=1.0,
    length_penalty=1.0,
    stop=["</s>", "\n\n"],  # 停止标记
    n=1,  # 生成数量
    best_of=1,  # 从n个中选最好的
    use_beam_search=False  # 是否使用beam search
)
```

---

## 7. 高级特性

### 7.1 多GPU推理

```python
# 🔥 张量并行（Tensor Parallelism）
llm = LLM(
    model="meta-llama/Llama-2-70b-chat-hf",
    tensor_parallel_size=4,  # 使用4个GPU
    dtype="float16"
)

# 流水线并行（Pipeline Parallelism）
llm = LLM(
    model="meta-llama/Llama-2-70b-chat-hf",
    pipeline_parallel_size=2,  # 流水线并行
    tensor_parallel_size=2,  # 张量并行
)
```

### 7.2 量化支持

```bash
# 使用量化模型
python -m vllm.entrypoints.openai.api_server \
    --model TheBloke/Llama-2-7B-Chat-GPTQ \
    --quantization gptq \
    --dtype float16
```

### 7.3 自定义模型

```python
from vllm import LLM

# 加载本地模型
llm = LLM(
    model="/path/to/local/model",
    tokenizer="/path/to/tokenizer",
    trust_remote_code=True
)
```

---

## 8. 生产部署

### 8.1 Docker部署

```dockerfile
# Dockerfile
FROM nvidia/cuda:11.8.0-devel-ubuntu22.04

# 安装Python和依赖
RUN apt-get update && apt-get install -y python3-pip
RUN pip3 install vllm

# 下载模型
RUN python3 -c "from huggingface_hub import snapshot_download; \
    snapshot_download('meta-llama/Llama-2-7b-chat-hf')"

# 启动服务
CMD ["python3", "-m", "vllm.entrypoints.openai.api_server", \
     "--model", "meta-llama/Llama-2-7b-chat-hf", \
     "--host", "0.0.0.0", \
     "--port", "8000"]
```

```bash
# 构建和运行
docker build -t vllm-server .
docker run --gpus all -p 8000:8000 vllm-server
```

### 8.2 Kubernetes部署

```yaml
# vllm-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: vllm-server
spec:
  replicas: 2
  selector:
    matchLabels:
      app: vllm
  template:
    metadata:
      labels:
        app: vllm
    spec:
      containers:
      - name: vllm
        image: vllm-server:latest
        ports:
        - containerPort: 8000
        resources:
          limits:
            nvidia.com/gpu: 1
---
apiVersion: v1
kind: Service
metadata:
  name: vllm-service
spec:
  selector:
    app: vllm
  ports:
  - port: 8000
    targetPort: 8000
  type: LoadBalancer
```

### 8.3 负载均衡

```python
# 使用Nginx进行负载均衡
# nginx.conf
upstream vllm_backend {
    least_conn;
    server vllm-1:8000;
    server vllm-2:8000;
    server vllm-3:8000;
}

server {
    listen 80;
    location / {
        proxy_pass http://vllm_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

---

## 9. 监控和调试

### 9.1 性能监控

```python
# 获取统计信息
from vllm import LLM

llm = LLM(model="meta-llama/Llama-2-7b-chat-hf")

# 生成并获取统计
outputs = llm.generate(prompts, sampling_params)

for output in outputs:
    # 查看生成统计
    print(f"Tokens generated: {len(output.outputs[0].token_ids)}")
    print(f"Finish reason: {output.outputs[0].finish_reason}")
```

### 9.2 日志配置

```python
import logging

# 配置日志
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
)

# vLLM日志
logger = logging.getLogger("vllm")
logger.setLevel(logging.DEBUG)
```

### 9.3 性能分析

```bash
# 使用nvidia-smi监控GPU
watch -n 1 nvidia-smi

# 使用nvtop
nvtop

# 使用Prometheus监控
# 启动时添加metrics端点
python -m vllm.entrypoints.openai.api_server \
    --model meta-llama/Llama-2-7b-chat-hf \
    --enable-metrics
```

---

## 10. 最佳实践

### 10.1 内存管理

```python
# ✅ 合理设置GPU内存使用率
llm = LLM(
    model="meta-llama/Llama-2-7b-chat-hf",
    gpu_memory_utilization=0.9  # 留10%给系统
)

# ✅ 设置合适的最大长度
llm = LLM(
    model="meta-llama/Llama-2-7b-chat-hf",
    max_model_len=2048  # 根据实际需求设置
)
```

### 10.2 批处理策略

```python
# ✅ 批量处理请求
prompts = [...]  # 收集多个请求
outputs = llm.generate(prompts, sampling_params)

# ❌ 避免单个请求循环
for prompt in prompts:
    output = llm.generate([prompt], sampling_params)  # 低效
```

### 10.3 错误处理

```python
from vllm import LLM
from vllm.utils import is_hip

try:
    llm = LLM(model="meta-llama/Llama-2-7b-chat-hf")
    outputs = llm.generate(prompts, sampling_params)
except RuntimeError as e:
    if "out of memory" in str(e):
        print("GPU内存不足，尝试减少batch size或max_model_len")
    else:
        raise
```

---

## 📝 学习检查清单

- [ ] 理解vLLM的核心原理
- [ ] 掌握PagedAttention机制
- [ ] 能够部署vLLM服务
- [ ] 掌握性能优化技巧
- [ ] 能够进行生产部署
- [ ] 了解监控和调试方法
- [ ] 掌握最佳实践

---

## 🔗 相关资源

- [vLLM官方文档](https://docs.vllm.ai/)
- [vLLM GitHub](https://github.com/vllm-project/vllm)
- [vLLM论文](https://arxiv.org/abs/2309.06180)
- [性能基准测试](https://github.com/vllm-project/vllm/tree/main/benchmarks)

---

@author erik.zhou
