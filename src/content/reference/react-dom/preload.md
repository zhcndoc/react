---
title: preload
---

<Note>

[基于 React 的框架](/learn/creating-a-react-app)通常会为你处理资源加载，因此你可能不需要自己调用此 API。有关详细信息，请查阅你的框架文档。

</Note>

<Intro>

`preload` 让你可以急切地获取你预计会使用的资源，例如样式表、字体或外部脚本。

```js
preload("https://example.com/font.woff2", {as: "font"});
```

</Intro>

<InlineToc />

---

## 参考 {/*reference*/}

### `preload(href, options)` {/*preload*/}

要预加载资源，请从 `react-dom` 调用 `preload` 函数。

```js
import { preload } from 'react-dom';

function AppRoot() {
  preload("https://example.com/font.woff2", {as: "font"});
  // ...
}

```

[在下面查看更多示例。](#usage)

`preload` 函数会向浏览器提供一个提示，告诉它应当开始下载给定资源，这可以节省时间。

#### 参数 {/*parameters*/}

* `href`: 字符串。你想要下载的资源 URL。
* `options`: 对象。它包含以下属性：
  *  `as`: 必需的字符串。资源类型。其[可能的值](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/link#as)为 `audio`、`document`、`embed`、`fetch`、`font`、`image`、`object`、`script`、`style`、`track`、`video`、`worker`。
  *  `crossOrigin`: 字符串。要使用的 [CORS 策略](https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes/crossorigin)。其可能的值为 `anonymous` 和 `use-credentials`。当 `as` 设置为 `"fetch"` 时这是必需的。
  *  `referrerPolicy`: 字符串。获取资源时发送的 [Referrer 标头](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/link#referrerpolicy)。其可能的值为 `no-referrer-when-downgrade`（默认值）、`no-referrer`、`origin`、`origin-when-cross-origin` 和 `unsafe-url`。
  *  `integrity`: 字符串。资源的加密哈希，用于[验证其真实性](https://developer.mozilla.org/en-US/docs/Web/Security/Subresource_Integrity)。
  *  `type`: 字符串。资源的 MIME 类型。
  *  `nonce`: 字符串。在使用严格的内容安全策略时，允许该资源的加密 [nonce](https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/nonce)。
  *  `fetchPriority`: 字符串。建议资源获取的相对优先级。可能的值为 `auto`（默认值）、`high` 和 `low`。
  *  `imageSrcSet`: 字符串。仅在 `as: "image"` 时使用。指定[图片的源集](https://developer.mozilla.org/en-US/docs/Learn/HTML/Multimedia_and_embedding/Responsive_images)。
  *  `imageSizes`: 字符串。仅在 `as: "image"` 时使用。指定[图片的尺寸](https://developer.mozilla.org/en-US/docs/Learn/HTML/Multimedia_and_embedding/Responsive_images)。

#### 返回值 {/*returns*/}

`preload` 不返回任何内容。

#### 注意事项 {/*caveats*/}

* 多次等效调用 `preload` 与单次调用效果相同。根据以下规则，`preload` 调用被视为等效：
  * 如果两次调用具有相同的 `href`，则它们等效，除非：
  * 如果 `as` 设置为 `image`，那么当两次调用具有相同的 `href`、`imageSrcSet` 和 `imageSizes` 时，它们才等效。
* 在浏览器中，你可以在任何情况下调用 `preload`：在渲染组件时、在 Effect 中、在事件处理函数中，等等。
* 在服务端渲染或渲染 Server Components 时，只有当你在渲染组件期间或在源自组件渲染的异步上下文中调用 `preload` 时，它才会生效。任何其他调用都会被忽略。

---

## 用法 {/*usage*/}

### 在渲染时预加载 {/*preloading-when-rendering*/}

如果你知道某个组件或其子组件会使用特定资源，那么在渲染该组件时调用 `preload`。

<Recipes titleText="预加载示例">

#### 预加载外部脚本 {/*preloading-an-external-script*/}

```js
import { preload } from 'react-dom';

function AppRoot() {
  preload("https://example.com/script.js", {as: "script"});
  return ...;
}
```

如果你想让浏览器立即开始执行脚本（而不仅仅是下载它），请改用 [`preinit`](/reference/react-dom/preinit)。如果你想加载一个 ESM 模块，请使用 [`preloadModule`](/reference/react-dom/preloadModule)。

<Solution />

#### 预加载样式表 {/*preloading-a-stylesheet*/}

```js
import { preload } from 'react-dom';

function AppRoot() {
  preload("https://example.com/style.css", {as: "style"});
  return ...;
}
```

如果你希望样式表立即被插入文档（这意味着浏览器会立即开始解析它，而不仅仅是下载它），请改用 [`preinit`](/reference/react-dom/preinit)。

<Solution />

#### 预加载字体 {/*preloading-a-font*/}

```js
import { preload } from 'react-dom';

function AppRoot() {
  preload("https://example.com/style.css", {as: "style"});
  preload("https://example.com/font.woff2", {as: "font"});
  return ...;
}
```

如果你预加载了样式表，最好也同时预加载该样式表引用的任何字体。这样，浏览器就可以在下载并解析样式表之前开始下载字体。

<Solution />

#### 预加载图片 {/*preloading-an-image*/}

```js
import { preload } from 'react-dom';

function AppRoot() {
  preload("/banner.png", {
    as: "image",
    imageSrcSet: "/banner512.png 512w, /banner1024.png 1024w",
    imageSizes: "(max-width: 512px) 512px, 1024px",
  });
  return ...;
}
```

预加载图片时，`imageSrcSet` 和 `imageSizes` 选项有助于浏览器[获取适合屏幕尺寸的正确图片](https://developer.mozilla.org/en-US/docs/Learn/HTML/Multimedia_and_embedding/Responsive_images)。

<Solution />

</Recipes>

### 在事件处理函数中预加载 {/*preloading-in-an-event-handler*/}

在切换到需要外部资源的页面或状态之前，在事件处理函数中调用 `preload`。这样会比你在新页面或状态的渲染期间调用它更早开始该过程。

```js
import { preload } from 'react-dom';

function CallToAction() {
  const onClick = () => {
    preload("https://example.com/wizardStyles.css", {as: "style"});
    startWizard();
  }
  return (
    <button onClick={onClick}>开始向导</button>
  );
}
```
