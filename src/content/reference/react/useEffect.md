---
title: useEffect
---

<Intro>

`useEffect` 是一个 React Hook，它允许你[将组件与外部系统同步。](/learn/synchronizing-with-effects)

```js
useEffect(setup, dependencies?)
```

</Intro>

<InlineToc />

---

## 参考 {/*reference*/}

### `useEffect(setup, dependencies?)` {/*useeffect*/}

在组件顶层调用 `useEffect` 来声明一个 Effect：

```js
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [serverUrl, roomId]);
  // ...
}
```

[查看下面的更多示例。](#usage)

#### 参数 {/*parameters*/}

* `setup`：包含你的 Effect 逻辑的函数。你的 setup 函数也可以选择返回一个 *cleanup* 函数。当你的[组件提交](/learn/render-and-commit#step-3-react-commits-changes-to-the-dom)时，React 会运行你的 setup 函数。每次依赖项发生变化并提交后，React 会先使用旧值运行 cleanup 函数（如果你提供了），然后再使用新值运行 setup 函数。当组件从 DOM 中移除后，React 会运行你的 cleanup 函数。

* **可选** `dependencies`：`setup` 代码中引用的所有响应式值列表。响应式值包括 props、state，以及直接在组件函数体中声明的所有变量和函数。如果你的 linter 已[为 React 配置](/learn/editor-setup#linting)，它会验证每个响应式值都被正确指定为依赖项。依赖项列表必须具有固定数量的项目，并以内联方式编写，如 `[dep1, dep2, dep3]`。React 会使用 [`Object.is`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is) 比较来对比每个依赖项与其上一次的值。如果你省略这个参数，你的 Effect 会在组件每次提交后重新运行。[查看传入依赖数组、空数组以及完全不传依赖之间的区别。](#examples-dependencies)

#### 返回值 {/*returns*/}

`useEffect` 返回 `undefined`。

#### 注意事项 {/*caveats*/}

* `useEffect` 是一个 Hook，所以你只能在**组件顶层**或你自己的 Hooks 中调用它。你不能在循环或条件中调用它。如果你需要那样做，提取出一个新组件并把状态移到其中。

* 如果你**并不是要与某个外部系统同步，**[你大概不需要 Effect。](/learn/you-might-not-need-an-effect)

* 当 Strict Mode 开启时，React 会在第一次真正的 setup 之前，**额外执行一次仅开发环境下的 setup+cleanup 循环**。这是一个压力测试，用来确保你的 cleanup 逻辑“镜像”了 setup 逻辑，并且它能停止或撤销 setup 所做的任何事情。如果这导致问题，[请实现 cleanup 函数。](/learn/synchronizing-with-effects#how-to-handle-the-effect-firing-twice-in-development)

* 如果你的某些依赖是组件内部定义的对象或函数，就有可能**导致 Effect 比需要的更频繁地重新运行。**要修复这个问题，请移除不必要的[对象](#removing-unnecessary-object-dependencies)和[函数](#removing-unnecessary-function-dependencies)依赖。你也可以在 Effect 外部[提取基于之前 state 的更新](#updating-state-based-on-previous-state-from-an-effect)和[非响应式逻辑](#reading-the-latest-props-and-state-from-an-effect)。

* 如果你的 Effect 不是由交互（比如点击）触发，React 通常会让浏览器**先绘制更新后的屏幕，再运行你的 Effect。**如果你的 Effect 做的是视觉相关的事情（例如定位 tooltip），而且延迟很明显（例如它闪烁），请用 [`useLayoutEffect` 替代 `useEffect`。](/reference/react/useLayoutEffect)

* 如果你的 Effect 是由交互（比如点击）触发的，**React 可能会在浏览器绘制更新后的屏幕之前运行你的 Effect**。这可确保事件系统能够观察到 Effect 的结果。通常这会按预期工作。不过，如果你必须把工作延后到绘制之后，比如 `alert()`，可以使用 `setTimeout`。更多信息请参见 [reactwg/react-18/128](https://github.com/reactwg/react-18/discussions/128)。

* 即使你的 Effect 是由交互（比如点击）触发的，**React 也可能允许浏览器在处理 Effect 内部的 state 更新之前重绘屏幕。**通常这会按预期工作。不过，如果你必须阻止浏览器重绘屏幕，需要把 `useEffect` 替换为 [`useLayoutEffect`。](/reference/react/useLayoutEffect)

* Effect **只在客户端运行。**它们不会在服务端渲染期间运行。

---

## 用法 {/*usage*/}

### 连接到外部系统 {/*connecting-to-an-external-system*/}

有些组件在页面上显示时，需要保持连接到网络、某个浏览器 API，或者第三方库。这些系统不受 React 控制，所以它们被称为 *外部* 系统。

要[将组件连接到某个外部系统，](/learn/synchronizing-with-effects)请在组件顶层调用 `useEffect`：

```js [[1, 8, "const connection = createConnection(serverUrl, roomId);"], [1, 9, "connection.connect();"], [2, 11, "connection.disconnect();"], [3, 13, "[serverUrl, roomId]"]]
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  useEffect(() => {
  	const connection = createConnection(serverUrl, roomId);
    connection.connect();
  	return () => {
      connection.disconnect();
  	};
  }, [serverUrl, roomId]);
  // ...
}
```

你需要向 `useEffect` 传入两个参数：

1. 一个带有 <CodeStep step={1}>setup 代码</CodeStep> 的 *setup 函数*，用于连接到那个系统。
   - 它应该返回一个带有 <CodeStep step={2}>cleanup 代码</CodeStep> 的 *cleanup 函数*，用于断开与该系统的连接。
2. 一个包含这些函数内部用到的组件中每个值的 <CodeStep step={3}>依赖列表</CodeStep>。

**React 会在必要时调用你的 setup 和 cleanup 函数，这可能会发生多次：**

1. 当组件被添加到页面时，你的 <CodeStep step={1}>setup 代码</CodeStep> 会运行 *(mounts)*。
2. 在组件每次提交后，只要 <CodeStep step={3}>dependencies</CodeStep> 发生了变化：
   - 首先，你的 <CodeStep step={2}>cleanup 代码</CodeStep> 会使用旧的 props 和 state 运行。
   - 然后，你的 <CodeStep step={1}>setup 代码</CodeStep> 会使用新的 props 和 state 运行。
3. 当组件从页面中移除时，你的 <CodeStep step={2}>cleanup 代码</CodeStep> 会最后运行一次 *(unmounts)*。

**让我们用上面的示例来说明这个顺序。**

当上面的 `ChatRoom` 组件被添加到页面时，它会使用初始的 `serverUrl` 和 `roomId` 连接到聊天室。如果 `serverUrl` 或 `roomId` 因某次提交而发生变化（例如用户在下拉菜单中选择了不同的聊天室），你的 Effect 会*断开与上一个房间的连接，并连接到下一个房间。*当 `ChatRoom` 组件从页面移除时，你的 Effect 会最后断开一次连接。

**为了[帮助你发现 bug，](/learn/synchronizing-with-effects#step-3-add-cleanup-if-needed)在开发环境中，React 会在 <CodeStep step={1}>setup</CodeStep> 之前额外运行一次 <CodeStep step={2}>cleanup</CodeStep>。**这是一个压力测试，用来验证你的 Effect 逻辑是否正确实现。如果这导致可见的问题，说明你的 cleanup 函数缺少了一些逻辑。cleanup 函数应该停止或撤销 setup 函数所做的一切。经验法则是，用户不应该能分辨出 setup 只运行一次（如生产环境）与 *setup* → *cleanup* → *setup* 这个序列（如开发环境）之间的区别。[查看常见解决方案。](/learn/synchronizing-with-effects#how-to-handle-the-effect-firing-twice-in-development)

**尝试[把每个 Effect 都写成一个独立的过程](/learn/lifecycle-of-reactive-effects#each-effect-represents-a-separate-synchronization-process)，并且[每次只考虑一个 setup/cleanup 循环。](/learn/lifecycle-of-reactive-effects#thinking-from-the-effects-perspective)** 组件是在挂载、更新还是卸载都不重要。当你的 cleanup 逻辑正确地“镜像”了 setup 逻辑时，你的 Effect 就能在需要时多次运行 setup 和 cleanup 而保持稳定。

<Note>

Effect 让你可以[让组件与某个外部系统保持同步](/learn/synchronizing-with-effects)（例如聊天服务）。这里的 *external system* 指任何不受 React 控制的代码，例如：

* 通过 <CodeStep step={1}>[`setInterval()`](https://developer.mozilla.org/en-US/docs/Web/API/setInterval)</CodeStep> 和 <CodeStep step={2}>[`clearInterval()`](https://developer.mozilla.org/en-US/docs/Web/API/clearInterval)</CodeStep> 管理的计时器。
* 通过 <CodeStep step={1}>[`window.addEventListener()`](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener)</CodeStep> 和 <CodeStep step={2}>[`window.removeEventListener()`](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/removeEventListener)</CodeStep> 实现的事件订阅。
* 带有类似 <CodeStep step={1}>`animation.start()`</CodeStep> 和 <CodeStep step={2}>`animation.reset()`</CodeStep> API 的第三方动画库。

**如果你没有连接任何外部系统，[你大概不需要 Effect。](/learn/you-might-not-need-an-effect)**

</Note>

<Recipes titleText="连接到外部系统的示例" titleId="examples-connecting">

#### 连接到聊天服务器 {/*connecting-to-a-chat-server*/}

在这个示例中，`ChatRoom` 组件使用 Effect 来保持与 `chat.js` 中定义的外部系统连接。点击“Open chat”让 `ChatRoom` 组件显示出来。这个沙箱运行在开发模式下，所以会有额外的连接和断开循环，如[此处所解释。](/learn/synchronizing-with-effects#step-3-add-cleanup-if-needed)尝试使用下拉菜单和输入框更改 `roomId` 和 `serverUrl`，看看 Effect 如何重新连接到聊天。点击“Close chat”可以看到 Effect 最后一次断开连接。

<Sandpack>

```js
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [roomId, serverUrl]);

  return (
    <>
      <label>
        Server URL:{' '}
        <input
          value={serverUrl}
          onChange={e => setServerUrl(e.target.value)}
        />
      </label>
      <h1>欢迎来到 {roomId} 房间！</h1>
    </>
  );
}

export default function App() {
  const [roomId, setRoomId] = useState('general');
  const [show, setShow] = useState(false);
  return (
    <>
      <label>
        选择聊天室：{' '}
        <select
          value={roomId}
          onChange={e => setRoomId(e.target.value)}
        >
          <option value="general">general</option>
          <option value="travel">travel</option>
          <option value="music">music</option>
        </select>
      </label>
      <button onClick={() => setShow(!show)}>
        {show ? 'Close chat' : 'Open chat'}
      </button>
      {show && <hr />}
      {show && <ChatRoom roomId={roomId} />}
    </>
  );
}
```

```js src/chat.js
export function createConnection(serverUrl, roomId) {
  // 真实实现会实际连接到服务器
  return {
    connect() {
      console.log('✅ Connecting to "' + roomId + '" room at ' + serverUrl + '...');
    },
    disconnect() {
      console.log('❌ Disconnected from "' + roomId + '" room at ' + serverUrl);
    }
  };
}
```

```css
input { display: block; margin-bottom: 20px; }
button { margin-left: 10px; }
```

</Sandpack>

<Solution />

#### 监听全局浏览器事件 {/*listening-to-a-global-browser-event*/}

在这个示例中，外部系统就是浏览器 DOM 本身。通常，你会通过 JSX 指定事件监听器，但不能用这种方式监听全局的 [`window`](https://developer.mozilla.org/en-US/docs/Web/API/Window) 对象。Effect 让你可以连接到 `window` 对象并监听它的事件。监听 `pointermove` 事件可以让你跟踪光标（或手指）的位置，并更新红点让它跟随移动。

<Sandpack>

```js
import { useState, useEffect } from 'react';

export default function App() {
  const [position, setPosition] = useState({ x: 0, y: 0 });

  useEffect(() => {
    function handleMove(e) {
      setPosition({ x: e.clientX, y: e.clientY });
    }
    window.addEventListener('pointermove', handleMove);
    return () => {
      window.removeEventListener('pointermove', handleMove);
    };
  }, []);

  return (
    <div style={{
      position: 'absolute',
      backgroundColor: 'pink',
      borderRadius: '50%',
      opacity: 0.6,
      transform: `translate(${position.x}px, ${position.y}px)`,
      pointerEvents: 'none',
      left: -20,
      top: -20,
      width: 40,
      height: 40,
    }} />
  );
}
```

```css
body {
  min-height: 300px;
}
```

</Sandpack>

<Solution />

#### 触发动画 {/*triggering-an-animation*/}

在这个示例中，外部系统是 `animation.js` 中的动画库。它提供了一个名为 `FadeInAnimation` 的 JavaScript 类，它以一个 DOM 节点作为参数，并暴露 `start()` 和 `stop()` 方法来控制动画。这个组件[使用 ref](/learn/manipulating-the-dom-with-refs) 来访问底层 DOM 节点。Effect 从 ref 中读取 DOM 节点，并在组件出现时自动为该节点启动动画。

<Sandpack>

```js
import { useState, useEffect, useRef } from 'react';
import { FadeInAnimation } from './animation.js';

function Welcome() {
  const ref = useRef(null);

  useEffect(() => {
    const animation = new FadeInAnimation(ref.current);
    animation.start(1000);
    return () => {
      animation.stop();
    };
  }, []);

  return (
    <h1
      ref={ref}
      style={{
        opacity: 0,
        color: 'white',
        padding: 50,
        textAlign: 'center',
        fontSize: 50,
        backgroundImage: 'radial-gradient(circle, rgba(63,94,251,1) 0%, rgba(252,70,107,1) 100%)'
      }}
    >
      Welcome
    </h1>
  );
}

export default function App() {
  const [show, setShow] = useState(false);
  return (
    <>
      <button onClick={() => setShow(!show)}>
        {show ? 'Remove' : 'Show'}
      </button>
      <hr />
      {show && <Welcome />}
    </>
  );
}
```

```js src/animation.js
export class FadeInAnimation {
  constructor(node) {
    this.node = node;
  }
  start(duration) {
    this.duration = duration;
    if (this.duration === 0) {
      // 立即跳到末尾
      this.onProgress(1);
    } else {
      this.onProgress(0);
      // 开始动画
      this.startTime = performance.now();
      this.frameId = requestAnimationFrame(() => this.onFrame());
    }
  }
  onFrame() {
    const timePassed = performance.now() - this.startTime;
    const progress = Math.min(timePassed / this.duration, 1);
    this.onProgress(progress);
    if (progress < 1) {
      // 我们还有更多帧需要绘制
      this.frameId = requestAnimationFrame(() => this.onFrame());
    }
  }
  onProgress(progress) {
    this.node.style.opacity = progress;
  }
  stop() {
    cancelAnimationFrame(this.frameId);
    this.startTime = null;
    this.frameId = null;
    this.duration = 0;
  }
}
```

```css
label, button { display: block; margin-bottom: 20px; }
html, body { min-height: 300px; }
```

</Sandpack>

<Solution />

#### 控制模态对话框 {/*controlling-a-modal-dialog*/}

在这个示例中，外部系统是浏览器 DOM。`ModalDialog` 组件渲染一个 [`<dialog>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/dialog) 元素。它使用 Effect 将 `isOpen` prop 同步到 [`showModal()`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLDialogElement/showModal) 和 [`close()`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLDialogElement/close) 方法调用上。

<Sandpack>

```js
import { useState } from 'react';
import ModalDialog from './ModalDialog.js';

export default function App() {
  const [show, setShow] = useState(false);
  return (
    <>
      <button onClick={() => setShow(true)}>
        打开对话框
      </button>
      <ModalDialog isOpen={show}>
        你好！
        <br />
        <button onClick={() => {
          setShow(false);
        }}>关闭</button>
      </ModalDialog>
    </>
  );
}
```

```js src/ModalDialog.js active
import { useEffect, useRef } from 'react';

export default function ModalDialog({ isOpen, children }) {
  const ref = useRef();

  useEffect(() => {
    if (!isOpen) {
      return;
    }
    const dialog = ref.current;
    dialog.showModal();
    return () => {
      dialog.close();
    };
  }, [isOpen]);

  return <dialog ref={ref}>{children}</dialog>;
}
```

```css
body {
  min-height: 300px;
}
```

</Sandpack>

<Solution />

#### 跟踪元素可见性 {/*tracking-element-visibility*/}

在这个示例中，外部系统同样是浏览器 DOM。`App` 组件先显示一个长列表，然后是一个 `Box` 组件，接着又是另一个长列表。向下滚动列表。注意，当 `Box` 组件完全出现在视口中时，背景颜色会变成黑色。为实现这一点，`Box` 组件使用 Effect 来管理一个 [`IntersectionObserver`](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API)。这个浏览器 API 会在 DOM 元素在视口中可见时通知你。

<Sandpack>

```js
import Box from './Box.js';

export default function App() {
  return (
    <>
      <LongSection />
      <Box />
      <LongSection />
      <Box />
      <LongSection />
    </>
  );
}

function LongSection() {
  const items = [];
  for (let i = 0; i < 50; i++) {
    items.push(<li key={i}>Item #{i} (keep scrolling)</li>);
  }
  return <ul>{items}</ul>;
}
```

```js src/Box.js active
import { useRef, useEffect } from 'react';

export default function Box() {
  const ref = useRef(null);

  useEffect(() => {
    const div = ref.current;
    const observer = new IntersectionObserver(entries => {
      const entry = entries[0];
      if (entry.isIntersecting) {
        document.body.style.backgroundColor = 'black';
        document.body.style.color = 'white';
      } else {
        document.body.style.backgroundColor = 'white';
        document.body.style.color = 'black';
      }
    }, {
       threshold: 1.0
    });
    observer.observe(div);
    return () => {
      observer.disconnect();
    }
  }, []);

  return (
    <div ref={ref} style={{
      margin: 20,
      height: 100,
      width: 100,
      border: '2px solid black',
      backgroundColor: 'blue'
    }} />
  );
}
```

</Sandpack>

<Solution />

</Recipes>

---

### 将 Effects 封装在自定义 Hooks 中 {/*wrapping-effects-in-custom-hooks*/}

Effects 是一种["逃生舱："](/learn/escape-hatches)当你需要“跳出 React”，且你的用例没有更好的内置方案时，就会用到它们。如果你经常需要手动编写 Effects，这通常意味着你需要为组件依赖的常见行为提取一些[自定义 Hooks](/learn/reusing-logic-with-custom-hooks)。

例如，这个 `useChatRoom` 自定义 Hook 通过更具声明性的 API “隐藏”了 Effect 的逻辑：

```js {1,11}
function useChatRoom({ serverUrl, roomId }) {
  useEffect(() => {
    const options = {
      serverUrl: serverUrl,
      roomId: roomId
    };
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId, serverUrl]);
}
```

然后你就可以在任何组件中这样使用它：

```js {4-7}
function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  useChatRoom({
    roomId: roomId,
    serverUrl: serverUrl
  });
  // ...
```

React 生态系统中还有许多适用于各种用途的优秀自定义 Hooks。

[了解更多关于将 Effects 封装在自定义 Hooks 中的信息。](/learn/reusing-logic-with-custom-hooks)

<Recipes titleText="将 Effects 封装在自定义 Hooks 中的示例" titleId="examples-custom-hooks">

#### 自定义 `useChatRoom` Hook {/*custom-usechatroom-hook*/}

这个示例与[前面的示例](#examples-connecting)中的一个完全相同，只是逻辑被提取到了一个自定义 Hook 中。

<Sandpack>

```js
import { useState } from 'react';
import { useChatRoom } from './useChatRoom.js';

function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  useChatRoom({
    roomId: roomId,
    serverUrl: serverUrl
  });

  return (
    <>
      <label>
        Server URL:{' '}
        <input
          value={serverUrl}
          onChange={e => setServerUrl(e.target.value)}
        />
      </label>
      <h1>欢迎来到 {roomId} 房间！</h1>
    </>
  );
}

export default function App() {
  const [roomId, setRoomId] = useState('general');
  const [show, setShow] = useState(false);
  return (
    <>
      <label>
        选择聊天室：{' '}
        <select
          value={roomId}
          onChange={e => setRoomId(e.target.value)}
        >
          <option value="general">general</option>
          <option value="travel">travel</option>
          <option value="music">music</option>
        </select>
      </label>
      <button onClick={() => setShow(!show)}>
        {show ? 'Close chat' : 'Open chat'}
      </button>
      {show && <hr />}
      {show && <ChatRoom roomId={roomId} />}
    </>
  );
}
```

```js src/useChatRoom.js
import { useEffect } from 'react';
import { createConnection } from './chat.js';

export function useChatRoom({ serverUrl, roomId }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [roomId, serverUrl]);
}
```

```js src/chat.js
export function createConnection(serverUrl, roomId) {
  // 真实实现会实际连接到服务器
  return {
    connect() {
      console.log('✅ Connecting to "' + roomId + '" room at ' + serverUrl + '...');
    },
    disconnect() {
      console.log('❌ Disconnected from "' + roomId + '" room at ' + serverUrl);
    }
  };
}
```

```css
input { display: block; margin-bottom: 20px; }
button { margin-left: 10px; }
```

</Sandpack>

<Solution />

#### 自定义 `useWindowListener` Hook {/*custom-usewindowlistener-hook*/}

这个示例与[前面的示例](#examples-connecting)中的一个完全相同，只是逻辑被提取到了一个自定义 Hook 中。

<Sandpack>

```js
import { useState } from 'react';
import { useWindowListener } from './useWindowListener.js';

export default function App() {
  const [position, setPosition] = useState({ x: 0, y: 0 });

  useWindowListener('pointermove', (e) => {
    setPosition({ x: e.clientX, y: e.clientY });
  });

  return (
    <div style={{
      position: 'absolute',
      backgroundColor: 'pink',
      borderRadius: '50%',
      opacity: 0.6,
      transform: `translate(${position.x}px, ${position.y}px)`,
      pointerEvents: 'none',
      left: -20,
      top: -20,
      width: 40,
      height: 40,
    }} />
  );
}
```

```js src/useWindowListener.js
import { useState, useEffect } from 'react';

export function useWindowListener(eventType, listener) {
  useEffect(() => {
    window.addEventListener(eventType, listener);
    return () => {
      window.removeEventListener(eventType, listener);
    };
  }, [eventType, listener]);
}
```

```css
body {
  min-height: 300px;
}
```

</Sandpack>

<Solution />

#### 自定义 `useIntersectionObserver` Hook {/*custom-useintersectionobserver-hook*/}

这个示例与[前面的示例](#examples-connecting)中的一个完全相同，只是逻辑部分提取到了一个自定义 Hook 中。

<Sandpack>

```js
import Box from './Box.js';

export default function App() {
  return (
    <>
      <LongSection />
      <Box />
      <LongSection />
      <Box />
      <LongSection />
    </>
  );
}

function LongSection() {
  const items = [];
  for (let i = 0; i < 50; i++) {
    items.push(<li key={i}>Item #{i} (keep scrolling)</li>);
  }
  return <ul>{items}</ul>;
}
```

```js src/Box.js active
import { useRef, useEffect } from 'react';
import { useIntersectionObserver } from './useIntersectionObserver.js';

export default function Box() {
  const ref = useRef(null);
  const isIntersecting = useIntersectionObserver(ref);

  useEffect(() => {
   if (isIntersecting) {
      document.body.style.backgroundColor = 'black';
      document.body.style.color = 'white';
    } else {
      document.body.style.backgroundColor = 'white';
      document.body.style.color = 'black';
    }
  }, [isIntersecting]);

  return (
    <div ref={ref} style={{
      margin: 20,
      height: 100,
      width: 100,
      border: '2px solid black',
      backgroundColor: 'blue'
    }} />
  );
}
```

```js src/useIntersectionObserver.js
import { useState, useEffect } from 'react';

export function useIntersectionObserver(ref) {
  const [isIntersecting, setIsIntersecting] = useState(false);

  useEffect(() => {
    const div = ref.current;
    const observer = new IntersectionObserver(entries => {
      const entry = entries[0];
      setIsIntersecting(entry.isIntersecting);
    }, {
       threshold: 1.0
    });
    observer.observe(div);
    return () => {
      observer.disconnect();
    }
  }, [ref]);

  return isIntersecting;
}
```

</Sandpack>

<Solution />

</Recipes>

---

### 控制非 React 组件 {/*controlling-a-non-react-widget*/}

有时，你希望让某个外部系统与你组件的某个 prop 或 state 保持同步。

例如，如果你有一个第三方地图组件或一个没有用 React 编写的视频播放器组件，你可以使用 Effect 调用它的方法，让它的状态与 React 组件的当前状态一致。这个 Effect 会创建一个定义在 `map-widget.js` 中的 `MapWidget` 类实例。当你更改 `Map` 组件的 `zoomLevel` prop 时，Effect 会调用该类实例上的 `setZoom()`，以保持同步：

<Sandpack>

```json package.json hidden
{
  "dependencies": {
    "leaflet": "1.9.1",
    "react": "latest",
    "react-dom": "latest",
    "react-scripts": "latest",
    "remarkable": "2.0.1"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject"
  }
}
```

```js src/App.js
import { useState } from 'react';
import Map from './Map.js';

export default function App() {
  const [zoomLevel, setZoomLevel] = useState(0);
  return (
    <>
      缩放级别：{zoomLevel}x
      <button onClick={() => setZoomLevel(zoomLevel + 1)}>+</button>
      <button onClick={() => setZoomLevel(zoomLevel - 1)}>-</button>
      <hr />
      <Map zoomLevel={zoomLevel} />
    </>
  );
}
```

```js src/Map.js active
import { useRef, useEffect } from 'react';
import { MapWidget } from './map-widget.js';

export default function Map({ zoomLevel }) {
  const containerRef = useRef(null);
  const mapRef = useRef(null);

  useEffect(() => {
    if (mapRef.current === null) {
      mapRef.current = new MapWidget(containerRef.current);
    }

    const map = mapRef.current;
    map.setZoom(zoomLevel);
  }, [zoomLevel]);

  return (
    <div
      style={{ width: 200, height: 200 }}
      ref={containerRef}
    />
  );
}
```

```js src/map-widget.js
import 'leaflet/dist/leaflet.css';
import * as L from 'leaflet';

export class MapWidget {
  constructor(domNode) {
    this.map = L.map(domNode, {
      zoomControl: false,
      doubleClickZoom: false,
      boxZoom: false,
      keyboard: false,
      scrollWheelZoom: false,
      zoomAnimation: false,
      touchZoom: false,
      zoomSnap: 0.1
    });
    L.tileLayer('https://tile.openstreetmap.org/{z}/{x}/{y}.png', {
      maxZoom: 19,
      attribution: '© OpenStreetMap'
    }).addTo(this.map);
    this.map.setView([0, 0], 0);
  }
  setZoom(level) {
    this.map.setZoom(level);
  }
}
```

```css
button { margin: 5px; }
```

</Sandpack>

在这个示例中，不需要 cleanup 函数，因为 `MapWidget` 类只管理传给它的 DOM 节点。当 `Map` React 组件从树中移除后，该 DOM 节点和 `MapWidget` 类实例都会被浏览器 JavaScript 引擎自动垃圾回收。

---

### 使用 Effects 获取数据 {/*fetching-data-with-effects*/}

你可以使用 Effect 为组件获取数据。注意，[如果你使用框架，](/learn/creating-a-react-app#full-stack-frameworks)使用框架自带的数据获取机制会比手动编写 Effects 高效得多。

如果你想手动在 Effect 中获取数据，你的代码可能会像这样：

```js
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
    };
  }, [person]);

  // ...
```

注意 `ignore` 变量，它初始值为 `false`，并在 cleanup 时被设为 `true`。这可确保[你的代码不会出现“竞态条件”：](https://maxrozen.com/race-conditions-fetching-data-react-with-useeffect)网络响应到达的顺序可能与发送顺序不同。

<Sandpack>

{/* TODO(@poteto) - investigate potential false positives in react compiler validation */}
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
      <p><i>{bio ?? 'Loading...'}</i></p>
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

你也可以使用 [`async` / `await`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function) 语法重写，但仍然需要提供 cleanup 函数：

<Sandpack>

```js src/App.js
import { useState, useEffect } from 'react';
import { fetchBio } from './api.js';

export default function Page() {
  const [person, setPerson] = useState('Alice');
  const [bio, setBio] = useState(null);
  useEffect(() => {
    async function startFetching() {
      setBio(null);
      const result = await fetchBio(person);
      if (!ignore) {
        setBio(result);
      }
    }

    let ignore = false;
    startFetching();
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
      <p><i>{bio ?? 'Loading...'}</i></p>
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

直接在 Effects 中编写数据获取会变得重复，并且以后很难再添加缓存和服务端渲染等优化。[更简单的做法是使用自定义 Hook——无论是你自己写的还是社区维护的。](/learn/reusing-logic-with-custom-hooks#when-to-use-custom-hooks)

<DeepDive>

#### 在 Effects 中获取数据的更好替代方案是什么？ {/*what-are-good-alternatives-to-data-fetching-in-effects*/}

在 Effects 中编写 `fetch` 调用是一种[很流行的数据获取方式](https://www.robinwieruch.de/react-hooks-fetch-data/)，尤其是在完全由客户端渲染的应用中。不过，这是一种非常手工的做法，而且有明显缺点：

- **Effects 不会在服务端运行。**这意味着初始的服务端渲染 HTML 里只会包含没有数据的加载状态。客户端计算机必须下载所有 JavaScript 并渲染你的应用，最终才发现它现在还需要加载数据。这并不高效。
- **直接在 Effects 中获取数据很容易造成“网络瀑布”。**你先渲染父组件，它获取一些数据，再渲染子组件，然后子组件又开始获取自己的数据。如果网络不够快，这会明显慢于并行获取所有数据。
- **直接在 Effects 中获取数据通常意味着你不会预加载或缓存数据。**例如，如果组件卸载后再挂载，就必须再次获取数据。
- **这在开发体验上并不理想。**以不会出现像[竞态条件。](https://maxrozen.com/race-conditions-fetching-data-react-with-useeffect)这类 bug 的方式编写 `fetch` 调用，需要不少样板代码。

这份缺点列表并不只针对 React。它适用于任何库中在挂载时获取数据的做法。和路由一样，数据获取并不容易做好，因此我们建议以下方法：

- **如果你使用[框架](/learn/creating-a-react-app#full-stack-frameworks)，请使用它内置的数据获取机制。**现代 React 框架集成了高效的数据获取机制，而且不会有上面这些问题。
- **否则，可以考虑使用或构建一个客户端缓存。**流行的开源方案包括 [TanStack Query](https://tanstack.com/query/latest/)、[useSWR](https://swr.vercel.app/) 和 [React Router 6.4+。](https://beta.reactrouter.com/en/main/start/overview)你也可以自己构建方案，在这种情况下，底层仍会使用 Effects，但还会增加请求去重、响应缓存，以及通过预加载数据或将数据需求上提到路由来避免网络瀑布的逻辑。

如果这两种方案都不适合你，你仍然可以继续直接在 Effects 中获取数据。

</DeepDive>

---

### 指定响应式依赖 {/*specifying-reactive-dependencies*/}

**注意，你不能“选择”Effect 的依赖项。**Effect 代码中使用的每个<CodeStep step={2}>响应式值</CodeStep>都必须声明为依赖项。Effect 的依赖列表由周围代码决定：

```js [[2, 1, "roomId"], [2, 2, "serverUrl"], [2, 5, "serverUrl"], [2, 5, "roomId"], [2, 8, "serverUrl"], [2, 8, "roomId"]]
function ChatRoom({ roomId }) { // 这是一个响应式值
  const [serverUrl, setServerUrl] = useState('https://localhost:1234'); // 这也是一个响应式值

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId); // 这个 Effect 读取这些响应式值
    connection.connect();
    return () => connection.disconnect();
  }, [serverUrl, roomId]); // ✅ 所以你必须把它们指定为 Effect 的依赖项
  // ...
}
```

如果 `serverUrl` 或 `roomId` 发生变化，你的 Effect 会使用新值重新连接到聊天室。

**[响应式值](/learn/lifecycle-of-reactive-effects#effects-react-to-reactive-values)包括 props 以及直接在组件中声明的所有变量和函数。**由于 `roomId` 和 `serverUrl` 是响应式值，你不能把它们从依赖项中移除。如果你试图省略它们，而[你的 linter 已正确为 React 配置，](/learn/editor-setup#linting)linter 会把这标记为需要修复的错误：

```js {8}
function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, []); // 🔴 React Hook useEffect has missing dependencies: 'roomId' and 'serverUrl'
  // ...
}
```

**要移除一个依赖，你需要[向 linter“证明”它*不需要*成为依赖。](/learn/removing-effect-dependencies#removing-unnecessary-dependencies)**例如，你可以把 `serverUrl` 移到组件外部，以证明它不是响应式的，不会在重新渲染时改变：

```js {1,8}
const serverUrl = 'https://localhost:1234'; // 不再是响应式值了

function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]); // ✅ 所有依赖项都已声明
  // ...
}
```

现在 `serverUrl` 不是响应式值了（并且不会在重新渲染时变化），所以它不需要作为依赖项。**如果你的 Effect 代码没有使用任何响应式值，它的依赖列表应该为空（`[]`）：**

```js {1,2,9}
const serverUrl = 'https://localhost:1234'; // 不再是响应式值了
const roomId = 'music'; // 不再是响应式值了

function ChatRoom() {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, []); // ✅ 所有依赖项都已声明
  // ...
}
```

[具有空依赖项的 Effect](/learn/lifecycle-of-reactive-effects#what-an-effect-with-empty-dependencies-means)不会在组件的 props 或 state 发生变化时重新运行。

<Pitfall>

如果你已有现有代码库，可能会有一些 Effects 像这样抑制了 linter：

```js {3-4}
useEffect(() => {
  // ...
  // 🔴 避免像这样抑制 linter：
  // eslint-ignore-next-line react-hooks/exhaustive-deps
}, []);
```

**当依赖项与代码不匹配时，很容易引入 bug。**通过抑制 linter，你是在对 React“撒谎”，隐瞒了你的 Effect 依赖哪些值。[相反，应该证明这些依赖是不必要的。](/learn/removing-effect-dependencies#removing-unnecessary-dependencies)

</Pitfall>

<Recipes titleText="传递响应式依赖的示例" titleId="examples-dependencies">

#### 传递依赖数组 {/*passing-a-dependency-array*/}

如果你指定了依赖项，你的 Effect 会在**初始提交之后，以及依赖项变化后的提交之后**运行。

```js {3}
useEffect(() => {
  // ...
}, [a, b]); // 如果 a 或 b 不同，就会再次运行
```

在下面的示例中，`serverUrl` 和 `roomId` 是[响应式值，](/learn/lifecycle-of-reactive-effects#effects-react-to-reactive-values)因此它们都必须被指定为依赖项。因此，在下拉菜单中选择不同的房间或编辑服务器 URL 输入框，都会导致聊天重新连接。不过，由于 `message` 没有在 Effect 中使用（因此它不是依赖项），编辑消息不会导致重新连接到聊天室。

<Sandpack>

```js
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');
  const [message, setMessage] = useState('');

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [serverUrl, roomId]);

  return (
    <>
      <label>
        Server URL:{' '}
        <input
          value={serverUrl}
          onChange={e => setServerUrl(e.target.value)}
        />
      </label>
      <h1>欢迎来到 {roomId} 房间！</h1>
      <label>
        你的消息：{' '}
        <input value={message} onChange={e => setMessage(e.target.value)} />
      </label>
    </>
  );
}

export default function App() {
  const [show, setShow] = useState(false);
  const [roomId, setRoomId] = useState('general');
  return (
    <>
      <label>
        选择聊天室：{' '}
        <select
          value={roomId}
          onChange={e => setRoomId(e.target.value)}
        >
          <option value="general">general</option>
          <option value="travel">travel</option>
          <option value="music">music</option>
        </select>
        <button onClick={() => setShow(!show)}>
          {show ? 'Close chat' : 'Open chat'}
        </button>
      </label>
      {show && <hr />}
      {show && <ChatRoom roomId={roomId}/>}
    </>
  );
}
```

```js src/chat.js
export function createConnection(serverUrl, roomId) {
  // 真实实现会实际连接到服务器
  return {
    connect() {
      console.log('✅ Connecting to "' + roomId + '" room at ' + serverUrl + '...');
    },
    disconnect() {
      console.log('❌ Disconnected from "' + roomId + '" room at ' + serverUrl);
    }
  };
}
```

```css
input { margin-bottom: 10px; }
button { margin-left: 5px; }
```

</Sandpack>

<Solution />

#### 传递空依赖数组 {/*passing-an-empty-dependency-array*/}

如果你的 Effect 确实没有使用任何响应式值，它只会在**初始提交之后运行。**

```js {3}
useEffect(() => {
  // ...
}, []); // 不会再次运行（开发环境中除外）
```

**即使依赖为空，setup 和 cleanup 也会[在开发环境中额外运行一次](/learn/synchronizing-with-effects#how-to-handle-the-effect-firing-twice-in-development)来帮助你发现 bug。**


在这个示例中，`serverUrl` 和 `roomId` 都是硬编码的。由于它们声明在组件外部，所以它们不是响应式值，因此也不是依赖项。依赖列表为空，所以 Effect 不会在重新渲染时重新运行。

<Sandpack>

```js
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

const serverUrl = 'https://localhost:1234';
const roomId = 'music';

function ChatRoom() {
  const [message, setMessage] = useState('');

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, []);

  return (
    <>
      <h1>欢迎来到 {roomId} 房间！</h1>
      <label>
        你的消息：{' '}
        <input value={message} onChange={e => setMessage(e.target.value)} />
      </label>
    </>
  );
}

export default function App() {
  const [show, setShow] = useState(false);
  return (
    <>
      <button onClick={() => setShow(!show)}>
        {show ? 'Close chat' : 'Open chat'}
      </button>
      {show && <hr />}
      {show && <ChatRoom />}
    </>
  );
}
```

```js src/chat.js
export function createConnection(serverUrl, roomId) {
  // 真实实现会实际连接到服务器
  return {
    connect() {
      console.log('✅ Connecting to "' + roomId + '" room at ' + serverUrl + '...');
    },
    disconnect() {
      console.log('❌ Disconnected from "' + roomId + '" room at ' + serverUrl);
    }
  };
}
```

</Sandpack>

<Solution />


#### 完全不传依赖数组 {/*passing-no-dependency-array-at-all*/}

如果你完全不传依赖数组，你的 Effect 会在组件的**每一次提交之后**运行。

```js {3}
useEffect(() => {
  // ...
}); // 总是会再次运行
```

在这个示例中，当你更改 `serverUrl` 和 `roomId` 时，Effect 会重新运行，这是合理的。不过，当你更改 `message` 时，它**也**会重新运行，这通常是不希望的。这就是为什么通常要指定依赖数组。

<Sandpack>

```js
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');
  const [message, setMessage] = useState('');

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }); // 完全没有依赖数组

  return (
    <>
      <label>
        Server URL:{' '}
        <input
          value={serverUrl}
          onChange={e => setServerUrl(e.target.value)}
        />
      </label>
      <h1>欢迎来到 {roomId} 房间！</h1>
      <label>
        你的消息：{' '}
        <input value={message} onChange={e => setMessage(e.target.value)} />
      </label>
    </>
  );
}

export default function App() {
  const [show, setShow] = useState(false);
  const [roomId, setRoomId] = useState('general');
  return (
    <>
      <label>
        选择聊天室：{' '}
        <select
          value={roomId}
          onChange={e => setRoomId(e.target.value)}
        >
          <option value="general">general</option>
          <option value="travel">travel</option>
          <option value="music">music</option>
        </select>
        <button onClick={() => setShow(!show)}>
          {show ? 'Close chat' : 'Open chat'}
        </button>
      </label>
      {show && <hr />}
      {show && <ChatRoom roomId={roomId}/>}
    </>
  );
}
```

```js src/chat.js
export function createConnection(serverUrl, roomId) {
  // 真实实现会实际连接到服务器
  return {
    connect() {
      console.log('✅ Connecting to "' + roomId + '" room at ' + serverUrl + '...');
    },
    disconnect() {
      console.log('❌ Disconnected from "' + roomId + '" room at ' + serverUrl);
    }
  };
}
```

```css
input { margin-bottom: 10px; }
button { margin-left: 5px; }
```

</Sandpack>

<Solution />

</Recipes>

---

### 基于 Effect 中之前的 state 更新 state {/*updating-state-based-on-previous-state-from-an-effect*/}

当你想基于 Effect 中之前的 state 来更新 state 时，可能会遇到一个问题：

```js {6,9}
function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const intervalId = setInterval(() => {
      setCount(count + 1); // 你希望每秒给计数器加 1...
    }, 1000)
    return () => clearInterval(intervalId);
  }, [count]); // 🚩 ... 但将 `count` 指定为依赖项会始终重置这个定时器。
  // ...
}
```

由于 `count` 是一个响应式值，它必须被指定在依赖列表中。然而，这会导致 Effect 在 `count` 每次变化时都重新执行 cleanup 和 setup。这并不理想。

要修复这个问题，可以[传入 `c => c + 1` 这个 state 更新器](/reference/react/useState#updating-state-based-on-the-previous-state)给 `setCount`：

<Sandpack>

```js
import { useState, useEffect } from 'react';

export default function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const intervalId = setInterval(() => {
      setCount(c => c + 1); // ✅ 传入一个 state 更新器
    }, 1000);
    return () => clearInterval(intervalId);
  }, []); // ✅ 现在 count 不是依赖项了

  return <h1>{count}</h1>;
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

现在你传入的是 `c => c + 1` 而不是 `count + 1`，[你的 Effect 就不再需要依赖 `count` 了。](/learn/removing-effect-dependencies#are-you-reading-some-state-to-calculate-the-next-state)这样一来，它就不需要在 `count` 每次变化时都重新 cleanup 和 setup 这个定时器了。

---


### 移除不必要的对象依赖 {/*removing-unnecessary-object-dependencies*/}

如果你的 Effect 依赖于渲染期间创建的对象或函数，它可能会运行得过于频繁。例如，这个 Effect 会在每次提交后重新连接，因为 `options` 对象[每次渲染都不同：](/learn/removing-effect-dependencies#does-some-reactive-value-change-unintentionally)

```js {6-9,12,15}
const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  const options = { // 🚩 这个对象在每次重新渲染时都会全新创建
    serverUrl: serverUrl,
    roomId: roomId
  };

  useEffect(() => {
    const connection = createConnection(options); // 它在 Effect 内部被使用
    connection.connect();
    return () => connection.disconnect();
  }, [options]); // 🚩 因此，这些依赖项在每次提交时都总是不同
  // ...
```

避免将渲染期间创建的对象作为依赖项。相反，把对象创建放到 Effect 内部：

<Sandpack>

```js
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  useEffect(() => {
    const options = {
      serverUrl: serverUrl,
      roomId: roomId
    };
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]);

  return (
    <>
      <h1>欢迎来到 {roomId} 聊天室！</h1>
      <input value={message} onChange={e => setMessage(e.target.value)} />
    </>
  );
}

export default function App() {
  const [roomId, setRoomId] = useState('general');
  return (
    <>
      <label>
        选择聊天室：{' '}
        <select
          value={roomId}
          onChange={e => setRoomId(e.target.value)}
        >
          <option value="general">general</option>
          <option value="travel">travel</option>
          <option value="music">music</option>
        </select>
      </label>
      <hr />
      <ChatRoom roomId={roomId} />
    </>
  );
}
```

```js src/chat.js
export function createConnection({ serverUrl, roomId }) {
  // 真实实现会实际连接到服务器
  return {
    connect() {
      console.log('✅ 正在连接到 "' + roomId + '" 聊天室，地址为 ' + serverUrl + '...');
    },
    disconnect() {
      console.log('❌ 已断开与 "' + roomId + '" 聊天室的连接，地址为 ' + serverUrl);
    }
  };
}
```

```css
input { display: block; margin-bottom: 20px; }
button { margin-left: 10px; }
```

</Sandpack>

现在你在 Effect 内部创建了 `options` 对象，Effect 本身只依赖于 `roomId` 字符串。

通过这个修复，在输入框中输入内容不会再重新连接聊天。与会被重新创建的对象不同，像 `roomId` 这样的字符串除非你把它设置为另一个值，否则不会改变。[阅读更多关于移除依赖项的内容。](/learn/removing-effect-dependencies)

---

### 移除不必要的函数依赖项 {/*removing-unnecessary-function-dependencies*/}

如果你的 Effect 依赖于在渲染期间创建的对象或函数，它可能会运行得过于频繁。例如，这个 Effect 会在每次提交后重新连接，因为 `createOptions` 函数[每次渲染都不同：](/learn/removing-effect-dependencies#does-some-reactive-value-change-unintentionally)

```js {4-9,12,16}
function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  function createOptions() { // 🚩 这个函数在每次重新渲染时都会全新创建
    return {
      serverUrl: serverUrl,
      roomId: roomId
    };
  }

  useEffect(() => {
    const options = createOptions(); // 它在 Effect 内部被使用
    const connection = createConnection();
    connection.connect();
    return () => connection.disconnect();
  }, [createOptions]); // 🚩 结果是，这些依赖项在提交时总是不同的
  // ...
```

每次重新渲染时从头创建一个函数本身不是问题。你不需要优化这一点。但是，如果你把它作为 Effect 的依赖项，就会导致你的 Effect 在每次提交后重新运行。

避免将渲染期间创建的函数作为依赖项。相反，把它声明在 Effect 内部：

<Sandpack>

```js
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  useEffect(() => {
    function createOptions() {
      return {
        serverUrl: serverUrl,
        roomId: roomId
      };
    }

    const options = createOptions();
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]);

  return (
    <>
      <h1>欢迎来到 {roomId} 聊天室！</h1>
      <input value={message} onChange={e => setMessage(e.target.value)} />
    </>
  );
}

export default function App() {
  const [roomId, setRoomId] = useState('general');
  return (
    <>
      <label>
        选择聊天室：{' '}
        <select
          value={roomId}
          onChange={e => setRoomId(e.target.value)}
        >
          <option value="general">general</option>
          <option value="travel">travel</option>
          <option value="music">music</option>
        </select>
      </label>
      <hr />
      <ChatRoom roomId={roomId} />
    </>
  );
}
```

```js src/chat.js
export function createConnection({ serverUrl, roomId }) {
  // 真实实现会实际连接到服务器
  return {
    connect() {
      console.log('✅ 正在连接到 "' + roomId + '" 聊天室，地址为 ' + serverUrl + '...');
    },
    disconnect() {
      console.log('❌ 已断开与 "' + roomId + '" 聊天室的连接，地址为 ' + serverUrl);
    }
  };
}
```

```css
input { display: block; margin-bottom: 20px; }
button { margin-left: 10px; }
```

</Sandpack>

现在你把 `createOptions` 函数定义在 Effect 内部，Effect 本身只依赖于 `roomId` 字符串。通过这个修复，在输入框中输入内容不会再重新连接聊天。与会被重新创建的函数不同，像 `roomId` 这样的字符串除非你把它设置为另一个值，否则不会改变。[阅读更多关于移除依赖项的内容。](/learn/removing-effect-dependencies)

---

### 从 Effect 中读取最新的 props 和 state {/*reading-the-latest-props-and-state-from-an-effect*/}

默认情况下，当你从 Effect 中读取一个响应式值时，必须把它添加为依赖项。这能确保你的 Effect 会对该值的每次变化“做出反应”。对于大多数依赖项来说，这就是你想要的行为。

**不过，有时你会希望从 Effect 中读取 *最新* 的 props 和 state，但又不想对它们“做出反应”。** 例如，假设你想在每次页面访问时记录购物车中的商品数量：

```js {3}
function Page({ url, shoppingCart }) {
  useEffect(() => {
    logVisit(url, shoppingCart.length);
  }, [url, shoppingCart]); // ✅ 声明了所有依赖项
  // ...
}
```

**如果你想在每次 `url` 变化后记录一次新的页面访问，但 *不* 希望仅仅因为 `shoppingCart` 变化就记录呢？** 你不能在不破坏[响应性规则](#specifying-reactive-dependencies)的前提下把 `shoppingCart` 从依赖项中排除。不过，你可以表达出你*不希望*某段代码对变化“做出反应”，即使它是在 Effect 内部被调用的。使用 [`useEffectEvent`](/reference/react/useEffectEvent) Hook [声明一个 *Effect Event*](/learn/separating-events-from-effects#declaring-an-effect-event)，并把读取 `shoppingCart` 的代码移到其中：

```js {2-4,7,8}
function Page({ url, shoppingCart }) {
  const onVisit = useEffectEvent(visitedUrl => {
    logVisit(visitedUrl, shoppingCart.length)
  });

  useEffect(() => {
    onVisit(url);
  }, [url]); // ✅ 声明了所有依赖项
  // ...
}
```

**Effect Event 不是响应式的，必须始终从你的 Effect 依赖项中省略。** 这正是它们能让你把非响应式代码（在其中你可以读取某些 props 和 state 的最新值）放进去的原因。通过在 `onVisit` 内部读取 `shoppingCart`，你可以确保 `shoppingCart` 不会让你的 Effect 重新运行。

[阅读更多关于 Effect Event 如何让你分离响应式和非响应式代码的内容。](/learn/separating-events-from-effects#reading-latest-props-and-state-with-effect-events)


---

### 在服务端和客户端显示不同的内容 {/*displaying-different-content-on-the-server-and-the-client*/}

如果你的应用使用服务器渲染（无论是[直接使用](/reference/react-dom/server)，还是通过[框架](/learn/creating-a-react-app#full-stack-frameworks)），你的组件会在两个不同的环境中渲染。在服务器上，它会渲染以生成初始 HTML。在客户端上，React 会再次运行渲染代码，以便它可以把事件处理器附加到该 HTML 上。这就是为什么要让[水合](/reference/react-dom/client/hydrateRoot#hydrating-server-rendered-html)正常工作，你的初始渲染输出在客户端和服务器上必须完全一致。

在少数情况下，你可能需要在客户端显示不同的内容。例如，如果你的应用从 [`localStorage`](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage) 中读取某些数据，那么它不可能在服务器上完成。下面是你可以如何实现这一点：

{/* TODO(@poteto) - 调查 react compiler 验证中潜在的误报 */}
```js {expectedErrors: {'react-compiler': [5]}}
function MyComponent() {
  const [didMount, setDidMount] = useState(false);

  useEffect(() => {
    setDidMount(true);
  }, []);

  if (didMount) {
    // ... 返回仅客户端的 JSX ...
  }  else {
    // ... 返回初始 JSX ...
  }
}
```

在应用加载期间，用户会看到初始渲染输出。然后，当它加载并完成水合后，你的 Effect 会运行并将 `didMount` 设为 `true`，触发一次重新渲染。这将切换为仅客户端的渲染输出。Effect 不会在服务器上运行，所以这就是为什么在初始服务器渲染期间 `didMount` 是 `false`。

请谨慎使用这种模式。请记住，网速较慢的用户会在相当长的一段时间内看到初始内容——可能是很多秒——所以你不希望对组件外观做出突兀的变化。在很多情况下，你可以通过使用 CSS 有条件地显示不同内容来避免这种需求。

---

## 故障排除 {/*troubleshooting*/}

### 我的 Effect 在组件挂载时运行了两次 {/*my-effect-runs-twice-when-the-component-mounts*/}

当 Strict Mode 开启时，在开发环境中，React 会在实际设置之前额外执行一次设置和清理。

这是一个压力测试，用于验证你的 Effect 逻辑是否正确实现。如果这导致了可见问题，说明你的清理函数缺少了一些逻辑。清理函数应该停止或撤销设置函数所做的一切。经验法则是，用户不应该能够区分“只调用一次 setup”（如生产环境）和“setup → cleanup → setup”序列（如开发环境）。

阅读更多关于[这如何帮助发现 bug](/learn/synchronizing-with-effects#step-3-add-cleanup-if-needed)以及[如何修复你的逻辑。](/learn/synchronizing-with-effects#how-to-handle-the-effect-firing-twice-in-development)

---

### 我的 Effect 在每次重新渲染后运行 {/*my-effect-runs-after-every-re-render*/}

首先，检查你是否忘记指定依赖数组：

```js {3}
useEffect(() => {
  // ...
}); // 🚩 没有依赖数组：每次提交后都会重新运行！
```

如果你已经指定了依赖数组，但你的 Effect 仍然在循环中重新运行，那是因为你的某个依赖项在每次重新渲染时都不同。

你可以通过手动将依赖项打印到控制台来调试这个问题：

```js {5}
  useEffect(() => {
    // ..
  }, [serverUrl, roomId]);

  console.log([serverUrl, roomId]);
```

然后你可以在控制台中右键点击不同重新渲染对应的数组，并为它们都选择“存储为全局变量”。假设第一个被保存为 `temp1`，第二个被保存为 `temp2`，那么你就可以在浏览器控制台中检查两个数组中的每个依赖项是否相同：

```js
Object.is(temp1[0], temp2[0]); // 第一个依赖项在两个数组之间是否相同？
Object.is(temp1[1], temp2[1]); // 第二个依赖项是否相同？
Object.is(temp1[2], temp2[2]); // ……依此类推，检查每个依赖项……
```

当你找到每次重新渲染都不同的依赖项后，通常可以用以下几种方式之一修复它：

- [基于 Effect 中上一次状态来更新 state](#updating-state-based-on-previous-state-from-an-effect)
- [移除不必要的对象依赖项](#removing-unnecessary-object-dependencies)
- [移除不必要的函数依赖项](#removing-unnecessary-function-dependencies)
- [从 Effect 中读取最新的 props 和 state](#reading-the-latest-props-and-state-from-an-effect)

作为最后的手段（如果这些方法都没有帮助），可以用 [`useMemo`](/reference/react/useMemo#memoizing-a-dependency-of-another-hook) 或 [`useCallback`](/reference/react/useCallback#preventing-an-effect-from-firing-too-often)（针对函数）包裹它的创建过程。

---

### 我的 Effect 一直在无限循环中重新运行 {/*my-effect-keeps-re-running-in-an-infinite-cycle*/}

如果你的 Effect 在一个无限循环中运行，必须同时满足以下两点：

- 你的 Effect 正在更新某些 state。
- 该 state 导致重新渲染，而这又使 Effect 的依赖项发生变化。

在开始修复问题之前，先问问自己你的 Effect 是否正在连接某个外部系统（例如 DOM、网络、第三方小部件等）。你的 Effect 为什么需要设置 state？它是在与那个外部系统同步吗？还是你试图用它来管理应用的数据流？

如果没有外部系统，可以考虑是否[完全移除 Effect](/learn/you-might-not-need-an-effect)会让你的逻辑更简单。

如果你确实是在与某个外部系统同步，请思考为什么以及在什么条件下你的 Effect 应该更新 state。是否有某些变化影响了组件的视觉输出？如果你需要跟踪一些不会用于渲染的数据，那么一个 [ref](/reference/react/useRef#referencing-a-value-with-a-ref)（不会触发重新渲染）可能更合适。确认你的 Effect 没有比必要时更多地更新 state（并触发重新渲染）。

最后，如果你的 Effect 在正确的时间更新了 state，但仍然存在循环，那是因为这次 state 更新导致了 Effect 的某个依赖项发生变化。[阅读如何调试依赖项变化。](/reference/react/useEffect#my-effect-runs-after-every-re-render)

---

### 我的清理逻辑在组件没有卸载的情况下也运行了 {/*my-cleanup-logic-runs-even-though-my-component-didnt-unmount*/}

清理函数不仅会在卸载时运行，还会在每次依赖项变化导致重新渲染之前运行。此外，在开发环境中，React 会在组件挂载后立即额外执行一次 setup+cleanup。[运行 setup+cleanup。](#my-effect-runs-twice-when-the-component-mounts)

如果你有清理代码却没有相应的设置代码，这通常是一个代码异味：

```js {2-5}
useEffect(() => {
  // 🔴 避免：只有清理逻辑，没有对应的设置逻辑
  return () => {
    doSomething();
  };
}, []);
```

你的清理逻辑应该与设置逻辑“对称”，并且应该停止或撤销设置所做的一切：

```js {2-3,5}
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [serverUrl, roomId]);
```

[了解 Effect 生命周期与组件生命周期的不同。](/learn/lifecycle-of-reactive-effects#the-lifecycle-of-an-effect)

---

### 我的 Effect 做了某些视觉上的事，并且我看到在它运行前有闪烁 {/*my-effect-does-something-visual-and-i-see-a-flicker-before-it-runs*/}

如果你的 Effect 必须阻止浏览器[绘制屏幕，](/learn/render-and-commit#epilogue-browser-paint)请用 [`useLayoutEffect`](/reference/react/useLayoutEffect) 替换 `useEffect`。请注意，**绝大多数 Effect 都不需要这样做。** 只有在必须让 Effect 在浏览器绘制之前运行时才需要这样做：例如，在用户看到工具提示之前测量并定位它。
