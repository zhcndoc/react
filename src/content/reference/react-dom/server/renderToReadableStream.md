---
title: renderToReadableStream
---

<Intro>

`renderToReadableStream` 会将 React 树渲染为 [Readable Web Stream。](https://developer.mozilla.org/en-US/docs/Web/API/ReadableStream)

```js
const stream = await renderToReadableStream(reactNode, options?)
```

</Intro>

<InlineToc />

<Note>

此 API 依赖于 [Web Streams。](https://developer.mozilla.org/en-US/docs/Web/API/Streams_API) 对于 Node.js，请改用 [`renderToPipeableStream`](/reference/react-dom/server/renderToPipeableStream)。

</Note>

---

## 参考 {/*reference*/}

### `renderToReadableStream(reactNode, options?)` {/*rendertoreadablestream*/}

调用 `renderToReadableStream` 可将你的 React 树以 HTML 的形式渲染到 [Readable Web Stream。](https://developer.mozilla.org/en-US/docs/Web/API/ReadableStream)

```js
import { renderToReadableStream } from 'react-dom/server';

async function handler(request) {
  const stream = await renderToReadableStream(<App />, {
    bootstrapScripts: ['/main.js']
  });
  return new Response(stream, {
    headers: { 'content-type': 'text/html' },
  });
}
```

在客户端，调用 [`hydrateRoot`](/reference/react-dom/client/hydrateRoot) 让服务端生成的 HTML 变得可交互。

[查看更多示例。](#usage)

#### 参数 {/*parameters*/}

* `reactNode`：你想要渲染为 HTML 的 React 节点。例如，像 `<App />` 这样的 JSX 元素。它应当表示整个文档，因此 `App` 组件应该渲染 `<html>` 标签。

* **可选** `options`：一个带有流式传输选项的对象。
  * **可选** `bootstrapScriptContent`：如果指定，这个字符串会被放入一个内联 `<script>` 标签中。
  * **可选** `bootstrapScripts`：一个字符串 URL 数组，用于在页面上发出 `<script>` 标签。使用它来包含调用 [`hydrateRoot`.](/reference/react-dom/client/hydrateRoot) 的 `<script>`。如果你完全不想在客户端运行 React，可以省略它。
  * **可选** `bootstrapModules`：与 `bootstrapScripts` 类似，但会改为发出 [`<script type="module">`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules)。
  * **可选** `identifierPrefix`：React 用于通过 [`useId`.](/reference/react/useId) 生成 ID 的字符串前缀。在同一页面使用多个根时，这有助于避免冲突。必须与传给 [`hydrateRoot`.](/reference/react-dom/client/hydrateRoot#parameters) 的前缀相同
  * **可选** `namespaceURI`：流的根 [namespace URI](https://developer.mozilla.org/en-US/docs/Web/API/Document/createElementNS#important_namespace_uris) 字符串。默认为普通 HTML。对于 SVG 传入 `'http://www.w3.org/2000/svg'`，对于 MathML 传入 `'http://www.w3.org/1998/Math/MathML'`。
  * **可选** `nonce`：一个 [`nonce`](http://developer.mozilla.org/en-US/docs/Web/HTML/Element/script#nonce) 字符串，用于允许 [`script-src` Content-Security-Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/script-src) 下的脚本。
  * **可选** `onError`：每当发生服务端错误时触发的回调，无论是[可恢复的](#recovering-from-errors-outside-the-shell)还是[不可恢复的。](#recovering-from-errors-inside-the-shell)默认情况下，这只会调用 `console.error`。如果你覆盖它以[记录崩溃报告，](#logging-crashes-on-the-server)请确保你仍然调用 `console.error`。你也可以用它在 shell 发出之前[调整状态码。](#setting-the-status-code)
  * **可选** `progressiveChunkSize`：每个块中的字节数。[阅读更多关于默认启发式的信息。](https://github.com/facebook/react/blob/14c2be8dac2d5482fda8a0906a31d239df8551fc/packages/react-server/src/ReactFizzServer.js#L210-L225)
  * **可选** `signal`：[中止信号](https://developer.mozilla.org/en-US/docs/Web/API/AbortSignal)，它允许你[中止服务端渲染](#aborting-server-rendering)并在客户端渲染剩余部分。


#### 返回值 {/*returns*/}

`renderToReadableStream` 返回一个 Promise：

- 如果渲染 [shell](#specifying-what-goes-into-the-shell) 成功，该 Promise 将解析为一个 [Readable Web Stream。](https://developer.mozilla.org/en-US/docs/Web/API/ReadableStream)
- 如果 shell 渲染失败，该 Promise 将被拒绝。[可用它来输出一个回退 shell。](#recovering-from-errors-inside-the-shell)

返回的流有一个额外属性：

* `allReady`：当所有渲染完成时解析的 Promise，包括 [shell](#specifying-what-goes-into-the-shell) 和所有额外[内容。](#streaming-more-content-as-it-loads) 你可以在返回响应前 `await stream.allReady`，[用于爬虫和静态生成。](#waiting-for-all-content-to-load-for-crawlers-and-static-generation) 如果这样做，就不会有任何渐进式加载。流将包含最终 HTML。

---

## 用法 {/*usage*/}

### 将 React 树作为 HTML 渲染到 Readable Web Stream {/*rendering-a-react-tree-as-html-to-a-readable-web-stream*/}

调用 `renderToReadableStream` 可将你的 React 树以 HTML 的形式渲染到 [Readable Web Stream:](https://developer.mozilla.org/en-US/docs/Web/API/ReadableStream)

```js [[1, 4, "<App />"], [2, 5, "['/main.js']"]]
import { renderToReadableStream } from 'react-dom/server';

async function handler(request) {
  const stream = await renderToReadableStream(<App />, {
    bootstrapScripts: ['/main.js']
  });
  return new Response(stream, {
    headers: { 'content-type': 'text/html' },
  });
}
```

除了 <CodeStep step={1}>根组件</CodeStep> 之外，你还需要提供一个 <CodeStep step={2}>bootstrap `<script>` 路径</CodeStep> 列表。你的根组件应该返回**整个文档，包括根 `<html>` 标签。**

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

在客户端，你的 bootstrap 脚本应该[通过调用 hydrateRoot 来 hydrate 整个 `document`：](/reference/react-dom/client/hydrateRoot#hydrating-an-entire-document)

```js [[1, 4, "<App />"]]
import { hydrateRoot } from 'react-dom/client';
import App from './App.js';

hydrateRoot(document, <App />);
```

这会将事件监听器附加到服务端生成的 HTML 上，并使其可交互。

<DeepDive>

#### 从构建输出中读取 CSS 和 JS 资源路径 {/*reading-css-and-js-asset-paths-from-the-build-output*/}

最终的资源 URL（例如 JavaScript 和 CSS 文件）通常会在构建后被哈希化。例如，你可能最终得到的不是 `styles.css`，而是 `styles.123456.css`。对静态资源文件名进行哈希处理，可以保证同一个资源的每次不同构建都会有不同的文件名。这很有用，因为它能让你安全地为静态资源启用长期缓存：某个名称的文件内容永远不会改变。

然而，如果你在构建后才知道资源 URL，就没有办法把它们写进源代码。例如，像前面那样在 JSX 中硬编码 `"/styles.css"` 就不会起作用。为了把它们排除在源代码之外，你的根组件可以从作为 prop 传入的映射中读取真实文件名：

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

在服务端，渲染 `<App assetMap={assetMap} />`，并把带有资源 URL 的 `assetMap` 传入：

```js {1-5,8,9}
// 你需要从构建工具中获取这个 JSON，例如从构建输出中读取。
const assetMap = {
  'styles.css': '/styles.123456.css',
  'main.js': '/main.123456.js'
};

async function handler(request) {
  const stream = await renderToReadableStream(<App assetMap={assetMap} />, {
    bootstrapScripts: [assetMap['/main.js']]
  });
  return new Response(stream, {
    headers: { 'content-type': 'text/html' },
  });
}
```

由于现在服务端正在渲染 `<App assetMap={assetMap} />`，你也需要在客户端使用 `assetMap` 来渲染，以避免 hydration 错误。你可以像这样把 `assetMap` 序列化并传递给客户端：

```js {9-10}
// 你需要从构建工具中获取这个 JSON。
const assetMap = {
  'styles.css': '/styles.123456.css',
  'main.js': '/main.123456.js'
};

async function handler(request) {
  const stream = await renderToReadableStream(<App assetMap={assetMap} />, {
    // 注意：对它进行 stringify() 是安全的，因为这些数据不是用户生成的。
    bootstrapScriptContent: `window.assetMap = ${JSON.stringify(assetMap)};`,
    bootstrapScripts: [assetMap['/main.js']],
  });
  return new Response(stream, {
    headers: { 'content-type': 'text/html' },
  });
}
```

在上面的示例中，`bootstrapScriptContent` 选项会额外添加一个内联 `<script>` 标签，在客户端设置全局 `window.assetMap` 变量。这让客户端代码可以读取相同的 `assetMap`：

```js {4}
import { hydrateRoot } from 'react-dom/client';
import App from './App.js';

hydrateRoot(document, <App assetMap={window.assetMap} />);
```

客户端和服务端都使用相同的 `assetMap` prop 渲染 `App`，因此不会有 hydration 错误。

</DeepDive>

---

### 流式加载更多内容 {/*streaming-more-content-as-it-loads*/}

流式传输允许用户在所有数据都加载到服务端之前，就先看到内容。例如，考虑一个显示封面、带有好友和照片的侧边栏，以及帖子列表的个人资料页面：

```js
function ProfilePage() {
  return (
    <ProfileLayout>
      <ProfileCover />
      <Sidebar>
        <Friends />
        <Photos />
      </Sidebar>
      <Posts />
    </ProfileLayout>
  );
}
```

假设 `<Posts />` 的数据加载需要一些时间。理想情况下，你会希望在不等待帖子加载的情况下，先向用户显示个人资料页面的其余内容。为此，[用 `<Suspense>` 边界包裹 `Posts`：](/reference/react/Suspense#displaying-a-fallback-while-content-is-loading)

```js {9,11}
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

这会告诉 React 在 `Posts` 加载数据之前就开始流式传输 HTML。React 会先发送加载回退内容（`PostsGlimmer`）的 HTML，然后当 `Posts` 完成数据加载时，React 会连同一个内联 `<script>` 标签一起发送剩余的 HTML，该标签会用这段 HTML 替换加载回退内容。从用户的角度看，页面会先显示 `PostsGlimmer`，之后被 `Posts` 替换。

你还可以进一步[嵌套 `<Suspense>` 边界](/reference/react/Suspense#revealing-nested-content-as-it-loads) 来创建更细粒度的加载顺序：

```js {5,13}
function ProfilePage() {
  return (
    <ProfileLayout>
      <ProfileCover />
      <Suspense fallback={<BigSpinner />}>
        <Sidebar>
          <Friends />
          <Photos />
        </Sidebar>
        <Suspense fallback={<PostsGlimmer />}>
          <Posts />
        </Suspense>
      </Suspense>
    </ProfileLayout>
  );
}
```


在这个示例中，React 可以更早开始流式传输页面。只有 `ProfileLayout` 和 `ProfileCover` 必须先完成渲染，因为它们没有被任何 `<Suspense>` 边界包裹。不过，如果 `Sidebar`、`Friends` 或 `Photos` 需要加载一些数据，React 会改为发送 `BigSpinner` 回退内容的 HTML。然后，随着更多数据变得可用，会继续揭示更多内容，直到全部内容都可见。

流式传输不需要等待 React 本身在浏览器中加载，也不需要等待你的应用变得可交互。来自服务端的 HTML 内容会在任何 `<script>` 标签加载之前逐步显示出来。

[阅读更多关于流式 HTML 工作方式的信息。](https://github.com/reactwg/react-18/discussions/37)

<Note>

**只有启用了 Suspense 的数据源才会激活 Suspense 组件。** 它们包括：

- 使用支持 Suspense 的框架进行数据获取，例如 [Relay](https://relay.dev/docs/guided-tour/rendering/loading-states/) 和 [Next.js](https://nextjs.org/docs/getting-started/react-essentials)
- 使用 [`lazy`](/reference/react/lazy) 懒加载组件代码
- 使用 [`use`](/reference/react/use) 读取 Promise 的值

Suspense **不会**检测数据是否在 Effect 或事件处理器内部被获取。

上面 `Posts` 组件中加载数据的具体方式取决于你的框架。如果你使用的是支持 Suspense 的框架，你会在其数据获取文档中找到详细信息。

目前还不支持在不使用有明确意见的框架的情况下进行支持 Suspense 的数据获取。实现支持 Suspense 的数据源所需的要求尚不稳定，而且没有文档说明。用于将数据源与 Suspense 集成的官方 API 将在未来的 React 版本中发布。

</Note>

---

### 指定 shell 中包含哪些内容 {/*specifying-what-goes-into-the-shell*/}

应用中位于任何 `<Suspense>` 边界之外的部分称为 *shell：*

```js {3-5,13,14}
function ProfilePage() {
  return (
    <ProfileLayout>
      <ProfileCover />
      <Suspense fallback={<BigSpinner />}>
        <Sidebar>
          <Friends />
          <Photos />
        </Sidebar>
        <Suspense fallback={<PostsGlimmer />}>
          <Posts />
        </Suspense>
      </Suspense>
    </ProfileLayout>
  );
}
```

它决定了用户可能看到的最早加载状态：

```js {3-5,13
<ProfileLayout>
  <ProfileCover />
  <BigSpinner />
</ProfileLayout>
```

如果你在根部将整个应用包裹进一个 `<Suspense>` 边界，shell 将只包含那个 spinner。然而，这不是一种令人愉快的用户体验，因为看到屏幕上一个大 spinner 往往比再等一会儿看到真实布局更慢、更烦人。这就是为什么通常你会希望放置 `<Suspense>` 边界，使得 shell 感觉*尽可能精简但完整*——就像整个页面布局的骨架一样。

对 `renderToReadableStream` 的异步调用会在整个 shell 渲染完成后立即解析为一个 `stream`。通常，你会在那时通过创建并返回带有该 `stream` 的响应来开始流式传输：

```js {5}
async function handler(request) {
  const stream = await renderToReadableStream(<App />, {
    bootstrapScripts: ['/main.js']
  });
  return new Response(stream, {
    headers: { 'content-type': 'text/html' },
  });
}
```

在 `stream` 返回时，嵌套 `<Suspense>` 边界中的组件可能仍在加载数据。

---

### 记录服务端崩溃 {/*logging-crashes-on-the-server*/}

默认情况下，服务端的所有错误都会记录到控制台。你可以覆盖这一行为来记录崩溃报告：

```js {4-7}
async function handler(request) {
  const stream = await renderToReadableStream(<App />, {
    bootstrapScripts: ['/main.js'],
    onError(error) {
      console.error(error);
      logServerCrashReport(error);
    }
  });
  return new Response(stream, {
    headers: { 'content-type': 'text/html' },
  });
}
```

如果你提供了自定义的 `onError` 实现，别忘了像上面那样也将错误记录到控制台。

---

### 恢复 shell 内部的错误 {/*recovering-from-errors-inside-the-shell*/}

在这个示例中，shell 包含 `ProfileLayout`、`ProfileCover` 和 `PostsGlimmer`：

```js {3-5,7-8}
function ProfilePage() {
  return (
    <ProfileLayout>
      <ProfileCover />
      <Suspense fallback={<PostsGlimmer />}>
        <Posts />
      </Suspense>
    </ProfileLayout>
  );
}
```

如果在渲染这些组件时发生错误，React 将没有任何有意义的 HTML 可发送给客户端。将你的 `renderToReadableStream` 调用包裹在 `try...catch` 中，作为最后手段发送一个不依赖服务端渲染的回退 HTML：

```js {2,13-18}
async function handler(request) {
  try {
    const stream = await renderToReadableStream(<App />, {
      bootstrapScripts: ['/main.js'],
      onError(error) {
        console.error(error);
        logServerCrashReport(error);
      }
    });
    return new Response(stream, {
      headers: { 'content-type': 'text/html' },
    });
  } catch (error) {
    return new Response('<h1>出错了</h1>', {
      status: 500,
      headers: { 'content-type': 'text/html' },
    });
  }
}
```

如果在生成 shell 时发生错误，`onError` 和你的 `catch` 块都会触发。使用 `onError` 进行错误上报，并使用 `catch` 块发送回退 HTML 文档。你的回退 HTML 不一定要是错误页面。相反，你可以包含一个仅在客户端渲染应用的替代 shell。

---

### 恢复 shell 外部的错误 {/*recovering-from-errors-outside-the-shell*/}

在这个示例中，`<Posts />` 组件被包裹在 `<Suspense>` 中，因此它*不*属于 shell：

```js {6}
function ProfilePage() {
  return (
    <ProfileLayout>
      <ProfileCover />
      <Suspense fallback={<PostsGlimmer />}>
        <Posts />
      </Suspense>
    </ProfileLayout>
  );
}
```

如果 `Posts` 组件或其内部某处发生错误，React 会[尝试从中恢复：](/reference/react/Suspense#providing-a-fallback-for-server-errors-and-client-only-content)

1. 它会将最近的 `<Suspense>` 边界（`PostsGlimmer`）的加载回退内容输出到 HTML 中。
2. 它会“放弃”继续尝试在服务端渲染 `Posts` 内容。
3. 当 JavaScript 代码在客户端加载后，React 会在客户端*重试*渲染 `Posts`。

如果在客户端重试渲染 `Posts` *也*失败了，React 会在客户端抛出该错误。与渲染期间抛出的所有错误一样，[最近的父级错误边界](/reference/react/Component#static-getderivedstatefromerror) 决定如何向用户呈现该错误。实际上，这意味着用户会看到加载指示器，直到确定该错误不可恢复为止。

如果在客户端重试渲染 `Posts` 成功了，来自服务端的加载回退内容将被客户端渲染输出替换。用户不会知道曾经发生过服务端错误。不过，服务端的 `onError` 回调和客户端的 [`onRecoverableError`](/reference/react-dom/client/hydrateRoot#hydrateroot) 回调都会触发，以便你收到错误通知。

---

### 设置状态码 {/*setting-the-status-code*/}

流式传输带来了一个权衡。你希望尽早开始流式传输页面，这样用户就能更快看到内容。然而，一旦开始流式传输，你就不能再设置响应状态码了。

通过将你的应用[划分为 shell](#specifying-what-goes-into-the-shell)（位于所有 `<Suspense>` 边界之上）和其余内容，你已经解决了这个问题的一部分。如果 shell 出错，你的 `catch` 块会运行，从而让你设置错误状态码。否则，你知道应用可能会在客户端恢复，所以你可以发送“OK”。

```js {11}
async function handler(request) {
  try {
    const stream = await renderToReadableStream(<App />, {
      bootstrapScripts: ['/main.js'],
      onError(error) {
        console.error(error);
        logServerCrashReport(error);
      }
    });
    return new Response(stream, {
      status: 200,
      headers: { 'content-type': 'text/html' },
    });
  } catch (error) {
    return new Response('<h1>出错了</h1>', {
      status: 500,
      headers: { 'content-type': 'text/html' },
    });
  }
}
```

如果某个位于 shell *之外* 的组件（即在 `<Suspense>` 边界内）抛出错误，React 不会停止渲染。这意味着 `onError` 回调会触发，但你的代码会继续运行而不会进入 `catch` 块。这是因为 React 会尝试在客户端恢复该错误，[如上所述。](#recovering-from-errors-outside-the-shell)

不过，如果你愿意，你可以利用发生错误这一事实来设置状态码：

```js {3,7,13}
async function handler(request) {
  try {
    let didError = false;
    const stream = await renderToReadableStream(<App />, {
      bootstrapScripts: ['/main.js'],
      onError(error) {
        didError = true;
        console.error(error);
        logServerCrashReport(error);
      }
    });
    return new Response(stream, {
      status: didError ? 500 : 200,
      headers: { 'content-type': 'text/html' },
    });
  } catch (error) {
    return new Response('<h1>出错了</h1>', {
      status: 500,
      headers: { 'content-type': 'text/html' },
    });
  }
}
```

这只会捕获在生成初始 shell 内容时发生的、shell 外部的错误，因此它并不完整。如果知道某些内容是否发生了错误至关重要，你可以把它移到 shell 中。

---

### 以不同方式处理不同错误 {/*handling-different-errors-in-different-ways*/}

你可以[创建自己的 `Error` 子类](https://javascript.info/custom-errors)，并使用 [`instanceof`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/instanceof) 运算符来检查抛出了哪种错误。例如，你可以定义一个自定义的 `NotFoundError` 并在组件中抛出它。然后你可以在 `onError` 中保存该错误，并根据错误类型在返回响应之前做不同处理：

```js {2-3,5-15,22,28,33}
async function handler(request) {
  let didError = false;
  let caughtError = null;

  function getStatusCode() {
    if (didError) {
      if (caughtError instanceof NotFoundError) {
        return 404;
      } else {
        return 500;
      }
    } else {
      return 200;
    }
  }

  try {
    const stream = await renderToReadableStream(<App />, {
      bootstrapScripts: ['/main.js'],
      onError(error) {
        didError = true;
        caughtError = error;
        console.error(error);
        logServerCrashReport(error);
      }
    });
    return new Response(stream, {
      status: getStatusCode(),
      headers: { 'content-type': 'text/html' },
    });
  } catch (error) {
    return new Response('<h1>出错了</h1>', {
      status: getStatusCode(),
      headers: { 'content-type': 'text/html' },
    });
  }
}
```

请记住，一旦你发出了 shell 并开始流式传输，就不能再更改状态码。

---

### 为爬虫和静态生成等待所有内容加载完成 {/*waiting-for-all-content-to-load-for-crawlers-and-static-generation*/}

流式传输提供了更好的用户体验，因为用户可以在内容变得可用时立即看到它。

然而，当爬虫访问你的页面，或者你在构建时生成页面时，你可能希望先让所有内容加载完成，然后再生成最终 HTML 输出，而不是逐步显示。

你可以通过等待 `stream.allReady` Promise 来等到所有内容加载完成：

```js {12-15}
async function handler(request) {
  try {
    let didError = false;
    const stream = await renderToReadableStream(<App />, {
      bootstrapScripts: ['/main.js'],
      onError(error) {
        didError = true;
        console.error(error);
        logServerCrashReport(error);
      }
    });
    let isCrawler = // ... 取决于你的机器人检测策略 ...
    if (isCrawler) {
      await stream.allReady;
    }
    return new Response(stream, {
      status: didError ? 500 : 200,
      headers: { 'content-type': 'text/html' },
    });
  } catch (error) {
    return new Response('<h1>出错了</h1>', {
      status: 500,
      headers: { 'content-type': 'text/html' },
    });
  }
}
```

普通访客会收到一个逐步加载内容的流。爬虫会在所有数据加载完成后收到最终 HTML 输出。不过，这也意味着爬虫必须等待*所有*数据，其中一些数据可能加载缓慢或出错。根据你的应用，你也可以选择同样把 shell 发送给爬虫。

---

### 中止服务端渲染 {/*aborting-server-rendering*/}

你可以通过超时强制服务端渲染“放弃”：

```js {3,4-6,9}
async function handler(request) {
  try {
    const controller = new AbortController();
    setTimeout(() => {
      controller.abort();
    }, 10000);

    const stream = await renderToReadableStream(<App />, {
      signal: controller.signal,
      bootstrapScripts: ['/main.js'],
      onError(error) {
        didError = true;
        console.error(error);
        logServerCrashReport(error);
      }
    });
    // ...
```

React 会把剩余的加载回退内容以 HTML 形式刷新出去，并会尝试在客户端渲染其余部分。
