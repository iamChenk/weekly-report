# Accurate and interpretable drug-drug interaction prediction enabled by knowledge subgraph learning

### 举例

```
G_DDI

V_DDI = { Aspirin, Warfarin }

E_DDI = { (Aspirin, "enhances bleeding risk", Warfarin) }

R_DDI = { "enhances bleeding risk" }

G_KG

V_KG = { Aspirin, Warfarin, Ibuprofen, COX1, Platelet, Hemostasis, ... }
R_KG = { "inhibits", "targets", "resembles", "causes", ... }

E_KG = {
  (Aspirin, "inhibits", COX1),
  (Warfarin, "affects", Hemostasis),
  (Ibuprofen, "inhibits", COX1),
  (Ibuprofen, "resembles", Aspirin)
}

```

模型的目标是：**对任意药物对 (h, t)，预测它们之间的交互关系类型 r**

#### Drug-flow subgraph construction

K跳封闭子图

方向性剪枝:移除所有不在“从 h 到 t 的路径”上的节点和边

#### Knowledge Subgraph Generation

计算连接强度，删除冗余/误导的边，新增resemble边