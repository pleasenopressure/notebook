# 📘 COMP6223 Computer Vision

## Lecture 4：Point Operators（点运算）

**完整课程梳理与学习指南（助教版）**

---

## 🔎 精炼摘要（先看这个，考试快读）

- **点运算（Point Operators）**：  
    每个像素**独立变换**，不依赖邻域
    
- 核心围绕 **直方图（Histogram）操控**
    
- 两大类任务：  
    👉 **图像显示增强** & **阈值分割**
    
- 重点方法：  
    **亮度映射 / 强度归一化 / 直方图均衡 / 阈值化（Otsu）**
---

## 一、课程定位与核心思想

📍 _Slide 1–2_

老师一开始就明确：

> **Point operators are largely about image display**

📌 含义：

- 不是做“理解”
    
- 而是做 **“看得更清楚”**
    
- 每个像素 **单独处理**
    
- 不涉及卷积、不看邻居
    

---

## 二、图像直方图（Histogram）——点运算的基础

### 2.1 什么是图像直方图？

> **直方图 = 每个灰度值出现的像素数量**

![Image](https://www.researchgate.net/publication/337829490/figure/fig4/AS%3A834018929029120%401575857016218/Eye-histogram-charts-of-different-quality-levels.ppm)

![Image](https://www.researchgate.net/publication/228744430/figure/fig2/AS%3A302009828823041%401449016162133/Histogram-Analysis-of-grayscale-image.png)

- 横轴：亮度值（0–255）
    
- 纵轴：像素个数
    
- 反映：
    
    - 对比度（spread）
        
    - 是否偏亮 / 偏暗
    
---

### 2.2 普通直方图 vs 累积直方图

📍 _Slide 4_

![Image](https://upload.wikimedia.org/wikipedia/commons/5/53/Cumulative_vs_normal_histogram.svg)

![Image](https://www2.tulane.edu/~salem/Images/Cumulative-Histogram.gif)


📌 **后面做 Histogram Equalisation 的核心工具**

---

## 三、基本强度映射（Intensity Mappings）

📍 _Slide 5–6_

### 3.1 线性点运算公式
$N = k\cdot O - l$  

- (O)：原像素值
    
- (N)：新像素值
    
- (k)：对比度（拉伸/压缩）
    
- (l)：亮度偏移
    
---

### 3.2 四种经典映射（必记）

![Image](https://www.researchgate.net/publication/3193623/figure/fig3/AS%3A668939117936655%401536498922735/a-Diagram-showing-the-recovery-of-the-intensity-mapping-function-from-registered-images.png)

![Image](https://dl-file.cyberlink.com/web/content/b1105/Thumbnail.jpg)

![Image](https://www.warehouse-lighting.com/cdn/shop/articles/Scale-of-Light-Brightness-Infographic_1.png?v=1581701253)

|操作|公式|作用|
|---|---|---|
|copy|(N=O)|不变|
|inversion|(N=255-O)|亮暗反转|
|addition|(N=O+l)|整体变亮|
|scaling|(N=kO-l)|拉伸对比度|
![[Pasted image 20260524121737.png]]
📌 **教师强调**：

> 这些操作**不改变像素相对位置**

---

## 四、非线性点运算：Log / Exp

### 4.1 对数压缩（Logarithmic Compression）

$N=\log(O)$  

![Image](https://homepages.inf.ed.ac.uk/rbf/HIPR2/figs/logaritm.gif)

![Image](https://www.researchgate.net/publication/270275573/figure/fig2/AS%3A295042678837257%401447355065064/Diffraction-Pattern-Enhanced-Through-Log-Transformation.png)

- 压缩高亮区域
    
- 放大暗部细节
    
- 常用于：
    
    - 医学影像
    - 动态范围过大的图像
        
---

### 4.2 指数扩展（Exponential Expansion）

$N=\exp(O)$  

- 强调亮区
    
- 暗区被压制
📌 **成对出现，考试容易问对比**
![[Pasted image 20260524122643.png]]

---

## 五、强度归一化（Intensity Normalisation）

📍 _Slide 8–9_

### 5.1 目的（老师原话）

> **Aim is to use all available grey levels for display**

---
### 5.2 核心公式（非常重要）
$N_{x,y}=\frac{N_{max}-N_{min}}{O_{max}-O_{min}}\cdot (O_{x,y}-O_{min}) + N_{min}$  
![[Pasted image 20260524122718.png]]
- 本质：  
    👉 **线性拉伸到 [0,255]**
    

![Image](https://intensity-normalization.readthedocs.io/en/latest/_images/intnorm_illustration.png)

![Image](https://staff.fnwi.uva.nl/r.vandenboomgaard/ComputerVision/_images/contraststretch.png)

📌 **关键特性**：

- 保持像素之间“比例关系”
    
- 不改变直方图形状，只拉宽
    
---

## 六、直方图均衡（Histogram Equalisation）

📍 _Slide 10–13_

### 6.1 核心思想（老师反复强调）

> **目标不是“看起来好看”，  
> 而是“让灰度值使用得更均匀”**

---

### 6.2 数学核心（累积直方图）
  
![[Pasted image 20260524123947.png]]
    
- 结果：  
    👉 近似“平坦直方图”
    

![Image](https://www.researchgate.net/publication/305421677/figure/fig1/AS%3A449003541274624%401484062194550/mages-and-their-histograms-before-and-after-application-of-Bi-Histogram-Equalization-The.png)

![Image](https://www.researchgate.net/publication/374850019/figure/fig4/AS%3A11431281200213351%401697805054050/Eye-image-processing-1-before-histogram-equalization-2-after-histogram-equalization.png)

---
![[Pasted image 20260206111704.png]]**直方图均衡的目标：**  
利用**原图的累积直方图（CDF）**，把每个灰度级 ppp 映射到一个新的灰度级 E(p,O)E(p,O)E(p,O)，  
使得**输出图像的直方图尽量“平坦”**（每个灰度区间像素数差不多）。
#### 左上：Original histogram（原始直方图）

- 横轴：**Brightness（灰度值）**
    
- 纵轴：**Number（像素数量）**
    
- 蓝色曲线：原图的灰度分布
    
- 红点 + 标注 **ppp**：  
    👉 表示“**当前正在处理的某一个灰度级 ppp**”
    

旁边文字解释（来自课件）：

- **O(p**：灰度级 ppp 处的直方图值（像素个数）
    
- **$\sum_{l=0}^{p} O(l)$**：  
    👉 从 0 到 ppp 的**累积直方图（CDF）**
    

> 这一步的核心信息：  
> **“灰度 ppp 之前，一共出现了多少像素？”**
> 这里是在**把图上的量转成公式语言**：

- **N^2**  
    👉 图像总像素数（假设是 $N \times N$ 图像）
    
- **O(p)**
    👉 原始直方图在灰度 ppp 的值
    
- **$\sum_{l=0}^{p} O(l)$**  
    👉 累积直方图（CDF）
    
- **E(p,O)**  
    👉 灰度 ppp 在“均衡化之后”被映射到的新灰度值
#### 左下：Target histogram（目标直方图）

这是**理想状态**：

- 横轴：灰度范围 [N_{\min}, N_{\max}]
    
- 纵轴：像素数量
    
- 蓝色水平线：  
    👉 **“每个灰度区间的像素数量应该一样多”**

这句话非常关键（等价翻译）：

> **“新的灰度值 E(p,O) 由原图在 p 之前累计了多少像素决定。”**
> 
#### 黄色箭头在“演示什么映射关系”？

这是这张图**最重要、也是最容易被忽略的部分**。

黄色箭头表示一个逻辑链条：

1. **在原图中**  
    灰度 p 之前有
    
    $\sum_{l=0}^{p} O(l)$个像素
    
2. **在目标（平坦）直方图中**  
    你也应该“走过同样数量的像素”
    
3. 于是，在目标直方图的横轴上，  
    找到一个位置 **E(p,O）**，  
    使得：
    
    走过的面积=$\sum_{l=0}^{p} O(l)$

👉 **这一步就是“用累积概率做映射”**

---

## 四、公式是怎么从图里“读出来的”？

图右下角给出了关键推导。

### 1️⃣ 平坦直方图的“密度”

在目标直方图中：

- 总像素数：$N \times N$
    
- 灰度范围：$N_{\max}-N_{\min}​$
    

所以单位灰度对应的像素数是：

$\frac{N^2}{N_{\max}-N_{\min}}$

---

### 2️⃣ 面积守恒（图形直觉）

$在目标直方图中，从 ⁡N_{\min}​ 到 E(p,O) 的“面积”应等于：$

$\sum_{l=0}^{p} O(l)$

于是有：

$\frac{N^2}{N_{\max}-N_{\min}} \big(E(p,O)-N_{\min}\big) = \sum_{l=0}^{p} O(l)$

---

### 3️⃣ 解出映射公式（你课件中的最终式）

 $E(p,O) = \frac{N_{\max}-N_{\min}}{N^2} \sum_{l=0}^{p} O(l) + N_{\min}$ 

👉 这正是你图右下角蓝色公式的来源。
- **直方图均衡是“非线性映射”**
    
    - 不像 normalisation 只是拉伸
        
    - 映射由数据分布本身决定
        
- **CDF 是关键**
    
    - 用“排在第几名”的思想重新分配灰度
### 6.3 Normalisation vs Equalisation（必考对比）

||Normalisation|Equalisation|
|---|---|---|
|是否线性|✅|❌|
|是否改形状|❌|✅|
|是否考虑人眼|❌|✅|
|Matlab imagesc|✅|❌|

---
### 最关键的考点：可逆性 (Invertibility)

这是老师在录音中反复提醒的理论重点：

- **强度归一化是可逆的 (Invertible)**：如果你知道缩放系数，你可以从结果图像完全恢复出原始数据 。
    
- **直方图均衡化是不可逆的 (Not Invertible)**：老师特别指出 "this is not inverse" 。
    
    - **原因**：均衡化过程可能会将原本不同的两个灰度级合并映射到同一个新的灰度级（多对一映射）(**但之所以会出现“合并（Merge）”的情况，核心原因在于数字图像是离散的整数系统。公式计算出的结果通常是小数，而在存回图片时必须强制取整**)，或者因为离散化导致信息丢失。一旦做了均衡化，就**无法**精确还原回原始的像素值了 。
        
    - **后果**：如果在做科学计算或数据分析，要小心使用均衡化，因为它改变了数据的物理意义；但如果只是为了让人眼看清细节（Display），它是很好的工具 。
        

### 总结速记

- **归一化 (Top)** = 线性拉伸 = `imagesc` = **可逆** = 保持原始数据比例。
    
- **均衡化 (Bottom)** = 非线性调整 = 增强对比度 = **不可逆** = 适合人眼观察，但会合并灰度级
## 七、阈值化（Thresholding）

- **基本原理**：选取阈值 $\tau$，大于 $\tau$ 设为白（255），小于则设为黑（0）。
    
- **Otsu 方法 (大津法)**：
    
    - **目的**：自动寻找最佳阈值 。
        
    - **原理**：寻找一个阈值，使得类间方差（Inter-class variance）最大，或类内方差（Intra-class variance）最小 。
        
    - **公式**：$\max_{\tau} \omega_0(\tau)\omega_1(\tau)[\mu_0(\tau) - \mu_1(\tau)]^2$ 。
    - ![[Pasted image 20260524130518.png]]
    - **(1) 类内方差 (Intra-class Variance)**

- **含义**：**同一个类内部**的像素，长得有多“乱”。
    
- **目标**：**最小化**。
    
- **直观理解**：我们希望分出来的“前景”里的像素越相似越好（都是白的），“背景”里的像素也越相似越好（都是黑的）。如果“前景”里混进了很多黑点，方差就会变大，说明分得不好。
- **(2) 类间方差 (Inter-class Variance)**

- **含义**：**两个类之间**的平均亮度差别有多大。
- **目标**：**最大化**。
- **直观理解**：我们希望分出来的“前景”和“背景”区别越大越好（一个很亮，一个很暗），这样对比度才强烈，分割才清晰。

---

## 九、教师重点复述（单独整理）

老师在多页反复强调：

1. **Point operators ≠ understanding**
    
2. Histogram 是核心工具
    
3. Normalisation ≠ Equalisation
    
4. Thresholding 用得非常多
    
5. Shape 是更高层（Slide 17）

| *直方图均衡化** (Histogram Equalization) | $E(p) = \frac{N_{max}-N_{min}}{N^2}\sum_{l=0}^{p}O(l) + N_{min}$ | 把“贫富差距”大的图像，强行变成“共同富裕”，让每个亮度级都有像素。 |
| ---------------------------------- | ---------------------------------------------------------------- | ---------------------------------- |
| **Otsu 阈值法** (Otsu's Method)       | $\max_{\tau} \omega_0\omega_1(\mu_0 - \mu_1)^2$                  | 自动找一个切分点，让前景和背景这两个类别的差异（方差）最大。     |
**Q3: 直方图均衡化后，直方图一定是完全平坦的吗？**

- **A**: **不一定**。这是离散信号。如果原始图像中有大量像素具有完全相同的灰度值（例如大片背景），均衡化无法将这些像素拆分到不同的灰度级，只能把它们整体移动。所以结果是“尽量平坦”，但在某些值上仍会有高峰 。
## 十、考试 / 作业风格思考题

1. 为什么直方图均衡是非线性的？
    
2. 为什么 Otsu 假设图像是双峰分布？
    
3. Normalisation 和 Equalisation 什么时候各自更合适？
    
4. 为什么点运算不能检测边缘？
    

---

## 十一、速查卡（考前 30 秒）

```text
Point operator：逐像素，不看邻域
Histogram：灰度分布
Normalisation：线性拉伸
Equalisation：累积分布映射
Thresholding：灰度 → 二值
Otsu：最大类间方差
```

---
![[Pasted image 20260524131038.png]]![[Pasted image 20260524131047.png]]