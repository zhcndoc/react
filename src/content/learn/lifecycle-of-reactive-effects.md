---
title: '响应式 Effect 的生命周期'
---

<Intro>

Effect 与组件有不同的生命周期。组件可能会挂载、更新或卸载。Effect 只能做两件事：开始同步某些内容，然后在之后停止同步。 如果你的 Effect 依赖会随时间变化的 props 和 state，这个循环可能会发生多次。React 提供了一条 linter 规则来检查你是否正确指定了 Effect 的依赖项。这可以让你的 Effect 与最新的 props 和 state 保持同步。

</Intro>

<YouWillLearn>

- Effect 的生命周期与组件的生命周期有何不同
- 如何独立地看待每一个 Effect
- 你的 Effect 何时需要重新同步，以及原因是什么
- 你的 Effect 的依赖项是如何确定的
- 值具有响应性是什么意思
- 空依赖数组意味着什么
- React 如何通过 linter 验证你的依赖项是否正确
- 当你不同意 linter 的判断时该怎么做

</YouWillLearn>

## Effect 的生命周期 {/*the-lifecycle-of-an-effect*/}

每个 React 组件都会经历相同的生命周期：

- 组件在被添加到屏幕上时会 _挂载_。
- 组件在接收到新的 props 或 state 时会 _更新_，通常是为了响应某个交互。
- 组件在从屏幕上移除时会 _卸载_。

**把这用于理解组件是很好的，但不适用于理解 Effect。** 相反，尝试把每个 Effect 看作是独立于组件生命周期的。Effect 描述的是如何将[外部系统同步](/learn/synchronizing-with-effects)到当前的 props 和 state。随着代码变化，同步需要更频繁地发生，或者更少地发生。

为了说明这一点，考虑这个将组件连接到聊天服务器的 Effect：

```js
const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
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

你的 Effect 主体指定了如何 **开始同步：**

```js {2-3}
    // ...
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
    // ...
```

由你的 Effect 返回的清理函数指定了如何 **停止同步：**

```js {5}
    // ...
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
    // ...
```

直觉上，你可能会认为 React 会在组件挂载时 **开始同步**，并在组件卸载时 **停止同步**。然而，事情并没有这么简单！有时，即使组件仍然处于挂载状态，也可能需要 **多次开始和停止同步**。

让我们看看这是 _为什么_ 需要的，_何时_ 会发生，以及 _如何_ 控制这种行为。

<Note>

有些 Effect 根本不会返回清理函数。[大多数情况下，](/learn/synchronizing-with-effects#how-to-handle-the-effect-firing-twice-in-development) 你都会希望返回一个清理函数——但如果你没有返回，React 的行为就会像你返回了一个空的清理函数一样。

</Note>

### 为什么同步可能需要发生不止一次 {/*why-synchronization-may-need-to-happen-more-than-once*/}

想象一下，这个 `ChatRoom` 组件接收了一个 `roomId` prop，而用户通过下拉框来选择它。假设一开始用户选择的是 `"general"` 房间作为 `roomId`。你的应用会显示 `"general"` 聊天室：

```js {3}
const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId /* "general" */ }) {
  // ...
  return <h1>Welcome to the {roomId} room!</h1>;
}
```

在 UI 显示出来之后，React 会运行你的 Effect 来 **开始同步。** 它会连接到 `"general"` 房间：

```js {3,4}
function ChatRoom({ roomId /* "general" */ }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId); // 连接到 "general" 房间
    connection.connect();
    return () => {
      connection.disconnect(); // 从 "general" 房间断开连接
    };
  }, [roomId]);
  // ...
```

到目前为止，一切都很好。

之后，用户在下拉框里选择了另一个房间（例如 `"travel"`）。首先，React 会更新 UI：

```js {1}
function ChatRoom({ roomId /* "travel" */ }) {
  // ...
  return <h1>Welcome to the {roomId} room!</h1>;
}
```

想一想接下来应该发生什么。用户在 UI 中看到 `"travel"` 是当前选中的聊天室。然而，上一次运行的 Effect 仍然连接在 `"general"` 房间。**`roomId` prop 已经变了，所以你的 Effect 之前所做的事情（连接到 `"general"` 房间）已经不再与 UI 匹配了。**

此时，你希望 React 做两件事：

1. 停止与旧的 `roomId` 同步（从 `"general"` 房间断开连接）
2. 开始与新的 `roomId` 同步（连接到 `"travel"` 房间）

**幸运的是，你已经教会 React 如何完成这两件事！** 你的 Effect 主体指定了如何开始同步，而清理函数指定了如何停止同步。React 现在需要做的就是以正确的顺序、使用正确的 props 和 state 来调用它们。让我们看看这具体是怎么发生的。

### React 如何重新同步你的 Effect {/*how-react-re-synchronizes-your-effect*/}

回想一下，你的 `ChatRoom` 组件收到了 `roomId` prop 的新值。它之前是 `"general"`，现在是 `"travel"`。React 需要重新同步你的 Effect，把你重新连接到另一个房间。

为了 **停止同步，** React 会调用你的 Effect 在连接到 `"general"` 房间后返回的清理函数。由于 `roomId` 是 `"general"`，清理函数会断开与 `"general"` 房间的连接：

```js {6}
function ChatRoom({ roomId /* "general" */ }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId); // 连接到 "general" 房间
    connection.connect();
    return () => {
      connection.disconnect(); // 从 "general" 房间断开连接
    };
    // ...
```

然后 React 会运行你在这次渲染中提供的 Effect。这一次，`roomId` 是 `"travel"`，所以它会 **开始同步** 到 `"travel"` 聊天室（直到它的清理函数最终也被调用）：

```js {3,4}
function ChatRoom({ roomId /* "travel" */ }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId); // 连接到 "travel" 房间
    connection.connect();
    // ...
```

得益于此，你现在已经连接到了用户在 UI 中选择的同一个房间。避免了一场灾难！

每次你的组件因为不同的 `roomId` 重新渲染后，你的 Effect 都会重新同步。例如，假设用户将 `roomId` 从 `"travel"` 改为 `"music"`。React 会再次通过调用清理函数来 **停止同步** 你的 Effect（把你从 `"travel"` 房间断开）。然后它会再次运行主体，并使用新的 `roomId` prop 来 **开始同步**（连接到 `"music"` 房间）。

最后，当用户切换到另一个界面时，`ChatRoom` 会卸载。此时已经完全没有必要继续保持连接。React 会最后一次 **停止同步** 你的 Effect，并让你从 `"music"` 聊天室断开连接。

### 从 Effect 的视角思考 {/*thinking-from-the-effects-perspective*/}

让我们回顾一下从 `ChatRoom` 组件的视角发生了什么：

1. `ChatRoom` 挂载，`roomId` 设为 `"general"`
1. `ChatRoom` 更新，`roomId` 设为 `"travel"`
1. `ChatRoom` 更新，`roomId` 设为 `"music"`
1. `ChatRoom` 卸载

在组件生命周期中的这些时刻里，你的 Effect 做了不同的事情：

1. 你的 Effect 连接到了 `"general"` 房间
1. 你的 Effect 从 `"general"` 房间断开，并连接到 `"travel"` 房间
1. 你的 Effect 从 `"travel"` 房间断开，并连接到 `"music"` 房间
1. 你的 Effect 从 `"music"` 房间断开

现在让我们从 Effect 本身的角度来思考发生了什么：

```js
  useEffect(() => {
    // 你的 Effect 连接到了由 roomId 指定的房间...
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      // ...直到它断开连接
      connection.disconnect();
    };
  }, [roomId]);
