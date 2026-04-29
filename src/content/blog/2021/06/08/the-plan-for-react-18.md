---
title: "React 18 的计划"
author: Andrew Clark, Brian Vaughn, Christine Abernathy, Dan Abramov, Rachel Nabors, Rick Hanlon, Sebastian Markbage, and Seth Webster
date: 2021/06/08
description: React 团队很高兴分享一些最新进展。我们已经开始着手 React 18 的发布工作，它将成为我们的下一个重大版本。我们创建了一个工作组，为社区逐步采用 React 18 中的新特性做好准备。我们还发布了 React 18 Alpha，方便库作者进行尝试并提供反馈...
---

2021 年 6 月 8 日，作者：[Andrew Clark](https://twitter.com/acdlite)、[Brian Vaughn](https://github.com/bvaughn)、[Christine Abernathy](https://twitter.com/abernathyca)、[Dan Abramov](https://bsky.app/profile/danabra.mov)、[Rachel Nabors](https://twitter.com/rachelnabors)、[Rick Hanlon](https://twitter.com/rickhanlonii)、[Sebastian Markbåge](https://twitter.com/sebmarkbage) 和 [Seth Webster](https://twitter.com/sethwebster)

---

<Intro>

React 团队很高兴分享一些最新进展：

1. 我们已经开始着手 React 18 的发布工作，它将成为我们的下一个重大版本。
2. 我们创建了一个工作组，为社区逐步采用 React 18 中的新特性做好准备。
3. 我们还发布了 React 18 Alpha，方便库作者进行尝试并提供反馈。

这些更新主要面向第三方库的维护者。如果你正在学习、教学，或者使用 React 构建面向用户的应用程序，你可以放心忽略这篇文章。不过，如果你感兴趣，也欢迎关注 React 18 工作组中的讨论！

---

</Intro>

## React 18 中即将推出什么 {/*whats-coming-in-react-18*/}

在发布时，React 18 将包含开箱即用的改进（例如 [自动批处理](https://github.com/reactwg/react-18/discussions/21)）、新的 API（例如 [`startTransition`](https://github.com/reactwg/react-18/discussions/41)），以及一个[新的流式服务端渲染器](https://github.com/reactwg/react-18/discussions/37)，并内置对 `React.lazy` 的支持。

这些特性之所以能够实现，得益于我们在 React 18 中引入的一种新的可选机制。它被称为“并发渲染”，允许 React 同时准备 UI 的多个版本。这个变化大多发生在幕后，但它为提升应用的实际性能和感知性能打开了新的可能性。

如果你一直在关注我们对 React 未来的研究（虽然我们并不指望你这样做！），你可能听说过某种叫作“并发模式”的东西，或者听说它可能会破坏你的应用。针对社区的这些反馈，我们重新设计了逐步采用的升级策略。并发渲染不再是一个非此即彼的“模式”，而只会对由新特性触发的更新启用。实际上，这意味着**你可以在不重写代码的情况下采用 React 18，并按照自己的节奏尝试新特性。**

## 逐步采用策略 {/*a-gradual-adoption-strategy*/}

由于 React 18 中的并发是可选启用的，因此组件行为不会出现显著的开箱即用破坏性变更。**你可以仅对应用代码做最少甚至不做修改，就升级到 React 18，其工作量与典型的 React 重大版本发布相当**。根据我们将多个应用迁移到 React 18 的经验，我们预计许多用户可以在一个下午内完成升级。

我们已经在 Facebook 成功将并发特性推送到数万个组件，并且根据我们的经验，大多数 React 组件“开箱即用”，无需额外修改。我们致力于确保整个社区都能顺利完成这次升级，因此今天我们宣布成立 React 18 工作组。

## 与社区协作 {/*working-with-the-community*/}

这次发布我们尝试一种新的做法：我们邀请了来自整个 React 社区的一组专家、开发者、库作者和教育者，参与我们的[React 18 工作组](https://github.com/reactwg/react-18)，提供反馈、提出问题并协作推进发布。我们没能把所有想邀请的人都邀请到这个最初的小组中，但如果这次试验顺利，我们希望未来还能邀请更多人加入！

**React 18 工作组的目标是为现有应用和库顺利、渐进地采用 React 18 做好生态准备。** 工作组托管在 [GitHub Discussions](https://github.com/reactwg/react-18/discussions) 上，任何人都可以公开阅读。工作组成员可以留下反馈、提出问题并分享想法。核心团队也会使用这个讨论仓库分享我们的研究发现。随着稳定版发布日期临近，任何重要信息也都会发布在这个博客上。

有关升级到 React 18 的更多信息，或关于本次发布的其他资源，请参阅 [React 18 公告文章](https://github.com/reactwg/react-18/discussions/4)。

## 访问 React 18 工作组 {/*accessing-the-react-18-working-group*/}

任何人都可以阅读 [React 18 工作组仓库](https://github.com/reactwg/react-18)中的讨论。

由于我们预计工作组一开始会受到大量关注，只有受邀成员才能创建主题或在主题下评论。不过，这些主题对公众完全可见，因此每个人都能获取相同的信息。我们认为，这在为工作组成员创造高效环境与保持更广泛社区透明度之间，达成了一个不错的折中。

一如既往，你可以向我们的[问题跟踪器](https://github.com/facebook/react/issues)提交 bug 报告、问题和一般反馈。

## 如何立即尝试 React 18 Alpha {/*how-to-try-react-18-alpha-today*/}

新的 alpha 版本会[定期使用 `@alpha` 标签发布到 npm](https://github.com/reactwg/react-18/discussions/9)。这些发布版本是基于我们主仓库中最新的提交构建的。当某个特性或 bug 修复合并后，它会在下一个工作日出现在 alpha 版本中。

不同 alpha 版本之间可能会有重大的行为或 API 变化。请记住，**不建议将 alpha 版本用于面向用户的生产应用**。

## React 18 预计发布时间线 {/*projected-react-18-release-timeline*/}

我们目前还没有安排具体的发布日期，但我们预计，在 React 18 适用于大多数生产应用之前，还需要数月的反馈和迭代。

* 库 Alpha：今天可用
* 公测版：至少数个月后
* 候选发布版（RC）：在 Beta 之后至少数周
* 正式可用：在 RC 之后至少数周

关于我们预计发布时间线的更多细节可在[工作组中查看](https://github.com/reactwg/react-18/discussions/9)。当我们更接近公开发布时，会在这个博客上发布更新。
