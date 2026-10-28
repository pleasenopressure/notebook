# Lecture 12–13 Transformer and Sequence Processing — Exam-focused Revision Note

## 第一部分：资料核对

本次使用资料如下：

|资料类型|使用情况|
|---|---|
|课程资料归类|已使用。课程资料归类中该部分对应 **Lecture 12–13: Transformer**，PDF 建议名为 `L12_L13_Transformer.pdf`，Directed Reading 为 **Jurafsky & Martin Chapter 8; Vaswani et al. 2017 Attention Is All You Need; self-attention blog**。|
|Slide/PDF|已使用：`12-13-NLP-transformer.pdf`。Slide 明确覆盖 Sequence Processing with Transformers, Self-attention, Transformer Block, Positional Embedding, Language Model Head。|
|Transcript|已使用：`12transformer language processing.txt` 和 `13transformer language processing(2).txt`。两份 transcript 对应 transformer double lecture：前半部分 self-attention，后半部分 transformer block / position embedding / LM head。|
|Reference book / directed reading|已使用：Jurafsky & Martin, _Speech and Language Processing_, Chapter 8 Transformers；补充 Vaswani et al. 2017。书中 Chapter 8 明确包含 attention、Transformer blocks、parallelized matrix computation、token/position embeddings、LM head、training、scale 等内容。|
|Exam guidance|已使用：考试为 2 小时，Section A 必答 2 题，其中一题 classic NLP、一题 neural NLP；Section B 选 1/3；每题结构为 5 marks recall、10 marks worked example/lab、10 marks directed & wider reading。|
|Revision lecture|已使用：Neural NLP Recap 明确把 **Self-attention → Transformer lectures #12 #13** 放入 Neural NLP architecture recap。|

**本 lecture 分类：Neural NLP。**  
更准确地说：它是 **Neural NLP 的核心 architecture lecture**，同时是后面 **LLM and Modern NLP** 的基础。Revision lecture 中 Transformer 被放在 Neural NLP Recap 的 architecture 部分，而 LLMs 是在 Transformer 之后继续展开。

**资料冲突说明：**

- 没有发现 transcript 与 slide 在核心概念上冲突。
    
- 有一个小心点：slide 中 Feed Forward Layer 示例写作 `dff = 2028`，但原始 Transformer / 常见教材通常写 `d_ff = 2048`。考试中不要死记这个具体数字，建议写 general form：`d_ff usually larger than d_model`，如果必须举原始 Transformer 例子，可写 `d_model = 512, d_ff ≈ 2048`。课程 slide 的重点不是 2028/2048，而是 **FFN 把维度升高再投回 d_model**。
    

---

## 第二部分：考试定位

### 可能出现位置

**最可能：Section A Neural NLP。**  
也可能作为 Section B 的 application / mixed topic 与 LLM、BERT、QA、RAG、IE 结合。

原因：

1. **Revision lecture 出现**：Neural NLP Recap 明确列出 “Self-attention → Transformer lectures #12 #13”。
    
2. **Exam guidance 结构支持**：Section A 一定有一道 neural NLP topic；每题还会包含 recall、worked example/lab、directed reading/wider reading。
    
3. **Slide 重点展开**：整份 slide 用两部分讲 self-attention、Transformer block、positional embedding、language model head。
    
4. **Transcript 老师口头强调**：老师说 transformer block “quite important for the module”，理解 Transformer 后面 LLM 会更容易，并明确说 “could be asked in the exam”，还提醒做 directed reading 和 BERT lab。
    
5. **Lab 对应**：对应 **BERT Lab**，老师说会运行一个简单 BERT model，12 layers / 12 transformer blocks，并打印 layer shapes 与 slides 对照。
    
6. **Reference book 要求阅读**：课程资料归类对应 Jurafsky & Martin Chapter 8 和 Vaswani 2017。
    

### 考试优先级

**A 级：必须掌握，最可能考。**

尤其是：

- 【高频】self-attention / multi-head attention 的 definition、formula、shape。
    
- 【Architecture题】Transformer block：Multi-head Attention + Feed Forward + LayerNorm + residual connection。
    
- 【数学题】attention score、softmax、mask、tensor shape。
    
- 【Lab重点】BERT layer output shape / classifier head / LM head。
    
- 【老师强调】Transformer 是现代 NLP 和 LLM 的核心，可能被要求画 architecture diagram 或做 shape worked example。
    

---

## 第三部分：核心知识点

### 1. Transformer 为什么替代 MLP/RNN？

**Definition**  
Transformer 是一种基于 self-attention 的 neural sequence model。它不依赖 RNN 的 recurrence，而是让每个 token 通过 attention 直接访问上下文中的其他 token。

