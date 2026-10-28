# ECSP6002 Coursework 3：Project Plan Portfolio 最终版复习笔记

**主题融合版：Project Plan 写作要求 + 老师课堂强调 + 你的 AI-generated Text Detection 项目方案**

## 0. 快速摘要

这次 Coursework 3 的本质不是写一篇完整 dissertation，而是写一份**可执行、可评估、可和导师讨论的项目计划书**。核心逻辑是：**Aim / Research Question 决定 Objectives，Objectives 决定 Activities，Activities 通过 Methodology 执行，Timeline 展示活动关系，Operational / Ethical / Legal / Commercial 部分证明你理解研究的现实约束**。老师反复强调：目标必须 realistic、achievable、outcome-based；计划不要写成一条线走到底，而要把风险、并行任务、fallback 和 dissertation 写作都放进计划中。对于你的项目，最强 framing 是：**不是再做一个 AI 文本检测器，而是把 AI-generated text detection 作为 robustness-evaluation problem 来研究。**

---

# 1. 课程与作业整体定位

## 1.1 Coursework 3 是什么？

**Coursework 3: The Project Plan Portfolio** 是一个约 **5 页**的项目计划作品集，目标是让你证明你已经理解了自己的暑期毕业项目主题、研究方法、实际资源需求、时间安排、伦理/法律/商业/环境影响。官方 guide 明确说，这份 portfolio 聚焦于 summer project 的 **aims and objectives**、执行要求、research methods，以及不同活动的 rough timescales and durations；同时还必须覆盖 professional and ethical aspects、data management plan、ethical / legal / commercial impact、sustainability 和 responsible innovation。

它不是普通 essay，也不是 poster 的简单扩写，而是一个**研究执行蓝图**。老师在课堂转写中也强调：Coursework 3 是一个 outlining project plan，每个 section 都有具体要求；你应该读 coursework document，并在和 supervisor 讨论时同步记笔记，因为这些讨论可以直接转化为各 section 的内容。

## 1.2 5 页结构

根据 lecture slide 和 coursework guide，结构大致是：

1. **1 page：Aims / Research Question / Objectives**
    
2. **1 page：Summary of proposed research and analysis methodology**
    
3. **1 page：Research plan Gantt chart / flow diagram / PERT chart / network diagram**
    
4. **约 2 pages：Data Management, ethical statements, health & safety, environmental impact, ethical/commercial/legal aspects**
    

Lecture slide 直接把 Project Plan 定义为一个 5-page document：第一页 aims/research question/objectives，第二页 methodology summary，第三页 Gantt chart or similar，最后两页 data management、ethical statements、commercial/legal aspects。

## 1.3 评分标准的核心含义

Marking scheme 中，A* 要求不是“写得长”，而是：

- **Aim**：clear, explanatory statement
    
- **Objectives**：excellent and comprehensive set，能 coherent guide to project
    
- **Methodology**：clear, succinct, explanatory description of experimental and analytical methods
    
- **Plan**：excellent chart，清楚展示 activities、relationships、outcomes、length of activities、inter-relationship
    
- **Ethics / Operational / Impact**：excellent data plan，clear ethical plan，insightful description of ethical, commercial and legal aspects of both project and research area
    

A 档 / Distinction 要求 aim 好、objectives well thought out、methodology outline 好、chart 能表示活动关系和时间需求、data plan 清楚、ethical/commercial/legal aspects 描述良好。

所以你要拿 Distinction，不能只写“我要训练模型、做实验、写报告”。你需要表现出：

**我知道我要回答什么研究问题；我知道如何产生证据；我知道每一步如何支撑 objective；我知道失败时怎么办；我知道这个研究的伦理、法律、商业和社会影响。**

---

# 2. 教师重点复述：这节课真正要记住什么

下面是老师反复强调、对拿高分最重要的内容。我把课堂转写与我们之前的问答融合成复习版。

## 2.1 Aim 必须 realistic and achievable

老师在 3:55–4:14 处强调，aim 的关键是 realistic 和 achievable，必须能在 **13 weeks** 里完成。

对你的项目来说，这意味着：

不要写成：

> This project aims to solve AI-generated text detection.

这太大、不可完成、也不是 MSc project 能证明的目标。

应该写成：

> This project aims to evaluate the robustness of representative AI-generated text detection methods under bilingual, paraphrased, short-text, and cross-domain/cross-generator conditions.

这个 aim 更好，因为它明确了：

- **对象**：representative detection methods
    
- **任务**：evaluate robustness
    
- **条件**：bilingual / paraphrased / short / cross-domain / cross-generator
    
- **边界**：不是解决整个 AI detection problem，而是研究可靠性和失效条件
    

## 2.2 Objectives 必须 outcome-based

老师说 objectives 应该是 “things which you can point to, things which you can prove”，也就是能展示、能收集信息、能写进 dissertation 的结果。

这点非常重要。Objective 不能写成“learn PyTorch”“read papers”“train a model”，因为这些是活动，不是研究结果。

弱 objective：

> To learn how to use Hugging Face Transformers.

强 objective：

> To implement and evaluate a supervised multilingual transformer detector on a bilingual human-vs-AI text dataset.

前者只是学习过程；后者有明确输出：implemented detector + evaluation results。

## 2.3 Aim / Objectives / Methodology / Timeline 是同一件事的不同表示

老师在 4:43–5:19 说，objectives form the basis of work activities；activities form the basis of the plan；mechanism for carrying them out is explained in methodology；everything fits together as different representations of the same information。

这句话是整份 coursework 的核心。

你应该把它理解成下面这个链条：

```text
Aim / Research Question
        ↓
Objectives: what evidence must be produced?
        ↓
Activities: what work must be done to produce that evidence?
        ↓
Methodology: how exactly will the work be done?
        ↓
Timeline: when will each activity happen, and what depends on what?
        ↓
Dissertation: where will each piece of evidence be written up?
```

如果你写得好，老师会感觉你的 plan 是一个 coherent research design，而不是零散任务清单。

## 2.4 Methodology = how，不是 what

老师在 6:43–7:02 明确区分：

- **Objectives**：what you are going to show / do
    
- **Methodology**：how you are going to do it
    

例如：

Objective：

> To compare traditional, supervised transformer-based, and zero-shot detection methods.

Methodology：

> Use TF-IDF + logistic regression / linear SVM as traditional baselines; fine-tune a multilingual transformer such as XLM-R or mDeBERTa for supervised classification; evaluate a statistical/zero-shot method such as perplexity-based detection or Fast-DetectGPT on a controlled subset due to computational cost.

这里 objective 是“比较三类方法”，methodology 是“具体用哪些方法、怎么实现、为什么这样做”。

## 2.5 研究计划不是为了预测未来，而是为了管理风险

老师在 14:21–14:52 说，plan of work 不会完全正确，因为 research changes as you do it；它的主要作用是给你 milestones，让你知道自己进展如何。

这意味着 Gantt chart 不需要假装每一步都一定成功，而是要表现出：

