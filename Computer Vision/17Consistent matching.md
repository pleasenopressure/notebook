
# 2. 这节课在 Computer Vision pipeline 中的位置

这节课的核心问题是：

> 已经通过 local interest points 和 descriptors 找到了一批 candidate matches，但里面一定会有 wrong matches / outliers。如何利用 **geometric consistency** 把错误匹配去掉？

前一讲 **Local features and matching** 解决的是：

```text
Image 1, Image 2
   ↓
Detect local interest points
   ↓
Extract descriptors, e.g. SIFT
   ↓
Compare descriptors by feature-space distance
   ↓
Candidate matches
```

但 descriptor matching 只看局部外观，很容易因为 occlusion、repeated texture、noise、similar patches 产生 mismatches。老师在 transcript 里强调：即使用了 best match / second-best ratio test，仍然可能留下错误匹配，因为某些点在另一张图中可能被遮挡或根本不存在。

本讲加入的是：

```text
Candidate matches
   ↓
Assume a global geometric mapping
   ↓
Estimate transform: Affine / Homography
   ↓
Check which matches follow the transform
   ↓
Inliers + outliers
   ↓
Use inliers for alignment / stitching / recognition / AR / 3D reconstruction
```

所以这节课在 pipeline 中的位置是 **matching verification / geometric verification**。它不是重新提取 feature，而是在已有 matches 上加一个 **constraint**：正确匹配应该共同服从同一个几何变换。这个 “constraints” 正好对应 CV23-Summary 里的后半课程关键词。

---

# 3. 核心复习笔记

## 3.1 为什么 descriptor matching 之后还会错？

【高频】【容易考】【老师强调】

即使 local descriptor 很好，也仍然会错。原因是 feature distinctiveness 有 trade-off：

- 如果 descriptor 太 distinctive，它对 noise、illumination change、小变形太敏感，导致本来应该匹配的点匹配不上。
    
- 如果 descriptor 不够 distinctive，它会把很多相似区域都匹配上。
    
- Occlusion 时，某个点在另一张图中根本不存在，但算法仍然会强行找 nearest neighbour。
    
- Repeated patterns，例如窗户、砖块、山石纹理，也会造成错误匹配。
    

handout 明确说：即使最先进的 local feature 也可能被 mismatched，而且 distinctiveness 永远有 trade-off。

直觉：

> Descriptor matching 像“看脸找人”，但只看局部 patch；Consistent matching 像“看这些人站的位置是否整体符合一个队形变化”。

---

## 3.2 Constrained matching 的核心假设

【高频】【设计/解释题】【老师强调】

假设我们已经有一批 point correspondences：

```text
p_i = (x_i, y_i) in image 1
p'_i = (x'_i, y'_i) in image 2
```

Consistent matching 的假设是：

> 正确匹配点应该大致服从同一个 geometric mapping。

例如 panorama stitching 中，一张图可以通过某个 transform 映射到另一张图。正确的 matches 映射后应该落在对应位置附近；错误 match 映射后会明显偏离。老师在 transcript 中反复用 image stitching 解释：大多数正确点应该 follow global transform，而 mismatch 不会 follow 这个 transform。

所以本讲的核心流程是：

```text
Given many tentative matches
Estimate a transform T
For each match:
    project point from image 1 to image 2
    compare predicted position with observed matched position
    small error → inlier
    large error → outlier
```

---

# 4. Geometric transforms

## 4.1 Point transform

handout 中先抽象成：

```text
x' = T x
```

其中：

- $$x$$：original coordinate，原图中的点；
    
- $$x'$$：transformed coordinate，变换后的点；
    
- $$T$$：transform matrix，几何变换矩阵。
    

---

## 4.2 Affine Transform

【高频】【计算题】【容易考】

Affine Transform 定义为：

$$  
\mathbf{x}' = A\mathbf{x} + \mathbf{b}  
$$

展开为：

$$  
\begin{bmatrix}  
x' \  
y'  
\end{bmatrix}

\begin{bmatrix}  
a_{11} & a_{12} \  
a_{21} & a_{22}  
\end{bmatrix}  
\begin{bmatrix}  
x \  
y  
\end{bmatrix}  
+  
\begin{bmatrix}  
b_1 \  
b_2  
\end{bmatrix}  
$$

变量含义：

- $$(x,y)$$：image 1 中的点坐标；
    
