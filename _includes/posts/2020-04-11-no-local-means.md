# 【计算机视觉】no-local means方法

## 大一统的高维高斯滤波表达式

![](/img/20200411/Figure1.png)

对任意的位置 `pi` 处的值 `vi` 进行滤波，与具有临近位置的其它值进行混合。通常这些值 `vi` 是均匀的像素颜色。

- 如果位置 `pi` 是两维的像素位置，那么该公式表达的是高斯模糊 `Gaussian blur`。
- 如果位置 `pi` 是像素位置与颜色的联合，比如`x,y,r,g,b`共五维，那么该公式表达的是颜色双边滤波 `color bilateral filter`。
- 如果位置矢量 `pi` 是来自每个像素的局部邻域，那么该公式表达的是非局部均值 `no-local means`。

本文主要探讨 no-local means 方法

## `no-local means`

### 引入

`Bennett and McMillan` 在用双边滤波器的组合对视频进行去噪时，注意到像素位置 `pi` 可以通过包括临近像素的颜色或者灰度值可以得到有用的扩增，对加性白噪声是最理想的去噪器。(`Buades et al.`)

### `no-local means`的含义

常规的高斯滤波，一个高斯核，以 `pi` 为中心，局部像素值的加权平均。这种都属于 `local means`。那么 `no-local means` 的含义指的是 位置 `pi` 被替换成像素位置的邻域每个像素，意思是用一个像素不足以表达像素之间的相似度，而用像素的`邻域小patch`就能更好地匹配相似模式。如下图所示：


![](/img/20200411/Figure2.png)

### 实现的流程

一般考虑到算法的复杂度，需要指定 `no-local means` 的`邻域小patch`的半径，比如7x7大小。然后还要指定对一个像素计算加权平均值的位置，也就是`搜索窗口半径`，比如21x21。表达的意思在计算该像素滤波输出值时，需要在以该像素位置为中心的21x21的图像块儿中，计算每个7x7的小patch与中心patch的相似度，并进行加权平均。

### 加速实现

很显然，这样的计算速度不会很快。很多加速算法，比如：一些采用预定义的区域来限制搜索区域（`Mahmoudi and Sapiro [MS05] preclassify regions of the image according to average intensity and gradient direction in order to restrict the search`）；结合积分图（`Darbon et al. [DCC*08] compute integral images of certain error terms, as do Wang et al. [WGY0*6]`）。这些方法都搜限制于位置矢量来自矩形的图像patches，而不能泛化至 结构更少的去噪任务，比如 `geometry denoising和photon density denoising`。


## 论文参考

- [Fast High-Dimensional Filtering Using the Permutohedral Lattice](http://graphics.stanford.edu/papers/permutohedral/)


-----
20200411