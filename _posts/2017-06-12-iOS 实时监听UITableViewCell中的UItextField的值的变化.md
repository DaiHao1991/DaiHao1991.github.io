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

最后，为了能够实时的监听textField的输入的改变，首先给UItextField添加一个编辑事件，每次这个事件触发一个函数，在相应的函数
中得到UItextField的值。当然你也可以在这个函数中做你想要的处理，比如将UItextField的值以通知的形式发送给相关的控制器，以便
做下一步处理。而在本项目中，只需要实时的记录每一个cell中的UItextField的值就可以。 

#### 代码，构建cell

```
#pragma mark -构建cell
-(UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath
{
    if (indexPath.row == 0) {
        GCAddAddressTableViewCell1 *cell = [tableView dequeueReusableCellWithIdentifier:@"cell1"];
        cell.selectionStyle = UITableViewCellSelectionStyleNone;
        cell.textField.delegate = self;
        cell.label.text = @"收货人";
//        cell.textField.text = _model.contactPerson;
        cell.textField.text = _contact_person;
        cell.textField.tag = indexPath.row;
        [cell.textField addTarget:self action:@selector(changedTextField:) forControlEvents:UIControlEventEditingChanged];
        return cell;
    }else if (indexPath.row == 1){
        GCAddAddressTableViewCell1 *cell = [tableView dequeueReusableCellWithIdentifier:@"cell1"];
        cell.selectionStyle = UITableViewCellSelectionStyleNone;
        cell.textField.delegate = self;
        cell.textField.keyboardType = UIKeyboardTypeNumberPad;
//        cell.textField.text = _model.phoneNumber;
        cell.textField.text = _phone_number;
        cell.textField.tag = indexPath.row;
        [cell.textField addTarget:self action:@selector(changedTextField:) forControlEvents:UIControlEventEditingChanged];
        cell.label.text = @"联系电话";
        return cell;
    }else if (indexPath.row == 2){
        GCAddAddressTableViewCell1 *cell = [tableView dequeueReusableCellWithIdentifier:@"cell1"];
        cell.selectionStyle = UITableViewCellSelectionStyleNone;
        cell.textField.delegate = self;
        cell.textField.placeholder = @"门牌号／楼层／楼号／街道";
//        cell.textField.text = _model.address1;
        cell.textField.text = _address1;
        cell.textField.tag = indexPath.row;
        [cell.textField addTarget:self action:@selector(changedTextField:) forControlEvents:UIControlEventEditingChanged];
        cell.label.text = @"地址一";
        return cell;
    }else if (indexPath.row == 3){
        GCAddAddressTableViewCell1 *cell = [tableView dequeueReusableCellWithIdentifier:@"cell1"];
        cell.selectionStyle = UITableViewCellSelectionStyleNone;
        cell.textField.delegate = self;
        cell.textField.placeholder = @"区域／公司／公寓／社区";
//        cell.textField.text = _model.address2;
        cell.textField.text = _address2;
        cell.textField.tag = indexPath.row;
        [cell.textField addTarget:self action:@selector(changedTextField:) forControlEvents:UIControlEventEditingChanged];
        cell.label.text = @"地址二";
        return cell;
    }else if (indexPath.row == 4){
        GCAddAddressTableViewCell1 *cell = [tableView dequeueReusableCellWithIdentifier:@"cell1"];
        cell.selectionStyle = UITableViewCellSelectionStyleNone;
        cell.textField.delegate = self;
//        cell.textField.text = _model.city;
        cell.textField.text = _city;
        cell.textField.tag = indexPath.row;
        [cell.textField addTarget:self action:@selector(changedTextField:) forControlEvents:UIControlEventEditingChanged];
        cell.label.text = @"城市";
        return cell;
    }else if (indexPath.row == 5){
        GCAddAddressTableViewCell2 *cell = [tableView dequeueReusableCellWithIdentifier:@"cell2"];
        cell.selectionStyle = UITableViewCellSelectionStyleNone;
        cell.label.text = @"省/州";
//        _provence = _model.province;
        cell.countryLabel.text = _provence;
        return cell;
    }else if (indexPath.row == 6){
        GCAddAddressTableViewCell2 *cell = [tableView dequeueReusableCellWithIdentifier:@"cell2"];
        cell.selectionStyle = UITableViewCellSelectionStyleNone;
        cell.label.text = @"国家";
//        _countryTemp = _model.country;
        cell.countryLabel.text = _countryTemp;

        return cell;
    }else{
        GCAddAddressTableViewCell1 *cell = [tableView dequeueReusableCellWithIdentifier:@"cell1"];
        cell.selectionStyle = UITableViewCellSelectionStyleNone;
        cell.textField.delegate = self;
        cell.label.text = @"邮编";
        cell.textField.keyboardType = UIKeyboardTypeNumberPad;
//        cell.textField.text = _model.postcode;
        cell.textField.tag = indexPath.row;
        cell.textField.text = _postcode;
        [cell.textField addTarget:self action:@selector(changedTextField:) forControlEvents:UIControlEventEditingChanged];

        return cell;
    }


}		

```

在构建cell的时候，就对每一个cell中的UItextField控件设置tag，tag的值是cell的indexPath.row。这样你可以在后期的工作中得到对应的cell。通知给每一个UItextField增加一个UIControlEventEditingChanged事件，每当修改时，就触发changedTextField:这个函数。在这个函数中，我们就可以实时得到UItextField的值，并进行保存。

```

#pragma mark -给每个cell中的textfield添加事件，只要值改变就调用此函数
-(void)changedTextField:(id)sender
{
    UITextField *tempTextField = (UITextField *)sender;
    NSLog(@"tag------%ld;值是---%@",tempTextField.tag,tempTextField.text);

    if (tempTextField.tag == 0) {
        _contact_person = tempTextField.text;
    }else if (tempTextField.tag == 1){
        _phone_number = tempTextField.text;
    }else if(tempTextField.tag == 2){
        _address1 = tempTextField.text;
    }else if (tempTextField.tag == 3){
        _address2 = tempTextField.text;
    }else if (tempTextField.tag == 4){
        _city = tempTextField.text;
    }else if (tempTextField.tag == 7){
        _postcode = tempTextField.text;
    }

}

```
