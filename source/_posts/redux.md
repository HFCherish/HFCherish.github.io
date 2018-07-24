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

以计数器为例，不使用 redux，即直接在本地定义 `state`，并通过 `setState` 修改状态，以实现重现渲染。([源代码](https://github.com/HFCherish/react-learning/blob/master/redux/counter/src/Counter.js))

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

如果使用 redux，就是将原本的 `state` 存储到一个 `store` 中，通过 `dispatch` 一个 `action`(eg. `{type: 'INCREMENT'}`)触发状态变化，`action` 如何改变 `state` 是由一个纯函数 `reducer` 定义的。（[源代码](https://github.com/HFCherish/react-learning/blob/master/redux/counter/src/CounterWithRedux.js)）

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
2. 由于状态的变更只能通过 `dispatch` 来触发，**解耦了父子组件之间的状态变更传递**，易于定位. （可以利用 [middleware](http://cn.redux.js.org/docs/advanced/Middleware.html) 记录 state 变更日志，即可实现 state 变化过程透明和可预测）

# 核心概念

redux 是一个状态容器，它解决了： 

* 状态在哪：`createStore()`
* 状态是什么：`store.getState()`
* 状态怎么变： reducer，即 `(preState, action) => nextState` 函数
* 触发状态变更：`store.dispatch(action)`
* 谁关心状态变化：`store.subscribe(callback)`

## [reducer](http://cn.redux.js.org/docs/basics/Reducers.html)

Reducers 指定了应用状态的变化如何响应 actions 并发送到 store 的，记住 **actions 只是描述了有事情发生了**这一事实，并没有描述应用如何更新 state。reducer 函数形式如下：

```
(preState, action) => nextState
```

因为 `state` 有很多属性，针对属性的处理也有很多，全放在一块就太大、难管理，所以可以拆 reducer（称为 slice reducer），再利用 [`combineReducer`](http://cn.redux.js.org/docs/api/combineReducers.html) 组合。<font color='gray'>（redux 总是一个根，多个子：一个 store，多个子状态；一个根 reducer，多个 slice reducers）</font>

使用 `combineReducer` 后有两个问题：

1. store state 和 slice reducer state 的关系？
2. slice reducer 如何被调用执行？

### store state 和 slice reducer state 的关系？

[官方 API 描述](http://cn.redux.js.org/docs/api/combineReducers.html)

>**`combineReducers()` 返回的 `state` 对象，会将传入的每个 reducer 返回的 `state` 按其传递给 `combineReducers()` 时对应的 key 进行命名，并将所有 slice reducer 返回的结果合并** 

举例：

```js
const reducer1  = (state='initA', action) => {...}
const reducer2  = (state='initB', action) => {...}

// 这里 `{reducer1, reducer2}` 使用的是 es2015 的 shorthand property names。
// 等价于 `{reducer1: reducer1, reducer2: reducer2}`
const store = createStore(combineReducer({reducer1, reducer2}));
console.log(store.getState()); //{reducer1: 'initA', reducer2: 'initB'}

// 也可以指定 key 名称
const store = createStore(combineReducer({a:reducer1, b:reducer2}));
console.log(store.getState()); //{a: 'initA', b: 'initB'}
```

### slice reducer 如何被调用执行？

[官方 API 描述](http://cn.redux.js.org/docs/api/combineReducers.html)

>返回值：
>(Function)：**一个调用 reducers 对象里所有 reducer 的 reducer，并且构造一个与 reducers 对象结构相同的 state 对象。**

即 combinedReducer 会调用执行所有的 slice reducer，以实现分层处理

## [初始化 state](http://cn.redux.js.org/docs/recipes/reducers/InitializingState.html)

有两种方式：

1. 使用 `createStore(reducers, [preloadedState], [enhancers])` 中的 `preloadedState`
2. 使用 reducer 中的默认属性（`function someReducer(state={defaultValue}, action){}`）

这其中有两条规则：

1. **preloadedState 先于 reducer 去填充 state**
2. **创建 store 后，redux 会 `dispatch` 一个虚拟的 `action` 到 reducer，以触发其中的默认值来填充 state。**

### 使用单一简单 reducer

>*使用 `preloadedState` 后，单一 reducer 的默认 state 赋值**会**失效，因为流程是这样的：*

1. 创建 store，并根据 `preloadedState` 填充 `state`；
2. `dispatch` 虚拟 `action` 来使用 reducer 默认值填充 `state`。而此时 `state` 已经不是 `undefind`，因此，这个默认值填充是无效的

举一个例子：

```js
// reducer
function counterReducer(state=0, action) {...}

// 仅使用 reducer 初始化
const store = createStore(counterReducer);
console.log(store.getState()); //0

// 同时使用 preloadedState 和 reducer
const store = createStore(counterReducer, 42);
console.log(store.getState()); //42
```

### 使用 `combineReducer()`

>*使用 `preloadedState` 后，使用了 `combineReducers` 的 reducer 默认 state 赋值**可能会**失效，因为流程是这样的：*

1. 创建 store，并根据 `preloadedState` 填充 `state`；
2. `dispatch` 虚拟 `action` 来调用所有子 reducers，
	* 如果 `preloadedState` 中定义了当前 `reducer` 的属性，则对当前 reducer，`state` 已经不是 `undefind`，此时默认值填充是无效的
	* 如果 `preloadedState` 中没有定义当前 `reducer` 的属性，则对当前 reducer，`state` 是 `undefind`，默认值填充有效

同样看一个例子：

```js
// reducer
const reducer1  = (state='initA', action) => {...}
const reducer2  = (state='initB', action) => {...}

const combinedReducer = combineReducers({a: reducer1, b: reducer2})

// 仅使用 reducer 初始化
const store = createStore(combinedReducer);
console.log(store.getState()); //{a: 'initA', b: 'initB'}

// 同时使用 preloadedState 和 reducer
const store = createStore(counterReducer, {a: 'initInPreload'});
console.log(store.getState()); //{a: 'initInPreload', b: 'initB'}
```

>*使用 `combineReducers` 时，`preloadedState` 必须包含 slice reducer 中的属性，与传入的 keys 保持同样的结构* (参见 [createStore API 说明](http://cn.redux.js.org/docs/api/createStore.html))

举例：

```js
const reducer1 = (state='initA', action) => {...}
const combinedReducer = combineReducers({a: reducer1, b: reducer2})

// 正确用法
const store = createStore(combinedReducer, {b: 'bala'});
console.log(store.getState()); //{a: 'initA', b: 'bala'};

// 错误用法. 此时不会包含 c 属性。
// 因为所有的 state 都必须映射到 reducer 上，必须保持同样的结构，映射不到的就会被忽略
const store = createStore(combinedReducer, {c: 'bala'});
console.log(store.getState()); //{a: 'initA', b: 'initB'};

```


# References
* [usage with react](http://redux.js.org/docs/basics/UsageWithReact.html)
* [显示和容器组件](http://redux.js.org/docs/basics/UsageWithReact.html)
* [为什么用redux？原理？](https://www.zhihu.com/question/41312576)
* [看漫画学redux](https://github.com/jasonslyvia/a-cartoon-intro-to-redux-cn)