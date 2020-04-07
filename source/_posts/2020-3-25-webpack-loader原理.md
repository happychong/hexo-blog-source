---
layout: default
title: webpack - loader 原理与实现
description: webpack
categories: [周边]
tags: [webpack, loader]
---

## 提前准备webpack项目

```javascript
// package.json
{
  "name": "10.mywebpack",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "build": "webpack"
  },
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "@babel/core": "^7.9.0",
    "@babel/preset-env": "^7.9.0",
    "webpack": "^4.42.1",
    "webpack-cli": "^3.3.11"
  }
}

```

``` javascript
// webpack.config.js

const path = require('path');
module.exports = {
  mode: 'development',
  devtool: 'none',
  entry: './src/index.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'bundle.js'
  },
  resolveLoader: {
    alias: {
      // 配置 module -> rules -> babel-loader 路径
      'babel-loader': path.resolve(__dirname, 'loaders/babel-loader.js')
    },
    // 配置 module -> rules 下配置的loader都先去当前路径的loaders目录下查找，再去node_modules下查找
    modules: [path.resolve(__dirname, 'loaders'), 'node_modules']
  },
  module: {
    rules: [
      {
        test: /\.js$/,
        use: ['babel-loader'] // 这样的写法会到node_modules下找babel-loader
      }
    ]
  },
  plugins: [

  ]
}
```

```javascript
// src/index.js
let title = require('./title');
console.log(title);
```

```javascript
// src/title.js
module.exports = 'title';
```

## 最简单的打包原理——最简单的 babel-loader

loader是个函数，参数接受一些代码内容，经过一些转换返回新的内容，loader的执行顺序是倒序执行

``` javascript
// loaders/babel-loader.js
const babel = require('@babel/core');

function loader(source, sourceMap) {
  // source - 输入的内容 要串换的内容
  console.log('this is my loader ! ~~~~~~~~~~~~');
  // console.log运行的时候，这里会被打印出2次，因为有2个文件啊，index.js & title.js
  const options = {
    presets: [], // babel预设 '@bebal/preset-env'
    inputSourceMap: sourceMap, // 输入的 源映射
    sourceMaps: true, // 输出 要 sourceMaps
    // this.request = loader1!loader2!loader3!indexedDB.css
    filename: this.request.split('!').pop() // 指定文件名
  }
  // code : 转译后代码  map：sourcemap   ast：抽象语法树
  let {code, map, ast} = babel.transform(source, options);
  return this.callback(null, code, map, ast)
}

module.exports = loader;
```

现在命令行执行 npm run build ，就会看到dist目录下有用了我们自己写的loader（loaders/babel-loader.js ）的 bundle.js 文件了

## loader 的加载顺序

### pitch 与 normal 的执行顺序

假如我们现在对.js类型的文件配置了3个loader ['loader1', 'loader2', 'loader3'],那么现在loader的执行顺序应该是 源代码（ex: index.js） -> loader3 -> loader2 -> loader1，实践代码如下

``` javascript
// webpack.config.js
{
  module: {
    rules: [
      // pitch
      {
        test: /\.js$/,
        use: ['patch-loader1', 'patch-loader2', 'patch-loader3']
			}
		]
  }
}
```

``` javascript
// loaders/patch-loader1.js
function loader(source, sourceMap) {
  console.log('patch-loader1111');
  return source + '///patch-loader1111///';
}
module.exports = loader;

// loaders/patch-loader2.js
function loader(source, sourceMap) {
  console.log('patch-loader222');
  return source + '///patch-loader222///';
}
module.exports = loader;

// loaders/patch-loader3.js
function loader(source, sourceMap) {
  console.log('patch-loader333');
  return source + '///patch-loader333///';
}
module.exports = loader;
```

命令行运行 npm run build 可以看到 loader 的执行顺序为 3 - 2 - 1，，dist下打包的bundle.js中也可以看到执行顺序，但是其实这里看到的执行顺序是不完全的，实际上每个loader有2个方法，以上的执行方法叫做 normal 方法，另外还有一个方法，就叫做 pitch，pitch 方法是可有可无的，如果有 pitch 方法的话，会先顺序执行每个 loader 的 pitch 方法，然后再倒序执行 loader 的 normal 方法，但是如果某个 loader 有 pitch 方法，并且 pitch 方法有返回值，那么就会跳过剩下的其他方法，直接执行上一个loader的一个 normal 方法，并且这个方法的source参数就是pitch方法的返回值。

完整的 loader 执行顺序如下图

