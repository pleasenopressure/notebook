# 任务介绍
## 1) 背景与任务动机

在临床技能训练或考试中，医学生会针对一个标准化病人（clinical case）写一段 **patient note**（尤其是 _history_ 部分），描述病人的症状、病史、生活方式等。评分人员会根据一份 **评分量表/要点清单（rubric）** 来判断学生是否提到了某些关键临床概念（clinical features）。

这个 Kaggle 任务的目标就是：**自动化地识别** patient note 中哪些文字片段表达了 rubric 里的某个临床要点，例如 rubric 写 “diminished appetite”，而学生可能写成 “eating less / not hungry / clothes fit looser”等不同表达。

---

## 2) 任务形式（输入与输出）

### 输入（对每一个样本）

一个样本可以理解为一个三元组：

1. **Clinical case（病例）**：一个病例场景（如某种疾病背景、主诉等），用 `case_num` 标识（在 `patient_notes.csv` / `features.csv` 中关联）。
    
2. **Patient note 文本**：医学生写的自由文本（主要是 history 部分），用 `pn_num` 标识。
    
3. **Feature（评分要点）**：该病例下的一条 rubric 要点文本（feature text），用 `feature_num` 标识。
    

在建模上，你可以把它看成：

> 给定（note 文本 + feature 文本），预测 note 里哪些字符位置对应这个 feature。

### 输出（你需要预测什么）

你需要输出的是：**在 patient note 中对应 feature 的文字片段位置（spans）**。竞赛用字符级区间来表示，例如用起止索引表示“从第 i 个字符到第 j 个字符”。

---

## 3) 标注数据（训练集标签长什么样）

训练集中，会给出每个（case, note, feature）的真实标注：

- `annotation`：标注的文本片段内容（可能有多个片段）
    
- `location`：这些片段在 note 里的**字符索引范围**（例如 “0 4” 代表 note 的前 5 个字符）
    

注意：并不是每条 patient note 的所有 feature 都有标注；通常是“部分 note-feature 对”被标注为包含/不包含该要点（有些 feature 在该 note 里可能根本没出现）。

---

## 4) 测试集形式

测试集会提供同样的 `patient_notes.csv`（note 文本），以及对应的 case/feature 信息；你需要对每个样本输出你预测到的 spans。Kaggle 的 data 页面也提到测试集中大约有 **2000** 条 patient notes。

---

## 5) 评测方式（你们最后比较模型的依据）

该竞赛的核心评测指标是 **micro-averaged F1**，它基于你预测的 spans 与真实 spans 的匹配/重叠来计算精确率与召回率，并做 micro 平均后得到 F1。

从项目角度，这非常适合你们做：

- baseline vs transformer基线与变压器
    
- 不同后处理（阈值、span 合并、字符对齐）
    
- case-wise / feature-wise error analysis逐个案例/逐个特征的错误分析
    
- ablation study消融研究
    

---

## 6) 用一句话总结这个问题

> **给定一段医学生写的临床 patient note，以及一条 rubric clinical feature，系统要在 note 中找出表达该 feature 的所有文本片段（字符级 spans），并用 micro-F1 进行评测。
>**

# 2.24号会议总结
## 主要议题
1. 分工
2. 讲解一下数据集
3. 讨论技术路线，baseline
## 数据集中存在的问题
1. 首先训练集的标签数据很少
2. 没有测试集（得提交kaggle）
3. 为什么使用BIO而不是直接的span（可消融）
4. train中的空的annotation是作为负样本保留还是直接删掉降低复杂度
5. 

