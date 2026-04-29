---
title: '使用 Refs 引用值'
---

<Intro>

当你希望组件“记住”某些信息，但又不希望这些信息[触发新的渲染](/learn/render-and-commit)时，你可以使用 *ref*。

</Intro>

<YouWillLearn>

- 如何给组件添加 ref
- 如何更新 ref 的值
- ref 与 state 有何不同
- 如何安全地使用 refs

</YouWillLearn>

## 给组件添加 ref {/*adding-a-ref-to-your-component*/}

你可以通过从 React 导入 `useRef` Hook 来给组件添加 ref：

```js
import { useRef } from 'react';
```

在组件内部，调用 `useRef` Hook，并将你想要引用的初始值作为唯一参数传入。例如，这里有一个指向值 `0` 的 ref：

```js
const ref = useRef(0);
```

`useRef` 返回一个像这样的对象：

```js
{
  current: 0 // 你传给 useRef 的值
}
```

<Illustration src="/images/docs/illustrations/i_ref.png" alt="一个带有写着 'current' 的箭头，塞进一个写着 'ref' 的口袋里。" />

你可以通过 `ref.current` 属性访问这个 ref 的当前值。这个值被设计为可变的，这意味着你既可以读取它，也可以写入它。它就像你组件里的一个秘密口袋，React 不会跟踪它。（这就是它为何能成为逃离 React 单向数据流的“逃生口”——下面会详细说明！）

这里，一个按钮会在每次点击时递增 `ref.current`：

<Sandpack>

```js
import { useRef } from 'react';

export default function Counter() {
  let ref = useRef(0);

  function handleClick() {
    ref.current = ref.current + 1;
    alert('You clicked ' + ref.current + ' times!');
  }

  return (
    <button onClick={handleClick}>
      Click me!
    </button>
  );
}
```

</Sandpack>

这个 ref 指向一个数字，但和[state](/learn/state-a-components-memory)一样，你也可以让它指向任何东西：字符串、对象，甚至函数。与 state 不同，ref 只是一个普通的 JavaScript 对象，带有你可以读取和修改的 `current` 属性。

请注意，**组件不会在每次递增时重新渲染。** 和 state 一样，ref 会被 React 在多次重新渲染之间保留。然而，设置 state 会重新渲染组件。改变 ref 不会！

## 示例：构建一个秒表 {/*example-building-a-stopwatch*/}

你可以在同一个组件中同时结合 refs 和 state。例如，我们来做一个秒表，用户可以通过按按钮来开始或停止它。为了显示用户按下“Start”后已经过去了多长时间，你需要跟踪 Start 按钮何时被按下，以及当前时间是什么。**这些信息用于渲染，所以你会把它保存在 state 中：**

```js
const [startTime, setStartTime] = useState(null);
const [now, setNow] = useState(null);
```

