以下笔记基于当前可见的课程材料、Lecture 11 Topic Modelling 讲义与转写稿进行重构，重点围绕 **Topic Modelling（主题建模）**、**LSA 回顾**、**NMF**、**PLSA**、**LDA** 以及它们之间的逻辑递进展开。课程讲义明确把 Topic Modelling 定义为一种 **unsupervised document classification / clustering method（无监督文档分类 / 聚类方法）**，并将本讲学习目标聚焦在 NMF、PLSA、LDA 的核心思想、步骤、优缺点和应用上。

# COMP6237 Data Mining — Lecture 11 Topic Modelling 最终复习笔记

## Finding Features II：从 LSA 到 NMF、PLSA、LDA
可以把这四个方法理解成一条**逐步修补前一个方法缺点**的路线：

```text
LSA  →  NMF  →  PLSA  →  LDA
找隐藏语义 → 让语义可解释 → 让语义变成概率模型 → 让新文档泛化更自然
```

课程总结页也基本按这个逻辑归纳：Topic Modelling 是无监督文档软聚类；LSA 有负权重难解释；NMF 得到非负 topics；PLSA 用概率方式引入 hidden topics；LDA 加入 Dirichlet priors 来改善新文档泛化。

---

# 1. LSA：为了解决“词表太稀疏、表面词不等于真实语义”的问题

**LSA（Latent Semantic Analysis，潜在语义分析）** 的提出，是为了解决原始 **document-word matrix（文档-词矩阵）** 太表面的问题。

原始词袋表示只能看到：

```text
document 里出现了哪些 word
```

但看不到更深层的语义关系。

例如：

```text
car
automobile
vehicle
```

这些词语义相近，但如果字面词不同，原始词频矩阵可能认为它们完全不同。

所以 LSA 想解决的是：

> **能不能从词和文档的共现关系中，找出隐藏的 latent semantic concepts（潜在语义概念）？**

它用 SVD 做降维：

```text
A ≈ UΣVᵀ
```

把高维稀疏词空间压缩成低维语义空间。

但是 LSA 的问题是：

> 它找到的 concept 权重可以为负，导致概念很难解释成自然语言中的 topic。课程讲义也指出 LSA 的 concept weights 没有限制，可以为负，因此难以解释。

所以 LSA 解决了：

```text
原始词频矩阵缺乏隐藏语义结构的问题
```

但留下了：

```text
concept 不够可解释的问题
```

---

# 2. NMF：为了解决“LSA 概念有负权重、难解释”的问题

**NMF（Non-negative Matrix Factorisation，非负矩阵分解）** 是在 LSA 之后引出的。

它要解决的核心问题是：

> **LSA 的 latent concept 虽然数学上有用，但权重可以正负混合，不像人能理解的主题。**

所以 NMF 加入一个关键约束：

```text
A ≈ WH
W ≥ 0
H ≥ 0
```

这样：

```text
W = topic-word association
H = document-topic association
```

而且所有权重都是非负的。

这样一个 topic 就可以解释为：

```text
Topic 1:
graph: 0.8
tree: 0.7
minor: 0.6
```

而不是 LSA 那种：

```text
Concept 1:
graph: +0.8
tree: +0.7
user: -0.5
system: -0.4
```

课程转写中也强调，NMF 的目标是保证 W 和 H 都是非负权重，这样就可以说某些词对 topic 做正贡献，从而更容易解释；W 表示 topic-word association，H 表示 document-topic association。

所以 NMF 解决了：

```text
LSA concept 不好解释的问题
```

但它仍然留下：

```text
它本质还是矩阵分解，不是完整概率生成模型
```

---

# 3. PLSA：为了解决“主题模型缺少概率解释”的问题

**PLSA（Probabilistic Latent Semantic Analysis，概率潜在语义分析）** 的提出，是为了把 LSA / NMF 这种“分解思想”变成一个概率模型。

它要解决的问题是：

> **能不能用概率方式解释：为什么某个 document 会出现某个 word？**

PLSA 引入隐藏主题 `z`，认为：

```text
文档 d 中出现词 w
是因为：
先从文档 d 的 topic distribution 里选一个 topic z，
再从 topic z 的 word distribution 里生成 word w。
```

核心公式是：

```text
P(w | d) = Σ_z P(w | z) P(z | d)
```

其中：

```text
P(z | d) = 文档 d 中 topic z 的比例
P(w | z) = topic z 生成 word w 的概率
```

课程讲义中也说，PLSA models co-occurrence，`P(w|z)` 和 `P(z|d)` 被建模为 multinomial distributions，并可用 EM 算法训练。

所以 PLSA 解决了：

```text
NMF 只是矩阵分解、缺少概率生成解释的问题
```

但它留下一个新问题：

```text
每个训练文档都有自己的 P(z|d)，新文档来了没有现成的 P(z|d)，需要额外 fitting step
```

课程转写中也提到，PLSA 对每个新文档都要估计 document-level distribution，这不够理想，因此引出了 LDA。

---

# 4. LDA：为了解决“PLSA 对新文档泛化不自然”的问题

**LDA（Latent Dirichlet Allocation，潜在狄利克雷分配）** 是 PLSA 的进一步发展。

它要解决的问题是：

> **PLSA 需要为每个训练文档单独学习一个 topic mixture，新文档来了还要额外拟合，这种泛化方式不够自然。**

LDA 的做法是：

```text
不要直接把每篇文档的 P(z|d) 当成独立参数死记下来；
而是假设文档的 topic mixture θ 来自一个 Dirichlet prior。
```

也就是：

```text
θ_d ~ Dirichlet(α)
φ_z ~ Dirichlet(β)
```

其中：

```text
θ_d = document-topic distribution
φ_z = topic-word distribution
```

这样 LDA 学到的不只是训练文档各自的 topic mixture，而是学到一种更一般的生成机制：

```text
一般文档的 topic mixture 应该长什么样；
一般 topic 的 word distribution 应该长什么样。
```

课程材料中也说，LDA 给 PLSA 加上 Dirichlet priors，从而对新文档有更好的 generalization。

所以 LDA 解决了：

```text
PLSA 对新文档需要额外 fitting、泛化能力不够自然的问题
```

同时它也保留了概率主题模型的优点：

```text
document = mixture of topics
topic = distribution over words
```

---

# 5. 最清晰的对比总结

