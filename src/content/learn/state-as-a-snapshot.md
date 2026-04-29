---
title: 状态如同快照
---

<Intro>

状态变量看起来可能像普通的 JavaScript 变量，你可以对它们读写。然而，状态的行为更像一张快照。对它赋值不会改变你已经拥有的状态变量，而是会触发重新渲染。

</Intro>

<YouWillLearn>

* 设置状态如何触发重新渲染
* 何时以及如何更新状态
* 为什么在你设置状态后它不会立即更新
* 事件处理函数如何访问状态的“快照”

</YouWillLearn>

## 设置状态会触发渲染 {/*setting-state-triggers-renders*/}

你可能会认为，用户界面会直接响应用户事件，比如点击。但在 React 中，它的工作方式和这种心理模型有些不同。在上一页，你已经看到 [设置状态会向 React 请求一次重新渲染](/learn/render-and-commit#step-1-trigger-a-render)。这意味着，要让界面对事件作出反应，你需要*更新状态*。

在这个示例中，当你按下“send”时，`setIsSent(true)` 会告诉 React 重新渲染 UI：

<Sandpack>

```js
import { useState } from 'react';

export default function Form() {
  const [isSent, setIsSent] = useState(false);
  const [message, setMessage] = useState('Hi!');
  if (isSent) {
    return <h1>Your message is on its way!</h1>
  }
  return (
    <form onSubmit={(e) => {
      e.preventDefault();
      setIsSent(true);
      sendMessage(message);
    }}>
      <textarea
        placeholder="Message"
        value={message}
        onChange={e => setMessage(e.target.value)}
      />
      <button type="submit">Send</button>
    </form>
  );
}

function sendMessage(message) {
  // ...
}
```

```css
label, textarea { margin-bottom: 10px; display: block; }
```

</Sandpack>

当你点击按钮时，会发生以下事情：

1. `onSubmit` 事件处理函数执行。
2. `setIsSent(true)` 将 `isSent` 设为 `true`，并排队一次新的渲染。
3. React 根据新的 `isSent` 值重新渲染组件。

让我们更仔细地看看状态和渲染之间的关系。

## 渲染会在某一时刻截取快照 {/*rendering-takes-a-snapshot-in-time*/}

[“渲染”](/learn/render-and-commit#step-2-react-renders-your-components) 的意思是 React 正在调用你的组件，而组件本身就是一个函数。你从这个函数返回的 JSX 就像是 UI 在某一时刻的快照。它的 props、事件处理函数和局部变量，都是**使用渲染发生时的状态**计算出来的。

与照片或电影帧不同，你返回的 UI“快照”是可交互的。它包含像事件处理函数这样的逻辑，用来指定输入发生时会发生什么。React 会更新屏幕以匹配这个快照，并连接这些事件处理函数。因此，按下按钮会触发你 JSX 中的点击处理函数。

当 React 重新渲染一个组件时：

1. React 再次调用你的函数。
2. 你的函数返回一个新的 JSX 快照。
3. 然后 React 更新屏幕以匹配你的函数返回的快照。

<IllustrationBlock sequential>
    <Illustration caption="React 正在执行函数" src="/images/docs/illustrations/i_render1.png" />
    <Illustration caption="计算快照" src="/images/docs/illustrations/i_render2.png" />
    <Illustration caption="更新 DOM 树" src="/images/docs/illustrations/i_render3.png" />
</IllustrationBlock>

作为组件的“记忆”，状态并不像普通变量那样会在函数返回后消失。状态实际上“存在”于 React 自身中——就像放在架子上一样！——而不是存在于你的函数外部。当 React 调用你的组件时，它会为该次渲染提供一份状态快照。你的组件会在 JSX 中返回一份 UI 快照，其中带有一组新的 props 和事件处理函数，所有这些都**使用该次渲染中的状态值**计算而来！

<IllustrationBlock sequential>
  <Illustration caption="你告诉 React 更新状态" src="/images/docs/illustrations/i_state-snapshot1.png" />
  <Illustration caption="React 更新状态值" src="/images/docs/illustrations/i_state-snapshot2.png" />
  <Illustration caption="React 将状态值的快照传递给组件" src="/images/docs/illustrations/i_state-snapshot3.png" />
</IllustrationBlock>

下面有个小实验可以展示它是如何工作的。在这个示例中，你可能会以为点击“+3”按钮会把计数器加三次，因为它调用了三次 `setNumber(number + 1)`。

看看点击“+3”按钮时会发生什么：

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

注意，`number` 每次点击只会增加一次！

**设置状态只会影响*下一次*渲染。** 在第一次渲染时，`number` 是 `0`。这就是为什么在*那次渲染*的 `onClick` 处理函数中，即使调用了 `setNumber(number + 1)`，`number` 的值仍然是 `0`：

```js
<button onClick={() => {
  setNumber(number + 1);
  setNumber(number + 1);
  setNumber(number + 1);
}}>+3</button>
```

下面是这个按钮的点击处理函数告诉 React 要做的事情：

1. `setNumber(number + 1)`：`number` 是 `0`，所以是 `setNumber(0 + 1)`。
    - React 准备在下一次渲染时把 `number` 改成 `1`。
2. `setNumber(number + 1)`：`number` 是 `0`，所以是 `setNumber(0 + 1)`。
    - React 准备在下一次渲染时把 `number` 改成 `1`。
3. `setNumber(number + 1)`：`number` 是 `0`，所以是 `setNumber(0 + 1)`。
    - React 准备在下一次渲染时把 `number` 改成 `1`。

尽管你调用了三次 `setNumber(number + 1)`，但在*这次渲染*的事件处理函数中，`number` 始终是 `0`，所以你把状态设成了 `1` 三次。这就是为什么事件处理函数执行完后，React 重新渲染组件时 `number` 是 `1` 而不是 `3`。

你也可以通过在 ذهن中把代码里的状态变量替换成它们的值来可视化这一点。由于 `number` 状态变量在*这次渲染*中是 `0`，它的事件处理函数看起来像这样：

```js
<button onClick={() => {
  setNumber(0 + 1);
  setNumber(0 + 1);
  setNumber(0 + 1);
}}>+3</button>
```

对于下一次渲染，`number` 是 `1`，所以*那次渲染*的点击处理函数看起来像这样：

```js
<button onClick={() => {
  setNumber(1 + 1);
  setNumber(1 + 1);
  setNumber(1 + 1);
}}>+3</button>
```

这就是为什么再次点击按钮会把计数器设为 `2`，下一次点击再设为 `3`，以此类推。

## 状态随时间变化 {/*state-over-time*/}

好了，这很有趣。试着猜猜点击这个按钮会弹出什么：

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
        alert(number);
      }}>+5</button>
    </>
  )
}
```

```css
button { display: inline-block; margin: 10px; font-size: 20px; }
h1 { display: inline-block; margin: 10px; width: 30px; text-align: center; }
```

</Sandpack>

如果你使用前面的替换方法，就能猜到 alert 显示的是“0”：

```js
setNumber(0 + 5);
alert(0);
```

但如果你给 alert 加一个定时器，让它只在组件重新渲染*之后*触发，会显示“0”还是“5”？猜猜看！

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
        setTimeout(() => {
          alert(number);
        }, 3000);
      }}>+5</button>
    </>
  )
}
```

