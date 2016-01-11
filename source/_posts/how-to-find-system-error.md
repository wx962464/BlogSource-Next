title: "Android 查找系统问题由哪儿导致的"
date: 2015-07-31 19:47:29
tags: Android
---
# 欢迎访问 [wxtlife.com](http://www.wxtlife.com)
最近和公司大神一起看一个bug，发现他的这个技能很厉害，故记录下来进行学习。
**问题**：在系统切换网络的时候会导致系统整个logcat 或者system_server 占40%多的cpu资源，导致系统很卡.
**解决过程**：   
*  查找相对应的进程
    首先进入adb shell模式，然后使用`ps | system_server`查找system_server进程 ,然后会得到下面的输出
```
ps | grep system_server
system    1600  891   505972 43496 ffffffff 40048aa8 S system_server
```
从上面看到使用  一些应用信息，比如当前system_server的pid为1600，然后system_server的父进程为891，然后在进行查找891是那个程序，以及他的父进程,如下所示：
```
root@mt5861:/ # ps | grep 891
ps | grep 891
root      891   1     409076 41840 ffffffff 400478d8 S zygote
system    1600  891   506104 43440 ffffffff 40048aa8 S system_server
```
发现891的进程是zygote也就是android系统的启动程序，而他的父进程号为1，因此可以zygote是由init函数来进行启动的，所以真个启动的流程我们就定位到了，下面开始分析每个父进程执行了哪些命令来启动了子进程。
*  根据进程号查找执行操作
在系统根目录会有个proc的文件夹，里面有很多数字的文件夹那就是每个应用程序的进程号，是以进程号来记录应用程序的一些信息，linux的知识。
然后进到刚刚看到的1600文件夹，里面又有很多的文件夹及文件，但是里面有个cmdline的文件是很重要的，从名字来看大概这里面记录的所有的操作命令。使用cat cmdline看看里面的内容。
```
root@mt5861:/proc/1600 # cat cmdline
cat cmdline
system_server
```
<!-- more --> 
发现system_server里面只有一条命令命令即为”system_server“，说明是执行了system_server 命令启动它的，
接着我们看看891文件夹里面的cmdline信息.
```
root@mt5861:/proc/891 # cat cmdline
cat cmdline
zygote   /bin/app_process -Xzygote   /system/bin  --zygote  --start-system-server    root
```
可以看出是在 是在 bin目录下的app_process启动的，这样可以定位到哪里启动它的。
然后进到 android 源码目录，通过init.rc 文件找上面的语音，因为上面的父进程为1，也就是init程序了。
*  在Android系统源码中查找上面定义的服务
在Android源码目录下面查找服务，因为父进程为1所以直接查找所有的init程序即可。
```
find . -name "*.rc" | xargs grep "zygote"
root@mt5861:~/android/kk-4.x/device/mtk/mtk/basic/configs$ find ./ -name "*.rc" | xargs grep "zygote"
./init_for_usb_disk.rc:    onrestart restart zygote
./init_for_usb_disk.rc:    onrestart restart zygote
./init_for_usb_disk.rc:service zygote /system/bin/app_process -Xzygote /system/bin --zygote --start-system-server
./init_for_usb_disk.rc:    socket zygote stream 666
./zygote-normal.rc:service zygote /system/bin/app_process -Xzygote /system/bin --zygote --start-system-server
./zygote-normal.rc:    socket zygote stream 666
./init.rc:    onrestart restart zygote
./init.rc:    onrestart restart zygote
./init.rc:service zygote /system/bin/app_process -Xzygote /system/bin --zygote --start-system-server
./init.rc:    socket zygote stream 660 root system
./zygote-blcr.rc:service zygote /sbin/sh /system/bin/quicklaunch.sh -Xzygote /system/bin --zygote --start-system-server
./zygote-blcr.rc:    socket zygote stream 666
```
可以看到好几个文件中都有这么一句话，` zygote /system/bin/app_process -Xzygote /system/bin --zygote --start-system-server`及`service zygote /system/bin/app_process -Xzygote /system/bin --zygote --start-system-server`也就是system_server 及 zygote是在这里定义启动起来的。 
带service ：代表定义了一个服务名字为zygote。其他其地方可以start server 即可启动服务

大致的方法是这样，只是记录个过程提供一个思路，当然这个不具有很强的代表性例子。

**学习方法很重要** 
## 欢迎访问 [wxtlife.com](http://wxtlife.com)
