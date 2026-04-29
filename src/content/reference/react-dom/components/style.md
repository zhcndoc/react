---
style: "<style>"
---

<Intro>

内置浏览器的 `<style>` 组件（https://developer.mozilla.org/en-US/docs/Web/HTML/Element/style）可让你向文档中添加内联 CSS 样式表。

```js
<style>{` p { color: red; } `}</style>
```

</Intro>

<InlineToc />

---

## 参考 {/*reference*/}

### `<style>` {/*style*/}

要向文档中添加内联样式，请渲染内置浏览器的 `<style>` 组件（https://developer.mozilla.org/en-US/docs/Web/HTML/Element/style）。你可以在任何组件中渲染 `<style>`，React 会在[某些情况下](#special-rendering-behavior)将对应的 DOM 元素放入文档的 head 中，并对相同样式进行去重。

```js
<style>{` p { color: red; } `}</style>
```

[查看更多示例。](#usage)

#### Props {/*props*/}

`<style>` 支持所有[常见元素 props。](/reference/react-dom/components/common#common-props)

* `children`：字符串，必需。样式表的内容。
* `precedence`：字符串。告诉 React 在文档 `<head>` 中 `<style>` DOM 节点相对于其他节点的排序，这决定了哪个样式表可以覆盖另一个。React 会推断它最先发现的 precedence 值是“较低”的，后发现的 precedence 值是“较高”的。许多样式系统都可以很好地使用单个 precedence 值，因为样式规则是原子的。具有相同 precedence 的样式表会放在一起，无论它们是 `<link>`、内联 `<style>` 标签，还是通过 [`preinit`](/reference/react-dom/preinit) 函数加载的。
* `href`：字符串。允许 React 对具有相同 `href` 的样式进行[去重](#special-rendering-behavior)。
* `media`：字符串。将样式表限制为某个[媒体查询](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_media_queries/Using_media_queries)。
* `nonce`：字符串。在使用严格内容安全策略时，这是一个用于允许该资源的加密[nonce](https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/nonce)。
* `title`：字符串。指定[备用样式表](https://developer.mozilla.org/en-US/docs/Web/CSS/Alternative_style_sheets)的名称。

以下 props **不建议**与 React 一起使用：

* `blocking`：字符串。如果设置为 `"render"`，则指示浏览器在样式表加载完成之前不要渲染页面。React 通过 Suspense 提供更细粒度的控制。

#### 特殊的渲染行为 {/*special-rendering-behavior*/}

React 可以将 `<style>` 组件移动到文档的 `<head>` 中，对相同的样式表进行去重，并在样式表加载期间[挂起](/reference/react/Suspense)。

要启用此行为，请提供 `href` 和 `precedence` props。若样式具有相同的 `href`，React 会对它们进行去重。precedence prop 告诉 React 在文档 `<head>` 中 `<style>` DOM 节点相对于其他节点的排序，这决定了哪个样式表可以覆盖另一个。

这种特殊处理有三个注意事项：

* React 会忽略样式渲染后对 props 的更改。（如果在开发环境中发生这种情况，React 会发出警告。）
* 当使用 `precedence` prop 时，React 会丢弃所有多余的 props（除了 `href` 和 `precedence`）。
* 即使渲染它的组件已经卸载，React 也可能仍会将该样式保留在 DOM 中。

---

## 用法 {/*usage*/}

### 渲染内联 CSS 样式表 {/*rendering-an-inline-css-stylesheet*/}

如果某个组件依赖特定的 CSS 样式才能正确显示，你可以在该组件内部渲染一个内联样式表。

`href` prop 应该能唯一标识该样式表，因为 React 会对具有相同 `href` 的样式表进行去重。
如果你提供了 `precedence` prop，React 会根据这些值在组件树中出现的顺序重新排列内联样式表。

内联样式表在加载期间不会触发 Suspense 边界。
即使它们加载的是字体或图片等异步资源也是如此。

<SandpackWithHTMLOutput>

```js src/App.js active
import ShowRenderedHTML from './ShowRenderedHTML.js';
import { useId } from 'react';

function PieChart({data, colors}) {
  const id = useId();
  const stylesheet = colors.map((color, index) =>
    `#${id} .color-${index}: \{ color: "${color}"; \}`
  ).join();
  return (
    <>
      <style href={"PieChart-" + JSON.stringify(colors)} precedence="medium">
        {stylesheet}
      </style>
      <svg id={id}>
        …
      </svg>
    </>
  );
}

export default function App() {
  return (
    <ShowRenderedHTML>
      <PieChart data="..." colors={['red', 'green', 'blue']} />
    </ShowRenderedHTML>
  );
}
```

</SandpackWithHTMLOutput>
