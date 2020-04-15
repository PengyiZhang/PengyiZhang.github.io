<head>
    <script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>
    <script type="text/x-mathjax-config">
        MathJax.Hub.Config({
            tex2jax: {
            skipTags: ['script', 'noscript', 'style', 'textarea', 'pre'],
            inlineMath: [['$','$']]
            }
        });
    </script>
</head>

# 【计算机视觉】如何用传统方法在实现目标检测？

## 问题描述

深度学习虽然很火热，但是数据量不大的时候，深度学习的性能还是比较差的，最主要的是模型能力太强，学出来之后不能泛化，因此一种可选的方法就是 `不要处处都是深度模型，试试返璞归真的传统方法`。

## 传统方法

基本上传统机器方法最直观的就是分类、回归，而对于计算机视觉中级任务：`目标检测` 该如何下手呢？一种最直观的想法就是用滑动窗口 `sliding window` 的方法，对滑动窗口内的图像块儿进行分类，根据滑动窗口的位置得到检测结果。那么对于深度模型，常常是端到端的，采用 `CNN` 进行 深度特征的提取，R-CNN系列则采用 `Selective Search`、`Region Proposal Network`作为候选样本的进行采集，同时加上了位置的回归，深度模型剩余的部分实际上也是进行的分类。整个解决思路并没有太大的冲突。

所以，传统方法解决方案包括三个步骤，一是选择合适的图像块儿 `image patch` 特征描述子，二是选择合适的分类器，三是用提取的特征及提供的标签对分类器进行训练。

