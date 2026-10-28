# 面向 Distinction 的社会引导机器人毕业项目升级诊断与研究方案重构

## 当前方案诊断

以下判断以你上传的最终版 poster 为主，其他文件只作为参考文献来判断升级空间。先给结论：**你的题目方向是对的，而且是有潜在 Distinction 气质的题目**；但目前版本更像“一个把 vision / audio 接到 RAG 的机器人原型”，还不像“一个围绕社会引导机制提出可检验学术主张的 MSc 研究”。你距离 Distinction 最大的一步，不是换更大的模型，而是把项目重心从“生成一句 prompt”升级成“机器人如何基于社会线索，在恰当时机、以恰当强度、用恰当形式介入陌生人的面对面互动”。fileciteturn2file0 citeturn3view0turn17view0turn11view4

根据 poster，你当前方案可以概括为：**目标**是开发并评估一个 reactive、多模态、embodied 的 social facilitator，用于帮助公共空间里的陌生人开启对话；**任务设定**是陌生人在共享空间中因为社交顾虑而不主动交流，机器人通过视觉、音频和 UI 观察情境；**方法**是 vision / audio / UI 输入，经由 RAG、knowledge base 和 session memory 生成安全且具体的 ice-breaking prompts，再由 state machine 协调 speech、gestures 和 facial expressions；**预期评估**只是宽泛地说要证明系统能“reduce social friction and increase conversation depth”；**预期贡献**是一个 functional prototype 和一套 multimodal interaction dataset。fileciteturn2file0

你这个方向最强的地方有三点。第一，**问题本身有真实社会价值**，不是为了套 LLM 而硬找场景。CHI 2023 的 stranger-conversation 研究已经证明，技术型 facilitator 的确能帮助陌生人开启更有意义的面对面交流；在那项研究里，机器人条件在“遵从更深层对话问题”方面最强，而机器人和交互屏幕都比海报更能提升人与人之间的 closeness 与 happiness。citeturn3view0turn4view4 第二，**embodiment 方向是对的**。近两年的 HRI 研究反复说明，机器人不一定要做成“第三位强势发言者”；很多有效影响其实来自 subtle 的 gaze、nodding、orientation、proxemics 或 spatial mediation。DIS 2024/2025 和 HRI 2026 都表明，机器人可以通过微妙非语言行为改变人-人关系，而不一定需要复杂长篇对话。citeturn16view0turn16view1turn16view3 fileciteturn3file16 fileciteturn1file12 第三，**你当前架构是可工程落地的**：perception、reasoning、actuation 分层，在 MSc 周期内是可做的。fileciteturn2file0

但它最弱的地方也很明确。第一，**RAG 不是你的研究贡献**。近两年已经有多篇工作把 LLM、多模态感知和 social robot 集成到 multi-party conversational systems 中；2024–2026 的工作已经不只是“能把 LLM 接上机器人”，而是在讨论 turn-taking、balance、multimodal evaluation、latency 和 addressee selection。也就是说，**“我把 RAG 接到机器人上”现在已经太标准了**，更像工程集成，而不是研究创新。citeturn6search2turn11view3turn11view4 第二，**你的 poster 说系统是 reactive，但文献前沿已经把问题推进到 proactive intervention**。Inner Thoughts 和 ProAct 都在强调：真正难的不是“说什么”，而是“何时说、为什么现在说、该不该说、说了会不会打扰”。如果你的 poster 还停留在“detect silence or clothing → generate prompt”，那么创新深度不够。fileciteturn3file10 fileciteturn3file3 第三，**评估设计几乎是空的**。目前没有清晰假设、没有明确 baseline、没有 intervention appropriateness 指标、没有 robustness design，也没有 failure taxonomy。对于 Distinction，这会是最大扣分点。fileciteturn2file0 citeturn11view4turn10search0

我认为你距离 Distinction 最大的差距是这一句：**你现在有一个 system idea，但还没有一个 sufficiently sharp research claim**。如果照当前 poster 直接往前做，你大概率会做出一个还不错的 prototype，达到“能跑、能 demo、做了用户测试”的 Merit 风格结果；但要进 Distinction，你需要让论文的核心知识主张变成：**某种特定的社会介入策略，比 reactive / generic prompting 更能在陌生人互动中提升 initiation、depth、closeness 和 timing appropriateness，同时不过度增加 intrusiveness。** 这是可以被证伪、可以被 baseline 对照、可以被 ablation 支撑的研究命题。fileciteturn2file0 citeturn3view0turn0search6turn0search3

## 创新缺口分析

下面这张表不是在否定你的方向，而是在说明：**现有 poster 哪些地方如果不升级，会显得过于“standard system build”。** 这个判断基于你的 poster 和近年的 HRI / facilitation 文献。fileciteturn2file0 citeturn11view3turn11view4turn17view0

