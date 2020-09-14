---
layout: post
commentid: ios-view-summary
title: UIView视图小结
category: iOS
tags: iOS,view,ui,hitTest
description:
---

>   UIView视图小结



## 前言

日常开发中，最基础的就是UI视图的展示，其中系统是如何找到能响应的视图的呢，为什么有些页面会存在卡顿的现象呢，下面我们就对UIView相关的知识点做下总结。主要有`UITableView相关`、`事件传递&视图响应`、`图像显示原理`、`卡顿&掉帧`、`绘制原理&异步绘制`、`离屏渲染`、这几个方面。



### UITableView相关

UITableView主要会考察：
- 重用机制
- 数据源同步：多线程下数据同步

> 重用机制

相关代码：
```
cell = [tableView dequeueReusableCellWithIdentifier:identifier];
```

![]({{site.url}}/assets/postImages/ios/view/view01.jpg)


> UI数据源同步

- 常见于新闻、资讯类的App当中

![]({{site.url}}/assets/postImages/ios/view/view02.jpg)

- 数据源同步解决方案
- 这里介绍两种方案：并发数据拷贝和串行访问

- 第一种：并发访问、数据拷贝

![]({{site.url}}/assets/postImages/ios/view/view03.jpg)

- 可能存在的弊端：需要记录删除动作并同步；拷贝大量数据，存在内存开销问题

- 第二种：串行访问

![]({{site.url}}/assets/postImages/ios/view/view04.jpg)

- 可能存在的弊端：若子线程的预操作耗时长，会出现卡顿的现象



### 事件传递&视图响应

- 我们先来看下UIView和CALayer的关系和区别

![]({{site.url}}/assets/postImages/ios/view/view05.jpg)

- 关系：
	- UIView为其提供内容，以及负责处理触摸等事件，参与响应者链
	- CALayer负责显示内容contents


> 事件传递(必备)与视图响应链

- 常见的视图布局

![]({{site.url}}/assets/postImages/ios/view/view06.jpg)

- 事件传递

- 主要涉及的函数:
	- -(UIView *)hitTest:(CGPoint)point withEvent:(UIEvent *)event;：返回值表示哪个视图可以响应事件
	- -(BOOL)pointInside:(CGPoint)point withEvent:(UIEvent *)event;：判断点击的位置是否在视图范围内

- 事件传递的流程

![]({{site.url}}/assets/postImages/ios/view/view07.jpg)

- 如果找不到响应的视图，但是是在UIWindow的范围内，则UIWindow就作为响应事件的视图


- hitTest:withEvent:的系统实现

![]({{site.url}}/assets/postImages/ios/view/view08.jpg)

- 在子视图中查找，采用的是倒序查找


- 应用场景：
	- 方形按钮指定区域接受事件响应
		- 在pointInside判断point是否在区域呢
		- 在hitTest中寻找最终的响应视图


> 视图事件响应

![]({{site.url}}/assets/postImages/ios/view/view09.jpg)

