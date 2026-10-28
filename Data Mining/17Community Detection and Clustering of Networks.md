# COMP6237 Data Mining 最终版复习笔记

## Community Detection and Clustering of Networks

覆盖 **community detection（社区发现）**、**modularity（模块度）**、**Louvain method**、**Girvan–Newman / edge betweenness**、**Laplacian / spectral partitioning**、**modularity matrix**，并补充此前问过的 **bipartite graph（二分图）** 与 **triangle-closing（三角闭合）**。课程 slides 明确本讲的主线是：为什么网络中会有 communities，如何用 modularity 衡量 community structure，以及 Louvain、Girvan-Newman、spectral partitioning 三类方法。

---

# 0. 本讲核心一句话总结

**Community detection（社区发现）** 的目标是：  
在一个网络中找到一些 **内部连接更密集、外部连接更稀疏** 的节点组。

也就是说，一个好的 community 应该满足：

```text
community 内部：边很多
community 之间：边很少
```

例如课程中的小图可以直观分成：![[Pasted image 20260514144013.png]]

```text
Community 1: {A, B, C}
Community 2: {D, E, F, G}
```

因为：

```text
A, B, C 之间连接很多
D, E, F, G 之间连接很多
两个 group 之间主要只有 B-D 这一条桥接边
```

这正是 slides 中对自然社区的定义：**nodes that are more connected between themselves than to the rest of the network**。

---

# 1. 为什么要做 Community Detection？

## 1.1 网络中为什么会自然出现 communities？

在现实网络中，节点之间的连接通常不是随机均匀分布的，而是带有结构。例如：

- **Social network（社交网络）**：朋友之间往往有相似兴趣、相似背景、相似活动圈。
    
- **Recommender system（推荐系统）**：相似用户可能喜欢相似商品、电影、音乐。
    
- **Advertising（广告投放）**：如果识别出一群用户属于同一个兴趣社区，可以进行更精准的广告推荐。
    
- **Biological networks（生物网络）**：例如 metabolic networks（代谢网络）或 gene regulatory networks（基因调控网络），community 可能对应某种功能模块。
    
- **Information networks（信息网络）**：网页、论文、用户、话题之间也会形成主题社区。
    

录音中特别强调：在 social networks 中，connected people are often similar；找到 communities 可以 reveal groups of people with similar preferences，这对 recommender systems 和 advertising 很重要。生物网络中，community 可以帮助理解 network 的 structure and function。

---

## 1.2 Community detection 和普通 clustering 的关系

**Clustering（聚类）** 是一个更宽泛的概念：  
给定一批对象，根据相似度把它们分成若干组。

**Community detection（社区发现）** 可以看成是 network data 上的 clustering：  
对象是 nodes，关系是 edges，我们希望根据网络连接结构把 nodes 分组。

但它和普通 clustering 有一个关键区别：

普通 clustering 通常依赖一个清晰的 **distance function（距离函数）**，例如 Euclidean distance。

而网络中的节点不一定天然有欧氏坐标，所以很难直接定义合理距离。slides 特别指出，如果简单定义：

$$  
d(x,y)=0 \quad \text{if there is an edge between } x \text{ and } y  
$$

$$  
d(x,y)=1 \quad \text{if there is no edge}  
$$

这个东西并不是真正的 distance measure，因为它会违反 **triangle inequality（三角不等式）**。

---

# 2. Community 的直观定义

## 2.1 直观定义

一个 **community（社区）** 是网络中的一个节点集合，其中：

```text
内部节点之间连接较多
与外部节点之间连接较少
```

更形式化地说，如果一个节点集合 $C$ 是 community，那么我们希望：

$$  
\text{number of edges within } C  
$$

明显大于某个随机基准下 expected number of edges within $C$。

注意这里有一个非常重要的思想：

> 不能只看 community 内部边多不多，还要看“相对于随机情况是否真的多”。

这就是 **modularity（模块度）** 的动机。

---

## 2.2 课程示例图

课程 slides 中的示例网络大致可以写成：

```text
A --- B --- D --- F
 \   /      |\   /|
   C        E --- G
```

更直观地分组：

```text
左侧 community:  A, B, C
右侧 community:  D, E, F, G
桥接边: B-D
```

ASCII 图：

```text
Community 1                  Community 2

    A                              E
   / \                            / \
  B---C       ---- B-D ----      D---F
                                  \ /
                                   G
```

更接近课程逻辑的简化表示：

```text
A
|\ 
| \ 
B--C     D--E
 \       |\/|
  \      |/\|
   ------F--G
```

核心不是图的几何位置，而是：

```text
{A,B,C} 内部边多
{D,E,F,G} 内部边多
两个 group 之间边少
```

录音中明确解释：A, B, C 之间有 3 条边，D, E, F, G 之间有 5 条边，而两个 groups 之间只有一条连接边，所以它们是自然 communities。

---

# 3. 基础图论符号

后面所有公式都会用到这些符号。

## 3.1 Graph（图）

一个网络通常写作：

$$  
G=(V,E)  
$$

其中：

- $V$：**vertices / nodes（节点集合）**
    
- $E$：**edges / links（边集合）**
    

如果有一条边连接节点 $v$ 和 $w$，则：

$$  
(v,w)\in E  
$$

---

## 3.2 Adjacency matrix（邻接矩阵）

**Adjacency matrix（邻接矩阵）** 记作 $A$。

对于无权无向图：

$$  
A_{vw} =  
\begin{cases}  
1, & \text{if there is an edge between } v \text{ and } w \  
0, & \text{otherwise}  
\end{cases}  
$$

所以 $A_{vw}$ 直接表示：

```text
v 和 w 是否真的有边
```

在 modularity 公式中，$A_{vw}$ 是“真实观察到的连接”。

---

## 3.3 Degree（度）

节点 $v$ 的 **degree（度）** 记作：

$$  
k_v  
$$

它表示节点 $v$ 有多少条边。

对于无向图：

$$  
k_v = \sum_w A_{vw}  
$$

---

## 3.4 Number of edges（边数）

课程 slides 中用 $L$ 表示边的总数。

在无向图中，每条边会贡献两个 degree endpoint，所以：

$$  
\sum_v k_v = 2L  
$$

这个公式非常重要，因为 modularity 的 random reference model 会用到 $2L$。

---

## 3.5 Community label（社区标签）

假设每个节点 $v$ 被分配到某个 community。

可以用 $c_v$ 表示节点 $v$ 所属的 community。

例如：

```text
c_A = 1
c_B = 1
c_C = 1
c_D = 2
c_E = 2
c_F = 2
c_G = 2
```

---

## 3.6 Delta function（同社区指示函数）

课程 slides 中使用 $\delta(v,w)$ 来表示两个节点是否属于同一个 community。

更标准的写法是：

$$  
\delta(c_v,c_w)=  
\begin{cases}  
1, & \text{if } c_v=c_w \  
0, & \text{otherwise}  
\end{cases}  
$$

含义：

```text
如果 v 和 w 在同一个 community，就把这对节点计入 modularity。
如果不在同一个 community，就不计入。
```

---

# 4. Modularity（模块度）

## 4.1 为什么需要 modularity？

我们想判断一个 community partition 好不好。

最直观的标准是：

```text
好的划分：community 内部边多，community 之间边少
```