|方法|主要想解决的问题|它怎么解决|新产生 / 仍存在的问题|
|---|---|---|---|
|**LSA**|原始词频矩阵太稀疏、只看表面词，缺少隐藏语义|用 SVD 找 latent semantic concepts|concept 可正可负，难解释|
|**NMF**|LSA 的 concept 有负权重，解释困难|强制 `W ≥ 0, H ≥ 0`，得到非负 topic-word/document-topic 表示|仍是矩阵分解，不是概率生成模型|
|**PLSA**|NMF/LSA 缺少概率解释|引入 hidden topic `z`，用 `P(w|z)`和`P(z|
|**LDA**|PLSA 对新文档泛化不自然，容易像给训练文档记参数|给 topic mixture 和 word distribution 加 Dirichlet priors|推断更复杂|

---

# 6. 一句话总记忆

> **LSA 是为了找隐藏语义；NMF 是为了让隐藏语义非负、可解释；PLSA 是为了给主题加入概率生成解释；LDA 是为了在 PLSA 基础上加入先验，使模型对新文档更自然、更泛化。**
---

# 0. 本讲核心主线

这一讲的主线非常清楚：

> 我们想从一组没有标签的文档中，自动发现隐藏的主题。  
> 但主题不仅要“数学上有用”，还要“人能解释”。

因此，整节课其实是在回答一个问题：

> **如何把文档表示成若干隐藏主题的组合，并且让这些主题具有可解释性？**

课程中的方法递进关系如下：

```text
LSA
↓
用线性代数找 latent concepts
但权重可正可负，解释困难

NMF
↓
仍然是矩阵分解
但强制所有权重非负，所以主题更容易解释

PLSA
↓
把主题建模成概率模型
文档是主题分布，主题是词分布
概率天然非负，可解释

LDA
↓
在 PLSA 基础上加入 Dirichlet priors
更像完整生成模型
对新文档泛化更自然
```

一句话总结：

> **LSA 解决“能不能找到隐藏结构”；NMF 解决“隐藏结构能不能解释”；PLSA 解决“能不能用概率方式解释”；LDA 解决“能不能更好泛化到新文档”。**

---

# 1. Topic Modelling 的问题定义

## 1.1 Topic Modelling 是什么？

**Topic Modelling（主题建模）** 是一种 **unsupervised learning（无监督学习）** 方法。它的目标不是用已有标签训练分类器，而是：

> 给定一批文档，在没有人工标签的情况下，自动发现这些文档背后潜在的主题结构。

讲义中给出的例子是一组文档标题，其中一部分文档明显和 **human / user / system / interface / response** 有关，另一部分明显和 **graph / trees / minors** 有关。课程用这个例子说明：即使没有标签，我们也能直觉上看出文档集合中存在不同主题。

例如：

```text
m1: Human machine interface for ABC computer applications
m2: A survey of user opinion of computer system response time
m3: The EPS user interface management system
m4: System and human system engineering testing of EPS
m5: Relation of user perceived response time to error measurement

g1: The generation of random, binary, ordered trees
g2: The intersection graph of paths in trees
g3: Graph minors IV: Widths of trees and well-quasi-ordering
g4: Graph minors: A survey
```

直观上可以发现：

```text
Topic A: human / user / system / interface / response
Topic B: graph / trees / minors
```

但这里的关键是：

> 这些主题不是人工给定的，而是模型需要自动从词与文档的共现关系中发现出来。

---

## 1.2 为什么 Topic Modelling 是无监督问题？

因为训练数据中没有给出：

```text
文档 m1 属于 topic_user_system
文档 g1 属于 topic_graph_tree
```

模型只能看到：

```text
文档中出现了哪些词；
每个词出现了多少次；
不同文档之间有哪些词共现模式。
```

因此 Topic Modelling 的任务是：

> 从 **document-word matrix（文档-词矩阵）** 中发现隐藏主题。

---

## 1.3 Topic Modelling 和普通聚类的区别

普通的 **hard clustering（硬聚类）** 通常假设：

```text
一个样本只属于一个簇。
```

例如：

```text
doc_1 belongs to cluster A
doc_2 belongs to cluster B
```

但 Topic Modelling 更一般，它更像 **soft clustering（软聚类）**：

```text
一个文档可以同时属于多个主题，只是比例不同。
```

例如一篇文档可能是：

```text
70% machine learning
20% healthcare
10% privacy
```

而不是简单地说：

```text
这篇文档只属于 machine learning。
```

转写稿中也强调，Topic Modelling 不做 hard clustering，而是 soft grouping / soft clustering，因为一个 document 可能包含多个 topic 的 mixture。

---

# 2. 文档-词矩阵：Topic Modelling 的输入

## 2.1 基本输入形式

主题建模通常从一个 **document-word matrix（文档-词矩阵）** 开始。

设：

```text
A ∈ R^{m × n}
```

其中：

```text
m = number of words / features
n = number of documents
A_{ij} = word_i 在 document_j 中的权重
```

这个权重可以是：

```text
word count
TF
TF-IDF
normalized frequency
```

在本讲中，为了理解 NMF、PLSA、LDA，可以先把它理解成：

> 每一列是一个文档，每一行是一个词，矩阵中的值表示某个词在某个文档中的重要程度。

---

## 2.2 一个小例子

假设词表是：

```text
["user", "system", "response", "graph", "tree", "minor"]
```

文档是：

```text
d1: user system response
d2: user response time
d3: graph tree minor
d4: graph tree
```

那么 document-word matrix 可以近似写成：

```text
          d1  d2  d3  d4
