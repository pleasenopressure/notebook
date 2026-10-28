
# 2. 这节课在 Computer Vision pipeline 中的位置

这节课解决的问题是：

> 如何从 2D image / multiple images / depth sensor signal 推断 3D scene structure。

前面的 lecture 大多处理的是 **2D image understanding**：feature extraction、classification、matching、retrieval。到了 Towards 3D Vision，问题变成：图像中的一个 pixel 到底来自真实世界中的哪个 3D point？

在 pipeline 中可以理解为：

```text
2D image / stereo images / multi-view images / depth sensor signal
        ↓
camera model + calibration
        ↓
correspondence / disparity / triangulation / depth sensing
        ↓
depth map / 3D point cloud / 3D mesh / camera pose
        ↓
3D applications: SLAM, self-driving, AR/VR, motion capture, forensics
```

**输入**可以是单张图片、stereo image pair、多视角图片、RGB-D sensor、LiDAR、structured light、Time-of-Flight sensor。

**输出**可以是 depth map、3D points、camera pose、3D reconstruction、mesh model。

它和前后 lecture 的关系：

- 和 **Local features / matching / consistent matching** 关系很强：wide-baseline 3D reconstruction 需要 SIFT/local features、feature matching、RANSAC、geometric constraints。
    
- 和 **BoVW / classification** 的区别是：BoVW 更关注 “这张图是什么”，3D vision 更关注 “这个世界的几何结构是什么”。
    
- 下一讲 Special Lecture / Media Production 把本讲的 3D reconstruction 思想应用到真实 film production pipeline，例如 capture → depth estimation → mesh generation → surface reconstruction → rendering。
    

---

# 3. 系统复习笔记

## 3.1 Depth perception：人类如何从 2D 图像感知 3D

handout 先从 human visual system 入手，列出 monocular cues 和 binocular cues。

### A. Monocular cues：单眼线索

单眼也能感知深度，因为我们的大脑利用经验和图像规律。

### 1. Motion Parallax

靠近我们的物体移动得更快，远处物体移动得更慢。

老师举的例子是坐火车看窗外：近处树木快速移动，远处房子慢一些，山几乎不动。这个 cue 可以让大脑推断 depth。

【容易考】解释 motion parallax：

```text
当 observer/camera 运动时，near objects 在 image plane 上产生更大的 apparent motion；
far objects 的 apparent motion 较小。
```

---

### 2. Occlusion

如果 A 挡住 B，通常 A 更近。

直觉：

```text
谁挡住谁，谁更靠前。
```

失败情况：如果物体形状特殊，可能产生误判。但自然场景里这种巧合较少，所以 occlusion 是强 depth cue。

---

### 3. Relative Size

如果我们知道两个物体真实大小接近，那么 image 上更大的通常更近。

例子：两辆车、两栋房子。如果真实大小大致相同，图像中更大的那个通常离相机更近。

【易错点】不是 “图像上大 = 一定近”，而是要有 prior knowledge：真实大小应该类似。

---

### 4. Texture Gradient

远处 texture 更密、更细、更模糊；近处 texture 更稀疏、更清楚。

例如地面砖块、草地、路面纹理。透视投影下，越远的纹理元素在图像上越小，排列越密。

---

### 5. Aerial Perspective / Distance Fog

远处因为空气、雾、散射，看起来更灰、更模糊、更低对比度。

失败情况：天气、雾、光照会干扰。这个 cue 是 heuristic，不是严格几何。

---

### 6. Shading

大脑默认光源通常来自上方，所以根据亮暗判断凸起/凹陷。

老师强调了一个很经典的 illusion：同一张 shading 图旋转 180° 后，原本看起来 convex 的区域会变成 concave。这说明 shading cue 很依赖人类经验，不是绝对可靠。

【设计/解释题】如果问 “为什么 shading 可以给出 depth cue？”  
答：因为 surface normal 与 light direction 决定 brightness，亮暗分布反映局部表面朝向。

---

### 7. Accommodation / Focus