但问题是：

```text
“边多”到底多到什么程度才算多？
```

如果一个节点本来 degree 就很高，那么它和很多节点相连并不奇怪。  
所以不能只数真实边数，还必须和一个 **random reference model（随机参考模型）** 比较。

slides 中对 modularity 的核心解释是：

```text
Modularity Q = observed number of within-community edges
             - expected number of within-community edges at random
```

更精确地说，是对每一对节点比较：

```text
实际是否有边 - 随机情况下预期是否有边
```

然后只统计同一个 community 内的节点对。

---

## 4.3 每一项分别是什么意思？

## 4.2 Modularity 的核心公式

课程 slides 中给出：

$$  
Q = \sum_{(v,w)\in E} (A_{vw}-e_{vw})\delta(v,w)  
$$

更常见的标准写法是：

$$  
Q = \frac{1}{2L}\sum_{v,w}  
\left(  
A_{vw} - \frac{k_vk_w}{2L}  
\right)  
\delta(c_v,c_w)  
$$

不同教材可能在归一化常数上略有不同，但核心思想一样：

```text
真实连接 A_vw
减去
随机图中预期连接 k_v k_w / 2L
只对同社区节点对统计
```

---

### $A_{vw}$

$A_{vw}$ 是 observed adjacency。

$$  
A_{vw}=1  
$$

表示 $v$ 和 $w$ 实际有边。

$$  
A_{vw}=0  
$$

表示 $v$ 和 $w$ 实际没有边。

---

### $e_{vw}$ 或 $\frac{k_vk_w}{2L}$

$e_{vw}$ 是 random reference model 中 $v$ 和 $w$ 之间预期有边的概率或期望连接强度。

课程中使用的 reference model 是：

```text
random graphs with the same degree sequence
```

也就是：

```text
每个节点的 degree 保持不变，
但边的连接对象随机打乱。
```

这也叫 **configuration model（配置模型）** 的思想。

---

### $\delta(c_v,c_w)$

$$  
\delta(c_v,c_w)=1  
$$

说明 $v,w$ 在同一个 community。

$$  
\delta(c_v,c_w)=0  
$$

说明 $v,w$ 不在同一个 community。

所以 modularity 只奖励 community 内部的“超出随机预期的连接”。

---

## 4.4 为什么随机参考模型要保持 degree sequence？

这是 modularity 最容易考选择题、也最容易误解的点。

如果不用 degree sequence，而是直接和完全随机图比较，会出现问题：

```text
高 degree 节点天然更容易有很多连接。
如果不控制 degree，高 degree 节点会让 community 看起来“虚假地强”。
```

所以 reference model 要保持每个节点的 degree：

```text
节点 v 仍然有 k_v 条边
节点 w 仍然有 k_w 条边
只是这些边随机连接
```

这样比较才公平：

```text
在“每个节点连接数不变”的前提下，
如果 v 和 w 仍然比随机情况更常连接，
才说明 community structure 真的强。
```

录音中特别解释：degree sequence 就是固定每个 node 的 number of connections；然后 edges otherwise occur at random。

---

## 4.5 推导 $e_{vw}\approx \frac{k_vk_w}{2L}$

这是本讲非常核心的推导。

假设：

- 节点 $v$ 的 degree 是 $k_v$
    
- 节点 $w$ 的 degree 是 $k_w$
    
- 图中总边数是 $L$
    
- 所有 edge ends / stubs 的数量是 $2L$
    

现在考虑：

```text
从 v 出发的一条 edge stub，随机连接到 w 的概率是多少？
```

因为 $w$ 有 $k_w$ 个 edge ends，所以概率大约是：

$$  
\frac{k_w}{2L-1}  
$$

因为从 $v$ 出发的那一个 stub 本身不能再被选，所以严格说是 $2L-1$。

但大图中：

$$  
2L-1 \approx 2L  
$$

因此：

$$  
\frac{k_w}{2L-1} \approx \frac{k_w}{2L}  
$$

而 $v$ 一共有 $k_v$ 条边，所以 $v$ 和 $w$ 之间的 expected number of edges 约为：

$$  
e_{vw} \approx k_v \frac{k_w}{2L}  
$$

即：

$$  
e_{vw} \approx \frac{k_vk_w}{2L}  
$$

slides 中正是这样从 $k_w/(2L-1)\approx k_w/(2L)$ 推到 $e_{vw}\approx k_vk_w/(2L)$。

---

## 4.6 Modularity 的直觉解释

对于同一个 community 内的一对节点 $v,w$：

### 情况 1：真实有边，而且随机期望很低

$$  
A_{vw}=1,\quad e_{vw}\text{ small}  
$$

则：

$$  
A_{vw}-e_{vw}>0  
$$

说明这条边比随机预期更“意外”，因此支持 community structure。

---

### 情况 2：真实有边，但随机期望也很高

$$  
A_{vw}=1,\quad e_{vw}\text{ large}  
$$

则贡献不一定很大。

原因：

```text
高 degree 节点之间本来就容易随机连上，
所以不能过度奖励。
```

---

### 情况 3：同 community 内没有边

$$  
A_{vw}=0  
$$

则：

$$  
A_{vw}-e_{vw}=-e_{vw}<0  
$$

这会惩罚该 partition。

因为：

```text
你说这两个节点属于同一个 community，
但它们连边都没有，
而随机模型下反而预期有一定连接，
所以这个划分不好。
```

---

## 4.7 Modularity 大说明什么？

如果 $Q$ 较大：

```text
同 community 内的实际边数明显超过随机预期
```

说明 community structure 强。

如果 $Q$ 接近 0：

```text
这个 partition 和随机图差不多
```

说明社区结构不明显。

如果 $Q$ 很低甚至负：

```text
这个 partition 可能比随机还差
```

说明划分不合理。

---

# 5. 为什么不能直接用普通 Hierarchical Clustering 或 K-Means？

这一部分是本讲很重要的概念题。

## 5.1 问题根源：网络节点没有天然欧式距离

普通 clustering 方法通常需要 distance，例如：

$$  
d(x,y)  
$$

但是在 network 中，节点只是通过 edges 相连，不一定有坐标。

于是有人可能会想定义一个 naive distance：

$$  
d(x,y)=  
\begin{cases}  
0, & \text{if } x \text{ and } y \text{ are connected} \  
1, & \text{otherwise}  
\end{cases}  
$$

但 slides 直接指出：这不是 proper distance measure，因为它违反 triangle inequality。

---

## 5.2 为什么这个 naive distance 违反 triangle inequality？

**Triangle inequality（三角不等式）** 要求：

$$  
d(x,z) \le d(x,y)+d(y,z)  
$$

考虑一个简单路径：

```text
A --- B --- C
```

按照 naive distance：

$$  
d(A,B)=0  
$$

$$  
d(B,C)=0  
$$

因为都有边。

但是如果 $A$ 和 $C$ 没有直接边：

$$  
d(A,C)=1  
$$

于是：

$$  
1 \le 0+0  
$$

这显然不成立。

所以它不是 metric。

---

## 5.3 为什么 Hierarchical Clustering 会失败？

假设我们用这个 naive distance 做 hierarchical clustering。

在课程示例中，B 与 A、C、D 都有边，所以：

$$  
d(B,A)=0  
$$

