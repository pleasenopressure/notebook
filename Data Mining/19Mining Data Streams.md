# COMP6237 Data Mining — Mining Data Streams 最终版复习笔记

本节课是 **COMP6237 – Mining Data Streams（数据流挖掘）**，讲义说明其内容基于 _Mining Massive Datasets_ 第 4 章，核心主题包括：**数据流处理模型（stream processing model）**、**从数据流中采样（sampling from streams）**、**用 Bloom filter 过滤数据流（filtering streams）**、以及 **Flajolet-Martin algorithm 估计数据流中不同元素数量（counting distinct elements）**。

---

## 0. 本章总览：这节课到底在解决什么问题？

### 0.1 传统 Data Mining vs Data Stream Mining

以前课程里大多数算法默认：

**数据已经存在数据库中。**

也就是说：

```text
你有一个完整的数据集
你可以反复访问它
你可以多次扫描它
你可以排序、聚类、训练模型
你可以需要的时候再回来查它
```

例如：

```text
customer.csv
transactions.db
document-term matrix
ratings matrix
```

这些数据是“静态的”。

但是 **Data Stream Mining（数据流挖掘）** 面对的是完全不同的场景：

```text
数据不断到达
速度很快
总量可能无限大
内存无法全部保存
如果当下不处理，数据可能永远丢失
```

讲义第 3 页明确强调：在数据流场景中，数据以 stream 形式到达，如果不能立即处理，就会丢失，而且不能全部保存在 active storage 中。

---

## 1. Data Stream Processing Model 数据流处理模型

### 1.1 基本结构

数据流系统可以抽象成下面的结构：

```text
Input Streams  --->  Stream Processor  --->  Output Streams
                         |
                         |
              Limited Working Storage
                         |
                  Archival Storage
```

其中：

- **Input streams（输入流）**：持续到来的数据，例如传感器读数、搜索查询、网络包、邮件。
    
- **Stream processor（流处理器）**：在数据到达时立即处理。
    
- **Output streams（输出流）**：处理后的结果，例如报警、过滤后的邮件、统计结果。
    
- **Limited working storage（有限工作内存）**：当前算法能快速使用的内存，例如 RAM。
    
- **Archival storage（归档存储）**：磁盘或长期存储，但也可能不够大，不能无限保存全部数据。
    
- **Standing queries（长期查询 / 固定查询）**：系统提前知道要回答的问题。
    
- **Ad-hoc queries（临时查询 / 即席查询）**：系统事先不知道用户会问什么。
    

---

## 2. Motivation and Applications 动机与应用场景

### 2.1 为什么数据流挖掘很重要？

因为现实世界中很多数据不是一次性给你的，而是持续产生的。

讲义给出的典型例子包括：

```text
1. Sensor data
2. Image data
3. Internet traffic
4. Search engine query streams
```

讲义第 4 页举例说，单个 ocean sensor 每 0.1 秒产生 4 bytes 数据，一天约 3.5MB，这本身可管理；但如果有 1 million 个这样的传感器，一天就会达到约 3.5TB。图像数据、卫星数据、监控摄像头、网络流量、搜索引擎查询也都是类似的数据流场景。

---

### 2.2 例子 1：Ocean Sensor + GPS

假设一个海洋传感器不断发送海面高度或温度数据：

```text
每 0.1 秒发送一次
每次 4 bytes
```

单个传感器：

```text
4 bytes / 0.1 sec
= 40 bytes / sec
≈ 3.5 MB / day
```

这还可以存。

但如果有：

```text
1,000,000 sensors
```

那么每天数据量大约是：

```text
3.5 MB/day * 1,000,000 = 3.5 TB/day
```

这就不能用“全部存下来再慢慢分析”的传统方式处理。

---

### 2.3 例子 2：Surveillance Cameras 监控摄像头

讲义提到 London 有大量 surveillance cameras。视频数据比普通数值传感器数据更大：

```text
每个摄像头每秒几十帧
每帧是图片
多个摄像头并行产生视频流
```

如果所有视频都完整存储并实时分析，成本非常高。

因此实际系统通常需要：

```text
只保留最近窗口
只保存异常事件
只输出检测结果
只抽样部分数据
```

---

### 2.4 例子 3：Internet Traffic 网络流量

网络交换机、路由器会不断看到 packet stream：

```text
(source IP, destination IP, port, protocol, timestamp, size, ...)
```

我们可能想实时检测：

```text
是否有 DDoS attack
是否有异常流量
是否有路由拥塞
是否某个 IP suddenly becomes suspicious
```

但网络包到达速度非常快，不可能把所有 packet 都完整保存并做多轮分析。

---

### 2.5 例子 4：Search Engine Queries 搜索查询流

搜索引擎不断收到查询：

```text
(user, query, time)
```

可以用这些查询分析：

```text
热点趋势
流感传播趋势
用户行为变化
广告策略
缓存策略
异常搜索行为
```

但是查询是持续到来的，而且量巨大。

---

## 3. 为什么 Mining Data Streams 很难？

### 3.1 难点 1：数据到达速度快

数据可能以很高频率到达：

```text
每秒几千条
每秒几百万条
甚至更高
```

算法不能太慢。

很多传统算法需要：

```text
多次扫描数据
排序全部数据
构造完整矩阵
存储所有历史记录
```

这些在数据流场景下都不现实。

---

### 3.2 难点 2：内存有限

数据流可能无限长：

```text
x1, x2, x3, x4, ...
```

如果每个元素都存储，内存迟早爆掉。

所以数据流算法通常要求：

```text
space << total stream size
```

也就是说，算法使用的内存必须远远小于整个数据流大小。

---

### 3.3 难点 3：只能看一次

很多数据流算法采用 **one-pass model（单遍扫描模型）**：

```text
每个元素来了以后，只能处理一次
不能回头看
```

这与数据库场景完全不同。

数据库里可以：

```text
select *
where ...
join ...
group by ...
scan many times
```

数据流里则更像：

```text
数据来了
你现在处理
处理完就过去了
```

---

### 3.4 难点 4：精确答案经常太贵

在很多数据流问题里，精确答案需要保存太多信息。

