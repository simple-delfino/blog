---
title: 今天谈一下null和undefined的区别？
id: '20200723001'
tags:
  - interview
date: 2020-07-23 23:26:52
tag: interview
category: web
---

> 
> 
> 今天弄得有点晚了，就写个简单的吧,null是一个表示"无"的对象，转为数值时为0；undefined是一个表示"无"的原始值，转为数值时为NaN。
> 
> 

**null是一个表示"无"的对象，转为数值时为0；undefined是一个表示"无"的原始值，转为数值时为NaN。**

undefined：
----------

*   （1）变量被声明了，但没有赋值时，就等于undefined。
    
*   （2) 调用函数时，应该提供的参数没有提供，该参数等于undefined。
    
*   （3）对象没有赋值的属性，该属性的值为undefined。
    
*   （4）函数没有返回值时，默认返回undefined。
    

null：
-----

*   （1） 作为函数的参数，表示该函数的参数不是对象。
    
*   （2） 作为对象原型链的终点。
    
