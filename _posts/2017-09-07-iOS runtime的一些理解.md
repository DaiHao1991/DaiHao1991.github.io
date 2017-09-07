---
layout:     post
title:      iOS runtime的一些理解
subtitle:   
date:       2017-09-07
author:     DH
header-img: img/post-bg-hacker.jpg  
catalog: true
tags:
    - iOS
    - runtime
---


# 概述

iOS runtime就是平时说的运行时，它和OC语言的动态性息息相关，它是一套C语言的API。

所谓动态性就是，OC在编译时并不是绑定函数，而是在运行的时候，才决定调用哪一个函数。

#### runtime中的一些术语

（1）SEL：他就是一个映射到方法的C字符串，我的理解是他就代表了一个方法，在一个类中，绝对不可能有两个重名的方法。

类型是typedef struct objc_selector *SEL

（2）id：结构体指针类型，指向OC的任何对象。

类型是typedef struct objc_object *id

这个结构体中包含一个isa指针，指向对象的类，这个指针的类型就是Class

（3）Class：指向结构体objc_class的指针

类型是typedef struct objc_class *Class

这个结构体如下所示：

```
struct objc_class {
    Class isa  OBJC_ISA_AVAILABILITY;
#if !__OBJC2__
    Class super_class   OBJC2_UNAVAILABLE;
    const char *name   OBJC2_UNAVAILABLE;
    long version     OBJC2_UNAVAILABLE;
    long info     OBJC2_UNAVAILABLE;
    long instance_size   OBJC2_UNAVAILABLE;
    struct objc_ivar_list *ivars  OBJC2_UNAVAILABLE;
    struct objc_method_list **methodLists  OBJC2_UNAVAILABLE;
    struct objc_cache *cache    OBJC2_UNAVAILABLE;
    struct objc_protocol_list *protocols   OBJC2_UNAVAILABLE;
#endif
} OBJC2_UNAVAILABLE;		

```

各个属性的意义：

isa：指向元类(mete class)

super_class：指向父类

name：类名

version：类的版本号

info：类的信息    

instance_size：类的实例对象的大小

ivars：指向类的成员变量列表的指针

methodLists：指向类的实例方法列表。 它将方法选择器和方法的实现地址联系起来，methodLists是一个指向指针的指针，它指向的指针是objc_method_list，
我们可以通过修改*methodLists来添加类的成员方法，这也就是category的原理。因此，我们也可以知道category无法添加成员变量的原因。

cache：被调用过的方法的缓存，runtime将被调用过的方法假如到cache，下次调用的时候更快。

protocols：类中的协议列表


#### 消息发送

在iOS中调用一个函数，就是给对象发送一个消息。例如[reciver message];这局代码会被编译器转化成objc_msgSend(reciver,selector)。

那么消息的发送经过哪些步骤呢？

（1）检查selector是否需要忽略。例如对于PC端的开发，有垃圾回收机制，会忽略retain和release消息；

（2）检查消息的接受者是不是nil，OC中像nil发送消息不会导致crash，runtime会忽略掉这个消息；

（3）在缓存cache中查找有无这个方法，有的话就执行；

（4）在类方法中查找有无这个方法，没有的话就去父类中查找，一直查找的NSObject

（5）如果NSObject也没有这个方法，就要进行动态方法解析。

而在消息的发送中，我们要用到下面四中
