---
title: renderToStaticMarkup
---

<Intro>

`renderToStaticMarkup` 将一个非交互式 React 树渲染为 HTML 字符串。

```js
const html = renderToStaticMarkup(reactNode, options?)
```

</Intro>

<InlineToc />

---

## 参考 {/*reference*/}

### `renderToStaticMarkup(reactNode, options?)` {/*rendertostaticmarkup*/}

在服务器上，调用 `renderToStaticMarkup` 将你的应用渲染为 HTML。

```js
import { renderToStaticMarkup } from 'react-dom/server';

const html = renderToStaticMarkup(<Page />);
```

它会生成你的 React 组件的非交互式 HTML 输出。

[查看更多示例。](#usage)

#### 参数 {/*parameters*/}

* `reactNode`：你希望渲染为 HTML 的 React 节点。例如，一个像 `<Page />` 这样的 JSX 节点。
* **可选** `options`：用于服务端渲染的对象。
  * **可选** `identifierPrefix`：React 用于通过 [`useId`.](/reference/react/useId) 生成 ID 的字符串前缀。在同一页面上使用多个根时，可用于避免冲突。

#### 返回值 {/*returns*/}

一个 HTML 字符串。

#### 注意事项 {/*caveats*/}

* `renderToStaticMarkup` 的输出不能被 hydrate。

* `renderToStaticMarkup` 对 Suspense 的支持有限。如果某个组件挂起，`renderToStaticMarkup` 会立即将其 fallback 作为 HTML 发送出去。

* `renderToStaticMarkup` 可以在浏览器中工作，但不建议在客户端代码中使用。如果你需要在浏览器中将组件渲染为 HTML，[请将其渲染到 DOM 节点中以获取 HTML。](/reference/react-dom/server/renderToString#removing-rendertostring-from-the-client-code)

---

## 用法 {/*usage*/}

### 将非交互式 React 树渲染为 HTML 字符串 {/*rendering-a-non-interactive-react-tree-as-html-to-a-string*/}

调用 `renderToStaticMarkup` 将你的应用渲染为一个 HTML 字符串，你可以随服务器响应一起发送：

```js {5-6}
import { renderToStaticMarkup } from 'react-dom/server';

// 路由处理程序语法取决于你的后端框架
app.use('/', (request, response) => {
  const html = renderToStaticMarkup(<Page />);
  response.send(html);
});
```

这将生成你的 React 组件初始的、非交互式的 HTML 输出。

<Pitfall>

此方法会渲染**无法被 hydrate 的非交互式 HTML。** 这在你想把 React 作为一个简单的静态页面生成器，或者你正在渲染完全静态的内容（例如电子邮件）时很有用。

交互式应用应在服务器上使用 [`renderToString`](/reference/react-dom/server/renderToString)，并在客户端使用 [`hydrateRoot`](/reference/react-dom/client/hydrateRoot)。

</Pitfall>
