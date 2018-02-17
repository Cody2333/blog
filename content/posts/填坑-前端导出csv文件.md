---
title: 填坑-前端导出csv文件的方法
tags: [
    "javascript",
    "web前端"
]
date: 2017-04-18
categories: [
    "javascript"
]
---


在前后端分离的情况下，通常前端浏览器提供用户文件下载的功能的实现方式如下：

1. 前端请求后端api
2. 后端根据前端的请求生成一个文件
3. 后端返回一个下载的url地址
4. 前端利用a标签的download属性，实现用户的点击下载。

在jquery中的dataTable有非常方便的导出csv的功能，但是jquery对于现在的web来说太重了，我也不需要其中的大部分功能，所以我实现了一个简单的csv文件生成导出的模块，方便日常使用。

## 是否应该在前端导出csv文件？
对用用户需要简单的导出表格中数据的功能，通过前端实现，一方面可以减少一次api的请求，提升响应速度，另一方面，有时候客户可能确实需要这样一个在前端导出csv文件的功能，比如用户需要选中表格中某几行数据进行导出，显然在前端可以更便捷灵活的操作。

## demo
本文中，我将实现一个将表格数据转换为csv的格式，并生成一个指定文件名的csv文件，供用户下载的功能。

我将该功能进行了封装发布到了npm上
[csv-exportor](https://www.npmjs.com/package/csv-exportor)
demo实现基于前端框架vue2.0，但是事实上适用于任何前端框架。
[示例demo](https://cody2333.github.io/csv-exportor/)
[源代码github地址](https://github.com/Cody2333/csv-exportor)
## 具体实现
本身的实现其实不难，但是遇到了挺多的坑，所以总结一下。

#### 将表格数据转换为csv格式
这里为了方便起见，我使用了库comma-separated-values，无论性能还是使用都没什么问题。
```js
import CSV from 'comma-separated-values';
// data为表格数据，options可配置表头等信息，返回编码后的csv
const encoded = new CSV(data, options).encode();
```

#### 如何生成文件?

前端如何生成二进制文件？
- Blob
使用Blob对象类型。实际上，File对象就是继承自Blob，所以在大多数地方，Blob和File是可以共用的。我们可以通过Blob来生成二进制流。

- URL
现代浏览器提供了方法`URL.createObjectURL`用来创建data url

所以我们可以使用Blob和URL来封装我们格式化的csv数据，得到dataUrl。

#### 如何下载文件?
为了实现点击一个按钮，下载导出csv文件的功能，我们可以利用a标签来完成。
1. 动态创建一个隐藏的a标签
2. href 属性 = 上一步生成的dataUrl
3. download 属性 = 指定文件名
4. 模拟点击a标签
5. 释放URL对象
6. 成功实现文件下载

具体实现如下：
```js
import CSV from 'comma-separated-values';

/**
 * generate data url of the generated csv file
 * @param {*} data 
 * @param {*} options 
 * @returns {String} dataUrl of generated csv file 
 */
function genUrl(data, options) {
  const encoded = new CSV(data, options).encode();
  const dataBlob = new Blob([`\ufeff${encoded}`], { type: 'text/plain;charset=utf-8' });
  return window.URL.createObjectURL(dataBlob);
}

/**
 * generate csv file from data, and automatically download the file for browser
 * @param {*} data 
 * @param {*} options 
 * @param {*} fileName 
 */
function downloadCsv(data, options, fileName) {
  const url = genUrl(data, options);
  const a = document.createElement('a');
  a.href = url;
  a.download = fileName;
  a.click();
  window.URL.revokeObjectURL(url);
}

export default {
  genUrl,
  downloadCsv,
};

```

## 遇到的问题

在过程中遇到的问题，其实大部分和excel对csv的呈现有关，但是考虑到用户在导出csv之后都是用excel打开的，所以不得不好好填坑了。

#### 中文编码
最初在生成csv之后，使用文本编辑器都是可以正常显示的，使用的是UTF-8的编码，但是用excel打开csv文件就会乱码

###### 解决方法

excel不能识别无BOM的UTF-8编码，需要转为带BOM的UTF-8编码，就是在文件头加上`\ufeff`即可
```
  const dataBlob = new Blob([`\ufeff${encoded}`], { type: 'text/plain;charset=utf-8' });
```

#### 文本自动转数字
excel打开csv的时候会自动把内容是数字的文本转换为数字，这导致了手机号会被用科学计数法显示，以及一些0开头的数字会被自动删除前面的0。

###### 解决方法
现在暂时没有完美的解决方法，只能在csv的数字文本前加一个单引号，如`"'0012","'0013"`
单引号表示的语义是【这个单元格是一个数字， 但是我想要用文本的方式来呈现】
但是excel不会自动去掉这些单引号，需要在excel中进行一步额外的操作
`全局替换文件中的单引号为单引号`
即可。
