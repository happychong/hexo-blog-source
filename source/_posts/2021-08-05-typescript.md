---
layout: default
title: typescript
description: typescript ts
categories: [周边]
tags: [typescript]
---
# {{ page.title }}

## 1.typescript 是个啥？

- 官方：TypeScript是JavaScript类型的超集，它可以编译成纯JavaScript。
- 我的：TypeScript包含JS的语法，但是类型等方面更严格，可以根据代码设置的类型等进行提示。

## 2.基础准备

### 安装

```bash
# 全局
npm install -g typescript

# 本地
npm install -D typescript
```

### 编译

```bash
# 命令方式
tsc helloworld.ts
```

```json
// package.json 脚本配置
{
  "scripts": {
    "tsc:build": "tsc",
    "tsc:watch": "tsc --watch"
  }
}
```

```bash
# Visual Studio Code - VSCode 编辑器 方式
英文：Terminal -> Run Task -> tsc:build - tsconfig.json
中文：终端 -> 运行任务 -> 选择就好了（虽然我的报错，about_Execution_Policies-执行策略的问题）
```

### npm 项目应用 typescript
```bash
# 首先 以下代码创建 package.json
# cnpm init -y

# 创建 typescript 配置文件 tsconfig.json
tsc --init
```

### 配置项解析（tsconfig.json）

```json
{
  /* 要把TS语言编译成es5语法 'ES3' (default), 'ES5', 'ES2015', 'ES2016', 'ES2017', 'ES2018', 'ES2019', 'ES2020', 'ES2021', or 'ESNEXT'. */
  "target": "es5",
  "module": "commonjs",  /* 模块形式 */
  "esModuleInterop": true,  /* commonjs和es6模块转译，包导出为commonjs方式，导入时可以用es6方式导入 */
  "strictNullChecks": false, /* 把 null 赋值给变量的时候，是否校验报错 */
}
```

### namespace & 模块（module）小技巧

```typescript
let name: string = 'a'; // 报错:无法重新声明块范围变量“name”。，因为生命name到全局globle了
export {} // 加了这句，上面代码不报错了，因为当前文件已经是一个模块了，会在当前的命名空间下
```

## 3.基础知识讲解

### 基本数据类型

```typescript
// 变量声明
let age: number = 9; // number
let name: string = 'a'; // string
let married: boolean = true; // boolean
let hobbies: string[] = ['sing', 'dance']; // Array : 每一项都是string
let goodAt: Array<string> = ['sing']; // 同上

// 元组: 类似数组，但是长度和类型都固定
// 特点： 1-长度固定   2-类型可以不一样
let point: [number, number] = [100, 100]; // 描述的是屏幕上的一个点
let person: [string, number] = ['Jack', 10]; // 描述的事一个人

// 枚举: 列出一个事物的所有类型
enum Gender {
  BOY, // 不设置指默认0开始递增
  GIRL
}
let s = 'M'
enum AnimalGender {
  // 属性值只可以是基本类型，也不能是计算表达式
  // N = true, // 报错：含字符串值成员的枚举中不允许使用计算值。
  // Y = s, // 报错：含字符串值成员的枚举中不允许使用计算值。
  M = 'M', 
  FM = 'FM'
}
console.log(`I'm a ${ Gender.GIRL }`);
enum Week {
  MONDAY = 1, // 设置值会按设置规则来
  TUESDAY = 2,
  SUNDAY = 7
}
console.log(`SUNDAY is ${ Week.SUNDAY }`);
// 常数枚举： 编译后找不到TrafficColors的定义，只剩下下面console这句了
const enum TrafficColors {
  Red,
  Yellow,
  Green
}
console.log(`TrafficColors Red is ${ TrafficColors.Red } `);

// 任意类型 any : 
// 实际自己写尽量不用any，除非数据结构太复杂太灵活
// 应用场景：第三方库没有类型定义、类型转换
// let root:HTMLElement | null = document.getElementById('root'); // 需要在浏览器执行才有document，所以先注释掉
// root!.style.color = 'red'; // ! 非空断言，root可能为null，不能给null赋值style属性，加！= 我能确定root是HTMLElement

// null - 空 undefined - 未定义 都是其他类型的子类型，所以可以把他们赋值给其他类型的变量，但是配置要关闭strictNullChecks
let string1: string = null;
let string2: string = undefined;
let x: number;
x = 1;
x = null;
x= undefined;

// void 空-没有
function greeting(name: string): void {
  console.log(`hello ${ name }`);
  // return 1; // 报错：不能将类型“number”分配给类型“void”。 - 因为上面定义了void，不能有返回值类型
  return null; // 不报错：因为null是任何类型的子类型
}
greeting('Marry')

