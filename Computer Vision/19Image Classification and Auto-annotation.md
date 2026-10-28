
# 2. 这节课在 Computer Vision pipeline 中的位置

这节课处在 pipeline 的高层：**Recognition / Scene Understanding / Semantic Interpretation**。

前面几讲解决的是：

```text
image pixels
→ features / descriptors
→ local features / matching
→ Bag of Visual Words
```

这一讲解决的是：

```text
image
→ feature representation, e.g. BoVW / PHOW / Dense SIFT histogram
→ classifier
→ class label(s), confidence, annotation keywords
```

输入是图像，或者图像中抽取出来的 descriptors。输出可以有不同层级：

```text
single label:        cat
multi-class label:   cat / dog / seal / horse
multi-label tags:    cat + dog
detection output:    dog + bounding box
scene/activity:      running / playing music / photographing
annotation keywords: sunset, sea, beach, road, sky
```

和前后 lecture 的关系：

- **前一讲 Large-scale Image Search / BoVW**：BoVW 在上一讲主要用于 retrieval/search；这一讲把 BoVW histogram 当作 feature vector 送进 classifier，用于 classification / auto-annotation。
    
- **更早的 Local Features / Matching**：SIFT、interest points、descriptor、codebook 都是这一讲的基础。
    
- **后一讲 Towards 3D Vision**：下一讲转向 3D understanding，例如 camera geometry、depth、stereo。这一讲仍然主要是 2D image semantic understanding。
    

老师在 transcript 中强调：classification 的基本形式就是 **extracting features + machine learning + recognition**；如果只是给图像打标签是 annotation，如果还要指出对象位置就是 object detection/localisation。

---

# 3. 系统复习笔记

## 3.1 核心概念

### 1. Image Classification

**Image classification** 是给整张图像预测一个类别。

例子：

```text
image → classifier → cat
image → classifier → dog
image → classifier → road scene
```

handout 第 2 页给出的基本框架就是：

```text
image goes in
→ Feature Extractor
→ Machine Learning
→ result comes out
```

这是本讲的核心骨架。

---

### 2. Binary / Multi-class / Multi-label

#### Binary classification

二分类只区分两个类别，例如：

```text
cat vs dog
real vs fake
car vs non-car
```

linear classifier 学的是 separating hyperplane，non-linear classifier 学的是 separating surface。

#### Multi-class classification

多分类是多个类别中选一个：

```text
cat / dog / seal / horse / human
```

通常可以用：

```text
one-vs-rest
one-vs-one
KNN
SVM
linear classifier
```

#### Multi-label classification / Automatic Annotation

一张图可以同时有多个标签：

```text
cat + dog
sky + road + person
sea + sunset + beach
```

handout 明确说：在图像语境下，**multi-label classification often called Automatic Annotation**。

---

### 3. Object Detection / Localisation

classification 只回答：

```text
What is in the image?
```

object detection / localisation 还要回答：

```text
Where is it?
```

例如：

```text
label = dog
bounding box = [x, y, w, h]
```

handout 第 6 页用 cat/dog 的区域框展示了 localisation。

【易错点】  
不要把 **auto-annotation** 和 **object detection** 混为一谈：

```text
auto-annotation: image contains dog, cat
object detection: dog is here, cat is there
semantic segmentation: every pixel belongs to road / sky / person / tree
```

---

### 4. Semantic Gap

**Semantic Gap** 是本讲最重要的概念之一。

低层机器看到的是：

```text
pixels
edges
corners
regions
histograms
SIFT descriptors
BoVW histograms
```

人类理解的是：

```text
a car accident
a dog lying beside a cat
a person riding a bike
a dangerous road scene
```

两者之间的差距就是 **semantic gap**。handout 把它称为 “The fundamental problem of computer vision”。