```

这段代码的结构也许会让你把发生的事情理解为一系列互不重叠的时间段：

1. 你的 Effect 连接到了 `"general"` 房间（直到它断开）
1. 你的 Effect 连接到了 `"travel"` 房间（直到它断开）
1. 你的 Effect 连接到了 `"music"` 房间（直到它断开）

之前，你是在从组件的角度思考。当你从组件的角度看时，很容易把 Effect 想成在某个特定时间触发的“回调”或“生命周期事件”，比如“在渲染之后”或“在卸载之前”。这种思考方式很快就会变得复杂，所以最好避免。

**相反，始终一次只关注一个开始/停止循环。组件是在挂载、更新还是卸载并不重要。你只需要描述如何开始同步，以及如何停止同步。如果你做得好，你的 Effect 就能在需要的任意多次启动和停止中保持健壮。**

这可能会让你联想到：在编写创建 JSX 的渲染逻辑时，你并不会去考虑组件是在挂载还是更新。你只需要描述屏幕上应该显示什么，剩下的由 React [来处理。](/learn/reacting-to-input-with-state)

### React 如何验证你的 Effect 可以重新同步 {/*how-react-verifies-that-your-effect-can-re-synchronize*/}

下面是一个可以交互的实时示例。点击 "Open chat" 来挂载 `ChatRoom` 组件：

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
export function createConnection(serverUrl, roomId) {
  // 真实实现会实际连接到服务器
  return {
    connect() {
      console.log('✅ 正在连接到 "' + roomId + '" 房间，服务器地址为 ' + serverUrl + '...');
    },
    disconnect() {
      console.log('❌ 已从 "' + roomId + '" 房间断开连接，服务器地址为 ' + serverUrl);
    }
  };
}
```

```css
input { display: block; margin-bottom: 20px; }
button { margin-left: 10px; }
```

</Sandpack>

请注意，当组件第一次挂载时，你会看到三条日志：

1. `✅ 正在连接到 "general" 房间，服务器地址为 https://localhost:1234...` *(仅开发环境)*
1. `❌ 已从 "general" 房间断开连接，服务器地址为 https://localhost:1234.` *(仅开发环境)*
1. `✅ 正在连接到 "general" 房间，服务器地址为 https://localhost:1234...`

前两条日志仅在开发环境中出现。在开发环境中，React 总是会让每个组件重新挂载一次。

**React 会在开发环境中强制立即执行一次，以此验证你的 Effect 是否能够重新同步。** 这有点像为了检查门锁是否正常工作而额外开一次门、关一次门。React 在开发环境中会额外启动和停止一次你的 Effect，以检查[你是否正确实现了清理逻辑。](/learn/synchronizing-with-effects#how-to-handle-the-effect-firing-twice-in-development)

你的 Effect 在实际使用中需要重新同步的主要原因，是它所使用的某些数据发生了变化。在上面的沙盒中，切换所选聊天室。注意当 `roomId` 变化时，你的 Effect 会重新同步。

不过，在一些更少见的情况下，也需要重新同步。例如，试着在聊天打开时编辑上面沙盒中的 `serverUrl`。注意 Effect 会在你对代码的编辑响应中重新同步。未来，React 可能会加入更多依赖重新同步的功能。

### React 如何知道它需要重新同步 Effect {/*how-react-knows-that-it-needs-to-re-synchronize-the-effect*/}

你可能会好奇，React 是怎么知道 `roomId` 变化后你的 Effect 需要重新同步的。这是因为你通过将它包含在[依赖列表中](/learn/synchronizing-with-effects#step-2-specify-the-effect-dependencies)，**告诉了 React** 它的代码依赖于 `roomId`：

```js {1,3,8}
function ChatRoom({ roomId }) { // roomId prop 可能会随时间变化
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId); // 这个 Effect 读取了 roomId
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [roomId]); // 因此你告诉 React，这个 Effect “依赖于” roomId
  // ...
```

它的工作方式如下：

1. 你知道 `roomId` 是一个 prop，这意味着它可能会随时间变化。
2. 你知道你的 Effect 读取了 `roomId`（因此它的逻辑依赖于一个以后可能变化的值）。
3. 这就是你将它指定为 Effect 依赖项的原因（这样当 `roomId` 变化时，它就会重新同步）。

每次组件重新渲染后，React 都会查看你传入的依赖数组。如果数组中的任意值与上一次渲染时对应位置传入的值不同，React 就会重新同步你的 Effect。

例如，如果你在初始渲染时传入了 `["general"]`，而在下一次渲染时传入了 `["travel"]`，React 就会比较 `"general"` 和 `"travel"`。这些是不同的值（使用 [`Object.is`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is) 比较），所以 React 会重新同步你的 Effect。另一方面，如果组件重新渲染，但 `roomId` 没有变化，你的 Effect 就会继续连接到同一个房间。

### 每个 Effect 都代表一个独立的同步过程 {/*each-effect-represents-a-separate-synchronization-process*/}

不要仅仅因为某段逻辑需要和你已经写好的 Effect 同时运行，就把无关的逻辑加到这个 Effect 里。例如，假设你希望在用户访问房间时发送一条分析事件。你已经有一个依赖 `roomId` 的 Effect，所以你可能会忍不住把分析调用也加进去：

```js {3}
function ChatRoom({ roomId }) {
  useEffect(() => {
    logVisit(roomId);
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [roomId]);
  // ...
}
```

但想象一下，你后来给这个 Effect 增加了另一个需要重新建立连接的依赖项。如果这个 Effect 重新同步了，它也会对同一个房间调用 `logVisit(roomId)`，而这并不是你的本意。记录访问日志 **是一个独立的过程**，和连接是分开的。应当把它们写成两个单独的 Effect：

```js {2-4}
function ChatRoom({ roomId }) {
  useEffect(() => {
    logVisit(roomId);
  }, [roomId]);

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    // ...
  }, [roomId]);
  // ...
}
```

**你代码中的每个 Effect 都应该代表一个独立的同步过程。**

在上面的例子里，删除一个 Effect 并不会破坏另一个 Effect 的逻辑。这说明它们同步的是不同的事情，因此把它们拆开是合理的。另一方面，如果你把一段内聚的逻辑拆成多个 Effect，代码看起来可能更“整洁”，但会[更难维护。](/learn/you-might-not-need-an-effect#chains-of-computations) 这就是为什么你应该思考这些过程是相同还是不同，而不是代码看起来是否更整洁。

## Effects “响应” reactive 值 {/*effects-react-to-reactive-values*/}

你的 Effect 读取了两个变量（`serverUrl` 和 `roomId`），但你只把 `roomId` 指定为了依赖项：

```js {5,10}
const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
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

为什么 `serverUrl` 不需要成为依赖项？

这是因为 `serverUrl` 不会因为重新渲染而改变。无论组件重新渲染多少次、因为什么原因，它始终都是同一个值。既然 `serverUrl` 不会改变，把它列为依赖项就没有意义。毕竟，依赖项只有在随时间发生变化时才有作用！

另一方面，`roomId` 在重新渲染时可能不同。**Props、state，以及组件内部声明的其他值都是 _reactive_（响应式）的，因为它们是在渲染期间计算出来的，并且参与 React 的数据流。**

如果 `serverUrl` 是一个 state 变量，那它就是 reactive 的。reactive 值必须包含在依赖项中：

```js {2,5,10}
function ChatRoom({ roomId }) { // Props 会随时间变化
  const [serverUrl, setServerUrl] = useState('https://localhost:1234'); // State 可能会随时间变化

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId); // 你的 Effect 读取了 props 和 state
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [roomId, serverUrl]); // 所以你要告诉 React，这个 Effect “依赖于” props 和 state
  // ...
}
```

把 `serverUrl` 加入依赖项后，你就能确保它变化之后，Effect 会重新同步。

试着在这个 sandbox 中更改选中的聊天房间，或编辑服务器 URL：

<Sandpack>

```js
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
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
      <h1>Welcome to the {roomId} room!</h1>
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
export function createConnection(serverUrl, roomId) {
  // 一个真实实现会真正连接到服务器
  return {
    connect() {
      console.log('✅ 正在连接到 "' + roomId + '" 房间，地址为 ' + serverUrl + '...');
    },
    disconnect() {
      console.log('❌ 已从 "' + roomId + '" 房间断开连接，地址为 ' + serverUrl);
    }
  };
}
```

```css
input { display: block; margin-bottom: 20px; }
button { margin-left: 10px; }
```

</Sandpack>

每当你更改像 `roomId` 或 `serverUrl` 这样的 reactive 值时，Effect 都会重新连接到聊天服务器。

### 空依赖项的 Effect 意味着什么 {/*what-an-effect-with-empty-dependencies-means*/}

如果你把 `serverUrl` 和 `roomId` 都移到组件外面，会发生什么？

```js {1,2}
const serverUrl = 'https://localhost:1234';
const roomId = 'general';

function ChatRoom() {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, []); // ✅ 已声明所有依赖项
  // ...
}
```

现在你的 Effect 代码不使用任何 reactive 值，因此它的依赖项可以为空（`[]`）。

从组件的角度看，空的 `[]` 依赖数组意味着：这个 Effect 只会在组件挂载时连接到聊天室，并且只会在组件卸载时断开连接。（请记住，在开发环境中，React 仍然会[额外重新同步一次](#how-react-verifies-that-your-effect-can-re-synchronize)来对你的逻辑进行压力测试。）


<Sandpack>

```js
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

