---
title: act
---

<Intro>

`act` 是一个测试辅助工具，用于在进行断言之前应用尚未处理的 React 更新。

```js
await act(async actFn)
```

</Intro>

在为断言准备组件时，请将渲染它并执行更新的代码包裹在一个 `await act()` 调用中。这会使你的测试行为更接近 React 在浏览器中的工作方式。

<Note>
你可能会觉得直接使用 `act()` 有点太啰嗦了。为了避免一些样板代码，你可以使用像 [React Testing Library](https://testing-library.com/docs/react-testing-library/intro) 这样的库，它的辅助工具已经用 `act()` 包裹好了。
</Note>


<InlineToc />

---

## 参考 {/*reference*/}

### `await act(async actFn)` {/*await-act-async-actfn*/}

在编写 UI 测试时，渲染、用户事件或数据获取之类的任务可以被视为与用户界面的“交互单元”。React 提供了一个名为 `act()` 的辅助工具，它可以确保与这些“单元”相关的所有更新都已在你进行任何断言之前被处理并应用到 DOM 上。

`act` 这个名字来自 [Arrange-Act-Assert](https://wiki.c2.com/?ArrangeActAssert) 模式。

```js {2,4}
it ('renders with button disabled', async () => {
  await act(async () => {
    root.render(<TestComponent />)
  });
  expect(container.querySelector('button')).toBeDisabled();
});
```

<Note>

我们建议将 `act` 与 `await` 和 `async` 函数一起使用。虽然同步版本在许多情况下都能工作，但并非所有情况都适用，而且由于 React 内部调度更新的方式，很难预测何时可以使用同步版本。

我们将来会弃用并移除同步版本。

</Note>

#### 参数 {/*parameters*/}

* `async actFn`: 一个包装被测组件渲染或交互的异步函数。`actFn` 内触发的任何更新都会被加入内部的 act 队列，然后一起刷新，以处理并应用到 DOM 的任何变更。由于它是异步的，React 还会运行任何跨越异步边界的代码，并刷新任何已调度的更新。

#### 返回值 {/*returns*/}

`act` 不返回任何内容。

## 用法 {/*usage*/}

在测试组件时，你可以使用 `act` 来对其输出进行断言。

例如，假设我们有这个 `Counter` 组件，下面的用法示例展示了如何测试它：

```js
function Counter() {
  const [count, setCount] = useState(0);
  const handleClick = () => {
    setCount(prev => prev + 1);
  }

  useEffect(() => {
    document.title = `You clicked ${count} times`;
  }, [count]);

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={handleClick}>
        Click me
      </button>
    </div>
  )
}
```

### 在测试中渲染组件 {/*rendering-components-in-tests*/}

要测试组件的渲染输出，请将渲染包裹在 `act()` 中：

```js  {10,12}
import {act} from 'react';
import ReactDOMClient from 'react-dom/client';
import Counter from './Counter';

it('can render and update a counter', async () => {
  container = document.createElement('div');
  document.body.appendChild(container);

  // ✅ 在 act() 中渲染组件。
  await act(() => {
    ReactDOMClient.createRoot(container).render(<Counter />);
  });

  const button = container.querySelector('button');
  const label = container.querySelector('p');
  expect(label.textContent).toBe('You clicked 0 times');
  expect(document.title).toBe('You clicked 0 times');
});
```

这里，我们创建一个容器，将其附加到文档中，并在 `act()` 内渲染 `Counter` 组件。这可以确保组件已完成渲染，并且其副作用已在进行断言之前被应用。

使用 `act` 可以确保在我们进行断言之前，所有更新都已被应用。

### 在测试中分发事件 {/*dispatching-events-in-tests*/}

要测试事件，请将事件分发包裹在 `act()` 中：

```js {14,16}
import {act} from 'react';
import ReactDOMClient from 'react-dom/client';
import Counter from './Counter';

it.only('can render and update a counter', async () => {
  const container = document.createElement('div');
  document.body.appendChild(container);

  await act( async () => {
    ReactDOMClient.createRoot(container).render(<Counter />);
  });

  // ✅ 在 act() 中分发事件。
  await act(async () => {
    button.dispatchEvent(new MouseEvent('click', { bubbles: true }));
  });

  const button = container.querySelector('button');
  const label = container.querySelector('p');
  expect(label.textContent).toBe('You clicked 1 times');
  expect(document.title).toBe('You clicked 1 times');
});
```

这里，我们使用 `act` 渲染组件，然后在另一个 `act()` 中分发事件。这可以确保来自该事件的所有更新都已在进行断言之前被应用。

<Pitfall>

不要忘记，只有当 DOM 容器被添加到文档中时，分发 DOM 事件才有效。你可以使用像 [React Testing Library](https://testing-library.com/docs/react-testing-library/intro) 这样的库来减少样板代码。

</Pitfall>

## 故障排查 {/*troubleshooting*/}

### 我收到一个错误：“当前测试环境未配置为支持 act(...)” {/*error-the-current-testing-environment-is-not-configured-to-support-act*/}

使用 `act` 需要在你的测试环境中设置 `global.IS_REACT_ACT_ENVIRONMENT=true`。这是为了确保 `act` 只在正确的环境中使用。

如果你没有设置这个全局变量，你会看到如下错误：

<ConsoleBlock level="error">

警告：当前测试环境未配置为支持 act(...)

</ConsoleBlock>

要修复此问题，请在 React 测试的全局初始化文件中添加以下内容：

```js
global.IS_REACT_ACT_ENVIRONMENT=true
```

<Note>

在像 [React Testing Library](https://testing-library.com/docs/react-testing-library/intro) 这样的测试框架中，`IS_REACT_ACT_ENVIRONMENT` 已经为你设置好了。

</Note>
