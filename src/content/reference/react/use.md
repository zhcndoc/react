---
title: use
---

<Intro>

`use` 是一个 React API，它让你能够读取像 [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) 或 [context](/learn/passing-data-deeply-with-context) 这样的资源的值。

```js
const value = use(resource);
```

</Intro>

<InlineToc />

---

## 参考 {/*reference*/}

### `use(resource)` {/*use*/}

在组件中调用 `use` 来读取像 [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) 或 [context](/learn/passing-data-deeply-with-context) 这样的资源的值。

```jsx
import { use } from 'react';

function MessageComponent({ messagePromise }) {
  const message = use(messagePromise);
  const theme = use(ThemeContext);
  // ...
```

与 React Hooks 不同，`use` 可以在循环和条件语句（如 `if`）中调用。与 React Hooks 一样，调用 `use` 的函数必须是一个组件或 Hook。

当传入 Promise 时，`use` API 会与 [`Suspense`](/reference/react/Suspense) 和 [Error Boundaries](/reference/react/Component#catching-rendering-errors-with-an-error-boundary) 集成。调用 `use` 的组件在传给 `use` 的 Promise 处于 pending 状态时会 *挂起*。如果调用 `use` 的组件被包裹在 Suspense 边界中，则会显示 fallback。一旦 Promise 被解决，Suspense 的 fallback 会被使用 `use` API 返回的数据渲染出的组件所替换。如果传给 `use` 的 Promise 被拒绝，则会显示最近的 Error Boundary 的 fallback。

[查看更多示例。](#usage)

#### 参数 {/*parameters*/}

* `resource`：这是你想从中读取值的数据源。资源可以是 [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) 或 [context](/learn/passing-data-deeply-with-context)。

#### 返回值 {/*returns*/}

`use` API 返回从资源中读取到的值，例如 [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) 或 [context](/learn/passing-data-deeply-with-context) 已解决后的值。

#### 注意事项 {/*caveats*/}

* `use` API 必须在组件或 Hook 内部调用。
* 在 [Server Component](/reference/rsc/server-components) 中获取数据时，优先使用 `async` 和 `await`，而不是 `use`。`async` 和 `await` 会从 `await` 被调用的位置继续渲染，而 `use` 会在数据解决后重新渲染组件。
* 更推荐在 [Server Components](/reference/rsc/server-components) 中创建 Promise 并将其传递给 [Client Components](/reference/rsc/use-client)，而不是在 Client Components 中创建 Promise。Client Components 中创建的 Promise 会在每次渲染时重新创建。从 Server Component 传递给 Client Component 的 Promise 在重新渲染之间是稳定的。[查看此示例](#streaming-data-from-server-to-client)。

---

## 用法 {/*usage*/}

### 使用 `use` 读取 context {/*reading-context-with-use*/}

当将 [context](/learn/passing-data-deeply-with-context) 传给 `use` 时，其工作方式类似于 [`useContext`](/reference/react/useContext)。`useContext` 必须在组件顶层调用，而 `use` 可以在 `if` 这样的条件语句和 `for` 这样的循环中调用。与 `useContext` 相比，更推荐使用 `use`，因为它更灵活。

```js [[2, 4, "theme"], [1, 4, "ThemeContext"]]
import { use } from 'react';

function Button() {
  const theme = use(ThemeContext);
  // ...
```

`use` 会为你传入的 <CodeStep step={1}>context</CodeStep> 返回 <CodeStep step={2}>context 值</CodeStep>。为了确定 context 值，React 会搜索组件树，并找到该特定 context 的**最近的、位于上方的 context 提供者**。

要向 `Button` 传递 context，请将它或它的父组件包裹在相应的 context provider 中。

```js [[1, 3, "ThemeContext"], [2, 3, "\\"dark\\""], [1, 5, "ThemeContext"]]
function MyPage() {
  return (
    <ThemeContext value="dark">
      <Form />
    </ThemeContext>
  );
}

function Form() {
  // ... 在内部渲染按钮 ...
}
```

Provider 和 `Button` 之间有多少层组件都没有关系。当 `Form` 内部的任意位置的 `Button` 调用 `use(ThemeContext)` 时，它都会接收到 `"dark"` 作为值。

与 [`useContext`](/reference/react/useContext) 不同，<CodeStep step={2}>`use`</CodeStep> 可以在 <CodeStep step={1}>`if`</CodeStep> 这样的条件语句和循环中调用。

```js [[1, 2, "if"], [2, 3, "use"]]
function HorizontalRule({ show }) {
  if (show) {
    const theme = use(ThemeContext);
    return <hr className={theme} />;
  }
  return false;
}
```

<CodeStep step={2}>`use`</CodeStep> 是在 <CodeStep step={1}>`if`</CodeStep> 语句内部调用的，这使你能够有条件地从 Context 中读取值。

<Pitfall>

和 `useContext` 一样，`use(context)` 总是会查找调用它的组件**上方**最近的 context provider。它会向上搜索，**不会**考虑你调用 `use(context)` 的那个组件内部的 context provider。

</Pitfall>

<Sandpack>

```js
import { createContext, use } from 'react';

const ThemeContext = createContext(null);

export default function MyApp() {
  return (
    <ThemeContext value="dark">
      <Form />
    </ThemeContext>
  )
}

function Form() {
  return (
    <Panel title="Welcome">
      <Button show={true}>Sign up</Button>
      <Button show={false}>Log in</Button>
    </Panel>
  );
}

function Panel({ title, children }) {
  const theme = use(ThemeContext);
  const className = 'panel-' + theme;
  return (
    <section className={className}>
      <h1>{title}</h1>
      {children}
    </section>
  )
}

function Button({ show, children }) {
  if (show) {
    const theme = use(ThemeContext);
    const className = 'button-' + theme;
    return (
      <button className={className}>
        {children}
      </button>
    );
  }
  return false
}
```

```css
.panel-light,
.panel-dark {
  border: 1px solid black;
  border-radius: 4px;
  padding: 20px;
}
.panel-light {
  color: #222;
  background: #fff;
}

.panel-dark {
  color: #fff;
  background: rgb(23, 32, 42);
}

.button-light,
.button-dark {
  border: 1px solid #777;
  padding: 5px;
  margin-right: 10px;
  margin-top: 10px;
}

.button-dark {
  background: #222;
  color: #fff;
}

.button-light {
  background: #fff;
  color: #222;
}
```

</Sandpack>

### 将数据从服务器流式传输到客户端 {/*streaming-data-from-server-to-client*/}

可以通过将 Promise 作为 prop 从 <CodeStep step={1}>Server Component</CodeStep> 传递给 <CodeStep step={2}>Client Component</CodeStep>，来把数据从服务器流式传输到客户端。

```js [[1, 4, "App"], [2, 2, "Message"], [3, 7, "Suspense"], [4, 8, "messagePromise", 30], [4, 5, "messagePromise"]]
import { fetchMessage } from './lib.js';
import { Message } from './message.js';

export default function App() {
  const messagePromise = fetchMessage();
  return (
    <Suspense fallback={<p>waiting for message...</p>}>
      <Message messagePromise={messagePromise} />
    </Suspense>
  );
}
```

然后 <CodeStep step={2}>Client Component</CodeStep> 会接收 <CodeStep step={4}>它作为 prop 收到的 Promise</CodeStep>，并将其传给 <CodeStep step={5}>`use`</CodeStep> API。这使得 <CodeStep step={2}>Client Component</CodeStep> 能够读取最初由 Server Component 创建的 <CodeStep step={4}>Promise</CodeStep> 的值。

```js [[2, 6, "Message"], [4, 6, "messagePromise"], [4, 7, "messagePromise"], [5, 7, "use"]]
// message.js
'use client';

import { use } from 'react';

export function Message({ messagePromise }) {
  const messageContent = use(messagePromise);
  return <p>这里是消息：{messageContent}</p>;
}
```
由于 <CodeStep step={2}>`Message`</CodeStep> 被 <CodeStep step={3}>[`Suspense`](/reference/react/Suspense)</CodeStep> 包裹，因此在 Promise 解决之前会显示 fallback。Promise 解决后，<CodeStep step={5}>`use`</CodeStep> API 会读取该值，而 <CodeStep step={2}>`Message`</CodeStep> 组件会替换 Suspense fallback。

<Sandpack>

```js src/message.js active
"use client";

import { use, Suspense } from "react";

function Message({ messagePromise }) {
  const messageContent = use(messagePromise);
  return <p>这里是消息：{messageContent}</p>;
}

export function MessageContainer({ messagePromise }) {
  return (
    <Suspense fallback={<p>⌛正在下载消息...</p>}>
      <Message messagePromise={messagePromise} />
    </Suspense>
  );
}
```

```js src/App.js hidden
import { useState } from "react";
import { MessageContainer } from "./message.js";

function fetchMessage() {
  return new Promise((resolve) => setTimeout(resolve, 1000, "⚛️"));
}

export default function App() {
  const [messagePromise, setMessagePromise] = useState(null);
  const [show, setShow] = useState(false);
  function download() {
    setMessagePromise(fetchMessage());
    setShow(true);
  }

  if (show) {
    return <MessageContainer messagePromise={messagePromise} />;
  } else {
    return <button onClick={download}>Download message</button>;
  }
}
```

```js src/index.js hidden
import React, { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import './styles.css';

// TODO: 更新此示例以使用
// 一旦 Codesandbox Server Component
// 演示环境创建完成
import App from './App';

const root = createRoot(document.getElementById('root'));
root.render(
  <StrictMode>
    <App />
  </StrictMode>
);
```

</Sandpack>

<Note>

当把 Promise 从 Server Component 传递给 Client Component 时，其解析后的值必须是可序列化的，才能在 server 和 client 之间传递。像函数这样的数据类型不可序列化，因此不能作为此类 Promise 的解析值。

</Note>


<DeepDive>

#### 我应该在 Server 还是 Client Component 中解析 Promise？ {/*resolve-promise-in-server-or-client-component*/}

Promise 可以从 Server Component 传递到 Client Component，并在 Client Component 中通过 `use` API 解析。你也可以在 Server Component 中使用 `await` 解析 Promise，然后将所需数据作为 prop 传给 Client Component。

```js
export default async function App() {
  const messageContent = await fetchMessage();
  return <Message messageContent={messageContent} />
}
```

但是，在 [Server Component](/reference/rsc/server-components) 中使用 `await` 会阻塞其渲染，直到 `await` 语句完成。将 Promise 从 Server Component 传递给 Client Component 可以避免 Promise 阻塞 Server Component 的渲染。

</DeepDive>

### 处理被拒绝的 Promise {/*dealing-with-rejected-promises*/}

在某些情况下，传给 `use` 的 Promise 可能会被拒绝。你可以通过以下两种方式处理被拒绝的 Promise：

1. [使用 Error Boundary 向用户显示错误。](#displaying-an-error-to-users-with-error-boundary)
2. [使用 `Promise.catch` 提供一个替代值](#providing-an-alternative-value-with-promise-catch)

<Pitfall>
不能在 try-catch 块中调用 `use`。不要使用 try-catch 块，而是[将组件包裹在 Error Boundary 中](#displaying-an-error-to-users-with-error-boundary)，或者[使用 Promise 的 `.catch` 方法提供一个可供 `use` 使用的替代值](#providing-an-alternative-value-with-promise-catch)。
</Pitfall>

#### 使用 Error Boundary 向用户显示错误 {/*displaying-an-error-to-users-with-error-boundary*/}

如果你希望在 Promise 被拒绝时向用户显示错误，可以使用 [Error Boundary](/reference/react/Component#catching-rendering-errors-with-an-error-boundary)。要使用 Error Boundary，请将你调用 `use` API 的组件包裹在 Error Boundary 中。如果传给 `use` 的 Promise 被拒绝，则会显示 Error Boundary 的 fallback。

<Sandpack>

```js src/message.js active
"use client";

import { use, Suspense } from "react";
import { ErrorBoundary } from "react-error-boundary";

export function MessageContainer({ messagePromise }) {
  return (
    <ErrorBoundary fallback={<p>⚠️出了点问题</p>}>
      <Suspense fallback={<p>⌛正在下载消息...</p>}>
        <Message messagePromise={messagePromise} />
      </Suspense>
    </ErrorBoundary>
  );
}

function Message({ messagePromise }) {
  const content = use(messagePromise);
  return <p>这里是消息：{content}</p>;
}
```

```js src/App.js hidden
import { useState } from "react";
import { MessageContainer } from "./message.js";

function fetchMessage() {
  return new Promise((resolve, reject) => setTimeout(reject, 1000));
}

export default function App() {
  const [messagePromise, setMessagePromise] = useState(null);
  const [show, setShow] = useState(false);
  function download() {
    setMessagePromise(fetchMessage());
    setShow(true);
  }

  if (show) {
    return <MessageContainer messagePromise={messagePromise} />;
  } else {
    return <button onClick={download}>Download message</button>;
  }
}
```

```js src/index.js hidden
import React, { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import './styles.css';

// TODO: 更新此示例以使用
// 一旦 Codesandbox Server Component
// 演示环境创建完成
import App from './App';

const root = createRoot(document.getElementById('root'));
root.render(
  <StrictMode>
    <App />
  </StrictMode>
);
```

```json package.json hidden
{
  "dependencies": {
    "react": "19.0.0",
    "react-dom": "19.0.0",
    "react-scripts": "^5.0.0",
    "react-error-boundary": "4.0.3"
  },
  "main": "/index.js"
}
```
</Sandpack>

#### 使用 `Promise.catch` 提供一个替代值 {/*providing-an-alternative-value-with-promise-catch*/}

如果你希望在传给 `use` 的 Promise 被拒绝时提供一个替代值，可以使用 Promise 的 <CodeStep step={1}>[`catch`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/catch)</CodeStep> 方法。

```js [[1, 6, "catch"],[2, 7, "return"]]
import { Message } from './message.js';

export default function App() {
  const messagePromise = new Promise((resolve, reject) => {
    reject();
  }).catch(() => {
    return "未找到新消息。";
  });

  return (
    <Suspense fallback={<p>waiting for message...</p>}>
      <Message messagePromise={messagePromise} />
    </Suspense>
  );
}
```

要使用 Promise 的 <CodeStep step={1}>`catch`</CodeStep> 方法，请在 Promise 对象上调用 <CodeStep step={1}>`catch`</CodeStep>。<CodeStep step={1}>`catch`</CodeStep> 接收一个参数：一个以错误消息作为参数的函数。传给 <CodeStep step={1}>`catch`</CodeStep> 的函数所 <CodeStep step={2}>返回</CodeStep>的任何内容，都将作为 Promise 的已解决值使用。

---

## 故障排查 {/*troubleshooting*/}

### “Suspense 异常：这不是真正的错误！” {/*suspense-exception-error*/}

你要么是在 React 组件或 Hook 函数之外调用了 `use`，要么是在 try–catch 块中调用了 `use`。如果你是在 try–catch 块中调用 `use`，请将你的组件包裹在 Error Boundary 中，或者调用 Promise 的 `catch` 来捕获错误，并用另一个值来解析该 Promise。[查看这些示例](#dealing-with-rejected-promises)。

如果你是在 React 组件或 Hook 函数之外调用了 `use`，请将 `use` 调用移动到 React 组件或 Hook 函数中。

```jsx
function MessageComponent({messagePromise}) {
  function download() {
    // ❌ 调用 `use` 的函数不是组件或 Hook
    const message = use(messagePromise);
    // ...
```

相反，请在任何组件闭包之外调用 `use`，也就是在调用 `use` 的函数本身是组件或 Hook 的地方。

```jsx
function MessageComponent({messagePromise}) {
  // ✅ `use` 正在从组件中被调用。
  const message = use(messagePromise);
  // ...
```
