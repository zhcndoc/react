---
title: '使用 Effects 进行同步'
---

<Intro>

有些组件需要与外部系统同步。例如，你可能想基于 React state 控制一个非 React 组件，建立服务器连接，或者在组件出现在屏幕上时发送一条分析日志。*Effects* 让你可以在渲染之后运行一些代码，从而使你的组件与 React 之外的某个系统保持同步。

</Intro>

<YouWillLearn>

- 什么是 Effects
- Effects 与事件有何不同
- 如何在组件中声明一个 Effect
- 如何避免不必要地重新运行 Effect
- 为什么 Effects 在开发环境中会运行两次，以及如何修复

</YouWillLearn>

## 什么是 Effects，它们与事件有何不同？ {/*what-are-effects-and-how-are-they-different-from-events*/}

在了解 Effects 之前，你需要先熟悉 React 组件内部的两类逻辑：

- **渲染代码**（在 [描述 UI](/learn/describing-the-ui) 中介绍）位于组件的顶层。这是你获取 props 和 state、对它们进行转换并返回你想在屏幕上看到的 JSX 的地方。[渲染代码必须是纯的。](/learn/keeping-components-pure) 就像数学公式一样，它应该只会 _计算_ 结果，而不做其他事情。

- **事件处理函数**（在 [添加交互](/learn/adding-interactivity) 中介绍）是组件内部嵌套的函数，它们会*做*一些事情，而不仅仅是计算。事件处理函数可能会更新输入框、提交一个用于购买商品的 HTTP POST 请求，或者把用户导航到另一个界面。事件处理函数包含由特定用户操作（例如按钮点击或输入）引起的 ["副作用"](https://en.wikipedia.org/wiki/Side_effect_(computer_science))（它们会改变程序的状态）。

有时这还不够。考虑一个 `ChatRoom` 组件：它在屏幕上可见时必须连接到聊天服务器。连接到服务器不是一个纯计算（它是一个副作用），所以它不能在渲染期间发生。然而，并没有某个像点击这样明确的事件会让 `ChatRoom` 显示出来。

***Effects* 让你可以指定由渲染本身而不是某个特定事件引起的副作用。** 在聊天中发送消息是一个 *事件*，因为它直接由用户点击某个特定按钮触发。然而，建立服务器连接是一个 *Effect*，因为无论是什么交互让组件出现，它都应该发生。Effects 会在屏幕更新后的 [提交](/learn/render-and-commit) 结束时运行。这是将 React 组件与某个外部系统（如网络或第三方库）同步的好时机。

<Note>

在这里以及后文中，首字母大写的 "Effect" 指的是上面所定义的 React 特有概念，也就是由渲染引起的副作用。若要指更广义的编程概念，我们会说 "side effect"。

</Note>


## 你可能不需要 Effect {/*you-might-not-need-an-effect*/}

**不要急着给组件添加 Effects。** 请记住，Effects 通常用于“走出”你的 React 代码，并与某个 *外部* 系统同步。这包括浏览器 API、第三方组件、网络等等。如果你的 Effect 只是根据其他 state 来调整某些 state，[你可能不需要 Effect。](/learn/you-might-not-need-an-effect)

## 如何编写一个 Effect {/*how-to-write-an-effect*/}

要编写一个 Effect，请遵循以下三个步骤：

1. **声明一个 Effect。** 默认情况下，你的 Effect 会在每次 [提交](/learn/render-and-commit) 后运行。
2. **指定 Effect 的依赖项。** 大多数 Effects 只应在*需要时*重新运行，而不是在每次渲染后都运行。例如，一个淡入动画只应在组件出现时触发。连接和断开聊天房间也只应在组件出现和消失时，或者聊天房间变化时发生。你将学习如何通过指定*依赖项*来控制这一点。
3. **如有需要，添加清理。** 有些 Effects 需要说明如何停止、撤销或清理它们所做的事情。例如，“connect” 需要 “disconnect”，“subscribe” 需要 “unsubscribe”，“fetch” 需要 “cancel” 或 “ignore”。你将学习如何通过返回一个 *cleanup function* 来实现这一点。

让我们详细看看这三个步骤。

### 步骤 1：声明一个 Effect {/*step-1-declare-an-effect*/}

要在组件中声明一个 Effect，请从 React 中导入 [`useEffect` Hook](/reference/react/useEffect)：

```js
import { useEffect } from 'react';
```

然后，在组件顶层调用它，并在 Effect 中放入一些代码：

```js {2-4}
function MyComponent() {
  useEffect(() => {
    // 这里的代码会在*每次*渲染后运行
  });
  return <div />;
}
```

每次组件渲染时，React 会先更新屏幕，*然后*运行 `useEffect` 内的代码。换句话说，**`useEffect` 会将一段代码“延后”到该次渲染已反映到屏幕上之后再执行。**

让我们看看如何使用 Effect 与外部系统同步。考虑一个 `<VideoPlayer>` React 组件。通过向它传入 `isPlaying` prop 来控制它播放还是暂停会很不错：

```js
<VideoPlayer isPlaying={isPlaying} />;
```

你的自定义 `VideoPlayer` 组件会渲染内置的浏览器 [`<video>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/video) 标签：

```js
function VideoPlayer({ src, isPlaying }) {
  // TODO: 对 isPlaying 做些处理
  return <video src={src} />;
}
```

然而，浏览器的 `<video>` 标签并没有 `isPlaying` prop。控制它的唯一方式是手动调用 DOM 元素上的 [`play()`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLMediaElement/play) 和 [`pause()`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLMediaElement/pause) 方法。**你需要将 `isPlaying` prop 的值——它表示视频当前*应该*处于播放状态还是暂停状态——与 `play()` 和 `pause()` 之类的调用同步起来。**

我们首先需要 [获取一个 ref](/learn/manipulating-the-dom-with-refs) 指向 `<video>` 的 DOM 节点。

你可能会想在渲染期间调用 `play()` 或 `pause()`，但这是不正确的：

<Sandpack>

```js {expectedErrors: {'react-compiler': [7, 9]}}
import { useState, useRef, useEffect } from 'react';

function VideoPlayer({ src, isPlaying }) {
  const ref = useRef(null);

  if (isPlaying) {
    ref.current.play();  // 在渲染时调用这些方法是不允许的。
  } else {
    ref.current.pause(); // 而且这会崩溃。
  }

  return <video ref={ref} src={src} loop playsInline />;
}

export default function App() {
  const [isPlaying, setIsPlaying] = useState(false);
  return (
    <>
      <button onClick={() => setIsPlaying(!isPlaying)}>
        {isPlaying ? '暂停' : '播放'}
      </button>
      <VideoPlayer
        isPlaying={isPlaying}
        src="https://interactive-examples.mdn.mozilla.net/media/cc0-videos/flower.mp4"
      />
    </>
  );
}
```

```css
button { display: block; margin-bottom: 20px; }
video { width: 250px; }
```

</Sandpack>

这段代码不正确的原因在于，它试图在渲染期间对 DOM 节点做一些事情。在 React 中，[渲染应该是对 JSX 的纯计算](/learn/keeping-components-pure)，不应包含诸如修改 DOM 之类的副作用。

此外，当 `VideoPlayer` 第一次被调用时，它的 DOM 还不存在！此时还没有 DOM 节点可供调用 `play()` 或 `pause()`，因为在你返回 JSX 之前，React 并不知道需要创建什么 DOM。

这里的解决方案是：**使用 `useEffect` 包裹这个副作用，把它移出渲染计算：**

```js {6,12}
import { useEffect, useRef } from 'react';

function VideoPlayer({ src, isPlaying }) {
  const ref = useRef(null);

  useEffect(() => {
    if (isPlaying) {
      ref.current.play();
    } else {
      ref.current.pause();
    }
  });

  return <video ref={ref} src={src} loop playsInline />;
}
```

通过将 DOM 更新包裹在 Effect 中，你让 React 先更新屏幕。然后你的 Effect 再运行。

当 `VideoPlayer` 组件渲染时（无论是第一次还是重新渲染），会发生几件事。首先，React 会更新屏幕，确保 `<video>` 标签以正确的 props 出现在 DOM 中。然后 React 会运行你的 Effect。最后，你的 Effect 会根据 `isPlaying` 的值调用 `play()` 或 `pause()`。

多次按下播放/暂停，看看视频播放器如何始终与 `isPlaying` 的值保持同步：

<Sandpack>

```js
import { useState, useRef, useEffect } from 'react';

function VideoPlayer({ src, isPlaying }) {
  const ref = useRef(null);

  useEffect(() => {
    if (isPlaying) {
      ref.current.play();
    } else {
      ref.current.pause();
    }
  });

  return <video ref={ref} src={src} loop playsInline />;
}

export default function App() {
  const [isPlaying, setIsPlaying] = useState(false);
  return (
    <>
      <button onClick={() => setIsPlaying(!isPlaying)}>
        {isPlaying ? '暂停' : '播放'}
      </button>
      <VideoPlayer
        isPlaying={isPlaying}
        src="https://interactive-examples.mdn.mozilla.net/media/cc0-videos/flower.mp4"
      />
    </>
  );
}
```

```css
button { display: block; margin-bottom: 20px; }
video { width: 250px; }
```

</Sandpack>

在这个例子中，你同步到 React state 的“外部系统”是浏览器媒体 API。你可以用类似的方法把旧的非 React 代码（如 jQuery 插件）包装成声明式的 React 组件。

请注意，在实践中控制视频播放器要复杂得多。调用 `play()` 可能会失败，用户也可能通过浏览器自带的控件播放或暂停，等等。这个例子非常简化，并不完整。

<Pitfall>

默认情况下，Effects 会在*每次*渲染后运行。这就是为什么像下面这样的代码会**造成无限循环：**

```js
const [count, setCount] = useState(0);
useEffect(() => {
  setCount(count + 1);
});
```

Effects 是作为渲染的*结果*而运行的。设置 state 会*触发*渲染。在 Effect 中立即设置 state 就像把电源插座接回它自己。Effect 运行，它设置 state，这会导致重新渲染，从而导致 Effect 再次运行，它又设置 state，如此反复。

Effects 通常应该将组件与一个*外部*系统同步。如果没有外部系统，而你只是想根据其他 state 调整某些 state，[你可能不需要 Effect。](/learn/you-might-not-need-an-effect)

</Pitfall>

### 步骤 2：指定 Effect 的依赖项 {/*step-2-specify-the-effect-dependencies*/}

默认情况下，Effects 会在*每次*渲染后运行。通常，这**并不是**你想要的：

- 有时，这很慢。与外部系统同步并不总是瞬时完成的，所以你可能希望在没有必要时跳过它。例如，你不希望在每次按键时都重新连接聊天服务器。
- 有时，这会出错。例如，你不希望在每次按键时都触发组件的淡入动画。该动画只应在组件第一次出现时播放一次。

为了演示这个问题，下面是前一个例子，加入了一些 `console.log` 调用，以及一个会更新父组件 state 的文本输入框。注意输入时会导致 Effect 重新运行：

<Sandpack>

```js
import { useState, useRef, useEffect } from 'react';

function VideoPlayer({ src, isPlaying }) {
  const ref = useRef(null);

  useEffect(() => {
    if (isPlaying) {
      console.log('正在调用 video.play()');
      ref.current.play();
    } else {
      console.log('正在调用 video.pause()');
      ref.current.pause();
    }
  });

  return <video ref={ref} src={src} loop playsInline />;
}

export default function App() {
  const [isPlaying, setIsPlaying] = useState(false);
  const [text, setText] = useState('');
  return (
    <>
      <input value={text} onChange={e => setText(e.target.value)} />
      <button onClick={() => setIsPlaying(!isPlaying)}>
        {isPlaying ? '暂停' : '播放'}
      </button>
      <VideoPlayer
        isPlaying={isPlaying}
        src="https://interactive-examples.mdn.mozilla.net/media/cc0-videos/flower.mp4"
      />
    </>
  );
}
```

```css
input, button { display: block; margin-bottom: 20px; }
video { width: 250px; }
```

</Sandpack>

你可以通过在 `useEffect` 调用的第二个参数中指定一个 *依赖项* 数组，告诉 React **跳过不必要的 Effect 重新运行**。先在上面第 14 行的示例中添加一个空的 `[]` 数组：

```js {3}
  useEffect(() => {
    // ...
  }, []);
```

你应该会看到一个错误提示：`React Hook useEffect has a missing dependency: 'isPlaying'`：

<Sandpack>

```js
import { useState, useRef, useEffect } from 'react';

function VideoPlayer({ src, isPlaying }) {
  const ref = useRef(null);

  useEffect(() => {
    if (isPlaying) {
      console.log('正在调用 video.play()');
      ref.current.play();
    } else {
      console.log('正在调用 video.pause()');
      ref.current.pause();
    }
  }, []); // 这会导致错误

  return <video ref={ref} src={src} loop playsInline />;
}

export default function App() {
  const [isPlaying, setIsPlaying] = useState(false);
  const [text, setText] = useState('');
  return (
    <>
      <input value={text} onChange={e => setText(e.target.value)} />
      <button onClick={() => setIsPlaying(!isPlaying)}>
        {isPlaying ? '暂停' : '播放'}
      </button>
      <VideoPlayer
        isPlaying={isPlaying}
        src="https://interactive-examples.mdn.mozilla.net/media/cc0-videos/flower.mp4"
      />
    </>
  );
}
```

```css
input, button { display: block; margin-bottom: 20px; }
video { width: 250px; }
```

</Sandpack>

问题在于，你的 Effect 内部代码*依赖于* `isPlaying` prop 来决定要做什么，但这个依赖并没有被显式声明。要修复这个问题，请把 `isPlaying` 添加到依赖数组中：

```js {2,7}
  useEffect(() => {
    if (isPlaying) { // 它在这里被使用了...
      // ...
    } else {
      // ...
    }
  }, [isPlaying]); // ...所以它必须在这里被声明！
```

现在所有依赖都已声明，因此不会再报错。将 `[isPlaying]` 指定为依赖数组，会告诉 React：如果 `isPlaying` 与上一次渲染时相同，就跳过重新运行你的 Effect。这样修改后，输入框中的输入不会导致 Effect 重新运行，但按下播放/暂停会：

<Sandpack>

```js
import { useState, useRef, useEffect } from 'react';

function VideoPlayer({ src, isPlaying }) {
  const ref = useRef(null);

  useEffect(() => {
    if (isPlaying) {
      console.log('正在调用 video.play()');
      ref.current.play();
    } else {
      console.log('正在调用 video.pause()');
      ref.current.pause();
    }
  }, [isPlaying]);

  return <video ref={ref} src={src} loop playsInline />;
}

export default function App() {
  const [isPlaying, setIsPlaying] = useState(false);
  const [text, setText] = useState('');
  return (
    <>
      <input value={text} onChange={e => setText(e.target.value)} />
      <button onClick={() => setIsPlaying(!isPlaying)}>
        {isPlaying ? '暂停' : '播放'}
      </button>
      <VideoPlayer
        isPlaying={isPlaying}
        src="https://interactive-examples.mdn.mozilla.net/media/cc0-videos/flower.mp4"
      />
    </>
  );
}
```

```css
input, button { display: block; margin-bottom: 20px; }
video { width: 250px; }
```

</Sandpack>

依赖数组可以包含多个依赖项。只有当你指定的*所有*依赖项在值上都与上一次渲染时完全相同，React 才会跳过重新运行 Effect。React 使用 [`Object.is`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is) 比较依赖值。详情请参见 [`useEffect` 参考](/reference/react/useEffect#reference)。

**请注意，你不能“选择”你的依赖项。** 如果你指定的依赖项与 React 根据 Effect 内部代码所期望的不匹配，你会收到 lint 错误。这有助于发现代码中的许多 bug。如果你不想让某些代码重新运行，[请*修改 Effect 代码本身*，让它不再“需要”那个依赖。](/learn/lifecycle-of-reactive-effects#what-to-do-when-you-dont-want-to-re-synchronize)

<Pitfall>

没有依赖数组和使用一个*空的* `[]` 依赖数组时，行为是不同的：

```js {3,7,11}
useEffect(() => {
  // 这会在每次渲染后运行
});

useEffect(() => {
  // 这只会在挂载时运行（组件出现时）
}, []);

useEffect(() => {
  // 这会在挂载时运行，*并且*如果 a 或 b 自上一次渲染以来发生变化，也会运行
}, [a, b]);
```

我们将在下一步仔细看看“挂载”是什么意思。

</Pitfall>

<DeepDive>

#### 为什么 ref 被省略出依赖数组？ {/*why-was-the-ref-omitted-from-the-dependency-array*/}

这个 Effect 同时使用了 `ref` 和 `isPlaying`，但依赖项中只声明了 `isPlaying`：

```js {9}
function VideoPlayer({ src, isPlaying }) {
  const ref = useRef(null);
  useEffect(() => {
    if (isPlaying) {
      ref.current.play();
    } else {
      ref.current.pause();
    }
  }, [isPlaying]);
```

这是因为 `ref` 对象具有*稳定的身份*：React 保证你每次从同一个 `useRef` 调用中都会得到[同一个对象](/reference/react/useRef#returns)。它永远不会改变，因此它本身永远不会导致 Effect 重新运行。所以，是否把它包括进去都无所谓。把它包含进去也没问题：

```js {9}
function VideoPlayer({ src, isPlaying }) {
  const ref = useRef(null);
  useEffect(() => {
    if (isPlaying) {
      ref.current.play();
    } else {
      ref.current.pause();
    }
  }, [isPlaying, ref]);
```

[`set` 函数](/reference/react/useState#setstate) 也有由 `useState` 返回的稳定身份，因此你经常也会看到它们被省略出依赖项。如果 linter 允许你省略某个依赖而不报错，那这样做是安全的。

省略始终稳定的依赖项只有在 linter 能“看出”该对象是稳定时才有效。例如，如果 `ref` 是从父组件传入的，你就必须把它指定在依赖数组中。不过这其实是好事，因为你无法知道父组件总是传入同一个 ref，还是会有条件地传入多个 ref 中的某一个。所以你的 Effect 确实会依赖于传入的是哪个 ref。

</DeepDive>

### 步骤 3：如有需要，添加清理 {/*step-3-add-cleanup-if-needed*/}

考虑另一个例子。你正在编写一个 `ChatRoom` 组件，它需要在出现时连接到聊天服务器。你会得到一个 `createConnection()` API，它返回一个带有 `connect()` 和 `disconnect()` 方法的对象。如何让组件在展示给用户时始终保持连接？

先写出 Effect 逻辑：

```js
useEffect(() => {
  const connection = createConnection();
  connection.connect();
});
```

每次重新渲染后都连接到聊天服务器会很慢，因此你添加依赖数组：

```js {4}
useEffect(() => {
  const connection = createConnection();
  connection.connect();
}, []);
```

**Effect 内部代码没有使用任何 props 或 state，所以你的依赖数组是 `[]`（空的）。这告诉 React 只在组件“挂载”时运行这段代码，也就是组件第一次出现在屏幕上时。**

让我们尝试运行这段代码：

<Sandpack>

```js
import { useEffect } from 'react';
import { createConnection } from './chat.js';

export default function ChatRoom() {
  useEffect(() => {
    const connection = createConnection();
    connection.connect();
  }, []);
  return <h1>Welcome to the chat!</h1>;
}
```

```js src/chat.js
export function createConnection() {
  // 实际实现会真正连接到服务器
  return {
    connect() {
      console.log('✅ 正在连接...');
    },
    disconnect() {
      console.log('❌ 已断开连接。');
    }
  };
}
```

```css
input { display: block; margin-bottom: 20px; }
```

</Sandpack>

这个 Effect 只在挂载时运行，所以你可能会以为 `"✅ Connecting..."` 只会在控制台打印一次。**然而，如果你检查控制台，会发现 `"✅ Connecting..."` 被打印了两次。为什么会这样？**

想象 `ChatRoom` 组件是一个更大应用的一部分，而这个应用有许多不同的页面。用户从 `ChatRoom` 页面开始他们的旅程。组件挂载并调用 `connection.connect()`。然后设想用户导航到另一个页面——例如设置页面。`ChatRoom` 组件卸载。最后，用户点击返回，`ChatRoom` 再次挂载。这样就会建立第二个连接——但第一个连接从未被销毁！随着用户在应用中导航，连接会不断累积。

如果没有大量的手动测试，这类 bug 很容易被忽略。为了帮助你快速发现它们，在开发环境中，React 会在每个组件初次挂载后立即再卸载并重新挂载一次。

看到 `"✅ Connecting..."` 这条日志出现两次，可以帮助你注意到真正的问题：组件卸载时，你的代码没有关闭连接。

要修复这个问题，请从 Effect 中返回一个 *cleanup function*：

```js {4-6}
  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, []);
```

React 会在每次 Effect 再次运行之前调用你的 cleanup function，并且在组件卸载（被移除）时最后再调用一次。让我们看看实现了 cleanup function 之后会发生什么：

<Sandpack>

```js
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

export default function ChatRoom() {
  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    return () => connection.disconnect();
  }, []);
  return <h1>Welcome to the chat!</h1>;
}
```

```js src/chat.js
export function createConnection() {
  // 实际实现会真正连接到服务器
  return {
    connect() {
      console.log('✅ 正在连接...');
    },
    disconnect() {
      console.log('❌ 已断开连接。');
    }
  };
}
```

```css
input { display: block; margin-bottom: 20px; }
```

</Sandpack>

现在在开发环境中你会看到三条控制台日志：

1. `"✅ 正在连接..."`
2. `"❌ 已断开连接。"`
3. `"✅ 正在连接..."`

**这在开发环境中是正确行为。** 通过重新挂载你的组件，React 验证离开再返回页面不会破坏你的代码。断开连接然后重新连接，正是应该发生的事情！当你把 cleanup 实现好之后，Effect 只运行一次，与“运行一次、清理、再运行一次”之间不应该有任何用户可见的差异。之所以会多出一对连接/断开调用，是因为 React 在开发环境中会探测你的代码是否有 bug。这很正常——不要试图把它消除掉！

**在生产环境中，你只会看到 `"✅ 正在连接..."` 打印一次。** 组件重新挂载只会在开发环境中发生，以帮助你发现那些需要清理的 Effects。你可以关闭 [Strict Mode](/reference/react/StrictMode) 来退出这种开发行为，但我们建议保持开启。这样你就能发现许多类似上面这样的 bug。

## 在开发环境中如何处理 Effect 触发两次？ {/*how-to-handle-the-effect-firing-twice-in-development*/}

React 会在开发环境中故意重新挂载你的组件，以便发现上一示例中的这类 bug。**正确的问题不是“如何让 Effect 只运行一次”，而是“如何修复我的 Effect，使它在重新挂载后仍能正常工作”。**

通常，答案是实现清理函数。清理函数应该停止或撤销 Effect 所做的一切。一个经验法则是：用户不应能分辨出 Effect 只运行了一次（如生产环境中那样）和一个 _setup → cleanup → setup_ 序列（如你在开发环境中看到的那样）之间的区别。

你写的大多数 Effect 都会符合下面这些常见模式之一。

<Pitfall>

#### 不要使用 ref 来阻止 Effect 触发 {/*dont-use-refs-to-prevent-effects-from-firing*/}

在开发环境中防止 Effect 触发两次的一个常见误区，是使用 `ref` 来阻止 Effect 运行不止一次。例如，你可以用 `useRef` “修复”上面的 bug：

```js {1,3-4}
  const connectionRef = useRef(null);
  useEffect(() => {
    // 🚩 这并不能修复这个 bug！！！
    if (!connectionRef.current) {
      connectionRef.current = createConnection();
      connectionRef.current.connect();
    }
  }, []);
```

这会让你在开发环境中只看到一次 `"✅ Connecting..."`，但它并没有修复 bug。

当用户离开页面时，连接仍然不会关闭；当他们返回时，又会创建一个新的连接。随着用户在应用中不断切换，这些连接会继续累积，就和“修复”之前一样。

要修复这个 bug，仅仅让 Effect 只运行一次是不够的。这个 Effect 需要在重新挂载后也能正常工作，这意味着连接需要像上面的方案那样被清理掉。

有关如何处理常见模式，请看下面的示例。

</Pitfall>

### 控制非 React 组件 {/*controlling-non-react-widgets*/}

有时你需要添加一些不是用 React 编写的 UI 组件。例如，假设你要在页面中添加一个地图组件。它有一个 `setZoomLevel()` 方法，而你希望缩放级别与 React 代码中的 `zoomLevel` 状态变量保持同步。你的 Effect 会像这样：

```js
useEffect(() => {
  const map = mapRef.current;
  map.setZoomLevel(zoomLevel);
}, [zoomLevel]);
```

注意，这种情况下不需要清理函数。在开发环境中，React 会调用两次这个 Effect，但这不是问题，因为用同样的值调用两次 `setZoomLevel` 不会产生任何作用。它可能会稍微慢一点，但这无关紧要，因为在生产环境中它不会无谓地重新挂载。

某些 API 不允许你连续调用两次。例如，内置 [`<dialog>`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLDialogElement) 元素的 [`showModal`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLDialogElement/showModal) 方法，如果你调用两次就会抛出错误。实现清理函数，让它关闭对话框：

```js {4}
useEffect(() => {
  const dialog = dialogRef.current;
  dialog.showModal();
  return () => dialog.close();
}, []);
```

在开发环境中，你的 Effect 会先调用 `showModal()`，然后立即调用 `close()`，再调用一次 `showModal()`。这与在生产环境中只调用一次 `showModal()` 的用户可见行为是相同的。

### 订阅事件 {/*subscribing-to-events*/}

如果你的 Effect 订阅了某个内容，清理函数应该取消订阅：

```js {6}
useEffect(() => {
  function handleScroll(e) {
    console.log(window.scrollX, window.scrollY);
  }
  window.addEventListener('scroll', handleScroll);
  return () => window.removeEventListener('scroll', handleScroll);
}, []);
```

在开发环境中，你的 Effect 会先调用 `addEventListener()`，然后立即调用 `removeEventListener()`，再使用同一个处理函数再次调用 `addEventListener()`。因此任意时刻都只会有一个有效订阅。这与生产环境中只调用一次 `addEventListener()` 的用户可见行为是相同的。

### 触发动画 {/*triggering-animations*/}

如果你的 Effect 会让某些内容产生动画，那么清理函数应该将动画重置为初始值：

```js {4-6}
useEffect(() => {
  const node = ref.current;
  node.style.opacity = 1; // 触发动画
  return () => {
    node.style.opacity = 0; // 重置为初始值
  };
}, []);
```

在开发环境中，opacity 会被设置为 `1`，然后变为 `0`，再变回 `1`。这应该与直接将它设置为 `1` 的用户可见行为相同，而这正是生产环境中会发生的情况。如果你使用带有补间动画支持的第三方动画库，清理函数应该把时间线重置回初始状态。

### 获取数据 {/*fetching-data*/}

如果你的 Effect 会请求某些数据，清理函数应该[中止这次请求](https://developer.mozilla.org/en-US/docs/Web/API/AbortController)或者忽略其结果：

```js {2,6,13-15}
useEffect(() => {
  let ignore = false;

  async function startFetching() {
    const json = await fetchTodos(userId);
    if (!ignore) {
      setTodos(json);
    }
  }

  startFetching();

  return () => {
    ignore = true;
  };
}, [userId]);
```

你无法“撤销”一个已经发生的网络请求，但你的清理函数应该确保那些_不再相关_的请求不会继续影响你的应用。如果 `userId` 从 `'Alice'` 变为 `'Bob'`，清理过程会确保即使 `'Alice'` 的响应在 `'Bob'` 之后到达，也会被忽略。

**在开发环境中，你会在 Network 面板里看到两个请求。** 这没有任何问题。使用上面的方案时，第一个 Effect 会立即被清理，因此它对应的 `ignore` 变量会被设为 `true`。所以即使多发出了一次请求，也不会因为 `if (!ignore)` 检查而影响状态。

**在生产环境中，只会有一个请求。** 如果开发环境中的第二个请求让你感到困扰，最好的办法是使用一种可以对请求去重并在组件之间缓存响应的方案：

```js
function TodoList() {
  const todos = useSomeDataLibrary(`/api/user/${userId}/todos`);
  // ...
```

这不仅会改善开发体验，也会让你的应用感觉更快。例如，用户按下后退按钮时，不必再等待某些数据重新加载，因为它已经被缓存了。你可以自己构建这样的缓存，或者使用其他多种替代手动在 Effect 中获取数据的方案之一。

<DeepDive>

#### 在 Effect 中获取数据有哪些好的替代方案？ {/*what-are-good-alternatives-to-data-fetching-in-effects*/}

在 Effects 中编写 `fetch` 调用是一种[很流行的数据获取方式](https://www.robinwieruch.de/react-hooks-fetch-data/)，尤其是在完全客户端渲染的应用中。然而，这是一种非常手工的做法，并且有明显缺点：

- **Effects 不会在服务器上运行。** 这意味着初始的服务器渲染 HTML 只会包含一个没有数据的加载状态。客户端电脑必须下载所有 JavaScript，渲染你的应用，然后才发现现在还需要加载数据。这并不是很高效。
- **直接在 Effect 中获取数据很容易造成“网络瀑布”。** 你先渲染父组件，它获取一些数据，随后渲染子组件，然后子组件又开始获取自己的数据。如果网络不够快，这会比并行获取所有数据慢得多。
- **直接在 Effect 中获取数据通常意味着你不会预加载或缓存数据。** 例如，如果组件卸载后又重新挂载，就必须再次请求数据。
- **这并不太符合人体工学。** 以不会出现[竞态条件](https://maxrozen.com/race-conditions-fetching-data-react-with-useeffect)等 bug 的方式编写 `fetch` 调用，会涉及不少样板代码。

这份缺点列表并不只针对 React。它适用于任何库中在挂载时获取数据的做法。就像路由一样，做好数据获取并不简单，所以我们推荐以下方式：

- **如果你使用的是[框架](/learn/creating-a-react-app#full-stack-frameworks)，请使用它内置的数据获取机制。** 现代 React 框架都集成了高效的数据获取机制，并且不会受到上面这些陷阱的影响。
- **否则，考虑使用或构建一个客户端缓存。** 常见的开源方案包括 [TanStack Query](https://tanstack.com/query/latest)、[useSWR](https://swr.vercel.app/) 和 [React Router 6.4+](https://beta.reactrouter.com/en/main/start/overview)。你也可以自己构建解决方案，在这种情况下，底层同样会使用 Effect，但会额外加入请求去重、响应缓存，以及避免网络瀑布的逻辑（通过预加载数据或把数据需求提升到路由层）。

如果这两种方式都不适合你，你也可以继续直接在 Effect 中获取数据。

</DeepDive>

### 发送分析数据 {/*sending-analytics*/}

考虑这段在页面访问时发送分析事件的代码：

```js
useEffect(() => {
  logVisit(url); // 发送一个 POST 请求
}, [url]);
```

在开发环境中，每个 URL 的 `logVisit` 都会被调用两次，所以你可能会想要修复它。**我们建议保持这段代码不变。** 和前面的例子一样，运行一次和运行两次之间没有任何*用户可见*的行为差异。从实际角度来看，`logVisit` 在开发环境中不应该做任何事情，因为你不希望开发机器上的日志扭曲生产环境的指标。每次你保存文件时，组件都会重新挂载，所以它在开发环境中本来就会额外记录访问。

**在生产环境中，不会有重复的访问日志。**

如果你想调试发送出去的分析事件，可以把应用部署到预发布环境（以生产模式运行），或者临时关闭 [Strict Mode](/reference/react/StrictMode) 及其仅在开发环境下进行的重新挂载检查。你也可以从路由变化事件处理函数中发送分析数据，而不是通过 Effects。对于更精确的分析，[交叉观察器](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API)可以帮助你跟踪哪些组件位于视口中，以及它们保持可见的时间。

### 不是 Effect：初始化应用 {/*not-an-effect-initializing-the-application*/}

有些逻辑只应该在应用启动时运行一次。你可以把它放到组件外部：

```js {2-3}
if (typeof window !== 'undefined') { // 检查我们是否在浏览器中运行。
  checkAuthToken();
  loadDataFromLocalStorage();
}

function App() {
  // ...
}
```

这可以保证此类逻辑只会在浏览器加载页面后运行一次。

### 不是 Effect：购买商品 {/*not-an-effect-buying-a-product*/}

有时，即使你写了清理函数，也无法阻止 Effect 运行两次所带来的用户可见后果。例如，也许你的 Effect 会发送一个 POST 请求，比如购买商品：

```js {2-3}
useEffect(() => {
  // 🔴 错误：这个 Effect 在开发环境中会触发两次，暴露出代码中的问题。
  fetch('/api/buy', { method: 'POST' });
}, []);
```

你不会希望把商品买两次。然而，这也正是你不应该把这段逻辑放进 Effect 的原因。如果用户去了另一个页面，然后按下返回键呢？你的 Effect 还会再次运行。你不希望用户在*访问*某个页面时完成购买；你希望在用户*点击*购买按钮时完成购买。

购买不是由渲染引起的；它是由某个特定交互引起的。它应该只在用户按下按钮时运行。**删除这个 Effect，并把你的 `/api/buy` 请求移动到购买按钮的事件处理函数中：**

```js {2-3}
  function handleClick() {
    // ✅ 购买是一个事件，因为它是由某次特定交互引起的。
    fetch('/api/buy', { method: 'POST' });
  }
```

**这说明，如果重新挂载破坏了你应用的逻辑，这通常是在揭示已有的 bug。** 从用户的角度来看，访问一个页面、点击一个链接再按下返回键回到该页面，和直接访问该页面不应该有区别。React 通过在开发环境中对组件重新挂载一次，来验证你的组件是否遵循这一原则。

## 把它们组合起来 {/*putting-it-all-together*/}

这个示例可以帮助你在实践中“感受” Effects 的工作方式。

这个示例使用 [`setTimeout`](https://developer.mozilla.org/en-US/docs/Web/API/setTimeout) 来安排一次控制台日志输出：Effect 运行后三秒，显示输入框中的文本。清理函数会取消待处理的超时。先点击“挂载组件”开始：

<Sandpack>

```js
import { useState, useEffect } from 'react';

function Playground() {
  const [text, setText] = useState('a');

  useEffect(() => {
    function onTimeout() {
      console.log('⏰ ' + text);
    }

    console.log('🔵 安排 "' + text + '" 的日志');
    const timeoutId = setTimeout(onTimeout, 3000);

    return () => {
      console.log('🟡 取消 "' + text + '" 的日志');
      clearTimeout(timeoutId);
    };
  }, [text]);

  return (
    <>
      <label>
        要记录什么：{' '}
        <input
          value={text}
          onChange={e => setText(e.target.value)}
        />
      </label>
      <h1>{text}</h1>
    </>
  );
}

export default function App() {
  const [show, setShow] = useState(false);
  return (
    <>
      <button onClick={() => setShow(!show)}>
        {show ? '卸载' : '挂载'}组件
      </button>
      {show && <hr />}
      {show && <Playground />}
    </>
  );
}
```

</Sandpack>

你最开始会看到三条日志：`安排 "a" 的日志`、`取消 "a" 的日志`，然后 আবার `安排 "a" 的日志`。三秒后还会看到一条 `a` 的日志。正如你前面学到的，多出来的一组安排/取消，是因为 React 在开发环境中会重新挂载一次组件，以验证你是否正确实现了清理。

现在把输入内容改成 `abc`。如果你操作够快，你会立刻看到 `安排 "ab" 的日志`，紧接着是 `取消 "ab" 的日志` 和 `安排 "abc" 的日志`。**React 总是在下一次渲染的 Effect 之前，先清理上一次渲染的 Effect。** 这就是为什么即使你快速输入，任意时刻也最多只有一个 timeout 在等待执行。多试几次并观察控制台，感受一下 Effects 是如何被清理的。

在输入框里输入一些内容，然后立刻点击“卸载组件”。注意卸载会如何清理最后一次渲染的 Effect。这里，它会在最后一个 timeout 有机会触发之前把它清除掉。

最后，编辑上面的组件并注释掉清理函数，这样 timeout 就不会被取消。试着快速输入 `abcde`。你觉得三秒后会发生什么？`timeout` 内部的 `console.log(text)` 会打印 *最新的* `text` 并输出五条 `abcde` 日志吗？试试看，验证一下你的直觉！

三秒后，你应该会看到一串日志（`a`、`ab`、`abc`、`abcd` 和 `abcde`），而不是五条 `abcde` 日志。**每个 Effect 都会“捕获”其对应渲染中的 `text` 值。** `text` 状态变了并不重要：来自 `text = 'ab'` 那次渲染的 Effect 永远只会看到 `'ab'`。换句话说，每次渲染中的 Effects 都是彼此隔离的。如果你好奇这是怎么做到的，可以阅读 [闭包](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures) 相关内容。

<DeepDive>

#### 每次渲染都有自己的 Effects {/*each-render-has-its-own-effects*/}

你可以把 `useEffect` 看作是把一段行为“附加”到渲染输出上。考虑这个 Effect：

```js
export default function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(roomId);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]);

  return <h1>Welcome to {roomId}!</h1>;
}
```

让我们看看用户在应用中切换时，具体会发生什么。

#### 初始渲染 {/*initial-render*/}

用户访问 `<ChatRoom roomId="general" />`。让我们在脑中把 `roomId` 替换成 `'general'`：

```js
  // 第一次渲染的 JSX（roomId = "general"）
  return <h1>Welcome to general!</h1>;
