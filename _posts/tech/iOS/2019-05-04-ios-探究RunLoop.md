---
layout: post
commentid: ios-RunLoop-summary
title: 探究RunLoop
category: iOS
tags: iOS,RunLoop
description:
---

>   探究RunLoop



## 前言

今天我们就思考下，一个程序从启动运行到最终退出的整个过程中，到底发生了什么事。这其实是事件循环RunLoop保证了程序可以一直运行下去。本文会从`RunLoop的概念`、`数据结构`、`事件循环机制`、`NSRunLoop与NSTimer`、`NSRunLoop与多线程关系`这几方面来总结下。



### 概念

- 什么是RunLoop？
	- 是通过内部维护的`事件循环`来对`事件/消息`进行管理的一个对象

- 事件循环概念：
(维护的事件循环可以用来不断处理消息或者事件，然后对它们进行管理;
绝对不止是死循环这么简单，实质上就是runloop内部状态的转换)
	- 没有消息处理时，休眠以避免资源占用
		- 状态的切换：从`用户态`到`内核态`；
	- 有消息需要处理时，立刻被唤醒
		- 状态的切换：从`内核态`到`用户态`；

	- 用户态：应用程序一般都是运行在用户态上，平时开发用到的api等都是用户态的操作
	- 内核态：系统调用，牵涉到操作系统资源调度，底层内核的相关指令



Q：为什么我们的main函数能保持一直运行状态而不退出呢？

- 在main函数中，会调用UIApplicationMain函数，在这个函数内部会启动主线程的RunLoop。而RunLoop会对事件循环进行维护，可以做到有事情的时候做事，没事情的时候休眠，中间其实是状态的切换。

![]({{site.url}}/assets/postImages/ios/runloop/runloop01.jpg)

- 等待：其实就是用户态-内核态的转换。注意--事件循环不是while死循环，而是状态转换。



### 数据结构

- 在OC中，NSRunLoop是CFRunLoop的封装，提供了面向对象的API。
- RunLoop相关的数据结构，主要有3个：
	- CFRunLoop
	- CFRunLoopMode
	- Source/Timer/Observer


#### CFRunLoop(开源的: [opensource](http://opensource.apple.com/source/CF/CF-1151.16/))

![]({{site.url}}/assets/postImages/ios/runloop/runloop02.jpg)

- pthread：C语言里的线程对象，代表runloop和线程的关系是一一对应的
- currentMode：代表runloop当前所处的模式，其实是CFRunLoopMode这样的数据结构
- modes：多个mode的集合，其实就是CFRunLoopMode的集合
- `commonModes`：里面是NSString的集合，与modes是不一样的
- commonModeItems：其实是一个集合，里面包含了多个Source/Timer/Observer


##### CFRunLoopMode

![]({{site.url}}/assets/postImages/ios/runloop/runloop03.jpg)

- name：模式名称，如NSDefaultRunLoopModel；别名定义，可通过字符串名称去找到对应的mode
- source0、source1：都是集合类型的
- observers、timers：都是数组类型的



> Source的数据结构

- 在CF框架中，全名是CFRunLoopSource
	- source0：需要`手动唤醒`线程；用于用户主动触发的事件，常见的source事件：比如用户点击按钮，拖拽，手势等事件。
	- source1：具备`唤醒`线程能力；通过内核和其它线程相互发送消息



> Timer的数据结构

- 在CF框架中，全名是CFRunLoopTimer
	- 基于事件的定时器
	- 和NSTimer是toll-free bridged的：免费桥转换的



##### 观测时间点：我们可以注册一些observer，来对RunLoop的相关时间点进行观察

- 有哪些时间点呢：
	- kCFRunLoopEntry：RunLoop的入口时机，当RunLoop准备启动的时候，系统会给我们一个回调通知
	- kCFRunLoopBeforeTimers：通知观察者，RunLoop要对Timers相关事件进行处理了
	- kCFRunLoopBeforeSources：通知观察者，要处理Sources相关事件了
	- `kCFRunLoopBeforeWaiting`：通知对应观察者，RunLoop即将要进入休眠状态
	- `kCFRunLoopAfterWaiting`：通知对应观察者，用户态切换到内核态不久
	- kCFRunLoopExit：代表RunLoop退出的通知
	- kCFRunloopAllActivities：观察所有状态改变



#### 各个数据结构之间的关系

![]({{site.url}}/assets/postImages/ios/runloop/runloop04.jpg)

- 一个runloop对应了多种mode ，每个mode下又有多种source/timer/Observer



#### RunLoop的Mode

![]({{site.url}}/assets/postImages/ios/runloop/runloop05.jpg)

- 每次RunLoop启动时，只能指定其中一个 Mode，这个Mode被称作CurrentMode
- 如果需要切换Mode，只能退出Loop，再重新指定一个Mode进入，这样做主要是为了分隔开不同组的Source/Timer/Observer，让其互不影响

- Q：为什么需要这么多的Mode
	- 当我们的RunLoop运行在某个mode上的时候，只能接收当前mode下的事件，而其他mode上的observer、timer的事件的回调是无法接收到的；所以有多个Mode实际上就是起到屏蔽的作用。



#### CommonMode的特殊性

- NSRunLoopCommonModes是怎样理解的？(是同步Source/Timer/Observer到多个Mode的`一种技术方案`)
	- CommonModes`不是实际存在的`一种Mode
	- 是同步Source/Timer/Observer到多个Mode的`一种技术方案`




