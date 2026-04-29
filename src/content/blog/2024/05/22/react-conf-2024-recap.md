---
title: "React Conf 2024 回顾"
author: Ricky Hanlon
date: 2024/05/22
description: 上周我们举办了 React Conf 2024，这是一场在内华达州亨德森举行的为期两天的会议，700 多名与会者亲临现场讨论 UI 工程领域的最新进展。在这篇文章中，我们将总结此次活动中的演讲和公告。
---

2024 年 5 月 22 日，作者 [Ricky Hanlon](https://twitter.com/rickhanlonii)。

---

<Intro>

上周我们举办了 React Conf 2024，这是一场在内华达州亨德森举行的为期两天的会议，700 多名与会者亲临现场讨论 UI 工程领域的最新进展。这是我们自 2019 年以来首次线下举办的会议，我们非常高兴能够再次将社区汇聚在一起。

</Intro>

---

在 React Conf 2024 上，我们公布了 [React 19 RC](/blog/2024/12/05/react-19)、[React Native 新架构 Beta 版](https://github.com/reactwg/react-native-new-architecture/discussions/189)，以及 [React Compiler](/learn/react-compiler) 的实验性版本。社区成员也登台宣布了 [React Router v7](https://remix.run/blog/merging-remix-and-react-router)、Expo Router 中的 [Universal Server Components](https://www.youtube.com/watch?v=T8TZQ6k4SLE&t=20765s)、[RedwoodJS](https://redwoodjs.com/blog/rsc-now-in-redwoodjs) 中的 React Server Components，以及更多内容。

完整的 [day 1](https://www.youtube.com/watch?v=T8TZQ6k4SLE) 和 [day 2](https://www.youtube.com/watch?v=0ckOUBiuxVY) 直播现已可在线查看。在这篇文章中，我们将总结此次活动中的演讲和公告。

## Day 1 {/*day-1*/}

_[在此观看 day 1 的完整直播。](https://www.youtube.com/watch?v=T8TZQ6k4SLE&t=973s)_

在 day 1 开场时，Meta CTO [Andrew "Boz" Bosworth](https://www.threads.net/@boztank) 先发送了欢迎致辞，随后由 [Seth Webster](https://twitter.com/sethwebster) 介绍，他负责 Meta 的 React Org，还有我们的主持人 [Ashley Narcisse](https://twitter.com/_darkfadr)。

在 day 1 的主题演讲中，[Joe Savona](https://twitter.com/en_JS) 分享了我们对 React 的目标和愿景：让任何人都能轻松构建出色的用户体验。[Lauren Tan](https://twitter.com/potetotes) 随后带来了《State of React》，她提到 React 在 2023 年的下载量超过了 10 亿次，并且 37% 的新开发者是使用 React 学习编程的。最后，她强调了 React 社区为让 React 成为“React”所做的工作。

想了解更多，可以查看会议后续来自社区的这些演讲：

- [Vanilla React](https://www.youtube.com/watch?v=T8TZQ6k4SLE&t=5542s)，由 [Ryan Florence](https://twitter.com/ryanflorence) 主讲
- [React Rhythm & Blues](https://www.youtube.com/watch?v=0ckOUBiuxVY&t=12728s)，由 [Lee Robinson](https://twitter.com/leeerob) 主讲
- [RedwoodJS, now with React Server Components](https://www.youtube.com/watch?v=T8TZQ6k4SLE&t=26815s)，由 [Amy Dutton](https://twitter.com/selfteachme) 主讲
- [Introducing Universal React Server Components in Expo Router](https://www.youtube.com/watch?v=T8TZQ6k4SLE&t=20765s)，由 [Evan Bacon](https://twitter.com/Baconbrix) 主讲

接下来在主题演讲中，[Josh Story](https://twitter.com/joshcstory) 和 [Andrew Clark](https://twitter.com/acdlite) 分享了 React 19 即将推出的新特性，并宣布了可用于生产环境测试的 React 19 RC。你可以在 [React 19 发布文章](/blog/2024/12/05/react-19)中查看所有特性，也可以观看这些深入介绍新特性的演讲：

- [React 19 有什么新内容](https://www.youtube.com/watch?v=T8TZQ6k4SLE&t=8880s)，由 [Lydia Hallie](https://twitter.com/lydiahallie) 主讲
- [React Unpacked: React 19 路线图](https://www.youtube.com/watch?v=T8TZQ6k4SLE&t=10112s)，由 [Sam Selikoff](https://twitter.com/samselikoff) 主讲
- [React 19 深入解析：协调 HTML](https://www.youtube.com/watch?v=T8TZQ6k4SLE&t=24916s)，由 [Josh Story](https://twitter.com/joshcstory) 主讲
- [使用 React Server Components 增强表单](https://www.youtube.com/watch?v=0ckOUBiuxVY&t=25280s)，由 [Aurora Walberg Scharff](https://twitter.com/aurorascharff) 主讲
- [React for Two Computers](https://www.youtube.com/watch?v=T8TZQ6k4SLE&t=18825s)，由 [Dan Abramov](https://bsky.app/profile/danabra.mov) 主讲
- [And Now You Understand React Server Components](https://www.youtube.com/watch?v=0ckOUBiuxVY&t=11256s)，由 [Kent C. Dodds](https://twitter.com/kentcdodds) 主讲

最后，我们在主题演讲结尾由 [Joe Savona](https://twitter.com/en_JS)、[Sathya Gunasekaran](https://twitter.com/_gsathya) 和 [Mofei Zhang](https://twitter.com/zmofei) 宣布 React Compiler 现已 [开源](https://github.com/facebook/react/pull/29061)，并分享了一个可供尝试的 React Compiler 实验版本。

如需了解更多关于如何使用 Compiler 以及其工作原理的信息，请查看 [文档](/learn/react-compiler)和这些演讲：

- [忘记 Memo 吧](https://www.youtube.com/watch?v=T8TZQ6k4SLE&t=12020s)，由 [Lauren Tan](https://twitter.com/potetotes) 主讲
- [React Compiler 深入解析](https://www.youtube.com/watch?v=0ckOUBiuxVY&t=9313s)，由 [Sathya Gunasekaran](https://twitter.com/_gsathya) 和 [Mofei Zhang](https://twitter.com/zmofei) 主讲

在此观看 day 1 的完整主题演讲：

<YouTubeIframe src="https://www.youtube.com/embed/T8TZQ6k4SLE?t=973s" />

## Day 2 {/*day-2*/}

_[在此观看 day 2 的完整直播。](https://www.youtube.com/watch?v=0ckOUBiuxVY&t=1720s)_

在 day 2 开场时，[Seth Webster](https://twitter.com/sethwebster) 发送了欢迎致辞，随后是 [Eli White](https://x.com/Eli_White) 的感谢致辞，以及由我们的首席氛围官 [Ashley Narcisse](https://twitter.com/_darkfadr) 进行的介绍。

在 day 2 的主题演讲中，[Nicola Corti](https://twitter.com/cortinico) 分享了 React Native 的现状，包括 2023 年的 7800 万次下载。他还重点介绍了使用 React Native 的应用，包括 Meta 内部使用的 2000 多个屏幕；Facebook Marketplace 中产品详情页，每天访问量超过 20 亿次；以及 Microsoft Windows 开始菜单的一部分，还有几乎每个微软 Office 产品在移动端和桌面端中的一些功能。

Nicola 还强调了社区为支持 React Native 所做的所有工作，包括库、框架和多个平台。想了解更多，可以查看社区的这些演讲：

- [将 React Native 扩展到移动端和桌面应用之外](https://www.youtube.com/watch?v=0ckOUBiuxVY&t=5798s)，由 [Chris Traganos](https://twitter.com/chris_trag) 和 [Anisha Malde](https://twitter.com/anisha_malde) 主讲
- [使用 React 进行空间计算](https://www.youtube.com/watch?v=0ckOUBiuxVY&t=22525s)，由 [Michał Pierzchała](https://twitter.com/thymikee) 主讲

[Riccardo Cipolleschi](https://twitter.com/cipolleschir) 继续主持 day 2 的主题演讲，宣布 React Native 新架构现已进入 Beta 阶段，并已准备好供应用在生产环境中采用。他分享了新架构中的新特性和改进，并介绍了 React Native 未来的路线图。更多内容请查看：

- [跨平台 React](https://www.youtube.com/watch?v=0ckOUBiuxVY&t=26569s)，由 [Olga Zinoveva](https://github.com/SlyCaptainFlint) 和 [Naman Goel](https://twitter.com/naman34) 主讲

接下来在主题演讲中，Nicola 宣布我们现在建议所有使用 React Native 创建的新应用都从 Expo 这样的框架开始。伴随这一变化，他还宣布了新的 React Native 首页和新的 Getting Started 文档。你可以在 [React Native 文档](https://reactnative.dev/docs/next/environment-setup)中查看新的入门指南。

最后，在主题演讲结束时，[Kadi Kraman](https://twitter.com/kadikraman) 分享了 Expo 的最新功能和改进，以及如何使用 Expo 开始进行 React Native 开发。

在此观看 day 2 的完整主题演讲：

<YouTubeIframe src="https://www.youtube.com/embed/0ckOUBiuxVY?t=1720s" />

## Q&A {/*q-and-a*/}

React 和 React Native 团队也在每天结束时举行了 Q&A 环节：

- [React Q&A](https://www.youtube.com/watch?v=T8TZQ6k4SLE&t=27518s)，由 [Michael Chan](https://twitter.com/chantastic) 主持
- [React Native Q&A](https://www.youtube.com/watch?v=0ckOUBiuxVY&t=27935s)，由 [Jamon Holmgren](https://twitter.com/jamonholmgren) 主持

## And more... {/*and-more*/}

我们还听到了关于可访问性、错误报告、css 等主题的演讲：

- [揭开 React 应用中可访问性的神秘面纱](https://www.youtube.com/watch?v=0ckOUBiuxVY&t=20655s)，由 [Kateryna Porshnieva](https://twitter.com/krambertech) 主讲
- [Pigment CSS，服务器组件时代的 CSS](https://www.youtube.com/watch?v=0ckOUBiuxVY&t=21696s)，由 [Olivier Tassinari](https://twitter.com/olivtassinari) 主讲
- [实时 React Server Components](https://www.youtube.com/watch?v=T8TZQ6k4SLE&t=24070s)，由 [Sunil Pai](https://twitter.com/threepointone) 主讲
- [让我们打破 React 规则](https://www.youtube.com/watch?v=T8TZQ6k4SLE&t=25862s)，由 [Charlotte Isambert](https://twitter.com/c_isambert) 主讲
- [解决你 100% 的错误](https://www.youtube.com/watch?v=0ckOUBiuxVY&t=19881s)，由 [Ryan Albrecht](https://github.com/ryan953) 主讲

## 感谢 {/*thank-you*/}

感谢所有让 React Conf 2024 成为可能的工作人员、演讲者和参与者。名单太长，无法一一列出，但我们特别想感谢其中几位。

感谢 [Barbara Markiewicz](https://twitter.com/barbara_markie)、[Callstack](https://www.callstack.com/) 团队，以及我们的 React Team Developer Advocate [Matt Carroll](https://twitter.com/mattcarrollcode) 帮助筹划了整个活动；也感谢 [Sunny Leggett](https://zeroslopeevents.com/about) 和 [Zero Slope](https://zeroslopeevents.com) 的所有成员帮助组织了此次活动。

感谢 [Ashley Narcisse](https://twitter.com/_darkfadr) 担任我们的主持人和首席氛围官；也感谢 [Michael Chan](https://twitter.com/chantastic) 和 [Jamon Holmgren](https://twitter.com/jamonholmgren) 主持 Q&A 环节。

感谢 [Seth Webster](https://twitter.com/sethwebster) 和 [Eli White](https://x.com/Eli_White) 每天欢迎我们的到来并就整体结构和内容提供指导；也感谢 [Tom Occhino](https://twitter.com/tomocchino) 在 after-party 期间与我们分享特别致辞。

感谢 [Ricky Hanlon](https://www.youtube.com/watch?v=FxTZL2U-uKg&t=1263s) 提供了关于演讲的详细反馈、参与幻灯片设计，并总体上补足细节、精益求精。

感谢 [Callstack](https://www.callstack.com/) 搭建会议网站；也感谢 [Kadi Kraman](https://twitter.com/kadikraman) 和 [Expo](https://expo.dev/) 团队开发会议移动应用。

感谢所有让此次活动成为可能的赞助商：[Remix](https://remix.run/)、[Amazon](https://developer.amazon.com/apps-and-games?cmp=US_2024_05_3P_React-Conf-2024&ch=prtnr&chlast=prtnr&pub=ref&publast=ref&type=org&typelast=org)、[MUI](https://mui.com/)、[Sentry](https://sentry.io/for/react/?utm_source=sponsored-conf&utm_medium=sponsored-event&utm_campaign=frontend-fy25q2-evergreen&utm_content=logo-reactconf2024-learnmore)、[Abbott](https://www.jobs.abbott/software)、[Expo](https://expo.dev/)、[RedwoodJS](https://rwsdk.com/) 和 [Vercel](https://vercel.com)。

感谢 AV 团队提供视觉、舞台和音响支持；也感谢 Westin Hotel 的接待。

感谢所有向社区分享知识和经验的演讲者。

最后，感谢所有亲临现场和在线参与的人，展示了什么是 React，React。React 不仅仅是一个库，它更是一个社区，看到大家齐聚一堂共同分享与学习，令人深受鼓舞。

下次再见！

