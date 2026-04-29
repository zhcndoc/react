---
meta: "<meta>"
---

<Intro>

[内置浏览器 `<meta>` 组件](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/meta) 可让你向文档添加元数据。

```js
<meta name="keywords" content="React, JavaScript, semantic markup, html" />
```

</Intro>

<InlineToc />

---

## 参考 {/*reference*/}

### `<meta>` {/*meta*/}

要添加文档元数据，请渲染 [内置浏览器 `<meta>` 组件](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/meta)。你可以在任何组件中渲染 `<meta>`，React 会始终将对应的 DOM 元素放置到文档的 head 中。

```js
<meta name="keywords" content="React, JavaScript, semantic markup, html" />
```

[查看更多示例。](#usage)

#### 属性 {/*props*/}

`<meta>` 支持所有 [通用元素属性。](/reference/react-dom/components/common#common-props)

它应当且仅应具有以下属性中的 *一个*：`name`、`httpEquiv`、`charset`、`itemProp`。`<meta>` 组件会根据指定的是这些属性中的哪一个而执行不同的操作。

* `name`：字符串。指定要附加到文档的 [元数据类型](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/meta/name)。
* `charset`：字符串。指定文档使用的字符集。唯一有效值是 `"utf-8"`。
* `httpEquiv`：字符串。指定文档处理指令。
* `itemProp`：字符串。指定文档中某个特定项目的元数据，而不是整个文档的元数据。
* `content`：字符串。与 `name` 或 `itemProp` 属性一起使用时，指定要附加的元数据；与 `httpEquiv` 属性一起使用时，指定该指令的行为。

#### 特殊的渲染行为 {/*special-rendering-behavior*/}

无论 `<meta>` 组件在 React 树中的何处渲染，React 都会始终将其对应的 DOM 元素放置在文档的 `<head>` 中。`<head>` 是 `<meta>` 在 DOM 中唯一有效的存在位置，但如果某个组件代表特定页面时能够自行渲染 `<meta>` 组件，这会更方便，也能保持可组合性。

对此有一个例外：如果 `<meta>` 具有 [`itemProp`](https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/itemprop) 属性，则不会有特殊行为，因为在这种情况下，它表示的不是关于文档的元数据，而是关于页面某个特定部分的元数据。

---

## 用法 {/*usage*/}

### 用元数据标注文档 {/*annotating-the-document-with-metadata*/}

你可以用关键词、摘要或作者姓名等元数据来标注文档。无论这些元数据在 React 树中的何处渲染，React 都会将其放置在文档的 `<head>` 中。

```html
<meta name="author" content="John Smith" />
<meta name="keywords" content="React, JavaScript, semantic markup, html" />
<meta name="description" content="React DOM 中 `<meta>` 组件的 API 参考" />
```

你可以在任何组件中渲染 `<meta>` 组件。React 会在文档的 `<head>` 中放置一个 `<meta>` DOM 节点。

<SandpackWithHTMLOutput>

```js src/App.js active
import ShowRenderedHTML from './ShowRenderedHTML.js';

export default function SiteMapPage() {
  return (
    <ShowRenderedHTML>
      <meta name="keywords" content="React" />
      <meta name="description" content="A site map for the React website" />
      <h1>Site Map</h1>
      <p>...</p>
    </ShowRenderedHTML>
  );
}
```

</SandpackWithHTMLOutput>

### 用元数据标注文档中的特定项目 {/*annotating-specific-items-within-the-document-with-metadata*/}

你可以将 `<meta>` 组件与 `itemProp` 属性一起使用，以便为文档中的特定项目添加元数据。在这种情况下，React *不会* 将这些标注放置到文档的 `<head>` 中，而是会像其他 React 组件一样放置它们。

```js
<section itemScope>
  <h3>标注特定项目</h3>
  <meta itemProp="description" content="使用 <meta> 和 itemProp 的 API 参考" />
  <p>...</p>
</section>
```
