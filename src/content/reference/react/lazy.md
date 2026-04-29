---
title: lazy
---

<Intro>

`lazy` 允许你延迟加载组件代码，直到它首次被渲染时才加载。

```js
const SomeComponent = lazy(load)
```

</Intro>

<InlineToc />

---

## 参考 {/*reference*/}

### `lazy(load)` {/*lazy*/}

在组件外部调用 `lazy` 来声明一个懒加载的 React 组件：

```js
import { lazy } from 'react';

const MarkdownPreview = lazy(() => import('./MarkdownPreview.js'));
```

[查看更多示例。](#usage)

#### 参数 {/*parameters*/}

* `load`：一个返回 [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) 或其他 *thenable*（一种带有 `then` 方法、类似 Promise 的对象）的函数。React 不会在你第一次尝试渲染返回的组件之前调用 `load`。在 React 首次调用 `load` 之后，它会等待其解析，然后将解析结果的 `.default` 作为 React 组件渲染。返回的 Promise 以及 Promise 的解析值都会被缓存，因此 React 不会多次调用 `load`。如果 Promise 被拒绝，React 会将拒绝原因 `throw` 出来，由最近的错误边界处理。

#### 返回值 {/*returns*/}

`lazy` 返回一个你可以在树中渲染的 React 组件。当懒加载组件的代码仍在加载时，尝试渲染它会 *suspend.* 使用 [`<Suspense>`](/reference/react/Suspense) 可以在加载期间显示加载指示器。

---

### `load` 函数 {/*load*/}

#### 参数 {/*load-parameters*/}

`load` 不接收任何参数。

#### 返回值 {/*load-returns*/}

你需要返回一个 [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) 或其他一些 *thenable*（一种带有 `then` 方法、类似 Promise 的对象）。它最终需要解析为一个对象，其 `.default` 属性是有效的 React 组件类型，例如函数、[`memo`](/reference/react/memo) 或 [`forwardRef`](/reference/react/forwardRef) 组件。

---

## 用法 {/*usage*/}

### 使用 Suspense 懒加载组件 {/*suspense-for-code-splitting*/}

通常，你会使用静态 [`import`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import) 声明来导入组件：

```js
import MarkdownPreview from './MarkdownPreview.js';
```

要将该组件的代码延迟到首次渲染时才加载，请将这个导入替换为：

```js
import { lazy } from 'react';

const MarkdownPreview = lazy(() => import('./MarkdownPreview.js'));
```

这段代码依赖于 [动态 `import()`,](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/import) 这可能需要你的打包工具或框架提供支持。使用这种模式要求你导入的懒加载组件是以 `default` 导出的形式导出的。

现在组件的代码按需加载了，你还需要指定在加载期间应显示什么内容。你可以通过将懒加载组件或其任意父组件包裹进 [`<Suspense>`](/reference/react/Suspense) 边界来实现：

```js {1,4}
<Suspense fallback={<Loading />}>
  <h2>预览</h2>
  <MarkdownPreview />
</Suspense>
```

在这个示例中，`MarkdownPreview` 的代码不会在你尝试渲染它之前加载。如果 `MarkdownPreview` 还未加载完成，将显示 `Loading` 作为替代。试着勾选复选框：

<Sandpack>

```js src/App.js
import { useState, Suspense, lazy } from 'react';
import Loading from './Loading.js';

const MarkdownPreview = lazy(() => delayForDemo(import('./MarkdownPreview.js')));

export default function MarkdownEditor() {
  const [showPreview, setShowPreview] = useState(false);
  const [markdown, setMarkdown] = useState('Hello, **world**!');
  return (
    <>
      <textarea value={markdown} onChange={e => setMarkdown(e.target.value)} />
      <label>
        <input type="checkbox" checked={showPreview} onChange={e => setShowPreview(e.target.checked)} />
        显示预览
      </label>
      <hr />
      {showPreview && (
        <Suspense fallback={<Loading />}>
          <h2>预览</h2>
          <MarkdownPreview markdown={markdown} />
        </Suspense>
      )}
    </>
  );
}

// 添加一个固定延迟，这样你就可以看到加载状态
function delayForDemo(promise) {
  return new Promise(resolve => {
    setTimeout(resolve, 2000);
  }).then(() => promise);
}
```

```js src/Loading.js
export default function Loading() {
  return <p><i>加载中...</i></p>;
}
```

```js src/MarkdownPreview.js
import { Remarkable } from 'remarkable';

const md = new Remarkable();

export default function MarkdownPreview({ markdown }) {
  return (
    <div
      className="content"
      dangerouslySetInnerHTML={{__html: md.render(markdown)}}
    />
  );
}
```

```json package.json hidden
{
  "dependencies": {
    "immer": "1.7.3",
    "react": "latest",
    "react-dom": "latest",
    "react-scripts": "latest",
    "remarkable": "2.0.1"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject"
  }
}
```

```css
label {
  display: block;
}

input, textarea {
  margin-bottom: 10px;
}

body {
  min-height: 200px;
}
```

</Sandpack>

这个演示会带有一个人为延迟进行加载。下次你取消勾选再勾选复选框时，`Preview` 会被缓存，因此不会再出现加载状态。要再次看到加载状态，请在 sandbox 中点击“Reset”。

[了解更多关于使用 Suspense 管理加载状态的信息。](/reference/react/Suspense)

---

## 故障排除 {/*troubleshooting*/}

### 我的 `lazy` 组件状态会意外重置 {/*my-lazy-components-state-gets-reset-unexpectedly*/}

不要在其他组件 *内部* 声明 `lazy` 组件：

```js {4-5}
import { lazy } from 'react';

function Editor() {
  // 🔴 不好：这会导致在重新渲染时所有状态都被重置
  const MarkdownPreview = lazy(() => import('./MarkdownPreview.js'));
  // ...
}
```

相反，请始终在模块顶层声明它们：

```js {3-4}
import { lazy } from 'react';

// ✅ 好：在组件外部声明懒加载组件
const MarkdownPreview = lazy(() => import('./MarkdownPreview.js'));

function Editor() {
  // ...
}
```