user       1   1   0   0
system     1   0   0   0
response   1   1   0   0
graph      0   0   1   1
tree       0   0   1   1
minor      0   0   1   0
```

Topic Modelling 希望自动发现：

```text
Topic 1: user, system, response
Topic 2: graph, tree, minor
```

并进一步告诉我们：

```text
d1 mostly belongs to Topic 1
d2 mostly belongs to Topic 1
d3 mostly belongs to Topic 2
d4 mostly belongs to Topic 2
```

如果一篇文档同时包含两类词，比如：

```text
d5: user system graph
```

那么它可能是：

```text
50% Topic 1
50% Topic 2
```

这就是 soft clustering。

---

# 3. LSA 回顾：为什么需要更可解释的方法？

## 3.1 LSA 是什么？

**LSA（Latent Semantic Analysis，潜在语义分析）** 是上一讲中的方法。它使用线性代数，尤其是 **SVD（Singular Value Decomposition，奇异值分解）**，从 document-word matrix 中找到隐藏的 **latent concepts（潜在概念）**。

大致形式是：

```text
A ≈ U Σ V^T
```

它可以把原始高维词空间压缩到低维语义空间，从而发现一些词和文档之间的隐藏结构。

---

## 3.2 LSA 的优点

LSA 的优点是：

```text
1. 可以发现词和文档之间的隐藏关系；
2. 可以进行降维；
3. 可以缓解同义词、相关词带来的稀疏性问题；
4. 数学上非常清楚。
```

例如：

```text
user, interface, system, response
```

这些词经常在相似文档中共同出现，LSA 可能会把它们放在同一个 latent concept 中。

---

## 3.3 LSA 的核心问题：权重可以为负

讲义指出，LSA 找到的 concepts 是不同文档中词的线性混合，但它的权重 **unconstrained（不受约束）**，所以可以是正数，也可以是负数。结果是：这些概念虽然数学上有用，但很难解释。

例如某个 LSA concept 中可能出现：

```text
+ graph
+ tree
- user
- system
```

问题是：

> “graph 和 tree 对这个主题是正贡献”还比较容易理解；  
> 但“user 和 system 对这个主题是负贡献”到底是什么意思？

如果我们想把一个 topic 解释给人听，通常希望说：

```text
这个 topic 和 graph、tree、minor 有关。
```

而不太希望说：

```text
这个 topic 和 graph、tree 正相关，和 user、system 负相关。
```

后者在数学上可以成立，但在人类解释上很不自然。

---

## 3.4 Deep Dive：为什么负权重难解释？

假设一个主题向量是：

```text
Topic z1:
graph: +0.8
tree: +0.7
minor: +0.6
user: -0.5
system: -0.4
```

数学上，它可能表示：

```text
这个方向上，graph/tree/minor 增加时，user/system 减少。
```

但主题建模的目标通常是给主题命名，例如：

```text
Topic z1 = graph theory
```

如果某个词是负数，我们很难把它自然地放进主题解释中。

负权重会导致两个问题：

### 第一，主题解释变得不直观

人类通常用“包含什么”来解释主题，而不是用“排斥什么”来解释主题。

容易理解的是：

```text
Topic: graph theory
Top words: graph, tree, minor, ordering
```

不容易理解的是：

```text
Topic: graph theory
Positive words: graph, tree, minor
Negative words: user, system, response
```

### 第二，主题不再像“组成部分”

如果权重都是非负的，我们可以说：

```text
文档 = 0.7 × Topic A + 0.3 × Topic B
```

这很像“用几个主题拼出文档”。

但如果允许负数，就可能出现：

```text
文档 = 0.7 × Topic A - 0.3 × Topic B
```

这个解释就很奇怪：

> 文档怎么能由“负的主题”组成？

所以课程接下来引入 NMF：

> 既然负权重难解释，那我们能不能强制所有权重非负？

---

# 4. NMF：Non-negative Matrix Factorisation

## 4.1 NMF 的核心思想

**NMF（Non-negative Matrix Factorisation，非负矩阵分解）** 的目标是把原始矩阵分解成两个较小的非负矩阵：

```text
A ≈ W H
```

讲义中明确给出 NMF 的基本形式：

```text
A ≈ W H
```

并说明 Topic Modelling 像 clustering，但我们希望得到 soft clusters，因为一个文档应该是多个 topic 的 weighted mixture。

其中：

```text
A ∈ R^{m × n}
W ∈ R^{m × k}
H ∈ R^{k × n}
```

含义是：

```text
m = 词的数量
n = 文档数量
k = 主题数量 / reduced dimensionality
```

---

## 4.2 W 和 H 分别代表什么？

讲义中说明：

```text
W tells us what the latent topics look like in terms of words.
H tells us how much each document belongs to those topics.
```

也就是说：

```text
W = word-topic association matrix
H = topic-document association matrix
```

更具体地说：

```text
W 的每一列 = 一个 topic 的词分布 / 词权重
H 的每一列 = 一个 document 的主题混合比例
```

---

## 4.3 用矩阵形状理解 NMF

假设：

```text
A: 1000 words × 500 documents
k = 10 topics
```

那么：

```text
W: 1000 words × 10 topics
H: 10 topics × 500 documents
```

矩阵分解为：

```text
A_{1000×500} ≈ W_{1000×10} H_{10×500}
```

解释：

```text
原始文档-词矩阵
≈
每个 topic 由哪些 words 构成
×
每个 document 由哪些 topics 构成
```

---

## 4.4 NMF 的可解释性来自哪里？

NMF 的关键约束是：

```text
W ≥ 0
H ≥ 0
```

也就是所有元素都非负。

这意味着：

```text
一个词对一个 topic 的贡献只能是 0 或正数；
一个 topic 对一个 document 的贡献只能是 0 或正数。
```

于是我们可以自然地解释：

```text
Topic 1:
graph: 0.8
tree: 0.7
minor: 0.6
ordering: 0.4
```

这就是一个与 graph theory 相关的 topic。

如果某个词权重接近 0，则表示它对这个 topic 几乎没有贡献。

---

## 4.5 Deep Dive：为什么 NMF 比 LSA 更像“parts-based representation”？

NMF 常被理解成 **parts-based representation（基于组成部分的表示）**。

因为它把一个对象表示成若干非负部分的加权和。

例如一个文档：

```text
d = 0.7 × Topic_user_system + 0.3 × Topic_graph_tree
```

这就像说：

```text
这篇文档主要讨论 user/system，也有少量 graph/tree 内容。
```

所有系数都是非负的，因此每个 topic 都是在“增加”文档内容，而不是用负数抵消另一个 topic。

这和现实中的解释非常一致：

```text
一篇文章可以包含更多 politics 内容；
一篇文章可以包含更少 entertainment 内容；
但很难说一篇文章包含 -30% entertainment 内容。
```

所以 NMF 的非负性让主题更像“语义积木”。

---

## 4.6 NMF 为什么是 soft clustering？

在 NMF 中，矩阵 H 的每一列表示一个文档在多个主题上的权重。

例如：

```text
        d1
z1     0.80
z2     0.10
z3     0.05
z4     0.05
```

这说明：

```text
d1 mostly belongs to topic z1
```

但它不是 hard assignment，因为其他 topic 也可以有非零权重。

另一个文档可能是：

```text
        d2
