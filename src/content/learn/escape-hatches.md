---
title: 逃生通道
---

<Intro>

你的某些组件可能需要控制并同步 React 之外的系统。例如，你可能需要使用浏览器 API 聚焦一个输入框，在不使用 React 实现的视频播放器中播放和暂停视频，或者连接到远程服务器并监听消息。在本章中，你将学习这些允许你“走出” React 并连接到外部系统的逃生通道。你应用中的大多数逻辑和数据流都不应该依赖这些特性。

</Intro>

<YouWillLearn isChapter={true}>

* [如何在不重新渲染的情况下“记住”信息](/learn/referencing-values-with-refs)
* [如何访问由 React 管理的 DOM 元素](/learn/manipulating-the-dom-with-refs)
* [如何将组件与外部系统同步](/learn/synchronizing-with-effects)
* [如何从组件中移除不必要的 Effect](/learn/you-might-not-need-an-effect)
* [Effect 的生命周期与组件的生命周期有何不同](/learn/lifecycle-of-reactive-effects)
* [如何防止某些值重新触发 Effects](/learn/separating-events-from-effects)
* [如何让你的 Effect 更少地重新运行](/learn/removing-effect-dependencies)
* [如何在组件之间共享逻辑](/learn/reusing-logic-with-custom-hooks)

</YouWillLearn>

## 使用 refs 引用值 {/*referencing-values-with-refs*/}

当你希望组件“记住”某些信息，但又不希望这些信息[触发新的渲染](/learn/render-and-commit)时，你可以使用 *ref*：

```js
const ref = useRef(0);
```

和 state 一样，refs 会在 React 的重新渲染之间保留。不过，设置 state 会重新渲染组件，而修改 ref 不会！你可以通过 `ref.current` 属性访问该 ref 的当前值。

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

ref 就像是组件里的一个秘密口袋，React 不会跟踪它。例如，你可以使用 ref 来存储[超时 ID](https://developer.mozilla.org/en-US/docs/Web/API/setTimeout#return_value)、[DOM 元素](https://developer.mozilla.org/en-US/docs/Web/API/Element)以及其他不会影响组件渲染输出的对象。

<LearnMore path="/learn/referencing-values-with-refs">

阅读 **[使用 refs 引用值](/learn/referencing-values-with-refs)**，了解如何使用 ref 来记住信息。

</LearnMore>

## 使用 refs 操作 DOM {/*manipulating-the-dom-with-refs*/}

React 会自动更新 DOM 以匹配你的渲染输出，因此你的组件通常不需要直接操作它。不过，有时你可能需要访问由 React 管理的 DOM 元素——例如，为一个节点聚焦、滚动到它，或者测量它的大小和位置。在 React 中没有内置方式来完成这些操作，所以你需要一个指向 DOM 节点的 ref。例如，点击按钮会使用 ref 聚焦输入框：

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

<LearnMore path="/learn/manipulating-the-dom-with-refs">

阅读 **[使用 refs 操作 DOM](/learn/manipulating-the-dom-with-refs)**，了解如何访问由 React 管理的 DOM 元素。

</LearnMore>

## 使用 Effects 同步 {/*synchronizing-with-effects*/}

某些组件需要与外部系统同步。例如，你可能希望基于 React state 控制一个非 React 组件，建立服务器连接，或者在组件出现在屏幕上时发送分析日志。与允许你处理特定事件的事件处理器不同，*Effects* 让你在渲染后运行一些代码。使用它们可以让你的组件与 React 之外的系统同步。

点击几次播放/暂停，看看视频播放器如何与 `isPlaying` prop 值保持同步：

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
  }, [isPlaying]);

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

许多 Effect 也会在结束时“清理”自身。例如，一个用于建立到聊天服务器连接的 Effect 应该返回一个 *cleanup function*，告诉 React 如何让你的组件断开与该服务器的连接：

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
  // 真正的实现实际上会连接到服务器
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

在开发环境中，React 会立即额外运行一次你的 Effect 并清理它。这就是你会看到 `"✅ 正在连接..."` 打印两次的原因。这样可以确保你不会忘记实现清理函数。

<LearnMore path="/learn/synchronizing-with-effects">

阅读 **[使用 Effects 同步](/learn/synchronizing-with-effects)**，了解如何将组件与外部系统同步。

</LearnMore>

## 你可能并不需要 Effect {/*you-might-not-need-an-effect*/}

Effects 是 React 范式中的一种逃生通道。它们让你“走出” React，并将组件与某个外部系统同步。如果没有涉及外部系统（例如，当某些 props 或 state 变化时更新组件的 state），你通常不需要 Effect。移除不必要的 Effects 会让你的代码更易理解、运行更快，并且更不容易出错。

