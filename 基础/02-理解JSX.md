## 什么是 JSX

**JSX**，是一个 `JavaScript` 的语法扩展；在 `JavaScript` 代码中可以直接写 `HTML`的标记。

```jsx
const name = "react";
const element = <h1>hello {name}</h1>;
```

**JSX 的本质：** 并不是模版引擎，而是动态创建组件的语法糖。

```js
const name = "react";
const element = React.createElement(
  h1, // type
  null, // attribute
  "hello ", // ...children
  name
);
```

## 如何使用

> **Expressions：** any valid unit of code that resolves to a value.
> **表达式：** 会返回值的任何有效代码单元。

#### 1. `JSX` 本身就是表达式

```jsx
const element = <h1>hello jsx</h1>;
```

#### 2. 在属性中使用表达式

```jsx
<h1 foo={1 + 2 + 3 + 4}></h1>
```

#### 3. 支持扩展运算符（...）

```jsx
const props = { firstName: "", lastName: "" };
<MyComponent {...props} />;
```

#### 4. 表达式作为子元素

`props.children` 和其他 `prop` 一样，它可以传递任意类型的数据，而不仅仅是 `React` 已知的可渲染类型。

例如，如果你有一个自定义组件，你可以把回调函数作为 `props.children` 进行传递，但要确保返回值必须是可 `render` 的节点：

```jsx
// 调用子元素回调 numTimes 次，来重复生成组件
function Repeat(props) {
  let items = [];
  for (let i = 0; i < props.numTimes; i++) {
    items.push(props.children(i));
  }
  return <div>{items}</div>;
}

function ListOfTenThings() {
  return (
    <Repeat numTimes={10}>
      {(index) => <div key={index}>This is item {index} in the list</div>}
    </Repeat>
  );
}
```

这种用法并不常见，但可以用于扩展 JSX。

#### 5. 使用点语法

当你在一个模块中导出许多 `React` 组件时，这会非常方便。例如，如果 `MyComponents.DatePicker` 是一个组件，你可以在 `JSX` 中直接使用：

```jsx
import React from "react";

const MyComponents = {
  DatePicker: function DatePicker(props) {
    return <div>Imagine a {props.color} datepicker here.</div>;
  },
};

function BlueDatePicker() {
  return <MyComponents.DatePicker color="blue" />;
}
```

## 约定

自定义组件使用大写字母开头。

- `React` 默认小写的 `tag` 为原生 `DOM` 节点，如 `div`；
- 所有大写字母开头的都是自定义组件，`React` 会去寻找它的定义并 `render`。

## 这是不被允许的

不能将通用表达式作为 `React` 元素类型。

```jsx
import React from 'react';
import { PhotoStory, VideoStory } from './stories';

const components = {
  photo: PhotoStory,
  video: VideoStory
};

function Story(props) {
  // 错误！JSX 类型不能是一个表达式。
  return <components[props.storyType] story={props.story} />;
}
```

如果遇到需要根据 `prop` 来渲染不同组件的情况，可以赋值给大写字母开头的变量

```jsx
function Story(props) {
  // 正确！JSX 类型可以是大写字母开头的变量。
  const SpecificStory = components[props.storyType];
  return <SpecificStory story={props.story} />;
}
```

## 优点

- 声明式创建界面更直观（类 `HTML`）
- 代码动态创建界面更灵活
- 语法和 js 相同，无需学习新的模版语言

## 参考资料

- JSX 简介：https://zh-hans.reactjs.org/docs/introducing-jsx.html
- 深入 JSX：https://zh-hans.reactjs.org/docs/jsx-in-depth.html
