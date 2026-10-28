### 📚 课程概览 (Executive Summary)

- **课程主题**：序列标注（Sequence Labelling），重点讲解词性标注（POS）和命名实体识别（NER）。
    
- **核心模型**：介绍了传统的统计模型——条件随机场（CRF，**Conditional Random Field**），作为理解后续深度学习模型的基础。
    
- **关联实验**：本课内容与课程 Lab 紧密相关，Lab 将使用 CRF 进行命名实体识别 。
    
- **核心逻辑**：
    
    1. 定义问题：输入序列 $X$ $\rightarrow$ 输出标签序列 $Y$。
        
    2. 语言学基础：什么是词性（名词/动词等），什么是命名实体（人名/地名等）。
        
    3. 标注方法：如何把文本变成机器能懂的标签（如 BIO 标注法）。
        
    4. 模型引入：CRF 如何利用特征（Feature Engineering）来解决歧义。
        

---

### 📝 详细内容梳理

#### 1. 词性标注 (Parts of Speech, POS)

- **定义**：将单词分配到特定的词汇类别（如名词、动词、形容词）的过程，也称为 POS Tagging 。
    
- **分类体系 (Classes)**：
    
    - **封闭类 (Closed Classes)**：成员固定，通常是功能词（如介词 of, in; 冠词 the）。这类词很少随时间变化 。
        
    - **开放类 (Open Classes)**：成员开放，随着语言发展不断增加新词（如名词、动词、形容词、副词）。比如 "texting" (发短信) 是后来才有的词 。
        
- **Tagsets (标签集)**：
    
    - 不同的数据集使用不同的标签符号集。
        
    - **Penn Treebank**：本课程和教材（Jurafsky）主要使用的标签集（如 `NN` 表示名词，`VB` 表示动词）。
        
    - **Universal Dependencies (UD)**：另一种通用的标签集（如 `NOUN`, `VERB`）。
        
- **作用**：POS 提供了单词的句法结构线索（例如：名词前通常有冠词，动词后通常有名词作为宾语）。
    

#### 2. 命名实体识别 (Named Entity Recognition, NER)

- **定义**：识别文本中具有特定名称的实体，如 **Person (PER)**, **Location (LOC)**, **Organization (ORG)** 等 。
- **与词性标注 (POS) 的区别**：POS 通常针对单个单词（Word），而 NER 往往涉及**多词短语 (Multi-word phrase)**。例如 "New York" 是两个词组成的短语，指代一个“地点”实体 。
    
- **难点（歧义性）**：
    
    - **分词界限**：实体可能由多个单词组成（例如 "New York" 是两个词，但指代一个实体）。
        
    - **类型歧义**：同一个词在不同上下文中意思不同。
        
        - _老师举例_："Chelsea" 既可以是伦敦的一个地名，也可以是一个女孩的名字。判断依据是上下文（Context）。
            
        - _老师举例_："Washington" 可以是人名（George Washington），也可以是地名（Washington state），还可以是组织（政府代称）。
            

#### 3. 序列标注任务的形式化 (Sequence Labelling Formulation)

- **输入**：$X = x_1, x_2, ..., x_n$ （单词序列/Tokens）。
    
- **输出**：$Y = y_1, y_2, ..., y_n$ （标签序列）。
    
- **关键约束**：输入和输出的长度必须相等，$len(X) == len(Y)$。这是一个 1 对 1 的映射 。
    

#### 4. BIO 标注法 (BIO Tagging)

为了解决 NER 中“一个实体包含多个单词”的问题，我们使用 BIO 标签 ：

- **B (Begin)**：实体的开始。
    
- **I (Inside)**：实体的内部。
    
- **O (Outside)**：非实体。
    
- **举例**：
    
    - United (B-ORG) Airlines (I-ORG) said (O) 。
        
- _老师提示_：虽然还有 BIOES 等变体，但 **BIO 是首选的标准方法** 。
    

#### 5. 条件随机场 (Conditional Random Fields, CRF)

- **模型定位**：一种判别式（Discriminative）序列模型，基于对数线性模型（Log-linear model）。
    
- **工作原理**：它不是预测下一个词（生成式），而是寻找最可能的标签序列 $Y$，使得条件概率 $P(Y|X)$ 最大化 。
    
- **核心公式**：
    
    $$P(Y|X) = \frac{1}{Z(X)} \exp\left(\sum_{k=1}^{K} w_k F_k(X, Y)\right)$$
    
    其中 $F_k$ 是全局特征，$w_k$ 是权重，$Z(X)$ 是归一化因子 。
    