- 哪些任务可以并行；
    
- 哪些任务依赖前置结果；
    
- 哪些是 core；
    
- 哪些是 optional / nice-to-have；
    
- 如果某一步失败，有什么 fallback；
    
- dissertation writing 从第一天开始并行。
    

## 2.6 不要问 “if something goes wrong”，要问 “when something goes wrong”

Lecture slide 直接把传统线性项目计划的问题写成：

> What do I do if something goes wrong?

然后下一页改成：

> What do I do WHEN something goes wrong?

这就是老师的风险管理核心：不要默认完美执行，而要默认研究一定会遇到问题。

对你的项目，典型风险包括：

- DetectGPT / Fast-DetectGPT 算力太贵；
    
- RAID / MGTBench 数据格式复杂或下载困难；
    
- paraphrase generation 成本过高；
    
- 中英文分布不均；
    
- transformer fine-tuning 显存不足；
    
- 某个方法结果很差；
    
- 时间不够做 full ablation。
    

你不应该在 plan 里回避这些，而应该写 fallback：

```text
If DetectGPT-style evaluation is computationally expensive, it will be restricted to a representative subset or replaced by a perplexity-based zero-shot baseline.

If full RAID/MGTBench evaluation is too large for the available compute, the project will use a controlled subset stratified by domain, generator, language, and text length.
```

## 2.7 Dissertation 从第一天开始写，不是最后才写

老师在 42:56–45:12 强调：所有人都有一个共同目标，就是写 dissertation；dissertation starts on the first day and ends on the last project；它应当和所有任务并行。

这对你非常关键。你的 timeline 里必须有一条贯穿 13 周的任务：

```text
Dissertation writing, experiment logging, result interpretation, figure/table preparation
```

而不是最后两周突然写 dissertation。

原因是：

- literature review 可以边读边写；
    
- dataset description 可以在清洗数据后写；
    
- methodology 可以在实现 pipeline 后写；
    
- baseline results 出来后可以先写 preliminary results；
    
- error analysis 可以边做边写；
    
- limitations 和 ethics 可以不断补充。
    

这也是降低风险的策略：如果某个实验卡住，你仍然可以推进写作。

---

# 3. Page 1：Aims / Research Question / Objectives

## 3.1 Page 1 的任务

官方 guide 要求第一页 lay out main aim / research question，并确保它 realistic、achievable、unambiguous；objectives 应该构成你打算做的 work activities 的基础，并 inform the plan of work。

Lecture slide 也强调，objectives should be outcome based，不能把“操作方式”误写成 objective。

## 3.2 Aim、Research Question、Objectives 的区别

### Aim

**Aim（研究目标 / 总体目的）** 是项目总体想完成什么。它通常是一段话，不要太长。

你的项目推荐 aim：

```text
The aim of this project is to investigate how reliable current AI-generated text detection methods are when evaluated beyond ideal benchmark settings. The project will compare representative detection approaches for distinguishing AI-generated from human-written text, with particular attention to bilingual English–Chinese data, paraphrased AI-generated text, short text samples, and cross-domain or cross-generator generalisation.
```

这版 aim 的优点是：

- 不承诺发明最强 detector；
    
- 不把项目写成普通 binary classification；
    
- 把贡献定位为 robustness evaluation；
    
- 和你的可用数据、方法、13 周时间匹配。
    

### Research Question

**Research Question（研究问题）** 是把 aim 转化成一个可以回答的问题。

你的项目推荐 RQ：

```text
How robust are representative AI-generated text detection methods when applied to bilingual, paraphrased, short, and cross-domain text, and which methodological choices provide the most reliable evidence for practical deployment?
```

这个 RQ 包含两层：

1. **诊断性问题**：detectors 在哪些条件下失败？
    
2. **方法论问题**：哪些 evaluation choices 更能提供可靠证据？
    

所以你的项目不是只问 “which model has highest accuracy?”，而是问 “under what conditions do detectors remain reliable or fail?”

## 3.3 Objectives 应该怎么写？

你的 objectives 应该是可以被证据支持的结果。推荐结构如下。

### Objective 1：文献与问题界定

```text
To critically review recent AI-generated text detection methods and identify robustness issues related to language, paraphrasing, text length, domain shift, and unseen generators.
```

输出证据：

- literature review section；
    
- method taxonomy；
    
- robustness dimensions table；
    
- justification for chosen datasets and methods。
    

### Objective 2：构建 evaluation framework

```text
To construct a reproducible evaluation framework using public datasets and controlled robustness settings, including bilingual English–Chinese evaluation, paraphrased text, short-text subsets, and cross-domain or cross-generator testing.
```

输出证据：

- dataset table；
    
- preprocessing scripts；
    
- split protocol；
    
- robustness subset definitions；
    
- leakage prevention说明。
    

### Objective 3：实现代表性 baseline / detector

```text
To implement and compare representative detection paradigms, including a traditional TF-IDF-based classifier, a supervised multilingual transformer detector, and a statistical or zero-shot detection baseline where computationally feasible.
```

输出证据：

- implemented models；
    
- training logs；
    
- hyperparameter table；
    
- reproducible configuration files。
    

### Objective 4：鲁棒性评估与误差分析

```text
To evaluate detector performance using aggregate and subgroup metrics, including accuracy, macro-F1, AUROC, false positive rate, confusion matrices, and performance degradation across robustness conditions.
```

输出证据：

- overall results table；
    
- subgroup results；
    
- confusion matrices；
    
- robustness degradation plots；
    
- false positive / false negative examples。
    

### Objective 5：测试一个可行改进

```text
To investigate whether a feasible robustness-oriented improvement, such as paraphrase-aware augmentation or consistency-based training, can reduce performance degradation under paraphrased or shifted input conditions.
```

输出证据：

- ablation study；
    
- before/after comparison；
    
- robustness gain table；
    
- limitations discussion。
    

### Objective 6：形成部署层面的结论

```text
To derive practical recommendations on when AI-generated text detectors should be trusted, when they should be treated with caution, and how their outputs should be communicated responsibly.
```

输出证据：

- discussion section；
    
- ethical implications；
    
- deployment recommendations；
    
- uncertainty / false positive warnings。
    

## 3.4 进阶解析：为什么你的贡献不是 “another detector”

你之前担心这句话：

```text
Many detectors report strong results on controlled datasets, but their reliability in realistic settings remains uncertain...
```

会显得没有依据，也显得你和别人一样。

更强的表达是：

```text
Rather than proposing another detector evaluated mainly by aggregate benchmark scores, this project treats AI-generated text detection as a robustness-evaluation problem. It focuses on how detector reliability changes under bilingual, paraphrased, shortened, and cross-domain input conditions, and on what forms of evidence are needed before such detectors can be used responsibly.
```

这句话的意义是：

- **Rather than proposing another detector**：明确区别于普通 model-building project；
    
