下面这份按 **Lecture 16: Local features and matching** 来复习。你上传的 transcript 文件名是 `18Local features and matching.txt`，但内容对应本讲 Local features and matching；主要 handout 使用 `16Local features and matching.pdf`；summary 使用 `CV23-Summary.pdf`；考试风格参考 `11Lecture_12_revision.pdf` 中的 problem solving / calculation / design and explain 要求。Revision 明确说明题目更偏 **problem solving / applying knowledge**，并会有数学/计算题和 design/explain 题。

---

# 1. 资料核对

本次使用资料：

1. **主要 handout/PDF**：`16Local features and matching.pdf`  
    handout 明确从 local interest points 回顾开始，然后进入 local features、panorama、matching pipeline、detector/descriptor 问题、narrow/wide baseline、SIFT 和 matching strategy。
    
2. **revision/summary**：`CV23-Summary.pdf`  
    本讲属于 Lecture 10–21，复习时要特别贴合 summary 中的关键词：**invariant, robust, repeatable, constraints**，以及 feature space 中通过 distance 比较 feature vectors 的思想。
    
3. **transcript**：`18Local features and matching.txt`  
    transcript 中老师反复强调：detected features 必须被表示成固定维度的数字向量，才能用于 image alignment、pose estimation、3D reconstruction、motion tracking、object recognition、image search 等任务。
    

资料冲突：  
没有发现明显冲突。handout 和 transcript 的主线一致：先检测稳定的 interest points，再为每个点构造 descriptor，最后用 feature-space distance 做 matching。

---

# 2. 这节课在 Computer Vision pipeline 中的位置

这节课的位置可以理解为：

```text
Image
  ↓
Interest point detection
Harris corners / DoG extrema
  ↓
Local descriptor extraction
SIFT-like descriptor
  ↓
Feature matching
nearest neighbour / ratio test
  ↓
Matched point pairs
  ↓
Geometric alignment / RANSAC / panorama / 3D / search
```

## 它解决什么问题？

上一讲已经解决了“在哪里找点”：找到 stable / repeatable 的 local interest points，比如 Harris corners 和 DoG blobs。handout 第3页也把这作为 recap。  
本讲解决的是：**找到点以后，如何描述这些点，并在另一张图里找到对应点。**

也就是说：

- detector 回答：这个点在哪里？
    
- descriptor 回答：这个点周围长什么样？
    
- matching 回答：两张图中哪些点是一对 correspondence？
    

## 输入是什么？

输入通常是两张图像，以及每张图像中检测到的 local interest points：

```text
Image 1: keypoints p1, p2, ..., pn
Image 2: keypoints q1, q2, ..., qm
```

每个 keypoint 附近取一个 patch，计算 descriptor。

## 输出是什么？

输出是 matched pairs：

```text
(p_i, q_j)
```

也就是第一张图中的某个 feature point 和第二张图中的某个 feature point 的对应关系。handout 中 panorama 例子明确展示了：detect feature points → find corresponding pairs → use pairs to align images。

## 和前后 lecture 的关系

前一讲 **Local interest points**：  
讲 Harris corners 和 DoG extrema，重点是 **stable / repeatable detector**。如果 detector 不 repeatable，两张图中检测到的点都不是同一个真实点，那后面 descriptor 再好也没用。老师在 transcript 中说得很直接：如果 detector 错了，“no chance of matching”。

本讲 **Local features and matching**：  
重点是 descriptor 和 feature matching。核心词是：

```text
invariant
robust
distinctive
repeatable
feature-space distance
```

下一讲 **Consistent matching**：  
本讲的 matching 仍然会产生 outliers。下一讲会用 geometric constraints、affine transform、homography、RANSAC 等方法筛掉不一致的匹配。下一讲 handout 也 recap：已经学过 stable interest points、robust descriptors、local feature matching，下一步问题是如何 eliminate mismatches。

---