// never 永远不：是其他类型的子类型，代表不会出现的值
function createErr(message: string): never {
  // never报错：返回“从不”的函数不能具有可访问的终结点。
    // 意思1:就是方法内不可以正常报错，要抛出错误才行
    // 意思2：死循环，永远不会执行到结束，下面sum例子
  console.log(1);
  throw new Error('error'); // 有了这句，never不报错了  
}
function sum():never {
  while(true) {
    console.log('hello sum');
  }
  console.log(2); // 检测到无法访问的代码。 - 这里永远不会执行，编辑器中会灰掉，这样情况never才不报错
}

// 类型推论
let no = 2;
no = 7;
// no = '4'; // 报错，因为定义赋值的时候，赋值2，已经预定no的类型是number了
let not ; // 定义的时候不设置类型，默认any
not = 4;
not = '4';

// 包装对象：自动在基本类型和对象类型之间切换 java中叫装箱和拆箱
// 1.基本类型上没有方法
// 2.在内部迅速的完成一个装箱的操作，把基本类型包装成对象类型，然后用对象来调方法
let nameObj: string = 'zf';
nameObj.toLocaleLowerCase(); // 相当于：let tempNameObj = new String(nameObj);     tempNameObj.toLocaleLowerCase();
let isOk: boolean = true;
let isOk2: boolean = Boolean(1); // Boolean()会返回基本类型的布尔值，所以ok
// let isOk3: boolean = new Boolean(1); // new Boolean() 返回的是布尔类型对象，所以报错

// 联合类型
let type: string | number;
type.toString(); // 这里只可以调用string和number都有的方法
type = 5;
type.toString(); // 这里确定了type的类型，可以调用number的所有方法
type = '5';
type.toString(); // 这里确定type是string类型，所以可以调佣string的所有方法

// 类型断言 - 把一个变量断言成具体类型处理
let type2: string | number;
// 这里只可以调用string和number的公用方法，但是想调用string的方法，怎么办呢，如下断言
(type2 as string).toLocaleLowerCase(); // 把type当做字符串处理，可以调用string的方法
(type2 as number).toFixed(2); // 把type当做字符串处理，可以调用string的方法
type2 = '3';
// (type2 as number).toFixed(2); // 报错，给type2赋值string后，就不能再 as nuber 了

// 字面量类型
let parentRole: 'Mum' | 'Dad'; // parentRole 只可以被赋值为 Mum 或 Dad
parentRole = 'Mum';
parentRole = 'Dad';
// parentRole = 'Child'; // 报错：不能将类型“"Child"”分配给类型“"Mum" | "Dad"”。

export {}
```

### 函数

```typescript
// 函数定义
function hello(name: string): void {
  // void - 返回值类型
  console.log('hello ' + name);
}
// 函数表达式
// type 定义一个类型或者类型别名       => 跟箭头函数没关系,后面内容定义的是返回值格式, 没有返回值用void
type GetUserNameType = (firstName: string, lastName: string) => string;
// GetUserNameType 的函数返回值为字符串   return 'aa'
type GetUserNameType2 = (firstName: string, lastName: string) => {
  name: string
};
// GetUserNameType2 的函数返回值为包含一个name属性，且属性值为string类型的对象  return { name: 'aa' }
let getUserName: GetUserNameType = function(firstName: string, lastName: string): string {
  return firstName + ' ' + lastName;
}

// 可选参数 用?
function print(name: string, age?: number, home?: string) {
}
print('jack'); // 不报错

// 默认参数
function ajax(url: string, method: string = 'GET') {
}
ajax('/user')

// 剩余参数
function sum(...numbers: Array<number>) {
  return numbers.reduce((pre, item) => pre + item, 0);
}

// 函数重载 - 限定参数的形式
let obj: any = {};
function attr(val: string) : void; // 相当于给attr函数增加了重载，接受string
function attr(val: number) : void; // 相当于给attr函数增加了重载，接受number
// console.log('a'); // 报错，因为重载的定义和函数生命中间不能有其他代码，要紧挨着
function attr(val: any): void {
  if (typeof val === 'string') {
    obj.name = val;
  } else if (typeof val === 'number') {
    obj.age = val;
  }
}
attr('jack');
attr(10);
// attr(true); // 定义了重载，报错

function sum2 (a: number, b: number): void;
function sum2 (a: string, b: string): void;
function sum2 (a: any, b:any): void {
  return a + b
}
sum2(1,2);
sum2('a', 'b');
// sum2(1, 'b'); // 报错

// TS中写箭头函数 - 与JS相同
let delay = (ms: number) => {
  setTimeout(function() {

  }, ms)
}

export {}
```

### 类-class

```typescript
// 定义类
class Person {
  name: string;
  age: number;
}
// 3.X版本的时候，属性名要初始化，如下写法
class Person3 {
  name: string = 'jack';
  age: number;
  constructor() {
    this.age = 10;
  }
}
let p1 = new Person();
console.log(p1);
let p3 = new Person3();
console.log(p3);


