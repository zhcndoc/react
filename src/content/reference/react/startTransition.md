---
title: startTransition
---

<Intro>

`startTransition` 让你可以在后台渲染 UI 的一部分。

```js
startTransition(action)
```

</Intro>

<InlineToc />

---

## 参考 {/*reference*/}

### `startTransition(action)` {/*starttransition*/}

`startTransition` 函数让你可以将一次状态更新标记为 Transition。

```js {7,9}
import { startTransition } from 'react';

function TabContainer() {
  const [tab, setTab] = useState('about');

  function selectTab(nextTab) {
    startTransition(() => {
      setTab(nextTab);
    });
  }
  // ...
}
```

[查看更多示例。](#usage)

#### 参数 {/*parameters*/}

* `action`: 一个通过调用一个或多个 [`set` 函数](/reference/react/useState#setstate) 来更新某些状态的函数。React 会立即以不带参数的方式调用 `action`，并将 `action` 函数调用期间同步安排的所有状态更新标记为 Transitions。`action` 中 await 的任何异步调用都将包含在 transition 中，但目前需要在 `await` 之后的任何 `set` 函数外再包一层额外的 `startTransition`（参见 [故障排除](/reference/react/useTransition#react-doesnt-treat-my-state-update-after-await-as-a-transition)）。标记为 Transitions 的状态更新将是[非阻塞的](#marking-a-state-update-as-a-non-blocking-transition)，并且[不会显示不需要的加载指示器。](/reference/react/useTransition#preventing-unwanted-loading-indicators)

#### 返回值 {/*returns*/}

`startTransition` 不返回任何内容。

#### 注意事项 {/*caveats*/}

* `startTransition` 不提供跟踪 Transition 是否正在挂起的方法。要在 Transition 进行中显示挂起指示器，你需要改用 [`useTransition`](/reference/react/useTransition)。

* 只有在你能够访问该状态的 `set` 函数时，才能将更新包装为 Transition。如果你想根据某个 prop 或自定义 Hook 的返回值来启动 Transition，请尝试改用 [`useDeferredValue`](/reference/react/useDeferredValue)。

* 传给 `startTransition` 的函数会立即调用，并将其执行期间发生的所有状态更新标记为 Transitions。比如，如果你尝试在 `setTimeout` 中执行状态更新，它们就不会被标记为 Transitions。

* 你必须将任何异步请求后的状态更新再包裹一层 `startTransition`，以将它们标记为 Transitions。这是一个已知限制，我们会在未来修复它（参见 [故障排除](/reference/react/useTransition#react-doesnt-treat-my-state-update-after-await-as-a-transition)）。

* 被标记为 Transition 的状态更新会被其他状态更新中断。例如，如果你在 Transition 中更新一个图表组件，但随后在图表重新渲染进行到一半时开始在输入框中输入内容，React 会在处理输入状态更新后重新开始图表组件的渲染工作。

* Transition 更新不能用于控制文本输入框。

* 如果有多个正在进行的 Transitions，React 目前会将它们批量处理在一起。这是一个限制，未来版本中可能会移除。

---

## 用法 {/*usage*/}

### 将状态更新标记为非阻塞的 Transition {/*marking-a-state-update-as-a-non-blocking-transition*/}

你可以通过将状态更新包裹在 `startTransition` 调用中，把它标记为一个 *Transition*：

```js {7,9}
import { startTransition } from 'react';

function TabContainer() {
  const [tab, setTab] = useState('about');

  function selectTab(nextTab) {
    startTransition(() => {
      setTab(nextTab);
    });
  }
  // ...
}
```

Transitions 让你即使在慢速设备上也能保持界面更新的响应性。

使用 Transition 时，你的 UI 在重新渲染过程中仍能保持响应。例如，如果用户点击了一个标签页，但随后改变主意又点击了另一个标签页，他们无需等待第一次重新渲染完成就可以这样做。

<Note>

`startTransition` 与 [`useTransition`](/reference/react/useTransition) 非常相似，不同之处在于它不提供 `isPending` 标志来跟踪 Transition 是否正在进行中。当 `useTransition` 不可用时，你可以调用 `startTransition`。例如，`startTransition` 可在组件外部使用，比如在数据库中。

[了解 Transition，并在 `useTransition` 页面查看更多示例。](/reference/react/useTransition)

</Note>
