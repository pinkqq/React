## 为什么需要 Virtual Dom

`Virtual Dom` 对应的是 `Real Dom`，即真正展示给用户看的 UI 背后的 DOM 结构。

- 每次操作 `Real Dom`，首先需要访问 `Real Dom`，这是消耗之一；
- 修改 `Real DOM` 会导致浏览器重新计算页面的几何变化、引发浏览器模板引擎的 **重排和重绘**，只是消耗之二，且巨大。

那如果可以将多次重排整合为一次，并且能局部只更新有修改的 dom，岂不是可以大大减少消耗了呢？于是，`Virtual Dom` 就带着使命来了！

## 什么是 Virtual Dom？

[JSX](https://juejin.cn/post/6966067460755685406) 的关键就是 Virtual Dom

```jsx
const element = <h1 className="text">hello {name}</h1>;
```

```js
const element = React.createElement(
  h1, // type
  { className: "text" }, // attribute
  "hello ", // ...children
  name
);
```

`Virtual DOM` 的表示方式：

```js
let VNode = {
  tag: "h1",
  attrs: {
    className: "text",
  },
  children: ["hello", name],
};
```

`Virtual DOM` 以对象的形式模拟了 `Real DOM` 树的结构。所以它是在内存中维护的，当所有对其的修改完成后（state 或 props 更新），`React`的 `render()` 方法会返回一颗新的树。`React` 会基于新树与旧树之间的差别（ `diff` ）来判断如何高效的更新 UI。而这个 `diff` 算法也成为了性能优化的关键所在。

## 优化策略

`React` 根据 UI 更新的特点对 `diff` 算法进行优化，最终提出了一套 O(n) 的启发式算法（通用的解决方案的复杂度为 O(n 3 )）：

- 两个不同类型的元素会产生出不同的树；
- 对同类型的元素设置 key 属性，相同 key 值在不同的渲染下可以保持不变；

### 广度优先分层比较

`React` 会从根结点开始按 “层” 比较两棵树，并且不会做跨 “层” 比较。

![未命名文件 (6).jpg](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a01a34fdebfc449685d1f428525bc1c0~tplv-k3u1fbpfcp-watermark.image)

### 类型变化，直接删除

当节点的类型不同时，`React` 不会去检查在别处是否复用，而是 “简单粗暴” 直接删除旧节点 + 添加新节点。比如节点跨层移动的情况，`React` 并不会为去匹配移动节点后，为其修改 `parentNode`。

这是 `React` 在 “组件的 `DOM` 结构是相对稳定” 的前提下，做出的【性能取舍】 -- 舍弃极少数情况的性能消耗，获取大部分情况的性能提升。

![未命名文件 (7).jpg](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/52c31339c52b4cc685e95648561e824b~tplv-k3u1fbpfcp-watermark.image)

### key 属性

为类型相同的兄弟节点添加 `key` 属性，作为唯一标识符。

`React` 会根据 `key` 分辨新旧元素以及元素顺序的变化。

```jsx
<ul>
  <li key="2015">Duke</li>
  <li key="2016">Villanova</li>
</ul>

<ul>
  <li key="2014">Connecticut</li>
  <li key="2015">Duke</li>
  <li key="2016">Villanova</li>
</ul>
```

当你的列表会发生重新排序时，千万不要用下标 `index` 作为 `key`，这会引起混乱，比如非受控组件的 `state`（比如输入框）可能会相互篡改。

在 Codepen 有两个例子，分别为 [展示使用下标作为 key 时](https://codepen.io/pen?&editors=0010)导致的问题，以及[不使用下标作为 key 的例子](https://codepen.io/pen?&editors=0010)的版本，修复了重新排列，排序，以及在列表头插入的问题。

## 参考资料

- 协调：https://zh-hans.reactjs.org/docs/reconciliation.html
- VNode 结构变化时，发生了什么？：https://supnate.github.io/react-dom-diff/index.html