老师在 transcript 中举了一个很关键的例子：系统看到一辆倒置的车，却描述成 “a car parked on double yellow lines”。这个结果局部上没完全错，因为确实有 car 和 double yellow lines；但语义上错了，因为真正重要的信息是 **car is upside down / accident-like situation**。老师用这个例子说明：现代模型虽然进步很大，但仍然可能抓不住 unusual condition 和 scene meaning。

---

## 3.2 BoVW for Classification

### 基本思想

**Bag of Visual Words, BoVW** 的思想是：

```text
local descriptors
→ vector quantisation
→ visual words
→ count visual words
→ histogram representation
→ classifier
```

在上一讲 image search 中，BoVW 用来做 retrieval；这一讲中，BoVW histogram 被送入 classifier，用于 classification 和 annotation。handout 明确说 BoVW histogram representations 对 image classification 和 object detection 很有用，并且常和 fast linear classifiers / SVMs 一起使用。

---

### 为什么 classification 的 codebook 可以比 image search 小？

这是老师 transcript 里强调的高价值考试点。

在 image search / matching 中，我们希望精确地区分非常相似的 local patches，所以需要很大的 visual vocabulary。

但在 scene recognition / classification 中，我们不需要把每个小变化都区分得特别细：

```text
red car / yellow car / small car / large car
→ 对 classification 来说都可能只是 car
```

所以 feature 可以 **less distinctive**，允许一定 variation。handout 第 17 页也说：classification 的 codebook vocabulary 应比 image search 小，通常 few hundred 到 few thousand visual words 就够。 transcript 也重复了这一点：scene recognition 需要 smaller codebook、smaller vector、less distinctive features，以容纳 small variations。

---

## 3.3 Dense SIFT / PHOW

### Interest-point SIFT 的问题

如果只在 DoG interest points 上抽 SIFT：

优点：

```text
stable
repeatable
compact
```

缺点：

```text
coverage 不完整
flat regions / texture-poor regions 可能没有 feature
classification 需要理解整张图，而不是只匹配几个关键点
```

所以 handout 提出：local features around interest points work okay, but densely sampled features can work better。

---

### Dense SIFT

**Dense SIFT** 不再只在 interest points 上抽 SIFT，而是在规则网格上密集抽取 SIFT：

```text
image grid positions
→ extract SIFT at each grid location
→ many descriptors
→ quantise into visual words
→ histogram
```

handout 第 26 页明确说：Rather than extracting SIFT at DoG interest points, extract them across a dense grid, giving more coverage of the entire image。

---

### Pyramid Dense SIFT

**Pyramid Dense SIFT** 是在 Gaussian pyramid 的多个 scale 上抽 Dense SIFT。handout 第 27 页强调：sampling region 固定，因此在更高 scale 上，一个 patch 覆盖更多 image content。

直觉：

```text
small scale: capture local texture/detail
large scale: capture bigger structures/context
```

---

### PHOW

handout 给出的定义：

```text
PHOW = Pyramid Histogram of Words
     = Hist(VQ(Pyramid Dense SIFT)) + Spatial Pyramid
```

意思是：

```text
Dense SIFT descriptors
→ vector quantisation
→ visual words
→ histogram
→ add spatial pyramid information
→ final image feature vector
```

【易错点】  
BoVW 本身丢失空间位置；Spatial Pyramid 是为了部分保留空间结构。

---

## 3.4 关键公式

### 1. K-means codebook learning

给定训练图像中抽出的 descriptors：

[  
X = {x_1, x_2, ..., x_N}  
]

其中：

- (x_i)：第 (i) 个 local descriptor，例如 SIFT descriptor
    
- (N)：训练集中用于建 codebook 的 descriptor 数量
    
- (K)：visual words 的数量，也就是 codebook size
    
- (\mu_k)：第 (k) 个 cluster centre / visual word
    

K-means 学 codebook：

[  
\min_{\mu_1,\ldots,\mu_K} \sum_{i=1}^{N} \min_{k} |x_i - \mu_k|^2  
]

含义：每个 descriptor 分配给最近的 cluster centre，让总距离最小。

---

### 2. Vector Quantisation

