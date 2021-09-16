# 从线性回归走进机器学习

## 认识线性回归

> 回归，是指研究一组随机变量(`Y1 ，Y2 ，…，Yi`)和另一组(`X1，X2，…，Xk`)变量之间关系的统计分析方法。

![](http://img2020.cnblogs.com/blog/1546632/202108/1546632-20210810145620814-401160334.png)

线性回归是统计学中最基础的数学模型，在很多学科的研究中都能看到线性回归的影子，比如量化金融、计量经济学等等。线性回归通过对已有数据建模，从而实现对未知数据的预测。下面会通过一个房价预测的例子来详细说明。

数据来自[https://www.kaggle.com/kennethjohn/housingprice](https://www.kaggle.com/kennethjohn/housingprice)，该数据集提供了波特兰市47套房屋的面积(`HouseSize`)、卧室数量(`Bedrooms`)和价格(`Price`)，如下：

```python
import pandas as pd

data = pd.read_csv('https://files.cnblogs.com/files/blogs/478024/Housing-Prices.txt.zip')
print(data.head(5))
```

| 房屋面积（平方英尺） | 卧室数量 | 价格（美元） |
| :------------------: | :------: | :----------: |
|         2104         |    3     |    399900    |
|         1600         |    3     |    329900    |
|         2400         |    3     |    369000    |
|         1416         |    2     |    232000    |
|         3000         |    4     |    539900    |

基于已有数据，我们希望通过计算机的学习，找到数据中的规律，并用来预测其他房屋的价格。这是机器学习最朴素的应用场景。这个过程也被称为监督学习（`Supervised Learning`），即给定一些数据，使用计算机学习到一种模型，然后用它来预测新的数据。

![](http://img2020.cnblogs.com/blog/1546632/202108/1546632-20210812151605054-1627775632.png)

在房价预测的例子中，想要预测的目标值房价是连续的，我们称这类问题为回归（`Regression`）问题。与之相对应，当目标值只能在一个有限的离散集合里选择，比如预测房价是否大于100万，结果只有“是”和“否”两种选项，我们称这类问题为分类（`Classification`）问题。

## 线性回归初体验

上面我们知道，通过训练已有数据，得到模型，然后预测其他房屋的价格，这里，先使用`scikit-learn`这个开源的python机器学习库感受一下。

官网：[https://scikit-learn.org/stable/index.html](https://scikit-learn.org/stable/index.html)

（初次接触机器学习没有环境的朋友可以参考之前的文章《Python开发环境搭建》：[https://www.cnblogs.com/bytesfly/p/python-environment.html](https://www.cnblogs.com/bytesfly/p/python-environment.html)）


```python
import numpy as np
import pandas as pd
from sklearn.linear_model import SGDRegressor
from sklearn.pipeline import make_pipeline
from sklearn.preprocessing import StandardScaler

# 获取数据集
data = pd.read_csv('https://files.cnblogs.com/files/blogs/478024/Housing-Prices.txt.zip')

# 特征标准化，并选择梯度下降法进行线性回归
reg = make_pipeline(StandardScaler(), SGDRegressor())
# 机器学习(模型训练)
reg.fit(data[['HouseSize', 'Bedrooms']], data['Price'])

# 待预测数据
predict_data = np.array([
    [1000, 2],
    [3000, 3],
    [4000, 4],
    [5000, 5]
])
# 预测
result = reg.predict(predict_data)
# 打印预测结果
print(np.c_[predict_data, result].astype(np.int))
```
预测结果：

| 房屋面积（平方英尺） | 卧室数量 | 价格（美元） |
| :------------------: | :------: | :----------: |
|         1000         |    2     |    211108    |
|         3000         |    3     |    480319    |
|         4000         |    4     |    610837    |
|         5000         |    5     |    741356    |

当然，实际生产中需要经过多次调优，且模型评估结果达到一定的要求才能正式上线。

## 特征工程

在讨论线性回归的数学表示之前，我觉得有必要提一下特征工程(`Feature Engineering`)。上面“线性回归初体验”的代码注释中，有`特征标准化`字眼，对应的代码是`StandardScaler()`，初学者应该会疑惑这个地方是什么意思，为什么需要这一步。

> 数据和特征决定了机器学习的上限，而模型和算法只是逼近这个上限而已。

可见，特征工程在机器学习中占有相当重要的地位。在实际应用当中，可以说特征工程是机器学习成功的关键。

> Feature engineering is the process of using domain knowledge of the data to create features that make machine learning algorithms work.

如果你想要你的预测模型达到最佳，你不仅要选择最优的算法，还要尽可能的从原始数据中获取更多有价值的信息。这就是特征工程要做的事，它的目的是得到更好的训练数据，是使用专业背景知识和技巧处理数据，使得特征能在机器学习算法上发挥更好的作用这一过程。

换句话说，特征工程就是一个把原始数据转化成特征的过程，这些特征可以很好的描述这些数据，并且利用它们建立的模型在未知数据上的表现效果可以尽可能的好。从数学的角度来看，特征工程就是人工地去设计输入自变量`X`。



特征工程包含以下内容：

- 特征提取：将任意数据（如文本、图像、声音等）转换为可用于机器学习的数字特征，因为机器学习进行的是数值相关的运算处理。这里我们收集影响房价的因素，比如房屋面积大小、卧室数量、出门到附近地铁口的距离、周围学校的数量等数字特征。

- 特征选择：从特征集合中挑选一组最具统计意义的特征子集，从而达到`降维`的效果。这里我们为了简化问题，只选择`房屋面积`和`卧室数量`这两个属性作为特征值。实际生产中肯定需要根据专业背景知识选择特征值。

- 特征预处理：通过一些转换函数将特征数据转换成更加适合算法模型的特征数据。

![](http://img2020.cnblogs.com/blog/1546632/202108/1546632-20210816094548806-1299955245.png)

> The `sklearn.preprocessing` package provides several common utility functions and transformer classes to change raw feature vectors into a representation that is more suitable for the downstream estimators.

`Feature Scaling`(常叫”特征归一化“、”标准化“)，是特征预处理中的重要技术，有时甚至决定了算法能不能work以及work得好不好。为什么需要`Feature Scaling`呢？

1. 特征间的单位（尺度）可能不同，比如这里的房屋面积和卧室数量，房屋面积的变化范围可能是[800,8000]，卧室数量的变化范围可能是[1,5]，在计算时，尺度大的特征(房屋面积)会起决定性作用，而尺度小的特征(卧室数量)其作用可能会被忽略，为了消除特征间单位和尺度差异的影响，以对每维特征同等看待，需要对特征进行归一化。
2. 可以有效提高梯度下降(`Gradient Descent`)的收敛速度。等后面讲过梯度下降法应该就能明白。

先看看如何使用`sklearn.preprocessing`下的`StandardScaler`：

```python
import pandas as pd
from sklearn.preprocessing import StandardScaler

# 获取数据集
data = pd.read_csv('https://files.cnblogs.com/files/blogs/478024/Housing-Prices.txt.zip',
                   usecols=['HouseSize', 'Bedrooms'])
# 这里为了演示，只选择前5行数据进行特征标准化
data = data[:5]

scaler = StandardScaler()
# Compute the mean and std to be used for later scaling
scaler.fit(data)
# Perform standardization by centering and scaling
print(scaler.transform(data))
```

这样，就把前5行原始数据转换成了：

```bash
[[ 0.          0.        ]
 [-0.88604177  0.        ]
 [ 0.52037374  0.        ]
 [-1.20951734 -1.58113883]
 [ 1.57518537  1.58113883]]
```

那么，`StandardScaler`到底对数据做了怎样的转换呢？

![](http://img2020.cnblogs.com/blog/1546632/202108/1546632-20210816101124628-193119658.png)

注意：该转换作用于每一列(即每个特征)。其中`mean`表示这一列数据的平均值，`σ`表示这一列数据的标准差。

这样，原始数据就被映射到均值为`0`，标准差为`1`范围内。

如何证明呢？其实并不困难。均值为0很显然，每个原始值都减去`mean`，再求和必然为0，均值也就自然是0。再看标准差：

![](http://img2020.cnblogs.com/blog/1546632/202108/1546632-20210816110104427-1559209730.png)

把均值0代进去，就变成了求x的平方和了，再看分子、分母，其实是相等的(能`get`到？想想均值与标准差之间的联系？标准差是如何来的？)。如此一来算出来的结果正好是1。

清楚了`StandardScaler`的转换过程，我们也可以自己实现一下，如下：

```python
import numpy as np
import pandas as pd

# 获取数据集
data = pd.read_csv('https://files.cnblogs.com/files/blogs/478024/Housing-Prices.txt.zip',
                   usecols=['HouseSize', 'Bedrooms'])
# 这里为了演示，只选择前5行数据进行标准化
data = data[:5]

for column in data.columns:
    # 取出每一列(即每个特征)的数据
    value = data[column]
    # 求平均值
    mean = np.mean(value)
    # 求标准差
    std = np.std(value)
    # 对原始数值进行转换
    data[column] = (value - mean) / std

print(data)
```

运行结果如下：

```bash
   HouseSize  Bedrooms
0   0.000000  0.000000
1  -0.886042  0.000000
2   0.520374  0.000000
3  -1.209517 -1.581139
4   1.575185  1.581139
```

对比前面用`sklearn.preprocessing`下的`StandardScaler`计算出来的结果是一样的。

温馨提示：上面代码中求标准差时用的是`numpy`中的`std`(计算时默认除以`N`)，而不是`pandas`中的`std`(计算时默认除以`N-1`)，否则计算出来的结果会有些差异。

## 线性回归的数学表示

现在把线性回归问题扩展到更一般的场景。假设x是多元的，或者说是多维的。比如，预测房价，需要考虑房屋面积大小、卧室数量、出门到附近地铁口的距离、周围学校或商场的数量等等。

![](http://img2020.cnblogs.com/blog/1546632/202108/1546632-20210816170319986-1823209032.png)
这里，训练已有房价数据就是为了确定参数`w`，一旦确定了`w`，就能根据`x`的取值，求出`y`，也就预测出了房价。

此时线性回归的监督学习过程就可以被定义为：给定n个数据对`(x, y)`，寻找最佳参数`w` ，使模型可以更好地拟合这些数据。

如何衡量模型是否以最优的方式拟合数据呢？

机器学习中常用损失函数（`Loss Function`）来衡量这个问题。损失函数又称为代价函数（`Cost Function`），它计算了模型预测值`y`和真实值`y`之间的差异程度。从名字也可以看出，这个函数计算了模型犯错的损失或代价，一般来说，损失函数值越小，数据拟合得可能越好（这里暂不考虑过拟合的情况）。

对于线性回归，一个简单实用的损失函数计算方式是预测值与真实值误差的平方的均值。数理统计中叫做均方误差(`MSE: Mean Squared Error`)。`MSE`的值越小，说明预测模型具有更好的精确度。

![](http://img2020.cnblogs.com/blog/1546632/202108/1546632-20210817102419696-1022285028.png)

同样，`scikit-learn`也提供了对`MSE`相关的描述与实现，详情见：

[https://scikit-learn.org/stable/modules/model_evaluation.html#mean-squared-error](https://scikit-learn.org/stable/modules/model_evaluation.html#mean-squared-error)

![](http://img2020.cnblogs.com/blog/1546632/202108/1546632-20210817103202555-2000470016.png)

下面简单看下如何用`scikit-learn`中的`mean_squared_error`来计算模型误差：

```python
from sklearn.metrics import mean_squared_error

y_true = [3, -0.5, 2, 7]
y_pred = [2.5, 0.0, 2, 8]

print(mean_squared_error(y_true, y_pred))
```

打印结果为：

```ba
0.375
```

与你口算结果是否一致？

## 梯度下降法的推导

上面我们描述了线性回归的数学表示以及计算损失的方法，这样一来，问题就转换成了，求解最佳参数`w`，代入所有已知的训练数据对`(x, y)`到上面的均方差公式中，使得计算出来的损失值`L(w)`尽可能小。其实，本质上是一个求解最优化问题。

`值得注意的是，在房价预测中，房价影响因素x是自变量，房价y是因变量，但在这里的损失函数中，训练数据对(x, y)是已知的(不再是变量了)，而w可以看成是自变量，损失值L(w)是因变量，损失值L(w)随着w取值的变化而变化。我们关心L(w)最小时，w的取值，这样w就变成已知的了，此时认为模型可以拟合这些训练数据，进而用来预测其他房屋的价格。`

讲到求损失函数的最小值，高中数学我们知道，一种办法可以直接让其导数为零，直接寻找损失函数的极小值点。这种方法求最优解，其实是在解这个矩阵方程，英文中称这种方法为正规方程（`Normal Equation`）。感兴趣的朋友可以自行网上查询一些资料学习一下。

由于正规方程法可能不存在唯一解，且当特征维度较大时，计算量比较大，非常耗时。下面重点介绍另外一种求解方法————梯度下降法。

> 求解损失函数最小问题，或者说求解使损失函数最小的最优化问题时，经常使用搜索的方法。具体而言，选择一个初始点作为起点，然后开始不断搜索，损失函数逐渐变小，当到达搜索迭代的结束条件时，该位置为搜索算法的最终结果。我们先随机猜测一个`w`，然后对`w`值不断进行调整，来让`L(w)`逐渐变小，最好能找到使得`L(w)`最小的`w`。

![](http://img2020.cnblogs.com/blog/1546632/202108/1546632-20210817161028621-151607314.png)

> 一般情况下，0 < α < 1 。α越大，表示我们希望损失函数以更快的速度下降，α越小，表示我们希望损失函数下降的速度变慢。如果α设置得不合适，每次的步长太大，损失函数很可能无法快速收敛到最小值(步长太大可能直接跨过了极小值点)；步长太小，计算次数过多，时间过长，效率就很差了。到这里，你能理解上面特征预处理时，对特征值进行归一化、标准化的好处吗？【可以有效提高梯度下降(`Gradient Descent`)的收敛速度】

当一个训练集有`m`个训练样本时，求导只需要对多条训练样本的数据做加和。

![](http://img2020.cnblogs.com/blog/1546632/202108/1546632-20210817162449483-771338874.png)

代入公式(15)，如下：

![](http://img2020.cnblogs.com/blog/1546632/202108/1546632-20210817162659898-1988297446.png)

`w`是一个向量，假设它是`n`维的，在更新`w`时，需要同时对`n`维所有`w`值进行更新，其中第`j`维就是使用这里的公式。

具体而言，这个算法为：

![](http://img2020.cnblogs.com/blog/1546632/202108/1546632-20210817163222156-757384512.png)

> 这一方法在每一次迭代时使用整个训练集中的所有样本来更新参数，也叫做批量梯度下降法（`Batch Gradient Descent，BGD`）。线性回归的损失函数`L`是一个凸二次函数（`Convex Quadratic Function`），凸函数的局部极小值就是全局最小值，线性回归的最优化问题只有一个全局解。也就是说，假设不把学习率`α`设置的过大，迭代次数足够多，梯度下降法总是收敛到全局最小值。

## 梯度下降法的实现

有了上面的层层铺垫，可以用`NumPy`自己实现一下。代码的关键地方大多加了注释，遇到不明白的步骤建议再读读前面的相关内容，遇到不熟悉的numpy函数建议单独看看什么意思，遇到矩阵乘法可以用纸笔简单演算一下。

```python
import numpy as np


class Transformer:
    def fit(self, x):
        pass

    def transform(self, x):
        return x


class StandardTransformer(Transformer):
    def __init__(self):
        self.means = []
        self.stds = []

    def fit(self, x):
        # 按照每个维度(特征)统计, 即按列统计
        self.means = np.mean(x, axis=0)
        self.stds = np.std(x, axis=0)

    def transform(self, x):
        x = x.copy()
        for i in range(x.shape[1]):
            # 对每列执行特征标准化
            x[:, i] = (x[:, i] - self.means[i]) / self.stds[i]
        return x


class LinearRegression:

    def __init__(self, transformer: Transformer):
        self.transformer = transformer
        self.w = None
        self.losses_history = []

    def fit(self, x, y, alpha=0.0001, num_iters=1000):
        num_of_samples, num_of_features = x.shape
        # 计算均值与标准差，用于后面的特征标准化
        self.transformer.fit(x)
        # 特征标准化
        x_ = self.transformer.transform(x)
        # 在线性回归的数学表示中,有过说明,为了简化公式方便使用矩阵运算，在特征值第一列前面插入全为1的一列
        x_ = np.column_stack((np.ones(num_of_samples), x_))

        # 把y变成列向量，后面参与矩阵运算
        y_ = y.reshape((num_of_samples, 1))

        # 使用梯度下降法迭代计算w
        self.gradient_descent(x_, y_, alpha, num_iters)

    def gradient_descent(self, x, y, alpha: float, num_iters: int):
        num_of_samples, num_of_features = x.shape
        # 初始化参数w全为0
        self.w = np.zeros((num_of_features, 1))

        # 开始迭代
        for i in range(num_iters):
            # (num_of_samples, num_of_features) * (num_of_features, 1)
            y_ = np.dot(x, self.w)

            diff = y_ - y

            # 这一步建议再回头对照上面推导出来的梯度下降法的迭代公式
            # 如果矩阵乘法有点难理解, 建议先在纸上对照公式看一个特征是怎么计算的(即x是按照一列一列参与运算的)
            gradient = np.dot(diff.T, x).T
            # 更新w
            self.w -= alpha * gradient

            # 计算损失
            loss = np.sum(diff * diff) / num_of_samples / 2
            # 加入到迭代过程中的损失统计列表
            self.losses_history.append(loss)

    def predict(self, x):
        num_of_samples, num_of_features = x.shape
        # 特征标准化
        x_ = self.transformer.transform(x)
        # 在线性回归的数学表示中,有过说明,为了简化公式方便使用矩阵运算，在特征值第一列前面插入全为1的一列
        x_ = np.column_stack((np.ones(num_of_samples), x_))
        # 根据训练数据计算出来的w, 直接代入计算即可(矩阵乘法)
        return np.dot(x_, self.w)


if __name__ == '__main__':
    import pandas as pd

    # 获取数据集
    data = pd.read_csv('https://files.cnblogs.com/files/blogs/478024/Housing-Prices.txt.zip')

    # 调用上面自己实现的线性回归(梯度下降法)
    reg = LinearRegression(StandardTransformer())
    # 训练数据
    reg.fit(data[['HouseSize', 'Bedrooms']].values, data['Price'].values)

    # 待预测数据
    predict_data = np.array([
        [1000, 2],
        [3000, 3],
        [4000, 4],
        [5000, 5]
    ])
    # 预测
    result = reg.predict(predict_data)
    # 打印预测结果
    print(np.c_[predict_data, result].astype(np.int))

    # 画图观察一下随着迭代次数的增加, 损失值的变化
    import matplotlib.pyplot as plt

    losses_history = reg.losses_history

    plt.plot(np.arange(0, len(losses_history)), losses_history)
    plt.xlabel("num_of_iter")
    plt.ylabel("loss")
    plt.show()
```

此时的预测结果为：

| 房屋面积（平方英尺） | 卧室数量 | 价格（美元） |
| :------------------: | :------: | :----------: |
|         1000         |    2     |    178440    |
|         3000         |    3     |    426503    |
|         4000         |    4     |    567997    |
|         5000         |    5     |    709491    |

再看随着迭代次数的增加，损失值的变化图：

![](http://img2020.cnblogs.com/blog/1546632/202108/1546632-20210818163053622-198898415.png)

可见，在迭代到500次的时候损失值基本不再下降了。当然迭代多少次趋于稳定，与步长(学习率`alpha`)密切相关。

## 总结

线性回归是机器学习技术的一个很好的起点，对初学者走进机器学习很有帮助。



参考:

线性回归的求解：[https://lulaoshi.info/machine-learning/linear-model/minimise-loss-function.html](https://lulaoshi.info/machine-learning/linear-model/minimise-loss-function.html)