| Current design in my poster | Why it may look standard / weak | What a stronger distinction-level version would look like | Feasibility for MSc project | Risk level |
|---|---|---|---|---|
| RAG + knowledge base + session memory 作为“认知核心” | 现在过于 generic；像工程堆栈，不像研究问题 | 把核心升级为**intervention policy**：研究 when / whom / how to intervene，RAG 只做辅助检索 | 高 | 低 |
| 通过 silence 或 clothing 等上下文触发 prompt | 线索过浅，容易显得 heuristic；appearance-based prompt 还有伦理风险 | 用**多模态 social cues**：pause、mutual orientation、openness、speaking balance、recent rejection history | 高 | 中 |
| state machine 控制 speech/gesture/facial expression | 只是执行层，不是贡献点 | 增加**干预阶梯**：rest → nonverbal cue → light verbal cue → deeper follow-up → withdraw | 高 | 低 |
| “public spaces / international students / elderly care” 都想覆盖 | 场景太散，评估变量爆炸 | 锁定一个主场景：**campus waiting/lounge dyads of unacquainted adults** | 高 | 低 |
| prototype + dataset 作为贡献 | 贡献太宽泛、太像 deliverable | 明确为**可解释的主动介入策略 + 评估协议 + ablation** | 高 | 低 |
| 评估目标是“reduce friction / increase depth” | 不可证伪，也无法严谨比较 | 制定清晰 H1–H4：start rate、depth、balance、closeness、timing appropriateness、intrusiveness | 高 | 低 |
| UI interaction 也写进 objective | 会稀释自然场景研究；陌生人破冰不应依赖额外触控操作 | 除非平台已有触控需求，否则把 UI 从主要贡献里降级 | 高 | 低 |
| reactive system framing | 已落后于近年的 proactive facilitation 叙事 | 升级为**conservative proactive facilitator**，强调 risk-aware 而非激进打断 | 高 | 中 |

如果把这张表再压缩成一句话：**你现在最大的问题不是“技术不够新”，而是“创新点还没有从系统组件层上升到 interaction mechanism 层”。** Distinction 往往奖赏的是机制、比较与论证，不只是功能堆叠。citeturn17view0turn11view2turn10search0

## 最新研究带来的技术升级候选

下面我给你五个升级方向，按**优先级**排序。重点不是越复杂越好，而是**哪个最能在你的题目下，兼顾新意、可实现性和可评估性**。

**候选一：风险感知的主动介入策略**

**相关研究**：Inner Thoughts（CHI 2025）指出，multi-party/social conversation 中单纯做 next-speaker prediction 很难解决 self-selection 问题，关键在于建模“内在动机”和介入时机；ProAct（2026）则把 embodied agent 拆成 fast behavioural loop 和 slower cognitive loop，以平衡实时性和长程社交推理。fileciteturn3file10 citeturn0search3turn0search6

**它解决你当前什么问题**：你的 poster 目前定义了“生成什么 prompt”，但没有研究“现在是不是应该说、该轻推还是退出”。这正是当前方案最薄弱的一环。fileciteturn2file0

**为什么更强**：一旦你把核心贡献改成 intervention policy，你的论文就从“系统集成”变成“机制研究”。这比换一个更大的 VLM 或更 fancy 的 RAG 都更像 Distinction。citeturn17view0turn11view4

**是否适合 MSc**：**非常适合**。你完全可以不用训练新模型，而是用一个可解释的 score-based policy 实现：  
`opportunity = openness + shared-context relevance + conversational lull + balance need - intrusion risk - recent failure`  
当分数过阈值时再决定用 nonverbal cue 还是 verbal opener。

**所需算力 / 数据 / 难度**：低到中。主要成本在系统整合和实验标注，不在 GPU。  

**适合写成的 contribution**：  
“An interpretable, risk-aware intervention policy for embodied social facilitation in stranger conversations.”

**候选二：gaze / orientation grounded 的多模态社会线索融合**

**相关研究**：GazeCoT（CHI 2026）把 gaze 信息引入 MLLM 的 social reasoning；Abbo 等人在 2025/2026 的 Furhat multi-party conversational robot 工作中，也明确依赖 voice direction, speaker diarisation, face tracking 等多模态感知来处理真实多用户对话。citeturn13search0turn12view0turn11view4

**它解决你当前什么问题**：你现在的线索主要是 silence 和表层 topic cue，这太浅。对于陌生人破冰来说，更关键的是：他们**有没有 mutual availability**、是不是都还在闭合状态、是否已经出现 eye-contact-like readiness、是否有人一直主导而另一人沉默。  

**为什么更强**：它让 your intervention 成为“socially grounded”而不是“prompt-first”。这会显著提升论文的学术说服力，因为你研究的是社会信号与介入机制之间的关系。  

**是否适合 MSc**：适合，但要**降级实现**。不要去复现 gaze-informed CoT 模型本身；只把 gaze / head orientation / body openness 当作特征输入到机会评分器里。  

