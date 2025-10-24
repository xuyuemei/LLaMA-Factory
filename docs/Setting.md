# ⚙️ 微调通用设置（Fine-tuning Settings）

> 本文档介绍了 LLM 微调的通用配置，包括模型选择、微调方法、量化技术、对话模板与加速策略等内容。

---

## 🧭 一、选择模型（Model Selection）

| 分类 | 标识 | 含义 | 示例 |
| :-- | :-- | :-- | :-- |
| **功能与任务类型** | Base | 基础模型（原始能力） | Qwen3-14B-Base |
|  | Chat | 对话优化模型 | DeepSeek-LLM-7B-Chat |
|  | Instruct | 指令微调模型 | Qwen3-0.6B-Instruct |
|  | Distill | 知识蒸馏模型 | DeepSeek-R1-1.5B-Distill |
|  | Math | 数学推理模型 | DeepSeek-Math-7B-Instruct |
|  | Coder | 编程任务模型 | DeepSeek-Coder-V2-16B |
| **多模态** | VL | 视觉-语言模型 | Kimi-VL-A3B-Instruct |
|  | Video | 视频多模态模型 | LLaVA-NeXT-Video-7B-Chat |
|  | Audio | 音频输入模型 | Qwen2-Audio-7B |
| **技术特性** | Int8 / Int4 | 权重量化模型 | Qwen2-VL-2B-Instruct-GPTQ-Int8 |
|  | AWQ / GPTQ | 特定量化技术 | Qwen2.5-VL-72B-Instruct-AWQ |
|  | MoE | 混合专家模型 | DeepSeek-MoE-16B-Chat |
|  | RL | 强化学习优化模型 | MiMo-7B-Instruct-RL |
| **版本** | v0.1 / v0.2 | 模型版本号 | Mistral-7B-v0.1 |
| **变体** | Pure | 纯净版模型 | Index-1.9B-Base |
|  | Character | 角色对话模型 | Index-1.9B-Character-Chat |
|  | Long-Chat | 长上下文模型 | Orion-14B-Long-Chat |
| **应用领域** | RAG | 检索增强生成模型 | Orion-14B-RAG-Chat |
|  | Chinese | 中文优化模型 | Llama-3-70B-Chinese-Chat |
|  | MT | 翻译模型 | BLOOMZ-7B1-mt |

---

## 🧠 二、微调方法（Fine-tuning Methods）

### 🧩 1. Full（全参数微调）

- **概念**：训练全部参数，最大化性能。  
- **类比**：拆掉房子重建。  
- **优点**：性能最强、完全适应性高。  
- **缺点**：显存与计算量需求极高。  
- **适用场景**：有充足算力、数据量大、任务差异大。

---

### 🧱 2. Freeze（参数冻结微调）

- **概念**：冻结部分参数，仅训练部分层。  
- **类比**：给房子局部装修。  
- **优点**：计算效率高、防止过拟合、节省内存。  
- **缺点**：性能略低、需专业调参。  
- **适用场景**：资源有限、小数据集、任务相似。

---

### 🔗 3. LoRA（低秩适配微调）

- **概念**：通过注入小型可训练矩阵实现高效微调。  
- **类比**：给房子加智能模块。  
- **优点**：
  - 训练参数减少 99%+
  - 显存占用低、训练速度快
  - 适配多任务与轻量化部署
- **缺点**：
  - 性能上限略低
  - 超参数调节复杂
- **适用场景**：
  - GPU 资源有限
  - 快速适配新任务
  - 多模态或迭代优化场景

> 💡 **LoRA 微调是当前主流方法**  
> 在保持预训练权重不变的情况下，LoRA 通过低秩矩阵学习特定任务特征，实现快速收敛、低显存占用与良好泛化性能。

---

## 🧮 三、模型量化（Model Quantization）

模型量化与蒸馏同属于 **模型压缩（Model Compression）** 技术。  
通过降低参数精度，可在**轻微精度损失**下显著减少显存与计算需求。

