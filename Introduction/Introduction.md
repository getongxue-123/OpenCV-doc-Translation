OpenCV（开源机器视觉库：[http://opencv.org](http://opencv.org ""))是一个基于BSD协议的开源库，它包含了上百种机器视觉算法。此教程介绍了OpenCV 2.x的API使用，提供了C++风格的API，而不是OpenCV 1.x中C语言风格的API（从2.4版本开始，C语言版本的API已经被弃用。并且这些代码没有用C编译器进行测试）。

OpenCV提供了模块化的结构，是由许多动态库或静态库组成的。OpenCV包含了以下子库：

* 核心库（core） ：压缩的模块，定义了基本的数据结构，其中包括高纬度的数组Mat和提供给其他内部库使用的基本函数。

* 图像处理库 （imgproc） ：包含了线性和非线性图形滤镜、图形的几何变换（放缩、放射、透视、基于通用表的重映射），色彩空间转换，直方图等功能的图像处理库。

* 视频分析（Video）：包含了运动预测，背景提取和物体轨迹描述算法的模块。

* 摄像机校准和3D重建（calib3d）:基本多视图几何算法，简单的立体相机校准，物体形态预测，立体匹配算法和3d重建元素绘制。

* 2D特征框架（features 2d）：突出特征检测装置、描述器和特征描述匹配。

* 物体检测（objdetect）：对物体和预先设置的类（如脸，眼睛，大杯子，人，车等）进行检测。

* 高级图形用户界面（highgui）：对用户非常友好的简单界面设计程序

* 视频I/O：提供简单易用的视频捕捉和视频解析接口

* 另外，还有一些帮助模块，例如FLANN和Google test封装接口，Python绑定模块和其他接口。

以后更为深入的章节将会详细介绍各类模块的详细用法。但是，在此之前，我们应该对OpenCV库中大多数API的用法有所了解。

API风格

cv命名空间

所有的OpenCV类和函数都放置在cv命名空间下。因此，如果要使用这些函数，要确保使用cv::标识符或者在使用前声明命名空间using namespace cv；

直接使用：

```
#include "opencv2/core.hpp"
...
cv::Mat H = cv::findHomography(points1, points2, cv::RANSAC, 5);
...
```

或者

```
#include "opencv2/core.hpp"
using namespace cv;
...
Mat H = findHomography(points1, points2, RANSAC, 5 );
...
```

由于不能保证目前或者未来某个时刻OpenCV的外部函数名会与STL或者是其它库发生冲突，尽可能使用显式命名空间标识符来解决命名空间冲突问题：

```
Mat a(100, 100, CV_32F);
randu(a, Scalar::all(1), Scalar::all(std::rand()));
cv::log(a, a);
a /= std::log(2.);
```

自动内存分配管理

OpenCV可以自动处理所有的内存分配问题

首先，std::vector，cv::Mat和其它在函数和方法中使用的数据结构都有析构函数，用于在需要的时候回收占用的内存缓冲区。这就意味着析构函数在Mat中不会总是去尝试回收缓冲区。它是用来表示可能的数据共享量。析构函数会每次执行之后在matrix数据缓冲区中将对应的引用计数减一。缓冲区在且仅在共享缓冲计数变为零的时候才会被回收。缓冲计数变为零表示没有其他的数据结构引用这个缓冲区域。同样的，当一个Mat实例被复制之后，并没有实际数据被复制。而是通过引用计数加一的方式来表示有其他的数据结构在使用同一个缓冲区。我们可以使用Mat::clone方法来创建一个Matrix数据的深拷贝。请查看下面的例子：

```
// create a big 8Mb matrix
Mat A(1000, 1000, CV_64F);
// create another header for the same matrix;
// this is an instant operation, regardless of the matrix size.
Mat B = A;
// create another header for the 3-rd row of A; no data is copied either
Mat C = B.row(3);
// now create a separate copy of the matrix
Mat D = B.clone();
// copy the 5-th row of B to C, that is, copy the 5-th row of A
// to the 3-rd row of A.
B.row(5).copyTo(C);
// now let A and D share the data; after that the modified version
// of A is still referenced by B and C.
A = D;
// now make B an empty matrix (which references no memory buffers),
// but the modified version of A will still be referenced by C,
// despite that C is just a single row of the original A
B.release();
// finally, make a full copy of C. As a result, the big modified
// matrix will be deallocated, since it is not referenced by anyone
C = C.clone();
```

你可以看到使用Mat和其他数据结构的用法是非常简单的。但是对于那些高级类或者甚至没有将自动内存处理考虑进去的自定义数据结构该怎么办呢？对此，OpenCV则提供了cv::Ptr模板类，这个类的使用方法和C++ 11中的std::shared_ptr类似。因此，与其使用普通指针类型：

```
T* ptr = new T(...);
```

你可以这么使用

```
Ptr<T> ptr(new T(...));
```

或者：

```
Ptr<T> ptr = makePtr<T>(...);
```

Ptr<T> 封装了一个指向T实例的指针和一个与指针相关的引用计数。cv::Ptr的描述页面提供了更多细节

对输出数据自动进行内存分配

OpenCV会自动回收内存，同样的，也会在大部分情况下为输出的函数变量自动分配内存。因此，如果一个函数有一个或者多个输入数组（cv::Mat实例）和一些输出数组，输出数组会自动分配或重分配。输出数组的大小和类型取决于输入数组的大小和类型。在需要的情况下，函数可以提供额外的变量来表明输出函数的特性。

例如:

```
#include "opencv2/imgproc.hpp"
#include "opencv2/highgui.hpp"
using namespace cv;
int main(int, char**)
{
    VideoCapture cap(0);
    if(!cap.isOpened()) return -1;
    Mat frame, edges;
    namedWindow("edges", WINDOW_AUTOSIZE);
    for(;;)
    {
        cap >> frame;
        cvtColor(frame, edges, COLOR_BGR2GRAY);
        GaussianBlur(edges, edges, Size(7,7), 1.5, 1.5);
        Canny(edges, edges, 0, 30, 3);
        imshow("edges", edges);
        if(waitKey(30) >= 0) break;
    }
    return 0;
}
```

在视频捕捉模块获取到视频数据集的分辨率和位深以后，我们可以通过>>的操作符来自动分配数据集。cvtColor函数可以自动分配数据集边界。他的大小和位深和输入数组的长度一致。频道数为1，因为cv::COLOR_BRG2GRAY经过颜色转换后便出现灰度转换的情况。需要注意的是，数据集和边界只会在循环体的第一次执行的时候被转换，因为接下来的视频帧有相同的分辨率。如果你的视频分辨率是有所改变的，数组会自动重新分配。

这种技术的核心关键是通过Mat::create方法来实现。这个方法可以生成所需要的数组大小和类型。如果数组已经分配了特定的大小和类型，这个方法不会做任何事情。而且，在一些条件下（这部分一般包括减少引用计数，并将其与0进行比较），他会释放之前已经分配的数据，并接着分配一个新的特定大小的缓冲区。许多函数都会通过调用Mat::create方法来生成输出数组，所以从这个方法开始输出函数的分配就已经自动实现了。

有一些函数方法，比如cv::mixChannels，cv::RNG::fill和其他一些函数或者方法的处理方式，这些函数方法在使用的时候需要注意一下。这些函数不会为输出数组自动分配空间，所以你需要在调用这些函数之前进行空间分配。

饱和算法

作为机器视觉库，OpenCV要处理非常多的图片像素，这些像素通常都会被编码为一个压缩形式，每频道有8位或者16位，因此压缩后的数据会在一个限定的范围内。另外，一些具体的图形操作，如色彩空间变换、亮度/对比度调节、锐化、复杂插值运算，会产生一些不在限定范围内的数据。如果只是将低8位的结果保存下来，图像最终会有视觉上的改变而且会影响后期的图形分析。为了解决这个问题，我们需要使用饱和算法来对结果进行处理。例如，为了保存一张8位图的一个处理结果的R分量，你要找到一个在0~255之间的最接近结果的值。

![](http://latex.codecogs.com/gif.latex?I(x,y)=\min(\max(\textrm{round}(r),0),255))

有符号的8位和16位和无符号位类型都使用了同样的规则。这样的语法规范在库中随处可见。在C++代码里，库使用的是cv::saturate_cast<>函数来模仿标准C++的类型转换操作。你可以通过下面的例子来理解之前给出的公式：

```
I.at<uchar>(y, x) = saturate_cast<uchar>(r);
```

cv::uchar是OpenCV中8位无符号整数类型。在这个已经优化了的单指令多数据流（SIMD，Single Instruction Multiple Data）中，比如将单指令多数据流技术扩展(SSE2, Streaming SIMD Extensions)用作PADDUSB、PACKUSWB等汇编指令。他们的应用可以让程序获得在C++代码中一样的行为。

>   注意
>
>       饱和运算并不能用在结果为32位整型的数据上。

保持像素格式不变。减少使用模板

模板在C++中有着非常强大的功能，使用起来非常棒，而且是一种安全的数据结构和算法实现。然而，如果过度使用模板会导致编译时间增加和代码规模的膨胀。另外，如果独立使用了模板，那么，区分开接口和模板的实现是非常困难的。因此，如果你只是使用了简单的算法，那么就不应该为了使用机器视觉库而导致你的算法代码变得不可维护。除了以上的原因，同时也为了简化和其他比如Python、Java、Matlab这些并没有实现模板或模板功能非常有限的语言的绑定流程，目前的OpenCV实现是基于多态性和运行时分派（没有更好的译法，runtime dispatch)不是采用模板的方式实现。而在部分地方，运行时分派在进行像素访问处理的时候，会变得比较困难（常规化cv::Ptr<>的实现），或者不是很方便（cv::sature_cast<>()）。因此，这一部分使用少量的模板类、方法和函数。而在其他地方，目前的OpenCV版本中尽可能不使用模板。

因此，OpenCV库只能处理一小部分的原始数据类型。换而言之，数组的元素必须为以下类型的一种：

* 8位无符号整型 (uchar)
* 8位符号整型 (schar)
* 16位无符号整型 (ushort)
* 16位符号整型 (short)
* 32位符号整型 (int)
* 32位浮点数 (float)
* 64位浮点数 (double)
* 如果我们有一个元组，是由几种数组组成在一起的，那么数组中的元素必须类型一致（上面的一种）。数组的元素是元组被称之为多频道数组，与单频道数组不同，单频道的数组元素是纯量的。CV_CN_MAX常量定义了最大的数组频道数，目前为512。

对这些基本类型，我们提供了一下枚举变量：

```
enum { CV_8U=0, CV_8S=1, CV_16U=2, CV_16S=3, CV_32S=4, CV_32F=5, CV_64F=6 };
```

多频道（n-channel）类型可以通过以下选项来说明：

* CV_8UC1 ... CV_64FC4常量(频道数为1至4)
* 当频道数大于4或者在编译阶段未知时，CV_8UC(n) ... CV_64FC(n)或者CV_MAKETYPE(CV_8U, n) ... CV_MAKETYPE(CV_64F, n)宏。

>   注意
>
>       CV_32FC1 == CV_32F、CV_32FC2 == CV_32FC(2) == CV_MAKETYPE(CV_32F, 2)、和 CV_MAKETYPE(depth, n) == ((depth&7) + ((n-1)<<3)。也就意味着，这些常量是通过深度来实现的，取值的低三位，频道数最小为一，取log2(CV_CN_MAX)最近的整数位。

例如:

```
Mat mtx(3, 3, CV_32F); // make a 3x3 floating-point matrix
Mat cmtx(10, 1, CV_64FC2); // make a 10x1 2-channel floating-point
                           // matrix (10-element complex vector)
Mat img(Size(1920, 1080), CV_8UC3); // make a 3-channel (color) image
                                    // of 1920 columns and 1080 rows.
Mat grayscale(image.size(), CV_MAKETYPE(image.depth(), 1)); // make a 1-channel image of
                                                            // the same size and same
                                                            // channel type as img
```

如果数组中含有比较复杂的元素就不能被OpenCV创建并加以使用。另外，每一个函数或者方法可以使用上述类型中的一部分。通常情况下，越是复杂的算法，支持的元素就越少。以下列出了一些常用算法支持的类型：

* 面部识别算法只支持8位灰阶或者彩色图形。
* 线性代数函数和大多数机器学习算法支持浮点类数组。
* 基本函数，如cv::add，则支持所有类型。
* 色彩空间转换函数支持8位无符号元素，16位无符号元素，32位浮点类型。

每一个函数支持的元素类型都由实际应用所决定，会基于以后的用户需求进行修改。

输入数组和输出数组

许多OpenCV函数处理密集型二维或多维数值型数组。通常情况下，这些函数会使用cppMat做为参数，但在一些情况下，使用std::vector<>更方便一点（比如，点的集合）或者cv::Matx<>（3x3的单应性矩阵等等）。为了避免API的重复，特殊的“代理”类型引入到OpenCV当中。基本的“代理”类型就是cv::InputArray。它用于一个函数输入中的只读数组传递。继承自输入函数类型的输出函数cv::OutputArray用于特定函数的输出数组。通常情况下，你不需要关心中间过程中的数据类型（你也不要显式地声明这些类型的变量）——这些内容OpenCV都会自动处理。你可以假设这样一种情况，除了输入数组/输入数组，你就直接使用Mat、std::vector<>、cv::Matx<>、cv::Vec<>或者cv::Scalar。但一个函数有可选的输入数字或输出数组时，你可以选择不设置输入数组/输出数组，直接传递cv::noArray()即可。

异常处理

OpenCV使用异常来表明重要的错误。当输入数据是正确的格式并且在特定的范围之内，但算法因为某些原因不能正确地处理（比如，优化函数没有完全考虑到这种情况），它会返回一个特定的错误值（通常情况下，只是一个boolean变量）。

异常可以是cv::Exception类或者继承他的类的实例。在这种情况下，cv::Exception是std::Exception的继承类。所以，他能被其他标准C++库组件所捕获，并进行处理。

异常通常情况下使用CV_ERROR(errcode， description)宏或者类似于printf的CV_Error_(errcode, (printf-spec, printf-args))变量，或CV_Assert(condition)宏来检查错误条件并在没有指明条件的情况下抛出一个异常。对于性能相关的代码，可以使用CV_DbgAssert(condition)方式来测试只与Debug配置相关的错误。由于OpenCV是可以自动进行内存管理，所有的中间缓存都是在出现突然的错误以后自动回收的。你只需要加一个try语句便可以捕获异常。在需要的情况下，请这样使用：

```
try
{
    ... // call OpenCV
}
catch (const cv::Exception& e)
{
    const char* err_msg = e.what();
    std::cout << "exception caught: " << err_msg << std::endl;
}
```

多线程和重进入能力

目前而言，OpenCV可以完全重进入。也就是说，不同类实例的同样的函数或者同样的方法可以被不同的线程所调用。同时，同样的Mat变量可以用在不同的线程中，因为引用计数操作使用的是体系特定的原子性操作。