**所需算力 / 数据 / 难度**：中。需要稳定的 head pose / orientation detection，但不需要训练新模型。  

**适合写成的 contribution**：  
“A multimodal social-cue fusion layer for grounding intervention timing in embodied stranger facilitation.”

**候选三：把 generic RAG 改成 structured social memory**

**相关研究**：Inner Thoughts 的核心不是大模型本身，而是 long-term + working memory 的动态检索；ProAct 也依赖长期上下文记忆去触发主动行为。更早的 D-Agree 说明，**实时 facilitation 往往依赖 tractable structure 和有限规则，而不是开放式生成**；该系统甚至使用了约 200 条 facilitation rules 来保持实时可控。fileciteturn3file10 fileciteturn1file17

**它解决你当前什么问题**：你 poster 里把 RAG 说得太大了，但陌生人破冰并不需要开放世界知识。真正需要的是：  
- 谁已经说过话，谁还没说  
- 哪个 prompt 刚刚被忽视了  
- 当前 pair 是否已经进入自然对话  
- 当前更适合 light prompt 还是 deeper follow-up  
- 哪些 topic 属于 safe / taboo / too personal  

**为什么更强**：因为它更贴合 facilitation task 的本质，也更安全、更快、更可解释。  

**是否适合 MSc**：**非常适合**。这是我最建议你做的“RAG 升级方式”：不是更大的 retrieval，而是更好的 memory schema。  

**所需算力 / 数据 / 难度**：低。更多是 data structure 设计，不是模型训练。  

**适合写成的 contribution**：  
“A structured session memory for socially grounded, context-aware prompt selection in robot-mediated ice-breaking.”

**候选四：非语言中介层与渐进式干预阶梯**

**相关研究**：DIS 2024 的 group ideation 研究表明，一个 simple gestural robot 能显著增加 consensus-building；该工作里 with-robot 组 80% 时间处于 consensus-building，而 without-robot 组是 55%。DIS 2025 进一步表明，affirmative nodding 会提升说话者被感知到的 extraversion / agreeableness / openness 和 supportive role，但不会显著改变其“内容本身”的评价。MirrorBot（HRI 2026）则说明，仅靠 spatial / mirror-based mediation 就能触发 low-stakes icebreaking；同时它也显示，机器人必须有“withdraw / rest”逻辑来降低 intrusiveness。citeturn16view0turn16view3 fileciteturn3file16 fileciteturn1file10 fileciteturn1file12

**它解决你当前什么问题**：你的 poster 里 speech 是主角，gesture 像附属物。但在 stranger facilitation 里，**speech 太早、太猛，往往就是打扰**。  

**为什么更强**：因为它能把机器人定位成“facilitator”而不是“judge”或“third participant”。这和最新文献非常一致。并且 DIS 2025 还提醒：支持性 nodding 容易让机器人被当作 in-group supportive presence，而否定性 gesture 更容易被看成 outsider。fileciteturn3file15

**是否适合 MSc**：适合，而且很划算。你不需要生成复杂 motion，只要几个 carefully designed gestures：看向双方、轻微 nod、邀请式手势、rest/withdraw posture。  

**所需算力 / 数据 / 难度**：低。主要是 interaction design，而不是 ML。  

**适合写成的 contribution**：  
“A graded embodied intervention ladder for low-intrusion social mediation.”

**候选五：会话平衡控制与 dominance-aware facilitation**

**相关研究**：Grassi 等人 2025 研究了如何用简单、可落地的政策在 off-the-shelf robot 上控制 multi-party conversation dynamics；他们将 dominance 建模为 speaking time 和 number of words 的加权和，并通过 selective addressing 设计平衡或不平衡会话的控制策略。与此同时，2025–2026 的 multi-party social robot 评估工作也显示，整合 LLM 和 multimodal system 的收益之一，就是 interaction balance 改善，但系统仍面临 latency 与 recognition 挑战。citeturn11view2turn11view3turn11view4

**它解决你当前什么问题**：你的 poster 只讲“让陌生人开始说话”，但没讲“开始之后如何避免一个人 monopolise conversation”。这会让论文停留在 initiation，而不是 facilitation。  

**为什么更强**：因为它把研究问题从“能不能开口”推进到“能不能维持 equitable conversation”。这是很好的 Distinction 升级。  

**是否适合 MSc**：中等适合。pair setting 也能做 dominance-aware facilitation，但若扩展到三人以上就会迅速增加复杂度。  

**所需算力 / 数据 / 难度**：低到中。关键是 diarisation / turn log 和 speaking balance calculation。  

**适合写成的 contribution**：  
“A dominance-aware social facilitation policy for sustaining balanced stranger conversation.”

