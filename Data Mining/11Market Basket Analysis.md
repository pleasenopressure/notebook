
# Lecture 9：Market Basket Analysis 最终复习笔记

## 1. Market Basket Analysis 的核心问题

**Market Basket Analysis（购物篮分析）**研究的是：

> 给定一批顾客交易记录，找出哪些商品经常被一起购买。

换句话说，我们不是在预测单个商品本身，而是在从大量交易中挖掘 **co-occurrence pattern（共现模式）**。例如，经典故事是 “Beer and Nappies / Beer and Diapers”，即有人发现购买尿布的顾客也经常购买啤酒。不过 slides 特别提醒，这个故事本身并没有很强的统计依据，但它非常适合作为理解购物篮分析直觉的例子。

从商业角度看，如果超市发现某些商品经常一起出现，就可以用于：

- **Product placement（商品摆放）**：把经常一起买的商品放近，方便顾客购买；
    
- **Promotion（促销）**：对组合商品做捆绑优惠；
    
- **Recommendation（推荐系统）**：顾客买了 X 后推荐 Y；
    
- **Customer behaviour understanding（顾客行为理解）**：理解顾客购买习惯。
    

课堂转写中老师强调，Market Basket Analysis 的目标就是观察 transaction dataset，然后找出 **groups of items that are frequently bought together**，也就是经常一起出现的商品组合。

---

## 2. 基本概念：Item、Itemset、Transaction、Dataset

### 2.1 Item（商品 / 项）

**Item（商品 / 项）**是购物篮里的一个单独产品。通常用小写字母：

```text
i
```

表示一个具体 item。

例如：

```text
Beer
Diaper
Milk
Baby Powder
Bread
```

都可以是 item。

---

### 2.2 I：所有商品的全集

用大写：

```text
I = {i1, i2, ..., im}
```

表示商店里所有可能出现的 item 的集合。slides 中把它定义为 **the set of all items sold in the store**。

例如，如果整个数据集中只出现过：

```text
Beer, Diaper, Baby Powder, Bread, Umbrella, Milk, Detergent, Coca-Cola
```

那么：

```text
I = {Beer, Diaper, Baby Powder, Bread, Umbrella, Milk, Detergent, Coca-Cola}
```

---

### 2.3 Transaction（交易 / 购物篮）

**Transaction（交易）**就是一次购物记录，也可以理解成一个 **basket（购物篮）**。每个 transaction 可能有一个 **TID（Transaction ID，交易编号）**。

例如 slides 里的 5 条 transaction 是：

```text
T1 = {Baby Powder, Beer, Bread, Diaper, Umbrella}
T2 = {Baby Powder, Diaper}
T3 = {Beer, Diaper, Milk}
T4 = {Beer, Detergent, Diaper}
T5 = {Beer, Coca-Cola, Milk}
```

每一行就是一个 transaction。

---

### 2.4 Transactional Dataset（交易数据集）

**Transactional dataset（交易数据集）**是一组 transactions：

```text
T = {t1, t2, ..., tn}
```

其中：

- `n` 是交易数量；
    
- 每个 `tj` 是一个 basket；
    
- 每个 basket 由若干 item 构成。
    

---

### 2.5 Itemset（项集 / 商品集合）

**Itemset（项集）**是若干 item 组成的集合：

```text
X ⊆ I
```

例如：

```text
X = {Baby Powder, Diaper}
```

因为它包含两个 item，所以它是一个 **2-itemset（二项集）**。slides 中也使用这个例子说明 itemset 的概念。

如果 itemset 里有 k 个 item，就称为：

```text
k-itemset
```

例如：

```text
{Beer}                         1-itemset
{Beer, Diaper}                 2-itemset
{Beer, Diaper, Milk}           3-itemset
{Beer, Diaper, Milk, Bread}    4-itemset
```

---

## 3. Association Rule（关联规则）

### 3.1 关联规则的形式

**Association rule（关联规则）**的标准形式是：

```text
X → Y
```

其中：

```text
X ⊆ I
Y ⊆ I
X ∩ Y = ∅
```

也就是说：

- `X` 和 `Y` 都是 itemset；
    
- `X` 和 `Y` 都来自全集 `I`；
    