- $$(x',y')$$：image 2 中的对应点坐标；
    
- $$A$$：2×2 linear part，控制 rotation、scale、aspect ratio、shear；
    
- $$\mathbf{b}$$：translation offset，控制平移；
    
- $$a_{11},a_{12},a_{21},a_{22},b_1,b_2$$：一共 6 个 unknown parameters。
    

通常写成 homogeneous coordinates：

$$  
\begin{bmatrix}  
x' \  
y' \  
1  
\end{bmatrix}

\begin{bmatrix}  
a_{11} & a_{12} & b_1 \  
a_{21} & a_{22} & b_2 \  
0 & 0 & 1  
\end{bmatrix}  
\begin{bmatrix}  
x \  
y \  
1  
\end{bmatrix}  
$$

handout 明确指出 affine transform 可以写成一个单独的 transform matrix，需要给 vector 加一维。

Affine Transform 的自由度：

```text
6 DoF = translation + rotation + scale + aspect ratio + shear
```

handout page 18 总结为 6 DoF。

### 最少需要几个匹配点？

一个 point match 提供两个方程：

$$  
x'_i = a_{11}x_i + a_{12}y_i + b_1  
$$

$$  
y'_i = a_{21}x_i + a_{22}y_i + b_2  
$$

Affine 有 6 个 unknowns，所以最少需要：

$$  
\frac{6}{2}=3  
$$

即 **3 对 point matches**。

老师在 transcript 中特别问了这个问题：Affine 有 6 个 unknown parameters，所以需要 6 个 equations；每个 match 给出 x 和 y 两个方程，因此需要 3 个 point matches。

---

## 4.3 Similarity Transform vs Affine Transform

【对比题】【易错点】

Similarity Transform 是 Affine 的限制版：

```text
Similarity Transform: translation + rotation + uniform scale
Affine Transform: translation + rotation + scale + aspect ratio + shear
```

区别：

|Transform|DoF|能做什么|保留什么|不能处理什么|
|---|--:|---|---|---|
|Similarity|4|平移、旋转、统一缩放|形状比例基本不变|aspect ratio change、shear、perspective|
|Affine|6|平移、旋转、缩放、aspect ratio、shear|直线仍是直线，平行线仍平行|strong perspective / keystone distortion|
|Homography|8|perspective/projective transform|平面上的投影关系|非平面大视差、复杂 3D parallax|

易错点：Affine 能 shear，但不能表达真实透视投影中“平行线汇聚”的效果。

---

## 4.4 Homography / Projective Transform

【高频】【容易考】【设计/解释题】

当图像存在 perspective effect，例如 projector keystone distortion、拍摄平面物体时相机角度不同，Affine 就不够了。这时需要 **Homography**，也叫：

- Planar Homography
    
- Projective Transformation
    
- Perspective Transformation
    

老师在 transcript 中用 projector 的 keystone distortion 举例：投影仪从上往下打到屏幕上，矩形会变成梯形；projector 里的 keystone correction 就是在校正这种 perspective distortion。

Homography 用 homogeneous coordinates 表示：

$$  
\begin{bmatrix}  
wx' \  
wy' \  
w  
\end{bmatrix}

# H  
\begin{bmatrix}  
x \  
y \  
1  
\end{bmatrix}

\begin{bmatrix}  
h_{11} & h_{12} & h_{13} \  
h_{21} & h_{22} & h_{23} \  
h_{31} & h_{32} & h_{33}  
\end{bmatrix}  
\begin{bmatrix}  
x \  
y \  
1  
\end{bmatrix}  
$$

归一化后：

$$  
x'=  
\frac{h_{11}x+h_{12}y+h_{13}}  
{h_{31}x+h_{32}y+h_{33}}  
$$

$$  
y'=  
\frac{h_{21}x+h_{22}y+h_{23}}  
{h_{31}x+h_{32}y+h_{33}}  
$$

变量含义：

- $$H$$：3×3 homography matrix；
    
- $$h_{ij}$$：homography parameters；
    
- $$w$$：homogeneous scale / normalisation factor；
    
- $$h_{31}, h_{32}$$：引入 perspective / keystone distortion 的关键项；
    
- 因为整体 scale 不重要，3×3 矩阵 9 个数只有 8 个 DoF。
    

### 最少需要几个匹配点？

Homography 有 8 个有效 unknowns，每个 point match 提供 2 个方程，所以最少需要：

