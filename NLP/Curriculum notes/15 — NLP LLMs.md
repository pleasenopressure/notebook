# Lecture 15 — NLP LLMs：Exam-focused revision note

## 第一部分：资料核对

本次使用资料如下：

1. **课程资料归类**：已找到。Lecture 15 对应 **Large Language Models**，建议 PDF 为 `L15_Large_Language_Models.pdf`，Directed Reading 包括 Jurafsky & Martin Chapter 7 的 7.1、7.3、7.4、7.5，Chapter 9 的 9.1，Chapter 10 的 10.1，以及 LLaMA、LLaMA2、RoPE、RMSNorm、SwiGLU、Self-refinement、Self-consistency、Raschka LLM architecture blog 等。
    
2. **Slide/PDF**：已找到 `15-NLP-LLMs.pdf`。内容包括 Large Language Models、LLM Prompting、LLM Decoding、Masked Language Models、Generative Language Models、LLM Training。
    
3. **Transcript**：已找到 `15-NLP-LLMs.txt`。它覆盖了 LLM 定义、pretraining、prompting、decoding、BERT/LLaMA、RoPE/RMSNorm/SwiGLU、SFT/RLHF 等内容。
    
4. **Reference book / directed reading**：已找到 Jurafsky & Martin _Speech and Language Processing_ 3rd ed. draft。目录中包含 Chapter 7 Large Language Models、Chapter 8 Transformers、Chapter 9 Masked Language Models、Chapter 10 Post-training、Instruction Tuning、Alignment 等。
    
5. **缺失资料**：LLaMA、LLaMA2、RoPE、RMSNorm、SwiGLU、Self-refinement、Self-consistency、Raschka blog 的完整 paper/blog 原文，当前资料中未找到；但 slide 中给出了它们作为 directed reading 的引用或链接。
    
6. **冲突检查**：slide 和 transcript 没有实质冲突。Transcript 是自动转写，有一些术语识别错误，例如 “Gwen” 应理解为 **Qwen**，“Lomond labs” 应理解为 **LLaMA labs**，“human reinforcement learning” 应按 slide 的正式术语理解为 **RLHF / Preference Alignment**。术语以 slide 为准；老师口头强调以 transcript 为补充。
    

根据课程资料归类，Lecture 15 属于：**LLM and Modern NLP**。

---

## 第二部分：考试定位

### 更可能出现在哪一部分？

**最可能：Section B application or mixed topic。**  
Revision lecture 的 Part B 明确把 **Large Language Models** 放进 Part B，包括 Encoder BERT、Decoder LLaMA、Encoder-Decoder Transformer、LLM architecture variants、distillation、training、prompting、decoding。

**也可能：Section A Neural NLP。**  
Section A 固定有一个 Neural NLP topic，而 Revision lecture 的 Neural NLP 部分包含 language models、word2vec、RNN、Transformer、training、architectures、layer shapes。LLM 依赖 Transformer、self-attention、language model head，所以也可能以 architecture / shape / decoding 的形式混入 Section A Neural NLP。

### 考试优先级

**A级：必须掌握，最可能考**

- LLM 三类架构：**encoder-only masked LM / decoder-only generative LM / encoder-decoder text-to-text LM**
    
- BERT vs LLaMA：pretraining objective、attention mask、input/output、适合任务
    
- Prompting：direct / zero-shot, in-context / few-shot, CoT, self-consistency, self-refinement
    
- Decoding：random sampling, greedy, top-k, top-p, temperature
    
- Language model head：logits、softmax、next-token probability、tensor shape
    
- BERT MLM：15% corruption，80/10/10，[MASK]，loss only on corrupted tokens
    
- LLaMA-style generative model：BPE, decoder-only Transformer, causal attention, RMSNorm, RoPE, SwiGLU
    
- SFT / Instruction tuning 与 RLHF / preference alignment 的训练 pipeline
    

**B级：需要理解，可能考**

- RMSNorm 为什么比 LayerNorm 更适合大模型
    
- RoPE 的 relative position intuition
    
- SwiGLU 与 ReLU 的区别
    
- Fine-tuning / domain adaptation / instruction tuning 的区别
    
- Reward model 为什么要把 LM head 换成 classifier head
    
- KV cache / PEFT / LoRA 作为 wider reading 加分点
    

**C级：了解即可**

- 具体模型参数量、GPU 小时、数据集大小，例如 LLaMA2 3.3M GPU hours、CommonCrawl 156B tokens
    
- RLHF 的强化学习细节
    
- RoPE/RMSNorm/SwiGLU 的完整论文级数学推导
    

依据：考试 guidance 说明全卷 2 小时、5 题、Section A 两题都答、Section B 三选一，题目结构通常是 5 marks lecture recall、10 marks lecture + worked example + labwork、10 marks lecture + directed/wider reading。 Revision lecture 也强调同样的 5/10/10 结构，并提示 worked example、maths、architecture diagram 都可能有部分分。

---

# 第三部分：核心知识点

## 1. Large Language Model, LLM【高频】

**Definition**  
LLM 是通常基于 **Transformer** 的大规模语言模型，slide 给出的经验定义是通常超过 **1B parameters**。Parameters 指模型中可学习的 floating-point weights，例如 self-attention 中的 query/key/value weight matrices。

