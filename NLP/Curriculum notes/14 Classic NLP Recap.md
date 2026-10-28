# Lecture 14 Classic NLP Recap — exam-focused revision note

## 第一部分：资料核对

|资料|使用情况|
|---|---|
|课程资料归类 mapping|已找到。Lecture 14 = **Recap Classic NLP**，建议 PDF 为 `L14_Recap_Classic_NLP.pdf`，定位为 Classic NLP 复习整合。mapping 中还把 L3 Regex、L4 n-Gram、L5–6 Sequence Labelling、L7 Grammar、L8 Vector Semantics 都归入 Classic NLP。|
|Slide/PDF|已找到并使用：`14-NLP-Classic-NLP-Recap.pdf`。Slide 明确覆盖：Regular expressions、n-gram language models、Constituency grammars、Lexical and vector semantics、Document retrieval TF-IDF。|
|Transcript|已上传：`14classic NLP recap.txt`。但内容明显不是 Classic NLP Recap，而是 **Transformer continuation**，开头说“today’s lecture is a continuation of the transformer lecture”，并讨论 self-attention、position embeddings、masked attention。 因此它和 Lecture 14 slide 冲突；本份笔记以 **slide + mapping + exam guidance + book** 为准。Lecture 14 的真实口头强调当前资料中未找到。|
|Reference book / directed reading|已使用 Jurafsky & Martin _Speech and Language Processing_ 3rd ed. draft。与本 lecture 相关章节包括 Chapter 2 Words/Regex/Edit Distance、Chapter 3 n-Gram LM、Chapter 5 Embeddings、Chapter 11 Retrieval、Chapter 17 Sequence Labelling、Chapter 18 CFG、Chapter 19 Dependency Parsing。|
|Exam guidance / revision|已使用。考试为 2 小时，5 题，答 3 题；Section A 必答 2 题，其中一题 classic NLP，一题 neural NLP；每题结构为 5 marks lecture recall、10 marks worked example/lab/book、10 marks directed/wider reading。|
|Past exam|已使用。2025 past exam Section A1 直接考了 n-gram smoothing、PPMI、sequence tagger/cross-validation；Section B1 考 Python regex。|

**本 lecture 分类：Classic NLP。**  
它是 Classic NLP 的 recap/revision-style lecture，但按课程内容类别应放在 **Classic NLP**，不是 Neural NLP，也不是 LLM/Downstream Application。

---

## 第二部分：考试定位

### 最可能出现位置

**Section A Classic NLP。**

原因很直接：revision slide 明确写了 **Part A always 1 classic NLP focussed question**，Classic NLP 范围包括 n-grams、MLE、smoothing、BIO/NER/POS/CRF、tokenization、TF-IDF、PPMI、CFG、dependency grammar、projectivity 等。 Exam guidance 也说明 Section A 必有一题 classic NLP。

### 考试优先级

**A 级：必须掌握，最可能考**

- 【高频】【数学题】n-gram LM：MLE、smoothing、perplexity。
    
- 【高频】【数学题】PPMI / TF-IDF / cosine similarity。
    
- 【高频】【Lab重点】Regex，尤其 Python regex syntax、greedy vs non-greedy、escaping、groups、lookaround。
    
- 【高频】Sequence labelling：POS、NER、BIO、CRF、tagger comparison、cross-validation。
    
- 【高频】CFG vs dependency grammar、ambiguity、projectivity。
    

**B 级：需要理解，可能考**

- Lexical semantics vs vector semantics。
    
- Synonym / similarity / relatedness 的区别。
    
- Treebanks and head finding。
    
- High precision vs high recall retrieval 场景，例如 regular search vs patent search。Slide 明确用 TF-IDF 解释 regular search 高 precision、patent search 高 recall。
    

**C 级：了解即可**

- regex 背后的 deterministic finite automaton 理论。
    
- Penn Treebank 具体数据集细节。
    
- 语义场、topic models 的扩展讨论。
    
- 具体历史人物/数据集年份。
    

---

