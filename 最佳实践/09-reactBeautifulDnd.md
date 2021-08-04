`React-Beautiful-DND`，一个强大的拖拽包，能够优雅的做出丰富的拖拽页面应用，适用于列表之间拖拽的场景，支持移动端，且简单易上手。[官方丰富的案例展示，正合心意。](https://react-beautiful-dnd.netlify.app/?path=/story/single-vertical-list--basic)

## 核心概念

![01.gif](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6f1c825c70fb483c9b10e899c49e70b3~tplv-k3u1fbpfcp-watermark.image)

### [DragDropContext](https://github.com/atlassian/react-beautiful-dnd/blob/master/docs/api/drag-drop-context.md)

```jsx
import { DragDropContext } from 'react-beautiful-dnd';

const App = () => {
  const onDragStart = () => {
    /*...*/
  };
  const onDragUpdate = () => {
    /*...*/
  };
  const onDragEnd = () => {
    // the only one that is required
  };

  return (
    <DragDropContext
      onDragStart={onDragStart}
      onDragUpdate={onDragUpdate}
      onDragEnd={onDragEnd}
    >
      <div>Hello world</div>
    </DragDropContext>
  );
};
```

**构建一个可以拖拽的范围**，把你想能够拖放的 react 代码放到 `DragDropContext` 中。

支持的事件：

- `onDragStart`：拖拽开始回调
- `onDragUpdate`：拖拽中的回调
- `onDragEnd`：拖拽结束时的回调

  ```js
  const onDragEnd = (result) => {
    console.log(result);
    /*
    {
      draggableId: "item-3",
      // 从这里
      source:{
        droppableId: "droppable",
        index: 2
      },
      // 移到这里
      destination:{
        droppableId: "droppable",
        index: 1
      }
    }
     */
  };
  ```

> **WARNING**：`DragDropContext` 不支持嵌套，且必须设置 `DranDropContext` 的 `onDragEnd` 钩子函数(拖拽后的数组重新排序操作在这里进行)

### [Droppable](https://github.com/atlassian/react-beautiful-dnd/blob/master/docs/api/droppable.md)

```jsx
import { Droppable } from 'react-beautiful-dnd';

<Droppable droppableId="droppable-1">
  {(provided, snapshot) => (
    <div
      ref={provided.innerRef}
      style={{ backgroundColor: snapshot.isDraggingOver ? 'blue' : 'grey' }}
      {...provided.droppableProps}
    >
      <h2>I am a droppable!</h2>
      {provided.placeholder}
    </div>
  )}
</Droppable>;
```

**构建一个可以被拖拽放入的区域块**。

参数介绍：

- `DroppableId`： 此属性是必须的，用于唯一标识，不要更改此 ID。
- `direction`：`vertical`（水平拖拽）/ `horizontal`（垂直拖拽）

`Droppable` 的 React 子元素必须是返回 `ReactElement` 的函数。该函数提供了两个参数：`provided` 和 `snapshot`：

- `provided.innerRef`: 必须绑定到 ReactElement 中最高的 DOM 节点。
- `provided.droppableProps`: Object，包含需要应用于 Droppable 元素的属性,包含一个数据属性，可以用它来控制一些不可见的 CSS

  ```js
  console.log(provided.droppableProps);

  /* 
    {
      data-rbd-droppable-context-id: "1",
      data-rbd-droppable-id: "droppable"
    }
  */
  ```

- `provided.placeholder`: 占位符
- `snapshot`： 当前拖动状态，可以用来在被拖动时改变 Droppable 的外观。
  ```js
  console.log(snapshot);
  /*
    {
      draggingFromThisWith: null,
      draggingOverWith: null,
      isDraggingOver: false, 拖拽状态
      isUsingPlaceholder: false
    }
  */
  ```

### [Draggalbe](https://github.com/atlassian/react-beautiful-dnd/blob/master/docs/api/draggable.md)

```jsx
import { Draggable } from 'react-beautiful-dnd';

<Draggable draggableId="draggable-1" index={0}>
  {(provided, snapshot) => (
    <div
      ref={provided.innerRef}
      {...provided.draggableProps}
      {...provided.dragHandleProps}
    >
      <h4>My draggable</h4>
    </div>
  )}
</Draggable>;
```

**可被拖拽的元素，**`<Draggable />` 必须始终包含在 `<Droppable />` 中，可以在 `<Droppable />` 内重新排序 `<Draggable />` 或移动到另一个 `<Droppable />`。

参数介绍：

- `draggableId`：`string`，必需，作为唯一标识符。
- `index`：`number`，匹配 `Droppable` 中 `Draggable` 的顺序。是列表中 `Draggable` 的唯一索引，索引数组必须是连续的，比如 `[0,1,2]`。
- `isDragDisabled`： 默认 `false`，一个可选标志，用于控制是否允许 `Draggable` 拖动。
- 其他同 `<Droppable />`。

## 开始项目

效果预览：
![03.gif](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5f62352ea0f2420ebac6a8126f8a7ea9~tplv-k3u1fbpfcp-watermark.image)

### 安装

```
# yarn
yarn add react-beautiful-dnd

# npm
npm install react-beautiful-dnd --save
```

使用 `react-beautiful-dnd`：

```js
import { DragDropContext } from 'react-beautiful-dnd';
```
