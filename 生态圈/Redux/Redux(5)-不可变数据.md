## 什么是「不可变数据」

![immutabledata.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b2f626ae146e455b8e48bd02a7b4371b~tplv-k3u1fbpfcp-watermark.image)

**不可变数据（immutable data）**：就是一旦创建，就不能更改的数据。

**不可变数据 实现的原理是 `Persistent Data Structure`（持久化数据结构）**，就是说在进行修改时，是返回一个新的 Immutable 对象，以此来保证旧数据同时可用且不变。

**不可变数据 采用了 Structural Sharing（结构共享）**，可以避免把所有节点都复制一遍带来的性能损耗，即只修改变化节点和受它影响的父节点，其它节点则进行共享。

如上图，我们要对 `initial tree` 进行修改的话，要先复制一颗一模一样的树，然后融合要修改的内容（只有修改部分及其路径变化，其他部分是不变的），产生右边的 `updated tree`。这样数据修改前后的引用地址是不同的，我们就以此来判断数据是否发生变化。

## 为什么需要「不可变数据」

**使用不可变数据是 Redux 的运行基础**，因为它：

- **优化性能**：不需要对比 Store 值的变化，只要判断引用是否相等
- **易于调试和测试**：因为前后的数据都不同，可以把这些数据放到一个数组里储存起来，随时使用。
- **节省内存空间**：通过结构复用，可以尽量复用内存，甚至以前使用的对象也可以再次被复用。没有被引用的对象会被垃圾回收。

## 如何操作「不可变数据」

介绍三种典型且不同的方式，它们有各自的优缺点和适用场景。

### 原生（ {...} , Object.assign ）

```js
const state = { filter: "completed", todos: ["Learn React"] };
// ...
const newState1 = {
  ...state,
  todos: [...state.todos, "Learn Redux"],
};
// Object.assign
const newState2 = Object.assign({}, state, {
  todos: [...state, "Learn Redux"],
});
```

#### 优点

- 不需要额外的安装库
- 不需要学习额外的 API
- 性能高

#### 缺点

- 遇到较深的层级会很头疼，看的人眼花缭乱

  ```js
  const newData1 = { ...myData, {
    x: { ...myData.x, {
      y:  {...myData.x.y, { z: 7 }
    },
    a: { ...myData.a, { b: myData.a.b.concat(9) }
  }
  const newData2 = Object.assign({}, myData, {
    x: Object.assign({}, myData.x, {
      y: Object.assign({}, myData.x.y, { z: 7 }),
    }),
    a: Object.assign({}, myData.a, { b: myData.a.b.concat(9) }),
  });
  ```

#### 介绍一下 Object.assign

`Object.assign()` 方法用于将所有可枚举属性的值从一个或多个源对象分配到目标对象。

`Object.assign()`特别需要注意的地方:

- 如果对象的属性值为简单类型（string，number），通过 `Object.assign({},srcobj)` 得到的新对象为深拷贝；
  ```js
  // 简单类型
  var obj1 = { name: "d" };
  var obj2 = { age: 23 };
  Object.assign(obj1, obj2);
  obj2.age = 29;
  console.log(obj1); // {name: "d", age: 23}
  ```
- 如果属性值为对象或其他引用类型，那对于这个对象而言其实是浅拷贝的

  ```js
  // 引用类型
  var obj1 = { name: "d" };
  var obj2 = { age: { year: 2019 } };
  Object.assign(obj1, obj2);
  obj1.age.year = 2020;
  console.log(obj2); // {name: "d", age: {year: 2020}}
  ```

- 如果属性值为对象或其他引用类型，是不会对对象直接的属性进行合并的。

  ```js
  var obj1 = { age: { year: 2020, mouth: 8 } };
  var obj2 = { age: { year: 2019 } };
  Object.assign(obj1, obj2);
  console.log(obj1); // { age: { year: 2019 } }
  ```

> **对象的浅拷贝**：浅拷贝是对象共用的一个内存地址，对象的变化相互印象。
> **对象的深拷贝**：简单理解深拷贝是将对象放到新的内存中，两个对象的改变不会相互影响。

### Immutability-helper

> 项目地址：https://github.com/kolodny/immutability-helper

