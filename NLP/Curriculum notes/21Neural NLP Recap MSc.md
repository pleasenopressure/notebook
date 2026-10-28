# Lecture 21 — Neural NLP Recap MSc：Exam-focused revision note

## 第一部分：资料核对

本次使用资料如下：

1. **课程资料归类 mapping**  
    Lecture 21 对应为 **Recap Neural NLP**，建议 PDF 为 `L21_Recap_Neural_NLP.pdf`，内容覆盖 **Word2Vec, RNN, encoder-decoder, attention, Transformer, LLM, IE, QA**。它属于 **Revision + Neural NLP recap**，同时连接部分 downstream applications。
    
2. **Slide/PDF**  
    使用 `21-NLP-Neural-NLP-Recap-MSc.pdf`。Slide 明确说明本讲 recap：Word2Vec、Encoder-Decoder + attention、Transformer、BERT/LLaMA/decoding/distillation、RE/SRL、QA/RAG。
    
3. **Transcript**  
    使用你上传的 `21-NLP-Neural-NLP-Recap-MSc.txt`。老师说明这节课不是重新讲新内容，而是快速 recap + Q&A，重点是 neural network side，并多次强调 worked examples、tensor shapes、attention maths、BERT lab、model head。
    
4. **Reference book / directed reading**  
    本 lecture 的 mapping 没有给出一个单独章节，而是要求复习 Neural NLP 相关内容。结合教材目录，最相关的是 Jurafsky & Martin：Chapter 5 Embeddings、Chapter 7 LLMs、Chapter 8 Transformers、Chapter 9 Masked Language Models、Chapter 11 Retrieval/RAG、Chapter 12 MT、Chapter 13 RNNs/LSTMs。  
    具体 lab notebook 原文当前资料中未找到；本 note 只根据 slide/transcript 中对 BERT/LLaMA lab 的说明推断代码考法。
    

**资料冲突说明：**  
Transcript 中有一些明显 ASR 转写错误，例如 “word to vehicle” 应为 **Word2Vec**，“mouse model” 很可能应为 **masked model**。另外 transcript 一处说 “Dense embeddings will change based on words around”，这与 slide/book 中 “dense vs sparse” 和 “static vs contextual” 的标准区分不一致。正确理解应是：**dense 不等于 contextual；Word2Vec 是 dense but static，BERT 是 dense and contextual**。这一点老师也口头强调不要混淆。

---

## 第二部分：考试定位

### 1. 最可能出现位置

**Section A Neural NLP：极高概率相关。**  
Exam guidance 说明 Section A 一定有一个 classic NLP question 和一个 neural NLP question；题型结构是 5 marks lecture recall、10 marks lecture/lab/book worked example、10 marks directed/wider reading。 Revision slide 也明确列出 Part A Neural NLP 覆盖：attention mechanisms、word2vec、RNN、Transformer、training、architectures、layer shapes、dense embeddings、static vs contextual。

**Section B application or mixed topic：也很可能相关。**  
因为本 lecture 后半部分 recap 了 RE、SRL、QA、RAG，这些在 revision slide 的 Part B 中属于 Applications of NLP、Generation、Span Labelling、LLMs。

### 2. 考试优先级

**A 级：必须掌握。**

判断依据：

- **Revision lecture 出现：** 这是专门的 Neural NLP recap，覆盖整个 neural half。
    
- **Exam guidance：** Section A 一定有 neural NLP；并且 10 marks 很可能要求 worked example / architecture diagram / maths / lab。
    
- **Slide 展开重点：** Word2Vec、encoder-decoder、attention、Transformer block、positional embedding、LM head、decoding、BERT QA、RAG 都有专门 slide。
    
- **Transcript 老师强调：** 老师多次说要会 worked examples、tensor shapes、Q/K/V maths、masking、BERT classifier head、通过 head 判断任务类型。
    
- **Lab：** BERT lab 被明确提到用于 attention tensor shapes；LLM lecture 也提到 BERT and LLaMA labs。
    
- **Reference book：** Book 覆盖 embeddings、Transformers、masked LMs、RAG、RNN/encoder-decoder 等章节。
    

---

## 第三部分：核心知识点

### 1. 【高频】Static vs contextual embeddings

**Definition：**  
Static embedding 是一个 word type 对应一个固定向量，例如 Word2Vec/GloVe。Contextual embedding 是同一个 token 在不同句子中会得到不同向量，例如 BERT/Transformer hidden states。

