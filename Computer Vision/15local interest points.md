
# COMP6223 Computer Vision：Local Interest Points 复习笔记

## 0. 本节课核心定位

这节课解决的问题是：

> 在图像中自动找到一些稳定、可重复、适合后续匹配的局部点。

这些点叫：

- **local interest point**
    
- **keypoint**
    
- **feature point**
    

它们会被用于后续的 **tracking, matching, image alignment, classification/search, robot navigation, 3D reconstruction** 等任务。handout 明确说，finding stable/repeatable interest points 是 modern computer vision 的关键问题。

在整个 pipeline 中，它的位置是：

```text
Image
  ↓
low-level processing / gradients / smoothing
  ↓
local interest point detection
  ↓
local feature descriptor extraction
  ↓
matching / alignment / retrieval / 3D reconstruction
```

本 lecture 主要讲 **detector**，也就是“在哪里找点”。下一讲 **Local features and matching** 才会讲 descriptor，也就是“如何描述这个点周围的内容”。transcript 中老师也强调，本节课讲 feature detector，下一讲会讲 local feature descriptors / feature vectors。

---

# 1. 什么是好的 Local Interest Point？

一个好的 interest point 必须满足：

1. **stable / repeatable**  
    同一个真实世界点，在不同图像中还应该被检测出来。
    
2. **invariant to brightness change**  
    亮度变化、局部光照变化后，点还应该存在。
    
3. **sufficient texture variation**  
    局部不能太平坦，否则没有可识别信息。
    
4. **not too repetitive**  
    纹理不能重复得太厉害。例如 checkerboard 上有很多强 corner，但它们太相似，容易 mismatch。
    
5. **invariant / robust to viewpoint, rotation, scale changes**  
    角度、位置、距离、旋转变化后还应尽量稳定。
    
6. **well-localised**  
    位置不能漂移。老师提到如果一个点一会儿出现、一会儿消失，或者位置 flickering，那就不是好 feature point。
    

CV23-Summary 中 Lecture 10–21 的核心线索是 **invariant, robust, repeatable, constraints**，这几个词正好就是本 lecture 判断 interest point 好坏的主线。

---

# 2. 本节课两类 detector

handout 明确说本节课 focus on two specific types：

1. **Corner detection：Harris and Stephens**
    
2. **Blob detection：Difference-of-Gaussian Extrema**
    

---

# 3. Corner 的直觉

## 3.1 Flat / Edge / Corner 的区别

Harris corner detector 的核心直觉是：

> 用一个小窗口观察局部区域。如果窗口向任意方向移动，窗口内 intensity 都发生明显变化，那么这里就是 corner。

三种情况：

```text
flat region:
窗口向任何方向移动，几乎没有变化
→ 不是好点

edge:
窗口沿 edge 方向移动，变化小
窗口垂直 edge 方向移动，变化大
→ 位置不唯一，不够好

corner:
窗口向任何方向移动，变化都大
→ 好 interest point
```

handout 第 7–8 页就是这个核心观察：flat region 没变化，edge 沿边方向没变化，corner 在所有方向都有显著变化。

---

# 4. Harris & Stephens Corner Detector

## 4.1 Harris 的目标

Harris 的目标不是直接识别“角”的形状，而是量化：

> 一个窗口和它轻微移动后的窗口相比，intensity 变化有多大？

如果所有方向变化都大，就是 corner。

---

## 4.2 Energy function：窗口移动前后差异

Harris 先定义一个窗口移动后的变化量：

$$
E(u,v)=\sum_{x,y} w(x,y)\left[I(x+u,y+v)-I(x,y)\right]^2
$$


变量含义：

- `I(x,y)`：图像在位置 `(x,y)` 的 intensity。
    
- `(u,v)`：窗口移动的位移。
    
- `w(x,y)`：weighting function，通常可以是 flat window 或 Gaussian window。
    
- `E(u,v)`：窗口移动后 intensity change 的总量。
    

