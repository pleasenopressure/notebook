# COMP6237 Data Mining

# Lecture 10: Semantic Spaces / Latent Semantic Analysis 最终复习笔记

本讲主题是 **Semantic Spaces（语义空间）**，也就是如何把文本、词语、文档，甚至图像等对象表示成向量，使得语义相近的对象在几何空间中距离更近。课程的核心方法是 **Latent Semantic Analysis, LSA（潜在语义分析）**，其数学基础是 **Singular Value Decomposition, SVD（奇异值分解）** 和 **Truncated SVD（截断奇异值分解）**。讲义明确把本节课放在 “Finding Features I” 下，说明它的本质不是简单的信息检索，而是**从原始高维稀疏表示中寻找更低维、更独立、更语义化的特征**。

---

## 1. 本讲的核心问题：如何让机器判断文档是否“语义相似”？

课程一开始给出了一组文档标题，例如：

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

人一眼可以看出，这些标题大致可以分成两类：

第一类是 **human-computer / system / user interface** 相关文档；第二类是 **graph / tree / minors** 相关文档。

但是机器不能直接“理解”标题。机器看到的只是字符串，所以问题变成：

> 能否把文档表示成数学空间中的点，使得语义相似的文档自动靠近，语义不相似的文档自动远离？

这就是 **Semantic Space（语义空间）** 的基本动机。讲师在转写中强调，关键始终是 **representation（表示）**：只要我们能找到合适的表示，就能得到有意义的空间；有了空间之后，就能用几何方法计算相似度、聚类、检索。

---

## 2. Semantic Space 的基本思想

**Semantic Space（语义空间）** 的基本思想是：

> 把词语、文档或其他对象表示成向量，使得它们的向量分布能够反映它们的语义关系。

讲义中给出了三个关键点：

1. **Represent word meanings as vectors**  
    把词义表示为向量，并记录词的分布历史。
    
2. **Focus on semantic similarity**  
    关注语义相似性，而不只是表面字符串是否相同。
    
3. **Measure similarity using geometrical methods**  
    使用几何方法，例如 **cosine similarity（余弦相似度）**，衡量两个向量是否接近。
    

例如，讲义给出：

```text
cosine similarity(PC, Windows) = 0.77
cosine similarity(PC, window)  = 0.13
```

这里 **Windows** 指操作系统，和 **PC** 语义上更接近；而 **window** 指房子的窗户，虽然表面词形相近，但语义上与 PC 关系弱。因此，一个好的语义空间应该让：

```text
PC  ≈ Windows
PC  ≠ window
```

这背后的基本假设叫做 **Distributional Hypothesis（分布假设）**：

> 出现在相似上下文中的词，往往具有相似含义。

换句话说，如果两个词经常出现在类似文档、类似句子、类似上下文中，那么它们的向量也应该相似。

---

## 3. 从 Bag of Words 到 Term-Document Matrix

在进入 LSA 之前，需要先构建一个基础表示：**Term-Document Matrix（词项-文档矩阵）**。

假设有：

- (m) 个词；
    
- (n) 篇文档；
    

那么可以构造矩阵：


$A \in \mathbb{R}^{m \times n}$  


其中：

- 每一行对应一个 **term（词项）**；
    
- 每一列对应一个 **document（文档）**；
    
- (A_{ij}) 表示第 (i) 个词在第 (j) 篇文档中的权重。
    

这个权重可以是：

```text
raw count
TF
TF-IDF
normalized frequency
```

讲义中的例子使用了一个 term-document matrix，行包括：

```text
computer, eps, graph, human, interface, minors, perceived,
response, survey, time, trees, user
```

列包括：

```text
m1, m2, m3, m4, m5, g1, g2, g3, g4
```

这个矩阵描述了哪些词出现在了哪些文档中。

---

## 4. TF-IDF 的作用：不只是数词频，而是衡量“区分能力”

讲师在转写中提到，最开始通常使用 **bag of words（词袋模型）** 或 **TF-IDF** 来表示文档。TF-IDF 的核心目的不是简单统计“一个词出现多少次”，而是衡量一个词对某篇文档的区分能力。

### 4.1 TF：Term Frequency

**TF（词频）** 衡量一个词在某篇文档中出现得多不多。

如果某个词在文档中频繁出现，它可能与该文档主题相关。

例如，在图论文档中：

```text
graph
tree
minor
```

可能频繁出现。

在人机交互文档中：

```text
user
interface
computer
system
response
```

可能频繁出现。

### 4.2 IDF：Inverse Document Frequency

**IDF（逆文档频率）** 衡量一个词在整个语料库中是否常见。

如果一个词在所有文档中都出现，例如：

```text
the
of
and
system
```

它的区分能力就比较弱。

如果一个词只在少数文档中出现，例如：

```text
graph
trees
interface
response
```

它更可能帮助区分主题。

所以 TF-IDF 的直觉是：

$$
\text{TF-IDF}(t,d)=\text{TF}(t,d)\times \text{IDF}(t)  

$$

其中：

$$
\text{IDF}(t)=\log\frac{N}{df(t)}  
$$

- (N)：文档总数；
    
- (df(t))：包含词 (t) 的文档数量。
    

