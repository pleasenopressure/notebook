	
## 1. 核心思想：启发式搜索 (Heuristic Search)

启发式搜索是一种**有信息搜索 (Informed Search)** 策略，它利用问题领域的特定知识来引导搜索过程，从而比无信息搜索（如 BFS, DFS, IDS）更加高效。

- **核心机制**: 使用一个**评估函数 f(n) (Evaluation Function)** 来估计每个节点的“期望代价”或“价值” 。
    
- **搜索策略**: 算法总是优先扩展（expand）评估函数值最优的节点 。
    
- **巨大优势**: 即使是一个很弱的启发式，也通常比盲目的无信息搜索要好得多 [transcript 23:43, 25:33]。教授强调，无信息搜索因为没有方向指引，会进行大量不必要的探索，效率极低 [transcript 24:05-24:15]。
#### **贪婪最佳优先搜索 (Greedy Best-First Search):**

- **公式：** $f(n) = h(n)$，其中 $h(n)$ 是从节点 $n$ 到目标的**估计**成本 10。
    
- **例子：** 罗马尼亚地图问题中，使用“直线距离”作为 $h(n)$ 。
    
- **缺陷：** 它只看终点方向，不顾已走过的路程，可能被困在死胡同或找不到最优解（不是最优的）
#### 1. 时间复杂度 (Time Complexity)

- **计算结果：** **$O(b^m)$** 
    
- **计算逻辑：**
    
    - **最坏情况 (Worst Case):** 贪婪搜索完全依赖启发式函数 $h(n)$ 来导航。如果启发式函数设计得很差（或者提供了误导性信息），算法可能会被引导进入一条无限深或者非常深的死胡同，或者在树中反复横跳。
        
    - 在这种情况下，它实际上可能需要遍历整个搜索空间才能发现“哎呀，这里走不通”，或者找到解。
        
    - 因此，其最坏情况下的行为类似于深度优先搜索 (DFS) 遍历整棵树，复杂度由树的最大深度 $m$ 决定，即 $b \times b \times \dots \times b$ ($m$ 次) = $b^m$。
        
- **实际情况：** 虽然理论上最坏情况很糟，但老师也提到，如果有一个**好的启发式函数**，效率会得到“巨大的提升 (dramatic improvement)” 2，实际搜索的节点数会远小于 $b^m$。
    

#### 2. 空间复杂度 (Space Complexity)

- **计算结果：** **$O(b^m)$** 
    
- **计算逻辑：**
    
    - **需要保留所有节点：** 与 DFS 可以“丢弃”已访问分支不同，Greedy Best-First Search（作为 Best-First Search 的一种特例）通常需要将所有已生成但未扩展的节点保留在 **Fringe (边缘集/队列)** 中，以便根据 $h(n)$ 值随时跳转到更有希望的节点 4。
        
    - **最坏情况：** 在最坏情况下（例如启发式函数没有区分度，或者引导算法像 BFS 一样铺开），内存中需要存储的节点数量会随着搜索深度指数级增长。
        
    - 这也意味着它和 BFS 一样，面临严重的**内存爆炸**风险。
- Complete? No – can get stuck in loops like depth first if heuristic is bad
- Optimal? No


## 2. 重点算法：A* 搜索 (A* Search)

A* 算法是启发式搜索中最著名且应用最广泛的算法之一，其目标是找到从起点到终点的**最优路径（最短路径）**。
![[557aa0ea-7f85-4cdc-b069-e08692d7ccb6.png]]
A* 空间复杂度极高，他需要存储生成的所有节点 O(b^d)
A*的性能取决于启发函数的质量。理论上可以比IDS好。
### 评估函数：f(n)=g(n)+h(n)

这是 A* 算法的灵魂，它由两部分构成：

- **g(n)**: 从**起始节点**到当前节点 n 的**真实路径代价** (the cost so far to reach n)。这部分是已知的、确定的。
    
