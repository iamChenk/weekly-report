# Toward expert-level medical question answering with large language models

## 信息

**期刊**：Nature Medicine（Volume 31，March 2025，页码 943–950）

**代码链接**：

- 本文模型（Med-PaLM 2）的源代码与权重未开源，仅以“MedLM”产品形式对外提供；相关信息可见 https://cloud.google.com/vertex-ai/generative-ai/docs/medlm/overview 
- 文中使用的数据集及其代码仓库链接（用于复现实验）：
  - MedQA (USMLE) 数据集：https://github.com/jind11/MedQA 
  - MedMCQA 数据集：https://medmcqa.github.io 
  - PubMedQA 数据集：https://pubmedqa.github.io 
  - LiveQA (TREC 2017) 数据集：https://github.com/abachaa/LiveQA_MedicalTask_TREC2017 
  - MedicationQA 数据集：https://github.com/abachaa/Medication_QA_MedInfo2019 
  - MMLU Clinical Topics 数据集：https://huggingface.co/datasets/hendrycks_test

## 摘要

​		大规模语言模型（LLMs）在医学问答领域表现出极大潜力，其中 Med-PaLM 是首个在美国医师执照考试（USMLE）风格题目中取得“及格”分数的模型。然而，在长文本医学问答以及真实临床工作流程中，LLMs 仍面临诸多挑战。本文提出 Med-PaLM 2，通过结合基础大模型更新、医学领域微调，以及“集成精炼”和“链式检索”等新策略，弥合了这些差距。Med-PaLM 2 在 MedQA 数据集上取得高达 86.5% 的准确率，比 Med-PaLM 提升超过 19%，并在 MedMCQA、PubMedQA 和 MMLU 临床专题数据集上展现出显著性能提升。我们设计了详尽的人类评估框架，结果表明医师在九个临床评价维度中，有八个更偏好 Med-PaLM 2 的生成答案。Med-PaLM 2 在所有评估指标上都明显优于上一代模型，尤其是在针对 LLMs 局限性设计的对抗性数据集上（P < 0.001）。在使用真实临床问题进行的试点研究中，专科医师在 65% 的情况下更倾向于 Med-PaLM 2 的回答，而尽管专科医师的答案整体仍被更频繁地选中，专科与全科医师均认为 Med-PaLM 2 的回答在安全性上可与人类答案媲美，这表明该模型在真实医学应用场景中的潜力日益凸显。

## 方法实现细节

### 1、基于PaLM2的指令微调

**基础模型：**以 Google 内部训练好的 PaLM 2（一般包含上百亿参数）为初始化权重。

**微调数据集**：论文使用了 MultiMedQA 系列数据集的多个子集，包含多项选择题和开放式长文本问答

#### 微调流程

- 数据清洗与预处理
- 混合训练策略（将微调数据集打乱）
- 训练超参数（优化器：AdamW、学习率、epoch、batch size ....)
- 目标函数（多项选择题：交叉熵损失；长文本问答：标准最大似然）

### 2、提示策略与示例

#### 2.1、Few-Shot Prompting

先在 Prompt 中附加若干条示例问答对，让模型通过“示例”预先学习如何解题。每个示例包含“问题 → （若为选择题）选项 → 答案” 或“问题 → 长文本答案”两种格式。

#### 2.2、Chain of Thought (CoT)

在Few-Shot Prompting示例中不仅给出“问题 → 答案”，还需显式给出“推理链”（例如一步一步地思考为什么选 A 而不是其他选项）。这样在实际推理时，模型会模仿示例的“思维过程”生成一串中间推理步骤，最后得出答案。

**好处**：

1. 降低“直接猜答案”而缺乏逻辑性的风险；
2. 当遇到复杂需要多步医学知识综合的题目时，能够提高正确率。

#### 2.3、Self-Consistency (SC)

对于同一个问题，多次让模型“采样”生成不同的推理链和答案，然后通过投票或一致性判断来选出最常见的答案。

#### 2.4、Ensemble Refinement (ER)

ER 的关键在于两个阶段：

1. **阶段一（Sampling）**：和 SC 一样，用温度采样得到 N 条不同的“推理链 + 答案”。
2. **阶段二（Refinement）**：将阶段一所有 N 条的完整推理链（含答案）作为上下文，让模型生成一条“精炼版推理链 + 最终答案”。然后对阶段二生成的结果再次重复 M 次（论文中 M = 3），再做投票选出最优答案。

### 3、链式检索（Chain of Retrieval）

1. 输入问题
2. 初始回答生成
3. 将初始回答拆解成若干可检索的陈述
4. 将每个陈述转化为关键词
5. 利用关键词，调用搜索引擎（如Google Scholar）进行检索
6. 下载检索前几篇相关论文的摘要，并让模型生成简短概括
7. 原始问题+初始回答+外部文献的简短概括，拼接成新的Prompt，生成最终回答

### 4、人类评估框架与示例

虽然链式检索能增强事实依据，但最终仍需验证模型在真实人类评估体系下的表现。本文在人类评估方面设计了三大模块：**独立评价、成对比较评价和三方效用排名**。