### 4.3 为什么 TF-IDF 仍然不够？

TF-IDF 可以构造初始向量，但它存在明显问题：

1. **高维**：词表越大，向量越长。
    
2. **稀疏**：多数文档只包含少量词，因此矩阵里大量是 0。
    
3. **同义词问题**：两个文档讲同一主题，但使用不同词，TF-IDF 可能认为它们不相似。
    
4. **噪声问题**：某些偶然出现的词可能影响相似度。
    
5. **表层匹配问题**：它主要基于词是否共同出现，而不是更深层的语义结构。
    

这就是为什么要引入 **LSA**。

---

## 5. LSA 的核心思想：用低秩近似发现“潜在语义概念”

**Latent Semantic Analysis, LSA（潜在语义分析）** 的核心是：

> 对 term-document matrix 做 **low-rank approximation（低秩近似）**，把高维稀疏的词-文档矩阵压缩到低维的 latent concept space（潜在概念空间）中。

讲义明确写到，LSA 对 term-document matrix 有两个假设：

1. **The matrix is noisy and should be de-noised**  
    词-文档矩阵包含噪声，应该去噪。
    
2. **The matrix is more sparse than it should be**  
    词-文档矩阵比真实语义结构更稀疏。
    

这两个假设非常重要。

---

## 6. 难点剖析：为什么 term-document matrix “比它应该的更稀疏”？

这是理解 LSA 的关键。

假设有两篇文档：

```text
d1: user interface design
d2: human computer interaction
```

从人类角度看，这两篇文档很可能语义相关，因为它们都与人机交互有关。

但是如果用词袋模型表示：

```text
d1 contains: user, interface, design
d2 contains: human, computer, interaction
```

它们几乎没有共同词。

在原始 term-document matrix 中，这两个文档的向量可能很不相似，因为共同非零位置很少。

但语义上，它们应该接近。

这就是 “more sparse than it should be” 的含义：

> 0 不一定代表真正没有关系；有时只是因为这篇短文刚好没有使用那个词。

LSA 通过低秩近似把这种稀疏矩阵压缩到潜在概念空间中，让：

```text
user, interface, human, computer
```

这些经常出现在相似上下文中的词，在低维空间中靠近。

于是，即使两篇文档没有大量共享词，它们也可能因为共享潜在概念而变得相似。

---

## 7. 难点剖析：LSA 为什么可以去噪？

假设原始矩阵中某个词偶然出现在某篇文档中，例如一个图论文档中偶然出现了 “computer” 这个词。

在原始 BoW / TF-IDF 中，这个非零值会影响文档向量。

但是 LSA 不是保留所有细节，而是只保留最重要的几个潜在模式。

如果 “computer” 在图论文档中的出现只是偶然噪声，它不会形成强大的全局共现结构，因此往往会被较小的奇异值捕捉。

截断 SVD 会丢弃较小奇异值对应的成分，因此这种噪声会被削弱。

所以 LSA 的去噪直觉是：

```text
强的、重复出现的、跨文档稳定存在的共现模式  → 保留
弱的、偶然的、局部的噪声模式              → 丢弃
```

这就是为什么低秩近似不仅是降维，也是语义平滑和去噪。

---

## 8. SVD 数学基础

LSA 使用的核心数学工具是 **Singular Value Decomposition, SVD（奇异值分解）**。

对于矩阵：

$$
[  
A \in \mathbb{R}^{m \times n}  
]

SVD 将其分解为：

[  
A = U \Sigma V^T  
]

$$
其中：

```text
A      : 原始 term-document matrix
U      : left singular vectors，左奇异向量矩阵
Σ      : singular values，奇异值对角矩阵
V^T    : right singular vectors 的转置
```

讲义中给出的维度是：

[  
$$
A_{m \times n}=U_{m \times p}\Sigma_{p \times p}V^T_{p \times n}  
$$
]

其中 (p) 是矩阵 (A) 的秩。

---

## 9. U、Σ、V 分别代表什么？

这是本讲最容易混淆的部分。

### 9.1 (U)：词与潜在概念的关系

在 term-document matrix 中，行是词，列是文档。

因此：

[  
U  
]

主要描述 **terms / words（词）** 如何与 latent concepts（潜在概念）相关。

更具体地说：

```text
U 的每一行：一个词在不同潜在概念上的权重
U 的每一列：一个潜在概念在所有词上的分布
```

例如某个概念可能在：

```text
graph
tree
minor
```

这些词上权重很高，那么它可能对应 “graph theory” 相关概念。

另一个概念可能在：

```text
user
interface
computer
response
```

这些词上权重高，那么它可能对应 “human-computer interaction” 相关概念。

讲师在转写中解释，低秩分解后的左侧向量可以用来描述一个词相对于概念的权重。

### 9.2 (\Sigma)：每个潜在概念的重要性

[  
\Sigma  
]

是对角矩阵，其中对角线上的值叫做 **singular values（奇异值）**。

奇异值通常从大到小排列：

[  
\sigma_1 \geq \sigma_2 \geq \cdots \geq \sigma_p  
]

奇异值越大，说明对应的潜在概念越重要，能解释原始矩阵中越多的信息。

所以：

