---
title: cacheSignal
---

<RSC>

`cacheSignal` 目前仅与 [React Server Components](/blog/2023/03/22/react-labs-what-we-have-been-working-on-march-2023#react-server-components) 一起使用。

</RSC>

<Intro>

`cacheSignal` 允许你知道 `cache()` 的生命周期何时结束。

```js
const signal = cacheSignal();
```

</Intro>

<InlineToc />

---

## 参考 {/*reference*/}

### `cacheSignal` {/*cachesignal*/}

调用 `cacheSignal` 以获取一个 `AbortSignal`。

```js {3,7}
import {cacheSignal} from 'react';
async function Component() {
  await fetch(url, { signal: cacheSignal() });
}
```

当 React 完成渲染时，`AbortSignal` 将被中止。这使你能够取消任何不再需要的进行中工作。
渲染被视为完成的时机包括：
- React 已成功完成渲染
- 渲染被中止
- 渲染失败

#### 参数 {/*parameters*/}

此函数不接受任何参数。

#### 返回值 {/*returns*/}

如果在渲染期间调用，`cacheSignal` 会返回一个 `AbortSignal`。否则，`cacheSignal()` 返回 `null`。

#### 注意事项 {/*caveats*/}

- `cacheSignal` 目前仅供 [React Server Components](/reference/rsc/server-components) 使用。在 Client Components 中，它始终返回 `null`。未来它也将用于 Client Component，当客户端缓存刷新或失效时。你不应假设它在客户端上始终为 null。
- 如果在渲染之外调用，`cacheSignal` 将返回 `null`，以明确当前作用域并未被永久缓存。

---

## 用法 {/*usage*/}

### 取消进行中的请求 {/*cancel-in-flight-requests*/}

调用 <CodeStep step={1}>`cacheSignal`</CodeStep> 来中止进行中的请求。

```js [[1, 4, "cacheSignal()"]]
import {cache, cacheSignal} from 'react';
const dedupedFetch = cache(fetch);
async function Component() {
  await dedupedFetch(url, { signal: cacheSignal() });
}
```

<Pitfall>
你不能使用 `cacheSignal` 来中止在渲染之外开始的异步工作，例如：

```js
import {cacheSignal} from 'react';
// 🚩 误区：如果 `Component` 的渲染已经完成，请求实际上不会被中止。
const response = fetch(url, { signal: cacheSignal() });
async function Component() {
  await response;
}
```
</Pitfall>

### 在 React 完成渲染后忽略错误 {/*ignore-errors-after-react-has-finished-rendering*/}

如果一个函数抛出错误，这可能是由于取消导致的（例如 <CodeStep step={1}>数据库连接</CodeStep> 已被关闭）。你可以使用 <CodeStep step={2}>`aborted` 属性</CodeStep> 来检查错误是由于取消还是实际错误。你可能希望<CodeStep step={3}>忽略</CodeStep>那些由取消引起的错误。

```js [[1, 2, "./database"], [2, 8, "cacheSignal()?.aborted"], [3, 12, "return null"]]
import {cacheSignal} from "react";
import {queryDatabase, logError} from "./database";

async function getData(id) {
  try {
     return await queryDatabase(id);
  } catch (x) {
     if (!cacheSignal()?.aborted) {
        // 仅在这是真实错误而非由取消引起时才记录
       logError(x);
     }
     return null;
  }
}

async function Component({id}) {
  const data = await getData(id);
  if (data === null) {
    return <div>没有可用数据</div>;
  }
  return <div>{data.name}</div>;
}
```
