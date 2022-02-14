---
layout: post
title:  swizzling_method 笔记
date: 2015-09-08
Author: aixz
categories:
tags: [笔记, iOS]
comments: true
---

method swizzling

```

+(void)load{
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        Class class = [self class];
        swizzleMethod(class, @selector(viewDidLoad), @selector(aop_ViewDidLoad));
        swizzleMethod(class, @selector(viewDidAppear:), @selector(aop_ViewDidAppear:));
        swizzleMethod(class, @selector(viewWillAppear:), @selector(aop_ViewWillAppear:));
        swizzleMethod(class, @selector(viewWillDisappear:), @selector(aop_ViewWillDisappear:));
    });
}
void swizzleMethod(Class class, SEL originalSelector, SEL  swizzledSelector){
    Method originalMethod = class_getClassMethod(class , originalSelector);
    Method swizzledMethod = class_getClassMethod(class , swizzledSelector);
    BOOL didAddMethod = class_addMethod(class, originalSelector, method_getImplementation(swizzledMethod), method_getTypeEncoding(swizzledMethod));
    if (didAddMethod) {
        class_replaceMethod(class, swizzledSelector, method_getImplementation(originalMethod), method_getTypeEncoding(originalMethod));
    }else{
        method_exchangeImplementations(originalMethod, swizzledMethod);
    }
}
-(void)aop_ViewDidLoad{
    [self aop_ViewDidLoad];
}
-(void)aop_ViewDidAppear:(BOOL)animated{
    [self aop_ViewDidAppear:animated];
}
-(void)aop_ViewWillAppear:(BOOL)animated{
    [self aop_ViewWillAppear:animated];
//    [MobClick beginLogPageView:NSStringFromClass([self class])];
}
-(void)aop_ViewWillDisappear:(BOOL)animated{
    [self aop_ViewWillDisappear:animated];
//    [MobClick endLogPageView:NSStringFromClass([self class])];
}

```