```

**Effect 也是渲染输出的一部分。** 第一次渲染的 Effect 变成：

```js
  // 第一次渲染的 Effect（roomId = "general"）
  () => {
    const connection = createConnection('general');
    connection.connect();
    return () => connection.disconnect();
  },
  // 第一次渲染的依赖项（roomId = "general"）
  ['general']
```

React 运行这个 Effect，它会连接到 `'general'` 聊天室。

#### 使用相同依赖项重新渲染 {/*re-render-with-same-dependencies*/}

假设 `<ChatRoom roomId="general" />` 重新渲染了。JSX 输出是一样的：

```js
  // 第二次渲染的 JSX（roomId = "general"）
  return <h1>Welcome to general!</h1>;
```

React 看到渲染输出没有变化，因此不会更新 DOM。

第二次渲染的 Effect 看起来是这样：

```js
  // 第二次渲染的 Effect（roomId = "general"）
  () => {
    const connection = createConnection('general');
    connection.connect();
    return () => connection.disconnect();
  },
  // 第二次渲染的依赖项（roomId = "general"）
  ['general']
```

React 将第二次渲染中的 `['general']` 与第一次渲染中的 `['general']` 进行比较。**因为所有依赖项都相同，React 会 *忽略* 第二次渲染的 Effect。** 它根本不会被调用。

#### 使用不同依赖项重新渲染 {/*re-render-with-different-dependencies*/}

接着，用户访问 `<ChatRoom roomId="travel" />`。这一次，组件返回不同的 JSX：

```js
  // 第三次渲染的 JSX（roomId = "travel"）
  return <h1>Welcome to travel!</h1>;
