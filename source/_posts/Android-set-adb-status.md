title: Androi系统设置Android adb 开关的方法  
date: 2015-11-24 17:38:26  
categories: Android   
tags: [Android,Android系统]
---

在整机系统开发中，一般系统默认的adb开发是打开的，那么在对外发布的系统中，肯定是不希望默认打开adb的，但是在开发的过程中，肯定希望能够通过某种操作打开adb，便于调试，这就需要在系统的某个部位做一个开关了。那么这篇文章就是说说这边是如何做到在系统中增加一个adb开关。
    
* 1. 在系统中有一个usb deubg的开关，此开关是打开usb调试的对adb 但是通过默认设置的开发者模式都可以将其打开。
* 2. adb 启动肯定会启动了一个`adbd`服务，那么手动将该服务kill掉就可以关闭adb服务了.当然这样是在adbd启动后可以这么做，但是还是直接默认就不启动服务吧，需要的时候在打开吧。
    
开启`adbd`服务实际是再系统启动中 `init.rc`文件中启动的，里面有很很多部分有调用`start adbd`或者`restart adbd`这部分是Android启动流程中zygote（受精卵）启动的，这部分涉及整个安卓的启动流程，以及`init.rc`文件的定义和使用大家可以查看网上资料。
    首先要将所有系统中`start adbd`和`restart adbd`的部分将其注释掉，不使用系统默认启动方式。一般都是在`init.rc`文件中,在使用`find . -iname "init*.rc" | xargs grep "adbd"` 将系统中所有有关adbd服务的都将其搜索出来，避免遗漏。  
再开看看`init.rc`文件中的adbd服务是怎么定义的：
    
```shell
# adbd is controlled via property triggers in init.<platform>.usb.rc
service adbd /sbin/adbd
    class core
        socket adbd stream 660 system system
    disabled
        seclabel u:r:adbd:s0
```
实际上它是定义了一个`sbin/adbd`文件为adbd服务，在`init.rc`文件中定义服务，那我们就使用`init.rc`文件中的触发器来控制adbd服务的打开与关闭。定义一个属性`persist.sys.adbd.on`来标记adb的开关状态，定义触发器内容如下：

```shell
on property:persist.sys.adbd.on=1
    start adbd
    
on property:persist.sys.adbd.on=0
    stop adbd
```
看字面上的意思也可以大致看出来当property系统属性`persist.sys.adbd.on`改变的时候在init.rc中能够收到改变的消息。且当属性值为1的时候，则会调用`start adbd`，当为0的时候则会调用`stop adbd`，因为adbd是一个服务，通过start和stop即可控制，这样通过程序中设置property属性即可切换adb的状态了。

大致的实现过程就是这样了，口才文采不行，表达不好，见谅。
这里面需要了解一些Android启动过程，以及init文件定义等。

提示：要设置系统属性需要系统权限才行，所以这种也只是自己开发系统应用才起作用。

关于权限相关的文章参考[http://blog.csdn.net/a345017062/article/details/6254402](http://blog.csdn.net/a345017062/article/details/6254402)

## 欢迎访问 [wxtlife.com](http://www.wxtlife.com)