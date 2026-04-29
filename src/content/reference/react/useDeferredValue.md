---
title: useDeferredValue
---

<Intro>

`useDeferredValue` 是一个 React Hook，它允许你延迟更新 UI 的某一部分。

```js
const deferredValue = useDeferredValue(value)
```

</Intro>

<InlineToc />

---

## 参考 {/*reference*/}

### `useDeferredValue(value, initialValue?)` {/*usedeferredvalue*/}

在组件顶层调用 `useDeferredValue`，以获取该值的延迟版本。

```js
import { useState, useDeferredValue } from 'react';

function SearchPage() {
  const [query, setQuery] = useState('');
  const deferredQuery = useDeferredValue(query);
  // ...
}
```

[查看下面的更多示例。](#usage)

#### 参数 {/*parameters*/}

* `value`：你想要延迟的值。它可以是任何类型。
* **可选** `initialValue`：组件初始渲染期间使用的值。如果省略此选项，`useDeferredValue` 在初始渲染期间不会延迟，因为没有可供替代渲染的 `value` 的上一个版本。


#### 返回值 {/*returns*/}

- `currentValue`：在初始渲染期间，返回的延迟值将是 `initialValue`，或者与你提供的值相同。在更新期间，React 会先尝试使用旧值重新渲染（因此它会返回旧值），然后在后台使用新值尝试另一次重新渲染（因此它会返回更新后的值）。

#### 注意事项 {/*caveats*/}

- 当更新位于 Transition 中时，`useDeferredValue` 总是返回新的 `value`，并且不会生成延迟渲染，因为该更新已经被延迟了。

- 传给 `useDeferredValue` 的值应该是原始值（如字符串和数字），或者是在渲染之外创建的对象。如果你在渲染期间创建了一个新对象并立即将其传给 `useDeferredValue`，那么它在每次渲染时都会不同，从而导致不必要的后台重新渲染。

- 当 `useDeferredValue` 接收到不同的值（与 [`Object.is`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is) 比较），除了当前渲染（此时它仍然使用之前的值）之外，它还会在后台调度一次使用新值的重新渲染。后台重新渲染是可中断的：如果 `value` 又有了新的更新，React 会从头重新开始后台重新渲染。例如，如果用户输入到输入框中的速度快于图表基于其延迟值重新渲染的速度，那么图表只有在用户停止输入后才会重新渲染。

- `useDeferredValue` 与 [`<Suspense>`](/reference/react/Suspense) 集成。如果由新值引起的后台更新使 UI 挂起，用户不会看到回退内容。他们会一直看到旧的延迟值，直到数据加载完成。

- `useDeferredValue` 本身不会阻止额外的网络请求。

- `useDeferredValue` 本身不会造成固定延迟。只要 React 完成原始重新渲染，React 就会立即开始处理使用新延迟值的后台重新渲染。由事件（如输入）引起的任何更新都会中断后台重新渲染，并获得比它更高的优先级。

- `useDeferredValue` 引起的后台重新渲染在提交到屏幕之前不会触发 Effects。如果后台重新渲染挂起，其 Effects 会在数据加载完成且 UI 更新后运行。

---

## 用法 {/*usage*/}

### 在新内容加载时显示旧内容 {/*showing-stale-content-while-fresh-content-is-loading*/}

在组件顶层调用 `useDeferredValue`，以延迟更新 UI 的某一部分。

```js [[1, 5, "query"], [2, 5, "deferredQuery"]]
import { useState, useDeferredValue } from 'react';

function SearchPage() {
  const [query, setQuery] = useState('');
  const deferredQuery = useDeferredValue(query);
  // ...
}
```

在初始渲染期间，<CodeStep step={2}>延迟值</CodeStep>将与您提供的<CodeStep step={1}>值</CodeStep>相同。

在更新期间，<CodeStep step={2}>延迟值</CodeStep>会“落后”于最新的<CodeStep step={1}>值</CodeStep>。具体来说，React 会首先在*不*更新延迟值的情况下重新渲染，然后尝试在后台使用新接收到的值重新渲染。

**让我们通过一个示例来看看这在什么时候有用。**

<Note>

此示例假设你使用的是支持 Suspense 的数据源：

- 使用支持 Suspense 的框架进行数据获取，例如 [Relay](https://relay.dev/docs/guided-tour/rendering/loading-states/) 和 [Next.js](https://nextjs.org/docs/app/getting-started/fetching-data#with-suspense)
- 使用 [`lazy`](/reference/react/lazy) 懒加载组件代码
- 使用 [`use`](/reference/react/use) 读取 Promise 的值

[了解有关 Suspense 及其限制的更多信息。](/reference/react/Suspense)

</Note>


在此示例中，`SearchResults` 组件在获取搜索结果时会[挂起](/reference/react/Suspense#displaying-a-fallback-while-content-is-loading)。试着输入 `"a"`，等待结果出现，然后把它改成 `"ab"`。`"a"` 的结果会被加载中的回退内容替换。

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
      <Suspense fallback={<h2>加载中...</h2>}>
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
    return <p>没有找到与 <i>"{query}"</i> 匹配的结果</p>;
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
// 注意：你进行数据获取的方式取决于
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
    throw Error('未实现');
  }
}

async function getSearchResults(query) {
  // 添加一个假的延迟，让等待变得明显。
  await new Promise(resolve => {
    setTimeout(resolve, 1000);
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

一种常见的替代 UI 模式是*延迟*更新结果列表，并在新结果准备好之前继续显示旧结果。调用 `useDeferredValue` 将查询的延迟版本向下传递：

```js {3,11}
export default function App() {
  const [query, setQuery] = useState('');
  const deferredQuery = useDeferredValue(query);
  return (
    <>
      <label>
        Search albums:
        <input value={query} onChange={e => setQuery(e.target.value)} />
      </label>
      <Suspense fallback={<h2>Loading...</h2>}>
        <SearchResults query={deferredQuery} />
      </Suspense>
    </>
  );
}
```

`query` 会立即更新，因此输入框会显示新值。然而，`deferredQuery` 会在数据加载完成之前保持其之前的值，因此 `SearchResults` 会短暂显示旧结果。

在下面的示例中输入 `"a"`，等待结果加载，然后将输入框编辑为 `"ab"`。注意，此时显示的不是 Suspense 回退内容，而是在新结果加载完成之前一直显示旧的结果列表：

<Sandpack>

```js src/App.js
import { Suspense, useState, useDeferredValue } from 'react';
import SearchResults from './SearchResults.js';

export default function App() {
  const [query, setQuery] = useState('');
  const deferredQuery = useDeferredValue(query);
  return (
    <>
      <label>
        搜索专辑：
        <input value={query} onChange={e => setQuery(e.target.value)} />
      </label>
      <Suspense fallback={<h2>加载中...</h2>}>
        <SearchResults query={deferredQuery} />
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
    return <p>没有找到与 <i>"{query}"</i> 匹配的结果</p>;
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
// 注意：你进行数据获取的方式取决于
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
    throw Error('未实现');
  }
}

async function getSearchResults(query) {
  // 添加一个假的延迟，让等待变得明显。
  await new Promise(resolve => {
    setTimeout(resolve, 1000);
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

<DeepDive>

#### 延迟值在底层是如何工作的？ {/*how-does-deferring-a-value-work-under-the-hood*/}

你可以把它理解为分两步发生：

1. **首先，React 使用新的 `query`（`"ab"`），但仍使用旧的 `deferredQuery`（还是 `"a"`）重新渲染。** 你传给结果列表的 `deferredQuery` 值是*延迟的*：它会“落后”于 `query` 值。

2. **然后在后台，React 尝试将 `query` 和 `deferredQuery` 都更新为 `"ab"` 并重新渲染。** 如果这次重新渲染完成，React 就会把它显示到屏幕上。然而，如果它挂起了（`"ab"` 的结果尚未加载），React 会放弃这次渲染尝试，并在数据加载完成后再次重试。用户会一直看到旧的延迟值，直到数据准备就绪。

这种延迟的“后台”渲染是可中断的。例如，如果你再次在输入框中输入，React 会放弃当前渲染并使用新值重新开始。React 总是会使用最新提供的值。

请注意，每次按键仍然会发起一次网络请求。这里被延迟的是结果的显示（直到它们准备好），而不是网络请求本身。即使用户继续输入，每次按键对应的响应也会被缓存，因此按退格键会立刻生效，并且不会再次发起请求。

</DeepDive>

---

### 指示内容已过期 {/*indicating-that-the-content-is-stale*/}

在上面的示例中，没有任何提示表明最新查询的结果列表仍在加载中。如果新结果需要较长时间才能加载，这可能会让用户感到困惑。为了更明显地告诉用户结果列表与最新查询不一致，你可以在显示旧结果列表时添加一个视觉提示：

```js {2}
<div style={{
  opacity: query !== deferredQuery ? 0.5 : 1,
}}>
  <SearchResults query={deferredQuery} />
</div>
```

这样改完后，一旦你开始输入，旧的结果列表就会稍微变暗，直到新的结果列表加载完成。你也可以添加 CSS 过渡效果来延迟变暗，使其看起来更平滑，就像下面这个示例一样：

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
        Search albums:
        <input value={query} onChange={e => setQuery(e.target.value)} />
      </label>
      <Suspense fallback={<h2>Loading...</h2>}>
        <div style={{
          opacity: isStale ? 0.5 : 1,
          transition: isStale ? 'opacity 0.2s 0.2s linear' : 'opacity 0s 0s linear'
        }}>
          <SearchResults query={deferredQuery} />
        </div>
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
    return <p>没有找到与 <i>"{query}"</i> 匹配的结果</p>;
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
// 注意：你进行数据获取的方式取决于
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
    throw Error('未实现');
  }
}

async function getSearchResults(query) {
  // 添加一个假的延迟，让等待变得明显。
  await new Promise(resolve => {
    setTimeout(resolve, 1000);
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

---

### 延迟 UI 某一部分的重新渲染 {/*deferring-re-rendering-for-a-part-of-the-ui*/}

你也可以将 `useDeferredValue` 作为一种性能优化手段。当 UI 的某一部分重新渲染很慢、又没有很容易的优化方法、并且你想避免它阻塞其余 UI 时，这会很有用。

假设你有一个文本字段和一个组件（比如图表或长列表），它会在每次按键时重新渲染：

```js
function App() {
  const [text, setText] = useState('');
  return (
    <>
      <input value={text} onChange={e => setText(e.target.value)} />
      <SlowList text={text} />
    </>
  );
}
```

首先，优化 `SlowList` 以在其 props 相同时跳过重新渲染。为此，[将其包裹在 `memo` 中：](/reference/react/memo#skipping-re-rendering-when-props-are-unchanged)

```js {1,3}
const SlowList = memo(function SlowList({ text }) {
  // ...
});
```

然而，这只有在 `SlowList` 的 props 与上一次渲染时*相同*时才有帮助。你现在面临的问题是，当它们*不同*时会很慢，而这正是你真正需要显示不同视觉输出的时候。

具体来说，主要的性能问题在于，每当你在输入框中输入时，`SlowList` 都会接收到新的 props，重新渲染整个树会让输入变得卡顿。在这种情况下，`useDeferredValue` 允许你将更新输入框（必须很快）的优先级高于更新结果列表（可以稍慢）：

```js {3,7}
function App() {
  const [text, setText] = useState('');
  const deferredText = useDeferredValue(text);
  return (
    <>
      <input value={text} onChange={e => setText(e.target.value)} />
      <SlowList text={deferredText} />
    </>
  );
}
```

这不会让 `SlowList` 的重新渲染更快。然而，它会告诉 React，可以降低列表重新渲染的优先级，从而不阻塞按键。列表会“落后”于输入，然后再“赶上”来。和之前一样，React 会尽快尝试更新列表，但不会阻止用户输入。

<Recipes titleText="useDeferredValue 与未优化重新渲染之间的差异" titleId="examples">

#### 列表的延迟重新渲染 {/*deferred-re-rendering-of-the-list*/}

在这个示例中，`SlowList` 组件中的每个条目都被**人为减速**，这样你就能看到 `useDeferredValue` 如何让输入保持响应。输入内容时，请注意键入依然很流畅，而列表会“落后”于输入。

<Sandpack>

```js
import { useState, useDeferredValue } from 'react';
import SlowList from './SlowList.js';

export default function App() {
  const [text, setText] = useState('');
  const deferredText = useDeferredValue(text);
  return (
    <>
      <input value={text} onChange={e => setText(e.target.value)} />
      <SlowList text={deferredText} />
    </>
  );
}
```

```js {expectedErrors: {'react-compiler': [19, 20]}} src/SlowList.js
import { memo } from 'react';

const SlowList = memo(function SlowList({ text }) {
  // 只记录一次日志。真正的减速在 SlowItem 内部。
  console.log('[ARTIFICIALLY SLOW] Rendering 250 <SlowItem />');

  let items = [];
  for (let i = 0; i < 250; i++) {
    items.push(<SlowItem key={i} text={text} />);
  }
  return (
    <ul className="items">
      {items}
    </ul>
  );
});

function SlowItem({ text }) {
  let startTime = performance.now();
  while (performance.now() - startTime < 1) {
    // 每个条目空转 1 毫秒，以模拟极慢的代码
  }

  return (
    <li className="item">
      文本：{text}
    </li>
  )
}

export default SlowList;
```

```css
.items {
  padding: 0;
}

.item {
  list-style: none;
  display: block;
  height: 40px;
  padding: 5px;
  margin-top: 10px;
  border-radius: 4px;
  border: 1px solid #aaa;
}
```

</Sandpack>

<Solution />

#### 未优化的列表重新渲染 {/*unoptimized-re-rendering-of-the-list*/}

在这个示例中，`SlowList` 组件中的每个条目都被**人为减速**，但没有使用 `useDeferredValue`。

注意，输入内容时会感觉非常卡顿。这是因为没有 `useDeferredValue` 时，每次按键都会迫使整个列表立即以不可中断的方式重新渲染。

<Sandpack>

```js
import { useState } from 'react';
import SlowList from './SlowList.js';

export default function App() {
  const [text, setText] = useState('');
  return (
    <>
      <input value={text} onChange={e => setText(e.target.value)} />
      <SlowList text={text} />
    </>
  );
}
```

```js {expectedErrors: {'react-compiler': [19, 20]}} src/SlowList.js
import { memo } from 'react';

const SlowList = memo(function SlowList({ text }) {
  // 只记录一次日志。真正的减速在 SlowItem 内部。
  console.log('[ARTIFICIALLY SLOW] Rendering 250 <SlowItem />');

  let items = [];
  for (let i = 0; i < 250; i++) {
    items.push(<SlowItem key={i} text={text} />);
  }
  return (
    <ul className="items">
      {items}
    </ul>
  );
});

function SlowItem({ text }) {
  let startTime = performance.now();
  while (performance.now() - startTime < 1) {
    // 每个条目空转 1 毫秒，以模拟极慢的代码
  }

  return (
    <li className="item">
      文本：{text}
    </li>
  )
}

export default SlowList;
```

```css
.items {
  padding: 0;
}

.item {
  list-style: none;
  display: block;
  height: 40px;
  padding: 5px;
  margin-top: 10px;
  border-radius: 4px;
  border: 1px solid #aaa;
}
```

</Sandpack>

<Solution />

</Recipes>

<Pitfall>

这种优化要求 `SlowList` 被 [`memo`](/reference/react/memo) 包裹。这是因为每当 `text` 改变时，React 需要能够快速重新渲染父组件。在那次重新渲染期间，`deferredText` 仍然保持其之前的值，因此 `SlowList` 可以跳过重新渲染（它的 props 没有变化）。如果没有 [`memo`](/reference/react/memo)，它无论如何都必须重新渲染，这就失去了优化的意义。

</Pitfall>

<DeepDive>

#### 延迟值与防抖和节流有何不同？ {/*how-is-deferring-a-value-different-from-debouncing-and-throttling*/}

在这种场景中，你之前可能用过两种常见的优化技术：

- *防抖* 意味着你会等用户停止输入（例如等一秒）后再更新列表。
- *节流* 意味着你会每隔一段时间更新一次列表（例如最多每秒一次）。

虽然这些技术在某些情况下很有帮助，但 `useDeferredValue` 更适合优化渲染，因为它与 React 本身深度集成，并且会适应用户的设备。

与防抖或节流不同，它不需要选择任何固定延迟。如果用户的设备很快（例如性能强劲的笔记本电脑），延迟重新渲染几乎会立刻发生，不会被察觉。如果用户的设备较慢，列表就会按设备速度相应地“落后”于输入。

此外，与防抖或节流不同，`useDeferredValue` 所做的延迟重新渲染默认是可中断的。这意味着，如果 React 正在重新渲染一个大型列表，而用户又输入了一个按键，React 会放弃那次重新渲染，处理这个按键，然后再次在后台开始渲染。相比之下，防抖和节流仍然会带来卡顿体验，因为它们是*阻塞式*的：它们只是推迟了渲染阻塞按键的时刻。

如果你要优化的工作并不发生在渲染期间，防抖和节流仍然很有用。例如，它们可以让你发起更少的网络请求。你也可以把这些技术结合起来使用。

</DeepDive>