直觉：

```text
E 很小：
窗口移动后看起来差不多
→ flat 或 edge 的某个方向

E 很大：
窗口移动后变化明显
→ 可能是 corner

所有方向 E 都大：
→ corner
```

---

## 4.3 为什么要 Gaussian weighting？

如果窗口边缘刚好有一个 corner，flat weighting 可能会把这个 corner 错误地定位到窗口边缘附近，localisation 不好。

Gaussian weighting 的思想是：

> 给窗口中心更大权重，让 detector 更倾向于把 corner 定位在窗口中心。

老师在 transcript 里补充说，Gaussian weighting 是为了让 corner point 更好地 localised 在 window centre。

---

## 4.4 Taylor expansion：为什么要近似？

直接计算 `E(u,v)` 很慢，因为你要对很多窗口、很多方向、很多 shift 做差异计算。Harris & Stephens 用一阶 Taylor expansion 把移动后的 intensity 近似掉。

公式：


$$
I(x+u,y+v)\approx I(x,y)+I_x(x,y)u+I_y(x,y)v
$$


变量含义：

- `I_x`：图像在 x 方向的 partial derivative / gradient。
    
- `I_y`：图像在 y 方向的 partial derivative / gradient。
    
- `u, v`：很小的位移。
    

代回去：


$$
E(u,v)\approx
\sum_{x,y}w(x,y)\left[I_x(x,y)u+I_y(x,y)v\right]^2
$$


写成矩阵形式：


$$
E(u,v)\approx
\begin{bmatrix}
u & v
\end{bmatrix}
M
\begin{bmatrix}
u\\
v
\end{bmatrix}
$$


其中：


$$
M=
\sum_{x,y}w(x,y)
\begin{bmatrix}
I_x^2 & I_xI_y\\
I_xI_y & I_y^2
\end{bmatrix}
$$


这个 `M` 叫：

- **Structure Tensor**
    
- **Second Moment Matrix**
    

handout 第 12 页说明，`M` 是 square symmetric matrix，用来编码 local intensity function 在 small shifts 下如何变化。

---

# 5. Structure Tensor 的 eigenvalues 怎么判断区域？

设 `M` 的两个 eigenvalues 是：


$$
\lambda_1,\lambda_2
$$


它们表示局部窗口在两个主方向上的变化强度。

判断规则：

```text
λ1 small, λ2 small
→ flat region

λ1 large, λ2 small
→ edge

λ1 small, λ2 large
→ edge

λ1 large, λ2 large
→ corner
```

原因：

- flat：所有方向都没变化。
    
- edge：只有垂直 edge 的方向变化大，沿 edge 的方向变化小。
    
- corner：两个主要方向变化都大。
    

handout 第 17 页明确给出：corner 是两个 eigenvalues 都大；edge 是一个远大于另一个；flat 是两个都小。

---

# 6. Harris Response Function

直接算 eigenvalues 还是有计算成本，所以 Harris 用 determinant 和 trace 近似判断。

Harris response：

 
$$
R=\det(M)-k(\operatorname{trace}(M))^2
$$
  

因为：

 
$$
\det(M)=\lambda_1\lambda_2
$$
 

 
$$
\operatorname{trace}(M)=\lambda_1+\lambda_2
$$
  

所以也可以写成：

 
$$
R=\lambda_1\lambda_2-k(\lambda_1+\lambda_2)^2
$$
  

变量含义：

- `R`：Harris corner response。
    
- `M`：structure tensor。
    
- `det(M)`：矩阵行列式。
    
- `trace(M)`：矩阵迹，也就是对角线元素之和。
    
- `k`：经验常数，通常在 `0.04–0.06` 左右。
    

判断：

```text
R 大于 0 且较大
→ corner

R 小于 0
→ edge

|R| 很小
→ flat
```

