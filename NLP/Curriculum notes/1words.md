你好！我是你的课程助教。这是为你整理的 **Natural Language Processing (COMP 3225/6253)** 第 2 讲：**Words (词)** 的详细学习指南。

这节课从自然语言与计算机语言的区别入手，核心讲解了 NLP 中最基础的单位——“词”的处理技术，包括**分词 (Tokenization)**、**子词编码 (BPE)** 和 **编辑距离 (Edit Distance)**。

---

### 📝 1. 课程精要 (Executive Summary)

- **核心主题**：如何将自然语言文本转化为机器可处理的格式。
    
- **关键概念**：
    
    - **自然语言的模糊性**：与精确的计算机语言不同，自然语言充满歧义（Ambiguity）和依赖上下文（Context）。
        
    - **术语区分**：Type（词型）vs Token（词例），Lemma（词元）vs Stem（词干）。
        
    - **预处理流水线**：从简单的分词到更高级的子词处理（Byte-Pair Encoding, BPE），后者是 Transformer 模型（如 BERT, GPT）的基础 。
        
    - **词的相似度**：通过 Levenshtein Distance（最小编辑距离）计算两个单词在拼写上的差异 。
        

---

### 🚨 2. 教师“重点复述” (Teacher's Emphasis)

_以下内容在讲座中被特别强调，考试或实验中极有可能涉及：_

- **必须阅读教材**：关于 **Levenshtein Distance** 的具体算法流程和图解，幻灯片上的图摘自 Jurafsky & Martin 的教材。老师明确表示：“这是对扩展阅读的一次很好的提醒，你需要去读那本书的章节” 。
    
- **分词工具的选择**：虽然课上讲了 BPE 的原理，但在实际工程中，老师的 **"Go-to" (首选)** 工具是 **SentencePiece**。它语言无关（Language independent），对多语言支持更好，处理中文等非空格分词语言也更有效 。
    
- **Naive 分词行不通**：仅仅用空格和标点符号切分单词（Naive solution）在 NLP 中是“不够好”的，因为它无法处理缩写（Ph.D.）、数字格式（1,000 vs 1.000）、社交媒体标签（#hashtag）等复杂情况 。
    
- **Type vs Token**：这是一个必须搞清楚的基础概念。Type 是词汇表里的唯一词，Token 是文本中出现的具体实例 。
    

---

### 📖 3. 课程详细梳理 (Content Breakdown)

#### 3.1 自然语言 vs 计算机语言 (Natural vs Computer Language)

- **对比**：计算机语言是模块化、无歧义的；自然语言是模糊的、抽象的，严重依赖语调、标点和上下文 。
    
- **歧义性 (Ambiguity)**：
    
    - **双关语 (Double Entendre)**：幽默往往来自于歧义。
        
    - _例_: "How do you make a Swiss roll?" -> "Push him down the mountainside!" (利用了 roll 作为名词“卷”和动词“滚”的歧义) 。
        
    - _例_: "Children make nutritious snacks." (孩子做零食 vs 孩子也是一种有营养的零食？) 。
        

#### 3.2 文本数据的多样性 (Variety of Text)

- 模型如果在一种文风（如正规新闻）上训练，在另一种文风（如社交媒体或莎士比亚戏剧）上表现会很差 。
    
- **常见类型**：
    
    - **正规文本**：新闻、论文（结构清晰）。
        
    - **口语 (Speech)**：包含犹豫、重复、非正规语法（"lovey dovey", "I don't know..."）。
        
    - **社交媒体**：包含大量缩写、Emoji、Hashtag、URL，且语法随意。**这是 NLP 处理的难点** 。
        

#### 3.3 核心术语 (Terminology)

- **Lemma (词元)**：具有相同词干、词性和词义的词的抽象形式（如 _say_ 是 _said, says, saying_ 的 lemma）。
    
- **Word Form (词形)**：文本中实际出现的词（如 _said_）。
    
- **Type (词型)**：词汇表中的不同单词数量（Vocabulary Size）。
    
- **Token (词例)**：文本中单词的总数。
    
    - _例子_：“Reading a book in a car to get a book.” -> 这里有两个 "book" 的 **Tokens**，但在词汇表中只有一个 "book" 的 **Type** 。
        
- **Function Words (虚词)**：如 _of, the, in_。频率极高（齐夫定律 Zipf's Law），但在深度学习时代通常**不再移除**它们，因为它们包含语法结构信息 。
    

#### 3.4 文本预处理流水线 (Preprocessing Pipeline)

1. **分词 (Tokenization)**：将字符流转换为单词序列。
    
    - **挑战**：日期格式（01/02/2021）、货币（$45.55）、缩写（m.p.h.）、连字符（rock 'n' roll）。
        
    - **Emoji**：包含情感信息，处理时应保留 Unicode 编码，不要暴力转为 ASCII 。
        
