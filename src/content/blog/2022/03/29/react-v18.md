---
title: "React v18.0"
author: React 团队
date: 2022/03/08
description: React 18 现已可在 npm 上获取！在上一篇文章中，我们分享了将你的应用升级到 React 18 的逐步说明。在这篇文章中，我们将概述 React 18 的新特性，以及这对未来意味着什么。
---

2022 年 3 月 29 日，来自 [React 团队](/community/team)

---

<Intro>

React 18 现已可在 npm 上获取！在上一篇文章中，我们分享了将你的应用升级到 [React 18](/blog/2022/03/08/react-18-upgrade-guide) 的逐步说明。在这篇文章中，我们将概述 React 18 的新特性，以及这对未来意味着什么。

</Intro>

---

我们的最新主要版本包含了开箱即用的改进，例如自动批处理、新的 API（如 startTransition），以及支持 Suspense 的流式服务端渲染。

React 18 中的许多特性都建立在我们新的并发渲染器之上，这是一个幕后变更，解锁了强大的新能力。并发 React 需要显式启用——只有在你使用并发特性时才会开启——但我们认为它会对人们构建应用的方式产生很大影响。

我们已经花了数年时间研究并开发 React 中对并发的支持，并且非常谨慎地为现有用户提供了渐进式采用路径。去年夏天， [我们组建了 React 18 工作组](/blog/2021/06/08/the-plan-for-react-18)，以收集社区专家的反馈，并确保整个 React 生态系统都能顺利升级。

如果你错过了，我们在 React Conf 2021 上分享了这一愿景的很多内容：