- **evaluated mainly by aggregate benchmark scores**：指出传统评价只看总体 accuracy/F1 的不足；
    
- **robustness-evaluation problem**：把项目升维到评估框架和 failure analysis；
    
- **what forms of evidence are needed**：强调贡献不只是模型结果，而是 practical deployment evidence。
    

这比“我会训练一个 detector，然后看 accuracy”更像 Distinction 级研究设计。

---

# 4. Page 2：Proposed Research and Analysis Methodology

## 4.1 Page 2 的任务

官方 guide 说，methodology page 是 practical research methods and analytical methods 的 simple summary；它不需要过度细节，但要证明你知道哪些方法 suitable / ideal / possible，并简要说明为什么。

Lecture slide 也说，这一页是 experimental or research design framework，用来和 supervisor 讨论。

老师在转写中进一步强调：proper experimental research methods approach involves both **acquisition of data** and **analysis of data**；计划时必须同时考虑这两部分。

所以你 Page 2 不能只写模型。你要写：

```text
Data acquisition and preparation
        +
Detector implementation
        +
Evaluation protocol
        +
Analysis methodology
        +
Resources / feasibility / fallback
```

## 4.2 你的项目方法论总框架

你的项目可以定义为：

**Comparative robustness evaluation of AI-generated text detection methods.**

它不是单一模型训练，而是一个 evaluation pipeline：

```text
Public datasets
    ↓
Preprocessing and split construction
    ↓
Robustness condition construction
    ↓
Detector implementation
    ↓
Aggregate evaluation
    ↓
Subgroup / robustness evaluation
    ↓
Failure analysis
    ↓
Practical recommendations
```

## 4.3 数据集选择

### HC3

HC3 适合作为主数据集，因为它包含 human-written 与 ChatGPT-generated answer，并且有 English–Chinese bilingual setting。它适合支撑你的 bilingual robustness evaluation。

你可以在 methodology 中写：

```text
HC3 will be used as the main bilingual benchmark because it provides human-written and ChatGPT-generated answers in both English and Chinese. This makes it suitable for evaluating whether detection methods behave consistently across languages rather than only on English text.
```

### RAID / MGTBench

RAID 或 MGTBench 可作为 robustness 或 cross-domain / cross-generator evaluation 的补充。由于它们可能更大、更复杂，计划里应写成 “considered” 或 “used on a controlled subset”。

推荐写法：

```text
RAID and/or MGTBench will be considered for broader robustness evaluation, particularly for domain shift, generator shift, and adversarial or paraphrased settings. If full-scale evaluation is not feasible within available compute, a stratified subset will be used.
```

这里体现了风险意识，不是盲目承诺。

## 4.4 方法选择：三类 detector

你的方法最好分成三类，因为这样能证明你不是只调一个模型，而是比较不同 detection paradigms。

### 方法一：Traditional baseline

推荐：

```text
TF-IDF + Logistic Regression
TF-IDF + Linear SVM
```

作用：

- 快速；
    
- 可解释；
    
- 低算力；
    
- 可作为 lower-bound baseline；
    
- 能帮助展示 transformer 是否真的必要。
    

传统方法的直觉：

AI 文本检测可以看作文本分类，TF-IDF 把文本转成词/子词统计特征，线性分类器学习哪些模式更像 human 或 AI。

TF-IDF 的核心思想可以写成源码公式：

`$\mathrm{tfidf}(t,d)=\mathrm{tf}(t,d)\times \log\frac{N}{\mathrm{df}(t)}$`

其中：

- `$t$` 是 token / term；
    
- `$d$` 是 document；
    
- `$N$` 是文档总数；
    
- `$\mathrm{df}(t)$` 是包含 term `$t$` 的文档数。
    

Logistic regression 的输出可以写成：

`$p(y=1|x)=\sigma(w^\top x+b)$`

其中 `$y=1$` 可以表示 AI-generated，`$y=0$` 表示 human-written。

### 方法二：Supervised multilingual transformer detector

推荐候选：

```text
XLM-R
mDeBERTa
multilingual RoBERTa-style encoder
```

你的项目中最适合写：

```text
A supervised multilingual transformer detector will be fine-tuned for binary classification. This method is expected to capture semantic and stylistic patterns beyond sparse lexical features, while still being feasible within the project timeframe using pretrained models.
```

工程实现上：

```python
from transformers import AutoTokenizer, AutoModelForSequenceClassification

model_name = "xlm-roberta-base"  # or "microsoft/mdeberta-v3-base"

tokenizer = AutoTokenizer.from_pretrained(model_name)
model = AutoModelForSequenceClassification.from_pretrained(
    model_name,
    num_labels=2
)
```

输入输出形态：

```text
Input:
    text: a human-written or AI-generated answer

Output:
    logits: [logit_human, logit_ai]
    probability:
        P(human), P(ai)
```

分类概率：

```text
$$
p(y=k|x)=\frac{\exp(z_k)}{\sum_{j=1}^{K}\exp(z_j)}
$$
```

二分类交叉熵：

```text
$$
\mathcal{L}_{CE}=-\left[y\log p+(1-y)\log(1-p)\right]
$$
```

关键超参数：

```text
max_length: 256 / 512 / 1024 depending on text length and GPU memory
learning_rate: 1e-5 to 3e-5
batch_size: constrained by GPU memory
epochs: usually 3–5 for initial experiments
early_stopping: based on validation macro-F1 or AUROC
random_seed: fixed for reproducibility
```

### 方法三：Statistical / Zero-shot baseline

候选：

```text
Perplexity-based detection
DetectGPT
Fast-DetectGPT
```

直觉：

- 人类文本和机器文本可能在语言模型下表现出不同 probability / curvature；
    
- DetectGPT 类方法通常不需要 supervised training，但可能需要大量语言模型推理；
    
- Fast-DetectGPT 更高效，但仍可能比传统 baseline 贵。
    

你应该这样写风险：

```text
Zero-shot methods such as DetectGPT-style curvature-based detection may be computationally expensive. Therefore, this component will be evaluated on a representative subset or replaced by a simpler perplexity-based baseline if necessary.
```

Perplexity 源码公式：

```text
$$
\mathrm{PPL}(x)=\exp\left(-\frac{1}{N}\sum_{i=1}^{N}\log p(x_i|x_{<i})\right)
$$
```

解释：

- PPL 越低，说明语言模型越“容易预测”这段文本；
    
- 某些机器生成文本可能更平滑、更 predictable；
    
- 但这不是绝对规律，所以它适合作为 baseline，不适合作为唯一证据。
    

## 4.5 Robustness conditions 怎么设计？

你的项目最重要的不是“跑一个 clean test”，而是构造不同 robustness condition。

### Condition 1：In-domain clean evaluation

目的：

```text
Measure baseline performance under standard benchmark conditions.
```

这相当于普通 benchmark 结果。

### Condition 2：Bilingual evaluation

比较：

```text
English subset vs Chinese subset
```

看 detector 是否对不同语言表现不一致。

风险：

