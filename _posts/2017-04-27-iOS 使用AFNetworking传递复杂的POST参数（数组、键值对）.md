---
layout:     post
title:      iOS 使用AFNetworking传递复杂的POST参数（数组、键值对）
subtitle:   
date:       2017-04-27
author:     DH
header-img: img/post-bg-coffee.jpeg 
catalog: true
tags:
    - iOS
    - 复杂参数
    - AFNetworking
---
#### 需求

>项目要进行一个购物车价格的计算，需要从客户端传递用户购物车中商品的信息，包括ID和数量，同时 涉及到优惠码。

一般我们用AFNetworking传递参数都是使用如下代码：

```
//  取得管理者
    AFHTTPRequestOperationManager *manager = [AFHTTPRequestOperationManager manager];

// 取得连接
    NSMutableArray *paramDics = ......
    NSString *shoppingCartUrl = @"http://000.000.00.0:8080/api/v1/products";
    [manager GET:shoppingCartUrl parameters:paramDics success:^(AFHTTPRequestOperation *operation, id responseObject) {

        NSMutableArray *dataDics = responseObject[@"data"];
        _collectionArrs = [GCProductModel mj_objectArrayWithKeyValuesArray:dataDics];

        // 刷新页面
        [_mainTableView reloadData];

    } failure:^(AFHTTPRequestOperation *operation, NSError *error) {
        NSLog(@"取得的数据失败----%@",error);
    }];	

```

这个方法可以满足绝大多数情况下的参数传递需求。

但是当参数很复杂的时候，例如： 
![](https://ws3.sinaimg.cn/large/006tNc79gy1fhgvwzgjm8j30lu0emmyd.jpg)

这个参数包含了字典、又包含了数组，比较复杂。直接上代码吧

#### 代码

```
   _manager = [AFHTTPRequestOperationManager manager];
    // 2 设置头部
    _manager.responseSerializer = [AFJSONResponseSerializer serializer];
    _manager.requestSerializer = [AFJSONRequestSerializer serializer];
    NSString *authorization = AUTHORIZTION;
    [_manager.requestSerializer setValue:authorization forHTTPHeaderField:@"Authorization"];

    // 调用接口计算总价
    NSString *caculateTotlePriceStr = @"http://xxxxxxx";

    // 构造参数
    NSMutableArray *paramArrs = [NSMutableArray array];
    for (int i = 0; i < _shoppingCartArrs.count ; i++ ) {
        GCShoppingCartProductModel *model = _shoppingCartArrs[i];

        NSString *productIDStr = [[NSString alloc]initWithFormat:@"%ld",model.productId];
        NSString *quantityStr = [[NSString alloc]initWithFormat:@"%ld",model.quantity];

        NSDictionary *temp = @{@"product_id": productIDStr,@"quantity":quantityStr};

        [paramArrs addObject:temp];
    }

    NSDictionary *couponDic = @{@"couponCode":coupon_code};
    NSDictionary *caculateParamsDic = @{@"products":paramArrs,@"order":couponDic};



    [_manager POST:caculateTotlePriceStr parameters:caculateParamsDic success:^(AFHTTPRequestOperation *operation, id responseObject) {

        NSMutableArray *caculateTotlePriceDic = [NSMutableArray array];
        caculateTotlePriceDic = responseObject[@"data"];
        _detailCellModel = [GCOrderCaculateModel mj_objectWithKeyValues:caculateTotlePriceDic];
        _priceDetailCell.model = _detailCellModel;
        [_mainTableView reloadData];
        [self setSubmitPrice];

    } failure:^(AFHTTPRequestOperation *operation, NSError *error) {
        NSLog(@"取得的数据失败----%@",error);
    }];		

```