## 第三部分：核心知识点

### 1. Regular Expressions

**Definition**  
Regular expression 是一种描述 string pattern 的形式语言，用于在文本中匹配、抽取或替换符合规则的字符序列。Lecture 14 recap 重点包括 alternatives/groups、greedy vs non-greedy、special escape。

**Intuition**  
Regex 不“理解”语言，只做 deterministic pattern matching。它适合处理格式规律明显的文本，例如 phone number、email、date、ID、文件名。

**Input / Output**  
Input 是 raw string；output 是 matched span、captured group、boolean match、替换后的 string，或被抽取出的列表。

**Why it matters in NLP**  
Regex 是很多 NLP preprocessing 的基础：cleaning、tokenization、rule-based extraction、lab 中文本处理。Past exam B1 明确说明会用 Python 3 regex syntax。

**Strengths / limitations**  
Strengths：快、可解释、无需训练数据、适合规则明确任务。  
Limitations：对语义、上下文、模糊表达不强；复杂 regex 难维护；容易出现 greedy overmatching；不同实现中 escaping 规则不同。Slide 特别提醒 Python 中要注意 escape special characters。

**Relation to other lectures**  
连接 Lecture 2 Words/tokenization、Lecture 5–6 sequence labelling、Lecture 17 IE。Regex 可以作为 rule-based baseline，也可以辅助生成 weak labels。

---

### 2. n-Gram Language Models

**Definition**  
n-gram LM 用前 $$N-1$$ 个 token 近似预测当前 token 的概率。Slide 明确说明 N=2 是 bigram，N=3 是 trigram，N 指 n-gram tuple 的总长度，不是 vocabulary 中 n-gram 数量。

**Formula**

$$P(w_n|w_1,\dots,w_{n-1}) \approx P(w_n|w_{n-N+1},\dots,w_{n-1})$$`

**Intuition**  
完整历史太长、太稀疏，所以只看最近几个词。比如 “take an ____” 中 umbrella 比 pizza 更可能。

**Input / Output**  
Input：token sequence。  
Output：next-word probability，或 whole sentence probability。

**Why it matters in NLP**  
语言模型是 speech recognition、spelling correction、machine translation、text generation 的基础。Classic LM 也帮助理解后面 neural LM / LLM 的概率建模思想。

**Strengths / limitations**  
Strengths：简单、可解释、可手算、适合考试 worked example。  
Limitations：long-range dependency 弱；data sparsity 严重；unseen n-gram 会导致 zero probability，需要 smoothing。

**Relation to other lectures**  
连接 Lecture 4 n-Gram、Lecture 10 RNN、Lecture 12–13 Transformer、Lecture 15 LLM。Perplexity 在 classic LM 和 modern LM 中都重要。

---

### 3. Maximum Likelihood Estimation and Smoothing

**Definition**  
MLE 通过 corpus counts 估计 n-gram probability。Slide 说明 MLE 是观察 n-gram counts，并通过 normalization 让概率落在 0 到 1 之间。

**Bigram MLE**

$$P(w_n|w_{n-1})=\frac{C(w_{n-1},w_n)}{C(w_{n-1})}$$`

**General n-gram MLE**

$$P(w_n|w_{n-N+1:n-1})=\frac{C(w_{n-N+1:n-1},w_n)}{C(w_{n-N+1:n-1})}$$`

**Smoothing definition**  
Smoothing / discounting 是为了避免 unseen n-gram 得到 zero probability。Slide 解释为从高频 n-gram 拿走一点 probability mass，分给 unseen n-grams。

**Laplace smoothing**

$$P_{\text{Laplace}}(w_n|h)=\frac{C(h,w_n)+1}{C(h)+|V|}$$`

其中：

- $$h$$` = history/context。
    
- $$C(h,w_n)$$` = context 后接当前词的 count。
    
- $$C(h)$$` = context 出现次数。
    
- $$|V|$$` = vocabulary size。
    
