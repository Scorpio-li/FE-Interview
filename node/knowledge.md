<!--
 * @Author: Li Zhiliang
 * @Date: 2020-12-16 18:11:31
 * @LastEditors: Li Zhiliang
 * @LastEditTime: 2020-12-19 10:33:44
 * @FilePath: /FE-Interview.git/node/knowledge.md
-->

# node项目中遇到的常见问题和解决方案

## 1. window和mac下设置NODE_ENV变量的问题

我们都知道在前端项目中会根据不同的环境变量来处理不同的逻辑, 在nodejs中也一样, 我们需要设置本地开发环境, 测试环境, 线上环境等, 此时有一直设置环境变量的方案是在package.json中的script属性中设置, 如下:

```json
"scripts": {
   "start": "export NODE_ENV=development && nodemon -w src --exec \"babel-node src\"",
   "build": "babel src --out-dir dist",
   "run-build": "node dist",
   "test": "echo \"Error: no test specified\" && exit 1"
 }
```

从start指令中我们可以发现我们用export NODE_ENV=development来定义开发环境的环境变量，由于笔者采用的是mac电脑，所以可以用export来定义一个node环境变量. 但是在和朋友合作开发项目时发现执行yarn start后会报错, 后面看错误信息才发现window下不识别export, 后面笔者发现window定义环境变量可以用set, 所以对于window用户, 如果你使用了以上方法设置NODE_ENV, 可以采用如下方式:

```json
"scripts": {
   "start": "set NODE_ENV=development && nodemon -w src --exec \"babel-node src\""
 }
```

## 2. 执行npm install发生node-gyp报错的问题

在项目开发过程中有时候拉取新的node项目代码后执行npm install, 会报如下错误:

**gyp ERR!**

node-gyp就是在node环境中使用的生成不同平台不同编译器的项目文件, 如果你遇到了相同的问题, 我们可以采用如下方案:

```js
npm install -g node-gyp
```

或者直接删除package-lock.json或者yarn.lock, 然后重新yarn install或者npm install即可, 笔者清测有效.

## 3. node + koa2项目中删除已设置的cookie的解决办法

> 由于HTTP是无状态协议，所以需要cookie来区分用户之间的身份。 我们可以把cookie作为是一个由浏览器和服务器共同协作实现的规范。

cookie的处理分为以下3步(基础且重要的知识)：

1. 服务器向客户端发送cookie

2. 浏览器将cookie保存（可以在后端设置expires或者maxAge，以session形式存在）

3. 每次浏览器都会将之前设置好的cookie发向服务器

在开发node后台项目时我们经常涉及用户管理模块, 这意味我们需要对用户进行登录态管理, 在用户退出时能及时删除用户的cookie, 好在koa2自带了处理cookie的方法, 我们可以通过如下的方式设置cookie:

```js
router.post(api.validVip,
    async ctx => {
      ctx.cookies.set('vid', 'xuxiaoxi', { maxAge: 24 * 3600 * 1000 });
    }
);
```

此时客户端的cookie将在下次请求时自动失效.

## 4. 由于nodejs第三方模块依赖特定node版本导致的报错解决方案

这个情况笔者之前也遇到过, 主要原因是第三方没有和node版本做到很好的向后兼容, 此时解决方案就是更新此第三方包到最新版本(如果还在维护的情况), 或者使用node包管理工具(n)切换到适配的node版本, 如下:

```shell
// 更新最新的包
npm i xxx@latest

// 使用包管理工具n
npm i -g n
```

## 5. nodejs如何创建定时任务

定时任务在后端开发中是很常见的功能之一, 其本质是根据时间规则，系统在后台自动执行相应的任务. 在java, PHP 等后台语言中有很丰富的定时任务的支持, 对于nodejs 这个兴起之秀来说, 虽然没有那么成熟的生态, 但是仍然有定时任务的模块, 比如node-schedule.