眼睛通过调节 lens focus 到不同距离的物体。近处和远处需要不同 focus。这个属于 monocular physiological cue。

---

## 3.2 Binocular cues：为什么需要两只眼睛

handout 明确列出 binocular cues：**convergence** 和 **stereopsis / binocular disparity**。

### 1. Convergence

当看近处物体时，两只眼睛需要更大角度向内转；看远处物体时，两眼几乎平行。

变量直觉：

- $$\alpha, \beta$$：两眼 convergence angle
    
- 物体越近，convergence angle 越大
    
- 物体越远，convergence angle 越小
    

---

### 2. Binocular Disparity / Stereopsis

左右眼位置不同，所以同一个 3D point 在 left retina 和 right retina 上的位置略有不同。这个差异叫 **disparity**。

handout 用 $$\gamma_L - \gamma_R$$ 表示 disparity。

老师举了 3D cinema / VR 的例子：3D 电影给左眼和右眼分别显示略有差异的图像，大脑融合后产生 depth perception。

【高频】这就是 stereo vision 的人类版本：

```text
two slightly different views
        ↓
find corresponding points
        ↓
measure disparity
        ↓
infer depth
```

---

# 4. Camera Geometry：从 3D 点到 2D 像素

这是本讲最核心的数学部分。

handout 第 9–12 页给出 camera geometry：一个 world point 通过 camera model 投影到 image point，并区分 **extrinsic parameters** 和 **intrinsic parameters**。

## 4.1 核心投影公式

标准写法：

$$  
\lambda  
\begin{bmatrix}  
x \  
y \  
1  
\end{bmatrix}

K  
\begin{bmatrix}  
R & t  
\end{bmatrix}  
\begin{bmatrix}  
X \  
Y \  
Z \  
1  
\end{bmatrix}  
$$

也可以写成：

$$  
\lambda \mathbf{x} = K [R|t]\mathbf{X}  
$$

其中：

- $$\mathbf{X} = [X,Y,Z,1]^T$$：world coordinate 中的 3D point，homogeneous coordinates
    
- $$\mathbf{x} = [x,y,1]^T$$：image plane 上的 2D pixel point，homogeneous coordinates
    
- $$\lambda$$：scale factor，因为 homogeneous coordinates 只定义到比例
    
- $$R$$：rotation matrix，描述 camera orientation
    
- $$t$$：translation vector，描述 camera position
    
- $$[R|t]$$：extrinsic parameters，把 world coordinates 转到 camera coordinates
    
- $$K$$：intrinsic matrix，把 camera coordinates 转到 pixel coordinates
    

---

## 4.2 Intrinsic parameters

常见 intrinsic matrix：

$$  
K =  
\begin{bmatrix}  
f_x & 0 & p_x \  
0 & f_y & p_y \  
0 & 0 & 1  
\end{bmatrix}  
$$

变量含义：

- $$f_x, f_y$$：focal length，以 pixel 为单位，分别对应 x/y 方向
    
- $$p_x, p_y$$：principal point，也就是 optical axis 与 image plane 交点在 sensor 上的位置
    
- 有时还会考虑 skew 和 radial distortion，但这门课重点是 focal length、principal point、distortion 的概念
    

老师在 transcript 里特别解释：理论上 lens center 和 sensor center 应该对齐，但机械制造中很小的 misalignment，比如 0.001mm，都可能在高分辨率图像中造成几个 pixel 的误差，所以 principal point 和 distortion 需要 calibration 补偿。

---

## 4.3 Extrinsic parameters

Extrinsic parameters 描述 camera 相对于 world 的位置和朝向：

$$  
[R|t]  
$$

- $$R$$：rotation，camera 朝哪个方向看
    
- $$t$$：translation，camera 在世界坐标中的位置
    

【易错点】  
Intrinsic 是相机自己的内部属性；extrinsic 是相机放在哪里、朝哪里看。

```text
Intrinsic: camera itself
Extrinsic: camera pose in the world
```

老师也强调：extrinsic 不是 camera 的 “nature”，而是取决于你怎么摆放 camera。

