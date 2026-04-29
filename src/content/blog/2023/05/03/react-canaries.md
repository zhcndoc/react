---
title: "React Canaries：在 Meta 之外实现渐进式功能发布"
author: Dan Abramov, Sophie Alpert, Rick Hanlon, Sebastian Markbage, and Andrew Clark
date: 2023/05/03
description: 我们希望为 React 社区提供一种选择，让大家能在单个新特性的设计接近最终定稿时，就尽早采用它们，而不必等到稳定版本发布——类似于 Meta 这些年来一直在内部使用 React 的前沿版本。我们正在引入一个新的、官方支持的 [Canary 发布通道](/community/versioning-policy#canary-channel)。它让框架等经过筛选的方案能够将单个 React 特性的采用与 React 的发布节奏解耦。
---

2023 年 5 月 3 日，作者 [Dan Abramov](https://bsky.app/profile/danabra.mov)、[Sophie Alpert](https://twitter.com/sophiebits)、[Rick Hanlon](https://twitter.com/rickhanlonii)、[Sebastian Markbåge](https://twitter.com/sebmarkbage) 和 [Andrew Clark](https://twitter.com/acdlite)

---

<Intro>

我们希望为 React 社区提供一种选择，让大家能在单个新特性的设计接近最终定稿时，就尽早采用它们，而不必等到稳定版本发布——类似于 Meta 这些年来一直在内部使用 React 的前沿版本。我们正在引入一个新的、官方支持的 [Canary 发布通道](/community/versioning-policy#canary-channel)。它让框架等经过筛选的方案能够将单个 React 特性的采用与 React 的发布节奏解耦。

</Intro>

---

## tl;dr {/*tldr*/}

* 我们正在为 React 引入一个官方支持的 [Canary 发布通道](/community/versioning-policy#canary-channel)。由于它是官方支持的，如果出现任何回归，我们会以类似处理稳定版缺陷的紧迫程度来对待。
* Canary 让你可以在单个新的 React 特性进入 semver 稳定版之前就开始使用它们。
* 与 [Experimental](/community/versioning-policy#experimental-channel) 通道不同，React Canary 只包含我们有理由认为已经准备好被采用的特性。我们鼓励框架考虑捆绑固定版本的 Canary React 发布。
* 我们会在 Canary 版本发布新变化和新特性时，在博客上进行公告。
* **一如既往，React 的每个 Stable 版本都会继续遵循 semver。**

## React 特性通常是如何开发的 {/*how-react-features-are-usually-developed*/}

通常来说，每个 React 特性都会经历相同的阶段：

1. 我们开发一个初始版本，并为其添加 `experimental_` 或 `unstable_` 前缀。该特性只会在 `experimental` 发布通道中可用。此时，这个特性预计还会发生较大变化。
2. 我们找到 Meta 中愿意帮助测试这个特性并提供反馈的团队。这会带来一轮改动。随着特性变得更加稳定，我们会与更多 Meta 团队一起试用它。
3. 最终，我们对这个设计有了信心。我们移除 API 名称中的前缀，并让该特性在默认情况下可用于 `main` 分支，而 Meta 的大多数产品都会使用该分支。此时，Meta 中的任何团队都可以使用这个特性。
4. 当我们对这个方向更有信心时，我们也会为这个新特性发布 RFC。此时我们知道这个设计适用于广泛的场景，但可能还会做一些最后的调整。
5. 当我们接近发布开源版本时，我们会为这个特性编写文档，并最终在稳定版 React 中发布它。

到目前为止，这套流程对我们发布的大多数特性都运作良好。不过，特性通常“准备好使用”的时间（第 3 步）和它在开源中发布的时间（第 5 步）之间，可能会有相当大的间隔。

**我们希望为 React 社区提供一种选择，让他们像 Meta 一样采用同样的方式，并更早地（在特性可用时）采用单个新特性，而不必等待 React 的下一个发布周期。**

一如既往，所有 React 特性最终都会进入 Stable 版本。

## 我们不能多做一些小版本发布吗？ {/*can-we-just-do-more-minor-releases*/}

通常来说，我们*确实*会使用小版本发布来引入新特性。

然而，这并不总是可行。有时，新的特性与*其他*尚未完全完成、且我们仍在积极迭代的特性相互关联。因为它们的实现彼此相关，我们无法将它们分开发布。因为它们会影响相同的包（例如 `react` 和 `react-dom`），我们也无法分别对它们进行版本控制。而且我们需要保留对尚未准备好的部分继续迭代的能力，而不是频繁地发布重大版本，这正是 semver 所要求我们去做的。

在 Meta，我们通过从 `main` 分支构建 React，并每周手动将其更新到某个固定提交来解决这个问题。这也是 React Native 版本在过去几年里一直采用的方式。React Native 的每个*稳定*版本都固定到 React 仓库 `main` 分支上的某个特定提交。这样，React Native 就能在框架层面引入重要的 bug 修复，并逐步采用新的 React 特性，而不会被绑定到全局的 React 发布节奏上。

我们希望将这种工作流提供给其他框架和经过筛选的方案。例如，它让建立在 React *之上*的框架可以在这个破坏性变更进入稳定版 React 发布*之前*就包含它。由于某些破坏性变更只会影响框架集成，这一点尤其有用。这样，框架就可以在自己的小版本中发布这类变更，而不会破坏 semver。

使用 Canaries 通道进行滚动发布，将使我们能够拥有更紧密的反馈循环，并确保新特性在社区中得到充分测试。这个工作流更接近 TC39（JavaScript 标准委员会）[处理分阶段变更的方式](https://tc39.es/process-document/)。新的 React 特性可能会在基于 React 构建的框架中先于 React 稳定版发布时可用，就像新的 JavaScript 特性会先在浏览器中发布，然后才正式作为规范的一部分被批准一样。

## 为什么不直接使用 experimental 发布版？ {/*why-not-use-experimental-releases-instead*/}

虽然从技术上讲你*可以*使用 [Experimental 发布版](/community/versioning-policy#canary-channel)，但我们不建议在生产环境中使用它们，因为实验性 API 在走向稳定的过程中可能会经历重大的破坏性变更（甚至可能被完全移除）。虽然 Canary 也可能包含错误（任何版本都可能如此），但从今以后，我们计划在博客上宣布 Canary 中任何重大的破坏性变更。Canary 最接近 Meta 内部运行的代码，因此通常你可以预期它们会相对稳定。不过，在更新这些固定提交之间的版本时，你*确实*需要保持版本固定，并手动查看 GitHub 提交日志。

**我们预计，大多数在经过筛选的方案之外（例如框架）使用 React 的人，会继续使用 Stable 版本。**不过，如果你正在构建一个框架，你可能会希望考虑捆绑一个固定到特定提交的 Canary 版 React，并按你自己的节奏更新它。这样做的好处是，你可以更早地为用户发布单个已完成的 React 特性和 bug 修复，并按照你自己的发布节奏发布，类似于 React Native 在过去几年里的做法。缺点是，你需要承担额外责任，去审查引入了哪些 React 提交，并向用户说明你的发布包含了哪些 React 变更。

如果你是框架作者，并且想尝试这种方法，请与我们联系。

## 提前宣布破坏性变更和新特性 {/*announcing-breaking-changes-and-new-features-early*/}

Canary 发布代表了我们在任何给定时间对下一个稳定版 React 将包含内容的最佳预测。

传统上，我们只会在发布周期*结束*时（即进行重大版本发布时）宣布破坏性变更。现在 Canary 发布已经成为一种官方支持的 React 消费方式，我们计划转向在破坏性变更和重要新特性*落地时*就进行宣布。例如，如果我们合并了一个会在 Canary 中发布的破坏性变更，我们会在 React 博客上发文介绍它，必要时还会包含 codemod 和迁移说明。然后，如果你是框架作者，正在发布一个主要版本，并将固定的 React Canary 更新以包含该变更，你就可以在发布说明中链接到我们的博客文章。最后，当 React 的稳定重大版本准备就绪时，我们会链接到这些已经发布的博客文章，希望这能帮助我们的团队更快推进。

我们计划在 API 随 Canary 发布时就对其进行文档说明——即使这些 API 尚未在 Canary 之外可用。仅在 Canary 中可用的 API 会在对应页面上用特别说明标记。这将包括像 [`use`](https://github.com/reactjs/rfcs/pull/229) 这样的 API，以及一些其他的 API（如 `cache` 和 `createServerContext`），我们会为它们发送 RFC。

## Canary 必须固定版本 {/*canaries-must-be-pinned*/}

如果你决定在应用或框架中采用 Canary 工作流，请确保始终固定你正在使用的 Canary 的*准确*版本。由于 Canary 是预发布版本，它们仍可能包含破坏性变更。

## 示例：React Server Components {/*example-react-server-components*/}

正如我们在 [3 月宣布的](/blog/2023/03/22/react-labs-what-we-have-been-working-on-march-2023#react-server-components)那样，React Server Components 的约定已经最终确定，我们不预计其面向用户的 API 合同还会有重大的破坏性变更。然而，我们目前还不能在 React 的稳定版本中发布对 React Server Components 的支持，因为我们仍在处理几个相互交织的仅框架特性（例如 [资源加载](/blog/2023/03/22/react-labs-what-we-have-been-working-on-march-2023#asset-loading)），并且预期那里还会有更多破坏性变更。

这意味着 React Server Components 已经准备好被框架采用了。不过，在下一个 React 大版本发布之前，框架采用它们的唯一方式，就是发布一个固定版本的 Canary React。（为了避免捆绑两份 React，想要这样做的框架需要强制将 `react` 和 `react-dom` 解析到与其框架一起发布的固定 Canary 版本，并向用户说明这一点。举个例子，这就是 Next.js App Router 的做法。）

## 针对稳定版和 Canary 版同时测试库 {/*testing-libraries-against-both-stable-and-canary-versions*/}

我们不期望库作者去测试每一个 Canary 发布版本，因为这会非常困难。不过，正如我们在三年前[首次介绍不同的 React 预发布通道时](https://legacy.reactjs.org/blog/2019/10/22/react-release-channels.html)一样，我们鼓励库同时针对*最新的稳定版*和*最新的 Canary 版*运行测试。如果你发现了未曾宣布的行为变更，请在 React 仓库中提交 bug，以便我们帮助诊断。我们预计，随着这一做法被广泛采用，它将减少将库升级到 React 新主版本所需的工作量，因为意外的回归会在它们刚出现时就被发现。

<Note>

严格来说，Canary 不是一个*新的*发布通道——它以前叫 Next。不过，我们决定将其重命名，以避免与 Next.js 混淆。我们把它作为一个*新的*发布通道来宣布，是为了传达新的预期，例如 Canary 是一种官方支持的使用 React 的方式。

</Note>

## 稳定版发布的工作方式与以往相同 {/*stable-releases-work-like-before*/}

我们不会对稳定版 React 发布引入任何更改。



