
# 2. 这节课在 Computer Vision pipeline 中的位置

这节课属于 **高层视觉 / recognition and retrieval** 部分。

前面几讲已经解决了：

- 如何找稳定点：**Local interest points**
    
- 如何描述局部图像块：**SIFT / local descriptors**
    
- 如何做 pairwise matching：**feature matching**
    
- 如何用几何约束去掉 mismatch：**RANSAC / consistent matching**
    

这节课进一步问：

> 如果不是只匹配两张图，而是在一个巨大 image database 里找最相似的图片，怎么办？

所以它解决的是 **content-based image retrieval** 问题。

输入是：

- 一个 image database / corpus；
    
- 每张图像中提取出的 local descriptors，例如 SIFT；
    
- 一个 query image。
    

输出是：

- 每张数据库图片和 query image 的 similarity score；
    
- 按相似度排序的 retrieved images。
    

它和下一讲 **Image classification and auto-annotation** 的关系很重要：这节课用 BoVW 做 **search / retrieval**，下一讲会把类似的 visual words 用到 **recognition / classification**。老师在 transcript 里特别强调：image search 通常需要更大的 codebook，因为要区分非常具体的对象；image recognition 可以用较小 codebook，因为只需要把不同外观的车都归为 “car”。

---

# 3. 系统复习笔记

## 3.1 从 Text Retrieval 到 Image Retrieval

老师先从 text search 讲起，因为 text 比 image 简单。Text retrieval 的核心流程是：

```text
document
→ tokenisation
→ stop-word removal
→ stemming / lemmatisation
→ Bag of Words
→ vector-space model
→ similarity ranking
```

handout 中给的例子是句子：

```text
the quick brown fox jumped over the lazy dog
```

经过处理后得到：

```text
brown, dog, fox, jump, lazi, over, quick
```

每个词出现一次，所以可以形成一个 count histogram。

### 核心直觉

Bag of Words 不关心句子顺序，只关心：

> 这个文档里有哪些词？每个词出现了多少次？

这就是为什么它叫 **bag**：像一个袋子，把词丢进去，不记录顺序，但记录数量。handout 明确说 bag 是一种 unordered data structure，类似 set，但允许同一个元素多次出现。

---

## 3.2 Vector-space Model

每个 document 变成一个 vector，每个 query 也变成一个 vector。假设是：

> 在 feature space 中距离近的向量，对应语义上相似的 document / image。

handout 中写得很明确：document 和 query 都被表示成 vector，然后用 similarity measure 对 documents 排名。

如果 vocabulary 有 $$K$$ 个词，那么一个文档向量是：

$$  
\mathbf{d} = [d_1, d_2, \dots, d_K]  
$$

其中：

- $$K$$：vocabulary size；
    
- $$d_i$$：第 $$i$$ 个 word 在该 document 中的权重，可以是 0/1、count、TF-IDF；
    
- $$\mathbf{d}$$：document vector。
    

### 易错点

这个 vector 通常非常高维、非常稀疏。比如 vocabulary 有 50,000 个词，但一个 query 可能只有 7 个词，那么绝大多数维度都是 0。老师在 transcript 中反复强调 sparse vector 会导致大量 “0 × 0” 的无效计算。

---

## 3.3 Cosine Similarity

### 公式

$$  
\cos(\mathbf{p}, \mathbf{q})

\frac{\mathbf{p} \cdot \mathbf{q}}  
{|\mathbf{p}||\mathbf{q}|}

\frac{\sum_{i=1}^{K} p_i q_i}  
{\sqrt{\sum_{i=1}^{K}p_i^2}\sqrt{\sum_{i=1}^{K}q_i^2}}  
$$

变量含义：

- $$\mathbf{p}$$：database document / image 的 feature vector；
    
- $$\mathbf{q}$$：query vector；
    
- $$p_i$$：database vector 第 $$i$$ 维的权重；
    
- $$q_i$$：query vector 第 $$i$$ 维的权重；
    