- **h(n)**: 从当前节点 n 到**目标节点**的**估计代价** (estimated cost to goal)。这部分是未知的，通过**启发式函数 (Heuristic Function)** 来估计。
    

> **注意**: A* 的目标是最小化**整个路径的长度**，而贪婪最佳优先搜索 (Greedy best-first search) 仅仅是最小化到目标的估计距离 h(n) [transcript 1:00-1:17]。A* 的方法在逻辑上更加完备。

### 关键执行规则

- **目标检查时机**: 必须在节点被**从 fringe (待扩展列表) 中选择出来进行扩展时**，才检查它是否是目标状态。不能在节点刚生成时就检查，否则可能因为提前发现一个次优路径上的目标而终止，导致无法找到最优解 [transcript 2:11-3:04]。

### 练习描述A* 算法：
The student can apply A* **algorithm** .the evalution function f(n) = g(n) + h(n), where g(n) is the actual distance from start node to node n, h(n) is the expected distance from node n to goal. I can choose Manhattan distance as heuristic function.
### 描述A*是optimal的
A* is guaranteed optimal because Manhattan distance heuristic is **admissible** / **consistent**, which means h(n) will never **overestimate** the true cost to reach the goal. Since the Manhattan distance **represent** the **minimum** number of moves that a **tile** must make, it must less than or equal to the true number of moves required. **Therefore** A* is optimal.
### 证明tree search中 heuristic Is admissible 就能说明optimal

    Assume that there is a suboptimal goal solution G2 in the fringe,The actual cost g(G2) must more than optimal cost C*.
    In the meantime ,there must exist an unexpanded node n in the fringe which is in the optimal path.
    Since heuristic is **admissible**, there is f(n) = g(n)+h(n)<C*. So there is no reason that algorithm will choose other node before try to expand node n.

## 3. A* 算法的最优性理论 (Optimality of A*)

A* 算法能否保证找到最优解，完全取决于启发式函数 h(n) 的质量。

### 可采纳启发式 (Admissible Heuristic)

- **定义**: 一个启发式函数是**可采纳的**，如果它**永远不会高估 (never overestimates)** 从当前节点到目标的实际最小成本。即，对于任意节点 n，满足 h(n)≤h∗(n)，其中 h∗(n) 是真实的最优路径成本。
    
- **本质**: 它是一个“乐观的”估计。例如，地图上两点间的直线距离就是一个可采纳启发式，因为实际路程只会更长或相等。
    
- **最优性定理**: 如果 h(n) 是可采纳的，那么使用**树搜索 (TREE-SEARCH)** 的 A* 算法**保证是最优的**。
    

### 最优性证明的直观理解

教授在课程中给出了一个非常直观的证明逻辑 [transcript 8:56]：

1. 假设 fringe 中有一个**次优的目标解 G2​**。它的实际成本 g(G2​) 必然大于最优成本 C∗。因为 h(G2​)=0，所以其评估值为 f(G2​)=g(G2​)>C∗。
    
2. 同时，在通往**最优解**的路径上，一定存在某个还未被扩展的节点 n。
    
3. 因为 h(n) 是可采纳的（h(n)≤h∗(n)），所以该节点的评估值 f(n)=g(n)+h(n)≤g(n)+h∗(n)=C∗。
    
4. **核心结论**: f(n)≤C∗<f(G2​)。这意味着，A* 算法总会优先扩展最优路径上的节点 n，而**永远不会**在找到最优解之前去选择那个次优的目标解 G2​。
    

> 对于**图搜索 (GRAPH-SEARCH)**，需要一个更强的条件——**一致性启发式 (Consistent Heuristic)** 才能保证最优性。
> ![[Pasted image 20260109171853.png]]

## 4. 启发式函数的设计 (Heuristic Design)

一个好的启发式是算法效率的关键。

### 案例分析：八数码问题 (8-Puzzle)

- **启发式函数设计**:
    
    - h1​(n) = **错位棋子数 (Number of misplaced tiles)** 。
        
    - h2​(n) = **曼哈顿距离总和 (Total Manhattan distance)** 。
        
