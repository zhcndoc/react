---
title: '将事件与 Effect 分离'
---

<Intro>

事件处理函数只有在你再次执行同样的交互时才会重新运行。与事件处理函数不同，如果某个 Effect 读取的值（比如 props 或 state 变量）与上一次渲染时不同，Effect 会重新同步。有时候，你也会希望同时具备这两种行为：一个会根据某些值重新运行、但不会因其他值而重新运行的 Effect。本页将教你如何做到这一点。

</Intro>

<YouWillLearn>

- 如何在事件处理函数和 Effect 之间做选择
- 为什么 Effect 是响应式的，而事件处理函数不是
- 当你希望 Effect 中的一部分代码不具备响应性时该怎么做
- 什么是 Effect Event，以及如何从 Effect 中提取它们
- 如何使用 Effect Event 从 Effect 中读取最新的 props 和 state

</YouWillLearn>

## 在事件处理函数和 Effect 之间做选择 {/*choosing-between-event-handlers-and-effects*/}

首先，让我们回顾一下事件处理函数和 Effect 的区别。

假设你正在实现一个聊天室组件。你的需求如下：

1. 组件应当自动连接到所选的聊天室。
1. 当你点击 “Send” 按钮时，它应该向聊天发送一条消息。

假设你已经为它们实现好了代码，但不确定该放在哪里。应该使用事件处理函数还是 Effect？每次你需要回答这个问题时，都要考虑 [*为什么* 这段代码需要运行。](/learn/synchronizing-with-effects#what-are-effects-and-how-are-they-different-from-events)

### 事件处理函数会响应特定交互而运行 {/*event-handlers-run-in-response-to-specific-interactions*/}

从用户的角度看，发送消息应该是*因为*点击了特定的 “Send” 按钮才发生的。如果你在其他时间或出于其他原因发送他们的消息，用户会相当不满。这就是为什么发送消息应该是一个事件处理函数。事件处理函数让你处理特定交互：

```js {4-6}
function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');
  // ...
  function handleSendClick() {
    sendMessage(message);
  }
  // ...
  return (
    <>
      <input value={message} onChange={e => setMessage(e.target.value)} />
      <button onClick={handleSendClick}>发送</button>
    </>
  );
}
```

有了事件处理函数，你可以确定 `sendMessage(message)` *只会*在用户按下按钮时运行。

### Effect 会在需要同步时运行 {/*effects-run-whenever-synchronization-is-needed*/}

还记得你也需要保持组件连接到聊天室。那段代码应该放在哪里？

运行这段代码的*原因*并不是某个特定交互。用户是如何、为什么进入聊天室界面的并不重要。现在他们正在查看它并可能与之交互，组件需要保持与所选聊天服务器的连接。即使聊天室组件是应用的初始界面，且用户根本没有进行任何交互，你*仍然*需要连接。这就是为什么它应该是一个 Effect：

```js {3-9}
function ChatRoom({ roomId }) {
  // ...
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [roomId]);
  // ...
}
```

有了这段代码，你可以确定始终有一个到当前所选聊天服务器的活动连接，*无论*用户执行了什么具体交互。无论用户只是打开了你的应用、选择了不同的房间，还是切换到别的界面再回来，你的 Effect 都会确保组件始终与当前所选房间*保持同步*，并且会在必要时[重新连接。](/learn/lifecycle-of-reactive-effects#why-synchronization-may-need-to-happen-more-than-once)

<Sandpack>

```js
import { useState, useEffect } from 'react';
import { createConnection, sendMessage } from './chat.js';

const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]);

  function handleSendClick() {
    sendMessage(message);
  }

  return (
    <>
      <h1>欢迎来到 {roomId} 房间！</h1>
      <input value={message} onChange={e => setMessage(e.target.value)} />
      <button onClick={handleSendClick}>发送</button>
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
        {show ? '关闭聊天' : '打开聊天'}
      </button>
      {show && <hr />}
      {show && <ChatRoom roomId={roomId} />}
    </>
  );
}
```

```js src/chat.js
export function sendMessage(message) {
  console.log('🔵 你发送了：' + message);
}

export function createConnection(serverUrl, roomId) {
  // 真实实现会实际连接到服务器
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
input, select { margin-right: 20px; }
```

</Sandpack>

## 响应式值与响应式逻辑 {/*reactive-values-and-reactive-logic*/}

直观地说，你可以认为事件处理函数总是被“手动”触发的，例如通过点击按钮。另一方面，Effect 是“自动”的：它们会按需运行和重新运行，以保持同步。

不过，我们可以更精确地理解这一点。

在组件函数体内声明的 props、state 和变量被称为<CodeStep step={2}>响应式值</CodeStep>。在这个例子中，`serverUrl` 不是响应式值，而 `roomId` 和 `message` 是。它们参与渲染数据流：

```js [[2, 3, "roomId"], [2, 4, "message"]]
const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  // ...
}
```

像这样的响应式值可能会因为重新渲染而变化。例如，用户可能编辑 `message`，或者在下拉菜单中选择不同的 `roomId`。事件处理函数和 Effect 对变化的响应方式不同：

- **事件处理函数中的逻辑*不是响应式的。*** 除非用户再次执行同样的交互（例如点击）否则它不会再次运行。事件处理函数可以读取响应式值，而不会对它们的变化“做出反应”。
- **Effect 中的逻辑是*响应式的。*** 如果你的 Effect 读取了一个响应式值，[你必须将它指定为依赖项。](/learn/lifecycle-of-reactive-effects#effects-react-to-reactive-values) 然后，如果一次重新渲染导致该值变化，React 会使用新值重新运行你的 Effect 逻辑。

让我们回到前面的例子来说明这种区别。

### 事件处理函数中的逻辑不是响应式的 {/*logic-inside-event-handlers-is-not-reactive*/}

看看这行代码。这里的逻辑应该是响应式的吗？

```js [[2, 2, "message"]]
    // ...
    sendMessage(message);
    // ...
```

从用户的角度看，**`message` 的变化并不意味着他们想要发送一条消息。** 它只意味着用户正在输入。换句话说，发送消息的逻辑不应该是响应式的。它不应该仅仅因为<CodeStep step={2}>响应式值</CodeStep>变化了就再次运行。这就是它属于事件处理函数的原因：

```js {2}
  function handleSendClick() {
    sendMessage(message);
  }
```

事件处理函数不是响应式的，所以 `sendMessage(message)` 只会在用户点击 Send 按钮时运行。

### Effect 中的逻辑是响应式的 {/*logic-inside-effects-is-reactive*/}

现在让我们回到这些代码行：

```js [[2, 2, "roomId"]]
    // ...
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    // ...
```

从用户的角度看，**`roomId` 的变化确实意味着他们想要连接到另一个房间。** 换句话说，连接房间的逻辑应该是响应式的。你*希望*这些代码行能“跟上”<CodeStep step={2}>响应式值</CodeStep>，并在该值变化时重新运行。这就是它属于 Effect 的原因：

```js {2-3}
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect()
    };
  }, [roomId]);
```

Effect 是响应式的，所以 `createConnection(serverUrl, roomId)` 和 `connection.connect()` 会在 `roomId` 的每个不同值上运行。你的 Effect 会让聊天连接与当前所选房间保持同步。

## 将非响应式逻辑从 Effect 中提取出来 {/*extracting-non-reactive-logic-out-of-effects*/}

当你想把响应式逻辑与非响应式逻辑混合在一起时，事情就会变得更复杂。

例如，假设你想在用户连接到聊天时显示一条通知。你从 props 中读取当前主题（深色或浅色），以便用正确的颜色显示通知：

```js {1,4-6}
function ChatRoom({ roomId, theme }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.on('connected', () => {
      showNotification('Connected!', theme);
    });
    connection.connect();
    // ...
```

然而，`theme` 是一个响应式值（它可能会因为重新渲染而变化），并且[Effect 读取到的每一个响应式值都必须被声明为依赖项。](/learn/lifecycle-of-reactive-effects#react-verifies-that-you-specified-every-reactive-value-as-a-dependency) 现在你必须把 `theme` 指定为 Effect 的依赖项：

```js {5,11}
function ChatRoom({ roomId, theme }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.on('connected', () => {
      showNotification('Connected!', theme);
    });
    connection.connect();
    return () => {
      connection.disconnect()
    };
  }, [roomId, theme]); // ✅ 已声明所有依赖项
  // ...
```

试试这个示例，看看你能否找出这个用户体验中的问题：

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
      showNotification('Connected!', theme);
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

当 `roomId` 变化时，聊天会按预期重新连接。但由于 `theme` 也是一个依赖项，所以每次你在深色和浅色主题之间切换时，聊天也会重新连接。这不太好！

换句话说，尽管这行代码位于一个 Effect 中（而 Effect 是响应式的），你*并不*希望它是响应式的：

```js
      // ...
      showNotification('Connected!', theme);
      // ...
```

你需要一种方法，把这段非响应式逻辑从其外层的响应式 Effect 中分离出来。

### 声明一个 Effect Event {/*declaring-an-effect-event*/}

使用一个名为 [`useEffectEvent`](/reference/react/useEffectEvent) 的特殊 Hook 来把这段非响应式逻辑从你的 Effect 中提取出来：

```js {1,4-6}
import { useEffect, useEffectEvent } from 'react';

function ChatRoom({ roomId, theme }) {
  const onConnected = useEffectEvent(() => {
    showNotification('Connected!', theme);
  });
  // ...
```

这里，`onConnected` 被称为一个 *Effect Event*。它是你的 Effect 逻辑的一部分，但行为更像一个事件处理函数。它内部的逻辑不是响应式的，而且总是能“看到”props 和 state 的最新值。

现在你可以在 Effect 内部调用 `onConnected` 这个 Effect Event：

```js {2-4,9,13}
function ChatRoom({ roomId, theme }) {
  const onConnected = useEffectEvent(() => {
    showNotification('Connected!', theme);
  });

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.on('connected', () => {
      onConnected();
    });
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]); // ✅ 已声明所有依赖项
  // ...
```

这解决了问题。注意，你必须把 `theme` 从 Effect 的依赖项列表中*移除*，因为它不再在 Effect 中使用了。你也不需要把 `onConnected` *添加*进去，因为**Effect Event 不是响应式的，必须从依赖项中省略。**

验证一下新的行为是否符合预期：

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
    showNotification('Connected!', theme);
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

你可以把 Effect Event 看作和事件处理函数非常相似。主要区别在于，事件处理函数是响应用户交互而运行，而 Effect Events 是由你从 Effect 中触发的。Effect Events 让你可以“打断”Effect 的响应式链条与那些不应该响应式的代码之间的联系。

### 使用 Effect Event 读取最新的 props 和 state {/*reading-latest-props-and-state-with-effect-events*/}

Effect Event 可以帮助你修复很多你可能想通过抑制依赖项检查器来处理的模式。

例如，假设你有一个用于记录页面访问的 Effect：

```js
function Page() {
  useEffect(() => {
    logVisit();
  }, []);
  // ...
}
```

后来，你的网站增加了多个路由。现在你的 `Page` 组件会接收一个表示当前路径的 `url` prop。你希望在 `logVisit` 调用中传入 `url`，但依赖项检查器会报错：

```js {1,3}
function Page({ url }) {
  useEffect(() => {
    logVisit(url);
  }, []); // 🔴 React Hook useEffect 缺少依赖项：'url'
  // ...
}
```

想一想你希望这段代码做什么。你*希望*针对不同的 URL 记录分别的访问，因为每个 URL 都代表一个不同的页面。换句话说，这次 `logVisit` 调用*应该*对 `url` 保持响应式。这就是为什么在这种情况下，遵循依赖项检查器并把 `url` 加入依赖项是合理的：

```js {4}
function Page({ url }) {
  useEffect(() => {
    logVisit(url);
  }, [url]); // ✅ 已声明所有依赖项
  // ...
}
```

现在假设你想在每次页面访问时都把购物车中的商品数量一起记录：

```js {2-3,6}
function Page({ url }) {
  const { items } = useContext(ShoppingCartContext);
  const numberOfItems = items.length;

  useEffect(() => {
    logVisit(url, numberOfItems);
  }, [url]); // 🔴 React Hook useEffect 缺少依赖项：'numberOfItems'
  // ...
}
```

你在 Effect 内部使用了 `numberOfItems`，所以检查器要求你把它添加为依赖项。然而，你*并不*希望 `logVisit` 调用对 `numberOfItems` 保持响应式。如果用户把商品放进购物车，导致 `numberOfItems` 变化，这*并不意味着*用户再次访问了页面。换句话说，*访问页面*在某种意义上是一个“事件”。它发生在时间上的某个精确时刻。

把代码拆成两部分：

```js {5-7,10}
function Page({ url }) {
  const { items } = useContext(ShoppingCartContext);
  const numberOfItems = items.length;

  const onVisit = useEffectEvent(visitedUrl => {
    logVisit(visitedUrl, numberOfItems);
  });

  useEffect(() => {
    onVisit(url);
  }, [url]); // ✅ 已声明所有依赖项
  // ...
}
```

这里的 `onVisit` 是一个 Effect Event。它内部的代码不是响应式的。这就是为什么你可以使用 `numberOfItems`（或任何其他响应式值！），而不用担心它会导致外层代码在变化时重新执行。

另一方面，Effect 本身仍然是响应式的。Effect 内部的代码使用了 `url` prop，因此 Effect 会在每次重新渲染且 `url` 不同时重新运行。而这又会调用 `onVisit` 这个 Effect Event。

结果就是，你会在 `url` 每次变化时都调用 `logVisit`，并且始终读取到最新的 `numberOfItems`。不过，如果 `numberOfItems` 自己变化了，这不会导致任何代码重新运行。

<Note>

你可能会想，是否可以直接不带参数地调用 `onVisit()`，然后在其中读取 `url`：

```js {2,6}
  const onVisit = useEffectEvent(() => {
    logVisit(url, numberOfItems);
  });

  useEffect(() => {
    onVisit();
  }, [url]);
```

这也能工作，但最好还是把这个 `url` 显式作为参数传给 Effect Event。**通过把 `url` 作为参数传给你的 Effect Event，你是在说，从用户的角度看，访问一个不同 `url` 的页面构成了一个单独的“事件”。** `visitedUrl` 是发生的“事件”的一*部分*：

```js {1-2,6}
  const onVisit = useEffectEvent(visitedUrl => {
    logVisit(visitedUrl, numberOfItems);
  });

  useEffect(() => {
    onVisit(url);
  }, [url]);
```

由于你的 Effect Event 明确“要求”传入 `visitedUrl`，现在你就不可能意外地从 Effect 的依赖项中移除 `url`。如果你移除了 `url` 依赖项（导致不同页面访问被算作一次），检查器会提醒你。你希望 `onVisit` 对 `url` 保持响应式，所以不要在内部读取 `url`（那样它就不会响应式），而是*从*你的 Effect 中把它传进去。

如果 Effect 中存在某些异步逻辑，这一点尤其重要：

```js {6,8}
  const onVisit = useEffectEvent(visitedUrl => {
    logVisit(visitedUrl, numberOfItems);
  });

  useEffect(() => {
    setTimeout(() => {
      onVisit(url);
    }, 5000); // 延迟记录访问
  }, [url]);
```

这里 `onVisit` 内部的 `url` 对应的是*最新的* `url`（它可能已经改变），而 `visitedUrl` 对应的是最初触发这个 Effect（以及这次 `onVisit` 调用）运行的那个 `url`。

</Note>

<DeepDive>

#### 直接抑制依赖项检查器可以吗？ {/*is-it-okay-to-suppress-the-dependency-linter-instead*/}

在现有代码库中，你有时可能会看到像这样抑制 lint 规则：

```js {expectedErrors: {'react-compiler': [8]}} {7-9}
function Page({ url }) {
  const { items } = useContext(ShoppingCartContext);
  const numberOfItems = items.length;

  useEffect(() => {
    logVisit(url, numberOfItems);
    // 🔴 不要像这样抑制检查器：
    // eslint-disable-next-line react-hooks/exhaustive-deps
  }, [url]);
  // ...
}
```

我们建议**永远不要抑制检查器**。

抑制规则的第一个缺点是：当你的 Effect 需要“响应”你后来添加到代码中的新响应式依赖时，React 将不再提醒你。在前面的例子里，你之所以把 `url` 加入依赖项，*正是因为* React 提醒了你这么做。如果你禁用了检查器，以后对这个 Effect 的任何修改都不会再得到这样的提醒。这会导致 bug。

下面是一个因抑制检查器而导致的令人困惑的 bug 示例。在这个例子里，`handleMove` 函数本应读取当前 `canMove` state 变量的值，以决定这个点是否应该跟随光标。然而，`handleMove` 内部的 `canMove` 总是 `true`。

你能看出原因吗？

<Sandpack>

```js {expectedErrors: {'react-compiler': [16]}}
import { useState, useEffect } from 'react';

export default function App() {
  const [position, setPosition] = useState({ x: 0, y: 0 });
  const [canMove, setCanMove] = useState(true);

  function handleMove(e) {
    if (canMove) {
      setPosition({ x: e.clientX, y: e.clientY });
    }
  }

  useEffect(() => {
    window.addEventListener('pointermove', handleMove);
    return () => window.removeEventListener('pointermove', handleMove);
    // eslint-disable-next-line react-hooks/exhaustive-deps
  }, []);

  return (
    <>
      <label>
        <input type="checkbox"
          checked={canMove}
          onChange={e => setCanMove(e.target.checked)}
        />
        允许这个点移动
      </label>
      <hr />
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
    </>
  );
}
```

```css
body {
  height: 200px;
}
```

</Sandpack>


这段代码的问题在于抑制了依赖项检查器。如果你移除这个抑制，React 会告诉你这个 Effect 的代码依赖于 `handleMove` 函数。这是有道理的：`handleMove` 定义在组件函数体内，因此它是一个响应式值。每个响应式值都必须被指定为依赖项，否则它就可能随着时间推移变得陈旧！

原始代码的作者对 React “撒了谎”，声称这个 Effect 不依赖任何响应式值（`[]`）。这就是为什么 `canMove` 改变后，React 没有重新同步这个 Effect（以及其中的 `handleMove`）。因为 React 没有重新同步这个 Effect，作为监听器附加上的 `handleMove` 就是初始渲染时创建的那个 `handleMove` 函数。在初始渲染时，`canMove` 是 `true`，这就是为什么初始渲染中的 `handleMove` 永远看到这个值。

**如果你从不抑制检查器，就永远不会看到陈旧值的问题。**

使用 `useEffectEvent` 时，就没有必要对检查器“撒谎”，代码也会如你所期望的那样工作：

<Sandpack>

```js
import { useState, useEffect } from 'react';
import { useEffectEvent } from 'react';

export default function App() {
  const [position, setPosition] = useState({ x: 0, y: 0 });
  const [canMove, setCanMove] = useState(true);

  const onMove = useEffectEvent(e => {
    if (canMove) {
      setPosition({ x: e.clientX, y: e.clientY });
    }
  });

  useEffect(() => {
    window.addEventListener('pointermove', onMove);
    return () => window.removeEventListener('pointermove', onMove);
  }, []);

  return (
    <>
      <label>
        <input type="checkbox"
          checked={canMove}
          onChange={e => setCanMove(e.target.checked)}
        />
        允许这个点移动
      </label>
      <hr />
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
    </>
  );
}
```

```css
body {
  height: 200px;
}
```

</Sandpack>

这并不意味着 `useEffectEvent` *总是*正确的解决方案。你只应将它应用于那些你不希望具备响应性的代码行。在上面的沙盒中，你不希望 Effect 的代码对 `canMove` 保持响应式。这就是为什么提取一个 Effect Event 是合理的。

请阅读[移除 Effect 依赖项](/learn/removing-effect-dependencies)了解其他可以正确替代抑制检查器的方法。

</DeepDive>

### Effect Event 的限制 {/*limitations-of-effect-events*/}

Effect Event 的使用非常受限：

* **只能在 Effect 内部调用它们。**
* **绝不要把它们传给其他组件或 Hook。**

例如，不要像这样声明并传递一个 Effect Event：

```js {4-6,8}
function Timer() {
  const [count, setCount] = useState(0);

  const onTick = useEffectEvent(() => {
    setCount(count + 1);
  });

  useTimer(onTick, 1000); // 🔴 避免：传递 Effect Event

  return <h1>{count}</h1>
}

function useTimer(callback, delay) {
  useEffect(() => {
    const id = setInterval(() => {
      callback();
    }, delay);
    return () => {
      clearInterval(id);
    };
  }, [delay, callback]); // 需要在依赖项中指定 "callback"
}
```

相反，始终把 Effect Event 直接声明在使用它的 Effect 附近：

```js {10-12,16,21}
function Timer() {
  const [count, setCount] = useState(0);
  useTimer(() => {
    setCount(count + 1);
  }, 1000);
  return <h1>{count}</h1>
}

function useTimer(callback, delay) {
  const onTick = useEffectEvent(() => {
    callback();
  });

  useEffect(() => {
    const id = setInterval(() => {
      onTick(); // ✅ 好：仅在 Effect 内部本地调用
    }, delay);
    return () => {
      clearInterval(id);
    };
  }, [delay]); // 不需要把 "onTick"（一个 Effect Event）指定为依赖项
}
```

Effect Event 是你的 Effect 代码中不具备响应性的“片段”。它们应该放在使用它们的 Effect 附近。

<Recap>

- 事件处理函数会响应特定交互而运行。
- Effect 会在需要同步时运行。
- 事件处理函数中的逻辑不是响应式的。
- Effect 中的逻辑是响应式的。
- 你可以把 Effect 中的非响应式逻辑移动到 Effect Event 中。
- 只能在 Effect 内部调用 Effect Event。
- 不要把 Effect Event 传给其他组件或 Hook。

</Recap>

<Challenges>

#### 修复一个不会更新的变量 {/*fix-a-variable-that-doesnt-update*/}

这个 `Timer` 组件维护了一个 `count` state 变量，它每秒增加一次。它每次增加的值存储在 `increment` state 变量中。你可以通过加号和减号按钮控制 `increment` 变量。

然而，不管你点击多少次加号按钮，计数器每秒仍然只会加一。这个代码哪里有问题？为什么在 Effect 的代码里 `increment` 总是等于 `1`？找出错误并修复它。

<Hint>

要修复这段代码，只需遵循规则即可。

</Hint>

<Sandpack>

```js {expectedErrors: {'react-compiler': [14]}}
import { useState, useEffect } from 'react';

export default function Timer() {
  const [count, setCount] = useState(0);
  const [increment, setIncrement] = useState(1);

  useEffect(() => {
    const id = setInterval(() => {
      setCount(c => c + increment);
    }, 1000);
    return () => {
      clearInterval(id);
    };
    // eslint-disable-next-line react-hooks/exhaustive-deps
  }, []);

  return (
    <>
      <h1>
        计数器：{count}
        <button onClick={() => setCount(0)}>重置</button>
      </h1>
      <hr />
      <p>
        每秒增加：
        <button disabled={increment === 0} onClick={() => {
          setIncrement(i => i - 1);
        }}>–</button>
        <b>{increment}</b>
        <button onClick={() => {
          setIncrement(i => i + 1);
        }}>+</button>
      </p>
    </>
  );
}
```

```css
button { margin: 10px; }
```

</Sandpack>

<Solution>

像往常一样，当你在 Effect 中查找 bug 时，先搜索 lint 抑制。

如果你移除这条抑制注释，React 会告诉你这个 Effect 的代码依赖于 `increment`，但你却对 React “撒了谎”，声称这个 Effect 不依赖任何响应式值（`[]`）。把 `increment` 加到依赖数组中：

<Sandpack>

```js
import { useState, useEffect } from 'react';

export default function Timer() {
  const [count, setCount] = useState(0);
  const [increment, setIncrement] = useState(1);

  useEffect(() => {
    const id = setInterval(() => {
      setCount(c => c + increment);
    }, 1000);
    return () => {
      clearInterval(id);
    };
  }, [increment]);

  return (
    <>
      <h1>
        计数器：{count}
        <button onClick={() => setCount(0)}>重置</button>
      </h1>
      <hr />
      <p>
        每秒增加：
        <button disabled={increment === 0} onClick={() => {
          setIncrement(i => i - 1);
        }}>–</button>
        <b>{increment}</b>
        <button onClick={() => {
          setIncrement(i => i + 1);
        }}>+</button>
      </p>
    </>
  );
}
```

```css
button { margin: 10px; }
```

</Sandpack>

现在，当 `increment` 变化时，React 会重新同步你的 Effect，并重新启动这个间隔计时器。

</Solution>

#### 修复一个冻结的计数器 {/*fix-a-freezing-counter*/}

这个 `Timer` 组件维护了一个 `count` state 变量，它每秒增加一次。它每次增加的值存储在 `increment` state 变量中，你可以用加号和减号按钮来控制它。例如，试着按九次加号按钮，你会注意到 `count` 现在每秒增加十，而不是一。

这个用户界面有一个小问题。你可能会注意到，如果你连续以每秒一次以上的速度按加号或减号按钮，计时器本身似乎会暂停。它只有在你最后一次按下按钮之后再过一秒才会恢复。找出原因，并修复这个问题，让计时器能够*每*秒都不中断地走动。

<Hint>

看起来设置计时器的 Effect 会对 `increment` 值“做出反应”。使用当前 `increment` 值来调用 `setCount` 的那一行真的需要具备响应式吗？

</Hint>

<Sandpack>

```js
import { useState, useEffect } from 'react';
import { useEffectEvent } from 'react';

export default function Timer() {
  const [count, setCount] = useState(0);
  const [increment, setIncrement] = useState(1);

  useEffect(() => {
    const id = setInterval(() => {
      setCount(c => c + increment);
    }, 1000);
    return () => {
      clearInterval(id);
    };
  }, [increment]);

  return (
    <>
      <h1>
        计数器：{count}
        <button onClick={() => setCount(0)}>重置</button>
      </h1>
      <hr />
      <p>
        每秒增加：
        <button disabled={increment === 0} onClick={() => {
          setIncrement(i => i - 1);
        }}>–</button>
        <b>{increment}</b>
        <button onClick={() => {
          setIncrement(i => i + 1);
        }}>+</button>
      </p>
    </>
  );
}
```

```css
button { margin: 10px; }
```

</Sandpack>

<Solution>

问题在于 Effect 内部的代码使用了 `increment` state 变量。由于它是这个 Effect 的依赖项，`increment` 的每次变化都会导致 Effect 重新同步，从而清除间隔计时器。如果你在计时器有机会触发之前不断清除它，它看起来就会像是卡住了一样。

要解决这个问题，从 Effect 中提取一个 `onTick` Effect Event：

<Sandpack>

```js
import { useState, useEffect } from 'react';
import { useEffectEvent } from 'react';

export default function Timer() {
  const [count, setCount] = useState(0);
  const [increment, setIncrement] = useState(1);

  const onTick = useEffectEvent(() => {
    setCount(c => c + increment);
  });

  useEffect(() => {
    const id = setInterval(() => {
      onTick();
    }, 1000);
    return () => {
      clearInterval(id);
    };
  }, []);

  return (
    <>
      <h1>
        计数器：{count}
        <button onClick={() => setCount(0)}>重置</button>
      </h1>
      <hr />
      <p>
        每秒增加：
        <button disabled={increment === 0} onClick={() => {
          setIncrement(i => i - 1);
        }}>–</button>
        <b>{increment}</b>
        <button onClick={() => {
          setIncrement(i => i + 1);
        }}>+</button>
      </p>
    </>
  );
}
```


```css
button { margin: 10px; }
```

</Sandpack>

由于 `onTick` 是一个 Effect Event，它内部的代码不是响应式的。`increment` 的变化不会触发任何 Effect。

</Solution>

#### 修复一个无法调整的延迟 {/*fix-a-non-adjustable-delay*/}

在这个示例中，你可以自定义计时间隔的延迟。它存储在一个 `delay` state 变量中，并通过两个按钮更新。然而，即使你一直按 “plus 100 ms” 按钮直到 `delay` 变为 1000 毫秒（也就是一秒），你会注意到计时器仍然增长得很快（每 100 毫秒一次）。就好像你对 `delay` 的修改被忽略了一样。找出并修复这个 bug。

<Hint>

Effect Event 中的代码不是响应式的。有没有一些情况，你会希望 `setInterval` 调用重新运行？

</Hint>

<Sandpack>

```js
import { useState, useEffect } from 'react';
import { useEffectEvent } from 'react';

export default function Timer() {
  const [count, setCount] = useState(0);
  const [increment, setIncrement] = useState(1);
  const [delay, setDelay] = useState(100);

  const onTick = useEffectEvent(() => {
    setCount(c => c + increment);
  });

  const onMount = useEffectEvent(() => {
    return setInterval(() => {
      onTick();
    }, delay);
  });

  useEffect(() => {
    const id = onMount();
    return () => {
      clearInterval(id);
    }
  }, []);

  return (
    <>
      <h1>
        计数器：{count}
        <button onClick={() => setCount(0)}>重置</button>
      </h1>
      <hr />
      <p>
        每秒增加：
        <button disabled={increment === 0} onClick={() => {
          setIncrement(i => i - 1);
        }}>–</button>
        <b>{increment}</b>
        <button onClick={() => {
          setIncrement(i => i + 1);
        }}>+</button>
      </p>
      <p>
        增加延迟：
        <button disabled={delay === 100} onClick={() => {
          setDelay(d => d - 100);
        }}>–100 毫秒</button>
        <b>{delay} 毫秒</b>
        <button onClick={() => {
          setDelay(d => d + 100);
        }}>+100 毫秒</button>
      </p>
    </>
  );
}
```


```css
button { margin: 10px; }
```

</Sandpack>

<Solution>

上面示例的问题在于，它提取了一个名为 `onMount` 的 Effect Event，却没有考虑代码实际上应该做什么。你只应在有特定原因时才提取 Effect Event：当你希望把代码的某一部分变为非响应式时。然而，`setInterval` 调用*应该*对 `delay` state 变量保持响应式。如果 `delay` 变化了，你就希望从头重新设置这个间隔计时器！要修复这段代码，需要把所有响应式代码放回 Effect 内部：

<Sandpack>

```js
import { useState, useEffect } from 'react';
import { useEffectEvent } from 'react';

export default function Timer() {
  const [count, setCount] = useState(0);
  const [increment, setIncrement] = useState(1);
  const [delay, setDelay] = useState(100);

  const onTick = useEffectEvent(() => {
    setCount(c => c + increment);
  });

  useEffect(() => {
    const id = setInterval(() => {
      onTick();
    }, delay);
    return () => {
      clearInterval(id);
    }
  }, [delay]);

  return (
    <>
      <h1>
        计数器：{count}
        <button onClick={() => setCount(0)}>重置</button>
      </h1>
      <hr />
      <p>
        每秒增加：
        <button disabled={increment === 0} onClick={() => {
          setIncrement(i => i - 1);
        }}>–</button>
        <b>{increment}</b>
        <button onClick={() => {
          setIncrement(i => i + 1);
        }}>+</button>
      </p>
      <p>
        增加延迟：
        <button disabled={delay === 100} onClick={() => {
          setDelay(d => d - 100);
        }}>–100 毫秒</button>
        <b>{delay} 毫秒</b>
        <button onClick={() => {
          setDelay(d => d + 100);
        }}>+100 毫秒</button>
      </p>
    </>
  );
}
```

```css
button { margin: 10px; }
```

</Sandpack>

一般来说，你应该对那些更关注*时机*而不是*目的*的函数，比如 `onMount`，保持警惕。一开始它们可能显得“更有描述性”，但其实会掩盖你的意图。通常来说，Effect Event 应该对应于从*用户*角度看发生的某件事。例如，`onMessage`、`onTick`、`onVisit` 或 `onConnected` 都是不错的 Effect Event 命名。它们内部的代码通常不需要具备响应式。另一方面，`onMount`、`onUpdate`、`onUnmount` 或 `onAfterRender` 太过通用，很容易不小心把本应具备响应式的代码放进去。这就是为什么你应该根据*用户认为发生了什么*来命名 Effect Event，而不是根据某段代码何时执行。

</Solution>

#### 修复一个延迟显示的通知 {/*fix-a-delayed-notification*/}

当你加入一个聊天室时，这个组件会显示一条通知。然而，它不会立即显示通知，而是故意延迟两秒，让用户有时间看看界面。

这几乎可行，但有一个 bug。试着快速从 “general” 切换到 “travel”，再切换到 “music”。如果你操作得足够快，你会看到两条通知（这本来应该如此！），但它们*都会*显示 “Welcome to music”。

修复它，使得当你快速从 “general” 切换到 “travel” 再切换到 “music” 时，你会看到两条通知，第一条是 “Welcome to travel”，第二条是 “Welcome to music”。（作为额外挑战，假设你*已经*让通知显示正确的房间名，请修改代码让只显示后者通知。）

<Hint>

你的 Effect 知道它连接到了哪个房间。有没有什么信息你可能想传给你的 Effect Event？

</Hint>

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
    showNotification('Welcome to ' + roomId, theme);
  });

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.on('connected', () => {
      setTimeout(() => {
        onConnected();
      }, 2000);
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

<Solution>

在你的 Effect Event 内部，`roomId` 是 Effect Event 被调用时的值。

你的 Effect Event 在两秒后才会被调用。如果你迅速从 travel 房间切换到 music 房间，那么等 travel 房间的通知显示出来时，`roomId` 已经是 `"music"` 了。这就是为什么两条通知都显示 “Welcome to music”。

要解决这个问题，不要在 Effect Event 内部读取*最新的* `roomId`，而是把它作为 Effect Event 的参数，例如下面的 `connectedRoomId`。然后通过调用 `onConnected(roomId)` 从你的 Effect 中传入 `roomId`：

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
  const onConnected = useEffectEvent(connectedRoomId => {
    showNotification('Welcome to ' + connectedRoomId, theme);
  });

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.on('connected', () => {
      setTimeout(() => {
        onConnected(roomId);
      }, 2000);
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

`roomId` 被设为 `"travel"` 的那个 Effect（因此它连接到了 `"travel"` 房间）会显示 `"travel"` 的通知。`roomId` 被设为 `"music"` 的那个 Effect（因此它连接到了 `"music"` 房间）会显示 `"music"` 的通知。换句话说，`connectedRoomId` 来自你的 Effect（它是响应式的），而 `theme` 则始终使用最新值。

要完成额外挑战，请保存通知的 timeout ID，并在 Effect 的清理函数中将其清除：

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
  const onConnected = useEffectEvent(connectedRoomId => {
    showNotification('欢迎来到 ' + connectedRoomId, theme);
  });

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    let notificationTimeoutId;
    connection.on('connected', () => {
      notificationTimeoutId = setTimeout(() => {
        onConnected(roomId);
      }, 2000);
    });
    connection.connect();
    return () => {
      connection.disconnect();
      if (notificationTimeoutId !== undefined) {
        clearTimeout(notificationTimeoutId);
      }
    };
  }, [roomId]);

  return <h1>欢迎来到 {roomId} 房间！</h1>
}

export default function App() {
  const [roomId, setRoomId] = useState('general');
  const [isDark, setIsDark] = useState(false);
  return (
    <>
      <label>
        选择聊天房间：{' '}
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
  // 实际实现会真正连接到服务器
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

这确保了当你切换房间时，已经排队（但尚未显示）的通知会被取消。

</Solution>

</Challenges>