- $$K$$：vocabulary size；
    
- $$\mathbf{p}\cdot\mathbf{q}$$：两个向量的 dot product；
    
- $$|\mathbf{p}|$$、$$|\mathbf{q}|$$：两个向量的 magnitude / norm。
    

### 为什么不用 Euclidean Distance？

Euclidean Distance 容易被 **向量长度 / 文档长度 / feature 数量** 影响。老师举的直觉是：一个长文档可能词很多，magnitude 很大；另一个短文档可能更相关，但 Euclidean distance 可能判断错误。Cosine similarity 更关注 **方向**，弱化 magnitude 的影响。

### 考试判断

【容易考】问你为什么 image retrieval / text retrieval 中常用 cosine similarity，而不是 Euclidean distance。

答法：

> 因为 BoW / BoVW vector 的 magnitude 会受到 document length 或 feature count 的影响。Cosine similarity 比较向量夹角，更关注 feature distribution pattern，而不是总数量。

---

## 3.4 Inverted Index

如果直接比较 query vector 和每个 document vector，需要对所有维度做乘法。对于 50,000 维 sparse vector，这很浪费。

所以使用 **inverted index**：

```text
word / visual word → [(document_id, count), ...]
```

handout 中给的例子：

```text
Movie → [doc2:10]
Star  → [doc1:13; doc2:4]
```

一个 posting 是：

```text
(document ID, term frequency)
```

也就是某个 word 在某个 document 中出现了多少次。

### 查询 “Movie Star” 的 accumulation

对于 query：

```text
Movie Star
```

只需要查 Movie 和 Star 的 postings，不需要遍历所有 vocabulary。

accumulation：

```text
doc2: 10 × 1 + 4 × 1
doc1: 13 × 1
```

再除以预计算好的 norm，得到 cosine similarity。handout 示例中 doc2 的 score 是 0.997，doc1 是 0.652，所以 doc2 排在前面。

### 伪代码

```text
Input:
    query vector q
    inverted index index[word] = list of (doc_id, weight)
    precomputed document norms norm_d[doc_id]

score = empty dictionary

for each non-zero word i in query:
    for each (doc_id, weight_i_doc) in index[i]:
        score[doc_id] += q_i * weight_i_doc

for each doc_id in score:
    score[doc_id] = score[doc_id] / (norm_q * norm_d[doc_id])

return documents sorted by score descending
```

### 考试重点

【计算题】给一个 inverted index 和 query，让你算 accumulation table。  
【易错点】inverted index 不是改变 similarity 本身，而是加速 sparse vector 的计算。

---

## 3.5 Weighting Schemes：Binary / Raw Frequency / TF-IDF

handout 中列了三类 weighting schemes：binary weights、raw frequency、TF-IDF。

### Binary Weight

 $$  
w_{i,d}

\begin{cases}  
1, & \text{if term } i \text{ appears in document } d\  
0, & \text{otherwise}  
\end{cases}  
$$

只看出现没出现，不看出现多少次。

### Raw Frequency

$$  
w_{i,d}=tf_{i,d}  
$$

其中：

- $$tf_{i,d}$$：term $$i$$ 在 document $$d$$ 中出现的次数。
    

### TF-IDF

$$  
w_{i,d}=tf_{i,d}\times idf_i  
$$

常见形式：

$$  
idf_i=\log\frac{N}{df_i}  
$$

变量含义：

- $$N$$：database 中 document 总数；
    
- $$df_i$$：包含 term $$i$$ 的 document 数量；
    
- $$tf_{i,d}$$：term $$i$$ 在 document $$d$$ 中出现的次数；
    
- $$idf_i$$：inverse document frequency；
    
- $$w_{i,d}$$：最终权重。
    

### 直觉

常见词，例如 “the, going, come”，区分能力弱，权重要低。  
某个词如果在一个文档中频繁出现，但在其他文档中很少出现，它对检索这个文档很重要，权重要高。老师在 transcript 里明确说 TF-IDF 会给 rare words 高值、common words 低值。

