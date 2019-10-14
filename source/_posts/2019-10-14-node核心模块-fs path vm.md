---
layout: default
title: node核心模块常用语法与功能 - fs、path
description: fs-文件读写，path-处理路径, vm-虚拟机模块可以执行代码
categories: [node]
tags: [fs, path, vm, node]
---
# {{ page.title }}

## 1 fs - 文件的读写模块

```javascript
// 导入 fs
const fs = require('fs');

//判断文件是否存在，如果不存在抛出异常
fs.accessSync('c.js');


```

## 2 path - 专门处理路径的模块
```javascript
// 导入 path
const path = require('path');

// __dirname ：表示当前目录名

// 以当前根路径下查找a.js的路径
path.resolve('./a.js');
// 当前目录下 拼接路径或文件
path.resolve(__dirname, './a.js'); // XXX/a.js
path.resolve(__dirname, 'a', 'b'); // XXX/a/b
path.resolve(__dirname, 'a', 'b', '/'); // 回到了/下，所以拼接/的时候，只能用join方法

// 简单的以/拼接路径 - 只拼接a和b的
path.join('a', 'b'); // a/b
path.join('a', 'b', '/'); // a/b/

// 取扩展名
path.extname('main.js'); // .js

// 取基本名（主名）,需要第二个参数提供扩展名才能得到basename
path.basename('main.js', '.js'); // .main

// 获取父路径
path.dirname(__dirname); // 得到当前目录的上级目录

```

## 3 vm - 虚拟机模块，执行字符串代码
让字符串执行的3个方法
1. new Function    
2. eval - 有作用域问题
3. vm - vm 可以提供沙箱环境，不受外界变量的干扰

```javascript
// 导入 vm()
const vm = require('vm');

// 在当前上下文中执行
let b = 1000;
vm.runInThisContext(`console.log(b)`);
// 会报错，因runInThisContext的参数字符串中，没有b变量

```