---
title: webpack性能优化
tag: webpack
category: other
date: 2020-07-30 20:21:00
id: 20200730003
---


# 1\. 剔除不必要的代码

## 1.1 引入未使用

对于js，tree-shaking 贯穿整个依赖树，检查被使用的依赖，移除无用的依赖。 对于css,purifycss-webpack未使用的css不会被打包进来

## 1.2 避免重复引入

### 1.2.1 externals

\(1\)使用变量或外部引用来引入模块，如果两个模块有公共部分，比如jquery，可以避免重复下载。只需在入口.html文件中，使用script引入即可。

\(2\)将模块挂载到window上，减少重复的请求模块。配置：

```
module.exports = {
	externals: {
	'react': 'React',
	'react-dom': 'ReactDOM',
	},
};
复制代码
```

### 1.2.2 CommonsChunkPlugin

提取公共部分。webpack4之后使用optimization.splitChunks代替。

## 1.3 避免执行依赖模块的无用代码

很多模块在生产production模式都会有一些警示性代码，如下图，而这类代码在实际环境中，并没什么用，却占用着资源。使用webpack的DefinePlugin可以解决这个问题。或者使用webpack.EnvironmentPlugin起到类似作用。参考：[webpack.js.org/plugins/env…](https://webpack.js.org/plugins/environment-plugin/)

```
//将process.env.NODE_ENV替换成production。
new webpack.DefinePlugin({
'process.env.NODE_ENV': '"production"',
}),

//上图里面的代码会变成：
if("production" !== 'production'){//永远为false，不会进入分支
    .......
}
复制代码
```

## 1.4 避免构建不必要的代码

loader时，使用exclude将不必要的文件去除，或者include将需要的文件加进来。

# 2\. 资源压缩

## 2.1 js压缩

### 2.1.1 babel-minify-webpack-plugin

loader的时候由于文件大小通常非常大，所以会慢很多，所以这个插件有个作用，就是可以在loader的时候进行优化，减少一定的文件体积。

### 2.1.2 uglifyjs-webpack-plugin

js后处理，具有剔除注释、代码压缩等功能。

## 2.2 css压缩

### 2.2.1 optimize-css-assets-webpack-plugin

css后处理，可以将注释剔除、css代码压缩等。

## 2.3 图片资源压缩

### 2.3.1 image-webpack-loader

压缩图片的作用。配合url-loader/svg-url-loader使用。

## 2.4 gzip压缩

CompressionWebpackPlugin将最后的资源进行gzip压缩，减少体积。

# 3\. 减少网络请求

## 3.1 图片资源压缩和内联

url-loader/svg-url-loader/image-webpack-loader url-loader/svg-url-loader设置一定大小内，图片使用内联方式插入html代码中，内联减少了http请求的数量。 image-webpack-loader有压缩图片的作用。

# 4\. 懒加载

## 4.1 模块按需引入

比如点击事情，需要用到两一个模块中的Function，只在点击的时候引入这个模块中的Function

## 4.2 组件按需引入

只在页面跳转后，将路由所需的组件加载进来，而不是在第一次刷新的时候，将所有组件都加载到一个文件中，避免文件体积过于庞大，且未使用的时候都算暂时无用的代码。

# 5\. 提高构建速度

## 5.1 预编译

### 5.1.1 dllplugin\&DllRefrencePlugin

DllPlugin结合DllRefrencePlugin插件的运用，对将要产出的bundle文件进行拆解 打包，将公共静态资源拆分打包，可以彻底地加快webpack的打包速度，从而在开发过程中极大地缩减构建时间。之后不管是dev还是production不会重复打包这部分静态资源，大大缩减了构建时间。

### 5.1.2 ModuleConcatenationPlugin

打包的时候，它将一些有联系的模块，放到一个闭包函数里面去，通过减少闭包函数数量从而加快JS的执行速度。

## 5.2 多核并行

在多核电脑上，HappyPack能将任务拆分成多个子进程并发的执行，提高构建速度。 webpack-uglify-parallel也是并行的方式，提升uslifyPlugin的构建速度。

## 5.3 减少搜索

### 5.3.1 Resolve.module\&resolve.alias

配置webpack去哪里寻找第三方，减少搜索遍历时间损耗。 resolve.alias设置别名，减少搜索路径的时间损耗。

## 5.4 devtool

配置souce-map为合适的值，有的会比较耗时。

# 6\. 自动化监控工具

## 6.1 webpack-bundle-analyzer

一个项目，大部分代码来自于依赖的模块，依赖的大小严重影响着项目构建包的大小。webpack-bundle-analyzer分析依赖之间的关系，能清晰看到使用到哪些依赖及对应的大小。可以帮助我们有针对性的去优化使用那些体积大的依赖。该工具会在浏览器中打开一个窗口，展示依赖图。

## 6.2 webpack-dashboard

是增强控制台用户体验的一款工具。dashboard里面按日志\(Log\)、状态\(Status\)、运行\(Operation\)、过程\(Progess\)、模块\(Modules\)、产出\(Assets\)这6个部分将信息按区展示。

![](https://user-gold-cdn.xitu.io/2019/8/12/16c83c50b451875e?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

## 6.3 bundlesize

设置类型文件的最大大小，当超出范围，会给与警告和提示，帮助分析那些模块体积过大。

[转自 【掘金**~cyndarila**】](https://juejin.im/post/6844903910973374478)