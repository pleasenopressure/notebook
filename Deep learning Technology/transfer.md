你好！我是你的课程助教。很高兴能协助你梳理南安普顿大学 ECS 学院 COMP6252 (Deep Learning Technologies) 课程关于“迁移学习 (Transfer Learning)”的内容 。

---
## 📚 课程概览 (Course Overview)

本节课深入探讨了**迁移学习 (Transfer Learning)** 的核心概念与工程实践 。与其从零开始训练深度神经网络（耗时且需要海量数据），不如利用已在大型数据集（如 ImageNet）上预训练好的模型（如 AlexNet, ResNet18），提取其学到的通用图像特征 。通过替换模型的最后一层（分类头）并进行微调 (Fine-tuning)，我们能在极少的 Epoch 内，在自定义数据集（如 5 类花卉图像）上实现极高的准确率 。课程还详细演示了如何在 Kaggle 环境下使用 PyTorch、`timm` 库进行数据加载、数据增强以及模型权重的冻结与保存 。

---

## 🚨 教师重点复述 (Teacher's Emphases)

老师在课堂上反复强调了以下几个工程实践中的“避坑指南”，请务必牢记：

- **绝对不要对验证集/测试集进行数据增强 (Data Augmentation)**：
    
    - **背景解释**：有同学提问是否为了增加测试数据量而对验证集做增强 。
    - **老师强调**：数据增强（如旋转、平移、添加噪声）仅限于**训练阶段**使用，目的是增加网络的泛化能力，让网络学会忽略噪声并理解图像的真正语义 。
    - **常见错误**：如果在验证集上做增强，会混淆模型，导致评估出的准确率异常降低，因为验证集本应反映模型在未见过的新鲜且真实的数据上的表现 。
- **保存模型时，仅保存 `state_dict` (权重字典)**：
    
    - **背景解释**：PyTorch 允许保存整个模型结构+权重，或者仅保存权重数字 。
        
    - **老师强调**：强烈建议**不要**保存整个模型架构，因为这会与具体的硬件/环境绑定（如跨 CPU/GPU 加载时容易报错） 。
        
    - **正确做法**：应该使用 `model.state_dict()` 仅保存序列化后的权重参数 。加载时，需先实例化模型代码，再调用 `load_state_dict()` 加载权重 。
        
- **为何要用自定义 Dataset 包装数据拆分**：
    
    - **背景解释**：PyTorch 提供了 `random_split` 用于划分训练集和验证集（如 80/20） 。
    - **老师强调**：`random_split` 无法让你为训练集和验证集传入**不同的** transforms（预处理流水线） 。因此，必须编写继承自 `torch.utils.data.Dataset` 的自定义类来包裹数据，以便针对训练集应用数据增强（如 `TrivialAugment`），而对验证集保持纯净 。
        
---

## 📖 按章节/逻辑详细讲解

#### 1. 迁移学习的直观理解与原理 (Slides 1-8, Transcript 0:00-7:00)

- **核心理念**：迁移学习是使用在不同数据集上预训练过的模型 。在倒数第二层之前，网络学习到的是适用于所有图像的“高级”特征（High-level features） 。
    
- **替换最后一层 (Replacing the Head)**：
    
    - 绝大多数视觉模型（如 ResNet, AlexNet）是在 ImageNet 竞赛数据集上训练的，最后一层输出 1000 个值（代表 1000 个类别） 。 * 对于我们的花卉分类任务（5 个类别），需要将最后一层的 1000 个输出替换为 5 个输出的全新全连接层 (`fc, 5`) 。
- **性能对比（为什么要迁移？）**：
    
    - 课件展示了 ResNet18 的验证集准确率曲线图：从零开始随机初始化权重（橙色线）在 90 个 Epoch 后仅达到约 0.75 左右的准确率 。
    - 使用预训练模型（绿色线）则迅速攀升并在极短时间内稳定在 0.9 以上的准确率 。这省去了数百个 Epoch 的算力消耗 。
        
