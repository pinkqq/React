Redux 是独立于 UI 框架外的状态管理框架，Redux 并不是专门为了 React 设计的，而是一个 JS 库，**所以 Redux 和 React 之间没有关系。**

但是 Redux 和 React 这类库的搭配效果是最好的，因为这类库允许你以 `state` 函数的形式来描述界面，Redux 通过 `action` 的形式来发起 `state` 变化。

那么如何在 React 中应用 Redux 呢？

我们通过一个 Counter（计数器）来走进 Redux 和 React 的故事叭。

## 在开始前

React-Redux 将所有组件分成两大类：[**展示组件**（presentational component）和 **容器组件**（container component）](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0)。

简单来说：**展示组件 只负责 UI 的呈现，容器组件 只负责管理数据和逻辑**。

体现到组件的结构上：外层是一个 **容器组件**，里面包了一个 **展示组件**。容器组件 负责与外部的通信，将数据传给 展示组件，再由 展示组件 渲染出视图。

React-Redux 提供了自动生成容器组件的方法，用户只需要提供展示组件。

## 创建 react 项目

使用脚手架 [create-react-app](https://github.com/facebook/create-react-app)

```
npx create-react-app react_in_action
```

**`npx`：** 只是安装到临时目录，使用完后自动删除；这样可以避免全局安装，每次使用（ `create-react-app` 脚手架）都是最新的版本。

## 安装

因为 create-react-app 默认使用了 yarn 管理包，所以我们之后的安装最好也用 yarn。

#### redux

- yarn（推荐）

  ```
  yarn add --dev redux
  ```

- npm
  ```
  npm install --save redux
  ```

#### react-redux

React Redux 是 Redux 官方推出的绑定 React 的专用库。

- yarn（推荐）

  ```
  yarn add --dev react-redux
  ```

- npm
  ```
  npm install --save react-redux
  ```

## 实现 store

关于 store、action、reducer 的相关介绍，可以[戳这里](https://juejin.cn/post/6970883342720270373)。

- 目录

  ```
  ├── src
      ├── store
          ├── action.js
          ├── reducer.js
          └── store.js
      └── ...
  ```

- action.js

  ```js
  // Action creator
  export function plusOne() {
    // action
    return { type: "PLUS_ONE" };
  }

  export function minusOne() {
    return { type: "MINUS_ONE" };
  }
  ```

- reducer.js

  ```js
  // Store initial state
  const initialState = { count: 0 };

  // reducer
  const counter = (state = initialState, action) => {
    switch (action.type) {
      case "PLUS_ONE":
        return { count: state.count + 1 };
      case "MINUS_ONE":
        return { count: state.count - 1 };
      case "CUSTOM_COUNT":
        return { count: state.count + action.payload.count };
      default:
        break;
    }
    return state;
  };

  export default counter;
  ```

- store.js

  ```js
  import { createStore } from "redux";
  import counter from "./reducers";

  // Create store
  const store = createStore(counter);

  export default store;
  ```

## 实现展示组件（Counter）

实现一个没有状态（即不使用 `this.state` 这个变量）的 “纯组件”，所有数据都由参数（`this.props`）提供，在这里不使用任何 Redux 的 API。

- 目录
  ```
  ├── src
      ├── components
          ├── Counter.css
          └── Counter.jsx
      └── ...
  ```
- Counter.jsx

  ```jsx
  import React from "react";
  import "./Counter.css";

  export class Counter extends React.Component {
    render() {
      const { count, plusOne, minusOne } = this.props;
      return (
        <div>
          <button onClick={minusOne}>-</button>
          <span style={{ display: "inline-block", margin: "0 10px" }}>
            {count}
          </span>
          <button onClick={plusOne}>+</button>
        </div>
      );
    }
  }

  export default Counter;
  ```

- Counter.css

  ```css
  button {
    border: none;
    outline: none;
    padding: 8px 15px;
    background-color: black;
    color: #fff;
    cursor: pointer;
  }
  ```

## 实现容器组件（ConnectedCounter）

我们使用 React-Redux 提供 `connect` 方法，生成上面说的容器组件（负责管理数据和业务逻辑）。

![未命名文件 (2).jpg](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/718d66b41e5b442baf07ef3b5f1a1baa~tplv-k3u1fbpfcp-watermark.image)

### connect

> connect([mapStateToProps], [mapDispatchToProps], [mergeProps], [options])

通过 `connect` 可以连接 React 组件与 Redux store。使得 Store 可以为 展示组件 提供数据，同时 展示组件 上的交互操作可以更新到 Store。

- [`mapStateToProps(state, [ownProps]): stateProps`] (Function):

  定义组件需要监听 `state` 上哪些字段，会将指定的字段与组件的 `props` 合并。

  为了避免不必要的访问和更新，我们要将字段限制到最小范围（必须范围），保持组件的高性能。

  ```js
  function mapStateToProps(state) {
    return {
      count: state.count,
    };
  }
  ```

- [`mapDispatchToProps(dispatch, [ownProps]): dispatchProps`] (Object or Function):

  会将指定的 `action` 绑定到组件上，与组件的 `props` 合并。

  ```js
  function mapDispatchToProps(dispatch) {
    return bindActionCreators({ plusOne, minusOne }, dispatch);
  }
  ```

- [`mergeProps(stateProps, dispatchProps, ownProps): props`] (Function):

  `mapStateToProps()` 与 `mapDispatchToProps()` 的执行结果和组件自身的 `props` 将传入到这个回调函数中。返回的对象将作为组件的 `props`。

- [`options`] (Object): 可以定制 `connector` 的行为。

### 实现

- 目录
  ```
  ├── src
      ├── containers
          └── ConnectedCounter.jsx
      └── ...
  ```
- ConnectedCounter.jsx

  ```jsx
  import { connect } from "react-redux";
  import { plusOne, minusOne } from "../store/actions";
  import Counter from "../components/Counter.jsx";

  function mapStateToProps(state) {
    return {
      count: state.count,
    };
  }

  const mapDispatchToProps = { plusOne, minusOne };

  /*
  ** 简写语法等同于
  
    function mapDispatchToProps(dispatch) {
    return bindActionCreators({ plusOne, minusOne }, dispatch);
    }
  
  */

  const ConnectedCounter = connect(
    mapStateToProps,
    mapDispatchToProps
  )(Counter);

  export default ConnectedCounter;
  ```

## 整合到 App.js

```js
// src/App.js

import ConnectedCounter from "./containers/ConnectedCounter.jsx";

function App() {
  return (
    <div className="App">
      <ConnectedCounter></ConnectedCounter>
    </div>
  );
}

export default App;
```

## Provider

### `<Provider store>`

React-Redux 提供 `Provider` 组件，可以让容器组件拿到 Redux Store (`state`)。

```jsx
<Provider store={store}>
  <App />
</Provider>
```

在根组件外面包了一层 `<Provider>`，`App` 的所有子节点都可以访问到 `state` 了。

它的原理是 React 的 [Context API](https://juejin.cn/post/6968738849719582727)。

### 实现

```jsx
// src/index.js

import React from "react";
import ReactDOM from "react-dom";
+ import { Provider } from "react-redux";

import "./index.css";
import App from "./App";
import reportWebVitals from "./reportWebVitals";
+ import store from "./store/store";

ReactDOM.render(
  <React.StrictMode>
+    <Provider store={store}>
      <App />
+    </Provider>
  </React.StrictMode>,
  document.getElementById("root")
);

reportWebVitals();

```

现在可以跑一下 `yarn start`，操作一下我们的 Counter。

## 参考资料

- 搭配 React：https://cn.redux.js.org/docs/basics/UsageWithReact.html
- 容器组件和展示组件相分离：https://cn.redux.js.org/docs/basics/UsageWithReact.html
- React 开发思想：https://zh-hans.reactjs.org/docs/thinking-in-react.html