$$  
\frac{8}{2}=4  
$$

即 **4 对 point matches**。transcript 中老师明确说：Affine 需要 3 个 matches，Homography 需要 4 个 matches。

---

# 5. Recovering a geometric mapping

## 5.1 Simultaneous equations

【计算题】【容易考】

如果有匹配点：

$$  
(x_i,y_i) \leftrightarrow (x'_i,y'_i)  
$$

可以把它们代入 transform equations，形成 simultaneous equations。

以 Affine 为例：

$$  
x'_i = a_{11}x_i+a_{12}y_i+b_1  
$$

$$  
y'_i = a_{21}x_i+a_{22}y_i+b_2  
$$

写成矩阵形式：

$$  
\mathbf{A}\theta = \mathbf{b}  
$$

其中：

- $$\theta = [a_{11},a_{12},b_1,a_{21},a_{22},b_2]^T$$；
    
- $$\mathbf{A}$$：由 input coordinates $$(x_i,y_i)$$ 构成；
    
- $$\mathbf{b}$$：由 target coordinates $$(x'_i,y'_i)$$ 构成。
    

handout 强调：可以通过 solving simultaneous equations 来 estimate transform matrix；Affine 至少 3 个 matches，Homography 至少 4 个 matches。

---

## 5.2 Least-squares solution

【高频】【计算题】【易错点】

实际中 matches 通常多于最小数量，而且有 noise，所以系统是 overdetermined system。不能指望所有点完全满足方程，只能找一个误差最小的 transform。

定义 residual：

$$  
r_i = \left| \mathbf{x}'_i - T\mathbf{x}_i \right|_2  
$$

其中：

- $$\mathbf{x}'_i$$：observed matched point；
    
- $$T\mathbf{x}_i$$：由 transform 预测出的 point；
    
- $$r_i$$：observed 和 predicted 之间的 Euclidean distance。
    

Least-squares 最小化：

$$  
SSE = \sum_i r_i^2  
$$

目标：

$$  
T^* = \arg\min_T \sum_i \left| \mathbf{x}'_i - T\mathbf{x}_i \right|_2^2  
$$

handout 用 line of best fit 解释 SSE：residual 是 observed 和 predicted 的差，SSE 是 residual squared 的和。

### Least-squares 的问题

Least-squares 对 outliers 很敏感。一个很远的错误点会被平方误差放大，导致最终 transform 被 outlier “拉偏”。老师在 transcript 里明确说：如果有 outliers，least-squares 的 final solution 会被严重 biased。

---

# 6. RANSAC

## 6.1 RANSAC 的核心思想

【高频】【设计/解释题】【老师强调】

RANSAC = **RAndom SAmple Consensus**。

它解决的问题是：

> 在一堆 candidate matches 中，不知道哪些是 inliers、哪些是 outliers。那就反复随机抽最小样本，估计模型，然后看有多少其他点支持这个模型。

handout 的 RANSAC algorithm 是：

```text
Assume:
M data items required to estimate model T
N data items in total

1. Select M data items at random
2. Estimate model T
3. Count how many of the N data items fit T within tolerance Tol
   These are inliers; others are outliers
4. If K is large enough:
      accept T, or recompute least-squares using all inliers
      exit success
5. Repeat for n iterations
6. Fail: no good T found
```

这是 handout page 32 的核心算法。

变量含义：

- $$M$$：估计模型所需的最少点数。Affine 中 $$M=3$$；Homography 中 $$M=4$$。
    
- $$N$$：总 candidate matches 数量。
    
- $$T$$：估计出来的 transform。
    
- $$Tol$$：residual threshold，判断某个点是否符合模型。
    
- $$K$$：inliers 数量，也叫 consensus size。
    
- $$n$$：最大迭代次数。
    
- $$r_i$$：第 $$i$$ 个 match 的 residual。
    

---

## 6.2 RANSAC 的直觉

想象你有 100 对 matches，其中 70 对正确，30 对错误。

Least-squares 会把 100 对都拿来拟合，所以 30 个错误点会污染结果。

RANSAC 不相信所有点，而是：

```text
随机抽 4 对点
估计 homography
检查剩下所有点是否也支持这个 homography
如果很多点都支持 → 这 4 对点大概率是正确点
如果只有很少点支持 → 这 4 对点可能包含 outlier，丢掉重试
```