- `X` 和 `Y` 不能有重叠 item；
    
- `X` 是前件，英文叫 **antecedent / left-hand side / LHS**；
    
- `Y` 是后件，英文叫 **consequent / right-hand side / RHS**。
    

slides 中明确写出 association rule 是 implication 的形式 `X → Y`，并要求 `X ∩ Y = ∅`。

---

### 3.2 关联规则的含义

规则：

```text
{Beer} → {Diaper}
```

含义不是：

> 买 Beer 一定会买 Diaper。

而是：

> 在当前交易数据集中，买了 Beer 的交易里，有较大比例也包含 Diaper。

所以 association rule 是一种 **统计规律（statistical pattern）**，不是因果关系（causality）。

这点非常重要：

```text
X → Y does not mean X causes Y.
It means Y often appears together with X in the transaction dataset.
```

---

### 3.3 关联规则是有方向的

这是一个非常容易错的点。

```text
{Beer} → {Diaper}
```

和：

```text
{Diaper} → {Beer}
```

不是同一条规则。

它们的 **support（支持度）**一样，因为它们都看 `{Beer, Diaper}` 同时出现的比例。

但是它们的 **confidence（置信度）**可能不同，因为 denominator 不同：

```text
confidence(Beer → Diaper) = count(Beer ∪ Diaper) / count(Beer)

confidence(Diaper → Beer) = count(Diaper ∪ Beer) / count(Diaper)
```

所以规则方向会影响 confidence。

---

## 4. Support Count、Support、Confidence

这是本节课最重要的计算部分。

---

### 4.1 Support Count（支持计数）

对于一个 itemset `A`，它的 **support count（支持计数）**表示：

> 数据集中有多少个 transactions 包含 A。

记作：

```text
A.count
```

例如：

```text
A = {Beer, Diaper}
```

在 5 条交易中：

```text
T1 = {Baby Powder, Beer, Bread, Diaper, Umbrella}  包含 Beer 和 Diaper
T2 = {Baby Powder, Diaper}                         不包含 Beer
T3 = {Beer, Diaper, Milk}                          包含 Beer 和 Diaper
T4 = {Beer, Detergent, Diaper}                     包含 Beer 和 Diaper
T5 = {Beer, Coca-Cola, Milk}                       不包含 Diaper
```

所以：

```text
count({Beer, Diaper}) = 3
```

---

### 4.2 Support（支持度）

对于 itemset `A`：

```text
support(A) = count(A) / n
```

其中：

- `count(A)` 是包含 A 的 transactions 数量；
    
- `n` 是总交易数。
    

对于规则：

```text
X → Y
```

它的 support 是：

```text
support(X → Y) = support(X ∪ Y)
               = count(X ∪ Y) / n
```

含义是：

> 整个数据集中，同时包含 X 和 Y 的交易占比是多少。

slides 中 support 的 numerator 是同时包含 X 和 Y 的交易数量，denominator 是总交易数量。

---

### 4.3 Confidence（置信度）

对于规则：

```text
X → Y
```

confidence 定义为：

```text
confidence(X → Y) = count(X ∪ Y) / count(X)
```

也可以理解为条件概率：

```text
confidence(X → Y) = P(Y | X)
```

含义是：

> 在已经购买 X 的交易里，有多少比例也购买了 Y。

slides 中 confidence 的 numerator 与 support 相同，都是同时包含 X 和 Y 的交易数量；但是 denominator 改成了只包含前件 X 的交易数量。

---

## 5. 例子：计算 Beer → Diaper

使用 slides 中的 5 条交易：

```text
T1 = {Baby Powder, Beer, Bread, Diaper, Umbrella}
T2 = {Baby Powder, Diaper}
T3 = {Beer, Diaper, Milk}
T4 = {Beer, Detergent, Diaper}
T5 = {Beer, Coca-Cola, Milk}
```

规则：

```text
{Beer} → {Diaper}
```

---

### 5.1 计算 support

先找同时包含 Beer 和 Diaper 的交易：

```text
T1: yes
T2: no
T3: yes
T4: yes
T5: no
```

所以：

```text
count({Beer, Diaper}) = 3
```

总交易数：