**Intuition**  
LLM 不是“懂语言的人”，而是一个通过大量文本学习 token co-occurrence patterns 的概率模型。老师在 transcript 里特别提醒：不要 anthropomorphise models；prompt 只是 text，不是在“教”模型，而是在激活预训练权重中相关的模式。

**Input / Output**

- Input：token sequence / prompt + input text
    
- Output：token probabilities 或 generated text
    
- 内部：token ids → embeddings → Transformer layers → LM head → logits → softmax distribution
    

**Why it matters in NLP**  
LLM 可以用 prompt 统一处理 sentiment analysis、translation、summarization、QA、IE 等任务。但 slide 也提醒：zero-shot generative LLMs can do many tasks, **but not always optimal choice**；对于结构化、高精度、低资源或可解释任务，classic / task-specific models 仍然重要。

**Strengths**

- 可 zero-shot / few-shot
    
- 可迁移到很多 NLP tasks
    
- 生成能力强
    
- 不需要为每个任务重新设计传统 pipeline
    

**Limitations**

- 计算成本高
    
- 训练数据和数据质量是瓶颈
    
- hallucination、bias、privacy、safety 问题
    
- 对高风险任务不一定最优
    
- prompt-sensitive，不同模型可能需要不同 prompt
    

**和其他 lecture 的关系**

- Lecture 2：BPE / tokenization
    
- Lecture 9：embedding
    
- Lecture 12–13：Transformer / self-attention / LM head
    
- Lecture 20：QA / RAG
    
- Lecture 16：LLM architecture variants / distillation
    

---

## 2. Three LLM architectures【高频】【Architecture题】

### A. Masked Language Model / Encoder-only model

**Definition**  
Masked LM 是 encoder-only Transformer，例如 **BERT, RoBERTa, SpanBERT, DeBERTa**。Pretraining task 是预测被 mask 的 token。slide 例子是 “The ___ of Walden Pond is so beautifully still → water”。

**Intuition**  
它可以同时看左边和右边 context，所以适合理解类任务，例如 classification、NER、POS tagging、span prediction。

**Input / Output**

- Input：corrupted sentence
    
- Output：每个 masked token 的 vocabulary distribution
    
- Training target：原始 token
    

**Formula**

$$L_{MLM}=-\sum_{i\in M}\log P(w_i \mid x_{\setminus M})$$

其中：

- $$M$$：被 mask / corrupted 的 token 位置集合
    
- $$w_i$$：位置 $$i$$ 的真实 token
    
- $$x_{\setminus M}$$：带 mask 的输入序列
    
- $$P(w_i \mid x_{\setminus M})$$：模型对真实 token 的预测概率
    

**Strengths**

- 双向 context
    
- 适合 interpretation tasks
    
- fine-tune 后适合 NER、POS、classification、QA span extraction
    

**Limitations**

- 不是自然的 left-to-right generator
    
- inference 时通常不使用 `[MASK]`，所以 BERT pretraining 中要用 80/10/10 trick 减少 train-test mismatch
    

---

### B. Generative Language Model / Decoder-only model

**Definition**  
Decoder-only model，例如 **GPT, LLaMA, Mixtral, Qwen**。Pretraining task 是 **next-token prediction**。slide 例子是 “The water of Walden Pond is so beautifully ___ → still”。

**Intuition**  
模型从左到右生成 token，每一步只看过去 tokens，不能偷看未来。它天然适合 text generation、chatbot、summarization、QA generation。

**Input / Output**

- Input：prompt + previous generated tokens
    
- Output：next token probability distribution
    
- Training target：下一个 token
    

**Formula**

$$P(w_{1:T})=\prod_{t=1}^{T}P(w_t\mid w_{<t})$$

$$L_{CLM}=-\frac{1}{T}\sum_{t=1}^{T}\log P(w_t\mid w_{<t})$$

其中：

- $$w_{1:T}$$：完整 token sequence
    
- $$w_{<t}$$：位置 $$t$$ 之前的 tokens
    
- $$P(w_t\mid w_{<t})$$：next-token probability
    
- $$L_{CLM}$$：causal language modelling loss
    

**Strengths**

- 生成能力强
    
- chat / instruction following 表现好
    
- 可用 decoding strategy 控制输出
    

**Limitations**

- 对 classification / sequence labelling 未必比 encoder-only 更合适
    
- 需要 causal mask
    
- 可能 hallucinate
    
- 推理逐 token 生成，速度慢
    

老师强调：比较 encoder-only 和 decoder-only 时，不能只看架构，还要比较模型大小。大 decoder-only 商业模型可能因为参数量巨大而表现更强；但如果 encoder-only 和 decoder-only 同规模，encoder-only 通常更适合理解/分类，decoder-only 更适合生成。

---

### C. Text-to-text / Encoder-decoder model

**Definition**  
Encoder-decoder model，例如 **T5**。Input sequence 和 output sequence 可以不同长度，适合 “text → text” 转换任务。slide 例子是 “Make it so ___ → Fallo così”。

**Intuition**  
Encoder 读完整输入，decoder 生成输出。适合 translation、summarization、grammar correction、QA 等 input/output sequence 不同的任务。

**Input / Output**

- Input：source text / task prompt
    
- Output：target text
    

**Strengths**

