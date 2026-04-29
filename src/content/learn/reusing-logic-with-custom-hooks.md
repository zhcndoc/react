---
title: '通过自定义 Hook 重用逻辑'
---

<Intro>

React 自带了一些内置 Hook，比如 `useState`、`useContext` 和 `useEffect`。有时候，你会希望针对某个更具体的用途也有一个 Hook：例如，用来获取数据、跟踪用户是否在线，或者连接到聊天房间。你也许不会在 React 中找到这些 Hook，但你可以为应用的需要创建自己的 Hook。

</Intro>

<YouWillLearn>

- 什么是自定义 Hook，以及如何编写自己的 Hook
- 如何在组件之间重用逻辑
- 如何命名和组织你的自定义 Hook
- 何时以及为何提取自定义 Hook

</YouWillLearn>

## 自定义 Hook：在组件之间共享逻辑 {/*custom-hooks-sharing-logic-between-components*/}

想象一下，你正在开发一个高度依赖网络的应用（就像大多数应用一样）。你希望在用户使用应用时，如果网络连接意外断开，就提醒用户。你会怎么做？看起来你在组件里需要两样东西：

1. 一个用于跟踪网络是否在线的状态。
2. 一个订阅全局 [`online`](https://developer.mozilla.org/en-US/docs/Web/API/Window/online_event) 和 [`offline`](https://developer.mozilla.org/en-US/docs/Web/API/Window/offline_event) 事件，并更新该状态的 Effect。

这会让你的组件与网络状态保持[同步](/learn/synchronizing-with-effects)。你可能会从下面这样的代码开始：

<Sandpack>

```js
import { useState, useEffect } from 'react';

export default function StatusBar() {
  const [isOnline, setIsOnline] = useState(true);
  useEffect(() => {
    function handleOnline() {
      setIsOnline(true);
    }
    function handleOffline() {
      setIsOnline(false);
    }
    window.addEventListener('online', handleOnline);
    window.addEventListener('offline', handleOffline);
    return () => {
      window.removeEventListener('online', handleOnline);
      window.removeEventListener('offline', handleOffline);
    };
  }, []);

  return <h1>{isOnline ? '✅ 在线' : '❌ 已断开连接'}</h1>;
}
```

</Sandpack>

试着把网络打开和关闭，看看这个 `StatusBar` 是如何根据你的操作更新的。

现在假设你*也*想在另一个组件中使用同样的逻辑。你想实现一个保存按钮：当网络断开时，它会被禁用，并显示“正在重新连接...”，而不是“保存”。

首先，你可以把 `isOnline` 状态和 Effect 复制到 `SaveButton` 中：

<Sandpack>

```js
import { useState, useEffect } from 'react';

export default function SaveButton() {
  const [isOnline, setIsOnline] = useState(true);
  useEffect(() => {
    function handleOnline() {
      setIsOnline(true);
    }
    function handleOffline() {
      setIsOnline(false);
    }
    window.addEventListener('online', handleOnline);
    window.addEventListener('offline', handleOffline);
    return () => {
      window.removeEventListener('online', handleOnline);
      window.removeEventListener('offline', handleOffline);
    };
  }, []);

  function handleSaveClick() {
    console.log('✅ 进度已保存');
  }

  return (
    <button disabled={!isOnline} onClick={handleSaveClick}>
      {isOnline ? '保存进度' : '正在重新连接...'}
    </button>
  );
}
```

</Sandpack>

确认一下，如果你关闭网络，按钮会改变外观。

这两个组件都能正常工作，但它们之间逻辑的重复并不理想。看起来虽然它们有不同的*视觉外观*，但你希望在它们之间重用逻辑。

### 从组件中提取你自己的自定义 Hook {/*extracting-your-own-custom-hook-from-a-component*/}

想象一下，类似于 [`useState`](/reference/react/useState) 和 [`useEffect`](/reference/react/useEffect)，有一个内置的 `useOnlineStatus` Hook。那么这两个组件就可以简化，并且你可以去掉它们之间的重复代码：

```js {2,7}
function StatusBar() {
  const isOnline = useOnlineStatus();
  return <h1>{isOnline ? '✅ 在线' : '❌ 已断开连接'}</h1>;
}

function SaveButton() {
  const isOnline = useOnlineStatus();

  function handleSaveClick() {
    console.log('✅ 进度已保存');
  }

  return (
    <button disabled={!isOnline} onClick={handleSaveClick}>
      {isOnline ? '保存进度' : '正在重新连接...'}
    </button>
  );
}
```

虽然并没有这样的内置 Hook，但你可以自己编写它。声明一个名为 `useOnlineStatus` 的函数，并把你之前编写的组件中的所有重复代码移动进去：

```js {2-16}
function useOnlineStatus() {
  const [isOnline, setIsOnline] = useState(true);
  useEffect(() => {
    function handleOnline() {
      setIsOnline(true);
    }
    function handleOffline() {
      setIsOnline(false);
    }
    window.addEventListener('online', handleOnline);
    window.addEventListener('offline', handleOffline);
    return () => {
      window.removeEventListener('online', handleOnline);
      window.removeEventListener('offline', handleOffline);
    };
  }, []);
  return isOnline;
}
```

在函数末尾，返回 `isOnline`。这样你的组件就可以读取这个值：

<Sandpack>

```js
import { useOnlineStatus } from './useOnlineStatus.js';

function StatusBar() {
  const isOnline = useOnlineStatus();
  return <h1>{isOnline ? '✅ 在线' : '❌ 已断开连接'}</h1>;
}

function SaveButton() {
  const isOnline = useOnlineStatus();

  function handleSaveClick() {
    console.log('✅ 进度已保存');
  }

  return (
    <button disabled={!isOnline} onClick={handleSaveClick}>
      {isOnline ? '保存进度' : '正在重新连接...'}
    </button>
  );
}

export default function App() {
  return (
    <>
      <SaveButton />
      <StatusBar />
    </>
  );
}
```

```js src/useOnlineStatus.js
import { useState, useEffect } from 'react';

export function useOnlineStatus() {
  const [isOnline, setIsOnline] = useState(true);
  useEffect(() => {
    function handleOnline() {
      setIsOnline(true);
    }
    function handleOffline() {
      setIsOnline(false);
    }
    window.addEventListener('online', handleOnline);
    window.addEventListener('offline', handleOffline);
    return () => {
      window.removeEventListener('online', handleOnline);
      window.removeEventListener('offline', handleOffline);
    };
  }, []);
  return isOnline;
}
```

</Sandpack>

确认切换网络的开关会同时更新这两个组件。

现在你的组件就没有那么多重复逻辑了。**更重要的是，它们内部的代码描述的是*它们想做什么*（使用在线状态！），而不是*如何去做*（通过订阅浏览器事件）。**

当你把逻辑提取到自定义 Hook 中时，你可以隐藏你与某些外部系统或浏览器 API 打交道时那些复杂的细节。组件的代码表达的是你的意图，而不是实现方式。

### Hook 名称总是以 `use` 开头 {/*hook-names-always-start-with-use*/}

React 应用由组件构成。组件由 Hook 构成，无论是内置的还是自定义的。你很可能经常会使用别人创建的自定义 Hook，但偶尔你也可能会自己编写一个！

你必须遵循以下命名规范：

1. **React 组件名必须以大写字母开头，**比如 `StatusBar` 和 `SaveButton`。React 组件还需要返回 React 知道如何展示的内容，比如一段 JSX。
2. **Hook 名称必须以 `use` 开头，后面跟一个大写字母，**比如 [`useState`](/reference/react/useState)（内置的）或 `useOnlineStatus`（像本页前面那样的自定义 Hook）。Hook 可以返回任意值。

这个约定保证了你总能查看某个组件，并知道它的状态、Effects 以及其他 React 特性可能“藏”在哪里。例如，如果你在组件内部看到一个 `getColor()` 函数调用，你可以确信它不可能在内部包含 React 状态，因为它的名字并不是以 `use` 开头的。然而，像 `useOnlineStatus()` 这样的函数调用，很可能内部还会调用其他 Hook！

<Note>

如果你的 linter 已为 [React 配置，](/learn/editor-setup#linting)它会强制执行这个命名约定。向上滚动到上面的 sandbox，把 `useOnlineStatus` 重命名为 `getOnlineStatus`。注意，linter 将不再允许你在其中调用 `useState` 或 `useEffect`。只有 Hook 和组件才能调用其他 Hook！

</Note>

<DeepDive>

#### 在渲染期间调用的所有函数都应该以 use 前缀开头吗？ {/*should-all-functions-called-during-rendering-start-with-the-use-prefix*/}

不是。那些不*调用* Hook 的函数不需要*成为* Hook。

如果你的函数不调用任何 Hook，就不要使用 `use` 前缀。相反，把它写成一个不带 `use` 前缀的普通函数。例如，下面的 `useSorted` 不会调用 Hook，所以应该把它叫做 `getSorted`：

```js
// 🔴 避免：一个不使用 Hook 的 Hook
function useSorted(items) {
  return items.slice().sort();
}

// ✅ 好的：一个不使用 Hook 的普通函数
function getSorted(items) {
  return items.slice().sort();
}
```

这样可以确保你的代码可以在任何地方调用这个普通函数，包括条件语句中：

```js
function List({ items, shouldSort }) {
  let displayedItems = items;
  if (shouldSort) {
    // ✅ 在条件中调用 getSorted() 没问题，因为它不是 Hook
    displayedItems = getSorted(items);
  }
  // ...
}
```

如果一个函数内部至少使用了一个 Hook，你就应该给它加上 `use` 前缀（从而把它变成一个 Hook）：

```js
// ✅ 好的：一个使用了其他 Hook 的 Hook
function useAuth() {
  return useContext(Auth);
}
```

从技术上讲，React 并不会强制这一点。原则上，你可以创建一个不调用其他 Hook 的 Hook。这通常会让人困惑并造成限制，所以最好避免这种模式。不过，某些罕见情况下它可能会有帮助。例如，也许你的函数现在还没有使用任何 Hook，但你计划将来为它添加一些 Hook 调用。那么，用 `use` 前缀来命名它就很合理：

```js {3-4}
// ✅ 好的：一个将来很可能会使用其他 Hook 的 Hook
function useAuth() {
  // TODO：在实现身份验证时用这一行替换：
  // return useContext(Auth);
  return TEST_USER;
}
```

这样，组件就不能条件式地调用它了。当你以后真的在其中加入 Hook 调用时，这一点会变得很重要。如果你不打算在其中使用 Hook（现在或以后都不打算），那就不要把它做成 Hook。

</DeepDive>

### 自定义 Hook 让你共享有状态逻辑，而不是状态本身 {/*custom-hooks-let-you-share-stateful-logic-not-state-itself*/}

在前面的例子中，当你打开和关闭网络时，这两个组件会一起更新。不过，认为它们共享同一个 `isOnline` 状态变量是错误的。看这段代码：

```js {2,7}
function StatusBar() {
  const isOnline = useOnlineStatus();
  // ...
}

function SaveButton() {
  const isOnline = useOnlineStatus();
  // ...
}
```

它的工作方式与提取重复代码之前一样：

```js {2-5,10-13}
function StatusBar() {
  const [isOnline, setIsOnline] = useState(true);
  useEffect(() => {
    // ...
  }, []);
  // ...
}

function SaveButton() {
  const [isOnline, setIsOnline] = useState(true);
  useEffect(() => {
    // ...
  }, []);
  // ...
}
```

这其实是两个完全独立的状态变量和 Effects！它们之所以会在同一时间拥有相同的值，只是因为你把它们与同一个外部值（网络是否开启）同步了。

为了更好地说明这一点，我们需要另一个例子。来看这个 `Form` 组件：

<Sandpack>

```js
import { useState } from 'react';

export default function Form() {
  const [firstName, setFirstName] = useState('Mary');
  const [lastName, setLastName] = useState('Poppins');

  function handleFirstNameChange(e) {
    setFirstName(e.target.value);
  }

  function handleLastNameChange(e) {
    setLastName(e.target.value);
  }

  return (
    <>
      <label>
        名字：
        <input value={firstName} onChange={handleFirstNameChange} />
      </label>
      <label>
        姓氏：
        <input value={lastName} onChange={handleLastNameChange} />
      </label>
      <p><b>早上好，{firstName} {lastName}。</b></p>
    </>
  );
}
```

```css
label { display: block; }
input { margin-left: 10px; }
```

</Sandpack>

每个表单字段都有一些重复逻辑：

1. 有一个状态（`firstName` 和 `lastName`）。
1. 有一个变更处理函数（`handleFirstNameChange` 和 `handleLastNameChange`）。
1. 有一段 JSX，用来为该输入框指定 `value` 和 `onChange` 属性。

你可以把这些重复逻辑提取到这个 `useFormInput` 自定义 Hook 中：

<Sandpack>

```js
import { useFormInput } from './useFormInput.js';

export default function Form() {
  const firstNameProps = useFormInput('Mary');
  const lastNameProps = useFormInput('Poppins');

  return (
    <>
      <label>
        名字：
        <input {...firstNameProps} />
      </label>
      <label>
        姓氏：
        <input {...lastNameProps} />
      </label>
      <p><b>早上好，{firstNameProps.value} {lastNameProps.value}。</b></p>
    </>
  );
}
```

```js src/useFormInput.js active
import { useState } from 'react';

export function useFormInput(initialValue) {
  const [value, setValue] = useState(initialValue);

  function handleChange(e) {
    setValue(e.target.value);
  }

  const inputProps = {
    value: value,
    onChange: handleChange
  };

  return inputProps;
}
```

```css
label { display: block; }
input { margin-left: 10px; }
```

</Sandpack>

注意，它只声明了一个名为 `value` 的状态变量。

然而，`Form` 组件*两次*调用了 `useFormInput`：

```js
function Form() {
  const firstNameProps = useFormInput('Mary');
  const lastNameProps = useFormInput('Poppins');
  // ...
```

这就是为什么它的工作方式就像声明了两个独立的状态变量一样！

**自定义 Hook 让你共享*有状态逻辑*，但不能共享*状态本身*。每次调用 Hook 都与对同一个 Hook 的其他调用完全独立。** 这就是为什么上面两个 sandbox 是完全等价的。如果你愿意，可以向上滚动并比较它们。提取自定义 Hook 前后的行为是完全相同的。

当你需要在多个组件之间共享状态本身时，请改为[提升状态并向下传递](/learn/sharing-state-between-components)。

## 在 Hooks 之间传递响应式值 {/*passing-reactive-values-between-hooks*/}

你自定义 Hook 内部的代码会在组件每次重新渲染时重新运行。这就是为什么像组件一样，自定义 Hook [也需要保持纯净。](/learn/keeping-components-pure) 可以把自定义 Hook 的代码看作是组件主体的一部分！

因为自定义 Hook 会和组件一起重新渲染，所以它们总能接收到最新的 props 和 state。为了理解这意味着什么，来看这个聊天室示例。更改服务器 URL 或聊天室：

<Sandpack>

```js src/App.js
import { useState } from 'react';
import ChatRoom from './ChatRoom.js';

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
      <ChatRoom
        roomId={roomId}
      />
    </>
  );
}
```

```js src/ChatRoom.js active
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';
import { showNotification } from './notifications.js';

export default function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  useEffect(() => {
    const options = {
      serverUrl: serverUrl,
      roomId: roomId
    };
    const connection = createConnection(options);
    connection.on('message', (msg) => {
      showNotification('新消息：' + msg);
    });
    connection.connect();
    return () => connection.disconnect();
  }, [roomId, serverUrl]);

  return (
    <>
      <label>
        服务器 URL：
        <input value={serverUrl} onChange={e => setServerUrl(e.target.value)} />
      </label>
      <h1>欢迎来到 {roomId} 房间！</h1>
    </>
  );
}
```

```js src/chat.js
export function createConnection({ serverUrl, roomId }) {
  // 真实实现会实际连接到服务器
  if (typeof serverUrl !== 'string') {
    throw Error('Expected serverUrl to be a string. Received: ' + serverUrl);
  }
  if (typeof roomId !== 'string') {
    throw Error('Expected roomId to be a string. Received: ' + roomId);
  }
  let intervalId;
  let messageCallback;
  return {
    connect() {
      console.log('✅ 正在连接到位于 ' + serverUrl + ' 的 "' + roomId + '" 房间...');
      clearInterval(intervalId);
      intervalId = setInterval(() => {
        if (messageCallback) {
          if (Math.random() > 0.5) {
            messageCallback('hey')
          } else {
            messageCallback('lol');
          }
        }
      }, 3000);
    },
    disconnect() {
      clearInterval(intervalId);
      messageCallback = null;
      console.log('❌ 已从位于 ' + serverUrl + ' 的 "' + roomId + '" 房间断开连接');
    },
    on(event, callback) {
      if (messageCallback) {
        throw Error('Cannot add the handler twice.');
      }
      if (event !== 'message') {
        throw Error('Only "message" event is supported.');
      }
      messageCallback = callback;
    },
  };
}
```

```js src/notifications.js
import Toastify from 'toastify-js';
import 'toastify-js/src/toastify.css';

export function showNotification(message, theme = 'dark') {
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

```css
input { display: block; margin-bottom: 20px; }
button { margin-left: 10px; }
```

</Sandpack>

当你更改 `serverUrl` 或 `roomId` 时，Effect 会[“响应”这些变化](/learn/lifecycle-of-reactive-effects#effects-react-to-reactive-values)并重新同步。你可以从控制台消息中看出，每次更改 Effect 的依赖项时，聊天室都会重新连接。

现在把 Effect 的代码移到一个自定义 Hook 中：

```js {2-13}
export function useChatRoom({ serverUrl, roomId }) {
  useEffect(() => {
    const options = {
      serverUrl: serverUrl,
      roomId: roomId
    };
    const connection = createConnection(options);
    connection.connect();
    connection.on('message', (msg) => {
      showNotification('新消息：' + msg);
    });
    return () => connection.disconnect();
  }, [roomId, serverUrl]);
}
```

这样你的 `ChatRoom` 组件就可以调用你的自定义 Hook，而不必关心它内部是如何工作的：

```js {4-7}
export default function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  useChatRoom({
    roomId: roomId,
    serverUrl: serverUrl
  });

  return (
    <>
      <label>
        服务器 URL：
        <input value={serverUrl} onChange={e => setServerUrl(e.target.value)} />
      </label>
      <h1>欢迎来到 {roomId} 房间！</h1>
    </>
  );
}
```

这样看起来简单多了！（但做的是同样的事情。）

注意逻辑 *仍然会响应* prop 和 state 的变化。试着编辑服务器 URL 或选中的房间：

<Sandpack>

```js src/App.js
import { useState } from 'react';
import ChatRoom from './ChatRoom.js';

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
      <ChatRoom
        roomId={roomId}
      />
    </>
  );
}
```

```js src/ChatRoom.js active
import { useState } from 'react';
import { useChatRoom } from './useChatRoom.js';

