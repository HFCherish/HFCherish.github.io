---
title: javascript 表达式和操作符
date: 2018-05-31 16:09:58
toc: true
tags:
	- javascript
---

`...obj` 是 js 的扩展运算符，可以将一个可迭代的对象在函数调用的位置展开成为多个参数,或者在数组字面量中展开成多个数组元素。(其他可参见[运算符介绍](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Expressions_and_Operators#Relational_operators)，[运算符和表达式清单 reference](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators))

eg.

```js
// 在数组字面量中展开
// 利用扩展运算符，实现了数组合并
var parts = ['shoulder', 'knees'];
var lyrics = ['head', ...parts, 'and', 'toes'];


// 在函数调用处展开
function f(x, y, z) { }
var args = [0, 1, 2];
f(...args);
```