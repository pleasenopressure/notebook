说明：以下笔记基于当前对话中可见内容、你上传的 **18_outlier_detection_ZH.pdf**、课堂转录 **14Outlier Detection.txt**，以及配套习题解答 **sol.pdf** 重构。如果更早对话中有部分内容没有保留在当前上下文里，我无法逐字恢复，但我会把当前可见材料中的知识点尽可能完整地整合成最终版复习笔记。

# COMP6237 Data Mining：Outlier Detection 最终复习笔记

## 1. 本章核心目标

**异常检测（Outlier Detection / Anomaly Detection）** 的目标是识别那些明显偏离数据正常模式的观测点。讲义中的定义是：outlier detection aims to identify observations that deviate significantly from the normal pattern of the data；outlier 是与大多数观测所遵循模式不一致的数据点。

本章考试重点非常明确：讲义最后的 learning outcomes 提示，考试以 **single-answer / multi-choice questions** 为主，会考概念、计算、方法优缺点，包括 **extreme value analysis**、用于异常检测的聚类方法步骤、以及各种异常检测方法的优缺点。

本章可以用一句话概括：

> **先定义“正常模式”，再判断某个点是否足够不符合这个正常模式。**

但真正困难的地方在于：  
**正常模式不是唯一的。**  
它可以是一个高斯分布，可以是多个高斯分布的混合，也可以是局部密度结构。不同方法对“正常”的假设不同，因此适用场景和失败场景也不同。

---

## 2. Outlier、Anomaly、Extreme Value、Noise 的区别

### 2.1 Outlier / Anomaly：异常点

**Outlier（离群点 / 异常点）** 指与大多数数据点所形成的模式不一致的对象。它并不一定是错误数据，也不一定是噪声；它可能代表一个真正有意义的少数类。

例如：

- 信用卡欺诈交易；
    
- 网络入侵；
    
- 医疗监控中心率异常；
    
- 搜索引擎中突然爆发的异常流量；
    
- 工业传感器中的异常读数。
    

在数据挖掘里，异常检测通常不是简单地查找“最大值”或“最小值”，而是查找 **不符合正常生成机制的数据对象**。

---

### 2.2 Extreme Value：极端值不一定是异常点

讲义特别强调：**Extreme values are not outliers!** 极端值只是正常分布尾部中罕见但仍然可能出现的值；真正的 outlier 则是过于不符合当前正常数据模式的点。

例如，假设人的身高服从近似正态分布：

- 190cm 可能是极端值，但仍然属于正常人类身高分布；
    
- 260cm 或数据中出现 “999cm”，则更可能是异常值或数据错误。
    

课堂转录中也强调，真实数据里经常出现一些很大或很小的值，但它们未必是 outliers，它们可能只是主数据分布的极端尾部。Extreme value theory 的作用就是帮助区分 **真实异常（real anomalies）** 和 **正常分布中的极端样本（extremes of the primary data distribution）**。

---

### 2.3 Noise：噪声与异常点不是一回事

**Noise（噪声）** 是数据中的随机扰动，通常没有分析价值；但 **outlier** 可能有分析价值。sol.pdf 中明确区分了二者：噪声本身通常不 interesting，但 outliers 可以 interesting；噪声可能导致 outlier，但不是所有噪声都是 outlier，也不是所有 outlier 都是噪声。

可以这样理解：

|概念|含义|是否有价值|例子|
|---|---|--:|---|
|Noise|随机扰动 / 测量误差|通常无价值|传感器抖动|
|Extreme value|正常分布尾部的极端样本|可能有价值|很高但合理的身高|
|Outlier / Anomaly|不符合正常模式的对象|常常有价值|欺诈交易、网络攻击|
|Error|数据录入或采集错误|通常要清洗|年龄 = 999|

考试陷阱：

> **Outlier 不等于 noise。Outlier 也不等于 extreme value。**

---

## 3. 异常检测的通用 Data Mining 思路

讲义给出的通用框架是：

1. **Model the data**：先建模正常数据；
    
2. **What does not fit is outlier**：不符合模型的数据就是异常；
    
3. 需要一个 **measure of fit（拟合程度度量）**。
    

课堂转录中也反复强调，不同异常检测方法的共同点是：都要衡量一个数据点 **how well it fits the model**。如果某个点在模型下太不可能、距离太远、密度太低或无法被正常簇覆盖，就可以被判为异常。

因此，异常检测的核心不是某一个算法，而是下面这个思想：

```text
给定数据 D
↓
建立一个“正常模式”模型 M
↓
对每个测试点 x 计算异常分数 score(x)
↓
如果 score(x) 超过阈值 threshold
↓
判定 x 为 outlier
```

异常分数可以来自：

- 概率低：probability-based；
    
- 距离远：distance-based；
    
- 密度低：density-based；
    
- 不属于任何聚类：cluster-based；
    
- 不符合规则：pattern-based。
    

本节讲义重点覆盖三类方法：

1. **Extreme Value Statistics**
    
2. **Gaussian / Gaussian Mixture Model**
    
3. **DBSCAN**
    

---

# Part I：Extreme Value Statistics

## 4. Extreme Value Statistics：为什么极端值不是异常值

### 4.1 基本思想

**Extreme Value Statistics（极值统计）** 用来建模正常数据分布中最大值或最小值的分布。它类似于 Central Limit Theorem 对均值分布的描述，只不过它研究的是极端值。讲义中称它也与 **Fisher–Tippett theorem** 有关，并指出极端值的极限分布属于三种 generalized extreme value distributions：**Gumbel、Fréchet、Weibull**。

这部分最重要的考试理解是：

> 极值统计不是为了把所有大值都判成异常，而是为了判断一个极端值是否仍然可能来自正常分布。

---

### 4.2 Gumbel Distribution 的作用

讲义中的 demo 使用 **Gumbel distribution（甘贝尔分布）** 来拟合正态分布样本最大值的分布。页面 4 的图展示了两种情况：每次从若干 Gaussian samples 中取最大值，然后观察这些最大值的直方图；随着样本数量增加，最大值分布更加清晰，并可以用 Gumbel 分布拟合。

直观解释：

假设每次从标准正态分布中抽样：

