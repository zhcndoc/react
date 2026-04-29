---
script: "<script>"
---

<Intro>

内置浏览器 `<script>` 组件（[built-in browser `<script>` component](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/script)）可让你向文档中添加脚本。

```js
<script> alert("hi!") </script>
```

</Intro>

<InlineToc />

---

## 参考 {/*reference*/}

### `<script>` {/*script*/}

要向文档中添加内联或外部脚本，请渲染 [内置浏览器 `<script>` 组件](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/script)。你可以在任意组件中渲染 `<script>`，React 会在[某些情况下](#special-rendering-behavior)将相应的 DOM 元素放置到文档的 head 中，并对相同脚本进行去重。

```js
<script> alert("hi!") </script>
<script src="script.js" />
```

[查看更多示例。](#usage)

#### Props {/*props*/}

`<script>` 支持所有 [通用元素 props。](/reference/react-dom/components/common#common-props)

它应该 *要么* 有 `children`，*要么* 有 `src` prop。

* `children`：一个字符串。内联脚本的源代码。
* `src`：一个字符串。外部脚本的 URL。

其他受支持的 props：

* `async`：一个布尔值。允许浏览器将脚本执行推迟到文档其余部分处理完成之后——这是更利于性能的首选行为。
* `crossOrigin`：一个字符串。要使用的 [CORS 策略](https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes/crossorigin)。其可能的值为 `anonymous` 和 `use-credentials`。
* `fetchPriority`：一个字符串。允许浏览器在同时获取多个脚本时按优先级对脚本排序。可以是 `"high"`、`"low"` 或 `"auto"`（默认值）。
* `integrity`：一个字符串。脚本的加密哈希，用于[验证其真实性](https://developer.mozilla.org/en-US/docs/Web/Security/Subresource_Integrity)。
* `noModule`：一个布尔值。禁用支持 ES 模块的浏览器中的脚本——为不支持的浏览器提供回退脚本。
* `nonce`：一个字符串。在使用严格的内容安全策略时，允许该资源的加密 [nonce](https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/nonce)。
* `referrer`：一个字符串。说明在获取脚本及该脚本随后获取的任何资源时要发送[哪个 Referer 请求头](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/script#referrerpolicy)。
* `type`：一个字符串。说明脚本是[经典脚本、ES 模块还是导入映射](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/script/type)。

会禁用 React 对脚本的 [特殊处理](#special-rendering-behavior)的 props：

* `onError`：一个函数。脚本加载失败时调用。
* `onLoad`：一个函数。脚本完成加载时调用。

**不建议**与 React 一起使用的 props：

* `blocking`：一个字符串。如果设置为 `"render"`，会指示浏览器在脚本表加载完成之前不要渲染页面。React 使用 Suspense 提供更细粒度的控制。
* `defer`：一个字符串。阻止浏览器在文档加载完成之前执行脚本。与流式服务器渲染组件不兼容。请改用 `async` prop。

#### 特殊渲染行为 {/*special-rendering-behavior*/}

React 可以将 `<script>` 组件移动到文档的 `<head>` 中，并对相同脚本进行去重。

要启用此行为，请提供 `src` 和 `async={true}` props。若脚本具有相同的 `src`，React 会对其去重。`async` prop 必须为 true，才能安全地移动脚本。

这种特殊处理有两个注意事项：

* React 会忽略脚本渲染后对 props 的更改。（如果在开发环境中发生这种情况，React 会发出警告。）
* 即使渲染它的组件已卸载，React 也可能将脚本保留在 DOM 中。（这没有影响，因为脚本在插入 DOM 时只会执行一次。）

---

## 用法 {/*usage*/}

### 渲染外部脚本 {/*rendering-an-external-script*/}

如果某个组件依赖某些脚本才能正确显示，你可以在该组件中渲染一个 `<script>`。
然而，组件可能会在脚本完成加载之前就已提交。
你可以在 `load` 事件触发后开始依赖脚本内容，例如使用 `onLoad` prop。

React 会对具有相同 `src` 的脚本去重，即使多个组件都渲染它，也只会将其中一个插入到 DOM 中。

<SandpackWithHTMLOutput>

```js src/App.js active
import ShowRenderedHTML from './ShowRenderedHTML.js';

function Map({lat, long}) {
  return (
    <>
      <script async src="map-api.js" onLoad={() => console.log('script loaded')} />
      <div id="map" data-lat={lat} data-long={long} />
    </>
  );
}

export default function Page() {
  return (
    <ShowRenderedHTML>
      <Map />
    </ShowRenderedHTML>
  );
}
```

</SandpackWithHTMLOutput>

<Note>
当你想使用脚本时，调用 [preinit](/reference/react-dom/preinit) 函数会很有帮助。调用此函数可能让浏览器比你仅仅渲染一个 `<script>` 组件时更早开始获取脚本，例如通过发送一个 [HTTP Early Hints 响应](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/103)。
</Note>

### 渲染内联脚本 {/*rendering-an-inline-script*/}

要包含内联脚本，请渲染 `<script>` 组件，并将脚本源代码作为其 children。内联脚本不会被去重，也不会被移动到文档的 `<head>` 中。

<SandpackWithHTMLOutput>

```js src/App.js active
import ShowRenderedHTML from './ShowRenderedHTML.js';

function Tracking() {
  return (
    <script>
      ga('send', 'pageview');
    </script>
  );
}

export default function Page() {
  return (
    <ShowRenderedHTML>
      <h1>My Website</h1>
      <Tracking />
      <p>欢迎</p>
    </ShowRenderedHTML>
  );
}
```

</SandpackWithHTMLOutput>
