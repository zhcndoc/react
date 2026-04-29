---
title: React DOM APIs
---

<Intro>

`react-dom` 包包含仅支持 Web 应用程序的方法（这些应用程序运行在浏览器 DOM 环境中）。它们不支持 React Native。

</Intro>

---

## APIs {/*apis*/}

这些 API 可以从你的组件中导入。它们很少使用：

* [`createPortal`](/reference/react-dom/createPortal) 让你可以在 DOM 树的不同部分渲染子组件。
* [`flushSync`](/reference/react-dom/flushSync) 让你可以强制 React 同步刷新状态更新并更新 DOM。

## Resource Preloading APIs {/*resource-preloading-apis*/}

这些 API 可以通过在你确定需要资源时尽早预加载脚本、样式表和字体来加快应用速度，例如在导航到另一个会使用这些资源的页面之前。

[基于 React 的框架](/learn/creating-a-react-app)通常会为你处理资源加载，因此你可能不需要自己调用这些 API。请查阅你所使用框架的文档了解详情。

* [`prefetchDNS`](/reference/react-dom/prefetchDNS) 让你可以预取你预期要连接的 DNS 域名的 IP 地址。
* [`preconnect`](/reference/react-dom/preconnect) 让你可以连接到你预期会从中请求资源的服务器，即使你还不知道最终需要哪些资源。
* [`preload`](/reference/react-dom/preload) 让你可以预加载你预期要使用的样式表、字体、图片或外部脚本。
* [`preloadModule`](/reference/react-dom/preloadModule) 让你可以预加载你预期要使用的 ESM 模块。
* [`preinit`](/reference/react-dom/preinit) 让你可以获取并执行外部脚本，或获取并插入样式表。
* [`preinitModule`](/reference/react-dom/preinitModule) 让你可以获取并执行 ESM 模块。

---

## Entry points {/*entry-points*/}

`react-dom` 包提供了两个额外的入口点：

* [`react-dom/client`](/reference/react-dom/client) 包含在客户端（浏览器中）渲染 React 组件的 API。
* [`react-dom/server`](/reference/react-dom/server) 包含在服务器上渲染 React 组件的 API。

---

## Removed APIs {/*removed-apis*/}

这些 API 已在 React 19 中移除：

* [`findDOMNode`](https://18.react.dev/reference/react-dom/findDOMNode)：请参见[替代方案](https://18.react.dev/reference/react-dom/findDOMNode#alternatives)。
* [`hydrate`](https://18.react.dev/reference/react-dom/hydrate)：请改用 [`hydrateRoot`](/reference/react-dom/client/hydrateRoot)。
* [`render`](https://18.react.dev/reference/react-dom/render)：请改用 [`createRoot`](/reference/react-dom/client/createRoot)。
* [`unmountComponentAtNode`](https://18.react.dev/reference/react-dom/unmountComponentAtNode)：请改用 [`root.unmount()`](/reference/react-dom/client/createRoot#root-unmount)。
* [`renderToNodeStream`](https://18.react.dev/reference/react-dom/server/renderToNodeStream)：请改用 [`react-dom/server`](/reference/react-dom/server) 的 API。
* [`renderToStaticNodeStream`](https://18.react.dev/reference/react-dom/server/renderToStaticNodeStream)：请改用 [`react-dom/server`](/reference/react-dom/server) 的 API。
