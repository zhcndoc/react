---
title: prerenderToNodeStream
---

<Intro>

`prerenderToNodeStream` 使用 [Node.js Stream.](https://nodejs.org/api/stream.html) 将 React 树渲染为静态 HTML 字符串。

```js
const {prelude, postponed} = await prerenderToNodeStream(reactNode, options?)
```

</Intro>

<InlineToc />

<Note>

此 API 仅适用于 Node.js。像 Deno 和现代边缘运行时这类具有 [Web Streams,](https://developer.mozilla.org/en-US/docs/Web/API/Streams_API) 的环境，应改用 [`prerender`](/reference/react-dom/static/prerender)。

</Note>

---

## Reference {/*reference*/}

### `prerenderToNodeStream(reactNode, options?)` {/*prerender*/}

调用 `prerenderToNodeStream` 将你的应用渲染为静态 HTML。

```js
import { prerenderToNodeStream } from 'react-dom/static';

// 路由处理程序语法取决于你的后端框架
app.use('/', async (request, response) => {
  const { prelude } = await prerenderToNodeStream(<App />, {
    bootstrapScripts: ['/main.js'],
  });

  response.setHeader('Content-Type', 'text/plain');
  prelude.pipe(response);
});
```

在客户端，调用 [`hydrateRoot`](/reference/react-dom/client/hydrateRoot) 使服务端生成的 HTML 具备交互性。

[查看更多示例。](#usage)

#### Parameters {/*parameters*/}

* `reactNode`：你希望渲染为 HTML 的 React 节点。例如，一个像 `<App />` 这样的 JSX 节点。它应当表示整个文档，因此 App 组件应该渲染 `<html>` 标签。

* **optional** `options`：包含静态生成选项的对象。
  * **optional** `bootstrapScriptContent`：如果指定，这个字符串会被放入内联 `<script>` 标签中。
  * **optional** `bootstrapScripts`：一组字符串 URL，用于页面中输出的 `<script>` 标签。用它来包含调用 [`hydrateRoot`.](/reference/react-dom/client/hydrateRoot) 的 `<script>`。如果你完全不想在客户端运行 React，可以省略它。
  * **optional** `bootstrapModules`：类似 `bootstrapScripts`，但会改为输出 [`<script type="module">`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules)。
  * **optional** `identifierPrefix`：React 用于 [`useId`.](/reference/react/useId) 生成的 ID 的字符串前缀。在同一页面使用多个根时，这有助于避免冲突。必须与传递给 [`hydrateRoot`.](/reference/react-dom/client/hydrateRoot#parameters) 的前缀相同。
  * **optional** `namespaceURI`：流的根 [namespace URI](https://developer.mozilla.org/en-US/docs/Web/API/Document/createElementNS#important_namespace_uris) 字符串。默认是普通 HTML。SVG 传入 `'http://www.w3.org/2000/svg'`，MathML 传入 `'http://www.w3.org/1998/Math/MathML'`。
  * **optional** `onError`：每当发生服务端错误时触发的回调，无论该错误是[可恢复的](/reference/react-dom/server/renderToPipeableStream#recovering-from-errors-outside-the-shell)还是[不可恢复的。](/reference/react-dom/server/renderToPipeableStream#recovering-from-errors-inside-the-shell) 默认情况下，这只会调用 `console.error`。如果你覆盖它来[记录崩溃报告，](/reference/react-dom/server/renderToPipeableStream#logging-crashes-on-the-server)请确保仍然调用 `console.error`。你也可以在 shell 输出之前用它来[调整状态码。](/reference/react-dom/server/renderToPipeableStream#setting-the-status-code)
  * **optional** `progressiveChunkSize`：每个分块中的字节数。[阅读更多关于默认启发式算法的内容。](https://github.com/facebook/react/blob/14c2be8dac2d5482fda8a0906a31d239df8551fc/packages/react-server/src/ReactFizzServer.js#L210-L225)
  * **optional** `signal`：一个 [abort signal](https://developer.mozilla.org/en-US/docs/Web/API/AbortSignal)，可让你[中止预渲染](#aborting-prerendering)，并在客户端渲染剩余部分。

#### Returns {/*returns*/}

`prerenderToNodeStream` 返回一个 Promise：
- 如果渲染成功，Promise 将解析为一个包含以下内容的对象：
  - `prelude`：一个 [Node.js Stream.](https://nodejs.org/api/stream.html) 的 HTML。你可以使用这个流分块发送响应，也可以将整个流读取为字符串。
  - `postponed`：一个可 JSON 序列化的、不透明的对象。如果 `prerenderToNodeStream` 未完成，它可以传递给 [`resumeToPipeableStream`](/reference/react-dom/server/resumeToPipeableStream)。否则为 `null`，表示 `prelude` 已包含全部内容，无需恢复。
- 如果渲染失败，Promise 将被拒绝。[可用于输出一个回退 shell。](/reference/react-dom/server/renderToPipeableStream#recovering-from-errors-inside-the-shell)

#### Caveats {/*caveats*/}

在预渲染时，`nonce` 不是可用选项。nonce 必须对每个请求唯一，如果你使用 nonce 通过 [CSP](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CSP) 保护应用，那么在预渲染结果中包含 nonce 值是不合适且不安全的。

<Note>

### 什么时候应该使用 `prerenderToNodeStream`？ {/*when-to-use-prerender*/}

静态 `prerenderToNodeStream` API 用于静态服务端生成（SSG）。与 `renderToString` 不同，`prerenderToNodeStream` 会等待所有数据加载完成后才解析。这使它适合生成完整页面的静态 HTML，包括需要通过 Suspense 拉取的数据。若要在内容加载时进行流式输出，请使用像 [renderToReadableStream](/reference/react-dom/server/renderToReadableStream) 这样的服务端流式渲染（SSR）API。

`prerenderToNodeStream` 可以被中止，并稍后通过 `resumeToPipeableStream` 恢复，以支持部分预渲染。

</Note>

---

## Usage {/*usage*/}

### 将 React 树渲染为静态 HTML 流 {/*rendering-a-react-tree-to-a-stream-of-static-html*/}

调用 `prerenderToNodeStream` 将你的 React 树渲染为 [Node.js Stream](https://nodejs.org/api/stream.html) 中的静态 HTML：

```js [[1, 5, "<App />"], [2, 6, "['/main.js']"]]
import { prerenderToNodeStream } from 'react-dom/static';

// 路由处理程序语法取决于你的后端框架
app.use('/', async (request, response) => {
  const { prelude } = await prerenderToNodeStream(<App />, {
    bootstrapScripts: ['/main.js'],
  });

  response.setHeader('Content-Type', 'text/plain');
  prelude.pipe(response);
});
```

除了 <CodeStep step={1}>根组件</CodeStep> 之外，你还需要提供一个 <CodeStep step={2}>bootstrap `<script>` 路径</CodeStep> 列表。你的根组件应当返回**整个文档，包括根 `<html>` 标签。**

例如，它可能看起来像这样：

```js [[1, 1, "App"]]
export default function App() {
  return (
    <html>
      <head>
        <meta charSet="utf-8" />
        <meta name="viewport" content="width=device-width, initial-scale=1" />
        <link rel="stylesheet" href="/styles.css"></link>
        <title>我的应用</title>
      </head>
      <body>
        <Router />
      </body>
    </html>
  );
}
```

React 会将 [doctype](https://developer.mozilla.org/en-US/docs/Glossary/Doctype) 和你的 <CodeStep step={2}>bootstrap `<script>` 标签</CodeStep> 注入到生成的 HTML 流中：

```html [[2, 5, "/main.js"]]
<!DOCTYPE html>
<html>
  <!-- ... 来自你的组件的 HTML ... -->
</html>
<script src="/main.js" async=""></script>
```

在客户端，你的 bootstrap 脚本应该通过调用 [`hydrateRoot`:](/reference/react-dom/client/hydrateRoot#hydrating-an-entire-document) [为整个 `document` 注水]

```js [[1, 4, "<App />"]]
import { hydrateRoot } from 'react-dom/client';
import App from './App.js';

hydrateRoot(document, <App />);
```

这会将事件监听器附加到静态的服务端生成 HTML 上，并使其具备交互性。

<DeepDive>

#### 从构建输出中读取 CSS 和 JS 资源路径 {/*reading-css-and-js-asset-paths-from-the-build-output*/}

最终的资源 URL（例如 JavaScript 和 CSS 文件）在构建后通常会被哈希化。例如，你可能不会得到 `styles.css`，而是得到 `styles.123456.css`。对静态资源文件名进行哈希处理，可以保证同一资源的每一次不同构建都拥有不同的文件名。这很有用，因为它允许你安全地为静态资源启用长期缓存：某个特定名称的文件内容永远不会变化。

不过，如果你在构建完成前不知道资源 URL，就无法把它们写进源代码里。例如，像前面那样在 JSX 中硬编码 `"/styles.css"` 就行不通。为了将它们排除在源代码之外，你的根组件可以从作为 prop 传入的 map 中读取真实文件名：

```js {1,6}
export default function App({ assetMap }) {
  return (
    <html>
      <head>
        <title>我的应用</title>
        <link rel="stylesheet" href={assetMap['styles.css']}></link>
      </head>
      ...
    </html>
  );
}
```

在服务端，渲染 `<App assetMap={assetMap} />`，并将你的 `assetMap` 和资源 URL 一起传入：

```js {1-5,8,9}
// 你需要从构建工具中获取这份 JSON，例如从构建输出中读取它。
const assetMap = {
  'styles.css': '/styles.123456.css',
  'main.js': '/main.123456.js'
};

app.use('/', async (request, response) => {
  const { prelude } = await prerenderToNodeStream(<App />, {
    bootstrapScripts: [assetMap['/main.js']]
  });

  response.setHeader('Content-Type', 'text/html');
  prelude.pipe(response);
});
```

由于你的服务端现在渲染的是 `<App assetMap={assetMap} />`，你也需要在客户端使用 `assetMap` 来渲染它，以避免 hydration 错误。你可以像这样序列化并将 `assetMap` 传给客户端：

```js {9-10}
// 你需要从构建工具中获取这份 JSON。
const assetMap = {
  'styles.css': '/styles.123456.css',
  'main.js': '/main.123456.js'
};

app.use('/', async (request, response) => {
  const { prelude } = await prerenderToNodeStream(<App />, {
    // 注意：对这个对象使用 stringify() 是安全的，因为这些数据不是用户生成的。
    bootstrapScriptContent: `window.assetMap = ${JSON.stringify(assetMap)};`,
    bootstrapScripts: [assetMap['/main.js']],
  });

  response.setHeader('Content-Type', 'text/html');
  prelude.pipe(response);
});
```

在上面的示例中，`bootstrapScriptContent` 选项会额外添加一个内联 `<script>` 标签，用于在客户端设置全局 `window.assetMap` 变量。这使客户端代码能够读取相同的 `assetMap`：

```js {4}
import { hydrateRoot } from 'react-dom/client';
import App from './App.js';

hydrateRoot(document, <App assetMap={window.assetMap} />);
```

客户端和服务端都使用相同的 `assetMap` prop 渲染 `App`，因此不会出现 hydration 错误。

</DeepDive>

---

### 将 React 树渲染为静态 HTML 字符串 {/*rendering-a-react-tree-to-a-string-of-static-html*/}

调用 `prerenderToNodeStream` 将你的应用渲染为一个静态 HTML 字符串：

```js
import { prerenderToNodeStream } from 'react-dom/static';

async function renderToString() {
  const {prelude} = await prerenderToNodeStream(<App />, {
    bootstrapScripts: ['/main.js']
  });

  return new Promise((resolve, reject) => {
    let data = '';
    prelude.on('data', chunk => {
      data += chunk;
    });
    prelude.on('end', () => resolve(data));
    prelude.on('error', reject);
  });
}
```

这将生成你的 React 组件初始的、非交互式的 HTML 输出。在客户端，你需要调用 [`hydrateRoot`](/reference/react-dom/client/hydrateRoot) 来*注水*该服务端生成的 HTML，并使其具备交互性。

---

### 等待所有数据加载完成 {/*waiting-for-all-data-to-load*/}

`prerenderToNodeStream` 会在完成静态 HTML 生成并解析之前等待所有数据加载完成。例如，考虑一个展示封面、带有好友和照片的侧边栏，以及帖子列表的个人资料页面：

```js
function ProfilePage() {
  return (
    <ProfileLayout>
      <ProfileCover />
      <Sidebar>
        <Friends />
        <Photos />
      </Sidebar>
      <Suspense fallback={<PostsGlimmer />}>
        <Posts />
      </Suspense>
    </ProfileLayout>
  );
}
```

设想 `<Posts />` 需要加载一些数据，这会花费一些时间。理想情况下，你会希望等待帖子加载完成，使其包含在 HTML 中。为此，你可以使用 Suspense 在数据上挂起，而 `prerenderToNodeStream` 会在解析为静态 HTML 之前等待挂起的内容完成。

<Note>

**只有启用了 Suspense 的数据源才会激活 Suspense 组件。** 它们包括：

- 使用支持 Suspense 的框架进行数据获取，例如 [Relay](https://relay.dev/docs/guided-tour/rendering/loading-states/) 和 [Next.js](https://nextjs.org/docs/getting-started/react-essentials)
- 使用 [`lazy`](/reference/react/lazy) 延迟加载组件代码
- 使用 [`use`](/reference/react/use) 读取 Promise 的值

Suspense **不会**检测数据是否在 Effect 或事件处理程序中被获取。

上面 `Posts` 组件中加载数据的具体方式取决于你的框架。如果你使用支持 Suspense 的框架，可以在其数据获取文档中找到详细信息。

目前尚不支持在不使用特定框架的情况下进行支持 Suspense 的数据获取。实现支持 Suspense 的数据源所需的要求仍不稳定且没有文档说明。用于将数据源与 Suspense 集成的官方 API 将在未来版本的 React 中发布。

</Note>

---

### 中止预渲染 {/*aborting-prerendering*/}

你可以强制预渲染在超时后“放弃”：

```js {2-5,11}
async function renderToString() {
  const controller = new AbortController();
  setTimeout(() => {
    controller.abort()
  }, 10000);

  try {
    // prelude 将包含在控制器中止之前
    // 已预渲染的所有 HTML。
    const {prelude} = await prerenderToNodeStream(<App />, {
      signal: controller.signal,
    });
    //...
```

任何子元素不完整的 Suspense 边界都会以回退状态包含在 prelude 中。

这可与 [`resumeToPipeableStream`](/reference/react-dom/server/resumeToPipeableStream) 或 [`resumeAndPrerenderToNodeStream`](/reference/react-dom/static/resumeAndPrerenderToNodeStream) 一起用于部分预渲染。

## 故障排除 {/*troubleshooting*/}

### 我的流直到整个应用渲染完成后才开始 {/*my-stream-doesnt-start-until-the-entire-app-is-rendered*/}

`prerenderToNodeStream` 响应会等待整个应用完成渲染，包括等待所有 Suspense 边界解析完成，然后才会进行解析。它是为提前进行静态站点生成（SSG）而设计的，不支持在内容加载时继续流式传输更多内容。

要在内容加载时进行流式传输，请使用类似 [renderToPipeableStream](/reference/react-dom/server/renderToPipeableStream) 的流式服务器渲染 API。