- **线性链 CRF (Linear Chain CRF)**：特征通常只依赖于当前位置的单词和**前一个位置的标签** ($y_{i-1}, y_i$) 。
    
- **特征工程 (Feature Engineering)**：CRF 依赖人工设计的特征模板，例如：
    
    - 当前词是什么？
        
    - 当前词是否大写？（对 NER 很重要）
        
    - 当前词的后缀（如 -ing, -ed）？
        
    - 前一个词是什么？
        
    - 是否在地名列表（Gazetteer）中？ 。
        

---

### 📢 教师重点复述 (Teacher's Emphasis)

_这部分包含老师在课堂上反复强调或语气加重的内容，通常是考试或 Lab 的关键。_

1. **Lab 的直接关联**：
    
    - 老师明确指出：“这正是 Lab 的内容。” 建议在本周四下节课之前就开始做 Lab，这样你能更好地理解模型内部是如何工作的 。Lab 使用的是 OntoNotes 数据集 。
        
2. **语言学知识 vs. 深度学习**：
    
    - 老师提到，过去（2015年左右）做 NLP 需要深厚的语言学知识来设计特征（如 CRF）。现在，基于 Transformer 的深度学习模型（如 BERT, GPT）通过海量数据“暴力”统计学习，效果通常优于人类语言学家定义的规则 。
        
    - **但是**，理解这些基础概念（POS, NER, CRF）对于构建良好的知识基础仍然至关重要 。
        
3. **NLTK 仅用于教学**：
    
    - 老师提到 PPT 中演示的 `nltk.pos_tag` 代码仅供教学和娱乐（"teaching and a bit of fun"）。在实际生产或高性能需求下，应该使用 Transformer 模型 。
        
4. **歧义处理 (Ambiguity)**：
    
    - 这是序列标注的核心难点。单词的意义由其**上下文 (Context)** 决定。模型需要通过观察周围的单词（特征）来消除歧义 。
        

---

### 🧪 重点公式与术语解释

#### 1. CRF 概率公式

$$P(Y|X) = \frac{1}{Z(X)} \exp(\sum_{k=1}^{K} w_k F_k(X,Y))$$

- **直观解释**：给定输入句子 $X$，标签序列 $Y$ 出现的概率，取决于所有特征 $F_k$ 的加权和。权重 $w_k$ 是通过训练学到的。如果某个特征（如“大写字母通常是人名”）很重要，对应的 $w$ 就会很大。
    
- **$Z(X)$**：配分函数（Partition Function），用于归一化，确保所有可能的 $Y$ 的概率加起来等于 1。
    

#### 3. 评估方法 (Evaluation)

- **POS:** 通常使用 **Accuracy (准确率)** 。
	- **定义**：对于 POS Tagging，因为每个词都有一个非空的标签（且标签数量相对平衡），我们通常只关心“对错”。
	
	**公式**：
	
	$$\text{Accuracy} = \frac{\text{预测正确的 Token 数}}{\text{总 Token 数}}$$
	
	**案例**：
	
	- **真实序列 ($Y$)**: `The/DT` `cat/NN` `sat/VBD`
	    
	- **预测序列 ($\hat{Y}$)**: `The/DT` `cat/NN` `sat/NN` (最后错把动词预测为名词)
	    
	
	**计算**：
	
	- 总数 = 3
	    
	- 正确数 = 2 ("The", "cat")
	    
	- **Accuracy** = $2 / 3 = \mathbf{66.7\%}$
    
- **NER:** 必须使用 **Precision, Recall, F1 Score** 。
- ![[Pasted image 20260215150059.png]]
    
    - **Micro F1:** 汇总所有实例计算（受大类影响大）。
        
    - **Macro F1:** 先按类别算 F1 再平均（对小类敏感，推荐用于处理类别不平衡） 。
        ![[Pasted image 20260215150124.png]]
    - **处理 'O' 标签:** 务必在报告结果时排除 'O' 标签，否则准确率会虚高 。