```text
Trial 1: 取 10 个 normal samples，记录最大值
Trial 2: 取 10 个 normal samples，记录最大值
...
重复 1000 次
```

这些“最大值”本身不是正态分布，而是形成一个新的极值分布。Gumbel distribution 可以估计：

> 某个很大的值是否仍然可能只是“正常数据中的最大值”。

---

### 4.3 Extreme Value 和 Outlier 的判断逻辑

假设某个点很大，我们不能立刻说它是 outlier，而要问：

1. 它是否可能只是正常分布尾部的 extreme value？
    
2. 在 extreme value model 下，它的概率是否仍然合理？
    
3. 如果它连极端值模型都解释不了，才更可能是真正 outlier。
    

因此：

```text
极端 ≠ 异常
极端 + 在正常极值分布下仍合理 → extreme value
极端 + 在正常极值分布下极不可能 → outlier
```

---

# Part II：Gaussian Distribution-Based Outlier Detection

## 5. 一维 Gaussian Model：最自然的第一种方法

### 5.1 基本假设

最简单的异常检测思路是：假设正常数据服从 **Gaussian distribution（高斯分布 / 正态分布）**。

一维正态分布：

[  
p(x)=\frac{1}{\sigma\sqrt{2\pi}}\exp\left(-\frac{1}{2}\frac{(x-\mu)^2}{\sigma^2}\right)  
]

其中：

- (\mu)：mean，均值，表示数据中心；
    
- (\sigma)：standard deviation，标准差，表示数据分散程度。
    

讲义中从一维 Gaussian 开始，说明我们只需要估计均值和标准差，然后看测试点是否离这个分布太远。

---

### 5.2 一维异常检测逻辑

对于测试点 (x)，可以计算它离均值有多远：


$z=\frac{x-\mu}{\sigma}$  


如果 (|z|) 很大，则说明 (x) 离均值很多个标准差，可能是异常值。

常见直觉：

```text
|z| < 2       通常比较正常
2 <= |z| < 3  比较极端
|z| >= 3      很少见，可能异常
```

但注意：**3σ rule 在大数据中会产生很多“异常”**。sol.pdf 中 Chapter 10 的习题指出，如果有 1,000,000 个正态样本，那么超过 3 个标准差的点并不少：单侧约 1,350 个，双侧约 2,700 个。这说明在大数据集中，固定的低概率阈值可能需要调整。

---

## 6. 多维 Gaussian Model：从均值和标准差扩展到均值向量和协方差矩阵

### 6.1 为什么需要 Multivariate Gaussian？

真实数据通常是多维的。例如：

```text
x = [height, weight, blood pressure, age]
```

一维 Gaussian 只考虑一个变量；多维情况下，需要用 **Multivariate Gaussian Distribution（多元高斯分布）** 来描述数据云。

多元高斯分布：

[  
$$
p(x)=\frac{1}{(2\pi)^{d/2}|C|^{1/2}}  
\exp\left(  
-\frac{1}{2}(x-m)^T C^{-1}(x-m)  
\right)  
]
$$

其中：

- (d)：维度数量；
    
- (m)：mean vector，均值向量；
    
- (C)：covariance matrix，协方差矩阵；
    
- (|C|)：协方差矩阵的 determinant；
    
- (C^{-1})：协方差矩阵的逆矩阵。
    

讲义中强调，多维 Gaussian 需要估计 **mean vector** 和 **covariance matrix**；协方差矩阵非常重要，因为它描述了数据云的形状以及多个维度之间的相关关系。

---

### 6.2 均值向量与协方差矩阵

均值向量：

[  
m=\frac{1}{N}\sum_{i=1}^{N}x_i  
]

协方差矩阵：

[  
C=\frac{1}{N}\sum_{i=1}^{N}(x_i-m)(x_i-m)^T  
]

协方差矩阵的作用不是简单地记录每个维度的方差，而是同时记录：

1. 每个维度自己的尺度；
    
2. 不同维度之间的相关性；
    
3. 数据云整体的椭圆方向和形状。
    

---

## 7. Mahalanobis Distance：为什么不用普通欧氏距离？

### 7.1 欧氏距离的问题

普通欧氏距离：

[  
d_E(x,m)=\sqrt{\sum_j(x_j-m_j)^2}  
]

它默认每个维度：

- 尺度相同；
    
- 相互独立；
    
- 数据云是圆形的。
    

但真实数据常常不是圆形，而是椭圆形。例如身高和体重高度相关：一个高的人体重也可能更高。如果一个点沿着正常相关方向偏离，它可能并不异常；如果它垂直于相关方向偏离，即使欧氏距离不大，也可能异常。

---

### 7.2 Mahalanobis Distance 的定义

**Mahalanobis distance（马氏距离）**：

[  
d_M^2(x)=(x-m)^T C^{-1}(x-m)  
]

它衡量的是：

> 点 (x) 离高斯数据云有多远，同时考虑数据方差和维度相关性。

课堂中提到，多维 Gaussian 模型通常用 **Mahalanobis distance** 或 Gaussian probability 来衡量一个点是否 fit 数据云。

---

### 7.3 Mahalanobis Distance 与 Gaussian Probability 的关系

sol.pdf 中 Chapter 10 Exercise 9 给出了一个非常重要的推导：多元正态分布的 log probability 等于一个与 (x) 无关的常数，加上 Mahalanobis distance 的负比例项。因此，如果只关心异常排序，低 log probability 与高 Mahalanobis distance 本质上是同一件事。

具体地：

$$
[  
\log p(x)

-\log\left((\sqrt{2\pi})^d |C|^{1/2}\right)  
-\frac{1}{2}(x-m)^TC^{-1}(x-m)  
]
$$

其中第一项与 (x) 无关，是常数；第二项就是 (-\frac{1}{2}d_M^2(x))。

因此：

```text
Mahalanobis distance 越大
↓
Gaussian probability 越小
↓
点越不像来自正常 Gaussian
↓
越可能是 outlier
```

这是一个非常容易考的概念题。

---

## 8. Gaussian Model 的阈值：Chi-square Threshold

### 8.1 为什么用 Chi-square？

如果数据真的来自 (d)-维多元高斯分布，那么：

$$
[  
d_M^2(x)\sim \chi^2_d  
]
$$

也就是说，**平方马氏距离服从自由度为 (d) 的卡方分布**。

