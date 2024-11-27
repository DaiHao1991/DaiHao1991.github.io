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
## Tabs
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


## List

### 基本用法

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

![](https://camo.githubusercontent.com/da27eff4809b565f32b7e8e1065e2639ffc4c76a26a5551af4c348e449c1541f/68747470733a2f2f692d626c6f672e6373646e696d672e636e2f6469726563742f62303461613166373838353634643237393338656366386163386331633861612e706e67)

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

List常用的用法是从网络请求到数据，然后渲染列表：

```
import axios, { Axios, AxiosResponse } from '@ohos/axios';
import { SongBean } from '../model/MusicData';

@Component
@Entry
struct AxiosDemoPage{
  url:string = "数据源";

   @State songList:SongBean[] = [];
  async getSongs(){
    const response:AxiosResponse = await axios.get<string,AxiosResponse<SongBean>,null>(this.url);
    this.songList = response.data;
    console.log("david",JSON.stringify(response))
  }

  build() {

    Column() {
      Button("请求网络数据")
        .onClick(() => {
          this.getSongs()
        })

      List() {
        ForEach(this.songList, (item: SongBean, index: number) => {
          ListItem() {
            Row() {
              Image(item.img).width(30)
                .aspectRatio(1)
              Text(item.name)
            }
          }.padding(10)
        })
      }.divider({strokeWidth:1,startMargin:30,endMargin:5,color:Color.Gray})
    }.padding(10)
  }
}
```

代码中getSongs函数异步请求一个接口，当数据返回时，网络库对数据完成解析，并对被状态变量修饰的数组songList赋值，状态改变，从而List进行渲染：

![](https://camo.githubusercontent.com/91c2c4c0143fab6ba1ae7dd2638e641a1c44c439209b497b34fde8937030ada0/68747470733a2f2f692d626c6f672e6373646e696d672e636e2f6469726563742f30613035373462333365313934343733386139346230336361656630346335382e706e67)

### 上拉刷新实现

在线上app中，List一个非常常见的功能就是上拉刷线。因为现在后端接口都会做分页，减少服务器端的压力，在上拉或下拉的时候，刷新或者加载更多，对List进行页面刷新。鸿蒙上实现有3种方式

#### 原生实现

思路：

1. 监听手指触摸事件，记录初始触摸位置；

2. 监听手势滑动事件，如果滑动距离 > 0 ，则表示向下移动；

3. 设置一个阈值，如果移动的距离大于阈值，则进行网络请求，并展示下拉刷新图标。

   这种思路是鸿蒙官网的例子使用的思路，较为复杂。鸿蒙还专门提供了一种刷新组件：refresh


#### Refresh实现

Refresh 组件使用起来就很方便。

1. 列表的外层使用Refresh进行包裹；

2. 定义刷新变量，用于控制刷新图标的展示和隐藏；

3. 监听事件，进行相应的数据处理逻辑。

需要注意其中的两个方法：

1. onStateChange：该方法在列表状态发生变化时回调，例如下拉、正在刷新等，可以找到对于的时机做一些处理；

2. onRefreshing：下拉刷新的回调，需要做两件事情，第一是网络请求，拿到数据后进行相应逻辑处理，第二是在网络请求完毕后，无论网络请求成功还是失败，都修改Refresh的状态，结束刷新；

例子中，点击按钮，获取数据，并刷新列表，下来列表时，在列表状态改变时，清空列表。在刷新时进行网络请求，请求结束后刷新列表，改变刷新状态：

```
import axios, { Axios, AxiosResponse } from '@ohos/axios';
import { SongBean } from '../model/MusicData';

@Component
@Entry
struct AxiosDemoPage{
  url:string = "数据源";

  @State songList:SongBean[] = [];


  /**
   * 控制是否刷新,初始化时，需要为false，通过引用传递至Refresh组件内部，会自动改变值
   */
  @State isRefresh:boolean = false;

  async getSongs(){
    const response:AxiosResponse = await axios.get<string,AxiosResponse<SongBean>,null>(this.url);
    this.songList = response.data;
    console.log("david",JSON.stringify(response))
  }

  build() {


    Column() {
      Button("请求网络数据")
        .onClick(() => {
          this.getSongs()
        })
      Refresh({refreshing:$$this.isRefresh}) {
        List() {
          ForEach(this.songList, (item: SongBean, index: number) => {
            ListItem() {
              Row() {
                Image(item.img).width(30)
                  .aspectRatio(1)
                Text(item.name)
              }
            }.padding(10)
            .swipeAction({
              end: this.listItemEnd(index)
            })
          })
        }
        .divider({
          strokeWidth: 1,
          startMargin: 30,
          endMargin: 5,
          color: Color.Gray
        }).padding(10)
      }.onStateChange((status : RefreshStatus)=>{
        console.info("divid statues = ",status)
        if (status == RefreshStatus.Drag) {
            this.songList = [];
          console.info("divid statues Drag= ",status)
        }
      }).onRefreshing(async ()=>{
        console.info("divid statues 开始请求")
        await this.getSongs();
        this.isRefresh = false
      })
    }
  }

  @Builder listItemEnd(index:number){
    Row(){
      Button("删除")
        .onClick(()=>{
          this.songList.splice(index,1);
        })
    }
  }
}

```


#### 第三方控件

在鸿蒙的开源库中有一个封装好的上下刷新、下拉加载的组件，可以参考：

https://ohpm.openharmony.cn/#/cn/result?sortedType=likes&page=1&q=refresh


