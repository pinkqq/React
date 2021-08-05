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
- `direction`：`vertical`（垂直拖拽，默认）/ `horizontal`（水平拖拽）
- `type`：指定可以被拖动的元素 class

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
![dnd.gif](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/97c75b60f7dd4112a8ba740842c13e84~tplv-k3u1fbpfcp-watermark.image)

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

### 目录

```
└── dndPro
    ├── component
        ├── Column.js
        ├── Item.js
        ├── ItemList.js
        └── Row.js
    ├── get-initial-data.js // 初始数据
    ├── style.css // 样式文件
    └── index.js
```

### Dom 结构

我们要实现三种拖拽需求：**容器可拖拽**、**元素可穿梭容器拖拽** 和 **容器内部元素的垂直拖拽**。我们将通过两层不通拖拽方向的 `<Droppable direction="?" />` 和虚拟列表模式实现。

#### 第一层：容器可拖拽

`<DragDropContext />` 包裹在最外层，构建一个可以拖拽的范围；添加第一个 `<Droppable />`，一个可以被拖拽放入的区域块，并指定拖拽方向为水平（`horizontal`）实现容器见的拖拽，指定拖拽类型为 `column` （只有 `className='column'` 元素可拖拽）。根据 `columnOrder: ['column-0', 'column-1']` 渲染两个 `Column` 组件。

```jsx
// index.js

import { DragDropContext, Droppable } from 'react-beautiful-dnd';

<DragDropContext onDragEnd={onDragEnd}>
  <div className="dnd-pro">
    <Droppable
      droppableId="all-droppables"
      direction="horizontal"
      type="column"
    >
      {(provided) => (
        <div
          className="columns"
          {...provided.droppableProps}
          ref={provided.innerRef}
        >
          {state.columnOrder.map((columnId, index) => (
            <Column
              key={columnId}
              column={state.columns[columnId]}
              index={index}
            />
          ))}
          {provided.placeholder}
        </div>
      )}
    </Droppable>
  </div>
</DragDropContext>;
```

`Column` 组件就是一个 `<Draggable />` 元素，到这里就实现了**容器可拖拽**，是不是很 easy ～

```jsx
// Column.js

import { Draggable } from 'react-beautiful-dnd';

<Draggable draggableId={column.id} index={index}>
  {(provided) => (
    <div
      className="column"
      {...provided.draggableProps}
      ref={provided.innerRef}
    >
      <h3 className="column-title" {...provided.dragHandleProps}>
        {column.title}
      </h3>
      <ItemList column={column} index={index} />
    </div>
  )}
</Draggable>;
```

#### 第二层：虚拟列表

`react-beautiful-dnd` 支持在虚拟列表内和虚拟列表之间拖放。一般情况，建议列表的规格不超过 500 个元素。虚拟列表，是对窗口性能的一种优化，详细介绍可以[看这边](https://github.com/atlassian/react-beautiful-dnd/blob/master/docs/patterns/virtual-lists.md)，我们直接来看看用法吧：

- 虚拟列表的行为与常规列表不同。我们需要告诉 `rbd` 我们的列表是虚拟列表。

  ```jsx
  <Droppable droppableId="droppable" mode="virtual">
    {/*...*/}
  </Droppable>
  ```

- 拖动项目的副本

  ```jsx
  <Droppable
    droppableId="droppable"
    mode="virtual"
    renderClone={(provided, snapshot, rubric) => (
      <div
        {...provided.draggableProps}
        {...provided.dragHandleProps}
        ref={provided.innerRef}
      >
        Item id: {items[rubric.source.index].id}
      </div>
    )}
  >
    {/*...*/}
  </Droppable>
  ```

需要注意的是，`placeholder` 在虚拟列表中会出现问题。因为虚拟列表的长度不再取决于元素的集体长度，而是计算视觉长度，所以我们需要做一些处理：

```jsx
const itemCount = snapshot.isUsingPlaceholder
  ? column.items.length + 1
  : column.items.length;
```

对虚拟列表有初步了解后，我们再回到 `<ItemList />` 来。

新开辟一个可以被拖拽放入的区域块 `<Droppable />`，拖拽方向取垂直（默认），指定虚拟列表模式（`mode="virtual"`），发生拖拽时，使用元素副本（`renderClone`）。

```jsx
// ItemList.js

import { FixedSizeList } from 'react-window';
import { Droppable } from 'react-beautiful-dnd';

<Droppable
  droppableId={column.id}
  mode="virtual"
  renderClone={(provided, snapshot, rubric) => (
    <Item
      provided={provided}
      isDragging={snapshot.isDragging}
      item={column.items[rubric.source.index]}
    />
  )}
>
  {(provided, snapshot) => {
    const itemCount = snapshot.isUsingPlaceholder
      ? column.items.length + 1
      : column.items.length;

    return (
      <FixedSizeList
        height={500}
        itemCount={itemCount}
        itemSize={80}
        width={300}
        outerRef={provided.innerRef}
        itemData={column.items}
        className="task-list"
        ref={listRef}
      >
        {Row}
      </FixedSizeList>
    );
  }}
</Droppable>;
```

`<Item />` 组件和 `<Row />` 组件都是一个 Draggable 元素。

### onDragEnd

容器级别的拖拽和同容器内的元素拖拽，简单的交换元素即可：

```js
// reordering list
if (result.type === 'column') {
  const columnOrder = reorderList(
    state.columnOrder,
    result.source.index,
    result.destination.index,
  );

  setState({
    ...state,
    columnOrder,
  });
  return;
}

// reordering in same list
if (result.source.droppableId === result.destination.droppableId) {
  const column = state.columns[result.source.droppableId];
  const items = reorderList(
    column.items,
    result.source.index,
    result.destination.index,
  );

  const newState = {
    ...state,
    columns: {
      ...state.columns,
      [column.id]: {
        ...column,
        items,
      },
    },
  };
  setState(newState);
  return;
}
```

元素跨容器拖拽，分为两步：

- 从源列表（source）中删除元素
- 将元素添加到目标列表（destination）

```jsx
// moving between lists
const sourceColumn = state.columns[result.source.droppableId];
const destinationColumn = state.columns[result.destination.droppableId];
const item = sourceColumn.items[result.source.index];

// 1. remove item from source column
const newSourceColumn = {
  ...sourceColumn,
  items: [...sourceColumn.items],
};
newSourceColumn.items.splice(result.source.index, 1);

// 2. insert into destination column
const newDestinationColumn = {
  ...destinationColumn,
  items: [...destinationColumn.items],
};
newDestinationColumn.items.splice(result.destination.index, 0, item);

const newState = {
  ...state,
  columns: {
    ...state.columns,
    [newSourceColumn.id]: newSourceColumn,
    [newDestinationColumn.id]: newDestinationColumn,
  },
};

setState(newState);
```

### 完整代码

主要逻辑和核心代码以上，完整代码[戳这里](https://github.com/pinkqq/react-antd/tree/main/src/pages/dndPro)。
