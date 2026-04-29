---
link: "<link>"
---

<Intro>

内置浏览器 `<link>` 组件](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/link) 允许你使用外部资源，例如样式表，或用链接元数据对文档进行注释。

```js
<link rel="icon" href="favicon.ico" />
```

</Intro>

<InlineToc />

---

## 参考 {/*reference*/}

### `<link>` {/*link*/}

要链接到样式表、字体和图标等外部资源，或用链接元数据对文档进行注释，请渲染 [内置浏览器 `<link>` 组件](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/link)。你可以在任何组件中渲染 `<link>`，React [在大多数情况下](#special-rendering-behavior) 会将相应的 DOM 元素放置到文档的 head 中。

```js
<link rel="icon" href="favicon.ico" />
```

[查看更多示例。](#usage)

#### 属性 {/*props*/}

`<link>` 支持所有 [常见元素属性。](/reference/react-dom/components/common#common-props)

* `rel`：字符串，必填。指定 [与资源的关系](https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes/rel)。React 会 [区别对待 `rel="stylesheet"` 的链接](#special-rendering-behavior)，与其他链接不同。

以下属性适用于 `rel="stylesheet"`：

* `precedence`：字符串。告诉 React 该 `<link>` DOM 节点相对于文档 `<head>` 中其他节点的优先级，这决定了哪个样式表可以覆盖另一个。React 会推断它先发现的 precedence 值为“较低”，后发现的 precedence 值为“较高”。许多样式系统只使用单个 precedence 值也能正常工作，因为样式规则是原子的。具有相同 precedence 的样式表会被放在一起，无论它们是 `<link>`、内联 `<style>` 标签，还是通过 [`preinit`](/reference/react-dom/preinit) 函数加载。
* `media`：字符串。将样式表限制为某个 [媒体查询](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_media_queries/Using_media_queries)。
* `title`：字符串。指定 [备用样式表](https://developer.mozilla.org/en-US/docs/Web/CSS/Alternative_style_sheets) 的名称。

以下属性适用于 `rel="stylesheet"`，但会禁用 React 对样式表的 [特殊处理](#special-rendering-behavior)：

* `disabled`：布尔值。禁用该样式表。
* `onError`：函数。样式表加载失败时调用。
* `onLoad`：函数。样式表加载完成时调用。

以下属性适用于 `rel="preload"` 或 `rel="modulepreload"`：

* `as`：字符串。资源类型。可选值为 `audio`、`document`、`embed`、`fetch`、`font`、`image`、`object`、`script`、`style`、`track`、`video`、`worker`。
* `imageSrcSet`：字符串。仅在 `as="image"` 时适用。指定 [图像的源集](https://developer.mozilla.org/en-US/docs/Learn/HTML/Multimedia_and_embedding/Responsive_images)。
* `imageSizes`：字符串。仅在 `as="image"` 时适用。指定 [图像的尺寸](https://developer.mozilla.org/en-US/docs/Learn/HTML/Multimedia_and_embedding/Responsive_images)。

以下属性适用于 `rel="icon"` 或 `rel="apple-touch-icon"`：

* `sizes`：字符串。[图标的尺寸](https://developer.mozilla.org/en-US/docs/Learn/HTML/Multimedia_and_embedding/Responsive_images)。

以下属性在所有情况下都适用：

* `href`：字符串。链接资源的 URL。
*  `crossOrigin`：字符串。要使用的 [CORS 策略](https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes/crossorigin)。可选值为 `anonymous` 和 `use-credentials`。当 `as` 被设置为 `"fetch"` 时必填。
*  `referrerPolicy`：字符串。获取资源时发送的 [Referrer 头](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/link#referrerpolicy)。可选值为 `no-referrer-when-downgrade`（默认）、`no-referrer`、`origin`、`origin-when-cross-origin` 和 `unsafe-url`。
* `fetchPriority`：字符串。提示获取资源时的相对优先级。可选值为 `auto`（默认）、`high` 和 `low`。
* `hrefLang`：字符串。链接资源的语言。
* `integrity`：字符串。资源的加密哈希，用于 [验证其真实性](https://developer.mozilla.org/en-US/docs/Web/Security/Subresource_Integrity)。
* `type`：字符串。链接资源的 MIME 类型。

**不建议**与 React 一起使用的属性：

* `blocking`：字符串。如果设置为 `"render"`，会指示浏览器在样式表加载完成之前不要渲染页面。React 通过 Suspense 提供更细粒度的控制。

#### 特殊渲染行为 {/*special-rendering-behavior*/}

无论 `<link>` 组件在 React 树中的哪个位置渲染，React 都会始终将其对应的 DOM 元素放置在文档的 `<head>` 内。`<head>` 是 `<link>` 在 DOM 中唯一有效的存在位置，不过如果某个代表特定页面的组件能够自己渲染 `<link>` 组件，这样会更方便，也更具组合性。

对此有少数例外：

* 如果 `<link>` 具有 `rel="stylesheet"` 属性，那么它还必须具有 `precedence` 属性才能获得这种特殊行为。这是因为文档中样式表的顺序很重要，因此 React 需要知道如何将该样式表相对于其他样式表进行排序，而这由 `precedence` 属性指定。如果省略 `precedence` 属性，则不会有特殊行为。
* 如果 `<link>` 具有 [`itemProp`](https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/itemprop) 属性，则不会有特殊行为，因为在这种情况下它不适用于整个文档，而是表示页面某一特定部分的元数据。
* 如果 `<link>` 具有 `onLoad` 或 `onError` 属性，因为在这种情况下你是在 React 组件中手动管理链接资源的加载。

#### 样式表的特殊行为 {/*special-behavior-for-stylesheets*/}

此外，如果 `<link>` 指向的是样式表（即其属性中包含 `rel="stylesheet"`），React 会以以下方式特殊处理：

* 渲染 `<link>` 的组件在样式表加载期间会 [挂起](/reference/react/Suspense)。
* 如果多个组件渲染指向同一样式表的链接，React 会对它们去重，并且只在 DOM 中放置一个 link。若两个链接具有相同的 `href` 属性，则视为相同。

这种特殊行为有两个例外：

* 如果该 link 没有 `precedence` 属性，则不会有特殊行为，因为文档中样式表的顺序很重要，因此 React 需要知道如何将该样式表相对于其他样式表进行排序，而这由 `precedence` 属性指定。
* 如果你提供了 `onLoad`、`onError` 或 `disabled` 中的任意属性，则不会有特殊行为，因为这些属性表明你正在组件中手动管理样式表的加载。

这种特殊处理有两个注意事项：

* React 会忽略 link 渲染后对属性的更改。（如果在开发环境中发生这种情况，React 会发出警告。）
* 即使渲染它的组件已经卸载，React 也可能仍将该 link 保留在 DOM 中。

---

## 用法 {/*usage*/}

### 链接相关资源 {/*linking-to-related-resources*/}

你可以使用指向图标、规范 URL 或 pingback 等相关资源的链接来为文档添加注释。无论它在 React 树中的哪个位置渲染，React 都会将这些元数据放置到文档的 `<head>` 中。

<SandpackWithHTMLOutput>

```js src/App.js active
import ShowRenderedHTML from './ShowRenderedHTML.js';

export default function BlogPage() {
  return (
    <ShowRenderedHTML>
      <link rel="icon" href="favicon.ico" />
      <link rel="pingback" href="http://www.example.com/xmlrpc.php" />
      <h1>我的博客</h1>
      <p>...</p>
    </ShowRenderedHTML>
  );
}
```

</SandpackWithHTMLOutput>

### 链接到样式表 {/*linking-to-a-stylesheet*/}

如果某个组件要正确显示，需要依赖某个样式表，那么你可以在该组件中渲染指向该样式表的链接。样式表加载期间，你的组件会 [挂起](/reference/react/Suspense)。你必须提供 `precedence` 属性，它会告诉 React 将该样式表放在相对于其他样式表的什么位置——优先级更高的样式表可以覆盖优先级更低的样式表。

<Note>
当你想使用样式表时，调用 [preinit](/reference/react-dom/preinit) 函数可能会更有益。调用此函数可能会让浏览器比你直接渲染 `<link>` 组件时更早开始获取样式表，例如通过发送 [HTTP Early Hints 响应](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/103)。
</Note>

<SandpackWithHTMLOutput>

```js src/App.js active
import ShowRenderedHTML from './ShowRenderedHTML.js';

export default function SiteMapPage() {
  return (
    <ShowRenderedHTML>
      <link rel="stylesheet" href="sitemap.css" precedence="medium" />
      <p>...</p>
    </ShowRenderedHTML>
  );
}
```

</SandpackWithHTMLOutput>

### 控制样式表优先级 {/*controlling-stylesheet-precedence*/}

样式表之间可能会发生冲突，而当冲突发生时，浏览器会采用文档中后出现的那个。React 允许你通过 `precedence` 属性控制样式表的顺序。在这个示例中，三个组件渲染样式表，而具有相同 precedence 的样式表会在 `<head>` 中被分组在一起。

<SandpackWithHTMLOutput>

```js src/App.js active
import ShowRenderedHTML from './ShowRenderedHTML.js';

export default function HomePage() {
  return (
    <ShowRenderedHTML>
      <FirstComponent />
      <SecondComponent />
      <ThirdComponent/>
      ...
    </ShowRenderedHTML>
  );
}

function FirstComponent() {
  return <link rel="stylesheet" href="first.css" precedence="first" />;
}

function SecondComponent() {
  return <link rel="stylesheet" href="second.css" precedence="second" />;
}

function ThirdComponent() {
  return <link rel="stylesheet" href="third.css" precedence="first" />;
}

```

</SandpackWithHTMLOutput>

注意 `precedence` 值本身是任意的，命名由你决定。React 会推断它先发现的 precedence 值为“较低”，后发现的 precedence 值为“较高”。

### 去重的样式表渲染 {/*deduplicated-stylesheet-rendering*/}

如果你从多个组件渲染同一样式表，React 会只在文档 head 中放置一个 `<link>`。

<SandpackWithHTMLOutput>

```js src/App.js active
import ShowRenderedHTML from './ShowRenderedHTML.js';

export default function HomePage() {
  return (
    <ShowRenderedHTML>
      <Component />
      <Component />
      ...
    </ShowRenderedHTML>
  );
}

function Component() {
  return <link rel="stylesheet" href="styles.css" precedence="medium" />;
}
```

</SandpackWithHTMLOutput>

### 使用链接为文档中的特定项目添加注释 {/*annotating-specific-items-within-the-document-with-links*/}

你可以将 `<link>` 组件与 `itemProp` 属性一起使用，为文档中的特定项目添加指向相关资源的链接。在这种情况下，React **不会** 将这些注释放入文档 `<head>`，而是会像处理任何其他 React 组件一样处理它们。

```js
<section itemScope>
  <h3>为特定项目添加注释</h3>
  <link itemProp="author" href="http://example.com/" />
  <p>...</p>
</section>
```
