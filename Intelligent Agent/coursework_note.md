船队调度
什么船拉什么货   amount权重

cost函数!!!
硬性运输成本 + 未来距离成本 + 对手因素（风险成本）

Bundling multiple cargoes on a single
vessel can reduce the total cost, potentially creating a superadditive
valuation compared to shipping each cargo independently. Never-
theless, allocating the resulting savings across individual bids risks
losses whenever only a subset of the bundled contracts is won

最终每一个trade 的bid基于什么权重

每次基于所有trade规划航线，然后遍历去掉每个trade查看哪几个使得最后的cost下降的最少然后去掉这几个点


航线调度
propose_schedules
find_schedules


优化方向：
船型选择，超参数调优，多线程特判


## 船型选择上：
vlcc船的消耗数不同于其他两个
![[Pasted image 20251125163908.png]]
这两个消耗数相同：
![[Pasted image 20251125163943.png]]

## 三种船型的完整信息对比

从这个代码文件中，你可以获得以下关于 **Suezmax**、**Aframax**、**VLCC** 三种船型的信息：

### 一、基本参数对比表

|参数|Suezmax|Aframax|VLCC|说明|
|---|---|---|---|---|
|**capacity**|145,000|100,000|285,000|载货容量（单位：吨或桶）|
|**loading_rate**|41,428|28,571|81,428|装载速率 = capacity/3.5|
|**speed**|14|14|14|航行速度（节）|
|**cargo_type**|Oil|Oil|Oil|货物类型|

### 二、燃油消耗参数对比

#### 1️⃣ 固定消耗（单位：吨/天）

| 消耗类型                      | Suezmax | Aframax | VLCC   | 差异说明    |
| ------------------------- | ------- | ------- | ------ | ------- |
| **idle_consumption**      | 7.733   | 7.733   | 7.13   | 闲置时燃油消耗 |
| **loading_consumption**   | 12.23   | 12.23   | 15.53  | 装载时燃油消耗 |
| **unloading_consumption** | 79.5    | 79.5    | 134.37 | 卸载时燃油消耗 |

**观察**：

- Suezmax 和 Aframax 的固定消耗**完全相同**
- VLCC 因为吨位大，装卸消耗明显更高

#### 2️⃣ 载货航行消耗曲线（Laden Consumption）

|参数|Suezmax|Aframax|VLCC|公式作用|
|---|---|---|---|---|
|**laden_base**|0.0473|0.0473|0.5503|基础系数|
|**laden_speed_power**|2.6356|2.6356|2.1920|速度指数|

**计算公式**：

python

laden_consumption = laden_base × speed^laden_speed_power × (1/24)

**Suezmax/Aframax**（示例，速度14节）：

python

消耗 = 0.0473 × 14^2.6356 ÷ 24  
     = 0.0473 × 245.8 ÷ 24  
     ≈ 0.484 吨/小时 ≈ 11.6 吨/天

**VLCC**（示例，速度14节）：

python

消耗 = 0.5503 × 14^2.19201 ÷ 24  
     = 0.5503 × 109.9 ÷ 24  
     ≈ 2.52 吨/小时 ≈ 60.5 吨/天

#### 3️⃣ 空驶航行消耗曲线（Ballast Consumption）

|参数|Suezmax|Aframax|VLCC|公式作用|
|---|---|---|---|---|
|**ballast_base**|0.0195|0.0195|0.1493|基础系数|
|**ballast_speed_power**|2.9185|2.9185|2.3268|速度指数|

**计算公式**：

python

ballast_consumption = ballast_base × speed^ballast_speed_power × (1/24)

### 三、关键差异总结

#### 1. **容量差异**

VLCC (285,000) > Suezmax (145,000) > Aframax (100,000)  
大型           中大型            中型

#### 2. **燃油效率差异**

**Suezmax vs Aframax**：

- ✅ **消耗曲线完全相同**（共用 
    
    ```
    default_suezmax_aframax_laden_balast_consumption()
    ```
    
    ）
- ✅ **固定消耗完全相同**
- 🔑 **唯一差异：capacity** (145k vs 100k)

**结论**：Suezmax 更经济（单位成本更低，因为容量大但消耗相同）

**VLCC vs Suezmax/Aframax**：

- ❌ **消耗曲线不同**（使用 
    
    ```
    default_vlcc_laden_balast_consumption()
    ```
    
    ）
