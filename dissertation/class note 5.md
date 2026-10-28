
# Ethics & Diversity for ECS Students

## 最终修正版补充笔记：基于正式 PPT 校准

---

## 1. 讲座基本信息修正

正式 PPT 标题为：

**Ethics & Diversity for ECS PhD Students**

讲者为：

**Dr Samantha Pearman-Kanza**  
**Deputy Chair of FEPS Ethics Committee**

时间为：

**11th March 2026**

这里需要注意，字幕转写中把姓名识别成了类似 “Samantha Peyman”，但正式 PPT 上应以 **Samantha Pearman-Kanza** 为准。

---

# 2. 为什么要关心 Research Ethics？

PPT 第 2 页的问题是：

```text
Why should we care about this?
What are the consequences of not doing so?
```

也就是这节课不是单纯介绍表格怎么填，而是从两个角度出发：

1. **为什么研究伦理重要？**
    
2. **如果不遵守伦理要求，会有什么后果？**
    

正式 slides 里还用了一个漫画式提示，暗示 ethics application 常见问题不是“表格太麻烦”，而是研究者经常遗漏、误填或低估伦理风险。

所以这节课的底层逻辑是：

```text
Ethics is not administrative decoration.
Ethics is part of research design, research integrity, data governance, and institutional accountability.
```

中文可以理解为：

**伦理不是研究完成后补上的行政手续，而是研究设计本身的一部分。**

---

# 3. Unethical Research Examples：两个反面案例

PPT 第 3 页正式列出两个 unethical research examples。

---

## 3.1 Tuskegee syphilis experiment

正式 PPT 信息：

```text
Tuskegee syphilis experiment
- US, 1932–1972
- Deliberate spread of misinformation
- Ultimately led to deaths due to participants misbelieving they were receiving free treatment
```

这个案例的核心不是“研究失败”，而是**研究设计本身违反伦理**。

它涉及：

- **Misinformation（误导信息）**
    
- **Lack of informed consent（缺乏知情同意）**
    
- **Harm to participants（伤害参与者）**
    
- **Exploitation（剥削弱势群体）**
    
- **Violation of research integrity（违反研究诚信）**
    

它说明：

```text
Participants must not be misled about what is happening to them.
Researchers must not hide information that affects participants’ safety, rights, or consent.
```

---

## 3.2 Facebook Secret “mood” study

正式 PPT 信息：

```text
Facebook Secret ‘mood’ study
- Web-based, 2012
- Manipulated users’ news feeds without consent or knowledge
- Tested whether exposure to positive or negative posts influenced emotional tone of users’ own posts
```

这个案例非常适合 AI / HCI / recommender system / social computing 项目理解。

问题不在于它造成了明显身体伤害，而在于：

- 用户不知道自己被纳入实验
    
- 用户没有 consent
    
- 研究者操控了用户信息环境
    
- 研究可能影响用户情绪
    
- 数据是在用户不知情的情况下被用于研究
    

这说明：

```text
No physical harm does not mean no ethical issue.
```

对于 AI 项目尤其重要。比如：

- chatbot A/B testing
    
- recommender system evaluation
    
- user study
    
- online behavioural analysis
    
- emotion detection
    
- social media text analysis
    

都可能存在类似问题。

---

# 4. ED&I：为什么研究需要 Equality, Diversity and Inclusion？

PPT 第 4 页标题是：

```text
Why we need ED&I in our research
```

ED&I 指：

**Equality, Diversity and Inclusion**  
中文可译为：**平等、多样性与包容性**

---

## 4.1 Biased systems

PPT 明确列出：

```text
Biased systems
- Facial recognition systems only trained on certain demographics
- Medical trials ran with certain demographics
- Biased data leads to biased systems and a further perpetuation of this bias
- YOUR RESEARCH IS ONLY AS GOOD AS THE DATA YOU FEED IT
```

这页是整个 AI ethics 的重点。

核心逻辑是：

```text
Biased researchers / participants / datasets
        ↓
Biased data
        ↓
Biased systems
        ↓
Biased decisions
        ↓
Bias is further perpetuated
```

---

## 4.2 Facial recognition example

如果 facial recognition system 主要训练在某些 demographic groups 上，它可能只对这些群体表现好。

