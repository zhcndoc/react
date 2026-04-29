---
title: useTransition
---

<Intro>

`useTransition` 是一个 React Hook，允许你在后台渲染 UI 的一部分。

```js
const [isPending, startTransition] = useTransition()
```

</Intro>

<InlineToc />

---

## 参考 {/*reference*/}

### `useTransition()` {/*usetransition*/}

在组件顶层调用 `useTransition`，以将某些状态更新标记为 Transition。

```js
import { useTransition } from 'react';

function TabContainer() {
  const [isPending, startTransition] = useTransition();
  // ...
}
```

[查看更多示例。](#usage)

#### 参数 {/*parameters*/}

`useTransition` 不接受任何参数。

#### 返回值 {/*returns*/}

`useTransition` 返回一个包含恰好两个项目的数组：

1. `isPending` 标志，告诉你是否存在一个待处理的 Transition。
2. [`startTransition` 函数](#starttransition)，让你可以将更新标记为 Transition。

---

### `startTransition(action)` {/*starttransition*/}

`useTransition` 返回的 `startTransition` 函数允许你将更新标记为 Transition。

```js {6,8}
function TabContainer() {
  const [isPending, startTransition] = useTransition();
  const [tab, setTab] = useState('about');

  function selectTab(nextTab) {
    startTransition(() => {
      setTab(nextTab);
    });
  }
  // ...
}
```

<Note>
#### 在 `startTransition` 中调用的函数被称为“Actions”。 {/*functions-called-in-starttransition-are-called-actions*/}

传递给 `startTransition` 的函数被称为 “Action”。按照约定，在 `startTransition` 内部调用的任何回调（例如回调 prop）都应命名为 `action`，或包含 “Action” 后缀：

```js {1,9}
function SubmitButton({ submitAction }) {
  const [isPending, startTransition] = useTransition();

  return (
    <button
      disabled={isPending}
      onClick={() => {
        startTransition(async () => {
          await submitAction();
        });
      }}
    >
      Submit
    </button>
  );
}

```

</Note>



#### 参数 {/*starttransition-parameters*/}

* `action`：一个通过调用一个或多个 [`set` 函数](/reference/react/useState#setstate) 来更新某些状态的函数。React 会立即以无参数的形式调用 `action`，并将 `action` 函数调用期间同步调度的所有状态更新标记为 Transitions。`action` 中任何被 `await` 的异步调用也会包含在 Transition 中，但目前需要将 `await` 之后的任何 `set` 函数再用一个额外的 `startTransition` 包裹起来（参见[故障排除](#react-doesnt-treat-my-state-update-after-await-as-a-transition)）。被标记为 Transitions 的状态更新将是[非阻塞的](#perform-non-blocking-updates-with-actions)，并且[不会显示不必要的加载指示器](#preventing-unwanted-loading-indicators)。

#### 返回值 {/*starttransition-returns*/}

`startTransition` 不返回任何内容。

#### 注意事项 {/*starttransition-caveats*/}

* `useTransition` 是一个 Hook，因此它只能在组件或自定义 Hook 内部调用。如果你需要在别处启动 Transition（例如从数据库中），请改用独立的 [`startTransition`](/reference/react/startTransition)。

* 只有在你能够访问该状态的 `set` 函数时，才能将更新包装为 Transition。如果你想根据某个 prop 或自定义 Hook 的值启动 Transition，请尝试改用 [`useDeferredValue`](/reference/react/useDeferredValue)。

* 传递给 `startTransition` 的函数会立即被调用，凡是在其执行期间发生的所有状态更新都会被标记为 Transitions。比如，如果你尝试在 `setTimeout` 中执行状态更新，它们就不会被标记为 Transitions。

* 你必须将任何异步请求后的状态更新再包裹在另一个 `startTransition` 中，才能将它们标记为 Transitions。这是一个已知限制，我们会在未来修复（参见[故障排除](#react-doesnt-treat-my-state-update-after-await-as-a-transition)）。

* `startTransition` 函数具有稳定的身份，因此你经常会在 Effect 依赖项中看到它被省略，但把它包含进去也不会导致 Effect 触发。如果 linter 允许你省略某个依赖而不报错，那么这样做是安全的。[了解更多关于移除 Effect 依赖项的内容。](/learn/removing-effect-dependencies#move-dynamic-objects-and-functions-inside-your-effect)

* 被标记为 Transition 的状态更新会被其他状态更新中断。比如，如果你在 Transition 中更新一个图表组件，但在图表重新渲染到一半时开始在输入框中输入，React 会在处理输入更新后重新开始图表组件的渲染工作。

* Transition 更新不能用于控制文本输入框。

* 如果存在多个正在进行的 Transitions，React 目前会将它们批量合并。这是一个在未来版本中可能被移除的限制。

## 用法 {/*usage*/}

### 使用 Actions 执行非阻塞更新 {/*perform-non-blocking-updates-with-actions*/}

在组件顶部调用 `useTransition` 来创建 Actions，并访问 pending 状态：

```js [[1, 4, "isPending"], [2, 4, "startTransition"]]
import {useState, useTransition} from 'react';

function CheckoutForm() {
  const [isPending, startTransition] = useTransition();
  // ...
}
```

`useTransition` 返回一个包含恰好两个项目的数组：

1. <CodeStep step={1}>`isPending` 标志</CodeStep>，告诉你是否存在一个待处理的 Transition。
2. <CodeStep step={2}>`startTransition` 函数</CodeStep>，让你创建一个 Action。

要启动一个 Transition，像这样向 `startTransition` 传入一个函数：

```js
import {useState, useTransition} from 'react';
import {updateQuantity} from './api';

function CheckoutForm() {
  const [isPending, startTransition] = useTransition();
  const [quantity, setQuantity] = useState(1);

  function onSubmit(newQuantity) {
    startTransition(async function () {
      const savedQuantity = await updateQuantity(newQuantity);
      startTransition(() => {
        setQuantity(savedQuantity);
      });
    });
  }
  // ...
}
```

传递给 `startTransition` 的函数被称为 “Action”。你可以在 Action 中更新状态，并且（可选地）执行副作用，这些工作会在后台完成，不会阻塞页面上的用户交互。一个 Transition 可以包含多个 Actions，而且在 Transition 进行期间，UI 会保持响应。例如，如果用户点击了一个标签页，然后又改变主意点击了另一个标签页，那么第二次点击会被立即处理，而无需等待第一次更新完成。

为了给用户提供正在进行中的 Transition 的反馈，`isPending` 状态会在第一次调用 `startTransition` 时切换为 `true`，并保持为 `true`，直到所有 Actions 都完成并且最终状态显示给用户。Transitions 会确保 Actions 中的副作用按顺序完成，以[防止不必要的加载指示器](#preventing-unwanted-loading-indicators)，并且你可以在 Transition 进行时使用 `useOptimistic` 提供即时反馈。

<Recipes titleText="Actions 与常规事件处理的区别">

#### 在 Action 中更新数量 {/*updating-the-quantity-in-an-action*/}

在这个示例中，`updateQuantity` 函数模拟向服务器发送请求，以更新购物车中商品的数量。这个函数被*人为放慢*，因此请求至少需要一秒才能完成。

快速多次更新数量。注意，在任何请求进行中时，pending 的 “Total” 状态会显示出来，而 “Total” 只会在最后一次请求完成后更新。由于更新是在 Action 中进行的，因此在请求进行期间，“quantity” 仍然可以继续更新。

<Sandpack>

```json package.json hidden
{
  "dependencies": {
    "react": "beta",
    "react-dom": "beta"
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
import { useState, useTransition } from "react";
import { updateQuantity } from "./api";
import Item from "./Item";
import Total from "./Total";

export default function App({}) {
  const [quantity, setQuantity] = useState(1);
  const [isPending, startTransition] = useTransition();

  const updateQuantityAction = async newQuantity => {
    // 要访问 Transition 的 pending 状态，
    // 再次调用 startTransition。
    startTransition(async () => {
      const savedQuantity = await updateQuantity(newQuantity);
      startTransition(() => {
        setQuantity(savedQuantity);
      });
    });
  };

  return (
    <div>
      <h1>结账</h1>
      <Item action={updateQuantityAction}/>
      <hr />
      <Total quantity={quantity} isPending={isPending} />
    </div>
  );
}
```

```js src/Item.js
import { startTransition } from "react";

export default function Item({action}) {
  function handleChange(event) {
    // 要暴露一个 action prop，请在 startTransition 中 await 该回调。
    startTransition(async () => {
      await action(event.target.value);
    })
  }
  return (
    <div className="item">
      <span>Eras Tour 门票</span>
      <label htmlFor="name">数量： </label>
      <input
        type="number"
        onChange={handleChange}
        defaultValue={1}
        min={1}
      />
    </div>
  )
}
```

```js src/Total.js
const intl = new Intl.NumberFormat("en-US", {
  style: "currency",
  currency: "USD"
});

export default function Total({quantity, isPending}) {
  return (
    <div className="total">
      <span>总计：</span>
      <span>
        {isPending ? "🌀 正在更新..." : `${intl.format(quantity * 9999)}`}
      </span>
    </div>
  )
}
```

```js src/api.js
export async function updateQuantity(newQuantity) {
  return new Promise((resolve, reject) => {
    // 模拟一个缓慢的网络请求。
    setTimeout(() => {
      resolve(newQuantity);
    }, 2000);
  });
}
```

```css
.item {
  display: flex;
  align-items: center;
  justify-content: start;
}

.item label {
  flex: 1;
  text-align: right;
}

.item input {
  margin-left: 4px;
  width: 60px;
  padding: 4px;
}

.total {
  height: 50px;
  line-height: 25px;
  display: flex;
  align-content: center;
  justify-content: space-between;
}
```

</Sandpack>

这是一个演示 Actions 如何工作的基础示例，但它没有处理请求乱序完成的问题。当多次更新数量时，先前的请求有可能在后续请求之后完成，从而导致数量更新顺序错乱。这是一个已知限制，我们会在未来修复（见下方[故障排除](#my-state-updates-in-transitions-are-out-of-order)）。

对于常见用例，React 提供了内置抽象，例如：
- [`useActionState`](/reference/react/useActionState)
- [`<form>` actions](/reference/react-dom/components/form)
- [Server Functions](/reference/rsc/server-functions)

这些解决方案会帮你处理请求顺序问题。使用 Transitions 构建自己管理异步状态转换的自定义 Hook 或库时，你拥有更高的请求顺序控制能力，但需要自行处理。

<Solution />

#### 不使用 Action 更新数量 {/*updating-the-users-name-without-an-action*/}

在这个示例中，`updateQuantity` 函数同样模拟了向服务器发送请求，以更新购物车中商品的数量。这个函数被*人为放慢*，因此请求至少需要一秒才能完成。

快速多次更新数量。注意，在任何请求进行中时，pending 的 “Total” 状态会显示出来，但每次点击 “quantity” 时，“Total” 会更新多次：

<Sandpack>

```json package.json hidden
{
  "dependencies": {
    "react": "beta",
    "react-dom": "beta"
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
import { useState } from "react";
import { updateQuantity } from "./api";
import Item from "./Item";
import Total from "./Total";

export default function App({}) {
  const [quantity, setQuantity] = useState(1);
  const [isPending, setIsPending] = useState(false);

  const onUpdateQuantity = async newQuantity => {
    // 手动设置 isPending 状态。
    setIsPending(true);
    const savedQuantity = await updateQuantity(newQuantity);
    setIsPending(false);
    setQuantity(savedQuantity);
  };

  return (
    <div>
      <h1>结账</h1>
      <Item onUpdateQuantity={onUpdateQuantity}/>
      <hr />
      <Total quantity={quantity} isPending={isPending} />
    </div>
  );
}

```

```js src/Item.js
export default function Item({onUpdateQuantity}) {
  function handleChange(event) {
    onUpdateQuantity(event.target.value);
  }
  return (
    <div className="item">
      <span>Eras Tour 门票</span>
      <label htmlFor="name">数量： </label>
      <input
        type="number"
        onChange={handleChange}
        defaultValue={1}
        min={1}
      />
    </div>
  )
}
```

```js src/Total.js
const intl = new Intl.NumberFormat("en-US", {
  style: "currency",
  currency: "USD"
});

export default function Total({quantity, isPending}) {
  return (
    <div className="total">
      <span>总计：</span>
      <span>
        {isPending ? "🌀 正在更新..." : `${intl.format(quantity * 9999)}`}
      </span>
    </div>
  )
}
```

```js src/api.js
export async function updateQuantity(newQuantity) {
  return new Promise((resolve, reject) => {
    // 模拟一个缓慢的网络请求。
    setTimeout(() => {
      resolve(newQuantity);
    }, 2000);
  });
}
```

```css
.item {
  display: flex;
  align-items: center;
  justify-content: start;
}

.item label {
  flex: 1;
  text-align: right;
}

.item input {
  margin-left: 4px;
  width: 60px;
  padding: 4px;
}

.total {
  height: 50px;
  line-height: 25px;
  display: flex;
  align-content: center;
  justify-content: space-between;
}
```

</Sandpack>

解决这个问题的一种常见方法是在数量更新期间阻止用户进行更改：

<Sandpack>

```json package.json hidden
{
  "dependencies": {
    "react": "beta",
    "react-dom": "beta"
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
import { useState, useTransition } from "react";
import { updateQuantity } from "./api";
import Item from "./Item";
import Total from "./Total";

export default function App({}) {
  const [quantity, setQuantity] = useState(1);
  const [isPending, setIsPending] = useState(false);

  const onUpdateQuantity = async event => {
    const newQuantity = event.target.value;
    // 手动设置 isPending 状态。
    setIsPending(true);
    const savedQuantity = await updateQuantity(newQuantity);
    setIsPending(false);
    setQuantity(savedQuantity);
  };

  return (
    <div>
      <h1>结账</h1>
      <Item isPending={isPending} onUpdateQuantity={onUpdateQuantity}/>
      <hr />
      <Total quantity={quantity} isPending={isPending} />
    </div>
  );
}

```

```js src/Item.js
export default function Item({isPending, onUpdateQuantity}) {
  return (
    <div className="item">
      <span>Eras Tour 门票</span>
      <label htmlFor="name">数量： </label>
      <input
        type="number"
        disabled={isPending}
        onChange={onUpdateQuantity}
        defaultValue={1}
        min={1}
      />
    </div>
  )
}
```

```js src/Total.js
const intl = new Intl.NumberFormat("en-US", {
  style: "currency",
  currency: "USD"
});

export default function Total({quantity, isPending}) {
  return (
    <div className="total">
      <span>总计：</span>
      <span>
        {isPending ? "🌀 正在更新..." : `${intl.format(quantity * 9999)}`}
      </span>
    </div>
  )
}
```

```js src/api.js
export async function updateQuantity(newQuantity) {
  return new Promise((resolve, reject) => {
    // 模拟一个缓慢的网络请求。
    setTimeout(() => {
      resolve(newQuantity);
    }, 2000);
  });
}
```

```css
.item {
  display: flex;
  align-items: center;
  justify-content: start;
}

.item label {
  flex: 1;
  text-align: right;
}

.item input {
  margin-left: 4px;
  width: 60px;
  padding: 4px;
}

.total {
  height: 50px;
  line-height: 25px;
  display: flex;
  align-content: center;
  justify-content: space-between;
}
```

</Sandpack>

这个解决方案会让应用显得很慢，因为用户每次更新数量都必须等待。你可以手动添加更复杂的处理逻辑，让用户在数量更新时仍能与 UI 交互，但 Actions 通过一个简单直接的内置 API 处理了这个场景。

<Solution />

</Recipes>

---

### 从组件中暴露 `action` prop {/*exposing-action-props-from-components*/}

你可以从组件中暴露一个 `action` prop，让父组件可以调用一个 Action。

例如，这个 `TabButton` 组件将其 `onClick` 逻辑封装在一个 `action` prop 中：

```js {8-12}
export default function TabButton({ action, children, isActive }) {
  const [isPending, startTransition] = useTransition();
  if (isActive) {
    return <b>{children}</b>
  }
  return (
    <button onClick={() => {
      startTransition(async () => {
        // 等待传入的 action。
        // 这样它既可以是同步的，也可以是异步的。
        await action();
      });
    }}>
      {children}
    </button>
  );
}
```

由于父组件在 `action` 中更新其状态，该状态更新会被标记为 Transition。这意味着你可以点击 “Posts”，然后立刻点击 “Contact”，而不会阻塞用户交互：

<Sandpack>

```js
import { useState } from 'react';
import TabButton from './TabButton.js';
import AboutTab from './AboutTab.js';
import PostsTab from './PostsTab.js';
import ContactTab from './ContactTab.js';

export default function TabContainer() {
  const [tab, setTab] = useState('about');
  return (
    <>
      <TabButton
        isActive={tab === 'about'}
        action={() => setTab('about')}
      >
        About
      </TabButton>
      <TabButton
        isActive={tab === 'posts'}
        action={() => setTab('posts')}
      >
        Posts (slow)
      </TabButton>
      <TabButton
        isActive={tab === 'contact'}
        action={() => setTab('contact')}
      >
        Contact
      </TabButton>
      <hr />
      {tab === 'about' && <AboutTab />}
      {tab === 'posts' && <PostsTab />}
      {tab === 'contact' && <ContactTab />}
    </>
  );
}
```

```js src/TabButton.js active
import { useTransition } from 'react';

export default function TabButton({ action, children, isActive }) {
  const [isPending, startTransition] = useTransition();
  if (isActive) {
    return <b>{children}</b>
  }
  if (isPending) {
    return <b className="pending">{children}</b>;
  }
  return (
    <button onClick={async () => {
      startTransition(async () => {
        // 等待传入的 action。
        // 这样它既可以是同步的，也可以是异步的。
        await action();
      });
    }}>
      {children}
    </button>
  );
}
```

```js src/AboutTab.js
export default function AboutTab() {
  return (
    <p>欢迎来到我的个人主页！</p>
  );
}
```

```js {expectedErrors: {'react-compiler': [19, 20]}} src/PostsTab.js
import { memo } from 'react';

const PostsTab = memo(function PostsTab() {
  // 记录一次日志。实际的变慢发生在 SlowPost 内部。
  console.log('[ARTIFICIALLY SLOW] Rendering 500 <SlowPost />');

  let items = [];
  for (let i = 0; i < 500; i++) {
    items.push(<SlowPost key={i} index={i} />);
  }
  return (
    <ul className="items">
      {items}
    </ul>
  );
});

function SlowPost({ index }) {
  let startTime = performance.now();
  while (performance.now() - startTime < 1) {
    // 每个项目空转 1 ms，以模拟极慢的代码
  }

  return (
    <li className="item">
      Post #{index + 1}
    </li>
  );
}

export default PostsTab;
```

```js src/ContactTab.js
export default function ContactTab() {
  return (
    <>
      <p>
        你可以在这里找到我：
      </p>
      <ul>
        <li>admin@mysite.com</li>
        <li>+123456789</li>
      </ul>
    </>
  );
}
```

```css
button { margin-right: 10px }
b { display: inline-block; margin-right: 10px; }
.pending { color: #777; }
```

</Sandpack>

<Note>

当从组件中暴露一个 `action` prop 时，你应该在 transition 内部 `await` 它。

这样 `action` 回调既可以是同步的，也可以是异步的，而不需要在 action 内部再用额外的 `startTransition` 去包裹 `await`。

</Note>

---

### 显示 pending 的视觉状态 {/*displaying-a-pending-visual-state*/}

你可以使用 `useTransition` 返回的 `isPending` 布尔值来向用户指示一个 Transition 正在进行中。例如，标签按钮可以有一个特殊的 “pending” 视觉状态：

```js {4-6}
function TabButton({ action, children, isActive }) {
  const [isPending, startTransition] = useTransition();
  // ...
  if (isPending) {
    return <b className="pending">{children}</b>;
  }
  // ...
```

注意，现在点击 “Posts” 感觉更灵敏了，因为标签按钮本身会立即更新：

<Sandpack>

```js
import { useState } from 'react';
import TabButton from './TabButton.js';
import AboutTab from './AboutTab.js';
import PostsTab from './PostsTab.js';
import ContactTab from './ContactTab.js';

export default function TabContainer() {
  const [tab, setTab] = useState('about');
  return (
    <>
      <TabButton
        isActive={tab === 'about'}
        action={() => setTab('about')}
      >
        About
      </TabButton>
      <TabButton
        isActive={tab === 'posts'}
        action={() => setTab('posts')}
      >
        Posts (slow)
      </TabButton>
      <TabButton
        isActive={tab === 'contact'}
        action={() => setTab('contact')}
      >
        Contact
      </TabButton>
      <hr />
      {tab === 'about' && <AboutTab />}
      {tab === 'posts' && <PostsTab />}
      {tab === 'contact' && <ContactTab />}
    </>
  );
}
```

```js src/TabButton.js active
import { useTransition } from 'react';

export default function TabButton({ action, children, isActive }) {
  const [isPending, startTransition] = useTransition();
  if (isActive) {
    return <b>{children}</b>
  }
  if (isPending) {
    return <b className="pending">{children}</b>;
  }
  return (
    <button onClick={() => {
      startTransition(async () => {
        await action();
      });
    }}>
      {children}
    </button>
  );
}
```

```js src/AboutTab.js
export default function AboutTab() {
  return (
    <p>欢迎来到我的个人主页！</p>
  );
}
```

```js {expectedErrors: {'react-compiler': [19, 20]}} src/PostsTab.js
import { memo } from 'react';

const PostsTab = memo(function PostsTab() {
  // 记录一次日志。实际的变慢发生在 SlowPost 内部。
  console.log('[ARTIFICIALLY SLOW] Rendering 500 <SlowPost />');

  let items = [];
  for (let i = 0; i < 500; i++) {
    items.push(<SlowPost key={i} index={i} />);
  }
  return (
    <ul className="items">
      {items}
    </ul>
  );
});

function SlowPost({ index }) {
  let startTime = performance.now();
  while (performance.now() - startTime < 1) {
    // 每个项目空转 1 ms，以模拟极慢的代码
  }

  return (
    <li className="item">
      Post #{index + 1}
    </li>
  );
}

export default PostsTab;
```

```js src/ContactTab.js
export default function ContactTab() {
  return (
    <>
      <p>
        你可以在这里找到我：
      </p>
      <ul>
        <li>admin@mysite.com</li>
        <li>+123456789</li>
      </ul>
    </>
  );
}
```

```css
button { margin-right: 10px }
b { display: inline-block; margin-right: 10px; }
.pending { color: #777; }
```

</Sandpack>

---

### 防止不必要的加载指示器 {/*preventing-unwanted-loading-indicators*/}

在这个示例中，`PostsTab` 组件使用 [use](/reference/react/use) 获取一些数据。当你点击 “Posts” 标签时，`PostsTab` 组件会*挂起*，导致最近的加载 fallback 出现：

<Sandpack>

```js
import { Suspense, useState } from 'react';
import TabButton from './TabButton.js';
import AboutTab from './AboutTab.js';
import PostsTab from './PostsTab.js';
import ContactTab from './ContactTab.js';

export default function TabContainer() {
  const [tab, setTab] = useState('about');
  return (
    <Suspense fallback={<h1>🌀 正在加载...</h1>}>
      <TabButton
        isActive={tab === 'about'}
        action={() => setTab('about')}
      >
        About
      </TabButton>
      <TabButton
        isActive={tab === 'posts'}
        action={() => setTab('posts')}
      >
        Posts
      </TabButton>
      <TabButton
        isActive={tab === 'contact'}
        action={() => setTab('contact')}
      >
        Contact
      </TabButton>
      <hr />
      {tab === 'about' && <AboutTab />}
      {tab === 'posts' && <PostsTab />}
      {tab === 'contact' && <ContactTab />}
    </Suspense>
  );
}
```

```js src/TabButton.js
export default function TabButton({ action, children, isActive }) {
  if (isActive) {
    return <b>{children}</b>
  }
  return (
    <button onClick={() => {
      action();
    }}>
      {children}
    </button>
  );
}
```

```js src/AboutTab.js hidden
export default function AboutTab() {
  return (
    <p>欢迎来到我的个人主页！</p>
  );
}
```

```js src/PostsTab.js hidden
import {use} from 'react';
import { fetchData } from './data.js';

function PostsTab() {
  const posts = use(fetchData('/posts'));
  return (
    <ul className="items">
      {posts.map(post =>
        <Post key={post.id} title={post.title} />
      )}
    </ul>
  );
}

function Post({ title }) {
  return (
    <li className="item">
      {title}
    </li>
  );
}

export default PostsTab;
```

```js src/ContactTab.js hidden
export default function ContactTab() {
  return (
    <>
      <p>
        你可以在这里找到我：
      </p>
      <ul>
        <li>admin@mysite.com</li>
        <li>+123456789</li>
      </ul>
    </>
  );
}
```


```js src/data.js hidden
// 注意：你如何进行数据获取取决于
// 你与 Suspense 一起使用的框架。
// 通常，缓存逻辑会在框架内部。

let cache = new Map();

export function fetchData(url) {
  if (!cache.has(url)) {
    cache.set(url, getData(url));
  }
  return cache.get(url);
}

async function getData(url) {
  if (url.startsWith('/posts')) {
    return await getPosts();
  } else {
    throw Error('Not implemented');
  }
}

async function getPosts() {
  // 增加一个假的延迟，让等待更明显。
  await new Promise(resolve => {
    setTimeout(resolve, 1000);
  });
  let posts = [];
  for (let i = 0; i < 500; i++) {
    posts.push({
      id: i,
      title: 'Post #' + (i + 1)
    });
  }
  return posts;
}
```

```css
button { margin-right: 10px }
b { display: inline-block; margin-right: 10px; }
.pending { color: #777; }
```

</Sandpack>

隐藏整个标签容器来显示加载指示器会让用户体验很突兀。如果你给 `TabButton` 添加 `useTransition`，就可以改为在标签按钮中显示 pending 状态。

注意，点击 “Posts” 不再会用一个 spinner 替换整个标签容器：

<Sandpack>

```js
import { Suspense, useState } from 'react';
import TabButton from './TabButton.js';
import AboutTab from './AboutTab.js';
import PostsTab from './PostsTab.js';
import ContactTab from './ContactTab.js';

export default function TabContainer() {
  const [tab, setTab] = useState('about');
  return (
    <Suspense fallback={<h1>🌀 正在加载...</h1>}>
      <TabButton
        isActive={tab === 'about'}
        action={() => setTab('about')}
      >
        About
      </TabButton>
      <TabButton
        isActive={tab === 'posts'}
        action={() => setTab('posts')}
      >
        Posts
      </TabButton>
      <TabButton
        isActive={tab === 'contact'}
        action={() => setTab('contact')}
      >
        Contact
      </TabButton>
      <hr />
      {tab === 'about' && <AboutTab />}
      {tab === 'posts' && <PostsTab />}
      {tab === 'contact' && <ContactTab />}
    </Suspense>
  );
}
```

```js src/TabButton.js active
import { useTransition } from 'react';

export default function TabButton({ action, children, isActive }) {
  const [isPending, startTransition] = useTransition();
  if (isActive) {
    return <b>{children}</b>
  }
  if (isPending) {
    return <b className="pending">{children}</b>;
  }
  return (
    <button onClick={() => {
      startTransition(async () => {
        await action();
      });
    }}>
      {children}
    </button>
  );
}
```

```js src/AboutTab.js hidden
export default function AboutTab() {
  return (
    <p>欢迎来到我的个人主页！</p>
  );
}
```

```js src/PostsTab.js hidden
import {use} from 'react';
import { fetchData } from './data.js';

function PostsTab() {
  const posts = use(fetchData('/posts'));
  return (
    <ul className="items">
      {posts.map(post =>
        <Post key={post.id} title={post.title} />
      )}
    </ul>
  );
}

function Post({ title }) {
  return (
    <li className="item">
      {title}
    </li>
  );
}

export default PostsTab;
```

```js src/ContactTab.js hidden
export default function ContactTab() {
  return (
    <>
      <p>
        你可以在这里找到我：
      </p>
      <ul>
        <li>admin@mysite.com</li>
        <li>+123456789</li>
      </ul>
    </>
  );
}
```


```js src/data.js hidden
// 注意：你如何进行数据获取取决于
// 你与 Suspense 一起使用的框架。
// 通常，缓存逻辑会在框架内部。

let cache = new Map();

export function fetchData(url) {
  if (!cache.has(url)) {
    cache.set(url, getData(url));
  }
  return cache.get(url);
}

async function getData(url) {
  if (url.startsWith('/posts')) {
    return await getPosts();
  } else {
    throw Error('Not implemented');
  }
}

async function getPosts() {
  // 增加一个假的延迟，让等待更明显。
  await new Promise(resolve => {
    setTimeout(resolve, 1000);
  });
  let posts = [];
  for (let i = 0; i < 500; i++) {
    posts.push({
      id: i,
      title: 'Post #' + (i + 1)
    });
  }
  return posts;
}
```

```css
button { margin-right: 10px }
b { display: inline-block; margin-right: 10px; }
.pending { color: #777; }
```

</Sandpack>

[阅读更多关于在 Suspense 中使用 Transitions 的内容。](/reference/react/Suspense#preventing-already-revealed-content-from-hiding)

<Note>

Transitions 只会“等待”足够长的时间，以避免隐藏*已经揭示*的内容（比如标签容器）。如果 Posts 标签有一个[嵌套的 `<Suspense>` 边界，](/reference/react/Suspense#revealing-nested-content-as-it-loads)那么 Transition 就不会“等待”它。

</Note>

---

### 构建一个支持 Suspense 的路由器 {/*building-a-suspense-enabled-router*/}

如果你正在构建一个 React 框架或路由器，我们建议将页面导航标记为 Transitions。

```js {3,6,8}
function Router() {
  const [page, setPage] = useState('/');
  const [isPending, startTransition] = useTransition();

  function navigate(url) {
    startTransition(() => {
      setPage(url);
    });
  }
  // ...
```

这样做有三个原因：

- [Transitions 是可中断的，](#perform-non-blocking-updates-with-actions)这让用户无需等待重新渲染完成就能切换出去。
- [Transitions 可防止不必要的加载指示器，](#preventing-unwanted-loading-indicators)这让用户在导航时避免突兀的跳转。
- [Transitions 会等待所有待处理的 actions](#perform-non-blocking-updates-with-actions)，这让用户在新页面显示前等待副作用完成。

下面是一个使用 Transitions 进行导航的简化路由器示例。

<Sandpack>

```js src/App.js
import { Suspense, useState, useTransition } from 'react';
import IndexPage from './IndexPage.js';
import ArtistPage from './ArtistPage.js';
import Layout from './Layout.js';

export default function App() {
  return (
    <Suspense fallback={<BigSpinner />}>
      <Router />
    </Suspense>
  );
}

function Router() {
  const [page, setPage] = useState('/');
  const [isPending, startTransition] = useTransition();

  function navigate(url) {
    startTransition(() => {
      setPage(url);
    });
  }

  let content;
  if (page === '/') {
    content = (
      <IndexPage navigate={navigate} />
    );
  } else if (page === '/the-beatles') {
    content = (
      <ArtistPage
        artist={{
          id: 'the-beatles',
          name: 'The Beatles',
        }}
      />
    );
  }
  return (
    <Layout isPending={isPending}>
      {content}
    </Layout>
  );
}

function BigSpinner() {
  return <h2>🌀 正在加载...</h2>;
}
```

```js src/Layout.js
export default function Layout({ children, isPending }) {
  return (
    <div className="layout">
      <section className="header" style={{
        opacity: isPending ? 0.7 : 1
      }}>
        Music Browser
      </section>
      <main>
        {children}
      </main>
    </div>
  );
}
```

```js src/IndexPage.js
export default function IndexPage({ navigate }) {
  return (
    <button onClick={() => navigate('/the-beatles')}>
      打开 The Beatles 艺术家页面
    </button>
  );
}
```

```js src/ArtistPage.js
import { Suspense } from 'react';
import Albums from './Albums.js';
import Biography from './Biography.js';
import Panel from './Panel.js';

export default function ArtistPage({ artist }) {
  return (
    <>
      <h1>{artist.name}</h1>
      <Biography artistId={artist.id} />
      <Suspense fallback={<AlbumsGlimmer />}>
        <Panel>
          <Albums artistId={artist.id} />
        </Panel>
      </Suspense>
    </>
  );
}

function AlbumsGlimmer() {
  return (
    <div className="glimmer-panel">
      <div className="glimmer-line" />
      <div className="glimmer-line" />
      <div className="glimmer-line" />
    </div>
  );
}
```

```js src/Albums.js
import {use} from 'react';
import { fetchData } from './data.js';

export default function Albums({ artistId }) {
  const albums = use(fetchData(`/${artistId}/albums`));
  return (
    <ul>
      {albums.map(album => (
        <li key={album.id}>
          {album.title} ({album.year})
        </li>
      ))}
    </ul>
  );
}
```

```js src/Biography.js
import {use} from 'react';
import { fetchData } from './data.js';

export default function Biography({ artistId }) {
  const bio = use(fetchData(`/${artistId}/bio`));
  return (
    <section>
      <p className="bio">{bio}</p>
    </section>
  );
}
```

```js src/Panel.js
export default function Panel({ children }) {
  return (
    <section className="panel">
      {children}
    </section>
  );
}
```

```js src/data.js hidden
// 注意：你如何进行数据获取取决于
// 你与 Suspense 一起使用的框架。
// 通常，缓存逻辑会在框架内部。

let cache = new Map();

export function fetchData(url) {
  if (!cache.has(url)) {
    cache.set(url, getData(url));
  }
  return cache.get(url);
}

async function getData(url) {
  if (url === '/the-beatles/albums') {
    return await getAlbums();
  } else if (url === '/the-beatles/bio') {
    return await getBio();
  } else {
    throw Error('Not implemented');
  }
}

async function getBio() {
  // 增加一个假的延迟，让等待更明显。
  await new Promise(resolve => {
    setTimeout(resolve, 500);
  });

  return `The Beatles 是一支英国摇滚乐队，
    成立于 1960 年的利物浦，由
    John Lennon、Paul McCartney、George Harrison
    和 Ringo Starr 组成。`;
}

async function getAlbums() {
  // 增加一个假的延迟，让等待更明显。
  await new Promise(resolve => {
    setTimeout(resolve, 3000);
  });

  return [{
    id: 13,
    title: 'Let It Be',
    year: 1970
  }, {
    id: 12,
    title: 'Abbey Road',
    year: 1969
  }, {
    id: 11,
    title: 'Yellow Submarine',
    year: 1969
  }, {
    id: 10,
    title: 'The Beatles',
    year: 1968
  }, {
    id: 9,
    title: 'Magical Mystery Tour',
    year: 1967
  }, {
    id: 8,
    title: 'Sgt. Pepper\'s Lonely Hearts Club Band',
    year: 1967
  }, {
    id: 7,
    title: 'Revolver',
    year: 1966
  }, {
    id: 6,
    title: 'Rubber Soul',
    year: 1965
  }, {
    id: 5,
    title: 'Help!',
    year: 1965
  }, {
    id: 4,
    title: 'Beatles For Sale',
    year: 1964
  }, {
    id: 3,
    title: 'A Hard Day\'s Night',
    year: 1964
  }, {
    id: 2,
    title: 'With The Beatles',
    year: 1963
  }, {
    id: 1,
    title: 'Please Please Me',
    year: 1963
  }];
}
```

```css
main {
  min-height: 200px;
  padding: 10px;
}

.layout {
  border: 1px solid black;
}

.header {
  background: #222;
  padding: 10px;
  text-align: center;
  color: white;
}

.bio { font-style: italic; }

.panel {
  border: 1px solid #aaa;
  border-radius: 6px;
  margin-top: 20px;
  padding: 10px;
}

.glimmer-panel {
  border: 1px dashed #aaa;
  background: linear-gradient(90deg, rgba(221,221,221,1) 0%, rgba(255,255,255,1) 100%);
  border-radius: 6px;
  margin-top: 20px;
  padding: 10px;
}

.glimmer-line {
  display: block;
  width: 60%;
  height: 20px;
  margin: 10px;
  border-radius: 4px;
  background: #f0f0f0;
}
```

</Sandpack>

<Note>

[支持 Suspense 的](/reference/react/Suspense)路由器预期会默认将导航更新包装为 Transitions。

</Note>

---

### 使用错误边界向用户显示错误 {/*displaying-an-error-to-users-with-error-boundary*/}

如果传递给 `startTransition` 的函数抛出错误，你可以使用 [error boundary](/reference/react/Component#catching-rendering-errors-with-an-error-boundary) 向用户显示错误。要使用 error boundary，请将调用 `useTransition` 的组件包裹在 error boundary 中。一旦传递给 `startTransition` 的函数出错，就会显示 error boundary 的 fallback。

<Sandpack>

```js src/AddCommentContainer.js active
import { useTransition } from "react";
import { ErrorBoundary } from "react-error-boundary";

export function AddCommentContainer() {
  return (
    <ErrorBoundary fallback={<p>⚠️ 出了点问题</p>}>
      <AddCommentButton />
    </ErrorBoundary>
  );
}

function addComment(comment) {
  // 为了演示以显示 Error Boundary
  if (comment == null) {
    throw new Error("示例错误：抛出一个错误以触发 error boundary");
  }
}

function AddCommentButton() {
  const [pending, startTransition] = useTransition();

  return (
    <button
      disabled={pending}
      onClick={() => {
        startTransition(() => {
          // 故意不传入 comment
          // 这样就会抛出错误
          addComment();
        });
      }}
    >
      添加评论
    </button>
  );
}
```

```js src/App.js hidden
import { AddCommentContainer } from "./AddCommentContainer.js";

export default function App() {
  return <AddCommentContainer />;
}
```

```js src/index.js hidden
import React, { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import './styles.css';
import App from './App';

const root = createRoot(document.getElementById('root'));
root.render(
  <StrictMode>
    <App />
  </StrictMode>
);
```

```json package.json hidden
{
  "dependencies": {
    "react": "19.0.0-rc-3edc000d-20240926",
    "react-dom": "19.0.0-rc-3edc000d-20240926",
    "react-scripts": "^5.0.0",
    "react-error-boundary": "4.0.3"
  },
  "main": "/index.js"
}
```
</Sandpack>

---

## 故障排查 {/*troubleshooting*/}

### 在 Transition 中更新输入框不起作用 {/*updating-an-input-in-a-transition-doesnt-work*/}

你不能对控制输入框的状态变量使用 Transition：

```js {4,10}
const [text, setText] = useState('');
// ...
function handleChange(e) {
  // ❌ 不能将 Transitions 用于受控输入的状态
  startTransition(() => {
    setText(e.target.value);
  });
}
// ...
return <input value={text} onChange={handleChange} />;
```

这是因为 Transitions 是非阻塞的，但响应 change 事件来更新输入框应该同步发生。如果你想在输入时运行 Transition，有两种选择：

1. 你可以声明两个独立的状态变量：一个用于输入框状态（始终同步更新），另一个用于在 Transition 中更新的状态。这样你就可以使用同步状态来控制输入框，并将 Transition 状态变量（它会“落后于”输入）传递给其余渲染逻辑。
2. 另外，你也可以只使用一个状态变量，并添加 [`useDeferredValue`](/reference/react/useDeferredValue)，它会“落后于”真实值。它会自动触发非阻塞重渲染来“追上”新值。

---

### React 没有把我的状态更新视为 Transition {/*react-doesnt-treat-my-state-update-as-a-transition*/}

当你把状态更新包裹在 Transition 中时，请确保它发生在 `startTransition` 调用*期间*：

```js
startTransition(() => {
  // ✅ 在 startTransition 调用*期间*设置状态
  setPage('/about');
});
```

你传给 `startTransition` 的函数必须是同步的。你不能像这样把更新标记为 Transition：

```js
startTransition(() => {
  // ❌ 在 startTransition 调用*之后*设置状态
  setTimeout(() => {
    setPage('/about');
  }, 1000);
});
```

相反，你可以这样做：

```js
setTimeout(() => {
  startTransition(() => {
    // ✅ 在 startTransition 调用*期间*设置状态
    setPage('/about');
  });
}, 1000);
```

---

### React 没有把 `await` 之后的状态更新视为 Transition {/*react-doesnt-treat-my-state-update-after-await-as-a-transition*/}

当你在 `startTransition` 函数内部使用 `await` 时，发生在 `await` 之后的状态更新不会被标记为 Transitions。你必须在每个 `await` 之后使用 `startTransition` 调用来包裹状态更新：

```js
startTransition(async () => {
  await someAsyncFunction();
  // ❌ 在 await 之后没有使用 startTransition
  setPage('/about');
});
```

不过，下面这样才可以：

```js
startTransition(async () => {
  await someAsyncFunction();
  // ✅ 在 await *之后* 使用 startTransition
  startTransition(() => {
    setPage('/about');
  });
});
```

这是一个 JavaScript 限制，因为 React 会丢失 async 上下文的作用域。未来当 [AsyncContext](https://github.com/tc39/proposal-async-context) 可用时，这一限制将被移除。

---

### 我想在组件外部调用 `useTransition` {/*i-want-to-call-usetransition-from-outside-a-component*/}

你不能在组件外部调用 `useTransition`，因为它是一个 Hook。在这种情况下，请改用独立的 [`startTransition`](/reference/react/startTransition) 方法。它的工作方式相同，但不会提供 `isPending` 指示器。

---

### 我传给 `startTransition` 的函数会立即执行 {/*the-function-i-pass-to-starttransition-executes-immediately*/}

如果你运行这段代码，它会打印 1、2、3：

```js {1,3,6}
console.log(1);
startTransition(() => {
  console.log(2);
  setPage('/about');
});
console.log(3);
```

**它预期会打印 1、2、3。** 你传给 `startTransition` 的函数不会被延迟。与浏览器的 `setTimeout` 不同，它不会稍后再运行回调。React 会立即执行你的函数，但任何在其*运行期间*调度的状态更新都会被标记为 Transitions。你可以把它想象成这样：

```js
// React 工作方式的简化版本

let isInsideTransition = false;

function startTransition(scope) {
  isInsideTransition = true;
  scope();
  isInsideTransition = false;
}

function setState() {
  if (isInsideTransition) {
    // ... 调度一个 Transition 状态更新 ...
  } else {
    // ... 调度一个紧急状态更新 ...
  }
}
```

### 我在 Transitions 中的状态更新顺序乱了 {/*my-state-updates-in-transitions-are-out-of-order*/}

如果你在 `startTransition` 中使用 `await`，你可能会看到更新顺序乱掉。

在这个示例中，`updateQuantity` 函数模拟了向服务器请求更新购物车中商品数量的过程。这个函数*人为地让每隔一个请求都比上一个更晚返回*，以模拟网络请求的竞态条件。

试着更新一次数量，然后快速多次更新。你可能会看到错误的总价：

<Sandpack>

```json package.json hidden
{
  "dependencies": {
    "react": "beta",
    "react-dom": "beta"
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
import { useState, useTransition } from "react";
import { updateQuantity } from "./api";
import Item from "./Item";
import Total from "./Total";

export default function App({}) {
  const [quantity, setQuantity] = useState(1);
  const [isPending, startTransition] = useTransition();
  // 将实际数量存储在单独的状态中，以显示不匹配。
  const [clientQuantity, setClientQuantity] = useState(1);

  const updateQuantityAction = newQuantity => {
    setClientQuantity(newQuantity);

    // 通过再次包裹在 startTransition 中，
    // 访问该 Transition 的 pending 状态。
    startTransition(async () => {
      const savedQuantity = await updateQuantity(newQuantity);
      startTransition(() => {
        setQuantity(savedQuantity);
      });
    });
  };

  return (
    <div>
      <h1>Checkout</h1>
      <Item action={updateQuantityAction}/>
      <hr />
      <Total clientQuantity={clientQuantity} savedQuantity={quantity} isPending={isPending} />
    </div>
  );
}

```

```js src/Item.js
import {startTransition} from 'react';

export default function Item({action}) {
  function handleChange(e) {
    // 在一个 Action 中更新数量。
    startTransition(async () => {
      await action(e.target.value);
    });
  }
  return (
    <div className="item">
      <span>Eras Tour Tickets</span>
      <label htmlFor="name">Quantity: </label>
      <input
        type="number"
        onChange={handleChange}
        defaultValue={1}
        min={1}
      />
    </div>
  )
}
```

```js src/Total.js
const intl = new Intl.NumberFormat("en-US", {
  style: "currency",
  currency: "USD"
});

export default function Total({ clientQuantity, savedQuantity, isPending }) {
  return (
    <div className="total">
      <span>Total:</span>
      <div>
        <div>
          {isPending
            ? "🌀 Updating..."
            : `${intl.format(savedQuantity * 9999)}`}
        </div>
        <div className="error">
          {!isPending &&
            clientQuantity !== savedQuantity &&
            `Wrong total, expected: ${intl.format(clientQuantity * 9999)}`}
        </div>
      </div>
    </div>
  );
}
```

```js src/api.js
let firstRequest = true;
export async function updateQuantity(newName) {
  return new Promise((resolve, reject) => {
    if (firstRequest === true) {
      firstRequest = false;
      setTimeout(() => {
        firstRequest = true;
        resolve(newName);
        // 模拟每隔一个请求都更慢
      }, 1000);
    } else {
      setTimeout(() => {
        resolve(newName);
      }, 50);
    }
  });
}
```

```css
.item {
  display: flex;
  align-items: center;
  justify-content: start;
}

.item label {
  flex: 1;
  text-align: right;
}

.item input {
  margin-left: 4px;
  width: 60px;
  padding: 4px;
}

.total {
  height: 50px;
  line-height: 25px;
  display: flex;
  align-content: center;
  justify-content: space-between;
}

.total div {
  display: flex;
  flex-direction: column;
  align-items: flex-end;
}

.error {
  color: red;
}
```

</Sandpack>


当多次点击时，先前的请求有可能在后来的请求之后完成。当这种情况发生时，React 目前没有办法知道期望的顺序。这是因为更新是异步调度的，而 React 会在异步边界之间丢失顺序上下文。

这是预期行为，因为 Transition 中的 Action 不保证执行顺序。对于常见用例，React 提供了更高层的抽象，例如 [`useActionState`](/reference/react/useActionState) 和 [`<form>` actions](/reference/react-dom/components/form)，它们会帮你处理顺序。对于高级用例，你需要自己实现排队和中止逻辑来处理这一点。


`useActionState` 处理执行顺序的示例：

<Sandpack>

```json package.json hidden
{
  "dependencies": {
    "react": "beta",
    "react-dom": "beta"
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
import { useState, useActionState } from "react";
import { updateQuantity } from "./api";
import Item from "./Item";
import Total from "./Total";

export default function App({}) {
  // 将实际数量存储在单独的状态中，以显示不匹配。
  const [clientQuantity, setClientQuantity] = useState(1);
  const [quantity, updateQuantityAction, isPending] = useActionState(
    async (prevState, payload) => {
      setClientQuantity(payload);
      const savedQuantity = await updateQuantity(payload);
      return savedQuantity; // 返回新数量以更新状态
    },
    1 // 初始数量
  );

  return (
    <div>
      <h1>Checkout</h1>
      <Item action={updateQuantityAction}/>
      <hr />
      <Total clientQuantity={clientQuantity} savedQuantity={quantity} isPending={isPending} />
    </div>
  );
}

```

```js src/Item.js
import {startTransition} from 'react';

export default function Item({action}) {
  function handleChange(e) {
    // 在一个 Action 中更新数量。
    startTransition(() => {
      action(e.target.value);
    });
  }
  return (
    <div className="item">
      <span>Eras Tour Tickets</span>
      <label htmlFor="name">Quantity: </label>
      <input
        type="number"
        onChange={handleChange}
        defaultValue={1}
        min={1}
      />
    </div>
  )
}
```

```js src/Total.js
const intl = new Intl.NumberFormat("en-US", {
  style: "currency",
  currency: "USD"
});

export default function Total({ clientQuantity, savedQuantity, isPending }) {
  return (
    <div className="total">
      <span>Total:</span>
      <div>
        <div>
          {isPending
            ? "🌀 Updating..."
            : `${intl.format(savedQuantity * 9999)}`}
        </div>
        <div className="error">
          {!isPending &&
            clientQuantity !== savedQuantity &&
            `Wrong total, expected: ${intl.format(clientQuantity * 9999)}`}
        </div>
      </div>
    </div>
  );
}
```

```js src/api.js
let firstRequest = true;
export async function updateQuantity(newName) {
  return new Promise((resolve, reject) => {
    if (firstRequest === true) {
      firstRequest = false;
      setTimeout(() => {
        firstRequest = true;
        resolve(newName);
        // 模拟每隔一个请求都更慢
      }, 1000);
    } else {
      setTimeout(() => {
        resolve(newName);
      }, 50);
    }
  });
}
```

```css
.item {
  display: flex;
  align-items: center;
  justify-content: start;
}

.item label {
  flex: 1;
  text-align: right;
}

.item input {
  margin-left: 4px;
  width: 60px;
  padding: 4px;
}

.total {
  height: 50px;
  line-height: 25px;
  display: flex;
  align-content: center;
  justify-content: space-between;
}

.total div {
  display: flex;
  flex-direction: column;
  align-items: flex-end;
}

.error {
  color: red;
}
```

</Sandpack>
