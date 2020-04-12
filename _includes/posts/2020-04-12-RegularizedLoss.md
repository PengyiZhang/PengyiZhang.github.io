# 【计算机视觉】关于`Regularized Loss`用于弱监督语义分割中的说明

## 弱监督标签

以两语义分割为例，背景和前景，给定的弱监督标签是只对前景个一小部分进行了标注。这个只是直接拿这个弱监督标签进行训练，会有一定的问题，因为大部分的前景标签都没有标注出来，所以前景类别会受到较大的抑制。


##




其余方法可以尝试 弱监督语义分割的相关技术，比如：[https://github.com/PengyiZhang/MIADeepSSL/tree/master/01_WeaklySupervised](https://github.com/PengyiZhang/MIADeepSSL/tree/master/01_WeaklySupervised)



-----
20200412