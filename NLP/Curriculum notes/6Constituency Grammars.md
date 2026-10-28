你好！我是你的课程助教。根据你上传的课堂录音（Transcript）和课件（PDF），我为你整理了关于 **Constituency Grammars（短语结构语法/成分语法）** 和 **Dependency Grammars（依存语法）** 的本节课详细学习指南。

这节课不仅涵盖了理论基础，还涉及了具体的解析算法步骤和考试重点（老师特别强调了手动推导解析树的能力）。

---

### 📚 课程概览 (Executive Summary)

本节课主要探讨了自然语言处理中**句法分析（Syntactic Parsing）**的两种核心流派：

1. **成分语法 (Constituency Grammars)**：基于上下文无关语法 (CFG)，关注单词如何组合成短语（如名词短语 NP、动词短语 VP），强调层次结构和词序 。
    
2. **依存语法 (Dependency Grammars)**：关注单词之间的二元关系（谁修饰谁，如主语、宾语），强调词与词的功能关系，对词序不敏感，适合处理自由词序语言 。

此外，课程还详细讲解了**歧义（Ambiguity）**的处理、**Treebanks（树库）**的标注标准，以及如何评估解析器的性能（LAS/UAS）。

---

### 📝 详细内容梳理

#### 1. 为什么要学习语法？ (Motivation)

老师列举了6个理由，解释为什么在 LLM 时代语法依然重要 ：

- **LLM 的静默失败**：LLM 产生不合语法的句子时不会报错，语法规则可以用来约束输出 。
    
- **高风险场景**：医疗或法律领域需要形式化语法明确判定“对”或“错”，而不是概率性的输出 。
    
- **结构决定意义**：词袋模型无法区分 "The dog bit the man" 和 "The man bit the dog"，结构至关重要 。
    
- **低资源语言**：缺乏海量数据训练 LLM 时，传统语法方法依然有效 。
    
- **生成能力**：CFG 可以生成无限的句子，而 LLM 受限于训练数据的频率 。
    
- **可解释性**：形式化语法解释了语言的内在结构知识 。
    

#### 2. 关键概念与定义 (Definitions)

- **Syntax (句法)**：单词排列的规则 。
    
- **Constituent (成分)**：作为一个单一单元起作用的单词组（例如：名词短语）。
    
- **Context-Free Grammar (CFG)**：也称短语结构语法。由**终结符**（单词）、**非终结符**（词性/短语标签）、**产生式规则**（Productions）和**开始符号**（S）组成 。
#### 3. 歧义 (Ambiguity)

歧义是解析的最大挑战。主要类型包括 ：

- **结构歧义 (Structural Ambiguity)**：同一句话有多种合法的解析树。
    
- **附着歧义 (Attachment Ambiguity)**：
    
    - _例句_："We saw the Eiffel Tower flying to Paris".
        
    - _解释_：是我们在飞往巴黎时看到了塔？还是塔自己在飞？(修饰语 `flying to Paris` 附着在 `We` 还是 `Tower` 上) 。
        
- **并列歧义 (Coordination Ambiguity)**：
    
    - _例句_："Old men and women".
        
    - _解释_：是 `(Old men) and women` 还是 `Old (men and women)`？。
#### 4. 上下文无关语法 (CFG) 详解

- **BNF 范式**：用于定义语法的元语言。例如 `NP -> Det Nominal` 表示名词短语由限定词和名词性成分组成 。
    
- **Nominal (名词性成分)**：老师特别提问了为什么要用 Nominal 而不仅仅是 Noun。
    
    - _原因_：Nominal 允许在该层级添加形容词或其他修饰语（如 "a _long morning_ flight"），保持结构的递归性 。
        
- **英语语法规则**：
    
    - **句子 (S)**：通常由 NP + VP 组成 。
        
    - **子句 (Clause)**：表达完整思想的单元，包含主语、动词等 。
        
    - **动词分类**：及物动词（带宾语）、不及物动词、双及物动词（直接+间接宾语，如 "She told me the story"）。
#### 5. 依存语法 (Dependency Grammars)

- **核心**：没有短语节点（如 NP, VP），只有单词节点和带标签的有向边（Arc/Edge）。
    
- **Root (根)**：没有入边（incoming arcs）的词，通常是主动词 。
    
- **投射性 (Projectivity)**：
    
    - 如果从头节点（Head）到依赖节点（Dependent）的路径覆盖了它们之间所有的单词，则该弧是投射的。
        
    - **非投射性 (Non-projective)**：常见于从句或长距离依赖。
        
    - _考点例子_："JetBlue canceled our flight this morning which was already late". 这里 `flight` 和 `was` 的连接跨越了 `this morning`，构成了非投射结构（交叉线）。