对一个 descriptor (x_i)，找到最近的 visual word：

[  
q(x_i) = \arg\min_{k} |x_i - \mu_k|^2  
]

变量含义：

- (q(x_i))：descriptor (x_i) 被量化到的 visual word index
    
- (\mu_k)：第 (k) 个 visual word
    
- (|\cdot|)：通常是 Euclidean distance
    

---

### 3. BoVW histogram

对图像 (I)，假设抽出 descriptors：

[  
x_1, x_2, ..., x_n  
]

第 (k) 个 bin 的计数是：

[  
h_k(I) = \sum_{i=1}^{n} \mathbf{1}[q(x_i)=k]  
]

变量含义：

- (h_k(I))：图像 (I) 中第 (k) 个 visual word 出现的次数
    
- (n)：该图像抽出的 descriptor 数量
    
- (\mathbf{1}[\cdot])：条件成立为 1，否则为 0
    

通常会归一化：

[  
\hat{h}(I) = \frac{h(I)}{\sum_{k=1}^{K} h_k(I)}  
]

这样不同图像即使 feature 数量不同，也可以比较。

---

### 4. Linear classifier / OvR classifier

对类别 (c)，训练一个 one-vs-rest classifier：

[  
s_c(I) = w_c^T \hat{h}(I) + b_c  
]

变量含义：

- (s_c(I))：图像属于类别 (c) 的 score / confidence
    
- (w_c)：类别 (c) 的权重向量
    
- (b_c)：bias
    
- (\hat{h}(I))：归一化后的 BoVW histogram
    

Multi-class：

[  
\hat{y} = \arg\max_c s_c(I)  
]

Multi-label / auto-annotation：

[  
\hat{y}_c =  
\begin{cases}  
1, & s_c(I) > \tau_c \  
0, & s_c(I) \le \tau_c  
\end{cases}  
]

其中 (\tau_c) 是类别 (c) 的 threshold。

---

### 5. Classification accuracy

handout 第 34 页说最简单的 summary measure 是 correct classifications 占 total predictions 的比例。