**Intuition：**  
Word2Vec 只学习 “这个词通常和哪些词一起出现”；BERT 会根据整句话动态理解这个词。比如 “bank” 在 “river bank” 和 “bank account” 中应有不同表示。

**Input/output：**  
Input 可以是 one-hot token 或 token id；output 是 dense vector。Word2Vec 输出 $$d$$ 维静态向量；BERT 输出每个 token 的 contextual hidden state，shape 通常是 $$N_{seq} \times d_{model}$$。

**Why matters：**  
这是 past paper 风格高频题。2025 past paper A2.a 就考过 static vs contextual embedding。

**Strengths / limitations：**  
Static embedding 快、简单、可迁移，但不能处理 polysemy。Contextual embedding 表达能力强，但计算贵、依赖大模型。

**和其他 lecture 关系：**  
L8 vector semantics → L9 Word2Vec → L12/13 Transformer → L15 BERT/LLM。

---

### 2. 【高频】【数学题】Word2Vec skip-gram + negative sampling

**Definition：**  
Skip-gram 的任务是：given target word，predict context words。Slide 说 Word2Vec 存两个矩阵：target embedding matrix $$W$$和 context embedding matrix $$C$$，embedding dimension $$d$$ 由经验选择。

**Intuition：**  
经常一起出现的词向量被拉近，不相关的词被推远。老师 transcript 用 “apricot closer to jam, further from matrix” 这种直觉说明。

**Input/output：**  
Input 是 target word 的 one-hot 或 token id；positive examples 来自 context window；negative examples 从词表中随机采样。Slide 说明 window $$L=2$$ 会给一个 target word 产生 4 个 positive examples，另采样 $$k$$ 个 negative examples。

**Formula：**

$$P(+|w,c)=\sigma(v_w \cdot u_c)$$

$$P(-|w,c)=1-P(+|w,c)$$

其中 $$v_w$$ 是 target word embedding，$$u_c$$ 是 context embedding，点积越大，说明两个词越可能共现。

**Strengths / limitations：**  
优点是 self-supervised，不需要人工标签；缺点是 static，不能表达上下文语义，也可能继承 corpus bias。

**关系：**  
连接 L8 sparse/dense embedding；连接 L12/13 Transformer，因为 Transformer 的 token embedding 也是 dense embedding，但后续 self-attention 会变成 contextual。

---

### 3. 【Architecture题】RNN encoder-decoder

**Definition：**  
Encoder-decoder model 把 source sequence $$X$$ 编码成 context vector $$c$$，再由 decoder 逐步生成 target sequence $$Y$$。Slide 说明最终 encoder state $$h_n$$ 是 context vector，decoder 每一步使用 $$c$$ 和前一个 target word。

**Architecture：**

```text
source tokens X
   ↓ embedding
Encoder RNN: h1, h2, ..., hn
   ↓
context vector c = hn
   ↓
Decoder RNN + previous target token
   ↓
softmax over vocabulary
   ↓
target token y_t
```

**Formula：**

$$h_t^{enc}=g(h_{t-1}^{enc},x_t)$$

$$c=h_n^{enc}$$

$$h_t^{dec}=g(c,h_{t-1}^{dec},y_{t-1})$$

$$y_t=softmax(c,h_{t-1}^{dec},y_{t-1})$$

$$L=\frac{1}{T}\sum_{i=1}^{T}L_i$$

**Why matters：**  
它是 MT、seq2seq、QA generation、summarization 的基础。

**Limitation：**  
单一 context vector $$c=h_n$$ 要压缩整个 source sequence，长句信息容易丢失；这正是 attention 出现的原因。

---

### 4. 【高频】【数学题】Attention

**Definition：**  
Attention 不是只用最后一个 encoder hidden state，而是对所有 encoder hidden states 加权求和，得到每个 decoder step 的 context vector。Slide 明确写出 context vector $$c$$ 是 encoder hidden states 的 weighted sum，attention weight $$\alpha$$ 由 attention mechanism 定义。

**Formula：**

$$score_{ij}=h_{i-1}^{dec}\cdot h_j^{enc}$$

$$\alpha_{ij}=softmax(score_{ij})$$

$$c_i=\sum_j \alpha_{ij}h_j^{enc}$$

