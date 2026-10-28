你好！我是你的课程助教。

欢迎来到 **COMP6237 Data Mining** 的第五讲：**搜索与排序 (Search and Rank)**。

这一讲我们暂时离开“降维”和“聚类”，进入了**信息检索 (Information Retrieval, IR)** 的领域。这是现代互联网搜索引擎（如 Google）的核心技术基础。课程从处理非结构化文本的挑战出发，详细讲解了如何将文本转化为计算机可理解的向量，以及如何高效地存储和检索这些数据。

以下是本节课的详细梳理与复习指南。

---

### 📚 1. 课程核心摘要 (Executive Summary)

- **核心目标**：从海量非结构化文本数据中，根据用户模糊的查询（Query），检索出最相关的信息并进行排序 。
    
- **关键挑战**：与结构化数据库（Database）不同，文本检索面临语义模糊、查询不完整以及“相关性”难以定义的问题 。
    
- **三大核心步骤**：
    
    1. **编码 (Encoding)**：将文档和查询转化为向量（VSM, Bag of Words）。
        
    2. **索引 (Indexing)**：构建倒排索引（Inverted Index）以实现快速查找。
        
    3. **评分 (Scoring)**：使用 TF-IDF 和余弦相似度计算相关性并排序 。
        

---

### 📖 2. 详细内容梳理 (Detailed Walkthrough)

#### 2.1 文本检索 vs. 数据库检索

- **数据库 (SQL)**：数据结构化，查询精确，结果要么匹配要么不匹配（Exact match）。
    
- **文本检索 (IR)**：数据非结构化（自然语言），查询由用户意图决定（往往模糊），结果是基于**相关性 (Relevance)** 的排序列表 。
    
- **检索模型**：
    
    - **布尔模型 (Boolean)**：简单的 True/False 匹配（无排名）。
        
    - **向量空间模型 (Vector Space Model, VSM)**：本课重点。将文档和查询看作空间中的点，距离越近越相关 。
        
    - **概率模型 (Probabilistic)**：基于概率计算相关性。
        

#### 2.2 步骤一：编码与预处理 (Encoding & Preprocessing)

我们要把文本变成计算机能算的数字（向量）。

- **词袋模型 (Bag of Words, BoW)**：忽略语序，只看词频。
    
- **预处理流水线** ：
    
    1. **分词 (Tokenisation)**：将句子切分成单词。
        
    2. **去停用词 (Stop-word Removal)**：去掉 "the", "is", "at" 等无意义高频词。
        
    3. **词干提取 (Stemming/Lemmatisation)**：将 "jumped", "jumping" 还原为词根 "jump"。
        