课堂转录中提到，判断阈值可以用 chi-square function，例如设置较高置信度 0.95 或 0.99；如果 Mahalanobis distance 超过阈值，就可以把点当作 outlier。

---

### 8.2 阈值计算方式

如果维度是 (d)，显著水平选择 (\alpha)，则阈值为：

[  
T=\chi^2_d(\alpha)  
]

判断规则：

[  
d_M^2(x)>T \Rightarrow x \text{ is outlier}  
]

例如，二维数据 (d=2)：

[  
\chi^2_2(0.95)\approx 5.99  
]

[  
\chi^2_2(0.99)\approx 9.21  
]

讲义 demo 中 Gaussian 和 GMM 的图里经常出现 threshold = 9.21，这对应的就是二维情况下 99% 左右的卡方阈值。

---

## 9. Gaussian Model 的 Deep Dive：为什么它有时失败？

讲义 demo 使用了四种数据集：

1. **Single correlated Gaussian cloud**：单个相关高斯云；
    
2. **Concentric circles**：同心圆；
    
3. **Two separated Gaussian clusters**：两个分离的高斯簇；
    
4. **Odd-shaped / spiral-like dataset**：非高斯、不规则形状数据。
    

页面 8 的图展示了单个 Gaussian 模型在四个数据集上的效果：它可以较好处理第一个椭圆状 Gaussian cloud，但在同心圆、两个高斯簇和奇异形状数据上会误检或漏检。

核心原因：

> **Single Gaussian 的模型假设是：正常数据大致是一个椭圆形、单峰的数据云。**

如果真实数据不满足这个假设，Gaussian model 就会失败。

---

### 9.1 失败场景 1：Concentric Circles

同心圆数据不是一个椭圆云，而是环状结构。  
单个 Gaussian 会倾向于在中心放一个椭圆，但中心区域可能实际上没有正常点。

结果：

- 圆环内部的一些 planted outliers 可能被误认为正常；
    
- 圆环外部点可能被判异常；
    
- 模型无法表达“中间空洞”的结构。
    

---

### 9.2 失败场景 2：Two Gaussian Clusters

两个分离高斯簇如果用一个 Gaussian 拟合，会得到一个很大的椭圆，覆盖两个簇之间的空白区域。

结果：

- 两个簇中间的点可能被误判为正常；
    
- 因为椭圆被拉大，真正异常点可能被漏检；
    
- 模型把多峰结构强行压成单峰结构。
    

课堂转录中明确说：two Gaussians 比单个 Gaussian 更复杂，如果只用一个 ellipse，会出现 model assumption 与 real dataset 之间的 gap。

---

### 9.3 失败场景 3：Spiral / Odd-shaped Data

螺旋或非凸形状完全不符合 Gaussian ellipse 假设。  
一个椭圆无法贴合这种弯曲的流形结构。

结果：

- 椭圆内部很多空白区域被认为正常；
    
- 螺旋边缘正常点可能被认为异常；
    
- 模型既可能 over-detect，也可能 under-detect。
    

---

## 10. Gaussian Model 总结

Gaussian-based outlier detection 的优点：

- 数学清晰；
    
- 可以给出 probability 或 Mahalanobis distance；
    
- 阈值可用 chi-square 分布解释；
    
- 对单峰、椭圆状数据效果好；
    
- 适合考试计算题。
    

缺点：

- 假设太强；
    
- 对非高斯形状失败；
    
- 对多峰数据失败；
    
- 均值和协方差容易被异常点影响；
    
- 高维下协方差矩阵估计困难，可能奇异；
    
- 阈值选择与数据规模有关。
    

考试一句话模板：

> Gaussian outlier detection models normal data as a single elliptical cloud. It works well when data is approximately unimodal Gaussian, but fails for multi-modal, ring-shaped, or irregular-shaped data because the model assumption does not match the true data distribution.

---

# Part III：Gaussian Mixture Model

## 11. 为什么需要 Gaussian Mixture Model？

Single Gaussian 只能建模一个椭圆形数据云。  
如果数据有多个簇，例如两个分离的 Gaussian clusters，一个 Gaussian 不够用。

自然想法：

> 用多个 Gaussian 来拟合多个正常区域。

这就是 **Gaussian Mixture Model（GMM，高斯混合模型）**。

讲义中明确写道：try using more than one Gaussian，即 Gaussian Mixture Model；GMM 可以看作是多个 Gaussian distribution 的加权和，并使用 Expectation Maximisation 估计参数。

---

## 12. GMM 的概率模型

GMM 的概率密度：

[  
p(x)=\sum_{k=1}^{K}\pi_k \mathcal{N}(x\mid \mu_k,C_k)  
]

其中：

- (K)：Gaussian components 的数量；
    
- (\pi_k)：第 (k) 个 Gaussian 的 mixing weight；
    
- (\mu_k)：第 (k) 个 Gaussian 的均值；
    
- (C_k)：第 (k) 个 Gaussian 的协方差矩阵；
    
- (\sum_{k=1}^{K}\pi_k=1)。
    

直观解释：

```text
一个数据点 x 可能来自 component 1，也可能来自 component 2，也可能来自 component K。
GMM 不直接硬分配，而是给每个 component 一个概率责任。
```

---

## 13. GMM 中需要估计哪些参数？

GMM 需要估计三类参数：

1. **Mixing weights**：(\pi_k)
    
2. **Means**：(\mu_k)
    
3. **Covariances**：(C_k)
    

困难在于：

> 如果知道每个点属于哪个 Gaussian，就容易估计每个 Gaussian 的参数；  
> 但如果知道每个 Gaussian 的参数，也容易估计每个点属于哪个 Gaussian。  
> 问题是：一开始两者都不知道。

因此需要 **EM algorithm（Expectation-Maximisation，期望最大化算法）**。

---

## 14. EM Algorithm：GMM 的核心优化方法

讲义中 GMM algorithm 分为：

- **E-step**：evaluate posterior probabilities；
    
- **M-step**：estimate new parameters；
    
- 然后 evaluate log-likelihood。
    

### 14.1 E-step：计算 responsibility

**Responsibility（责任度 / 软分配概率）**：

