# Lecture 10/11: Improving Performance

## 1. 本 lecture 的资料组合

|项目|内容|
|---|---|
|Handout/PDF|`Improving Performance.pdf`|
|课程录音转写|已提供：`Improving Performance.txt`|
|Notebook|`lecture6/performance.ipynb`、`lecture6/transforms.ipynb`、`lecture6/data-augmentation.ipynb`，你的资料映射表把 Lecture 10 作为理论部分，Lecture 11 作为 notebook 实践部分。|
|Revision 关联|Revision 1 直接出现了 Dropout 和 Batch Normalization；Revision 2 说明考试为 4 题、2 小时、closed-book，Hikmat 负责前 3 题，通常覆盖 lower-level deep learning strategies。|
|考试优先级|**高**|
|原因|这节课把概念、公式、训练流程、`model.train()` / `model.eval()`、validation loop、PyTorch API 全部连在一起，特别适合出代码阅读、bug 识别和概念解释题。|

---

## 2. 这节课在考试中的定位

这节更可能属于 **Hikmat 前 3 题的 lower-level deep learning strategies**。

它不像 CNN 那样特别偏手算 kernel/shape，但非常适合考：

- **概念解释题**：什么是 overfitting？为什么 dropout 能减少 overfitting？
    
- **代码阅读题**：为什么 validation 前要 `model.eval()`？
    
- **bug 识别题**：validation 时忘记 `model.eval()` 会怎样？
    
- **训练流程题**：`zero_grad → forward → loss → backward → step` 的顺序。
    
- **公式题**：BatchNorm 的 mean / variance / normalized output 怎么算。
    
- **设计题**：给一条 train/validation accuracy 曲线，让你判断是否 overfitting，是否该 early stopping。
    

---

## 3. 课程录音转写中的老师重点

### 【老师强调】这节课不是追求“最优模型”，而是看方法是否提升性能

老师开头说，这节课的重点不是把模型调到 100% 最优，而是展示使用 early stopping、dropout、batch normalization、data augmentation 这些方法前后，performance 会有什么差异。

### 【高频】DNN 强表达能力带来两个问题：overfitting 和 convergence

Handout 说 DNN 的 expressiveness 让它能建模复杂输入输出关系，但也会导致 overfitting 和 convergence 问题：模型可能学到训练集噪声，训练也可能很慢甚至不收敛。 老师在录音里补充了直觉：参数越多，模型越灵活，越容易“记住数据”；训练深层网络时，梯度连乘可能变得非常小，导致前面层参数几乎不更新。

### 【容易考】如何判断 overfitting

老师反复强调一个模式：

> training accuracy 继续上升，但 validation accuracy 停住甚至下降。

这说明模型开始 memorise training data，而不是继续提升 generalization。validation performance 是 test performance 的一个指示器。

### 【老师强调】early stopping 不能看到 validation accuracy 一下降就停

老师明确说，validation curve 会抖动，所以不能简单写：

```python
if val_acc < previous_val_acc:
    stop
```

正确思路是看 **trend**，例如保存最近几次 validation loss / accuracy，用 patience 判断是否持续变差。

### 【代码重点】`model.train()` / `model.eval()` 只改变 Dropout 和 BatchNorm 行为

老师特别强调，这两个 switch 不是控制 gradient 的。它们主要影响：

- Dropout：training 时随机置零；eval 时关闭 dropout。
    
- BatchNorm：training 时用当前 mini-batch mean/variance；eval 时用 running mean/variance。
    

老师明确说这和 gradient computation 没关系。

---

## 4. Handout 核心知识点总结

## 4.1 Overfitting

**核心定义**：模型在 training set 上表现越来越好，但在 validation/test set 上不再提升，甚至下降。原因是模型不仅学到了 pattern，也学到了 noise。

**Handout 表达**：DNN 很强大，能学习复杂关系，但也可能学习 training data 中 test data 不存在的噪声，从而降低 generalization。

**老师补充**：如果 training accuracy 接近 99%，但 unseen data 上表现差，这通常不是模型真的“理解了”，而是 memorised training data。

