# COMP6237 Data Mining 最终复习笔记：Information Theory and Feature Selection

下面这份笔记围绕 **Information Theory and Feature Selection（信息论与特征选择）** 重构。课程 slides 的主线是：**information（信息）→ entropy/coding（熵与编码）→ Kullback-Leibler divergence（KL 散度）→ cross entropy（交叉熵）→ conditional entropy（条件熵）→ mutual information（互信息）→ feature selection（特征选择）**。这也正好对应历年卷中 Information Theory and Feature Selection / Feature Extraction 的考试重点。

---

## 1. 这章到底在学什么？

**Information Theory（信息论）** 在这门 Data Mining 里的作用不是通信系统本身，而是提供一种量化方法：

> 一个事件出现以后，它到底给了我们多少“信息”？  
> 一个特征被观察到以后，它到底减少了多少对类别的不确定性？  
> 一个模型分布和真实分布差多少？  
> 哪些特征最值得保留，哪些特征是冗余的？

所以本章核心可以压缩成一句话：

**信息论把“有用性”“不确定性”“惊讶程度”“分布差异”“特征重要性”变成可以计算的数学量。**

在 Data Mining 中，这些量主要用于：

1. **衡量一个分布本身的不确定性**：Entropy。
    
2. **衡量两个分布的差异**：KL divergence。
    
3. **衡量用错误模型编码真实数据的代价**：Cross entropy。
    
4. **衡量一个特征对类别的帮助**：Mutual information。
    
5. **选择有用但不冗余的特征**：Conditional mutual information。
    

---

# 2. Information：信息是什么？

## 2.1 直觉：信息 = 找到答案需要多少 yes/no questions

课程最开始用的是 **chessboard example（棋盘例子）**。假设一个棋子在 $8 \times 8$ 棋盘上的某个格子里，你一开始不知道它在哪里。棋盘共有 $64$ 个格子。如果每次只能问一个 **yes/no question（是/否问题）**，最有效的方法是每次把剩余区域一分为二，例如先问“它在左半边吗？”，再问“它在上半边吗？”不断二分，直到定位到唯一格子。slides 中明确说，64 个格子需要的问题数量是 $\log_2 64 = 6$。

这说明：

$$  
\text{information} = \text{minimum number of binary questions needed to identify the outcome}  
$$

如果有 $N$ 个等概率可能结果，那么知道真实结果的信息量是：

$$  
\log_2 N  
$$

因为每个 yes/no question 最多把可能性减半，所以问 $k$ 个问题最多能区分：

$$  
2^k  
$$

个结果。要区分 $N$ 个结果，就需要：

$$  
2^k = N  
$$

所以：

$$  
k = \log_2 N  
$$

棋盘例子里：

$$  
N = 64  
$$

$$  
k = \log_2 64 = 6  
$$

因此，知道棋子具体在哪一格提供了 **6 bits** 的信息。

---

## 2.2 从棋盘推广到概率空间

如果所有结果不再等概率，就不能只看“有多少个可能结果”，还要看每个结果本身的概率。

假设有一个概率空间 $W$，每个结果 $w$ 有概率 $P(w)$。如果某个结果 $w$ 被观察到了，那么这个观察结果带来的信息量取决于它有多“意外”。

**Self-information（自信息量）** 定义为：

$$  
I(w) = \log_2 \frac{1}{P(w)}  
$$

也可以写成：

$$  
I(w) = -\log_2 P(w)  
$$

这个公式非常重要。

### 为什么是 $\log_2 \frac{1}{P(w)}$？

如果某个事件概率是 $P(w)$，它大概相当于在：

$$  
\frac{1}{P(w)}  
$$

个等概率候选中选中了其中一个。

所以信息量是：

$$  
\log_2 \frac{1}{P(w)}  
$$

例如：

如果某事件概率是：

$$  
P(w)=\frac{1}{64}  
$$

那么：

$$  
I(w)=\log_2 \frac{1}{1/64}  
$$

$$  
I(w)=\log_2 64  
$$

$$  
I(w)=6  
$$

这就和棋盘例子完全一致。

---

## 2.3 稀有事件为什么信息量更大？

如果一个事件很常见，例如：

$$  
P(x)=\frac{1}{2}  
$$

那么：

$$  
I(x)=\log_2 2 = 1  
$$

它只提供 1 bit 信息。

如果一个事件很稀有，例如：

$$  
P(x)=\frac{1}{8}  
$$

那么：

$$  
I(x)=\log_2 8 = 3  
$$

它提供 3 bits 信息。

所以：

**Rare event（稀有事件）更难被猜中，因此一旦发生，信息量更大。**

这也可以理解为 **surprise（惊讶程度）**：

- 常见事件发生：不惊讶，信息少。
    
- 稀有事件发生：很惊讶，信息多。
    

slides 和 lecture transcript 中都强调：rare event carries more information / causes more surprise。

---

# 3. A/B/C/D 随机信号例子：不等概率事件的信息量

课程中第二个核心例子是一个随机源发出符号 $A,B,C,D$：

$$  
P(A)=\frac{1}{2}  
$$

$$  
P(B)=\frac{1}{4}  
$$

$$  
P(C)=\frac{1}{8}  
$$

$$  
P(D)=\frac{1}{8}  
$$

这些概率加起来是：

$$  
\frac{1}{2}+\frac{1}{4}+\frac{1}{8}+\frac{1}{8}=1  
$$

## 3.1 最优 yes/no questions

因为 $A$ 最常见，所以最优策略应该先问：

> Is it A?

如果是 $A$，只要问 1 个问题就结束。

如果不是 $A$，再问：

> Is it B?

如果是 $B$，一共问 2 个问题。

如果不是 $A$ 也不是 $B$，再问：

> Is it C?

如果是 $C$，一共问 3 个问题。  
如果不是 $C$，那只能是 $D$，也相当于 3 个问题。

因此：

|Symbol|Probability|Questions needed|Information|
|---|--:|--:|--:|
|$A$|$1/2$|$1$|$1$ bit|
|$B$|$1/4$|$2$|$2$ bits|
|$C$|$1/8$|$3$|$3$ bits|
|$D$|$1/8$|$3$|$3$ bits|

公式验证：

$$  
I(A)=\log_2 \frac{1}{1/2}=\log_2 2=1  
$$

$$  
I(B)=\log_2 \frac{1}{1/4}=\log_2 4=2  
$$

$$  
I(C)=\log_2 \frac{1}{1/8}=\log_2 8=3  
$$

$$  
I(D)=\log_2 \frac{1}{1/8}=\log_2 8=3  
$$

slides 中也给出了这个例子：$A$ 需要 $1$ bit，$B$ 需要 $2$ bits，$C,D$ 需要 $3$ bits。

---

## 3.2 从 yes/no questions 到 binary code

这个例子也可以理解成 **binary coding（二进制编码）**。

一种可行编码是：

$$  
A \rightarrow 1  
$$

$$  
B \rightarrow 01  
$$

$$  
C \rightarrow 001  
$$

$$  
D \rightarrow 000  
$$

为什么这样编码？

因为每一位可以看成一个 yes/no question 的回答：

- $1$：第一个问题 “Is it A?” 的回答是 yes。
    
- $01$：第一个问题回答 no，第二个问题 “Is it B?” 回答 yes。
    
- $001$：不是 $A$，不是 $B$，是 $C$。
    
- $000$：不是 $A$，不是 $B$，也不是 $C$，所以是 $D$。
    

这体现了一个很重要的编码原则：

**Frequent symbols get short codes, infrequent symbols get long codes.**

也就是：

- 高频符号：短编码。
    
- 低频符号：长编码。
    

课程 slides 中也明确说，entropy 与 optimal code length 有关，frequent symbols get short codes, infrequent symbols get long codes。

---

# 4. Entropy：熵

## 4.1 Entropy 的核心定义

如果 **self-information** 是某个具体结果 $x$ 的信息量，那么 **entropy（熵）** 是整个概率分布的平均信息量。

