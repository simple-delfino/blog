---
title: 'Webpack常用插件总结'
id: '20200718001'
tags:
  - webpack
date: 2020-07-18 11:11:28
tag: webpack
category: other
---

刚完成一个基于React+Webpack的项目，渐渐地熟悉了Webpack的一些配置，开发过程中用到了不少Webpack插件，现在把觉得有用的Webpack插件总结一下当巩固学习啦，主要涉及热模块替换\[Hot Module Replacement\]、ProvidePlugin、CommonsChunkPlugin、UglifyJsPlugin 、ExtractTextWebpackPlugin、DefinePlugin、DllPlugin等。

## 1、热模块替换Hot Module Replacement

热模块替换（HMR）是webpack提供的最有用的特性之一，热模块替换可以让模块在没有页面刷新的情况下实时更新代码改动结果；

#### 安装webpack-dev-server

webpack-dev-server 是一个小型的Node.js Express服务器，它通过使用webpack-dev-middleware来为webpack打包的资源文件提供服务。可以认为webpack-dev-server就是一个拥有实时重载能力的静态资源服务器（建议只在开发环境使用）  
通过npm安装：

```
npm install webpack-dev-server --save-dev
```
运行
```

webpack-dev-server \--open

```
/*
 热更新授权
一个简单webpackHMR实例，这里感觉官方代码比较简单明了，直接复制过来啦
*/
const path = require('path');
const webpack = require('webpack');

module.exports = {
  entry: './index.js',

  plugins: [
    new webpack.HotModuleReplacementPlugin() // Enable HMR
  ],

  output: {
    filename: 'main.js',
    path: path.resolve(__dirname, 'dist'),
    publicPath: '/'
  },

  devServer: {
    hot: true, // Tell the dev-server we're using HMR
    contentBase: resolve(__dirname, 'dist'),
    publicPath: '/'
  }
};
```

上述代码配置中需要注意两个地方：  
（1）plugins：添加webpack.HotModuleReplacementPlugin模块热更新插件  
（2）devServer配置hot:true 开启模块热更新配置，更多配置详见devServer  
同时还需注意的几个devServer配置属性

```
inline: true|false
```

inline属性用于切换webpack-der-server编译并刷新浏览器的两种不同模式：  
（1）第一种也是默认的inline模式，这种模式是将脚本插入打包资源文件中复制热更新，并在浏览器控制台中输出过程信息，访问格式访问格式是http://:/；  
（2）iframe 模式：使用iframe加载页面，访问格式http://:/webpack-dev-server/  
可以通过配置

```
inline: false//启用iframe
```

![在这里插入图片描述](https://upload-images.jianshu.io/upload_images/280923-e36e18e6ebe08e87.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)  
官方还是推荐使用inline模式


在你的代码中插入热替换代码  
index.js 在入口文件结尾处插入

```
if (module.hot) {
    module.hot.accept();
}
```

只有被 "accept"的代码模块才会被热更新，所以你需要在父节点或者父节点的父节点… module.hot.accept 调用模块。如以上代码所示，在入口文件加入 module.hot.accept之后在入口文件引用的任何子模块更新都会触发热更新。模块更新完成，浏览器会输出类型以下信息

![在这里插入图片描述](https://upload-images.jianshu.io/upload_images/280923-9ab2ffb00c080f14.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)  
也可以像官方提供的实例一样accept特定的模块，如下实例，当’./library’有任何更新是都会触发热模块更新  
import Lib from ‘./library’;

```
if (module.hot) {
  module.hot.accept('./library', function() {
    console.log('Accepting the updated library module!');
    Library.log();
  })
}
```

###### 更多Hot Module Replacement资料参考

[Hot Module Replacement](https://link.jianshu.com/?t=https://webpack.js.org/guides/hot-module-replacement/)  
[hot module replacement with webpack](https://link.jianshu.com/?t=https://github.com/webpack/docs/wiki/hot-module-replacement-with-webpack)  
[webpack 热加载你站住，我对你好奇很久了](https://link.jianshu.com/?t=https://sanwen.net/a/wflzwqo.html)


## 2、ProvidePlugin

ProvidePlugin 可以在任何地方自动加载模块而不需要import 或 require 方法:  
例如：通过如下定义，在任何代码中使用identifier变量，'module1’模块都会被加载，identifier所代表的正式‘module1’模块export的内容

```
new webpack.ProvidePlugin({
  identifier: 'module1',
  // ...
})
```

#### 用途

#### \(1）全局变量

如果你的项目里需要使用jQuery，如果每次使用的时候都需要通过import 或 require 去引入jQuery的话未免也太麻烦。这时候ProvidePlugin就可以帮上大忙了

```
//Webpack plugins定义
new webpack.ProvidePlugin({
  $: 'jquery',
  jQuery: 'jquery'
})

