---
title: redux
date: 2018-05-29 10:46:35
toc: true
tags:
- react
---

# what's redux

[redux 中文文档](https://cn.redux.js.org/) 中的几个关键特性：

1. **状态容器**，提供**可预测化**的状态管理
2. 跨平台，客户端、服务端、原生应用都能用
3. 易于测试
4. 轻量，支持 react 等界面库

其中第一点，讲明了 redux 的主要用途：**状态容器**

以 react 为例，页面是渲染状态树得到，有静态状态（`props`）、动态状态（`state`）。通过在代码中 `setState` 来修改状态树，状态自上而下传递到各个子组件，最终触发组件树的重新渲染。

使用 redux，我们就将状态的定义和允许的迁移 function 挪出去，放到一个状态容器里，来有效管理组件的所有状态和状态迁移。此时 `component` 代码中就没有 `state`、`setState` 等定义了。

## counter without redux

以计数器为例，不使用 redux，即直接在本地定义 `state`，并通过 `setState` 修改状态，以实现重现渲染。

```jsx
import React, {Component} from 'react';

export default class Counter extends Component {
  state = {
    value: 0,
  };

  render() {
    return (
      <div>
        {this.state.value}
        <button onClick={this.increment}>+</button>
        <button onClick={this.decrement}>-</button>
      </div>
    );
  }

  decrement = () => {
    this.setState({
      value: this.state.value - 1
    });
  }

  increment = () => {
    this.setState({
      value: this.state.value + 1
    });
  }
}
```

## counter with redux

如果使用 redux，就是将原本的 `state` 存储到一个 `store` 中，通过 `dispatch` 一个 `action`(eg. `{type: 'INCREMENT'}`)触发状态变化，`action` 如何改变 `state` 是由一个纯函数 `reducer` 定义的。

```jsx
// counter.js
import React, {Component} from 'react';

export default class Counter extends Component {

  render() {
    return (
      <div>
        {this.props.store.getState()}
        <button onClick={this.increment}>+</button>
        <button onClick={this.decrement}>-</button>
      </div>
    );
  }

  decrement = () => {
  // 改变内部 state 惟一方法是 dispatch 一个 action。
    this.props.store.dispatch({type: 'DECREMENT'});
  }

  increment = () => {
    this.props.store.dispatch({type: 'INCREMENT'});
  }
}


// reducer.js
/**
 * 这是一个 reducer，形式为 (state, action) => state 的纯函数。
 * 描述了 action 如何把 state 转变成下一个 state。
 */
export default function counter(state = 0, action) {
  switch (action.type) {
    case 'INCREMENT':
      return state + 1;
    case 'DECREMENT':
      return state - 1;
    default:
      return state;
  }
}


// index.js
import React from 'react';
import * as ReactDOM from "react-dom";
import Counter from "./Counter";
import counter from "./reducer";
import {createStore} from 'redux';

const store = createStore(counter);

const render = () => ReactDOM.render(
  <Counter store={store}/>,
  document.getElementById('root')
);

render();

// store 的订阅接口，注册 listener（这里即 render）来订阅 store 的状态更新。
store.subscribe(render);

```

# why redux

不使用 redux 时，

1. **状态（model）、状态如何被改变（controller）、页面模板/html（view）是耦合在一块的。**<font color='gray'>（react 提供的就是一种界面库，render 函数其实写的就是页面模板，将 model 填充进去，渲染得到最后的 view）</font>
2. **组件之间强耦合，难以确定状态的改变是在哪里被谁触发，bug 追踪难。**

使用 redux 后，

1. **model 被抽出来了**（即当前视图所对应的 model 对象），而且这个 model 对象不是一个贫血模型，它提供基本的 action 来保证 model 的完整性。
2. 由于状态的变更只能通过 `dispatch` 来触发，**解耦了父子组件之间的状态变更传递**，易于定位

# 核心概念

redux 是一个状态容器，它解决了： 

* 状态在哪：`createStore()`
* 状态是什么：`store.getState()`
* 状态怎么变： reducer，即 `(preState, action) => nextState` 函数
* 触发状态变更：`store.dispatch(action)`
* 谁关心状态变化：`store.subscribe(callback)`


# References
* [usage with react](http://redux.js.org/docs/basics/UsageWithReact.html)
* [显示和容器组件](http://redux.js.org/docs/basics/UsageWithReact.html)
* [为什么用redux？原理？](https://www.zhihu.com/question/41312576)
* [看漫画学redux](https://github.com/jasonslyvia/a-cartoon-intro-to-redux-cn)