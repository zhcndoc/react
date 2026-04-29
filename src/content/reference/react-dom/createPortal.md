---
title: createPortal
---

<Intro>

`createPortal` 让你可以将一些子元素渲染到 DOM 的不同部分。


```js
<div>
  <SomeComponent />
  {createPortal(children, domNode, key?)}
</div>
```

</Intro>

<InlineToc />

---

## 参考 {/*reference*/}

### `createPortal(children, domNode, key?)` {/*createportal*/}

要创建一个 portal，请调用 `createPortal`，传入一些 JSX，以及它应该渲染到的 DOM 节点：

```js
import { createPortal } from 'react-dom';

// ...

<div>
  <p>这个子元素被放置在父级 div 中。</p>
  {createPortal(
    <p>这个子元素被放置在 document body 中。</p>,
    document.body
  )}
</div>
```

[查看更多示例。](#usage)

portal 只会改变 DOM 节点的物理位置。除此之外，你渲染到 portal 中的 JSX 的行为，和作为渲染它的 React 组件的子节点并无不同。例如，子元素可以访问由父级树提供的上下文，并且事件会按照 React 树从子节点冒泡到父节点。

#### 参数 {/*parameters*/}

* `children`：任何可以由 React 渲染的内容，例如一段 JSX（比如 `<div />` 或 `<SomeComponent />`）、一个 [Fragment](/reference/react/Fragment)（`<>...</>`）、字符串或数字，或者它们组成的数组。

* `domNode`：某个 DOM 节点，例如 `document.getElementById()` 返回的节点。该节点必须已经存在。在更新期间传入不同的 DOM 节点会导致 portal 内容被重新创建。

* **可选** `key`：用于作为 portal 的 [key](/learn/rendering-lists#keeping-list-items-in-order-with-key) 的唯一字符串或数字。

#### 返回值 {/*returns*/}

`createPortal` 返回一个 React 节点，它可以包含在 JSX 中，或从 React 组件中返回。如果 React 在渲染输出中遇到它，它会将提供的 `children` 放入提供的 `domNode` 内。

#### 注意事项 {/*caveats*/}

* 来自 portal 的事件会按照 React 树而不是 DOM 树传播。例如，如果你在 portal 内点击，并且 portal 被 `<div onClick>` 包裹，那么该 `onClick` 处理函数会被触发。如果这会导致问题，可以选择阻止 portal 内部事件继续传播，或者将 portal 本身在 React 树中向上移动。

---

## 用法 {/*usage*/}

### 渲染到 DOM 的不同部分 {/*rendering-to-a-different-part-of-the-dom*/}

*Portal* 允许你的组件将其中一些子元素渲染到 DOM 中的不同位置。这让组件的一部分可以从它可能处于的任何容器中“逃逸”出来。例如，组件可以显示一个弹窗对话框或一个工具提示，使其显示在页面其余内容的上方和外部。

要创建一个 portal，请渲染 `createPortal` 的结果，并传入 <CodeStep step={1}>一些 JSX</CodeStep> 以及 <CodeStep step={2}>它应该放置到的 DOM 节点</CodeStep>：

```js [[1, 8, "<p>This child is placed in the document body.</p>"], [2, 9, "document.body"]]
import { createPortal } from 'react-dom';

function MyComponent() {
  return (
    <div style={{ border: '2px solid black' }}>
      <p>这个子元素被放置在父级 div 中。</p>
      {createPortal(
        <p>这个子元素被放置在 document body 中。</p>,
        document.body
      )}
    </div>
  );
}
```

React 会将 <CodeStep step={1}>你传入的 JSX</CodeStep> 对应的 DOM 节点放入 <CodeStep step={2}>你提供的 DOM 节点</CodeStep> 内。

如果没有 portal，第二个 `<p>` 会被放置在父级 `<div>` 中，但 portal 将它“传送”到了 [`document.body`:](https://developer.mozilla.org/en-US/docs/Web/API/Document/body)

<Sandpack>

```js
import { createPortal } from 'react-dom';

export default function MyComponent() {
  return (
    <div style={{ border: '2px solid black' }}>
      <p>这个子元素被放置在父级 div 中。</p>
      {createPortal(
        <p>这个子元素被放置在 document body 中。</p>,
        document.body
      )}
    </div>
  );
}
```

</Sandpack>

注意第二个段落在视觉上是如何显示在带边框的父级 `<div>` 外部的。如果你使用开发者工具检查 DOM 结构，你会看到第二个 `<p>` 被直接放置到了 `<body>` 中：

```html {4-6,9}
<body>
  <div id="root">
    ...
      <div style="border: 2px solid black">
        <p>这个子元素被放置在父级 div 中。</p>
      </div>
    ...
  </div>
  <p>这个子元素被放置在 document body 中。</p>
</body>
```

portal 只会改变 DOM 节点的物理位置。除此之外，你渲染到 portal 中的 JSX 的行为，和作为渲染它的 React 组件的子节点并无不同。例如，子元素可以访问由父级树提供的上下文，并且事件仍会按照 React 树从子节点冒泡到父节点。

---

### 使用 portal 渲染模态对话框 {/*rendering-a-modal-dialog-with-a-portal*/}

你可以使用 portal 来创建一个悬浮在页面其余内容之上的模态对话框，即使调用该对话框的组件位于带有 `overflow: hidden` 或其他会干扰对话框样式的容器中。

在这个例子中，两个容器都有会破坏模态对话框的样式，但渲染到 portal 中的那个不会受影响，因为在 DOM 中，模态并不包含在父级 JSX 元素内。

<Sandpack>

```js src/App.js active
import NoPortalExample from './NoPortalExample';
import PortalExample from './PortalExample';

export default function App() {
  return (
    <>
      <div className="clipping-container">
        <NoPortalExample  />
      </div>
      <div className="clipping-container">
        <PortalExample />
      </div>
    </>
  );
}
```

```js src/NoPortalExample.js
import { useState } from 'react';
import ModalContent from './ModalContent.js';

export default function NoPortalExample() {
  const [showModal, setShowModal] = useState(false);
  return (
    <>
      <button onClick={() => setShowModal(true)}>
        不使用 portal 显示模态框
      </button>
      {showModal && (
        <ModalContent onClose={() => setShowModal(false)} />
      )}
    </>
  );
}
```

```js src/PortalExample.js active
import { useState } from 'react';
import { createPortal } from 'react-dom';
import ModalContent from './ModalContent.js';

export default function PortalExample() {
  const [showModal, setShowModal] = useState(false);
  return (
    <>
      <button onClick={() => setShowModal(true)}>
        使用 portal 显示模态框
      </button>
      {showModal && createPortal(
        <ModalContent onClose={() => setShowModal(false)} />,
        document.body
      )}
    </>
  );
}
```

```js src/ModalContent.js
export default function ModalContent({ onClose }) {
  return (
    <div className="modal">
      <div>我是一个模态对话框</div>
      <button onClick={onClose}>关闭</button>
    </div>
  );
}
```


```css src/styles.css
.clipping-container {
  position: relative;
  border: 1px solid #aaa;
  margin-bottom: 12px;
  padding: 12px;
  width: 250px;
  height: 80px;
  overflow: hidden;
}

.modal {
  display: flex;
  justify-content: space-evenly;
  align-items: center;
  box-shadow: rgba(100, 100, 111, 0.3) 0px 7px 29px 0px;
  background-color: white;
  border: 2px solid rgb(240, 240, 240);
  border-radius: 12px;
  position:  absolute;
  width: 250px;
  top: 70px;
  left: calc(50% - 125px);
  bottom: 70px;
}
```

</Sandpack>

<Pitfall>

在使用 portal 时，确保你的应用具有可访问性非常重要。例如，你可能需要管理键盘焦点，以便用户能够自然地将焦点移入和移出 portal。

创建模态框时，请遵循 [WAI-ARIA Modal Authoring Practices](https://www.w3.org/WAI/ARIA/apg/patterns/dialog-modal)。如果你使用社区提供的包，请确保它具备可访问性并遵循这些指南。

</Pitfall>

---

### 将 React 组件渲染到非 React 的服务器标记中 {/*rendering-react-components-into-non-react-server-markup*/}

如果你的 React 根节点只是一个静态页面或服务器渲染页面的一部分，而该页面并不是用 React 构建的，那么 portal 也会很有用。例如，如果你的页面是用 Rails 之类的服务器框架构建的，你可以在侧边栏等静态区域中创建交互区域。与拥有[多个独立的 React 根节点](/reference/react-dom/client/createRoot#rendering-a-page-partially-built-with-react)相比，portal 让你可以将应用视为一个共享状态的单一 React 树，即使它的各个部分会渲染到 DOM 的不同位置。

<Sandpack>

```html public/index.html
<!DOCTYPE html>
<html>
  <head><title>我的应用</title></head>
  <body>
    <h1>欢迎来到我的混合应用</h1>
    <div class="parent">
      <div class="sidebar">
        这是服务器端非 React 标记
        <div id="sidebar-content"></div>
      </div>
      <div id="root"></div>
    </div>
  </body>
</html>
```

```js src/index.js
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import App from './App.js';
import './styles.css';

const root = createRoot(document.getElementById('root'));
root.render(
  <StrictMode>
    <App />
  </StrictMode>
);
```

```js src/App.js active
import { createPortal } from 'react-dom';

const sidebarContentEl = document.getElementById('sidebar-content');

export default function App() {
  return (
    <>
      <MainContent />
      {createPortal(
        <SidebarContent />,
        sidebarContentEl
      )}
    </>
  );
}

function MainContent() {
  return <p>这一部分由 React 渲染</p>;
}

function SidebarContent() {
  return <p>这一部分也由 React 渲染！</p>;
}
```

```css
.parent {
  display: flex;
  flex-direction: row;
}

#root {
  margin-top: 12px;
}

.sidebar {
  padding:  12px;
  background-color: #eee;
  width: 200px;
  height: 200px;
  margin-right: 12px;
}

#sidebar-content {
  margin-top: 18px;
  display: block;
  background-color: white;
}

p {
  margin: 0;
}
```

</Sandpack>

---

### 将 React 组件渲染到非 React 的 DOM 节点中 {/*rendering-react-components-into-non-react-dom-nodes*/}

你也可以使用 portal 来管理由 React 之外的系统管理的 DOM 节点中的内容。例如，假设你正在与一个非 React 的地图组件集成，并且想要在弹出窗口中渲染 React 内容。为此，声明一个 `popupContainer` 状态变量，用来保存你将要渲染到其中的 DOM 节点：

```js
const [popupContainer, setPopupContainer] = useState(null);
```

当你创建第三方组件时，把该组件返回的 DOM 节点保存起来，这样你就可以向其中渲染内容：

```js {5-6}
useEffect(() => {
  if (mapRef.current === null) {
    const map = createMapWidget(containerRef.current);
    mapRef.current = map;
    const popupDiv = addPopupToMapWidget(map);
    setPopupContainer(popupDiv);
  }
}, []);
```

这让你可以在 `popupContainer` 可用后，使用 `createPortal` 将 React 内容渲染到其中：

```js {3-6}
return (
  <div style={{ width: 250, height: 250 }} ref={containerRef}>
    {popupContainer !== null && createPortal(
      <p>Hello from React!</p>,
      popupContainer
    )}
  </div>
);
```

下面是一个你可以实际尝试的完整示例：

<Sandpack>

```json package.json hidden
{
  "dependencies": {
    "leaflet": "1.9.1",
    "react": "latest",
    "react-dom": "latest",
    "react-scripts": "latest",
    "remarkable": "2.0.1"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject"
  }
}
```

```js src/App.js
import { useRef, useEffect, useState } from 'react';
import { createPortal } from 'react-dom';
import { createMapWidget, addPopupToMapWidget } from './map-widget.js';

export default function Map() {
  const containerRef = useRef(null);
  const mapRef = useRef(null);
  const [popupContainer, setPopupContainer] = useState(null);

  useEffect(() => {
    if (mapRef.current === null) {
      const map = createMapWidget(containerRef.current);
      mapRef.current = map;
      const popupDiv = addPopupToMapWidget(map);
      setPopupContainer(popupDiv);
    }
  }, []);

  return (
    <div style={{ width: 250, height: 250 }} ref={containerRef}>
      {popupContainer !== null && createPortal(
        <p>Hello from React!</p>,
        popupContainer
      )}
    </div>
  );
}
```

```js src/map-widget.js
import 'leaflet/dist/leaflet.css';
import * as L from 'leaflet';

export function createMapWidget(containerDomNode) {
  const map = L.map(containerDomNode);
  map.setView([0, 0], 0);
  L.tileLayer('https://tile.openstreetmap.org/{z}/{x}/{y}.png', {
    maxZoom: 19,
    attribution: '© OpenStreetMap'
  }).addTo(map);
  return map;
}

export function addPopupToMapWidget(map) {
  const popupDiv = document.createElement('div');
  L.popup()
    .setLatLng([0, 0])
    .setContent(popupDiv)
    .openOn(map);
  return popupDiv;
}
```

```css
button { margin: 5px; }
```

</Sandpack>
