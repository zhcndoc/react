---
title: "如何升级到 React 18"
author: Rick Hanlon
date: 2022/03/08
description: 正如我们在发布文章中所分享的，React 18 通过我们新的并发渲染器引入了诸多功能，并为现有应用提供了渐进式采用策略。在这篇文章中，我们将引导你完成升级到 React 18 的步骤。
---

2022 年 3 月 8 日，作者 [Rick Hanlon](https://twitter.com/rickhanlonii)

---

<Intro>

正如我们在 [发布文章](/blog/2022/03/29/react-v18) 中所分享的，React 18 通过我们新的并发渲染器引入了诸多功能，并为现有应用提供了渐进式采用策略。在这篇文章中，我们将引导你完成升级到 React 18 的步骤。

如果你在升级到 React 18 的过程中遇到任何问题，请 [报告任何问题](https://github.com/facebook/react/issues/new/choose)。

</Intro>

<Note>

对于 React Native 用户，React 18 将在未来版本的 React Native 中发布。这是因为 React 18 依赖于新的 React Native 架构，才能从本博客文章中介绍的新能力中受益。更多信息请参见 [React Conf 主题演讲](https://www.youtube.com/watch?v=FZ0cG47msEk&t=1530s)。

</Note>

---

## 安装 {/*installing*/}

要安装最新版本的 React：

```bash
npm install react react-dom
```

或者如果你使用的是 yarn：

```bash
yarn add react react-dom
```

## 客户端渲染 API 的更新 {/*updates-to-client-rendering-apis*/}

当你首次安装 React 18 时，你会在控制台中看到一条警告：

<ConsoleBlock level="error">

ReactDOM.render 在 React 18 中不再受支持。请改用 createRoot。在你切换到新 API 之前，你的应用将表现得像是在运行 React 17。了解更多：https://reactjs.org/link/switch-to-createroot

</ConsoleBlock>

React 18 引入了一个新的 root API，它为管理 root 提供了更好的易用性。新的 root API 还启用了新的并发渲染器，使你可以选择启用并发特性。

```js
// 之前
import { render } from 'react-dom';
const container = document.getElementById('app');
render(<App tab="home" />, container);

// 之后
import { createRoot } from 'react-dom/client';
const container = document.getElementById('app');
const root = createRoot(container); // 如果你使用 TypeScript，则为 createRoot(container!)
root.render(<App tab="home" />);
```

我们还将 `unmountComponentAtNode` 改为了 `root.unmount`：

```js
// 之前
unmountComponentAtNode(container);

// 之后
root.unmount();
```

我们还移除了 render 中的回调函数，因为在使用 Suspense 时它通常不会得到预期结果：

```js
// 之前
const container = document.getElementById('app');
render(<App tab="home" />, container, () => {
  console.log('rendered');
});

// 之后
function AppWithCallbackAfterRender() {
  useEffect(() => {
    console.log('rendered');
  });

  return <App tab="home" />
}

const container = document.getElementById('app');
const root = createRoot(container);
root.render(<AppWithCallbackAfterRender />);
```

<Note>

旧的 render 回调 API 没有一对一的替代方案——这取决于你的使用场景。更多信息请参见工作组文章 [将 render 替换为 createRoot](https://github.com/reactwg/react-18/discussions/5)。

</Note>

最后，如果你的应用使用带有 hydration 的服务端渲染，请将 `hydrate` 升级为 `hydrateRoot`：

```js
// 之前
import { hydrate } from 'react-dom';
const container = document.getElementById('app');
hydrate(<App tab="home" />, container);

// 之后
import { hydrateRoot } from 'react-dom/client';
const container = document.getElementById('app');
const root = hydrateRoot(container, <App tab="home" />);
// 与 createRoot 不同，这里你不需要单独调用 root.render()。
```

更多信息请参见 [工作组讨论](https://github.com/reactwg/react-18/discussions/5)。

<Note>

**如果你的应用在升级后无法运行，请检查它是否被 `<StrictMode>` 包裹。** [Strict Mode 在 React 18 中变得更严格了](#updates-to-strict-mode)，并不是所有组件都能承受它在开发模式下新增的检查。如果移除 Strict Mode 可以修复你的应用，你可以在升级期间先移除它，然后在修复它指出的问题后再把它加回去（可以加在最顶层，或者只包裹树的一部分）。

</Note>

## 服务端渲染 API 的更新 {/*updates-to-server-rendering-apis*/}

在这个版本中，我们正在重构 `react-dom/server` API，以便在服务端全面支持 Suspense 和流式 SSR。作为这些变更的一部分，我们将弃用旧的 Node 流式 API，因为它不支持在服务端进行增量式 Suspense 流式传输。

使用此 API 现在会发出警告：

* `renderToNodeStream`：**已弃用 ⛔️️**

相反，在 Node 环境中进行流式传输时，请使用：
* `renderToPipeableStream`：**新增 ✨**

我们还引入了一个新的 API，用于支持现代边缘运行时环境（例如 Deno 和 Cloudflare workers）中的带 Suspense 的流式 SSR：
* `renderToReadableStream`：**新增 ✨**

以下 API 仍将继续工作，但对 Suspense 的支持有限：
* `renderToString`：**有限** ⚠️
* `renderToStaticMarkup`：**有限** ⚠️

最后，这个 API 将继续用于渲染电子邮件：
* `renderToStaticNodeStream`

有关服务端渲染 API 变更的更多信息，请参见工作组文章 [在服务端升级到 React 18](https://github.com/reactwg/react-18/discussions/22)、[关于新的 Suspense SSR 架构的深入解析](https://github.com/reactwg/react-18/discussions/37)，以及 [Shaundai Person](https://twitter.com/shaundai) 在 React Conf 2021 上关于 [使用 Suspense 进行流式服务端渲染](https://www.youtube.com/watch?v=pj5N-Khihgc) 的演讲。

## TypeScript 定义的更新 {/*updates-to-typescript-definitions*/}

如果你的项目使用 TypeScript，你需要将 `@types/react` 和 `@types/react-dom` 依赖更新到最新版本。新的类型更安全，并能捕获过去会被类型检查器忽略的问题。最显著的变化是，在定义 props 时，`children` prop 现在需要显式列出，例如：

```typescript{3}
interface MyButtonProps {
  color: string;
  children?: React.ReactNode;
}
```

请参见 [React 18 typings 拉取请求](https://github.com/DefinitelyTyped/DefinitelyTyped/pull/56210)，了解完整的仅类型变更列表。它链接到了库类型中的示例修复，因此你可以看到如何调整你的代码。你可以使用 [自动迁移脚本](https://github.com/eps1lon/types-react-codemod) 更快地帮助将你的应用代码迁移到新的、更安全的类型定义。

如果你在这些类型定义中发现了 bug，请在 DefinitelyTyped 仓库中 [提交 issue](https://github.com/DefinitelyTyped/DefinitelyTyped/discussions/new?category=issues-with-a-types-package)。

## 自动批处理 {/*automatic-batching*/}

React 18 通过默认进行更多批处理，带来了开箱即用的性能提升。批处理是指 React 将多个状态更新合并为一次重新渲染，以获得更好的性能。在 React 18 之前，我们只会在 React 事件处理程序内部批处理更新。promise、setTimeout、原生事件处理程序或任何其他事件内部的更新默认都不会在 React 中批处理：

```js
// 仅在 React 18 之前，只有 React 事件会被批处理

function handleClick() {
  setCount(c => c + 1);
  setFlag(f => !f);
  // React 只会在最后重新渲染一次（这就是批处理！）
}

setTimeout(() => {
  setCount(c => c + 1);
  setFlag(f => !f);
  // React 将渲染两次，每次状态更新一次（没有批处理）
}, 1000);
```


从 React 18 开始，使用 `createRoot` 后，所有更新都会自动批处理，无论它们来自哪里。这意味着，timeout、promise、原生事件处理程序或任何其他事件中的更新，都会像 React 事件中的更新一样被批处理：

```js
// 在 React 18 之后，timeout、promise、
// 原生事件处理程序或任何其他事件中的更新都会被批处理。

function handleClick() {
  setCount(c => c + 1);
  setFlag(f => !f);
  // React 只会在最后重新渲染一次（这就是批处理！）
}

setTimeout(() => {
  setCount(c => c + 1);
  setFlag(f => !f);
  // React 只会在最后重新渲染一次（这就是批处理！）
}, 1000);
```

这是一项破坏性变更，但我们预计这会减少渲染工作量，因此提升应用性能。若要退出自动批处理，你可以使用 `flushSync`：

```js
import { flushSync } from 'react-dom';

function handleClick() {
  flushSync(() => {
    setCounter(c => c + 1);
  });
  // 到此时 React 已经更新了 DOM
  flushSync(() => {
    setFlag(f => !f);
  });
  // 到此时 React 已经更新了 DOM
}
```

更多信息请参见 [自动批处理深度解析](https://github.com/reactwg/react-18/discussions/21)。

## 面向库的新 API {/*new-apis-for-libraries*/}

在 React 18 工作组中，我们与库维护者合作，创建了支持并发渲染所需的新 API，以满足其在样式、外部存储等领域的特定使用场景。为了支持 React 18，一些库可能需要切换到以下 API 之一：

* `useSyncExternalStore` 是一个新的 Hook，它通过强制对 store 的更新同步执行，从而允许外部 store 支持并发读取。对于任何与 React 之外的状态集成的库，推荐使用这个新 API。更多信息请参见 [useSyncExternalStore 概览文章](https://github.com/reactwg/react-18/discussions/70) 和 [useSyncExternalStore API 详情](https://github.com/reactwg/react-18/discussions/86)。
* `useInsertionEffect` 是一个新的 Hook，它允许 CSS-in-JS 库解决在渲染期间注入样式所带来的性能问题。除非你已经构建了一个 CSS-in-JS 库，否则我们不预计你会用到它。这个 Hook 会在 DOM 被修改之后、布局 effect 读取新布局之前运行。这解决了 React 17 及以下版本中已经存在的问题，但在 React 18 中更为重要，因为 React 在并发渲染期间会让出执行权给浏览器，从而让浏览器有机会重新计算布局。更多信息请参见 [针对 `<style>` 的库升级指南](https://github.com/reactwg/react-18/discussions/110)。

React 18 还引入了用于并发渲染的新 API，例如 `startTransition`、`useDeferredValue` 和 `useId`，我们在 [发布文章](/blog/2022/03/29/react-v18) 中有更多介绍。

## 严格模式更新 {/*updates-to-strict-mode*/}

未来，我们希望添加一个特性，允许 React 在保留状态的同时添加和移除 UI 的部分区域。例如，当用户从某个屏幕切换出去又返回时，React 应该能够立即显示之前的屏幕。为此，React 会使用与之前相同的组件状态来卸载并重新挂载树。

这个特性会让 React 开箱即用地拥有更好的性能，但也要求组件能够承受 effect 被多次挂载和销毁。大多数 effect 无需任何更改即可工作，但有些 effect 会假定它们只会被挂载或销毁一次。

为了帮助暴露这些问题，React 18 在 Strict Mode 中引入了一个仅开发环境下的新检查。这个新检查会在组件首次挂载时自动卸载并重新挂载每个组件，并在第二次挂载时恢复之前的状态。

在此更改之前，React 会挂载组件并创建这些 effects：

```
* React 挂载组件。
    * 创建布局 effect。
    * 创建 Effect effects。
```

在 React 18 的 Strict Mode 下，React 会在开发模式中模拟卸载并重新挂载组件：

```
* React 挂载组件。
    * 创建布局 effect。
    * 创建 Effect effects。
* React 模拟卸载组件。
    * 销毁布局 effect。
    * 销毁 effects。
* React 模拟使用之前的状态挂载组件。
    * 布局 effect 的设置代码运行
    * Effect 的设置代码运行
```

有关更多信息，请参阅工作组帖子 [向 StrictMode 添加可复用状态](https://github.com/reactwg/react-18/discussions/19) 和 [如何在 effects 中支持可复用状态](https://github.com/reactwg/react-18/discussions/18)。

## 配置测试环境 {/*configuring-your-testing-environment*/}

当你第一次更新测试以使用 `createRoot` 时，可能会在测试控制台中看到此警告：

<ConsoleBlock level="error">

当前测试环境未配置以支持 act(...)

</ConsoleBlock>

要修复此问题，请在运行测试之前将 `globalThis.IS_REACT_ACT_ENVIRONMENT` 设置为 `true`：

```js
// 在你的测试设置文件中
globalThis.IS_REACT_ACT_ENVIRONMENT = true;
```

该标志的目的是告诉 React，它运行在类似单元测试的环境中。如果你忘记使用 `act` 包裹更新，React 会输出有帮助的警告。

你也可以将该标志设置为 `false`，告诉 React 不需要 `act`。这对于模拟完整浏览器环境的端到端测试很有用。

最终，我们预计测试库会自动为你配置这一点。例如，[下一版 React Testing Library 已内置对 React 18 的支持](https://github.com/testing-library/react-testing-library/issues/509#issuecomment-917989936)，无需任何额外配置。

有关 `act` 测试 API 及相关更改的[更多背景信息](https://github.com/reactwg/react-18/discussions/102)可在工作组中查看。

## 放弃对 Internet Explorer 的支持 {/*dropping-support-for-internet-explorer*/}

在此版本中，React 将放弃对 Internet Explorer 的支持，而 Internet Explorer [将于 2022 年 6 月 15 日停止支持](https://blogs.windows.com/windowsexperience/2021/05/19/the-future-of-internet-explorer-on-windows-10-is-in-microsoft-edge)。我们现在做出这一更改，是因为 React 18 中引入的新特性使用了现代浏览器特性，例如微任务，而这些特性无法在 IE 中得到充分的 polyfill 支持。

如果你需要支持 Internet Explorer，我们建议你继续使用 React 17。

## 弃用项 {/*deprecations*/}

* `react-dom`：`ReactDOM.render` 已被弃用。使用它会发出警告，并让你的应用以 React 17 模式运行。
* `react-dom`：`ReactDOM.hydrate` 已被弃用。使用它会发出警告，并让你的应用以 React 17 模式运行。
* `react-dom`：`ReactDOM.unmountComponentAtNode` 已被弃用。
* `react-dom`：`ReactDOM.renderSubtreeIntoContainer` 已被弃用。
* `react-dom/server`：`ReactDOMServer.renderToNodeStream` 已被弃用。

## 其他破坏性变更 {/*other-breaking-changes*/}

* **一致的 useEffect 时机**：如果更新是在离散用户输入事件期间触发的，例如点击或 keydown 事件，React 现在总是会同步刷新 effect 函数。以前，这种行为并不总是可预测或一致。
* **更严格的 hydration 错误**：由于缺失或多余文本内容导致的 hydration 不匹配，现在会被视为错误而不是警告。React 将不再尝试通过在客户端插入或删除节点来“修补”单个节点以匹配服务器标记，而是回退到树中最近的 `<Suspense>` 边界之前的客户端渲染。这确保了 hydration 后的树是一致的，并避免了 hydration 不匹配可能导致的隐私和安全漏洞。
* **Suspense 树始终保持一致：** 如果某个组件在完全添加到树中之前就挂起了，React 不会以不完整的状态将其添加到树中，也不会触发它的 effects。相反，React 会完全丢弃新树，等待异步操作完成，然后从头重新尝试渲染。React 会并发地渲染重试尝试，并且不会阻塞浏览器。
* **带有 Suspense 的布局 Effects**：当某棵树重新挂起并回退到 fallback 时，React 现在会清理布局 effects，然后在边界内的内容再次显示时重新创建它们。这修复了一个问题，该问题曾阻止组件库在与 Suspense 一起使用时正确测量布局。
* **新的 JS 环境要求**：React 现在依赖现代浏览器特性，包括 `Promise`、`Symbol` 和 `Object.assign`。如果你支持不原生提供现代浏览器特性或实现不兼容的旧浏览器和设备，例如 Internet Explorer，请考虑在打包后的应用中包含全局 polyfill。

## 其他值得注意的变更 {/*other-notable-changes*/}

### React {/*react*/}

* **组件现在可以渲染 `undefined`：** 如果你从组件返回 `undefined`，React 不再发出警告。这使得允许的组件返回值与组件树中间允许的值保持一致。我们建议使用 linter 来防止诸如在 JSX 之前忘记写 `return` 之类的错误。
* **在测试中，`act` 警告现在需要显式启用：** 如果你正在运行端到端测试，那么 `act` 警告是不必要的。我们引入了一个[显式启用](https://github.com/reactwg/react-18/discussions/102)机制，因此你可以只在它们有用且有益的单元测试中启用它们。
* **不再警告在未挂载组件上调用 `setState`：** 以前，当你在一个未挂载组件上调用 `setState` 时，React 会警告内存泄漏。这个警告是为订阅场景添加的，但人们主要是在设置状态其实没问题的场景中遇到它，而规避方案会让代码更糟。我们已经[移除了](https://github.com/facebook/react/pull/22114)这个警告。
* **不再抑制控制台日志：** 当你使用 Strict Mode 时，React 会将每个组件渲染两次，以帮助你发现意外的副作用。在 React 17 中，我们抑制了两次渲染中的一次控制台日志，以便日志更易于阅读。针对社区关于这会造成困惑的[反馈](https://github.com/facebook/react/issues/21783)，我们移除了这种抑制。相反，如果你安装了 React DevTools，第二次日志中的渲染将以灰色显示，并且会有一个选项（默认关闭）来完全抑制它们。
* **改进内存使用：** React 现在在卸载时会清理更多内部字段，从而减轻应用代码中可能存在且未修复的内存泄漏所带来的影响。

### React DOM Server {/*react-dom-server*/}

* **`renderToString`：** 在服务器上挂起时将不再报错。相反，它会为最近的 `<Suspense>` 边界输出 fallback HTML，然后在客户端重试渲染相同内容。仍然建议你改用流式 API，例如 `renderToPipeableStream` 或 `renderToReadableStream`。
* **`renderToStaticMarkup`：** 在服务器上挂起时将不再报错。相反，它会为最近的 `<Suspense>` 边界输出 fallback HTML。

## 更新日志 {/*changelog*/}

你可以在这里查看[完整更新日志](https://github.com/facebook/react/blob/main/CHANGELOG.md)。