---

### 🌐 1. 什么是模型量化？

量化是指**用更少的比特表示模型权重与激活值**。  
简而言之，就是“用更小的文件保存模型”。

> 🎧 类比：
> - FLAC → FP32 精度模型（高保真）  
> - MP3 → INT8 / INT4 量化模型（更轻但有压缩）

---

### 🧩 2. 模型参数精度格式

| 精度类型 | 示例 | 含义 | 特点 |
|-----------|--------|------|------|
| **FP（Floating Point）** | FP16 / FP32 | 浮点数格式 | 高精度，占用显存多 |
| **BF（Brain Floating Point）** | BF16 | 为深度学习优化的浮点格式 | 精度与效率平衡 |
| **INT（Integer）** | INT8 / INT4 | 整数量化 | 极低显存占用，精度略降 |

💡 **内存占用差异：**
- FP32：4 字节 / 参数  
- FP16：2 字节 / 参数  
- INT8：1 字节 / 参数  

---

### ⚙️ 3. QLoRA 简介

> **QLoRA（Quantized LoRA）** = **LoRA + 4-bit 动态量化**

通过：
- 4-bit 量化基础模型  
- 双量化与训练时反量化机制  

QLoRA 实现：
- 🚀 显存占用极低  
- ✅ 保留梯度精度  
- 💨 训练效率极高  

> 📌 在 LLaMA Factory 中，当选择 “4-bit 量化” 时，默认启用的就是 **QLoRA 微调方法**。

---

## 💬 四、对话模板（Conversation Template）

对话模板用于定义模型的输入格式，使训练与推理保持一致。

---

### 🧩 1. 主要作用

1. **训练-推理一致性**：确保输入结构一致，防止性能下降。  
2. **上下文管理**：区分 `User` / `Assistant` / `System`。  
3. **支持高级功能**：系统提示、思维链、工具调用、多模态输入等。

---

### 🧱 2. Qwen 模板示例

```python
register_template(
    name="qwen",
    format_user=StringFormatter(slots=["<|im_start|>user\n{{content}}<|im_end|>\n<|im_start|>assistant\n"]),
    format_assistant=StringFormatter(slots=["{{content}}<|im_end|>\n"]),
    format_system=StringFormatter(slots=["<|im_start|>system\n{{content}}<|im_end|>\n"]),
    format_function=FunctionFormatter(slots=["{{content}}<|im_end|>\n"], tool_format="qwen"),
    format_observation=StringFormatter(
        slots=["<|im_start|>user\n<tool_response>\n{{content}}\n</tool_response><|im_end|>\n<|im_start|>assistant\n"]
    ),
    format_tools=ToolFormatter(tool_format="qwen"),
    default_system="You are Qwen, created by Alibaba Cloud. You are a helpful assistant.",
    stop_words=["<|im_end|>"],
    replace_eos=True,
)
````

---

### 📜 3. 格式化示例

**用户输入：**

> 帮我写一首关于春天的诗

**格式化后：**

```
<|im_start|>system
You are Qwen, created by Alibaba Cloud. You are a helpful assistant.
<|im_end|>

<|im_start|>user
帮我写一首关于春天的诗
<|im_end|>

<|im_start|>assistant
```

---

## ⚡ 五、加速方式（Acceleration Methods）

LLaMA Factory 支持多种加速机制：

| 加速方法                 | 说明                      | 配置参数                        |
| -------------------- | ----------------------- | --------------------------- |
| **FlashAttention-2** | 提高注意力计算速度、减少显存占用        | `flash_attn: fa2`           |
| **Unsloth**          | 优化 QLoRA / LoRA 微调速度与显存 | `use_unsloth: True`         |
| **Liger Kernel**     | 提升吞吐量与显存效率              | `enable_liger_kernel: True` |

> ⚙️ **推荐配置**
>
> * 若硬件资源有限：`Unsloth + 4-bit 量化`
> * 若性能要求一般：默认 `FlashAttention-2`