**Intuition**  
MLP 需要 fixed-length input，通常用 sliding window；问题是 window 外的信息会丢失。RNN 有 hidden state memory，但长序列中早期信息会 fade。Transformer 让模型在每一层都可以直接对整个 sequence 计算 attention，所以更适合 long-range dependency。Slide 明确对比了 MLP sliding window、RNN memory fading 和 Transformer attention full sequence available。

**Input / Output**

- Input: token sequence，例如 `So long and thanks for all`
    
- Token IDs / one-hot tokens → token embeddings + position embeddings
    
- Output: contextual hidden representations；对于 decoder-only LM，再经过 LM head 输出 next-token probability distribution。
    

**Why it matters in NLP**

- 支撑 BERT、GPT、LLaMA、T5 等现代 NLP 模型。
    
- 解决 word meaning depends on context 的问题。
    
- 后续 QA、IE、RAG、LLM 都建立在 Transformer 表示能力上。
    

**Strengths**

- 全局上下文访问。
    
- 可并行矩阵计算，比 RNN 更适合 GPU。
    
- 产生 contextual embeddings。
    
- 可堆叠成 very deep models。
    

**Limitations**

- Vanilla self-attention 有 sequence length quadratic cost，因为 attention matrix 是 $$N_{seq} \times N_{seq}$$。
    
- 需要 positional encoding，否则模型不知道 token 顺序。
    
- Decoder-only 训练必须 mask future tokens，否则模型会 cheat。
    

**和其他 lecture 的关系**

- 对比 Lecture 9 Word2Vec：Word2Vec 是 static embedding，Transformer 是 contextual embedding。
    
- 对比 Lecture 10 RNN：Transformer 不靠 recurrent memory，而靠 self-attention。
    
- 连接 Lecture 11 Encoder-Decoder：Transformer 仍可用于 encoder-decoder，但 attention 机制更强。
    
- 连接 Lecture 15 LLM：LLM 通常是 large Transformer-based model。
    
- 连接 Lecture 20 QA / RAG：BERT / DPR / ColBERT 都依赖 Transformer encoder。
    

---

### 2. Self-attention

**Definition**  
Self-attention 是一种在同一个 sequence 内部计算 token-token relevance 的机制。每个 token 根据 attention weights 从上下文 token 中聚合信息，形成新的 contextual representation。Slide 写明 self-attention maps input sequence `x1...xn` to output sequence `a1...an` with same length。

**Intuition**  
对于当前 token `it`，模型可以学会关注前面的 `chicken` 或 `road`。不是人为指定规则，而是通过训练学出哪些 token 对当前 prediction 有帮助。

**Input / Output**

- Input: $$X \in \mathbb{R}^{N_{seq} \times d_{model}}$$
    
- Output: $$A \in \mathbb{R}^{N_{seq} \times d_{model}}$$
    
- 每个 token 的输出 shape 与输入相同，因此可以继续 stack Transformer blocks。
    

**Why it matters**

它是 contextual embedding 的核心：同一个 word 在不同句子中会得到不同 representation。Transcript 明确说 Transformer embedding 会被上下文影响，而 Word2Vec 不会。

**Strengths**

- 捕捉 long-distance dependencies。
    
- 每个 token 可以选择性关注其他 token。
    
- 可矩阵化并行计算。
    

**Limitations**

- Attention matrix 尺寸随 sequence length 增长为 $$N_{seq}^2$$。
    
- Attention weights 不一定等价于 human-interpretable explanation。
    
- 需要 mask 来防止 causal LM 偷看未来。
    

---

### 3. Query / Key / Value

**Definition**

在 single-head self-attention 中，每个 token vector 会通过不同 trainable matrices 投影成：

- Query：当前 token 想找什么信息。
    
- Key：每个 token 提供的匹配索引。
    
- Value：真正被加权求和传递的信息。
    

公式：

$$q_i = x_i W_Q$$

$$k_j = x_j W_K$$

$$v_j = x_j W_V$$

其中：

- $$x_i$$：当前 token hidden vector，shape $$1 \times d_{model}$$
    
- $$W_Q$$：query projection matrix，shape $$d_{model} \times d_k$$
    
- $$W_K$$：key projection matrix，shape $$d_{model} \times d_k$$
    
- $$W_V$$：value projection matrix，shape $$d_{model} \times d_v$$
    
- $$q_i, k_j$$：shape $$1 \times d_k$$
    
- $$v_j$$：shape $$1 \times d_v$$
    

**Intuition**

Query 和 Key 用来算“当前 token 和另一个 token 有多相关”；Value 是“如果相关，就把这个 token 的哪些信息搬过来”。

**Why it matters**

Q/K/V 是 Transformer 相比普通 dot-product attention 更灵活的地方：模型不是直接比较 raw embeddings，而是学习不同 projection space。

---

### 4. Scaled dot-product attention

核心公式：

$$score(i,j)=\frac{q_i k_j^T}{\sqrt{d_k}}$$

$$\alpha_{ij}=softmax(score(i,j))$$

