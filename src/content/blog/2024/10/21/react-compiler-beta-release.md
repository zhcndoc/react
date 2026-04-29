---
title: "React Compiler Beta 版本发布"
author: Lauren Tan
date: 2024/10/21
description: 在 React Conf 2024 上，我们宣布了 React Compiler 的实验性发布，这是一个通过自动记忆化优化你的 React 应用的构建时工具。在这篇文章中，我们想分享开源接下来的计划，以及我们在编译器上的进展。

---

2024 年 10 月 21 日，作者 [Lauren Tan](https://twitter.com/potetotes)。

---

<Note>

### React Compiler 现已稳定！ {/*react-compiler-is-now-in-rc*/}

详情请参见[稳定版发布博文](/blog/2025/10/07/react-compiler-1)。

</Note>

<Intro>

React 团队很高兴分享最新更新：

</Intro>

1. 我们今天发布 React Compiler Beta 版，以便早期采用者和库维护者可以试用并提供反馈。
2. 我们通过一个可选的 `react-compiler-runtime` 包，正式支持 React 17+ 上的 React Compiler。
3. 我们正在向公众开放 [React Compiler Working Group](https://github.com/reactwg/react-compiler) 的成员资格，为社区逐步采用编译器做准备。

---

在 [React Conf 2024](/blog/2024/05/22/react-conf-2024-recap) 上，我们宣布了 React Compiler 的实验性发布，这是一个通过自动记忆化优化你的 React 应用的构建时工具。[你可以在这里找到 React Compiler 的介绍](/learn/react-compiler)。

自首次发布以来，我们修复了 React 社区报告的许多 bug，收到了若干高质量的 bug 修复和对编译器的贡献[^1]，使编译器对广泛多样的 JavaScript 模式更加稳健，并继续在 Meta 更广泛地推广编译器。

在这篇文章中，我们想分享 React Compiler 接下来的计划。

## 今天就试用 React Compiler Beta 版 {/*try-react-compiler-beta-today*/}

在 [React India 2024](https://www.youtube.com/watch?v=qd5yk2gxbtg) 上，我们分享了 React Compiler 的更新。今天，我们很高兴宣布 React Compiler 和 ESLint 插件的一个新的 Beta 版本。新的 beta 版本通过 npm 使用 `@beta` 标签发布。

安装 React Compiler Beta 版：

<TerminalBlock>
npm install -D babel-plugin-react-compiler@beta eslint-plugin-react-compiler@beta
</TerminalBlock>

或者，如果你使用 Yarn：

<TerminalBlock>
yarn add -D babel-plugin-react-compiler@beta eslint-plugin-react-compiler@beta
</TerminalBlock>

你可以在这里观看 [Sathya Gunasekaran](https://twitter.com/_gsathya) 在 React India 的演讲：

<YouTubeIframe src="https://www.youtube.com/embed/qd5yk2gxbtg" />

## 我们建议每个人今天都使用 React Compiler 的 linter {/*we-recommend-everyone-use-the-react-compiler-linter-today*/}

React Compiler 的 ESLint 插件帮助开发者主动识别并修正 [React 规则](/reference/rules) 违规。**我们强烈建议每个人今天都使用这个 linter**。该 linter 不要求你已经安装编译器，因此即使你还没准备好试用编译器，也可以独立使用它。

仅安装 linter：

<TerminalBlock>
npm install -D eslint-plugin-react-compiler@beta
</TerminalBlock>

或者，如果你使用 Yarn：

<TerminalBlock>
yarn add -D eslint-plugin-react-compiler@beta
</TerminalBlock>

安装后，你可以通过[将其添加到你的 ESLint 配置中](/learn/react-compiler/installation#eslint-integration)来启用这个 linter。使用 linter 有助于识别对 React 规则的破坏，从而在编译器正式发布时更容易采用它。

## 向后兼容性 {/*backwards-compatibility*/}

React Compiler 生成的代码依赖于在 React 19 中新增的运行时 API，但我们后来又增加了让编译器也能与 React 17 和 18 配合工作的支持。如果你还没有使用 React 19，那么在 Beta 版本中，你现在可以通过在编译器配置中指定一个最低 `target`，并添加 `react-compiler-runtime` 作为依赖来试用 React Compiler。[你可以在这里找到相关文档](/reference/react-compiler/configuration#react-17-18)。

## 在库中使用 React Compiler {/*using-react-compiler-in-libraries*/}

我们最初的发布重点是识别在应用中使用编译器的重大问题。自那以后，我们收到了很棒的反馈，并且已经大幅改进了编译器。我们现在已经准备好从社区获得更广泛的反馈，也欢迎库作者试用编译器，以提升性能并改善维护你的库时的开发体验。

React Compiler 也可以用于编译库。因为 React Compiler 需要在任何代码转换之前，作用于原始源代码，所以应用的构建流程不可能去编译它所使用的库。因此，我们建议库维护者独立地用编译器编译并测试自己的库，然后将编译后的代码发布到 npm。

由于你的代码已经预编译，你的库用户无需启用编译器，也能从应用到你库上的自动记忆化中受益。如果你的库面向尚未升级到 React 19 的应用，请指定一个最低 `target`，并将 `react-compiler-runtime` 添加为直接依赖。运行时包会根据应用的版本使用正确的 API 实现，并在必要时为缺失的 API 提供 polyfill。

[你可以在这里找到更多相关文档。](/reference/react-compiler/compiling-libraries)

## 向所有人开放 React Compiler Working Group {/*opening-up-react-compiler-working-group-to-everyone*/}

我们之前在 React Conf 上宣布了仅限邀请加入的 [React Compiler Working Group](https://github.com/reactwg/react-compiler)，用于提供反馈、提问并协作推进编译器的实验性发布。

从今天起，随着 React Compiler Beta 版一起，我们将向所有人开放 Working Group 成员资格。React Compiler Working Group 的目标是帮助现有应用和库为平稳、渐进地采用 React Compiler 做好生态准备。请继续在 [React 仓库](https://github.com/facebook/react) 中提交 bug 报告，但请在 [Working Group 讨论论坛](https://github.com/reactwg/react-compiler/discussions) 中留下反馈、提出问题或分享想法。

核心团队也会使用 discussions 仓库分享我们的研究发现。随着稳定版发布临近，任何重要信息也都会发布在这个论坛上。

## Meta 的 React Compiler {/*react-compiler-at-meta*/}

在 [React Conf](/blog/2024/05/22/react-conf-2024-recap) 上，我们分享了编译器在 Quest Store 和 Instagram 上的推广是成功的。自那以后，我们又在 Meta 的更多大型 Web 应用中部署了 React Compiler，包括 [Facebook](https://www.facebook.com) 和 [Threads](https://www.threads.net)。这意味着，如果你最近使用过这些应用中的任何一个，你的体验可能由编译器提供支持。我们能够以很少的代码改动将这些应用接入编译器，而且是在一个拥有超过 100,000 个 React 组件的 monorepo 中完成的。

我们在所有这些应用中都看到了显著的性能提升。随着推广的推进，我们继续看到与[我们此前在 ReactConf 分享的成果](https://youtu.be/lyEKhv8-3n0?t=3223)同量级的结果。这些应用多年来已经被 Meta 工程师和 React 专家进行了大量手工调优和优化，因此即便是几个百分点的提升，对我们来说也是巨大的胜利。

我们也预期 React Compiler 会带来开发效率上的收益。为此，我们与 Meta 的数据科学伙伴[^2]合作，对手动记忆化对开发效率的影响进行了深入的统计分析。在 Meta 推广编译器之前，我们发现只有大约 8% 的 React pull request 使用了手动记忆化，而这些 pull request 的编写时间长了 31-46%[^3]。这验证了我们的直觉：手动记忆化会增加认知负担，我们预计 React Compiler 将带来更高效的代码编写和评审。值得注意的是，React Compiler 还确保默认情况下*所有*代码都会被记忆化，而不仅仅是（在我们的情况下）开发者显式应用记忆化的那 8%。

## 通往稳定版的路线图 {/*roadmap-to-stable*/}

*这不是最终路线图，可能会发生变化。*

我们计划在 Beta 版发布后不久，在不远的将来推出编译器的 Release Candidate，当时大多数遵循 React 规则的应用和库已经被证明能够很好地与编译器协作。经过一段来自社区的最终反馈期后，我们计划为编译器发布稳定版。稳定版将标志着 React 新基础的开始，并且会强烈建议所有应用和库使用编译器和 ESLint 插件。

* ✅ 实验性：已在 React Conf 2024 发布，主要用于从早期采用者那里获取反馈。
* ✅ 公共 Beta：今天可用，用于获取更广泛社区的反馈。
* 🚧 Release Candidate (RC)：React Compiler 能够无问题地适用于大多数遵循规则的应用和库。
* 🚧 一般可用：在来自社区的最终反馈期之后。

这些发布还包括编译器的 ESLint 插件，它会显示由编译器静态分析出来的诊断信息。我们计划将现有的 eslint-plugin-react-hooks 插件与编译器的 ESLint 插件合并，这样只需要安装一个插件。

在稳定版之后，我们计划增加更多编译器优化和改进。这既包括对自动记忆化的持续改进，也包括全新的优化，而对产品代码几乎不需要做任何更改。升级到编译器的每个新版本都旨在保持简单，而每次升级都会继续提升性能，并更好地处理多样的 JavaScript 和 React 模式。

在整个过程中，我们还计划为 React 原型开发一个 IDE 扩展。由于研究仍处于非常早期阶段，我们预计会在未来的 React Labs 博文中与大家分享更多发现。

---

感谢 [Sathya Gunasekaran](https://twitter.com/_gsathya)、[Joe Savona](https://twitter.com/en_JS)、[Ricky Hanlon](https://twitter.com/rickhanlonii)、[Alex Taylor](https://github.com/alexmckenley)、[Jason Bonta](https://twitter.com/someextent) 和 [Eli White](https://twitter.com/Eli_White) 对本文进行审阅和编辑。

---

[^1]: 感谢 [@nikeee](https://github.com/facebook/react/pulls?q=is%3Apr+author%3Anikeee)、[@henryqdineen](https://github.com/facebook/react/pulls?q=is%3Apr+author%3Ahenryqdineen)、[@TrickyPi](https://github.com/facebook/react/pulls?q=is%3Apr+author%3ATrickyPi) 以及其他几位为编译器作出贡献的人。

[^2]: 感谢 [Vaishali Garg](https://www.linkedin.com/in/vaishaligarg09) 在 Meta 领导这项关于 React Compiler 的研究，并审阅本文。

[^3]: 在控制作者任职时长、diff 长度/复杂度以及其他潜在混杂因素之后。