老师在 transcript 中解释，determinant 给出两个 eigenvalues 的乘积，trace 给出两个 eigenvalues 的和，因此不需要显式 eigendecomposition 就可以快速判断 corner / edge / flat。

---

# 7. Harris Detector 流程 / Pseudocode

```text
Input: image I
Output: corner points

1. Compute image gradients:
      Ix, Iy

2. For each pixel, compute:
      Ix^2, Iy^2, IxIy

3. Apply Gaussian weighting / smoothing to these products.

4. Build structure tensor M:
      M = [[sum(w Ix^2),  sum(w IxIy)],
           [sum(w IxIy),  sum(w Iy^2)]]

5. Compute Harris response:
      R = det(M) - k(trace(M))^2

6. Threshold:
      keep points with R > threshold

7. Local maxima selection:
      keep points whose R is larger than neighbouring pixels

8. Return remaining points as Harris corners.
```

---

# 8. Harris 的优点、缺点、失败情况

## 优点

- Fast。
    
- Lightweight。
    
- Stable。
    
- 适合 tracking / alignment / matching 的初始 keypoints。
    
- 不需要 object model，只需要 local gradients。
    

老师也说 Harris detector 很快、轻量、稳定，而且现在很多库都可以直接调用；但更重要的是理解它如何从 observation 发展成算法。

## 缺点

- 原始 Harris 不是天然 scale-invariant。
    
- 对 noise 敏感，需要 threshold 和 local maxima selection。
    
- 对重复纹理容易产生很多相似点，后续 matching 容易错。
    
- 对强 viewpoint change 不一定稳定。
    

## 失败情况

```text
flat wall / sky:
没有明显 gradient
→ 检测不到

long straight edge:
只有一个方向变化大
→ 不是好 corner

checkerboard:
corner 很多，但太相似
→ 容易 mismatch

scale change:
固定 window 在一个尺度可行，在另一个尺度可能失败
```

---

# 9. Scale Problem

## 9.1 什么是 scale problem？

同一个真实世界物体，在不同拍摄距离、不同 zoom 下，会在图像中占据不同 pixel size。

例如：

```text
远处拍一个人：
手臂只占很小区域，corner 可能能被固定窗口检测到。

近距离 zoom 到手臂：
同一个结构变得很大，原来的固定窗口可能看不到完整结构。
```

老师在 transcript 中明确说 scale problem 很重要：人知道场景尺度，但机器不知道，所以 detector 可能在一个尺度成功，在另一个尺度失败。

一句话：

> 固定大小的 detector window 只能适合某个尺度；真实图像中结构大小会变，所以需要 multi-scale。

---

# 10. Scale Space

## 10.1 Scale space 的定义

Scale space 的想法是：

> 不只看原图，而是看一系列不同 blur 程度的图像。

Gaussian scale space：

 
$$
L(x,y,t)=G(x,y,t)*I(x,y)
$$
 

Gaussian kernel：

 
$$
G(x,y,t)=\frac{1}{2\pi t}\exp\left(-\frac{x^2+y^2}{2t}\right)
$$
 

scale parameter：

 
$$
t=\sigma^2
$$
 

变量含义：

- `I(x,y)`：原始图像。
    
- `G(x,y,t)`：Gaussian kernel。
    
- `L(x,y,t)`：scale `t` 下的图像。
    
- `t`：scale parameter，也就是 Gaussian variance。
    
- `sigma`：Gaussian standard deviation。
    
- `*`：convolution。
    

---

## 10.2 “structures smaller than sqrt(t) are smoothed away” 是什么意思？

因为：

 
$$
t=\sigma^2
$$
 

所以：

 
$$
\sqrt{t}=\sigma
$$
 

直觉：

> 在 Gaussian smoothing 之后，小于大约 `sigma` 尺寸的细节会被模糊掉。

例如：

```text
t = 4
sqrt(t) = 2
→ 小于大约 2 pixels 的噪声、毛刺、细纹理会被 smoothing 掉。

t = 100
sqrt(t) = 10
→ 小于大约 10 pixels 的结构会被模糊掉，只保留更大的结构。
```

