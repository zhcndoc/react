---
title: RSC 沙箱测试
---

## 基础服务端组件 {/*basic-server-component*/}

<SandpackRSC>

```js src/App.js
export default function App() {
  return <h1>来自服务端组件的问候！</h1>;
}
```

</SandpackRSC>

## 服务端 + 客户端组件 {/*server-client*/}

<SandpackRSC>

```js src/App.js
import Counter from './Counter';

export default function App() {
  return (
    <div>
      <h1>服务端组件</h1>
      <p>这段文本在服务端渲染。</p>
      <Counter />
    </div>
  );
}
```

```js src/Counter.js
'use client';
import { useState } from 'react';

export default function Counter() {
  const [count, setCount] = useState(0);
  return (
    <button onClick={() => setCount(count + 1)}>
      计数：{count}
    </button>
  );
}
```

</SandpackRSC>

## 带有 Suspense 的异步服务端组件 {/*async-suspense*/}

<SandpackRSC>

```js src/App.js
import { Suspense } from 'react';
import Albums from './Albums';

export default function App() {
  return (
    <div>
      <h1>音乐</h1>
      <Suspense fallback={<p>正在加载专辑...</p>}>
        <Albums />
      </Suspense>
    </div>
  );
}
```

```js src/Albums.js
async function fetchAlbums() {
  await new Promise(resolve => setTimeout(resolve, 1000));
  return ['Abbey Road', 'Let It Be', 'Revolver'];
}

export default async function Albums() {
  const albums = await fetchAlbums();
  return (
    <ul>
      {albums.map(album => (
        <li key={album}>{album}</li>
      ))}
    </ul>
  );
}
```

</SandpackRSC>

## 流式传输证明 {/*streaming-proof*/}

此演示证明流式传输是增量式的。外壳会立即渲染，并带有一个 `<Suspense>` 回退内容。2 秒后，异步组件流式传入并替换它——而不会重新渲染外层内容。时间戳显示了这段间隔。

<SandpackRSC>

```js src/App.js
import { Suspense } from 'react';
import SlowData from './SlowData';
import Timestamp from './Timestamp';

export default function App() {
  return (
    <div>
      <h1>流式传输证明</h1>
      <p>外壳渲染于：<Timestamp /></p>
      <Suspense fallback={<p>⏳ 等待数据流入...</p>}>
        <SlowData />
      </Suspense>
    </div>
  );
}
```

```js src/SlowData.js
import Timestamp from './Timestamp';

async function fetchData() {
  await new Promise(resolve => setTimeout(resolve, 2000));
  return ['Chunk A', 'Chunk B', 'Chunk C'];
}

export default async function SlowData() {
  const items = await fetchData();
  return (
    <div>
      <p>数据流入于：<Timestamp /></p>
      <ul>
        {items.map(item => (
          <li key={item}>{item}</li>
        ))}
      </ul>
    </div>
  );
}
```

```js src/Timestamp.js
'use client';

export default function Timestamp() {
  return <strong>{new Date().toLocaleTimeString()}</strong>;
}
```

</SandpackRSC>

## Flight 数据类型 {/*flight-data-types*/}

此演示将 Map、Set、Date 和 BigInt 从服务端组件通过 Flight 流传递给客户端组件，证明完整的 Flight 协议类型系统端到端可用。

<SandpackRSC>

```js src/App.js
import DataViewer from './DataViewer';

export default function App() {
  const map = new Map([
    ['alice', 100],
    ['bob', 200],
  ]);
  const set = new Set(['react', 'next', 'remix']);
  const date = new Date('2025-06-15T12:00:00Z');
  const big = 9007199254740993n;

  return (
    <div>
      <h1>Flight 数据类型</h1>
      <DataViewer map={map} set={set} date={date} big={big} />
    </div>
  );
}
```

```js src/DataViewer.js
'use client';

export default function DataViewer({ map, set, date, big }) {
  const checks = [
    ['Map', map instanceof Map, () => (
      <ul>{[...map.entries()].map(([k, v]) => <li key={k}>{k}: {v}</li>)}</ul>
    )],
    ['Set', set instanceof Set, () => (
      <ul>{[...set].map(v => <li key={v}>{v}</li>)}</ul>
    )],
    ['Date', date instanceof Date, () => (
      <p>{date.toISOString()}</p>
    )],
    ['BigInt', typeof big === 'bigint', () => (
      <p>{big.toString()}</p>
    )],
  ];

  return (
    <div>
      {checks.map(([label, passed, render]) => (
        <div key={label} style={{ marginBottom: 12 }}>
          <strong>{label}: {passed ? '通过' : '失败'}</strong>
          {render()}
        </div>
      ))}
    </div>
  );
}
```

