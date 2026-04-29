---
title: Server React DOM APIs
---

<Intro>

`react-dom/server` APIs 允许你将 React 组件在服务端渲染为 HTML。这些 API 仅在服务器上、应用的顶层使用，用于生成初始 HTML。某个 [framework](/learn/creating-a-react-app#full-stack-frameworks) 可能会替你调用它们。你的大多数组件都不需要导入或使用它们。

</Intro>

---

## 适用于 Web Streams 的服务器 API {/*server-apis-for-web-streams*/}

这些方法仅在支持 [Web Streams](https://developer.mozilla.org/en-US/docs/Web/API/Streams_API) 的环境中可用，这包括浏览器、Deno，以及一些现代边缘运行时：

* [`renderToReadableStream`](/reference/react-dom/server/renderToReadableStream) 将一个 React 树渲染为一个 [可读的 Web 流。](https://developer.mozilla.org/en-US/docs/Web/API/ReadableStream)
* [`resume`](/reference/react-dom/server/renderToPipeableStream) 将 [`prerender`](/reference/react-dom/static/prerender) 恢复为一个 [Readable Web Stream](https://developer.mozilla.org/en-US/docs/Web/API/ReadableStream)。


<Note>

Node.js 也包含这些方法以保证兼容性，但由于性能更差，不推荐使用。请改用 [专门的 Node.js API](#server-apis-for-nodejs-streams)。

</Note>
---

## 适用于 Node.js Streams 的服务器 API {/*server-apis-for-nodejs-streams*/}

这些方法仅在支持 [Node.js Streams:](https://nodejs.org/api/stream.html) 的环境中可用：

* [`renderToPipeableStream`](/reference/react-dom/server/renderToPipeableStream) 将一个 React 树渲染为一个可管道化的 [Node.js Stream.](https://nodejs.org/api/stream.html)
* [`resumeToPipeableStream`](/reference/react-dom/server/renderToPipeableStream) 将 [`prerenderToNodeStream`](/reference/react-dom/static/prerenderToNodeStream) 恢复为一个可管道化的 [Node.js Stream.](https://nodejs.org/api/stream.html)

---

## 适用于非流式环境的旧版服务器 API {/*legacy-server-apis-for-non-streaming-environments*/}

这些方法可用于不支持流的环境：

* [`renderToString`](/reference/react-dom/server/renderToString) 将一个 React 树渲染为字符串。
* [`renderToStaticMarkup`](/reference/react-dom/server/renderToStaticMarkup) 将一个不可交互的 React 树渲染为字符串。

与流式 API 相比，它们的功能有限。