// 代码模块中调用
$('#item'); // <= 生效
jQuery('#item'); // <= just works
// $ is automatically set to the exports of module "jquery"
```

上述代码可以看出通过ProvidePlugin定义的‘’被调用时是直接生效的，webpack会自动把\&quot;jquery\&quot;给注入进模块，而‘’被调用时是直接生效的，webpack会自动把\&quot;jquery\&quot;给注入进模块，而‘ ’被调用时是直接生效的，webpack会自动把\&quot;jquery\&quot;给注入进模块，而‘\<math>\<semantics>\<mrow>\<mi mathvariant="normal">’\</mi>\<mi mathvariant="normal">被\</mi>\<mi mathvariant="normal">调\</mi>\<mi mathvariant="normal">用\</mi>\<mi mathvariant="normal">时\</mi>\<mi mathvariant="normal">是\</mi>\<mi mathvariant="normal">直\</mi>\<mi mathvariant="normal">接\</mi>\<mi mathvariant="normal">生\</mi>\<mi mathvariant="normal">效\</mi>\<mi mathvariant="normal">的\</mi>\<mi mathvariant="normal">，\</mi>\<mi>w\</mi>\<mi>e\</mi>\<mi>b\</mi>\<mi>p\</mi>\<mi>a\</mi>\<mi>c\</mi>\<mi>k\</mi>\<mi mathvariant="normal">会\</mi>\<mi mathvariant="normal">自\</mi>\<mi mathvariant="normal">动\</mi>\<mi mathvariant="normal">把\</mi>\<mi mathvariant="normal">\&#x26;quot;\</mi>\<mi>j\</mi>\<mi>q\</mi>\<mi>u\</mi>\<mi>e\</mi>\<mi>r\</mi>\<mi>y\</mi>\<mi mathvariant="normal">\&#x26;quot;\</mi>\<mi mathvariant="normal">给\</mi>\<mi mathvariant="normal">注\</mi>\<mi mathvariant="normal">入\</mi>\<mi mathvariant="normal">进\</mi>\<mi mathvariant="normal">模\</mi>\<mi mathvariant="normal">块\</mi>\<mi mathvariant="normal">，\</mi>\<mi mathvariant="normal">而\</mi>\<mi mathvariant="normal">‘\</mi>\</mrow>\<annotation encoding="application/x-tex">’被调用时是直接生效的，webpack会自动把\&#x26;quot;jquery\&#x26;quot;给注入进模块，而‘\</annotation>\</semantics>\</math>’被调用时是直接生效的，webpack会自动把"jquery"给注入进模块，而‘’在模块中则代表了‘jquery’ export的内容。这样就不需要先let \$=require\(‘jquery’\)再调用啦。

#### （2）ProvidePlugin还可以根据不同环境使用不同配置

在实际的项目开发中我们通常会根据不同环境采用不同的webpack配置文件，如下代码使用了三个不同文件代表了不同环境的配置，development.js、test.js、production.js分别代表了开发、测试、线上环境它们都输出了一个包含name属性的对象：

```
//development.js开发
module.exports = {
    name:'development'
};
//test.js测试
module.exports = {
    name:'test'
};

//production.js线上
module.exports = {
    name:'production'
};

//webpack.dev.config.js 开发环境
    new webpack.ProvidePlugin({
            ENV: 'development'
        })
//webpack.test.config.js 测试环境
new webpack.ProvidePlugin({
            ENV: "test"
        })
//webpack.pub.config.js 线上环境
    new webpack.ProvidePlugin({
            ENV: "production"
        })
