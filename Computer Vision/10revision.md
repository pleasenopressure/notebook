你好！我是你的课程助教。结合你上传的期末复习课件（Lecture 12 Revision）以及极为关键的课堂录音转写稿，这节课可以说是考前“最核心的冲刺指南”。**老师在录音中直接“透题”了多道期末考试必考的大题思路！**

以下为你提炼的完整梳理和期末学习指南：

---

### 🌟 一、 课程与考试概览 (Exam Overview)

这是一节干货满满的复习课，核心聚焦于底层计算机视觉（Lower-level vision），串联了从图像输入到特征提取的完整流水线：**图像采集 $\rightarrow$ 频域与采样 $\rightarrow$ 图像增强 $\rightarrow$ 边缘检测 $\rightarrow$ 形状检测** 。

**📝 期末考试形式 (Exam Format):**

- **时长与题量**：2小时闭卷，共4道大题（必须全答），每题25分，总分100分 。
    
- **题目分布**：Xiaohao Cai 老师出前2题（聚焦本半期涵盖的底层视觉），Hansung 老师出后2题（聚焦高层视觉）。
    
- **评分标准**：90% 的内容直接来自课件与笔记，10% 考察课外扩展阅读（参考书）。题目包含多个小问，会明确标出分数，请根据分数决定作答篇幅 。


---

### 🔥 二、 教师重点复述 (Teacher's Exam Leaks & Emphasis)

根据课堂录音，老师**明确指出了以下四个必考/极易考知识点**，请务必作为复习的第一优先级：

1. **🚨 致命警告：字迹清晰度**
    
    - **背景**：老师反复强调，考卷字迹如果不清晰，考官无法阅读，将直接不给分（"if we can't find any information from your writing, it's... zero"）。
        
2. **🏆 必考大题 1：边缘提取 + 形状检测综合题**
    
    - **考点**：给你一张图像，要求你写出检测图中特定形状（如直线、圆或自定义形状）的**完整算法设计过程** 。
        
    - **满分思路**：你不能只写一句“用霍夫变换”，必须详细写出前置步骤：**原图 $\rightarrow$ 灰度化 $\rightarrow$ 高斯平滑去噪 $\rightarrow$ Canny 边缘检测 $\rightarrow$ 提取边缘点坐标和梯度方向 $\rightarrow$ 构建/查询 R-table $\rightarrow$ 累加器投票 $\rightarrow$ 寻找峰值** 。
        
    - **对比分析**：必须准备好论述该方法的优缺点。优点：抗遮挡、抗噪声强 。缺点：对于多维参数（如带缩放和旋转的圆），参数空间极大，极其消耗内存和计算时间 。
        
3. **🏆 必考大题 2：图像降频与抗混叠 (Anti-Aliasing)**
    
    - **考点**：给一张高分辨率图像，问如何正确地把它缩小（降采样）。
        
    - **满分思路**：绝对不能直接丢弃像素！必须说明**先使用低通滤波器（如高斯滤波）去除高频信息**，然后再缩小尺寸。如果不做滤波直接缩小，会导致高频信号“混叠（Aliasing）”到低频中，产生伪影 。
        
4. **🏆 必考大题 3：点操作与图像增强 (Point Operators)**
    
    - **考点**：如果给定一张细节看不清的极暗或极亮图像，如何增强可视化效果？
        
    - **满分思路**：使用**直方图均衡化 (Histogram Equalisation)**。不仅要写名字，**必须记住并写出公式**，解释如何计算累积直方图并映射到新的像素值 。
        
5. **🏆 必考大题 4：卷积加速与边界处理**
    
    - **考点**：如何提高图像去噪（高斯滤波）的效率？
        
    - **满分思路**：使用**可分离滤波器 (Separable filter)**，将一个二维的卷积模板拆分为两个一维模板分别进行行列计算，以显著降低时间复杂度 。
        
    - **拓展**：还可能会问边界处理策略（Padding）对输出图像尺寸的影响 。
        

---

### 📖 三、 核心知识与章节梳理 (Detailed Breakdown)

1. 频域与采样 (Lecture 2 & 3)

