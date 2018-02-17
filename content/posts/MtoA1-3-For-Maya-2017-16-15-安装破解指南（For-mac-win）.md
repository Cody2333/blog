---
title: MtoA1.3 For Maya2017/16/15 安装破解指南（For mac/win）
tags: [
    "动画",
    "Maya",
    "其他"
]
date: 2016-08-18
---

# MtoA1.3 For Maya 2017/16/15 安装破解指南（For mac）

> 资源下载地址：链接: https://pan.baidu.com/s/1jH8IVSa 密码: 9bh4
> 为了在mac上装arnold solidangle mtoa 并且破解还是花了我不少时间的，网上的破解教程都很不完善，所以在这里分享一下我的经验。本人在maya2016 + mac 上成功了，其他的版本以及windows版本理论上来说也都是可行的。

### STEP 1：下载安装包
下载安装包，根据自己对应的版本，解压文件。注意压缩包是分文件夹的，把它们拷到一个文件夹下在解压。得到 `amped` 文件夹


![文件结构.png](http://upload-images.jianshu.io/upload_images/2563527-40a904a2cfe1609c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![文件结构2.png](http://upload-images.jianshu.io/upload_images/2563527-1dc6f2acfd933241.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### STEP 2 ：安装破解
##### 文件内容解释
- rlm : 启动rlm服务器的文件，请在终端中运行。`cd` 到 `rlm` 所在文件目录，运行 `chmod +x rlm`，然后运行`./rlm`，即可启动`rlm`服务器
- rlmutil : rlm服务器辅助脚本，可以不用，使用方式同`rlm`。
- *.pkg : mtoa的安装包，双击运行安装
- amped.txt : 安装教程，其实只要看这个安装教程就好了

### STEP 3 ：使用方法与原理
首先需要配置一下系统环境变量，具体方式在amped.txt中都有

![破解方式.png](http://upload-images.jianshu.io/upload_images/2563527-e459c720441661ed.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

注意使用MtoA的时候RLM Server一定要是运行状态，否则就无法破解。破解之后，arnold的水印就会消失，就可以正常使用了。


原理上应该就是在自己的机器上启动一个 RLM Server , 在 mtoa验证破解的时候，伪装成它的验证服务器，所以在使用MtoA的时候，我们就必须一直开着服务器，也就是说要让rlm脚本一直运行在终端。

done！
