---
title: hydrateRoot
---

<Intro>

`hydrateRoot` 让你可以在浏览器 DOM 节点内显示 React 组件，而该节点中的 HTML 内容此前是由 [`react-dom/server`.](/reference/react-dom/server) 生成的。

```js
const root = hydrateRoot(domNode, reactNode, options?)
```

</Intro>

<InlineToc />

---

## 参考 {/*reference*/}

### `hydrateRoot(domNode, reactNode, options?)` {/*hydrateroot*/}

调用 `hydrateRoot` 可以将 React “附加”到服务器环境中已经由 React 渲染好的现有 HTML 上。

```js
import { hydrateRoot } from 'react-dom/client';

const domNode = document.getElementById('root');
const root = hydrateRoot(domNode, reactNode);
```

React 会附加到 `domNode` 内部已有的 HTML，并接管其中 DOM 的管理。一个完全用 React 构建的应用通常只会有一次 `hydrateRoot` 调用，且对应其根组件。

[查看更多示例。](#usage)

#### 参数 {/*parameters*/}

* `domNode`：一个 [DOM 元素](https://developer.mozilla.org/en-US/docs/Web/API/Element)，它在服务器端被渲染为根元素。

* `reactNode`：用于渲染现有 HTML 的 “React 节点”。这通常是一段 JSX，例如 `<App />`，它是通过某个 `ReactDOM Server` 方法渲染出来的，例如 `renderToPipeableStream(<App />)`。

* **可选** `options`：一个包含此 React 根配置项的对象。

  * **可选** `onCaughtError`：当 React 在 Error Boundary 中捕获到错误时调用的回调。它会接收被 Error Boundary 捕获的 `error`，以及一个包含 `componentStack` 的 `errorInfo` 对象。
  * **可选** `onUncaughtError`：当抛出错误且未被 Error Boundary 捕获时调用的回调。它会接收被抛出的 `error`，以及一个包含 `componentStack` 的 `errorInfo` 对象。
  * **可选** `onRecoverableError`：当 React 自动从错误中恢复时调用的回调。它会接收 React 抛出的 `error`，以及一个包含 `componentStack` 的 `errorInfo` 对象。某些可恢复错误可能会把原始错误原因作为 `error.cause`。
  * **可选** `identifierPrefix`：React 用于通过 [`useId`.](/reference/react/useId) 生成 ID 的字符串前缀。在同一页面使用多个根时，这有助于避免冲突。该前缀必须与服务器端使用的前缀相同。


#### 返回值 {/*returns*/}

`hydrateRoot` 返回一个带有两个方法的对象：[`render`](#root-render) 和 [`unmount`.](#root-unmount)

#### 注意事项 {/*caveats*/}

* `hydrateRoot()` 期望渲染内容与服务端渲染内容完全一致。你应该把不匹配视为 bug 并修复它们。
* 在开发模式下，React 会在 hydration 期间对不匹配情况发出警告。如果存在不匹配，不能保证属性差异会被修补。这一点对性能很重要，因为在大多数应用中，不匹配很少见，因此验证所有标记会非常昂贵。
* 你的应用里大概率只需要一次 `hydrateRoot` 调用。如果你使用的是框架，它可能已经帮你调用过了。
* 如果你的应用是纯客户端渲染，且之前没有生成任何 HTML，那么不支持使用 `hydrateRoot()`。请改用 [`createRoot()`](/reference/react-dom/client/createRoot)。

---

### `root.render(reactNode)` {/*root-render*/}

调用 `root.render` 可以更新浏览器 DOM 元素中已 hydration 的 React 根里的某个 React 组件。

```js
root.render(<App />);
```

React 会更新已 hydration 的 `root` 中的 `<App />`。

[查看更多示例。](#usage)

#### 参数 {/*root-render-parameters*/}

* `reactNode`：你想要更新的 “React 节点”。这通常是一段 JSX，例如 `<App />`，但你也可以传入通过 [`createElement()`](/reference/react/createElement) 构造的 React 元素、字符串、数字、`null` 或 `undefined`。


#### 返回值 {/*root-render-returns*/}

`root.render` 返回 `undefined`。

#### 注意事项 {/*root-render-caveats*/}

* 如果你在 root 完成 hydration 之前调用 `root.render`，React 会清除现有的服务端渲染 HTML 内容，并把整个 root 切换为客户端渲染。

---

### `root.unmount()` {/*root-unmount*/}

调用 `root.unmount` 可以销毁 React 根内部已渲染的树。

```js
root.unmount();
```

一个完全用 React 构建的应用通常不会调用任何 `root.unmount`。

这主要适用于你的 React root 的 DOM 节点（或其任意祖先）可能会被其他代码从 DOM 中移除的情况。例如，设想一个 jQuery 选项卡面板会把未激活的标签页从 DOM 中移除。如果某个标签页被移除，其中的所有内容（包括内部的 React roots）也会一并从 DOM 中移除。你需要通过调用 `root.unmount` 来告诉 React 停止管理已移除 root 的内容。否则，已移除 root 内的组件不会进行清理，也不会释放诸如订阅之类的资源。

调用 `root.unmount` 会卸载 root 中的所有组件，并将 React 从 root DOM 节点上“分离”，包括移除树中的任何事件处理器或状态。


#### 参数 {/*root-unmount-parameters*/}

`root.unmount` 不接受任何参数。


#### 返回值 {/*root-unmount-returns*/}

`root.unmount` 返回 `undefined`。

#### 注意事项 {/*root-unmount-caveats*/}

* 调用 `root.unmount` 会卸载树中的所有组件，并将 React 从 root DOM 节点上“分离”。

* 一旦调用了 `root.unmount`，你就不能再对该 root 调用 `root.render`。尝试对已卸载的 root 调用 `root.render` 会抛出 “Cannot update an unmounted root” 错误。

---

## 使用 {/*usage*/}

### 对服务器渲染的 HTML 进行 hydration {/*hydrating-server-rendered-html*/}

如果你的应用 HTML 是由 [`react-dom/server`](/reference/react-dom/client/createRoot) 生成的，那么你需要在客户端对其进行 *hydrate*。

```js [[1, 3, "document.getElementById('root')"], [2, 3, "<App />"]]
import { hydrateRoot } from 'react-dom/client';

hydrateRoot(document.getElementById('root'), <App />);
```

这会把你的应用的服务端 HTML 在 <CodeStep step={1}>浏览器 DOM 节点</CodeStep> 内进行 hydration，并配上 <CodeStep step={2}>React 组件</CodeStep>。通常你只需要在启动时执行一次。如果你使用的是框架，它可能已经在幕后帮你做了这件事。

要对你的应用进行 hydration，React 会把你组件的逻辑“附加”到来自服务器的初始生成 HTML 上。Hydration 会把来自服务器的初始 HTML 快照转换为一个可在浏览器中运行的完整交互式应用。

<Sandpack>

```html public/index.html
<!--
  <div id="root">...</div> 内的 HTML 内容
  是由 react-dom/server 根据 App 生成的。
-->
<div id="root"><h1>Hello, world!</h1><button>You clicked me <!-- -->0<!-- --> times</button></div>
```

```js src/index.js active
import './styles.css';
import { hydrateRoot } from 'react-dom/client';
import App from './App.js';

hydrateRoot(
  document.getElementById('root'),
  <App />
);
```

```js src/App.js
import { useState } from 'react';

export default function App() {
  return (
    <>
      <h1>Hello, world!</h1>
      <Counter />
    </>
  );
}

function Counter() {
  const [count, setCount] = useState(0);
  return (
    <button onClick={() => setCount(count + 1)}>
      你点击了我 {count} 次
    </button>
  );
}
```

</Sandpack>

你不应该需要再次调用 `hydrateRoot`，也不应该在更多地方调用它。从这一点开始，React 会接管并管理你应用的 DOM。要更新 UI，请改为在组件内部使用 [state](/reference/react/useState)。

<Pitfall>

传给 `hydrateRoot` 的 React 树需要生成与服务端**相同的输出**。

这一点对用户体验很重要。用户会在你的 JavaScript 代码加载之前，先看到一段时间的服务端生成 HTML。服务端渲染通过展示输出的 HTML 快照，营造出应用加载更快的错觉。突然显示不同的内容会破坏这种错觉。这就是为什么服务端渲染输出必须与客户端的初始渲染输出匹配。

导致 hydration 错误的最常见原因包括：

* root 节点内由 React 生成的 HTML 周围存在额外空白字符（例如换行）。
* 在渲染逻辑中使用类似 `typeof window !== 'undefined'` 的检查。
* 在渲染逻辑中使用仅浏览器可用的 API，例如 [`window.matchMedia`](https://developer.mozilla.org/en-US/docs/Web/API/Window/matchMedia)。
* 在服务端和客户端渲染不同的数据。

React 会从某些 hydration 错误中恢复，但**你必须像修复其他 bug 一样修复它们。** 在最好的情况下，它们会导致性能变慢；在最坏的情况下，事件处理器可能会被绑定到错误的元素上。

</Pitfall>

---

### 对整个文档进行 hydration {/*hydrating-an-entire-document*/}

完全用 React 构建的应用可以把整个文档都作为 JSX 渲染，包括 [`<html>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/html) 标签：

```js {3,13}
function App() {
  return (
    <html>
      <head>
        <meta charSet="utf-8" />
        <meta name="viewport" content="width=device-width, initial-scale=1" />
        <link rel="stylesheet" href="/styles.css"></link>
        <title>我的应用</title>
      </head>
      <body>
        <Router />
      </body>
    </html>
  );
}
```

要对整个文档进行 hydration，请将 [`document`](https://developer.mozilla.org/en-US/docs/Web/API/Window/document) 全局对象作为 `hydrateRoot` 的第一个参数：

```js {4}
import { hydrateRoot } from 'react-dom/client';
import App from './App.js';

hydrateRoot(document, <App />);
```

---

### 抑制不可避免的 hydration 不匹配错误 {/*suppressing-unavoidable-hydration-mismatch-errors*/}

如果单个元素的属性或文本内容在服务端和客户端之间不可避免地不同（例如时间戳），你可以忽略 hydration 不匹配警告。

要忽略某个元素上的 hydration 警告，请添加 `suppressHydrationWarning={true}`：

<Sandpack>

```html public/index.html
<!--
  <div id="root">...</div> 内的 HTML 内容
  是由 react-dom/server 根据 App 生成的。
-->
<div id="root"><h1>当前日期：<!-- -->01/01/2020</h1></div>
```

```js src/index.js
import './styles.css';
import { hydrateRoot } from 'react-dom/client';
import App from './App.js';

hydrateRoot(document.getElementById('root'), <App />);
```

```js src/App.js active
export default function App() {
  return (
    <h1 suppressHydrationWarning={true}>
      当前日期：{new Date().toLocaleDateString()}
    </h1>
  );
}
```

</Sandpack>

这只会在一层深度内生效，且其设计用途是作为一种应急出口。不要过度使用。React **不会** 尝试修补不匹配的文本内容。

---

### 处理不同的客户端和服务端内容 {/*handling-different-client-and-server-content*/}

如果你有意需要在服务端和客户端渲染不同的内容，可以采用两遍渲染。客户端上渲染不同内容的组件可以读取类似 `isClient` 的 [state 变量](/reference/react/useState)，并在 [Effect](/reference/react/useEffect) 中把它设为 `true`：

<Sandpack>

```html public/index.html
<!--
  <div id="root">...</div> 内的 HTML 内容
  是由 react-dom/server 根据 App 生成的。
-->
<div id="root"><h1>Is Server</h1></div>
```

```js src/index.js
import './styles.css';
import { hydrateRoot } from 'react-dom/client';
import App from './App.js';

hydrateRoot(document.getElementById('root'), <App />);
```

{/* kind of an edge case, seems fine to use this hack here */}
```js {expectedErrors: {'react-compiler': [7]}} src/App.js active
import { useState, useEffect } from "react";

export default function App() {
  const [isClient, setIsClient] = useState(false);

  useEffect(() => {
    setIsClient(true);
  }, []);

  return (
    <h1>
      {isClient ? 'Is Client' : 'Is Server'}
    </h1>
  );
}
```

</Sandpack>

这样，初始渲染会输出与服务端相同的内容，从而避免不匹配；但在 hydration 之后会立即同步进行额外一次渲染。

<Pitfall>

这种方式会让 hydration 变慢，因为你的组件必须渲染两次。请注意慢速连接下的用户体验。JavaScript 代码的加载可能会比初始 HTML 渲染晚很多，因此在 hydration 之后立刻渲染不同的 UI，对用户来说也可能会显得突兀。

</Pitfall>

---

### 更新已 hydration 的 root 组件 {/*updating-a-hydrated-root-component*/}

在 root 完成 hydration 之后，你可以调用 [`root.render`](#root-render) 来更新 root React 组件。**与 [`createRoot`](/reference/react-dom/client/createRoot) 不同，你通常不需要这样做，因为初始内容已经以 HTML 的形式渲染出来了。**

如果你在 hydration 之后某个时刻调用 `root.render`，并且组件树结构与之前渲染的内容匹配，React 会[保留状态。](/learn/preserving-and-resetting-state) 注意你仍然可以在输入框中输入，这意味着本示例中每秒重复调用 `render` 所带来的更新并不会破坏内容：

<Sandpack>

```html public/index.html
<!--
  <div id="root">...</div> 内的所有 HTML 内容都
  是通过使用 react-dom/server 渲染 <App /> 生成的。
-->
<div id="root"><h1>Hello, world! <!-- -->0</h1><input placeholder="在这里输入一些内容"/></div>
```

```js src/index.js active
import { hydrateRoot } from 'react-dom/client';
import './styles.css';
import App from './App.js';

const root = hydrateRoot(
  document.getElementById('root'),
  <App counter={0} />
);

let i = 0;
setInterval(() => {
  root.render(<App counter={i} />);
  i++;
}, 1000);
```

```js src/App.js
export default function App({counter}) {
  return (
    <>
      <h1>Hello, world! {counter}</h1>
      <input placeholder="在这里输入一些内容" />
    </>
  );
}
```

</Sandpack>

在已 hydration 的 root 上调用 [`root.render`](#root-render) 并不常见。通常你会改为在某个组件内部[更新 state](/reference/react/useState)。

### 生产环境中的错误日志记录 {/*error-logging-in-production*/}

默认情况下，React 会将所有错误记录到控制台。要实现你自己的错误上报，可以提供可选的错误处理 root 配置项 `onUncaughtError`、`onCaughtError` 和 `onRecoverableError`：

```js [[1, 7, "onCaughtError"], [2, 7, "error", 1], [3, 7, "errorInfo"], [4, 11, "componentStack", 15]]
import { hydrateRoot } from "react-dom/client";
import App from "./App.js";
import { reportCaughtError } from "./reportError";

const container = document.getElementById("root");
const root = hydrateRoot(container, <App />, {
  onCaughtError: (error, errorInfo) => {
    if (error.message !== "Known error") {
      reportCaughtError({
        error,
        componentStack: errorInfo.componentStack,
      });
    }
  },
});
```

<CodeStep step={1}>onCaughtError</CodeStep> 选项是一个函数，它接收两个参数调用：

1. 被抛出的 <CodeStep step={2}>error</CodeStep>。
2. 一个包含错误 <CodeStep step={4}>componentStack</CodeStep> 的 <CodeStep step={3}>errorInfo</CodeStep> 对象。

结合 `onUncaughtError` 和 `onRecoverableError`，你可以实现自己的错误上报系统：

<Sandpack>

```js src/reportError.js
function reportError({ type, error, errorInfo }) {
  // 具体实现由你决定。
  // `console.error()` 仅用于演示目的。
  console.error(type, error, "Component Stack: ");
  console.error("Component Stack: ", errorInfo.componentStack);
}

export function onCaughtErrorProd(error, errorInfo) {
  if (error.message !== "Known error") {
    reportError({ type: "Caught", error, errorInfo });
  }
}

export function onUncaughtErrorProd(error, errorInfo) {
  reportError({ type: "Uncaught", error, errorInfo });
}

export function onRecoverableErrorProd(error, errorInfo) {
  reportError({ type: "Recoverable", error, errorInfo });
}
```

```js src/index.js active
import { hydrateRoot } from "react-dom/client";
import App from "./App.js";
import {
  onCaughtErrorProd,
  onRecoverableErrorProd,
  onUncaughtErrorProd,
} from "./reportError";

const container = document.getElementById("root");
hydrateRoot(container, <App />, {
  // 请记得在开发环境中移除这些选项，以便利用
  // React 的默认处理器，或者为开发环境实现你自己的覆盖层。
  // 这里无条件指定这些处理器仅用于演示目的。
  onCaughtError: onCaughtErrorProd,
  onRecoverableError: onRecoverableErrorProd,
  onUncaughtError: onUncaughtErrorProd,
});
```

```js src/App.js
import { Component, useState } from "react";

function Boom() {
  foo.bar = "baz";
}

class ErrorBoundary extends Component {
  state = { hasError: false };

  static getDerivedStateFromError(error) {
    return { hasError: true };
  }

  render() {
    if (this.state.hasError) {
      return <h1>出错了。</h1>;
    }
    return this.props.children;
  }
}

export default function App() {
  const [triggerUncaughtError, settriggerUncaughtError] = useState(false);
  const [triggerCaughtError, setTriggerCaughtError] = useState(false);

  return (
    <>
      <button onClick={() => settriggerUncaughtError(true)}>
        触发未捕获错误
      </button>
      {triggerUncaughtError && <Boom />}
      <button onClick={() => setTriggerCaughtError(true)}>
        触发已捕获错误
      </button>
      {triggerCaughtError && (
        <ErrorBoundary>
          <Boom />
        </ErrorBoundary>
      )}
    </>
  );
}
```

```html public/index.html hidden
<!DOCTYPE html>
<html>
<head>
  <title>我的应用</title>
</head>
<body>
<!--
  故意使用与服务端渲染内容不同的 HTML 内容，以触发可恢复错误。
-->
<div id="root">Hydration 之前的服务端内容。</div>
</body>
</html>
```
</Sandpack>

## 故障排查 {/*troubleshooting*/}


### 我收到一个错误："You passed a second argument to root.render" {/*im-getting-an-error-you-passed-a-second-argument-to-root-render*/}

一个常见的错误是把 `hydrateRoot` 的选项传给 `root.render(...)`：

<ConsoleBlock level="error">

警告：你向 root.render(...) 传递了第二个参数，但它只接受一个参数。

</ConsoleBlock>

要修复，请将根选项传给 `hydrateRoot(...)`，而不是 `root.render(...)`：
```js {2,5}
// 🚩 错误：root.render 只接受一个参数。
root.render(App, {onUncaughtError});

// ✅ 正确：将选项传给 createRoot。
const root = hydrateRoot(container, <App />, {onUncaughtError});
```
