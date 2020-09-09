---
layout: post
commentid: ios-Runtime-summary
title: iOS开发Runtime基础知识
category: iOS
tags: iOS,Runtime
description:
---

>   iOS开发-Runtime基础知识


## 前言

Runtime又简称为运行时。其提供了对Objective-C语言动态属性的支持。Runtime的特性主要是消息(方法)传递，如果消息(方法)在对象中找不到,就进行转发。下面我们就从`数据结构`、`类对象与元类对象`、`消息传递`、`方法缓存`、`消息转发`、`Method-Swizzling`、`动态添加方法`、`动态方法解析`这些方面来探寻一下。


### 数据结构

主要存在一下结构：
- objc_object
- objc_class
- isa指针
- method_t


> objc_object

- id = objc_object：OC中的id类型在runtime中就是对应objc_object这样的数据结构

![]({{site.url}}/assets/postImages/ios/runtime/runtime01.jpg)


> objc_class

- Class = objc_class：Class代表一个类，在runtime中就是对应objc_class这样的数据结构
- objc_class继承自objc_object

![]({{site.url}}/assets/postImages/ios/runtime/runtime02.jpg)

- cache_t：表达了方法缓存的结构，在消息传递时会使用到
- class_data_bits_t：关于一个类，定义的变量、属性、方法都在class_data_bits_t结构当中


> isa指针

- 实际上是C++中的共用体isa_t结构
	- 指针型isa：isa的`值`代表Class的地址
	- 非指针型isa：isa的`值的部分`代表Class的地址

- isa指向
- 关于`对象`，其指向`类对象`。
- 关于`类对象`，其指向`元类对象`(MetaClass)。
(当我们在进行方法调用的时候，
若调用对象的实例方法，实际上是通过对象的isa指针到类对象当中去进行方法查找；
若我们调用的是类方法，则是通过类对象的isa指针到元类对象中去进行方法查找；)


> cache_t

- 用于`快速`查找方法执行函数
- 是可`增量扩展`的`哈希表`结构(在结构中随着存储内容增大，也会增量扩大内存空间)
- 是`局部性原理`的最佳应用
(简单说明局部性原理：
可能我们在调用的方法就集中那么几个方法，也就是这几个方法调用的频次是最高的，我们就可以把这些方法放到缓存当中，可能下次命中就更高一些)


- cache_t可以理解为一个数组实现的，里面每一个对象都是`bucket_t`这样的结构体。
- bucket_t由key和IMP组成；key对应OC中的选择器SEL，IMP可以理解为一个无类型的函数指针，即方法的实现


> class_data_bits_t

- `class_data_bits_t`主要是对`class_rw_t`的封装
- `class_rw_t`代表了类相关的`读写`信息(比如给类的分类添加的方法、属性、协议)，是对`class_ro_t`的封装(rw：可读可写；ro：只读)
- `class_ro_t`代表了类相关的`只读`信息


> class_rw_t

![]({{site.url}}/assets/postImages/ios/runtime/runtime03.jpg)

(以methods举例，methods装载了类的不同分类的方法集合；)


> class_ro_t

![]({{site.url}}/assets/postImages/ios/runtime/runtime04.jpg)


> method_t

- 首先回顾下函数的四要素：返回值、名称、参数、函数体。
- struct method_t：实际上是一个结构体类型，包含以下数据结构
	- SEL name：代表方法的名称
	- const char* types：对应的是方法的返回值和参数的组合
	- IMP imp：实际上是一个无类型的函数指针，所对应的就是函数体

![]({{site.url}}/assets/postImages/ios/runtime/runtime05.jpg)


下面讲解下const char* types，那么OC中是怎么实现的呢，这就要涉及苹果开发中Type Encodings技术。
- Type Encodings
	- const char* types；

![]({{site.url}}/assets/postImages/ios/runtime/runtime06.jpg)

- v 代表返回值是void类型的
- @ 代表第一个参数类型是id，即self
- : 代表第二个参数是SEL类型的





#### 整体数据结构

![]({{site.url}}/assets/postImages/ios/runtime/runtime07.jpg)


### 对象、类对象、元类对象

- `类对象`存储实例方法列表等信息；
- `元类对象`存储类方法列表等信息；

![]({{site.url}}/assets/postImages/ios/runtime/runtime08.jpg)



### 消息传递

我们先看下这段代买的结果如何

```

```

方法调用会编译器转为objc_msgSend、objc_msgSendSuper