老师在 transcript 中强调：如果随机抽到的点不可靠，就回去重新随机抽；重复 n 次；如果一直找不到好的 transform，就宁愿 fail，也不要给出错误结果。

---

## 6.3 RANSAC vs Least-squares

【高频】【对比题】

|方法|思路|优点|缺点|适合情况|
|---|---|---|---|---|
|Least-squares|用所有 matches 最小化 SSE|简单、快速、数学清楚|对 outliers 极其敏感|matches 基本都正确，noise 小|
|RANSAC|随机抽最小集合，找最大 consensus|能识别 inliers/outliers，robust|随机、不稳定；outlier 太多会失败；依赖 threshold 和 iterations|matches 中有较多错误点|

老师在 transcript 中说：没有 outliers 时，least-squares 和 RANSAC 结果相似；但有 outlier 时，least-squares 会被 outlier 拉偏，而 RANSAC 可以通过 consensus 排除它。

---

# 7. Direct local feature matching 的效率问题

【容易考】【设计题】【老师强调】

这节课后半部分还讲了 nearest neighbour search 的效率问题。

一个典型图像可能有：

```text
~2000 DoG/SIFT interest points
Each SIFT descriptor = 128 dimensions
```

如果两张图各有 2000 个 descriptors，暴力匹配需要比较：

$$  
2000 \times 2000 = 4,000,000  
$$

对 descriptor distance 来说，每次比较又是 128 维，所以大致有：

$$  
4,000,000 \times 128 = 512,000,000  
$$

个维度级比较量。

handout 明确指出：local feature matching is slow，典型 800×600 图像可能有约 2000 个 DoG/SIFT descriptors，每个 SIFT descriptor 是 128 dimensions；直接计算 every query feature 与 every other feature 的距离非常昂贵。

---

## 7.1 K-D Tree

K-D Tree 的思想：

```text
Partition feature space using axis-aligned hyperplanes
Build a binary tree
For query point:
    walk down tree to a leaf
    brute-force search inside that leaf
    backtrack only when neighbouring partitions may contain closer points
```

handout 说 K-D Tree 是一种 binary tree structure，用 axis-aligned hyperplanes 分割空间，通常按 median split。

缺点：

- 对 low-dimensional data 比较有效；
    
- 对 high-dimensional descriptors，例如 SIFT 128D，效果会变差；
    
- 可能最终还是要搜索大部分 tree。
    

老师 transcript 也强调：K-D Tree 在 low-dimensional data 中还可以，但 high-dimensional data 中可能很低效。

---

## 7.2 Hashing / LSH / Sketching

【容易考】【低优先级但可作为设计题加分点】

Hashing 的方向是：

```text
Map similar vectors to similar hash codes
Search by hash code rather than full Euclidean search
```

LSH = **Locality Sensitive Hashing**。

handout 总结说：LSH creates hash codes for vectors such that similar vectors have similar hash codes；Sketching 可以把 binary hashes 拼接成 bit string，Hamming distance 可以近似 Euclidean distance，而且 bitwise operations 很便宜。

---

# 8. 典型应用场景

【设计/解释题】

Consistent matching 用在：

1. **Image stitching / panorama**  
    用 Homography 或 Affine 把图像对齐，RANSAC 去除错误 matches。
    
2. **Object recognition**  
    如果 object model 和 scene image 之间存在 consistent match，就可以认为目标被识别到。handout 提到 object recognition 中，object image matched against scene，consistent match 可以支持 recognition。
    
3. **Augmented Reality, AR**  
    找到平面目标和场景之间的 homography 后，可以把虚拟内容贴到真实场景上。
    
4. **3D reconstruction / SLAM**  
    多视角图像中的 point correspondences 可以用于 depth estimation、3D reconstruction、SLAM。handout 提到 matching local features 可用于 3D reconstruction 和 SLAM。
    
5. **Audio matching / Shazam-like system**  
    声音可以转成 spectrogram image，再提取 local features 匹配。handout 和 transcript 都提到 spectrogram / music matching。
    

---

# 9. 方法优点、缺点、失败情况

## Affine Transform

优点：

- 参数少，估计稳定；
    
- 只需要 3 对 matches；
    
- 对小视角变化、平面近似、图像局部对齐有用。
    

缺点：

- 不能表达强 perspective distortion；
    
- 不能处理明显 3D parallax；
    
- 多个物体独立运动时，一个 global affine 不够。
    

失败情况：

- 相机视角变化大；
    
