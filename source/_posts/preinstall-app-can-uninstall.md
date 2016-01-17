title: Android系统预制可自由卸载apk
date: 2016-01-08 22:03:56
categories: Android系统
tags: [Android,Android系统]
---
在我们都痛恨手机厂商预装着一堆无用的app时，是否考虑怎么实现将app预装在data区，让用户可以自由卸载，做一个有良心的厂商，下面就把来说说如何实现预装app能够让用户卸载。
### 系统识别的app位置  
系统能够识别到app的位置有三个，分别是`/system/app/`,`/system/priv-app` `/data/app/` 下面。但是只有`/data/app`下面的app才可以被用户可以卸载。那就得想办法将apk放在`/data/app`下面。
### 尝试将预装app放进升级包data区中
根据编译的情况知道，一般在源码下编译的应用都会被放在`out/targe/.../system/app`或者`out/targe/.../system/priv-app`下面，用户区是在/data/app/ ，如果在编译后打包前copy某个app到/data/app/下面，且打包的系统img是只会在system区起作用，在data区是不起作用的。所以这种方案不行，只能将app放在system区的文件中了。
	
### 在编译时预制到系统system目录中
先将放在系统编译到某个目录下面，我这边放在`android/kk-4.x/device/mtk/mtk/preinstall/apk`下面，由于现在使用的平台为mtk所以放在mtk下面，preinstall为自己创建的文件夹，以及子目录apk，然后将所有app放在此目录下，当然这样是不能被打包到系统目录的，然后在preinstall目录下面创建一个`preinstall.mk`文件，当然名字随便了，里面的主要功能就是copy预装的app到系统分区目录中。  
<!-- more -->   
主要内容如下：

```Makefile
###################################################
#Copy proprietary apk to system/usr/app
###################################################
#定义mk函数，$(1)和$(2)为定义函数的前两个参数
define all-data-files-under
$(patsubst ./%,%, \
$(shell cd $(LOCAL_PATH)/$(1) ; \
      find ./ -maxdepth 1  -name "$(2)") \
)
endef
#copy apk files to /system/user/app/
#调用函数，将已apk后缀的copy到目标位置
COPY_APK_TARGET := $(call                                                 all-data-files-under,apk,*.apk)
PRODUCT_COPY_FILES += $(foreach apkName, (COPY_APK_TARGET), \
$(addprefix $(LOCAL_PATH)/apk/, $(apkName)):$(addprefix system/usr/app/, $(apkName)))
```
`PRODUCT_COPY_FILES`为Android系统提供的一个copy机制。但是默认是不允许copy apk的，它建议我们用pre_build来替代copy apk方式，但是pre_build的方式在多个应用情况下就会比较麻烦。那就直接修改`Makefile`文件，将其禁止copy的警告校验去掉即可。在`build/core/Makefile`中定义了一个函数如下：