这里我明确告诉你：**最不值得做的升级**，是把项目改造成“训练一个小型多模态基础模型”或者“复现 ProAct 的动作生成 pipeline”。你上传的 small multimodal LLM poster 本身就说明了那条路涉及 image / speech / motion 的离散 tokenisation、扩词表、连续预训练和 SFT；而 ProAct 涉及流式 motion generation、ControlNet 条件注入和 physical robot motion modelling。Inner Thoughts 连 proactivity threshold 的 data-driven / RL 学习都还放在 future work。对 MSc 来说，这些太重，也偏离你真正的问题。fileciteturn1file5 fileciteturn3file3 fileciteturn1file9

## 推荐的核心贡献与备选方案

**我最推荐你作为主创新点的方向是：**  
**一个基于多模态社会线索与结构化 session memory 的、风险感知主动介入策略，用于 embodied stranger facilitation。**

换成更具体的研究语言，就是：**研究机器人如何判断何时、是否、以及以什么强度介入陌生人的面对面互动，并验证这种介入是否确实改善人-人社会结果。** 这比“一个 RAG 机器人会说更自然的话”强得多。fileciteturn2file0 citeturn0search6turn0search3turn17view0

**这个创新点和已有工作的区别**在于：  
它不是简单复现 CHI 2023 的“robot vs screen vs poster”结果，因为那篇工作已经相当充分地说明了交互式 facilitator 的价值；你真正要做的，是追问**哪种机器人介入机制最有效**。它也不是直接复现 Inner Thoughts，因为后者主要是 text-based conversational AI，不是 public-space embodied social mediation。它更不是 ProAct 的 motion-generation 复现，因为你的科学问题不是“怎么生成最丝滑的 motion stream”，而是“怎样的 intervention policy 能改善陌生人的真实互动”。citeturn3view0turn4view4turn0search6turn0search3

**它为什么不是简单复现已有方法**：  
因为你的核心对象是**陌生人之间的人-人连接**，不是 human-AI task chat；你的关键变量是**social appropriateness、intrusiveness、initiation、depth、closeness**，不是单纯 response quality；你的技术核心是**intervention mechanism**，不是 prompt 生成本身。你会把多模态感知、结构化记忆、embodied cue 和 conservative proactive policy 结合在一个特定任务——public-space stranger ice-breaking——中，并用 human-human outcome 指标来证明其价值。现有文献正好是分散地覆盖这些点，而你可以把它们收束成一个清晰的 MSc 问题。citeturn17view0turn11view4turn10search0

**它如何体现 research contribution**：  
你的 dissertation 可以明确声称三条贡献：  
第一，提出一个**可解释的 intervention scoring policy**，而不是黑箱式“LLM 自己决定”；  
第二，提出一个**structured social memory schema**，说明哪些 interaction state 真正对 stranger facilitation 有用；  
第三，提出一个**严谨的评估协议**，把 initiation、depth、balance、timing appropriateness 和 intrusiveness 分开测。fileciteturn3file10 fileciteturn1file17 citeturn10search0turn11view4

**它如何在 dissertation 中被评估和证明**：  
不是靠“我的结果更高”一句话，而是靠：  
- reactive baseline vs proposed policy 的用户实验；  
- no-memory / no-visual-cues / speech-only 的 ablation；  
- intervention appropriateness 的人工标注；  
- 失败案例分析；  
- 客观 behavioural metrics 与主观问卷的交叉验证。  
这样写出来，论文会非常像一个成熟的 MSc research project，而不是系统展示。citeturn4view2turn11view2turn3file9

**它的最小可行实现版本**：  
只做**pair-based** scene；只用**pause + speaking balance + head orientation** 三类 cue；只做**3–4 类安全 prompt**；只做**2–3 个非语言动作**（look, nod, invite, withdraw）；用一个**可解释 score** 决定是否介入。  

**它的增强版本**：  
加入 recent rejection memory、轻度 personalised threshold、gesture-first escalation、deeper prompt progression、失败恢复逻辑，以及一个小规模的 cross-condition robustness test。  

作为备选，你还可以按不同风险级别这样选：

| 风险级别 | 备选创新点 | 优点 | 风险 | 需要的数据 | 需要的模型 / 算法 | 最终可写成的 dissertation contribution |
|---|---|---|---|---|---|---|
| Low-risk | **Embodied intervention ladder**：先 nonverbal cue，再 light verbal cue，再 deeper follow-up | 最容易实现；最容易做语音/手势 ablation；很贴合 stranger setting | 创新性中等，必须靠评估设计拉高档次 | 15–20 对 dyads 也可做出像样结果 | rule-based trigger + curated prompt retrieval + scripted gestures | “Subtle embodied mediation improves the quality of first-contact conversations” |
| Medium-risk | **Dominance-aware facilitator**：开口后继续调节谁说得更多、谁被忽略 | 更有研究味；能超出“只负责开场”的浅层贡献 | 需要更稳定的 turn logging 与 balance metric | 20–30 对 dyads 或少量 triads | speaking-time / word-count dominance estimation + selective addressing | “A robot can improve not only conversation initiation but also participation balance” |
| High-risk | **Gaze-informed adaptive planner**：把 gaze / openness 送入 learned classifier 或小型 reasoning module 来自动学 threshold | 研究前沿感强；更像新方法 | 数据、调参、延迟、稳定性风险都高；容易拖垮 MSc 周期 | 你自己的 annotated facilitation dataset 需要更大 | cue classifier / learned policy / small MLLM reasoning | “Learning social intervention timing from multimodal cues in embodied facilitation” |