- 很适合 seq2seq tasks
    
- 比纯 decoder-only 更自然地处理 source-target mapping
    

**Limitations**

- 现代商业 LLM 多采用大规模 decoder-only，导致同等比较困难
    
- 架构和训练成本更复杂
    

---

## 3. Prompting【高频】【老师强调】

### Direct prompting / zero-shot prompting

**Definition**  
给模型一个 basic prompt template + input，不提供例子。

**Intuition**  
通过 prompt 中的 task words 激活模型预训练中相关模式。例如 “Translate this into French” 中的 “French” 引导模型生成法语。老师强调：prompt 是 text，不是人类式理解，而是让相关词汇在 Transformer weights 中“resonate”。

**Input / Output**

- Input：instruction + text
    
- Output：generated answer
    

**Strengths**

- 简单
    
- 不需要 labelled examples
    
- 适合大模型快速试任务
    

**Limitations**

- 对 prompt wording 敏感
    
- 输出不稳定
    
- 对复杂推理任务可能较弱
    

---

### In-context prompting / few-shot prompting

**Definition**  
在 prompt 中提供若干 examples，再给 test input。slide 称为 in-context prompting，也叫 few-shot prompting。

**Intuition**  
不是更新参数，而是通过上下文 examples 让模型模仿输入输出格式。

**Input / Output**

```text
Example 1: input -> output
Example 2: input -> output
New input -> ?
```

**Strengths**

- 不训练模型
    
- 能控制格式，如 JSON / label / short answer
    
- 对任务定义不清时很有用
    

**Limitations**

- 消耗 context window
    
- examples 选择会影响结果
    
- “few-shot” 也可能指 few-shot fine-tuning，术语易混淆；老师更偏向用 in-context prompting 表达这个意思。
    

---

### Chain-of-Thought, CoT

**Definition**  
要求模型 “think step by step”，让模型先生成中间推理，再给答案。slide 还提到 Tree of Thoughts 和 Graph of Thoughts。

**Intuition**  
额外生成的 reasoning tokens 可以激活更适合推理的模式，提高复杂任务准确率。

**Strengths**

- 对 multi-step reasoning / maths / commonsense tasks 有帮助
    
- 可与 in-context prompting 结合
    

**Limitations**

- 生成 token 更多，inference 更慢
    
- reasoning 可能看似合理但错误
    
- 不是严格逻辑证明
    

老师提醒：CoT 往往提高 accuracy，但代价是生成更多 tokens，所以推理速度更慢。

---

### Self-consistency

**Definition**  
让模型生成多个答案，然后 majority vote。slide 指出通常使用较高 temperature，例如 temperature = 1.0，以获得最大变化。

**Intuition**  
如果多个不同采样路径都得到同一答案，这个答案更可靠。

**Strengths**

- 提高鲁棒性
    
- 适合 reasoning / QA
    

**Limitations**

- 成本更高，因为要生成多个答案
    
- majority vote 对开放式生成不一定好用
    

---

### Self-refinement

**Definition**  
先生成答案，再让模型 critique 自己的回答，把 critique 加回 prompt，再生成改进版。

**Intuition**  
把模型当作 critic + generator 两步使用。

**Strengths**

- 可以提升答案质量
    
- 对写作、解释、代码 review 有帮助
    

**Limitations**

- 模型可能无法发现自己的真实错误
    
- 多轮生成成本高
    

---

## 4. LLM Decoding【高频】【数学题】

**Definition**  
Decoding 是根据 model probabilities 选择下一个 token。slide 明确定义：LLM decoding == choose next word based on model probabilities。

### Random sampling

从整个 vocabulary 的 softmax distribution 中随机采样。

**Strength**：多样性高  
**Limitation**：long tail 中 rare words 可能被抽到，输出 weird/gibberish；老师口头也强调随机采样会偶尔生成很怪的词。

---

### Greedy decoding

每次选概率最高 token。

$$\hat{w}=\arg\max_{w\in V}P(w\mid context)$$

**Strength**：稳定、快  
**Limitation**：缺少多样性，可能陷入局部最优

---

### Top-k sampling

保留概率最高的 k 个 token，重新 normalize，再采样。

$$V_k=\text{TopK}(P(w\mid context), k)$$

$$P_k(w)=\frac{P(w)}{\sum_{v\in V_k}P(v)} \quad w\in V_k$$

如果 $$k=1$$，就是 greedy decoding。slide 和 transcript 都强调 top-k 的缺点是 optimal k 依赖 input context。

---

### Top-p sampling / nucleus sampling

保留累计概率质量达到 p 的最小 token 集合。

$$V_p=\min S \subseteq V \text{ such that } \sum_{w\in S}P(w)\ge p$$

**Strength**：相比固定 k 更 adaptive，可以移除 unlikely words。  
**Limitation**：仍然需要调 p；开放式生成可能不稳定。

---

### Temperature sampling

slide 给出公式：temperature sampling 是将 LM head logits $$u$$ 除以 temperature parameter $$t$$，再 softmax。

$$y_i=\frac{\exp(u_i/t)}{\sum_j \exp(u_j/t)}$$

其中：

- $$u_i$$：token $$i$$ 的 logit
    
- $$t$$：temperature
    
- $$y_i$$：softmax 后 token $$i$$ 的概率
    

