# 普通 GNN（消息传递/MPNN）通用流程

**目标**：用图结构 + 节点特征，迭代更新每个节点表示，最后做节点/边/图级任务。

1. **输入**

- 图 G=(V,E)G=(V,E)，邻接关系 AA
- 节点特征矩阵 X∈R∣V∣×dX\in\mathbb{R}^{|V|\times d}
   -（可选）有标签的节点集合用于监督

1. **一层 GNN 的通用模板**

mij(l)=MESSAGE(l)(hi(l), hj(l), eij)mˉi(l)=AGGREGATE(l)({ mij(l):j∈N(i) })hi(l+1)=UPDATE(l)(hi(l), mˉi(l))\begin{aligned} m_{ij}^{(l)} &= \text{MESSAGE}^{(l)}\big(h_i^{(l)},\,h_j^{(l)},\,e_{ij}\big) \\ \bar m_i^{(l)} &= \text{AGGREGATE}^{(l)}\big(\{\,m_{ij}^{(l)}: j\in\mathcal N(i)\,\}\big) \\ h_i^{(l+1)} &= \text{UPDATE}^{(l)}\big(h_i^{(l)},\,\bar m_i^{(l)}\big) \end{aligned}

常见：MESSAGE 就是对邻居做线性变换；AGGREGATE 用 **sum/mean/max**；UPDATE 再做一层线性+激活。

1. **以 GCN 为例（最典型的“普通 GNN”）**

A~=A+I,D~ii=∑jA~ij\tilde A = A + I,\quad \tilde D_{ii}=\sum_j \tilde A_{ij}H(l+1)=σ ⁣(D~−1/2A~ D~−1/2H(l)W(l))H^{(l+1)}=\sigma\!\big(\tilde D^{-1/2}\tilde A\,\tilde D^{-1/2} H^{(l)} W^{(l)}\big)

- **权重分配**：由 D~−1/2A~ D~−1/2\tilde D^{-1/2}\tilde A\,\tilde D^{-1/2} 固定给定（度归一化），**所有邻居一视同仁**（结构决定的、不可学习）。
- **代价**：每层 O(∣E∣d)O(|E|d)。

1. **堆叠与读出**

- 叠 2–3 层（太多会过平滑）
- 节点分类：最后一层直接输出 logits
- 图分类：对节点表示做 pooling（mean/sum）再分类

1. **训练/推理**

- 训练：有监督交叉熵（只对有标签的节点计算），Adam
- 推理：关闭 dropout/BN，前向一次得到预测

------

# GAT（Graph Attention Network）流程

**核心**：把「邻居聚合权重」从**固定**变为**可学习的注意力权重**，为不同邻居分配不同重要性。

1. **注意力打分（每条边）**

eij(l)=LeakyReLU ⁣( a(l)⊤[ W(l)hi(l) ∥ W(l)hj(l) ])e_{ij}^{(l)}=\text{LeakyReLU}\!\big(\,a^{(l)\top}[\,W^{(l)}h_i^{(l)} \,\Vert\, W^{(l)}h_j^{(l)}\,]\big)

- 共享参数：线性变换 W(l)W^{(l)} 和打分向量 a(l)a^{(l)}

1. **邻居内归一化（softmax）**

αij(l)=exp⁡(eij(l))∑k∈N(i)exp⁡(eik(l))\alpha_{ij}^{(l)}=\frac{\exp(e_{ij}^{(l)})}{\sum_{k\in\mathcal N(i)}\exp(e_{ik}^{(l)})}

- 得到「节点 ii 的注意力分布」，**不同邻居权重不同**。

1. **聚合更新**

hi(l+1)=σ ⁣(∑j∈N(i)αij(l) W(l)hj(l))h_i^{(l+1)}=\sigma\!\Big(\sum_{j\in\mathcal N(i)} \alpha_{ij}^{(l)}\, W^{(l)}h_j^{(l)}\Big)

1. **多头注意力**

- 中间层：**concat** 多个头的输出（更丰富）
- 最后一层：**average** 多头（更稳定）

1. **训练/推理**

- 与普通 GNN 相同（交叉熵、Adam、eval 模式关闭 dropout）
- 计算量：仍是 O(∣E∣d)O(|E|d)，但多了注意力打分与 softmax；高入度点更吃算力。

------

# 一眼看懂：GCN vs GAT

| 维度           | GCN（普通 GNN代表）      | GAT                                 |
| -------------- | ------------------------ | ----------------------------------- |
| 邻居权重       | 固定（度归一化矩阵给定） | **可学习**（注意力 αij\alpha_{ij}） |
| 区分邻居重要性 | 不能                     | **能**（按内容分配权重）            |
| 先验/归一化    | 依赖拉普拉斯/度矩阵      | 不需要谱先验                        |
| 超参           | 层数、隐藏维、dropout    | 另有**头数**、注意力温度等          |
| 计算           | 快、稳定                 | 稍重，尤其高入度时                  |
| 适用           | 同质、同构、较平滑关系   | 邻居贡献差异大、异质信息混杂场景    |

------

# 典型训练/推理步骤（两者通用）

1. 准备 A,X,ylabeledA, X, y_{\text{labeled}}，划分 train/val/test（或给定 mask）
2. 堆叠 2–3 层 GCN/GAT（GAT 设定 heads、dropout）
3. 损失：只对训练掩码上的节点做交叉熵
4. 早停：监控 val acc / loss
5. 推理：eval 模式前向，取 argmax

------

# 和 GoR 的关系（为什么论文里选 GAT）

- GoR 的图里**邻居类型多样**（文本块节点、历史响应节点），不同邻居对“全局摘要/检索”的贡献差异很大；
- **GAT 能学出“哪些邻居更关键”**，比固定权重的 GCN 更契合这类“筛选关键信息”的需求。

