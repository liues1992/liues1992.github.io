---
layout: post
title:  "Swift中引用Objc中static常量时的编译错误"
date:   2014-11-05 16:41:31
---
在swift代码中引用objc头文件中的static变量时出现错误:

>Undefined symbols for architecture armv7:
>"_xxxx", referenced from:

常量定义形式为

    static NSString *const = @“xxxx"

找了官方的guide也没看到相关的说明,最后找到的方法是:

在.h中改为

    extern NSString *const;

在.m中改为

    NSString *const = @“xxxx”;