$$head_i=\sum_{j \le i}\alpha_{ij}v_j$$

如果是 full matrix version：


$$
Q = XW_Q,\quad K = XW_K,\quad V = XW_V
$$

$$
Head = softmax(mask(\frac{QK^T}{\sqrt{d_k}}))V
$$

**每个数学项含义**

- $$Q$$：all query vectors，shape $$N_{seq} \times d_k$$
    
- $$K$$：all key vectors，shape $$N_{seq} \times d_k$$
    
- $$V$$：all value vectors，shape $$N_{seq} \times d_v$$
    
- $$QK^T$$：所有 token pair 的 similarity scores，shape $$N_{seq} \times N_{seq}$$
    
- $$\sqrt{d_k}$$：scale factor，防止 dot product 过大导致 softmax saturation
    
- $$mask(\cdot)$$：causal decoder 中把 future token score 设为 $$-\infty$$
    
- $$softmax$$：把 scores 转换成 attention weights
    
- $$Head$$：attention 加权后的输出，shape $$N_{seq} \times d_v$$
    

Jurafsky & Martin Chapter 8 明确补充了 matrix parallelization：把 `N` 个 token embeddings pack 成 $$X \in \mathbb{R}^{N \times d}$$，一次性计算 `Q, K, V`，再通过 `QK^T` 得到 `N × N` token-token score matrix。

---

### 5. Causal mask / Masked self-attention

**Definition**  
Causal mask 用在 decoder-only language model 中，使 token `i` 不能 attend to future tokens `j > i`。

公式：

$$score(i,j)=-\infty \quad \text{if } j>i$$

然后：

$$softmax(-\infty)=0$$

**Intuition**  
训练 next-token prediction 时，如果模型看到未来 token，就相当于考试偷看答案。Transcript 里老师反复强调：future tokens 要 mask out，否则模型会 cheat。

**Input / Output**

- Input: full sequence still goes into the model.
    
- Mask: applied inside attention score matrix, not by deleting tokens at the input.
    
- Output: each position only uses previous/current information.
    

**易错点**

【易错】不是先把 future tokens 从 input 删除，而是在 attention score matrix 里把 future positions 设为 $$-\infty$$。Transcript 也强调不直接 mask token，而是依赖 mask matrix 防止 look ahead。

---

### 6. Multi-head self-attention

**Definition**  
Multi-head attention 是多个 self-attention heads 并行运行，每个 head 有自己的 $$W_Q, W_K, W_V$$，最后 concatenate，再用 $$W_O$$ 投影回 $$d_{model}$$。

公式：


$$
head_c = softmax(mask(\frac{Q_cK_c^T}{\sqrt{d_k}}))V_c
$$

$$
MultiHead(X)=concat(head_1,\ldots,head_A)W_O
$$

Shape：

- $$A$$：number of heads
    
- $$head_c \in \mathbb{R}^{1 \times d_v}$$
    
- $$concat(heads) \in \mathbb{R}^{1 \times A d_v}$$
    
- $$W_O \in \mathbb{R}^{A d_v \times d_{model}}$$
    
- output $$a_i \in \mathbb{R}^{1 \times d_{model}}$$
    

Slide 明确写出 multi-head self-attention：each head has own weights；attention output 是 concatenation of head outputs；hypothesis 是 heads learn different relationships。 Transcript 也解释了各 head 随机初始化、独立训练，因此会逐渐 specialise。

**Why it matters**

不同 head 可以学习不同关系，例如 syntactic relation、coreference、semantic relation、position relation。考试中可以写：multi-head gives the model multiple learned subspaces for token-token interaction。

---

### 7. Transformer Block

**Definition**  
Transformer block 是 Transformer 的重复基本单元，由：

1. Multi-head self-attention
    
2. Residual connection
    
3. LayerNorm
    
4. Feed Forward Network
    
5. Residual connection
    
6. LayerNorm
    

组成。课程 slide 明确列出 Transformer Block 包含 Multi-head Attention Layer、Feed Forward Layer、Normalization Layer。

**ASCII architecture diagram**

```text
Input X: (N_seq, d_model)
        |
        v
   LayerNorm
        |
        v
Multi-Head Self-Attention
        |
        +---------+
        |         |
        v         |
   Add residual <- X
        |
        v
   LayerNorm
        |
        v
 Feed Forward Network
        |
        +---------+
        |         |
        v         |
   Add residual <- previous residual stream
        |
        v
Output H: (N_seq, d_model)
```

**Transformer block equation**


$$
t_i^1 = LayerNorm(x_i)
$$

$$
t_i^2 = MultiHeadAttention(t_i^1, t_1^1,\ldots,t_N^1)
$$

$$
t_i^3 = t_i^2 + x_i
$$

$$
t_i^4 = LayerNorm(t_i^3)
$$

$$
t_i^5 = FFN(t_i^4)
$$

$$
h_i = t_i^5 + t_i^3
$$