这不是说图像没了，而是说：

```text
small scale:
看 fine details

large scale:
fine details 被平滑掉
更关注整体结构 / context / large flow
```

老师在 transcript 中也说，大尺度时不再关心 fine details，而是看 scene 的 overall big changes / context。

---

# 11. 为什么 Gaussian filtering 后可以 downsample？

## 11.1 Downsample 本质

Downsample 就是少取点。例如：

```text
原图:
p1 p2 p3 p4 p5 p6 p7 p8

downsample by 2:
p1    p3    p5    p7
```

如果原图还有高频细节，比如黑白快速交替：

```text
黑 白 黑 白 黑 白 黑 白
```

直接 downsample 可能变成：

```text
黑 黑 黑 黑
```

这就是 aliasing。

---

## 11.2 为什么先 Gaussian blur 就可以？

Gaussian blur 是 low-pass filtering，会先去掉高频细节。去掉高频后，图像变化变慢，所以可以用更少的 pixels 表示。

直觉：

```text
blur 前:
黑白黑白黑白黑白
变化太快，不能少采样

blur 后:
灰灰灰灰灰灰灰灰
变化变慢，可以少采样
```

老师在 transcript 中把这和 Shannon sampling theorem 联系起来：low-pass filtering 以后，高频信息已经被去掉，此时只取 subset of pixels 不会额外丢掉当前 scale 下的重要信息。

关键 distinction：

> downsample 后不是和原图完全等价，而是和已经 Gaussian-filtered 的图像在当前尺度上等价。

---

## 11.3 为什么可以用同样大小的 window？

如果图像 downsample 到一半大小，那么 downsample 图像上的一个小窗口，对应原图中更大的区域。

例如：

```text
原图 1000×1000:
8×8 window 看小细节

downsample 到 500×500:
8×8 window 对应原图 16×16 区域

再 downsample 到 250×250:
8×8 window 对应原图 32×32 区域
```

所以不需要把 window 越做越大，只需要：

```text
Gaussian filtering
↓
downsampling
↓
same-size window
```

老师明确说，不用让 window bigger and bigger，而是 downsample 图像后用相同大小的 window，这样更快、内存更少。

---

# 12. Multi-scale Harris

Multi-scale Harris 的做法：

```text
Input: image I
Output: corners at multiple scales

1. Build Gaussian scale space / Gaussian pyramid.

2. For each scale:
      a. smooth image
      b. optionally downsample
      c. compute Harris response R
      d. threshold R
      e. keep local maxima

3. Combine detected points across scales.

4. Return points with position and scale.
```

handout 第 30 页说明，将 Harris 扩展到 scale-space 很直接：在固定的一组 scales 上计算 corner response，然后保留超过 threshold 的点。

输出可以理解成：

```text
(x, y, scale)
```

而不只是：

```text
(x, y)
```

---

# 13. Blob 的定义

在本节课中，blob 可以这样定义：

> Blob 是图像中一个局部、相对封闭的区域，这个区域内部某种属性近似一致，并且相对于周围背景有明显区别。

这些属性可以是：

- brightness / intensity
    
- colour
    
- texture
    
- filter response
    

handout 第 5 页也给出 blob 的定义：a region of an image in which some properties are constant or approximately constant。

直觉图：

```text
亮 blob:

背景背景背景背景
背景  亮亮  背景
背景  亮亮  背景
背景背景背景背景

暗 blob:

亮背景亮背景亮
亮背景 暗暗 背景
亮背景 暗暗 背景
亮背景亮背景亮
```

Blob 不一定是完美圆形，但通常是 compact local region。

---

# 14. 为什么二阶导数中 local minima / maxima 代表 blob？

## 14.1 Blob 像“小山包”或“小坑”

亮 blob 可以看成 brightness 曲线里的小山包：