如果你的目标是 Distinction 而不是冒险，我会建议：**主线走“推荐主创新点”，不要直接跳 high-risk。**

## 方法学升级

这里我给你的不是“把模型换掉”的建议，而是**把 methodology 真正改造成 dissertation-friendly 的版本**。

**Dataset / environment setup**  
我建议你把主场景锁定为：**campus waiting-room-like 或 lounge-like 的 semi-public setting，参与者为彼此不认识的 dyads**。这和你的 poster 一致，也和最接近你题目的文献一致：CHI 2023 用的是 waiting-room stranger conversation；MirrorBot 2026 也是 waiting-room stranger encounter。这样你既保留“public-space ice-breaking”的主题，又避免真实开放公共空间带来的不可控性与伦理复杂度。样本量上，MSc 现实目标可以是：pilot 8–10 对 dyads，main study 24–30 对 dyads，另加一个小规模 robustness subset。fileciteturn2file0 citeturn3view0turn4view3 fileciteturn1file12

**Baseline methods**  
我建议至少设三个主条件：  
- **Baseline A：No facilitator / minimal poster cue**  
- **Baseline B：Poster-style reactive robot** —— 这就是你当前 poster 的自然 baseline：检测 silence 或显著 topic cue，然后给出一个 generic but contextualised prompt  
- **Baseline C：Proposed opportunity-aware robot** —— 用多模态 cue + structured memory + intervention ladder  
如果人手和时间允许，再加一个小型附加比较：  
- **Ablation D：Proposed without structured memory**  
- **Ablation E：Proposed speech-only**  
这样你就不是在重复“机器人有没有用”，而是在研究**什么样的机器人介入更有用**。fileciteturn2file0 citeturn3view0turn4view4

**Proposed method**  
我建议你把系统重构为两个 loop，而不是一个笼统的 RAG core。  
- **Fast loop**：每 200–500ms 更新 VAD、pause length、head orientation、face presence、speaking-time balance、是否已经进入自然人-人对话。  
- **Slow loop**：每 2–3s 更新 structured session memory，计算 intervention opportunity score，并在“rest / nonverbal / light verbal / deeper follow-up / withdraw”之间选择动作。  
- **Memory schema** 不要是抽象 RAG，而是显式字段：  
  - current state: no-talk / opening / engaged / stalled / dominated  
  - participants availability  
  - recent prompts attempted / ignored / accepted  
  - safe shared context tags  
  - current balance score  
  - last successful conversational bridge  
- **Generation policy** 也不要 fully open-ended。用**curated prompt library + LLM paraphrasing** 最合适。也就是说，先检索一条合适的 prompt type，再让模型把它说自然，而不是让模型自由编一切。D-Agree 这类工作说明，实时 facilitation 往往正是通过 tractable structure 和有限规则实现的。fileciteturn1file17 fileciteturn3file10 fileciteturn3file3

**Ablation study**  
至少做四个 ablation：  
- 去掉 **structured memory**：检验 memory 是否真的帮助 timing 和 continuity  
- 去掉 **visual social cues**：只保留 audio / pause，检验 gaze-orientation cue 的价值  
- 去掉 **embodiment**：speech-only 或最小动作，检验 gesture 是否真的提升效果  
- 去掉 **risk penalty**：检验 conservative policy 是否减少 intrusiveness  
这一步非常关键，因为 Distinction 项目不能只是“我用了新方法，所以更好”；必须说明**哪一个组件为什么有效**。citeturn3view0turn11view2turn10search0

**Evaluation metrics**  
你需要把指标拆成三类。  
第一类是**behavioural outcome**：  
- conversation start success  
- time to first human-human turn  
- prompt uptake rate  
- total human-human speaking time after first robot intervention  
- depth score / personal-topic proportion  
- speaking balance / dominance error  

第二类是**subjective outcome**：  
- awkwardness  
- ease of starting  
- vulnerability  
- interpersonal closeness  
- happiness / enjoyment  
- timing appropriateness  
- social presence  
- perceived support  
- willingness to use again  