**Low temperature**：例如 $$t=0.1$$，distribution 更尖锐，更 greedy。  
**High temperature**：例如 $$t=1.0$$，distribution 更平滑，低概率词更可能被采样，输出更多样。

---

## 5. Masked Language Models: BERT【高频】【Lab重点】

BERT pretraining 包括两个重点：

### A. Masked Token Prediction / Cloze task

slide 说明 BERT 会 corrupt 15% tokens，其中 80% 替换为 `[MASK]`，10% 替换为 random token，10% 保持 original token；只有 corrupted tokens contribute to loss。

**Architecture shape**

假设：

- batch size = $$B$$
    
- sequence length = $$N_{seq}$$
    
- vocabulary size = $$|V|$$
    
- hidden size = $$d_{model}$$
    

Input token ids:

$$input\_ids \in \mathbb{R}^{B\times N_{seq}}$$

Embedding output:

$$X \in \mathbb{R}^{B\times N_{seq}\times d_{model}}$$

BERT encoder output:

$$H \in \mathbb{R}^{B\times N_{seq}\times d_{model}}$$

MLM head logits:

$$logits \in \mathbb{R}^{B\times N_{seq}\times |V|}$$

Loss only on masked/corrupted positions:

$$L_{MLM}=-\sum_{i\in M}\log P(w_i\mid x_{\setminus M})$$

---

### B. Next Sentence Prediction, NSP

slide：输入是 sentence pair `(A, B)`，如果 B follows A，则 label = 1，否则 label = 0；BERT 使用 50% random next sentence 和 50% original next sentence；`[CLS]` 是 input start，`[SEP]` 是句子分隔符。

**注意**：RoBERTa 等模型去掉 NSP。

---

## 6. Generative LLM: LLaMA-style architecture【Architecture题】【高频】

slide 中 LLaMA family 是 generative LLM，LLaMA2 up to 70B，LLaMA3 up to 405B；训练数据包括 web、books、code、papers，并使用 BPE tokenization；核心组件包括 Transformer、Pre-normalization layer、SwiGLU activation、RoPE position。

### ASCII architecture diagram

```text
Raw text / prompt
        ↓
BPE tokenizer
        ↓
token ids: (B, Nseq)
        ↓
Token embedding + positional mechanism / RoPE
        ↓
X: (B, Nseq, dmodel)
        ↓
Stacked decoder-only Transformer blocks
        ├─ RMSNorm / pre-norm
        ├─ Masked self-attention / causal attention
        ├─ residual connection
        ├─ RMSNorm / pre-norm
        ├─ Feed-forward network with SwiGLU
        └─ residual connection
        ↓
H: (B, Nseq, dmodel)
        ↓
Language Model Head / unembedding
        ↓
logits: (B, Nseq, |V|)
        ↓
softmax
        ↓
P(next token | previous tokens)
        ↓
decoding: greedy / top-k / top-p / temperature
```

### Language Model Head

Book 中解释：LM head 把最后一层 Transformer 的最后 token hidden state 映射到 vocabulary logits；常用 weight tying，即使用 embedding matrix 的 transpose 作为 unembedding layer。

$$u=h_N^L E^T$$

$$y=softmax(u)$$

其中：

- $$h_N^L \in \mathbb{R}^{1\times d}$$：第 L 层、最后 token 的 hidden state
    
- $$E \in \mathbb{R}^{|V|\times d}$$：token embedding matrix
    
- $$E^T \in \mathbb{R}^{d\times |V|}$$：unembedding matrix
    
- $$u \in \mathbb{R}^{1\times |V|}$$：logits
    
- $$y \in \mathbb{R}^{1\times |V|}$$：softmax probabilities
    

---

### RMSNorm【B级】【易错】

**Definition**  
RMSNorm 是 LayerNorm 的变体。slide 说明 LayerNorm 在 Transformer 中较慢，会成为 LLM bottleneck；RMSNorm 可获得类似性能但降低 runtime。

$$RMSNorm(x)=\frac{x}{\sqrt{\frac{1}{d}\sum_{i=1}^{d}x_i^2+\epsilon}}\odot g$$

其中：

- $$x$$：hidden vector
    
- $$d$$：hidden dimension
    
- $$\epsilon$$：防止除 0 的小常数
    
- $$g$$：learnable scale parameter
    
- $$\odot$$：element-wise multiplication
    

**考试写法**：RMSNorm removes/reduces some LayerNorm computation and is used for efficiency in very deep LLMs.

---

### RoPE: Rotary Position Embedding【B级】【Architecture题】

**Definition**  
RoPE 是 rotary / relative positional embedding。slide 给出两步：先把 encoded query/key 投影到 position $$m$$ 的 latent space，再乘以 sin/cos rotation matrix。优势是 long-term decay：距离远的 token pairs connection 更弱。

简化二维旋转：

$$R_m= \begin{bmatrix} \cos(m\theta) & -\sin(m\theta) \\ \sin(m\theta) & \cos(m\theta) \end{bmatrix}$$

$$q_m'=R_m q_m$$

$$k_n'=R_n k_n$$

**Intuition**  
不是给每个 absolute position 学一个固定 embedding，而是在 query/key 空间中旋转表示，使 attention score 能表达 relative distance。