```css
button { display: inline-block; margin: 10px; font-size: 20px; }
h1 { display: inline-block; margin: 10px; width: 30px; text-align: center; }
```

</Sandpack>

惊讶吗？如果你使用替换方法，就能看到传给 alert 的状态“快照”。

```js
setNumber(0 + 5);
setTimeout(() => {
  alert(0);
}, 3000);
```

等到 alert 运行时，React 中存储的状态可能已经改变了，但它是使用用户交互时的状态快照来安排的！

**状态变量的值在一次渲染中永远不会改变，** 即使它的事件处理函数代码是异步的。在*那次渲染*的 `onClick` 中，即使调用了 `setNumber(number + 5)`，`number` 的值仍然会是 `0`。当 React 通过调用你的组件“截取快照”时，它的值就已经“固定”了。

下面是一个例子，说明这如何让你的事件处理函数不那么容易出错。下面是一个会在五秒后发送消息的表单。设想这种场景：

1. 你按下“Send”按钮，把“Hello”发送给 Alice。
2. 在五秒延迟结束之前，你把“To”字段的值改成 Bob。

你希望 `alert` 显示什么？它会显示“You said Hello to Alice”吗？还是会显示“You said Hello to Bob”？根据你目前所学猜一猜，然后试试：

<Sandpack>