```text
大的 singular value     → 重要的全局语义模式
小的 singular value     → 细节、噪声、局部变化
```

### 9.3 (V^T)：文档与潜在概念的关系

[  
V^T  
]

与文档有关。

如果 (A) 的列是文档，那么 (V^T) 的列可以理解为文档在潜在概念上的表示。

讲义中写到：

> Each column of (V_r) describes a document as a vector of weights with respect to (r) concepts.

也就是说，每篇文档不再被表示成几万个词的高维稀疏向量，而是被表示成几个潜在概念上的权重。

例如：

```text
m1 ≈ [0.8 HCI concept, 0.1 graph concept]
g1 ≈ [0.1 HCI concept, 0.9 graph concept]
```

这就是 LSA 的压缩效果。

---

## 10. Truncated SVD：只保留最重要的 r 个概念

完整 SVD 会保留所有奇异值和向量：

[  
A = U \Sigma V^T  
]

但是 LSA 的关键是 **Truncated SVD（截断 SVD）**：

[  
A \approx A_r = U_r \Sigma_r V_r^T  
]

其中：

- (U_r)：只保留前 (r) 个左奇异向量；
    
- (\Sigma_r)：只保留前 (r) 个最大奇异值；
    
- (V_r^T)：只保留前 (r) 个右奇异向量。
    

讲义强调，Truncated SVD 只使用最大的 (r) 个 singular values 及其对应的 left/right vectors，并得到矩阵 (A) 的低秩近似。这个近似具有一个重要性质：它最小化原矩阵 (A) 和近似矩阵 (\hat{A}) 之间的 **Frobenius norm（弗罗贝尼乌斯范数）** 差异。

也就是说，在所有 rank-(r) 的矩阵中，Truncated SVD 给出的 (A_r) 是最接近原矩阵 (A) 的那个。

---

## 11. 难点剖析：Frobenius norm 最小化是什么意思？

Frobenius norm 可以理解为矩阵版的欧几里得距离。

对于矩阵误差：


$$
A-\hat{A}  

$$

Frobenius norm 是：

$$

|A-\hat{A}| = 

\sqrt{  
\sum_i \sum_j (A_{ij}-\hat{A}_{ij})^2  
}  
$$


也就是说，它把每个位置上的误差平方相加，再开方。

所以 “minimising the Frobenius norm” 的意思是：

> 在所有只能使用 (r) 个潜在维度的矩阵近似中，SVD 找到的那个近似矩阵整体重构误差最小。

这和 PCA 的思想非常接近：

```text
PCA         : 找保留最多方差的低维方向
Truncated SVD / LSA : 找最能重构 term-document matrix 的低秩结构
```

---

## 12. 选择 rank r：保留多少概念？

LSA 需要选择一个超参数：
r


也就是保留多少个潜在概念。

讲义中的图展示了随着 rank (r) 增加，保留信息量逐渐增加的曲线。rank 越大，cumulative explained energy 越高。

### 12.1 r 太小的问题

如果 (r) 太小：

```text
优点：去噪强，表示非常简洁
缺点：信息损失大，不同主题可能被混在一起
```

例如只保留 1 个概念，可能只能粗略区分“重要词”和“不重要词”，无法分开 HCI 和 graph theory。

### 12.2 r 太大的问题

如果 (r) 太大：

```text
优点：保留更多细节
缺点：噪声也被保留，降维效果变弱
```

如果 (r) 接近原矩阵秩，LSA 就退化成接近原始 TF-IDF 空间，失去了语义平滑和去噪效果。

### 12.3 实际选择方法

常见方法：

```text
1. 看 cumulative explained energy 曲线
2. 找 elbow point
3. 用验证集评估 retrieval / clustering / classification performance
4. 根据任务要求选择 r
```

对于考试，重点不是记住某个固定 (r)，而是理解：

> (r) 控制语义抽象程度与信息保留程度之间的 trade-off。

---

## 13. LSA 中的“concept”到底是什么？

LSA 中的 **concept（概念）** 不是人工预先定义的标签。

它不是说模型提前知道：

```text
concept 1 = human-computer interaction
concept 2 = graph theory
```

而是 SVD 从 term-document matrix 中自动发现的数学方向。

因此，concept 可以理解为：

> 一组在文档中共同变化、共同出现、共同区分文档的词语模式。

讲义中显示了不同 concept 上词语的权重分布。某个 concept 可能与：

```text
trees
graph
minors
```

关系很强；另一个 concept 可能与：

```text
user
interface
computer
human
response
```

关系较强。

但是讲义也强调：

> These concepts are not always particularly meaningful.

也就是说，有些 concept 可以被人解释成某个主题，但有些只是数学上最优的方向，可能混合了多个主题，难以用自然语言解释。

---

## 14. 难点剖析：为什么 concept 不一定能解释？

SVD 的目标是数学优化，不是生成可解释主题。

它优化的是：

[  
|A - U_r\Sigma_rV_r^T|_F  
]

而不是优化：

```text
让每个 concept 都能被人类命名
```

因此，一个 latent concept 可能是多个语义因素的混合。

例如某个 concept 的高权重词可能包括：

```text
user
time
response
survey
computer
graph
```