- **三种微调策略 (Fine-tuning Approaches)**：
    
    - **策略 1**：冻结 (Freeze) 所有中间层的权重，仅训练你替换的最后一层 。
        
    - **策略 2**：训练所有层（起点是预训练权重，而非随机权重） 。课件图表显示，在此特定数据集中，训练所有层（橙色线，~0.94）比仅训练最后一层（绿色线，~0.90）的准确率更高 。
    - **策略 3**：混合方法，先训练最后一层，然后“解冻”中间层进行全面微调 。具体采用哪种方法高度依赖于你的目标数据集 。

#### 2. 工程实现与 PyTorch 技巧 (Transcript 7:00-34:00)

- **获取预训练模型的途径**：
    
    - 老师提到了可以使用 `torchvision.models`、`timm` (PyTorch Image Models) 库，或者 Hugging Face 。
        
    - `timm` 库只需传入字符串名称（如包含 'resnet18'）且自带快捷的方法，而 `torchvision` 需要显式调用对应的类 。* **处理数据集 (`ImageFolder`)**：
        
    - 利用 `torchvision.datasets.ImageFolder` 可以极大地简化分类数据的加载 。
        
    - 只要你的文件夹按照 `parent_folder/class_name/images...` 结构组织（例如 `flowers/daisy/`, `flowers/rose/`），`ImageFolder` 就会自动推断出类别和标签 。
    * **代码层面的“冻结”与“替换”操作**：
    - **冻结参数**：通过遍历模型的参数（`parameters()`），并将它们的 `requires_grad` 属性设置为 `False`，使得优化器在反向传播时不更新这些权重 。
    
    - **替换层**：在 `timm` 中可以方便地使用 `reset_classifier()` 方法自动创建新的分类头 。但在 `torchvision` 中，需要了解模型架构，手动将最后一层（通常叫 `.fc` 或 `.classifier`）赋值为一个新的 `nn.Linear` 模块 。默认情况下，新创建的层 `requires_grad` 为 `True`，即可被训练 。
        

---

## 📝 专有名词与公式解释 (Glossary)

- **Transfer Learning (迁移学习)**:
    
    - _定义_: 一种机器学习技术，将模型在一个任务上学到的知识（通常是海量数据）应用到另一个相关任务上 。
        
    - _直观理解_: 就像一个学会了骑自行车的人，学骑摩托车会比完全没有平衡感的新手快得多。网络前几层提取边缘、轮廓等通用特征，这些特征在大部分视觉任务中都是通用的 。
        
        +1
        
- **Fine-tuning (微调)**:
    
    - _定义_: 在迁移学习中，使用目标任务的数据对预训练模型的参数进行小幅度的调整训练 。
        
        
- **`requires_grad`**:
    
    - _定义_: PyTorch 中 Tensor 的一个布尔属性。设置为 `False` 表示冻结该张量，使其在反向传播时不计算梯度，即权重不更新 。
        
- **`ImageFolder`**:
    
    - _定义_: PyTorch 提供的一个便捷数据集类。它假设所有的图像都按照类别存放在各自的子文件夹中，并自动将文件夹名称映射为类别索引 。
        
        +1
        
- **`state_dict` (状态字典)**:
    
    - _定义_: PyTorch 中一个 Python 字典对象，将每一层映射到其对应的参数张量（权重和偏置） 。它是模型保存和加载的最佳实践 。
        
        +1
        

---

## 💡 配套例题与思考题 (Practice Questions)

1. **概念题**：如果你的目标数据集与预训练模型的数据集（如 ImageNet）差异极大（例如：你想要识别医学显微镜下的细胞，而 ImageNet 主要是生活照片），你认为“冻结中间层只训练最后一层”的策略还奏效吗？为什么？
    
    - _(提示：回忆高级特征的适用性。可能需要解冻更多层甚至采用混合策略。)_
        