---

# 4. Vector Quantisation 与 Visual Words

## 4.1 为什么需要 Vector Quantisation？

图像没有天然的 “word”。SIFT descriptor 是一个连续的 128-dimensional vector，不可能直接像单词一样做 histogram。

所以需要：

> 把连续 descriptor 聚类成有限个类别，每个类别当作一个 visual word。

这就是 **Vector Quantisation**。

handout 定义：Vector quantisation 是一种 lossy data compression technique，可以用 K-Means 学到固定数量的 representative vectors；这些 representative vectors 组成 **codebook**。

---

## 4.2 K-Means Codebook

给定很多 local descriptors：

$$  
\mathbf{x}_1, \mathbf{x}_2, \dots, \mathbf{x}_N  
$$

用 K-Means 学 $$K$$ 个 cluster centres：

$$  
\boldsymbol{\mu}_1, \boldsymbol{\mu}_2, \dots, \boldsymbol{\mu}_K  
$$

K-Means 目标函数：

$$  
\min_{\boldsymbol{\mu}_1,\dots,\boldsymbol{\mu}_K}  
\sum_{n=1}^{N}  
\min_{k}  
|\mathbf{x}_n-\boldsymbol{\mu}_k|^2  
$$

变量含义：

- $$N$$：所有 training descriptors 的数量；
    
- $$K$$：codebook size / vocabulary size；
    
- $$\mathbf{x}_n$$：第 $$n$$ 个 local descriptor；
    
- $$\boldsymbol{\mu}_k$$：第 $$k$$ 个 cluster centre，也就是第 $$k$$ 个 visual word；
    
- $$|\mathbf{x}_n-\boldsymbol{\mu}_k|^2$$：descriptor 和 cluster centre 的距离。
    

---

## 4.3 Descriptor Assignment

对每个 descriptor，找到最近的 visual word：

$$  
c(\mathbf{x})

\arg\min_{k}  
|\mathbf{x}-\boldsymbol{\mu}_k|^2  
$$

变量含义：

- $$\mathbf{x}$$：某个 local descriptor；
    
- $$\boldsymbol{\mu}_k$$：第 $$k$$ 个 visual word centre；
    
- $$c(\mathbf{x})$$：该 descriptor 被分配到的 visual word index。
    

老师强调 vector quantisation 的一个重要性质是：你可以通过设置 $$K$$ 来控制 vocabulary size，也就是最终 feature vector 的长度。

---

# 5. Bag of Visual Words

## 5.1 Visual Word 是什么？

一个 **visual word** 可以理解为：

> 一类相似 local image patches 的代表。

例如很多图像中都有类似的 corner、edge junction、blob texture，这些 SIFT descriptors 被聚到同一个 cluster 后，就对应同一个 visual word。

handout 明确说：可以对 SIFT descriptors 或其他 local features 做 vector quantisation，每个 descriptor 被替换成一个 representative vector，也就是 visual word；codebook 就是视觉版 vocabulary。

---

## 5.2 BoVW Histogram

一张图像有很多 descriptors：

$$  
\mathbf{x}_1,\mathbf{x}_2,\dots,\mathbf{x}_{m_I}  
$$

每个 descriptor 被分配到一个 visual word：

$$  
c(\mathbf{x}_j)\in{1,\dots,K}  
$$

于是图像 $$I$$ 的 BoVW histogram 是：

$$  
h_k(I)

\sum_{j=1}^{m_I}  
\mathbf{1}[c(\mathbf{x}_j)=k]  
$$

变量含义：

- $$I$$：一张图像；
    
- $$m_I$$：图像 $$I$$ 中 local descriptors 的数量；
    
- $$K$$：visual vocabulary size；
    
- $$h_k(I)$$：第 $$k$$ 个 visual word 在图像 $$I$$ 中出现的次数；
    
- $$\mathbf{1}[\cdot]$$：indicator function，条件成立为 1，否则为 0。
    

最终：

$$  
\mathbf{h}(I)