Jurafsky & Martin 用 residual stream 解释 Transformer block，并强调 input/output dimensions match so blocks can be stacked。 Transcript 也强调 `X`, intermediate tensors, output `H` 都保持 $$N_{seq} \times d_{model}$$，这是为了让 blocks 可以 stack。

**Feed Forward Network**

公式：


$$
FFN(x)=f(xW_1+b_1)W_2+b_2
$$

Shape：

- $$W_1 \in \mathbb{R}^{d_{model}\times d_{ff}}$$
    
- $$W_2 \in \mathbb{R}^{d_{ff}\times d_{model}}$$
    
- $$d_{ff} > d_{model}$$
    

**Intuition**

- Attention layer：从 input context 中“搬运/混合”有用信息。
    
- Feed-forward layer：对每个 token position 独立做 nonlinear transformation，存储/表达 pretrained knowledge。Transcript 中老师说有证据表明 feed forward layer 编码大量 pretrained knowledge，而 self-attention 学会关注 input 中重要内容。
    

---

### 8. LayerNorm

**Definition**  
LayerNorm 对 hidden vector 的维度做 normalization，使训练更稳定。

公式：


$$
\mu = \frac{1}{d}\sum_{k=1}^{d}x_k
$$

$$
\sigma = \sqrt{\frac{1}{d}\sum_{k=1}^{d}(x_k-\mu)^2}
$$

$$
LayerNorm(x)=\gamma\frac{x-\mu}{\sigma}+\beta
$$


其中：

- $$x$$：hidden vector
    
- $$d$$：hidden dimension
    
- $$\mu$$：mean
    
- $$\sigma$$：standard deviation
    
- $$\gamma$$：learnable gain parameter
    
- $$\beta$$：learnable offset parameter
    

Slide 说明 LayerNorm 是为了把 hidden layer values normalize 到 gradient-based training 更容易工作的范围。

---

### 9. Positional Embedding

**Definition**  
Position embedding 是给 Transformer 输入加入 token order 信息的方法。因为 self-attention 本身不天然知道 sequence order，所以必须加入 positional information。

**Input / Output**

Slide shape：

- one-hot tokens: $$N_{seq} \times |V|$$
    
- token embedding matrix: $$E_{token} \in \mathbb{R}^{|V|\times d_{embed}}$$
    
- token embedding output: $$N_{seq} \times d_{embed}$$
    
- position index: $$N_{seq} \times 1$$
    
- position embedding: $$E_{pos} \in \mathbb{R}^{N_{seq}\times d_{embed}}$$
    
- final input: $$X \in \mathbb{R}^{N_{seq}\times d_{embed}}$$
    

Formula：


$$
X = TokenEmbedding(tokens) + PositionEmbedding(positions)
$$


Slide 明确说 Transformers need input embedding and positional embedding，并列出 Absolute Position Embedding、sine/cosine、Relative Position Embedding、RoPE。

**Intuition**

句子 “dog bites man” 和 “man bites dog” 的 token 集合相同，但顺序不同。没有 position embedding，模型很难区分顺序。

**Types**

- Absolute Position Embedding：直接学习 position 1,2,3,...
    
- Sinusoidal Position Embedding：用 sine/cosine，使数值范围稳定。
    
- Relative Position Embedding：编码相对距离，例如 target word 左边一个 / 右边一个。
    
- RoPE：现代 LLM 常用，尤其与 LLaMA 相关。
    

Transcript 中老师解释 position embedding 的 shape：word embedding 与 position embedding 都输出 $$N_{seq}\times d_{embed}$$，然后 arithmetic sum，shape 不变。

---

### 10. Language Model Head

**Definition**  
Language model head 是加在 Transformer 顶部的输出层，用最后一层 hidden state 预测下一个 token。
![[Pasted image 20260601143519.png]]![[Pasted image 20260601143608.png]]![[Pasted image 20260601143616.png]]![[Pasted image 20260601143752.png]]
Formula：


$$
logits = H_N U
$$

$$
P(w_{N+1}|w_{\le N}) = softmax(logits)
$$


Shape：

- $$X \in \mathbb{R}^{N_{seq}\times d_{model}}$$
    
- $$H \in \mathbb{R}^{N_{seq}\times d_{model}}$$
    
- $$H_N \in \mathbb{R}^{1\times d_{model}}$$
    
- $$U \in \mathbb{R}^{d_{model}\times |V|}$$
    
- $$Y \in \mathbb{R}^{1\times |V|}$$
    

Slide 明确写出 LM head：convert last Transformer layer output token N to predict token N+1；linear layer U generates logits；也可用 weight tying，用 transposed embedding matrix E as weights。 Transcript 也说明：取 last token hidden layer，乘以 `U` 得到 `1 × |V|` logits，再 softmax 成 word distribution。

**Training objective**

Decoder-only LM 的目标是 next-token prediction：