z1     0.45
z2     0.40
z3     0.10
z4     0.05
```

这说明：

```text
d2 同时属于 z1 和 z2。
```

因此 NMF 自然给出 soft clustering view。讲义中的 NMF 图也用 stacked bar 展示了不同文档可以混合多个 topic。

---

# 5. NMF 的优化目标

## 5.1 NMF 要解决什么优化问题？

NMF 要找到两个非负矩阵 W 和 H，使得：

```text
A ≈ W H
```

也就是重构误差尽量小。

常见目标函数有两种。

---

## 5.2 Euclidean Norm / Frobenius Norm 重构误差

第一种是欧氏距离，也常写成 Frobenius norm：

```text
minimize ||A - WH||_F^2
subject to W ≥ 0, H ≥ 0
```

直观含义：

> 如果 WH 很接近 A，说明 W 和 H 很好地重构了原始 document-word matrix。

也就是说：

```text
原始文档中的词频模式
≈
模型用 topics 组合出来的词频模式
```

误差越小，说明分解越能解释原始数据。

---

## 5.3 KL Divergence

第二种是 **KL Divergence（Kullback-Leibler Divergence，KL 散度）** 风格的目标：

```text
D(A || WH) = Σ_ij [ A_ij log(A_ij / (WH)_ij) - A_ij + (WH)_ij ]
```

讲义中也列出了 NMF 可以最小化 Euclidean norm 或 KL divergence，并指出 W 和 H 可以通过迭代式 EM-like process 找到。

直观理解：

```text
Euclidean distance 更像数值重构误差；
KL divergence 更像两个分布之间的差异。
```

对于文本数据，词频、词分布天然带有概率 / 计数性质，因此 KL divergence 很常见。

---

## 5.4 Deep Dive：为什么 KL divergence 更像概率解释？

如果我们把：

```text
A_ij = 真实数据中 word_i 在 document_j 中的重要性
(WH)_ij = 模型预测的 word_i 在 document_j 中的重要性
```

那么 KL divergence 衡量的是：

> 模型生成的词分布和真实词分布之间差多少。

如果一个词在真实文档中很重要，但模型给它很低权重，KL divergence 会惩罚它。

因此 KL divergence 的直觉是：

```text
不要只让 WH 在数值上接近 A；
还要让 WH 在“词分布形状”上接近 A。
```

---

# 6. NMF 的更新算法

## 6.1 基本过程

NMF 通常使用迭代更新：

```text
Input: A, number of topics k
Initialize W ≥ 0 randomly
Initialize H ≥ 0 randomly

while not converged:
    update W
    update H

return W, H
```

转写稿中也强调，NMF 从 random non-negative W and H initialization 开始，然后不断迭代，直到 reconstruction error 低于阈值或收敛。

---

## 6.2 为什么初始化必须非负？

因为 NMF 的约束是：

```text
W ≥ 0
H ≥ 0
```

如果初始 W、H 都是非负的，并且使用乘法更新规则，更新后通常仍然保持非负。

这就是 NMF multiplicative update 的一个重要性质：

> 用非负数乘以非负比例，结果仍然非负。

---

## 6.3 Euclidean Distance 下的乘法更新

讲义中给出 Euclidean distance 版本的 NMF 更新规则，大意是：

```text
W_ij ← W_ij * (A H^T)_ij / (W H H^T)_ij

然后对 W 做归一化

H_ij ← H_ij * (W^T A)_ij / (W^T W H)_ij
```

对应优化目标是：

```text
minimize ||A - WH||_F^2
subject to W ≥ 0, H ≥ 0
```

---

## 6.4 KL Divergence 下的乘法更新

KL divergence 版本的更新思想类似：

```text
Initialize W ≥ 0, H ≥ 0
repeat:
    update W using A / (WH)
    normalize W
    update H using A / (WH)
until convergence
```

对应优化目标是：

```text
minimize generalized KL divergence
subject to W ≥ 0, H ≥ 0
```

课程中并不要求深入推导每一步更新公式，而是要求理解：

```text
1. 初始化 W 和 H 为非负；
2. 通过迭代减少重构误差或 KL divergence；
3. 更新过程保持 W 和 H 非负；
4. 最终得到可解释的 topic-word matrix 和 document-topic matrix。
```

---

# 7. NMF 例子详解

## 7.1 W 矩阵：topic 如何由 words 表示？

假设 NMF 找到 4 个 topics：

```text
z1, z2, z3, z4
```

W 可能显示：

```text
z1: trees
z2: eps, human, interface
z3: user, response, time, survey
z4: graph, minors
```

那么我们可以给主题命名：

```text
z1 = tree-related topic
z2 = interface/system topic
z3 = user response topic
z4 = graph minors topic
```

因为所有权重都是非负的，所以我们只需要看每个 topic 中权重最高的词。

---

## 7.2 H 矩阵：document 如何由 topics 表示？

H 可能显示：

```text
m1: mostly z2
m2: mixture of z2 and z3
m3: mostly z2
m4: mostly z2
m5: mostly z3

g1: mostly z1
g2: mixture of z1 and z4
g3: mixture of z1 and z4
g4: mostly z4
```

这说明 NMF 不仅能发现主题，还能说明每篇文档和主题之间的关系。

---

## 7.3 Deep Dive：如何从 H 得到 hard assignment？

虽然 NMF 是 soft clustering，但也可以从 H 中得到 hard assignment。

方法很简单：

```text
对每个 document，看 H 中最大权重对应的 topic。
```

例如：

```text
d1:
z1 = 0.1
z2 = 0.8
z3 = 0.1

