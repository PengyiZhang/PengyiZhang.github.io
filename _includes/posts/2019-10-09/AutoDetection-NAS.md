#【网络结构搜索】AutoDetection-NAS与目标检测的结合

@(2019-2020年度论文阅读)[模型部署, 网络结构搜索]

-----------
今年4月份Google Brain提出NAS-FPN：learning scalable feature pyramid architecture for object detection，在COCO上的test-dev AP 48.3%的成绩，可谓神经网络架构结构搜索在目标检测方面的开山之作。

## `1. NAS-FPN`
论文：https://arxiv.org/abs/1904.07392
Git非官方开源代码： https://github.com/DetectionTeamUCAS/NAS_FPN_Tensorflow
沿用RetinaNet的网络架构：（1）Backbone网络生成feature maps；（2）用NAS-FPN取代原有人工设计的FPN结构，跨尺度将低层high resolution的特征图与高层high semantic的特征图进行融合成更强的不同尺度的新特征；（3）利用次级子网络分别对class和bounding box进行分类和回归。
![Alt text](/img/201910/1570627903462.png)

事实上，NAS-FPN只是NAS在目标检测中的牛刀小试，因为它只是对整个网络的特征金字塔部分进行了搜索，用自动搜索得到的特征构成金字塔去替代原有的FPN。

我们具体来看NAS-FPN是如何定义搜索空间并实现搜索的。根据原文，其搜索空间定义可简述为包含Merging cell的特征金字塔，其中的merging cell用以合成新的feature map作为后续子网络的输入层。
![Alt text](/img/201910/1570627933271.png)
该图直观地展示了merging cell的机制：从现有的feature map集中不放回地任意选取两个feature map$\rightarrow$选择输出的分辨率$\rightarrow$选择此次的Binary Op，将选择的两个feature map融合并按照上一步选择的分辨率生成新的feature map$\rightarrow$将新生成的feature map添加到feature map集从而可基于此生成的feature map进一步搜索。

值得注意的是，Binary Op中包括sum和global pooling两个Op，而此处定义的global pooling沿用了语义分割论文Pyramid Attention Network for Semantic Segmentation中提出的Global Attention Upsample module即全局注意力上采样模块：
![Alt text](/img/201910/1570628308957.png)
通过该注意力机制，实现了High-level feature在特征融合时对low-level feature的引导，加强了生成的feature map中高层feature的信息。

最后，文中给出了通过设置merge次数为7时搜索出的网络架构：
![Alt text](/img/201910/1570628396365.png)
该架构在不同的backbone上进行实验，均优于原RetinaNet架构下表现，NAS-FPN AmoebaNet （input size为1280*1280）在COCO数据集上表现为test-Dev AP 48.3%，超越了当时state-of-the-art算法。

## `2. 推陈出现的FPN`

### `a. PaNet: Path Aggregation Network (From Sense Time&Tecent)`
论文：https://arxiv.org/abs/1803.01534
![Alt text](/img/201910/1570628557986.png)


`Path Aggregation Network`的核心思想是在原有特征金字塔的基础上添加了一条bottom-up path，再一次利用low-level特征增强resolution信息。


### `b.M2Det: Stacked U-shape Model  (From Ali Damo Academy&PKU)`
论文：https://arxiv.org/abs/1811.04533
![Alt text](/img/201910/1570628640902.png)


M2Det借用了语义分割处U-shape net的思想，通过在TUM(Thin U-shape Module)中模拟原始fpn的横向连接，形成类特征金字塔结构，并对TUM进行堆叠，得到不同层级的类金字塔结构，然后使用SFAM模块对其进行集成。值得注意的是，在SFAM模块中同样使用global pooling再激活引入了attention机制。如下图：

![Alt text](/img/201910/1570628668269.png)
此外，还有ZigZagNet通过dense connection对FPN进行改进等。总起来看，对FPN的改进多从语义分割典型网络结构借鉴思路；另外，注意力机制逐渐成为搭建特征金字塔的标配。

## `3. NAS多方位大展身手`

在NAS-FPN初试锋芒之后，其他团队纷纷一拥而上，针对目标检测中网络结构的不同部分进行了搜索实验：

### `a.NAS-FCOS`
论文：https://arxiv.org/abs/1906.04423
![Alt text](/img/201910/1570628743975.png)
如图所示，NAS-FCOS基于FCOS，尝试对FPN和prediction head同时进行搜索，并通过定制化的强化学习来控制搜索过程，并得到了以下结构：
![Alt text](/img/201910/1570628761131.png)
### `b. DetNAS (From Megvii. Inc, Chinese Academy of Science)`
论文：https://arxiv.org/abs/1903.10979

不同于NAS-FCOS与NAS-FPN，DetNAS向着backbone的搜索发起了冲击。通常目标检测使用的backbone是为图像分类设计的，某种程度上是次优的选择。该论文首次使用NAS对目标检测的backbone展开搜索；基于one-shot supernet，其搜索步骤如下：

![Alt text](/img/201910/1570628799367.png)
实验证明，搜索出的DetNAS比常用的ShuffleNetV2-40及ResNet-101精度表现提高5%以上。

## `4. 结语`

一方面，FPN的人工改进如火如荼，而不断推陈出新的特征融合模式也为NAS拓广搜索空间提供了灵感；另一方面，NAS在模型不同模块的实验成功也促使人们朝着更大的搜索域迈进。笔者不禁好奇，接下来的下半年，NAS与目标检测又将碰撞出怎样夺目的火花，让我们拭目以待！

----
2019-10-09 