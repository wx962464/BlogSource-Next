title: Anroid系统解决扫码枪无法输入字母和字符问题
date: 2016-11-09 22:19:43
tags: [Android系统]
---
### 问题：
在使用扫码枪扫码条码的时候明明有字母和字符，但是输入到Android系统却没哟，输入到电脑是正常的，这就很奇怪，让一个搞上层开发的摸不着头脑，最后和系统讨论才知道是系统按键部分映射被删除导致的。

### 解决办法：
1. 在Android系统层`frameworks/base/data/keyboards`文件夹下面有`Generic.kl`这个文件，此文件为Android默认的按键映射对应表，还有其他的比如：`qwerty.kl`文件，以及一些自定义码值的kl文件。
2. 打开`Generic.kl`看看类型也许就明白了.
```
 key 11      0
 key 2       1
 key 3       2
 key 4       3
 key 5       4
 key 6       5
 key 7       6
 key 8       7
 key 9       8
 key 10      9
 key 12      MINUS
 key 13      EQUALS
 key 14      DEL
 key 15      TAB
```
里面是键与键值的映射，比如：键值11 对应的按键为 0 这个，以此类推。那解决就明朗了，将所有字母和字符的按键映射添加进行就ok了，至于按键值是多少我这边直接参考了另外一个平台的`Generic.kl`文件。重新编译系统验证，此问题解决了。

### 疑惑问题：
1. 用相同Android版本的android.jar 查看keyCode对应的值和`Generic.kl`文件里描述的不一样，此问题还没有弄明白为什么，系统说两个是不相关的？
2. 发现在两个平台上有大部分按键值在一致的，但存在分别的是不样的，不明白怎么定义的。依据是啥？

#### 参考文章
[http://blog.csdn.net/kangear/article/details/12110951](http://blog.csdn.net/kangear/article/details/12110951)
[http://blog.csdn.net/mjsornp/article/details/39988275](http://blog.csdn.net/mjsornp/article/details/39988275)
[https://www.zhihu.com/question/20830530](https://www.zhihu.com/question/20830530)