```text
n = 5
```

因此：

```text
support(Beer → Diaper) = count({Beer, Diaper}) / 5
                       = 3 / 5
                       = 0.6
                       = 60%
```

---

### 5.2 计算 confidence

先找包含 Beer 的交易：

```text
T1: yes
T2: no
T3: yes
T4: yes
T5: yes
```

所以：

```text
count({Beer}) = 4
```

再看其中有多少也包含 Diaper：

```text
count({Beer, Diaper}) = 3
```

因此：

```text
confidence(Beer → Diaper) = count({Beer, Diaper}) / count({Beer})
                          = 3 / 4
                          = 0.75
                          = 75%
```

这与 slides 中 Beer → Diaper 的例子一致：support = 3/5 = 60%，confidence = 3/4 = 75%。

---

## 6. Support 和 Confidence 的本质区别

### 6.1 Support 衡量“规则有多常见”

**Support（支持度）**回答的是：

> 这条规则对应的 item combination 在整个数据集中出现得多不多？

例如：

```text
support(Beer → Diaper) = 60%
```

意思是：

> 60% 的所有购物篮同时包含 Beer 和 Diaper。

所以 support 更关注 **global frequency（全局频率）**。

---

### 6.2 Confidence 衡量“规则有多可靠”

**Confidence（置信度）**回答的是：

> 如果 X 出现了，Y 出现的概率有多大？

例如：

```text
confidence(Beer → Diaper) = 75%
```

意思是：

> 在买 Beer 的交易中，75% 也买了 Diaper。

所以 confidence 更关注 **conditional reliability（条件可靠性）**。

---

### 6.3 为什么需要两个指标？

只看 support 不够，因为一个规则可能很常见，但不一定可靠。

只看 confidence 也不够，因为一个规则可能 confidence 很高，但出现次数太少，没有商业价值。

例如：

```text
{Luxury Caviar} → {Imported Champagne}
```

如果只出现过一次，并且那一次两个都买了：

```text
support = 1 / 1,000,000 = 0.0001%
confidence = 1 / 1 = 100%
```

confidence 很高，但 support 极低，这种规则不适合拿来指导超市整体策略。

所以我们需要同时设置：

```text
min_sup
min_conf
```

---

## 7. Minimum Support、Minimum Confidence、Frequent Itemset、Strong Rule

### 7.1 Minimum Support Threshold（最小支持度阈值）

**Minimum support threshold（最小支持度阈值）**通常写作：

```text
min_sup
```

例如：

```text
min_sup = 40%
```

如果一个 itemset 的 support 大于等于 `min_sup`，就叫：

```text
frequent itemset（频繁项集）
```

slides 中明确说明：如果一个 itemset 的 support 至少达到 min_sup，那么称它为 frequent itemset。

---

### 7.2 Minimum Confidence Threshold（最小置信度阈值）

**Minimum confidence threshold（最小置信度阈值）**通常写作：

```text
min_conf
```

例如：

```text
min_conf = 70%
```

如果一条 rule 的 confidence 大于等于 `min_conf`，说明它比较可靠。

---

### 7.3 Strong Rule（强规则）

一条 association rule 要成为 **strong rule（强规则）**，必须同时满足：

```text
support ≥ min_sup
confidence ≥ min_conf
```

slides 中给出的 association rule mining 目标就是：在 transaction dataset `T` 中找出所有 support 和 confidence 都达到阈值的 rules。

---

## 8. Association Rule Mining 的目标

**Association Rule Mining（关联规则挖掘）**的目标是：

> 给定交易数据集 `T`，找出所有满足最小支持度和最小置信度的规则。

形式化地说：

```text
Find all rules X → Y such that:

support(X → Y) ≥ min_sup
confidence(X → Y) ≥ min_conf
```

---

## 9. 最朴素方法：Brute-force Approach（暴力枚举）

最直接的方法是：

```text
1. List all possible association rules.
2. Compute support and confidence for every rule.
3. Remove rules that fail min_sup or min_conf.
```

slides 中称这种方法 computationally expensive，因为它需要列出所有可能规则并逐个计算 support 和 confidence。

---

### 9.1 为什么暴力方法很贵？

假设有 `d` 个不同 item。