```Makefile
define check-product-copy-files
$(if $(filter %.apk, $(1)),$(error \
    Prebuilt apk found in PRODUCT_COPY_FILES: $(1), use BUILD_PREBUILT instead!))
endef
```
这就是提示警告的地方，把调用`check-product-copy-files`的地方注释掉即可，当然这里面不能用#进行简单注释，具体查看Makefile的注释
上面的脚本为Makefile的一些用法，可以参考网上[Makefile总结](http://www.cnblogs.com/wang_yb/p/3990952.html) 
**此方法的弊端就是系统包会因为app的数量及大小而增大。**

### 如何在系统中加载preinstall.mk
这里参考其他Mk调用的方式来实现。一般增加一个编译源码下的应用到系统，默认都会在`build/target/product`下面的`generic_no_telephony.mk`中的`PRODUCT_PACKAGES`添加应用名称，当然不同厂商可能目录位置不同。
在这个文件中有加载很多mk的代码，所以照着加载之前mk的写法，

```Makefile
$(call inherit-product-if-exists, device/mtk/mtk/preinstall/preinstall.mk)
$(call inherit-product-if-exists, frameworks/base/data/fonts/fonts.mk)
$(call inherit-product-if-exists, external/noto-fonts/fonts.mk)
$(call inherit-product-if-exists, external/naver-fonts/fonts.mk)
$(call inherit-product-if-exists, external/sil-fonts/fonts.mk)
$(call inherit-product-if-exists, frameworks/base/data/keyboards/keyboards.mk)
$(call inherit-product-if-exists, frameworks/webview/chromium/chromium.mk)
$(call inherit-product, $(SRC_TARGET_DIR)/product/core.mk)
```
上面的就是调用加载其他的mk文件，第一行就为加载自己添加的mk文件如果存在(路径为前面自己定义的)，这样在编译的时候就会执行定义的mk，也就是把预制的app全部copy到系统的指定目录下面。
### 将系统中的app放到data区的app中
前面的操作步骤已经能够将预制的app打包进系统包中，但是系统上那些app还是在system区的，并不能在data区，那么就要想在用户第一次使用的情况下将app`复制`到data区的app安装位置，，且不能让用户有察觉。
> **注意：**上面提到的是复制，原因是如果是移动的话，源文件会没有，当用户恢复出厂设置的时候，data区app会没有，重新启动后，系统里面要复制的app也不存在了，则预装的app也就无法安装了，相当于恢复出厂后有漏洞了。

下面就是如何复制到data区的问题了，我们主要利用shell脚本来做应用复制，在配合上系统的init.rc初始化来做，因为方便快捷。当然也有其他方法，但不是最佳，等会会说。
现说怎么实现首次copy应用到data区，下面就是主要是shell脚本了。

```Bash
#!/sbin/sh
CUSTOMIZED_APK=/system/usr/app
DATA_APK=/data/app
echo "CUSTOMIZED_APK=${CUSTOMIZED_APK}"
#获取是否已经预安装过标记位
PREINSTALL_RESULT=`getprop persist.sys.preinstall.value`
echo "PREINSTALL_RESULT=${PREINSTALL_RESULT}"
apk_files=""
#判断标记位是否为空，为空则没有预装过。然后将所有apk均copy到data/app下面。
if [ -z "${PREINSTALL_RESULT}" ]; then
        cd ${CUSTOMIZED_APK}
        apk_files=$(ls *.apk )
        echo "apks files = ${apk_files}"
        for apkfile in $apk_files
        do 
                echo " apkfiles = ${apkfile} " 
                busybox cp -vf ${CUSTOMIZED_APK}/${apkfile} ${DATA_APK}/${apkfile}
                echo "start copy "
                chmod 777 ${DATA_APK}/${apkfile}
        done 
        #设置标记位
        setprop persist.sys.preinstall.value 1
        cd ../..
fi       
```
注意上面开头为`#!/sbin/sh`这个要根据系统的sh位置来进行确认，为了实现首次安装所以需要用到一个标记，这里采用系统的property来做标记，在复制时使用`busybox cp`是因为好多shell被Android系统精简了，还有copy完成后记得修改应用的权限，避免权限不够而知无法安装的问题，所有都copy完成后将标记为修改了，避免下次还会执行此操作。

另外一种方法就是单独一个app，在系统起来后进行首次的app转移。这种就是实际上已经进入到launcher界面了，如果开机广播慢的话，用户就会察觉到怎么突然多了几个应用，体验很不好。
### shell如何放入系统
此方法基本和前面写好的preinstall.mk复制应用方法 一致，也是在`generic_no_telephony.mk`中用Android自带的`PRODUCT_COPY_FILES`来将shell脚本copy进系统的bin文件下，然后在需要的时候进行执行。

```Makefile   
PRODUCT_COPY_FILES += \
    device/mtk/mtk/preinstall/preinstall.sh:/system/bin/preintall.sh
```
上面目的就是将脚本copy至系统的bin目录下。这样我们准备的东西都已经就绪了，下面就是如何让这些脚本运作起来。

### 如何执行shell脚本
前面提到利用Android启动时的初始化脚本也就是init.rc 进行处理，前面也有[文章](http://www.wxtlife.com/2015/11/24/Android-set-adb-status/)是用到init.rc来开发一些功能的，这里还是在说下
首先定义一个service,并且执行这个服务的名字，以及服务执行的脚本，对于init.rc的语言大家可以参考网上的，对此不是很熟悉。

```Makefile
service preinstall /system/bin/preinstall.sh
        class main
        user root
        group root
        oneshot
```
大概解释下里面的内容，preinstall就为执行服务的名称，后面的preinstall.sh即为服务执行的脚本，下面class main 为类别，user和ground分别指定权限。oneshot指明只执行一次，当然这里定义的服务默认就是开机要执行的，若要不需开机执行则需要再加一个关键字disabled。所以开机会执行定义的服务，故执行定义的shell脚本，虽然每次开机都执行但是脚本中有标记位判断，所以不会重复执行。

### 总结
通过上面的步骤就可以实现系统预装可卸载的应用了。归纳下主要步骤，然后按照实现如下步骤即可实现，步骤如下：  
> 
1. 复制预装应用至系统中
2. 将系统应用copy到data区的脚本
3. 在开机时能够执行脚本

当然上面仅仅只是一种方案，如果有更好的实现方案请指教，请留言，谢谢。