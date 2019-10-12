---
layout: default
title: vue nextTick原理
description: vue nextTick
categories: [vue]
tags: [vue, nextTick]
---
# {{ page.title }}

vue中的视图更新有缓存机制，会把多次数据的修改集中在一次视图更新中来更新视图，我们想视图更新之后再获取数据，那么就可用vue的nextTick，nextTick中的代码可以在下一队列中执行，实现代码的延迟执行，多次调用nextTick，会将其中的方法放入一个队列中，视图更新的时候会调用flushCallbacks方法，这个方法会执行所有nextTick中的方法。至于下一队列的执行时机，会按以下列表优先级进行选择。

- prosise.then (微任务)
- MutationObserver (微任务，非IE，h5方法)
- setImmediate (宏任务，IE)
- setTimeout (宏任务)