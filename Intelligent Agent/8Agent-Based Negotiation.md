你好！我是你的课程助教。这份讲义涵盖了**智能体协商 (Agent-Based Negotiation)** 的核心内容，从基础的冲突解决概念，到形式化的效用建模，再到具体的协商策略。

这份材料非常系统，分为三个部分：**协议 (Protocols)**、**形式化模型与公平性 (Formalisation & Fairness)**、以及**策略 (Strategies)**。

以下是为你整理的详细学习指南与复习笔记。

---

### 📖 1. 课程概览 (Executive Summary)

本课程主要探讨在多智能体系统（Multi-Agent Systems）中，自利（Self-interested）智能体如何通过**协商**来解决利益冲突。

- **Part 1** 介绍了协商在冲突解决中的地位，以及几种经典的**协商协议**（如最后通牒博弈、轮流出价等）。
    
- **Part 2** 引入了数学符号，重点讲解**效用函数 (Utility Function)**、**帕累托最优 (Pareto Efficiency)** 以及四种**公平性 (Fairness)** 指标（包括纳什议价解 NBS）。
    
- **Part 3** 讨论了在不确定性环境下的**启发式策略**（如基于时间的让步策略 Boulware/Conceder）和对手建模。
    

---

### 📝 2. 详细内容梳理

#### Part 1: 协商基础与协议 (Negotiation Protocols)

1.1 冲突解决的三种机制 [Slide 5, 23-28]

当智能体目标不同且存在利益冲突时，可通过以下方式解决：

- **拍卖 (Auctions):** 适用于稀缺资源分配（如广告位、云资源）。特点是规则明确，通常需要第三方（拍卖师），利用竞争 [Slide 6, Transcript 04:41]。
    
- **投票 (Voting):** 适用于群体决策/社会选择（如去哪吃饭、选举）。从有限选项中选出一个 [Slide 7, Transcript 07:09]。
    
- **协商 (Negotiation):** 规则（协议）定义了交互方式，但在协议内更加灵活。通常是双边（Bilateral）的，支持复杂的**多议题 (Multi-issue)** 协议 [Slide 8, Transcript 08:52]。
    

**1.2 协商环境 [Slide 9, 65-67]**

- **单议题 (Single-issue):** 也就是分蛋糕 (Distributive bargaining)。通常是零和博弈（Win-Lose），例如只谈价格，你多赚我就少赚。
    
- **多议题 (Multi-issue):** 也就是整合性谈判 (Integrative bargaining)。引入更多议题（如保修期、服务质量），可实现双赢 (Win-Win)。
    

1.3 四种核心协商协议 [Slide 10, 73]

这是 Part 1 的重点，规定了谁在什么时候能说什么。

1. **最后通牒博弈 (Ultimatum Game/Take-it-or-leave-it) [Slide 14]:**
    
    - Agent 1 提议分蛋糕的比例，Agent 2 只能接受或拒绝。
        
    - 如果拒绝，两人都得不到任何东西（蛋糕被扔掉）。
        
    - **理性分析:** 如果 Agent 2 是理性的，哪怕只给一点渣（只要 >0）也会接受。因此 Agent 1 占有**先发优势** [Slide 17, Transcript 22:17]。
        
2. **轮流出价协议 (Alternating Offers) [Slide 18]:**
    
    - 双方轮流出价，直到达成一致或达到截止时间（Deadline）。
        
    - **关键点:** 如果有固定截止轮次，**最后一轮**其实就变成了“最后通牒博弈”。理性的 Agent 可能会拖到最后一轮利用先发优势 [Slide 20, Transcript 25:08]。
        
3. **单调让步协议 (Monotonic Concession) [Slide 21]:**
    
    - 双方同时出价。如果不匹配，下一轮**必须**至少有一方让步（Concede），否则谈判破裂。这强迫双方不断向中间靠拢。
        
4. **切分与选择 (Divide and Choose) [Slide 22]:**
    
    - 适用于连续资源（如土地、蛋糕）。
        
    - 规则：Agent 1 切蛋糕，Agent 2 先选。
        
    - **策略:** Agent 1 为了防止自己拿到小份，一定会切得非常均匀（50/50）。这是一种无需第三方干预的公平协议 [Slide 24, Transcript 29:32]。
        

