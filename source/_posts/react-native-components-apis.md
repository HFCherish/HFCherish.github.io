---
title: react native components & apis
tags:
  - react native
---

react 提倡组件化开发，以促进复用。react native 也一样是组件化开发思想。不同的是，react 中是使用原生的 html 组件作为基本组件（div、a...），而 react native 使用的是另一些原生组件。用户的自定义组件也是基于这些原生组件。

所以要用 react native，必须了解这些原生组件（就跟学 html 组件差不多）

# general props
一些所有组件或大部分组件都有的属性。

## [style](https://reactnative.cn/docs/0.51/style.html#content)

所有的核心组件都接受名为 `style` 的属性（类似 html 标签中的 `html`）。这些样式名基本上是遵循了 web 上的 CSS 的命名，只是按照 JS 的语法要求使用了驼峰命名法，例如将 `background-color` 改为`backgroundColor`。

实际开发中组件的样式会越来越复杂，我们建议使用StyleSheet.create来集中定义组件的样式。常见的做法是按顺序声明和使用 `style` 属性，以借鉴 CSS 中的“层叠”做法（即后声明的属性会覆盖先声明的同名属性）。比如像下面这样：

```js
import React, { Component } from 'react';
import { AppRegistry, StyleSheet, Text, View } from 'react-native';

export default class LotsOfStyles extends Component {
  render() {
    return (
      <View>
        <Text style={styles.red}>just red</Text>
        <Text style={styles.bigblue}>just bigblue</Text>
        <Text style={[styles.bigblue, styles.red]}>bigblue, then red</Text>
        <Text style={[styles.red, styles.bigblue]}>red, then bigblue</Text>
      </View>
    );
  }
}

const styles = StyleSheet.create({
  bigblue: {
    color: 'blue',
    fontWeight: 'bold',
    fontSize: 30,
  },
  red: {
    color: 'red',
  },
});

// 注册应用(registerComponent)后才能正确渲染
// 注意：只把应用作为一个整体注册一次，而不是每个组件/模块都注册
AppRegistry.registerComponent('LotsOfStyles', () => LotsOfStyles);
```

### [组件宽度高度](https://reactnative.cn/docs/0.51/height-and-width.html#content)

有两种设定方法：

**1. 指定宽高**

最简单的给组件设定尺寸的方式就是在样式中指定固定的 `width`和 `height`。React Native中的尺寸都是无单位的，表示的是与设备像素密度无关的逻辑像素点。

**2. 弹性宽高**

在组件样式中使用 `flex` 可以使其在可利用的空间中动态地扩张或收缩。一般而言我们会使用 `flex:1` 来指定某个组件扩张以撑满所有剩余的空间。如果有多个并列的子组件使用了 `flex:1`，则这些子组件会平分父容器中剩余的空间。如果这些并列的子组件的 `flex` 值不一样，则谁的值更大，谁占据剩余空间的比例就更大（即占据剩余空间的比等于并列组件间 `flex` 值的比）。

>组件能够撑满剩余空间的前提是其父容器的尺寸不为零。如果父容器既没有固定的 `width` 和 `height`，也没有设定 `flex`，则父容器的尺寸为零。其子组件如果使用了 `flex`，也是无法显示的。

```js
import React, { Component } from 'react';
import { AppRegistry, View } from 'react-native';

export default class FlexDimensionsBasics extends Component {
  render() {
    return (
      // 试试去掉父View中的`flex: 1`。
      // 则父View不再具有尺寸，因此子组件也无法再撑开。
      <View style={{flex: 1}}>
      // 这里设定的是固定宽高
        <View style={{width: 50, height: 50, backgroundColor: 'powderblue'}} />
        // 弹性宽高
        <View style={{flex: 2, backgroundColor: 'skyblue'}} />
        <View style={{flex: 3, backgroundColor: 'steelblue'}} />
      </View>
    );
  }
};

AppRegistry.registerComponent('AwesomeProject', () => FlexDimensionsBasics);
```

### [flexbox 布局](https://reactnative.cn/docs/0.51/layout-with-flexbox.html#content)