const serverUrl = 'https://localhost:1234';
const roomId = 'general';

function ChatRoom() {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, []);
  return <h1>Welcome to the {roomId} room!</h1>;
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
  // 一个真实实现会真正连接到服务器
  return {
    connect() {
      console.log('✅ 正在连接到 "' + roomId + '" 房间，地址为 ' + serverUrl + '...');
    },
    disconnect() {
      console.log('❌ 已从 "' + roomId + '" 房间断开连接，地址为 ' + serverUrl);
    }
  };
}
```

```css
input { display: block; margin-bottom: 20px; }
button { margin-left: 10px; }
```

</Sandpack>

然而，如果你[从 Effect 的角度思考，](#thinking-from-the-effects-perspective)你根本不需要考虑挂载和卸载。重要的是，你已经明确描述了 Effect 如何开始和停止同步。到目前为止，它没有任何 reactive 依赖项。但是如果你以后希望用户随着时间更改 `roomId` 或 `serverUrl`（它们就会变成 reactive 的），你的 Effect 代码也不需要改变。你只需要把它们加入依赖项。

### 组件函数体内声明的所有变量都是 reactive 的 {/*all-variables-declared-in-the-component-body-are-reactive*/}

Props 和 state 不是唯一的 reactive 值。你从它们计算出来的值也是 reactive 的。如果 props 或 state 改变了，组件会重新渲染，从它们计算出来的值也会改变。这就是为什么 Effect 使用到的组件函数体内所有变量都应该出现在 Effect 依赖列表中。

假设用户可以在下拉框中选择一个聊天服务器，但他们也可以在设置里配置默认服务器。假设你已经把设置 state 放在了一个 [context](/learn/scaling-up-with-reducer-and-context) 中，因此你可以从这个 context 中读取 `settings`。现在你根据 props 中选中的服务器和默认服务器来计算 `serverUrl`：

```js {3,5,10}
function ChatRoom({ roomId, selectedServerUrl }) { // roomId 是 reactive 的
  const settings = useContext(SettingsContext); // settings 是 reactive 的
  const serverUrl = selectedServerUrl ?? settings.defaultServerUrl; // serverUrl 是 reactive 的
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId); // 你的 Effect 读取了 roomId 和 serverUrl
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [roomId, serverUrl]); // 所以当它们任意一个变化时，都需要重新同步！
  // ...
}
```

在这个例子中，`serverUrl` 既不是 prop，也不是 state 变量。它只是你在渲染期间计算出来的普通变量。但它是在渲染期间计算的，所以它可能因为重新渲染而改变。这就是它为什么是 reactive 的。

**组件内部的所有值（包括 props、state，以及组件函数体内的变量）都是 reactive 的。任何 reactive 值都可能在重新渲染时改变，所以你需要把 reactive 值作为 Effect 的依赖项。**

换句话说，Effects 会对组件函数体中的所有值“响应”。

<DeepDive>

#### 全局值或可变值可以作为依赖项吗？ {/*can-global-or-mutable-values-be-dependencies*/}

可变值（包括全局变量）不是 reactive 的。

**像 [`location.pathname`](https://developer.mozilla.org/en-US/docs/Web/API/Location/pathname) 这样的可变值不能作为依赖项。** 它是可变的，因此它可以在完全脱离 React 渲染数据流的任何时候发生变化。更改它不会触发组件重新渲染。因此，即使你把它写进依赖项里，React 也*不会知道*在它变化时需要重新同步 Effect。这也违反了 React 的规则，因为在渲染期间读取可变数据（而这正是你计算依赖项的时候）会破坏[渲染纯度。](/learn/keeping-components-pure)相反，你应该使用 [`useSyncExternalStore`.](/learn/you-might-not-need-an-effect#subscribing-to-an-external-store) 来读取并订阅外部可变值。

**像 [`ref.current`](/reference/react/useRef#reference) 这样的可变值，或者你从它读取到的内容，也不能作为依赖项。** `useRef` 返回的 ref 对象本身可以作为依赖项，但它的 `current` 属性是刻意设计成可变的。它可以让你[在不触发重新渲染的情况下追踪某些值。](/learn/referencing-values-with-refs)但是由于修改它不会触发重新渲染，所以它不是 reactive 值，React 也不会在它变化时知道要重新运行你的 Effect。

正如下文所学，linter 会自动检查这些问题。

</DeepDive>

### React 会验证你是否将每个 reactive 值都指定为依赖项 {/*react-verifies-that-you-specified-every-reactive-value-as-a-dependency*/}

如果你的 linter 已为 [React 配置，](/learn/editor-setup#linting)它会检查你的 Effect 代码中使用到的每个 reactive 值是否都被声明为依赖项。比如，下面这段代码会报 lint 错误，因为 `roomId` 和 `serverUrl` 都是 reactive 的：

<Sandpack>

```js
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