因此课程强调：

**approximate answer（近似答案）通常比 exact answer（精确答案）更现实。**

讲义第 6 页说明，对于 ad-hoc queries，通常会使用 sliding window，并且近似答案往往比精确答案更可取；这些技术通常依赖 clever use of hashing。

---

## 4. Standing Queries vs Ad-hoc Queries

这是本章的第一个核心概念。

---

## 4.1 Standing Queries 固定查询 / 长期查询

### 4.1.1 定义

**Standing query（固定查询）** 是系统提前知道的问题。

也就是说：

```text
流处理器在设计时就知道要监控什么
因此可以专门为这个查询设计处理逻辑
```

讲义第 5 页定义 standing queries 为 predetermined queries，并说明可以为这类查询定制 stream processor。

---

### 4.1.2 例子：温度传感器

假设 ocean sensor 持续发送温度：

```text
23.1, 24.7, 25.2, 24.9, 26.0, ...
```

#### 查询 1：温度超过 25 度就报警

```text
Alert whenever temperature > 25
```

处理方式非常简单：

```text
for each new temperature x:
    if x > 25:
        raise alert
```

只需要看当前元素。

不需要存历史数据。

---

#### 查询 2：计算 all-time average temperature

要计算所有历史温度的平均值：

块级公式源码：

$$  
\text{average} = \frac{\text{sum of readings}}{\text{number of readings}}  
$$

只需要保存两个变量：

```text
sum
count
```

每来一个新温度 $x$：

```text
sum = sum + x
count = count + 1
average = sum / count
```

不需要保存所有历史温度。

---

#### 查询 3：记录历史最高温度

```text
Max temperature ever recorded
```

只需要保存一个变量：

```text
current_max
```

每来一个新温度 $x$：

```text
if x > current_max:
    current_max = x
```

所以 standing query 通常比较容易，因为系统知道目标，能专门设计少量状态变量。

---

## 4.2 Ad-hoc Queries 临时查询 / 即席查询

### 4.2.1 定义

**Ad-hoc query（临时查询）** 是用户突然问的问题，系统事先不知道。

例如：

```text
过去一个月中，典型用户有多少查询是重复查询？
最近 1000 个搜索查询中，哪个 topic 突然变热？
过去一小时内，多少独立 IP 访问了网站？
```

这些问题在数据到来时可能并不知道。

---

### 4.2.2 为什么 ad-hoc query 难？

因为系统不知道未来会问什么，所以很难提前保存“刚好需要的信息”。

如果你保存所有数据：

```text
内存不够
存储不够
处理速度不够
```

如果你保存太少：

```text
之后无法回答临时查询
```

所以 ad-hoc query 通常依赖：

```text
sampling
sliding window
hashing
probabilistic data structures
approximation
```

---

### 4.2.3 Sliding Window 滑动窗口

一种常见策略是只保存最近一段数据：

```text
最近 n 个元素
最近 t 时间之后的元素
最近 1 小时数据
最近 1000 条数据
```

例如：

```text
stream = x1, x2, x3, ..., x1000000
window size = 1000
只保存 x999001 到 x1000000
```

优点：

```text
内存可控
能回答最近状态的问题
```

缺点：

```text
无法回答全历史问题
窗口外数据丢失
只能近似反映整体
```

---

# 5. Sampling from Streams 从数据流中采样

讲义第 8 页提出的采样问题是：选择一个数据流子集，使它在统计上代表原始 stream，并能帮助回答 ad-hoc queries。例子是搜索引擎收到 `(user, query, time)` 形式的数据流，并且只能保存 1/10 的 stream。

---

## 5.1 采样的目标

我们希望从巨大 stream 中抽取一小部分：

```text
原始流: x1, x2, x3, x4, x5, ...
样本:   x3, x9, x15, ...
```

目标是：

```text
样本足够小，可以存储
样本足够代表原始数据，可以回答问题
```

但是重点在于：

**不是所有 query 都能靠“随机抽一些元素”正确回答。**

这正是本节课的难点。

---

## 5.2 重要例子：搜索查询重复率

### 5.2.1 数据形式

搜索引擎收到的数据流是：

```text
(user, query, time)
```

例如：

```text
(Alice, "weather Southampton", 10:01)
(Alice, "weather Southampton", 10:05)
(Bob, "football score", 10:06)
(Charlie, "macbook air", 10:07)
```

现在只能保存：

```text
1/10 of the stream
```

问题是：

```text
What fraction of typical user queries were repeated over the last month?
```

中文意思：

```text
一个典型用户过去一个月中，有多少比例的查询是重复查询？
```

---

## 5.3 真实答案是什么？

假设一个典型用户：

- 有 $s$ 个 query 只出现过一次。
    
- 有 $d$ 个 query 出现过两次。
    
- 没有 query 出现超过两次。
    

那么这个用户的不同 query 总数是：

$$  
s + d  
$$

其中重复过的 query 数量是：

$$  
d  
$$

所以真实的重复查询比例是：

$$  
\frac{d}{s+d}  
$$

这个是 full stream 上的正确答案。

---

## 5.4 错误方法：随机抽取 1/10 的查询事件

最直观的想法是：

```text
每来一条查询记录，就以 1/10 的概率保存
```

也就是 event-level sampling：

```text
sample each query event independently with probability p = 1/10
```

看起来很合理，但它会给这个问题造成偏差。

---

## 5.5 为什么随机抽 1/10 查询事件会错？

设采样概率：

$$  
p = \frac{1}{10}  
$$

---

### 5.5.1 对只出现一次的 query

一个只出现一次的 query 被采到的概率是：

$$  
p = \frac{1}{10}  
$$

所以 $s$ 个只出现一次的 query 中，样本里期望看到：

$$  
\frac{s}{10}  
$$

个。

---

### 5.5.2 对出现两次的 query：两次都被采到

一个重复 query 在 full stream 中出现两次。

要在 sample 中仍然看起来“重复”，必须两次都采到。

概率是：

$$  
p^2 = \frac{1}{10} \times \frac{1}{10} = \frac{1}{100}  
$$

