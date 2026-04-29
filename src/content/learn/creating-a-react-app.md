---
title: 创建 React 应用
---

<Intro>

如果你想使用 React 构建一个新的应用或网站，我们建议从框架开始。

</Intro>

如果你的应用有现有框架无法很好满足的约束，或者你更倾向于构建自己的框架，或者你只是想了解 React 应用的基础，那么你可以[从零开始构建一个 React 应用](/learn/build-a-react-app-from-scratch)。

## 全栈框架 {/*full-stack-frameworks*/}

这些推荐的框架支持你在生产环境中部署和扩展应用所需的全部功能。它们集成了最新的 React 特性，并充分利用了 React 的架构。

<Note>

#### 全栈框架不需要服务器。 {/*react-frameworks-do-not-require-a-server*/}

本页列出的所有框架都支持客户端渲染（[CSR](https://developer.mozilla.org/en-US/docs/Glossary/CSR)）、单页应用（[SPA](https://developer.mozilla.org/en-US/docs/Glossary/SPA)）和静态站点生成（[SSG](https://developer.mozilla.org/en-US/docs/Glossary/SSG)）。这些应用可以部署到 [CDN](https://developer.mozilla.org/en-US/docs/Glossary/CDN) 或静态托管服务，而不需要服务器。此外，这些框架还允许你在有必要时，按路由启用服务器端渲染。

这使你可以先从仅客户端的应用开始，如果以后需求发生变化，也可以针对单独路由按需启用服务器功能，而无需重写应用。有关配置渲染策略，请参阅你所用框架的文档。

</Note>

### Next.js（App Router） {/*nextjs-app-router*/}

**[Next.js 的 App Router](https://nextjs.org/docs) 是一个 React 框架，它充分利用 React 的架构来支持全栈 React 应用。**

<TerminalBlock>
npx create-next-app@latest
</TerminalBlock>

Next.js 由 [Vercel](https://vercel.com/) 维护。你可以将 [Next.js 应用部署](https://nextjs.org/docs/app/building-your-application/deploying)到任何支持 Node.js 或 Docker 容器的托管服务，也可以部署到你自己的服务器上。Next.js 还支持 [静态导出](https://nextjs.org/docs/app/building-your-application/deploying/static-exports)，不需要服务器。

### React Router（v7） {/*react-router-v7*/}

**[React Router](https://reactrouter.com/start/framework/installation) 是 React 最受欢迎的路由库，可以与 Vite 搭配创建一个全栈 React 框架**。它强调标准 Web API，并提供了多个适用于不同 JavaScript 运行时和平台的[可直接部署模板](https://github.com/remix-run/react-router-templates)。

要创建一个新的 React Router 框架项目，请运行：

<TerminalBlock>
npx create-react-router@latest
</TerminalBlock>

React Router 由 [Shopify](https://www.shopify.com) 维护。

### Expo（用于原生应用） {/*expo*/}

**[Expo](https://expo.dev/) 是一个 React 框架，让你能够使用真正原生的 UI 创建适用于 Android、iOS 和 Web 的通用应用。** 它为 [React Native](https://reactnative.dev/) 提供了一个 SDK，使原生部分更易于使用。要创建一个新的 Expo 项目，请运行：

<TerminalBlock>
npx create-expo-app@latest
</TerminalBlock>

如果你刚接触 Expo，请查看 [Expo 教程](https://docs.expo.dev/tutorial/introduction/)。

Expo 由 [Expo（公司）](https://expo.dev/about) 维护。使用 Expo 构建应用是免费的，你可以将其提交到 Google 和 Apple 应用商店，不受限制。Expo 还提供可选的付费云服务。


## 其他框架 {/*other-frameworks*/}

还有一些正在发展中的框架，正朝着我们的全栈 React 愿景迈进：

- [TanStack Start（Beta）](https://tanstack.com/start/)：TanStack Start 是一个由 TanStack Router 驱动的全栈 React 框架。它使用 Nitro 和 Vite 等工具，提供完整文档 SSR、流式传输、服务器函数、打包等功能。
- [RedwoodSDK](https://rwsdk.com/)：Redwood 是一个全栈 React 框架，预装了大量包和配置，让构建全栈 Web 应用变得更容易。

<DeepDive>

#### 哪些特性构成了 React 团队的全栈架构愿景？ {/*which-features-make-up-the-react-teams-full-stack-architecture-vision*/}

Next.js 的 App Router 打包器完全实现了官方的 [React Server Components 规范](https://github.com/reactjs/rfcs/blob/main/text/0188-server-components.md)。这使你可以在同一棵 React 树中混合构建时组件、仅服务器组件和交互式组件。

例如，你可以编写一个仅服务器端的 React 组件，它是一个 `async` 函数，从数据库或文件中读取数据。然后你可以把数据传递给你的交互式组件：

```js
// 这个组件*仅*在服务器上运行（或在构建期间运行）。
async function Talks({ confId }) {
  // 1. 你在服务器上，因此可以访问数据层。不需要 API 端点。
  const talks = await db.Talks.findAll({ confId });

  // 2. 添加任意数量的渲染逻辑。它不会让你的 JavaScript 包变得更大。
  const videos = talks.map(talk => talk.video);

  // 3. 将数据传递给会在浏览器中运行的组件。
  return <SearchableVideoList videos={videos} />;
}
```

Next.js 的 App Router 还集成了 [将数据获取与 Suspense 结合](/blog/2022/03/29/react-v18#suspense-in-data-frameworks)。这使你可以直接在 React 树中为用户界面的不同部分指定加载状态（例如骨架屏占位符）：

```js
<Suspense fallback={<TalksLoading />}>
  <Talks confId={conf.id} />
</Suspense>
```

Server Components 和 Suspense 是 React 的特性，而不是 Next.js 的特性。不过，在框架层面采用它们需要认可并付出不小的实现工作。目前，Next.js App Router 是最完整的实现。React 团队正在与打包器开发者合作，让这些特性在下一代框架中更容易实现。

</DeepDive>

## 从零开始 {/*start-from-scratch*/}

如果你的应用有现有框架无法很好满足的约束，或者你更倾向于构建自己的框架，或者你只是想了解 React 应用的基础，那么也有其他可用于从零开始启动 React 项目的选项。

从零开始会给你更大的灵活性，但也确实要求你自己决定用于路由、数据获取以及其他常见使用模式的工具。它很像自己构建一个框架，而不是使用一个已经存在的框架。我们[推荐的框架](#full-stack-frameworks)已经内置了解决这些问题的方案。

如果你想构建自己的解决方案，请参阅我们的[从零开始构建一个 React 应用](/learn/build-a-react-app-from-scratch)指南，了解如何从一个构建工具开始设置一个新的 React 项目，例如 [Vite](https://vite.dev/)、[Parcel](https://parceljs.org/) 或 [RSbuild](https://rsbuild.dev/)。

-----

_如果你是框架作者，并希望你的框架被收录在此页面中，[请告诉我们](https://github.com/reactjs/react.dev/issues/new?assignees=&labels=type%3A+framework&projects=&template=3-framework.yml&title=%5BFramework%5D%3A+)。_
