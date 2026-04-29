---
title: resumeToPipeableStream
---

<Intro>

`resumeToPipeableStream` 将一个预渲染的 React 树流式传输到可写入的 [Node.js Stream.](https://nodejs.org/api/stream.html)

```js
const {pipe, abort} = await resumeToPipeableStream(reactNode, postponedState, options?)
```

</Intro>

<InlineToc />

<Note>

此 API 仅适用于 Node.js。具有 [Web Streams,](https://developer.mozilla.org/en-US/docs/Web/API/Streams_API) 的环境，如 Deno 和现代边缘运行时，应改用 [`resume`](/reference/react-dom/server/renderToReadableStream)。

</Note>

---

## Reference {/*reference*/}

### `resumeToPipeableStream(node, postponed, options?)` {/*resume-to-pipeable-stream*/}

调用 `resume`，将一个预渲染的 React 树作为 HTML 恢复渲染到 [Node.js Stream.](https://nodejs.org/api/stream.html#writable-streams)

```js
import { resume } from 'react-dom/server';
import {getPostponedState} from './storage';

async function handler(request, response) {
  const postponed = await getPostponedState(request);
  const {pipe} = resumeToPipeableStream(<App />, postponed, {
    onShellReady: () => {
      pipe(response);
    }
  });
}
```

[查看更多示例。](#usage)

#### Parameters {/*parameters*/}

* `reactNode`: 你使用 `prerender` 调用时传入的 React 节点。例如，像 `<App />` 这样的 JSX 元素。它应表示整个文档，因此 `App` 组件应该渲染 `<html>` 标签。
* `postponedState`: 从 [prerender API](/reference/react-dom/static/index) 返回的、不透明的 `postpone` 对象，加载自你存储它的任意位置（例如 redis、文件或 S3）。
* **optional** `options`: 包含流式传输选项的对象。
  * **optional** `nonce`: 一个 [`nonce`](http://developer.mozilla.org/en-US/docs/Web/HTML/Element/script#nonce) 字符串，用于允许 [`script-src` Content-Security-Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/script-src) 的脚本。
  * **optional** `signal`: 一个 [abort signal](https://developer.mozilla.org/en-US/docs/Web/API/AbortSignal)，可让你 [中止服务器渲染](#aborting-server-rendering) 并在客户端渲染其余部分。
  * **optional** `onError`: 每当发生服务器错误时都会触发的回调，无论是[可恢复的](/reference/react-dom/server/renderToReadableStream#recovering-from-errors-outside-the-shell)还是[不可恢复的。](/reference/react-dom/server/renderToReadableStream#recovering-from-errors-inside-the-shell) 默认情况下，它只会调用 `console.error`。如果你覆盖它以[记录崩溃报告，](/reference/react-dom/server/renderToReadableStream#logging-crashes-on-the-server)请确保你仍然调用 `console.error`。
  * **optional** `onShellReady`: 在 [shell](#specifying-what-goes-into-the-shell) 完成后立即触发的回调。你可以在这里调用 `pipe` 来开始流式传输。React 会在 shell 之后[流式传输额外内容](#streaming-more-content-as-it-loads)，并附带内联 `<script>` 标签，用内容替换 HTML 加载回退内容。
  * **optional** `onShellError`: 如果渲染 shell 时发生错误就会触发的回调。它会将错误作为参数接收。此时流中还没有输出任何字节，并且 `onShellReady` 和 `onAllReady` 都不会被调用，因此你可以[输出一个回退 HTML shell](#recovering-from-errors-inside-the-shell)或使用 prelude。


#### Returns {/*returns*/}

`resume` 返回一个包含两个方法的对象：

* `pipe` 将 HTML 输出到提供的 [Writable Node.js Stream.](https://nodejs.org/api/stream.html#writable-streams) 如果你想启用流式传输，请在 `onShellReady` 中调用 `pipe`；如果是爬虫和静态生成，则在 `onAllReady` 中调用。
* `abort` 可让你[中止服务器渲染](#aborting-server-rendering)并在客户端渲染其余部分。

#### Caveats {/*caveats*/}

- `resumeToPipeableStream` 不接受 `bootstrapScripts`、`bootstrapScriptContent` 或 `bootstrapModules` 选项。相反，你需要将这些选项传递给生成 `postponedState` 的 `prerender` 调用。你也可以手动将 bootstrap 内容注入可写流中。
- `resumeToPipeableStream` 不接受 `identifierPrefix`，因为该前缀需要在 `prerender` 和 `resumeToPipeableStream` 中保持一致。
- 由于 `nonce` 不能提供给 prerender，因此只有在你没有向 prerender 提供脚本时，才应向 `resumeToPipeableStream` 提供 `nonce`。
- `resumeToPipeableStream` 会从根开始重新渲染，直到找到一个未完全预渲染的组件。只有完全预渲染的组件（该组件及其子组件都已完成预渲染）才会被完全跳过。

## Usage {/*usage*/}

### Further reading {/*further-reading*/}

恢复的行为类似于 `renderToReadableStream`。更多示例请查看 [`renderToReadableStream` 的使用部分](/reference/react-dom/server/renderToReadableStream#usage)。
[`prerender` 的使用部分](/reference/react-dom/static/prerender#usage)包含了如何专门使用 `prerenderToNodeStream` 的示例。