
---

# 深度学习进阶：从词向量到多模态与序列模型 (Comprehensive Study Guide)

## 📌 课程与考务概览 (Course Overview & Exam Tips)

* **期末考试重点**：期末考试共4道题，其中针对基础深度学习结构（如CNN、层参数计算、损失函数等）的考题需要极其仔细地复习细节 。


* **特殊主题（Special Topics）免考提醒**：本次课程涉及的高阶主题（如词嵌入详细公式、LSTM门控内部结构等）**不会出现在期末考试中**，旨在拓展思路并辅助完成 Coursework 2 。


* **作业要求**：Coursework 2 为小组作业，强烈建议在计算机视觉（CV）项目中，将深度学习方法与传统的“手工特征提取 (handcrafted features)”进行横向对比 。



---

## 模块一：词汇表征的演进与潜在语义分析 (Evolution of Word Representations & LSA)

### 1. 传统表征的困境：独热编码 (One-Hot Encoding)

在早期的自然语言处理中，最直观的单词表示方法是**独热编码 (One-Hot Encoding)** 。若英文词汇表有超过 171k 个单词，每个词都会被表示为一个长度为 171k 的向量，仅在对应词汇的索引位置为 1，其余全为 0 。

* **灾难性缺陷**：
1. 
**维度爆炸**：向量极其冗长且稀疏（Sparse），给计算带来巨大负担 。


2. 
**语义孤岛**：任何两个不同词汇的距离计算结果都一样，完全**无法捕捉同义词或代数语义相关性 (No notion of synonymy)** 。


### 2. 破局：分布假说 (Distributional Hypothesis)

现代词嵌入理论的基石是**分布假说**：词汇的语义由其上下文决定。在文本中经常出现在相似上下文里的词，其意义大概率相近 。我们需要一个低维的**密集向量 (Dense Representation)**，使得语义相近的词（如 Cat 和 Kitten）在空间中的距离更近 。
#### Latent Space 与深度自编码器 

- **Latent Space (潜在空间)**：它的维度远小于输入数据的维度 。我们希望将意义相近的词映射到这个空间中相近的位置 。
    
- **Autoencoder (自编码器)**：由 Encoder（将输入压缩为较短的表征）和 Decoder（从表征重建输入）组成，中间的隐藏层就可以作为输入的 Embedding 。
    
- 降维技术如 PCA 或 T-SVD 虽然古老，但在去除冗余信息、应对小数据集（Few-shot learning）时依然非常有用 。

### 💡 难点剖析 (Deep Dive)：潜在语义分析 (LSA) 与截断奇异值分解 (T-SVD)

在引入神经网络之前，传统的降维技术如 **LSA (Latent Semantic Analysis)** 依然极具价值，尤其在训练数据集较小（如小样本学习 Few-shot learning）时能有效提升模型表现 。其底层数学工具是**截断奇异值分解 (Truncated SVD)** 。

公式表达为：$M \approx U_r \Sigma_r V_r^T$

* 
**$M$ 矩阵 (Terms $\times$ Docs)**：原始的“词汇-文档”矩阵。每一行代表词汇表里的一个具体单词，每一列代表语料库里的一篇文档。该矩阵极其庞大且稀疏 。


* 
**$U_r$ 矩阵 (Terms $\times r$)**：左奇异矩阵。每一行依然对应一个具体单词，但列被大幅压缩成了 $r$ 个**“隐含主题”或“潜在概念” (Latent Concepts)**。**物理意义**：这就是我们在低维潜空间中的**词向量 (Word Embedding)**！它刻画了词汇与各类隐藏特征的相关度 。


* 
**$\Sigma_r$ 矩阵 ($r \times r$)**：包含奇异值的对角矩阵。**物理意义**：各个隐含主题的**“重要性得分”或“能量”**。数值越大，说明该潜在主题在语料库中的主导地位越强 。


* 
**$V_r^T$ 矩阵 ($r \times$ Docs)**：右奇异矩阵的转置。每一列对应原始的一篇文档。**物理意义**：这是这篇文档的**文档向量 (Document Representation)**，表示该文档在各隐含主题上的权重分配 。