[h_1(I),h_2(I),\dots,h_K(I)]  
$$

这就是图像的 fixed-length vector。

handout 强调：BoVW 的好处是把 variable number of local descriptors 聚合成 fixed length vector，既能用于 machine learning，也能把 text retrieval 技术直接用于 image retrieval。

---

## 5.3 BoVW 的核心牺牲：忽略空间位置

BoVW 会忽略 local features 来自图像哪里，包括 position 和 scale。handout 明确写到：quantise local features 后放入 bag，本质上忽略它们在图像中的位置。

老师在 transcript 里解释了为什么有时这是合理的：比如 stars 不一定总在图像上方，因为拍摄角度和 pose 会变，所以很多场景下 location 不稳定。

### 直觉

BoVW 看一张图像像看一个购物袋：

```text
这张图里有 10 个 edge-like patches，
5 个 corner-like patches，
3 个 blob-like patches，
...
```

但它不关心：

```text
corner 在左上角还是右下角
edge 在天空还是地面
```

所以它更 invariant，但更容易丢失 geometry。

---

# 6. BoVW Image Retrieval Pipeline

## 6.1 Indexing / Training Stage

```text
Input:
    image corpus / database

Step 1: Extract local features
    For each image:
        detect interest points
        compute descriptors, e.g. SIFT

Step 2: Learn codebook
    Collect many descriptors from training images
    Optionally sample descriptors if too many
    Run K-Means with K clusters
    Cluster centres become visual vocabulary / codebook

Step 3: Quantise descriptors
    For each descriptor:
        assign it to nearest visual word

Step 4: Build BoVW histograms
    For each image:
        count visual word occurrences
        optionally apply TF-IDF weighting
        normalise vector

Step 5: Build inverted index
    For each visual word:
        store list of images where it appears and its weight
```

handout 的 overall process 也正是：collect corpus、extract local features、learn large codebook、vector quantise、build BoVW representations、optional inverted index。

---

## 6.2 Query / Search Stage

```text
Input:
    query image

Step 1: Extract local features from query image
Step 2: Compute descriptors
Step 3: Assign descriptors to nearest visual words
Step 4: Build query BoVW histogram
Step 5: Compare query histogram with database histograms
        usually using cosine similarity
Step 6: Rank database images by similarity
Step 7: Return top-N images
```

handout 中的 image retrieval 框架是：

```text
Image → descriptor extraction → Descriptors → similarity matcher → search result
```

BoVW retrieval 之后，可以把 text retrieval 的 vector-space model、cosine similarity、weighting schemes、inverted index 直接用于图像。

---

# 7. Codebook Size：这节课最重要的设计参数

## 7.1 太小的问题

如果 $$K$$ 太小：

```text
很多不同 descriptors 都被映射到同一个 visual word
```

结果：

- vector 不够 distinctive；
    
- 很多图像 histogram 看起来差不多；
    
- 搜索结果容易混。
    

handout 说：too small, and all vectors look the same → not distinctive。

---

## 7.2 太大的问题

如果 $$K$$ 太大：

```text
同一个视觉模式在不同图像中可能被分到不同 visual words
```

结果：

- too distinctive；
    
- 相似图像之间反而没有共同 visual words；
    
- matching 变差；
    
- K-Means 训练非常重；
    
- inverted index 可能极度稀疏；
    
- memory 和 computation 变大。
    

handout 说：too big, and the same visual words might never appear across images → too distinctive。

---

## 7.3 Image Search vs Image Recognition

这是 transcript 里老师很强调的点：

- **Image search / retrieval**：需要找最接近的具体图片或具体对象，比如 Porsche 911，所以 codebook 要更大、更 distinctive。
    
- **Image recognition / classification**：只需要判断是不是 car，不需要区分红车、黄车、SUV、卡车等细节，所以 codebook 可以更小、更 tolerant。
    

考试中可以这样答：

> Retrieval requires fine-grained distinctiveness; classification requires category-level generalisation. Therefore, retrieval often prefers a larger visual vocabulary, while recognition may use a smaller one to allow intra-class variation.