[  
Accuracy = \frac{# correct}{# predictions}  
]

【易错点】  
严格机器学习里 **Average Precision** 和 **Accuracy** 不是一个东西；但这份 slide 这里讲的是“正确分类比例”这个最简单指标。考试回答时可以写 “simple accuracy / proportion correct”，避免术语混乱。

---

## 3.5 BoVW scene classifier algorithm / pseudocode

### Training stage

```text
Input:
    Training images with labels
    Codebook size K
    Feature extractor, e.g. Dense SIFT / PHOW

1. Split dataset into training / validation / test
2. For each training image:
       extract raw local descriptors
3. Optionally randomly sample descriptors for efficiency
4. Learn codebook using K-means on training descriptors only
5. For each training image:
       quantise descriptors using the learned codebook
       build BoVW histogram
       normalise histogram
6. Train classifier:
       multi-class: linear classifier / SVM / KNN
       multi-label: one-vs-rest classifiers
7. Use validation set to choose parameters:
       K, SIFT step size, classifier C, threshold τ
8. Re-train with chosen parameters if needed
```

handout 第 30–32 页明确强调：dataset 要分 training / test；only training set can be used to train classifier；codebook 用 training features 学；classifier 用 histograms 训练；也可以用 cross-validation 选择参数。

---

### Testing stage

```text
Input:
    Test image
    Trained codebook
    Trained classifier

1. Extract features from test image
2. Quantise features using the training codebook
3. Build occurrence histogram
4. Normalise histogram
5. Apply classifier
6. Output:
       class label
       or multiple annotation labels
       or confidence scores
```

handout 第 33 页明确说 test 时必须使用从 training set 学出来的 codebook。 老师也在 transcript 中强调 test data 不能进入 training，否则会导致 bias / overfitting。

---

# 4. 图像处理直觉解释

你可以把这一讲理解成：

```text
把图像翻译成“视觉单词句子”，再用机器学习读这个句子。
```

类比文本分类：

```text
document:
    words → word histogram → topic classifier

image:
    local patches → visual words → visual word histogram → image classifier
```

BoVW 的关键是：机器不直接理解 “cat” 或 “road accident”，它先看到很多局部 pattern：

```text
fur texture
wheel-like edge pattern
road stripe pattern
sky colour region
grass-like texture
```

然后把这些局部 pattern 量化成 visual words，再统计它们出现多少次。分类器学的是：

```text
如果 visual word 12, 58, 203 很多，可能是 grass / animal
如果 visual word 7, 91, 410 很多，可能是 road / building
```

但这也解释了为什么有 semantic gap：  
BoVW 可以知道 “有 car-like pattern + yellow-line-like pattern”，但不一定知道 “车是倒着的，所以这是事故”。老师的 upside-down car 例子就是这个核心直觉。

---

# 5. 方法优点、缺点、失败情况

## BoVW + Linear Classifier

优点：

- 【高频】结构清晰，考试容易要求写 pipeline。
    
- 可以把 variable number of local features 变成 fixed-length vector。
    
- 配合 linear classifier / SVM 比较 scalable。
    
- 对小的 local variation 比较 robust。
    
- 适合 scene classification / object category classification。
    

缺点：

- 原始 BoVW 丢失 spatial layout。
    
- codebook size 需要调参。
    
- hard quantisation 会丢信息。
    
- 对真正的 scene meaning 理解有限。
    
- multi-label OvR 往往忽略 label correlation。
    

失败情况：

```text
same visual words but different layout
rare/unusual scene
object orientation matters
需要 reasoning 的场景
训练集 bias 明显
test distribution 和 train distribution 不同
```

---

## Dense SIFT / PHOW

优点：

- coverage 更完整。
    
- 比只用 interest points 更适合 classification。
    
- pyramid 可以捕捉 multi-scale context。
    
- spatial pyramid 部分保留空间信息。
    

缺点：

- feature 数量大，计算更慢。
    
- descriptors 多，内存压力大。
    
- 仍然是 hand-crafted features，不如 CNN 自动学习 semantic hierarchy。
    
- 参数多：grid step、patch size、scale levels、codebook size。
    

失败情况：

```text
需要精确 object boundary
强遮挡
非常复杂的 scene relation
类别依赖高级语义而非 texture
```

---

## Deep Learning

老师提到现在 CNN / YOLO 等 deep learning 工具使 annotation、segmentation、human detection / tracking 的性能大幅提升，但仍存在 semantic gap。

优点：

```text
learn features automatically
higher accuracy
can combine detection / segmentation / pose
```

缺点：

```text
需要大量 labelled data
容易受 dataset bias 影响
仍可能在 unusual scenes 上语义误判
解释性弱
```

---

# 6. 考试重点标注

## 【高频】

1. **classification vs annotation vs detection/localisation**
    
2. **semantic gap**
    
3. **BoVW classification pipeline**
    
4. **codebook size trade-off**
    
5. **Dense SIFT / PHOW**
    
6. **train/test split and no leakage**
    
7. **feature space + classifier**
    

---

## 【容易考】

### “Explain the semantic gap”

标准答案结构：

```text
Semantic gap is the difference between low-level visual features that computers process and high-level semantic meaning humans understand.
Example: pixels/SIFT/BoVW may detect car and yellow lines, but fail to infer accident when car is upside down.
```

---

## 【计算题】

可能会考：

1. 给 descriptors 到 visual words 的距离，让你做 vector quantisation。
    
2. 给 visual word assignments，让你构造 BoVW histogram。
    
3. 给 histogram 和 linear classifier weights，让你算 class score。
    
4. 给 multi-label thresholds，让你判断输出哪些 labels。
    

---

## 【设计/解释题】

典型问法：

```text
Design an image classification system using BoVW.
Explain how you train and evaluate it.
How would you adapt it for automatic annotation?
```

要答：

```text
features → codebook → quantisation → histogram → classifier → validation/test
```

并强调：

```text
codebook only learned from training data
test set never used in training
multi-label 用 OvR + threshold
```

---

## 【对比题】

常见对比：

|对比|核心区别|
|---|---|
|classification vs detection|是否定位|
|multi-class vs multi-label|一个标签 vs 多个标签|
|image search BoVW vs classification BoVW|search 需要更大、更distinctive codebook；classification 用较小 codebook|
|interest-point SIFT vs Dense SIFT|repeatable keypoints vs full image coverage|
|BoVW vs PHOW|unordered histogram vs 加入 multi-scale + spatial pyramid|
|BoVW vs CNN|hand-crafted + codebook vs learned representation|

---

## 【易错点】

1. **BoVW 不是直接识别 object，而是统计 visual word occurrence。**
    
2. **codebook 必须从 training set 学，不能用 test set。**
    
3. **auto-annotation 是 multi-label，不是 object detection。**
    
4. **classification 的小 codebook 不是“更差”，而是为了泛化和效率。**
    
5. **Dense SIFT 更 dense，不代表更 semantic；它只是 coverage 更好。**
    

---

## 【老师强调】

老师反复强调了这些：

1. 这讲是前面 feature / matching / BoVW 技术的另一个应用，不是完全新技术。
    
2. Annotation 是给图像打标签；localisation / detection 是指出对象在哪里。
    
3. 语义理解仍然困难，semantic gap 仍然存在，upside-down car 是关键例子。
    
4. Classification / scene recognition 的 visual vocabulary 要比 image search 小，因为它需要容纳类内变化。
    
5. Training/test split 很重要，test data 不能进入 training，否则 bias / overfitting。
    

老师快速带过、考试优先级较低的部分：

```text
具体老系统 2008 auto-annotation 的历史细节
某些 demo 的具体软件名称
YOLO 等现代 deep learning 细节
Final coursework 行政信息
```

但 **deep learning 进步 + semantic gap 仍在** 这个观点要记住。

---

# 7. 考试风格题

## 7.1 概念解释题

### Q1. Explain the semantic gap in computer vision.

答案思路：

```text
Semantic gap = low-level image representation and high-level human meaning 之间的差距。
Computer sees pixels, colours, edges, descriptors, visual words.
Human understands events, intentions, abnormality, context.
Example: car + double yellow lines may be recognised, but upside-down car implies accident.
```

---

### Q2. What is automatic annotation, and how is it related to multi-label classification?

答案思路：

```text
Automatic annotation means assigning semantic keywords/tags to an image.
It is usually multi-label because one image can contain multiple concepts.
Example: sky + road + person + tree.
Different from multi-class classification where only one class is selected.
```

---

### Q3. What is a visual word in BoVW?

答案思路：

```text
A visual word is a representative descriptor obtained by vector quantising local image descriptors.
Usually learned by K-means.
Each local feature is assigned to the nearest codeword.
The image is represented by a histogram of visual word occurrences.
```

---

## 7.2 对比题

### Q4. Compare image classification and object detection.

答案思路：

```text
Classification predicts what is in the image.
Detection predicts what and where.
Classification output: label/confidence.
Detection output: label + bounding box or region.
Detection is harder because it includes localisation.
```

---

### Q5. Compare BoVW for image search and BoVW for image classification.

答案思路：

```text
Image search needs precise matching, so codebook tends to be larger and more distinctive.
Classification needs general category recognition, so codebook can be smaller and less distinctive.
Smaller codebook improves efficiency and allows intra-class variation.
```

---

### Q6. Compare interest-point SIFT and Dense SIFT for classification.

答案思路：

```text
Interest-point SIFT extracts features at stable points, good for matching.
Dense SIFT extracts features on a grid, gives better coverage.
For classification, coverage is often more important than exact repeatability.
Dense SIFT is more expensive but can improve classification performance.
```

---

## 7.3 计算/推导题

### Q7. Given visual word assignments `[1, 2, 2, 4, 1, 3, 2]` and codebook size (K=4), compute the BoVW histogram.

答案：

```text
word 1 appears 2 times
word 2 appears 3 times
word 3 appears 1 time
word 4 appears 1 time
```

[  
h = [2, 3, 1, 1]  
]

L1-normalised：

[  
\hat{h} = [2/7, 3/7, 1/7, 1/7]  
]

---

### Q8. A two-class linear classifier has scores:

[  
s_{cat}=w_{cat}^T h+b_{cat}=1.2  
]

[  
s_{dog}=w_{dog}^T h+b_{dog}=0.7  
]

What is the predicted class?

答案：

```text
Choose the class with maximum score.
1.2 > 0.7, so predicted class = cat.
```

如果是 multi-label 且 threshold (\tau=0.5)，则 cat 和 dog 都会被标注；如果 threshold (\tau=1.0)，只有 cat 被标注。

---

## 7.4 设计应用题

### Q9. Design a BoVW image classifier for scene classification.

答案思路：

```text
1. Split labelled dataset into train/validation/test.
2. Extract Dense SIFT or PHOW features from training images.
3. Learn codebook using K-means on training descriptors only.
4. Quantise descriptors into visual words.
5. Build normalised histogram for each image.
6. Train linear SVM / OvR classifiers.
7. Use validation set to tune K, grid step, classifier parameter C.
8. Evaluate on test set.
```

必须强调：

```text
test data must not be used for codebook learning or classifier training.
```

---

### Q10. You need to build an automatic annotation system for images containing multiple objects. How would you modify a multi-class classifier?

答案思路：

```text
Use multi-label classification.
Train one-vs-rest classifier for each label.
For each class c, compute score s_c.
Apply threshold τ_c.
Output all labels whose scores exceed threshold.
```

可以补充：

```text
Use Dense SIFT / PHOW + BoVW histogram as image feature.
Need tune thresholds for precision/recall trade-off.
```

---

# 8. 考前速记版

## 8.1 Must-know points

1. **Image classification = feature extraction + machine learning + label prediction.**
    
2. **Automatic annotation = multi-label image classification.**
    
3. **Object detection/localisation = classification + where.**
    
4. **Semantic gap = low-level visual features vs high-level human meaning.**
    
5. **BoVW pipeline = descriptors → codebook → quantisation → histogram → classifier.**
    
6. **Classification codebook smaller than image search codebook.**
    
7. **Dense SIFT gives better coverage than interest-point SIFT.**
    
8. **PHOW = Pyramid Dense SIFT + visual word histogram + spatial pyramid.**
    
9. **Train/test leakage is a serious error.**
    
10. **Linear classifiers / SVMs are commonly used with BoVW histograms.**
    

---

## 8.2 High-risk mistakes

1. 把 **auto-annotation** 说成 object detection。
    
2. 忘记 BoVW 丢失 spatial layout。
    
3. 用 test set 学 codebook。
    
4. 认为 codebook 越大 classification 一定越好。
    
5. 忽略 semantic gap，只写“分类器输出标签”。
    

---

## 8.3 Likely exam questions

1. **Explain the semantic gap with an example.**
    
2. **Describe the full BoVW classification pipeline.**
    
3. **Compare BoVW for image search and image classification.**
    
4. **Why might Dense SIFT / PHOW improve classification performance?**
    
5. **How would you adapt a classifier for automatic annotation?**
    

---

## 8.4 最应该背的流程

```text
Training:
images + labels
→ extract Dense SIFT / PHOW features
→ learn codebook using K-means on training features
→ quantise descriptors
→ build BoVW histograms
→ train classifier, e.g. linear SVM / OvR
→ validate parameters

Testing:
test image
→ extract same features
→ quantise using training codebook
→ build histogram
→ classifier predicts label(s)
```

最应该理解的一句话：

> **BoVW classification does not directly understand the image; it turns local visual patterns into a histogram, then learns which histograms correspond to which semantic labels. The hard part is bridging the semantic gap.**