- $$+1$$` = 给所有可能词加一个伪计数。
    

**Strengths / limitations**  
Laplace smoothing 简单，但会过度惩罚 seen n-grams，尤其 vocabulary 很大时，会把太多 probability mass 分给实际上很不可能的词。

---

### 4. Perplexity

**Definition**  
Perplexity 衡量 LM 对 test set 的预测能力。Slide 解释为 “How surprised is this LM?”，高 PP = bad LM，低 PP = good LM。

**Formula**

$$PP(W)=\sqrt[N]{\prod_{i=1}^{N}\frac{1}{P(w_i|w_{i-1})}}$$`

等价写法：

$$PP(W)=\left(\prod_{i=1}^{N}P(w_i|w_{i-1})\right)^{-\frac{1}{N}}$$`

每个项含义：

- $$W$$` = test sequence。
    
- $$N$$` = number of predicted tokens。
    
- $$P(w_i|w_{i-1})$$` = bigram model 给出的当前词概率。
    
- $$\prod$$` = 把每一步预测概率连乘。
    
- $$-\frac{1}{N}$$` = 做长度归一化，避免长句概率天然更小。
    

**Exam point**  
【易错】Raw probability 会随 sentence length 变长而越来越小，所以不能直接比较不同长度句子的概率；perplexity 做了 length normalization。

---

### 5. Constituency Grammar and CFG

**Definition**  
Constituency grammar 把句子分解成 constituents，例如 NP、VP、PP。CFG 使用 production rules 生成 parse tree。Slide 展示了 “I prefer the morning flight through Denver” 的 CFG parse tree。

**Intuition**  
句子不是词的平铺，而是层级结构。比如 “the morning flight” 可以整体作为 NP。

**Input / Output**  
Input：sentence + grammar rules + lexicon。  
Output：parse tree。

**Why it matters in NLP**  
Syntax 帮助处理 ambiguity、relation extraction、semantic parsing、question answering。

**Strengths / limitations**  
Strengths：结构显式、可解释、高风险场景中比黑盒模型更可控。  
Limitations：grammar engineering 成本高；ambiguity 会产生多个 parse trees；coverage 可能不足。

**Relation to other lectures**  
连接 Lecture 7 constituency grammar、Lecture 18 SRL、Lecture 17 IE。Parsing 提供结构，IE/SRL 使用结构帮助判断 argument。

---

### 6. Dependency Grammar and Projectivity

**Definition**  
Dependency grammar 把句子表示为 directed graph：nodes 是 words，arcs 是 grammatical relations。Slide 定义 dependency tree 为 `G=(V,A)`，其中 `V` 是 vertices/nodes，`A` 是 arcs/grammatical-function relationships。

**Intuition**  
CFG 问“哪些词组成短语”；dependency grammar 问“哪个词依赖哪个词”。例如 verb 是 head，subject/object 是 dependent。

**Input / Output**  
Input：sentence。  
Output：dependency tree with labelled arcs。

**Why it matters in NLP**  
Dependency 更接近 predicate-argument structure，常用于 relation extraction、semantic role labelling、information extraction。

**Strengths / limitations**  
Strengths：适合自由词序语言；head-dependent relation 清晰。  
Limitations：projectivity、non-projective arcs、attachment ambiguity 都可能造成解析困难。

**Projectivity**  
如果 head 和 dependent 之间所有中间词都能由 head 连接覆盖，则 arc 是 projective。Slide 给出 not projective 的例子。

---

### 7. Lexical Semantics vs Vector Semantics

**Definition**  
Lexical semantics 研究 word meaning；vector semantics 用 numerical vector 表示 word meaning。Slide 明确区分二者，并问 vector semantics 是否能完全捕捉 lexical semantics 的 nuance。

**Key distinctions**

- Synonym：sense identical or nearly identical，例如 dog / hound。
    
- Similarity：语义类别相似，但不一定同义，例如 cat / dog。
    
- Relatedness：共享语境但不相似，例如 tea / cup。
    
- Semantic field：同一 domain 的相关词集合。
    

**Input / Output**  
Input：word、document、co-occurrence counts。  
Output：word vector、document vector、similarity score。

