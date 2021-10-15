# PyTorch中backward()函数的gradient参数作用

这篇文章讲得比较清晰，特地备份一下： [pytorch中backward函数的gradient参数作用](https://www.cnblogs.com/zhouyang209117/p/11023160.html)

## 问题引入

在深度学习中，经常需要对函数求梯度（`gradient`）。`PyTorch`提供的`autograd`包能够根据输入和前向传播过程自动构建计算图，并执行反向传播。

`PyTorch`中，`torch.Tensor`是存储和变换数据的主要工具。如果你之前用过`NumPy`，你会发现`Tensor`和`NumPy`的多维数组非常类似。

然而，`Tensor`提供`GPU`计算和自动求梯度等更多功能，这些使`Tensor`更加适合深度学习。

> `PyTorch`中`tensor`这个单词一般可译作`张量`，张量可以看作是一个多维数组。标量可以看作是零维张量，向量可以看作一维张量，矩阵可以看作是二维张量。

如果将`PyTorch`中的`tensor`属性`requires_grad`设置为`True`，它将开始追踪(`track`)在其上的所有操作（这样就可以利用链式法则进行梯度传播了）。  

完成计算后，可以调用`backward()`来完成所有梯度计算。此`tensor`的梯度将累积到`grad`属性中。 如下：
```python
import torch

x = torch.tensor([1.0, 2.0, 3.0], requires_grad=True)
y = x ** 2 + 2
z = torch.sum(y)
z.backward()
print(x.grad)
```
输出:
```bash
tensor([2., 4., 6.])
```
下面先解释下这个`grad`怎么算的。

## 导数、偏导数的简单计算

从数学定义上讲,求导或者求偏导大多是函数对自变量而言。但是很多机器学习的资料和开源库都会看到说标量对向量求导。 

先简单解释下上面的例子：

设 $x=[x_1,x_2,x_3]$ 

$$
1.02^{365} \approx 1377.4
$$
