---
title: 基于 Egg.js 框架的 Node.js 服务构建之用户管理设计
id: '20200803001'
tags:
  - node
date: 2020-08-03 09:13:32
tag: node
category: backend

---



>近来公司需要构建一套 EMM（Enterprise Mobility Management）的管理平台，就这种面向企业的应用管理本身需要考虑的需求是十分复杂的，技术层面管理端和服务端构建是架构核心，客户端本身初期倒不需要那么复杂，作为移动端的负责人（其实也就是一个打杂的小组长），这个平台架构我自然是免不了去参与的，作为一个前端 jser 来公司这边总是接到这种不太像前端的工作，要是以前我可能会有些抵触这种业务层面需要考虑的很多，技术实现本身又不太容易积累技术成长的活。这一年我成长了太多，总是尝试着去做一些可能自己谈不上喜欢但还是有意义的事情，所以这次接手这个任务还是想好好把这个事情做好，所以想考虑参与到 EMM 服务端构建。其实话又说回来，任何事只要想去把它做好，怎么会存在有意义还是没意义的区别呢？

考虑到基于 Node.js 构建的服务目前越来越流行，也方便后续放在平台容器云上构建微服务，另外作为一个前端 jser 出身的程序员，使用 Node.js 来构建服务格外熟悉。之前学习过一段时间 Egg.js，这次毫不犹豫的选择了基于 Egg.js 框架来搭建。

为什么是 Egg.js ？
-------------