```
import numpy as np
import itertools

# 1. 定义输入和标签
sentence = ["Bill", "codes"]
tags = ["N", "V"] # Noun, Verb

# 2. 定义特征函数 (Features)
# 格式: f(index, y_prev, y_curr, sentence) -> 0 or 1
# 为了简单，我们假设 y_prev 在序列开始时是 "START"

def f_word_is_Bill_tag_is_N(i, y_prev, y_curr, x):
    return 1 if x[i] == "Bill" and y_curr == "N" else 0

def f_word_is_codes_tag_is_V(i, y_prev, y_curr, x):
    return 1 if x[i] == "codes" and y_curr == "V" else 0

def f_prev_N_curr_V(i, y_prev, y_curr, x):
    return 1 if y_prev == "N" and y_curr == "V" else 0

def f_word_is_codes_tag_is_N(i, y_prev, y_curr, x):
    return 1 if x[i] == "codes" and y_curr == "N" else 0

# 3. 定义权重 (Weights)
weights = {
    "word_Bill_tag_N": 3.0,
    "word_codes_tag_V": 2.0,
    "trans_N_to_V": 2.0,
    "word_codes_tag_N": 0.5
}

# 特征列表
features = [
    (f_word_is_Bill_tag_is_N, weights["word_Bill_tag_N"]),
    (f_word_is_codes_tag_is_V, weights["word_codes_tag_V"]),
    (f_prev_N_curr_V, weights["trans_N_to_V"]),
    (f_word_is_codes_tag_is_N, weights["word_codes_tag_N"])
]

# 4. 枚举所有可能的路径 (Tag Sequences)
# 只有两个词，所以是 2^2 = 4 种情况
possible_sequences = list(itertools.product(tags, repeat=len(sentence)))

results = []

print(f"输入句子: {sentence}")
print("-" * 30)

Z = 0
for seq in possible_sequences:
    score_sum = 0
    feature_details = []
    
    # 遍历序列中的每个位置
    for i in range(len(sentence)):
        y_curr = seq[i]
        y_prev = "START" if i == 0 else seq[i-1]
        
        # 遍历每个特征
        for func, weight in features:
            val = func(i, y_prev, y_curr, sentence)
            if val > 0:
                score_sum += weight * val
                feature_details.append(f"{func.__name__}({weight})")
    
    exp_score = np.exp(score_sum)
    Z += exp_score
    results.append({
        "seq": seq,
        "score_sum": score_sum,
        "exp_score": exp_score,
        "details": feature_details
    })

print(f"归一化因子 Z = {Z:.4f}")
print("-" * 30)

for res in results:
    prob = res["exp_score"] / Z
    print(f"路径: {res['seq']}")
    print(f"  Sum(w*f): {res['score_sum']}")
    print(f"  Exp Score: {res['exp_score']:.4f}")
    print(f"  Probability: {prob:.4f}")
    print(f"  Active Features: {res['details']}")
    print("-" * 30)


```

代码输出

```
输入句子: ['Bill', 'codes']
------------------------------
归一化因子 Z = 1138.7864
------------------------------
路径: ('N', 'N')
  Sum(w*f): 3.5
  Exp Score: 33.1155
  Probability: 0.0291
  Active Features: ['f_word_is_Bill_tag_is_N(3.0)', 'f_word_is_codes_tag_is_N(0.5)']
------------------------------
路径: ('N', 'V')
  Sum(w*f): 7.0
  Exp Score: 1096.6332
  Probability: 0.9630
  Active Features: ['f_word_is_Bill_tag_is_N(3.0)', 'f_word_is_codes_tag_is_V(2.0)', 'f_prev_N_curr_V(2.0)']
------------------------------
路径: ('V', 'N')
  Sum(w*f): 0.5
  Exp Score: 1.6487
  Probability: 0.0014
  Active Features: ['f_word_is_codes_tag_is_N(0.5)']
------------------------------
路径: ('V', 'V')
  Sum(w*f): 2.0
  Exp Score: 7.3891
  Probability: 0.0065
  Active Features: ['f_word_is_codes_tag_is_V(2.0)']
------------------------------

```

这是一个简单的条件随机场 (CRF) 计算示例。为了让你直观理解，我们只用 2 个词和 2 个可能的标签。
#### 局部特征和全局特征
**局部特征**是针对句子中**某一个具体位置**的快照；而**全局特征**是整个句子中所有局部快照的**总和**。
##### 1. 局部特征函数 (Local Feature Function, $f_k$)

局部特征函数 $f_k$ 关注的是序列中的**单个时刻（单个位置 $i$）**。它是一个“检测器”，用来判断当前位置是否满足某种特定的语言学规律 。

- **定义：** $f_k(y_{i-1}, y_i, X, i)$
    
    - 它查看的内容包括：
        
        - **当前标签 ($y_i$)**
            
        - **前一个标签 ($y_{i-1}$)** （这就是“线性链” Linear Chain 的来源）
            
        - **整个输入句子 ($X$)**
            
        - **当前位置索引 ($i$)**
            
