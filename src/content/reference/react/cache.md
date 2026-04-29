---
title: cache
---

<RSC>

`cache` 仅适用于 [React Server Components](/reference/rsc/server-components)。

</RSC>

<Intro>

`cache` 允许你缓存数据获取或计算的结果。

```js
const cachedFn = cache(fn);
```

</Intro>

<InlineToc />

---

## 参考 {/*reference*/}

### `cache(fn)` {/*cache*/}

在任何组件外调用 `cache`，以创建带缓存的函数版本。

```js {4,7}
import {cache} from 'react';
import calculateMetrics from 'lib/metrics';

const getMetrics = cache(calculateMetrics);

function Chart({data}) {
  const report = getMetrics(data);
  // ...
}
```

当首次使用 `data` 调用 `getMetrics` 时，`getMetrics` 会调用 `calculateMetrics(data)`，并将结果存入缓存。如果再次使用相同的 `data` 调用 `getMetrics`，它会返回缓存结果，而不是再次调用 `calculateMetrics(data)`。

[查看更多示例。](#usage)

#### 参数 {/*parameters*/}

- `fn`：你希望缓存结果的函数。`fn` 可以接受任意参数并返回任意值。

#### 返回值 {/*returns*/}

`cache` 返回一个与 `fn` 具有相同类型签名的缓存版本。这个过程不会调用 `fn`。

当使用给定参数调用 `cachedFn` 时，它首先检查缓存中是否存在已缓存结果。如果存在，则返回该结果；如果不存在，则使用这些参数调用 `fn`，将结果存入缓存，并返回结果。只有在缓存未命中时才会调用 `fn`。

<Note>

根据输入缓存返回值的优化称为 [_memoization_](https://en.wikipedia.org/wiki/Memoization)。我们将 `cache` 返回的函数称为记忆化函数。

</Note>

#### 注意事项 {/*caveats*/}

- React 会在每次服务器请求时使所有记忆化函数的缓存失效。
- 每次调用 `cache` 都会创建一个新函数。这意味着用同一个函数多次调用 `cache` 会返回不同的记忆化函数，它们不共享同一缓存。
- `cachedFn` 也会缓存错误。如果 `fn` 对某些参数抛出错误，该错误会被缓存，并且当使用相同参数调用 `cachedFn` 时会重新抛出相同错误。
- `cache` 仅用于 [Server Components](/reference/rsc/server-components)。

---

## 用法 {/*usage*/}

### 缓存昂贵的计算 {/*cache-expensive-computation*/}

使用 `cache` 来跳过重复工作。

```js [[1, 7, "getUserMetrics(user)"],[2, 13, "getUserMetrics(user)"]]
import {cache} from 'react';
import calculateUserMetrics from 'lib/user';

const getUserMetrics = cache(calculateUserMetrics);

function Profile({user}) {
  const metrics = getUserMetrics(user);
  // ...
}

function TeamReport({users}) {
  for (let user in users) {
    const metrics = getUserMetrics(user);
    // ...
  }
  // ...
}
```

如果同一个 `user` 对象同时在 `Profile` 和 `TeamReport` 中渲染，这两个组件就可以共享工作，只需对该 `user` 调用一次 `calculateUserMetrics`。

假设 `Profile` 先渲染。它会调用 <CodeStep step={1}>`getUserMetrics`</CodeStep>，并检查是否存在缓存结果。由于这是第一次用该 `user` 调用 `getUserMetrics`，因此会发生缓存未命中。随后 `getUserMetrics` 会使用该 `user` 调用 `calculateUserMetrics`，并把结果写入缓存。

当 `TeamReport` 渲染其 `users` 列表并到达同一个 `user` 对象时，它会调用 <CodeStep step={2}>`getUserMetrics`</CodeStep>，并从缓存中读取结果。

如果 `calculateUserMetrics` 可以通过传入 [`AbortSignal`](https://developer.mozilla.org/en-US/docs/Web/API/AbortSignal) 中止，你可以使用 [`cacheSignal()`](/reference/react/cacheSignal) 在 React 完成渲染时取消这项昂贵计算。`calculateUserMetrics` 也可能已经通过直接使用 `cacheSignal` 在内部处理了取消。

<Pitfall>

##### 调用不同的记忆化函数会从不同的缓存中读取。 {/*pitfall-different-memoized-functions*/}

要访问同一个缓存，组件必须调用同一个记忆化函数。

```js [[1, 7, "getWeekReport"], [1, 7, "cache(calculateWeekReport)"], [1, 8, "getWeekReport"]]
// Temperature.js
import {cache} from 'react';
import {calculateWeekReport} from './report';

export function Temperature({cityData}) {
  // 🚩 错误：在组件中调用 `cache` 会在每次渲染时创建新的 `getWeekReport`
  const getWeekReport = cache(calculateWeekReport);
  const report = getWeekReport(cityData);
  // ...
}
```

```js [[2, 6, "getWeekReport"], [2, 6, "cache(calculateWeekReport)"], [2, 9, "getWeekReport"]]
// Precipitation.js
import {cache} from 'react';
import {calculateWeekReport} from './report';

// 🚩 错误：`getWeekReport` 只能被 `Precipitation` 组件访问。
const getWeekReport = cache(calculateWeekReport);

export function Precipitation({cityData}) {
  const report = getWeekReport(cityData);
  // ...
}
```

在上面的示例中，<CodeStep step={2}>`Precipitation`</CodeStep> 和 <CodeStep step={1}>`Temperature`</CodeStep> 都分别调用 `cache` 来创建具有各自缓存查找的新记忆化函数。如果两个组件为同一个 `cityData` 渲染，它们会重复执行 `calculateWeekReport`。

此外，`Temperature` 每次组件渲染时都会创建一个 <CodeStep step={1}>新的记忆化函数</CodeStep>，这不允许任何缓存共享。

为了最大化缓存命中并减少工作量，这两个组件应该调用同一个记忆化函数来访问同一个缓存。相反，应该在一个专门的模块中定义该记忆化函数，并让它可以在各组件之间通过 [`import` 导入](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import)。

```js [[3, 5, "export default cache(calculateWeekReport)"]]
// getWeekReport.js
import {cache} from 'react';
import {calculateWeekReport} from './report';

export default cache(calculateWeekReport);
```

```js [[3, 2, "getWeekReport", 0], [3, 5, "getWeekReport"]]
// Temperature.js
import getWeekReport from './getWeekReport';

export default function Temperature({cityData}) {
	const report = getWeekReport(cityData);
  // ...
}
```

```js [[3, 2, "getWeekReport", 0], [3, 5, "getWeekReport"]]
// Precipitation.js
import getWeekReport from './getWeekReport';

export default function Precipitation({cityData}) {
  const report = getWeekReport(cityData);
  // ...
}
```
这里，这两个组件都调用从 `./getWeekReport.js` 导出的 <CodeStep step={3}>同一个记忆化函数</CodeStep>，从而读写同一个缓存。
</Pitfall>

### 共享数据快照 {/*take-and-share-snapshot-of-data*/}

要在组件之间共享数据快照，可以将 `cache` 与类似 `fetch` 的数据获取函数一起调用。当多个组件进行相同的数据请求时，只会发出一次请求，返回的数据会被缓存并在组件之间共享。所有组件在整个服务器渲染过程中引用同一个数据快照。

```js [[1, 4, "city"], [1, 5, "fetchTemperature(city)"], [2, 4, "getTemperature"], [2, 9, "getTemperature"], [1, 9, "city"], [2, 14, "getTemperature"], [1, 14, "city"]]
import {cache} from 'react';
import {fetchTemperature} from './api.js';

const getTemperature = cache(async (city) => {
	return await fetchTemperature(city);
});

async function AnimatedWeatherCard({city}) {
	const temperature = await getTemperature(city);
	// ...
}

async function MinimalWeatherCard({city}) {
	const temperature = await getTemperature(city);
	// ...
}
```

如果 `AnimatedWeatherCard` 和 `MinimalWeatherCard` 都为同一个 <CodeStep step={1}>city</CodeStep> 渲染，它们将从 <CodeStep step={2}>记忆化函数</CodeStep> 中获得相同的数据快照。

如果 `AnimatedWeatherCard` 和 `MinimalWeatherCard` 向 <CodeStep step={2}>`getTemperature`</CodeStep> 提供不同的 <CodeStep step={1}>city</CodeStep> 参数，那么 `fetchTemperature` 会被调用两次，并且每个调用点都会收到不同的数据。

<CodeStep step={1}>city</CodeStep> 充当缓存键。

<Note>

<CodeStep step={3}>异步渲染</CodeStep> 仅支持 Server Components。

```js [[3, 1, "async"], [3, 2, "await"]]
async function AnimatedWeatherCard({city}) {
	const temperature = await getTemperature(city);
	// ...
}
```

要在 Client Components 中渲染使用异步数据的组件，请参阅 [`use()` 文档](/reference/react/use)。

</Note>

### 预加载数据 {/*preload-data*/}

通过缓存一个耗时的数据获取，你可以在渲染组件之前就开始异步工作。

```jsx [[2, 6, "await getUser(id)"], [1, 17, "getUser(id)"]]
const getUser = cache(async (id) => {
  return await db.user.query(id);
});

async function Profile({id}) {
  const user = await getUser(id);
  return (
    <section>
      <img src={user.profilePic} />
      <h2>{user.name}</h2>
    </section>
  );
}

function Page({id}) {
  // ✅ 好：开始获取用户数据
  getUser(id);
  // ... 一些计算工作
  return (
    <>
      <Profile id={id} />
    </>
  );
}
```

渲染 `Page` 时，组件会调用 <CodeStep step={1}>`getUser`</CodeStep>，但请注意它并未使用返回的数据。这个提前的 <CodeStep step={1}>`getUser`</CodeStep> 调用会启动异步数据库查询，而此时 `Page` 正在执行其他计算工作并渲染子组件。

渲染 `Profile` 时，我们再次调用 <CodeStep step={2}>`getUser`</CodeStep>。如果最初的 <CodeStep step={1}>`getUser`</CodeStep> 调用已经返回并缓存了用户数据，那么当 `Profile` <CodeStep step={2}>请求并等待这些数据</CodeStep> 时，它可以直接从缓存中读取，而无需再次进行远程过程调用。如果 <CodeStep step={1}>最初的数据请求</CodeStep> 尚未完成，这种预加载模式可以减少数据获取延迟。

<DeepDive>

#### 缓存异步工作 {/*caching-asynchronous-work*/}

在计算一个 [异步函数](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function) 时，你会得到该工作的一个 [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)。promise 持有这项工作的状态（_pending_、_fulfilled_、_failed_）以及其最终完成结果。

在这个示例中，异步函数 <CodeStep step={1}>`fetchData`</CodeStep> 返回一个正在等待 `fetch` 的 promise。

```js [[1, 1, "fetchData()"], [2, 8, "getData()"], [3, 10, "getData()"]]
async function fetchData() {
  return await fetch(`https://...`);
}

const getData = cache(fetchData);

async function MyComponent() {
  getData();
  // ... 一些计算工作
  await getData();
  // ...
}
```

在第一次调用 <CodeStep step={2}>`getData`</CodeStep> 时，来自 <CodeStep step={1}>`fetchData`</CodeStep> 的 promise 会被缓存。后续查找将返回同一个 promise。

注意，第一次 <CodeStep step={2}>`getData`</CodeStep> 调用没有使用 `await`，而 <CodeStep step={3}>第二次</CodeStep> 使用了。[`await`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/await) 是一个 JavaScript 运算符，它会等待并返回 promise 的最终结果。第一次 <CodeStep step={2}>`getData`</CodeStep> 调用只是启动 `fetch`，以缓存该 promise，供第二次 <CodeStep step={3}>`getData`</CodeStep> 查找。

如果到 <CodeStep step={3}>第二次调用</CodeStep> 时该 promise 仍然处于 _pending_，那么 `await` 会暂停等待结果。优化之处在于：当我们等待 `fetch` 时，React 可以继续执行计算工作，从而减少 <CodeStep step={3}>第二次调用</CodeStep> 的等待时间。

如果该 promise 已经完成，无论是错误还是 _fulfilled_ 结果，`await` 都会立即返回该值。在这两种情况下，性能都会受益。
</DeepDive>

<Pitfall>

##### 在组件外调用记忆化函数不会使用缓存。 {/*pitfall-memoized-call-outside-component*/}

```jsx [[1, 3, "getUser"]]
import {cache} from 'react';

const getUser = cache(async (userId) => {
  return await db.user.query(userId);
});

// 🚩 错误：在组件外调用记忆化函数不会进行记忆化。
getUser('demo-id');

async function DemoProfile() {
  // ✅ 好：`getUser` 会进行记忆化。
  const user = await getUser('demo-id');
  return <Profile user={user} />;
}
```

React 只会在组件中为记忆化函数提供缓存访问权限。当在组件外调用 <CodeStep step={1}>`getUser`</CodeStep> 时，它仍然会执行该函数，但不会读取或更新缓存。

这是因为缓存访问是通过一个 [context](/learn/passing-data-deeply-with-context) 提供的，而该 context 只能从组件中访问。

</Pitfall>

<DeepDive>

#### 什么时候应该使用 `cache`、[`memo`](/reference/react/memo) 或 [`useMemo`](/reference/react/useMemo)？ {/*cache-memo-usememo*/}

上述 API 都提供记忆化功能，但区别在于它们各自 intended to memoize 的内容、谁可以访问缓存，以及缓存何时失效。

#### `useMemo` {/*deep-dive-use-memo*/}

通常，你应该在 Client Component 中使用 [`useMemo`](/reference/react/useMemo) 来跨渲染缓存昂贵的计算。比如，对组件内部的数据变换进行记忆化。

```jsx {expectedErrors: {'react-compiler': [4]}} {4}
'use client';

function WeatherReport({record}) {
  const avgTemp = useMemo(() => calculateAvg(record), record);
  // ...
}

function App() {
  const record = getRecord();
  return (
    <>
      <WeatherReport record={record} />
      <WeatherReport record={record} />
    </>
  );
}
```
在这个示例中，`App` 使用相同的 record 渲染了两个 `WeatherReport`。尽管两个组件做的是相同的工作，它们也不能共享工作。`useMemo` 的缓存只局限于组件本身。

不过，`useMemo` 确实可以保证：如果 `App` 重新渲染且 `record` 对象没有变化，每个组件实例都会跳过工作并使用 `avgTemp` 的记忆化值。`useMemo` 只会缓存给定依赖下 `avgTemp` 的最后一次计算。

#### `cache` {/*deep-dive-cache*/}

通常，你应该在 Server Components 中使用 `cache` 来对可在组件之间共享的工作进行记忆化。

```js [[1, 12, "<WeatherReport city={city} />"], [3, 13, "<WeatherReport city={city} />"], [2, 1, "cache(fetchReport)"]]
const cachedFetchReport = cache(fetchReport);

function WeatherReport({city}) {
  const report = cachedFetchReport(city);
  // ...
}

function App() {
  const city = "Los Angeles";
  return (
    <>
      <WeatherReport city={city} />
      <WeatherReport city={city} />
    </>
  );
}
```
将前面的示例改写为使用 `cache` 后，在这种情况下，<CodeStep step={3}>第二个 `WeatherReport` 实例</CodeStep> 将能够跳过重复工作，并从与 <CodeStep step={1}>第一个 `WeatherReport`</CodeStep> 相同的缓存中读取。与前一个示例相比，另一个不同之处在于，`cache` 也推荐用于 <CodeStep step={2}>对数据请求进行记忆化</CodeStep>，而 `useMemo` 只应用于计算。

目前，`cache` 只能用于 Server Components，并且缓存会在不同服务器请求之间失效。

#### `memo` {/*deep-dive-memo*/}

如果组件的 props 未变化，你应该使用 [`memo`](reference/react/memo) 来防止组件重新渲染。

```js
'use client';

function WeatherReport({record}) {
  const avgTemp = calculateAvg(record);
  // ...
}

const MemoWeatherReport = memo(WeatherReport);

function App() {
  const record = getRecord();
  return (
    <>
      <MemoWeatherReport record={record} />
      <MemoWeatherReport record={record} />
    </>
  );
}
```

在这个示例中，两个 `MemoWeatherReport` 组件在首次渲染时都会调用 `calculateAvg`。不过，如果 `App` 重新渲染，而 `record` 没有变化，那么 props 都没有变化，`MemoWeatherReport` 就不会重新渲染。

与 `useMemo` 相比，`memo` 是基于 props 而不是具体计算来对组件渲染进行记忆化。与 `useMemo` 类似，被记忆化的组件只会缓存上一次 props 值对应的最后一次渲染。一旦 props 改变，缓存就会失效，组件会重新渲染。

</DeepDive>

---

## 故障排查 {/*troubleshooting*/}

### 即使我用相同的参数调用了它，我的记忆化函数还是会运行 {/*memoized-function-still-runs*/}

请参见前面提到的陷阱
* [调用不同的记忆化函数会读取不同的缓存。](#pitfall-different-memoized-functions)
* [在组件外调用记忆化函数不会使用缓存。](#pitfall-memoized-call-outside-component)

如果以上都不适用，可能是 React 检查缓存中是否存在某项内容的方式有问题。

如果你的参数不是 [原始类型](https://developer.mozilla.org/en-US/docs/Glossary/Primitive)（例如对象、函数、数组），请确保你传入的是同一个对象引用。

在调用记忆化函数时，React 会查找输入参数，以查看结果是否已经被缓存。React 会使用参数的浅比较来判断是否命中缓存。

```js
import {cache} from 'react';

const calculateNorm = cache((vector) => {
  // ...
});

function MapMarker(props) {
  // 🚩 错误：props 是一个每次渲染都会变化的对象。
  const length = calculateNorm(props);
  // ...
}

function App() {
  return (
    <>
      <MapMarker x={10} y={10} z={10} />
      <MapMarker x={10} y={10} z={10} />
    </>
  );
}
```

在这种情况下，这两个 `MapMarker` 看起来像是在做同样的工作，并且用相同的 `{x: 10, y: 10, z:10}` 值调用 `calculateNorm`。即使这些对象包含相同的值，它们也不是同一个对象引用，因为每个组件都会创建自己的 `props` 对象。

React 会在输入上调用 [`Object.is`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is) 来验证是否命中缓存。

```js {3,9}
import {cache} from 'react';

const calculateNorm = cache((x, y, z) => {
  // ...
});

function MapMarker(props) {
  // ✅ 好：向记忆化函数传递原始类型
  const length = calculateNorm(props.x, props.y, props.z);
  // ...
}

function App() {
  return (
    <>
      <MapMarker x={10} y={10} z={10} />
      <MapMarker x={10} y={10} z={10} />
    </>
  );
}
```

一种解决方法是将向量维度传递给 `calculateNorm`。这可行是因为这些维度本身就是原始类型。

另一种解决方案可能是将向量对象本身作为 prop 传递给组件。我们需要向两个组件实例传递同一个对象。

```js {3,9,14}
import {cache} from 'react';

const calculateNorm = cache((vector) => {
  // ...
});

function MapMarker(props) {
  // ✅ 好：传递相同的 `vector` 对象
  const length = calculateNorm(props.vector);
  // ...
}

function App() {
  const vector = [10, 10, 10];
  return (
    <>
      <MapMarker vector={vector} />
      <MapMarker vector={vector} />
    </>
  );
}
```