这时你很难说它到底是 “user interface” 还是 “survey” 或 “graph”。

所以 LSA 的 concept 有两层含义：

```text
数学含义：低维奇异向量方向
语义含义：可能对应某种主题或语义模式
```

考试或报告中不要过度声称：

```text
LSA always discovers clear human-readable topics.
```

更准确的说法是：

```text
LSA can reveal latent semantic patterns, but the resulting concepts may not always be directly interpretable in natural language.
```

---

## 15. 难点剖析：为什么 concept 中会出现负数？

在 LSA 的图中，有些词的 concept weight 是负数，有些是正数。

这容易让人误解为：

```text
正数 = 好
负数 = 坏
```

这是错误的。

SVD 中奇异向量的方向本身是任意的。

如果 (u) 是一个奇异向量，那么 (-u) 也是同样有效的方向，只要对应的 (v) 也改变符号，重构结果不变。

所以负数表示：

> 这个词位于该 latent axis 的相反方向。

它不表示词“不属于语义”，也不表示负面含义。

例如一个 concept 轴可能把：

```text
graph/tree/minor
```

放在负方向，把：

```text
user/interface/computer
```

放在正方向。

这说明它在区分两个语义群体，而不是判断好坏。

---

## 16. 文档坐标和词坐标如何计算？

讲义代码中给出的形式是：

```python
r = 2
Ur = U[:, :r]
Sigma = np.diag(s[:r])
VTr = VT[:r, :]

doc_coords = (Sigma @ VTr).T
word_coords = Ur @ Sigma
```

含义是：

### 16.1 文档坐标

[  
\text{doc_coords} = (\Sigma_r V_r^T)^T  
]

每一行对应一篇文档在 (r)-维 latent semantic space 中的坐标。

### 16.2 词坐标

[  
\text{word_coords} = U_r \Sigma_r  
]

每一行对应一个词在 (r)-维 concept space 中的坐标。

### 16.3 为什么要乘 (\Sigma_r)？

(\Sigma_r) 表示各个 concept 的重要性。

如果不乘 (\Sigma_r)，每个 concept 被看作同等重要。

乘上 (\Sigma_r) 后，重要 concept 对坐标贡献更大，不重要 concept 贡献更小。

---

## 17. 文档相似度：LSA 如何判断 m1 和 m2 相似？

LSA 把每篇文档变成一个低维向量后，就可以计算文档之间的相似度。

讲义中，当 (r=2) 时：

```text
cosine similarity(m1, m2) = 0.93
cosine similarity(g1, g2) = 0.83
cosine similarity(g1, m1) = 0.18
```

这说明：

```text
m1 和 m2 都属于 human-computer/system/user interface 主题，因此接近
g1 和 g2 都属于 graph/tree 主题，因此接近
g1 和 m1 主题不同，因此相似度低
```

讲义还指出，可以在这些 document vectors 上使用 clustering algorithms。

---

## 18. Cosine Similarity：为什么常用于语义空间？

**Cosine similarity（余弦相似度）** 衡量两个向量方向是否相似：

$\cos(\theta)=$$\frac{x \cdot y}{|x||y|}$  

其中：

- (x \cdot y)：点积；
    
- (|x|)：向量 (x) 的长度；
    
- (|y|)：向量 (y) 的长度。
    

### 18.1 为什么不用欧几里得距离？

在文本中，文档长度可能差异很大。

长文档通常有更多词，向量长度更大。

如果直接用 Euclidean distance，文档长度会严重影响距离。

Cosine similarity 更关注方向：

```text
方向相似 → 词分布比例相似 → 语义更可能相似
```

所以对于文本向量，cosine similarity 很常用。

### 18.2 直观例子

假设：

```text
d1 = [computer: 10, interface: 8, user: 7]
d2 = [computer: 100, interface: 80, user: 70]
```

d2 比 d1 长很多，但主题比例几乎一样。

Euclidean distance 会觉得它们差很多；cosine similarity 会觉得它们非常相似。

---

## 19. 词相似度：LSA 不仅能比较文档，也能比较词

LSA 还可以把词放到 latent semantic space 中。

讲义中给出：

```text
cosine similarity(human, interface) = 0.95
cosine similarity(human, user)      = 0.11
cosine similarity(graph, minor)     = 0.90
```

这说明：

```text
human 和 interface 在这个小语料中上下文相近
graph 和 minor 在图论文档中强相关
human 和 user 反而在这个低维表示中不那么接近
```

这也提醒我们：

> LSA 的相似度依赖语料和 rank 选择，不一定完全符合人类直觉。

---

## 20. 难点剖析：为什么 “human” 和 “user” 相似度可能低？

从常识看，human 和 user 很相关。

但 LSA 是基于当前语料的共现模式，而不是基于词典知识。

在这个小数据集中：

```text
human
```

可能主要出现在 m1、m4；

```text
user
```

可能主要出现在 m2、m3、m5。

如果它们没有强烈共享相似的文档分布，或者在 (r=2) 的投影中落在不同位置，那么 cosine similarity 可能较低。

所以 LSA 学到的是：

```text
corpus-specific semantic similarity
```

而不是 universal semantic truth。

