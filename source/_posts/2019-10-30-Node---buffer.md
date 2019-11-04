---
layout: default
title: Node --- buffer
description: buffer - 流
categories: [Node]
tags: [buffer]
---
# {{ page.title }}

## 一：简述

Buffer 类是作为 Node.js API 的一部分引入的，用于在 TCP 流、文件系统操作、以及其他上下文中与八位字节流进行交互。

Buffer 类的实例类似于从 0 到 255 之间的整数数组（其他整数会通过 ＆ 255 操作强制转换到此范围）

Buffer 的大小在创建时确定，且无法更改。

Buffer 类在全局作用域中，因此无需使用 require('buffer').Buffer。

Buffer的实例存放的是内存地址。

## 二：buffer的常用方法

### 1.创建

```javascript
// 创建一个长度为 10、且用零填充的 Buffer。
const buf1 = Buffer.alloc(10);

// 创建一个长度为 10、且用 0x1 填充的 Buffer。 
const buf2 = Buffer.alloc(10, 1);

// 创建一个长度为 10、且未初始化的 Buffer。
// 这个方法比调用 Buffer.alloc() 更快，
// 但返回的 Buffer 实例可能包含旧数据，
// 因此需要使用 fill() 或 write() 重写。
const buf3 = Buffer.allocUnsafe(10);

// 创建一个包含 [0x1, 0x2, 0x3] 的 Buffer。
const buf4 = Buffer.from([1, 2, 3]);

// 创建一个包含 UTF-8 字节 [0x74, 0xc3, 0xa9, 0x73, 0x74] 的 Buffer。
const buf5 = Buffer.from('tést');

// 创建一个包含 Latin-1 字节 [0x74, 0xe9, 0x73, 0x74] 的 Buffer。
const buf6 = Buffer.from('tést', 'latin1');
```

### 2.写入

#### 2.1 fill

用指定的 value 填充 buf。 如果没有指定 offset 与 end，则填充整个 buf

语法：buf.fill(value[, offset[, end]][, encoding])

- value : 用来填充 buf 的值
- offset : 开始填充 buf 的偏移量。默认值: 0
- end : 结束填充 buf 的偏移量（不包含）。默认值: buf.length
- encoding : 如果 value 是字符串，则指定 value 的字符编码。默认值: 'utf8'
- return : 返回 buf 的引用

```javascript
const buf = Buffer.allocUnsafe(5)
buf.fill('h');
console.log(buf.toString());
// 打印: hhhhh
```

#### 2.2 write

buf.write(string[, offset[, length]][, encoding])

- string : 要写入 buf 的字符串
- offset : 开始写入 string 之前要跳过的字节数。默认值: 0
- length : 要写入的字节数。默认值: buf.length - offset
- encoding : string 的字符编码。默认值: 'utf8'
- return : 返回 已写入的字节数

```javascript
const buf = Buffer.alloc(10);
console.log(buf.length); // 10
const len = buf.write('abcdef', 0);
console.log(len); // 6
console.log(buf.toString()); // abcdef
```


### 3.slice - 拷贝

```javascript
let buffer = Buffer.from('hello world');
let newBuffer = buffer.slice(0);
newBuffer[0] = 100;
// 此时，buffer的第1项也会被修改，因为buffer存放的是内存地址
```

### 4.isBuffer - 判断是否是buffer类型

```javascript
let buffer = Buffer.from('hello world');
Buffer.isBuffer(buffer) // true
```

### 5.copy - 拷贝

```javascript
let buffer1 = Buffer.from('你');
let buffer2 = Buffer.from('好');
let buff = Buffer.alloc(6); // 创建可以放下6个字符大小的Buffer实例
buffer1.copy(buff, 0, 0, 3);
buffer2.copy(buff, 3, 0, 3);
    // buffer2 : 源buffer
    // buff : 目标buffer
    // 3 : 目标的开始位置
    // 0 : 源的开始拷贝位置
    // 3 : 源的结束拷贝位置
console.log(buff.toString()); // 你好
```

copy原理如下

```javascript
Buffer.prototype.copy = function (targetBuffer, targetStart, sourceStart=0, sourceEnd=this.length) {
    for (let i = 0; i < sourceEnd - sourceStart; i++) {
        targetBuffer[targetStart + i] = this[sourceStart + i];
    }
}
```

### 6.concat - 拼接

```javascript
// concat - 类上的方法
let buffer1 = Buffer.from('你');
let buffer2 = Buffer.from('好');
let buff = Buffer.concat([buffer1, buffer2]); // 你好
let buff2 = Buffer.concat([buffer1, buffer2], 3); // 你
    // Buffer.concat(list[, totalLength])
    // list : 要合并的 Buffer 数组
    // totalLength : 合并后 list 中的 Buffer 实例的总长度
    // return : 返回一个合并了 list 中所有 Buffer 实例的新 Buffer
```

concat原理

```javascript
Buffer.concat = function (list, totalLength = list.reduce((pre, cur) => pre+cur.length, 0)) {
    let newBuf = Buffer.alloc(totalLength);
    let index = 0;
    for (let buf of list) {
        if (index >= newBuf.length) {
            break;
        }
        buf.copy(newBuf, index);
        index += buf.length;
    }
    return newBuf
}
```

### 7.indexOf or lastIndexOf - 取索引

buf.indexOf(value[, byteOffset][, encoding])

- value : 要查找的值
- byteOffset : 中开始查找的偏移量。默认值: 0
- encoding : 如果 value 是字符串，则指定 value 的字符编码。默认值: 'utf8'
- return : 返回 buf 中首次出现 value 的索引，如果 buf 没包含 value 则返回 -1

buf.lastIndexOf(value[, byteOffset][, encoding])

lastIndexOf 与 indexOf 不同的是：lastIndexOf 会从buf最后的位置往前查找，但是返回的索引值仍然是从前往后的位置

```javascript
const buf = Buffer.from('你好你好吗');
console.log(buf.indexOf('你')); // 0
console.log(buf.indexOf('你', 2)); // 6
```

## 三：buffer的扩展

### 1 split - 分割

```javascript
Buffer.prototype.split = function (sep) {
    // 分隔符的长度
    sep = Buffer.from(sep);
    let len = sep.length;
    let offset = 0;
    let result = [];
    let current;
    while ((current = this.indexOf(sep, offset)) > -1) {
        result.push(this.slice(offset, current));
        offset = current + len;
    }
    result.push(this.slice(offset));
    return result
}

const buf = Buffer.from(`你1
你好2
你好你3
你好你好4`);

for (let item of buf.split('\n')) {
    console.log(item.toString());
}
// 你1
// 你好2
// 你好你3
// 你好你好4
```
