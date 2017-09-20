---
layout: post
commentid: ios-push-certificate
title: 创建App IDs和推送证书
category: iOS
tags: iOS,推送证书
description:
---

>   创建App IDs和推送证书

## 开发环境与生产环境

苹果推送服务器区分开发环境（Development）和生产环境（Production），两个环境的服务器不同，使用的 P12 证书不同，完全隔离。

有几点需要注意的。

-   deviceToken 是唯一标识客户端的凭证，所以必须上传应用服务器才能使用远程推送。
-   模拟器收不到远程推送。
-   越狱的设备 APNs 服务不能保证，所以不一定能收到远程推送。
-   APNs 使用 BundleID 区分 App，使用通配符 BundleID 的 App 将无法使用远程推送。

## 远程推送配置

1.为 App 开启远程推送服务
登录 [Apple Developer](https://idmsa.apple.com/IDMSWebAuth/login?appIdKey=891bd3417a7776362562d2197f89480a8547b108fd934911bcbea0110d07f757&path=%2Faccount%2Fios%2Fcertificate%2F&rv=1)，进入 Identifiers 选择 App IDs。

![]({{site.url}}/assets/postImages/ios/pushset01.png)

![]({{site.url}}/assets/postImages/ios/pushset02.png)

我们可以新建一个 AppID，或者在原有的 AppID 上增加 Push Notification 的 Service。 需要注意的是，我们 App 的 BundleID 不能使用通配符，否则将无法使用远程推送服务。

![]({{site.url}}/assets/postImages/ios/pushset03.png)

开启远程推送服务。

![]({{site.url}}/assets/postImages/ios/pushset04.png)

2.生成并上传 P12 证书

选中 AppID ，选择 Edit。

![]({{site.url}}/assets/postImages/ios/pushset05.png)

可以看见，在 Push Notification 下方有两个 SSL Certificate ，分别是用于开发环境和生产环境的远程推送证书。

![]({{site.url}}/assets/postImages/ios/pushset06.png)

点击 Create Certificate ，这时候会提示需要一个 Certificate Signing Request（CSR）。

![]({{site.url}}/assets/postImages/ios/pushset07.png)

可以根据其说明，在 Mac 上打开钥匙串应用，在菜单中点击“从证书颁发机构请求证书”。

![]({{site.url}}/assets/postImages/ios/pushset08.png)

输入我们的邮箱、姓名或公司名，选择保存到磁盘，点击继续，就会生成一个 *.certSigningRequest 文件。

![]({{site.url}}/assets/postImages/ios/pushset09.png)

然后返回 Apple Developer 网站，点击 Continue，上传生成的 .certSigningRequest 文件，点击 Generate ，即可生成推送证书。

![]({{site.url}}/assets/postImages/ios/pushset10.png)

按照上面同样的步骤，生成生产环境的推送证书。

![]({{site.url}}/assets/postImages/ios/pushset11.png)

注意:
从 iOS 9.2开始，Apple Developer 上生成的生产环境推送证书，名称为 Apple Push Services: XXX, 之前生成的生产环境推送证书名称为 Apple Production IOS Push Services: XXX。 


此时，您可以在 Push Notification 下方看见目前每个环境对应的推送证书。

![]({{site.url}}/assets/postImages/ios/pushset12.png)

将上面的 SSL Certificate 都下载到 Mac 本地，双击打开，系统会将其导入钥匙串中。 打开钥匙串应用，选中对应的证书，右键选择导出。保存 P12 文件时，可以为其设置密码，也可以不设置密码。

![]({{site.url}}/assets/postImages/ios/pushset13.png)

![]({{site.url}}/assets/postImages/ios/pushset14.png)

之后，将P12文件上传到第三方推送服务商(如极光、个推等)即可。

## 最后

如果对大家有帮助，请[github上follow和star](https://github.com/jifengchao)，本文发布在[戴超的技术博客](https://jifengchao.github.io/)，转载请注明出处
