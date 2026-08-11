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
* `children`：你打算渲染的实际 UI。如果 `children` 在渲染过程中暂停，Suspense 边界将切换为渲染 `fallback`。
* `fallback`：如果实际 UI 尚未加载完成，则替代实际 UI 进行渲染的 UI。接受任何有效的 React 节点，但在实践中，`fallback` 通常是一个轻量级的占位视图，例如加载指示器或骨架屏。当 `children` 暂停时，Suspense 会自动切换为 `fallback`，数据准备就绪后再切换回 `children`。如果 `fallback` 在渲染过程中暂停，它将激活最近的父级 Suspense 边界。
* <ExperimentalBadge /> **可选** `defer`：一个布尔值。当为 `true` 时，即使 `children` 中没有任何内容暂停，React 也可能先显示 `fallback`，稍后再渲染或流式传输 `children`。用于渲染成本较高的内容。默认为 `false`。

#### 注意事项 {/*caveats*/}

- Suspense 无法检测在 Effect 或事件处理函数中获取数据的情况。它只会在[下面列出的情况](#what-activates-a-suspense-boundary)下激活。
- 对于在首次挂载前就暂停的渲染，React 不会保留其任何状态。组件加载完成后，React 将从头开始重新渲染暂停的树。
- 如果 Suspense 正在显示该树的内容，但随后再次暂停，除非导致该更新的操作是由 [`startTransition`](/reference/react/startTransition) 或 [`useDeferredValue`](/reference/react/useDeferredValue) 引起的，否则将再次显示 `fallback`。
- React 每 300 毫秒最多显示一次暂停的内容，该时间从上次显示开始计算。在此时间窗口内准备就绪的边界会[一起显示](/blog/2025/10/01/react-19-2#batching-suspense-boundaries-for-ssr)，而不是逐个显示。
- 如果 React 需要隐藏已经可见的内容，因为它再次暂停了，React 将清理内容树中的[布局 Effect](/reference/react/useLayoutEffect)。内容准备好再次显示时，React 将再次触发布局 Effect。这可以确保测量 DOM 布局的 Effect 不会在内容隐藏时尝试执行此操作。
- React 包含与 Suspense 集成的底层优化，例如*流式服务器渲染*和*选择性水合*。阅读[架构概览](https://github.com/reactwg/react-18/discussions/37)并观看[技术演讲](https://www.youtube.com/watch?v=pj5N-Khihgc)以了解更多信息。

---

### 什么会激活 Suspense 边界 {/*what-activates-a-suspense-boundary*/}

Suspense 边界会等待其内容准备就绪后再显示内容。以下任一情况都会使边界无法显示其内容：

- 使用 [`lazy`](/reference/react/lazy) 延迟加载组件代码。
- 使用 [`use`](/reference/react/use) 读取 Promise，包括从[服务器组件](/reference/rsc/server-components)流式传输的数据，或通过[支持 Suspense 的框架](#suspense-enabled-frameworks)加载的数据。
- 加载使用 [`<link rel="stylesheet">` 和 `precedence` 属性渲染的样式表。](/reference/react-dom/components/link#special-rendering-behavior)React 会阻塞边界，直到样式表加载完成，但有超时限制。[参见下面的示例。](#waiting-for-a-stylesheet-to-load)
- 在流式服务器渲染期间等待大型边界的 HTML 到达。发送 HTML 需要时间，因此即使其中没有任何内容暂停，包含足够内容的边界也会被激活。React 会随着 HTML 的到达显示内容。
- <CanaryBadge /> 加载字体。默认情况下，Suspense 不会等待字体，但在 [`<ViewTransition>`](/reference/react/ViewTransition) 更新期间，会等待新字体加载完成，但有超时限制，以避免文本使用后备字体闪烁。[参见下面的示例。](#waiting-for-a-font-to-load)
- <CanaryBadge /> 加载图片。默认情况下，Suspense 不会等待图片，但在 [`<ViewTransition>`](/reference/react/ViewTransition) 更新期间，React 会阻塞边界，直到图片加载完成，但有超时限制。添加 `onLoad` 处理函数可以让特定图片不受此行为影响。[参见下面的示例。](#waiting-for-an-image-to-load)
- <ExperimentalBadge /> 在 [`<Suspense defer>`](#props) 边界内执行 CPU 密集型渲染工作。

<Note>

#### 支持 Suspense 的框架 {/*suspense-enabled-frameworks*/}

*支持 Suspense 的框架*提供了一种在组件中读取数据的方式，并以此激活最近的 Suspense 边界。加载数据的具体方式取决于你使用的框架，详情请参阅其文档。在底层，支持 Suspense 的框架会维护一个 Promise 缓存，并调用 [`use`](/reference/react/use) 来暂停于某个 Promise。

不使用框架时，只要 Promise 已[缓存，以便在多次渲染之间复用同一个实例](/reference/react/use#caching-promises-for-client-components)，你也可以直接使用 `use` 读取 Promise。

</Note>

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

相比之下，在 `use` 外部获取数据的代码（例如在 Effect 内部获取数据）不会激活边界：

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
import EffectAlbums from './EffectAlbums.js';

export default function ArtistPage({ artist }) {
  return (
    <>
      <h1>{artist.name}</h1>
      <Suspense fallback={<Loading />}>
        <EffectAlbums artistId={artist.id} />
      </Suspense>
    </>
  );
}

function Loading() {
  return <h2>🌀 Loading...</h2>;
}
```

```js src/EffectAlbums.js
import { useState, useEffect } from 'react';
import { fetchData } from './data.js';

export default function EffectAlbums({ artistId }) {
  const [albums, setAlbums] = useState([]);

  useEffect(() => {
    let active = true;
    fetchData(`/${artistId}/albums`).then(result => {
      if (active) {
        setAlbums(result);
      }
    });
    return () => {
      active = false;
    };
  }, [artistId]);

  // Suspense 无法看到这次数据获取，因此它的回退界面
  // 永远不会显示。列表会保持为空，直到数据到达。
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

在流式服务器渲染期间，当边界的 HTML 仍在传输时，该边界也会被激活。使用任何流式服务器渲染 API 时，React 会先发送带有 `fallback` 的[外壳](/reference/react-dom/server/renderToPipeableStream#specifying-what-goes-into-the-shell)，然后传输每个边界的 HTML，并在内容到达后替换其 `fallback`。点击“渲染页面”即可观察页面逐步传输：

<Sandpack>

```js src/App.js hidden
```

```html public/index.html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>流式 SSR</title>
</head>
<body>
  <button id="render">Render the page</button>
  <br /><br />
  <iframe id="container" style="width: 100%; height: 180px; border: 1px solid #aaa;"></iframe>
</body>
</html>
```

```js src/index.js
import { flushReadableStreamToFrame } from './demo-helpers.js';
import { Suspense, use } from 'react';
import { renderToReadableStream } from 'react-dom/server';

let posts = null;

function Posts() {
  const text = use(posts.promise);
  return <p>{text}</p>;
}

function ProfilePage() {
  return (
    <html>
      <body>
        <h1>Alice</h1>
        <p>Photographer and traveler.</p>
        <Suspense fallback={<p>⌛ Loading posts...</p>}>
          <Posts />
        </Suspense>
      </body>
    </html>
  );
}

async function main(frame) {
  posts = Promise.withResolvers();
  const stream = await renderToReadableStream(<ProfilePage />);

  // 帖子会在外壳传输完成后解析，因此 React
  // 会传输它们的 HTML，并替换回退界面。
  setTimeout(() => {
    posts.resolve(
      'Just got back from two weeks along the coast. The drive ' +
      'was longer than expected, but every stop was worth it. ' +
      'A full write-up and more photos are coming soon.'
    );
  }, 1500);

  await flushReadableStreamToFrame(stream, frame);
}

document.getElementById('render').addEventListener('click', () => {
  main(document.getElementById('container'));
});
```

```js src/demo-helpers.js hidden
export async function flushReadableStreamToFrame(readable, frame) {
  const doc = frame.contentWindow.document;
  const decoder = new TextDecoder();
  for await (const chunk of readable) {
    doc.write(decoder.decode(chunk, { stream: true }));
  }
  doc.close();
}
```

</Sandpack>

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
        打开披头士艺术家页面
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
  return <h2>🌀 加载中...</h2>;
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
        打开披头士乐队页面
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
  return <h2>🌀 加载中……</h2>;
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
  return <h2>🌀 加载中...</h2>;
}
```

```js src/Layout.js
export default function Layout({ children }) {
  return (
    <div className="layout">
      <section className="header">
        音乐浏览器
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
      打开披头士艺术家页面
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
  return <h2>🌀 加载中...</h2>;
}
```

```js src/Layout.js
export default function Layout({ children }) {
  return (
    <div className="layout">
      <section className="header">
        音乐浏览器
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
      打开披头士艺术家页面
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

在 Transition 期间，React 会避免隐藏已经显示的内容。但是，当你导航到*不同*的内容时，例如另一位用户的个人资料，你会希望边界显示 fallback，而不是之前的内容。你可以使用 `key` 来表达这一点：

```js
<ProfilePage key={queryParams.id} />
```

使用不同的 `key` 后，React 会将这些个人资料视为不同的内容，并在导航期间重置 Suspense 边界。`key` 可以放在边界本身，也可以放在它上方的组件上。与 Suspense 集成的路由器应该会自动执行此操作。

在下面的示例中，打开个人资料页面会加载第一个个人资料。按下“Bob”会导航到另一个个人资料，而 `key` 会重置边界，因此会显示 fallback，而不是之前用户的简介。尝试移除 `key`：在下一个简介加载期间，之前的简介仍会保持可见：

<Sandpack>

```js src/App.js hidden
import { useState } from 'react';
import ProfilePage from './ProfilePage.js';

export default function App() {
  const [show, setShow] = useState(false);
  if (show) {
    return <ProfilePage />;
  }
  return (
    <button onClick={() => setShow(true)}>
      Open profile page
    </button>
  );
}
```

```js src/ProfilePage.js active
import { Suspense, useState, startTransition } from 'react';
import Bio from './Bio.js';
import { fetchBio } from './data.js';

export default function ProfilePage() {
  const [user, setUser] = useState(() => ({
    id: 'alice',
    bioPromise: fetchBio('alice'),
  }));
  function navigate(id) {
    startTransition(() => {
      setUser({ id, bioPromise: fetchBio(id) });
    });
  }
  return (
    <>
      <button onClick={() => navigate('alice')}>
        Alice
      </button>
      <button onClick={() => navigate('bob')}>
        Bob
      </button>
      <Suspense key={user.id} fallback={<p>⌛ Loading profile...</p>}>
        <Bio bioPromise={user.bioPromise} />
      </Suspense>
    </>
  );
}
```

```js src/Bio.js
import { use } from 'react';

export default function Bio({ bioPromise }) {
  const bio = use(bioPromise);
  return <p>{bio}</p>;
}
```

```js src/data.js hidden
// 注意：数据获取的方式取决于
// 你与 Suspense 搭配使用的框架。

export async function fetchBio(userId) {
  // 添加一个人为延迟，让等待过程更加明显。
  await new Promise(resolve => {
    setTimeout(resolve, 1500);
  });

  return userId === 'alice'
    ? 'Alice is a photographer and traveler.'
    : 'Bob collects vintage synthesizers.';
}
```

```css
button {
  margin-right: 8px;
}
```

</Sandpack>

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

### 等待样式表加载 {/*waiting-for-a-stylesheet-to-load*/}

使用带有 [`<link rel="stylesheet">` 和 `precedence` 属性](/reference/react-dom/components/link#special-rendering-behavior) 渲染的样式表会阻塞 Suspense 边界，直到样式表加载完成（最长等待一段时间），这样内容就不会以未应用样式的状态出现。

在下面的示例中，`Card` 组件使用 `precedence` 渲染一个样式表。按下“Show card”：React 会一直显示回退内容，直到样式表加载完成，然后显示应用了样式的卡片。

作为对比，第二个按钮在一个独立的文档中执行相同的更新，但不使用 React。不会等待样式表加载，因此卡片文本会先以回退字体显示，然后再切换：

<Sandpack>

```js
import { Suspense, useState, startTransition } from 'react';
import { freshStylesheetUrl } from './styles.js';
import VanillaCard from './VanillaCard.js';

function Card({ href }) {
  return (
    <>
      <link rel="stylesheet" href={href} precedence="default" />
      <div className="fancy-card">This card uses a font from the stylesheet.</div>
    </>
  );
}

export default function App() {
  const [href, setHref] = useState(null);
  return (
    <>
      <button
        onClick={() => {
          startTransition(() => {
            setHref(freshStylesheetUrl());
          });
        }}>
        Show card
      </button>
      {href && (
        <Suspense fallback={<p>⌛ Loading styles...</p>}>
          <Card href={href} />
        </Suspense>
      )}
      <hr />
      <VanillaCard />
    </>
  );
}
```

```js src/VanillaCard.js
import { useRef } from 'react';
import { freshStylesheetUrl } from './styles.js';

export default function VanillaCard() {
  const ref = useRef(null);
  function show() {
    const doc = ref.current.contentWindow.document;
    doc.open();
    doc.write(`
      <style>
        body { margin: 0; }
        .fancy-card {
          padding: 20px;
          border-radius: 8px;
          color: white;
          font-family: 'Caveat', sans-serif;
          font-size: 24px;
          background: linear-gradient(135deg, #087ea4, #2b3491);
        }
      </style>
      <div class="fancy-card">This card uses a font from the stylesheet.</div>
      <link rel="stylesheet" href="${freshStylesheetUrl()}">
    `);
    doc.close();
  }
  return (
    <>
      <button onClick={show}>Show card (without React)</button>
      <iframe ref={ref} title="Vanilla card" className="vanilla-frame" />
    </>
  );
}
```

```js src/styles.js hidden
// 添加唯一参数，使样式表不会被缓存，
// 并确保每次运行都会显示加载状态。
export function freshStylesheetUrl() {
  return (
    'https://fonts.googleapis.com/css2?family=Caveat&display=swap' +
    '&t=' +
    Date.now()
  );
}
```

```css
#root {
  min-height: 300px;
}
button {
  margin-right: 8px;
}
hr {
  margin: 16px 0;
}
.fancy-card {
  margin-top: 1em;
  padding: 20px;
  border-radius: 8px;
  color: white;
  font-family: 'Caveat', sans-serif;
  font-size: 24px;
  background: linear-gradient(135deg, #087ea4, #2b3491);
}
.vanilla-frame {
  display: block;
  margin-top: 1em;
  border: none;
  width: 100%;
  height: 90px;
}
```

</Sandpack>

---

### <CanaryBadge /> 从 Suspense 内容进行动画 {/*animating-from-suspense-content*/}

Suspense 可以与 [`<ViewTransition>`](/reference/react/ViewTransition) 组合使用，以实现从回退内容切换到实际内容时的动画效果。将边界包裹在 `<ViewTransition>` 中，React 会将这次切换视为一次更新，默认在回退内容和实际内容之间进行交叉淡化：

<Sandpack>

```js src/Video.js hidden
function Thumbnail({video, children}) {
  return (
    <div
      aria-hidden="true"
      tabIndex={-1}
      className={`thumbnail ${video.image}`}
    />
  );
}

export function Video({video}) {
  return (
    <div className="video">
      <div className="link">
        <Thumbnail video={video}></Thumbnail>
        <div className="info">
          <div className="video-title">{video.title}</div>
          <div className="video-description">{video.description}</div>
        </div>
      </div>
    </div>
  );
}

export function VideoPlaceholder() {
  const video = {image: 'loading'};
  return (
    <div className="video">
      <div className="link">
        <Thumbnail video={video}></Thumbnail>
        <div className="info">
          <div className="video-title loading" />
          <div className="video-description loading" />
        </div>
      </div>
    </div>
  );
}
```

```js
import {ViewTransition, useState, startTransition, Suspense} from 'react';
import {Video, VideoPlaceholder} from './Video';
import {useLazyVideoData} from './data';

function LazyVideo() {
  const video = useLazyVideoData();
  return <Video video={video} />;
}

export default function Component() {
  const [showItem, setShowItem] = useState(false);
  return (
    <>
      <button
        onClick={() => {
          startTransition(() => {
            setShowItem((prev) => !prev);
          });
        }}>
        {showItem ? '➖' : '➕'}
      </button>
      {showItem ? (
        <ViewTransition>
          <Suspense fallback={<VideoPlaceholder />}>
            <LazyVideo />
          </Suspense>
        </ViewTransition>
      ) : null}
    </>
  );
}
```

```js src/data.js hidden
import {use} from 'react';

let cache = null;

function fetchVideo() {
  if (!cache) {
    cache = new Promise((resolve) => {
      setTimeout(() => {
        resolve({
          id: '1',
          title: 'First video',
          description: 'Video description',
          image: 'blue',
        });
      }, 1000);
    });
  }
  return cache;
}

export function useLazyVideoData() {
  return use(fetchVideo());
}
```

```css
#root {
  display: flex;
  flex-direction: column;
  align-items: center;
  min-height: 200px;
}
button {
  border: none;
  border-radius: 50%;
  width: 50px;
  height: 50px;
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #f0f8ff;
  color: white;
  font-size: 20px;
  cursor: pointer;
  transition: background-color 0.3s, border 0.3s;
}
button:hover {
  border: 2px solid #ccc;
  background-color: #e0e8ff;
}
.thumbnail {
  position: relative;
  aspect-ratio: 16 / 9;
  display: flex;
  overflow: hidden;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  border-radius: 0.5rem;
  outline-offset: 2px;
  width: 8rem;
  vertical-align: middle;
  background-color: #ffffff;
  background-size: cover;
  user-select: none;
}
.thumbnail.blue {
  background-image: conic-gradient(at top right, #c76a15, #087ea4, #2b3491);
}
.loading {
  background-image: linear-gradient(
    90deg,
    rgba(173, 216, 230, 0.3) 25%,
    rgba(135, 206, 250, 0.5) 50%,
    rgba(173, 216, 230, 0.3) 75%
  );
  background-size: 200% 100%;
  animation: shimmer 1.5s infinite;
}
@keyframes shimmer {
  0% {
    background-position: -200% 0;
  }
  100% {
    background-position: 200% 0;
  }
}
.video {
  display: flex;
  flex-direction: row;
  gap: 0.75rem;
  align-items: center;
  margin-top: 1em;
}
.video .link {
  display: flex;
  flex-direction: row;
  flex: 1 1 0;
  gap: 0.125rem;
  outline-offset: 4px;
  cursor: pointer;
}
.video .info {
  display: flex;
  flex-direction: column;
  justify-content: center;
  margin-left: 8px;
  gap: 0.125rem;
}
.video .info:hover {
  text-decoration: underline;
}
.video-title {
  font-size: 15px;
  line-height: 1.25;
  font-weight: 700;
  color: #23272f;
}
.video-title.loading {
  height: 20px;
  width: 80px;
  border-radius: 0.5rem;
}
.video-description {
  color: #5e687e;
  font-size: 13px;
  border-radius: 0.5rem;
}
.video-description.loading {
  height: 15px;
  width: 100px;
}
```

```json package.json hidden
{
  "dependencies": {
    "react": "canary",
    "react-dom": "canary",
    "react-scripts": "latest"
  }
}
```

</Sandpack>

<Note>

将 `<ViewTransition>` 放置在边界的不同位置，会决定回退内容和实际内容是作为一次更新进行交叉淡化，还是分别进行退出和进入动画。你还可以使用 View Transition 类来[自定义动画](/reference/react/ViewTransition#customizing-animations)。

[详细了解如何从 Suspense 内容进行动画。](/reference/react/ViewTransition#animating-from-suspense-content)

</Note>

---

### <CanaryBadge /> 等待字体加载 {/*waiting-for-a-font-to-load*/}

当 [`<ViewTransition>`](/reference/react/ViewTransition) 为 Suspense 边界的显示设置动画时，React 会等待内容引入的新字体，直到超时为止，这样文本就不会以备用字体闪现。此行为仅发生在 `<ViewTransition>` 更新期间。

在下面的示例中，Suspense 边界被包裹在 `<ViewTransition>` 中，并且 `Quote` 组件会在其数据加载时暂停。渲染引用内容会开始下载字体。React 会保持备用内容可见，直到字体加载完成，因此引用内容一开始就会以其指定的字体显示。

作为对比，第二个按钮执行相同的更新，但不使用 React。没有任何机制等待字体加载，因此文本会先以备用字体显示，然后再切换：

<Sandpack>

```js
import { ViewTransition, Suspense, use, useState, startTransition } from 'react';
import { fetchQuote } from './data.js';
import { freshFontUrl } from './font.js';
import VanillaQuote from './VanillaQuote.js';

function Quote({ fontSrc }) {
  const quote = use(fetchQuote());
  return (
    <>
      <style href={fontSrc} precedence="default">
        {`@font-face {
          font-family: 'Fancy';
          src: url(${fontSrc}) format('truetype');
          font-display: swap;
        }`}
      </style>
      <p className="quote fancy">{quote}</p>
    </>
  );
}

export default function App() {
  const [fontSrc, setFontSrc] = useState(null);
  return (
    <>
      <button
        onClick={() => {
          startTransition(() => {
            setFontSrc(freshFontUrl());
          });
        }}>
        Show quote
      </button>
      {fontSrc && (
        <ViewTransition>
          <Suspense fallback={<p className="quote">⌛ Loading quote...</p>}>
            <Quote fontSrc={fontSrc} />
          </Suspense>
        </ViewTransition>
      )}
      <hr />
      <VanillaQuote />
    </>
  );
}
```

```js src/VanillaQuote.js
import { useRef } from 'react';
import { freshFontUrl } from './font.js';

export default function VanillaQuote() {
  const ref = useRef(null);
  function show() {
    const style = document.createElement('style');
    style.textContent = `@font-face {
      font-family: 'VanillaFancy';
      src: url(${freshFontUrl()}) format('truetype');
      font-display: swap;
    }`;
    document.head.appendChild(style);
    ref.current.innerHTML = `<p class="quote vanilla-fancy">The best way to predict the future is to invent it.</p>`;
  }
  return (
    <>
      <button onClick={show}>Show quote (without React)</button>
      <div ref={ref} />
    </>
  );
}
```

```js src/font.js hidden
// 添加一个唯一参数，使字体不会被缓存，
// 并确保每次运行都会显示加载状态。
export function freshFontUrl() {
  return (
    'https://raw.githubusercontent.com/google/fonts/main/ofl/caveat/Caveat%5Bwght%5D.ttf' +
    '?t=' +
    Date.now()
  );
}
```

```js src/data.js hidden
// 注意：数据获取的具体方式取决于
// 你与 Suspense 一起使用的框架。
// 通常，缓存逻辑应位于框架内部。

let cache = null;

export function fetchQuote() {
  if (!cache) {
    cache = new Promise((resolve) => {
      // 添加一个人为的延迟，让等待过程更加明显。
      setTimeout(() => {
        resolve(
          'The best way to predict the future is to invent it.'
        );
      }, 500);
    });
  }
  return cache;
}
```

```css
#root {
  min-height: 260px;
}
.quote {
  font-size: 20px;
  margin-top: 1em;
}
.fancy {
  font-family: 'Fancy', sans-serif;
}
.vanilla-fancy {
  font-family: 'VanillaFancy', sans-serif;
}
hr {
  margin: 16px 0;
}
```

```json package.json hidden
{
  "dependencies": {
    "react": "canary",
    "react-dom": "canary",
    "react-scripts": "latest"
  }
}
```

</Sandpack>

---

### <CanaryBadge /> 等待图像加载 {/*waiting-for-an-image-to-load*/}

当 [`<ViewTransition>`](/reference/react/ViewTransition) 为 Suspense 边界的显示过程添加动画时，React 会等待可见图像加载完成，直到超时为止，这样动画就不会在图像加载一半时开始。此行为仅发生在 `<ViewTransition>` 更新期间。添加 `onLoad` 处理函数可以让特定图像跳过等待，即使它位于 `<ViewTransition>` 内部也是如此。

在下面的示例中，Suspense 边界被包裹在 `<ViewTransition>` 中，并在头像加载完成前显示个人资料骨架屏。

作为对比，第二个按钮执行相同的更新，但不使用 React。不会等待图像加载，因此卡片会立即出现，而图像加载完成后才显示：

<Sandpack>

```js
import { ViewTransition, Suspense, useState, startTransition } from 'react';
import { freshImageUrl } from './image.js';
import VanillaProfile from './VanillaProfile.js';

function Profile({ src }) {
  return (
    <div className="card">
      <img src={src} alt="Jack Pope" width={80} height={80} />
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
  const [src, setSrc] = useState(null);
  return (
    <>
      <button
        onClick={() => {
          startTransition(() => {
            setSrc(freshImageUrl());
          });
        }}>
        显示个人资料
      </button>
      {src && (
        <ViewTransition>
          <Suspense fallback={<ProfilePlaceholder />}>
            <Profile src={src} />
          </Suspense>
        </ViewTransition>
      )}
      <hr />
      <VanillaProfile />
    </>
  );
}
```

```js src/VanillaProfile.js
import { useRef } from 'react';
import { freshImageUrl } from './image.js';

export default function VanillaProfile() {
  const ref = useRef(null);
  function show() {
    ref.current.innerHTML = `<div class="card">
      <img src="${freshImageUrl()}" alt="Jack Pope" width="80" height="80" />
      <p>Jack Pope</p>
    </div>`;
  }
  return (
    <>
      <button onClick={show}>显示个人资料（不使用 React）</button>
      <div ref={ref} />
    </>
  );
}
```

```js src/image.js hidden
// 添加一个唯一参数，使图像不会被缓存，
// 并确保每次运行都显示加载状态。
export function freshImageUrl() {
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
    "react": "canary",
    "react-dom": "canary",
    "react-scripts": "latest"
  }
}
```

</Sandpack>

---

### <CanaryBadge /> 协调字体、图像和样式表 {/*coordinating-fonts-images-and-stylesheets*/}

Suspense 边界可以同时等待数据、样式表、字体和图像。等待字体和图像只会发生在 [`<ViewTransition>`](/reference/react/ViewTransition) 更新期间。在下面的示例中，`ProfileCard` 组件会在数据加载时暂停渲染，并使用 `precedence` 渲染一个样式表、以新字体显示文本，以及渲染一幅肖像。React 会在数据和样式表加载期间保持骨架屏可见。随后，`<ViewTransition>` 的显示过程会等待字体和图像，因此卡片会以完整状态出现。

作为对比，不使用 React 的版本会加载相同的数据，并让每个资源按照自己的时间表到达：

<Sandpack>

```js
import { ViewTransition, Suspense, use, useState, startTransition } from 'react';
import { fetchQuote } from './data.js';
import { freshStylesheetUrl, freshImageUrl } from './resources.js';
import VanillaProfileCard from './VanillaProfileCard.js';

function ProfileCard({ resources }) {
  const quote = use(resources.quotePromise);
  return (
    <>
      <link rel="stylesheet" href={resources.stylesheet} precedence="default" />
      <div className="profile-card">
        <img src={resources.image} alt="Jack Pope" width={80} height={80} />
        <div>
          <p className="name">Jack Pope</p>
          <p className="bio">{quote}</p>
        </div>
      </div>
    </>
  );
}

function ProfileCardPlaceholder() {
  return (
    <div className="profile-card">
      <div className="avatar-placeholder" />
      <div>
        <p className="name name-placeholder">&nbsp;</p>
        <p className="bio bio-placeholder">&nbsp;</p>
      </div>
    </div>
  );
}

export default function App() {
  const [resources, setResources] = useState(null);
  return (
    <>
      <button
        onClick={() => {
          startTransition(() => {
            setResources({
              quotePromise: fetchQuote(),
              stylesheet: freshStylesheetUrl(),
              image: freshImageUrl(),
            });
          });
        }}>
        显示个人资料
      </button>
      {resources && (
        <ViewTransition>
          <Suspense fallback={<ProfileCardPlaceholder />}>
            <ProfileCard resources={resources} />
          </Suspense>
        </ViewTransition>
      )}
      <hr />
      <VanillaProfileCard />
    </>
  );
}
```

```js src/VanillaProfileCard.js
import { useRef } from 'react';
import { fetchQuote } from './data.js';
import { freshStylesheetUrl, freshImageUrl } from './resources.js';

export default function VanillaProfileCard() {
  const ref = useRef(null);
  async function show() {
    const quote = await fetchQuote();
    const doc = ref.current.contentWindow.document;
    doc.open();
    doc.write(`
      <style>
        body { margin: 0; font-family: sans-serif; }
        .profile-card { display: flex; gap: 12px; align-items: center; }
        .profile-card img { border-radius: 50%; background: #dfe3e9; }
        .name { margin: 0 0 4px; font-family: 'Caveat', sans-serif; font-size: 22px; line-height: 28px; font-weight: bold; }
        .bio { margin: 0; font-family: 'Caveat', sans-serif; font-size: 20px; line-height: 26px; }
      </style>
      <div class="profile-card">
        <img src="${freshImageUrl()}" alt="Jack Pope" width="80" height="80" />
        <div>
          <p class="name">Jack Pope</p>
          <p class="bio">${quote}</p>
        </div>
      </div>
      <link rel="stylesheet" href="${freshStylesheetUrl()}">
    `);
    doc.close();
  }
  return (
    <>
      <button onClick={show}>显示个人资料（不使用 React）</button>
      <iframe ref={ref} title="Vanilla 个人资料卡片" className="vanilla-frame" />
    </>
  );
}
```

```js src/resources.js hidden
// 添加一个唯一参数，使资源不会被缓存，
// 并确保每次运行都显示加载状态。
export function freshStylesheetUrl() {
  return (
    'https://fonts.googleapis.com/css2?family=Caveat&display=swap' +
    '&t=' +
    Date.now()
  );
}

export function freshImageUrl() {
  return 'https://react.dev/images/team/jack-pope.jpg?t=' + Date.now();
}
```

```js src/data.js hidden
// 注意：数据获取的具体方式取决于你使用的框架，
// 以及该框架如何与 Suspense 配合使用。

export async function fetchQuote() {
  // 添加一个人为的延迟，让等待过程更明显。
  await new Promise((resolve) => {
    setTimeout(resolve, 1000);
  });
  return 'The best way to predict the future is to invent it.';
}
```

```css
#root {
  min-height: 320px;
}
button {
  margin-right: 8px;
}
hr {
  margin: 16px 0;
}
.profile-card {
  display: flex;
  gap: 12px;
  align-items: center;
  margin-top: 1em;
}
.profile-card img {
  border-radius: 50%;
  background: #dfe3e9;
}
.name {
  margin: 0 0 4px;
  font-family: 'Caveat', sans-serif;
  font-size: 22px;
  line-height: 28px;
  font-weight: bold;
}
.bio {
  margin: 0;
  font-family: 'Caveat', sans-serif;
  font-size: 20px;
  line-height: 26px;
}
.profile-card img {
  display: block;
}
.avatar-placeholder {
  width: 80px;
  height: 80px;
  border-radius: 50%;
  background: #dfe3e9;
}
.name-placeholder,
.bio-placeholder {
  border-radius: 4px;
  background: #dfe3e9;
  color: transparent;
}
.name-placeholder {
  width: 90px;
}
.bio-placeholder {
  width: 220px;
}
.vanilla-frame {
  display: block;
  margin-top: 1em;
  border: none;
  width: 100%;
  height: 110px;
}
```

```json package.json hidden
{
  "dependencies": {
    "react": "canary",
    "react-dom": "canary",
    "react-scripts": "latest"
  }
}
```

</Sandpack>

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
