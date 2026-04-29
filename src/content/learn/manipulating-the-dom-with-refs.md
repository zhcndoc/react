---
title: '使用 Refs 操作 DOM'
---

<Intro>

React 会自动更新 [DOM](https://developer.mozilla.org/docs/Web/API/Document_Object_Model/Introduction) 以匹配你的渲染输出，所以你的组件通常不需要手动操作它。然而，有时你可能需要访问由 React 管理的 DOM 元素——例如，聚焦某个节点、滚动到它，或者测量它的大小和位置。在 React 中没有内置的方式来做这些事情，所以你需要一个指向 DOM 节点的 *ref*。

</Intro>

<YouWillLearn>

- 如何使用 `ref` 属性访问由 React 管理的 DOM 节点
- `ref` 这个 JSX 属性与 `useRef` Hook 的关系
- 如何访问另一个组件的 DOM 节点
- 在哪些情况下安全地修改由 React 管理的 DOM

</YouWillLearn>

## 获取指向节点的 ref {/*getting-a-ref-to-the-node*/}

要访问由 React 管理的 DOM 节点，首先导入 `useRef` Hook：

```js
import { useRef } from 'react';
```

然后，在组件内部使用它声明一个 ref：

```js
const myRef = useRef(null);
```

最后，将你的 ref 作为 `ref` 属性传给你想获取 DOM 节点的 JSX 标签：

```js
<div ref={myRef}>
```

`useRef` Hook 会返回一个只包含名为 `current` 的单一属性的对象。初始时，`myRef.current` 会是 `null`。当 React 为这个 `<div>` 创建 DOM 节点时，React 会把对这个节点的引用放入 `myRef.current`。然后你就可以在你的 [事件处理程序](/learn/responding-to-events) 中访问这个 DOM 节点，并使用其上定义的内置 [浏览器 API](https://developer.mozilla.org/docs/Web/API/Element)。

```js
// 你可以使用任何浏览器 API，例如：
myRef.current.scrollIntoView();
```

### 示例：聚焦文本输入框 {/*example-focusing-a-text-input*/}

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

要实现这一点：

1. 使用 `useRef` Hook 声明 `inputRef`。
2. 将它传给 `<input ref={inputRef}>`。这会告诉 React **把这个 `<input>` 的 DOM 节点放入 `inputRef.current`。**
3. 在 `handleClick` 函数中，从 `inputRef.current` 读取输入框的 DOM 节点，并对其调用 [`focus()`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/focus)：`inputRef.current.focus()`。
4. 使用 `onClick` 将 `handleClick` 事件处理程序传给 `<button>`。

虽然操作 DOM 是 refs 最常见的使用场景，但 `useRef` Hook 也可以用于存储 React 之外的其他内容，比如计时器 ID。与 state 类似，refs 会在多次渲染之间保持不变。Refs 有点像 state 变量，但当你设置它们时不会触发重新渲染。阅读 [使用 Refs 引用值。](/learn/referencing-values-with-refs)

### 示例：滚动到某个元素 {/*example-scrolling-to-an-element*/}

在一个组件中你可以拥有多个 ref。在这个示例中，有一个包含三张图片的轮播。每个按钮都会通过调用浏览器的 [`scrollIntoView()`](https://developer.mozilla.org/en-US/docs/Web/API/Element/scrollIntoView) 方法，使对应的 DOM 节点居中：

<Sandpack>

```js
import { useRef } from 'react';

export default function CatFriends() {
  const firstCatRef = useRef(null);
  const secondCatRef = useRef(null);
  const thirdCatRef = useRef(null);

  function handleScrollToFirstCat() {
    firstCatRef.current.scrollIntoView({
      behavior: 'smooth',
      block: 'nearest',
      inline: 'center'
    });
  }

  function handleScrollToSecondCat() {
    secondCatRef.current.scrollIntoView({
      behavior: 'smooth',
      block: 'nearest',
      inline: 'center'
    });
  }

  function handleScrollToThirdCat() {
    thirdCatRef.current.scrollIntoView({
      behavior: 'smooth',
      block: 'nearest',
      inline: 'center'
    });
  }

  return (
    <>
      <nav>
        <button onClick={handleScrollToFirstCat}>
          Neo
        </button>
        <button onClick={handleScrollToSecondCat}>
          Millie
        </button>
        <button onClick={handleScrollToThirdCat}>
          Bella
        </button>
      </nav>
      <div>
        <ul>
          <li>
            <img
              src="https://placecats.com/neo/300/200"
              alt="Neo"
              ref={firstCatRef}
            />
          </li>
          <li>
            <img
              src="https://placecats.com/millie/200/200"
              alt="Millie"
              ref={secondCatRef}
            />
          </li>
          <li>
            <img
              src="https://placecats.com/bella/199/200"
              alt="Bella"
              ref={thirdCatRef}
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

<DeepDive>

#### 如何使用 ref 回调管理 ref 列表 {/*how-to-manage-a-list-of-refs-using-a-ref-callback*/}

在上面的示例中，ref 的数量是预先定义好的。然而，有时你可能需要为列表中的每一项都提供一个 ref，而你并不知道会有多少项。像下面这样 **不会起作用**：

```js
<ul>
  {items.map((item) => {
    // 不起作用！
    const ref = useRef(null);
    return <li ref={ref} />;
  })}
</ul>
```

这是因为 **Hooks 只能在组件顶层调用。** 你不能在循环中、条件语句中，或者 `map()` 调用内部调用 `useRef`。

一种可能的解决办法是，只获取其父元素的一个 ref，然后使用像 [`querySelectorAll`](https://developer.mozilla.org/en-US/docs/Web/API/Document/querySelectorAll) 这样的 DOM 操作方法，从中“查找”各个子节点。不过，这种方法很脆弱，如果 DOM 结构发生变化就可能失效。

另一个解决方案是 **向 `ref` 属性传入一个函数。** 这称为 [`ref` 回调。](/reference/react-dom/components/common#ref-callback) React 会在需要设置 ref 时用 DOM 节点调用你的 ref 回调，并在需要清除它时调用回调返回的清理函数。这样你就可以维护自己的数组或一个 [Map](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map)，并通过索引或某种 ID 访问任意 ref。

这个示例展示了如何使用这种方法滚动到长列表中的任意节点：

<Sandpack>

```js
import { useRef, useState } from "react";

export default function CatFriends() {
  const itemsRef = useRef(null);
  const [catList, setCatList] = useState(setupCatList);

  function scrollToCat(cat) {
    const map = getMap();
    const node = map.get(cat);
    node.scrollIntoView({
      behavior: "smooth",
      block: "nearest",
      inline: "center",
    });
  }

  function getMap() {
    if (!itemsRef.current) {
      // 首次使用时初始化 Map。
      itemsRef.current = new Map();
    }
    return itemsRef.current;
  }

  return (
    <>
      <nav>
        <button onClick={() => scrollToCat(catList[0])}>Neo</button>
        <button onClick={() => scrollToCat(catList[5])}>Millie</button>
        <button onClick={() => scrollToCat(catList[8])}>Bella</button>
      </nav>
      <div>
        <ul>
          {catList.map((cat) => (
            <li
              key={cat.id}
              ref={(node) => {
                const map = getMap();
                map.set(cat, node);

                return () => {
                  map.delete(cat);
                };
              }}
            >
              <img src={cat.imageUrl} />
            </li>
          ))}
        </ul>
      </div>
    </>
  );
}

function setupCatList() {
  const catCount = 10;
  const catList = new Array(catCount)
  for (let i = 0; i < catCount; i++) {
    let imageUrl = '';
    if (i < 5) {
      imageUrl = "https://placecats.com/neo/320/240";
    } else if (i < 8) {
      imageUrl = "https://placecats.com/millie/320/240";
    } else {
      imageUrl = "https://placecats.com/bella/320/240";
    }
    catList[i] = {
      id: i,
      imageUrl,
    };
  }
  return catList;
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

在这个示例中，`itemsRef` 并不保存单个 DOM 节点。相反，它保存的是一个从项目 ID 到 DOM 节点的 [Map](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Map)。([Refs 可以保存任何值！](/learn/referencing-values-with-refs)) 每个列表项上的 [`ref` 回调](/reference/react-dom/components/common#ref-callback) 负责更新这个 Map：

```js
<li
  key={cat.id}
  ref={node => {
    const map = getMap();
    // 添加到 Map
    map.set(cat, node);

    return () => {
      // 从 Map 中移除
      map.delete(cat);
    };
  }}
>
```

这样你之后就可以从 Map 中读取单独的 DOM 节点。

<Note>

当启用 Strict Mode 时，ref 回调在开发环境中会执行两次。

阅读关于 [这如何帮助发现 bug](/reference/react/StrictMode#fixing-bugs-found-by-re-running-ref-callbacks-in-development) 的更多内容，了解回调 refs。

</Note>

</DeepDive>

## 访问另一个组件的 DOM 节点 {/*accessing-another-components-dom-nodes*/}

<Pitfall>
Refs 是一种逃生舱。手动操作_另一个_组件的 DOM 节点可能会让你的代码变得脆弱。
</Pitfall>

你可以像传递其他任何 prop 一样，将 refs 从父组件传递给子组件 [。](/learn/passing-props-to-a-component)

```js {3-4,9}
import { useRef } from 'react';

function MyInput({ ref }) {
  return <input ref={ref} />;
}

function MyForm() {
  const inputRef = useRef(null);
  return <MyInput ref={inputRef} />
}
```

在上面的示例中，ref 在父组件 `MyForm` 中创建，并传递给子组件 `MyInput`。然后 `MyInput` 再把这个 ref 传给 `<input>`。因为 `<input>` 是一个 [内置组件](/reference/react-dom/components/common)，React 会将 ref 的 `.current` 属性设置为 `<input>` 的 DOM 元素。

现在，`MyForm` 中创建的 `inputRef` 指向了 `MyInput` 返回的 `<input>` DOM 元素。`MyForm` 中创建的点击处理程序可以访问 `inputRef`，并调用 `focus()` 来将焦点设置到 `<input>` 上。

<Sandpack>

```js
import { useRef } from 'react';

function MyInput({ ref }) {
  return <input ref={ref} />;
}

export default function MyForm() {
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

<DeepDive>

#### 使用 imperative handle 暴露 API 的一部分 {/*exposing-a-subset-of-the-api-with-an-imperative-handle*/}

在上面的示例中，传给 `MyInput` 的 ref 会继续传递给原始的 DOM input 元素。这样父组件就可以对其调用 `focus()`。然而，这也会让父组件做其他事情——例如，修改它的 CSS 样式。在一些不常见的情况下，你可能希望限制暴露出去的功能。你可以使用 [`useImperativeHandle`](/reference/react/useImperativeHandle) 来做到这一点：

<Sandpack>

```js
import { useRef, useImperativeHandle } from "react";

function MyInput({ ref }) {
  const realInputRef = useRef(null);
  useImperativeHandle(ref, () => ({
    // 只暴露 focus，不暴露其他任何内容
    focus() {
      realInputRef.current.focus();
    },
  }));
  return <input ref={realInputRef} />;
};

export default function Form() {
  const inputRef = useRef(null);

  function handleClick() {
    inputRef.current.focus();
  }

  return (
    <>
      <MyInput ref={inputRef} />
      <button onClick={handleClick}>聚焦输入框</button>
    </>
  );
}
```

</Sandpack>

这里，`MyInput` 内部的 `realInputRef` 持有的是实际的 input DOM 节点。然而，[`useImperativeHandle`](/reference/react/useImperativeHandle) 会指示 React 向父组件提供你自定义的特殊对象，作为 ref 的值。因此，`Form` 组件内部的 `inputRef.current` 将只拥有 `focus` 方法。在这种情况下，ref 的“句柄”不是 DOM 节点，而是你在调用 [`useImperativeHandle`](/reference/react/useImperativeHandle) 时创建的自定义对象。

</DeepDive>

## React 何时附加 refs {/*when-react-attaches-the-refs*/}

在 React 中，每次更新都会被拆分为[两个阶段](/learn/render-and-commit#step-3-react-commits-changes-to-the-dom)：

* 在**渲染**期间，React 会调用你的组件来决定屏幕上应该显示什么。
* 在**提交**期间，React 会将更改应用到 DOM。

通常，你[不应该](/learn/referencing-values-with-refs#best-practices-for-refs)在渲染期间访问 refs。对于持有 DOM 节点的 refs 也是如此。在第一次渲染时，DOM 节点还没有创建，所以 `ref.current` 会是 `null`。而在更新的渲染过程中，DOM 节点还没有被更新。因此此时读取它们还为时过早。

React 会在提交阶段设置 `ref.current`。在更新 DOM 之前，React 会把受影响的 `ref.current` 值设为 `null`。在更新 DOM 之后，React 会立即将它们设置为对应的 DOM 节点。

**通常，你会在事件处理器中访问 refs。** 如果你想用 ref 做些事情，但没有合适的事件可以触发，那你可能需要使用 Effect。我们会在后面的页面讨论 Effect。

<DeepDive>

#### 使用 flushSync 同步刷新状态更新 {/*flushing-state-updates-synchronously-with-flush-sync*/}

考虑如下代码，它会添加一个新的待办事项，并将屏幕滚动到列表的最后一个子项。注意，不知为何，它总是会滚动到那个**刚好在最后新增项之前**的待办事项：

<Sandpack>

```js
import { useState, useRef } from 'react';

export default function TodoList() {
  const listRef = useRef(null);
  const [text, setText] = useState('');
  const [todos, setTodos] = useState(
    initialTodos
  );

  function handleAdd() {
    const newTodo = { id: nextId++, text: text };
    setText('');
    setTodos([ ...todos, newTodo]);
    listRef.current.lastChild.scrollIntoView({
      behavior: 'smooth',
      block: 'nearest'
    });
  }

  return (
    <>
      <button onClick={handleAdd}>
        Add
      </button>
      <input
        value={text}
        onChange={e => setText(e.target.value)}
      />
      <ul ref={listRef}>
        {todos.map(todo => (
          <li key={todo.id}>{todo.text}</li>
        ))}
      </ul>
    </>
  );
}

let nextId = 0;
let initialTodos = [];
for (let i = 0; i < 20; i++) {
  initialTodos.push({
    id: nextId++,
    text: 'Todo #' + (i + 1)
  });
}
```

</Sandpack>

问题出在这两行：

```js
setTodos([ ...todos, newTodo]);
listRef.current.lastChild.scrollIntoView();
```

在 React 中，[状态更新会被排队。](/learn/queueing-a-series-of-state-updates)通常这正是你想要的。不过，这里它会导致问题，因为 `setTodos` 不会立即更新 DOM。所以当你把列表滚动到最后一个元素时，待办事项还没有被添加进去。这就是为什么滚动总会“落后”一个条目。

要修复这个问题，你可以强制 React 同步更新（“刷新”）DOM。为此，从 `react-dom` 导入 `flushSync`，并将**状态更新包裹起来**，放进一次 `flushSync` 调用中：

```js
flushSync(() => {
  setTodos([ ...todos, newTodo]);
});
listRef.current.lastChild.scrollIntoView();
```

这样会指示 React 在 `flushSync` 包裹的代码执行完后立即同步更新 DOM。结果就是，当你尝试滚动到最后一个待办事项时，它已经在 DOM 里了：

<Sandpack>

```js
import { useState, useRef } from 'react';
import { flushSync } from 'react-dom';

export default function TodoList() {
  const listRef = useRef(null);
  const [text, setText] = useState('');
  const [todos, setTodos] = useState(
    initialTodos
  );

  function handleAdd() {
    const newTodo = { id: nextId++, text: text };
    flushSync(() => {
      setText('');
      setTodos([ ...todos, newTodo]);
    });
    listRef.current.lastChild.scrollIntoView({
      behavior: 'smooth',
      block: 'nearest'
    });
  }

  return (
    <>
      <button onClick={handleAdd}>
        Add
      </button>
      <input
        value={text}
        onChange={e => setText(e.target.value)}
      />
      <ul ref={listRef}>
        {todos.map(todo => (
          <li key={todo.id}>{todo.text}</li>
        ))}
      </ul>
    </>
  );
}

let nextId = 0;
let initialTodos = [];
for (let i = 0; i < 20; i++) {
  initialTodos.push({
    id: nextId++,
    text: 'Todo #' + (i + 1)
  });
}
```

</Sandpack>

</DeepDive>

## 使用 refs 进行 DOM 操作的最佳实践 {/*best-practices-for-dom-manipulation-with-refs*/}

Refs 是一个逃生舱。你应该只在必须“跳出 React”时使用它们。常见例子包括管理焦点、滚动位置，或调用 React 没有暴露的浏览器 API。

如果你只做像聚焦和滚动这类非破坏性的操作，通常不会遇到问题。不过，如果你尝试手动**修改** DOM，就可能会和 React 正在进行的更改发生冲突。

为了说明这个问题，这个示例包含一条欢迎消息和两个按钮。第一个按钮像你在 React 中通常会做的那样，使用[条件渲染](/learn/conditional-rendering)和[状态](/learn/state-a-components-memory)来切换它是否显示。第二个按钮使用 [`remove()` DOM API](https://developer.mozilla.org/en-US/docs/Web/API/Element/remove) 在 React 控制之外强行将它从 DOM 中移除。

试着按几次“Toggle with setState”。消息应该会消失并再次出现。然后按“Remove from the DOM”。这会强行将它移除。最后，再按一次“Toggle with setState”：

<Sandpack>

```js
import { useState, useRef } from 'react';

export default function Counter() {
  const [show, setShow] = useState(true);
  const ref = useRef(null);

  return (
    <div>
      <button
        onClick={() => {
          setShow(!show);
        }}>
        Toggle with setState
      </button>
      <button
        onClick={() => {
          ref.current.remove();
        }}>
        Remove from the DOM
      </button>
      {show && <p ref={ref}>Hello world</p>}
    </div>
  );
}
```

```css
p,
button {
  display: block;
  margin: 10px;
}
```

</Sandpack>

在你手动移除了 DOM 元素之后，再尝试使用 `setState` 将其显示出来会导致崩溃。这是因为你已经更改了 DOM，而 React 不知道如何正确地继续管理它。

**避免更改由 React 管理的 DOM 节点。** 修改、向其添加子元素，或从由 React 管理的元素中移除子元素，可能会导致不一致的视觉结果，或像上面那样的崩溃。

不过，这并不意味着你完全不能这么做。只是需要谨慎。**你可以安全地修改 React _没有理由_ 去更新的 DOM 部分。** 例如，如果某个 `<div>` 在 JSX 中始终是空的，那么 React 就没有理由去碰它的子元素列表。因此，在那里手动添加或移除元素是安全的。

<Recap>

- refs 是一个通用概念，但最常见的用途是保存 DOM 元素。
- 你可以通过传递 `<div ref={myRef}>` 来告诉 React 把一个 DOM 节点放入 `myRef.current`。
- 通常，你会把 refs 用于非破坏性的操作，比如聚焦、滚动，或测量 DOM 元素。
- 组件默认不会暴露它的 DOM 节点。你可以通过使用 `ref` prop 来选择暴露一个 DOM 节点。
- 避免更改由 React 管理的 DOM 节点。
- 如果你确实修改了由 React 管理的 DOM 节点，请修改那些 React 没有理由去更新的部分。

</Recap>



<Challenges>

#### 播放和暂停视频 {/*play-and-pause-the-video*/}

在这个示例中，按钮会切换一个状态变量，在播放和暂停状态之间切换。不过，要真正播放或暂停视频，仅仅切换状态还不够。你还需要在 `<video>` 的 DOM 元素上调用 [`play()`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLMediaElement/play) 和 [`pause()`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLMediaElement/pause)。给它添加一个 ref，并让按钮工作起来。

<Sandpack>

```js
import { useState, useRef } from 'react';

export default function VideoPlayer() {
  const [isPlaying, setIsPlaying] = useState(false);

  function handleClick() {
    const nextIsPlaying = !isPlaying;
    setIsPlaying(nextIsPlaying);
  }

  return (
    <>
      <button onClick={handleClick}>
        {isPlaying ? 'Pause' : 'Play'}
      </button>
      <video width="250">
        <source
          src="https://interactive-examples.mdn.mozilla.net/media/cc0-videos/flower.mp4"
          type="video/mp4"
        />
      </video>
    </>
  )
}
```

```css
button { display: block; margin-bottom: 20px; }
```

</Sandpack>

额外挑战：即使用户右键点击视频，并使用浏览器内置的媒体控制播放视频，也要让“Play”按钮与视频是否正在播放保持同步。你可能需要监听视频的 `onPlay` 和 `onPause` 来做到这一点。

<Solution>

声明一个 ref，并把它放到 `<video>` 元素上。然后在事件处理器中根据下一个状态调用 `ref.current.play()` 和 `ref.current.pause()`。

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
        {isPlaying ? 'Pause' : 'Play'}
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
  )
}
```

```css
button { display: block; margin-bottom: 20px; }
```

</Sandpack>

为了处理浏览器内置控制，你可以给 `<video>` 元素添加 `onPlay` 和 `onPause` 处理器，并在其中调用 `setIsPlaying`。这样，如果用户通过浏览器控制播放视频，状态也会相应调整。

</Solution>

#### 聚焦搜索框 {/*focus-the-search-field*/}

让点击“Search”按钮时，焦点进入输入框。

<Sandpack>

```js
export default function Page() {
  return (
    <>
      <nav>
        <button>Search</button>
      </nav>
      <input
        placeholder="Looking for something?"
      />
    </>
  );
}
```

```css
button { display: block; margin-bottom: 10px; }
```

</Sandpack>

<Solution>

给输入框添加一个 ref，并在 DOM 节点上调用 `focus()` 以聚焦它：

<Sandpack>

```js
import { useRef } from 'react';

export default function Page() {
  const inputRef = useRef(null);
  return (
    <>
      <nav>
        <button onClick={() => {
          inputRef.current.focus();
        }}>
          Search
        </button>
      </nav>
      <input
        ref={inputRef}
        placeholder="Looking for something?"
      />
    </>
  );
}
```

```css
button { display: block; margin-bottom: 10px; }
```

</Sandpack>

</Solution>

#### 滚动图片轮播 {/*scrolling-an-image-carousel*/}

这个图片轮播有一个“Next”按钮，用来切换当前激活的图片。让画廊在点击时水平滚动到当前激活的图片。你需要对当前激活图片的 DOM 节点调用 [`scrollIntoView()`](https://developer.mozilla.org/en-US/docs/Web/API/Element/scrollIntoView)：

```js
node.scrollIntoView({
  behavior: 'smooth',
  block: 'nearest',
  inline: 'center'
});
```

<Hint>

这个练习不需要为每张图片都持有一个 ref。只需为当前激活的图片，或者为列表本身持有一个 ref 就够了。使用 `flushSync` 来确保在滚动之前 DOM 已经更新。

</Hint>

<Sandpack>

```js
import { useState } from 'react';

export default function CatFriends() {
  const [index, setIndex] = useState(0);
  return (
    <>
      <nav>
        <button onClick={() => {
          if (index < catList.length - 1) {
            setIndex(index + 1);
          } else {
            setIndex(0);
          }
        }}>
          Next
        </button>
      </nav>
      <div>
        <ul>
          {catList.map((cat, i) => (
            <li key={cat.id}>
              <img
                className={
                  index === i ?
                    'active' :
                    ''
                }
                src={cat.imageUrl}
                alt={'Cat #' + cat.id}
              />
            </li>
          ))}
        </ul>
      </div>
    </>
  );
}

const catCount = 10;
const catList = new Array(catCount);
for (let i = 0; i < catCount; i++) {
  const bucket = Math.floor(Math.random() * catCount) % 2;
  let imageUrl = '';
  switch (bucket) {
    case 0: {
      imageUrl = "https://placecats.com/neo/250/200";
      break;
    }
    case 1: {
      imageUrl = "https://placecats.com/millie/250/200";
      break;
    }
    case 2:
    default: {
      imageUrl = "https://placecats.com/bella/250/200";
      break;
    }
  }
  catList[i] = {
    id: i,
    imageUrl,
  };
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

img {
  padding: 10px;
  margin: -10px;
  transition: background 0.2s linear;
}

.active {
  background: rgba(0, 100, 150, 0.4);
}
```

</Sandpack>

<Solution>

你可以声明一个 `selectedRef`，然后只在当前图片上有条件地把它传进去：

```js
<li ref={index === i ? selectedRef : null}>
```

当 `index === i` 时，也就是该图片是被选中的图片时，`<li>` 会接收 `selectedRef`。React 会确保 `selectedRef.current` 始终指向正确的 DOM 节点。

请注意，`flushSync` 调用是必要的，因为它可以强制 React 在滚动之前更新 DOM。否则，`selectedRef.current` 总是会指向之前选中的那个项目。

<Sandpack>

```js
import { useRef, useState } from 'react';
import { flushSync } from 'react-dom';

export default function CatFriends() {
  const selectedRef = useRef(null);
  const [index, setIndex] = useState(0);

  return (
    <>
      <nav>
        <button onClick={() => {
          flushSync(() => {
            if (index < catList.length - 1) {
              setIndex(index + 1);
            } else {
              setIndex(0);
            }
          });
          selectedRef.current.scrollIntoView({
            behavior: 'smooth',
            block: 'nearest',
            inline: 'center'
          });
        }}>
          Next
        </button>
      </nav>
      <div>
        <ul>
          {catList.map((cat, i) => (
            <li
              key={cat.id}
              ref={index === i ?
                selectedRef :
                null
              }
            >
              <img
                className={
                  index === i ?
                    'active'
                    : ''
                }
                src={cat.imageUrl}
                alt={'Cat #' + cat.id}
              />
            </li>
          ))}
        </ul>
      </div>
    </>
  );
}

const catCount = 10;
const catList = new Array(catCount);
for (let i = 0; i < catCount; i++) {
  const bucket = Math.floor(Math.random() * catCount) % 2;
  let imageUrl = '';
  switch (bucket) {
    case 0: {
      imageUrl = "https://placecats.com/neo/250/200";
      break;
    }
    case 1: {
      imageUrl = "https://placecats.com/millie/250/200";
      break;
    }
    case 2:
    default: {
      imageUrl = "https://placecats.com/bella/250/200";
      break;
    }
  }
  catList[i] = {
    id: i,
    imageUrl,
  };
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

img {
  padding: 10px;
  margin: -10px;
  transition: background 0.2s linear;
}

.active {
  background: rgba(0, 100, 150, 0.4);
}
```

</Sandpack>

</Solution>

#### 使用分离组件聚焦搜索框 {/*focus-the-search-field-with-separate-components*/}

让点击“Search”按钮时，焦点进入输入框。注意，每个组件都定义在单独的文件中，而且不应该把它们移出原文件。你要如何把它们连接起来？

<Hint>

你需要将 `ref` 作为 prop 传递，才能选择从你自己的组件中暴露一个 DOM 节点，比如 `SearchInput`。

</Hint>

<Sandpack>

```js src/App.js
import SearchButton from './SearchButton.js';
import SearchInput from './SearchInput.js';

export default function Page() {
  return (
    <>
      <nav>
        <SearchButton />
      </nav>
      <SearchInput />
    </>
  );
}
```

```js src/SearchButton.js
export default function SearchButton() {
  return (
    <button>
      Search
    </button>
  );
}
```

```js src/SearchInput.js
export default function SearchInput() {
  return (
    <input
      placeholder="Looking for something?"
    />
  );
}
```

```css
button { display: block; margin-bottom: 10px; }
```

</Sandpack>

<Solution>

你需要给 `SearchButton` 添加一个 `onClick` prop，并让 `SearchButton` 将它传递给浏览器的 `<button>`。你还需要把一个 ref 传给 `<SearchInput>`，它会把这个 ref 转发到真正的 `<input>` 上并填充它。最后，在点击处理器中，你会对保存在该 ref 里的 DOM 节点调用 `focus`。

<Sandpack>

```js src/App.js
import { useRef } from 'react';
import SearchButton from './SearchButton.js';
import SearchInput from './SearchInput.js';

export default function Page() {
  const inputRef = useRef(null);
  return (
    <>
      <nav>
        <SearchButton onClick={() => {
          inputRef.current.focus();
        }} />
      </nav>
      <SearchInput ref={inputRef} />
    </>
  );
}
```

```js src/SearchButton.js
export default function SearchButton({ onClick }) {
  return (
    <button onClick={onClick}>
      Search
    </button>
  );
}
```

```js src/SearchInput.js
export default function SearchInput({ ref }) {
  return (
    <input
      ref={ref}
      placeholder="Looking for something?"
    />
  );
}
```

```css
button { display: block; margin-bottom: 10px; }
```

</Sandpack>

</Solution>

</Challenges>
