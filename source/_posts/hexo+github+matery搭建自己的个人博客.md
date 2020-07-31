---
title: hexo+github+matery搭建自己的个人博客
tag: blog
category: other
date: 2020-07-31 20:21:00
id: 20200731001
---

>之前用`wordpress`折腾了个博客，服务器、域名、备案、`docker`、乱七八糟的折腾了好久，现在推荐一个生成静态网站的工具，用来写博客，简直不需要太爽，`Hexo`配合`github`以及配合 `Travis CI` 工具自动打包，happy啊.....

**本人用的是mac，windows大同小异，水平有限，有毛病随时交流哈.......**

[Hexo 官网地址](https://hexo.io/zh-cn/docs/)

## 准备工作

本地开发环境先装上`node`以及`git`,具体安装方式自行**百度**

## 本地部分

### 1.全局安装`hexo-cli`
```
npm install -g hexo-cli
```
### 2.进入一个本地文件夹，初始化hexo
- 初始化hexo
```
//名字随便取，我已经部署过blog了，这里用delfino
hexo init delfino
```
- 初始化后，进入该目录，安装依赖
```
cd delfino
npm install
```
- 安装完了之后就是启动服务了
```
hexo server
//端口默认4000
//可以指定端口运行，如下：
hexo server -p 5000
//自定义IP
hexo server -i 192.168.1.1
```
- 浏览器打开试一下是否成功[http://localhost:4000/](http://localhost:4000/)

- 初始化本地的git仓库，方便提交到github
```
git init
```

## github部分
### 1.创建一个test的仓库
**没有github的自行注册**
![](https://imgkr2.cn-bj.ufileos.com/62a22056-41b2-471f-9f5a-4b8dc5b69d12.jpg?UCloudPublicKey=TOKEN_8d8b72be-579a-4e83-bfd0-5f6ce1546f13&Signature=MvsPE6RgxjzEuwe9FKdobURcCzY%253D&Expires=1596260088)

仓库选择公开，点击创建,成功后如下图：
![](https://imgkr2.cn-bj.ufileos.com/b13b351b-a714-4de2-bf6d-489284e2c5c9.jpg?UCloudPublicKey=TOKEN_8d8b72be-579a-4e83-bfd0-5f6ce1546f13&Signature=IcuuD6bkJCLeyqSi8wKqKYUveQE%253D&Expires=1596260163)

### 2.将本地的项目推送到github
```
//进入本地项目目录
git remote add origin https://github.com/simple-delfino/delfino.git
git add .
git commit -m "提交Hexo"
//第一次提交到github上加上 -u
git push -u origin master
//后面如果需要提交,直接
git push origin master
```
## Travis CI 部分
### 1.将 [Travis CI](https://github.com/marketplace/travis-ci)添加到你的 GitHub 账户中。
进入到Travis CI官网，安装：
![](https://imgkr2.cn-bj.ufileos.com/9ba11bb0-9413-46d7-a1f8-6f5c48fb8db1.jpg?UCloudPublicKey=TOKEN_8d8b72be-579a-4e83-bfd0-5f6ce1546f13&Signature=O4xCHESMyn0IlX%252FADr%252FlIIBCwfU%253D&Expires=1596260569)
**进入官网往下翻，看到如上图的位置，选择Open Source，你有钱当我没说，安装**

### 2.配置 Travis CI 权限
**前往 `GitHub` 的 Applications settings，配置 Travis CI 权限，使其能够访问你的 repository。**
![](https://imgkr2.cn-bj.ufileos.com/6e2f5bc0-59cc-4596-9507-6ca7f1ea1c67.jpg?UCloudPublicKey=TOKEN_8d8b72be-579a-4e83-bfd0-5f6ce1546f13&Signature=N6Ya6o%252B4T5pk8Jg9AY1Bl8RRDYs%253D&Expires=1596260819)

- 点击Travis CI的**Configure**
![](https://imgkr2.cn-bj.ufileos.com/a9c5202b-e4fc-4dde-9367-5367cf3ef70b.jpg?UCloudPublicKey=TOKEN_8d8b72be-579a-4e83-bfd0-5f6ce1546f13&Signature=bOkr3%252F%252F0S7gRG51DmfxP4PP%252F%252BgY%253D&Expires=1596260979)
- 设置**Repository access**，我这里选择了All repositories，你也可以只选择你当前的博客仓库，随意，点击保存
![](https://imgkr2.cn-bj.ufileos.com/4ca0f55c-042d-4d70-be7e-96ec4e61cecc.jpg?UCloudPublicKey=TOKEN_8d8b72be-579a-4e83-bfd0-5f6ce1546f13&Signature=WIWTMojc%252FrdZ3X8D8oM37NRLRZc%253D&Expires=1596261107)

### 3.Travis CI 的页面
**你应该会被重定向到 Travis CI 的页面。如果没有，请 [手动前往](https://travis-ci.com/dashboard)。**

### 4.新建 [Personal Access Token]
**在浏览器新建一个标签页，前往 GitHub 新建 [Personal Access Token](https://github.com/settings/tokens)，只勾选 repo 的权限并生成一个新的 Token。Token 生成后请复制并保存好。**
![](https://imgkr2.cn-bj.ufileos.com/e1bc538f-7d14-4db3-ac2c-7868ecdc698d.jpg?UCloudPublicKey=TOKEN_8d8b72be-579a-4e83-bfd0-5f6ce1546f13&Signature=BVrIY1dRyntKvma7sjdlKZd01Gs%253D&Expires=1596261375)

### 5.新建Token环境变量
**回到 Travis CI，前往你的 repository 的设置页面，在 Environment Variables 下新建一个环境变量，Name 为 GH_TOKEN，Value 为刚才你在 GitHub 生成的 Token。确保 DISPLAY VALUE IN BUILD LOG 保持 不被勾选 避免你的 Token 泄漏。点击 Add 保存**
![](https://imgkr2.cn-bj.ufileos.com/6c00b26b-cb7c-4ac0-a6a6-19130f0178ab.jpg?UCloudPublicKey=TOKEN_8d8b72be-579a-4e83-bfd0-5f6ce1546f13&Signature=FK1V%252F07RYKyDIdZ32d%252FfZn2BF6I%253D&Expires=1596261478)

![](https://imgkr2.cn-bj.ufileos.com/e4db1191-5315-4be4-ad98-9df9dec5ba99.jpg?UCloudPublicKey=TOKEN_8d8b72be-579a-4e83-bfd0-5f6ce1546f13&Signature=HD45NxNDxL%252B0qVy4Nbto%252FoPjrV0%253D&Expires=1596261591)

### 6.新建.travis.yml
**在你的 Hexo 站点文件夹中新建一个 .travis.yml 文件：**
```
sudo: false
language: node_js
node_js:
  - 10 # use nodejs v10 LTS
cache: npm
branches:
  only:
    - master # build master branch only
script:
  - hexo generate # generate static files
deploy:
  provider: pages
  skip-cleanup: true
  github-token: $GH_TOKEN
  keep-history: true
  on:
    branch: master
  local-dir: public
  ```
### 7.设置url
**编辑本地项目的`_config.yml`,修改如下：**
```
url: https://simple-delfino.github.io/delfino
root: /delfino
```
### 8.将 .travis.yml以及修改的_config.yml推送到 repository 中。
Travis CI 应该会自动开始运行，并将生成的文件推送到同一 repository 下的 gh-pages 分支下
```
git add .
git commit -m 'push .travis.yml'
git push origin master
```
### 9.修改 GitHub Pages 的部署分支为 gh-pages
在 GitHub 中前往你的 repository 的设置页面，修改 GitHub Pages 的部署分支为 gh-pages

### 10.尝试访问
前往 https://<你的github用户名>.github.io/<博客仓库目录>,eg:https://simple-delfino.github.io/delfino 查看你的站点是否可以访问。这可能需要一些时间。
![](https://imgkr2.cn-bj.ufileos.com/97115fac-535b-442c-94f6-5583004a083b.jpg?UCloudPublicKey=TOKEN_8d8b72be-579a-4e83-bfd0-5f6ce1546f13&Signature=uv0qrk%252FXj9p1mWJqW4npVInaHyQ%253D&Expires=1596262739)

## 安装主题
默认的主题有点丑，可以自己安装主题，我自己现在用的是matery，可以从我的[博客](https://simple-delfino.github.io/blog/)上看效果,
当然beauty也不错，还有其他的极简风格的黑白主题，去百度搜索吧
### 1.下载matery
[matery地址](https://github.com/blinkfox/hexo-theme-matery)

进入到本地项目的theme目录下

```
cd theme
git clone https://github.com/blinkfox/hexo-theme-matery.git
```
### 2.切换主题
修改 Hexo 根目录下的 _config.yml 的 `theme` 的值：`theme: hexo-theme-matery`

### 3.新建分类 categories 页
categories 页是用来展示所有分类的页面，如果在你的博客 source 目录下还没有 categories/index.md 文件，那么你就需要新建一个，命令如下：

```
hexo new page "categories"
```
编辑你刚刚新建的页面文件 /source/categories/index.md，至少需要以下内容：
```
---
title: categories
date: 2018-09-30 17:25:30
type: "categories"
layout: "categories"
---
```
### 4.新建标签 tags 页
tags 页是用来展示所有标签的页面，如果在你的博客 source 目录下还没有 tags/index.md 文件，那么你就需要新建一个，命令如下：
```
hexo new page "tags"
```
编辑你刚刚新建的页面文件 /source/tags/index.md，至少需要以下内容：
```
---
title: tags
date: 2018-09-30 18:23:38
type: "tags"
layout: "tags"
---
```
### 5.新建关于我 about 页
about 页是用来展示关于我和我的博客信息的页面，如果在你的博客 source 目录下还没有 about/index.md 文件，那么你就需要新建一个，命令如下：
```
hexo new page "about"
```
编辑你刚刚新建的页面文件 /source/about/index.md，至少需要以下内容：
```
---
title: about
date: 2018-09-30 17:25:30
type: "about"
layout: "about"
---
```
### 6.新建留言板 contact 页（可选的）
contact 页是用来展示留言板信息的页面，如果在你的博客 source 目录下还没有 contact/index.md 文件，那么你就需要新建一个，命令如下：
```
hexo new page "contact"
```
编辑你刚刚新建的页面文件 /source/contact/index.md，至少需要以下内容：
```
---
title: contact
date: 2018-09-30 17:25:30
type: "contact"
layout: "contact"
---
```
>注：本留言板功能依赖于第三方评论系统，请激活你的评论系统才有效果。并且在主题的 _config.yml 文件中，第 19 至 21 行的“菜单”配置，取消关于留言板的注释即可。

## 写在最后
hexo还是比较好用的，其他的设置自己捉摸吧
比如文章的永久链接，我用的是
```
url: https://simple-delfino.github.io/blog
root: /blog
permalink: :category/:id/
permalink_defaults:
```
当然你文章的`Front-matter`必须有`category`和`id`(id我就是固定的数字，年月日+001或者002)

其他的配置自己折腾吧，`hexo`的配置在根目录下的`_config.yml`，主题的配置在该主题目录下的`_config.yml`，有一些比较好玩的优化，比如每日一句等等 去百度吧
