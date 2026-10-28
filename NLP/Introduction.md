### 1. 课程精要 (Executive Summary)

- **课程名称**：Natural Language Processing (COMP 3225/6253)
    
- **授课教师**：Prof. Stuart Middleton (负责 LLM, 信息抽取) & Dr. Jennifer Williams (负责语音, Deepfake) 。
    
- **核心目标**：学习 NLP 的基础（如 Regex, 语言模型）、深度学习模型（RNN, Transformers）、以及最新的 LLM 技术（BERT, GPT, LLaMa）。
    
- **考核方式**：**100% 闭卷考试**。没有课程作业（Coursework），但必须完成并通过实验（Labs）签到 。
    
- **关键演示**：演示了 ChatGPT 的四种提示策略（直接提示、角色扮演、上下文提示、思维链 CoT）。
    

---

### 🚨 2. 教师“重点复述” (Teacher's Emphasis)

_以下内容在讲座中被反复强调或标记为考试重点，请务必留意：_

- **考试结构非常固定**：考试包含 5 道题，每题 25 分。每题结构如下：
    
    1. **5分**：基础记忆（Recall，来自课件）。
        
    2. **10分**：应用与计算/实验相关（Worked examples/Labs）。**特别警告**：经常会考“Tensor Shapes”（张量形状）的变化，这需要你亲自做实验才能掌握 。
        
    3. **10分**：深度理解与扩展阅读（Wider reading）。这部分需要阅读 Jurafsky & Martin 的在线书和论文，光看 PPT 是拿不到这 10 分的 。
        
- **实验（Labs）是强制性的**：
    
    - 虽然实验不直接评分，但必须“Sign-off”（签到验收）。
        
    - **死线**：复活节前必须完成 Regex 和 CRF 的实验签到，否则会被个人导师（PAT）“约谈” 。
        
    - **资源**：如果你没有强大的 GPU，建议使用学校实验室机器（Zepler Building 3）或远程 GPU 服务 。
        
- **教材版本警告**：必须使用 Jurafsky & Martin 的 **第三版（在线版）**。不要用 2009 年的第二版纸质书，内容已经过时了 。
    
- **学术诚信与 AI 使用**：
    
    - 可以使用学校提供的 **Microsoft Copilot (Enterprise Edition)**，因为它保护数据隐私 。
        
    - **严禁**使用 Grok 或 DeepSeek 处理任何个人数据或作业，因为它们会抓取数据 。
        
    - **严禁**直接复制粘贴 AI 生成的内容作为自己的报告，否则视为抄袭 。
        

---

### 📖 3. 课程详细梳理 (Content Breakdown)

#### 3.1 什么是 NLP？ (What is NLP)

- **定义**：引用 ACL (Association for Computational Linguistics) 的定义——“从计算视角对语言进行的科学研究” 。
    
- **应用场景**：拼写检查（Word）、机器翻译（Google Translate）、问答系统（Watson）、对话代理（Siri/Alexa）、新闻生成（BBC 自动撰稿后人工校对）。
    
- **现状**：虽然 ChatGPT 可以零样本（Zero-shot）完成上述所有任务，但并不总是最优解。课程目标之一是让你知道**何时使用 LLM，何时使用传统方法** 。
    

#### 3.2 课程大纲 (Module Roadmap)

课程内容由浅入深，分为几个阶段 ：

1. **基础 (Basics)**：词汇（Words）、正则表达式（Regex）、语言模型。
    
2. **序列标注 (Sequence Labeling)**：词性标注（POS）、命名实体识别（NER）。
    
3. **深度学习入门 (Deep Learning Intro)**：向量语义（Vector Semantics）、Word2Vec。
    
4. **序列处理 (Sequence Processing)**：RNN、Encoder-Decoder、**Transformers（重点，会有双倍课时讲解数学原理）** 。
    
5. **大型语言模型 (LLMs)**：BERT, GPT, LLaMa。会有来自清华大学/智谱 AI 的客座教授 Ji Tang 分享 。
    
6. **应用 (Applications)**：机器翻译、问答、RAG（检索增强生成）。
    

#### 3.3 实验安排 (Lab Details)

- **技术栈**：Python, PyTorch, Scikit-learn, Conda 。
    
- **核心实验**：
    
    1. **Regex** (Week 2): 正则表达式。
        
    2. **CRF** (Week 3): 条件随机场，用于 NER（非深度学习方法）。
        
    3. **BERT** (Week 5): 文本分类与摘要。
        
    4. **LLaMa** (Week 8): 聊天机器人风格的问答 。
        