第三类是**system integrity metric**：  
- false-positive intervention rate  
- ignored-prompt rate  
- latency to response  
- sensory failure cases  
这些指标不是凭空想的。CHI 2023 给了你 awkwardness / vulnerability / closeness / happiness / depth 的成熟评估思路；Inner Thoughts 提供了 initiative、engagement、timing appropriateness 等 conversational quality 维度；ProAct 则给了 motion naturalness、active agency、perceived presence 的 embodied evaluation 参考。citeturn4view2turn4view4 fileciteturn3file9 fileciteturn3file1

**Error analysis**  
你的 dissertation 一定要有 failure taxonomy。我建议至少编码这几类失败：  
- wrong moment：人还没准备好，机器人过早说话  
- wrong target：机器人选错了该看谁 / 对谁说  
- wrong content：prompt 太 personal、太 generic、太重复  
- wrong intensity：应该用 nod，却上来就 verbal interruption  
- recovery failure：第一次没成功后没有 graceful withdraw  
这会让你的论文看起来非常成熟，因为 reviewer 最喜欢看到你知道系统为什么失败，而不是只秀一个均值。fileciteturn1file10 citeturn12view1turn11view4

**Robustness / generalisation test**  
我建议做一个小而精的 generalisation design，不要做太大。比如：  
- quiet waiting room vs mildly noisy lounge  
- participants facing roughly toward each other vs side-by-side  
- both initially on phones vs one open / one closed  
如果你特别想加入 international student 维度，可以做**exploratory analysis**，而不是主假设。因为 DIS 2025 已明确提醒：social cues 的解释具有文化特异性。这个点可以写进 discussion，但不建议做成论文主线，否则样本要求会迅速增加。fileciteturn1file2

**Statistical and qualitative analysis**  
定量上，报告显著性之前，先报告 effect sizes 和 confidence intervals。你的 N 不会很大，所以 honest reporting 非常重要。主观量表可用 non-parametric test 或 mixed-effects model；depth / appropriateness 的人工编码要做 inter-rater reliability。CHI 2023 的 depth coding 就采用了多名 coder 和一致性检查，这种做法很适合借鉴。定性上，用 short semi-structured interview + thematic analysis，提炼“robot was supportive / awkward / judge-like / helpful / intrusive”等主题。citeturn4view2 fileciteturn3file16 fileciteturn1file12

**Expected results and how to interpret them**  
你真正理想的结果，不应只是“proposed > baseline”。更成熟的解释会是：  
- proposed system **更容易触发开场与持续互动**；  
- structured memory **提升 prompt continuity，降低重复和失误**；  
- embodiment **主要改善 socio-emotional channel，而不是 idea quality 本身**；  
- conservative policy **降低 intrusiveness**；  
- deeper interaction 可能会**同时提高 vulnerability 与 happiness**。  
最后这一点非常重要：CHI 2023 发现，机器人促进 meaningful conversation 的同时，也可能让人感觉更 vulnerable，但这未必是坏事；它可能正是更真实 social connection 的代价。因此你不能把“降低 awkwardness”当成唯一目标，而要看 meaningfulness 与 comfort 之间的平衡。citeturn4view4 fileciteturn3file15

## 评估设计

这部分我建议你单独强化，因为 open-ended embodied social facilitation **目前没有成熟统一 benchmark**。Frontiers 2026 直接指出，open-ended multi-party robot interaction 缺少 widely adopted baselines 和 standardised evaluation protocols；Inner Thoughts 也强调 proactivity 评估难、资源密集；而 2026 的 MPCEval 则说明，单一分数会掩盖 speaker modelling、content quality 和 speaker-content consistency 的差异。对你来说，这反而是机会：**你可以把 evaluation protocol 本身做成贡献的一部分。** citeturn11view4turn10search0turn10search1 fileciteturn1file9

| Evaluation setting | 你应该怎么做 | 它为什么能支撑研究问题 |
|---|---|---|
| Clean test setting | 安静、半控制 waiting-room / lounge；固定 seating；每组只经历一个主条件 | 让你先回答最核心问题：机器人介入策略是否真的改善 stranger ice-breaking |
| Robustness setting | 加入轻度环境噪声、phone distraction、侧向 seating | 检验系统是否只在理想环境下有效，还是对真实公共环境有一定韧性 |
| Cross-condition setting | 同一系统在 waiting-like 与 networking-like 布局各测一小批 | 支撑“不是只对一个房间布局有效”的 generalisation claim |
| Ablation setting | no-memory / no-visual-cues / speech-only / no-risk-penalty | 证明 improvement 来自哪个组件，而不是“整体黑箱” |
| Failure case analysis | 明确标注 wrong timing / wrong content / wrong target / recovery failure | 支撑你关于 system limitation 的论证，提升论文可信度 |
| Human-interpretable analysis | 对代表性片段画 intervention timeline：cue score、memory state、action selected、human response | 证明机器人不是随机说话，而是按可解释规则行动 |
| Quantitative metrics | start success、time-to-first-talk、depth、balance、closeness、awkwardness、timing appropriateness、intrusiveness | 这些指标分别对应“是否开口、是否持续、是否平衡、是否舒服、是否合适” |
| Qualitative examples | 成功引导、被忽视、过早打断、gesture 帮助、withdraw 成功 | 让 dissertation 不只停留在表格均值，而是展示 interaction mechanism |