- 英文训练数据多，中文少；
    
- tokenizer 对中文和英文处理方式不同；
    
- detector 可能学到语言分布偏差，而不是 AI/human 特征。
    

### Condition 3：Short-text evaluation

把文本按长度分组：

```text
short
medium
long
```

例如：

```text
short: <= 100 words / tokens
medium: 100–300
long: > 300
```

原因：

短文本信息少，风格信号弱，detector 更容易误判。

### Condition 4：Paraphrased AI text

把 AI-generated text 经过 paraphrasing 后再检测。

目的：

```text
Test whether detectors rely on surface-level generation artifacts that disappear after rewriting.
```

这能很好体现你的 robustness contribution。

### Condition 5：Cross-domain / cross-generator

训练在一个 domain / generator，测试在另一个 domain / generator。

目的：

```text
Evaluate whether the detector generalises beyond the data distribution it was trained on.
```

这比普通 random split 更有研究价值。

## 4.6 分析指标与公式

你的 evaluation 不能只写 accuracy，因为 AI 文本检测有 false positive 的伦理风险。你需要 accuracy、macro-F1、AUROC、false positive rate、confusion matrix、subgroup breakdown。

### Accuracy

```text
$$
\mathrm{Accuracy}=\frac{TP+TN}{TP+TN+FP+FN}
$$
```

优点：直观。  
缺点：类别不平衡时可能误导。

### Precision

```text
$$
\mathrm{Precision}=\frac{TP}{TP+FP}
$$
```

含义：被模型判为 AI 的文本中，真正 AI 的比例。

### Recall

```text
$$
\mathrm{Recall}=\frac{TP}{TP+FN}
$$
```

含义：所有 AI 文本中，被检测出来的比例。

### F1

```text
$$
F1=\frac{2\cdot \mathrm{Precision}\cdot \mathrm{Recall}}{\mathrm{Precision}+\mathrm{Recall}}
$$
```

### Macro-F1

```text
$$
\mathrm{MacroF1}=\frac{1}{C}\sum_{c=1}^{C}F1_c
$$
```

含义：先对每个类别算 F1，再平均。  
优点：比 accuracy 更关注少数类表现。

### False Positive Rate

```text
$$
\mathrm{FPR}=\frac{FP}{FP+TN}
$$
```

在你的项目里，FPR 极其重要。  
如果 human-written text 被误判为 AI-generated，这可能导致学术不公、错误指控、信任损害。

### Robustness degradation

你可以定义一个 performance drop：

```text
$$
\Delta_{\mathrm{robust}}=M_{\mathrm{clean}}-M_{\mathrm{robust}}
$$
```

其中：

- `$M_{\mathrm{clean}}$` 是 clean setting 的指标；
    
- `$M_{\mathrm{robust}}$` 是 paraphrased / short / cross-domain setting 的指标。
    

这能直接回答：

> detector 在现实扰动下掉了多少？

### Subgroup gap

语言或长度组之间的差异：

```text
$$
\mathrm{Gap}_{lang}=|M_{\mathrm{English}}-M_{\mathrm{Chinese}}|
$$
```

如果 gap 很大，就说明 detector 对不同语言不公平或不稳定。

## 4.7 进阶解析：为什么评估不需要大量 human volunteers？

你之前担心很多评估方法需要志愿者，短期项目不现实。

你的项目可以避免这个问题，因为它不是研究“人类是否觉得聊天愉快”这种主观评价，而是研究已有 public datasets 上的 detector robustness。评价方式是 automatic metrics + subgroup analysis + error analysis。

你可以和导师这样解释：

```text
Because this project focuses on detector robustness rather than user perception, the main evaluation can be conducted using public labelled datasets and controlled perturbation settings. This avoids the need for a large-scale human-subject study while still allowing rigorous quantitative analysis.
```

也就是说，你的项目不需要招募很多人来打分，而是用：

- public labelled datasets；
    
- controlled splits；
    
- automatic metrics；
    
- error case inspection；
    
- robustness degradation analysis。
    

这更适合 MSc 13-week project。

---

# 5. Page 3：Timeline / Gantt Chart / PERT Chart

## 5.1 Page 3 的任务

官方 guide 要求使用 flow diagram、network diagram、PERT chart 或 Gantt chart，展示 rough idea of plan of work；重点是 relative timescales、inter-relationship between activities、objective endpoints or milestones，并且前 3–4 周要更详细。

Lecture slide 也强调，chart 要展示 relative lengths、inter-relationship、milestones，以及 optional / “nice to have” aspects。

## 5.2 Gantt chart 为什么“错但要用”？

老师课堂上说，academics are obsessed with Gantt charts；严格来说 Gantt chart 更适合 running projects，不是 planning projects，但因为 supervisors expect one，所以没有特别想法时就用 Gantt chart。

这句话的实际含义是：

- 你可以用 Gantt chart，因为老师熟悉；
    
- 但不要把它做成死板线性图；
    
- 你要加 dependencies、milestones、parallel tasks、optional tasks；
    
- 最好用颜色或标记区分 core / optional / fallback。
    

## 5.3 推荐 13 周计划

下面是适合你 AI-generated text detection 项目的 13 周计划。

```text
Week 1–2:
    Literature review and final scope definition
    Confirm datasets, methods, robustness dimensions
    Start dissertation notes from day one

Week 2–3:
    Data acquisition and data management setup
    Download HC3 and candidate robustness datasets
    Define raw/processed/logs/models/results directory structure

Week 3–4:
    Dataset cleaning, EDA, split construction
    Build in-domain, bilingual, short-text, and cross-domain/cross-generator evaluation splits
    Milestone M1: reproducible dataset pipeline completed

Week 4–5:
    Traditional baseline implementation
    TF-IDF + logistic regression / linear SVM
    Milestone M2: first baseline results

Week 5–7:
    Supervised multilingual transformer implementation
    Fine-tune XLM-R / mDeBERTa
    Tune basic hyperparameters
    Milestone M3: transformer detector results

Week 6–8:
    Statistical / zero-shot baseline
    Perplexity-based or Fast-DetectGPT subset evaluation
    Fallback: use smaller subset or simpler perplexity baseline
    Milestone M4: zero-shot comparison available

Week 7–9:
    Robustness evaluation
    Paraphrased text, short text, bilingual subgroup, domain/generator shift
    Milestone M5: robustness tables and degradation plots

Week 8–10:
    Robustness improvement / ablation
    Paraphrase-aware augmentation or consistency-based training
    Milestone M6: before/after robustness comparison

Week 10–11:
    Error analysis and practical interpretation
    False positives, language gap, short-text failure cases
    Milestone M7: failure-mode analysis completed

Week 11–13:
    Dissertation consolidation, final figures, final editing
    Risk buffer for missing experiments
    Milestone M8: final dissertation completed

Week 1–13:
    Continuous dissertation writing, experiment logging, reference management, supervisor feedback
```

## 5.4 进阶解析：线性计划为什么危险？