- **支配性 (Dominance)**: 如果对于任意节点 n，都有 h2​(n)≥h1​(n)，那么我们称 h2​ **支配** h1​。一个支配性更强的启发式通常更接近真实成本，因此更有效。
    

### 为什么 h2​ 的有效分支因子更低？

- **信息粒度**: h1​ 的信息非常粗糙，它只能判断棋子“在”或“不在”正确位置 [transcript 23:00-23:17]。而 h2​ 的信息更精细，它还能衡量棋子离目标位置的**远近** [transcript 17:44-17:54]。
    
- **剪枝效果**: 因为 h2​ 提供了更准确的成本估计，它能让 A* 算法更早地识别出那些没有前途的搜索路径（它们的 f 值会变得很高），从而进行更有效的剪枝。这减少了大量不必要的节点扩展。
    
- **结果**: 更少的节点扩展意味着搜索树更“稀疏”，搜索过程更“聚焦”，因此**有效分支因子 (Effective Branching Factor)** 自然就更低，搜索成本也随之降低。
    

### 通用设计方法：松弛问题 (Relaxed Problem)

这是创造可采纳启发式的一个强大技巧。

- **核心思想**: 通过**移除或放宽原问题的一些约束**来创建一个更简单的新问题（即松弛问题） [transcript 29:57-30:03]。
    
- **定理**: **松弛问题的最优解成本，是原问题的一个可采纳的启发式函数**。
    
- **例子**:
    
    - **8-Puzzle**:
        
        - 规则放宽为“棋子可瞬移到任意位置”，最优步数即为 h1​。
            
        - 规则放宽为“棋子可移动到任意相邻格子，无视阻挡”，最优步数即为 h2​。
            
    - **地图寻路**:
        
        - 规则放宽为“可以不沿道路行驶，直接穿越田野”，最优路径即为**直线距离** [transcript 29:16-29:21]。
            

## 5. 另一类搜索范式：局部搜索 (Local Search)

- **适用场景**: 当我们**只关心最终解的状态，而不关心到达解的路径**时使用，例如优化问题和约束满足问题 (CSP) [transcript 32:40-33:16]。
    
- **核心思想**: 从一个完整的配置（状态）开始，不断地对其进行小的、局部的修改，以期逐步改善解的质量 [transcript 33:40-33:46]。
    
- **代表算法：爬山法 (Hill Climbing)**: 检查所有邻近的状态，并选择能带来最大改进的方向移动 [transcript 35:19]。
    
- **主要缺陷：局部最优 (Local Optima)**: 算法很容易陷入一个比所有邻居都好，但却不是全局最好的解的状态，从而卡住无法继续前进 [transcript 36:52, 38:29]。
    

## 6. 核心概念与术语总结

- **Heuristic Search (启发式搜索)**: 利用问题特定知识引导搜索的有信息搜索策略。
    
- **A* Search (A* 算法)**: 一种寻找最优路径的启发式搜索算法，使用评估函数 f(n)=g(n)+h(n)。
    
- **Admissible Heuristic (可采纳启发式)**: 永远不会高估到目标真实成本的启发式函数，是 A* 算法最优性的保证。
    
- **Search Cost (搜索成本)**: 通常用算法扩展的节点总数来衡量。
    
- **Effective Branching Factor (有效分支因子)**: 衡量搜索树实际“浓密”程度的指标，越低越好。
    
- **Manhattan Distance (曼哈顿距离)**: 在网格中两点之间水平和垂直距离之和，是 8-Puzzle 问题的一个优秀启发式。
    
- **Relaxed Problem (松弛问题)**: 通过移除原问题约束得到的简化版问题，用于系统性地创建可采纳启发式。
    
- **Local Search (局部搜索)**: 一类只关心最终解状态的优化算法，通过迭代改进当前状态来求解。
    
- **Hill Climbing (爬山法)**: 一种简单的局部搜索算法。
    
- **Local Optima (局部最优)**: 局部搜索算法可能陷入的一个非全局最优的解。