这点非常重要。

---

## 21. LSI：Latent Semantic Indexing

**Latent Semantic Indexing, LSI（潜在语义索引）** 可以理解为 LSA 在文档检索中的应用。

讲义中写到，LSA 可以用于 document retrieval：

1. 给定一个 query；
    
2. 把 query 看作一个向量 (q)；
    
3. 将 (q) 投影到 document semantic space；
    
4. 与所有文档向量比较；
    
5. 返回最近的文档。
    

例如 query 是：

```text
"user response time"
```

它会被投影到同一个低维 semantic space 中，然后靠近 m2、m5 等与 user response time 相关的文档。

---

## 22. 难点剖析：query 如何投影到 LSA 空间？

这是 LSI 中最容易出错的数学点。

假设：

[  
A \approx U_r \Sigma_r V_r^T  
]

原始 query 是一个 term vector：

[  
q \in \mathbb{R}^{m}  
]

也就是说，query 和原来的词表对齐。例如：

```text
q = [computer: 0, eps: 0, graph: 0, human: 0, interface: 0,
     minors: 0, perceived: 0, response: 1, survey: 0,
     time: 1, trees: 0, user: 1]
```

### 22.1 如果文档坐标使用 (\Sigma_r V_r^T)

如果文档坐标定义为：

[  
d_j = \Sigma_r V_r^T[:,j]  
]

那么 query 可以投影为：

[  
q_r = U_r^T q  
]

因为 (U_r) 是 term space 到 latent concept basis 的映射。

### 22.2 如果文档坐标使用 (V_r^T)

有些教材或实现把文档坐标定义为 (V_r^T) 的列，而不是 (\Sigma_r V_r^T)。

这时 query 常写成：

[  
q_r = \Sigma_r^{-1}U_r^T q  
]

或者行向量形式：

[  
q_r^T = q^T U_r \Sigma_r^{-1}  
]

### 22.3 为什么会有两种写法？

因为不同实现对 latent coordinates 的缩放方式不同。

关键不是死记公式，而是保持一致：

```text
文档向量怎么定义，query 向量就要投影到同一个坐标系统。
```

如果文档坐标包含 (\Sigma_r)，query 也要使用对应缩放；如果文档坐标不包含 (\Sigma_r)，query 需要相应除以 (\Sigma_r)。

---

## 23. LSA 的完整算法流程

下面是一个标准 LSA / LSI 流程：

```text
Input:
    documents = [d1, d2, ..., dn]
    rank r

Step 1: Text preprocessing
    - tokenize
    - lowercase
    - remove stop words
    - optionally stemming / lemmatization

Step 2: Build vocabulary
    vocab = all terms after preprocessing

Step 3: Construct term-document matrix A
    A[i, j] = TF-IDF weight of term i in document j

Step 4: Apply SVD
    A = U Σ V^T

Step 5: Truncate
    keep top r singular values:
    A_r = U_r Σ_r V_r^T

Step 6: Build semantic coordinates
    doc_coords = (Σ_r V_r^T)^T
    word_coords = U_r Σ_r

Step 7: Use the semantic space
    - document-document similarity
    - word-word similarity
    - clustering
    - retrieval
    - visualization
```

---

## 24. Python 风格伪代码

```python
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.decomposition import TruncatedSVD
from sklearn.metrics.pairwise import cosine_similarity

docs = [
    "human machine interface for ABC computer applications",
    "a survey of user opinion of computer system response time",
    "the EPS user interface management system",
    "system and human system engineering testing of EPS",
    "relation of user perceived response time to error measurement",
    "the generation of random binary ordered trees",
    "the intersection graph of paths in trees",
    "graph minors IV widths of trees and well quasi ordering",
    "graph minors a survey"
]

# 1. Build TF-IDF matrix
vectorizer = TfidfVectorizer(stop_words="english")
A = vectorizer.fit_transform(docs)

# 2. Apply truncated SVD
r = 2
svd = TruncatedSVD(n_components=r, random_state=0)
doc_coords = svd.fit_transform(A)

# 3. Compare documents
sim = cosine_similarity(doc_coords)

# 4. Project query
query = ["user response time"]
q_vec = vectorizer.transform(query)
q_coords = svd.transform(q_vec)

# 5. Compare query with documents
query_doc_sim = cosine_similarity(q_coords, doc_coords)
```

这段代码对应的思想就是：

```text
TF-IDF matrix → Truncated SVD → semantic space → cosine similarity
```

---

## 25. LSA 与 PCA 的关系

讲义在 SVD recap 中提到：

> If (A) is a matrix of mean centred feature vectors, (V) contains principal components of the covariance matrix.

这说明 LSA 和 PCA 在数学上有紧密联系。

### 25.1 相同点

```text
PCA 和 LSA 都使用低维表示
PCA 和 LSA 都试图保留主要信息
PCA 和 LSA 都可以通过 SVD 实现
PCA 和 LSA 都能去除部分噪声
```

### 25.2 不同点

```text
PCA 通常用于连续数值特征矩阵，并且通常需要 mean centering
LSA 通常用于 term-document matrix，尤其是 count / TF-IDF 矩阵
PCA 的主成分强调方差最大方向
LSA 的 latent concepts 强调词-文档共现结构
```

