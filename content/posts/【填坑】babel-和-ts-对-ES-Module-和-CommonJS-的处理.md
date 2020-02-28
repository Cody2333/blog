---
title: 【填坑】babel-和-ts-对-ES-Module-和-CommonJS-的处理
tags: [
    "tech","js"
]
date: 2019-05-12
---

## 简介

ES Module 导入导出的语法我们其实已经用了很久了，但是通常我们都是通过一些工具将我们的代码转换到 cjs 模块再去执行，比如 babel，但是不同的工具，甚至 babel 的不同版本的行为也是不一致的，这会给我们的使用带来很多困扰。这篇文章就简单介绍一下 ESM 和 CJS 两者之间**相互导入导出**存在的问题，各工具实现**标准不一致**的原因，以及现阶段可能的**最佳实践**。

## ES Module

```
// 导出
// deafult 导出，可以导出任何普通常量或者对象，只允许有一个 default 导出
// file1.js
// export default const a = 1
// export default class {}
export default {a:1, b:2}
// 普通导出，可以有任意多个
export const c = 3
const d = 4
const e = 5
export {d, e}

// 导入
import x from './file1' // { a: 1, b: 2 }
import * as x from './file1' // { default: { a: 1, b: 2 }, c: 3, d: 4, e: 5 }
import {d} from './file1' // 4
```

## CJS

```
// CJS 比较简单，每个模块只有一个导出 module.exports
// file2.js
// 导出
module.exports = {a:1, b:2}

// 导入
const x = require('./file2')  // { a: 1, b: 2 }
```

## 两者的相互导入

由于没有标准的定义两者相互导入的行为，这就导致了不同工具不一致的行为，二罪魁祸首就是 default 导出这个行为。如果 ESM 没有 default 导出，那么
`import x from './file1'` 就会返回 `undefined`，而对于 cjs 的模块，默认导入这一行为不同的工具有可能会有不一致的行为了。

在最初尝试将项目从 babel 迁移到 typescript 上时，就遇到了这个问题。对于 babel 而言，会把 cjs 模块作为 default 导入。参考下面这个例子
```
// cjs.js
module.exports = {
  a:1,
  b:2
}
// esm.js
export default {a:1,b:2}
export const c = 3
const d = 4
const e = 5
export {d, e}

// index.js
import c from './cjs'
import {a} from './cjs'
import es from './esm'
import * as es2 from './esm'
import {d} from './esm'
console.log(a)
console.log(c)
console.log(es)
console.log(es2)
console.log(d)
```
经过 babel 转译后的代码如下：
```
// index.js
var _cjs = require('./cjs');
var _cjs2 = _interopRequireDefault(_cjs);
var _esm = require('./esm');
var es2 = _interopRequireWildcard(_esm);
function _interopRequireWildcard(obj) { if (obj && obj.__esModule) { return obj; } else { var newObj = {}; if (obj != null) { for (var key in obj) { if (Object.prototype.hasOwnProperty.call(obj, key)) newObj[key] = obj[key]; } } newObj.default = obj; return newObj; } }
function _interopRequireDefault(obj) { return obj && obj.__esModule ? obj : { default: obj }; }
console.log(_cjs2.default);
console.log(_cjs.a);
console.log(es2.default);
console.log(es2);
console.log(_esm.d);

// esm.js
"use strict";

Object.defineProperty(exports, "__esModule", {
  value: true
});
exports.default = { a: 1, b: 2 };
var c = exports.c = 3;
var d = 4;
var e = 5;
exports.d = d;
exports.e = e;
```

对于 ESM 文件， babel 添加了 _esModule 属性表示该模块为 ES Module， 然后在 exports 上绑定了 default 输出

对于 index.js 文件，可以发现在 导入 cjs 文件时，首先 require 该文件，然后将其所有导出的字段绑定到了 default 字段。

然而对于 Typescript，最初它是和 babel 这样的导入形式不兼容的。
`import React from 'react'` 这样会报错。

TypeScript 的出现远早于 ES2015 定稿。而 ES 规范中的标准化模块系统是从 ES 2015 中才开始出现的，所以最初 TS 使用的模块系统叫做 Namespace。后来由于 ESM 的标准定稿， TS 也逐渐放弃了自己的模块系统转而支持 ESM 的规范。相关的讨论可以看下面的 PR
https://github.com/Microsoft/TypeScript/pull/2460

TS 把 CJS 模块作为一个 Namespace 导入，所以，为了解决上面提到的报错，需要这样导入 CJS 模块，以及任何没有 default 导出的模块：
`import * as React from 'react'`

这样子的代码，如果从 babel 迁移到 TS 就需要大幅的改动代码，不过 TS 也注意到了这个问题，添加了一个 compile option 支持 babel 的这种写法 `esModuleInterop`， PR 在下面
https://github.com/Microsoft/TypeScript/pull/19675
```
// tsconfig.json
{
    "esModuleInterop": true
}
```

## 最后

由于历史遗留问题， ESM 和 CJS及其他现存的模块系统之间的交互总是会或多或少遇到一些坑，这样的讨论也到处可见，所以团队中还是应该遵循统一的 模块导入导出 标准，能基于约定形成一套标准在之后需要改变迁移的时候可能也会更加方便一些。
