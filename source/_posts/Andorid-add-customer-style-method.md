title: "Android 系统添加自定义主题属性"
date: 2015-07-21 21:20:27
categories: Android
tags: [Android,Android系统]
---

### 问题
在开发中需要将开发的多个应用在刚启动的时候使用同一张图作为启动的背景图，且可以动态变化，图片放置在本地的某一个目录下面。一般设置每个应用程序启动的背景可以在主题中使用`android:windowBackground`属性开设置一张图片，如果每个应用要去使用同一张图那么就要在每一个应用中取添加配置且添加同样的图片进去，显然这样可以解决办法，但是不是万全的方法，每增加一个应用就得添加重复的图片以及添加属性。那么有没有更加便捷的方法呢，那就去看看系统添加这个背景是如何实现的。  
### 现有的windowBackground的实现方案

#### 查找windowBackground源码及使用  
`windowBackground`属性是在framework层提供的，所以直接在源码下搜索关键字`windowBackground`,加载什么资源在上层去处理，使用`jgrep windowBackground` 命令来搜索，jgrep 是只搜索java文件，没有工具包的，可以直接这样来进行搜索
`find ./ -iname "*.java" | xargs grep "windowBackground"`  
搜索得到的结果文件如下:
`frameworks/base/policy/src/com/android/internal/policy/impl/PhoneWindow.java`
在代码文件中，它使用的全名叫`com.android.internal.R.styleable.Window_windowBackground`,且是通过`mBackgroundResource = a.getResourceId(com.android.internal.R.styleable.Window_windowBackground, 0);`来获取设置的背景图的资源id，实际上这个值在系统中是有默认值的，取默认值这个方法是在`generateLayout(DecorView decor)`中，接着往这个方法向下看， 根据mBackgroundResource 这个变量发现在下面有将这个Id转化为Drawable对象，然后对DecorView 调用`setWindowBackground(drawable)`将转换的图片设置进去，看到这里是不是应该知道怎么怎么处理了呢。

#### 模仿windowBackground实现思路
对每个应用都设置一个类似windowBackground的属性，就实现自定义的背景，则这样只需要直接在系统进行统一处理，应用层就不用进行每个单独设置相同的图了，改起来也很方便。
为了不能一竿子打死，故这里要加个属性来区分是用统一的背景图还是用应用程序自己设置的，这样就不会影响第三方应用了。  
当然想到的是使用一个类似`android:windowBackground`的属性，比如`android:windowCustomBackground`,所以考虑如何在系统中添加一个自定义的属性，就相当于自己添加一个系统属性了。  
#### 实现自定义属性
在Android 源码系统目录下，要添加属性描述需要在`frameworks/base/core/res/res/values/public.xml`文件中添加自定义属性，搜索下参照`android:windowBackground`进行添加,然后在其后面添加要添加的属性，比如：`android:windowCustomBackground`，定义的语句按照上面的来，但是必须把id改了，且不能与其他的一样。  
**注意：** id必须不能重复，特别注意不能重复了。  
然后在`frameworks/base/core/res/res/values/attrs.xml`中添加相应的定义以及值的范围，attrs.xml 是自定义变量中常用到的属性描述文件。定义的内容还是按照前面`android:windowBackground`来，添加属性:`<attr name="windowCustomBackground" />`,添加属性值的限定， `<attr name="windowCustomBackground" format="boolean" />` 限定为boolean 类型的.
#### 取自定义属性值
在系统代码`frameworks/base/policy/src/com/android/internal/policy/impl/PhoneWindow.java`中间中取android:windowBackground的值，使用代码`getWindowStyle().getBoolean(com.android.internal.R.styleable.Window_windowCustomBackground, false)` 默认为false，如果为true则说明在应用XML中中有设置使用系统统一的背景图，在系统中图和实现呢，看下面代码：

```java  
	// 预定好的设置背景图的图片路径及名称    
	File fileBackground = new File("/data/theme/bg.png");
	if (getWindowStyle().getBoolean(com.android.internal.R.styleable.Window_windowCustomBackground, false) == true) {
	    if (fileBackground.exists()) {
	        drawable = new BitmapDrawable(BitmapFactory.decodeFile("/data/theme/bg.png"));
	    } else {
	        drawable = getContext().getResources().getDrawable(com.android.internal.R.drawable.default_wallpaper);
	    }
	} else {
	    drawable = getContext().getResources().getDrawable(mBackgroundResource);
	}  
```
上面其实也很简单，判断如果是使用系统统一的背景，然后若背景图存在则将其转换为drawable并赋值，如果不存在则使用系统默认的壁纸，如果不是统一的，则使用原生style自带的那张图。

#### 使用方法

使用很简单，和windowBackground类似，在项目中的style中，可以就按照下面的方式设置上面添加的style了。
```xml
    <item name="android:windowCustomBackground">true</item>
```
这样就行了。编译问题看下面的注意事项。

### 总结及注意事项：
1.	所以到这里大部分的功能都已经ok了，但是还有一个需要注意的地方。在Android源码目录下面`frameworks/base/api/current.txt`，此文件记录了所有的当前系统style以及各种资源文件，系统方法等。此文件为系统生成的，将资源文件添加完毕后，通过运行`make update-api`后，自动更新上述文件，参考文章[http://blog.chinaunix.net/uid-29535415-id-4144841.html](http://blog.chinaunix.net/uid-29535415-id-4144841.html)  

2.	如果要在本地的sdk中使用新的api或者资源文件，则需要把前面修改的`frameworks/base/core/res/res/values/attrs.xml`和`frameworks/base/core/res/res/values/public.xml`文件copy到本地sdk的`sdk/platforms/android-19/data/res/values/`目录下，这样Android Studio（Eclipse）就能识别到新的接口。注意是19，则需要编译时选择4.4进行编译。如果放在其他sdk目录下，注意下编译的版本就行。

最后编译完系统后，升级系统，安装上我们更新后的应用查看是否有效果。  
至此Android 添加自定义系统属性的方法就完成了，如有任何问题请留言交流，谢谢！