```text
brightness
    ^
    |        /\
    |       /  \
    |______/    \______
              x
```

暗 blob 可以看成小坑：

```text
brightness
    ^
    |______      ______
    |      \    /
    |       \  /
    |        \/
              x
```

在中心处：

- 一阶导数 slope 大概是 0。
    
- 二阶导数 curvature 最强。
    
- 所以二阶导 response 在中心会出现强响应。
    

因此：

```text
亮 blob center
→ strong response of one sign

暗 blob center
→ strong response of opposite sign
```

具体正负取决于 filter 符号约定，但关键是：

> bright blob 和 dark blob 都会在二阶导 / LoG / DoG response 中形成 local extremum。

所以要找：

```text
local maximum or local minimum
```

而不是只找 maximum。

---

## 14.2 Edge detection 和 Blob detection 的区别

LoG 在 edge detection 中常找 zero-crossing；但在 blob detection 中找 extrema。

```text
edge:
brightness 从暗到亮
一阶导数最大
二阶导数过零点 zero crossing

blob:
brightness 先升后降，或先降后升
中心处曲率最强
二阶导数出现 local maximum / local minimum
```

老师在 transcript 中也说，LoG 原来可以用于 edge detection 找 zero crossings，但在 blob detector 中，不找 zero crossings，而是找 local maximum or local minimum。

---

## 14.3 LoG / DoG 像 centre-surround operator

LoG / DoG 可以直觉理解成：

> 比较中心区域和周围环形区域。

如果 filter 正好放在 blob 中心：

```text
filter centre 对准 blob centre
→ centre-surround contrast 最大
→ response 最大或最小
```

如果 filter 偏离中心：

```text
filter centre 偏了
→ 中心和周围结构不对齐
→ response 变弱
```

所以 blob center 自然就是 response map 中的 local extrema。

---

# 15. LoG Blob Detector

LoG 全称：

```text
Laplacian of Gaussian
```

它可以用于 blob detection。

二阶 Laplacian：

 
$$
\nabla^2 L = L_{xx}+L_{yy}
$$
```

Normalised LoG 常写成：

 
$$
t\nabla^2L(x,y,t)
$$
```

变量含义：

- `L(x,y,t)`：scale-space image。
    
- `L_xx`：x 方向二阶导。
    
- `L_yy`：y 方向二阶导。
    
- `nabla^2`：Laplacian operator。
    
- `t`：scale parameter。
    
- `t nabla^2 L`：scale-normalised LoG response。
    

在 scale-space 中寻找 extrema：

```text
local maximum / local minimum in (x, y, scale)
→ blob center and representative scale
```

---

# 16. DoG：Difference of Gaussian

## 16.1 为什么需要 DoG？

LoG scale space 计算贵，所以用 DoG 近似 LoG。

handout 第 36 页说，LoG scale space 在实践中计算昂贵，可以用 Difference-of-Gaussians 近似，也就是 subtracting adjacent scales of a Gaussian scale space。

---

## 16.2 DoG 公式

 
$$
D(x,y,\sigma)=L(x,y,k\sigma)-L(x,y,\sigma)
$$
```

也可以写成：

 
$$
D(x,y,\sigma)=\left[G(x,y,k\sigma)-G(x,y,\sigma)\right]*I(x,y)
$$
```

变量含义：

- `D(x,y,sigma)`：DoG response。
    
- `L(x,y,sigma)`：用 sigma 做 Gaussian smoothing 后的图像。
    
- `k`：相邻 scale 的比例。
    
- `G`：Gaussian kernel。
    
- `I`：原图。
    

直觉：

```text
轻微 blur 的图像
-
更强 blur 的图像
=
在这个 scale 范围内消失/变化明显的结构
```

如果某个 blob 的大小刚好对应这个 scale，它在 DoG 图里会特别突出。

---

## 16.3 DoG 和 LoG 的关系

