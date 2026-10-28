# 决策树分裂揭秘：`splitter`, `max_features` 与分割点选择

决策树的核心思想是通过一系列的“问题”将数据集不断划分，直到每个区域都足够“纯净”。这个过程中的关键在于，在树的每个节点上，如何选择最好的“问题”来进行划分。一个“问题”通常由两部分组成：**要对哪个特征提问**，以及**用什么阈值来划分**。

下面，我们将深入探讨控制这一过程的几个关键参数和算法机制。

## 1. 分割策略：`splitter` 参数

`splitter` 参数决定了在选定一个特征后，如何寻找最佳分割阈值的策略。

### `splitter="best"` (默认值)

这是最标准、最精确的策略。

*   **工作原理**：当算法决定要用某个特征进行分裂时，它会评估该特征**所有有意义的候选分割点**（详见第三部分）。对于每一个候选点，它都会计算划分后带来的“不纯度减少量”（Impurity Decrease）或“信息增益”（Information Gain）。
*   **目标**：选择那个能带来**最大**不纯度减少量的分割点，作为该特征的“最佳”分割点。
*   **特点**：
    *   **贪心算法**：在每一步都做出局部最优的选择。
    *   **计算密集**：因为要评估所有可能的分割点，计算成本相对较高。
    *   **适用模型**：`DecisionTreeClassifier` 和 `DecisionTreeRegressor` 的默认行为。

> 简单来说，`"best"` 策略会尽力在每一步都找到最优的分割方案。

### `splitter="random"`

这个策略引入了更多的随机性，通常用于构建极端随机树（Extra-Trees）。

*   **工作原理**：算法不再 exhaustive 地搜索最佳分割点。相反，它会在该特征的取值范围内**随机生成一个分割点**。然后，它会从多个随机生成的分割规则中，选择表现最好的那一个。
*   **目标**：在随机生成的候选项中找到最好的一个。
*   **特点**：
    *   **随机性更强**：树的结构会因为这种随机性而变得更多样化。
    *   **计算更快**：避免了寻找最优分割点的昂贵计算。
    *   **减少过拟合**：增加的随机性可以降低模型方差，在集成模型中使用时有助于防止过拟合。
    *   **适用模型**：`ExtraTreeClassifier` 和 `ExtraTreeRegressor` 的默认行为。

> 简单来说，`"random"` 策略是在一堆随机的分割方案里“矮子里面拔将军”。

### 对比总结

| 特性 | `splitter="best"` | `splitter="random"` |
| :--- | :--- | :--- |
| **策略** | 在所有可能的分裂中寻找最优解 | 在随机生成的分裂中寻找最优解 |
| **计算成本** | 较高 | 较低 |
| **随机性** | 较弱 | 较强（分割点也是随机的） |
| **模型方差** | 较高 | 较低 |
| **默认使用者** | `DecisionTreeClassifier` | `ExtraTreeClassifier` |

---

## 2. 特征选择：`max_features` 参数

在寻找分割点之前，算法首先要决定**考虑哪些特征**。这个过程由 `max_features` 参数控制。

*   **最根本的来源**：特征就是你提供给模型的训练数据 `X` 的所有列。
*   **算法的选择**：在分裂节点时，算法不一定会考察全部特征。

`max_features` 参数决定了在每个节点寻找最佳分裂时，要从总特征中随机选择多少个特征来参与评估。

*   **`max_features=None` (默认)**: 考虑所有特征。算法会为每个特征计算最佳分割，然后选择全局最好的那个。
*   **`max_features="sqrt"`**: 随机选择 `sqrt(总特征数)` 个特征。这是随机森林分类器的常用默认值。
*   **`max_features="log2"`**: 随机选择 `log2(总特征数)` 个特征。
*   **`int` 或 `float`**: 指定具体的数量或比例。


**为什么要限制特征数量？**
1.  **加快训练速度**：评估的特征少了，计算自然就快了。
2.  **增加模型多样性，防止过拟合**：这是更重要的原因，尤其在随机森林等集成模型中。通过限制特征，可以确保森林中的每棵树都是从不同角度学习数据，使得树之间差异更大，集成后的模型泛化能力更强。

---

## 3. 分割点的确定：“所有可能的分割点”

当 `splitter="best"` 时，算法如何高效地找到“所有可能的分割点”呢？它并非测试无限个数值，而是采用了一个聪明的有限集策略。

以下是针对一个**连续型特征**（如“年龄”）寻找最佳分割点的具体步骤：

