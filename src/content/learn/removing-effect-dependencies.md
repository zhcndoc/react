---
title: '移除 Effect 依赖'
---

<Intro>

当你编写一个 Effect 时，linter 会验证你是否已经把这个 Effect 读取的每个响应式值（比如 props 和 state）都包含在 Effect 的依赖列表中。这可以确保你的 Effect 始终与组件的最新 props 和 state 保持同步。不必要的依赖可能会导致你的 Effect 运行过于频繁，甚至创建无限循环。请按照本指南来检查并移除 Effect 中不必要的依赖。

</Intro>

<YouWillLearn>

- 如何修复无限的 Effect 依赖循环
- 当你想移除一个依赖时该怎么做
- 如何在不“响应”某个值的情况下从 Effect 中读取它
- 如何以及为什么要避免对象和函数依赖
- 为什么屏蔽 dependency linter 很危险，以及应该如何替代

</YouWillLearn>

## 依赖应该与代码匹配 {/*dependencies-should-match-the-code*/}

当你编写一个 Effect 时，你首先要指定如何 [开始和停止](/learn/lifecycle-of-reactive-effects#the-lifecycle-of-an-effect) 你希望 Effect 执行的内容：

```js {5-7}
const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  	// ...
}
```

然后，如果你把 Effect 的依赖留空（`[]`），linter 会建议正确的依赖：

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
  }, []); // <-- 在这里修正错误！
  return <h1>Welcome to the {roomId} room!</h1>;
}

