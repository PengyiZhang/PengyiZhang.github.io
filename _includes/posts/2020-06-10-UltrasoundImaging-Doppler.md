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

# 【医学图像处理】超声成像之多普勒


## 摘要

- 多普勒
- - 多普勒效应：

- - - 流量消失=回流频率降低
- - - 流向=返回频率的增加
- - 连续波多普勒

- - - 独立的发射和接收元件，用于连续测量流量，需要专用探头
- - - 无法区分两种结构
- - - 无法确定深度
- - 脉冲多普勒

- - - 可以显示在B模式的上方（称为双工超声）
- - - 彩色多普勒：由色标表示的速度平均值
- - - 功率多普勒：显示的是速度幅度，而不是方向
- - - 频谱多普勒：随时间变化的速度范围。电阻指数和脉动指数，用于计算高电阻容器和低电阻容器
- 伪影
- - 由于PRF太低而造成的混淆
- - 由于湍流和速度在容器中心更快，光谱展宽
- - 多普勒角度必须小于60°才能准确估算
- - 壁式过滤器可能会消除真正的低速

![](/img/20200610/Figure01.png)

当声音从运动对象（例如血细胞）反射时，返回的回声与原始声源的频率不同，并且频率的变化量与界面速度成正比。

- 如果物体远离光源，则频率会降低。
- 如果物体朝着光源移动，则频率会增加。

随着发射器与界面之间的角度（声入角）接近90°，界面速度估算的准确性降低。通常使用小于60°的声入角来准确估计速度。

## 连续波多普勒

这些通常是专用的手持设备（例如ABPI，胎儿心电波的心电图）。当多普勒频移在可听声音频率范围内时，多普勒效应会作为可听声音发出：音调越高，速度越大；声音越刺耳，气流越湍急。当它们连续发送（并因此接收）时，它们必须包含两个单独的发送和接收元素。

- 优点
- - 便宜的
- - 易于使用
- - 对流量敏感

- 缺点
- - 无法测量速度
- - 使光束路径中的所有血管发出声波，直到光束衰减为止。这意味着，由于动脉和静脉通常并列在一起，因此输出通常会结合动静脉信号。
- - 无法确定深度

## 脉冲多普勒

在脉冲多普勒仪中，相同的元件用于发射和接收，并且发射短暂的超声能量脉冲。范围门控仅用于接受从特定深度返回的回波。双工涉及叠加在B模式成像上的多普勒成像。

超声仪中使用三种类型的脉冲多普勒：

- 颜色
- 功率
- 光谱

### 彩色多普勒


![](/img/20200610/Figure02.jpg)

在彩色多普勒仪中，设置采样量，并计算移动结构速度的均值和方差。然后，该速度由范围从负（远离换能器）到零（没有计算出的速度）到正（朝着换能器）的任意颜色的比例表示。脉冲帧频会影响实时彩色多普勒测量。较低的帧速率会导致彩色多普勒信号不稳定，例如使用较大的多普勒采样盒，这需要更多的多普勒脉冲，因此会降低帧速率。

### 功率多普勒

功率多普勒图像仅映射多普勒信号的幅度 ，而没有任何速度指示。所有运动，无论相位如何，都对振幅有所贡献。这意味着功率多普勒强调血流量。


- 优点
- - 较少依赖于声纳角
- - 可以显示非常低的流量
- - 不受锯齿

- 缺点
- - 无流向指示
- - 组织运动产生假象

### 频谱多普勒

频谱多普勒显示随时间推移返回的多普勒频率范围并以超声图显示。
容器壁电阻的差异会产生不同的光谱轨迹。血管壁的特性可以用电阻指数（RI）和脉动指数（PI）来表示。

RI = (收缩期峰值频率-舒张末期频率)/收缩期峰值频率
PI = (收缩期峰值频率-最小频率)/时间平均最大频率

#### 高阻容器
搏动性强，上冲锋利，速度范围狭窄，例如股动脉和主动脉等外周血管。 
#### 低阻力动脉
低脉搏，具有大范围的速度，例如在供应重要器官的血管中，即使在舒张期也是如此，例如肾动脉，颈内动脉。

正常RI = 0.6-0.7

异常RI = 0.8-1.0

## 伪影

### 混叠

![](/img/20200610/Figure03.png)

奈奎斯特极限规定采样频率必须大于输入信号最高频率的两倍，以便能够准确表示图像。

奈奎斯特极限= PRF / 2

如果流动速度大于奈奎斯特极限，则多普勒频移超过刻度并且发生“环绕”。

### 光谱展宽

靠近血管壁内部流动的血液比血管中部的流动慢。在特定的时间范围内，这种较大的频率范围会导致频谱图变宽以及彩色多普勒仪中的不同颜色。这在湍流（例如狭窄的血管）中也会发生，因为湍流会产生不同速度和方向的流动。

### 多普勒角度

流速估计要求流动沿超声波束的方向。如果它是垂直的，即横穿光束，则很难检测到流量。声波角度应始终小于60°，以实现最准确的速度估算。

### 壁式过滤器

电子滤波器应用于返回的数据，以消除低频信号，因为这些信号通常是由低速结构（例如血管壁）产生的。如果过滤器使用不当，则来自低速血流的真实信号将被消除。


-----
20200610
