#【深度学习】Conditional Batch Normalization 详解

@(2019-2020年度论文阅读)[超分辨率]

-----------
![Alt text](./1569582434919.png)

Conditional Batch Normalization 的概念来源于这篇文章：[Modulating early visual processing by language](https://papers.nips.cc/paper/7237-modulating-early-visual-processing-by-language.pdf)后来又先后被用在 [cGANs With Projection Discriminator](https://arxiv.org/pdf/1802.05637.pdf) 和[Self-Attention Generative Adversarial Networks](https://arxiv.org/pdf/1805.08318.pdf) 。本文将首先简略介绍 Modulating early visual processing by language ，接着结合 Self-Attention GANs 的 pytorch 代码，详细分析 categorical conditional Batch Normalization 的具体实现。

传统的 Batch Normalization (BN) 公式为：

![Alt text](./1569582585154.png)
条件BN中，scale和bias的系数是把feature输入到一个小神经网络多层感知机，前向传播的网络输出，而不是学习得到的网络参数。由于scale和bias依赖于输入feature这个condition，因此这个改进版本的Batch Normalization叫做`Conditional Batch Normalization`。

## Modulating early visual processing by language
这篇文章改进了一个基于图片的问答系统 (VQA: Visual Question Answering)。系统的输入为一张图片和一个针对图片的问题，系统输出问题的答案，如下图所示：
![Alt text](./1569582434919.png)

这类系统通常是这样设计的：一个预训练的图像识别网络，例如 ResNet，用于提取图片特征；一个 sequential 模型，例如 LSTM、GRU 等，用于提取句子的特征，并根据句子预测应该关注图片的什么位置（attention）；将语言特征、由 attention 加权过后的图片特征结合起来，共同输入一个网络，最终输出问题的答案。

上图左侧为传统的 VQA 系统，我们发现，LSTM 提取的特征只在 ResNet 的顶层才和图片特征结合起来，因为通常意义上讲，神经网络的底层提取的是基础的几何特征，顶层是有具体含义的语义特征，因此，应该把语言模型提取的句子特征在网络顶层和图片特征结合。然而，作者认为，底层的图片特征也应该结合语言特征。理由是，神经科学证明：语言会帮助图片识别。例如，如果事先告诉一个人关于图片的内容，然后再让他看图片，那么这个人识别图片的速度会大大加快。因此，作者首创了将图片底层信息和语言信息结合的模型，如上图右侧所示。


具体是如何结合的呢？首先，ResNet 是预训练的网络，用于提取图片特征，因此不能轻易修改里面 filter 的参数。而其中的BN层有两组参数scale和bias，用于对feature map施加缩放和偏置操作。这两个参数量不大，而且从含义上讲可以解释为：强调feature map的某部分channel，忽略另外一些channel。柿子捡软的捏，作者决定通过修改scale和bias的方式，达到有针对性地提取图片部分信息的目的。而修改的方式就是用LSTM提取的句子特征。例如上图，输入的句子问：伞上下颠倒了吗？LSTM 很大概率会提取出关键词：伞，把这个关键词的特征作为条件，输入到多层感知机 (MLP) 中，输出新的权重bias和scale，通过训练，这些权重最后将会有针对性地强调图片特征中与伞有关的channel，而忽略与伞无关的channel。而由于ResNet是预训练网络，即便是里面的BN层的参数，也是轻易不能动的。因此，作者没有直接用MLP的输出作为BN层新的scale和bias，而是作为一个小的增量，加在原来的参数上：


![Alt text](./1569583144266.png)
这个想法用最小的代价（只修改了 BN 层参数），在图像的底层 feature 中结合了自然语言信息，取得了很好的表现。相关的代码为：https%3A//github.com/ap229997/Conditional-Batch-Norm/blob/master/model/cbn.py

## Categorical Conditional Batch Normalization

在 conditional generative model 里面，存在一个隐隐让人不安的问题：一个 batch 里面不同类别的训练数据，放在一起做 Batch Normalization 不太妥当。因为不同类别的数据理应对应不同的均值和方差，其归一化、放缩、偏置也应该不同。针对这个问题，一个解决方案是不再考虑整个 batch 的统计特征，各个图像只在自己的 feature map 内部归一化，例如采用 Instance Normalization 和 Layer Normalization 来代替 BN。但是这些替代品的表现都不如 BN 稳定，接受程度不如 BN 高。

这时我们想到了上一节中介绍的 conditional BN。CBN 以 LSTM 提取的自然语言特征作为 condition，预测 BN 层参数的增量，达到对不同的输入，都有相对应的归一化参数。既然自然语言特征可以作为 condition，用于预测 BN 参数的变化，那么图片的类别信息自然也可以作为 condition 来预测 BN 层的参数。因此 cGANs With Projection Discriminator 和 Self-Attention GANs 借鉴了 CBN 里面的 condition 的思想，稍加修改，用在了自己的 conditional GAN 模型中。

Modulating early visual processing by language 一文中，由于使用了预训练的 ResNet，不敢对预训练网络 BN 层的参数做大修改，因此 MLP 的输出为 BN 层参数的增量，而不是直接输出新的 BN 层参数。conditional GANs 没有用到预训练网络，因此没有了历史包袱，直接用图片的 categorical 信息，预测新的scale和bias。
接下来我们将研究其具体的实现，代码来自：https://github.com/crcrpar/pytorch.sngan_projection/blob/master/links/conditional_batchnorm.py

我们看到，这个 ConditionalBatchNorm2d类，继承自 pytorch 的 BatchNorm2d类，对比这个代码和官方的 BatchNorm2d 的代码，发现其构造函数的参数和BatchNorm2d完全相同，构造函数中直接调用了基类，也就是BatchNorm2d的构造函数。而 forward函数中，多了weight和bias两个参数。forward的代码大部分也是直接 copy 自 BatchNorm2d的基类_BatchNorm的代码，无非是设置一下 moving average 的 momentum，记录一下总共读取了多少个 batch，以便在没有设置 momentum 的情况下，在全体样本上计算均值和方差。直到调用官方的底层 C 函数库 F.batch_norm，代码完全没有对_BatchNorm类的forward函数做出任何修改，其output 就是对输入的 feature map 做了一次 BatchNorm2d。 真正修改的是后面加的几行：

	  if weight.dim() == 1:
	            weight = weight.unsqueeze(0)
	        if bias.dim() == 1:
	            bias = bias.unsqueeze(0)
	        size = output.size()
	        weight = weight.unsqueeze(-1).unsqueeze(-1).expand(size)
	        bias = bias.unsqueeze(-1).unsqueeze(-1).expand(size)
	        return weight * output + bias 

这里用到了forward函数参数中的 weight和bias。由于是在图像 feature 上操作，需要对 weight 和 bias 的维度做一些改变，使其与 feature map output的维度相同。最后代码返回weight*output+bias 。似乎很 naive，可是说好的 condition 呢？说好的 categorical 信息呢？别着急，它们都隐藏在 weight和bias中。这个类只不过是个基类，下面的类才是真正要用到的类：

```
class CategoricalConditionalBatchNorm2d(ConditionalBatchNorm2d):

    def __init__(self, num_classes, num_features, eps=1e-5, momentum=0.1,
                 affine=False, track_running_stats=True):
        super(CategoricalConditionalBatchNorm2d, self).__init__(
            num_features, eps, momentum, affine, track_running_stats
        )
        self.weights = nn.Embedding(num_classes, num_features)
        self.biases = nn.Embedding(num_classes, num_features)

        self._initialize()

    def _initialize(self):
        init.ones_(self.weights.weight.data)
        init.zeros_(self.biases.weight.data)

    def forward(self, input, c, **kwargs):
        weight = self.weights(c)
        bias = self.biases(c)

        return super(CategoricalConditionalBatchNorm2d, self).forward(input, weight, bias)
```
这个类的构造函数中比它的基类多加了一项num_classes。构造函数中，首先调用了它的基类，也就是ConditionalBatchNorm2d的构造函数，用于初始化大部分参数。接下来设置了两个网络层：

	self.weights = nn.Embedding(num_classes, num_features)
    self.biases = nn.Embedding(num_classes, num_features)

`nn.Embedding`层的作用是，把图片的 label 转换成 dense 向量，而不像 `one-hot-encoding`，只能把 label 转换成稀疏向量。nn.Embedding的第一个参数表示总共有多少个类，第二个参数表示每个 label 映射成多少维的向量。这个网络层的好处是，可以任意指定 label vector 的 dimension，它的本质是一个 num_classes行num_feature列的矩阵，这个矩阵的参数随着网络的训练不断更新。前向传播时，label 是几就取第几行的向量出来，用以表示这个 label。其实这个 Embedding 相当于把 one-hot encoding 输入一个 bias 为 0 的 linear layer。

在构造函数的最后，通过调用 self._initialize初始化 self.weights 和 self.bias，分别把它们初始化为全 1 和全 0。这样在网络训练的初期，这俩相当于不存在一样，整个类就是一个BatchNorm2d。
接下来看前向传播函数： 

```
def forward(self, input, c, **kwargs):
    weight = self.weights(c)
    bias = self.biases(c)

    return super(CategoricalConditionalBatchNorm2d, self).forward(
                 input, weight, bias)
```

这个函数也很简单，输入 feature map input和类别标签c，注意c 应该是 LongTensor 格式的，否则会报错。接下来，根据 c 挑出 weights embedding 层和 biases embedding 层中的第c行，作为 weight 和 bias 输入基类的前向传播函数，最终得到 Conditional Batch Normalization 的输出。这个 categorical condition 发挥作用的阶段，就是 embedding 的阶段。

这个类的实现，对原始 Modulating early visual processing by language 论文做了几点改动：

![Alt text](./1569584148787.png)


## 总结
提出 conditional Batch Normalization 这一思想的论文 Modulating early visual processing by language，是为了解决特定问题：即在预训练 ResNet 提取的图片底层信息中，融合进自然语言信息，用于辅助图片信息的提取。

而后面的 cGANs With Projection Discriminator 和Self-Attention Generative Adversarial Networks 则是利用 condition 的思想，把图片的 categorical 信息用来指导生成 BN 层的映射参数。`我们发现，网络训练完成后，同一个类别的图片，将对应同一套 BN 层参数，不同类别的图片，将对应不同的 BN 层参数`。

通过这个微小的改动，我们终于可以愉快地在 conditional generative model 上使用 Batch Normalization 操作，而不必担心不同类别的图片对应不同的映射参数了。

----
2019-09-27 张朋艺