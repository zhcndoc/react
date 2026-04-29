---
title: resumeAndPrerenderToNodeStream
---

<Intro>

`resumeAndPrerenderToNodeStream` 使用 [Node.js Stream.](https://nodejs.org/api/stream.html) 将已预渲染的 React 树继续生成静态 HTML 字符串。

```js
const {prelude, postponed} = await resumeAndPrerenderToNodeStream(reactNode, postponedState, options?)
```

</Intro>

<InlineToc />

<Note>

此 API 专用于 Node.js。具有 [Web Streams,](https://developer.mozilla.org/en-US/docs/Web/API/Streams_API) 的环境，例如 Deno 和现代边缘运行时，应改用 [`prerender`](/reference/react-dom/static/prerender)。

</Note>

---

## Reference {/*reference*/}

### `resumeAndPrerenderToNodeStream(reactNode, postponedState, options?)` {/*resumeandprerendertolnodestream*/}

调用 `resumeAndPrerenderToNodeStream` 以继续将已预渲染的 React 树生成静态 HTML 字符串。

```js
import { resumeAndPrerenderToNodeStream } from 'react-dom/static';
import { getPostponedState } from 'storage';

async function handler(request, writable) {
  const postponedState = getPostponedState(request);
  const { prelude } = await resumeAndPrerenderToNodeStream(<App />, JSON.parse(postponedState));
  prelude.pipe(writable);
}
```

在客户端，调用 [`hydrateRoot`](/reference/react-dom/client/hydrateRoot) 使服务器生成的 HTML 具备交互性。

[查看更多示例。](#usage)

#### Parameters {/*parameters*/}

* `reactNode`：你调用 `prerender`（或之前的 `resumeAndPrerenderToNodeStream`）时传入的 React 节点。例如，像 `<App />` 这样的 JSX 元素。它应当表示整个文档，因此 `App` 组件应该渲染 `<html>` 标签。
* `postponedState`：由 [prerender API](/reference/react-dom/static/index) 返回的、不透明的 `postpone` 对象，从你存储它的任何位置加载而来（例如 redis、文件或 S3）。
* **optional** `options`：一个包含流式选项的对象。
  * **optional** `signal`：一个 [abort signal](https://developer.mozilla.org/en-US/docs/Web/API/AbortSignal)，允许你[中止服务器渲染](#aborting-server-rendering)，并在客户端渲染其余部分。
  * **optional** `onError`：一个回调函数，只要发生服务器错误就会触发，无论该错误是[可恢复的](#recovering-from-errors-outside-the-shell)还是[不可恢复的](#recovering-from-errors-inside-the-shell)。默认情况下，这只会调用 `console.error`。如果你覆盖它以[记录崩溃报告，](#logging-crashes-on-the-server)请确保你仍然调用 `console.error`。

#### Returns {/*returns*/}

`resumeAndPrerenderToNodeStream` 返回一个 Promise：
- 如果渲染成功，Promise 将解析为一个包含以下内容的对象：
  - `prelude`：一个 HTML 的 [Web Stream](https://developer.mozilla.org/en-US/docs/Web/API/Streams_API)。你可以使用这个流分块发送响应，也可以将整个流读取为字符串。
  - `postponed`：一个可 JSON 序列化的、不透明对象，如果 `resumeAndPrerenderToNodeStream` 被中止，它可以传递给 [`resumeToNodeStream`](/reference/react-dom/server/resume) 或 [`resumeAndPrerenderToNodeStream`](/reference/react-dom/static/resumeAndPrerenderToNodeStream)。
- 如果渲染失败，Promise 将被拒绝。[使用它来输出一个备用 shell。](/reference/react-dom/server/renderToReadableStream#recovering-from-errors-inside-the-shell)

#### Caveats {/*caveats*/}

当进行预渲染时，`nonce` 不是可用的选项。nonce 必须对每个请求唯一，如果你使用 nonce 通过 [CSP](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CSP) 保护你的应用，那么将 nonce 值包含在预渲染结果中是不合适且不安全的。

<Note>

### 什么时候应该使用 `resumeAndPrerenderToNodeStream`？ {/*when-to-use-prerender*/}

静态 `resumeAndPrerenderToNodeStream` API 用于静态服务端生成（SSG）。与 `renderToString` 不同，`resumeAndPrerenderToNodeStream` 会等到所有数据加载完成后才解析。这使它适合为整页生成静态 HTML，包括需要使用 Suspense 获取的数据。要在内容加载时进行流式传输，请使用类似 [renderToReadableStream](/reference/react-dom/server/renderToReadableStream) 的流式服务端渲染（SSR）API。

`resumeAndPrerenderToNodeStream` 可以被中止，并在之后使用另一个 `resumeAndPrerenderToNodeStream` 继续，或者使用 `resume` 恢复，以支持部分预渲染。

</Note>

---

## Usage {/*usage*/}

### Further reading {/*further-reading*/}

`resumeAndPrerenderToNodeStream` 的行为与 [`prerender`](/reference/react-dom/static/prerender) 类似，但可用于继续之前已开始且被中止的预渲染过程。
有关恢复已预渲染树的更多信息，请参阅 [resume 文档](/reference/react-dom/server/resume#resuming-a-prerender)。