$$
[  
r_{ik}

 P(z_i=k\mid x_i)=

\frac{\pi_k\mathcal{N}(x_i\mid \mu_k,C_k)}  
{\sum_{j=1}^{K}\pi_j\mathcal{N}(x_i\mid \mu_j,C_j)}  
]
$$

它表示：

> 第 (k) 个 Gaussian 对点 (x_i) 负责的概率是多少。

注意，这不是 hard assignment，而是 soft assignment。

例如：

```text
x_i belongs to Gaussian 1 with probability 0.8
x_i belongs to Gaussian 2 with probability 0.2
```

---

### 14.2 M-step：更新参数

先计算每个 component 的有效样本数：

[  
N_k=\sum_{i=1}^{N}r_{ik}  
]

更新 mixing weight：

[  
\pi_k=\frac{N_k}{N}  
]

更新均值：

[  
\mu_k=\frac{1}{N_k}\sum_{i=1}^{N}r_{ik}x_i  
]

更新协方差：

[  
C_k=\frac{1}{N_k}\sum_{i=1}^{N}r_{ik}(x_i-\mu_k)(x_i-\mu_k)^T  
]

---

### 14.3 Log-likelihood：判断是否收敛

GMM 的 log-likelihood：

[  
\ell(\theta)=\sum_{i=1}^{N}\log\left(  
\sum_{k=1}^{K}\pi_k\mathcal{N}(x_i\mid \mu_k,C_k)  
\right)  
]

EM 每轮迭代通常会提高或不降低 log-likelihood。讲义中展示了 GMM convergence curve：前几轮 log-likelihood 快速上升，之后趋于平稳。

停止条件通常是：

```text
如果 log-likelihood 的提升小于某个 tolerance
或者达到最大迭代次数
则停止
```

---

## 15. GMM 伪代码

```text
Input:
    Data X = {x_1, ..., x_N}
    Number of components K

Initialise:
    π_k, μ_k, C_k for k = 1,...,K

Repeat until convergence:

    E-step:
        For each point x_i:
            For each component k:
                Compute responsibility r_ik
                r_ik = π_k N(x_i | μ_k, C_k) /
                       Σ_j π_j N(x_i | μ_j, C_j)

    M-step:
        For each component k:
            N_k = Σ_i r_ik
            π_k = N_k / N
            μ_k = (1/N_k) Σ_i r_ik x_i
            C_k = (1/N_k) Σ_i r_ik (x_i - μ_k)(x_i - μ_k)^T

    Evaluate:
        log-likelihood

Output:
    Estimated mixture model
```

---

## 16. GMM 与 K-means 的关系

讲义中说 EM 是 K-means 的 generalisation。

可以这样理解：

|K-means|GMM|
|---|---|
|Hard assignment|Soft assignment|
|每个点只属于一个 cluster|每个点属于每个 component 有概率|
|只考虑距离|同时考虑距离、方差、协方差、混合权重|
|cluster 形状通常近似球形|component 可以是椭圆形|
|优化 SSE|优化 log-likelihood|

sol.pdf 中关于 EM clustering 的解释也指出，EM 中点属于某个簇的概率不仅取决于距离，还取决于 cluster 的 spread / variance；一个点即使离某个 centroid 更近，也可能因为另一个 cluster 方差更大而对另一个 cluster 有更高概率。K-means 则只考虑到最近中心的距离，相当于假设所有簇方差相同。

---

## 17. GMM 初始化问题

讲义强调：

- random initialization can cause issues；
    
- use K-means works quite well。
    

原因是 GMM 的目标函数非凸，EM 只能保证收敛到局部最优。初始点不好时，可能出现：

- component 重叠；
    
- component 空掉；
    
- 协方差估计不稳定；
    
- 收敛到差的局部最优；
    
- 某个 component 被 outlier 拉走。
    

因此实践中经常先用 K-means 初始化：

```text
Step 1: 用 K-means 找 K 个 cluster
Step 2: 用 cluster mean 初始化 μ_k
Step 3: 用 cluster covariance 初始化 C_k
Step 4: 用 cluster proportion 初始化 π_k
Step 5: 再运行 EM
```

---

## 18. 用 GMM 做 Outlier Detection

训练好 GMM 后，可以用以下方式给点打异常分数：

### 方法 1：用 mixture probability

[  
score(x)=-\log p(x)  
]

如果 (p(x)) 很低，说明该点不太可能来自任何一个 Gaussian component。

判断：

[  
-\log p(x)>T \Rightarrow outlier  
]

---

### 方法 2：对每个 component 计算 Mahalanobis distance

对每个 component：

[  
d_{M,k}^2(x)=(x-\mu_k)^TC_k^{-1}(x-\mu_k)  
]

取最近或最大责任 component：

[  
\min_k d_{M,k}^2(x)  
]

如果点对所有 component 都很远，则异常。

---

### 方法 3：责任度低 / 不属于任何 component

如果：

```text
max_k r_ik 很低
```

说明没有任何一个 component 能很好解释这个点，也可以判断为异常。

---

## 19. GMM 的适用场景与失败场景

页面 14 的 demo 显示，GMM 对两个分离 Gaussian clusters 明显比 single Gaussian 更合理，但对于同心圆和不规则形状数据仍然不理想。

### 19.1 GMM 适合什么？

GMM 适合：

- 多峰分布；
    
- 每个簇近似 Gaussian；
    
- 每个正常区域可以用椭圆表示；
    
- 数据分布可以看作多个椭圆云叠加。
    

例如：

```text
客户群体分成多个正常消费模式
传感器正常状态有多个工作模式
图像特征形成多个类高斯簇
```

---

### 19.2 GMM 不适合什么？

GMM 不适合：

- 同心圆；
    
- 螺旋；
    
- 非凸形状；
    
- manifold-like data；
    
- 密度变化很大的数据；
    
- 异常点很多且会扭曲参数估计的数据。
    

讲义 summary 中也指出，GMM models the system as a mixture of Gaussian distributions, uses EM to find parameters, but can be distorted by outliers。

---

# Part IV：DBSCAN-Based Outlier Detection

## 20. 为什么引入 DBSCAN？

Gaussian 和 GMM 都是模型拟合方法。它们试图用一个或多个 Gaussian 来解释数据。  
但如果正常数据形状非常复杂，比如：

- 同心圆；
    
