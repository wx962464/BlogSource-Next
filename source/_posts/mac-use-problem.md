title: Mac 使用及开发技巧汇总（持续更新）
date: 2015-10-21 00:15:50
tags: Mac
---
# 欢迎访问 [wxtlife.com](http://www.wxtlife.com)

1.  在Mac中有时候想要看到隐藏文件，可以在终端使用下面命令显    示或隐藏隐藏文件。
    `defaults write com.apple.finder AppleShowAllFiles -bool     true`  **显示隐藏文件命令**
    `defaults write com.apple.finder AppleShowAllFiles -bool     false`  **不显示隐藏文件命令**

    命令运行之后需要重新加载Finder：快捷键option+command+esc，选中Finder，重新启动即可。
    
2. 配置全局环境变量或者用户环境变量
    有时候需要配置开发环境，比如`JAVA_HOME` `ANDROID_HOME` `adb` `Nodejs`的环境变量，我们可以直接修改系统的环境变量，但是一般不建议这样修改，因为修改系统内容层级太高。我们可以在用户的根目录下面创建一个环境变量的文件，每次启动的时候会自动加载该配置文件，文件名为：`.bash_profile` （**注：Linux 里面是 .bashrc 而 Mac 是 .bash_profile**） 然后放在根目录，也就是`"~/"`下面,例如我配置的环境变量如下:
    ```shell
    cat ~/.bash_profile

    JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.7.0_71.jdk/Contents/Home
    export JAVA_HOME
    PS1="[\u@\w]\$ "
    export PS1
    GRADLE_HOME=/Users/Aaron/common/gradle-2.4
    export GRADLE_HOME
    export PATH=$PATH:$GRADLE_HOME/bin
    ANDROID_HOME=/Users/Aaron/Library/Android/sdk
    export PATH=$PATH:$ANDROID_HOME/platform-tools
    NODE_JS_PATH=/usr/local/bin
    export PATH=$PATH:$NODE_JS_PATH
    ```
    上面配置地址最好使用绝对路径，按照上面篇日志的话，就只有当前用户有这些环境变量，如果电脑之后自己一个账号使用则没有问题的，如果多人使用则其他用户使用不了这些环境变量，如果要使用则要配置系统的环境变量，可以修改系统的`/etc/bashrc`,`/etc/profile`,`/etc/paths`这三个文件。
Mac系统的环境变量，加载顺序为：
**/etc/profile > /etc/paths > ~/.bash_profile > ~/.bash_login > ~/.profile**
当前配置好后，需要注销下，如果不想要注销则直接使用`source ~/.barsh_profile`来加载环境变量，按照上面的就可以配置不同权限的环境变量了。参考地址：[http://www.flakor.cn/2014-09-14-714.html](http://www.flakor.cn/2014-09-14-714.html )

3.  Mac连接Android手机进行调试
    首先要开启开发者模式，一般都是在系统设置中，狂点系统版本号，在接上mac后没有反应，找不到设备，是因为没有把的usb设备加入到adb设备列表中，先接上手机 具体操作方法如下：
**关于本机 > 系统报告 > USB** 在最下面会看到你手机的信息，主要看有个**供应商ID** 比如小米的为`0x2717`，一般同一个品牌的ID是一致的，然后将这个ID写入到.andorid目录下的adb_usb.ini 文件中，命令`echo 0x2717 >> ~/.android/adb_usb.ini`,或者直接在`~/.android/`下面新建一个文件名为`adb_usb.ini`的文件，然后将供应商ID写入文件中，如果有过个就写在不同行中，然后重启adb服务再看看就OK了（`adb kill-server,adb start-server`）。
参考地址：[http://mobile.51cto.com/aprogram-386942.htm](http://mobile.51cto.com/aprogram-386942.htm)

> **后面遇到的问题会持续更新**

## **更新** ##
1.  在Finer中如何显示文件的完整路径
    执行以下命令：`defaults write com.apple.finder _FXShowPosixPathInTitle -bool TRUE;killall Finder` 即可在Finder中显示完整的文件路径了。想取消此项设置，则执行以下命令即可解决：`defaults delete com.apple.finder _FXShowPosixPathInTitle;killall Finder `

## 欢迎访问 [wxtlife.com](http://www.wxtlife.com)