所有非空 itemset 的数量是：

```text
2^d - 1
```

原因是：

- 每个 item 都有两种选择：选或不选；
    
- 所以所有 subset 数量是 `2^d`；
    
- 去掉空集，所以是 `2^d - 1`。
    

slides 中用 `{A, B, C, D, E}` 这 5 个 item 画出 lattice（项集格），从 Level 1 的单个 item，到 Level 5 的 `{A, B, C, D, E}`，并说明给定 `d` 个 item，就有 `2^d - 1` 个 candidate itemsets。

例如：

```text
d = 5
2^5 - 1 = 31
```

如果：

```text
d = 100
2^100 - 1
```

这个数量已经大到无法实际枚举。

---

### 9.2 Brute-force 的复杂度

slides 中给出 frequent itemset generation 的暴力复杂度近似为：

```text
O(N M w)
```

其中：

- `N` = number of transactions；
    
- `M` = number of candidate itemsets；
    
- `w` = transaction 的最大宽度 / 平均 item 数相关量；
    
- `M = 2^d - 1`。
    

因为 `M` 随 item 数 `d` 指数增长，所以暴力方法非常昂贵。

---

## 10. Apriori Algorithm 的核心思想

为了解决 brute-force 太贵的问题，我们使用 **Apriori Algorithm（Apriori 算法）**。

slides 中把 Apriori 分成两个阶段：

```text
Phase 1: Find all frequent itemsets.
Phase 2: Generate association rules from these frequent itemsets.
```

也就是说，Apriori 不直接一开始就生成所有 rules，而是先找 frequent itemsets。

为什么？

因为如果一个 rule：

```text
X → Y
```

要满足 support 阈值，那么：

```text
X ∪ Y
```

必须是 frequent itemset。

所以可以先找 frequent itemsets，再从 frequent itemsets 中生成 rules。

---

## 11. Apriori Principle / Anti-Monotone Property

这是 Apriori 算法最关键的数学性质。

### 11.1 正向表达

**Apriori Principle（Apriori 原理）**：

> Any subset of a frequent itemset must also be frequent.

中文：

> 如果一个 itemset 是频繁的，那么它的所有子集也一定是频繁的。

slides 中例子是：

```text
{Beer, Diaper, Milk} is frequent
⇒ {Beer, Diaper} must also be frequent
```

因为只要一条 transaction 包含 `{Beer, Diaper, Milk}`，它一定也包含 `{Beer, Diaper}`。所以三项集出现的次数不可能超过它的二项子集；如果三项集都已经频繁，那么二项子集必然也频繁。

---

### 11.2 逆否表达：剪枝真正使用的形式

Apriori 更常用的是它的逆否命题：

> Any superset of an infrequent itemset must also be infrequent.

中文：

> 如果一个 itemset 是不频繁的，那么它的所有超集都不可能频繁。

例如：

```text
{A, B} is infrequent
```

那么所有包含 `{A, B}` 的更大集合都一定 infrequent：

```text
{A, B, C}
{A, B, D}
{A, B, E}
{A, B, C, D}
{A, B, C, E}
{A, B, D, E}
{A, B, C, D, E}
```

这些都不需要再生成或测试。

slides 中用 lattice 图展示：如果 `AB` 被发现是 infrequent，那么所有包含 `AB` 的 supersets 都可以被 prune 掉。

---

### 11.3 为什么这是对的？

核心原因是 **support 单调不增**：

如果：

```text
A ⊆ B
```

那么：

```text
support(B) ≤ support(A)
```

因为包含大集合 `B` 的 transaction，一定也包含小集合 `A`；但包含小集合 `A` 的 transaction，不一定包含大集合 `B`。

例如：

```text
A = {Beer}
B = {Beer, Diaper}
```

每个包含 `{Beer, Diaper}` 的 basket 一定包含 `{Beer}`，但包含 `{Beer}` 的 basket 不一定包含 `{Diaper}`。

所以：

```text
count({Beer, Diaper}) ≤ count({Beer})
support({Beer, Diaper}) ≤ support({Beer})
```

这就是 Apriori 可以剪枝的数学基础。

---

## 12. Apriori Algorithm 的完整流程

