**React 实现对话框有两种方案**：

- 使用 UI 组件库：比如 Antd 的组件 -- [Modal](https://ant.design/components/modal-cn/)
- 原生 React Portals

![dialog-1.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a78794e45aa04d75a97ab99ed7a0eeca~tplv-k3u1fbpfcp-watermark.image)
在实际开发中，我们大多会选择直接使用 Antd 的对话框组件，其实 Antd 对话框的实现也是基于 React Portals 这个特性，所以了解它可以让我们的漂浮层之路走的更宽哦：

![dialog-2.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a7928b0cf74c443280a4cf5c49a1a6f6~tplv-k3u1fbpfcp-watermark.image)

![dialog-3.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f172246f8abb402181981fd96f01e0ae~tplv-k3u1fbpfcp-watermark.image)

## [React Portals](https://zh-hans.reactjs.org/docs/portals.html#gatsby-focus-wrapper) 是什么？

`Portal` 是 React 16.3 新引入的 API，React 是在内存中维护了一棵 Virtual Dom Tree ，这棵树会映射到真实的 Dom 节点，而 `Portal` 可以打断这种映射关系，它提供了一种将子节点渲染到存在于父组件以外的 `DOM` 节点的优秀的方案，一举解决了漂浮层的问题，如：Dialog、Tooltip 等。

### 用法

```js
ReactDOM.createPortal(child, container);
```

第一个参数（`child`）是任何可渲染的 React 子元素，例如一个元素，字符串或 fragment。第二个参数（`container`）是一个 DOM 元素。

需要注意，虽然 child 看似显示在另一颗 Dom 树上，但在 Virtual Dom 中，其实是同一棵，在下面的例子中也会看到。

## Antd 的 Dialog

按照[官网文档](https://ant.design/components/modal-cn/)，上手非常简单

```jsx
import React, { useState } from 'react';
import { Modal, Button } from 'antd';

const DialogPage = () => {
  const [isModalVisible, setIsModalVisible] = useState(false);

  const showModal = () => {
    setIsModalVisible(true);
  };

  const handleOk = () => {
    setIsModalVisible(false);
  };

  const handleCancel = () => {
    setIsModalVisible(false);
  };

  return (
    <>
      <Button type="primary" onClick={showModal}>
        Open Antd Modal
      </Button>
      <Modal
        title="Basic Modal"
        visible={isModalVisible}
        onOk={handleOk}
        onCancel={handleCancel}
      >
        <p>Some contents...</p>
        <p>Some contents...</p>
        <p>Some contents...</p>
      </Modal>
    </>
  );
};

export default DialogPage;
```

## Portals 的 Dialog

在 index.html 中，创建一个 dialog 的容器元素：

```html
<div id="root"></div>
<div id="dialog-root"></div>
```

通过组件内部状态（`visible`） 控制 Dialog 的渲染：

```jsx
{
  visible
    ? createPortal(
        <div className="portal-sample">
          {children}
          <Button onClick={onHide}>close</Button>
        </div>,
        document.getElementById('dialog-root'),
      )
    : null;
}
```

从 React Components 可以看出来，PortalDialog 和 DialogPage 的父子关系依然存在：

![dialog-5.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d71da27edbc048128fe18839b1659286~tplv-k3u1fbpfcp-watermark.image)

### 完整代码

#### pages/dialog.js

```jsx
import React, { useState } from 'react';
import { Button } from 'antd';
import PortalDialog from '@/components/PortalDialog';

const DialogPage = () => {
  const [isPortalVisible, setIsPortalVisible] = useState(false);

  const showPortal = () => {
    setIsPortalVisible(true);
  };

  const hidePortal = () => {
    setIsPortalVisible(false);
  };

  return (
    <>
      <Button style={{ marginLeft: '20px' }} onClick={showPortal}>
        Open Dialog（React Portals）
      </Button>
      <PortalDialog visible={isPortalVisible} onHide={hidePortal}>
        <div>dialog-children</div>
      </PortalDialog>
    </>
  );
};

export default DialogPage;
```

#### components/PortalDialog/index.js

```jsx
import { createPortal } from 'react-dom';
import { Button } from 'antd';
import './style.css';

const PortalDialog = (props) => {
  const { visible, children, onHide } = props;
  return visible
    ? createPortal(
        <div className="portal-sample">
          {children}
          <Button onClick={onHide}>close</Button>
        </div>,
        document.getElementById('dialog-root'),
      )
    : null;
};

export default PortalDialog;
```

#### components/PortalDialog/style.css

```css
.portal-sample {
  position: absolute;
  padding: 20px;
  width: 500px;
  height: 300px;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);
  background-color: #fff;
  border-radius: 10px;
  border: 1px solid #ddd;
  box-shadow: 0px 0px 20px 2px #ddd;
}
```
