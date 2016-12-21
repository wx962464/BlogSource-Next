title: Android默认系统声音/大小修改及配置
date: 2016-12-21 22:36:39
tags: [Android,Android系统]
---
**本文是基于Android5.1的代码**
在做定制需求的时候，需要修改系统通知的声音，将其禁用掉，避免第三方应用发送通知时，声音很大，吓着用户。索性就把通知声音关掉。下面就说说关闭声音的几种方法,以及修改系统默认声音的方法。

### 1. 直接修改系统层默认的声音大小
在系统代码`frameworks/base/media/java/android/media/AudioService.java`的开头定义了两个数组，一个`MAX_STREAM_VOLUME` 这里面定义了各种声音的最大值（**最大值不是100**，所以需要`AudioManager.getStreamMaxVolume(type)`来获取各个音量的最大值），然后进行设置。
还定义了一个数组`DEFAULT_STREAM_VOLUME` 这里面则和`MAX_STREAM_VOLUME`里定义的顺序是一样，表明了各种声音的默认的大小。此块代码如下：
```java
   /** @hide Maximum volume index values for audio streams */
    private static int[] MAX_STREAM_VOLUME = new int[] {
        5,  // STREAM_VOICE_CALL
        7,  // STREAM_SYSTEMX_STREAM_VOLUMEMAX_STREAM_VOLUMEMAX_STREAM_VOLUME
        7,  // STREAM_RING
        15, // STREAM_MUSIC
        7,  // STREAM_ALARM
        7,  // STREAM_NOTIFICATION
        15, // STREAM_BLUETOOTH_SCO
        7,  // STREAM_SYSTEM_ENFORCED
        15, // STREAM_DTMF
        15  // STREAM_TTS
    };

    private static int[] DEFAULT_STREAM_VOLUME = new int[] {
        4,  // STREAM_VOICE_CALL
        7,  // STREAM_SYSTEM
        5,  // STREAM_RING
        11, // STREAM_MUSIC
        6,  // STREAM_ALARM
        5,  // STREAM_NOTIFICATION
        7,  // STREAM_BLUETOOTH_SCO
        7,  // STREAM_SYSTEM_ENFORCED
        11, // STREAM_DTMF
        11  // STREAM_TTS
    };
```
如果我们需要修改默认的通知声音，则可以将`STREAM_NOTIFICATION` 前面的数值 5 给为 0即可，这样默认声音就为0 了。

### 2. 修改数据库中的通知声音值
媒体声音这些数据在数据库中都会默认的存放数据，我们知道大多数的数据都是系统初次启动的时候在`SettingProvider`应用中加载初始化的值，当然通知的声音也在里面。

<!--more -->

具体的代码在`frameworks/base/packages/SettingsProvider/src/com/android/providers/settings/DatabaseHelper.java` 其中有个方法`loadVolumeLevels(db)` 此方法则是加载所有默认声音大小的地方，具体代码如下：
```java
stmt = db.compileStatement("INSERT OR IGNORE INTO system(name,value)" + " VALUES(?,?);");

loadSetting(stmt, Settings.System.VOLUME_MUSIC,     AudioService.getDefaultStreamVolume(AudioManager.STREAM_MUSIC));

loadSetting(stmt, Settings.System.VOLUME_RING,          AudioService.getDefaultStreamVolume(AudioManager.STREAM_RING));

loadSetting(stmt, Settings.System.VOLUME_SYSTEM,        AudioService.getDefaultStreamVolume(AudioManager.STREAM_SYSTEM));

loadSetting(stmt,Settings.System.VOLUME_VOICE,
AudioService.getDefaultStreamVolume(AudioManager.STREAM_VOICE_CALL));

loadSetting(stmt, Settings.System.VOLUME_ALARM,         AudioService.getDefaultStreamVolume(AudioManager.STREAM_ALARM));

loadSetting(stmt,Settings.System.VOLUME_NOTIFICATION,
AudioService.getDefaultStreamVolume(AudioManager.STREAM_NOTIFICATION));

loadSetting(stmt,Settings.System.VOLUME_BLUETOOTH_SCO,
AudioService.getDefaultStreamVolume(AudioManager.STREAM_BLUETOOTH_SCO));

```
我们发现loadSetting中把所有声音相关默认值大小的都写入数据库中了，那么我们就可以从这里下手了，在`Settings.System.VOLUME_NOTIFICATION`的设置项中我们就把他设置为0，则系统通知默认的声音就为0 ，我们再看看`AudioService.getDefaultStreamVolume`这个方法的实现.
```java
    public static int getDefaultStreamVolume(int streamType) {
        return DEFAULT_STREAM_VOLUME[streamType];
    }
```
实际就是返回了我们在方案一中系统里面默认音量大小数组里面的值。所以方案一和方案二实际是一个效果

### 3. 修改ro.config.notification_sound的属性值
此属性值的意思就是通知默认的音乐文件文件名，我们在系统代码`build/target/product/full_base.mk` 中定义了，如果我们不想有声音那么我们可以将默认值改为不存在的文件，则不会播放通知声音了，当然我们也可以在客户定义的mk中使用`PRODUCT_PROPERTY_OVERRIDES` 去复写此属性，将其指定为不存在文件或者为空，这样就不会有通知声音响了。

### 4. 修改默认的声音 
系统默认了很多的声音，那么我们要修改一些系统里默认的音效文件呢，那么我们可以修改`frameworks/base/data/sounds`下面文件及文件夹中的声音文件，如果改了名字记得要在mk中将原来的替换为新的名字。这下面的mk的作用是将这些音乐文件全部打包到系统`system/media/audio`下面各个模块的文件，然后在系统开机的时候，扫描这些文件，将其加入到数据库中，之后在设置中更换声音时，则直接从数据库中查询这些音乐文件，然后供用户选择。

### 总结
系统的媒体这块是很重也很大的一块，这里只是一点点皮毛，只是用到时查到的，要想系统系统学习还得很多工作需要研究。如有问题请及时留言反馈。