$$  
d(B,C)=0  
$$

$$  
d(B,D)=0  
$$

也就是说，B 到 D 的“距离”和 B 到 A、C 的距离一样近。

同理，D 到 B、E、F、G 也都一样近。

于是 hierarchical clustering 很可能在某一步把 $B$ 和 $D$ 合并到同一个 cluster。

但直觉上 $B-D$ 是两个 communities 之间的 bridge edge，而不应该说明 B 和 D 属于同一社区。

slides 和录音都强调：因为 B 和 D 在 naive distance 下和各自 community 内节点一样近，所以大图中很容易错误合并不该合并的节点。

---

## 5.4 为什么 K-Means 更不适合？

K-Means 的问题更严重。

### 第一，K-Means 需要坐标或 centroid

K-Means 的核心是：

```text
assign points to nearest centroid
update centroid
```

但 graph nodes 没有天然 Euclidean coordinates，也没有自然 centroid。

---

### 第二，即使强行用 naive distance，也会有初始化问题

假设 $k=2$。

K-Means 通常可能随机选一个节点，再选一个距离较远的节点作为初始中心。

但在 naive distance 下：

```text
只要没有直接边，距离都是 1
```

所以很可能选到同一个 community 里的两个节点作为初始中心。

slides 中指出，即使用“随机节点 + 离它较远的节点”的策略，也仍然可能选到同一 community 的节点。

---

### 第三，即使选到 B 和 F，D 也会模糊

假设我们幸运地选了：

```text
中心1 = B
中心2 = F
```

那么：

```text
A, C 会分给 B
E, G 会分给 F
```

但是 D 怎么办？

D 和 B 有边：

$$  
d(D,B)=0  
$$

D 和 F 也有边：

$$  
d(D,F)=0  
$$

所以 D 分到 B 的 cluster 或 F 的 cluster 都一样合理。

但实际上 D 更应属于右侧 community。

因此 K-Means 很容易产生不稳定或错误结果。

---

## 5.5 总结：普通 clustering 方法的问题

```text
普通 clustering:
    需要合理距离或坐标

network community detection:
    节点之间只有连接关系
    naive distance 不是真正 metric
    容易把 bridge edge 当成强相似
    K-Means centroid 概念不自然
```

所以需要专门的 network community detection 方法。

---

# 6. Louvain Method

## 6.1 Louvain Method 的核心思想

**Louvain method（Louvain 方法）** 是一种快速的 modularity maximization heuristic。

它的目标是：

$$  
\max Q  
$$

也就是寻找一个 community partition，使 modularity 尽可能大。

但直接最大化 modularity 是一个非常困难的 combinatorial optimization problem。

slides 中明确说：直接 maximize modularity 是 difficult combinatorial optimization problem；simulated annealing 等方法可用于小网络，但大网络上太慢。Louvain 通过 heuristic 加速搜索，并反复执行两个步骤：local reassignment 和 renormalization。

---

## 6.2 为什么不能暴力搜索所有 partitions？

假设有 $n$ 个节点，要把它们分成若干 communities。

可能的 partition 数量会随 $n$ 急剧增长。

这不是简单的 $2^n$，而是 Bell number 级别增长。

所以：

```text
节点稍微多一点，所有划分数量就爆炸。
```

因此，精确搜索最大 modularity 通常不可行。

---

## 6.3 Louvain 的两阶段循环

Louvain 反复执行两个阶段：

```text
Phase 1: Local reassignment
Phase 2: Renormalization / Aggregation
```

然后在压缩后的新网络上继续做 Phase 1 和 Phase 2。

---

## 6.4 Phase 1: Local Reassignment（局部重分配）

初始状态：

```text
每个节点都是自己的 community
```

也就是：

```text
A | B | C | D | E | F | G
```

然后对每个 node $v$：

1. 看 $v$ 的 neighboring communities。
    
2. 尝试把 $v$ 移动到某个邻居所在 community。
    
3. 计算 modularity change：
    

$$  
\Delta Q  
$$

4. 如果 $\Delta Q>0$，接受这个移动。
    
5. 重复直到没有任何单点移动能提高 modularity。
    

伪代码：

```text
Initialize each node as its own community

repeat:
    improvement = false

    for each node v:
        current_community = community(v)
        candidate_communities = communities of neighbors of v

        for each community C in candidate_communities:
            compute ΔQ if v is moved to C

        move v to the community with the largest positive ΔQ
        if best ΔQ > 0:
            improvement = true

until improvement == false
```

录音中特别强调：虽然完整 modularity 计算要 sum over all edges，但 local reassignment 时只需重新考虑受该移动影响的相关边，因此比全局重算快很多。

---

## 6.5 Phase 2: Renormalization / Aggregation（重整化 / 聚合）

当 local reassignment 达到局部最优后，Louvain 会把每个 community 压缩成一个新节点。

例如 Phase 1 得到：

```text
Community 1: {A, B, C}
Community 2: {D, E, F, G}
```

那么新图变成：

```text
Node X = {A,B,C}
Node Y = {D,E,F,G}
```

原来 community 内部的边变成 **self-loop（自环）**：

```text
A-B, A-C, B-C 这些内部边
变成 X 节点上的 self-loop
```

原来 communities 之间的边变成新节点之间的 **weighted edge（加权边）**：

```text
B-D
变成 X-Y 之间的一条 weighted link
```

slides 中明确说明：communities form new nodes；old intra-community links form self-loops；inter-community links become weighted links。

---

## 6.6 为什么 renormalization 能跳出 local optimum？

Phase 1 只移动单个节点，所以容易卡在局部最优。

Phase 2 把 community 压缩成 node 后，相当于改变了搜索空间：

```text
原来移动单个 node
现在可以移动整个 community-level super-node
```

这让算法有机会发现更大尺度的 community structure。

因此 Louvain 是一种 **multi-level / hierarchical heuristic**。

---

## 6.7 Louvain 完整流程

```text
Input:
    Graph G

Initialize:
    each node is its own community

while modularity can still improve:
    Phase 1:
        repeatedly move nodes to neighboring communities
        accept move if modularity increases

    Phase 2:
        aggregate each community into a super-node
        convert internal edges into self-loops
        convert inter-community edges into weighted edges

Output:
    community partition with high modularity
```

---

## 6.8 Louvain 的优点

### 优点 1：非常快

Louvain 是本讲中最适合大规模网络的方法。

slides summary 中明确说 Louvain method 在实践中 by far the quickest and best for large networks。

---

### 优点 2：适合 large networks

比 simulated annealing、Girvan-Newman 这种方法更适合百万级节点网络。

---

### 优点 3：自然优化 modularity

它直接围绕 modularity improvement 设计，得到的 partition 通常 modularity 较高。

---

### 优点 4：产生层次结构

由于它不断 aggregate communities，因此也有 hierarchical 的意味。

---

## 6.9 Louvain 的局限

### 局限 1：heuristic，不保证全局最优

Louvain 找的是 high modularity partition，但不是 guaranteed global maximum。

---

### 局限 2：可能受节点遍历顺序影响

节点移动顺序不同，结果可能略有不同。

---

### 局限 3：modularity 本身有 resolution limit

Modularity maximization 可能会把一些小 communities 合并成大 community。

直觉上：

