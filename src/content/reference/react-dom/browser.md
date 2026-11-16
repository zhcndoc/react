---
title: browser
---

<Intro>

`browser` 让你在服务端渲染期间将组件标记为仅在浏览器中渲染。

```js
use(browser(reason?))
```

</Intro>

<InlineToc />

---

## 参考 {/*reference*/}

### `browser(reason?)` {/*browser*/}

在 [`use`](/reference/react/use) 中调用 `browser`，即可在服务端渲染期间将组件标记为仅在浏览器中渲染：

```js
import { use } from 'react';
import { browser } from 'react-dom';

function BrowserOnly() {
  use(browser('This component requires browser APIs.'));
  return <BrowserContent />;
}
```

服务端渲染期间，`use(browser())` 会暂停该组件的渲染，并保留最近的 [`<Suspense>`](/reference/react/Suspense) 边界的回退内容。在浏览器中，`use(browser())` 会返回 `undefined`，因此组件会正常渲染。

[查看更多示例。](#usage)

#### 参数 {/*parameters*/}

* **可选** `reason`：用于说明内容为何需要在浏览器中渲染的字符串或函数。该字符串或函数的返回值会成为传给 [`onBrowserBailout`](#reporting-browser-only-rendering-on-the-server) 的 `Error` 的 `cause`。每当服务端渲染器遇到 `browser` 返回的值时，React 都会调用 reason 函数；在浏览器中则不会调用。如果生成 reason 的开销较大，可以传入 `() => new Error(...)` 这样的函数。

#### 返回值 {/*returns*/}

`browser` 会返回一个不透明值，你可以在组件中将其传给 `use`，也可以在[中止服务端渲染](#aborting-pending-server-rendering-for-the-browser)时将其用作 reason。在浏览器中，将此值传给 `use` 会返回 `undefined`。

#### 注意事项 {/*caveats*/}

* 服务端渲染期间，`use(browser())` 必须位于 `<Suspense>` 边界内，否则服务端渲染会失败。
* 必须从 [Client Component](/reference/rsc/use-client)（而不是 [Server Component](/reference/rsc/server-components)）中调用 `use(browser())`。
* 单独调用 `browser()` 不会产生任何效果。要将组件标记为仅在浏览器中渲染，请把 `browser` 返回的值传给 `use`。不要抛出该值。

---

## 用法 {/*usage*/}

### 仅在浏览器中渲染内容 {/*rendering-content-only-in-the-browser*/}

在只应于浏览器中渲染的组件内，在 `use` 中调用 `browser`：

你可以用这种方式替代检查 `typeof window`、等待 [`Effect`](/reference/react/useEffect) 设置 mounted state，或使用框架选项禁用服务端渲染。

点击 **重新加载**，即可在初始 HTML 中看到加载回退内容。hydration 完成后，React 会显示从 `localStorage` 加载的草稿。

<Sandpack>

```js src/App.js active
import { Suspense, use, useState } from 'react';
import { browser } from 'react-dom';

function SavedDraft() {
  use(browser('The draft is stored in localStorage.'));
  const [draft, setDraft] = useState(
    () => localStorage.getItem('draft') ?? ''
  );

  function handleChange(event) {
    const nextDraft = event.target.value;
    setDraft(nextDraft);
    localStorage.setItem('draft', nextDraft);
  }

  return (
    <label>
      Draft:
      <textarea
        value={draft}
        onChange={handleChange}
        rows={4}
        cols={30}
      />
    </label>
  );
}

export default function App() {
  return (
    <>
      <h1>Saved draft</h1>
      <Suspense fallback={<p>Loading draft...</p>}>
        <SavedDraft />
      </Suspense>
    </>
  );
}
```

```js src/Document.js hidden
import App from './App.js';

export default function Document() {
  return (
    <html lang="en">
      <head>
        <title>Saved draft</title>
        <style>{`
          h1 { font-size: 24px; margin-top: 0; }
          label, textarea { display: block; }
          textarea { margin-top: 5px; }
        `}</style>
      </head>
      <body>
        <App />
      </body>
    </html>
  );
}
```

```js src/index.js hidden
import { hydrateRoot } from 'react-dom/client';
import { renderToReadableStream } from 'react-dom/server';
import Document from './Document.js';
import { flushReadableStreamToFrame } from './demo-helpers.js';
import './styles.css';

async function main(frame) {
  const stream = await renderToReadableStream(<Document />);
  await flushReadableStreamToFrame(stream, frame);

  // Wait so both the fallback and hydrated content are visible.
  await new Promise(resolve => setTimeout(resolve, 1200));
  hydrateRoot(frame.contentDocument, <Document />);
}

main(document.getElementById('preview'));
```

```js src/demo-helpers.js hidden
export async function flushReadableStreamToFrame(readable, frame) {
  const doc = frame.contentWindow.document;
  const decoder = new TextDecoder();
  const reader = readable.getReader();

  while (true) {
    const {done, value} = await reader.read();
    if (done) {
      break;
    }
    doc.write(decoder.decode(value, {stream: true}));
  }

  doc.write(decoder.decode());
  doc.close();
}
```

```html public/index.html hidden
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Browser-only rendering</title>
</head>
<body>
  <iframe id="preview" title="Rendered page"></iframe>
</body>
</html>
```

```css src/styles.css hidden
iframe {
  width: 100%;
  height: 160px;
  border: 0;
}
```

```json package.json hidden
{
  "dependencies": {
    "react": "19.3.0-canary-f1f7ed2a-20260904",
    "react-dom": "19.3.0-canary-f1f7ed2a-20260904",
    "react-scripts": "latest"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject"
  }
}
```

</Sandpack>

<Note>

必须从 Client Component 中调用 `use(browser())`。如果你的框架默认使用 Server Components，请在该文件中添加 [`'use client'`](/reference/rsc/use-client) 指令，或将此调用移到子 Client Component 中：

```js {1}
'use client';

import { use, useState } from 'react';
import { browser } from 'react-dom';

export default function SavedDraft() {
  use(browser('The saved draft is stored in localStorage.'));
  const [draft] = useState(() => localStorage.getItem('draft') ?? '');
  return <DraftEditor initialDraft={draft} />;
}
```

</Note>

---

### 在服务端有条件地渲染 {/*conditionally-rendering-on-the-server*/}

与其他 [`use`](/reference/react/use) 调用一样，`use(browser())` 可以放在条件语句中，或提早返回之后调用。这样，组件或自定义 Hook 就能根据某个条件（例如 prop 的值）选择不参与服务端渲染。

例如，`useTimeZone` Hook 接受一个可选的默认值。提供默认值时，React 会在初始 HTML 和浏览器中都渲染该值。如果没有默认值，组件会在服务端渲染期间暂停，并在浏览器中显示设备的本地时区。

点击 **重新加载**，即可在用户时区出现之前看到加载回退内容。

<Sandpack>

```js src/App.js
import { Suspense } from 'react';
import { useTimeZone } from './useTimeZone.js';

function TimeZone({label, defaultTimeZone}) {
  const timeZone = useTimeZone(defaultTimeZone);
  return <p>{label}: <strong>{timeZone}</strong></p>;
}

export default function App() {
  return (
    <>
      <h1>Event details</h1>
      <TimeZone
        label="Event time zone"
        defaultTimeZone="America/New_York"
      />
      <Suspense fallback={<p>Loading your time zone...</p>}>
        <TimeZone label="Your time zone" />
      </Suspense>
    </>
  );
}
```

```js src/useTimeZone.js active
import { use } from 'react';
import { browser } from 'react-dom';

export function useTimeZone(defaultTimeZone) {
  if (defaultTimeZone !== undefined) {
    return defaultTimeZone;
  }

  use(browser('No default time zone was provided.'));
  return Intl.DateTimeFormat().resolvedOptions().timeZone;
}
```

```js src/Document.js hidden
import App from './App.js';

export default function Document() {
  return (
    <html lang="en">
      <head>
        <title>Event details</title>
        <style>{`
          h1 { font-size: 24px; margin-top: 0; }
        `}</style>
      </head>
      <body>
        <App />
      </body>
    </html>
  );
}
```

```js src/index.js hidden
import { hydrateRoot } from 'react-dom/client';
import { renderToReadableStream } from 'react-dom/server';
import Document from './Document.js';
import { flushReadableStreamToFrame } from './demo-helpers.js';
import './styles.css';

async function main(frame) {
  const stream = await renderToReadableStream(<Document />);
  await flushReadableStreamToFrame(stream, frame);

  // Wait so both the fallback and hydrated content are visible.
  await new Promise(resolve => setTimeout(resolve, 1200));
  hydrateRoot(frame.contentDocument, <Document />);
}

main(document.getElementById('preview'));
```

```js src/demo-helpers.js hidden
export async function flushReadableStreamToFrame(readable, frame) {
  const doc = frame.contentWindow.document;
  const decoder = new TextDecoder();
  const reader = readable.getReader();

  while (true) {
    const {done, value} = await reader.read();
    if (done) {
      break;
    }
    doc.write(decoder.decode(value, {stream: true}));
  }

  doc.write(decoder.decode());
  doc.close();
}
```

```html public/index.html hidden
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Conditional browser rendering</title>
</head>
<body>
  <iframe id="preview" title="Rendered page"></iframe>
</body>
</html>
```

```css src/styles.css hidden
iframe {
  width: 100%;
  height: 240px;
  border: 0;
}
```

```json package.json hidden
{
  "dependencies": {
    "react": "19.3.0-canary-f1f7ed2a-20260904",
    "react-dom": "19.3.0-canary-f1f7ed2a-20260904",
    "react-scripts": "latest"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject"
  }
}
```

</Sandpack>

使用支持 Suspense 的数据获取库时，也可以应用类似模式来有条件地跳过服务端渲染：

```js {3}
function useBrowserQuery(query, options) {
  if (options.initialData === undefined) {
    use(browser('useBrowserQuery: No initial data was provided.'));
  }

  return useQuery(query, options);
}

function ProductDetails({ productId, initialData }) {
  const product = useBrowserQuery(`/api/products/${productId}`, {
    initialData,
  });

  return <h1>{product.name}</h1>;
}
```

提供 `initialData` 时，React 会在服务端将组件渲染为 HTML。没有该值时，React 会在 HTML 中保留最近的 [`<Suspense>`](/reference/react/Suspense) 边界的回退内容。在浏览器中，`useQuery` 会照常获取数据或从客户端缓存中读取数据。

---

### 在服务端报告仅浏览器渲染 {/*reporting-browser-only-rendering-on-the-server*/}

向服务端渲染器传入 `onBrowserBailout` 回调，以便报告仅浏览器渲染。React 为浏览器保留 Suspense 回退内容时，不会调用服务端渲染器的 `onError` 回调，也不会调用 [`hydrateRoot` 的 `onRecoverableError`](/reference/react-dom/client/hydrateRoot#error-logging-in-production) 回调。此示例还传入了一个 reason，可通过上报错误的 `cause` 访问：

```js
import { Suspense, use, useState } from 'react';
import { browser } from 'react-dom';
import { renderToPipeableStream } from 'react-dom/server';

function SavedDraft() {
  use(browser(() => new Error('The saved draft is stored in localStorage.')));
  const [draft] = useState(() => localStorage.getItem('draft') ?? '');
  return <DraftEditor initialDraft={draft} />;
}

function App() {
  return (
    <Suspense fallback={<p>Loading saved draft...</p>}>
      <SavedDraft />
    </Suspense>
  );
}

const { pipe } = renderToPipeableStream(<App />, {
  onShellReady() {
    pipe(response);
  },
  onBrowserBailout(error, errorInfo) {
    logBrowserBailout(error, errorInfo);
  }
});
```

`onBrowserBailout` 会接收两个参数：

1. 一个描述仅浏览器渲染的 `Error`。如果你向 `browser` 传入了 reason，可以通过该错误的 `cause` 访问。
2. 一个包含 `componentStack` 的 `errorInfo` 对象，用来显示发生仅浏览器渲染的位置。

reason 函数可以返回任意值。返回一个新的 `Error`，即可为 cause 提供独立的堆栈，同时避免在浏览器中创建该 `Error`。React 不会将 reason 序列化到 HTML 中。

如果没有 Suspense 边界提供回退内容，服务端渲染就会失败。React 会通过渲染器常规的错误回调报告失败，而不是调用 `onBrowserBailout`。

---

### 为浏览器中止等待中的服务端渲染 {/*aborting-pending-server-rendering-for-the-browser*/}

如果你直接调用服务端渲染 API，可以停止等待尚未完成的内容，并让浏览器完成渲染。中止服务端渲染时，将 `browser` 返回的值作为 reason 传入。这样 React 会让等待中的 Suspense 边界保持在回退状态，并在浏览器中渲染其内容：

```js {1,8}
import { browser } from 'react-dom';
import { renderToPipeableStream } from 'react-dom/server';

const { pipe, abort } = renderToPipeableStream(<App />, {
  onShellReady() {
    pipe(response);
    setTimeout(() => {
      abort(browser('The server render timed out.'));
    }, 10000);
  }
});
```

`browser` 中止 reason 不会触发服务端渲染器的 `onError` 回调，也不会触发 `hydrateRoot` 的 `onRecoverableError` 回调。相反，服务端渲染器会通过 `onBrowserBailout` 报告每一个已恢复的 Suspense 边界。

对于接受 [`AbortSignal`](https://developer.mozilla.org/en-US/docs/Web/API/AbortSignal) 的服务端渲染 API，将 `browser()` 作为 reason 传给 [`AbortController.abort`](https://developer.mozilla.org/en-US/docs/Web/API/AbortController/abort)。