1.  **收集特征值**：
    首先，算法收集当前节点上所有样本在该特征上的具体数值。
    *例如，一个节点上有6个样本，它们的“年龄”分别是 `[20, 22, 25, 30, 30, 35]`。*

2.  **排序并去重**：
    将这些值进行排序并移除重复值，得到一个唯一的、有序的列表。
    *排序去重后得到：`[20, 22, 25, 30, 35]`。*

3.  **生成候选分割点**：
    算法只在相邻的两个唯一值之间生成候选分割点，通常是取它们的中点。因为在两个相邻值之间的任意点进行分割，其划分结果都是完全一样的。
    *候选分割点为：*
    *   `(20 + 22) / 2 = 21`
    *   `(22 + 25) / 2 = 23.5`
    *   `(25 + 30) / 2 = 27.5`
    *   `(30 + 35) / 2 = 32.5`
    *所以，候选分割点集合是 `{21, 23.5, 27.5, 32.5}`。*

4.  **评估所有候选点**：
    算法遍历这几个候选点，用每个点作为阈值去划分数据，并计算其带来的不纯度降低量。

5.  **选择最佳分割点**：
    选择那个能带来最大不纯度降低的候选点，作为该特征的最佳分割点。

---

## 4. 总结：一个节点的完整分裂流程

现在，我们可以将所有部分串联起来，看看决策树在一个节点上进行分裂的完整流程：

1.  **选择特征子集**：
    根据 `max_features` 参数，从所有特征中随机选择一个子集进行评估。

2.  **遍历特征子集**：
    对于子集中的**每一个特征**：
    a. **寻找最佳分割点**：根据 `splitter` 的策略（`"best"` 或 `"random"`）和上述的分割点确定方法，找到这个特征的最佳分割点。
    b. **计算收益**：计算出这个 `(特征, 分割点)` 组合能带来的不纯度降低量。

3.  **最终决策**：
    比较所有候选特征找到的最佳分裂规则，选择那个能带来**全局最大不纯度降低**的规则，用它来正式分裂当前节点，生成两个子节点。

这个过程在每个新生成的节点上递归地重复进行，直到满足停止条件（如达到 `max_depth`、节点样本数小于 `min_samples_split` 等），决策树的生长就完成了。



# 机器学习实验课笔记：决策树、随机森林与集成学习 (Machine Learning Lab Notes: Decision Trees, Random Forests & Ensemble Learning)

### 一、 实验环境与准备 (Environment Setup & Preparation)

- **环境信息 (`conda info`)**: 用于显示当前 Conda 环境配置，包括 Python 版本、库路径等，确保实验环境一致性。
    
- **库导入与设置 (`import ...`)**:
    
    - 导入核心库：NumPy (`np`) 用于数值计算，Pandas (`pd`) 用于数据处理（本次实验涉及较少），Scikit-learn (`sk`) 用于机器学习任务。
        
    - 导入绘图库：Matplotlib (`mpl`, `plt`) 用于数据可视化。
        
    - `%matplotlib inline`: Jupyter魔法命令，使图形直接在Notebook中显示。
        
    - 设置图像保存路径和 `save_fig` 辅助函数，方便保存实验结果图。
        
- **安装库 (`!pip install ...`)**:
    
    - `!pip install scikit-learn`: 安装或更新 Scikit-learn 机器学习库。
        
    - `!pip install graphviz`: 安装 Graphviz 的 Python 接口库，用于可视化决策树（通常还需要系统级安装 Graphviz）。
        

### 二、 决策树 (Decision Trees)

#### 1. 基本概念与训练 (Basic Concept & Training)

- **数据集**: 使用经典的鸢尾花数据集 (Iris dataset)。
    
- **特征选择**: 选取花瓣长度 (petal length) 和花瓣宽度 (petal width) 作为特征 `X`，方便二维可视化。`y` 为目标类别标签。
    
- **模型训练 (`DecisionTreeClassifier`)**:
    
    - 使用 Scikit-learn 的 `DecisionTreeClassifier` 类构建分类模型。
        
    - `max_depth=2`: 设置树的最大深度为 2，作为**正则化 (regularization)** 手段防止过拟合 (overfitting)。
        
    - `random_state=42`: 设置随机种子以保证结果的可复现性 (reproducibility)。
        
    - `tree_clf.fit(X, y)`: 使用训练数据训练模型。
        