**Intuition：**  
翻译当前词时，模型可以“看”source sentence 中最相关的位置，而不是把整句压到一个 vector。

**Strengths / limitations：**  
优点是改善 long-range dependency；缺点是计算量随 sequence length 增长，并且需要理解矩阵 shape。

**关系：**  
RNN attention → Transformer self-attention → BERT/LLM/RAG。

---

### 5. 【高频】【Architecture题】Transformer block

**Definition：**  
Transformer block 主要由 Multi-head Attention、Feed Forward Layer、Normalization Layer、Residual connection 组成。Slide explicitly lists these components。

**Architecture：**

```text
X = token embedding + positional embedding
        ↓
LayerNorm
        ↓
Multi-head Self-Attention
        ↓
Residual Add
        ↓
LayerNorm
        ↓
Feed Forward Network
        ↓
Residual Add
        ↓
H
```

**Slide shape：**

$$X \in \mathbb{R}^{N_{seq}\times d_{model}}$$

$$T \in \mathbb{R}^{N_{seq}\times d_{model}}$$

$$H \in \mathbb{R}^{N_{seq}\times d_{model}}$$

Slide 的 transformer block computation 中明确给出 `X`, `T`, `H` 都是 $$N_{seq}\times d_{model}$$。

**Why matters：**  
考试很可能给 architecture diagram，让你判断 input/output shape、task type、head type。

**Limitations：**  
Self-attention 需要 position embedding；decoder-only LM 必须使用 causal mask，否则训练时会偷看 future tokens。

---

### 6. 【老师强调】【易错】Causal mask

**Definition：**  
Causal mask 在 decoder-only LM 中阻止 token $$i$$ 看到 future tokens $$j>i$$。Slide 写到：mask tokens in $$X \ge i$$，masked dot product 设置为 $$-\infty$$。

**Formula：**

$$score_{ij}=\frac{Q_iK_j^T}{\sqrt{d_k}}$$

$$score_{ij}=-\infty \quad \text{if } j>i$$

$$Attention(Q,K,V)=softmax(score)V$$

**Intuition：**  
如果训练时看到答案，模型会学会作弊；inference 时没有 future token，就会崩。

Transcript 老师特别强调：masking 是避免 causal language model cheating；要逐元素理解它如何隐藏 future tokens。

---

### 7. 【高频】Positional embedding

**Definition：**  
Transformer self-attention 本身不编码顺序，因此需要 positional embedding。Slide 列出 absolute positional embedding、sine/cosine、relative positional embedding、RoPE。

**Input/output：**

$$onehot(tokens) \in \mathbb{R}^{N_{seq}\times |V|}$$

$$E_{token}\in \mathbb{R}^{|V|\times d_{model}}$$

$$E_{pos}\in \mathbb{R}^{N_{seq}\times d_{model}}$$

$$X=TokenEmbedding+PositionEmbedding$$

$$X\in \mathbb{R}^{N_{seq}\times d_{model}}$$

**老师提醒：**  
Absolute positional embedding 要会 worked example；sine/cosine、relative、RoPE 至少能一句话解释 strengths/weaknesses，不一定要完整数学推导。

---

### 8. 【高频】Language model head vs classifier head

**Definition：**  
LM head 把最后一层 hidden state 映射到 vocabulary distribution；classifier head 把 hidden state 映射到 task label space。

**LM head formula：**

$$H_n\in \mathbb{R}^{1\times d_{model}}$$

$$U\in \mathbb{R}^{d_{model}\times |V|}$$

$$Y=softmax(H_nU)$$

$$Y\in \mathbb{R}^{1\times |V|}$$

Slide 说明 LM head 用最后 transformer output 预测 next token；也提到 weight tying 可用 transposed embedding matrix。

**Classifier head formula：**

$$y=softmax(h_{CLS}^{L}W_C)$$

Book 说明 BERT 的 $$[CLS]$$ 最后一层输出可作为整个 sequence representation，并接 classifier head；例如 sentiment 三分类时 $$W_C$$ shape 是 $$d\times 3$$。

**老师强调：**  
考试给 architecture 时，可以通过 head 的输出维度判断任务：vocab size 是 LM；K classes 是 classification；$$N_{seq}\times |tags|$$ 是 sequence labelling；$$N_{seq}\times 2$$ 可能是 QA start/end span。

---

## 第四部分：公式、算法、architecture

### 1. Word2Vec pipeline

