# COMP6237 Data Mining 复习笔记：Link Prediction on Networks

> 主题：**Link Prediction（链接预测）**  
> 核心问题：给定一个已经观测到的网络，能否预测哪些目前不存在或尚未观测到的链接最可能出现？课程 slides 明确把本讲分成 problem statement、pipeline、similarity-based approaches、regression approach、MLE-based hierarchical models、stochastic block models 等部分；考试重点不是死记定义，而是能够把算法应用到小网络例子上，并判断什么时候适合使用哪类方法。

---

## 1. Link Prediction 的核心直觉

**Link Prediction（链接预测）**研究的是：

$$  
\text{Given a network, can we predict the most likely links to form?}  
$$

也就是说，给你一个网络快照，例如现在的社交网络、商品购买网络、蛋白质交互网络，你要回答：

> 在当前没有边的节点对中，哪些节点对最可能在未来形成边，或者哪些边其实已经存在但由于数据不完整而没有被观测到？

因此，link prediction 可以对应两种任务：

1. **Predict future links（预测未来链接）**  
    例如 Facebook 推荐好友、Tinder 推荐匹配对象、Amazon 推荐商品。
    
2. **Predict missing links（预测缺失链接）**  
    例如生物网络中很多蛋白质之间的 interaction 还没有被实验验证，我们可以预测哪些 interaction 最值得优先做实验。
    

slides 中也明确指出，link prediction 可以用来预测未来可能出现的链接，也可以预测由于 incomplete data 而缺失的链接；同一页还给出了基本术语：$G(V,E)$、edge $(x,y)$、similarity score $s_{xy}$、adjacency matrix $A$。

---

## 2. 为什么 Link Prediction 是 Data Mining 问题？

### 2.1 商业应用：推荐系统本质上可以看成链接预测

在很多商业平台中，系统背后都可以抽象成一个 **graph/network（图/网络）**。

#### 例子 1：Facebook 好友推荐

- **Node（节点）**：用户。
    
- **Edge（边）**：两个人是好友。
    
- **Link prediction 目标**：预测哪些没有好友关系的人未来可能成为好友。
    

如果两个用户有很多共同好友，那么系统可能会认为他们之间形成 friendship link 的概率较高。

#### 例子 2：Dating platform 匹配推荐

- **Node（节点）**：平台用户。
    
- **Edge（边）**：两个人互相感兴趣、交换消息、匹配成功。
    
- **Link prediction 目标**：推荐新的 potential matches。
    

#### 例子 3：Amazon 商品推荐

这类网络通常是 **bipartite graph（二分图）**：

- 一边是 users/customers。
    
- 另一边是 items/products。
    
- 如果用户购买过某商品，就在 user 和 item 之间连边。
    
- 推荐商品就是预测新的 user-item link。
    

这也是为什么课程中特别强调 bipartite graph 不能随便使用普通的 triangle-closing 方法，因为用户和商品属于两类不同节点，网络结构机制不同。slides 用 Facebook、Tinder 和 Amazon 分别说明 proposing friendships、proposing matches、proposing items to purchase 都可以看成 link prediction。

---

### 2.2 科学应用：实验昂贵时，用预测来指导实验

在科学场景里，link prediction 的意义不是“推荐用户点击”，而是：

> 当直接观测所有 links 太贵时，用算法预测最可能存在的 links，然后优先验证这些 links。

#### 例子 1：Metabolic network / protein interaction network

- **Node**：蛋白质、分子或 biological components。
    
- **Edge**：分子之间有 chemical reaction / interaction。
    
- 实验验证 interaction 很贵。
    
- 如果所有可能节点对都做实验，成本极高。
    
- Link prediction 可以帮助研究者优先检查最可能存在的 interaction。
    

slides 中提到，收集 protein interaction 数据很昂贵，需要 experimental testing；并给出 yeast 和 human molecular interactions 大量未知的例子，说明 link prediction 可以帮助把实验集中到最可能存在的 links 上。

#### 例子 2：Food web（食物网）

- **Node**：species（物种）。
    
- **Edge**：一种物种吃另一种物种，即 biomass flow。
    
- 观测方式可能需要野外采样、解剖、检查胃内容物。
    
- 成本高、耗时长。
    
- Link prediction 可以帮助判断应该重点检查哪些 predator-prey relationship。
    

所以，在科学应用中，link prediction 解决的是 **incomplete network reconstruction（不完整网络重建）** 问题。

---

## 3. 图论基础与符号总表

### 3.1 Graph / Network

一个图记作：

$$  
G = (V,E)  
$$

其中：

- $G$：graph/network。
    
- $V$：vertex set / node set，节点集合。
    
- $E$：edge set，边集合。
    
- $|V|$：节点数量。
    
- $|E|$：边数量。
    

如果 $x,y \in V$，那么：

$$  
(x,y) \in E  
$$

表示 $x$ 和 $y$ 之间存在一条边。

---

### 3.2 Undirected graph 与 Directed graph

#### Undirected graph（无向图）

如果 $(x,y)$ 和 $(y,x)$ 没有区别，则是无向图。

例如 friendship network：

$$  
(x,y) \in E \iff (y,x) \in E  
$$

对应的 adjacency matrix 是对称矩阵：

$$  
A = A^T  
$$

#### Directed graph（有向图）

如果 $(x,y)$ 和 $(y,x)$ 有区别，则是有向图。

例如网页链接：

- 页面 $A$ 指向页面 $B$。
    
- 不代表页面 $B$ 指向页面 $A$。
    

有向图的 adjacency matrix 一般不是对称的。

---

### 3.3 Adjacency Matrix（邻接矩阵）

对于无权图，邻接矩阵 $A$ 定义为：

$$  
A_{ij} =  
\begin{cases}  
1, & \text{if there is an edge between node } i \text{ and node } j \  
0, & \text{otherwise}  
\end{cases}  
$$

如果是 weighted graph（加权图），则 $A_{ij}$ 可以表示边的权重。

例如：

$$  
A_{ij} = 3.7  
$$

可以表示节点 $i$ 和 $j$ 之间的连接强度为 $3.7$。

---

### 3.4 为什么真实网络通常不用 dense adjacency matrix 存？

真实网络经常是 **sparse network（稀疏网络）**：

$$  
|E| \ll |V|^2  
$$

也就是说，可能的边很多，但实际存在的边很少。

如果用完整 adjacency matrix 存储，矩阵中大部分元素都是 $0$，非常浪费空间。因此实际系统中更常用：

- **edge list（边列表）**
    
- **adjacency list（邻接表）**
    
- sparse matrix format
    

lecture transcript 也特别强调，真实网络通常边数相对于节点数较少，因此 adjacency matrix 大部分为空，实际存储时可以考虑 list 形式；但在算法讲解中，adjacency matrix 仍然非常方便，因为矩阵幂可以表示路径数量。

---

## 4. Link Prediction 的机器学习 Pipeline

Link prediction 的基本流程和一般 **supervised learning** 很像，但对象不是普通样本，而是 **edges（边）**。

slides 中的 pipeline 是：split graph into training/test set、choose algorithm、run algorithm on training set、evaluate on test set、check accuracy、compare with other algorithms。

---

### 4.1 Step 1：划分训练边和测试边

给定原始边集合 $E$，将其划分为：

$$  
E = E_{\text{train}} \cup E_{\text{test}}  
$$

并且通常要求：

$$  
E_{\text{train}} \cap E_{\text{test}} = \emptyset  
$$

含义：

- $E_{\text{train}}$：训练时允许算法看到的边。
    
- $E_{\text{test}}$：训练时隐藏起来，用来评价预测是否正确的边。
    

直觉上：

> 我们故意从原网络中拿掉一部分真实边，然后看算法能不能把它们排到高分位置。

---

### 4.2 Step 2：定义所有可能边的集合 $U$

对于一个无向图，如果不考虑 self-loop，则所有可能边数量是：

$$  
|U| = \frac{|V|(|V|-1)}{2}  
$$

其中：

- $U$：所有 possible edges 的集合。
    
- $|V|$：节点数。
    
- $\frac{|V|(|V|-1)}{2}$：无向图中所有 unordered node pairs 的数量。
    

如果有 $N=1000$ 个节点，则可能边数量是：

$$  
|U| = \frac{1000 \times 999}{2} = 499500  
$$

这说明 link prediction 的 candidate space 很大。即使节点数不大，可能边数也会快速增长。

---

### 4.3 Step 3：对候选边打分

算法一般不是直接输出 “yes/no”，而是为每一个候选边 $(x,y)$ 输出一个 **similarity score（相似度分数）**：

$$  
s_{xy}  
$$

通常只对训练集中没有出现的边打分：

$$  
(x,y) \in U - E_{\text{train}}  
$$

然后按照 $s_{xy}$ 从高到低排序。

分数越高，表示算法认为这条边越可能存在或未来形成。

---

### 4.4 Step 4：排序与推荐

算法输出类似：

```text
Candidate edge ranking:

1. (A, D) score = 4.7
2. (B, F) score = 3.9
3. (C, E) score = 3.2
4. (A, F) score = 1.1
...
```

然后我们可以取前 $L$ 条作为预测结果。

---

## 5. Performance Measures：如何评价 Link Prediction？

slides 中明确提到两类评价指标：

1. **Precision（关注 top L）**
    
2. **AUC（使用全部 predictions）**
    

---

### 5.1 Precision @L

如果我们取排名最高的 $L$ 条候选边，其中有 $L_r$ 条确实属于测试集 $E_{\text{test}}$，则：

$$  
\text{Precision@L} = \frac{L_r}{L}  
$$

其中：

- $L$：我们推荐/预测的边数。
    
- $L_r$：top $L$ 中真正 relevant 的边数，也就是实际在 $E_{\text{test}}$ 中的边。
    

#### 例子

假设算法推荐 top 5：

```text
1. (A,B)  true
2. (C,D)  false
3. (E,F)  true
4. (G,H)  false
5. (I,J)  true
```

其中 3 条在 test set 中真实存在，则：

$$  
\text{Precision@5} = \frac{3}{5} = 0.6  
$$

#### Precision@L 的直觉

Precision@L 适合推荐系统场景，因为真实系统通常只展示前几个推荐结果。

例如：

- Facebook 只推荐 top 10 friends。
    
- Amazon 只展示 top 20 products。
    
- 生物实验只优先验证 top 100 predicted interactions。
    

因此，top-ranked links 的质量非常重要。

---

### 5.2 AUC

AUC 的 link prediction 解释是：

> 随机选一条测试集中真实缺失边，再随机选一条真正不存在的边，算法给真实缺失边的分数更高的概率。

形式化地：

- 从 $E_{\text{test}}$ 中随机选一条边 $e^+$。
    