function ChatRoom({ roomId }) { // roomId 是 reactive 的
  const [serverUrl, setServerUrl] = useState('https://localhost:1234'); // serverUrl 是 reactive 的

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, []); // <-- 这里有问题！

  return (
    <>
      <label>
        Server URL:{' '}
        <input
          value={serverUrl}
          onChange={e => setServerUrl(e.target.value)}
        />
      </label>
      <h1>Welcome to the {roomId} room!</h1>
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
export function createConnection(serverUrl, roomId) {
  // 一个真实实现会真正连接到服务器
  return {
    connect() {
      console.log('✅ 正在连接到 "' + roomId + '" 房间，地址为 ' + serverUrl + '...');
    },
    disconnect() {
      console.log('❌ 已从 "' + roomId + '" 房间断开连接，地址为 ' + serverUrl);
    }
  };
}
```

```css
input { display: block; margin-bottom: 20px; }
button { margin-left: 10px; }
```

</Sandpack>

这看起来像是 React 报错了，但实际上 React 只是指出了你代码中的一个 bug。`roomId` 和 `serverUrl` 都可能随时间变化，但你忘了在它们变化时重新同步你的 Effect。即使用户在界面中选择了不同的值，你仍然会连接到初始的 `roomId` 和 `serverUrl`。

要修复这个 bug，请按照 linter 的建议，把 `roomId` 和 `serverUrl` 指定为 Effect 的依赖项：

```js {9}
function ChatRoom({ roomId }) { // roomId 是 reactive 的
  const [serverUrl, setServerUrl] = useState('https://localhost:1234'); // serverUrl 是 reactive 的
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [serverUrl, roomId]); // ✅ 已声明所有依赖项
  // ...
}
```

试着在上面的 sandbox 中做这个修复。确认 linter 错误消失，并且聊天会在需要时重新连接。

<Note>

在某些情况下，即使一个值声明在组件内部，React 也*知道*它永远不会改变。例如，`useState` 返回的 [`set` 函数](/reference/react/useState#setstate)以及 [`useRef`](/reference/react/useRef) 返回的 ref 对象都是 *stable*（稳定的）——它们保证不会在重新渲染时改变。稳定值不是 reactive 的，所以你可以把它们从列表中省略。把它们加进去也可以：它们不会变，所以无所谓。

</Note>

### 当你不想重新同步时该怎么做 {/*what-to-do-when-you-dont-want-to-re-synchronize*/}

在前面的例子中，你通过把 `roomId` 和 `serverUrl` 列为依赖项修复了 lint 错误。

**不过，你也可以向 linter“证明”这些值不是 reactive 的，**也就是说，它们*不会*因为重新渲染而改变。例如，如果 `serverUrl` 和 `roomId` 不依赖于渲染，而且始终保持相同的值，你可以把它们移到组件外面。这样它们就不需要作为依赖项了：

```js {1,2,11}
const serverUrl = 'https://localhost:1234'; // serverUrl 不是 reactive 的
const roomId = 'general'; // roomId 不是 reactive 的