* 
**截断 (Truncation) 的奥秘**：由于原始数据包含大量噪音和冗余信息，我们故意丢弃较小奇异值对应的部分，仅保留最大的 $r$ 个“主成分 (principal component)”。这不仅实现了降维，还神奇地将语义相似的词挤压到了潜空间相近的位置 。



---

## 模块二：Word2Vec 核心架构与数学本质 (Word2Vec Core Architecture & Mathematical Essence)

Word2Vec 的伟大之处在于它能捕捉到高级的**代数语义 (Algebraic Semantics)**，例如 $word2vec("Brother") - word2vec("Man") + word2vec("Woman") = word2vec("Sister")$ 。

### 1. 极简的神经网络架构

老师反复强调，Word2Vec 的网络架构极其简单：**仅有一个隐藏层 (One hidden layer)**，并且**没有任何激活函数 (No extra activation function)**，即纯粹的线性映射 。

### 💡 难点剖析 (Deep Dive)：Embedding 的数学本质——“查表操作”

在网络中，输入层到隐藏层是通过一个 $V \times N$ 的权重矩阵 $W_1$ 连接的（$V$ 是词表大小，$N$ 是潜空间维度）。
当我们输入一个表示“词表第二个单词”的独热向量 `[0 1 0 ...]` 并与矩阵 $W_1$ 相乘时，数学上并没有进行复杂的混合运算，而是**直接把矩阵 $W_1$ 的第二行给“提取”或“复制”了出来** 。
**核心结论**：训练完成后，权重矩阵 $W_1$ 的每一行，实际上就是词汇表中对应那个单词的最终**词向量 (Word Representation)** ！
**局限性**：由于这种查表机制，在这个基础模型中，无论一个词有多少种含义（如 "cat" 是动物也是缩写），它在矩阵里**永远只有一行，只有一种固定的向量表示 (only one representation)** 。

### 💡 难点剖析 (Deep Dive)：为什么要构建“词对 (Word Pairs)”？

深度学习需要输入（特征）和输出（标签）。但我们拥有的只是无监督的纯文本。

* 
**滑动窗口机制 (Sliding Window)**：我们在句子上设定一个滑动窗口，遍历所有词。窗口中间的词称为**“中心词” (Focus word)**，周围的词称为**“上下文词” (Context words)** 。


* 
**化无监督为有监督**：通过将它们两两组合提取出训练样本（例如提取出 `(quick, the)`, `(quick, brown)`），我们人为制造了用于反向传播的 $X$ 和 $Y$ 。



### 2. 两大经典训练流派：CBOW 与 Skip-gram

* **CBOW (Continuous Bag-of-Words)**：
* 
**机制**：输入多个上下文词去预测一个中心词 。
* **优势**：可以一次性输入所有上下文词并在隐层求平均，这使得它的**训练速度非常快 (Several times faster to train)**，对高频词汇的效果很好 。




* **Skip-gram** 
**机制**：反其道而行之，输入一个中心词，去预测它周围的多个上下文词 。


* **优势**：虽然需要**一对一对 (pair by pair)** 地训练，导致效率较低，但这能强迫模型更精细地记忆特征，它**能为生僻词 (rare words) 或在小数据集下提供更优质的特征表示** 。

---

## 模块三：工程优化与 OOV 解决方案 (Engineering Optimization & OOV Solutions)

早期的 Word2Vec 面临两个严峻的工程挑战：一是计算量过大，二是无法处理未登录词。

### 💡 难点剖析 (Deep Dive)：Subword Tokenization (子词分词) 解决 OOV 危机

* 
**传统痛点**：Word2Vec 无法处理词汇表外的词 **(Cannot deal with out of vocabulary (OOV) words)** 。因为输入依赖独热编码，遇到新词（如网络新造词）在矩阵中找不到索引，会导致严重的语义丢失。