$$
P(w_{1:N})=\prod_{t=1}^{N}P(w_t|w_{<t})
$$

$$
\mathcal{L}=-\sum_{t=1}^{N}\log P(y_t|x_{\le t})
$$


通常使用 cross-entropy loss。

---

## 第四部分：Architecture 总图

### Decoder-only Transformer LM

```text
Input tokens
   |
   v
Token IDs / one-hot tokens
   |
   v
Token Embedding: (N_seq, d_embed)
   +
Position Embedding: (N_seq, d_embed)
   |
   v
X: (N_seq, d_model)
   |
   v
[Transformer Block] x L
   |
   |-- LayerNorm
   |-- Masked Multi-Head Self-Attention
   |-- Residual Add
   |-- LayerNorm
   |-- Feed Forward Network
   |-- Residual Add
   v
H: (N_seq, d_model)
   |
   v
Take last token H_N: (1, d_model)
   |
   v
LM Head U: (d_model, |V|)
   |
   v
Logits: (1, |V|)
   |
   v
Softmax
   |
   v
Next-token probability distribution
```

### Neural model required details

| Component           |         Shape without batch |                  Shape with batch in code |
| ------------------- | --------------------------: | ----------------------------------------: |
| token IDs           |                 $$N_{seq}$$ |                      $$B \times N_{seq}$$ |
| one-hot tokens      |         $$N_{seq}\times V$$ |                                           |
| embeddings          | $$N_{seq}\times d_{embed}$$ |       $$B\times N_{seq}\times d_{embed}$$ |
| Transformer input X | $$N_{seq}\times d_{model}$$ |       $$B\times N_{seq}\times d_{model}$$ |
| Q/K                 |       $$N_{seq}\times d_k$$ |     $$B\times A\times N_{seq}\times d_k$$ |
| V                   |       $$N_{seq}\times d_v$$ |     $$B\times A\times N_{seq}\times d_v$$ |
| attention scores    |   $$N_{seq}\times N_{seq}$$ | $$B\times A\times N_{seq}\times N_{seq}$$ |
| attention output    | $$N_{seq}\times d_{model}$$ |       $$B\times N_{seq}\times d_{model}$$ |
| LM head output      |               $$1\times V$$ |                                           |

---

## 第五部分：Worked example【数学题】【Shape题】

### Question

Given a decoder-only Transformer with:

- $$N_{seq}=3$$
    
- $$d_{model}=4$$
    
- $$d_k=2$$
    
- $$d_v=2$$
    
- number of heads $$A=2$$
    
- vocabulary size $$|V|=5$$
    

For token 3, suppose:


$$
q_3=[1,2]
$$

$$
k_1=[1,0],\quad k_2=[0,1],\quad k_3=[1,1]
$$

$$
v_1=[1,0],\quad v_2=[0,2],\quad v_3=[3,1]
$$


Compute the attention output for token 3 in one head.

### Step 1: scaled dot-product scores

Formula:


$$
score(3,j)=\frac{q_3k_j^T}{\sqrt{d_k}}
$$


Since $$d_k=2$$:


$$
\sqrt{d_k}=\sqrt{2}\approx1.414
$$

Scores:


$$
score(3,1)=\frac{[1,2]\cdot[1,0]}{1.414}=\frac{1}{1.414}=0.707
$$

$$
score(3,2)=\frac{[1,2]\cdot[0,1]}{1.414}=\frac{2}{1.414}=1.414
$$

$$
score(3,3)=\frac{[1,2]\cdot[1,1]}{1.414}=\frac{3}{1.414}=2.121
$$

### Step 2: softmax attention weights

 
$$
softmax(s_j)=\frac{e^{s_j}}{\sum_k e^{s_k}}
$$


Approximate exponentials:

 
$$
e^{0.707}=2.028,\quad e^{1.414}=4.113,\quad e^{2.121}=8.340
$$

$$
sum=2.028+4.113+8.340=14.481
$$

Attention weights:

 
$$
\alpha_{31}=2.028/14.481=0.140
$$

$$
\alpha_{32}=4.113/14.481=0.284
$$

$$
\alpha_{33}=8.340/14.481=0.576
$$

### Step 3: weighted sum of values

Formula:

 
$$
head_3=\alpha_{31}v_1+\alpha_{32}v_2+\alpha_{33}v_3
$$

Calculation:

 
$$
head_3=0.140[1,0]+0.284[0,2]+0.576[3,1]
$$

$$
head_3=[0.140,0]+[0,0.568]+[1.728,0.576]
$$

$$
head_3=[1.868,1.144]
$$

So one head output for token 3 is:

 
$$
head_3=[1.868,1.144]
$$

### Step 4: multi-head shape

Each head output:

 
$$
head_i \in \mathbb{R}^{1\times d_v}=1\times2
$$

Two heads concatenated:

 
$$
concat(heads)\in\mathbb{R}^{1\times A d_v}=1\times4
$$