slides 中给出的流程是：

```text
1. Let k = 1.
2. Generate frequent 1-itemsets.
3. Repeat until no new frequent itemsets are identified:
   a. Generate candidate (k+1)-itemsets from frequent k-itemsets.
   b. Scan the database and count support for each candidate.
   c. Eliminate candidates that are infrequent.
   d. Keep only frequent candidates as L(k+1).
```

用符号表示：

```text
Ck = candidate k-itemsets
Lk = frequent k-itemsets
```

流程是：

```text
C1 → L1
L1 → C2 → L2
L2 → C3 → L3
...
直到 Lk 为空
```

---

## 13. Apriori 示例：完整手算过程

使用 slides 中的 5 条交易：

```text
T1 = {Beer, Diaper, Baby Powder, Bread, Umbrella}
T2 = {Diaper, Baby Powder}
T3 = {Beer, Diaper, Milk}
T4 = {Diaper, Beer, Detergent}
T5 = {Beer, Milk, Coca-Cola}
```

设置：

```text
min_sup = 40% = 2/5
```

也就是说，一个 itemset 至少要出现在 2 条交易中，才算 frequent。

---

### 13.1 Step 1：生成 C1

`C1` 是所有 1-itemset：

```text
{Beer}
{Diaper}
{Baby Powder}
{Bread}
{Umbrella}
{Milk}
{Detergent}
{Coca-Cola}
```

计算 support count：

```text
Beer         4/5
Diaper       4/5
Baby Powder 2/5
Bread        1/5
Umbrella     1/5
Milk         2/5
Detergent    1/5
Coca-Cola    1/5
```

---

### 13.2 Step 2：得到 L1

保留 support ≥ 2/5 的 item：

```text
L1 = {
  {Beer},
  {Diaper},
  {Baby Powder},
  {Milk}
}
```

被删除的 item：

```text
{Bread}
{Umbrella}
{Detergent}
{Coca-Cola}
```

因为它们只出现了 1/5，小于 2/5。

slides 中 C1 到 L1 的结果正是保留 Beer、Diaper、Baby Powder、Milk。

---

### 13.3 Step 3：由 L1 生成 C2

从 L1 中两两组合，生成 candidate 2-itemsets：

```text
{Beer, Diaper}
{Beer, Baby Powder}
{Beer, Milk}
{Diaper, Baby Powder}
{Diaper, Milk}
{Baby Powder, Milk}
```

计算 support：

```text
{Beer, Diaper}          3/5
{Beer, Baby Powder}     1/5
{Beer, Milk}            2/5
{Diaper, Baby Powder}   2/5
{Diaper, Milk}          1/5
{Baby Powder, Milk}     0/5
```

---

### 13.4 Step 4：得到 L2

保留 support ≥ 2/5 的 2-itemsets：

```text
L2 = {
  {Beer, Diaper},
  {Beer, Milk},
  {Diaper, Baby Powder}
}
```

被删除的是：

```text
{Beer, Baby Powder}
{Diaper, Milk}
{Baby Powder, Milk}
```

slides 中 C2 到 L2 的结果也是这三个 frequent 2-itemsets。

---

### 13.5 Step 5：由 L2 生成 C3

理论上可考虑的 3-itemsets 包括：

```text
{Beer, Diaper, Baby Powder}
{Beer, Diaper, Milk}
{Beer, Milk, Baby Powder}
{Diaper, Baby Powder, Milk}
```

计算 support：

```text
{Beer, Diaper, Baby Powder}  1/5
{Beer, Diaper, Milk}         1/5
{Beer, Milk, Baby Powder}    0/5
{Diaper, Baby Powder, Milk}  0/5
```

它们都小于 `2/5`。

所以：

```text
L3 = ∅
```

算法停止。

slides 中 C3 为空，表示没有新的 frequent 3-itemset。

---

## 14. 从 Frequent Itemsets 生成 Association Rules

前面 Apriori 的第一阶段得到：

```text
L2 = {
  {Beer, Diaper},
  {Beer, Milk},
  {Diaper, Baby Powder}
}
```

现在进入第二阶段：

```text
Generate rules from frequent itemsets.
```

---

