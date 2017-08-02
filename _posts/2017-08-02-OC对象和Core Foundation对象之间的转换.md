---
layout:     post
title:      OC对象和Core Foundation对象之间的转换
subtitle:   
date:       2017-08-02
author:     DH
header-img: img/post-bg-miui6.jpg 
catalog: true
tags:
    - iOS
    - 桥接
---
#### OC对象和Core Foundation对象

Core Foundation对象主要用于C语言编写的Core Foundation框架中，他是一组C语言的接口，主要为iOS应用提供基本的数据管理和服务功能。主要管理的数据包括：

（1）数组集合等群体数据

（2）程序包

（3）字符串

（4）时间日期

（5）原始数据块

提供的服务主要包括：

（1）偏好管理

（2）线程和RunLoop

（3）URL和数据流

（4）端口和socket

Core Foundation对象也是用引用计数，在ARC无效时，Core Foundation中的retain/release对应CFRetain/CFRelease。

Core Foundation对象和OC对象差别很少，区别就是OC对象使用的是Foundation框架。无论是哪个框架生成的对象，都可以在不同的框架中使用，Foundation框架生成
并持有的对象可以在Core Foundation框架中释放，反之亦然。

#### Core Foundation对象和Objective-C对象的转换

- **ARC无效时**

因为Core Foundation对象和Objective-C对象是相同的，所以只需要简单的C语言就能实现转换，并且这种转换不需要额外的CPU资源，被称之为
免费桥（Toll-Free Brodge）。



- **ARC有效时**


#### 分析

假如我们