```

React 更新 DOM，把 `"Welcome to general"` 改成 `"Welcome to travel"`。

第三次渲染的 Effect 看起来是这样：

```js
  // 第三次渲染的 Effect（roomId = "travel"）
  () => {
    const connection = createConnection('travel');
    connection.connect();
    return () => connection.disconnect();
  },
  // 第三次渲染的依赖项（roomId = "travel"）
  ['travel']
```

React 将第三次渲染中的 `['travel']` 与第二次渲染中的 `['general']` 进行比较。有一个依赖项不同：`Object.is('travel', 'general')` 为 `false`。这个 Effect 不能被跳过。

**在 React 应用第三次渲染的 Effect 之前，它需要先清理上一个 *确实运行过* 的 Effect。** 第二次渲染的 Effect 被跳过了，所以 React 需要清理第一次渲染的 Effect。如果你往上看第一次渲染，就会发现它的清理函数会对使用 `createConnection('general')` 创建的连接调用 `disconnect()`。这会让应用断开与 `'general'` 聊天室的连接。

之后，React 运行第三次渲染的 Effect。它会连接到 `'travel'` 聊天室。

#### 卸载 {/*unmount*/}

最后，假设用户离开页面，而 `ChatRoom` 组件被卸载。React 会运行最后一个 Effect 的清理函数。最后一个 Effect 来自第三次渲染。第三次渲染的清理函数会销毁 `createConnection('travel')` 连接。因此应用会断开与 `'travel'` 房间的连接。

#### 仅在开发环境中的行为 {/*development-only-behaviors*/}

当开启 [Strict Mode](/reference/react/StrictMode) 时，React 会在挂载后让每个组件重新挂载一次（state 和 DOM 会被保留）。这有助于你及早发现需要清理的 Effects，并暴露诸如竞态条件之类的 bug。此外，React 还会在你在开发环境中保存文件时重新挂载 Effects。这两种行为都只会出现在开发环境中。

</DeepDive>

<Recap>

- 与事件不同，Effects 是由渲染本身引起的，而不是由某个特定交互引起的。
- Effects 让你可以将组件与某个外部系统（第三方 API、网络等）同步起来。
- 默认情况下，Effects 会在每次渲染后运行（包括初始渲染）。
- 如果某个 Effect 的所有依赖项与上一次渲染时的值相同，React 会跳过它。
- 你不能“选择”你的依赖项。它们由 Effect 内部的代码决定。
- 空依赖数组（`[]`）对应组件的“挂载”，也就是被添加到屏幕上。
- 在 Strict Mode 中，React 会把组件挂载两次（仅在开发环境！），以对 Effects 进行压力测试。
- 如果你的 Effect 因为重新挂载而出错，你需要实现一个清理函数。
- React 会在下次运行 Effect 之前，以及在卸载期间，调用你的清理函数。

</Recap>

<Challenges>

#### 在挂载时聚焦一个字段 {/*focus-a-field-on-mount*/}

在这个示例中，表单渲染了一个 `<MyInput />` 组件。

使用输入框的 [`focus()`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/focus) 方法，让 `MyInput` 在出现在屏幕上时自动获得焦点。这里已经有一个被注释掉的实现，但它不太正常。找出它为什么不工作，并修复它。（如果你熟悉 `autoFocus` 属性，请假装它不存在：我们要从零重新实现同样的功能。）

<Sandpack>

```js src/MyInput.js active
import { useEffect, useRef } from 'react';