# 3. 系统复习笔记

## 3.1 Local feature 是什么？【高频】【容易考】

handout 第4页给了最核心定义：interest points are detected in the image, and a feature is extracted from the surrounding pixels；每个 local interest point 产生一个 feature vector。

可以这样理解：

```text
local interest point = 图像中值得关注的位置
local patch = 这个点周围的一小块区域
descriptor = 用固定长度数字向量描述这个 patch
local feature = keypoint location + descriptor
```

比如：

```text
keypoint:
    location = (x, y)
    scale = σ
    orientation = θ

descriptor:
    f = [0.12, 0.04, 0.30, ..., 0.01]
```

考试里不要把 **detector** 和 **descriptor** 混在一起：

|概念|回答的问题|例子|
|---|---|---|
|detector|where is the point?|Harris, DoG|
|descriptor|what does the neighbourhood look like?|SIFT descriptor|
|matcher|which two descriptors correspond?|Euclidean distance, ratio test|

---

## 3.2 为什么不能直接用 RGB / raw pixel values？【老师强调】【易错点】

老师强调：如果一个 feature 只用几个 RGB / brightness values 描述，那么光照稍微变化，真实对应点的 RGB 可能就变了，反而和错误点更接近。因此 raw pixel values 不够 robust，需要 **invariant, robust and distinctive descriptor**。

直觉：

```text
同一个物体：
早上拍：暗一点
下午拍：亮一点

raw pixel value 变化很大
但边缘方向、局部纹理结构可能还相似
```

所以 descriptor 要尽量描述“结构”，而不是死记像素值。

---

## 3.3 好 descriptor 的三个核心性质【高频】【设计/解释题】

### 1. Invariant

图像发生某些变化后，descriptor 尽量不变：

```text
brightness change
rotation
scale change
small viewpoint change
small localisation error
```

### 2. Robust

面对噪声、轻微模糊、局部遮挡、检测位置小偏移时，descriptor 仍然稳定。

### 3. Distinctive / discriminative

正确匹配的 similarity 很高，错误匹配的 similarity 很低。老师解释 distinctive 时说：正确 match 应该有很高 similarity，其他候选点 similarity 很低，也就是 matching score 只在正确点附近有明显 peak。

一个考试表达模板：

```text
A good local descriptor should be invariant to expected transformations,
robust to noise and small localisation errors, and distinctive enough so
that the correct match is much closer than incorrect alternatives in feature space.
```

---

# 4. Narrow baseline vs Wide baseline

这是本讲非常容易考的对比题。

## 4.1 Narrow-baseline matching【对比题】

**Narrow baseline** 指两张图拍摄位置很近，图像变化小。老师举例包括 stereo pair、video frame-to-frame tracking、人眼两眼之间的小 baseline。transcript 中提到两张图位置很接近，场景看起来很相似，位移也比较小。

典型场景：

```text
stereo vision
video tracking
相邻帧 matching
small camera displacement
```

可用方法：

```text
template matching / SSD matching / search window
```

因为点不会移动太远，只需要在附近 window 里搜索。

## 4.2 Wide-baseline matching【高频】【对比题】

**Wide baseline** 指两张图拍摄角度、位置、时间差较大。比如从不同角度拍同一个物体，或者不同时间拍同一地点。老师说 wide baseline 中位置变化大、外观也可能变化很大，因此需要更好的 descriptor。

典型场景：

```text
panorama stitching
object recognition
image retrieval
random image collection matching
large viewpoint change
```

template matching 在 wide baseline 下通常失败，因为它：

- 不 robust to rotation；
    
- 对 localisation error 敏感；
    
- 大范围搜索很贵；
    
- 遇到 occlusion 可能根本没有对应 patch；
    
- viewpoint 变化后 patch 外观不再相似。
    

---

# 5. Template matching / SSD matching

## 5.1 核心公式【计算题】

