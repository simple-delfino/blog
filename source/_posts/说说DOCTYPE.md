---
title: 说说DOCTYPE
id: '20200702001'
tags:
  - interview
date: 2020-07-20 23:33:36
tag: interview
category: interview
---

> DOCTYPE不是 HTML 标签；它是指示 web 浏览器关于页面使用哪个 HTML 版本进行编写的指令，用于声明文档类型和DTD（Document Type Definition）规范，确保不同浏览器以相同的方式解析文档，以及执行相同的渲染模式：

`<!DOCTYPE html
PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN" 
"http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">
 /*在上面的声明中，声明了文档的根元素是 html，它在公共标识符被定义为 "-//W3C//DTD XHTML 1.0 Strict//EN" 的 DTD 中进行了定义。浏览器将明白如何寻找匹配此公共标识符的 DTD。如果找不到，浏览器将使用公共标识符后面的 URL 作为寻找 DTD 的位置。*/` 

**主要分为三种：**

*   1、HTML5
    
*   2、HTML4.01
    
*   3、XHTML1.0
    

HTML5
=====

`<!DOCTYPE html\>` 

这个不用多说，告诉浏览器用H5的方式渲染，与HTML4.0.1不同，他不是基于 SGML，不需要使用DTD；

HTML4.0.1
=========

HTML4.0.1中的DTD可分为三种：`Strict`、`Transitional` 以及 `Frameset`。具体如下

### 1、HTML Strict DTD ---严格型

`//写法：
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN" "
http://www.w3.org/TR/html4/strict.dtd">
 /**
特点：包含所有的HTML元素和属性，但不包含已废弃的，如（font，center等）也不包括框架相关的元素（如frame）
*/` 

### 2、HTML Transitional DTD---过渡型

`//写法
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "
http://www.w3.org/TR/html4/loose.dtd">
//仅不包含框架相关的元素` 

### 3、Frameset DTD---框架集

`//写法
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Frameset//EN" "
http://www.w3.org/TR/html4/frameset.dtd">
/*包含所有的HTML元素和属性
除 frameset 元素取代了 body 元素之外，Frameset DTD 等同于 Transitional DTD：*/` 

XHTML
=====

XHTML 1.0 ,与HTML4.0.1一样，同样规定了三种 XML 文档类型：`Strict`、`Transitional` 以及 `Frameset`。写法如下,不赘述

`//Strict
<!DOCTYPE html
PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN" 
"http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">
//Transitional
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
//Frameset
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Frameset//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-frameset.dtd">` 

拓展：XHTML与HTML的区别：
=================

*   1、XHTML文档需要良好的文档结构，元素要合理嵌套，错误的例子<p><span>hello world</p></span>
    
*   2、XHTML文档区分大小写，并且元素属性和名称要小写
    
*   3、XHTML必须要有结束标签，就算是空元素，也要有/>结尾
    
*   4、XHTML可运用各种XML应用，如MathMl，SVG
    
*   5、注释标签<!-- -->将被忽略
    
*   6、XHTML中CDATA的内容可被执行，作用是防止解析到非法字符串就中断
    
*   7、XHTML中，不推荐a、applet、form、img、map、iframe、frame、map等拥有name属性，但加上也不会报错
    
*   8、HTML用脚本取到的HTML标签名和属性会以大写形式返回
    
*   9、XHTML元素的属性必须用引号包裹，并且禁止简化
    
*   10、XHTML中特殊字符必须替换为实体引用，如"<"替换为"&lt;"
    
