---
title: createRoot
---

<Intro>

`createRoot` 可让你创建一个 root，以在浏览器 DOM 节点内显示 React 组件。

```js
const root = createRoot(domNode, options?)
```

</Intro>

<InlineToc />

---

## 参考 {/*reference*/}

### `createRoot(domNode, options?)` {/*createroot*/}

调用 `createRoot` 可创建一个 React root，用于在浏览器 DOM 元素内显示内容。

```js
import { createRoot } from 'react-dom/client';

const domNode = document.getElementById('root');
const root = createRoot(domNode);
```

React 会为 `domNode` 创建一个 root，并接管其内部 DOM 的管理。创建 root 后，你需要调用 [`root.render`](#root-render) 才能在其中显示 React 组件：

```js
root.render(<App />);
```

一个完全使用 React 构建的应用通常只需要为其根组件调用一次 `createRoot`。如果页面只是在某些部分零散地使用 React，则可以根据需要创建任意多个独立 root。

[查看更多示例。](#usage)

#### 参数 {/*parameters*/}

* `domNode`：一个 [DOM 元素。](https://developer.mozilla.org/en-US/docs/Web/API/Element) React 会为这个 DOM 元素创建一个 root，并允许你在该 root 上调用函数，例如用 `render` 显示渲染后的 React 内容。

* **可选** `options`：一个包含此 React root 选项的对象。

  * **可选** `onCaughtError`：当 React 在错误边界中捕获到错误时调用的回调。调用时会传入被错误边界捕获的 `error`，以及一个包含 `componentStack` 的 `errorInfo` 对象。
  * **可选** `onUncaughtError`：当抛出错误但未被错误边界捕获时调用的回调。调用时会传入抛出的 `error`，以及一个包含 `componentStack` 的 `errorInfo` 对象。
  * **可选** `onRecoverableError`：当 React 自动从错误中恢复时调用的回调。调用时会传入 React 抛出的 `error`，以及一个包含 `componentStack` 的 `errorInfo` 对象。某些可恢复错误可能会在 `error.cause` 中包含原始错误原因。
  * **可选** `identifierPrefix`：React 用于由 [`useId`](/reference/react/useId) 生成的 ID 的字符串前缀。在同一页面使用多个 root 时，这有助于避免冲突。

#### 返回值 {/*returns*/}

`createRoot` 返回一个包含两个方法的对象：[`render`](#root-render) 和 [`unmount`.](#root-unmount)

#### 注意事项 {/*caveats*/}
* 如果你的应用是服务端渲染的，则不支持使用 `createRoot()`。请改用 [`hydrateRoot()`](/reference/react-dom/client/hydrateRoot)。
* 你的应用中大概率只需要调用一次 `createRoot`。如果你使用框架，它可能已经替你完成了这次调用。
* 当你想要在 DOM 树中某个不是你组件子节点的不同位置渲染一段 JSX 时（例如模态框或提示框），请使用 [`createPortal`](/reference/react-dom/createPortal) 而不是 `createRoot`。

---

### `root.render(reactNode)` {/*root-render*/}

调用 `root.render` 可将一段 [JSX](/learn/writing-markup-with-jsx)（“React 节点”）显示到 React root 的浏览器 DOM 节点中。

```js
root.render(<App />);
```

React 会在 `root` 中显示 `<App />`，并接管其内部 DOM 的管理。

[查看更多示例。](#usage)

#### 参数 {/*root-render-parameters*/}

* `reactNode`：你想要显示的 *React 节点*。这通常是一段 JSX，例如 `<App />`，但你也可以传入由 [`createElement()`](/reference/react/createElement) 构造的 React 元素、字符串、数字、`null` 或 `undefined`。


#### 返回值 {/*root-render-returns*/}

`root.render` 返回 `undefined`。

#### 注意事项 {/*root-render-caveats*/}

* 你第一次调用 `root.render` 时，React 会在将 React 组件渲染进去之前清空 React root 内现有的所有 HTML 内容。

* 如果你的 root 的 DOM 节点中包含由 React 在服务端或构建期间生成的 HTML，请改用 [`hydrateRoot()`](/reference/react-dom/client/hydrateRoot)，它会将事件处理函数附加到现有 HTML 上。

* 如果你在同一个 root 上多次调用 `render`，React 会根据需要更新 DOM，以反映你传入的最新 JSX。React 会通过与之前渲染的树进行["匹配"](/learn/preserving-and-resetting-state)来决定哪些 DOM 部分可以复用，哪些需要重新创建。在同一个 root 上再次调用 `render`，类似于在 root 组件上调用 [`set` 函数](/reference/react/useState#setstate)：React 会避免不必要的 DOM 更新。

* 尽管渲染一旦开始就是同步的，但 `root.render(...)` 本身并不是同步的。这意味着在 `root.render()` 之后的代码，可能会在此次渲染的任何效果（`useLayoutEffect`、`useEffect`）执行之前运行。这通常没有问题，也很少需要调整。在极少数效果时序很重要的情况下，你可以将 `root.render(...)` 包裹在 [`flushSync`](https://react.dev/reference/react-dom/flushSync) 中，以确保初始渲染完全同步执行。

  ```js
  const root = createRoot(document.getElementById('root'));
  root.render(<App />);
  // 🚩 HTML 目前还不会包含已渲染的 <App />：
  console.log(document.body.innerHTML);
  ```

---

### `root.unmount()` {/*root-unmount*/}

调用 `root.unmount` 可销毁 React root 内已渲染的树。

```js
root.unmount();
```

一个完全使用 React 构建的应用通常不会调用 `root.unmount`。

当你的 React root 的 DOM 节点（或其任意祖先节点）可能会被其他代码从 DOM 中移除时，这个方法尤其有用。例如，设想一个 jQuery 标签页面板会把未激活的标签页从 DOM 中移除。如果某个标签页被移除，其中的一切内容（包括内部的 React root）也会一并从 DOM 中移除。在这种情况下，你需要通过调用 `root.unmount` 告诉 React “停止”管理被移除的 root 内容。否则，被移除 root 中的组件不会知道需要清理并释放全局资源，比如订阅。

调用 `root.unmount` 会卸载 root 中所有组件，并将 React 从 root DOM 节点上“分离”，包括移除树中的任何事件处理函数或状态。


#### 参数 {/*root-unmount-parameters*/}

`root.unmount` 不接受任何参数。


#### 返回值 {/*root-unmount-returns*/}

`root.unmount` 返回 `undefined`。

#### 注意事项 {/*root-unmount-caveats*/}

* 调用 `root.unmount` 会卸载树中的所有组件，并将 React 从 root DOM 节点上“分离”。

* 一旦调用了 `root.unmount`，你就不能再在同一个 root 上调用 `root.render`。尝试在已卸载的 root 上调用 `root.render` 会抛出 “Cannot update an unmounted root” 错误。不过，在某个 DOM 节点上的之前的 root 被卸载后，你可以为同一个 DOM 节点创建一个新的 root。

---

## 用法 {/*usage*/}

### 渲染一个完全使用 React 构建的应用 {/*rendering-an-app-fully-built-with-react*/}

如果你的应用完全使用 React 构建，请为整个应用创建一个单独的 root。

```js [[1, 3, "document.getElementById('root')"], [2, 4, "<App />"]]
import { createRoot } from 'react-dom/client';

const root = createRoot(document.getElementById('root'));
root.render(<App />);
```

通常，你只需要在启动时运行一次这段代码。它会：

1. 找到你 HTML 中定义的 <CodeStep step={1}>浏览器 DOM 节点</CodeStep>。
2. 在其中显示应用的 <CodeStep step={2}>React 组件</CodeStep>。

<Sandpack>

```html public/index.html
<!DOCTYPE html>
<html>
  <head><title>我的应用</title></head>
  <body>
    <!-- 这是 DOM 节点 -->
    <div id="root"></div>
  </body>
</html>
```

```js src/index.js active
import { createRoot } from 'react-dom/client';
import App from './App.js';
import './styles.css';

const root = createRoot(document.getElementById('root'));
root.render(<App />);
```

```js src/App.js
import { useState } from 'react';

export default function App() {
  return (
    <>
      <h1>你好，世界！</h1>
      <Counter />
    </>
  );
}

function Counter() {
  const [count, setCount] = useState(0);
  return (
    <button onClick={() => setCount(count + 1)}>
      你已经点击我 {count} 次
    </button>
  );
}
```

</Sandpack>

**如果你的应用完全使用 React 构建，你通常不需要再创建更多 root，也不需要再次调用 [`root.render`](#root-render)。**

从现在起，React 将管理你整个应用的 DOM。要添加更多组件，请将它们[嵌套在 `App` 组件内部。](/learn/importing-and-exporting-components) 当你需要更新 UI 时，每个组件都可以通过[使用 state。](/reference/react/useState) 来完成。当你需要在 DOM 节点外显示诸如模态框或提示框之类的额外内容时，请[使用 portal 渲染。](/reference/react-dom/createPortal)

<Note>

当 HTML 为空时，用户会在应用的 JavaScript 代码加载并运行之前看到一个空白页面：

```html
<div id="root"></div>
```

这可能会让人感觉非常慢！要解决这个问题，你可以[在服务端或构建期间](/reference/react-dom/server)从组件生成初始 HTML。这样你的访客就能在任何 JavaScript 代码加载之前读取文本、查看图片并点击链接。我们建议[使用框架](/learn/creating-a-react-app#full-stack-frameworks)来开箱即用地完成这一优化。根据运行时机不同，这称为*服务端渲染（SSR）*或*静态站点生成（SSG）*。

</Note>

<Pitfall>

**使用服务端渲染或静态生成的应用必须调用 [`hydrateRoot`](/reference/react-dom/client/hydrateRoot) 而不是 `createRoot`。** 然后 React 会对 HTML 中已有的 DOM 节点进行 *hydrate*（复用），而不是销毁并重新创建它们。

</Pitfall>

---

### 渲染一个部分使用 React 构建的页面 {/*rendering-a-page-partially-built-with-react*/}

如果你的页面[不是完全使用 React 构建的](/learn/add-react-to-an-existing-project#using-react-for-a-part-of-your-existing-page)，你可以多次调用 `createRoot`，为由 React 管理的每个顶层 UI 部分创建一个 root。你可以通过调用 [`root.render`](#root-render) 在每个 root 中显示不同的内容。

这里，两个不同的 React 组件被渲染到 `index.html` 文件中定义的两个 DOM 节点里：

<Sandpack>

```html public/index.html
<!DOCTYPE html>
<html>
  <head><title>我的应用</title></head>
  <body>
    <nav id="navigation"></nav>
    <main>
      <p>这段段落不是由 React 渲染的（打开 index.html 即可验证）。</p>
      <section id="comments"></section>
    </main>
  </body>
</html>
```

```js src/index.js active
import './styles.css';
import { createRoot } from 'react-dom/client';
import { Comments, Navigation } from './Components.js';

const navDomNode = document.getElementById('navigation');
const navRoot = createRoot(navDomNode);
navRoot.render(<Navigation />);

const commentDomNode = document.getElementById('comments');
const commentRoot = createRoot(commentDomNode);
commentRoot.render(<Comments />);
```

```js src/Components.js
export function Navigation() {
  return (
    <ul>
      <NavLink href="/">主页</NavLink>
      <NavLink href="/about">关于</NavLink>
    </ul>
  );
}

function NavLink({ href, children }) {
  return (
    <li>
      <a href={href}>{children}</a>
    </li>
  );
}

export function Comments() {
  return (
    <>
      <h2>评论</h2>
      <Comment text="你好！" author="Sophie" />
      <Comment text="你好吗？" author="Sunil" />
    </>
  );
}

function Comment({ text, author }) {
  return (
    <p>{text} — <i>{author}</i></p>
  );
}
```

```css
nav ul { padding: 0; margin: 0; }
nav ul li { display: inline-block; margin-right: 20px; }
```

</Sandpack>

你也可以使用 [`document.createElement()`](https://developer.mozilla.org/en-US/docs/Web/API/Document/createElement) 创建一个新的 DOM 节点，然后手动将其添加到文档中。

```js
const domNode = document.createElement('div');
const root = createRoot(domNode);
root.render(<Comment />);
document.body.appendChild(domNode); // 你可以把它添加到文档中的任何位置
```

要将 React 树从 DOM 节点中移除并清理其使用的所有资源，请调用 [`root.unmount`.](#root-unmount)

```js
root.unmount();
```

如果你的 React 组件位于用其他框架编写的应用中，这种方式尤其有用。

---

### 更新 root 组件 {/*updating-a-root-component*/}

你可以在同一个 root 上多次调用 `render`。只要组件树结构与之前渲染的内容相匹配，React 就会[保留状态。](/learn/preserving-and-resetting-state) 注意你可以在输入框中输入内容，这意味着本例中每秒重复调用 `render` 所产生的更新并不会破坏现有内容：

<Sandpack>

```js src/index.js active
import { createRoot } from 'react-dom/client';
import './styles.css';
import App from './App.js';

const root = createRoot(document.getElementById('root'));

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

多次调用 `render` 并不常见。通常，你的组件会改为[更新 state。](/reference/react/useState)

### 生产环境中的错误日志记录 {/*error-logging-in-production*/}

默认情况下，React 会将所有错误记录到控制台。要实现你自己的错误上报，你可以提供可选的错误处理 root 选项 `onUncaughtError`、`onCaughtError` 和 `onRecoverableError`：

```js [[1, 6, "onCaughtError"], [2, 6, "error", 1], [3, 6, "errorInfo"], [4, 10, "componentStack", 15]]
import { createRoot } from "react-dom/client";
import { reportCaughtError } from "./reportError";

const container = document.getElementById("root");
const root = createRoot(container, {
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

<CodeStep step={1}>onCaughtError</CodeStep> 选项是一个接收两个参数的函数：

1. <CodeStep step={2}>error</CodeStep>，即被抛出的错误。
2. <CodeStep step={3}>errorInfo</CodeStep> 对象，其中包含该错误的 <CodeStep step={4}>componentStack</CodeStep>。

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
import { createRoot } from "react-dom/client";
import App from "./App.js";
import {
  onCaughtErrorProd,
  onRecoverableErrorProd,
  onUncaughtErrorProd,
} from "./reportError";

const container = document.getElementById("root");
const root = createRoot(container, {
  // 请记住在开发环境中移除这些选项，以便使用
  // React 的默认处理程序，或者为开发环境实现你自己的覆盖层。
  // 这些处理程序仅在此处无条件指定用于演示目的。
  onCaughtError: onCaughtErrorProd,
  onRecoverableError: onRecoverableErrorProd,
  onUncaughtError: onUncaughtErrorProd,
});
root.render(<App />);
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

</Sandpack>

## 故障排查 {/*troubleshooting*/}

### 我已经创建了一个 root，但没有显示任何内容 {/*ive-created-a-root-but-nothing-is-displayed*/}

请确保你没有忘记把你的应用实际 *渲染* 到 root 中：

```js {5}
import { createRoot } from 'react-dom/client';
import App from './App.js';

const root = createRoot(document.getElementById('root'));
root.render(<App />);
```

在你这样做之前，不会显示任何内容。

---

### 我遇到一个错误：“You passed a second argument to root.render” {/*im-getting-an-error-you-passed-a-second-argument-to-root-render*/}

一个常见错误是把 `createRoot` 的选项传给 `root.render(...)`：

<ConsoleBlock level="error">

Warning: 你传递了第二个参数给 root.render(...)，但它只接受一个参数。

</ConsoleBlock>

要修复这个问题，请把 root 选项传给 `createRoot(...)`，而不是 `root.render(...)`：
```js {2,5}
// 🚩 错误：root.render 只接受一个参数。
root.render(App, {onUncaughtError});

// ✅ 正确：将选项传给 createRoot。
const root = createRoot(container, {onUncaughtError});
root.render(<App />);
```

---

### 我遇到一个错误：“Target container is not a DOM element” {/*im-getting-an-error-target-container-is-not-a-dom-element*/}

这个错误意味着你传给 `createRoot` 的内容不是一个 DOM 节点。

如果你不确定发生了什么，试着把它打印出来：

```js {2}
const domNode = document.getElementById('root');
console.log(domNode); // ???
const root = createRoot(domNode);
root.render(<App />);
```

例如，如果 `domNode` 是 `null`，这意味着在你调用时，[`getElementById`](https://developer.mozilla.org/en-US/docs/Web/API/Document/getElementById) 返回了 `null`。如果在你调用时，文档中还没有带有给定 ID 的节点，就会发生这种情况。可能有以下几个原因：

1. 你要查找的 ID 可能与你在 HTML 文件中使用的 ID 不同。检查是否有拼写错误！
2. 你构建产物中的 `<script>` 标签无法“看到” HTML 中出现在它 *之后* 的任何 DOM 节点。

另一种常见的导致此错误的方式，是写成 `createRoot(<App />)`，而不是 `createRoot(domNode)`。

---

### 我遇到一个错误：“Functions are not valid as a React child.” {/*im-getting-an-error-functions-are-not-valid-as-a-react-child*/}

这个错误意味着你传给 `root.render` 的内容不是一个 React 组件。

如果你使用 `Component` 而不是 `<Component />` 调用 `root.render`，就可能发生这种情况：

```js {2,5}
// 🚩 错误：App 是一个函数，不是一个组件。
root.render(App);

// ✅ 正确：<App /> 是一个组件。
root.render(<App />);
```

或者，如果你传给 `root.render` 的是一个函数，而不是调用它后的结果：

```js {2,5}
// 🚩 错误：createApp 是一个函数，不是一个组件。
root.render(createApp);

// ✅ 正确：调用 createApp 以返回一个组件。
root.render(createApp());
```

---

### 我的服务端渲染 HTML 被从头重新创建了 {/*my-server-rendered-html-gets-re-created-from-scratch*/}

如果你的应用是服务端渲染的，并且包含 React 生成的初始 HTML，你可能会注意到，创建 root 并调用 `root.render` 会删除所有这些 HTML，然后从头重新创建所有 DOM 节点。这可能更慢，会重置焦点和滚动位置，并且可能丢失其他用户输入。

服务端渲染的应用必须使用 [`hydrateRoot`](/reference/react-dom/client/hydrateRoot) 而不是 `createRoot`：

```js {1,4-7}
import { hydrateRoot } from 'react-dom/client';
import App from './App.js';

hydrateRoot(
  document.getElementById('root'),
  <App />
);
```

请注意，它的 API 是不同的。尤其是，通常不会再有后续的 `root.render` 调用。