对于 Image 1 中以点 $$p=(x,y)$$ 为中心的 patch，在 Image 2 中搜索位移 $$(u,v)$$：

$$  
E(u,v)=\sum_{(i,j)\in W}  
\left[  
I_1(x+i,y+j)-I_2(x+i+u,y+j+v)  
\right]^2  
$$

选择使误差最小的位置：

$$  
(u^_,v^_)=\arg\min_{u,v} E(u,v)  
$$

变量含义：

|符号|含义|
|---|---|
|$$I_1, I_2$$|两张图像|
|$$(x,y)$$|第一张图中 feature point 位置|
|$$W$$|patch/window 的局部区域|
|$$(i,j)$$|patch 内部相对坐标|
|$$(u,v)$$|第二张图中的候选位移|
|$$E(u,v)$$|SSD matching error|
|$$(u^_,v^_)$$|最优位移，即 best match|

## 5.2 直觉

把第一张图中的小方块拿出来，在第二张图附近滑动。每滑动一个位置，就比较两个 patch 的像素差平方和。差越小，越可能匹配。

## 5.3 优缺点【容易考】

优点：

- 简单直观；
    
- 对 narrow baseline 有效；
    
- 适合相邻帧、stereo matching、small displacement tracking。
    

缺点：

- 对 rotation 不鲁棒；
    
- 对 scale change 不鲁棒；
    
- 对 brightness change 敏感；
    
- 对 localisation error 敏感；
    
- 对 wide baseline 不适合；
    
- 搜索范围大时计算量高。
    

---

# 6. 从 raw patch 到 gradient-based descriptor

## 6.1 为什么用 gradient？【老师强调】【易错点】

老师解释 brightness change 时说：同一物体早上和晚上拍，像素绝对值可能完全不同；但像素之间的变化关系，即 brightness change pattern，可能仍然相似。因此可以使用 $$I_x, I_y$$ 这样的 derivative / gradient，而不是直接使用 pixel values。

## 6.2 梯度公式【计算题】

水平方向变化：

$$  
I_x(x,y)=I(x+1,y)-I(x,y)  
$$

垂直方向变化：

$$  
I_y(x,y)=I(x,y+1)-I(x,y)  
$$

梯度 magnitude：

$$  
m(x,y)=\sqrt{I_x(x,y)^2+I_y(x,y)^2}  
$$

梯度 orientation：

$$  
\theta(x,y)=\tan^{-1}\left(\frac{I_y(x,y)}{I_x(x,y)}\right)  
$$

更准确写法通常用：

$$  
\theta(x,y)=\operatorname{atan2}(I_y(x,y),I_x(x,y))  
$$

变量含义：

|符号|含义|
|---|---|
|$$I(x,y)$$|图像在 $$(x,y)$$ 的亮度|
|$$I_x$$|x 方向亮度变化|
|$$I_y$$|y 方向亮度变化|
|$$m$$|梯度强度，表示变化有多强|
|$$\theta$$|梯度方向，表示变化朝哪个方向|

直觉：

```text
raw intensity = 这个点有多亮
gradient = 这个地方的亮度怎么变化
```

descriptor 使用 gradient 后，不再那么依赖整体亮度。

---

# 7. Gradient histogram

## 7.1 基本思想【高频】

把 patch 中每个像素的 gradient orientation 分到若干 bins 中，然后把对应 magnitude 加进去。

比如 8 bins，每个 bin 覆盖 45°：

```text
0°–45°
45°–90°
90°–135°
...
315°–360°
```

老师在 transcript 中详细解释：根据 orientation 所在范围，把 magnitude 累加到对应 bin，最后得到一个 8-dimensional vector。

## 7.2 公式

设有 $$B$$ 个 orientation bins：

$$  
h_b=\sum_{(x,y)\in W} w(x,y),m(x,y),\mathbf{1}[\theta(x,y)\in \text{bin }b]  
$$

