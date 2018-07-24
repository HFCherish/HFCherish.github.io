---
title: react setState 的坑
date: 2018-06-05 19:39:46
toc: true
tags:
	- react
---

# 问题

`setState` 是更新 `state` 的 API，进而会引发组件的重新渲染。然而在使用过程中发现有些坑(参见 [react 正确使用状态](https://doc.react-china.org/docs/state-and-lifecycle.html#%E6%AD%A3%E7%A1%AE%E5%9C%B0%E4%BD%BF%E7%94%A8%E7%8A%B6%E6%80%81))：

>1. **`setState(obj)`一般情况下不会立即更新 `state` 的值；**
>2. **同一 cycle 的多次 `setState` 调用可能会合并（性能考虑）**

对于第一点，引用下边的例子：

```jsx
function incrementMultiple() {
  this.setState({count: this.state.count + 1});
  this.setState({count: this.state.count + 1});
  this.setState({count: this.state.count + 1});
}
```

代码运行时，虽然是对 `state` 加了三次，但是每次加操作都是针对初始的 `state`，所以最终相当于仅加了一次。即上述代码等同于下边的代码：

```jsx
Object.assign(
  previousState,
  {quantity: state.quantity + 1},
  {quantity: state.quantity + 1},
  ...
)
```

# 为什么
## 从 react 生命周期看 setState 生效时间

要理解其原因，我们要先看一下 [react 生命周期](https://github.com/superman66/Front-End-Blog/issues/2)：

![react 生命周期](https://cloud.githubusercontent.com/assets/12592949/24903814/1b2ff98c-1ee1-11e7-9f5a-59eb84171b53.png)

`setState` 引起状态更新涉及四个函数：

* shouldComponentUpdate（默认 true）
* componentWillUpdate
* render
* componentDidUpdate

**但直到 `render` 被调用时，`state` 才被更新**。

（如果 `shouldComponentUpdate` 返回 `false`，则 `render` 不会被执行，但是 `state` 会被更新）

因此多次调用 `setState` 时，不能依靠 `this.state` 来计算下一个 `state` 的值，因为 `this.state` 一直没更新。

>setState() does not always immediately update the component. It may batch or defer the update until later. This makes reading this.state right after calling setState() a potential pitfall.

但是可以使用 setState(func) api 来解决上述问题

## [setState api](https://doc.react-china.org/docs/react-component.html#setstate)

`setState` api 如下：

```js
setState(updater[, callback])
```

其中，`callback` 是在 `setState` 完成且组件 re-render 完成后执行（一般建议用 `componentDidUpdate` 来实现类似逻辑）。

而 `updater` 可以是：

* obj：异步将 obj **shallow merge** 到旧 `state`，构建新 `state`
* func：形式如下：

```js
(prevState, props) => stateChange
```

# 解决方案

上述例子做如下修改即可实现真正的累加：

```jsx
function increment(state) {
  return {count: state.count + 1};
}

function incrementMultiple() {
  this.setState(increment);
  this.setState(increment);
  this.setState(increment);
}
```


[更详细请点击参考这篇文章](https://www.jianshu.com/p/9278c4835c55)