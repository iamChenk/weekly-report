## 论文

**AutoTimes: Autoregressive Time Series Forecasters via Large Language Models**

### 为什么可以用LLM来做时序预测？有什么优势？

1、LLM本质是自回归模型，擅长根据给定的上下文预测下一个token。这种特性可以直接应用于时 间序列预测。即通过将时序数据转换为令牌序列，LLM可以预测未来的多个时间点

2、可以处理多模态数据

### 方法及框架

![0327-photo-Autotimes](images\0327-photo-Autotimes.png)

### 关键技术

1、时序数据嵌入

2、利用时间戳信息生成位置嵌入

3、上下文预测：利用相关的上下文信息（如历史时间序列数据）作为提示符（Prompt），扩展预测的上下文范围。