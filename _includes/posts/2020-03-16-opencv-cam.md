# 【计算机视觉】opencv读取多个摄像头

标签（空格分隔）： 【图像处理】

---
说明：今天蹭了机器视觉课程，讲到了stereopsis，立体视觉，讲到了关于通过多个摄像头获取object的depth信息的事情，因为想到从来没有试过打开多个摄像头进行过处理，这次进行了测试，这里小小记录一下。


----------

opencv提供的VideoCapture可以很方便的打开视频、摄像头设备，而且直接输入对应的摄像头标号即可，或者视频的名字即可，一个open全部搞定，我通常喜欢用这个来类来获取图像数据。
下面是对应的详细代码：

    // MultiCAM.cpp : 定义控制台应用程序的入口点。
    //

    #include "stdafx.h"

    #include <opencv2/core/core.hpp>
    #include <opencv2/highgui/highgui.hpp>
    #include <opencv/cv.h>
    using namespace std;
    using namespace cv;

    int main(int argc, _TCHAR* argv[])
    {
    VideoCapture capture, capture1, capture2;
    capture.open(0);capture1.open(1);capture2.open(2);
    // Read options
    //read_options(argc, argv, capture);
    // Init camera
    if (!capture.isOpened())
    {
        cout << "capture device 0 failed to open!" << endl;
        return 1;
    }
    if (!capture1.isOpened())
    {
        cout << "capture device 1 failed to open!" << endl;
        return 1;
    }
    if (!capture2.isOpened())
    {
        cout << "capture device 2 failed to open!" << endl;
        return 1;
    }
    // Register mouse callback to draw the tracking box
    namedWindow("0", CV_WINDOW_AUTOSIZE);
    namedWindow("1", CV_WINDOW_AUTOSIZE);
    namedWindow("2", CV_WINDOW_AUTOSIZE);
    Mat frame;
    Mat frame1;
    Mat frame2;
    while(capture.read(frame) && capture1.read(frame1) && capture2.read(frame2))
    {
        imshow("0", frame);
        imshow("1", frame1);
        imshow("2", frame2);
        if (cvWaitKey(33) == 'q') {	break; }
    }

    waitKey();
    return 0;
    }


笔记本上自带的WEBCAM，外接了两个USB CAM，共三个摄像头，一般来说笔记本自带的WEB CAM应该对应的标号为0，之后是哪个USB接上的标号为1，以此类推。

可以在此工程上借助两个摄像头或者3个摄像头获取目标的深度信息的测试。





