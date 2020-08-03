---
title: 判断js数据类型的四种方法，以及各自的优缺点
id: '20200721002'
tags: 
  - interview
date: 2020-07-21 23:45:18
tag: interview
category: web
---

> 首先我们简单的说一下js中的几种数据类型，数据类型分为基本类型和引用类型：

**「基本类型：」**

*   String
    
*   Number
    
*   Boolean
    
*   Null
    
*   Undefined
    

**「引用类型：」**

*   Object
    
*   Array
    
*   Date
    
*   Function
    
*   Error
    
*   RegExp
    
*   Math
    
*   Number
    
*   String
    
*   Boolean
    
*   Globle。
    

然后判断数据类型的方法一般可以通过：`typeof、instanceof、constructor、toString`四种常用方法

1、typeof
========

只能检测出 `undefined，string， number， boolean， symbol， function，object`

### **「返回值」**

*   基本类型，除 null 以外，均可以返回正确的结果。
    
*   引用类型，除 function 以外，一律返回 object 类型。
    
*   null ，返回 object 类型。
    
*   function 返回 function 类型。
    

### **「原理」**

*   js 在底层存储变量的时候，会在变量的机器码的低位 1-3 位存储其类型信息
    
*   000：对象
    
*   010：浮点数
    
*   100：字符串
    
*   110：布尔
    
*   1：整数
    
*   null：所有机器码均为 0
    
*   undefined：用 −230 整数来表示
    
*   **「typeof 在判断 null 的时候由于 null 的所有机器码均为 0，因此直接被当做了对象来看待。」**
    

2、instanceof


判断对象和构造函数在原型链上是否有关系，如果有关系，返回真，否则返回假


```
function Aaa() {}
var a1 = new Aaa(); 
//alert( a1 instanceof Aaa);  
//true判断a1和Aaa是否在同一个原型链上，是的话返回真，否则返回假 
var arr = []; 
alert( arr instanceof Aaa);//false
``` 

我们来看一下

```
var str = 'hello';
alert(str instanceof String); //false
var bool = true;
alert(bool instanceof Boolean); //false
var num = 123;
alert(num instanceof Number); //false
var nul = null;
alert(nul instanceof Object); //false
var und = undefined;
alert(und instanceof Object); //false
var oDate = new Date();
alert(oDate instanceof Date); //true
var json = {};
alert(json instanceof Object); //true
var arr = [];
alert(arr instanceof Array); //true
var reg = /a/;
alert(reg instanceof RegExp); //true
var fun = function () {};
alert(fun instanceof Function); //true
var error = new Error();
alert(error instanceof Error); //true
``` 

从上面的运行结果我们可以看到，基本数据类型是没有检测出他们的类型，但是我们使用下面的方式创建num、str、boolean，是可以检测出类型的：

```
var num = new Number(123);
var str = new String('abcdef');
var boolean = new Boolean(true);
``` 

3、constructor：查看对象对应的构造函数


`constructor` 在其对应对象的原型下面，是自动生成的。当我们写一个构造函数的时候，程序会自动添加：`构造函数名.prototype.constructor = 构造函数名`

```
function Aaa(){}
//Aaa.prototype.constructor = Aaa;   
//每一个函数都会有的，都是自动生成的 
//Aaa.prototype.constructor = Aaa;
``` 

判断数据类型的方法

```
var str = 'hello';
alert(str.constructor == String); //true
var bool = true;
alert(bool.constructor == Boolean); //true
var num = 123;
alert(num.constructor == Number); //true
// var nul = null;
// alert(nul.constructor == Object);//报错
//var und = undefined;
//alert(und.constructor == Object);//报错
var oDate = new Date();
alert(oDate.constructor == Date); //true
var json = {};
alert(json.constructor == Object); //true
var arr = [];
alert(arr.constructor == Array); //true
var reg = /a/;
alert(reg.constructor == RegExp); //true
var fun = function () {};
alert(fun.constructor == Function); //true
var error = new Error();
alert(error.constructor == Error); //true
``` 

从上面的测试中我们可以看到，`undefined`和`null`是不能够判断出类型的，并且会报错。因为`null`和`undefined`是无效的对象，因此是不会有`constructor`存在的同时我们也需要注意到的是：使用`constructor`是不保险的，因为`constructor`属性是可以被修改的，会导致检测出的结果不正确

```
function Aaa() {}
Aaa.prototype.constructor = Aaa;
//程序可以自动添加，当我们写个构造函数的时候，程序会自动添加这句代码
function BBB() {}
Aaa.prototype.constructor = BBB;
//此时我们就修改了Aaa构造函数的指向问题
alert(Aaa.construtor == Aaa); //false
``` 

**「可以看出，constructor并没有正确检测出正确的构造函数」**

4、Object.prototype.toString

**「可以说不管是什么类型，它都可以立即判断出-----这才是最强大的」**

`toString`是`Object`原型对象上的一个方法，该方法默认返回其调用者的具体类型，更严格的讲，是`toString`运行时`this`指向的对象类型, 返回的类型

格式为\[object xxx\],xxx是具体的数据类型，其中包括： `String,Number,Boolean,Undefined,Null,Function,Date,Array,RegExp,Error,HTMLDocument,...` 基本上所有对象的类型都可以通过这个方法获取到。

```
var str = 'hello';
console.log(Object.prototype.toString.call(str)); //[object String]
var bool = true;
console.log(Object.prototype.toString.call(bool)); //[object Boolean]
var num = 123;
console.log(Object.prototype.toString.call(num)); //[object Number]
var nul = null;
console.log(Object.prototype.toString.call(nul)); //[object Null]
var und = undefined;
console.log(Object.prototype.toString.call(und)); //[object Undefined]
var oDate = new Date();
console.log(Object.prototype.toString.call(oDate)); //[object Date]
var json = {};
console.log(Object.prototype.toString.call(json)); //[object Object]
var arr = [];
console.log(Object.prototype.toString.call(arr)); //[object Array]
var reg = /a/;
console.log(Object.prototype.toString.call(reg)); //[object RegExp]
var fun = function () {};
console.log(Object.prototype.toString.call(fun)); //[object Function]
var error = new Error();
console.log(Object.prototype.toString.call(error)); //[object Error]
``` 

从这个结果也可以看出，不管是什么类型的，`Object.prototype.toString.call\(\);`都可以判断出其具体的类型。  
接下来我们分析一下四种方法各自的优缺点

不同类型的优缺点

`typeof`

`instanceof`

`constructor`

`Object.prototype.toString.call`

优点

使用简单

能检测出引用类型

基本能检测所有的类型（除了`null`和`undefined`）

检测出所有的类型

缺点

只能检测出基本类型（除`null`）

不能检测出基本类型，且不能跨`iframe`

`constructor`易被修改，也不能跨`iframe`

IE6下，undefined和null均为Object

从上表中我们看到了，`instanceof`和`constructor`不能跨iframe,上面没有细说，所以下面我们直接上例子喽

例：跨页面判断是否是数组
------------

```
window.onload = function () {
 var oF = document.createElement('iframe');
 document.body.appendChild(oF);
 var ifArray = window.frames[0].Array;
 var arr = new ifArray();
 //alert( arr.constructor == Array );  //false
 //alert( arr instanceof Array );  //false
 alert(Object.prototype.toString.call(arr) == '[object Array]'); //true
};
``` 

从结果中可以看出，`constructor`和`instanceof`都没有正确的判断出类型，只有`object.prototype.toString.call\(\)；`正确判断出了

其实面试官也经常喜欢让说一种最简单的判断是数组的方法，记住喽是`object.prototype.toString.call()`哦！