```text
raw corpus
  ↓
choose target word w
  ↓
context window L
  ↓
positive pairs: (w, real context c+)
  ↓
negative sampling: (w, random c-)
  ↓
binary classifier predicts P(+|w,c)
  ↓
update W and C
  ↓
use learned W or averaged W/C as embeddings
```

核心 loss 可写成：

$$L=-\log\sigma(v_w\cdot u_{c^+})-\sum_{i=1}^{k}\log\sigma(-v_w\cdot u_{c_i^-})$$

每个项含义：

- $$v_w$$：target word embedding；
    
- $$u_c$$：context word embedding；
    
- $$c^+$$：真实上下文词；
    
- $$c_i^-$$：negative sample；
    
- $$k$$：negative examples 数量；
    
- $$\sigma$$：sigmoid；
    
- loss 最小化后，共现词 dot product 增大，不共现词 dot product 减小。
    

---

### 2. Scaled dot-product self-attention

$$Q=XW_Q$$

$$K=XW_K$$

$$V=XW_V$$

$$Attention(Q,K,V)=softmax(\frac{QK^T}{\sqrt{d_k}})V$$

Shape：

- $$X\in\mathbb{R}^{N_{seq}\times d_{model}}$$
    
- $$W_Q,W_K\in\mathbb{R}^{d_{model}\times d_k}$$
    
- $$W_V\in\mathbb{R}^{d_{model}\times d_v}$$
    
- $$Q,K\in\mathbb{R}^{N_{seq}\times d_k}$$
    
- $$V\in\mathbb{R}^{N_{seq}\times d_v}$$
    
- $$QK^T\in\mathbb{R}^{N_{seq}\times N_{seq}}$$
    
- $$softmax(QK^T/\sqrt{d_k})V\in\mathbb{R}^{N_{seq}\times d_v}$$
    

---

### 3. Transformer block architecture

```text
Input tokens
  ↓
Token embedding:       (Nseq, dmodel)
Position embedding:    (Nseq, dmodel)
  ↓ add
X:                     (Nseq, dmodel)
  ↓
Multi-head Attention
  ↓
Add & Norm
  ↓
Feed Forward Network
  ↓
Add & Norm
  ↓
H:                     (Nseq, dmodel)
```

---

### 4. BERT QA architecture

Slide 中 BERT QA 是：question + passage 输入 BERT，接 QA classifier head，输出每个 token 是 answer start / answer end 的 logits。

```text
[CLS] question [SEP] paragraph [SEP]
        ↓
BERT encoder
        ↓
H: (Nseq, dmodel)
        ↓
QA head U: (dmodel, 2)
        ↓
Y: (Nseq, 2)
        ↓
start logits + end logits
        ↓
softmax over token positions
        ↓
answer span
```

Loss：

$$L=\frac{L_{start}+L_{end}}{2}$$

其中：

- $$L_{start}$$：正确 answer start token 的 cross-entropy loss；
    
- $$L_{end}$$：正确 answer end token 的 cross-entropy loss；
    
- slide 还说明可以给 start loss 更高权重，因为 start 更关键。
    

---

## 第五部分：worked examples

### Worked example 1：【数学题】Self-attention tensor shape

假设：

- $$N_{seq}=4$$
    
- $$d_{model}=8$$
    
- $$num\_heads=2$$
    
- 每个 head：$$d_k=d_v=4$$
    

Input：

$$X\in\mathbb{R}^{4\times 8}$$

For one head：

$$W_Q\in\mathbb{R}^{8\times 4}$$

$$Q=XW_Q\in\mathbb{R}^{4\times 4}$$

同理：

$$K\in\mathbb{R}^{4\times 4}$$

$$V\in\mathbb{R}^{4\times 4}$$

Attention scores：

$$QK^T\in\mathbb{R}^{4\times 4}$$

因为：

$$(4\times 4)(4\times 4)^T=(4\times 4)(4\times 4)=4\times 4$$

Softmax 后仍然是：

$$A\in\mathbb{R}^{4\times 4}$$

Weighted sum：

$$AV\in\mathbb{R}^{4\times 4}$$

两个 heads concatenate：

$$Concat(head_1,head_2)\in\mathbb{R}^{4\times 8}$$

再经过 output projection 回到：

$$H\in\mathbb{R}^{4\times 8}$$