对于离散随机变量 $X$，其概率分布是 $p(x)$，entropy 定义为：

$$  
H(X) = -\sum_x p(x)\log_2 p(x)  
$$

也可以写成：

$$  
H(X)=\sum_x p(x)\log_2 \frac{1}{p(x)}  
$$

这两个式子完全等价，因为：

$$  
\log_2 \frac{1}{p(x)}=-\log_2 p(x)  
$$

所以：


$$  
\sum_x p(x)\log_2 \frac{1}{p(x)}

-\sum_x p(x)\log_2 p(x)  
$$

## 4.2 Entropy 的直觉

**Entropy = expected information = average surprise = uncertainty.**

也就是说：

$$  
H(X)=E[I(X)]  
$$

其中：

$$  
I(x)=-\log_2 p(x)  
$$

所以：

$$  
H(X)=\sum_x p(x)I(x)  
$$

代入：

$$  
H(X)=\sum_x p(x)(-\log_2 p(x))  
$$

得到：

$$  
H(X)=-\sum_x p(x)\log_2 p(x)  
$$

slides 中明确说，entropy is the expectation of information / average surprise。

---

## 4.3 Entropy 和 Information 的区别

这是很容易混淆的地方。

### Information / self-information

针对 **某一个具体事件**：

$$  
I(x)=-\log_2 p(x)  
$$

例如观察到 $C$：

$$  
I(C)=3  
$$

### Entropy

针对 **整个分布**：

$$  
H(X)=\sum_x p(x)I(x)  
$$

例如 $A,B,C,D$ 的平均信息量：

$$  
H(X)=P(A)I(A)+P(B)I(B)+P(C)I(C)+P(D)I(D)  
$$

代入：

$$  
H(X)=\frac{1}{2}\cdot 1+\frac{1}{4}\cdot 2+\frac{1}{8}\cdot 3+\frac{1}{8}\cdot 3  
$$

$$  
H(X)=0.5+0.5+0.375+0.375  
$$

$$  
H(X)=1.75  
$$

所以这个随机源的 entropy 是：

$$  
H(X)=1.75 \text{ bits}  
$$

解释：

- 每次看到一个符号，具体信息量可能是 1、2、3 bits。
    
- 但长期平均下来，每个符号需要大约 1.75 bits。
    

---

# 5. Unfair Coin：不公平硬币的 entropy

课程里用不公平硬币说明 entropy 的形状。

假设硬币正面概率是 $p$，反面概率是 $1-p$。

那么 entropy 是：

$$  
H(X)=-p\log_2 p-(1-p)\log_2(1-p)  
$$

如果用自然对数 $\ln$，公式写成：

$$  
H(X)=-p\ln p-(1-p)\ln(1-p)  
$$

区别只是单位不同：

- 用 $\log_2$：单位是 **bits**。
    
- 用 $\ln$：单位是 **nats**。
    

考试里一般给 $\log_2$ 表，所以更常用 bits。

---

## 5.1 当 $p=0$ 或 $p=1$ 时：entropy = 0

如果：

$$  
p=1  
$$

那结果一定是 heads，没有任何不确定性。

此时：

$$  
H(X)=0  
$$

如果：

$$  
p=0  
$$

那结果一定是 tails，也没有任何不确定性。

此时：

$$  
H(X)=0  
$$

直觉是：

> 如果我已经知道一定会发生什么，那么观察结果不会给我任何 surprise。

---

## 5.2 当 $p=0.5$ 时：entropy 最大

如果：

$$  
p=0.5  
$$

那么正反面概率相等，最难预测。

此时：

$$  
H(X)=-0.5\log_2 0.5-0.5\log_2 0.5  
$$

因为：

$$  
\log_2 0.5=-1  
$$

所以：

$$  
H(X)=0.5+0.5=1  
$$

最大 entropy 是：

$$  
1 \text{ bit}  
$$

这说明：

**二元变量最大不确定性发生在两个结果等概率时。**

---

## 5.3 为什么最大值在 $p=0.5$？

对：

$$  
H(p)=-p\log_2 p-(1-p)\log_2(1-p)  
$$

求导，得到：

$$  
H'(p)=\log_2\frac{1-p}{p}  
$$

令：

$$  
H'(p)=0  
$$

得到：

$$  
\log_2\frac{1-p}{p}=0  
$$

所以：

$$  
\frac{1-p}{p}=1  
$$

因此：

$$  
p=0.5  
$$

再看二阶导数：

$$  
H''(p)<0  
$$

所以 $p=0.5$ 是最大值。

---

# 6. Entropy 和 Coding：为什么频繁词短编码，稀有词长编码？

Entropy 还可以理解为 **最优编码的平均长度下界**。

如果一个数据流不断发出符号，每个符号按照分布 $p(x)$ 出现，那么我们希望设计一个编码系统，使得消息整体编码尽可能短。

一个符号 $x$ 的理想码长大约是：

$$  
-\log_2 p(x)  
$$

也就是：

$$  
\log_2 \frac{1}{p(x)}  
$$

所以：

- 如果 $p(x)$ 大，$-\log_2 p(x)$ 小，码长短。
    
- 如果 $p(x)$ 小，$-\log_2 p(x)$ 大，码长长。
    

这就是：

**Frequent symbols get short codes, infrequent symbols get long codes.**

---

# 7. Newspaper / Bag-of-Words 例子：信息论如何分析文本？

课程中用 New York Times 文章做例子：假设从 2004 年和 2005 年的报纸中抽取所有词，并用 **bag of words（词袋模型）** 统计每个词的频率。这样可以把一篇文章或一期报纸表示成一个概率向量。slides 中说明这种方法能把词频解释成 probability vector，但它会忽略词之间的相关性。

## 7.1 Bag-of-words 如何变成概率分布？

假设某一期报纸中所有词总数是 $N$，某个词 $w$ 出现次数是 $n_w$，那么：

$$  
P(w)=\frac{n_w}{N}  
$$

这样，每个词都有一个概率。

所有词的概率加起来：

$$  
\sum_w P(w)=1  
$$

于是我们可以计算这一期报纸的 entropy：

$$  
H(P)=-\sum_w P(w)\log_2 P(w)  
$$

课程 slides 中给出示例结果：

$$  
\sum_x P(x)\log\frac{1}{P(x)}=12.94 \text{ bits}  
$$

$$  
\sum_x Q(x)\log\frac{1}{Q(x)}=12.77 \text{ bits}  
$$

---

## 7.2 “the / to / of / a” 这类词为什么信息量低？

像：

- the
    
- to
    
- of
    
- a
    
- and
    
- in
    

这些词在英文文章中非常常见。它们出现概率高，所以：

$$  
-\log_2 P(w)  
$$

较小。

这类词通常称为 **non-content words（非内容词 / 停用词）**。

它们有两个特点：

1. 出现频繁，所以编码所需 bits 较少。
    
2. 它们在不同年份、不同主题中比例都比较稳定，所以对区分主题帮助不大。
    

因此，虽然它们在文本里出现很多次，但它们通常不是有意义的分类特征。

---

## 7.3 高频词、低频词和 topic information 的区别

注意：

**一个词信息量大，不等于它一定适合分类。**

例如某个很稀有的词，因为 $P(w)$ 很小，所以：

$$  
-\log_2 P(w)  
$$

很大。

但它可能只出现一次，统计不稳定，不一定能可靠区分类别。

真正用于分类时，我们更关心：

> 这个词是否能减少我们对 class label 的不确定性？

这就引出后面的 **mutual information（互信息）**。

---

# 8. Kullback-Leibler Divergence：KL 散度

## 8.1 KL divergence 的定义

给定两个定义在同一随机变量 $X$ 上的概率分布：

$$  
f(x)  
$$

和：

$$  
g(x)  
$$

**Kullback-Leibler divergence（KL 散度 / 相对熵）** 定义为：

$$  
D_{KL}(f||g)=\sum_{x\in X} f(x)\log\frac{f(x)}{g(x)}  
$$

如果使用 $\log_2$，单位是 bits。