去年在 gitchat [JavaScript 进阶之 Vue.js + Node.js 入门实战开发](http://gitbook.cn/books/598144f5e64f69311fe1a813/index.html) 中安利过 Egg.js，那个时候是初接触 Egg.js，但是还是被它惊艳到了，**Egg 继承于 Koa，奉行『约定优于配置』，按照一套统一的约定进行应用开发，插件机制也比较完善**。虽然说 Egg 继承于 Koa，大家可能觉得完全可以自己基于 Koa 去实现一套，没必要基于这个框架去搞，但是其实自己去设计一套这样的框架，最终也是需要去借鉴各家所长，时间成本上短期是划不来的。Koa 是一个小而精的框架，而 Egg 正如文档说的**为企业级框架和应用而生**，对于我们快速搭建一个完备的企业级应用还是很方便的。Egg 功能已经比较完善，另外如果没有实现的功能，自己根据 Koa 社区提供的插件封装一下也是不难的。

ORM 设计选型
--------

在数据库选择上本次项目考虑使用 MySQL，而不是 MongoDB，开始使用的是 egg-mysql 插件，写了一部分后发现 service 里面写了太多东西，表字段修改会影响太多代码，在设计上缺乏对 Model 的管理，看到资料说可以引入 ORM 框架，比如 sequelize，而 Egg 官方恰好提供了 egg-sequelize 插件。

### 什么是 ORM ?

首先了解一下什么是 ORM ?

> 对象关系映射（英语：Object Relational Mapping，简称 ORM，或 O/RM，或 O/R mapping），是一种程序设计技术，用于实现面向对象编程语言里不同类型系统的数据之间的转换。从效果上说，它其实是创建了一个可在编程语言里使用的“虚拟对象数据库”。

类似于 J2EE 中的 DAO 设计模式，将程序中的数据对象自动地转化为关系型数据库中对应的表和列，数据对象间的引用也可以通过这个工具转化为表。这样就可以很好的解决我遇到的那个问题，对于表结构修改和数据对象操作是两个独立的部分，从而使得代码更好维护。其实是否选择 ORM 框架，和以前前端是选择模板引擎还是手动拼字符串一样，ORM 框架避免了在开发的时候手动拼接 SQL 语句，可以防止 SQL 注入，另外也将数据库和数据 CRUD 解耦，更换数据库也相对更容易。

### sequelize 框架

sequelize 是 Node.js 社区比较流行的一个 ORM 框架，相关文档：

*   sequelize.js 文档：[http://docs.sequelizejs.com/](http://docs.sequelizejs.com/)

#### sequelize 使用

**安装：**

```
$ npm install --save sequelize

```
**建立连接：**
```
const Sequelize = require("sequelize");

// 完整用法
const sequelize = new Sequelize("database", "username", "password", {
  host: "localhost",
  dialect: "mysql" | "sqlite" | "postgres" | "mssql",
  operatorsAliases: false,
  pool: {
    max: 5,
    min: 0,
    acquire: 30000,
    idle: 10000
  },
  // SQLite only
  storage: "path/to/database.sqlite"
);
```

// 简单用法
```
const sequelize = new Sequelize("postgres://user:pass@example.com:5432dbname");
```
**校验连接是否正确：**
```
sequelize
  .authenticate()
  .then(() => {
    console.log("Connection has been established successfully.");
  })
  .catch(err => {
    console.error("Unable to connect to the database:", err);
  });
```
**定义 Model ：**

定义一个 Model 的基本语法：
```
sequelize.define("name", { attributes }, { options });
```
例如：
```
const User = sequelize.define("user", {
  username: {
    type: Sequelize.STRING
  },
  password: {
    type: Sequelize.STRING
  }
});
```
对于一个 Model 字段类型设计，主要考虑以下几个方面：

Sequelize 默认会添加 createdAt 和 updatedAt，这样可以很方便的知道数据创建和更新的时间。如果不想使用可以通过设置 attributes 的 timestamps: false；

Sequelize 支持丰富的数据类型，例如：STRING、CHAR、TEXT、INTEGER、FLOAT、DOUBLE、BOOLEAN、DATE、UUID 、JSON 等多种不同的数据类型，具体可以看文档：[DataTypes](http://docs.sequelizejs.com/variable/index.html#static-variable-DataTypes) 。

Getters & setters 支持，当我们需要对字段进行处理的时候十分有用，例如：对字段值大小写转换处理。
```
const Employee = sequelize.define("employee", {
  name: {
    type: Sequelize.STRING,
    allowNull: false,
    get() {
      const title = this.getDataValue("title");
      return this.getDataValue("name") + " (" + title + ")";
    }
  },
  title: {
    type: Sequelize.STRING,
    allowNull: false,
    set(val) {
      this.setDataValue("title", val.toUpperCase());
    }
  }
});
```
字段校验有两种类型：非空校验及类型校验，Sequelize 中非空校验通过字段的 allowNull 属性判定，类型校验是通过 validate 进行判定，底层是通过 [validator.js](https://github.com/chriso/validator.js) 实现的。如果模型的特定字段设置为允许 null（allowNull：true），并且该值已设置为 null，则 validate 属性不生效。例如，有一个字符串字段，allowNull 设置为 true，validate 验证其长度至少为 5 个字符，但也允许为空。
```
const ValidateMe = sequelize.define("foo", {
  foo: {
    type: Sequelize.STRING,
    validate: {
      is: \["^\[a-z\]+$", "i"\], // will only allow letters
      is: /^\[a-z\]+$/i, // same as the previous example using real RegExp
      not: \["\[a-z\]", "i"\], // will not allow letters
      isEmail: true, // checks for email format (foo@bar.com)
      isUrl: true, // checks for url format (http://foo.com)
      isIP: true, // checks for IPv4 (129.89.23.1) or IPv6 format
      isIPv4: true, // checks for IPv4 (129.89.23.1)
      isIPv6: true, // checks for IPv6 format
      isAlpha: true, // will only allow letters
      isAlphanumeric: true, // will only allow alphanumeric characters, so "\_abc" will fail
      isNumeric: true, // will only allow numbers
      isInt: true, // checks for valid integers
      isFloat: true, // checks for valid floating point numbers
      isDecimal: true, // checks for any numbers
      isLowercase: true, // checks for lowercase
      isUppercase: true, // checks for uppercase
      notNull: true, // won't allow null
      isNull: true, // only allows null
      notEmpty: true, // don't allow empty strings
      equals: "specific value", // only allow a specific value
      contains: "foo", // force specific substrings
      notIn: \[\["foo", "bar"\]\], // check the value is not one of these
      isIn: \[\["foo", "bar"\]\], // check the value is one of these
      notContains: "bar", // don't allow specific substrings
      len: \[2, 10\], // only allow values with length between 2 and 10
      isUUID: 4, // only allow uuids
      isDate: true, // only allow date strings
      isAfter: "2011-11-05", // only allow date strings after a specific date
      isBefore: "2011-11-05", // only allow date strings before a specific date
      max: 23, // only allow values <= 23
      min: 23, // only allow values >= 23
      isCreditCard: true, // check for valid credit card numbers

      // custom validations are also possible:
      isEven(value) {
        if (parseInt(value) % 2 != 0) {
          throw new Error("Only even values are allowed!");
          // we also are in the model's context here, so this.otherField
          // would get the value of otherField if it existed
        }
      }
    }

  }
});
```
最后我们说明一个最重要的字段**主键 id** 的设计， 需要通过字段 `primaryKey: true` 指定为主键。MySQL 里面主键设计主要有两种方式：**自动递增**；**UUID**。

自动递增设置 `autoIncrement: true` 即可，对于一般的小型系统这种方式是最方便，查询效率最高的，但是这种不利于分布式集群部署，这种基本用过 MySQL 里面应用都用过，这里不做深入讨论。

UUID, 又名全球独立标识(Globally Unique Identifier)，UUID 是 128 位(长度固定)unsigned integer, 能够保证在空间(Space)与时间(Time)上的唯一性。而且无需注册机制保证, 可以按需随时生成。据 WIKI, 随机算法生成的 UUID 的重复概率为 170 亿分之一。Sequelize 数据类型中有 UUID，UUID1，UUID4 三种类型，基于[node-uuid](https://github.com/kelektiv/node-uuid) 遵循 [RFC4122](http://www.ietf.org/rfc/rfc4122.txt) 。例如：
```
const User = sequelize.define("user", {
  id: {
    type: Sequelize.UUID,
    primaryKey: true,
    allowNull: false,
    defaultValue: Sequelize.UUID1
  }
});
```
这样 id 默认值生成一个 uuid 字符串，例如：'1c572360-faca-11e7-83ee-9d836d45ff41'，很多时候我们不太想要这个 `-` 字符，我们可以通过设置 defaultValue 实现，例如：
```
const uuidv1 = require("uuid/v1");

const User = sequelize.define("user", {
  id: {
    type: Sequelize.UUID,
    primaryKey: true,
    allowNull: false,
    defaultValue: function() {
      return uuidv1().replace(/-/g, "");
    }
  }
});
```
**使用 Model 对象：**

对于 Model 对象操作，Sequelize 提供了一系列的方法：

*   find：搜索数据库中的一个特定元素，可以通过 findById 或 findOne；
*   findOrCreate：搜索特定元素或在不可用时创建它；
*   findAndCountAll：搜索数据库中的多个元素，返回数据和总数；
*   findAll：在数据库中搜索多个元素；
*   复杂的过滤/ OR / NOT 查询；
*   使用 limit(限制)，offset(偏移量)，order(顺序)和 group(组)操作数据集;
*   count：计算数据库中元素的出现次数；
*   max：获取特定表格中特定属性的最大值；
*   min：获取特定表格中特定属性的最小值；
*   sum：特定属性的值求和；
*   create：创建数据库 Model 实例；
*   update：更新数据库 Model 实例；
*   destroy：销毁数据库 Model 实例。

通过上述提供的一系列方法可以实现数据的增删改查（CRUD），例如：
```
User.create({ username: "fnord", job: "omnomnom" })
  .then(() =>
    User.findOrCreate({
      where: { username: "fnord" },
      defaults: { job: "something else" }
    })
  )
  .spread((user, created) => {
    console.log(
      user.get({
        plain: true
      })
    );
    console.log(created);
    /\*
    In this example, findOrCreate returns an array like this:
    \[ {
        username: 'fnord',
        job: 'omnomnom',
        id: 2,
        createdAt: Fri Mar 22 2013 21: 28: 34 GMT + 0100(CET),
        updatedAt: Fri Mar 22 2013 21: 28: 34 GMT + 0100(CET)
      },
      false
    \]
    \*/
  });
```
### egg-sequelize 插件

文档：egg-sequelize：[https://github.com/eggjs/egg-sequelize](https://github.com/eggjs/egg-sequelize)

##### 源码简析

这里我们暂时先不分析 egg 插件规范，暂时先只看看 egg-sequelize/lib/loader.js 里面的实现：
```
"use strict";

const path = require("path");
const Sequelize = require("sequelize");
const MODELS = Symbol("loadedModels");
const chalk = require("chalk");

Sequelize.prototype.log = function() {
  if (this.options.logging === false) {
    return;
  }
  const args = Array.prototype.slice.call(arguments);
  const sql = args\[0\].replace(/Executed \\(.+?\\):\\s{0,1}/, "");
  this.options.logging.info("\[model\]", chalk.magenta(sql), \`(${args\[1\]}ms)\`);
};

module.exports = app => {
  const defaultConfig = {
    logging: app.logger,
    host: "localhost",
    port: 3306,
    username: "root",
    benchmark: true,
    define: {
      freezeTableName: false,
      underscored: true
    }
  };
  const config = Object.assign(defaultConfig, app.config.sequelize);

  app.Sequelize = Sequelize;

  const sequelize = new Sequelize(
    config.database,
    config.username,
    config.password,
    config
  );

  // app.sequelize
  Object.defineProperty(app, "model", {
    value: sequelize,
    writable: false,
    configurable: false
  });

  loadModel(app);

  app.beforeStart(function\*() {
    yield app.model.authenticate();
  });
};

function loadModel(app) {
  const modelDir = path.join(app.baseDir, "app/model");
  app.loader.loadToApp(modelDir, MODELS, {
    inject: app,
    caseStyle: "upper",
    ignore: "index.js"
  });

  for (const name of Object.keys(app\[MODELS\])) {
    const klass = app\[MODELS\]\[name\];

    // only this Sequelize Model class
    if ("sequelize" in klass) {
      app.model\[name\] = klass;
    
      if (
        "classMethods" in klass.options ||
        "instanceMethods" in klass.options
      ) {
        app.logger
          .error(\`${name} model has classMethods/instanceMethods, but it was removed supports in Sequelize V4.\\

see: http://docs.sequelizejs.com/manual/tutorial/models-definition.html#expansion-of-models\`);
      }
    }
  }

  for (const name of Object.keys(app\[MODELS\])) {
    const klass = app\[MODELS\]\[name\];

    if ("associate" in klass) {
      klass.associate();
    }

  }
}
```
很明显在插件初始化的时候进行了 Sequelize 对象的实例化，并将 Sequelize 对象挂载在 app 对象下，即我们可以通过 app.Sequelize 访问 Sequelize 对象，同时我们可以通过 app.model 对 Sequelize 实例化进行访问，app/model 文件夹下存放 model 对象文件。

##### 用户 Model 设计

这里我们以 egg-sequelize 的使用为例加以说明。

**安装：**
```
$ npm i --save egg-sequelize
$ npm install --save mysql2 # For both mysql and mariadb dialects
```
**配置：**
```
app/config/plugin.js 配置：

exports.sequelize = {
  enable: true,
  package: "egg-sequelize"
};

app/config/config.default.js 配置：

// 数据库信息配置
exports.sequelize = {
  // 数据库类型
  dialect: "mysql",
  // host
  host: "localhost",
  // 端口号
  port: "3306",
  // 用户名
  username: "root",
  // 密码
  password: "xxx",
  // 数据库名
  database: "AEMM"
};
```
**Model 层：**

直接使用 Sequelize 虽然可以，但是存在一些问题。团队开发时，有人喜欢自己加 timestamp，有人又喜欢自增主键，并且自定义表名。一个大型 Web App 通常都有几十个映射表，一个映射表就是一个 Model。如果按照各自喜好，那业务代码就不好写。Model 不统一，很多代码也无法复用。所以我们需要一个统一的模型，强迫所有 Model 都遵守同一个规范，这样不但实现简单，而且容易统一风格。

我们首先要定义的就是 Model 存放的文件夹必须在 models 内，并且以 Model 名字命名，例如：Pet.js，User.js 等等。其次，每个 Model 必须遵守一套规范：

*   统一主键，名称必须是 id，类型必须是 UUID；
*   所有字段默认为 NULL，除非显式指定；
*   统一 timestamp 机制，每个 Model 必须有 createdAt、updatedAt 和 version，分别记录创建时间、修改时间和版本号。

所以，我们不要直接使用 Sequelize 的 API，而是通过 db.js 间接地定义 Model。例如，User.js 应该定义如下：
```
app/db.js：

const uuidv1 = require("uuid/v1");

function generateUUID() {
  return uuidv1().replace(/-/g, "");
}

function defineModel(app, name, attributes) {
  const { UUID } = app.Sequelize;

  let attrs = {};
  for (let key in attributes) {
    let value = attributes\[key\];
    if (typeof value === "object" && value\["type"\]) {
      value.allowNull = value.allowNull || true;
      attrs\[key\] = value;
    } else {
      attrs\[key\] = {
        type: value,
        allowNull: true
      };
    }
  }

  attrs.id = {
    type: UUID,
    primaryKey: true,
    defaultValue: () => {
      return generateUUID();
    }
  };

  return app.model.define(name, attrs, {
    createdAt: "createdAt",
    updatedAt: "updatedAt",
    version: true,
    freezeTableName: true
  });
}

module.exports = { defineModel };
```
我们定义的 defineModel 就是为了强制实现上述规则。
```
app/model/User.js：

const db = require("../db");

module.exports = app => {
  const { STRING, INTEGER, DATE, BOOLEAN } = app.Sequelize;

  const User = db.defineModel(app, "users", {
    username: { type: STRING, unique: true, allowNull: false }, // 用户名
    email: { type: STRING, unique: true, allowNull: false }, // 邮箱
    password: { type: STRING, allowNull: false }, // 密码
    name: STRING, // 姓名
    sex: INTEGER, // 用户性别：1男性, 2女性, 0未知
    age: INTEGER, // 年龄
    avatar: STRING, // 头像
    company: STRING, // 公司
    department: STRING, // 部门
    telePhone: STRING, // 联系电话
    mobilePhone: STRING, // 手机号码
    info: STRING, // 备注说明
    roleId: STRING, // 角色id
    status: STRING, // 用户状态
    token: STRING, // 认证 token
    lastSignInAt: DATE // 上次登录时间
  });

  return User;
};
```
在数据库操作设计中，我们一般是通过脚本提前生成表结构，如果手动写创建表的 SQL，每次修改表结构其实是一件麻烦事。Sequelize 提供了[Migrations](http://docs.sequelizejs.com/manual/tutorial/migrations.html) 帮助创建或迁移数据库，egg-sequelize 里面也提供了方便的方法。如果是开发阶段，可以使用下面的方法自动执行：
```
// {app\_root}/app.js
module.exports = app => {
  if (app.config.env === "local") {
    app.beforeStart(function\*() {
      yield app.model.sync({ force: true });
    });
  }
};
```
当然也可以在 package.json 里面添加下面的脚本：

命令

说明
```
npm run migrate:new
```
在 ./migrations/ 中创建一个 迁移文件 to
```
npm run migrate:up
```
执行迁移
```
npm run migrate:down
```
回滚一次迁移
```
package.json：

...
"scripts": {
  "migrate:new": "egg-sequelize migration:create --name init",
  "migrate:up": "egg-sequelize db:migrate",
  "migrate:down": "egg-sequelize db:migrate:undo"
}
...
```
执行 npm run migrate:new 后修改 migrations 文件夹下的文件：
```
module.exports = {
  async up(queryInterface, Sequelize) {
    const { UUID, STRING, INTEGER, DATE, BOOLEAN } = Sequelize;

    await queryInterface.createTable("users", {
      id: {
        type: UUID,
        primaryKey: true,
        allowNull: false
      }, // 用户 ID（主键）
      username: { 
        type: STRING, 
        unique: true, 
        allowNull: false 
      }, // 用户名
      email: { 
        type: STRING, 
        unique: true, 
        allowNull: false
      }, // 邮箱
      password: { 
        type: STRING, 
        allowNull: false 
      }, // 登录密码
      name: STRING, // 姓名
      age: INTEGER, // 用户年龄
      info: STRING, // 备注说明
      sex: INTEGER, // 用户性别：1男性, 2女性, 0未知
      telePhone: STRING, // 联系电话
      mobilePhone: STRING, // 手机号码
      roleId: STRING, // 角色ID
      location: STRING, // 常住地
      avatar: STRING, // 头像
      company: STRING, // 公司
      department: STRING, // 部门
      emailVerified: BOOLEAN, // 邮箱验证
      token: STRING, // 身份认证令牌
      status: { type: INTEGER, allowNull: false }, // 用户状态：1启用, 0禁用, 2隐藏, 3删除
      createdAt: DATE, // 用户创建时间
      updatedAt: DATE, // 用户信息更新时间
      lastSignInAt: DATE // 上次登录时间
    });

  },

  async down(queryInterface, Sequelize) {
    await queryInterface.dropTable("users");
  }
};
```
用户认证选型
------

所谓用户认证（Authentication），就是让用户登录，并且在接下来的一段时间内让用户访问网站时可以使用其账户，而不需要再次登录的机制。

> 小知识：可别把用户认证和用户授权（Authorization）搞混了。用户授权指的是规定并允许用户使用自己的权限，例如发布帖子、管理站点等。

用户认证主要分为两个部分：

*   用户通过用户名和密码登录生成并且获取 Token；
*   用户通过 Token 验证用户身份获取相关信息。

### JSON Web Token（JWT）规范

[JSON Web Token](https://jwt.io/) （JWT）是一个非常轻巧的[规范](https://tools.ietf.org/html/draft-ietf-oauth-json-web-token-32) 。这个规范允许我们使用 JWT 在用户和服务器之间传递安全可靠的信息。

#### JWT 的组成

一个 JWT 实际上就是一个字符串，它由三部分组成，头部、载荷与签名。

**头部（Header）**

JWT 需要一个头部，头部用于描述关于该 JWT 的最基本的信息，例如其类型以及签名所用的算法等。这也可以被表示成一个 JSON 对象。
```
{
  "typ": "JWT",
  "alg": "HS256"
}
```
在这里，我们说明了这是一个 JWT，并且我们所用的签名算法是 HS256 算法。对它也要进行 Base64 编码，之后的字符串就成了 JWT 的 Header（头部）。
```
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9
```
这里我们使用 base64url 模块进行 Base64 编码来得到这个字符串，测试代码如下：
```
const base64url = require("base64url");

let header = {
  typ: "JWT",
  alg: "HS256"
};

console.log("header: " + base64url(JSON.stringify(header)));
// header: eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9
```
> 小知识：Base64 是一种编码，也就是说，它是可以被翻译回原来的样子来的。它并不是一种加密过程。

**载荷（Payload）**

说白了就是我们需要包含的数据，类似于网络请求的请求体 body，例如：
```
{
  "iss": "zhaomenghaun",
  "sub": "\*@agree.com.cn",
  "aud": "www.agree.com.cn",
  "exp": 1526875179,
  "iat": 1526871579,
  "id": "49a9dd505c9d11e8b5e86b9776bb3c4f"
}
```
这里面的前五个字段都是由 JWT 的标准所定义的。

*   iss: 该 JWT 的签发者
*   sub: 该 JWT 所面向的用户
*   aud: 接收该 JWT 的一方
*   exp(expires): 什么时候过期，这里是一个 Unix 时间戳
*   iat(issued at): 在什么时候签发的

将下面的 JSON 对象进行**base64 编码**可以得到下面的字符串，这个字符串我们将它称作 JWT 的 Payload（载荷）。
```
const base64url = require("base64url");

let payload = {
  id: "49a9dd505c9d11e8b5e86b9776bb3c4f",
  iat: 1526871579,
  exp: 1526875179
};
console.log("payload: " + base64url(JSON.stringify(payload)));
// payload: eyJpZCI6IjQ5YTlkZDUwNWM5ZDExZThiNWU4NmI5Nzc2YmIzYzRmIiwiaWF0IjoxNTI2ODcxNTc5LCJleHAiOjE1MjY4NzUxNzl9
```
**签名（Signature）**

将上面的两个编码后的字符串都用句号.连接在一起（头部在前），就形成了:
```
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpZCI6IjQ5YTlkZDUwNWM5ZDExZThiNWU4NmI5Nzc2YmIzYzRmIiwiaWF0IjoxNTI2ODcxNTc5LCJleHAiOjE1MjY4NzUxNzl9

```
最后，我们将上面拼接完的字符串用 HS256 算法进行加密。在加密的时候，我们还需要提供一个密钥（secret）。我们可以使用 [node-jwa](https://github.com/brianloveswords/node-jwa) 进行 HS256 算法加密。如果我们用 123456 作为密钥的话，那么就可以得到我们加密后的内容，这一部分又叫做签名。最后一步签名的过程，实际上是对头部以及载荷内容进行签名。

![](https://blog.leapoahead.com/2015/09/06/understanding-jwt/sig1.png)
```
const jwa = require("jwa");
const hmac = jwa("HS256");

let secret = "123456";
const signature = hmac.sign(
  "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpZCI6IjQ5YTlkZDUwNWM5ZDExZThiNWU4NmI5Nzc2YmIzYzRmIiwiaWF0IjoxNTI2ODcxNTc5LCJleHAiOjE1MjY4NzUxNzl9",
  secret
);
console.log("signature: " + signature);
// signature: JtrTx9QaN3BD1QkZhY58MTu6WHn\_vQwRBxO9VwJgkhE

```
最后将这一部分签名也拼接在被签名的字符串后面，我们就得到了完整的 JWT，如下：
```
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpZCI6IjQ5YTlkZDUwNWM5ZDExZThiNWU4NmI5Nzc2YmIzYzRmIiwiaWF0IjoxNTI2ODcxNTc5LCJleHAiOjE1MjY4NzUxNzl9.JtrTx9QaN3BD1QkZhY58MTu6WHn\_vQwRBxO9VwJgkhE
```
整个完整过程走下来我们需要思考一下问题，Token 是否安全，是否可以传输敏感信息？

我们现在明白了一个 token 是由 Header 的 Base64 编码 + Payload 的 Base64 编码 + Signature 三段组成，当其他人拿到我们的 Token，可以通过 Token 前两段 Base64 解码得到 Header 和 Payload 对象，这里我们通过 [node-jsonwebtoken](https://github.com/auth0/node-jsonwebtoken) 模块 decode 方法直接 "破解" 我们的 Token。
```
const jwt = require("jsonwebtoken");

let decoded = jwt.decode(
  "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpZCI6IjQ5YTlkZDUwNWM5ZDExZThiNWU4NmI5Nzc2YmIzYzRmIiwiaWF0IjoxNTI2ODcxNTc5LCJleHAiOjE1MjY4NzUxNzl9.JtrTx9QaN3BD1QkZhY58MTu6WHn\_vQwRBxO9VwJgkhE",
  { complete: true }
);
console.log("jsonwebtoken: " + JSON.stringify(decoded));
// jsonwebtoken: {"header":{"typ":"JWT","alg":"HS256"},"payload":{"id":"49a9dd505c9d11e8b5e86b9776bb3c4f","iat":1526871579,"exp":1526875179},"signature":"JtrTx9QaN3BD1QkZhY58MTu6WHn\_vQwRBxO9VwJgkhE"}
```
所以我们的 payload 不能里面不能包含诸如密码这种敏感信息，对于我们这里的 id 是一串 uuid，即使拿到也无法直接判定相关内容，从而不会直接泄露我们的内容。

一般而言，加密算法对于不同的输入产生的输出总是不一样的。对于两个不同的输入，产生同样的输出的概率极其地小。如果有人对头部以及载荷的内容解码之后进行修改，再进行编码的话，那么新的头部和载荷的签名和之前的签名就将是不一样的，而且如果不知道服务器加密的时候用的密钥的话，得出来的签名也一定会是不一样的。

所以服务端拿到 JWT 后，首先会校验签名是否过期，以及对头部和载荷的内容用同一算法（通过 JWT 的头部 alg 字段指定）再次签名得到的 JWT 和用户传递的 JWT 是否一致。如果服务器应用对头部和载荷再次以同样方法签名之后发现，自己计算出来的签名和接受到的签名不一样，那么就说明这个 Token 的内容被别人动过的，我们应该拒绝这个 Token，返回一个 HTTP 401 Unauthorized 响应。

### egg-jwt 插件

文档：[https://github.com/okoala/egg-jwt](https://github.com/okoala/egg-jwt)

egg-jwt 基于 [node-jsonwebtoken](https://github.com/auth0/node-jsonwebtoken) 实现，完整文档可以参考 [https://github.com/auth0/node-jsonwebtoken](https://github.com/auth0/node-jsonwebtoken) 。jwt 对象挂载在 app 对象下，可以通过 app.jwt 访问 jwt 的三个方法：

*   jwt.sign(payload, secretOrPrivateKey, \[options, callback\])————生成 token 字符串
*   jwt.verify(token, secretOrPublicKey, \[options, callback\])————校验 token 合法性
*   jwt.decode(token \[, options\])————token 译码

**安装：**
```
$ npm i egg-jwt --save
```
**配置：**
```
app/config/plugin.js 配置：

exports.jwt = {
  enable: true,
  package: "egg-jwt"
};

app/config/config.default.js 配置：

exports.jwt = {
  enable: false,
  secret: "xxxxxxxxxxxxx"
};
```
**调用：**

请求头：
```
Authorization: Bearer {access\_token}
```
注：access\_token 为登录后返回的 token 值。
```
app/service/user.js：

/\*\*
 \* 生成 Token
 \* @param {Object} data
 \*/
createToken(data) {
  return app.jwt.sign(data, app.config.jwt.secret, {
    expiresIn: "12h"
  });
}

/\*\*
 \* 验证token的合法性
 \* @param {String} token
 \*/
verifyToken(token) {
  return new Promise((resolve, reject) => {
    app.jwt.verify(token, app.config.jwt.secret, function(err, decoded) {
      let result = {};
      if (err) {
        /\*
          err = {
            name: 'TokenExpiredError',
            message: 'jwt expired',
            expiredAt: 1408621000
          }
        \*/
        result.verify = false;
        result.message = err.message;
      } else {
        result.verify = true;
        result.message = decoded;
      }
      resolve(result);
    });
  });
}

extend/helper.js：

// 获取 Token
exports.getAccessToken = ctx => {
  let bearerToken = ctx.request.header.authorization;
  return bearerToken && bearerToken.replace("Bearer ", "");
};

// 校验 Token
exports.verifyToken = async (ctx, userId) => {
  let token = this.getAccessToken(ctx);
  let verifyResult = await ctx.service.user.verifyToken(token);
  if (!verifyResult.verify) {
    ctx.helper.error(ctx, 401, verifyResult.message);
    return false;
  }
  if (userId != verifyResult.message.id) {
    ctx.helper.error(ctx, 401, "用户 ID 与 Token 不一致");
    return false;
  }
  return true;
};

// 处理成功响应
exports.success = (ctx, result = null, message = "请求成功", status = 200) => {
  ctx.body = {
    code: 0,
    message: message,
    data: result
  };
  ctx.status = status;
};

// 处理失败响应
exports.error = (ctx, code, message) => {
  ctx.body = {
    code: code,
    message: message
  };
  ctx.status = code;
};

controller 中调用：

// 生成Token
let token = ctx.service.user.createToken({ id: user.id });

// 校验Token合法性
let isVerify = await ctx.helper.verifyToken(ctx, id);
if (isVerify) {
  // 合法逻辑
  // ...
}
```
这样对于需要进行身份认证的 restful API，就可以通过 token 进行认证，从而实现用户认证和授权。

参考
--

*   [JSON Web Token - 在 Web 应用间安全地传递信息](https://blog.leapoahead.com/2015/09/06/understanding-jwt/)
*   [八幅漫画理解使用 JSON Web Token 设计单点登录系统](https://blog.leapoahead.com/2015/09/07/user-authentication-with-jwt/)

【转自】[https://zhaomenghuan.js.org/blog/nodejs-eggjs-usersytem.html](https://zhaomenghuan.js.org/blog/nodejs-eggjs-usersytem.html)