对 quantitative 部分，我建议你至少设置两套主要 outcome。**Primary outcomes**：conversation start success、depth、timing appropriateness、interpersonal closeness。**Secondary outcomes**：balance、happiness、vulnerability、intrusiveness、future use intention。这样写会很像成熟研究设计，而不是把所有指标平铺。citeturn4view4 fileciteturn3file9

对 qualitative 部分，我建议你挑 4–6 个最有代表性的片段写成“mini case analysis”，每个片段回答三个问题：  
- 机器人为什么在此刻介入？  
- 人类如何响应？  
- 这一响应支持还是反驳你的机制假设？  
这种写法比单纯附几段 transcript 更有说服力。

## 可行性与风险控制

如果你按我推荐的主线来做，**这个升级方案是可行的**。原因很简单：你不是去训练 multimodal foundation model，也不是去做 ProAct 那种流式 gesture generation；你是在已有机器人平台上做一个**可解释的、保守的、以 interaction policy 为中心的系统**。这条路线把最大的瓶颈从 GPU 变成了 integration、recruitment 和 annotation——而后者更适合 MSc。fileciteturn1file5 fileciteturn3file3 citeturn16view0turn16view3

| Week | 重点任务 | 关键产出 |
|---|---|---|
| 1 | 冻结研究问题、场景、条件与假设；起草 ethics | 明确 RQ / H1–H4 / study protocol |
| 2 | 精读核心文献；定义 prompt taxonomy 与 safety rules | facilitation design spec |
| 3 | 完成 baseline reactive pipeline | 可运行 Baseline B |
| 4 | 接入 ASR / VAD / speaking-time logging / basic head orientation | perception log pipeline |
| 5 | 设计 structured session memory | memory schema + logging |
| 6 | 实现 intervention score 与 state transitions | first proposed policy |
| 7 | 接入 gestures / gaze / withdraw state | embodied ladder v1 |
| 8 | pilot study（8–10 对 dyads） | threshold tuning + obvious bug list |
| 9 | 冻结主实验版本；准备 annotation protocol | stable experimental build |
| 10 | 主实验第一轮 | collected sessions part 1 |
| 11 | 主实验第二轮 + interviews | full dataset |
| 12 | annotation、statistics、ablation 与 robustness run | results + failure taxonomy |
| 13 | dissertation writing、figures、discussion | final write-up |

**Minimal viable version**  
如果时间很紧，你至少要保住以下内容：  
- dyad setting  
- 3 个主条件中的 2–3 个  
- speaking-time / pause / basic orientation cue  
- structured memory 的最小字段版本  
- 1 次 main study + 1 次 ablation  
- 主观 + 客观双指标  
这已经足以写成一篇很扎实的 MSc dissertation。

**Full distinction version**  
在 MVP 基础上，再加：  
- gesture-first escalation  
- recent rejection memory  
- robustness subset  
- intervention appropriateness annotation  
- detailed failure taxonomy  
- stronger discussion on vulnerability vs meaningfulness trade-off  

**Potential technical risks**  
最大的技术风险不是“模型不够大”，而是：  
- perception 不稳，特别是 orientation / addressee 判断  
- ASR / diarisation 造成 timing 错位  
- latency 让 prompt 不合时宜  
- prompt 过于 generic 或过于 personal  
- gesture 与 speech 同步不好，导致 robot role 变怪  
这些风险在最近 multi-party social robot 文献里都是真实存在的，尤其是 recognition errors 和 response latency。citeturn12view1turn11view4

**Fallback plan**  
如果 advanced method 失败，我建议你这样退而不崩：  
- perception 不稳 → 降级成 audio + pause + speaking balance 为主，visual cues 仅辅助  
- LLM 生成不稳 → 改成 curated prompt library + paraphrase，而不是自由生成  
- timing 判断不稳 → 保留 conservative threshold，减少说话次数  
- autonomy 不稳 → 在 pilot 阶段允许 WoZ-assisted trigger，主实验只保留已自动化的部分  
MirrorBot 的 field/lab design 就使用了 WoZ 来探索接受度和互动机制；对 MSc 来说，这是合理的设计 fallback，只要你在 dissertation 里诚实说明边界。fileciteturn1file10

**Compute requirement**  
这条路线的 compute requirement 很温和：  
- 如果你用 API 或已部署模型，基本不需要 GPU 训练；  
- 如果本地运行，小型或中等规模 inference 即可；  
- 真正重的是 annotation、video review、统计分析。  
你应该把时间预算更多放在 pilot + study protocol + annotation quality 上，而不是过度投入模型工程。

