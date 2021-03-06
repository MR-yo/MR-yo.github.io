---
layout: post
author: syea
title: 关于 GYDataCenter 数据库取值问题
# subtitle:
# header-img: 
# header-mask:  
# catalog: 
# multilingual: 
tags:
    - iOS
    - Objective-C
---

## 关于 GYDataCenter 数据库取值问题

今天在使用 `GYDataCenter` ([GYDataCenter]() 是微信读书团队出的一个封装`FMDB`的开源框架) 的时候遇到一个奇怪的错误，以 string 形式存入数据库的字段，取出的时候变成了 number，导致应用某处用`isEqualToString`做判断的时候直接 crash。

接下来就是进行错误的排查：

### 1. 进行多次测试，确定下来出错有哪些可能
通过多次不同机型，不同网络，不同系统版本号情况下的测试，

> 真机测试改变网络情况:设置->开发者->Network Link Conditioner->Very Bed Network


得到的现象总结出来是


* 网络情况良好的情况下，从数据库取出对象，它的 code 字段类型依然是 string

<img src = 'http://owlvwomsh.bkt.clouddn.com/E37AAD79-A5B7-4C03-AB73-308AC4DFADCF.png' width = '300' height = '130'>

* 网络情况很差的情况下，从数据库取出对象，它的 code 字段类型是 number
<img src = 'http://owlvwomsh.bkt.clouddn.com/code.png' width = '300' height = '160'>


得出排查方向 :

1. `NSTaggedPointerString` 什么鬼，是不是它干的好事，id 跟 code 都是这个类型，最后取出来的类型却不一样。
2. 肯定是取值过程中出了问题，一步步断点走下去。

### 2. 错误排查

1. 首先我查了一波 `NSTaggedPointerString` [http://ios.jobbole.com/82456/](http://ios.jobbole.com/82456/) 有点深奥。。没怎么看懂，简单取一些信息展示
  
> 对象在内存中是对齐的，它们的地址总是指针大小的整数倍，通常为16的倍数。对象指针是一个64位的整数，而为了对齐，一些位将永远是零。
> 
> Tagged Pointer利用了这一现状，它使对象指针中非零位有了特殊的含义。在苹果的64位Objective-C实现中，若对象指针的最低有效位为1(即奇数)，则该指针为Tagged Pointer。这种指针不通过解引用isa来获取其所属类，而是通过接下来三位的一个类表的索引。该索引是用来查找所属类是采用Tagged Pointer的哪个类。剩下的60位则留给类来使用。
> 
> Tagged Pointer有一个简单的应用，那就是NSNumber。它使用60位来存储数值。最低位置1。剩下3位为NSNumber的标志。在这个例子中，就可以存储任何所需内存小于60位的数值。
>
>NSString也是如此。对于那些所需内存小于60位的字符串，它可以创建一个Tagged Pointer。其余的则被放置在真正的NSString对象里。这使得常用的短字符串的性能得到明显的提升。实际代码就是如此吗？似乎Apple是这么认为的，因为他们这么做了并实现了它。

就是说iOS在数值小的情况下，会把值放在地址中。是不是它的原因呢，从`NSTaggedPointerString`取值的时候把类型搞错了？感觉不像，但是也清楚如何考证。

2.从取值过程排查
	
一步一步走下去。最后到达关键位置
![](http://owlvwomsh.bkt.clouddn.com/6A0EA14A-13B2-4C9E-B59B-765E31A43E7E.png)

这是 FMDB 的 `FMResultSet.m` 文件

* 在网络很 ok 的情况下 columnType 是 0。最后取出的 returnValue 是 string 类型
* 在网络很不 ok 的情况下 columnType 是 1。最后取出的 returnValue 是 number 类型

<img src = 'http://owlvwomsh.bkt.clouddn.com/timg.jpeg' width = '100' height = '100'>

不是 `sqlite3_column_type` 的问题就是 `[_statement statement]` 的问题

1. 点进去`sqlite3_column_type`

![](http://owlvwomsh.bkt.clouddn.com/4AC01ACD-A226-41BF-867F-B21E42717E90.png)

<img src = 'http://owlvwomsh.bkt.clouddn.com/timg.jpeg' width = '100' height = '100'>

底层 sqlite3.h 文件，应该不会有什么问题。

2. 点进去 `statement` 没什么东西。

bug 要改好上线了 只能先暂时强转 number 解决。以后继续研究。