---

#### Part 2: 形式化模型与公平性 (Formalisation & Fairness)

**2.1 效用函数 (Utility Functions) [Slide 28]**

- **序数偏好 (Ordinal):** 只知道 $A$ 比 $B$ 好 ($U(A) > U(B)$)，不知道好多少。
    
- **基数偏好 (Cardinal):** 有具体数值 ($U(A)=0.7, U(B)=0.5$)。
    
- **协议空间 (Agreement Space):** 买家最高出价 ($v$) 与卖家底价 ($r$) 之间的重叠区域。只有在此区域内才可能达成协议 [Slide 30]。
    

2.2 时间压力 (Time Pressure) [Slide 32, Transcript 08:28]

时间就是金钱，蛋糕会随时间“融化”。

- **折扣因子 (Discount Factor, $\delta$):** $U_i^t = U_i \cdot \delta_i^t$ (其中 $\delta < 1$)。
    
- 时间越久，效用越低。
    

2.3 多议题效用 (Multi-Issue Utility) [Slide 33, 255]

通常使用加权累加函数 (Weighted Additive Utility Function):

$$U_i(o) = \sum_{j} w_{i,j} \cdot U_{i,j}(o_j)$$

- **例子:** 两个蛋糕，Agent 1 偏好蛋糕1（权重0.7），Agent 2 偏好蛋糕2（权重0.7）。
    
- **关键:** 这种差异创造了双赢空间。如果Agent 1拿走全部蛋糕1，Agent 2拿走全部蛋糕2，双方效用都高。
    

**2.4 帕累托最优 (Pareto Efficiency) [Slide 37, Transcript 19:17]**

- **定义:** 一个协议是帕累托最优的，当且仅当**不存在**另一个协议，能在**不降低**某一方效用的前提下，提高另一方的效用。
    
- **帕累托前沿 (Pareto Frontier):** 所有帕累托最优点的集合（图表中的右上边界）。
    

2.5 四种公平性标准 (Fairness Concepts) [Slide 39-41, Transcript 22:01]

假设效用分别为 $U_1$ 和 $U_2$：

1. **功利主义 (Utilitarian Social Welfare):** 最大化总和 $\max(U_1 + U_2)$。
    
2. **平均主义 (Egalitarian Social Welfare):** 最大化最小值 $\max(\min(U_1, U_2))$。照顾最惨的一方。
    
3. **纳什议价解 (Nash Bargaining Solution, NBS):** 最大化超出“不协议点”的乘积。
    
    - 公式: $\max (U_1(o) - U_1(dis)) \cdot (U_2(o) - U_2(dis))$。
        
    - **特点:** 满足个体理性、帕累托最优、且与效用单位缩放无关（Scale Invariant）[Slide 40]。
        
4. **无妒忌 (Envy-freeness):**
    
    - **定义:** 没有任何一方想要对方分到的资源 [Slide 41]。
        
    - **测试方法 (Swap Test):** 如果把你得到的跟我得到的**互换**，我的效用变高了，那我就是妒忌你。如果互换后效用没变高，就是无妒忌 [Slide 44]。
        

---

#### Part 3: 协商策略 (Negotiation Strategies)

**3.1 两种研究路径 [Slide 49]**

- **博弈论 (Game Theoretic):** 假设完全理性、完全信息。寻找纳什均衡。
    
- **启发式 (Heuristic):** 假设信息不完全、有限理性。通过经验法则（如“我也让一点”）来行动。适合复杂环境。
    

3.2 让步策略 (Concession Strategies) [Slide 51-52, Transcript 04:20]

决定在时间 $t$ 我应该把目标效用定为多少。

公式通常为：$U_{target}(t) = U_{min} + (1 - F(t))(U_{max} - U_{min})$

其中 $F(t) = (\frac{t}{T_{max}})^{1/\beta}$。

- **Boulware 策略 ($\beta < 1$):** **强硬**。直到最后期限前几乎不让步，最后一刻才妥协。
    
