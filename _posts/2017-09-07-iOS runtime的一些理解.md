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

而在消息的发送中，编译器会根据情况在objc_msgSend,objc_msgSend_stret,objc_msgSendSuper,objc_msgSendSuper_stret之间进行选择，只会调用一个。

其中：
（1）消息传递给父类会调用带有super的；
（2）消息返回的值是数据结构，而不是简单的值，会调用带有stret的函数。


看看消息发送的图示：

![](https://ws4.sinaimg.cn/large/006tKfTcgy1fje8h7lesqj30fa0fz0tr.jpg)


#### 动态方法解析

上面我们说过一个消息的发送，如果一直寻找到NSObject类也没有找到这个方法，就要调用动态方法解析啦。

首先还是直接上图：

![](https://ws1.sinaimg.cn/large/006tKfTcgy1fje8qf9l33j30go0cwt8z.jpg)

图中包含了三个函数，分别是：resolveInstanceMethod,forwardingTargetForSelector，forwardInvocation。

（1）首先进入resolveInstanceMethod，如果这个函数返回yes，则要利用class_addMethod函数动态的添加方法，消息得到处理，那么就能OK了；如果resolveInstanceMethod返回No，那么就要进入消息的重定向，也就是进入函数forwardingTargetForSelector；

(2) forwardingTargetForSelector是runtime给我们第二次机会去，指定哪个对象去相应selector方法(重定向)。假如返回否个对象，就调用这个对象的方法
，如果返回nil,则进入forwardInvocation（转发）。

(3)这是runtime给的最后一次机会了，我们可以在forwardInvocation方法中修改方法的实现和响应对象。而forwardInvocation函数包含一个参数NSInvocation,我们要通过MethodSignatureForSelector获取方法签名。

如果以上三步，还没有响应的话，那么runtime也救不了你了，程序崩溃。

# runtime应用实例

平时开发，完全感觉不到runtime有什么作用，但是很多人都用过一个字典转模型的框架MJ_extension，这个框架就要用到runtime。下面说一下，我所了解的runtime的应用，可能不全面，希望了解其他应用的朋友可以交流。

#### 字典模型互相转换

按照一般的思想，字典给model赋值要这样写：

```
-(instancetype)initWithDictionary:(NSDictionary *)dict{
    if (self = [super init]) {
        self.age = dict[@"age"];
        self.name = dict[@"name"];
    }
    return self;
}		

```

但是加入这个对象的属性，有一万个呢，这样一个个去写，会让程序员崩溃的。那么这时候runtime就派上用场了。

(1) 字典转模型

```
1.根据字典的key生成set方法，
2.使用msg_send调用setter方法为模型的属性赋值；（也可以用KVC）

//字典转模型
+(id)objectWithKeyValues:(NSDictionary *)aDictionary{
    id objc = [[self alloc] init];
    for (NSString *key in aDictionary.allKeys) {
        id value = aDictionary[key];
        /*判断当前属性是不是Model*/
        objc_property_t property = class_getProperty(self, key.UTF8String);
        unsigned int outCount = 0;
        objc_property_attribute_t *attributeList = property_copyAttributeList(property, &outCount);
        objc_property_attribute_t attribute = attributeList[0];
        NSString *typeString = [NSString stringWithUTF8String:attribute.value];
        if ([typeString isEqualToString:@"@\"TestModel\""]) {
            value = [self objectWithKeyValues:value];
        }
        /**********************/
        //生成setter方法，并用objc_msgSend调用
        NSString *methodName = [NSString stringWithFormat:@"set%@%@:",[key substringToIndex:1].uppercaseString,[key substringFromIndex:1]];
        SEL setter = sel_registerName(methodName.UTF8String);
        if ([objc respondsToSelector:setter]) {
            ((void (*) (id,SEL,id)) objc_msgSend) (objc,setter,value);
        }
    }
    return objc;
}

```

(2) 模型转字典

```
1.调用Class_copyPropertyList获取所有model的属性，
2.调用property_getName取得属性名字；
3.调用属性的getter方法
4.利用msg_send调用getter方法获取值

//模型转字典
-(NSDictionary *)keyValuesWithObject{
    unsigned int outCount = 0;
    objc_property_t *propertyList = class_copyPropertyList([self class], &outCount);
    NSMutableDictionary *dict = [NSMutableDictionary dictionary];
    for (int i = 0; i < outCount; i ++) {
        objc_property_t property = propertyList[i];
        //生成getter方法，并用objc_msgSend调用
        const char *propertyName = property_getName(property);
        SEL getter = sel_registerName(propertyName);
        if ([self respondsToSelector:getter]) {
            id value = ((id (*) (id,SEL)) objc_msgSend) (self,getter);
            /*判断当前属性是不是Model*/
            if ([value isKindOfClass:[self class]] && value) {
                value = [value keyValuesWithObject];
            }
            /**********************/
            if (value) {
                NSString *key = [NSString stringWithUTF8String:propertyName];
                [dict setObject:value forKey:key];
            }
        }
    }
    return dict;
}

```

#### 自动归档

不使用runtime时，归档要这样写：

```

- (void)encodeWithCoder:(NSCoder *)aCoder{
    [aCoder encodeObject:self.name forKey:@"name"];
    [aCoder encodeObject:self.ID forKey:@"ID"];
}
- (id)initWithCoder:(NSCoder *)aDecoder{
    if (self = [super init]) {
        self.ID = [aDecoder decodeObjectForKey:@"ID"];
        self.name = [aDecoder decodeObjectForKey:@"name"];
    }
    return self;
}		

```

而使用了runtime之后：

```

1.使用Class_copyIvarList获取当前model的所有成员变量；

2.使用ival_getName，获取属性的名称；

3.使用KVC赋值和取得值

- (void)encodeWithCoder:(NSCoder *)aCoder{
    unsigned int outCount = 0;
    Ivar *vars = class_copyIvarList([self class], &outCount);
    for (int i = 0; i < outCount; i ++) {
        Ivar var = vars[i];
        const char *name = ivar_getName(var);
        NSString *key = [NSString stringWithUTF8String:name];
        // 注意kvc的特性是，如果能找到key这个属性的setter方法，则调用setter方法
        // 如果找不到setter方法，则查找成员变量key或者成员变量_key，并且为其赋值
        // 所以这里不需要再另外处理成员变量名称的“_”前缀
        id value = [self valueForKey:key];
        [aCoder encodeObject:value forKey:key];
    }
}
- (nullable instancetype)initWithCoder:(NSCoder *)aDecoder{
    if (self = [super init]) {
        unsigned int outCount = 0;
        Ivar *vars = class_copyIvarList([self class], &outCount);
        for (int i = 0; i < outCount; i ++) {
            Ivar var = vars[i];
            const char *name = ivar_getName(var);
            NSString *key = [NSString stringWithUTF8String:name];
            id value = [aDecoder decodeObjectForKey:key];
            [self setValue:value forKey:key];
        }
    }
    return self;
}

```

#### 对象关联

我们知道category不能为对象增加属性，但是我们通过对象关联，可以为category增加自定义属性。

这个我没有用过，后期再补充

# 参考文章

http://www.cocoachina.com/ios/20160523/16386.html

http://www.cnblogs.com/ioshe/p/5489086.html

感谢两位作者的分享。