参考 [https://www.pyimagesearch.com/2014/11/10/histogram-oriented-gradients-object-detection/](https://www.pyimagesearch.com/2014/11/10/histogram-oriented-gradients-object-detection/) 中传统方法进行目标检测共有六个步骤，这里摘录一睹为快：

（1） 从训练样本中抽取正样本 (包含目标的图像块儿) `P` 个，并提取其 `HOG` 特征；

（2） 从训练样本中抽取负样本（不包含目标的图像块儿）`N` 个，并提取其`HOG` 特征，N >> P;

（3） 在正负样本集上训练线性支撑向量机；

（4） 实施 `hard-negative mining`：对负样本训练集中的每个图像及其可能的尺度图像，用滑动窗口的技术在图像上滑动，对每个窗口的图像块儿提取HOG特征，并用应用训练过的SVM。如果分类器将负样本识别为目标，那么记录该特征矢量及其关联的假阳性图像块儿和对应的概率；

（5） 使用分类器输出的概率对这些假阳性的样本进行排序，并用这些假阳性的样本对分类器进行再次训练。可以迭代使用（4）和（5）多次，但是在实际中只使用一次就似乎是足够的了；

（6） 对训练好的模型在测试集上进行测试。在测试集的每张图像及其可能的尺度图像上，应用滑动窗口，提取HOG特征，用分类器分类，当输出结果的概率足够大，记录滑动窗口的位置。当扫描完图像，应用非极大值抑制方法剔除重复的检测。

![](/img/Figure1.gif)

最为典型的两个传统检测方法：

- [deformable parts model](http://cs.brown.edu/~pff/papers/lsvm-pami.pdf)  
- [Exemplar SVMs](http://www.cs.cmu.edu/~efros/exemplarsvm-iccv11.pdf)



下面我主要按照三个步骤的描述进行介绍：

### 图像块儿 `image patch` 特征描述子

直接采用 `Raw Features` 即RGB像素 进行重排，构成一维度的特征矢量，这样的方法太朴素，没有发挥传统方法中精妙的手工特征设计的优势，所以，这里不对这些比较朴素的特征进行介绍。


#### 什么是特征描述子 `feature descriptor`？

这里面介绍的非常到位：
[https://www.analyticsvidhya.com/blog/2019/09/feature-engineering-images-introduction-hog-feature-descriptor/](https://www.analyticsvidhya.com/blog/2019/09/feature-engineering-images-introduction-hog-feature-descriptor/)

摘录一二：
- 特征描述子做了什么：`It is a simplified representation of the image that contains only the most important information about the image.`

- 最常用的特征描述子：

    HOG: Histogram of Oriented Gradients
    SIFT: Scale Invariant Feature Transform
    SURF: Speeded-Up Robust Feature


#### 方向梯度直方图 `Histogram of Oriented Gradients(HOG)` 

- HOG描述子聚焦的是目标的结构或者形状，与单纯的判断像素是否属于边缘edge不同，HOG能够通过抽取边缘的梯度和方向（或者幅度和方向）来提供边缘的方向；

- 这些 `orientation` 是在局部提取去的，这意味着需要将完整的图像打碎成更小的区域，对每个区域进行梯度和方向的计算；

- 最后会对每个这些区域单独计算一个像素值的梯度和方向的直方图，故而被称之为方向梯度直方图 HOG。

`The HOG feature descriptor counts the occurrences of gradient orientation in localized portions of an image.`

##### 直观感受

先直观看一下对应MATLAB版本的HOG提取方法及可视化结果：

```matlab
img = imread('cameraman.tif');
[featureVector,hogVisualization] = extractHOGFeatures(img);

figure;
imshow(img); 
hold on;
plot(hogVisualization);
```

![](/img/Figure2.png)



##### 计算过程

###### 第一步：计算`x,y`方向梯度

![](/img/Figure3.png)

类似用sobel算子进行的梯度计算：

![](/img/Figure4.gif)

`Change in X direction(Gx) = 89 – 78 = 11`

`Change in Y direction(Gy) = 68 – 56 = 8`

###### 第二步：计算梯度的幅度和方向

![](/img/Figure5.png)

`Total Gradient Magnitude =  √[(Gx)2+(Gy)2]` ==> `Total Gradient Magnitude =  √[(11)2+(8)2] = 13.6`

`tan(Φ) = Gy / Gx` ==> `Φ = atan(Gy / Gx)`

对每个像素都这样计算梯度的方向和幅度。

###### 第三步：计算梯度的直方图

####### 几种利用梯度幅度和方向计算直方图的方法

方法一：以梯度方向从1-180°，`bin=1`：

![](/img/Figure6.png)

方法二：以梯度方向，`bin=20`

![](/img/Figure7.png)


方法三：上面两种均没有将梯度值考虑在内，这次不去统计梯度方向的次数，而是将对应方向的梯度值加进去

![](/img/Figure8.png)


方法四：在方法三的基础上，考虑到一个梯度方向可能刚好跨进两边，那么就计算一个根据距离的加权的幅度值：

![](/img/Figure9.png)

这就是HOG特征描述子如何计算梯度方向直方图的。

####### 在8x8的cells内计算9x1的梯度直方图

梯度直方图不是在整幅图像上计算的，而是先把图像分解为8x8的cells，然后带方向的梯度的直方图分别在每个cell中进行统计。


![](/img/Figure10.png)

###### 第四步：在16x16的cell中归一化梯度

因为图像的梯度对于 `overall lighting` 十分敏感，图像中的一些部分与其它部分相比会比较明亮。虽然不能完全消除这种影响，但是能够通过归一化梯度减少光照变化的影响。然后以16x16的大小为block来进行。

![](/img/Figure11.png)

所以一个block中有4个cells，从而构成了36x1的直方图。进行归一化即可。

###### 第五步：构建完整图像的特征

将所有的blocks合并在一起构成了整幅图像的梯度直方图。以上面64x128的图像为例，共分成了7x15=105个block，那么105x36x1=3780个features。


```python
from skimage.feature import hog
from skimage import io
im = io.imread('./timg.jpg',as_grey=True)
normalised_blocks, hog_image = hog(im, orientations=9, pixels_per_cell=(8, 8), cells_per_block=(8, 8), visualise=True)
io.imshow(hog_image)
```

#### 方向梯度直方图 `Histogram of Oriented Gradients(HOG)` 




### 传统模型

#### `Support Vector Machine`

![](/img/20200414/Figure10.png)

这张图足以说明支撑向量的含义，通过拉格朗日乘子法进行求解，一些原理性的介绍可以参看 [https://docs.opencv.org/3.4/d1/d73/tutorial_introduction_to_svm.html](https://docs.opencv.org/3.4/d1/d73/tutorial_introduction_to_svm.html)

#### `Random Forest`

随机深林是由一堆决策树 `decision trees` 构建的，决策树类似下面的图示：

![](/img/20200414/Figure11.png)

而随机深林中每个决策树都进行类别预测，因为构建随机深林的决策树尽可能的是不相关的模型，构成一个投票委员会 `committee`，从而实现互补增强，是属于 `Bagging`的效果。

![](/img/20200414/Figure12.png)


一些原理性的介绍可以参看 [https://towardsdatascience.com/understanding-random-forest-58381e0602d2](https://towardsdatascience.com/understanding-random-forest-58381e0602d2)



#### `Gradient Boosting Classifier`

`Boosting` 就是一堆串行的模型集成起来构建鲁棒的模型。与上面介绍的随机森林不同的是，`Boosting` 采用的是串行过滤的方式。在刚开始做跟踪的时候接触的AdaBoost方法，以及基于Haar-like特征构建朴素贝叶斯分类器的方法，基本上都算是此类。主要是数据经过一个个串行集成的弱分类器，最终比较强的分类器。通常的构成也是决策树，跟随机深林不同的是Gradient Boosting是串行决策，而Random Forest是并行决策，下面的图示可以很好的说明两者的区别：


![](/img/20200414/Figure13.png)

![](/img/20200414/Figure14.png)

一些细节可以参考：[https://medium.com/mlreview/gradient-boosting-from-scratch-1e317ae4587d](https://medium.com/mlreview/gradient-boosting-from-scratch-1e317ae4587d)



## 基于传统方法进行语义分割例程说明

### demo任务描述

原始图像：

![](/img/20200414/Figure15.jpg)

分割GT掩摸：

![](/img/20200414/Figure16.png)


### 选择合适的像素特征描述子

这里直接选用上面介绍的几种特征构建用于训练分类器的特征，包括[Raw features: R\G\B]\[LBP texture features: LBP]\[Haralick texture features: Angular second moment\Contrast\Correlation\Sum of Square: variance\Inverse difference moment\Sum average\Sum variance\Sum entropy\Entropy]，共构成13维的特征矢量。

#### 提取 `LBP texture features`

需要提供灰度图，设置邻域半径`lbp_radius`以及点数`lbp_points`：

![](/img/20200414/Figure17.png)

```python
from skimage import feature
lbp_radius = 24 # local binary pattern neighbourhood
lbp_points = lbp_radius*8
lbp = feature.local_binary_pattern(img, lbp_points, lbp_radius)
```

#### 提取 `Haralick texture features`

需要提供灰度图，以及设置窗口大小。这里选择设置计算共生矩阵的distance的默认值为1，上面介绍的四个角度。而对应的ROI即为提取好的图像块儿，也就是滑动窗口抠出来的。最后计算它四个方向上的均值即可。

```
import mahotas as mt

feature_vec = []

texture_features = mt.features.haralick(roi)
mean_ht = texture_features.mean(axis=0)

[feature_vec.append(i) for i in mean_ht[0:9]]

```

### 选择合适的分类器

#### SVM 

```python
from sklearn.svm import SVC
model = SVC()
```

#### Random Forest 

```python
from sklearn.ensemble import RandomForestClassifier

print ('[INFO] Training Random Forest model.')

model = RandomForestClassifier(n_estimators=250, max_depth=12, random_state=42)

```


#### Gradient Boosting Classifier 

```python
from sklearn.ensemble import GradientBoostingClassifier
        model = GradientBoostingClassifier(n_estimators=100, learning_rate=1.0, max_depth=1, random_state=0)
    
```


### 用提取的特征及提供的标签对分类器进行训练

首先填入 `Raw Features`：
```python
feature_img = np.zeros((img.shape[0],img.shape[1],4))
feature_img[:,:,:3] = img
```

其次填入 `LBP texture features`:

```python
feature_img[:,:,3] = create_binary_pattern(img_gray, lbp_points, lbp_radius)
```

最后填入 `Haralick texture features`：

```python
features = feature_img.reshape(feature_img.shape[0]*feature_img.shape[1], feature_img.shape[2])

h_features = harlick_features(img_gray, h_neigh, ss_idx)

features = np.hstack((features, h_features))
```

因为图像分辨率为`600x600`，特征数目比较多，因此，这里直接从大概 `600x600` 个像素中随机选择`1000`点进行训练，而由于计算 `Haralick texture features` 比较慢，所以对选出来的1000个像素对应的位置才做 `Haralick texture features` 的计算。

最后得到了1000x13的训练样本的特征矢量。直接调用：

```python
model.fit(X, y)
```

### 训练结果展示

![](/img/20200414/Figure18.jpg)


## 参考文档

[https://github.com/cerr/CERR/wiki/GLCM_global_features](https://github.com/cerr/CERR/wiki/GLCM_global_features)

[https://github.com/cerr/CERR/wiki/Haralick-texture-features](https://github.com/cerr/CERR/wiki/Haralick-texture-features)





-----
20200414

