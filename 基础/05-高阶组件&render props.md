`React` 组件的最大好处在于【复用】，但基础组件的编写还是无法满足一些场景，比如对相似又有差异的组件，我们是否可以再抽离出公共部分？这时就诞生了另外两种方式：高阶组件（ `HOC` ）和函数作为子组件。

## 高阶组件

高阶组件（ `HOC: Higher-Order Component` ）并不是组件，它是基于 `React` 的组合特性而形成的设计模式，是 `React` 中用于复用组件逻辑的一种高级技巧。

`HOC` 的本质是 接受组件作为参数，返回新组件的一个函数。

```js
const EnhancedComponent = higherOrderComponent(WrappedComponent);
```

`HOC` 是会对传入组件（WrappedComponent）进行一个封装（添加额外功能 / 数据），`HOC` 一般不会有自己的 `UI`。

### 组件如何获取外部资源

![未命名文件.jpg](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/248876140ffd47ffadbed94b2afc8002~tplv-k3u1fbpfcp-watermark.image)

在组件树上，父组件可以通过 `props` 传递数据给子组件。如果外部资源的起点在父组件的父组件，数据就要经过两层的 `props` 传递才能抵达目标组件，而且可能部分数据对于非目标组件都是没有用，只是为了往下传递。

这种情况下，我们可以采用另一种方式 -- **高阶组件**，直接获取外部资源。

### 举个 🌰

封装一个时间组件（ `withTimer` ），`withTimer` 只负责获取并更新时间的逻辑，在 `UI` 上展示的格式又 `WrappedComponent` 决定。

- withTimer.vue

```jsx
import React from "react";

export default function withTimer(WrapperComponent) {
  return class extends React.Component {
    state = {
      time: new Date(),
    };
    componentDidMount() {
      this.timerID = setInterval(() => this.tick(), 1000);
    }
    componentWillUnmount() {
      clearInterval(this.timerID);
    }
    tick() {
      this.setState({
        time: new Date(),
      });
    }
    render() {
      return <WrapperComponent time={this.state.time} />;
    }
  };
}
```

- OneComponent.vue

```jsx
import React from "react";
import withTimer from "./withTimer";

export class OneComponent extends React.Component {
  // ...
  render() {
    renturn(<div>{this.props.time.toLocaleString()}</div>);
  }
}

export default widthTimer(OneComponent);
```

### 使用注意

- 不要改变原始组件，否则原始组件再也无法像 `HOC` 增强之前那样使用了
- 这是一个约定。用 `HOC` 包住被包装组件的显示名称。比如高阶组件名为 `withSubscription`，并且被包装组件的显示名称为 `CommentList`，显示名称应该为 `WithSubscription(CommentList)`。
- 不要在 `render` 方法中使用 `HOC`
  ```jsx
  render() {
  // 每次调用 render 函数都会创建一个新的 EnhancedComponent
  // EnhancedComponent1 !== EnhancedComponent2
  const EnhancedComponent = enhance(MyComponent);
  // 这将导致子树每次渲染都会进行卸载，和重新挂载的操作！
  return <EnhancedComponent />;
  }
  ```
- `Refs` 不会被传递，高阶组件的约定是将所有 `props` 传递给被包装组件，但 `ref` 并不是一个 `prop`。

## Render props

**Render props** 就是指一个外部（组件使用者）可以通过它来告知组件需要渲染什么内容的 **函数 prop** ， 是实现在 `React` 组件之间共享代码的一种简单技术，使组件能在不增加自身代码的前提下提高了灵活性。

### 如何使用

**Render props** 可以是函数作为子组件（`this.props.children`），也可以是其他任何函数 `prop`，例如 `something prop`（`this.props.something`）。

```jsx
<Mouse>
{(mouse) => (
    <p>
      鼠标的位置是 {mouse.x}，{mouse.y}
    </p>
)}
</Mouse>

<Mouse
  something={(mouse) => (
    <p>
      鼠标的位置是 {mouse.x}，{mouse.y}
    </p>
  )}
/>
```

### 与 React.PureComponent 一起使用

`React.PureComponent` 中以浅层对比 `prop` 和 `state` 的方式来实现了 `shouldComponentUpdate()`，在赋予 `React` 组件相同的 `props` 和 `state` 的情况下，`React.PureComponent` 可以提高性能。

但在使用 `render prop` 时，浅比较 `props` 的时候总会得到 `false`，所以这会使 `Pure.Component` 失去优势。

## 小结

- 高阶组件和 Render props，都不是新类型的组件，而是设计模式
- 他们都是为了实现组件在更多场景的复用

## 参考资料

- 高阶组件：https://zh-hans.reactjs.org/docs/higher-order-components.html
- Render Props：https://zh-hans.reactjs.org/docs/render-props.html