* **现代解决方案**：现代 NLP 模型（如 Transformer 系列）采用了 **Subword Tokenization**（如 BPE、WordPiece）。
* **切碎新词**：当遇到未知的单词（如 "unbelievably"），它不再报错，而是将其拆解为在训练集中见过的“高频子词”组件（如 `["un", "##believ", "##ably"]`）。
* **重组语义**：这些子词在矩阵中早已拥有自己的 Embedding 向量。模型将这些已知子词的向量组合起来，就能“拼凑”推断出这个未登记词的大致含义。



### 💡 难点剖析 (Deep Dive)：加速魔法——下采样与负采样

* **下采样 (Subsampling)**：
* 像 "the" 这样的高频词在上下文中无处不在，无法提供任何有用的同义词线索 。


* **底层动作**：在扫描文本构建滑动窗口时，代码会根据词频计算一个“丢弃概率”。模型在扫描时“掷骰子”，高频词有很大几率直接被忽略，不生成训练样本，从而大幅砍掉无意义的计算 。




* **负采样 (Negative Sampling)**：
* 更新整个庞大的输出权重矩阵（$W_2$）极其耗时 。


* **底层动作**：负采样将原本庞大的“多分类问题”简化为局部的“二分类问题”。每次训练，模型只取 1 个真实的**正样本**（努力拉近它们的向量距离，预测为 1），并从词表中随机抽取少量（如 5-20个）没出现过的**负样本**（努力推远距离，预测为 0）。在反向传播时，**仅仅只更新这几个局部对应的权重**，速度飙升 ！





### 💡 难点剖析 (Deep Dive)：非凸优化 (Non-convex) 与局部最优解

* 
**现象**：即便你固定了网络架构和所有超参数，每次重新打乱数据集（Reshuffle）跑出来的词向量数值依然会有微小区别 。


* 
**底层机理**：因为神经网络的损失函数地貌是一个连绵起伏的“丘陵地带”——即**非凸函数 (Non-convex)** 。


梯度下降算法就像一个蒙着眼睛下山的人。由于起始位置不同（权重随机初始化或数据打乱），他只能顺着脚下的坡度往下走，最终大概率会掉进不同的“坑底”，即**局部最优解 (Local Minima)**，而非绝对的全局最低点 。



---

## 模块四：多模态潜空间与概率语义嵌入 (Multi-Modal Latent Space & Probabilistic Semantic Embedding)

不仅文字可以被映射到潜空间，图像同样可以。通过构建跨模态的桥梁，我们可以实现**以图搜图 (Searching by image)**、**以词搜图 (Searching by Keyword)** 甚至**图像自动描述 (Image captioning)** 。

### 💡 难点剖析 (Deep Dive)：图像与文字的特征对齐

两个长得完全不一样的模态（像素 vs 词表），是如何塞进同一个潜空间里的？

1. **独立的特征提取 (Separate Encoders)**：
* 文字经过它专属的 Encoder 2（如 RNN 或查表）。
* 图像经过它专属的 Encoder 1，通常是**深度卷积神经网络 (Deep CNN)** 。




2. **化矩阵为坐标点 (Flatten & Projection)**：
* CNN 会像扫描仪一样，逐层浓缩提取图像的高级语义特征（例如形状、纹理）。
* 在最末端，这些特征会被**展平 (Flatten)** 成一维向量，并通过一个**全连接层 (Fully Connected Layer)** 进行线性映射。这个矩阵乘法的作用，就是将图像的维度强行压缩到与文本预设潜空间**完全一致的维度**。


3. **KL-divergence 语义绑定**：
* 对齐不是在输入端发生的，而是在输出端。模型通过计算 **KL 散度 (KL-divergence)** 作为损失函数，强迫描述同一事物的“图像分布”和“文本分布”在潜空间中尽可能重合、靠拢 。





### 💡 难点剖析 (Deep Dive)：概率语义嵌入与重参数化魔法 (Reparameterization Trick)

在上述架构图（Slide 19）中，这被称为**概率语义嵌入 (Probabilistic semantic embedding)** 。

