---
layout: post
title: 人像美容系统
author: wfwu
comments: true
permalink: /2016/04/13/Facialbeauty.html
keywords:
description:
categories:
  - 技术
tags:
  - c
  - 图像处理
---

### 一、概述

拍照作为手机中的一大功能，正日益受到人们和厂商的关注。对于用户来说，拍照已经成为生活中不可或缺的活动，特别是自拍，已经成为了这个时代的潮流。

因此，许多互联网企业针对普通拍照和自拍，都精心打造了各种美化照片的软件让用户能够拍出高质量的照片，有更好的留影体验。

本文研究的美容算法，也是出于美化图片的目的，通过对人脸皮肤区域的美化，实现人脸美容的功能，让照片中的人更美。本文所使用的算法的核心是YCrCb高斯肤色模型、Logarithmic curve变换，双边滤波、双指数边缘保留算法

### 二、算法实现及效果展示

### 1 皮肤检测

#### 1.1 YCrCb高斯肤色模型

--------------------------------

肤色识别的本质是聚类，在指定的色彩空间利用大量样本训练出肤色在该空间的分布情况，从而实现分类。目前，主流的算法区别在于色彩空间选择的不同，例如：

* RGB色彩空间
* YCrCb色彩空间
* YUV色彩空间
* HSV色彩空间

其中由于RGB色彩空间色度信息耦合，所以识别准确率最低；其余三个色彩空间亮度和色度被编码在不同维度，基于特定肤色模型，均有识别率较高的算法。下面主要介绍基于YCrcb色彩空间的高斯肤色模型。

**YCrCb色彩空间**

{% include image.html
            img="public/img/2016/04/15/RGB2YCbCr.png"
            title="image.html模板"
            caption="RGB空间转换到YCrCb空间" %}

* Y亮度分量
* Cb蓝色色度
* Cr红色色度

**RGB2YCbCr公式**

$$
\begin{bmatrix} Y \\ Cr \\ Cb\\\end{bmatrix}=
\begin{bmatrix} 0.299&0.578&0.114 \\ -0.1687&-0.3313&-0.4187 \\ 0.5&-0.4187&-0.0813\\\end{bmatrix}
\begin{bmatrix} R \\ G \\ B\\\end{bmatrix}+
\begin{bmatrix} 0 \\ 128 \\ 128\\\end{bmatrix}
$$

**高斯肤色模型**

对大量人种肤色样本的研究后发现，肤色在YCrCb色彩空间呈现二位高斯分布特性，称为混合高斯模型。

下图分别为肤色肤色统计特性与高斯肤色模型：


<table><tr><td><img src="/public/img/2016/04/15/skin_gauss.png" width="50%"></td><td><img src="/public/img/2016/04/15/gauss.jpg" width="50%" ></td></tr></table>


**相似度计算公式**

$$ p(x) = exp[-0.5 *(x-\mu)^T\sigma(x-\mu)] $$

说明：

1.x=$$[Cr,Cb]^T$$为像素点在CrCb空间中的向量，$$\mu$$为均值 $$\sigma$$为方差。

2.依照公式训练参数$$\mu$$和$$\sigma$$:

 $$\mu = (156.5599,117.4361)^T$$

 $$\sigma =\begin{bmatrix} 299.4574 & 12.1430\\ 12.1430&160.1301\\\end{bmatrix}$$

#### 1.2 算法评测

-------------------------------

![cgi_principle](/public/img/2016/04/15/facedetction.jpg)

如图所示，像素点越亮表示该点被判断为皮肤的概率越大。从中可以看出基于YcrCb色彩空间的高斯肤色模型能很好检测图像中的皮肤区域。

#### 1.3 性能优化

--------------------------------

由于色彩在YCbCr空间中是离散分布，所以可以用一个映射表存储相应点的概率，利用查表来识别肤色，从而避免的大量的浮点计算。

### 2 皮肤美白

#### 2.1 概述

皮肤美白作为市场上任何美容软件的必备功能之一，深受爱美女性的喜爱，所谓“一白遮百丑”，可见该功能需求之大。美白的本质就是把皮肤区域变白变亮，而白亮在图像处理中意味着像素点的值增大，但这种增大要在合理的范围内，否则将导致图像失真，所以必须满足对原图的色阶有所增强，且亮度两端增强比例稍弱，中间稍强。