> **Node Schedule** 是用于Node.js的灵活的 cron 类和非 cron 类作业调度程序。它允许我们使用可选的重复规则来安排作业（任意函数）在特定日期执行。它在任何给定时间仅使用一个计时器（而不是每秒钟/分钟重新评估即将到来的作业）。

```js
let schedule = require('node-schedule');

let testJob = schedule.scheduleJob('42 * * * *', function(){
  console.log('将在未来的每个时刻的42分时执行此代码, 比如22:42, 23:42');
});
```

## 6. 在nodejs项目中使用import, export和修饰器@decorator语法

我们都知道现在nodejs版本已经到14.0+版本了, 对最新的es语法支持的也足够好, 但是目前仍然有一些语法不支持, 比如es的模块导入导出(import, export), 装饰器(@decorator)等,  此时我们要在node项目中使用这些新特性, 我们就不得不借助工具, 这里笔者采用babel7来解决上述问题, 如下:

```js
# .babelrc
{
  "presets": [
    [
      "@babel/preset-env",
      {
        "targets": {
          "node": "current"
        }
      }
    ]
  ],
  "plugins": [
    ["@babel/plugin-proposal-decorators", { "legacy": true }],
    ["@babel/plugin-proposal-class-properties", { "loose" : true }]
  ]
}
```

 我们只需要在项目根目录里新建并写入如上文件, 并安装babel对应的模块即可, 如下:

```js
yarn add 
@babel/cli 
@babel/core 
@babel/node 
@babel/plugin-proposal-class-properties 
@babel/plugin-proposal-decorators 
@babel/preset-env
```


## 7. nodejs中优雅的处理json文件以及提高json读写性能

对于nodejs优化方面其实有很多要聊的, 这里主要来说说json相关的优化方案. 我们需要从2个方面来优化, 一个就是json文件的读写性能, 此时我们可以采用fast-json-stringify 来大大提高json的读写速度, 其本质是提供了一套json-schema约束, 让json结构更加有序, 从而提高json的读取查询速度. 如下使用方式:

```js
const fastJson = require('fast-json-stringify')
const stringify = fastJson({
  title: 'H5 Dooring Schema',
  type: 'object',
  properties: {
    firstName: {
      type: 'string'
    },
    lastName: {
      type: 'string'
    },
    age: {
      description: 'Age in years',
      type: 'integer'
    },
    reg: {
      type: 'string'
    }
  }
})
```

有很多需要频繁读写json数据的接口, 此时使用fast-json-stringify对读写性能会有很大的提升.

另一方面, 我们在node 端操作json, 如果用原生的写法会非常麻烦, 此时我们最好自己对json读取进行封装来提高代码的简约性, 或者我们直接使用第三方库jsonfile 来轻松读写json文件, 如下使用案例:

```js
const json = require('jsonfile')
const fileName = 'h5-dooring.json'
const jsonData = jsonFile.readFileSync(fileName)
```

## 8. nodejs读取大文件报错解决方案

在nodejs中 我们可以使用两种方式来读写文件, 如下:

1. **fs.readFile()** 一次性将文件读取进内存中, 如果文件过大会导致node内存不够而报错

2. **fs.createReadStream()** 以文件流的方式读取, 此时可以不用担心文件的大小

由以上介绍可知如果我们要读取的文件可能会很大(比如视频等大文件), 我们一开始就要使用fs.createReadStream(), 其实如果我们需要对文件进行解析, 比如要对简历等文件进行逐行解析提取关键语料, 我们可以使用node的readline模块, 此时我们就可以对文件进行逐行读取并解析, 如下案例:

```js
const fs = require("fs");
const path = require("path");
const readline = require("readline");

const readlineTask = readline.createInterface({
    input: fs.createReadStream(path.join(__dirname, './h5-dooring')),
});
 
readlineTask.on('line', function(chunk) {
  // 读取每一行数据
});
 
readlineTask.on('close', function() {
  //文件读取结束的逻辑
}
```

##
##
##
##