**依存语法（Dependency Grammars）**的核心思想是：摒弃传统的短语结构（如 NP、VP），直接分析句子中**词与词之间的二元修饰关系** 。这种结构呈现为一个有向图，由“中心词”（Head）指向它的“依赖词”（Dependent），并标明它们之间的语法功能（如主语、宾语等） 。

下面我结合课件中老师布置的随堂推导例子，为你详细拆解依存语法是如何一步步分析句子的。

---

##### 📝 具体分析步骤与例子

**例句**：_"I love NLP with parse trees"_

我们可以按照课件（Slide 30）提供的标准 6 个步骤来进行分析 ：

- **第一步：确定根节点 (Step 1: Identify the root)** 我们需要找到句子的核心词（通常是主要谓语动词）。在这里是 `love` 。根节点是图中唯一一个**没有入边**（没有箭头指向它）的词 。
    
    - _操作_：将 `love` 标记为 `ROOT`。
        
- **第二步：连接主语 (Step 2: Attach subject)** 找出是谁发出了 `love` 这个动作。这里是 `I` 。
    - _操作_：画一条从中心词 `love` 指向依赖词 `I` 的箭头，并在箭头上标注关系为 `nsubj`（名词主语） 。
- **第三步：连接直接宾语 (Step 3: Attach direct object)** 找出 `love` 的动作承受者，也就是被爱的对象。这里是 `NLP` 。
    
    - _操作_：画一条从 `love` 指向 `NLP` 的箭头，标注关系为 `dobj`（直接宾语） 。
        
- **第四步：连接介词修饰语 (Step 4: Attach prepositional modifier)** 句子中有一个介词 `with`，在这里它作为一个修饰成分附加在主动词上 。
    - _操作_：画一条从 `love` 指向 `with` 的箭头，标注关系为 `prep`（介词修饰） 。
        
- **第五步：连接介词宾语 (Step 5: Attach object of preposition)** 介词 `with` 后面带了什么宾语？是 `trees` 。
    - _操作_：画一条从 `with` 指向 `trees` 的箭头，标注关系为 `pobj`（介词宾语） 。
- **第六步：连接复合名词修饰语 (Step 6: Attach compound noun modifier)** 最后，名词 `trees` 前面还有一个名词 `parse` 在修饰它，共同构成复合词“解析树” 。
    - _操作_：画一条从中心词 `trees` 指向依赖词 `parse` 的箭头，标注关系为 `compound`（复合修饰） 。
![[Pasted image 20260223131948.png]]
---

##### 📖 核心关系标签速查 (Terminology Dictionary)

在上述分析中，我们用到了一些缩写标签，这些标签由语言学家定义，用于描述具体的语法功能 ：


- **ROOT**: 根节点，句子的起点。
    
- **nsubj** (Nominal subject): 名词主语 。
    
- **dobj** (Direct object): 直接宾语 。
    
- **nmod / prep**: 名词修饰语 / 介词修饰语 。
    
- **case**: 介词或格标记（在某些依存标注标准中，介词如 _with, to_ 会被标为 case 并指向其后面的名词） 。

---
##### ⚠️ 教师重点复述 (Key Takeaways for Exams)

在课堂录音中，老师对于这种依存树的画法特别强调了以下几点，请在复习时牢记：

1. **箭头的方向绝对不能画反！** (Transcript @41:54) 老师明确指出，如果在考试中要求画依存图，你必须清楚箭头是从 **Head（中心词）** 指向 **Dependent（依赖词）** 。这是最容易失分的地方。
2. **找根节点的技巧** (Transcript @44:28) 有些同学喜欢从 Root 开始往下画（Top-down），有些同学喜欢先把明确的修饰词搭配画好，最后剩下的那个没有被任何人指着的词就是 Root 。不管用哪种方法，验证 Root 的唯一标准就是它**没有接收任何输入弧线（incoming arcs）** 。
    
3. **为什么这种图有用？** (Transcript @43:03) 尽管它不关注词序（word order doesn't matter），但它精准提炼了**谓语和论元（语义角色）之间的关系**，这对于高阶的自然语言推理（Reasoning）任务非常有用 。
---

### ⚠️ 教师重点强调 (Key Emphasis)

1. **考试预警：手动解析 (Manual Parsing)**
    
    - 老师明确提到："If we were to ask this on an exam..." 。
        
    - **要求**：你需要能够根据给定的语法规则，手动画出句子的 **CFG 解析树** 或 **依存关系图**。
        
    - **策略**：对于 CFG，建议使用**自底向上 (Bottom-up)** 的策略。先标记每个单词的词性（Terminals），然后根据规则两两合并（Reduce），直到形成 S 节点 。
        
