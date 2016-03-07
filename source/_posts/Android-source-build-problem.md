title: "Android 源码编译服务器问题汇总"  
date: 2015-06-21 18:40:24  
categories: Linux 
tags: [Android,Linux]
---

### **安装工具及服务**  
需要安装的服务有`samba ssh vim procmail`,其中procmail为mtk系统编译中用到的，其他平台可不需要此。  

###  **开启smaba服务**
   - 首先，在终端窗口输入 `sudo apt-get update` 然后按照需要输入密码。然后进行安装samba 服务，安装命令`sudo apt-get install samba samba-common`等待安装完成
   - 后需要修改samba的配置文件，配置samba开放用户目录，修改命令`sudo vim /etc/samba/smb.conf` 找到[homes]位置, 改为如下:
   - 
    ```
    246 # user's home director as
    247 [homes] (修改点)
    248 comment = Home Directories (修改点)
    249 browseable = no  (修改点)
    250
    251 # By default, the home directories are exported read-only. Change the
    252 # next parameter to 'no' if you want to be able to write to them.
    253 read only = no (修改点)
    254
    255 # File creation mask is set to 0700 for security reasons. If you want to
    256 # create files with group=rw permissions, set next parameter to 0775.
    257 ; create mask = 0700
    ```
在修改完后，还需要在最下面添加如下的命令,这样在挂载其他非系统盘的时候就可以自由的访问了，下面这个坑了很久。故记录之。
    
    ```
    [global]
    follow symlinks = yes
    wide links = yes
    unix extensions = no
    ```
   -  接着开通samba用户， 命令`sudo smbpasswd -a xxxx` (xxxx 为用户的名称) 输入xxxx的密码(第一次sudo 要输入) --> samba 密码--> 确认samba 密码。
   - 重新启动samba服务，命令`sudo service smbd start (ubuntu 10.4)` 或者`sudo service samba restart (ubuntu 12.04`
   
   **安装samba参考** [http://jingyan.baidu.com/article/00a07f38b9194082d028dc08.html](http://jingyan.baidu.com/article/00a07f38b9194082d028dc08.html)    

>  提示，创建用户如下所示：  

	添加账户：useradd -r -m -s /bin/bash username  
	修改密码：sudo passwd username 

   <!-- more --> 
   
### **ssh的安装与配置**
 - 安装ssh参考
    [http://jingyan.baidu.com/article/9c69d48fb9fd7b13c8024e6b.html](http://jingyan.baidu.com/article/9c69d48fb9fd7b13c8024e6b.html)
在Ubuntu 14.04上面要注意，使用gedit修改配置文件"/etc/ssh/sshd_config"
打开"终端窗口"，输入`sudo gedit /etc/ssh/sshd_config`-->回车-->把配置文件中的`PermitRootLogin without-password`加一个"#"号,把它注释掉-->再增加一句`PermitRootLogin yes`-->保存，修改成功。第一个操作是允许设置密码的root用户登录，第二个操作是允许root用户登录。

大致上述操作就能满足基本使用了。

### 特技

####如何让sudo操作不用输入密码
修改/etc/sudoers 里面的配置：
     
```
     # User privilege specification
       root    ALL=(ALL:ALL) ALL
     # Members of the admin group may gain root privileges
       %admin ALL=(ALL) NOPASSWD: ALL #(写入NOPASSWD)
    # Allow members of group sudo to execute any command
      %sudo   ALL=(ALL) NOPASSWD: ALL 
      cvtouch ALL=(ALL) NOPASSWD: ALL
```   
### 总结及注意事项

1.	上面注意要修改sudoers的权限，但是注意不要改为777，不然后面很麻烦。如果已经改了，可以见下面这个文章中的操作来解决.[http://blog.csdn.net/davidsky11/article/details/23478131](http://blog.csdn.net/davidsky11/article/details/23478131)   
    
2.	当我们用root用户账号登录，使用远程操作如jsch库来操作，操作`git pull及 repo forall -c git pull` 都不起作用，导致远程无法更新代码，原因是权限太高使用的不是user/bin 下面的repo 而是系统/bin 下面的repo，导致无法更新，操作办法是将 user/bin 下面的复制到系统的/bin下面，再次重新登录尝试就ok了。