在以下两种常见情况下，你不需要 Effects：
- **你不需要用 Effects 来转换用于渲染的数据。**
- **你不需要用 Effects 来处理用户事件。**

例如，你不需要用 Effect 来根据其他 state 调整某些 state：

```js {expectedErrors: {'react-compiler': [8]}} {5-9}
function Form() {
  const [firstName, setFirstName] = useState('Taylor');
  const [lastName, setLastName] = useState('Swift');

  // 🔴 避免：冗余的 state 和不必要的 Effect
  const [fullName, setFullName] = useState('');
  useEffect(() => {
    setFullName(firstName + ' ' + lastName);
  }, [firstName, lastName]);
  // ...
}
```

相反，请在渲染时尽可能多地计算：

```js {4-5}
function Form() {
  const [firstName, setFirstName] = useState('Taylor');
  const [lastName, setLastName] = useState('Swift');
  // ✅ 好：在渲染期间计算
  const fullName = firstName + ' ' + lastName;
  // ...
}
```

不过，你*确实*需要 Effects 来与外部系统同步。

<LearnMore path="/learn/you-might-not-need-an-effect">

阅读 **[你可能并不需要 Effect](/learn/you-might-not-need-an-effect)**，了解如何移除不必要的 Effects。

</LearnMore>

## 响应式 effects 的生命周期 {/*lifecycle-of-reactive-effects*/}

Effects 的生命周期与组件不同。组件可能挂载、更新或卸载。Effect 只能做两件事：开始同步某些内容，以及之后停止同步它。如果你的 Effect 依赖会随时间变化的 props 和 state，那么这个循环可能会发生多次。

这个 Effect 依赖于 `roomId` prop 的值。Props 是 *响应式值*，这意味着它们可能在重新渲染时发生变化。请注意，如果 `roomId` 改变，Effect 会 *重新同步*（并重新连接到服务器）：

<Sandpack>

```js
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]);

  return <h1>Welcome to the {roomId} room!</h1>;
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
export function createConnection(serverUrl, roomId) {
  // 真正的实现实际上会连接到服务器
  return {
    connect() {
      console.log('✅ 正在连接到 "' + roomId + '" 房间，地址 ' + serverUrl + '...');
    },
    disconnect() {
      console.log('❌ 已从 "' + roomId + '" 房间断开，地址 ' + serverUrl);
    }
  };
}
```

```css
input { display: block; margin-bottom: 20px; }
button { margin-left: 10px; }
```

</Sandpack>

React 提供了一条 lint 规则来检查你是否正确指定了 Effect 的依赖项。如果你在上面的示例中忘记在依赖列表里指定 `roomId`，lint 工具会自动发现这个 bug。

<LearnMore path="/learn/lifecycle-of-reactive-effects">

阅读 **[响应式事件的生命周期](/learn/lifecycle-of-reactive-effects)**，了解 Effect 的生命周期与组件的生命周期有何不同。

</LearnMore>

## 将事件与 Effects 分离 {/*separating-events-from-effects*/}

只有在你再次执行同样的交互时，事件处理函数才会重新运行。与事件处理函数不同，如果 Effect 读取的任何值（例如 props 或 state）与上次渲染时不同，Effect 就会重新同步。有时，你希望两种行为兼而有之：让某个 Effect 在某些值变化时重新运行，但在其他值变化时不重新运行。

Effects 中的所有代码都是*响应式的。* 如果它读取的某个响应式值因为重新渲染而发生变化，它就会再次运行。例如，如果 `roomId` 或 `theme` 发生变化，这个 Effect 就会重新连接到聊天：

<Sandpack>

```json package.json hidden
{
  "dependencies": {
    "react": "latest",
    "react-dom": "latest",
    "react-scripts": "latest",
    "toastify-js": "1.12.0"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject"
  }
}
```

```js
import { useState, useEffect } from 'react';
import { createConnection, sendMessage } from './chat.js';
import { showNotification } from './notifications.js';

const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId, theme }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.on('connected', () => {
      showNotification('已连接！', theme);
    });
    connection.connect();
    return () => connection.disconnect();
  }, [roomId, theme]);

  return <h1>欢迎来到 {roomId} 房间！</h1>
}

export default function App() {
  const [roomId, setRoomId] = useState('general');
  const [isDark, setIsDark] = useState(false);
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
      <label>
        <input
          type="checkbox"
          checked={isDark}
          onChange={e => setIsDark(e.target.checked)}
        />
        使用深色主题
      </label>
      <hr />
      <ChatRoom
        roomId={roomId}
        theme={isDark ? 'dark' : 'light'}
      />
    </>
  );
}
```