例如：

- white faces
    
- male faces
    
- young people
    
- high-quality frontal images
    

但对其他群体可能表现差：

- darker skin tones
    
- women
    
- elderly people
    
- children
    
- non-Western populations
    
- low-light images
    

所以模型的 **overall accuracy** 不能代表它公平。

在 AI 项目中应该关注：

```text
Overall performance is not enough.
Subgroup performance matters.
```

---

## 4.3 Medical trials example

如果医学试验只在某些 demographic groups 上进行，结果可能只对这些群体有效。

这对应到 AI-generated text detection 项目：

如果 detector 主要在英文、长文本、标准语法、clean benchmark 上训练，那么它可能对以下场景不可靠：

- Chinese text
    
- bilingual text
    
- non-native English writing
    
- short text
    
- paraphrased text
    
- translated text
    
- unseen generator output
    
- mixed human-AI writing
    

因此你项目里强调 robustness evaluation 是合理的，而且和 ethics 有内在联系。

---

## 4.4 PPT 中最关键句子

PPT 第 4 页底部有一句非常重要：

```text
If we don’t diversify our researchers and research participants,
how can we ever expect to collect or produce diverse data?
```

这句话可以用于理解：

**研究者群体、参与者群体和数据来源的单一性，会直接限制研究结果的外部有效性和公平性。**

---

# 5. Potential for subversion / leaks：数据泄露与误用风险

PPT 第 5 页标题是：

```text
Potential for subversion / leaks
```

正式列出三个问题：

```text
Always think about the risks linked to the data you collect and share.
What are you accidentally giving away?
Could this be misused?
```

这页配了 Strava military base 的例子。

---

## 5.1 Strava example

Strava 是 running / fitness app，会记录跑步路线和位置。

问题是：

- 一些军事人员使用 Strava 跑步
    
- 他们的运动路线被上传
    
- 多个轨迹叠加后暴露了军事基地位置
    
- 这不是研究者或用户故意泄露
    
- 但数据组合产生了敏感推断
    

这个例子的关键不是“不要用跑步 app”，而是：

```text
Data can reveal more than researchers or users intended.
```

---

## 5.2 对 AI / Data Mining 项目的启示

很多学生以为，只要不收集姓名和邮箱，就没有 personal data。

这是错误的。

即使没有直接身份字段，也可能通过以下信息重新识别：

- location
    
- timestamp
    
- rare behaviour
    
- writing style
    
- metadata
    
- IP address
    
- device information
    
- linked datasets
    
- exact demographic combinations
    

所以你在 Data Management Plan 中最好不要写：

```text
No names will be collected, so the data is anonymous.
```

更严谨的写法是：

```text
Although direct identifiers such as names and email addresses will not be collected, the project will still consider whether indirect identifiers, metadata, or dataset linkage could create re-identification risks.
```

---

# 6. Ethics is complex for humans：机器伦理决策的困难

PPT 第 6 页标题是：

```text
Ethics is complex for humans,
Should we embed these decisions in machines?
```

它引用了电影 _I, Robot_ 的例子：

```text
“I was the logical choice. It calculated that I had a 45% chance of survival.
Sarah only had an 11% chance. That was somebody’s baby.
11% is more than enough. A human being would’ve known that.”
```

---

## 6.1 这个例子的意义

机器人根据 survival probability 做出了“逻辑上最优”的选择：

- adult male survival chance = 45%
    
- child survival chance = 11%
    

但人类角色认为：

```text
A human being would’ve known that.
```

这说明：

**伦理判断不能简单等价于概率最大化。**

---

## 6.2 对 AI 系统设计的启示

AI 系统经常倾向于优化 measurable objective，例如：

- accuracy
    
- reward
    
- survival probability
    
- click-through rate
    
- efficiency
    
- ranking score
    
- loss function
    

但 ethical decision 可能涉及：

- dignity
    
- rights
    
- fairness
    
- vulnerability
    
- social expectation
    
- explainability
    
- accountability
    
- uncertainty
    
- harm distribution
    

因此，在 AI 项目中不能只说：

```text
The model achieved high accuracy.
```

还要问：