#### 3.4 演示：提示工程 (Demo: Prompt Engineering)

原计划演示机器人，但未准备好，改为演示 ChatGPT 提示策略 。

老师演示了四种从简单到复杂的提示方法，以“The Matrix (黑客帝国)”电影为例：

1. **直接提示 (Direct Prompting)**：
    
    - _Prompt_: "Provide details about the locations used for The Matrix."
        
    - _结果_: 给出基本的悉尼、美国等地点，包含少量细节 。
        
2. **角色扮演提示 (Role-based Prompting)**：
    
    - _Prompt_: "**You are a film director.** Provide details..."
        
    - _原理_: 增加上下文词汇，激活模型中特定的参数区域。
        
    - _结果_: 输出更专业，包含具体的拍摄建筑、场景细节 。
        
3. **上下文提示/少样本提示 (In-context / Few-shot Prompting)**：
    
    - _Prompt_: 给定几个示例（Example 1 -> Class X, Example 2 -> Class Y），然后问新问题。
        
    - _技巧_: 这是一个虚构的任务（XYZ分类），模型通过学习给定的例子来模仿输出格式（例如只输出标签 'Class X'） 。
        
4. **思维链提示 (Chain of Thought, CoT)**：
    
    - _Prompt_: "Let's think step by step." (让我们一步步思考) 。
        
    - _原理_: 强制模型在生成最终答案前生成中间推理步骤。
        
    - _效果_: 现在的模型（如演示中的 Copilot）通常会在后台自动执行此操作（显示为 "Thinking..."），能显著提高复杂推理的准确性 。
        

---

### 📚 4. 核心术语表 (Terminology & Concepts)

|**术语**|**全称**|**解释与备注**|
|---|---|---|
|**ACL**|Association for Computational Linguistics|NLP 领域的顶级学术组织和会议。若论文发表在 ACL，通常质量很高 。|
|**NLP**|Natural Language Processing|从计算角度研究语言的科学 。|
|**Transformer**|-|现代 LLM 的基石架构。课程将深入讲解其架构和数学原理 。|
|**Zero-shot**|Zero-shot Learning|零样本学习。指直接给模型任务指令，而不提供任何示例（如直接问 ChatGPT）。|
|**CoT**|Chain of Thought|思维链。通过提示模型“一步步思考”或生成中间推理步骤来提高准确率 。|
|**Tensor Shape**|-|张量形状。深度学习中数据在神经网络各层流动时的维度变化（如 `[batch_size, seq_len, hidden_dim]`）。**考试必考点** 。|
|**RAG**|Retrieval Augmented Generation|检索增强生成。结合搜索引擎/数据库与 LLM 来生成答案，减少幻觉 。|

---

### 🛠️ 5. 实操建议 (Actionable Advice)

#### 关于环境配置

- 如果你打算用自己的电脑，确保安装好 **Anaconda**, **Python**, **PyTorch**。
    
- 虽然 Google Colab 可用，但可能需要付费才能跑长时间的任务。
    
- **建议**：先做完 Python Optional Lab（如果基础薄弱），熟悉 Numpy 和 Pandas 。
    

#### 关于复习策略

- **每周投入**：建议预留 2 个下午（约 7 小时）进行阅读和实验 。
    
- **阅读材料**：不要只看 PPT。考试中有 10 分是专门针对“扩展阅读”的（比如问你除了 Word2Vec 还有哪些 Embedding 方法，这在 PPT 里可能没有）。
    

---

### 🧠 6. 复习速查卡 (Review Flashcards)

**Q1: 这门课的考核方式是什么？**

- A: 100% 闭卷考试。没有 Coursework。
    

**Q2: 考试中“Tensor Shapes”题目怎么复习？**

- A: 必须亲自做实验（Labs）。实验代码会打印出模型各层的张量形状，这是回答此类问题的关键。
    

**Q3: 什么是 Chain of Thought (CoT) 提示？**

- A: 一种提示工程技术，通过要求模型“Let's think step by step”或提供推理步骤示例，让模型在给出最终答案前进行逻辑推理，从而提高准确率。
    

**Q4: 为什么不建议在任何场景都使用 LLM？**

- A: LLM 成本高、速度慢、可能产生幻觉。对于简单的任务（如拼写检查或特定领域的实体识别），传统的小模型可能更快、更准、更便宜。
    

**Q5: NLP 的定义是什么？**

- A: The scientific study of language from a computational perspective (ACL definition).