```js
import { useState } from 'react';

export default function Form() {
  const [to, setTo] = useState('Alice');
  const [message, setMessage] = useState('Hello');

  function handleSubmit(e) {
    e.preventDefault();
    setTimeout(() => {
      alert(`You said ${message} to ${to}`);
    }, 5000);
  }

  return (
    <form onSubmit={handleSubmit}>
      <label>
        To:{' '}
        <select
          value={to}
          onChange={e => setTo(e.target.value)}>
          <option value="Alice">Alice</option>
          <option value="Bob">Bob</option>
        </select>
      </label>
      <textarea
        placeholder="Message"
        value={message}
        onChange={e => setMessage(e.target.value)}
      />
      <button type="submit">Send</button>
    </form>
  );
}
```

```css
label, textarea { margin-bottom: 10px; display: block; }
```

</Sandpack>

**React 会在一次渲染的事件处理函数中把状态值“固定”住。** 你不需要担心代码运行时状态是否已经改变。

但如果你想在重新渲染之前读取最新的状态呢？你会想使用下一页会介绍的 [state updater function](/learn/queueing-a-series-of-state-updates)！

<Recap>

* 设置状态会请求一次新的渲染。
* React 将状态存储在组件外部，就像放在架子上一样。
* 当你调用 `useState` 时，React 会为*那次渲染*提供一份状态快照。
* 变量和事件处理函数不会“在”重新渲染之间存活下来。每次渲染都有自己的事件处理函数。
* 每次渲染（以及其中的函数）总会“看到”React 为*那次*渲染提供的状态快照。
* 你可以在脑中把事件处理函数里的状态替换成具体值，就像你思考已渲染的 JSX 一样。
* 过去创建的事件处理函数拥有它们创建时那次渲染的状态值。

</Recap>



<Challenges>

#### 实现一个交通灯 {/*implement-a-traffic-light*/}

下面是一个人行横道灯组件，按下按钮时会切换：

<Sandpack>

```js
import { useState } from 'react';

export default function TrafficLight() {
  const [walk, setWalk] = useState(true);

  function handleClick() {
    setWalk(!walk);
  }

  return (
    <>
      <button onClick={handleClick}>
        Change to {walk ? 'Stop' : 'Walk'}
      </button>
      <h1 style={{
        color: walk ? 'darkgreen' : 'darkred'
      }}>
        {walk ? 'Walk' : 'Stop'}
      </h1>
    </>
  );
}
```

```css
h1 { margin-top: 20px; }
```

</Sandpack>

给点击处理函数添加一个 `alert`。当灯是绿色并显示“Walk”时，点击按钮应显示“Stop is next”。当灯是红色并显示“Stop”时，点击按钮应显示“Walk is next”。

把 `alert` 放在 `setWalk` 调用之前或之后有区别吗？

<Solution>

你的 `alert` 应该像这样：

<Sandpack>

```js
import { useState } from 'react';

export default function TrafficLight() {
  const [walk, setWalk] = useState(true);

  function handleClick() {
    setWalk(!walk);
    alert(walk ? 'Stop is next' : 'Walk is next');
  }

  return (
    <>
      <button onClick={handleClick}>
        Change to {walk ? 'Stop' : 'Walk'}
      </button>
      <h1 style={{
        color: walk ? 'darkgreen' : 'darkred'
      }}>
        {walk ? 'Walk' : 'Stop'}
      </h1>
    </>
  );
}
```

```css
h1 { margin-top: 20px; }
```

</Sandpack>

把它放在 `setWalk` 调用之前或之后都没有区别。那次渲染中的 `walk` 值是固定的。调用 `setWalk` 只会在*下一次*渲染中改变它，但不会影响上一次渲染中的事件处理函数。

这一行起初可能看起来有点违反直觉：

```js
alert(walk ? 'Stop is next' : 'Walk is next');
```

但如果把它理解为：“如果交通灯显示‘Walk now’，那么消息应该说‘Stop is next’。” 就很合理了。事件处理函数中的 `walk` 变量与那次渲染中的 `walk` 值一致，并且不会改变。

你可以通过应用替换方法来验证这是正确的。当 `walk` 是 `true` 时，你会得到：

```js
<button onClick={() => {
  setWalk(false);
  alert('Stop is next');
}}>
  Change to Stop
</button>
<h1 style={{color: 'darkgreen'}}>
  Walk
</h1>
```

所以点击“Change to Stop”会排队一次将 `walk` 设为 `false` 的渲染，并弹出“Stop is next”。

</Solution>

</Challenges>