```text
如果一个小社区在全图规模下贡献太小，
modularity 可能认为单独拆出来不值得。
```

这不是 Louvain 独有问题，而是 modularity objective 的问题。

---

# 7. Edge Betweenness 与 Girvan–Newman Algorithm

## 7.1 Betweenness 的核心直觉

**Betweenness（介数）** 衡量一条边是否处在许多 shortest paths 上。

如果一条边连接两个 communities，那么很多从左侧 community 到右侧 community 的 shortest path 都必须经过它。

因此：

```text
高 edge betweenness 的边
很可能是 communities 之间的 bridge
而不是 community 内部边
```

slides 中定义 edge betweenness：edge $(a,b)$ 的 betweenness 是有多少 node pairs $x,y$ 的 shortest path 会经过该边；如果 shortest path 不止一条，则按经过该边的 shortest paths fraction 分配 credit。

---

## 7.2 Edge Betweenness 的正式定义

对于边 $e$，edge betweenness 可以写成：

$$  
B(e)=\sum_{s\ne t}\frac{\sigma_{st}(e)}{\sigma_{st}}  
$$

其中：

- $\sigma_{st}$：从 $s$ 到 $t$ 的 shortest paths 总数
    
- $\sigma_{st}(e)$：从 $s$ 到 $t$ 且经过边 $e$ 的 shortest paths 数量
    

如果只有一条 shortest path，并且经过 $e$：

$$  
\frac{\sigma_{st}(e)}{\sigma_{st}}=1  
$$

如果有两条 shortest paths，其中一条经过 $e$：

$$  
\frac{\sigma_{st}(e)}{\sigma_{st}}=\frac{1}{2}  
$$

这就是 slides 中说的：

```text
There might be several shortest paths.
Credit the edge with the fraction of shortest paths that go through it.
```

---

## 7.3 课程例子：为什么 $(B,D)$ 的 betweenness 很高？

课程图中：![[Pasted image 20260514180551.png]]

```text
左侧 community: {A, B, C}
右侧 community: {D, E, F, G}
桥接边: (B,D)
```

所有从左侧到右侧的 shortest paths 都必须经过 $(B,D)$。

左侧有 3 个节点：

$$  
{A,B,C}  
$$

右侧有 4 个节点：

$$  
{D,E,F,G}  
$$

所以跨社区节点对数量是：

$$  
3\times 4=12  
$$

因此：

$$  
B(B,D)=12  
$$

slides 中给出的结论正是：Betweenness of $(B,D)$ is $3*4=12$。

---

## 7.4 为什么 $(D,F)$ 的 betweenness 较低？

$(D,F)$ 是右侧 community 内部的边。

课程 slides 中给出：

```text
Shortest paths from A, B, C, D to F go through (D,F)
```

所以贡献是：

$$  
4  
$$

即：

$$  
B(D,F)=4  
$$

这明显小于：

$$  
B(B,D)=12  
$$

因此 $(B,D)$ 更像 community boundary edge。

---

## 7.5 Girvan–Newman Algorithm 的思想

**Girvan–Newman algorithm（Girvan-Newman 算法）** 是一种 divisive hierarchical clustering method。

它不是从小 cluster 合并，而是从完整图开始不断删除边。

核心步骤：

```text
1. 计算所有边的 edge betweenness
2. 删除 betweenness 最高的边
3. 重新计算所有边的 betweenness
4. 重复删除最高 betweenness 边
5. 直到达到想要的 community 数量，或没有边
6. 过程中产生 dendrogram
```
![[Pasted image 20260514190437.png|697]]

![[Pasted image 20260514190500.png]]![[Pasted image 20260514190425.png]]
slides 中给出的 algorithm 正是：calculate betweenness centrality of all edges；start removing edges beginning with highest centrality；recalculate betweenness；stop when desired number of communities remains；produces dendrogram。

---

## 7.6 为什么每删除一条边后要重新计算 betweenness？

这是考试容易问的点。

因为删除一条高 betweenness edge 后，网络中的 shortest paths 会改变。

例如：

```text
原来从左到右都走 B-D
删除 B-D 后
路径可能改走其他边
```

于是其他边的 betweenness 会突然变高。

所以不能只在开始计算一次 betweenness 然后按顺序删除。

必须：

```text
remove edge
recompute betweenness
remove next edge
recompute betweenness
...
```

---

## 7.7 Girvan–Newman 为什么能产生 dendrogram？

因为它是逐步切断网络。

每次删除 bridge-like edge 后，网络会逐渐分裂：

```text
初始：所有节点在一个网络中
删除最高 betweenness edge
分成较大的 communities
继续删除
得到更细粒度 communities
```

这就自然形成 **dendrogram（树状图）**。

课程 slides 中展示的 dendrogram 第一层切分揭示了预期 cluster，同时还能给出 more fine-grained hierarchy。

---

## 7.8 Girvan–Newman 的优点

### 优点 1：直觉非常强

```text
连接 communities 的 bridge edge
应该有高 betweenness
删除它们就能分开 communities
```

很容易理解。

---

### 优点 2：产生层次结构

它不只是给一个 partition，而是给一个 hierarchy。

---

### 优点 3：适合教学和小图分析

在小网络上可以手算或画图理解。

---

## 7.9 Girvan–Newman 的缺点

### 缺点 1：慢

每次删除边都要重新计算 betweenness。

计算 shortest paths 本身就不便宜，反复重算更慢。

slides summary 明确指出 Newman-Girvan algorithm 的问题是 slow calculations of betweenness。

---

### 缺点 2：不适合 very large networks

对于百万节点、千万边的大图，Girvan–Newman 通常不可行。

---

### 缺点 3：对 noisy bridge 可能敏感

如果网络中有很多随机跨社区边，betweenness 可能不稳定。

---

# 8. Louvain vs Girvan–Newman

|方法|核心思想|类型|优点|缺点|适用场景|
|---|---|---|---|---|---|
|**Louvain**|最大化 modularity|Agglomerative / multi-level heuristic|很快，适合大图，实际常用|不保证全局最优，受 modularity 局限影响|大规模社交网络、推荐网络|
|**Girvan–Newman**|删除 high edge betweenness 边|Divisive hierarchical method|直观，可产生 dendrogram|计算慢，不适合大图|小图、教学、需要层次结构的分析|
|**Spectral Partitioning**|用 Laplacian eigenvectors 切图|Matrix / eigenvector method|数学优雅，和图割相关|需要 group size constraint，可能不自然|图划分、supercomputing、部分 clustering|

---

# 9. Spectral Partitioning（谱划分）

## 9.1 Spectral Partitioning 想解决什么问题？

**Spectral partitioning（谱划分）** 的目标是把图分成两部分，并让两部分之间的 cut edges 尽可能少。

这类问题在 computer science 中很重要，尤其是 supercomputing：

```text
有 n 个 processes
它们之间有 m 个 dependencies
想把它们分配到不同 processors 上
希望 processor 之间通信尽量少
```

slides 中正是用这个例子引入：把有 interdependencies 的 processes 切分到 processors 上，可以看成把网络分成 parts with minimal number of edges between them。

---

## 9.2 Cut size 的定义

假设我们把节点分成两个 groups。

定义 cut size：

$$  
R = \frac{1}{2}\sum_{i,j \in \text{different groups}} A_{ij}  
$$

