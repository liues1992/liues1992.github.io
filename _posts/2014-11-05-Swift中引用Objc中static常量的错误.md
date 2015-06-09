---
layout: post
title:  "Swift中引用Objc中static常量时的编译错误"
date:   2014-11-05 16:41:31
---
在swift代码中引用objc头文件中的static变量时出现错误:

>Undefined symbols for architecture armv7:
>"_xxxx", referenced from:

常量定义形式为

    static NSString *const kXXX = @“xxxx"

找了官方的guide也没看到相关的说明,最后找到的方法是:

在.h中改为

    extern NSString *const kXXX;

在.m中改为

    NSString *const kXXX = @“xxxx”;

后者应该说是正确的常量定义形式。用我之前的方法通常程序不会出错，不过其实每个不同文件引用到的都是不同的static变量，虽然他们的值都是一样的（对应固定的字符串常量地址）。
后者则是引用一个全局唯一的全局常量。