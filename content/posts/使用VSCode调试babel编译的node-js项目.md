---
title: 使用VSCode调试babel编译的nodeJs项目
tags: [
    "vscode",
    "nodejs"
]
date: 2017-08-16
categories: [
    "nodejs"
]
---

一般情况下，我们会在node项目中使用babel，以便充分的使用最新的JS语法规范，以及提前使用还在草案阶段的语法。但是当我们使用babel之后，我们在开发调试的时候一般会使用 babel-node 来代替node指令，使用babel-node之后进行调试，与直接使用node会有一些细微的区别，本文介绍如何使用VSCode调试使用babel-node启动的程序。

在package.json的scripts中，添加debug指令。
```
"scripts": {
   debug": "babel-node src/main --inspect --debug-brk --nolazy src/main"
}
```

在 项目目录下，新建.vscode文件夹，创建launch.js文件，内容如下：
```
{
  // Use IntelliSense to learn about possible Node.js debug attributes.
  // Hover to view descriptions of existing attributes.
  // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "request": "launch",
      "name": "Launch-vai-babel-node",
      "runtimeExecutable": "npm",
      "runtimeArgs": [
        "run",
        "debug"
      ],
      "port":9229,
      "env": {
          "NODE_ENV": "develop"
      },
      "console": "internalConsole"

    }
  ]
}
```

至此配置基本完成。

切换到VSCode的debug标签，运行就可以了。


![image.png](http://upload-images.jianshu.io/upload_images/2563527-d0d639278b3f6e7e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

然后就可以实现基本的断点和单步调试等功能了。
