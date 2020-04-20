# 【医学图像处理】CT成像技术之图像获取（1）：物理设备（机架和检测器）在获取图像中的作用

本系列重点介绍`CT成像技术`，并将介绍用于`获取图像的设备`，`如何形成和显示图像`，`影响图像质量的因素`以及`如何测量剂量`。

本章节重点介绍`CT成像技术之图像获取（1）：物理设备（机架和检测器）在获取图像中的作用`


## 轴向与螺旋扫描

### 轴向扫描

![](/img/20200420/Figure1.png)

#### “一步一步射击”

1. 台架停下来旋转以从单个切片中获取数据

2. X射线关闭

3. 病人移至下一个切片

4. 旋转以从下一个切片获取数据

### 螺旋扫描

![](/img/20200420/Figure2.png)

1. aka螺旋

2. 台架保持旋转，不断释放X射线束。

3. The couch 同时移动；

4. 得到连续的螺旋扫描图案


#### 好处

- 避免在一次呼吸中进行扫描时因不同呼吸而出现呼吸失准

- 由于可以更快地扫描，因此可以更有效地使用造影剂，因为一次扫描可以在多个阶段进行扫描，例如门静脉，血管造影，延迟

- 重叠的切片可以更好地重建并有助于显示较小的病变

- 间距> 1可用于减少扫描时间和/或辐射剂量，并且仍然覆盖相同的体积

现在都以这种方式获取所有图像。

## Pitch

The pitch is the measure of overlap during scanning.就是扫描期间对于重叠的测量。

Pitch = distance couch travels / width of slice

下面为不同的pitch示意图：

![](/img/20200420/Figure3.png)

`Pitch = 20/10 = 2`

![](/img/20200420/Figure4.png)

`Pitch = 10/10 = 1`

![](/img/20200420/Figure5.png)

`Pitch = 5/10 = 0.5`

- pitch数目大于1表示 couch移动距离超过 beam 射线束的宽度，会有空隙；

- pitch数目小于1表示 couch移动距离少于 beam 射线束的宽度，会有重叠。

较高的pitch数目优缺点如下：

- 优点是 较少的放射剂量以及快速扫描

- 缺点是 更加稀疏的采样

## 多切片扫描

现在，我们不再只有一排检测器，而是有多排平行的检测器。然后可以选择某些行的检测器来改变切片厚度以及准直仪。

![](/img/20200420/Figure6.png)

好处是：

- 由于有源探测器总宽度更宽，扫描速度更快

- 更快的扫描速度，更好的动态成像

- slice更薄

- 通过slice实现3D成像

- 同时采集多个切片

## 探测器阵列

有多种探测器类型：

- 线性的
- 自适应的
- 混合阵列的

### 线性阵列

![](/img/20200420/Figure7.png)

探测器所有的行都同宽。

### 自适应阵列

![](/img/20200420/Figure8.png)

中央检测器行中的元素最薄，并且朝向外部变宽。

- 好处：激活尽可能少的检测器元件，仍然可以提供大范围的检测器切片；激活的检测器行越少，意味着将行分开的隔膜越少。这提高了剂量效率。

- 坏处：升级到更多数据通道需要更换昂贵的检测器。


### 混合阵列

![](/img/20200420/Figure9.png)

- 与线性阵列类似，检测器行内的元素的宽度相同。但是，检测器行的中心组比外部行窄。

- 这些是用于16层及以上扫描仪的主要检测器阵列。

## 多层螺距

有两种方法可以计算多层扫描仪中的间距。第一个（间距d）类似于单个切片的间距，仅考虑x射线束的宽度。

`Pitch_d  = 每转床移动的距离/ X射线束的宽度`

但是，这并不能完全代表X射线束的重叠，而是现在使用的是`Pitch_x`。

`Pitch_x =每转沙发床行程/同时获取的切片的总宽度`

这与单层螺旋扫描的间距定义相当，因为总准直宽度类似于单层螺旋扫描的检测器子组宽度。


## 关键点

### Pitch

- Single slice pitch = detector pitch = couch travel per rotation / detector width。单个slice pitch等于探测器的pitch等于每转儿床移动的距离比上探测器宽度。

- Multislice pitch = beam pitch = couch travel per rotation / total width of simultaneously acquired slices。多切片 pitch 等于 射线束pitch等于每转儿床移动距离比上所有同时获取的slices的总宽度。


### Slice thickness

- Single slice CT = determined by collimation. Limited by detector row width. 单个切片的取决于准直器，受探测器行宽限制；

- Multisclice CT = determined by width of detector rows。多层CT的由探测器的行宽确定。

-----
20200420