所以可以说：

> LSA 是把 SVD / PCA 式的低秩思想应用到文本语义表示中的方法。

---

## 26. LSA 的优势

### 26.1 降维

原始 term-document matrix 可能有几万个词维度。

LSA 可以把文档压缩到几十或几百个 latent dimensions。

```text
原始空间：10000 维
LSA 空间：100 维
```

这样可以降低计算成本。

### 26.2 去噪

截断小奇异值可以去掉偶然词、局部噪声、低重要性模式。

### 26.3 缓解同义词问题

如果两个词经常出现在相似上下文中，LSA 可以让它们在潜在空间中接近。

例如：

```text
car
automobile
vehicle
```

即使不完全共现，也可能通过相似文档分布靠近。

### 26.4 可以用于聚类和检索

LSA 后的 document vectors 可以直接用于：

```text
K-means clustering
hierarchical clustering
nearest-neighbour retrieval
document ranking
query matching
```

### 26.5 可以扩展到非文本对象

只要对象可以表示成 “bag of something”，就可以构造矩阵并应用类似思想。

讲义后半部分就展示了 **bag of visual words（视觉词袋）** 的扩展。

---

## 27. LSA 的局限性

讲义明确列出了几个问题：

1. **Polysemous words are not captured well**  
    多义词无法很好处理。
    
2. **Word order is ignored**  
    词序被忽略。
    
3. **LSA assumes words and documents form a joint Gaussian distribution, but a Poisson distribution is observed**  
    LSA 的数学假设与词频数据的真实分布不完全匹配。
    

下面逐一解释。

---

## 28. 局限 1：多义词问题 Polysemy

**Polysemy（多义性）** 指一个词有多个含义。

例如：

```text
fit
```

可以是形容词：

```text
a fit person
```

也可以是动词：

```text
fit the model
```

还可以是名词：

```text
a sudden fit
```

LSA 会给每个词一个向量表示，但这个向量会平均它在所有上下文中的用法。

所以：

```text
fit_vector ≈ average(all meanings of fit)
```

这会导致不同含义混在一起。

### 28.1 为什么 LSA 处理不好多义词？

因为 LSA 的基本单位是词项。

如果词表中只有一个 “fit”，那么所有 “fit” 的出现都会累积到同一行。

它不会自动区分：

```text
fit_adjective
fit_verb
fit_medical_noun
```

因此，LSA 对 polysemy 的处理比较弱。

---

## 29. 局限 2：词序被忽略 Word Order Ignored

Bag of Words 的核心问题是：

> 它只记录词出现了什么，不记录词的顺序。

例如：

```text
dog bites man
man bites dog
```

BoW 表示完全一样：

```text
dog: 1
bites: 1
man: 1
```

但是意思完全不同。

因此，LSA 继承了 BoW 的缺点：它无法直接理解语法顺序。

### 29.1 n-gram 可以缓解这个问题

讲义提到：

> An n-gram is a sequence of n words.

例如：

```text
unigram: user, response, time
bigram: user response, response time
trigram: user response time
```

使用 n-gram 可以保留部分词序信息。

例如：

```text
New York
machine learning
response time
user interface
```

这些词组比单个词更有语义。

### 29.2 n-gram 的代价

但是 n-gram 会带来新问题：

```text
1. 词表规模急剧变大
2. 矩阵更加稀疏
3. 计算成本更高
4. 很多 n-gram 出现次数很少
```

所以 n-gram 是一种折中方案。

---

## 30. 局限 3：统计分布假设不匹配

讲义提到，LSA 假设 words and documents 形成某种 joint Gaussian distribution，但实际词频数据更接近 Poisson distribution。

这句话可以理解为：

LSA 使用的是线性代数方法，比较适合连续、近似高斯的数值矩阵。

但词频数据本质上是：

```text
非负
离散
计数型
大量为 0
长尾分布
```

这和高斯分布差异很大。

因此，LSA 虽然有效，但从概率建模角度看不是最自然的文本生成模型。

这也是后续方法，例如：

```text
pLSA
LDA
word2vec
neural embeddings
BERT embeddings
```

继续发展的原因之一。

---

## 31. Bag of Visual Words：LSA 不只适用于文本

讲义后半部分强调，虽然前面讨论的是 natural language 中的 **Bag of Words, BoW（词袋模型）**，但同样的数学思想可以应用于其他对象。

只要我们能把对象表示为某种 “occurrence composition（出现次数组合）”，就可以构建矩阵并应用 LSA。

例如图像搜索中，我们可以把图片表示成：

```text
circle:   2
triangle: 0
square:   3
cross:    0
star:     0
```

这就是 **Bag of Visual Words（视觉词袋）**。

### 31.1 图像作为 visual word vector

假设有三张图片：

```text
img_A: 2 circles, 3 squares, 0 triangles, 0 stars
img_B: 0 circles, 0 squares, 5 triangles, 0 stars
img_C: 1 circle, 2 squares, 0 triangles, 0 stars
```

可以构造 image-feature matrix：

```text
        circle  square  triangle  star
img_A      2       3       0       0
img_B      0       0       5       0
img_C      1       2       0       0
```