export default function MyInput({ value, onChange }) {
  const ref = useRef(null);

  // TODO：这不太正常。请修复它。
  // ref.current.focus()

  return (
    <input
      ref={ref}
      value={value}
      onChange={onChange}
    />
  );
}
```

```js src/App.js hidden
import { useState } from 'react';
import MyInput from './MyInput.js';

export default function Form() {
  const [show, setShow] = useState(false);
  const [name, setName] = useState('Taylor');
  const [upper, setUpper] = useState(false);
  return (
    <>
      <button onClick={() => setShow(s => !s)}>{show ? '隐藏' : '显示'}表单</button>
      <br />
      <hr />
      {show && (
        <>
          <label>
            输入你的名字：
            <MyInput
              value={name}
              onChange={e => setName(e.target.value)}
            />
          </label>
          <label>
            <input
              type="checkbox"
              checked={upper}
              onChange={e => setUpper(e.target.checked)}
            />
            转为大写
          </label>
          <p>你好，<b>{upper ? name.toUpperCase() : name}</b></p>
        </>
      )}
    </>
  );
}
```

```css
label {
  display: block;
  margin-top: 20px;
  margin-bottom: 20px;
}

body {
  min-height: 150px;
}
```

</Sandpack>


要验证你的解决方案是否有效，点击“显示表单”，确认输入框获得了焦点（被高亮，光标位于其中）。点击“隐藏表单”再点击“显示表单”。确认输入框再次高亮。

`MyInput` 应该只在 _挂载时_ 聚焦，而不是在每次渲染后都聚焦。为了验证行为是否正确，点击“显示表单”，然后反复点击“转为大写”复选框。点击复选框时，下面的输入框不应获得焦点。

<Solution>

在渲染期间调用 `ref.current.focus()` 是错误的，因为这属于 *副作用*。副作用应该放在事件处理器中，或者通过 `useEffect` 声明。在这个例子里，副作用是由组件出现在界面上引起的，而不是由某个具体交互引起的，因此把它放进 Effect 里更合理。

要修复这个错误，把 `ref.current.focus()` 调用包裹到一个 Effect 声明中。然后，为了确保这个 Effect 只在挂载时运行，而不是在每次渲染后都运行，给它添加空的 `[]` 依赖项。

<Sandpack>

```js src/MyInput.js active
import { useEffect, useRef } from 'react';

