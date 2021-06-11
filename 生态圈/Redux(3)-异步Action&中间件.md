## 说在前面

**异步 Action 是一个设计模式，即异步 Action 并不是一个特殊的 Action，而是同步 Action 的组合使用。**

请带着这句话理解异步 Action 和中间件，这非常重要。

## 异步 Action

首先，**什么是异步 Action？**

- Action 发出以后，Reducer 立即算出 State，是同步；
- Action 发出以后，过一段时间再执行 Reducer，就是异步。

上一篇 -- [Redux（1）：入门三件套（Action、Reducer、Store）](https://juejin.cn/post/6970883342720270373)介绍的都是同步 Action，用户发射（`dispatch`）一个 Action ，Reducer 会同步计算新的 State，从而引发视图（View）更新。如下图：

![redux.jpg](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d662572ec9b345f39708f47d3bd05eed~tplv-k3u1fbpfcp-watermark.image)

相对比同步 Action，异步 Action 至少需要 dispatch 三种 action：

- 请求开始的 action
- 请求成功的 action
- 请求失败的 action

```js
{ type: 'FETCH_POSTS_REQUEST' }
{ type: 'FETCH_POSTS_FAILURE', error: 'Oops' }
{ type: 'FETCH_POSTS_SUCCESS', response: { ... } }
```

了解了异步 Action，那么怎么才能实现？又如何能在发送 action 后（`store.dispatch()`）将其截获，并且在一段时间后又自动发出 action 呢？

我们先来看一下异步 Action 的工作流程 ⬇️ 。

## 异步数据流

![redux.gif](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/da4145f06eb24bca8d87b63708db6e78~tplv-k3u1fbpfcp-watermark.image)

对比同步数据流，显而易见的差别是 **多了一个 Middlewares（中间件）**。

在异步数据流中，即 Redux 的异步请求，我们发现 `store.dispatch()` 被重新封装并添加了新功能，Action 不再直接发射给 reducer，而是需要经过 Middlewares 预处理。

## Middleware（中间件）

通过使用指定的 Middleware 可以截获某种类型的 Action（比如 redux-thunk 可以截获 ajax 类型），截获后会通过 API 进行预处理，最后再将结果 dispatch 给 Reducer。

简单来说，Middleware 就是一个改造了 `store.dispatch` 的函数。他的任务就是：

- 截获 action
- 处理 action
- 发出 action

**要注意，Actions 发射的是标准 action，Middlewares 返回的也是标准 action。** （回想一下说在前面的话）

## 开发实践

我们来开发一个异步的应用：使用 Reddit API 来获取 reactjs 相关的文章列表。它就像这样 ⬇️

![rekkit.gif](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/171bb0b95c9347719b79877fdd67dfcd~tplv-k3u1fbpfcp-watermark.image)

我们会用到的依赖：

- 用 [axios](https://github.com/axios/axios) 来发送请求
- 用 [redux-thunk](https://github.com/reduxjs/redux-thunk) 处理异步 Action
- 用 [redux-logger](https://github.com/LogRocket/redux-logger) 帮我们打印派发 action 的日志

### 准备工作

- 创建 react 项目

```
npx create-react-app my_redux-thunk
```

- 安装 redux & react-redux

```
yarn add --dev redux react-redux
```

- 安装 redux-thunk & redux-logger

```
yarn add --dev redux-thunk redux-logger
```

- 安装 axios

```
yarn add axios
```

### 入口

#### `index.js`

```js
import React from "react";
import ReactDOM from "react-dom";
import Root from "./containers/Root";

import "./index.css";
import reportWebVitals from "./reportWebVitals";

ReactDOM.render(
  <React.StrictMode>
    <Root />
  </React.StrictMode>,
  document.getElementById("root")
);

reportWebVitals();
```

### Action Creators 和 Constants

#### `store/actions.js`

```js
import axios from "axios";

export const FETCH_POSTS_REQUEST = "FETCH_POSTS_REQUEST";
export const FETCH_POSTS_SUCCESS = "FETCH_POSTS_SUCCESS";
export const FETCH_POSTS_FAILURE = "FETCH_POSTS_FAILURE";

// 请求开始的 action
function fetchPostsRequest() {
  return {
    type: FETCH_POSTS_REQUEST,
  };
}

// 请求成功的 action
function fetchPostsSuccess(json) {
  return {
    type: FETCH_POSTS_SUCCESS,
    posts: json.data.children.map((child) => child.data),
    receivedAt: Date.now(),
  };
}

// 请求失败的 action
function fetchPostsFailure(error) {
  return {
    type: FETCH_POSTS_FAILURE,
    error,
  };
}

/*
 ** 异步 Action
 ** 返回一个函数
 */
function fetchPosts() {
  return (dispatch) => {
    // 请求开始 action
    dispatch(fetchPostsRequest());
    return new Promise((resolve, reject) => {
      const doRequest = axios.get("https://www.reddit.com/r/reactjs.json");
      doRequest
        // 请求成功 action
        .then((res) => {
          dispatch(fetchPostsSuccess(res.data));
          resolve(res);
        })
        // 请求失败 action
        .catch((err) => {
          dispatch(fetchPostsFailure(err.message));
          reject(err);
        });
    });
  };
}

function shouldFetchPosts(state) {
  if (state.isFetching) {
    return false;
  } else {
    return true;
  }
}

export function fetchPostsIfNeeded() {
  return (dispatch, getState) => {
    // 判断是否需要 fetch
    if (shouldFetchPosts(getState())) {
      return dispatch(fetchPosts());
    }
  };
}
```

### Reducers

#### `store/reducers.js`

```js
import {
  FETCH_POSTS_REQUEST,
  FETCH_POSTS_SUCCESS,
  FETCH_POSTS_FAILURE,
} from "./actions";

export default function postsReducer(
  state = {
    isFetching: false,
    items: [],
  },
  action
) {
  switch (action.type) {
    case FETCH_POSTS_REQUEST:
      return {
        ...state,
        isFetching: true,
        error: null,
      };
    case FETCH_POSTS_SUCCESS:
      return {
        ...state,
        isFetching: false,
        items: action.posts,
        lastUpdated: action.receivedAt,
        error: null,
      };
    case FETCH_POSTS_FAILURE:
      return {
        ...state,
        isFetching: false,
        error: action.error,
      };
    default:
      return state;
  }
}
```

### Store

#### `store/store.js`

```js
import { createStore, applyMiddleware } from "redux";
import thunkMiddleware from "redux-thunk";
import logger from "redux-logger";
import postsReducer from "./reducers";

export default function configureStore(preloadedState) {
  return createStore(
    postsReducer,
    preloadedState,
    applyMiddleware(thunkMiddleware, logger)
  );
}
```

### 容器组件

#### `containers/AsyncApp.jsx`

```jsx
import React, { Component } from "react";
import { connect } from "react-redux";
import { fetchPostsIfNeeded } from "../store/actions";
import Picker from "../components/Picker";
import Posts from "../components/Posts";

class AsyncApp extends Component {
  constructor(props) {
    super(props);
    this.handleFetch = this.handleFetch.bind(this);
  }

  componentDidMount() {
    this.handleFetch();
  }

  handleFetch() {
    this.props.fetchPostsIfNeeded();
  }

  render() {
    const { items, isFetching, lastUpdated, error } = this.props;
    return (
      <div>
        <Picker onFetch={this.handleFetch} />
        <p>
          {error && <span style={{ color: "red" }}>Fail To Load: {error}</span>}
        </p>
        <p>
          {lastUpdated && (
            <span>
              Last updated at {new Date(lastUpdated).toLocaleTimeString()}.{" "}
            </span>
          )}
        </p>
        {}
        {!isFetching && items.length === 0 && <h2>Empty.</h2>}
        {items.length > 0 && (
          <div style={{ opacity: isFetching ? 0.5 : 1 }}>
            <Posts items={items} />
          </div>
        )}
      </div>
    );
  }
}

function mapStateToProps(state) {
  const { isFetching, lastUpdated, items, error } = state || {
    isFetching: true,
    items: [],
  };

  return {
    items,
    isFetching,
    lastUpdated,
    error,
  };
}

const mapDispatchToProps = { fetchPostsIfNeeded };

export default connect(mapStateToProps, mapDispatchToProps)(AsyncApp);
```

#### `contianers/Root.jsx`

```jsx
import React, { Component } from "react";
import { Provider } from "react-redux";
import configureStore from "../store/store";
import AsyncApp from "./AsyncApp";

const store = configureStore();

export default class Root extends Component {
  render() {
    return (
      <Provider store={store}>
        <AsyncApp />
      </Provider>
    );
  }
}
```

### 展示组件

#### `components/Picker.js`

```jsx
import React, { Component } from "react";

export default class Picker extends Component {
  render() {
    const { onFetch } = this.props;

    return (
      <div>
        <h1>获取 Reddit 上 Reactjs 相关文章列表</h1>
        <button onClick={() => onFetch("reactjs")}>点击我 fetch 列表</button>
      </div>
    );
  }
}
```

#### `components/Posts.js`

```js
import React, { Component } from "react";

export default class Posts extends Component {
  render() {
    return (
      <ul>
        {this.props.items.map((item, i) => (
          <li key={i}>{item.title}</li>
        ))}
      </ul>
    );
  }
}
```

## 参考资料

- 异步 Action：https://cn.redux.js.org/docs/advanced/AsyncActions.html
- Middleware：https://cn.redux.js.org/docs/advanced/Middleware.html
- redux-thunk：https://github.com/reduxjs/redux-thunk