---

# 5. Camera Calibration

handout 定义：**Camera calibration is the process of estimating the parameters of a camera**，还包括真实镜头的 non-linear radial distortion parameters，通常通过 calibration pattern 的 point correspondences 求解。

## 5.1 Calibration 要估计什么？

Camera calibration 主要估计：

1. **Intrinsic parameters**
    
    - $$f_x, f_y$$
        
    - $$p_x, p_y$$
        
    - radial distortion parameters，例如 $$k_1, k_2$$
        
2. **Extrinsic parameters**
    
    - $$R$$
        
    - $$t$$
        
3. **Multi-camera calibration**
    
    - 每个 camera 自己的 intrinsic
        
    - camera 之间的 relative pose / extrinsic relationship
        

---

## 5.2 为什么 calibration 很重要？

老师说得非常明确：camera calibration 是 3D computer vision 里最重要的过程之一，因为如果不知道 intrinsic/extrinsic，就没有办法把 image world 和 real world 联系起来。

【老师强调】【高频】

```text
No calibration
→ no reliable mapping between 2D pixels and 3D world points
→ depth / triangulation / reconstruction will be wrong
```

---

## 5.3 Calibration 的直觉流程

```text
Input:
    calibration pattern, e.g. checkerboard
    known 3D / real-world coordinates of pattern points
    multiple images from camera

Process:
    detect corners / feature points on checkerboard
    match image points to known real-world points
    solve projection equations
    optimise intrinsic + extrinsic + distortion parameters

Output:
    camera matrix K
    distortion parameters
    camera pose R, t
```

pseudocode：

```text
for each calibration image:
    detect checkerboard corners
    assign each detected corner to known world coordinate

initialise K, R, t, distortion parameters

minimise reprojection error:
    projected_point = project(K, R, t, world_point)
    error = distance(projected_point, observed_image_point)

return calibrated parameters
```

核心优化目标：

$$  
\min_{K,R,t,k}  
\sum_i  
\left|  
\mathbf{x}_i -  
\hat{\mathbf{x}}_i(K,R,t,k,\mathbf{X}_i)  
\right|^2  
$$

变量含义：

- $$\mathbf{X}_i$$：已知 3D calibration point
    
- $$\mathbf{x}_i$$：图像中检测到的 2D point
    
- $$\hat{\mathbf{x}}_i$$：用 camera model 投影出来的 2D point
    
- $$k$$：distortion parameters
    
- 目标：让 projected point 尽量接近 observed point
    

---

# 6. Measuring Depth：从 2D 回到 3D

## 6.1 单张图片为什么无法唯一恢复 3D？

单个 image point 对应的是从 camera center 出发的一条 ray。ray 上有无限多个 3D points 都会投影到同一个 pixel。

```text
one pixel
→ one projection ray
→ infinitely many possible 3D points
```

所以单目几何上是 ill-posed inverse problem。

【易错点】  
单张图像可以用 heuristic / learning / shading / known object size 估计 depth，但严格几何上不能唯一确定 depth。

---

## 6.2 Stereo Vision：两张图如何恢复深度

如果有两个 camera，并且知道 calibration，那么同一个 3D point 会投影到两个 image points：

```text
left image point + right image point
        ↓
back-project two rays
        ↓
rays intersect / closest intersection
        ↓
3D point
```

这个过程叫 **triangulation**。

老师在 transcript 中强调：一个 view 只能得到一条 ray；有第二个 view 后，两条 ray 相交，就能得到 3D point。

---

# 7. Epipolar Geometry

## 7.1 为什么需要 epipolar geometry？

对应点搜索太贵。如果在 right image 里为 left image 的一个点搜索整个 2D image，非常慢，而且容易错。

Epipolar geometry 告诉我们：

> left image 中一个点，在 right image 中的对应点一定落在某条 epipolar line 上。

所以搜索范围从 2D image 降成 1D line。

handout 和 CV23-Summary 都把 epipolar geometry / epipolar line 作为 3D vision 复习重点。

---

## 7.2 直觉解释

