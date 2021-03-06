# 【计算机视觉】多示例学习

## 问题描述

有这样的训练数据，给定的标签是按照包 `bag` 来组织的，也就是说 positive 的包至少含有一个正样本，而 negative 的包中都是负样本。如何训练模型实现良好的分类效果？这正是多示例学习所研究的问题设定。


## 在应用中的对照

特别是目标检测、跟踪中，负样本很好选取，而最优正样本则很难。通常正样本的选取是在标记的矩形框附近做一些扰动，得到一堆儿框放入 positive bag中，总有一个是最佳的。

## 求解方法

如果所有的样本标记已知，那问题就转化为监督学习的问题。所以问题是正样本包中只能保证有一个是正样本，而且并不知道是哪一个？

解决方法就是迭代优化：假设知道了所有样本标记，用监督学习得到一个分类模型，然后通过这个分类模型对正样本进行预测，然后更新它们的标记，之后就可以继续进行模型训练。就这样两个步骤，EM steps：监督学习，标记更新。

注意点：

- 监督训练模型的时候，只从正样本包中挑选最正确的（分类得分最高）的那个。正样本包中的其它样本，不管是正的还是负的，通通不要；

- 如果负样本足够多的话，可以只挑出来负样本中最像正样本的作为负样本进行训练，其实就是结合做 `data mining` 找到 `hard samples` 困难样本。

下面可以参考一些具体多示例的应用进行说明：

![](/img/20200415/Figure12.png)

这个是 `Latent-SVM` 的训练过程，给定正样本包和负样本图像，以及初始模型。开始负样本集合初始化为空集∅。每次 `relabel` 的过程，首先将正样本训练集设置为空集，然后用模型来对正样本包中的图像进行检测，将概率最大的正样本加入到正样本集中。然后进行 `data mining` 过程，选取最难分的样本加入到负样本集中。之后将正样本集合和负样本集合合并采用梯度下降的方法对模型参数进行训练。

`EM` 过程最直观的可以想象 meanshift算法。初始不知道重心在哪，随机选择一个重心（随机初始化一个模型），然后以该重心为中心，画一个半径为R的圆(球)，计算该圆的重心，再将其设置为新的重心。重复这个过程，直到重心的位置不再发生大的变化。

- `E step`：根据现有模型求期望；
- `M step`：根据最大化更新模型。

- `E step`. Estimate the expected value for each latent variable.
- `M step`. Optimize the parameters of the distribution using maximum likelihood.

这种交替优化的方式十分值得注意NOTICE！！！因为非常有用，将一个看上去无法完成的任务，通过先明确一个状态A，去估计另一个状态B，然后再用状态B去优化状态A。这种方法在一定的情况下是能够保证收敛到最优的，即使不是最优，也能使次优吧！

## 混合高斯模型的拟合例子

比如用混合高斯模型拟合一个多峰的数据分布，可以先用直方图查看一下数据分布情况，设置对应的混合高斯的高斯个数为2。

```python
# example of a bimodal constructed from two gaussian processes
import numpy as np 
from matplotlib import pyplot 
from sklearn.mixture import GaussianMixture

X1 = np.random.normal(loc=20, scale=5, size=3000)
X2 = np.random.normal(loc=40, scale=5, size=7000)

X = np.hstack((X1,X2))

pyplot.hist(X, bins=50, density=True)
pyplot.show()

X = X.reshape((len(X), 1))

model = GaussianMixture(n_components=2, init_params='random')
model.fit(X)

yhat = model.predict(X)
print(yhat[:100])
print(yhat[-100:])

```



## 参考内容

[https://machinelearningmastery.com/expectation-maximization-em-algorithm/](https://machinelearningmastery.com/expectation-maximization-em-algorithm/)



-----
20200415

