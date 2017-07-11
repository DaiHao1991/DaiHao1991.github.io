---
layout:     post
title:      封装好的可设置标题、正文字体的默认样式UIAlertController
subtitle:   封装好的UIAlertController
date:       2017-06-14
author:     DH
header-img: img/post-bg-hacker.jpg
catalog: true
tags:
    - iOS
    - UIAlertController
---

>iOS8之后，主要使用UIAlertController进行信息的显示，而默认的样式使用的比较多，所以封装了一个，以方便以后使用。 

![](https://ws1.sinaimg.cn/large/006tKfTcgy1fhg7m7oo1wj30af0j53z7.jpg)

调用以下函数即可，参数需要自己设置：


```

-(void)showIntroDuction:(NSString *)title :(NSString *)showText :(NSString *)btnStr :(UIFont *)titleFont :(UIFont *) showTextFont :(UIColor *) titleColor :(UIColor *)showTextColor :(UIColor *)btnColor
{


    UIAlertController *alertController = [UIAlertController alertControllerWithTitle:title message:showText preferredStyle:UIAlertControllerStyleAlert];

    // 设置标题的字体
    NSMutableAttributedString *attributedStrTitle = [[NSMutableAttributedString alloc]initWithString:title];
    [attributedStrTitle addAttribute:NSFontAttributeName value:titleFont range:NSMakeRange(0, [title length])];
    [attributedStrTitle addAttribute:NSForegroundColorAttributeName value:titleColor range:NSMakeRange(0, [title length])];

    // 利用KVC赋值
    [alertController setValue:attributedStrTitle forKey:@"attributedTitle"];

    // 设置正文的字体
    NSMutableAttributedString *messageAtt = [[NSMutableAttributedString alloc] initWithString:showText];
    [messageAtt addAttribute:NSFontAttributeName value:showTextFont range:NSMakeRange(0, [showText length])];
    [messageAtt addAttribute:NSForegroundColorAttributeName value:showTextColor range:NSMakeRange(0, [showText length])];
    [alertController setValue:messageAtt forKey:@"attributedMessage"];

    // 按钮
    UIAlertAction *alertAction = [UIAlertAction actionWithTitle:btnStr style:UIAlertActionStyleDefault handler:nil];
    [alertController addAction:alertAction];
    // 按钮颜色
    [alertAction setValue:btnColor forKey:@"_titleTextColor"];


    [self presentViewController:alertController animated:YES completion:nil];

}		

```
