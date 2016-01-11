title: Android开发中一些容易踩的坑
date: 2015-12-31 00:20:38
tags: Android
---

# 欢迎访问 [wxtlife.com](http://www.wxtlife.com)
1. Activity调用`startActivityForResult`会立马返回，不能正常调用
> **原因：**
原因为Activity LauncherMode 为`singleTask` `singleInstance`，这种情况下Android不允许这么做，具体分析原因见[http://www.360doc.com/content/15/0123/14/12928831_443085580.shtml](http://www.360doc.com/content/15/0123/14/12928831_443085580.shtml)
**解决方法:**
修改activity的launchMode，或者应用一个空白的activity来做个跳转桥梁。具体见[http://mthli.github.io/singleTask-onActivityResult/](http://mthli.github.io/singleTask-onActivityResult/)

2. 在PopupWindow中的EditText不能获取焦点，显示键盘
> **原因：**
原因为Popupwindow 默认没有获取到焦点，需要手动设置焦点，这样子view才能获取到事件的监听。
**解决办法：**
在创建完popwindow后设置他的焦点，`popupWindow.setFocusable(true);  `就可以了。
 <!-- more --> 
3.  Popupwindow默认在区域外点击不消失
> **原因：**
据说这是个PopupWindow的Bug，但也不确定是不是Popupwindow故意这样设计的，对于点击不想消失的提供了一个方法。
**解决办法：**
要对PopupWindow 设置一个背景图`popWindow.setBackgroundDrawable(new BitmapDrawable()); `要创建一个空对象，设置为null是不行的，或者就创建一个全透明的背景图。
4. AS创建的项目，在源码下编译出来，使用`packageManager.getPackageInfo` 获取versionCode错误.
> **原因:**
AS创建的项目默认在Manifests中是没有versionCode和versionName的，而是写在了moudle的build.gradle中,所以导致在源码下编译是找不到AndroidManifest中的versionName以及versionCode的。
**解决办法：**
手动在AndroidManifest中添加versionCode 以及versionName 字段并且与build.gradle中保持一致，避免其他问题。


**后面遇到的问题会持续更新**

## 欢迎访问 [wxtlife.com](http://www.wxtlife.com)