### 14.1 由 2-itemset 生成规则

每个 frequent 2-itemset 可以生成 2 条方向相反的规则。

例如：

```text
{Beer, Diaper}
```

可以生成：

```text
{Beer} → {Diaper}
{Diaper} → {Beer}
```

同理：

```text
{Beer, Milk}
```

生成：

```text
{Beer} → {Milk}
{Milk} → {Beer}
```

```text
{Diaper, Baby Powder}
```

生成：

```text
{Diaper} → {Baby Powder}
{Baby Powder} → {Diaper}
```

所以一共 6 条 candidate rules。

---

### 14.2 重要难点：从 frequent itemset 生成的 rule，support 自动合格，但 confidence 不一定合格

这是课堂转写中特别强调的点：基于 frequent itemsets 生成的 association rules，并不是默认就是 strong rules；它们的 support 已经满足阈值，但 confidence 仍然需要计算和过滤。

原因是：

如果 rule 来自 frequent itemset：

```text
X ∪ Y
```

那么：

```text
support(X → Y) = support(X ∪ Y)
```

因为 `X ∪ Y` 是 frequent itemset，所以 support 一定 ≥ min_sup。

但是：

```text
confidence(X → Y) = count(X ∪ Y) / count(X)
```

这个值是否 ≥ min_conf 并不确定。

所以第二阶段必须继续计算 confidence。

---

## 15. 规则生成示例：min_conf = 70%

设置：

```text
min_sup = 40%
min_conf = 70%
```

已经得到 frequent 2-itemsets：

```text
{Beer, Diaper}          support = 3/5 = 60%
{Beer, Milk}            support = 2/5 = 40%
{Diaper, Baby Powder}   support = 2/5 = 40%
```

生成 6 条规则并计算 confidence：

---

### 15.1 Rule 1：Beer → Diaper

```text
support = count({Beer, Diaper}) / 5
        = 3 / 5
        = 60%

confidence = count({Beer, Diaper}) / count({Beer})
           = 3 / 4
           = 75%
```

满足：

```text
support ≥ 40%
confidence ≥ 70%
```

所以保留。

---

### 15.2 Rule 2：Beer → Milk

```text
support = count({Beer, Milk}) / 5
        = 2 / 5
        = 40%

confidence = count({Beer, Milk}) / count({Beer})
           = 2 / 4
           = 50%
```

support 合格，但 confidence 不合格。

所以删除。

---

### 15.3 Rule 3：Diaper → Baby Powder

```text
support = count({Diaper, Baby Powder}) / 5
        = 2 / 5
        = 40%

confidence = count({Diaper, Baby Powder}) / count({Diaper})
           = 2 / 4
           = 50%
```

support 合格，但 confidence 不合格。

所以删除。

---

### 15.4 Rule 4：Diaper → Beer

```text
support = count({Diaper, Beer}) / 5
        = 3 / 5
        = 60%

confidence = count({Diaper, Beer}) / count({Diaper})
           = 3 / 4
           = 75%
```

保留。

---

### 15.5 Rule 5：Milk → Beer

```text
support = count({Milk, Beer}) / 5
        = 2 / 5
        = 40%

confidence = count({Milk, Beer}) / count({Milk})
           = 2 / 2
           = 100%
```

保留。

---

### 15.6 Rule 6：Baby Powder → Diaper

```text
support = count({Baby Powder, Diaper}) / 5
        = 2 / 5
        = 40%

confidence = count({Baby Powder, Diaper}) / count({Baby Powder})
           = 2 / 2
           = 100%
```

保留。

---

### 15.7 最终 Strong Rules

最终满足：

```text
min_sup = 40%
min_conf = 70%
```

的 strong rules 是：

```text
{Beer} → {Diaper}
support = 60%
confidence = 75%

{Diaper} → {Beer}
support = 60%
confidence = 75%

{Milk} → {Beer}
support = 40%
confidence = 100%

{Baby Powder} → {Diaper}
support = 40%
confidence = 100%
```

slides 中的结果页也给出这四条 association rules，其中两条 support 60%、confidence 75%，另外两条 support 40%、confidence 100%。

---

## 16. 为什么 Milk → Beer 的 confidence 是 100%，但 support 只有 40%？

