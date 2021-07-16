### react 是怎么保证多个 useState 的相互独立的？

我们调用了三次 useState，每次我们传的参数只是一个值（如 42，‘banana’），我们根本没有告诉 react 这些值对应的 key 是哪个，那 react 是怎么保证这三个 useState 找到它对应的 state 呢？

答案是，react 是根据 useState 出现的顺序来定的。

鉴于此，**react 规定我们必须把 hooks 写在函数的最外层**，不能写在 ifelse 等条件语句当中，来确保 hooks 的执行顺序一致。

## 参考

- [30 分钟精通 React Hooks](https://juejin.cn/post/6844903709927800846#heading-2)