- 主要涉及的函数:
	- - (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event;
	- - (void)touchesMoved:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event;
	- - (void)touchesEnded:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event;

![]({{site.url}}/assets/postImages/ios/view/view10.jpg)

- 若响应传递到UIApplication中，仍没有视图处理事件？
	- 忽略这个事件，什么反应都没有



### 图像显示原理

- 主要为UI掉帧和卡顿做技术铺垫

![]({{site.url}}/assets/postImages/ios/view/view11.jpg)

- 一般CPU中会输出的是`位图`，在GPU中会对位图做图层的渲染，包括纹理的合成；之后将其放入`帧缓冲区`中。之后视频控制器会提取`帧缓冲区`的内容，并展示在手机的显示器上。

- UIView展示内容的过程

![]({{site.url}}/assets/postImages/ios/view/view12.jpg)

- CPU的工作

![]({{site.url}}/assets/postImages/ios/view/view13.jpg)

- Layout：UI布局、文本计算
- Display：绘制(drawRect)
- Prepare：图片编解码
- Commit：提交位图


- GPU渲染管线的内容

![]({{site.url}}/assets/postImages/ios/view/view14.jpg)



### UI卡顿、掉帧的原因

- 先看下画面的渲染时间：

![]({{site.url}}/assets/postImages/ios/view/view15.jpg)

- 1秒钟有60帧画面更新；16.7ms = 1000ms / 60
卡顿、掉帧的原因：在下一帧画面到来时，CPU和GPU并未在16.7ms内完成画面的合成，于是就导致卡顿、掉帧


 > 滑动优化方案

- 基于TableView、ScrollView都有哪些优化方案？
- 从CPU角度：
	- 对象的创建、调整、销毁(放到子线程去处理)
	- 预判版处理(布局计算、文本计算放到子线程去处理)
	- 预渲染(文本等异步绘制，图片编解码等)

- 从GPU角度：
	- 纹理渲染：尽量避免离屏渲染
	- 视图混合：减少视图层级的复杂性，减轻合成的工作量



### UIView的绘制原理(高级考察点)

- 先看下流程图：

![]({{site.url}}/assets/postImages/ios/view/view16.jpg)

- setNeedsDisplay：并没有立即进行视图的绘制工作，而是在之后的某一时机才进行视图的绘制工作
	- 相当于在layer上打上了脏标记
	- 在当前RunLoop将要结束的时候，会调用`[CALayer display]`方法，真正的进行视图的绘制工作


- 下面我们看下`系统绘制`的流程

![]({{site.url}}/assets/postImages/ios/view/view17.jpg)

- Context：视图、控件的上下文

- 一句话：CALayer内部会创建一个上下文，然后调用[UIView drawRect:]方法，最后将绘制好的位图上传给GPU，这就结束了UI绘制的过程


- 我们再看下`异步绘制`的流程：怎样进行异步绘制的

- 其实是基于系统给我们的方法：需要实现此方法
	- - [layer.delegate displayLayer:]
	- 代理负责生成对应的bitmap
	- 设置改bitmap作为layer.contents属性的值

- 我们看下时序图

![]({{site.url}}/assets/postImages/ios/view/view18.jpg)

- 一句话：在主队列中设置了setNeedsDisplay方法，再调用CALayer的display方法时，若其代理实现了displayLayer方法，则去全局队列中创建位图，此时主队列正常工作，后面全局队列会返回创建的位图到主队列中，并设置CALayer的contents内容，从而完成绘制



### 离屏渲染(源于GPU层面)

- 什么是离屏渲染，怎样去理解离屏渲染

- 有离屏渲染，就有`在屏渲染`。我们来看下概念：
	- 在屏渲染：On-Screen Rendering
		- 意为当前屏幕渲染，指的是`GPU`的渲染操作是在当前用于显示的屏幕缓冲区中进行
	- 离屏渲染：Off-Screen Rendering
		- 意为离屏渲染，指的是`GPU`在当前屏幕缓冲区以外`新开辟`一个缓冲区进行渲染操作

- 离屏渲染通俗的讲：
	- 离屏渲染起源于GPU层面，当我们设置某一项UI视图的图层属性，视图的效果不能直接呈现给屏幕，而需要在别的地方做额外的处理预合成，这就会触发离屏渲染

- 何时会触发离屏渲染？
	- 设置圆角(当和maskToBounds一起使用时)
	- 图层蒙版
	- 视图的阴影
	- 光栅化

- 为何要避免离屏渲染？
	- 在触发离屏渲染的时候，会增加GPU的工作量，很可能会导致CPU、GPU总耗时超出一帧的时间，可能就会导致UI的卡顿和掉帧，所以要避免离屏渲染。






### 参考文章
- [iOS 关于离屏渲染的理解 以及解决方案
](https://www.jianshu.com/p/cff0d1b3c915)



## 最后

如果对大家有帮助，请[github上follow和star](https://github.com/jifengchao)，本文发布在[戴超的技术博客](https://jifengchao.github.io/)，转载请注明出处
