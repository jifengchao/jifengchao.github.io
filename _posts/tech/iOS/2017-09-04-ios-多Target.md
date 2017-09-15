---
layout: post
title: 添加新target (支持cocoapods)
category: iOS
tags: iOS,Target,cocoapods
description:
---

>   添加新target (支持cocoapods)


##前言

到目前为止，也做了不少的项目了，之前的项目开发时，测试人员经常要求发布不同接口环境的app，当时采用的就是通过手动修改代码，多次以后，经常会把环境弄混淆了，尤其是接口环境还会发生变化，后来发现别人的项目针对这种情况，是采用多Target进行开发的。
下面去实现下。

> 1.创建多个Target

1.1首先直接在现有target下copy出来一个

![Snip20170904_2.png](http://upload-images.jianshu.io/upload_images/847061-6e0652d618c7538f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

1.2修改Target的名称

![Snip20170904_3.png](http://upload-images.jianshu.io/upload_images/847061-2ceda59026b28d04.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

1.3修改scheme的名称

![Snip20170904_4.png](http://upload-images.jianshu.io/upload_images/847061-59da436d79034f0b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![Snip20170904_5.png](http://upload-images.jianshu.io/upload_images/847061-ee344554bb6c27fd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

1.4修改plist文件名称，注意同时要修改Build Setting里的Info.plist File

![Snip20170904_6.png](http://upload-images.jianshu.io/upload_images/847061-cea7a9d60b3cf118.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

1.5在Build Setting里，为每个Target设置宏


![Snip20170904_7.png](http://upload-images.jianshu.io/upload_images/847061-57ac4ffb27eb999a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

1.6在代码中，根据宏，设置对应的代码

![Snip20170904_10.png](http://upload-images.jianshu.io/upload_images/847061-30db1ce48d1f58a8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
#ifdef PROD

/** 接口域名、IP地址*/
NSString * const kHttpUrl = @""; // 正式服务器

#elif UAT

/** 接口域名、IP地址*/
NSString * const kHttpUrl = @""; // 测试服务器

#else

#endif
```

> 2.Cocoapods配置

2.1 配置Podfile文件格式

![](http://upload-images.jianshu.io/upload_images/847061-76aada1c2250d6a0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```

platform :ios, '8.0'
inhibit_all_warnings!

# 共用的pod第三方
pod 'Masonry'
pod 'MJExtension'
pod 'MJRefresh'

# 项目中的target各自的设置
target 'UCows' do
#可以在这里添加UCows独自引用的pod第三方
end

target 'UCowsUAT' do
#可以在这里添加UCowsInternal独自引用的pod第三方
end

```

> 注意事项

如果项目是使用cocoapods管理的，先pod install后，再执行上面的操作

> 弊端

1.如果存在手动管理第三方的SDK，需要不同的Target都单独设置一次



## 最后

如果对大家有帮助，请[github上follow和star](https://github.com/jifengchao)，本文发布在[戴超的技术博客](https://jifengchao.github.io/)，转载请注明出处
