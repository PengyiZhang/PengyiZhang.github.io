# 【计算机视觉】局部图像特征local iamge features

标签（空格分隔）： 【图像处理】 【Matlab开发】

---

说明：本系列是整理课程计算机视觉（英语）的笔记所成，不对之处还望指出。


----------

上一节课主要讲述了图像局部特征中的边缘信息，边缘主要发生在像素梯度处，因此通过特种梯度模板进行梯度计算，然后提取边缘。但是记住了，噪声点也是像素变化或者破坏局部性的一个因素，因此，噪声点的位置也是对应着梯度的位置，因此，进行边缘提取，很重要的一步就是进行噪声的抑制。通常canny算子是一个比较好的边缘提取方法，具体可以参看上一篇笔记中的论述，先进行平滑滤波，然后进行梯度计算，之后进行非极大值抑制，最后再通过双阈值来边缘的提取和生长。

##边缘信息

先再重复一下边缘，edge，总是出现在梯度大的地方，但是噪声也是出现梯度大的地方。所以重视平滑滤波。
整个思路是：图像像素的局部性（pixel趋向于等同于它的neighbors），受到破坏的地方认为是边缘，然后考虑到噪声的因素，所以平滑滤波。

关于边缘信息有以下的缺点：
edge maps depend on shading说得是如果图像变得更加brighter，或者是darker的话（可能由于camera gain 变大或变小，更多的光照，像素值乘以了一个常数等），梯度的幅度就会变大或变小，因此，scaling image brightness changes the edge map（因为一些幅度会超过test threshold）；同一个图片的brighter/darker的复制品的edge maps也不同。


----------

##orientations

虽然梯度的幅度收到光照变化的影响，**但是梯度的方向却不受影响**；所以，注意到梯度的方向可能更好，对于求取边缘特征。不同的尺度下进行梯度计算会影响到orientation field，不同的pattern具有不同的orientation histograms。那么如何build orientation representation呢？

我们想在一个image patch里面represent a pattern，用来检测、匹配点等，它需要的特点是我们必须知道描述的patch是哪一个，知道它的center和size。
那么，现在假定已经知道了上面的条件：
那么期望的features是啥呢？

    - representation 不发生太大的改变，当center稍微错误的话；
    - representation 不发生太大的改变，当size稍微错误的话；
    - representation 是distinctive的；
    - representation 不发生太大改变，当patch变得brighter或者darker的话；
    - 大的gradients比小的gradients个重要；

对于前两个可以使用histograms来做，因为与center和size不太关系；对于第四个，用orientation来做，因为上面说了梯度的方向与光照不关系；对于第三个，可以讲window内部再打碎成boxes，分别描述；对于第五个，可以使用加权方向直方图。

##**Histograms of oriented gradients： HOG features**
策略是：将patch打碎成blocks，在blocks内部构造histogram representing gradient orientations（当patch稍微move的话，不会改变太多，通过幅度大小加权）

crucial points：

    - 首先是gradient orientations不受intensity的影响；
    - 然后是具有更大值得orientations更加重要；
    - 最后是描述了一个已知大小和center的image window。


--------

笔记整理