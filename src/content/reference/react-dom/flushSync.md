---
title: flushSync
---

<Pitfall>

使用 `flushSync` 并不常见，而且可能会损害你应用的性能。

</Pitfall>

<Intro>

`flushSync` 允许你强制 React 同步刷新提供的回调中的任何更新。这可以确保 DOM 立即更新。

```js
flushSync(callback)
```

</Intro>

<InlineToc />

---

## 参考 {/*reference*/}

### `flushSync(callback)` {/*flushsync*/}

调用 `flushSync` 以强制 React 同步刷新任何待处理的工作并更新 DOM。

```js
import { flushSync } from 'react-dom';

flushSync(() => {
  setSomething(123);
});
```

大多数时候，可以避免使用 `flushSync`。请将 `flushSync` 作为最后的手段。

[查看更多示例。](#usage)

#### 参数 {/*parameters*/}


* `callback`：一个函数。React 会立即调用这个回调，并同步刷新其中包含的任何更新。它也可能刷新任何待处理的更新、Effects，或 Effects 内部的更新。如果某个更新因这次 `flushSync` 调用而挂起，fallback 可能会被重新显示。

#### 返回值 {/*returns*/}

`flushSync` 返回 `undefined`。

#### 注意事项 {/*caveats*/}

* `flushSync` 会显著影响性能。请谨慎使用。
* `flushSync` 可能会强制尚未完成的 Suspense 边界显示其 `fallback` 状态。
* `flushSync` 可能会在返回前运行待处理的 Effects，并同步应用其中包含的任何更新。
* `flushSync` 在必要时可能会在刷新回调内更新之前，先刷新回调外部的更新。例如，如果有来自点击事件的待处理更新，React 可能会先刷新这些更新，再刷新回调中的更新。

---

## 用法 {/*usage*/}

### 为第三方集成刷新更新 {/*flushing-updates-for-third-party-integrations*/}

当与第三方代码（如浏览器 API 或 UI 库）集成时，可能需要强制 React 刷新更新。使用 `flushSync` 可以强制 React 同步刷新回调中的任何 <CodeStep step={1}>状态更新</CodeStep>：

```js [[1, 2, "setSomething(123)"]]
flushSync(() => {
  setSomething(123);
});
// 到这一行时，DOM 已更新。
```

这可以确保在下一行代码执行时，React 已经更新了 DOM。

**使用 `flushSync` 并不常见，而且频繁使用会显著损害应用性能。** 如果你的应用只使用 React API，且不与第三方库集成，那么通常不需要 `flushSync`。

不过，在与浏览器 API 等第三方代码集成时，它会很有帮助。

某些浏览器 API 期望回调中的结果在回调结束时就同步写入 DOM，以便浏览器可以对渲染后的 DOM 执行某些操作。在大多数情况下，React 会自动为你处理。但在某些情况下，可能需要强制同步更新。

例如，浏览器的 `onbeforeprint` API 允许你在打印对话框打开之前立即更改页面。这对于应用自定义打印样式、让文档在打印时显示得更好很有用。在下面的示例中，你在 `onbeforeprint` 回调中使用 `flushSync`，将 React 状态立即“刷新”到 DOM。然后在打印对话框打开时，`isPrinting` 会显示为 "yes"：

<Sandpack>

```js src/App.js active
import { useState, useEffect } from 'react';
import { flushSync } from 'react-dom';

export default function PrintApp() {
  const [isPrinting, setIsPrinting] = useState(false);

  useEffect(() => {
    function handleBeforePrint() {
      flushSync(() => {
        setIsPrinting(true);
      })
    }

    function handleAfterPrint() {
      setIsPrinting(false);
    }

    window.addEventListener('beforeprint', handleBeforePrint);
    window.addEventListener('afterprint', handleAfterPrint);
    return () => {
      window.removeEventListener('beforeprint', handleBeforePrint);
      window.removeEventListener('afterprint', handleAfterPrint);
    }
  }, []);

  return (
    <>
      <h1>isPrinting: {isPrinting ? 'yes' : 'no'}</h1>
      <button onClick={() => window.print()}>
        打印
      </button>
    </>
  );
}
```

</Sandpack>

如果不使用 `flushSync`，打印对话框会将 `isPrinting` 显示为 "no"。这是因为 React 会异步批处理更新，而打印对话框在状态更新之前就已经显示了。

<Pitfall>

`flushSync` 会显著影响性能，并且可能意外强制尚未完成的 Suspense 边界显示其 fallback 状态。

大多数时候都可以避免使用 `flushSync`，所以请将 `flushSync` 作为最后的手段。

</Pitfall>

---

## 故障排除 {/*troubleshooting*/}

### 我收到一个错误：“flushSync was called from inside a lifecycle method” {/*im-getting-an-error-flushsync-was-called-from-inside-a-lifecycle-method*/}


React 不能在渲染过程中间执行 `flushSync`。如果这样做，它会变成 noop 并发出警告：

<ConsoleBlock level="error">

警告：`flushSync` 是在生命周期方法内部调用的。React 不能在 React 已经处于渲染中时进行刷新。请考虑将此调用移到调度器任务或微任务中。

</ConsoleBlock>

这包括在以下位置调用 `flushSync`：

- 在渲染组件时。
- 在 `useLayoutEffect` 或 `useEffect` Hooks 中。
- 在类组件的生命周期方法中。

例如，在 Effect 中调用 `flushSync` 会变成 noop 并发出警告：

```js
import { useEffect } from 'react';
import { flushSync } from 'react-dom';

function MyComponent() {
  useEffect(() => {
    // 🚩 错误：在 effect 内部调用 flushSync
    flushSync(() => {
      setSomething(newValue);
    });
  }, []);

  return <div>{/* ... */}</div>;
}
```

要修复这个问题，你通常需要把 `flushSync` 调用移到一个事件中：

```js
function handleClick() {
  // ✅ 正确：在事件处理函数中调用 flushSync 是安全的
  flushSync(() => {
    setSomething(newValue);
  });
}
```


如果很难移到事件中，你可以在微任务中延迟执行 `flushSync`：

```js {3,7}
useEffect(() => {
  // ✅ 正确：将 flushSync 延迟到微任务中
  queueMicrotask(() => {
    flushSync(() => {
      setSomething(newValue);
    });
  });
}, []);
```

这将允许当前渲染完成，并调度另一个同步渲染来刷新更新。

<Pitfall>

`flushSync` 会显著影响性能，但这种特定模式对性能的影响更糟。在将 `flushSync` 作为逃生舱口在微任务中调用之前，请先穷尽其他所有选项。

</Pitfall>