```js src/chat.js
export function createConnection(serverUrl, roomId) {
  // 真实实现会实际连接到服务器
  let connectedCallback;
  let timeout;
  return {
    connect() {
      timeout = setTimeout(() => {
        if (connectedCallback) {
          connectedCallback();
        }
      }, 100);
    },
    on(event, callback) {
      if (connectedCallback) {
        throw Error('Cannot add the handler twice.');
      }
      if (event !== 'connected') {
        throw Error('Only "connected" event is supported.');
      }
      connectedCallback = callback;
    },
    disconnect() {
      clearTimeout(timeout);
    }
  };
}
```

```js src/notifications.js
import Toastify from 'toastify-js';
import 'toastify-js/src/toastify.css';

export function showNotification(message, theme) {
  Toastify({
    text: message,
    duration: 2000,
    gravity: 'top',
    position: 'right',
    style: {
      background: theme === 'dark' ? 'black' : 'white',
      color: theme === 'dark' ? 'white' : 'black',
    },
  }).showToast();
}
```

```css
label { display: block; margin-top: 10px; }
```

</Sandpack>

这并不理想。你希望只有在 `roomId` 发生变化时才重新连接聊天。切换 `theme` 不应该让聊天重新连接！把读取 `theme` 的代码从 Effect 中移到一个 *Effect Event* 中：

<Sandpack>

```json package.json hidden
{
  "dependencies": {
    "react": "latest",
    "react-dom": "latest",
    "react-scripts": "latest",
    "toastify-js": "1.12.0"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject"
  }
}
```

```js
import { useState, useEffect } from 'react';
import { useEffectEvent } from 'react';
import { createConnection, sendMessage } from './chat.js';
import { showNotification } from './notifications.js';

const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId, theme }) {
  const onConnected = useEffectEvent(() => {
    showNotification('已连接！', theme);
  });

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.on('connected', () => {
      onConnected();
    });
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]);

  return <h1>欢迎来到 {roomId} 房间！</h1>
}

export default function App() {
  const [roomId, setRoomId] = useState('general');
  const [isDark, setIsDark] = useState(false);
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
      <label>
        <input
          type="checkbox"
          checked={isDark}
          onChange={e => setIsDark(e.target.checked)}
        />
        使用深色主题
      </label>
      <hr />
      <ChatRoom
        roomId={roomId}
        theme={isDark ? 'dark' : 'light'}
      />
    </>
  );
}
```

```js src/chat.js
export function createConnection(serverUrl, roomId) {
  // 真实实现会实际连接到服务器
  let connectedCallback;
  let timeout;
  return {
    connect() {
      timeout = setTimeout(() => {
        if (connectedCallback) {
          connectedCallback();
        }
      }, 100);
    },
    on(event, callback) {
      if (connectedCallback) {
        throw Error('Cannot add the handler twice.');
      }
      if (event !== 'connected') {
        throw Error('Only "connected" event is supported.');
      }
      connectedCallback = callback;
    },
    disconnect() {
      clearTimeout(timeout);
    }
  };
}
```

```js src/notifications.js hidden
import Toastify from 'toastify-js';
import 'toastify-js/src/toastify.css';

export function showNotification(message, theme) {
  Toastify({
    text: message,
    duration: 2000,
    gravity: 'top',
    position: 'right',
    style: {
      background: theme === 'dark' ? 'black' : 'white',
      color: theme === 'dark' ? 'white' : 'black',
    },
  }).showToast();
}
```

```css
label { display: block; margin-top: 10px; }
```

</Sandpack>

Effect Event 中的代码不是响应式的，所以更改 `theme` 不再会让你的 Effect 重新连接。

<LearnMore path="/learn/separating-events-from-effects">

阅读 **[将事件与 Effects 分离](/learn/separating-events-from-effects)**，了解如何防止某些值重新触发 Effects。

</LearnMore>

## 移除 Effect 依赖项 {/*removing-effect-dependencies*/}

当你编写 Effect 时，linter 会检查你是否已经把 Effect 读取的每个响应式值（如 props 和 state）都包含在 Effect 依赖项列表中。这可以确保你的 Effect 始终与组件的最新 props 和 state 保持同步。不必要的依赖项可能会导致 Effect 运行过于频繁，甚至创建无限循环。移除它们的方法取决于具体情况。

例如，这个 Effect 依赖于 `options` 对象，而该对象会在你每次编辑输入框时重新创建：

<Sandpack>

```js
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  const options = {
    serverUrl: serverUrl,
    roomId: roomId
  };

  useEffect(() => {
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [options]);

  return (
    <>
      <h1>欢迎来到 {roomId} 房间！</h1>
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
      console.log('✅ 连接到 "' + roomId + '" 房间，位于 ' + serverUrl + '...');
    },
    disconnect() {
      console.log('❌ 已从 "' + roomId + '" 房间断开，位于 ' + serverUrl);
    }
  };
}
```

