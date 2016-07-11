title: 使用系统key文件生成keystore
date: 2016-07-11 20:36:16
tags: [Android系统,keystore]
---
首先找到系统对应系统签名keys的位置，一般在源码的`android\build\target\product\security`，此文件夹下面有4个标准的key。

- testkey -- a generic key for packages that do not otherwise specify a key.
- platform -- a test key for packages that are part of the core platform.
- shared -- a test key for things that are shared in the home/contacts process.
- media -- a test key for packages that are part of the media/download system.

**注：**我们需要制作的keystore需要用到platform的key。

直接签名和制作keystore都是需要使用`platform.x509.pem`和`platform.pk8`这两个文件。

## 直接给应用签名
直接签名还需要一个`signapk.jar`文件，此文件位置在`out/host/linux-x86/framework/signapk.jar` 在利用上面文件签名，命令如下：
`java -jar signapk.jar platform.x509.pem platform.pk8 app.apk app_signed.apk `

`app_signed.apk`为经过系统签名的apk了。

## 制作keystore
使用上面两个文件来生成keysotre。按照如下步骤进行生成：
1. 生成platform.pem
`openssl pkcs8 -inform DER -nocrypt -in platform.pk8 -out platform.pem`
2. 生成platform.pk12
`openssl pkcs12 -export -in platform.x509.pem -out platform.p12 -inkey platform.pem -password pass:android -name androiddebugkey`
3. 生成keystore文件
`keytool -importkeystore -deststorepass android -destkeystore ./platform.keystore -srckeystore ./platform.p12 -srcstoretype PKCS12 -srcstorepass android`

**注：**有的生成的为`platform.jks`,可以直接改为.keystore后缀，不影响使用的。

基于以上我们就能够制作出keystore。

> 参考链接：
http://elsila.blog.163.com/blog/static/173197158201211172281242/
https://developer.android.com/studio/publish/app-signing.html
http://jmlinnik.blogspot.com/2011/12/keystores.html