slides 中给出的解释是：KL divergence 比较同一个随机变量上的两个分布，可以被理解为当真实分布是 $f(x)$，却使用适合 $g(x)$ 的编码时，需要的额外 bits。

---

## 8.2 KL divergence 的直觉：用错误编码会浪费多少 bits

假设真实数据来自分布 $f$，但你误以为它来自分布 $g$，于是你使用针对 $g$ 最优的编码方式。

那么平均码长会比最优情况更长。

这个额外长度就是：

$$  
D_{KL}(f||g)  
$$

所以：

$$  
D_{KL}(f||g)

\text{extra average code length}  
$$

也就是：

> true distribution 是 $f$，model distribution 是 $g$，KL 散度衡量 $g$ 作为模型有多差。

---

## 8.3 KL divergence 和 newspaper 例子

假设：

- $P(x)$ 是 2005 年文章的词分布。
    
- $Q(x)$ 是 2004 年文章的词分布。
    

如果我们用 2004 年的编码方式去编码 2005 年文章，那么平均码长是：

$$  
H(P,Q)=\sum_x P(x)\log\frac{1}{Q(x)}  
$$

如果我们用 2005 年自己的最优编码方式，平均码长是：

$$  
H(P)=\sum_x P(x)\log\frac{1}{P(x)}  
$$

二者差值是：

$$  
H(P,Q)-H(P)  
$$

展开：
$$  
H(P,Q)-H(P)

## \sum_x P(x)\log\frac{1}{Q(x)}

\sum_x P(x)\log\frac{1}{P(x)}  
$$

合并：

$$

\sum_x P(x)  
\left[  
\log\frac{1}{Q(x)}-\log\frac{1}{P(x)}  
\right]  
$$

根据对数法则：

$$  
\log\frac{1}{Q(x)}-\log\frac{1}{P(x)}

\log\frac{P(x)}{Q(x)}  
$$

所以：

$$  
H(P,Q)-H(P)=

\sum_x P(x)\log\frac{P(x)}{Q(x)}  
$$

这正是：

$$  
D_{KL}(P||Q)  
$$

slides 的 newspaper 例子中也通过 “用 2004 code 编码 2005 paper” 和 “用 2005 frequencies 编码” 的平均码长差引出 KL divergence。

---

## 8.4 KL divergence 为什么不是严格的 distance？

考试常问：

> Can KL divergence be used as a distance measure?

严格回答：

**KL divergence can measure distributional discrepancy, but it is not a proper metric distance.**

原因有三个。

### 原因 1：KL 不对称

一般情况下：

$$  
D_{KL}(f||g)\neq D_{KL}(g||f)  
$$

例如，真实分布 $f$ 用模型 $g$ 编码的代价，不等于真实分布 $g$ 用模型 $f$ 编码的代价。

而真正的距离通常要求：

$$  
d(f,g)=d(g,f)  
$$

KL 不满足这一点。

### 原因 2：KL 不满足三角不等式

真正的 metric distance 要满足：

$$  
d(f,h)\leq d(f,g)+d(g,h)  
$$

KL divergence 一般不满足。

### 原因 3：KL 可能是无穷大

如果存在某个 $x$：

$$  
f(x)>0  
$$

但：

$$  
g(x)=0  
$$

那么：

$$  
\log\frac{f(x)}{g(x)}  
$$

会发散，导致：

$$  
D_{KL}(f||g)=\infty  
$$

直觉是：

> 真实数据中某事件会发生，但模型认为它绝不可能发生。  
> 那么模型会受到无限惩罚。

因此，考试中如果问 KL 是否是 distance，推荐回答：

> KL divergence measures how inefficient it is to encode data from $f$ using a code optimized for $g$. It is a divergence or discrepancy measure, but not a true metric distance because it is asymmetric and does not generally satisfy the triangle inequality.

---

# 9. Cross Entropy：交叉熵

## 9.1 Cross entropy 的定义

如果真实分布是 $p(x)$，但我们用模型分布 $q(x)$ 的编码方式来编码数据，那么平均编码长度是：

$$  
H(p,q)=-\sum_x p(x)\log_2 q(x)  
$$

这就是 **cross entropy（交叉熵）**。

它和 entropy 的区别是：

### Entropy

真实分布 $p$，也用 $p$ 自己的最优编码：

$$  
H(p)=-\sum_x p(x)\log_2 p(x)  
$$

### Cross entropy

真实分布 $p$，但用 $q$ 的编码：

$$  
H(p,q)=-\sum_x p(x)\log_2 q(x)  
$$

slides 中定义 cross entropy 时也是这个思路：measure the information content of $p(x)$ based on a code optimal for artificial $q(x)$。

---

## 9.2 Cross entropy 和 KL divergence 的关系

核心公式：

$$  
H(p,q)=H(p)+D_{KL}(p||q)  
$$

证明如下。

从 KL 出发：

$$  
D_{KL}(p||q)=\sum_x p(x)\log_2\frac{p(x)}{q(x)}  
$$

展开：

$$  
D_{KL}(p||q)=\sum_x p(x)\left[\log_2 p(x)-\log_2 q(x)\right]  
$$

$$  
D_{KL}(p||q)=\sum_x p(x)\log_2 p(x)-\sum_x p(x)\log_2 q(x)  
$$

因为：

$$  
H(p)=-\sum_x p(x)\log_2 p(x)  
$$

所以：

$$  
\sum_x p(x)\log_2 p(x)=-H(p)  
$$

又因为：

$$  
H(p,q)=-\sum_x p(x)\log_2 q(x)  
$$

所以：

$$  
D_{KL}(p||q)=-H(p)+H(p,q)  
$$

移项：

$$  
H(p,q)=H(p)+D_{KL}(p||q)  
$$

这个公式的含义非常重要：

> Cross entropy = intrinsic uncertainty of data + penalty for using wrong model.

也就是：

- $H(p)$ 是真实数据本身不可避免的不确定性。
    
- $D_{KL}(p||q)$ 是模型 $q$ 和真实分布 $p$ 不一致造成的额外损失。
    

---

## 9.3 为什么最小化 cross entropy 等价于最小化 KL？

如果训练数据固定，真实经验分布 $p$ 是固定的，那么：

$$  
H(p)  
$$

是常数。

因此最小化：

$$  
H(p,q)  
$$

等价于最小化：

$$  
D_{KL}(p||q)  
$$

因为：

$$  
H(p,q)=H(p)+D_{KL}(p||q)  
$$

其中 $H(p)$ 不随模型 $q$ 改变。

所以模型训练中常说：

> minimizing cross entropy is equivalent to minimizing KL divergence from data distribution to model distribution.

slides 中也提到，在固定 reference distribution $p$ 时，minimizing KL is equivalent to minimizing cross entropy。

---

## 9.4 Cross entropy 和 Maximum Likelihood Estimation 的关系

假设训练集中 outcome $i$ 的经验频率是：

$$  
p_i  
$$

总样本数是：

$$  
N  
$$

那么 outcome $i$ 出现次数是：

$$  
Np_i  
$$

模型给 outcome $i$ 的概率是：

$$  
q_i  
$$

则 likelihood 是：

$$  
L(q)=\prod_i q_i^{Np_i}  
$$

取 log：

$$  
\log L(q)=\sum_i Np_i\log q_i  
$$

除以 $N$：

$$  
\frac{1}{N}\log L(q)=\sum_i p_i\log q_i  
$$

最大化 log-likelihood 等价于最大化：

$$  
\sum_i p_i\log q_i  
$$

也等价于最小化负号版本：

$$  
-\sum_i p_i\log q_i  
$$

而这正是 cross entropy：

$$  
H(p,q)=-\sum_i p_i\log q_i  
$$

所以：

**Maximum likelihood estimation = minimizing cross entropy = minimizing KL divergence to empirical distribution.**

这个联系是本章和 regression / classification training loss 之间的关键桥梁。

---

# 10. Conditional Entropy：条件熵

## 10.1 条件熵的定义

假设有两个随机变量：