- **输出：** 通常是布尔值（0 或 1）。如果满足条件输出 1，否则输出 0。也可以是数值（实数值）。
    
- **例子：**
    
    假设我们正在做 **词性标注 (POS Tagging)**。
    
    - **局部特征 $f_{100}$**：检测“如果当前词是以 'ly' 结尾，且当前标签是副词 (Adverb)，则为 1”。
        
    - 在句子 _"He runs **quickly**"_ 中：
        
        - 当 $i$ 指向 "runs" 时，$f_{100}$ 输出 **0**（结尾不是 ly）。
            
        - 当 $i$ 指向 "quickly" 时，$f_{100}$ 输出 **1**（结尾是 ly 且标注为副词）。
            

---

##### 2. 全局特征函数 (Global Feature Function, $F_k$)

全局特征函数 $F_k$ 是对**整个序列**的评分。它是同一个局部特征在句子中每一个位置 $i$ 的**求和** 。

- **定义：** $F_k(X, Y) = \sum_{i=1}^{n} f_k(y_{i-1}, y_i, X, i)$
    
    - 这意味着：我们拿着同一个“检测器”（局部特征），把句子从头到尾扫一遍，看看这个特征被触发了多少次。
        
- **作用：** 在 CRF 的概率公式中，权重 $w_k$ 是与**全局特征** $F_k$ 相乘的 。模型学习的是“整个句子中出现这种特征的重要性”。
- **“局部特征一共有 $n \times k$ 个”**
    
    - **✅ 正确。**
        
    - **解释：** 假设句子长度为 $n$（有 $n$ 个 Token/位置），我们设计了 $k$ 个特征模板（Feature Templates，例如“当前词是大写吗”、“前一个词是‘the’吗”）。
        
    - **计算过程：** 机器会拿着这 $k$ 个模板，在句子的每一个位置 $i$（共 $n$ 个位置）都计算一遍。
        
    - **结果：** 这一步确实产生了一个 $n \times k$ 的矩阵（或者是 $n$ 个长度为 $k$ 的局部向量）。
        
    - _来源支持：_ 课件 Slide 27 提到局部特征 $f_k$ 是针对每一个索引位置 $i$ 定义的 。
        
- **“每 $n$ 个局部特征求和”**
    
    - **✅ 正确。**
        
    - **解释：** 对于第 $j$ 个特征模板（比如“是否是地点名词”），我们把它在位置 1、位置 2 ... 直到位置 $n$ 的得分全部加起来。
        
    - **数学表示：** $\sum_{i=1}^{n} f_j(\dots)$。这一步就是把 $n$ 这个维度“压缩”掉了。
        
    - _来源支持：_ 课件 Slide 28 明确写道：“Feature values are summed over entire sentence X” 。
        
- **“得到一个全局特征，全局特征有 $k$ 个”**
    
    - **✅ 正确。**
        
    - **解释：** 因为我们有 $k$ 个特征模板，每个模板都从头到尾加了一遍，所以最终得到了 $k$ 个总分。
        
    - **结果：** 最终的全局特征向量 $F(X, Y)$ 的长度永远是 $k$，不管句子长度 $n$ 是多少。

### 2. 定义特征函数 (Features)

CRF 的核心在于定义“特征函数” ($f_k$) 并给每个特征赋予“权重” ($w_k$)。特征可以是局部的（看当前词），也可以是转移的（看前一个标签）。

#### 3. 直观举例：NER 任务 (New York 案例)

假设我们要识别句子中的 **地点 (LOC)**。

**句子 ($X$):** "I love **New York** but I live in **York**."

**假设标签序列 ($Y$):** `O`, `O`, `B-LOC`, `I-LOC`, `O`, `O`, `O`, `O`, `B-LOC`

我们设计一个局部特征 $f_{city}$：

- **规则：** 如果当前词在地名列表中 (Gazetteer Lookup) 且标签是 `LOC` (包括 B-LOC 或 I-LOC)，则为 1，否则为 0 。
    

#### 步骤 A：计算局部特征 (Scanning)

模型会遍历句子中的每一个词 ($i=1$ 到 $9$)：

1. **I (O):** 词不在列表 -> $f_{city} = 0$
    
2. **love (O):** 词不在列表 -> $f_{city} = 0$
    
3. **New (B-LOC):** 词在列表? 是(假设New在列表)。标签是LOC? 是 -> **$f_{city} = 1$**
    