- **CART 算法**: Scikit-learn 使用 **CART (Classification and Regression Tree)** 算法训练决策树。
    
    - **贪心算法 (Greedy Algorithm)**: CART 在每个节点选择**当前步骤**最优的分裂特征和阈值（最大化不纯度降低），而不考虑全局最优。这使得算法高效，但不保证找到绝对最优的树。
        

#### 2. 可视化 (Visualization)

- **`export_graphviz`**: 将训练好的决策树导出为 `.dot` 文件格式。
    
    - 参数 `feature_names`, `class_names` 用于在图中显示名称。
        
    - 参数 `rounded`, `filled` 控制节点样式。
        
- **.dot 文件**: 可以使用 Graphviz 工具渲染成树状图。
    
- **决策边界 (`plot_decision_boundary` 函数)**:
    
    - 该函数通过在特征空间中创建网格 (`np.meshgrid`)，对网格上的每个点进行预测，并用颜色填充 (`plt.contourf`) 来展示模型学到的分类区域。
        
    - `X_new = np.c_[x1.ravel(), x2.ravel()]`: 这行代码是 `plot_decision_boundary` 函数内部的关键步骤。
        
        - `x1`, `x2` 是 `meshgrid` 的输出，包含了网格所有点的x和y坐标矩阵。
            
        - `.ravel()` 将坐标矩阵**展平 (flatten)** 成一维数组。
            
        - `np.c_[]` 将展平后的x坐标数组和y坐标数组**按列合并 (concatenate column-wise)**，形成一个 `(N_points, 2)` 形状的数组，其中每一行代表网格上一个点的 `(x, y)` 坐标，符合 Scikit-learn `predict` 方法的输入要求。
            
    - 在图上绘制实际的分裂阈值线 (`plt.plot`) 可以更清晰地理解边界的形成。
        

#### 3. 预测与节点信息 (Prediction & Node Information)

- **预测过程**: 从根节点 (root node) 开始，根据样本特征值与节点分裂阈值的比较，向下遍历，直至到达叶节点 (leaf node)，叶节点的预测即为最终结果。
    
- **节点属性**:
    
    - `samples`: 落入该节点的训练样本数。
        
    - `value`: 落入该节点的各类别样本数。
        
    - `gini`: **基尼不纯度 (Gini impurity)**，衡量节点纯度。 $G_i = 1 - \sum_{k=1}^{n} p_{i,k}^2$，其中 $p_{i,k}$ 是节点 $i$ 中类别 $k$ 样本的比例。Gini=0 表示节点完全纯净。CART 分裂时选择能最大程度降低子节点（加权平均）基尼不纯度的分裂点。
        
- **概率预测 (`predict_proba`)**: 返回样本属于每个类别的概率，等于其最终落入的叶节点中各类别样本的比例。
    
- **类别预测 (`predict`)**: 返回具有最高概率的类别标签。
    

#### 4. 正则化 (Regularization)

- **目的**: 防止决策树过拟合训练数据。
    
- **方法**: 限制树的复杂度。
    
- **超参数 (Hyperparameters)**:
    
    - `max_depth`: 最大深度。
        
    - `min_samples_split`: 节点可分裂的最小样本数。
        
    - `min_samples_leaf`: 叶节点必须包含的最小样本数。（增大 `min_*` 参数可增强正则化）
        
    - `max_leaf_nodes`: 最大叶节点数。（减小 `max_*` 参数可增强正则化）
        
    - `max_features`: 每次分裂时考虑的最大特征数。
        
- **效果**: 正则化后的模型决策边界通常更平滑，泛化能力 (generalization ability) 可能更好。
    

#### 5. 回归树 (Regression Trees)

- **应用**: 预测连续数值。
    
- **`DecisionTreeRegressor`**: Scikit-learn 中的回归树类。
    
- **区别**:
    
    - 分裂标准：通常最小化**均方误差 (Mean Squared Error, MSE)** 而不是不纯度。
        
    - 叶节点预测：预测值为落入该叶节点的所有训练样本目标值的**平均值 (average)**。
        
- **可视化**: 结构图与分类树类似，但节点显示 MSE 和预测的平均值 `value`。预测结果图呈**阶梯状 (step-function)**。
    
- **正则化**: 同样需要正则化（如 `max_depth`, `min_samples_leaf`）来防止过拟合。
    

### 三、 集成学习 (Ensemble Learning) 与 随机森林 (Random Forests)

#### 1. 核心思想

组合多个预测器（学习器）来获得比单个预测器更好的性能。

#### 2. 投票分类器 (Voting Classifiers)