所以 $d$ 个重复 query 中，样本里期望有：

$$  
\frac{d}{100}  
$$

个仍然出现两次。

---

### 5.5.3 对出现两次的 query：只采到一次

一个重复 query 只在样本中出现一次，有两种情况：

```text
第一次采到，第二次没采到
第一次没采到，第二次采到
```

概率是：

$$  
2p(1-p)  
$$

代入 $p = \frac{1}{10}$：

$$  
2 \times \frac{1}{10} \times \frac{9}{10}  
= \frac{18}{100}  
$$

所以 $d$ 个重复 query 中，样本里期望有：

$$  
\frac{18d}{100}  
$$

个只出现一次。

---

## 5.6 样本中会看到什么？

样本中“出现一次”的 query 数量来自两部分：

```text
原本只出现一次，并且被采到
原本出现两次，但只采到其中一次
```

所以样本中出现一次的 query 数量期望是：

$$  
\frac{s}{10} + \frac{18d}{100}  
$$

样本中出现两次的 query 数量期望是：

$$  
\frac{d}{100}  
$$

样本中 distinct query 总数是：

$$  
\frac{s}{10} + \frac{18d}{100} + \frac{d}{100}  
$$

也就是：

$$  
\frac{s}{10} + \frac{19d}{100}  
$$

因此 sample 中观察到的重复比例是：

$$  
\frac{\frac{d}{100}}{\frac{s}{10}+\frac{19d}{100}}  
$$

把分子分母同时乘以 100：

$$  
\frac{d}{10s+19d}  
$$

---

## 5.7 为什么这是错的？

真实答案是：

$$  
\frac{d}{s+d}  
$$

随机采样 query event 得到的是：

$$  
\frac{d}{10s+19d}  
$$

这两个通常不相等。

讲义第 9 页也给出了同样结论：1/10 sample 会观察到 $s/10$ 个只出现一次的 queries、$d/100$ 个出现两次的 queries、以及 $18d/100$ 个原本出现两次但样本中只出现一次的 queries，最后得到 $d/(10s+19d)$，因此这类 query 不能通过随机采样 stream events 来回答。

---

## 5.8 直觉解释：为什么 event-level sampling 会破坏重复结构？

这个问题的关键不是“总共有多少条查询事件”，而是：

```text
同一个用户的同一个 query 是否重复出现
```

重复性是一个 **group-level / user-level property**。

但 event-level sampling 是把每条记录独立抽样：

```text
第一次 query 可能被采到
第二次 query 可能没被采到
```

这样一个本来重复的 query，很容易在样本里变成只出现一次。

也就是说：

```text
随机抽事件会打断同一个 query 的重复关系
```

因此对“重复率”这种结构性问题，普通随机采样会产生系统性低估。

---

## 5.9 正确思路：Sample Users, Not Query Events

更好的方法是：

```text
不要随机抽查询事件
而是随机抽用户
一旦某个用户被抽中，就保存这个用户的所有查询
```

也就是：

```text
sample 1/10 of users
store all queries from sampled users
```

讲义第 10 页给出的改进方案正是 sample 1/10 of users and store all their queries；可以用用户列表判断，也可以用 hashing 把用户名映射到 10 个 buckets，只保留映射到 bucket 0 的用户查询。

---

### 5.9.1 为什么 sample users 是对的？

因为问题问的是：

```text
typical user queries
```

所以采样单位应该是：

```text
user
```

而不是：

```text
individual query event
```

如果用户被选中：

```text
该用户所有 query 都保存
重复关系保留完整
```

如果用户没被选中：

```text
该用户所有 query 都不保存
```

这样不会把一个用户的历史记录拆碎。

---

### 5.9.2 举例说明

假设 Alice 的查询是：

```text
weather
weather
macbook
restaurant
```

其中：

```text
weather 重复
macbook 出现一次
restaurant 出现一次
```

如果按 query event 采样：

```text
可能只采到一个 weather
于是 weather 在 sample 中不再重复
```

如果按 user 采样：

```text
Alice 被选中 -> 保存全部查询
Alice 没被选中 -> 不保存
```

只要 Alice 被选中，她的重复结构就是完整的。

---

## 5.10 Hashing 用于构造样本

### 5.10.1 Hash function 是什么？

**Hash function（哈希函数）** 接收一个 key，然后把它映射到某个 bucket / location。

讲义第 11 页定义 hash function 为：以 key 作为输入，用于在数据存储和检索应用中识别对应 datum 或 record；优点是访问快，缺点是可能 collision。

形式上：

$$  
h(\text{key}) \rightarrow {0,1,2,\dots,b-1}  
$$

例如：

```text
h("Alice")   = 0
h("Bob")     = 7
h("Charlie") = 3
h("David")   = 0
```

---

### 5.10.2 Hashing 的优点

```text
非常快
不需要存完整用户列表
同一个 key 永远映射到同一个 bucket
可以稳定地决定某个用户是否进入 sample
```

例如：

```text
if h(user) == 0:
    store this user's query
else:
    ignore
```

---

### 5.10.3 Hashing 的缺点：Collision

**Collision（碰撞）** 指两个不同 key 被映射到同一个位置：

```text
h("Alice") = 0
h("David") = 0
```

在普通数据存储中，collision 可能导致不准确。

但在采样场景中，collision 本身不是严重问题，因为 bucket 的作用只是决定是否接受一个 key。只要 hash function 足够 uniform，就可以近似随机地选择用户。

---

## 5.11 General Key-based Sampling 一般化的 key-based sampling

数据流元素通常有多个 component。

例如：

```text
(user, query, time)
```

如果我们想按用户采样，那么 key 是：

```text
user
```

如果我们想按 IP 采样，那么 key 是：

```text
IP address
```

如果我们想按 product 采样，那么 key 是：

```text
product ID
```

通用方法：

```text
stream element = (key, value1, value2, ...)
hash key to b buckets
accept if hash(key) < a
```

采样比例是：

$$  
\frac{a}{b}  
$$

例如要采样 1/10：

$$  
a = 1,\quad b = 10  
$$

接受条件可以是：

$$  
h(\text{key}) = 0  
$$