然后也可以对这个矩阵做低秩近似。

### 31.2 语义空间可以跨模态

讲师在转写中进一步解释，如果我们能把文本、图片、视觉词、人类关键词放在同一个矩阵中，就可以构造一个共同的低维空间。这样，文本和图像都可以被表示在同一个 semantic space 中。

这可以支持：

```text
1. 用文字搜索图片
2. 用图片搜索相似图片
3. 给未标注图片返回可能关键词
4. 找词语和视觉元素之间的关系
```

讲义中也列出低维空间可以用于：

```text
Find images using similar words
Find images with similar images
Return possible key words for an image
Find relationships between words, and between words and visual terms
```

---

## 32. 难点剖析：为什么图像也能用 BoW 思想？

这背后的抽象是：

```text
文本：文档 = word 的组合
图像：图片 = visual word 的组合
音频：片段 = acoustic feature 的组合
视频：视频 = visual/action token 的组合
```

只要我们能定义一个 vocabulary：

```text
text vocabulary
visual vocabulary
audio vocabulary
action vocabulary
```

就可以把对象表示成向量。

LSA 关心的不是对象原本是文字还是图像，而是：

```text
能否构造一个 occurrence matrix？
能否用低秩结构发现潜在概念？
```

所以 Semantic Space 是一种很通用的思想。

---

## 33. LSA 与现代深度学习 embedding 的关系

讲师在结尾强调，虽然现在是 deep learning 时代，但这种传统方法仍然重要，因为现代 deep learning 中很多思想都借鉴了这种基础表示学习思想。

### 33.1 共同点

LSA 和现代 embedding 方法都做一件事：

```text
把离散对象变成连续向量
让相似对象在向量空间中接近
用几何关系表达语义关系
```

例如：

```text
LSA document vector
word2vec word vector
BERT sentence embedding
CLIP image-text embedding
```

都属于广义上的 semantic representation。

### 33.2 不同点

LSA 是：

```text
linear
matrix factorization based
global co-occurrence based
static representation
```

而现代 deep learning embedding 通常是：

```text
nonlinear
neural network based
contextual
trained with large-scale objectives
```

例如 BERT 中的 “bank” 会根据上下文有不同表示：

```text
river bank
bank account
```

而 LSA 通常只有一个平均表示。

所以可以把 LSA 理解为现代 embedding 方法的传统线性前身。

---

## 34. LSA / LSI / SVD 之间的关系总结

```text
SVD
    是数学工具：把矩阵分解成 U Σ V^T

Truncated SVD
    是低秩近似方法：只保留前 r 个最大奇异值和向量

LSA
    是语义分析方法：对 term-document matrix 做 Truncated SVD，
    得到 latent semantic space

LSI
    是信息检索应用：把 query 投影到 latent semantic space，
    与文档向量比较并返回相似文档
```

一句话总结：

> LSA 用 Truncated SVD 把词-文档矩阵压缩到低维语义空间；LSI 用这个语义空间做检索。

---

## 35. 最容易考的概念辨析

### 35.1 Term-document matrix

```text
Rows    = terms / words
Columns = documents
Values  = counts or TF-IDF weights
```

### 35.2 Semantic space

```text
A vector space where semantic similarity is represented geometrically.
```

中文：

```text
一种向量空间，在这个空间中，语义相似的对象距离更近。
```

### 35.3 Latent concept

```text
A hidden dimension discovered by SVD that captures a pattern of word-document co-occurrence.
```

中文：

```text
由 SVD 自动发现的潜在维度，表示词和文档之间的共现模式。
```

### 35.4 Singular value

```text
A value indicating the importance of a latent dimension.
```

中文：

```text
表示某个潜在概念重要性的数值。
```

### 35.5 Low-rank approximation

```text
Approximating a matrix using fewer latent dimensions.
```

中文：

```text
用更少的潜在维度近似原始矩阵。
```

### 35.6 Cosine similarity

```text
A measure of directional similarity between vectors.
```

中文：

```text
衡量两个向量方向是否相近的指标。
```

---

## 36. 典型考试题：解释 LSA 的工作流程

可以这样回答：

LSA 首先将文本集合表示为一个 **term-document matrix**，矩阵的行表示词项，列表示文档，元素通常是词频或 TF-IDF 权重。由于该矩阵通常高维、稀疏并包含噪声，LSA 使用 **SVD** 将其分解为 (A=U\Sigma V^T)。然后通过 **Truncated SVD** 只保留最大的 (r) 个奇异值及其对应的左右奇异向量，得到低秩近似 (A_r=U_r\Sigma_rV_r^T)。在这个低维空间中，文档和词可以被表示为 latent concept vectors。最后，可以使用 cosine similarity 比较文档或词之间的语义相似度，也可以用于聚类和信息检索。

---

## 37. 典型考试题：LSA 为什么可以提高文档相似度计算？

可以这样回答：

原始 BoW 或 TF-IDF 表示只关注词是否直接重叠，因此如果两篇文档使用不同但语义相关的词，它们可能被错误地认为不相似。LSA 通过 SVD 发现词和文档之间的全局共现模式，把文档投影到低维 latent semantic space 中。这样，即使两篇文档没有大量共享词，只要它们和相似的潜在概念相关，它们也会在语义空间中靠近。同时，Truncated SVD 丢弃较小奇异值对应的噪声成分，因此还能起到降维和去噪作用。