- 月牙形；
    
- 螺旋形；
    
- 任意非凸 cluster；
    

Gaussian/GMM 都会很难处理。

于是引入 **DBSCAN（Density-Based Spatial Clustering of Applications with Noise）**。

讲义中说明，DBSCAN 既适合 clustering，也适合 outlier detection；它需要两个参数：maximum radius 和 minimum number。

---

## 21. DBSCAN 的核心直觉：密度

DBSCAN 的基本思想非常直观：

> 正常点通常处于 dense region；异常点通常处于 sparse region。

课堂中用“教室座位”的例子解释：如果一群人坐得很密集，那么这个区域是 dense region；如果一个人孤零零地坐在远处，则可能是 sparse region 中的异常点。

---

## 22. DBSCAN 的两个核心参数

### 22.1 ($$\varepsilon$$)：neighbourhood radius

[  
$$
N_{\varepsilon}(x)={y\mid \delta(x,y)\leq \varepsilon}  
$$
]

它表示：

$$
 在半径 (\varepsilon) 内，点 (x) 的邻居集合。

(\varepsilon) 越大，邻域越大；(\varepsilon) 越小，邻域越小。
$$

---

### 22.2 minPts：minimum number of neighbours

DBSCAN 判断一个点是否在 dense region，需要看它的 (\varepsilon)-neighbourhood 里是否有足够多点：

[  
|N_{\varepsilon}(x)|\geq minPts  
]

满足这个条件的点叫 **core point（核心点）**。

注意：很多实现中 (N_{\varepsilon}(x)) 包含点 (x) 自己，因此 minPts 往往包括自己。

---

## 23. DBSCAN 中三类点

### 23.1 Core Point：核心点

如果一个点在半径 (\varepsilon) 内有至少 (minPts) 个点：

[  
|N_{\varepsilon}(x)|\geq minPts  
]

则它是 core point。

核心点代表 dense region 的内部。

---

### 23.2 Border Point：边界点

如果一个点本身不是 core point，但它在某个 core point 的邻域内，则它是 border point。

直观理解：

```text
它靠近 dense region，但自己周围不够密。
```

border point 会被分配到 cluster，但不会继续扩展 cluster。

---

### 23.3 Noise / Outlier：噪声点 / 异常点

如果一个点：

- 不是 core point；
    
- 也不能从任何 core point 到达；
    

则它是 noise point，也就是 DBSCAN 中的 outlier。

课堂转录中总结：DBSCAN 会输出 clusters，而那些 unreachable / untouched points 可以被视为 outliers。

---

## 24. DBSCAN 算法流程

讲义页面 16 给出了 DBSCAN pseudo-code：先找 core points，然后从每个未访问 core point 出发，把邻居加入 queue，持续扩展 cluster；如果邻居也是 core point，就继续加入它的邻居。

整理后的伪代码：

```text
Input:
    Data X
    eps
    minPts

Step 1:
    For each point x:
        Compute eps-neighbourhood N_eps(x)
        If |N_eps(x)| >= minPts:
            Mark x as core point

Step 2:
    Initialise all labels as unassigned
    cluster_id = 1

Step 3:
    For each unvisited core point p:
        Create a new cluster
        Add p to a queue

        While queue is not empty:
            q = pop(queue)
            Assign q to current cluster

            For each neighbour n in N_eps(q):
                If n is unassigned:
                    Assign n to current cluster

                If n is core point:
                    Add neighbours of n to queue

        cluster_id += 1

Step 4:
    Points not assigned to any cluster are noise / outliers

Output:
    Cluster labels and noise points
```

---

## 25. DBSCAN 的 Deep Dive：为什么它能处理任意形状？

DBSCAN 不是用一个椭圆拟合数据，而是通过 **density reachability（密度可达）** 把 core points 连起来。

如果两个 core points 的 (\varepsilon)-neighbourhood 重叠，它们就可能属于同一个 cluster。

因此，cluster 可以沿着密集区域不断延伸，形成：

- 弯曲形状；
    
- 月牙形；
    
- 环形；
    
- 螺旋形；
    
- 非凸形状。
    

这就是 DBSCAN 相比 Gaussian/GMM 的最大优势：

> 它不假设 cluster 必须是圆形或椭圆形。

---

## 26. DBSCAN 的异常检测逻辑

DBSCAN 输出的 noise points 就是候选 outliers：

```text
正常点：属于某个 dense cluster
边界点：可由 core point reach，仍属于 cluster
异常点：无法由任何 core point reach
```

所以 DBSCAN 做异常检测时的判断是：

> 如果一个点周围没有足够多邻居，并且也不靠近任何核心密度区域，则它是异常点。

---

## 27. DBSCAN 参数影响

### 27.1 (\varepsilon) 太小

如果 (\varepsilon) 太小：

- 每个点邻居很少；
    
- core points 数量减少；
    
- 很多正常点被判为 noise；
    
- 出现 over-detection。
    

结果：

```text
正常点被误判为异常
```

---

### 27.2 (\varepsilon) 太大

如果 (\varepsilon) 太大：

- 很多点都互为邻居；
    
- 不同 cluster 可能被连接起来；
    
- outliers 可能被吸收到 cluster；
    
- 出现 under-detection。
    

结果：

```text
真正异常点被漏检
```

---

### 27.3 minPts 太小

如果 minPts 太小：

- 很容易成为 core point；
    
- 小噪声团也可能成为 cluster；
    
- 异常点可能被认为正常。
    

---

### 27.4 minPts 太大

如果 minPts 太大：

- 成为 core point 的条件太严格；
    
- 稀疏但正常的 cluster 可能被判为 noise；
    
- 异常检测过度。
    

---

## 28. DBSCAN 为什么需要 Normalisation？

课堂转录中特别强调：DBSCAN 是 distance-based method，因此通常需要 normalisation；如果不同维度尺度不同，距离会被大尺度特征主导，导致邻域判断不可靠。

例如：

```text
Feature 1: age, range 0–100
Feature 2: income, range 0–100000
```

如果不归一化，Euclidean distance 几乎完全由 income 决定，age 的影响被淹没。

因此在 DBSCAN 前通常要做：

[  
x'_j=\frac{x_j-\mu_j}{\sigma_j}  
]

或者 min-max scaling：