export default function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  useChatRoom({
    roomId: roomId,
    serverUrl: serverUrl
  });

  return (
    <>
      <label>
        服务器 URL：
        <input value={serverUrl} onChange={e => setServerUrl(e.target.value)} />
      </label>
      <h1>欢迎来到 {roomId} 房间！</h1>
    </>
  );
}
```

```js src/useChatRoom.js
import { useEffect } from 'react';
import { createConnection } from './chat.js';
import { showNotification } from './notifications.js';

export function useChatRoom({ serverUrl, roomId }) {
  useEffect(() => {
    const options = {
      serverUrl: serverUrl,
      roomId: roomId
    };
    const connection = createConnection(options);
    connection.connect();
    connection.on('message', (msg) => {
      showNotification('新消息：' + msg);
    });
    return () => connection.disconnect();
  }, [roomId, serverUrl]);
}
```

```js src/chat.js
export function createConnection({ serverUrl, roomId }) {
  // 真实实现会实际连接到服务器
  if (typeof serverUrl !== 'string') {
    throw Error('Expected serverUrl to be a string. Received: ' + serverUrl);
  }
  if (typeof roomId !== 'string') {
    throw Error('Expected roomId to be a string. Received: ' + roomId);
  }
  let intervalId;
  let messageCallback;
  return {
    connect() {
      console.log('✅ 正在连接到位于 ' + serverUrl + ' 的 "' + roomId + '" 房间...');
      clearInterval(intervalId);
      intervalId = setInterval(() => {
        if (messageCallback) {
          if (Math.random() > 0.5) {
            messageCallback('hey')
          } else {
            messageCallback('lol');
          }
        }
      }, 3000);
    },
    disconnect() {
      clearInterval(intervalId);
      messageCallback = null;
      console.log('❌ 已从位于 ' + serverUrl + ' 的 "' + roomId + '" 房间断开连接');
    },
    on(event, callback) {
      if (messageCallback) {
        throw Error('Cannot add the handler twice.');
      }
      if (event !== 'message') {
        throw Error('Only "message" event is supported.');
      }
      messageCallback = callback;
    },
  };
}
```

```js src/notifications.js
import Toastify from 'toastify-js';
import 'toastify-js/src/toastify.css';

