---
title: "React Labs：我们一直在做什么 – 2023 年 3 月"
author: Joseph Savona, Josh Story, Lauren Tan, Mengdi Chen, Samuel Susla, Sathya Gunasekaran, Sebastian Markbage, and Andrew Clark
date: 2023/03/22
description: 在 React Labs 系列文章中，我们会介绍正在积极研究和开发中的项目。自上次更新以来，我们在这些项目上取得了显著进展，并希望与大家分享我们的收获。
---

2023 年 3 月 22 日，作者：[Joseph Savona](https://twitter.com/en_JS)、[Josh Story](https://twitter.com/joshcstory)、[Lauren Tan](https://twitter.com/potetotes)、[Mengdi Chen](https://twitter.com/mengdi_en)、[Samuel Susla](https://twitter.com/SamuelSusla)、[Sathya Gunasekaran](https://twitter.com/_gsathya)、[Sebastian Markbåge](https://twitter.com/sebmarkbage) 和 [Andrew Clark](https://twitter.com/acdlite)

---

<Intro>

在 React Labs 系列文章中，我们会介绍正在积极研究和开发中的项目。自从我们的[上次更新](/blog/2022/06/15/react-labs-what-we-have-been-working-on-june-2022)以来，我们在这些项目上取得了显著进展，并希望与大家分享我们的收获。

</Intro>

---

## React Server Components {/*react-server-components*/}

React Server Components（或 RSC）是 React 团队设计的一种新的应用架构。

我们最初是在一次[介绍性演讲](/blog/2020/12/21/data-fetching-with-react-server-components)和一份 [RFC](https://github.com/reactjs/rfcs/pull/188) 中分享了关于 RSC 的研究。简单回顾一下，我们引入了一种新的组件——Server Components——它们会提前运行，并且不包含在你的 JavaScript 打包产物中。Server Components 可以在构建期间运行，让你读取文件系统或获取静态内容。它们也可以在服务器上运行，让你无需构建 API 就能访问数据层。你可以通过 props 将数据从 Server Components 传递给浏览器中的交互式 Client Components。

RSC 将以服务器为中心的多页应用的简单“请求/响应”心智模型，与以客户端为中心的单页应用的无缝交互性结合起来，为你提供两者的最佳体验。

自上次更新以来，我们已经合并了 [React Server Components RFC](https://github.com/reactjs/rfcs/blob/main/text/0188-server-components.md) 以确认该提案。我们解决了 [React Server Module Conventions](https://github.com/reactjs/rfcs/blob/main/text/0227-server-module-conventions.md) 提案中尚未解决的问题，并与合作伙伴达成一致，采用 `"use client"` 约定。这些文档也作为 RSC 兼容实现应该支持什么的规范。

最大的变化是，我们引入了 [`async` / `await`](https://github.com/reactjs/rfcs/pull/229) 作为从 Server Components 中获取数据的主要方式。我们还计划通过引入一个名为 `use` 的新 Hook 来支持从客户端加载数据，它可以解包 Promises。虽然我们不能在仅客户端应用中为任意组件支持 `async / await`，但如果你将仅客户端应用组织得与 RSC 应用的结构相似，我们计划在那种情况下为其添加支持。

现在数据获取已经基本理顺，我们正在探索另一个方向：从客户端向服务器发送数据，这样你就可以执行数据库变更并实现表单。我们正在通过允许你跨越 server/client 边界传递 Server Action 函数来实现这一点，然后客户端就可以调用它们，从而提供无缝的 RPC。Server Actions 还可以在 JavaScript 加载之前就为你提供渐进增强的表单。

React Server Components 已经在 [Next.js App Router](/learn/creating-a-react-app#nextjs-app-router) 中发布。这展示了一个路由器的深度集成，它真正将 RSC 作为基础能力来使用，但这并不是构建兼容 RSC 的路由器和框架的唯一方式。RSC 规范所提供的功能与其实现之间有明确的分离。React Server Components 的目标是作为一种规范，适用于在兼容的 React 框架之间工作的组件。

我们通常建议使用现有框架，但如果你需要构建自己的自定义框架，也是可以的。构建你自己的兼容 RSC 的框架并没有我们希望的那么容易，主要是因为需要与打包器进行深度集成。当前这一代打包器在客户端使用方面很出色，但它们并不是为在服务器和客户端之间对单个模块图进行一等支持的拆分而设计的。这就是为什么我们现在正直接与打包器开发者合作，把 RSC 所需的原语内置进去。

## 资源加载 {/*asset-loading*/}

[Suspense](/reference/react/Suspense) 允许你指定当组件的数据或代码仍在加载时屏幕上显示什么。这使你的用户在页面加载以及在路由导航加载更多数据和代码时，都能逐步看到更多内容。然而，从用户的角度来看，在判断新内容是否已准备好时，数据加载和渲染并不能说明全部情况。默认情况下，浏览器会独立加载样式表、字体和图片，这可能导致 UI 跳动和连续的布局偏移。

我们正在努力将 Suspense 与样式表、字体和图片的加载生命周期完全整合起来，这样 React 就能将它们纳入考量，以判断内容是否已准备好显示。在不改变你编写 React 组件方式的前提下，更新将以更加一致、令人愉悦的方式呈现。作为一项优化，我们还将提供一种手动方式，让你可以直接从组件中预加载诸如字体之类的资源。

我们目前正在实现这些功能，很快会带来更多分享。

## 文档元数据 {/*document-metadata*/}

应用中的不同页面和屏幕可能会有不同的元数据，例如 `<title>` 标签、描述以及该屏幕特有的其他 `<meta>` 标签。从维护角度看，把这些信息放在该页面或屏幕对应的 React 组件附近会更易扩展。然而，这些元数据对应的 HTML 标签需要放在文档的 `<head>` 中，而这通常是在你应用最顶层的组件里渲染的。

目前，人们通常用两种技术来解决这个问题。

一种技术是渲染一个特殊的第三方组件，它会把其中的 `<title>`、`<meta>` 以及其他标签移动到文档的 `<head>` 中。这对主流浏览器有效，但有很多客户端并不会运行客户端 JavaScript，例如 Open Graph 解析器，因此这种技术并不普适。

另一种技术是将页面在服务器端分成两部分渲染。首先渲染主内容并收集所有这类标签。然后，用这些标签渲染 `<head>`。最后，将 `<head>` 和主内容发送给浏览器。这种方法是可行的，但它会让你无法利用 [React 18 的流式服务器渲染器](/reference/react-dom/server/renderToReadableStream)，因为你必须等所有内容都渲染完才能发送 `<head>`。

这就是为什么我们要内置支持在组件树中的任意位置直接渲染 `<title>`、`<meta>` 和元数据 `<link>` 标签。它在所有环境中都会以相同方式工作，包括完全客户端代码、SSR，以及未来的 RSC。我们很快会分享更多细节。

## React 优化编译器 {/*react-optimizing-compiler*/}

自上次更新以来，我们一直在积极迭代 [React Forget](/blog/2022/06/15/react-labs-what-we-have-been-working-on-june-2022#react-compiler) 的设计，这是一个面向 React 的优化编译器。我们之前曾将它称为“自动 memo 化编译器”，从某种意义上说这没错。但构建这个编译器帮助我们更深入地理解了 React 的编程模型。理解 React Forget 的更好方式，是把它看作一种自动的*响应性*编译器。

React 的核心思想是：开发者将 UI 定义为当前状态的函数。你使用普通的 JavaScript 值——数字、字符串、数组、对象——并使用标准的 JavaScript 习惯写法——if/else、for 等——来描述组件逻辑。其心智模型是：只要应用状态发生变化，React 就会重新渲染。我们相信，这种简单的心智模型以及尽量贴近 JavaScript 语义，是 React 编程模型中的重要原则。

问题在于，React 有时可能“过于”响应：它可能会重新渲染过多。例如，在 JavaScript 中，我们没有廉价的方法来比较两个对象或数组是否等价（是否具有相同的键和值），因此在每次渲染时创建一个新的对象或数组，可能会让 React 做比实际需要更多的工作。这意味着开发者必须显式地对组件进行 memo 化，以免对变化反应过度。

React Forget 的目标是确保 React 应用默认拥有恰到好处的响应性：应用只会在状态值*有意义地*变化时重新渲染。从实现角度看，这意味着自动进行 memo 化，但我们认为以响应性来理解 React 和 Forget 更为恰当。可以这样理解：React 目前会在对象标识变化时重新渲染；而有了 Forget，React 会在语义值变化时重新渲染——但不会因此付出深度比较的运行时成本。

就具体进展而言，自上次更新以来，我们在编译器设计上进行了大量迭代，以便与这种自动响应性的思路保持一致，并吸收在内部使用编译器时得到的反馈。在去年年底开始对编译器进行一些重大重构之后，我们现在已经开始在 Meta 的有限范围生产环境中使用它。等我们在生产中证明它可行后，计划将其开源。

最后，很多人对编译器是如何工作的表示了兴趣。我们很期待在验证编译器并将其开源时分享更多细节。不过现在我们可以先分享一些内容：

编译器的核心几乎完全与 Babel 解耦，其核心编译器 API（大致上）是输入旧 AST，输出新 AST（同时保留源位置数据）。在底层，我们使用自定义的代码表示和转换流水线来进行低层级语义分析。不过，编译器的主要公开接口将通过 Babel 和其他构建系统插件提供。为了便于测试，我们目前有一个 Babel 插件，它只是一个非常薄的封装：调用编译器生成每个函数的新版本，然后将其替换进去。

在过去几个月重构编译器的过程中，我们希望重点完善核心编译模型，以确保我们能够处理条件分支、循环、重新赋值和变异等复杂情况。不过，JavaScript 有很多方式来表达这些特性：if/else、三元表达式、for、for-in、for-of 等。如果一开始就试图支持完整语言，会推迟我们验证核心模型的时机。相反，我们从一个较小但具有代表性的语言子集开始：let/const、if/else、for 循环、对象、数组、原始值、函数调用，以及其他一些特性。随着我们对核心模型越来越有信心，并逐步完善内部抽象，我们扩展了所支持的语言子集。对于我们尚不支持的语法，我们也会明确标记，记录诊断信息，并跳过对不支持输入的编译。我们有工具可以在 Meta 的代码库上尝试运行编译器，查看哪些不支持的特性最常见，以便我们优先处理它们。我们会继续逐步扩展，直到支持整个语言。

要让 React 组件中的普通 JavaScript 具备响应性，需要一个对语义有深刻理解的编译器，这样它才能准确理解代码在做什么。通过这种方法，我们正在创建一个 JavaScript 内部的响应性系统，让你能够用这门语言的完整表达能力编写任意复杂的产品代码，而不必受限于某种领域特定语言。

## 离屏渲染 {/*offscreen-rendering*/}

离屏渲染是 React 中一项即将推出的能力，可在后台渲染界面，而不会带来额外的性能开销。你可以把它看作一种不仅适用于 DOM 元素，也适用于 React 组件的 [`content-visibility` CSS 属性](https://developer.mozilla.org/en-US/docs/Web/CSS/content-visibility) 版本。在我们的研究中，我们发现了多种使用场景：

- 路由器可以在后台预渲染界面，这样当用户导航到它们时，就能立即显示。
- 标签页切换组件可以保留隐藏标签页的状态，因此用户可以在它们之间切换，而不会丢失进度。
- 虚拟化列表组件可以在可视窗口的上方和下方预渲染更多行。
- 打开模态框或弹出层时，应用的其余部分可以进入“后台”模式，从而禁用除模态框之外的一切事件和更新。

大多数 React 开发者不会直接与 React 的离屏 API 交互。相反，离屏渲染会被集成到路由器和 UI 库等工具中，然后使用这些库的开发者就可以自动受益，而无需额外工作。

其核心思想是，你应该能够在不改变编写组件方式的情况下，将任何 React 树渲染到离屏状态。当某个组件被离屏渲染时，它实际上不会被 *挂载*，直到该组件变为可见时才会挂载——它的副作用不会被触发。例如，如果某个组件使用 `useEffect` 在它第一次出现时记录分析数据，那么预渲染不会影响这些分析数据的准确性。同样地，当某个组件进入离屏状态时，它的副作用也会被卸载。离屏渲染的一个关键特性是，你可以在不丢失状态的情况下切换组件的可见性。

自上次更新以来，我们已经在 Meta 内部的 Android 和 iOS React Native 应用中测试了一个实验性的预渲染版本，并取得了积极的性能结果。我们还改进了离屏渲染与 Suspense 的协作方式——在离屏树中发生挂起时不会触发 Suspense 回退。我们剩余的工作是最终确定向库开发者暴露的原语。我们预计会在今年晚些时候发布一份 RFC，并同时提供一个用于测试和反馈的实验性 API。

## 过渡追踪 {/*transition-tracing*/}

Transition Tracing API 让你能够检测 [React Transitions](/reference/react/useTransition) 何时变慢，并调查其变慢原因。继上次更新之后，我们已经完成了该 API 的初始设计，并发布了一份 [RFC](https://github.com/reactjs/rfcs/pull/238)。基础能力也已经实现。该项目目前处于搁置状态。我们欢迎大家对 RFC 提出反馈，并期待恢复该项目的开发，以便为 React 提供更好的性能测量工具。这对构建在 React Transitions 之上的路由器尤其有用，例如 [Next.js App Router](/learn/creating-a-react-app#nextjs-app-router)。

* * *
除了这次更新之外，我们团队最近还受邀参与了社区播客和直播节目，更深入地介绍我们的工作并回答问题。

* [Dan Abramov](https://bsky.app/profile/danabra.mov) 和 [Joe Savona](https://twitter.com/en_JS) 接受了 [Kent C. Dodds 在他的 YouTube 频道上的采访](https://www.youtube.com/watch?v=h7tur48JSaw)，在其中他们讨论了围绕 React Server Components 的担忧。
* [Dan Abramov](https://bsky.app/profile/danabra.mov) 和 [Joe Savona](https://twitter.com/en_JS) 做客了 [JSParty 播客](https://jsparty.fm/267)，并分享了他们对 React 未来的看法。

感谢 [Andrew Clark](https://twitter.com/acdlite)、[Dan Abramov](https://bsky.app/profile/danabra.mov)、[Dave McCabe](https://twitter.com/mcc_abe)、[Luna Wei](https://twitter.com/lunaleaps)、[Matt Carroll](https://twitter.com/mattcarrollcode)、[Sean Keegan](https://twitter.com/DevRelSean)、[Sebastian Silbermann](https://twitter.com/sebsilbermann)、[Seth Webster](https://twitter.com/sethwebster) 和 [Sophie Alpert](https://twitter.com/sophiebits) 对这篇文章进行审阅。

感谢阅读，我们下次更新再见！