或者更一般：

$$  
h(\text{key}) < a  
$$

---

## 5.12 如果 sample size 仍然太大怎么办？

即使采样 1/10，stream 可能仍然巨大。

讲义第 12 页提出：1/10 of the stream 可能依然很大，因此更希望被 active storage 的大小 $t$ 限制；可以 hash 到很大的值域 $0,1,\dots,B-1$，使用阈值，只保留 $h(key) \leq t$ 的元素；如果样本太大，就降低阈值并删除不再满足条件的元素。

为了避免符号混乱，这里把阈值记为 $\tau$。

---

### 5.12.1 动态阈值采样

设 hash function 输出范围：

$$  
h(\text{key}) \in {0,1,\dots,B-1}  
$$

初始阈值：

$$  
\tau = B-1  
$$

保留条件：

$$  
h(\text{key}) \leq \tau  
$$

如果 sample 太大：

```text
decrease tau
remove elements whose h(key) > tau
```

这样 sample size 可以根据内存限制动态调整。

---

### 5.12.2 伪代码

```text
Input stream: elements with key
Hash range: 0, 1, ..., B-1
Threshold: tau

For each incoming element e:
    key = key(e)

    if h(key) <= tau:
        store e

    if sample_size > memory_limit:
        decrease tau
        remove all stored elements whose h(key) > tau
```

---

### 5.12.3 这个方法的核心思想

不是直接说：

```text
我要固定采 1/10
```

而是说：

```text
我要让样本大小适应内存
```

如果 stream 比预期大，就降低接受比例。

如果 stream 较小，可以接受更多 key。

---

# 6. Filtering Streams 数据流过滤

## 6.1 问题定义

**Filtering streams（过滤数据流）** 的目标是：

```text
只让满足某个条件的 tuple 通过
不满足条件的 tuple 被丢弃
```

讲义第 14 页把 filtering stream 描述为：incoming stream 中的 tuple 形式为 `(key, value)`，例如 email stream 中 key 是 email address，value 是 email content，输出 stream 例如 non-spam emails。

结构如下：

```text
Incoming stream                     Outgoing stream
(key, value)  ---> Stream filter ---> accepted tuples only
```

---

## 6.2 简单过滤 vs 困难过滤

### 6.2.1 简单情况：条件可以直接计算

例如：

```text
temperature > 25
packet_size > 1000
email_content contains banned word
```

这些条件可以直接从 tuple 中计算。

伪代码：

```text
if criterion(tuple) == True:
    pass tuple
else:
    reject tuple
```

---

### 6.2.2 困难情况：条件是 set membership

难的是这种问题：

```text
这个 key 是否属于某个巨大集合 S？
```

例如：

```text
email address 是否在 allowed email addresses 集合中？
IP address 是否在 known malicious IP set 中？
URL 是否在 blacklist 中？
user ID 是否在 paid users set 中？
```

如果集合 $S$ 很大，不能全部存进内存。

这就需要 **Bloom filter（布隆过滤器）**。

讲义第 15 页明确说：如果 selection criterion 是 tuple 的可计算属性，很容易；如果是某个大集合中的 membership，尤其集合很大，就困难，需要 Bloom filtering。

---

# 7. Bloom Filter 布隆过滤器

## 7.1 Bloom Filter 解决什么问题？

Bloom filter 用来回答：

```text
某个元素 x 是否可能在集合 S 中？
```

它的回答不是完全精确的，而是：

```text
definitely not in S
possibly in S
```

讲义第 18 页给出 Bloom filter 的核心性质：它是一种 space-efficient probabilistic data structure，用于测试元素是否属于集合；false positives 可能出现，但 false negatives 不会出现，因此查询结果要么是 “possibly in set”，要么是 “definitely not in set”。

---

## 7.2 Bloom Filter 的核心特点

### 7.2.1 可能有 False Positive

**False positive（假阳性）**：

```text
x 不在 S 中
Bloom filter 却说 x 可能在 S 中
```

例子：

```text
一个 spam email address 不在 allowed set 里
但它 hash 到的位置刚好是 1
系统误以为它可能合法
于是放行
```

---

### 7.2.2 不会有 False Negative

**False negative（假阴性）**：

```text
x 在 S 中
Bloom filter 却说 x definitely not in S
```

标准 Bloom filter 不会出现这种情况。

因为如果 $x$ 被加入过集合 $S$，那么它对应的 hash bits 一定已经被设置为 1。

---

## 7.3 单 hash function 的 Bloom Filter：Spam Detection 例子

### 7.3.1 场景

假设我们有：

```text
1GB main memory
1 billion allowed email addresses
```

1GB 内存大约是：

$$  
8 \text{ billion bits}  
$$

我们不能直接存 1 billion email addresses，因为 email address 本身是字符串，每个可能几十 bytes，远大于 1GB。

于是我们使用 bit array。

---

### 7.3.2 Preparation 初始化阶段

讲义第 16 页的 spam detection 例子是：用 main memory 作为 8 billion bits 的 bit array，全部初始化为 0；设计 hash function，把 email addresses 映射到 8 billion buckets；对集合 $S$ 中所有 email addresses，把对应 bit 设置为 1。

具体步骤：

```text
1. 创建一个长度为 8 billion 的 bit array
2. 所有 bit 初始化为 0
3. 对每个 allowed email address x:
       i = h(x)
       bit_array[i] = 1
```

形式化写法：

$$  
A[h(x)] = 1,\quad \forall x \in S  
$$

---

### 7.3.3 Processing 查询阶段

当一个新 email 到来：

```text
email_address = key
i = h(email_address)
```

如果：

$$  
A[h(\text{email})] = 1  
$$

则：

```text
accept
```

如果：

$$  
A[h(\text{email})] = 0  
$$

则：

```text
reject
```

---

## 7.4 为什么 bit 为 0 时一定不在集合中？

假设某个 email address $x$ 真正在 allowed set $S$ 中。

那么初始化时一定执行过：

$$  
A[h(x)] = 1  
$$

所以查询时不可能看到：

$$  
A[h(x)] = 0  
$$

因此如果查询时发现：