弱计划：

```text
Week 1: Read papers
Week 2: Clean data
Week 3: Train baseline
Week 4: Train transformer
Week 5: Run DetectGPT
Week 6: Do robustness evaluation
Week 7: Write dissertation
```

问题：

- 如果 Week 4 transformer 失败，后面全断；
    
- 如果 DetectGPT 太慢，后面无结果；
    
- 如果最后才写 dissertation，风险极高；
    
- 没有 fallback；
    
- 没有 optional/core 区分；
    
- 没有体现 objective 之间的关系。
    

强计划：

```text
Core path:
    Dataset pipeline → traditional baseline → transformer → robustness evaluation → dissertation

Parallel path:
    Literature review + writing + data management + ethics + result logging

Optional path:
    Fast-DetectGPT / full RAID / paraphrase augmentation

Fallback path:
    smaller subset / perplexity baseline / reduced model / fewer robustness conditions
```

这体现了老师说的 **non-linear** 和 **risk-managed** plan。

## 5.5 SMART objectives 与 measurable / time-based

Lecture slide 提到 SMART：Specific、Measurable、Attainable、Relevant、Time Based，并且老师特别强调 measurable 和 time-based。

对你来说：

- **Specific**：不是“study AI detection”，而是“evaluate robustness of representative detectors under bilingual/paraphrased/short/cross-domain settings”；
    
- **Measurable**：用 accuracy、macro-F1、AUROC、FPR、performance drop；
    
- **Attainable**：用 public datasets、pretrained models、subset fallback；
    
- **Relevant**：和 AI-generated text detection 可靠性、false accusation risk、deployment relevance 相关；
    
- **Time-based**：每个 objective 有 milestone 和 deadline。
    

---

# 6. Section 2：Operational Aspects

## 6.1 Data Management Plan

官方 guide 明确说，所有项目都有 recording and storing research data 的 ethical requirement；Data Management Plan applies to ALL projects。

老师也强调，everybody requires a data management plan，即使只是把 code 放在 GitHub、最后存档，也必须说明。

你的项目 Data Management Plan 应该包含：

### 数据来源

```text
The project will use public text datasets only, such as HC3 and, if feasible, RAID or MGTBench. Dataset names, versions, access dates, licences or terms of use, and original URLs or access routes will be recorded in a data log.
```

### 原始数据与处理数据分离

```text
Raw datasets will be stored separately from processed datasets. Raw files will not be manually edited. All cleaning, filtering, and split construction decisions will be implemented through reproducible scripts.
```

### 文件夹结构

```text
project/
    data/
        raw/
        processed/
        splits/
    src/
        preprocessing/
        training/
        evaluation/
    configs/
    models/
    logs/
    results/
    figures/
    dissertation/
```

### 版本控制

```text
Source code, configuration files, README documentation, and small example files will be stored in a Git repository. Large datasets and model checkpoints will not be committed directly to version control.
```

### 实验记录

```text
Each experiment will be logged with model name, dataset split, robustness condition, random seed, hyperparameters, date, hardware, and output metrics.
```

### 隐私与文本样例

```text
Although the datasets are public, human-written text will be treated carefully. Any personally identifying information discovered during inspection will not be reproduced in the dissertation. Example texts will be paraphrased, shortened, or anonymised where necessary.
```

## 6.2 Ethical Statement

官方 guide 说，如果项目有 direct ethical issues，例如 human data，需要说明 issues、controls，以及如何申请 University ethical approval；即使没有直接伦理审批，也要讨论技术应用领域的 ethical implications。

Lecture slide 也强调，technology always has ethical implications。

老师课堂中解释 morals 和 ethics 的区别：

- **Morals**：个人或群体对 right / wrong 的主观判断；
    
- **Ethics**：为了以 moral way 行动而共同认可的规则；
    
- 技术只要影响人，就有 ethical implications。
    

你的项目伦理重点不是“我会不会伤害参与者”，而是：

### False positive harm

AI detector 最大伦理风险是把人类写作误判为 AI-generated，尤其在教育场景中可能导致学生被错误指控。

你应该写：

```text
A key ethical concern is false positive classification, where human-written text is incorrectly labelled as AI-generated. In educational or professional contexts, this could lead to unfair suspicion, reputational harm, or punitive decisions.
```

### Cross-language bias

如果 detector 英文表现好、中文表现差，就可能对 multilingual users 不公平。

```text
The project will therefore report subgroup performance by language and text length rather than relying only on aggregate scores.
```

### Responsible communication

不要把 detector 输出写成 absolute truth。

```text
Detector outputs should be interpreted as probabilistic and fallible evidence, not as automatic proof of misconduct.
```

这和你项目的 practical contribution 强相关：你不是做惩罚工具，而是做 reliability evaluation。

## 6.3 Health and Safety Statement

对于你的项目，Health and Safety 主要是 computer-based work / DSE risk，不涉及实验室硬件、化学品、机器人或人体实验。

可写：

```text
The project is computer-based and does not involve physical laboratory work. The main health and safety considerations are standard display screen equipment risks, including eye strain, posture, and prolonged computer use. These will be managed by regular breaks, appropriate workstation setup, and safe working practices.
```

## 6.4 Environmental Impact Statement

你的项目环境影响主要来自 GPU training / inference。

可写：

```text
The environmental impact is mainly associated with computational energy use during model fine-tuning and inference. This will be reduced by using pretrained models, limiting unnecessary hyperparameter sweeps, caching intermediate outputs, using early stopping, and restricting computationally expensive zero-shot methods to representative subsets where necessary.
```

这和老师课堂中关于 AI/data centre energy impact 的讨论一致：AI 技术不是没有环境成本，不能只问 “can I do it”，还要问 “should I do it / what is the cost?”。

---

# 7. Section 3：Project Impact and Limitations

## 7.1 Section 3 的任务

官方 guide 说，这部分是 summary discussion，不需要 in-depth investigation，但要 clear and easy to follow；ethical、commercial、legal 三部分不是完全可分的，可以 cross-reference；更重要的是要覆盖 broader research area，而不只是你的 narrow project idea。

Page 4 具体要求：

- **1/2 page Sustainability, ethical and environmental aspects，minimum 200 words**
    
- **1/2 page Commercial aspects，minimum 200 words**
    
- **1/2 page Legal aspects，minimum 200 words**
    

## 7.2 Sustainability, ethical and environmental aspects

你的项目可以这样展开：

AI-generated text detection 的社会价值在于帮助教育、出版、新闻、招聘、平台治理等领域理解文本来源。但 detector 不是中立工具，它的错误会直接影响人。尤其是 false positive，会把真实作者错误标记为 AI 使用者；false negative 则可能让大量机器文本绕过审查。因此，真正负责任的 detector 不应该只输出 “AI / human” 二元标签，而应输出概率、置信度、限制说明和不确定性警告。