4. **York (I-LOC):** 词在列表? 是。标签是LOC? 是 -> **$f_{city} = 1$**
    
5. **but (O):** -> 0
    
6. **I (O):** -> 0
    
7. **live (O):** -> 0
    
8. **in (O):** -> 0
    
9. **York (B-LOC):** 词在列表? 是。标签是LOC? 是 -> **$f_{city} = 1$**
    

#### 步骤 B：计算全局特征 (Summing)

全局特征 $F_{city}$ 就是上面所有局部值的总和：

$$F_{city}(X, Y) = 0+0+1+1+0+0+0+0+1 = \mathbf{3}$$

#### 步骤 C：模型打分

如果训练好的权重 $w_{city} = 2.5$（表示这个特征对判断地点很重要），那么这个特征对整个句子得分的贡献就是：

$$w_{city} \times F_{city}(X, Y) = 2.5 \times 3 = 7.5$$

#### 计算得分 (Score)

CRF 对每个可能的标签序列 $Y$ 计算一个“非归一化得分”：

$$Score(Y|X) = \exp\left(\sum_{i} \sum_{k} w_k f_k(y_{i-1}, y_i, x, i)\right)$$

简单说就是：**把路径上所有触发的特征权重加起来，然后取指数 $e^x$。**

---

### 4. 归一化 (Partition Function Z)
$$Z(X) = \sum_{Y' \in \mathcal{Y}} \exp \left( \sum_{k=1}^{K} w_k F_k(X, Y') \right)$$
我们把这个公式像剥洋葱一样拆开来看：

- **$Y'$**：代表**某一种**可能的标签序列组合。
    
- **$\mathcal{Y}$**：代表给定长度为 $N$ 的句子下，**所有可能**的标签序列集合 。
    
- **$\exp(\dots)$**：代表序列 $Y'$ 在当前特征和权重下的**非归一化得分（Unnormalized Score）**。
    

**用一句话总结：** $Z(X)$ 就是把输入句子 $X$ 搭配**所有可能**的标签序列分别打个分（求指数），然后把这些分数**全部加起来**。

$$Z(X) = \text{Score}(N,N) + \text{Score}(N,V) + \text{Score}(V,N) + \text{Score}(V,V)$$

$$Z(X) \approx 20.08 + 1096.63 + 1.0 + 7.39 \approx 1125.1$$假设我们有一个非常短的句子，且只有 3 种可能的标签序列组合（我们称之为序列 A、B、C）。

1. **打分**：模型通过特征和权重 $w \cdot F$，计算出三个序列的基础得分：
    
    - 序列 A 得分：$2$
        
    - 序列 B 得分：$5$
        
    - 序列 C 得分：$3$
        
2. **取指数（求 $\exp$）**：为了保证分数都是正数，并且拉大好坏序列之间的差距，我们取指数：
    
    - 序列 A：$e^2 \approx 7.4$
        
    - 序列 B：$e^5 \approx 148.4$
        
    - 序列 C：$e^3 \approx 20.1$
        
3. **计算 $Z(X)$**：把它们全加起来！
    
    - $$Z(X) = 7.4 + 148.4 + 20.1 = 175.9$$
        
4. **转化为概率**：现在，我们想知道序列 B 是正确答案的概率 $P(B|X)$，只需用它的得分除以 $Z(X)$：
    
    - $$P(B|X) = \frac{148.4}{175.9} \approx 84.4\%$$
        

通过除以这个总和 $Z(X)$，我们就把原本绝对的分数，变成了加起来等于 100% 的相对概率。

### 5. 计算最终概率

最后，正确路径 **[N, V]** 的概率是：

$$P(Y=[N, V] | X) = \frac{\text{Score}(N, V)}{Z(X)} = \frac{1096.63}{1125.1} \approx \mathbf{97.47\%}$$

而错误路径 **[N, N]** 的概率是：

$$P(Y=[N, N] | X) = \frac{20.08}{1125.1} \approx \mathbf{1.78\%}$$

## CRF 模型推理

### 1. 目标：找到概率最大的序列 ($\hat{Y}$)

最上面的公式 $\hat{Y} = \text{argmax}_{Y \in \mathcal{Y}} P(Y|X)$ 是我们的终极目标。

- **$X$**: 输入的句子（例如 "I love New York"）。
    
- **$Y$**: 所有可能的标签序列组合（这个空间非常巨大，称为 $\mathcal{Y}$）。
    
- **$P(Y|X)$**: 条件概率。我们需要遍历所有可能的 $Y$，找到那个让概率最大的序列。
    

