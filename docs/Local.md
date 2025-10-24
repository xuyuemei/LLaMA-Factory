# ⚙️ 本地调用微调后的模型

微调完成后，我们可以在本地部署模型并通过 API 的方式调用。目前主流的两种本地部署方案是 **Ollama** 和 **VLLM**。

---

## 🚀 部署方案概览

| 工具 | 定位 | 特点 | 适用场景 | 推荐场景 |
|------|------|------|-----------|----------|
| **Ollama** | 轻量级本地部署工具 | 开箱即用、自动量化、显存占用低、支持流式输出与 JSON 响应 | 个人开发、隐私数据处理、小团队内部工具 | 快速部署、硬件门槛低、交互式测试 |
| **VLLM** | 高性能推理框架 | 基于 PyTorch、PagedAttention、高吞吐、支持多 GPU 与长文本推理 | 企业级推理、电商搜索、智能客服、高并发任务 | 高性能推理、多 GPU 扩展 |

---

### 🧩 Ollama 简介

Ollama 是一款轻量级大模型本地部署工具，以 **“开箱即用”** 为核心理念。  

- ✅ 支持全平台（Windows / macOS / Linux）  
- 🧠 内置 1700+ 主流模型（Llama、Qwen、Mistral 等）  
- ⚙️ 自动下载量化版本（如 int4 量化，显存占用降低约 50%）  
- 💬 提供类 ChatGPT 交互界面，支持流式输出与 JSON 响应  
- 🧮 支持 CPU 推理（≥16GB 内存）与 GPU 加速  

最新版本已支持视觉模型，可处理多模态任务。

---

### ⚡ VLLM 简介

VLLM 是专为高性能推理设计的企业级框架，基于 **PyTorch** 构建。  
其核心技术包括：

- 🧠 **PagedAttention**：将 KV Cache 分块存储，显存利用率提升约 30%。  
- 🚀 **动态批处理**：高吞吐率（Llama-8B 在 H100 上可达 5000+ tokens/s）。  
- 🧩 **多 GPU 张量并行**：支持 8×H100 部署 70B 模型，延迟 <500ms。  
- 📊 **监控能力**：原生支持 Prometheus 监控与自动故障恢复。  

非常适合电商搜索、智能客服、多模态场景及科研机构大规模部署。

---

## 🔧 使用 VLLM 启动 API 服务

在 **LLaMA Factory** 中，我们可以通过以下命令使用 `vLLM` 启动一个可调用的 API 服务：

```bash
API_PORT=6006 API_MODEL_NAME=security llamafactory-cli api \
    --model_name_or_path /root/autodl-tmp/save/Qwen2.5-7B-Instruct-Security \
    --template qwen \
    --infer_backend vllm \
    --vllm_enforce_eager
````

### 🧱 环境变量说明

| 变量                        | 说明                          |
| ------------------------- | --------------------------- |
| `API_PORT=6006`           | API 服务监听端口号                 |
| `API_MODEL_NAME=security` | 模型的自定义名称，默认 `gpt-3.5-turbo` |

### ⚙️ 命令参数说明

| 参数                     | 含义                    |
| ---------------------- | --------------------- |
| `--model_name_or_path` | 指定模型路径                |
| `--template qwen`      | 对话模板，与 Qwen 模型格式适配    |
| `--infer_backend vllm` | 启用 vLLM 推理引擎          |
| `--vllm_enforce_eager` | 强制使用动态图模式（便于调试与确定性执行） |

---

## 🧠 使用 Ollama 运行模型

Ollama 基于开源推理引擎 **llama.cpp**，支持运行 **GGUF 格式** 模型。

### 🪶 什么是 GGUF？

GGUF（**GGML Universal File**）是一种为大型语言模型设计的高效文件格式，优化了存储方式。

* **BF16 精度**：每参数占 16 位（2 字节）
* **4-bit 量化**：每参数占 4 位（0.5 字节）→ 显存占用降低 75%

因此，运行 7B 模型仅需约 **4GB 显存**。
但量化会牺牲部分精度，可能导致模型表现略“迟钝”。

---

### 🧩 模型格式转换

LLaMA Factory 默认不会输出 GGUF 格式，需要使用 `llama.cpp` 转换工具。

```bash
git clone https://github.com/ggerganov/llama.cpp.git
cd llama.cpp/gguf-py
pip install --editable .
```

然后执行转换命令：

```bash
python convert_hf_to_gguf.py /root/autodl-tmp/save/Qwen2.5-7B-Instruct-Security --outtype q8_0
```

转换成功后，会生成 GGUF 文件：

```bash
/root/autodl-tmp/save/Qwen2.5-7B-Instruct-Security/Qwen2.5-7B-Instruct-Security-F16.gguf
```

---

### 🧱 本地安装与运行 Ollama

安装 Ollama：

```bash
curl -fsSL https://ollama.com/install.sh | sh
```

启动服务：

```bash
OLLAMA_MODELS=/root/autodl-tmp/ollama ollama serve
```

注册模型（使用 Modelfile 文件而非 GGUF 文件）：

```bash
OLLAMA_MODELS=/root/autodl-tmp/ollama ollama create security -f /root/autodl-tmp/save/Qwen2.5-7B-Instruct-Security/Modelfile
```

运行模型：

```bash
ollama run security
```

---

## 🧾 模型效果对比验证

为了验证微调后的效果，可以使用 **Easy Dataset Playground** 进行测试。
我们采用 **VLLM 部署** 的服务（不损失精度、速度快）。

配置如下：

| 参数          | 说明         |
| ----------- | ---------- |
| 接口地址        | VLLM 服务地址  |
| 模型名称        | `security` |
| 提供商、API Key | 可自定义填写     |

---

### 🔍 对比模型

| 模型                    | 描述     |
| --------------------- | ------ |
| `Qwen2.5-7B-Instruct` | 基础模型   |
| `Security`            | 微调后的模型 |

---

### 📊 对比结论

相比基础模型，微调后的模型表现如下：

| 维度                   | 微调后提升                   |
| -------------------- | ----------------------- |
| **特定数据集问题**          | 能正确推理并给出更丰富回答，学习到了数据集知识 |
| **Web 安全领域问题** | 拥有较好推理能力与泛化表现           |
| **知识体系结构化输出**        | 能系统整合 Web 安全知识并形成逻辑性回答  |
| **跨领域迁移**            | 保持原有跨领域能力，能对非安全问题进行合理推理 |

---

## ✅ 总结

| 模块            | 内容                                 |
| ------------- | ---------------------------------- |
| **VLLM 部署**   | 高性能 API 调用、多 GPU 支持、OpenAI 接口兼容    |
| **Ollama 部署** | 快速本地运行、低门槛、支持 GGUF 格式              |
| **效果验证**      | 可在 Easy Dataset Playground 中对比输出表现 |

---