Since $$d_{model}=4$$, choose:

 
$$
W_O\in\mathbb{R}^{4\times4}
$$

Final attention output:

 
$$
a_i\in\mathbb{R}^{1\times4}
$$


This can be added back to input $$x_i\in\mathbb{R}^{1\times4}$$ through residual connection.

### Exam answer point

你要写清楚：**attention 不是只输出一个 scalar；softmax weights 是 scalar distribution，但最终输出是 weighted sum of value vectors。**

---

## 第六部分：Lab / code 考法【Lab重点】

### 是否对应 lab？

**对应：BERT Lab。**  
不是 Regex lab，不是 CRF lab，不是 LLaMA lab。Transformer lecture 直接关联 **BERT Lab**：老师在 transcript 中说 BERT lab 会运行 simple BERT model，有 12 layers / 12 transformer blocks，并打印 layer shapes，要求与 slides 对照。

---

### 1. 代码阅读题

**Question**  
Given:

```python
outputs = bert_model(input_ids=input_ids, attention_mask=attention_mask)
last_hidden = outputs.last_hidden_state
```

If `input_ids.shape = [8, 32]` and BERT hidden size is 768, what is `last_hidden.shape`?

**Answer**  
`[8, 32, 768]`

**思路**  
BERT 输出每个 token 的 contextual embedding。batch size = 8，sequence length = 32，hidden size = 768。

---

### 2. Bug identification 题

**Question**  
A student writes:

```python
logits = classifier(outputs.last_hidden_state)
loss = loss_fn(logits, labels)
```

For sentence classification, why might this be wrong?

**Answer**  
`outputs.last_hidden_state` has shape `[B, N_seq, hidden]`，直接送入 classifier 会给每个 token 一个 label；sentence classification 通常需要 `[CLS]` token representation 或 pooled output，shape `[B, hidden]`。

Correct version:

```python
cls_vec = outputs.last_hidden_state[:, 0, :]
logits = classifier(cls_vec)
```

**思路**  
区分 sequence labelling 与 sentence classification。

---

### 3. Output shape 判断题

**Question**  
BERT encoder output is `[B, N_seq, d_model] = [4, 20, 768]`. A classifier head is `Linear(768, 3)` applied to the `[CLS]` vector. What is logits shape?

**Answer**  
`[4, 3]`

**思路**  
取 `[CLS]` 后 shape `[4, 768]`，Linear maps 768 → 3。

---

### 4. 参数 / tensor shape 计算题

**Question**  
In one attention head, `d_model=768`, `d_k=64`, `d_v=64`. What are the shapes of `W_Q`, `W_K`, `W_V`?

**Answer**

- `W_Q`: `[768, 64]`
    
- `W_K`: `[768, 64]`
    
- `W_V`: `[768, 64]`
    

如果有 12 heads，concat 后 shape:

 
$$
12 \times 64 = 768
$$


所以 `W_O` shape:

 
$$
768 \times 768
$$

---

### 5. Training pipeline 排序题

**Question**  
Put the following steps in order for fine-tuning BERT for classification:

A. Add classifier head  
B. Tokenize text and create attention mask  
C. Load pretrained BERT  
D. Compute cross-entropy loss  
E. Forward pass through BERT  
F. Backpropagate and update parameters  
G. Use `[CLS]` representation

**Answer**  
B → C → A → E → G → D → F

更自然写法：

```text
Tokenize input
→ Load pretrained BERT
→ Pass input_ids + attention_mask into BERT
→ Extract [CLS] hidden state
→ Feed into classifier head
→ Compute cross-entropy loss
→ Backpropagate / update
```

---

## 第七部分：Reference book / wider reading 整合

### Slide 没讲那么细，但 book 补充的内容

1. **Residual stream 视角**  
    Jurafsky & Martin 把 Transformer block 看成 residual stream：attention head 可以把 Token A 的 residual stream 信息移动到 Token B 的 stream 中。这个点可以让你的 10 marks answer 更有深度。
    
2. **Matrix parallelization**  
    Book 解释了如何把所有 token pack 到 $$X \in \mathbb{R}^{N\times d}$$，一次性计算 `Q,K,V`，并用 `QK^T` 得到 $$N\times N$$ scores。这比只讲单 token attention 更适合写 worked example。
    
3. **Stacking requirement**  
    Book 明确说明 Transformer block input/output dimensions must match，才能 stack deeper networks。这个点与 transcript 的 shape 强调一致。
    
4. **Efficiency / scale**  
    Book Chapter 8 还补充 large model scale、KV cache、PEFT / LoRA 等内容；这些可以作为 wider reading，但如果题目只问 Transformer basics，不要跑题。
    

### 10 marks wider reading 可写点

- Vaswani et al. 2017 的核心贡献：Attention is All You Need，用 self-attention 替代 recurrence/convolution。
    
