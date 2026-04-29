---
title: "React Conf 2021 回顾"
author: Jesslyn Tannady and Rick Hanlon
date: 2021/12/17
description: 上周我们举办了第 6 届 React Conf。过去几年里，我们一直利用 React Conf 的舞台发布改变行业的公告，例如 React Native 和 React Hooks。今年，我们分享了 React 的多平台愿景，从发布 React 18 和逐步采用并发特性开始。
---

2021 年 12 月 17 日，由 [Jesslyn Tannady](https://twitter.com/jtannady) 和 [Rick Hanlon](https://twitter.com/rickhanlonii) 撰写

---

<Intro>

上周我们举办了第 6 届 React Conf。过去几年里，我们一直利用 React Conf 的舞台发布改变行业的公告，例如 [_React Native_](https://engineering.fb.com/2015/03/26/android/react-native-bringing-modern-web-techniques-to-mobile/) 和 [_React Hooks_](https://reactjs.org/docs/hooks-intro.html)。今年，我们分享了 React 的多平台愿景，从发布 React 18 和逐步采用并发特性开始。

</Intro>

---

这是 React Conf 首次在线举办，并且免费直播，翻译成了 8 种不同语言。来自世界各地的参与者加入了我们的会议 Discord 和回放活动，以便在所有时区都能无障碍参与。共有超过 50,000 人注册，19 场演讲的观看量超过 60,000 次，两个活动在 Discord 中共有 5,000 名参与者。

所有演讲都可在[线上观看](https://www.youtube.com/watch?v=FZ0cG47msEk&list=PLNG_1j3cPCaZZ7etkzWA7JfdmKWT0pMsa)。

以下是舞台上分享内容的摘要：

## React 18 和并发特性 {/*react-18-and-concurrent-features*/}

在主题演讲中，我们分享了 React 未来的愿景，从 React 18 开始。

React 18 加入了期待已久的并发渲染器，并对 Suspense 进行了更新，而且没有任何重大破坏性变更。应用可以升级到 React 18，并开始逐步采用并发特性，所需投入与其他任何主要版本更新相当。

**这意味着不存在并发模式，只有并发特性。**

在主题演讲中，我们还分享了我们对 Suspense、Server Components、新的 React 工作组，以及 React Native 长期多平台愿景的看法。

可在此观看 [Andrew Clark](https://twitter.com/acdlite)、[Juan Tejada](https://twitter.com/_jstejada)、[Lauren Tan](https://twitter.com/potetotes) 和 [Rick Hanlon](https://twitter.com/rickhanlonii) 的完整主题演讲：

<YouTubeIframe src="https://www.youtube.com/embed/FZ0cG47msEk" />

## 面向应用开发者的 React 18 {/*react-18-for-application-developers*/}

在主题演讲中，我们还宣布 React 18 RC 现已可供试用。在等待进一步反馈期间，这就是我们计划在明年年初发布到稳定版的 React 版本。

要试用 React 18 RC，请升级你的依赖：

```bash
npm install react@rc react-dom@rc
```

并切换到新的 `createRoot` API：

```js
// 之前
const container = document.getElementById('root');
ReactDOM.render(<App />, container);

// 之后
const container = document.getElementById('root');
const root = ReactDOM.createRoot(container);
root.render(<App/>);
```

如需观看升级到 React 18 的演示，请在此查看 [Shruti Kapoor](https://twitter.com/shrutikapoor08) 的演讲：

<YouTubeIframe src="https://www.youtube.com/embed/ytudH8je5ko" />

## 使用 Suspense 进行流式服务端渲染 {/*streaming-server-rendering-with-suspense*/}

React 18 还包括使用 Suspense 改进服务端渲染性能的功能。

流式服务端渲染可以让你在服务器上从 React 组件生成 HTML，并将该 HTML 流式传输给用户。在 React 18 中，你可以使用 `Suspense` 将应用拆分为更小的独立单元，这些单元可以彼此独立地流式传输，而不会阻塞应用的其余部分。这意味着用户会更早看到你的内容，并且能够更快开始与之交互。

如需深入了解，请在此查看 [Shaundai Person](https://twitter.com/shaundai) 的演讲：

<YouTubeIframe src="https://www.youtube.com/embed/pj5N-Khihgc" />

## 第一个 React 工作组 {/*the-first-react-working-group*/}

为了 React 18，我们创建了第一个工作组，与专家、开发者、库维护者和教育者小组协作。我们一起制定了逐步采用策略，并完善了诸如 `useId`、`useSyncExternalStore` 和 `useInsertionEffect` 等新 API。

如需查看这项工作的概述，请观看 [Aakansha' Doshi](https://twitter.com/aakansha1216) 的演讲：

<YouTubeIframe src="https://www.youtube.com/embed/qn7gRClrC9U" />

## React 开发者工具 {/*react-developer-tooling*/}

为了支持本次发布中的新特性，我们还宣布了新成立的 React DevTools 团队，以及一个新的时间线分析器，帮助开发者调试他们的 React 应用。

如需了解更多信息并观看新 DevTools 功能演示，请查看 [Brian Vaughn](https://twitter.com/brian_d_vaughn) 的演讲：

<YouTubeIframe src="https://www.youtube.com/embed/oxDfrke8rZg" />

## 不使用 memo 的 React {/*react-without-memo*/}

展望更远的未来，[Xuan Huang (黄玄)](https://twitter.com/Huxpro) 分享了来自 React Labs 关于自动 memo 化编译器研究的最新进展。查看这场演讲，了解更多信息并观看编译器原型演示：

<YouTubeIframe src="https://www.youtube.com/embed/lGEMwh32soc" />

## React 文档主题演讲 {/*react-docs-keynote*/}

[Rachel Nabors](https://twitter.com/rachelnabors) 以一场关于我们对 React 新文档投入的主题演讲拉开了关于使用 React 学习与设计的一系列演讲的序幕（[现已作为 react.dev 发布](/blog/2023/03/16/introducing-react-dev)）：

<YouTubeIframe src="https://www.youtube.com/embed/mneDaMYOKP8" />

## 以及更多... {/*and-more*/}

**我们还听到了关于使用 React 学习与设计的演讲：**

* Debbie O'Brien: [我从新的 React 文档中学到的东西](https://youtu.be/-7odLW_hG7s)。
* Sarah Rainsberger: [在浏览器中学习](https://youtu.be/5X-WEQflCL0)。
* Linton Ye: [使用 React 进行设计的投资回报率](https://youtu.be/7cPWmID5XAk)。
* Delba de Oliveira: [使用 React 的交互式游乐场](https://youtu.be/zL8cz2W0z34)。

**来自 Relay、React Native 和 PyTorch 团队的演讲：**

* Robert Balicki: [重新介绍 Relay](https://youtu.be/lhVGdErZuN4)。
* Eric Rozell 和 Steven Moyes: [React Native Desktop](https://youtu.be/9L4FFrvwJwY)。
* Roman Rädle: [面向 React Native 的端侧机器学习](https://youtu.be/NLj73vrc2I8)

**以及来自社区关于无障碍、工具和 Server Components 的演讲：**

* Daishi Kato: [面向外部存储库的 React 18](https://youtu.be/oPfSC5bQPR8)。
* Diego Haz: [在 React 18 中构建无障碍组件](https://youtu.be/dcm8fjBfro8)。
* Tafu Nakazaki: [使用 React 构建可访问的日文表单组件](https://youtu.be/S4a0QlsH0pU)。
* Lyle Troxell: [面向艺术家的 UI 工具](https://youtu.be/b3l4WxipFsE)。
* Helen Lin: [Hydrogen + React 18](https://youtu.be/HS6vIYkSNks)。

## 感谢 {/*thank-you*/}

这是我们首次自行策划会议，我们有很多人需要感谢。

首先，感谢所有演讲者 [Aakansha Doshi](https://twitter.com/aakansha1216)、[Andrew Clark](https://twitter.com/acdlite)、[Brian Vaughn](https://twitter.com/brian_d_vaughn)、[Daishi Kato](https://twitter.com/dai_shi)、[Debbie O'Brien](https://twitter.com/debs_obrien)、[Delba de Oliveira](https://twitter.com/delba_oliveira)、[Diego Haz](https://twitter.com/diegohaz)、[Eric Rozell](https://twitter.com/EricRozell)、[Helen Lin](https://twitter.com/wizardlyhel)、[Juan Tejada](https://twitter.com/_jstejada)、[Lauren Tan](https://twitter.com/potetotes)、[Linton Ye](https://twitter.com/lintonye)、[Lyle Troxell](https://twitter.com/lyle)、[Rachel Nabors](https://twitter.com/rachelnabors)、[Rick Hanlon](https://twitter.com/rickhanlonii)、[Robert Balicki](https://twitter.com/StatisticsFTW)、[Roman Rädle](https://twitter.com/raedle)、[Sarah Rainsberger](https://twitter.com/sarah11918)、[Shaundai Person](https://twitter.com/shaundai)、[Shruti Kapoor](https://twitter.com/shrutikapoor08)、[Steven Moyes](https://twitter.com/moyessa)、[Tafu Nakazaki](https://twitter.com/hawaiiman0)，以及 [Xuan Huang (黄玄)](https://twitter.com/Huxpro)。

感谢所有帮助提供演讲反馈的人，包括 [Andrew Clark](https://twitter.com/acdlite)、[Dan Abramov](https://bsky.app/profile/danabra.mov)、[Dave McCabe](https://twitter.com/mcc_abe)、[Eli White](https://twitter.com/Eli_White)、[Joe Savona](https://twitter.com/en_JS)、[Lauren Tan](https://twitter.com/potetotes)、[Rachel Nabors](https://twitter.com/rachelnabors) 和 [Tim Yung](https://twitter.com/yungsters)。

感谢 [Lauren Tan](https://twitter.com/potetotes) 搭建会议 Discord 并担任我们的 Discord 管理员。

感谢 [Seth Webster](https://twitter.com/sethwebster) 对整体方向的反馈，以及确保我们专注于多样性和包容性。

感谢 [Rachel Nabors](https://twitter.com/rachelnabors) 牵头我们的审核工作，以及 [Aisha Blake](https://twitter.com/AishaBlake) 制定审核指南、领导我们的审核团队、培训翻译和审核员，并帮助主持这两场活动。

感谢我们的审核员 [Jesslyn Tannady](https://twitter.com/jtannady)、[Suzie Grange](https://twitter.com/missuze)、[Becca Bailey](https://twitter.com/beccaliz)、[Luna Wei](https://twitter.com/lunaleaps)、[Joe Previte](https://twitter.com/jsjoeio)、[Nicola Corti](https://twitter.com/Cortinico)、[Gijs Weterings](https://twitter.com/gweterings)、[Claudio Procida](https://twitter.com/claudiopro)、Julia Neumann、Mengdi Chen、Jean Zhang、Ricky Li，以及 [Xuan Huang (黄玄)](https://twitter.com/Huxpro)。

感谢来自 [React India](https://www.reactindia.io/) 的 [Manjula Dube](https://twitter.com/manjula_dube)、[Sahil Mhapsekar](https://twitter.com/apheri0) 和 Vihang Patel，以及来自 [React China](https://twitter.com/ReactChina) 的 [Jasmine Xie](https://twitter.com/jasmine_xby)、[QiChang Li](https://twitter.com/QCL15) 和 [YanLun Li](https://twitter.com/anneincoding)，感谢你们帮助主持我们的回放活动，并让它持续对社区保持吸引力。

感谢 Vercel 发布了他们的 [Virtual Event Starter Kit](https://vercel.com/virtual-event-starter-kit)，会议网站就是基于它构建的；也感谢 [Lee Robinson](https://twitter.com/leeerob) 和 [Delba de Oliveira](https://twitter.com/delba_oliveira) 分享他们举办 Next.js Conf 的经验。

感谢 [Leah Silber](https://twitter.com/wifelette) 分享她举办会议的经验、举办 [RustConf](https://rustconf.com/) 的心得，以及她的书 [Event Driven](https://leanpub.com/eventdriven/) 和其中关于举办会议的建议。

感谢 [Kevin Lewis](https://twitter.com/_phzn) 和 [Rachel Nabors](https://twitter.com/rachelnabors) 分享她们举办 Women of React Conf 的经验。

感谢 [Aakansha Doshi](https://twitter.com/aakansha1216)、[Laurie Barth](https://twitter.com/laurieontech)、[Michael Chan](https://twitter.com/chantastic) 和 [Shaundai Person](https://twitter.com/shaundai) 在整个筹备过程中提供的建议和想法。

感谢 [Dan Lebowitz](https://twitter.com/lebo) 帮助设计和构建会议网站与门票系统。

感谢 Facebook Video Productions 团队的 Laura Podolak Waddell、Desmond Osei-Acheampong、Mark Rossi、Josh Toberman 以及其他成员为主题演讲和 Meta 员工演讲录制视频。

感谢我们的合作伙伴 HitPlay 帮助组织会议、编辑直播中的所有视频、翻译所有演讲，并以多种语言管理 Discord。

最后，感谢所有参与者，让这届 React Conf 如此精彩！
