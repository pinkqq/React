## 背景

随着前端技术功能越来越强大，开发也变得越来越复杂：

- 一个单独的功能需要多个文件组成（组件、action、reducer、router）
- 代码模版复杂
- 重构困难
- 项目复杂后很难理解和维护

## Rekit：更好的代码导航

1. 语义化组织源代码文件
2. 使用子 Tab 来展示项目元素的各个部分
3. 直观的显示和导航某个功能的依赖

![rekit.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/57ad252c1f9246e9ae2bfba8bfbab804~tplv-k3u1fbpfcp-watermark.image)

## Rekit：一键生成项目元素

1. 可以通过直观的 UI 方式，用于生成组件、action、reducer 等

2. 模版代码遵循最佳实践
3. 支持命令行方式创建项目元素

log：
![ui.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d5c7f3df47aa41f6a1b20b4a9b73fc90~tplv-k3u1fbpfcp-watermark.image)

## Rekit：重构非常容易

1. 右键菜单重命名或者删除某个项目元素
2. 所有代码都会一次性重构从而保证一致性
3. 详细的 log 信息显示重构的完整修改

## Rekit：可视化的项目结构

1. 项目总体架构的可视化图表
2. 项目依赖关系的图表

![charts.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b3103701c98c4ee49afa9b81bfe45ff1~tplv-k3u1fbpfcp-watermark.image)

## Rekit 如何工作？

1. 定义了基于 feature 的可扩展文件架结构
2. 基于最佳实践生成代码和管理项目元素
3. 提供工具和 IDE 来确保代码和文夹结构始终遵循最佳实践

## 遵循最佳实践

1. 以 feature 方式组织代码
2. 拆分组件、action、reducer
3. 拆分路由配置

## 通过代码自动生成来保持一致性

1. 文件夹结构一致性
2. 文件名一致性：组件 -- 大写字母开头，action -- 驼峰
3. 变量名一致性
4. 代码逻辑一致性

## 参考资料

- rekit：https://github.com/rekit/rekit
