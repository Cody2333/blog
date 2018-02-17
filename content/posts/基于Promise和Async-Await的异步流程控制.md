---
title: 基于Promise和Async-Await的异步流程控制
tags: [
    "javascript",
    "nodejs"
]
date: 2017-04-18
categories: [
    "nodejs"
]
---

## 简介

很久以前，在Promise出现之前，编写js的异步代码是一件十分蛋疼的事情，一旦异步流程一复杂，回调地狱就会等着你，所以基于node.js的服务端编码开发的体验非常差，项目的可维护性和可读性也非常差，这也是最初node.js最为人所诟病的地方。

在现代的js项目中，回调函数已经基本被Promise取代，而async/await的出现更是解决的promise在使用中会产生大量的胶水代码，以及不同Promise之间共享数据困难的问题，这又大大提升的编写异步流程的开发体验，但是async/await本身是基于promise的，而在很多情况下，async/await并不能完全取代promise的位置，理解promise是编写高质量的异步代码的基础。下面我们就来探讨一下在js中的异步流程控制的问题。如何在异步代码中保证异步流程的执行效率以及可读性和可维护性。以下为一个例子：

## 如何制作一个披萨？

1. 制作饼皮（dough）

2. 制作酱汁（sauce）

3. 品尝一下酱汁，根据酱汁的口味决定放哪种芝士（cheese）

## 第一版实现
```
async function makePizza(sauceType = 'red') {
  
  let dough  = await makeDough();
  let sauce  = await makeSauce(sauceType);
  let cheese = await grateCheese(sauce.determineCheese());
  
  dough.add(sauce);
  dough.add(cheese);
  
  return dough;

}
```
基于async/await，看起来逻辑很清晰，但是它最大的问题是这不是一个最优的异步流程控制。

## 第二版实现
```
// 原来的流程
|-------- dough --------> |-------- sauce --------> |-- cheese -->

// 改进的流程
|-------- dough -------->
|-------- sauce --------> |-- cheese -->

async function makePizza(sauceType = 'red') {
  
  let [ dough, sauce ] =
    await Promise.all([ makeDough(), makeSauce(sauceType) ]);
  let cheese = await grateCheese(sauce.determineCheese());
  
  dough.add(sauce);
  dough.add(cheese);
  
  return dough;
}
```
效率比第一版就高了很多。使用promise.all并行执行异步任务，流程也比较清晰。

## 第三版实现

由于制作dough的时间是比较长的，我们的流程其实还有改进的空间，如下。
```
// 上一步改进的流程
|-------- dough -------->
|--- sauce --->           |-- cheese -->

// 进一步改进的流程
|--------- dough --------->
|---- sauce ----> |-- cheese -->

function makePizza(sauceType = 'red') {
  let doughPromise  = makeDough();
  let saucePromise  = makeSauce(sauceType);
  let cheesePromise = saucePromise.then(sauce => {
    return grateCheese(sauce.determineCheese());
  });
  
  return Promise.all([ doughPromise, saucePromise, cheesePromise ])
    .then(([ dough, sauce, cheese ]) => {
      
      dough.add(sauce);
      dough.add(cheese);
      
      return dough;
    });
}
```
现在的异步流程应该是最优的了，但是问题来了，这样的流程用async/await表示的话就有点麻烦了，我们这里先用promise实现了这个最优流程，可以看到promise的弊端很明显，充满了胶水代码，我们再也不能像我们的第一版实现一样，一眼就能看出我们这个函数的异步流程是怎么样的了，可读性下降了。

## 第四版实现
```
// 最优流程
|--------- dough --------->
|---- sauce ----> |-- cheese -->

async function makePizza(sauceType = 'red') {
  
  let doughPromise = makeDough();
  let saucePromise = makeSauce(sauceType);
  
  let sauce  = await saucePromise;
  let cheese = await grateCheese(sauce.determineCheese());
  let dough  = await doughPromise;
  
  dough.add(sauce);
  dough.add(cheese);
  
  return dough;
}
```
这一版实现，代码看起来整洁了很多，但是它的异步流程更加不清晰了，通过提前创建promise然后在之后再进行await，我们得到了整洁的代码，但是却使得流程更加不清晰，隐式的执行promise然后await，让await的语义变得很不明了。

## 第五版实现
```
async function makePizza(sauceType = 'red') {
  
  let prepareDough  = memoize(async () => makeDough());
  let prepareSauce  = memoize(async () => makeSauce(sauceType));
  let prepareCheese = memoize(async () => {
    return grateCheese((await prepareSauce()).determineCheese());
  });
  
  let [ dough, sauce, cheese ] = 
    await Promise.all([
      prepareDough(), prepareSauce(), prepareCheese()
    ]);
    
  dough.add(sauce);
  dough.add(cheese);
  
  return dough;
}
```
这个版本相比于上一个版本的区别主要是现在我们不再提前创建promise，而保存为一个生成函数，并且使用memoize缓存结果，从而使得promise只需要resolve一次就可以缓存结果。这样子的异步流程相比于上一个版本就清楚了很多，而代码依然可以保持很好的可读性。

## 总结

感觉Promise.all和await/async的结合依然不是很优雅，总感觉有点难受，在未来，js的异步流程控制肯定还会有更优雅的解决方案。