常见近似关系：

 
$$
D(x,y,\sigma)\approx C\cdot\sigma^2\nabla^2L(x,y,\sigma)
$$
```

也就是说：

```text
DoG ≈ scale-normalised LoG
```

考试里不用纠结常数 `C`，重点是：

```text
DoG 用相邻 Gaussian blurred images 相减，近似 LoG，但计算更快。
```

---

# 17. DoG Blob Detector 流程 / Pseudocode

```text
Input: image I
Output: blob keypoints

1. Build Gaussian scale space:
      L(x, y, sigma1)
      L(x, y, sigma2)
      L(x, y, sigma3)
      ...

2. Compute DoG images:
      D(x, y, sigma_i) = L(x, y, sigma_{i+1}) - L(x, y, sigma_i)

3. Stack DoG images into a scale-space pyramid.

4. For each pixel in each DoG level:
      compare it with neighbours in:
        - same scale
        - scale above
        - scale below

5. If it is local maximum or local minimum:
      mark it as blob keypoint.

6. Return:
      (x, y, sigma)
```

老师在 transcript 中也总结：对原图用不同 sigma 做 Gaussian filtering，然后相减，再在这些结果中找 local minimum or maximum，就是 blob point。

---

# 18. Octave / Pyramid 的效率直觉

在 DoG pyramid 中，每一层已经做过 Gaussian filtering。进入下一 octave 时，可以复用前面结果，只需要 downsample，而不是从头重新做所有 Gaussian filtering。

老师说：

```text
current scale:
Gaussian filtering + subtraction

next octave:
很多结果可以 downsample 后复用
只需要少量额外 Gaussian filtering
```

这样 multi-scale blob detector 可以从 small scale 到 large scale 检测 blob，而且非常快，接近 real-time。

---

# 19. Harris vs DoG 对比

```text
Harris:
- 找 corner
- 基于 structure tensor
- 看窗口向不同方向移动时 intensity 是否都变化大
- 原始版本不是天然 scale-invariant
- multi-scale Harris 可以缓解 scale problem

DoG:
- 找 blob
- 基于 Gaussian scale space
- 用相邻 Gaussian blurred images 相减
- 找 scale-space extrema
- 更自然地处理 scale
- 后续和 SIFT keypoint detection 关系很强
```

handout summary 也总结：本节课看了 corners 和 blobs；Harris & Stephens 是常见 corner detector；multi-scale DoG pyramid 中找 extrema 提供 robust blob detector；scale space 可以找到不同大小的 features。

---

# 20. Detector vs Descriptor

非常容易考，也非常容易混。

```text
Detector:
回答 “where?”
在哪里有稳定点？
例如 Harris corner, DoG extrema