$$  
A[h(x)] = 0  
$$

就可以确定：

$$  
x \notin S  
$$

这就是 **no false negatives（无假阴性）** 的原因。

---

## 7.5 为什么 bit 为 1 时不能确定在集合中？

如果：

$$  
A[h(x)] = 1  
$$

有两种可能：

```text
情况 1：x 确实在 S 中，是 x 自己把这个 bit 设置成 1
情况 2：x 不在 S 中，但是另一个 y in S 恰好 h(y)=h(x)，把这个 bit 设置成 1
```

第二种情况就是 collision 导致的 false positive。

所以 Bloom filter 的判断是：

```text
0 -> definitely not in set
1 -> possibly in set
```

---

## 7.6 单 hash function 下 false positive 的近似概率

在讲义例子中：

```text
bit array size = 8 billion
allowed addresses = 1 billion
```

如果忽略 collision 的复杂影响，大约有：

$$  
\frac{1}{8}  
$$

的 bit 会被设置为 1。

所以一个不在 $S$ 中的 email address 随机 hash 到一个 bit，有大约：

$$  
\frac{1}{8}  
$$

概率落在 1 上，从而 false positive。

讲义第 17 页也指出：大约 1/8 的 array values 会是 1；如果某个不在 $S$ 中的 key 恰好 hash 到 1，就会被错误放行，因此约 1/8 的不在 $S$ 中的 email addresses 会通过。

---

## 7.7 Bloom Filter 的优点

### 7.7.1 非常节省空间

它不用存完整元素：

```text
不存 email string
只存 bit array
```

例如：

```text
1 billion email addresses
```

如果每个 email 需要几十 bytes，存储会非常大。

Bloom filter 只需要：

```text
n bits
```

---

### 7.7.2 查询速度非常快

查询只需要：

```text
计算 hash function
访问 bit array
```

时间复杂度大致是：

$$  
O(k)  
$$

其中 $k$ 是 hash functions 数量。

单 hash function 时就是：

$$  
O(1)  
$$

讲义第 17 页也强调 Bloom filter 查询非常快，成本基本就是计算 hash function 的成本。

---

### 7.7.3 没有 false negatives

对于很多过滤场景，这一点非常重要。

例如：

```text
如果系统说某个 email definitely not allowed
那它真的不在 allowed set 中
```

---

## 7.8 Bloom Filter 的缺点

### 7.8.1 有 false positives

它可能错误地接受不该接受的元素。

例如：

```text
spam email 可能被当成 allowed email
```

---

### 7.8.2 标准 Bloom Filter 不能删除元素

为什么不能删除？

假设：

```text
h(x) = 5
h(y) = 5
```

也就是说 $x$ 和 $y$ 都把 bit 5 设置成 1。

如果现在想删除 $x$，你不能直接把：

$$  
A[5] = 0  
$$

因为这样会破坏 $y$ 的信息。

之后查询 $y$ 时会错误地得到：

```text
definitely not in S
```

这就产生 false negative。

因此标准 Bloom filter 不支持安全删除。

---

### 7.8.3 删除的解决思路：Counting Bloom Filter

进阶地，可以用 **Counting Bloom Filter（计数布隆过滤器）**。

不是存 bit：

```text
0 / 1
```

而是存 counter：

```text
0, 1, 2, 3, ...
```

插入元素时：

```text
counter[h_i(x)] += 1
```

删除元素时：

```text
counter[h_i(x)] -= 1
```

查询时：

```text
所有 counter > 0 -> possibly in S
否则 -> definitely not in S
```

但这需要更多内存。

---

## 7.9 多 hash function 的 Bloom Filter

讲义第 19 页给出一般 Bloom filter 结构：一个长度为 $n$ 的 bit array，初始为 0；一组 $k$ 个 hash functions $h_1,h_2,\dots,h_k$；集合 $S$ 有 $m$ 个 key values；初始化时对每个 key 设置所有 $h_i(key)$ 对应 bit；查询时检查所有 $h_1(key),\dots,h_k(key)$ 是否都为 1。

---

### 7.9.1 初始化

对于每个元素 $x \in S$：

$$  
A[h_1(x)] = 1  
$$

$$  
A[h_2(x)] = 1  
$$

一直到：

$$  
A[h_k(x)] = 1  
$$

---

### 7.9.2 查询

对于新元素 $y$：

```text
if A[h1(y)] == 1 and A[h2(y)] == 1 and ... and A[hk(y)] == 1:
    return "possibly in S"
else:
    return "definitely not in S"
```

---

### 7.9.3 为什么多个 hash functions 能降低 false positive？

单 hash function 中，false positive 只需要：

```text
一个位置刚好是 1
```

多 hash function 中，false positive 需要：

```text
k 个位置全部刚好是 1
```

这更难发生。

所以 false positive probability 降低。

---

## 7.10 Bloom Filter 的 false positive 公式

设：

- bit array 长度为 $n$
    
- 集合中有 $m$ 个元素
    
- 使用 $k$ 个 hash functions
    

某一个 bit 在插入一个 hash result 后仍为 0 的概率是：

$$  
1 - \frac{1}{n}  
$$

总共有 $km$ 次 hash 写入，所以某个 bit 仍为 0 的概率约为：

$$  
\left(1-\frac{1}{n}\right)^{km}  
$$

当 $n$ 很大时：

$$  
\left(1-\frac{1}{n}\right)^{km} \approx e^{-km/n}  
$$

所以某个 bit 为 1 的概率约为：

$$  
1 - e^{-km/n}  
$$

false positive 要求 $k$ 个位置全为 1，因此：

$$  
p_{fp} \approx \left(1-e^{-km/n}\right)^k  
$$

---

## 7.11 Bloom Filter 参数选择直觉

### 7.11.1 $n$ 越大越好

$n$ 是 bit array 长度。

更大的 $n$ 意味着：

```text
更多 bucket
collision 更少
false positive 更低
```

---

### 7.11.2 $m$ 越大 false positive 越高

$m$ 是集合大小。

元素越多，设置为 1 的 bit 越多，false positive 越高。

---

### 7.11.3 $k$ 不是越大越好

