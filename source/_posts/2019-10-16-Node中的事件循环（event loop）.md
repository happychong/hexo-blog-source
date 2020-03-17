---
layout: default
title: Node中的事件轮询（event loop）
description: Node中的EventLoop
categories: [Node]
tags: ['EventLoop', 'event loop']
---
# {{ page.title }}

## 前言

Node中的事件循环和js在浏览器中的事件循环基本一致。

Node.js启动的时候，它会初始化Event Loop, 处理提供的输入脚本, 这可能会使异步API调用，调用timers，或者调用process.nextTick, 然后开始处理事件循环。
下图简单展示了事件循环的操作顺序:

![Node-event loop](/images/nodeEventLoop.png)

---
* 新版本node中的事件环和浏览器中的基本一样

* 浏览器中的事件循环，请看 [这里](http://happychong.github.io/2019/10/12/%E6%B5%8F%E8%A7%88%E5%99%A8%E4%B8%ADjs%E7%9A%84%E4%BA%8B%E4%BB%B6%E5%BE%AA%E7%8E%AF%EF%BC%88event%20loop%EF%BC%89/)

## setTimeout vs setImmediate
setTimeout 和 setImmediate 哪个回调方法先执行，由node的启动速度和当前代码的上下文有关，如下例子

```javascript
setTimeout(() => {
    console.log('timeout');
})
setImmediate(() => {
    console.log('setImmediate');
})
// 这种情况
// 情况1：如果node启动的快，就会先执行timeout，因为当第一次进入event loop轮询阶段的时候，timer里面已经有了timeout的回调，所以直接执行了，再进入poll阶段
// 情况2：如果node启动的慢，第一次进入event loop轮询的时候，timeout还没有被放入timer队列，直接进入了poll阶段，清空poll的时候，继续执行check阶段的setImmediate回调，所以就会先执行setImmediate，再执行timeout
```

```javascript
let fs = require('fs');
fs.readFile('./a.txt', function () {
    setTimeout(() => {
        console.log('timeout');
    })
    setImmediate(() => {
        console.log('setImmediate');
    })
})
// 会先执行setImmediate，再执行timeout，setImmediate如果在I/O之后，会立即执行
// fs.readFile的回调在poll阶段，poll清空后，会执行check阶段的setImmediate回调，然后重新到timer阶段
```