**考试写法：**  
The attention matrix has shape $$N_{seq}\times N_{seq}$$ because every token attends to every token. The output returns to $$N_{seq}\times d_{model}$$ after concatenating heads and applying the output projection.

---

### Worked example 2：【Architecture题】BERT QA shape

Question:  
Given $$N_{seq}=128$$, $$d_{model}=768$$, BERT QA head $$U\in\mathbb{R}^{768\times 2}$$. What is output shape?

Input after BERT：

$$H\in\mathbb{R}^{128\times 768}$$

QA head：

$$Y=HU$$

Shape：

$$(128\times 768)(768\times 2)=128\times 2$$

So：

$$Y\in\mathbb{R}^{128\times 2}$$

Interpretation：

- column 1：start logits；
    
- column 2：end logits；
    
- softmax over positions gives probability distribution over possible start/end token positions.
    

---

## 第六部分：Lab / code 考法

### 是否对应 lab？

- **Regex lab：本 lecture 不直接对应。**
    
- **CRF lab：本 lecture 不直接对应，但 sequence labelling 是 Part A/Part B 可能考点。**
    
- **BERT lab：强相关。老师明确说 BERT lab 有 attention tensor shapes，要跑 tensor shapes、理解 maths。**
    
- **LLaMA lab：相关。LLM lecture slide 提到 directed reading / BERT and LLaMA labs。**
    
- 具体 lab notebook 当前资料中未找到。
    

### 1. 代码阅读题

**Question：**

```python
outputs = bert(input_ids, attention_mask=mask)
H = outputs.last_hidden_state
cls = H[:, 0, :]
logits = classifier(cls)
probs = torch.softmax(logits, dim=-1)
```

问：这是哪类 NLP task？

**Answer：**  
Sequence classification，例如 sentiment classification / relation classification。因为取 $$[CLS]$$ hidden state，再接 classifier head，输出 label distribution。

**思路：**  
看 head：如果 output 是 $$batch\times K$$，通常是 classification。

---

### 2. Bug identification 题

**Question：**

```python
probs = torch.softmax(logits, dim=0)
```

如果 `logits.shape = (batch_size, num_classes)`，这里有什么问题？

**Answer：**  
应该用 `dim=-1` 或 `dim=1`。`dim=0` 会在 batch 维度归一化，把不同样本混在一起。

**思路：**  
Softmax 应该 over label space，不是 over batch。

---

### 3. Output shape 判断题

**Question：**  
BERT output $$H=(32,128,768)$$，QA head $$U=(768,2)$$，输出 shape？

**Answer：**

$$(32,128,768)(768,2)=(32,128,2)$$

含义：每个样本、每个 token 都有 start/end 两个 logits。

---

### 4. 参数 / tensor shape 计算题

**Question：**  
LM head 使用 $$U\in\mathbb{R}^{768\times 30000}$$，不考虑 bias，有多少参数？

**Answer：**

$$768\times 30000=23,040,000$$

如果使用 weight tying，可以用 embedding matrix transpose，减少单独学习 LM head 参数。Slide 明确提到 weight tying。

---

### 5. Training pipeline 排序题

**Question：**  
把 BERT QA fine-tuning pipeline 排序：

A. Compute start/end cross entropy  
B. Tokenize `[CLS] question [SEP] paragraph [SEP]`  
C. Pass through BERT encoder  
D. Apply QA head  
E. Backpropagate and update head / selected layers

**Answer：**  
B → C → D → A → E

**思路：**  
输入先编码，再得到 hidden states，再用 task-specific head，再算 loss。

---

## 第七部分：Reference book / wider reading 整合

### 1. Book 补充了什么？

**Fine-tuning 分类任务：**  
Book 比 slide 更明确地解释了 BERT 的 $$[CLS]$$ token 如何作为 sequence representation，以及 classifier head 如何把 $$h_{CLS}^L$$ 映射到 label scores。它还补充说 fine-tuning 可以只训练 head，也可以对 pretrained LM 参数做少量更新。

可写进 exam 的 stronger point：

> In practice, a pretrained BERT encoder can be used either as a frozen feature extractor with a newly trained classifier head, or fine-tuned partially/end-to-end depending on the size of labelled data and compute budget.

**Temperature sampling：**  
Book 解释了 temperature scaling 为什么会让 distribution 更 greedy 或更 flat：低 temperature 会放大 logits，使 high-probability token 更接近 1。

