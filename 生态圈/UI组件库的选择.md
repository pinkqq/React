大部分崽崽都有了自己熟悉或者正在使用的 UI 组件库，可能是因为热门，或者缘分。其实 UI 组件库的选择对 React 应用的开发是十分重要，组件库的稳定性和完善程度直接影响开发效率和质量。

选择主流的没有什么问题，但你不想弄清楚什么它成为了主流？

目前主流的 UI 库有：`Ant Design`、`Material UI`、`Semantic UI`

## Ant Design

> https://ant.design/index-cn

目标是服务于企业级产品，适合交互复杂的场景。从设计模式到使用说明文档描述都非常详尽。

## Material UI

> https://material-ui.com/

国际上非常流行的 UI 库，设计风格更花哨，适合面向消费者的场景。

## Semantic UI

> https://semantic-ui.com/

这是一个古老的 UI 库，在 jquery 时代就有了，他的特点是 UI 以 language 的形式描述，他定义了许多的 class ，通过 class 使用实现效果。适合以前用过 semantic 的玩家。

## 选择 UI 库的考虑因素

- 组件库是否齐全
- 样式风格是否符合业务需求（企业级应用：简介明了，数据展示合理）
- API 设计是否便捷和灵活
- 技术支持是否完善（issue 处理）
- 开发是否活跃（稳定团队提供迭代和维护）

除了 `Ant Design` ，`Material UI` 和 `Semantic UI` 都没有提供 `Tree`。在组件库齐全程度上，`Ant Design` 略胜一筹。

对于 `Table` 组件，`Ant Design` 只需要提供表格内容数据，UI 的渲染交给 `Ant Design` 完成，使用者只需要关注数据，而 `Material UI` 和 `Semantic UI` 都需要使用者自己处理页面元素结构，又回到了自己写 `HTML` 的时候。所以相比较 `Ant Design` 的模式更合理。

## 小结

选 `Ant Design`。
