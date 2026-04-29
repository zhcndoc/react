---
title: resumeAndPrerender
---

<Intro>

`resumeAndPrerender` 使用 [Web Stream](https://developer.mozilla.org/en-US/docs/Web/API/Streams_API) 将预渲染的 React 树继续渲染为静态 HTML 字符串。

```js
const { prelude,postpone } = await resumeAndPrerender(reactNode, postponedState, options?)
```

</Intro>

<InlineToc />

<Note>

此 API 依赖于 [Web Streams.](https://developer.mozilla.org/en-US/docs/Web/API/Streams_API) 对于 Node.js，请改用 [`resumeAndPrerenderToNodeStream`](/reference/react-dom/static/resumeAndPrerenderToNodeStream)。

</Note>

---

## 参考 {/*reference*/}

### `resumeAndPrerender(reactNode, postponedState, options?)` {/*resumeandprerender*/}

调用 `resumeAndPrerender` 可将预渲染的 React 树继续渲染为静态 HTML 字符串。

```js
import { resumeAndPrerender } from 'react-dom/static';
import { getPostponedState } from 'storage';

async function handler(request, response) {
  const postponedState = getPostponedState(request);
  const { prelude } = await resumeAndPrerender(<App />, postponedState, {
    bootstrapScripts: ['/main.js']
  });
  return new Response(prelude, {
    headers: { 'content-type': 'text/html' },
  });
}
```

在客户端，调用 [`hydrateRoot`](/reference/react-dom/client/hydrateRoot) 让服务器生成的 HTML 变得可交互。

[查看更多示例。](#usage)

#### 参数 {/*parameters*/}

* `reactNode`：你调用 `prerender`（或之前的 `resumeAndPrerender`）时传入的 React 节点。例如，一个像 `<App />` 这样的 JSX 元素。它应表示整个文档，因此 `App` 组件应该渲染 `<html>` 标签。
* `postponedState`：由 [prerender API](/reference/react-dom/static/index) 返回的、不透明的 `postpone` 对象，从你保存它的任意位置加载而来（例如 redis、文件或 S3）。
* **可选** `options`：一个包含流式选项的对象。
  * **可选** `signal`：一个 [abort signal](https://developer.mozilla.org/en-US/docs/Web/API/AbortSignal)，可让你 [中止服务器渲染](#aborting-server-rendering) 并在客户端渲染剩余部分。
  * **可选** `onError`：每当发生服务器错误时触发的回调，无论错误是 [可恢复的](#recovering-from-errors-outside-the-shell) 还是 [不可恢复的。](#recovering-from-errors-inside-the-shell) 默认情况下，它只会调用 `console.error`。如果你重写它来 [记录崩溃报告，](#logging-crashes-on-the-server) 请确保你仍然调用 `console.error`。

#### 返回值 {/*returns*/}

`prerender` 返回一个 Promise：
- 如果渲染成功，Promise 将解析为一个包含以下内容的对象：
  - `prelude`：HTML 的 [Web Stream](https://developer.mozilla.org/en-US/docs/Web/API/Streams_API)。你可以使用这个流分块发送响应，也可以将整个流读取为字符串。
  - `postponed`：一个可 JSON 序列化的、不透明对象；如果 `prerender` 被中止，它可以传递给 [`resume`](/reference/react-dom/server/resume) 或 [`resumeAndPrerender`](/reference/react-dom/static/resumeAndPrerender)。
- 如果渲染失败，Promise 将被拒绝。[可用它来输出一个回退壳层。](/reference/react-dom/server/renderToReadableStream#recovering-from-errors-inside-the-shell)

#### 注意事项 {/*caveats*/}

在预渲染时，`nonce` 不是可用选项。nonce 必须在每次请求中唯一，如果你使用 nonce 通过 [CSP](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CSP) 保护应用，那么在预渲染结果中包含 nonce 值是不合适且不安全的。

<Note>

### 什么时候应该使用 `resumeAndPrerender`？ {/*when-to-use-prerender*/}

静态的 `resumeAndPrerender` API 用于静态服务端生成（SSG）。与 `renderToString` 不同，`resumeAndPrerender` 会等待所有数据加载完成后才解析。这使它适合用于生成整页的静态 HTML，包括需要通过 Suspense 获取的数据。若要在内容加载时进行流式输出，请使用像 [renderToReadableStream](/reference/react-dom/server/renderToReadableStream) 这样的流式服务端渲染（SSR）API。

`resumeAndPrerender` 可以被中止，然后稍后通过另一个 `resumeAndPrerender` 继续，或者使用 `resume` 恢复，以支持部分预渲染。

</Note>

---

## 用法 {/*usage*/}

### 延伸阅读 {/*further-reading*/}

`resumeAndPrerender` 的行为与 [`prerender`](/reference/react-dom/static/prerender) 类似，但它可用于继续之前已开始、但被中止的预渲染流程。
有关恢复预渲染树的更多信息，请参阅 [resume 文档](/reference/react-dom/server/resume#resuming-a-prerender)。