- ❌ **载货消耗更高**（laden_base: 0.5503 vs 0.0473，约**11倍**！）
- ❌ **空驶消耗更高**（ballast_base: 0.1493 vs 0.0195，约**7.7倍**）
- ❌ **装卸消耗更high**（loading: 15.53 vs 12.23, unloading: 134.37 vs 79.5）

### 四、实际应用中的影响

#### 在你的优化代码中可以利用这些信息：

python

### 1. 根据订单量选择船型  
if trade.amount <= 100000:  
    prefer_aframax  # 小订单，避免大船空驶  
elif trade.amount <= 145000:  
    prefer_suezmax  # 中等订单，最经济  
else:  
    prefer_vlcc     # 大订单，必须用VLCC  
  
### 2. 计算真实成本时考虑船型差异  
def _calculate_real_cost(vessel, schedule):  
    # Aframax和Suezmax计算相同  
    # VLCC需要特别注意，燃油消耗大得多  
    if vessel.capacity == 285000:  # VLCC  
        cost_multiplier = 11  # 载货消耗约11倍  
    else:  
        cost_multiplier = 1  
      
    # ... 计算逻辑

#### 3. 战略意义

python

### Suezmax 性价比分析  
Aframax容量:  100,000  消耗: 11.6吨/天  
Suezmax容量:  145,000  消耗: 11.6吨/天  ← 容量+45%，消耗不变！  
  
每吨货物成本:  
Aframax: 11.6 ÷ 100,000 = 0.000116  
Suezmax: 11.6 ÷ 145,000 = 0.000080  ← 便宜31%！  
  
### VLCC 只适合超大订单  
VLCC容量: 285,000  消耗: 60.5吨/天  
每吨货物成本: 60.5 ÷ 285,000 = 0.000212  ← 最贵！  
  
只有订单量>145,000时才值得用VLCC

### 五、代码中获取这些信息的方法

python

### 获取船只信息  
vessel = get_vessel_suezmax("MyShip")  
  
### 可访问的属性  
vessel.speed                           # 14  
vessel.cargo_capacities[0].capacity    # 145000  
vessel.cargo_capacities[0].loading_rate # 41428  
vessel.cargo_capacities[0].cargo_type  # "Oil"  
  
### 燃油消耗  
engine = vessel.propelling_engine  
engine.idle_consumption                # 7.733/24  
engine.loading_consumption             # 12.23/24  
engine.unloading_consumption           # 79.5/24  
  
### 消耗曲线  
engine.laden_consumption_rate.base     # 0.0473  
engine.laden_consumption_rate.speed_power # 2.6356  
engine.ballast_consumption_rate.base   # 0.0195  
engine.ballast_consumption_rate.speed_power # 2.9185  
  
### 计算方法  
travel_time = vessel.get_travel_time(distance)  
laden_fuel = vessel.get_laden_consumption(time, speed)  
ballast_fuel = vessel.get_ballast_consumption(time, speed)  
loading_time = vessel.get_loading_time(cargo_type, amount)

### 总结

从这个文件你可以获得**7大类信息**：

1. ✅ **基本容量**：145k / 100k / 285k
2. ✅ **装载速率**：capacity / 3.5
3. ✅ **航行速度**：14节（三种船相同）
4. ✅ **闲置消耗**：7.733 / 7.733 / 7.13 吨/天
5. ✅ **装卸消耗**：12.23/79.5 vs 15.53/134.37 吨/天
6. ✅ **载货消耗曲线**：base × speed^power（VLCC高11倍）
7. ✅ **空驶消耗曲线**：base × speed^power（VLCC高7.7倍）

**核心发现**：Suezmax 是性价比之王！🚢

## 超参数调优

## 边际成本不是真实成本而是简化为时间成本
这个可能存在时间短但实际燃油消耗大的问题。
![[Pasted image 20251125165215.png]]
非常好的思考！你理解得非常准确。让我详细分析如果改成真实成本计算会有什么影响。

### 一、好处分析

### ✅ 好处1：更准确的决策

#### 当前问题（时间成本）：

python

### 场景：两艘船，不同燃油效率  
快速船（高油耗）:  
  时间增量 = 10小时  
  真实成本增量 = 500单位燃油 ❌ 实际更贵  
  