Descriptor:
回答 “what does it look like?”
这个点周围长什么样？
例如 SIFT descriptor
```

本节课是：

```text
interest point detection
```

下一讲是：

```text
local feature description and matching
```

下一讲 handout 也回顾说上一讲找 robust, stable, repeatable interest points，包括 Harris Corners 和 scale-space DoG extrema。

---

# 21. 常见考试题型整理

## 21.1 概念解释题

### Q1. What makes a good interest point?

答案要点：

```text
A good interest point should be stable, repeatable, robust and invariant to changes such as brightness, noise, rotation, viewpoint and scale. It should have sufficient local texture variation, but should not be too repetitive, otherwise matching becomes ambiguous.
```

中文：

```text
好的 interest point 要稳定、可重复、对亮度/噪声/旋转/视角/尺度变化鲁棒。局部要有足够纹理变化，但不能像 checkerboard 那样过于重复，否则容易 mismatch。
```

---

### Q2. Why is a corner better than an edge for matching?

答案要点：

```text
An edge is ambiguous along its own direction. Moving a window along the edge produces little change, so localisation is poor. A corner changes significantly in all directions, so it is better localised and more distinctive.
```

---

### Q3. What is a blob?

答案要点：

```text
A blob is a local image region whose properties, such as intensity, colour or texture, are approximately constant inside the region and different from the surrounding area. In LoG/DoG blob detection, it is detected as a local extremum in scale-space.
```

---

## 21.2 对比题

### Q4. Harris vs DoG

```text
Harris detects corners using local gradient changes and the structure tensor. It is fast and lightweight, but the original version is not scale-invariant. DoG detects blobs by subtracting adjacent Gaussian-smoothed images in scale-space and finding local extrema. DoG is more naturally suited to scale changes.
```

---

### Q5. LoG vs DoG

```text
LoG directly uses the Laplacian of Gaussian and detects extrema for blobs, but it is computationally expensive. DoG approximates LoG by subtracting adjacent Gaussian scales, making it faster and practical for scale-space detection.
```

---

### Q6. Detector vs Descriptor

```text
A detector finds where keypoints are. A descriptor describes the local appearance around each detected keypoint as a feature vector. Harris and DoG are detectors; SIFT descriptor is a descriptor.
```

---

## 21.3 计算题

### Q7. 给定 structure tensor，计算 Harris response

题目：

 
$$
M=
\begin{bmatrix}
5 & 0\\
0 & 4
\end{bmatrix},
\quad
k=0.04
$$
```

计算：

 
$$
\det(M)=5\times4-0=20
$$
```

 
$$
\operatorname{trace}(M)=5+4=9
$$
```

 
$$
R=20-0.04\times9^2=20-3.24=16.76
$$
```

结论：

```text
R > 0 且较大，所以是 corner。
```

---

### Q8. 给 eigenvalues 判断区域类型

```text
λ1 = 0.1, λ2 = 0.2
→ flat

λ1 = 100, λ2 = 2
→ edge

λ1 = 80, λ2 = 70
→ corner
```

---

### Q9. Scale transform

如果 blob 在原图中检测到：

 
$$
(x_0,y_0,t_0)
$$
```

图像被放大 `s` 倍后，对应位置和尺度变成：

 
$$
(sx_0,sy_0,s^2t_0)
$$
```

例子：

 
$$
(x_0,y_0,t_0)=(20,30,4), \quad s=2
$$
```

 
$$
(20,30,4)\rightarrow(40,60,16)
$$
```

---

## 21.4 设计题

### Q10. 设计 panorama matching pipeline

题目：

```text
You need to build a panorama from two overlapping images with possible scale and rotation changes. What feature strategy would you use?
```

答案思路：

```text
1. Use a repeatable local interest point detector.
2. Because scale changes may exist, use multi-scale DoG or multi-scale Harris.
3. Extract descriptors around each detected keypoint, e.g. SIFT.
4. Match descriptors between images.
5. Use geometric constraints / RANSAC to remove mismatches.
6. Estimate transformation and align images.
```

---

# 22. 本节课高频考点

## 【高频】

```text
1. Good interest point 的标准：
   stable, repeatable, robust, invariant

2. Harris 的核心直觉：
   window shift in any direction gives large intensity change

3. Structure tensor M：
   用 Ix, Iy 构造，描述 local intensity changes

4. Eigenvalue 判断：
   flat / edge / corner

5. Harris response：
   R = det(M) - k(trace(M))^2

6. Scale problem：
   固定窗口无法适应不同大小的结构

7. Scale space：
   Gaussian smoothing + 多尺度表示

8. DoG：
   adjacent Gaussian blurred images subtraction

9. Blob detection：
   find local extrema in scale-space

10. Detector vs Descriptor：
    detector 找点，descriptor 描述点