这里的 $\frac{1}{2}$ 是因为无向图中，$A_{ij}$ 和 $A_{ji}$ 会重复计数。

$R$ 越小，说明两个 groups 之间边越少。

---

## 9.3 用 $s_i$ 表示二分结果

对于 two-way partition，定义：

$$  
s_i =  
\begin{cases}  
+1, & \text{if node } i \text{ is in group 1}\  ;
-1, & \text{if node } i \text{ is in group 2}  
\end{cases}  
$$

那么：

### 如果 $i,j$ 在同一组

$$  
s_is_j=1  
$$

所以：

$$  
\frac{1}{2}(1-s_is_j)=0  
$$

表示这条边不属于 cut。

---

### 如果 $i,j$ 在不同组

$$  
s_is_j=-1  
$$

所以：

$$  
\frac{1}{2}(1-s_is_j)=1  
$$

表示这条边属于 cut。

因此：

$$  
\frac{1}{2}(1-s_is_j)  
$$

就是判断 $i,j$ 是否在不同组的 indicator。

---

## 9.4 Cut size 写成矩阵形式

cut size 可以写成：

$$  
R = \frac{1}{4}\sum_{i,j}(1-s_is_j)A_{ij}  
$$

因为：

- $\frac{1}{2}(1-s_is_j)$ 判断是否跨组
    
- 外面额外的 $\frac{1}{2}$ 用于修正无向图重复计数
    

展开：

$$  
R = \frac{1}{4}\sum_{i,j}A_{ij} - \frac{1}{4}\sum_{i,j}s_iA_{ij}s_j  
$$

进一步可写成：

$$  
R=\frac{1}{4}\sum_{i,j}(k_i\delta_{ij}-A_{ij})s_is_j  
$$

定义 **Laplacian matrix（拉普拉斯矩阵）**：

$$  
L = D-A  
$$

其中：

$$  
D_{ij}=k_i\delta_{ij}  
$$

所以：

$$  
L_{ij}=k_i\delta_{ij}-A_{ij}  
$$

于是：

$$  
R = \frac{1}{4}s^TLs  
$$

slides 中给出 exactly this result：$R=1/4 s^T Ls$，其中 $L$ 是 Laplacian matrix。

---

# 10. Laplacian Matrix（拉普拉斯矩阵）

## 10.1 定义

对于无向无权图：

$$  
L = D - A  
$$

其中：

- $D$ 是 degree matrix
    
- $A$ 是 adjacency matrix
    


![[Pasted image 20260515115126.png]]

这和 slides 中给出的定义一致。
![[Pasted image 20260515130656.png]]

---

## 10.2 Laplacian 的重要性质 1：行和为 0

对任意一行 $i$：

$$  
\sum_j L_{ij}

= \sum_j (k_i\delta_{ij}-A_{ij})=

 k_i-\sum_j A_{ij}=

 k_i-k_i=

0  
$$

所以：

$$  
L\mathbf{1}=0  
$$

其中：

$$  
\mathbf{1}=(1,1,\dots,1)^T  

$$
根据
![[Pasted image 20260515131735.png]]

因此，$\mathbf{1}$ 是 eigenvector，对应 eigenvalue：

$$  
\lambda_1=0  
$$

slides 中也明确指出：0 is always an eigenvalue of $L$，因为 rows and columns sum to 0，$(1,1,\ldots,1)$ 是 eigenvalue 0 的 eigenvector。

---

## 10.3 Laplacian 的重要性质 2：所有 eigenvalues 非负

Laplacian 是 positive semi-definite。

对任意向量 $x$：

$$  
x^TLx = \sum_{(i,j)\in E}(x_i-x_j)^2  
$$

有些写法会带 $\frac{1}{2}$，取决于边是否重复计数。

由于平方项非负，所以：

$$  
x^TLx\ge 0  
$$

因此所有 eigenvalues 都满足：

$$  
\lambda_i\ge 0  
$$

slides 中说 Laplacian eigenvalues can be shown to be all non-negative。

---

## 10.4 Eigenvalues 排序

通常把 Laplacian eigenvalues 排序：

$$  
\lambda_1\le \lambda_2\le \dots \le \lambda_n  
$$

其中：

$$  
\lambda_1=0  
$$

第一个非零 eigenvalue：

$$  
\lambda_2  
$$

叫做 **algebraic connectivity（代数连通度）**。

对应的 eigenvector 叫：

**Fiedler vector（Fiedler 向量）**。

slides 中明确写到：first non-zero eigenvalue $\lambda_2$ is called algebraic connectivity, corresponding eigenvector is the Fiedler vector。

---

# 11. Fiedler Vector 如何用于图划分？

## 11.1 把 $s$ 展开到 eigenvector basis

假设 $L$ 的 normalized orthogonal eigenvectors 是：

$$  
v_1,v_2,\dots,v_n  
$$

对应 eigenvalues：

$$  
\lambda_1,\lambda_2,\dots,\lambda_n  
$$

可以把 partition vector $s$ 写成：

$$  
s=\sum_{i=1}^n a_i v_i  
$$

其中：

$$  
a_i=v_i^Ts  
$$

因为每个 $s_i$ 都是 $+1$ 或 $-1$，所以：

$$  
s^Ts=n  
$$

因此：

$$  
\sum_i a_i^2=n  
$$

---

## 11.2 Cut objective 的 eigenvalue 形式

因为：

$$  
R=\frac{1}{4}s^TLs  
$$

代入：

$$  
s=\sum_i a_iv_i  
$$

得到：
$$  
R

\frac{1}{4}  
\left(\sum_i a_iv_i\right)^T  
L  
\left(\sum_j a_jv_j\right)  
$$

由于：

$$  
Lv_j=\lambda_jv_j  
$$

并且 eigenvectors 正交：

$$  
v_i^Tv_j=0 \quad i\ne j  
$$

所以：

$$  
R=\frac{1}{4}\sum_i a_i^2\lambda_i  
$$

slides 中也给出这一推导：$R=1/4\sum_i a_i^2\lambda_i$。

---

## 11.3 为什么最小化 $R$ 会得到 trivial solution？

由于：

$$  
\lambda_1=0  
$$

如果我们想让 $R$ 最小，就会把所有 weight 放到 $a_1$ 上。

而：

$$  
v_1=(1,1,\dots,1)  
$$

所以 $s$ 平行于 $v_1$，意味着：

$$  
s=(1,1,\dots,1)  
$$

也就是所有节点都在同一个 group。

这就是 trivial solution：

```text
不切图
所有节点一个社区
cut size = 0
```

它数学上最优，但对 partitioning 没有意义。

slides 中明确说：Since $\lambda_1=0$, $R$ is minimized if all weight is on $a_1$；because $v_1=(1,\dots,1)$, this means putting all nodes into group 1, which is unwanted trivial solution。

---

## 11.4 如何避免 trivial solution？

需要加入 constraint，例如：

```text
两个 group 的大小必须固定或接近平衡
```

这样不能把所有节点放进同一个 group。

一旦固定 group size，就相当于固定了 $a_1$。

剩下能优化的是：

$$  
a_2,a_3,\dots,a_n  
$$

因为：

$$  
\lambda_2  
$$

是除 0 之外最小的 eigenvalue，所以最优方向倾向于放在：