变量含义：

|符号|含义|
|---|---|
|$$h_b$$|第 $$b$$ 个 histogram bin 的值|
|$$W$$|当前 patch 或 spatial cell|
|$$w(x,y)$$|Gaussian weighting|
|$$m(x,y)$$|梯度 magnitude|
|$$\theta(x,y)$$|梯度方向|
|$$\mathbf{1}[\cdot]$$|条件成立为1，否则为0|
|$$B$$|bin 数量，SIFT 常用 8|

## 7.3 为什么 histogram 有用？

它不关心某个具体像素一定在哪里，而关心这个 patch 中“有哪些方向的边缘/纹理结构”。

所以它比 raw patch 更 robust：

```text
raw patch:
    每个像素位置都要对齐

gradient histogram:
    只统计局部方向分布，允许小偏移
```

---

# 8. Gaussian weighting 的作用【老师强调】【易错点】

老师强调 Gaussian weighting 可以缓解两个问题：

1. patch 边缘处的像素不稳定，给低权重；
    
2. descriptor 更关注 keypoint 中心附近，提高 localisation 稳定性。
    

公式：

$$  
w(x,y)=\exp\left(-\frac{(x-x_0)^2+(y-y_0)^2}{2\sigma^2}\right)  
$$

变量含义：

|符号|含义|
|---|---|
|$$(x_0,y_0)$$|keypoint 中心|
|$$(x,y)$$|patch 内像素位置|
|$$\sigma$$|Gaussian scale，控制权重下降速度|
|$$w(x,y)$$|距离中心越近越大，越远越小|

直觉：

```text
越靠近 interest point 中心，越可信；
越靠近 patch 边界，越可能因为小位移/旋转/背景变化而不稳定。
```

---

# 9. Rotation invariance：dominant orientation alignment

gradient histogram 本身不是天然 rotation invariant，因为图像旋转后，orientation bin 会整体移动。老师特别说：gradient histogram depends on the angles；解决方法是像 chain code 一样做 circular rotation，把 dominant vector / dominant orientation 对齐到固定位置。

直觉：

```text
原图主方向是 90°
旋转后主方向变成 150°

如果直接比较 histogram，会觉得不同；
如果先把主方向对齐，再比较，就会相似。
```

考试表达：

```text
To achieve rotation invariance, orientations are measured relative to the dominant orientation of the local patch rather than the global image axes.
```

---

# 10. SIFT descriptor【高频】【计算题】【老师强调】

SIFT = **Scale Invariant Feature Transform**。

handout 明确说，SIFT 建立在 local gradient histogram 上，并加入 spatial binning：标准 SIFT 使用 $$4\times4$$ spatial grid，每个 cell 有 8 个 orientation bins，所以得到 128-dimensional descriptor。

## 10.1 SIFT descriptor 维度

$$  
4\times4\times8=128  
$$

变量含义：

|项|含义|
|---|---|
|$$4\times4$$|patch 被分成 16 个 spatial cells|
|$$8$$|每个 cell 中的 orientation histogram 有 8 个 bins|
|$$128$$|拼接后 descriptor 的总维度|

老师在 transcript 中也反复讲了这个计算：16 patches × 8 orientations = 128 numbers。

## 10.2 SIFT descriptor 构造流程【必须会写 pseudocode】

```text
Input:
    image I
    keypoint p = (x, y, scale σ, orientation θ0)

Output:
    128-dimensional SIFT descriptor f

Algorithm:
    1. Around keypoint p, take a local patch at the detected scale.
    2. Compute image gradients Ix and Iy.
    3. For each pixel in the patch:
          compute magnitude m = sqrt(Ix^2 + Iy^2)
          compute orientation θ = atan2(Iy, Ix)
    4. Apply Gaussian weighting:
          pixels closer to the keypoint centre have higher weight.
    5. Rotate orientations relative to the dominant orientation θ0.
    6. Split the patch into a 4 × 4 grid.
    7. For each grid cell:
          build an 8-bin gradient orientation histogram.
    8. Concatenate all 16 histograms:
          descriptor dimension = 4 × 4 × 8 = 128.
    9. Optionally normalise the descriptor.
```