</SandpackRSC>

## 使用 use() 进行 Promise 流式传输 {/*promise-streaming-use*/}

服务端创建了一个 promise（2 秒后 resolve），并通过一个会挂起 3 秒的父级异步组件将其作为 prop 传递。当父级在约 3 秒时显示出来时，这个 promise 已经 resolved 了——因此 `use()` 会立即返回，不会出现内部回退内容。经过的时间应该约为 3000ms（父级的延迟），而不是约 5000ms（这意味着该 promise 在客户端重新开始了）。

<SandpackRSC>

```js src/App.js
import { Suspense } from 'react';
import SlowParent from './SlowParent';
import UserCard from './UserCard';

async function fetchUser() {
  await new Promise(resolve => setTimeout(resolve, 2000));
  return { name: 'Alice', role: 'Engineer' };
}

function now() {
  return Date.now();
}

export default function App() {
  const serverTime = now();
  const userPromise = fetchUser();
  return (
    <div>
      <h1>Promise 流式传输</h1>
      <p>Promise 2 秒后解析。父级挂起 3 秒。</p>
      <Suspense fallback={<p>外层：正在等待父级（3 秒）...</p>}>
        <SlowParent>
          <Suspense fallback={<p>内层：正在等待数据（不应出现！）</p>}>
            <UserCard userPromise={userPromise} serverTime={serverTime} />
          </Suspense>
        </SlowParent>
      </Suspense>
    </div>
  );
}
```

```js src/SlowParent.js
export default async function SlowParent({ children }) {
  await new Promise(resolve => setTimeout(resolve, 3000));
  return <div>{children}</div>;
}
```

```js src/UserCard.js
'use client';
import { use } from 'react';

function now() {
  return Date.now();
}
export default function UserCard({ userPromise, serverTime }) {
  const user = use(userPromise);
  const elapsed = now() - serverTime;
  return (
    <div style={{
      border: '1px solid #ccc',
      borderRadius: 8,
      padding: 16,
    }}>
      <strong>{user.name}</strong>
      <p>{user.role}</p>
      <p style={{ fontSize: 13 }}>
        在服务端创建该 promise 后经过 {elapsed}ms 渲染。
      </p>
      <p style={{ color: '#666', fontSize: 12 }}>
        ~3000ms = promise 已经解析，只等待了父级。
        ~5000ms 则表示 promise 在客户端重新开始了。
      </p>
    </div>
  );
}
```

</SandpackRSC>

## 服务端操作中的 Flight 数据类型 {/*flight-data-types-actions*/}

此演示通过 `encodeReply`/`decodeReply` 将 Map、Set、Date 和 BigInt 从客户端组件发送到服务端操作，然后验证这些类型在往返过程中是否保留。

<SandpackRSC>

```js src/App.js
import { testTypes, getResults } from './actions';
import TestButton from './TestButton';

export default async function App() {
  const results = await getResults();
  return (
    <div>
      <h1>服务端操作中的 Flight 类型</h1>
      <TestButton testTypes={testTypes} />
      {results ? (
        <div>
          {results.map(r => (
            <div key={r.label} style={{ marginBottom: 12 }}>
              <strong>{r.label}: {r.ok ? '通过' : '失败'}</strong>
              <p>{r.detail}</p>
            </div>
          ))}
        </div>
      ) : (
        <p>点击按钮将带类型的数据发送到服务端操作。</p>
      )}
    </div>
  );
}
```

```js src/actions.js
'use server';

let results = null;

export async function testTypes(map, set, date, big) {
  results = [
    {
      label: 'Map',
      ok: map instanceof Map,
      detail: map instanceof Map
        ? 'entries: ' + JSON.stringify([...map.entries()])
        : 'received: ' + typeof map,
    },
    {
      label: 'Set',
      ok: set instanceof Set,
      detail: set instanceof Set
        ? 'values: ' + JSON.stringify([...set])
        : 'received: ' + typeof set,
    },
    {
      label: 'Date',
      ok: date instanceof Date,
      detail: date instanceof Date
        ? date.toISOString()
        : 'received: ' + typeof date,
    },
    {
      label: 'BigInt',
      ok: typeof big === 'bigint',
      detail: typeof big === 'bigint'
        ? big.toString()
        : 'received: ' + typeof big,
    },
  ];
}

export async function getResults() {
  return results;
}
```

