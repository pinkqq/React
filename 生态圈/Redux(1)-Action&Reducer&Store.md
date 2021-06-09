> 这是我参与更文挑战的第 3 天，活动详情查看： [更文挑战](https://juejin.cn/post/6967194882926444557)

## 诞生

`Redux` 是一个完整前端状态管理框架，它的灵感来源于 `Flux` 的几个重要特性（更新逻辑集中管理、不允许程序直接修改数据），最早由 `Dan` 大神在 2015 年提出。

随着 `JavaScript` 单页应用开发日趋复杂，伴随着 `state` 数量的增长以及 `model` 和 `view` 之前多对多的关系，使得 `state` 变得难以控制且不可预测。而 Redux 就是来处理这些问题的。

## 三大特性

### 一、单一数据源（Single Source of Truth）

`Redux` 将组件中的 `state` 都抽离出来（`object tree`），放入一个外部 `store`，**全局只有一个唯一的 store，store 负责提供应用所有的状态。** 这可以让组件间通讯更加容易，不再需要考虑组件间的层级关系、如何传递数据等问题，所有的组件只需和 `store` 通讯，`store` 的变化会引起相关组件的变化。

此外，受益于单一的 `state tree` ，使 bug 的追踪更加准确，只要检查当前状态是否正确；还有以前难以实现的如“撤销/重做”这类功能也变得轻而易举。

![redux.svg](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f65d5f1d63b14d83ae5a36f883973a8f~tplv-k3u1fbpfcp-watermark.image)
（图片来源：https://css-tricks.com/learning-react-redux/）

---

### 二、State 是只读的（State is Read-Only）

`store` 本身只有四个方法：

- `store.dispatch(action)`
- `store.subscribe(listener)`
- `store.getState()`
- `replaceReducer(nextReducer)`

正如你所见，没有一个方法用于修改 `state` 的（`setting state`），
**唯一改变 `state` 的方法就是触发 `action`**（一个用于描述已发生事件的普通对象）。

```js
var action = {
  type: "ADD_USER",
  user: { name: "Dan" },
};

// 假设已经创建了一个 store 对象
store.dispatch(action);
```

强制使用 `action` 来描述所有变化的好处是可以清晰地知道到底 **发生了什么**，它们可以被日志打印、序列化、储存、后期调试或测试时回放出来。

## 三、纯函数更新 store

> 纯函数：输出结果完全取决于输入参数，便于测试和预测结果。

**为了描述 action 如何改变 state tree ，你需要编写 reducers。** `reducer` 是一个接收 `state` 和 `action`，并返回新的 `state` 的函数 -- **State + Action = New State** 。

```js
// Reducer Function
var someReducer = function(state, action) {
  ...
  return state;
}
```

正因为 `reducer` 只是函数，你可以控制它们被调用的顺序，传入附加数据，甚至编写可复用的 `reducer` 来处理一些通用任务，如分页器。

# 理解 Action

**Action** 只是一个描述变化的普通对象，它是改变 `store` 数据的唯一方式。通过 `store.dispatch(action)` 将数据从应用传到 `store`。

我们约定，`action` 内必须使用一个字符串类型的 `type` 字段来表示将要执行的动作（`reducer` 通过 `type` 识别）。比如：

```js
{
  type:'ADD_TODO',
  text: 'Build my first Redux app'
}
```

除了 `type` 字段外，action 对象的结构完全由你自己决定。

## 一些建议

### 1、 样板文件使用

多数情况下，`type` 会被定义成字符串常量。对于小应用来说，使用字符串做 `action type` 更方便些。不过当应用规模越来越大时，建议使用单独的模块或文件来存放 `action`。

```js
// actionTypes.js
const ADD_TODO = "ADD_TODO";
```

```js
import { ADD_TODO } from "../actionTypes";

// action
{
  type: ADD_TODO,
  text: 'Build my first Redux app'
}
```

这样做有什么好处？

- 有助于维护 **命名的一致性** ，因为所有的 `action type` 汇总在同一位置。
- **避免重复命名**（可能已经有人添加了你所需要的 action）。
- 帮助团队中的所有人 **及时追踪新功能** 的范围与实现。在 `Pull Request` 中能查到所有添加，删除，修改 `Action types` 列表的记录

### 2、 如何构造 action（参照 Flux 标准）

**Action 必须是：**

- 一个普通 JavaScript 对象
- 使用一个字符串类型的 `type` 字段：作为将要执行的动作的标识（ID）

**Action 可以有：**

- `error` 字段：除了 `true` 以外的值（包括 `undefined` 和 `null`），应该都为 `false`
- `payload` 字段：

  - 任何不属于动作类型或状态的动作信息；

  - `error` 为 `true` 时，`payload` 作为错误信息

- `meta` 字段：`payload` 以外的信息

**Action 不应该包含除了 `type`、`error`、`payload`、`meta` 以外的其他字段。**

## Action Creator（创建函数）

Action Creator（创建函数）就是生成 action 的方法。

```js
function addTodo(text) {
  return {
    type: ADD_TODO,
    text,
  };
}
```

这样做将使 `action` 创建函数更容易被移植和测试。

`Redux` 中把 `action` 创建函数的结果传给 `dispatch()` 方法即可发起一次 `dispatch` 过程。

```js
dispatch(addTodo(text));
```

或者创建一个 **被绑定的 action 创建函数 来自动** dispatch，然后直接调用它们：

```js
const boundAddTodo = (text) => dispatch(addTodo(text));
boundAddTodo(text);
```

### bindActionCreators

> **actionCreators (Function or Object)**: 一个 action creator，或者一个 value 是 action creator 的对象。
>
> **dispatch (Function)**: 一个由 Store 实例提供的 dispatch 函数。
>
> **bindActionCreators(actionCreators, dispatch)**

`bindActionCreators()` 作为工具函数，可以帮助我们更方便地使用 `action` ；它可以自动把多个 `action` 创建函数 绑定到 `dispatch()` 方法上。

当你需要把 `action creator` 往下传到一个组件上，却不想让这个组件觉察到 `Redux` 的存在，而且不希望把 `dispatch` 或 `Redux store` 传给它，可以使用 `bindActionCreators`。

```js
addTodo = bindActionCreators(addTodo, store.dispatch);
addTodo();

// 等同于 dispatch(addTodo(text));
```

# 理解 Reducer

`Reducer` 是根据 `action` 更新 `state` 的纯函数，它接收旧的 `state` 和 `action`，返回新的 `state`。

```
;(previousState, action) => newState
```

![reducer.svg](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/13bfd3bccddd40c7906db536546cf67b~tplv-k3u1fbpfcp-watermark.image)
保持 `reducer` 纯净非常重要。永远不要在 `reducer` 里做这些操作：

- 修改传入参数；
- 执行有副作用的操作，如 API 请求和路由跳转；
- 调用非纯函数，如 `Date.now()` 或 `Math.random()`

```js
const initialState = {
  visibilityFilter: "SHOW_ALL",
};

// action
{
  type: 'SET_VISIBILITY_FILTER',
  filter: 'SHOW_COMPLETED'
}

// reducer
// 为 state 指定默认值 initialState
function todoApp(state = initialState, action) {
  switch (action.type) {
    case 'SET_VISIBILITY_FILTER':
      return  {
        ...state,
        visibilityFilter: action.filter,
      };
    default:
      return state;
  }
}
```

实际开发中需要注意：

- **不要修改 state。** 使用 `Object.assign()` 或者 对象展开运算符(`{ ...state, ...newState }`) 新建了一个副本。
- **在 default 情况下返回旧的 state。**

## 拆分 Reducer

![reducer.svg](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ffb7305af2ca4a16a22a1cfcada03e3e~tplv-k3u1fbpfcp-watermark.image)

当有多个 action 时，就像这样：

```js
function todoApp(state = initialState, action) {
  switch (action.type) {
    case SET_VISIBILITY_FILTER:
      return {
        ...state,
        visibilityFilter: action.filter,
      };
    case ADD_TODO:
      return {
        ...state,
        todos: [
          ...state.todos,
          {
            text: action.text,
            completed: false,
          },
        ],
      };
    case TOGGLE_TODO:
      return {
        ...state,
        todos: state.todos.map((todo, index) => {
          if (index === action.index) {
            return {
              ...state,
              completed: !todo.completed,
            };
          }
          return todo;
        }),
      };
    default:
      return state;
  }
}
```

这样的代码显得冗余且不易阅读，就该案例我们把 `todos` 和 `visibilityFilter` 拆分开：

```js
function todos(state = [], action) {
  switch (action.type) {
    case ADD_TODO:
      return [
        ...state,
        {
          text: action.text,
          completed: false,
        },
      ];
    case TOGGLE_TODO:
      return state.map((todo, index) => {
        if (index === action.index) {
          return {
            ...state
            completed: !todo.completed,
          };
        }
        return todo;
      });
    default:
      return state;
  }
}

function visibilityFilter(state = SHOW_ALL, action) {
  switch (action.type) {
    case SET_VISIBILITY_FILTER:
      return action.filter;
    default:
      return state;
  }
}

function todoApp(state = {}, action) {
  return {
    visibilityFilter: visibilityFilter(state.visibilityFilter, action),
    todos: todos(state.todos, action),
  };
}
```

**注意每个 reducer 只负责管理全局 state 中它负责的一部分。每个 reducer 的 state 参数都不同，分别对应它管理的那部分 state 数据。**

### combineReducers

> combineReducers(reducers)

`combineReducers` 的作用是，把一个由多个不同 `reducer` 函数作为 `value` 的 `object`，合并成一个最终的 `reducer` 函数。

```js
import { combineReducers } from "redux";

const todoApp = combineReducers({
  visibilityFilter,
  todos,
});

export default todoApp;
```

上面的写法和下面完全等价：

```js
export default function todoApp(state = {}, action) {
  return {
    visibilityFilter: visibilityFilter(state.visibilityFilter, action),
    todos: todos(state.todos, action),
  };
}
```

# 理解 Store

**Redux 的核心就是 只有一个单一的 store。**

`Store` 可以说是一个虚拟的概念，它是把 `state`、`dispatcher(action)`、`reducer` 联系到一起的对象。`Store` 有以下职责：

- 维持应用的 state；
- 提供 `getState()` 方法获取 state；
- 提供 `dispatch(action)` 方法更新 state；
- 通过 `subscribe(listener)` 注册监听器;
- 通过 `subscribe(listener)` 返回的函数注销监听器。

![store.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6921e4759daa4081a57f18256bff8934~tplv-k3u1fbpfcp-watermark.image)

## 创建 Store（createStore）

> createStore(reducer, [preloadedState], enhancer)

参数：

1. reducer (Function)
2. [preloadedState] (any): 初始时的 state。
3. enhancer (Function): Store enhancer 是一个组合 store creator 的高阶函数，返回一个新的强化过的 store creator

## 发起 Actions

```js
import { createStore } from "redux";

function todos(state = [], action) {
  switch (action.type) {
    case "ADD_TODO":
      return state.concat([action.text]);
    default:
      return state;
  }
}

let store = createStore(todos, ["Use Redux"]);

// 每次 state 更新时，打印日志
// 注意 subscribe() 返回一个函数用来注销监听器
const unsubscribe = store.subscribe(() => console.log(store.getState()));

// 发起 action
store.dispatch({
  type: "ADD_TODO",
  text: "Read the docs",
});
// 打印：[ 'Use Redux', 'Read the docs' ]

// 停止监听 state 更新
unsubscribe();
```

因为都是纯函数，在还没有开发界面的时候，我们就可以定义程序的行为。而且这时候已经可以写 `reducer` 和 `action` 创建函数的测试。

## 参考资料

- Redux 中文文档：https://cn.redux.js.org/
- Redux 官方网站：https://redux.js.org/
- Leveling Up with React: Redux：https://css-tricks.com/learning-react-redux/
