---
title: react test
toc: true
date: 2018-12-07 14:30:40
tags:
	- react
	- test
	- jest
	- enzyme
---

# Several library

Reference: [Testing React with Jest and Enzyme](https://medium.com/codeclan/testing-react-with-jest-and-enzyme-20505fec4675)

## [enzyme](https://airbnb.io/enzyme/)

>Enzyme is a JavaScript Testing utility for React that makes it easier to assert, manipulate, and traverse your React Components’ output.

Enzyme, created by Airbnb, adds some great additional utility methods for **rendering a component** (or multiple components), **finding elements**, and **interacting with elements**.

Enzyme can only be used in React. It provide additional testing utilities to test components

## [Jest](https://jestjs.io/docs/en/api)

> Jest is a JavaScript unit testing framework, used by Facebook to test services and React applications.

CRA (create react app) comes bundled with Jest; it does not need to be installed separately.

Jest acts as a **test runner**, **assertion library**, and **mocking library**.

Jest also provides **Snapshot testing**, the ability to create a rendered ‘snapshot’ of a component and compare it to a previously saved ‘snapshot’. The test will fail if the two do not match. Snapshots will be saved for you beside the test file that created them in an auto-generate `__snapshots__` folder.

Snapshot testing must be complimented with in browser testing and looking at the snapshot created when creating the initial snapshots, to ensure that the snapshot reflects the intended outcome. An example snapshot:

```
exports[`List shallow renders correctly with no props 1`] = `
<List
  ordered={false}
>
  <ListItem>
    Item 1
  </ListItem>
  <ListItem>
    Item 1
  </ListItem>
</List>
`;
```

Jest is designed to test React, but can be used with any other js app

### [Configuration](https://jestjs.io/docs/en/configuration.html)

When working with enzyme, you can config the `package.json`:

```json
"jest": {
  "snapshotSerializers": ["enzyme-to-json/serializer"]
}
```

This avoids `toJson` when use snapshot and enzyme:

```js
expect(toJson(rawRendereComponent)).toMatchSnapshot();
```

#### setupTest files

You can use [`setupFiles`](https://jestjs.io/docs/en/configuration.html#setupfiles-array) to configure the test enviroment. The file will be executed only once for each test file. e.g.

```react
import { configure } from 'enzyme';
import Adapter from 'enzyme-adapter-react-16';
configure({ adapter: new Adapter() });
```

### Usage

Jest looks for test in the following places:

- Files with `.js` suffix in `__tests__` folders.
- Files with `.test.js` suffix.
- Files with `.spec.js` suffix.

It is convention to put each test file next to the code it is testing.

# [Renderer](https://reactjs.org/docs/test-renderer.html#overview) vs [shallow](https://airbnb.io/enzyme/docs/api/ShallowWrapper/shallow.html) vs [mount](https://airbnb.io/enzyme/docs/api/ReactWrapper/mount.html)

reference from '[enzyme render diffs](https://gist.github.com/fokusferit/e4558d384e4e9cab95d04e5f35d4f913)'

## Shallow

Real unit test (isolation, no children render)

### Simple shallow

Calls:

- constructor
- render

### Shallow + setProps

Calls:

- componentWillReceiveProps
- shouldComponentUpdate
- componentWillUpdate
- render

### Shallow + unmount

Calls:

- componentWillUnmount

### Mount

The only way to test componentDidMount and componentDidUpdate.
Full rendering including child components.
Requires a DOM (jsdom, domino).
More constly in execution time.
**If react is included before JSDOM, it can require some tricks:**

**`require('fbjs/lib/ExecutionEnvironment').canUseDOM = true;`** 

### Simple mount

Calls:

- constructor
- render
- componentDidMount

### Mount + setProps

Calls:

- componentWillReceiveProps
- shouldComponentUpdate
- componentWillUpdate
- render
- componentDidUpdate

### Mount + unmount

Calls:

- componentWillUnmount

### Render

only calls render but renders all children.

So my rule of thumbs is:

- Always begin with shallow
- If componentDidMount or componentDidUpdate should be tested, use mount
- If you want to test component lifecycle and children behavior, use mount
- If you want to test children rendering with less overhead than mount and you are not interested in lifecycle methods, use render

There seems to be a very tiny use case for render. I like it because it seems snappier than requiring jsdom but as @ljharb said, we cannot really test React internals with this.

I wonder if it would be possible to emulate lifecycle methods with the render method just like shallow ?
I would really appreciate if you could give me the use cases you have for render internally or what use cases you have seen in the wild.

I'm also curious to know why shallow does not call componentDidUpdate.

Kudos goes to https://github.com/airbnb/enzyme/issues/465#issuecomment-227697726 this gist is basically a copy of the comment but I wanted to separate it from there as it includes a lot of general Enzyme information which is missing in the docs.