2. **代码实现题**：假设你加载了一个 `torchvision.models.resnet50(pretrained=True)` 模型。请写出两行核心伪代码，演示如何冻结该模型的所有参数，并将输出类别数修改为 10 类。
    
    - _(提示：遍历 `.parameters()`，修改 `.fc`)_
        
3. **除错题**：一位同学发现自己的模型在训练集上准确率稳步上升，但在验证集上准确率停滞不前甚至下降。他检查代码后发现自己对验证集应用了 `RandomHorizontalFlip` 和 `ColorJitter`。请根据老师的讲解解释其原因。
    

---

## ⚠️ 考前避坑警示

- **测试集/验证集绝对不能做数据增强** (只能 Resize 和 Normalize)，否则会导致评估指标完全失效！
    
- 保存模型结构 `.save(model)` 会带来硬件绑定问题，永远只保存 `state_dict`。
    
- 如果使用 `random_split`，需留意其无法对两部分应用不同的 Transform 流水线，建议自定义 Dataset 类进行包裹包装。
    
## 🚨 教师重点代码复述 (Teacher's Emphases in Code)

在之前的转写中老师反复强调的几个“避坑指南”，在代码中都有极其精彩的实现，请务必重点复习：

#### 1. 为什么一定要自己写一个 `MyDataset` 类？ (对应转写 16:51)

- **痛点**：通常我们用 `random_split` 将数据按 8:2 划分为训练集和验证集。但 PyTorch 原生的 split 出来的对象只是一个 Subset，它**无法让你对训练集和验证集应用不同的 Transform（预处理变换）**。
    
- **代码实现**：老师写了一个自定义类，继承自 `Dataset`，它接收一个 Subset 和一个专属的 Transform。
    
    Python
    
    ```
    class MyDataset(Dataset):
        def __init__(self, subset, transform=None):
            self.subset = subset
            self.transform = transform
        def __getitem__(self, idx):
            x, y = self.subset[idx]
            if self.transform:
                x = self.transform(x) # 在取数据时，动态应用传入的独立 transform
            return x, y
    ```
    
- **应用**：借此，我们可以给 Train 传入含有 `TrivialAugmentWide()` 的流水线，给 Val 传入只有 Crop 和 Normalize 的纯净流水线。**完美贯彻了“绝不对验证集做数据增强”的铁律！**
    

#### 2. “仅微调最后一层” 的代码开关 (对应转写 29:02)

- **实现方式**：代码中定义了一个全局变量 `last_layer_only = False`。如果是 `True`，则通过遍历 `.parameters()` 并设置 `requires_grad = False` 来实现**冻结 (Freezing)**。
    
    Python
    
    ```
    if last_layer_only:
        for p in model.parameters():
            p.requires_grad = False
    ```
    
    _(助教提示：由于新创建的分类头默认 `requires_grad=True`，所以这样操作后，优化器就只会更新最后一层了。)_
    

#### 3. 正确保存模型：永远只保存 `state_dict` (对应转写 33:27)

- **代码示范**：Notebook 最后的 Cell 给出了标准答案。
    
    Python
    
    ```
    # 存
    torch.save(model.state_dict(), "last-only-weights")
    # 取（切记先实例化同结构模型）
    model.load_state_dict(torch.load("last-only-weights"))
    ```
    

---

## 📖 核心代码逻辑拆解 (Step-by-Step Code Analysis)

#### Step 1: 图像尺寸为何是 224？

- **原理解释**：在 `data_transforms` 字典中，无论训练还是验证，都有一步 `transforms.CenterCrop(224)`。
    
- **为什么**：因为绝大多数在 ImageNet 上预训练的经典卷积神经网络（如 ResNet, VGG 等）的输入张量期望尺寸都是 $224 \times 224$。这是迁移学习的数据对齐基准。
    

#### Step 2: 加载预训练模型的三大门派

