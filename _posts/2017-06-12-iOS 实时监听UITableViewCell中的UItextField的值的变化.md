---
layout:     post
title:      iOS 实时监听UITableViewCell中的UItextField的值的变化
subtitle:   
date:       2017-06-12
author:     DH
header-img: img/post-bg-miui-ux.jpg
catalog: true
tags:
    - iOS
    - UITableView
    - UItextField
---
# 需求

>项目中有一个需求：每一个UITableViewCell中有一个UItextField，当所有的UItextField输入完成后，点击保存按钮，上传信息。  

在这个过程中，只要任意一个cell中的UItextField的输入发生变化，就要得到UItextField中的值，并对当前控制器中的相应属性进行赋值。其中有两个cell
是选择国家和省市的，点击这两个cell需要弹出一个控制器，点击相应的选择后，发出通知，然后到最开始的控制器接收到通知后，刷新UITableView。 
具体如下图所示：

![](https://ws2.sinaimg.cn/large/006tKfTcgy1fhg8aqlzelj307u0ejq3j.jpg)

点击省/州cell之后，弹出新控制器，选择省州后，到朱控制器进行更新：

![](https://ws4.sinaimg.cn/large/006tKfTcgy1fhg8bgqqltj307u0ejq3k.jpg)

更新主控制，就意味着要调用reloaddata方法，所以我们必须记录每一个cell中的对应的UItextField的值，否则一旦刷新就会是空。


#### 分析

这个时候，自然会想到使用UItextField的代理方法：

```
#pragma mark -监听uitextfield的值得变化
- (BOOL)textField:(UITextField *)textField shouldChangeCharactersInRange:(NSRange)range replacementString:(NSString *)string{
    NSLog(@"textField4 - 正在编辑, 当前输入框内容为: %@",textField.text);
    return YES;
}	

```

但是这个方法有一个问题，他只会监听每次输入或者删除前的值是多少，当修改后，你并不知道最后一个字符改变后，UITextField中的值是多少。

也可能会想到，每次结束编辑的时候，将相应的UITextField中的值传出，这个时候调用：

```

-(BOOL)textFieldShouldEndEditing:(UITextField *)textField
{
    UITableViewCell *cell = (UITableViewCell *)[textField superview];
    NSIndexPath *indexPath = [_mainTableView indexPathForCell:cell];
    if (indexPath.row == 0) {
        _contact_person = textField.text;
    }else if (indexPath.row == 1){
        _phone_number = textField.text;
    }else if(indexPath.row == 2){
        _address1 = textField.text;
    }else if (indexPath.row == 3){
        _address2 = textField.text;
    }else if (indexPath.row == 4){
        _city = textField.text;
    }else if (indexPath.row == 7){
        _postcode = textField.text;
    }

    return YES;
}

```

这个方法的前提是你点击了return按钮，但是在以下情况下，这个函数并不会被调用： 

![](https://ws3.sinaimg.cn/large/006tKfTcgy1fhg8evnm5vj307u0ejab4.jpg)

用户在一个cell输入完成后，直接点击到另一个cell的textField进行输入。

#### 方案