假设左相机看到一个 pixel：

```text
left pixel
→ back-project to a 3D ray
→ this 3D ray projected into right image
→ becomes an epipolar line
```

因此 right image 中的 matching point 不用到处找，只需要沿着 epipolar line 找。

【计算/设计题】如果问 “epipolar geometry 有什么用？”  
答案：减少 correspondence search space，从 2D search 变成 1D search，提高效率，也降低 mismatch 风险。

---

# 8. Rectification and Disparity

## 8.1 Rectification

handout 第 19 页：rectification 把两张图 warp 到一种更简单的 epipolar geometry，使 matching pixels 位于同一 scanline 上。

老师解释得很清楚：epipolar line 每次都要算很麻烦，所以通常把两张图投影到两个平行 image planes 上。这样对应点就在同一行，matching 只需要横向搜索。

```text
before rectification:
    match along arbitrary epipolar line

after rectification:
    match along same image row
```

---

## 8.2 Disparity

对于 rectified stereo pair，同一个点在左右图中的 x 坐标差叫 **disparity**：

$$  
d = x_L - x_R  
$$

深度公式：

$$  
Z = \frac{fB}{d}  
$$

变量含义：

- $$Z$$：depth，物体到 camera 的距离
    
- $$f$$：focal length
    
- $$B$$：baseline，两台相机之间的距离
    
- $$d$$：disparity，左右图像中对应点的水平位移
    

【计算题】这条公式非常容易考。

直觉：

```text
near object → large disparity → small Z
far object → small disparity → large Z
```

如果 disparity 接近 0，说明物体非常远，depth 会变得非常大，同时对噪声非常敏感。

---

## 8.3 Stereo reconstruction algorithm

```text
Input:
    left image, right image
    camera calibration parameters

1. Calibrate cameras
2. Rectify stereo pair
3. For each pixel / feature in left image:
       search corresponding point along same row in right image
4. Compute disparity:
       d = x_L - x_R
5. Compute depth:
       Z = fB / d
6. Back-project pixel + depth to 3D point
7. Build depth map / point cloud
```

---

# 9. Narrow Baseline Stereo vs Wide Baseline Stereo

CV23-Summary 明确把两类 matching problem 分开：**Narrow-baseline stereo – Template matching**，**Wide-baseline stereo – Feature matching**。

## 9.1 Narrow Baseline Stereo

两个 camera 距离近、视角差小。

特点：

- 图像外观变化小
    
- 可以使用 template matching / block matching
    
- rectification 很有效
    
- 适合 dense disparity map
    

流程：

```text
calibration → rectification → template/block matching → disparity map → depth
```

优点：

- 可以得到 dense depth
    
- matching 相对简单
    
- stereo camera 常用这种设计
    

缺点：

- baseline 太小会导致 disparity 小，远处深度不准确
    
- textureless region 难匹配
    
- reflective / transparent surface 容易失败
    
- occlusion 区域没有对应点
    

---

## 9.2 Wide Baseline Stereo

两个 camera 相隔远，或者图片来自不同时间、不同角度、不同设备。

特点：

- viewpoint 变化大
    
- 不能简单用 template matching
    
- 需要 local features，例如 SIFT
    
- 需要 robust matching 和 geometric verification
    
- 常见于 “Building Rome in a Day” 这种从互联网图片重建 3D scene 的任务
    

老师强调 wide baseline 中图片可能来自不同时间、白天晚上、晴天雨天、不同视角，所以要用 local features，而不是简单 template matching。

流程：

```text
detect local features
extract descriptors
match descriptors
remove outliers with geometric constraints / RANSAC
estimate camera pose
triangulate 3D points
bundle adjustment / joint optimisation
```

优点：

- 可以从普通照片集合重建 3D scene
    
- 不要求固定 stereo rig
    
- 适合 large-scale reconstruction
    

缺点：

- matching 更难
    
- occlusion、illumination、viewpoint changes 更严重
    
- 结果通常是 sparse point cloud，不一定 dense
    
- 需要 robust optimisation
    