你的研究贡献是把 detector 放在更现实的鲁棒性条件下评价：bilingual、paraphrased、short text、cross-domain、unseen generator。这能揭示 detector 什么时候可靠、什么时候不可靠。环境方面，项目会尽量减少 compute waste，通过 pretrained models、limited sweeps、early stopping、subset evaluation 和 caching 控制 GPU 使用。

## 7.3 Commercial aspects

Commercial potential 不能只写“这个系统可以赚钱”。要写 endpoint of the research area，而不是只写你的 MSc 项目 prototype。官方 guide 明确说 commercial aspects 应聚焦 likely endpoint of the research，而不只是你的具体项目终点。

你的商业分析可以写：

AI-generated text detection 在教育、出版、新闻、招聘、内容审核和网络安全领域有商业潜力。但商业价值取决于 reliability、explainability 和 responsible use。一个只在 clean benchmark 上表现好的 detector 不适合直接部署，因为真实用户可能 paraphrase、translate、shorten、mix human and AI writing，或者使用 unseen models。

因此，你项目的商业价值不是“做一个 detector dashboard”，而是为未来 detector 产品提供 evaluation evidence。一个更合理的商业 endpoint 是：

```text
A detector dashboard that provides probability scores, confidence indicators, subgroup reliability warnings, and evidence summaries, while explicitly warning users that the result should support human review rather than replace it.
```

## 7.4 Legal aspects

你的 legal section 可以覆盖以下方面，但要写成“relevant frameworks”，不要写成法律建议。