增加 $k$ 有两个相反影响：

```text
好处：query 需要多个 bit 都为 1，false positive 降低
坏处：每个元素会设置更多 bit，bit array 更快变满
```

所以 $k$ 有最佳值。

常见近似最佳值是：

$$  
k = \frac{n}{m}\ln 2  
$$

直觉是让 bit array 中大约一半 bit 为 1，这样信息利用率较好。

---

# 8. Counting Distinct Elements in a Stream 统计数据流中不同元素数量

## 8.1 问题定义

给定一个 stream：

```text
1, 2, 1, 3, 2, 4, 4, 4, 5, ...
```

问题是：

```text
How many different elements have appeared?
```

也就是 distinct count。

讲义第 21 页提出的问题是：stream elements 从集合 $S$ 中选择，想知道有多少不同元素出现过；例子包括网站统计 unique users，比如 Amazon 用户名或 Google 中由 IP addresses 识别的搜索查询。显然可以存储所有元素再计数，但如果 distinct elements 太多就不可行，因此要用 hashing 估计。

---

## 8.2 例子：Unique Users

网站日志流：

```text
Alice
Bob
Alice
Charlie
Bob
David
```

不同用户是：

```text
Alice, Bob, Charlie, David
```

所以 distinct count 是：

$$  
4  
$$

---

## 8.3 最直接方法：存 set

```text
seen = empty set

for each x in stream:
    seen.add(x)

answer = len(seen)
```

问题是：

```text
如果 unique users 有几亿
set 太大
内存不够
```

因此需要近似算法。

---

# 9. Flajolet-Martin Algorithm

## 9.1 核心思想

**Flajolet-Martin algorithm** 用于估计 stream 中 distinct elements 的数量。

它的核心思想是：

```text
不同元素越多
看到不同 hash values 的机会越多
看到“非常罕见”的 hash pattern 的机会越大
```

讲义第 22 页说明：将元素 hash 成足够长的 bit string；不同元素越多，hash values 越多，也越可能出现 unusual hash value；这里关注的是 tail length，即 hash 末尾 0 的数量；令 $R$ 为目前见过的最大 tail length，然后估计 distinct elements 数量为 $2^R$。

---

## 9.2 Tail Length 尾部 0 的数量

定义：

**tail length（尾长）** = 一个二进制 hash value 末尾连续 0 的数量。

例如：

```text
1011000
```

末尾有 3 个 0，所以 tail length 是：

$$  
3  
$$

再例如：

```text
101010
```

末尾有 1 个 0，所以 tail length 是：

$$  
1  
$$

再例如：

```text
111011
```

末尾没有 0，所以 tail length 是：

$$  
0  
$$

---

## 9.3 为什么 tail length 能估计 distinct count？

假设 hash function 是 uniform random。

一个 hash value 末尾至少有 $r$ 个 0 的概率是：

$$  
2^{-r}  
$$

原因：

```text
末尾 1 个 bit 是 0 的概率 = 1/2
末尾 2 个 bit 都是 0 的概率 = 1/4
末尾 r 个 bit 都是 0 的概率 = 1/2^r
```

所以：

$$  
P(\text{tail length} \geq r) = 2^{-r}  
$$

---

## 9.4 m 个 distinct elements 时发生什么？

假设 stream 中有 $m$ 个 distinct elements。

每个 distinct element 经过 hash 后，都有概率 $2^{-r}$ 出现至少 $r$ 个 trailing zeros。

---

### 9.4.1 没有任何元素出现 tail length 至少 r 的概率

单个元素没有 tail length 至少 $r$ 的概率是：

$$  
1 - 2^{-r}  
$$

$m$ 个 distinct elements 都没有的概率是：

$$  
(1 - 2^{-r})^m  
$$

当 $2^{-r}$ 较小、$m$ 较大时，可以近似为：

$$  
(1 - 2^{-r})^m \approx e^{-m2^{-r}}  
$$

---

### 9.4.2 至少一个元素出现 tail length 至少 r 的概率

所以至少一个元素有 tail length 至少 $r$ 的概率是：

$$  
1 - (1 - 2^{-r})^m  
$$

近似为：

$$  
1 - e^{-m2^{-r}}  
$$

---

## 9.5 阈值直觉：为什么估计是 $2^R$？

关键看 $m$ 和 $2^r$ 的关系。

---

### 9.5.1 如果 $m \gg 2^r$

这时：

$$  
m2^{-r} = \frac{m}{2^r}  
$$

很大。

所以：

$$  
e^{-m2^{-r}} \approx 0  
$$

因此：

$$  
1 - e^{-m2^{-r}} \approx 1  
$$

也就是说：

```text
如果 distinct elements 数量远大于 2^r
几乎一定能看到至少 r 个 trailing zeros
```

---

### 9.5.2 如果 $m \ll 2^r$

这时：

$$  
m2^{-r} = \frac{m}{2^r}  
$$

很小。

所以：

$$  
1 - e^{-m2^{-r}} \approx 0  
$$

也就是说：

```text
如果 distinct elements 数量远小于 2^r
几乎不可能看到至少 r 个 trailing zeros
```

---

### 9.5.3 临界点

发生转变的位置大约是：

$$  
m \approx 2^r  
$$

如果目前看到的最大 tail length 是 $R$，说明 distinct count 大概在：

$$  
2^R  
$$

这个数量级。

所以 Flajolet-Martin 的基本估计是：

$$  
\hat{m} = 2^R  
$$

讲义第 23 页给出的解释也是：当 $m$ much larger than $2^r$ 时，找到至少长度 $r$ 的 tail 的概率接近 1；当 $m$ much smaller than $2^r$ 时，概率接近 0，因此 $2^R$ 不太可能过高或过低。

---

## 9.6 Flajolet-Martin 算法步骤

### 9.6.1 单 hash function 版本

```text
Input: data stream
Initialize R = 0

For each element x in stream:
    y = h(x)
    r = tail_length(y)
    R = max(R, r)

Estimate distinct count = 2^R
```

---

### 9.6.2 关键变量