当用户按下“Start”时，你会使用 [`setInterval`](https://developer.mozilla.org/docs/Web/API/setInterval) 每隔 10 毫秒更新时间：

<Sandpack>

```js
import { useState } from 'react';

export default function Stopwatch() {
  const [startTime, setStartTime] = useState(null);
  const [now, setNow] = useState(null);

  function handleStart() {
    // 开始计时。
    setStartTime(Date.now());
    setNow(Date.now());

    setInterval(() => {
      // 每 10 毫秒更新当前时间。
      setNow(Date.now());
    }, 10);
  }

  let secondsPassed = 0;
  if (startTime != null && now != null) {
    secondsPassed = (now - startTime) / 1000;
  }

  return (
    <>
      <h1>已经过去的时间：{secondsPassed.toFixed(3)}</h1>
      <button onClick={handleStart}>
        开始
      </button>
    </>
  );
}
```

</Sandpack>

当按下“Stop”按钮时，你需要取消现有的 interval，以便它停止更新 `now` 状态变量。你可以通过调用 [`clearInterval`](https://developer.mozilla.org/en-US/docs/Web/API/clearInterval) 来做到这一点，但你需要传入之前在用户按下 Start 时由 `setInterval` 调用返回的 interval ID。你需要把这个 interval ID 保存在某个地方。**由于 interval ID 不用于渲染，你可以把它保存在 ref 中：**

<Sandpack>

```js
import { useState, useRef } from 'react';

export default function Stopwatch() {
  const [startTime, setStartTime] = useState(null);
  const [now, setNow] = useState(null);
  const intervalRef = useRef(null);

  function handleStart() {
    setStartTime(Date.now());
    setNow(Date.now());

    clearInterval(intervalRef.current);
    intervalRef.current = setInterval(() => {
      setNow(Date.now());
    }, 10);
  }

  function handleStop() {
    clearInterval(intervalRef.current);
  }

  let secondsPassed = 0;
  if (startTime != null && now != null) {
    secondsPassed = (now - startTime) / 1000;
  }

  return (
    <>
      <h1>已经过去的时间：{secondsPassed.toFixed(3)}</h1>
      <button onClick={handleStart}>
        开始
      </button>
      <button onClick={handleStop}>
        停止
      </button>
    </>
  );
}
```

</Sandpack>

当某些信息用于渲染时，把它保存在 state 中。当某些信息只被事件处理函数需要，并且修改它不需要重新渲染时，使用 ref 可能会更高效。

## refs 和 state 的区别 {/*differences-between-refs-and-state*/}

也许你会觉得 ref 比 state “不那么严格”——例如，你可以直接修改它们，而不是总是必须使用 state 设置函数。但在大多数情况下，你会想使用 state。refs 是一种你不常需要的“逃生口”。下面是 state 和 ref 的对比：

| refs                                                                                  | state                                                                                                                     |
| ------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------- |
| `useRef(initialValue)` 返回 `{ current: initialValue }`                            | `useState(initialValue)` 返回 state 变量的当前值和一个 state 设置函数（`[value, setValue]`） |
| 当你修改它时，不会触发重新渲染。                                         | 当你修改它时，会触发重新渲染。                                                                                    |
| 可变——你可以在渲染过程之外修改和更新 `current` 的值。 | “不可变”——你必须使用 state 设置函数来修改 state 变量，以便排队触发重新渲染。                       |
| 你不应该在渲染期间读取（或写入）`current` 的值。 | 你可以随时读取 state。不过，每次渲染都有自己那一份不会改变的 [快照](/learn/state-as-a-snapshot)。 |

下面是一个使用 state 实现的计数按钮：

<Sandpack>

```js
import { useState } from 'react';

export default function Counter() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1);
  }

  return (
    <button onClick={handleClick}>
      你点击了 {count} 次
    </button>
  );
}
```

</Sandpack>

因为 `count` 值会显示出来，所以把它作为 state 值是合理的。当通过 `setCount()` 设置计数器的值时，React 会重新渲染组件，屏幕也会更新以反映新的计数。

如果你尝试用 ref 来实现这个功能，React 就永远不会重新渲染组件，所以你永远看不到计数变化！看看点击这个按钮时**不会更新其文本**：

<Sandpack>

```js {expectedErrors: {'react-compiler': [13]}}
import { useRef } from 'react';

export default function Counter() {
  let countRef = useRef(0);

  function handleClick() {
    // 这不会重新渲染组件！
    countRef.current = countRef.current + 1;
  }

  return (
    <button onClick={handleClick}>
      你点击了 {countRef.current} 次
    </button>
  );
}
```

</Sandpack>

这就是为什么在渲染期间读取 `ref.current` 会导致代码不可靠。如果你需要这样做，请改用 state。

<DeepDive>

#### useRef 在内部是如何工作的？ {/*how-does-use-ref-work-inside*/}

虽然 `useState` 和 `useRef` 都由 React 提供，但原则上 `useRef` 可以在 `useState` 的基础上实现。你可以把 React 内部的 `useRef` 想象成这样实现的：

```js
// 在 React 内部
function useRef(initialValue) {
  const [ref, unused] = useState({ current: initialValue });
  return ref;
}
```

在第一次渲染期间，`useRef` 返回 `{ current: initialValue }`。这个对象会被 React 存储起来，因此在下一次渲染时会返回同一个对象。请注意在这个例子中 state setter 并未使用。它是不必要的，因为 `useRef` 总是需要返回同一个对象！

React 提供了内置版本的 `useRef`，因为它在实践中已经足够常见。但你可以把它理解为一个没有 setter 的普通 state 变量。如果你熟悉面向对象编程，ref 可能会让你联想到实例字段——只是你写的是 `somethingRef.current`，而不是 `this.something`。

</DeepDive>

## 何时使用 refs {/*when-to-use-refs*/}

通常，当你的组件需要“跳出” React 并与外部 API 通信时，你会使用 ref——通常是不会影响组件外观的浏览器 API。以下是这些少见情况中的几个：

- 存储 [timeout ID](https://developer.mozilla.org/docs/Web/API/setTimeout)
- 存储和操作 [DOM 元素](https://developer.mozilla.org/docs/Web/API/Element)，我们会在[下一页](/learn/manipulating-the-dom-with-refs)介绍
- 存储其他不需要用于计算 JSX 的对象。

如果你的组件需要存储某个值，但它不会影响渲染逻辑，请选择 refs。

## refs 的最佳实践 {/*best-practices-for-refs*/}

遵循这些原则会让你的组件更可预测：

- **将 refs 视为一种逃生口。** 当你与外部系统或浏览器 API 交互时，refs 非常有用。如果你的应用逻辑和数据流很大一部分都依赖 refs，你可能需要重新思考你的方案。
- **不要在渲染期间读取或写入 `ref.current`。** 如果某些信息在渲染期间需要使用，请改用 [state](/learn/state-a-components-memory)。由于 React 不知道 `ref.current` 何时变化，即使在渲染时读取它，也会让组件行为变得难以预测。（唯一的例外是像 `if (!ref.current) ref.current = new Thing()` 这样的代码，它只会在第一次渲染时设置一次 ref。）

React state 的限制不适用于 refs。例如，state 的行为就像每次渲染都有一个 [快照](/learn/state-as-a-snapshot)，并且 [不会同步更新。](/learn/queueing-a-series-of-state-updates) 但当你修改 ref 的当前值时，它会立即改变：

```js
ref.current = 5;
console.log(ref.current); // 5
```

这是因为**ref 本身就是一个普通的 JavaScript 对象，** 因而它的表现也像一个普通对象。

当你使用 ref 时，也不需要担心[避免突变](/learn/updating-objects-in-state)。只要你修改的对象不用于渲染，React 并不关心你对 ref 或其内容做了什么。

## Refs 和 DOM {/*refs-and-the-dom*/}

你可以将 ref 指向任何值。不过，ref 最常见的用途是访问 DOM 元素。例如，如果你想以编程方式聚焦一个输入框，这就很方便。当你在 JSX 中将 ref 传递给 `ref` 属性时，例如 `<div ref={myRef}>`，React 会把对应的 DOM 元素放到 `myRef.current` 中。一旦该元素从 DOM 中移除，React 就会把 `myRef.current` 更新为 `null`。你可以在 [使用 Refs 操作 DOM。](/learn/manipulating-the-dom-with-refs) 中了解更多内容。

<Recap>

- Ref 是一种“逃生通道”，用于保存那些不参与渲染的值。你不会经常需要它们。
- Ref 是一个普通的 JavaScript 对象，只有一个名为 `current` 的属性，你可以读取或设置它。
- 你可以通过调用 `useRef` Hook 让 React 给你一个 ref。
- 和 state 一样，ref 允许你在组件重新渲染之间保留信息。
- 不同于 state，设置 ref 的 `current` 值不会触发重新渲染。
- 不要在渲染期间读取或写入 `ref.current`。这会让你的组件难以预测。

</Recap>



<Challenges>

#### 修复一个损坏的聊天输入框 {/*fix-a-broken-chat-input*/}

输入一条消息并点击 "Send"。你会注意到，在看到 "Sent!" 提示之前有 3 秒延迟。在这段延迟期间，你可以看到一个 "Undo" 按钮。点击它。这个 "Undo" 按钮本应该阻止 "Sent!" 消息出现。它通过调用 [`clearTimeout`](https://developer.mozilla.org/en-US/docs/Web/API/clearTimeout) 来取消在 `handleSend` 中保存的定时器 ID 来实现这一点。然而，即使点击了 "Undo"，"Sent!" 消息仍然会出现。找出它为什么不起作用，并修复它。

<Hint>

像 `let timeoutID` 这样的普通变量不会在重新渲染之间“存活下来”，因为每次渲染都会重新运行你的组件（并从头初始化其中的变量）。你应该把这个定时器 ID 存到别的地方吗？

</Hint>

<Sandpack>

```js {expectedErrors: {'react-compiler': [10]}}
import { useState } from 'react';

export default function Chat() {
  const [text, setText] = useState('');
  const [isSending, setIsSending] = useState(false);
  let timeoutID = null;

  function handleSend() {
    setIsSending(true);
    timeoutID = setTimeout(() => {
      alert('Sent!');
      setIsSending(false);
    }, 3000);
  }

  function handleUndo() {
    setIsSending(false);
    clearTimeout(timeoutID);
  }

  return (
    <>
      <input
        disabled={isSending}
        value={text}
        onChange={e => setText(e.target.value)}
      />
      <button
        disabled={isSending}
        onClick={handleSend}>
        {isSending ? 'Sending...' : 'Send'}
      </button>
      {isSending &&
        <button onClick={handleUndo}>
          Undo
        </button>
      }
    </>
  );
}
```

</Sandpack>

<Solution>

每当你的组件重新渲染时（例如当你设置 state 时），所有局部变量都会从头开始初始化。这就是为什么你不能把定时器 ID 存在像 `timeoutID` 这样的局部变量里，然后还指望别的事件处理函数在未来“看到”它。相反，把它存到 ref 中，React 会在各次渲染之间保留它。

<Sandpack>

```js
import { useState, useRef } from 'react';

export default function Chat() {
  const [text, setText] = useState('');
  const [isSending, setIsSending] = useState(false);
  const timeoutRef = useRef(null);

  function handleSend() {
    setIsSending(true);
    timeoutRef.current = setTimeout(() => {
      alert('Sent!');
      setIsSending(false);
    }, 3000);
  }

  function handleUndo() {
    setIsSending(false);
    clearTimeout(timeoutRef.current);
  }

  return (
    <>
      <input
        disabled={isSending}
        value={text}
        onChange={e => setText(e.target.value)}
      />
      <button
        disabled={isSending}
        onClick={handleSend}>
        {isSending ? 'Sending...' : 'Send'}
      </button>
      {isSending &&
        <button onClick={handleUndo}>
          Undo
        </button>
      }
    </>
  );
}
```

</Sandpack>

</Solution>


#### 修复一个无法重新渲染的组件 {/*fix-a-component-failing-to-re-render*/}

这个按钮本应该在显示 "On" 和 "Off" 之间切换。然而，它总是显示 "Off"。这段代码有什么问题？修复它。

<Sandpack>

```js {expectedErrors: {'react-compiler': [10]}}
import { useRef } from 'react';

export default function Toggle() {
  const isOnRef = useRef(false);

  return (
    <button onClick={() => {
      isOnRef.current = !isOnRef.current;
    }}>
      {isOnRef.current ? 'On' : 'Off'}
    </button>
  );
}
```

</Sandpack>

<Solution>

在这个例子中，ref 的当前值被用于计算渲染输出：`{isOnRef.current ? 'On' : 'Off'}`。这表明这些信息不应该放在 ref 中，而应该放在 state 中。要修复它，移除 ref，改用 state：

<Sandpack>

```js
import { useState } from 'react';

export default function Toggle() {
  const [isOn, setIsOn] = useState(false);

  return (
    <button onClick={() => {
      setIsOn(!isOn);
    }}>
      {isOn ? 'On' : 'Off'}
    </button>
  );
}
```

</Sandpack>

</Solution>

#### 修复防抖 {/*fix-debouncing*/}

在这个例子中，所有按钮点击处理函数都经过了 [“防抖”。](https://kettanaito.com/blog/debounce-vs-throttle) 要了解这是什么意思，按下其中一个按钮。注意消息会在一秒后出现。如果你在等待消息出现时再次按下按钮，计时器会重置。因此，如果你快速连续点击同一个按钮很多次，消息要到你停止点击后一秒才会出现。防抖允许你把某些操作延迟到用户“停止操作”之后再执行。

这个例子是可运行的，但并不完全符合预期。这些按钮并不是独立的。要看到问题，点击其中一个按钮，然后立刻点击另一个按钮。你会以为延迟之后会看到两个按钮的消息，但实际上只会显示最后一个按钮的消息。第一个按钮的消息丢失了。

为什么这些按钮会相互干扰？找出并修复这个问题。

<Hint>

最后那个定时器 ID 变量是所有 `DebouncedButton` 组件共享的。这就是为什么点击一个按钮会重置另一个按钮的定时器。你能为每个按钮分别保存一个单独的定时器 ID 吗？

</Hint>

<Sandpack>

```js
let timeoutID;

function DebouncedButton({ onClick, children }) {
  return (
    <button onClick={() => {
      clearTimeout(timeoutID);
      timeoutID = setTimeout(() => {
        onClick();
      }, 1000);
    }}>
      {children}
    </button>
  );
}

export default function Dashboard() {
  return (
    <>
      <DebouncedButton
        onClick={() => alert('Spaceship launched!')}
      >
        Launch the spaceship
      </DebouncedButton>
      <DebouncedButton
        onClick={() => alert('Soup boiled!')}
      >
        Boil the soup
      </DebouncedButton>
      <DebouncedButton
        onClick={() => alert('Lullaby sung!')}
      >
        Sing a lullaby
      </DebouncedButton>
    </>
  )
}
```

```css
button { display: block; margin: 10px; }
```

</Sandpack>

<Solution>

像 `timeoutID` 这样的变量会在所有组件之间共享。这就是为什么点击第二个按钮会重置第一个按钮尚未触发的定时器。要修复它，你可以把定时器保存在 ref 中。每个按钮都会得到自己的 ref，因此它们不会互相冲突。注意快速点击两个按钮会显示两个消息。

<Sandpack>

```js
import { useRef } from 'react';

function DebouncedButton({ onClick, children }) {
  const timeoutRef = useRef(null);
  return (
    <button onClick={() => {
      clearTimeout(timeoutRef.current);
      timeoutRef.current = setTimeout(() => {
        onClick();
      }, 1000);
    }}>
      {children}
    </button>
  );
}

export default function Dashboard() {
  return (
    <>
      <DebouncedButton
        onClick={() => alert('Spaceship launched!')}
      >
        Launch the spaceship
      </DebouncedButton>
      <DebouncedButton
        onClick={() => alert('Soup boiled!')}
      >
        Boil the soup
      </DebouncedButton>
      <DebouncedButton
        onClick={() => alert('Lullaby sung!')}
      >
        Sing a lullaby
      </DebouncedButton>
    </>
  )
}
```

```css
button { display: block; margin: 10px; }
```

</Sandpack>

</Solution>

#### 读取最新的 state {/*read-the-latest-state*/}

在这个例子中，按下 "Send" 后，消息显示之前会有一个小延迟。输入 "hello"，按下 Send，然后快速再次编辑输入框。尽管你进行了编辑，提示仍然会显示 "hello"（也就是按钮被点击时 state 的[那个时刻](/learn/state-as-a-snapshot#state-over-time)的值）。

通常，在应用中这种行为正是你想要的。不过，在某些情况下，你可能希望某些异步代码读取某个 state 的*最新*版本。你能想到一种方法，让提示显示*当前*的输入文本，而不是点击时的内容吗？

<Sandpack>

```js
import { useState, useRef } from 'react';

export default function Chat() {
  const [text, setText] = useState('');

  function handleSend() {
    setTimeout(() => {
      alert('Sending: ' + text);
    }, 3000);
  }

  return (
    <>
      <input
        value={text}
        onChange={e => setText(e.target.value)}
      />
      <button
        onClick={handleSend}>
        Send
      </button>
    </>
  );
}
```

</Sandpack>

<Solution>

state 的工作方式[像快照一样](/learn/state-as-a-snapshot)，所以你不能从像定时器这样的异步操作中读取最新的 state。不过，你可以把最新的输入文本保存在 ref 中。ref 是可变的，所以你可以随时读取 `current` 属性。由于当前文本也用于渲染，在这个例子中，你将需要*同时*使用一个 state 变量（用于渲染）和一个 ref（用于在定时器中读取它）。你需要手动更新 ref 的当前值。

<Sandpack>

```js
import { useState, useRef } from 'react';

export default function Chat() {
  const [text, setText] = useState('');
  const textRef = useRef(text);

  function handleChange(e) {
    setText(e.target.value);
    textRef.current = e.target.value;
  }

  function handleSend() {
    setTimeout(() => {
      alert('Sending: ' + textRef.current);
    }, 3000);
  }

  return (
    <>
      <input
        value={text}
        onChange={handleChange}
      />
      <button
        onClick={handleSend}>
        Send
      </button>
    </>
  );
}
```

</Sandpack>

</Solution>

</Challenges>