#### 2.2 Logarithmic curve变换

**对数曲线公式**

$$ \upsilon(x,y) = \frac{log(\omega(x,y)*(\beta-1)+1)}{log(\beta)} $$

其中，$$\omega(x,y)$$表示输入图像，$$\upsilon(x,y)$$表示输出图像，$$\beta$$为可调节参数，此参数越大，美白程度越强。

**对数曲线图像**

![cgi_principle](/public/img/2016/04/15/log_curve.png)

#### 2.3 算法评测

{% include image.html
            img="public/img/2016/04/15/whiten_src.jpg"
            title="image.html模板"
            caption="原图" %}

当$$\beta=2 $$、$$\beta=5 $$时：

<table><tr><td><img src="/public/img/2016/04/15/whiten_2.jpg" ></td><td><img src="/public/img/2016/04/15/whiten_5.jpg" ></td></tr></table>

### 3 磨皮祛斑

#### 3.1 引言

--------------------------------

磨皮处理本质上是一个滤波的过程，但是滤波算法的设计上必须满足对面部的斑点，即噪声有很好的滤除作用；同时还要保证边缘不被滤除，即保边；甚至优秀的磨皮算法还要保证处理后皮肤的质感。本文介绍两种算法：基于双指数边缘保留算法和双边滤波算法。

#### 3.2 双指数边缘保留算法

--------------------------------

在《Bi-Exponential Edge-Preserving Smoother》一文中提出了一个算法流程：

![cgi_principle](/public/img/2016/04/15/BEEP.jpg)

主要是有三个过程组成：

* 前向迭代（1）
* 反向迭代（3）
* 加权合并两个结果（5）

然而上述是个一维的过程，对于二维的图像数据，论文中也给出了解决方式：

- 对原始图像进行一次水平迭代计算，然后再进行垂直迭代计算，该过程称之为BEEPSHorizontalVertical。
- 对原始图像进行一次垂直迭代计算，再对其进行垂直迭代计算，该过程称之为BEEPSVerticalHorizontal。
- 滤波后的图像结果为（BEEPSHorizontalVertical+BEEPSVerticalHorizontal）/2。

#### 3.3 双指数边缘保留算法评测

--------------------------------

取$$\lambda$$=0.995,$$\theta$$=20,算法处理效果：

<table><tr><td><img src="/public/img/2016/04/15/beaute_src.jpg" ></td><td><img src="/public/img/2016/04/15/beaute_1.jpg" ></td></tr></table>

#### 3.4 双边滤波器

--------------------------------

参照《Bilateral Filtering for Gray and Color Images》（C. Tomasi）

双边滤波器是一种可以保边去噪的滤波器。该滤波器主要由两个函数组成，一个函数是几何空间距离决定的滤波系数，另一个函数是由像素差值决定的滤波系数。

**基本原理**

![cgi_principle](/public/img/2016/04/15/bilateral_filter.jpg)

**双边滤波公式**

$$ BF[I]_\vec{p} = \frac {1}{W_\vec{p}}\sum G_{\sigma_s}(||\vec{p}-\vec{q}||)G_{\sigma_r}(|I_\vec{p}-I_\vec{q}|)I_\vec{q}$$

其中：
  $$I_\vec{q}$$为输入图像在位置q处的像素，$$BF[I]_\vec{p}$$为处理后输出像素，$$W_\vec{p}$$为P处邻域内归一化值，$$G_\sigma (x)=e^{-\frac{x^2}{\sigma^2}}$$用来计算滤波系数和像素差滤波系数的高斯函数。

#### 3.5 算法评测

取领域 d=15,$$\sigma_s=\sigma_r=50$$时：

<table><tr><td><img src="/public/img/2016/04/15/beaute_src.jpg" ></td><td><img src="/public/img/2016/04/15/beaute_2.jpg" ></td></tr></table>

### 4 整合效果

将高斯肤色模型、Logarithmic curve变换、双边滤波算法整合，得出效果图：

![cgi_principle](/public/img/2016/04/15/d5.jpg)

![cgi_principle](/public/img/2016/04/15/d3.png)

![cgi_principle](/public/img/2016/04/15/d2.jpg)

![cgi_principle](/public/img/2016/04/15/d3.jpg)

![cgi_principle](/public/img/2016/04/15/d4.png)