可写 stronger point：

> Temperature modifies the entropy of the output distribution without changing the model parameters; low temperature approximates greedy decoding, while high temperature increases diversity.

**BERT training：**  
Book 补充 NSP head 和 MLM loss 的训练机制；BERT 用 $$[CLS]$$ 做 NSP，$$[SEP]$$ 分隔句子，和 lecture 中 QA / classifier head 的用法直接相关。

### 2. Wider reading / papers 可支持什么？

- **Vaswani 2017**：支持 Transformer / self-attention architecture answer。Slide 引用 Attention Is All You Need。
    
- **Devlin 2019 BERT**：支持 BERT masked LM、NSP、QA fine-tuning answer。
    
- **Lewis 2021 RAG**：支持 RAG retrieve-then-read methodology。
    
- **Joshi 2020 SpanBERT**：支持 relation extraction 中用 SpanBERT/RoBERTa 而不是 vanilla BERT 的 stronger point。
    
- **Raschka 2025 LLM architecture blog**：支持 pre-norm/RMSNorm/RoPE/GQA 等 architecture design choice，老师也说这些 strengths/weaknesses 要有 handle。
    

---

## 第八部分：可能考试题预测

### Q1. [5 marks recall]

Explain the difference between static and contextual word embeddings. Give one example model for each.

**中文答题思路：**  
定义 static/contextual；Word2Vec/GloVe vs BERT；说明 dense 不等于 contextual；举 polysemy 例子。

---

### Q2. [10 marks worked example]

Given a one-hot word vector and an embedding matrix, show how the embedding vector is obtained. Then explain how Word2Vec skip-gram learns embeddings using positive and negative examples.

**中文答题思路：**  
写 one-hot shape；矩阵乘法；context window；positive pairs；negative sampling；dot product + sigmoid；更新 W/C。

---

### Q3. [10 marks worked example]

Given $$X\in\mathbb{R}^{N_{seq}\times d_{model}}$$, derive the shapes of Q, K, V, attention matrix, and output in scaled dot-product attention.

**中文答题思路：**  
写 $$Q=XW_Q$$ 等；注意 $$QK^T$$ 是 $$N_{seq}\times N_{seq}$$；最后乘 V；multi-head concat 回 $$d_{model}$$。

---

### Q4. [10 marks wider reading]

Compare causal decoder-only models and bidirectional encoder-only models. Explain which tasks each is suitable for.

**中文答题思路：**  
Causal = mask future, left-to-right generation，适合 generation/QA/summarization；BERT = no causal mask, whole sequence context，适合 NER/POS/classification/QA span extraction。Slide 明确对比 causal decoder-only 和 bidirectional encoder-only。

---

### Q5. [10 marks worked example]

A BERT QA model has hidden states $$H\in\mathbb{R}^{N_{seq}\times d_{model}}$$ and QA head $$U\in\mathbb{R}^{d_{model}\times 2}$$. Explain the output shape and loss function.

**中文答题思路：**  
输出 $$N_{seq}\times 2$$；两个 logits：start/end；softmax over positions；loss 是 start/end cross entropy 平均。

---

### Q6. [10 marks wider reading]

Explain how RAG improves open-domain QA compared with a pure language model.

**中文答题思路：**  
Retriever 从 vector index / web / corpus 找 context；reader/generator 用 question + context 生成答案；优点是 grounding，减少 hallucination；缺点是 retrieval quality bottleneck、index cost。

---

### 最可能考的一题：完整英文 exam-style answer

**Question:**  
Explain how a BERT model can be adapted for a downstream NLP task such as question answering or relation extraction. Your answer should refer to input format, hidden states, classifier head, tensor shapes, softmax output and loss.

**English exam-style answer:**

A pretrained BERT model is an encoder-only bidirectional Transformer. It can be adapted to a downstream task by adding a task-specific classifier head on top of the pretrained encoder. The input is first tokenized and formatted using BERT special tokens. For question answering, the input is usually `[CLS] question [SEP] passage [SEP]`. BERT processes the full sequence bidirectionally and outputs contextual hidden states for every token.

If the sequence length is $$N_{seq}$$ and the model dimension is $$d_{model}$$, the final hidden states have shape $$H\in\mathbb{R}^{N_{seq}\times d_{model}}$$. For extractive QA, a classifier head with weights $$U\in\mathbb{R}^{d_{model}\times 2}$$ is applied to each token:

