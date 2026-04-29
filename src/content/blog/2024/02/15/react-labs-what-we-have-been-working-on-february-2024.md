---
title: "React Labs: 我们一直在做什么 – 2024年2月"
author: Joseph Savona, Ricky Hanlon, Andrew Clark, Matt Carroll, and Dan Abramov
date: 2024/02/15
description: 在 React Labs 的文章中，我们会介绍正在积极研究和开发的项目。自上次更新以来，我们取得了显著进展，并希望与大家分享我们的进展。
---

2024年2月15日，作者 [Joseph Savona](https://twitter.com/en_JS)、[Ricky Hanlon](https://twitter.com/rickhanlonii)、[Andrew Clark](https://twitter.com/acdlite)、[Matt Carroll](https://twitter.com/mattcarrollcode) 和 [Dan Abramov](https://bsky.app/profile/danabra.mov)。

---

<Intro>

在 React Labs 的文章中，我们会介绍正在积极研究和开发的项目。自从我们的[上次更新](/blog/2023/03/22/react-labs-what-we-have-been-working-on-march-2023)以来，我们取得了显著进展，并希望与大家分享我们的进展。

</Intro>

---

## React 编译器 {/*react-compiler*/}

React Compiler 不再是一个研究项目：这个编译器现在已在生产环境中为 instagram.com 提供支持，我们也在努力将其推广到 Meta 的更多场景，并准备首次开源发布。

正如我们在[上一篇文章](/blog/2023/03/22/react-labs-what-we-have-been-working-on-march-2023#react-optimizing-compiler)中所讨论的那样，当状态变化时，React *有时* 会重新渲染过多内容。自 React 早期以来，我们针对这类情况的解决方案一直是手动记忆化。在我们当前的 API 中，这意味着使用 [`useMemo`](/reference/react/useMemo)、[`useCallback`](/reference/react/useCallback) 和 [`memo`](/reference/react/memo) 这些 API，手动调整 React 在状态变化时重新渲染多少内容。但手动记忆化是一种折中方案。它会让代码变得臃肿，容易出错，而且需要额外工作来保持更新。

手动记忆化是一个合理的折中，但我们并不满足。我们的愿景是：当状态变化时，React 能*自动*只重新渲染 UI 中恰到好处的部分，*同时不损害 React 的核心心智模型*。我们相信，React 的这种方式——将 UI 视为状态的简单函数，并使用标准的 JavaScript 值和惯用法——是 React 能被如此多开发者接受的重要原因。这也是为什么我们一直在投入开发 React 的优化编译器。

JavaScript 是一门众所周知难以优化的语言，这得益于它宽松的规则和动态特性。React Compiler 能够通过同时建模 JavaScript 的规则以及“React 的规则”来安全地编译代码。例如，React 组件必须是幂等的——在相同输入下返回相同的值——并且不能修改 props 或 state 值。这些规则限制了开发者可以做的事情，并帮助为编译器划定一个安全的优化空间。

当然，我们理解开发者有时会稍微违反这些规则，我们的目标是让 React Compiler 尽可能在更多代码上开箱即用。这个编译器会尝试检测代码是否严格遵循 React 的规则，并在安全时编译代码，或在不安全时跳过编译。我们正在使用 Meta 大而多样的代码库进行测试，以帮助验证这种方法。

对于那些希望确认自己的代码是否遵循 React 规则的开发者，我们建议[启用 Strict Mode](/reference/react/StrictMode)并[配置 React 的 ESLint 插件](/learn/editor-setup#linting)。这些工具可以帮助捕捉 React 代码中的细微 bug，提升你当前应用的质量，并让你的应用为 React Compiler 等即将到来的功能做好未来兼容性准备。我们也在整理 React 规则的统一文档，并更新我们的 ESLint 插件，以帮助团队理解并应用这些规则，从而构建更健壮的应用。

如果你想看看编译器实际运行的效果，可以查看我们[去年秋天的演讲](https://www.youtube.com/watch?v=qOQClO3g8-Y)。在演讲时，我们只是针对 instagram.com 的一个页面获得了早期实验数据。此后，我们已经将编译器推向 instagram.com 的生产环境。我们也扩大了团队规模，以加速将其推广到 Meta 的更多场景并开源。我们对前方的道路感到兴奋，并会在接下来的几个月里分享更多内容。

## Actions {/*actions*/}


我们[之前分享过](/blog/2023/03/22/react-labs-what-we-have-been-working-on-march-2023#react-server-components)，我们正在探索使用 Server Actions 将数据从客户端发送到服务器的解决方案，这样你就可以执行数据库变更并实现表单。在开发 Server Actions 的过程中，我们还扩展了这些 API，使其也支持仅客户端应用中的数据处理。

我们将这一更广泛的特性集合简称为“Actions”。Actions 允许你将函数传递给 DOM 元素，例如 [`<form/>`](/reference/react-dom/components/form)：

```js
<form action={search}>
  <input name="query" />
  <button type="submit">搜索</button>
</form>
```

`action` 函数可以同步或异步执行。你可以在客户端使用标准 JavaScript 定义它们，也可以在服务器上使用 [`'use server'`](/reference/rsc/use-server) 指令定义。使用 action 时，React 会替你管理数据提交的生命周期，并提供诸如 [`useFormStatus`](/reference/react-dom/hooks/useFormStatus) 和 [`useActionState`](/reference/react/useActionState) 之类的钩子，以访问表单 action 的当前状态和响应。

默认情况下，Actions 会在一个 [transition](/reference/react/useTransition) 中提交，在 action 处理期间保持当前页面可交互。由于 Actions 支持异步函数，我们也增加了在 transition 中使用 `async/await` 的能力。这使你可以在像 `fetch` 这样的异步请求开始时，通过 transition 的 `isPending` 状态展示等待中的 UI，并一直显示该等待 UI，直到更新被应用。

除了 Actions，我们还引入了一个名为 [`useOptimistic`](/reference/react/useOptimistic) 的功能，用于管理乐观状态更新。借助这个钩子，你可以应用临时更新，并在最终状态提交后自动回滚。对于 Actions，这允许你在客户端乐观地设置数据的最终状态，假设提交成功，然后在收到来自服务器的数据后回退到对应的值。它使用普通的 `async`/`await` 实现，因此无论你是在客户端使用 `fetch`，还是使用来自服务器的 Server Action，它的工作方式都是一样的。

库作者可以使用 `useTransition` 在自己的组件中实现自定义的 `action={fn}` 属性。我们的意图是让库在设计组件 API 时采用 Actions 模式，为 React 开发者提供一致的体验。例如，如果你的库提供了一个 `<Calendar onSelect={eventHandler}>` 组件，也可以考虑同时提供一个 `<Calendar selectAction={action}>` API。

虽然我们最初关注的是用于客户端与服务器之间数据传输的 Server Actions，但我们对 React 的理念是在所有平台和环境中提供相同的编程模型。在可能的情况下，如果我们在客户端引入某个功能，我们也会尽量让它在服务器上同样可用，反之亦然。这种理念使我们能够创建一套无论你的应用运行在哪里都能工作的 API，从而更容易在以后切换到不同环境。

Actions 现在已经可在 Canary 频道中使用，并将在 React 的下一个版本中发布。

## React Canary 中的新功能 {/*new-features-in-react-canary*/}

我们推出了 [React Canaries](/blog/2023/05/03/react-canaries)，作为一种选择：在某个新稳定功能的设计接近最终定稿时，就尽早采用它，而不必等到它以稳定 semver 版本发布。

Canaries 是我们开发 React 方式的一次变化。以前，功能会在 Meta 内部私下研究和构建，因此用户只有在 Stable 版发布时才会看到最终打磨好的产品。借助 Canaries，我们在社区帮助下以公开方式构建，来最终确定我们在 React Labs 博客系列中分享的功能。这意味着你会更早听到新功能的信息，因为它们是在定稿过程中，而不是完成之后才被介绍。

React Server Components、Asset Loading、Document Metadata 和 Actions 都已经进入 React Canary，并且我们已经在 react.dev 上为这些功能添加了文档：

- **指令**：[`"use client"`](/reference/rsc/use-client) 和 [`"use server"`](/reference/rsc/use-server) 是为全栈 React 框架设计的打包器特性。它们标记了两个环境之间的“分割点”：`"use client"` 指示打包器生成一个 `<script>` 标签（类似 [Astro Islands](https://docs.astro.build/en/concepts/islands/#creating-an-island)），而 `"use server"` 则告诉打包器生成一个 POST 端点（类似 [tRPC Mutations](https://trpc.io/docs/concepts)）。它们结合起来，让你可以编写可复用组件，将客户端交互与相关的服务器端逻辑组合在一起。

- **文档元数据**：我们新增了对在组件树中任意位置渲染 [`<title>`](/reference/react-dom/components/title)、[`<meta>`](/reference/react-dom/components/meta) 和元数据 [`<link>`](/reference/react-dom/components/link) 标签的内置支持。这些功能在所有环境中都以相同方式工作，包括纯客户端代码、SSR 和 RSC。这为像 [React Helmet](https://github.com/nfl/react-helmet) 这类库开创的功能提供了内置支持。

- **资源加载**：我们将 Suspense 与样式表、字体和脚本等资源的加载生命周期集成在一起，使 React 能够在判断诸如 [`<style>`](/reference/react-dom/components/style)、[`<link>`](/reference/react-dom/components/link) 和 [`<script>`](/reference/react-dom/components/script) 这样的元素中的内容是否已准备好显示时将它们纳入考虑。我们还新增了像 `preload` 和 `preinit` 这样的 [Resource Loading APIs](/reference/react-dom#resource-preloading-apis)，以便更精细地控制资源何时应当加载和初始化。

- **Actions**：如上所述，我们已经添加了 Actions 来管理将数据从客户端发送到服务器。你可以在诸如 [`<form/>`](/reference/react-dom/components/form) 的元素上添加 `action`，使用 [`useFormStatus`](/reference/react-dom/hooks/useFormStatus) 访问状态，用 [`useActionState`](/reference/react/useActionState) 处理结果，并用 [`useOptimistic`](/reference/react/useOptimistic) 乐观更新 UI。

由于这些功能彼此协同工作，因此很难将它们分别在 Stable 频道中发布。如果没有用于访问表单状态的配套钩子，单独发布 Actions 会限制其实用性。如果引入 React Server Components 却不集成 Server Actions，那么在服务器上修改数据就会变得复杂。

在我们将一组功能发布到 Stable 频道之前，需要确保它们能够协同工作，并且开发者拥有在生产环境中使用它们所需的一切。React Canaries 让我们能够分别开发这些功能，并逐步发布稳定 API，直到整个功能集合完成。

当前 React Canary 中的这组功能已经完善，可以发布了。

## React 的下一个主要版本 {/*the-next-major-version-of-react*/}

经过几年的迭代，`react@canary` 现在已经可以发布到 `react@latest` 了。上面提到的新功能与应用运行所在的任何环境都兼容，并提供生产环境所需的一切。由于 Asset Loading 和 Document Metadata 对某些应用来说可能是破坏性变更，React 的下一个版本将是一个主要版本：**React 19**。

为发布做准备仍有更多工作要完成。在 React 19 中，我们还加入了一些长期以来备受期待的改进，这些改进需要破坏性变更，例如对 Web Components 的支持。我们目前的重点是落地这些变更、准备发布、完成新功能的文档，并公布所包含内容的公告。

在接下来的几个月里，我们会分享更多关于 React 19 所包含内容、如何采用新的客户端功能，以及如何为 React Server Components 构建支持的信息。

## Offscreen（已重命名为 Activity）。 {/*offscreen-renamed-to-activity*/}

自从我们上次更新以来，我们将一个正在研究中的能力从“Offscreen”重命名为“Activity”。“Offscreen”这个名字暗示它只适用于应用中不可见的部分，但在研究这个特性时，我们意识到应用中有些部分可以是可见但不活跃的，例如模态框后面的内容。这个新名称更准确地反映了将应用的某些部分标记为“active”或“inactive”的行为。

Activity 仍处于研究阶段，我们剩余的工作是最终确定向库开发者暴露的原语。在我们专注于发布更完整的功能时，这一领域的优先级已经被降低。

* * *

除了这次更新之外，我们团队还在各类大会和播客中分享了更多关于我们工作的内容，并回答了问题。

- [Sathya Gunasekaran](https://github.com/gsathya) 在 [React India](https://www.youtube.com/watch?v=kjOacmVsLSE) 大会上讲述了 React Compiler

- [Dan Abramov](/community/team#dan-abramov) 在 [RemixConf](https://www.youtube.com/watch?v=zMf_xeGPn6s) 上发表了题为“来自另一个维度的 React”的演讲，探讨了 React Server Components 和 Actions 可能被创造出来的另一种历史

- [Dan Abramov](/community/team#dan-abramov) 接受了 [the Changelog’s JS Party 播客](https://changelog.com/jsparty/311) 的采访，讨论了 React Server Components

- [Matt Carroll](/community/team#matt-carroll) 接受了 [Front-End Fire 播客](https://www.buzzsprout.com/2226499/14462424-interview-the-two-reacts-with-rachel-nabors-evan-bacon-and-matt-carroll) 的采访，在那里他讨论了 [The Two Reacts](https://overreacted.io/the-two-reacts/)

感谢 [Lauren Tan](https://twitter.com/potetotes)、[Sophie Alpert](https://twitter.com/sophiebits)、[Jason Bonta](https://threads.net/someextent)、[Eli White](https://twitter.com/Eli_White) 和 [Sathya Gunasekaran](https://twitter.com/_gsathya) 审阅这篇文章。

感谢阅读，欢迎[在 React Conf 见！](https://conf.react.dev/)