|符号|含义|
|---|---|
|$x$|stream 中到来的元素|
|$h(x)$|元素 $x$ 的 hash value|
|$r$|当前 hash value 的 tail length|
|$R$|到目前为止观察到的最大 tail length|
|$\hat{m}$|distinct elements 数量估计|
|$\hat{m} = 2^R$|Flajolet-Martin 的基本估计|

---

## 9.7 为什么只需要存一个整数？

对一个 hash function 来说，整个算法只需要保存：

```text
R = max tail length seen so far
```

不需要保存：

```text
所有元素
所有 hash values
所有出现过的用户
```

这就是它适合 data stream 的原因。

讲义第 25 页总结其优点：不需要存储任何 stream element；每个 hash function 只需一个 integer；hash functions 越多估计越好，但实际限制是计算 hash functions 的成本；缺点是只提供 approximate solution。

---

# 10. Flajolet-Martin 示例详解

讲义第 26 页给出的例子是：

```text
stream: 1, 2, 1, 3
hash function: h(x) = 2x + 2 mod 4
treated as 3-bit integer
```

对应：

```text
1 -> 0 -> 000
2 -> 2 -> 010
3 -> 0 -> 000
```

其中：

```text
010 的 tail length = 1
```

所以：

$$  
R = 1  
$$

估计 distinct count：

$$  
2^R = 2^1 = 2  
$$

但真实 distinct elements 是：

```text
{1, 2, 3}
```

真实数量是：

$$  
3  
$$

所以这个估计不太好。

讲义也指出这个 hash function 给出的估计不是很好，需要更多 hash functions 改进。

---

## 10.1 为什么这个例子估计不好？

原因有几个：

```text
1. hash value 空间太小
2. hash function 太简单
3. 多个元素被 hash 到 0
4. stream 太短
5. 只用了一个 hash function
```

尤其是：

```text
1 -> 000
3 -> 000
```

这两个不同元素映射到同一个值，信息损失严重。

---

## 10.2 为什么 mapping to 0 的值要小心？

二进制：

```text
000
```

看起来有很多 trailing zeros。

但如果 hash value 全是 0，这通常不能像普通 tail length 一样解释，因为它可能会造成极端或无意义估计。

讲义例子旁边也提示：mapping to 0 时应该 ignore，因为它们不提供信息。

---

# 11. 多个 hash functions 如何组合？

单个 hash function 的估计波动很大。

所以实际中会用多个 hash functions：

```text
h1, h2, h3, ..., hk
```

每个 hash function 得到一个估计：

$$  
2^{R_1}, 2^{R_2}, \dots, 2^{R_k}  
$$

问题是：

```text
怎么把这些估计合成一个更稳定的估计？
```

---

## 11.1 直接取平均值为什么不好？

直觉上可以取：

$$  
\frac{1}{k}\sum_{i=1}^{k} 2^{R_i}  
$$

但讲义第 24 页指出：直接取 expectation / mean 通常会 overestimate。

原因是：

```text
2^R 对 R 非常敏感
R 偶尔大 1，估计就翻倍
R 偶尔很大，会把平均值拉得很高
```

例如估计值：

```text
8, 8, 16, 8, 1024
```

平均值是：

$$  
\frac{8+8+16+8+1024}{5}=212.8  
$$

但大多数估计都在 8 或 16 附近。

这个平均值被 1024 严重拉高。

---

## 11.2 直接取 median 为什么也不完美？

Median 不容易被极端值影响。

例如：

```text
8, 8, 16, 8, 1024
```

median 是：

$$  
8  
$$

比 mean 稳定。

但问题是 Flajolet-Martin 的单个估计通常是：

$$  
2^R  
$$

即：

```text
1, 2, 4, 8, 16, 32, 64, ...
```

它只能是 2 的幂。

所以 median 也比较粗糙，难以得到细致估计。

讲义第 24 页也指出：median 不受偶尔很大的 $2^R$ 影响，但总是 power of 2，因此难以得到 close estimate。

---

## 11.3 正确组合策略：mean within groups, median across groups

课程给出的折中办法是：

```text
把多个 hash functions 分成若干小组
每组内部取平均
再对各组平均值取 median
```

也就是：

```text
Group 1: h1, h2, h3, h4       -> average estimate A1
Group 2: h5, h6, h7, h8       -> average estimate A2
Group 3: h9, h10, h11, h12    -> average estimate A3

Final estimate = median(A1, A2, A3)
```

---

### 11.3.1 为什么这样有效？

组内平均：

```text
减少单个 hash function 的随机波动
```

组间 median：

```text
防止某一组被极端大值严重拉偏
```

所以它结合了：

```text
mean 的平滑性
median 的鲁棒性
```

---

# 12. 三个核心问题的统一理解

本节课其实围绕三个典型数据流问题：

```text
1. Sampling from streams
2. Filtering streams
3. Counting distinct elements
```

它们背后的共同思想是：

```text
不能完整保存数据
所以用少量状态 + hashing + 概率近似
```

---

## 12.1 对比表

|问题|目标|传统精确做法|数据流困难|本课方法|
|---|---|---|---|---|
|Sampling|保存代表性样本|保存全数据后抽样|stream 太大，不能保存全部|key-based hashing sampling|
|Filtering|判断元素是否属于集合|存完整集合 $S$|$S$ 太大|Bloom filter|
|Distinct counting|统计 unique elements|存 set|distinct elements 太多|Flajolet-Martin|

---

## 12.2 三者都依赖 hashing

### Sampling 中 hashing 的作用

```text
用 hash(key) 决定是否采样某个 key
```

### Bloom filter 中 hashing 的作用

```text
用 hash(key) 映射到 bit array
用 bit pattern 近似表示集合 membership
```

### Flajolet-Martin 中 hashing 的作用

```text
用 hash value 的 trailing zeros 估计 distinct count
```

---

# 13. 常见易错点 Deep Dive

## 13.1 易错点 1：随机采样 stream events 不一定代表所有统计问题

很多人会想：

```text
随机采样是无偏的，所以一定可以回答问题
```

这是错的。

随机采样 event 对一些简单统计可能可以，例如：

```text
平均 temperature
平均 packet size
关键词出现比例
```

