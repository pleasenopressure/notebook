你好！我是本课程的 AI 助教。这节课的主讲是 Jennifer Williams 博士 ，她虽然带病上课 ，但为大家带来了一节干货满满的讲座。
以下是我为你整理的第 8 讲《词汇与向量语义学 (Lexical and Vector Semantics)》的完整学习指南。
### 课程概览 (精炼摘要)

本讲主要探讨了如何让计算机理解和表示语言的意义 。课程对比了侧重于语言学定义的“词汇语义学”与侧重于数值表示的“向量语义学” 。核心算法部分详细讲解了三种关键的计算方法：用于信息检索的 TF-IDF 、用于衡量向量间距离的余弦相似度 (Cosine Similarity) 、以及用于衡量词汇共现关系的逐点互信息 (PMI/PPMI) 。这些都是自然语言处理 (NLP) 和搜索引擎底层的基础技术 。

---

### 按章节详细讲解

**1. 词汇语义学 (Lexical Semantics) - Slide 3-4 / 0:47-6:59** 词汇语义学是对词义的语言学或逻辑学研究 。

- 词元 (Lemma) 是词的引用形式或“字典形式” 。
    
- 一个词元可以有多个词义 (Word senses)，例如 mouse 可以指啮齿动物或电脑鼠标，这需要词义消歧 (Word sense disambiguation) 技术 。
    
- 词形 (Wordform) 是词元在文本中的具体变形，例如 sing 的变形可以是 sung 或 sang，但它们保持相同的词义 。
- 同义词 (Synonym) 是词义相同或几乎相同的词，例如 dog 和 hound 。
- 词汇相似度 (Word Similarity) 指两个词具有相似的关系（如同属一个类别），例如 cat 和 dog 都是宠物，或者 doctor 和 nurse 都是医疗从业者，但它们不是同义词 。
- 词汇相关性 (Word Relatedness / Association) 指词汇共享共同的上下文，例如 tea 和 cup，或者 king 和 queen 。

**2. 向量语义学 (Vector Semantics) - Slide 5-6 / 7:08-12:50** 向量语义学利用嵌入 (Embeddings) 在多维空间中以数值形式表示词义 。
- 这是一种表示学习 (Representation learning)，通过大型语料库自动学习，比手工提取特征更强大 。
- 嵌入向量可以是稀疏的 (如 TF-IDF) 或密集的 (如 Word2Vec) 。
- 空间中距离相近的词（如 good 和 fantastic）表示它们相似 。

**3. 信息检索与 TF-IDF - Slide 7-14 / 12:50-27:17** 信息检索旨在找到与查询词匹配的文档，这需要构建文档向量和查询向量 。
- 普通搜索引擎注重精确率，而专利搜索（如查找发声生物识别专利）极其注重召回率 (Recall)，即找出所有可能相关的文档 。
- 词频 (TF) 指一个词在文档中出现的次数，但仅用 TF 会给予常用停用词（如 the, and）过高的权重 。
- 逆文档频率 (IDF) 衡量一个词在所有文档中出现的罕见程度 。
- TF-IDF 是两者的平衡，用于评估一个词对特定文档的重要程度 。![[Pasted image 20260223134853.png]]![[Pasted image 20260223135022.png]]
- 我们使用以 2 为底的对数 ($\log_2$)，这在自然语言处理的信息论指标中很常见，能产生更大范围的分数 。
**4. 余弦相似度 (Cosine Similarity) - Slide 15 / 27:25-28:25** 用于衡量两个向量在多维空间中的夹角/相似度 。
- 纯点积会偏袒较长的向量 。
- 余弦相似度通过向量长度进行归一化，解决了这个问题 。
**5. 逐点互信息 (PMI & PPMI) - Slide 16-17 / 28:25-36:00** 用于衡量两个词的共现频率与它们独立出现时的期望频率之间的差异 。
- 正 PMI 意味着它们共现的频率高于独立预期 。
- 负 PMI 通常不可靠（除非语料库极大），因此在 PPMI (Positive PMI) 中将负值替换为 0 。
- PMI 可用于发现同义词、进行查询扩展，或者识别特定类别的实体 。
![[Pasted image 20260223140409.png]]
---

### 教师重点复述 (Exam / Key Highlights)

- **区分同义词与词汇相似度**：老师明确指出，“如果我在考试中让你给出同义词或词汇相似度的例子，你需要知道它们之间的区别” 。**务必记住**：猫 (cat) 和狗 (dog) 是相似的 (Similar)，因为它们都是宠物；但它们不是同义词 (Synonyms) 。
- **哲学思考**：老师留下了一个贯穿职业生涯的思考题：“我们能否仅仅使用向量语义学，来完全捕捉词汇语义学的细微差别？” 。这提醒我们，虽然基于数学的向量很强大，但语言的内在复杂性有时可能超出了纯数值表示的范畴。
- **换底公式技巧**：在计算 IDF 或 PMI 时，由于常用计算器可能只提供 $\log_{10}$ 或 $\ln$，老师特意给出了换底公式 $\log_2(x) = \frac{\log_{10}(x)}{\log_{10}(2)}$ 并在考试可能的语境下强调了这一点 。
---

### 专有名词与核心公式解释

- **TF (Term Frequency)**：$tf_{t,d} = \log_2(\text{count}(t,d) + 1)$。加入对数是为了避免极端高频词（奖励）过度扭曲结果 。
    ![[Pasted image 20260223135234.png]]
    
- **IDF (Inverse Document Frequency)**：$idf_{t} = \log_2(\frac{N}{df_t})$。其中 $N$ 是总文档数，$df_t$ 是包含该词的文档数 。![[Pasted image 20260223135414.png]]
    
- **TF-IDF**：$w_{t,d} = tf_{t,d} \times idf_{t}$ 。
    
- **Cosine Similarity (余弦相似度)**：
    
    $$\text{cosine}(v, w) = \frac{v \cdot w}{|v||w|} = \frac{\sum_{i=1}^{N} v_i w_i}{\sqrt{\sum_{i=1}^{N} v_i^2} \sqrt{\sum_{i=1}^{N} w_i^2}}$$
    
    这是两个向量的点积除以它们长度的乘积 。
    
- **PPMI (Positive Pointwise Mutual Information)**：
    
    $$PPMI(w,c) = \max\left(\log_2\frac{P(w,c)}{P(w)P(c)}, 0\right)$$
    
    其中 $P(w,c)$ 是词汇和上下文共现的概率，$P(w)$ 和 $P(c)$ 是各自独立出现的概率 。
    

---

### Python 工程实现 (延伸补充)

虽然课程侧重理论，但在实际工程中，我们通常不需要手动计算 TF-IDF 或余弦相似度。可以使用 `scikit-learn` 快速实现：

Python

```
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.metrics.pairwise import cosine_similarity

# 给定文档
documents = ["Siri chatbot methods", "camera phone button", "blender plug"]

# 1. 计算 TF-IDF 矩阵
vectorizer = TfidfVectorizer()
tfidf_matrix = vectorizer.fit_transform(documents)

# 2. 获取词汇表
print("Vocabulary:", vectorizer.get_feature_names_out())

# 3. 计算文档1和文档2的余弦相似度
similarity = cosine_similarity(tfidf_matrix[0], tfidf_matrix[1])
print(f"Cosine Similarity between Doc 0 and Doc 1: {similarity[0][0]:.4f}")
```

---