---

### 2. 简化的魔法：为什么要划掉 $\frac{1}{Z(X)}$ 和 $\exp$？

图片中间最核心的部分是那个**红色的叉**。原本的概率公式很复杂：

$$P(Y|X) = \frac{1}{Z(X)} \exp \left( \sum w_k F_k(X, Y) \right)$$

但在做推理（找最大值）时，我们可以把公式简化，**只计算指数里面的部分**。这也是老师在课堂上强调的“让问题变得可计算（Tractable）和高效（Efficient）”的关键步骤 。

#### **(1) 为什么要忽略 $Z(X)$？**

- **数学原因：** $Z(X)$ 是归一化因子（Normalization Factor），它的作用是把分数变成 0 到 1 之间的概率。
    
- **逻辑原因：** 老师在录音中解释道，$Z(X)$ 只是关于输入 $X$ 的函数，和我们正在选择的标签 $Y$ 无关 。
    
- **结论：** 对于同一个句子 $X$，无论你怎么换标签 $Y$，$Z(X)$ 都是同一个常数。在比较大小时，除以同一个常数不会改变排名。所以可以直接丢掉 。
    

#### **(2) 为什么要忽略 $\exp()$？**

- **数学原因：** 指数函数 $e^x$ 是**单调递增**的（Monotonically Increasing）。
    
- **逻辑原因：** 如果 $A > B$，那么 $e^A > e^B$。
    
- **结论：** 老师指出，我们只关心**相对排名（Ranking）**，不需要知道绝对的概率值 。只要找到让指数内部得分最高的序列，就等于找到了概率最高的序列。
    

---

### 3. 最终的计算公式（沉底的公式）

划掉那些多余项后，我们得到了最终用于写代码实现的公式：

$$\hat{Y} = \text{argmax}_{Y \in \mathcal{Y}} \sum_{i=1}^{n} \sum_{k=1}^{K} w_k f_k(y_{i-1}, y_i, X, i)$$

这个公式把一个复杂的概率问题，变成了简单的**加法打分问题**：

1. **双重求和 ($\sum \sum$)：**
    
    - 外层循环 $i=1 \dots n$：遍历句子的每一个位置（每一个词）。
        
    - 内层循环 $k=1 \dots K$：遍历每一个特征模板（Feature Template）。
        
2. **$w_k f_k$：** 用训练好的权重乘以局部特征值。
    
3. **含义：** 只要把这句话中所有位置、所有被触发特征的权重加起来，得分最高的那个序列 $\hat{Y}$ 就是我们的预测结果。
### 总结

CRF 通过学习权重（比如 $w_3=2.0$ 告诉模型“名词后接动词的可能性很大”），利用全局信息（$Z(X)$ 考虑了所有可能性）来消除歧义。即便局部特征对 "codes" 是名词还是动词犹豫不决，转移特征 N $\to$ V 的高权重会帮助模型选择正确的 V。 

#### 3. 特征模板 (Feature Templates)

在 CRF 中，你需要定义“局部特征”。

- **代码示例 (Python伪代码)**：
    
    Python
    
    ```
    def get_features(sentence, i):
        word = sentence[i]
        features = {
            'word': word,
            'is_capitalized': word[0].isupper(), # 有助于识别专有名词
            'suffix_3': word[-3:],               # 有助于识别词性 (如 -ing -> 动词)
            'prev_word': sentence[i-1] if i > 0 else '<START>', # 上下文
        }
        return features
    ```
    

---

### 💡 助教复习速查卡 (Cheat Sheet)

|**概念**|**解释**|**关键点/例子**|
|---|---|---|
|**POS**|词性标注|名词(NN), 动词(VB), 形容词(JJ)。Penn Treebank 是常用标签集。|
|**NER**|命名实体识别|识别 PER, ORG, LOC。解决歧义依赖上下文。|
|**BIO Tagging**|序列标注编码法|**B**egin (开始), **I**nside (内部), **O**utside (外部)。|
|**CRF**|条件随机场|判别式模型，利用全局归一化。公式含 $exp(\sum w F)$。|
|**Sequence Labelling**|任务定义|输入 $X$ 和输出 $Y$ 长度必须相等 ($1:1$ 映射)。|
|**Evaluation**|评估指标|使用 F1 分数。**不要**只看 Accuracy，**忽略** 'O' 标签。|
|**Closed Class**|封闭词类|介词、代词、冠词（很少变动）。|
|**Open Class**|开放词类|名词、动词（随网络/文化造词而增加）。|

---