$$  
v_2  
$$

也就是 Fiedler vector。

slides 中说：group size constraint fixes $a_1$；next best solution is to put all weight on $a_2$；this implies $s$ should be parallel to $v_2$。

---

## 11.5 但 $s_i$ 只能是 $+1$ 或 $-1$

Fiedler vector 的分量是连续值：

$$  
v_i^{(2)}  
$$

但 partition vector 要求：

$$  
s_i\in{+1,-1}  
$$

所以不能直接令：

$$  
s=v_2  
$$

只能近似。

最常见做法：

$$  
s_i =  
\begin{cases}  
+1, & v_i^{(2)}\ge 0\  
-1, & v_i^{(2)}<0  
\end{cases}  
$$

也就是按照 Fiedler vector 分量的正负号划分节点。

slides 中也写：choose $s_i=+1$ if $v_i^{(2)}\ge 0$ and $s_i=-1$ otherwise。

---

## 11.6 Fiedler vector 的直觉

Fiedler vector 给每个节点一个连续 score。

```text
score 正的一边分到 group 1
score 负的一边分到 group 2
```

如果两个节点连接紧密，它们倾向于有相近的 Fiedler score。

如果一条边跨越 graph bottleneck，它连接的两个节点可能在 Fiedler score 上差异较大。

所以 Fiedler vector 可以揭示图中的“自然切割方向”。

---

# 12. Spectral Partitioning 的问题

## 12.1 它更像 graph partitioning，而不完全是 community detection

Spectral partitioning 原始目标是：

```text
minimize cut size
```

但 community detection 的目标不是单纯 minimize cut。

如果只 minimize cut，不加限制，trivial solution 是所有节点在一个 group。

如果加 group size constraint，又可能强迫图被切成类似大小的 groups。

slides 中明确指出：spectral partitioning demands fixing group sizes；这对 community detection 不理想，因为它 kind of forces a workable solution on the data。

---

## 12.2 Community 大小本来可能不平衡

真实网络中的 communities 可能大小差异很大：

```text
一个 community 有 10000 个节点
另一个只有 50 个节点
```

如果 spectral partitioning 强行平衡 group size，可能破坏真实 community structure。

---

# 13. Modularity Matrix（模块度矩阵）

## 13.1 为什么需要 modularity matrix？

为了让 spectral 方法更适合 community detection，可以不用 Laplacian cut objective，而改用 modularity objective。

回顾 modularity：

$$  
Q=\sum_{v,w}(A_{vw}-P_{vw})\delta(c_v,c_w)  
$$

其中：

$$  
P_{vw}  
$$

是 reference model 中的 expected edge。

定义 **modularity matrix（模块度矩阵）**：

$$  
B=A-P  
$$

对于 standard degree-preserving reference model：

$$  
P_{vw}=\frac{k_vk_w}{2L}  
$$

所以：

$$  
B_{vw}=A_{vw}-\frac{k_vk_w}{2L}  
$$

slides 中明确说：method can be modified for community detection by working on the modularity matrix $B=A-P$ where $A$ is adjacency matrix and $P$ a reference model。

---

## 13.2 用 modularity matrix 做 spectral community detection

对于二分情况，可以将 modularity 写成类似 quadratic form：

$$  
Q \propto s^TBs  
$$

目标变成 maximize modularity，而不是 minimize cut。

为了最大化：

$$  
s^TBs  
$$

应该看 $B$ 的 largest eigenvalue 对应的 eigenvector。

slides 中说：maximizing $Q$ implies finding the eigenvector for the largest eigenvalue of $B$，然后根据 eigenvector components 的 signs 来划分 communities。

---

## 13.3 Laplacian spectral partitioning vs modularity spectral partitioning

|项目|Laplacian spectral partitioning|Modularity spectral partitioning|
|---|---|---|
|矩阵|$L=D-A$|$B=A-P$|
|目标|minimize cut size|maximize modularity|
|eigenvector|Fiedler vector，第二小 eigenvalue|largest eigenvalue 对应 eigenvector|
|划分方式|sign of $v_2$|sign of leading eigenvector of $B$|
|问题|需要 group size constraint|更贴近 community detection|
|更适合|balanced graph partitioning|community detection|

---

# 14. Bipartite Graph（二分图）

这是之前你问过的难点之一，这里放到 network community / link prediction 的相关概念中。

## 14.1 定义

**Bipartite graph（二分图）** 是一种特殊图，它的节点可以分成两个互不重叠的集合：

$$  
U  
$$

和：

$$  
V  
$$

并且所有边都只能连接 $U$ 和 $V$ 之间的节点，不能连接同一侧内部节点。

也就是说：

```text
允许：U 中节点 -- V 中节点
不允许：U 中节点 -- U 中节点
不允许：V 中节点 -- V 中节点
```

形式化：

$$  
G=(U,V,E)  
$$

其中：

$$  
E\subseteq U\times V  
$$

---

## 14.2 图示例子：用户-电影二分图

假设：

```text
左侧是 users
右侧是 movies
边表示 user 看过 / 喜欢 movie
```

```text
Users                         Movies

 Alice  --------------------  Titanic
   | \                         Matrix
   |  \----------------------  Avatar
   |
 Bob    --------------------  Matrix
   \------------------------  Inception

 Carol  --------------------  Titanic
   \------------------------  Avatar
```

这里：

```text
User 和 User 之间没有边
Movie 和 Movie 之间没有边
只有 User-Movie 边
```

所以这是 bipartite graph。

---

## 14.3 常见 bipartite graph 例子

```text
用户 - 商品
用户 - 电影
作者 - 论文
学生 - 课程
疾病 - 症状
词语 - 文档
```

例如 recommender system 中经常使用：

$$  
G=(U,I,E)  
$$

其中：

- $U$ 是 users
    
- $I$ 是 items
    
- $E$ 是 interactions
    

---

## 14.4 Bipartite graph 和 community detection 的关系

在 bipartite graph 中，community 通常不是单侧节点形成的 dense subgraph，而是两侧节点之间形成的 dense block。

例如：

```text
一群用户都喜欢同一批电影
```

就可以形成一个 bipartite community：

```text
Community:
    Users:  {Alice, Carol}
    Movies: {Titanic, Avatar}
```

图示：

```text
        Titanic
       /       \
 Alice           Carol
       \       /
        Avatar
```

这里没有 Alice-Carol 边，也没有 Titanic-Avatar 边，但它仍然是 community，因为两侧之间连接密集。

---

## 14.5 二分图中的 projection（投影）

为了使用普通图算法，有时会把 bipartite graph 投影成一侧图。

### User projection

如果两个 users 喜欢过同一个 item，就在两个 users 之间连边：

```text
Alice -- Carol
因为她们都喜欢 Titanic 和 Avatar
```

### Item projection

如果两个 items 被同一个 user 喜欢，就在两个 items 之间连边：

```text
Titanic -- Avatar
因为 Alice 和 Carol 都喜欢它们
```

---

## 14.6 Projection 的风险

Projection 会丢失信息。

例如一个热门电影被很多人看过，会导致 user projection 里大量用户互相连接，产生虚假的 dense community。

所以对 bipartite graph 做 community detection 时，最好考虑 bipartite-specific 方法，而不是总是简单投影。

---

# 15. Triangle-Closing（三角闭合）