- 场景不是单一平面；
    
- 近处和远处物体产生不同位移；
    
- 有 rolling shutter、non-rigid deformation。
    

---

## Homography

优点：

- 能处理 perspective / keystone distortion；
    
- 对 planar surface matching 很强；
    
- 图像拼接、AR、document rectification 常用。
    

缺点：

- 假设 scene approximately planar，或相机是 pure rotation；
    
- 对非平面场景、大视差、多层深度会失败；
    
- 需要至少 4 对可靠 matches。
    

失败情况：

- 场景里前景和背景深度差很大；
    
- 有 moving objects；
    
- match 点集中在很小区域，几何约束不稳定；
    
- 4 个点 nearly collinear，Homography 估计会退化。
    

---

## Least-squares

优点：

- 简单；
    
- 速度快；
    
- 当 outliers 很少时效果好；
    
- 可以在 RANSAC 找出 inliers 后作为 refinement。
    

缺点：

- 对 outlier 敏感；
    
- 不会自动区分 inlier/outlier；
    
- outlier 的大 residual 被平方后影响更大。
    

失败情况：

- mismatches 多；
    
- repeated texture；
    
- occlusion；
    
- 一个错误 match 离真实位置很远。
    

---

## RANSAC

优点：

- robust to outliers；
    
- 可以同时估计 transform 和识别 inliers/outliers；
    
- 实用性强，是 image alignment / stitching / AR / 3D 中常见基础方法。
    

缺点：

- 随机算法，每次结果可能不同；
    
- outlier ratio 太高时需要很多 iterations；
    
- threshold $$Tol$$ 不好选；
    
- 如果存在多个不同 motions / planes，一个模型可能不够；
    
- 如果随机样本退化，例如 homography 的 4 点共线，会估计失败。
    

老师在 transcript 中特别说：RANSAC 很有用，但如果 outliers 很多，每次结果可能不同，而且会变得 inefficient。

---

# 10. 考试重点标注

## 【高频】

1. 为什么 descriptor matching 后还需要 consistent matching？
    
2. Inlier / outlier 的定义。
    
3. Geometric constraint 的直觉。
    
4. Affine Transform 和 Homography 的公式。
    
5. Affine 最少 3 对点，Homography 最少 4 对点。
    
6. Least-squares 如何估计 transform。
    
7. Least-squares 为什么容易被 outlier 影响。
    
8. RANSAC 流程。
    

---

## 【容易考】

1. 给一堆 matches，让你解释如何去掉 mismatches。
    
2. 给场景选择 Affine 还是 Homography。
    
3. 比较 Least-squares 和 RANSAC。
    
4. 解释 K-D Tree / Hashing 为什么用于加速 nearest neighbour search。
    
5. 解释 feature distinctiveness trade-off。
    

---

## 【计算题】

1. Affine 有 6 个参数，每个 match 给 2 个方程，所以最少 3 个 matches。
    
2. Homography 有 8 个有效参数，每个 match 给 2 个方程，所以最少 4 个 matches。
    
3. 暴力匹配复杂度：若两张图各有 $$n$$ 和 $$m$$ 个 descriptors，每个 descriptor 维度 $$d$$，则大约是：
    

$$  
O(nmd)  
$$

例如：

$$  
2000 \times 2000 \times 128 = 512,000,000  
$$

---

## 【设计/解释题】

题目可能给你一个应用场景：

> 两张图片要拼接成 panorama，但 SIFT matches 中有很多错误点。设计一个 pipeline。

标准回答：

```text
1. Detect local interest points, e.g. DoG / Harris
2. Extract descriptors, e.g. SIFT
3. Match descriptors using Euclidean distance + ratio test
4. Use RANSAC to estimate Homography
5. Compute residuals for all matches
6. Keep inliers, reject outliers
7. Refit Homography using all inliers by least-squares
8. Warp and stitch images
```

---

## 【对比题】

重点对比：

- Affine vs Homography
    
- Similarity vs Affine
    
- Least-squares vs RANSAC
    
- Brute-force matching vs K-D Tree / Hashing
    
- Descriptor distinctiveness too high vs too low
    

---

## 【易错点】

1. **不要说 descriptor matching 本身就能保证正确匹配。**  
    它只能给 candidate matches，不保证 geometric consistency。
    
2. **不要把 Affine 和 Homography 混为一谈。**  
    Affine 保持平行线，Homography 可以处理 perspective。
    