**Why it matters in NLP**  
它支持 IR、classification、clustering、semantic search、word similarity、embeddings。

**Strengths / limitations**  
Strengths：可计算 similarity；能用于 retrieval 和 classifier。  
Limitations：vector 不一定能捕捉 polysemy、pragmatics、world knowledge、sense distinction。

**Relation to other lectures**  
连接 Lecture 8 vector semantics、Lecture 9 Word2Vec、Lecture 20 QA/RAG retrieval。

---

### 8. TF-IDF and Document Retrieval

**Definition**  
TF-IDF 平衡 term frequency 和 inverse document frequency。Slide 说明 TF alone 不够区分文档，IDF alone 会偏向几乎不出现的词。

**Formula**

$$tfidf(t,d)=tf(t,d)\times idf(t)$$`

Book 中给出的 log TF：

$$tf_{t,d}= \begin{cases} 1+\log_{10}(count(t,d)), & count(t,d)>0\\ 0, & count(t,d)=0 \end{cases}$$`

IDF：

$$idf_t=\log_{10}\frac{N}{df_t}$$`

Cosine scoring：

$$score(q,d)=cos(q,d)=\frac{q\cdot d}{|q||d|}$$`

Book 明确说明 TF-IDF retrieval 会把 query 和 document 表示成 weighted vectors，然后用 cosine similarity 给文档打分。

**Input / Output**  
Input：query + document collection。  
Output：ranked documents。

**Why it matters in NLP**  
IR 是 QA/RAG 的基础。TF-IDF 虽然 classic，但在 high precision / high recall search 中仍然有价值。Slide 特别用 patent search 说明需要 high recall。

**Strengths / limitations**  
Strengths：简单、快、可解释、无需 supervised data。  
Limitations：需要 exact word overlap；难处理 synonymy 和 vocabulary mismatch。Book 指出 classic TF-IDF/BM25 的概念缺陷是 query 和 document 必须有 exact word overlap。

---

## 第四部分：公式、算法、architecture

### A. n-gram LM pipeline

```text
Raw corpus
↓
Tokenization
↓
Add <s> and </s>
↓
Count unigrams / bigrams / trigrams
↓
Estimate probabilities using MLE
↓
Apply smoothing if needed
↓
Compute sentence probability or perplexity
```

Core formulas:

$$P(w_n|w_{n-1})=\frac{C(w_{n-1},w_n)}{C(w_{n-1})}$$`

$$P(w_1,\dots,w_N)=\prod_{i=1}^{N}P(w_i|w_{i-1})$$`

$$P_{\text{Laplace}}(w_i|w_{i-1})=\frac{C(w_{i-1},w_i)+1}{C(w_{i-1})+|V|}$$`

$$PP(W)=\left(\prod_{i=1}^{N}P(w_i|w_{i-1})\right)^{-\frac{1}{N}}$$`

---

### B. PPMI formula

$$PMI(w,c)=\log_2\frac{P(w,c)}{P(w)P(c)}$$`

$$PPMI(w,c)=\max(PMI(w,c),0)$$`

含义：

- $$w$$` = target word。
    
- $$c$$` = context word。
    
- $$P(w,c)$$` = word 和 context 共现概率。
    
- $$P(w)P(c)$$` = 如果二者独立时的期望共现概率。
    
- PMI > 0：二者比随机独立更常一起出现。
    
- PPMI 把负值截断为 0。
    

---

### C. TF-IDF retrieval architecture

```text
Query text
   ↓
Tokenize / normalize
   ↓
Query vector q using TF-IDF
   ↓
Compare with each document vector d
   ↓
Cosine similarity score
   ↓
Rank documents
   ↓
Return top N
```

Formula:

$$tfidf(t,d)=tf(t,d)\times idf(t)$$`

$$idf_t=\log_{10}\frac{N}{df_t}$$`

$$score(q,d)=\frac{q\cdot d}{|q||d|}$$`

---

### D. CFG parsing architecture

```text
Sentence
  "I prefer the morning flight"
        ↓
