title: Android Studio gradle 编译提示‘default not found’ 解决办法
date: 2015-09-26 16:05:48
tags: [Android,Gradle]
---
# 欢迎访问 [wxtlife.com](http://www.wxtlife.com)
在导入studio工程的时候，进行sync的时候，提示`Error:Configuration with name 'default' not found.` 
之前由于对gradle不熟悉，所以没有找到原因，其实也是偷懒，没有认真去排查问题，今天又遇到了，就折腾了会，把所有的配置文件都打开看，最终解决问题了，发现尽然是个低级的不能低级的问题，故记录下，警醒自己。
     
1.  打开settings.gradle发现里面有很多个`include ':app'`这样的include，然而发现在工程的目录下面根本没有include的项目，所以将需要include的项目添加进来，如果include的项目不需要，则将其include语句直接删掉，重新sync尝试。
2.  按照上面的操作，要么添加了相应inlcude的工程进来，但是发现还是会提示这样的`default not found`语句，怎么回事呢，原来用gradle编译的工程，每个工程下面都必须要有`build.gradle`文件,才能够编译include的工程。整个大工程才能sync通过。把include工程中都添加上相应的gradle配置文件，再重新进行sync，整个工程都通过了。
     
终于处理掉了这个问题，其实这个问题是很简单的问题，只是怪自己年少无知，故现在记录下这些曾经无知的过错，等以后反过来看自己也许会一笑而过。

## 欢迎访问 [wxtlife.com](http://www.wxtlife.com)