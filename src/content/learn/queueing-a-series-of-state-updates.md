---
title: 队列化一系列状态更新
---

<Intro>

设置一个状态变量会触发下一次渲染。但有时你可能想在触发下一次渲染之前对这个值执行多个操作。要做到这一点，理解 React 如何批量处理状态更新会很有帮助。

</Intro>

<YouWillLearn>

* 什么是“批处理”，以及 React 如何使用它来处理多个状态更新
* 如何连续对同一个状态变量应用多个更新

</YouWillLearn>

## React 会批量处理状态更新 {/*react-batches-state-updates*/}

你可能会以为，点击 "+3" 按钮会让计数器增加三次，因为它连续调用了三次 `setNumber(number + 1)`：

<Sandpack>

```js
import { useState } from 'react';

export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button onClick={() => {
        setNumber(number + 1);
        setNumber(number + 1);
        setNumber(number + 1);
      }}>+3</button>
    </>
  )
}
```

```css
button { display: inline-block; margin: 10px; font-size: 20px; }
h1 { display: inline-block; margin: 10px; width: 30px; text-align: center; }
```

</Sandpack>

然而，正如你在上一节中可能还记得的那样，[每次渲染时的状态值都是固定的](/learn/state-as-a-snapshot#rendering-takes-a-snapshot-in-time)，所以第一次渲染中的事件处理函数里 `number` 的值始终是 `0`，不管你调用多少次 `setNumber(1)`：

```js
setNumber(0 + 1);
setNumber(0 + 1);
setNumber(0 + 1);
```

但这里还有另一个因素在起作用。**React 会等到事件处理函数中的 *所有* 代码都执行完后，才处理你的状态更新。** 这就是为什么只有在所有这些 `setNumber()` 调用之后，才会发生重新渲染。

这可能会让你联想到餐厅里的服务员记订单。服务员不会因为你点了第一道菜就立刻冲进厨房！相反，他们会等你点完，允许你修改订单，甚至还会帮同桌的其他人一起下单。

<Illustration src="/images/docs/illustrations/i_react-batching.png"  alt="一位优雅的顾客在餐厅中多次下单，React 扮演服务员的角色。她多次调用 setState() 后，服务员把她最后一次要求的内容记为最终订单。" />

这使你能够更新多个状态变量——即使来自多个组件——而不会触发过多的[重新渲染。](/learn/render-and-commit#re-renders-when-state-updates) 但这也意味着，UI 不会在你的事件处理函数以及其中的任何代码执行完毕之前更新。这种行为，也被称为**批处理**，能让你的 React 应用运行得快得多。它还避免了处理令人困惑的“半完成”渲染——也就是只有部分变量被更新的情况。

**React 不会跨越多个有意图的事件（如点击）进行批处理**——每次点击都会单独处理。请放心，React 只会在通常安全的情况下进行批处理。这确保了例如第一次按钮点击如果禁用了表单，那么第二次点击就不会再次提交它。

## 在下一次渲染前多次更新同一个状态 {/*updating-the-same-state-multiple-times-before-the-next-render*/}

这不是一种常见用法，但如果你想在下一次渲染前多次更新同一个状态变量，与其传入*下一个状态值*（例如 `setNumber(number + 1)`），你可以传入一个*函数*，它会基于队列中前一个值计算下一个状态，例如 `setNumber(n => n + 1)`。这是一种告诉 React“对这个状态值做点什么”而不是直接替换它的方法。

现在试着让计数器增加：

<Sandpack>

```js
import { useState } from 'react';

export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button onClick={() => {
        setNumber(n => n + 1);
        setNumber(n => n + 1);
        setNumber(n => n + 1);
      }}>+3</button>
    </>
  )
}
```

```css
button { display: inline-block; margin: 10px; font-size: 20px; }
h1 { display: inline-block; margin: 10px; width: 30px; text-align: center; }
```

</Sandpack>

这里，`n => n + 1` 被称为**更新函数**。当你把它传给状态设置函数时：

1. React 会将这个函数排入队列，等到事件处理函数中的其他代码都执行完后再处理。
2. 在下一次渲染时，React 会遍历这个队列，并给你最终更新后的状态。

```js
setNumber(n => n + 1);
setNumber(n => n + 1);
setNumber(n => n + 1);
```

下面是 React 在执行事件处理函数时如何处理这些代码行的：

1. `setNumber(n => n + 1)`：`n => n + 1` 是一个函数。React 将它加入队列。
1. `setNumber(n => n + 1)`：`n => n + 1` 是一个函数。React 将它加入队列。
1. `setNumber(n => n + 1)`：`n => n + 1` 是一个函数。React 将它加入队列。

当你在下一次渲染时调用 `useState`，React 会遍历队列。之前的 `number` 状态是 `0`，所以 React 会把它作为第一个更新函数的 `n` 参数传入。然后 React 取出上一个更新函数的返回值，把它作为下一个更新函数的 `n`，依此类推：

|  队列中的更新 | `n` | 返回值 |
|--------------|---------|-----|
| `n => n + 1` | `0` | `0 + 1 = 1` |
| `n => n + 1` | `1` | `1 + 1 = 2` |
| `n => n + 1` | `2` | `2 + 1 = 3` |

React 将 `3` 作为最终结果存储起来，并从 `useState` 返回它。

这就是为什么在上面的示例中点击 "+3" 会正确地让值增加 3。
### 如果在替换状态后再更新它，会发生什么 {/*what-happens-if-you-update-state-after-replacing-it*/}

看看这个事件处理函数？你觉得下一次渲染时 `number` 会是多少？

```js
<button onClick={() => {
  setNumber(number + 5);
  setNumber(n => n + 1);
}}>
```

<Sandpack>

```js
import { useState } from 'react';

export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button onClick={() => {
        setNumber(number + 5);
        setNumber(n => n + 1);
      }}>Increase the number</button>
    </>
  )
}
```

```css
button { display: inline-block; margin: 10px; font-size: 20px; }
h1 { display: inline-block; margin: 10px; width: 30px; text-align: center; }
```

</Sandpack>

这个事件处理函数告诉 React 要做的是：

1. `setNumber(number + 5)`：`number` 是 `0`，所以是 `setNumber(0 + 5)`。React 会把*"替换为 `5`"*加入队列。
2. `setNumber(n => n + 1)`：`n => n + 1` 是一个更新函数。React 会把*这个函数*加入队列。

在下一次渲染期间，React 会遍历状态队列：

|   队列中的更新       | `n` | 返回值 |
|--------------|---------|-----|
| "替换为 `5`" | `0`（未使用） | `5` |
| `n => n + 1` | `5` | `5 + 1 = 6` |

React 将 `6` 作为最终结果存储起来，并从 `useState` 返回它。

<Note>

你可能已经注意到，`setState(5)` 实际上就像 `setState(n => 5)` 一样工作，只是 `n` 没有被使用！

</Note>

### 如果在更新后再替换状态，会发生什么 {/*what-happens-if-you-replace-state-after-updating-it*/}

我们再试一个例子。你觉得下一次渲染时 `number` 会是多少？

```js
<button onClick={() => {
  setNumber(number + 5);
  setNumber(n => n + 1);
  setNumber(42);
}}>
```

<Sandpack>

```js
import { useState } from 'react';

export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button onClick={() => {
        setNumber(number + 5);
        setNumber(n => n + 1);
        setNumber(42);
      }}>Increase the number</button>
    </>
  )
}
```

```css
button { display: inline-block; margin: 10px; font-size: 20px; }
h1 { display: inline-block; margin: 10px; width: 30px; text-align: center; }
```

</Sandpack>

下面是 React 在执行这个事件处理函数时如何处理这些代码行的：

1. `setNumber(number + 5)`：`number` 是 `0`，所以是 `setNumber(0 + 5)`。React 会把*"替换为 `5`"*加入队列。
2. `setNumber(n => n + 1)`：`n => n + 1` 是一个更新函数。React 会把*这个函数*加入队列。
3. `setNumber(42)`：React 会把*"替换为 `42`"*加入队列。

在下一次渲染期间，React 会遍历状态队列：

|   队列中的更新       | `n` | 返回值 |
|--------------|---------|-----|
| "替换为 `5`" | `0`（未使用） | `5` |
| `n => n + 1` | `5` | `5 + 1 = 6` |
| "替换为 `42`" | `6`（未使用） | `42` |

然后 React 将 `42` 作为最终结果存储起来，并从 `useState` 返回它。

总结一下，你可以这样理解传给 `setNumber` 状态设置函数的内容：

* **更新函数**（例如 `n => n + 1`）会被加入队列。
* **任何其他值**（例如数字 `5`）都会把*"替换为 `5`"*加入队列，并忽略队列里已有的内容。

事件处理函数执行完毕后，React 会触发一次重新渲染。在重新渲染期间，React 会处理这个队列。更新函数会在渲染期间运行，所以**更新函数必须是[纯函数](/learn/keeping-components-pure)**，并且只需*返回*结果。不要试图在其中设置状态或执行其他副作用。在严格模式下，React 会把每个更新函数执行两次（但会丢弃第二次结果），以帮助你发现错误。

### 命名约定 {/*naming-conventions*/}

通常会用对应状态变量前几个字母来命名更新函数的参数：

```js
setEnabled(e => !e);
setLastName(ln => ln.reverse());
setFriendCount(fc => fc * 2);
```

如果你更喜欢更详细的代码，另一种常见约定是重复完整的状态变量名，例如 `setEnabled(enabled => !enabled)`，或者使用前缀，例如 `setEnabled(prevEnabled => !prevEnabled)`。

<Recap>

* 设置状态不会改变现有渲染中的变量，但会请求一次新的渲染。
* React 会在事件处理函数执行完毕后处理状态更新。这叫做批处理。
* 如果要在一个事件中多次更新某些状态，可以使用 `setNumber(n => n + 1)` 这样的更新函数。

</Recap>



<Challenges>

#### 修复请求计数器 {/*fix-a-request-counter*/}

你正在开发一个艺术品交易市场应用，用户可以同时为一件艺术品提交多个订单。每次用户按下 "Buy" 按钮时，“Pending” 计数器都应该加一。三秒后，“Pending” 计数器应该减少，“Completed” 计数器应该增加。

然而，“Pending” 计数器的行为并不符合预期。当你按下 "Buy" 时，它会变成 `-1`（这本不该发生！）。而且如果你快速点击两次，这两个计数器的行为似乎都不可预测。

为什么会这样？请修复这两个计数器。

<Sandpack>

```js
import { useState } from 'react';

export default function RequestTracker() {
  const [pending, setPending] = useState(0);
  const [completed, setCompleted] = useState(0);

  async function handleClick() {
    setPending(pending + 1);
    await delay(3000);
    setPending(pending - 1);
    setCompleted(completed + 1);
  }

  return (
    <>
      <h3>
        Pending: {pending}
      </h3>
      <h3>
        Completed: {completed}
      </h3>
      <button onClick={handleClick}>
        Buy
      </button>
    </>
  );
}

function delay(ms) {
  return new Promise(resolve => {
    setTimeout(resolve, ms);
  });
}
```

</Sandpack>

<Solution>

在 `handleClick` 事件处理函数中，`pending` 和 `completed` 的值对应于点击事件发生时它们的值。对于第一次渲染，`pending` 是 `0`，所以 `setPending(pending - 1)` 变成了 `setPending(-1)`，这显然是错的。由于你想要的是让计数器*增加*或*减少*，而不是把它们设置为点击时确定的具体值，所以你可以改为传入更新函数：

<Sandpack>

```js
import { useState } from 'react';

export default function RequestTracker() {
  const [pending, setPending] = useState(0);
  const [completed, setCompleted] = useState(0);

  async function handleClick() {
    setPending(p => p + 1);
    await delay(3000);
    setPending(p => p - 1);
    setCompleted(c => c + 1);
  }

  return (
    <>
      <h3>
        Pending: {pending}
      </h3>
      <h3>
        Completed: {completed}
      </h3>
      <button onClick={handleClick}>
        Buy
      </button>
    </>
  );
}

function delay(ms) {
  return new Promise(resolve => {
    setTimeout(resolve, ms);
  });
}
```

</Sandpack>

这样可以确保当你增加或减少某个计数器时，依据的是它的*最新*状态，而不是点击发生时的状态。

</Solution>

#### 自己实现状态队列 {/*implement-the-state-queue-yourself*/}

在这个挑战中，你将从零开始重写 React 的一小部分！这听起来没那么难。

滚动查看 sandbox 预览。注意它展示了**四个测试用例**。它们对应于你之前在本页看到的示例。你的任务是实现 `getFinalState` 函数，使它为这些情况返回正确结果。如果你实现正确，四个测试都应该通过。

你会收到两个参数：`baseState` 是初始状态（例如 `0`），`queue` 是一个数组，按添加顺序包含数字（例如 `5`）和更新函数（例如 `n => n + 1`）的混合内容。

你的任务是返回最终状态，就像本页中的表格展示的那样！

<Hint>

如果你卡住了，可以从这个代码结构开始：

```js
export function getFinalState(baseState, queue) {
  let finalState = baseState;

  for (let update of queue) {
    if (typeof update === 'function') {
      // TODO: 应用更新函数
    } else {
      // TODO: 替换状态
    }
  }

  return finalState;
}
```

把缺少的几行补上！

</Hint>

<Sandpack>

```js src/processQueue.js active
export function getFinalState(baseState, queue) {
  let finalState = baseState;

  // TODO: 对队列做点什么...

  return finalState;
}
```

```js src/App.js
import { getFinalState } from './processQueue.js';

function increment(n) {
  return n + 1;
}
increment.toString = () => 'n => n+1';

export default function App() {
  return (
    <>
      <TestCase
        baseState={0}
        queue={[1, 1, 1]}
        expected={1}
      />
      <hr />
      <TestCase
        baseState={0}
        queue={[
          increment,
          increment,
          increment
        ]}
        expected={3}
      />
      <hr />
      <TestCase
        baseState={0}
        queue={[
          5,
          increment,
        ]}
        expected={6}
      />
      <hr />
      <TestCase
        baseState={0}
        queue={[
          5,
          increment,
          42,
        ]}
        expected={42}
      />
    </>
  );
}

function TestCase({
  baseState,
  queue,
  expected
}) {
  const actual = getFinalState(baseState, queue);
  return (
    <>
      <p>Base state: <b>{baseState}</b></p>
      <p>Queue: <b>[{queue.join(', ')}]</b></p>
      <p>Expected result: <b>{expected}</b></p>
      <p style={{
        color: actual === expected ?
          'green' :
          'red'
      }}>
        Your result: <b>{actual}</b>
        {' '}
        ({actual === expected ?
          'correct' :
          'wrong'
        })
      </p>
    </>
  );
}
```

</Sandpack>

<Solution>

这正是本页描述的 React 计算最终状态所使用的算法：

<Sandpack>

```js src/processQueue.js active
export function getFinalState(baseState, queue) {
  let finalState = baseState;

  for (let update of queue) {
    if (typeof update === 'function') {
      // 应用更新函数。
      finalState = update(finalState);
    } else {
      // 替换下一个状态。
      finalState = update;
    }
  }

  return finalState;
}
```

```js src/App.js
import { getFinalState } from './processQueue.js';

function increment(n) {
  return n + 1;
}
increment.toString = () => 'n => n+1';

export default function App() {
  return (
    <>
      <TestCase
        baseState={0}
        queue={[1, 1, 1]}
        expected={1}
      />
      <hr />
      <TestCase
        baseState={0}
        queue={[
          increment,
          increment,
          increment
        ]}
        expected={3}
      />
      <hr />
      <TestCase
        baseState={0}
        queue={[
          5,
          increment,
        ]}
        expected={6}
      />
      <hr />
      <TestCase
        baseState={0}
        queue={[
          5,
          increment,
          42,
        ]}
        expected={42}
      />
    </>
  );
}

function TestCase({
  baseState,
  queue,
  expected
}) {
  const actual = getFinalState(baseState, queue);
  return (
    <>
      <p>Base state: <b>{baseState}</b></p>
      <p>Queue: <b>[{queue.join(', ')}]</b></p>
      <p>Expected result: <b>{expected}</b></p>
      <p style={{
        color: actual === expected ?
          'green' :
          'red'
      }}>
        Your result: <b>{actual}</b>
        {' '}
        ({actual === expected ?
          'correct' :
          'wrong'
        })
      </p>
    </>
  );
}
```

</Sandpack>

现在你知道这部分 React 是如何工作的了！

</Solution>

</Challenges>