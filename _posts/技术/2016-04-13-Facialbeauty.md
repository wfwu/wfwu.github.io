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

#### 1.1YCrCb高斯肤色模型

--------------------------------

肤色识别的本质是聚类，在指定的色彩空间利用大量样本训练出肤色在该空间的分布情况，从而实现分类。目前，主流的算法区别在于色彩空间选择的不同，例如：

* RGB色彩空间
* YCrCb色彩空间
* YUV色彩空间
* HSV色彩空间

其中由于RGB色彩空间色度信息耦合，所以识别准确率最低；其余三个色彩空间亮度和色度被编码在不同维度，基于特定肤色模型，均有识别率较高的算法。下面主要介绍基于YCrcb色彩空间的高斯肤色模型。

**YCrCb色彩空间**

![cgi_principle](/public/img/2016/04/15/RGB2YCbCr.png)

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

![cgi_principle](/public/img/2016/04/15/skin_gauss.png)

**相似度计算公式**

$$ p(x) = exp[-0.5 *(x-\mu)^T\sigma(x-\mu)] $$

说明：

1. x=$$[Cr,Cb]^T$$为像素点在CrCb空间中的向量，$$\mu$$为均值 $$\sigma$$为方差。

2. 依照公式训练参数$$\mu$$和$$\sigma$$:

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

#### 2.1概述

皮肤美白作为市场上任何美容软件的必备功能之一，深受爱美女性的喜爱，所谓“一白遮百丑”，可见该功能需求之大。美白的本质就是把皮肤区域变白变亮，而白亮在图像处理中意味着像素点的值增大，但这种增大要在合理的范围内，否则将导致图像失真，所以必须满足对原图的色阶有所增强，且亮度两端增强比例稍弱，中间稍强。

#### 2.2Logarithmic curve变换

**对数曲线公式**

$$ \upsilon(x,y) = \frac{log(\omega(x,y)*(\beta-1)+1)}{log(\beta)} $$

其中，$$\omega(x,y)$$表示输入图像，$$\upsilon(x,y)$$表示输出图像，$$\beta$$为可调节参数，此参数越大，美白程度越强。

**对数曲线图像**

![cgi_principle](/public/img/2016/04/15/log_curve.png)

#### 2.1 FastCGI原理

--------------------------------

FastCGI的处理流程如下图所示：


![fastcgi_principle](/public/img/2016/02/24/fastcgi_principle.png)


流程说明：

1. Web服务器启动时启动并初始化FastCGI进程。 例如IIS ISAPI、apache mod_fastcgi、 nginx ngx_http_fastcgi、 lighttpd mod_fastcgi。
2. FastCGI进程管理器自身初始化，启动多个CGI解释器进程并等待来自Web 服务器的连接。（启动FastCGI进程时，可以配置以ip和UNIX域socket两种方式启动）
3. 当客户端请求Web服务器时， Web服务器将请求采用socket方式发送到FastCGI主进程。FastCGI主进程选择并连接到一个CGI解释器。Web 服务器将CGI环境变量和标准输入发送到FastCGI子进程
4. FastCGI子进程完成处理后将标准输出和错误信息从同一socket连接返回Web 服务器。当FastCGI子进程关闭连接时，请求便处理完成。
5. Web服务器利用FastCGI主进程返回的结果构建HTTP Response响应客户端。

#### 2.2 小结

--------------------------------

FastCGI程序并不需要不断的创建、消亡新进程，从而大大提高了效率，从[FastCGI官网](http://www.fastcgi.com/drupal/node/6?q=node/15)给出的数据，其效率至少比CGI高出5倍。同时
它还支持分布式部署，可以将FastCGI程序放在Web服务器以外的专门的应用服务器上。

### 3 总结

> FastCGI像是一个常驻(long-live)型的CGI，它可以一直执行着，只要激活后，不会每次都要花费时间去fork一次(这是CGI最为人诟病的fork-and-execute 模式)。它还支持分布式的运算, 即 FastCGI 程序可以在网站服务器以外的主机上执行并且接受来自其它网站服务器来的请求。
> FastCGI是语言无关的、可伸缩架构的CGI开放扩展，其主要行为是将CGI解释器进程保持在内存中并因此获得较高的性能。众所周知，CGI解释器的反复加载是CGI性能低下的主要原因，如果CGI解释器保持在内存中并接受FastCGI进程管理器调度，则可以提供良好的性能、伸缩性、Fail- Over特性等等。