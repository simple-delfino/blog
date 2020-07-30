---
title: 手写async await的最简实现（20行搞定）面试必考！
tag: interview
category: interview
date: 2020-07-30 20:21:00
id: 20200730002
---


[转自 图雀社区](https://mp.weixin.qq.com/s/eASescI3ZO0InccGQgQSjg)

> 本文由掘金作者 晨曦时梦见兮\[1\] 写作而成，此次转载已得到原作者授权，点击阅读原文查看作者掘金链接，感谢作者的优质输出，让我们的技术世界变得更加美好😆

## 前言

如果让你手写 async 函数的实现，你是不是会觉得很复杂？这篇文章带你用 20 行搞定它的核心。经常有人说 async 函数是 generator 函数的语法糖，那么到底是怎么样一个糖呢？让我们来一层层的剥开它的糖衣。有的同学想说，既然用了 generator 函数何必还要实现 async 呢？这篇文章的目的就是带大家理解清楚 async 和 generator 之间到底是如何相互协作，管理异步的。

## 示例

```
const getData = () =>newPromise(resolve => setTimeout(() => resolve("data"), 1000))
 asyncfunction test() {
 const data = await getData()
 console.log('data: ', data);
 const data2 = await getData()
 console.log('data2: ', data2);
 return'success'
}
 // 这样的一个函数 应该再1秒后打印data 再过一秒打印data2 最后打印success
test().then(res =>console.log(res))
```

## 思路

对于这个简单的案例来说，如果我们把它用 generator 函数表达，会是怎么样的呢？

```
function* testG() {
 // await被编译成了yield
 const data = yield getData()
 console.log('data: ', data);
 const data2 = yield getData()
 console.log('data2: ', data2);
 return'success'
}
``` 

我们知道，generator 函数是不会自动执行的，每一次调用它的 next 方法，会停留在下一个 yield 的位置。利用这个特性，我们只要编写一个自动执行的函数，就可以让这个 generator 函数完全实现 async 函数的功能。

```
const getData = () =>newPromise(resolve => setTimeout(() => resolve("data"), 1000))
 var test = asyncToGenerator(
 function* testG() {
 // await被编译成了yield
 const data = yield getData()
 console.log('data: ', data);
 const data2 = yield getData()
 console.log('data2: ', data2);
 return'success'
 }
)
 test().then(res =>console.log(res))
 ```

那么大体上的思路已经确定了，`asyncToGenerator`接受一个`generator`函数，返回一个`promise`，关键就在于，里面用`yield`来划分的异步流程，应该如何自动执行。

## 如果是手动执行

在编写这个函数之前，我们先模拟手动去调用这个`generator`函数去一步步的把流程走完，有助于后面的思考。

```
function* testG() {
 // await被编译成了yield
 const data = yield getData()
 console.log('data: ', data);
 const data2 = yield getData()
 console.log('data2: ', data2);
 return'success'
}
```

我们先调用`testG`生成一个迭代器

```
// 返回了一个迭代器
var gen = testG()
```

然后开始执行第一次`next`

```
// 第一次调用next 停留在第一个yield的位置
// 返回的promise里 包含了data需要的数据
var dataPromise = gen.next()
```

这里返回了一个`promise`，就是第一次`getData()`所返回的`promise`，注意

```
const data = yield getData()
``` 

这段代码要切割成左右两部分来看，第一次调用`next`，其实只是停留在了`yield getData()`这里，`data`的值并没有被确定。那么什么时候 data 的值会被确定呢？**「下一次调用 next 的时候，传的参数会被作为上一个 yield 前面接受的值」**也就是说，我们再次调用`gen.next('这个参数才会被赋给data变量')`的时候`data`的值才会被确定为`'这个参数才会被赋给data变量'`

```
gen.next('这个参数才会被赋给data变量')
 // 然后这里的data才有值
const data = yield getData()
 // 然后打印出data
console.log('data: ', data);
 // 然后继续走到下一个yield
const data2 = yield getData() 
```
然后往下执行，直到遇到下一个`yield`，继续这样的流程...这是 generator 函数设计的一个比较难理解的点，但是为了实现我们的目标，还是得去学习它\~借助这个特性，如果我们这样去控制 yield 的流程，是不是就能实现异步串行了？

```
function* testG() {
 // await被编译成了yield
 const data = yield getData()
 console.log('data: ', data);
 const data2 = yield getData()
 console.log('data2: ', data2);
 return'success'
}
 var gen = testG()
 var dataPromise = gen.next()
 dataPromise.then((value1) => {
 // data1的value被拿到了 继续调用next并且传递给data
 var data2Promise = gen.next(value1)
 // console.log('data: ', data);
 // 此时就会打印出data
 data2Promise.then((value2) => {
 // data2的value拿到了 继续调用next并且传递value2
 gen.next(value2)
 // console.log('data2: ', data2);
 // 此时就会打印出data2
 })
})
```

这样的一个看着像`callback hell`的调用，就可以让我们的 generator 函数把异步安排的明明白白。

## 实现

有了这样的思路，实现这个高阶函数就变得很简单了。先整体看一下结构，有个印象，然后我们逐行注释讲解。

```
function asyncToGenerator(generatorFunc) {
 returnfunction() {
 const gen = generatorFunc.apply(this, arguments)
 returnnewPromise((resolve, reject) => {
 function step(key, arg) {
 let generatorResult
 try {
 generatorResult = gen[key](arg)
 } catch (error) {
 return reject(error)
 }
 const { value, done } = generatorResult
 if (done) {
 return resolve(value)
 } else {
 returnPromise.resolve(value).then(val => step('next', val), err => step('throw', err))
 }
 }
 step("next")
 })
 }
}
```

不多不少，22 行。接下来逐行讲解。

```
function asyncToGenerator(generatorFunc) {
 // 返回的是一个新的函数
 returnfunction() {
 // 先调用generator函数 生成迭代器
 // 对应 var gen = testG()
 const gen = generatorFunc.apply(this, arguments)
 // 返回一个promise 因为外部是用.then的方式 或者await的方式去使用这个函数的返回值的
 // var test = asyncToGenerator(testG)
 // test().then(res => console.log(res))
 returnnewPromise((resolve, reject) => {
 // 内部定义一个step函数 用来一步一步的跨过yield的阻碍
 // key有next和throw两种取值，分别对应了gen的next和throw方法
 // arg参数则是用来把promise resolve出来的值交给下一个yield
 function step(key, arg) {
 let generatorResult
 // 这个方法需要包裹在try catch中
 // 如果报错了 就把promise给reject掉 外部通过.catch可以获取到错误
 try {
 generatorResult = gen[key](arg)
 } catch (error) {
 return reject(error)
 }
 // gen.next() 得到的结果是一个 { value, done } 的结构
 const { value, done } = generatorResult
 if (done) {
 // 如果已经完成了 就直接resolve这个promise
 // 这个done是在最后一次调用next后才会为true
 // 以本文的例子来说 此时的结果是 { done: true, value: 'success' }
 // 这个value也就是generator函数最后的返回值
 return resolve(value)
 } else {
 // 除了最后结束的时候外，每次调用gen.next()
 // 其实是返回 { value: Promise, done: false } 的结构，
 // 这里要注意的是Promise.resolve可以接受一个promise为参数
 // 并且这个promise参数被resolve的时候，这个then才会被调用
 returnPromise.resolve(
 // 这个value对应的是yield后面的promise
 value
 ).then(
 // value这个promise被resove的时候，就会执行next
 // 并且只要done不是true的时候 就会递归的往下解开promise
 // 对应gen.next().value.then(value => {
 //    gen.next(value).value.then(value2 => {
 //       gen.next()
 //
 //      // 此时done为true了 整个promise被resolve了
 //      // 最外部的test().then(res => console.log(res))的then就开始执行了
 //    })
 // })
 function onResolve(val) {
 step("next", val)
 },
 // 如果promise被reject了 就再次进入step函数
 // 不同的是，这次的try catch中调用的是gen.throw(err)
 // 那么自然就被catch到 然后把promise给reject掉啦
 function onReject(err) {
 step("throw", err)
 },
 )
 }
 }
 step("next")
 })
 }
}
```

## 源码地址

这个 js 文件的代码可以直接放进浏览器里运行，欢迎调戏。  
github.com/sl1673495/f…\[2\]

## 总结

本文用最简单的方式实现了 asyncToGenerator 这个函数，这是 babel 编译 async 函数的核心，当然在 babel 中，generator 函数也被编译成了一个很原始的形式，本文我们直接以 generator 替代。这也是实现 promise 串行的一个很棒的模式，如果本篇文章对你有帮助，点个赞就好啦。