---

# 8. 优点、缺点、失败情况

## 优点

1. **Scalable**：可以通过 inverted index 做大规模检索。
    
2. **Fixed-length representation**：把每张图不同数量的 descriptors 变成固定长度 histogram。
    
3. **复用 text retrieval 技术**：vector-space model、cosine similarity、TF-IDF、inverted index 都可直接迁移。
    
4. **对位置变化有一定鲁棒性**：因为忽略 spatial position。
    
5. **适合 content-based image retrieval**：不需要人工标签，只靠图像内容检索。
    

## 缺点

1. **丢失 spatial layout**：两个 visual words 的相对位置完全没保留。
    
2. **codebook size 难选**：太小不 distinctive，太大过度 distinctive。
    
3. **K-Means 成本高**：大规模 SIFT descriptors 可能是千万级、上亿级。
    
4. **quantisation error**：连续 descriptor 被硬分配到一个 cluster，会损失细节。
    
5. **对 repeated textures / common patches 容易误匹配**：类似 stop words，需要 TF-IDF 或 weighting 降低影响。
    
6. **对语义理解有限**：BoVW 匹配的是局部视觉模式，不是真正理解图像语义。
    

## 失败情况

- 大量重复纹理，例如砖墙、草地、窗户；
    
- query image 和 database image 视角差太大；
    
- 局部特征太少，例如纯色物体；
    
- codebook 太小导致所有图像 histogram 相似；
    
- codebook 太大导致相似图像没有 shared visual words；
    
- BoVW 忽略空间结构，导致 “相同局部元素但布局完全不同” 的图像被错误匹配。
    

---

# 9. 考试重点标注

## 【高频】

1. Bag of Words → Bag of Visual Words 的类比。
    
2. Vector-space model：document/image/query 都变成 vector。
    
3. Cosine similarity 的公式和直觉。
    
4. Inverted index 如何加速 sparse vector matching。
    
5. Vector quantisation / K-Means codebook。
    
6. BoVW histogram 如何构造。
    
7. Codebook size 的 trade-off。
    

## 【容易考】

- 为什么 BoVW 忽略空间位置？
    
- 为什么 cosine similarity 比 Euclidean distance 更适合 BoW / BoVW？
    
- 为什么 inverted index 能加速？
    
- 为什么 image search 需要比 image recognition 更大的 vocabulary？
    
- 为什么 TF-IDF 能提高检索效果？
    

## 【计算题】

1. 给 postings list 和 query，算 accumulation table。
    
2. 给两个 BoVW vectors，算 cosine similarity。
    
3. 给 visual word assignments，构造 histogram。
    
4. 给 $$tf$$、$$df$$、$$N$$，算 TF-IDF。
    
5. 给 cluster centres 和 descriptor，判断它属于哪个 visual word。
    

## 【设计/解释题】

1. 设计一个 image retrieval system。
    
2. 解释如何从 SIFT descriptors 构建 visual vocabulary。
    
3. 选择 codebook size，并说明 trade-off。
    
4. 数据集很大时如何提高检索效率？
    
5. 为什么可以把 text retrieval 技术用于 image retrieval？
    

## 【对比题】

1. Bag of Words vs Bag of Visual Words。
    
2. Euclidean distance vs Cosine similarity。
    
3. Binary weighting vs Raw frequency vs TF-IDF。
    
4. Small codebook vs Large codebook。
    
5. Image retrieval vs Image classification。
    

## 【易错点】

1. BoVW 的 “word” 不是真实语义词，而是 cluster centre。
    
2. Codebook 是从 descriptors 学出来的，不是人工定义的。
    
3. Inverted index 不是 similarity measure，而是加速结构。
    
4. TF-IDF 不是只看出现次数，还看这个 word 在全库中是否稀有。
    
5. BoVW 忽略空间信息，所以不能可靠表达 object geometry。
    

## 【老师强调】

老师反复强调：

