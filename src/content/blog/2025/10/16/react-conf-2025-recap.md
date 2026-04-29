---
title: "React Conf 2025 回顾"
author: Matt Carroll 和 Ricky Hanlon
date: 2025/10/16
description: 上周我们举办了 React Conf 2025，在这篇文章中，我们总结了此次活动中的演讲和公告...
---

2025 年 10 月 16 日，作者 [Matt Carroll](https://x.com/mattcarrollcode) 和 [Ricky Hanlon](https://bsky.app/profile/ricky.fm)

---

<Intro>

上周我们举办了 React Conf 2025，在会上我们宣布了 [React Foundation](/blog/2025/10/07/introducing-the-react-foundation)，并展示了即将来到 React 和 React Native 的新特性。

</Intro>

---

React Conf 2025 于 2025 年 10 月 7 日至 8 日在内华达州亨德森举行。

完整的 [第一天](https://www.youtube.com/watch?v=zyVRg2QR6LA&t=1067s) 和 [第二天](https://www.youtube.com/watch?v=p9OcztRyDl0&t=2299s) 直播都可在线查看，你也可以在[这里](https://conf.react.dev/photos)查看活动照片。

在这篇文章中，我们将总结此次活动中的演讲和公告。


## 第一天主题演讲 {/*day-1-keynote*/}

_观看完整的第一天直播 [这里。](https://www.youtube.com/watch?v=zyVRg2QR6LA&t=1067s)_

在第一天主题演讲中，Joe Savona 分享了自上次 React Conf 以来团队和社区的更新，以及来自 React 19.0 和 19.1 的亮点。

Mofei Zhang 重点介绍了 React 19.2 中的新特性，包括：
* [`<Activity />`](https://react.dev/reference/react/Activity)  — 一个用于管理可见性的全新组件。
* [`useEffectEvent`](https://react.dev/reference/react/useEffectEvent) 用于从 Effects 中触发事件。
* [Performance Tracks](https://react.dev/reference/dev-tools/react-performance-tracks) — DevTools 中一个新的性能分析工具。
* [Partial Pre-Rendering](https://react.dev/blog/2025/10/01/react-19-2#partial-pre-rendering) 用于提前预渲染应用的一部分，并在之后恢复渲染。

Jack Pope 宣布了 Canary 中的新特性，包括：

* [`<ViewTransition />`](https://react.dev/reference/react/ViewTransition) — 一个用于动画化页面过渡的新组件。
* [Fragment Refs](https://react.dev/reference/react/Fragment#fragmentinstance) — 一种与 Fragment 包裹的 DOM 节点交互的新方式。

Lauren Tan 发布了 [React Compiler v1.0](https://react.dev/blog/2025/10/07/react-compiler-1)，并建议所有应用都使用 React Compiler，以获得以下收益：
* [自动记忆化](https://react.dev/blog/2025/10/07/react-compiler-1) 能理解 React 代码。
* [新的 lint 规则](/learn/react-compiler/installation#eslint-integration) 由 React Compiler 驱动，用于传授最佳实践。
* [默认支持](/learn/react-compiler/installation#basic-setup) 适用于 Vite、Next.js 和 Expo 中的新应用。
* [迁移指南](/learn/react-compiler/incremental-adoption) 适用于迁移到 React Compiler 的现有应用。

最后，Seth Webster 宣布了 [React Foundation](/blog/2025/10/07/introducing-the-react-foundation)，以负责 React 开源开发和社区治理。

在这里观看第一天：

<YouTubeIframe src="https://www.youtube.com/embed/zyVRg2QR6LA?si=z-8t_xCc12HwGJH_&t=1067s" />

## 第二天主题演讲 {/*day-2-keynote*/}

_观看完整的第二天直播 [这里。](https://www.youtube.com/watch?v=p9OcztRyDl0&t=2299s)_

Jorge Cohen 和 Nicola Corti 以展示 React Native 的惊人增长作为第二天的开场：每周下载量达到 400 万次（同比增长 100%），以及来自 Shopify、Zalando 和 HelloFresh 的一些值得关注的应用迁移、RISE、RUNNA 和 Partyful 等获奖应用，以及来自 Mistral、Replit 和 v0 的 AI 应用。

Riccardo Cipolleschi 为 React Native 介绍了两项重大公告：
- [React Native 0.82 将仅支持 New Architecture](https://reactnative.dev/blog/2025/10/08/react-native-0.82#new-architecture-only)
- [Hermes V1 实验性支持](https://reactnative.dev/blog/2025/10/08/react-native-0.82#experimental-hermes-v1)

Ruben Norte 和 Alex Hunt 以以下公告结束了主题演讲：
- [新的与 Web 对齐的 DOM APIs](https://reactnative.dev/blog/2025/10/08/react-native-0.82#dom-node-apis)，以提升与 Web 上 React 的兼容性。
- [新的 Performance APIs](https://reactnative.dev/blog/2025/10/08/react-native-0.82#web-performance-apis-canary)，配备新的网络面板和桌面应用。

在这里观看第二天：

<YouTubeIframe src="https://www.youtube.com/embed/p9OcztRyDl0?si=qPTHftsUE07cjZpS&t=2299s" />


## React 团队演讲 {/*react-team-talks*/}

在整个大会期间，React 团队带来了以下演讲：
* [Async React Part I](https://www.youtube.com/watch?v=zyVRg2QR6LA&t=10907s) 和 [Part II](https://www.youtube.com/watch?v=p9OcztRyDl0&t=29073s) [(Ricky Hanlon)](https://x.com/rickhanlonii) 展示了利用过去 10 年创新成果所能实现的可能性。
* [Exploring React Performance](https://www.youtube.com/watch?v=zyVRg2QR6LA&t=20274s) [(Joe Savona)](https://x.com/en_js) 展示了我们关于 React 性能研究的结果。
* [Reimagining Lists in React Native](https://www.youtube.com/watch?v=p9OcztRyDl0&t=10382s) [(Luna Wei)](https://x.com/lunaleaps) 介绍了 Virtual View，这是一种新的列表原语，使用基于模式的渲染（hidden/pre-render/visible）来管理可见性。
* [Profiling with React Performance tracks](https://www.youtube.com/watch?v=zyVRg2QR6LA&t=8276s) [(Ruslan Lesiutin)](https://x.com/ruslanlesiutin) 展示了如何使用新的 React Performance Tracks 来调试性能问题并构建优秀应用。
* [React Strict DOM](https://www.youtube.com/watch?v=p9OcztRyDl0&t=9026s) [(Nicolas Gallagher)](https://nicolasgallagher.com/) 介绍了 Meta 在原生端使用 Web 代码的方法。
* [View Transitions and Activity](https://www.youtube.com/watch?v=zyVRg2QR6LA&t=4870s) [(Chance Strickland)](https://x.com/chancethedev) — Chance 与 React 团队合作，展示如何使用 `<Activity />` 和 `<ViewTransition />` 构建快速、具有原生体验的动画。
* [In case you missed the memo](https://www.youtube.com/watch?v=zyVRg2QR6LA&t=9525s) [(Cody Olsen)](https://bsky.app/profile/codey.bsky.social) - Cody 与 React 团队合作，在 Sanity Studio 中采用 Compiler，并分享了整个过程的体验。
## React 框架演讲 {/*react-framework-talks*/}

第二天后半段有一系列来自 React Framework 团队的演讲，包括：

* [React Native, Amplified](https://www.youtube.com/watch?v=p9OcztRyDl0&t=5737s) 由 [Giovanni Laquidara](https://x.com/giolaq) 和 [Eric Fahsl](https://x.com/efahsl) 主讲。
* [React Everywhere: Bringing React Into Native Apps](https://www.youtube.com/watch?v=p9OcztRyDl0&t=18213s) 由 [Mike Grabowski](https://x.com/grabbou) 主讲。
* [How Parcel Bundles React Server Components](https://www.youtube.com/watch?v=p9OcztRyDl0&t=19538s) 由 [Devon Govett](https://x.com/devonovett) 主讲。
* [Designing Page Transitions](https://www.youtube.com/watch?v=p9OcztRyDl0&t=20640s) 由 [Delba de Oliveira](https://x.com/delba_oliveira) 主讲。
* [Build Fast, Deploy Faster — Expo in 2025](https://www.youtube.com/watch?v=p9OcztRyDl0&t=21350s) 由 [Evan Bacon](https://x.com/baconbrix) 主讲。
* [The React Router's take on RSC](https://www.youtube.com/watch?v=p9OcztRyDl0&t=22367s) 由 [Kent C. Dodds](https://x.com/kentcdodds) 主讲。
* [RedwoodSDK: Web Standards Meet Full-Stack React](https://www.youtube.com/watch?v=p9OcztRyDl0&t=24992s) 由 [Peter Pistorius](https://x.com/appfactory) 和 [Aurora Scharff](https://x.com/aurorascharff) 主讲。
* [TanStack Start](https://www.youtube.com/watch?v=p9OcztRyDl0&t=26065s) 由 [Tanner Linsley](https://x.com/tannerlinsley) 主讲。

## 问答 {/*q-and-a*/}
大会期间共有三场问答环节：

* [Meta 的 React 团队问答](https://www.youtube.com/watch?v=zyVRg2QR6LA&t=26304s) 由 [Shruti Kapoor](https://x.com/shrutikapoor08) 主持
* [React Frameworks 问答](https://www.youtube.com/watch?v=p9OcztRyDl0&t=26812s) 由 [Jack Herrington](https://x.com/jherr) 主持
* [React 与 AI 圆桌](https://www.youtube.com/watch?v=zyVRg2QR6LA&t=18741s) 由 [Lee Robinson](https://x.com/leerob) 主持

## 以及更多... {/*and-more*/}

我们还听到了来自社区的演讲，包括：
* [构建一个 MCP 服务器](https://www.youtube.com/watch?v=zyVRg2QR6LA&t=24204s) 由 [James Swinton](https://x.com/JamesSwintonDev) 主讲 ([AG Grid](https://www.ag-grid.com/?utm_source=react-conf&utm_medium=react-conf-homepage&utm_campaign=react-conf-sponsorship-2025))
* [使用 React 制作现代电子邮件](https://www.youtube.com/watch?v=zyVRg2QR6LA&t=25521s) 由 [Zeno Rocha](https://x.com/zenorocha) 主讲 ([Resend](https://resend.com/))
* [为什么 React Native 应用最赚钱](https://www.youtube.com/watch?v=zyVRg2QR6LA&t=24917s) 由 [Perttu Lähteenlahti](https://x.com/plahteenlahti) 主讲 ([RevenueCat](https://www.revenuecat.com/))
* [优秀 UX 的隐形工艺](https://www.youtube.com/watch?v=zyVRg2QR6LA&t=23400s) 由 [Michał Dudak](https://x.com/michaldudak) 主讲 ([MUI](https://mui.com/))

## 致谢 {/*thanks*/}

感谢所有工作人员、演讲者和参与者，是你们让 React Conf 2025 成为可能。名单太长，无法一一列出，但我们特别想感谢以下几位。

感谢 [Matt Carroll](https://x.com/mattcarrollcode) 规划整个活动并搭建大会网站。

感谢 [Michael Chan](https://x.com/chantastic) 以极大的投入和活力担任 React Conf 的主持人，贯穿整个活动带来了深思熟虑的演讲者介绍、有趣的笑话以及真挚的热情。感谢 [Jorge Cohen](https://x.com/JorgeWritesCode) 主持直播、采访每位演讲者，并将线下 React Conf 体验带到线上。

感谢 [Mateusz Kornacki](https://x.com/mat_kornacki)、[Mike Grabowski](https://x.com/grabbou)、[Kris Lis](https://www.linkedin.com/in/krzysztoflisakakris/) 以及 [Callstack](https://www.callstack.com/) 团队共同组织 React Conf，并提供设计、工程和市场支持。感谢 [ZeroSlope 团队](https://zeroslopeevents.com/contact-us/)：Sunny Leggett、Tracey Harrison、Tara Larish、Whitney Pogue 和 Brianne Smythia 帮助组织此次活动。

感谢 [Jorge Cabiedes Acosta](https://github.com/jorge-cab)、[Gijs Weterings](https://x.com/gweterings)、[Tim Yung](https://x.com/yungsters) 和 [Jason Bonta](https://x.com/someextent) 将 Discord 中的问题带到直播中。感谢 [Lynn Yu](https://github.com/lynnshaoyu) 负责 Discord 的审核工作。感谢 [Seth Webster](https://x.com/sethwebster) 每天欢迎我们的到来；也感谢 [Christopher Chedeau](https://x.com/vjeux)、[Kevin Gozali](https://x.com/fkgozali) 和 [Pieter De Baets](https://x.com/Javache) 在 after-party 期间带来特别信息。

感谢 [Kadi Kraman](https://x.com/kadikraman)、[Beto](https://x.com/betomoedano) 和 [Nicolas Solerieu](https://www.linkedin.com/in/nicolas-solerieu/) 开发大会移动应用。感谢 [Wojtek Szafraniec](https://x.com/wojteg1337) 对大会网站提供帮助。感谢 [Mustache](https://www.mustachepower.com/) 和 [Cornerstone](https://cornerstoneav.com/) 提供视觉设计、舞台和音响支持；也感谢 Westin Hotel 的接待。

感谢所有让此次活动得以实现的赞助商：[Amazon](https://www.developer.amazon.com)、[MUI](https://mui.com/)、[Vercel](https://vercel.com/)、[Expo](https://expo.dev/)、[RedwoodSDK](https://rwsdk.com)、[Ag Grid](https://www.ag-grid.com)、[RevenueCat](https://www.revenuecat.com/)、[Resend](https://resend.com)、[Mux](https://www.mux.com/)、[Old Mission](https://www.oldmissioncapital.com/)、[Arcjet](https://arcjet.com)、[Infinite Red](https://infinite.red/)，以及 [RenderATL](https://renderatl.com)。

感谢所有与社区分享知识和经验的演讲者。

最后，感谢所有亲临现场和在线参与的人，向大家展示了 React 之所以是 React。React 不只是一个库，它还是一个社区，看到大家齐聚一堂共同分享和学习，令人深受鼓舞。

下次再见！