慢速船（低油耗）:  
  时间增量 = 15小时  
  真实成本增量 = 300单位燃油 ✅ 实际更便宜  
  
当前算法会选快速船（10 < 15）→ 错误决策！

#### 改进后（真实成本）：

python

快速船: 成本增量 = 500  
慢速船: 成本增量 = 300 ✅ 正确选择

### ✅ 好处2：考虑船只特性差异

python

# 不同船只的燃油消耗率可能差异很大  
新船: 燃油效率高，但速度快  
老船: 燃油效率低，但速度慢  
  
### 真实成本计算能反映这些差异  
vessel.get_laden_consumption(time, speed)  # 考虑具体船只  
vessel.get_loading_consumption(time)       # 考虑装卸效率

### ✅ 好处3：更准确的报价

#### 当前流程：

python

### 在 _generate_greedy_solution 中用时间成本  
incremental_cost = time_after - time_before  
  
### 但在报价时用真实成本（第83行）  
absolute_cost = self._calculate_absolute_cost_cached(trade)  
                ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^  
                这里计算的是真实燃油成本  
  
### 不一致！预调度用时间，报价用成本

#### 改进后：

python

### 统一使用真实成本  
incremental_cost = fuel_cost_after - fuel_cost_before  
absolute_cost = 真实燃油成本  
  
### 一致性更好，报价更准确

### ✅ 好处4：空驶 vs 载货的成本差异

python

### 真实场景：空驶和载货燃油消耗不同  
空驶: vessel.get_laden_consumption(time, speed) * 0.7  # 轻载  
载货: vessel.get_laden_consumption(time, speed) * 1.0  # 重载  
  
### 时间成本无法区分，真实成本可以

### 二、坏处分析

### ❌ 坏处1：计算复杂度大幅增加

#### 当前实现（时间成本）：

python

def _calculate_schedule_total_cost(self, vessel, schedule):  
    total_time = schedule.completion_time()  # O(1) 或 O(n)，很快  
    return total_time  
  
### 在搜索过程中调用次数  
插入点数量: 假设每个schedule有10个插入点  
订单数量: 20个  
船只数量: 5艘  
插入组合: 每个订单约50种插入方式  
  
每次迭代计算次数 ≈ 20 × 5 × 50 = 5000次  
总调用次数（15次迭代）= 5000 × 15 = 75,000次  
  
当前每次耗时: ~0.0001秒  
总耗时: 7.5秒

### 改成真实成本：

python

def _calculate_schedule_total_cost(self, vessel, schedule):  
    total_cost = 0  
    # 遍历schedule中的每个航段  
    for leg in schedule.legs:  
        # 计算距离  
        distance = self.headquarters.get_network_distance(leg.from, leg.to)  
          
        # 计算时间  
        travel_time = vessel.get_travel_time(distance)  
          
        # 计算燃油消耗（区分空驶/载货）  
        if leg.is_laden:  
            fuel = vessel.get_laden_consumption(travel_time, vessel.speed)  
        else:  
            fuel = vessel.get_ballast_consumption(travel_time, vessel.speed)  
          
        total_cost += fuel  
          
        # 装卸成本  
        if leg.has_loading:  
            loading_time = vessel.get_loading_time(leg.cargo_type, leg.amount)  
            total_cost += vessel.get_loading_consumption(loading_time)  
            total_cost += vessel.get_unloading_consumption(loading_time)  
      
    return total_cost  
  
### 假设每个schedule平均5个航段  
每次调用耗时: ~0.001秒（10倍增加）  
总耗时: 75秒！  ← 10倍性能下降！

### ❌ 坏处2：可能需要更复杂的缓存

python

### 时间成本：简单  
cache_key = id(schedule)  
cached_time = schedule.completion_time()  # schedule对象自己可能有缓存  
  
### 真实成本：复杂  
### 每次schedule变化都需要重新计算，缓存失效频繁  
### schedule是动态构建的，难以有效缓存

### ❌ 坏处3：边际收益可能有限

**关键洞察：我们只需要相对排序，不需要绝对精确值**

python

### 测试场景：100个候选方案  
  
时间成本排序:  
方案A: 100小时 (实际燃油: 500)  
方案B: 105小时 (实际燃油: 480)   
方案C: 110小时 (实际燃油: 600)  
排序: A < B < C  
  
真实成本排序:  
方案B: 480 ✅ 最优  
方案A: 500  
方案C: 600  
排序: B < A < C  
  
