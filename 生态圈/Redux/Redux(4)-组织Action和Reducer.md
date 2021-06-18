## 为什么想找新形式

在 `Redux` 官方的入门示例中，我们会把所有 `Action` 放在一个文件、所有 `Reducer` 放在一个文件，这样的“标准”形式会带来什么问题呢：

- 所有 `Action` / `Reducer` 放在一个文件中，会无限扩展（文件过大）
- `Action` 和 `Reducer` 分开，实现业务逻辑是需要来回切换（在 `Action` 中发请求，在 `Reducer` 中做处理）
- 系统中有哪些 `Action` 不够直观（只能打开相关文件查看）

## 新形式是什么

可以将单个 `Action` 和 `Reducer` 放在同一个文件

- **易与开发**：不用在 `action` 文件和 `reducer` 文件间来回切换
- **易于维护**：每个 `action` 都很小
- **易于测试**：每个业务逻辑只需对应一个测试文件
- **易于理解**：文件名是 `action` 名，文件列表就是 `action` 列表

`counterPlusOne.js`

```js
import { COUNTER_PLUS_ONE } from "./constants";
export function counterPlusOne() {
  return {
    type: "COUNTER_PLUS_ONE",
  };
}
export function reducer(state, action) {
  switch (action.type) {
    case "COUNTER_PLUS_ONE":
      return {
        ...state,
        count: state.count++,
      };
    default:
      return state;
  }
}
```

## 新形式演示

我们以新方式改造一下 Redux 官方的 [todos 示例](https://github.com/reduxjs/redux/tree/master/examples/todos)。

前后目录对比：

![mulu.jpg](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/244cbd6cb10748b5baead5db3f54df75~tplv-k3u1fbpfcp-watermark.image)

### store

#### `store/actions.js`

没有了静态类型和 `action creator`，`action.js` 中只有 `actions` 的名称列表，项目中有哪些 Action 一目了然。

```js
export { addTodo } from "../actions/addTodo";
export { setVisibilityFilter } from "../actions/setVisibilityFilter";
export { toggleTodo } from "../actions/toggleTodo";
export { VisibilityFilters } from "../actions/visibilityFilters";
```

#### `store/reducers.js`

```js
import { combineReducers } from "redux";
import { reducer as addTodoReducer } from "../actions/addTodo";
import { reducer as toggleTodoReducer } from "../actions/toggleTodo";
import { reducer as visibilityFilter } from "../actions/setVisibilityFilter";

const todoRreducers = [addTodoReducer, toggleTodoReducer];

const initialState = [
  {
    id: 0,
    text: "Start to use Rekit!",
    completed: false,
  },
];

function todos(state = initialState, action) {
  let newState;
  switch (action.type) {
    // Handle cross-topic actions here
    default:
      newState = state;
      break;
  }
  return todoRreducers.reduce((s, r) => r(s, action), newState);
}

export default combineReducers({ todos, visibilityFilter });
```

### actions

一个文件对应一个业务逻辑，是完整的 `action` + `reducer` 流程。

#### `actions/addTodo.js`

```js
import { ADD_TODO } from "../constants";

export const addTodo = (text) => ({
  type: ADD_TODO,
  text,
});

export const reducer = (state = [], action) => {
  switch (action.type) {
    case "ADD_TODO":
      return [
        ...state,
        {
          id: state.reduce((maxId, todo) => Math.max(todo.id, maxId), -1) + 1,
          text: action.text,
          completed: false,
        },
      ];
    default:
      return state;
  }
};
```

#### `actions/setVisibilityFilter.js`

```js
import { SET_VISIBILITY_FILTER } from "../constants";
import { VisibilityFilters } from "./visibilityFilters";

export const setVisibilityFilter = (filter) => ({
  type: SET_VISIBILITY_FILTER,
  filter,
});

export const reducer = (state = VisibilityFilters.SHOW_ALL, action) => {
  switch (action.type) {
    case "SET_VISIBILITY_FILTER":
      return action.filter;
    default:
      return state;
  }
};
```

#### `actions/toggleTodo.js`

```js
import { TOGGLE_TODO } from "../constants";

export const toggleTodo = (id) => ({
  type: TOGGLE_TODO,
  id,
});
export const reducer = (state, action) => {
  switch (action.type) {
    case "TOGGLE_TODO":
      return state.map((todo) =>
        todo.id === action.id ? { ...todo, completed: !todo.completed } : todo
      );
    default:
      return state;
  }
};
```

#### `actions/visibilityFilters.js`

```js
import { SHOW_ALL, SHOW_COMPLETED, SHOW_ACTIVE } from "../constants";

export const VisibilityFilters = {
  SHOW_ALL: SHOW_ALL,
  SHOW_COMPLETED: SHOW_COMPLETED,
  SHOW_ACTIVE: SHOW_ACTIVE,
};
```
