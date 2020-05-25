---
title: es6
tags:
  - ES6
categories:
  - front-end
date: 2018-12-19 16:13:42
---

## ES6

### Babel

是一个广泛使用的 ES6 转码器，可以将 ES6 代码转为 ES5 代码

Babel 的配置文件是`.babelrc`，存放在项目的根目录下。使用 Babel 的第一步，就是配置这个文件。Babel 使用`babel-cli`工具，用于命令行转码。

```js
// 转码前
input.map((item) => item + 1);

// 转码后
input.map(function (item) {
  return item + 1;
});
```

### let 和 const

```js
let 局部作用域，var全局作用域，const静态变量
直接使用未声明，输出为：undefined
```

### 变量解析

变量的解构赋值用途很多。

交换变量的值

```js
let x = 1;
let y = 2;

[x, y] = [y, x];
```

上面代码交换变量`x`和`y`的值，这样的写法不仅简洁，而且易读，语义非常清晰。

从函数返回多个值

函数只能返回一个值，如果要返回多个值，只能将它们放在数组或对象里返回。有了解构赋值，取出这些值就非常方便。

```js
// 返回一个数组

function example() {
  return [1, 2, 3];
}
let [a, b, c] = example();

// 返回一个对象

function example() {
  return {
    foo: 1,
    bar: 2,
  };
}
let { foo, bar } = example();
```

函数参数的定义

解构赋值可以方便地将一组参数与变量名对应起来。

```js
// 参数是一组有次序的值
function f([x, y, z]) { ... }
f([1, 2, 3]);

// 参数是一组无次序的值
function f({x, y, z}) { ... }
f({z: 3, y: 2, x: 1});
```

提取 JSON 数据

解构赋值对提取 JSON 对象中的数据，尤其有用。

```js
let jsonData = {
  id: 42,
  status: "OK",
  data: [867, 5309],
};

let { id, status, data: number } = jsonData;

console.log(id, status, number);
// 42, "OK", [867, 5309]
```

上面代码可以快速提取 JSON 数据的值。

函数参数的默认值

```js
jQuery.ajax = function (
  url,
  {
    async = true,
    beforeSend = function () {},
    cache = true,
    complete = function () {},
    crossDomain = false,
    global = true,
    // ... more config
  } = {}
) {
  // ... do stuff
};
```

指定参数的默认值，就避免了在函数体内部再写`var foo = config.foo || 'default foo';`这样的语句。

遍历 Map 结构

任何部署了 Iterator 接口的对象，都可以用`for...of`循环遍历。Map 结构原生支持 Iterator 接口，配合变量的解构赋值，获取键名和键值就非常方便。

```js
const map = new Map();
map.set("first", "hello");
map.set("second", "world");

for (let [key, value] of map) {
  console.log(key + " is " + value);
}
// first is hello
// second is world
```

如果只想获取键名，或者只想获取键值，可以写成下面这样。

```js
// 获取键名
for (let [key] of map) {
  // ...
}

// 获取键值
for (let [, value] of map) {
  // ...
}
```

输入模块的指定方法

加载模块时，往往需要指定输入哪些方法。解构赋值使得输入语句非常清晰。

```js
const { SourceMapConsumer, SourceNode } = require("source-map");
```

### 字符串使用

Unicode 表示

```js
"z" === "z"; // true
"\172" === "z"; // true
"\x7A" === "z"; // true
"\u007A" === "z"; // true
"\u{7A}" === "z"; // true
```

codePointAt()

```js
var s = "𠮷";

s.length; // 2
s.charAt(0); // ''
s.charAt(1); // ''
s.charCodeAt(0); // 55362
s.charCodeAt(1); // 57271
```

String.fromCodePoint()

```js
String.fromCodePoint(0x20bb7);
// "𠮷"
String.fromCodePoint(0x78, 0x1f680, 0x79) === "x\uD83D\uDE80y";
// true
```

遍历 for let i of x

```js
for (let codePoint of "foo") {
  console.log(codePoint);
}
// "f"
// "o"
// "o"
```

字符串常用函数

```js
includes()
startsWitch()
endsWith()
repeat()
matchAll()
match()、replace()、search()、split()
```

### 函数

```js
//默认值
function log(x, y = "World") {
  console.log(x, y);
}

log("Hello"); // Hello World
log("Hello", "China"); // Hello China
log("Hello", ""); // Hello

//...函数[和golang差不多]
function add(...values) {
  let sum = 0;

  for (var val of values) {
    sum += val;
  }

  return sum;
}

add(2, 5, 3); // 10
add(...numbers);

//lamda表达式
var sum = (num1, num2) => {
  return num1 + num2;
};
var sum = (num1, num2) => num1 + num2;
// 等同于
var sum = function (num1, num2) {
  return num1 + num2;
};
```

### 数组

```js
//Array.from()
let arrayLike = {
  "0": "a",
  "1": "b",
  "2": "c",
  length: 3,
};

// ES6的写法
let arr2 = Array.from(arrayLike); // ['a', 'b', 'c']
//Array.of()
Array.of(3, 11, 8); // [3,11,8]

//遍历数组
for (let [index, elem] of ["a", "b"].entries()) {
  console.log(index, elem);
}
// 0 "a"
// 1 "b"
```

### SET

```js
// 例一
const set = new Set([1, 2, 3, 4, 4]);
[...set];
// [1, 2, 3, 4]

// 例二
const items = new Set([1, 2, 3, 4, 5, 5, 5, 5]);
items.size; // 5
```

### class

```js
class Point {
  constructor(x, y) {
    this.x = x;
    this.y = y;
  }

  toString() {
    return "(" + this.x + ", " + this.y + ")";
  }
}
```

### 模块加载

```js
// ES6 模块不是对象，而是通过export命令显式指定输出的代码，再通过import命令输入。
export var firstName = "Michael";
export var lastName = "Jackson";
export var year = 1958;
import { stat, exists, readFile } from "fs";
export { area as circleArea } from "circle";
```