## 10.3 SIFT 为什么强？

SIFT 是一堆思想的组合，而不是单一 trick。老师最后总结说：SIFT 把 rotation、scale、brightness change、gradient histogram 等 ideas 结合起来；好的算法通常是多个 ideas 的 integration。

SIFT 的鲁棒性来源：

|问题|SIFT 的处理|
|---|---|
|brightness change|使用 gradient 而非 raw intensity|
|rotation|dominant orientation alignment|
|small localisation error|Gaussian weighting + spatial histogram|
|scale change|keypoint scale / scale-space detector|
|ambiguity|descriptor 更 distinctive|
|matching error|ratio test|

---

# 11. Feature matching

## 11.1 Feature-space distance【高频】

CV23-Summary 强调：feature extractor 应该让相似输入产生 close feature vectors；两个 vectors 的接近程度通过 feature-space distance 计算。

常用 Euclidean distance：

$$  
d(\mathbf{f},\mathbf{g})

 |\mathbf{f}-\mathbf{g}|_2

\sqrt{\sum_{k=1}^{D}(f_k-g_k)^2}  
$$

也可以用 squared Euclidean distance：

$$  
d^2(\mathbf{f},\mathbf{g})

\sum_{k=1}^{D}(f_k-g_k)^2  
$$

变量含义：

|符号|含义|
|---|---|
|$$\mathbf{f}$$|Image 1 中某个 keypoint 的 descriptor|
|$$\mathbf{g}$$|Image 2 中某个 keypoint 的 descriptor|
|$$D$$|descriptor 维度，SIFT 中 $$D=128$$|
|$$f_k,g_k$$|第 $$k$$ 个 descriptor component|
|$$d$$|两个 descriptors 的距离，越小越相似|

## 11.2 Nearest neighbour matching

```text
For each descriptor f_i in image 1:
    compute distance to every descriptor g_j in image 2
    choose the g_j with smallest distance
```

公式：

$$  
j^*=\arg\min_j d(\mathbf{f}_i,\mathbf{g}_j)  
$$

问题：  
即使真实匹配不存在，例如 occlusion，算法仍然会强行找一个 closest match。所以仅仅 nearest neighbour 不够。

---

# 12. Thresholding and Lowe’s ratio test【高频】【计算题】【老师强调】

## 12.1 普通 threshold

如果 best distance 太大，说明匹配不可信，拒绝：

$$  
d_1 < T  
$$

其中：

|符号|含义|
|---|---|
|$$d_1$$|最近邻距离|
|$$T$$|人工设定或经验设定的 threshold|

问题：  
threshold 太严格，matches 太少；threshold 太宽松，wrong matches 太多。老师在 transcript 中也强调了这个 trade-off：提高 threshold 可以得到更好匹配，但 detected pairs 可能太少。

## 12.2 Ratio test

handout 第48页明确给出改进方法：对第一张图的每个 feature，在第二张图中找最近和第二近两个 features；只有当最近距离与第二近距离之比小于 threshold 时才接受；典型 threshold 是 0.8。

公式：

$$  
\frac{d_1}{d_2}<\tau  
$$

通常：

$$  
\tau=0.8  
$$

变量含义：

|符号|含义|
|---|---|
|$$d_1$$|best match 的距离|
|$$d_2$$|second-best match 的距离|
|$$\tau$$|ratio threshold，常用 0.8|
|$$d_1/d_2$$|越小代表 best match 越独特|

判断例子：

```text
Case A:
d1 = 6, d2 = 10
d1/d2 = 0.6 < 0.8
Accept

Case B:
d1 = 6, d2 = 7
d1/d2 = 0.857 > 0.8
Reject
```

