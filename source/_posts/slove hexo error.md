title: "Hexo 使用中遇到的问题总结"
date: 2015-03-24 22:53:13
tags: hexo
---
# 欢迎访问 [wxtlife.com](http://www.wxtlife.com)
1.  安装NoteJs，出现问题，安装到最后提示`error 52**`
    过程：重新下载安装了几次都不行，不懂为什么，最后通过Hexo的文档提供下载地址进行下载，然后安装问题就没有出现了。
    
    可能原因：
        a.  下载的安装包有问题的原因
        b.  我的C盘占用过多，盘符标红，然后卸载一些不常用软件解决，再安装再加上上面重新下载的安装包，之后安装成功。
<!-- more --> 
2.  部署提示找不到git
    解决办法：
    在Hexo 3.0版本后`deploy git` 被分开的，所以需要安装，安装命令如下:`npm install hexo-deployer-git --save` ,安装好后在尝试一下就ok。
    
3.  部署提示 ｀event type error **\*｀
     解决办法： 
安装了`git bash`没有配置到环境变量`path`中，添加进去在试试。

4. 部署的时候执行：`hexo deploy` 命令行没有任何输出，也没有错误。
    解决办法：
    在部署的`_config.yml`文件中，找到`deploy:`标签，在每个冒号后面必须要空格，否则就会出现上述问题。我的配置如下：
    ```
    deploy:
  type: git
  repository: https://github.com/wx962464/wx962464.github.io.git
  branch: master
  ```
  顺便提示下，如果使用ssh部署不成功的话，请使用https的方式试试，这个就是每次会让你输入用户名和密码。其实效果是一样的。
  
5. 修改主题不起作用，而且`hexo generate`还报错
    解决办法：
    需要到相应的主题文件夹下面进行修改，比如我的主题为：`themes\jacman` 则在根目录下找到该文件夹下，修改`_config.yml`文件，根目录下面也有个同样的名字，不注意，容易弄混，要主要修改的文件是否正确。

6. 执行`hexo server`显示`running at  http://0.0.0.0:4000/`
    问题说明：
    开始的时候以为启动服务器有问题，一直在找问题，找了半天没有答案，最后在浏览器直接尝试http://0.0.0.0:4000/ 是没办法访问的，然后就试了下[http://localhost:4000/](http://localhost:4000) 发现是可以访问的，大喜！~~

7. 执行`hexo server`提示找不到该指令
    解决办法：
    在Hexo 3.0 后server被单独出来了，需要安装server，安装的命令如下：`npm install hexo-server --save` 安装此server后再试，问题解决。

以上就是我在这几天使用Hexo的一些问题，当然问题列的不够详细，只是一个大致思路，这些也是凭着自己的印象做的笔记，所以有些错误的地方希望大家指出，共同学习，共同进步！

## 欢迎访问 [wxtlife.com](http://www.wxtlife.com)