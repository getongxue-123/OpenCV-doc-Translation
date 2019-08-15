目的

在这篇教程中，你将会学到：

* 使用OpenCV中的Sobel()函数来计算图像的派生值。
* 使用OpenCV中的Scharr()函数通过3*3的核函数来计算更为精确的派生值。

理论

> 注意
> 
> 以下内容来自Bradski和Kaehler的《OpenCV》入门一书。

1.在前两篇教程中我们已经看到了一些实际的卷积例子。一个特别重要的卷积例子是计算图片的派生值（或近似的派生值）。

2.为什么计算图像的派生值特别重要？想象一下，我们想检测显示在图像中的边界。例如：

![](https://docs.opencv.org/4.1.0/Sobel_Derivatives_Tutorial_Theory_0.jpg)

你可以清楚地看到在边缘上，像素颜色有非常明显的变化。一个非常好的方法来描述这个变化便是衍生值。在图像中，径向上有高度变化值表示有显著的变化。

3.为了能更形象一点，我们假设一张1维图形。在下图中，强度值跳跃的地方表示的便是边界：

![](https://docs.opencv.org/4.1.0/Sobel_Derivatives_Tutorial_Theory_Intensity_Function.jpg)

4.边界条约如果我们使用了派生值的话会更加明显（实际上，这里是最大值）

![](https://docs.opencv.org/4.1.0/Sobel_Derivatives_Tutorial_Theory_dIntensity_Function.jpg)

5.因此，通过上述解释，我们可以得出一种检测边界的方法，图像可以通过检测固定像素位置的径向变化，要是大于其领域像素的径向变化，就有可能是边界（或者更一半的情况，比阈值高）。

6.更多详细内容，请参阅Bradski和Kaehler的《OpenCV》入门一书。

索贝尔算法

1.索贝尔算法是离散微积分算子。他是近似估计图片像素的径向变化值的函数。

2.索贝算子结合了高斯模糊和微分。

公式化处理

我们把需要处理的图像记做I：

1.计算两个派生值：

    a.水平变化：这会将I用一个奇数大小的核函数Gx来进行卷积处理。例如对于一大小为三的核函数，Gx会用如下公式表是：
    
    ![](http://latex.codecogs.com/gif.latex?G_%7Bx%7D%20%3D%20%5Cbegin%7Bbmatrix%7D%20-1%20%26%200%20%26%20+1%20%5C%5C%20-2%20%26%200%20%26%20+2%20%5C%5C%20-1%20%26%200%20%26%20+1%20%5Cend%7Bbmatrix%7D%20*%20I)

    b.垂直变化：这会将I用一个奇数大小的核函数Gy来进行卷积处理。例如对于一大小为三的核函数，Gy会用如下公式表是：
    
    ![](http://latex.codecogs.com/gif.latex?G_%7By%7D%20%3D%20%5Cbegin%7Bbmatrix%7D%20-1%20%26%20-2%20%26%20-1%20%5C%5C%200%20%26%200%20%26%200%20%5C%5C%20+1%20%26%20+2%20%26%20+1%20%5Cend%7Bbmatrix%7D%20*%20I)

    在图像每一个点上，我们可以通过计算所有的结果来估计这个点上的径向改变值：

    ![](http://latex.codecogs.com/gif.latex?G%20%3D%20%5Csqrt%7B%20G_%7Bx%7D%5E%7B2%7D%20+%20G_%7By%7D%5E%7B2%7D%20%7D)

2.有的时候公式可以适当简化一下：

![](http://latex.codecogs.com/gif.download?G%20%3D%20%7CG_%7Bx%7D%7C%20+%20%7CG_%7By%7D%7C)

> 注意
> 
> 当核的大小为3的时候，上面所提及的索贝尔算子可能会造成相当大的误差（尽管如此，索贝尔算子是唯一可使用的派生值算法）。如果在OpenCV中使用了Scharr()函数，那么OpenCV会在你设置核函数大小为3的时候提示这函数的结果可能是不准确的。我们有另外的快速而且精确的方法来实现索贝尔算子函数。它使用了以下核函数：
>
> ![](http://latex.codecogs.com/gif.download?G_%7Bx%7D%20%3D%20%5Cbegin%7Bbmatrix%7D%20-3%20%26%200%20%26%20+3%20%5C%5C%20-10%20%26%200%20%26%20+10%20%5C%5C%20-3%20%26%200%20%26%20+3%20%5Cend%7Bbmatrix%7D)
>
> ![](http://latex.codecogs.com/gif.download?G_%7By%7D%20%3D%20%5Cbegin%7Bbmatrix%7D%20-3%20%26%20-10%20%26%20-3%20%5C%5C%200%20%26%200%20%26%200%20%5C%5C%20+3%20%26%20+10%20%26%20+3%20%5Cend%7Bbmatrix%7D)
> 
> You can check out more information of this function in the OpenCV reference - Scharr() . Also, in the sample code below, you will notice that above the code for Sobel() function there is also code for the Scharr() function commented. Uncommenting it (and obviously commenting the Sobel stuff) should give you an idea of how this function works.