- $C$：class label，例如是否下雨、文章主题、书籍类别。
    
- $X$：feature，例如是否 cloudy、某个词是否出现。
    

如果观察到：

$$  
X=x  
$$

那么我们对 $C$ 的不确定性变成：

$$  
H(C|X=x)  
$$

定义为：

$$  
H(C|X=x)=-\sum_c P(C=c|X=x)\log_2 P(C=c|X=x)  
$$

如果不固定某个 $x$，而是对所有 $x$ 加权平均，则：

$$  
H(C|X)=\sum_x P(X=x)H(C|X=x)  
$$

也可以写成：

$$  
H(C|X)=-\sum_x\sum_c P(X=x,C=c)\log_2 P(C=c|X=x)  
$$

---

## 10.2 条件熵的直觉

$H(C)$ 表示：

> 没有观察任何 feature 时，我们对 class $C$ 的不确定性。

$H(C|X=x)$ 表示：

> 已经观察到 $X=x$ 后，我们对 $C$ 的不确定性。

如果观察 $X=x$ 后更确定了，那么：

$$  
H(C|X=x)<H(C)  
$$

如果观察 $X=x$ 后反而更迷惑了，那么：

$$  
H(C|X=x)>H(C)  
$$

这就引出 **realized information（实现信息量 / 已实现信息）**。

---

# 11. Realized Information：单次观察带来的信息量

## 11.1 定义

观察到某个具体值 $X=x$ 后，关于 $C$ 的 realized information 是：

$$  
I(C;X=x)=H(C)-H(C|X=x)  
$$

它表示：

> 观察到 $X=x$ 以后，对 $C$ 的不确定性减少了多少。

如果：

$$  
I(C;X=x)>0  
$$

说明观察 $x$ 后不确定性减少。

如果：

$$  
I(C;X=x)=0  
$$

说明观察 $x$ 没有改变不确定性。

如果：

$$  
I(C;X=x)<0  
$$

说明观察 $x$ 后不确定性反而增加。

slides 和 transcript 都强调：realized information is not necessarily positive。

---

## 11.2 Rain / Cloudy 例子：为什么 realized information 可以是负数？

课程和历年卷都反复考这个例子。2015/16 年试卷直接问：平均 7 天中 1 天下雨，但如果有云，平均每 2 天中 1 天下雨；观察到 clouds 后 realized information 是多少，负值意味着什么。

设：

$$  
C=\text{it rains today}  
$$

已知：

$$  
P(C=\text{rain})=\frac{1}{7}  
$$

$$  
P(C=\text{no rain})=\frac{6}{7}  
$$

所以：

$$  
H(C)=-\frac{1}{7}\log_2\frac{1}{7}-\frac{6}{7}\log_2\frac{6}{7}  
$$

计算约为：

$$  
H(C)\approx 0.59 \text{ bits}  
$$

现在观察到：

$$  
X=\text{cloudy}  
$$

并且：

$$  
P(C=\text{rain}|X=\text{cloudy})=\frac{1}{2}  
$$

$$  
P(C=\text{no rain}|X=\text{cloudy})=\frac{1}{2}  
$$

所以：

$$  
H(C|X=\text{cloudy})

-\frac{1}{2}\log_2\frac{1}{2}  
-\frac{1}{2}\log_2\frac{1}{2}  
$$

$$  
H(C|X=\text{cloudy})=1  
$$

因此 realized information 是：

$$  
I(C;X=\text{cloudy})

H(C)-H(C|X=\text{cloudy})  
$$

$$  
I(C;X=\text{cloudy})

0.59-1  
$$

$$  
I(C;X=\text{cloudy})

-0.41 \text{ bits}  
$$

---

## 11.3 负 realized information 的含义

负值不是说“云没有信息”，而是说：

> 在没有观察云之前，下雨本来是低概率事件，所以我们比较确定今天大概率不下雨。  
> 观察到云以后，下雨概率变成 $1/2$，这反而让我们更不确定了。

原来：

$$  
P(\text{rain})=\frac{1}{7}  
$$

很偏，entropy 较低。

观察云之后：

$$  
P(\text{rain}|\text{cloudy})=\frac{1}{2}  
$$

变成最不确定的二元分布，entropy 变高。

所以 realized information 为负。

这也是考试容易出的概念题：

**Realized information can be negative because a particular observation can increase uncertainty.**

---

# 12. Mutual Information：互信息

## 12.1 Mutual information 的定义

**Mutual information（互信息）** 衡量一个随机变量平均能告诉我们另一个随机变量多少信息。

在特征选择中，我们通常关心：

$$  
I(C;X)  
$$

也就是：

> feature $X$ 平均能减少多少关于 class $C$ 的不确定性？

定义：

$$  
I(C;X)=H(C)-H(C|X)  
$$

展开：

$$  
I(C;X)=H(C)-\sum_x P(X=x)H(C|X=x)  
$$

slides 中也给出这个定义：mutual information is the expected information a feature gives us about a class。

---

## 12.2 Mutual information 是 realized information 的期望

前面 realized information 是：

$$  
I(C;X=x)=H(C)-H(C|X=x)  
$$

这是观察到某个具体 $x$ 后的信息变化。

Mutual information 是对所有可能 $x$ 做期望：

$$  
I(C;X)=\sum_x P(X=x)I(C;X=x)  
$$

代入：

$$  
I(C;X)=\sum_x P(X=x)\left[H(C)-H(C|X=x)\right]  
$$

因为 $H(C)$ 与 $x$ 无关：

$$  
I(C;X)=H(C)\sum_x P(X=x)-\sum_x P(X=x)H(C|X=x)  
$$

又因为：

$$  
\sum_x P(X=x)=1  
$$

所以：

$$  
I(C;X)=H(C)-\sum_x P(X=x)H(C|X=x)  
$$

这就是互信息公式。

---

## 12.3 Mutual information 的三个重要性质

### 性质 1：互信息非负

$$  
I(C;X)\geq 0  
$$

这和 realized information 不同。

**Realized information 可以为负。**  
**Mutual information 作为平均量不会为负。**

这点非常重要，因为历年题里出现过“negative mutual information”的表述，但标准信息论中普通 pairwise mutual information 不可能为负。slides 和 transcript 也明确说 mutual information is always positive / zero if statistically independent。

更严谨地说：

$$  
I(C;X)\geq 0  
$$

并且：

$$  
I(C;X)=0  
$$

当且仅当 $C$ 和 $X$ 独立。

### 性质 2：独立时互信息为 0

如果：

$$  
P(C,X)=P(C)P(X)  
$$

则：

$$  
I(C;X)=0  
$$

直觉：

> 如果知道 $X$ 不会改变我们对 $C$ 的判断，那么 $X$ 对 $C$ 没有信息。

### 性质 3：互信息对称

$$  
I(C;X)=I(X;C)  
$$

也就是说：

> $X$ 告诉我们多少关于 $C$ 的信息，等价于 $C$ 告诉我们多少关于 $X$ 的信息。

另一个常见公式是：

$$  
I(X;Y)=H(X)+H(Y)-H(X,Y)  
$$

这个公式天然对称。

---

# 13. 普通 Mutual Information 能不能为负？

这是你复习时一定要记住的陷阱。

## 13.1 标准答案：不能

普通两变量互信息：

$$  
I(X;Y)  
$$

不能为负。

因为：

$$  
I(X;Y)=D_{KL}(P(X,Y)||P(X)P(Y))  
$$

而 KL divergence 总是非负：

$$  
D_{KL}(p||q)\geq 0  
$$

所以：

$$  
I(X;Y)\geq 0  
$$

## 13.2 那为什么题目会问 negative mutual information？

历年题中曾经要求给出 mutual information 为 negative 的情况。严格来说，这个要求对 ordinary mutual information 是不正确的。更合理的解释有两种：

### 情况 1：题目其实想问 realized information

realized information：

$$  
I(C;X=x)=H(C)-H(C|X=x)  
$$

可以为负。

例如 clouds / rain 例子中：

$$  
I(C;X=\text{cloudy})=-0.41  
$$

