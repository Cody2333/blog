---
title: koa-swagger-decorator:基于注解根据API自动生成可视化swagger文档
tags: [
    "swagger",
    "nodejs",
    "javascript"
]
date: 2018-02-10
categories: [
    "nodejs"
]
---

### 背景介绍

在前后端分离的架构下，后端如果能够维护一份与代码实时同步更新，并且可读性良好的文档，可以极大的减少前后端沟通交流的成本，也可以提升前后端的开发效率。

Swagger是一种Rest API的 简单但强大的表示方式，标准的，语言无关，这种 表示方式不但人可读，而且机器可读。具体的定义可以参考 [Swagger Definition](https://swagger.io/specification/)

通过使用 Swagger UI 使得用 swagger 规范的接口文档可以被很好的可视化，并且直接在web网页中进行接口的调试，既方便了后端开发时的调试，也使得前端对接口有了更加清楚明确的理解，基本可以让我们抛弃 postman 等接口调试工具了。

### 简介

本文主要想要探讨的是**在快速迭代的中小型项目的敏捷开发**中，维护接口文档的最优方式，所以会更倾向于保证开发的效率，并且同时能够提供可读性良好，实时更新的接口文档。

本文将介绍几种基于 swagger 的维护项目接口文档的方式，并分析各种方式的优劣，并且提出了一种基于decorator来自动生成swagger json文档的方法。

项目地址:

[npm/koa-swagger-decorator](https://www.npmjs.com/package/koa-swagger-decorator)

[github/koa-swagger-decorator](https://github.com/Cody2333/koa-swagger-decorator)

#### 直接按照 swagger 规范 使用 yaml 的格式书写文档

这是最基本的使用 swagger 维护文档的方法。文档与后端代码和业务逻辑完全解耦。虽然使用 yaml 的格式来书写文档一定程度上保证了机器的可读性，也保证了人的可读性，但是当项目体积逐渐变大，维护成本依然很高，对于某个接口的细微修改，都需要重新去定位到接口文档中的对应位置进行修改，并且接口文档很有可能被忘记修改。

此外它还有一个很大的问题，就是重复的代码太多了，只有schema可以被复用，其他的都需要大量的复制粘贴才能完成。所以一开始这个方案就被排除了。

#### 使用 swagger-jsdoc 维护文档

[swagger-jsdoc](https://www.npmjs.com/package/swagger-jsdoc) 使得我们可以将swagger融入到基于JsDoc的注释当中去，这确实是一个对代码侵入性较小，同时能够大大方便对项目文档维护的方式。
示例如下：
```
/**
 * @swagger
 * /logger/apps:
 *   get:
 *     summary: 获取有日志记录的应用
 *     description: 获取有日志记录的应用
 *     tags:
 *       - Logger
 *     parameters:
 *       - name: appName
 *         in: query
 *         required: true
 *         description: 应用名称
 *         type: string
 *       - name: userId
 *         in: query
 *         required: true
 *         description: 用户id
 *         type: string
 *     responses:
 *       200:
 *         description: 成功获取
 */
router.get('/apps', async (req, res, next) => {
  try {
    const result = ['自定义字段'];

    return res.send({ result });
  } catch (err) {
    next(err);
  }
});
```
在注释中使用@swagger开头，然后就可以按照 swagger 的语法来进行接口的定义。这样子极大地方便了文档的书写和修改，在程序员对接口进行修改之后，也不至于忘记维护接口文档，同时可以更加方便的实时的调试自己的接口。

基于swagger ui 可以自动生成如下的接口文档界面

![image.png](http://upload-images.jianshu.io/upload_images/2563527-43d1f69e054e9da3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

并且可以直接在该界面中调试接口

![image.png](http://upload-images.jianshu.io/upload_images/2563527-595545001c750423.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

基于 swagger-jsdoc 的解决方案对于快递迭代的中小型项目来说已经是一个不错的选择了。我们基于这样的方案也开发的较长的一段时间，相对于原来的接口文档维护的方案可以说是体验大大的提升了。但是依然不是一个十分理想的解决方案。

##### 优点：
1. 文档不再与代码分离，每个接口的定义都分布在对于的接口路由处，方便修改维护，减轻文档维护的成本
2. 实际上文档与代码依然是解耦的，文档的定义不影响代码的逻辑，开发时依然只需要专注于业务逻辑。

##### 缺点：
1. 文档与代码解耦有其优势但也有其缺陷。接口文档理论上应该是与业务逻辑有一定的一致性的，比如当我们需要对一个接口的路由进行修改的时候，我们需要对于的修改swagger-jsdoc中的定义，这其实也是冗余的步骤。

2. 有很多重复的文档书写工作。比如response相关的参数，比如可能很多接口会有相似的query参数，但是我们依然每次都得进行一次复制粘贴来进行新的接口的swagger的定义。

3. 报错信息不清晰。对于一个需要接受的参数很多的接口，swagger文档会写的比较长，当在swgger文档中出现语法错误，缩进错误时，错误提示很不清楚，无法直接定位到出错点。

#### 一些想法

经过多 swagger-jsdoc 一段时间的时候，我们认为我们需要的这么一个工具：能提供类似swagger-jsdoc的文档书写方式，但是同时能够相比于在注释中书写的方式，提供与代码一定的耦合性。进一步减轻维护文档的负担，提升开发效率。

#### 基于注解的swagger 文档书写形式

借鉴 java 的 spring 框架中使用注解的形式来生成文档的形式，我们考虑在 node 中 利用 decorator 的特性，实现类似的功能，但是并不对代码产生更多的侵入性的改造，保持后端API服务的简洁性。

koa-swagger-decorator库所提供的功能：

1. 提供路由定义的功能
2. 提供参数验证的功能
3. 基于API生成swagger文档

直接来看最终实现的形式

```
// 定义可复用的schema
const userSchema = {
  type: 'object',
  properties: {
    name: { type: 'string' },
    gender: { type: 'string' }
  }
};


  @request('post', '/users')
  @tag('User')
  @summary('创建用户')
  @body([{
    name: 'data',
    description: '用户信息',
    schema: userSchema,
  }])
  static async postUser(ctx) {
    const body = ctx.request.body;
    ...
    ctx.body = { result: body };
  }
```
查看完整示例请看：[Github/koa-swagger-decorator](https://github.com/Cody2333/koa-swagger-decorator)
项目需要koa2+ koa-router 的依赖

通过这样的方式，我们就可以抛弃使用 yaml 的语法书写文档了，可以直接用 js 的对象来维护文档，并且自然的支持各个对象的复用了。

这个方式对于原有rest api实现方式而言唯一的一个改动就是：由于注解只能在Class中使用，所以我们只能将处理函数放在Class中，并且使用静态方法的方式来使用，这是相对之前的开发模式最大的改变，其他的体验依然基本保持一致。

## 最后
该项目现在已经支持参数的验证和自动化swagger文档的构建。未来将提供OpenApi 3.0版本的支持