[  
x'_j=\frac{x_j-\min(x_j)}{\max(x_j)-\min(x_j)}  
]

---

## 29. DBSCAN demo 的关键现象

讲义页面 18 展示 raw DBSCAN 结果，页面 19 展示 normalisation and adjusting eps 后的结果。raw DBSCAN 可能出现 over-detection 或 under-detection；归一化并调节 eps 后效果改善，但仍然可能有误检或漏检。

课堂转录中也提到，DBSCAN 在理论上能处理 odd-shaped data，但如果参数没有调好，会表现很差；它检测到的 noise 数量可能超过真实 planted outliers，属于 over-detection；也可能漏掉某些 outlier，属于 under-detection。

---

## 30. DBSCAN 的优缺点

### 优点

- 可以发现任意形状 cluster；
    
- 不需要预先指定 cluster 数量；
    
- 自然输出 noise / outliers；
    
- 对 outliers 相对 robust；
    
- 很适合非凸形状数据。
    

讲义 summary 中也指出，DBSCAN is used for outlier detection and robust to outliers。

---

### 缺点

- 需要调 (\varepsilon) 和 minPts；
    
- 对距离尺度敏感，需要 normalisation；
    
- 不适合密度差异很大的数据；
    
- 高维空间中距离度量失效；
    
- 全局 (\varepsilon) 难以同时适配不同 density region；
    
- 输出的 anomaly score 不如概率模型自然。
    

考试模板：

> DBSCAN is effective for detecting outliers in arbitrary-shaped clusters because it relies on local density rather than Gaussian assumptions. However, it is sensitive to eps and minPts, requires feature scaling, and struggles with varying densities or high-dimensional data.

---

# Part V：三类方法系统对比

## 31. Gaussian vs GMM vs DBSCAN

|方法|正常模式假设|异常判定|适合数据|主要优点|主要缺点|
|---|---|---|---|---|---|
|Gaussian|单个椭圆高斯云|Mahalanobis distance / low probability|单峰椭圆数据|数学简单，可解释|对多峰、环形、复杂形状失败|
|GMM|多个 Gaussian components|low mixture probability / all components far|多峰椭圆簇|比 single Gaussian 灵活|仍不适合非凸形状；需选 K|
|DBSCAN|dense regions 是正常|unreachable noise points|任意形状 cluster|不需指定 K，能找 noise|参数敏感，需标准化，不适合变密度|

---

## 32. 四种数据形状下的方法选择

### 32.1 Single correlated Gaussian cloud

最佳方法：

```text
Gaussian model
```

原因：

- 数据近似单个椭圆；
    
- mean + covariance 足够描述；
    
- Mahalanobis distance 有清晰解释。
    

---

### 32.2 Two separated Gaussian clusters

最佳方法：

```text
GMM
```

原因：

- 数据有两个模式；
    
- single Gaussian 会把两个簇之间的空白区域误认为正常；
    
- GMM 可以用两个 component 分别拟合。
    

---

### 32.3 Concentric circles

较合适方法：

```text
DBSCAN
```

原因：

- 环形结构不是 Gaussian；
    
- GMM 需要很多 components 才能近似；
    
- DBSCAN 可以沿密度结构形成环形 cluster。
    

---

### 32.4 Spiral / odd-shaped data

较合适方法：

```text
DBSCAN
```

原因：

- 正常点沿复杂形状分布；
    
- 椭圆模型无法表达；
    
- density reachability 可以沿着螺旋扩展 cluster。
    

---

# Part VI：异常检测评估指标

## 33. Detection Rate

**Detection rate（检测率）**：

[  
Detection\ Rate=  
\frac{\text{number of anomalies detected}}  
{\text{total number of anomalies}}  
]

它衡量：

> 真正异常中有多少被检测出来。

类似于 supervised classification 里的 recall / true positive rate。

---

## 34. False Alarm Rate

sol.pdf 中的定义是：

[  
False\ Alarm\ Rate=  
\frac{\text{number of false anomalies}}  
{\text{number of objects classified as anomalies}}  
]

这个定义更接近“被标记为异常的对象中，有多少其实是正常的”，类似 false discovery proportion，而不是传统机器学习中 FPR 的定义。sol.pdf 给出的例子是：如果 99% 的对象正常，正常对象被误判异常概率是 0.01，异常对象被检测为异常概率是 0.99，那么 detection rate 是 99%，但 false alarm rate 是 50%。

计算逻辑：

假设总数为 (m)：

正常对象数：

[  
0.99m  
]

异常对象数：

[  
0.01m  
]

误报正常对象：

[  
0.99m\times 0.01=0.0099m  
]

检测到的异常对象：

[  
0.01m\times 0.99=0.0099m  
]

所有被标为异常的对象：

[  
0.0099m+0.0099m=0.0198m  
]

false alarm rate：

[  
\frac{0.0099m}{0.0198m}=0.5  
]

所以即使模型看起来“每类都 99% 正确”，由于异常极少，被标为异常的对象中仍然可能有一半是假警报。

---

## 35. 类别极不平衡下的陷阱

异常检测几乎总是面对极端不平衡：

```text
normal: 99%
anomaly: 1%
```

因此不能只看 accuracy。

例如：

```text
全部预测 normal
accuracy = 99%
但 detection rate = 0%
```

这在异常检测中是完全不可接受的。

应该关注：

- detection rate / recall；
    
- false alarm rate；
    
- precision；
    
- anomaly ranking quality；
    
- top-k detected anomalies；
    
- domain investigation cost。
    

---

# Part VII：Supervised vs Unsupervised Anomaly Detection

## 36. Supervised Anomaly Detection

如果有大量标注好的 normal 和 anomaly 数据，可以训练 supervised classifier：

```text
input x → classifier → normal / anomaly
```

优点：

- 对已知异常类型效果好；
    
- 可以直接优化分类性能；
    
- 可以使用 precision、recall、F1、ROC-AUC 等指标。
    

缺点：

- 需要标注异常样本；
    
- 异常样本通常很少；
    
- 对新型异常不一定有效。
    

---

## 37. Unsupervised Anomaly Detection

无监督异常检测只用数据本身建模：

```text
大多数数据是 normal
少数不符合模式的是 anomaly
```

优点：

- 不需要异常标签；
    