export default function MyInput({ value, onChange }) {
  const ref = useRef(null);

  useEffect(() => {
    ref.current.focus();
  }, []);

  return (
    <input
      ref={ref}
      value={value}
      onChange={onChange}
    />
  );
}
```

```js src/App.js hidden
import { useState } from 'react';
import MyInput from './MyInput.js';

export default function Form() {
  const [show, setShow] = useState(false);
  const [name, setName] = useState('Taylor');
  const [upper, setUpper] = useState(false);
  return (
    <>
      <button onClick={() => setShow(s => !s)}>{show ? '隐藏' : '显示'}表单</button>
      <br />
      <hr />
      {show && (
        <>
          <label>
            输入你的名字：
            <MyInput
              value={name}
              onChange={e => setName(e.target.value)}
            />
          </label>
          <label>
            <input
              type="checkbox"
              checked={upper}
              onChange={e => setUpper(e.target.checked)}
            />
            转为大写
          </label>
          <p>你好，<b>{upper ? name.toUpperCase() : name}</b></p>
        </>
      )}
    </>
  );
}
```

```css
label {
  display: block;
  margin-top: 20px;
  margin-bottom: 20px;
}

body {
  min-height: 150px;
}
```

</Sandpack>

</Solution>

#### 有条件地聚焦一个字段 {/*focus-a-field-conditionally*/}

这个表单渲染了两个 `<MyInput />` 组件。

点击“显示表单”，你会发现第二个字段会自动获得焦点。这是因为两个 `<MyInput />` 组件都试图聚焦它内部的输入框。当你连续对两个输入框调用 `focus()` 时，最后一个总是“胜出”。

假设你想让第一个字段获得焦点。现在第一个 `MyInput` 组件会接收到一个布尔值 `shouldFocus`，并被设为 `true`。修改逻辑，使得只有当 `MyInput` 接收到的 `shouldFocus` prop 为 `true` 时才调用 `focus()`。

<Sandpack>

```js src/MyInput.js active
import { useEffect, useRef } from 'react';

