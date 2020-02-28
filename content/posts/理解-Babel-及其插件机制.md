---
title: 理解 Babel 及其插件机制
tags: [
    "tech","js"
]
date: 2018-08-03
---

## 简介

Babel 是 JavaScript 编译器，更确切地说是源码到源码的编译器，通常也叫做“转换编译器（transpiler）”。 意思是说你为 Babel 提供一些 JavaScript 代码，Babel 更改这些代码，然后返回给你新生成的代码。

## Babel 处理的三个步骤

- 解析 `parse`
- 转换 `transform`
- 生成 `generate`

## 相关包和 API

- @babel/parser (babylon)
- babel-core
- babel-types
- babel-traverse
- babel-generator

### 解析

该步骤接收代码并输出 AST。 这个步骤分为两个阶段：[**词法分析(Lexical Analysis)**](https://en.wikipedia.org/wiki/Lexical_analysis) 和  [**语法分析(Syntactic Analysis)**](https://en.wikipedia.org/wiki/Parsing)。

```javascript
const parser = require('@babel/parser')
const code = 'function square(n) {return n * n;}';
const node = parser.parse(code) // 解析代码返回AST树
```

要方便的查看代码的 AST 结构，可以访问 http://astexplorer.net/ 进行测试。Babel 使用一个基于 [ESTree](https://github.com/estree/estree) 并修改过的 AST，它的内核说明文档可以在 [这里](https://github/com/babel/babel/blob/master/doc/ast/spec.md) 找到。

```
function square(n) {
  return n * n;
}
```

该函数的 AST 结构如下 （简化了一些属性）

```
{
  type: "FunctionDeclaration",
  id: {
    type: "Identifier",
    name: "square"
  },
  params: [{
    type: "Identifier",
    name: "n"
  }],
  body: {
    type: "BlockStatement",
    body: [{
      type: "ReturnStatement",
      argument: {
        type: "BinaryExpression",
        operator: "*",
        left: {
          type: "Identifier",
          name: "n"
        },
        right: {
          type: "Identifier",
          name: "n"
        }
      }
    }]
  }
}
```

每一个节点都有如下所示的接口（Interface）：

```
interface Node {
  type: string;
}
```

AST 就是由多层嵌套的 Node 结合而成。

## 转换

[转换](https://en.wikipedia.org/wiki/Program_transformation)步骤接收 AST 并对其进行遍历，在此过程中对节点进行添加、更新及移除等操作。 这是 Babel 或是其他编译器中最复杂的过程 同时也是插件将要介入工作的部分。

`babel-core` 向外暴露出 `babel.transform` 接口

```
const result = babel.transform(code, {
    plugins: [
        myPluginName
    ]
})
```

## 生成

[代码生成](https://en.wikipedia.org/wiki/Code_generation_(compiler))步骤把最终（经过一系列转换之后）的 AST 转换成字符串形式的代码，同时还会创建[源码映射（source maps）](http://www.html5rocks.com/en/tutorials/developertools/sourcemaps/)。.
代码生成其实很简单：深度优先遍历整个 AST，然后构建可以表示转换后代码的字符串。利用 `babel-generator` 将 AST 输出为转码后的代码字符串

## babel-traverse

Babel Traverse（遍历）模块维护了整棵树的状态，并且负责替换、移除和添加节点。

我们可以和 Babylon 一起使用来遍历和更新节点：

```javascript
import * as babylon from "@babel/parser";
import traverse from "babel-traverse";

const code = `function square(n) {
  return n * n;
}`;

const ast = babylon.parse(code);

traverse(ast, {
  enter(path) {
    if (
      path.node.type === "Identifier" &&
      path.node.name === "n"
    ) {
      path.node.name = "x";
    }
  }
});
```

## babel-types

Babel Types模块是一个用于 AST 节点的 Lodash 式工具库，它包含了构造、验证以及变换 AST 节点的方法。 该工具库包含考虑周到的工具方法，对编写处理AST逻辑非常有用。

简单的说就是封装了一系列方法，方便操作变换 AST 的节点。

```
import traverse from "babel-traverse";
import * as t from "babel-types";

traverse(ast, {
  enter(path) {
    if (t.isIdentifier(path.node, { name: "n" })) {
      path.node.name = "x";
    }
  }
});
```

## 写一个简单的 babel 插件

### 使用 Visitor 模式遍历访问 Node 节点

利用深度遍历每个节点的时候，包含两个时机：进入节点enter 和离开节点 exit，类似 koa 的一个洋葱模型。我们也应该在这两个时机的时候去对 AST 树做对应的转换。以下是Visitor 访问的例子。

```
const MyVisitor = {
  // 简化 enter 的形式
  Identifier() {
    console.log("Called!");
  }
  // 完整形式
  //Identifier() {
  //  enter() {...}
  //  exit() {...}
  //}
};

// 对于这个函数
function square(n) {
  return n * n;
}

// 遍历包含了四个 Identifier （1个 square 和 3个 n）
path.traverse(MyVisitor);
// 输出四次结果
Called!
Called!
Called!
Called!
```
### Path

path 的概念是为了简化对 AST 树的操作。Path 是表示两个节点之间连接的对象。
在某种意义上，路径是一个节点在树中的位置以及关于该节点各种信息的响应式 Reactive 表示。 当你调用一个修改树的方法后，路径信息也会被更新。 Babel 帮你管理这一切，从而使得节点操作简单，尽可能做到无状态。

### 首先编写插件
这个插件的作用就是简单的**把所有变量名为 foo 的变量改为 bar**，具体实现如下：
```
module.exports = function myPlugin(babel) {
  return {
    visitor: {
      Identifier(path) {
        if (path.node.name === 'foo') {
          path.node.name = 'bar';
        }
      }
    }
  };
};
```

### 调用插件

```
const babel = require('babel-core');
const myPlugin = require('./myPlugin');
const example = 'const foo = 1; console.log(foo);'
const res = babel.transform(example, {plugins: [myPlugin]});
// res.code = "const bar = 1; console.log(bar);"
```

## 总结

简单介绍了 babel 转换代码的基本内容，实现了一个简单的插件。在具体开发插件的过程中，最重要的还是对 AST 的转换，所以需要对 AST 的操作和相关 API 有更加具体的理解。详细信息可以看[babel-notebook](https://github.com/jamiebuilds/babel-handbook)的介绍，本文也有很大一部分参考了 babel-notebook。
