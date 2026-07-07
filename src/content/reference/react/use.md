---
title: use
---

<Intro>

`use` 是一个 React API，可让你读取 [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) 或 [上下文](/learn/passing-data-deeply-with-context) 的值。

```js
const value = use(resource);
```

</Intro>

<InlineToc />

---

## 参考 {/*reference*/}

### `use(context)` {/*use-context*/}

使用 [上下文](/learn/passing-data-deeply-with-context) 调用 `use` 来读取其值。与 [`useContext`](/reference/react/useContext) 不同，`use` 可以在循环和 `if` 之类的条件语句中调用。

```js
import { use } from 'react';

function Button() {
  const theme = use(ThemeContext);
  // ...
```

[查看更多示例。](#usage-context)

#### 参数 {/*context-parameters*/}

* `context`：一个使用 [`createContext`](/reference/react/createContext) 创建的 [上下文](/learn/passing-data-deeply-with-context)。

#### 返回值 {/*context-returns*/}

传入的上下文的上下文值，由调用组件上方最近的上下文提供者决定。如果没有提供者，返回的值将是传递给 [`createContext`](/reference/react/createContext) 的 `defaultValue`。

#### 注意事项 {/*context-caveats*/}

* `use` 必须在组件或 Hook 内部调用。
* 不支持在 [服务器组件](/reference/rsc/server-components) 中使用 `use` 读取上下文。

---

### `use(promise)` {/*use-promise*/}

使用 [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) 调用 `use` 来读取其已解决的值。调用 `use` 的组件在 Promise 处于 pending 状态时会 *挂起*。尽管名字如此，`use` 不是一个 Hook。与 Hooks 不同，它可以在循环和 `if` 之类的条件语句中调用。

```js
import { use } from 'react';

function MessageComponent({ messagePromise }) {
  const message = use(messagePromise);
  // ...
```

如果调用 `use` 的组件被包裹在 [Suspense](/reference/react/Suspense) 边界中，那么在 Promise 处于 pending 状态时会显示回退内容。一旦 Promise 解析完成，Suspense 回退内容会被使用 `use` 返回的数据渲染出的组件所替换。如果 Promise 被拒绝，将显示最近的 [错误边界](/reference/react/Component#catching-rendering-errors-with-an-error-boundary) 的回退内容。

[查看更多示例。](#usage-promises)

#### 参数 {/*promise-parameters*/}

* `promise`：一个你想读取其已解决值的 [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)。该 Promise 必须被[缓存](#caching-promises-for-client-components)，以便在重新渲染之间复用同一个实例。

#### 返回值 {/*promise-returns*/}

Promise 解析后的值。

#### 注意事项 {/*promise-caveats*/}

* `use` 必须在组件或 Hook 内部调用。
* `use` 不能在 try-catch 块内部调用。请改为将组件包裹在 [错误边界](#displaying-an-error-with-an-error-boundary) 中，以捕获错误并显示回退内容。
* 传递给 `use` 的 Promise 必须被缓存，以便在重新渲染之间复用同一个 Promise 实例。[请参阅下方关于缓存 Promise 的说明。](#caching-promises-for-client-components)
* 当将 Promise 从服务器组件传递到客户端组件时，其解析值必须是[可序列化的](/reference/rsc/use-client#serializable-types)。

---

## 使用（上下文） {/*usage-context*/}

### 使用 `use` 读取上下文 {/*reading-context-with-use*/}

当将 [上下文](/learn/passing-data-deeply-with-context) 传递给 `use` 时，它的工作方式与 [`useContext`](/reference/react/useContext) 类似。虽然 `useContext` 必须在组件的顶层调用，但 `use` 可以在 `if` 这样的条件判断和 `for` 这样的循环中调用。

```js [[2, 4, "theme"], [1, 4, "ThemeContext"]]
import { use } from 'react';

function Button() {
  const theme = use(ThemeContext);
  // ...
```

`use` 会为你传入的 <CodeStep step={1}>上下文</CodeStep> 返回 <CodeStep step={2}>上下文值</CodeStep>。为了确定上下文值，React 会搜索组件树，并找到该特定上下文的**最近的、位于上方的上下文提供者**。

要向 `Button` 传递上下文，请将它或它的父组件包裹在相应的上下文提供者中。

```js [[1, 3, "ThemeContext"], [2, 3, "\\"dark\\""], [1, 5, "ThemeContext"]]
function MyPage() {
  return (
    <ThemeContext value="dark">
      <Form />
    </ThemeContext>
  );
}

function Form() {
  // ... 在内部渲染按钮 ...
}
```

提供者和 `Button` 之间有多少层组件都没有关系。当 `Form` 内部任意位置的 `Button` 调用 `use(ThemeContext)` 时，它都会接收到 `"dark"` 作为值。

与 [`useContext`](/reference/react/useContext) 不同，<CodeStep step={2}>`use`</CodeStep> 可以在 <CodeStep step={1}>`if`</CodeStep> 这样的条件语句和循环中调用。

```js [[1, 2, "if"], [2, 3, "use"]]
function HorizontalRule({ show }) {
  if (show) {
    const theme = use(ThemeContext);
    return <hr className={theme} />;
  }
  return false;
}
```

<CodeStep step={2}>`use`</CodeStep> 是在 <CodeStep step={1}>`if`</CodeStep> 语句内部调用的，这使你能够有条件地从上下文中读取值。

<Pitfall>

和 `useContext` 一样，`use(context)` 总是会查找调用它的组件**上方**最近的上下文提供者。它会向上搜索，**不会**考虑你调用 `use(context)` 的那个组件内部的上下文提供者。

</Pitfall>

<Sandpack>

```js
import { createContext, use } from 'react';

const ThemeContext = createContext(null);

export default function MyApp() {
  return (
    <ThemeContext value="dark">
      <Form />
    </ThemeContext>
  )
}

function Form() {
  return (
    <Panel title="Welcome">
      <Button show={true}>Sign up</Button>
      <Button show={false}>Log in</Button>
    </Panel>
  );
}

function Panel({ title, children }) {
  const theme = use(ThemeContext);
  const className = 'panel-' + theme;
  return (
    <section className={className}>
      <h1>{title}</h1>
      {children}
    </section>
  )
}

function Button({ show, children }) {
  if (show) {
    const theme = use(ThemeContext);
    const className = 'button-' + theme;
    return (
      <button className={className}>
        {children}
      </button>
    );
  }
  return false
}
```

```css
.panel-light,
.panel-dark {
  border: 1px solid black;
  border-radius: 4px;
  padding: 20px;
}
.panel-light {
  color: #222;
  background: #fff;
}

.panel-dark {
  color: #fff;
  background: rgb(23, 32, 42);
}

.button-light,
.button-dark {
  border: 1px solid #777;
  padding: 5px;
  margin-right: 10px;
  margin-top: 10px;
}

.button-dark {
  background: #222;
  color: #fff;
}

.button-light {
  background: #fff;
  color: #222;
}
```

</Sandpack>

### 从上下文中读取 Promise {/*reading-a-promise-from-context*/}

要在不进行属性逐层传递的情况下共享异步数据，可以将 Promise 设为上下文值，然后使用 `use(context)` 读取它，再用 `use(promise)` 解析它：

```js
import { use } from 'react';
import { UserContext } from './UserContext';

function Profile() {
  const userPromise = use(UserContext);
  const user = use(userPromise);
  return <h1>{user.name}</h1>;
}
```

读取该值需要两次 `use` 调用，因为上下文本身的值不会被等待。请参阅 [使用上下文之前](/learn/passing-data-deeply-with-context#before-you-use-context)，了解在使用上下文之前可考虑的替代方案。

将读取 Promise 的组件包裹在 [Suspense](/reference/react/Suspense) 边界中，这样只有该子树会在 Promise 处于 pending 状态时挂起。关于使用 `use` 读取 Promise 的更多内容，请参阅下面的 [用法（Promises）](#usage-promises)。

<Pitfall>

当这个模式与 [服务端组件](/reference/rsc/server-components) 一起使用时，重新获取 Promise 需要重新获取将该 Promise 设置到上下文中的服务端组件。避免将 Promise 放在树的较高位置的上下文中，因为那样会不必要地重新获取应用的大部分内容。

</Pitfall>

---

## 用法（Promises）{/*usage-promises*/}

### 使用 `use` 读取 Promise {/*reading-a-promise-with-use*/}

使用 `use` 读取 Promise 的解析值。组件会在 Promise 处于 pending 状态时 [suspend](/reference/react/Suspense)。

```js [[1, 4, "use(albumsPromise)"]]
import { use } from 'react';

function Albums({ albumsPromise }) {
  const albums = use(albumsPromise);
  return (
    <ul>
      {albums.map(album => (
        <li key={album.id}>
          {album.title} ({album.year})
        </li>
      ))}
    </ul>
  );
}
```

将调用 <CodeStep step={1}>`use`</CodeStep> 的组件包裹在 [Suspense](/reference/react/Suspense) 边界中，这样 React 就可以在 Promise 处于 pending 状态时显示回退内容。位于发生 suspend 的组件上方最近的 Suspense 边界会显示它的回退内容。一旦 Promise resolved，React 就会使用 `use` 读取其值，并用渲染后的组件替换回退内容。

<Recipes titleText="使用 use 读取 Promise vs 在 Effect 中获取数据" titleId="examples-promise">

#### 使用 `use` 获取数据 {/*fetching-data-with-use*/}

在这个示例中，`Albums` 使用 `use` 读取一个已缓存的 Promise。组件会在 Promise 处于 pending 状态时 suspend，React 会显示最近的 Suspense 回退内容。被拒绝的 Promises 会向最近的 [Error Boundary](/reference/react/Component#catching-rendering-errors-with-an-error-boundary) 传播。

<Sandpack>

```js src/App.js active
import { use, Suspense } from 'react';
import { ErrorBoundary } from 'react-error-boundary';
import { fetchData } from './data.js';

export default function App() {
  return (
    <ErrorBoundary fallback={<p>无法获取专辑。</p>}>
      <Suspense fallback={<Loading />}>
        <Albums />
      </Suspense>
    </ErrorBoundary>
  );
}

function Albums() {
  const albums = use(fetchData('/albums'));
  return (
    <ul>
      {albums.map(album => (
        <li key={album.id}>
          {album.title} ({album.year})
        </li>
      ))}
    </ul>
  );
}

function Loading() {
  return <h2>加载中...</h2>;
}
```

```js src/data.js hidden
// 注意：你如何进行数据获取取决于
// 你与 Suspense 一起使用的框架。
// 通常，缓存逻辑会位于框架内部。

let cache = new Map();

export function fetchData(url) {
  if (!cache.has(url)) {
    cache.set(url, getData(url));
  }
  return cache.get(url);
}

async function getData(url) {
  if (url === '/albums') {
    return await getAlbums();
  } else {
    throw Error('Not implemented');
  }
}

async function getAlbums() {
  // 添加一个假的延迟，让等待更明显。
  await new Promise(resolve => {
    setTimeout(resolve, 1000);
  });

  return [{
    id: 13,
    title: 'Let It Be',
    year: 1970
  }, {
    id: 12,
    title: 'Abbey Road',
    year: 1969
  }, {
    id: 11,
    title: 'Yellow Submarine',
    year: 1969
  }, {
    id: 10,
    title: 'The Beatles',
    year: 1968
  }];
}
```

```json package.json hidden
{
  "dependencies": {
    "react": "19.0.0",
    "react-dom": "19.0.0",
    "react-scripts": "^5.0.0",
    "react-error-boundary": "4.0.3"
  },
  "main": "/index.js"
}
```

</Sandpack>

<Solution />

#### 使用 `useEffect` 获取数据 {/*fetching-data-with-useeffect*/}

在 `use` 之前，常见的做法是在 Effect 中获取数据，并在数据到达时更新 state。与 `use` 相比，这种方法需要手动管理加载状态和错误状态。关于为什么不鼓励在 Effect 中获取数据的更多细节，请参见 [你可能不需要 Effect](/learn/you-might-not-need-an-effect#fetching-data)。

<Sandpack>

```js src/App.js active
import { useState, useEffect } from 'react';
import { fetchAlbums } from './data.js';

export default function App() {
  const [albums, setAlbums] = useState(null);
  const [isLoading, setIsLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    fetchAlbums()
      .then(data => {
        setAlbums(data);
        setIsLoading(false);
      })
      .catch(err => {
        setError(err);
        setIsLoading(false);
      });
  }, []);

  if (isLoading) {
    return <h2>加载中...</h2>;
  }

  if (error) {
    return <p>错误：{error.message}</p>;
  }

  return (
    <ul>
      {albums.map(album => (
        <li key={album.id}>
          {album.title} ({album.year})
        </li>
      ))}
    </ul>
  );
}
```

```js src/data.js hidden
export async function fetchAlbums() {
  // 添加一个假的延迟，让等待更明显。
  await new Promise(resolve => {
    setTimeout(resolve, 1000);
  });

  return [{
    id: 13,
    title: 'Let It Be',
    year: 1970
  }, {
    id: 12,
    title: 'Abbey Road',
    year: 1969
  }, {
    id: 11,
    title: 'Yellow Submarine',
    year: 1969
  }, {
    id: 10,
    title: 'The Beatles',
    year: 1968
  }];
}
```

</Sandpack>

<Solution />

</Recipes>

<Pitfall>

##### 传递给 `use` 的 Promises 必须被缓存 {/*promises-must-cached*/}

在渲染期间创建的 Promises 会在每次渲染时重新创建，这会导致 React 反复显示 Suspense 回退内容，并阻止内容出现。

```js
function Albums() {
  // 🔴 `fetch` 会在每次渲染时创建一个新的 Promise。
  const albums = use(fetch('/albums'));
  // ...
}
```

相反，应传入来自缓存、支持 Suspense 的框架或 Server Component 的 Promise：

```js
// ✅ fetchData 从缓存中读取 Promise。
const albums = use(fetchData('/albums'));
```

</Pitfall>

<DeepDive>

#### 为什么 Promises 会在每次渲染时重新创建？ {/*why-promises-recreated*/}

[React 不会为在挂载前就已经 suspend 的渲染保留 state](/reference/react/Suspense#caveats)。每次 suspend 之后，React 都会从头重新尝试渲染，因此在渲染期间创建的任何 Promise 都会被重新创建。

在渲染期间，Promise 可能会被无意中重新创建的常见方式包括：

```js
function Albums() {
  // 🔴 `fetch` 会在每次渲染时创建一个新的 Promise。
  const albums = use(fetch('/albums'));

  // 🔴 未缓存的 `async` 函数调用会在每次渲染时创建一个新的 Promise。
  const albums = use((async () => {
    const res = await fetch('/albums');
    return res.json();
  })());

  // 🔴 即使 `fetchData` 已经缓存，添加 `.then` 也会在每次渲染时返回一个新的 Promise。
  const albums = use(fetchData('/albums').then(res => res.json()));
  // ...
}
```

理想情况下，Promises 应该在渲染之前创建，例如在事件处理函数、路由加载器或 Server Component 中创建，并传递给调用 `use` 的组件。在渲染中惰性地获取数据会延迟网络请求，并可能造成瀑布式请求。

```js
// ✅ fetchData 从缓存中读取 Promise。
const albums = use(fetchData('/albums'));
```

</DeepDive>

---

### 为客户端组件缓存 Promise {/*caching-promises-for-client-components*/}

传递给客户端组件中 `use` 的 Promise 必须被缓存，这样在重新渲染之间才能复用同一个 Promise 实例。如果在渲染过程中直接创建新的 Promise，React 会在每次重新渲染时都显示 Suspense 回退内容。

```js
// ✅ 缓存 Promise，这样在不同的渲染之间会复用同一个 Promise
let cache = new Map();

export function fetchData(url) {
  if (!cache.has(url)) {
    cache.set(url, getData(url));
  }
  return cache.get(url);
}
```

`fetchData` 函数每次使用相同的 URL 调用时，都会返回同一个 Promise。当 `use` 在重新渲染时接收到相同的 Promise，它会同步读取已经解析的值，而不会再次挂起。

<Note>

Promise 的缓存方式取决于你与 Suspense 一起使用的框架。框架通常会提供内置的缓存机制。如果你不使用框架，可以像上面那样使用一个简单的模块级缓存，或者使用一个 [支持 Suspense 的数据源](/reference/react/Suspense#displaying-a-fallback-while-content-is-loading)。

</Note>

在下面的示例中，点击“Re-render”会更新 `App` 中的状态并触发重新渲染。由于 `fetchData` 返回的是相同的已缓存 Promise，`Albums` 会同步读取该值，而不是再次显示 Suspense 回退内容。

<Sandpack>

```js src/App.js active
import { use, Suspense, useState } from 'react';
import { fetchData } from './data.js';

export default function App() {
  const [count, setCount] = useState(0);
  return (
    <>
      <button onClick={() => setCount(count + 1)}>
        Re-render
      </button>
      <p>Render count: {count}</p>
      <Suspense fallback={<p>Loading...</p>}>
        <Albums />
      </Suspense>
    </>
  );
}

function Albums() {
  const albums = use(fetchData('/albums'));
  return (
    <ul>
      {albums.map(album => (
        <li key={album.id}>
          {album.title} ({album.year})
        </li>
      ))}
    </ul>
  );
}
```

```js src/data.js hidden
// 注意：你如何进行数据获取取决于
// 你与 Suspense 一起使用的框架。
// 通常，缓存逻辑会在框架内部。

let cache = new Map();

export function fetchData(url) {
  if (!cache.has(url)) {
    cache.set(url, getData(url));
  }
  return cache.get(url);
}

async function getData(url) {
  if (url === '/albums') {
    return await getAlbums();
  } else {
    throw Error('未实现');
  }
}

async function getAlbums() {
  // 添加一个假的延迟，让等待更加明显。
  await new Promise(resolve => {
    setTimeout(resolve, 1000);
  });

  return [{
    id: 13,
    title: 'Let It Be',
    year: 1970
  }, {
    id: 12,
    title: 'Abbey Road',
    year: 1969
  }, {
    id: 11,
    title: 'Yellow Submarine',
    year: 1969
  }];
}
```

</Sandpack>

<DeepDive>

#### 如何实现 Promise 缓存 {/*how-to-implement-a-promise-cache*/}

一个基本的缓存会以 URL 为键存储 Promise，这样在不同的渲染之间就能复用同一个实例。为了在数据已经可用时也避免不必要的 Suspense 回退内容，你可以在 Promise 上设置 `status` 和 `value`（或 `reason`）字段。React 在调用 `use` 时会检查这些字段：如果 `status` 是 `'fulfilled'`，它会同步读取 `value` 而不会挂起；如果 `status` 是 `'rejected'`，它会抛出 `reason`；如果字段缺失或为 `'pending'`，它就会挂起。

```js
let cache = new Map();

function fetchData(url) {
  if (!cache.has(url)) {
    const promise = getData(url);
    promise.status = 'pending';
    promise.then(
      value => {
        promise.status = 'fulfilled';
        promise.value = value;
      },
      reason => {
        promise.status = 'rejected';
        promise.reason = reason;
      },
    );
    cache.set(url, promise);
  }
  return cache.get(url);
}
```

这主要适用于构建兼容 Suspense 的数据层的库作者。对于没有这些字段的 Promise，React 会自行设置 `status` 字段，但你自己设置它可以在数据已经可用时避免一次额外的渲染。

这种缓存模式是[重新获取数据](#re-fetching-data-in-client-components)（其中改变缓存键会触发新的获取）和[在悬停时预加载数据](#preloading-data-on-hover)（其中提前调用 `fetchData` 意味着当 `use` 读取它时，Promise 可能已经解析）的基础。

</DeepDive>

<Pitfall>

不要根据 Promise 是否已经完成而跳过调用 `use`。

与其他 Hook 不同，`use` 可以在条件分支和循环中调用——但它必须始终针对 Promise 本身被调用。绝不要直接读取 `promise.status` 或 `promise.value` 来绕过 `use`；始终将 Promise 传给 `use`，让 React 来处理它。


```js
// 🔴 不要通过直接读取 promise 状态来绕过 `use`
if (promise.status === 'fulfilled') {
  return promise.value;
}
const value = use(promise);
```

```js
// ✅ 将 promise 传给 `use`，让 React 跟踪这个 promise
const value = use(promise);
```

以这种方式绕过 `use` 可能会破坏 React Suspense 的优化以及 React DevTools 的 Suspense 功能。你可以有条件地使用 `use(promise)`，但不要基于 promise 本身有条件地调用 `use(promise)`。

</Pitfall>

---

### 在客户端组件中重新获取数据 {/*re-fetching-data-in-client-components*/}

要刷新同一 URL 的数据（例如通过一个“刷新”按钮），可以使缓存条目失效，并在一个 [`startTransition`](/reference/react/startTransition) 内发起新的请求。将返回的 Promise 存储到 state 中，以触发重新渲染。当新的 Promise 仍在等待时，React 会继续显示现有内容，因为这次更新是在 Transition 中进行的。

```js
function App() {
  const [albumsPromise, setAlbumsPromise] = useState(fetchData('/albums'));
  const [isPending, startTransition] = useTransition();

  function handleRefresh() {
    startTransition(() => {
      setAlbumsPromise(refetchData('/albums'));
    });
  }
  // ...
}
```

`refetchData` 会清除旧的缓存条目，并在相同的 URL 上发起新的请求。将返回的 Promise 存储到 state 中，会触发 Transition 内的重新渲染。重新渲染时，`Albums` 会接收到新的 Promise，而 `use` 会在其上挂起，同时 React 继续显示旧内容。

<Sandpack>

```js src/App.js active
import { Suspense, useState, useTransition } from 'react';
import { use } from 'react';
import { fetchData, refetchData } from './data.js';

export default function App() {
  const [albumsPromise, setAlbumsPromise] = useState(
    () => fetchData('/the-beatles/albums')
  );
  const [isPending, startTransition] = useTransition();

  function handleRefresh() {
    startTransition(() => {
      setAlbumsPromise(refetchData('/the-beatles/albums'));
    });
  }

  return (
    <>
      <button
        onClick={handleRefresh}
        disabled={isPending}
      >
        {isPending ? '正在刷新...' : '刷新'}
      </button>
      <div style={{ opacity: isPending ? 0.6 : 1 }}>
        <Suspense fallback={<Loading />}>
          <Albums albumsPromise={albumsPromise} />
        </Suspense>
      </div>
    </>
  );
}

function Albums({ albumsPromise }) {
  const albums = use(albumsPromise);
  return (
    <ul>
      {albums.map(album => (
        <li key={album.id}>
          {album.title} ({album.year})
        </li>
      ))}
    </ul>
  );
}

function Loading() {
  return <h2>加载中...</h2>;
}
```

```js src/data.js hidden
// 注意：你实际进行数据获取的方式取决于
// 你与 Suspense 一起使用的框架。
// 通常，缓存逻辑会位于框架内部。

let cache = new Map();

export function fetchData(url) {
  if (!cache.has(url)) {
    cache.set(url, getData(url));
  }
  return cache.get(url);
}

export function refetchData(url) {
  cache.delete(url);
  return fetchData(url);
}

async function getData(url) {
  if (url.startsWith('/the-beatles/albums')) {
    return await getAlbums();
  } else {
    throw Error('未实现');
  }
}

async function getAlbums() {
  // 添加一个假的延迟，让等待更明显。
  await new Promise(resolve => {
    setTimeout(resolve, 1000);
  });

  return [{
    id: 13,
    title: 'Let It Be',
    year: 1970
  }, {
    id: 12,
    title: 'Abbey Road',
    year: 1969
  }, {
    id: 11,
    title: 'Yellow Submarine',
    year: 1969
  }, {
    id: 10,
    title: 'The Beatles',
    year: 1968
  }, {
    id: 9,
    title: 'Magical Mystery Tour',
    year: 1967
  }];
}
```

```css
button { margin-bottom: 10px; }
```

</Sandpack>

<Note>

支持 Suspense 的框架通常会提供自己的缓存和失效机制。上面的自定义缓存有助于理解这种模式，但在实际开发中，应优先使用框架提供的数据获取方案。

</Note>

---

### 悬停时预加载数据 {/*preloading-data-on-hover*/}

你可以在需要数据之前，通过在悬停事件期间调用 `fetchData` 来开始加载数据。由于 `fetchData` 会缓存 Promise，因此在用户点击时，数据可能已经可用了。如果在 `use` 读取它时 Promise 已经解析，React 会立即渲染组件，而不会显示 Suspense 回退界面。

```js
<button
  onMouseEnter={() => fetchData(`/${id}/albums`)}
  onClick={() => {
    startTransition(() => {
      setArtistId(id);
    });
  }}
>
```

在这个示例中，悬停在艺术家按钮上会在后台开始获取他们的专辑。如果不先悬停，点击时会显示加载回退界面。试着在点击前先把鼠标悬停在按钮上一会儿，看看区别。

<Sandpack>

```js src/App.js active
import { Suspense, useState, useTransition } from 'react';
import Albums from './Albums.js';
import { fetchData } from './data.js';

export default function App() {
  const [artistId, setArtistId] = useState('the-beatles');
  const [isPending, startTransition] = useTransition();

  return (
    <>
      <div>
        {['the-beatles', 'led-zeppelin', 'pink-floyd'].map(id => (
          <button
            key={id}
            onMouseEnter={() => {
              fetchData(`/${id}/albums`);
            }}
            onClick={() => {
              startTransition(() => {
                setArtistId(id);
              });
            }}
          >
            {id === 'the-beatles' ? '披头士乐队' :
             id === 'led-zeppelin' ? '齐柏林飞艇' :
             '平克·弗洛伊德'}
          </button>
        ))}
      </div>
      <Suspense key={artistId} fallback={<Loading />}>
        <Albums artistId={artistId} />
      </Suspense>
    </>
  );
}

function Loading() {
  return <h2>加载中...</h2>;
}
```

```js src/Albums.js
import { use } from 'react';
import { fetchData } from './data.js';

export default function Albums({ artistId }) {
  const albums = use(fetchData(`/${artistId}/albums`));
  return (
    <ul>
      {albums.map(album => (
        <li key={album.id}>
          {album.title} ({album.year})
        </li>
      ))}
    </ul>
  );
}
```

```js src/data.js hidden
// 注意：你会如何进行数据获取，取决于
// 你与 Suspense 一起使用的框架。
// 通常，缓存逻辑会在框架内部。

let cache = new Map();

export function fetchData(url) {
  if (!cache.has(url)) {
    const promise = getData(url);
    // 设置状态字段，这样如果 Promise 在
    // `use` 被调用之前就已解析，React 就可以同步读取值
    // （例如在悬停时预加载的情况）。
    promise.status = 'pending';
    promise.then(
      value => {
        promise.status = 'fulfilled';
        promise.value = value;
      },
      reason => {
        promise.status = 'rejected';
        promise.reason = reason;
      },
    );
    cache.set(url, promise);
  }
  return cache.get(url);
}

async function getData(url) {
  if (url.startsWith('/the-beatles/albums')) {
    return await getAlbums('the-beatles');
  } else if (url.startsWith('/led-zeppelin/albums')) {
    return await getAlbums('led-zeppelin');
  } else if (url.startsWith('/pink-floyd/albums')) {
    return await getAlbums('pink-floyd');
  } else {
    throw Error('未实现');
  }
}

async function getAlbums(artistId) {
  // 添加一个假的延迟，让等待变得更明显。
  await new Promise(resolve => {
    setTimeout(resolve, 800);
  });

  if (artistId === 'the-beatles') {
    return [{
      id: 13,
      title: 'Let It Be',
      year: 1970
    }, {
      id: 12,
      title: 'Abbey Road',
      year: 1969
    }, {
      id: 11,
      title: 'Yellow Submarine',
      year: 1969
    }];
  } else if (artistId === 'led-zeppelin') {
    return [{
      id: 10,
      title: 'Coda',
      year: 1982
    }, {
      id: 9,
      title: 'In Through the Out Door',
      year: 1979
    }, {
      id: 8,
      title: 'Presence',
      year: 1976
    }];
  } else {
    return [{
      id: 7,
      title: 'The Wall',
      year: 1979
    }, {
      id: 6,
      title: 'Animals',
      year: 1977
    }, {
      id: 5,
      title: 'Wish You Were Here',
      year: 1975
    }];
  }
}
```

```css
button { margin-right: 10px; }
```

</Sandpack>

---

### 从服务器向客户端流式传输数据 {/*streaming-data-from-server-to-client*/}

可以通过将 Promise 作为属性从 Server Component 传递给 Client Component，来将数据从服务器流式传输到客户端。

```js
import { fetchMessage } from './lib.js';
import { Message } from './message.js';

export default function App() {
  const messagePromise = fetchMessage();
  return (
    <Suspense fallback={<p>等待消息中...</p>}>
      <Message messagePromise={messagePromise} />
    </Suspense>
  );
}
```

然后，Client Component 接收作为属性传入的 Promise，并将其传递给 `use` API。这样 Client Component 就可以读取最初由 Server Component 创建的 Promise 中的值。

```js
// message.js
'use client';

import { use } from 'react';

export function Message({ messagePromise }) {
  const messageContent = use(messagePromise);
  return <p>这里是消息：{messageContent}</p>;
}
```
由于 `Message` 被包裹在 [Suspense](/reference/react/Suspense) 边界中，因此在 Promise 解析完成之前会显示回退内容。当 Promise 解析完成后，`use` API 会读取其值，`Message` 组件将替换 Suspense 回退内容。

<Sandpack>

```js src/message.js active
"use client";

import { use, Suspense } from "react";

function Message({ messagePromise }) {
  const messageContent = use(messagePromise);
  return <p>这里是消息：{messageContent}</p>;
}

export function MessageContainer({ messagePromise }) {
  return (
    <Suspense fallback={<p>⌛正在下载消息...</p>}>
      <Message messagePromise={messagePromise} />
    </Suspense>
  );
}
```

```js src/App.js hidden
import { useState } from "react";
import { MessageContainer } from "./message.js";

function fetchMessage() {
  return new Promise((resolve) => setTimeout(resolve, 1000, "⚛️"));
}

export default function App() {
  const [messagePromise, setMessagePromise] = useState(null);
  const [show, setShow] = useState(false);
  function download() {
    setMessagePromise(fetchMessage());
    setShow(true);
  }

  if (show) {
    return <MessageContainer messagePromise={messagePromise} />;
  } else {
    return <button onClick={download}>下载消息</button>;
  }
}
```

```js src/index.js hidden
import React, { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import './styles.css';

// TODO: 更新此示例以使用
// 一旦 Codesandbox Server Component
// 演示环境创建完成
import App from './App';

const root = createRoot(document.getElementById('root'));
root.render(
  <StrictMode>
    <App />
  </StrictMode>
);
```

</Sandpack>

<DeepDive>

#### 我应该在 Server 还是 Client Component 中解析 Promise？ {/*resolve-promise-in-server-or-client-component*/}

可以在 Server Component 中使用 `await` 解析 Promise，或者将其作为属性传递给 Client Component，并在那里使用 `use` 进行解析。

在 Server Component 中使用 `await` 会让 Server Component 本身挂起，而 Client Component 会将已解析的值作为属性接收：

```js
// Server Component
export default async function App() {
  // 将挂起 Server Component。
  const messageContent = await fetchMessage();
  return <Message messageContent={messageContent} />;
}
```

Server Component 也可以在不等待 Promise 的情况下先启动它，然后将该 Promise 传递给 Client Component。Server Component 会立即返回，而 Client Component 会在调用 `use` 时挂起：

```js
// Server Component
export default function App() {
  // 未等待：在此处启动，在客户端解析。
  const messagePromise = fetchMessage();
  return <Message messagePromise={messagePromise} />;
}
```

```js
// Client Component
'use client';
import { use } from 'react';

export function Message({ messagePromise }) {
  // 在数据可用之前会挂起。
  const messageContent = use(messagePromise);
  return <p>{messageContent}</p>;
}
```

在可能的情况下，优先在 Server Component 中使用 `await`，因为这样可以将数据获取保留在服务器端。如果上层的 Server Component 已经等待了数据，那么请将已解析的值作为属性向下传递，而不是创建一个新的 Promise 再调用 `use`。

你也可以在不等待的情况下将 promise 作为属性传递给 Client Component，然后使用 `use(promise)` 在树的更深处挂起。这样在 Promise 仍处于挂起状态时，周围更多的 UI 就可以完成渲染。一个常见场景是弹出层和工具提示等交互式内容，这些内容只有在悬停或点击后才需要数据。Client Components 不能使用 `await`，因此它们依赖 `use` 来在 Promise 上挂起。

无论哪种情况，都要把读取 Promise 的组件包裹在 Suspense 边界中，这样 React 就可以在 Promise 处于挂起状态时显示回退内容。有关边界放置的指导，请参见[一起同时显示内容](/reference/react/Suspense#revealing-content-together-at-once)。

</DeepDive>

---

### 使用错误边界显示错误 {/*displaying-an-error-with-an-error-boundary*/}

如果传递给 `use` 的 Promise 被拒绝，错误会传播到最近的 [Error Boundary](/reference/react/Component#catching-rendering-errors-with-an-error-boundary)。将调用 `use` 的组件包裹在 Error Boundary 中，以便在 Promise 被拒绝时显示备用内容。

在下面的示例中，`fetchData` 在第一次尝试时会被拒绝，在重试时会成功。Error Boundary 会捕获该拒绝并显示带有 “Try again” 按钮的备用内容。

<Sandpack>

```js src/App.js active
import { use, Suspense, useState, startTransition } from "react";
import { ErrorBoundary } from "react-error-boundary";
import { fetchData, refetchData } from "./data.js";

export default function App() {
  const [albumsPromise, setAlbumsPromise] = useState(
    () => fetchData('/the-beatles/albums')
  );

  function handleRetry() {
    startTransition(() => {
      setAlbumsPromise(refetchData('/the-beatles/albums'));
    });
  }

  return (
    <ErrorBoundary
      resetKeys={[albumsPromise]}
      fallbackRender={() => (
        <>
          <p>⚠️ 加载专辑时出错了。</p>
          <button onClick={handleRetry}>再试一次</button>
        </>
      )}
    >
      <Suspense fallback={<p>正在加载...</p>}>
        <Albums albumsPromise={albumsPromise} />
      </Suspense>
    </ErrorBoundary>
  );
}

function Albums({ albumsPromise }) {
  const albums = use(albumsPromise);
  return (
    <ul>
      {albums.map(album => (
        <li key={album.id}>
          {album.title} ({album.year})
        </li>
      ))}
    </ul>
  );
}
```

```js src/data.js hidden
// 注意：你应该如何进行数据获取，取决于
// 你与 Suspense 一起使用的框架。
// 通常，缓存逻辑会放在框架内部。

let cache = new Map();
let retried = false;

export function fetchData(url) {
  if (!cache.has(url)) {
    cache.set(url, getData(url));
  }
  return cache.get(url);
}

export function refetchData(url) {
  cache.delete(url);
  retried = true;
  return fetchData(url);
}

async function getData(url) {
  // 添加一个假的延迟，以使加载状态可见。
  await new Promise(resolve => setTimeout(resolve, 1000));
  if (url === '/the-beatles/albums') {
    // 首次尝试失败，以演示 Error Boundary，
    // 然后在重试时成功。
    if (!retried) {
      throw new Error('示例错误：获取专辑失败');
    }
    return [{
      id: 13,
      title: 'Let It Be',
      year: 1970
    }, {
      id: 12,
      title: 'Abbey Road',
      year: 1969
    }, {
      id: 11,
      title: 'Yellow Submarine',
      year: 1969
    }, {
      id: 10,
      title: 'The Beatles',
      year: 1968
    }];
  }
  throw new Error('未实现');
}
```

```json package.json hidden
{
  "dependencies": {
    "react": "19.0.0",
    "react-dom": "19.0.0",
    "react-scripts": "^5.0.0",
    "react-error-boundary": "4.0.3"
  },
  "main": "/index.js"
}
```
</Sandpack>

---

## 故障排查 {/*troubleshooting*/}

### 我遇到一个错误：“Suspense 异常：这不是真正的错误！” {/*suspense-exception-error*/}

你在 `use` 外面包裹了一个 try-catch 代码块。`use` 会在内部抛出异常以集成 Suspense，因此不能被 try-catch 包裹。相反，请将调用 `use` 的组件包裹在 [错误边界](#displaying-an-error-with-an-error-boundary) 中来处理错误。

```jsx
function Albums({ albumsPromise }) {
  try {
    // ❌ 不要用 try-catch 包裹 `use`
    const albums = use(albumsPromise);
  } catch (e) {
    return <p>错误</p>;
  }
  // ...
```

相反，请将组件包裹在错误边界中：

```jsx
function Albums({ albumsPromise }) {
  // ✅ 不要用 try-catch 调用 `use`
  const albums = use(albumsPromise);
  // ...
```

```jsx
// ✅ 使用错误边界来处理错误
<ErrorBoundary fallback={<p>错误</p>}>
  <Albums albumsPromise={albumsPromise} />
</ErrorBoundary>
```

---

### 我收到一个警告：“一个组件因未缓存的 Promise 而被挂起” {/*uncached-promise-error*/}

传递给 `use` 的 Promise 没有被缓存，因此 React 无法在重新渲染之间复用它。

这种情况通常发生在直接在渲染中调用 `fetch` 或 `async` 函数时：

```js
function Albums() {
  // 🔴 这会在每次渲染时创建一个新的 Promise
  const albums = use(fetch('/albums'));
  // ...
}
```

要修复此问题，请缓存该 Promise，以便复用同一个实例：

```js
// ✅ 对于相同的 URL，fetchData 返回相同的 Promise
const albums = use(fetchData('/albums'));
```

有关更多细节，请参阅 [为客户端组件缓存 Promise](#caching-promises-for-client-components)。
