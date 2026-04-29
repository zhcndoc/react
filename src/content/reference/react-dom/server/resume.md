---
title: resume
---

<Intro>

`resume` 会将一个预渲染的 React 树流式传输到一个 [Readable Web Stream.](https://developer.mozilla.org/en-US/docs/Web/API/ReadableStream)

```js
const stream = await resume(reactNode, postponedState, options?)
```

</Intro>

<InlineToc />

<Note>

此 API 依赖 [Web Streams.](https://developer.mozilla.org/en-US/docs/Web/API/Streams_API) 对于 Node.js，请改用 [`resumeToNodeStream`](/reference/react-dom/server/renderToPipeableStream)。

</Note>

---

## Reference {/*reference*/}

### `resume(node, postponedState, options?)` {/*resume*/}

调用 `resume` 将一个预渲染的 React 树以 HTML 的形式恢复渲染到一个 [Readable Web Stream.](https://developer.mozilla.org/en-US/docs/Web/API/ReadableStream)

```js
import { resume } from 'react-dom/server';
import {getPostponedState} from './storage';

async function handler(request, writable) {
  const postponed = await getPostponedState(request);
  const resumeStream = await resume(<App />, postponed);
  return resumeStream.pipeTo(writable)
}
```

[查看下面的更多示例。](#usage)

#### Parameters {/*parameters*/}

* `reactNode`：你在调用 `prerender` 时使用的 React 节点。例如，一个像 `<App />` 这样的 JSX 元素。它应当表示整个文档，因此 `App` 组件应该渲染 `<html>` 标签。
* `postponedState`：从 [prerender API](/reference/react-dom/static/index) 返回的、不透明的 `postpone` 对象，它是从你存储它的任何位置加载的（例如 redis、文件或 S3）。
* **optional** `options`：包含流式传输选项的对象。
  * **optional** `nonce`：一个 [`nonce`](http://developer.mozilla.org/en-US/docs/Web/HTML/Element/script#nonce) 字符串，用于允许脚本满足 [`script-src` Content-Security-Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/script-src)。
  * **optional** `signal`：一个 [abort signal](https://developer.mozilla.org/en-US/docs/Web/API/AbortSignal)，可让你 [中止服务器渲染](#aborting-server-rendering)，并在客户端渲染剩余内容。
  * **optional** `onError`：每当发生服务器错误时触发的回调，无论该错误是否 [可恢复](/reference/react-dom/server/renderToReadableStream#recovering-from-errors-outside-the-shell) 或 [不可恢复。](/reference/react-dom/server/renderToReadableStream#recovering-from-errors-inside-the-shell) 默认情况下，它只会调用 `console.error`。如果你将其覆盖为 [记录崩溃报告，](/reference/react-dom/server/renderToReadableStream#logging-crashes-on-the-server) 请确保你仍然调用 `console.error`。


#### Returns {/*returns*/}

`resume` 返回一个 Promise：

- 如果 `resume` 成功生成了一个 [shell](/reference/react-dom/server/renderToReadableStream#specifying-what-goes-into-the-shell)，该 Promise 将解析为一个 [Readable Web Stream.](https://developer.mozilla.org/en-US/docs/Web/API/ReadableStream)，它可以被管道传输到一个 [Writable Web Stream.](https://developer.mozilla.org/en-US/docs/Web/API/WritableStream)。
- 如果 shell 中发生错误，Promise 将以该错误拒绝。

返回的流还有一个额外属性：

* `allReady`：一个 Promise，当所有渲染完成时会解析。你可以在返回响应之前 `await stream.allReady` [用于爬虫和静态生成。](/reference/react-dom/server/renderToReadableStream#waiting-for-all-content-to-load-for-crawlers-and-static-generation) 如果这样做，你将不会获得任何渐进式加载。流中将包含最终的 HTML。

#### Caveats {/*caveats*/}

- `resume` 不接受 `bootstrapScripts`、`bootstrapScriptContent` 或 `bootstrapModules` 的选项。相反，你需要将这些选项传递给生成 `postponedState` 的 `prerender` 调用。你也可以手动将 bootstrap 内容注入到 writable stream 中。
- `resume` 不接受 `identifierPrefix`，因为该前缀在 `prerender` 和 `resume` 中都需要保持一致。
- 由于 `nonce` 不能提供给 prerender，因此只有在你没有向 prerender 提供脚本时，才应将 `nonce` 提供给 `resume`。
- `resume` 会从根节点重新渲染，直到找到一个尚未完全预渲染的组件。只有完全预渲染的组件（该组件及其子组件都已完成预渲染）才会被完全跳过。

## Usage {/*usage*/}

### Resuming a prerender {/*resuming-a-prerender*/}

<Sandpack>

```js src/App.js hidden
```

```html public/index.html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>文档</title>
</head>
<body>
  <iframe id="container"></iframe>
</body>
</html>
```

```js src/index.js
import {
  flushReadableStreamToFrame,
  getUser,
  Postponed,
  sleep,
} from "./demo-helpers";
import { StrictMode, Suspense, use, useEffect } from "react";
import { prerender } from "react-dom/static";
import { resume } from "react-dom/server";
import { hydrateRoot } from "react-dom/client";

function Header() {
  return <header>我和我的后代都可以被预渲染</header>;
}

const { promise: cookies, resolve: resolveCookies } = Promise.withResolvers();

function Main() {
  const { sessionID } = use(cookies);
  const user = getUser(sessionID);

  useEffect(() => {
    console.log("已到达交互性！");
  }, []);

  return (
    <main>
      你好，{user.name}！
      <button onClick={() => console.log("已完成 hydration！")}>
        点击我需要 hydration。
      </button>
    </main>
  );
}

function Shell({ children }) {
  // 在真实应用中，这里应该放置你的 html 和 body。
  // 这里只是为了演示，在现有的 body 中包含这些标签
  return (
    <html>
      <body>{children}</body>
    </html>
  );
}

function App() {
  return (
    <Shell>
      <Suspense fallback="正在加载 header">
        <Header />
      </Suspense>
      <Suspense fallback="正在加载 main">
        <Main />
      </Suspense>
    </Shell>
  );
}

async function main(frame) {
  // 第 1 层
  const controller = new AbortController();
  const prerenderedApp = prerender(<App />, {
    signal: controller.signal,
    onError(error) {
      if (error instanceof Postponed) {
      } else {
        console.error(error);
      }
    },
  });
  // 我们在一个宏任务中立即中止。
  // 任何无法同步获得、或无法在微任务中获得的数据获取都不会完成。
  setTimeout(() => {
    controller.abort(new Postponed());
  });

  const { prelude, postponed } = await prerenderedApp;
  await flushReadableStreamToFrame(prelude, frame);

  // 第 2 层
  // 这里只是为了演示而等待。
  // 在真实应用中，prelude 和 postponed state 本应在第 1 层被序列化，随后由第 2 层反序列化它们。
  // 当 React 继续从 prerender 中断的位置渲染时，prelude 内容可以作为普通 HTML 立即刷新输出。
  await sleep(2000);

  // 你会从传入的 HTTP 请求中获取 cookies
  resolveCookies({ sessionID: "abc" });

  const stream = await resume(<App />, postponed);

  await flushReadableStreamToFrame(stream, frame);

  // 第 3 层
  // 这里只是为了演示而等待。
  await sleep(2000);

  hydrateRoot(frame.contentWindow.document, <App />);
}

main(document.getElementById("container"));

```

```js src/demo-helpers.js
export async function flushReadableStreamToFrame(readable, frame) {
  const document = frame.contentWindow.document;
  const decoder = new TextDecoder();
  for await (const chunk of readable) {
    const partialHTML = decoder.decode(chunk);
    document.write(partialHTML);
  }
}

// 这不一定需要是一个错误。
// 你可以使用任何其他方式来检查 prerender 期间的错误是否
// 来自有意的中止还是一个真实错误。
export class Postponed extends Error {}

// 这里我们只是硬编码了一个 session。
export function getUser(sessionID) {
  return {
    name: "Alice",
  };
}

export function sleep(timeoutMS) {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve();
    }, timeoutMS);
  });
}
```

</Sandpack>

### 进一步阅读 {/*further-reading*/}

`resuming` 的行为类似于 `renderToReadableStream`。更多示例请查看 [`renderToReadableStream` 的使用部分](/reference/react-dom/server/renderToReadableStream#usage)。
[`prerender` 的使用部分](/reference/react-dom/static/prerender#usage) 包含了如何专门使用 `prerender` 的示例。