---

# 10. Monocular / Single-sensor depth techniques

handout 后半部分介绍了 single sensor / active sensing 的方法，包括 structured light、LiDAR、PrimeSense/Kinect、Time-of-Flight。

老师强调：严格来说，单张图像理论上不可能唯一恢复 depth，但可以通过额外信息或主动传感得到 depth。

---

## 10.1 Structured Light Imaging

基本思想：

```text
project known light pattern onto object
        ↓
3D surface distorts the pattern
        ↓
observe distortion with camera
        ↓
infer surface shape / depth
```

优点：

- 可以获得较密集的 surface geometry
    
- 对受控环境有效
    

缺点 / 失败情况：

- visible pattern 会改变物体外观，不适合某些场景
    
- 对强光、反光、透明物体敏感
    
- 如果 pattern 看不到，就无法恢复 depth
    

老师提到这种方法可以通过 pattern distortion 获取 surface curvature，但也说它 “not very nice”，因为会改变颜色/外观，使用起来麻烦。

---

## 10.2 LiDAR

基本思想：

```text
emit laser
measure return signal / distance
construct 3D points
```

优点：

- 精度高
    
- 可用于工业测量、自动驾驶、建筑扫描
    

缺点：

- 贵
    
- 可能慢
    
- 设备复杂
    
- 对某些材料/表面也会失败
    

老师举例说 LiDAR 非常准确，但设备昂贵且扫描速度可能较慢，因此更适合工业环境。

---

## 10.3 PrimeSense / Kinect 1：coded structured IR light

handout 明确写了 Kinect 1 的原理：使用 coded structured IR light，IR emitter 投射随机点阵，IR camera 记录点阵，通过 template matching 比较实际 dot location 和已知距离平面上的 dot location 来估计 depth。

流程：

```text
IR emitter projects known dot pattern
IR camera observes distorted dot pattern
template matching compares observed vs expected dot positions
compute depth
RGB camera provides colour
```

失败情况：

- window / mirror / monitor 等反光或透明物体无法正常反射 IR
    
- emitter 和 sensor 有位置差，会造成 occlusion gap
    
- pattern 不够 dense，depth map 可能稀疏或有 holes
    

老师特别解释：某些 gap 来自 emitter 和 camera 的位置差，camera 看得到的区域，不一定被 IR pattern 投射到，因此会产生 depth holes。

---

## 10.4 Time-of-Flight Imaging / Azure Kinect

基本思想：

```text
emit modulated light
light travels to object and returns
measure phase shift / time delay
compute distance
```

直觉：

```text
光走得越久，物体越远。
phase shift 越大，对应距离越远。
```

优点：

- depth 更 dense
    
- 通常比 coded pattern 方法更平滑、更方便
    
- 适合 real-time RGB-D sensing
    

缺点：

- occlusion 仍然存在
    
- reflective / transparent surfaces 仍然困难
    
- multi-path reflection 可能造成错误
    
- 多个 sensor 之间可能互相干扰
    

---

# 11. 方法优点、缺点、失败情况总表

|方法|核心输入|输出|优点|缺点 / 失败情况|
|---|---|---|---|---|
|Monocular cues|single image|approximate depth perception|不需要额外相机|heuristic，不保证正确|
|Camera calibration|calibration images + known pattern|intrinsic/extrinsic/distortion|连接 2D image 和 3D world|corner detection 错、pattern 不准会影响结果|
|Narrow baseline stereo|calibrated stereo pair|dense depth map|可做 dense reconstruction|textureless、occlusion、small disparity 失败|
|Wide baseline stereo|many images / large viewpoint changes|sparse 3D points + camera pose|可从互联网照片重建场景|matching 难，需要 robust features/RANSAC|
|Structured light|projected known pattern|dense-ish depth|controlled scene 有效|可见光影响外观，反光/透明/强光失败|
|LiDAR|laser measurement|accurate point cloud|精度高、距离远|昂贵、慢、设备复杂|
|Kinect / coded IR|IR dot pattern|RGB-D|便宜、实时|IR 反射问题、occlusion holes|
|Time-of-Flight|phase/time delay|dense depth|实时、较 dense|reflective surface、multi-path、occlusion|