- Original Transformer 使用 encoder-decoder structure；现代 GPT/LLaMA 多用 decoder-only causal Transformer。
    
- Scaled dot-product attention 的 scale factor $$\sqrt{d_k}$$ 防止 dot product 过大。
    
- Multi-head attention 允许模型在多个 learned subspaces 中捕捉不同关系。
    
- Causal mask 是 decoder-only LM 训练 next-token prediction 的必要条件。
    
- Transformer 的主要 computational bottleneck 是 attention matrix $$N_{seq}\times N_{seq}$$。
    

---

## 第八部分：可能考试题预测

### Q1. [5 marks recall] Explain why Transformers are better suited than MLPs and RNNs for long sequence processing.

中文答题思路：

- MLP fixed window，window 外上下文丢失。
    
- RNN sequential memory fades，long dependency 难。
    
- Transformer self-attention 让每个 token attend to context。
    
- 支持 contextual embeddings。
    
- 但注意 vanilla attention cost 高。
    

---

### Q2. [10 marks worked example] Given Q, K, V vectors, compute scaled dot-product attention for one token.

中文答题思路：

- 写 $$q_i=x_iW_Q$$, $$k_j=x_jW_K$$, $$v_j=x_jW_V$$
    
- 算 $$q_i k_j^T/\sqrt{d_k}$$
    
- 对 scores softmax 得 $$\alpha_{ij}$$
    
- weighted sum values 得 head output
    
- 如果是 decoder-only，future scores mask 为 $$-\infty$$
    

---

### Q3. [10 marks worked example] Draw and explain a Transformer block with tensor shapes.

中文答题思路：

- 画 LayerNorm → Multi-head Attention → residual add → LayerNorm → FFN → residual add。
    
- 强调 input/output 都是 $$N_{seq}\times d_{model}$$。
    
- 说明 why same shape: stack blocks。
    
- 说明 FFN uses $$d_{model}\rightarrow d_{ff}\rightarrow d_{model}$$。
    

---

### Q4. [10 marks wider reading] Explain positional embeddings and compare absolute, sinusoidal, relative, and RoPE.

中文答题思路：

- Self-attention 无 order awareness。
    
- Token embedding + position embedding 相加。
    
- Absolute position 简单可学习。
    
- Sin/cos 数值范围稳定，normalize sequence length。
    
- Relative position 适合 relation extraction / syntax。
    
- RoPE 是 modern LLM 常见方法。
    

---

### Q5. [10 marks lab/code] In a BERT lab, explain the output tensor shapes and how to attach a classifier head.

中文答题思路：

- BERT encoder output `[B, N_seq, hidden]`。
    
- Classification 用 `[CLS]` 或 pooled output `[B, hidden]`。
    
- Classifier head maps `[B, hidden] → [B, num_labels]`。
    
- Loss 用 cross entropy。
    
- 不要把整个 `[B,N,H]` 直接用于 sentence-level labels。
    

---

### Q6. [10 marks wider reading] Compare decoder-only causal Transformer and bidirectional BERT encoder.

中文答题思路：

- Decoder-only: causal mask, next-token prediction, generation。
    
- BERT encoder: bidirectional attention, masked language modeling / classification。
    
- Decoder-only good for text generation / chatbots。
    
- BERT good for interpretation tasks, NER/POS/classification。
    
- Mask difference 是关键。
    

---

## 最可能考的一题：完整英文 exam-style answer

**Question:**  
Explain the Transformer architecture for language modelling. Your answer should include self-attention, multi-head attention, positional embeddings, masking, the language model head, and relevant tensor shapes.

**Answer:**

A Transformer is a neural sequence model based on self-attention rather than recurrence. Unlike an MLP with a fixed sliding window, or an RNN whose hidden-state memory can fade over long sequences, a Transformer can allow each token representation to attend to other tokens in the input sequence. This makes it suitable for modelling long-range dependencies and for producing contextual word embeddings.

The input is first converted into token embeddings. If the sequence length is $$N_{seq}$$ and the vocabulary size is $$|V|$$, the one-hot token matrix has shape $$N_{seq}\times |V|$$. Multiplying this by an embedding matrix gives token embeddings of shape $$N_{seq}\times d_{embed}$$. Since self-attention alone has no notion of word order, a positional embedding of the same shape is added:

$$X = TokenEmbedding(tokens) + PositionEmbedding(positions)$$

so the Transformer input has shape $$N_{seq}\times d_{model}$$.

In self-attention, each token vector is projected into query, key, and value vectors:

$$Q=XW_Q,\quad K=XW_K,\quad V=XW_V$$

For a single attention head, the attention scores are computed using scaled dot-product attention:

$$Head=softmax(mask(\frac{QK^T}{\sqrt{d_k}}))V$$

