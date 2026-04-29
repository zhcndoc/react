---
title: 从零开始构建 React 应用
---

<Intro>

如果你的应用有现有框架难以满足的约束，或者你更希望构建自己的框架，又或者你只是想了解 React 应用的基础知识，那么你可以从零开始构建一个 React 应用。

</Intro>

<DeepDive>

#### 考虑使用框架 {/*consider-using-a-framework*/}

从零开始是上手 React 的一种简单方式，但需要注意的一个主要取舍是，走这条路通常等同于构建你自己的临时框架。随着需求不断演进，你可能需要解决更多类似框架的问题，而我们推荐的框架已经为这些问题提供了完善且受支持的解决方案。

例如，如果未来你的应用需要支持服务端渲染（SSR）、静态站点生成（SSG）和/或 React 服务端组件（RSC），你就必须自己实现这些功能。同样，未来需要在框架层面集成的 React 新特性，如果你想使用，也必须自行实现。

我们推荐的框架还能帮助你构建性能更好的应用。例如，减少或消除网络请求瀑布流可以带来更好的用户体验。对于玩具项目来说，这可能不是首要任务，但如果你的应用开始有用户，你可能会希望优化它的性能。

走这条路也会让获取支持变得更困难，因为你开发路由、数据获取以及其他功能的方式会因你的情况而独一无二。只有在你愿意自己解决这些问题，或者确信永远不需要这些功能时，才应该选择这种方案。

有关推荐框架的列表，请查看[创建一个 React 应用](/learn/creating-a-react-app)。

</DeepDive>


## 第 1 步：安装构建工具 {/*step-1-install-a-build-tool*/}

第一步是安装像 `vite`、`parcel` 或 `rsbuild` 这样的构建工具。这些构建工具提供了打包和运行源代码的功能，提供用于本地开发的开发服务器，以及将应用部署到生产服务器的构建命令。

### Vite {/*vite*/}