- 可以发现新型异常；
    
- 适合 fraud detection / intrusion detection 中不断变化的新模式。
    

缺点：

- 假设异常是少数；
    
- 阈值难选；
    
- 结果解释和评估更难；
    
- 误报可能较多。
    

sol.pdf 中指出，当新型异常不断出现时，必须使用 unsupervised anomaly detection；但 supervised 方法仍然适合检测已知异常类型，因此两者最好结合使用，例如网络入侵检测中，已知攻击可以用 signatures，未知攻击需要 unsupervised 方法。

---

# Part VIII：重要难点剖析

## 38. 难点 1：为什么 “low probability” 不一定意味着异常？

因为在大数据中，即使概率很低的事件也会出现很多次。

例如正态分布下：

```text
P(|Z| > 3) ≈ 0.0027
```

如果只有 100 个样本：

```text
期望数量 ≈ 0.27
```

可能很少见。

但如果有 1,000,000 个样本：

```text
期望数量 ≈ 2700
```

这时 3σ 外的点很多，不能简单认为每一个都是异常。sol.pdf 中正是用这个例子说明大数据场景下传统统计 outlier test 的局限。

---

## 39. 难点 2：为什么 Mahalanobis distance 更适合相关数据？

假设二维数据中两个变量高度相关，例如：

```text
x1 = height
x2 = weight
```

正常人群大致沿着“身高越高，体重越高”的方向形成椭圆。

现在有两个点：

```text
A: 很高且较重
B: 中等身高但极重
```

欧氏距离可能认为 A 离中心更远；但 Mahalanobis distance 会考虑数据云方向：

- A 沿着正常相关方向偏离，可能仍然合理；
    
- B 垂直于正常相关方向偏离，更可能异常。
    

所以 Mahalanobis distance 判断的是：

> 相对于数据自身的形状和方向，这个点有多不正常。

---

## 40. 难点 3：为什么 single Gaussian 会漏掉两个 cluster 中间的异常点？

两个分离 cluster 的数据形状如下：

```text
Cluster 1          Cluster 2
  xxxx               xxxx
 xxxxxx             xxxxxx
  xxxx               xxxx
```

如果用 single Gaussian 拟合，它会生成一个大椭圆：

```text
( Cluster 1     empty gap      Cluster 2 )
```

中间空白区域虽然没有正常点，但可能被大椭圆覆盖。  
因此，一个落在中间空白区域的 planted outlier 可能被认为是正常。

根本原因：

> single Gaussian 只知道整体均值和整体协方差，不知道数据其实有两个模式。

---

## 41. 难点 4：GMM 为什么仍然不能很好处理同心圆？

同心圆是这样的：

```text
   outer ring
  ooooooooooo
 oo         oo
oo  inner    oo
oo  ring     oo
 oo         oo
  ooooooooooo
```

每个 Gaussian component 是椭圆形。  
要用 GMM 拟合同心圆，需要很多小椭圆沿圆环排列。

如果 K 太小：

- 无法表达环形；
    
- 中心空洞可能被误判正常；
    
- outlier 可能被漏检。
    

如果 K 很大：

- 参数复杂；
    
- 容易过拟合；
    
- 初始化和收敛更困难。
    

因此，GMM 比 single Gaussian 灵活，但它仍然主要适合“多个椭圆云”，而不是任意形状流形。

---

## 42. 难点 5：DBSCAN 为什么对参数敏感？

DBSCAN 的密度定义完全依赖 (\varepsilon) 和 minPts。

同一个数据集：

```text
eps = 0.1 → 很多点没有邻居 → 大量 noise
eps = 1.0 → 很多区域连起来 → outlier 被吸收
```

所以 DBSCAN 不是“自动解决一切”的方法。它能处理复杂形状，但前提是：

1. 距离度量合理；
    
2. 特征尺度已标准化；
    
3. (\varepsilon) 和 minPts 设置合适；
    
4. 数据各区域密度差异不是过大。
    

---

## 43. 难点 6：为什么 DBSCAN 对 varying density 数据不好？

假设数据有两个正常 cluster：

```text
Dense cluster: 很密
Sparse cluster: 比较稀疏
```

如果选择小 (\varepsilon)：

- dense cluster 正常；
    
- sparse cluster 很多点被判 noise。
    

如果选择大 (\varepsilon)：

- sparse cluster 正常；
    
- dense cluster 可能与周围噪声连接，outlier 被吸收。
    

问题在于：

> 标准 DBSCAN 使用全局同一个 (\varepsilon)，无法同时适配不同密度区域。

---

# Part IX：考试高频概念题模板

## 44. “Extreme value 和 outlier 的区别是什么？”

答案模板：

> Extreme values are rare but still plausible observations from the primary data distribution, often lying in the tail of the normal distribution. Outliers are observations that are too unlikely under the fitted normal model or inconsistent with the main data pattern. Therefore, an extreme value is not necessarily an outlier.

---

## 45. “Gaussian outlier detection 的基本流程是什么？”

答案模板：

```text
1. Estimate mean vector and covariance matrix from normal data.
2. Compute Mahalanobis distance or Gaussian probability for each point.
3. Choose a threshold, often using chi-square distribution.
4. Points with distance above the threshold or probability below threshold are labelled as outliers.
```

---

## 46. “为什么 Gaussian model 在第一个数据集表现好，在其他数据集表现差？”

答案模板：

> Because a single Gaussian assumes the data forms one unimodal elliptical cloud. The first dataset matches this assumption. The other datasets contain concentric circles, multiple clusters, or irregular shapes, which violate the single Gaussian assumption, causing false positives and false negatives.

---

## 47. “GMM 相比 Gaussian 的改进是什么？”

答案模板：

> GMM models data as a mixture of multiple Gaussian components, allowing it to represent multi-modal distributions. It estimates component weights, means, and covariances using EM. It is more flexible than a single Gaussian but still assumes each component is Gaussian-like.

---

## 48. “EM algorithm 的 E-step 和 M-step 分别做什么？”

答案模板：

> In the E-step, EM computes the posterior probability or responsibility that each component generated each point. In the M-step, it updates the mixture weights, means, and covariance matrices using these responsibilities. The process repeats until log-likelihood converges.

---

## 49. “DBSCAN 如何用于 outlier detection？”