export default function MyInput({ shouldFocus, value, onChange }) {
  const ref = useRef(null);

  // TODO：仅当 shouldFocus 为 true 时调用 focus()。
  useEffect(() => {
    ref.current.focus();
  }, []);

  return (
    <input
      ref={ref}
      value={value}
      onChange={onChange}
    />
  );
}
```

```js src/App.js hidden
import { useState } from 'react';
import MyInput from './MyInput.js';

export default function Form() {
  const [show, setShow] = useState(false);
  const [firstName, setFirstName] = useState('Taylor');
  const [lastName, setLastName] = useState('Swift');
  const [upper, setUpper] = useState(false);
  const name = firstName + ' ' + lastName;
  return (
    <>
      <button onClick={() => setShow(s => !s)}>{show ? '隐藏' : '显示'}表单</button>
      <br />
      <hr />
      {show && (
        <>
          <label>
            输入你的名字：
            <MyInput
              value={firstName}
              onChange={e => setFirstName(e.target.value)}
              shouldFocus={true}
            />
          </label>
          <label>
            输入你的姓氏：
            <MyInput
              value={lastName}
              onChange={e => setLastName(e.target.value)}
              shouldFocus={false}
            />
          </label>
          <p>你好，<b>{upper ? name.toUpperCase() : name}</b></p>
        </>
      )}
    </>
  );
}
```

```css
label {
  display: block;
  margin-top: 20px;
  margin-bottom: 20px;
}