---

## 38. 典型考试题：LSA 的优缺点

### 优点

```text
1. Reduces dimensionality
2. Denoises sparse term-document matrices
3. Captures latent semantic relationships
4. Helps with synonymy
5. Enables document clustering and retrieval
6. Can be extended beyond text to visual words and multimodal data
```

### 缺点

```text
1. Polysemy is not well handled
2. Word order is ignored
3. Latent concepts are not always interpretable
4. Choice of rank r is non-trivial
5. Linear method, cannot model complex nonlinear semantics
6. Statistical assumptions do not perfectly match count data
```

---

## 39. 典型考试题：为什么 LSA 中的 concept 不一定有自然语言意义？

可以这样回答：

LSA 的 latent concepts 是通过 SVD 的数学优化自动得到的。SVD 的目标是最小化低秩近似与原始矩阵之间的 Frobenius norm reconstruction error，而不是生成可被人类命名的主题。因此，某些 concept 可能对应清晰主题，例如 graph/tree 或 user/interface；但也可能混合多个词群，难以用自然语言解释。这也是 LSA 的一个主要局限。

---

## 40. 典型考试题：LSI 如何处理 query？

可以这样回答：

LSI 首先将 query 表示成与 term-document matrix 使用同一词表的 term vector。然后利用训练好的 LSA 分解结果，将 query 投影到 latent semantic space。投影后的 query vector 与所有 document vectors 使用 cosine similarity 比较，距离最近或相似度最高的文档会被返回。这样，LSI 不仅依赖 query 和文档之间的表面词匹配，也利用了潜在语义结构。

---

## 41. 一句话总复习

**Semantic Space** 的核心是把词、文档或图像表示成向量；**LSA** 的核心是对 term-document matrix 做 **Truncated SVD**，用少量 latent concepts 近似原始高维稀疏矩阵；这样可以去噪、降维，并让语义相似的对象在几何空间中靠近；**LSI** 则把 query 投影到同一空间中，用 cosine similarity 做文档检索。

---

## 42. 最终知识框架图

```text
Raw Documents
    ↓
Tokenization / Preprocessing
    ↓
Bag of Words / TF-IDF
    ↓
Term-Document Matrix A
    ↓
SVD:
    A = U Σ V^T
    ↓
Truncated SVD:
    A ≈ U_r Σ_r V_r^T
    ↓
Latent Semantic Space
    ↓
Applications:
    - word similarity
    - document similarity
    - clustering
    - query retrieval / LSI
    - image search via bag of visual words
    - multimodal semantic spaces
```

---

## 43. 你需要真正记住的底层逻辑

不要把 LSA 机械理解成“一个降维算法”。它真正的逻辑链条是：

```text
文档语义不能直接被机器理解
        ↓
需要把文档表示成向量
        ↓
BoW / TF-IDF 得到 term-document matrix
        ↓
但这个矩阵高维、稀疏、有噪声、只看表面词重叠
        ↓
SVD 可以发现矩阵中的主要共现结构
        ↓
Truncated SVD 保留主要结构，丢弃细节和噪声
        ↓
得到低维 latent semantic space
        ↓
在这个空间中，词和文档可以用几何方式比较语义相似度
```

这就是本讲 Semantic Spaces 的完整主线。

# 习题训练
## 如何计算向量的similarity or distance
### cosine similarity
**用途：** 衡量两个向量的“方向”是否相似，不太关心长度大小。  
在 LSA / LSI 中，document vectors 或 query vectors 经常用 cosine similarity 比较。

公式：

$$
cos(x,y)=\frac{x\cdot y}{\|x\|\|y\|}​
$$

其中：

$$
x⋅y=∑​xi *​yi​​​
$$
$$
∥x∥=\sqrt{∑​x_i^2}​​
$$
也就是：

```
cosine similarity= 两个向量的点积 / 两个向量长度的乘积
```

#### 例子：sol.pdf 2.19(d)

x=(1,1,0,1,0,1)x=(1,1,0,1,0,1)x=(1,1,0,1,0,1) y=(1,1,1,0,0,1)y=(1,1,1,0,0,1)y=(1,1,1,0,0,1)

先算点积：

$$
x⋅y=1×1+1×1+0×1+1×0+0×0+1×1=3
$$
再算长度：
$$
\|x\|=\sqrt{1^2+1^2+0^2+1^2+0^2+1^2}=\sqrt{4}=2, 
\|y\|=\sqrt{1^2+1^2+1^2+0^2+0^2+1^2}=\sqrt{4}=2
$$

所以：

$$
\cos(x,y)=\frac{3}{2\times2}=\frac{3}{4}=0.75
$$
### Euclidean Distance 欧几里得距离
![[Pasted image 20260506130501.png]]![[Pasted image 20260506130517.png]]
### Correlation 相关系数 / Pearson Correlation
![[Pasted image 20260506130603.png]]![[Pasted image 20260506130617.png]]![[Pasted image 20260506130632.png]]