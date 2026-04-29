---
title: "<title>"
---

<Intro>

内置浏览器 [`<title>` 组件](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/title) 允许你指定文档的标题。

```js
<title>My Blog</title>
```

</Intro>

<InlineToc />

---

## 参考 {/*reference*/}

### `<title>` {/*title*/}

要指定文档的标题，请渲染 [内置浏览器 `<title>` 组件](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/title)。你可以从任何组件中渲染 `<title>`，React 总会将对应的 DOM 元素放置到文档的 head 中。

```js
<title>My Blog</title>
```

[查看更多示例。](#usage)

#### Props {/*props*/}

`<title>` 支持所有 [通用元素 props。](/reference/react-dom/components/common#common-props)

* `children`：`<title>` 只接受文本作为子内容。这段文本将成为文档的标题。你也可以传入你自己的组件，只要它们最终只渲染文本。

#### 特殊渲染行为 {/*special-rendering-behavior*/}

React 总会将与 `<title>` 组件对应的 DOM 元素放在文档的 `<head>` 中，无论它在 React 树中的什么位置被渲染。`<head>` 是 `<title>` 在 DOM 中存在的唯一有效位置，但如果表示某个特定页面的组件可以自己渲染它的 `<title>`，这会很方便，并且更有利于组合。

这里有两个例外：
* 如果 `<title>` 位于 `<svg>` 组件中，那么就没有特殊行为，因为在这种情况下它表示的不是文档标题，而是该 SVG 图形的 [可访问性注释](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/title)。
* 如果 `<title>` 带有 [`itemProp`](https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/itemprop) 属性，那么就没有特殊行为，因为在这种情况下它表示的不是文档标题，而是页面某个特定部分的元数据。

<Pitfall>

一次只渲染一个 `<title>`。如果有多个组件同时渲染 `<title>` 标签，React 会把所有这些标题都放到文档的 head 中。发生这种情况时，浏览器和搜索引擎的行为未定义。

</Pitfall>

---

## 用法 {/*usage*/}

### 设置文档标题 {/*set-the-document-title*/}

从任意组件中渲染 `<title>` 组件，并将文本作为其子内容。React 会将一个 `<title>` DOM 节点放入文档的 `<head>` 中。

<SandpackWithHTMLOutput>

```js src/App.js active
import ShowRenderedHTML from './ShowRenderedHTML.js';

export default function ContactUsPage() {
  return (
    <ShowRenderedHTML>
      <title>My Site: Contact Us</title>
      <h1>Contact Us</h1>
      <p>Email us at support@example.com</p>
    </ShowRenderedHTML>
  );
}
```

</SandpackWithHTMLOutput>

### 在标题中使用变量 {/*use-variables-in-the-title*/}

`<title>` 组件的子内容必须是单个文本字符串。（或者是单个数字，或者是一个带有 `toString` 方法的单个对象。）这可能不太明显，但像这样使用 JSX 花括号：

```js
<title>Results page {pageNumber}</title> // 🔴 问题：这不是单个字符串
```

……实际上会导致 `<title>` 组件将一个包含两个元素的数组作为其子内容（字符串 `"Results page"` 和 `pageNumber` 的值）。这会导致错误。相反，请使用字符串插值向 `<title>` 传入单个字符串：

```js
<title>{`Results page ${pageNumber}`}</title>
```

