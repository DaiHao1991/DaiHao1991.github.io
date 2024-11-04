---
layout:     post
title:      3. Harmony UI组件
subtitle:   
date:       2024-10-28
author:     DH
header-img: img/post-bg-digital-native.jpg
catalog: true
tags:
    - 鸿蒙
    - 鸿蒙基础
    - Harmony
---
#### Tabs
Tabs一般作为应用的骨架，能够让应用通过视图的切换，快速达到不同的功能。
![](https://i-blog.csdnimg.cn/direct/1c81cbdb577f437783fac59591d6d995.png)

包含两个部分：

1. TabBar：TabBar是导航区域，也就是上图的下方4个按钮，可以根据用于的需要设置添加的数量，位置可以设置为上下左右

2. TabContent：承载每一个TabBar的内容显示。切换TabBar就会切换到对应的显示内容；

用法如下：

首先定义一个数据集，用于初始化TabBar，每一个TabBar包含图片和名字，因此：

1. TabBar的bean:

```

export class IconBean{
  iconNormalImage:Resource;
  iconPressedImage:Resource;
  iconName:string;

  constructor(iconNormalImage: Resource, iconPressedImage: Resource, iconName: string) {
    this.iconNormalImage = iconNormalImage;
    this.iconPressedImage = iconPressedImage;
    this.iconName = iconName;
  }
}


```

2. TabBar数据集：

```

export function getIconBeanData(): IconBean[] {
  return [
    new IconBean($r('app.media.ic_icon'), $r('app.media.ic_ok'), "消息"),
    new IconBean($r('app.media.ic_icon'), $r('app.media.ic_ok'), "发现"),
    new IconBean($r('app.media.ic_icon'), $r('app.media.ic_ok'), "主页"),
    new IconBean($r('app.media.ic_icon'), $r('app.media.ic_ok'), "动态"),
    new IconBean($r('app.media.ic_icon'), $r('app.media.ic_ok'), "我的"),
  ];
}


```