body {
  min-height: 150px;
}
```

</Sandpack>

要验证你的解决方案，请反复点击“显示表单”和“隐藏表单”。当表单出现时，只有 *第一个* 输入框应该获得焦点。这是因为父组件渲染第一个输入框时传入了 `shouldFocus={true}`，而第二个输入框传入了 `shouldFocus={false}`。另外也要检查两个输入框仍然可用，并且你可以在它们两个里面输入内容。

<Hint>

你不能有条件地声明一个 Effect，但你的 Effect 可以包含条件逻辑。

</Hint>

<Solution>

把条件逻辑放到 Effect 内部。你需要把 `shouldFocus` 指定为依赖项，因为你在 Effect 中使用了它。（这意味着如果某个输入框的 `shouldFocus` 从 `false` 变成 `true`，它会在挂载后获得焦点。）

<Sandpack>

```js src/MyInput.js active
import { useEffect, useRef } from 'react';

export default function MyInput({ shouldFocus, value, onChange }) {
  const ref = useRef(null);

  useEffect(() => {
    if (shouldFocus) {
      ref.current.focus();
    }
  }, [shouldFocus]);

  return (
    <input
      ref={ref}
      value={value}
      onChange={onChange}
    />
  );
}
```

```js src/App.js hidden
import { useState } from 'react';
import MyInput from './MyInput.js';

export default function Form() {
  const [show, setShow] = useState(false);
  const [firstName, setFirstName] = useState('Taylor');
  const [lastName, setLastName] = useState('Swift');
  const [upper, setUpper] = useState(false);
  const name = firstName + ' ' + lastName;
  return (
    <>
      <button onClick={() => setShow(s => !s)}>{show ? '隐藏' : '显示'}表单</button>
      <br />
      <hr />
      {show && (
        <>
          <label>
            输入你的名字：
            <MyInput
              value={firstName}
              onChange={e => setFirstName(e.target.value)}
              shouldFocus={true}
            />
          </label>
          <label>
            输入你的姓氏：
            <MyInput
              value={lastName}
              onChange={e => setLastName(e.target.value)}
              shouldFocus={false}
            />
          </label>
          <p>你好，<b>{upper ? name.toUpperCase() : name}</b></p>
        </>
      )}
    </>
  );
}
```

```css
label {
  display: block;
  margin-top: 20px;
  margin-bottom: 20px;
}

body {
  min-height: 150px;
}
```

</Sandpack>

</Solution>

#### 修复一个会触发两次的 interval {/*fix-an-interval-that-fires-twice*/}

这个 `Counter` 组件显示一个应该每秒递增一次的计数器。在挂载时，它会调用 [`setInterval`.](https://developer.mozilla.org/en-US/docs/Web/API/setInterval) 这会让 `onTick` 每秒运行一次。`onTick` 函数会递增计数器。

然而，它不是每秒递增一次，而是递增了两次。为什么会这样？找出 bug 的原因并修复它。

<Hint>

记住，`setInterval` 会返回一个 interval ID，你可以把它传给 [`clearInterval`](https://developer.mozilla.org/en-US/docs/Web/API/clearInterval) 来停止这个 interval。

</Hint>

<Sandpack>

```js src/Counter.js active
import { useState, useEffect } from 'react';

export default function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    function onTick() {
      setCount(c => c + 1);
    }

    setInterval(onTick, 1000);
  }, []);

  return <h1>{count}</h1>;
}
```

```js src/App.js hidden
import { useState } from 'react';
import Counter from './Counter.js';

export default function Form() {
  const [show, setShow] = useState(false);
  return (
    <>
      <button onClick={() => setShow(s => !s)}>{show ? '隐藏' : '显示'}计数器</button>
      <br />
      <hr />
      {show && <Counter />}
    </>
  );
}
```

```css
label {
  display: block;
  margin-top: 20px;
  margin-bottom: 20px;
}

