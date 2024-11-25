---
layout:     post
title:      3. Harmony 高级UI组件
subtitle:   
date:       2024-11-04
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
![](https://camo.githubusercontent.com/26a14f8c7194cd2de530235fef0bb842f6216d26765e548ef8d42cb83ac6218b/68747470733a2f2f692d626c6f672e6373646e696d672e636e2f6469726563742f31633831636264623537376634333737383366616335393539316436643939352e706e67)

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

3. 初始化Tabs:

```
@Component
@Preview
@Entry
struct mainPage{
  @State tabsData:IconBean[] = getIconBeanData();

  build() {
      Tabs({barPosition:BarPosition.End}){
        ForEach(this.tabsData,(item:IconBean,index:number)=>{
          TabContent(){
              Text(item.iconName)
          }.tabBar(item.iconName)
        })
      }

        .width('100%')
  }
}
```
因为有了数据集，直接使用ForEach循环初始化。要设置Tabs的位置，可以通过barPosition设置。每一个TabBar的内容通过TabContent设置。

最终效果：
![](https://camo.githubusercontent.com/c44ab61da6f92c6418bd9fd20ef60b5ceed43662839025dd7239edbbb557e929/68747470733a2f2f692d626c6f672e6373646e696d672e636e2f6469726563742f35376330373537316137393734326462613036633537643465623235303832642e706e67)

TabBar的UI支持自定义：

```

@Builder tabBuilder(title:string,imageUrlNormal:Resource){
    Column(){
      Image(imageUrlNormal).
      width(30)
        .aspectRatio(1)
      Text(title)
    }
}
```

整体效果：

![](https://camo.githubusercontent.com/19a497c56c686034afaa63908fe02d55e67e037e0cd481f2f424f1a391f7a147/68747470733a2f2f692d626c6f672e6373646e696d672e636e2f6469726563742f65393030393134626130363634313763613433643138363538616135616662302e706e67)


#### List

List是一个列表展示。比如通讯录、设置等等。并且支持分组展示，内容超出屏幕时，能够滚动，支持下拉刷新：

![](https://camo.githubusercontent.com/f9613fe6d71a73ae50fa5ea59d639a6f3a1d52f054c94a783ea8be658897b1ad/68747470733a2f2f692d626c6f672e6373646e696d672e636e2f6469726563742f36336237316637393034633834383631393933356163373030366363663264352e706e67)


基本用法：

数据集依然使用tab的数据集，也是用Foreach进行初始化：


```
@Component
@Entry
@Preview
struct ListDemoPage{
  @State tabsData:IconBean[] = getIconBeanData();
  build() {
    List(){
      ForEach(this.tabsData,(item:IconBean,index:number)=>{
          ListItem(){
            Text(item.iconName);
          }
      })
    }
  }
}
```

![](https://i-blog.csdnimg.cn/direct/b04aa1f788564d27938ecf8ac8c1c8aa.png)

同时，可以自定义每个Item的样式，通过divider函数为List设置分割线：

```
@Component
@Entry
@Preview
struct ListDemoPage{
  @State tabsData:IconBean[] = getListBeanData();
  build() {
    Column() {
      Row() {
        Text("鸿蒙高级UI组件")
      }

      List() {
        ForEach(this.tabsData, (item: IconBean, index: number) => {
          ListItem() {
            Row() {
              Image($r('app.media.ic_screenshot_thickness'))
                .width(30)
                .aspectRatio(1)
              Text(item.iconName);
            }
          }.padding(10) // 设置每个Item的padding
        })
      }
      .divider({
        strokeWidth: 1,
        startMargin: 30,
        endMargin: 10,
        color: Color.Grey
      }) // 设置List的分割线
    }
  }
}

```

![](https://camo.githubusercontent.com/10221ace7d652f162dc2b4714ec79840851d420501989a1731649210bab10b3a/68747470733a2f2f692d626c6f672e6373646e696d672e636e2f6469726563742f66313764633930306432353234663335623662323931386539666132646439612e706e67)