直觉：  
不是只问“best match 好不好”，而是问“best match 是否明显比 second-best 好”。如果第一名和第二名差不多，说明这个点 ambiguous，容易误匹配。

---

# 13. 失败情况总结【高频】【易错点】

即使用 SIFT + ratio test，仍然可能失败。老师最后提到，有些 outliers 仍然会存在，因为背景改变、旋转、可见区域不同，导致局部区域完全不同。

典型失败情况：

1. **repetitive texture**  
    例如建筑窗户、砖墙、山脉纹理，很多 patch 都长得像，ratio test 可能拒绝很多点，也可能仍保留错误点。
    
2. **low-texture region**  
    平坦区域没有明显 gradient，descriptor 不 distinctive。
    
3. **large viewpoint change**  
    局部 patch 形状发生 projective deformation，SIFT 不一定能处理。
    
4. **occlusion**  
    一个点在另一张图中不可见，但 nearest neighbour 仍会强行找匹配。
    
5. **non-rigid deformation**  
    物体形变导致局部结构变化。
    
6. **illumination change too strong**  
    gradient 比 raw intensity 更鲁棒，但不是完全 invariant。
    
7. **too many features / large database**  
    SIFT matching 很慢。下一讲资料提到，一张 800×600 图可能有约 2000 个 DoG/SIFT descriptors，每个 SIFT 是 128 维，和数据库匹配时需要计算大量 feature distances。
    

---

# 14. 考试重点标注

## 【高频】

- local feature = keypoint + descriptor
    
- detector vs descriptor
    
- good descriptor: invariant / robust / distinctive
    
- narrow baseline vs wide baseline
    
- template matching 为什么只适合 narrow baseline
    
- gradient histogram
    
- SIFT = $$4\times4\times8=128$$
    
- Euclidean distance matching
    
- ratio test: $$d_1/d_2<0.8$$
    

## 【容易考】

解释为什么 raw pixel patch 不适合 wide baseline matching。  
标准答案要提：

```text
illumination change
rotation
scale change
localisation error
viewpoint change
occlusion
```

## 【计算题】

可能让你算：

1. SIFT descriptor 维度：  
    $$4\times4\times8=128$$
    
2. Euclidean distance：  
    $$d=\sqrt{\sum_k(f_k-g_k)^2}$$
    
3. Ratio test：  
    给 $$d_1,d_2,\tau$$，判断 accept/reject。
    
4. SSD matching：  
    给两个小 patch，计算 SSD，选最小。
    

## 【设计/解释题】

给一个任务：

```text
你要把两张有重叠区域的山景图拼成 panorama，设计 pipeline。
```

答案：

```text
1. Detect repeatable interest points in both images.
2. Extract robust descriptors, e.g. SIFT.
3. Match descriptors using Euclidean distance.
4. Apply ratio test to remove ambiguous matches.
5. Estimate geometric transform, e.g. affine/homography, using RANSAC.
6. Warp and blend images.
```

## 【对比题】

- narrow baseline vs wide baseline
    
- raw patch descriptor vs gradient histogram descriptor
    
- template matching vs SIFT matching
    
- threshold matching vs ratio test
    
- detector repeatability vs descriptor distinctiveness
    

## 【易错点】

1. **repeatable detector 不等于 distinctive descriptor**  
    detector 负责“同一个点能否被重复检测到”；descriptor 负责“这个点能否被正确认出来”。
    
2. **gradient histogram 不是天然 rotation invariant**  
    需要 dominant orientation alignment。
    
3. **nearest neighbour 不代表正确匹配**  
    如果真实点不可见，nearest neighbour 也会硬找一个最近的错误点。
    
4. **ratio test 用的是最近距离和第二近距离的比值**  
    对 Euclidean distance 来说，越小越好，所以接受条件是 $$d_1/d_2<\tau$$。
    