```text
Who benefits?
Who is harmed?
Which subgroup fails?
What happens when the model is wrong?
Can users contest the result?
Is the model being used beyond its valid context?
```

---

# 7. Ethical Research & Planning

PPT 第 7 页正式定义了三块内容：

1. **Ethical Research**
    
2. **Data Management Plans**
    
3. **Ethics Applications**
    

---

## 7.1 Ethical Research

正式 PPT 列出：

```text
Ethical Research
- Does not cause harm
- Has a positive impact
- Respects laws - individual’s rights / expectations
- Is safe
```

可以扩展为：

|英文术语|中文含义|对项目的要求|
|---|---|---|
|Does not cause harm|不造成伤害|避免身体、心理、数据、社会风险|
|Has a positive impact|有正面影响|研究要有合理价值|
|Respects laws|遵守法律|遵守 GDPR、licence、copyright、学校政策|
|Individual’s rights / expectations|尊重个人权利和合理期待|consent、withdrawal、privacy|
|Is safe|安全|实验环境、数据安全、计算环境安全|

---

## 7.2 Data Management Plans

PPT 定义：

```text
Data Management Plans
- Outlines how research data will be collected, organized, stored, shared, and preserved throughout and after a research project.
```

注意这里比我们之前笔记中的“收集、存储、分享”更完整。

正式 PPT 明确包含：

```text
collected
organized
stored
shared
preserved
throughout and after a research project
```

也就是说，DMP 不只管项目期间，还管项目之后。

---

## 7.3 Ethics Applications

PPT 定义：

```text
Ethics Applications
- Submitting your plan to an independent panel for ethical review
```

关键点是：

**ethics application 是把你的研究计划交给 independent panel 审查，而不是自己觉得没问题就可以开始。**

---

# 8. Ethical Commitments：Research Integrity, RRI, FAIR, CARE

PPT 第 8 页是非常重要的理论页。

---

## 8.1 Research Integrity

PPT 定义：

```text
Research integrity refers to the adherence to ethical principles and professional standards throughout the research process.
It underpins trust in research and ensures that research is conducted responsibly and transparently.
```

中文理解：

**Research Integrity（研究诚信）** 是指在整个研究过程中遵守伦理原则和专业标准，使研究可信、负责、透明。

它包括：

- honest reporting
    
- transparent methods
    
- responsible data use
    
- no fabrication
    
- no falsification
    
- no plagiarism
    
- proper consent
    
- proper approval
    
- reproducibility
    
- acknowledgement of limitations
    

---

## 8.2 Responsible Research & Innovation

PPT 定义：

```text
Responsible Research & Innovation - RRI is a framework that ensures scientific research and innovation are conducted in ways that are ethically acceptable, socially desirable, and sustainable.
It emphasizes the alignment of research outcomes with societal values, needs, and expectations.
```

中文理解：

**Responsible Research and Innovation（负责任研究与创新）** 不只是问：

```text
Can we build this?
```

还要问：

```text
Should we build this?
Is it socially desirable?
Is it ethically acceptable?
Is it sustainable?
Does it align with social values and expectations?
```

---

## 8.3 FAIR Principles

PPT 中 FAIR 指：

```text
F = Findable
A = Accessible
I = Interoperable
R = Reusable
```

即：

|FAIR|含义|中文|
|---|---|---|
|Findable|data can be found|可发现|
|Accessible|data can be accessed|可访问|
|Interoperable|data can work with other systems/tools|可互操作|
|Reusable|data can be reused|可复用|

但注意：

**FAIR 不等于所有数据都必须公开。**

如果数据包含 personal data、licence 限制或 consent 限制，那么你只能在合规范围内开放。

更准确的原则是：

```text
As open as possible, as closed as necessary.
```

---

## 8.4 CARE Principles

PPT 中 CARE 指：

```text
C = Collective Benefit
A = Authority to Control
R = Responsibility
E = Ethics
```

中文：

|CARE|中文含义|
|---|---|
|Collective Benefit|集体利益|
|Authority to Control|控制权 / 数据主体或群体对数据的控制|
|Responsibility|责任|
|Ethics|伦理|

FAIR 更关注 data reuse，CARE 更强调 data responsibility。