- **机制**: 聚合多个不同分类器的预测结果。
    
- **硬投票 (Hard Voting)**: 基于预测类别标签进行简单多数投票。每个分类器权重相同。
    
- **软投票 (Soft Voting)**: 基于预测概率进行平均，选择平均概率最高的类别。
    
    - **优势**: 通常性能优于硬投票，因为它**隐式地给予了置信度高（预测概率极端）的分类器更大的权重**，更好地利用了各分类器的信息。
        
    - **要求**: 所有基分类器必须能预测概率（`predict_proba` 方法）。SVC 需设置 `probability=True`。
        

#### 3. Bagging 与 Pasting

- **机制**: 让多个（通常是同类型的）基学习器在训练集的不同随机子集上训练。
    
- **Bagging (Bootstrap Aggregating, 装袋法)**: 通过**有放回抽样 (sampling with replacement)** 创建训练子集。允许样本重复，引入更多样性。
    
- **Pasting (粘贴法)**: 通过**无放回抽样 (sampling without replacement)** 创建训练子集。样本不重复。
    
- **Scikit-learn 实现 (`BaggingClassifier`)**:
    
    - `bootstrap=True` (默认): Bagging。
        
    - `bootstrap=False`: Pasting。
        
- **比较**: Bagging 通常能更好地**降低方差 (variance)**（因为树之间差异更大），代价是可能略微增加**偏差 (bias)**。总体上 Bagging 更常用且效果往往更好。
    
- **交叉验证 (Cross-Validation)**: 可用于**比较 Bagging 和 Pasting** 在特定数据集上的性能，选择更优的方法。
    

#### 4. 随机森林 (Random Forests)

- **定义**: 决策树的集成，通常使用 Bagging 训练。
    
- **与 Bagging (DT) 的关键区别**: 除了 Bagging 的样本随机性外，随机森林在**构建树的每个节点**进行分裂时，还会从**随机选择的特征子集 (random subset of features)** 中寻找最佳分裂点（而不是考虑所有特征）。
    
- **效果**: 这种双重随机性（样本随机 + 特征随机）进一步降低了树之间的相关性，通常能带来更低的方差和更好的泛化能力。
    
- **Scikit-learn 实现 (`RandomForestClassifier`, `RandomForestRegressor`)**: 提供了优化的高效实现。
    
- **代码比较**: 实验代码通过对比 `RandomForestClassifier` 和一个手动配置了 `DecisionTreeClassifier(max_features="sqrt")` 的 `BaggingClassifier`，验证了两者行为的相似性。
    

#### 5. 特征重要性 (Feature Importance) - 随机森林

- **计算原理**: 通过衡量一个特征在森林中所有树上，平均能带来多少**（加权的）不纯度降低 (weighted impurity reduction)** 来确定。分裂影响的样本越多，权重越大。
    
- **Scikit-learn**: 训练后通过模型的 `feature_importances_` 属性获取，分数归一化，和为 1。
    

#### 6. 提升方法 (Boosting)

- **机制**: **顺序 (sequentially)** 训练一系列学习器，每个学习器都试图纠正前一个学习器的错误。目标是将弱学习器组合成强学习器。
    

#### 7. AdaBoost (Adaptive Boosting, 自适应增强)

- **工作方式**:
    
    1. 初始化样本权重（均等）。
        
    2. 迭代训练弱学习器（如决策树桩 decision stumps）。
        
    3. 计算当前学习器的**加权错误率 (weighted error rate)** `r`。
        
    4. 计算该学习器的权重 `alpha`（错误率越低，权重越高）。
        
    5. **增加**被**错误分类**样本的权重，**降低**被正确分类样本的权重。
        
    6. 归一化样本权重。
        
    7. 最终预测是所有弱学习器的**加权投票**。
        
- **学习率 (Learning Rate)**: 控制样本权重更新的幅度 (`alpha` 的大小)。
    
- **代码模拟**: 实验中的代码模拟了这个过程：
    
    - `sample_weights[y_pred != y_train]`: 通过**布尔索引 (boolean indexing)** 获取**被错分样本的当前权重**。该操作返回一个新数组，其**值**是错分样本的权重，其**下标**是该新数组中的位置（从0开始），**不直接等于原始样本编号**。
        
    - `sample_weights[y_pred != y_train] *= np.exp(alpha)`: 增加错分样本的权重。
        

#### 8. 梯度提升 (Gradient Boosting, GBRT)

