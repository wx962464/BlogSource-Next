title: Android sdk content loader 一直显示0% 解决办法  
date: 2015-09-27 18:45:00
categories: Android
tags: [Android,Eclipse]
---

上面这种情况只有在eclipse 中才会出现，而且现在已经很少使用eclipse,都已经开始使用AS了，但是这个问题还是说下，处理方法比较简单。

> * 首先关闭eclipse。无法关闭则使用进程管理将其kill掉
> * 打开本地的用户目录，找到.android 文件夹。
> * 删除.android下面的cache文件夹
> * 删除.android下面的ddms.cfg文件
> * 重新启动eclipse，解决问题。

上面的方法能够解决大部分一直loading 显示在0%的情况。