所以在 AI / data science 项目中，不能只问：

```text
Can this dataset be reused?
```

还要问：

```text
Should this dataset be reused in this way?
Who might be affected?
Does the use respect the people or groups represented in the data?
```

---

# 9. When to get ethics approval

PPT 第 9 页正式区分了：

1. 需要 ethics approval 的研究
    
2. 不需要 ethics approval 的研究
    

---

## 9.1 需要 ethics approval 的研究

PPT 写道：

```text
Research that requires ethics approval
- Research that involves human participants
  either by primary data collection or secondary data analysis:
  e.g. their data, cells/tissue, themselves, their possessions
```

这里有几个关键点。

### 第一，human participants 不只是“人亲自来实验室”

包括：

- themselves
    
- their data
    
- their cells/tissue
    
- their possessions
    

所以，如果你使用人的数据，也可能算涉及 human participants。

---

### 第二，primary data collection 和 secondary data analysis 都可能触发审批

也就是说：

```text
I am only using an existing dataset.
```

不一定能免除 ethics approval。

如果 existing dataset 包含 human data，你仍需要检查是否需要审批。

---

## 9.2 不需要 ethics approval 的研究

PPT 列出：

```text
Research that doesn’t require ethics approval
- Analysis of anonymised aggregated data on individuals or organisations
  e.g. GDP, labour force participation rates, fertility rates
- Meta-analyses
- Literature reviews or reviews/analyses of reports, policies, documents, meeting minutes, newspaper articles, films
```

重点是：

### anonymised aggregated data

必须同时注意：

- anonymised
    
- aggregated
    

如果只是 pseudonymised，不一定免审批。

例如：

```text
student_001, age 23, department, submission time, writing sample
```

这可能仍然是 personal data。

但：

```text
Average grade by department
GDP by country
Labour force participation rate
Fertility rate
```

这类 aggregated data 通常风险低得多。

---

## 9.3 最重要句子：Retrospective approval is NOT possible

PPT 第 9 页明确写着：

```text
Retrospective approval is NOT possible
```

意思是：

**如果你需要 ethics approval，你必须在研究开始前获得。不能先做研究，再补申请。**

这包括：

- 不能先发问卷再申请
    
- 不能先访谈再申请
    
- 不能先招募再申请
    
- 不能先收集用户数据再申请
    
- 不能先分析需要审批的人类数据再申请
    

---

# 10. Where / How do I submit?

PPT 第 10 页说明提交系统：

```text
https://ergo2.soton.ac.uk/
```

即 **ERGO II**。

同时，PPT 明确说明：

```text
Failure to obtain ethics approval or to follow approval conditions may constitute:
- Major breach of Academic Integrity (student)
- Research Misconduct (staff)
```

这点非常重要。

对学生来说，不遵守 ethics approval 可能构成：

**Major breach of Academic Integrity**

这比普通格式错误严重得多。

---

# 11. Submission Interface

PPT 第 11 页列出 ERGO II submission interface 需要做的事：

```text
- Fill in Submission Questionnaire
- Add Supporting Documents
- Add Coordinators or Supervisors
- Receive Feedback
```

对于学生项目，最重要的是：

**Add Supervisors**

因为学生需要 supervisor initial review。

---

# 12. Submission Process：审批流程与类别

PPT 第 12 页是审批流程页，比字幕转写更清楚。

---

## 12.1 学生必须先经过 supervisor review

PPT 明确写：

```text
All Students require an initial review by their supervisor, staff do not.
```

流程可以表示为：

```text
Applying for Approval
        ↓
Supervisor Review
        ↓
Faculty Ethics Committee Review
        ↓
Research Integrity and Governance Review, if required
        ↓
Approval and Revisions
```

---

## 12.2 Category B, C & D

PPT 写：

```text
Category B, C & D only require Faculty Review
```

也就是说，较低类别的研究通常只需要 Faculty Review。

---

## 12.3 Category A

PPT 写：

```text
Category A goes to Faculty Review then onto RIG
```

其中 RIG 是：

**Research Integrity and Governance**

即更高层级的研究诚信与治理审查。

---

## 12.4 Category A*

PPT 写：

```text
Category A* go straight to RIG
```

