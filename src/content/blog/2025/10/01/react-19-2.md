---
title: "React 19.2"
author: The React Team
date: 2025/10/01
description: React 19.2 添加了 Activity、React Performance Tracks、useEffectEvent 等新功能。
---

2025 年 10 月 1 日，来自 [The React Team](/community/team)

---

<Intro>

React 19.2 现已在 npm 上可用！

</Intro>

这是我们在过去一年中的第三个版本，前两个分别是 12 月的 React 19 和 6 月的 React 19.1。在这篇文章中，我们将概述 React 19.2 中的新功能，并重点介绍一些值得注意的变化。

<InlineToc />

---

## 新的 React 功能 {/*new-react-features*/}

### `<Activity />` {/*activity*/}

`<Activity>` 让你可以将应用拆分为可被控制和优先级调度的“活动”。

你可以将 Activity 作为有条件渲染应用部分内容的替代方案：

```js
// 之前
{isVisible && <Page />}

// 之后
<Activity mode={isVisible ? 'visible' : 'hidden'}>
  <Page />
</Activity>
```

在 React 19.2 中，Activity 支持两种模式：`visible` 和 `hidden`。

- `hidden`：隐藏子元素，卸载 effects，并推迟所有更新，直到 React 没有剩余工作可做。
- `visible`：显示子元素，挂载 effects，并允许更新正常处理。

这意味着你可以预渲染并持续渲染应用中不可见的部分，而不会影响屏幕上任何可见内容的性能。

你可以使用 Activity 来渲染用户接下来很可能会导航到的应用隐藏部分，或者保存用户离开页面部分的状态。这有助于通过在后台加载数据、css 和图片来加快导航速度，并允许返回导航保留输入框等状态。

未来，我们计划为不同的使用场景向 Activity 添加更多模式。

有关如何使用 Activity 的示例，请查看 [Activity 文档](/reference/react/Activity)。

---

### `useEffectEvent` {/*use-effect-event*/}

`useEffect` 的一个常见模式是通知应用代码来自外部系统的某种“事件”。例如，当聊天室连接成功时，你可能希望显示通知：

```js {5,11}
function ChatRoom({ roomId, theme }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.on('connected', () => {
      showNotification('已连接！', theme);
    });
    connection.connect();
    return () => {
      connection.disconnect()
    };
  }, [roomId, theme]);
  // ...
```

上面代码的问题在于，在此类“事件”内部使用的任何值发生变化，都会导致外层 Effect 重新运行。例如，修改 `theme` 会导致聊天室重新连接。对于与 Effect 逻辑本身相关的值，比如 `roomId`，这很合理，但对于 `theme` 来说并不合理。

为了解决这个问题，大多数用户只是禁用 lint 规则并排除该依赖项。但这可能会导致 bug，因为如果你之后需要更新 Effect，linter 将无法再帮助你保持依赖项最新。

使用 `useEffectEvent`，你可以将这段逻辑中的“事件”部分从触发它的 Effect 中拆分出来：

```js {2,3,4,9}
function ChatRoom({ roomId, theme }) {
  const onConnected = useEffectEvent(() => {
    showNotification('已连接！', theme);
  });

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.on('connected', () => {
      onConnected();
    });
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]); // ✅ 已声明所有依赖项（Effect Events 不是依赖项）
  // ...
```

类似于 DOM 事件，Effect Events 始终“看到”最新的 props 和 state。

**Effect Events 不应该 _被_ 声明在依赖数组中**。你需要升级到 `eslint-plugin-react-hooks@latest`，这样 linter 就不会尝试将它们插入为依赖项。请注意，Effect Events 只能在与“它们的” Effect 相同的组件或 Hook 中声明。这些限制会由 linter 进行校验。

<Note>

#### 何时使用 `useEffectEvent` {/*when-to-use-useeffectevent*/}

你应该将 `useEffectEvent` 用于那些概念上属于“事件”、但恰好由 Effect 触发的函数，而不是用户事件（这就是它成为“Effect Event”的原因）。你不需要把所有东西都包装进 `useEffectEvent`，也不应该仅仅为了消除 lint 错误而这样做，因为这可能会导致 bug。

