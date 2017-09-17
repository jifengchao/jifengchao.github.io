---
layout: post
title: 设置UITextField/UISearchBar键盘上return key不可点击
category: iOS
tags: iOS,Target,cocoapods
description:
---

>   设置UITextField/UISearchBar键盘上return key不可点击


对于iOS开发者而言，项目中可以输入文字的控件有以下三类:
- UITextField
- UITextView
- UISearchBar

这个大家并不陌生，最近一个项目中，偶然发现`UISearchBar`默认在输入框内没有文字的时候，系统软键盘右下方的Search按钮是不可以点击的。

![searchBar-search.png](http://upload-images.jianshu.io/upload_images/847061-6731fb96ef59eb18.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

同时对比了`UITextField`和`UITextView`，发现默认情况下，Return(Search)按钮是可以点击的。原本也是想通过代理实现UISearchBar在没有文字的时候也可以点击，在UISearchBar的介绍中，发现系统提供了API，简单又方便。
例如：

```
UISearchBar *searchBar = [[UISearchBar alloc] initWithFrame:CGRectZero];
searchBar.returnKeyType = UIReturnKeySearch; // 设置按键类型
searchBar.enablesReturnKeyAutomatically = NO; //这里设置为无文字,还可以点击
```
同样的，我们也可以去设置`UITextField`在无文字的时候按钮不可以点击:
```
UITextField *textField = [[UITextField alloc] initWithFrame:CGRectZero];  
textField.returnKeyType = UIReturnKeySearch; //设置按键类型  
textField.enablesReturnKeyAutomatically = YES; //这里设置为无文字就灰色不可点  


## 最后

如果对大家有帮助，请[github上follow和star](https://github.com/jifengchao)，本文发布在[戴超的技术博客](https://jifengchao.github.io/)，转载请注明出处