namespace b {
// 存取器 getter setter
class Person {
  myName: string;
  constructor(name: string) {
    this.myName = name;
  }
  get name() {
    return this.myName
  }
  set name(val: string) {
    this.myName = val.toUpperCase();
  }
}
let p = new Person('jack');
console.log(p.name);
p.name = 'Mary'; // 执行的set name() 
console.log(p.name);

}


namespace c {
  // 参数属性 - public
  class Person {
    // name: string;
    constructor(public name: string) {
    }
  }
  let p = new Person('jack');
  p.name; // 加了 public ，不报错，public 把name属性定义到了类上,等同于name: string;
}

namespace c {
  // readonly
  class Person {
    // name: string;
    constructor(public readonly name: string) {
    }
  }
  let p = new Person('jack');
  // p.name = 'aa'; // 报错，不能赋值，因为readonly
}

namespace d {
  // 继承
  class Person {
    name: string;
    age: number;
    constructor(name: string, age: number) {
      this.name = name;
      this.age = age;
    }
    getName() {
      return this.name;
    }
    setName(newName: string) {
      this.name = newName;
    }
  }
  class Student extends Person {
    stuNo: number;
    constructor(name: string, age: number, stuNo: number) {
      super(name, age); // super - 父类函数
      this.stuNo = stuNo;
    }
    getStuNo() {
      return this.stuNo;
    }
    setStuNo(newStuNo: number) {
      this.stuNo = newStuNo;
    }
  }
  let s = new Student('Ali', 4, 9);
  console.log(s.getName()); // 继承到了 Person 的 getName

}


namespace e {
  // 修饰符
  // public - 公开（自己和自己的子类都可以访问）
  // protected - 受保护的（自己和自己的子类能访问，其他类不能访问）
  // private - 私有（只能自己访问，子类也不能访问）
  class Person {
    public name: string;
    protected age: number;
    private password: number;
    constructor(name: string, age: number) {
      // 自己，都可以访问
      this.name = name;
      this.age = age;
      this.password = 111;
    }
    getName() {
      return this.name;
    }
    setName(newName: string) {
      this.name = newName;
    }
  }
  class Student extends Person {
    // 静态属性 静态方法 static
    // 静态 - 不需要实例化，类上本身的属性和方法
    static type = 'cn'
    stuNo: number;
    constructor(name: string, age: number, stuNo: number) {
      super(name, age); // super - 父类函数
      this.stuNo = stuNo;
    }
    static getType() {
      return Student.type
    }
    getStuNo() {
      return this.name + this.age + this.stuNo;
      // protected age 可以访问, private paasword 不能访问
    }
    setStuNo(newStuNo: number) {
      this.stuNo = newStuNo;
    }
  }
  let s = new Student('jack', 10, 123);
  console.log(s.name);
  // console.log(s.age); // 报错：protected
  // console.log(s.paasword); // 报错：private

  console.log(Student.type); // 静态属性
  console.log(Student.getType()); // 静态方法 
}

```

### 装饰器

#### 1-类装饰器

```typescript
namespace a {
  // tsconfig中需要放开"experimentalDecorators": true,
  // 1-类装饰器 ： 在类声明致歉生命，用来监视、修改或替换类定义,使用场景：别人写的类，想增强这个类，又不想改动别人的代码，可以写个类装饰器 ex：@more
  interface Person {
    xx: string;
    yy: string
  }
  function more(target: any) {
    // target - 装饰类的时候，指向Person定义
    target.prototype.xx = 'xx';
  }
  @more // 名字随意取
  class Person {

  }
  let p = new Person();
  console.log(p.xx);
}

namespace b {
  interface Person {
    age: number;
  }
  function addAge(target: any) {
    // 这里直接重写了类，person已经不是person了
    // 但是Person里面有的属性，这里也必须要有，只能是增强的作用，不能减少，不能完全不同
    // return class {
    //   age: number = 10;
    //   firstName: string = 'Chinese';
    // }
    // 以上不想重复定义firstName，可以返回继承的类
    return class extends Person {
      constructor() {
        super();
      }
      age: number = 10;
    }
  }
  @addAge // 名字随意取
  class Person {
    firstName: string = 'person';
  }
  let p = new Person();
  console.log('---', p.age, '---');
  console.log(p.firstName)
  
}

namespace c {
  // 装饰器工厂 - 装饰器函数返回真正的装饰器  @addAge('Mary')
  interface Person {
    lastName: string;
  }
  function addAge(name: string) {
    return function (target: any) { // 这里才是真正的装饰器
      return class extends Person {
        lastName: string = name;
        constructor() {
          super();
        }
        getName() {
          return this.firstName + ' ' + this.lastName
        }
      }
    }
  }
  
