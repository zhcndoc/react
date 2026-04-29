---
title: prerender
---

<Intro>

`prerender` 使用 [Web Stream](https://developer.mozilla.org/en-US/docs/Web/API/Streams_API) 将 React 树渲染为静态 HTML 字符串。

```js
const {prelude, postponed} = await prerender(reactNode, options?)
```

</Intro>

<InlineToc />

<Note>

此 API 依赖 [Web Streams.](https://developer.mozilla.org/en-US/docs/Web/API/Streams_API) 对于 Node.js，请改用 [`prerenderToNodeStream`](/reference/react-dom/static/prerenderToNodeStream)。

</Note>

---

## 参考 {/*reference*/}

### `prerender(reactNode, options?)` {/*prerender*/}

调用 `prerender` 将你的应用渲染为静态 HTML。

```js
import { prerender } from 'react-dom/static';

async function handler(request, response) {
  const {prelude} = await prerender(<App />, {
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

* `reactNode`：你希望渲染为 HTML 的 React 节点。例如，一个像 `<App />` 这样的 JSX 节点。它应当代表整个文档，因此 App 组件应该渲染 `<html>` 标签。

* **可选** `options`：一个包含静态生成选项的对象。
  * **可选** `bootstrapScriptContent`：如果指定，这个字符串会放入一个内联的 `<script>` 标签中。
  * **可选** `bootstrapScripts`：用于在页面上发出 `<script>` 标签的字符串 URL 数组。用它来包含调用 [`hydrateRoot`.](/reference/react-dom/client/hydrateRoot) 的 `<script>`。如果你不想在客户端运行 React，可以省略它。
  * **可选** `bootstrapModules`：类似 `bootstrapScripts`，但会改为发出 [`<script type="module">`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules)。
  * **可选** `identifierPrefix`：React 为 [`useId`.](/reference/react/useId) 生成的 ID 使用的字符串前缀。在同一页面上使用多个根时，这有助于避免冲突。必须与传递给 [`hydrateRoot`.](/reference/react-dom/client/hydrateRoot#parameters) 的前缀相同。
  * **可选** `namespaceURI`：流的根 [namespace URI](https://developer.mozilla.org/en-US/docs/Web/API/Document/createElementNS#important_namespace_uris) 字符串。默认为常规 HTML。SVG 请传入 `'http://www.w3.org/2000/svg'`，MathML 请传入 `'http://www.w3.org/1998/Math/MathML'`。
  * **可选** `onError`：每当发生服务器错误时触发的回调，无论是 [可恢复的](/reference/react-dom/server/renderToReadableStream#recovering-from-errors-outside-the-shell) 还是 [不可恢复的](/reference/react-dom/server/renderToReadableStream#recovering-from-errors-inside-the-shell)。默认情况下，它只会调用 `console.error`。如果你覆盖它来 [记录崩溃报告，](/reference/react-dom/server/renderToReadableStream#logging-crashes-on-the-server) 请确保仍然调用 `console.error`。你也可以用它在 shell 发出之前 [调整状态码](/reference/react-dom/server/renderToReadableStream#setting-the-status-code)。
  * **可选** `progressiveChunkSize`：每个分块中的字节数。[阅读更多关于默认启发式的信息。](https://github.com/facebook/react/blob/14c2be8dac2d5482fda8a0906a31d239df8551fc/packages/react-server/src/ReactFizzServer.js#L210-L225)
  * **可选** `signal`：一个 [abort signal](https://developer.mozilla.org/en-US/docs/Web/API/AbortSignal)，允许你 [中止预渲染](#aborting-prerendering)，并让其余内容在客户端渲染。

#### 返回值 {/*returns*/}

`prerender` 返回一个 Promise：
- 如果渲染成功，Promise 将解析为一个包含以下内容的对象：
  - `prelude`：一个 HTML 的 [Web Stream](https://developer.mozilla.org/en-US/docs/Web/API/Streams_API)。你可以使用这个流按块发送响应，或者将整个流读取为字符串。
  - `postponed`：一个可 JSON 序列化的、不透明的对象。如果 `prerender` 未完成，它可以传递给 [`resume`](/reference/react-dom/server/resume)。否则为 `null`，表示 `prelude` 已包含全部内容，不需要恢复。
- 如果渲染失败，Promise 将被拒绝。[可用它输出一个备用 shell。](/reference/react-dom/server/renderToReadableStream#recovering-from-errors-inside-the-shell)

#### 注意事项 {/*caveats*/}

预渲染时 `nonce` 不是可用选项。Nonce 必须对每个请求都唯一，如果你使用 nonce 通过 [CSP](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CSP) 保护应用，那么在预渲染中包含 nonce 值是不合适且不安全的。

<Note>

### 我什么时候应该使用 `prerender`？ {/*when-to-use-prerender*/}

静态 `prerender` API 用于静态服务端生成（SSG）。与 `renderToString` 不同，`prerender` 会等待所有数据加载完成后才解析。这使它适合生成整页的静态 HTML，包括需要通过 Suspense 获取的数据。若要在内容加载时进行流式输出，请使用类似 [renderToReadableStream](/reference/react-dom/server/renderToReadableStream) 的流式服务端渲染（SSR）API。

`prerender` 可以被中止，之后可通过 `resumeAndPrerender` 继续，或通过 `resume` 恢复，以支持部分预渲染。

</Note>

---

## 用法 {/*usage*/}

### 将 React 树渲染为静态 HTML 流 {/*rendering-a-react-tree-to-a-stream-of-static-html*/}

调用 `prerender` 将你的 React 树渲染为静态 HTML，并输出到一个 [Readable Web Stream:](https://developer.mozilla.org/en-US/docs/Web/API/ReadableStream)：

```js [[1, 4, "<App />"], [2, 5, "['/main.js']"]]
import { prerender } from 'react-dom/static';

async function handler(request) {
  const {prelude} = await prerender(<App />, {
    bootstrapScripts: ['/main.js']
  });
  return new Response(prelude, {
    headers: { 'content-type': 'text/html' },
  });
}
```

除了 <CodeStep step={1}>根组件</CodeStep> 之外，你还需要提供一组 <CodeStep step={2}>bootstrap `<script>` 路径</CodeStep>。你的根组件应该返回 **整个文档，包括根 `<html>` 标签。**

例如，它可能如下所示：

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

React 会将 [doctype](https://developer.mozilla.org/en-US/docs/Glossary/Doctype) 以及你的 <CodeStep step={2}>bootstrap `<script>` 标签</CodeStep> 注入到最终生成的 HTML 流中：

```html [[2, 5, "/main.js"]]
<!DOCTYPE html>
<html>
  <!-- ... 来自你的组件的 HTML ... -->
</html>
<script src="/main.js" async=""></script>
```

在客户端，你的 bootstrap 脚本应该通过调用 [`hydrateRoot`:](/reference/react-dom/client/hydrateRoot#hydrating-an-entire-document) [将整个 `document` 进行 hydrate]

```js [[1, 4, "<App />"]]
import { hydrateRoot } from 'react-dom/client';
import App from './App.js';

hydrateRoot(document, <App />);
```

这会将事件监听器附加到静态的服务器生成 HTML 上，并让它变得可交互。

<DeepDive>

#### 从构建输出中读取 CSS 和 JS 资源路径 {/*reading-css-and-js-asset-paths-from-the-build-output*/}

最终的资源 URL（如 JavaScript 和 CSS 文件）通常会在构建后变为哈希名。例如，不是 `styles.css`，而是可能变成 `styles.123456.css`。对静态资源文件名进行哈希化可以保证同一资源的每个不同构建都会有不同的文件名。这很有用，因为它可以让你安全地为静态资源启用长期缓存：一个特定名称的文件内容永远不会改变。

然而，如果你在构建完成之前不知道资源 URL，就无法把它们写进源代码中。例如，像前面那样在 JSX 中硬编码 `"/styles.css"` 就行不通。为了将它们排除在源代码之外，你的根组件可以从作为 prop 传入的 map 中读取真实文件名：

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

在服务器端，渲染 `<App assetMap={assetMap} />` 并传入包含资源 URL 的 `assetMap`：

```js {1-5,8,9}
// 你需要从构建工具中获取这份 JSON，例如从构建输出中读取。
const assetMap = {
  'styles.css': '/styles.123456.css',
  'main.js': '/main.123456.js'
};

async function handler(request) {
  const {prelude} = await prerender(<App assetMap={assetMap} />, {
    bootstrapScripts: [assetMap['/main.js']]
  });
  return new Response(prelude, {
    headers: { 'content-type': 'text/html' },
  });
}
```

由于你的服务器现在正在渲染 `<App assetMap={assetMap} />`，你也需要在客户端用 `assetMap` 来渲染它，以避免 hydrate 错误。你可以像这样将 `assetMap` 序列化并传递给客户端：

```js {9-10}
// 你需要从构建工具中获取这份 JSON。
const assetMap = {
  'styles.css': '/styles.123456.css',
  'main.js': '/main.123456.js'
};

async function handler(request) {
  const {prelude} = await prerender(<App assetMap={assetMap} />, {
    // 注意：这里可以安全地使用 stringify()，因为这些数据不是用户生成的。
    bootstrapScriptContent: `window.assetMap = ${JSON.stringify(assetMap)};`,
    bootstrapScripts: [assetMap['/main.js']],
  });
  return new Response(prelude, {
    headers: { 'content-type': 'text/html' },
  });
}
```

在上面的示例中，`bootstrapScriptContent` 选项会额外添加一个内联 `<script>` 标签，用于在客户端设置全局 `window.assetMap` 变量。这使客户端代码可以读取相同的 `assetMap`：

```js {4}
import { hydrateRoot } from 'react-dom/client';
import App from './App.js';

hydrateRoot(document, <App assetMap={window.assetMap} />);
```

客户端和服务器都会使用相同的 `assetMap` prop 渲染 `App`，因此不会出现 hydrate 错误。

</DeepDive>

---

### 将 React 树渲染为静态 HTML 字符串 {/*rendering-a-react-tree-to-a-string-of-static-html*/}

调用 `prerender` 将你的应用渲染为静态 HTML 字符串：

```js
import { prerender } from 'react-dom/static';

async function renderToString() {
  const {prelude} = await prerender(<App />, {
    bootstrapScripts: ['/main.js']
  });

  const reader = prelude.getReader();
  let content = '';
  while (true) {
    const {done, value} = await reader.read();
    if (done) {
      return content;
    }
    content += Buffer.from(value).toString('utf8');
  }
}
```

这将生成你的 React 组件初始的非交互式 HTML 输出。在客户端，你需要调用 [`hydrateRoot`](/reference/react-dom/client/hydrateRoot) 来 *hydrate* 这个服务器生成的 HTML，并使其可交互。

---

### 等待所有数据加载完成 {/*waiting-for-all-data-to-load*/}

`prerender` 会等待所有数据加载完成后，再结束静态 HTML 生成并解析。例如，考虑一个展示封面、好友和照片侧边栏以及文章列表的个人资料页面：

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

设想 `<Posts />` 需要加载一些数据，这会花一些时间。理想情况下，你会希望等文章加载完成后再把它包含进 HTML 中。为此，你可以使用 Suspense 在数据上挂起，而 `prerender` 会等待挂起的内容完成后再解析为静态 HTML。

<Note>

**只有支持 Suspense 的数据源才会激活 Suspense 组件。** 它们包括：

- 使用支持 Suspense 的框架进行数据获取，例如 [Relay](https://relay.dev/docs/guided-tour/rendering/loading-states/) 和 [Next.js](https://nextjs.org/docs/getting-started/react-essentials)
- 使用 [`lazy`](/reference/react/lazy) 懒加载组件代码
- 使用 [`use`](/reference/react/use) 读取 Promise 的值

Suspense **不会** 检测数据是否在 Effect 或事件处理函数中获取。

上面的 `Posts` 组件中具体如何加载数据取决于你的框架。如果你使用支持 Suspense 的框架，你可以在其数据获取文档中找到详细信息。

目前还不支持不使用特定框架的 Suspense 数据获取。实现支持 Suspense 的数据源所需的要求仍不稳定且没有文档说明。用于将数据源与 Suspense 集成的官方 API 将在未来版本的 React 中发布。

</Note>

---

### 中止预渲染 {/*aborting-prerendering*/}

你可以通过超时强制让 prerender“放弃”：

```js {2-5,11}
async function renderToString() {
  const controller = new AbortController();
  setTimeout(() => {
    controller.abort()
  }, 10000);

  try {
    // prelude 将包含在控制器中止之前
    // 预渲染完成的所有 HTML。
    const {prelude} = await prerender(<App />, {
      signal: controller.signal,
    });
    //...
```

任何带有未完成子项的 Suspense 边界都将以 fallback 状态包含在 prelude 中。

这可与 [`resume`](/reference/react-dom/server/resume) 或 [`resumeAndPrerender`](/reference/react-dom/static/resumeAndPrerender) 一起用于部分预渲染。

## 故障排除 {/*troubleshooting*/}

### 我的流直到整个应用渲染完成才开始 {/*my-stream-doesnt-start-until-the-entire-app-is-rendered*/}

`prerender` 响应会等待整个应用完成渲染，包括等待所有 Suspense 边界解析完成后，才会返回。它是为提前进行静态站点生成（SSG）而设计的，并不支持在内容加载时继续流式传输更多内容。

要在内容加载时进行流式传输，请使用类似 [renderToReadableStream](/reference/react-dom/server/renderToReadableStream) 这样的流式服务器渲染 API。
