# Machine Learning categories
## Supervised Learning
### example tasks
classification
Regression
## Unsupervised Learning
### example tasks
clustering
## Semi-supervised 
### description

Semi-supervised就是数据集有部分（少量）有label标注，这个方法主要的idea是先使用Unsuprvised 的method through clustering dataset，然后根据每个聚类可以用户一次label一大堆，或者直接根据每个聚类中存在的已label的数据将这个cluster中的所有instance都打上同样的label。
After labeling every instance, we can use supervised learning method.
## Reinforcement learning
### description
强化学习通过设立一个智能体，以及设定了一系列的奖惩规则。智能体通过观察环境选择和执行动作并获得奖励或者惩罚。然后他会自己学习什么是最好的策略。

## self-supervised
### description
This is totally based on unlabeled datasets. Once the algorithm labeled every instance of dataset, it can be regarded as an input of supervised learning algorithm.
![[Pasted image 20250930232735.png]]
## 系统能否从输入数据流中进行增量学习分类
### online learning
### batch learning
## classify by its generalization（泛化能力）
### based on instance
基于实例学习就是基于已经分类好的dataset，通过algorithm分析new instance 与已有的instance的相似度，在相似度图中，如果这个new instance 更接近一种category，那么就预测为这个分类
![[Pasted image 20251001141349.png]]
### based on model
这个就是训练一个模型出来，然后用model进行prediction。

## 训练过程
### 过拟合overfit
model在训练集上have good performance, constrastly,  在test set have lower performance。
Commonly, it is due to model structure is too complex  and dataset is too limited。
解决方法：通过正则化regularization and set hyperparameter.
### 欠拟合underfit
reason and methods:
model is too simple;
get more representative data(feature engineer)
adjust (reduce) regularization hyper parameter.

## 如何调整超参数（hyper parameter)
以上两个训练过程的问题都聚焦到了关于超参数的设置，由于这个超参数要在训练前设置且在训练过程中几乎不变，所以选取的策略很重要（过大容易underfit，过小容易overfit）
因此引入验证集
### 验证集validate
已知training set and test set 是为了区分（训练误差和泛化误差）training error and generalization error。
通过这两个可以初步知道model是underfit or overfit.
那么如何调整超参数？
![[Pasted image 20251001204253.png]]![[Pasted image 20251001204310.png]]
### 交叉验证_cross-validation
![[Pasted image 20251001211022.png]]
### 数据不匹配data mismatch 
当需要训练一个model，但没有针对的dataset时，可以使用相似的dataset，但这会导致data mismatch的问题（例如这里识别app拍照图片的植物种类，但是dataset是网上找的图片，即不是通过app拍的），这时候可以增加设置一个train_dev，结合dev dataset区分是overfit问题还是data mismatch问题。
![[Pasted image 20251001231204.png]]