2. **术语定义的准确性**
    
    - 建议背诵 Syntax, Constituent, Clause 等术语的定义，考试中可能会利用精准的术语简化问题描述 。
3. **依存关系的方向**
    
    - 在画依存图时，箭头必须是从 **Head（中心词）** 指向 **Dependent（修饰词/依赖词）**。例如：在 "I prefer" 中，`prefer` 是 Head，`I` 是 Dependent，箭头 `prefer -> I` 。
        

---

### 🧠 核心算法与代码逻辑

#### 1. 自底向上解析 (Bottom-Up Parsing Example)

针对句子 _"I love NLP with parse trees"_ 的解析步骤（对应幻灯片 20-23）：

1. **Tagging**：查词典，标记词性。
    
    - I (PRP), love (VBP), NLP (NN), with (IN), parse (NN), trees (NNS).
        
2. **Reduction (归约)**：
    
    - `PRP -> NP` (I 变成 NP).
        
    - `IN NN NNS` 无法直接归约，但 `NN (parse) -> NP`.
        
    - `IN (with) NP (parse) NNS (trees)` -> 此时规则匹配 `PP -> IN NP` (假设有此规则) 或者根据课件 `PP -> IN NN NNS` (课件 slide 21 步骤3) 直接归约成 PP。
        
    - `VBP (love) NP (NLP) PP (with...)` -> 归约为 `VP`。
        
    - 最后 `NP (I) + VP (...)` -> `S`.
        

#### 2. NLTK 代码片段 (Python)

老师展示了使用 Python NLTK 库进行解析的基本逻辑 ：

Python

```
import nltk

# 1. 定义句子 (Token list)
sent = ['I', 'shot', 'some', 'dinosaurs']

# 2. 定义语法 (CFG)
# 注意：语法必须覆盖句子中的词汇
dino_grammar = nltk.CFG.fromstring("""
  S -> NP VP
  NP -> Det N | Pro
  VP -> V NP
  Det -> 'some'
  N -> 'dinosaurs'
  V -> 'shot'
  Pro -> 'I'
""")

# 3. 创建解析器 (Chart Parser)
parser = nltk.parse.chart.ChartParser(dino_grammar)

# 4. 解析并打印树
for tree in parser.parse(sent):
    print(tree)
    # tree.draw() # 如果在本地运行可由图形化显示
```

---

### 📊 复习速查表 (Cheat Sheet)

|**概念**|**成分语法 (Constituency)**|**依存语法 (Dependency)**|
|---|---|---|
|**结构核心**|短语 (NP, VP, PP)|词与词的关系 (Head-Dependent)|
|**词序**|非常重要，结构固定|词序较自由，关注功能关系|
|**图结构**|树状层次结构 (Parse Tree)|有向图 (Directed Graph)|
|**根节点**|句子符号 (S)|Root (通常是主动词)|
|**主要应用**|语法检查，精确结构分析|语义角色标注，多语言分析|
|**评估指标**|ParseEval (括号匹配)|**LAS** (标签+附件正确率), **UAS** (仅附件正确率)|

**常见依存关系标签 (Universal Dependencies):**

- `nsubj`: 名词主语 (Nominal subject)
    
- `dobj`: 直接宾语 (Direct object)
    
- `nmod`: 名词修饰语
    
- `det`: 限定词 (the, a)
    
- `case`: 介词 (prepositions)
    

---

### ✍️ 课后练习 (Self-Practice Solution)

**题目**：对句子 _"I love NLP with parse trees"_ 进行依存句法分析 。

+1

**推导步骤**：

1. **Root**: `love` (主动词，无入边) 。
    
2. **nsubj (主语)**: `I` 是 `love` 的主语 -> `love` 指向 `I`。
    
3. **dobj (宾语)**: `NLP` 是 `love` 的对象 -> `love` 指向 `NLP`。
    
4. **prep (介词修饰)**: `with` 修饰 `love` (表示方式/伴随) -> `love` 指向 `with`。
    
5. **pobj (介词宾语)**: `trees` 是 `with` 的宾语 -> `with` 指向 `trees`。
    
6. **compound (复合词/修饰)**: `parse` 修饰 `trees` -> `trees` 指向 `parse`。
    
    _(注：不同标注集可能将 with parse trees 整体视为 NLP 的修饰，取决于语义理解是“用解析树爱NLP”还是“爱带解析树的NLP”。根据幻灯片30页的图，`with` 是指回 `love` 的)_。
    

**最终结构 (对应 Slide 30)**:

- love -> I (nsubj)
    
- love -> NLP (dobj)
    
- love -> with (prep)
    
- with -> trees (pobj)
    
- trees -> parse (compound)
    

建议你遮住答案，自己尝试画一遍这个图。