那是某一天，`Immutability-helper` 出现在 [react 官方文档](https://zh-hans.reactjs.org/docs/update.html)， 它可以看成是 `Immutable.js` 的极简版。对比 Immutable.js 的功能齐全，Immutability-helper 可能更像是 Object.assign 的语法糖，但也因此更轻量好上手。

比如这个例子：

```js
const newData = Object.assign({}, myData, {
  x: Object.assign({}, myData.x, {
    y: Object.assign({}, myData.x.y, { z: 7 }),
  }),
  a: Object.assign({}, myData.a, { b: myData.a.b.concat(9) }),
});
```

用 immutability-hepler 可以这样写

```js
import update from "immutability-hepler";
const newData = update(myData, {
  x: { y: { z: { $set: 7 } } },
  a: { b: { $push: [9] } },
});
```

#### 优点

- 可读性增加
- 适合较深层次的节点

#### 缺点

- 需要引入额外的类库
- 需要学习新的语法

#### 简单聊一下 API：

- `{$push: array}`： 类似数组的 push

  ```js
  const initialArray = [1, 2, 3];
  const newArray = update(initialArray, { $push: [4] }); // => [1, 2, 3, 4]
  ```

- `{$unshift: array}`：类似数组的 unshift
  ```js
  const initialArray = [1, 2, 3];
  const newArray = update(initialArray, { $unshift: [4] }); // => [4, 1, 2, 3]
  ```
- `{$splice: array of arrays}`：类似数组的 splice

  ```js
  const collection = [1, 2, { a: [12, 17, 15] }];
  const newCollection = update(collection, {
    2: { a: { $splice: [[1, 1, 13, 14]] } },
  });
  // => [1, 2, {a: [12, 13, 14, 15]}]
  ```

  访问 collection 的索引 2 的键 a，并在插入 13 和 14 的同时对从索引 1 开始的 1 项进行拼接（删除 17）

- `{$set: any}`：有重复的值时，覆盖，没有就新增

  ```js
  const myData = { a: "1", b: "2" };
  const newData = update(myData, {
    a: { $set: "0" },
    c: { $set: "3" },
  });
  // => {a: "0", b: "2", c: "3"}
  ```

- `{$merge: object}`：做对象的合并操作

  ```js
  const myData = { a: "1", b: "2" };
  const newData = update(myData, {
    $merge: { a: "0", c: "3" },
  });
  // => { a: "0", b: "2", c: "3" };
  ```

- `{$apply: function}`：基于当前值进行一个函数运算，从而得到新的值

  ```js
  const myData = [1, 2, 3, 4, 5];
  const newData = update(myData, {
    $apply: (val) => val.reverse(),
  });
  // => [5, 4, 3, 2, 1]
  ```

### Immer（推荐）

> 项目地址：https://github.com/immerjs/immer

immer 是 mobx 的作者在 immutable 方面做的新的尝试，[作者传送门](https://github.com/mweststrate)。immer 可以以更方便的方式处理不可变数据，它基于 copy-on-write（写时复制） 机制来优化使用效率。

**Immer 的基本工作流程：** 所有的更改会应用到一个临时的 DraftState（作为 CurrentState 的代理），Immer 再根据 DraftState 的更改情况生成 NextState。

![immer.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c44b5d7ebe1e4906a51afc961c831974~tplv-k3u1fbpfcp-watermark.image)

**immer 最大的特点在于使用原生数据结构的 API**。它的写法看起来就像是直接在修改数据：

```js
import produce from "immer";

const baseState = [
  {
    todo: "Learn typescript",
    done: true,
  },
  {
    todo: "Try immer",
    done: false,
  },
];

const nextState = produce(baseState, (draftState) => {
  draftState.push({ todo: "Tweet about it" });
  draftState[1].done = true;
});
```

#### 优点

- 十分轻量，压缩后体积只有 3kb
- 不需要学习新的 API，学习成本低

#### 性能

来自[官方的性能测试](https://immerjs.github.io/immer/performance/)数据：

![immer.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8130031a175f4dd99ab3f2d34513afe1~tplv-k3u1fbpfcp-watermark.image)

通过上图的观察，我们看到：

- Immer（Proxy）大约比手写 reducer 慢两三倍，这在实际中可以忽略不计
- Immer 的运行效率受到环境因素影响较大。Immer 的 ES5 版本中使用 defineProperty 来实现，它的测试速度明显较慢。所以尽量在支持 Proxy 的环境中使用 Immer。

**性能优化的建议：**

- **预冻结数据**

  在添加大型数据到 produce 前，先进行 `freeze(json)`。Immer 将能更快地添加新数据，因为它将避免递归扫描和冻结新数据。

- **昂贵的搜索操作，在原 state 中进行，而不是 draft**

- **灵活选择是否使用 Immer**

  可以搭配一些纯 Jacascript 的原生操作，即使是在 produce 内部。

- **尽可能的提升 produce**

  ```js
  // ❌
  for (let x of y) produce(base, (d) => d.push(x));

  // ✅
  produce(base, (d) => {
    for (let x of y) d.push(x);
  });
  ```