export default function App() {
  const [roomId, setRoomId] = useState('general');
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
      <hr />
      <ChatRoom roomId={roomId} />
    </>
  );
}
```

```js src/chat.js
export function createConnection(serverUrl, roomId) {
  // 一个真实的实现实际上会连接到服务器
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

按照 linter 的提示把它们补全：

```js {6}
function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]); // ✅ 已声明所有依赖
  // ...
}
```

[Effects 会对响应式值“做出反应”。](/learn/lifecycle-of-reactive-effects#effects-react-to-reactive-values) 由于 `roomId` 是一个响应式值（它可能因为重新渲染而改变），linter 会验证你是否将它指定为依赖项。如果 `roomId` 接收到了不同的值，React 会重新同步你的 Effect。这确保了聊天会保持连接到所选房间，并且会对下拉菜单变化“作出反应”：

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
      <hr />
      <ChatRoom roomId={roomId} />
    </>
  );
}
```

```js src/chat.js
export function createConnection(serverUrl, roomId) {
  // 一个真实的实现实际上会连接到服务器
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

### 要移除一个依赖，就要证明它不是依赖 {/*to-remove-a-dependency-prove-that-its-not-a-dependency*/}

注意，你不能“选择” Effect 的依赖。你的 Effect 代码使用的每一个 <CodeStep step={2}>响应式值</CodeStep> 都必须声明在依赖列表中。依赖列表由周围的代码决定：

```js [[2, 3, "roomId"], [2, 5, "roomId"], [2, 8, "roomId"]]
const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) { // 这是一个响应式值
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId); // 这个 Effect 读取了那个响应式值
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]); // ✅ 所以你必须把那个响应式值指定为 Effect 的依赖
  // ...
}
```

[响应式值](/learn/lifecycle-of-reactive-effects#all-variables-declared-in-the-component-body-are-reactive) 包括 props 以及所有直接在组件内部声明的变量和函数。由于 `roomId` 是一个响应式值，你不能把它从依赖列表中移除。linter 不会允许这样做：

```js {8}
const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, []); // 🔴 React Hook useEffect 缺少依赖项：'roomId'
  // ...
}
```

而且 linter 是对的！由于 `roomId` 可能会随着时间变化，这会在你的代码中引入 bug。

**要移除一个依赖，就要向 linter “证明”它*不需要*成为依赖。** 例如，你可以把 `roomId` 移到组件外部，以证明它不是响应式的，也不会在重新渲染时改变：

```js {2,9}
const serverUrl = 'https://localhost:1234';
const roomId = 'music'; // 不再是响应式值

function ChatRoom() {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, []); // ✅ 已声明所有依赖
  // ...
}
```

现在 `roomId` 不是响应式值了（并且不会在重新渲染时改变），因此它不需要成为依赖：

<Sandpack>

```js
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

const serverUrl = 'https://localhost:1234';
const roomId = 'music';

export default function ChatRoom() {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, []);
  return <h1>Welcome to the {roomId} room!</h1>;
}
```

```js src/chat.js
export function createConnection(serverUrl, roomId) {
  // 一个真实的实现实际上会连接到服务器
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

这就是为什么你现在可以指定一个 [空的（`[]`）依赖列表。](/learn/lifecycle-of-reactive-effects#what-an-effect-with-empty-dependencies-means) 你的 Effect *确实* 不再依赖任何响应式值，因此当组件的任何 props 或 state 发生变化时，它 *确实* 不需要重新运行。

### 要改变依赖，就要改变代码 {/*to-change-the-dependencies-change-the-code*/}

你可能已经在工作流中注意到了一个模式：

1. 首先，你**修改 Effect 的代码**或响应式值的声明方式。
2. 然后，你遵循 linter 并调整依赖，使其**与已修改的代码匹配。**
3. 如果你对依赖列表不满意，你就**回到第一步**（再次修改代码）。

最后这一点很重要。**如果你想改变依赖，先改变周围的代码。** 你可以把依赖列表看作是 [Effect 代码使用的所有响应式值的列表。](/learn/lifecycle-of-reactive-effects#react-verifies-that-you-specified-every-reactive-value-as-a-dependency) 你并不是在*选择*要把什么放进这个列表里。这个列表是在*描述*你的代码。要改变依赖列表，就改变代码。

这可能会让人觉得像在解方程。你可能会先有一个目标（例如移除一个依赖），然后你需要“找到”符合这个目标的代码。并不是每个人都觉得解方程有趣，而写 Effects 也可能如此！幸运的是，下面有一系列常见的处理方法供你尝试。

<Pitfall>

如果你已有一个现成的代码库，你可能会有一些像这样屏蔽 linter 的 Effect：

```js {3-4}
useEffect(() => {
  // ...
  // 🔴 避免像这样屏蔽 linter：
  // eslint-ignore-next-line react-hooks/exhaustive-deps
}, []);
```

**当依赖与代码不匹配时，引入 bug 的风险非常高。** 通过屏蔽 linter，你是在向 React “撒谎”，隐瞒了你的 Effect 依赖的值。

相反，请使用下面这些技巧。

</Pitfall>

<DeepDive>

#### 为什么屏蔽 dependency linter 如此危险？ {/*why-is-suppressing-the-dependency-linter-so-dangerous*/}

屏蔽 linter 会导致非常不直观、难以发现和修复的 bug。这里有一个例子：

<Sandpack>

```js {expectedErrors: {'react-compiler': [14]}}
import { useState, useEffect } from 'react';

export default function Timer() {
  const [count, setCount] = useState(0);
  const [increment, setIncrement] = useState(1);

  function onTick() {
	setCount(count + increment);
  }

  useEffect(() => {
    const id = setInterval(onTick, 1000);
    return () => clearInterval(id);
    // eslint-disable-next-line react-hooks/exhaustive-deps
  }, []);

  return (
    <>
      <h1>
        Counter: {count}
        <button onClick={() => setCount(0)}>Reset</button>
      </h1>
      <hr />
      <p>
        Every second, increment by:
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

假设你想让这个 Effect“只在挂载时运行”。你读到过 [空的（`[]`）依赖](/learn/lifecycle-of-reactive-effects#what-an-effect-with-empty-dependencies-means) 可以做到这一点，于是你决定忽略 linter，并强行将 `[]` 指定为依赖。

这个计数器本应每秒按两个按钮可配置的数值递增。然而，由于你对 React “撒谎”说这个 Effect 不依赖任何东西，React 会永远继续使用初始渲染时的 `onTick` 函数。[在那次渲染期间，](/learn/state-as-a-snapshot#rendering-takes-a-snapshot-in-time) `count` 是 `0`，`increment` 是 `1`。这就是为什么那次渲染中的 `onTick` 每秒都会一直调用 `setCount(0 + 1)`，而你总是只看到 `1`。这类 bug 如果分散在多个组件中，会更难修复。

永远都存在比忽略 linter 更好的解决方案！要修复这段代码，你需要把 `onTick` 加入依赖列表。（为了确保 interval 只设置一次，[把 `onTick` 变成一个 Effect Event。](/learn/separating-events-from-effects#reading-latest-props-and-state-with-effect-events)）

**我们建议把 dependency lint 错误视为编译错误。如果你不屏蔽它，你就永远不会看到这类 bug。** 本页其余部分会记录针对这一情况以及其他情况的替代方案。

</DeepDive>

## 移除不必要的依赖 {/*removing-unnecessary-dependencies*/}

每次你调整 Effect 的依赖项以反映代码时，都要看看依赖列表。Effect 在这些依赖项中的任何一个变化时重新运行，这样做合理吗？有时，答案是“不”：

* 你可能希望在不同条件下重新执行 Effect 的*不同部分*。
* 你可能只想读取某个依赖的*最新值*，而不是“响应”它的变化。
* 某个依赖可能由于它是对象或函数而*无意中*变化得太频繁。

要找到正确的解决方案，你需要先回答一些关于 Effect 的问题。我们来一起看看。

### 这段代码应该移到事件处理函数中吗？ {/*should-this-code-move-to-an-event-handler*/}

你首先应该思考的是，这段代码是否应该是一个 Effect。

想象一个表单。在提交时，你把 `submitted` 状态变量设为 `true`。你需要发送一个 POST 请求并显示一条通知。你把这段逻辑放进了一个会在 `submitted` 为 `true` 时“响应”的 Effect 中：

```js {6-8}
function Form() {
  const [submitted, setSubmitted] = useState(false);

  useEffect(() => {
    if (submitted) {
      // 🔴 避免：Effect 中包含特定于事件的逻辑
      post('/api/register');
      showNotification('Successfully registered!');
    }
  }, [submitted]);

  function handleSubmit() {
    setSubmitted(true);
  }

  // ...
}
```

后来，你想根据当前主题来设置通知消息的样式，所以你读取了当前主题。由于 `theme` 是在组件主体中声明的，所以它是一个响应式值，因此你把它添加为依赖项：

```js {3,9,11}
function Form() {
  const [submitted, setSubmitted] = useState(false);
  const theme = useContext(ThemeContext);

  useEffect(() => {
    if (submitted) {
      // 🔴 避免：Effect 中包含特定于事件的逻辑
      post('/api/register');
      showNotification('Successfully registered!', theme);
    }
  }, [submitted, theme]); // ✅ 已声明所有依赖项

  function handleSubmit() {
    setSubmitted(true);
  }

  // ...
}
```

这样做之后，你引入了一个 bug。假设你先提交表单，然后在深色和浅色主题之间切换。`theme` 会改变，Effect 会重新运行，于是同样的通知又显示了一次！

**这里的问题在于，这段代码一开始就不应该是一个 Effect。** 你想要在*提交表单*这个特定交互发生时发送这个 POST 请求并显示通知。要响应某个特定交互执行一些代码，就把这段逻辑直接放到对应的事件处理函数里：

```js {6-7}
function Form() {
  const theme = useContext(ThemeContext);

  function handleSubmit() {
    // ✅ 好：特定于事件的逻辑由事件处理函数调用
    post('/api/register');
    showNotification('Successfully registered!', theme);
  }

  // ...
}
```

现在代码已经在事件处理函数中了，它不再具有响应性——因此它只会在用户提交表单时运行。你可以阅读更多关于[在事件处理函数和 Effect 之间做选择](/learn/separating-events-from-effects#reactive-values-and-reactive-logic)以及[如何删除不必要的 Effect](/learn/you-might-not-need-an-effect)的内容。

### 你的 Effect 是否在做几个互不相关的事情？ {/*is-your-effect-doing-several-unrelated-things*/}

你接下来应该问自己的问题是，Effect 是否在做几个互不相关的事情。

假设你正在创建一个配送表单，用户需要选择所在城市和区域。你根据所选的 `country` 从服务器获取 `cities` 列表，并在下拉框中展示它们：

```js
function ShippingForm({ country }) {
  const [cities, setCities] = useState(null);
  const [city, setCity] = useState(null);

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
  }, [country]); // ✅ 已声明所有依赖项

  // ...
```

这是一个[在 Effect 中获取数据](/learn/you-might-not-need-an-effect#fetching-data)的好例子。你正在根据 `country` 属性将 `cities` 状态与网络同步。你不能把它放到事件处理函数里，因为你需要在 `ShippingForm` 显示出来时，以及 `country` 改变时立即获取数据（无论是哪种交互导致的）。

现在假设你要再添加一个用于城市区域的第二个选择框，它应该根据当前选中的 `city` 获取 `areas`。你可能会先在同一个 Effect 里再添加一次 `fetch` 调用，用来获取区域列表：

```js {15-24,28}
function ShippingForm({ country }) {
  const [cities, setCities] = useState(null);
  const [city, setCity] = useState(null);
  const [areas, setAreas] = useState(null);

  useEffect(() => {
    let ignore = false;
    fetch(`/api/cities?country=${country}`)
      .then(response => response.json())
      .then(json => {
        if (!ignore) {
          setCities(json);
        }
      });
    // 🔴 避免：单个 Effect 同步两个独立的过程
    if (city) {
      fetch(`/api/areas?city=${city}`)
        .then(response => response.json())
        .then(json => {
          if (!ignore) {
            setAreas(json);
          }
        });
    }
    return () => {
      ignore = true;
    };
  }, [country, city]); // ✅ 已声明所有依赖项

  // ...
```

然而，由于这个 Effect 现在使用了 `city` 状态变量，你不得不把 `city` 加入依赖列表。这样一来，又引入了一个问题：当用户选择不同的城市时，Effect 会重新运行并调用 `fetchCities(country)`。结果就是，你会不必要地多次重新获取城市列表。

**这段代码的问题在于，你在同步两个不同且互不相关的东西：**

1. 你想根据 `country` 属性把 `cities` 状态同步到网络。
1. 你想根据 `city` 状态把 `areas` 状态同步到网络。

把逻辑拆分成两个 Effect，每个 Effect 只响应它需要同步的那个属性：

```js {19-33}
function ShippingForm({ country }) {
  const [cities, setCities] = useState(null);
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
  }, [country]); // ✅ 已声明所有依赖项

  const [city, setCity] = useState(null);
  const [areas, setAreas] = useState(null);
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
  }, [city]); // ✅ 已声明所有依赖项

  // ...
```

现在，第一个 Effect 只会在 `country` 变化时重新运行，而第二个 Effect 则会在 `city` 变化时重新运行。你已经按目的将它们分开：两个不同的事情由两个独立的 Effect 同步。两个独立的 Effect 也有两个独立的依赖列表，因此它们不会无意中触发彼此。

最终代码比原始代码更长，但拆分这些 Effect 仍然是正确的。[每个 Effect 都应该代表一个独立的同步过程。](/learn/lifecycle-of-reactive-effects#each-effect-represents-a-separate-synchronization-process) 在这个例子中，删除一个 Effect 并不会破坏另一个 Effect 的逻辑。这说明它们在*同步不同的东西*，因此拆分它们是好的。如果你担心重复代码，可以通过[把重复逻辑提取到自定义 Hook 中](/learn/reusing-logic-with-custom-hooks#when-to-use-custom-hooks)来改进。

### 你是否在读取某些状态来计算下一个状态？ {/*are-you-reading-some-state-to-calculate-the-next-state*/}

这个 Effect 会在每次收到新消息时，使用新创建的数组更新 `messages` 状态变量：

```js {2,6-8}
function ChatRoom({ roomId }) {
  const [messages, setMessages] = useState([]);
  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    connection.on('message', (receivedMessage) => {
      setMessages([...messages, receivedMessage]);
    });
    // ...
```

它使用 `messages` 变量[创建一个新数组](/learn/updating-arrays-in-state)，这个新数组以所有已有消息开头，并把新消息追加到末尾。然而，由于 `messages` 是一个被 Effect 读取的响应式值，它必须作为依赖项：

```js {7,10}
function ChatRoom({ roomId }) {
  const [messages, setMessages] = useState([]);
  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    connection.on('message', (receivedMessage) => {
      setMessages([...messages, receivedMessage]);
    });
    return () => connection.disconnect();
  }, [roomId, messages]); // ✅ 已声明所有依赖项
  // ...
```

而把 `messages` 作为依赖会带来一个问题。

每次你收到消息时，`setMessages()` 都会让组件重新渲染，并生成一个包含收到消息的新 `messages` 数组。然而，由于这个 Effect 现在依赖于 `messages`，这也会*重新同步*这个 Effect。所以每条新消息都会让聊天重新连接。用户不会喜欢这样的体验！

要修复这个问题，不要在 Effect 内部读取 `messages`。相反，把一个[更新函数](/reference/react/useState#updating-state-based-on-the-previous-state)传给 `setMessages`：

```js {7,10}
function ChatRoom({ roomId }) {
  const [messages, setMessages] = useState([]);
  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    connection.on('message', (receivedMessage) => {
      setMessages(msgs => [...msgs, receivedMessage]);
    });
    return () => connection.disconnect();
  }, [roomId]); // ✅ 已声明所有依赖项
  // ...
```

**注意，现在你的 Effect 根本没有读取 `messages` 变量。** 你只需要传入一个更新函数，例如 `msgs => [...msgs, receivedMessage]`。React 会把你的更新函数[放入队列](/learn/queueing-a-series-of-state-updates)，并在下一次渲染时向它提供 `msgs` 参数。这就是为什么 Effect 本身不再需要依赖 `messages`。修复之后，接收到聊天消息将不再导致聊天重新连接。

### 你是否想在不“响应”变化的情况下读取某个值？ {/*do-you-want-to-read-a-value-without-reacting-to-its-changes*/}

假设你想在用户收到新消息时播放声音，除非 `isMuted` 为 `true`：

```js {3,10-12}
function ChatRoom({ roomId }) {
  const [messages, setMessages] = useState([]);
  const [isMuted, setIsMuted] = useState(false);

  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    connection.on('message', (receivedMessage) => {
      setMessages(msgs => [...msgs, receivedMessage]);
      if (!isMuted) {
        playSound();
      }
    });
    // ...
```

由于你的 Effect 现在在代码中使用了 `isMuted`，你必须把它加入依赖项：

```js {10,15}
function ChatRoom({ roomId }) {
  const [messages, setMessages] = useState([]);
  const [isMuted, setIsMuted] = useState(false);

  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    connection.on('message', (receivedMessage) => {
      setMessages(msgs => [...msgs, receivedMessage]);
      if (!isMuted) {
        playSound();
      }
    });
    return () => connection.disconnect();
  }, [roomId, isMuted]); // ✅ 已声明所有依赖项
  // ...
```

问题在于，每当 `isMuted` 变化时（例如用户按下“静音”切换按钮），Effect 都会重新同步，并重新连接到聊天。这不是理想的用户体验！（在这个例子中，即使禁用 lint 规则也没用——如果你那样做，`isMuted` 会“卡住”在旧值上。）

要解决这个问题，你需要把不应该具有响应性的逻辑从 Effect 中提取出来。你不希望这个 Effect 对 `isMuted` 的变化“响应”。[把这部分非响应式逻辑移动到一个 Effect Event 中：](/learn/separating-events-from-effects#declaring-an-effect-event)

```js {1,7-12,18,21}
import { useState, useEffect, useEffectEvent } from 'react';

function ChatRoom({ roomId }) {
  const [messages, setMessages] = useState([]);
  const [isMuted, setIsMuted] = useState(false);

  const onMessage = useEffectEvent(receivedMessage => {
    setMessages(msgs => [...msgs, receivedMessage]);
    if (!isMuted) {
      playSound();
    }
  });

  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    connection.on('message', (receivedMessage) => {
      onMessage(receivedMessage);
    });
    return () => connection.disconnect();
  }, [roomId]); // ✅ 已声明所有依赖项
  // ...
```

Effect Event 允许你把一个 Effect 拆分为响应式部分（它们应该对 `roomId` 等响应式值及其变化作出“响应”）和非响应式部分（它们只读取最新值，例如 `onMessage` 读取 `isMuted`）。**既然你在 Effect Event 中读取了 `isMuted`，它就不需要成为 Effect 的依赖项了。** 这样，当你切换“静音”设置时，聊天就不会重新连接，从而解决了最初的问题！

#### 包装来自 props 的事件处理函数 {/*wrapping-an-event-handler-from-the-props*/}

当组件接收到一个作为 prop 的事件处理函数时，你也可能遇到类似的问题：

```js {1,8,11}
function ChatRoom({ roomId, onReceiveMessage }) {
  const [messages, setMessages] = useState([]);

  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    connection.on('message', (receivedMessage) => {
      onReceiveMessage(receivedMessage);
    });
    return () => connection.disconnect();
  }, [roomId, onReceiveMessage]); // ✅ 已声明所有依赖项
  // ...
```

假设父组件在每次渲染时都传入一个*不同的* `onReceiveMessage` 函数：

```js {3-5}
<ChatRoom
  roomId={roomId}
  onReceiveMessage={receivedMessage => {
    // ...
  }}
/>
```

由于 `onReceiveMessage` 是一个依赖项，这会导致 Effect 在父组件每次重新渲染后都重新同步。这会让它重新连接到聊天。要解决这个问题，请把调用包装到一个 Effect Event 中：

```js {4-6,12,15}
function ChatRoom({ roomId, onReceiveMessage }) {
  const [messages, setMessages] = useState([]);

  const onMessage = useEffectEvent(receivedMessage => {
    onReceiveMessage(receivedMessage);
  });

  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    connection.on('message', (receivedMessage) => {
      onMessage(receivedMessage);
    });
    return () => connection.disconnect();
  }, [roomId]); // ✅ 已声明所有依赖项
  // ...
```

Effect Event 不是响应式的，所以你不需要把它们指定为依赖项。这样，即使父组件每次重新渲染时都传入一个不同的函数，聊天也不会重新连接。

#### 分离响应式和非响应式代码 {/*separating-reactive-and-non-reactive-code*/}

在这个例子中，你希望每次 `roomId` 改变时都记录一次访问。你希望每条日志都包含当前的 `notificationCount`，但你*不*希望 `notificationCount` 的变化触发一次日志事件。

解决方案同样是把非响应式代码拆分到一个 Effect Event 中：

```js {2-4,7}
function Chat({ roomId, notificationCount }) {
  const onVisit = useEffectEvent(visitedRoomId => {
    logVisit(visitedRoomId, notificationCount);
  });

  useEffect(() => {
    onVisit(roomId);
  }, [roomId]); // ✅ 已声明所有依赖项
  // ...
}
```

你希望你的逻辑在 `roomId` 方面具有响应性，因此你在 Effect 中读取 `roomId`。然而，你不希望 `notificationCount` 的变化记录一次额外访问，因此你在 Effect Event 中读取 `notificationCount`。[了解更多关于使用 Effect Events 从 Effect 中读取最新 props 和 state 的内容。](/learn/separating-events-from-effects#reading-latest-props-and-state-with-effect-events)

### 某个响应式值是否被无意中改变了？ {/*does-some-reactive-value-change-unintentionally*/}

有时，你*确实*希望你的 Effect 对某个值“响应”，但这个值变化得比你希望的更频繁——而且可能并不反映用户视角下的实际变化。例如，假设你在组件主体中创建了一个 `options` 对象，然后在 Effect 中读取这个对象：

```js {3-6,9}
function ChatRoom({ roomId }) {
  // ...
  const options = {
    serverUrl: serverUrl,
    roomId: roomId
  };

  useEffect(() => {
    const connection = createConnection(options);
    connection.connect();
    // ...
```

这个对象是在组件主体中声明的，所以它是一个[响应式值。](/learn/lifecycle-of-reactive-effects#effects-react-to-reactive-values) 当你在 Effect 中读取这样的响应式值时，你需要把它声明为依赖项。这能确保你的 Effect 对它的变化“响应”：

```js {3,6}
  // ...
  useEffect(() => {
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [options]); // ✅ 已声明所有依赖项
  // ...
```

把它声明为依赖项很重要！例如，这能确保如果 `roomId` 变化，你的 Effect 会使用新的 `options` 重新连接到聊天。然而，上面这段代码也有一个问题。要看到这个问题，试着在下面的沙盒中输入一些内容，观察控制台里发生了什么：

<Sandpack>

```js {expectedErrors: {'react-compiler': [10]}}
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  // 临时禁用 lint 规则以演示问题
  // eslint-disable-next-line react-hooks/exhaustive-deps
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
      <h1>Welcome to the {roomId} room!</h1>
      <input value={message} onChange={e => setMessage(e.target.value)} />
    </>
  );
}

export default function App() {
  const [roomId, setRoomId] = useState('general');
  return (
    <>
      <label>
        Choose the chat room:{' '}
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
      console.log('✅ 正在连接到 "' + roomId + '" 房间，地址 ' + serverUrl + '...');
    },
    disconnect() {
      console.log('❌ 已从 "' + roomId + '" 房间，地址 ' + serverUrl + ' 断开连接');
    }
  };
}
```

```css
input { display: block; margin-bottom: 20px; }
button { margin-left: 10px; }
```

</Sandpack>

在上面的沙盒中，输入框只会更新 `message` 状态变量。从用户角度看，这不应该影响聊天连接。然而，每次你更新 `message`，组件都会重新渲染。当组件重新渲染时，内部代码会从头开始再次执行。

`ChatRoom` 组件每次重新渲染时，都会从头创建一个新的 `options` 对象。React 看到这个 `options` 对象与上一次渲染创建的 `options` 对象是*不同的对象*。这就是为什么它会重新同步你的 Effect（它依赖于 `options`），并且在你输入时聊天会重新连接。

**这个问题只影响对象和函数。在 JavaScript 中，每个新创建的对象和函数都被视为与其他对象和函数不同。即使它们内部的内容相同，也无关紧要！**

```js {7-8}
// 在第一次渲染期间
const options1 = { serverUrl: 'https://localhost:1234', roomId: 'music' };

// 在下一次渲染期间
const options2 = { serverUrl: 'https://localhost:1234', roomId: 'music' };

// 这两个是不同的对象！
console.log(Object.is(options1, options2)); // false
```

**对象和函数依赖会让你的 Effect 比你需要的更频繁地重新同步。**

这就是为什么，只要有可能，你都应该尽量避免把对象和函数作为 Effect 的依赖项。相反，尝试把它们移到组件外部、移到 Effect 内部，或者从中提取出原始值。

#### 将静态对象和函数移到组件外部 {/*move-static-objects-and-functions-outside-your-component*/}

如果对象不依赖任何 props 和 state，你可以把这个对象移到组件外部：

```js {1-4,13}
const options = {
  serverUrl: 'https://localhost:1234',
  roomId: 'music'
};

function ChatRoom() {
  const [message, setMessage] = useState('');

  useEffect(() => {
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, []); // ✅ 已声明所有依赖项
  // ...
```

这样，你就在向 lint 规则*证明*它不是响应式的。它不会因为重新渲染而改变，因此不需要成为依赖项。现在重新渲染 `ChatRoom` 不会导致你的 Effect 重新同步。

这对函数同样有效：

```js {1-6,12}
function createOptions() {
  return {
    serverUrl: 'https://localhost:1234',
    roomId: 'music'
  };
}

function ChatRoom() {
  const [message, setMessage] = useState('');

  useEffect(() => {
    const options = createOptions();
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, []); // ✅ 已声明所有依赖项
  // ...
```

由于 `createOptions` 是在组件外部声明的，所以它不是响应式值。这就是为什么它不需要出现在 Effect 的依赖项中，也不会导致 Effect 重新同步。

#### 将动态对象和函数移到 Effect 内部 {/*move-dynamic-objects-and-functions-inside-your-effect*/}

如果你的对象依赖于某个可能因重新渲染而改变的响应式值，比如 `roomId` 属性，那么你不能把它移到组件外部。不过，你可以把它的创建过程移到 Effect 代码内部：

```js {7-10,11,14}
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
  }, [roomId]); // ✅ 已声明所有依赖项
  // ...
```

既然 `options` 是在 Effect 内部声明的，它就不再是 Effect 的依赖项了。此时，Effect 使用的唯一响应式值是 `roomId`。由于 `roomId` 不是对象或函数，你可以确定它不会*无意中*变得不同。在 JavaScript 中，数字和字符串是按照内容比较的：

```js {7-8}
// 在第一次渲染期间
const roomId1 = 'music';

// 在下一次渲染期间
const roomId2 = 'music';

// 这两个字符串是相同的！
console.log(Object.is(roomId1, roomId2)); // true
```

得益于这个修复，如果你编辑输入框，聊天将不再重新连接：

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
      <h1>Welcome to the {roomId} room!</h1>
      <input value={message} onChange={e => setMessage(e.target.value)} />
    </>
  );
}

export default function App() {
  const [roomId, setRoomId] = useState('general');
  return (
    <>
      <label>
        Choose the chat room:{' '}
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
      console.log('✅ 正在连接到 "' + roomId + '" 房间，地址 ' + serverUrl + '...');
    },
    disconnect() {
      console.log('❌ 已从 "' + roomId + '" 房间，地址 ' + serverUrl + ' 断开连接');
    }
  };
}
```

```css
input { display: block; margin-bottom: 20px; }
button { margin-left: 10px; }
```

</Sandpack>

不过，当你更改 `roomId` 下拉框时，它*确实*会重新连接，这正是你所期望的。

这对函数也同样适用：

```js {7-12,14}
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
  }, [roomId]); // ✅ 已声明所有依赖项
  // ...
```

你可以自己编写函数，把 Effect 内部的各部分逻辑组合起来。只要你也把它们*定义在* Effect 内部，它们就不是响应式值，因此不需要作为 Effect 的依赖项。

#### 从对象中读取原始值 {/*read-primitive-values-from-objects*/}

有时，你可能会从 props 接收到一个对象：

```js {1,5,8}
function ChatRoom({ options }) {
  const [message, setMessage] = useState('');

  useEffect(() => {
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [options]); // ✅ 已声明所有依赖项
  // ...
```

这里的风险在于，父组件会在渲染期间创建这个对象：

```js {3-6}
<ChatRoom
  roomId={roomId}
  options={{
    serverUrl: serverUrl,
    roomId: roomId
  }}
/>
```

这会导致每次父组件重新渲染时，你的 Effect 都重新连接。要修复这个问题，请在 Effect *外部* 从对象中读取信息，并避免对象和函数依赖：

```js {4,7-8,12}
function ChatRoom({ options }) {
  const [message, setMessage] = useState('');

  const { roomId, serverUrl } = options;
  useEffect(() => {
    const connection = createConnection({
      roomId: roomId,
      serverUrl: serverUrl
    });
    connection.connect();
    return () => connection.disconnect();
  }, [roomId, serverUrl]); // ✅ 已声明所有依赖项
  // ...
```

逻辑会稍微重复一些（你在 Effect 外部从对象中读取一些值，然后在 Effect 内部用相同的值创建一个对象）。但这样能非常明确地表达你的 Effect *实际*依赖的信息。如果对象是被父组件无意中重新创建的，聊天不会重新连接。不过，如果 `options.roomId` 或 `options.serverUrl` 真的不同了，聊天就会重新连接。

#### 从函数中计算原始值 {/*calculate-primitive-values-from-functions*/}

同样的方法也适用于函数。例如，假设父组件传入了一个函数：

```js {3-8}
<ChatRoom
  roomId={roomId}
  getOptions={() => {
    return {
      serverUrl: serverUrl,
      roomId: roomId
    };
  }}
/>
```

为了避免把它变成依赖项（以及导致它在重新渲染时重新连接），请在 Effect 外部调用它。这样你就得到了不是对象的 `roomId` 和 `serverUrl` 值，并且可以在 Effect 内部读取它们：

```js {1,4}
function ChatRoom({ getOptions }) {
  const [message, setMessage] = useState('');

  const { roomId, serverUrl } = getOptions();
  useEffect(() => {
    const connection = createConnection({
      roomId: roomId,
      serverUrl: serverUrl
    });
    connection.connect();
    return () => connection.disconnect();
  }, [roomId, serverUrl]); // ✅ 已声明所有依赖项
  // ...
```

这只适用于[纯](/learn/keeping-components-pure)函数，因为它们可以安全地在渲染期间调用。如果你的函数是一个事件处理函数，但你又不希望它的变化重新同步你的 Effect，那么[改为把它包装成一个 Effect Event。](#do-you-want-to-read-a-value-without-reacting-to-its-changes)

<Recap>

- 依赖项应始终与代码保持一致。
- 当你对依赖项不满意时，需要修改的是代码。
- 抑制 lint 规则会导致非常令人困惑的 bug，你应该始终避免这样做。
- 要删除某个依赖项，你需要向 lint 规则“证明”它不是必需的。
- 如果某些代码应该响应特定交互而运行，就把它移到事件处理函数中。
- 如果 Effect 的不同部分应该因为不同原因重新运行，就把它拆成多个 Effect。
- 如果你想基于上一个状态来更新某个状态，请传入一个更新函数。
- 如果你想在不“响应”变化的情况下读取最新值，请从 Effect 中提取一个 Effect Event。
- 在 JavaScript 中，如果对象和函数是在不同时间创建的，它们会被视为不同的值。
- 尽量避免对象和函数作为依赖项。把它们移到组件外部或 Effect 内部。

</Recap>

<Challenges>

#### 修复一个会重置的定时器 {/*fix-a-resetting-interval*/}

这个 Effect 设置了一个每秒触发一次的定时器。你注意到有些奇怪的事情：它似乎会在每次触发时被销毁并重新创建。修复代码，使定时器不会被不断重新创建。

<Hint>

看起来这个 Effect 的代码依赖于 `count`。有没有办法不需要这个依赖项？应该可以通过基于前一个值更新 `count` 状态，而无需添加对该值的依赖。

</Hint>

<Sandpack>

```js
import { useState, useEffect } from 'react';

export default function Timer() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    console.log('✅ 正在创建一个定时器');
    const id = setInterval(() => {
      console.log('⏰ 定时器触发');
      setCount(count + 1);
    }, 1000);
    return () => {
      console.log('❌ 正在清除一个定时器');
      clearInterval(id);
    };
  }, [count]);

  return <h1>Counter: {count}</h1>
}
```

</Sandpack>

<Solution>

你想在 Effect 内部把 `count` 状态更新为 `count + 1`。然而，这会让你的 Effect 依赖于 `count`，而它会随着每次触发而变化，这就是为什么你的定时器会在每次触发时被重新创建。

要解决这个问题，请使用[更新函数](/reference/react/useState#updating-state-based-on-the-previous-state)，把 `setCount(count + 1)` 改写为 `setCount(c => c + 1)`：

<Sandpack>

```js
import { useState, useEffect } from 'react';

export default function Timer() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    console.log('✅ 正在创建一个定时器');
    const id = setInterval(() => {
      console.log('⏰ 定时器触发');
      setCount(c => c + 1);
    }, 1000);
    return () => {
      console.log('❌ 正在清除一个定时器');
      clearInterval(id);
    };
  }, []);

  return <h1>Counter: {count}</h1>
}
```

</Sandpack>

你不再在 Effect 内部读取 `count`，而是把一条 `c => c + 1` 指令（“把这个数字加一！”）传给 React。React 会在下一次渲染时应用它。而且由于你不再需要在 Effect 内部读取 `count` 的值，因此你可以把 Effect 的依赖项保持为空（`[]`）。这可以防止你的 Effect 在每次触发时重新创建定时器。

</Solution>

#### 修复一个会重复触发的动画 {/*fix-a-retriggering-animation*/}

在这个例子中，当你按下“Show”时，欢迎消息会淡入显示。动画需要一秒钟。当你按下“Remove”时，欢迎消息会立即消失。淡入动画的逻辑在 `animation.js` 文件中以普通 JavaScript [动画循环](https://developer.mozilla.org/en-US/docs/Web/API/window/requestAnimationFrame)的形式实现。你不需要修改这部分逻辑。你可以把它当作第三方库。你的 Effect 会为 DOM 节点创建一个 `FadeInAnimation` 实例，然后调用 `start(duration)` 或 `stop()` 来控制动画。`duration` 由滑块控制。调整滑块，看看动画如何变化。

这段代码已经能正常工作，但你想改一处。当前，当你移动控制 `duration` 状态变量的滑块时，它会重新触发动画。请修改行为，使 Effect 不会对 `duration` 变量“响应”。当你按下“Show”时，Effect 应该使用滑块当前的 `duration`。然而，移动滑块本身不应该重新触发动画。

<Hint>

Effect 内部是否有一行代码不应该是响应式的？如何把非响应式代码移出 Effect？

</Hint>

<Sandpack>

```js
import { useState, useEffect, useRef } from 'react';
import { useEffectEvent } from 'react';
import { FadeInAnimation } from './animation.js';

function Welcome({ duration }) {
  const ref = useRef(null);

  useEffect(() => {
    const animation = new FadeInAnimation(ref.current);
    animation.start(duration);
    return () => {
      animation.stop();
    };
  }, [duration]);

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
  const [duration, setDuration] = useState(1000);
  const [show, setShow] = useState(false);

  return (
    <>
      <label>
        <input
          type="range"
          min="100"
          max="3000"
          value={duration}
          onChange={e => setDuration(Number(e.target.value))}
        />
        <br />
        淡入时长：{duration} 毫秒
      </label>
      <button onClick={() => setShow(!show)}>
        {show ? 'Remove' : 'Show'}
      </button>
      <hr />
      {show && <Welcome duration={duration} />}
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
      // 立即跳到结尾
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

<Solution>

你的 Effect 需要读取 `duration` 的最新值，但你不希望它对 `duration` 的变化“响应”。你用 `duration` 来启动动画，但启动动画本身并不是响应式的。把非响应式那一行代码提取到一个 Effect Event 中，然后在 Effect 中调用这个函数。

<Sandpack>

```js
import { useState, useEffect, useRef } from 'react';
import { FadeInAnimation } from './animation.js';
import { useEffectEvent } from 'react';

function Welcome({ duration }) {
  const ref = useRef(null);

  const onAppear = useEffectEvent(animation => {
    animation.start(duration);
  });

  useEffect(() => {
    const animation = new FadeInAnimation(ref.current);
    onAppear(animation);
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
  const [duration, setDuration] = useState(1000);
  const [show, setShow] = useState(false);

  return (
    <>
      <label>
        <input
          type="range"
          min="100"
          max="3000"
          value={duration}
          onChange={e => setDuration(Number(e.target.value))}
        />
        <br />
        淡入时长：{duration} 毫秒
      </label>
      <button onClick={() => setShow(!show)}>
        {show ? 'Remove' : 'Show'}
      </button>
      <hr />
      {show && <Welcome duration={duration} />}
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
    this.onProgress(0);
    this.startTime = performance.now();
    this.frameId = requestAnimationFrame(() => this.onFrame());
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

像 `onAppear` 这样的 Effect Event 不是响应式的，所以你可以在其中读取 `duration`，而不会重新触发动画。

</Solution>

#### 修复一个会重新连接的聊天 {/*fix-a-reconnecting-chat*/}

在这个例子中，每次你按下“Toggle theme”时，聊天都会重新连接。为什么会这样？修复这个错误，使聊天只在你编辑 Server URL 或选择不同聊天室时才重新连接。

把 `chat.js` 当作外部第三方库：你可以查看它来了解 API，但不要编辑它。

<Hint>

有不止一种修复方式，但最终你要避免把对象作为依赖项。

</Hint>

<Sandpack>

```js src/App.js
import { useState } from 'react';
import ChatRoom from './ChatRoom.js';

export default function App() {
  const [isDark, setIsDark] = useState(false);
  const [roomId, setRoomId] = useState('general');
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  const options = {
    serverUrl: serverUrl,
    roomId: roomId
  };

  return (
    <div className={isDark ? 'dark' : 'light'}>
      <button onClick={() => setIsDark(!isDark)}>
        Toggle theme
      </button>
      <label>
        Server URL:{' '}
        <input
          value={serverUrl}
          onChange={e => setServerUrl(e.target.value)}
        />
      </label>
      <label>
        Choose the chat room:{' '}
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
      <ChatRoom options={options} />
    </div>
  );
}
```

```js src/ChatRoom.js active
import { useEffect } from 'react';
import { createConnection } from './chat.js';

export default function ChatRoom({ options }) {
  useEffect(() => {
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [options]);

  return <h1>Welcome to the {options.roomId} room!</h1>;
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
  return {
    connect() {
      console.log('✅ 正在连接到 "' + roomId + '" 房间，地址 ' + serverUrl + '...');
    },
    disconnect() {
      console.log('❌ 已从 "' + roomId + '" 房间，地址 ' + serverUrl + ' 断开连接');
    }
  };
}
```

```css
label, button { display: block; margin-bottom: 5px; }
.dark { background: #222; color: #eee; }
```

</Sandpack>

<Solution>

你的 Effect 之所以重新运行，是因为它依赖于 `options` 对象。对象可能会被无意中重新创建，因此在可能的情况下，你应该尽量避免把它们作为 Effect 的依赖项。

最小改动的修复方式是，在 Effect 外部读取 `roomId` 和 `serverUrl`，然后让 Effect 依赖这些原始值（它们不会无意中改变）。在 Effect 内部创建一个对象，并把它传给 `createConnection`：

<Sandpack>

```js src/App.js
import { useState } from 'react';
import ChatRoom from './ChatRoom.js';

export default function App() {
  const [isDark, setIsDark] = useState(false);
  const [roomId, setRoomId] = useState('general');
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  const options = {
    serverUrl: serverUrl,
    roomId: roomId
  };

  return (
    <div className={isDark ? 'dark' : 'light'}>
      <button onClick={() => setIsDark(!isDark)}>
        Toggle theme
      </button>
      <label>
        Server URL:{' '}
        <input
          value={serverUrl}
          onChange={e => setServerUrl(e.target.value)}
        />
      </label>
      <label>
        Choose the chat room:{' '}
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
      <ChatRoom options={options} />
    </div>
  );
}
```

```js src/ChatRoom.js active
import { useEffect } from 'react';
import { createConnection } from './chat.js';

export default function ChatRoom({ options }) {
  const { roomId, serverUrl } = options;
  useEffect(() => {
    const connection = createConnection({
      roomId: roomId,
      serverUrl: serverUrl
    });
    connection.connect();
    return () => connection.disconnect();
  }, [roomId, serverUrl]);

  return <h1>Welcome to the {options.roomId} room!</h1>;
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
  return {
    connect() {
      console.log('✅ 正在连接到 "' + roomId + '" 房间，地址 ' + serverUrl + '...');
    },
    disconnect() {
      console.log('❌ 已从 "' + roomId + '" 房间，地址 ' + serverUrl + ' 断开连接');
    }
  };
}
```

```css
label, button { display: block; margin-bottom: 5px; }
.dark { background: #222; color: #eee; }
```

</Sandpack>

如果把 `options` 这个对象 prop 替换成更具体的 `roomId` 和 `serverUrl` props，会更好：

<Sandpack>

```js src/App.js
import { useState } from 'react';
import ChatRoom from './ChatRoom.js';

export default function App() {
  const [isDark, setIsDark] = useState(false);
  const [roomId, setRoomId] = useState('general');
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  return (
    <div className={isDark ? 'dark' : 'light'}>
      <button onClick={() => setIsDark(!isDark)}>
        Toggle theme
      </button>
      <label>
        Server URL:{' '}
        <input
          value={serverUrl}
          onChange={e => setServerUrl(e.target.value)}
        />
      </label>
      <label>
        Choose the chat room:{' '}
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
        serverUrl={serverUrl}
      />
    </div>
  );
}
```

```js src/ChatRoom.js active
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

export default function ChatRoom({ roomId, serverUrl }) {
  useEffect(() => {
    const connection = createConnection({
      roomId: roomId,
      serverUrl: serverUrl
    });
    connection.connect();
    return () => connection.disconnect();
  }, [roomId, serverUrl]);

  return <h1>Welcome to the {roomId} room!</h1>;
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
  return {
    connect() {
      console.log('✅ 正在连接到 "' + roomId + '" 房间，地址 ' + serverUrl + '...');
    },
    disconnect() {
      console.log('❌ 已从 "' + roomId + '" 房间，地址 ' + serverUrl + ' 断开连接');
    }
  };
}
```

```css
label, button { display: block; margin-bottom: 5px; }
.dark { background: #222; color: #eee; }
```

</Sandpack>

尽量使用原始类型的 props，会让你以后更容易优化组件。

</Solution>

#### 再次修复一个会重新连接的聊天 {/*fix-a-reconnecting-chat-again*/}

这个例子会以启用或不启用加密的方式连接到聊天。切换复选框，注意当加密开启和关闭时控制台中的消息不同。尝试切换房间。然后，尝试切换主题。当你连接到某个聊天室时，每隔几秒会收到新消息。确认它们的颜色与你选择的主题一致。

在这个例子中，每次你尝试更改主题时，聊天都会重新连接。修复这个问题。修复后，改变主题不应重新连接聊天，但切换加密设置或更改房间时应重新连接。

不要修改 `chat.js` 中的任何代码。除此之外，你可以修改任何代码，只要最终行为一致即可。例如，你可能会发现更改向下传递的 props 很有帮助。

<Hint>

你传下去了两个函数：`onMessage` 和 `createConnection`。它们在 `App` 每次重新渲染时都会重新创建。每次都会被视为新值，这就是为什么它们会重新触发 Effect。

其中一个函数是事件处理函数。你知道有什么办法可以在 Effect 中调用事件处理函数，而又不“响应”这个事件处理函数的新值吗？这会很有用！

另一个函数只是为了把某些状态传给一个导入的 API 方法。这个函数真的有必要吗？传递下来的核心信息是什么？你可能需要把一些导入从 `App.js` 移到 `ChatRoom.js`。

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

```js src/App.js
import { useState } from 'react';
import ChatRoom from './ChatRoom.js';
import {
  createEncryptedConnection,
  createUnencryptedConnection,
} from './chat.js';
import { showNotification } from './notifications.js';

export default function App() {
  const [isDark, setIsDark] = useState(false);
  const [roomId, setRoomId] = useState('general');
  const [isEncrypted, setIsEncrypted] = useState(false);

  return (
    <>
      <label>
        <input
          type="checkbox"
          checked={isDark}
          onChange={e => setIsDark(e.target.checked)}
        />
        使用深色主题
      </label>
      <label>
        <input
          type="checkbox"
          checked={isEncrypted}
          onChange={e => setIsEncrypted(e.target.checked)}
        />
        启用加密
      </label>
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
        onMessage={msg => {
          showNotification('新消息：' + msg, isDark ? 'dark' : 'light');
        }}
        createConnection={() => {
          const options = {
            serverUrl: 'https://localhost:1234',
            roomId: roomId
          };
          if (isEncrypted) {
            return createEncryptedConnection(options);
          } else {
            return createUnencryptedConnection(options);
          }
        }}
      />
    </>
  );
}
```

```js src/ChatRoom.js active
import { useState, useEffect } from 'react';
import { useEffectEvent } from 'react';

export default function ChatRoom({ roomId, createConnection, onMessage }) {
  useEffect(() => {
    const connection = createConnection();
    connection.on('message', (msg) => onMessage(msg));
    connection.connect();
    return () => connection.disconnect();
  }, [createConnection, onMessage]);

  return <h1>欢迎来到 {roomId} 房间！</h1>;
}
```

```js src/chat.js
export function createEncryptedConnection({ serverUrl, roomId }) {
  // 一个真实的实现会实际连接到服务器
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
      console.log('✅ 🔐 Connecting to "' + roomId + '" room... (encrypted)');
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
      console.log('❌ 🔐 Disconnected from "' + roomId + '" room (encrypted)');
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

export function createUnencryptedConnection({ serverUrl, roomId }) {
  // 一个真实的实现会实际连接到服务器
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
      console.log('✅ Connecting to "' + roomId + '" room (unencrypted)...');
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
      console.log('❌ Disconnected from "' + roomId + '" room (unencrypted)');
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
label, button { display: block; margin-bottom: 5px; }
```

</Sandpack>

<Solution>

解决这个问题有不止一种正确方法，但这里提供一种可能的方案。

在原始示例中，切换主题会导致创建并传递不同的 `onMessage` 和 `createConnection` 函数。由于 Effect 依赖这些函数，每次切换主题时聊天室都会重新连接。

要修复 `onMessage` 的问题，你需要把它包装进一个 Effect Event：

```js {1,2,6}
export default function ChatRoom({ roomId, createConnection, onMessage }) {
  const onReceiveMessage = useEffectEvent(onMessage);

  useEffect(() => {
    const connection = createConnection();
    connection.on('message', (msg) => onReceiveMessage(msg));
    // ...
```

与 `onMessage` 属性不同，`onReceiveMessage` 这个 Effect Event 不是响应式的。这就是为什么它不需要作为 Effect 的依赖项。因此，`onMessage` 的变化不会导致聊天室重新连接。

你不能对 `createConnection` 做同样的处理，因为它 *应该* 是响应式的。你 *希望* 当用户在加密连接和未加密连接之间切换，或者切换当前房间时，Effect 重新触发。然而，由于 `createConnection` 是一个函数，你无法判断它读取的信息是否 *真的* 发生了变化。为了解决这个问题，不要从 `App` 组件向下传递 `createConnection`，而是传递原始的 `roomId` 和 `isEncrypted` 值：

```js {2-3}
      <ChatRoom
        roomId={roomId}
        isEncrypted={isEncrypted}
        onMessage={msg => {
          showNotification('新消息：' + msg, isDark ? 'dark' : 'light');
        }}
      />
```

现在你可以把 `createConnection` 函数 *放到* Effect 内部，而不是从 `App` 中传递下来：

```js {1-4,6,10-20}
import {
  createEncryptedConnection,
  createUnencryptedConnection,
} from './chat.js';

export default function ChatRoom({ roomId, isEncrypted, onMessage }) {
  const onReceiveMessage = useEffectEvent(onMessage);

  useEffect(() => {
    function createConnection() {
      const options = {
        serverUrl: 'https://localhost:1234',
        roomId: roomId
      };
      if (isEncrypted) {
        return createEncryptedConnection(options);
      } else {
        return createUnencryptedConnection(options);
      }
    }
    // ...
```

完成这两处修改后，你的 Effect 就不再依赖任何函数值了：

```js {1,8,10,21}
export default function ChatRoom({ roomId, isEncrypted, onMessage }) { // 响应式值
  const onReceiveMessage = useEffectEvent(onMessage); // 非响应式

  useEffect(() => {
    function createConnection() {
      const options = {
        serverUrl: 'https://localhost:1234',
        roomId: roomId // 读取一个响应式值
      };
      if (isEncrypted) { // 读取一个响应式值
        return createEncryptedConnection(options);
      } else {
        return createUnencryptedConnection(options);
      }
    }

    const connection = createConnection();
    connection.on('message', (msg) => onReceiveMessage(msg));
    connection.connect();
    return () => connection.disconnect();
  }, [roomId, isEncrypted]); // ✅ 已声明所有依赖项
```

因此，聊天室只会在某些有意义的内容（`roomId` 或 `isEncrypted`）发生变化时重新连接：

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

```js src/App.js
import { useState } from 'react';
import ChatRoom from './ChatRoom.js';

import { showNotification } from './notifications.js';

export default function App() {
  const [isDark, setIsDark] = useState(false);
  const [roomId, setRoomId] = useState('general');
  const [isEncrypted, setIsEncrypted] = useState(false);

  return (
    <>
      <label>
        <input
          type="checkbox"
          checked={isDark}
          onChange={e => setIsDark(e.target.checked)}
        />
        使用深色主题
      </label>
      <label>
        <input
          type="checkbox"
          checked={isEncrypted}
          onChange={e => setIsEncrypted(e.target.checked)}
        />
        启用加密
      </label>
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
        isEncrypted={isEncrypted}
        onMessage={msg => {
          showNotification('新消息：' + msg, isDark ? 'dark' : 'light');
        }}
      />
    </>
  );
}
```

```js src/ChatRoom.js active
import { useState, useEffect } from 'react';
import { useEffectEvent } from 'react';
import {
  createEncryptedConnection,
  createUnencryptedConnection,
} from './chat.js';

export default function ChatRoom({ roomId, isEncrypted, onMessage }) {
  const onReceiveMessage = useEffectEvent(onMessage);

  useEffect(() => {
    function createConnection() {
      const options = {
        serverUrl: 'https://localhost:1234',
        roomId: roomId
      };
      if (isEncrypted) {
        return createEncryptedConnection(options);
      } else {
        return createUnencryptedConnection(options);
      }
    }

    const connection = createConnection();
    connection.on('message', (msg) => onReceiveMessage(msg));
    connection.connect();
    return () => connection.disconnect();
  }, [roomId, isEncrypted]);

  return <h1>欢迎来到 {roomId} 房间！</h1>;
}
```

```js src/chat.js
export function createEncryptedConnection({ serverUrl, roomId }) {
  // 一个真实的实现会实际连接到服务器
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
      console.log('✅ 🔐 Connecting to "' + roomId + '" room... (encrypted)');
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
      console.log('❌ 🔐 Disconnected from "' + roomId + '" room (encrypted)');
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

export function createUnencryptedConnection({ serverUrl, roomId }) {
  // 一个真实的实现会实际连接到服务器
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
      console.log('✅ Connecting to "' + roomId + '" room (unencrypted)...');
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
      console.log('❌ Disconnected from "' + roomId + '" room (unencrypted)');
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
label, button { display: block; margin-bottom: 5px; }
```

</Sandpack>

</Solution>

</Challenges>