- **机制**: 也是顺序训练。与 AdaBoost 不同，每个新的学习器拟合的是前一个学习器预测结果与真实值之间的**残差错误 (residual errors)**。
    
- **Scikit-learn 实现 (`GradientBoostingRegressor`, `GradientBoostingClassifier`)**:
    
    - `n_estimators`: 树的数量。
        
    - `learning_rate`: 学习率（收缩率 Shrinkage），控制每棵树的贡献大小。低学习率通常需要更多树，是一种正则化手段。
        

#### 9. 早停法 (Early Stopping) - 用于梯度提升

- **目的**: 找到最优的 `n_estimators`（树的数量），防止过拟合。
    
- **方法**: 在训练过程中监控模型在**验证集 (validation set)** 上的性能。当性能不再提升或开始下降时停止训练。
    
- **`staged_predict(X)` 函数**:
    
    - 返回一个**迭代器 (iterator)**，依次产生集成包含 1 棵树、2 棵树... 直到 `n_estimators` 棵树时的预测结果。
        
    - 使得可以在不重新训练多个模型的情况下，高效评估不同集成规模的性能。
        
- **代码实现**:
    
    1. 划分训练集和验证集。
        
    2. 训练一个较多树的 GBRT 模型。
        
    3. 使用 `staged_predict` 在验证集上获取各阶段预测，计算误差 `errors`。
        
    4. 找到误差最小时的树数量 `bst_n_estimators = np.argmin(errors) + 1`。
        
    5. 使用 `bst_n_estimators` 重新在完整训练集上训练最终模型 `gbrt_best`。
        

#### 10. XGBoost (Extreme Gradient Boosting)

- 一个**优化**的梯度提升库，通常比 Scikit-learn 实现更快、功能更强（如更好的正则化、缺失值处理）。
    
- 提供 Scikit-learn 风格 API (`XGBClassifier`, `XGBRegressor`)。
    

### 四、 Python/NumPy 语法细节

- **`numpy.meshgrid(*xi, indexing='xy', sparse=False)`**:
    
    - 从一维坐标向量 `xi` 创建 N 维坐标网格矩阵。
        
    - `indexing='xy'` (默认): 笛卡尔索引，适用于绘图，二维输出形状 `(N, M)`。
        
    - `indexing='ij'`: 矩阵索引，符合数组索引习惯，二维输出形状 `(M, N)`。
        
    - `sparse=True`: 生成稀疏网格，节省内存，配合广播使用。
        
- **`.ravel()`**: 将 NumPy 多维数组**展平 (flatten)** 为一维数组。
    
- **`numpy.c_[arr1, arr2, ...]`**: 将多个数组**按列 (column-wise)** 连接/堆叠成一个新的二维数组。常用于将 `meshgrid` 的展平输出组合成 `(n_samples, n_features)` 的格式。
    
- **布尔索引 (Boolean Indexing)**: 使用一个布尔数组（True/False 掩码）作为索引来选择 NumPy 数组中的元素。`array[boolean_mask]` 会返回 `array` 中对应 `boolean_mask` 为 `True` 位置的元素组成的新数组。
    

### 五、 回答 Recap 中的问题

1. **构建最优决策树的难度**: NP 难问题，因为需要搜索的可能树结构数量呈指数级增长。
    
2. **CART 为何是贪心算法**: 因为它在每个节点只选择当前能最大程度降低不纯度的分裂，而不考虑这个选择对全局最优性的长期影响，也不会回溯。
    
3. **Extra-Trees vs. 随机森林**: 主要区别在于 Extra-Trees 在选择分裂阈值时是随机的（而不是最优的），并且通常不使用自助采样法（bootstrap）。这使得 Extra-Trees 随机性更强，方差通常更低，偏差略高，训练更快。
    
4. **AdaBoost 工作原理**: 顺序训练弱学习器，迭代地增加被前一个学习器错分样本的权重，最终通过加权投票/平均进行预测。
    
5. **随机森林学习特征重要性**: 通过计算特征在所有树中平均带来的（加权的）不纯度降低量。
    
6. **堆叠 (Stacking)**: 一种集成方法，训练一个元学习器（meta-learner）来组合多个不同基学习器（base learners）的预测结果。训练元学习器时通常使用交叉验证机制生成输入特征。
    
7. **使用 XGBoost 实现梯度提升**: XGBoost 是梯度提升的高效优化库，提供 Scikit-learn 风格 API (`XGBClassifier`/`XGBRegressor`)，具有速度快、正则化强、可处理缺失值等优点。
    