- **Conceder 策略 ($\beta > 1$):** **软弱/妥协者**。一开始就快速让步。
    
- **Linear 策略 ($\beta = 1$):** 线性匀速让步。
    

**3.3 以牙还牙 (Tit-for-Tat) [Slide 53]**

- 观察对手上一轮让步了多少效用，下一轮我也让步相同的量。
    
- 难点：需要估算对手的效用函数才能知道他“让步了多少”。由于没办法知道，所以这个策略是基于自己的utility进行计算的（例如对手更新了他的offer，我计算发现这让我的utility提高了x。这个x就被我认定为对手的concession，因此下一次我的offer要让步x，这次的x要基于我自己的utility函数计算）
    

**3.4 不确定性 (Uncertainty) [Slide 56-57]**

- **对手不确定性:** 不知道对手的 $U$。可以通过观察其出价来学习（例如：对手在某个议题上让步很快，说明该议题对他权重低）。
    
- **自身偏好不确定性:** Agent 可能不知道用户到底喜欢什么。需要**偏好获取 (Preference Elicitation)**，但这有认知成本（Cognitive Cost），不能问用户太多问题。
    

---

### 🔍 3. 教师重点复述 (Teacher's Highlights)

_这部分包含了讲师在转写中反复强调或带有解释性的重点内容。_

1. **冲突并非坏事:** 老师特别提到冲突在多智能体系统中是常态（different preferences），并不是指“战争”，而是指利益不一致（Transcript 01:23）。
    
2. **最后通牒效应 (The "Last Round" Effect):** 在讲解轮流出价协议时，老师强调如果有截止日期，**最后一轮**本质上就是最后通牒博弈。理性主体会利用这一点，导致前面的轮次可能无效（Transcript 25:08）。
    
3. **NBS $\neq$ 纳什均衡:** 老师明确指出，Nash Bargaining Solution (NBS) 和 Nash Equilibrium 是两个完全不同的概念，虽然都是纳什提出的，不要混淆（Transcript 25:05）。
    
4. **单议题 vs 多议题:** 老师反复强调单议题通常是零和的（Win-Lose），而多议题才是协商最有趣的地方，因为可以创造**双赢 (Mutually beneficial/Win-Win)**（Transcript 12:19, 19:22）。
    
5. **"Money left on the table":** 如果协议不是帕累托最优的，老师将其比喻为“把钱留在桌子上没拿走”，即双方本可以达成更好的协议但没达成（Transcript 32:08）。
    

---

### 🧮 4. 核心术语与公式速查

| **术语 (CN/EN)**               | **定义/公式**                       | **直观解释**                             |
| ---------------------------- | ------------------------------- | ------------------------------------ |
| **效用函数 (Utility Function)**  | $U(o)$                          | 把一个结果映射为一个数字，数字越大越满意。                |
| **协议空间 (Agreement Space)**   | $[r_{seller}, v_{buyer}]$       | 双方都能接受的区域（买家出价 $\le$ 卖家底价）。          |
| **折扣因子 (Discount Factor)**   | $U^t = U \cdot \delta^t$        | 模拟“冰淇淋融化”，越晚达成协议，价值越低。               |
| **帕累托最优 (Pareto Efficient)** | 无公式，指一种状态                       | 想要让一个人更爽，就必须让另一个人不爽的状态。也就是榨干了所有合作红利。 |
| **纳什议价解 (NBS)**              | $\max \Pi (U_i - U_{dis})$      | 公平性指标，最大化双方“净收益”的**乘积**。             |
| **Boulware 策略**              | $\beta < 1$ 的时间函数               | “死硬派”，死守高价，直到死线才松口。                  |
| **无妒忌 (Envy-free)**          | $U_1(alloc_1) \ge U_1(alloc_2)$ | 我不想要你盘子里的东西，我觉得我自己的更好。               |

---

### 💻 5. 伪代码：基于时间的让步策略

基于 Slide 51-52 的公式，这是实现一个协商 Agent 核心逻辑的 Python 示例。

Python