* 
**输出不再是一个死板的点**：编码器输出的 $q_{\phi_x}(\boldsymbol{z}|\boldsymbol{x})$ 实际上代表一个正态分布，包含**均值向量 ($\boldsymbol{\mu}$)** 和 **标准差/方差向量 ($\boldsymbol{\sigma}$)** 。这表示“这张图的特征在这个圈里”。


* **采样 (Sampling) 与重参数化**：
公式 $z \sim q$ 意味着我们需要在这个分布里随机抽取一个点 $\boldsymbol{z}$ 作为最终的 Embedding Vector 用于后续的解码重建 (Reconstruction $\hat{\boldsymbol{x}}$) 。
但是，**“随机抽取”这个动作在代码中是不可导的 (non-differentiable)**，会导致反向传播中断。
**解决魔法**：



$z = \boldsymbol{\mu} + \boldsymbol{\sigma} \odot \boldsymbol{\epsilon}$



通过引入一个从标准正态分布中随机抽取的纯噪音向量 $\boldsymbol{\epsilon}$，我们将随机性全部剥离了出去。由于 $\boldsymbol{\mu}$ 和 $\boldsymbol{\sigma}$ 依然是通过明确的数学运算连接，梯度可以顺畅无阻地流回编码器进行权重更新！这种做法极大地增强了模型的鲁棒性和泛化能力。

---

## 模块五：序列模型与 RNN 家族 (Sequence Models & RNN Family)

当我们需要处理语音识别、机器翻译、假新闻检测（Fake news detection）等序列数据时，基础网络不再适用 。

### 💡 难点剖析 (Deep Dive)：为何标准神经网络 (Standard NNs) 会失效？

以文本中的人名检测 (Name detection in text) 为例 ：

1. 
**致命缺陷一：可变长度 (Variable Lengths)**。在不同的训练样本中，输入句子和对应的输出长度（$T_x$ 和 $T_y$）总是不同的 。标准网络要求输入节点数在初始化时严格固定。


2. 
**致命缺陷二：无法特征共享 (No Feature Sharing)**。标准网络在文本的不同位置学习到的特征无法互通共享 。句首识别出的人名特征，无法直接复用于句尾。



**解决思路**：必须让网络记住过去发生的事情 (remember what happened in the past) 。因此，我们在网络中引入了一个**带有时间延迟的反馈环路 (a feedback loop delayed in time)**，这就是 **循环神经网络 (RNN)** 的本质 。每次前向传播，网络都会把当前的隐藏状态（记忆）连同新的单词一起输入计算。

### 1. Vanilla RNN 的阿喀琉斯之踵

普通 RNN 结构极简，内部仅依靠一个 `tanh` 激活函数运行 。
![[Pasted image 20260312142043.png]]
* 
**致命缺陷**：面对长序列时，它会遭受**梯度消失 (Vanishing gradients)** 的折磨 。在反向传播过程中，小于 1 的数值经历极其漫长的连乘后会趋于 0，导致模型彻底遗忘序列开头的信息，只能关注节点上最新的信息 。



### 2. 进化：门控机制 (Gates)

为了克服梯度消失，控制信息的流动，科学家引入了带有智能阀门（Gates）的架构：**GRU** 与 **LSTM**。

* 
**数学机制**：门控本质上是使用 `Sigmoid` ($\sigma$) 激活函数的网络层 。由于其输出域在 0 到 1 之间，它可以完美扮演阀门的角色：如果输出为 1，代表历史信息全部放行；如果为 0，代表历史信息被彻底丢弃 。



### 3. GRU 与 LSTM 终极对决

* 
**GRU (Gated Recurrent Unit)**：包含 **2 个门**（重置门 Reset gate、更新门 Update gate）。
![[Pasted image 20260312142729.png]]
在这张图中，GRU 巧妙地通过 **2 个门 (2 gates)** 来管理记忆 ：**重置门 (Reset gate)** 和 **更新门 (Update gate)** 。

## 1. 基础输入与输出的对齐

- $x_t$：当前时间步的输入向量 (input vector) 。
    
- $h_{t-1}$：上一个时间步传过来的历史记忆（隐藏层向量） 。
    