一般用以下三个属性就可以完成布局的要求了，完整的布局样式属性参照 [这篇文章](https://reactnative.cn/docs/0.51/layout-props.html)（也可以参考[图解布局模式](https://weibo.com/1712131295/CoRnElNkZ?ref=collection&type=comment#_rnd1526464804589)）：

1. `flexDirection`: 定义布局主轴。`row`（水平轴）、`column`（default）
2. `justifyContent`: 定义主轴上元素排列的方式。`flex-start`、`center`、`flex-end`、`space-around`（元素在主轴上均匀分布）、`space-between`(元素间距均匀分布)
3. `alignItems`: 定义子元素在次轴（与主轴垂直）上的排列方式。`flex-start`、`flex-end`、`center`、`stretch`（要使 `stretch` 选项生效的话，子元素在次轴方向上不能有固定的尺寸）

# 核心组件 components

react native 提供了很多内置的 components，社区也有很多开发者提供有很多实用的 components（可以直接从 npm 搜 [`react native`](https://www.npmjs.com/search?q=react-native&page=1&ranking=optimal)，或者查看 [awesome react native](http://www.awesome-react-native.com/#components) 里总结的一些列表）

大部分组件，最终都是基于下边这些基本组件写的。[components & apis](https://facebook.github.io/react-native/docs/components-and-apis.html#user-interface) 总结了 basic components、用于 ui 渲染的、list 的、ios specific 的、android specific 的（学习这些相当于是学习 html 基本组件的过程）

## [View](https://reactnative.cn/docs/0.51/view.html#content)
View 常用作其他组件的容器，来帮助控制布局和样式（类似于 div）

## [Text](https://reactnative.cn/docs/0.51/text.html#content)
写文本的

## [TextInput](https://reactnative.cn/docs/0.51/textinput.html#content)
是一个允许用户输入文本的基础组件

## [Image](https://reactnative.cn/docs/0.51/image.html#content)
放图片的


## [SrollView](https://reactnative.cn/docs/0.51/scrollview.html#content)

`ScrollView` 是一个通用的可滚动的容器，你可以在其中放入多个组件和视图，而且这些组件并不需要是同类型的。`ScrollView` 不仅可以垂直滚动，还能水平滚动（通过 `horizontal` 属性来设置）。


## [StyleSheet](https://reactnative.cn/docs/0.51/stylesheet.html#content)

这其实是一个接口

StyleSheet提供了一种类似CSS样式表的抽象

## [FlatList](https://reactnative.cn/docs/0.51/flatlist.html#content)

react native 有几种用于显示长列表的组件：`flatlist`、`sectionlist`、`scrollview` 等。

`FlatList` 组件用于显示一个垂直的滚动列表，其中的元素之间结构近似而仅数据不同，且元素个数可以增删。和 `ScrollView` 不同的是，`FlatList` 并不立即渲染所有元素，而是优先渲染屏幕上可见的元素。而那些已经渲染好了但移动到了屏幕之外的元素，则会从原生视图结构中移除（以提高性能）.

`FlatList` 组件必须的两个属性是 `data` 和 `renderItem`。`data` 是列表的数据源，而 `renderItem` 则从数据源中逐个解析数据，然后返回一个设定好格式的组件来渲染。

```js
export default class FlatListBasics extends Component {
  render() {
    return (
      <View style={styles.container}>
        <FlatList
          data={[
            {key: 'Devin'},
            {key: 'Jackson'},
            {key: 'James'},
            {key: 'Joel'},
            {key: 'John'},
            {key: 'Jillian'},
            {key: 'Jimmy'},
            {key: 'Julie'},
          ]}
          renderItem={({item}) => <Text style={styles.item}>{item.key}</Text>}
        />
      </View>
    );
  }
}
```

## [SectionList](https://reactnative.cn/docs/0.51/sectionlist.html#content)

如果要渲染的是一组需要分组的数据，也许还带有分组标签的，那么 `SectionList` 将是个不错的选择

```js
render() {
    return (
      <View style={styles.container}>
        <SectionList
          sections={[
            {title: 'D', data: ['Devin']},
            {title: 'J', data: ['Jackson', 'James', 'Jillian', 'Jimmy', 'Joel', 'John', 'Julie']},
          ]}
          renderItem={({item}) => <Text style={styles.item}>{item}</Text>}
          renderSectionHeader={({section}) => <Text style={styles.sectionHeader}>{section.title}</Text>}
        />
      </View>
    );
  }
```

# API

接口其实是仅提供接口功能的简单组件。这些组件可能没有渲染功能。
