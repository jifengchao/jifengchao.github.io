---
layout: post
commentid: tool-sourceTree-recommit
title:  SourceTree回退撤销commit
category: Tool
tags: tool,sourceTree,recommit
description:
---

>   SourceTree回退撤销commit


## 前言

公司的源代码管理是通过git管理的，托管在oschina上。第三方托管的平台有一个规则就是单个文件不能超过100M，否则本地的推送就无法同步到远程的仓库中。经常一不小心进行了提交操作，这时候就需要我们撤销刚刚的提交了。

![]({{site.url}}/assets/postImages/tool/sourceTreeRecommit01.png)

提交了不想提交的文件到本地仓库如下：

![]({{site.url}}/assets/postImages/tool/sourceTreeRecommit02.png)

现在的想法就是把"提交代码"这条回退回来，回退后状态如下：

![]({{site.url}}/assets/postImages/tool/sourceTreeRecommit03.png)

操作步骤：

-   选中提交之前的版本，然后右击，弹出菜单如下：

![]({{site.url}}/assets/postImages/tool/sourceTreeRecommit04.png)

-   选择回退模式
    - 三个选项含义如下:
    - 1.回退到暂存区
    - 2.回退到未暂存区
    - 3.直接把提交的文件reset （最好不要用）

![]({{site.url}}/assets/postImages/tool/sourceTreeRecommit05.png)

第一个和第二个是安全操作，可以根据需要选择。

-   最后将刚刚新增、修改的文件进行移除、丢弃操作

![]({{site.url}}/assets/postImages/tool/sourceTreeRecommit06.png)

![]({{site.url}}/assets/postImages/tool/sourceTreeRecommit07.png)


注意:

如果在你提交之前，有未拉取的代码，这个时候应该拉取完成之后再commit。否则经过上面的操作，会把别人提交的代码回退回去。


## 最后

如果对大家有帮助，请[github上follow和star](https://github.com/jifengchao)，本文发布在[戴超的技术博客](https://jifengchao.github.io/)，转载请注明出处
