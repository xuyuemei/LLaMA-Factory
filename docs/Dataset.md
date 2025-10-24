# 📦 数据集指南（Dataset & Construction Guide）

> 本文档整合了 **LLaMA Factory** 的数据集配置说明与 **Easy Dataset（EDS）** 数据构造指南，涵盖数据加载、格式定义、增强方法与质量控制标准。

../images/data1.jpg
---

## 📑 目录

1. [数据集格式与加载（LLaMA Factory）](#一-数据集格式与加载-llama-factory)
2. [数据集构造思路（Easy Dataset）](#二-数据集构造思路-easy-dataset)
3. [数据集质量控制要点](#三-数据集质量控制要点)
4. [数据构造步骤（EDS 实践）](#四-数据构造步骤-eds-实践)
5. [数据集 Review 标准](#五-数据集-review-标准)
6. [数据增强（Data Augmentation）](#六-数据增强-data-augmentation)

---

## 一、数据集格式与加载（LLaMA Factory）

在 **LLaMA Factory** 中主要支持以下两种数据格式：

* 🟦 **Alpaca 格式**（单轮指令数据）
* 🟩 **ShareGPT 格式**（多轮对话数据）

### 📁 数据集配置位置

在 `Train` 选项下，LLaMA Factory 提供了两个与数据集相关的配置项：

| 配置项 | 说明 |
| :------ | :------ |
| **数据路径（data path）** | 默认指向项目文件夹下的 `data` 目录 |
| **数据集（dataset_info.json）** | 管理所有数据集的加载与映射配置 |

基本结构如下：

```json
{
  "数据集名称": {
    "配置项1": "值1",
    "配置项2": "值2"
  }
}
```

### 🌐 数据源定义方式

#### ① 在线数据集

来源平台：**Hugging Face Hub**、**ModelScope Hub**、**OpenMind Hub**

```json
"alpaca_en": {
  "hf_hub_url": "llamafactory/alpaca_en",
  "ms_hub_url": "llamafactory/alpaca_en",
  "om_hub_url": "HaM/alpaca_en"
}
```

#### ② 本地文件

```json
"alpaca_en_demo": {
  "file_name": "alpaca_en_demo.json"
}
```

#### ③ 自定义脚本生成数据集

```json
"belle_multiturn": {
  "script_url": "belle_multiturn",
  "formatting": "sharegpt"
}
```

### 🧩 数据格式配置

#### （1）基本格式类型

```json
"slimorca": {
  "hf_hub_url": "Open-Orca/SlimOrca",
  "formatting": "sharegpt"
}
```

#### （2）列映射配置

```json
"openorca": {
  "hf_hub_url": "Open-Orca/OpenOrca",
  "columns": {
    "prompt": "question",
    "response": "response",
    "system": "system_prompt"
  }
}
```

#### （3）角色标签配置（ShareGPT 格式）

```json
"mllm_demo": {
  "formatting": "sharegpt",
  "tags": {
    "role_tag": "role",
    "content_tag": "content",
    "user_tag": "user",
    "assistant_tag": "assistant"
  }
}
```

#### （4）多模态数据支持

```json
"mllm_demo": {
  "file_name": "mllm_demo.json",
  "formatting": "sharegpt",
  "columns": {
    "messages": "messages",
    "images": "images"
  }
}
```

### ⚙️ 特殊训练任务配置

#### ✅ 排序 / 对比学习数据（RLHF / DPO）

```json
"webgpt": {
  "hf_hub_url": "openai/webgpt_comparisons",
  "ranking": true,
  "columns": {
    "prompt": "question",
    "chosen": "answer_0",
    "rejected": "answer_1"
  }
}
```

#### 🧰 工具调用（Tool Calling）数据

```json
"glaive_toolcall_en_demo": {
  "file_name": "glaive_toolcall_en_demo.json",
  "formatting": "sharegpt",
  "columns": {
    "messages": "conversations",
    "tools": "tools"
  }
}
```

---

## 二、数据集构造思路（Easy Dataset）

数据集是大语言模型（LLM）微调的核心。其质量直接决定了模型在目标任务中的表现。

本项目采用系统化的数据构造策略，确保数据的 **相关性、均衡性与高质量**：

1. **领域知识提取**
   * 从《白帽子讲 Web 安全》等专业书籍中提取结构化问答样本

2. **前沿知识扩充**
   * 从最新论文中提取问题-答案对，涵盖 Web 安全研究新趋势：
     * 《The Hidden Risks of LLM-Generated Web Application Code》
     * 《WASP: Benchmarking Web Agent Security Against Prompt Injection Attacks》
     * 《A Human Study of Cognitive Biases in Web Application Security》

3. **教师模型蒸馏（Knowledge Distillation）**
   * 使用 **DeepSeek R1 0528 满血版模型** 生成高质量问答与推理链（CoT）数据

4. **多样性增强（GA：Genre-Audience 方法）**
   * 对样本从体裁与受众维度进行扩充，涵盖多种任务类型（解释、推理、漏洞分析等）

5. **专家审校（Expert Review）**
   * 邀请 Web 安全领域专家对最终数据集进行质量评估与修订，确保专业性与一致性

### 🧰 Easy Dataset（EDS）工具介绍

**Easy Dataset（EDS）** 是一款专为大语言模型数据集构造设计的工具，支持高效、结构化的数据生产与管理。

* 🏠 **GitHub**：[ConardLi/easy-dataset](https://github.com/ConardLi/easy-dataset)
* 📖 **官方文档**：[docs.easy-dataset.com](https://docs.easy-dataset.com/)（已被 **EMNLP** 收录）

#### 💡 功能特点

* 可视化数据构造：支持多模板编辑、实时生成与预览
* 完全兼容 OpenAI / LLaMA / Qwen 等 LLM API 格式
* 自动清洗与去重机制，减少噪声数据
* 导出为 json、jsonl、csv 等多种格式

---

## 三、数据集质量控制要点

下列问题是导致微调失败的常见原因，构造过程中需重点规避：

| 问题类别 | 说明 |
| :---------- | :----------------------------------------------- |
| **数据量太小** | 样本不足导致过拟合。领域任务建议 ≥ 1000 条（7B 模型起步），模型参数越大所需数据越多。 |
| **噪声数据多** | 包含错别字、重复、乱码或错误标注，需清洗与规范化。 |
| **样本偏差严重** | 训练数据与真实应用分布差异过大，如正负样本不平衡。 |
| **任务相关性不足** | 数据形式与目标任务不符，如问答任务混入叙述性新闻。 |
| **数据多样性不足** | 指令、场景或特征维度单一，缺乏覆盖不同输入形式的样本。 |

> ✅ **优先保证质量，而非数量。**
> 数据越多并不一定更好；低质量数据会放大模型误差。

---

## 四、数据构造步骤（EDS 实践）

### 1️⃣ 准备原始素材

* Web 安全书籍章节、论文、研究报告
* 专家标注问答、漏洞场景说明

### 2️⃣ 在 EDS 中创建任务

* 选择模板：Instruction → Response（Alpaca 格式）
* 或 Conversation → Messages（ShareGPT 格式）
* 导入素材文本，并配置指令生成策略

### 3️⃣ 语料清洗与格式统一

* 去除冗余段落、HTML符号、重复问答
* 对齐字段名（如 instruction、input、output）

### 4️⃣ 教师模型蒸馏增强

* 调用 DeepSeek R1 生成高质量回答
* 提取其 **推理链（Chain-of-Thought, CoT）**
* 标注字段：

```json
{
  "instruction": "解释XSS攻击的原理。",
  "output": "XSS是一种前端输入未过滤导致的跨站脚本漏洞...",
  "cot": "用户输入未进行转义 → 浏览器执行恶意JS → 数据泄露。"
}
```

### 5️⃣ 多样性扩充（GA增强）

* 使用 Genre（任务类型） + Audience（目标受众）策略
* 如面向开发者的技术问答、面向管理者的策略建议

### 6️⃣ 专家人工复审

* 检查样本合理性、一致性与事实准确性
* 对模糊或冲突样本进行修订或剔除

---

## 五、数据集 Review 标准（专家审查环节）

| 审查维度 | 要求 |
| :---------- | :----------------------------------- |
| **事实一致性** | 答案与文献事实一致，不得出现错误的漏洞类型、来源或样本量。 |
| **范围准确性** | 答案内容不超出原文范围，不添加推断或假设信息。 |
| **问答对应性** | 答案需直接回应问题，避免冗余或偏题内容。 |
| **任务聚焦性** | 保留与"数据集属性"相关的问题，剔除与模型方法无关内容。 |
| **关键信息完整性** | 明确时间、数据格式、标注一致性等指标。 |
| **术语统一** | 保持技术术语一致（如统一使用 "XSS" 而非 "跨站脚本攻击"）。 |
| **重复合并** | 同类问题仅保留一个，避免冗余。 |
| **定量明确化** | 模糊表述需具体化（如"质量较高"→"标注一致率Kappa=0.85"）。 |

✅ **所有数据集均经 Web 安全专家人工 Review。**

---

## 六、数据增强（Data Augmentation）

### 6.1 面临的问题

在大模型训练中，**数据的规模与质量** 直接决定了模型的性能，但实际中我们常面临以下两大矛盾：

- **数据稀缺性**
  高质量语料（如学术文献、专业文本）总量有限。
  公开数据集（如 *C4*、*RefinedWeb*）经严格过滤后仅保留不到 **10%** 的原始内容，难以支撑模型持续扩展。

- **重复退化问题**
  在传统深度学习中，数据重复可帮助模型稳定训练；
  但在 **LLM 训练** 中，过度重复反而会导致：
  - 泛化能力下降
  - 优化稳定性变差
  - 模型出现"记忆化"现象

例如：
当使用 **1950 亿 tokens** 的高质量数据训练 **130 亿参数模型** 时，若直接重复 10 次，
模型在 **GSM8K 数学题** 的准确率下降 23%，验证损失上升 17%。

👉 说明数据重复并非简单的"量的补充"，而是需要**质的多样性重构**。

### 6.2 字节跳动 MGA 数据增强方法

字节跳动 Seed 团队在论文 [Reformulation for Pretraining Data Augmentation](https://arxiv.org/abs/2407.11398) 中提出了 **Massive Genre-Audience (MGA)** 方法，通过轻量级框架将现有语料重构为多样化变体。

其核心思想是：
> 基于不同 "体裁（Genre）" 和 "受众（Audience）" 生成语义多样的内容变体，
> 在保留核心知识的同时创造新的表达形式。

虽然论文主要用于预训练数据增强，但该思路在 **微调阶段的数据集构造** 同样适用。

### 6.3 MGA 方法核心概念

#### 🧩 "Massive" 的含义

- **大规模多样性生成**：每篇原始文档生成多个"类型-受众"组合。
  论文中默认生成 **5 对 GA 组合**，实现 **3.9 倍 token 扩展**。
- **覆盖广泛场景**：适用于大规模语料扩展，解决数据稀缺与重复问题。

#### 🧠 "Genre-Audience" 的含义

- **Genre（类型）**：知识表达框架
  - 沟通目的（教育 / 分析 / 叙事）
  - 内容结构（教程 / 论文 / 对话体）
  - 语言风格（学术 / 故事 / 通俗）
  - 知识深度（入门 / 高级）

例如：
> 将同一篇科普文章重构为"学术论文"或"儿童故事"，语言风格和结构不同，但核心知识相同。

- **Audience（受众）**：目标读者群体
  - 年龄、职业、教育背景
  - 知识基础与动机

例如：
> "急救指南"面向医学生会加入专业术语，面向普通职员则更注重实用性。

### 6.4 MGA 的技术实现

MGA 包括三个核心阶段：

#### 🏗 阶段 1：Genre-Audience 对生成

- 使用 **3.3B 参数的混合专家模型（MoE）**
- 从原始文档自适应提取 **5 组** 不同的"体裁-受众"组合
- 示例：
  ```
  "学术论文 - 科研人员"
  "对话体 - 老年人"
  "教科书 - 中学生"
  ```

#### 🧱 阶段 2：文本重构

- 使用量化后的轻量级工具模型重写文本
- 依据 GA 对调整语言、结构、知识深度
- 例如：
  - 面向小学生 → 对话体故事，减少术语
  - 面向研究者 → 强化数据论证与结构

#### ⚖ 阶段 3：质量控制（Limited Consistency 准则）

- 通过 LLM 裁判模型评估文本一致性：
  - **允许表达不同**
  - **但必须保留核心信息**
  - 若重构丢失核心信息则判为无效（得分 < 3）

### 6.5 MGA 的实验效果

| 模型规模 | 任务类型 | MGA 提升幅度 | 备注 |
|-----------|------------|----------------|------|
| 13B → 130B | 推理任务（TriviaQA、GSM8K） | ↑ 2.03% – 15.47% | 17B 模型在 GSM8K 解题率从 7.81% → 13.87% |
| 同上 | 知识任务（MMLU-Pro） | ↑ 2.15% | 捕捉知识多维表达 |
| - | 抗重复能力 | 验证损失上升仅 0.08（vs. 0.25） | 泛化能力增强 |

相比 Cosmopedia、Nemotron 等方案，MGA 的优势：
- 不依赖超大模型（3.3B 即可）
- 无需复杂种子模板
- 平衡多样性与保真度

### 6.6 为什么 MGA 有效？

| 挑战 | MGA 解决思路 |
|------|----------------|
| 数据重复导致"记忆偏差" | 不同体裁打破固定表达模式，迫使模型学习抽象语义 |
| 特异性记忆问题 | 促使模型学习跨体裁通用模式，提高泛化性 |
| 合成数据坍塌 | 动态 GA 对生成使嵌入分布更广泛、更自然 |

### 6.7 在 Easy Dataset 中使用 MGA

我们使用 [Easy Dataset (EDS)](https://github.com/ConardLi/easy-dataset) 1.3.6 版本对数据集进行 MGA 增强。

#### 步骤概览：

1. **创建项目并上传文献**
   在 EDS 中创建新项目，上传待处理文献。

2. **启用 MGA 模式**
   - 文献处理模型中启用 "MGA" 选项
   - 系统将基于文献内容自动生成 **5 个 Genre-Audience 对（GA 对）**
   - 用户可选择：
     - 自动生成
     - 手动调整
     - 启用/禁用或删除部分 GA 对

3. **批量生成 GA 对**
   - 对全部文献批量创建 GA 对
   - 可选择追加或覆盖模式

4. **查看 GA 详情**
   - 在文献列表中查看生成的 GA 对
   - 每条文献的 GA 对都会用于后续问答与数据生成

5. **数据量扩展**
   - 启用 MGA 模式后，数据量呈倍数增长
   - 例如：
     - 1500 字文本在基础模式下生成 6 个问题
     - 启用 5 个 GA 后生成 **30 个问题**

⚠ 注意：
启用 MGA 会显著增加 Token 消耗与生成时间。

#### 示例（幽默科普型 / 对技术感兴趣的中学生）

```json
{
  "genre": "Humorous Science Story",
  "audience": "Tech-Interested Teenagers", 
  "question": "为什么 XSS 攻击能绕过常规防护？请用一个日常生活的比喻解释。",
  "answer": "就像有人在你家的邮箱里偷偷放广告，XSS 也是在网页中偷偷塞入恶意脚本。"
}
```

### 6.8 后续优化方向

* 在项目配置中支持 **全局 GA 启用**（默认开启）
* 在问题与数据集管理中展示关键 GA Pair 信息
* 批量生成 GA 对时显示生成进度条
* 扩展 GA 结构以支持更多字段
* 引入 **LLM Judger** 系统：
  * 自动评估生成样本质量
  * 过滤低分样本，提升最终数据集质量