5. **SIFT descriptor 是 128 维，不是 16 维或 8 维**  
    因为 $$4\times4$$ cells，每个 cell 8 bins。
    

## 【老师强调】

老师明确说上节课的 detector 和 Gaussian / scale-space 相关内容很重要，需要 understand and remember。  
本讲中老师反复强调：features 必须变成固定长度数字向量；panorama 需要先 matching 再 alignment；如果 detector 不 repeatable 就没有 matching 的机会；descriptor 需要 invariant、robust、distinctive；SIFT 是多个 ideas 的组合，而不是一个单独技巧。

---

# 15. 考试风格题与答案思路

## 概念解释题 1

**Question:** What is a local feature descriptor, and why is it needed after detecting interest points?

答案思路：  
interest point 只告诉我们位置；descriptor 用固定维度向量描述该点周围 patch 的 appearance/structure。只有把 feature 表示成数字向量，才能用 distance 做 matching。可提 SIFT descriptor。

---

## 概念解释题 2

**Question:** Explain what it means for a descriptor to be invariant, robust and distinctive.

答案思路：

```text
invariant: expected transformations do not significantly change descriptor
robust: noise/localisation/illumination changes do not destroy it
distinctive: correct match is much closer than incorrect matches
```

---

## 概念解释题 3

**Question:** Why are gradient-based descriptors more robust to illumination change than raw pixel descriptors?

答案思路：  
raw pixel intensity 会随整体亮度变化而整体上升/下降；gradient 描述的是局部变化结构，整体 brightness offset 对局部变化影响较小。因此 gradient magnitude/orientation 更适合 matching。

---

## 对比题 1

**Question:** Compare narrow-baseline and wide-baseline matching.

答案思路：

|Narrow baseline|Wide baseline|
|---|---|
|camera positions close|camera positions/angles far apart|
|small displacement|large displacement/viewpoint change|
|patch appearance similar|patch appearance may differ|
|template/SSD works|needs robust descriptor, e.g. SIFT|
|stereo/video tracking|panorama/object recognition/image retrieval|

---

## 对比题 2

**Question:** Compare template matching and SIFT matching.

答案思路：

|Template matching|SIFT matching|
|---|---|
|compares raw patches|compares descriptors|
|simple|more complex|
|good for small displacement|good for wider baseline|
|sensitive to rotation/scale/brightness|more robust to those changes|
|search window based|feature-space nearest neighbour|

---

## 对比题 3

**Question:** Compare simple nearest-neighbour matching and ratio-test matching.

答案思路：  
nearest-neighbour 只找最小 distance，但可能 ambiguous；ratio test 比较 best 和 second-best，如果 best 没有明显更好就拒绝。ratio test 更 robust，可以减少 false matches。

---

## 计算/推导题 1

**Question:** A SIFT descriptor uses a $$4\times4$$ grid and 8 orientation bins per grid cell. What is the descriptor dimension?

答案：

$$  
4\times4\times8=128  
$$

答案解释：  
patch 被分为 16 个 cells，每个 cell 有 8-bin histogram，拼接后为 128-dimensional vector。

---

## 计算/推导题 2

**Question:** For a feature descriptor, the closest distance is $$d_1=5.2$$ and the second closest distance is $$d_2=7.0$$. Using a ratio threshold $$\tau=0.8$$, should the match be accepted?

计算：

$$  
\frac{d_1}{d_2}=\frac{5.2}{7.0}=0.743  
$$

因为：

$$  
0.743<0.8  
$$

所以 accept。

如果改成 $$d_1=5.2,d_2=6.0$$：

$$  
5.2/6.0=0.867>0.8  
$$

则 reject，因为 best 和 second-best 太接近，说明 ambiguous。

---

## 设计应用题 1

**Question:** Design a pipeline for building a panorama from two overlapping images.

答案思路：