Here $$QK^T$$ compares every query with every key, producing an attention score matrix of shape $$N_{seq}\times N_{seq}$$. The scale factor $$\sqrt{d_k}$$ prevents dot products from becoming too large. In a causal decoder-only language model, future positions are masked by setting their scores to $$-\infty$$ before softmax, so the model cannot look ahead and cheat during next-token prediction.

Transformers use multi-head attention, meaning several attention heads are computed in parallel. Each head has its own learned projection matrices and can learn different token relationships. The head outputs are concatenated and projected back to $$d_{model}$$:

$$MultiHead(X)=concat(head_1,\ldots,head_A)W_O$$

A Transformer block combines multi-head attention with residual connections, layer normalization, and a feed-forward network. The feed-forward network usually maps $$d_{model}$$ to a larger $$d_{ff}$$ dimension and then back to $$d_{model}$$. The input and output shapes of each block remain $$N_{seq}\times d_{model}$$, allowing many blocks to be stacked.

For language modelling, the final Transformer output $$H$$ also has shape $$N_{seq}\times d_{model}$$. To predict the next token, the model takes the hidden state for the final token, $$H_N$$, with shape $$1\times d_{model}$$, and passes it through a language model head:

$$logits=H_NU$$

where $$U$$ has shape $$d_{model}\times |V|$$. The logits have shape $$1\times |V|$$, and softmax converts them into a probability distribution over the vocabulary. The model is trained with next-token prediction using cross-entropy loss.

A key strength of the Transformer is that it produces contextual embeddings and can be efficiently parallelized using matrix multiplication. A limitation is that vanilla attention requires an $$N_{seq}\times N_{seq}$$ attention matrix, so memory and computation grow quadratically with sequence length.

---

## 第九部分：考前速记版

### 5–10 个 must-know points

1. 【高频】Transformer = self-attention based sequence model，不是 RNN。
    
2. 【高频】Word2Vec 是 static embedding；Transformer 是 contextual embedding。
    
3. 【数学题】Self-attention 核心公式：$$softmax(mask(QK^T/\sqrt{d_k}))V$$
    
4. 【Architecture题】Transformer block = LayerNorm + Multi-head Attention + residual + LayerNorm + FFN + residual。
    
5. 【Shape题】Transformer block input/output 都是 $$N_{seq}\times d_{model}$$，这样才能 stack。
    
6. 【易错】Causal mask 是在 attention scores 里把 future token 设为 $$-\infty$$，不是删除 input token。
    
7. 【高频】Multi-head attention：多个 heads，各自有自己的 $$W_Q,W_K,W_V$$，concat 后用 $$W_O$$ 投回 $$d_{model}$$。
    
8. 【高频】Position embedding 必须加，因为 self-attention 本身不知道顺序。
    
9. 【LM Head】最后 token hidden state $$H_N$$ 乘 $$U$$ 得 $$1\times |V|$$ logits，再 softmax。
    
10. 【Lab重点】BERT 输出常见 shape 是 `[batch, seq_len, hidden]`；classification 通常取 `[CLS]`。
    

### 3–5 个 high-risk mistakes

1. 【易错】把 attention weight 当成最终输出。正确：weight 是 scalar，最终输出是 value vectors 的 weighted sum。
    
2. 【易错】把 residual add 写成 concatenation。正确：residual 是 element-wise addition，shape 不变。
    
3. 【易错】忘记 $$\sqrt{d_k}$$ scaling。
    
4. 【易错】混淆 decoder-only causal Transformer 和 BERT bidirectional encoder。
    
5. 【易错】认为 Transformer 一次只处理一个 token。正确：full sequence 输入，只是 causal mask 控制可见范围。
    

### 3–5 个 likely exam questions

1. Explain self-attention and compute a worked example with Q/K/V.
    
2. Draw a Transformer block and explain tensor shapes.
    
3. Explain why positional embeddings are needed and compare different types.
    
4. Explain language model head and next-token prediction objective.
    
5. In the BERT lab, identify output shapes and attach a classifier head.
    

### 最应该背的 formula / architecture / algorithm

```text
$$
Q=XW_Q,\quad K=XW_K,\quad V=XW_V
$$

$$
Head=softmax(mask(\frac{QK^T}{\sqrt{d_k}}))V
$$

$$
MultiHead(X)=concat(head_1,\ldots,head_A)W_O
$$

$$
logits=H_NU
$$

$$
P(w_{N+1}|w_{\le N})=softmax(logits)
$$
```

### Transcript 中最值得记住的老师提醒

- 【老师强调】Transformer double lecture 分两部分：self-attention；然后 transformer block / position embedding / LM head。
    
- 【老师强调】Transformer 很重要，理解它会帮助理解后面 LLM；可能在 exam 中被问到。
    
- 【Lab重点】做 BERT lab：会运行 12-layer BERT model，并打印 layer shapes，要和 slides 对照。
    
- 【易错】模型接收 full sequence，但 causal LM 必须 mask future tokens，否则会 cheat。