**易错**  
RoPE 不是普通 learned absolute position embedding；它作用在 query/key 上，不是简单把 position embedding 加到 token embedding 上。

---

### SwiGLU【B级】

slide：SwiGLU 是 classic ReLU activation function 的变体。

常见表达：

$$SwiGLU(x)=(xW_1)\odot Swish(xW_2)$$

$$Swish(z)=z\sigma(z)$$

**Intuition**  
GLU-like gate 控制哪些 hidden features 通过 FFN，比普通 ReLU 更灵活。

---

## 7. LLM Training pipeline【高频】【老师强调】

### Stage 1: Pretraining

大规模 self-supervised training：

- masked LM：predict masked word
    
- causal LM：predict next token
    
- text-to-text：generate target response
    

老师强调：这些 pretraining tasks 能 scale，因为不需要人工标注；直接从已有文本中 mask 或 shift target。

---

### Stage 2: Fine-tuning / domain adaptation

**Definition**  
继续训练 pretrained model，使其适应某个 domain / task。

老师例子：如果要做 mental health classifier，可以下载开源 LLaMA model，然后在相关任务数据上 fine-tune；commercial proprietary model 通常不能下载，只能 API zero-shot 使用。

---

### Stage 3: SFT / Instruction tuning

slide：LLaMA2 Chat 使用 **Supervised Fine Tuning, SFT == Instruction Tuning**；用 27,540 human helpful/safe prompt-answer pairs 继续训练 LLaMA2，仍使用 language model head 和 pretraining objective。

**Exam pipeline**

```text
Base pretrained LLaMA
        ↓
Human-written prompt-answer pairs
        ↓
Continue training with LM objective
        ↓
Instruction-following SFT model
```

---

### Stage 4: RLHF / Preference Alignment

slide：RLHF == Preference Alignment。用 SFT model + temperature 生成两个 possible answers，收集 1.4M human preference decisions，然后把 LLaMA2 的 LM head 替换成 classifier head，训练 reward function，再用 RL fine-tune language model head。

```text
SFT model
   ↓ generate answer A and answer B
Human preference: A better than B
   ↓
Replace LM head with classifier head
   ↓
Train reward model
   ↓
Use reward model in RL loop
   ↓
Aligned chat model
```

Book 也补充了 reward model 的理论 framing：在 RL 里，actions 是 autoregressive generation 中 token choices，states 是当前 decoding context，policy 是 pretrained LLM，reward 来自 preference-trained reward model。

---

# 第四部分：公式、算法、architecture 总结

## 1. Softmax

$$softmax(u_i)=\frac{\exp(u_i)}{\sum_j\exp(u_j)}$$

## 2. Temperature softmax

$$y_i=\frac{\exp(u_i/t)}{\sum_j\exp(u_j/t)}$$

## 3. Causal LM objective

$$P(w_{1:T})=\prod_{t=1}^{T}P(w_t\mid w_{<t})$$

$$L_{CLM}=-\frac{1}{T}\sum_{t=1}^{T}\log P(w_t\mid w_{<t})$$

## 4. Masked LM objective

$$L_{MLM}=-\sum_{i\in M}\log P(w_i\mid x_{\setminus M})$$

## 5. Language model head

$$u=h_N^L E^T$$

$$y=softmax(u)$$

## 6. Self-attention with causal mask

$$A=softmax(\frac{QK^T+M}{\sqrt{d_k}})V$$

其中 causal mask：

$$M_{ij}=0 \text{ if } j\le i$$

$$M_{ij}=-\infty \text{ if } j>i$$

这样当前位置不能看未来 token。

---

# 第五部分：Worked examples

## Worked example 1：Temperature + top-k + top-p decoding【数学题】

假设 vocabulary 只有 3 个 token：

```text
A = "water"
B = "still"
C = "dog"
```

模型输出 logits：

$$u_A=2,\quad u_B=1,\quad u_C=0$$

### Step 1: temperature = 1

$$P(A)=\frac{e^2}{e^2+e^1+e^0}$$

$$P(B)=\frac{e^1}{e^2+e^1+e^0}$$

$$P(C)=\frac{e^0}{e^2+e^1+e^0}$$

数值：

```text
e^2 = 7.389
e^1 = 2.718
e^0 = 1
sum = 11.107
```

所以：

```text
P(A)=0.665
P(B)=0.245
P(C)=0.090
```

### Step 2: temperature = 0.5

先除以 temperature：

```text
u/t = [2/0.5, 1/0.5, 0/0.5] = [4, 2, 0]
```

```text
e^4 = 54.598
e^2 = 7.389
e^0 = 1
sum = 62.987
```

所以：

```text
P(A)=0.867
P(B)=0.117
P(C)=0.016
```

**Conclusion**：temperature 更低，distribution 更尖锐，模型更 greedy。

### Step 3: top-k with k = 2

保留 A 和 B，去掉 C，renormalize：

```text
P(A)=0.665 / (0.665+0.245)=0.731
P(B)=0.245 / (0.665+0.245)=0.269
```

### Step 4: top-p with p = 0.9

按概率排序：

```text
A: 0.665, cumulative = 0.665
B: 0.245, cumulative = 0.910
C: 0.090
```

因为 A+B 已经超过 0.9，所以保留 A 和 B，再 renormalize，结果同 top-k k=2。