```

---

## 【容易考】

```text
1. 为什么 edge 不如 corner？
2. 为什么 Gaussian weighting 改善 localisation？
3. 为什么用 Taylor expansion？
4. 为什么 determinant 和 trace 可以替代 eigenvalue 计算？
5. 为什么 Gaussian filtering 后可以 downsample？
6. 为什么二阶导数 extrema 对应 blob？
7. 为什么 DoG 比 LoG 快？
```

---

## 【易错点】

```text
1. 不要把 Harris detector 说成 descriptor。
2. 不要说 edge 是最好的 interest point。
3. 不要说 structure tensor 就是 covariance matrix。
4. 不要说 DoG 找 zero-crossing；blob detection 找 extrema。
5. 不要忘记 scale-space 中 t = sigma^2。
6. 不要把 sqrt(t) 理解成“图像大小”，它对应 Gaussian smoothing 的尺度。
7. 不要说 downsample 完全不丢原图信息；准确说是对 low-pass filtered image 不额外丢当前尺度信息。
```

---

# 23. 老师强调内容

老师最强调的不是“如何调用 Harris API”，而是：

```text
observation
→ energy function
→ Taylor approximation
→ structure tensor
→ eigenvalue interpretation
→ Harris response
→ fast real-time detector
```

他明确说，现在很多库都可以直接调用 Harris；更重要的是理解它的 motivation、observation 和 development process。

另外老师强调：

```text
scale problem 很重要
```

因为机器不知道场景实际尺度，所以同一个 detector 在不同距离/zoom 下可能失败。

DoG 部分老师最后总结：

```text
本节课学了 Harris corner detector 和 DoG blob detector；
它们需要对 rotation、lighting、scale 等变化 robust；
DoG 会用于 SIFT；
scale-space 可以合并不同 scale 的 detector results，从而适应 scale problem。
```

---

# 24. 最后一页速记版

## Must-know points

```text
1. Local interest point = keypoint = feature point.
2. 好点必须 stable / repeatable / robust / invariant.
3. Harris 找 corner，DoG 找 blob.
4. Corner：window 向任意方向移动都有 large intensity change.
5. Edge：沿 edge 方向移动变化小，所以不够唯一.
6. Structure tensor M 编码局部 intensity 对 small shift 的变化.
7. 两个 eigenvalues 都大 → corner.
8. Harris response 用 det 和 trace 快速判断 corner.
9. Scale-space 用 Gaussian smoothing 表示不同尺度.
10. DoG 在 scale-space 中找 local extrema 来检测 blob.
```

---

## 最该背的公式

### Harris energy

 
$$
E(u,v)=\sum_{x,y} w(x,y)\left[I(x+u,y+v)-I(x,y)\right]^2
$$
```

### Taylor approximation

 
$$
I(x+u,y+v)\approx I(x,y)+I_xu+I_yv
$$
```

### Structure tensor

 
$$
M=
\sum_{x,y}w(x,y)
\begin{bmatrix}
I_x^2 & I_xI_y\\
I_xI_y & I_y^2
\end{bmatrix}
$$
```

### Harris response

 
$$
R=\det(M)-k(\operatorname{trace}(M))^2
$$
```

### Gaussian scale space

 
$$
L(x,y,t)=G(x,y,t)*I(x,y)
$$
```

### Scale parameter

 
$$
t=\sigma^2
$$
```

### DoG

 
$$
D(x,y,\sigma)=L(x,y,k\sigma)-L(x,y,\sigma)
$$
```

---

## 最该背的流程

```text
Harris:
image
→ compute Ix, Iy
→ build structure tensor M
→ compute R = det(M) - k(trace(M))^2
→ threshold
→ local maxima
→ corners
```

```text
DoG:
image
→ Gaussian scale space
→ subtract adjacent Gaussian scales
→ DoG pyramid
→ find local maxima/minima in x, y, scale
→ blobs
```

---

## 一句话总结整节课

> Local interest points 是图像中稳定、可重复、可匹配的局部位置。Harris 通过窗口移动后的 intensity change 找 corner；DoG 通过 scale-space 中的 Gaussian 差分 extrema 找 blob。Scale-space 的作用是让 detector 能适应不同大小的图像结构，从而提高 robustness 和 repeatability。