说明 A* 是最高风险类别之一，会直接进入更高级别审查。

---

## 12.5 审批时间

PPT 写：

```text
10 working days to approve
```

但你写 project plan 时不要理解成“提前 10 个工作日一定够”。

因为如果材料不完整或需要修改，会出现：

```text
Submit
    ↓
Feedback
    ↓
Revision
    ↓
Resubmission
    ↓
Review again
```

所以实际要提前更多时间。

---

# 13. What do I submit?

PPT 第 13 页列出材料要求。

---

## 13.1 Submission Form 永远需要

PPT 写：

```text
Submission Form (Always)
```

这是所有申请都要填的。

---

## 13.2 Primary Data Collection 或 Secondary Data Analysis 二选一

PPT 写：

```text
Ethics Form (for Primary Data Collection)
or
Secondary Data Analysis Form
```

也就是说，你要根据项目类型选择正确表格。

---

## 13.3 如果是 Primary Data Collection

PPT 列出需要提交：

```text
- Data Gathering
- Risk Assessment
- Participant Information
- Consent Form
- Data Protection Act Plan
- Debrief Plan
  only where studies are intrusive, involve risk or harm, or involve deception
```

可以整理成：

|材料|什么时候需要|作用|
|---|---|---|
|Data Gathering|primary data collection|说明你怎么收集数据、问什么、记录什么|
|Risk Assessment|primary data collection|评估身体、心理、数据、环境等风险|
|Participant Information|有参与者|告诉参与者研究目的、过程、数据使用、风险、退出权|
|Consent Form|有参与者|获得 informed consent|
|Data Protection Act Plan|涉及 personal/sensitive data|说明数据保护、存储、访问、处理|
|Debrief Plan|deception / risk / harm / intrusive study|研究后解释、支持、退出机会|

---

# 14. Beware these pitfalls：常见陷阱

PPT 第 14 页是最实用的一页。

---

## 14.1 If you aren’t sure, check

PPT 写：

```text
If you aren’t sure if you need ethics, CHECK!
```

这意味着你不能自己猜。

可以问：

- supervisor
    
- FEPS Ethics Committee
    
- ERGO / RGO support
    
- module leader
    

---

## 14.2 Secondary Data Analysis 陷阱

PPT 写：

```text
If you are re-using data that contains human data from previous projects,
does EVERYONE have permission to use the data?
```

重点是：

**不是某个人有数据，你就能用。**

需要确认：

- supervisor 是否仍有权限
    
- student 是否有权限
    
- project team 是否有权限
    
- data sharing agreement 是否覆盖当前使用
    
- 原始 consent 是否允许当前用途
    

---

PPT 还写：

```text
If you are using a public dataset for secondary analysis –
does it have an appropriate license,
and WAS IT COLLECTED WITH CONSENT
[Just because it exists on Kaggle doesn’t mean you can use it…]
```

这句话极其重要，尤其对 AI / ML 学生。

不能写：

```text
The dataset is public, so there is no ethical issue.
```

正确逻辑是：

```text
Public availability does not automatically imply ethical or legal permission for reuse.
```

你需要检查：

```text
source
licence
terms of use
consent
collection method
ethics statement
reuse permission
redistribution permission
```

---

## 14.3 Primary Data Collection 陷阱

PPT 写：

```text
Have you included ALL of your data gathering/collection mechanisms?
Have you included ALL of the relevant forms e.g. risk assessment?
```

意思是，不能只说“我会做一个问卷”。

你需要提供：

- 问卷题目
    
- 招募方式
    
- consent form
    
- participant information sheet
    
- data storage plan
    
- risk assessment
    
- withdrawal process
    

如果是 interview，还要提供 interview guide。

如果是 user study，还要提供 task description。

---

## 14.4 General 陷阱

PPT 写：

```text
START EARLY – these applications can take more time than you need
BE REALISTIC with start and finish dates –
these should include analysis time not just data collection!
```

这里特别重要：

ethics approval 的时间范围不应该只覆盖 data collection，还要覆盖 data analysis。

所以 MSc 项目可以把 end date 设置到项目结束或 master’s end date，而不是只写问卷结束日期。

---

## 14.5 Mass Mailing is NOT permitted