### 情况 2：题目其实想问 interaction information

三变量中的 **interaction information（交互信息）** 可以为负：

$$  
I(X;Y;Z)=I(X;Y)-I(X;Y|Z)  
$$

如果：

$$  
I(X;Y|Z)>I(X;Y)  
$$

那么：

$$  
I(X;Y;Z)<0  
$$

这通常表示 **synergy（协同信息）**：两个变量单独看没什么用，但结合起来很有用。

最经典例子是 XOR：

- $X$ 和 $Y$ 独立。
    
- $Z=XOR(X,Y)$。
    
- 单独知道 $X$ 或 $Y$ 无法确定 $Z$。
    
- 同时知道 $X$ 和 $Y$ 可以完全确定 $Z$。
    

所以：

$$  
I(Z;X)=0  
$$

但：

$$  
I(Z;X|Y)>0  
$$

于是：

$$  
I(Z;X;Y)=I(Z;X)-I(Z;X|Y)<0  
$$

考试答题建议：

> Ordinary mutual information cannot be negative. If the question refers to realized information or interaction information, negative values are possible.

---

# 14. Words / Topics / Information：paint 例子完整解析

课程中用词 “paint” 来判断文章是 art 还是 music。slides 中给出的表格是：

|Class $C$|$X=\text{paint}$|$X=\text{not paint}$|
|---|--:|--:|
|art|12|45|
|music|0|45|

总文章数：

$$  
N=12+45+0+45=102  
$$

art 总数：

$$  
57  
$$

music 总数：

$$  
45  
$$

---

## 14.1 先算 class entropy $H(C)$

$$  
P(C=\text{art})=\frac{57}{102}  
$$

$$  
P(C=\text{music})=\frac{45}{102}  
$$

所以：

$$  
H(C)=

-\frac{57}{102}\log_2\frac{57}{102}  
-\frac{45}{102}\log_2\frac{45}{102}  
$$

slides 中给出近似：

$$  
H(C)=0.99  
$$

---

## 14.2 观察到 paint 时的条件熵

如果：

$$  
X=\text{paint}  
$$

那么表中有：

- art: 12
    
- music: 0
    

所以：

$$  
P(\text{art}|X=\text{paint})=1  
$$

$$  
P(\text{music}|X=\text{paint})=0  
$$

因此：

$$  
H(C|X=\text{paint})=0  
$$

直觉：

> 如果出现 paint，就可以确定文章是 art。

---

## 14.3 观察到 not paint 时的条件熵

如果：

$$  
X=\text{not paint}  
$$

那么：

- art: 45
    
- music: 45
    

所以：

$$  
P(\text{art}|X=\text{not paint})=\frac{1}{2}  
$$

$$  
P(\text{music}|X=\text{not paint})=\frac{1}{2}  
$$

因此：

$$  
H(C|X=\text{not paint})=1  
$$

直觉：

> 如果没有 paint，art 和 music 各一半，像 fair coin 一样不确定。

---

## 14.4 计算 mutual information

先算：

$$  
P(X=\text{paint})=\frac{12}{102}  
$$

$$  
P(X=\text{not paint})=\frac{90}{102}  
$$

所以：

$$  
I(C;X)=

H(C)-P(X=\text{paint})H(C|X=\text{paint})

-P(X=\text{not paint})H(C|X=\text{not paint})  
$$

代入：

$$  
I(C;X)

= 0.99

- \frac{12}{102}\cdot 0

-\frac{90}{102}\cdot 1  
$$

$$  
I(C;X)

0.99-0.882  
$$

$$  
I(C;X)\approx 0.11  
$$

slides 中给出的结果也是：

$$  
I(C;X)=0.11  
$$

---

## 14.5 为什么 paint 看起来很强，但 MI 只有 0.11？

这是这道例子的重点。

当 paint 出现时，它确实非常强：

$$  
H(C|X=\text{paint})=0  
$$

但是 paint 出现概率很低：

$$  
P(X=\text{paint})=\frac{12}{102}  
$$

大多数文章没有 paint：

$$  
P(X=\text{not paint})=\frac{90}{102}  
$$

而在 not paint 情况下，条件熵是：

$$  
H(C|X=\text{not paint})=1  
$$

几乎没有帮助。

所以平均下来，paint 的互信息不大。

这说明：

**一个特征在某种取值下很有用，不代表它整体 mutual information 一定高。**

Mutual information 看的是：

> 按 feature 所有可能取值加权平均后，它平均减少多少不确定性。

---

# 15. Joint Entropy：联合熵

## 15.1 定义

如果有两个随机变量 $X,Y$，它们的 joint entropy 是：

$$  
H(X,Y)

-\sum_x\sum_y P(X=x,Y=y)\log_2 P(X=x,Y=y)  
$$

它衡量：

> 同时描述 $X$ 和 $Y$ 的总不确定性。

## 15.2 Joint entropy 和 individual entropy 的关系

一般有：

$$  
H(X,Y)\leq H(X)+H(Y)  
$$

如果 $X,Y$ 独立，那么：

$$  
H(X,Y)=H(X)+H(Y)  
$$

如果它们相关，那么二者有重叠信息，所以：

$$  
H(X,Y)<H(X)+H(Y)  
$$

这个“重叠部分”就是 mutual information。

---

# 16. Conditional Entropy 的另一种写法

## 16.1 二变量条件熵

$$  
H(X|Y)=H(X,Y)-H(Y)  
$$

也可以写成：

$$  
H(X|Y)

-\sum_x\sum_y P(X=x,Y=y)\log_2 P(X=x|Y=y)  
$$

直觉：

> $H(X,Y)$ 是同时描述 $X,Y$ 需要的信息量。  
> 如果已经知道 $Y$，还需要额外多少信息描述 $X$？  
> 这个额外量就是 $H(X|Y)$。
> ![[Pasted image 20260512172541.png]]

---

## 16.2 Mutual information 的 Venn diagram 公式

对于两个随机变量：

$$  
I(X;Y)=H(X)-H(X|Y)  
$$

也可以写成：

$$  
I(X;Y)=H(Y)-H(Y|X)  
$$

也可以写成：

$$  
I(X;Y)=H(X)+H(Y)-H(X,Y)  
$$

Venn diagram 里：

- 左圆：$H(X)$
    
- 右圆：$H(Y)$
    
- 重叠部分：$I(X;Y)$
    
- 左边不重叠部分：$H(X|Y)$
    
- 右边不重叠部分：$H(Y|X)$
    
- 两个圆总面积：$H(X,Y)$
    

slides 后面专门用 Venn diagrams 来帮助记忆这些关系。

---

# 17. Conditional Mutual Information：条件互信息

## 17.1 定义

**Conditional mutual information（条件互信息）** 衡量：

> 在已经知道 $Z$ 的情况下，$X$ 还能告诉我们多少关于 $Y$ 的信息？

定义为：

$$  
I(X;Y|Z)=H(X|Z)-H(X|Y,Z)  
$$

也可以写成：

$$  
I(X;Y|Z)=H(Y|Z)-H(Y|X,Z)  
$$

还可以写成联合熵形式：

$$  
I(X;Y|Z)

H(X,Z)+H(Y,Z)-H(Z)-H(X,Y,Z)  
$$

在特征选择中，经常写成：

$$  
I(C;Y|X)=H(C|X)-H(C|Y,X)  
$$

意思是：

> 已经选择了特征 $X$ 之后，新特征 $Y$ 还能额外减少多少关于类别 $C$ 的不确定性？

---

## 17.2 为什么 feature selection 需要 conditional MI？

只用 mutual information 排序有一个大问题：

**会选择很多重复表达同一信息的特征。**

例如文本分类里：

- $X_1=$ 是否出现 “car”
    
- $X_2=$ 是否出现 “automobile”
    

这两个词可能都和 “vehicle” 主题有关，所以：

$$  
I(C;X_1)  
$$

和：

$$  
I(C;X_2)  
$$

都很高。

