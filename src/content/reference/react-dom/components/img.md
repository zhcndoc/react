---
title: "<img>"
---

<Intro>

内置的浏览器 [`<img>` 组件](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/img)可让你嵌入图片。

```js
<img src="photo.jpg" alt="A person walking through a park" />
```

</Intro>

<InlineToc />

---

## 参考 {/*reference*/}

### `<img>` {/*img*/}

要显示图片，请渲染内置的浏览器 [`<img>` 组件](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/img)。

```js
<img src="photo.jpg" alt="A person walking through a park" />
```

[查看更多示例。](#usage)

#### 属性 {/*props*/}

`<img>` 支持所有[通用元素属性。](/reference/react-dom/components/common#common-props)

* `alt`：字符串。指定图片的替代文本。纯装饰图片应使用空字符串。
* `crossOrigin`：字符串。指定获取图片时使用的 [CORS 策略](https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes/crossorigin)。可选值为 `anonymous` 和 `use-credentials`。
* `decoding`：字符串。建议浏览器是否应在显示其他内容前等待图片解码。可选值为 `async`、`sync` 和 `auto`（默认值）。
* `fetchPriority`：字符串。建议获取图片的相对优先级。可选值为 `high`、`low` 和 `auto`（默认值）。服务端渲染期间，`fetchPriority="low"` 也会阻止 React [自动预加载图片。](#controlling-image-preloading-during-server-rendering)
* `height`：数字或字符串。指定图片的渲染高度。
* `loading`：字符串。指定浏览器是否应延迟加载图片，直到图片接近视口。可选值为 `eager`（默认值）和 `lazy`。设置 `loading="lazy"` 会阻止 React [自动预加载图片。](#controlling-image-preloading-during-server-rendering)
* `onError`：[事件处理函数](/reference/react-dom/components/common#event-handler)。图片加载失败时触发。
* `onLoad`：[事件处理函数](/reference/react-dom/components/common#event-handler)。图片加载完成时触发。传入 `onLoad` 会阻止 React [在客户端渲染的 View Transition 更新期间等待图片。](#waiting-for-an-image-during-a-view-transition)
* `referrerPolicy`：字符串。指定获取图片时要发送的[引用来源信息](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/img#referrerpolicy)。
* `sizes`：字符串。指定不同页面布局下的图片尺寸。与 `srcSet` 一起使用。
* `src`：字符串。指定图片的 URL。
* `srcSet`：字符串。指定一个或多个供浏览器选择的候选图片来源。
* `useMap`：字符串。将图片关联到[客户端图片映射](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/map)。
* `width`：数字或字符串。指定图片的渲染宽度。

#### 注意事项 {/*caveats*/}

* 不要给 `src` 传入空字符串，这可能会导致浏览器再次请求当前页面。React 会在开发环境中发出警告，并省略该属性。若要不渲染图片，请省略 `<img>`，或给 `src` 传入 `null`。
* `<img>` 不能包含子元素，也不能使用 `dangerouslySetInnerHTML`。如果传入其中任一项，React 都会抛出错误。
* `fetchPriority="low"` 不会阻止 React 在客户端渲染的 View Transition 更新期间等待图片加载。使用 `loading="lazy"` 或 `onLoad` 处理函数可以选择跳过此行为。

---

## 用法 {/*usage*/}

### 显示图片 {/*displaying-an-image*/}

将图片 URL 传给 `src`，并将文本描述传给 `alt`：

<Sandpack>

```js
export default function Profile() {
  return (
    <img
      src="https://react.dev/images/docs/scientists/yXOvdOSs.jpg"
      alt="Hedy Lamarr"
      width={100}
      height={100}
    />
  );
}
```

```css
img {
  border-radius: 50%;
  object-fit: cover;
}
```

</Sandpack>

如果知道图片尺寸，请指定 `width` 和 `height`，这样浏览器就能在图片加载前预留空间。对于纯装饰图片，请传入 `alt=""`，让屏幕阅读器忽略它。

---

### 控制服务端渲染期间的图片预加载 {/*controlling-image-preloading-during-server-rendering*/}

默认情况下，服务端渲染期间 React 会自动为 `<img>` 生成预加载提示。这样浏览器在遇到渲染后 HTML 中的 `<img>` 之前就可以开始获取图片。

对于不应接收此提示的图片，请添加 `loading="lazy"` 或 `fetchPriority="low"`：

```js
function ProductPage() {
  return (
    <>
      <img src="hero.jpg" alt="Featured product" />
      <img src="thumbnail.jpg" alt="Related product" loading="lazy" />
      <img src="secondary.jpg" alt="Another product" fetchPriority="low" />
    </>
  );
}
```

在此示例中，React 只会为 `hero.jpg` 生成预加载提示。根据所用的服务端 API 或框架，React 可能会渲染与此元素等效的内容：

```html
<link rel="preload" as="image" href="hero.jpg" />
```

React 也可能改为在 `Link` 响应头中提供相同的提示。其他两张图片会在渲染后的 HTML 中保留 `loading` 和 `fetchPriority` 属性，但 React 不会为它们生成预加载提示。`loading="lazy"` 属性会让浏览器延迟加载图片，直到图片接近视口。`fetchPriority="low"` 属性允许图片立即开始加载，但会告知浏览器以较低优先级获取它。

如果图片位于 `<picture>` 或 `<noscript>` 元素内，或者其 `src` 或 `srcSet` 是 data URL，React 也不会自动预加载图片。

如果你通过框架或组件库渲染图片，请查阅其文档了解默认行为。React 会根据底层 `<img>` 的属性决定是否生成自动预加载提示。例如，图片组件可能默认添加 `loading="lazy"`，并另外提供一个选项，以显式预加载所选图片。

要创建显式的预加载提示，请调用 [`preload`](/reference/react-dom/preload)。

---

### 在 View Transition 期间等待图片 {/*waiting-for-an-image-during-a-view-transition*/}

在 [`<ViewTransition>`](/reference/react/ViewTransition) 更新期间，React 会等待最多 500 毫秒，让可见图片加载完成后再开始动画。这包括新渲染的 `<img>` 元素，以及 `src` 或 `srcSet` 发生变化的现有图片。设置 `loading="lazy"` 或添加 `onLoad` 处理函数可以让图片跳过等待。

比较同一张图片在 Suspense 边界的内容于 `<ViewTransition>` 内外显示时的效果：

<Sandpack>

```js
import {
  ViewTransition,
  Suspense,
  use,
  useState,
} from 'react';
import { fetchImageSrc } from './image.js';

function Profile({ cacheKey }) {
  const src = use(fetchImageSrc(cacheKey));
  return (
    <div className="card">
      <img src={src} alt="" width={80} height={80} />
      <p>Jack Pope</p>
    </div>
  );
}

function ProfilePlaceholder() {
  return (
    <div className="card">
      <div className="avatar-placeholder" />
      <p className="name-placeholder">&nbsp;</p>
    </div>
  );
}

export default function App() {
  const [showWithTransition, setShowWithTransition] =
    useState(false);
  const [showWithoutTransition, setShowWithoutTransition] =
    useState(false);
  return (
    <>
      <button onClick={() => setShowWithTransition(true)}>
        Show profile with View Transition
      </button>
      {showWithTransition && (
        <ViewTransition>
          <Suspense fallback={<ProfilePlaceholder />}>
            <Profile cacheKey="with-transition" />
          </Suspense>
        </ViewTransition>
      )}
      <hr />
      <button onClick={() => setShowWithoutTransition(true)}>
        Show profile without View Transition
      </button>
      {showWithoutTransition && (
        <Suspense fallback={<ProfilePlaceholder />}>
          <Profile cacheKey="without-transition" />
        </Suspense>
      )}
    </>
  );
}
```

```js src/image.js hidden
// Normally, the caching logic would be inside a framework.
const cache = new Map();

export function fetchImageSrc(cacheKey) {
  if (!cache.has(cacheKey)) {
    cache.set(cacheKey, loadImageSrc());
  }
  return cache.get(cacheKey);
}

async function loadImageSrc() {
  // Delay the response so the Suspense fallback is visible.
  await new Promise(resolve => setTimeout(resolve, 1000));
  // Add a unique parameter so the image isn't cached.
  return 'https://react.dev/images/team/jack-pope.jpg?t=' + Date.now();
}
```

```css
#root {
  min-height: 390px;
}
.card {
  margin-top: 1em;
}
.card img {
  display: block;
  border-radius: 50%;
  background: #dfe3e9;
}
.card p {
  font-weight: bold;
}
.avatar-placeholder {
  width: 80px;
  height: 80px;
  border-radius: 50%;
  background: #dfe3e9;
}
.name-placeholder {
  width: 90px;
  border-radius: 4px;
  background: #dfe3e9;
}
hr {
  margin: 16px 0;
}
```

```json package.json hidden
{
  "dependencies": {
    "react": "19.3.0",
    "react-dom": "19.3.0",
    "react-scripts": "latest"
  }
}
```

</Sandpack>

使用 `<ViewTransition>` 时，React 会在图片加载期间将骨架屏保持最多 500 毫秒，这样卡片显示时图片也已就绪。不使用 `<ViewTransition>` 时，Suspense 会在 Promise 解析后立即停止显示骨架屏。如果图片仍在加载，卡片会先出现，图片随后再显示。