```css
input { display: block; margin-bottom: 20px; }
button { margin-left: 10px; }
```

</Sandpack>

你不希望每次开始在聊天室里输入消息时，聊天都重新连接。要修复这个问题，把 `options` 对象的创建移到 Effect 内部，这样 Effect 就只依赖于 `roomId` 字符串：

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
      <h1>欢迎来到 {roomId} 房间！</h1>
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
      console.log('✅ 连接到 "' + roomId + '" 房间，位于 ' + serverUrl + '...');
    },
    disconnect() {
      console.log('❌ 已从 "' + roomId + '" 房间断开，位于 ' + serverUrl);
    }
  };
}
```

```css
input { display: block; margin-bottom: 20px; }
button { margin-left: 10px; }
```

</Sandpack>

请注意，你并不是先通过编辑依赖项列表来移除 `options` 依赖项的。那样做是错误的。相反，你修改了周围的代码，使这个依赖项变得*不再必要。* 可以把依赖项列表看作是 Effect 代码所使用的所有响应式值的列表。你并不是有意选择把什么放进去。这个列表是在描述你的代码。要改变依赖项列表，就修改代码。

<LearnMore path="/learn/removing-effect-dependencies">

阅读 **[移除 Effect 依赖项](/learn/removing-effect-dependencies)**，了解如何让你的 Effect 更少地重新运行。

</LearnMore>

## 使用自定义 Hook 复用逻辑 {/*reusing-logic-with-custom-hooks*/}

React 自带了一些内置 Hook，比如 `useState`、`useContext` 和 `useEffect`。有时，你可能会希望有一个用于更具体目的的 Hook：例如，用于获取数据、跟踪用户是否在线，或者连接到聊天室。为此，你可以根据应用的需要创建自己的 Hook。

在这个示例中，`usePointerPosition` 自定义 Hook 跟踪光标位置，而 `useDelayedValue` 自定义 Hook 会返回一个比你传入的值“落后”若干毫秒的值。将光标移动到沙盒预览区域上方，查看一串跟随光标移动的点：

<Sandpack>

```js
import { usePointerPosition } from './usePointerPosition.js';
import { useDelayedValue } from './useDelayedValue.js';

export default function Canvas() {
  const pos1 = usePointerPosition();
  const pos2 = useDelayedValue(pos1, 100);
  const pos3 = useDelayedValue(pos2, 200);
  const pos4 = useDelayedValue(pos3, 100);
  const pos5 = useDelayedValue(pos4, 50);
  return (
    <>
      <Dot position={pos1} opacity={1} />
      <Dot position={pos2} opacity={0.8} />
      <Dot position={pos3} opacity={0.6} />
      <Dot position={pos4} opacity={0.4} />
      <Dot position={pos5} opacity={0.2} />
    </>
  );
}

function Dot({ position, opacity }) {
  return (
    <div style={{
      position: 'absolute',
      backgroundColor: 'pink',
      borderRadius: '50%',
      opacity,
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

```js src/usePointerPosition.js
import { useState, useEffect } from 'react';

export function usePointerPosition() {
  const [position, setPosition] = useState({ x: 0, y: 0 });
  useEffect(() => {
    function handleMove(e) {
      setPosition({ x: e.clientX, y: e.clientY });
    }
    window.addEventListener('pointermove', handleMove);
    return () => window.removeEventListener('pointermove', handleMove);
  }, []);
  return position;
}
```

```js src/useDelayedValue.js
import { useState, useEffect } from 'react';

export function useDelayedValue(value, delay) {
  const [delayedValue, setDelayedValue] = useState(value);

  useEffect(() => {
    setTimeout(() => {
      setDelayedValue(value);
    }, delay);
  }, [value, delay]);

  return delayedValue;
}
```

```css
body { min-height: 300px; }
```

</Sandpack>

你可以创建自定义 Hook，将它们组合在一起，在它们之间传递数据，并在组件之间复用它们。随着应用的增长，你将会手写更少的 Effect，因为你可以复用已经编写好的自定义 Hook。React 社区还维护了许多优秀的自定义 Hook。

<LearnMore path="/learn/reusing-logic-with-custom-hooks">

阅读 **[使用自定义 Hook 复用逻辑](/learn/reusing-logic-with-custom-hooks)**，了解如何在组件之间共享逻辑。

</LearnMore>

## 接下来是什么？ {/*whats-next*/}

前往 [使用 Refs 引用值](/learn/referencing-values-with-refs) 开始逐页阅读本章！
