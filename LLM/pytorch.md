# 常用函数介绍
```
### 这可以看到module/class中的子module或者方法和属性
dir（module）
### 可以展示函数的官方注解
help(method)
### 地址拼接param,param
import os
os.path.join(param,param)
### 字符串绑定
str = "***{}".format(param)
### 创建地址列表
os.listdir(file_path)

```
# 可视化函数
tensorboard的中的add_scalar（）
参数：
- **`tag`** (str):
	    就是图标标题
    - 数据的名称，通常是一个字符串，用于标识你记录的标量。例如：`"train/loss"` 或 `"validation/accuracy"`。这个 tag 会作为图表的标签显示。
        
- **`scalar_value`** (float or int):
	    就是纵坐标值
    - 你要记录的标量值。通常是在训练过程中计算得到的损失（loss）值、准确率（accuracy）值等。
        
- **`global_step`** (int, optional):
	    横坐标值
    - 记录这个标量值时的步数（或 epoch）。这个步骤数会在 TensorBoard 中显示，用于绘制标量的变化趋势。如果没有指定，默认值为 `None`，这意味着不会记录步数（不建议省略）。
# transform类

# 一些函数设计细节
当一个类（class）实现了一个对象（object），可以通过object（）的方法直接调用这个object的__call__()方法。
此外torch.nn.Module类中，pytorch希望将模型逻辑的定义和实例化对象的调用行为分离
具体：[[__call__() and forward()]]
## torch.tensor()
这里的dtype可以规定里面数据的类型
![[Pasted image 20250915110247.png]]

# torchvision
这里的torchvision.datasets中有常用的图像数据集，如图
![[Pasted image 20250913114902.png]]
这里如果下载太慢了，可以直接进入数据集的定义中找到url进行迅雷这种工具下载。
# Dataloader
多看官方文档
https://docs.pytorch.org/docs/stable/data.html#torch.utils.data.DataLoader
# 如何更改模型结构
![[Pasted image 20250916150515.png]]如图，可以通过
```
### vgg是模型对象
vgg.classifier.add_module("新层的名字"，具体层的对象创建函数)
### 或者修改模型中的第x层
vgg.classifier[x] = 具体层的对象创建函数
```
# 模型的保存和加载
这里的
![[Pasted image 20250916151511.png]]
![[Pasted image 20250916151527.png]]
#grad #requires_grad #grad_fn
1、requires_grad
requires_grad是pytorch中tensor的一个属性，如果requires_grad=True，在进行反向传播的时候会记录t该tensor梯度信息。为了下面方便举例，我将进行运算的tensor称为父tensor，运算后的tensor称为子tensor，比如：c=a+b，c就是子tensor，a、b都为父tensor。
```
import torch
a=torch.tensor([1.,2,3,4],requires_grad=True)
b=torch.tensor([1.,2,3,4],requires_grad=False) #默认False
c=torch.sum(a*2)
c.backward()
print(a.grad)

d=torch.sum(b+a)
d.backward()
print(b.grad)
print(b.requires_grad)
'''结果
tensor([2., 2., 2., 2.])
None
True   #只要父tensor里有requires_grad=True的，子tensor的requires_grad=True
'''
```
可以看到requires_grad=True时进行反向传播会记录tensor的梯度信息，很方便我们使用梯度下降法。
**2、grad_fn**  
grad_fn也是tensor的一个属性，它记录的是tensor的运算信息，比如c=a+b，那么c.grad_fn=<AddBackward0>，记录grad_fn信息的意义有利于我们使用反向传播算法，由子tensor计算父tensor的梯度。这就是一个计算图
```
import torch
a=torch.tensor([1.,2,3,4],requires_grad=True)
b=torch.tensor([1.,2,3,4],requires_grad=False) #默认false
c=a+b
print(a.grad_fn,b.grad_fn)
print(c.grad_fn)
print(c)
'''结果
None None
<AddBackward0 object at 0x000002293EBCE7B8>
tensor([2., 4., 6., 8.], grad_fn=<AddBackward0>)
'''
```
a、b是直接创建的、没有经过运算的tensor，其grad_fn=None，还有一个值得注意的地方，如果父tensor的requires_grad都为False，子tensor的grad_fn=None。因为你都不准备记录父tensor的梯度信息了，那么再记录子tensor的grad_fn信息也没什么必要了。
3、leaf Variable
在写leaf Variable之前，我想先写一下Variable，可以帮助理清leaf Variable、requires_grad、grad_fn之间的关系。我们都知道，用pytorch搭建神经网络，数据都是tensor类型的，在先前的一些pytorch版本中（到底哪些我也不清楚，当前v1.3.1），tensor似乎只包含data信息，不会记录requires_grad、grad_fn信息，转化成variable之后才会有requires_grad和grad_fn属性。现在Variable基本可以被直接用tensor替代了，虽然现在还保留着Variable方法，我觉得用不到了，直接用tensor就可以了。
```
#Variable的使用
from torch.autograd import Variable
a=Variable(tensor,requires_gard=bool值)

```
leaf variable，你直接理解为手动创建的tensor也无妨，运算得到的tensor都不是leaf variable，可以用 .is_leaf查看tensor是否为leaf variable。

# 冷门但需要知道的函数
```
### Method resolution order (MRO)它描述了在多重继承的情况下，Python 如何按照特定顺序查找方法或属性。
print(Dataset.mro())
### output is "[<class 'torch.utils.data.dataset.Dataset'>, <class 'typing.Generic'>, <class 'object'>]".it means the order of retrieval is from Dataset to object.
```