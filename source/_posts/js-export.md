---
title: js export
toc: true
date: 2018-06-15 18:17:19
tags:
	- javascript
---

[export](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/export) 用于从 module 中导出函数、对象、原始值，在其他地方通过 `import` 使用这些函数、对象、原始值。

有两种导出方式：命名导出（`export`），默认导出（`export default`）

# 1. 命名导出
就是导出 module 中有命名的函数、对象、原始值。相应的，import 时，必须使用相同的命名引入（当然可以使用 `import a as b from './module'` 来修改名称）

举例：

```js
// 定义时导出
export const a = 'a', b = 'b'; // 适用于 var, let
export function a(){};
export class a{};

// 定义后导出
const a = 'a', b = 'b';
export {a, b as newB};

// 导出其他模块的导出。
// 此时仅会导出 otherModule 的命名导出，所以这里最终导出的还是有命名的
export * from 'otherModule';
export {a, b as newB} from 'otherModule';

// 如果需要导出 otherModule 的默认导出，只能这样写：
import defaultExport from 'otherModule';
export [default] defaultExport;
```

# 2. 默认导出
默认导出（`export default`）的函数、类可以认为没有固定名称，即 import 时，可以用任何名称导入

一个 module 中只能有一个默认导出。

`export default` 后不能跟 `const`, `let`, `var`

举例：

```js
// 默认导出函数（定义时）
export default function(){}
import myname from 'module';

export default function funcName(){}
import funcNameMy from 'module2';

// 默认导出类（定义时）
export default class className{}

// 默认导出 expression
export default a = 'a'

// 默认导出（定义后）
export {a as default, b, c as newC}
```