- **傅里叶变换**：将图像从空间域转换到频域。公式核心在于幅度 $|Fp(\omega)|$ 和相位 $arg(Fp(\omega))$ 。
    
- **奈奎斯特采样定理 (Nyquist Theorem)**：最小采样频率必须大于等于信号最高频率的两倍（$f_{sample} \ge 2 \times f_{max}$），否则就会在频域发生频谱重叠，这就是**混叠 (Aliasing)** 的数学本质 。 _(注：可想象频谱周期性延展时互相发生交叠的现象)_
    

2. 点操作与组操作 (Lecture 4 & 5)

- **直方图均衡化 (Histogram Equalisation)**：
    
    - **目标**：强行拉平直方图，使得像素分布均匀，提高对比度 。
        
    - **公式**：$E(p,0) = \frac{N_{max}-N_{min}}{N^2} \sum_{l=0}^{p}O(l) + N_{min}$ 。(考前必背)
        
- **模板卷积加速 (Template Convolution)**：
    
    - **时域卷积 = 频域乘积**：$P * T = \Im^{-1}(\Im(P) \times \Im(T))$。对于大于 $7 \times 7$ 的模板，利用 FFT 算频域乘积比直接算空间滑动窗口更快 。
        
- **边界处理**：Constant（补黑边）、Wrap-around（环绕）、Symmetric（对称补全）。
    

3. 边缘检测 (Lecture 6 & 7)

- **一阶边缘算子**：Sobel。它输出的是矢量，具有幅值 $M$ 和边缘梯度方向 $\Theta$（极度重要，后续霍夫变换依赖此方向） 。
    
- **二阶边缘算子**：Laplacian of Gaussian (LoG)。通过寻找二阶导数的“过零点(zero-crossing)”来定位边缘 。
    
- **⭐ Canny 边缘检测 (Canny Edge Detector)**：目前最强的经典边缘检测。四步走：
    
    1. 高斯平滑 (Gaussian smoothing)：去噪。
        
    2. Sobel 算子 (Sobel edge detection)：算幅值和方向。
        
    3. **非极大值抑制 (Non-maximum suppression)**：沿着边缘梯度的方向“走”，只保留山脊顶部的最高点，使得边缘变细（单像素宽） 。
        
    4. **滞后阈值化 (Hysteresis thresholding)**：使用高、低两个阈值。强边缘（过高阈值）直接保留，弱边缘（过低阈值但未达高阈值）只有在连接到强边缘时才保留 。
        
计算机视觉中非常基础但极易踩坑的概念：**图像滤波/卷积操作时的“边界处理策略 (Border/Padding Strategies)”**。
当我们在图像上滑动卷积核（如 $3\times3$ 的高斯平滑模板或 Sobel 边缘检测模板）时，如果把模板中心对准图像最边缘的像素，模板的一部分就会“悬空”（超出图像范围），导致无法计算。在遇到类似 _"设计一个降噪/边缘检测系统，并说明相关参数与策略"_ 的大题时，除了写出核心算法，**千万别忘了写边界处理！**

图片列出了三种解决这个尴尬情况的策略 ：

- **策略一：Set border to black (将边界设为黑色)**
    
    +2
    
    - **解释**：最简单粗暴的方法，卷积后图像边缘算不出来的地方，直接全部填 0（黑色）。
        
    - **适用场景**：正如幻灯片右下角所说，因为“我们通常假设感兴趣的目标在图像中心 (Normally we assume object of interest is near centre)”，所以边缘是黑的也无伤大雅 。图中的油画倾斜后，四周填补黑色就是一个直观的例子。
        
- **策略二：Padding the original image (对原图进行像素填充/扩边)**
    
    - **Constant (常量填充)**：用固定的值（比如全 0 或原图边缘的颜色）向外扩展。
        
    - **Wrap-around / Periodic (环绕/周期性填充)**：把图像左边的边界像素“穿越”补到右边，上边的补到下边 。
        
        +2
        
    - **Symmetric (对称/镜像填充)**：以图像边缘为镜面，把内部的像素镜像折叠出去补齐（这种方法在去噪时最不容易产生突兀的假边缘） 。
        
        +2
        