- Google Lens 这类系统为什么能 “within one second” 搜索，是因为用 image features 和高效 indexing。
    
- text retrieval 是理解 image retrieval 的入口。
    
- sparse high-dimensional vector 直接算很浪费。
    
- cosine similarity 比 Euclidean distance 更适合这种 histogram vector。
    
- vector quantisation 是非常重要的概念，它让我们能控制 vocabulary size / vector length。
    
- codebook size 要根据应用决定，search 更需要 distinctive，recognition 更需要 generalisation。
    
- BoVW 是较老的方法，现代 Google 当然用更 advanced algorithms，但这些思想是基础。
    

---

# 10. 考试风格题与答案思路

## A. 概念解释题

### Q1. Explain the idea of Bag of Visual Words.

答案思路：

BoVW 把一张图像表示成 visual word histogram。先提取 local descriptors，例如 SIFT；再用 K-Means 学 codebook；每个 descriptor 被分配到最近的 visual word；最后统计每个 visual word 出现次数。它忽略 spatial position，但得到 fixed-length vector，便于 retrieval 和 classification。

---

### Q2. What is a visual vocabulary / codebook?

答案思路：

Codebook 是一组 representative descriptor vectors，通常由 K-Means 从大量 local descriptors 中学习得到。每个 cluster centre 是一个 visual word。Vocabulary size $$K$$ 决定 BoVW vector 的维度。

---

### Q3. Why is BoVW similar to Bag of Words in text retrieval?

答案思路：

Text BoW 把 document 切成 words，统计 word occurrence；BoVW 把 image 切成 local descriptors，再量化成 visual words，统计 visual word occurrence。两者都忽略顺序/位置，都形成 histogram vector，都可以用 vector-space model、cosine similarity、TF-IDF 和 inverted index。

---

## B. 对比题

### Q4. Compare Euclidean distance and cosine similarity for BoVW retrieval.

答案思路：

Euclidean distance 受 vector magnitude 影响，可能把 feature 数量多的图片判断得更远或更近。Cosine similarity 比较方向，关注 visual word distribution pattern，更适合 sparse histogram vectors。缺点是 cosine similarity 仍然不考虑 spatial layout。

---

### Q5. Compare small and large codebooks.

答案思路：

Small codebook：更 compact、更快、更有 generalisation，但不够 distinctive，很多不同 patches 会被合并。  
Large codebook：更 distinctive，适合 fine-grained retrieval，但训练 K-Means 成本高、memory 大、可能 too sparse，同一视觉模式可能难以匹配。

---

### Q6. Compare image retrieval and image classification when using BoVW.

答案思路：

Retrieval 目标是找最相似的具体图像或实例，因此需要更大的 codebook 和更强 distinctiveness。Classification 目标是归到类别，例如 car，所以需要容忍 intra-class variation，codebook 可以更小，visual words 可以不那么细。

---

## C. 计算/推导题

### Q7. Given two BoVW vectors, compute cosine similarity.

题目：

$$  
\mathbf{p}=[2,0,1,3],\quad \mathbf{q}=[1,0,1,0]  
$$

答案：

$$  
\mathbf{p}\cdot\mathbf{q}=2\times1+0\times0+1\times1+3\times0=3  
$$

$$  
|\mathbf{p}|=\sqrt{2^2+0^2+1^2+3^2}=\sqrt{14}  
$$

$$  
|\mathbf{q}|=\sqrt{1^2+0^2+1^2+0^2}=\sqrt{2}  
$$

$$  
\cos(\mathbf{p},\mathbf{q})=\frac{3}{\sqrt{14}\sqrt{2}}=\frac{3}{\sqrt{28}}\approx0.567  
$$

---

### Q8. Given assignments, build a BoVW histogram.

题目：

Codebook size $$K=5$$。某图像的 8 个 descriptors 被分配到 visual words：

```text
1, 3, 3, 5, 2, 1, 3, 5
```

答案：

$$  
\mathbf{h}=[2,1,3,0,2]  
$$