**可能考法**：给你一张 training vs validation accuracy 曲线，问是否 overfitting。

**易错点**：只看 training loss / training accuracy。考试里一定要强调 validation。

---

## 4.2 Early Stopping

**核心定义**：当 validation performance 不再提升时，提前停止训练。

**Handout 表达**：例子中 maximum validation accuracy 在 epoch 14 附近出现；如果在那里停止，不仅 accuracy 更好，还能节省训练时间。最简单的方法之一是 monitor validation loss。

**老师强调**：不要因为 validation accuracy 单次下降就停，要看 trend；validation loss 通常比 accuracy 更敏感，更适合作为 early stopping 指标。

**伪代码**：

```python
best_val_loss = float("inf")
patience_counter = 0

for epoch in range(epochs):
    train_one_epoch()
    val_loss = validate()

    if val_loss < best_val_loss:
        best_val_loss = val_loss
        patience_counter = 0
        save_model()
    else:
        patience_counter += 1

    if patience_counter >= patience:
        break
```

**易错点**：用 test set 做 early stopping。正确做法是用 validation set，test set 只在最后评估。

---

## 4.3 Dropout

**核心定义**：训练时随机把一些 neuron output 设为 0，使网络不能过度依赖某些节点。

**Handout 表达**：每次 training pass 会 drop 不同节点；有 n 个 dropout 节点时，可以看成 2ⁿ 个 smaller networks 的 ensemble，随机噪声会在 averaging 中互相抵消。

**老师补充直觉**：不是物理上删除节点，而是把该节点输出设成 0；每次 iteration 删除不同节点，相当于训练多个共享权重的小网络。

**PyTorch 重点**：

```python
drop = nn.Dropout(p=0.5)
model.train()  # dropout active
model.eval()   # dropout becomes identity
```