代码展示了在 PyTorch 生态中获取模型的 3 种主流方式，通过全局变量 `WHERE_TO_GET_MODEL` 控制：

1. **`timm` (PyTorch Image Models)**：
    
    - _特点_：极其强大的第三方库，接口统一。
        
    - _代码_：`model = timm.create_model('resnet18', pretrained=True)`
        
    - _换头_：极其优雅，只需一句 `model.reset_classifier(len(class_names))` 即可自动丢弃旧头并创建适应新类别数的新头。
        
2. **`torchvision.models`**：
    
    - _特点_：PyTorch 官方自带，最常用。
        
    - _代码_：`model = torchvision.models.resnet18(weights=torchvision.models.ResNet18_Weights.DEFAULT)`
        
    - _换头_：需手动探除原特征维度并替换。`model.fc = nn.Linear(model.fc.in_features, len(class_names))`
        
3. **`torch.hub`**：
    
    - _特点_：可直接从 Github 仓库（如 NVIDIA 的优化版或 HuggingFace）拉取非标准模型。
        

#### Step 3: 训练与验证循环 (`train_model` 函数)

- **评估函数 (`evaluate`)**：使用 `torch.no_grad()` 包装，强制设为 `model.eval()`（关闭 Dropout 和 BatchNorm 的训练特性），遍历计算验证集 Loss 和 Accuracy。
    
- **保存最优模型**：在训练 Epoch 循环中，不仅打印指标，还通过 `copy.deepcopy(model.state_dict())` **动态在内存中备份**当前验证集准确率最高时的权重，并在训练结束后返回这个最优模型。
    
- **工程利器**：引入了 `comet_ml`。这是一个实验记录工具，`experiment.log_metrics()` 可以将训练曲线实时同步到云端看板，方便分析。
    

---

## 📝 深度学习工程专有名词解释 (Engineering Glossary)

- **`Subset`**: PyTorch 中 `random_split` 函数返回的对象类型。它不包含实际数据，只包含原数据集的引用和被选中样本的索引列表。
    
- **`timm`**: 由 Ross Wightman 维护的开源库，包含了几乎所有最新的 SOTA 视觉模型实现和预训练权重，是目前 CV 竞赛和工程的标配。
    
- **`copy.deepcopy()`**: Python 标准库中的深拷贝。在训练中，如果直接赋值 `best_weights = model.state_dict()`，由于字典传递的是引用，最优权重会被后续 Epoch 覆盖。必须使用深拷贝才能真正保存下当时的权重矩阵。
    
- **`lr_scheduler.StepLR`**: 学习率衰减器。代码中 `StepLR(optimizer, step_size=30, gamma=0.5)` 意味着每训练 30 个 epoch，学习率乘以 0.5。这有助于模型在训练后期更好地收敛到极小值。
    

---

## 💡 实验思考题 (Code Challenge)

1. **观察与思考**：在 `train_model` 函数内部，每次进入训练 batch 循环前都要调用一次 `model.train()`，并在 `evaluate` 中调用 `model.eval()`。如果我们忘记写这两行代码，对 ResNet 这类包含 BatchNorm 层的模型会有什么致命影响？
    
2. **代码修改**：目前代码通过 `len(class_names)` 自动推断输出类别数（5类）。如果我们只想做一个简单的“二分类”模型（例如只识别 Tulip 和非 Tulip），在不修改数据集文件夹结构的前提下，你应该如何在 `__getitem__` 或训练循环中修改标签 `y` 的逻辑？
    
3. **超参分析**：老师在优化器中使用了 `optim.SGD(..., lr=0.001, momentum=0.9)`。结合我们使用的是“预训练模型”，为什么这里的初始学习率 (lr=0.001) 通常比从零训练 (例如 lr=0.1) 设置得要小得多？
    

_(助教提示：针对第三题，迁移学习的本质是“微调”，如果学习率太大，会破坏预训练模型原本已经学到的优秀特征提取能力，导致“灾难性遗忘”。)_