Lexicon
  I=Pronoun, prefer=Verb, the=Det, morning=Noun, flight=Noun
        ↓
Grammar rules
  S -> NP VP
  NP -> Pronoun
  VP -> Verb NP
  NP -> Det Nominal
  Nominal -> Nominal Noun | Noun
        ↓
Parse tree
        S
      /   \
     NP    VP
     |    /  \
    I  prefer NP
              |
          the morning flight
```

---

### E. Regex pipeline

```text
Raw text
↓
Design regex pattern
↓
Test with examples and edge cases
↓
Use re.search / re.findall / re.sub
↓
Inspect captured groups
↓
Fix escaping / greediness / boundaries
↓
Return extracted spans or cleaned text
```

【易错】`.*` is greedy；`.*?` is non-greedy。  
【易错】在 Python 中，反斜杠最好使用 raw string，例如 `r"\d+"`。

---

### Neural model requirement

本 lecture 本身是 **Classic NLP Recap**，没有新的 neural architecture。因此 input tensor shape、embedding shape、hidden/output shape、classifier head、softmax、loss function、training objective 当前 lecture 中未展开。  
不过 past exam 会把 classic 与 neural 分开考；不要把本 lecture 的 TF-IDF/PPMI/CFG 写成 BERT/Transformer 结构题。

---

## 第五部分：worked examples

### Worked Example 1：PPMI calculation【数学题】【高频】

Past exam A1.b 给出 co-occurrence matrix，并要求计算 “natural” 和 “language” 的 PPMI。表中：

- $$count(natural,language)=45$$`
    
- $$count(natural)=75$$`
    
- $$count(language)=165$$`
    
- $$N=310$$`
    

Step 1：

$$P(w,c)=P(natural,language)=\frac{45}{310}$$`

Step 2：

$$P(w)=P(natural)=\frac{75}{310}$$`

Step 3：

$$P(c)=P(language)=\frac{165}{310}$$`

Step 4：

$$PMI(natural,language)=\log_2\frac{45/310}{(75/310)(165/310)}$$`

Step 5：

$$=\log_2\frac{45\times310}{75\times165}$$`

Step 6：

$$=\log_2\frac{13950}{12375}$$`

Step 7：

$$=\log_2(1.1273)$$`

Step 8：

$$\approx 0.172$$`

Step 9：

$$PPMI=\max(0.172,0)=0.172$$`

**Answer:** $$PPMI(natural,language)\approx0.172$$`

【易错】不要用 raw count 直接算；必须先转成概率或用等价 count ratio。

---

### Worked Example 2：Bigram sentence probability【高频】

Slide 中比较：

Sentence A: `<s> i want english food </s>`  
Sentence B: `<s> i want chinese food </s>`

已知：

$$P(i|<s>)=0.25$$`

$$P(want|i)=0.33$$`

$$P(english|want)=0.0011$$`

$$P(food|english)=0.5$$`

$$P(</s>|food)=0.68$$`

Sentence A:

$$P(A)=0.25\times0.33\times0.0011\times0.5\times0.68=0.000031$$`

Sentence B:

$$P(B)=0.25\times0.33\times0.0065\times0.52\times0.68=0.000190$$`

Conclusion：Sentence B 更 probable。Slide 的 worked example 也显示 Chinese food 句子的概率更高。

---

## 第六部分：Lab / code 考法

### 本 lecture 对应 lab 判断

根据课程资料归类，lab 包括 Regex、CRF、BERT、LLaMA。 对 Lecture 14 Classic NLP Recap 来说，最相关的是：

- 【Lab重点】Regex lab
    
- 【Lab重点】CRF / sequence labelling lab
    
- BERT / LLaMA 属于 neural/LLM，不是本 lecture 重点
    

---

### 1. 代码阅读题：Regex

**Question**  
Python regex:

```python
import re
text = "Hello there! It's nice to see you! Goodbye!"
m = re.search(r"H.*!", text)
print(m.group())
```

