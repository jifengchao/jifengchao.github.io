---
layout: post
commentid: ios-fastlane
title: fastlane自动集成(一)
category: iOS
tags: iOS,fastlane,自动集成
description:
---

>   fastlane自动集成(一)


fastlane是什么
--------------------------------------------
>fastlane是一款给iOS和Android开发人员实现自动化构建的工具，可以将繁琐的任务变得更简单，如生成屏幕截图，处理配置文件以及发布应用程序等，以此来提高开发者的开发效率。这是[Github地址](https://github.com/fastlane/fastlane)和[官方技术文档](https://docs.fastlane.tools)。

本文实现的目标
---------------------------------------------
- 终端输入`fastlane exportIpa`自动打包`development`和`ad-hoc`版本的IPA，并将IPA文件导出到指定路径。
- 终端输入`fastlane firAdhoc`，将`ad-hoc`版本的IPA上传至[Fir.im](https://fir.im)应用分发平台
- 终端输入`fastlane pgyerAdhoc`，将`ad-hoc`版本的IPA上传至[蒲公英](https://www.pgyer.com)应用分发平台

####安装fastlane

- 系统要求: macOS或Linux，Ruby版本2.0.0以上
```
#查看Ruby版本
$ ruby -v
```

- 检测`Xcode`命令行工具的安装情况，在终端中输入命令
```
$ xcode-select --install
```
如果安装过会有提示，没安装过就会自动进行安装。

- 安装`fastlane`，在终端中输入命令
```
$ sudo gem install fastlane --verbose

# 如果安装时出现错误无法安装，就使用以下命令安装。
$ sudo gem install -n /usr/local/bin fastlane
```

- 安装完成后可以查看`fastlane`是否安装成功，在终端中输入
```
$ fastlane --version
```

###项目配置
- 在Xcode中打开项目，点击`Manage Schemes`
![Snip20180309_15.png](http://upload-images.jianshu.io/upload_images/847061-0778cde57a02c1b8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 将对应的scheme后的Shared
![Snip20180309_16.png](http://upload-images.jianshu.io/upload_images/847061-7609404856b53373.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###初始化fastlane配置

- 在终端中，`cd`到项目的根目录，在终端中输入
```
fastlane init
```
选择一下任意一种初始化fastlane(我们可以选择第一个)
![Snip20180309_17.png](http://upload-images.jianshu.io/upload_images/847061-60a979d4b8841712.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这时项目根目录会多出`fastlane文件夹`和`Gemfile`文件，`Gemfile`稍后会讲解。其中`Fastfile `是最重要的一个文件，在`Fastfile `里面可以编写和定制我们打包脚本的一个文件，所有自定义的功能都写在这里。

### 打包IPA文件

- 打包IPA文件，需要了解一些fastlane的语法。先使用编辑器打开`Fastfile `文件，在`platform :ios do`后添加以下内容
```
lane : exportIpa do
puts "以 ad-hoc 方式打包"
gym(
# 指定打包所使用的输出方式 (可选: app-store, package, ad-hoc, enterprise, development)
export_method: "ad-hoc",
# 指定项目的 scheme 名称
scheme: "ThirteenmakeFriends",
# 指定输出的文件夹地址
output_directory: "./archive/adhoc/" + Time.new.strftime("%Y-%m-%d %H-%M-%S"),
# 指定打包方式 (可选: Release, Debug)
configuration: "Debug"
)
end
```
这时候在终端中输入命令
```
$ fastlane exportIpa
```
等待一段时间后，在终端中出现`successfully 🎉`，就表示打包成功了，在项目目录中，也能看到生成的IPA文件了。

###自动打包到`Fir.im`平台

- 注意需要先安装`Fir.im`插件，[GitHub 教程](https://github.com/FIRHQ/fir-cli)
```
$ gem install fir-cli
```

- 安装`Fir.im`的 Fastlane 插件，这是[Github 地址](https://github.com/dongorigin/fastlane-plugin-fir)
```
$ fastlane add_plugin versioning
$ fastlane add_plugin fir
```

- `fastlane`初始化生成的`Gemfile `文件，就是用于管理`fastlane `插件的，如果iOS项目是通过cocoapods管理，需要在`Gemfile `中添加
```
gem 'cocoapods'
```

- 我们在`Fastfile `配置`Fir.im`参数，其实就是在打包IPA的内容中添加`Fir.im`信息
```
lane : uploadFir do
puts "以 ad-hoc 方式打包"
gym(
# 指定打包所使用的输出方式 (可选: app-store, package, ad-hoc, enterprise, development)
export_method: "ad-hoc",
# 指定项目的 scheme 名称
scheme: "ThirteenmakeFriends",
# 指定输出的文件夹地址
output_directory: "./archive/adhoc/" + Time.new.strftime("%Y-%m-%d %H-%M-%S"),
# 指定打包方式 (可选: Release, Debug)
configuration: "Debug"
)
firim(
firim_api_token: "xxxxxxx",
app_changelog: "测试环境"
)
end
```

- 在终端中输入`fastlane uploadFir `，等待一段时间，顺利的就可以在`Fir.im`中看到我们的应用了。

###自动打包到`蒲公英`平台

- 安装步骤与`Fir.im`是一致的

- 安装`蒲公英`的 Fastlane 插件，这是[Github 地址](https://github.com/shishirui/fastlane-plugin-pgyer)
```
$ fastlane add_plugin versioning
$ fastlane add_plugin pgyer
```

- 我们在`Fastfile `配置`蒲公英`参数，其实就是在打包IPA的内容中添加`蒲公英`信息
```
lane : uploadPgyer do
puts "以 ad-hoc 方式打包"
gym(
# 指定打包所使用的输出方式 (可选: app-store, package, ad-hoc, enterprise, development)
export_method: "ad-hoc",
# 指定项目的 scheme 名称
scheme: "ThirteenmakeFriends",
# 指定输出的文件夹地址
output_directory: "./archive/adhoc/" + Time.new.strftime("%Y-%m-%d %H-%M-%S"),
# 指定打包方式 (可选: Release, Debug)
configuration: "Debug"
)
pgyer(api_key: "xxxxxx", user_key: "xxxxxx")
end
```

- 在终端中输入`fastlane uploadPgyer `，等待一段时间，顺利的就可以在`蒲公英`中看到我们的应用了。

>注意，iOS项目中得先配置好PP文件(之后研究自动管理PP文件)

参考文章目录:
- [使用 fastlane 实现 iOS 持续集成（二）](https://www.jianshu.com/p/6ef62c0415dc)

- [使用 Fastlane 自动化打包并发布 iOS 项目](https://www.jianshu.com/p/662677cb1b47)

- [Fastlane：入门与实战 蒲公英](https://www.jianshu.com/p/65c3c98a5b78)

- [iOS自动化打包发布(fastlane)](http://blog.csdn.net/cdut100/article/details/76381605)


## 最后

如果对大家有帮助，请[github上follow和star](https://github.com/jifengchao)，本文发布在[戴超的技术博客](https://jifengchao.github.io/)，转载请注明出处
