---
title: useEffectEvent
---

<Intro>

`useEffectEvent` 是一个 React Hook，它让你可以将事件与 Effects 分离。

```js
const onEvent = useEffectEvent(callback)
```

</Intro>

<InlineToc />

---

## 参考 {/*reference*/}

### `useEffectEvent(callback)` {/*useeffectevent*/}

在组件顶层调用 `useEffectEvent` 来创建一个 Effect Event。

```js {4,6}
import { useEffectEvent, useEffect } from 'react';

function ChatRoom({ roomId, theme }) {
  const onConnected = useEffectEvent(() => {
    showNotification('已连接！', theme);
  });
}
```

Effect Events 是你 Effect 逻辑的一部分，但它们的行为更像事件处理函数。它们总是能“看到”渲染中的最新值（例如 props 和 state），而不会重新同步你的 Effect，因此它们会被排除在 Effect 依赖项之外。查看[将事件与 Effects 分离](/learn/separating-events-from-effects#extracting-non-reactive-logic-out-of-effects)以了解更多。

[查看更多示例。](#usage)

#### 参数 {/*parameters*/}

* `callback`：包含你的 Effect Event 逻辑的函数。该函数可以接受任意数量的参数并返回任意值。当你调用返回的 Effect Event 函数时，`callback` 总是在调用时访问最近一次提交的渲染值。

#### 返回值 {/*returns*/}

`useEffectEvent` 返回一个 Effect Event 函数，其类型签名与 `callback` 相同。

你可以在 `useEffect`、`useLayoutEffect`、`useInsertionEffect` 中调用这个函数，或者在同一组件内的其他 Effect Events 中调用它。

#### 注意事项 {/*caveats*/}

* `useEffectEvent` 是一个 Hook，所以你只能在**组件顶层**或你自己的 Hooks 中调用它。你不能在循环或条件语句中调用它。如果你需要这样做，请拆分出一个新组件，并把 Effect Event 放进去。
* Effect Events 只能从 Effects 或其他 Effect Events 内部调用。不要在渲染期间调用它们，也不要把它们传给其他组件或 Hooks。[`eslint-plugin-react-hooks`](/reference/eslint-plugin-react-hooks) 规则会强制执行这一限制。
* 不要使用 `useEffectEvent` 来逃避在 Effect 依赖数组中声明依赖。这会隐藏 bug，并让代码更难理解。只在它确实用于从 Effects 触发的事件逻辑时使用。
* Effect Event 函数没有稳定的标识。它们的标识会在每次渲染时有意改变。

<DeepDive>

#### 为什么 Effect Events 不是稳定的？ {/*why-are-effect-events-not-stable*/}

与来自 `useState` 的 `set` 函数或 refs 不同，Effect Event 函数没有稳定的标识。它们的标识会在每次渲染时有意改变：

```js
// 🔴 错误：将 Effect Event 包含在依赖项中
useEffect(() => {
  onSomething();
}, [onSomething]); // ESLint 会对此发出警告
```

这是一个刻意的设计选择。Effect Events 只应从同一组件内的 Effects 中调用。由于你只能在本地调用它们，不能将它们传给其他组件，也不能把它们放进依赖数组中，因此稳定的标识并没有用途，反而会掩盖 bug。

这种不稳定的标识充当了运行时断言：如果你的代码错误地依赖于函数标识，你会看到 Effect 在每次渲染时都重新运行，从而让 bug 变得明显。

这种设计进一步强化了这样一个概念：Effect Events 在概念上属于某个特定的 effect，而不是一个用于规避响应性的通用 API。

</DeepDive>

---

## 用法 {/*usage*/}


### 在 Effect 中使用事件 {/*using-an-event-in-an-effect*/}

在组件顶层调用 `useEffectEvent` 来创建一个 *Effect Event*：


```js [[1, 1, "onConnected"]]
const onConnected = useEffectEvent(() => {
  if (!muted) {
    showNotification('已连接！');
  }
});
```

`useEffectEvent` 接受一个 `event callback` 并返回一个 <CodeStep step={1}>Effect Event</CodeStep>。Effect Event 是一个可以在 Effects 内部调用而不会导致 Effect 重新连接的函数：

```js [[1, 3, "onConnected"]]
useEffect(() => {
  const connection = createConnection(roomId);
  connection.on('connected', onConnected);
  connection.connect();
  return () => {
    connection.disconnect();
  }
}, [roomId]);
```

由于 `onConnected` 是一个 <CodeStep step={1}>Effect Event</CodeStep>，`muted` 和 `onConnect` 不需要出现在 Effect 依赖项中。

<Pitfall>

##### 不要使用 Effect Events 来跳过依赖项 {/*pitfall-skip-dependencies*/}

你可能会想用 `useEffectEvent` 来避免列出你认为“没必要”的依赖项。然而，这会隐藏 bug，并让代码更难理解：

```js
// 🔴 错误：使用 Effect Events 来隐藏依赖项
const logVisit = useEffectEvent(() => {
  log(pageUrl);
});

useEffect(() => {
  logVisit()
}, []); // 缺少 pageUrl 会导致你漏记日志
```

如果某个值应该让你的 Effect 重新运行，就把它保留为依赖项。只有在确实不应该重新触发 Effect 的逻辑中才使用 Effect Events。

查看[将事件与 Effects 分离](/learn/separating-events-from-effects)以了解更多。

</Pitfall>

---

### 在定时器中使用最新值 {/*using-a-timer-with-latest-values*/}

当你在 Effect 中使用 `setInterval` 或 `setTimeout` 时，你通常希望在回调中读取渲染中的最新值，而不是在这些值变化时重启定时器。

这个计数器每秒会按照当前 `increment` 的值增加 `count`。`onTick` Effect Event 会读取最新的 `count` 和 `increment`，而不会导致间隔重新开始：

<Sandpack>

```js
import { useState, useEffect, useEffectEvent } from 'react';

export default function Timer() {
  const [count, setCount] = useState(0);
  const [increment, setIncrement] = useState(1);

  const onTick = useEffectEvent(() => {
    setCount(count + increment);
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

尝试在定时器运行时更改 increment 的值。计数器会立即使用新的 increment 值，但定时器会继续平稳地运行，而不会重启。

---

### 在事件监听器中使用最新值 {/*using-an-event-listener-with-latest-values*/}

当你在 Effect 中设置事件监听器时，通常需要在回调中读取渲染中的最新值。若没有 `useEffectEvent`，你就需要把这些值包含在依赖项中，这会导致监听器在每次变化时被移除并重新添加。

这个示例展示了一个会跟随光标移动的点，但仅在勾选“允许移动”时生效。`onMove` Effect Event 总是会读取最新的 `canMove` 值，而不会重新运行 Effect：

<Sandpack>

```js
import { useState, useEffect, useEffectEvent } from 'react';

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
        <input
          type="checkbox"
          checked={canMove}
          onChange={e => setCanMove(e.target.checked)}
        />
        允许点移动
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

切换复选框并移动鼠标。点会立即响应复选框状态，但事件监听器只会在组件挂载时设置一次。

---

### 避免重新连接外部系统 {/*showing-a-notification-without-reconnecting*/}

`useEffectEvent` 的一个常见用例是：当你想在 Effect 响应中做某件事，但那件事依赖于一个你不希望对其产生响应的值。

在这个示例中，一个聊天组件连接到房间，并在连接后显示通知。用户可以通过复选框将通知静音。不过，你并不希望用户每次更改设置时都重新连接到聊天房间：

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
import { useState, useEffect, useEffectEvent } from 'react';
import { createConnection } from './chat.js';
import { showNotification } from './notifications.js';

function ChatRoom({ roomId, muted }) {
  const onConnected = useEffectEvent((roomId) => {
    console.log('✅ 已连接到 ' + roomId + '（静音：' + muted + '）');
    if (!muted) {
      showNotification('已连接到 ' + roomId);
    }
  });

  useEffect(() => {
    const connection = createConnection(roomId);
    console.log('⏳ 正在连接到 ' + roomId + '...');
    connection.on('connected', () => {
      onConnected(roomId);
    });
    connection.connect();
    return () => {
      console.log('❌ 已与 ' + roomId + ' 断开连接');
      connection.disconnect();
    }
  }, [roomId]);

  return <h1>欢迎来到 {roomId} 房间！</h1>;
}

export default function App() {
  const [roomId, setRoomId] = useState('general');
  const [muted, setMuted] = useState(false);
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
          checked={muted}
          onChange={e => setMuted(e.target.checked)}
        />
        静音通知
      </label>
      <hr />
      <ChatRoom
        roomId={roomId}
        muted={muted}
      />
    </>
  );
}
```

```js src/chat.js
const serverUrl = 'https://localhost:1234';

export function createConnection(roomId) {
  // 一个真实的实现实际上会连接到服务器
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

尝试切换房间。聊天会重新连接并显示通知。现在将通知静音。由于 `muted` 是在 Effect Event 中读取的，而不是在 Effect 中读取的，所以聊天会保持连接。

---

### 在自定义 Hooks 中使用 Effect Events {/*using-effect-events-in-custom-hooks*/}

你可以在自己的自定义 Hooks 中使用 `useEffectEvent`。这让你可以创建可复用的 Hooks，在封装 Effects 的同时保持某些值为非响应式：

<Sandpack>

```js
import { useState, useEffect, useEffectEvent } from 'react';

function useInterval(callback, delay) {
  const onTick = useEffectEvent(callback);

  useEffect(() => {
    if (delay === null) {
      return;
    }
    const id = setInterval(() => {
      onTick();
    }, delay);
    return () => clearInterval(id);
  }, [delay]);
}

function Counter({ incrementBy }) {
  const [count, setCount] = useState(0);

  useInterval(() => {
    setCount(c => c + incrementBy);
  }, 1000);

  return (
    <div>
      <h2>计数：{count}</h2>
      <p>每秒增加 {incrementBy}</p>
    </div>
  );
}

export default function App() {
  const [incrementBy, setIncrementBy] = useState(1);

  return (
    <>
      <label>
        每次增加：{' '}
        <select
          value={incrementBy}
          onChange={(e) => setIncrementBy(Number(e.target.value))}
        >
          <option value={1}>1</option>
          <option value={5}>5</option>
          <option value={10}>10</option>
        </select>
      </label>
      <hr />
      <Counter incrementBy={incrementBy} />
    </>
  );
}
```

```css
label { display: block; margin-bottom: 8px; }
```

</Sandpack>

在这个示例中，`useInterval` 是一个设置定时器的自定义 Hook。传给它的 `callback` 会被包装在一个 Effect Event 中，因此即使每次渲染都会传入一个新的 `callback`，定时器也不会重置。

---

## 故障排除 {/*troubleshooting*/}

### 我遇到一个错误："A function wrapped in useEffectEvent can't be called during rendering" {/*cant-call-during-rendering*/}

这个错误意味着你在组件的渲染阶段调用了一个 Effect Event 函数。Effect Event 只能从 Effect 或其他 Effect Event 内部调用。

```js
function MyComponent({ data }) {
  const onLog = useEffectEvent(() => {
    console.log(data);
  });

  // 🔴 错误：在渲染期间调用
  onLog();

  // ✅ 正确：从 Effect 中调用
  useEffect(() => {
    onLog();
  }, []);

  return <div>{data}</div>;
}
```

如果你需要在渲染期间运行逻辑，不要把它包装到 `useEffectEvent` 中。直接调用这段逻辑，或者把它移到 Effect 里。

---

### 我遇到一个 lint 错误："Functions returned from useEffectEvent must not be included in the dependency array" {/*effect-event-in-deps*/}

如果你看到类似 "Functions returned from `useEffectEvent` must not be included in the dependency array" 的警告，请从依赖项中移除这个 Effect Event：

```js
const onSomething = useEffectEvent(() => {
  // ...
});

// 🔴 错误：依赖项中包含 Effect Event
useEffect(() => {
  onSomething();
}, [onSomething]);

// ✅ 正确：依赖项中不包含 Effect Event
useEffect(() => {
  onSomething();
}, []);
```

Effect Event 的设计是可以在 Effect 中调用，而不必把它列为依赖项。lint 工具会强制这样做，因为这个函数的身份 [有意地不稳定](#why-are-effect-events-not-stable)。如果把它包含进去，你的 Effect 就会在每次渲染时重新运行。

---

### 我遇到一个 lint 错误："... is a function created with useEffectEvent, and can only be called from Effects" {/*effect-event-called-outside-effect*/}

如果你看到类似 "... is a function created with React Hook `useEffectEvent`, and can only be called from Effects and Effect Events" 的警告，说明你调用这个函数的位置不对：

```js
const onSomething = useEffectEvent(() => {
  console.log(value);
});

// 🔴 错误：从事件处理器中调用
function handleClick() {
  onSomething();
}

// 🔴 错误：传递给子组件
return <Child onSomething={onSomething} />;

// ✅ 正确：从 Effect 中调用
useEffect(() => {
  onSomething();
}, []);
```

Effect Event 的专门用途是在其定义所在组件内部的 Effect 中使用。如果你需要一个用于事件处理器的回调，或者需要传递给子组件，请改用普通函数或 `useCallback`。