function ChatRoom() {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, []); // ✅ 已声明所有依赖项
  // ...
}
```

你也可以把它们移到 *Effect 内部*。它们不是在渲染期间计算出来的，所以它们不是 reactive 的：

```js {3,4,10}
function ChatRoom() {
  useEffect(() => {
    const serverUrl = 'https://localhost:1234'; // serverUrl 不是 reactive 的
    const roomId = 'general'; // roomId 不是 reactive 的
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, []); // ✅ 已声明所有依赖项
  // ...
}
```

**Effects 是 reactive 的代码块。** 当你在其中读取的值发生变化时，它们会重新同步。与事件处理器不同，事件处理器每次交互只运行一次，而 Effects 会在需要同步时运行。

**你不能“选择”你的依赖项。** 依赖项必须包含你在 Effect 中读取的每一个[reactive 值](#all-variables-declared-in-the-component-body-are-reactive)。linter 会强制执行这一点。有时这会导致诸如无限循环之类的问题，也会让你的 Effect 过于频繁地重新同步。不要通过关闭 linter 来修复这些问题！你应该尝试以下做法：

* **检查你的 Effect 是否表示一个独立的同步过程。** 如果你的 Effect 根本没有同步任何东西，[它也许是不必要的。](/learn/you-might-not-need-an-effect)如果它在同步多个彼此独立的内容，应该[把它拆开。](#each-effect-represents-a-separate-synchronization-process)

* **如果你想读取 props 或 state 的最新值，但又不想“响应”它并重新同步 Effect，**你可以把 Effect 分成 reactive 部分（保留在 Effect 中）和 non-reactive 部分（提取到所谓的 _Effect Event_ 中）。[阅读关于将 Events 与 Effects 分离的内容。](/learn/separating-events-from-effects)

* **避免依赖对象和函数。** 如果你在渲染期间创建对象和函数，然后在 Effect 中读取它们，它们在每次渲染时都会不同。这会导致你的 Effect 每次都重新同步。[阅读更多关于从 Effects 中移除不必要依赖项的内容。](/learn/removing-effect-dependencies)

<Pitfall>

linter 是你的朋友，但它的能力有限。linter 只能知道依赖项什么时候是*错的*，但它不知道每种情况*最好的*解决方式。如果 linter 建议添加某个依赖项，但添加后导致循环，这并不意味着可以忽略 linter。你需要修改 Effect 内部（或外部）的代码，让那个值不再是 reactive 的，也就不再*需要*成为依赖项。

如果你已经有一个现成的代码库，你可能会看到一些 Effect 像这样抑制了 linter：

```js {3-4}
useEffect(() => {
  // ...
  // 🔴 不要像这样抑制 linter：
  // eslint-ignore-next-line react-hooks/exhaustive-deps
}, []);
```

在[下一页](/learn/separating-events-from-effects)[后续页面](/learn/removing-effect-dependencies)中，你会学到如何在不破坏规则的情况下修复这段代码。修复它总是值得的！

</Pitfall>

<Recap>

- 组件可以挂载、更新和卸载。
- 每个 Effect 都有独立于其外层组件的生命周期。
- 每个 Effect 都描述了一个可以*开始*和*停止*的独立同步过程。
- 在编写和读取 Effects 时，要从每个单独 Effect 的角度思考（如何开始和停止同步），而不是从组件的角度思考（它如何挂载、更新或卸载）。
- 在组件函数体内声明的值都是“reactive”的。
- reactive 值应该重新同步 Effect，因为它们会随时间变化。
- linter 会验证在 Effect 内使用到的所有 reactive 值都被指定为依赖项。
- linter 标出的所有错误都是真实存在的问题。总能找到一种修复代码且不违反规则的方法。

</Recap>

<Challenges>

#### 修复每次按键都会重新连接的问题 {/*fix-reconnecting-on-every-keystroke*/}

在这个例子中，`ChatRoom` 组件在组件挂载时连接到聊天室，在卸载时断开连接，并在你选择不同聊天室时重新连接。这个行为是正确的，所以你需要保持它正常工作。

不过，这里有个问题。每当你在底部的消息输入框中打字时，`ChatRoom` *也会*重新连接到聊天。你可以通过清空控制台并在输入框中输入来观察这一点。请修复这个问题，使它不再发生。

<Hint>

你可能需要为这个 Effect 添加一个依赖数组。应该包含哪些依赖项？

</Hint>

<Sandpack>

```js
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  });

  return (
    <>
      <h1>Welcome to the {roomId} room!</h1>
      <input
        value={message}
        onChange={e => setMessage(e.target.value)}
      />
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
export function createConnection(serverUrl, roomId) {
  // 一个真实实现会真正连接到服务器
  return {
    connect() {
      console.log('✅ 正在连接到 "' + roomId + '" 房间，地址为 ' + serverUrl + '...');
    },
    disconnect() {
      console.log('❌ 已从 "' + roomId + '" 房间断开连接，地址为 ' + serverUrl);
    }
  };
}
```

```css
input { display: block; margin-bottom: 20px; }
button { margin-left: 10px; }
```

</Sandpack>

<Solution>

这个 Effect 一开始根本没有依赖数组，所以它会在每次重新渲染后重新同步。首先，添加一个依赖数组。然后，确保 Effect 使用到的每个 reactive 值都被指定在数组中。例如，`roomId` 是 reactive 的（因为它是一个 prop），所以应该把它包含进去。这样可以确保当用户选择不同房间时，聊天会重新连接。另一方面，`serverUrl` 定义在组件外部，所以它不需要放进数组里。

<Sandpack>

```js
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]);

  return (
    <>
      <h1>Welcome to the {roomId} room!</h1>
      <input
        value={message}
        onChange={e => setMessage(e.target.value)}
      />
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
export function createConnection(serverUrl, roomId) {
  // 一个真实实现会真正连接到服务器
  return {
    connect() {
      console.log('✅ 正在连接到 "' + roomId + '" 房间，地址为 ' + serverUrl + '...');
    },
    disconnect() {
      console.log('❌ 已从 "' + roomId + '" 房间断开连接，地址为 ' + serverUrl);
    }
  };
}
```

```css
input { display: block; margin-bottom: 20px; }
button { margin-left: 10px; }
```

</Sandpack>

</Solution>

#### 开关同步的启用与关闭 {/*switch-synchronization-on-and-off*/}

在这个例子中，一个 Effect 订阅了窗口的 [`pointermove`](https://developer.mozilla.org/en-US/docs/Web/API/Element/pointermove_event) 事件，从而让屏幕上的粉色圆点移动。试着将鼠标悬停在预览区域上（如果你在移动设备上，就触摸屏幕），看看粉色圆点如何跟随你的移动。

这里还有一个复选框。勾选复选框会切换 `canMove` 这个 state 变量，但代码里并没有在任何地方使用这个 state 变量。你的任务是修改代码，使得当 `canMove` 为 `false`（复选框未勾选）时，圆点停止移动。等你把复选框重新打开（把 `canMove` 设为 `true`）后，圆点应该再次跟随移动。换句话说，圆点能否移动，应该始终与复选框是否勾选保持同步。

<Hint>

你不能有条件地声明一个 Effect。不过，Effect 内部的代码可以使用条件判断！

</Hint>

<Sandpack>

```js
import { useState, useEffect } from 'react';

export default function App() {
  const [position, setPosition] = useState({ x: 0, y: 0 });
  const [canMove, setCanMove] = useState(true);

  useEffect(() => {
    function handleMove(e) {
      setPosition({ x: e.clientX, y: e.clientY });
    }
    window.addEventListener('pointermove', handleMove);
    return () => window.removeEventListener('pointermove', handleMove);
  }, []);

  return (
    <>
      <label>
        <input type="checkbox"
          checked={canMove}
          onChange={e => setCanMove(e.target.checked)}
        />
        允许圆点移动
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

<Solution>

一种解决方案是把 `setPosition` 调用包裹在 `if (canMove) { ... }` 条件中：

<Sandpack>

```js
import { useState, useEffect } from 'react';

export default function App() {
  const [position, setPosition] = useState({ x: 0, y: 0 });
  const [canMove, setCanMove] = useState(true);

  useEffect(() => {
    function handleMove(e) {
      if (canMove) {
        setPosition({ x: e.clientX, y: e.clientY });
      }
    }
    window.addEventListener('pointermove', handleMove);
    return () => window.removeEventListener('pointermove', handleMove);
  }, [canMove]);

  return (
    <>
      <label>
        <input type="checkbox"
          checked={canMove}
          onChange={e => setCanMove(e.target.checked)}
        />
        允许圆点移动
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

或者，你也可以把*事件订阅*逻辑包裹在 `if (canMove) { ... }` 条件中：

<Sandpack>

```js
import { useState, useEffect } from 'react';

export default function App() {
  const [position, setPosition] = useState({ x: 0, y: 0 });
  const [canMove, setCanMove] = useState(true);

  useEffect(() => {
    function handleMove(e) {
      setPosition({ x: e.clientX, y: e.clientY });
    }
    if (canMove) {
      window.addEventListener('pointermove', handleMove);
      return () => window.removeEventListener('pointermove', handleMove);
    }
  }, [canMove]);

  return (
    <>
      <label>
        <input type="checkbox"
          checked={canMove}
          onChange={e => setCanMove(e.target.checked)}
        />
        允许圆点移动
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

在这两种情况下，`canMove` 都是你在 Effect 中读取的 reactive 变量。这就是为什么它必须被指定在 Effect 依赖列表中。这样可以确保每当它的值改变时，Effect 都会重新同步。

</Solution>

#### 调查一个 stale value bug {/*investigate-a-stale-value-bug*/}

在这个例子中，粉色圆点应该在复选框打开时移动，在复选框关闭时停止移动。相关逻辑已经实现了：`handleMove` 事件处理器会检查 `canMove` state 变量。

然而，不知为何，`handleMove` 内部的 `canMove` state 变量似乎是“stale”的：它总是 `true`，即使你取消勾选复选框后也是如此。这是怎么发生的？找出代码中的错误并修复它。

<Hint>

如果你看到有 linter 规则被抑制了，请移除这个抑制！错误通常就藏在那里。

</Hint>

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
        允许圆点移动
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

<Solution>

原始代码的问题在于抑制了依赖 linter。如果你移除这个抑制，就会看到这个 Effect 依赖于 `handleMove` 函数。这是有道理的：`handleMove` 是在组件函数体内声明的，因此它是一个 reactive 值。每个 reactive 值都必须被指定为依赖项，否则它可能会随着时间变得 stale！

原始代码的作者对 React “撒了谎”，声称这个 Effect 不依赖任何 reactive 值（`[]`）。这就是为什么当 `canMove` 改变后（以及 `handleMove` 一起改变），React 没有重新同步这个 Effect。由于 React 没有重新同步 Effect，作为监听器附加的 `handleMove` 其实是初次渲染时创建的那个 `handleMove` 函数。在初次渲染时，`canMove` 是 `true`，这就是为什么初次渲染时的 `handleMove` 会永远看到这个值。

**如果你从不抑制 linter，就永远不会遇到 stale value 问题。** 这个 bug 有几种不同的解决方式，但你应该始终先移除 linter 抑制，然后修改代码以修复 lint 错误。

你可以把 Effect 依赖改成 `[handleMove]`，但由于它每次渲染都会是一个新定义的函数，不如直接去掉依赖数组。这样 Effect 就会在每次重新渲染后重新同步：

<Sandpack>

```js
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
  });

  return (
    <>
      <label>
        <input type="checkbox"
          checked={canMove}
          onChange={e => setCanMove(e.target.checked)}
        />
        允许圆点移动
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

这个方案可行，但并不理想。如果你在 Effect 中放入 `console.log('Resubscribing')`，你会发现它在每次重新渲染后都会重新订阅。重新订阅很快，但如果能避免这么频繁地做，还是更好。

更好的修复方式是把 `handleMove` 函数移到 *Effect 内部*。这样 `handleMove` 就不会是 reactive 值，因此你的 Effect 也就不会依赖某个函数了。相反，它需要依赖 `canMove`，因为你的代码现在是在 Effect 内部读取它。这与期望的行为一致，因为你的 Effect 现在会和 `canMove` 的值保持同步：

<Sandpack>

```js
import { useState, useEffect } from 'react';

export default function App() {
  const [position, setPosition] = useState({ x: 0, y: 0 });
  const [canMove, setCanMove] = useState(true);

  useEffect(() => {
    function handleMove(e) {
      if (canMove) {
        setPosition({ x: e.clientX, y: e.clientY });
      }
    }

    window.addEventListener('pointermove', handleMove);
    return () => window.removeEventListener('pointermove', handleMove);
  }, [canMove]);

  return (
    <>
      <label>
        <input type="checkbox"
          checked={canMove}
          onChange={e => setCanMove(e.target.checked)}
        />
        允许圆点移动
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

试着在 Effect 主体中添加 `console.log('Resubscribing')`，你会发现现在它只会在你切换复选框（`canMove` 改变）或者编辑代码时重新订阅。这比前一种总是重新订阅的做法更好。

在[将 Events 与 Effects 分离。](/learn/separating-events-from-effects)中，你会学到一种更通用的方法来处理这类问题

</Solution>

#### 修复连接切换问题 {/*fix-a-connection-switch*/}

在这个例子中，`chat.js` 中的聊天服务暴露了两个不同的 API：`createEncryptedConnection` 和 `createUnencryptedConnection`。根组件 `App` 允许用户选择是否启用加密，然后把对应的 API 方法作为 `createConnection` prop 传给子组件 `ChatRoom`。

请注意，最开始控制台日志会显示连接未加密。试着勾选复选框：什么都不会发生。不过，如果你随后更改选中的房间，聊天会重新连接，并且启用加密（你可以从控制台消息中看到这一点）。这是一个 bug。请修复它，使得切换复选框时*也*会导致聊天重新连接。

<Hint>

抑制 linter 总是可疑的。这会不会就是一个 bug？

</Hint>

<Sandpack>

```js src/App.js
import { useState } from 'react';
import ChatRoom from './ChatRoom.js';
import {
  createEncryptedConnection,
  createUnencryptedConnection,
} from './chat.js';

export default function App() {
  const [roomId, setRoomId] = useState('general');
  const [isEncrypted, setIsEncrypted] = useState(false);
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
      <label>
        <input
          type="checkbox"
          checked={isEncrypted}
          onChange={e => setIsEncrypted(e.target.checked)}
        />
        Enable encryption
      </label>
      <hr />
      <ChatRoom
        roomId={roomId}
        createConnection={isEncrypted ?
          createEncryptedConnection :
          createUnencryptedConnection
        }
      />
    </>
  );
}
```

```js {expectedErrors: {'react-compiler': [8]}} src/ChatRoom.js active
import { useState, useEffect } from 'react';

export default function ChatRoom({ roomId, createConnection }) {
  useEffect(() => {
    const connection = createConnection(roomId);
    connection.connect();
    return () => connection.disconnect();
    // eslint-disable-next-line react-hooks/exhaustive-deps
  }, [roomId]);

  return <h1>Welcome to the {roomId} room!</h1>;
}
```

```js src/chat.js
export function createEncryptedConnection(roomId) {
  // 一个真实实现会真正连接到服务器
  return {
    connect() {
      console.log('✅ 🔐 正在连接到 "' + roomId + '"...（已加密）');
    },
    disconnect() {
      console.log('❌ 🔐 已从 "' + roomId + '" 房间断开连接（已加密）');
    }
  };
}

export function createUnencryptedConnection(roomId) {
  // 一个真实实现会真正连接到服务器
  return {
    connect() {
      console.log('✅ 正在连接到 "' + roomId + '"...（未加密）');
    },
    disconnect() {
      console.log('❌ 已从 "' + roomId + '" 房间断开连接（未加密）');
    }
  };
}
```

```css
label { display: block; margin-bottom: 10px; }
```

</Sandpack>

<Solution>

如果你移除 linter 抑制，你会看到一个 lint 错误。问题在于 `createConnection` 是一个 prop，所以它是 reactive 值。它可能会随着时间变化！（而且确实应该变化——当用户勾选复选框时，父组件会传入不同的 `createConnection` prop。）这就是为什么它应该成为依赖项。把它加入列表即可修复这个 bug：

<Sandpack>

```js src/App.js
import { useState } from 'react';
import ChatRoom from './ChatRoom.js';
import {
  createEncryptedConnection,
  createUnencryptedConnection,
} from './chat.js';

export default function App() {
  const [roomId, setRoomId] = useState('general');
  const [isEncrypted, setIsEncrypted] = useState(false);
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
      <label>
        <input
          type="checkbox"
          checked={isEncrypted}
          onChange={e => setIsEncrypted(e.target.checked)}
        />
        Enable encryption
      </label>
      <hr />
      <ChatRoom
        roomId={roomId}
        createConnection={isEncrypted ?
          createEncryptedConnection :
          createUnencryptedConnection
        }
      />
    </>
  );
}
```

```js src/ChatRoom.js active
import { useState, useEffect } from 'react';

export default function ChatRoom({ roomId, createConnection }) {
  useEffect(() => {
    const connection = createConnection(roomId);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId, createConnection]);

  return <h1>Welcome to the {roomId} room!</h1>;
}
```

```js src/chat.js
export function createEncryptedConnection(roomId) {
  // 一个真实实现会真正连接到服务器
  return {
    connect() {
      console.log('✅ 🔐 正在连接到 "' + roomId + '"...（已加密）');
    },
    disconnect() {
      console.log('❌ 🔐 已从 "' + roomId + '" 房间断开连接（已加密）');
    }
  };
}

export function createUnencryptedConnection(roomId) {
  // 一个真实实现会真正连接到服务器
  return {
    connect() {
      console.log('✅ 正在连接到 "' + roomId + '"...（未加密）');
    },
    disconnect() {
      console.log('❌ 已从 "' + roomId + '" 房间断开连接（未加密）');
    }
  };
}
```

```css
label { display: block; margin-bottom: 10px; }
```

</Sandpack>

`createConnection` 确实是一个依赖项。不过，这段代码有点脆弱，因为有人可能会编辑 `App` 组件，让它把一个内联函数作为这个 prop 的值。在那种情况下，它的值会在 `App` 组件每次重新渲染时都不同，所以 Effect 可能会过于频繁地重新同步。为了避免这种情况，你可以改为传递 `isEncrypted`：

<Sandpack>

```js src/App.js
import { useState } from 'react';
import ChatRoom from './ChatRoom.js';

export default function App() {
  const [roomId, setRoomId] = useState('general');
  const [isEncrypted, setIsEncrypted] = useState(false);
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
      <label>
        <input
          type="checkbox"
          checked={isEncrypted}
          onChange={e => setIsEncrypted(e.target.checked)}
        />
        Enable encryption
      </label>
      <hr />
      <ChatRoom
        roomId={roomId}
        isEncrypted={isEncrypted}
      />
    </>
  );
}
```

```js src/ChatRoom.js active
import { useState, useEffect } from 'react';
import {
  createEncryptedConnection,
  createUnencryptedConnection,
} from './chat.js';

export default function ChatRoom({ roomId, isEncrypted }) {
  useEffect(() => {
    const createConnection = isEncrypted ?
      createEncryptedConnection :
      createUnencryptedConnection;
    const connection = createConnection(roomId);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId, isEncrypted]);

  return <h1>Welcome to the {roomId} room!</h1>;
}
```

```js src/chat.js
export function createEncryptedConnection(roomId) {
  // 一个真实实现会真正连接到服务器
  return {
    connect() {
      console.log('✅ 🔐 正在连接到 "' + roomId + '"...（已加密）');
    },
    disconnect() {
      console.log('❌ 🔐 已从 "' + roomId + '" 房间断开连接（已加密）');
    }
  };
}

export function createUnencryptedConnection(roomId) {
  // 一个真实实现会真正连接到服务器
  return {
    connect() {
      console.log('✅ 正在连接到 "' + roomId + '"...（未加密）');
    },
    disconnect() {
      console.log('❌ 已从 "' + roomId + '" 房间断开连接（未加密）');
    }
  };
}
```

```css
label { display: block; margin-bottom: 10px; }
```

</Sandpack>

在这个版本中，`App` 组件传递的是一个布尔 prop，而不是一个函数。在 Effect 内部，你决定使用哪个函数。由于 `createEncryptedConnection` 和 `createUnencryptedConnection` 都是在组件外部声明的，它们不是 reactive 的，因此不需要作为依赖项。你会在[移除 Effect 依赖项。](/learn/removing-effect-dependencies)中了解更多关于这一点的内容

</Solution>

#### 填充一串下拉框 {/*populate-a-chain-of-select-boxes*/}

在这个例子中，有两个下拉框。一个下拉框让用户选择一个星球。另一个下拉框让用户选择该星球上的一个地点。第二个下拉框目前还不能工作。你的任务是让它显示所选星球上的地点。

看看第一个下拉框是如何工作的。它通过 `"/planets"` API 调用的结果来填充 `planetList` state。当前选中的星球 ID 保存在 `planetId` state 变量中。你需要找到合适的位置添加一些额外代码，让 `placeList` state 变量通过 `"/planets/" + planetId + "/places"` API 调用的结果来填充。

如果你实现正确，选择一个星球就会填充地点列表。切换星球也应该改变地点列表。

<Hint>

如果你有两个独立的同步过程，你需要编写两个单独的 Effects。

</Hint>

<Sandpack>

```js src/App.js
import { useState, useEffect } from 'react';
import { fetchData } from './api.js';

export default function Page() {
  const [planetList, setPlanetList] = useState([])
  const [planetId, setPlanetId] = useState('');

  const [placeList, setPlaceList] = useState([]);
  const [placeId, setPlaceId] = useState('');

  useEffect(() => {
    let ignore = false;
    fetchData('/planets').then(result => {
      if (!ignore) {
        console.log('Fetched a list of planets.');
        setPlanetList(result);
        setPlanetId(result[0].id); // 选择第一个星球
      }
    });
    return () => {
      ignore = true;
    }
  }, []);

  return (
    <>
      <label>
        Pick a planet:{' '}
        <select value={planetId} onChange={e => {
          setPlanetId(e.target.value);
        }}>
          {planetList.map(planet =>
            <option key={planet.id} value={planet.id}>{planet.name}</option>
          )}
        </select>
      </label>
      <label>
        Pick a place:{' '}
        <select value={placeId} onChange={e => {
          setPlaceId(e.target.value);
        }}>
          {placeList.map(place =>
            <option key={place.id} value={place.id}>{place.name}</option>
          )}
        </select>
      </label>
      <hr />
      <p>You are going to: {placeId || '???'} on {planetId || '???'} </p>
    </>
  );
}
```

```js src/api.js hidden
export function fetchData(url) {
  if (url === '/planets') {
    return fetchPlanets();
  } else if (url.startsWith('/planets/')) {
    const match = url.match(/^\/planets\/([\w-]+)\/places(\/)?$/);
    if (!match || !match[1] || !match[1].length) {
      throw Error('Expected URL like "/planets/earth/places". Received: "' + url + '".');
    }
    return fetchPlaces(match[1]);
  } else throw Error('Expected URL like "/planets" or "/planets/earth/places". Received: "' + url + '".');
}

async function fetchPlanets() {
  return new Promise(resolve => {
    setTimeout(() => {
      resolve([{
        id: 'earth',
        name: 'Earth'
      }, {
        id: 'venus',
        name: 'Venus'
      }, {
        id: 'mars',
        name: 'Mars'
      }]);
    }, 1000);
  });
}

async function fetchPlaces(planetId) {
  if (typeof planetId !== 'string') {
    throw Error(
      'fetchPlaces(planetId) expects a string argument. ' +
      'Instead received: ' + planetId + '.'
    );
  }
  return new Promise(resolve => {
    setTimeout(() => {
      if (planetId === 'earth') {
        resolve([{
          id: 'laos',
          name: 'Laos'
        }, {
          id: 'spain',
          name: 'Spain'
        }, {
          id: 'vietnam',
          name: 'Vietnam'
        }]);
      } else if (planetId === 'venus') {
        resolve([{
          id: 'aurelia',
          name: 'Aurelia'
        }, {
          id: 'diana-chasma',
          name: 'Diana Chasma'
        }, {
          id: 'kumsong-vallis',
          name: 'Kŭmsŏng Vallis'
        }]);
      } else if (planetId === 'mars') {
        resolve([{
          id: 'aluminum-city',
          name: 'Aluminum City'
        }, {
          id: 'new-new-york',
          name: 'New New York'
        }, {
          id: 'vishniac',
          name: 'Vishniac'
        }]);
      } else throw Error('Unknown planet ID: ' + planetId);
    }, 1000);
  });
}
```

```css
label { display: block; margin-bottom: 10px; }
```

</Sandpack>

<Solution>

这里有两个彼此独立的同步过程：

- 第一个下拉框与远程星球列表保持同步。
- 第二个下拉框与当前 `planetId` 对应的远程地点列表保持同步。

这就是为什么把它们描述为两个独立 Effects 是合理的。下面是一个实现示例：

<Sandpack>

```js src/App.js
import { useState, useEffect } from 'react';
import { fetchData } from './api.js';

export default function Page() {
  const [planetList, setPlanetList] = useState([])
  const [planetId, setPlanetId] = useState('');

  const [placeList, setPlaceList] = useState([]);
  const [placeId, setPlaceId] = useState('');

  useEffect(() => {
    let ignore = false;
    fetchData('/planets').then(result => {
      if (!ignore) {
        console.log('Fetched a list of planets.');
        setPlanetList(result);
        setPlanetId(result[0].id); // 选择第一个星球
      }
    });
    return () => {
      ignore = true;
    }
  }, []);

  useEffect(() => {
    if (planetId === '') {
      // 第一个框里还没有选中任何内容
      return;
    }

    let ignore = false;
    fetchData('/planets/' + planetId + '/places').then(result => {
      if (!ignore) {
        console.log('Fetched a list of places on "' + planetId + '".');
        setPlaceList(result);
        setPlaceId(result[0].id); // 选择第一个地点
      }
    });
    return () => {
      ignore = true;
    }
  }, [planetId]);

  return (
    <>
      <label>
        Pick a planet:{' '}
        <select value={planetId} onChange={e => {
          setPlanetId(e.target.value);
        }}>
          {planetList.map(planet =>
            <option key={planet.id} value={planet.id}>{planet.name}</option>
          )}
        </select>
      </label>
      <label>
        Pick a place:{' '}
        <select value={placeId} onChange={e => {
          setPlaceId(e.target.value);
        }}>
          {placeList.map(place =>
            <option key={place.id} value={place.id}>{place.name}</option>
          )}
        </select>
      </label>
      <hr />
      <p>You are going to: {placeId || '???'} on {planetId || '???'} </p>
    </>
  );
}
```

```js src/api.js hidden
export function fetchData(url) {
  if (url === '/planets') {
    return fetchPlanets();
  } else if (url.startsWith('/planets/')) {
    const match = url.match(/^\/planets\/([\w-]+)\/places(\/)?$/);
    if (!match || !match[1] || !match[1].length) {
      throw Error('期望的 URL 形式为 "/planets/earth/places"。收到的是："' + url + '"。');
    }
    return fetchPlaces(match[1]);
  } else throw Error('期望的 URL 形式为 "/planets" 或 "/planets/earth/places"。收到的是："' + url + '"。');
}

async function fetchPlanets() {
  return new Promise(resolve => {
    setTimeout(() => {
      resolve([{
        id: 'earth',
        name: 'Earth'
      }, {
        id: 'venus',
        name: 'Venus'
      }, {
        id: 'mars',
        name: 'Mars'
      }]);
    }, 1000);
  });
}

async function fetchPlaces(planetId) {
  if (typeof planetId !== 'string') {
    throw Error(
      'fetchPlaces(planetId) 期望一个字符串参数。' +
      '实际收到的是：' + planetId + '。'
    );
  }
  return new Promise(resolve => {
    setTimeout(() => {
      if (planetId === 'earth') {
        resolve([{
          id: 'laos',
          name: 'Laos'
        }, {
          id: 'spain',
          name: 'Spain'
        }, {
          id: 'vietnam',
          name: 'Vietnam'
        }]);
      } else if (planetId === 'venus') {
        resolve([{
          id: 'aurelia',
          name: 'Aurelia'
        }, {
          id: 'diana-chasma',
          name: 'Diana Chasma'
        }, {
          id: 'kumsong-vallis',
          name: 'Kŭmsŏng Vallis'
        }]);
      } else if (planetId === 'mars') {
        resolve([{
          id: 'aluminum-city',
          name: 'Aluminum City'
        }, {
          id: 'new-new-york',
          name: 'New New York'
        }, {
          id: 'vishniac',
          name: 'Vishniac'
        }]);
      } else throw Error('未知的行星 ID：' + planetId);
    }, 1000);
  });
}
```

```css
label { display: block; margin-bottom: 10px; }
```

</Sandpack>

这段代码有点重复。不过，这并不是把它合并成单个 Effect 的好理由！如果你这么做，就必须把两个 Effect 的依赖项合并到一个列表中，然后更改行星时会重新获取所有行星的列表。Effect 不是用于代码复用的工具。

相反，为了减少重复，你可以把一些逻辑提取到下面这样的自定义 Hook 中，比如 `useSelectOptions`：

<Sandpack>

```js src/App.js
import { useState } from 'react';
import { useSelectOptions } from './useSelectOptions.js';

export default function Page() {
  const [
    planetList,
    planetId,
    setPlanetId
  ] = useSelectOptions('/planets');

  const [
    placeList,
    placeId,
    setPlaceId
  ] = useSelectOptions(planetId ? `/planets/${planetId}/places` : null);

  return (
    <>
      <label>
        选择一个行星：{' '}
        <select value={planetId} onChange={e => {
          setPlanetId(e.target.value);
        }}>
          {planetList?.map(planet =>
            <option key={planet.id} value={planet.id}>{planet.name}</option>
          )}
        </select>
      </label>
      <label>
        选择一个地点：{' '}
        <select value={placeId} onChange={e => {
          setPlaceId(e.target.value);
        }}>
          {placeList?.map(place =>
            <option key={place.id} value={place.id}>{place.name}</option>
          )}
        </select>
      </label>
      <hr />
      <p>你将去往：{placeId || '...'}，位于 {planetId || '...'} </p>
    </>
  );
}
```

```js src/useSelectOptions.js
import { useState, useEffect } from 'react';
import { fetchData } from './api.js';

export function useSelectOptions(url) {
  const [list, setList] = useState(null);
  const [selectedId, setSelectedId] = useState('');
  useEffect(() => {
    if (url === null) {
      return;
    }

    let ignore = false;
    fetchData(url).then(result => {
      if (!ignore) {
        setList(result);
        setSelectedId(result[0].id);
      }
    });
    return () => {
      ignore = true;
    }
  }, [url]);
  return [list, selectedId, setSelectedId];
}
```

```js src/api.js hidden
export function fetchData(url) {
  if (url === '/planets') {
    return fetchPlanets();
  } else if (url.startsWith('/planets/')) {
    const match = url.match(/^\/planets\/([\w-]+)\/places(\/)?$/);
    if (!match || !match[1] || !match[1].length) {
      throw Error('期望的 URL 形式为 "/planets/earth/places"。收到的是："' + url + '"。');
    }
    return fetchPlaces(match[1]);
  } else throw Error('期望的 URL 形式为 "/planets" 或 "/planets/earth/places"。收到的是："' + url + '"。');
}

async function fetchPlanets() {
  return new Promise(resolve => {
    setTimeout(() => {
      resolve([{
        id: 'earth',
        name: 'Earth'
      }, {
        id: 'venus',
        name: 'Venus'
      }, {
        id: 'mars',
        name: 'Mars'
      }]);
    }, 1000);
  });
}

async function fetchPlaces(planetId) {
  if (typeof planetId !== 'string') {
    throw Error(
      'fetchPlaces(planetId) 期望一个字符串参数。' +
      '实际收到的是：' + planetId + '。'
    );
  }
  return new Promise(resolve => {
    setTimeout(() => {
      if (planetId === 'earth') {
        resolve([{
          id: 'laos',
          name: 'Laos'
        }, {
          id: 'spain',
          name: 'Spain'
        }, {
          id: 'vietnam',
          name: 'Vietnam'
        }]);
      } else if (planetId === 'venus') {
        resolve([{
          id: 'aurelia',
          name: 'Aurelia'
        }, {
          id: 'diana-chasma',
          name: 'Diana Chasma'
        }, {
          id: 'kumsong-vallis',
          name: 'Kŭmsŏng Vallis'
        }]);
      } else if (planetId === 'mars') {
        resolve([{
          id: 'aluminum-city',
          name: 'Aluminum City'
        }, {
          id: 'new-new-york',
          name: 'New New York'
        }, {
          id: 'vishniac',
          name: 'Vishniac'
        }]);
      } else throw Error('未知的行星 ID：' + planetId);
    }, 1000);
  });
}
```

```css
label { display: block; margin-bottom: 10px; }
```

</Sandpack>

查看沙盒中的 `useSelectOptions.js` 选项卡，了解它是如何工作的。理想情况下，你应用中的大多数 Effect 最终都应该被自定义 Hook 替代，无论这些 Hook 是你自己编写的还是社区提供的。自定义 Hook 会隐藏同步逻辑，因此调用它的组件无需了解 Effect。随着你继续开发应用，你会逐渐积累一套可供选择的 Hook，最终就不需要再在组件中频繁编写 Effect 了。

</Solution>

</Challenges>
