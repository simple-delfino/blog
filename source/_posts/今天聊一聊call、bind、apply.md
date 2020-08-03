---
title: 今天聊一聊call、bind、apply
id: '20200722001'
tags:
  - interview
date: 2020-07-22 21:29:41
tag: interview
category: web
---

在JS中，这三者都是用来改变函数的this对象的指向的，他们有什么样的区别呢。  
在说区别之前还是先总结一下三者的相似之处：

* 1、都是用来改变函数的this对象的指向的。
* 2、第一个参数都是this要指向的对象。
* 3、都可以利用后续参数传参。

**「那么他们的区别在哪里的，先看一个例子。」**

```
var xw = {
 name: '小王',
 gender: '男',
 age: 24,
 say: function () {
 alert(this.name + ' , ' + this.gender + ' ,今年' + this.age);
 },
};
var xh = {
 name: '小红',
 gender: '女',
 age: 18,
};
xw.say();
``` 

本身没什么好说的，显示的肯定是小王 ， 男 ， 今年24。那么如何用xw的say方法来显示xh的数据呢。

**「对于call可以这样：」**

`xw.say.call\(xh\);` 

**「对于apply可以这样：」**

`xw.say.apply\(xh\);` 

**「而对于bind来说需要这样：」**

`xw.say.bind\(xh\)\(\);` 

如果直接写xw.say.bind\(xh\)是不会有任何结果的，看到区别了吗？call和apply都是对函数的直接调用，而bind方法返回的仍然是一个函数，因此后面还需要\(\)来进行调用才可以。  
那么call和apply有什么区别呢？我们把例子稍微改写一下。

```
var xw = {
 name: '小王',
 gender: '男',
 age: 24,
 say: function (school, grade) {
 alert(
 this.name +
 ' , ' +
 this.gender +
 ' ,今年' +
 this.age +
 ' ,在' +
 school +
 '上' +
 grade
 );
 },
};
var xh = {
 name: '小红',
 gender: '女',
 age: 18,
};
``` 

可以看到say方法多了两个参数，我们通过call/apply的参数进行传参。

**「对于call来说是这样的」**

`xw.say.call\(xh,"实验小学","六年级"\);` 

**「而对于apply来说是这样的」**

`xw.say.apply\(xh,\["实验小学","六年级"\]\);` 

看到区别了吗，call后面的参数与say方法中是一一对应的，而apply的第二个参数是一个数组，数组中的元素是和say方法中一一对应的，这就是两者最大的区别。  
那么bind怎么传参呢？它可以像call那样传参。

`xw.say.bind\(xh,"实验小学","六年级"\)\(\);` 

但是由于bind返回的仍然是一个函数，所以我们还可以在调用的时候再进行传参。

`xw.say.bind\(xh\)\("实验小学","六年级"\);` 

## 那要怎么手动去实现这三个函数呢？

#### **「实现一个call函数」**

```
Function.prototype.myCall = function (context) {
 if (typeof this !== 'function') {
 throw new TypeError('Error');
 }
 if (context === null || context === undefined) {
 context = window; // 指定为 null 和 undefined 的 this 值会自动指向全局对象(浏览器中为window)
 } else {
 context = Object(context); // 值为原始值（数字，字符串，布尔值）的 this 会指向该原始值的实例对象
 }
 context.fn = this;
 //通过参数伪数组将context后面的参数取出来
 let args = [...arguments].slice(1);
 let result = context.fn(...args);
 //删除 fn
 delete context.fn;
 return result;
}; 
```
**「实现思路：」**

* 首先，判断调用`mycall`的是不是函数，如果不是，则直接抛出异常；
* 接着，判断是否传入了第一个参数`context`，也就是要指定的`this`值，如果没有传入，则默认为`window`全局对象；
* 然后，谁将来调用`mycall`，那么`this`就是谁，将其赋给`context.fn`;
* 然后，通过参数伪数组将`context`后面的参数取出来,并传给`context.fn`获得执行结果`result`；
* 最后，删除掉`context.fn`，并将`result`返回；

#### **「实现一个apply函数」**

```
Function.prototype.myApply = function (context) {
 if (typeof this !== 'function') {
 throw new TypeError('Error');
 }
 if (context === null || context === undefined) {
 context = window; // 指定为 null 和 undefined 的 this 值会自动指向全局对象(浏览器中为window)
 } else {
 context = Object(context); // 值为原始值（数字，字符串，布尔值）的 this 会指向该原始值的实例对象
 }
 context.fn = this;
 let result;
 //判断是否存在第二个参数
 //如果存在就将第二个参数也展开
 if (arguments[1]) {
 result = context.fn(...arguments[1]);
 } else {
 result = context.fn();
 }
 delete context.fn;
 return result;
};
``` 

**「实现思路：」**

* 首先，判断调用`myapply`的是不是函数，如果不是，则直接抛出异常；
* 接着，判断是否传入了第一个参数`context`，也就是要指定的`this`值，如果没有传入，则默认为`window`全局对象；
* 然后，谁将来调用`myapply`，那么`this`就是谁，将其赋给`context.fn`;
* 然后，判断是否传入了第二个参数，如果传入了则将其使用展开运算符`...`传给`context.fn`获得执行结果`result`，如果没有传入，则直接调用`context.fn`获得执行结果`result`；
* 最后，删除掉`context.fn`，并将`result`返回；

#### **「实现一个bind函数」**

```
Function.prototype.mybind = function (context) {
 if (typeof this !== 'function') {
 throw new TypeError('Error');
 }
 const _this = this;
 const args = [...arguments].slice(1);
 //返回一个函数
 return function F() {
 if (this instanceof F) {
 // this是否是F的实例 也就是返回的F是否通过new调用
 return new _this(...args, ...arguments);
 }
 return _this.apply(context, args.concat(...arguments));
 };
}; 
```
**「实现思路：」**

* 首先，判断调用`mybind`的是不是函数，如果不是，则直接抛出异常；
* 接着，谁将来调用`mybind`，那么`this`就是谁，将其赋给`_this`，缓存一下;
* 然后，通过参数伪数组将`context`后面的参数\(预先添加到绑定函数的参数\)取出来，记作`args`,
* 然后，返回一个函数，并判断如果使用`new`运算符构造绑定函数，则忽略传入的第一个参数`context`，并将预先添加到绑定函数的参数`args`和将来传入新函数的参数`arguments`分别通过展开运算符`...`依次传入给调用`mybind`的调用者，并将结果返回。
* 最后，如果不是使用`new`运算符构造绑定函数，则对调用者使用`apply`方法，将传入的第一个参数以及预先添加到绑定函数的参数`args`和将来传入新函数的参数`arguments`分别通过展开运算符`...`依次传入给调用`mybind`的调用者，并将结果返回；

这里只是一个简略版的，但是应付面试已经够了.......