body {
  min-height: 150px;
}
```

</Sandpack>

<Solution>

当 [Strict Mode](/reference/react/StrictMode) 开启时（就像这个网站上的沙盒一样），React 会在开发环境中让每个组件重新挂载一次。这会导致 interval 被设置两次，因此计数器每秒会递增两次。

不过，React 的这种行为并不是 bug 的 *原因*：bug 本来就已经存在于代码中了。React 的行为只是让 bug 更容易被注意到。真正的原因是，这个 Effect 启动了一个过程，但没有提供清理它的方法。

要修复这段代码，保存 `setInterval` 返回的 interval ID，并使用 [`clearInterval`](https://developer.mozilla.org/en-US/docs/Web/API/clearInterval) 实现一个清理函数：

<Sandpack>

```js src/Counter.js active
import { useState, useEffect } from 'react';

export default function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    function onTick() {
      setCount(c => c + 1);
    }

    const intervalId = setInterval(onTick, 1000);
    return () => clearInterval(intervalId);
  }, []);

  return <h1>{count}</h1>;
}
```

```js src/App.js hidden
import { useState } from 'react';
import Counter from './Counter.js';

export default function App() {
  const [show, setShow] = useState(false);
  return (
    <>
      <button onClick={() => setShow(s => !s)}>{show ? '隐藏' : '显示'}计数器</button>
      <br />
      <hr />
      {show && <Counter />}
    </>
  );
}
```

```css
label {
  display: block;
  margin-top: 20px;
  margin-bottom: 20px;
}

body {
  min-height: 150px;
}
```

</Sandpack>

在开发环境中，React 仍然会重新挂载你的组件一次，以验证你是否正确实现了清理。因此会有一次 `setInterval` 调用，紧接着是 `clearInterval`，然后 আবার `setInterval`。在生产环境中，只会有一次 `setInterval` 调用。两种情况下用户看到的行为都是一样的：计数器每秒递增一次。

</Solution>

#### 修复 Effect 中的请求 {/*fix-fetching-inside-an-effect*/}

这个组件显示所选人员的简介。它在挂载时以及 `person` 改变时，通过调用一个异步函数 `fetchBio(person)` 来加载简介。这个异步函数返回一个 [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)，最终会解析为一个字符串。获取完成后，它会调用 `setBio`，把该字符串显示在下拉框下方。

<Sandpack>

{/* not the most efficient, but this validation is enabled in the linter only, so it's fine to ignore it here since we know what we're doing */}
```js {expectedErrors: {'react-compiler': [9]}} src/App.js
import { useState, useEffect } from 'react';
import { fetchBio } from './api.js';

export default function Page() {
  const [person, setPerson] = useState('Alice');
  const [bio, setBio] = useState(null);

  useEffect(() => {
    setBio(null);
    fetchBio(person).then(result => {
      setBio(result);
    });
  }, [person]);

  return (
    <>
      <select value={person} onChange={e => {
        setPerson(e.target.value);
      }}>
        <option value="Alice">Alice</option>
        <option value="Bob">Bob</option>
        <option value="Taylor">Taylor</option>
      </select>
      <hr />
      <p><i>{bio ?? '加载中...'}</i></p>
    </>
  );
}
```

```js src/api.js hidden
export async function fetchBio(person) {
  const delay = person === 'Bob' ? 2000 : 200;
  return new Promise(resolve => {
    setTimeout(() => {
      resolve('This is ' + person + '’s bio.');
    }, delay);
  })
}

```

</Sandpack>


这段代码里有一个 bug。先选择 "Alice"。然后选择 "Bob"，紧接着再选择 "Taylor"。如果你操作够快，你就会发现这个 bug：Taylor 已经被选中了，但下面的段落却显示 “This is Bob's bio.”

为什么会这样？请在这个 Effect 内部修复这个 bug。

<Hint>

如果一个 Effect 异步地获取某些内容，它通常需要清理。

</Hint>

<Solution>

要触发这个 bug，事情需要按以下顺序发生：

- 选择 `'Bob'` 会触发 `fetchBio('Bob')`
- 选择 `'Taylor'` 会触发 `fetchBio('Taylor')`
- **获取 `'Taylor'` 比获取 `'Bob'` 更早完成**
- 来自 `'Taylor'` 渲染的 Effect 调用 `setBio('This is Taylor’s bio')`
- 获取 `'Bob'` 完成
- 来自 `'Bob'` 渲染的 Effect 调用 `setBio('This is Bob’s bio')`

这就是为什么即使已经选择了 Taylor，你仍然会看到 Bob 的简介。这类 bug 被称为 [竞态条件](https://en.wikipedia.org/wiki/Race_condition)，因为两个异步操作在彼此“竞争”，并且可能以意料之外的顺序完成。

要修复这个竞态条件，添加一个清理函数：

<Sandpack>

{/* not the most efficient, but this validation is enabled in the linter only, so it's fine to ignore it here since we know what we're doing */}
```js {expectedErrors: {'react-compiler': [9]}} src/App.js
import { useState, useEffect } from 'react';
import { fetchBio } from './api.js';

export default function Page() {
  const [person, setPerson] = useState('Alice');
  const [bio, setBio] = useState(null);
  useEffect(() => {
    let ignore = false;
    setBio(null);
    fetchBio(person).then(result => {
      if (!ignore) {
        setBio(result);
      }
    });
    return () => {
      ignore = true;
    }
  }, [person]);

  return (
    <>
      <select value={person} onChange={e => {
        setPerson(e.target.value);
      }}>
        <option value="Alice">Alice</option>
        <option value="Bob">Bob</option>
        <option value="Taylor">Taylor</option>
      </select>
      <hr />
      <p><i>{bio ?? '加载中...'}</i></p>
    </>
  );
}
```

```js src/api.js hidden
export async function fetchBio(person) {
  const delay = person === 'Bob' ? 2000 : 200;
  return new Promise(resolve => {
    setTimeout(() => {
      resolve('This is ' + person + '’s bio.');
    }, delay);
  })
}

```

</Sandpack>

每次渲染的 Effect 都有自己的 `ignore` 变量。最开始，`ignore` 变量被设为 `false`。但是，如果某个 Effect 被清理了（例如你选择了另一个人），它的 `ignore` 变量就会变成 `true`。这样一来，请求以什么顺序完成就无所谓了。只有最后那个人对应的 Effect 才会把 `ignore` 设为 `false`，因此它会调用 `setBio(result)`。之前的 Effects 已经被清理，所以 `if (!ignore)` 检查会阻止它们调用 `setBio`：

- 选择 `'Bob'` 会触发 `fetchBio('Bob')`
- 选择 `'Taylor'` 会触发 `fetchBio('Taylor')`，**并清理之前（Bob 的）Effect**
- 获取 `'Taylor'` 比获取 `'Bob'` 更早完成
- 来自 `'Taylor'` 渲染的 Effect 调用 `setBio('This is Taylor’s bio')`
- 获取 `'Bob'` 完成
- 来自 `'Bob'` 渲染的 Effect **不会做任何事，因为它的 `ignore` 标志已经被设为 `true`**

除了忽略过时 API 调用的结果之外，你还可以使用 [`AbortController`](https://developer.mozilla.org/en-US/docs/Web/API/AbortController) 来取消不再需要的请求。不过，仅靠这一点并不足以防止竞态条件。因为在 fetch 之后还可能串联更多异步步骤，所以使用像 `ignore` 这样的显式标志，是修复这类问题最可靠的方法。

</Solution>

</Challenges>
