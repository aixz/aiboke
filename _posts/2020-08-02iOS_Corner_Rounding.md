---
layout: post
title:  iOS Corner Rounding
date: 2020-08-01
Author: aixz
categories:
tags: [笔记, iOS]
comments: true
---

在 iOS UI中 当我们想要实现 圆角，我们第一反应是使用 CALayer.cornerRadius 属性。但是，因为这个简便的属性会极大的消耗性能，所以我强烈建议仅当你没有其它选择的时候才使用它。
**1.为什么你不应该使用 CALayer.cornerRadius****2.其他实现圆角的高性能方法以及使用的最佳时机****3.选择绘制圆角策略的流程图**
CALayer.cornerRadius 使用的代价是非常大的   为什么.cornerRadius 的开销非常大？当界面滑动时，使用CALayer.cornerRadius 属性执行裁剪操作会在 60 FPS的每一帧 都触发离屏渲染，即使展示区域的内容没有变更。这意味着 在将所有layer以及使用了.cornerRadius的layer进行合并渲染时， GPU会在每一帧绘制的时候都切换context去进行离屏渲染。最重要的是，离屏渲染的开销没有在 Time Profiler 中展示，因为CoreAnimation Render Server 替你将离屏渲染的工作做完了，而离屏渲染又会极大的消耗设备性能。在 iPhone 4、iPhone 4S和5/5C（连同 iPads/iPods）设备上性能消耗的尤其严重，而在 iPhone 5S及之后的设备上因为会降低最大消耗值(headrom)以降低掉帧的情况，所以你几乎无法察觉到离屏渲染所带来的性能影响。
Corner Rounding 策略选择
裁剪圆角这个策略是将四个不透明圆角分别覆盖在四个角上，这是一个很取巧的方法，性能也非常好。由CPU创建四个小的分开的圆角layer，分置于四个角。

裁剪圆角适用于以下两个情况下：1，当圆角有很多元素交叉时2，在圆角上方有文字或者图片背景时，使用圆角剪切覆盖的方式很有效。
使用CALayer.cornerRadius属性是否总是有用？
在一些很少的情况下使用。cornerRadius是非常合适的。例如有动画内容在圆角content或底部执行，在动画执行过程中这是不可避免的，但是在大多数情况下，你可以很容易就通过设计避免圆角corner出现类似上面讨论的那几种交互情况。
如果屏幕上没有变动的内容，使用.cornerRadius很方便。但是一旦在屏幕上有移动，即使这些移动不会影响圆角，一样会造成,cornerRadius发生额外的性能消耗。举个例子，在navigation bar 上有一个圆形的元素，然后navigatoni bar 底部有滑动view的时候会影响，即使他们没有重叠，任何在屏幕上的动画，即使没有交互，也会造成影响。另外，任何类型的屏幕刷新都会造成圆角剪切的消耗。
光栅化及layer关联
有些朋友建议使用CALayer.shoulldRasterize可以提高使用.cornerRadius属性的效率。不完全了解这个操作而去使用，通常都是很危险的。如果没有能够引起光栅化的操作（没有移动，没有点击改变颜色，没有在tableview傻姑娘滑动等）那么可以使用它。通常我们不鼓励使用它，因为这非常容易引起糟糕的性能。如果对app 构架没有很好的掌握，却坚持使用.cornerRadius（他们的app也不是很好），这会有很大的不同。然后，如果你从基础开始搭建你的app，我强烈建议你选择以下几个策略去剪切圆角。