**考试答案要点**：  
temperature 改变 probability sharpness；top-k 固定保留 k 个 token；top-p 根据累计概率质量动态保留 token；greedy 是 k=1 的特殊情况。

---

## Worked example 2：Decoder-only LM tensor shape【Architecture题】

假设：

```text
No batching
Nseq = 5
|V| = 30,000
dmodel = 512
```

Input one-hot tokens：

$$X_{onehot}\in\mathbb{R}^{5\times 30000}$$

Embedding matrix：

$$E\in\mathbb{R}^{30000\times 512}$$

Embedding output：

$$X=X_{onehot}E\in\mathbb{R}^{5\times 512}$$

Transformer output：

$$H\in\mathbb{R}^{5\times 512}$$

取最后一个 token：

$$h_N\in\mathbb{R}^{1\times 512}$$

LM head：

$$U\in\mathbb{R}^{512\times 30000}$$

Logits：

$$u=h_NU\in\mathbb{R}^{1\times 30000}$$

Softmax output：

$$y=softmax(u)\in\mathbb{R}^{1\times 30000}$$

**Softmax 输出是什么？**  
它是 vocabulary 上的 probability distribution，每个位置表示下一个 token 是该 vocabulary item 的概率。

---

# 第六部分：Lab / code 考法

## 是否对应 lab？

- Regex lab：不是本 lecture 重点。
    
- CRF lab：不是本 lecture 重点。
    
- **BERT lab：对应，Revision Part B labs 明确列出 BERT。**
    
- **LLaMA lab：slide / transcript 提到 BERT and LLaMA labs 会用 prompts，但 revision lab list 只明确写 Regex、CRF、BERT。因此 LLaMA lab 可作为本 lecture practical support，但考试确定性低于 BERT lab。**
    

---

## 1. 代码阅读题【Lab重点】

**Question**  
For a causal LM, tokenizer output has shape `(B, Nseq)`. The model output `logits` has shape `(B, Nseq, |V|)`. What does `logits[:, -1, :]` represent?

**Answer**  
It represents the vocabulary logits for the next token prediction after the final input token. Shape is `(B, |V|)`.

**思路**  
Decoder-only LM 每个 position 都预测 next token；生成时通常只需要最后一个 position 的 distribution 来采样下一个 token。

---

## 2. Bug identification 题【易错】

**Buggy code**

```python
probs = torch.softmax(logits, dim=-1)
loss = torch.nn.CrossEntropyLoss()(probs.view(-1, vocab_size), labels.view(-1))
```

**Bug**  
`CrossEntropyLoss` expects raw logits, not probabilities after softmax.

**Correct idea**

```python
loss = torch.nn.CrossEntropyLoss()(logits.view(-1, vocab_size), labels.view(-1))
```

**思路**  
PyTorch `CrossEntropyLoss` internally applies log-softmax + negative log likelihood。

---

## 3. Output shape 判断题【Lab重点】

**Question**  
BERT MLM input has shape `(B, Nseq)`, hidden size is `dmodel`, vocab size is `|V|`. What are the shapes of encoder output and MLM logits?

**Answer**

```text
Encoder output H: (B, Nseq, dmodel)
MLM logits:       (B, Nseq, |V|)
```

**思路**  
BERT 是 encoder-only；每个 token position 都有 contextual embedding；MLM head 对每个 position 输出 vocabulary distribution。

---

## 4. 参数计算题【数学题】

**Question**  
A language model head maps hidden size `dmodel = 4096` to vocabulary size `|V| = 32000`. If no weight tying is used, how many parameters are in the linear weight matrix?

**Answer**

$$4096\times 32000=131072000$$

即 **131,072,000 parameters**，如果还有 bias，则再加 $$32000$$。

**思路**  
Linear head shape 是 $$d_{model}\times |V|$$。如果使用 weight tying，LM head 可以使用 embedding matrix transpose，减少额外参数。Book 中明确说明 LM head 常用 weight tying。

---

## 5. Training pipeline 排序题【高频】

**Question**  
Put the following LLaMA-chat training steps in the correct order:

```text
A. Train reward model from human preferences
B. Pretrain base causal LM on web-scale text
C. Supervised fine-tuning / instruction tuning on prompt-answer pairs
D. RL fine-tuning using reward model
E. Generate candidate answers using SFT model
```

**Answer**

```text
B → C → E → A → D
```

**思路**  
先 pretrain base model，再 SFT，让模型能 follow instructions；然后采样多个候选答案并收集 human preference，用 classifier/reward head 训练 reward model，最后 RLHF alignment。

---

# 第七部分：Reference book / wider reading 整合

## 1. Book 补充了 slide 没展开的内容

### A. LM head / unembedding / weight tying【可写进10 marks】

Book 详细解释 LM head 如何从最后 token hidden state $$h_N^L$$ 通过 $$E^T$$ 映射到 vocabulary logits，并用 softmax 得到 probabilities；还解释了 weight tying，即 input embedding matrix 和 output unembedding matrix 共享权重。

**Stronger exam point**  
In modern LMs, the output projection can be tied to the input embedding matrix, reducing parameters and encouraging consistency between encoding and decoding word representations.

---

### B. KV cache【wider reading 加分】