关于如何理解 Event Effects 的深入介绍，请参见：[将事件与 Effects 分离](/learn/separating-events-from-effects#extracting-non-reactive-logic-out-of-effects)。

</Note>

---

### `cacheSignal` {/*cache-signal*/}

<RSC>

`cacheSignal` 仅用于 [React Server Components](/reference/rsc/server-components)。

</RSC>

`cacheSignal` 让你可以知道 [`cache()`](/reference/react/cache) 的生命周期何时结束：

```
import {cache, cacheSignal} from 'react';
const dedupedFetch = cache(fetch);

async function Component() {
  await dedupedFetch(url, { signal: cacheSignal() });
}
```

这使你可以在结果不再会被缓存使用时清理或中止工作，例如：

- React 已成功完成渲染
- 渲染已被中止
- 渲染已失败

更多信息请参阅 [`cacheSignal` 文档](/reference/react/cacheSignal)。

---

### 性能轨道 {/*performance-tracks*/}

React 19.2 为 Chrome DevTools 性能剖析添加了一组新的 [自定义轨道](https://developer.chrome.com/docs/devtools/performance/extension)，以提供更多关于 React 应用性能的信息：

<div style={{display: 'flex', justifyContent: 'center', marginBottom: '1rem'}}>
  <picture >
      <source srcset="/images/blog/react-labs-april-2025/perf_tracks.png" />
      <img className="w-full light-image" src="/images/blog/react-labs-april-2025/perf_tracks.webp" />
  </picture>
  <picture >
      <source srcset="/images/blog/react-labs-april-2025/perf_tracks_dark.png" />
      <img className="w-full dark-image" src="/images/blog/react-labs-april-2025/perf_tracks_dark.webp" />
  </picture>
</div>

[React Performance Tracks 文档](/reference/dev-tools/react-performance-tracks) 解释了轨道中包含的所有内容，但这里先给出一个高层概览。

#### 调度器 ⚛ {/*scheduler-*/}

Scheduler 轨道会显示 React 正在处理哪些不同优先级的工作，例如用于用户交互的“blocking”，或者用于 startTransition 内部更新的“transition”。在每个轨道中，你会看到正在执行的工作类型，例如触发更新的事件，以及该更新的渲染发生的时间。

我们还会显示一些信息，例如当某个更新因等待不同优先级而被阻塞，或者当 React 在继续之前等待绘制完成。Scheduler 轨道可以帮助你理解 React 如何将代码拆分为不同优先级，以及它完成工作的顺序。

请参阅 [Scheduler 轨道](/reference/dev-tools/react-performance-tracks#scheduler) 文档，查看包含的全部内容。

#### 组件 ⚛ {/*components-*/}

Components 轨道会显示 React 正在处理的组件树，无论是用于渲染还是运行 effects。在其中你会看到诸如“Mount”之类的标签，表示子元素挂载或 effects 挂载；或者“Blocked”，表示由于让位给 React 之外的工作，渲染被阻塞。

Components 轨道可以帮助你理解组件何时被渲染或运行 effects，以及完成这些工作所需的时间，从而帮助识别性能问题。

请参阅 [Components 轨道文档](/reference/dev-tools/react-performance-tracks#components)，查看包含的全部内容。

---

## React DOM 新功能 {/*new-react-dom-features*/}

### 部分预渲染 {/*partial-pre-rendering*/}

在 19.2 中，我们添加了一项新能力，可以提前预渲染应用的一部分，并在之后恢复渲染。

这项功能称为“部分预渲染（Partial Pre-rendering）”，它允许你预渲染应用的静态部分并通过 CDN 提供，然后稍后恢复渲染外壳，用动态内容将其填充完整。

要预渲染一个应用并在之后恢复，首先使用 `AbortController` 调用 `prerender`：

```
const {prelude, postponed} = await prerender(<App />, {
  signal: controller.signal,
});

// 保存 postponed 状态以供之后使用
await savePostponedState(postponed);

// 将 prelude 发送到客户端或 CDN。
```

然后，你可以将 `prelude` 外壳返回给客户端，之后再调用 `resume` 以“恢复”为一个 SSR 流：

```
const postponed = await getPostponedState(request);
const resumeStream = await resume(<App />, postponed);

// 将流发送到客户端。
```

或者你也可以调用 `resumeAndPrerender` 进行恢复，以获取用于 SSG 的静态 HTML：

```
const postponedState = await getPostponedState(request);
const { prelude } = await resumeAndPrerender(<App />, postponedState);

// 将完整的 HTML prelude 发送到 CDN。
```

更多信息请参阅这些新 API 的文档：
- `react-dom/server`
  - [`resume`](/reference/react-dom/server/resume)：用于 Web Streams。
  - [`resumeToPipeableStream`](/reference/react-dom/server/resumeToPipeableStream)：用于 Node Streams。
- `react-dom/static`
  - [`resumeAndPrerender`](/reference/react-dom/static/resumeAndPrerender)：用于 Web Streams。
  - [`resumeAndPrerenderToNodeStream`](/reference/react-dom/static/resumeAndPrerenderToNodeStream)：用于 Node Streams。

此外，prerender API 现在会返回一个 `postpone` 状态，以传递给 `resume` API。

---

## 值得注意的变化 {/*notable-changes*/}

### 为 SSR 批量处理 Suspense 边界 {/*batching-suspense-boundaries-for-ssr*/}

我们修复了一个行为 bug：Suspense 边界的显示方式会因其是在客户端渲染还是在服务端渲染流式输出中渲染而不同。

从 19.2 开始，React 会在一小段时间内批量显示服务端渲染的 Suspense 边界，以便让更多内容一起显示，并与客户端渲染的行为保持一致。

<Diagram name="19_2_batching_before" height={162} width={1270} alt="Diagram with three sections, with an arrow transitioning each section in between. The first section contains a page rectangle showing a glimmer loading state with faded bars. The second panel shows the top half of the page revealed and highlighted in blue. The third panel shows the entire the page revealed and highlighted in blue.">

此前，在流式服务端渲染期间，suspense 内容会立即替换回退内容。

</Diagram>

<Diagram name="19_2_batching_after" height={162} width={1270} alt="Diagram with three sections, with an arrow transitioning each section in between. The first section contains a page rectangle showing a glimmer loading state with faded bars. The second panel shows the same page. The third panel shows the entire the page revealed and highlighted in blue.">

在 React 19.2 中，suspense 边界会在一小段时间内批量处理，以便让更多内容一起显示。

</Diagram>

这个修复还为应用支持 SSR 期间的 `<ViewTransition>` 做好了准备。通过让更多内容一起显示，动画可以在更大的内容批次上运行，并避免对紧密到达的流式内容进行串联动画。

<Note>

React 使用启发式策略来确保限流不会影响核心 Web 指标和搜索排名。

例如，如果页面总加载时间接近 2.5 秒（这是 [LCP](https://web.dev/articles/lcp) 被认为“良好”的时间），React 将停止批量处理并立即显示内容，这样限流就不会成为错过该指标的原因。

</Note>

---

### SSR：Node 支持 Web Streams {/*ssr-web-streams-support-for-node*/}

React 19.2 为 Node.js 中的流式 SSR 添加了 Web Streams 支持：
- [`renderToReadableStream`](/reference/react-dom/server/renderToReadableStream) 现在可用于 Node.js
- [`prerender`](/reference/react-dom/static/prerender) 现在可用于 Node.js

以及新的 `resume` API：
- [`resume`](/reference/react-dom/server/resume) 可用于 Node.js。
- [`resumeAndPrerender`](/reference/react-dom/static/resumeAndPrerender) 可用于 Node.js。


<Pitfall>

#### 在 Node.js 中进行服务端渲染时，优先使用 Node Streams {/*prefer-node-streams-for-server-side-rendering-in-nodejs*/}

在 Node.js 环境中，我们仍然强烈建议使用 Node Streams API：

- [`renderToPipeableStream`](/reference/react-dom/server/renderToPipeableStream)
- [`resumeToPipeableStream`](/reference/react-dom/server/resumeToPipeableStream)
- [`prerenderToNodeStream`](/reference/react-dom/static/prerenderToNodeStream)
- [`resumeAndPrerenderToNodeStream`](/reference/react-dom/static/resumeAndPrerenderToNodeStream)

这是因为在 Node 中，Node Streams 比 Web Streams 快得多，而且 Web Streams 默认不支持压缩，这会导致用户不小心错失流式处理带来的好处。

</Pitfall>

---

### `eslint-plugin-react-hooks` v6 {/*eslint-plugin-react-hooks*/}

我们还发布了默认在 `recommended` 预设中使用 flat config 的 `eslint-plugin-react-hooks@latest`，并为新的、由 React Compiler 支持的规则提供了可选启用。

要继续使用旧版配置，你可以改用 `recommended-legacy`：

```diff
- extends: ['plugin:react-hooks/recommended']
+ extends: ['plugin:react-hooks/recommended-legacy']
```

如需查看启用编译器的完整规则列表，请[查看 linter 文档](/reference/eslint-plugin-react-hooks#recommended)。

查看 `eslint-plugin-react-hooks` 的[更新日志以了解完整变更列表](https://github.com/facebook/react/blob/main/packages/eslint-plugin-react-hooks/CHANGELOG.md#610)。

---

### 更新默认的 `useId` 前缀 {/*update-the-default-useid-prefix*/}

在 19.2 中，我们将默认的 `useId` 前缀从 `:r:`（19.0.0）或 `«r»`（19.1.0）更新为 `_r_`。

使用不适用于 CSS 选择器的特殊字符的初衷，是为了尽量减少与用户编写的 ID 冲突。然而，为了支持 View Transitions，我们需要确保 `useId` 生成的 ID 对 `view-transition-name` 和 XML 1.0 名称都是有效的。

## 更新日志 {/*changelog*/}

其他值得注意的变更
- `react-dom`：允许在可提升样式上使用 nonce [#32461](https://github.com/facebook/react/pull/32461)
- `react-dom`：当一个由 React 拥有的节点同时包含文本内容时，警告不要将其用作 Container [#32774](https://github.com/facebook/react/pull/32774)

值得注意的错误修复
- `react`：将 context 字符串化为 "SomeContext"，而不是 "SomeContext.Provider" [#33507](https://github.com/facebook/react/pull/33507)
- `react`：修复在 popstate 事件中出现的无限 useDeferredValue 循环 [#32821](https://github.com/facebook/react/pull/32821)
- `react`：修复将初始值传递给 useDeferredValue 时的一个 bug [#34376](https://github.com/facebook/react/pull/34376)
- `react`：修复在使用 Client Actions 提交表单时的崩溃 [#33055](https://github.com/facebook/react/pull/33055)
- `react`：如果已脱水的 suspense 边界重新挂起，则隐藏/取消隐藏其内容 [#32900](https://github.com/facebook/react/pull/32900)
- `react`：避免在 Hot Reload 期间对宽树发生栈溢出 [#34145](https://github.com/facebook/react/pull/34145)
- `react`：在多个位置改进组件栈信息 [#33629](https://github.com/facebook/react/pull/33629), [#33724](https://github.com/facebook/react/pull/33724), [#32735](https://github.com/facebook/react/pull/32735), [#33723](https://github.com/facebook/react/pull/33723)
- `react`：修复在 React.lazy-ed Component 中使用 React.use 时的一个 bug [#33941](https://github.com/facebook/react/pull/33941)
- `react-dom`：在使用 ARIA 1.3 属性时不再警告 [#34264](https://github.com/facebook/react/pull/34264)
- `react-dom`：修复 Suspense fallback 中嵌套过深的 Suspense 的一个 bug [#33467](https://github.com/facebook/react/pull/33467)
- `react-dom`：避免在渲染时中止后再次挂起时卡住 [#34192](https://github.com/facebook/react/pull/34192)

如需查看完整变更列表，请参阅 [更新日志](https://github.com/facebook/react/blob/main/CHANGELOG.md)。


---

_感谢 [Ricky Hanlon](https://bsky.app/profile/ricky.fm) [撰写本文](https://www.youtube.com/shorts/T9X3YkgZRG0)，以及 [Dan Abramov](https://bsky.app/profile/danabra.mov)、[Matt Carroll](https://twitter.com/mattcarrollcode)、[Jack Pope](https://jackpope.me) 和 [Joe Savona](https://x.com/en_JS) 对本文进行审阅。_