PPT 第 14 页明确写：

```text
Mass Mailing IS NOT PERMITTED!
You may not use @soton.ac.uk email addresses for recruiting participants,
and you may not use university wide distribution lists,
there is a trial for using certain school / faculty mailing lists
```

所以不能随便向全校学生邮箱发招募邮件。

这对 user study 很关键。

允许的可能是：

- approved mailing list
    
- opt-in list
    
- supervisor-approved recruitment channel
    
- school/faculty trial route
    
- posters
    
- voluntary sign-up link
    

但不允许：

- 把 @soton.ac.uk 邮箱批量爬出来群发
    
- 用 university-wide distribution list
    
- 直接 spam 全体学生
    
- 强迫同学参与
    

---

# 15. Useful Links

PPT 第 15 页列出了一组学校资源。

你写 project plan 不一定需要全部列出，但要知道它们的作用：

```text
Research Data Management Guidelines
Ethics SharePoint
ERGO2 Guidance
Template Documents
Categories of Research
Using FEPS Mailing Lists for Human Participant Study Invitations
```

这些对应：

- 数据管理
    
- 伦理政策
    
- ERGO2 使用说明
    
- 模板文件
    
- 研究类别判断
    
- 招募邮件规则
    

---

# 16. New AI Guidelines

PPT 第 16 页是新增且非常重要的 AI 相关内容。

---

## 16.1 AI guidelines 适用范围

PPT 写：

```text
This guidance is for researchers integrating third party platforms
and AI tools, including Generative AI tools, into their research processes.
```

也就是说，如果研究中使用：

- ChatGPT
    
- Copilot
    
- Claude
    
- Gemini
    
- recruitment platforms
    
- AI transcription tools
    
- AI translation tools
    
- AI coding / summarisation tools
    
- AI data cleaning tools
    

都可能涉及这份 guidance。

---

## 16.2 不需要在 ethics application 中声明的例子

PPT 给了两个例子：

```text
AI tool used for translation of documents not containing personal data.
Using AI tool for systematic review or meta-analysis.
```

也就是说，如果文档不含 personal data，用 AI 翻译通常风险较低。

系统综述或 meta-analysis 中使用 AI 工具也可能不需要特别声明，但仍要遵守 academic integrity 和 critical use。

---

## 16.3 AI tool 使用警告

PPT 的 caution 部分非常重要：

```text
Researchers should only consider using a third-party platform / AI tool
if it is essential for the project
or whether other methods could achieve the same ends.
```

意思是：

不要因为方便就随便用第三方 AI 工具处理研究数据。

必须问：

```text
Is this essential?
Can the same aim be achieved without uploading data to a third-party tool?
Is there a safer university-approved alternative?
```

---

## 16.4 Use Co-Pilot if you need to use one

PPT 明确写：

```text
Use Co-Pilot if you need to use one!
```

这里的意思是，如果确实需要用 AI 工具，应优先使用学校支持或有 institutional arrangement 的工具，而不是随便用外部服务。

但这不等于可以随便上传 personal data。

---

## 16.5 AI model 中的数据通常无法删除

PPT 写：

```text
Once you put your data into an AI model,
then erasing it is often not viable.
This can affect participant withdrawal.
Make sure to INFORM PARTICIPANTS with CORRECT INFORMATION.
```

这是 AI ethics 的核心点。

如果你把参与者数据上传到 AI 工具：

- 数据可能被外部系统保留
    
- 可能无法彻底删除
    
- 可能影响 right of withdrawal
    
- 参与者必须提前知道
    
- consent form 里要写清楚
    

所以不能让参与者只同意：

```text
I agree that the researcher can analyse my data.
```

如果要用 AI 工具，必须让他们知道：

```text
My data may be processed using an AI tool or third-party platform.
```

---

## 16.6 Secondary data 输入 AI 工具也要小心

PPT 写：

```text
If researchers are planning to input any secondary data into third-party platforms / AI tools,
they should refer to the University’s secondary data analysis guidelines.
```

这说明，即使数据不是你自己收集的，只要你要把它放进 AI 工具，也要考虑：

- 原数据 licence 是否允许
    
- 原 consent 是否允许
    
