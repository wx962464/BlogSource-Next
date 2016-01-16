title: "如何使用android系统隐藏hide的类和方法"
date: 2015-03-31 21:48:37
categories: Android系统 
tags: [Android,Android系统]
---

在应用开发过程中，可能会需要使用到系统的方法，比如：`SystemProperties` 以及系统隐藏hide的方法和类，比如：Android 4.2中的  `Surface.screenshot(x,y)`方法是隐藏的， Android 4.3后面上面的那个方法变成了`SurfaceControl.screenshot(x,y)` 并且`SurfaceControl`这个类也变成了隐藏的了。那么要直接在eclipse或者android studio 中怎么弄呢？下面教大家方法：

<!-- more --> 

1.  首先编译Android的系统，其实直接编译framework也行，在编译完成后在`out\target\common\obj\JAVA_LIBRARIES\framework_intermediates`下面有个`classes.jar`的文件，我们就需要这个jar文件。
2.  将classes.jar放在某个文件夹下面，然后将其解压，我们可以得到一个`android`文件夹和`META_INF`文件夹，
3.  找到我们常使用的sdk版本目录下面的`android.jar`,比如:`sdk\platforms\android-19\android.jar`,将其放在某个文件夹下面解压。解压后会得到很多个文件夹，包括:`android,java,com,javax,org,META_INF`等文件夹以及`resources.arsc`文件。
4.  将上面`classes.jar`解压出来的android文件夹下面的所有文件，复制到`android.jar`解压出来的android文件夹里面，并覆盖相同文件名的文件及文件夹。其实你注意上面的两个android文件夹内容大致一样。
5.  下面到了最关键的一步，就是把现有的文件夹中的classes打包成java文件，首先在解压android.jar的根目录下面打开cmd命令窗口，输入命令`jar cvfm android.jar META-INF/MANIFEST.MF ./` 。如果找不到jar，请先配置环境变量。接着就看到这个打包的详细信息在控制台输出。关于jar的详细命令参数请看这里[jar命令详解](http://blog.csdn.net/kiss0931/article/details/210201)
6.  顺利的话，就可以得到一个`android.jar` 文件了，然后在把得到的jar文件解压看看是否和原来的结构一直，不要多打包一层文件目录哟，不然肯定没办法用的，如果正确的话，将替换我们sdk中的`android.jar`文件。例如替换：`sdk\platforms\android-19\android.jar`文件。
7.  然后测试，打开eclipse，随便在一个android工程中的java文件中输入`SurfaceControl`看看系统是否会提示我们导入`SurfaceControl`包,注意4.3以上才有`SurfaceControl`哟，如果没有则看看检测上面哪一步是否出错了。

##易错点总结：
1.  要保证第一步生成的`classes.jar`文件是正确的，有的系统打包的classes默认是在classes.odex中，如果解压出来没有android文件夹基本就是这个问题了。
2.  是否用`classes.jar`解压出来的android文件夹完全覆盖`android.jar`解压出来的android文件夹下面的内容。因为隐藏的方法都是在`classes.jar`解压出来的android文件夹下面。
3. 在打包命令的时候，文件夹跟目录错误，导致打包出来的jar多一层文件夹，或者少一层（少打包其他文件夹），大概`android.jar` 大小有20M左右。
4. jar 命令参数有误，请具体参看jar方法参数的使用方法和含义。