Book 解释 inference 和 training 不同：training 可并行计算 attention，但 inference 逐 token 生成。如果每一步都重新计算过去 tokens 的 key/value 会浪费，因此存储过去的 K/V vectors 到 **KV cache**。

**Stronger exam point**  
KV cache improves autoregressive inference efficiency by avoiding recomputation of keys and values for previous tokens.

---

### C. PEFT / LoRA【wider reading 加分】

Book 说明大模型 full fine-tuning 很贵，因此有 **parameter-efficient fine-tuning, PEFT**；LoRA 是一种 Low-Rank Adaptation 方法，只更新部分低秩参数，而不是所有 LLM weights。

**Stronger exam point**  
For domain adaptation, full fine-tuning may be impractical for very large LLMs, so PEFT methods such as LoRA freeze most parameters and train a small number of adaptation parameters.

---

### D. Preference learning / reward model【wider reading 加分】

Book 对 RLHF 做了理论化解释：LLM 的 token choices 是 actions，decoding context 是 state，pretrained LLM 是 policy，reward 来自 preference-trained reward model。

**Stronger exam point**  
RLHF can be understood as optimizing a language-model policy using rewards learned from human preference comparisons.

---

## 2. Paper / blog 可支持的考试答案

- **Self-consistency, Wang 2023**：支持 “generate multiple answers + majority vote”。
    
- **Self-refinement, Madaan 2023**：支持 “generate → critique → refine”。
    
- **BERT, Devlin 2018**：支持 MLM + NSP + encoder-only Transformer。
    
- **LLaMA2, Touvron 2023**：支持 LLaMA architecture、SFT、RLHF。
    
- **RMSNorm, Zhang 2019**：支持 normalization efficiency。
    
- **RoPE, Su 2024**：支持 rotary / relative positional encoding。
    
- **SwiGLU, Shazeer 2020**：支持 FFN activation variant。
    

这些 paper/blog 的完整文本当前资料中未找到；slide 中列为 directed reading 或引用来源。

---

# 第八部分：可能考试题预测

## Q1. [5 marks recall]

**Question**  
Name and explain three types of large language model architecture.

**中文答题思路**  
写 encoder-only masked LM、decoder-only causal/generative LM、encoder-decoder text-to-text LM。每个写 example、pretraining objective、适合任务。

---

## Q2. [10 marks worked example]

**Question**  
Given logits for next-token prediction, compute the softmax distribution under different temperatures and explain greedy, top-k, and top-p decoding.

**中文答题思路**  
先 softmax；temperature 低则更 greedy，高则更多样；top-k 固定取 k 个；top-p 按累计概率质量取候选集合；k=1 是 greedy。

---

## Q3. [10 marks worked example]

**Question**  
Draw an architecture diagram for a decoder-only LLM and explain the tensor shapes from token input to softmax output.

**中文答题思路**  
画 BPE → embedding → decoder Transformer blocks → LM head → logits → softmax。写 `(B,N)`、`(B,N,d)`、`(B,N,|V|)`。说明 causal mask 和 next-token prediction。

---

## Q4. [10 marks wider reading]

**Question**  
Explain how instruction tuning and RLHF are used to adapt a pretrained LLM into a chat model. Include the role of human preference data.

**中文答题思路**  
pretraining → SFT/instruction tuning with prompt-answer pairs → generate two candidate answers → human preference labels → train reward model/classifier head → RL fine-tune LM。加 book 中 policy/action/state/reward framing 可得高分。

---

## Q5. [10 marks wider reading]

**Question**  
Compare BERT and LLaMA as language models. Discuss architecture, attention direction, pretraining objective, output head, and suitable NLP tasks.

**中文答题思路**  
BERT encoder-only，bidirectional，MLM + NSP，适合理解、classification、NER、QA span。LLaMA decoder-only，causal attention，next-token prediction，适合 generation、chat、summarization。注意模型大小影响公平比较。

---

## Q6. [10 marks wider reading]

**Question**  
Explain RMSNorm, RoPE, and SwiGLU in modern LLM architectures and why they are useful.

**中文答题思路**  
RMSNorm 提升效率；RoPE 用旋转表达 relative position，长距离 decay；SwiGLU 是 gated activation。不要过度数学推导，重点说它们解决什么工程/建模问题。

---

## 最可能考的一题：完整英文 exam-style answer

**Question**  
Compare masked language models such as BERT with generative decoder-only language models such as LLaMA. Your answer should discuss architecture, training objective, attention direction, input/output, and suitable NLP applications.

**Exam-style answer**

Masked language models such as BERT and generative decoder-only models such as LLaMA are both Transformer-based language models, but they are designed for different objectives and NLP tasks.

BERT is an encoder-only Transformer. Its main pretraining objective is masked language modelling. During training, some tokens in the input sentence are corrupted, for example by replacing them with `[MASK]`, and the model is trained to recover the original tokens. This can be written as $$L_{MLM}=-\sum_{i\in M}\log P(w_i\mid x_{\setminus M})$$, where $$M$$ is the set of masked positions. Because BERT uses bidirectional self-attention, each token can attend to both left and right context. This makes BERT especially suitable for interpretation tasks such as text classification, named entity recognition, part-of-speech tagging, and extractive question answering. Its output is usually a contextual embedding for each input token, which can then be passed to a task-specific classifier head.