这是之前你问过的另一个难点，和 network link prediction、community structure 都高度相关。

## 15.1 Triangle 是什么？

在图中，如果三个节点两两相连，就形成一个 triangle（三角形）。

例如：

```text
A ----- B
 \     /
  \   /
    C
```

边集合：

$$  
(A,B),(A,C),(B,C)  
$$

---

## 15.2 Triangle-closing 是什么？

**Triangle-closing（三角闭合）** 指的是：

如果 $A$ 和 $B$ 都连接到 $C$，那么 $A$ 和 $B$ 未来也很可能连接。

也就是：

```text
A 和 B 有共同朋友 C
所以 A 和 B 未来可能成为朋友
```

未闭合三角形：

```text
A       B
 \     /
  \   /
    C
```

如果后来出现边 $A-B$：

```text
A ----- B
 \     /
  \   /
    C
```

这个过程就叫 triangle closing。

---

## 15.3 社交网络中的例子

假设：

```text
Alice 认识 Carol
Bob 认识 Carol
Alice 和 Bob 暂时不认识
```

图：

```text
Alice      Bob
   \      /
    Carol
```

因为 Alice 和 Bob 有共同朋友 Carol，所以他们更可能认识。

如果 Carol 介绍两人认识：

```text
Alice ---- Bob
   \      /
    Carol
```

这个就是 triangle closing。

---

## 15.4 Triangle-closing 和 link prediction 的关系

在 link prediction 中，一个非常基本的特征是 **common neighbors（共同邻居）**。

对于两个未连接节点 $u,v$：

$$  
\text{CommonNeighbors}(u,v)=|\Gamma(u)\cap \Gamma(v)|  
$$

其中：

- $\Gamma(u)$ 是 $u$ 的邻居集合
    
- $\Gamma(v)$ 是 $v$ 的邻居集合
    

如果 $u$ 和 $v$ 有很多共同邻居，则更可能出现未来边：

$$  
u-v  
$$

这本质上就是 triangle-closing。

---

## 15.5 Triangle-closing 和 community detection 的关系

强 community 内部通常有很多 triangles。

原因：

```text
如果一群人属于同一个社交圈
他们之间共同朋友很多
所以 triangle 很容易闭合
```

例如：

```text
A, B, C, D 是同一个朋友圈
```

可能出现：

```text
A---B
|\ /|
| X |
|/ \|
C---D
```

内部 triangles 很多，说明 community 内部连接密集。

---

## 15.6 Triangle-closing 不适用于所有网络

在某些网络中，triangle-closing 很自然，比如：

```text
社交网络
合作网络
论文引用附近的 co-author 网络
```

但在 bipartite graph 中，原始图里不会有 triangle。

因为 bipartite graph 不允许同侧连接。

例如 user-item graph：

```text
User -- Item -- User
```

如果再连 User-User 边，就破坏 bipartite 性质。

所以 bipartite graph 中不存在 odd cycle，尤其不存在 triangle。

这点很重要：

```text
普通社交图：triangle-closing 很自然
二分图：原始结构中没有 triangle
```

但投影之后会出现 triangle。例如 user projection 中，如果多个 users 共同购买 items，就可能形成 user-user triangles。

---

# 16. 本讲三个方法的统一理解

## 16.1 三种方法其实在回答不同问题

### Louvain

问的是：

```text
怎样快速找到 high modularity partition？
```

目标：

$$  
\max Q  
$$

---

### Girvan–Newman

问的是：

```text
哪些边最像 community 之间的 bridge？
```

目标：

```text
删除 high edge betweenness edges
```

---

### Spectral Partitioning

问的是：

```text
能不能用矩阵特征向量找到 graph 的自然切分方向？
```

目标：

$$  
\min s^TLs  
$$

或者 modularity version：

$$  
\max s^TBs  
$$

---

## 16.2 方法选择

```text
如果考试问 large network:
    Louvain

如果考试问 edge betweenness / dendrogram:
    Girvan-Newman

如果考试问 Laplacian / eigenvalue / Fiedler vector:
    Spectral Partitioning

如果考试问 community quality measure:
    Modularity
```

---

# 17. 最重要公式总表

## 17.1 Degree

$$  
k_v=\sum_w A_{vw}  
$$

---

## 17.2 Total degree

$$  
\sum_v k_v=2L  
$$

---

## 17.3 Modularity reference model

$$  
e_{vw}\approx \frac{k_vk_w}{2L}  
$$

---

## 17.4 Modularity

$$  
Q=\sum_{v,w}\left(A_{vw}-e_{vw}\right)\delta(c_v,c_w)  
$$

标准形式：

$$  
Q = \frac{1}{2L}\sum_{v,w}  
\left(  
A_{vw}-\frac{k_vk_w}{2L}  
\right)  
\delta(c_v,c_w)  
$$

---

## 17.5 Edge betweenness

$$  
B(e)=\sum_{s\ne t}\frac{\sigma_{st}(e)}{\sigma_{st}}  
$$

---

## 17.6 Cut size

$$  
R=\frac{1}{2}\sum_{i,j\in \text{different groups}}A_{ij}  
$$

---

## 17.7 Group membership variable

$$  
s_i=  
\begin{cases}  
+1, & i\in \text{group 1}\  
-1, & i\in \text{group 2}  
\end{cases}  
$$

---

## 17.8 Cut indicator

$$  
\frac{1}{2}(1-s_is_j)

\begin{cases}  
0, & i,j \text{ in same group}\  
1, & i,j \text{ in different groups}  
\end{cases}  
$$

---

## 17.9 Laplacian matrix

$$  
L=D-A  
$$

元素形式：

$$  
L_{ij}=k_i\delta_{ij}-A_{ij}  
$$

---

## 17.10 Cut objective as quadratic form

$$  
R=\frac{1}{4}s^TLs  
$$

---

## 17.11 Spectral decomposition of cut objective

$$  
s=\sum_{i=1}^{n}a_iv_i  
$$

$$  
a_i=v_i^Ts  
$$

$$  
R=\frac{1}{4}\sum_i a_i^2\lambda_i  
$$

---

## 17.12 Modularity matrix

$$  
B=A-P  
$$

其中：

$$  
P_{vw}=\frac{k_vk_w}{2L}  
$$

所以：

$$  
B_{vw}=A_{vw}-\frac{k_vk_w}{2L}  
$$

---

# 18. 算法伪代码总表

## 18.1 Louvain Method

```text
Input:
    Graph G = (V, E)

Initialize:
    assign each node to its own community

repeat:
    # Phase 1: local reassignment
    repeat:
        improvement = false

        for each node v in V:
            remove v temporarily from its current community

            for each neighboring community C:
                compute ΔQ if v is inserted into C

            move v to the community with maximum positive ΔQ

            if max ΔQ > 0:
                improvement = true

    until improvement == false

    # Phase 2: aggregation / renormalization
    build a new graph:
        each community becomes a super-node
        intra-community edges become self-loops
        inter-community edges become weighted edges

until modularity no longer increases

Output:
    final community partition
```

---

## 18.2 Girvan–Newman Algorithm

```text
Input:
    Graph G = (V, E)

repeat:
    compute edge betweenness for all edges

    remove edge with highest betweenness

    record current connected components as one level of dendrogram

until desired number of communities is reached
      or no edges remain

Output:
    hierarchy of communities / dendrogram
```

