title: "Linux 多硬盘开机自动挂载"
date: 2015-05-04 22:24:46
categories: Linux
tags: Linux
---

在Linux系统上使用多个固态硬盘的时候，默认只挂载系统盘，要想使用其他盘，我们需要挂载硬盘，下面就简单介绍下，我在挂载新硬盘的操作方法。  
###  查看系统中的未挂载的硬盘    
`sudo hdparm -I /dev/sdb`      硬盘硬件安装后，此命令测试linux系统是否能找到挂载的未分区硬盘。 
### 硬盘分区
由于我们这里不需要分区，所以就不用分区的操作了。
<!-- more --> 

###  格式化硬盘
使用命令`sudo mkfs -t ext3 /dev/sdb1 ` /dev/sdb1 为新硬盘在linux上的描述符。上面创建的新硬盘分区格式化为ext3格式，这个要等一会才能自动结束。
###  设置新硬盘的卷标
`sudo e2label /dev/sdb1 /code ` /code 为在/dev/sdb1新硬盘根目录下面起的名字
###  设置挂载点
`sudo mkdir /code `  //在根路径下创建挂载点  
###  设置开机自动挂载

使用命令`sudo vim /etc/fstab `编辑文件，文件的配置列表如下：

```
file system   mount point   type  options  dump  pass 
     a             b         c       d      e     f    
```
- a	指代文件系统的设备名。最初，该字段只包含待挂载分区的设备名（如/dev/sda1）。现在，除设备名外，还可以包含LABEL或UUID
- b	文件系统挂载点。文件系统包含挂载点下整个目录树结构里的所有数据，除非其中某个目录又挂载了另一个文件系统
- c	文件系统类型。下面是多数常见文件系统类型（ext3,tmpfs,devpts,sysfs,proc,swap,vfat）
- d	mount命令选项。mount选项包括noauto（启动时不挂载该文件系统）和ro（只读方式挂载文件系统）等。在该字段里添加用户或属主选项，即可允许该用户挂载文件系统。多个选项之间必须用逗号隔开。其他选项的相关信息可参看mount命令手册页（-o选项处）
- e	转储文件系统？该字段只在用dump备份时才有意义。数字1表示该文件系统需要转储，0表示不需要转储
- f	文件系统检查？该字段里的数字表示文件系统是否需要用fsck检查。0表示不必检查该文件系统，数字1示意该文件系统需要先行检查（用于根文件系统）。数字2则表示完成根文件系统检查后，再检查该文件系统

我们需要按照上面的格式将配置写入到文件中，注意每个之间要使用tab键来进行分开，我们的配置信息如下：
`/dev/sdb1     /code      ext3      defaults      1      2  ` /dev/sdb1 为硬盘的标识符
然后记得保存下文件。

### 重启查看是否挂载成功
使用`df -h`命令查看分区空间使用情况，就可以看到/code已经自动挂载。查看的结果如下：

```
Filesystem            Size  Used Avail Use% Mounted on   
/dev/sda1             139G  121G   12G  92% /
varrun                1.3G   68K  1.3G   1% /var/run
varlock               1.3G     0  1.3G   0% /var/lock
udev                  1.3G   32K  1.3G   1% /dev
devshm                1.3G     0  1.3G   0% /dev/shm  
/dev/sdb1             924G   11G  867G   2% /code
```
看到我们的/code 已经挂在上，则已经大功告成！

参考地址[http://www.cnblogs.com/avril/archive/2010/03/23/1692783.html](http://www.cnblogs.com/avril/archive/2010/03/23/1692783.html)