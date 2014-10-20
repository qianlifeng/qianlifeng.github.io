title: 使用OpenCV进行简单验证码识别
category: Computer Vision
date: 2014-06-29 18:55:17
tags: [验证码,OpenCV]
---

最近对计算机视觉产生了点兴趣，具体来说应该是简单的图形识别方向。本文介绍了如何使用OpenCV进行简单的验证码识别。
<!--more-->

从头开始学习计算机视觉需要很强的数学功底，从图像预处理到图像匹配都需要用到数学知识，这对于我们这种初学者来说绝对是道很大很大很大很大.....很大的门槛（原谅我对数学敬畏!_!）。不过，不要灰心，我们还有神器[OpenCV](http://opencv.org/)。OpenCV是一套开源的计算机视觉库，他封装了许多计算机视觉处理中常用的方法。在我们这边文章中，我们主要用到了他的阈值处理，轮廓查找，KNN分类器等等。  

先看成品：
[http://nbviewer.ipython.org/gist/qianlifeng/95023d6c8ce8b28518b8](http://nbviewer.ipython.org/gist/qianlifeng/95023d6c8ce8b28518b8)  
我摘一些主要的代码片段介绍一下：  

原始图像：  
<img src="http://scott-tuchuang.qiniudn.com/origin.png" />

所有的验证码处理一般都会先对图像进行一些预处理，包括去除噪点，灰度化，二值化，去除干扰线等等。二值化是为了降低图片的维度，从原来的RGBA变为只有0和1的一维数组。我们使用cvtColor进行灰度化处理。
```
gray = cv2.cvtColor(im,cv2.COLOR_BGR2GRAY)
```
<img src="http://scott-tuchuang.qiniudn.com/gray.png" />  

接着去噪，简单的去噪原理如下。我们会设置一个阈值，高于这个阈值的点变为黑色，低于的变成白色。（这里使用的是名为自适应阈值的去噪方法，效果好一点）
```
threshold = cv2.adaptiveThreshold(gray, 255, cv2.ADAPTIVE_THRESH_MEAN_C,cv2.THRESH_BINARY, 11, 40)
```
<img src="http://scott-tuchuang.qiniudn.com/captcha_threshold.png" />  

到这边图像预处理基本已经差不多了，剩下来就是寻找里面的图像轮廓，进行切图。  

```
(cnts, _) = cv2.findContours(ts.copy(), cv2.RETR_TREE, cv2.CHAIN_APPROX_NONE)
```
这句话会找出图片给里面所有的轮廓信息，当然里面有很多无用的轮廓，比如0这个数字，他会找出两个轮廓出来。一个是0的外圈，一个是0的内圈。我们还需要一些简单的代码去除这些多余的轮廓。  

```
cmax = 100
cmin = 20
cts = []
for item in cnts:
    if cmin < len(item) < cmax:
        (x,y,w,h) = cv2.boundingRect(item)
        cts.append((x,y,w,h))
```  

未完待续!