```text
1. Detect repeatable keypoints in both images, e.g. DoG/Harris.
2. Extract local descriptors, e.g. SIFT.
3. Match descriptors using Euclidean distance.
4. Apply ratio test to reject ambiguous matches.
5. Estimate geometric transform, e.g. homography, using matched pairs.
6. Use RANSAC to remove outliers.
7. Warp images and blend them.
```

要点：  
本讲负责 1–4，下一讲负责 geometric consistency / RANSAC。

---

## 设计应用题 2

**Question:** You need to match two images of the same object taken from very different viewpoints and under different lighting. Would you use raw template matching? Explain your choice.

答案思路：  
不建议 raw template matching。因为 wide baseline 下存在 rotation、scale、viewpoint、illumination、occlusion、large displacement。应使用 repeatable detector + robust descriptor，例如 SIFT；用 gradient histogram 和 dominant orientation alignment 获得更好的 robustness；用 ratio test 删除 ambiguous matches。

---

# 16. 考前速记版

## 5–10 个 must-know points

1. **Local feature = interest point + descriptor**。
    
2. **Detector** 负责找点；**descriptor** 负责描述点；**matcher** 负责找 correspondence。
    
3. 好 detector 要 **repeatable**；好 descriptor 要 **invariant, robust, distinctive**。
    
4. raw RGB / pixel patch 对 brightness、rotation、localisation error 很敏感。
    
5. narrow baseline 可用 template/SSD；wide baseline 需要 SIFT-like descriptor。
    
6. gradient descriptor 用 $$I_x,I_y,m,\theta$$ 描述局部变化，而不是绝对亮度。
    
7. Gaussian weighting 让中心更重要，减少边界和定位误差影响。
    
8. SIFT 标准维度：$$4\times4\times8=128$$。
    
9. Matching 常用 Euclidean distance。
    
10. Ratio test：$$d_1/d_2<0.8$$，用于拒绝 ambiguous matches。
    

## 3–5 个 high-risk mistakes

1. 把 detector 和 descriptor 混为一谈。
    
2. 以为 nearest neighbour 一定是正确匹配。
    
3. 忘记 ratio test 是和 second-best 比，不只是看 best distance。
    
4. 说 gradient histogram 天然 rotation invariant；更准确是需要 dominant orientation alignment。
    
5. 忘记 SIFT 仍可能有 outliers，下一讲还需要 geometric constraints / RANSAC。
    

## 3–5 个 likely exam questions

1. Explain why a good local descriptor should be invariant, robust and distinctive.
    
2. Compare narrow-baseline and wide-baseline matching.
    
3. Describe the construction of a SIFT descriptor and explain why it is 128-dimensional.
    
4. Given two nearest-neighbour distances, apply Lowe’s ratio test.
    
5. Design a panorama stitching pipeline using local features and matching.
    

## 最应该背/理解的公式与流程

### Gradient

$$  
I_x(x,y)=I(x+1,y)-I(x,y)  
$$

$$  
I_y(x,y)=I(x,y+1)-I(x,y)  
$$

$$  
m(x,y)=\sqrt{I_x^2+I_y^2}  
$$

$$  
\theta(x,y)=\operatorname{atan2}(I_y,I_x)  
$$

### SIFT 维度

$$  
4\times4\times8=128  
$$

### Euclidean distance

$$  
d(\mathbf{f},\mathbf{g})=  
\sqrt{\sum_{k=1}^{D}(f_k-g_k)^2}  
$$

### Ratio test

$$  
\frac{d_1}{d_2}<\tau,\qquad \tau\approx0.8  
$$

### Matching pipeline

```text
Detect repeatable keypoints
  ↓
Extract invariant/robust/distinctive descriptors
  ↓
Compute descriptor distances
  ↓
Nearest-neighbour matching
  ↓
Apply ratio test
  ↓
Matched point pairs
  ↓
Geometric verification in next lecture
```