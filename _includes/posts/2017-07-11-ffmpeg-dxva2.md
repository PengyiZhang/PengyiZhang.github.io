﻿# 【硬解码】【FFMPEG】采用硬解码加速后显示以及从加速设备中copy图像数据的考量

标签（空格分隔）： 【FFMPEG】 【CUDA开发】

---

> 最主要的问题是在使用CUDA硬解码或者FFMPEG硬解码中加速，总是不可避免的涉及到将硬解码后的图像数据copy到内存中进行相关算法的处理，这个时候问题就来了，硬解码效率比较高，如果直接用DirectX进行显示，几乎没有什么CPU占用率，但是如果将数据从加速设备中拷贝出来则出现了CPU占用率急速飙升。

（1）**ffmpeg只实现了dxva2硬件解码的内容**。我所翻译的第一篇、第二篇文章的那部分内容除了解码部分，都要由用户自己去实现。这一块颇有一点复杂，不过不
用担心，VLC和ffmpeg都有例子可以参考。这一部分的内容需要对以上两篇翻译的内容有所了解才能比较好的理解代码的逻辑。
（2）**要想真正看到硬件加速的效果，解码后的数据不建议再copy到内存中用CPU进行处理**。我一开始就是因为拷贝到吧解码后的数据又copy回内存导致不仅gpu的
使用率看不到明显变化，而且CPU的使用率相对于不使用dxva2反而提高了。后来我修改为把解码后的数据直接显示出来，GPU使用率一下子就上去了，CPU使用率也
降下来了。

所以，最好的解决方案是直接在GPU中部署相关的算法，根据这个数据流串起来，最后还是使用DShow或者OpenGL进行渲染。数据进入CPU后就会严重影响硬解码的效率。


--------
2017-07-11 11:29
张朋艺 pyZhangBIT2010@126.com