但如果已经选择了 $X_1$，再选择 $X_2$ 可能帮助不大，因为它们信息重叠。

所以我们要看：

$$  
I(C;X_2|X_1)  
$$

如果它很小，说明：

> 在已经知道 $X_1$ 的前提下，$X_2$ 几乎没有额外贡献。

这就是 redundancy（冗余）。

slides 中也明确说，简单 mutual information ranking 忽略了 feature combinations 和 redundancy，需要看 interactions。

---

# 18. Interaction：交互信息、冗余与协同

## 18.1 课程中的 interaction 定义

课程中定义：

$$  
I(C;Y;X)=I(C;Y)-I(C;Y|X)  
$$

这个量叫 **interaction information（交互信息）**。

它比较的是：

- 不控制 $X$ 时，$Y$ 对 $C$ 的信息量：
    

$$  
I(C;Y)  
$$

- 控制 $X$ 后，$Y$ 对 $C$ 的额外信息量：
    

$$  
I(C;Y|X)  
$$

slides 中说明：$I(C;Y|X)$ 可能小于、大于或等于 $I(C;Y)$；如果更小，说明 $Y$ 关于 $C$ 的部分信息在给定 $X$ 后变得冗余。

---

## 18.2 情况 1：冗余 redundancy

如果：

$$  
I(C;Y|X)<I(C;Y)  
$$

则：

$$  
I(C;Y;X)>0  
$$

说明：

> $Y$ 本来对 $C$ 有信息，但这部分信息和 $X$ 重叠。  
> 一旦知道 $X$，$Y$ 的额外贡献变小。

例子：

- $C$：文章是否关于足球。
    
- $X$：是否出现 “football”。
    
- $Y$：是否出现 “soccer”。
    

$X$ 和 $Y$ 都指向同一主题，所以信息冗余。

---

## 18.3 情况 2：无交互 / 无额外影响

如果：

$$  
I(C;Y|X)=I(C;Y)  
$$

则：

$$  
I(C;Y;X)=0  
$$

说明：

> 知道 $X$ 不改变 $Y$ 对 $C$ 的信息贡献。

注意一个严谨点：

标准概率论里，真正的 “$C$ 和 $Y$ 在给定 $X$ 下条件独立” 是：

$$  
I(C;Y|X)=0  
$$

而不是：

$$  
I(C;Y|X)=I(C;Y)  
$$

但在本课程的 feature interaction 语境中，比较 $I(C;Y|X)$ 和 $I(C;Y)$ 是为了判断 $X$ 是否改变 $Y$ 对 $C$ 的信息贡献。考试如果问 interaction，按 lecture definition 写：

$$  
I(C;Y;X)=I(C;Y)-I(C;Y|X)  
$$

并解释 non-zero interaction。

---

## 18.4 情况 3：协同 synergy

如果：

$$  
I(C;Y|X)>I(C;Y)  
$$

则：

$$  
I(C;Y;X)<0  
$$

说明：

> 单独看 $Y$ 对 $C$ 信息不多，但在知道 $X$ 后，$Y$ 变得更有用。

这就是 **synergy（协同）**。

经典例子是 XOR：

|$X$|$Y$|$C=XOR(X,Y)$|
|--:|--:|--:|
|0|0|0|
|0|1|1|
|1|0|1|
|1|1|0|

单独知道 $X$，不能判断 $C$：

$$  
I(C;X)=0  
$$

单独知道 $Y$，也不能判断 $C$：

$$  
I(C;Y)=0  
$$

但如果知道 $X$，再知道 $Y$，就能完全确定 $C$：

$$  
I(C;Y|X)>0  
$$

所以：

$$  
I(C;Y;X)=I(C;Y)-I(C;Y|X)<0  
$$

这就是为什么三变量 interaction information 可以为负。

---

# 19. Information-Theoretic Feature Selection：基于信息论的特征选择

## 19.1 最简单的 Mutual Information Ranking

假设有：

- class variable：$C$
    
- features：$X_1,X_2,\dots,X_p$
    

最简单方法是：

1. 对每个 feature $X_i$ 计算：
    

$$  
I(C;X_i)  
$$

2. 按互信息从大到小排序。
    
3. 选择前 $m$ 个特征。
    

伪代码：

```text
Input:
    Dataset D with class C and features X1, X2, ..., Xp
    Number of features m

For each feature Xi:
    Compute MI[i] = I(C; Xi)

Sort features by MI[i] in descending order

Return top m features
```

slides 中也给出类似流程：统计每个 class 出现次数；对每个 word 建立 $K \times 2$ 的 class-by-word indicator table；计算每个表的 mutual information；返回 mutual information 最大的 $m$ 个词。

---

## 19.2 为什么简单 MI ranking 不够？

简单 MI ranking 有两个问题。

### 问题 1：它忽略 feature combinations

有些特征单独看没用，组合起来很有用。

例如 XOR：

$$  
I(C;X)=0  
$$

$$  
I(C;Y)=0  
$$

但：

$$  
I(C;X,Y)>0  
$$

如果只看单变量 mutual information，XOR 类型特征会被错过。

### 问题 2：它忽略 redundancy

两个特征可能都和类别高度相关，但它们表达的是同一件事。

例如：

- $X_1=$ 是否出现 “car”
    
- $X_2=$ 是否出现 “automobile”
    

它们都可能有高 MI，但同时选入会浪费特征名额。

---

## 19.3 Conditional Mutual Information Greedy Selection

为了减少冗余，可以用 **conditional mutual information（条件互信息）**。

算法思想：

1. 先选一个和类别 $C$ mutual information 最大的特征。
    
2. 之后每一步选择在已选特征集合 $S$ 条件下，对 $C$ 仍然最有额外信息的特征。
    

假设已选特征集合是：

$$  
S={X_1,\dots,X_k}  
$$

对于每个未选特征 $X_i$，计算：

$$  
I(C;X_i|S)  
$$

选择最大的那个：

$$  
X_{k+1}=\arg\max_{X_i\notin S} I(C;X_i|S)  
$$

slides 中给出的改进算法正是：先选 $I[C;X_i]$ 最大的特征；已有 $k$ 个特征后，对所有未选变量计算 $I[C;X_i|X_1,\dots,X_k]$，再选 conditional mutual information 最大的特征，并设置 stopping condition。

伪代码：

```text
Input:
    Dataset D
    Class variable C
    Features X1, X2, ..., Xp
    Desired number of features m
    Optional threshold epsilon

Initialize:
    S = empty set

Step 1:
    For each feature Xi:
        score[i] = I(C; Xi)
    Select X_best with maximum score
    S = S ∪ {X_best}

Step 2:
    While |S| < m:
        For each unselected feature Xi not in S:
            score[i] = I(C; Xi | S)

        Let X_best be the feature with maximum score

        If score[X_best] < epsilon:
            stop

        S = S ∪ {X_best}

Return S
```

---

## 19.4 这个算法为什么是 greedy？

因为每一步都只做当前看起来最好的选择：

$$  
\arg\max I(C;X_i|S)  
$$

它没有穷举所有可能特征组合。

所以它不保证找到全局最优特征子集。

slides 中也明确提醒：This is a greedy algorithm, so it does not necessarily come up with the best combination of features。

---

# 20. 如何从 contingency table 计算 mutual information？

考试里最常见的计算题就是：

> 给你 class counts 和 word occurrence counts，让你算哪个 word 更适合分类。

通用步骤如下。

假设有 $K$ 个类别：

$$  
c_1,c_2,\dots,c_K  
$$

某个 binary feature 是：

$$  
X\in{0,1}  
$$

其中：

- $X=1$：词出现。
    
- $X=0$：词不出现。
    

给定计数表：

|Class|$X=1$|$X=0$|
|---|--:|--:|
|$c_1$|$n_{1,1}$|$n_{1,0}$|
|$c_2$|$n_{2,1}$|$n_{2,0}$|
|$\cdots$|$\cdots$|$\cdots$|
|$c_K$|$n_{K,1}$|$n_{K,0}$|

总数：