**Answer**  
输出：`Hello there! It's nice to see you! Goodbye!`

**思路**  
`.*` 是 greedy，会尽可能多匹配，直到最后一个 `!`。

---

### 2. Bug identification：Regex escaping

**Question**  
下面代码想匹配 literal dot，但结果匹配了任意字符：

```python
re.findall(".", "a.b")
```

问题是什么？

**Answer**  
`.` 在 regex 中表示 any character。要匹配 literal dot，应写：

```python
re.findall(r"\.", "a.b")
```

**思路**  
Slide 明确提醒 special character 要用 backslash 恢复普通含义，尤其在 Python 中要注意 escaping。

---

### 3. Output shape 判断题：Sequence labelling

**Question**  
输入 tokens：

```text
["Richard", "Nixon", "visited", "London"]
```

使用 BIO NER tagging，输出长度是多少？

**Answer**  
输出长度 = 4，因为 sequence tagging 是 one label per token。

Example：

```text
Richard/B-PER Nixon/I-PER visited/O London/B-LOC
```

**思路**  
Sequence labelling input/output 长度相同；BIO 把 multi-word entity 转成 token-level labels。Sequence labelling lecture 明确说输入 `X=x1...xn`，输出 `Y=y1...yn`。

---

### 4. 参数 / shape 计算题：TF-IDF matrix

**Question**  
有 6 个 terms，4 个 documents，term-document matrix shape 是多少？

**Answer**  
$$(6,4)$$`

**思路**  
Rows = terms，columns = documents。Slide 中 TF-IDF 页面展示了 term-document matrix。

---

### 5. Training pipeline 排序题：CRF sequence tagger

**Question**  
把 CRF NER training pipeline 排序：

A. Extract token-level features  
B. Tokenize sentences  
C. Train CRF  
D. Convert named entities to BIO labels  
E. Evaluate with precision/recall/F1

**Answer**  
B → D → A → C → E

**思路**  
先有 token，再有 gold BIO labels，再抽特征训练，最后评估。

---

## 第七部分：Reference book / wider reading 整合

### Slide 没展开但 book 能补强的点

1. **Tokenization / Regex / BPE / Edit Distance**  
    Book Chapter 2 不只讲 regex，还讲 Unicode、BPE、minimum edit distance。它能帮助你在 Section A classic NLP 中把 regex 放到 preprocessing/tokenization 的大背景里。
    
2. **n-Gram smoothing and perplexity**  
    Slide 讲了 MLE、smoothing、perplexity；Book Chapter 3 更系统地讲 training/test split、generalization vs overfitting、interpolation/backoff。
    
3. **Vector semantics / embeddings**  
    Book Chapter 5 补充 simple count-based embeddings、cosine similarity、Word2Vec、bias and evaluation。
    
4. **TF-IDF and retrieval**  
    Book Chapter 11 讲 TF-IDF、cosine scoring，也补充 dense retrieval 和 vocabulary mismatch problem。这个可以作为 10 marks wider reading 的 stronger point：classic sparse retrieval 可解释、便宜，但 exact lexical overlap 是短板；dense retrieval 可缓解 synonymy/vocabulary mismatch。
    
5. **CFG / dependency parsing**  
    Book Chapter 18/19 覆盖 CFG、treebanks、CKY、dependency relations、transition-based parsing、graph-based parsing。即使 slide 只 recap 基本概念，book 能让你在 wider reading 中写 parser evaluation、ambiguity、projectivity。
    

---

### 可写进 10 marks wider reading 的 stronger points

- TF-IDF/BM25 的核心问题是 **vocabulary mismatch**：query 必须猜中文档作者使用的词。Dense retrieval 用 embeddings 缓解 synonymy。
    
- Regex 是 deterministic 和可解释的，但不适合语义 ambiguity；可以作为 rule-based baseline 或 preprocessing。
    
- n-gram 的 Markov assumption 牺牲 long-range context 来换可估计性；Transformer 后来用 self-attention 解决长距离依赖。
    
- CFG 显式表达结构，LLM 可能 fluent but fail silently；formal grammar 更适合 high-stakes 需要 explicit failure 的场景。
    
- Sequence labelling 可以从 rule-based、probabilistic、CRF 到 neural models；不同方法 trade-off 在 data requirement、interpretability、performance。
    

---

## 第八部分：可能考试题预测

### Q1. [5 marks recall]

**Name two smoothing techniques for n-gram language models and explain how they work.**

中文答题思路：  
写 Laplace/add-one smoothing 和 backoff/interpolation。说明 smoothing 是为了解决 unseen n-gram 的 zero probability。Laplace 是所有 count +1；backoff 是高阶 n-gram 不可靠时退回低阶模型；interpolation 是加权混合 unigram/bigram/trigram。

---

### Q2. [10 marks worked example]

**Given a co-occurrence matrix, calculate the PPMI of two terms.**

中文答题思路：  
写公式 $$PPMI=\max(\log_2(P(w,c)/(P(w)P(c))),0)$$`，从表中找 joint count、row total、column total、overall total，代入计算。Past exam 已经考过类似题。