hard assignment: d1 belongs to z2
```

但要注意：

> hard assignment 是后处理结果，不是 NMF 的本质。  
> NMF 的本质是保留 mixture 信息。

如果考试问：

```text
Why does NMF provide soft clustering?
```

可以回答：

```text
Because each document is represented by a non-negative weighted combination of multiple topics in H, rather than being assigned to exactly one topic.
```

---

# 8. NMF 中 topic number / reduced dimensionality 的选择

## 8.1 k 是什么？

在 NMF 中：

```text
k = number of topics
```

也就是 reduced dimensionality。

如果：

```text
A ∈ R^{m×n}
W ∈ R^{m×k}
H ∈ R^{k×n}
```

那么 k 决定了模型会发现多少个 latent topics。

---

## 8.2 k 通常需要手动选择

讲义中指出，对 NMF 和 LSA 来说，reduced dimensionality 的大小通常是 manually chosen，可以参考 reconstruction error，也可以运行多次建立 consensus matrix，或检查多个随机初始化下的稳定性。

也就是说，k 并没有一个绝对正确答案。

---

## 8.3 k 太小的问题

如果 k 太小：

```text
真实有 5 个主题
但你只设 k = 2
```

模型会被迫把多个主题合并。

例如：

```text
Topic 1: user, system, response, graph, tree
Topic 2: minor, survey, time
```

这会导致主题混乱，不容易解释。

---

## 8.4 k 太大的问题

如果 k 太大：

```text
真实有 5 个主题
但你设 k = 50
```

模型可能会把一个自然主题切碎成很多小主题。

例如：

```text
Topic 1: user, system
Topic 2: interface, response
Topic 3: response, time
Topic 4: survey, opinion
```

这样 reconstruction error 可能变小，但主题不稳定，也可能难以解释。

---

## 8.5 Deep Dive：为什么 topic 越多 reconstruction error 越低，但不一定越好？

因为模型容量变大了。

就像聚类中：

```text
K-means 的 K 越大，within-cluster error 通常越小。
```

极端情况下：

```text
每个文档一个 topic
```

重构误差会非常小，但这没有学习到有意义的共享结构。

所以选择 k 要平衡：

```text
1. Reconstruction error
2. Topic interpretability
3. Stability across random initializations
4. Whether topics are meaningful for downstream task
```

课程转写稿中也强调：更多 topics 通常会降低 reconstruction error，但可能不稳定；topics 太少又可能无法表达文档内容，因此这是一个 trade-off。

---

# 9. Probabilistic Topic Models：概率主题模型

NMF 仍然是矩阵分解方法。接下来课程进入 **probabilistic models（概率模型）**。

概率主题模型的核心思想是：

```text
A document is a mixture of topics.
A topic is a distribution over words.
```

讲义中也明确写出：我们把 document 建模为 topics 的 mixture，把 topic 建模为 words 上的 distribution。

---

## 9.1 两个核心分布

概率主题模型中最重要的是两个分布：

```text
P(z | d)
P(w | z)
```

含义分别是：

```text
P(z | d): 在文档 d 中选择 topic z 的概率
P(w | z): 在 topic z 下生成 word w 的概率
```

---

## 9.2 生成一个词的过程

概率模型通常从 **generative idea（生成思想）** 出发。

生成文档中的每个词时：

```text
Step 1: Choose a topic according to the document's topic distribution.
Step 2: Choose a word according to that topic's word distribution.
```

例如某篇文档的 topic distribution 是：

```text
P(topic_A | d) = 0.7
P(topic_B | d) = 0.3
```

topic_A 的 word distribution 是：

```text
P(user | topic_A) = 0.3
P(system | topic_A) = 0.25
P(response | topic_A) = 0.2
```

topic_B 的 word distribution 是：

```text
P(graph | topic_B) = 0.4
P(tree | topic_B) = 0.3
P(minor | topic_B) = 0.2
```

生成一个词时：

```text
先以 0.7 的概率选择 topic_A；
再从 topic_A 的词分布中抽一个词，比如 user。
```

---

## 9.3 Deep Dive：为什么说 topic 是 distribution over words？

因为 topic 不是一个单独的词，而是一个词概率分布。

例如：

```text
Topic_graph:
P(graph) = 0.30
P(tree) = 0.25
P(minor) = 0.20
P(ordering) = 0.10
P(user) = 0.01
```

这个 topic 不等于某一个词，而是一个概率分布。

人类给 topic 命名时，一般看概率最高的几个词：

```text
graph, tree, minor, ordering
```

于是命名为：

```text
Graph Theory Topic
```

---

# 10. PLSA：Probabilistic Latent Semantic Analysis

## 10.1 PLSA 是什么？

**PLSA（Probabilistic Latent Semantic Analysis，概率潜在语义分析）** 是 LSA 的概率版本。

它引入隐藏变量：

```text
z = latent topic
```

**并用概率方式建模文档和词的共现关系。**

讲义中写出 PLSA 的基本生成过程：

```text
Select document d with probability P(d)
Select a latent class z with probability P(z|d)
Generate a word w with probability P(w|z)
```

并给出联合概率：

```text
P(d, w) = P(d) P(w|d)
        = P(d) Σ_z P(w|z) P(z|d)
