---
title: <StrictMode>
---


<Intro>

`<StrictMode>` 让你在开发过程中尽早发现组件中的常见 bug。


```js
<StrictMode>
  <App />
</StrictMode>
```

</Intro>

<InlineToc />

---

## 参考 {/*reference*/}

### `<StrictMode>` {/*strictmode*/}

使用 `StrictMode` 为内部的组件树启用额外的开发行为和警告：

```js
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';

const root = createRoot(document.getElementById('root'));
root.render(
  <StrictMode>
    <App />
  </StrictMode>
);
```

[查看更多示例。](#usage)

严格模式会启用以下仅限开发环境的行为：

- 你的组件会 [额外重新渲染一次](#fixing-bugs-found-by-double-rendering-in-development)，以发现由不纯渲染引起的 bug。
- 你的组件会 [额外重新运行 Effects 一次](#fixing-bugs-found-by-re-running-effects-in-development)，以发现由缺少 Effect 清理引起的 bug。
- 你的组件会 [额外重新运行 ref 回调一次](#fixing-bugs-found-by-re-running-ref-callbacks-in-development)，以发现由缺少 ref 清理引起的 bug。
- 你的组件会 [检查是否使用了已弃用的 API。](#fixing-deprecation-warnings-enabled-by-strict-mode)

#### Props {/*props*/}

`StrictMode` 不接受任何 props。

#### 注意事项 {/*caveats*/}

* 在被 `<StrictMode>` 包裹的树中，没有办法选择退出严格模式。这能让你确信 `<StrictMode>` 内的所有组件都会被检查。如果两个正在开发同一产品的团队对于这些检查是否有价值存在分歧，他们需要达成共识，或者将 `<StrictMode>` 下移到树的更深处。

---

## 用法 {/*usage*/}

### 为整个应用启用严格模式 {/*enabling-strict-mode-for-entire-app*/}

严格模式会为 `<StrictMode>` 组件内部的整个组件树启用额外的仅限开发环境的检查。这些检查有助于你在开发过程的早期发现组件中的常见 bug。


要为整个应用启用严格模式，在渲染根组件时用 `<StrictMode>` 包裹它：

```js {6,8}
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';

const root = createRoot(document.getElementById('root'));
root.render(
  <StrictMode>
    <App />
  </StrictMode>
);
```

我们建议将整个应用都包裹在严格模式中，尤其是新创建的应用。如果你使用的框架已经替你调用了 [`createRoot`](/reference/react-dom/client/createRoot)，请查看其文档，了解如何启用严格模式。

虽然严格模式检查 **只在开发环境运行，** 但它们能帮助你发现代码中已经存在、却很难在生产环境中稳定复现的 bug。严格模式让你能够在用户报告之前修复这些 bug。

<Note>

严格模式在开发环境中会启用以下检查：

- 你的组件会 [额外重新渲染一次](#fixing-bugs-found-by-double-rendering-in-development)，以发现由不纯渲染引起的 bug。
- 你的组件会 [额外重新运行 Effects 一次](#fixing-bugs-found-by-re-running-effects-in-development)，以发现由缺少 Effect 清理引起的 bug。
- 你的组件会 [额外重新运行 ref 回调一次](#fixing-bugs-found-by-re-running-ref-callbacks-in-development)，以发现由缺少 ref 清理引起的 bug。
- 你的组件会 [检查是否使用了已弃用的 API。](#fixing-deprecation-warnings-enabled-by-strict-mode)

**所有这些检查都只在开发环境中运行，不会影响生产构建。**

</Note>

---

### 为应用的一部分启用严格模式 {/*enabling-strict-mode-for-a-part-of-the-app*/}

你也可以为应用中的任意一部分启用严格模式：

```js {7,12}
import { StrictMode } from 'react';

function App() {
  return (
    <>
      <Header />
      <StrictMode>
        <main>
          <Sidebar />
          <Content />
        </main>
      </StrictMode>
      <Footer />
    </>
  );
}
```

在这个示例中，严格模式检查不会对 `Header` 和 `Footer` 组件运行。不过，它们会对 `Sidebar` 和 `Content` 运行，以及它们内部的所有组件运行，不论层级有多深。

<Note>

当为应用的一部分启用 `StrictMode` 时，React 只会启用那些在生产环境中也可能发生的行为。例如，如果 `<StrictMode>` 没有在应用根部启用，那么它不会在初次挂载时 [额外重新运行 Effects 一次](#fixing-bugs-found-by-re-running-effects-in-development)，因为这会导致子组件的 Effect 被执行两次，而父组件的 Effect 没有相应执行一次，这种情况在生产环境中不会发生。

</Note>

---

### 修复通过开发环境中双重渲染发现的 bug {/*fixing-bugs-found-by-double-rendering-in-development*/}

[React 假定你编写的每个组件都是纯函数。](/learn/keeping-components-pure) 这意味着你编写的 React 组件在相同输入（props、state 和 context）下，必须始终返回相同的 JSX。

违反这一规则的组件行为不可预测，并会导致 bug。为了帮助你发现意外的不纯代码，严格模式会在开发环境中 **将你的一些函数调用两次**（仅限应该是纯函数的那些）。这包括：

- 你的组件函数体（仅顶层逻辑，因此不包括事件处理器中的代码）
- 你传递给 [`useState`](/reference/react/useState)、[`set` 函数](/reference/react/useState#setstate)、[`useMemo`](/reference/react/useMemo) 或 [`useReducer`](/reference/react/useReducer) 的函数
- 某些类组件方法，例如 [`constructor`](/reference/react/Component#constructor)、[`render`](/reference/react/Component#render)、[`shouldComponentUpdate`](/reference/react/Component#shouldcomponentupdate)（[查看完整列表](https://reactjs.org/docs/strict-mode.html#detecting-unexpected-side-effects)）

如果一个函数是纯函数，运行两次不会改变其行为，因为纯函数每次都会产生相同的结果。但是，如果一个函数是不纯的（例如，它会修改自己收到的数据），运行两次通常就会很明显（这正是不纯的表现！）。这能帮助你尽早发现并修复 bug。

**下面是一个示例，用来说明严格模式中的双重渲染如何帮助你尽早发现 bug。**

这个 `StoryTray` 组件接收一个 `stories` 数组，并在末尾添加一个最后的 “Create Story” 项：

<Sandpack>

```js src/index.js
import { createRoot } from 'react-dom/client';
import './styles.css';

import App from './App';

const root = createRoot(document.getElementById("root"));
root.render(<App />);
```

```js src/App.js
import { useState } from 'react';
import StoryTray from './StoryTray.js';

let initialStories = [
  {id: 0, label: "Ankit's Story" },
  {id: 1, label: "Taylor's Story" },
];

export default function App() {
  let [stories, setStories] = useState(initialStories)
  return (
    <div
      style={{
        width: '100%',
        height: '100%',
        textAlign: 'center',
      }}
    >
      <StoryTray stories={stories} />
    </div>
  );
}
```

```js src/StoryTray.js active
export default function StoryTray({ stories }) {
  const items = stories;
  items.push({ id: 'create', label: 'Create Story' });
  return (
    <ul>
      {items.map(story => (
        <li key={story.id}>
          {story.label}
        </li>
      ))}
    </ul>
  );
}
```

```css
ul {
  margin: 0;
  list-style-type: none;
  height: 100%;
  display: flex;
  flex-wrap: wrap;
  padding: 10px;
}

li {
  border: 1px solid #aaa;
  border-radius: 6px;
  float: left;
  margin: 5px;
  padding: 5px;
  width: 70px;
  height: 100px;
}
```

</Sandpack>

上面的代码有一个错误。不过，这个错误很容易被忽略，因为初始输出看起来是正确的。

如果 `StoryTray` 组件多次重新渲染，这个错误会变得更明显。例如，我们来让 `StoryTray` 在你悬停其上时使用不同的背景色重新渲染：

<Sandpack>

```js src/index.js
import { createRoot } from 'react-dom/client';
import './styles.css';

import App from './App';

const root = createRoot(document.getElementById("root"));
root.render(<App />);
```

```js src/App.js
import { useState } from 'react';
import StoryTray from './StoryTray.js';

let initialStories = [
  {id: 0, label: "Ankit's Story" },
  {id: 1, label: "Taylor's Story" },
];

export default function App() {
  let [stories, setStories] = useState(initialStories)
  return (
    <div
      style={{
        width: '100%',
        height: '100%',
        textAlign: 'center',
      }}
    >
      <StoryTray stories={stories} />
    </div>
  );
}
```

```js src/StoryTray.js active
import { useState } from 'react';

export default function StoryTray({ stories }) {
  const [isHover, setIsHover] = useState(false);
  const items = stories;
  items.push({ id: 'create', label: 'Create Story' });
  return (
    <ul
      onPointerEnter={() => setIsHover(true)}
      onPointerLeave={() => setIsHover(false)}
      style={{
        backgroundColor: isHover ? '#ddd' : '#fff'
      }}
    >
      {items.map(story => (
        <li key={story.id}>
          {story.label}
        </li>
      ))}
    </ul>
  );
}
```

```css
ul {
  margin: 0;
  list-style-type: none;
  height: 100%;
  display: flex;
  flex-wrap: wrap;
  padding: 10px;
}

li {
  border: 1px solid #aaa;
  border-radius: 6px;
  float: left;
  margin: 5px;
  padding: 5px;
  width: 70px;
  height: 100px;
}
```

</Sandpack>

注意，每次你悬停在 `StoryTray` 组件上时，“Create Story” 都会再次被添加到列表中。代码的本意是只在末尾添加一次。但 `StoryTray` 直接修改了来自 props 的 `stories` 数组。每次 `StoryTray` 渲染时，它都会在同一个数组末尾再次添加 “Create Story”。换句话说，`StoryTray` 不是纯函数——多次运行它会产生不同结果。

要修复这个问题，你可以先复制数组，然后修改这份副本，而不是修改原数组：

```js {2}
export default function StoryTray({ stories }) {
  const items = stories.slice(); // 克隆数组
  // ✅ 好：向一个新数组中追加
  items.push({ id: 'create', label: 'Create Story' });
```

这样就会 [让 `StoryTray` 函数成为纯函数。](/learn/keeping-components-pure) 每次调用它时，它只会修改数组的新副本，而不会影响任何外部对象或变量。这解决了 bug，但你必须让组件更频繁地重新渲染，问题才会变得明显。

**在原始示例中，这个 bug 并不明显。现在让我们把原始（有 bug 的）代码包裹在 `<StrictMode>` 中：**

<Sandpack>

```js src/index.js
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import './styles.css';

import App from './App';

const root = createRoot(document.getElementById("root"));
root.render(
  <StrictMode>
    <App />
  </StrictMode>
);
```

```js src/App.js
import { useState } from 'react';
import StoryTray from './StoryTray.js';

let initialStories = [
  {id: 0, label: "Ankit's Story" },
  {id: 1, label: "Taylor's Story" },
];

export default function App() {
  let [stories, setStories] = useState(initialStories)
  return (
    <div
      style={{
        width: '100%',
        height: '100%',
        textAlign: 'center',
      }}
    >
      <StoryTray stories={stories} />
    </div>
  );
}
```

```js src/StoryTray.js active
export default function StoryTray({ stories }) {
  const items = stories;
  items.push({ id: 'create', label: 'Create Story' });
  return (
    <ul>
      {items.map(story => (
        <li key={story.id}>
          {story.label}
        </li>
      ))}
    </ul>
  );
}
```

```css
ul {
  margin: 0;
  list-style-type: none;
  height: 100%;
  display: flex;
  flex-wrap: wrap;
  padding: 10px;
}

li {
  border: 1px solid #aaa;
  border-radius: 6px;
  float: left;
  margin: 5px;
  padding: 5px;
  width: 70px;
  height: 100px;
}
```

</Sandpack>

**严格模式 *总是* 会将你的渲染函数调用两次，所以你能立刻看出错误**（“Create Story” 出现了两次）。这让你能够在流程早期发现这类错误。当你修复组件以便在严格模式下渲染时，你也 *同时* 修复了许多未来可能出现的生产环境 bug，比如前面的悬停功能：

<Sandpack>

```js src/index.js
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import './styles.css';

import App from './App';

const root = createRoot(document.getElementById("root"));
root.render(
  <StrictMode>
    <App />
  </StrictMode>
);
```

```js src/App.js
import { useState } from 'react';
import StoryTray from './StoryTray.js';

let initialStories = [
  {id: 0, label: "Ankit's Story" },
  {id: 1, label: "Taylor's Story" },
];

export default function App() {
  let [stories, setStories] = useState(initialStories)
  return (
    <div
      style={{
        width: '100%',
        height: '100%',
        textAlign: 'center',
      }}
    >
      <StoryTray stories={stories} />
    </div>
  );
}
```

```js src/StoryTray.js active
import { useState } from 'react';

export default function StoryTray({ stories }) {
  const [isHover, setIsHover] = useState(false);
  const items = stories.slice(); // 克隆数组
  items.push({ id: 'create', label: 'Create Story' });
  return (
    <ul
      onPointerEnter={() => setIsHover(true)}
      onPointerLeave={() => setIsHover(false)}
      style={{
        backgroundColor: isHover ? '#ddd' : '#fff'
      }}
    >
      {items.map(story => (
        <li key={story.id}>
          {story.label}
        </li>
      ))}
    </ul>
  );
}
```

```css
ul {
  margin: 0;
  list-style-type: none;
  height: 100%;
  display: flex;
  flex-wrap: wrap;
  padding: 10px;
}

li {
  border: 1px solid #aaa;
  border-radius: 6px;
  float: left;
  margin: 5px;
  padding: 5px;
  width: 70px;
  height: 100px;
}
```

</Sandpack>

如果没有严格模式，你很容易在添加更多重新渲染之前错过这个 bug。严格模式让同样的 bug 立刻出现。严格模式帮助你在把 bug 推送给团队和用户之前发现它们。

[阅读更多关于保持组件纯净的内容。](/learn/keeping-components-pure)

<Note>

如果你安装了 [React DevTools](/learn/react-developer-tools)，第二次渲染调用期间的任何 `console.log` 输出都会显得稍微暗一些。React DevTools 还提供了一个设置（默认关闭），可以完全抑制这些日志。

</Note>

---

### 修复通过开发环境中重新运行 Effects 发现的 bug {/*fixing-bugs-found-by-re-running-effects-in-development*/}

严格模式也可以帮助发现 [Effects](/learn/synchronizing-with-effects) 中的 bug。

每个 Effect 都有一些设置代码，并且可能有一些清理代码。通常，React 会在组件 *挂载*（添加到屏幕上）时调用设置，并在组件 *卸载*（从屏幕上移除）时调用清理。然后，如果依赖项自上次渲染以来发生变化，React 会再次调用清理和设置。

当开启严格模式时，React 还会为 **每个 Effect 在开发环境中额外运行一次设置+清理循环。** 这可能看起来有些意外，但它有助于暴露那些很难手动捕捉的细微 bug。

**下面是一个示例，用来说明在严格模式中重新运行 Effects 如何帮助你尽早发现 bug。**

考虑这个把组件连接到聊天的示例：

<Sandpack>

```js src/index.js
import { createRoot } from 'react-dom/client';
import './styles.css';

import App from './App';

const root = createRoot(document.getElementById("root"));
root.render(<App />);
```

```js
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

const serverUrl = 'https://localhost:1234';
const roomId = 'general';

export default function ChatRoom() {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
  }, []);
  return <h1>Welcome to the {roomId} room!</h1>;
}
```

```js src/chat.js
let connections = 0;

export function createConnection(serverUrl, roomId) {
  // 真实实现会实际连接到服务器
  return {
    connect() {
      console.log('✅ Connecting to "' + roomId + '" room at ' + serverUrl + '...');
      connections++;
      console.log('Active connections: ' + connections);
    },
    disconnect() {
      console.log('❌ Disconnected from "' + roomId + '" room at ' + serverUrl);
      connections--;
      console.log('Active connections: ' + connections);
    }
  };
}
```

```css
input { display: block; margin-bottom: 20px; }
button { margin-left: 10px; }
```

</Sandpack>

这段代码有个问题，但一开始可能并不明显。

为了让问题更明显，我们来实现一个功能。在下面的示例中，`roomId` 不再是硬编码的。相反，用户可以从下拉菜单中选择他们想连接的 `roomId`。点击“Open chat”，然后依次选择不同的聊天室。请留意控制台中活动连接的数量：

<Sandpack>

```js src/index.js
import { createRoot } from 'react-dom/client';
import './styles.css';

import App from './App';

const root = createRoot(document.getElementById("root"));
root.render(<App />);
```

```js
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
  }, [roomId]);

  return <h1>Welcome to the {roomId} room!</h1>;
}

export default function App() {
  const [roomId, setRoomId] = useState('general');
  const [show, setShow] = useState(false);
  return (
    <>
      <label>
        选择聊天室：{' '}
        <select
          value={roomId}
          onChange={e => setRoomId(e.target.value)}
        >
          <option value="general">general</option>
          <option value="travel">travel</option>
          <option value="music">music</option>
        </select>
      </label>
      <button onClick={() => setShow(!show)}>
        {show ? 'Close chat' : 'Open chat'}
      </button>
      {show && <hr />}
      {show && <ChatRoom roomId={roomId} />}
    </>
  );
}
```

```js src/chat.js
let connections = 0;

export function createConnection(serverUrl, roomId) {
  // 真实实现会实际连接到服务器
  return {
    connect() {
      console.log('✅ Connecting to "' + roomId + '" room at ' + serverUrl + '...');
      connections++;
      console.log('Active connections: ' + connections);
    },
    disconnect() {
      console.log('❌ Disconnected from "' + roomId + '" room at ' + serverUrl);
      connections--;
      console.log('Active connections: ' + connections);
    }
  };
}
```

```css
input { display: block; margin-bottom: 20px; }
button { margin-left: 10px; }
```

</Sandpack>

你会注意到，打开的连接数量总是在持续增长。在真实应用中，这会导致性能和网络问题。问题在于 [你的 Effect 缺少清理函数：](/learn/synchronizing-with-effects#step-3-add-cleanup-if-needed)

```js {4}
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]);
```

现在你的 Effect 会在自己结束后“清理”，并销毁过时的连接，泄漏问题就解决了。不过，请注意，在你添加更多功能（下拉框）之前，这个问题并不明显。

**在原始示例中，这个 bug 并不明显。现在让我们把原始（有 bug 的）代码包裹在 `<StrictMode>` 中：**

<Sandpack>

```js src/index.js
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import './styles.css';

import App from './App';

const root = createRoot(document.getElementById("root"));
root.render(
  <StrictMode>
    <App />
  </StrictMode>
);
```

```js
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

const serverUrl = 'https://localhost:1234';
const roomId = 'general';

export default function ChatRoom() {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
  }, []);
  return <h1>Welcome to the {roomId} room!</h1>;
}
```

```js src/chat.js
let connections = 0;

export function createConnection(serverUrl, roomId) {
  // 真实实现会实际连接到服务器
  return {
    connect() {
      console.log('✅ Connecting to "' + roomId + '" room at ' + serverUrl + '...');
      connections++;
      console.log('Active connections: ' + connections);
    },
    disconnect() {
      console.log('❌ Disconnected from "' + roomId + '" room at ' + serverUrl);
      connections--;
      console.log('Active connections: ' + connections);
    }
  };
}
```

```css
input { display: block; margin-bottom: 20px; }
button { margin-left: 10px; }
```

</Sandpack>

**有了严格模式，你会立刻看到这里存在问题**（活动连接数跳到了 2）。严格模式会为每个 Effect 额外运行一次设置+清理循环。这个 Effect 没有清理逻辑，所以它创建了一个额外连接，却没有销毁它。这提示你缺少一个清理函数。

严格模式让你能够尽早注意到这些错误。当你在严格模式下通过添加清理函数来修复 Effect 时，你也 *同时* 修复了许多未来可能出现的生产环境 bug，比如前面的下拉选择框：

<Sandpack>

```js src/index.js
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import './styles.css';

import App from './App';

const root = createRoot(document.getElementById("root"));
root.render(
  <StrictMode>
    <App />
  </StrictMode>
);
```

```js
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]);

  return <h1>Welcome to the {roomId} room!</h1>;
}

export default function App() {
  const [roomId, setRoomId] = useState('general');
  const [show, setShow] = useState(false);
  return (
    <>
      <label>
        选择聊天室：{' '}
        <select
          value={roomId}
          onChange={e => setRoomId(e.target.value)}
        >
          <option value="general">general</option>
          <option value="travel">travel</option>
          <option value="music">music</option>
        </select>
      </label>
      <button onClick={() => setShow(!show)}>
        {show ? 'Close chat' : 'Open chat'}
      </button>
      {show && <hr />}
      {show && <ChatRoom roomId={roomId} />}
    </>
  );
}
```

```js src/chat.js
let connections = 0;

export function createConnection(serverUrl, roomId) {
  // 真实实现会实际连接到服务器
  return {
    connect() {
      console.log('✅ Connecting to "' + roomId + '" room at ' + serverUrl + '...');
      connections++;
      console.log('Active connections: ' + connections);
    },
    disconnect() {
      console.log('❌ Disconnected from "' + roomId + '" room at ' + serverUrl);
      connections--;
      console.log('Active connections: ' + connections);
    }
  };
}
```

```css
input { display: block; margin-bottom: 20px; }
button { margin-left: 10px; }
```

</Sandpack>

注意控制台中的活动连接数不再持续增长了。

如果没有严格模式，你很容易忽略你的 Effect 需要清理。通过在开发环境中对你的 Effect 执行 *设置 → 清理 → 设置*，而不是 *设置*，严格模式让缺失的清理逻辑变得更明显。

[阅读更多关于实现 Effect 清理的内容。](/learn/synchronizing-with-effects#how-to-handle-the-effect-firing-twice-in-development)

---
### 修复通过开发环境中重新运行 ref 回调发现的 bug {/*fixing-bugs-found-by-re-running-ref-callbacks-in-development*/}

严格模式也可以帮助发现 [回调 refs](/learn/manipulating-the-dom-with-refs) 中的 bug。

每个回调 `ref` 都有一些设置代码，并且可能有一些清理代码。通常，React 会在元素 *创建*（添加到 DOM 中）时调用设置，并在元素 *移除*（从 DOM 中移除）时调用清理。

当开启严格模式时，React 还会为 **每个回调 `ref` 在开发环境中额外运行一次设置+清理循环。** 这可能看起来有些意外，但它有助于暴露那些很难手动捕捉的细微 bug。

考虑这个示例：它允许你选择一只动物，然后滚动到其中一只。注意，当你从 “Cats” 切换到 “Dogs” 时，控制台日志显示列表中的动物数量不断增加，而且 “Scroll to” 按钮停止工作：

<Sandpack>

```js src/index.js
import { createRoot } from 'react-dom/client';
import './styles.css';

import App from './App';

const root = createRoot(document.getElementById("root"));
// ❌ 未使用 StrictMode。
root.render(<App />);
```

```js src/App.js active
import { useRef, useState } from "react";

export default function CatFriends() {
  const itemsRef = useRef([]);
  const [catList, setCatList] = useState(setupCatList);
  const [cat, setCat] = useState('neo');

  function scrollToCat(index) {
    const list = itemsRef.current;
    const {node} = list[index];
    node.scrollIntoView({
      behavior: "smooth",
      block: "nearest",
      inline: "center",
    });
  }

  const cats = catList.filter(c => c.type === cat)

  return (
    <>
      <nav>
        <button onClick={() => setCat('neo')}>Neo</button>
        <button onClick={() => setCat('millie')}>Millie</button>
      </nav>
      <hr />
      <nav>
        <span>Scroll to:</span>{cats.map((cat, index) => (
          <button key={cat.src} onClick={() => scrollToCat(index)}>
            {index}
          </button>
        ))}
      </nav>
      <div>
        <ul>
          {cats.map((cat) => (
            <li
              key={cat.src}
              ref={(node) => {
                const list = itemsRef.current;
                const item = {cat: cat, node};
                list.push(item);
                console.log(`✅ Adding cat to the map. Total cats: ${list.length}`);
                if (list.length > 10) {
                  console.log('❌ Too many cats in the list!');
                }
                return () => {
                  // 🚩 没有清理，这是个 bug！
                }
              }}
            >
              <img src={cat.src} />
            </li>
          ))}
        </ul>
      </div>
    </>
  );
}

function setupCatList() {
  const catList = [];
  for (let i = 0; i < 10; i++) {
    catList.push({type: 'neo', src: "https://placecats.com/neo/320/240?" + i});
  }
  for (let i = 0; i < 10; i++) {
    catList.push({type: 'millie', src: "https://placecats.com/millie/320/240?" + i});
  }

  return catList;
}

```

```css
div {
  width: 100%;
  overflow: hidden;
}

nav {
  text-align: center;
}

button {
  margin: .25rem;
}

ul,
li {
  list-style: none;
  white-space: nowrap;
}

li {
  display: inline;
  padding: 0.5rem;
}
```

</Sandpack>


**这是一个生产环境 bug！** 由于 ref 回调没有在清理阶段将动物从列表中移除，所以动物列表会持续增长。这是一个内存泄漏，会在真实应用中造成性能问题，并破坏应用行为。

问题在于 ref 回调没有在完成后进行清理：

```js {6-8}
<li
  ref={node => {
    const list = itemsRef.current;
    const item = {animal, node};
    list.push(item);
    return () => {
      // 🚩 没有清理，这是个 bug！
    }
  }}
</li>
```

现在让我们把原始（有 bug 的）代码包裹在 `<StrictMode>` 中：

<Sandpack>

```js src/index.js
import { createRoot } from 'react-dom/client';
import {StrictMode} from 'react';
import './styles.css';

import App from './App';

const root = createRoot(document.getElementById("root"));
// ✅ 使用 StrictMode。
root.render(
  <StrictMode>
    <App />
  </StrictMode>
);
```

```js src/App.js active
import { useRef, useState } from "react";

export default function CatFriends() {
  const itemsRef = useRef([]);
  const [catList, setCatList] = useState(setupCatList);
  const [cat, setCat] = useState('neo');

  function scrollToCat(index) {
    const list = itemsRef.current;
    const {node} = list[index];
    node.scrollIntoView({
      behavior: "smooth",
      block: "nearest",
      inline: "center",
    });
  }

  const cats = catList.filter(c => c.type === cat)

  return (
    <>
      <nav>
        <button onClick={() => setCat('neo')}>Neo</button>
        <button onClick={() => setCat('millie')}>Millie</button>
      </nav>
      <hr />
      <nav>
        <span>Scroll to:</span>{cats.map((cat, index) => (
          <button key={cat.src} onClick={() => scrollToCat(index)}>
            {index}
          </button>
        ))}
      </nav>
      <div>
        <ul>
          {cats.map((cat) => (
            <li
              key={cat.src}
              ref={(node) => {
                const list = itemsRef.current;
                const item = {cat: cat, node};
                list.push(item);
                console.log(`✅ Adding cat to the map. Total cats: ${list.length}`);
                if (list.length > 10) {
                  console.log('❌ Too many cats in the list!');
                }
                return () => {
                  // 🚩 没有清理，这是个 bug！
                }
              }}
            >
              <img src={cat.src} />
            </li>
          ))}
        </ul>
      </div>
    </>
  );
}

function setupCatList() {
  const catList = [];
  for (let i = 0; i < 10; i++) {
    catList.push({type: 'neo', src: "https://placecats.com/neo/320/240?" + i});
  }
  for (let i = 0; i < 10; i++) {
    catList.push({type: 'millie', src: "https://placecats.com/millie/320/240?" + i});
  }

  return catList;
}

```

```css
div {
  width: 100%;
  overflow: hidden;
}

nav {
  text-align: center;
}

button {
  margin: .25rem;
}

ul,
li {
  list-style: none;
  white-space: nowrap;
}

li {
  display: inline;
  padding: 0.5rem;
}
```

</Sandpack>

**有了严格模式，你会立刻看到这里存在问题**。严格模式会为每个回调 ref 额外运行一次设置+清理循环。这个回调 ref 没有清理逻辑，所以它添加了 refs，却没有移除它们。这提示你缺少一个清理函数。

严格模式让你能够尽早发现回调 ref 中的错误。当你在严格模式下通过添加清理函数来修复回调时，你也 *同时* 修复了许多未来可能出现的生产环境 bug，比如前面的 “Scroll to” bug：

<Sandpack>

```js src/index.js
import { createRoot } from 'react-dom/client';
import {StrictMode} from 'react';
import './styles.css';

import App from './App';

const root = createRoot(document.getElementById("root"));
// ✅ 使用 StrictMode。
root.render(
  <StrictMode>
    <App />
  </StrictMode>
);
```

```js src/App.js active
import { useRef, useState } from "react";

export default function CatFriends() {
  const itemsRef = useRef([]);
  const [catList, setCatList] = useState(setupCatList);
  const [cat, setCat] = useState('neo');

  function scrollToCat(index) {
    const list = itemsRef.current;
    const {node} = list[index];
    node.scrollIntoView({
      behavior: "smooth",
      block: "nearest",
      inline: "center",
    });
  }

  const cats = catList.filter(c => c.type === cat)

  return (
    <>
      <nav>
        <button onClick={() => setCat('neo')}>Neo</button>
        <button onClick={() => setCat('millie')}>Millie</button>
      </nav>
      <hr />
      <nav>
        <span>Scroll to:</span>{cats.map((cat, index) => (
          <button key={cat.src} onClick={() => scrollToCat(index)}>
            {index}
          </button>
        ))}
      </nav>
      <div>
        <ul>
          {cats.map((cat) => (
            <li
              key={cat.src}
              ref={(node) => {
                const list = itemsRef.current;
                const item = {cat: cat, node};
                list.push(item);
                console.log(`✅ Adding cat to the map. Total cats: ${list.length}`);
                if (list.length > 10) {
                  console.log('❌ Too many cats in the list!');
                }
                return () => {
                  list.splice(list.indexOf(item), 1);
                  console.log(`❌ Removing cat from the map. Total cats: ${itemsRef.current.length}`);
                }
              }}
            >
              <img src={cat.src} />
            </li>
          ))}
        </ul>
      </div>
    </>
  );
}

function setupCatList() {
  const catList = [];
  for (let i = 0; i < 10; i++) {
    catList.push({type: 'neo', src: "https://placecats.com/neo/320/240?" + i});
  }
  for (let i = 0; i < 10; i++) {
    catList.push({type: 'millie', src: "https://placecats.com/millie/320/240?" + i});
  }

  return catList;
}

```

```css
div {
  width: 100%;
  overflow: hidden;
}

nav {
  text-align: center;
}

button {
  margin: .25rem;
}

ul,
li {
  list-style: none;
  white-space: nowrap;
}

li {
  display: inline;
  padding: 0.5rem;
}
```

</Sandpack>

现在，在 StrictMode 初始挂载时，ref 回调会全部执行设置、清理，然后再次执行设置：

```
...
✅ Adding animal to the map. Total animals: 10
...
❌ Removing animal from the map. Total animals: 0
...
✅ Adding animal to the map. Total animals: 10
```

**这是符合预期的。** 严格模式确认 ref 回调被正确清理，因此大小不会增长到预期值以上。修复之后，不会有内存泄漏，所有功能都能按预期工作。

如果没有严格模式，你很容易在四处点击后才发现某些功能坏了，从而错过这个 bug。严格模式让这些 bug 立刻显现出来，在你把它们推送到生产环境之前。

---
### 修复严格模式启用的弃用警告 {/*fixing-deprecation-warnings-enabled-by-strict-mode*/}

如果 `<StrictMode>` 树中的某个组件使用了以下已弃用的 API，React 会发出警告：

* `UNSAFE_` 类生命周期方法，例如 [`UNSAFE_componentWillMount`](/reference/react/Component#unsafe_componentwillmount)。[查看替代方案。](https://reactjs.org/blog/2018/03/27/update-on-async-rendering.html#migrating-from-legacy-lifecycles)

这些 API 主要用于较老的 [类组件](/reference/react/Component)，因此它们在现代应用中很少出现。