```
import math

class NegotiatingAgent:
    def __init__(self, u_max, u_min, deadline, beta):
        """
        u_max: 理想的最高效用 (开始时的目标)
        u_min: 最低可接受效用 (底线/保留价格)
        deadline: 协商的最大轮次或时间
        beta: 策略参数 (<1: 强硬 Boulware, >1: 妥协 Conceder)
        """
        self.u_max = u_max
        self.u_min = u_min
        self.deadline = deadline
        self.beta = beta

    def get_target_utility(self, current_time):
        """计算当前时间点的目标效用"""
        if current_time >= self.deadline:
            return self.u_min # 到时间了，给出底线
        
        # 计算时间比例 (0 到 1)
        time_ratio = min(current_time, self.deadline) / self.deadline
        
        # 计算多项式因子 F(t)
        # Slide 51 公式: F(t) = (t / T_max)^(1/beta)
        # 注意：Slide上的公式是 (1/beta)，这决定了曲线形状
        f_t = math.pow(time_ratio, 1.0 / self.beta)
        
        # 计算目标效用
        # U(t) = U_min + (1 - F(t)) * (U_max - U_min)
        target_utility = self.u_min + (1 - f_t) * (self.u_max - self.u_min)
        
        return target_utility

# === 使用示例 ===
# 一个强硬的代理 (Boulware)，beta = 0.1
tough_agent = NegotiatingAgent(u_max=1.0, u_min=0.5, deadline=10, beta=0.1)

# 一个容易妥协的代理 (Conceder)，beta = 10
soft_agent = NegotiatingAgent(u_max=1.0, u_min=0.5, deadline=10, beta=10.0)

print("Time | Tough Agent Target | Soft Agent Target")
for t in range(0, 11):
    u_tough = tough_agent.get_target_utility(t)
    u_soft = soft_agent.get_target_utility(t)
    print(f" {t:2d}  |       {u_tough:.3f}        |       {u_soft:.3f}")

# 输出解释:
# Tough Agent 会在大部分时间保持接近 1.0，直到最后才掉到 0.5
# Soft Agent 会在 t=1 时就迅速掉到接近 0.5
```

---

### 🎯 6. 复习与思考题 (Review Exercises)

例题 1: 公平性计算 (参考 Slide 45)

假设有两个 Offer，效用如下：

- Offer A: $U_1=10, U_2=0$ (不协议效用均为0)
    
- Offer B: $U_1=5, U_2=5$
    

**问：** 计算 utilitarian sum 和 Nash product。

- **解答:**
    
    - **Utilitarian:** Offer A = $10+0=10$; Offer B = $5+5=10$。两者在功利主义下一样好。
        
    - **Nash Product:** Offer A = $(10-0)\times(0-0)=0$; Offer B = $(5-0)\times(5-0)=25$。
        
    - **结论:** Offer B 更符合纳什公平解。
        

思考题 2: 无妒忌测试 (Envy-free)

假设 Agent 1 分到 70% 蛋糕，Agent 2 分到 30%。

- Agent 1 也是这么认为的 ($U_1 = 0.7$)。
    
- Agent 2 也是这么认为的 ($U_2 = 0.3$)。
    
- **问:** 这个分配是 Envy-free 的吗？
    
- **提示:**
    
    - Agent 1 拥有 0.7，Agent 2 拥有 0.3。Agent 1 肯定不羡慕 Agent 2。
        
    - Agent 2 拥有 0.3，但他看 Agent 1 拥有 0.7。如果互换，Agent 2 能得到 0.7。因为 $0.7 > 0.3$，所以 Agent 2 **妒忌** Agent 1。
        
    - **结论:** 不是 Envy-free。
        

下一步建议：

你可以尝试计算讲义 Slide 46 中提到的练习：验证 Offer (1.0, 0.2) 为什么是 Envy-free 的，而 (1.0, 0.3) 不是。这需要你代入 Slide 34 的具体权重公式进行计算。如果你需要我带你一步步算这个，请告诉我！

## envy-free易错点
![[Pasted image 20260119105348.png]]这里千万不要直接交换o1 and o2，以offer6为例，而是要分析一下原来o1是0.8，交换完相当于player1得到0.2（原来是player2的分配结果）。