- 从 $U - E$ 中随机选一条不存在的边 $e^-$。
    
- 比较二者分数：
    

$$  
s(e^+) \quad \text{vs.} \quad s(e^-)  
$$

如果多次随机采样：

- $n'$：$s(e^+) > s(e^-)$ 的次数。
    
- $n''$：$s(e^+) = s(e^-)$ 的次数。
    
- $n$：总比较次数。
    

则标准写法是：

$$  
\text{AUC} = \frac{n' + 0.5n''}{n}  
$$

其中 tie 给 $0.5$ 分。

#### AUC 的直觉

AUC 不只看 top $L$，而是评价整体 ranking 能否把真实边排在不存在的边前面。

如果：

$$  
\text{AUC} = 0.5  
$$

说明算法大致等于随机排序。

如果：

$$  
\text{AUC} > 0.5  
$$

说明算法有预测能力。

如果：

$$  
\text{AUC} = 1  
$$

说明所有真实缺失边都排在所有不存在边之前，是完美排序。

---

### 5.3 为什么普通 Accuracy 不适合 Link Prediction？

因为 link prediction 中 negative class 极多。

对于 $N$ 个节点，无向图可能边数量是：

$$  
\frac{N(N-1)}{2}  
$$

而真实边通常远远少于这个数量。

例如：

- 可能边：1,000,000 条。
    
- 真实边：10,000 条。
    
- 不存在边：990,000 条。
    

如果一个傻模型永远预测 “no edge”，它的普通 accuracy 可能很高：

$$  
\text{Accuracy} = \frac{990000}{1000000} = 0.99  
$$

但它完全没有预测任何有用的新链接。

所以 link prediction 更关注：

- top-ranked predicted links 是否真实。
    
- ranking 是否把真实边排在不存在边前面。
    

---

## 6. Link Prediction Algorithms 总览

课程中主要讲两大类方法：

1. **Similarity-based techniques（基于相似度的方法）**
    
    - Local similarity indices
        
    - Global similarity indices
        
    - Quasi-local similarity indices
        
2. **MLE-based techniques（基于最大似然估计的方法）**
    
    - Hierarchical models
        
    - Stochastic block models
        

slides 把这些算法类别列为 similarity based、maximum likelihood algorithms 和 other probabilistic methods；其中 similarity based 又分 local/global/quasi-local，MLE 又包括 hierarchical 和 stochastic block models。

---

# Part I：Similarity-based Link Prediction

## 7. Similarity-based Techniques 的基本思想

**Similarity-based link prediction（基于相似度的链接预测）**的核心思想是：

> 如果两个节点在网络结构上很相似，那么它们之间更可能形成链接。

它并不直接学习复杂模型，而是定义一个分数：

$$  
s_{xy}  
$$

然后把没有边的节点对按照这个分数排序。

---

## 8. Local Similarity：局部相似度

### 8.1 Triadic Closure（三角闭合）

局部相似度方法的核心直觉是 **triadic closure（三角闭合）**：

> 如果 $x$ 和 $y$ 有共同朋友，那么 $x$ 和 $y$ 更可能认识。

在社交网络中，这非常自然：

```text
A is friend with B.
A is friend with C.
Then B and C may become friends.
```

图结构上就是：

```text
B --- A --- C
```

预测：

```text
B --- C
```

形成三角形：

```text
B
| \
|  \
A---C
```

slides 明确指出 local indices 只基于 local contact structure，主要思想是 closing triangles；更一般地说，是 counting paths of short length。

---

## 9. 基础符号：Neighbour Set 与 Degree

对于节点 $x$，定义其邻居集合：

$$  
\Gamma(x) = {z \in V : (x,z) \in E}  
$$

其中：

- $\Gamma(x)$：node $x$ 的 neighbours。
    
- $z$：与 $x$ 相连的节点。
    

节点 $x$ 的 degree（度）是：

$$  
k_x = |\Gamma(x)|  
$$

含义：

> $k_x$ 表示节点 $x$ 有多少个邻居。

---

## 10. Common Neighbours Index（共同邻居指标）

### 10.1 定义

**Common Neighbours（共同邻居）**的分数定义为：

$$  
s_{xy}^{CN} = |\Gamma(x) \cap \Gamma(y)|  
$$

含义：

> $x$ 和 $y$ 有多少个共同邻居。

如果共同邻居越多，说明二者越可能形成链接。

---

### 10.2 直觉解释

假设：

```text
x 的朋友：A, B, C
y 的朋友：B, C, D
```

那么：

$$  
\Gamma(x) = {A,B,C}  
$$

$$  
\Gamma(y) = {B,C,D}  
$$

共同邻居是：

$$  
\Gamma(x) \cap \Gamma(y) = {B,C}  
$$

所以：

$$  
s_{xy}^{CN} = 2  
$$

这表示 $x$ 和 $y$ 之间可以通过两个不同的共同邻居形成两个 triangle。

---

### 10.3 课程例子中的直觉

lecture 中用一个彩色小网络解释：

- red node 和 green node 有两个共同的 blue neighbours，所以 common neighbour score 是 2。
    
- 两个 blue nodes 也有两个共同邻居 red 和 green，所以分数也是 2。
    
- pink node 和某个 blue node 只有 green 这一个共同邻居，所以分数是 1。
    

因此，common neighbours 会优先推荐能闭合更多三角形的链接。transcript 中对此例子做了详细说明：red-green 共享两个 blue neighbours，blue-blue 共享 red 和 green，pink-blue 只共享 green。

---

### 10.4 Common Neighbours 与 adjacency matrix 的关系

对于无权无向图，邻接矩阵平方的元素：

$$  
(A^2)_{xy}  
$$

表示从 $x$ 到 $y$ 的长度为 2 的 walk 数量。

而长度为 2 的路径：

```text
x -> z -> y
```

中的中间节点 $z$ 正好是 $x$ 和 $y$ 的共同邻居。

因此：

$$  
s_{xy}^{CN} = (A^2)_{xy}  
$$

这是一个非常重要的考试点。
![[Pasted image 20260514112825.png]]

---

### 10.5 Common Neighbours 的优点

优点：

1. **简单**
    
2. **计算快**
    
3. **适合大规模网络**
    
4. **容易手算**
    
5. **考试最容易考**
    

对于一个小图，手算步骤就是：

```text
For each candidate pair (x,y):
    1. Write down Γ(x)
    2. Write down Γ(y)
    3. Compute Γ(x) ∩ Γ(y)
    4. Count its size
    5. Rank candidate links by score
```

---

### 10.6 Common Neighbours 的缺点

缺点是它不考虑节点 degree 的差异。

例如，一个超级 hub node 连接了很多人：

```text
Hub connected to 10,000 users.
```

那么很多节点都会共享这个 hub 作为 common neighbour。

但共享一个 hub 并不一定说明两个节点真的相似。

例如：

- 两个人都关注了 BBC，不代表他们是朋友。
    
- 两个人都买过 iPhone，不代表他们购物偏好高度相似。
    
- 两篇论文都引用了某篇经典论文，不代表它们主题非常接近。
    

因此 Common Neighbours 可能被 high-degree common neighbours 干扰。

---

## 11. Jaccard Index（Jaccard 指数）

### 11.1 定义

**Jaccard Index（Jaccard 相似度）**定义为：

$$  
s_{xy}^{Jaccard} =  
\frac{|\Gamma(x) \cap \Gamma(y)|}  
{|\Gamma(x) \cup \Gamma(y)|}  
$$

它不是只看共同邻居数量，而是看：

> 共同邻居占总邻居集合的比例。

---

### 11.2 直觉解释

假设：

$$  
\Gamma(x) = {A,B,C}  
$$

$$  
\Gamma(y) = {B,C,D,E}  
$$

那么：

$$  
\Gamma(x) \cap \Gamma(y) = {B,C}  
$$

$$  
|\Gamma(x) \cap \Gamma(y)| = 2  
$$

$$  
\Gamma(x) \cup \Gamma(y) = {A,B,C,D,E}  
$$

$$  
|\Gamma(x) \cup \Gamma(y)| = 5  
$$

所以：

$$  
s_{xy}^{Jaccard} = \frac{2}{5} = 0.4  
$$

---

### 11.3 Jaccard 为什么比 Common Neighbours 更细？

Common Neighbours 只看 numerator：

$$  
|\Gamma(x) \cap \Gamma(y)|  
$$

Jaccard 同时考虑 denominator：

$$  
|\Gamma(x) \cup \Gamma(y)|  
$$

这意味着：

- 两个节点共同邻居很多，但各自还有很多不同邻居，则相似度不一定高。
    
- 两个节点共同邻居数量不算特别多，但它们的邻居几乎完全重合，则相似度很高。
    

---

### 11.4 课程例子中的 Jaccard 深入解释

lecture 中提到，如果只看 common neighbours：

- red-green score = 2。
    
- blue-blue score = 2。
    

但如果使用 Jaccard：

- 对 red-green 来说，green 还有额外的 pink neighbour，所以 union 更大，Jaccard 会被拉低。
    
- 对 blue-blue 来说，它们的 neighbours 更高度重合，因此 Jaccard 更高。
    

所以 Jaccard 会给出和 Common Neighbours 不完全一样的推荐排序。

---

### 11.5 Jaccard 的优点与缺点

优点：

1. 对 degree 做了 normalization。
    
2. 更关注“邻居集合的重合比例”。
    
3. 对于不同 degree 的节点比较更公平。
    

缺点：

1. 可能低估 high-degree nodes 之间的潜在连接。
    
2. 如果两个节点本身 degree 很小，Jaccard 可能不稳定。
    
3. 对大规模图仍然需要计算 intersection 和 union。
    

---

## 12. Resource Allocation Index（资源分配指数）

### 12.1 定义

**Resource Allocation Index（资源分配指数）**定义为：

$$  
s_{xy}^{RA} =  
\sum_{z \in \Gamma(x) \cap \Gamma(y)}  
\frac{1}{k_z}  
$$

其中：

- $z$：$x$ 和 $y$ 的共同邻居。
    
- $k_z$：共同邻居 $z$ 的 degree。
    
- $\frac{1}{k_z}$：共同邻居 $z$ 对 $x,y$ 相似度的贡献。
    

---

### 12.2 直觉解释

Resource Allocation 的核心思想是：

> 共享一个 low-degree neighbour 比共享一个 high-degree hub neighbour 更有信息量。

如果一个共同邻居 $z$ 只连接了很少的人，那么 $x$ 和 $y$ 都连接到 $z$ 这件事更特殊。

如果一个共同邻居 $z$ 连接了成千上万人，那么 $x$ 和 $y$ 都连接到它并不奇怪。

slides 中也直接写到：如果我们共享 low-degree neighbours，那么更可能连接；共享一个 common hub neighbour 没那么重要。

---

### 12.3 课程例子：为什么要 discount hub？

lecture 中给了一个具体解释：

- red 和 green 的共同邻居是两个 blue nodes。
    
- 两个 blue nodes 的 degree 都是 2。
    
- 所以每个贡献 $\frac{1}{2}$。
    
- 总分：
    

$$  
s_{red,green}^{RA} = \frac{1}{2} + \frac{1}{2} = 1  
$$

再看两个 blue nodes：

- 它们的共同邻居是 red 和 green。
    
- red 的 degree 是 2，所以贡献 $\frac{1}{2}$。
    
- green 的 degree 是 3，所以贡献 $\frac{1}{3}$。
    
- 总分：
    

$$  
s_{blue,blue}^{RA} = \frac{1}{2} + \frac{1}{3} = \frac{5}{6}  
$$

因此虽然 Common Neighbours 会认为这两个候选边都共享 2 个共同邻居，但 Resource Allocation 会认为 red-green 更强，因为它们共享的共同邻居 degree 更低。transcript 明确说明 Resource Allocation 会按共同邻居的 degree 折扣，degree 高的共同邻居贡献更小。

---

### 12.4 Resource Allocation 与 Jaccard 的区别

Jaccard 的 normalization 是对目标节点 $x,y$ 的邻居集合做归一化：

$$  
\frac{|\Gamma(x) \cap \Gamma(y)|}  
{|\Gamma(x) \cup \Gamma(y)|}  
$$

Resource Allocation 的 normalization 是对共同邻居 $z$ 的 degree 做折扣：

$$  
\sum_{z \in \Gamma(x) \cap \Gamma(y)}  
\frac{1}{k_z}  
$$

区别：

|方法|主要惩罚什么？|直觉|
|---|---|---|
|Common Neighbours|不惩罚|共同邻居越多越好|
|Jaccard|惩罚目标节点邻居集合太大|共同邻居比例越高越好|
|Resource Allocation|惩罚共同邻居本身 degree 太大|共同的低度邻居更有意义|

---

## 13. 三个 Local Indices 的考试对比

### 13.1 公式总表
| 方法                  | 主要惩罚什么？            | 直觉          |
| ------------------- | ------------------ | ----------- |
| Common Neighbours   | 不惩罚                | 共同邻居越多越好    |
| Jaccard             | 惩罚目标节点邻居集合太大       | 共同邻居比例越高越好  |
| Resource Allocation | 惩罚共同邻居本身 degree 太大 | 共同的低度邻居更有意义 |

---

### 13.2 手算模板

考试中给你一个小图，让你预测下一条最可能的 link 时，直接按下面步骤：

```text
Input:
    Graph G = (V,E)
    Candidate non-edges C = U - E

For each candidate pair (x,y) in C:
    1. List Γ(x)
    2. List Γ(y)
    3. Compute common neighbours Γ(x) ∩ Γ(y)
    4. Apply required index:
        - Common Neighbours: count common neighbours
        - Jaccard: common / union
        - Resource Allocation: sum 1/degree(z)
    5. Record score s_xy

Rank candidate pairs by descending s_xy.
Highest score = most likely predicted link.
```

---

### 13.3 Tie-breaking

如果两个 candidate links 得分一样，算法本身无法区分它们。

例如：

$$  
s_{ab} = 2  
$$

$$  
s_{cd} = 2  
$$

则它们并列。

考试中应写：

> These two candidate links receive the same score under this index, so this method ranks them equally. A different index or additional information would be needed to break the tie.

---

# Part II：From Local to Global Similarity

## 14. 为什么 adjacency matrix powers 可以数路径？

这是 link prediction 中非常重要的数学机制。

对于 adjacency matrix $A$：

$$  
(A^2)_{xy}  
$$

表示从 $x$ 到 $y$ 的长度为 2 的 walks 数量。

$$  
(A^3)_{xy}  
$$

表示从 $x$ 到 $y$ 的长度为 3 的 walks 数量。

一般地：

$$  
(A^\ell)_{xy}  
$$

表示从 $x$ 到 $y$ 的长度为 $\ell$ 的 walks 数量。

lecture transcript 明确解释：长度为 2 的路径可以通过 adjacency matrix square 得到，长度为 3 的路径可以通过 adjacency matrix third power 得到；一般的 graph path counting 与 adjacency matrix powers 相关。

---

## 15. Common Neighbours 其实就是长度为 2 的路径数量

因为：

```text
x -> z -> y
```

中间节点 $z$ 是 $x$ 和 $y$ 的共同邻居。

所以：

$$  
s_{xy}^{CN} = |\Gamma(x)\cap\Gamma(y)| = (A^2)_{xy}  
$$

这说明 local similarity 其实可以被看成 path counting 的特例。

---

## 16. 为什么要考虑更长路径？

Local indices 只看很近的结构，例如长度为 2 的路径。

但两个节点即使没有共同邻居，也可能通过更长路径在同一社区中。

例如：

```text
x -- a -- b -- c -- y
```

$x$ 和 $y$ 没有共同邻居，但它们之间有长度为 4 的路径，可能说明它们处于同一局部区域。

因此可以把 link prediction 从 local 扩展到 global：

> 不只看 triangles，也看 longer loops / longer paths。

---

## 17. 长路径的问题：数量爆炸

如果直接把所有长度的路径都加起来，会出现问题：

$$  
(A^2)_{xy} + (A^3)_{xy} + (A^4)_{xy} + \cdots  
$$

因为随着路径长度增加，walk 数量通常会快速增长。

长路径太多会淹没短路径的信息。

但在 link prediction 中，短路径通常更有意义：

- 共同好友比隔着 6 个人更有意义。
    
- 共同购买模式中的短路径更可靠。
    
- 网络局部结构比非常远的连接更可信。
    

所以需要对长路径进行 **discounting（折扣）**。

---

## 18. Graph Exponential Kernel（图指数核）

### 18.1 定义

课程中给出的 global similarity 例子是 **graph exponential（图指数）**：

 $$  
\exp(\alpha A)=

I + \alpha A + \frac{\alpha^2 A^2}{2!}

- \frac{\alpha^3 A^3}{3!}
    
- \cdots  
    $$
    

因此可以定义：

![[Pasted image 20260514114028.png]]

其中：

- $A^\ell$：长度为 $\ell$ 的 walks。
    
- $\alpha$：damping factor / scaling factor。
    
- $\ell!$：factorial discount。
    
- $I$：identity matrix。
    

slides 中用 graph exponential 展示如何把更长路径纳入 global similarity，同时给更长路径更小权重。

---

### 18.2 每一项的含义

展开：

$$  
I  
$$

表示长度为 0 的 walk，即节点到自身。

$$  
\alpha A  
$$

表示长度为 1 的直接边。

$$  
\frac{\alpha^2 A^2}{2!}  
$$

表示长度为 2 的路径，也就是 common-neighbour 信息。

$$  
\frac{\alpha^3 A^3}{3!}  
$$

表示长度为 3 的路径。

一般地：

$$  
\frac{\alpha^\ell A^\ell}{\ell!}  
$$

表示长度为 $\ell$ 的路径贡献。

---

### 18.3 为什么 factorial discount 有意义？

因为：

$$  
2! = 2  
$$

$$  
3! = 6  
$$

$$  
4! = 24  
$$

$$  
5! = 120  
$$

路径越长，分母越大，贡献越小。

因此 graph exponential 允许我们利用 global structure，但不会让 long paths 完全主导结果。

lecture 中也强调，路径长度增加时路径数量会快速增长，因此需要用相应的 discount factor；graph exponential 中 factorial 自然地降低了长路径权重。

---

### 18.4 $\alpha$ 的作用

$\alpha$ 控制长路径的重要性。

如果 $\alpha$ 很小：

- 高阶项 $\alpha^\ell$ 很快变小。
    
- 主要依赖短路径。
    
- 更接近 local method。
    

如果 $\alpha$ 较大：

- 较长路径贡献增加。
    
- 更 global。
    
- 但可能引入噪声或过度平滑。
    

---

## 19. Local vs Global Similarity

|Type|Uses|Advantage|Disadvantage|
|---|---|---|---|
|Local|neighbourhood, short paths|fast, easy, scalable|ignores wider network|
|Global|many path lengths, whole graph|uses richer structure|slower, may overcount long walks|
|Quasi-local|intermediate paths|compromise|parameter choice needed|

---

## 20. Graph Kernel 的核心考试思想

课程中有一个非常重要的 **Key Tip**：

> 选择 link prediction algorithm 本质上是在假设 graph 是如何增长或形成的，也就是你选择了某种 graph kernel。

slides 明确写到：link prediction algorithm 的选择隐含了关于 graph 如何增长或已经如何增长的假设；不同图，例如 social、academic、user-item，有不同机制，因此不同算法在不同图上表现不同。

---

### 20.1 为什么算法选择等于假设网络形成机制？

如果你用 Common Neighbours，你就在假设：

> triadic closure 是主要机制。

也就是：

```text
friends of friends become friends
```

这对 social network 很合理。

但对其他网络不一定合理。

---

### 20.2 社交网络

社交网络中，triangle closing 很常见：

```text
A knows B.
A knows C.
B and C may meet through A.
```

所以 Common Neighbours、Jaccard、Resource Allocation 往往合理。

---

### 20.3 Citation network / academic network

论文引用网络是 directed，而且有时间方向：

- 未来论文引用过去论文。
    
- 过去论文不会引用未来论文。
    

普通 undirected triangle closure 不一定合适。

---

### 20.4 User-item bipartite graph

在 user-item network 中：

- 用户不会直接连用户。
    
- 商品不会直接连商品。
    
- 合法边只能是 user-product。
    

因此普通 $A^2$ 会把用户连到用户，或者商品连到商品，不适合直接作为 user-item link prediction。

这就是为什么课程专门讲 bipartite graph 要用 odd function。

---

# Part III：Bipartite Graphs

## 21. Bipartite Graph（二分图）

### 21.1 定义

**Bipartite graph（二分图）**的节点分成两个集合：

$$  
V = U \cup W  
$$

并且：

$$  
U \cap W = \emptyset  
$$

边只允许跨集合：

$$  
E \subseteq U \times W  
$$

不允许：

$$  
(u_1,u_2), \quad u_1,u_2 \in U  
$$

也不允许：

$$  
(w_1,w_2), \quad w_1,w_2 \in W  
$$

---

### 21.2 推荐系统中的二分图

在商品推荐中：

- $U$：users。
    
- $W$：items。
    
- $(u,w)$：用户 $u$ 买过或喜欢商品 $w$。
    

目标是预测新的：

$$  
(u,w)  
$$

也就是 user-item edge。

---

## 22. 为什么 bipartite graph 不能直接用 $A^2$？

在二分图中：

- 长度为 1 的路径：从 user 到 item。
    
- 长度为 2 的路径：从 user 到 user，或者从 item 到 item。
    
- 长度为 3 的路径：从 user 到 item。
    
- 长度为 4 的路径：从 user 到 user。
    

也就是说：

- odd length paths 保持在不同集合之间。
    
- even length paths 回到同一集合。
    

如果我们要预测 user-item link，就要使用 odd powers：

$$  
A^1, A^3, A^5, \cdots  
$$

而不是 even powers：

$$  
A^2, A^4, A^6, \cdots  
$$

slides 明确指出 bipartite graphs 应该使用 odd function 作为 graph kernel，例如 local 的 $A^3$ 和 global 的 $\sinh(aA)$。

---

## 23. $A^3$ 在二分图中的意义

假设：

```text
User u -> Item i -> User v -> Item j
```

这是一条长度为 3 的路径。

如果用户 $u$ 和用户 $v$ 买过相似商品，而用户 $v$ 买过 item $j$，那么可以推荐 item $j$ 给 user $u$。

因此：

$$  
(A^3)_{u,j}  
$$

可以作为 user-item link prediction 的 local score。

---

## 24. Hyperbolic Sine Kernel：$\sinh(\alpha A)$

### 24.1 定义
$$  
\sinh(\alpha A)

\alpha A  
+  
\frac{\alpha^3 A^3}{3!}  
+  
\frac{\alpha^5 A^5}{5!}  
+  
\cdots  
$$

它只包含 odd powers。

因此适合 bipartite graph 中的 cross-partition link prediction。

---

### 24.2 为什么不用 $\exp(\alpha A)$？

因为：

$$  
\exp(\alpha A)

I + \alpha A + \frac{\alpha^2 A^2}{2!}

- \frac{\alpha^3 A^3}{3!}
    
- \cdots  
    $$
    

它包含 even powers：

$$  
A^2, A^4, \cdots  
$$

这些项会把 user 连回 user，或者 item 连回 item，不适合预测 user-item links。

而：

$$  
\sinh(\alpha A)  
$$

只包含 odd powers，因此保留二分图结构。

lecture transcript 也解释了这个点：二分图中不能使用 even powers，因为它们连接同一侧节点；可以使用 $A^3$，global measure 则可以用 hyperbolic sine，因为它只包含 odd powers。

---

# Part IV：Link Prediction as Regression

## 25. 为什么可以把 Link Prediction 看成 Regression？

课程中提出一个思路：

> 给定一个 source adjacency matrix，学习一个函数，把它映射到 target adjacency matrix。

记：

- $A_{\text{source}}$：训练图的邻接矩阵。
    
- $A_{\text{target}}$：目标图的邻接矩阵。
    
- $F(A_{\text{source}})$：对 source graph 应用某个 matrix function 后得到的预测矩阵。
    

目标是让：

$$  
F(A_{\text{source}})  
$$

尽可能接近：

$$  
A_{\text{target}}  
$$

可以写成优化问题：

$$  
\min_F |F(A_{\text{source}}) - A_{\text{target}}|_F  
$$

其中：

- $|\cdot|_F$：Frobenius norm（矩阵 Frobenius 范数）。
    

slides 中把 link prediction reformulate 成 regression problem：给定 $A_{\text{source}}$ 预测 $A_{\text{target}}$，通过最小化矩阵差异；并指出无向网络的 adjacency matrix 是 real、square、symmetric，可以使用 eigendecomposition。

---

## 26. Frobenius Norm 是什么？

对于矩阵 $M$：

$$  
|M|_F =  
\sqrt{  
\sum_i \sum_j M_{ij}^2  
}  
$$

所以：

$$  
|F(A_{\text{source}})-A_{\text{target}}|_F  
$$

表示预测矩阵和目标矩阵之间所有 entry 的平方误差总和的平方根。

如果去掉平方根，也常写成：

![[Pasted image 20260514115811.png]]

这非常像 regression 的 sum of squared errors。

---

## 27. Matrix Function 如何计算？

对于无向图：

$$  
A = A^T  
$$

因此可以做 eigendecomposition：

$$  
A = U\Lambda U^T  
$$

其中：

- $U$：eigenvectors 组成的正交矩阵。
    
- $\Lambda$：eigenvalues 组成的对角矩阵。
    
- $U^T$：$U$ 的转置。
    

如果要计算某个 matrix function：

$$  
F(A)  
$$

可以定义：

$$  
F(A)=UF(\Lambda)U^T  
$$

其中：

$$  
F(\Lambda)  
$$

就是对 $\Lambda$ 对角线上的 eigenvalues 分别应用函数 $f$：

![[Pasted image 20260514120756.png]]

---

## 28. Regression 形式的优化推导

原目标：

$$  
\min_F |UF(\Lambda)U^T - A_{\text{target}}|_F  
$$

由于 $U$ 是正交矩阵，Frobenius norm 在正交变换下保持不变，因此可转化为：

$$  
\min_F |F(\Lambda) - U^T A_{\text{target}} U|_F  
$$

进一步，如果只拟合对角元素，可以写成类似：

 $$  
\min_f  
\sum_i  
\left(  
f(\lambda_i)

(U^T A_{\text{target}} U)_{ii}  
\right)^2  
$$

这就变成了一个函数拟合问题：

> 输入是 eigenvalue $\lambda_i$，输出目标是 transformed target matrix 的对应对角项。

---

## 29. Regression Approach 的问题与局限

这个方法看起来优雅，但实际并不是最常用的 link prediction 方法。

主要问题包括：

### 29.1 需要 source-target graph pairs

普通 similarity method 只需要一个 graph snapshot。

但 regression approach 通常需要：

$$  
A_{\text{source}} \rightarrow A_{\text{target}}  
$$

也就是你要有某种前后时间的网络，或者训练目标。

如果只有一个 incomplete graph，不一定有足够数据学习 $F$。

---

### 29.2 可能过拟合具体网络

如果学习到的函数 $F$ 只适合某个特定网络，它可能不能泛化到其他网络。

---

### 29.3 计算成本高

eigendecomposition 对大矩阵可能很昂贵，尤其是大型稀疏网络。

---

### 29.4 解释性不如 local indices

Common Neighbours 的解释非常直接：

> 有共同朋友，所以可能连接。

Regression function $F$ 的解释可能更抽象。

---

# Part V：MLE-based Techniques

## 30. MLE-based Link Prediction 的核心思想

**MLE-based techniques（基于最大似然估计的方法）**不只是定义一个局部 similarity score，而是先假设一个 probabilistic model of the graph。

基本思路：

1. 假设网络由某种概率模型生成。
    
2. 模型中有未知参数，例如某两类节点之间形成边的概率。
    
3. 使用 observed network，通过 **Maximum Likelihood Estimation（最大似然估计）**估计参数。
    
4. 用估计出的模型计算 missing links 的概率。
    
5. 按概率排序，预测最可能存在的 links。
    

slides 中明确说，要使用 MLE，需要先有一个具体的 probabilistic model；这些模型包含在某些条件下边存在的未知概率；然后用 MLE 找这些概率。本讲考虑 hierarchical 和 stochastic block 两类模型。

---

## 31. MLE 方法与 Similarity 方法的区别

|方面|Similarity-based|MLE-based|
|---|---|---|
|核心思想|节点结构越相似越可能连边|网络由概率模型生成|
|是否显式建模生成机制|弱|强|
|是否需要估计参数|通常不需要|需要|
|计算速度|快|慢|
|可解释结构|局部解释|全局拓扑结构|
|大规模网络|更适合|通常困难|
|考试要求|会手算 local indices|理解 principled idea 和适用条件|

---

# Part VI：Hierarchical Models

## 32. Hierarchical Model 的基本直觉

很多真实网络具有 **hierarchical organisation（层级组织结构）**。

例如：

```text
University
├── Faculty of Engineering
│   ├── ECS
│   └── Mechanical Engineering
└── Faculty of Medicine
    ├── Biology
    └── Clinical Medicine
```

同一层级下更近的节点可能更容易连接。

在网络中，这种层级结构可以用 **dendrogram（树状图）**表示。

slides 中指出，许多真实世界网络是 hierarchical 的，可以通过有 $N-1$ 个 internal nodes 的 dendrogram 来捕捉；两个真实节点的连接概率由它们的 lowest common ancestor 决定。

---

## 33. Dendrogram 中的节点含义

在 hierarchical link prediction 中：

- 叶子节点是真实网络中的 nodes。
    
- 内部节点是 latent hierarchical groups。
    
- 任意两个真实节点 $i,j$ 的连接概率由它们在 dendrogram 中的 **lowest common ancestor（最低共同祖先）**决定。
    

如果 $i$ 和 $j$ 的 lowest common ancestor 是 $r$，则：

$$  
p_{ij} = p_r  
$$

其中：

- $p_r$：internal node $r$ 对应的连接概率。
    

---

## 34. 例子：lowest common ancestor 决定连接概率

![[Pasted image 20260514131828.png]]假设 dendrogram 中：

$$  
p(1,3) = 0.3  
$$

$$  
p(3,4) = 0.4  
$$

$$  
p(4,5) = 0.8  
$$

这说明：

- nodes 4 和 5 在层级中更近，所以连接概率更高。
    
- nodes 1 和 3 在层级中较远，所以连接概率更低。
    

---

## 35. 给定 Dendrogram，如何用 MLE 估计 $p_r$？

假设给定真实网络 $G$ 和 dendrogram $D$。

对于 dendrogram 中某个 internal node $r$：

- $L_r$：$r$ 左子树中的真实节点数量。
    
- $R_r$：$r$ 右子树中的真实节点数量。
    
- $E_r$：那些 endpoints 的 lowest common ancestor 是 $r$ 的真实边数量。
    

在 $r$ 的左右子树之间，最多可能有：

$$  
L_r R_r  
$$

条边。

如果我们假设这些边独立出现，并且每条边出现概率都是 $p_r$，那么 $E_r$ 服从 binomial-like likelihood。

对 $p_r$ 做 MLE，得到：

$$  
p_r = \frac{E_r}{L_r R_r}  
$$

含义：

> 在 $r$ 的左右子树之间，实际出现的边数 / 可能出现的边数。

slides 中也给出：给定网络 $G$ 和 dendrogram $D$，令 $E_r$ 为 endpoints 以 $r$ 为最低共同祖先的边数，$L_r,R_r$ 为左右子树节点数；对于给定 $D$，可以容易确定 $p_r$。

---

## 36. Hierarchical Model 的 likelihood

对于每个 internal node $r$，其对应的可能边数是：

$$  
L_r R_r  
$$

其中实际存在：

$$  
E_r  
$$

不存在：

$$  
L_r R_r - E_r  
$$

因此 likelihood 可写成：

$$  
\mathcal{L}(D,{p_r})
=
\prod_r  
p_r^{E_r}  
(1-p_r)^{L_rR_r-E_r}  
$$

给定 $D$ 时，MLE 是：

$$  
p_r^\ast = \frac{E_r}{L_rR_r}  
$$

---

## 37. 真正困难：Dendrogram 不知道

如果 dendrogram $D$ 已知，估计 $p_r$ 很容易。

真正困难的是：

> 我们不知道真实网络对应哪个 dendrogram。

可能的 dendrogram 数量非常巨大，不可能枚举所有。

所以方法通常是：

1. 用 Monte Carlo 方法采样大量 dendrogram。
    
2. 每个 dendrogram 按 likelihood 赋权。
    
3. 对每个未连接节点对 $(i,j)$，计算它在不同 dendrogram 下的连接概率。
    
4. 对这些概率取平均：
    

$$  
\langle p_{ij} \rangle  
$$

5. 按 $\langle p_{ij} \rangle$ 排序。
    

slides 中写到下一步是按 likelihood 采样大量 dendrogram，对每个未连接节点对计算 $\langle p_{ij}\rangle$，再按 descending order 排序；困难在于要用 Monte Carlo 技术采样所有 dendrogram。

---

## 38. Hierarchical Model 的完整流程

```text
Input:
    Observed graph G = (V,E)

Goal:
    Predict missing links

Algorithm:
    1. Consider a probabilistic hierarchical model.
    2. Sample many dendrograms D with probability proportional to likelihood L(D).
    3. For each sampled dendrogram D:
        a. For each internal node r:
            estimate p_r = E_r / (L_r R_r)
        b. For each unconnected pair (i,j):
            find their lowest common ancestor r
            assign p_ij = p_r
    4. Average p_ij over sampled dendrograms:
            <p_ij> = average_D p_ij(D)
    5. Rank all unconnected pairs by <p_ij>.
    6. Highest ranked pairs are predicted missing links.
```

---

## 39. Hierarchical Model 的优点

1. **比 local similarity 更全局**
    
    - 不只看共同邻居。
        
    - 建模整个网络的 hierarchical topology。
        
2. **可以发现网络结构**
    
    - 预测 links 的同时，还能得到 hierarchical organisation。
        
3. **适合某些 biological networks**
    
    - 生物网络中经常存在模块化、层级结构。
        

---

## 40. Hierarchical Model 的缺点

1. **计算昂贵**
    
    - 需要采样大量 dendrogram。
        
    - Monte Carlo sampling 成本高。
        
2. **不适合超大网络**
    
    - lecture 中提到这类方法无法用于 millions of nodes，通常只能用于较小网络，可能最大在 $10^4$ 节点级别还需要高性能计算。
        
3. **需要模型假设**
    
    - 如果真实网络不是 hierarchical，模型可能不合适。
        

transcript 明确指出，MLE/hierarchical/block model 需要对大量 partition 或 dendrogram 做 Monte Carlo sampling，计算非常昂贵，不能用于 millions of nodes；但优势是有全局拓扑模型，可能发现超出 local triadic closure 的结构。

---

# Part VII：Stochastic Block Models

## 41. Stochastic Block Model 的基本思想

**Stochastic Block Model（随机块模型，SBM）**假设：

> 网络中的节点属于不同 group/block/community，两个节点是否连接主要取决于它们分别属于哪个 group。

例如：

- group 1：computer science students。
    
- group 2：biology students。
    
- group 3：mathematics students。
    

连接概率可能是：

```text
CS-CS: high
CS-Biology: medium
CS-Math: medium
Biology-Biology: high
Math-Math: high
Biology-Math: low
```

slides 中说明 SBM 是非常 general 的 network model；它假设网络结构主要由 group membership 决定，即两个节点相连的概率主要是它们所属 group 的函数。

---

## 42. SBM 的符号

设：

- $M$：一个 partition，即节点到 group 的分配。
    
- $\alpha,\beta$：group indices。
    
- $Q_{\alpha\beta}$：group $\alpha$ 和 group $\beta$ 之间的连接概率。
    
- $A$：observed adjacency matrix。
    

如果节点 $i$ 属于 group $\alpha$，节点 $j$ 属于 group $\beta$，那么：

$$  
P(A_{ij}=1) = Q_{\alpha\beta}  
$$

---

## 43. SBM 的 likelihood

设：

- $l_{\alpha\beta}$：group $\alpha$ 与 group $\beta$ 之间实际观测到的边数。
    
- $r_{\alpha\beta}$：group $\alpha$ 与 group $\beta$ 之间所有可能 node pairs 数量。
    

那么给定 partition $M$ 和参数 $Q$，likelihood 可以写成：

$$  
\mathcal{L}(A|M,Q)=

\prod_{\alpha \le \beta}  
Q_{\alpha\beta}^{l_{\alpha\beta}}  
(1-Q_{\alpha\beta})^{r_{\alpha\beta}-l_{\alpha\beta}}  
$$

对 $Q_{\alpha\beta}$ 做 MLE：

 $$  
Q_{\alpha\beta}=

\frac{l_{\alpha\beta}}{r_{\alpha\beta}}  
$$

含义：

> 两个 block 之间实际出现的边数 / 两个 block 之间可能出现的边数。

slides 中在 SBM 部分给出 likelihood，并解释 $l_{\alpha\beta}$ 是 groups $\alpha,\beta$ 之间的 edges，$r_{\alpha\beta}$ 是一边在 $\alpha$ 一边在 $\beta$ 的 node pairs 数；同时给出 $Q_{\alpha\beta}^{\ast}=l_{\alpha\beta}/r_{\alpha\beta}$。

---

## 44. Link Reliability：用 SBM 预测边

目标是估计某条边 $(x,y)$ 存在的 reliability：

$$  
R_{xy}=

P(A_{xy}=1|A)  
$$

如果不知道真实 partition $M$，就要对所有可能 partitions 做加权平均：
 $$  
R_{xy}=

\frac{  
\int_{\Omega}  
P(A_{xy}=1|M)  
\mathcal{L}(A|M)  
\rho(M)  
dM  
}{  
\int_{\Omega}  
\mathcal{L}(A|M)  
\rho(M)  
dM  
}  
$$

其中：

- $\Omega$：所有可能 partitions 的集合。
    
- $\rho(M)$：partition $M$ 的 prior。如果没有先验知识，可设为常数。
    
- $\mathcal{L}(A|M)$：给定 partition $M$ 时 observed graph 的 likelihood。
    

实际做法也是 sampling：

```text
Sample many partitions M.
For each M:
    estimate Q_ab.
    compute edge reliability R_xy.
Average over sampled partitions.
Rank links by R_xy.
```

---

## 45. SBM 如何识别 missing 和 spurious links？

SBM 不仅可以预测 missing links，还可以识别 **spurious links（可疑/异常链接）**。

### Missing link

如果当前没有边：

$$  
A_{xy}=0  
$$

但模型给出高 reliability：

$$  
R_{xy} \text{ is high}  
$$

说明这条边可能是 missing link。

### Spurious link

如果当前有边：

$$  
A_{xy}=1  
$$

但模型给出低 reliability：

$$  
R_{xy} \text{ is low}  
$$

说明这条边可能是 spurious link，即可能是噪声或异常连接。

slides 中明确写到，可以 rank links by reliability 来 identify missing or spurious links。

---

## 46. SBM 的优点

1. **非常 general**
    
2. **能表示 community / group structure**
    
3. **比 local similarity 更全局**
    
4. **可以发现 network blocks**
    
5. **可以预测 missing links，也可以发现 spurious links**
    

---

## 47. SBM 的缺点

1. **需要推断 group membership**
    
2. **partition 空间巨大**
    
3. **通常需要 Monte Carlo / Bayesian sampling**
    
4. **大规模网络计算困难**
    
5. **如果网络不是由 group structure 主导，模型不一定好**
    

---

# Part VIII：Similarity-based vs MLE-based Methods

## 48. 总体对比

|Dimension|Similarity-based methods|MLE-based methods|
|---|---|---|
|Main assumption|similar nodes form links|graph generated by probabilistic model|
|Typical examples|CN, Jaccard, RA, graph exponential|hierarchical model, SBM|
|Uses local structure|strong|depends|
|Uses global topology|limited for local methods; stronger for global kernels|strong|
|Parameter learning|little or none|required|
|Interpretability|very intuitive|model-based|
|Scalability|usually high|often low|
|Exam hand calculation|likely|unlikely|
|Best for|large networks, quick ranking|small/medium networks with meaningful structure|

slides 总结 MLE-based approaches 的优点是有时性能更好，并能发现 real-world networks 的额外信息；缺点是对超过约 $10^4$ nodes 的大网络通常非常慢。

---

## 49. 为什么简单方法在工业中仍然有价值？

虽然 MLE models 更 sophisticated，但在 Facebook/Amazon 这种场景，网络可能有：

$$  
10^6 \sim 10^9  
$$

级别的节点。

这时：

- 算法必须非常快。
    
- 不能对所有 partitions 或 dendrograms 做 sampling。
    
- 不能对巨大矩阵做完整 eigendecomposition。
    

因此简单的 local similarity 方法仍然很有用。

lecture 中也提到，similarity measures 不是 necessarily state of the art，但优点是对很大网络也容易计算，适合大规模场景。

---

# Part IX：Challenges and Extensions

## 50. Cold Start Problem（冷启动问题）

### 50.1 问题定义

**Cold start problem（冷启动问题）**指新节点刚进入网络时没有或几乎没有边。

例如：

- 新注册用户没有好友。
    
- 新商品还没人购买。
    
- 新论文刚发表还没人引用。
    

如果一个节点 $x$ 没有邻居：

$$  
\Gamma(x) = \emptyset  
$$

那么：

$$  
|\Gamma(x)\cap\Gamma(y)| = 0  
$$

对所有 $y$ 都成立。

所以 Common Neighbours、Jaccard、Resource Allocation 都无法有效工作。

slides 在 challenges 中明确列出 cold start problem，即 new nodes。

---

### 50.2 如何缓解 cold start？

可以引入非拓扑信息：

- user profile
    
- item features
    
- text description
    
- location
    
- time
    
- demographic information
    
- node attributes
    

例如 Amazon 新商品：

- 虽然没有购买记录。
    
- 但有 category、brand、price、description。
    
- 可以用 content-based recommendation。
    

这属于：

> augment topology-based link prediction with node information.

lecture 末尾也提到，除了 topology-based information，还可以加入 node information，这是 open research direction。

---

## 51. Negative Links / Signed Networks

有些网络中 links 不只是 positive。

例如：

- friendship vs hostility。
    
- trust vs distrust。
    
- like vs dislike。
    
- support vs oppose。
    

这种网络叫：

**signed network（符号网络）**。

普通 link prediction 通常只预测：

$$  
A_{xy}=1  
$$

但 signed link prediction 需要预测：

$$  
A_{xy} \in {+1, -1, 0}  
$$

也就是：

- 正边
    
- 负边
    
- 无边
    

slides 的 challenges 也提到 link prediction with links of a different nature，例如 negative links。

---

## 52. Temporal Networks（时间网络）

真实网络会随时间变化：

- links can be created。
    
- links can be destroyed。
    
- nodes can enter。
    
- nodes can leave。
    

如果我们只有一个静态 snapshot，就丢失了时间信息。

更一般地，网络可以写成：

$$  
G(t) = (V(t),E(t))  
$$

Link prediction 目标是：

$$  
E(t+\Delta t)  
$$

中的新边。

slides challenges 中也明确提到 temporal networks：links can be created and destroyed over time，new nodes come in and out of system。

---

## 53. Temporal Link Prediction 的难点

静态 link prediction 只问：

> 哪些当前没有边的节点对可能有边？

时间 link prediction 还要问：

> 什么时候形成边？边会持续多久？旧边是否会消失？

例如社交平台：

- 两个人今天互关。
    
- 三个月后取消关注。
    

如果只用静态图，无法表达这种变化。

---

# Part X：考试重点与答题策略

## 54. 课程对考试能力的要求

lecture 开头明确说，网络相关 lectures 会进入 assessment；对于某些主题，需要能够 apply algorithms，而不是只 memorise knowledge；也需要判断 algorithms 在什么情况下 appropriate。

因此 link prediction 的考试重点大概率是：

1. 给小图，让你手算 local similarity scores。
    
2. 让你解释 pipeline。
    
3. 让你比较 precision 和 AUC。
    
4. 让你说明不同方法适合什么网络。
    
5. 让你解释 MLE-based models 的 principle，而不是手算复杂 sampling。
    
6. 让你解释 bipartite graph 为什么用 odd powers。
    

---

## 55. 最容易考的手算：Common Neighbours / Jaccard / RA

### 55.1 标准答题格式

如果题目要求对某个候选边 $(x,y)$ 计算分数，应写：

```text
For node x:
Γ(x) = {...}

For node y:
Γ(y) = {...}

Common neighbours:
Γ(x) ∩ Γ(y) = {...}

Union:
Γ(x) ∪ Γ(y) = {...}

Degrees of common neighbours:
k_z = ...

Therefore:
CN = ...
Jaccard = ...
RA = ...
```

---

### 55.2 示例

假设：

$$  
\Gamma(x)={a,b,c}  
$$

$$  
\Gamma(y)={b,c,d,e}  
$$

共同邻居：

$$  
\Gamma(x)\cap\Gamma(y)={b,c}  
$$

并集：

$$  
\Gamma(x)\cup\Gamma(y)={a,b,c,d,e}  
$$

如果：

$$  
k_b=2,\quad k_c=5  
$$

则：

Common Neighbours：

$$  
s_{xy}^{CN}=2  
$$

Jaccard：

$$  
s_{xy}^{Jaccard}=\frac{2}{5}  
$$

Resource Allocation：

$$  
s_{xy}^{RA}=\frac{1}{2}+\frac{1}{5}=0.7  
$$

---

## 56. 如果题目问：为什么 shared hub 不重要？

答：

> A high-degree common neighbour is less informative because it connects to many nodes. Two nodes sharing such a hub may happen by chance. Resource Allocation discounts the contribution of a common neighbour $z$ by $\frac{1}{k_z}$, so low-degree common neighbours contribute more than hubs.

中文解释：

> 一个高度数 hub 本来就连接很多节点，所以两个节点共同连接到它并不稀奇；相反，如果两个节点共享一个低度数邻居，这说明它们在局部结构上更接近。因此 RA 用 $\frac{1}{k_z}$ 对共同邻居贡献打折。

---

## 57. 如果题目问：为什么 graph exponential 是 global？

答：

> Because $\exp(\alpha A)$ includes powers $A^2,A^3,A^4,\ldots$, and $(A^\ell)_{xy}$ counts walks of length $\ell$ between $x$ and $y$. Therefore it uses paths of many lengths, not just length-2 paths. Longer paths are discounted by $\frac{\alpha^\ell}{\ell!}$.

中文解释：

> 因为 graph exponential 不只包含 $A^2$，还包含更高阶的 $A^3,A^4,\ldots$，所以它利用了更大范围的网络结构；同时通过 $\frac{\alpha^\ell}{\ell!}$ 给长路径更小权重，避免长路径数量爆炸。

---

## 58. 如果题目问：为什么 bipartite graph 用 odd powers？

答：

> In a bipartite graph, edges only connect nodes from different partitions. A walk of even length starts and ends in the same partition, while a walk of odd length starts and ends in different partitions. Since user-item link prediction predicts links across the two partitions, the kernel should contain odd powers such as $A^3$ or $\sinh(\alpha A)$.

中文解释：

> 二分图中合法边只能跨越两个节点集合。偶数长度路径会回到同一侧，不能直接表示 user-item candidate link；奇数长度路径才从 user 到 item。因此要用 $A^3$ 或只包含奇数阶的 $\sinh(\alpha A)$。

---

## 59. 如果题目问：Similarity-based 和 MLE-based 方法区别？

答题模板：

```text
Similarity-based methods assign a score to each candidate edge based on structural similarity, such as common neighbours, Jaccard index, or resource allocation. They are simple, fast, and scalable, and are especially useful for large networks. However, they make relatively local assumptions such as triadic closure and may miss global structure.

MLE-based methods assume a probabilistic generative model for the network, such as a hierarchical model or stochastic block model. They estimate model parameters by maximum likelihood and then rank missing links by estimated probability or reliability. These methods can capture global network organisation and may perform better in some cases, but they are computationally expensive and usually unsuitable for very large networks.
```

---

## 60. 如果题目问：Precision 和 AUC 区别？

答：

```text
Precision@L evaluates only the top L ranked predictions. It is useful when only a limited number of recommendations or experiments can be made. If L_r of the top L predicted links are true test links, then Precision@L = L_r/L.

AUC evaluates the whole ranking. It is the probability that a randomly chosen missing link from the test set receives a higher score than a randomly chosen nonexistent link. It is less focused on the very top recommendations but gives a global measure of ranking quality.
```

---

# Part XI：完整符号表

|Symbol|Meaning|
|---|---|
|$G$|graph/network|
|$V$|set of vertices/nodes|
|$E$|set of observed edges|
|$x,y,z$|nodes|
|$(x,y)$|edge or candidate edge between $x$ and $y$|
|$A$|adjacency matrix|
|$A_{ij}$|adjacency matrix entry for nodes $i,j$|
|$U$|set of all possible edges|
|$E_{\text{train}}$|training edge set|
|$E_{\text{test}}$|test edge set|
|$s_{xy}$|similarity score for candidate edge $(x,y)$|
|$\Gamma(x)$|neighbour set of node $x$|
|$k_x$|degree of node $x$, $k_x=|
|$s_{xy}^{CN}$|Common Neighbours score|
|$s_{xy}^{Jaccard}$|Jaccard score|
|$s_{xy}^{RA}$|Resource Allocation score|
|$(A^\ell)_{xy}$|number of length-$\ell$ walks from $x$ to $y$|
|$\alpha$|damping/scaling factor in graph kernels|
|$I$|identity matrix|
|$\exp(\alpha A)$|graph exponential kernel|
|$\sinh(\alpha A)$|odd-power kernel suitable for bipartite graphs|
|$A_{\text{source}}$|source/training adjacency matrix in regression view|
|$A_{\text{target}}$|target adjacency matrix|
|$F(A)$|matrix function applied to adjacency matrix|
|$\|\cdot\|_F$|Frobenius norm|
|$U\Lambda U^T$|eigendecomposition of symmetric adjacency matrix|
|$\lambda_i$|eigenvalue|
|$D$|dendrogram in hierarchical model|
|$r$|internal node of dendrogram|
|$p_r$|connection probability associated with internal node $r$|
|$L_r$|number of real nodes in left subtree of $r$|
|$R_r$|number of real nodes in right subtree of $r$|
|$E_r$|number of observed edges whose endpoints have $r$ as lowest common ancestor|
|$M$|partition of nodes into groups in SBM|
|$\Omega$|set of all possible partitions|
|$Q_{\alpha\beta}$|probability of edge between group $\alpha$ and group $\beta$|
|$l_{\alpha\beta}$|observed number of edges between group $\alpha$ and $\beta$|
|$r_{\alpha\beta}$|possible number of node pairs between group $\alpha$ and $\beta$|
|$R_{xy}$|reliability/probability of link $(x,y)$ under SBM|
|$\rho(M)$|prior probability over partitions|
|$L$|number of top predicted links considered|
|$L_r$|number of relevant links among top $L$; note this symbol can also mean left-subtree size in hierarchical models, so context matters|
|$n'$|number of AUC comparisons where missing link score is higher|
|$n''$|number of AUC comparisons with tied scores|
|$n$|total number of AUC comparisons|

---

# Part XII：最终压缩版记忆框架

## 61. 一句话总结 Link Prediction

**Link Prediction（链接预测）**就是：给定网络快照 $G=(V,E)$，对当前没有边的节点对 $(x,y)\in U-E_{\text{train}}$ 计算分数 $s_{xy}$，并把最可能存在或未来形成的边排在前面。

---

## 62. 一句话总结 Local Similarity

Local similarity 的核心是假设：

> nodes with many meaningful common neighbours are likely to connect.

对应三大公式：

$$  
s_{xy}^{CN}=|\Gamma(x)\cap\Gamma(y)|  
$$

$$  
s_{xy}^{Jaccard}=

\frac{|\Gamma(x)\cap\Gamma(y)|}  
{|\Gamma(x)\cup\Gamma(y)|}  
$$

$$  
s_{xy}^{RA}=

\sum_{z\in\Gamma(x)\cap\Gamma(y)}  
\frac{1}{k_z}  
$$

---

## 63. 一句话总结 Global Similarity

Global similarity 把 link prediction 看成 path counting：

$$  
(A^\ell)_{xy}  
$$

表示长度为 $\ell$ 的 walks 数量。

Graph exponential 把不同长度路径加权相加：

$$  
s_{xy}

= [\exp(\alpha A)]_{xy}

\sum_{\ell=0}^{\infty}  
\frac{\alpha^\ell}{\ell!}  
(A^\ell)_{xy}  
$$

---

## 64. 一句话总结 Bipartite Graph

二分图预测跨集合边，所以要用 odd powers：

$$  
A^3  
$$

或者：

$$  
\sinh(\alpha A)

\alpha A  
+  
\frac{\alpha^3A^3}{3!}  
+  
\frac{\alpha^5A^5}{5!}  
+\cdots  
$$

---

## 65. 一句话总结 MLE-based Models

MLE-based methods 假设网络由概率模型生成，然后用 observed graph 最大化 likelihood，估计边出现概率，再预测 missing links。

Hierarchical model：

$$  
p_r = \frac{E_r}{L_rR_r}  
$$

SBM：

$$  
Q_{\alpha\beta}

\frac{l_{\alpha\beta}}{r_{\alpha\beta}}  
$$

---

## 66. 最重要的考试判断

如果题目给小网络让你算：

> 用 Common Neighbours / Jaccard / Resource Allocation 手算 ranking。

如果题目问应用场景：

> social network 可用 triadic closure；user-item bipartite graph 要用 odd powers；biological network 可用 link prediction 指导昂贵实验。

如果题目问方法优劣：

> similarity-based 快、简单、可扩展；MLE-based 更全局、更有模型解释，但慢，不适合超大网络。

如果题目问评价：

> Precision@L 看 top recommendations；AUC 看整体 ranking。

下面把我们这段对话里提到过的 **Link Prediction（链接预测）方法**统一整理成一份对比表。你可以把它理解成考试时答 **“which method should be used and why?”** 的复习框架。

---

# 1. Link Prediction 方法总览

我们提到的方法可以分成四大类：

```text
Link Prediction Methods
│
├── 1. Local similarity methods
│   ├── Common Neighbours
│   ├── Jaccard Similarity
│   └── Resource Allocation
│
├── 2. Global / path-based methods
│   ├── Graph Exponential: exp(aA)
│   ├── Bipartite local kernel: A^3
│   ├── Bipartite global kernel: sinh(aA)
│   └── Wrong bipartite kernel example: cosh(aA)
│
├── 3. Regression-based method
│   └── Learn a function F(A_source) ≈ A_target
│
└── 4. MLE-based probabilistic methods
    ├── Hierarchical Models
    └── Stochastic Block Models
```

核心区别是：

- **Local similarity methods**：看局部邻居结构，快，适合手算和大图。
    
- **Global/path-based methods**：看更长路径，利用更大范围网络结构。
    
- **Regression-based methods**：把 link prediction 转化成矩阵预测问题。
    
- **MLE-based methods**：假设网络由某种概率模型生成，然后估计链接概率。
    

---

# 2. Common Neighbours

## 方法思想

**Common Neighbours（共同邻居）**认为：

> 两个节点共同邻居越多，它们越可能形成链接。

公式：

```text
CN(x, y) = |Γ(x) ∩ Γ(y)|
```

其中：

```text
Γ(x) = 节点 x 的邻居集合
```

例如：

```text
      A
     / \
    B   C
```

B 和 C 共同邻居是 A，所以预测：

```text
      A
     / \
    B---C
```

这就是 **triangle-closing（三角闭合）**。

## Pros

|优点|解释|
|---|---|
|简单直观|“朋友的朋友可能成为朋友”|
|非常容易手算|只需要数共同邻居|
|计算速度快|只看局部邻域|
|适合社交网络|社交网络中 triangle-closing 很常见|
|适合考试题|最容易出现在小图计算题里|

## Cons

|缺点|解释|
|---|---|
|不考虑节点度数|high-degree hub 会造成误导|
|不归一化|只看共同邻居数量，不看邻居集合大小|
|只看局部结构|忽略更远的网络路径|
|对 bipartite graph 不一定合适|普通 triangle-closing 会产生同组连接|
|容易给 hub 附近节点高分|共享大 hub 不一定说明真正相似|

---

# 3. Jaccard Similarity

## 方法思想

**Jaccard Similarity** 不只看共同邻居数量，还看共同邻居占总邻居的比例。

公式：

```text
Jaccard(x, y) = |Γ(x) ∩ Γ(y)| / |Γ(x) ∪ Γ(y)|
```

也就是：

```text
共同邻居数量 / 两个节点所有邻居的总集合大小
```

在我们做过的例子中：

```text
Γ(1) = {2,4}
Γ(3) = {2,4}
```

所以：

```text
Jaccard(1,3) = 2/2 = 1
```

而：

```text
Γ(2) = {1,3}
Γ(4) = {1,3,5,6}
```

共同邻居虽然有两个：

```text
{1,3}
```

但总邻居集合更大：

```text
{1,3,5,6}
```

所以：

```text
Jaccard(2,4) = 2/4 = 0.5
```

## Pros

|优点|解释|
|---|---|
|比 Common Neighbours 更公平|会考虑总邻居集合大小|
|能处理 degree 差异|不只是数共同邻居|
|适合比较邻居集合相似性|如果两个节点邻居几乎一样，分数高|
|结果更细致|可以避免把大邻居集合节点过度高估|

## Cons

|缺点|解释|
|---|---|
|可能高估低度节点|如果两个低度节点只有一个共同邻居，Jaccard 也可能等于 1|
|可能低估 hub 附近的真实链接|因为 union 太大，分母变大|
|仍然只看局部结构|不考虑长路径|
|遇到 tie 仍需额外规则|例如 `(1,3)` 和 `(5,6)` 都可能 Jaccard = 1|

## 和 Common Neighbours 的核心区别

```text
Common Neighbours:
只问：共同邻居有几个？

Jaccard:
问：共同邻居占总邻居的比例是多少？
```

所以在我们之前的例子中：

```text
Common Neighbours 预测：(1,3) 和 (2,4)

Jaccard 预测：(1,3) 和 (5,6)
```

---

# 4. Resource Allocation

## 方法思想

**Resource Allocation（资源分配指数）**认为：

> 共享低度数邻居比共享高度数 hub 更有意义。

公式：

```text
RA(x, y) = Σ 1/k_z
```

其中：

```text
z ∈ Γ(x) ∩ Γ(y)
k_z = 共同邻居 z 的 degree
```

也就是：

```text
每个共同邻居的贡献 = 1 / 这个共同邻居的度数
```

如果共同邻居是 hub，degree 很大：

```text
1/k_z 很小
```

它贡献就小。

如果共同邻居只连接很少节点：

```text
1/k_z 较大
```

它贡献就大。

## Pros

|优点|解释|
|---|---|
|能削弱 hub 的影响|high-degree common neighbour 不会贡献太多|
|比 Common Neighbours 更细致|不同共同邻居权重不同|
|适合社交网络/信息网络|共享小圈子朋友比共享大明星节点更有意义|
|仍然比较容易计算|只需要知道共同邻居的 degree|

## Cons

|缺点|解释|
|---|---|
|仍然只看局部结构|不考虑长路径|
|需要计算共同邻居 degree|比 Common Neighbours 稍复杂|
|对某些网络假设不一定成立|有些网络里 hub 可能确实很重要|
|不适合直接处理 bipartite 推荐边|仍需注意跨组结构|

## 与前两个方法的对比

|方法|关注点|
|---|---|
|Common Neighbours|共同邻居数量|
|Jaccard|共同邻居占总邻居比例|
|Resource Allocation|共同邻居本身是否是 hub|

---

# 5. Graph Exponential：`exp(aA)`

## 方法思想

Local methods 只看短路径，尤其是长度为 2 的路径。

但是 **Graph Exponential** 会把不同长度的路径都考虑进去：

```text
exp(aA) = I + aA + a^2A^2/2! + a^3A^3/3! + ...
```

其中：

```text
A^k 表示长度为 k 的路径数量
```

所以：

```text
A^2 = 长度为 2 的路径
A^3 = 长度为 3 的路径
A^4 = 长度为 4 的路径
```

`exp(aA)` 的思想是：

> 所有路径都提供信息，但长路径权重要小一些。

因为有：

```text
a^k / k!
```

路径越长，通常权重越小。

## Pros

|优点|解释|
|---|---|
|利用 global structure|不只看共同邻居|
|能考虑更长路径|捕捉间接关系|
|数学形式优雅|用矩阵函数统一表示|
|长路径有折扣|避免长路径数量完全主导结果|

## Cons

|缺点|解释|
|---|---|
|计算成本高|需要矩阵函数或高阶矩阵运算|
|参数 `a` 需要选择|damping factor 会影响结果|
|可能引入噪声|很长路径未必有真实意义|
|不一定适合 bipartite graph|因为包含偶数和奇数路径，可能产生同组连接问题|
|不适合手算复杂大图|考试中更可能考概念而不是完整计算|

---

# 6. `A^3` for Bipartite Graph

## 方法思想

在 **bipartite graph（二分图）**中，节点分成两类：

```text
Users        Items
```

合法边只能是：

```text
User -- Item
```

不能有：

```text
User -- User
Item -- Item
```

在二分图中：

```text
长度 1：User -> Item
长度 2：User -> Item -> User
长度 3：User -> Item -> User -> Item
```

所以如果要推荐商品，应该预测：

```text
User -> Item
```

也就是跨组连接。

长度为 3 的路径正好从 User 回到 Item：

```text
U1 -> I1 -> U2 -> I2
```

这表示：

```text
U1 买过 I1
U2 也买过 I1
U2 还买过 I2
所以可以推荐 I2 给 U1
```

## Pros

|优点|解释|
|---|---|
|适合 bipartite graph|保持 user-item 跨组连接|
|比普通 triangle-closing 更合理|不会直接推荐 User-User 边|
|直觉清楚|“相似用户喜欢的商品也推荐给我”|
|计算比全局方法简单|只看长度 3 的路径|
|适合推荐系统|用户—商品网络非常典型|

## Cons

|缺点|解释|
|---|---|
|只看固定长度路径|只考虑长度 3，不考虑更长路径|
|仍然比较局部|信息范围有限|
|可能受热门商品影响|热门商品会连接很多用户|
|需要 bipartite 假设|不适合普通单类型社交网络|
|可能需要归一化|否则高活跃用户/热门商品影响大|

---

# 7. `sinh(aA)` for Bipartite Graph

## 方法思想

`sinh(aA)` 是 bipartite graph 中更合适的 global kernel。

公式：

```text
sinh(aA) = aA + a^3A^3/3! + a^5A^5/5! + ...
```

它只包含奇数次幂：

```text
A, A^3, A^5, ...
```

在二分图中，奇数长度路径一定是跨组的：

```text
User -> Item
User -> Item -> User -> Item
User -> Item -> User -> Item -> User -> Item
```

所以它适合预测：

```text
User -- Item
```

## Pros

|优点|解释|
|---|---|
|非常适合 bipartite graph|只使用奇数路径，保持跨组连接|
|比 `A^3` 更 global|同时考虑长度 1、3、5 等路径|
|长路径有折扣|用 `a^k/k!` 降低长路径权重|
|适合推荐系统|可以综合多层间接关系|
|避免同组预测|不会像偶数路径那样主要得到 User-User 或 Item-Item|

## Cons

|缺点|解释|
|---|---|
|计算比 `A^3` 更复杂|涉及矩阵函数|
|参数 `a` 需要调节|太大可能让长路径影响过强|
|仍可能受热门节点影响|热门 item/user 会产生很多路径|
|解释性不如 `A^3`|多阶路径混合后不如单一路径直观|
|大规模图上成本较高|工业级推荐系统要考虑近似计算|

---

# 8. `cosh(aA)`：不适合 Bipartite Recommendation 的例子

## 方法思想

`cosh(aA)` 的展开是：

```text
cosh(aA) = I + a^2A^2/2! + a^4A^4/4! + ...
```

它只包含偶数次幂：

```text
I, A^2, A^4, A^6, ...
```

在二分图中，偶数长度路径会回到同一组：

```text
User -> Item -> User
Item -> User -> Item
```

所以 `cosh(aA)` 主要得到的是：

```text
User-User similarity
Item-Item similarity
```

而不是：

```text
User-Item recommendation
```

## Pros

|优点|解释|
|---|---|
|可以衡量同组相似性|例如 user-user similarity 或 item-item similarity|
|数学上有意义|偶数路径确实能表达同侧节点相似|
|可用于辅助分析|可以先算用户相似度，再做其他推荐步骤|

## Cons

|缺点|解释|
|---|---|
|不适合直接做 user-item link prediction|因为推荐边应该跨组|
|会产生同组连接|User-User 或 Item-Item，不是合法推荐边|
|不能直接输出商品推荐|分数位置不对应 user-item candidate links|
|和任务目标不匹配|目标是预测 User-Item link，但它关注 same-side links|
|考试中应判断为 not sensible|这是典型概念陷阱|

一句话：

```text
Bipartite recommendation 要奇数路径；
cosh(aA) 是偶数路径；
所以不适合直接推荐 user-item links。
```

---

# 9. Regression-based Link Prediction

## 方法思想

把 link prediction 看成矩阵预测问题。

给定：

```text
A_source = 当前/训练网络的邻接矩阵
A_target = 未来/目标网络的邻接矩阵
```

希望学习一个函数：

```text
F(A_source) ≈ A_target
```

目标可以写成：

```text
min ||F(A_source) - A_target||
```

如果网络是无向图，邻接矩阵 $A$ 是对称矩阵，可以做 eigendecomposition：

```text
A = UΛU^T
```

然后：

```text
F(A) = UF(Λ)U^T
```

这样可以把矩阵函数学习转成 regression-like problem。

## Pros

|优点|解释|
|---|---|
|思路系统|把 link prediction 转成监督学习/回归问题|
|可以学习 graph kernel|不必手工指定固定函数|
|数学上优雅|利用矩阵分解和函数拟合|
|适合有时间序列网络的情况|有 source graph 和 target graph 时更自然|
|可以适配特定网络|学到适合该网络演化的函数|

## Cons

|缺点|解释|
|---|---|
|需要 target graph|不是所有任务都有未来网络作为训练目标|
|计算成本高|需要矩阵分解|
|可能过拟合|学到的函数可能只适合某个网络|
|解释性较弱|不如 common neighbours 直观|
|大规模网络困难|完整 eigendecomposition 很贵|
|考试中不太可能手算|更可能考概念理解|

---

# 10. Hierarchical Model

## 方法思想

**Hierarchical Model（层级模型）**假设：

> 真实网络有层级结构，层级上越接近的节点越可能连接。

用 **dendrogram（树状图）**表示隐藏的层级结构。

两个节点的连接概率由它们的 **lowest common ancestor（最低共同祖先）**决定。

如果两个节点在树上很近：

```text
连接概率高
```

如果在树上很远：

```text
连接概率低
```

给定某个 internal node $r$：

```text
p_r = E_r / (L_r R_r)
```

其中：

```text
E_r = 左右子树之间实际存在的边数
L_r R_r = 左右子树之间所有可能边数
```

然后用 MLE 估计这些概率。

## Pros

|优点|解释|
|---|---|
|能捕捉全局层级结构|不只看局部邻居|
|适合 hierarchical networks|例如生物网络、组织网络|
|可以发现隐藏结构|预测 link 的同时获得层级解释|
|概率解释明确|输出可以解释为连接概率|
|有时比 similarity methods 更准|如果网络确实有层级结构|

## Cons

|缺点|解释|
|---|---|
|计算非常慢|需要采样大量 dendrograms|
|不适合大规模网络|节点很多时几乎不可行|
|模型假设强|如果网络没有层级结构，效果不好|
|实现复杂|比 local indices 难很多|
|不适合考试手算|主要考概念、优缺点、适用场景|
|需要 Monte Carlo sampling|搜索所有 dendrograms 不现实|

---

# 11. Stochastic Block Model, SBM

## 方法思想

**Stochastic Block Model（随机块模型）**假设：

> 节点属于不同 groups/blocks，两个节点连接概率主要取决于它们所属的 group。

例如：

```text
CS students    Biology students    Math students
```

连接概率由 group pair 决定：

```text
Q_ab = group a 和 group b 之间的连接概率
```

如果节点 $i$ 属于 group $a$，节点 $j$ 属于 group $b$，那么：

```text
P(edge i-j exists) = Q_ab
```

MLE 估计：

```text
Q_ab = l_ab / r_ab
```

其中：

```text
l_ab = group a 和 group b 之间实际边数
r_ab = group a 和 group b 之间可能边数
```

## Pros

|优点|解释|
|---|---|
|能捕捉 community/block structure|适合社区结构明显的网络|
|概率解释清楚|输出 link reliability|
|可以预测 missing links|没有边但概率高，可能是缺失边|
|可以识别 spurious links|有边但概率低，可能是异常边|
|比 local methods 更全局|看整体 group-level pattern|
|很 general|可以表达不同 group 之间不同连接模式|

## Cons

|缺点|解释|
|---|---|
|需要推断 group membership|节点属于哪个 block 未必已知|
|partition 空间巨大|所有分组方式无法穷举|
|计算复杂|通常需要 sampling 或近似推断|
|不适合超大网络|大规模图上成本高|
|模型假设较强|如果网络不是 block-driven，效果不一定好|
|解释粒度较粗|group-level 可能忽略个体差异|

---

# 12. 方法横向总对比

|方法|类型|主要依据|适合场景|最大优点|最大缺点|
|---|---|---|---|---|---|
|Common Neighbours|Local similarity|共同邻居数量|社交网络、小图手算|简单快|不考虑 degree/hub|
|Jaccard|Local similarity|共同邻居比例|邻居集合相似性|有归一化|可能高估低度节点|
|Resource Allocation|Local similarity|低度共同邻居更重要|hub 较多的社交/信息网络|抑制 hub|仍然只看局部|
|`exp(aA)`|Global path-based|所有路径加权|普通图，全局结构有意义|利用长路径|计算贵，不适合二分推荐直接用|
|`A^3`|Bipartite local|长度 3 跨组路径|用户—商品推荐|保持跨组预测|只看固定长度路径|
|`sinh(aA)`|Bipartite global|奇数路径加权|二分图推荐|适合 user-item link prediction|计算较复杂|
|`cosh(aA)`|Even-path kernel|偶数路径|同组相似性分析|可算 user-user/item-item 相似|不适合直接 user-item 推荐|
|Regression method|Learning-based|学习 $F(A)$|有 source-target 图|可以学习 kernel|需要目标图，计算贵|
|Hierarchical Model|MLE-based|层级结构|层级网络、生物网络|捕捉 hidden hierarchy|采样慢|
|SBM|MLE-based|block/group structure|社区明显网络|概率建模强|推断复杂、慢|

---

# 13. 考试中如何选择方法？

## 如果是普通社交网络

优先考虑：

```text
Common Neighbours
Jaccard
Resource Allocation
```

原因：

```text
社交网络中 triangle-closing 很自然。
朋友的朋友可能成为朋友。
```

如果题目强调 hub 很多：

```text
Resource Allocation 更合适
```

如果题目强调公平比较邻居集合：

```text
Jaccard 更合适
```

---

## 如果是用户—商品推荐网络

这是 bipartite graph。

不要直接用普通 triangle-closing。

更合适：

```text
A^3
sinh(aA)
```

不合适：

```text
cosh(aA)
普通 A^2 triangle-closing
```

原因：

```text
推荐边必须是 User-Item。
奇数路径保持跨组。
偶数路径回到同组。
```

---

## 如果网络有明显社区结构

可以考虑：

```text
Stochastic Block Model
```

例如：

```text
学生社交网络
科研合作网络
组织内部网络
```

如果节点属于不同 group，而且 group pair 决定连接概率，SBM 很合理。

---

## 如果网络有明显层级结构

可以考虑：

```text
Hierarchical Model
```

例如：

```text
生物网络
组织结构网络
分类层级网络
```

---

## 如果有时间序列图

可以考虑：

```text
Regression-based link prediction
```

例如：

```text
A_source = 今年网络
A_target = 明年网络
```

可以学习：

```text
F(A_source) -> A_target
```

---

# 14. 最重要的记忆版总结

```text
Common Neighbours:
数共同邻居，简单但不归一化。

Jaccard:
共同邻居 / 总邻居，更公平，但可能高估低度节点。

Resource Allocation:
共同邻居按 1/degree 打折，能降低 hub 影响。

exp(aA):
考虑所有长度路径，global，但计算更贵，也不专门适配二分图。

A^3:
二分图中的局部推荐路径，User -> Item -> User -> Item。

sinh(aA):
二分图中的 global 推荐 kernel，只含奇数路径，适合跨组预测。

cosh(aA):
只含偶数路径，会得到 User-User 或 Item-Item，不适合直接推荐商品。

Regression:
学习矩阵函数 F(A)，适合有 source-target 网络，但计算贵。

Hierarchical Model:
假设网络有层级结构，用 MLE 估计连接概率，强但慢。

SBM:
假设节点分 group，连接概率由 group pair 决定，适合社区结构网络，但推断复杂。
```

一句话总括：

> **Local methods 快、简单、适合考试手算；global/path methods 利用更多网络结构；bipartite graph 要特别用奇数路径；MLE-based methods 更 principled 但更慢；选择方法本质上是在选择你认为网络是如何形成的。**