---

### Q3. [10 marks worked example]

**Use Python regex syntax to extract phone numbers / dates / emails from raw text. Explain greedy vs non-greedy matching.**

中文答题思路：  
写 pattern，解释 `\d`、`\s`、`[]`、`()`, `|`, `?`, `*`, `+`。指出 `.*` greedy，`.*?` non-greedy。注意 Python raw string 和 escaping。

---

### Q4. [10 marks wider reading]

**Compare TF-IDF sparse retrieval with dense embedding retrieval.**

中文答题思路：  
TF-IDF：term-document matrix、IDF 降低 stopword 权重、cosine similarity 排序、可解释、快。缺点：exact word overlap / vocabulary mismatch。Dense retrieval：embedding 表示语义，可处理 synonymy，但训练/计算成本高、可解释性低。

---

### Q5. [10 marks wider reading]

**Compare constituency grammar and dependency grammar, using an example of ambiguity.**

中文答题思路：  
CFG 是 phrase-structure / constituent hierarchy；dependency 是 head-dependent directed graph。用 “old men and women” 或 “We saw the Eiffel Tower flying to Paris” 解释 ambiguity。说明 dependency grammar 对自由词序语言更灵活，CFG 更适合显式 phrase structure。

---

### Q6. [10 marks worked example]

**Given a small corpus, compute bigram probabilities and sentence probability.**

中文答题思路：  
先 count bigrams/unigrams，再用 MLE 算 $$P(w_i|w_{i-1})$$`，最后连乘。遇到 0 count 要 smoothing。

---

### 最可能考的一题：完整英文 exam-style answer

**Question:**  
Explain how TF-IDF represents documents for information retrieval. Include the roles of term frequency and inverse document frequency, describe how cosine similarity is used to rank documents, and discuss one strength and one limitation of TF-IDF compared with dense retrieval.

**Answer:**  
TF-IDF is a classic sparse vector representation for information retrieval. Each document is represented as a vector over a vocabulary of terms. The value of each term in a document is the product of its term frequency and inverse document frequency:

$$tfidf(t,d)=tf(t,d)\times idf(t)$$`

Term frequency measures how often a term occurs in a document, so terms that appear more often in a document are treated as more representative of that document. In practice, log weighting can be used so that very frequent words are not rewarded too strongly. Inverse document frequency measures how rare or discriminative a term is across the whole collection:

$$idf_t=\log_{10}\frac{N}{df_t}$$`

where $$N$$` is the number of documents and $$df_t$$` is the number of documents containing term $$t$$`. A term that appears in almost every document receives a low IDF score, while a term that appears in only a few documents receives a higher score.

For retrieval, the query is also represented as a TF-IDF vector. Each document is scored by cosine similarity between the query vector and the document vector:

$$score(q,d)=\frac{q\cdot d}{|q||d|}$$`

Documents with higher cosine similarity are ranked higher because their weighted term vectors are geometrically closer to the query vector.

A strength of TF-IDF is that it is simple, efficient, interpretable, and does not require labelled training data. This makes it useful for classic search settings and high-recall search tasks. A limitation is that it relies on exact lexical overlap. If the query uses a synonym or different wording from the document, TF-IDF may fail to retrieve the relevant document. Dense retrieval methods use embeddings to reduce this vocabulary mismatch problem, but they are more computationally expensive and less interpretable.

---

## 第九部分：考前速记版

### 5–10 个 must-know points

1. 【高频】Section A 一定有一道 Classic NLP 题。
    
2. 【高频】n-gram LM = Markov assumption + MLE counts + smoothing + perplexity。
    
3. 【数学题】PPMI 必须用 joint probability 和 marginal probability，不是直接用 raw count。
    
4. 【数学题】TF-IDF = TF × IDF；retrieval 用 cosine similarity 排序。
    
5. 【Lab重点】Regex 要会 Python syntax，尤其 `|`, `()`, `[]`, `\d`, `\w`, `.*`, `.*?`, lookahead/lookbehind。
    
6. 【易错】`cat|dog person` 和 `(cat|dog) person` scope 不一样。
    
7. 【易错】`.*` greedy 会尽可能匹配到最后；`.*?` non-greedy 会尽早停止。
    
8. 【高频】CFG 是 constituent hierarchy；dependency grammar 是 head-dependent graph。
    
9. 【高频】Lexical semantics ≠ vector semantics；synonym ≠ similarity ≠ relatedness。
    
10. 【高频】Sequence labelling input/output 长度相同；NER 常用 BIO tagging。
    

### 3–5 个 high-risk mistakes

1. 【易错】把 `N` in n-gram 误解为 vocabulary 中 n-gram 数量；它其实是 tuple length。
    
2. 【易错】忘记 smoothing，导致整句 probability 因一个 unseen bigram 变成 0。
    
3. 【易错】PPMI 中忘记 `max(PMI,0)`。
    
4. 【易错】TF-IDF 中认为 TF 越高一定越好；stopwords 高频但通常不 discriminative。
    
5. 【易错】把 dependency grammar 画成 CFG hierarchy；dependency nodes 是 words，edges 是 grammatical relations。
    

### 3–5 个 likely exam questions

1. Calculate PPMI from a co-occurrence matrix。
    
2. Explain n-gram smoothing and calculate sentence probability。
    
3. Write/debug a Python regex and explain greedy vs non-greedy。
    
4. Compare TF-IDF retrieval with dense retrieval。
    
5. Explain CFG vs dependency grammar with ambiguity example。
    

### 本 lecture 最应该背的 formula / algorithm

**n-gram MLE**

$$P(w_n|w_{n-1})=\frac{C(w_{n-1},w_n)}{C(w_{n-1})}$$`

**Laplace smoothing**

$$P_{\text{Laplace}}(w_n|h)=\frac{C(h,w_n)+1}{C(h)+|V|}$$`

**Perplexity**

$$PP(W)=\left(\prod_{i=1}^{N}P(w_i|w_{i-1})\right)^{-\frac{1}{N}}$$`

**PPMI**

$$PPMI(w,c)=\max\left(\log_2\frac{P(w,c)}{P(w)P(c)},0\right)$$`

**TF-IDF + cosine**

$$tfidf(t,d)=tf(t,d)\times idf(t)$$`

$$score(q,d)=\frac{q\cdot d}{|q||d|}$$`

### Transcript 中最值得记住的老师提醒

当前上传的 transcript 与本 lecture 不匹配，它讨论的是 Transformer block、masked self-attention、position embeddings 等内容，而不是 Classic NLP Recap。  
所以本 lecture 的真实 transcript 口头强调：**当前资料中未找到**。从 slide 可确认的重点提醒是：

- 【老师强调/Slide重点】Python regex 要特别注意 escaping special characters。
    
- 【老师强调/Slide重点】Perplexity 要熟悉，因为后续 LLM 也会用到。