```

PLSA 因此是在建模 document-word co-occurrence。

---

## 10.2 PLSA 的核心公式

PLSA 的核心是：

```text
P(w | d) = Σ_z P(w | z) P(z | d)
```

含义是：

> 文档 d 中出现词 w 的概率，是所有 topic 对这个词贡献的加权和。

具体解释：

```text
P(z | d) = 文档 d 中 topic z 的比例
P(w | z) = topic z 生成 word w 的概率
```

于是：

```text
如果 d 很偏向 topic_graph，
而 topic_graph 很容易生成 word graph，
那么 P(graph | d) 就会很高。
```

---

## 10.3 PLSA 的两个主要参数

PLSA 要学习：

```text
P(w | z): 每个 topic 的 word distribution
P(z | d): 每个 document 的 topic distribution
```

其中：

```text
P(d)
```

可以直接从 corpus 中估计，例如文档出现频率。

讲义中指出，P(w|z) 和 P(z|d) 被建模为 multinomial distributions，并可以用 **EM algorithm（Expectation-Maximization，期望最大化算法）** 训练。

---

## 10.4 PLSA 为什么可解释？

因为所有概率都是非负的：

```text
P(w | z) ≥ 0
P(z | d) ≥ 0
```

并且概率和为 1：

```text
Σ_w P(w | z) = 1
Σ_z P(z | d) = 1
```

所以它天然满足：

```text
topic 是 words 的非负组合；
document 是 topics 的非负组合。
```

这和 NMF 的解释很像，但 PLSA 进一步提供了概率语义。

---

## 10.5 Deep Dive：PLSA 和 NMF 的直觉关系

NMF 和 PLSA 都可以看成在分解 document-word 关系。

NMF 是：

```text
A ≈ W H
```

PLSA 是：

```text
P(w | d) = Σ_z P(w | z) P(z | d)
```

它们的结构非常相似：

```text
W      ≈ P(w | z)
H      ≈ P(z | d)
WH     ≈ P(w | d)
```

区别是：

```text
NMF 是矩阵分解；
PLSA 是概率生成模型。
```

NMF 中的权重只要求非负，不一定严格是概率分布。

PLSA 中的参数是概率，所以需要满足归一化约束。

---

## 10.6 PLSA 的训练直觉：EM algorithm

PLSA 有隐藏变量 z。我们只观察到：

```text
document d
word w
```

但不知道每个词是由哪个 topic 生成的。

因此需要 EM：

### E-step

估计：

```text
P(z | d, w)
```

也就是：

> 在文档 d 中看到词 w 时，它来自 topic z 的概率是多少？

根据贝叶斯公式：

```text
P(z | d, w) ∝ P(w | z) P(z | d)
```

完整形式：

```text
P(z | d, w) =
P(w | z) P(z | d)
/
Σ_{z'} P(w | z') P(z' | d)
```

### M-step

用 E-step 中估计的 topic assignment probability 更新：

```text
P(w | z)
P(z | d)
```

直觉：

```text
如果某个词经常被分配给某个 topic，
那么这个词在该 topic 下的概率就应该变大。

如果某篇文档中的词经常被分配给某个 topic，
那么这篇文档中该 topic 的比例就应该变大。
```

---

# 11. PLSA 的新文档问题

## 11.1 问题是什么？

PLSA 的一个重要缺点是：

> 对训练集中没有见过的新文档，需要额外 fitting step。

讲义中明确说明：训练后我们已经有 learned topics `P(w|z)`，但新文档不在训练集中，因此没有自己的 `P(z|d)`；测试时需要固定 `P(w|z)`，然后为新文档重新拟合一个新的 topic mixture。

---

## 11.2 为什么 PLSA 对新文档尴尬？

训练时，PLSA 学到了：

```text
P(w | z): topic-word distributions
P(z | d_train): 每个训练文档的 topic mixture
```

但对于一个新文档：

```text
d_new
```

PLSA 并没有一个直接的规则告诉你：

```text
P(z | d_new)
```

所以只能：

```text
1. 固定训练好的 P(w | z)
2. 初始化 P(z | d_new)
3. 迭代优化，让 P(z | d_new) 最能解释新文档中的 words
```

这就是额外 fitting step。

转写稿中也强调，对于新文档，需要估计第二个分布，也就是 document-level distribution，这使得 PLSA 在 inference 上不够 smart。

---

## 11.3 Deep Dive：为什么这会影响泛化？

因为 PLSA 的 `P(z|d)` 是针对每个训练文档单独学习出来的参数。

如果训练集有 1000 篇文档，就有 1000 个 document-specific topic distributions。

如果来了第 1001 篇文档，模型没有提前学到一个“如何为任意新文档生成 topic distribution”的机制。

这和监督学习中的模型不同。

例如分类器训练好后：

```text
f(x) -> y
```

新样本 x 可以直接输入。

但 PLSA 对新文档更像：

```text
固定 topics；
重新优化这个新文档的 topic mixture。
```

因此它对新文档的处理不够自然。

---

# 12. LDA：Latent Dirichlet Allocation

## 12.1 LDA 是什么？

**LDA（Latent Dirichlet Allocation，潜在狄利克雷分配）** 是 PLSA 的改进版。

它也是概率主题模型，但比 PLSA 更完整，因为它加入了：

```text
Dirichlet priors（狄利克雷先验）
```

讲义中写到：LDA 使用 Dirichlet prior 作用于每个文档的 topic distribution，是 fully generative，并通过 Bayesian inference 学习参数；对于小数据集，它通常优于 PLSA，否则结果相似。

---

## 12.2 LDA 和 PLSA 的核心区别

PLSA 直接估计：

```text
P(z | d)
P(w | z)
```

LDA 认为这些分布本身也是随机变量，并且来自 Dirichlet distribution：

```text
θ_d ~ Dirichlet(α)
φ_z ~ Dirichlet(β)
```

其中：

```text
θ_d = document d 的 topic distribution
φ_z = topic z 的 word distribution
```

也就是说：

```text
PLSA: 每篇文档直接有一个 P(z|d)
LDA: 每篇文档的 θ_d 是从 Dirichlet(α) 中抽出来的
```

---

## 12.3 LDA 的生成过程

LDA 假设文档是这样生成的：

### Step 1：为每个 topic 抽取 word distribution

```text
φ_z ~ Dirichlet(β)
```

含义：

```text
每个 topic z 都有一个 word distribution。
β 控制这个 topic 的 word distribution 长什么样。
```

### Step 2：为每个 document 抽取 topic distribution

```text
θ_d ~ Dirichlet(α)
```

含义：

```text
每篇 document d 都有一个 topic mixture。
α 控制一篇文档通常包含多少个 topic。
```

### Step 3：为文档中的每个词位置生成 topic

```text
z_dn ~ Multinomial(θ_d)
```

含义：

```text
对文档 d 的第 n 个词，先从 θ_d 中选一个 topic。
```

### Step 4：根据 topic 生成 word

```text
w_dn ~ Multinomial(φ_{z_dn})
```

含义：

```text
选定 topic 后，再从该 topic 的 word distribution 中生成具体 word。
```

---

## 12.4 LDA 图模型中的变量

LDA 中常见变量：

```text
α: document-topic distribution 的 Dirichlet prior
β: topic-word distribution 的 Dirichlet prior
θ_d: document d 的 topic mixture
φ_z: topic z 的 word distribution
z_dn: 第 d 篇文档第 n 个词的 topic assignment
w_dn: 观察到的 word
```

其中：

```text
w 是 observed variable
z, θ, φ 是 hidden variables
```

讲义中指出，从 corpus 中我们只观察到 words `w`，但 LDA 模型包含隐藏变量：每个词的 topic assignment `z`、每个文档的 topic mixture `θ`、每个 topic 的 word distribution `φ`；理想情况下我们想得到 posterior distribution。

---

# 13. Dirichlet Prior 深入解释

## 13.1 Dirichlet distribution 是什么？

**Dirichlet distribution（狄利克雷分布）** 是一种“分布上的分布”。

这句话非常重要：

> Dirichlet distribution does not generate a word directly.  
> It generates a probability distribution.

例如：

```text
θ_d ~ Dirichlet(α)
```

不是直接生成 topic，而是生成一个 topic probability vector：

```text
θ_d = [0.7, 0.2, 0.1]
```

然后才用这个 θ_d 去生成 topic：

```text
z ~ Multinomial(θ_d)
```

---

## 13.2 Deep Dive：什么叫“distribution over distributions”？

假设有 3 个 topics。一个 document-topic distribution 可能是：

```text
[0.9, 0.05, 0.05]
```

也可能是：

```text
[0.33, 0.33, 0.34]
```

也可能是：

```text
[0.5, 0.5, 0.0]
```

Dirichlet distribution 决定：

> 哪种 topic mixture 更可能出现。

所以它是：

```text
a distribution over possible topic distributions
```

也就是“分布的分布”。

---

## 13.3 α 控制什么？

`α` 控制的是：

```text
document-topic distribution θ_d 的形状
```

也就是：

> 一篇文档通常会包含很多 topic，还是只集中在少数 topic 上？

讲义中也说明，Dirichlet prior 中 `α` controls how many topics appear per document。

---

## 13.4 α 小：sparse topic mixture

如果：

```text
α < 1
```

通常会得到 sparse mixture：

```text
θ_d = [0.90, 0.05, 0.03, 0.02]
```

含义：

```text
一篇文档主要属于少数几个 topic。
```

这通常符合现实：

```text
一篇论文通常不会平均讨论 100 个主题；
它通常集中在 1-3 个主题上。
```

转写稿中也强调，小 α 会让 mixture 更 sparse，表示一个 document 只和少数 topic 关联；大 α 会让 document 同时由很多 topic 组成。

---

## 13.5 α 大：dense topic mixture

如果：

```text
α > 1
```

会得到更 dense / smooth 的 mixture：

```text
θ_d = [0.25, 0.25, 0.25, 0.25]
```

含义：

```text
一篇文档均匀混合多个 topics。
```

这有时不太符合主题建模的直觉，因为大多数文档不是平均讨论所有主题。

---

## 13.6 β 控制什么？

`β` 控制的是：

```text
topic-word distribution φ_z 的形状
```

也就是：

> 一个 topic 是由少数几个关键词主导，还是比较平均地覆盖很多词？

讲义中写到，`β` controls how many words dominate a topic。

如果 β 小：

```text
Topic z:
graph: 0.4
tree: 0.3
minor: 0.2
others: very small
```

这说明 topic 很集中，容易解释。

如果 β 大：

```text
Topic z:
many words have similar probabilities
```

这说明 topic 更平滑，但可能不够鲜明。

---

# 14. LDA 的推断：为什么需要 Bayesian Inference？

## 14.1 我们观察到了什么？

在真实 corpus 中，我们只观察到：

```text
w = words in documents
```

但不知道：

```text
每个词属于哪个 topic；
每个 document 的 topic mixture；
每个 topic 的 word distribution。
```

也就是：

```text
z hidden
θ hidden
φ hidden
```

所以我们想求：

```text
P(θ, φ, z | w, α, β)
```

这就是 posterior distribution。

---

## 14.2 为什么不能直接精确求？

因为所有隐藏变量互相依赖，而且组合数量巨大。

例如：

```text
1000 documents
每篇 500 words
20 topics
```

每个词都可能属于 20 个 topic。

那么 topic assignment 的可能组合是天文数字：

```text
20^(500000)
```

所以需要 approximate inference。

讲义中说明，LDA 使用 Bayesian inference 学习参数，原始 LDA paper 中使用的是用更简单分布近似 posterior distribution 并优化它。

---

## 14.3 常见推断方法

LDA 常见推断方法包括：

```text
Variational Inference
Gibbs Sampling
Collapsed Gibbs Sampling
Expectation Propagation
```

课程层面不要求深入推导，但要理解：

```text
LDA 不是直接观察 topic；
而是通过 words 反推出最可能的 topics、topic mixtures 和 word distributions。
```

---

# 15. LDA 对新文档为什么比 PLSA 更自然？

## 15.1 PLSA 的问题回顾

PLSA 中，每个 document 有自己的参数：

```text
P(z | d)
```

新文档没有这个参数，因此需要额外 fitting。

---

## 15.2 LDA 的优势

LDA 学到的不只是训练文档的 topic mixtures，还学到了：

```text
1. topic-word distributions φ
2. document-topic mixtures 的 prior α
```

所以对新文档：

```text
不需要重建整个模型；
只需要在已学习的 topic system 和 prior 下推断 θ_new。
```

讲义中也明确说明，LDA 测试时不需要 rebuild the whole model；训练已经学到 topic-word distributions 和 Dirichlet prior，新文档只需要推断 document-topic mixture。

---

## 15.3 Deep Dive：PLSA 和 LDA 处理新文档的差异

### PLSA

```text
Training:
learn P(w|z)
learn P(z|d_train) for each training document

Testing:
new document has no P(z|d_new)
must fit P(z|d_new) separately
```

### LDA

```text
Training:
learn topic-word distributions
learn/use priors α and β
infer posterior structure

Testing:
use learned topics and priors
infer θ_new for new document
```

LDA 的 prior 相当于告诉模型：

```text
一般来说，一篇文档应该如何混合 topics。
```

所以它对新文档更有指导，而不是完全从零开始拟合。

转写稿中也强调，LDA 通过 additional prior 增强了对 document-level distribution θ 的估计，不需要对每个新文档都从头做非常耗时的估计。

---

# 16. NMF、PLSA、LDA 的统一视角

虽然三者形式不同，但可以放在同一个框架中理解。

## 16.1 共同假设

三者都接受这个基本想法：

```text
document = mixture of topics
topic = mixture/distribution of words
```

---

## 16.2 不同点

|方法|核心表示|是否概率模型|是否非负|新文档处理|可解释性|
|---|---|---|---|---|---|
|LSA|SVD / linear algebra|否|否，允许负权重|可投影|较弱|
|NMF|A ≈ WH|否，矩阵分解|是|可通过固定 W 求 H|强|
|PLSA|P(w,d)|是|是，概率非负|需要额外 fitting|强|
|LDA|Bayesian generative model|是|是，概率非负|用 learned topics + priors 推断 θ_new|强，泛化更好|

---

## 16.3 一句话区分

```text
LSA:
Find latent semantic dimensions using SVD, but weights may be negative.

NMF:
Factorize document-word matrix into non-negative topic-word and topic-document matrices.

PLSA:
Model document-word co-occurrence probabilistically using latent topics.

LDA:
Add Dirichlet priors to PLSA-like topic model, making it fully generative and better for new documents.
```

课程总结页也给出类似结论：Topic modelling 是 unsupervised soft clustering；LSA 有负权重难解释；NMF 给出非负、parts-based topics；PLSA 引入 probabilistic hidden topics 但新文档需要额外 fitting；LDA 加入 Dirichlet priors，从而对新文档有更好的泛化。

---

# 17. 进阶难点：co-occurrence 到底是什么？

## 17.1 PLSA models co-occurrence

PLSA 说它在建模 co-occurrence，意思是：

```text
哪些 words 经常和哪些 documents 同时出现？
```

例如：

```text
user, system, response
```

经常出现在同一批文档中，因此它们可能属于同一个 topic。

```text
graph, tree, minor
```

也经常共同出现，因此属于另一个 topic。

---

## 17.2 为什么 co-occurrence 能发现 topic？

因为主题本质上是一种词的共现模式。

如果某组词经常一起出现：

```text
hospital, patient, diagnosis, treatment
```

我们可能推断出：

```text
medical topic
```

如果另一组词经常一起出现：

```text
graph, node, edge, tree
```

我们可能推断出：

```text
graph theory topic
```

Topic Modelling 就是在自动发现这种模式。

---

# 18. 进阶难点：topic 是真实存在的吗？

Topic 是 **latent variable（潜变量）**。

它不是数据中直接观测到的东西。

我们看到的是：

```text
words
documents
```

模型引入 topic 是为了更好解释：

```text
为什么这些 words 会在这些 documents 中一起出现？
```

因此 topic 是一种解释性结构。

它不一定是真实世界中“唯一正确”的分类，而是模型为了压缩和解释数据而学习出来的 hidden structure。

---

# 19. 进阶难点：为什么主题模型不是普通分类？

分类问题通常有标签：

```text
document -> label
```

例如：

```text
email -> spam / not spam
news -> politics / sports / finance
```

Topic Modelling 没有标签，而是自动发现：

```text
document -> mixture of latent topics
```

因此：

```text
分类 = supervised, labels known
主题建模 = unsupervised, topics discovered
```

同时，主题建模输出不是单一类别，而是 topic mixture：

```text
d = 0.6 topic_1 + 0.3 topic_2 + 0.1 topic_3
```

---

# 20. 进阶难点：为什么 LDA 叫 Allocation？

**Latent Dirichlet Allocation** 中：

```text
Latent = topics are hidden
Dirichlet = priors over multinomial distributions
Allocation = each word token is allocated to a latent topic
```

也就是说：

```text
每个词 token 都会被分配一个 topic assignment z。
```

例如文档：

```text
"graph tree user system"
```

LDA 可能推断：

```text
graph -> topic_graph
tree -> topic_graph
user -> topic_system
system -> topic_system
```

所以 allocation 指的是：

> 把每个 word occurrence 分配给某个 latent topic。

---

# 21. 进阶难点：LDA 中 word token 和 word type 的区别

这是理解 LDA 很重要的一点。

## 21.1 Word type

词表中的一个词项：

```text
"graph"
```

这是 word type。

## 21.2 Word token

文档中每一次具体出现：

```text
Document 1: graph appears once
Document 2: graph appears twice
```

每次出现都是一个 word token。

LDA 为每个 token 分配 topic。

例如：

```text
Document 1:
graph_1 -> topic_graph

Document 2:
graph_1 -> topic_graph
graph_2 -> topic_network
```

同一个 word type 在不同上下文中可以被分配到不同 topic。

这就是 LDA 比简单词聚类更灵活的地方。

---

# 22. 考试 / 复习重点

## 22.1 必须会解释的概念

```text
Topic Modelling
Soft Clustering
Document-word Matrix
Latent Topic
LSA
SVD
Negative Weight
NMF
Non-negative Constraint
W Matrix
H Matrix
Reconstruction Error
KL Divergence
PLSA
P(w|z)
P(z|d)
EM Algorithm
LDA
Dirichlet Prior
α
β
Posterior Inference
New Document Inference
```

---

## 22.2 高频问法 1：What is topic modelling?

可以答：

```text
Topic modelling is an unsupervised method for discovering latent topics in a collection of documents. Unlike hard clustering, it represents each document as a mixture of topics, so a document can partially belong to multiple topics.
```

中文解释：

```text
主题建模是一种无监督方法，用于从文档集合中发现潜在主题。它不是把每篇文档硬分到一个类别，而是把文档表示成多个主题的混合。
```

---

## 22.3 高频问法 2：Why is LSA difficult to interpret?

可以答：

```text
LSA uses unconstrained linear combinations of words, so topic/concept weights can be positive or negative. Negative weights are mathematically valid but difficult to interpret as semantic topics.
```

中文解释：

```text
LSA 的主题权重没有非负约束，因此可能出现负权重。负权重在数学上可以表示方向差异，但很难解释成“某个词属于某个主题的程度”。
```

---

## 22.4 高频问法 3：What do W and H mean in NMF?

可以答：

```text
In NMF, A ≈ WH. W is the word-topic matrix, where each column describes a topic as a non-negative mixture of words. H is the topic-document matrix, where each column describes a document as a non-negative mixture of topics.
```

中文解释：

```text
W 表示每个 topic 由哪些 words 构成；H 表示每篇 document 由哪些 topics 构成。
```

---

## 22.5 高频问法 4：Why does NMF give soft clustering?

可以答：

```text
Because each document is represented by a vector of topic weights in H. A document can have positive weights for several topics rather than being assigned to only one cluster.
```

---

## 22.6 高频问法 5：What is the generative idea of PLSA?

可以答：

```text
For each word in a document, PLSA first chooses a latent topic according to the document-specific topic distribution P(z|d), and then chooses a word according to the topic-specific word distribution P(w|z).
```

核心公式：

```text
P(w | d) = Σ_z P(w | z) P(z | d)
```

---

## 22.7 高频问法 6：What is the weakness of PLSA?

可以答：

```text
PLSA learns a separate topic distribution P(z|d) for each training document. For a new document, this distribution is not available, so an additional fitting step is required to infer its topic mixture.
```

---

## 22.8 高频问法 7：How does LDA improve PLSA?

可以答：

```text
LDA adds Dirichlet priors over document-topic and topic-word distributions. This makes it a fully generative Bayesian model and improves generalisation to new documents, because new document topic mixtures can be inferred under the learned topic system and prior assumptions.
```

---

# 23. 最终总复习图

```text
Input:
Document-word matrix A

Goal:
Discover latent topics without labels

1. LSA
A ≈ UΣV^T
+ finds latent semantic concepts
- weights may be negative
- hard to interpret

2. NMF
A ≈ WH, W ≥ 0, H ≥ 0
+ non-negative
+ interpretable topics
+ soft clustering
- need choose k manually
- optimization can depend on initialization

3. PLSA
P(w|d) = Σ_z P(w|z)P(z|d)
+ probabilistic interpretation
+ all probabilities non-negative
+ models word-document co-occurrence
- new documents need extra fitting step

4. LDA
φ_z ~ Dirichlet(β)
θ_d ~ Dirichlet(α)
z_dn ~ Multinomial(θ_d)
w_dn ~ Multinomial(φ_z)
+ fully generative
+ priors regularize estimation
+ better generalization to new documents
- inference more complex
```

---

# 24. 最终一句话记忆版

> **Topic Modelling 是无监督软聚类；LSA 能找潜在概念但负权重难解释；NMF 用非负矩阵分解让 topics 更像可解释的词组合；PLSA 把 topic 变成概率隐藏变量来解释 document-word co-occurrence；LDA 在 PLSA 基础上加入 Dirichlet priors，使主题模型更完整、更可泛化。**