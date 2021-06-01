## 什么是 Context API

其实，`Context API` 在 `React` 的历史上一直实验性的存在，直到 `React 16.3` 之后正式成为了一个 `Public API`。

`Context` 是组件间通讯的一种解决方案，它提供了一种在组件之间共享数据的方式，避免了手动添加 props、数据层层传递的麻烦。

### 应用场景

`Context` 主要应用场景在于**很多不同层级**的组件需要访问一些**相同**的数据。如下图：

左侧是一颗组件树，子节点们（`consumer`）共享一个全局数据，P 结点（`provider`）作为根结点提供全局数据。如果通过 props 属性自上而下（ `P -> B -> ... -> C` ）进行传递，过程繁琐且不易维护。

如果使用 `Context`，P 结点（`provider`）只要将数据一股脑儿丢给 `Context`，子节点们（`consumer`）通过 `Context` 就可以直接访问数据。

![react-context.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/23ebcf59a68e431fb329a92a7cdd9657~tplv-k3u1fbpfcp-watermark.image)

## 基础使用

```jsx
import React from "react";

// 创建一个 Context 对象。
const ThemeContext = React.createContext("light");

// 提供初始 context 值的 App 组件
class App extends React.Component {
  render() {
    return (
      <ThemeContext.Provider value="dark">
        <ThemeComponent />
        <ThemeButton />
      </ThemeContext.Provider>
    );
  }
}

// contextType 属性能让你使用 this.context 来获取 Context 的值
class ThemeComponent extends React.Component {
  render() {
    return <div>{this.context}</div>;
  }
}

ThemeComponent.contextType = ThemeContext;

// Context.Consumer  是订阅 Context 的另一种方式，
// 可以在函数式组件中使用
function ThemeButton(props) {
  return (
    <ThemeContext.Cousmer>
      {(theme) => <Button {...props} theme={theme}></Button>}
    </ThemeContext.Cousmer>
  );
}
```

## API

### - React.createContext

```jsx
const MyContext = React.createContext(defaultValue);
```

创建一个 `Context` 对象。

只有当组件所处的树中没有匹配到 `Provider` 时，其 `defaultValue` 参数才会生效。比如 `<MyContext.Consumer>` 没有被 `<MyContext.Provider>` 包裹。

### - Context.Provider

```jsx
<MyContext.Provider value={/* 某个值 */}>
```

每个 `Context` 对象都会返回一个 Provider React 组件，它允许消费组件（Consumer）订阅 context 的变化。

`Provider` 接收一个 `value` 属性，传递给消费组件。一个 `Provider` 可以和多个消费组件有对应关系。多个 `Provider` 也可以嵌套使用，里层的会覆盖外层的数据。

当 `Provider` 的 `value` 值发生变化时，它内部的所有消费组件都会重新渲染。`Provider` 及其内部 `consumer` 组件都不受制于 `shouldComponentUpdate` 函数，因此当 `consumer` 组件在其祖先组件退出更新的情况下也能更新。

使用了与 `Object.is` 相同的算法，来判断新旧值是否变化：

> 如果以下条件之一成立，则两个值相同：
>
> - 都是 undefined
> - 都是 null
> - 都是 true 或都是 false
> - 两者都是字符串，**长度相同**、**字符相同**且字符的**排列顺序相同**
> - 两者都是同一个对象（意味着两个值都引用了内存中的同一个对象）
> - 两者都是数字，并且
>   - 都是 +0
>   - 都是 -0
>   - 都是 NaN
>   - 非零、非 NaN 且两者值相等

### - Class.contextType

```jsx
class MyClass extends React.Component {
  componentDidMount() {
    let value = this.context;
    /* 在组件挂载完成后，使用 MyContext 组件的值来执行一些有副作用的操作 */
  }
  componentDidUpdate() {
    let value = this.context;
    /* ... */
  }
  componentWillUnmount() {
    let value = this.context;
    /* ... */
  }
  render() {
    let value = this.context;
    /* 基于 MyContext 组件的值进行渲染 */
  }
}

MyClass.contextType = MyContext;
```

此属性能让你使用 `this.context` 来消费最近 `Context` 上的那个值。你可以在任何生命周期中访问到它，包括 `render` 函数中。

### - Context.Consumer

```jsx
<MyContext.Consumer>
  {value => /* 基于 context 值进行渲染*/}
</MyContext.Consumer>
```

`Context.Consumer` 是订阅 `Context` 的另一种方式，该组件可以在函数式组件中使用。

这种方法需要一个函数作为子元素， `value` 值等价于组件树上方离这个 `context` 最近的 `Provider` 提供的 `value` 值（没有 `Provider`，用 `defaultValue`）。

### - Context.displayName

`React DevTools` 使用该字符串来确定 `context` 要显示的内容。

```jsx
const ThemeContext = React.createContext(themes.light);
ThemeContext.displayName = "MyDisplayName";

<ThemeContext.Provider> // "MyDisplayName.Provider" 在 DevTools 中
```

## 参考资料

- Context：https://zh-hans.reactjs.org/docs/context.html
- 组合 vs 继承：https://zh-hans.reactjs.org/docs/composition-vs-inheritance.html
- Render Props：https://zh-hans.reactjs.org/docs/render-props.html
- Object.is：https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is#description
- JavaScript 中的相等性判断：https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Equality_comparisons_and_sameness
- 过时的 context 文档：https://zh-hans.reactjs.org/docs/legacy-context.html