- $h_t$：当前时间步计算完毕后，输出的最新隐藏层向量 (hidden layer vectors) 。
    
- $W, b$：分别代表参数矩阵 (parameter matrices) 和偏置向量 (bias vector) 。
    

## 2. 重置门 (Reset Gate, $r_t$)：决定“遗忘多少过去”

- **公式**：$r_t = \sigma(W_r \cdot [h_{t-1}, x_t] + b_r)$
    
- **机制解析**：如图中结构所示，模型将当前输入 $x_t$ 和历史记忆 $h_{t-1}$ 拼接后，乘上权重矩阵 $W_r$ 并加上偏置 $b_r$，最后通过 Sigmoid 阀门 ($\sigma$) 激活 。计算出的 $r_t$ 决定了我们在生成新的候选记忆时，要**丢弃多少历史包袱**。
    

## 3. 生成候选记忆 ($\tilde{h}_t$)

- **公式**：$\tilde{h}_t = \tanh(W_h \cdot [r_t \odot h_{t-1}, x_t] + b_h)$
    
- **机制解析**：这里引入了 $\tanh$ 激活函数来规范特征值域 。请注意公式中极其精妙的 $r_t \odot h_{t-1}$（对应图中央指向带有 $\tanh$ 框的交叉乘法节点）：这正是上一步算出的重置门 $r_t$ 作用于历史记忆 $h_{t-1}$ 。如果阀门 $r_t$ 计算出为 0，历史记忆就在这里被无情清空，模型仅依赖当前输入 $x_t$ 来生成新的候选记忆。
    

## 4. 更新门 (Update Gate, $z_t$)：决定“采纳多少新知”

- **公式**：$z_t = \sigma(W_z \cdot [h_{t-1}, x_t] + b_z)$
    
- **机制解析**：同样通过 Sigmoid 函数计算出一个控制系数 $z_t$ 。它是掌控全局、决定最终输出状态的最关键因素。
    

## 5. 最终记忆的融合与输出 ($h_t$)

- **公式**：$h_t = (1 - z_t) \odot h_{t-1} + z_t \odot \tilde{h}_t$
    
- **老师在课堂上的硬核举例 (极端情况分析)**：
    
    - **情况 A (极度“喜新厌旧”)**：假设阀门 $z_t = 1$ 。那么公式前半部分的 $(1 - z_t)$ 就变成了 $0$。老师强调，这意味着历史信息会被**完全阻断丢失 (totally get lost)** ，当前时刻的最终记忆 $h_t$ 将完全等同于刚计算出的候选记忆 $\tilde{h}_t$。
        
    - **情况 B (极度“因循守旧”)**：假设阀门 $z_t = 0$ 。那么 $(1 - z_t)$ 就是 $1$，而公式后半部分归零。老师强调，这意味着**只有来自上一步的隐藏层历史信息 (hidden information from the previous time) 会被直接、无损地传递到下一个时间步** ，新输入的特征被彻底忽略。
---

* **LSTM (Long Short Term Memory)**：包含 **3 个门**（遗忘门 Forget gate、输入门 Input gate、输出门 Output gate）以及贯穿始终的“传送带”——**细胞状态 (Cell State, $C_t$)**，让长期记忆无损流动 。
![[Pasted image 20260312142738.png]]


**工程与实战建议 (Effectiveness & Tips)**：

* 
**参数与难度**：门控数量越多，训练难度越高。RNN < GRU < LSTM (LSTM 实际上非常难训练) 。


* 
**模型选择**：总体效果上 $RNN < GRU \approx LSTM$ 。


* 
**小型数据集 (Small datasets)**：首选架构相对简单的 **GRU** 。


* 
**大型数据集 (Large datasets)**：拥有极其庞大的数据时，首选特征捕获能力更强的 **LSTM** 。




* 
**时代局限**：尽管 LSTM 曾经立下赫赫战功（Apple Siri, Google Translate），但老师留下了最终的悬念——**LSTM 无法实现高度的并行化训练 (LSTM cannot be trained in parallel)**，这也正式宣告了接下来 **Transformer 时代**的到来 ！