- **策略三：Make template smaller near edges (在靠近边缘处缩小模板)**
    
    - **解释**：不补齐像素，而是当靠近边缘时，原本 $3\times3$ 的模板动态缩小成 $2\times2$ 甚至 $1\times1$ 来适应。这种方法计算逻辑会变得复杂。


4. 形状检测 (Lecture 8 & 9)

- **广义霍夫变换 (GHT, Generalized Hough Transform)**：
    
    - **原理**：针对无解析方程的任意形状，通过建立 **R-table** 来寻找形状的参考点（Reference point）。
        
    - **建表过程**：基于模板形状，以边缘梯度方向 $\phi$ 作为索引，记录该边缘点到参考点的距离 $r$ 和相对角度 $\alpha$ 。
        
    - **检测过程**：$x_c = x_i - r \cdot S \cos(\alpha + \theta)$, $y_c = y_i - r \cdot S \sin(\alpha + \theta)$。利用此公式处理缩放（$S$）和旋转（$\theta$）。
        
    - **陷阱提示**：如果有旋转 $\theta$，在去 R-table 查表之前，必须先将测得的边缘梯度方向减去 $\theta$ ($\phi_{new} - \theta$) 。
        

---

### 💻 四、 考试实战编程思想 (Exam Code/Logic Templates)

针对试卷上的算法设计题（如 Question 3a/b/c 统计直线、圆的个数），你需要写出类似下面的算法伪代码逻辑：

Python

```
# 必考题模板：如何检测并统计图像中的特定形状（例如直线或圆）
def detect_and_count_shapes(image):
    # Step 1: 预处理与降噪 (极重要，通常占分数)
    smoothed_img = apply_gaussian_filter(image, sigma=1.0)
    
    # Step 2: 边缘检测 (首选 Canny，写出步骤)
    edges, gradient_directions = canny_edge_detector(
        smoothed_img, 
        low_threshold=50, 
        high_threshold=150
    )
    
    # Step 3: 初始化霍夫累加器 (Hough Accumulator)
    # 对于直线：参数为 (r, theta)；对于圆：参数为 (x_center, y_center, radius)
    accumulator = initialize_accumulator()
    
    # Step 4: 投票机制 (Voting)
    for each pixel (x,y) in edges:
        if pixel_value > 0:
            # 优化点：利用 gradient_directions 缩小投票范围！
            vote_in_accumulator(accumulator, x, y, gradient_directions[x,y])
            
    # Step 5: 寻找峰值并计数 (Peak Finding)
    peaks = find_local_maxima(accumulator, threshold=voting_threshold)
    
    return len(peaks) # 返回检测到的形状数量
```

---

### 🗂️ 五、 考前一页速查卡 (Revision Cheat Sheet)

|**核心概念**|**核心原理 / 公式**|**优势 / 应用**|**劣势**|
|---|---|---|---|
|**直方图均衡化**|$E(p,0) = \frac{N_{max}-N_{min}}{N^2} \sum_{l=0}^{p}O(l) + N_{min}$|提升全局对比度，增强暗部细节|不改变空间分辨率，可能放大噪声|
|**抗混叠降采样**|高斯低通滤波 $\rightarrow$ 下采样|避免高频信号映射为低频伪影|会丢失锐利细节|
|**Canny 边缘提取**|高斯去噪 + Sobel梯度 + NMS(非极大值抑制) + 双阈值连接|边缘极细（单像素），定位精准|算法复杂，参数多（需要设定高低阈值）|
|**模板卷积加速**|$P * T = \Im^{-1}(\Im(P) \times \Im(T))$|大于7x7模板时，利用FFT极大幅度提速|-|
|**广义霍夫变换 (GHT)**|建立 **R-table**，映射：边缘点 $\rightarrow$ 参数空间峰值|抗部分遮挡、抗噪，能检测任意形状|带缩放/旋转时，参数空间极其庞大 ($O(N^4)$)|

**复习建议：** 重点掌握 Canny 算子的四个文字步骤，以及 GHT 查表解决旋转和缩放的原理。考试时，步骤写得越详细（尤其是写出如何用高斯核平滑、如何用梯度方向优化投票），能拿到的分数就越高。祝你考试顺利 (Good luck)！如果有某道样题需要我给出完整的解答参考，请随时告诉我。