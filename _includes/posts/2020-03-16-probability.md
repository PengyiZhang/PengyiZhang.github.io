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

# 【计算机视觉】Introduction to probability

标签（空格分隔）： 【图像处理】

---
说明：本系列文档是依据Simon J.D. Prince的2012年的著作《Computer vision: models, learning and inference》提供的slice撰写而成，主要讲重点大纲简明扼要的列出来，帮助知识的归纳理解。中间可能插入别的临时需要先掌握的内容，以标号001开始。


----------

##关于随机变量的理解

- 表示不确定的量；
- 可能是实验结果，或者现实实际中一次测量；
- 观察多次结果不一样；
- 有些值出现的多，且这些信息可以通过概率分布捕获。

离散的、连续的，联合概率密度、边缘概率分布（对不需要的随机变量进行积分可得）、条件概率

##关于Bayes' Rule的一些术语

- likelihood：指的是给定y的某一值得情况下，观察x某一个的值，P(x|y)
- Prior：在观察x之前，我们对y了解些什么，P(y)
- Posterior：在观察了x之后，我们对y了解些什么，P(y|X)
- Evidence：就是贝叶斯公式下面那个积分，是一个常数，为了保证左边是一个有效的分布。

**关于独立，说得是知道了x，对了解y没有任何帮助。**

如果独立了，联合概率密度可通过边缘概率密度的乘积得到。
关于期望，以x某个分布下f(x)的平均值；期望的一般情况还有：mean，$k^{th}$ moment about zero（k阶原点矩），$k^{th}$ moment about mean（k阶中心矩），variance，skew（k=3偏度），kurtosis（k=4峰度），covariance of x and y（协方差）

------

笔记整理 张朋艺