答案模板：

> DBSCAN defines dense regions using eps and minPts. Core points have enough neighbours within eps. Clusters are expanded from connected core points, and border points are assigned to nearby clusters. Points that are neither core points nor density-reachable from any core point are labelled as noise, which can be treated as outliers.

---

## 50. “为什么 DBSCAN 需要 normalisation？”

答案模板：

> DBSCAN is distance-based. If features have different scales, large-scale features dominate the distance calculation and distort neighbourhoods. Normalisation ensures each feature contributes comparably to the distance measure.

---

## 51. “DBSCAN 的优缺点是什么？”

答案模板：

> Advantages: It can discover arbitrary-shaped clusters, does not require specifying the number of clusters, and naturally identifies noise points as outliers.  
> Disadvantages: It is sensitive to eps and minPts, requires feature scaling, struggles with varying density, and becomes less reliable in high-dimensional spaces.

---

# Part X：计算题模板

## 52. Mahalanobis Distance 计算题

给定：

[  
x,\quad m,\quad C  
]

步骤：

```text
1. Compute x - m
2. Compute C^{-1}
3. Compute d_M^2 = (x-m)^T C^{-1}(x-m)
4. Compare d_M^2 with chi-square threshold
5. If d_M^2 > threshold, classify as outlier
```

---

## 53. GMM Responsibility 计算题

给定：

[  
\pi_1,\pi_2,\mathcal{N}_1(x),\mathcal{N}_2(x)  
]

计算：

[  
r_{i1}=  
\frac{\pi_1\mathcal{N}_1(x_i)}  
{\pi_1\mathcal{N}_1(x_i)+\pi_2\mathcal{N}_2(x_i)}  
]

[  
r_{i2}=1-r_{i1}  
]

解释：

```text
r_i1 越大，x_i 越可能来自 component 1
r_i2 越大，x_i 越可能来自 component 2
```

---

## 54. DBSCAN 判断题

给定：

```text
eps = 0.5
minPts = 4
```

对某点 (x)，如果：

```text
|N_eps(x)| = 5
```

则：

```text
x is a core point
```

如果：

```text
|N_eps(x)| = 2
但 x 在某个 core point 的 eps-neighbourhood 内
```

则：

```text
x is a border point
```

如果：

```text
|N_eps(x)| = 1
且不在任何 core point 邻域中
```

则：

```text
x is noise / outlier
```

---

# Part XI：本章对应 sol.pdf 习题重点

结合讲义 learning outcomes 和 sol.pdf，当前章节最相关的是 **Chapter 10 Anomaly Detection**。尤其建议重点看：

## 必做题

### Chapter 10 Exercise 8：3σ 与大数据异常数量

核心考点：

- 低概率事件在大数据中会大量出现；
    
- 固定 3σ threshold 不一定适合大数据；
    
- outlier threshold 与数据规模有关。
    

---

### Chapter 10 Exercise 9：log probability 与 Mahalanobis distance

核心考点：

- 多元高斯概率密度；
    
- log probability；
    
- Mahalanobis distance；
    
- 二者在异常排序上的等价性。
    

这是最贴合讲义 Gaussian detection 的计算/推导题。

---

### Chapter 10 Exercise 12：Detection rate 与 false alarm rate

核心考点：

- anomaly detection 的评估指标；
    
- 类别极不平衡下 false alarm 的反直觉现象；
    
- 99% detection 也可能有 50% false alarm among flagged anomalies。
    

---

## 巩固题

### Chapter 10 Exercise 11：K-means / relative distance outlier detection

核心考点：

- cluster-based outlier detection；
    
- relative distance；
    
- 过大的 K 可能让极端 outlier 自己成为一个 cluster，从而不再被识别为 outlier。
    

---

### Chapter 10 Exercise 13：Supervised vs Unsupervised anomaly detection

核心考点：

- 已知异常适合 supervised；
    
- 新型异常必须依赖 unsupervised；
    
- 实践中二者结合。
    

---

## 概念拓展题

### Chapter 2 Exercise 12：Noise vs Outlier

核心考点：

- noise 和 outlier 的区别；
    
- outlier 可以有意义；
    
- noise 可以造成 outlier；
    
- outlier 不一定是 noise。
    

---

# 55. 最终复习总表

|你需要记住的问题|标准答案|
|---|---|
|Outlier 是什么？|显著偏离正常数据模式的点|
|Extreme value 是 outlier 吗？|不一定，极端值可能仍来自正常分布尾部|
|Gaussian 方法如何检测异常？|拟合均值和协方差，用 Mahalanobis distance 或 probability 判断|
|Mahalanobis distance 为什么重要？|它考虑尺度和特征相关性|
|阈值怎么选？|多维 Gaussian 下 (d_M^2\sim \chi^2_d)，可用 chi-square threshold|
|Single Gaussian 何时失败？|多峰、环形、非凸、不规则数据|
|GMM 解决了什么？|用多个 Gaussian 表示多峰分布|
|EM 做什么？|E-step 算 responsibility，M-step 更新参数|
|GMM 仍然有什么问题？|仍假设 component 是 Gaussian，易受 outlier 和初始化影响|
|DBSCAN 核心思想？|dense region 是 normal，unreachable sparse points 是 outliers|
|DBSCAN 两个参数？|(\varepsilon) 和 minPts|
|DBSCAN 三类点？|core point, border point, noise point|
|DBSCAN 优点？|任意形状、不需 K、自然找 noise|
|DBSCAN 缺点？|参数敏感、需标准化、变密度困难|
|异常检测为什么不能只看 accuracy？|类别极不平衡，全部预测 normal 也可能高 accuracy|
|Supervised 和 unsupervised 怎么选？|已知异常用 supervised，新型异常用 unsupervised，实践中结合|

---

# 56. 一句话终极总结

**Outlier Detection 的本质不是“找最大最小值”，而是先定义正常数据的生成模式，再找那些在该模式下过于不可能、距离过远、密度过低或无法被正常簇解释的数据点。Gaussian 方法适合单峰椭圆数据，GMM 适合多峰椭圆数据，DBSCAN 适合任意形状密度簇；但每种方法都依赖自己的假设，考试最常考的就是这些假设、适用场景、失败场景和阈值判断。**