---

## 18.3 Spectral Partitioning using Laplacian

```text
Input:
    Graph G = (V, E)

Build:
    adjacency matrix A
    degree matrix D
    Laplacian L = D - A

Compute:
    eigenvalues and eigenvectors of L

Find:
    Fiedler vector v2
    corresponding to the second-smallest eigenvalue λ2

Partition:
    assign node i to group 1 if v2_i >= 0
    assign node i to group 2 if v2_i < 0

Output:
    two-way graph partition
```

---

## 18.4 Modularity Spectral Partitioning

```text
Input:
    Graph G = (V, E)

Build:
    adjacency matrix A
    reference matrix P
    modularity matrix B = A - P

Compute:
    leading eigenvector of B
    corresponding to the largest eigenvalue

Partition:
    assign node i based on the sign of the leading eigenvector component

Output:
    community partition approximately maximizing modularity
```

---

# 19. 易错点总结

## 易错点 1：community 不是“距离近”，而是“内部连接比随机预期更密”

错误理解：

```text
只要两个节点有边，它们就一定属于同一个 community
```

正确理解：

```text
桥接边也有边，但它连接的是两个 communities
```

例如 $(B,D)$ 有边，但它更像 bridge，不代表 B 和 D 同社区。

---

## 易错点 2：modularity 不是只数内部边

错误：

$$  
Q=\text{number of within-community edges}  
$$

正确：

$$  
Q=\text{observed within-community edges}

\text{expected within-community edges at random}  
$$

必须减去 random reference model。

---

## 易错点 3：reference model 要保持 degree sequence

错误：

```text
随机图就是任意随机连边
```

正确：

```text
随机图要保持每个节点 degree 不变
```

否则 high-degree nodes 会造成虚假的 community signal。

---

## 易错点 4：Girvan–Newman 删除的是 edge，不是 node

它删除 high **edge betweenness** 的边。

不是删除 high degree node，也不是删除 high node betweenness node。

---

## 易错点 5：Girvan–Newman 每次删除后必须重新算 betweenness

因为 shortest paths 会变化。

---

## 易错点 6：Louvain 是 heuristic，不保证全局最优

它很快、实际好用，但不是 exact optimizer。

---

## 易错点 7：Fiedler vector 是 Laplacian 的第二小 eigenvalue 对应 eigenvector

不是最大 eigenvalue。

对于 Laplacian：

```text
λ1 = 0
v1 = all-ones vector
v2 = Fiedler vector
```

---

## 易错点 8：modularity matrix 用最大 eigenvalue

对于 modularity matrix $B$：

```text
找 largest eigenvalue 对应 eigenvector
```

不要和 Laplacian 的 second-smallest eigenvalue 混淆。

---

## 易错点 9：spectral partitioning 会有 trivial solution

如果不加 group size constraint，最小 cut 是：

```text
所有节点放一个 group
```

所以需要约束。

---

## 易错点 10：bipartite graph 原始图中没有 triangle

因为 bipartite graph 没有同侧边，所以不存在三角形。

---

# 20. 考试选择题 / 简答题高频判断

## 20.1 如果题目说：哪种方法最适合 very large networks？

答案优先：

```text
Louvain method
```

原因：

```text
fast
heuristic modularity maximization
multi-level aggregation
```

slides summary 也明确说 Louvain 在实践中最快、最适合 large networks。

---

## 20.2 如果题目说：哪种方法 produces dendrogram？

答案：

```text
Girvan–Newman algorithm
```

原因：

```text
repeatedly remove high-betweenness edges
creates hierarchical splits
```

---

## 20.3 如果题目说：edge likely connecting communities has what property？

答案：

```text
high edge betweenness
```

原因：

```text
many shortest paths between communities pass through bridge edges
```

---

## 20.4 如果题目说：Laplacian matrix 的定义？

答案：

$$  
L=D-A  
$$

元素形式：

$$  
L_{ij}=k_i\delta_{ij}-A_{ij}  
$$

---

## 20.5 如果题目说：Fiedler vector 是什么？

答案：

```text
Laplacian matrix second-smallest eigenvalue corresponding eigenvector
```

即：

$$  
\lambda_2  
$$

对应 eigenvector。

---

## 20.6 如果题目说：modularity matrix 是什么？

答案：

$$  
B=A-P  
$$

其中 $P$ 是 reference model expected adjacency。

常见：

$$  
P_{ij}=\frac{k_ik_j}{2L}  
$$

---

## 20.7 如果题目说：为什么 ordinary K-Means 不适合 network community detection？

答案：

```text
K-Means needs meaningful distances and centroids.
Network nodes do not naturally have Euclidean coordinates.
A naive edge/no-edge distance violates triangle inequality.
Bridge edges can make nodes from different communities appear as close as internal nodes.
```

---

# 21. 最终知识地图

```text
Community Detection
│
├── Motivation
│   ├── social networks
│   ├── recommender systems
│   ├── advertising
│   └── biological networks
│
├── What is a community?
│   └── more edges within groups than between groups
│
├── Measuring community structure
│   └── Modularity Q
│       ├── observed edges: A_vw
│       ├── expected random edges: k_v k_w / 2L
│       ├── same-community indicator: δ(c_v,c_w)
│       └── reference model: same degree sequence
│
├── Why ordinary clustering fails
│   ├── no natural distance
│   ├── naive distance violates triangle inequality
│   ├── hierarchical clustering may merge bridge nodes
│   └── K-Means needs centroids and has ambiguous assignments
│
├── Louvain Method
│   ├── optimize modularity
│   ├── local reassignment
│   ├── renormalization
│   ├── fast for large networks
│   └── heuristic, not exact
│
├── Girvan-Newman Algorithm
│   ├── edge betweenness
│   ├── remove high-betweenness edges
│   ├── recompute betweenness after each removal
│   ├── produces dendrogram
│   └── slow for large networks
│
├── Spectral Partitioning
│   ├── minimize cut
│   ├── Laplacian L = D - A
│   ├── R = 1/4 s^T L s
│   ├── λ1 = 0, v1 = all-ones
│   ├── Fiedler vector = eigenvector of λ2
│   └── sign of Fiedler vector gives partition
│
├── Modularity Spectral Method
│   ├── modularity matrix B = A - P
│   ├── maximize Q
│   └── use leading eigenvector of B
│
└── Related Network Concepts
    ├── Bipartite graph
    │   ├── two node sets
    │   ├── edges only across sets
    │   └── no triangles in original graph
    │
    └── Triangle-closing
        ├── common neighbors predict future links
        ├── important for social networks
        └── related to dense community structure
```

---

# 22. 一句话背诵版

**Community detection** finds groups of nodes with more internal than external connections.  
**Modularity** measures whether within-community edges exceed what is expected in a random graph with the same degree sequence.  
**Louvain** greedily maximizes modularity by local node moves and community aggregation, so it is fast for large networks.  
**Girvan–Newman** removes high edge-betweenness edges to split communities and produce a dendrogram, but it is slow.  
**Spectral partitioning** uses the Laplacian $L=D-A$ and the Fiedler vector to find a graph cut, while modularity spectral methods use $B=A-P$ and the leading eigenvector to maximize modularity.