---

# 12. 老师强调内容提取

## 明确强调

1. 【老师强调】【高频】3D vision 对 final exam 很重要。老师开头说这部分是最后两讲 3D reconstruction 相关内容，并且对 final exam 很重要。
    
2. 【老师强调】Camera calibration 非常重要。老师说 calibration 是 3D computer vision 中最重要的过程之一，因为没有它无法把 image world 和 real world 连接起来。
    
3. 【老师强调】Narrow baseline vs wide baseline 是 very important concept。transcript 中老师明确说 narrow baseline 和 wide baseline 是重要概念。
    
4. 【老师强调】Rectification 的作用是让 matching point 在同一 scanline 上，从而简化 matching。
    
5. 【老师强调】Single image depth 从严格几何上不唯一，但可以用 structured light、LiDAR、Kinect、ToF 等额外 sensing 解决。
    

---

## 老师举的例子

- 火车窗外：motion parallax
    
- 两辆车 / 草地 / 树木：relative size、occlusion、texture gradient
    
- 旋转 shading 图：convex / concave illusion
    
- 3D cinema / VR glasses：binocular disparity
    
- checkerboard calibration：已知真实尺寸，对应 image points，求 camera parameters
    
- RGB-D camera 多摄像头 calibration：对齐不同 camera 的图像
    
- Rome / Venice reconstruction：wide-baseline / internet photo 3D reconstruction
    
- Kinect / IR pattern / ToF：single-sensor depth
    
- self-driving、SLAM、virtual tourism、forensics、motion capture：应用场景
    

---

## 快速带过、考试优先级较低

- 某些具体设备价格、老师自己的项目细节、演示视频细节，不需要死背。
    
- 应用场景可以记 3–5 个典型例子，但不用背所有 slide。
    
- LiDAR 具体精度数字不如理解 “accurate but expensive/slow” 重要。
    
- Azure Kinect 的具体硬件型号不如理解 Time-of-Flight principle 重要。
    

---

# 13. 考试重点标注

## 【高频】

- Camera projection model:  
    $$\lambda \mathbf{x} = K[R|t]\mathbf{X}$$
    
- Intrinsic vs extrinsic parameters
    
- Camera calibration 的目的与流程
    
- Epipolar geometry 如何减少 matching search
    
- Rectification 的作用
    
- Disparity-depth relationship:  
    $$Z = \frac{fB}{d}$$
    
- Narrow baseline vs wide baseline
    
- Stereo vs monocular / active sensing
    

---

## 【计算题】

1. 给定 $$K, R, t, X$$，算 image point $$x,y$$。
    
2. 给定 $$f, B, x_L, x_R$$，算 depth $$Z$$。
    
3. 给定 disparity map，判断哪里近哪里远。
    
4. 给定 reprojection error，解释 calibration optimisation。
    

---

## 【设计/解释题】

1. 给自动驾驶设计 depth sensing pipeline。
    
2. 给博物馆文物扫描选择 LiDAR / structured light / multi-view stereo。
    
3. 给实时 AR/VR 选择 RGB-D / ToF / SLAM。
    
4. 解释为什么 reflective / transparent surfaces 会导致 depth sensing failure。
    

---

## 【对比题】

1. Narrow baseline vs wide baseline
    
2. Stereo matching vs feature matching
    
3. Passive stereo vs active sensing
    
4. Structured light vs Time-of-Flight vs LiDAR
    
5. Monocular cues vs binocular disparity
    

---

## 【易错点】

1. Disparity 大表示近，不是远。
    
2. Intrinsic 和 extrinsic 不要混。
    
3. Epipolar line 不是 depth，本质是 correspondence constraint。
    
4. Calibration 不是只求 focal length，还包括 pose 和 distortion。
    
5. 单张图片不能严格唯一恢复 3D，除非有额外 assumption / active signal / learned prior。
    
