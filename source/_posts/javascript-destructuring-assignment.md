---
title: javascript 解构语法
date: 2018-05-31 11:24:20
toc: true
tags:
	- javascript
---


具体可参见 [mdn javascript 解构语法](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment). 这里简单总结一下。

# 解构是做什么的
解构就是一种方便变量赋值的语法，由编译器完成真正的变量赋值

# 数组解构

* **将数组元素赋值给变量**
* **赋值依据是元素顺序**
* 指定变量名时，可以提供默认值，以避免 `undefined` 赋值
* 支持忽略一些元素（添加 `,` ，但不提供变量名）
* 支持 	`rest` 数组赋值

eg.

```js
// 基本赋值
var a, b, rest;
[a, b] = [10, 20];
console.log(a); // 10
console.log(b); // 20


// 默认值
var a, b;

[a=5, b=7] = [1];
console.log(a); // 1
console.log(b); // 7


// 忽略某些元素
function f() {
  return [1, 2, 3];
}

var [a, , b] = f();
console.log(a); // 1
console.log(b); // 3


// rest 赋值
[a, b, ...rest] = [10, 20, 30, 40, 50];
console.log(a); // 10
console.log(b); // 20
console.log(rest); // [30, 40, 50]
```

# 对象解构
和数组解构差不多：

* **将对象属性赋值给变量**
* **赋值依据是属性名称**
* 指定属性名时，可以给默认值，避免 `undefined` 赋值
* 变量名称默认是属性名，也可以自定义，通过 `propertyName: customName` 方式定义
* 支持 `rest` 对象赋值（在 proposal 中）

eg.

```js
// 基本赋值
({ a, b } = { a: 10, b: 20 });
console.log(a); // 10
console.log(b); // 20


// 默认值
var {a = 10, b = 5} = {a: 3};

console.log(a); // 3
console.log(b); // 5


// 自定义变量名 + 默认值
var o = {p: 42, q: true};
var {p: foo, q: bar, m: other='haha'} = o;
 
console.log(foo); // 42 
console.log(bar); // true
console.log(other); // haha


// rest 赋值
// Stage 3 proposal
({a, b, ...rest} = {a: 10, b: 20, c: 30, d: 40});
console.log(a); // 10
console.log(b); // 20
console.log(rest); //{c: 30, d: 40}

// 无声明赋值
// 相当于 var {a,b} = {a:1, b:2};
// 括号去掉， {a,b} 是块代码，不是对象，所以不能去掉
var a, b;

({a, b} = {a: 1, b: 2});

```