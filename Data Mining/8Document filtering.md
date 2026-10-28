你好！我是你的课程助教。这节关于“文档过滤（Document Filtering）”的课程包含了不少概率论和数学推导，听起来可能会让人觉得有些繁重和吃力，这是完全正常的感受。不过不用担心，我已经帮你把核心脉络理清了。我们将主要围绕如何使用机器学习（特别是朴素贝叶斯和 Fisher 方法）来区分垃圾邮件（Spam）和正常邮件（Ham）展开。

以下是我为你整理的复习指南和重点梳理。

## 📝 课程精炼摘要

本节课主要探讨如何通过监督式机器学习将文档（如电子邮件）分类为垃圾信息（Spam）或正常信息（Ham）。课程详细讲解了特征工程（如词袋模型、TF-IDF）的实现步骤，并深入剖析了两种概率分类模型：朴素贝叶斯（Naive Bayes）和 Fisher 方法 。核心思路是将文本转化为特征向量，并通过概率模型计算文档属于特定类别的可能性。

---

## 🚨 教师反复强调的考试与作业重点

- **考试范围预警**：老师明确指出，**Fisher 方法不会在考试中测试**，但你可以将其应用到你的课程作业（Coursework）中 。复习时请将重心放在朴素贝叶斯（Naive Bayes）上。
    
- **代价敏感的错误分类**：在垃圾邮件过滤中，**假阳性（False Positive，即把正常邮件误判为垃圾邮件）的代价远高于假阴性（False Negative）** 。因此，模型评估时应优先追求高精确率（High Precision），即使这会牺牲一定的召回率（Recall）。
    
- **“简单粗暴”但有效的假设**：朴素贝叶斯之所以“朴素”，是因为它假设所有单词（特征）的出现是完全独立的 。老师强调，虽然这个假设在现实中（如语言的上下文关系）是错的，但它在实际应用中通常表现得非常好 。
    
- **特征工程的决定性作用**：选择好的特征（Feature engineering）与选择好的分类器同样重要 。如果没有好的特征，哪怕算法再强也难以发挥作用 。
    

---

## 📖 核心知识点与章节精讲

#### 1. 特征工程 (Feature Engineering) (Slides 10, 29-30 | 08:26-13:42)

在将文本送入模型前，必须先将其转化为机器可读的特征。

- **词袋模型 (Bag of Words)**：对文本进行简单的分词（基于非字母字符分割），转换为小写，并统计词汇的出现频率或是否存在 。
    
- **代码实现**：在 Python 演示中，使用了 `CountVectorizer` 将文本转化为向量形状（如 60份文档 $\times$ 228个词汇） 。
    
- **进阶特征**：为了保留上下文，可以使用 N-Grams（如相邻的2到3个词组合），或者利用 TF-IDF 来评估词汇的重要程度 。
    

#### 2. 朴素贝叶斯分类器 (Naive Bayes) (Slides 10-23 | 14:01-41:31)

这是本课的绝对核心内容。

- **条件概率计算**：首先计算单词在特定类别中出现的概率 $p(f|c)=\frac{n_{fc}}{n_{c}}$ 。
    
- **零概率陷阱 (Zero-frequency Problem)**：如果一个词（如“tea”）在训练集的垃圾邮件中从未出现过，其计算概率为0 。这会导致整个文档的连乘概率瞬间归零，这显然不符合现实逻辑 。
    
- **平滑处理 (Smoothing)**：为了解决零概率问题，引入了“假设概率（Assumed Probability）”或先验常识，通过加权将其与数据中观察到的概率混合，从而防止概率为0 。
    
- **文档总概率与对数技巧 (Log Space)**：依据独立性假设，整篇文档的概率是各个单词概率的乘积 。由于大量小数连乘会导致浮点数下溢（Underflow）归零，我们在实际操作中对公式取对数（Log），将乘法转化为加法计算 。公式如下：$\log(p(c|d)) \propto \log(p(c)) + \sum_{f \in d} \log(p(f|c))$ 。
    
- **MAP 决策规则**：计算出文档属于 Spam 和 Ham 的后验概率后，比较两者大小，选取概率更大的类别（Maximum A Posteriori） 。
    

#### 处理零概率陷阱 

这页幻灯片介绍了**贝叶斯平滑（Bayesian Smoothing）**技术。当我们在训练集中遇到某个单词在特定类别（例如垃圾邮件）中**从未出现过**时，原始概率会计算为 $0$。这会导致整篇文档的后验概率连乘结果直接归零。为了防止这种极端情况，我们引入了一个**“假设概率（Assumed Probability / Prior）”**，通过加权平均的方式，把我们的“常识”和“实际观察到的数据”混合起来，得到一个更稳定、不为零的平滑概率。

---

## 🔍 逐行原理解析

#### 1. 理论基础：抛弃单点样本，拥抱概率分布

> _"Using Bayesian Statistics, assuming that random variables are drawn from probability distributions rather than point samples"_