6. Rectification 不等于 reconstruction，它只是简化 matching。
    

---

# 14. 考试风格题与答案思路

## 14.1 概念解释题

### Q1. Explain the difference between intrinsic and extrinsic camera parameters.

答案思路：

- Intrinsic 描述 camera 内部成像属性，如 $$f_x, f_y, p_x, p_y$$，以及 distortion。
    
- Extrinsic 描述 camera 在 world 中的位置和方向，即 $$R,t$$。
    
- Intrinsic 是 camera 本身属性；extrinsic 取决于 camera 如何摆放。
    
- 二者共同决定 3D world point 如何投影到 2D image point。
    

---

### Q2. What is camera calibration and why is it important in 3D vision?

答案思路：

- Calibration 是估计 camera parameters 的过程。
    
- 通过 calibration pattern 和 point correspondences 求解 intrinsic/extrinsic/distortion。
    
- 重要性：没有 calibration，就无法可靠地把 pixel ray 和 real-world geometry 联系起来。
    
- depth estimation、triangulation、multi-camera reconstruction 都依赖 calibration。
    

---

### Q3. What is epipolar geometry and how does it help stereo matching?

答案思路：

- 一个 image point 在另一个 view 中的 corresponding point 必须位于 epipolar line 上。
    
- 它把 2D search 降为 1D search。
    
- Rectification 后，epipolar lines 变成 horizontal scanlines，matching 更简单。
    
- 这是 stereo correspondence 的核心 geometric constraint。
    

---

## 14.2 对比题

### Q4. Compare monocular depth cues and binocular disparity.

答案思路：

- Monocular cues 只需要一张图/一只眼，例如 size、occlusion、texture gradient、shading。
    
- Binocular disparity 需要两个 view，利用左右图像位置差。
    
- Monocular cues 多为 heuristic，可能受经验、光照、物体大小假设影响。
    
- Binocular disparity 更几何化，可用于 stereo depth calculation。
    

---

### Q5. Compare narrow-baseline stereo and wide-baseline stereo.

答案思路：

- Narrow baseline：camera close，viewpoint difference small；常用 template/block matching；适合 dense disparity。
    
- Wide baseline：viewpoint difference large；需要 SIFT/local features；适合 unordered photo collections / large-scale reconstruction。
    
- Narrow baseline matching 简单但远处 depth 不稳定；wide baseline 更复杂但应用更灵活。
    
- CV23-Summary 明确对应：narrow-baseline stereo → template matching；wide-baseline stereo → feature matching。
    

---

### Q6. Compare structured light, LiDAR, and Time-of-Flight.

答案思路：

- Structured light：投射已知 pattern，看 distortion。
    
- LiDAR：发射 laser，测距离，高精度但贵。
    
- Time-of-Flight：测 phase shift / travel time，适合实时 dense depth。
    
- 三者都属于主动 sensing；都可能受到 reflective/transparent surfaces、occlusion、环境光影响。
    

---

## 14.3 计算 / 推导题

### Q7. A stereo camera has focal length $$f=800$$ pixels and baseline $$B=0.10$$ m. A point appears at $$x_L=320$$ and $$x_R=300$$. Compute depth.

答案：

$$  
d = x_L - x_R = 320 - 300 = 20  
$$

$$  
Z = \frac{fB}{d}  
= \frac{800 \times 0.10}{20}  
= 4 \text{ m}  
$$

答案思路：

- 先算 disparity
    
- 再套 $$Z=fB/d$$
    
- disparity 越大，depth 越小
    

---

### Q8. Given

$$  
K=  
\begin{bmatrix}  
1000 & 0 & 320 \  
0 & 1000 & 240 \  
0 & 0 & 1  
\end{bmatrix}  
$$

Assume $$R=I$$, $$t=0$$, and 3D point $$X=(1,2,10)$$. Find image coordinates.

答案：

Camera coordinates:

$$  
X_c=1,\quad Y_c=2,\quad Z_c=10  
$$

Projection:

$$  
x = f_x \frac{X_c}{Z_c} + p_x  
=1000\frac{1}{10}+320  
=420  
$$