3. **不要说 Least-squares 可以自动去掉 outliers。**  
    Least-squares 会被 outliers 影响；RANSAC 才是 robust estimation。
    
4. **不要忘记 Homography 的 homogeneous normalisation。**  
    最后要除以 $$w$$。
    
5. **不要把 RANSAC 说成 deterministic。**  
    它是 random sampling，结果受 iterations、threshold、outlier ratio 影响。
    

---

## 【老师强调】

1. 即使 best/second-best ratio test 之后，仍然会留下 mismatches。
    
2. 正确 matches 应该 follow one global transform；错误 matches 不 follow。
    
3. Affine 6 个参数，所以最少 3 对点；Homography 8 个有效参数，所以最少 4 对点。
    
4. Least-squares 有 outliers 时会被严重 bias。
    
5. RANSAC 是 practical and useful，但 outliers 太多时不稳定、低效。
    
6. Direct feature matching 很慢，2000×2000 个 features 会产生巨大比较量，因此需要 K-D Tree / Hashing 等加速方法。
    

---

# 11. 考试风格题与答案思路

## A. 概念解释题

### Q1. What is consistent matching, and why is it needed after local feature matching?

答案思路：

Consistent matching 是利用 geometric constraints 检查一批 feature matches 是否共同服从同一个 transform。Local feature matching 只比较 descriptor distance，可能因 noise、occlusion、repeated texture 产生 mismatches。Consistent matching 通过估计 Affine 或 Homography，把符合 transform 的 matches 作为 inliers，把不符合的作为 outliers。

---

### Q2. Explain the difference between inliers and outliers in geometric matching.

答案思路：

Inliers 是正确匹配点，它们经过估计的 transform 后，predicted position 和 observed matched position 的 residual 很小。Outliers 是错误匹配点，它们不服从同一个 geometric mapping，residual 大于 threshold。

公式：

$$  
r_i = \left| \mathbf{x}'_i - T\mathbf{x}_i \right|_2  
$$

若：

$$  
r_i < Tol  
$$

则为 inlier；否则为 outlier。

---

### Q3. What is the role of homogeneous coordinates in Homography?

答案思路：

Homogeneous coordinates 允许用 3×3 矩阵表示 projective transformation。Homography 计算后得到：

$$  
[wx', wy', w]^T  
$$

需要除以 $$w$$ 才得到真实 image coordinates：

$$  
x'=\frac{wx'}{w}, \quad y'=\frac{wy'}{w}  
$$

它可以表达 perspective distortion，例如 keystone distortion。

---

## B. 对比题

### Q4. Compare Affine Transform and Homography.

答案思路：

Affine 有 6 DoF，形式是：

$$  
\mathbf{x}'=A\mathbf{x}+\mathbf{b}  
$$

最少需要 3 对 points。它能表示 translation、rotation、scale、aspect ratio、shear，但不能处理 strong perspective。

Homography 有 8 DoF，使用 3×3 projective matrix，最少需要 4 对 points。它可以处理 planar perspective distortion，例如 image stitching、AR、document rectification，但对 non-planar scene / parallax 会失败。

---

### Q5. Compare Least-squares and RANSAC for estimating geometric transforms.

答案思路：

Least-squares 使用所有 matches，最小化 SSE：

$$  
SSE=\sum_i r_i^2  
$$

优点是简单快速，缺点是对 outliers 敏感。

RANSAC 随机抽最小样本估计模型，然后统计 consensus/inliers。优点是 robust to outliers，缺点是随机、依赖 threshold 和 iterations，outlier ratio 太高时会低效或失败。

---

### Q6. Compare brute-force matching and K-D Tree / Hashing.

答案思路：

Brute-force 需要计算每个 query descriptor 到所有 database descriptors 的距离，复杂度高：

$$  
O(nmd)  
$$

K-D Tree 通过空间划分减少搜索，但高维下效果变差。Hashing / LSH 把相似向量映射到相似 hash codes，可以加速 approximate nearest neighbour search，适合大规模检索。

---

## C. 计算 / 推导题

### Q7. How many point correspondences are needed to estimate an Affine Transform? Explain.

答案思路：

Affine Transform 有 6 个 unknowns：

$$  
a_{11}, a_{12}, a_{21}, a_{22}, b_1, b_2  
$$

每个 point match 给两个方程：

