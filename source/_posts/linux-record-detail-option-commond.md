title: Linux下记录操作命令的详细信息  
date: 2016-04-12 22:42:17  
tags: Linux  
---
最近编译服务器出现一个诡异问题，代码被莫名的修改了，于是乎就查找服务器上又有的登录信息，以及操作指令，但最终都没有找到比较有效的信息。但为什么还写这个呢，是因为这里记录下，怎么把服务器上的所有操作记录的信息记录全点呢？以及查看信息用到的一些简单命令。

## 查看用户登录的情况
常用的有`who`和`last`命令
### who命令
首先使用`who`命令可以查看当前服务器上有哪些用户在使用。
**语法：** who [-Himqsw][--help][--version][am i][记录文件] 
参数: 
　　-H   显示各栏位的标题信息列。 
　　-i或-u  显示闲置时间，若该用户在前一分钟之内有进行任何动作，将标示成"."号，如果该用户已超过24小时没有任何动作，则标示出"old"字符串。 
　　-m 　此参数的效果和指定"am i"字符串相同。 
　　-q或--count 只显示登入系统的帐号名称和总人数
who am i (whoami)这个命令查看当前终端是哪个用户的信息。
### last命令
使用`last`命令可以查看最近的服务器登录情况。
**语法：** last [-adRx][-f <记录文件>][-n <显示列数>][帐号名称...][终端机编号...] 
参数: 
　　-a 　把从何处登入系统的主机名称或IP地址，显示在最后一行。
　　-d 　将IP地址转换成主机名称。
　　-f　 <记录文件> 　指定记录文件。
　　-n　 <显示列数>或-<显示列数> 　设置列出名单的显示列数。
　　-R 　不显示登入系统的主机名称或IP地址。
　　-x 　显示系统关机，重新开机，以及执行等级的改变等信息。 
　　-i　 显示指定ip的登录情况   
　　-t　 显示YYYYMMDDHHMMSS之前的信息
单独执行last指令，它会读取位于/var/log目录下，名称为wtmp的文件，并把该给文件的内容记录的登入系统的用户名单全部显示出来。
> 默认读取的是wtmp文件，还有一个/var/log/btmp文件，这里面记录了更加全的信息，可以查看。可使用-f 参数指定文件，显示出来。 

## 查看历史操作记录
历史命令的操作我们常用的就是`history` 这个命令能看所有的操作命令，但是默认的很单调没有很多的详细信息，下面就是修改配置，来增加我们的详细信息。
### history命令
默认显示的是一个简单的编号和命令，想要查找一些有用的信息都无法查看到，所以要对其进行修改。
1. 修改history命令记录的长度和文件大小及显示时间格式
在系统修改`/etc/bash.bashrc`文件，在最后面加入下面的语句
```linux
HISTFILESIZE=20000
HISTSIZE=20000
HISTTIMEFORMAT="%Y%m%d-%H%M%S:"
export HISTTIMEFORMAT
```
2. 设置history默认的格式
在系统中修改`/etc/profile`文件，在文件的最后加入下面语句：
```shell
 #设置history格式
 export HISTTIMEFORMAT="[%F %T] [`who am i 2>/dev/null| awk '{print $NF}'|sed -e 's/[()]//g'`] "
 #实时记录用户在shell中执行的每一条命令
 export PROMPT_COMMAND='\
     if [ -z "$OLD_PWD" ];then
         export OLD_PWD=$PWD;
     fi;
     if [ ! -z "$LAST_CMD" ] && [ "$(history 1)" != "$LAST_CMD" ]; then
         logger -t `whoami`_shell_cmd "[$OLD_PWD]$(history 1)";
     fi ;
     export LAST_CMD="$(history 1)";
     export OLD_PWD=$PWD;'
```
上面这个脚本可以记录下在哪个目录执行了哪些操作以及时间等信息，这样就可以方便我们查看在服务器上谁什么时间在哪个目录执行了哪些操作。

> 需要退出终端重新进行登录，然后在执行`history` 就能看到效果了。

## 其他
1. time 函数
time 函数可以统计命令执行的时间，包括程序的实际运行时间(real time)，以及程序运行在用户态的时间(user time)和内核态的时间(sys time)。
例如：`time git pull` 则是统计更新代码花费多长时间