$$  
N=\sum_c\sum_x n_{c,x}  
$$

---

## 20.1 第一步：算 class entropy

每个类别概率：

$$  
P(C=c)=\frac{n_{c,1}+n_{c,0}}{N}  
$$

类别熵：

$$  
H(C)=-\sum_c P(C=c)\log_2 P(C=c)  
$$

---

## 20.2 第二步：算 feature 取值概率

$$  
P(X=1)=\frac{\sum_c n_{c,1}}{N}  
$$

$$  
P(X=0)=\frac{\sum_c n_{c,0}}{N}  
$$

---

## 20.3 第三步：算条件分布

对于 $X=1$：

$$  
P(C=c|X=1)=\frac{n_{c,1}}{\sum_c n_{c,1}}  
$$

对于 $X=0$：

$$  
P(C=c|X=0)=\frac{n_{c,0}}{\sum_c n_{c,0}}  
$$

---

## 20.4 第四步：算条件熵

$$  
H(C|X=1)=-\sum_c P(C=c|X=1)\log_2 P(C=c|X=1)  
$$

$$  
H(C|X=0)=-\sum_c P(C=c|X=0)\log_2 P(C=c|X=0)  
$$

---

## 20.5 第五步：算平均条件熵

$$  
H(C|X)=P(X=1)H(C|X=1)+P(X=0)H(C|X=0)  
$$

---

## 20.6 第六步：算 mutual information

$$  
I(C;X)=H(C)-H(C|X)  
$$

选择 mutual information 更大的 feature。

---

# 21. 2017/18 真题例子：love vs flower 哪个更适合预测 genre？

2017/18 试卷 Question 2(c) 给出 1000 本书，类别是 romance、military、gardening，要求用 mutual information 比较 “love” 和 “flower” 哪个词更适合预测 genre。题目数据为：500 本 romance，200 本 military，300 本 gardening；romance 中 300 本含 love、400 本含 flower；military 中 50 本含 love、100 本含 flower；gardening 中 150 本含 love、100 本含 flower。

---

## 21.1 先算 class entropy

类别分布：

$$  
P(R)=\frac{500}{1000}=0.5  
$$

$$  
P(M)=\frac{200}{1000}=0.2  
$$

$$  
P(G)=\frac{300}{1000}=0.3  
$$

所以：

$$  
H(C)

-0.5\log_2 0.5  
-0.2\log_2 0.2  
-0.3\log_2 0.3  
$$

近似：

$$  
H(C)\approx 1.485  
$$

---

## 21.2 Word = love

love 出现表：

|Genre|love|not love|
|---|--:|--:|
|romance|300|200|
|military|50|150|
|gardening|150|150|

出现 love 的总数：

$$  
300+50+150=500  
$$

不出现 love 的总数：

$$  
200+150+150=500  
$$

所以：

$$  
P(X=\text{love})=0.5  
$$

$$  
P(X=\text{not love})=0.5  
$$

### love 出现时的 class distribution

$$  
P(R|love)=\frac{300}{500}=0.6  
$$

$$  
P(M|love)=\frac{50}{500}=0.1  
$$

$$  
P(G|love)=\frac{150}{500}=0.3  
$$

所以：

$$  
H(C|love)

-0.6\log_2 0.6  
-0.1\log_2 0.1  
-0.3\log_2 0.3  
$$

近似：

$$  
H(C|love)\approx 1.295  
$$

### love 不出现时的 class distribution

$$  
P(R|not\ love)=\frac{200}{500}=0.4  
$$

$$  
P(M|not\ love)=\frac{150}{500}=0.3  
$$

$$  
P(G|not\ love)=\frac{150}{500}=0.3  
$$

所以：

$$  
H(C|not\ love)

-0.4\log_2 0.4  
-0.3\log_2 0.3  
-0.3\log_2 0.3  
$$

近似：

$$  
H(C|not\ love)\approx 1.571  
$$

因此：

$$  
H(C|X_{love})

0.5\times 1.295+0.5\times 1.571  
$$

$$  
H(C|X_{love})\approx 1.433  
$$

所以：

$$  
I(C;X_{love})

1.485-1.433  
$$

$$  
I(C;X_{love})\approx 0.052  
$$

---

## 21.3 Word = flower

flower 出现表：

|Genre|flower|not flower|
|---|--:|--:|
|romance|400|100|
|military|100|100|
|gardening|100|200|

flower 出现总数：

$$  
400+100+100=600  
$$

flower 不出现总数：

$$  
100+100+200=400  
$$

所以：

$$  
P(X=\text{flower})=0.6  
$$

$$  
P(X=\text{not flower})=0.4  
$$

### flower 出现时的 class distribution

$$  
P(R|flower)=\frac{400}{600}=\frac{2}{3}  
$$

$$  
P(M|flower)=\frac{100}{600}=\frac{1}{6}  
$$

$$  
P(G|flower)=\frac{100}{600}=\frac{1}{6}  
$$

所以：

$$  
H(C|flower)

-\frac{2}{3}\log_2\frac{2}{3}  
-\frac{1}{6}\log_2\frac{1}{6}  
-\frac{1}{6}\log_2\frac{1}{6}  
$$

近似：

$$  
H(C|flower)\approx 1.252  
$$

### flower 不出现时的 class distribution

$$  
P(R|not\ flower)=\frac{100}{400}=0.25  
$$

$$  
P(M|not\ flower)=\frac{100}{400}=0.25  
$$

$$  
P(G|not\ flower)=\frac{200}{400}=0.5  
$$

所以：

$$  
H(C|not\ flower)

-0.25\log_2 0.25  
-0.25\log_2 0.25  
-0.5\log_2 0.5  
$$

$$  
H(C|not\ flower)=1.5  
$$

因此：

$$  
H(C|X_{flower})

0.6\times 1.252+0.4\times 1.5  
$$

$$  
H(C|X_{flower})  
\approx 0.751+0.6  
$$

$$  
H(C|X_{flower})  
\approx 1.351  
$$

所以：

$$  
I(C;X_{flower})

1.485-1.351  
$$

$$  
I(C;X_{flower})\approx 0.134  
$$

---

## 21.4 结论

因为：

$$  
I(C;X_{flower})\approx 0.134  
$$

而：

$$  
I(C;X_{love})\approx 0.052  
$$

所以：

$$  
I(C;X_{flower})>I(C;X_{love})  
$$

结论：

**flower is better suited to predict the genre than love, according to mutual information.**

直觉上这可能有点反直觉，因为 love 看起来很像 romance 词。但 mutual information 不只看某个词和某一个类别的直觉关联，而是看它对整个 class distribution 的平均不确定性减少。flower 的出现和不出现都更明显改变了 genre distribution，因此 MI 更高。

---

# 22. Feature Selection 考试答题模板

如果题目问：

> Describe an algorithm based on information theory to extract the $k$ most meaningful non-redundant features.

推荐答案结构如下。

```text
Input:
    Dataset with class variable Y
    Candidate features X1, X2, ..., Xn
    Desired number of features k

1. Estimate empirical distributions from the data:
       P(Y)
       P(Xi)
       P(Y, Xi)
       and later P(Y, Xi, selected features)

2. For each feature Xi:
       Compute mutual information I(Y; Xi)

3. Select the feature X_best with the largest I(Y; Xi)
       S = {X_best}

4. While |S| < k:
       For every unselected feature Xi:
           Compute conditional mutual information I(Y; Xi | S)

       Select the feature with the largest conditional mutual information:
           X_best = argmax I(Y; Xi | S)

       Add it to S

       Optionally stop if the best conditional MI is below a threshold epsilon

5. Return S
```

解释部分要写：

- Mutual information measures how much uncertainty about $Y$ is reduced by observing $X_i$.
    
- Conditional mutual information avoids selecting redundant features because it measures additional information given already selected features.
    
- The algorithm is greedy, so it does not guarantee the globally optimal feature subset.
    
- A stopping criterion is needed, such as selecting exactly $k$ features or stopping when conditional MI is below a threshold.
    

---

# 23. 本章公式总表