```js src/TestButton.js
'use client';
import { useTransition } from 'react';

export default function TestButton({ testTypes }) {
  const [pending, startTransition] = useTransition();

  function handleClick() {
    startTransition(async () => {
      await testTypes(
        new Map([['alice', 100], ['bob', 200]]),
        new Set(['react', 'next', 'remix']),
        new Date('2025-06-15T12:00:00Z'),
        9007199254740993n
      );
    });
  }

  return (
    <button onClick={handleClick} disabled={pending}>
      {pending ? '正在发送...' : '向服务端发送带类型的数据'}
    </button>
  );
}
```

</SandpackRSC>

## 服务端操作变更 + 重新渲染 {/*action-mutation-rerender*/}

服务端操作会修改服务端数据并返回确认字符串。更新后的列表之所以可见，是因为框架会在操作完成后自动重新渲染整个服务端组件树——服务端组件重新读取数据，并将新的 UI 流式传输给客户端。

<SandpackRSC>

```js src/App.js
import { getTodos } from './db';
import { createTodo } from './actions';
import AddTodo from './AddTodo';

export default function App() {
  const todos = getTodos();
  return (
    <div>
      <h1>待办事项列表</h1>
      <p style={{ color: '#666', fontSize: 13 }}>
        此列表由服务端组件渲染，
        读取的是服务端数据。它只会更新，因为
        服务端会在每次操作后重新渲染。
      </p>
      <ul>
        {todos.map((todo, i) => (
          <li key={i}>{todo}</li>
        ))}
      </ul>
      <AddTodo createTodo={createTodo} />
    </div>
  );
}
```

```js src/db.js
let todos = ['Buy groceries'];

export function getTodos() {
  return [...todos];
}

export function addTodo(text) {
  todos.push(text);
}
```

```js src/actions.js
'use server';
import { addTodo } from './db';

export async function createTodo(text) {
  if (!text) return '请输入一个待办事项。';
  addTodo(text);
  return '已添加：' + text;
}
```

```js src/AddTodo.js
'use client';
import { useState, useTransition } from 'react';

export default function AddTodo({ createTodo }) {
  const [text, setText] = useState('');
  const [message, setMessage] = useState('');
  const [pending, startTransition] = useTransition();

  function handleSubmit(e) {
    e.preventDefault();
    startTransition(async () => {
      const result = await createTodo(text);
      setMessage(result);
      setText('');
    });
  }

  return (
    <div>
      <form onSubmit={handleSubmit}>
        <input
          value={text}
          onChange={e => setText(e.target.value)}
          placeholder="新待办事项"
        />
        <button disabled={pending}>
          {pending ? '正在添加...' : '添加'}
        </button>
      </form>
      {message && (
        <p style={{ color: '#666', fontSize: 13 }}>
          操作返回：“{message}”
        </p>
      )}
    </div>
  );
}
```

</SandpackRSC>

## 行内服务器操作 {/*inline-server-actions*/}

在服务器组件中内联定义的服务器操作，在函数体内使用 `'use server'`。该操作会闭包捕获模块级状态，并作为 prop 传递——无需单独的 `actions.js` 文件。

<SandpackRSC>

```js src/App.js
import LikeButton from './LikeButton';

let count = 0;

export default function App() {
  async function addLike() {
    'use server';
    count++;
  }

  return (
    <div>
      <h1>行内服务器操作</h1>
      <p>点赞数：{count}</p>
      <LikeButton addLike={addLike} />
    </div>
  );
}
```

```js src/LikeButton.js
'use client';

export default function LikeButton({ addLike }) {
  return (
    <form action={addLike}>
      <button type="submit">点赞</button>
    </form>
  );
}
```

</SandpackRSC>

## 服务器函数 {/*server-functions*/}

<SandpackRSC>

```js src/App.js
import { addLike, getLikeCount } from './actions';
import LikeButton from './LikeButton';

export default async function App() {
  const count = await getLikeCount();
  return (
    <div>
      <h1>服务器函数</h1>
      <p>点赞数：{count}</p>
      <LikeButton addLike={addLike} />
    </div>
  );
}
```

```js src/actions.js
'use server';

let count = 0;

export async function addLike() {
  count++;
}

export async function getLikeCount() {
  return count;
}
```

```js src/LikeButton.js
'use client';

export default function LikeButton({ addLike }) {
  return (
    <form action={addLike}>
      <button type="submit">点赞</button>
    </form>
  );
}
```

</SandpackRSC>