解释：

- visual word 1 出现 2 次；
    
- visual word 2 出现 1 次；
    
- visual word 3 出现 3 次；
    
- visual word 4 出现 0 次；
    
- visual word 5 出现 2 次。
    

---

## D. 设计应用题

### Q9. Design a system for searching landmarks from a query photo.

答案思路：

1. Collect landmark image database.
    
2. Extract local features, e.g. DoG-SIFT.
    
3. Sample descriptors if too many.
    
4. Learn large K-Means codebook.
    
5. Quantise descriptors into visual words.
    
6. Build BoVW histograms for each image.
    
7. Apply TF-IDF weighting.
    
8. Build inverted index.
    
9. For query image, repeat feature extraction and quantisation.
    
10. Use cosine similarity to rank database images.
    
11. Return top-ranked landmark images.
    

---

### Q10. Your BoVW retrieval system returns many irrelevant images. What could be wrong?

答案思路：

可能原因：

- codebook too small，visual words 不够 distinctive；
    
- common visual words 权重太高，需要 TF-IDF；
    
- local descriptors 不够 robust；
    
- image contains repeated textures；
    
- BoVW 忽略 spatial layout，需要加入 geometric verification；
    
- query 和 database domain 差异太大；
    
- K-Means codebook 训练数据不代表真实数据库。
    

---

# 11. 考前速记版

## Must-know points

1. **BoVW = local descriptors → visual words → histogram vector。**
    
2. **Visual vocabulary / codebook 通常由 K-Means 学出来。**
    
3. **每个 cluster centre 是一个 visual word。**
    
4. **BoVW 忽略 spatial position，所以更 invariant，但丢失 geometry。**
    
5. **Cosine similarity 更适合 sparse histogram vectors。**
    
6. **Inverted index 用 word → document postings 加速检索。**
    
7. **TF-IDF 提高 rare but discriminative words 的权重。**
    
8. **Codebook size 是关键参数：too small 不 distinctive，too large too distinctive。**
    
9. **Image search 通常需要 larger codebook；classification 通常可以 smaller codebook。**
    
10. **BoVW 的本质是把 text retrieval 方法迁移到 image retrieval。**
    

## High-risk mistakes

1. 把 visual word 理解成真实语义类别，例如 “car” 或 “dog”。错，它只是 descriptor cluster。
    
2. 以为 inverted index 改变了匹配结果。错，它主要是加速 sparse matching。
    
3. 忘记 BoVW 丢失空间布局。
    
4. 认为 codebook 越大越好。错，太大会过度 distinctive，训练和匹配都困难。
    
5. 用 Euclidean distance 时不考虑 vector magnitude 的影响。
    

## Likely exam questions

1. Explain how Bag of Visual Words is constructed from local descriptors.
    
2. Why is cosine similarity preferred over Euclidean distance in vector-space retrieval?
    
3. Describe how an inverted index speeds up BoVW image retrieval.
    
4. Discuss the effect of codebook size on image search performance.
    
5. Compare the use of BoVW for image retrieval and image classification.
    

## 最应该背/理解的公式与流程

### Cosine Similarity

$$  
\cos(\mathbf{p}, \mathbf{q})

\frac{\sum_i p_i q_i}  
{\sqrt{\sum_i p_i^2}\sqrt{\sum_i q_i^2}}  
$$

### Vector Quantisation Assignment

$$  
c(\mathbf{x})

\arg\min_k  
|\mathbf{x}-\boldsymbol{\mu}_k|^2  
$$

### BoVW Histogram

$$  
h_k(I)

\sum_{j=1}^{m_I}  
\mathbf{1}[c(\mathbf{x}_j)=k]  
$$

### BoVW Retrieval Pipeline

```text
database images
→ extract local features
→ learn K-Means codebook
→ assign descriptors to visual words
→ build BoVW histograms
→ optional TF-IDF
→ optional inverted index
→ query image follows same pipeline
→ cosine similarity
→ ranked retrieval results
```