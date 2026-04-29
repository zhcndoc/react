---
title: "告别 Create React App"
author: Matt Carroll and Ricky Hanlon
date: 2025/02/14
description: 今天，我们将对新的应用弃用 Create React App，并鼓励现有应用迁移到框架，或者迁移到像 Vite、Parcel 或 RSBuild 这样的构建工具。我们也提供了文档，帮助那些觉得框架不适合项目、想要构建自己的框架，或者只是想通过从零构建一个 React 应用来了解 React 的工作原理的人。
---

2025 年 2 月 14 日，作者 [Matt Carroll](https://twitter.com/mattcarrollcode) 和 [Ricky Hanlon](https://bsky.app/profile/ricky.fm)

---

<Intro>

今天，我们将对新的应用弃用 [Create React App](https://create-react-app.dev/)，并鼓励现有应用迁移到 [框架](#how-to-migrate-to-a-framework)，或者迁移到像 Vite、Parcel 或 RSBuild 这样的 [构建工具](#how-to-migrate-to-a-build-tool)。

我们也提供了文档，帮助那些觉得框架不适合项目、想要构建自己的框架，或者只是想通过 [从零构建一个 React 应用](/learn/build-a-react-app-from-scratch) 来了解 React 的工作原理的人。

</Intro>

-----

当我们在 2016 年发布 Create React App 时，还没有一种清晰的方式来构建一个新的 React 应用。

要创建一个 React 应用，你必须安装一堆工具，并亲手把它们连接起来，以支持 JSX、lint 检查和热重载等基础功能。这非常难以正确完成，所以 [社区](https://github.com/react-boilerplate/react-boilerplate) 为许多 [常见](https://github.com/gaearon/react-hot-boilerplate) [配置](https://github.com/erikras/react-redux-universal-hot-example) 创建了 [样板代码](https://github.com/petehunt/react-boilerplate) [示例](https://github.com/kriasoft/react-starter-kit)。然而，这些样板代码很难更新，而且碎片化使得 React 很难发布新功能。

Create React App 通过将多种工具组合成一个单一的推荐配置，解决了这些问题。这让应用可以用一种简单的方式升级到新的工具特性，也让 React 团队能够把一些并不简单的工具链变更（如 Fast Refresh 支持、React Hooks lint 规则）推送给尽可能广泛的用户。

这种模式变得如此流行，以至于今天已经出现了一个完整的工具类别都在这样工作。

## 弃用 Create React App {/*deprecating-create-react-app*/}

尽管 Create React App 让上手变得很容易，但它有 [一些限制](#limitations-of-build-tools)，使得构建高性能生产应用变得困难。原则上，我们本可以通过把它逐步演进成一个 [框架](#why-we-recommend-frameworks) 来解决这些问题。

不过，由于 Create React App 目前没有活跃的维护者，而且已经有很多现成的框架能够解决这些问题，我们决定弃用 Create React App。

从今天开始，如果你安装一个新的应用，你会看到一条弃用警告：

<ConsoleBlockMulti>
<ConsoleLogLine level="error">

create-react-app 已弃用。
{'\n\n'}
你可以在 react.dev 上找到最新的 React 框架列表
更多信息请参见：react.dev/link/cra
{'\n\n'}
此错误信息每次安装只会显示一次。

</ConsoleLogLine>
</ConsoleBlockMulti>

我们也已经在 Create React App 的 [网站](https://create-react-app.dev/) 和 GitHub [仓库](https://github.com/facebook/create-react-app) 中添加了弃用提示。Create React App 将继续以维护模式运行，我们还发布了一个支持 React 19 的 Create React App 新版本。

## 如何迁移到框架 {/*how-to-migrate-to-a-framework*/}
我们推荐使用框架来 [创建新的 React 应用](/learn/creating-a-react-app)。我们推荐的所有框架都支持客户端渲染（[CSR](https://developer.mozilla.org/en-US/docs/Glossary/CSR)）和单页应用（[SPA](https://developer.mozilla.org/en-US/docs/Glossary/SPA)），并且可以部署到 CDN 或静态托管服务上，而不需要服务器。

对于现有应用，这些指南可以帮助你迁移到仅客户端的 SPA：

* [Next.js 的 Create React App 迁移指南](https://nextjs.org/docs/app/building-your-application/upgrading/from-create-react-app)
* [React Router 的框架采用指南](https://reactrouter.com/upgrading/component-routes).
* [Expo webpack 到 Expo Router 的迁移指南](https://docs.expo.dev/router/migrate/from-expo-webpack/)

## 如何迁移到构建工具 {/*how-to-migrate-to-a-build-tool*/}

如果你的应用有一些特殊限制，或者你更愿意通过构建自己的框架来解决这些问题，或者你只是想从零学习 React 是如何工作的，那么你可以使用 Vite、Parcel 或 Rsbuild 和 React 搭建自己的自定义方案。

对于现有应用，这些指南可以帮助你迁移到构建工具：

* [Vite Create React App 迁移指南](https://www.robinwieruch.de/vite-create-react-app/)
* [Parcel Create React App 迁移指南](https://parceljs.org/migration/cra/)
* [Rsbuild Create React App 迁移指南](https://rsbuild.dev/guide/migration/cra)

为了帮助你开始使用 Vite、Parcel 或 Rsbuild，我们新增了关于 [从零构建一个 React 应用](/learn/build-a-react-app-from-scratch) 的文档。

<DeepDive>

#### 我需要框架吗？ {/*do-i-need-a-framework*/}

大多数应用都会从框架中受益，但也有一些合理的场景适合从零构建 React 应用。一个经验法则是：如果你的应用需要路由，那么你大概率会从框架中受益。

就像 Svelte 有 Sveltekit、Vue 有 Nuxt、Solid 有 SolidStart 一样，[React 推荐使用框架](#why-we-recommend-frameworks)，因为框架可以开箱即用地把路由与数据获取、代码分割等功能完全整合起来。这样可以避免你自己编写复杂配置、实际上相当于自己再造一个框架的痛苦。

不过，你始终可以使用 Vite、Parcel 或 Rsbuild 这样的构建工具来 [从零构建一个 React 应用](/learn/build-a-react-app-from-scratch)。

</DeepDive>

继续阅读，了解更多关于 [构建工具的限制](#limitations-of-build-tools) 以及 [我们为什么推荐框架](#why-we-recommend-frameworks)。

## 构建工具的限制 {/*limitations-of-build-tools*/}

Create React App 和类似的构建工具让开始构建 React 应用变得很容易。在运行 `npx create-react-app my-app` 之后，你会得到一个完整配置好的 React 应用，包含开发服务器、lint 检查和生产构建。

例如，如果你正在构建一个内部管理工具，你可以先从一个落地页开始：

```js
export default function App() {
  return (
    <div>
      <h1>欢迎使用管理工具！</h1>
    </div>
  )
}
```

这让你可以立即开始使用 React 编码，并获得 JSX、默认 lint 规则以及可在开发和生产环境中运行的打包器等功能。然而，这种配置缺少构建真正生产级应用所需的工具。

大多数生产应用都需要针对路由、数据获取和代码分割等问题的解决方案。

### 路由 {/*routing*/}

Create React App 不包含专门的路由解决方案。如果你刚开始上手，一种选择是使用 `useState` 来在不同路由之间切换。但这样做意味着你无法分享指向应用的链接——每个链接都会跳到同一页——而且随着时间推移，你的应用结构也会变得难以组织：

```js
import {useState} from 'react';

import Home from './Home';
import Dashboard from './Dashboard';

export default function App() {
  // ❌ 将路由放在状态中不会创建 URL
  const [route, setRoute] = useState('home');
  return (
    <div>
      {route === 'home' && <Home />}
      {route === 'dashboard' && <Dashboard />}
    </div>
  )
}
```

这就是为什么大多数使用 Create React App 的应用会借助像 [React Router](https://reactrouter.com/) 或 [Tanstack Router](https://tanstack.com/router/latest) 这样的路由库来添加路由。使用路由库，你可以为应用添加额外路由，这会对应用结构提供一些约束，并允许你开始分享指向各个路由的链接。例如，使用 React Router，你可以定义路由：

```js
import {RouterProvider, createBrowserRouter} from 'react-router';

import Home from './Home';
import Dashboard from './Dashboard';

// ✅ 每个路由都有自己的 URL
const router = createBrowserRouter([
  {path: '/', element: <Home />},
  {path: '/dashboard', element: <Dashboard />}
]);

export default function App() {
  return (
    <RouterProvider value={router} />
  )
}
```

通过这个改动，你可以分享一个指向 `/dashboard` 的链接，应用会跳转到仪表盘页面。一旦你有了路由库，就可以添加嵌套路由、路由守卫和路由过渡等额外功能，而这些在没有路由库的情况下很难实现。

这里存在一种权衡：路由库增加了应用的复杂度，但它也带来了那些离开它就很难实现的功能。

### 数据获取 {/*data-fetching*/}

Create React App 中另一个常见问题是数据获取。Create React App 不包含专门的数据获取解决方案。如果你刚开始上手，一个常见做法是在 effect 中使用 `fetch` 加载数据。

但这样做意味着数据会在组件渲染之后才被获取，这可能导致网络瀑布。网络瀑布的原因是：数据是在应用渲染时才获取，而不是在代码下载的同时并行获取：

```js
export default function Dashboard() {
  const [data, setData] = useState(null);

  // ❌ 在组件中获取数据会导致网络瀑布
  useEffect(() => {
    fetch('/api/data')
      .then(response => response.json())
      .then(data => setData(data));
  }, []);

  return (
    <div>
      {data.map(item => <div key={item.id}>{item.name}</div>)}
    </div>
  )
}
```

在 effect 中获取数据意味着用户必须等待更久才能看到内容，即使数据本可以更早获取。为了解决这个问题，你可以使用 [TanStack Query](https://tanstack.com/query/)、[SWR](https://swr.vercel.app/)、[Apollo](https://www.apollographql.com/docs/react) 或 [Relay](https://relay.dev/) 之类的数据获取库，它们提供了预取数据的选项，因此请求会在组件渲染前就开始。

当这些库与路由的“loader”模式集成时效果最好，这样可以在路由级别指定数据依赖，路由器也就能够优化数据请求：

```js
export async function loader() {
  const response = await fetch(`/api/data`);
  const data = await response.json();
  return data;
}

// ✅ 在代码下载的同时并行获取数据
export default function Dashboard({loaderData}) {
  return (
    <div>
      {loaderData.map(item => <div key={item.id}>{item.name}</div>)}
    </div>
  )
}
```

在初次加载时，路由器可以在路由渲染之前立即获取数据。当用户在应用中导航时，路由器能够同时获取数据和路由，实现并行请求。这缩短了内容显示在屏幕上的时间，并可以改善用户体验。

不过，这需要你在应用中正确配置 loaders，并且是以复杂度换取性能。

### 代码分割 {/*code-splitting*/}

Create React App 中另一个常见问题是 [代码分割](https://www.patterns.dev/vanilla/bundle-splitting)。Create React App 不包含专门的代码分割解决方案。如果你刚开始上手，可能根本不会考虑代码分割。

这意味着你的应用会以单个 bundle 的形式发布：

```txt
- bundle.js    75kb
```

但为了获得理想性能，你应该把代码“拆分”为多个独立 bundle，这样用户只需要下载他们真正需要的部分。这样可以通过只下载用户当前页面所需的代码，减少用户等待应用加载的时间。

```txt
- core.js      25kb
- home.js      25kb
- dashboard.js 25kb
```

实现代码分割的一种方式是使用 `React.lazy`。然而，这意味着代码要等到组件渲染时才会被获取，这可能导致网络瀑布。更优化的方案是使用路由器的功能，在代码下载时并行获取代码。例如，React Router 提供了 `lazy` 选项，用于指定某个路由应该进行代码分割，并优化其加载时机：

```js
import Home from './Home';
import Dashboard from './Dashboard';

// ✅ 路由会在渲染前下载
const router = createBrowserRouter([
  {path: '/', lazy: () => import('./Home')},
  {path: '/dashboard', lazy: () => import('Dashboard')}
]);
```

优化代码分割很难正确实现，而且很容易犯错，导致用户下载了比实际需要更多的代码。它与路由器和数据加载方案结合使用时效果最好，因为这样可以最大化缓存、并行化请求，并支持 ["交互时导入"](https://www.patterns.dev/vanilla/import-on-interaction) 模式。

### 以及更多... {/*and-more*/}

这些只是 Create React App 局限性的几个例子。

一旦你集成了路由、数据获取和代码分割，你还需要考虑待处理状态、导航中断、向用户显示的错误信息，以及数据重新验证。用户需要解决的问题还有很多，像是：

<div style={{display: 'flex', width: '100%', justifyContent: 'space-around'}}>
  <ul>
    <li>可访问性</li>
    <li>资源加载</li>
    <li>身份验证</li>
    <li>缓存</li>
  </ul>
  <ul>
    <li>错误处理</li>
    <li>数据变更</li>
    <li>导航</li>
    <li>乐观更新</li>
  </ul>
  <ul>
    <li>渐进增强</li>
    <li>服务端渲染</li>
    <li>静态站点生成</li>
    <li>流式传输</li>
  </ul>
</div>

所有这些都会协同工作，从而形成最优的 [加载序列](https://www.patterns.dev/vanilla/loading-sequence)。

在 Create React App 中逐个解决这些问题可能很困难，因为每个问题都与其他问题相互关联，并且可能需要用户并不熟悉的领域中的深厚专业知识。为了真正解决这些问题，用户最终会在 Create React App 之上构建自己的定制方案，而这正是 Create React App 最初试图解决的问题。

## 为什么我们推荐框架 {/*why-we-recommend-frameworks*/}

虽然你可以在诸如 Create React App、Vite 或 Parcel 这样的构建工具中自己解决所有这些问题，但要做好是很难的。就像 Create React App 本身集成了多个构建工具一样，你也需要一个工具把所有这些功能整合起来，为用户提供最佳体验。

这类整合了构建工具、渲染、路由、数据获取和代码分割的工具被称为“框架”——或者如果你更愿意把 React 本身称为框架，也可以把它们称为“元框架”。

框架会对应用的组织方式施加一些约定，以提供更好的用户体验，这和构建工具为了让工具链更易用而施加一些约定是一样的。这就是为什么我们开始推荐新项目使用像 [Next.js](https://nextjs.org/)、[React Router](https://reactrouter.com/) 和 [Expo](https://expo.dev/) 这样的框架。

框架提供了与 Create React App 相同的上手体验，但也为用户在真实生产应用中本来就需要解决的问题提供了解决方案。

<DeepDive>

#### 服务端渲染是可选的 {/*server-rendering-is-optional*/}

我们推荐的框架都提供创建 [客户端渲染（CSR）](https://developer.mozilla.org/en-US/docs/Glossary/CSR) 应用的选项。

在某些情况下，CSR 是页面的正确选择，但很多时候并不是。即使你的应用大部分是客户端渲染，也通常会有一些单独的页面能够从服务端渲染特性中受益，比如 [静态站点生成（SSG）](https://developer.mozilla.org/en-US/docs/Glossary/SSG) 或 [服务端渲染（SSR）](https://developer.mozilla.org/en-US/docs/Glossary/SSR)，例如服务条款页面或文档页面。

服务端渲染通常会向客户端发送更少的 JavaScript，以及一个完整的 HTML 文档，这通过减少 [总阻塞时间（TBD）](https://web.dev/articles/tbt) 产生更快的 [首次内容绘制（FCP）](https://web.dev/articles/fcp)，这也可能降低 [下一次绘制的交互（INP）](https://web.dev/articles/inp)。这就是为什么 [Chrome 团队一直鼓励](https://web.dev/articles/rendering-on-the-web) 开发者考虑使用静态或服务端渲染，而不是完全依赖客户端渲染，以实现尽可能好的性能。

使用服务端也有权衡，并且它并不总是每个页面的最佳选择。在服务器上生成页面会产生额外成本并且需要时间，这可能会增加 [首次字节时间（TTFB）](https://web.dev/articles/ttfb)。性能最好的应用能够基于每种策略的权衡，为每个页面选择合适的渲染策略。

如果你愿意，框架允许你在任何页面上使用服务端，但不会强制你使用服务端。这让你能够为应用中的每个页面选择合适的渲染策略。

#### 那么服务端组件呢 {/*server-components*/}

我们推荐的框架还包含对 React 服务端组件的支持。

服务端组件通过将路由和数据获取移到服务端来帮助解决这些问题，并允许根据你渲染的数据，而不仅仅是渲染的路由，对客户端组件进行代码分割，从而减少传输的 JavaScript 量，以获得尽可能好的 [加载序列](https://www.patterns.dev/vanilla/loading-sequence)。

服务端组件不需要服务器。它们可以在构建时于你的 CI 服务器上运行，生成一个静态站点生成（SSG）应用，也可以在运行时于 Web 服务器上运行，生成一个服务端渲染（SSR）应用。

更多信息请参见 [介绍零包体积的 React 服务端组件](/blog/2020/12/21/data-fetching-with-react-server-components) 和 [文档](/reference/rsc/server-components)。

</DeepDive>

<Note>

#### 服务端渲染不只是为了 SEO {/*server-rendering-is-not-just-for-seo*/}

一个常见的误解是，服务端渲染只用于 [SEO](https://developer.mozilla.org/en-US/docs/Glossary/SEO)。

虽然服务端渲染可以提升 SEO，但它也通过减少用户在看到屏幕内容之前需要下载和解析的 JavaScript 数量来提升性能。

这就是为什么 Chrome 团队 [一直鼓励](https://web.dev/articles/rendering-on-the-web) 开发者考虑使用静态或服务端渲染，而不是完全依赖客户端渲染，以实现尽可能好的性能。

</Note>

---

_感谢 [Dan Abramov](https://bsky.app/profile/danabra.mov) 创建了 Create React App，也感谢 [Joe Haddad](https://github.com/Timer)、[Ian Schmitz](https://github.com/ianschmitz)、[Brody McKee](https://github.com/mrmckeb) 和 [许多其他人](https://github.com/facebook/create-react-app/graphs/contributors) 多年来维护 Create React App。感谢 [Brooks Lybrand](https://bsky.app/profile/brookslybrand.bsky.social)、[Dan Abramov](https://bsky.app/profile/danabra.mov)、[Devon Govett](https://bsky.app/profile/devongovett.bsky.social)、[Eli White](https://x.com/Eli_White)、[Jack Herrington](https://bsky.app/profile/jherr.dev)、[Joe Savona](https://x.com/en_JS)、[Lauren Tan](https://bsky.app/profile/no.lol)、[Lee Robinson](https://x.com/leeerob)、[Mark Erikson](https://bsky.app/profile/acemarke.dev)、[Ryan Florence](https://x.com/ryanflorence)、[Sophie Alpert](https://bsky.app/profile/sophiebits.com)、[Tanner Linsley](https://bsky.app/profile/tannerlinsley.com) 和 [Theo Browne](https://x.com/theo) 对这篇文章进行审阅并提供反馈。_