这是理解 support 和 confidence 差异的好例子。

交易中 Milk 出现在：

```text
T3 = {Beer, Diaper, Milk}
T5 = {Beer, Coca-Cola, Milk}
```

Milk 出现了 2 次，而且这 2 次都和 Beer 一起出现。

所以：

```text
confidence(Milk → Beer) = 2 / 2 = 100%
```

但是 `{Milk, Beer}` 只在 5 条交易中出现了 2 次：

```text
support(Milk → Beer) = 2 / 5 = 40%
```

所以：

- confidence 高，说明只要看到 Milk，Beer 很可能出现；
    
- support 不高，说明 Milk + Beer 这个组合在全体交易中并不特别常见。
    

这就是为什么实际应用中不能只看 confidence。

---

## 17. 阈值 min_sup 和 min_conf 怎么选择？

课堂转写中老师提到，这两个 threshold 可以根据经验设置，也可以根据 validation / cross-validation 来调。

### 17.1 min_sup 太高会怎样？

如果：

```text
min_sup too high
```

结果：

- 只会保留非常常见的组合；
    
- 很多有趣但不那么频繁的 pattern 会被删掉；
    
- 规则数量较少；
    
- 计算更快；
    
- 可能错过 niche pattern。
    

例如：

```text
min_sup = 80%
```

在 5 条交易中至少要出现 4 次，这样只有非常常见的 itemset 才能留下。

---

### 17.2 min_sup 太低会怎样？

如果：

```text
min_sup too low
```

结果：

- frequent itemsets 数量暴增；
    
- candidate rules 数量暴增；
    
- 计算更慢；
    
- 可能产生很多偶然共现的无意义规则。
    

例如：

```text
min_sup = 1%
```

很多只出现少数几次的组合也会被保留。

---

### 17.3 min_conf 太高会怎样？

如果：

```text
min_conf too high
```

结果：

- 只保留非常可靠的规则；
    
- 规则更少；
    
- 可能过滤掉一些有商业价值但 confidence 稍低的规则。
    

---

### 17.4 min_conf 太低会怎样？

如果：

```text
min_conf too low
```

结果：

- 很多弱规则也被保留；
    
- 推荐或促销可能不可靠；
    
- 规则解释价值下降。
    

---

## 18. Apriori 的“剪枝”到底节省了什么？

暴力方法会枚举所有 itemset。

Apriori 的改进是：

```text
If an itemset is infrequent,
do not generate or test any of its supersets.
```

例如：

```text
{A, B} is infrequent
```

那么：

```text
{A, B, C}
{A, B, D}
{A, B, E}
{A, B, C, D}
{A, B, C, E}
{A, B, D, E}
{A, B, C, D, E}
```

都不用看。

这就是 slides 中从 lattice 图中把 AB 的所有 supersets 圈出并剪掉的含义。

---

## 19. Apriori 的伪代码

```text
Input:
    T: transaction dataset
    min_sup: minimum support threshold
    min_conf: minimum confidence threshold

Output:
    Strong association rules

Step 1: Frequent itemset generation

    k = 1
    C1 = all 1-itemsets in T
    L1 = itemsets in C1 whose support >= min_sup

    while Lk is not empty:
        C(k+1) = generate candidates from Lk
        prune candidates whose any k-subset is not frequent
        scan T and count support for each candidate
        L(k+1) = candidates whose support >= min_sup
        k = k + 1

    L = union of all Lk

Step 2: Rule generation

    For each frequent itemset F in L:
        For each non-empty proper subset X of F:
            Y = F - X
            rule = X -> Y
            confidence = support(F) / support(X)
            if confidence >= min_conf:
                output rule
```

---

## 20. Python 风格实现逻辑

下面不是为了让你背代码，而是帮助你理解算法结构。

```python
def support_count(itemset, transactions):
    count = 0
    itemset = set(itemset)

    for t in transactions:
        if itemset.issubset(t):
            count += 1

    return count


def support(itemset, transactions):
    return support_count(itemset, transactions) / len(transactions)


def confidence(X, Y, transactions):
    X = set(X)
    Y = set(Y)
    return support_count(X | Y, transactions) / support_count(X, transactions)
```