### 事件循环的实现机制

- 在我们实际开发过程所调用的NSRunLoopRun和CFRunLoopRun方法最终都会调用到`void CFRunLoopRun()`方法
- 过程：

![]({{site.url}}/assets/postImages/ios/runloop/runloop06.jpg)

- Q：当处于休眠的RunLoop，我们能通过哪些方式来唤醒？(3种)
	- Source1
	- Timer事件
	- 外部手动唤醒

- Q：当我们点击App的图标，从程序启动运行到最终退出，在这个过程中，系统都发生了什么？
	- 调用main函数，mian函数会调用UIApplicationMain函数，UIApplicationMain函数中会启动主线程的runloop；
	- 在处理完事件后，会调用 mach_msg() 函数，经过系统调用，系统就会进入`内核态`；
	- 当有外部事件唤醒线程时，会触发mach_msg()回调到`用户态`去处理事件。

![]({{site.url}}/assets/postImages/ios/runloop/runloop07.jpg)





### RunLoop与NSTimer

- 场景：滑动TableView的时候我们的定时器还会生效么？
	- 答案：不生效
	- 原因：一般我们线程的runloop运行在`kCFRunLoopDefaultModel`上，当滑动TableView时，会发生Mode切换，会切换到`UITrackingRunLoopMode`上。之前说过，RunLoop只能接收当前mode下的事件，而其他mode上的事件回调是无法接收到的。
	- 解决方案：通过`CommonMode `这个技术方案
		- 调用`CFRunLoopAddTimer`或者`NSRunLoop的addTimer:forMode`的方法
		- 最终会把timer添加到多个打了CommonMode标记的Mode中，UITrackingRunLoopMode也是被打了CommonMode标记的



#### 补充点：GCD的定时器是不受Runloop的mode影响的

- 举例：

```
		__block NSInteger timeOut = 60; // 倒计时时间
    dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
    dispatch_source_t _timer = dispatch_source_create(DISPATCH_SOURCE_TYPE_TIMER, 0, 0,queue);
    dispatch_source_set_timer(_timer, dispatch_walltime(NULL, 0), 1.0 * NSEC_PER_SEC, 0); // 每秒执行
    dispatch_source_set_event_handler(_timer, ^{
        if (timeOut <= 0) { // 倒计时结束，关闭
            dispatch_source_cancel(_timer);
        } else {
            timeOut--;
        }
    });
    dispatch_resume(_timer);
```



### RunLoop与多线程

- 线程和RunLoop的关系：线程是和RunLoop一一对应的。(数据结构)
- 自己创建的线程默认是没有RunLoop的。需手动添加

- 场景：怎样实现一个常驻线程
- 可以从以下几点来回答：
	- 为当前线程开启一个RunLoop
	- 向该RunLoop中添加一个Port/Source等维持RunLoop的事件循环
		- (RunLoop如果没有资源或者事件源需要处理的话，默认是不能维持自己事件循环的，就会直接退出)
	- 启动该RunLoop

实现一个常驻线程：

```
static BOOL runAlways = YES; // 是否继续维持事件循环
static NSThread *_thread = nil;

+ (NSThread *)oneThread
{
    if (!_thread) {
        @synchronized (self) {
            if (!_thread) {
                // 线程创建
                _thread = [[NSThread alloc] initWithTarget:self selector:@selector(runThread) object:nil];
                [_thread setName:@"threadName"];
                // 启动
                [_thread start];
            }
        }
    }
    return _thread;
}

// 创建一个Source
+ (void)runThread
{
    // 创建一个Source
    CFRunLoopSourceContext context = {0, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL};
    CFRunLoopSourceRef source = CFRunLoopSourceCreate(kCFAllocatorDefault, 0, &context);
    // 创建RunLoop，同时向Runloop的DefaultMode下添加Source
    CFRunLoopAddSource(CFRunLoopGetCurrent(), source, kCFRunLoopDefaultMode);
    // 外部控制条件
    while (runAlways) {
        @autoreleasepool {
            // 当前的RunLoop开始运行，在DefaultMode下运行
            CFRunLoopRunInMode(kCFRunLoopDefaultMode, 1.0e10, true); // 会状态切换，不会死循环
        }
    }
    
    // 某一时机 静态变量runAlways=NO，可以保证跳出RunLoop，线程退出
    CFRunLoopRemoveSource(CFRunLoopGetCurrent(), source, kCFRunLoopDefaultMode);
    CFRelease(source);
}
```





### RunLoop的总结

- 什么是 RunLoop，它是怎样做到有事做事，没事休息的？(含义+状态切换)

- RunLoop与线程是怎样的关系？(2点)

- 如何实现一个常驻线程？(3步骤)

- 怎样保证子线程数据回来更新UI的时候不打断用户的滑动操作？
	- (因为滑动，RunLoop会运行在UITrackingRunLoopMode下；我们可以将UI更新的逻辑封装成Block，提交到DefaultMode下；当滑动停止后，切换到DefaultMode下，就会执行刚刚的Block了)








### 参考文章
- [教你如何轻松搞定 Runloop](https://www.jianshu.com/p/4bad817df9ae)



## 最后

如果对大家有帮助，请[github上follow和star](https://github.com/jifengchao)，本文发布在[戴超的技术博客](https://jifengchao.github.io/)，转载请注明出处
