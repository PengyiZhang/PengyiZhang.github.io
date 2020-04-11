# 【计算机视觉】双边滤波 `bilateral filter` 方法

## 大一统的高维高斯滤波表达式

![](/img/20200411/Figure1.png)

对任意的位置 `pi` 处的值 `vi` 进行滤波，与具有临近位置的其它值进行混合。通常这些值 `vi` 是均匀的像素颜色。

- 如果位置 `pi` 是两维的像素位置，那么该公式表达的是高斯模糊 `Gaussian blur`。
- 如果位置 `pi` 是像素位置与颜色的联合，比如`x,y,r,g,b`共五维，那么该公式表达的是颜色双边滤波 `color bilateral filter`。
- 如果位置矢量 `pi` 是来自每个像素的局部邻域，那么该公式表达的是非局部均值 `no-local means`。

本文主要探讨 bilateral filter 方法

![](/img/20200411/Figure5.png)

高维度的高斯滤波实现起来，可以首先将在位置`p`处的值`v`插入到高维空间 (`splatting`)，然后在这个高维空间执行 Gaussian blur (`blurring`)，然后在原来位置 `p` 处进行空间采样 (`slicing`)。

这个就是一个一维信号的双边滤波器实现。


## `bilateral filter`

### 引入

高斯模糊的缺点就是使得边缘位置由于取自边缘两边的像素的平均，所以会出现边缘 `edges` 被平滑，导致细节丢失。而双边滤波器则正是将领域颜色也作为相似性的判断。当像素处在边缘位置，两边像素差异比较大，因此在颜色相似度几乎为0，也就是权重几乎为0，这样就会使得即使两个像素位置非常相近，但是由于颜色差异较大，在进行加权平均去噪时，贡献的分量很少，从而实现边缘的保持。正所谓`保边滤波器`。 一些 `notable uses` 包括 `tone mapping, halo-free sharpening, photographic tone transfer`。

### 公式化描述

双边滤波使用在位置和颜色值附近的像素进行加权平均。对应到大一统的描述中去，对应的值为 `vi=[ri,gi,bi,1]`，对应的位置为 `pi=[xi/σs,yi/σs,ri/σc,gi/σc,bi/σc]`。`σs` 为空间标准差，`σc` 为颜色空间标准差。对于灰度图，只含有灰度值 `luminance`，因此是一个三维的滤波器。也有4-D，6-D的例子。

![](/img/20200411/Figure3.jpg)

![](/img/20200411/Figure4.png)



### 联合双边滤波器

位置 `pi` 来自一张图 A，而值 `pi` 来自另一张图 B，可以按照不跨越图像B中边缘的方式对图像A进行平滑。这个被用来`combining images taken with and without flash` (闪光)。

这种思想被拓展用作 `upsampling` 技术：由低分辨率输入图像决定的位置，使用高分辨率参考图像进行`slicing`，低分辨率的图像可以通过保持高分辨率参考图中边缘的形式内插出来。

### 快速实现

#### FFT加速

离散化intensity，使用FFT进行在每个intensity level执行常规的高斯模糊，之后可以在这些高斯模糊之间进行内插得到灰度双边滤波器。

#### `bilateral grid`

将空间和intensity间都进行离散化，引入 splat-blur-slice的pipeline进行高维度滤波。

![](/img/20200411/Figure5.png)

`Chen et al. [CPD07] implemented it on a GPU for various interactive applications`.

#### 积分图像-积分直方图

发现使用没有必要使用fft，而是使用更加快速的基于积分图的方法来对每个intensity level进行模糊化blur，因而构建积分直方图（`每个位置存储的是从该位置到左上角的矩形区域的像素分布直方图`）。


#### `plane-sweep`：Real-Time O(1) Bilateral Filtering

not explicitly representing the entire space，而是在intensity levels上扫过一个平面，按照intensity的顺序计算输出。 这个是已知最快速的灰度双边滤波器，这种plane-sweep方法不能很好地泛化至更高维度。



## 论文参考

- [Fast High-Dimensional Filtering Using the Permutohedral Lattice](http://graphics.stanford.edu/papers/permutohedral/)


-----
20200411