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

在ARC无效时，比较简单，在转换后，使用完毕之后，手动释放即可。而且ARC无效时，不涉及到桥接。

（1）Objective-C 转 Core-Foundation

```
    NSString *string = @"OC TO CF";
    NSString *str = [[NSString alloc]initWithFormat:@"%@",string];
    CFStringRef cfStringRef = (CFStringRef)(str);
    
    /*
     
     使用cfStringRef对象
     
     */
    
    
    // 使用完毕进行释放
    CFRelease(cfStringRef);		

```

（2）Core-Foundation 转 Objective-C 

```
    
    CFMutableArrayRef cfMutableArr = CFArrayCreateMutable(kCFAllocatorDefault, 0, NULL);
    
    NSMutableArray *OCArrs = (NSMutableArray *)(cfMutableArr);
    
    /*
     
     使用OCArrs对象
     
     */
    
    
    // 使用完毕进行释放
    [OCArrs release];		

```


- **ARC有效时**


ARC有效的时候就要复杂一点，需要用到桥接。

有一点需要明确的是，ARC只对Objective-C对象进行管理，而不能对Core-Foundation对象进行管理。Core-Foundation对象必须使用CFRelease 和 CFRetain来
进行内存管理。所以，当Objective-C 转 Core-Foundation对象时，就必须自己手动管理，Core-Foundation 转 Objective-C就使用ARC进行管理。

这样就能避免内存泄漏和double free。

也因此有如下三种不同的转换：

__bridge 不改变对象的所有权，只进行单纯的赋值，容易造成悬垂指针。

__bridge_retain（CFBridgingRetain()） 使用在Objective-C 转 Core-Foundation时，它能让转换赋值的对象也持有所赋值的对象，简单点就是将原来的OC对象的引用+1，需要注意的是这个时候我们已经解除了ARC的所有权，需要手动释放对象。

__bridge_transfer（CFBridgingRelease()） 使用在Core-Foundation 转 Objective-C，它让被转转的对象在转换后立马释放。并且给予ARC所有权，让ARC进行管理。不需要手动释放。


（1）Objective-C 转 Core-Foundation

```
    NSArray *array = [[NSArray alloc]init];
    CFArrayRef CFArr = (__bridge_retained  CFArrayRef)array;// 同：CFArrayRef CFArr = (CFArrayRef)CFBridgingRetain(array);
    
    
    /*
     
     使用CFArr
     
     */
    
    // 使用完毕需要释放
    CFRelease(CFArr);		

```


如果不加最后这句CFRelease(CFArr);就会造成内存泄漏。


（2）Core-Foundation 转 Objective-C 

```
    CFArrayRef CFArr = CFArrayCreate(kCFAllocatorDefault, NULL, 0, NULL);
    
    NSArray *arr = (__bridge_transfer id)CFArr;
    
    /*
     
     使用arr
     
     */
    
    // 使用完毕不需要手动释放		
        
```

请看下面一个例子：

```
    NSArray *array = [[NSArray alloc]init];
    CFArrayRef CFArr = (__bridge_retained  CFArrayRef)array;// 同：CFArrayRef CFArr = (CFArrayRef)CFBridgingRetain(array);
    
    NSArray *arrayNew = (__bridge_transfer id)CFArr；
    
```

运行上述代码并不会造成内存泄漏，因为“__bridge_transfer”重新让ARC取得可控制权。

#### 使用__bridge时需要特别注意

__bridge仅仅会赋值（类型转换），并不会改变对象的所有权。意思就是说从OC转CF，ARC管理内存，从CF转OC，需要开发者手动释放，不归ARC管。


下面进行具体说明：


OC转CF

```
    NSString *aNSString = [[NSString alloc]initWithFormat:@"test"];   
    CFStringRef aCFString = (__bridge CFStringRef)aNSString;   
      
    (void)aCFString;  		

```

CF转OC
```
    CFStringRef aCFString = CFStringCreateWithCString(NULL, "test", kCFStringEncodingASCII);   
    NSString *aNSString = (__bridge NSString *)aCFString;  
      
    (void)aNSString;  
      
    CFRelease(aCFString);	

```

需要注意的两个问题，图中说的很明白了：

![](https://ws1.sinaimg.cn/large/006tKfTcgy1fi6dqdq69bj30rc0qe4d1.jpg)

![](https://ws2.sinaimg.cn/large/006tKfTcgy1fi6dqlr2nqj30rh0rl18o.jpg)