$$  
x'_i = a_{11}x_i+a_{12}y_i+b_1  
$$

$$  
y'_i = a_{21}x_i+a_{22}y_i+b_2  
$$

所以需要：

$$  
6/2 = 3  
$$

即最少 3 对 point correspondences。

---

### Q8. Two images each have 2000 SIFT descriptors. Each descriptor is 128-dimensional. Estimate the number of dimension-level comparisons for brute-force matching.

答案思路：

每个 descriptor pair 要比较 128 维。

descriptor pair 数量：

$$  
2000 \times 2000 = 4,000,000  
$$

维度级比较量：

$$  
4,000,000 \times 128 = 512,000,000  
$$

所以暴力匹配非常慢，需要 K-D Tree、Hashing、LSH 或 BoVW 等加速策略。

---

## D. 设计应用题

### Q9. Design a pipeline for panorama stitching when many local feature matches are wrong.

答案思路：

```text
1. Detect local interest points in both images
2. Extract local descriptors, e.g. SIFT
3. Match descriptors using Euclidean distance
4. Apply ratio test to remove ambiguous matches
5. Use RANSAC to estimate Homography
6. Compute residuals for all candidate matches
7. Keep inliers and remove outliers
8. Re-estimate Homography using all inliers with least-squares
9. Warp one image into the coordinate frame of the other
10. Blend / stitch
```

关键解释：

Panorama 通常存在 perspective relation，所以 Homography 比 Affine 更合适。

---

### Q10. You are building an AR system that overlays graphics on a poster. Which transform and matching method would you use?

答案思路：

Poster 是 planar object，因此使用 Homography。Pipeline：

```text
1. Detect features on poster template and camera frame
2. Extract descriptors
3. Match descriptors
4. Use RANSAC to estimate Homography
5. Reject outliers
6. Warp virtual graphic using estimated Homography
7. Overlay graphic onto poster plane
```

为什么不用普通 least-squares？

因为 live camera frame 中可能有 wrong matches、occlusion、lighting change，所以需要 RANSAC robust estimation。

---

# 12. 考前速记版

## Must-know points

1. Consistent matching = 用 **geometric constraints** 验证 local feature matches。
    
2. Descriptor matching 只产生 candidate matches，不保证正确。
    
3. 正确 matches 应该共同服从一个 **global transform**。
    
4. Affine Transform：6 DoF，最少 3 对点。
    
5. Homography：8 DoF，最少 4 对点。
    
6. Least-squares 最小化：
    

$$  
SSE=\sum_i r_i^2  
$$

7. Residual：
    

$$  
r_i=|\mathbf{x}'_i-T\mathbf{x}_i|_2  
$$

8. RANSAC = random sample → estimate model → count inliers → repeat。
    
9. RANSAC robust to outliers，但依赖 threshold、iterations、outlier ratio。
    
10. Direct SIFT matching 很慢：$$O(nmd)$$，可用 K-D Tree / Hashing / LSH 加速。
    

---

## High-risk mistakes

1. 把 **descriptor similarity** 当成 **geometric correctness**。
    
2. 忘记 Homography 最后要除以 $$w$$。
    
3. 说 Affine 可以处理所有 perspective distortion。
    
4. 说 Least-squares 能自动排除 outliers。
    
5. 忘记 RANSAC 是 random，不保证每次都一样。
    

---

## Likely exam questions

1. Explain how RANSAC removes mismatches in local feature matching.
    
2. Compare Affine Transform and Homography for image alignment.
    
3. Why is least-squares sensitive to outliers?
    
4. How many point matches are required for Affine / Homography and why?
    
5. Design a pipeline for image stitching using local features and robust matching.
    

---

## 最应该背的流程：RANSAC for Consistent Matching

```text
Input:
    Candidate matches between two images
    Model type T, e.g. Affine or Homography
    Minimal sample size M
    Threshold Tol
    Max iterations n

For iter = 1 to n:
    Randomly select M matches
    Estimate transform T
    For every match:
        Project point using T
        Compute residual
        If residual < Tol:
            mark as inlier
        Else:
            mark as outlier
    If number of inliers is large enough:
        Re-estimate T using all inliers by least-squares
        Return T and inliers

Fail if no good transform is found
```

这节课一句话记忆：

> **Local feature matching finds possible correspondences; consistent matching uses geometric constraints, usually Affine or Homography plus RANSAC, to keep inliers and reject outliers.**