PyTorch 中 `nn.Dropout(p)` 的 `p` 是 **drop probability**，不是 keep probability。notebook 里自定义了一个 `DP(prob)`，内部用 `nn.Dropout(1-prob)`，所以这个 `prob` 实际被当成 keep probability。([GitHub](https://raw.githubusercontent.com/UOS-COMP6252/public/main/lecture6/performance.ipynb "raw.githubusercontent.com"))

**易错点**：

- `model.eval()` 不等于 `torch.no_grad()`。
    
- Dropout 在 eval 时不是继续随机 drop，而是 identity。
    
- PyTorch 会在 training 时对保留下来的 activation 做 scaling。
    
**Dropout 在训练时会随机把一部分神经元输出变成 0；为了不让整体输出变小，PyTorch 会把没被丢掉的那些输出放大。**

比如：

```
x = torch.tensor([10., 10., 10., 10.])dropout = nn.Dropout(p=0.5)
```

这里 `p=0.5` 表示 **50% 的 activation 会被丢掉**。

假设这一次随机丢掉第 2、4 个，保留第 1、3 个。

如果不 scaling：

```
原来: [10, 10, 10, 10]drop 后: [10, 0, 10, 0]
```

平均值从 10 变成了 5，整体信号变小了。

所以 PyTorch 会把保留下来的值除以 keep probability：

```
keep probability = 1 - p = 0.5保留下来的值 × 1/0.5 = ×2
```

所以实际输出可能是：

```
[20, 0, 20, 0]
```

这样虽然有一半被丢掉，但从期望上看，整体大小仍然差不多。

---

## 4.4 Batch Normalization

**核心定义**：BatchNorm 对每个 channel 的 mini-batch activation 做 normalization，使其接近 zero mean 和 unit variance，然后再用 learnable parameters `γ` 和 `β` 做缩放和平移。

**Handout 表达**：BatchNorm 可以改善 convergence、提升 generalization、减少 overfitting，并降低训练对 learning rate 和 initialization 的敏感性。

**核心公式**：

给定输入 shape 为：

```text
(S, C, H, W)
```

其中：

- `S`：batch size
    
- `C`：channel 数
    
- `H, W`：height 和 width
    
- 对每个 channel `c`，统计 batch、height、width 上的所有值
    

公式：

$$
[  
N = S \times H \times W  
]
$$
$$
[  
\mu_c = \frac{1}{N}\sum_{i=1}^{N}x_{ci}  
]
$$

$$
[  
\sigma_c^2 = \frac{1}{N}\sum_{i=1}^{N}(x_{ci}-\mu_c)^2  
]
$$
\\
$$
[  
\hat{x}_{ci} = \frac{x_{ci}-\mu_c}{\sqrt{\sigma_c^2+\epsilon}}  
]
$$
\\
$$
[  
y_{ci} = \gamma_c\hat{x}_{ci}+\beta_c  
]
$$

其中 `γ` 和 `β` 是 learnable parameters。Handout 和 notebook 都强调 BatchNorm 是 per-channel 做的；notebook 里用 `a.mean([0,2,3])` 和 `a.var([0,2,3])` 手动验证，即跳过 channel 维度，只在 batch、height、width 上统计。 ([GitHub](https://raw.githubusercontent.com/UOS-COMP6252/public/main/lecture6/performance.ipynb "raw.githubusercontent.com"))

**Training vs inference**：

- Training：用当前 mini-batch 的 mean / variance。
    
- Inference / validation：用 training 阶段累计的 running mean / running variance。
    
这里的 **running mean** 可以理解为：

> **BatchNorm 在训练过程中，慢慢积累出来的“历史平均值”。**

不是某一个 batch 的 mean，而是很多 batch 的 mean 的移动平均。

---

假设 BatchNorm 每次看到一个 mini-batch，都会算这个 batch 的均值：

```text
batch 1 mean = 4
batch 2 mean = 6
batch 3 mean = 5
...
```

但是 test / inference 的时候，我们可能只输入一张图片，或者一个很小的 batch。这个时候如果还用当前 batch 的 mean，会非常不稳定。

所以 BatchNorm 在 training 时会一边正常训练，一边偷偷记录一个“历史平均 mean”：

```text
running mean ≈ 训练过程中见过的 batch mean 的综合平均
```

课件里写的更新方式是：

$$
[  
\hat{\mu}_c \leftarrow 0.9\hat{\mu}_c + 0.1\mu_c  
]
$$

意思是：

```text
新的 running mean =
0.9 × 旧的 running mean + 0.1 × 当前 batch 的 mean
```

也就是说，它不会完全相信当前 batch，而是保留大部分历史信息，再加入一点当前 batch 信息。课件也说明 training 时每一步会算 mini-batch 的 mean/variance，同时更新 running averages；inference 时则使用这些 running averages。

---

举个简单例子：

一开始：

```text
running_mean = 0
```

第一个 batch 的 mean 是 10：

```text
running_mean = 0.9 × 0 + 0.1 × 10 = 1
```

第二个 batch 的 mean 是 20：

```text
running_mean = 0.9 × 1 + 0.1 × 20 = 2.9
```

第三个 batch 的 mean 是 15：

```text
running_mean = 0.9 × 2.9 + 0.1 × 15 = 4.11
```

它不是简单平均：

```text
(10 + 20 + 15) / 3 = 15
```

而是 **moving average / exponential moving average**，更偏向“逐步积累”。

---

最关键区别是：

```text
训练时：
用当前 mini-batch 的 mean 做 normalization
同时更新 running_mean

测试 / validation / inference 时：
不用当前 batch 的 mean
用训练期间积累好的 running_mean
```

老师转写里也强调：running mean / running variance 是 training 时积累的，training 时不用它们做主要 normalization；到了 inference 才使用它们。

---

在 PyTorch 里你可以这样看：

```python
bn = torch.nn.BatchNorm2d(32)

print(bn.running_mean.shape)
print(bn.running_var.shape)
```

如果是：

```python
nn.BatchNorm2d(32)
```

那么：

```text
running_mean 有 32 个值
running_var 有 32 个值
```

因为 BatchNorm 是 **per channel** 做的，每个 channel 都有自己的 running mean 和 running variance。课件也说多个 channel 时，每个 channel 独立做 BatchNorm。

一句话记忆：

> **running mean = BatchNorm 在训练过程中为每个 channel 记录下来的历史平均值，用于 `model.eval()` 阶段稳定地做 normalization。**


**易错点**：

- BatchNorm 不是对整个 tensor 求一个 mean，而是 **每个 channel 单独求**。
    
- `nn.BatchNorm2d(num_features=2)` 说明有 2 个 channel，因此 `γ` 和 `β` 各有 2 个值。
    
- validation/test 时忘记 `model.eval()`，BatchNorm 会继续用当前 batch statistics，结果不稳定。
    

---

## 4.5 Data Augmentation

**核心定义**：通过对训练图片做随机变换，制造更多“合理变化”的训练样本，让模型更鲁棒。

**Handout 表达**：Data augmentation 通常在 PyTorch 中 on-the-fly 完成，也就是通过 transforms 在读取 input 时动态变换；例子包括 rotation、translation、horizontal flip、random crop 等。

**Notebook 体现**：

`transforms.ipynb` 用一张 cat image 展示：

- `RandomHorizontalFlip()`
    
- `RandomAffine(degrees=0, translate=(0.3,0.3))`
    
- `RandomCrop((150,150))`
    
- `RandomRotation(45)`
    

每次重复运行会得到不同图像，体现 random transform。([GitHub](https://raw.githubusercontent.com/UOS-COMP6252/public/main/lecture6/transforms.ipynb "raw.githubusercontent.com"))

`data-augmentation.ipynb` 用 flowers dataset，训练集使用 augmentation，validation set 只做 `Resize + ToTensor`，这是非常重要的代码习惯。([GitHub](https://raw.githubusercontent.com/UOS-COMP6252/public/main/lecture6/data-augmentation.ipynb "raw.githubusercontent.com"))

**易错点**：不要对 validation/test 使用随机 augmentation，否则评估不稳定，也不公平。

---

## 5. Notebook 代码重点

## 5.1 `performance.ipynb`

### 整体流程

1. 导入 `torch`, `torch.nn`, `torchvision`, `DataLoader`, `Adam/SGD`。
    
2. 演示 Dropout training vs eval 行为。
    
3. 手动计算 BatchNorm mean / variance，并和 `nn.BatchNorm2d` 对比。
    
4. 构建 CIFAR10 CNN。
    
5. 用 `norm_layers=True/False` 控制是否使用 BatchNorm。
    
6. 写 `evaluate()` 函数，用 `@torch.no_grad()` 和 `model.eval()`。
    
7. 写 training loop。
    
8. 记录 train/validation loss 和 accuracy。
    
9. 输出 confusion matrix。([GitHub](https://raw.githubusercontent.com/UOS-COMP6252/public/main/lecture6/performance.ipynb "raw.githubusercontent.com"))
    

### Dropout 代码重点

notebook 中：

```python
class DP(nn.Module):
    def __init__(self, prob=0.5):
        super().__init__()
        self.dp = nn.Dropout(1-prob)

    def forward(self, x):
        return self.dp(x)
```

这里 `prob` 是 keep probability，所以用了 `1-prob` 传给 PyTorch 的 `nn.Dropout`。考试里如果直接看到 `nn.Dropout(p=0.3)`，要理解为 drop 30%，keep 70%。

### BatchNorm 代码重点

notebook 用：

```python
var = a.var([0,2,3], unbiased=False)
mean = a.mean([0,2,3])
```

这说明对于 `(S,C,H,W)`，BatchNorm2d 是对 `[batch, height, width]` 统计，每个 channel 独立 normalization。([GitHub](https://raw.githubusercontent.com/UOS-COMP6252/public/main/lecture6/performance.ipynb "raw.githubusercontent.com"))

### CIFAR10 CNN 结构

notebook 中模型大致是：

```python
Conv2d(3, 32, 3)
BatchNorm2d(32)
ReLU
Conv2d(32, 32, 3)
BatchNorm2d(32)
ReLU
MaxPool2d(2)

Conv2d(32, 64, 3)
BatchNorm2d(64)
ReLU
Conv2d(64, 64, 3)
BatchNorm2d(64)
ReLU
MaxPool2d(2)

Flatten
Linear(5*5*64, 128)
ReLU
Linear(128, 10)
```

输入 CIFAR10 是 `(batch, 3, 32, 32)`，经过卷积和 pooling 后 flatten 成 `64*5*5`。这个 shape 很适合考试问。([GitHub](https://raw.githubusercontent.com/UOS-COMP6252/public/main/lecture6/performance.ipynb "raw.githubusercontent.com"))

### Training loop 重点

标准结构：

```python
for epoch in range(epochs):
    model.train()

    for imgs, labels in loader_train:
        optimizer.zero_grad()
        outputs = model(imgs)
        loss = loss_fn(outputs, labels)
        loss.backward()
        optimizer.step()

    val_loss, val_acc = evaluate(model, loader_test, loss_fn)
```

notebook 的 `evaluate()` 里用了：

```python
@torch.no_grad()
def evaluate(...):
    model.eval()
```

这是考试高频点：`model.eval()` 控制 Dropout/BatchNorm，`torch.no_grad()` 控制不记录 gradient。两者不是一回事。([GitHub](https://raw.githubusercontent.com/UOS-COMP6252/public/main/lecture6/performance.ipynb "raw.githubusercontent.com"))

---

## 5.2 `transforms.ipynb`

这个 notebook 主要是可视化 transforms：

```python
RandomHorizontalFlip()
RandomAffine(degrees=0, translate=(0.3,0.3))
RandomCrop((150,150))
RandomRotation(45)
```

重点不是训练模型，而是让你理解 data augmentation 是对输入图片做随机变换，而且每次调用 transform 都可能得到不同结果。([GitHub](https://raw.githubusercontent.com/UOS-COMP6252/public/main/lecture6/transforms.ipynb "raw.githubusercontent.com"))

---

## 5.3 `data-augmentation.ipynb`

这个 notebook 更接近完整训练实验：

1. 用 `ImageFolder("flowers")` 读取 flowers dataset。
    
2. 用 `random_split(dataset, lengths=[0.8,0.2])` 分 train/valid。
    
3. 自定义 `MyDataset`，在 `__getitem__` 中对样本应用 transform。
    
4. training transform 可以选择 `TrivialAugmentWide`, `RandAugment`, `AugMix`, `AutoAugment`。
    
5. validation transform 只做 `Resize((180,180))` 和 `ToTensor()`。
    
6. CNN 输入为 `(3,180,180)`，三次 pooling 后变成 `(64,22,22)`，所以 `fc1` 输入是 `64*22*22`。
    
7. 用 TensorBoard 记录 train/valid loss 和 accuracy。([GitHub](https://raw.githubusercontent.com/UOS-COMP6252/public/main/lecture6/data-augmentation.ipynb "raw.githubusercontent.com"))
    

【代码重点】validation 不应使用 random augmentation。训练集可以随机变换，validation/test 要保持 deterministic。

---

## 6. 理论、代码、录音转写对应表

|理论概念|Handout 中的表达|Notebook 中的代码体现|录音转写中的老师强调|可能考法|
|---|---|---|---|---|
|Overfitting|train accuracy 上升，validation plateau/decrease|训练后比较 train/val loss 和 accuracy|training acc 高不代表 generalization 好|看曲线判断是否 overfitting|
|Early stopping|monitor validation loss|`EarlyStopping(patience=4)` 的设计意图|不能一次下降就停，要看 trend|写伪代码 / 找 bug|
|Dropout|随机 drop nodes，相当于 ensemble|`nn.Dropout`, `dp.train()`, `dp.eval()`|training 和 inference 行为不同|判断 dropout 输出、解释 eval|
|BatchNorm|per-channel zero mean / unit variance|`BatchNorm2d`, `mean([0,2,3])`, `var([0,2,3])`|train 用 batch stats，eval 用 running stats|手算 BN / 判断统计维度|
|Data augmentation|on-the-fly transforms|`RandomRotation`, `RandomCrop`, `TrivialAugmentWide`|改善 accuracy，不是追求绝对最优|判断 train/valid transform 是否合理|
|train/eval mode|slides 没有特别展开 API|`model.train()`, `model.eval()`|只影响 Dropout 和 BatchNorm，不控制 gradient|问 `eval()` vs `no_grad()`|

---

## 7. 必须掌握的公式 / 计算

### 7.1 Dropout

如果 drop probability 是 (p)，keep probability 是 (q=1-p)。

Training 时：

[  
\tilde{y}_l = r_l * y_l  
]

其中：

[  
r_l \sim Bernoulli(q)  
]

PyTorch training 时通常会对保留的 activation 做 scaling：

[  
\tilde{y}_l = \frac{r_l * y_l}{q}  
]

Eval 时：

[  
\tilde{y}_l = y_l  
]

也就是 identity。

---

### 7.2 BatchNorm

输入：

[  
x \in \mathbb{R}^{S \times C \times H \times W}  
]

对每个 channel：

[  
\mu_c = \frac{1}{S H W}\sum x_{ci}  
]

[  
\sigma_c^2 = \frac{1}{S H W}\sum (x_{ci}-\mu_c)^2  
]

[  
\hat{x}_{ci} = \frac{x_{ci}-\mu_c}{\sqrt{\sigma_c^2+\epsilon}}  
]

[  
y_{ci} = \gamma_c\hat{x}_{ci}+\beta_c  
]

【易错】不是对整个 batch 全部 channel 一起求 mean，而是每个 channel 单独求。

---

### 7.3 Shape 计算：data-augmentation CNN

输入：

```text
(batch, 3, 180, 180)
```

因为 Conv2d 使用 `padding='same'`，conv 不改变 H/W；MaxPool2d(2) 每次约减半：

```text
(batch, 3, 180, 180)
→ Conv1 → (batch, 16, 180, 180)
→ Pool1 → (batch, 16, 90, 90)
→ Conv2 → (batch, 32, 90, 90)
→ Pool2 → (batch, 32, 45, 45)
→ Conv3 → (batch, 64, 45, 45)
→ Pool3 → (batch, 64, 22, 22)
→ Flatten → (batch, 64*22*22)
```

所以：

```python
nn.Linear(64*22*22, 128)
```

这是 notebook 中非常适合考 shape 的地方。([GitHub](https://raw.githubusercontent.com/UOS-COMP6252/public/main/lecture6/data-augmentation.ipynb "raw.githubusercontent.com"))

---

## 8. 代码考法预测

### 题型 1：代码填空题

```python
for epoch in range(epochs):
    model.____()

    for imgs, labels in train_loader:
        optimizer.____()
        outputs = model(imgs)
        loss = loss_fn(outputs, labels)
        loss.____()
        optimizer.____()

    val_loss, val_acc = evaluate(model, valid_loader, loss_fn)
```

答案：

```python
model.train()
optimizer.zero_grad()
loss.backward()
optimizer.step()
```

---

### 题型 2：validation 函数补全

```python
@torch.no_grad()
def evaluate(model, loader, loss_fn):
    model.____()
    ...
```

答案：

```python
model.eval()
```

解释：`eval()` 改变 Dropout/BatchNorm 行为；`no_grad()` 避免记录计算图，节省显存和计算。

---

### 题型 3：bug 识别题

```python
def evaluate(model, loader):
    model.train()
    ...
```

问题：validation 时用了 `model.train()`。

后果：

- Dropout 仍然随机丢节点；
    
- BatchNorm 继续使用当前 validation batch 的 statistics；
    
- validation result 不稳定，不能代表真实 inference performance。
    

---

### 题型 4：Dropout 输出判断

```python
dp = nn.Dropout(p=0.5)
x = torch.tensor([2., 4.])
dp.train()
print(dp(x))
```

可能输出：

```text
tensor([0., 8.])
```

原因：drop probability = 0.5；保留下来的值会除以 keep probability 0.5，所以 `4 → 8`。

如果：

```python
dp.eval()
print(dp(x))
```

输出就是：

```text
tensor([2., 4.])
```

---

### 题型 5：BatchNorm 统计维度

输入：

```python
x.shape == (8, 16, 32, 32)
bn = nn.BatchNorm2d(16)
```

问：BatchNorm 对哪些维度求 mean/variance？

答案：对 `[0,2,3]`，也就是 batch、height、width；每个 channel 独立统计。所以 mean 和 variance 的 shape 是 `(16,)`。

---

### 题型 6：Data augmentation bug

```python
valid_t = Compose([
    RandAugment(),
    Resize((180,180)),
    ToTensor()
])
```

问题：validation transform 中用了 random augmentation。

正确做法：

```python
valid_t = Compose([
    Resize((180,180)),
    ToTensor()
])
```

训练集可以随机增强，validation/test 应该稳定。

---

## 9. 这节课最可能考的内容排序

### A 级：必须掌握

1. **Overfitting 判断**：train accuracy 上升，validation accuracy 停住/下降。
    
2. **Early stopping**：monitor validation loss，不要单次下降就停。
    
3. **Dropout training vs eval**：training 随机置零并 scaling，eval identity。
    
4. **BatchNorm training vs inference**：training 用 mini-batch stats，eval 用 running mean/var。
    
5. **`model.train()` vs `model.eval()` vs `torch.no_grad()`**。
    
6. **BatchNorm per-channel 计算维度**：`mean([0,2,3])`。
    

### B 级：需要理解

1. Dropout 的 ensemble intuition。
    
2. Data augmentation 的 on-the-fly transforms。
    
3. `RandomRotation`, `RandomCrop`, `RandomHorizontalFlip`, `AutoAugment/RandAugment` 的作用。
    
4. Confusion matrix 如何解释模型错误。
    

### C 级：了解即可

1. BatchNorm 为什么有效的不同理论解释，例如 internal covariate shift、length-direction decoupling。
    
2. Comet ML / TensorBoard logging 细节。
    
3. 具体 augmentation policy 的论文来源。
    

---

## 10. 考前速记版

### Must-know points

1. **Overfitting**：train acc ↑，val acc plateau/down。
    
2. **Early stopping**：看 validation loss trend，用 patience。
    
3. **Dropout**：train 随机置零；eval identity。
    
4. **PyTorch `nn.Dropout(p)`：p 是 drop probability。**
    
5. **BatchNorm train**：用当前 mini-batch mean/var。
    
6. **BatchNorm eval**：用 running mean/var。
    
7. **BatchNorm2d 对 `(S,C,H,W)` 的 `[0,2,3]` 求统计量，每个 channel 独立。**
    
8. **`model.eval()` 不关闭 gradient；`torch.no_grad()` 才是不建计算图。**
    
9. **Data augmentation 只用于 train，不要随机增强 validation/test。**
    
10. **`model.train()` 要在每个 epoch train loop 前设置；validation/test 前用 `model.eval()`。**
    

### High-risk mistakes

1. 把 `model.eval()` 误以为等于 `torch.no_grad()`。
    
2. validation 时忘记 `model.eval()`。
    
3. BatchNorm 对所有 channel 一起求 mean。
    
4. 把 PyTorch Dropout 的 `p` 当成 keep probability。
    
5. 对 validation/test 使用 random augmentation。
    

### 最可能的考试题

1. 给曲线判断 overfitting，并说明 early stopping 应该停在哪里。
    
2. 给 dropout 代码，问 train/eval 输出为什么不同。
    
3. 给 BatchNorm 输入 shape，问 mean/var 的 shape 和统计维度。
    
4. 给 training loop，填 `zero_grad`, `backward`, `step`, `train`, `eval`。
    
5. 找 bug：validation 里用了 `model.train()` 或没有 `torch.no_grad()`。
    

### 最应该背的伪代码

```python
for epoch in range(epochs):
    model.train()

    for imgs, labels in train_loader:
        optimizer.zero_grad()
        outputs = model(imgs)
        loss = loss_fn(outputs, labels)
        loss.backward()
        optimizer.step()

    model.eval()
    with torch.no_grad():
        val_loss, val_acc = evaluate(model, valid_loader)
```

这节课的核心一句话：**Improving performance 不是只调 accuracy，而是控制 generalization：用 validation 判断 overfitting，用 early stopping/dropout/BatchNorm/data augmentation 改善训练和泛化。**