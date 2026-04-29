---
title: Static React DOM APIs
---

<Intro>

`react-dom/static` API 可让你为 React 组件生成静态 HTML。与流式 API 相比，它们的功能有限。[框架](/learn/creating-a-react-app#full-stack-frameworks) 可能会替你调用它们。大多数组件都不需要导入或使用它们。

</Intro>

---

## Web Streams 的静态 API {/*static-apis-for-web-streams*/}

这些方法仅在支持 [Web Streams](https://developer.mozilla.org/en-US/docs/Web/API/Streams_API) 的环境中可用，包括浏览器、Deno 以及一些现代边缘运行时：

* [`prerender`](/reference/react-dom/static/prerender) 使用 [Readable Web Stream](https://developer.mozilla.org/en-US/docs/Web/API/ReadableStream) 将 React 树渲染为静态 HTML。
* <ExperimentalBadge /> [`resumeAndPrerender`](/reference/react-dom/static/resumeAndPrerender) 使用 [Readable Web Stream](https://developer.mozilla.org/en-US/docs/Web/API/ReadableStream) 继续将已预渲染的 React 树渲染为静态 HTML。

为了兼容，Node.js 也包含这些方法，但由于性能较差，不推荐使用。请改用 [专用的 Node.js API](#static-apis-for-nodejs-streams)。

---

## Node.js Streams 的静态 API {/*static-apis-for-nodejs-streams*/}

这些方法仅在支持 [Node.js Streams](https://nodejs.org/api/stream.html) 的环境中可用：

* [`prerenderToNodeStream`](/reference/react-dom/static/prerenderToNodeStream) 使用 [Node.js Stream](https://nodejs.org/api/stream.html) 将 React 树渲染为静态 HTML。
* <ExperimentalBadge /> [`resumeAndPrerenderToNodeStream`](/reference/react-dom/static/resumeAndPrerenderToNodeStream) 使用 [Node.js Stream](https://nodejs.org/api/stream.html) 继续将已预渲染的 React 树渲染为静态 HTML。

