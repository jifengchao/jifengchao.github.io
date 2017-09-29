---
layout: post
commentid: ios-podsearch-unable
title: pod search无法搜索到第三方库
category: iOS
tags: iOS,cocoapods,search
description:
---

>   pod search无法搜索到第三方库


## 前言

今天在浏览技术文章时，发现了一款实用的第三方库[LifetimeTracker](https://github.com/krzysztofzablocki/LifetimeTracker)，让开发者在开发过程中，直观快速的发现应用生命周期内的内存泄漏。
LifetimeTracker支持CocoaPods安装，但是在终端中$ pod search LifetimeTracker 却提示我:

```
[!] Unable to find a pod with name, author, summary, or descriptionmatching '······'
```

通常验证库是否在Cocoapods的Repos中存在，可以去[cocoapods.org](https://cocoapods.org)搜索。在`cocoapods.org`中发现LifetimeTracker是存在的。

### 解决方案

-   执行pod setup
    - 在终端输入$ pod setup, 会出现 Setting up CocoaPods master repo, 等几分钟会输出 Setup completed, 说明 pod setup执行成功。

-   删除~/Library/Caches/CocoaPods目录下的search_index.json文件
    - 终端输入$ rm ~/Library/Caches/CocoaPods/search_index.json

-   执行pod search
    - 终端输入$ pod search LifetimeTracker
    - 输出 Creating search index for spec repo 'master'.. Done!, 稍等片刻就会出现所有带LifetimeTracker字段的库出现。


## 最后

如果对大家有帮助，请[github上follow和star](https://github.com/jifengchao)，本文发布在[戴超的技术博客](https://jifengchao.github.io/)，转载请注明出处
