---
title: useSyncExternalStore
---

<Intro>

`useSyncExternalStore` 是一个 React Hook，允许你订阅一个外部 store。

```js
const snapshot = useSyncExternalStore(subscribe, getSnapshot, getServerSnapshot?)
```

</Intro>

<InlineToc />

---

## 参考 {/*reference*/}

### `useSyncExternalStore(subscribe, getSnapshot, getServerSnapshot?)` {/*usesyncexternalstore*/}

在组件顶层调用 `useSyncExternalStore`，以便从外部数据 store 中读取一个值。

```js
import { useSyncExternalStore } from 'react';
import { todosStore } from './todoStore.js';

function TodosApp() {
  const todos = useSyncExternalStore(todosStore.subscribe, todosStore.getSnapshot);
  // ...
}
```

它返回 store 中数据的快照。你需要传入两个函数作为参数：

1. `subscribe` 函数应该订阅该 store，并返回一个用于取消订阅的函数。
2. `getSnapshot` 函数应该从 store 中读取一份数据快照。

[查看更多示例。](#usage)

#### 参数 {/*parameters*/}

* `subscribe`：一个接收单个 `callback` 参数的函数，并将其订阅到 store。当 store 变化时，它应调用所提供的 `callback`，这会让 React 重新调用 `getSnapshot`，并在需要时重新渲染组件。`subscribe` 函数应返回一个用于清理订阅的函数。

* `getSnapshot`：一个返回组件所需 store 数据快照的函数。当 store 未变化时，对 `getSnapshot` 的重复调用必须返回相同的值。如果 store 发生变化且返回值不同（按 [`Object.is`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is) 比较），React 会重新渲染该组件。

* **可选** `getServerSnapshot`：一个返回 store 中数据初始快照的函数。它只会在服务端渲染期间，以及客户端对服务端渲染内容进行 hydration 期间使用。服务端快照必须在客户端和服务端之间保持一致，通常会被序列化并从服务端传递到客户端。如果省略这个参数，在服务端渲染组件时会抛出错误。

#### 返回值 {/*returns*/}

当前 store 的快照，你可以在渲染逻辑中使用它。

#### 注意事项 {/*caveats*/}

* `getSnapshot` 返回的 store 快照必须是不可变的。如果底层 store 拥有可变数据，请在数据变化时返回一个新的不可变快照。否则，返回缓存的上一次快照。

* 如果在重新渲染期间传入了不同的 `subscribe` 函数，React 会使用新传入的 `subscribe` 函数重新订阅该 store。你可以通过在组件外部声明 `subscribe` 来避免这一点。

* 如果在一次 [非阻塞的 Transition 更新](/reference/react/useTransition) 期间 store 被修改，React 会回退为将该更新作为阻塞更新来执行。具体来说，对于每一次 Transition 更新，React 会在将变更应用到 DOM 之前再次调用一次 `getSnapshot`。如果它返回的值与最初调用时不同，React 会从头重新开始更新，这一次会作为阻塞更新执行，以确保屏幕上的每个组件都反映同一个 store 版本。

* 不建议基于 `useSyncExternalStore` 返回的 store 值来在渲染中进行 _suspend_。原因是，外部 store 的变更无法被标记为 [非阻塞的 Transition 更新](/reference/react/useTransition)，因此它们会触发最近的 [`Suspense` fallback](/reference/react/Suspense)，用加载指示器替换屏幕上已经渲染的内容，这通常会带来较差的用户体验。

  例如，以下做法不推荐：

  ```js
  const LazyProductDetailPage = lazy(() => import('./ProductDetailPage.js'));

  function ShoppingApp() {
    const selectedProductId = useSyncExternalStore(...);

    // ❌ 使用依赖于 `selectedProductId` 的 Promise 调用 `use`
    const data = use(fetchItem(selectedProductId))

    // ❌ 根据 `selectedProductId` 有条件地渲染懒加载组件
    return selectedProductId != null ? <LazyProductDetailPage /> : <FeaturedProducts />;
  }
  ```

---

## 用法 {/*usage*/}

### 订阅外部 store {/*subscribing-to-an-external-store*/}

大多数 React 组件只会从它们的 [props,](/learn/passing-props-to-a-component) [state,](/reference/react/useState) 和 [context.](/reference/react/useContext) 中读取数据。然而，有时组件需要从 React 之外的某个会随时间变化的 store 中读取一些数据。这包括：

* 保存 React 之外状态的第三方状态管理库。
* 暴露可变值以及用于订阅其变化事件的浏览器 API。

在组件顶层调用 `useSyncExternalStore`，以便从外部数据 store 中读取一个值。

```js [[1, 5, "todosStore.subscribe"], [2, 5, "todosStore.getSnapshot"], [3, 5, "todos", 0]]
import { useSyncExternalStore } from 'react';
import { todosStore } from './todoStore.js';

function TodosApp() {
  const todos = useSyncExternalStore(todosStore.subscribe, todosStore.getSnapshot);
  // ...
}
```

它返回 store 中数据的 <CodeStep step={3}>快照</CodeStep>。你需要传入两个函数作为参数：

1. <CodeStep step={1}>`subscribe` 函数</CodeStep> 应该订阅该 store，并返回一个用于取消订阅的函数。
2. <CodeStep step={2}>`getSnapshot` 函数</CodeStep> 应该从 store 中读取一份数据快照。

React 会使用这些函数保持组件订阅该 store，并在发生变化时重新渲染。

例如，在下面的沙盒中，`todosStore` 被实现为一个位于 React 之外、用于存储数据的外部 store。`TodosApp` 组件通过 `useSyncExternalStore` Hook 连接到该外部 store。

<Sandpack>

```js
import { useSyncExternalStore } from 'react';
import { todosStore } from './todoStore.js';

export default function TodosApp() {
  const todos = useSyncExternalStore(todosStore.subscribe, todosStore.getSnapshot);
  return (
    <>
      <button onClick={() => todosStore.addTodo()}>添加待办</button>
      <hr />
      <ul>
        {todos.map(todo => (
          <li key={todo.id}>{todo.text}</li>
        ))}
      </ul>
    </>
  );
}
```

```js src/todoStore.js
// 这是一个第三方 store 的示例
// 你可能需要将它与你的 React 代码集成。

// 如果你的应用完全由 React 构建，
// 我们建议改用 React state。

let nextId = 0;
let todos = [{ id: nextId++, text: 'Todo #1' }];
let listeners = [];

export const todosStore = {
  addTodo() {
    todos = [...todos, { id: nextId++, text: 'Todo #' + nextId }]
    emitChange();
  },
  subscribe(listener) {
    listeners = [...listeners, listener];
    return () => {
      listeners = listeners.filter(l => l !== listener);
    };
  },
  getSnapshot() {
    return todos;
  }
};

function emitChange() {
  for (let listener of listeners) {
    listener();
  }
}
```

</Sandpack>

<Note>

在可能的情况下，我们建议改用内置的 React state，例如 [`useState`](/reference/react/useState) 和 [`useReducer`](/reference/react/useReducer)。`useSyncExternalStore` API 主要适用于需要与已有的非 React 代码集成的场景。

</Note>

---

### 订阅浏览器 API {/*subscribing-to-a-browser-api*/}

添加 `useSyncExternalStore` 的另一个原因是：当你想订阅浏览器暴露的某个会随时间变化的值时。例如，假设你希望组件显示网络连接是否处于活动状态。浏览器会通过名为 [`navigator.onLine`.](https://developer.mozilla.org/en-US/docs/Web/API/Navigator/onLine) 的属性暴露这一信息。

这个值可以在 React 不知情的情况下变化，所以你应该使用 `useSyncExternalStore` 来读取它。

```js
import { useSyncExternalStore } from 'react';

function ChatIndicator() {
  const isOnline = useSyncExternalStore(subscribe, getSnapshot);
  // ...
}
```

要实现 `getSnapshot` 函数，请从浏览器 API 中读取当前值：

```js
function getSnapshot() {
  return navigator.onLine;
}
```

接下来，你需要实现 `subscribe` 函数。例如，当 `navigator.onLine` 变化时，浏览器会在 `window` 对象上触发 [`online`](https://developer.mozilla.org/en-US/docs/Web/API/Window/online_event) 和 [`offline`](https://developer.mozilla.org/en-US/docs/Web/API/Window/offline_event) 事件。你需要将 `callback` 参数订阅到相应事件上，然后返回一个用于清理这些订阅的函数：

```js
function subscribe(callback) {
  window.addEventListener('online', callback);
  window.addEventListener('offline', callback);
  return () => {
    window.removeEventListener('online', callback);
    window.removeEventListener('offline', callback);
  };
}
```

现在 React 知道如何从外部的 `navigator.onLine` API 中读取值，以及如何订阅它的变化。断开设备的网络连接，看看组件是否会响应式重新渲染：

<Sandpack>

```js
import { useSyncExternalStore } from 'react';

export default function ChatIndicator() {
  const isOnline = useSyncExternalStore(subscribe, getSnapshot);
  return <h1>{isOnline ? '✅ 在线' : '❌ 已断开连接'}</h1>;
}

function getSnapshot() {
  return navigator.onLine;
}

function subscribe(callback) {
  window.addEventListener('online', callback);
  window.addEventListener('offline', callback);
  return () => {
    window.removeEventListener('online', callback);
    window.removeEventListener('offline', callback);
  };
}
```

</Sandpack>

---

### 将逻辑抽取到自定义 Hook 中 {/*extracting-the-logic-to-a-custom-hook*/}

通常你不会直接在组件中编写 `useSyncExternalStore`。相反，你通常会从自己编写的自定义 Hook 中调用它。这让你可以在不同组件中使用同一个外部 store。

例如，这个自定义 `useOnlineStatus` Hook 会追踪网络是否在线：

```js {3,6}
import { useSyncExternalStore } from 'react';

export function useOnlineStatus() {
  const isOnline = useSyncExternalStore(subscribe, getSnapshot);
  return isOnline;
}

function getSnapshot() {
  // ...
}

function subscribe(callback) {
  // ...
}
```

现在不同的组件可以调用 `useOnlineStatus`，而无需重复底层实现：

<Sandpack>

```js
import { useOnlineStatus } from './useOnlineStatus.js';

function StatusBar() {
  const isOnline = useOnlineStatus();
  return <h1>{isOnline ? '✅ 在线' : '❌ 已断开连接'}</h1>;
}

function SaveButton() {
  const isOnline = useOnlineStatus();

  function handleSaveClick() {
    console.log('✅ 进度已保存');
  }

  return (
    <button disabled={!isOnline} onClick={handleSaveClick}>
      {isOnline ? '保存进度' : '正在重新连接...'}
    </button>
  );
}

export default function App() {
  return (
    <>
      <SaveButton />
      <StatusBar />
    </>
  );
}
```

```js src/useOnlineStatus.js
import { useSyncExternalStore } from 'react';

export function useOnlineStatus() {
  const isOnline = useSyncExternalStore(subscribe, getSnapshot);
  return isOnline;
}

function getSnapshot() {
  return navigator.onLine;
}

function subscribe(callback) {
  window.addEventListener('online', callback);
  window.addEventListener('offline', callback);
  return () => {
    window.removeEventListener('online', callback);
    window.removeEventListener('offline', callback);
  };
}
```

</Sandpack>

---

### 添加对服务端渲染的支持 {/*adding-support-for-server-rendering*/}

如果你的 React 应用使用了[服务端渲染，](/reference/react-dom/server)你的 React 组件也会在浏览器环境之外运行，以生成初始 HTML。在连接外部 store 时，这会带来一些挑战：

- 如果你连接的是仅浏览器可用的 API，它将无法工作，因为它在服务器上不存在。
- 如果你连接的是第三方数据 store，你需要确保服务端和客户端上的数据一致。

要解决这些问题，请将一个 `getServerSnapshot` 函数作为 `useSyncExternalStore` 的第三个参数传入：

```js {4,12-14}
import { useSyncExternalStore } from 'react';

export function useOnlineStatus() {
  const isOnline = useSyncExternalStore(subscribe, getSnapshot, getServerSnapshot);
  return isOnline;
}

function getSnapshot() {
  return navigator.onLine;
}

function getServerSnapshot() {
  return true; // 对于服务端生成的 HTML，始终显示“在线”
}

function subscribe(callback) {
  // ...
}
```

`getServerSnapshot` 函数类似于 `getSnapshot`，但它只在两种情况下运行：

- 它在服务器上生成 HTML 时运行。
- 它在客户端进行 [hydration](/reference/react-dom/client/hydrateRoot) 时运行，也就是 React 接收服务端 HTML 并使其具备交互能力时运行。

这让你可以提供一个初始快照值，该值会在应用变得可交互之前使用。如果服务端渲染没有有意义的初始值，请省略此参数以[强制在客户端渲染。](/reference/react/Suspense#providing-a-fallback-for-server-errors-and-client-only-content)

<Note>

请确保 `getServerSnapshot` 在初始客户端渲染时返回的数据与它在服务器上返回的数据完全一致。例如，如果 `getServerSnapshot` 在服务器上返回了一些预填充的 store 内容，你需要将这些内容传输到客户端。一种做法是在服务端渲染期间输出一个 `<script>` 标签，用来设置诸如 `window.MY_STORE_DATA` 之类的全局变量，然后在客户端的 `getServerSnapshot` 中从这个全局变量读取。你的外部 store 应该提供如何执行此操作的说明。

</Note>

---

## 故障排除 {/*troubleshooting*/}

### 我遇到一个错误："`getSnapshot` 的结果应该被缓存" {/*im-getting-an-error-the-result-of-getsnapshot-should-be-cached*/}

这个错误意味着你的 `getSnapshot` 函数每次被调用时都会返回一个新对象，例如：

```js {2-5}
function getSnapshot() {
  // 🔴 不要总是从 getSnapshot 返回不同的对象
  return {
    todos: myStore.todos
  };
}
```

如果 `getSnapshot` 的返回值与上一次不同，React 会重新渲染组件。这就是为什么如果你总是返回不同的值，就会进入无限循环并出现这个错误。

你的 `getSnapshot` 对象只有在某些内容确实发生变化时才应该返回不同的对象。如果你的 store 包含不可变数据，你可以直接返回这些数据：

```js {2-3}
function getSnapshot() {
  // ✅ 你可以返回不可变数据
  return myStore.todos;
}
```

如果你的 store 数据是可变的，你的 `getSnapshot` 函数应该返回它的一个不可变快照。这意味着它 *确实* 需要创建新对象，但不应该在每次调用时都这样做。相反，它应该保存上一次计算出的快照，并在 store 中的数据没有改变时返回与上一次相同的快照。如何判断可变数据是否发生变化，取决于你的可变 store。

---

### 我的 `subscribe` 函数在每次重新渲染后都会被调用 {/*my-subscribe-function-gets-called-after-every-re-render*/}

这个 `subscribe` 函数定义在组件 *内部*，因此它在每次重新渲染时都不同：

```js {2-5}
function ChatIndicator() {
  // 🚩 总是不同的函数，因此 React 会在每次重新渲染时重新订阅
  function subscribe() {
    // ...
  }

  const isOnline = useSyncExternalStore(subscribe, getSnapshot);

  // ...
}
```

如果你在重新渲染之间传入不同的 `subscribe` 函数，React 就会重新订阅你的 store。如果这导致性能问题，而你想避免重新订阅，可以把 `subscribe` 函数移到外部：

```js {1-4}
// ✅ 始终是同一个函数，因此 React 不需要重新订阅
function subscribe() {
  // ...
}

function ChatIndicator() {
  const isOnline = useSyncExternalStore(subscribe, getSnapshot);
  // ...
}
```

或者，把 `subscribe` 包装进 [`useCallback`](/reference/react/useCallback) 中，这样只有在某个参数改变时才会重新订阅：

```js {2-5}
function ChatIndicator({ userId }) {
  // ✅ 只要 userId 不变，就是同一个函数
  const subscribe = useCallback(() => {
    // ...
  }, [userId]);

  const isOnline = useSyncExternalStore(subscribe, getSnapshot);

  // ...
}
```