```

假如在代码模块中这么调用
```
if ([ENV.name](http://ENV.name) == ‘development’) {  
console.log(‘做一些开发环境的事情’)  
} else if ([ENV.name](http://ENV.name) == ‘test’) {  
console.log(‘做一些测试环境的事情’)  
} else if ([ENV.name](http://ENV.name) == ‘production’) {  
console.log(‘有些事情必须留到线上环境做’)  
}
```
如上ProvidePlugin中定义的ENV在不同环境中就代表了不同模块，这样就可以区别的做一些开发、测试、生产环境不同的事情啦。

###### 更多ProvidePlugin资料参考

[ProvidePlugin](https://link.jianshu.com/?t=https://webpack.js.org/plugins/provide-plugin/)  
[webpack 巧解环境配置问题](https://link.jianshu.com/?t=https://segmentfault.com/a/1190000004053607?utm_source=tuicool&utm_medium=referral)



## 3、CommonsChunkPlugin

CommonsChunkPlugin是一个可以用来提取公共模块的插件，配置：

```
{
  name: string, // or
  names: string[],
  // 这是 common chunk 的名称。已经存在的 chunk 可以通过传入一个已存在的 chunk 名称而被选择。
  // 如果一个字符串数组被传入，这相当于插件针对每个 chunk 名被多次调用
  // 如果该选项被忽略，同时 `options.async` 或者 `options.children` 被设置，所有的 chunk 都会被使用，否则 `options.filename` 会用于作为 chunk 名。

  filename: string,
  // common chunk 的文件名模板。可以包含与 `output.filename` 相同的占位符。
  // 如果被忽略，原本的文件名不会被修改(通常是 `output.filename` 或者 `output.chunkFilename`)

  minChunks: number|Infinity|function(module, count) -> boolean,
  // 在传入  公共chunk(commons chunk) 之前所需要包含的最少数量的 chunks 。
  // 数量必须大于等于2，或者少于等于 chunks的数量
  // 传入 `Infinity` 会马上生成 公共chunk，但里面没有模块。
  // 你可以传入一个 `function` ，以添加定制的逻辑（默认是 chunk 的数量）

  chunks: string[],
  // 通过 chunk name 去选择 chunks 的来源。chunk 必须是  公共chunk 的子模块。
  // 如果被忽略，所有的，所有的 入口chunk (entry chunk) 都会被选择。


  children: boolean,
  // 如果设置为 `true`，所有  公共chunk 的子模块都会被选择

  async: boolean|string,
  // 如果设置为 `true`，一个异步的  公共chunk 会作为 `options.name` 的子模块，和 `options.chunks` 的兄弟模块被创建。
  // 它会与 `options.chunks` 并行被加载。可以通过提供想要的字符串，而不是 `true` 来对输出的文件进行更换名称。

  minSize: number,
  // 在 公共chunk 被创建立之前，所有 公共模块 (common module) 的最少大小。
}
```

webpack用插件CommonsChunkPlugin进行打包的时候，将符合引用次数\(minChunks\)的模块打包到name参数的数组的第一个块里（chunk）,然后数组后面的块依次打包\(查找entry里的key,没有找到相关的key就生成一个空的块\)，最后一个块包含webpack生成的在浏览器上使用各个块的加载代码，所以页面上使用的时候最后一个块必须最先加载。  
打包公共文件

```
new webpack.optimize.CommonsChunkPlugin({
  name: "commons",
  // ( 公共chunk(commnons chunk) 的名称)

  filename: "commons.js",
  // ( 公共chunk 的文件名)

  // minChunks: 3,
  // (模块必须被3个 入口chunk 共享)

  // chunks: ["pageA", "pageB"],
  // (只使用这些 入口chunk)
})
```

配置例子：  
文件目录结构  
![在这里插入图片描述](https://upload-images.jianshu.io/upload_images/280923-37515c561433b583.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/408/format/webp)

```
//main.js
import A from './a'

//main1.js
import A from './a'
import B from './b'

//a.js
var A = 1;
exports.A = A;

//b.js
var B = 1;
exports.B = B;

//lib/jquery.js
var Jquery = 'fake jquery';
exports.$ = Jquery;

//lib/vue.js
var Vue = 'Fake Vue';
exports.Vue = Vue;

//package.json
{
  "name": "CommonsChunkPluginExample",
  "version": "1.0.0",
  "main": "main.js",
  "license": "MIT",
  "scripts": {
    "start": "webpack  --config webpack.dev.config.js "
  },
  "dependencies": {
    "webpack": "^2.6.1"
  }
}
```

实例代码，以上main.js、main1.js为入口文件；a.js、b.js为代码模块文件；lib/jquery.js、lib/vue.js模拟代码库文件

#### 打包main.js、main1.js的的公共模块

```
var path = require('path')
var webpack = require('webpack')

module.exports = {
  entry: {
    main: './main.js',
    main1: './main1.js',
 },
  output: {
    path: path.join(__dirname, 'dist'),
    filename: '[name].js',
    chunkFilename: '[name].js',
  },
  resolve: {
    extensions: [' ', '*', '.js', '.jsx'],
  },
  plugins: [
    new webpack.optimize.CommonsChunkPlugin({
      name: 'commons'
    }),
    new webpack.optimize.CommonsChunkPlugin({
      name: 'load'
    })

  ],
}
```

按照预期  
main.js、main1.js都引用了a.js，所以a.js被打包进commons.js中；b.js只被main1.js引用，将会被打包进main1.js中；打包生成的/dist/load.js包含了Webpack的加载代码：

![在这里插入图片描述](https://upload-images.jianshu.io/upload_images/280923-5dbf39acfa7a8089.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)

```
///dist/commons.js
webpackJsonp([2],[
/* 0 */
/***/ (function(module, exports) {

/**
 * Created by thea on 2017/6/9.
 */
var A = 1;
exports.A = A;

/***/ })
]);

//dist/main1.js

webpackJsonp([0],[
/* 0 */,
/* 1 */
/***/ (function(module, exports) {

var B = 1;
exports.B = B;

/***/ }),
/* 2 */,
/* 3 */
/***/ (function(module, __webpack_exports__, __webpack_require__) {

"use strict";
Object.defineProperty(__webpack_exports__, "__esModule", { value: true });
/* harmony import */ var __WEBPACK_IMPORTED_MODULE_0__a__ = __webpack_require__(0);
/* harmony import */ var __WEBPACK_IMPORTED_MODULE_0__a___default = __webpack_require__.n(__WEBPACK_IMPORTED_MODULE_0__a__);
/* harmony import */ var __WEBPACK_IMPORTED_MODULE_1__b__ = __webpack_require__(1);
/* harmony import */ var __WEBPACK_IMPORTED_MODULE_1__b___default = __webpack_require__.n(__WEBPACK_IMPORTED_MODULE_1__b__);

/***/ })
],[3]);
```

#### 提取第三方库

在通常的项目开发中我们通常会引入一些第三方库，在打包的时候我们通常也会希望将代码拆分成公共代码和应用代码。将webpack.dev.config.js文件配置变化如下：

```
//webpack.dev.config.js
var path = require('path')
var webpack = require('webpack')

module.exports = {
  entry: {
    main: './main.js',
    main1: './main1.js',
    lib:['./lib/jquery.js','./lib/vue.js']//第三方库
  },
  output: {
    path: path.join(__dirname, 'dist'),
    filename: '[name].js',
    chunkFilename: '[name].js',
  },
  resolve: {
    extensions: [' ', '*', '.js', '.jsx'],
  },
  plugins: [
    new webpack.optimize.CommonsChunkPlugin({
      names:['commons','lib']//'lib'提取入口entry key 'lib'代表的文件单独打包
    }),
    new webpack.optimize.CommonsChunkPlugin({
      name: 'load'
    })
  ],
}
```

预期/lib/jquery.js、/lib/vue.js共同打包成为/dist/lib.js

![在这里插入图片描述](https://upload-images.jianshu.io/upload_images/280923-64f11075587a9406.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/478/format/webp)

```
///dist/lib.js
webpackJsonp([0],[
/* 0 */,
/* 1 */,
/* 2 */
/***/ (function(module, exports) {

var Jquery = 'fake jquery';
exports.$ = Jquery;

/***/ }),
/* 3 */
/***/ (function(module, exports) {

var Vue = 'Fake Vue';
exports.Vue = Vue;

/***/ }),
/* 4 */,
/* 5 */,
/* 6 */
/***/ (function(module, exports, __webpack_require__) {

__webpack_require__(2);
module.exports = __webpack_require__(3);


/***/ })
],[6]);
```

#### CommonsChunkPlugin更多资料参考

[CommonsChunkPlugin](https://link.jianshu.com/?t=https://webpack.js.org/plugins/commons-chunk-plugin/)  
[webpack CommonsChunkPlugin详细教程](https://segmentfault.com/a/1190000006808865)


## 4、ExtractTextWebpackPlugin 分离 CSS

Webpack打包默认会把css和js打包在一块，然而我们通常习惯将css代码中放在标签中，而将js引用放在页面底部；将css代码放在页面头部可以避免 FOUC 问题\(表现为页面内容已经加载出来，但是没有样式，过了一会儿样式文件加载出来之后页面回复正常\)；同时如果css和js分离也有利于这加强样式的可缓存性；这是我们需要ExtractTextWebpackPlugin来分离js与css使得样式文件单独打包。

#### webpack处理css

在代码中想JavaScript模块一样引入css文件  
import styles from ‘./style.css’

需要借助css-loader  
和 style-loader  
：

```
npm install --save-dev css-loader style-loader
```

在 webpack.config.js 中配置如下：

```
module.exports = {
    module: {
        rules: [{
            test: /\.css$/,
            use: [ 'style-loader', 'css-loader' ]
        }]
    }
}
```

这样，CSS 会跟你的 JavaScript 打包在一起，并且在初始加载后，通过一个

#### 使用 ExtractTextWebpackPlugin

安装

```
npm install --save-dev extract-text-webpack-plugin
```

为了使用这个插件，它需要通过三步被配置到 webpack.config.js 文件中。

```
var ExtractTextPlugin = require('extract-text-webpack-plugin');
module.exports = {
    module: {
         rules: [{
             test: /\.css$/,
            use: ExtractTextPlugin.extract({
                     fallback: 'style-loader',
                    use: [
                        {
                            loader: 'css-loader',
                            options: {
                                sourceMap: true,
                                importLoaders: 1,
                                modules: true,
                                localIdentName: "[local]---[hash:base64:5]",
                                camelCase: true
                            }
                        }]
            })
         }]
     },
    plugins: [
        new ExtractTextPlugin( ({
            filename: '[name].css',//使用模块名命名
            allChunks: true
        })
    ]
}
```

通过加入ExtractTextWebpackPlugin，每个模块的css都会生成一个新文件，此时你可以作为一个单独标签添加到html文件中。

#### 更多ExtractTextWebpackPlugin资料参考

[webpack-contrib/extract-text-webpack-plugin](https://github.com/webpack-contrib/extract-text-webpack-plugin)  
[webpack ExtractTextWebpackPlugin](https://link.jianshu.com/?t=https://webpack.js.org/plugins/extract-text-webpack-plugin/)



## 5、UglifyJsPlugin代码压缩输出

代码压缩插件UglifyJsPlugin通过UglifyJS2来完成代码压缩，配置参考UglifyJS2，  
调用例子：

```
 new webpack.optimize.UglifyJsPlugin({
      compress: {
        warnings: false
      }
    })
```



##6、设置全局变量插件  
[DefinePlugin](https://webpack.js.org/plugins/define-plugin)

```
// webpack.config.js
const webpack = require('webpack');

module.exports = {
  /*...*/
  plugins:[
    new webpack.DefinePlugin({
      'process.env.NODE_ENV': JSON.stringify('production')
    })
  ]
};
```

通过DefinePlugin定义的变量可在全局代码中使用，例如Webpack配置文件定义了process.env.NODE\_ENV=‘production’，如果代码中存在如下调用：

```
if (process.env.NODE_ENV !== 'production') console.log('...') 
```

即等同于

```
if (false) console.log('...')
```

原理：DefinePlugin做的工作是在源代码基础上执行的全局查找替换工作，将DefinePlugin插件中定义的变量替换为插件中定义的变量值。  
参考  
[DefinePlugin](https://webpack.js.org/plugins/define-plugin)



## DllPlugin\& DllReferencePlugin

在前端项目构建中，为了提高打包效率，往往会将业务代码合第三方库打包。  
之前有将将到过CommonsChunkPlugin可提供第三方库单独打包方法。但是由于每次运行Webpack，即使没有代码更新，也会重新打包，Webpack打包慢也是一直被诟病的问题，竟然减少打包内容才是王道；  
其实Webpack还提供了一个提供了另外一个配置项Externals，提供了「从输出的 bundle 中排除依赖」的方法：  
如

```
index.html
...
<script src="https://code.jquery.com/jquery-3.1.0.js"
  integrity="sha256-slogkvB1K3VOkzAI8QITxV3VzpOnkeNVsKvtkYLMjfk="
  crossorigin="anonymous"></script>
...

webpack.config.js
externals: {
  jquery: 'jQuery'
}
```

这样就剥离了那些不需要改动的依赖模块，换句话，下面展示的代码还可以正常运行：

```
import $ from 'jquery';

$('.my-element').animate(...);
```

不过externals虽然解决了外部引用问题，但如果遇到像react模块本身引入react入口文件，但是 Webpack 不知道这个入口文件等效于 react 模块本身时，会出现重复打包现象，详见彻底解决 Webpack 打包性能问题。  
Webpack提供了DllPlugin、DllReferencePlugin两个插件，既能解决第三方库代码无变化仍然要打包增加打包时间问题，也能解决通过外链引用但可能第三方库还是被打包进来的问题。  
DllPlugin插件用来把需要独立打包的第三方库分离出来单独打包最后会输出两个文件，一个是打包好的第三方库bundle还有一个是用来给 DllReferencePlugin  
映射依赖的manifest.json  
使用方法：  
（1）新建一个专门用来打包第三方库的Webpack配置文件

```
//webpack.dll.config.js
const webpack = require('webpack');
module.exports = {
    entry: {
        react: ['./res/polyfill.js', 'react', 'react-dom', 'redux', 'react-redux', 'react-router']
    },
    output: {
          filename: '[name].bundle.js',
        path: path.join(__dirname, 'res'),
        library: '[name]_lib'
    },
    plugins: [
        new webpack.DllPlugin({
            path: '.res/[name]-manifest.json',//manifest.json输出路径，DllReferencePlugin需要用到

            name: '[name]_library',//打包库文件名
           context：__dirname//可选，引用manifest.json的上下文,默认为Webpack配置文件上下文
        })
    ]
};
```

运行webpack.dll.config.js

```
webpack --config webpack.dll.config.js
```

生成打包好的库bundle，和manifest.json文件 ‘bundle.manifest.json’，大致内容如下：

```
{
 "name": "react_lib",
  "content": {
    "./node_modules/core-js/modules/_export.js": {
      "id": 0,
      "meta": {}
    },
    "./node_modules/core-js/modules/_global.js": {
      "id": 1,
      "meta": {}
    },
    "./node_modules/preact-compat/dist/preact-compat.es.js": {
      "id": 2,
      "meta": {
        "harmonyModule": true
      },
//其他引用
}
```

2、配置webpack.config.js中的插件DllReferencePlugin

```
 plugins: [
        new webpack.DllReferencePlugin({
            context: '.',//需要与webpack.dll.config.js中DllPlugin上下文一致
            manifest: require('./res/react-manifest.json')//DllPlugin输出的manifest.json文件
        })]
```

通过两步完美分离第三方库\~\~\~\~

###### DllPlugin\&DllReferencePlugin资料

[DllPlugin](https://link.jianshu.com/?t=https://webpack.js.org/plugins/dll-plugin/#components/sidebar/sidebar.jsx)  
[彻底解决 Webpack 打包性能问题](https://link.jianshu.com/?t=https://juejin.im/entry/57996222128fe1005411c649)  
[webpack dllPlugin 使用](https://link.jianshu.com/?t=https://github.com/chenchunyong/webpack-dllPlugin)


[转自：清晓凝露 链接：https://www.jianshu.com/p/08859e5651fc 来源：简书](https://link.jianshu.com/?t=https://github.com/chenchunyong/webpack-dllPlugin)