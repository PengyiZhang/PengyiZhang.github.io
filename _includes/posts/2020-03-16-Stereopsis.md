# 【计算机视觉】Stereopsis立体视觉

标签（空格分隔）： 【图像处理】

---

说明：本系列是整理课程计算机视觉（英语）的笔记所成，不对之处还望指出。


----------
##**单目视觉与双目视觉**

single camera，是没法获取图像的深度信息的（depth），对于在同一条光线上（line of sight）的目标都将被映射为象平面上（image plane）的同一个点。那么如何获取depth信息呢？想一想我们的双眼，实际上单眼是无法目标或者图像的depth信息，但是由于人对这个世界、环境有了足够的knowledges，那么结合着这些knowledge，是可以实现单眼获取depth information的。但是通常是需要two or more cameras，通过三角测量法，还记得在郭老师的机器人的客场上读过的两篇文献吗？就是通过多个雷达还是声纳来获取目标的三维位置信息，这里的道理实际上是一致的。

two cameras：通过fusing the pictures，找到两幅图中对应的点。利用视差，difference or disparity来获取一个较强的depth感知。

而stereopsis 就是设计和实现这样功能的算法来执行depth感知任务。在visual robot navigation，cartography 地图制作，空中侦察（aerial reconnaissance）、close-range photogrammetry近距照相测量术等领域非常重要。

##**stereo version**

involve 两个过程：

    - 融合两个或多个摄像机观测到的特征，也就是在两幅图片中找到同一个点的对应；
    - 重建3D原象（原来3D的是作为映射的原象，而生成的两幅图片都是2D内的象，通过两个或多个2D内的象，是有可能恢复出3D原象的）

###**如何找到对应的点呢？**

一个非常直观的想法就是在image 1中的位置处，设置一定的domain，2D的，进行逐个搜索，这样做很显然计算资源浪费的比较多。但是实际上根据双目视觉的体系，是有一个叫做epipolar constraint的约束来帮助我们快速找到在image 2中的匹配点。

###**Epipolar constraint**

实际上很简单，P1，P2，P3对于camera 1是在同一条line of sight上，因此，成像点在camera 1的相平面上是一个同一个点。这也是为何单目视觉无法完成对depth的信息的获取的原因。但是对于另一个camera 2，就不同了，P1，P2，P3在camera 2的相平面上是构成同一条直线的，以3D空间为例，O1是camera 1的位置，TT1是camera 1的相平面，O2是camera 2的位置，TT2是camera 2的相平面，空间内的一个目标为P，PO1O2构成的平面与两个camera的相平面TT1和TT2都有一个交线，这两个交线就叫做epipolar line，那么已知camera 1相平面上的一个像素点，要求在另一个camera的相平面上对应的像素点，进行匹配的时候，如果能够得到epipolar line的话，就可以直接在epipolar line上进行search and match。

###如何计算epipolar line呢？

实际上对于在不同位置的camera，它们所成象的之间的空间几何的关系，实际上就是坐标系的变化，比如transportation移动、rotation等。使用矩阵是可以求解的。由此，可以建立两幅图像上的某个像素的对应关系。最后可以得到一个叫做fundamental matrix的矩阵，满足$p^TFp^{'}=0$，这样就可以像解方程一样，如果已知了几个对应点，那么根据这些列出8个方程组，就可以顺利求解F出来，从而利用F计算出epiploar line。这样就只用搜索epipolar line上的像素点就可以了。如果找的对于点是N个，N大于8了，方程组是一个overfit的超定方程，可以使用SVD求解最小二乘方法来做。

###**disparity and depth**

想一想，如果epipolar line，现在叫做scanline，是斜着的，那么这条线可能过多个难以取舍的像素点，访问起来似乎也不方便，所以通常会将双目摄像头放置在同一个depth下，这样的O1O2构成的baseline，就与相平面完全平行了。便于后续的计算。
disparity就是同一个物体，其像素点在两个像平面内的坐标位置差（通常设定为到左边界的距离差，x coordinate）；因此，可以有disparity map，encoded with grey scale image（closer points are brighter）

主要是因为，当points（P）是closer to the camera的话，disparity 就会更大，而距离camera远的点，disparity就会小。所以，disparity map就是代表depth的图。

###**双目重建binocular reconstruction**

三边测量法：由两个camera O1，O2发出的射线经左右相平面上的对应的像素点后的交叉点是可以计算的。但是实际上由于校准的误差等会导致并不交叉，而解决的方案就是找到space point距离两条射线都最小的位置。

###**图像矫正image rectification**

就如前面所介绍的，为了使得寻找corresponding points更加方便，可以将对应的点限制在相同的image scanline上。就是让双目摄像头平行放置。（双目测距，我以前居然并不太知道）

###**stereo correspondence立体匹配**

目的是在stereo pari中找到相对应的点；有一些挑战是：specular surface（镜像），perspective distortions（前景畸变）；Uniform/ambiguous regions，repetitive/ambiguous patterns；

**限制条件是：**

    - uniqueness：最多有一个匹配点；
    - smoothness：disparity is 典型的smooth function
    - ordering ：有序，相应的点的顺序应当保持不变；

**最简单的local approach：**
在epipolar line上进行扫描，匹配。匹配的消耗是基于像素值的差绝对值，disparity computation：winner takes all。

**基于window的local approach：**

matching cost变为了整个window内的像素匹配，计算的方法肯定是各种距离吧：
SAD、SSD，SAD指的是window内的所有像素差绝对值和，而SSD是差平方和;当然还有normalized cross correlation。当然，很明显，基于window的方法要比基于pixel的方法要好。

**global approaches：**

局部的方法，忽略了限制条件，而global方法则将这个问题表述为最小化一个能量函数，将ordering，smoothness constrains嵌入了进去。

有dynamic programming，scanline optimization，graph cuts，belief propagation等方法。
当然使用更多的camera效果可定更好。


【增补】

主要讲了active sensors，因为前面所讲的利用多个摄像头来计算disparity获取depth的方法都是属于被动的。今天说的是主动的。

比如，我们要测量的某个plane上找到对应的匹配点，因为这个plane上的这个区域内的点像素都是相近的，因此很难通过主动的方法获得对应的匹配点，也就无法很好的获取depth信息。这个时候一个非常主动的方法是既然这些附近的像素都一样，我能不能通过映射一些light patterns on the scene，make the same pixel different，而这些light patterns呢，有比较好计算，同时区别还足够大。这就是active sensors的思想。通常light patterns中主动project的light是invisible，因为避免人的反感，所以，通常采用infrared light。

今天说的active sensor有两个：一个是microsoft kinect，一个是intel RealSense。两者的机理相似。
首先说Kinect，是对light coding，意思是在object上form different patterns at different distance。比如简单的说是在1米的位置上pattern是三角形，在2米的位置上pattern是矩形，等等。总之是直接获取depth信息，而不是像被动方式那样。当然使用前需要进行校准calibration。time of flight。比如在体感游戏的时候，它可以获取人体的skeleton，从而实现各种体感控制，

然后是intel的RealSense，英特尔的实感3D摄像头，比较小，可近距离使用，也是通过主动红外获取depth。可用于距离测量，3D扫描，gestures control等。




----

笔记整理