2. **词干提取 (Stemming)**：
    
    - **Porter Stemmer**：一种基于规则的粗糙算法，直接切除后缀（如 -ing, -ed）。
        
    - _缺点_：可能产生非单词（如 "walls" -> "wall", 但 "this" -> "thi"）。
        
3. **词形还原 (Lemmatization)**：更高级，利用词典和形态学分析还原词根（如 _better_ -> _good_），但计算代价较高。
    

#### 3.5 子词编码：Byte-Pair Encoding (BPE)

**这是现代 LLM（如 GPT, BERT）的核心分词技术**。

- **动机**：解决未登录词（Unknown words / OOV）问题。如果词汇表没有“unhappiness”，模型就无法理解。但如果切分为 `un` + `happi` + `ness`，模型就能理解 。
    
- **算法流程** ：
    
    1. **初始化**：词汇表只包含所有单个字符。
        
    2. **统计**：在语料库中统计所有相邻字符对的频率。
        
    3. **合并 (Merge)**：找到频率最高的字符对（如 'e' 和 'r'），将其合并为新符号 'er'，并加入词汇表。
        
    4. **迭代**：重复 k 次（k 是超参数），直到词汇表达到预定大小。
        
- **结果**：常见词（如 "the"）保留为完整单词；罕见词（如 "unhappiness"）被拆解为常见的子词片段。
    
- **变体**：
    
    - **WordPiece** (BERT使用)：基于概率而非单纯频率。
        
    - **SentencePiece** (Llama/Google使用)：**推荐使用**，直接在原始句子上操作，不依赖空格分词 。
        

#### 3.6 最小编辑距离 (Minimum Edit Distance)

- **定义**：将一个字符串转换为另一个字符串所需的最少操作次数（插入 Insert、删除 Delete、替换 Substitute）。
    
- **Levenshtein Distance**：
    
    - 最常用的算法。
        
    - **代价 (Cost)**：通常插入和删除的代价为 1，替换的代价可以是 1 或 2（视作删除+插入）。
        
    - **应用**：拼写检查（Spell check）、抄袭检测、机器翻译评估 。
        
- **算法实现**：使用动态规划（Dynamic Programming）或递归计算（见 Slide 35 公式）。
    

---

### 🧪 4. 算法伪代码与解释 (Algorithm Check)

#### Levenshtein Distance (递归版)

这部分对应 Slide 35 和老师的口述解释 。

Python

```
def lev(a, b):
    # 终止条件：如果一个字符串为空，距离就是另一个字符串的长度（全插入或全删除）
    if len(b) == 0: return len(a)
    if len(a) == 0: return len(b)
    
    # 递归步骤：
    # 如果首字符相同，不需要操作，直接比较剩下的部分 (tail)
    if a[0] == b[0]:
        return lev(a[1:], b[1:])
    
    # 如果首字符不同，取以下三种操作的最小值 + 1：
    else:
        return 1 + min(
            lev(a[1:], b),     # Delete (删除 a 的首字符)
            lev(a, b[1:]),     # Insert (在 a 前插入 b 的首字符)
            lev(a[1:], b[1:])  # Replace (将 a 的首字符替换为 b 的首字符)
        )
```

_注意：递归效率极低，实际应用（如考试手算或代码）通常使用动态规划矩阵表格法。建议参考教材中的表格填写法。_

---

### 🧠 5. 复习速查卡 (Review Flashcards)

**Q1: 为什么在现代 Transformer 模型中常用 BPE 而不是简单的空格分词？**

- A: 因为 BPE 可以处理 **未登录词 (OOV)** 问题，将罕见词拆解为常见的子词（Subwords），既减小了词汇表大小（Vocab size），又保留了语义信息。
    

**Q2: 在 NLP 中，Type 和 Token 的区别是什么？**

- A: **Type** 是词汇表中独特的单词（种类）；**Token** 是文本中出现的单词总数（实例）。比如 "to be or not to be" 有 6 个 tokens，但只有 4 个 types (to, be, or, not)。
    

**Q3: Levenshtein Distance 中，如果将单词 "cat" 变成 "hat"，距离是多少？（假设替换代价为 1）**

- A: 距离是 1。操作是：Substitute 'c' with 'h'。
    

**Q4: 什么是 Lemmatization？它与 Stemming 有何不同？**

- A: Lemmatization（词形还原）是将词还原为字典中的原型（如 _better_ -> _good_），通常需要词性信息；Stemming（词干提取）是粗暴地切除后缀（如 _better_ 可能不变，但 _running_ -> _run_），不需要字典，速度快但精度低。
    

**Q5: 社交媒体文本（Social Media Text）对 NLP 提出了什么特殊挑战？**

- A: 非标准语法、拼写错误、大量的 Emoji、Hashtags、URL 以及独特的缩写（如 "u" 代替 "you"）。
    

---

### 🤝 下一步 (Next Step)

**你需要我为你详细演示 Levenshtein Distance 的动态规划矩阵填表过程（考试常考题型），还是想深入了解如何在 PyTorch 中调用 SentencePiece 进行分词？**