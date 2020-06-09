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

# 【医学图像处理】超声成像之超声伪影

图像形成假定：

- 声音直线传播
- 以恒定的速度
- 衰减均匀
- 每个接口仅反映一次
- 当回声不能以这种方式运行并且系统对它进行错误解释时，就会产生伪影。

## 声学增强

![](/img/20200611/Figure01.png)

充满流体的结构衰减较弱，较大比例和更大振幅的光束穿过后部区域的结构。机器将其解释为声反射的增加，并且这些结构在图像上显得更亮。

## 声影

![](/img/20200611/Figure02.png)

坚硬的钙质物质和柔软的组织-空气界面几乎反射了所有声波。没有从结构后面的区域收到任何信息。

## 混响
![](/img/20200611/Figure03.png)

在换能器表面与靠近表面的相对强烈反射的界面之间来回多次反射会产生一系列延迟的回波。这些看起来像是在充满流体的结构中的条纹。

存在两种类型的混响伪像：

- 彗尾：来自金属或钙化物体
- 敲响：来自气泡的集合

## 反射/镜面人工伪影

![](/img/20200611/Figure04.png)

声音从强烈反射的物体上反弹，该物体充当镜子并将脉冲反射到另一个组织界面。图像的解释是第二界面超出了第一表面，非常类似于混响伪像。这最经常发生在隔膜处，因为声波从隔膜反射回来，在胸腔中可以看到肝脏。

-----
20200611