$$Y=HU$$

so the output has shape $$Y\in\mathbb{R}^{N_{seq}\times 2}$$. The two output dimensions represent the logits for whether each token is the start or the end of the answer span. A softmax is applied over token positions to obtain a probability distribution for the start position and another distribution for the end position. The model is trained using cross-entropy loss for both the gold start token and the gold end token:

$$L=\frac{L_{start}+L_{end}}{2}$$

For relation extraction, a similar idea can be used, but the classifier head often takes the final hidden state of the `[CLS]` token as a sentence-level representation. The output dimension is the relation label space, for example $$K$$ possible relations plus a no-relation class. Named entity markers or replacements such as `SUBJ` and `OBJ` can be added to signal which entity pair is being classified. In low-data settings, the BERT encoder may be frozen and only the classifier head trained; alternatively, some or all layers can be fine-tuned. The key idea is that pretrained contextual representations are reused, while the new head maps them to the label space required by the downstream task.

---

## 第九部分：考前速记版

### 5–10 个 must-know points

1. 【高频】Dense vs sparse 不等于 contextual vs static；Word2Vec 是 dense + static，BERT 是 dense + contextual。
    
2. 【高频】Word2Vec skip-gram：target word → predict context words；positive from context window，negative by sampling。
    
3. 【数学题】Attention 核心公式：$$Attention(Q,K,V)=softmax(\frac{QK^T}{\sqrt{d_k}})V$$。
    
4. 【易错】$$QK^T$$ shape 是 $$N_{seq}\times N_{seq}$$，不是 $$d_{model}\times d_{model}$$。
    
5. 【老师强调】Causal mask 防止 decoder-only LM 偷看 future tokens。
    
6. 【Architecture题】Transformer block：MHA → Add/Norm → FFN → Add/Norm。
    
7. 【高频】Position embedding 是因为 self-attention 本身不含顺序信息。
    
8. 【Architecture题】LM head 输出 $$|V|$$；classifier head 输出 $$K$$；QA head 输出 $$N_{seq}\times 2$$。
    
9. 【Lab重点】BERT lab 要会 tensor shapes，尤其 attention 和 classifier/QA head。
    
10. 【应用题】RAG = retrieve context + read/generate answer。
    

### 3–5 个 high-risk mistakes

1. 【易错】把 dense embedding 说成一定 contextual。
    
2. 【易错】把 BERT 当成 decoder-only model；BERT 是 encoder-only。老师最后特别提醒：如果要 decoder，用 LLaMA/GPT；BERT 不要混成 decoder。
    
3. 【易错】忘记 causal mask，导致训练时模型偷看答案。
    
4. 【易错】softmax 维度写错：classification 是 over labels，不是 over batch。
    
5. 【易错】看到 architecture 不看 head，导致判断错任务类型。
    

### 3–5 个 likely exam questions

1. Static vs contextual embeddings + examples.
    
2. Word2Vec skip-gram worked example.
    
3. Transformer self-attention tensor shape calculation.
    
4. Given BERT QA/classifier architecture，判断 output shape、softmax、loss。
    
5. Compare causal decoder-only vs bidirectional encoder-only models.
    

### 本 lecture 最应该背的 formula / architecture / algorithm

最应该背：

$$Q=XW_Q,\quad K=XW_K,\quad V=XW_V$$

$$Attention(Q,K,V)=softmax(\frac{QK^T}{\sqrt{d_k}})V$$

$$Y=softmax(H_nU),\quad U\in\mathbb{R}^{d_{model}\times |V|}$$

$$Y_{QA}=HU,\quad H\in\mathbb{R}^{N_{seq}\times d_{model}},\quad U\in\mathbb{R}^{d_{model}\times 2},\quad Y\in\mathbb{R}^{N_{seq}\times 2}$$

### Transcript 中最值得记住的老师提醒

- 要会 **worked example**，尤其 one-hot → embedding matrix → embedding vector。
    
- Word2Vec 要能讲 positive/negative examples 和 context window。
    
- Encoder-decoder 要能讲 final hidden state 作为 context vector，以及 attention 如何改进它。
    
- Transformer 要会 Q/K/V、tensor shapes、masking、position embeddings、LM head。
    
- BERT lab 的 tensor shapes 很重要；考试给 architecture 时，经常可以通过 head 的 shape 判断任务。