![webpack-loader-执行顺序](/images/webpack-loader.png)

``` javascript
// 我们可以修改之前的代码验证一下

// patch-loader1.js
function loader(source, sourceMap) {
  console.log('normal-loader1111');
  return source + '///normal-loader1111///';
}
loader.pitch = function () {
  console.log('pitch1');
  // return '1'
}
module.exports = loader;

// patch-loader2.js
function loader(source, sourceMap) {
  console.log('normal-loader222');
  return source + '///normal-loader222///';
}
loader.pitch = function () {
  console.log('pitch2');
  return 'loader2 --- pitch2'
  // 这里返回的值，会作为下一个normal方法的source参数传入
}
module.exports = loader;

// patch-loader3.js
function loader(source, sourceMap) {
  // source - 输入的内容 要串换的内容
  console.log('normal-loader333');
  return source + '///normal-loader333///';
}
loader.pitch = function () {
  console.log('pitch3');
  // return 'loader3 --- pitch3'
  // 如果loader2的pitch方法有返回值，那这个loader3的方法都不执行了
}
module.exports = loader;
```

命令行运行 npm run build 可以看到 loader 的执行顺序为 loader1(pitch) -> loader2(pitch) -> loader1(normal)

### 不同类型 loader 的执行顺序

对单文件打包的方式的loader被称为行内（inline）loader；对于rules中的loader，webpack还定义了一个属性 enforce，可取值有 pre（为pre loader）、post（为post loader），如果没有值则为（normal loader）。所以loader在webpack中有4种:normal，inline，pre，post。

loader的叠加顺序 = post(后置) + inline(内联) + normal(正常) + pre(前置)

## loaders/#configuration——特殊配置

- !   前缀禁用配置文件中的普通loader，比如：require("!raw!./script.coffee")
- !!  前缀禁用配置文件中所有的loader，比如：require("!!raw!./script.coffee")
- -!  禁用pre loader和普通loader，但是不包括post loader，比如：require("!!raw!./script.coffee")

关于loader的禁用，webpack官方的建议是：除非从另一个loader处理生成的，一般不建议主动使用，以下下是使用场景

- pre loader 配置：图片压缩
- 普通loader 配置：coffee-script转换
- inline loader 配置：bundle loader
- post loader 配置： 代码覆盖率工具

## webpack 的 loader-runner 库的原理

``` javascript
// loaders/myLoader-runner-getPath.js
// 此文件功能：按loader的类别及loaders/#configuration配置获得要执行的loader绝对路径的数组

// loader 的4种类型：inline normal pre post
// ex：eslint 代码风格检查 一般用于前置

let path = require('path');
// 正常情况下module的查找路径（变量模拟）
let nodeModules = path.resolve(__dirname, 'node_modules');
// 模拟需要加载的文件 - 行内（inline）loader 
// 业务代码示例：import a from ''!inline-loader1!inline-loader2!./style.css'' 或者 require('!inline-loader1!inline-loader2!./style.css'u)
let request = '!inline-loader1!inline-loader2!./style.css';
// 模拟webpack loader 的配置
let rules = [
  // enforce ：配置loader类型-手动配置loader执行顺序
  { test: /\.css$/, enforce: 'pre', use:['pre-loader1', 'pre-loader2'] }, // pre 前置的
  { test: /\.css$/, use:['normal-loader1', 'normal-loader2'] }, // normal 普通的
  { test: /\.css$/, enforce: 'post', use:['post-loader1', 'post-loader2'] } // post 后置
];

// 以上模拟数据结束，一下开始原理部分

// Fn：把路径变为绝对路径
var resolveLoader = loader => path.resolve(nodeModules, loader + '.js');

// loaders/#configuration——特殊配置 相关
// 不要前置&普通的loader，只剩下inline&post
const noPreAutoLoaders = request.startsWith('-!');
// 不要普通的loader
const noAutoLoaders = noPreAutoLoaders || request.startsWith('!');
// 不要pre&post&auto，只剩下inline了
const noPrePostAutoLoaders = request.startsWith('!!');

let inlineLoaders = request.replace(/^-?!+/, '').replace(/^!!+/g, '!').split('!'); // ["inline-loader1", "inline-loader2", "./style.css"]
// 加载的资源
let resource = inlineLoaders.pop(); // "./style.css"

let preLoaders = [];
let postLoaders = [];
let normalLoaders = [];
for (let i = 0; i < rules.length; i++) {
  let rule = rules[i];
  if (rule.test.test(resource)) {
    if (rule.enforce === 'pre') {
      preLoaders.push(...rule.use);
    } else if (rule.enforce === 'post') {
      postLoaders.push(...rule.use);
    } else {
      normalLoaders.push(...rule.use);
    }
  }
}
let loaders = null;
if (noPrePostAutoLoaders) {
  loaders = [...inlineLoaders];
} else if (noPreAutoLoaders) {
  loaders = [...postLoaders, ...inlineLoaders];
} else if (noAutoLoaders) {
  loaders = [...postLoaders, ...inlineLoaders, ...preLoaders];
} else {
  loaders = [...postLoaders, ...inlineLoaders, ...normalLoaders, ...preLoaders];
}

// 把模块名转变成绝对路径
loaders = loaders.map(loader => resolveLoader(loader));
console.log(loaders);
```

