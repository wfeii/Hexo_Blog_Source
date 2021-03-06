---
layout: post
title: "UI优化"
category: 
	- 外功招式
	- Android
	- Android性能
tags: 
	- Android性能

date: 2016-08-06 16:40:12
---

# UI优化
总结来源与Google发布的性能优化的视频 **[Android Performance Patterns](https://www.youtube.com/playlist?list=PLWz5rJ2EKKc9CBxr3BVjPTPoDPLdPIFCE)** 和   **[Android Performance Optimizing Apps for Speed and Usability](https://www.udacity.com/course/android-performance--ud825)**
对于用户感到卡顿,不流畅的原因有很多,比如Layout层级结构过深,动画过多,界面刷新,等等导致了CPU或者GPU的负担过重,16ms内无法完成一帧的绘制,导致了掉帧,从而表现出卡顿,不流畅.

## 绘制原理
为了更好的理解UI优化,在此之前先要说明一下硬件的基础.

### VSYNC
两个概念 :

- Refresh Rate：代表了屏幕在一秒内刷新屏幕的次数，这取决于硬件的参数，例如60Hz。
- Frame Rate：代表了GPU在一秒内绘制的帧数,例如60fps。

GPU获取图形数据绘制,然后硬件负责把绘制的内容显示再屏幕上,二者协调正常.

如果帧速率高于屏幕刷新速率,就会出现撕裂的现象,一部分显示当前的帧,一部分显示现在的帧,这种解决的方案就是双缓冲机制,GPU将帧写进存储器被成为back buffer,而存储器的次级区域成为frame buffer,当写入一帧时,它会开始填充back buffer,而framen buffer保持不变,现在刷新屏幕,使用frame buffer进行绘制,刷新屏幕,vsync触发,也会把back buffer复制到fragme buffer.
更具体的内容参考: **[Graphics](http://source.android.com/devices/graphics/index.html)**

如果屏幕刷新速率高于帧速率时,就会导致屏幕显示还是上一帧的内容,从而出现卡顿,动画不流畅等现象

### 一帧画面绘制过程
   了解了VSYNC的原理,下面该考虑应用程序是如何把画面绘制到屏幕上的了?或者是如何把XML文件转换成用户能够看到并理解的图像的?

  其实这个过程的核心就是进行光栅化(rasterization)的处理过程.这个过程就是把一些高级的对象,比如字符串,形状等转换成屏幕上纹理中的像素点.光栅化是一个非常耗时的过程.正因如此,手机硬件上有一个特殊的部分用于提高光栅化的过程,叫做图像处理器,或者说GPU.
  ![1-1](https://raw.githubusercontent.com/wangfei1991/wangfei1991.github.com_raw_important/master/android%E7%95%8C%E9%9D%A2%E6%98%BE%E7%A4%BA%E6%8A%BD%E8%B1%A1%E5%9B%BE.png )

图像绘制到屏幕上首先在CPU上转换成多边形或者纹理,然后再传递到GPU进行光栅化处理,处理UI对象并转化成多边形和纹理并不是很快的操作,同样从CPU传递给GPU也不是那么快,这行就想办法较少UI对象转换的数量,和提交到GPU的数量,OpenGL ES API允许你将内容传递到GPU,并保存到GPU中,当以后再次绘制一个按钮的时候,只需要参考GPU内存中的已经存在的纹理.

 渲染优化,尽可能多且快的将更多的数据上传到GPU,然后留在GPU中,尽可能长的时间里不去修改.每次更新GPU中的资源,你将会失去宝贵的处理时间.

 下面说明什么时候会导致GPU资源更新.
 Android把xml语言转化成GPU能够识别的资源从而渲染到屏幕上,是通过Display List来实现的,一个Display List对象中基本上包含了所有GPU需要的渲染信息,Display List中包含了GPU可能需要的常用的资源列表,同时也包含了Open GL命令列表.

 ![Alt text](https://raw.githubusercontent.com/wangfei1991/wangfei1991.github.com_raw_important/master/displayList%E5%88%9B%E5%BB%BA%E8%BF%87%E7%A8%8B.png)
在某个视图第一次需要被渲染时,Display List就会被创建,当视图需要显示在屏幕时,会执行GPU的相关指令来进行渲染.

![Alt text](https://raw.githubusercontent.com/wangfei1991/wangfei1991.github.com_raw_important/master/layout%E6%B8%B2%E6%9F%93%E8%BF%87%E7%A8%8B.png)
从上图可以看出
- 只是View属性改变时,只是参考GPU内存中的已经存在,然后执行相关Open GL进行渲染,
- InvaliDated View也不会重新创建Display List
- View 布局改变,比如可见性,大小等

## 优化点

### 概述
渲染操作通常依赖于两个核心组件：CPU与GPU。CPU负责包括Measure，Layout，Record，Execute的计算操作，GPU负责Rasterization(栅格化)操作。CPU通常存在的问题的原因是存在非必需的视图组件，它不仅仅会带来重复的计算操作，而且还会占用额外的GPU资源。
![Alt text](https://raw.githubusercontent.com/wangfei1991/wangfei1991.github.com_raw_important/master/android_performance_course_render_problems.jpg)
对于CPU,应该尽量减少布局的层级深度,CPU减少measure等操作的时间,而对于GPU主要是减少不必要的过度绘制.

### 过度绘制
过度绘制是指同一帧内一个像素点绘制了多次,通过查看overdraw(开发者选项-> 调试GPU过度绘制)
![Alt text](https://raw.githubusercontent.com/wangfei1991/wangfei1991.github.com_raw_important/master/overdraw.png)

颜色代表了像素点过度绘制的次数:

- True color: No overdraw
- Blue: Overdrawn once
- Green: Overdrawn twice
- Pink: Overdrawn three times
- Red: Overdrawn four or more times

下面举例说明,通过不必要的background导致了过度绘制,
![Alt text](https://raw.githubusercontent.com/wangfei1991/wangfei1991.github.com_raw_important/master/overdraw_example.png)
左边是使用了大量的backgroud导致了严重的过度绘制,而右图可以看到移除不必要的background后的效果.
有些过度绘制是不可避免的,优化的目标是尽可能的大部分布局显示真是颜色,少部分显示一层过度绘制.

避免过渡绘制

- 移除window的背景
- 移除布局中不必要的背景

相关的代码可以通过git获得

- [chart with overdraw](https://github.com/udacity/ud825-render/tree/1.11_chat_with_overdraws)
- [ chat overdraws reduced](https://github.com/udacity/ud825-render/tree/1.12_chat_overdraws_reduced)

### 布局的层级深度
了解布局的层级结构可以使用hierarchy viewer,它可以很直接的**呈现布局的层次关系**，**视图组件的各种属性**(可以查看Vie
w可见性,焦点等,可以很好的调试View问题).
对于提升布局性能的关键是避免出现重复的嵌套布局,尽可能的减小层级的深度.
以性能优化视频中的List Item优化为例,
![Alt text](https://raw.githubusercontent.com/wangfei1991/wangfei1991.github.com_raw_important/master/%E5%87%8F%E5%B0%91%E5%B1%82%E7%BA%A7%E7%BB%93%E6%9E%84.png)
把LinearLayout的布局改为RelativeLayout布局从而减少了布局层级深度

相关的代码:

- [comparing layouts](https://github.com/udacity/ud825-render/tree/1.31_comparing_layouts)
- [using a flattened hierarchy](https://github.com/udacity/ud825-render/tree/1.32_using_a_flattened_hierarchy)
