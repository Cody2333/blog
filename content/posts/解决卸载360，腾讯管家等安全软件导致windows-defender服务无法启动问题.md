---
title: 解决卸载360，腾讯管家等安全软件导致windows-defender服务无法启动问题
tags: [
    "windows defender",
    "其他"
]
date: 2017-09-18
categories: [
    "其他"
]
---

## 简介
不小心被安装了腾讯管家，卸载之后windows defender无法打开。

如果尝试在服务中手动启动，会报577错误。

网上找了很多解决方案都无效

## 解决方法
使用管理员身份打开命令行CMD。
输入如下命令：
`reg add "HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows Defender" /v "DisableAntiSpyware" /d 0 /t REG_DWORD /f `

再次打开windows defender 即可。

实际上是就是在注册表中加了一个key，表示不禁用windows defender。


![image.png](http://upload-images.jianshu.io/upload_images/2563527-a0ec57fc1d871dac.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
