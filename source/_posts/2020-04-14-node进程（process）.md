---
layout: default
title: Node 的 进程
description: process
categories: [Node]
tags: [process]
---
# {{ page.title }}

## 1 进程 & 线程

- 进程（process）是计算机中的程序，是系统进行资源分配和调度的基本单位，比如qq就是一个进程
- 线程（Thread）是操作系统能够进行运算调度的最小单位，被包含在进程之中，是进程中的实际运作单位，比如qq中可以做多个事

## 2 Node中的进程和线程

主线程是单线程的：一个进程只开一个主线程，基于事件驱动的、异步非阻塞I/O，可以应用与高并发场景，Nodejs中没有多线程，为了充分利用多核cpu，可以使用子进程实现内核的负载均衡，不适合cup密集型，会导致其他请求会等待处理完成后再进行处理

## 3 项目中我们要解决的问题

- Node.js 做耗时的计算时候阻塞问题
- Node.js 如何开启多进程
- 开发过程中如何实现进程守护

## 4 创建子进程

### 进程间 pipe 方式通信

pipe 通信方式要监听事件，不方便

```javascript
// 2.spawn.js

let path = require('path');
// child_process 子进程模块，核心模块
let { spawn } = require('child_process'); // spawn: 产卵
// 创建子进程，执行test下的a.js
let cp = spawn('node', ['a.js'], {
  cwd: path.resolve(__dirname, 'test'),
  // stdio: 'ignore' // 忽略子进程中的输出
    // process.stdin:标准输入（0）   process.stdout：标准输出（1）    process.stderr：错误输出（2）     stdio: [0, 1, 2](默认)
  // stdio: [process.stdin, process.stdout, process.stderr]  // 子进程共享父进程中的输入输出  这样的方式只能打印一些日志
  stdio: 'pipe' // 子进程和父进程都是通过pipe通信    'pipe' = ['pipe', 'pipe', 'pipe']
});
cp.stdout.on('data', function (data) {
  // 这里的stdout，要和子进程中的stdout对应上，还可以是stdin | stderr
  console.log(data + '');
});
cp.on('error', function (err) {
  console.log(err);
});
cp.on('close', function() {
  console.log('close');
});
cp.on('exit', function() {
  console.log('exit');
});
```

```javascript
// /test/a.js

let total = 0;
for (let i = 0; i < 100000; i++) {
  total += i;
}
// console.log(total);
process.stdout.write(total + ''); // write的参数只能是 String or Bugger
```

### 进程间 ipc 方式通信

```javascript
// 3.ipc.js

let path = require('path');
// child_process 子进程模块，核心模块
let { spawn } = require('child_process'); // spawn: 产卵
// 创建子进程，执行test下的a.js
let cp = spawn('node', ['b.js'], {
  cwd: path.resolve(__dirname, 'test'),
  stdio: [0, 1, 2, 'ipc'] // ipc: 通过 message 和 send 方法进行通信，类似webworder   这里的设置不能只包括ipc
});
cp.on('message', function(data) {
  console.log(data);
});
// 主进程给子进程发送消息
cp.send('我是主进程send的消息');
// cp.kill(); // 杀掉子进程，但是杀掉监听message就拿不到消息了，所以注释掉
// 子进程要听从父进程，父进程kill掉，子进程会自动关闭
```

```javascript
// /test/b.js

let total = 0;
for (let i = 0; i < 100000; i++) {
  total += i;
}
process.send(total);
process.on('message', function(data) {
  console.log('child log：', data, process.pid);
  // 接收到消息后，子进程退出
  process.exit();
});
```

### 解除主子进程关系的 ipc 通信

```javascript
// 4.ipc.js

let path = require('path');
let { spawn } = require('child_process');
// ex：爬虫 不定期的从别的网站上获取数据，再展示给客户端，，可以一直启动着，主子之间没有关系，主线程挂了，子进程可以接着爬
let cp = spawn('node', ['c.js'], {
  cwd: path.resolve(__dirname, 'test'),
  stdio: 'ignore', // 忽略子进程中的输出， 主子之间不传递消息了
  detached: true // detached：独立的
});
cp.unref(); // 父进程放弃子进程控制，让子进程自己跑
console.log(cp.pid); // 获取子进程的id号

```

```javascript
// /test/c.js

let fs = require('fs');
let total = 0;
for (let i = 0; i < 100000; i++) {
  total += i;
}
setInterval(() => {
  fs.appendFileSync('1.txt', 'zf');
}, 1000)
```

## 5 httpServer 中应用子进程 示例

### ipc 方式示例

```javascript
// 1.server.js
const http = require('http');
const { spawn } = require('child_process');
const path = require('path');
http.createServer((req, res) => {
  if (req.url === '/sum') {
    // 如果有个/sum请求，代码在这里计算，之后又有个其他接口请求，代码会/sum计算完成后，再处理其他接口，单线程阻塞了
    // let total = 0;
    // for (let i = 0; i < 10000000000; i++) {
    //   total += i;
    // }
    // res.end(total);

    // 开启子进程单独处理计算问题，不阻塞其他接口请求的处理
    let cp = spawn('node', ['sum.js'], {
      cwd: path.resolve(__dirname, 'test'),
      stdio: [0, 1, 2, 'ipc'] // 内容少可以用 ipc 方式，内容多可以用pipe
    });
    cp.on('message', function(data) {
      res.end('tatal:' + data);
    });
  } else {
    res.end('end ok');
  }
}).listen(3000);
```

```javascript
// sum.js

let total = 0;
for (let i = 0; i < 10000000000; i++) {
  total += i;
}
process.send(total);
```

### fork 方式示例

```javascript
// 5.fork.js

const http = require('http');
const { fork } = require('child_process'); // fork:复制，克隆
// fork 就是封装了 spawn，可以少设置一些参数，使用方便一些，如果只执行某个文件，可以直接用fork
const path = require('path');
http.createServer((req, res) => {
  if (req.url === '/sum') {
    // fork默认会加node属性
    let cp = fork('sum.js', {
      cwd: path.resolve(__dirname, 'test')
    });
    cp.on('message', function(data) {
      res.end('tatal:' + data);
    });
  } else {
    res.end('end ok');
  }
}).listen(3000);
```

## 其他子进程方式

```javascript
const { fork, execFile, exec } = require('child_process');

// spawn 可以应用pipe读取大文件
// fork 可以使用ipc   - 如果执行node脚本，而且需要获取内容，可以使用fork
// execFile 功能是执行命令 核心也基于spawn，  把结果都整合好，触发回调函数，默认数据最大200k 
//  - 如果执行命令，用exec
execFile('ls', ['-ll'], (err, stdout, stderr) => {
  // 读取文件夹信息
  console.log(stdout);
});

// 与 execFile 相比，命令可以写一个字符串里了  默认exec会启动一个shell窗口执行命令
exec('ls -ll', (err, stdout, stderr) => {
  console.log(stdout);
})


```