但对结构性问题可能不行，例如：

```text
某个用户是否重复查询
某个 IP 是否多次访问
某个 item 是否在同一个 basket 中和另一个 item 共现
```

因为这些问题依赖多个事件之间的关系。

---

## 13.2 易错点 2：采样单位必须和问题单位一致

问题问：

```text
typical user queries
```

所以采样单位应该是：

```text
user
```

不是：

```text
query event
```

类似地：

```text
如果问题问 typical session -> sample sessions
如果问题问 typical customer -> sample customers
如果问题问 product-level repetition -> sample products
```

---

## 13.3 易错点 3：Bloom filter 的 positive 不是真 positive

Bloom filter 返回：

```text
possibly in S
```

不是：

```text
definitely in S
```

所以：

```text
bit check all 1 -> 可能在集合中
bit check any 0 -> 一定不在集合中
```

这句话要背熟。

---

## 13.4 易错点 4：Bloom filter 没有 false negatives，但有 false positives

标准 Bloom filter：

```text
False positive: possible
False negative: impossible
```

考试中常见说法判断：

```text
Bloom filter may reject an element that is actually in the set.
```

这是错的。

```text
Bloom filter may accept an element that is not actually in the set.
```

这是对的。

---

## 13.5 易错点 5：Bloom filter 不能直接删除

不能把 bit 从 1 改成 0，因为这个 bit 可能被多个元素共享。

---

## 13.6 易错点 6：Flajolet-Martin 估计的是 distinct elements，不是 stream length

如果 stream 是：

```text
1, 1, 1, 1, 1, 1
```

stream length 是：

$$  
6  
$$

distinct count 是：

$$  
1  
$$

Flajolet-Martin 估计的是 distinct count。

重复出现同一个元素不会增加 distinct count。

---

## 13.7 易错点 7：Flajolet-Martin 中 $R$ 是最大 tail length

不是：

```text
平均 tail length
最后一个元素的 tail length
所有 tail length 总和
```

而是：

$$  
R = \max_i r_i  
$$

其中 $r_i$ 是每个 distinct hash value 的 trailing zeros 数量。

---

# 14. 算法伪代码总整理

## 14.1 Key-based Sampling

```text
Input:
    Stream elements e = (key, value)
    Hash function h
    Number of buckets b
    Accept buckets: 0, 1, ..., a-1

For each incoming element e:
    key = extract_key(e)

    if h(key) mod b < a:
        store e
    else:
        discard e
```

采样比例：

$$  
\frac{a}{b}  
$$

---

## 14.2 Dynamic Hash Sampling with Memory Bound

```text
Input:
    Stream elements e
    Hash function h mapping keys to 0,...,B-1
    Threshold tau
    Memory limit M

Initialize:
    sample = empty
    tau = B - 1

For each incoming element e:
    key = extract_key(e)

    if h(key) <= tau:
        add e to sample

    while size(sample) > M:
        decrease tau
        remove all elements e from sample where h(key(e)) > tau
```

---

## 14.3 Bloom Filter

```text
Input:
    Set S
    Bit array A of length n
    Hash functions h1, h2, ..., hk

Initialization:
    for i in 0 to n-1:
        A[i] = 0

    for each x in S:
        for j in 1 to k:
            A[hj(x)] = 1

Query(y):
    for j in 1 to k:
        if A[hj(y)] == 0:
            return "definitely not in S"

    return "possibly in S"
```

---

## 14.4 Flajolet-Martin Algorithm

```text
Input:
    Stream elements x1, x2, ...
    Hash function h

Initialize:
    R = 0

For each incoming element x:
    y = h(x)
    r = number_of_trailing_zeros(y)

    if r > R:
        R = r

Return:
    estimate = 2^R
```

---

## 14.5 Flajolet-Martin with Multiple Hash Functions

```text
Input:
    Stream elements
    Hash functions h1, h2, ..., hk

Initialize:
    R[j] = 0 for each hash function hj

For each incoming element x:
    for j in 1 to k:
        y = hj(x)
        r = trailing_zeros(y)
        R[j] = max(R[j], r)

For each j:
    estimate[j] = 2^R[j]

Group estimates into groups
Compute average within each group
Return median of group averages
```

---

# 15. 考试/复习重点总结

老师在课堂转录开头提到 exam 中不会提供公式，因此这些公式和推导需要自己掌握。

---

## 15.1 必须会解释的概念

```text
Data stream
Stream processor
Limited working storage
Standing query
Ad-hoc query
Sliding window
Hashing
Collision
Key-based sampling
Bloom filter
False positive
False negative
Flajolet-Martin algorithm
Tail length / trailing zeros
Distinct elements
```

---

## 15.2 必须会推导的公式

### 15.2.1 Query repetition sampling bias

真实重复比例：

$$  
\frac{d}{s+d}  
$$

event-level 1/10 sample 中：

# $$  
\text{sample repeated fraction}

\frac{d}{10s+19d}  
$$

必须理解为什么：

$$  
\frac{d}{100}  
$$

是两次都采到的数量，而：

$$  
\frac{18d}{100}  
$$

是只采到一次的数量。

---

### 15.2.2 Bloom filter false positive 概率

一般近似：

$$  
p_{fp} \approx \left(1-e^{-km/n}\right)^k  
$$

其中：

```text
n = bit array length
m = number of inserted elements
k = number of hash functions
```

---

### 15.2.3 Flajolet-Martin trailing zeros 概率

某个 hash value 至少有 $r$ 个 trailing zeros 的概率：

$$  
2^{-r}  
$$

$m$ 个 distinct elements 中，至少一个有 $r$ 个 trailing zeros 的概率：

$$  
1 - (1 - 2^{-r})^m  
$$

近似：

$$  
1 - e^{-m2^{-r}}  
$$

估计：

$$  
\hat{m} = 2^R  
$$

---

# 16. 一句话总括

**Mining Data Streams 的核心不是“如何在大数据上跑传统算法”，而是“在数据高速到达、无法完整存储、通常只能处理一次的条件下，用少量内存和 hashing/probabilistic methods 得到足够好的近似答案”。**