### 如果大部分情况下排序一致性> 90%  
### 那么10倍的计算时间可能不值得

### ❌ 坏处4：代码复杂度增加

python

### 需要处理各种边界情况  
if schedule.legs:  
    for leg in schedule.legs:  
        # 判断是否空驶  
        # 判断是否有装卸  
        # 处理多种货物类型  
        # 考虑船只容量限制  
        # ...  
  
### vs 当前简洁的一行  
total_time = schedule.completion_time()

### 三、性能测试对比

让我估算一下实际影响：

Analyzed

groupn_optimized.py#L260-275

### 性能影响估算：

python

### 搜索复杂度  
for trade in trades:              # 20个订单  
    for vessel in fleet:          # 5艘船  
        for pick in insertions:   # 平均10个pickup点  
            for drop in drops:    # 平均10个drop点  
                # 这里调用两次 _calculate_schedule_total_cost  
                cost_before = ...  
                cost_after = ...  
  
### 总调用次数（单次迭代）  
20 × 5 × 10 × 10 × 2 = 20,000次  
  
### 15次并行迭代（4线程）  
实际执行: 15 ÷ 4 ≈ 4轮  
总调用次数: 20,000 × 15 = 300,000次  
  
### 性能对比  
┌─────────────────┬──────────────┬──────────────┐  
│                 │  时间成本    │  真实成本    │  
├─────────────────┼──────────────┼──────────────┤  
│ 单次调用耗时    │  0.0001秒    │  0.001秒     │  
│ 总调用次数      │  300,000次   │  300,000次   │  
│ 总耗时(串行)    │  30秒        │  300秒       │  
│ 总耗时(4线程)   │  7.5秒       │  75秒        │  
└─────────────────┴──────────────┴──────────────┘

### 四、建议的折衷方案

### 方案1：混合策略

python

def _calculate_schedule_total_cost(self, vessel, schedule, precise=False):  
    """  
    precise=False: 快速模式，用时间近似  
    precise=True: 精确模式，计算真实成本  
    """  
    if not precise:  
        return schedule.completion_time()  
    else:  
        return self._calculate_precise_fuel_cost(vessel, schedule)  
  
### 在搜索阶段用快速模式  
cost_before = self._calculate_schedule_total_cost(vessel, schedule, precise=False)  
  
### 在最终选择top k方案时用精确模式  
for plan in top_k_candidates:  
    plan['real_cost'] = self._calculate_schedule_total_cost(  
        vessel, plan['schedule'], precise=True  
    )

### 方案2：采样策略

python

### 前80%的搜索用时间成本（快速探索）  
### 后20%的搜索用真实成本（精确优化）  
if iteration < self.M_SEARCH_ITERATIONS * 0.8:  
    use_time_cost = True  
else:  
    use_time_cost = False

### 方案3：缓存优化

python

### 如果能有效缓存航段成本  
@lru_cache(maxsize=10000)  
def _get_leg_cost(self, vessel_id, from_loc_id, to_loc_id, is_laden):  
    # 计算单个航段的燃油成本  
    ...  
  
### 那么真实成本计算也能很快

### 五、总结

### 好处：

|优势|重要性|说明|
|---|---|---|
|更准确的决策|⭐⭐⭐⭐|考虑船只差异，选择真正更便宜的方案|
|一致性|⭐⭐⭐|搜索和报价使用同一成本体系|
|真实性|⭐⭐⭐|反映实际业务成本|

### 坏处：

|劣势|影响程度|说明|
|---|---|---|
|性能下降|⭐⭐⭐⭐⭐|10倍计算时间，严重影响响应速度|
|复杂度增加|⭐⭐⭐|代码更复杂，bug风险增加|
|边际收益递减|⭐⭐⭐⭐|排序结果可能差异不大|

### 我的建议：

**在你的优化版代码中**：

- ✅ **保持时间成本用于搜索阶段** - 性能优先
- ✅ **真实成本用于最终报价** - 已经这样做了（第83行）
- 🤔 **可选：在top k方案选择时用真实成本重新排序** - 平衡精度和性能

**最佳实践**：

python

### 搜索时：快速探索，时间成本  
### 报价时：精确计算，真实成本  
### 两全其美！

这是一个经典的**速度 vs 精度**的权衡问题！🎯