- **直观理解**：频率学派认为“我没看到（单点样本为0），那概率就是0”。但贝叶斯学派认为，真实概率是一个**分布**。即使我们目前没在样本里观察到某个词（比如“tea”在垃圾邮件里没出现），这并不意味着它在真实世界的垃圾邮件中绝对不可能出现。我们只是“暂时还没遇到”而已。
    

#### 2. 数学假设：二项分布与 Beta 先验

> _"In this case, Spam/Ham classification for a feature $f$ is binomial with a beta distributed prior"_

- **专有名词解释**：
    
    - **二项分布 (Binomial)**：指的是一封邮件要么是垃圾邮件（Spam），要么是正常邮件（Ham），只有两种结果。
        
    - **Beta 先验 (Beta distributed prior)**：在贝叶斯统计中，Beta 分布是二项分布的“共轭先验”。你不必深究复杂的推导过程，只需记住它的作用：**它允许我们在看到真实数据之前，先注入一个我们主观猜测的“基础概率”（即常识）。**
        

#### 3. 🚨 重点公式拆解：平滑概率的计算

这是老师在课堂上花费大量时间解释的核心公式。你需要深刻理解它的每一个变量：

$$p_w(f|c) = \frac{weight \times assumed + count \times p_{raw}(f|c)}{count + weight}$$

这个公式本质上是一个**加权平均（Weighted Average）**，它在衡量“你的主观常识”与“客观数据”之间的比重：

- **$p_w(f|c)$**：我们最终需要的**平滑后的条件概率**（用来替代那个危险的 $0$）。
    
- **$assumed$**：**假设概率（常识）**。比如，在没有任何数据前，我们猜测一个词出现在垃圾邮件和正常邮件的概率各占一半，即 $0.5$。
    
- **$weight$**：**假设的权重（信念强度）**。你有多相信你的“常识”？数字越大，表示你越坚持主观假设。
    
- **$p_{raw}(f|c)$**：**原始条件概率**。也就是我们根据训练集真实统计出来的概率（例如 $0/30 = 0$）。
    
- **$count$**：**该单词在所有类别文档中出现的总次数**。这代表了“客观数据”的充足程度。（注意：不是在单个类别中的次数，而是所有类别中的总次数）。
    

💡 **老师的重点提示（直观理解）**：

老师在课堂上强调了分母的作用是**归一化（Normalization）**。

- 如果一个词很生僻，$count$ 非常小（接近 $0$），那么公式会变成由 $weight \times assumed$ **主导**。这意味着在数据不足时，我们**听从常识**。
    
- 如果一个词很常见，$count$ 非常大，那么公式会变成由 $count \times p_{raw}(f|c)$ **主导**。这意味着在数据充足时，我们**相信数据**。
    

#### 4. 实践中的超参数设置

> _"In practice, the values for weight and assumed are found through testing to optimize performance. Reasonable starting points are 1 and .5 for weight and assumed, respectively"_

- 在实际写代码做 Coursework 时，你需要设置这两个超参数。
    
- **默认推荐值**：通常将 **$weight = 1$**，**$assumed = 0.5$**。这也就是著名的拉普拉斯平滑（Laplace Smoothing）或其变体。
    
- **如何优化**：你可以把训练集分成两部分（训练集和验证集），尝试不同的 $weight$ 值，看看哪个能让验证集上的分类精确率（Precision）最高。
    

---

## 🧮 结合上一节课的具体例题推演

在上一页幻灯片中，你可能记得有一个尴尬的情况：

单词 `"tea"` 在 30 封垃圾邮件（Spam）中出现了 0 次，在 70 封正常邮件（Ham）中出现了 15 次。

如果我们直接算 $p_{raw}(\text{"tea"} | \text{Spam}) = \frac{0}{30} = 0$。这个 $0$ 会毁了后续的连乘计算。

现在，让我们套用这张幻灯片的**平滑公式**来拯救它！

假设我们采用默认起点参数：**$weight = 1$**，**$assumed = 0.5$**。

1. **确定变量**：
    
    - $p_{raw}(\text{"tea"} | \text{Spam}) = 0$
        
    - $count$ = `"tea"` 在所有邮件出现的总次数 = 在垃圾邮件的 $0$ 次 + 在正常邮件的 $15$ 次 = **$15$** 次。
        
2. **代入公式计算**：
    
    $$p_w(\text{"tea"} | \text{Spam}) = \frac{1 \times 0.5 + 15 \times 0}{15 + 1}$$
    
    $$p_w(\text{"tea"} | \text{Spam}) = \frac{0.5 + 0}{16} = 0.03125$$
    

你看！通过平滑公式，原本冷冰冰的 $0$ 变成了一个很小但不为 $0$ 的安全数值 **$0.03125$**。这样不仅符合“茶（tea）确实极少出现在垃圾邮件中”的数据事实，也避免了概率直接归零导致整个模型崩溃的技术故障。
#### Naive Bayes

![[Pasted image 20260311142538.png]]
## 1. 什么是“朴素（Naïve）”假设？