In contrast, LLaMA is a decoder-only generative language model. Its pretraining objective is causal next-token prediction. The model estimates $$P(w_t\mid w_{<t})$$ and is trained with a loss such as $$L_{CLM}=-\frac{1}{T}\sum_{t=1}^{T}\log P(w_t\mid w_{<t})$$. It uses causal self-attention, so a token can only attend to previous tokens and not future tokens. The final hidden state is passed through a language modelling head to produce logits over the vocabulary, and a softmax converts these logits into a probability distribution for the next token. This architecture is naturally suited to generation tasks, including dialogue, summarization, open-ended question answering, and instruction following.

The two model types also differ in decoding. BERT is not normally used to generate long text left-to-right, whereas LLaMA generates one token at a time using decoding strategies such as greedy decoding, top-k sampling, top-p sampling, or temperature sampling. Modern LLaMA-style models also use architectural variants such as RMSNorm, RoPE, and SwiGLU to improve efficiency and performance at scale.

Overall, BERT is stronger as a bidirectional text understanding model, while LLaMA is stronger as an autoregressive text generation model. However, when comparing real systems, model size and training data must also be considered, since a much larger decoder-only model may outperform a smaller encoder-only model even on some understanding tasks.

---

# 第九部分：考前速记版

## 5–10 个 must-know points

1. 【高频】LLM 通常是 Transformer-based，slide 定义为通常 `>1B parameters`。
    
2. 【高频】三类架构：BERT = encoder-only masked LM；LLaMA/GPT = decoder-only generative LM；T5 = encoder-decoder text-to-text LM。
    
3. 【高频】BERT pretraining：MLM + NSP；MLM corrupts 15% tokens，80% `[MASK]`，10% random，10% original，只在 corrupted tokens 上算 loss。
    
4. 【高频】Decoder-only LM objective：next-token prediction，使用 causal mask，不能看未来 token。
    
5. 【数学题】Temperature formula：$$y_i=\frac{\exp(u_i/t)}{\sum_j\exp(u_j/t)}$$。
    
6. 【高频】Top-k：保留 top k；k=1 是 greedy。Top-p：保留累计概率达到 p 的 token set。
    
7. 【Architecture题】LM head：hidden state → logits → softmax vocabulary distribution。
    
8. 【高频】LLaMA-style components：BPE、causal Transformer、RMSNorm、RoPE、SwiGLU。
    
9. 【高频】Chat model training：pretraining → SFT / instruction tuning → reward model → RLHF / preference alignment。
    
10. 【老师强调】Prompt 是 text，不要 anthropomorphise model；不同模型可能需要不同 prompt。
    

## 3–5 个 high-risk mistakes【易错】

1. 把 BERT 当成普通 left-to-right generator。BERT 是 bidirectional encoder-only，不是 causal decoder。
    
2. 以为 prompt 会更新模型参数。Prompting 不更新 weights；fine-tuning 才更新。
    
3. 把 top-k 和 top-p 混淆。Top-k 固定数量；top-p 固定累计概率质量。
    
4. 忘记 CrossEntropyLoss 需要 raw logits，不要先 softmax。
    
5. 比较 BERT 和 LLaMA 时只比较架构，不考虑参数量和训练数据规模。老师口头强调模型大小会强烈影响实际表现。
    

## 3–5 个 likely exam questions

1. Compare BERT, LLaMA, and T5 architectures.
    
2. Explain decoding strategies with a worked softmax / temperature example.
    
3. Draw a decoder-only LLM architecture and compute tensor shapes.
    
4. Explain BERT MLM training and why 80/10/10 corruption is used.
    
5. Explain SFT and RLHF in LLaMA2 Chat training.
    

## 最应该背的 formula / architecture / algorithm

**Formula**

$$y_i=\frac{\exp(u_i/t)}{\sum_j\exp(u_j/t)}$$

$$L_{CLM}=-\frac{1}{T}\sum_{t=1}^{T}\log P(w_t\mid w_{<t})$$

$$L_{MLM}=-\sum_{i\in M}\log P(w_i\mid x_{\setminus M})$$

**Architecture**

```text
Prompt → BPE tokens → embeddings → causal Transformer blocks
→ LM head → logits → softmax → decoding → generated token
```

**Training algorithm**

```text
Pretrain on web-scale text
→ fine-tune / domain adaptation
→ SFT / instruction tuning with prompt-answer pairs
→ collect human preferences
→ train reward model
→ RLHF / preference alignment
```

## transcript 中最值得记住的老师提醒【老师强调】

- 不要 anthropomorphise LLM；它们不是人，只是 Transformer models。
    
- Prompt 是 text，本质是用词汇激活 pretrained weights 中的相关模式。
    
- 不同模型需要不同 prompt，不能假设一个 prompt 在所有模型上都好用。
    
- CoT / self-consistency 可能提高准确率，但会增加生成 token，推理更慢。
    
- Top-k 的最佳 k 很依赖输入和任务；temperature 是更常用的 controllability knob。
    
- 比较 encoder-only 和 decoder-only 时必须考虑 model size；大模型可能靠 brute force 表现更强。
    
- SFT 让模型更会 follow prompts；RLHF / preference alignment 让回答更 helpful/safe/human-like。