export function showNotification(message, theme = 'dark') {
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

```css
input { display: block; margin-bottom: 20px; }
button { margin-left: 10px; }
```

</Sandpack>

注意你是如何取出一个 Hook 的返回值：

```js {2}
export default function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  useChatRoom({
    roomId: roomId,
    serverUrl: serverUrl
  });
  // ...
```

并把它作为输入传给另一个 Hook 的：

```js {6}
export default function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  useChatRoom({
    roomId: roomId,
    serverUrl: serverUrl
  });
  // ...
```

每次 `ChatRoom` 组件重新渲染时，它都会把最新的 `roomId` 和 `serverUrl` 传给你的 Hook。这就是为什么在重新渲染后，只要它们的值发生变化，你的 Effect 就会重新连接聊天室。（如果你曾经使用过音频或视频处理软件，像这样串联 Hooks 可能会让你联想到串联视觉或音频效果。就好像 `useState` 的输出“流入”了 `useChatRoom` 的输入一样。）

### 将事件处理函数传给自定义 Hooks {/*passing-event-handlers-to-custom-hooks*/}

当你开始在更多组件中使用 `useChatRoom` 时，你可能希望让组件自定义它的行为。例如，目前，当消息到达时该怎么做的逻辑是直接硬编码在 Hook 里的：

```js {9-11}
export function useChatRoom({ serverUrl, roomId }) {
  useEffect(() => {
    const options = {
      serverUrl: serverUrl,
      roomId: roomId
    };
    const connection = createConnection(options);
    connection.connect();
    connection.on('message', (msg) => {
      showNotification('新消息：' + msg);
    });
    return () => connection.disconnect();
  }, [roomId, serverUrl]);
}
```

假设你想把这部分逻辑移回组件中：

```js {7-9}
export default function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  useChatRoom({
    roomId: roomId,
    serverUrl: serverUrl,
    onReceiveMessage(msg) {
      showNotification('新消息：' + msg);
    }
  });
  // ...
```

要实现这一点，请修改你的自定义 Hook，让它把 `onReceiveMessage` 作为一个命名选项接收：

```js {1,10,13}
export function useChatRoom({ serverUrl, roomId, onReceiveMessage }) {
  useEffect(() => {
    const options = {
      serverUrl: serverUrl,
      roomId: roomId
    };
    const connection = createConnection(options);
    connection.connect();
    connection.on('message', (msg) => {
      onReceiveMessage(msg);
    });
    return () => connection.disconnect();
  }, [roomId, serverUrl, onReceiveMessage]); // ✅ 已声明所有依赖项
}
```

这样可以工作，但如果你的自定义 Hook 接受事件处理函数，还有一个可以进一步改进的地方。

为 `onReceiveMessage` 添加依赖并不理想，因为这会导致组件每次重新渲染时聊天室都重新连接。[把这个事件处理函数包装进 Effect Event，以将它从依赖项中移除：](/learn/removing-effect-dependencies#wrapping-an-event-handler-from-the-props)

```js {1,4,5,15,18}
import { useEffect, useEffectEvent } from 'react';
// ...

export function useChatRoom({ serverUrl, roomId, onReceiveMessage }) {
  const onMessage = useEffectEvent(onReceiveMessage);

  useEffect(() => {
    const options = {
      serverUrl: serverUrl,
      roomId: roomId
    };
    const connection = createConnection(options);
    connection.connect();
    connection.on('message', (msg) => {
      onMessage(msg);
    });
    return () => connection.disconnect();
  }, [roomId, serverUrl]); // ✅ 已声明所有依赖项
}
```

现在，聊天室不会在 `ChatRoom` 组件每次重新渲染时都重新连接了。下面是一个完整可运行的示例，演示如何把事件处理函数传给自定义 Hook，你可以自己试试：

<Sandpack>

```js src/App.js
import { useState } from 'react';
import ChatRoom from './ChatRoom.js';

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
      <ChatRoom
        roomId={roomId}
      />
    </>
  );
}
```

```js src/ChatRoom.js active
import { useState } from 'react';
import { useChatRoom } from './useChatRoom.js';
import { showNotification } from './notifications.js';

export default function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  useChatRoom({
    roomId: roomId,
    serverUrl: serverUrl,
    onReceiveMessage(msg) {
      showNotification('新消息：' + msg);
    }
  });

  return (
    <>
      <label>
        服务器 URL：
        <input value={serverUrl} onChange={e => setServerUrl(e.target.value)} />
      </label>
      <h1>欢迎来到 {roomId} 房间！</h1>
    </>
  );
}
```

```js src/useChatRoom.js
import { useEffect } from 'react';
import { useEffectEvent } from 'react';
import { createConnection } from './chat.js';

export function useChatRoom({ serverUrl, roomId, onReceiveMessage }) {
  const onMessage = useEffectEvent(onReceiveMessage);

  useEffect(() => {
    const options = {
      serverUrl: serverUrl,
      roomId: roomId
    };
    const connection = createConnection(options);
    connection.connect();
    connection.on('message', (msg) => {
      onMessage(msg);
    });
    return () => connection.disconnect();
  }, [roomId, serverUrl]);
}
```

```js src/chat.js
export function createConnection({ serverUrl, roomId }) {
  // 真实实现会实际连接到服务器
  if (typeof serverUrl !== 'string') {
    throw Error('Expected serverUrl to be a string. Received: ' + serverUrl);
  }
  if (typeof roomId !== 'string') {
    throw Error('Expected roomId to be a string. Received: ' + roomId);
  }
  let intervalId;
  let messageCallback;
  return {
    connect() {
      console.log('✅ 正在连接到位于 ' + serverUrl + ' 的 "' + roomId + '" 房间...');
      clearInterval(intervalId);
      intervalId = setInterval(() => {
        if (messageCallback) {
          if (Math.random() > 0.5) {
            messageCallback('hey')
          } else {
            messageCallback('lol');
          }
        }
      }, 3000);
    },
    disconnect() {
      clearInterval(intervalId);
      messageCallback = null;
      console.log('❌ 已从位于 ' + serverUrl + ' 的 "' + roomId + '" 房间断开连接');
    },
    on(event, callback) {
      if (messageCallback) {
        throw Error('Cannot add the handler twice.');
      }
      if (event !== 'message') {
        throw Error('Only "message" event is supported.');
      }
      messageCallback = callback;
    },
  };
}
```

```js src/notifications.js
import Toastify from 'toastify-js';
import 'toastify-js/src/toastify.css';

export function showNotification(message, theme = 'dark') {
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

```css
input { display: block; margin-bottom: 20px; }
button { margin-left: 10px; }
```

</Sandpack>

注意，当你使用它时，不再需要知道 `useChatRoom` 是如何工作的。你可以把它加到任何其他组件中，传入任何其他选项，它的工作方式都一样。这就是自定义 Hook 的力量。

## 何时使用自定义 Hook {/*when-to-use-custom-hooks*/}

你不需要为每一小段重复代码都提取一个自定义 Hook。有一些重复是完全可以接受的。例如，像前面那样抽取一个 `useFormInput` Hook 来包装单独一次 `useState` 调用，可能并没有必要。

不过，每当你编写一个 Effect 时，都可以考虑是否把它也封装进一个自定义 Hook 里会更清晰。[你应该不会经常需要 Effects，](/learn/you-might-not-need-an-effect)所以如果你正在写一个 Effect，这意味着你需要“跳出 React”去和某个外部系统同步，或者去做某些 React 没有内置 API 的事情。把它封装成自定义 Hook 可以让你精确表达你的意图，以及数据如何在其中流动。

例如，考虑一个 `ShippingForm` 组件，它显示两个下拉框：一个显示城市列表，另一个显示所选城市中的区域列表。你可能会先写出如下代码：

```js {3-16,20-35}
function ShippingForm({ country }) {
  const [cities, setCities] = useState(null);
  // 这个 Effect 会为某个国家获取城市列表
  useEffect(() => {
    let ignore = false;
    fetch(`/api/cities?country=${country}`)
      .then(response => response.json())
      .then(json => {
        if (!ignore) {
          setCities(json);
        }
      });
    return () => {
      ignore = true;
    };
  }, [country]);

  const [city, setCity] = useState(null);
  const [areas, setAreas] = useState(null);
  // 这个 Effect 会为所选城市获取区域列表
  useEffect(() => {
    if (city) {
      let ignore = false;
      fetch(`/api/areas?city=${city}`)
        .then(response => response.json())
        .then(json => {
          if (!ignore) {
            setAreas(json);
          }
        });
      return () => {
        ignore = true;
      };
    }
  }, [city]);

  // ...
```

尽管这段代码相当重复，[但把这些 Effects 分开是正确的。](/learn/removing-effect-dependencies#is-your-effect-doing-several-unrelated-things)它们同步的是两件不同的事情，所以你不应该把它们合并成一个 Effect。相反，你可以通过把它们之间的通用逻辑提取到你自己的 `useData` Hook 中，来简化上面的 `ShippingForm` 组件：

```js {2-18}
function useData(url) {
  const [data, setData] = useState(null);
  useEffect(() => {
    if (url) {
      let ignore = false;
      fetch(url)
        .then(response => response.json())
        .then(json => {
          if (!ignore) {
            setData(json);
          }
        });
      return () => {
        ignore = true;
      };
    }
  }, [url]);
  return data;
}
```

现在你可以用对 `useData` 的调用来替换 `ShippingForm` 组件中的两个 Effects：

```js {2,4}
function ShippingForm({ country }) {
  const cities = useData(`/api/cities?country=${country}`);
  const [city, setCity] = useState(null);
  const areas = useData(city ? `/api/areas?city=${city}` : null);
  // ...
```

提取自定义 Hook 会让数据流变得更明确。你传入 `url`，然后得到 `data`。通过把 Effect “隐藏”在 `useData` 里面，你还可以防止在 `ShippingForm` 组件中工作的其他人给它添加[不必要的依赖项](/learn/removing-effect-dependencies)。随着时间推移，你应用中的大多数 Effects 都会位于自定义 Hooks 中。

<DeepDive>

#### 让你的自定义 Hook 聚焦于具体的高层用例 {/*keep-your-custom-hooks-focused-on-concrete-high-level-use-cases*/}

先从给你的自定义 Hook 命名开始。如果你很难想出一个清晰的名字，这可能意味着你的 Effect 与组件其余逻辑耦合得太紧，还不适合被提取出来。

理想情况下，你的自定义 Hook 名称应该足够清晰，以至于即使是很少写代码的人，也能大致猜到你的自定义 Hook 做什么、接收什么、返回什么：

* ✅ `useData(url)`
* ✅ `useImpressionLog(eventName, extraData)`
* ✅ `useChatRoom(options)`

当你与外部系统同步时，自定义 Hook 的名字可能会更偏技术性，并使用该系统特有的术语。只要对熟悉该系统的人来说是清楚的就很好：

* ✅ `useMediaQuery(query)`
* ✅ `useSocket(url)`
* ✅ `useIntersectionObserver(ref, options)`

**让自定义 Hook 聚焦于具体的高层用例。**避免创建和使用那种作为 `useEffect` API 本身的替代品和便捷包装的自定义“生命周期” Hooks：

* 🔴 `useMount(fn)`
* 🔴 `useEffectOnce(fn)`
* 🔴 `useUpdateEffect(fn)`

例如，这个 `useMount` Hook 试图确保某些代码只在“挂载时”运行：

```js {4-5,14-15}
function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  // 🔴 避免：使用自定义“生命周期” Hooks
  useMount(() => {
    const connection = createConnection({ roomId, serverUrl });
    connection.connect();

    post('/analytics/event', { eventName: 'visit_chat' });
  });
  // ...
}

// 🔴 避免：创建自定义“生命周期” Hooks
function useMount(fn) {
  useEffect(() => {
    fn();
  }, []); // 🔴 React Hook useEffect 缺少一个依赖项：'fn'
}
```

**像 `useMount` 这样的自定义“生命周期” Hooks 并不适合 React 的范式。**例如，这段代码有一个错误（它不会对 `roomId` 或 `serverUrl` 的变化作出“响应”），但 lint 工具不会提醒你，因为它只检查直接的 `useEffect` 调用。它不会知道你的 Hook。

如果你正在编写一个 Effect，先直接使用 React API：

```js
function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  // ✅ 好：两个原始 Effects，按用途分开

  useEffect(() => {
    const connection = createConnection({ serverUrl, roomId });
    connection.connect();
    return () => connection.disconnect();
  }, [serverUrl, roomId]);

  useEffect(() => {
    post('/analytics/event', { eventName: 'visit_chat', roomId });
  }, [roomId]);

  // ...
}
```

然后，你可以（但不一定非得）针对不同的高层用例提取自定义 Hooks：

```js
function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  // ✅ 很好：按用途命名的自定义 Hooks
  useChatRoom({ serverUrl, roomId });
  useImpressionLog('visit_chat', { roomId });
  // ...
}
```

**一个好的自定义 Hook 会通过限制其功能，让调用代码更具声明性。**例如，`useChatRoom(options)` 只能连接到聊天室，而 `useImpressionLog(eventName, extraData)` 只能向分析系统发送曝光日志。如果你的自定义 Hook API 没有约束使用场景，而且过于抽象，从长远来看，它很可能带来的问题比解决的问题更多。

</DeepDive>

### 自定义 Hooks 帮助你迁移到更好的模式 {/*custom-hooks-help-you-migrate-to-better-patterns*/}

Effects 是一种["逃生舱口"](/learn/escape-hatches)：当你需要“跳出 React”，并且你的用例没有更好的内置解决方案时，你就会用到它们。随着时间推移，React 团队的目标是通过为更具体的问题提供更具体的解决方案，将你应用中的 Effects 数量尽量减少到最低。把你的 Effects 封装进自定义 Hooks，可以让你在这些解决方案可用时更容易升级代码。

让我们回到这个例子：

<Sandpack>

```js
import { useOnlineStatus } from './useOnlineStatus.js';

function StatusBar() {
  const isOnline = useOnlineStatus();
  return <h1>{isOnline ? '✅ Online' : '❌ Disconnected'}</h1>;
}

function SaveButton() {
  const isOnline = useOnlineStatus();

  function handleSaveClick() {
    console.log('✅ Progress saved');
  }

  return (
    <button disabled={!isOnline} onClick={handleSaveClick}>
      {isOnline ? 'Save progress' : 'Reconnecting...'}
    </button>
  );
}

export default function App() {
  return (
    <>
      <SaveButton />
      <StatusBar />
    </>
  );
}
```

```js src/useOnlineStatus.js active
import { useState, useEffect } from 'react';

export function useOnlineStatus() {
  const [isOnline, setIsOnline] = useState(true);
  useEffect(() => {
    function handleOnline() {
      setIsOnline(true);
    }
    function handleOffline() {
      setIsOnline(false);
    }
    window.addEventListener('online', handleOnline);
    window.addEventListener('offline', handleOffline);
    return () => {
      window.removeEventListener('online', handleOnline);
      window.removeEventListener('offline', handleOffline);
    };
  }, []);
  return isOnline;
}
```

</Sandpack>

在上面的例子中，`useOnlineStatus` 是用一对 [`useState`](/reference/react/useState) 和 [`useEffect`.](/reference/react/useEffect) 实现的。不过，这不是最好的可能方案。它没有考虑到一些边缘情况。例如，它假设组件挂载时 `isOnline` 已经是 `true`，但如果网络已经离线，这可能是错的。你可以使用浏览器的 [`navigator.onLine`](https://developer.mozilla.org/en-US/docs/Web/API/Navigator/onLine) API 来检查这一点，但直接使用它在服务端生成初始 HTML 时不会生效。简而言之，这段代码还可以改进。

React 包含一个专门的 API，叫做 [`useSyncExternalStore`](/reference/react/useSyncExternalStore)，它会帮你处理这些问题。下面是你的 `useOnlineStatus` Hook，重写后利用了这个新的 API：

<Sandpack>

```js
import { useOnlineStatus } from './useOnlineStatus.js';

function StatusBar() {
  const isOnline = useOnlineStatus();
  return <h1>{isOnline ? '✅ Online' : '❌ Disconnected'}</h1>;
}

function SaveButton() {
  const isOnline = useOnlineStatus();

  function handleSaveClick() {
    console.log('✅ Progress saved');
  }

  return (
    <button disabled={!isOnline} onClick={handleSaveClick}>
      {isOnline ? 'Save progress' : 'Reconnecting...'}
    </button>
  );
}

export default function App() {
  return (
    <>
      <SaveButton />
      <StatusBar />
    </>
  );
}
```

```js src/useOnlineStatus.js active
import { useSyncExternalStore } from 'react';

function subscribe(callback) {
  window.addEventListener('online', callback);
  window.addEventListener('offline', callback);
  return () => {
    window.removeEventListener('online', callback);
    window.removeEventListener('offline', callback);
  };
}

export function useOnlineStatus() {
  return useSyncExternalStore(
    subscribe,
    () => navigator.onLine, // 如何在客户端获取值
    () => true // 如何在服务端获取值
  );
}

```

</Sandpack>

注意，为了完成这次迁移，**你不需要修改任何组件**：

```js {2,7}
function StatusBar() {
  const isOnline = useOnlineStatus();
  // ...
}

function SaveButton() {
  const isOnline = useOnlineStatus();
  // ...
}
```

这也是为什么把 Effects 封装进自定义 Hooks 往往是有益的另一个原因：

1. 你让数据进出 Effects 的流向变得非常明确。
2. 你让组件专注于意图，而不是 Effects 的具体实现。
3. 当 React 增加新特性时，你可以在不修改任何组件的情况下移除这些 Effects。

类似于[设计系统，](https://uxdesign.cc/everything-you-need-to-know-about-design-systems-54b109851969)你可能会发现，把应用组件中常见的模式提取到自定义 Hooks 里会很有帮助。这会让组件代码聚焦于意图，并让你避免频繁编写原始 Effects。许多优秀的自定义 Hooks 都由 React 社区维护。

<DeepDive>

#### React 会为数据获取提供任何内置方案吗？ {/*will-react-provide-any-built-in-solution-for-data-fetching*/}

今天，借助 [`use`](/reference/react/use#streaming-data-from-server-to-client) API，可以通过向 `use` 传入一个 [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) 在渲染时读取数据：

```js {1,4,11}
import { use, Suspense } from "react";

function Message({ messagePromise }) {
  const messageContent = use(messagePromise);
  return <p>Here is the message: {messageContent}</p>;
}

export function MessageContainer({ messagePromise }) {
  return (
    <Suspense fallback={<p>⌛Downloading message...</p>}>
      <Message messagePromise={messagePromise} />
    </Suspense>
  );
}
```

我们仍在完善细节，但我们预计未来你会像这样编写数据获取：

```js {1,4,6}
import { use } from 'react';

function ShippingForm({ country }) {
  const cities = use(fetch(`/api/cities?country=${country}`));
  const [city, setCity] = useState(null);
  const areas = city ? use(fetch(`/api/areas?city=${city}`)) : null;
  // ...
```

如果你在应用中使用像上面 `useData` 这样的自定义 Hooks，那么迁移到最终推荐的方法时所需的改动会更少，比起在每个组件里手动编写原始 Effects 要轻松得多。不过，旧方法依然能正常工作，所以如果你更喜欢写原始 Effects，也可以继续这样做。

</DeepDive>

### 实现方式不止一种 {/*there-is-more-than-one-way-to-do-it*/}

假设你想使用浏览器的 [`requestAnimationFrame`](https://developer.mozilla.org/en-US/docs/Web/API/window/requestAnimationFrame) API 从零开始实现一个淡入动画。你可以先写一个 Effect 来设置动画循环。在动画的每一帧中，你可以逐步改变你[保存在 ref 中](/learn/manipulating-the-dom-with-refs)的 DOM 节点的不透明度，直到它达到 `1`。你的代码可能会像这样开始：

<Sandpack>

```js
import { useState, useEffect, useRef } from 'react';

function Welcome() {
  const ref = useRef(null);

  useEffect(() => {
    const duration = 1000;
    const node = ref.current;

    let startTime = performance.now();
    let frameId = null;

    function onFrame(now) {
      const timePassed = now - startTime;
      const progress = Math.min(timePassed / duration, 1);
      onProgress(progress);
      if (progress < 1) {
        // 我们还有更多帧要绘制
        frameId = requestAnimationFrame(onFrame);
      }
    }

    function onProgress(progress) {
      node.style.opacity = progress;
    }

    function start() {
      onProgress(0);
      startTime = performance.now();
      frameId = requestAnimationFrame(onFrame);
    }

    function stop() {
      cancelAnimationFrame(frameId);
      startTime = null;
      frameId = null;
    }

    start();
    return () => stop();
  }, []);

  return (
    <h1 className="welcome" ref={ref}>
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

```css
label, button { display: block; margin-bottom: 20px; }
html, body { min-height: 300px; }
.welcome {
  opacity: 0;
  color: white;
  padding: 50px;
  text-align: center;
  font-size: 50px;
  background-image: radial-gradient(circle, rgba(63,94,251,1) 0%, rgba(252,70,107,1) 100%);
}
```

</Sandpack>

为了让组件更易读，你可以把逻辑提取到一个 `useFadeIn` 自定义 Hook 中：

<Sandpack>

```js
import { useState, useEffect, useRef } from 'react';
import { useFadeIn } from './useFadeIn.js';

function Welcome() {
  const ref = useRef(null);

  useFadeIn(ref, 1000);

  return (
    <h1 className="welcome" ref={ref}>
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

```js src/useFadeIn.js
import { useEffect } from 'react';

export function useFadeIn(ref, duration) {
  useEffect(() => {
    const node = ref.current;

    let startTime = performance.now();
    let frameId = null;

    function onFrame(now) {
      const timePassed = now - startTime;
      const progress = Math.min(timePassed / duration, 1);
      onProgress(progress);
      if (progress < 1) {
        // 我们还有更多帧要绘制
        frameId = requestAnimationFrame(onFrame);
      }
    }

    function onProgress(progress) {
      node.style.opacity = progress;
    }

    function start() {
      onProgress(0);
      startTime = performance.now();
      frameId = requestAnimationFrame(onFrame);
    }

    function stop() {
      cancelAnimationFrame(frameId);
      startTime = null;
      frameId = null;
    }

    start();
    return () => stop();
  }, [ref, duration]);
}
```

```css
label, button { display: block; margin-bottom: 20px; }
html, body { min-height: 300px; }
.welcome {
  opacity: 0;
  color: white;
  padding: 50px;
  text-align: center;
  font-size: 50px;
  background-image: radial-gradient(circle, rgba(63,94,251,1) 0%, rgba(252,70,107,1) 100%);
}
```

</Sandpack>

你可以保持 `useFadeIn` 的代码原样，但也可以进一步重构它。例如，你可以把设置动画循环的逻辑从 `useFadeIn` 中提取到一个自定义 `useAnimationLoop` Hook 里：

<Sandpack>

```js
import { useState, useEffect, useRef } from 'react';
import { useFadeIn } from './useFadeIn.js';

function Welcome() {
  const ref = useRef(null);

  useFadeIn(ref, 1000);

  return (
    <h1 className="welcome" ref={ref}>
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

```js src/useFadeIn.js active
import { useState, useEffect } from 'react';
import { useEffectEvent } from 'react';

export function useFadeIn(ref, duration) {
  const [isRunning, setIsRunning] = useState(true);

  useAnimationLoop(isRunning, (timePassed) => {
    const progress = Math.min(timePassed / duration, 1);
    ref.current.style.opacity = progress;
    if (progress === 1) {
      setIsRunning(false);
    }
  });
}

function useAnimationLoop(isRunning, drawFrame) {
  const onFrame = useEffectEvent(drawFrame);

  useEffect(() => {
    if (!isRunning) {
      return;
    }

    const startTime = performance.now();
    let frameId = null;

    function tick(now) {
      const timePassed = now - startTime;
      onFrame(timePassed);
      frameId = requestAnimationFrame(tick);
    }

    tick();
    return () => cancelAnimationFrame(frameId);
  }, [isRunning]);
}
```

```css
label, button { display: block; margin-bottom: 20px; }
html, body { min-height: 300px; }
.welcome {
  opacity: 0;
  color: white;
  padding: 50px;
  text-align: center;
  font-size: 50px;
  background-image: radial-gradient(circle, rgba(63,94,251,1) 0%, rgba(252,70,107,1) 100%);
}
```

</Sandpack>

不过，你并**不一定必须**这样做。和普通函数一样，最终由你决定如何在代码的不同部分之间划定边界。你也可以采用完全不同的方法。与其把逻辑保留在 Effect 里，不如把大部分命令式逻辑移到一个 JavaScript [class:](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes)

<Sandpack>

```js
import { useState, useEffect, useRef } from 'react';
import { useFadeIn } from './useFadeIn.js';

function Welcome() {
  const ref = useRef(null);

  useFadeIn(ref, 1000);

  return (
    <h1 className="welcome" ref={ref}>
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

```js src/useFadeIn.js active
import { useState, useEffect } from 'react';
import { FadeInAnimation } from './animation.js';

export function useFadeIn(ref, duration) {
  useEffect(() => {
    const animation = new FadeInAnimation(ref.current);
    animation.start(duration);
    return () => {
      animation.stop();
    };
  }, [ref, duration]);
}
```

```js src/animation.js
export class FadeInAnimation {
  constructor(node) {
    this.node = node;
  }
  start(duration) {
    this.duration = duration;
    this.onProgress(0);
    this.startTime = performance.now();
    this.frameId = requestAnimationFrame(() => this.onFrame());
  }
  onFrame() {
    const timePassed = performance.now() - this.startTime;
    const progress = Math.min(timePassed / this.duration, 1);
    this.onProgress(progress);
    if (progress === 1) {
      this.stop();
    } else {
      // 我们还有更多帧要绘制
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
.welcome {
  opacity: 0;
  color: white;
  padding: 50px;
  text-align: center;
  font-size: 50px;
  background-image: radial-gradient(circle, rgba(63,94,251,1) 0%, rgba(252,70,107,1) 100%);
}
```

</Sandpack>

Effects 让你把 React 连接到外部系统。Effects 之间需要协调得越多（例如，串联多个动画），就越有理由像上面的沙盒那样，把这些逻辑**完全**从 Effects 和 Hooks 中提取出来。这样，被你提取出来的代码就**成为了**“外部系统”。这能让你的 Effects 保持简单，因为它们只需要向你搬到 React 外面的系统发送消息。

上面的例子假设淡入逻辑需要用 JavaScript 来编写。不过，这种淡入动画用纯 [CSS Animation:](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Animations/Using_CSS_animations) 来实现既更简单，也更高效：

<Sandpack>

```js
import { useState, useEffect, useRef } from 'react';
import './welcome.css';

function Welcome() {
  return (
    <h1 className="welcome">
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

```css src/styles.css
label, button { display: block; margin-bottom: 20px; }
html, body { min-height: 300px; }
```

```css src/welcome.css active
.welcome {
  color: white;
  padding: 50px;
  text-align: center;
  font-size: 50px;
  background-image: radial-gradient(circle, rgba(63,94,251,1) 0%, rgba(252,70,107,1) 100%);

  animation: fadeIn 1000ms;
}

@keyframes fadeIn {
  0% { opacity: 0; }
  100% { opacity: 1; }
}

```

</Sandpack>

有时候，你甚至不需要 Hook！

<Recap>

- 自定义 Hooks 让你可以在组件之间共享逻辑。
- 自定义 Hooks 的命名必须以 `use` 开头，后面跟一个大写字母。
- 自定义 Hooks 只共享有状态逻辑，不共享 state 本身。
- 你可以把响应式值从一个 Hook 传给另一个，它们会保持最新。
- 组件每次重新渲染时，所有 Hooks 都会重新运行。
- 自定义 Hooks 的代码应该像组件代码一样是纯的。
- 把自定义 Hooks 接收到的事件处理函数封装进 Effect Events。
- 不要创建像 `useMount` 这样的自定义 Hooks。保持用途具体。
- 代码边界如何划分、在哪里划分，由你决定。

</Recap>

<Challenges>

#### 提取一个 `useCounter` Hook {/*extract-a-usecounter-hook*/}

这个组件使用一个 state 变量和一个 Effect 来显示每秒递增的数字。把这段逻辑提取到一个名为 `useCounter` 的自定义 Hook 中。你的目标是让 `Counter` 组件的实现看起来完全像这样：

```js
export default function Counter() {
  const count = useCounter();
  return <h1>Seconds passed: {count}</h1>;
}
```

你需要在 `useCounter.js` 中编写你的自定义 Hook，并将它导入到 `App.js` 文件中。

<Sandpack>

```js
import { useState, useEffect } from 'react';

export default function Counter() {
  const [count, setCount] = useState(0);
  useEffect(() => {
    const id = setInterval(() => {
      setCount(c => c + 1);
    }, 1000);
    return () => clearInterval(id);
  }, []);
  return <h1>Seconds passed: {count}</h1>;
}
```

```js src/useCounter.js
// 在这个文件中编写你的自定义 Hook！
```

</Sandpack>

<Solution>

你的代码应该像这样：

<Sandpack>

```js
import { useCounter } from './useCounter.js';

export default function Counter() {
  const count = useCounter();
  return <h1>Seconds passed: {count}</h1>;
}
```

```js src/useCounter.js
import { useState, useEffect } from 'react';

export function useCounter() {
  const [count, setCount] = useState(0);
  useEffect(() => {
    const id = setInterval(() => {
      setCount(c => c + 1);
    }, 1000);
    return () => clearInterval(id);
  }, []);
  return count;
}
```

</Sandpack>

注意，`App.js` 已经不再需要导入 `useState` 或 `useEffect` 了。

</Solution>

#### 让计数器的延迟可配置 {/*make-the-counter-delay-configurable*/}

在这个例子中，有一个由滑块控制的 `delay` state 变量，但它的值没有被使用。把 `delay` 值传给你的自定义 `useCounter` Hook，并修改 `useCounter` Hook，让它使用传入的 `delay`，而不是硬编码的 `1000` ms。

<Sandpack>

```js
import { useState } from 'react';
import { useCounter } from './useCounter.js';

export default function Counter() {
  const [delay, setDelay] = useState(1000);
  const count = useCounter();
  return (
    <>
      <label>
        Tick duration: {delay} ms
        <br />
        <input
          type="range"
          value={delay}
          min="10"
          max="2000"
          onChange={e => setDelay(Number(e.target.value))}
        />
      </label>
      <hr />
      <h1>Ticks: {count}</h1>
    </>
  );
}
```

```js src/useCounter.js
import { useState, useEffect } from 'react';

export function useCounter() {
  const [count, setCount] = useState(0);
  useEffect(() => {
    const id = setInterval(() => {
      setCount(c => c + 1);
    }, 1000);
    return () => clearInterval(id);
  }, []);
  return count;
}
```

</Sandpack>

<Solution>

通过 `useCounter(delay)` 把 `delay` 传给你的 Hook。然后在 Hook 内部，使用 `delay` 而不是硬编码的 `1000`。你需要把 `delay` 加到 Effect 的依赖项中。这可以确保 `delay` 的变化会重置这个间隔。

<Sandpack>

```js
import { useState } from 'react';
import { useCounter } from './useCounter.js';

export default function Counter() {
  const [delay, setDelay] = useState(1000);
  const count = useCounter(delay);
  return (
    <>
      <label>
        Tick duration: {delay} ms
        <br />
        <input
          type="range"
          value={delay}
          min="10"
          max="2000"
          onChange={e => setDelay(Number(e.target.value))}
        />
      </label>
      <hr />
      <h1>Ticks: {count}</h1>
    </>
  );
}
```

```js src/useCounter.js
import { useState, useEffect } from 'react';

export function useCounter(delay) {
  const [count, setCount] = useState(0);
  useEffect(() => {
    const id = setInterval(() => {
      setCount(c => c + 1);
    }, delay);
    return () => clearInterval(id);
  }, [delay]);
  return count;
}
```

</Sandpack>

</Solution>

#### 从 `useCounter` 中提取 `useInterval` {/*extract-useinterval-out-of-usecounter*/}

目前，你的 `useCounter` Hook 做了两件事。它设置了一个间隔，同时还在每次间隔触发时递增一个 state 变量。把设置间隔的逻辑拆分到一个单独的 Hook 中，叫做 `useInterval`。它应该接收两个参数：`onTick` 回调和 `delay`。完成这个改动后，你的 `useCounter` 实现应该看起来像这样：

```js
export function useCounter(delay) {
  const [count, setCount] = useState(0);
  useInterval(() => {
    setCount(c => c + 1);
  }, delay);
  return count;
}
```

在 `useInterval.js` 文件中编写 `useInterval`，并将它导入到 `useCounter.js` 文件中。

<Sandpack>

```js
import { useCounter } from './useCounter.js';

export default function Counter() {
  const count = useCounter(1000);
  return <h1>Seconds passed: {count}</h1>;
}
```

```js src/useCounter.js
import { useState, useEffect } from 'react';

export function useCounter(delay) {
  const [count, setCount] = useState(0);
  useEffect(() => {
    const id = setInterval(() => {
      setCount(c => c + 1);
    }, delay);
    return () => clearInterval(id);
  }, [delay]);
  return count;
}
```

```js src/useInterval.js
// 在这里编写你的 Hook！
```

</Sandpack>

<Solution>

`useInterval` 内部的逻辑应该负责设置和清除这个间隔。它不需要做任何别的事。

<Sandpack>

```js
import { useCounter } from './useCounter.js';

export default function Counter() {
  const count = useCounter(1000);
  return <h1>Seconds passed: {count}</h1>;
}
```

```js src/useCounter.js
import { useState } from 'react';
import { useInterval } from './useInterval.js';

export function useCounter(delay) {
  const [count, setCount] = useState(0);
  useInterval(() => {
    setCount(c => c + 1);
  }, delay);
  return count;
}
```

```js src/useInterval.js active
import { useEffect } from 'react';

export function useInterval(onTick, delay) {
  useEffect(() => {
    const id = setInterval(onTick, delay);
    return () => clearInterval(id);
  }, [onTick, delay]);
}
```

</Sandpack>

注意，这个解决方案有一点小问题，你会在下一个挑战中解决它。

</Solution>

#### 修复一个会重置的间隔 {/*fix-a-resetting-interval*/}

在这个例子中，有 *两个* 独立的间隔。

`App` 组件调用了 `useCounter`，而 `useCounter` 又调用 `useInterval` 来每秒更新计数器。但 `App` 组件 *还* 调用了 `useInterval` 来每两秒随机更新页面背景颜色。

由于某些原因，更新页面背景的回调从未运行。在 `useInterval` 中添加一些日志：

```js {2,5}
  useEffect(() => {
    console.log('✅ 使用延迟 ', delay, ' 设置一个间隔')
    const id = setInterval(onTick, delay);
    return () => {
      console.log('❌ 使用延迟 ', delay, ' 清除一个间隔')
      clearInterval(id);
    };
  }, [onTick, delay]);
```

这些日志和你预期的行为一致吗？如果你的某些 Effects 似乎在不必要地重新同步，你能猜出是哪个依赖项导致的吗？有没有办法从这个 Effect 中[移除那个依赖项](/learn/removing-effect-dependencies)？

修复这个问题后，你应该能看到页面背景每两秒更新一次。

<Hint>

看起来你的 `useInterval` Hook 把一个事件监听器作为参数接收。你能想到一种方法把这个事件监听器包装起来，使它不需要作为你 Effect 的依赖项吗？

</Hint>

<Sandpack>

```js
import { useCounter } from './useCounter.js';
import { useInterval } from './useInterval.js';

export default function Counter() {
  const count = useCounter(1000);

  useInterval(() => {
    const randomColor = `hsla(${Math.random() * 360}, 100%, 50%, 0.2)`;
    document.body.style.backgroundColor = randomColor;
  }, 2000);

  return <h1>Seconds passed: {count}</h1>;
}
```

```js src/useCounter.js
import { useState } from 'react';
import { useInterval } from './useInterval.js';

export function useCounter(delay) {
  const [count, setCount] = useState(0);
  useInterval(() => {
    setCount(c => c + 1);
  }, delay);
  return count;
}
```

```js src/useInterval.js
import { useEffect } from 'react';
import { useEffectEvent } from 'react';

export function useInterval(onTick, delay) {
  useEffect(() => {
    const id = setInterval(onTick, delay);
    return () => {
      clearInterval(id);
    };
  }, [onTick, delay]);
}
```

</Sandpack>

<Solution>

在 `useInterval` 内部，将 tick 回调包装成一个 Effect Event，就像你在[本页前面](/learn/reusing-logic-with-custom-hooks#passing-event-handlers-to-custom-hooks)那样做的。

这样就可以把 `onTick` 从 Effect 的依赖项中移除。Effect 就不会在组件每次重新渲染时重新同步，因此页面背景颜色的间隔不会每秒在有机会触发之前就被重置。

完成这个改动后，两个间隔都能按预期工作，而且彼此不会干扰：

<Sandpack>


```js
import { useCounter } from './useCounter.js';
import { useInterval } from './useInterval.js';

export default function Counter() {
  const count = useCounter(1000);

  useInterval(() => {
    const randomColor = `hsla(${Math.random() * 360}, 100%, 50%, 0.2)`;
    document.body.style.backgroundColor = randomColor;
  }, 2000);

  return <h1>Seconds passed: {count}</h1>;
}
```

```js src/useCounter.js
import { useState } from 'react';
import { useInterval } from './useInterval.js';

export function useCounter(delay) {
  const [count, setCount] = useState(0);
  useInterval(() => {
    setCount(c => c + 1);
  }, delay);
  return count;
}
```

```js src/useInterval.js active
import { useEffect } from 'react';
import { useEffectEvent } from 'react';

export function useInterval(callback, delay) {
  const onTick = useEffectEvent(callback);
  useEffect(() => {
    const id = setInterval(onTick, delay);
    return () => clearInterval(id);
  }, [delay]);
}
```

</Sandpack>

</Solution>

#### 实现一种错位移动 {/*implement-a-staggering-movement*/}

在这个例子中，`usePointerPosition()` Hook 会跟踪当前指针的位置。试着在预览区域内移动你的鼠标或手指，看看红点跟随你的移动。它的位置保存在 `pos1` 变量中。

事实上，页面上渲染了五个不同的红点（！）。你看不到它们，因为它们目前都出现在同一个位置。你需要修复这个问题。你要实现的是一种“错位”的移动：每个点都应该“跟随”前一个点的轨迹。例如，如果你快速移动鼠标，第一个点应该立即跟随，第二个点应该稍微延迟地跟随第一个点，第三个点再跟随第二个点，以此类推。

你需要实现 `useDelayedValue` 自定义 Hook。它当前的实现直接返回传入的 `value`。而你希望返回 `delay` 毫秒之前的值。为此，你可能需要一些 state 和一个 Effect。

在实现 `useDelayedValue` 之后，你应该能看到这些点一个接一个地移动。

<Hint>

你需要在自定义 Hook 内部把 `delayedValue` 存成一个 state 变量。当 `value` 变化时，你需要运行一个 Effect。这个 Effect 应该在 `delay` 之后更新 `delayedValue`。你可能会发现调用 `setTimeout` 很有帮助。

这个 Effect 需要清理吗？为什么需要，或者为什么不需要？

</Hint>

<Sandpack>

```js
import { usePointerPosition } from './usePointerPosition.js';

function useDelayedValue(value, delay) {
  // TODO: 实现这个 Hook
  return value;
}

export default function Canvas() {
  const pos1 = usePointerPosition();
  const pos2 = useDelayedValue(pos1, 100);
  const pos3 = useDelayedValue(pos2, 200);
  const pos4 = useDelayedValue(pos3, 100);
  const pos5 = useDelayedValue(pos3, 50);
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

```css
body { min-height: 300px; }
```

</Sandpack>

<Solution>

这是一个可工作的版本。你把 `delayedValue` 保持为一个 state 变量。当 `value` 更新时，你的 Effect 会安排一个超时来更新 `delayedValue`。这就是为什么 `delayedValue` 总是“落后”于实际的 `value`。

<Sandpack>

```js
import { useState, useEffect } from 'react';
import { usePointerPosition } from './usePointerPosition.js';

function useDelayedValue(value, delay) {
  const [delayedValue, setDelayedValue] = useState(value);

  useEffect(() => {
    setTimeout(() => {
      setDelayedValue(value);
    }, delay);
  }, [value, delay]);

  return delayedValue;
}

export default function Canvas() {
  const pos1 = usePointerPosition();
  const pos2 = useDelayedValue(pos1, 100);
  const pos3 = useDelayedValue(pos2, 200);
  const pos4 = useDelayedValue(pos3, 100);
  const pos5 = useDelayedValue(pos3, 50);
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

```css
body { min-height: 300px; }
```

</Sandpack>

注意，这个 Effect **不需要**清理。如果你在清理函数中调用 `clearTimeout`，那么每次 `value` 变化时，它都会重置已经安排好的超时。为了保持移动的连续性，你希望所有超时都能触发。

</Solution>

</Challenges>