- **独立性假设**：幻灯片顶部提到，朴素假设允许我们将条件概率表示为所有特征（单词）条件概率的乘积 。老师在课上强调这是一个“极其简化甚至有些粗暴”的假设，它假定文档中所有的单词彼此之间是**完全独立**的，没有任何关联 。虽然我们在现实说话时单词间肯定有联系，但这个假设在垃圾邮件过滤中实际表现得非常好 。
    
- **似然度公式（Likelihood）**：基于上述假设，一整篇文档 $d$ 给定在某个类别 $c$ 下的概率（似然度），就可以简单粗暴地写成该文档中所有单词出现概率的**连乘积** 。公式表达为：
- $$p(d|c) = \prod_{f \in d} p(f|c)$$
- ## 2. 贝叶斯公式全拆解

接下来，幻灯片展示了经典的贝叶斯分类公式：

$$P(c|d) = \frac{P(d|c) P(c)}{p(d)}$$

每个变量的直观含义如下：

- **Posterior (后验概率) $P(c|d)$**：给定文档 $d$ 的内容后，它是类别 $c$（例如垃圾邮件）的概率 。这也是我们分类器最终想要得出的预测结果！
    
- **Likelihood (似然概率) $P(d|c)$**：如果已知这封邮件是类别 $c$（垃圾邮件），它长成文档 $d$ 这样（即包含这些特定单词）的概率 。这就是我们刚刚用上面那个连乘公式算出来的东西。
    
- **Prior (先验概率) $P(c)$**：在没看文档具体内容之前，某个类别本身在整体数据中出现的概率（例如你的收件箱里平时有多少比例是垃圾邮件） 。
    
- **Evidence (证据) $p(d)$**：这篇文档 $d$ 在整个数据集中出现的总体概率 。
- $$\log(p(c|d)) \propto \log(p(c)) + \sum_{f \in d} \log(p(f|c))$$

## 根据问题调整预测
![[Pasted image 20260311205130.png]]
在这个分类诈骗邮件的任务中，明显的是 consider ham as spam is more expensive ,which means FP can't be torelated . so we can apply the method above to adjust the classification threshould.

---

#### 3. Fisher 方法 (Fisher's Method) (Slides 24-28 | 42:37-47:46)

此部分不考，但可用于拓展作业。

- **核心差异**：与朴素贝叶斯从单词概率推导文档概率不同，Fisher 方法交换了条件的位置，直接计算给定单词时类别的概率 $p(c|f)$ 。
- $$p(c|d) = K^{-1}(-2 \sum_{i=1}^k \ln(p(c = C|f_i)), 2k) = K^{-1}(-2 \ln(\prod_{i=1}^k p(c = C|f_i)), 2k)$$
    
- **概率组合**：它利用逆卡方函数（Inverse chi-squared function），将各个单词的概率值合并为一个单一的检验统计量进行分类判断 。
    

---

## 💻 实验与代码实践建议 (供 Coursework 参考)

老师在课上演示了如何处理文本，你可以使用 Python 的 `scikit-learn` 库快速复现：

Python

```
from sklearn.feature_extraction.text import CountVectorizer, TfidfVectorizer
from sklearn.naive_bayes import MultinomialNB

# 1. 实例化向量化工具 (对应老师讲解的 Tokenization 和特征化)
vectorizer = CountVectorizer(lowercase=True) # 或者使用 TfidfVectorizer

# 2. 拟合训练集并转换为矩阵
X_train_vec = vectorizer.fit_transform(X_train_text)

# 3. 训练朴素贝叶斯模型 (MultinomialNB 内部自带平滑参数 alpha，可解决零概率问题)
clf = MultinomialNB(alpha=1.0) 
clf.fit(X_train_vec, y_train)

# 4. 预测与评估 (务必关注 Precision 指标)
```

---

## 🗂️ 考前速查卡 (可直接复制到笔记)

- **核心目标**：将文档映射为标签（Spam 或 Ham） 。
    
- **特征化步骤**：分词 $\rightarrow$ 转小写 $\rightarrow$ 统计词频/TF-IDF $\rightarrow$ 向量化 。
    
- **Naive Bayes 假设**：所有单词（特征）相互独立（尽管实际上并非如此） 。
    
- **解决零概率**：采用平滑技术（引入 assumed probability 和 weight）防止个别词汇拉垮整体概率 。
    
- **防下溢策略**：取对数运算（Log transform），化乘为加 。
    
- **业务逻辑**：宁可漏掉垃圾邮件（False Negative），绝不误杀正常邮件（False Positive）。优先优化 Precision 。
    
- **Fisher vs NB**：NB 计算 $p(f|c)$ 并连乘；Fisher 计算 $p(c|f)$ 并用卡方分布合并（考试不考） 。
    

---

**缺失或待确认信息：**

- 提供的幻灯片第 11-14 页中有一个关于 "money", "viagra", "tea" 的概率计算表格，转写文本中虽然提到了计算，但没有完全覆盖具体的平滑公式代入计算过程。
    

你希望我结合幻灯片里 "viagra" 和 "tea" 的表格数据，为你手把手推演一遍朴素贝叶斯的具体计算过程吗？这对应对多选题（MCQs）会非常有帮助！