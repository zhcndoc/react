---
title: useRef
---

<Intro>

`useRef` 是一个 React Hook，它可以让你引用一个对渲染不需要的值。

```js
const ref = useRef(initialValue)
```

</Intro>

<InlineToc />

---

## 参考 {/*reference*/}

### `useRef(initialValue)` {/*useref*/}

在组件顶层调用 `useRef` 来声明一个 [ref.](/learn/referencing-values-with-refs)

```js
import { useRef } from 'react';

function MyComponent() {
  const intervalRef = useRef(0);
  const inputRef = useRef(null);
  // ...
```

[查看更多示例。](#usage)

#### 参数 {/*parameters*/}

* `initialValue`：你希望 ref 对象的 `current` 属性初始时的值。它可以是任何类型的值。该参数在初次渲染之后会被忽略。

#### 返回值 {/*returns*/}

`useRef` 返回一个只有单个属性的对象：

* `current`：初始时，它被设置为你传入的 `initialValue`。之后你可以把它设为别的值。如果你把 ref 对象作为 JSX 节点的 `ref` 属性传给 React，React 会设置它的 `current` 属性。

在后续渲染中，`useRef` 会返回同一个对象。

#### 注意事项 {/*caveats*/}

* 你可以修改 `ref.current` 属性。与 state 不同，它是可变的。不过，如果它持有一个用于渲染的对象（例如，你的一部分 state），那么你不应该修改那个对象。
* 当你改变 `ref.current` 属性时，React 不会重新渲染你的组件。React 不知道你何时改变了它，因为 ref 只是一个普通的 JavaScript 对象。
* 在渲染期间不要写入 _或读取_ `ref.current`，初始化除外。这样会让组件行为变得不可预测。
* 在严格模式下，React 会为帮助你发现意外的副作用而 **调用你的组件函数两次**。这是仅开发环境下的行为，不会影响生产环境。每个 ref 对象都会创建两次，但其中一个版本会被丢弃。如果你的组件函数是纯函数（它本应如此），这不应该影响行为。

---

## 用法 {/*usage*/}

### 用 ref 引用一个值 {/*referencing-a-value-with-a-ref*/}

在组件顶层调用 `useRef` 来声明一个或多个 [refs.](/learn/referencing-values-with-refs)

```js [[1, 4, "intervalRef"], [3, 4, "0"]]
import { useRef } from 'react';

function Stopwatch() {
  const intervalRef = useRef(0);
  // ...
```

`useRef` 返回一个 <CodeStep step={1}>ref 对象</CodeStep>，它有一个单独的 <CodeStep step={2}>`current` 属性</CodeStep>，初始值设为你提供的 <CodeStep step={3}>初始值</CodeStep>。

在后续渲染中，`useRef` 会返回同一个对象。你可以修改它的 `current` 属性来存储信息，并在之后读取。这可能会让你联想到 [state](/reference/react/useState)，但二者有一个重要区别。

**修改 ref 不会触发重新渲染。** 这意味着 ref 非常适合存储那些不会影响组件视觉输出的信息。比如，如果你需要存储一个 [interval ID](https://developer.mozilla.org/en-US/docs/Web/API/setInterval) 并在之后取回它，你可以把它放进 ref。要更新 ref 里的值，你需要手动修改它的 <CodeStep step={2}>`current` 属性</CodeStep>：

```js [[2, 5, "intervalRef.current"]]
function handleStartClick() {
  const intervalId = setInterval(() => {
    // ...
  }, 1000);
  intervalRef.current = intervalId;
}
```

之后，你可以从 ref 中读取这个 interval ID，以便调用 [清除该计时器](https://developer.mozilla.org/en-US/docs/Web/API/clearInterval)：

```js [[2, 2, "intervalRef.current"]]
function handleStopClick() {
  const intervalId = intervalRef.current;
  clearInterval(intervalId);
}
```

使用 ref，可以确保：

- 你可以在重新渲染之间 **存储信息**（不同于普通变量，它们会在每次渲染时重置）。
- 修改它 **不会触发重新渲染**（不同于 state 变量，它们会触发重新渲染）。
- **信息是局部的**，属于组件的每个副本（不同于外部变量，它们是共享的）。

修改 ref 不会触发重新渲染，所以 ref 不适合存储你想显示在屏幕上的信息。对此请改用 state。更多内容请阅读 [在 `useRef` 和 `useState` 之间进行选择。](/learn/referencing-values-with-refs#differences-between-refs-and-state)

<Recipes titleText="useRef 引用值的示例" titleId="examples-value">

#### 点击计数器 {/*click-counter*/}

这个组件使用 ref 来跟踪按钮被点击了多少次。请注意，这里使用 ref 而不是 state 是可以的，因为点击次数只会在事件处理函数中读取和写入。

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

如果你在 JSX 中显示 `{ref.current}`，数字不会在点击时更新。这是因为设置 `ref.current` 不会触发重新渲染。用于渲染的信息应该改用 state。

<Solution />

#### 一个秒表 {/*a-stopwatch*/}

这个示例结合使用了 state 和 ref。`startTime` 和 `now` 都是 state 变量，因为它们用于渲染。但我们还需要保存一个 [interval ID](https://developer.mozilla.org/en-US/docs/Web/API/setInterval)，以便在按下按钮时停止计时器。由于 interval ID 不用于渲染，把它保存在 ref 中并手动更新它是合适的。

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
      <h1>已过时间：{secondsPassed.toFixed(3)}</h1>
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

<Solution />

</Recipes>

<Pitfall>

**在渲染期间不要写入 _或读取_ `ref.current`。**

React 期望你的组件主体 [表现得像一个纯函数](/learn/keeping-components-pure)：

- 如果输入（[props](/learn/passing-props-to-a-component)、[state](/learn/state-a-components-memory) 和 [context](/learn/passing-data-deeply-with-context)）相同，它应该返回完全相同的 JSX。
- 以不同的顺序调用它，或传入不同的参数，不应该影响其他调用的结果。

在 **渲染期间** 读取或写入 ref 会破坏这些预期。

```js {expectedErrors: {'react-compiler': [4]}} {3-4,6-7}
function MyComponent() {
  // ...
  // 🚩 不要在渲染期间写入 ref
  myRef.current = 123;
  // ...
  // 🚩 不要在渲染期间读取 ref
  return <h1>{myOtherRef.current}</h1>;
}
```

你可以改为在 **事件处理函数或 Effects** 中读取或写入 ref。

```js {4-5,9-10}
function MyComponent() {
  // ...
  useEffect(() => {
    // ✅ 你可以在 effects 中读取或写入 ref
    myRef.current = 123;
  });
  // ...
  function handleClick() {
    // ✅ 你可以在事件处理函数中读取或写入 ref
    doSomething(myOtherRef.current);
  }
  // ...
}
```

如果你 *必须* 在渲染期间读取 [或写入](/reference/react/useState#storing-information-from-previous-renders) 某些内容，请改用 [use state](/reference/react/useState)。

当你违反这些规则时，你的组件可能仍然能工作，但我们为 React 添加的大多数更新颖的功能都会依赖这些预期。更多内容请阅读 [保持组件纯净。](/learn/keeping-components-pure#where-you-_can_-cause-side-effects)

</Pitfall>

---

### 使用 ref 操作 DOM {/*manipulating-the-dom-with-a-ref*/}

使用 ref 来操作 [DOM](https://developer.mozilla.org/en-US/docs/Web/API/HTML_DOM_API) 是特别常见的。React 对此提供了内置支持。

首先，使用 `null` 作为 <CodeStep step={3}>初始值</CodeStep> 声明一个 <CodeStep step={1}>ref 对象</CodeStep>：

```js [[1, 4, "inputRef"], [3, 4, "null"]]
import { useRef } from 'react';

function MyComponent() {
  const inputRef = useRef(null);
  // ...
```

然后把你的 ref 对象作为 `ref` 属性传给你想操作的 DOM 节点的 JSX：

```js [[1, 2, "inputRef"]]
  // ...
  return <input ref={inputRef} />;
```

在 React 创建 DOM 节点并将其显示到屏幕后，React 会把你的 ref 对象的 <CodeStep step={2}>`current` 属性</CodeStep>设置为那个 DOM 节点。现在你可以访问 `<input>` 的 DOM 节点，并调用像 [`focus()`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/focus) 这样的方：

```js [[2, 2, "inputRef.current"]]
  function handleClick() {
    inputRef.current.focus();
  }
```

当节点从屏幕上移除时，React 会把 `current` 属性重新设为 `null`。

更多内容请阅读 [使用 refs 操作 DOM。](/learn/manipulating-the-dom-with-refs)

<Recipes titleText="useRef 操作 DOM 的示例" titleId="examples-dom">

#### 聚焦文本输入框 {/*focusing-a-text-input*/}

在这个示例中，点击按钮会聚焦输入框：

<Sandpack>

```js
import { useRef } from 'react';

export default function Form() {
  const inputRef = useRef(null);

  function handleClick() {
    inputRef.current.focus();
  }

  return (
    <>
      <input ref={inputRef} />
      <button onClick={handleClick}>
        聚焦输入框
      </button>
    </>
  );
}
```

</Sandpack>

<Solution />

#### 将图片滚动到视图中 {/*scrolling-an-image-into-view*/}

在这个示例中，点击按钮会将图片滚动到视图中。它使用一个指向列表 DOM 节点的 ref，然后调用 DOM [`querySelectorAll`](https://developer.mozilla.org/en-US/docs/Web/API/Document/querySelectorAll) API 来找到我们想滚动到的图片。

<Sandpack>

```js
import { useRef } from 'react';

export default function CatFriends() {
  const listRef = useRef(null);

  function scrollToIndex(index) {
    const listNode = listRef.current;
    // 这一行假设了一种特定的 DOM 结构：
    const imgNode = listNode.querySelectorAll('li > img')[index];
    imgNode.scrollIntoView({
      behavior: 'smooth',
      block: 'nearest',
      inline: 'center'
    });
  }

  return (
    <>
      <nav>
        <button onClick={() => scrollToIndex(0)}>
          Neo
        </button>
        <button onClick={() => scrollToIndex(1)}>
          Millie
        </button>
        <button onClick={() => scrollToIndex(2)}>
          Bella
        </button>
      </nav>
      <div>
        <ul ref={listRef}>
          <li>
            <img
              src="https://placecats.com/neo/300/200"
              alt="Neo"
            />
          </li>
          <li>
            <img
              src="https://placecats.com/millie/200/200"
              alt="Millie"
            />
          </li>
          <li>
            <img
              src="https://placecats.com/bella/199/200"
              alt="Bella"
            />
          </li>
        </ul>
      </div>
    </>
  );
}
```

```css
div {
  width: 100%;
  overflow: hidden;
}

nav {
  text-align: center;
}

button {
  margin: .25rem;
}

ul,
li {
  list-style: none;
  white-space: nowrap;
}

li {
  display: inline;
  padding: 0.5rem;
}
```

</Sandpack>

<Solution />

#### 播放和暂停视频 {/*playing-and-pausing-a-video*/}

这个示例使用 ref 在 `<video>` DOM 节点上调用 [`play()`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLMediaElement/play) 和 [`pause()`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLMediaElement/pause)。

<Sandpack>

```js
import { useState, useRef } from 'react';

export default function VideoPlayer() {
  const [isPlaying, setIsPlaying] = useState(false);
  const ref = useRef(null);

  function handleClick() {
    const nextIsPlaying = !isPlaying;
    setIsPlaying(nextIsPlaying);

    if (nextIsPlaying) {
      ref.current.play();
    } else {
      ref.current.pause();
    }
  }

  return (
    <>
      <button onClick={handleClick}>
        {isPlaying ? '暂停' : '播放'}
      </button>
      <video
        width="250"
        ref={ref}
        onPlay={() => setIsPlaying(true)}
        onPause={() => setIsPlaying(false)}
      >
        <source
          src="https://interactive-examples.mdn.mozilla.net/media/cc0-videos/flower.mp4"
          type="video/mp4"
        />
      </video>
    </>
  );
}
```

```css
button { display: block; margin-bottom: 20px; }
```

</Sandpack>

<Solution />

#### 在你自己的组件中暴露一个 ref {/*exposing-a-ref-to-your-own-component*/}

有时，你可能希望让父组件操作你组件内部的 DOM。例如，也许你正在编写一个 `MyInput` 组件，但你希望父组件能够聚焦输入框（而父组件本身无法直接访问）。你可以在父组件中创建一个 `ref`，然后把这个 `ref` 作为 prop 传给子组件。这里有一份[详细讲解](/learn/manipulating-the-dom-with-refs#accessing-another-components-dom-nodes)。

<Sandpack>

```js
import { useRef } from 'react';

function MyInput({ ref }) {
  return <input ref={ref} />;
};

export default function Form() {
  const inputRef = useRef(null);

  function handleClick() {
    inputRef.current.focus();
  }

  return (
    <>
      <MyInput ref={inputRef} />
      <button onClick={handleClick}>
        聚焦输入框
      </button>
    </>
  );
}
```

</Sandpack>

<Solution />

</Recipes>

---

### 避免重新创建 ref 内容 {/*avoiding-recreating-the-ref-contents*/}

React 会只保存一次初始 ref 值，并在后续渲染中忽略它。

```js
function Video() {
  const playerRef = useRef(new VideoPlayer());
  // ...
```

虽然 `new VideoPlayer()` 的结果只在初次渲染中使用，但你仍然会在每次渲染时调用这个函数。如果它创建的是开销很大的对象，这会很浪费。

要解决这个问题，你可以像下面这样初始化 ref：

```js
function Video() {
  const playerRef = useRef(null);
  if (playerRef.current === null) {
    playerRef.current = new VideoPlayer();
  }
  // ...
```

通常来说，在渲染期间写入或读取 `ref.current` 是不允许的。不过，在这个例子中是可以的，因为结果始终相同，而且这个条件只会在初始化时执行，所以它是完全可预测的。

<DeepDive>

#### 在稍后初始化 useRef 时如何避免 null 检查 {/*how-to-avoid-null-checks-when-initializing-use-ref-later*/}

如果你使用类型检查器，并且不想总是检查 `null`，你可以尝试下面这种模式：

```js
function Video() {
  const playerRef = useRef(null);

  function getPlayer() {
    if (playerRef.current !== null) {
      return playerRef.current;
    }
    const player = new VideoPlayer();
    playerRef.current = player;
    return player;
  }

  // ...
```

这里，`playerRef` 本身是可空的。不过，你应该能够让你的类型检查器相信，`getPlayer()` 不会返回 `null`。然后在事件处理函数中使用 `getPlayer()`。

</DeepDive>

---

## 故障排除 {/*troubleshooting*/}

### 我无法获取自定义组件的 ref {/*i-cant-get-a-ref-to-a-custom-component*/}

如果你尝试像这样将 `ref` 传递给自己的组件：

```js
const inputRef = useRef(null);

return <MyInput ref={inputRef} />;
```

你可能会在控制台中看到一个错误：

<ConsoleBlock level="error">

TypeError: 无法读取 null 的属性

</ConsoleBlock>

默认情况下，自己的组件不会向外暴露其内部 DOM 节点的 refs。

要修复这个问题，请找到你想要获取 ref 的组件：

```js
export default function MyInput({ value, onChange }) {
  return (
    <input
      value={value}
      onChange={onChange}
    />
  );
}
```

然后将 `ref` 添加到组件接受的 props 列表中，并像这样把 `ref` 作为 prop 传递给相关的子 [内置组件](/reference/react-dom/components/common)：

```js {1,6}
function MyInput({ value, onChange, ref }) {
  return (
    <input
      value={value}
      onChange={onChange}
      ref={ref}
    />
  );
};

export default MyInput;
```

这样父组件就可以获取到它的 ref 了。

阅读更多关于[访问另一个组件的 DOM 节点。](/learn/manipulating-the-dom-with-refs#accessing-another-components-dom-nodes)
