---
layout: post
title: JNI笔记
date: 2021-11-04
Author: aixz
categories:
tags: [笔记, Android，JNI]
comments: true
---

### android PhoneWindow的创建时机



Activity启动过程最后 在ActivityThread的performLaunchActivity中会调用activit.attach方法

其中 有:

>mWindow = new PhoneWindow(this, window, activityConfigCallback);

可以发现每个activity在创建的时候都会新建一个PhoneWindow实例。

其他Alert等也会创建PhoneWindow实例
