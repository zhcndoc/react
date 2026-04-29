---
title: <Suspense>
---

<Intro>

`<Suspense>` 允许你在其子组件加载完成之前显示一个回退界面。


```js
<Suspense fallback={<Loading />}>
  <SomeComponent />
</Suspense>
```

</Intro>

<InlineToc />

---

## 参考 {/*reference*/}

### `<Suspense>` {/*suspense*/}

#### 属性 {/*props*/}
* `children`：你打算渲染的实际 UI。如果 `children` 在渲染时挂起，Suspense 边界将切换为渲染 `fallback`。
* `fallback`：如果实际 UI 还未完成加载，用来替代渲染的备用 UI。任何有效的 React 节点都可以接受，不过在实践中，回退界面通常是一个轻量级占位视图，例如加载中的转圈器或骨架屏。当 `children` 挂起时，Suspense 会自动切换到 `fallback`；当数据准备好后，再切换回 `children`。如果 `fallback` 在渲染时挂起，它会激活最近的父级 Suspense 边界。

#### 注意事项 {/*caveats*/}

- 对于那些在第一次挂载之前就挂起的渲染，React 不会保留任何状态。当组件加载完成后，React 会从头重新尝试渲染挂起的树。
- 如果 Suspense 曾经显示过该树的内容，但随后它再次挂起，则会再次显示 `fallback`，除非导致它挂起的更新是由 [`startTransition`](/reference/react/startTransition) 或 [`useDeferredValue`](/reference/react/useDeferredValue) 引起的。
- 如果 React 需要隐藏已经可见的内容，因为它再次挂起了，那么它会清理内容树中的 [布局 Effect](/reference/react/useLayoutEffect)。当内容再次准备好显示时，React 会再次触发布局 Effect。这可以确保测量 DOM 布局的 Effect 不会在内容被隐藏时尝试这么做。
- React 在底层包含了一些优化，例如与 Suspense 集成的 *流式服务端渲染* 和 *选择性 Hydration*。阅读 [架构概览](https://github.com/reactwg/react-18/discussions/37) 并观看 [技术演讲](https://www.youtube.com/watch?v=pj5N-Khihgc) 了解更多。

---

## 用法 {/*usage*/}

### 在内容加载时显示回退界面 {/*displaying-a-fallback-while-content-is-loading*/}

你可以用一个 Suspense 边界包裹应用的任意部分：

```js [[1, 1, "<Loading />"], [2, 2, "<Albums />"]]
<Suspense fallback={<Loading />}>
  <Albums />
</Suspense>
```

React 会显示你的 <CodeStep step={1}>加载回退界面</CodeStep>，直到 <CodeStep step={2}>子组件</CodeStep> 所需的所有代码和数据都已加载完成。

在下面的示例中，`Albums` 组件在获取专辑列表时会*挂起*。在它准备好渲染之前，React 会把上方最近的 Suspense 边界切换为显示回退界面——你的 `Loading` 组件。然后，当数据加载完成后，React 会隐藏 `Loading` 回退界面，并带着数据渲染 `Albums` 组件。

<Sandpack>

```js src/App.js hidden
import { useState } from 'react';
import ArtistPage from './ArtistPage.js';

export default function App() {
  const [show, setShow] = useState(false);
  if (show) {
    return (
      <ArtistPage
        artist={{
          id: 'the-beatles',
          name: 'The Beatles',
        }}
      />
    );
  } else {
    return (
      <button onClick={() => setShow(true)}>
        Open The Beatles artist page
      </button>
    );
  }
}
```

```js src/ArtistPage.js active
import { Suspense } from 'react';
import Albums from './Albums.js';

export default function ArtistPage({ artist }) {
  return (
    <>
      <h1>{artist.name}</h1>
      <Suspense fallback={<Loading />}>
        <Albums artistId={artist.id} />
      </Suspense>
    </>
  );
}

function Loading() {
  return <h2>🌀 Loading...</h2>;
}
```

```js src/Albums.js
import {use} from 'react';
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
  if (url === '/the-beatles/albums') {
    return await getAlbums();
  } else {
    throw Error('Not implemented');
  }
}

async function getAlbums() {
  // 添加一个假的延迟，让等待更明显。
  await new Promise(resolve => {
    setTimeout(resolve, 3000);
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
  }, {
    id: 8,
    title: 'Sgt. Pepper\'s Lonely Hearts Club Band',
    year: 1967
  }, {
    id: 7,
    title: 'Revolver',
    year: 1966
  }, {
    id: 6,
    title: 'Rubber Soul',
    year: 1965
  }, {
    id: 5,
    title: 'Help!',
    year: 1965
  }, {
    id: 4,
    title: 'Beatles For Sale',
    year: 1964
  }, {
    id: 3,
    title: 'A Hard Day\'s Night',
    year: 1964
  }, {
    id: 2,
    title: 'With The Beatles',
    year: 1963
  }, {
    id: 1,
    title: 'Please Please Me',
    year: 1963
  }];
}
```

</Sandpack>

<Note>

**只有支持 Suspense 的数据源才会激活 Suspense 组件。** 它们包括：

- 使用像 [Relay](https://relay.dev/docs/guided-tour/rendering/loading-states/) 和 [Next.js](https://nextjs.org/docs/app/building-your-application/routing/loading-ui-and-streaming#streaming-with-suspense) 这样的支持 Suspense 的框架进行数据获取
- 使用 [`lazy`](/reference/react/lazy) 懒加载组件代码
- 使用 [`use`](/reference/react/use) 读取缓存 Promise 的值

Suspense **不会** 检测在 Effect 或事件处理函数内部获取的数据。

上面 `Albums` 组件中加载数据的具体方式取决于你的框架。如果你使用支持 Suspense 的框架，你会在其数据获取文档中找到详细说明。

目前尚不支持在不使用有明确约定框架的情况下进行支持 Suspense 的数据获取。实现支持 Suspense 的数据源所需的条件是不稳定且没有文档记录的。用于将数据源与 Suspense 集成的官方 API 将在 React 的未来版本中发布。

</Note>

---

### 一次性一起显示内容 {/*revealing-content-together-at-once*/}

默认情况下，Suspense 内部的整个树会被视为一个整体。例如，即使这些组件中*只有一个*在挂起等待某些数据，*所有*组件也会一起被加载指示器替换：

```js {2-5}
<Suspense fallback={<Loading />}>
  <Biography />
  <Panel>
    <Albums />
  </Panel>
</Suspense>
```

然后，当它们全部准备好显示时，它们会一起同时出现。

在下面的示例中，`Biography` 和 `Albums` 都会获取一些数据。不过，因为它们被归在同一个 Suspense 边界下，这些组件总是会在同一时间一起“出现”。

<Sandpack>

```js src/App.js hidden
import { useState } from 'react';
import ArtistPage from './ArtistPage.js';

export default function App() {
  const [show, setShow] = useState(false);
  if (show) {
    return (
      <ArtistPage
        artist={{
          id: 'the-beatles',
          name: 'The Beatles',
        }}
      />
    );
  } else {
    return (
      <button onClick={() => setShow(true)}>
        Open The Beatles artist page
      </button>
    );
  }
}
```

```js src/ArtistPage.js active
import { Suspense } from 'react';
import Albums from './Albums.js';
import Biography from './Biography.js';
import Panel from './Panel.js';

export default function ArtistPage({ artist }) {
  return (
    <>
      <h1>{artist.name}</h1>
      <Suspense fallback={<Loading />}>
        <Biography artistId={artist.id} />
        <Panel>
          <Albums artistId={artist.id} />
        </Panel>
      </Suspense>
    </>
  );
}

function Loading() {
  return <h2>🌀 Loading...</h2>;
}
```

```js src/Panel.js
export default function Panel({ children }) {
  return (
    <section className="panel">
      {children}
    </section>
  );
}
```

```js src/Biography.js
import {use} from 'react';
import { fetchData } from './data.js';

export default function Biography({ artistId }) {
  const bio = use(fetchData(`/${artistId}/bio`));
  return (
    <section>
      <p className="bio">{bio}</p>
    </section>
  );
}
```

```js src/Albums.js
import {use} from 'react';
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
  if (url === '/the-beatles/albums') {
    return await getAlbums();
  } else if (url === '/the-beatles/bio') {
    return await getBio();
  } else {
    throw Error('Not implemented');
  }
}

async function getBio() {
  // 添加一个假的延迟，让等待更明显。
  await new Promise(resolve => {
    setTimeout(resolve, 1500);
  });

  return `The Beatles 是一支英国摇滚乐队，
    成立于 1960 年的利物浦，由
    John Lennon、Paul McCartney、George Harrison
    和 Ringo Starr 组成。`;
}

async function getAlbums() {
  // 添加一个假的延迟，让等待更明显。
  await new Promise(resolve => {
    setTimeout(resolve, 3000);
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
  }, {
    id: 8,
    title: 'Sgt. Pepper\'s Lonely Hearts Club Band',
    year: 1967
  }, {
    id: 7,
    title: 'Revolver',
    year: 1966
  }, {
    id: 6,
    title: 'Rubber Soul',
    year: 1965
  }, {
    id: 5,
    title: 'Help!',
    year: 1965
  }, {
    id: 4,
    title: 'Beatles For Sale',
    year: 1964
  }, {
    id: 3,
    title: 'A Hard Day\'s Night',
    year: 1964
  }, {
    id: 2,
    title: 'With The Beatles',
    year: 1963
  }, {
    id: 1,
    title: 'Please Please Me',
    year: 1963
  }];
}
```

```css
.bio { font-style: italic; }

.panel {
  border: 1px solid #aaa;
  border-radius: 6px;
  margin-top: 20px;
  padding: 10px;
}
```

</Sandpack>

加载数据的组件不必是 Suspense 边界的直接子组件。例如，你可以把 `Biography` 和 `Albums` 移到一个新的 `Details` 组件中。这不会改变行为。`Biography` 和 `Albums` 共享同一个最近的父级 Suspense 边界，所以它们的显示是协调一致的。

```js {2,8-11}
<Suspense fallback={<Loading />}>
  <Details artistId={artist.id} />
</Suspense>

function Details({ artistId }) {
  return (
    <>
      <Biography artistId={artistId} />
      <Panel>
        <Albums artistId={artistId} />
      </Panel>
    </>
  );
}
```

---

### 随着内容加载逐步显示嵌套内容 {/*revealing-nested-content-as-it-loads*/}

当组件挂起时，最近的父级 Suspense 组件会显示回退界面。这使你能够嵌套多个 Suspense 组件来创建加载顺序。每个 Suspense 边界的回退界面都会在下一层内容可用时填充出来。例如，你可以给专辑列表提供自己的回退界面：

```js {3,7}
<Suspense fallback={<BigSpinner />}>
  <Biography />
  <Suspense fallback={<AlbumsGlimmer />}>
    <Panel>
      <Albums />
    </Panel>
  </Suspense>
</Suspense>
```

通过这个改动，显示 `Biography` 时就不需要“等待” `Albums` 加载完成。

这个顺序会是：

1. 如果 `Biography` 还没加载完成，则用 `BigSpinner` 替换整个内容区域显示。
2. 一旦 `Biography` 加载完成，`BigSpinner` 会被内容替换。
3. 如果 `Albums` 还没加载完成，则 `AlbumsGlimmer` 会替换 `Albums` 及其父级 `Panel` 显示。
4. 最后，一旦 `Albums` 加载完成，它会替换 `AlbumsGlimmer`。

<Sandpack>

```js src/App.js hidden
import { useState } from 'react';
import ArtistPage from './ArtistPage.js';

export default function App() {
  const [show, setShow] = useState(false);
  if (show) {
    return (
      <ArtistPage
        artist={{
          id: 'the-beatles',
          name: 'The Beatles',
        }}
      />
    );
  } else {
    return (
      <button onClick={() => setShow(true)}>
        Open The Beatles artist page
      </button>
    );
  }
}
```

```js src/ArtistPage.js active
import { Suspense } from 'react';
import Albums from './Albums.js';
import Biography from './Biography.js';
import Panel from './Panel.js';

export default function ArtistPage({ artist }) {
  return (
    <>
      <h1>{artist.name}</h1>
      <Suspense fallback={<BigSpinner />}>
        <Biography artistId={artist.id} />
        <Suspense fallback={<AlbumsGlimmer />}>
          <Panel>
            <Albums artistId={artist.id} />
          </Panel>
        </Suspense>
      </Suspense>
    </>
  );
}

function BigSpinner() {
  return <h2>🌀 Loading...</h2>;
}

function AlbumsGlimmer() {
  return (
    <div className="glimmer-panel">
      <div className="glimmer-line" />
      <div className="glimmer-line" />
      <div className="glimmer-line" />
    </div>
  );
}
```

```js src/Panel.js
export default function Panel({ children }) {
  return (
    <section className="panel">
      {children}
    </section>
  );
}
```

```js src/Biography.js
import {use} from 'react';
import { fetchData } from './data.js';

export default function Biography({ artistId }) {
  const bio = use(fetchData(`/${artistId}/bio`));
  return (
    <section>
      <p className="bio">{bio}</p>
    </section>
  );
}
```

```js src/Albums.js
import {use} from 'react';
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
  if (url === '/the-beatles/albums') {
    return await getAlbums();
  } else if (url === '/the-beatles/bio') {
    return await getBio();
  } else {
    throw Error('Not implemented');
  }
}

async function getBio() {
  // 添加一个假的延迟，让等待更明显。
  await new Promise(resolve => {
    setTimeout(resolve, 500);
  });

  return `The Beatles 是一支英国摇滚乐队，
    成立于 1960 年的利物浦，由
    John Lennon、Paul McCartney、George Harrison
    和 Ringo Starr 组成。`;
}

async function getAlbums() {
  // 添加一个假的延迟，让等待更明显。
  await new Promise(resolve => {
    setTimeout(resolve, 3000);
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
  }, {
    id: 8,
    title: 'Sgt. Pepper\'s Lonely Hearts Club Band',
    year: 1967
  }, {
    id: 7,
    title: 'Revolver',
    year: 1966
  }, {
    id: 6,
    title: 'Rubber Soul',
    year: 1965
  }, {
    id: 5,
    title: 'Help!',
    year: 1965
  }, {
    id: 4,
    title: 'Beatles For Sale',
    year: 1964
  }, {
    id: 3,
    title: 'A Hard Day\'s Night',
    year: 1964
  }, {
    id: 2,
    title: 'With The Beatles',
    year: 1963
  }, {
    id: 1,
    title: 'Please Please Me',
    year: 1963
  }];
}
```

```css
.bio { font-style: italic; }

.panel {
  border: 1px solid #aaa;
  border-radius: 6px;
  margin-top: 20px;
  padding: 10px;
}

.glimmer-panel {
  border: 1px dashed #aaa;
  background: linear-gradient(90deg, rgba(221,221,221,1) 0%, rgba(255,255,255,1) 100%);
  border-radius: 6px;
  margin-top: 20px;
  padding: 10px;
}

.glimmer-line {
  display: block;
  width: 60%;
  height: 20px;
  margin: 10px;
  border-radius: 4px;
  background: #f0f0f0;
}
```

</Sandpack>

Suspense 边界让你可以协调 UI 中哪些部分应该总是同时“出现”，以及哪些部分应该在一系列加载状态中逐步显现。你可以在树中的任何位置添加、移动或删除 Suspense 边界，而不会影响应用其余部分的行为。

不要为每个组件都包一层 Suspense 边界。Suspense 边界的粒度不应比你希望用户感受到的加载顺序更细。如果你和设计师一起工作，问问他们加载状态应该放在哪里——很可能他们已经在设计线框图里包含了这些内容。

---

### 在新内容加载时显示旧内容 {/*showing-stale-content-while-fresh-content-is-loading*/}

在这个示例中，`SearchResults` 组件在获取搜索结果时会挂起。输入 `"a"`，等待结果出现，然后将其编辑为 `"ab"`。`"a"` 的结果会被加载回退界面替换。

<Sandpack>

```js src/App.js
import { Suspense, useState } from 'react';
import SearchResults from './SearchResults.js';

export default function App() {
  const [query, setQuery] = useState('');
  return (
    <>
      <label>
        搜索专辑：
        <input value={query} onChange={e => setQuery(e.target.value)} />
      </label>
      <Suspense fallback={<h2>Loading...</h2>}>
        <SearchResults query={query} />
      </Suspense>
    </>
  );
}
```

```js src/SearchResults.js
import {use} from 'react';
import { fetchData } from './data.js';

export default function SearchResults({ query }) {
  if (query === '') {
    return null;
  }
  const albums = use(fetchData(`/search?q=${query}`));
  if (albums.length === 0) {
    return <p>没有匹配项 <i>"{query}"</i></p>;
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
  if (url.startsWith('/search?q=')) {
    return await getSearchResults(url.slice('/search?q='.length));
  } else {
    throw Error('Not implemented');
  }
}

async function getSearchResults(query) {
  // 添加一个假的延迟，让等待更明显。
  await new Promise(resolve => {
    setTimeout(resolve, 500);
  });

  const allAlbums = [{
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
  }, {
    id: 8,
    title: 'Sgt. Pepper\'s Lonely Hearts Club Band',
    year: 1967
  }, {
    id: 7,
    title: 'Revolver',
    year: 1966
  }, {
    id: 6,
    title: 'Rubber Soul',
    year: 1965
  }, {
    id: 5,
    title: 'Help!',
    year: 1965
  }, {
    id: 4,
    title: 'Beatles For Sale',
    year: 1964
  }, {
    id: 3,
    title: 'A Hard Day\'s Night',
    year: 1964
  }, {
    id: 2,
    title: 'With The Beatles',
    year: 1963
  }, {
    id: 1,
    title: 'Please Please Me',
    year: 1963
  }];

  const lowerQuery = query.trim().toLowerCase();
  return allAlbums.filter(album => {
    const lowerTitle = album.title.toLowerCase();
    return (
      lowerTitle.startsWith(lowerQuery) ||
      lowerTitle.indexOf(' ' + lowerQuery) !== -1
    )
  });
}
```

```css
input { margin: 10px; }
```

</Sandpack>

一种常见的替代 UI 模式是*延迟*更新列表，并在新结果准备好之前继续显示之前的结果。[`useDeferredValue`](/reference/react/useDeferredValue) Hook 允许你向下传递查询的延迟版本：

```js {3,11}
export default function App() {
  const [query, setQuery] = useState('');
  const deferredQuery = useDeferredValue(query);
  return (
    <>
      <label>
        搜索专辑：
        <input value={query} onChange={e => setQuery(e.target.value)} />
      </label>
      <Suspense fallback={<h2>Loading...</h2>}>
        <SearchResults query={deferredQuery} />
      </Suspense>
    </>
  );
}
```

`query` 会立即更新，所以输入框会显示新值。不过，`deferredQuery` 会在数据加载完成之前保持上一个值，因此 `SearchResults` 会在一段时间内显示旧结果。

为了让用户更容易察觉，你可以在显示旧结果列表时添加一个视觉提示：

```js {2}
<div style={{
  opacity: query !== deferredQuery ? 0.5 : 1
}}>
  <SearchResults query={deferredQuery} />
</div>
```

在下面的示例中输入 `"a"`，等待结果加载完成，然后把输入框编辑为 `"ab"`。请注意，现在你看到的不是 Suspense 回退界面，而是在新结果加载完成前显示的变暗旧结果列表：


<Sandpack>

```js src/App.js
import { Suspense, useState, useDeferredValue } from 'react';
import SearchResults from './SearchResults.js';

export default function App() {
  const [query, setQuery] = useState('');
  const deferredQuery = useDeferredValue(query);
  const isStale = query !== deferredQuery;
  return (
    <>
      <label>
        搜索专辑：
        <input value={query} onChange={e => setQuery(e.target.value)} />
      </label>
      <Suspense fallback={<h2>Loading...</h2>}>
        <div style={{ opacity: isStale ? 0.5 : 1 }}>
          <SearchResults query={deferredQuery} />
        </div>
      </Suspense>
    </>
  );
}
```

```js src/SearchResults.js hidden
import {use} from 'react';
import { fetchData } from './data.js';

export default function SearchResults({ query }) {
  if (query === '') {
    return null;
  }
  const albums = use(fetchData(`/search?q=${query}`));
  if (albums.length === 0) {
    return <p>没有匹配项 <i>"{query}"</i></p>;
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
  if (url.startsWith('/search?q=')) {
    return await getSearchResults(url.slice('/search?q='.length));
  } else {
    throw Error('Not implemented');
  }
}

async function getSearchResults(query) {
  // 添加一个假的延迟，让等待更明显。
  await new Promise(resolve => {
    setTimeout(resolve, 500);
  });

  const allAlbums = [{
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
  }, {
    id: 8,
    title: 'Sgt. Pepper\'s Lonely Hearts Club Band',
    year: 1967
  }, {
    id: 7,
    title: 'Revolver',
    year: 1966
  }, {
    id: 6,
    title: 'Rubber Soul',
    year: 1965
  }, {
    id: 5,
    title: 'Help!',
    year: 1965
  }, {
    id: 4,
    title: 'Beatles For Sale',
    year: 1964
  }, {
    id: 3,
    title: 'A Hard Day\'s Night',
    year: 1964
  }, {
    id: 2,
    title: 'With The Beatles',
    year: 1963
  }, {
    id: 1,
    title: 'Please Please Me',
    year: 1963
  }];

  const lowerQuery = query.trim().toLowerCase();
  return allAlbums.filter(album => {
    const lowerTitle = album.title.toLowerCase();
    return (
      lowerTitle.startsWith(lowerQuery) ||
      lowerTitle.indexOf(' ' + lowerQuery) !== -1
    )
  });
}
```

```css
input { margin: 10px; }
```

</Sandpack>

<Note>

延迟值和[过渡](#preventing-already-revealed-content-from-hiding)都可以让你避免显示 Suspense 回退界面，而改用内联指示器。过渡会把整个更新标记为非紧急，因此通常由框架和路由库用于导航。另一方面，延迟值主要适用于应用代码，在那里你想把 UI 的某一部分标记为非紧急，并让它“落后于”其余 UI。

</Note>

---

### 防止已显示内容被隐藏 {/*preventing-already-revealed-content-from-hiding*/}

当组件挂起时，最近的父级 Suspense 边界会切换为显示回退界面。如果它已经显示了一些内容，这可能会导致令人不适的用户体验。试着点击这个按钮：

<Sandpack>

```js src/App.js
import { Suspense, useState } from 'react';
import IndexPage from './IndexPage.js';
import ArtistPage from './ArtistPage.js';
import Layout from './Layout.js';

export default function App() {
  return (
    <Suspense fallback={<BigSpinner />}>
      <Router />
    </Suspense>
  );
}

function Router() {
  const [page, setPage] = useState('/');

  function navigate(url) {
    setPage(url);
  }

  let content;
  if (page === '/') {
    content = (
      <IndexPage navigate={navigate} />
    );
  } else if (page === '/the-beatles') {
    content = (
      <ArtistPage
        artist={{
          id: 'the-beatles',
          name: 'The Beatles',
        }}
      />
    );
  }
  return (
    <Layout>
      {content}
    </Layout>
  );
}

function BigSpinner() {
  return <h2>🌀 Loading...</h2>;
}
```

```js src/Layout.js
export default function Layout({ children }) {
  return (
    <div className="layout">
      <section className="header">
        Music Browser
      </section>
      <main>
        {children}
      </main>
    </div>
  );
}
```

```js src/IndexPage.js
export default function IndexPage({ navigate }) {
  return (
    <button onClick={() => navigate('/the-beatles')}>
      Open The Beatles artist page
    </button>
  );
}
```

```js src/ArtistPage.js
import { Suspense } from 'react';
import Albums from './Albums.js';
import Biography from './Biography.js';
import Panel from './Panel.js';

export default function ArtistPage({ artist }) {
  return (
    <>
      <h1>{artist.name}</h1>
      <Biography artistId={artist.id} />
      <Suspense fallback={<AlbumsGlimmer />}>
        <Panel>
          <Albums artistId={artist.id} />
        </Panel>
      </Suspense>
    </>
  );
}

function AlbumsGlimmer() {
  return (
    <div className="glimmer-panel">
      <div className="glimmer-line" />
      <div className="glimmer-line" />
      <div className="glimmer-line" />
    </div>
  );
}
```

```js src/Albums.js
import {use} from 'react';
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

```js src/Biography.js
import {use} from 'react';
import { fetchData } from './data.js';

export default function Biography({ artistId }) {
  const bio = use(fetchData(`/${artistId}/bio`));
  return (
    <section>
      <p className="bio">{bio}</p>
    </section>
  );
}
```

```js src/Panel.js
export default function Panel({ children }) {
  return (
    <section className="panel">
      {children}
    </section>
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
  if (url === '/the-beatles/albums') {
    return await getAlbums();
  } else if (url === '/the-beatles/bio') {
    return await getBio();
  } else {
    throw Error('Not implemented');
  }
}

async function getBio() {
  // 添加一个假的延迟，让等待更明显。
  await new Promise(resolve => {
    setTimeout(resolve, 500);
  });

  return `The Beatles 是一支英国摇滚乐队，
    成立于 1960 年的利物浦，由
    John Lennon、Paul McCartney、George Harrison
    和 Ringo Starr 组成。`;
}

async function getAlbums() {
  // 添加一个假的延迟，让等待更明显。
  await new Promise(resolve => {
    setTimeout(resolve, 3000);
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
  }, {
    id: 8,
    title: 'Sgt. Pepper\'s Lonely Hearts Club Band',
    year: 1967
  }, {
    id: 7,
    title: 'Revolver',
    year: 1966
  }, {
    id: 6,
    title: 'Rubber Soul',
    year: 1965
  }, {
    id: 5,
    title: 'Help!',
    year: 1965
  }, {
    id: 4,
    title: 'Beatles For Sale',
    year: 1964
  }, {
    id: 3,
    title: 'A Hard Day\'s Night',
    year: 1964
  }, {
    id: 2,
    title: 'With The Beatles',
    year: 1963
  }, {
    id: 1,
    title: 'Please Please Me',
    year: 1963
  }];
}
```

```css
main {
  min-height: 200px;
  padding: 10px;
}

.layout {
  border: 1px solid black;
}

.header {
  background: #222;
  padding: 10px;
  text-align: center;
  color: white;
}

.bio { font-style: italic; }

.panel {
  border: 1px solid #aaa;
  border-radius: 6px;
  margin-top: 20px;
  padding: 10px;
}

.glimmer-panel {
  border: 1px dashed #aaa;
  background: linear-gradient(90deg, rgba(221,221,221,1) 0%, rgba(255,255,255,1) 100%);
  border-radius: 6px;
  margin-top: 20px;
  padding: 10px;
}

.glimmer-line {
  display: block;
  width: 60%;
  height: 20px;
  margin: 10px;
  border-radius: 4px;
  background: #f0f0f0;
}
```

</Sandpack>

当你点击按钮时，`Router` 组件渲染了 `ArtistPage`，而不是 `IndexPage`。`ArtistPage` 内部的某个组件挂起了，因此最近的 Suspense 边界开始显示回退界面。最近的 Suspense 边界靠近根节点，所以整个网站布局都被 `BigSpinner` 替换了。

为避免这种情况，你可以使用 [`startTransition`](/reference/react/startTransition) 将导航状态更新标记为*过渡*：

```js {5,7}
function Router() {
  const [page, setPage] = useState('/');

  function navigate(url) {
    startTransition(() => {
      setPage(url);
    });
  }
  // ...
```

这会告诉 React，这个状态转换不是紧急的，最好继续显示上一页，而不要隐藏任何已经显示出来的内容。现在点击按钮会“等待” `Biography` 加载完成：

<Sandpack>

```js src/App.js
import { Suspense, startTransition, useState } from 'react';
import IndexPage from './IndexPage.js';
import ArtistPage from './ArtistPage.js';
import Layout from './Layout.js';

export default function App() {
  return (
    <Suspense fallback={<BigSpinner />}>
      <Router />
    </Suspense>
  );
}

function Router() {
  const [page, setPage] = useState('/');

  function navigate(url) {
    startTransition(() => {
      setPage(url);
    });
  }

  let content;
  if (page === '/') {
    content = (
      <IndexPage navigate={navigate} />
    );
  } else if (page === '/the-beatles') {
    content = (
      <ArtistPage
        artist={{
          id: 'the-beatles',
          name: 'The Beatles',
        }}
      />
    );
  }
  return (
    <Layout>
      {content}
    </Layout>
  );
}

function BigSpinner() {
  return <h2>🌀 Loading...</h2>;
}
```

```js src/Layout.js
export default function Layout({ children }) {
  return (
    <div className="layout">
      <section className="header">
        Music Browser
      </section>
      <main>
        {children}
      </main>
    </div>
  );
}
```

```js src/IndexPage.js
export default function IndexPage({ navigate }) {
  return (
    <button onClick={() => navigate('/the-beatles')}>
      Open The Beatles artist page
    </button>
  );
}
```

```js src/ArtistPage.js
import { Suspense } from 'react';
import Albums from './Albums.js';
import Biography from './Biography.js';
import Panel from './Panel.js';

export default function ArtistPage({ artist }) {
  return (
    <>
      <h1>{artist.name}</h1>
      <Biography artistId={artist.id} />
      <Suspense fallback={<AlbumsGlimmer />}>
        <Panel>
          <Albums artistId={artist.id} />
        </Panel>
      </Suspense>
    </>
  );
}

function AlbumsGlimmer() {
  return (
    <div className="glimmer-panel">
      <div className="glimmer-line" />
      <div className="glimmer-line" />
      <div className="glimmer-line" />
    </div>
  );
}
```

```js src/Albums.js
import {use} from 'react';
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

```js src/Biography.js
import {use} from 'react';
import { fetchData } from './data.js';

export default function Biography({ artistId }) {
  const bio = use(fetchData(`/${artistId}/bio`));
  return (
    <section>
      <p className="bio">{bio}</p>
    </section>
  );
}
```

```js src/Panel.js
export default function Panel({ children }) {
  return (
    <section className="panel">
      {children}
    </section>
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
  if (url === '/the-beatles/albums') {
    return await getAlbums();
  } else if (url === '/the-beatles/bio') {
    return await getBio();
  } else {
    throw Error('Not implemented');
  }
}

async function getBio() {
  // 添加一个假的延迟，让等待更明显。
  await new Promise(resolve => {
    setTimeout(resolve, 500);
  });

  return `The Beatles 是一支英国摇滚乐队，
    成立于 1960 年的利物浦，由
    John Lennon、Paul McCartney、George Harrison
    和 Ringo Starr 组成。`;
}

async function getAlbums() {
  // 添加一个假的延迟，让等待更明显。
  await new Promise(resolve => {
    setTimeout(resolve, 3000);
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
  }, {
    id: 8,
    title: 'Sgt. Pepper\'s Lonely Hearts Club Band',
    year: 1967
  }, {
    id: 7,
    title: 'Revolver',
    year: 1966
  }, {
    id: 6,
    title: 'Rubber Soul',
    year: 1965
  }, {
    id: 5,
    title: 'Help!',
    year: 1965
  }, {
    id: 4,
    title: 'Beatles For Sale',
    year: 1964
  }, {
    id: 3,
    title: 'A Hard Day\'s Night',
    year: 1964
  }, {
    id: 2,
    title: 'With The Beatles',
    year: 1963
  }, {
    id: 1,
    title: 'Please Please Me',
    year: 1963
  }];
}
```

```css
main {
  min-height: 200px;
  padding: 10px;
}

.layout {
  border: 1px solid black;
}

.header {
  background: #222;
  padding: 10px;
  text-align: center;
  color: white;
}

.bio { font-style: italic; }

.panel {
  border: 1px solid #aaa;
  border-radius: 6px;
  margin-top: 20px;
  padding: 10px;
}

.glimmer-panel {
  border: 1px dashed #aaa;
  background: linear-gradient(90deg, rgba(221,221,221,1) 0%, rgba(255,255,255,1) 100%);
  border-radius: 6px;
  margin-top: 20px;
  padding: 10px;
}

.glimmer-line {
  display: block;
  width: 60%;
  height: 20px;
  margin: 10px;
  border-radius: 4px;
  background: #f0f0f0;
}
```

</Sandpack>

过渡不会等待*所有*内容都加载完成。它只会等待足够长的时间，以避免隐藏已经显示出来的内容。例如，网站的 `Layout` 已经显示出来了，因此把它隐藏在加载转圈器后面是不合适的。不过，包裹 `Albums` 的嵌套 `Suspense` 边界是新的，所以过渡不会等待它。

<Note>

支持 Suspense 的路由器通常会默认把导航更新包装到过渡中。

</Note>

---

### 指示正在发生过渡 {/*indicating-that-a-transition-is-happening*/}

在上面的示例中，一旦你点击按钮，就没有视觉提示表明导航正在进行。要添加指示器，你可以用 [`useTransition`](/reference/react/useTransition) 替换 [`startTransition`](/reference/react/startTransition)，它会提供一个布尔值 `isPending`。在下面的示例中，它用于在过渡进行时更改网站头部的样式：

<Sandpack>

```js src/App.js
import { Suspense, useState, useTransition } from 'react';
import IndexPage from './IndexPage.js';
import ArtistPage from './ArtistPage.js';
import Layout from './Layout.js';

export default function App() {
  return (
    <Suspense fallback={<BigSpinner />}>
      <Router />
    </Suspense>
  );
}

function Router() {
  const [page, setPage] = useState('/');
  const [isPending, startTransition] = useTransition();

  function navigate(url) {
    startTransition(() => {
      setPage(url);
    });
  }

  let content;
  if (page === '/') {
    content = (
      <IndexPage navigate={navigate} />
    );
  } else if (page === '/the-beatles') {
    content = (
      <ArtistPage
        artist={{
          id: 'the-beatles',
          name: 'The Beatles',
        }}
      />
    );
  }
  return (
    <Layout isPending={isPending}>
      {content}
    </Layout>
  );
}

function BigSpinner() {
  return <h2>🌀 Loading...</h2>;
}
```

```js src/Layout.js
export default function Layout({ children, isPending }) {
  return (
    <div className="layout">
      <section className="header" style={{
        opacity: isPending ? 0.7 : 1
      }}>
        Music Browser
      </section>
      <main>
        {children}
      </main>
    </div>
  );
}
```

```js src/IndexPage.js
export default function IndexPage({ navigate }) {
  return (
    <button onClick={() => navigate('/the-beatles')}>
      Open The Beatles artist page
    </button>
  );
}
```

```js src/ArtistPage.js
import { Suspense } from 'react';
import Albums from './Albums.js';
import Biography from './Biography.js';
import Panel from './Panel.js';

export default function ArtistPage({ artist }) {
  return (
    <>
      <h1>{artist.name}</h1>
      <Biography artistId={artist.id} />
      <Suspense fallback={<AlbumsGlimmer />}>
        <Panel>
          <Albums artistId={artist.id} />
        </Panel>
      </Suspense>
    </>
  );
}

function AlbumsGlimmer() {
  return (
    <div className="glimmer-panel">
      <div className="glimmer-line" />
      <div className="glimmer-line" />
      <div className="glimmer-line" />
    </div>
  );
}
```

```js src/Albums.js
import {use} from 'react';
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

```js src/Biography.js
import {use} from 'react';
import { fetchData } from './data.js';

export default function Biography({ artistId }) {
  const bio = use(fetchData(`/${artistId}/bio`));
  return (
    <section>
      <p className="bio">{bio}</p>
    </section>
  );
}
```

```js src/Panel.js
export default function Panel({ children }) {
  return (
    <section className="panel">
      {children}
    </section>
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
  if (url === '/the-beatles/albums') {
    return await getAlbums();
  } else if (url === '/the-beatles/bio') {
    return await getBio();
  } else {
    throw Error('Not implemented');
  }
}

async function getBio() {
  // 添加一个假的延迟，让等待更明显。
  await new Promise(resolve => {
    setTimeout(resolve, 500);
  });

  return `The Beatles 是一支英国摇滚乐队，
    成立于 1960 年的利物浦，由
    John Lennon、Paul McCartney、George Harrison
    和 Ringo Starr 组成。`;
}

async function getAlbums() {
  // 添加一个假的延迟，让等待更明显。
  await new Promise(resolve => {
    setTimeout(resolve, 3000);
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
  }, {
    id: 8,
    title: 'Sgt. Pepper\'s Lonely Hearts Club Band',
    year: 1967
  }, {
    id: 7,
    title: 'Revolver',
    year: 1966
  }, {
    id: 6,
    title: 'Rubber Soul',
    year: 1965
  }, {
    id: 5,
    title: 'Help!',
    year: 1965
  }, {
    id: 4,
    title: 'Beatles For Sale',
    year: 1964
  }, {
    id: 3,
    title: 'A Hard Day\'s Night',
    year: 1964
  }, {
    id: 2,
    title: 'With The Beatles',
    year: 1963
  }, {
    id: 1,
    title: 'Please Please Me',
    year: 1963
  }];
}
```

```css
main {
  min-height: 200px;
  padding: 10px;
}

.layout {
  border: 1px solid black;
}

.header {
  background: #222;
  padding: 10px;
  text-align: center;
  color: white;
}

.bio { font-style: italic; }

.panel {
  border: 1px solid #aaa;
  border-radius: 6px;
  margin-top: 20px;
  padding: 10px;
}

.glimmer-panel {
  border: 1px dashed #aaa;
  background: linear-gradient(90deg, rgba(221,221,221,1) 0%, rgba(255,255,255,1) 100%);
  border-radius: 6px;
  margin-top: 20px;
  padding: 10px;
}

.glimmer-line {
  display: block;
  width: 60%;
  height: 20px;
  margin: 10px;
  border-radius: 4px;
  background: #f0f0f0;
}
```

</Sandpack>

---

### 在导航时重置 Suspense 边界 {/*resetting-suspense-boundaries-on-navigation*/}

在过渡期间，React 会避免隐藏已经显示出来的内容。不过，如果你导航到一个带有不同参数的路由，你可能希望告诉 React 这是*不同*的内容。你可以用 `key` 来表达这一点：

```js
<ProfilePage key={queryParams.id} />
```

假设你正在用户资料页内切换，而且某些内容挂起了。如果该更新被包裹在过渡中，它不会为已经可见的内容触发回退界面。这是预期行为。

不过，现在再想象你在两个不同的用户资料之间切换。在这种情况下，显示回退界面是合理的。例如，一个用户的时间线与另一个用户的时间线是*不同内容*。通过指定 `key`，你可以确保 React 将不同用户的资料视为不同组件，并在导航期间重置 Suspense 边界。集成了 Suspense 的路由器应该会自动这样做。

---

### 为服务器错误和仅客户端内容提供回退界面 {/*providing-a-fallback-for-server-errors-and-client-only-content*/}

如果你使用的是 [流式服务端渲染 API](/reference/react-dom/server) 之一（或依赖它们的框架），React 也会使用你的 `<Suspense>` 边界来处理服务器上的错误。如果某个组件在服务器上抛出错误，React 不会中止服务器渲染。相反，它会找到它上方最近的 `<Suspense>` 组件，并把它的回退界面（例如一个转圈器）包含进生成的服务器 HTML 中。用户一开始会看到一个转圈器。

在客户端，React 会再次尝试渲染同一个组件。如果它在客户端也出错了，React 会抛出该错误并显示最近的 [错误边界。](/reference/react/Component#static-getderivedstatefromerror) 但是，如果它在客户端没有出错，React 就不会向用户显示该错误，因为内容最终已经成功显示了。

你可以利用这一点，让某些组件不在服务器上渲染。为此，在服务器环境中抛出一个错误，然后用 `<Suspense>` 边界将它们包裹起来，用回退界面替换它们的 HTML：

```js
<Suspense fallback={<Loading />}>
  <Chat />
</Suspense>

function Chat() {
  if (typeof window === 'undefined') {
    throw Error('Chat should only render on the client.');
  }
  // ...
}
```

服务器 HTML 会包含加载指示器。到客户端后，它会被 `Chat` 组件替换。

---

## 故障排除 {/*troubleshooting*/}

### 如何防止在更新期间 UI 被回退内容替换？ {/*preventing-unwanted-fallbacks*/}

用回退内容替换可见 UI 会造成突兀的用户体验。当一次更新导致某个组件挂起，并且最近的 Suspense 边界已经在向用户显示内容时，就可能发生这种情况。

为了防止这种情况发生，[使用 `startTransition` 将更新标记为非紧急](#preventing-already-revealed-content-from-hiding)。在一次 Transition 期间，React 会等待，直到有足够的数据加载完成，从而避免出现不必要的回退内容：

```js {2-3,5}
function handleNextPageClick() {
  // 如果此更新发生挂起，不要隐藏已经显示的内容
  startTransition(() => {
    setCurrentPage(currentPage + 1);
  });
}
```

这样可以避免隐藏已有内容。不过，任何新渲染的 `Suspense` 边界仍会立即显示回退内容，以避免阻塞 UI，并让用户在内容可用时立即看到它。

**React 只会在非紧急更新期间防止不必要的回退内容**。如果一次渲染是由紧急更新导致的，它不会延迟渲染。你必须通过诸如 [`startTransition`](/reference/react/startTransition) 或 [`useDeferredValue`](/reference/react/useDeferredValue) 这样的 API 明确启用这一行为。

如果你的路由器与 Suspense 集成，它应该自动将其更新包装进 [`startTransition`](/reference/react/startTransition) 中。
