---
title: useDebugValue
---

<Intro>

`useDebugValue` 是一个 React Hook，它让你可以在 [React DevTools.](/learn/react-developer-tools) 中为自定义 Hook 添加一个标签

```js
useDebugValue(value, format?)
```

</Intro>

<InlineToc />

---

## Reference {/*reference*/}

### `useDebugValue(value, format?)` {/*usedebugvalue*/}

在你的 [自定义 Hook](/learn/reusing-logic-with-custom-hooks) 的顶层调用 `useDebugValue`，以显示一个可读的调试值：

```js
import { useDebugValue } from 'react';

function useOnlineStatus() {
  // ...
  useDebugValue(isOnline ? '在线' : '离线');
  // ...
}
```

[查看更多示例。](#usage)

#### Parameters {/*parameters*/}

* `value`：你想在 React DevTools 中显示的值。它可以是任意类型。
* **可选** `format`：一个格式化函数。当组件被检查时，React DevTools 会以 `value` 作为参数调用该格式化函数，然后显示返回的格式化值（该值也可以是任意类型）。如果你不指定格式化函数，则会直接显示原始的 `value`。

#### Returns {/*returns*/}

`useDebugValue` 不返回任何内容。

## Usage {/*usage*/}

### 为自定义 Hook 添加标签 {/*adding-a-label-to-a-custom-hook*/}

在你的 [自定义 Hook](/learn/reusing-logic-with-custom-hooks) 的顶层调用 `useDebugValue`，为 [React DevTools.](/learn/react-developer-tools) 显示一个可读的 <CodeStep step={1}>调试值</CodeStep>

```js [[1, 5, "isOnline ? 'Online' : 'Offline'"]]
import { useDebugValue } from 'react';

function useOnlineStatus() {
  // ...
  useDebugValue(isOnline ? 'Online' : 'Offline');
  // ...
}
```

当你检查这些组件时，这会给调用 `useOnlineStatus` 的组件显示一个类似 `OnlineStatus: "Online"` 的标签：

![A screenshot of React DevTools showing the debug value](/images/docs/react-devtools-usedebugvalue.png)

如果没有 `useDebugValue` 调用，只会显示底层数据（在这个例子中是 `true`）。

<Sandpack>

```js
import { useOnlineStatus } from './useOnlineStatus.js';

function StatusBar() {
  const isOnline = useOnlineStatus();
  return <h1>{isOnline ? '✅ 在线' : '❌ 断开连接'}</h1>;
}

export default function App() {
  return <StatusBar />;
}
```

```js src/useOnlineStatus.js active
import { useSyncExternalStore, useDebugValue } from 'react';

export function useOnlineStatus() {
  const isOnline = useSyncExternalStore(subscribe, () => navigator.onLine, () => true);
  useDebugValue(isOnline ? 'Online' : 'Offline');
  return isOnline;
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

<Note>

不要给每个自定义 Hook 都添加调试值。它对共享库中的自定义 Hook 最有价值，尤其是那些具有复杂内部数据结构、难以检查的 Hook。

</Note>

---

### 延迟格式化调试值 {/*deferring-formatting-of-a-debug-value*/}

你也可以将格式化函数作为 `useDebugValue` 的第二个参数传入：

```js [[1, 1, "date", 18], [2, 1, "date.toDateString()"]]
useDebugValue(date, date => date.toDateString());
```

你的格式化函数将接收 <CodeStep step={1}>调试值</CodeStep> 作为参数，并且应该返回一个 <CodeStep step={2}>格式化后的显示值</CodeStep>。当组件被检查时，React DevTools 会调用这个函数并显示其结果。

这样你就可以避免运行可能代价高昂的格式化逻辑，除非组件真的被检查。例如，如果 `date` 是一个 Date 值，这样可以避免在每次渲染时都对它调用 `toDateString()`。