[Vite](https://vite.dev/) 是一个构建工具，旨在为现代 Web 项目提供更快、更轻量的开发体验。

<TerminalBlock>
npm create vite@latest my-app -- --template react-ts
</TerminalBlock>

Vite 具有明确的设计取向，并开箱即用地提供了合理的默认配置。Vite 拥有丰富的插件生态，可支持快速刷新、JSX、Babel/SWC 以及其他常见功能。可查看 Vite 的 [React 插件](https://vite.dev/plugins/#vitejs-plugin-react) 或 [React SWC 插件](https://vite.dev/plugins/#vitejs-plugin-react-swc) 以及 [React SSR 示例项目](https://vite.dev/guide/ssr.html#example-projects) 以开始使用。

Vite 已经作为构建工具被用于我们某个[推荐框架](/learn/creating-a-react-app)中：[React Router](https://reactrouter.com/start/framework/installation)。

### Parcel {/*parcel*/}

[Parcel](https://parceljs.org/) 将出色的开箱即用开发体验与可扩展架构结合在一起，能够让你的项目从刚开始使用一直扩展到大规模生产应用。

<TerminalBlock>
npm install --save-dev parcel
</TerminalBlock>

Parcel 原生支持快速刷新、JSX、TypeScript、Flow 和样式。可查看 [Parcel 的 React 配方](https://parceljs.org/recipes/react/#getting-started) 以开始使用。

### Rsbuild {/*rsbuild*/}

[Rsbuild](https://rsbuild.dev/) 是一款由 Rspack 驱动的构建工具，为 React 应用提供流畅的开发体验。它带有经过精心调优的默认配置和可直接使用的性能优化。

<TerminalBlock>
npx create-rsbuild --template react
</TerminalBlock>

Rsbuild 内置支持 React 特性，例如快速刷新、JSX、TypeScript 和样式。可查看 [Rsbuild 的 React 指南](https://rsbuild.dev/guide/framework/react) 以开始使用。

<Note>

#### React Native 的 Metro {/*react-native*/}

如果你正从零开始使用 React Native，你需要使用 [Metro](https://metrobundler.dev/)，它是 React Native 的 JavaScript 打包器。Metro 支持为 iOS 和 Android 等平台进行打包，但与这里提到的工具相比缺少许多功能。除非你的项目需要 React Native 支持，否则我们建议从 Vite、Parcel 或 Rsbuild 开始。

</Note>

## 第 2 步：构建常见应用模式 {/*step-2-build-common-application-patterns*/}

上面列出的构建工具一开始都是仅客户端、单页应用（SPA），但并未针对路由、数据获取或样式等常见功能提供进一步的解决方案。

React 生态中包含许多用于解决这些问题的工具。我们列出了一些广泛使用的工具作为起点，但如果其他工具更适合你，也完全可以选择它们。

### 路由 {/*routing*/}

路由决定用户访问特定 URL 时显示哪些内容或页面。你需要设置一个路由器，将 URL 映射到应用的不同部分。你还需要处理嵌套路由、路由参数和查询参数。路由器可以在代码中配置，也可以基于组件文件夹和文件结构来定义。

路由器是现代应用的核心部分，通常会与数据获取集成（包括为整个页面预取数据以加快加载速度）、代码拆分（以最小化客户端包大小）以及页面渲染方式（用来决定每个页面如何生成）相结合。

我们建议使用：

- [React Router](https://reactrouter.com/start/data/custom)
- [Tanstack Router](https://tanstack.com/router/latest)


### 数据获取 {/*data-fetching*/}

从服务器或其他数据源获取数据是大多数应用的关键部分。要正确完成这件事，需要处理加载状态、错误状态以及对获取到的数据进行缓存，这可能相当复杂。

专为此打造的数据获取库会替你完成获取和缓存数据的繁重工作，让你专注于应用需要哪些数据以及如何展示它们。这些库通常直接在组件中使用，但也可以集成到路由加载器中，以实现更快的预取和更好的性能，在服务端渲染中也同样适用。

请注意，直接在组件中获取数据可能会因为网络请求瀑布流而导致更慢的加载时间，因此我们建议尽可能在路由加载器或服务端进行数据预取！这使得页面数据可以在页面显示时一次性全部获取。

如果你正在从大多数后端或 REST 风格 API 获取数据，我们建议使用：

- [TanStack Query](https://tanstack.com/query/)
- [SWR](https://swr.vercel.app/)
- [RTK Query](https://redux-toolkit.js.org/rtk-query/overview)

如果你正在从 GraphQL API 获取数据，我们建议使用：

- [Apollo](https://www.apollographql.com/docs/react)
- [Relay](https://relay.dev/)


### 代码拆分 {/*code-splitting*/}

代码拆分是将应用拆分成更小的包，并按需加载的过程。随着每个新功能和额外依赖的加入，应用的代码体积都会增加。由于在可使用之前需要先传输整个应用的全部代码，应用可能会变得加载缓慢。缓存、减少功能/依赖，以及将部分代码迁移到服务端执行，都有助于缓解加载缓慢的问题，但这些都不是完整的解决方案，而且如果过度使用，还可能牺牲功能性。

类似地，如果你依赖使用你框架的应用自己进行代码拆分，你可能会遇到这样一种情况：加载速度反而比完全不进行代码拆分时更慢。例如，[延迟加载](/reference/react/lazy) 一个图表会延迟发送渲染该图表所需的代码，把图表代码与应用其余部分分离开来。[Parcel 支持与 React.lazy 一起进行代码拆分](https://parceljs.org/recipes/react/#code-splitting)。不过，如果图表在初次渲染之后才加载其数据，那么你现在就要等待两次。这就是瀑布流：你不是同时获取图表数据并发送其渲染代码，而是必须等待每一步按顺序一个接一个完成。

当按路由拆分代码并与打包和数据获取结合时，可以减少应用的初始加载时间，以及应用中最大可见内容的渲染时间（[最大内容绘制](https://web.dev/articles/lcp)）。

有关代码拆分的说明，请参阅你的构建工具文档：
- [Vite 构建优化](https://vite.dev/guide/features.html#build-optimizations)
- [Parcel 代码拆分](https://parceljs.org/features/code-splitting/)
- [Rsbuild 代码拆分](https://rsbuild.dev/guide/optimization/code-splitting)

### 提升应用性能 {/*improving-application-performance*/}

由于你选择的构建工具只支持单页应用（SPA），你还需要实现其他[渲染模式](https://www.patterns.dev/vanilla/rendering-patterns)，例如服务端渲染（SSR）、静态站点生成（SSG）和/或 React 服务端组件（RSC）。即使你一开始不需要这些功能，未来也可能有某些路由会从 SSR、SSG 或 RSC 中受益。

* **单页应用（SPA）** 加载单个 HTML 页面，并在用户与应用交互时动态更新页面。SPA 更容易上手，但初始加载时间可能更慢。SPA 是大多数构建工具的默认架构。

* **流式服务端渲染（SSR）** 在服务端渲染页面并将完整渲染后的页面发送给客户端。SSR 可以提升性能，但与单页应用相比，配置和维护可能更复杂。加入流式传输后，SSR 的配置和维护会变得非常复杂。请参阅 [Vite 的 SSR 指南]( https://vite.dev/guide/ssr)。

* **静态站点生成（SSG）** 在构建时为你的应用生成静态 HTML 文件。SSG 可以提升性能，但与服务端渲染相比，配置和维护可能更复杂。请参阅 [Vite 的 SSG 指南](https://vite.dev/guide/ssr.html#pre-rendering-ssg)。

* **React 服务端组件（RSC）** 让你可以在单个 React 树中混合构建时组件、仅服务端组件和交互式组件。RSC 可以提升性能，但目前配置和维护需要相当深入的专业知识。请参阅 [Parcel 的 RSC 示例](https://github.com/parcel-bundler/rsc-examples)。

你的渲染策略需要与你的路由器集成，这样用你的框架构建的应用就可以按路由级别选择渲染策略。这将支持不同的渲染策略，而无需重写整个应用。例如，你的应用落地页可能更适合静态生成（SSG），而带有内容信息流的页面可能在服务端渲染下表现最佳。

为合适的路由使用合适的渲染策略，可以减少内容首字节到达时间（[首字节时间](https://web.dev/articles/ttfb)）、首个内容绘制时间（[首次内容绘制](https://web.dev/articles/fcp)）以及应用中最大可见内容的渲染时间（[最大内容绘制](https://web.dev/articles/lcp)）。

### 以及更多... {/*and-more*/}

这些只是从零开始构建新应用时需要考虑的功能中的几个例子。你会遇到的许多限制都很难解决，因为每个问题都与其他问题相互关联，并且可能需要你并不熟悉的问题领域中的深厚专业知识。

如果你不想自己解决这些问题，可以使用一个[框架开始上手](/learn/creating-a-react-app)，它会开箱即用地提供这些功能。
