---
layout: post
title: 线性回归笔记
date: 2020-11-29
Author: aixz
categories:
tags: [笔记, 深度学习, 线性回归]
comments: true
---


###线性回归笔记

######记录学习过程，MXNET中的方法函数使用，含义等

激活函数 sigmod 函数


$\hat{y}$

损失函数 
```
我们期望损失函数的值越小越好
```


L($\hat{y}$,y) = \frac{1}{2} * ($\hat{y}$ - y)^2

L($\hat{y}$,y) = -(y * \lg{$\hat{y}$} + (1 - y) *\lg{1 - $\hat{y}$})

