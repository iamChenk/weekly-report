# 论文

Medical Graph RAG: Towards Safe Medical Large Language Model via Graph Retrieval-Augmented Generation

关键词：医药RAG框架、知识图谱、检索方法

![](images\0320-photo.jpg)

## 框架说明

### 1、Doc chunking

若段落P与当前数据块chunk语义主题相同，则将P加入当前块chunk中；否则，段落P作为新块。

### 2、Entriries Extraction

提取实体，**{Name，Type，Context}**

Name由文本提取或语言模型L生成

Tpye根据UMLS（Unified Medical Language System，统一医学语言系统）

Context由语言模型L提取上下文

### 3、Triple Linking

三层图：用户RAG（如来自电子病历）、医学文献图、医学词典图（基于UMLS）

计算余弦相似度，判断上下图层的实体是否建立连接

**{RAG entrity，source，definition}**

三元组邻居定义
示例：若k=1，ec的邻居包括
E1中 1 跳内的实体
E2中 2 跳内的文献实体（1+1（层差 1））
E3中 3 跳内的词典定义

### 4、relationship Linking

生成用户RAG有向图

### 5、Tag the Graphs

对每个图谱标记一个tag（由语言模型L）

根据tag聚类分层

有利于提升索引效率

### 6、U-Retrieval

自上而下精确检索

自下而上响应优化

## Dataset

**MIMIC-IV**：是由麻省理工学院与贝斯以色列女执事医疗中心联合发布的公开临床数据集，主要包含重症监护病房（ICU）患者的电子健康记录（EHR）。

**MedC-K**：针对中文医学领域构建的知识问答数据集，主要聚焦于医疗领域的知识挖掘与问答任务。

**FakeHealth**：主要用于健康领域中的虚假信息检测，旨在识别与健康相关的虚假或误导性内容。

**PubHealth**：聚焦于公共卫生领域，通常收集与公共健康、健康政策和健康行为相关的文献或报道。

**MultiMedQA（Test Data）**：一个多任务、多模态的医学问答数据集，旨在推动医学问答系统的发展。