``` javascript
// loaders/myLoader-runner-run.js
// 此文件功能：按顺序执行loader的pitch和normal

const fs = require('fs');
const path = require('path');
// 把loader变成对象
function createLoaderObject(loader) {
  let loaderObj = {data: {}};
  loaderObj.request = loader;
  loaderObj.normal = require(loader);
  loaderObj.pitch = loaderObj.normal.pitch;
  return loaderObj;
}

function runLoaders(options, callback) {
  // 设置loader函数中的this对象
  let loadercontext= {
    loaderIndex: 0, // 当前索引
    readResource: fs, // 读文件的模块
    resource: options.resource, // 要加载的资源 (index.js)
    loaders: options.loaders // [loader1, loader2, loader3] pathStr
  };
  let loaders = options.loaders;
  // loaders 路径Str -> 有paitch， normal，request属性的对象
  loadercontext.loaders = loaders.map(loader => createLoaderObject(loader)); // [{normal, pitch}, {normal, pitch}]
  iteratePitchLoader(loadercontext, callback);
  function processResource (loadercontext, callback) {
    // 读源文件
    let buffer = loadercontext.readResource.readFileSync(loadercontext.resource, 'utf8');
    iterateNormalLoader(loadercontext, buffer, callback);
  }
  function iterateNormalLoader(loadercontext, code, callback) {
    // 遍历loaders 的 normal 方法
    if (loadercontext.loaderIndex < 0) {
      return callback(null, code);
    }
    let curLoaderObj = loadercontext.loaders[loadercontext.loaderIndex];
    let normalFn = curLoaderObj.normal;
    let result = normalFn.call(loadercontext, code);
    loadercontext.loaderIndex--;
    iterateNormalLoader(loadercontext, result, callback);
  }
  function iteratePitchLoader(loadercontext, callback) {
    // 遍历pitch方法
    if (loadercontext.loaderIndex >= loadercontext.loaders.length) {
      // pitch 方法执行完了，读文件资源，翻转执行normal方法
      loadercontext.loaderIndex--;
      return processResource(loadercontext, callback);
    }
    let curLoaderObj = loadercontext.loaders[loadercontext.loaderIndex];
    let pitchFn = curLoaderObj.pitch;
    if (!pitchFn) {
      // 没有pitch函数，下一个pitch方法
      loadercontext.loaderIndex++;
      return iteratePitchLoader(loadercontext, callback);
    }
    let result = pitchFn.apply(loadercontext);
    // 有pitch方法
    if (result) {
      // pitch方法有返回值，要返回上一个loader的normal，且pitch的返回值作为normal方法的第一个参数，没有上一个loader的话，直接返回pitch的值
      loadercontext.loaderIndex--;
      iterateNormalLoader(loadercontext, result, callback);
    } else {
      // pitch方法没有返回值 下一个pitch
      loadercontext.loaderIndex++;
      iteratePitchLoader(loadercontext, callback);
    }
  }
}

// 以上原理部分，以下是原理的方法调用

let entry = '../src/index.js';
let options = {
  resource: path.resolve(__dirname, entry),
  loaders: [ // 这里的数据实际应该是 myLoader-runner-getPath.js 中获取到的loaders
    path.resolve(__dirname, 'patch-loader1.js'),
    path.resolve(__dirname, 'patch-loader2.js'),
    path.resolve(__dirname, 'patch-loader3.js')
  ]
}
runLoaders(options, (err, result) => {
  console.error('执行完毕');
  console.error(result);
});
```

[webpack原理代码地址点击这里](https://github.com/happychong/2019-zhufeng/tree/master/10.myWebpack/3-webpack-loader)