在英国语境下，涉及人类文本、用户数据、自动化判断和教育/就业场景时，data protection 是核心法律维度。Data Protection Act 2018 的官方 explanatory notes 说明，该法案为数字时代的个人数据处理设置新的保护标准，并包括 general data processing、law enforcement processing、intelligence services processing 和 regulatory oversight/enforcement 等内容。([legislation.gov.uk](https://www.legislation.gov.uk/ukpga/2018/12/notes/division/2/index.htm?utm_source=chatgpt.com "Data Protection Act 2018"))

如果 detector 被用于处理可识别个人的文本，UK GDPR / Data Protection Act 2018 下的 lawful basis、transparency、data minimisation、security、accountability 等原则会变得重要。ICO 关于 AI 和数据保护的材料也通常把 AI 系统中的个人数据处理、透明性、公平性和自动化决策风险视为关键合规问题。([legislation.gov.uk](https://www.legislation.gov.uk/ukpga/2018/12/notes/division/9/index.htm?utm_source=chatgpt.com "Data Protection Act 2018"))

如果项目或未来产品面向欧盟市场，还应注意 EU AI Act。European Commission 说明，AI Act 已于 2024 年 8 月 1 日生效，采用 risk-based approach，目标是促进 responsible AI development and deployment，并处理 health, safety and fundamental rights risks。([European Commission](https://commission.europa.eu/news/ai-act-enters-force-2024-08-01_en?utm_source=chatgpt.com "AI Act enters into force - European Commission")) Article 113 的官方应用时间表显示，AI Act 的多数规定从 2026 年 8 月 2 日适用，但部分章节从 2025 年 2 月 2 日和 2025 年 8 月 2 日开始适用，高风险系统相关部分有延后安排。([ai-act-service-desk.ec.europa.eu](https://ai-act-service-desk.ec.europa.eu/en/ai-act/article-113?utm_source=chatgpt.com "Article 113: Entry into force and application | AI Act Service Desk"))

对你的项目，legal implication 的重点是：

- public dataset licence / terms of use；
    
- human-written text 的隐私和再发布限制；
    
- detector 输出不能被误用为自动惩罚依据；
    
- 如果商业化，需要明确 liability；
    
- 如果涉及 educational misconduct decision，需要 human review 和 appeal mechanism；
    
- model / dataset / generated paraphrase 的 copyright 与 IP 问题。
    

---

# 8. 你的最终项目方案：Distinction-level framing

## 8.1 项目标题

推荐标题：

```text
Robust and Cross-Lingual Evaluation of AI-Generated Text Detection Methods
```

中文解释：

**AI 生成文本检测方法的鲁棒性与跨语言评估**

这个标题比 “AI-generated text detection” 更好，因为它明确告诉评分人你的创新点是：

- robust evaluation；
    
- cross-lingual setting；
    
- method comparison；
    
- failure condition analysis。
    

## 8.2 项目中心论点

你可以把整个 project plan 的中心论点写成：

```text
The project does not aim to claim that one detector can reliably solve AI-generated text detection. Instead, it investigates how different detection paradigms behave when benchmark assumptions are relaxed. Its main contribution is a structured evaluation of reliability, degradation, and failure modes under conditions closer to practical use.
```

中文含义：

你不承诺“解决 AI 文本检测”，而是研究：

- detector 在理想 benchmark 外是否可靠；
    
- 哪些条件下 performance 会下降；
    
- 哪些方法更稳；
    
- 哪些部署场景必须谨慎；
    
- 应该用什么证据支撑实际使用。
    

这就是你的 distinction angle。

## 8.3 项目贡献层次

### Contribution 1：Comparative evaluation

比较三类方法：

```text
Traditional lexical baseline
Supervised multilingual transformer
Statistical / zero-shot baseline
```

### Contribution 2：Robustness framework

不仅 clean test，还包括：

```text
bilingual
paraphrased
short-text
cross-domain
cross-generator
```

### Contribution 3：Failure analysis

不是只报总体分数，而是分析：

```text
Which language fails?
Which length group fails?
Which detector has high false positive rate?
Does paraphrasing destroy detection?
Does supervised training overfit to dataset-specific artifacts?
```

### Contribution 4：Feasible robustness improvement

例如：

```text
paraphrase-aware augmentation
consistency training
threshold calibration
```

### Contribution 5：Responsible deployment recommendations

输出：

```text
Do not use detector as automatic proof.
Report uncertainty.
Report subgroup reliability.
Use human review.
Warn about short text and cross-language limitations.
```

---

# 9. 实验设计细化：从数据到结果

## 9.1 Pipeline 总览

```text
1. Load public datasets
2. Standardise label format: human / AI
3. Clean and filter text
4. Construct splits
5. Construct robustness conditions
6. Train / run detectors
7. Evaluate aggregate metrics
8. Evaluate subgroup metrics
9. Analyse failures
10. Test robustness improvement
11. Write practical recommendations
```

## 9.2 Label definition

你的二分类标签可以定义为：

```text
human-written: y = 0
AI-generated: y = 1
```

模型输出：

```text
P(y=1|x) = probability that text x is AI-generated
```

## 9.3 Split strategy

要避免 data leakage。

推荐：

```text
Train / validation / test split should be created before augmentation.
Paraphrased versions of a test example should not appear in training.
If domain or generator labels are available, cross-domain or cross-generator splits should be constructed explicitly.
```

为什么重要？

如果 paraphrased version 泄漏进训练集，模型可能记住内容，而不是学习 detector robustness。

## 9.4 Paraphrase-aware augmentation

简单版本：

```text
Training data:
    original AI text
    paraphrased AI text
    original human text

Goal:
    make the detector less dependent on superficial generation artifacts
```

Consistency-based idea：

同一个 AI 文本和它的 paraphrase 应该输出相似 probability。

源码公式：

```text
$$
\mathcal{L}=\mathcal{L}_{CE}(x,y)+\lambda \cdot \left(p_\theta(x)-p_\theta(\tilde{x})\right)^2
$$
```

其中：

- `$x$` 是原文本；
    
- `$\tilde{x}$` 是 paraphrased version；
    
- `$p_\theta(x)$` 是模型预测 AI 的概率；
    
- `$\lambda$` 控制 consistency loss 强度。
    

工程上，MSc project 可以先做更简单的 augmentation，不一定要实现复杂 contrastive learning。

## 9.5 Error analysis 怎么写才高级？

不要只写：

```text
The model performs worse on Chinese.
```

要写：

```text
The detector shows a larger false positive rate on Chinese human-written text, suggesting that the model may rely partly on language-specific or dataset-specific surface patterns rather than robust evidence of machine generation.
```

再进一步：

```text
Short texts show unstable predictions because they contain fewer stylistic and discourse-level signals. This suggests that detector confidence should be treated with caution when text length is below a certain threshold.
```

这就变成了有解释的 analysis。

---

# 10. 风险管理与 fallback

## 10.1 主要风险表

```text
Risk 1: Zero-shot methods are too computationally expensive.
Mitigation:
    Use a smaller representative subset.
    Replace DetectGPT-style method with perplexity-based baseline.

Risk 2: Full robustness datasets are too large or difficult to process.
Mitigation:
    Use HC3 as the core dataset.
    Use RAID/MGTBench only as optional or subset-based evaluation.

Risk 3: Transformer fine-tuning exceeds GPU memory.
Mitigation:
    Use base-size models.
    Reduce max_length.
    Use gradient accumulation.
    Use early stopping.
    Use traditional baseline as guaranteed result.

Risk 4: Paraphrase generation is expensive.
Mitigation:
    Generate paraphrases only for a controlled subset.
    Use existing paraphrased/adversarial data if available.

Risk 5: Results are not strong.
Mitigation:
    The project is framed as robustness evaluation, so failure cases are valuable results.
    Analyse when and why methods fail rather than treating low scores as project failure.
```

## 10.2 进阶解析：为什么“结果不好”不等于项目失败？

因为你的 research question 不是：

```text
Can I build the best detector?
```

而是：

```text
How robust are representative detectors under realistic conditions?
```

所以如果某些方法表现差，这反而是重要发现：

- 说明 clean benchmark 不足；
    
- 说明 detector 不能直接部署；
    
- 说明某些语言/长度/扰动条件下风险更高；
    
- 支撑你的 ethical discussion。
    

这就是 robustness-evaluation framing 的优势。

---

# 11. 写作策略：如何让 plan 看起来像 Distinction

## 11.1 第一页写作策略

第一页不要塞太多背景。结构建议：

```text
Project Aim
Research Question
Objectives
```

Aim 一段，Research Question 一句加粗，Objectives 用列表。

老师也说 aim 是 paragraph，objectives are a list。

## 11.2 第二页写作策略

用小标题：

```text
Research Design
Data Acquisition and Preparation
Detection Methods
Evaluation and Analysis
Resources and Feasibility
```

不要写成流水账。每段都回答：

```text
What will I do?
Why is this method suitable?
What evidence will it produce?
What is the fallback?
```

## 11.3 第三页写作策略

Gantt chart 旁边可以加 legend：

```text
Core task
Parallel writing/logging
Optional extension
Milestone
Fallback / risk buffer
```

高分关键不是图漂亮，而是图能展示：

- length；
    
- dependency；
    
- milestone；
    
- optional aspects；
    
- non-linear structure；
    
- early detail；
    
- dissertation parallel。
    

## 11.4 后两页写作策略

不要把 ethics / commercial / legal 写成模板话。一定要贴合 AI-generated text detection。

弱写法：

```text
This project has no ethical issues because it uses public datasets.
```

老师明确说“this project has no ethical implications”不是正确答案，因为 ethical statement 关注的是 research area，不只是 narrow project。

强写法：

```text
Although the project uses public datasets and does not recruit participants, the research area has clear ethical implications because AI-generated text detectors may affect trust, academic integrity decisions, and reputational outcomes. False positives are particularly harmful when human-written work is incorrectly labelled as AI-generated.
```

---

# 12. 术语表

## Aim / Research Aim

**研究总目标**。说明项目总体想完成什么。必须 realistic、achievable、unambiguous。

## Research Question

**研究问题**。把 aim 转换成可回答的问题。好的 RQ 应该能通过实验和分析回答。

## Objective

**研究目标 / 具体目标**。必须 outcome-based，即能产生可展示证据。不是 task，也不是 learning activity。

## Activity

**活动 / 工作任务**。为了完成 objective 需要做的具体工作，例如 cleaning dataset、training baseline、running evaluation。

## Milestone

**里程碑**。某个阶段完成的可检查节点。所有 objectives 都应该能对应 milestone，但不是所有 milestones 都是 objectives。

## Methodology

**方法论 / 研究方法**。解释如何完成 objectives，包括 data acquisition、model implementation、evaluation、analysis。

## Gantt Chart

**甘特图**。用横向时间轴展示任务持续时间、重叠关系和依赖关系。老师说它更适合 running project，但 supervisors expect it，所以可用。

## PERT Chart

**项目评估与审查技术图**。更适合展示任务依赖、路径和不确定性。适合表达非线性计划。

## Data Management Plan

**数据管理计划**。说明数据如何获取、存储、处理、命名、记录、保护、归档。所有项目都需要。

## Ethical Statement

**伦理声明**。说明项目及研究领域的伦理风险、控制措施、是否需要 ethics approval。

## Robustness

**鲁棒性**。模型在数据扰动、分布变化、语言变化、文本长度变化、生成器变化下维持性能的能力。

## Cross-domain Generalisation

**跨领域泛化**。模型在训练领域之外的数据上仍能表现良好的能力。

## Cross-generator Generalisation

**跨生成器泛化**。模型在未见过的 LLM 生成文本上仍能检测的能力。

## False Positive

**假阳性**。human text 被误判为 AI-generated。你的项目中这是重要伦理风险。

## False Negative

**假阴性**。AI-generated text 被误判为 human-written。

## Macro-F1

**宏平均 F1**。对每个类别分别算 F1 再平均，适合类别不平衡或关注少数类表现的任务。

## AUROC

**Area Under ROC Curve**。衡量模型在不同阈值下区分正负样本的能力。

## Perplexity

**困惑度**。语言模型对文本预测难度的度量。越低表示模型越容易预测该文本。

---

# 13. 复习速查卡

## Card 1：Coursework 3 是什么？

```text
A 5-page Project Plan Portfolio:
1 page aim/RQ/objectives
1 page methodology
1 page timeline/chart
2 pages data management, ethics, health & safety, environmental, commercial, legal aspects
```

## Card 2：老师最强调什么？

```text
Aim must be realistic and achievable.
Objectives must be outcome-based.
Methodology explains how.
Timeline shows activities, relationships, milestones and risk.
Dissertation writing starts on day one.
```

## Card 3：Objectives 怎么判断好不好？

```text
Bad:
    To learn Python.
    To train a model.
Good:
    To evaluate transformer-based AI text detectors under bilingual and paraphrased conditions.
```

## Card 4：你的项目一句话

```text
This project treats AI-generated text detection as a robustness-evaluation problem rather than simply proposing another detector.
```

## Card 5：你的核心贡献

```text
Compare representative detectors.
Evaluate bilingual, paraphrased, short-text, cross-domain/cross-generator robustness.
Analyse false positives and failure modes.
Test a feasible robustness-oriented improvement.
Provide responsible deployment recommendations.
```

## Card 6：为什么 false positive 重要？

```text
Because a human-written text incorrectly labelled as AI-generated can cause unfair suspicion, reputational harm, or punitive academic/professional decisions.
```

## Card 7：为什么不能只看 accuracy？

```text
Accuracy hides subgroup failures.
A detector may have good overall accuracy but high false positive rate on Chinese, short texts, or paraphrased inputs.
```

## Card 8：Gantt chart 要展示什么？

```text
Task duration
Task dependency
Milestones
Parallel work
Optional extensions
Fallbacks
Dissertation writing across all weeks
```

## Card 9：Data Management Plan 必须写什么？

```text
Data sources
Licences / access dates
Raw vs processed data
Reproducible scripts
Git repository
Experiment logs
Model checkpoints
Privacy handling
Archiving
```

## Card 10：Legal section 可写什么？

```text
Dataset licence
Data protection
Human-written text privacy
Automated decision risk
Liability
Copyright/IP
EU AI Act if future deployment enters EU context
```

---

# 14. 可复制到笔记的简洁版

```markdown
# ECSP6002 CW3 Project Plan Portfolio 核心笔记

CW3 是 5 页 project plan，不是完整 dissertation。结构：
1. Aim / Research Question / Objectives
2. Proposed research and analysis methodology
3. Gantt chart / PERT / flow diagram
4–5. Data management, ethics, health & safety, environmental impact, commercial and legal aspects

## 老师重点
- Aim 必须 realistic, achievable, unambiguous。
- Objectives 必须 outcome-based：能展示、能证明、能收集证据、能写进 dissertation。
- Objectives = what；Methodology = how。
- Activities 应该支撑 objectives；methodology 解释 activities 如何执行；timeline 展示 activities 的时间和关系。
- Project plan 不是预测未来，而是管理风险。
- 不要问 “if something goes wrong”，要问 “when something goes wrong”。
- Dissertation writing 从 day one 开始，并行贯穿整个项目。

## 我的项目 framing
Title:
Robust and Cross-Lingual Evaluation of AI-Generated Text Detection Methods

Core idea:
Rather than proposing another detector evaluated mainly by aggregate benchmark scores, this project treats AI-generated text detection as a robustness-evaluation problem.

Research Question:
How robust are representative AI-generated text detection methods when applied to bilingual, paraphrased, short, and cross-domain text, and which methodological choices provide the most reliable evidence for practical deployment?

## Methods
- Traditional baseline: TF-IDF + Logistic Regression / Linear SVM
- Supervised detector: XLM-R / mDeBERTa fine-tuned for binary classification
- Statistical / zero-shot: perplexity-based baseline or Fast-DetectGPT subset evaluation

## Evaluation
Metrics:
- Accuracy
- Macro-F1
- AUROC
- False Positive Rate
- Confusion matrix
- Subgroup breakdown by language, length, domain, generator
- Robustness degradation

Robustness settings:
- Clean in-domain
- English vs Chinese
- Short vs medium vs long text
- Paraphrased AI text
- Cross-domain / cross-generator

## Ethics
Main risk is false positive: human-written text wrongly labelled as AI-generated.
Detector outputs should be probabilistic and fallible, not automatic proof of misconduct.

## Risk management
Fallbacks:
- If DetectGPT is too expensive → subset or perplexity baseline
- If full RAID/MGTBench is too large → stratified subset
- If transformer training is too expensive → base-size model, shorter max length, early stopping
- If results are weak → analyse failure modes; this still answers robustness RQ
```

---

# 15. 自测题 / 思考题

## Q1：为什么 “To train a transformer model” 不是好的 objective？

因为它描述的是 activity，不是 outcome。好的 objective 应该能产生可展示证据，例如：

```text
To evaluate a supervised multilingual transformer detector under bilingual and paraphrased robustness settings.
```

## Q2：你的项目为什么不能只用 accuracy？

因为 AI detector 的风险不是平均性能，而是错误类型和错误分布。尤其 false positive 会把 human text 误判为 AI text，可能带来不公平指控。还要看 macro-F1、FPR、AUROC、subgroup gap。

## Q3：为什么 dissertation writing 要从第一天开始？

因为 dissertation 是最终被评估的产物。literature、methodology、data description、experiment logs、figures、limitations 都可以边做边写。老师明确强调 dissertation writing 与所有任务并行，而不是最后才开始。

## Q4：为什么你的项目可以拿 Distinction？

因为它不是单纯实现一个 detector，而是：

```text
研究问题明确
方法有代表性
评估条件现实
指标覆盖可靠性和风险
有 fallback
有伦理/法律/商业意识
能产出 failure analysis 和 responsible deployment recommendations
```

---

# 16. 最终提醒：你这份 Project Plan 最容易失分的地方

1. **Aim 太大**：写成 “solve AI-generated text detection”。
    
2. **Objectives 像任务清单**：写成 “read papers / train model / write report”。
    
3. **Methodology 只写模型**：忘了 data acquisition、split、evaluation、analysis、resources。
    
4. **Timeline 太线性**：没有并行任务、fallback、milestone、dissertation writing。
    
5. **Ethics 写成无关紧要**：说 public data 所以 no ethical issues。
    
6. **Commercial 太空泛**：只说 detector 可以用于教育，没有讨论 reliability 和 responsible use。
    
7. **Legal 太模板**：没有联系 dataset licence、data protection、automated decision、liability。
    
8. **没有体现你的区别**：看起来只是另一个 detector project，而不是 robustness-evaluation project。
    

---

## 17. 当前资料限制

这版笔记已经融合了当前可见对话历史、上传的 coursework guide、marking scheme、lecture slides 和课堂转写内容；但我这里没有重新看到你最终 poster 原文件本身，因此项目细节是基于我们之前已经确定的方向：**Robust and Cross-Lingual Evaluation of AI-Generated Text Detection Methods**。补充最终 poster 原件后，可以继续把这份笔记改成逐页对齐 poster 的版本。