## 23.1 Self-information

$$  
I(x)=\log_2\frac{1}{p(x)}  
$$

$$  
I(x)=-\log_2 p(x)  
$$

---

## 23.2 Entropy

$$  
H(X)=-\sum_x p(x)\log_2 p(x)  
$$

$$  
H(X)=\sum_x p(x)\log_2\frac{1}{p(x)}  
$$

---

## 23.3 Binary entropy

$$  
H(X)=-p\log_2 p-(1-p)\log_2(1-p)  
$$

最大值在：

$$  
p=0.5  
$$

最大值：

$$  
H(X)=1  
$$

---

## 23.4 Joint entropy

$$  
H(X,Y)=-\sum_x\sum_y p(x,y)\log_2 p(x,y)  
$$

---

## 23.5 Conditional entropy

$$  
H(X|Y)=-\sum_y p(y)\sum_x p(x|y)\log_2 p(x|y)  
$$

$$  
H(X|Y)=H(X,Y)-H(Y)  
$$

---

## 23.6 Realized information

$$  
I(C;X=x)=H(C)-H(C|X=x)  
$$

可以为负。

---

## 23.7 Mutual information

$$  
I(X;Y)=H(X)-H(X|Y)  
$$

$$  
I(X;Y)=H(Y)-H(Y|X)  
$$

$$  
I(X;Y)=H(X)+H(Y)-H(X,Y)  
$$

$$  
I(X;Y)=D_{KL}(p(x,y)||p(x)p(y))  
$$

普通 mutual information 非负：

$$  
I(X;Y)\geq 0  
$$

---

## 23.8 Conditional mutual information

$$  
I(X;Y|Z)=H(X|Z)-H(X|Y,Z)  
$$

$$  
I(X;Y|Z)=H(Y|Z)-H(Y|X,Z)  
$$

$$  
I(X;Y|Z)=H(X,Z)+H(Y,Z)-H(Z)-H(X,Y,Z)  
$$

---

## 23.9 Interaction information

$$  
I(X;Y;Z)=I(X;Y)-I(X;Y|Z)  
$$

可能为正、零或负。

---

## 23.10 KL divergence

$$  
D_{KL}(p||q)=\sum_x p(x)\log_2\frac{p(x)}{q(x)}  
$$

---

## 23.11 Cross entropy

$$  
H(p,q)=-\sum_x p(x)\log_2 q(x)  
$$

---

## 23.12 Cross entropy 与 KL

$$  
H(p,q)=H(p)+D_{KL}(p||q)  
$$

---

# 24. 本章最容易错的点

## 24.1 Information 和 Entropy 不要混淆

**Information** 是单个事件的信息量：

$$  
I(x)=-\log_2 p(x)  
$$

**Entropy** 是整个分布的平均信息量：

$$  
H(X)=E[I(X)]  
$$

---

## 24.2 Entropy 高不等于事件稀有

Entropy 是分布整体的不确定性。

一个具体事件稀有，self-information 高。  
但一个分布 entropy 高，表示整体难预测。

---

## 24.3 Realized information 可以为负，但 mutual information 不可以

Realized information：

$$  
I(C;X=x)=H(C)-H(C|X=x)  
$$

可以为负。

Mutual information：

$$  
I(C;X)=H(C)-H(C|X)  
$$

作为期望，不会为负。

---

## 24.4 KL divergence 不是距离

KL 可以衡量分布差异，但不是 metric distance，因为：

$$  
D_{KL}(p||q)\neq D_{KL}(q||p)  
$$

并且一般不满足三角不等式。

---

## 24.5 Cross entropy 和 entropy 不一样

Entropy：

$$  
H(p)=-\sum_x p(x)\log p(x)  
$$

Cross entropy：

$$  
H(p,q)=-\sum_x p(x)\log q(x)  
$$

前者用真实分布 $p$ 编码自己。  
后者用模型分布 $q$ 编码真实分布 $p$。

---

## 24.6 Feature MI 高不代表一定应该一起选

两个 feature 都有高 MI，也可能互相冗余。

所以要用：

$$  
I(C;X_i|S)  
$$

判断它在已选特征集合 $S$ 之外还有多少额外贡献。

---

# 25. 考前快速答题模板

## 25.1 “What is entropy?”

可以答：

**Entropy is the expected information content or average uncertainty of a probability distribution. For a discrete random variable $X$ with distribution $p(x)$, it is defined as:**

$$  
H(X)=-\sum_x p(x)\log_2 p(x)  
$$

**For a binary variable with probabilities $p$ and $1-p$, entropy is maximized at $p=0.5$, where uncertainty is greatest.**

---

## 25.2 “What does KL divergence measure?”

可以答：

**KL divergence between $f(x)$ and $g(x)$ is:**

$$  
D_{KL}(f||g)=\sum_x f(x)\log\frac{f(x)}{g(x)}  
$$

**It measures the expected additional number of bits needed to encode samples from the true distribution $f$ using a code optimized for $g$. It is not a true distance metric because it is asymmetric and does not generally satisfy the triangle inequality.**

---

## 25.3 “What is realized information? Can it be negative?”

可以答：

**Realized information from observing $X=x$ about class $C$ is:**

$$  
I(C;X=x)=H(C)-H(C|X=x)  
$$

**It can be negative if the observation increases uncertainty. For example, if rain is rare overall but becomes equally likely after observing clouds, the conditional entropy can be larger than the original entropy.**

---

## 25.4 “What is mutual information?”

可以答：

**Mutual information is the expected reduction in uncertainty about one variable after observing another. In feature selection, $I(C;X)$ measures how informative feature $X$ is about class $C$:**

$$  
I(C;X)=H(C)-H(C|X)  
$$

**It is non-negative, symmetric, and equals zero if and only if the variables are statistically independent.**

---

## 25.5 “How can information theory be used for feature selection?”

可以答：

**For each candidate feature $X_i$, compute its mutual information with the class $C$:**

$$  
I(C;X_i)  
$$

**Select features with the highest mutual information. To avoid redundancy, use conditional mutual information after some features have already been selected:**

$$  
I(C;X_i|S)  
$$

**This selects features that provide additional information about $C$ beyond the already selected set $S$. The resulting method is greedy and needs a stopping condition such as a fixed number of features or a minimum information threshold.**

---

# 26. 最终核心总结

这章的逻辑链是：

1. **Self-information** 衡量单个事件有多 surprising：
    

$$  
I(x)=-\log_2 p(x)  
$$

2. **Entropy** 是平均 surprise，也是不确定性：
    

$$  
H(X)=-\sum_x p(x)\log_2 p(x)  
$$

3. **Conditional entropy** 衡量观察某个变量之后还剩多少不确定性：
    

$$  
H(C|X)  
$$

4. **Realized information** 衡量某个具体观察值让不确定性减少多少，可以为负：
    

$$  
I(C;X=x)=H(C)-H(C|X=x)  
$$

5. **Mutual information** 衡量一个特征平均减少多少类别不确定性，不能为负：
    

$$  
I(C;X)=H(C)-H(C|X)  
$$

6. **KL divergence** 衡量两个分布的差异，也可解释为额外编码代价：
    

$$  
D_{KL}(p||q)=\sum_x p(x)\log\frac{p(x)}{q(x)}  
$$

7. **Cross entropy** 是用模型分布 $q$ 编码真实分布 $p$ 的平均代价：
    

$$  
H(p,q)=-\sum_x p(x)\log q(x)  
$$

8. 三者关系：
    

$$  
H(p,q)=H(p)+D_{KL}(p||q)  
$$

9. **Feature selection** 用 mutual information 找有用特征，用 conditional mutual information 避免冗余：
    

$$  
X_{k+1}=\arg\max_{X_i\notin S} I(C;X_i|S)  
$$

最终你需要记住的最大考试原则是：

> 信息论特征选择的目标不是找“出现频率高的词”，也不是找“看起来有意义的词”，而是找那些能够最大程度减少 class uncertainty，并且在已有特征之外仍然提供额外信息的特征。