  @addAge('Mary')
  class Person {
    firstName: string = 'person';
  }
  let p = new Person();
  console.log(p.firstName)
  console.log(p.lastName);
}

```

#### 2-属性装饰器

```typescript
namespace d {
  // 目的：赋值就变大写
  function upperCase(target:any, propertyName: string) {
    // target - 如果装饰的事普通属性(name)，target 指向类的原型(Person.prototype)
    // target - 如果装饰的事类的属性（static），那么target 指向类的定义
    let value = target[propertyName]; // jack
    const getter = () => value;
    const setter = (newVal: string) => {
      value = newVal.toUpperCase();
    }
    Reflect.deleteProperty(target, propertyName);
    Object.defineProperty(target, propertyName, {
      get: getter,
      set: setter,
      enumerable: true, // 是否可枚举
      configurable: true, // 是否可配置
      // writable: true, // 是否可写
    })
  }
  function methodEnumerable(flag: boolean) {
    // 描述方法 有3个参数
    return function (target:any, methodKey:string, propertyDescriptor:PropertyDescriptor): void {
      // target : 原型 | 类
      // methodKey ： 方法名
      // descriptor：属性描述器
      propertyDescriptor.enumerable = flag;
    }
  }
  function propEnumerable(val: number) {
    return function(target:any, propertyName: string) {
      let value = target[propertyName]; // jack
      const getter = () => value;
      const setter = (newVal: string) => {
        value = newVal;
      }
      Reflect.deleteProperty(target, propertyName);
      Object.defineProperty(target, propertyName, {
        get: getter,
        set: setter,
        enumerable: false, // 是否可枚举
        configurable: true, // 是否可配置
        // writable: true, // 是否可写
      })
    }
  }
  function setAddr(addr: string) {
    // 描述方法 有3个参数
    return function (target:any, methodKey:string, propertyDescriptor:PropertyDescriptor): void {
      // target - 装饰的是类的属性（static 静态方法），那么target 指向类的定义
      target.addr = addr;
    }
  }
  function sunForNumber(target:any, methodKey:string, propertyDescriptor:PropertyDescriptor) {
    let oldMethod = propertyDescriptor.value; // sum方法
    propertyDescriptor.value = function (...args: any[]) {
      args = args.map(item => parseFloat(item));
      return oldMethod.apply(this, args)
    }
  }
  class Person {
    static addr: string;
    @upperCase
    name: string = 'jack';
    
    @propEnumerable(6) // 装饰属性，默认可枚举，改成不可枚举
    age: number = 1;
    
    @methodEnumerable(true)  // 装饰方法，默认不可枚举，改成可枚举
    getName() {
      console.log('getName');
    }

    @setAddr('No 100')  // 装饰!静态!方法
    static getAddr() {
    }
    @sunForNumber
    sum(...args: any[]) {
      return args.reduce((pre, item) => pre + item, 0);
    }
  }
  let p = new Person();
  console.log('upperCase：name被大写了', p.name);
  console.log(p.age);
  for (let attr in p) {
    console.log('propEnumerable & methodEnumerable  设置为没有age，却有getName了--attr ==== ', attr);
  }
  console.log('setAddr：装饰器给静态addr赋值======', Person.addr);
  console.log('sum：sunForNumber: ', p.sum(1, '2', 3));
  
}
```

#### 3-参数装饰器

```typescript
namespace d {
  interface Person {
    mixPass: string
  }
  function toMix(target:any, methodKey:string, paramsIndex: Number) {
    // target : Person - prototype
    console.log(target, methodKey, paramsIndex);
    target.mixPass = '123';
  }
  class Person {
    login(username: string, @toMix password:string) {
      console.log('username : password : mixPass === ', username, password, this.mixPass);
    }
  }
  let p = new Person();
  p.login('aa', '11');
}
```

#### 4-装饰器的执行顺序

```typescrip
  // @Class1()
  // @Class2()
  // class Person {
  //   @propName('name')
  //   name: string = 'name';
  //   @propAge('age')
  //   age: number = 19;
  //   @methodGreet()
  //   greet( @Param1() p1: string, @Param2() p2: string) {
  //   }
  // }

  /**
   * 总体：：：属性 -> 参数 -> 方法  -> 类
   * 具体第一步：属性 | 方法  ， 谁在前谁先
   * 具体第二步：方法（参数 + 方法），参数在前，方法在后
   * 具体最后一步：类
   * 同一类型的，位置挨着定义内容最近的先执行，ex类的：先@Class2()，再@Class1()
   * @propName('name')
   * @propAge('age')
   * @Param2()
   * @Param1()
   * @methodGreet()
   * @Class2()
   * @Class1()
   */
```


 