- **齐普夫定律 (Zipf's Law)**：
    
    - 词频与排名的倒数成正比（第2常见的词频率是第1的一半）。
        
    - **特征选择策略**：去掉最高频的词（停用词，无区分度）和极低频的词（长尾词，噪音/稀疏性），只保留中间部分 。
        

#### 2.3 步骤二：索引 (Indexing)

面对数十亿文档，不能逐个遍历（效率太低）。我们需要**倒排索引 (Inverted Index)**。
这里的关键在于：**在文本检索中，绝大多数维度的数值都是 0。**

##### 1. “笨办法”：稠密向量点积 (Dense Calculation)

假设我们的词汇表有 **100,000 个单词**（维度 $N=100,000$）。![[Pasted image 20260516133955.png]]

我们要计算你的查询向量 $Q$ 和某篇文档向量 $D$ 的相似度（点积）。

公式是：

$$Q \cdot D = \sum_{i=1}^{100,000} (Q_i \times D_i)$$

- **你的查询**：只搜了 "Alice Cooper"（2 个词）。这意味着 $Q$ 中只有 **2 个位置是 1**，剩下 **99,998 个位置全是 0**。
    
- **计算过程**：
    
    - 第 1 次乘法：$Q_{\text{a}} \times D_{\text{a}}$ (0 $\times$ 0 = 0)
        
    - ...
        
    - 第 500 次乘法（Alice）：**$1 \times D_{\text{Alice}}$ (有意义)**
        
    - ...
        
    - 第 100,000 次乘法...
        

**结果**：你做了 100,000 次乘法和加法，但其中 **99,998 次都是在算 $0 \times \text{某数} = 0$**。这是巨大的浪费！而且如果你有 10 亿篇文档，你就要对每一篇都做这 10 万次运算。

---

##### 2. “聪明办法”：倒排索引 (Inverted Index)

这里的 **inverted index** 指的是 **倒排索引**，是文本检索 / 搜索引擎里最核心的数据结构。

一句话：

> **倒排索引就是：从“词 → 出现在哪些文档中”的索引**
> 
> 倒排索引让我们**只看非零元素**，完全跳过那些 0。
![[Pasted image 20260516134016.png]]
**逻辑流：**

1. **查询**："Alice", "Cooper"。
    
2. **查表**：
    
    - 谁含有 "Alice"？ $\rightarrow$ 文档列表 $L_1 = [D_0, D_1, D_2]$。
        
    - 谁含有 "Cooper"？ $\rightarrow$ 文档列表 $L_2 = [D_1]$。
        
3. **累加 (Accumulate)**：
    
    - 我们只处理出现在 $L_1$ 和 $L_2$ 里的文档。
        
    - **对于 $D_1$**：
        
        - 遇到 "Alice"：分数 $+ (Q_{\text{Alice}} \times D1_{\text{Alice}})$
            
        - 遇到 "Cooper"：分数 $+ (Q_{\text{Cooper}} \times D1_{\text{Cooper}})$
            
    - **对于 $D_{999}$**（它不含这两个词）：我们**看都不看它一眼**。
        

**结果**：

- 我们完全没去遍历那 100,000 个维度。我们只计算了命中的那几次乘法。
    
- 我们完全没去遍历数据库里的几亿篇无关文档。

- **正向索引**：文档 ID $\rightarrow$ 内容。
    
- **倒排索引**：单词 (Term) $\rightarrow$ 出现该词的文档列表 (Posting List) 。
    
    - 结构：Term ID $\rightarrow$ [(Doc ID, Frequency), (Doc ID, Frequency)...]
        
    - _例子_：查询 "Alice Cooper" 时，只需查找 "Alice" 的列表和 "Cooper" 的列表，然后求交集或累加分数，无需扫描全库 。
      这里例子中，求交集或累加分数的意义在于**衡量文档与查询的“相关性”并用于排序**。
    - 数学意义：向量点积 (Dot Product)
		在向量空间模型 (Vector Space Model) 中，我们通过计算两个向量的夹角余弦值来衡量相似度：

$$\text{Similarity}(Q, D) = \frac{Q \cdot D}{||Q|| \cdot ||D||}$$

其中，**$Q \cdot D$ (点积)** 正是这个“累加”的过程：

1. 假设查询向量 $Q$ 中，"alic" 和 "cooper" 的权重都为 1（简单起见）。
    
    $$Q = [\text{alic}: 1, \text{cooper}: 1]$$
    
2. 文档 1 的向量 $D_1$ 是：
    
    $$D_1 = [\text{alic}: 207, \text{cooper}: 398]$$
    
3. **点积计算**：
    
    $$Q \cdot D_1 = (1 \times 207) + (1 \times 398) = 207 + 398 = 605$$
    

**结论**：图中的 `207 + 398` 就是在计算分子部分的点积。

**总结**：累加分数的意义是**快速筛选出最匹配“Alice”和“Cooper”这两个概念组合的文档，并将其排在首位。**
        
#### 2.4 步骤三：权重与评分 (Weighting & Scoring)

单纯用词频（Raw Frequency）作为权重是不够的，因为常用词（如 "good"）频率高但信息量小。

- **TF-IDF 算法** ：
    
    - **TF (Term Frequency)**：词 $t$ 在文档 $d$ 中出现的频率。表示该词对**这篇文档**的重要性。
        
        - 公式变体：$tf(t,d) = \frac{f_{t,d}}{\sum f_{t',d}}$ (归一化) 。
            
    - **IDF (Inverse Document Frequency)**：逆文档频率。表示该词在**整个语料库**中的稀有程度。越稀有，IDF 越高。
        
        - 公式：$idf(t, D) = \log \frac{N}{n_t}$ ($N$是文档总数，$n_t$是包含词$t$的文档数) 。
            
    - **最终权重**：$Weight = TF \times IDF$。
        
- **余弦相似度 (Cosine Similarity)**：
    
    - 衡量查询向量 $q$ 和文档向量 $d$ 之间的夹角。
        
    - $Score(q,d) = \frac{q \cdot d}{||q|| \cdot ||d||}$。
        
    - 因为向量通常是稀疏的，我们通过倒排索引只计算非零元素，极大提高了速度 。
        

#### 2.5 高级概念：Web 搜索与反馈

- **链接分析 (PageRank)**：利用网页间的链接结构来判断重要性（将在后续课程详细讲）。
    
- **用户反馈 (User Feedback)**：点击率作为隐式反馈，用于调整排名 。
    
- **结果多样化 (Diversification)**：搜索 "Jaguar" 时，应同时展示“汽车”和“动物”的结果，而不是全是一种 。
    

---

### 🔔 3. 教师重点复述 (Teacher's Emphasis)

> **"You cannot prove mathematically what the best way... it is empirically defined."**
> 
> **“你无法从数学上证明某种文本检索方法是最好的……它是经验定义的。”**

- **出处**：
    
- **助教解读**：这是 IR 和 纯数学/数据库 的区别。没有绝对的“正确答案”，只有“用户觉得好不好”。这就是为什么我们需要测试和用户反馈。
    

> **"Vectors will have very high number of dimensions, but are very sparse."**
> 
> **“向量维数极高，但非常稀疏。”**

- **出处**：
    
- **助教解读**：这是一个考点。词表可能有 10万 个词，但一篇文档可能只有几百个词。这也是为什么我们需要**倒排索引**（只存储非零值）的原因。
    

> **"A high weight in TF-IDF is reached by a high frequency in a given document, but low frequency in the whole collection."**
> 
> **“TF-IDF 的高权重来自于：在当前文档中高频出现，但在整个文集中低频出现。”**

- **出处**：
    
- **助教解读**：这是 TF-IDF 的灵魂。必须记住：**高频词不一定重要，独特的词才重要**。
    

---

### 🔧 4. 关键术语与公式速查 (Terminology & Formulas)

|**术语 (CN/EN)**|**解释**|**关键公式/特征**|
|---|---|---|
|**词袋模型 (Bag of Words)**|忽略语序，将文档视为词频向量。|向量维度 = 词汇表大小 (Vocabulary Size)。|
|**停用词 (Stop Words)**|极其常见但在检索中缺乏区分度的词。|例如：the, a, is, of。|
|**词干提取 (Stemming)**|将单词还原为词根形式。|fishing, fished, fisher $\rightarrow$ fish。|
|**倒排索引 (Inverted Index)**|搜索引擎的核心数据结构，从词映射到文档列表。|Map: Term $\rightarrow$ List[DocID]。|
|**TF (Term Frequency)**|词频，衡量词在当前文档中的重要性。|$TF = \frac{count(t, d)}{size(d)}$ (常用变体)。|
|**IDF (Inv. Doc Frequency)**|逆文档频率，衡量词的全局稀有度。|$IDF = \log \frac{N}{df_t}$。|
|**余弦相似度 (Cosine Sim)**|衡量两个向量方向的一致性（内容相似度）。|$\cos(\theta) = \frac{A \cdot B}{|

---

### 📝 5. 复习与思考 (Review & Flashcards)

场景应用题 (Based on Learning Outcomes ):

- **场景**：你要设计一个搜索引擎。
    
- **问题 1**：如果只用词频 (Raw Count) 排序，会发生什么？
    
    - _答案_：长文档会仅仅因为包含更多词而排名靠前；常用词（如 "the"）会主导相关性计算。解决方法：使用归一化 TF 和 IDF。
        
- **问题 2**：为什么要移除极其稀有的词（Rare words）？
    
    - _答案_：根据 Zipf's Law，它们构成了长尾，对整体检索贡献小，但会显著增加向量维度和计算量（噪音）。
        

**自测闪示卡 (Flashcards):**

- **Q1: 在倒排索引中，什么是 Posting？**
    
    - _A1:_ 一个 Posting 是一个二元组 (Document ID, Frequency)，表示某个词在某文档中出现了多少次 。
        
- **Q2: 如果一个词在所有文档中都出现，它的 IDF 是多少？**
    
    - _A2:_ 接近 0。因为 $n_t \approx N$，$\log(1) = 0$。这说明该词无区分度。
        
- **Q3: 相比于数据库查询，文本检索的 Query 有什么特点？**
    
    - _A3:_ 模糊 (Ambiguous)、不完整 (Incomplete)、非结构化 。
        

---

### 💡 6. 助教建议 (Next Steps)

1. **实验练习**：LO2 要求实现搜索算法。请务必去跑一下提供的 `Inverted index ipynb demo` 和 `TF-IDF ipynb demo` 。尝试修改里面的 Query，看看倒排索引是如何一步步找出 Document ID 的。
    
2. **手动计算**：考试可能会给出 3 个短文档，让你手算某个词的 TF-IDF 值，或者计算两个文档向量的余弦相似度。**一定要熟练掌握 $TF \times IDF$ 的计算过程**。
    
3. **回顾上节课**：讲座最后回顾了 SOM 和 t-SNE 。如果你对 SOM 的“邻域更新”或 t-SNE 的“t分布防拥挤”还有疑问，可以回看这一段的转写，老师做了一个很棒的总结。
    

需要我为你生成一道 TF-IDF 的手算练习题吗？