* 在 [主题演讲](https://www.youtube.com/watch?v=FZ0cG47msEk&list=PLNG_1j3cPCaZZ7etkzWA7JfdmKWT0pMsa) 中，我们解释了 React 18 如何契合我们的使命：让开发者更容易构建出色的用户体验
* [Shruti Kapoor](https://twitter.com/shrutikapoor08) [演示了如何使用 React 18 中的新特性](https://www.youtube.com/watch?v=ytudH8je5ko&list=PLNG_1j3cPCaZZ7etkzWA7JfdmKWT0pMsa&index=2)
* [Shaundai Person](https://twitter.com/shaundai) 为我们概述了 [使用 Suspense 的流式服务端渲染](https://www.youtube.com/watch?v=pj5N-Khihgc&list=PLNG_1j3cPCaZZ7etkzWA7JfdmKWT0pMsa&index=3)

下面是本次发布的完整概览，从并发渲染开始。

<Note>

对于 React Native 用户，React 18 将随新的 React Native 架构一同在 React Native 中发布。更多信息请参见 [这里的 React Conf 主题演讲](https://www.youtube.com/watch?v=FZ0cG47msEk&t=1530s)。

</Note>

## 什么是并发 React？ {/*what-is-concurrent-react*/}

React 18 最重要的新增内容，是我们希望你永远不必去思考的东西：并发。我们认为这对应用开发者来说基本如此，不过对于库维护者而言，情况可能会稍微复杂一些。

并发本身并不是一个特性。它是一种新的幕后机制，使 React 能够同时准备 UI 的多个版本。你可以把并发看作一种实现细节——它之所以有价值，是因为它解锁了新的特性。React 在内部实现中使用了复杂的技术，比如优先级队列和多重缓冲。但你不会在我们的公共 API 中看到这些概念。

当我们设计 API 时，我们会尽量向开发者隐藏实现细节。作为 React 开发者，你关注的是你希望用户体验看起来怎样，而 React 负责处理如何交付这种体验。所以我们并不期望 React 开发者知道并发在底层是如何工作的。

不过，并发 React 比普通的实现细节更重要——它是对 React 核心渲染模型的基础性更新。因此，虽然了解并发如何工作并不是特别重要，但从高层次上理解它是什么可能还是值得的。

并发 React 的一个关键特性是渲染是可中断的。当你刚升级到 React 18 时，在添加任何并发特性之前，更新的渲染方式与之前版本的 React 相同——在一个单一、不中断、同步的事务中完成。使用同步渲染时，一旦某个更新开始渲染，就没有任何东西能打断它，直到用户能在屏幕上看到结果为止。

而在并发渲染中，情况并不总是如此。React 可能开始渲染一个更新，在中途暂停，然后稍后继续。它甚至可能直接放弃一个正在进行的渲染。React 保证即使渲染被中断，UI 也会保持一致。为此，它会等到最后再执行 DOM 变更，等整个树都评估完成之后再进行。借助这一能力，React 可以在后台准备新界面，而不会阻塞主线程。这意味着即使 UI 正在处理一个大型渲染任务，也能立即响应用户输入，从而创建流畅的用户体验。

另一个例子是可复用状态。并发 React 可以将 UI 的某些部分从屏幕上移除，然后稍后再重新添加，同时复用之前的状态。例如，当用户从某个界面切换离开又返回时，React 应该能够把先前的界面恢复到离开前的同一状态。在即将发布的一个小版本中，我们计划添加一个名为 `<Offscreen>` 的新组件来实现这种模式。同样，你也可以使用 Offscreen 在后台准备新的 UI，这样在用户展示它之前它就已经准备好了。

并发渲染是 React 中一个强大的新工具，我们的大多数新特性都围绕它构建，包括 Suspense、过渡和流式服务端渲染。但 React 18 只是我们希望在这一新基础上构建的一切的开始。

## 逐步采用并发特性 {/*gradually-adopting-concurrent-features*/}

从技术上讲，并发渲染是一个破坏性变更。由于并发渲染是可中断的，当它启用时，组件的行为会略有不同。

在我们的测试中，我们已经将成千上万个组件升级到了 React 18。我们发现，几乎所有现有组件在并发渲染下都能“直接工作”，无需任何更改。不过，其中一些可能需要额外的迁移工作。虽然这些变更通常很小，但你仍然可以按照自己的节奏进行。React 18 中的新渲染行为**只会在你的应用中使用新特性的部分启用。**

整体升级策略是在不破坏现有代码的前提下，让你的应用先在 React 18 上运行起来。然后你可以按照自己的节奏逐步开始添加并发特性。你可以使用 [`<StrictMode>`](/reference/react/StrictMode) 在开发过程中帮助暴露与并发相关的 bug。严格模式不会影响生产环境行为，但在开发时它会记录额外警告，并对预期幂等的函数执行两次调用。它不能捕获所有问题，但在防止最常见的错误类型方面非常有效。

在升级到 React 18 之后，你就可以立即开始使用并发特性。例如，你可以使用 startTransition 在屏幕之间导航，而不会阻塞用户输入。或者使用 useDeferredValue 来节流高成本的重新渲染。

不过，从长远来看，我们预计你为应用添加并发的主要方式，将是使用支持并发的库或框架。在大多数情况下，你不会直接与并发 API 交互。例如，与其让开发者在每次导航到新界面时都调用 startTransition，路由库会自动将导航包装在 startTransition 中。

库升级为并发兼容可能需要一些时间。我们已经提供了新的 API，让库更容易利用并发特性。在此期间，当我们逐步迁移 React 生态系统时，也请对维护者保持耐心。

更多信息请参见我们之前的文章：[如何升级到 React 18](/blog/2022/03/08/react-18-upgrade-guide)。

## 数据框架中的 Suspense {/*suspense-in-data-frameworks*/}

在 React 18 中，你可以开始在 Relay、Next.js、Hydrogen 或 Remix 等有主见的框架中，将 [Suspense](/reference/react/Suspense) 用于数据获取。临时性的 Suspense 数据获取在技术上是可行的，但作为通用策略仍不推荐。

未来，我们可能会暴露更多原语，使你更容易通过 Suspense 访问数据，也许甚至不需要使用有主见的框架。不过，当 Suspense 深度集成到你的应用架构中时，它的效果最好：包括你的路由器、数据层和服务端渲染环境。因此，即便从长远来看，我们也预计库和框架将在 React 生态系统中扮演关键角色。

与之前版本的 React 一样，你也可以在客户端使用 React.lazy 配合 Suspense 进行代码拆分。但我们对 Suspense 的愿景从来都不只是加载代码——目标是扩展对 Suspense 的支持，使最终同一个声明式的 Suspense fallback 能处理任何异步操作（加载代码、数据、图片等）。

## Server Components 仍在开发中 {/*server-components-is-still-in-development*/}

[**Server Components**](/blog/2020/12/21/data-fetching-with-react-server-components) 是一个即将推出的特性，它允许开发者构建跨越服务端与客户端的应用，将客户端应用丰富的交互性与传统服务端渲染的性能提升结合起来。Server Components 本身并不与并发 React 绑定，但它被设计为能与 Suspense 和流式服务端渲染等并发特性更好地协同工作。

Server Components 仍处于实验阶段，但我们预计会在一个 18.x 的小版本中发布初始版本。与此同时，我们正在与 Next.js、Hydrogen 和 Remix 等框架合作，以推进该提案并使其适合广泛采用。

## React 18 中的新内容 {/*whats-new-in-react-18*/}

### 新特性：自动批处理 {/*new-feature-automatic-batching*/}

批处理是指 React 将多个状态更新合并为一次重新渲染，以获得更好的性能。没有自动批处理时，我们只会在 React 事件处理函数内部对更新进行批处理。默认情况下，Promise、setTimeout、原生事件处理函数或其他任何事件中的更新都不会在 React 中被批处理。使用自动批处理后，这些更新将会自动被批处理：


```js
// 之前：只有 React 事件会被批处理。
setTimeout(() => {
  setCount(c => c + 1);
  setFlag(f => !f);
  // React 会渲染两次，每次状态更新一次（没有批处理）
}, 1000);

// 之后：timeout、Promise、
// 原生事件处理函数或其他任何事件中的更新都会被批处理。
setTimeout(() => {
  setCount(c => c + 1);
  setFlag(f => !f);
  // React 最终只会在结束时重新渲染一次（这就是批处理！）
}, 1000);
```

更多信息请参见这篇文章：[React 18 中更少渲染的自动批处理](https://github.com/reactwg/react-18/discussions/21)。

### 新特性：过渡 {/*new-feature-transitions*/}

过渡是 React 中一个新的概念，用于区分紧急更新和非紧急更新。

* **紧急更新** 反映直接交互，例如输入、点击、按键等。
* **过渡更新** 将 UI 从一个视图过渡到另一个视图。

像输入、点击或按键这样的紧急更新，需要立即响应，以符合我们对物理对象行为的直觉。否则它们会让人感觉“有问题”。不过，过渡不同，因为用户并不期待在屏幕上看到每一个中间值。

例如，当你在下拉菜单中选择一个筛选器时，你会期望筛选按钮本身在点击后立即响应。不过，实际结果可以单独过渡显示。短暂的延迟是难以察觉的，而且往往也是符合预期的。如果你在结果还没渲染完成前又更改了筛选器，你只关心看到最新结果。

通常，为了获得最佳用户体验，一次用户输入应该同时产生一个紧急更新和一个非紧急更新。你可以在输入事件中使用 startTransition API 来告知 React 哪些更新是紧急的，哪些是“过渡”：


```js
import { startTransition } from 'react';

// 紧急：显示输入的内容
setInputValue(input);

// 将其中的任何状态更新标记为过渡
startTransition(() => {
  // 过渡：显示结果
  setSearchQuery(input);
});
```


被 startTransition 包裹的更新会被当作非紧急更新处理，如果有更紧急的更新（例如点击或按键）进入，就会被中断。如果一个过渡被用户中断（例如连续输入多个字符），React 会放弃尚未完成的过时渲染工作，只渲染最新的更新。


* `useTransition`：用于启动过渡的 Hook，包括一个用于跟踪 pending 状态的值。
* `startTransition`：当不能使用 Hook 时，用于启动过渡的方法。

过渡会启用并发渲染，从而允许更新被中断。如果内容再次挂起，过渡还会告诉 React 在后台渲染过渡内容时继续显示当前内容（更多信息请参见 [Suspense RFC](https://github.com/reactjs/rfcs/blob/main/text/0213-suspense-in-react-18.md)）。

[在这里查看过渡文档](/reference/react/useTransition)。

### 新的 Suspense 特性 {/*new-suspense-features*/}

Suspense 允许你以声明式方式指定组件树某一部分尚未准备好显示时的加载状态：

```js
<Suspense fallback={<Spinner />}>
  <Comments />
</Suspense>
```

Suspense 将“UI 加载状态”作为 React 编程模型中的一等声明式概念。这使我们能够在其基础上构建更高级的特性。

我们几年前引入了一个有限版本的 Suspense。不过，当时唯一受支持的用例是使用 React.lazy 进行代码拆分，而且在服务端渲染时完全不受支持。

在 React 18 中，我们增加了对服务端 Suspense 的支持，并通过并发渲染特性扩展了它的能力。

React 18 中的 Suspense 最适合与 transition API 结合使用。如果你在过渡期间挂起，React 会阻止已经可见的内容被 fallback 替换。相反，React 会延迟渲染，直到加载了足够的数据，以避免出现糟糕的加载状态。

更多内容请参见 [React 18 中 Suspense 的 RFC](https://github.com/reactjs/rfcs/blob/main/text/0213-suspense-in-react-18.md)。

### 新的客户端和服务端渲染 API {/*new-client-and-server-rendering-apis*/}

在本次发布中，我们借此机会重新设计了用于客户端和服务端渲染的 API。这些变化允许用户在升级到 React 18 的新 API 之前，继续在 React 17 模式下使用旧 API。

#### React DOM Client {/*react-dom-client*/}

这些新的 API 现已从 `react-dom/client` 导出：

* `createRoot`：用于创建一个 root 以 `render` 或 `unmount` 的新方法。请用它代替 `ReactDOM.render`。没有它，React 18 中的新特性无法工作。
* `hydrateRoot`：用于水合服务端渲染应用的新方法。请将它与新的 React DOM Server API 配合使用，代替 `ReactDOM.hydrate`。没有它，React 18 中的新特性无法工作。

`createRoot` 和 `hydrateRoot` 都接受一个名为 `onRecoverableError` 的新选项，以便在 React 从渲染或水合过程中的错误中恢复时通知你进行日志记录。默认情况下，React 会使用 [`reportError`](https://developer.mozilla.org/en-US/docs/Web/API/reportError)，在较旧的浏览器中则使用 `console.error`。

[在这里查看 React DOM Client 文档](/reference/react-dom/client)。

#### React DOM Server {/*react-dom-server*/}

这些新的 API 现已从 `react-dom/server` 导出，并且完整支持在服务端流式处理 Suspense：

* `renderToPipeableStream`：用于在 Node 环境中进行流式渲染。
* `renderToReadableStream`：用于现代边缘运行时环境，例如 Deno 和 Cloudflare workers。

现有的 `renderToString` 方法仍可继续使用，但不建议使用。

[在这里查看 React DOM Server 文档](/reference/react-dom/server)。

### 新的 Strict Mode 行为 {/*new-strict-mode-behaviors*/}

未来，我们希望添加一个特性，让 React 在保留状态的同时添加和移除 UI 的某些部分。例如，当用户从某个界面切换离开又返回时，React 应该能够立即显示先前的界面。为此，React 会使用与之前相同的组件状态来卸载并重新挂载树。

这个特性会让 React 应用开箱即有更好的性能，但要求组件能够对多次挂载和销毁 effects 保持健壮。大多数 effects 无需任何改动就能工作，但有些 effects 假设它们只会被挂载或销毁一次。

为了帮助暴露这些问题，React 18 在严格模式中引入了一个仅开发环境下的新检查。每当某个组件首次挂载时，这个新检查会自动卸载并重新挂载每个组件，在第二次挂载时恢复之前的状态。

在此变更之前，React 会挂载组件并创建 effects：

```
* React 挂载组件。
  * 创建布局 effects。
  * 创建 effects。
```


在 React 18 的严格模式中，React 会在开发模式下模拟卸载并重新挂载组件：

```
* React 挂载组件。
  * 创建布局 effects。
  * 创建 effects。
* React 模拟卸载组件。
  * 销毁布局 effects。
  * 销毁 effects。
* React 模拟使用之前的状态挂载组件。
  * 创建布局 effects。
  * 创建 effects。
```

[在这里查看确保可复用状态的文档](/reference/react/StrictMode#fixing-bugs-found-by-re-running-effects-in-development)。

### 新的 Hooks {/*new-hooks*/}

#### useId {/*useid*/}

`useId` 是一个用于在客户端和服务端生成唯一 ID 的新 Hook，同时避免水合不匹配。它主要适用于集成了可访问性 API、且这些 API 需要唯一 ID 的组件库。这解决了 React 17 及更早版本中已经存在的问题，但由于新的流式服务端渲染器会乱序输出 HTML，这在 React 18 中变得更加重要。[在这里查看文档](/reference/react/useId)。

> 注意
>
> `useId` **不是** 用来生成 [列表中的 key](/learn/rendering-lists#where-to-get-your-key) 的。key 应该从你的数据中生成。

#### useTransition {/*usetransition*/}

`useTransition` 和 `startTransition` 允许你将某些状态更新标记为非紧急。其他状态更新默认被视为紧急。React 会允许紧急状态更新（例如更新文本输入框）中断非紧急状态更新（例如渲染搜索结果列表）。[在这里查看文档](/reference/react/useTransition)。

#### useDeferredValue {/*usedeferredvalue*/}

`useDeferredValue` 允许你延迟重新渲染树中非紧急的部分。它类似于防抖，但与之相比有一些优势。它没有固定的时间延迟，因此 React 会在第一次渲染已经反映到屏幕上后立即尝试延迟渲染。延迟渲染是可中断的，并且不会阻塞用户输入。[在这里查看文档](/reference/react/useDeferredValue)。

#### useSyncExternalStore {/*usesyncexternalstore*/}

`useSyncExternalStore` 是一个新的 Hook，它通过强制对 store 的更新为同步，允许外部 store 支持并发读取。它在实现对外部数据源的订阅时，不再需要 useEffect，并且推荐任何与 React 之外状态集成的库使用。[在这里查看文档](/reference/react/useSyncExternalStore)。

> 注意
>
> `useSyncExternalStore`  предназначен用于库，而不是应用代码。

#### useInsertionEffect {/*useinsertioneffect*/}

`useInsertionEffect` 是一个新的 Hook，允许 CSS-in-JS 库解决在渲染过程中注入样式所带来的性能问题。除非你已经构建了一个 CSS-in-JS 库，否则我们不认为你会用到它。这个 Hook 会在 DOM 变更之后执行，但在布局 effects 读取新的布局之前执行。这解决了 React 17 及更早版本中已经存在的问题，但在 React 18 中更为重要，因为 React 在并发渲染期间会让出给浏览器，从而给浏览器机会重新计算布局。[在这里查看文档](/reference/react/useInsertionEffect)。

> 注意
>
> `useInsertionEffect`  предназначен用于库，而不是应用代码。

## 如何升级 {/*how-to-upgrade*/}

有关逐步说明和完整的破坏性及显著变更列表，请参见 [如何升级到 React 18](/blog/2022/03/08/react-18-upgrade-guide)。

## 更新日志 {/*changelog*/}

### React {/*react*/}

* 添加 `useTransition` 和 `useDeferredValue`，以将紧急更新与过渡分离。([#10426](https://github.com/facebook/react/pull/10426), [#10715](https://github.com/facebook/react/pull/10715), [#15593](https://github.com/facebook/react/pull/15593), [#15272](https://github.com/facebook/react/pull/15272), [#15578](https://github.com/facebook/react/pull/15578), [#15769](https://github.com/facebook/react/pull/15769), [#17058](https://github.com/facebook/react/pull/17058), [#18796](https://github.com/facebook/react/pull/18796), [#19121](https://github.com/facebook/react/pull/19121), [#19703](https://github.com/facebook/react/pull/19703), [#19719](https://github.com/facebook/react/pull/19719), [#19724](https://github.com/facebook/react/pull/19724), [#20672](https://github.com/facebook/react/pull/20672), [#20976](https://github.com/facebook/react/pull/20976) 由 [@acdlite](https://github.com/acdlite), [@lunaruan](https://github.com/lunaruan), [@rickhanlonii](https://github.com/rickhanlonii) 和 [@sebmarkbage](https://github.com/sebmarkbage) 提交）
* 添加 `useId` 用于生成唯一 ID。([#17322](https://github.com/facebook/react/pull/17322), [#18576](https://github.com/facebook/react/pull/18576), [#22644](https://github.com/facebook/react/pull/22644), [#22672](https://github.com/facebook/react/pull/22672), [#21260](https://github.com/facebook/react/pull/21260) 由 [@acdlite](https://github.com/acdlite), [@lunaruan](https://github.com/lunaruan) 和 [@sebmarkbage](https://github.com/sebmarkbage) 提交）
* 添加 `useSyncExternalStore` 以帮助外部存储库与 React 集成。([#15022](https://github.com/facebook/react/pull/15022), [#18000](https://github.com/facebook/react/pull/18000), [#18771](https://github.com/facebook/react/pull/18771), [#22211](https://github.com/facebook/react/pull/22211), [#22292](https://github.com/facebook/react/pull/22292), [#22239](https://github.com/facebook/react/pull/22239), [#22347](https://github.com/facebook/react/pull/22347), [#23150](https://github.com/facebook/react/pull/23150) 由 [@acdlite](https://github.com/acdlite), [@bvaughn](https://github.com/bvaughn) 和 [@drarmstr](https://github.com/drarmstr) 提交）
* 添加 `startTransition`，作为不带待处理反馈的 `useTransition` 版本。([#19696](https://github.com/facebook/react/pull/19696) 由 [@rickhanlonii](https://github.com/rickhanlonii) 提交）
* 为 CSS-in-JS 库添加 `useInsertionEffect`。([#21913](https://github.com/facebook/react/pull/21913) 由 [@rickhanlonii](https://github.com/rickhanlonii) 提交）
* 当内容重新出现时，让 Suspense 重新挂载布局效果。([#19322](https://github.com/facebook/react/pull/19322), [#19374](https://github.com/facebook/react/pull/19374), [#19523](https://github.com/facebook/react/pull/19523), [#20625](https://github.com/facebook/react/pull/20625), [#21079](https://github.com/facebook/react/pull/21079) 由 [@acdlite](https://github.com/acdlite), [@bvaughn](https://github.com/bvaughn) 和 [@lunaruan](https://github.com/lunaruan) 提交）
* 让 `<StrictMode>` 重新运行效果以检查可恢复状态。([#19523](https://github.com/facebook/react/pull/19523) , [#21418](https://github.com/facebook/react/pull/21418) 由 [@bvaughn](https://github.com/bvaughn) 和 [@lunaruan](https://github.com/lunaruan) 提交）
* 假定 Symbols 始终可用。([#23348](https://github.com/facebook/react/pull/23348) 由 [@sebmarkbage](https://github.com/sebmarkbage) 提交）
* 移除 `object-assign` polyfill。([#23351](https://github.com/facebook/react/pull/23351) 由 [@sebmarkbage](https://github.com/sebmarkbage) 提交）
* 移除不受支持的 `unstable_changedBits` API。([#20953](https://github.com/facebook/react/pull/20953) 由 [@acdlite](https://github.com/acdlite) 提交）
* 允许组件渲染 undefined。([#21869](https://github.com/facebook/react/pull/21869) 由 [@rickhanlonii](https://github.com/rickhanlonii) 提交）
* 将由点击等离散事件产生的 `useEffect` 同步刷新。([#21150](https://github.com/facebook/react/pull/21150) 由 [@acdlite](https://github.com/acdlite) 提交）
* 现在 Suspense 中的 `fallback={undefined}` 行为与 `null` 相同，并且不会被忽略。([#21854](https://github.com/facebook/react/pull/21854) 由 [@rickhanlonii](https://github.com/rickhanlonii) 提交）
* 将所有解析为同一组件的 `lazy()` 视为等价。([#20357](https://github.com/facebook/react/pull/20357) 由 [@sebmarkbage](https://github.com/sebmarkbage) 提交）
* 首次渲染时不要修补 console。([#22308](https://github.com/facebook/react/pull/22308) 由 [@lunaruan](https://github.com/lunaruan) 提交）
* 改进内存使用。([#21039](https://github.com/facebook/react/pull/21039) 由 [@bgirard](https://github.com/bgirard) 提交）
* 改进字符串强制转换抛出异常时的消息（Temporal.*、Symbol 等）。([#22064](https://github.com/facebook/react/pull/22064) 由 [@justingrant](https://github.com/justingrant) 提交）
* 在可用时优先使用 `setImmediate` 而不是 `MessageChannel`。([#20834](https://github.com/facebook/react/pull/20834) 由 [@gaearon](https://github.com/gaearon) 提交）
* 修复上下文无法在挂起树内传播的问题。([#23095](https://github.com/facebook/react/pull/23095) 由 [@gaearon](https://github.com/gaearon) 提交）
* 通过移除 eager bailout 机制修复 `useReducer` 观察到错误 props 的问题。([#22445](https://github.com/facebook/react/pull/22445) 由 [@josephsavona](https://github.com/josephsavona) 提交）
* 修复在 Safari 中追加 iframe 时 `setState` 被忽略的问题。([#23111](https://github.com/facebook/react/pull/23111) 由 [@gaearon](https://github.com/gaearon) 提交）
* 修复在树中渲染 `ZonedDateTime` 时的崩溃。([#20617](https://github.com/facebook/react/pull/20617) 由 [@dimaqq](https://github.com/dimaqq) 提交）
* 修复测试中将 document 设为 `null` 时的崩溃。([#22695](https://github.com/facebook/react/pull/22695) 由 [@SimenB](https://github.com/SimenB) 提交）
* 修复在启用并发特性时 `onLoad` 不触发的问题。([#23316](https://github.com/facebook/react/pull/23316) 由 [@gnoff](https://github.com/gnoff) 提交）
* 修复选择器返回 `NaN` 时的警告。([#23333](https://github.com/facebook/react/pull/23333) 由 [@hachibeeDI](https://github.com/hachibeeDI) 提交）
* 修复测试中将 document 设为 `null` 时的崩溃。([#22695](https://github.com/facebook/react/pull/22695) 由 [@SimenB](https://github.com/SimenB) 提交）
* 修复生成的许可证头。([#23004](https://github.com/facebook/react/pull/23004) 由 [@vitaliemiron](https://github.com/vitaliemiron) 提交）
* 将 `package.json` 添加为入口点之一。([#22954](https://github.com/facebook/react/pull/22954) 由 [@Jack](https://github.com/Jack-Works) 提交）
* 允许在 Suspense 边界之外挂起。([#23267](https://github.com/facebook/react/pull/23267) 由 [@acdlite](https://github.com/acdlite) 提交）
* 每当 hydration 失败时记录一个可恢复错误。([#23319](https://github.com/facebook/react/pull/23319) 由 [@acdlite](https://github.com/acdlite) 提交）

### React DOM {/*react-dom*/}

* 添加 `createRoot` 和 `hydrateRoot`。([#10239](https://github.com/facebook/react/pull/10239), [#11225](https://github.com/facebook/react/pull/11225), [#12117](https://github.com/facebook/react/pull/12117), [#13732](https://github.com/facebook/react/pull/13732), [#15502](https://github.com/facebook/react/pull/15502), [#15532](https://github.com/facebook/react/pull/15532), [#17035](https://github.com/facebook/react/pull/17035), [#17165](https://github.com/facebook/react/pull/17165), [#20669](https://github.com/facebook/react/pull/20669), [#20748](https://github.com/facebook/react/pull/20748), [#20888](https://github.com/facebook/react/pull/20888), [#21072](https://github.com/facebook/react/pull/21072), [#21417](https://github.com/facebook/react/pull/21417), [#21652](https://github.com/facebook/react/pull/21652), [#21687](https://github.com/facebook/react/pull/21687), [#23207](https://github.com/facebook/react/pull/23207), [#23385](https://github.com/facebook/react/pull/23385) 由 [@acdlite](https://github.com/acdlite), [@bvaughn](https://github.com/bvaughn), [@gaearon](https://github.com/gaearon), [@lunaruan](https://github.com/lunaruan), [@rickhanlonii](https://github.com/rickhanlonii), [@trueadm](https://github.com/trueadm) 和 [@sebmarkbage](https://github.com/sebmarkbage) 提交）
* 添加选择性 hydration。([#14717](https://github.com/facebook/react/pull/14717), [#14884](https://github.com/facebook/react/pull/14884), [#16725](https://github.com/facebook/react/pull/16725), [#16880](https://github.com/facebook/react/pull/16880), [#17004](https://github.com/facebook/react/pull/17004), [#22416](https://github.com/facebook/react/pull/22416), [#22629](https://github.com/facebook/react/pull/22629), [#22448](https://github.com/facebook/react/pull/22448), [#22856](https://github.com/facebook/react/pull/22856), [#23176](https://github.com/facebook/react/pull/23176) 由 [@acdlite](https://github.com/acdlite), [@gaearon](https://github.com/gaearon), [@salazarm](https://github.com/salazarm) 和 [@sebmarkbage](https://github.com/sebmarkbage) 提交）
* 将 `aria-description` 添加到已知 ARIA 属性列表中。([#22142](https://github.com/facebook/react/pull/22142) 由 [@mahyareb](https://github.com/mahyareb) 提交）
* 为 video 元素添加 `onResize` 事件。([#21973](https://github.com/facebook/react/pull/21973) 由 [@rileyjshaw](https://github.com/rileyjshaw) 提交）
* 将 `imageSizes` 和 `imageSrcSet` 添加到已知 props。([#22550](https://github.com/facebook/react/pull/22550) 由 [@eps1lon](https://github.com/eps1lon) 提交）
* 如果提供了 `value`，则允许非字符串 `<option>` 子元素。([#21431](https://github.com/facebook/react/pull/21431) 由 [@sebmarkbage](https://github.com/sebmarkbage) 提交）
* 修复 `aspectRatio` 样式未被应用的问题。([#21100](https://github.com/facebook/react/pull/21100) 由 [@gaearon](https://github.com/gaearon) 提交）
* 当调用 `renderSubtreeIntoContainer` 时发出警告。([#23355](https://github.com/facebook/react/pull/23355) 由 [@acdlite](https://github.com/acdlite) 提交）

### React DOM Server {/*react-dom-server-1*/}

* 添加新的流式渲染器。([#14144](https://github.com/facebook/react/pull/14144), [#20970](https://github.com/facebook/react/pull/20970), [#21056](https://github.com/facebook/react/pull/21056), [#21255](https://github.com/facebook/react/pull/21255), [#21200](https://github.com/facebook/react/pull/21200), [#21257](https://github.com/facebook/react/pull/21257), [#21276](https://github.com/facebook/react/pull/21276), [#22443](https://github.com/facebook/react/pull/22443), [#22450](https://github.com/facebook/react/pull/22450), [#23247](https://github.com/facebook/react/pull/23247), [#24025](https://github.com/facebook/react/pull/24025), [#24030](https://github.com/facebook/react/pull/24030) 由 [@sebmarkbage](https://github.com/sebmarkbage) 提交）
* 修复在处理多个请求时 SSR 中的上下文提供器。([#23171](https://github.com/facebook/react/pull/23171) 由 [@frandiox](https://github.com/frandiox) 提交）
* 文本不匹配时回退到客户端渲染。([#23354](https://github.com/facebook/react/pull/23354) 由 [@acdlite](https://github.com/acdlite) 提交）
* 弃用 `renderToNodeStream`。([#23359](https://github.com/facebook/react/pull/23359) 由 [@sebmarkbage](https://github.com/sebmarkbage) 提交）
* 修复新的服务端渲染器中的一个误报错误日志。([#24043](https://github.com/facebook/react/pull/24043) 由 [@eps1lon](https://github.com/eps1lon) 提交）
* 修复新的服务端渲染器中的一个 bug。([#22617](https://github.com/facebook/react/pull/22617) 由 [@shuding](https://github.com/shuding) 提交）
* 在服务器端忽略自定义元素中的函数和值符号值。([#21157](https://github.com/facebook/react/pull/21157) 由 [@sebmarkbage](https://github.com/sebmarkbage) 提交）

### React DOM Test Utils {/*react-dom-test-utils*/}

* 当在生产环境中使用 `act` 时抛出错误。([#21686](https://github.com/facebook/react/pull/21686) 由 [@acdlite](https://github.com/acdlite) 提交）
* 支持使用 `global.IS_REACT_ACT_ENVIRONMENT` 禁用多余的 act 警告。([#22561](https://github.com/facebook/react/pull/22561) 由 [@acdlite](https://github.com/acdlite) 提交）
* 扩展 act 警告以涵盖所有可能调度 React 工作的 API。([#22607](https://github.com/facebook/react/pull/22607) 由 [@acdlite](https://github.com/acdlite) 提交）
* 让 `act` 批量更新。([#21797](https://github.com/facebook/react/pull/21797) 由 [@acdlite](https://github.com/acdlite) 提交）
* 移除对悬挂的被动效果的警告。([#22609](https://github.com/facebook/react/pull/22609) 由 [@acdlite](https://github.com/acdlite) 提交）

### React Refresh {/*react-refresh*/}

* 在 Fast Refresh 中跟踪后挂载的根。([#22740](https://github.com/facebook/react/pull/22740) 由 [@anc95](https://github.com/anc95) 提交）
* 向 `package.json` 添加 `exports` 字段。([#23087](https://github.com/facebook/react/pull/23087) 由 [@otakustay](https://github.com/otakustay) 提交）

### Server Components (Experimental) {/*server-components-experimental*/}

* 添加 Server Context 支持。([#23244](https://github.com/facebook/react/pull/23244) 由 [@salazarm](https://github.com/salazarm) 提交）
* 添加 `lazy` 支持。([#24068](https://github.com/facebook/react/pull/24068) 由 [@gnoff](https://github.com/gnoff) 提交）
* 为 webpack 5 更新 webpack 插件。([#22739](https://github.com/facebook/react/pull/22739) 由 [@michenly](https://github.com/michenly) 提交）
* 修复 Node 加载器中的一个错误。([#22537](https://github.com/facebook/react/pull/22537) 由 [@btea](https://github.com/btea) 提交）
* 在边缘环境中使用 `globalThis` 而不是 `window`。([#22777](https://github.com/facebook/react/pull/22777) 由 [@huozhi](https://github.com/huozhi) 提交）
