---
layout:     post
title:      iOS AFNetWorking与线程同步
subtitle:   
date:       2017-05-15
author:     DH
header-img: img/post-bg-rwd.jpg
catalog: true
tags:
    - iOS
    - 线程同步
    - AFNetWorking
---
#### 需求

>在iOS开发中，页面的数据并不是通过一个接口进行获取的，有时候需要从服务器的多个接口获取数据，然后进行页面的更行，有时候
需要在获取一个接口的数据后，根据得到的数据再获取其他接口的数据。

而开源框架AFNetWorking是一个常用的强大的第三方框架，我们可以利用AFNetWorking和GCD结合的方式完成以上两个需求。

#### 方式一

获取多个接口数据后，创建或者reload UITableview。将网络获取数据的操作放到线程组中，当每个线程都完成了后，在主线程根新页面。

![](https://ws2.sinaimg.cn/large/006tKfTcgy1fhgt11jk09j30af082my5.jpg)

1和2的数据来自两个不同的接口。项目要求，一起加载。那么我们就可以利用线程组进行解决。

#### 方式一代码

```

#pragma mark -将取得用户收货地址和商品列表放入队列中
-(void)getInitInfoFromServer
{
    dispatch_group_t group = dispatch_group_create();


    // 2 设置头部
    _manager.responseSerializer = [AFJSONResponseSerializer serializer];
    _manager.requestSerializer = [AFJSONRequestSerializer serializer];


    GCToken *token = [GCTokenManager getToken];
    NSString *tokenStr = [GCTokenManager getFullToken:token];
    [_manager.requestSerializer setValue:tokenStr forHTTPHeaderField:@"Authorization"];

    //Enter group
    dispatch_group_enter(group);
    // 获取默认
    NSString *addressStr = @"http://XXXXXXXXXXXXXXXXX";
    [_manager GET:addressStr parameters:nil success:^(AFHTTPRequestOperation *operation, id responseObject) {

        NSMutableArray *dataDics = responseObject[@"data"];

        _defaultAddressModel = [GCAddressModel mj_objectWithKeyValues:dataDics];

        dispatch_group_leave(group);
    } failure:^(AFHTTPRequestOperation *operation, NSError *error) {
        NSLog(@"取得的收货地址列表失败----%@",error);
        dispatch_group_leave(group);
    }];


    dispatch_group_enter(group);
    NSString *shoppingCartUrl = @"http://XXXXXXXXXXXX";
    [_manager GET:shoppingCartUrl parameters:nil success:^(AFHTTPRequestOperation *operation, id responseObject) {

        NSMutableArray *dataDics = responseObject[@"items"];

        _shoppingCartArrs = [GCShoppingCartProductModel mj_objectArrayWithKeyValuesArray:dataDics];


        dispatch_group_leave(group);
    } failure:^(AFHTTPRequestOperation *operation, NSError *error) {
        NSLog(@"取得的购物车数据失败----%@",error);
        dispatch_group_leave(group);
    }];


    dispatch_group_notify(group, dispatch_get_main_queue(), ^(){
        //初始化页面或更新页面
        [self initAllView];
    });

}		

```

#### 方式二

需要在获取一个接口的数据后，根据得到的数据再获取其他接口的数据。

有一个需求是修改用户收货地址，保存并使用，保存收货地址是一个接口，而后取得收货地址更新页面。在这里有两种做法，第一种是
保存收货地址成功后，在回调的block中修改模型，并跳转到相应页面。第二种是，在修改模型的接口，修改模型成功后，在调用获取
收货地址的接口，获取到收货地址，并进行下一步操作。 
假如项目要求使用两次与服务器进行交互的这种方式，我们依然可以使用线程组进行简单的解决。

现在有具体需求： 
点击收货地址，将收货地址设置为默认收货地址，并重新更新页面。

![](https://ws2.sinaimg.cn/large/006tKfTcgy1fhgt3oe9qdj30af0j5dgn.jpg)

在设置默认的收货地址成功后，在回调函数中直接进行收货地址列表的获取，并更新页面，在这一些列操作完成后，利用
dispatch_group_notify(group, dispatch_get_main_queue() 进行其他想要的操作，类似于通知。


#### 方式二代码

```

-(void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath
{
    _HUD = [MBProgressHUD showHUDAddedTo:self.navigationController.view animated:YES];
    _HUD.labelText = @"请稍等";

    dispatch_group_t group = dispatch_group_create();

    dispatch_group_enter(group);
    _selectedIndex = (int)indexPath.row;
    // 取得连接
    GCAddressModel *model = _addressArrs[indexPath.row];
    NSString *defaultAddressStr = [[NSString alloc]initWithFormat:@"http://XXXXXXXX;

    [_manager POST:defaultAddressStr parameters:nil success:^(AFHTTPRequestOperation *operation, id responseObject) {

        NSString *addressStr = @"http://XXXXXXXX";
        [_manager GET:addressStr parameters:nil success:^(AFHTTPRequestOperation *operation, id responseObject) {

            NSMutableArray *dataDics = responseObject[@"data"];

            _addressArrs = [GCAddressModel mj_objectArrayWithKeyValuesArray:dataDics];


            [_mainTableView reloadData];


        } failure:^(AFHTTPRequestOperation *operation, NSError *error) {
            NSLog(@"获取地址失败--%@",error);
        }];

        dispatch_group_leave(group);

    } failure:^(AFHTTPRequestOperation *operation, NSError *error) {
        dispatch_group_leave(group);
        NSLog(@"设置默认收货地址失败");

    }];




    dispatch_group_notify(group, dispatch_get_main_queue(), ^{
        [_HUD removeFromSuperview];
        _HUD  = nil;
        // 发送通知
        [[NSNotificationCenter defaultCenter]postNotificationName:@"editDefaultAddress" object:nil];

    });

}		

```
