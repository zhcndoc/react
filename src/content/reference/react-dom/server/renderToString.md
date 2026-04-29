---
title: renderToString
---

<Pitfall>

`renderToString` 不支持流式传输或等待数据。[查看替代方案。](#alternatives)

</Pitfall>

<Intro>

`renderToString` 会将 React 树渲染为 HTML 字符串。

```js
const html = renderToString(reactNode, options?)
```

</Intro>

<InlineToc />

---

## 参考 {/*reference*/}

### `renderToString(reactNode, options?)` {/*rendertostring*/}

在服务器上，调用 `renderToString` 将你的应用渲染为 HTML。

```js
import { renderToString } from 'react-dom/server';

const html = renderToString(<App />);
```

在客户端上，调用 [`hydrateRoot`](/reference/react-dom/client/hydrateRoot) 使服务器生成的 HTML 具有交互性。

[查看更多示例。](#usage)

#### 参数 {/*parameters*/}

* `reactNode`：你想要渲染为 HTML 的 React 节点。例如，像 `<App />` 这样的 JSX 节点。

* **可选** `options`：用于服务器渲染的对象。
  * **可选** `identifierPrefix`：React 用于通过 [`useId`.](/reference/react/useId) 生成 ID 的字符串前缀。适用于在同一页面上使用多个根时避免冲突。必须与传递给 [`hydrateRoot`.](/reference/react-dom/client/hydrateRoot#parameters) 的前缀相同

#### 返回值 {/*returns*/}

一个 HTML 字符串。

#### 注意事项 {/*caveats*/}

* `renderToString` 对 Suspense 的支持有限。如果某个组件挂起，`renderToString` 会立即将其回退内容作为 HTML 发送。

* `renderToString` 可以在浏览器中工作，但不[建议](#removing-rendertostring-from-the-client-code)在客户端代码中使用它。

---

## 用法 {/*usage*/}

### 将 React 树渲染为字符串形式的 HTML {/*rendering-a-react-tree-as-html-to-a-string*/}

调用 `renderToString` 将你的应用渲染为一个 HTML 字符串，然后可以将其随服务器响应一起发送：

```js {5-6}
import { renderToString } from 'react-dom/server';

// 路由处理程序语法取决于你的后端框架
app.use('/', (request, response) => {
  const html = renderToString(<App />);
  response.send(html);
});
```

这将生成 React 组件初始的、不可交互的 HTML 输出。在客户端上，你需要调用 [`hydrateRoot`](/reference/react-dom/client/hydrateRoot) 来 *hydrate* 这个服务器生成的 HTML，并使其具有交互性。


<Pitfall>

`renderToString` 不支持流式传输或等待数据。[查看替代方案。](#alternatives)

</Pitfall>

---

## 替代方案 {/*alternatives*/}

### 从 `renderToString` 迁移到服务器上的流式渲染 {/*migrating-from-rendertostring-to-a-streaming-method-on-the-server*/}

`renderToString` 会立即返回一个字符串，因此它不支持在内容加载时进行流式传输。

在可能的情况下，我们建议使用这些功能完整的替代方案：

* 如果你使用 Node.js，请使用 [`renderToPipeableStream`.](/reference/react-dom/server/renderToPipeableStream)
* 如果你使用 Deno，或者使用支持 [Web Streams](https://developer.mozilla.org/en-US/docs/Web/API/Streams_API) 的现代边缘运行时，请使用 [`renderToReadableStream`.](/reference/react-dom/server/renderToReadableStream)

如果你的服务器环境不支持流，你可以继续使用 `renderToString`。

---

### 从 `renderToString` 迁移到服务器上的静态预渲染 {/*migrating-from-rendertostring-to-a-static-prerender-on-the-server*/}

`renderToString` 会立即返回一个字符串，因此它不支持等待数据加载来生成静态 HTML。

我们建议使用这些功能完整的替代方案：

* 如果你使用 Node.js，请使用 [`prerenderToNodeStream`.](/reference/react-dom/static/prerenderToNodeStream)
* 如果你使用 Deno，或者使用支持 [Web Streams](https://developer.mozilla.org/en-US/docs/Web/API/Streams_API) 的现代边缘运行时，请使用 [`prerender`.](/reference/react-dom/static/prerender)

如果你的静态站点生成环境不支持流，你可以继续使用 `renderToString`。

---

### 从客户端代码中移除 `renderToString` {/*removing-rendertostring-from-the-client-code*/}

有时，`renderToString` 会在客户端上用于将某个组件转换为 HTML。

```js {1-2}
// 🚩 不必要：在客户端上使用 renderToString
import { renderToString } from 'react-dom/server';

const html = renderToString(<MyIcon />);
console.log(html); // 例如，"<svg>...</svg>"
```

在**客户端**导入 `react-dom/server` 会不必要地增大你的包体积，应当避免。如果你需要在浏览器中将某个组件渲染为 HTML，请使用 [`createRoot`](/reference/react-dom/client/createRoot) 并从 DOM 中读取 HTML：

```js
import { createRoot } from 'react-dom/client';
import { flushSync } from 'react-dom';

const div = document.createElement('div');
const root = createRoot(div);
flushSync(() => {
  root.render(<MyIcon />);
});
console.log(div.innerHTML); // 例如，"<svg>...</svg>"
```

[`flushSync`](/reference/react-dom/flushSync) 调用是必要的，这样在读取其 [`innerHTML`](https://developer.mozilla.org/en-US/docs/Web/API/Element/innerHTML) 属性之前，DOM 才会更新。

---

## 故障排除 {/*troubleshooting*/}

### 当某个组件挂起时，HTML 总是包含回退内容 {/*when-a-component-suspends-the-html-always-contains-a-fallback*/}

`renderToString` 并不完全支持 Suspense。

如果某个组件挂起（例如，因为它是用 [`lazy`](/reference/react/lazy) 定义的，或者会获取数据），`renderToString` 不会等待其内容解析。相反，`renderToString` 会找到其上方最近的 [`<Suspense>`](/reference/react/Suspense) 边界，并在 HTML 中渲染其 `fallback` 属性。直到客户端代码加载之前，内容都不会出现。

要解决这个问题，请使用[推荐的流式解决方案。](#alternatives)对于服务器端渲染，它们可以在服务器上按块流式输出内容，随着内容解析逐步发送给用户，让用户在客户端代码加载之前就能看到页面逐渐填充完成。对于静态站点生成，它们可以在生成静态 HTML 之前等待所有内容解析完成。

