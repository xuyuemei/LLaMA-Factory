# 🦙 LLaMA Factory

> **参考资料**  
> - [（一）LLaMA Factory](https://mp.weixin.qq.com/s/aQCY8873d09zFIhMhrx7Pg)
> - [（二）数据集](https://mp.weixin.qq.com/s/N8LdX3eRuaIJ-yxkpxZJ5w)
> - [（三）参数](https://mp.weixin.qq.com/s/AbyWaTaPOp9sr5mz5SOVwg)
> - [（四）评估与本地](https://mp.weixin.qq.com/s/6sNGvqLktPk6AP7kPs9JyA)
> - [Bilibili 教程视频](https://www.bilibili.com/video/BV1oTEwzcEeZ?t=0.2)
---

## 一、介绍

### 🧩 1. 全场景模型微调能力

- **多模型支持**：高效适配超 100 个主流模型，涵盖 Qwen、DeepSeek、LLaMA、Gemma、LLaVA、Mistral、Mixtral-MoE、Yi、Baichuan、ChatGLM、Phi 等，满足 NLP、多模态等多领域需求。  
- **多样化算法与精度**：集成 LoRA、GaLore、DoRA 等微调技术，支持（增量）预训练、（多模态）指令监督微调、奖励模型训练、PPO/DPO/KTO/ORPO 等强化学习方法；提供 16 比特全参数微调、冻结微调、LoRA 微调，以及基于 AQLM/AWQ/GPTQ 等技术的 2/3/4/5/6/8 比特 QLoRA 低比特量化微调。  
- **数据集灵活配置**：支持自定义或社区数据集（HuggingFace、ModelScope），自动下载缓存模型与数据集，适配多卡训练。  
- **多种加速技巧**：支持 FlashAttention-2、Unsloth 等高效算子。

### ⚙️ 2. 极简操作与高效工具链

- **零代码 Web UI**：通过可视化界面完成模型配置、数据加载、参数调优、训练监控与日志查看。  
- **实时监控与评估**：集成 Wandb、MLflow、SwanLab、TensorBoard 等工具实现可视化训练与性能分析。

### 🧱 3. 工程化部署与资源利用

- **模型部署**：LoRA 适配器可一键合并为完整模型，便于本地化部署。  
- **推理引擎**：内置 Transformers 与基于 vLLM 的 OpenAI 风格 API。  
- **资源优化**：支持单机多卡与多硬件适配。

---

## 二、LLaMA Factory VS Unsloth

| 对比项 | **LLaMA Factory** | **Unsloth** |
| :-- | :-- | :-- |
| **主要特点** | 全场景微调平台，支持多模型与强化学习 | 高速低显存的微调工具 |
| **性能优化** | 支持多种加速技术与量化微调 | 微调速度提升 2-5 倍，显存占用下降 50%-80% |
| **硬件需求** | 适配多卡与企业级 GPU | 7GB 显存可训练 1.5B 参数模型 |
| **适用场景** | 企业级部署、通用任务、零代码用户 | 个人开发者、低资源场景、快速迭代 |
| **可视化支持** | WebUI、API Server、监控系统 | Colab Notebook 快速上手 |

> ✅ **结论**：  
> - **Unsloth**：适合资源有限、需要快速实验的场景。  
> - **LLaMA Factory**：适合通用大模型训练、企业部署与教学展示。

---

## 三、安装方式

推荐使用 **Conda 虚拟环境** 进行隔离安装。

```bash
# 创建虚拟环境
conda create -n llama_factory python=3.10
conda activate llama_factory

# 安装依赖
pip install -U pip
pip install -U llama-factory
````

---
  
## 四、WebUI 模块结构

LLaMA Factory 的 WebUI 主要可分为五大功能区域：

### 1. **通用设置** ⚙️
**功能**：配置训练的基础环境和基础参数
- **界面语言**：中英文切换
- **模型选择**：基础模型加载与配置
- **微调方法**：Full Fine-tuning / LoRA / QLoRA 等
- **量化配置**：4bit/8bit 量化、GPU 加速选项

> 📋 详细配置说明参见：[`Setting.md`](./docs/Setting.md)

### 2. **微调训练** 🏋️
**功能**：完整的训练流程配置

| 配置区块 | 核心参数 | 关联文档 |
|---------|----------|----------|
| **训练阶段** | 预训练 / 指令微调 / 强化学习 | |
| **数据集配置** | 数据路径、验证集比例、格式映射 | [`Dataset.md`](./docs/Dataset.md) |
| **训练参数** | 学习率、训练轮数、批量大小 | [`Parameters.md`](./docs/Parameters.md) |
| **高级配置** | LoRA 秩、RLHF 参数、优化器选择 | |
| **可视化** | SwanLab 监控、Loss 曲线配置 | |

### 3. **模型评估** 📊
**功能**：模型性能验证与测试
- **评估数据集**：支持多数据集并行评估
- **自动化测试**：批量推理与指标计算
- **结果分析**：准确率、Loss 趋势、性能对比
> 详细结果分析可查看  [`Loss.md`](./docs/Loss.md)

### 4. **在线推理** 💬
**功能**：实时模型对话测试 （Chat）
- **推理引擎**：HuggingFace / vLLM / 其他后端
- **对话界面**：多轮对话、角色扮演、流式输出
- **参数调节**：Temperature、Top-p、Max tokens

### 5. **模型导出** 📦
**功能**：训练成果导出与部署 （Export）
```yaml
导出配置：
  - 模型格式: HuggingFace / ONNX / GGUF
  - 适配器: LoRA 权重合并
  - 量化等级: Q4/Q8/FP16/FP32
  - 设备兼容: CPU/GPU 优化
  - 输出目录: 自定义保存路径
```

> 📦 模型导出详细操作可查看：[`Export.md`](./docs/Export.md)

---

### 6. **本地部署与快速推理** 🖥️

**功能**：模型在本地或服务器环境中的加载、测试与部署

* **环境依赖**：Transformers / vLLM / GGUF
* **命令行推理**：单轮与多轮对话支持
* **GPU 优化**：自动显存管理与批量生成

> 🚀 本地推理配置详见：[`Local.md`](./docs/Local.md)

---