**What to do if the advanced method fails**  
如果最后 advanced cue fusion 效果不稳定，不要硬撑一个“learning-based social policy”。直接把论文定位成：  
**“A conservative, interpretable social facilitation policy for embodied stranger interaction, with mixed-method evaluation.”**  
只要实验和分析扎实，这条线依然有 Distinction 潜力。

## Project plan 英文草稿与最终建议

**Project Aim**  
This project aims to design and evaluate an embodied social robot that facilitates first-contact conversations between unacquainted people in semi-public waiting spaces. Rather than positioning the robot as a full conversational partner, the project treats it as a low-intrusion social mediator that identifies appropriate moments to intervene and delivers context-aware verbal and non-verbal cues to help strangers begin and sustain meaningful face-to-face interaction.

**Research Question**  
How can an embodied social robot use multimodal social cues and short-term interaction memory to decide when and how to intervene in stranger conversations without becoming intrusive?  
A secondary question is which components of the intervention policy contribute most to improved conversational outcomes, including initiation, depth, participation balance, and perceived appropriateness.

**Objectives**  
The project has four objectives. First, to develop a multimodal perception pipeline that captures conversational pauses, speaking balance, and basic visual availability cues. Second, to implement a structured session memory that represents the local social state of an interaction more explicitly than a generic retrieval pipeline. Third, to design a conservative intervention policy that selects between non-verbal prompting, light verbal ice-breakers, deeper follow-up prompts, and withdrawal. Fourth, to evaluate the proposed system against reactive baselines through a controlled user study and targeted ablations.

**Methodology Summary**  
The study will focus on dyadic interactions between unacquainted participants in a waiting-room-like campus setting. The system will be implemented on an existing social robot platform using a modular architecture with a fast perception-and-behaviour loop and a slower decision loop. The proposed method will combine multimodal social cue estimation, structured session memory, and a scored intervention policy. A reactive robot baseline and a no-facilitator baseline will be implemented for comparison. Additional ablations will remove memory, visual cues, or embodied behaviours to examine the contribution of each component.

**Main Contribution**  
The main contribution of the project will be an interpretable, risk-aware intervention policy for embodied social facilitation. Instead of claiming novelty through a general-purpose LLM or retrieval framework, the project will contribute a task-specific account of how a robot can decide whether to intervene, when to intervene, and how strongly to intervene in a socially appropriate manner. A second contribution will be a reproducible evaluation protocol for robot-mediated stranger interaction, including behavioural, subjective, and error-analysis measures.

**Evaluation Strategy**  
Evaluation will combine objective and subjective measures. Objective measures will include conversation start success, time to first human-human exchange, prompt uptake rate, speaking balance, and manually coded conversation depth. Subjective measures will include awkwardness, ease of starting, interpersonal closeness, happiness, perceived timing appropriateness, social presence, and willingness to use the system again. The experimental design will include a clean controlled setting, a small robustness subset, and component ablations. Qualitative interviews and failure case analyses will complement the quantitative results.

**Risk and Fallback Plan**  
The main risks concern perception reliability, response latency, and overly intrusive intervention behaviour. To reduce these risks, the project will use conservative intervention thresholds, a curated prompt library, and a small set of pre-defined embodied behaviours rather than unconstrained generation. If multimodal perception proves unreliable, the system will be simplified to audio-led cues and speaking-balance estimates. If fully autonomous triggering is unstable, Wizard-of-Oz support may be used at the pilot stage to validate the interaction design before freezing the final experimental system. The project scope will remain centred on a semi-controlled campus setting to preserve feasibility within the MSc timeframe.

你当前 poster **最应该修改的 3 个地方**是：  
- **把“RAG cognitive framework”从主创新点降级为实现组件**，把主问题明确改写成 **when / whether / how to intervene**。  
- **把场景收窄**：不要同时写 public spaces、international students、elderly care。请选择一个主环境，最建议 campus waiting / lounge stranger dyads。  
- **把评估写实**：明确条件、假设、metrics、ablation、failure analysis；否则再好的系统也像 demo。fileciteturn2file0 citeturn3view0turn11view4turn10search0

你**最应该加入的 1 个新技术方向**是：  
- **风险感知的主动介入策略**，由多模态 social cues + structured memory 驱动，并用 embodied intervention ladder 实现。fileciteturn3file10 citeturn0search3turn11view2

你**最应该避免的 1 个方向**是：  
- **训练或复现一个端到端 small multimodal foundation model / full embodied motion generation system**。这会把你的论文从“社会中介机制研究”拖成“模型与系统工程泥潭”。fileciteturn1file5 fileciteturn3file3

如果目标是 Distinction，我建议你把最终 project **定位成一句话**：  
**An embodied social mediation project that studies how a robot can use multimodal social cues and structured interaction memory to intervene at the right moment, with the right intensity, to help strangers start meaningful face-to-face conversations.**