- void objc_msgSend(void /* id self, SEL op, … */ )
- void objc_msgSendSuper(void /* struct objc_super *super, SEL op, … */ )

```
objc_super的结构：
struct objc_super {
	/// Specifies an instance of a class.
	__sunsafe_unretained id receiver; // 就是当前对象
}
```

(其实super只是编译器关键字，最后还是struct objc_super中的receiver接收消息)

- objc_msgSend和objc_msgSendSuper的区别
	- objc_msgSendSuper从父类的方法列表中开始查找

消息转发的流程：

![]({{site.url}}/assets/postImages/ios/runtime/runtime09.jpg)

下面我们继续分析三个判断条件里的实现


> 缓存查找

- 例：给定值是`SEL`，目标值是对应`bucket_t`中的`IMP`。
- 通过Hash查找，cache_key_t--f(key)—>bucket_t 


> 当前类中查找

- 对于`已排序好`的列表，采用`二分查找`算法查找方法对应执行函数
- 对于`没有排序`的列表，采用`一般遍历`查找方法对应执行函数


> 父类逐级查找

![]({{site.url}}/assets/postImages/ios/runtime/runtime10.jpg)




### 消息转发流程

- 转发流程，相关的函数：

![]({{site.url}}/assets/postImages/ios/runtime/runtime11.jpg)

- resolveInstanceMethod
- forwardingTargetForSelector
- methodSignatureForSelector



### Method-Swizzling

- Method Swizzling是一种改变一个selector的实际实现的技术。通过这一技术，我们可以在运行时通过修改类的分发表中selector对应的函数，来修改方法的实现。

![]({{site.url}}/assets/postImages/ios/runtime/runtime11.jpg)

- 举例，代码如下：

```
@implementation NSMutableDictionary (HYLSafe)

+ (void)load 
{
    Class dictCls = NSClassFromString(@"__NSDictionaryM");
    Method originalMethod = class_getInstanceMethod(dictCls, @selector(setObject:forKey:));
    Method swizzledMethod = class_getInstanceMethod(dictCls, @selector(hyl_setObject:forKey:));
    // 确保不插入nil对象
    method_exchangeImplementations(originalMethod, swizzledMethod);
}

- (void)hyl_setObject:(id)anObject forKey:(id<NSCopying>)aKey 
{
    if (!anObject)
        return;
    [self hyl_setObject:anObject forKey:aKey];
}

@end
```

- 使用场景：
	- 替换系统方法，如viewDidLoad，统一处理
	- NSMutableDictionary的setObject方法，保证程序不崩溃



### 动态添加方法

- 应用场景：可能有人会问，你有用过performSelector方法么？(隐式询问是否用过动态添加方法，因为类可能在编译时未实现某个方法，运行时才添加了方法)
- 具体添加实现:

```
void testImp(void)
{
    NSLog(@"test invoke");
}

+ (BOOL)resolveInstanceMethod:(SEL)sel
{
    if (sel == @selector(test)) {
        // 动态添加 test方法的实现
        class_addMethod(self, @selector(test), testImp, "v@:");
        return YES;
    }
    return [super resolveInstanceMethod:sel];
}
```



### 动态方法解析

- 应用场景：可能有人会问，你有使用过@dynamic这个编译器关键字？(隐式考察`编译时`语言与`动态运行时`语言的区别)
	- 当我们声明的属性，在实现中标志为@dynamic的时候，相当于属性的get、set方法是在运行时添加的，而不是编译时自动添加实现的

Q：`编译时`语言与`动态运行时`语言的区别？
- 动态运行时语言将函数决议推迟到运行时；
- 编译时语言在编译期进行函数决议；




### 总结

- [obj foo]和objc_msgSend（）函数之间有什么关系？
- runtime如何通过Selector找到对应的IMP地址的？(消息传递的流程)
- 能否向`编译后`的类中增加实例变量？(不能，可为动态添加的类增加实例变量)




### 参考文章
- [iOS Runtime详解](https://www.jianshu.com/p/6ebda3cd8052)
- [iOS方法的调用过程](https://blog.csdn.net/nathan1987_/article/details/76855326)
- [Method swizzling的正确姿势](https://www.jianshu.com/p/674bd221aac2)
- [iOS之RunTime动态添加方法属性](https://www.jianshu.com/p/60773495dc1e)



## 最后

如果对大家有帮助，请[github上follow和star](https://github.com/jifengchao)，本文发布在[戴超的技术博客](https://jifengchao.github.io/)，转载请注明出处