$$  
y = f_y \frac{Y_c}{Z_c} + p_y  
=1000\frac{2}{10}+240  
=440  
$$

所以 image point 是：

$$  
(420, 440)  
$$

---

## 14.4 设计应用题

### Q9. You need to build a real-time depth sensing system for indoor human motion capture. Which method would you choose?

答案思路：

- 可选 RGB-D / Time-of-Flight camera，例如 Azure Kinect。
    
- 理由：实时、dense depth、适合人体姿态与 motion capture。
    
- 多个 camera 可以减少 occlusion。
    
- 需要 multi-camera calibration。
    
- 失败情况：遮挡、反光衣物、传感器互相干扰。
    

---

### Q10. You are given thousands of tourist photos of the same landmark from different viewpoints and times. How would you reconstruct 3D structure?

答案思路：

- 这是 wide-baseline / multi-view reconstruction。
    
- pipeline：
    
    ```text
    detect SIFT/local features
    match features between images
    use RANSAC/geometric constraints to remove outliers
    estimate camera poses
    triangulate sparse 3D points
    optimise via bundle adjustment
    ```
    
- 不适合 simple template matching，因为 viewpoint/lighting/time difference 很大。
    
- 可输出 sparse point cloud 和 camera positions。
    

---

# 15. 考前速记版

## 15.1 Must-know points

1. 3D vision 的核心是把 **2D image point** 和 **3D world point** 联系起来。
    
2. Camera projection model：  
    $$\lambda \mathbf{x} = K[R|t]\mathbf{X}$$
    
3. Intrinsic = camera 内部参数；extrinsic = camera pose。
    
4. Camera calibration 估计 intrinsic、extrinsic、distortion，是 3D vision 的基础。
    
5. 单张图像几何上不能唯一恢复 depth，因为一个 pixel 对应一条 3D ray。
    
6. Stereo 通过 correspondence + triangulation 恢复 3D。
    
7. Epipolar geometry 把 matching search 从 2D 降到 1D。
    
8. Rectification 让对应点落在同一 scanline。
    
9. Stereo depth：  
    $$Z=\frac{fB}{d}$$
    
10. Narrow baseline 用 template matching；wide baseline 用 feature matching。
    

---

## 15.2 High-risk mistakes

1. 把 disparity 和 depth 关系说反：**disparity 越大，depth 越小**。
    
2. 把 intrinsic / extrinsic 混淆。
    
3. 以为 rectification 直接给 depth；实际上它只是简化 matching。
    
4. 以为 monocular image 可以严格恢复 3D；实际上需要 assumption 或额外 sensing。
    
5. 忽略 calibration，直接用 pixel distance 当真实距离。
    

---

## 15.3 Likely exam questions

1. Explain camera calibration and distinguish intrinsic/extrinsic parameters.
    
2. Derive or apply the stereo depth formula $$Z=fB/d$$.
    
3. Compare narrow-baseline stereo and wide-baseline stereo.
    
4. Explain epipolar geometry and rectification.
    
5. Choose a 3D sensing method for a real-world application and justify trade-offs.
    

---

## 15.4 最应该背 / 理解的公式与流程

### 投影公式

$$  
\lambda  
\begin{bmatrix}  
x \  
y \  
1  
\end{bmatrix}

K  
\begin{bmatrix}  
R & t  
\end{bmatrix}  
\begin{bmatrix}  
X \  
Y \  
Z \  
1  
\end{bmatrix}  
$$

### Stereo depth

$$  
d = x_L - x_R  
$$

$$  
Z = \frac{fB}{d}  
$$

### Stereo pipeline

```text
calibrate cameras
→ rectify images
→ find correspondences along scanlines
→ compute disparity
→ compute depth
→ triangulate / reconstruct 3D
```

### Wide-baseline reconstruction pipeline

```text
detect local features
→ match descriptors
→ reject outliers with RANSAC / geometry
→ estimate camera poses
→ triangulate 3D points
→ optimise reconstruction
```