- 是否包含 personal data
    
- 是否允许 third-party processing
    
- 是否允许 model training / external processing
    
- 是否违反数据集 terms of use
    

---

# 17. Contacts

PPT 第 17 页提供了联系人：

```text
d.j.taunton@soton.ac.uk
s.pearman-kanza@soton.ac.uk
risethic@soton.ac.uk
rgoinfo@soton.ac.uk
```

如果你写项目计划时不确定是否需要 ethics approval，可以联系：

- supervisor
    
- ethics support
    
- RGO / research governance contacts
    

---

# 18. 和你 MSc Project Plan 最相关的最终写法

结合这份 PPT，你的 AI-generated text detection 项目可以这样写 ethics 部分。

---

## 18.1 Ethical Statement 推荐版本

```text
This project does not plan to recruit human participants or collect new primary personal data. It will use public or research datasets only. However, public availability does not automatically imply ethical or legal permission for reuse. Therefore, before using each dataset, the project will check its source, licence, reuse terms, original collection method, and whether the data was collected with appropriate consent.

If any dataset contains human-generated text or potentially identifiable information, the project will assess whether secondary data analysis ethics approval is required. Any change to the project scope involving surveys, interviews, user studies, participant recruitment, or new human data collection will not begin until the supervisor has been consulted and any required ERGO II ethics approval has been obtained.

The project will also consider the downstream ethical risks of AI-generated text detection. False positives may unfairly affect students, non-native writers, or users from under-represented language groups. Therefore, the evaluation will report not only aggregate accuracy but also robustness across language, text length, paraphrasing, domain, and generator type. Detector outputs will be treated as uncertain model predictions rather than definitive evidence of misconduct.
```

---

## 18.2 Data Management Plan 推荐版本

```text
The project will maintain a structured data management process. Raw datasets will be stored separately from processed versions, and all cleaning, filtering, and split construction will be implemented through reproducible scripts rather than manual editing. A data log will record each dataset’s name, version, source, licence, access date, original collection method, permitted reuse terms, preprocessing steps, and known limitations or biases.

Large datasets, model checkpoints, and generated outputs will not be committed directly to version control. The repository will contain source code, configuration files, documentation, and small examples only. Where dataset licences restrict redistribution, the raw data will not be republished. Any examples used in the dissertation will be checked to avoid reproducing personal or sensitive information.

If any personal or participant data is introduced later, it will be stored only on university-approved secure platforms, and third-party or AI tools will not be used to process such data unless this is essential, approved where required, and clearly explained to participants in advance.
```

---

## 18.3 AI Tools 使用声明推荐版本

```text
Generative AI tools may be used to support non-sensitive tasks such as drafting, code explanation, or summarising documents that do not contain personal data. They will not be used to process participant data, identifiable text, or restricted datasets unless this is necessary for the project, permitted by the relevant dataset terms, approved where required, and clearly disclosed in the ethics documentation. If an AI tool is required, university-supported tools will be preferred where possible.
```

---

# 19. 最终考试 / 写作重点总结

这份 PPT 最重要的考点和 project plan 重点是：

1. **Research involving human participants or human data may require ethics approval**, whether through primary data collection or secondary data analysis.
    
2. **Retrospective approval is not possible**：如果需要审批，必须先审批再开始。
    
3. **Failure to obtain ethics approval may constitute a major breach of academic integrity for students**.
    
4. **Public dataset 不等于 automatically usable**：Kaggle 上有数据，也不代表你可以用。
    
5. **Secondary data analysis must check licence, consent, reuse terms and permissions**.
    
6. **Primary data collection requires participant information, consent, risk assessment, data gathering documents, and possibly data protection and debrief plans**.
    
7. **Mass mailing is not permitted**：不能随便用 @soton.ac.uk 或 university-wide distribution lists 招募参与者。
    
8. **AI tools introduce new risks**：特别是 personal data 上传后可能无法删除，会影响 participant withdrawal。
    
9. **Data Management Plan 要覆盖 collected, organized, stored, shared, preserved，而且包括项目之后的数据处理。**
    
10. **Ethics 和 diversity 紧密相关**：biased data 会产生 biased systems，并进一步 perpetuate bias。