规则：

```python
X -> Y
```

的 support 和 confidence：

```python
rule_support = support(X | Y, transactions)
rule_confidence = confidence(X, Y, transactions)
```

注意：

```python
support(X -> Y) == support(Y -> X)
```

但：

```python
confidence(X -> Y) != confidence(Y -> X)
```

一般不相等。

---

## 21. 高频考试点与易错点

### 易错点 1：把 support 和 confidence 分母搞反

错误写法：

```text
confidence = count(X ∪ Y) / n
```

这是 support，不是 confidence。

正确写法：

```text
support(X → Y) = count(X ∪ Y) / n

confidence(X → Y) = count(X ∪ Y) / count(X)
```

---

### 易错点 2：认为 X → Y 和 Y → X 是一样的

它们不是一样的。

```text
support(X → Y) = support(Y → X)
```

但：

```text
confidence(X → Y) may differ from confidence(Y → X)
```

因为：

```text
count(X) may differ from count(Y)
```

---

### 易错点 3：从 frequent itemset 生成的 rule 默认就是 strong rule

错误。

从 frequent itemset 生成 rule，只能保证：

```text
support ≥ min_sup
```

不能保证：

```text
confidence ≥ min_conf
```

所以必须继续算 confidence。

课堂转写中老师明确纠正了这个点：association rules based on frequent itemsets are not strong association rules by default; confidence still needs to be computed。

---

### 易错点 4：Apriori Principle 的方向搞错

正确的是：

```text
Frequent itemset → all subsets are frequent
```

以及它的逆否命题：

```text
Infrequent itemset → all supersets are infrequent
```

错误说法：

```text
If a subset is frequent, all supersets are frequent.
```

这是错的。

例如：

```text
{Beer} frequent
```

并不代表：

```text
{Beer, Milk, Diaper, Bread}
```

也 frequent。

---

### 易错点 5：把 association rule 当成 causality

```text
Beer → Diaper
```

不表示 Beer 导致 Diaper，也不表示买 Beer 是买 Diaper 的原因。

它只表示：

```text
P(Diaper | Beer)
```

在当前数据集中比较高。

---

## 22. 最终知识框架总结

这节课可以压缩成一条完整主线：

```text
Transaction data
    ↓
Find item combinations that appear frequently
    ↓
Use support to identify frequent itemsets
    ↓
Use Apriori Principle to avoid brute-force enumeration
    ↓
Generate association rules from frequent itemsets
    ↓
Use confidence to filter reliable rules
    ↓
Output strong rules
```

更完整地说：

1. **Market Basket Analysis** 的目标是从 transactions 中发现经常一起购买的商品组合；
    
2. 数据由 **items、itemsets、transactions、transactional dataset** 构成；
    
3. **Association rule** 形式为 `X → Y`，表示买了 X 的交易中经常也包含 Y；
    
4. **Support** 衡量规则或 itemset 在整个数据集中多常见；
    
5. **Confidence** 衡量给定 X 后 Y 出现的可靠程度；
    
6. **Frequent itemset** 是 support 达到 `min_sup` 的 itemset；
    
7. **Strong rule** 是 support 和 confidence 都达到阈值的 rule；
    
8. 暴力枚举会产生 `2^d - 1` 个 candidate itemsets，计算非常昂贵；
    
9. **Apriori Algorithm** 利用 **Anti-Monotone Property** 剪枝；
    
10. Apriori 的核心性质是：如果一个 itemset 不频繁，它的所有 supersets 都不可能频繁；
    
11. Apriori 先找 frequent itemsets，再从它们生成 rules；
    
12. 从 frequent itemsets 生成的 rules 仍然必须计算 confidence；
    
13. 最终只保留满足 `min_sup` 和 `min_conf` 的 strong rules。
    

---

## 23. 一句话记忆版

**Market Basket Analysis** 是找“哪些商品经常一起买”；  
**Support** 看“这个组合整体出现得多不多”；  
**Confidence** 看“买了 X 以后买 Y 的比例高不高”；  
**Apriori** 用“频繁项集的子集一定频繁 / 不频繁项集的超集一定不频繁”来剪枝，避免暴力枚举所有组合。