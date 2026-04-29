---
title: "React Labs：我们一直在做的工作 – 2022 年 6 月"
author:  Andrew Clark, Dan Abramov, Jan Kassens, Joseph Savona, Josh Story, Lauren Tan, Luna Ruan, Mengdi Chen, Rick Hanlon, Robert Zhang, Sathya Gunasekaran, Sebastian Markbage, and Xuan Huang
date: 2022/06/15
description: React 18 的诞生历经多年，也为 React 团队带来了宝贵的经验。它的发布是多年研究和探索众多路径的结果。其中一些路径取得了成功；更多的则是死胡同，但也带来了新的洞见。我们学到的一课是：社区在等待新功能时，如果不了解我们正在探索的这些路径，会感到沮丧。
---

2022 年 6 月 15 日，作者：[Andrew Clark](https://twitter.com/acdlite)、[Dan Abramov](https://bsky.app/profile/danabra.mov)、[Jan Kassens](https://twitter.com/kassens)、[Joseph Savona](https://twitter.com/en_JS)、[Josh Story](https://twitter.com/joshcstory)、[Lauren Tan](https://twitter.com/potetotes)、[Luna Ruan](https://twitter.com/lunaruan)、[Mengdi Chen](https://twitter.com/mengdi_en)、[Rick Hanlon](https://twitter.com/rickhanlonii)、[Robert Zhang](https://twitter.com/jiaxuanzhang01)、[Sathya Gunasekaran](https://twitter.com/_gsathya)、[Sebastian Markbåge](https://twitter.com/sebmarkbage) 和 [Xuan Huang](https://twitter.com/Huxpro)

---

<Intro>

[React 18](/blog/2022/03/29/react-v18) 的诞生历经多年，也为 React 团队带来了宝贵的经验。它的发布是多年研究和探索众多路径的结果。其中一些路径取得了成功；更多的则是死胡同，但也带来了新的洞见。我们学到的一课是：社区在等待新功能时，如果不了解我们正在探索的这些路径，会感到沮丧。

</Intro>

---

我们通常会同时推进许多项目，从更具实验性到定义明确的项目都有。展望未来，我们希望开始 नियमित地与社区分享我们在这些项目上正在做的更多工作。

先说明一下，这不是一份带有明确时间表的路线图。由于其中许多项目仍在积极研究中，很难给出具体的发布时间。根据我们所学到的内容，它们甚至有可能永远不会以当前版本发布。相反，我们想与你们分享我们正在积极思考的问题空间，以及目前为止的收获。

## Server Components {/*server-components*/}

我们在 2020 年 12 月宣布了 [React Server Components 的实验性演示](https://legacy.reactjs.org/blog/2020/12/21/data-fetching-with-react-server-components.html)（RSC）。从那时起，我们一直在完成 React 18 中的相关依赖，并根据实验反馈推进改动。

具体来说，我们正在放弃拥有分叉 I/O 库（例如 react-fetch）的想法，转而采用 async/await 模型，以获得更好的兼容性。严格来说，这并不会阻止 RSC 发布，因为你也可以使用路由器进行数据获取。另一个变化是，我们也在从文件扩展名方案转向[标注边界](https://github.com/reactjs/rfcs/pull/189#issuecomment-1116482278)。

我们正在与 Vercel 和 Shopify 合作，统一 webpack 和 Vite 中对共享语义的打包器支持。在发布之前，我们希望确保 RSC 在整个 React 生态系统中的语义一致。这是达到稳定版的主要阻碍。

## Asset Loading {/*asset-loading*/}

目前，脚本、外部样式、字体和图片等资源通常通过外部系统进行预加载和加载。这会让在流式传输、Server Components 等新环境之间进行协调变得棘手。
我们正在研究通过 React API 添加预加载和加载去重后的外部资源的能力，并使其能在所有 React 环境中工作。

我们也在考虑让这些能力支持 Suspense，这样你就可以拥有图片、CSS 和字体，它们会阻塞显示，直到加载完成，但不会阻塞流式传输和并发渲染。这有助于避免 [“popcorning“](https://twitter.com/sebmarkbage/status/1516852731251724293)，即视觉内容一闪而过并引发布局偏移的情况。

## Static Server Rendering Optimizations {/*static-server-rendering-optimizations*/}

静态站点生成（SSG）和增量静态再生（ISR）是为可缓存页面提供性能的好方法，但我们认为还可以添加一些功能来提升动态服务端渲染（SSR）的性能——尤其是在内容大部分但并非全部都可缓存的时候。我们正在探索利用编译和静态遍历来优化服务端渲染的方法。

## React Optimizing Compiler {/*react-compiler*/}

我们在 React Conf 2021 上给出了 React Forget 的[早期预览](https://www.youtube.com/watch?v=lGEMwh32soc)。它是一个编译器，可以自动生成等价于 `useMemo` 和 `useCallback` 调用的代码，以尽量降低重新渲染的成本，同时保留 React 的编程模型。

最近，我们完成了对这个编译器的重写，使其更加可靠且能力更强。这种新的架构使我们能够分析并记忆更复杂的模式，例如对[局部变更](/learn/keeping-components-pure#local-mutation-your-components-little-secret)的使用，并且除了与记忆化 Hooks 持平之外，还开启了许多新的编译期优化机会。

我们也在为这个编译器制作一个 playground，用来探索其许多方面。虽然 playground 的目标是让编译器开发更容易，但我们认为它也会让人更容易上手尝试，并建立对编译器工作的直觉。它会揭示其底层工作方式的各种洞见，并在你输入时实时渲染编译器的输出。它将与编译器一起在发布时一并推出。

## Offscreen {/*offscreen*/}

如今，如果你想隐藏并显示一个组件，你有两种选择。一种是将其从树中彻底添加或移除。这种方法的问题在于，每次你卸载时，UI 的状态都会丢失，包括存储在 DOM 中的状态，比如滚动位置。

另一种选择是保持组件挂载，并使用 CSS 在视觉上切换其显示状态。这可以保留 UI 的状态，但会带来性能成本，因为 React 必须在收到新更新时继续渲染隐藏的组件及其所有子组件。

Offscreen 引入了第三种选择：在视觉上隐藏 UI，但降低其内容的优先级。这个想法在精神上类似于 `content-visibility` CSS 属性：当内容被隐藏时，它不需要与 UI 的其他部分保持同步。React 可以将渲染工作推迟到应用其余部分空闲时，或直到内容再次变为可见时。

Offscreen 是一种低级能力，它解锁了高级特性。与 React 的其他并发特性类似，比如 `startTransition`，在大多数情况下你不会直接与 Offscreen API 交互，而是通过一个有明确意见的框架来实现如下模式：

* **即时切换。** 一些路由框架已经会预取数据来加快后续导航，例如在悬停链接时。借助 Offscreen，它们还可以在后台预渲染下一个界面。
* **可复用状态。** 同样地，在路由或标签页之间导航时，你可以使用 Offscreen 来保留上一个界面的状态，这样你就可以切回去并从离开的地方继续。
* **虚拟化列表渲染。** 在显示大型项目列表时，虚拟化列表框架会预渲染比当前可见更多的行。你可以使用 Offscreen 以低于列表中可见项目的优先级来预渲染隐藏的行。
* **后台内容。** 我们也在探索一种相关功能，用于在不隐藏内容的情况下，将后台内容降级处理，比如在显示模态层时。

## Transition Tracing {/*transition-tracing*/}

目前，React 有两个性能分析工具。[原始 Profiler](https://legacy.reactjs.org/blog/2018/09/10/introducing-the-react-profiler.html) 会显示一次分析会话中所有提交的概览。对于每次提交，它还会显示所有渲染的组件，以及它们各自渲染所花费的时间。我们在 React 18 中还引入了一个 beta 版本的 [Timeline Profiler](https://github.com/reactwg/react-18/discussions/76)，它会显示组件何时调度更新，以及 React 何时处理这些更新。这两个分析工具都能帮助开发者识别代码中的性能问题。

我们意识到，开发者并不觉得了解单个缓慢提交或脱离上下文的组件有多大用处。更有用的是知道到底是什么导致了这些缓慢提交。而且开发者希望能够跟踪特定交互（例如按钮点击、初始加载或页面导航），以观察性能回退，并理解为什么某次交互变慢以及如何修复。

我们之前尝试通过创建一个 [Interaction Tracing API](https://gist.github.com/bvaughn/8de925562903afd2e7a12554adcdda16) 来解决这个问题，但它存在一些根本性的设计缺陷，降低了追踪某次交互为何变慢的准确性，并且有时会导致交互永远不会结束。由于这些问题，我们最终[移除了这个 API](https://github.com/facebook/react/pull/20037)。

我们正在为 Interaction Tracing API 开发一个新版本（暂定名为 Transition Tracing，因为它是通过 `startTransition` 启动的），以解决这些问题。

## New React Docs {/*new-react-docs*/}

去年，我们宣布了新 React 文档网站的 beta 版本（[后来作为 react.dev 发布](/blog/2023/03/16/introducing-react-dev)）的新 React 文档网站。新的学习材料以 Hooks 为起点，配有新的图表、插图，以及许多交互式示例和练习。我们暂停了这项工作，去专注于 React 18 的发布，但现在 React 18 已经推出，我们正在积极完成并发布新的文档。

我们目前正在撰写关于 effects 的详细章节，因为我们听说这对新手和有经验的 React 用户来说，都是更具挑战性的主题之一。[使用 Effects 保持同步](/learn/synchronizing-with-effects) 是这一系列中首个发布的页面，未来几周还会有更多内容推出。当我们开始撰写关于 effects 的详细章节时，我们意识到许多常见的 effect 模式可以通过向 React 添加一个新的原语来简化。我们在 [useEvent RFC](https://github.com/reactjs/rfcs/pull/220) 中分享了对此的一些初步想法。它目前仍处于早期研究阶段，我们还在继续迭代这一想法。我们很感谢社区至今对该 RFC 的评论，以及对持续进行中的文档重写所提供的[反馈](https://github.com/reactjs/react.dev/issues/3308)和贡献。我们特别要感谢 [Harish Kumar](https://github.com/harish-sethuraman) 提交并审阅了新网站实现中的许多改进。

*感谢 [Sophie Alpert](https://twitter.com/sophiebits) 审阅这篇博文！*
