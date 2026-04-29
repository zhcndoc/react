---
title: useOptimistic
---

<Intro>

`useOptimistic` 是一个 React Hook，允许你以乐观方式更新 UI。

```js
const [optimisticState, setOptimistic] = useOptimistic(value, reducer?);
```

</Intro>

<InlineToc />

---

## 参考 {/*reference*/}

### `useOptimistic(value, reducer?)` {/*useoptimistic*/}

在组件顶层调用 `useOptimistic` 来为某个值创建乐观状态。

```js
import { useOptimistic } from 'react';

function MyComponent({name, todos}) {
  const [optimisticAge, setOptimisticAge] = useOptimistic(28);
  const [optimisticName, setOptimisticName] = useOptimistic(name);
  const [optimisticTodos, setOptimisticTodos] = useOptimistic(todos, todoReducer);
  // ...
}
```

[查看更多示例。](#usage)

#### 参数 {/*parameters*/}

* `value`：当没有待处理的 Action 时返回的值。
* **可选** `reducer(currentState, action)`：指定乐观状态如何更新的 reducer 函数。它必须是纯函数，应接收当前状态和 reducer action 参数，并返回下一个乐观状态。

#### 返回值 {/*returns*/}

`useOptimistic` 返回一个恰好包含两个值的数组：

1. `optimisticState`：当前的乐观状态。它等于 `value`，除非有 Action 处于待处理状态；在这种情况下，它等于 `reducer` 返回的状态（如果没有提供 `reducer`，则等于传给 set 函数的值）。
2. [`set` 函数](#setoptimistic)，允许你在 Action 内部将乐观状态更新为不同的值。

---

### `set` 函数，例如 `setOptimistic(optimisticState)` {/*setoptimistic*/}

`useOptimistic` 返回的 `set` 函数允许你在 [Action](reference/react/useTransition#functions-called-in-starttransition-are-called-actions) 持续期间更新状态。你可以直接传入下一个状态，或者传入一个根据前一个状态计算结果的函数：

```js
const [optimisticLike, setOptimisticLike] = useOptimistic(false);
const [optimisticSubs, setOptimisticSubs] = useOptimistic(subs);

function handleClick() {
  startTransition(async () => {
    setOptimisticLike(true);
    setOptimisticSubs(a => a + 1);
    await saveChanges();
  });
}
```

#### 参数 {/*setoptimistic-parameters*/}

* `optimisticState`：你希望在 [Action](reference/react/useTransition#functions-called-in-starttransition-are-called-actions) 期间乐观状态所处的值。如果你为 `useOptimistic` 提供了 `reducer`，这个值将作为第二个参数传给 reducer。它可以是任意类型的值。
    * 如果你把函数作为 `optimisticState` 传入，它会被视为一个 _updater function_。它必须是纯函数，应将待处理状态作为唯一参数，并返回下一个乐观状态。React 会将你的 updater function 放入队列并重新渲染组件。在下一次渲染期间，React 会通过将队列中的 updaters 应用到前一个状态来计算下一个状态，这与 [`useState` updaters](/reference/react/useState#setstate-parameters) 类似。

#### 返回值 {/*setoptimistic-returns*/}

`set` 函数没有返回值。

#### 注意事项 {/*setoptimistic-caveats*/}

* `set` 函数必须在 [Action](reference/react/useTransition#functions-called-in-starttransition-are-called-actions) 内部调用。如果你在 Action 之外调用 setter，[React 会显示警告](#an-optimistic-state-update-occurred-outside-a-transition-or-action)，并且乐观状态会短暂渲染。

<DeepDive>

#### 乐观状态如何工作 {/*how-optimistic-state-works*/}

`useOptimistic` 允许你在 Action 执行期间显示一个临时值：

```js
const [value, setValue] = useState('a');
const [optimistic, setOptimistic] = useOptimistic(value);

startTransition(async () => {
  setOptimistic('b');
  const newValue = await saveChanges('b');
  setValue(newValue);
});
```

当 setter 在 Action 内部被调用时，`useOptimistic` 会触发一次重新渲染，在 Action 执行期间显示该状态。否则，将返回传给 `useOptimistic` 的 `value`。

这个状态之所以称为“optimistic（乐观）”，是因为它会立即向用户展示执行某个 Action 的结果，即使这个 Action 实际上需要一些时间才能完成。

**更新流程如何进行**

1. **立即更新**：当调用 `setOptimistic('b')` 时，React 会立即使用 `'b'` 进行渲染。

2. **（可选）在 Action 中等待**：如果你在 Action 中执行 await，React 会继续显示 `'b'`。

3. **安排 Transition**：`setValue(newValue)` 会安排对真实状态的更新。

4. **（可选）等待 Suspense**：如果 `newValue` 触发暂停，React 会继续显示 `'b'`。

5. **单次渲染提交**：最后，`newValue` 会同时作为 `value` 和 `optimistic` 提交。

不会有额外的渲染来“清除”乐观状态。当 Transition 完成时，乐观状态和真实状态会在同一次渲染中收敛。

<Note>

#### 乐观状态是临时的 {/*optimistic-state-is-temporary*/}

乐观状态只会在 Action 执行期间渲染，否则会渲染 `value`。

如果 `saveChanges` 返回 `'c'`，那么 `value` 和 `optimistic` 都将是 `'c'`，而不是 `'b'`。

</Note>

**最终状态如何确定**

传给 `useOptimistic` 的 `value` 参数决定了 Action 完成后显示什么。这具体取决于你使用的模式：

- **硬编码值**，例如 `useOptimistic(false)`：Action 结束后，`state` 仍然是 `false`，因此 UI 显示 `false`。这适用于始终从 `false` 开始的待处理状态。

- **传入的 props 或 state**，例如 `useOptimistic(isLiked)`：如果父组件在 Action 期间更新了 `isLiked`，那么 Action 完成后会使用新值。这就是 UI 反映 Action 结果的方式。

- **reducer 模式**，例如 `useOptimistic(items, fn)`：如果在 Action 待处理期间 `items` 改变了，React 会使用新的 `items` 重新运行你的 `reducer` 来重新计算状态。这样可以让你的乐观增量叠加在最新数据之上。

**Action 失败时会发生什么**

如果 Action 抛出错误，Transition 仍然会结束，而 React 会使用当前的 `value` 进行渲染。由于父组件通常只在成功时更新 `value`，失败意味着 `value` 没有变化，所以 UI 会显示乐观更新之前的内容。你可以捕获错误来向用户显示消息。

</DeepDive>

---

## 用法 {/*usage*/}

### 向组件添加乐观状态 {/*adding-optimistic-state-to-a-component*/}

在组件顶层调用 `useOptimistic` 来声明一个或多个乐观状态。

```js [[1, 4, "age"], [1, 5, "name"], [1, 6, "todos"], [2, 4, "optimisticAge"], [2, 5, "optimisticName"], [2, 6, "optimisticTodos"], [3, 4, "setOptimisticAge"], [3, 5, "setOptimisticName"], [3, 6, "setOptimisticTodos"], [4, 6, "reducer"]]
import { useOptimistic } from 'react';

function MyComponent({age, name, todos}) {
  const [optimisticAge, setOptimisticAge] = useOptimistic(age);
  const [optimisticName, setOptimisticName] = useOptimistic(name);
  const [optimisticTodos, setOptimisticTodos] = useOptimistic(todos, reducer);
  // ...
```

`useOptimistic` 恰好返回两个项：

1. <CodeStep step={2}>乐观状态</CodeStep>，初始设置为提供的 <CodeStep step={1}>值</CodeStep>。
2. <CodeStep step={3}>set 函数</CodeStep>，允许你在 [Action](reference/react/useTransition#functions-called-in-starttransition-are-called-actions) 期间临时更改状态。
   * 如果提供了 <CodeStep step={4}>reducer</CodeStep>，它会在返回乐观状态之前运行。

要使用 <CodeStep step={2}>乐观状态</CodeStep>，请在 Action 内部调用 `set` 函数。

Action 是在 `startTransition` 内部调用的函数：

```js {3}
function onAgeChange(e) {
  startTransition(async () => {
    setOptimisticAge(42);
    const newAge = await postAge(42);
    setAge(newAge);
  });
}
```

React 会先渲染乐观状态 `42`，同时 `age` 仍然是当前年龄。Action 会等待 POST，然后再为 `age` 和 `optimisticAge` 同时渲染 `newAge`。

深入了解请参见 [乐观状态如何工作](#how-optimistic-state-works)。

<Note>

使用 [Action props](/reference/react/useTransition#exposing-action-props-from-components) 时，你可以不使用 `startTransition` 直接调用 set 函数：

```js [[3, 2, "setOptimisticName"]]
async function submitAction() {
  setOptimisticName('Taylor');
  await updateName('Taylor');
}
```

这是因为 Action props 本身就是在 `startTransition` 内部被调用的。

示例请参见：[在 Action props 中使用乐观状态](#using-optimistic-state-in-action-props)。

</Note>

---

### 在 Action props 中使用乐观状态 {/*using-optimistic-state-in-action-props*/}

在 [Action prop](/reference/react/useTransition#exposing-action-props-from-components) 中，你可以直接调用乐观 setter，而不需要 `startTransition`。

这个示例在 `<form>` 的 `submitAction` prop 内设置乐观状态：

<Sandpack>

```js src/App.js
import { useState, startTransition } from 'react';
import EditName from './EditName';

export default function App() {
  const [name, setName] = useState('Alice');

  return <EditName name={name} action={setName} />;
}
```

```js src/EditName.js active
import { useOptimistic, startTransition } from 'react';
import { updateName } from './actions.js';

export default function EditName({ name, action }) {
  const [optimisticName, setOptimisticName] = useOptimistic(name);

  async function submitAction(formData) {
    const newName = formData.get('name');
    setOptimisticName(newName);

    const updatedName = await updateName(newName);
    startTransition(() => {
      action(updatedName);
    })
  }

  return (
    <form action={submitAction}>
      <p>Your name is: {optimisticName}</p>
      <p>
        <label>Change it: </label>
        <input
          type="text"
          name="name"
          disabled={name !== optimisticName}
        />
      </p>
    </form>
  );
}
```

```js src/actions.js hidden
export async function updateName(name) {
  await new Promise((res) => setTimeout(res, 1000));
  return name;
}
```

</Sandpack>

在这个示例中，当用户提交表单时，`optimisticName` 会立即更新，在服务器请求进行期间以乐观方式显示 `newName`。当请求完成后，`name` 和 `optimisticName` 会使用响应中的实际 `updatedName` 进行渲染。

<DeepDive>

#### 为什么这里不需要 `startTransition`？ {/*why-doesnt-this-need-starttransition*/}

按照惯例，在 `startTransition` 内部调用的 props 会命名为 “Action”。

由于 `submitAction` 的命名包含 “Action”，你就知道它本身已经是在 `startTransition` 内部被调用的。

有关 Action prop 模式，请参见 [从组件中暴露 `action` prop](/reference/react/useTransition#exposing-action-props-from-components)。

</DeepDive>

---

### 向 Action props 添加乐观状态 {/*adding-optimistic-state-to-action-props*/}

在创建 [Action prop](/reference/react/useTransition#exposing-action-props-from-components) 时，你可以添加 `useOptimistic` 来显示即时反馈。

这里有一个按钮，会在 `action` 处于待处理状态时显示“Submitting...”：

<Sandpack>

```js src/App.js
import { useState, startTransition } from 'react';
import Button from './Button';
import { submitForm } from './actions.js';

export default function App() {
  const [count, setCount] = useState(0);
  return (
    <div>
      <Button action={async () => {
        await submitForm();
        startTransition(() => {
          setCount(c => c + 1);
        });
      }}>Increment</Button>
      {count > 0 && <p>Submitted {count}!</p>}
    </div>
  );
}
```

```js src/Button.js active
import { useOptimistic, startTransition } from 'react';

export default function Button({ action, children }) {
  const [isPending, setIsPending] = useOptimistic(false);

  return (
    <button
      disabled={isPending}
      onClick={() => {
        startTransition(async () => {
          setIsPending(true);
          await action();
        });
      }}
    >
      {isPending ? 'Submitting...' : children}
    </button>
  );
}
```

```js src/actions.js hidden
export async function submitForm() {
  await new Promise((res) => setTimeout(res, 1000));
}
```

</Sandpack>

当按钮被点击时，`setIsPending(true)` 会使用乐观状态立即显示“Submitting...”并禁用按钮。当 Action 完成时，`isPending` 会自动渲染为 `false`。

无论 `action` prop 如何与 `Button` 一起使用，这种模式都会自动显示待处理状态：

```js
// 显示状态更新的待处理状态
<Button action={() => { setState(c => c + 1) }} />

// 显示导航的待处理状态
<Button action={() => { navigate('/done') }} />

// 显示 POST 的待处理状态
<Button action={async () => { await fetch(/* ... */) }} />

// 显示任意组合的待处理状态
<Button action={async () => {
  setState(c => c + 1);
  await fetch(/* ... */);
  navigate('/done');
}} />
```

待处理状态会一直显示，直到 `action` prop 中的所有内容都完成。

<Note>

你也可以使用 [`useTransition`](/reference/react/useTransition) 通过 `isPending` 获取待处理状态。

区别在于，`useTransition` 会提供 `startTransition` 函数，而 `useOptimistic` 可与任何 Transition 配合使用。请选择最适合你组件需求的方案。

</Note>

---

### 乐观地更新 props 或 state {/*updating-props-or-state-optimistically*/}

你可以将 props 或 state 包装进 `useOptimistic`，以便在 Action 执行期间立即更新它。

在这个示例中，`LikeButton` 接收 `isLiked` 作为 prop，并在点击时立即切换它：

<Sandpack>

```js src/App.js
import { useState, useOptimistic, startTransition } from 'react';
import { toggleLike } from './actions.js';

export default function App() {
  const [isLiked, setIsLiked] = useState(false);
  const [optimisticIsLiked, setOptimisticIsLiked] = useOptimistic(isLiked);

  function handleClick() {
    startTransition(async () => {
      const newValue = !optimisticIsLiked
      console.log('⏳ setting optimistic state: ' + newValue);

      setOptimisticIsLiked(newValue);
      const updatedValue = await toggleLike(newValue);

      startTransition(() => {
        console.log('⏳ setting real state: ' + updatedValue );
        setIsLiked(updatedValue);
      });
    });
  }

  if (optimisticIsLiked !== isLiked) {
    console.log('✅ rendering optimistic state: ' + optimisticIsLiked);
  } else {
    console.log('✅ rendering real value: ' + optimisticIsLiked);
  }


  return (
    <button onClick={handleClick}>
      {optimisticIsLiked ? '❤️ Unlike' : '🤍 Like'}
    </button>
  );
}
```

```js src/actions.js hidden
export async function toggleLike(value) {
  return await new Promise((res) => setTimeout(() => res(value), 1000));
  // 在真实应用中，这会更新服务器
}
```

```js src/index.js hidden
import React from 'react';
import {createRoot} from 'react-dom/client';
import './styles.css';

import App from './App';

const root = createRoot(document.getElementById('root'));
// 未使用 StrictMode，因此不会显示双重渲染日志。
root.render(<App />);
```

</Sandpack>

当按钮被点击时，`setOptimisticIsLiked` 会立即更新显示的状态，把心形显示为已点赞。与此同时，`await toggleLike` 在后台运行。当 `await` 完成后，`setIsLiked` 父组件更新 “真实” 的 `isLiked` 状态，随后乐观状态会按这个新值进行渲染。

<Note>

这个示例读取 `optimisticIsLiked` 来计算下一个值。当基础状态不会变化时，这种方式有效；但如果在 Action 处于待处理状态时基础状态可能发生变化，你可能更适合使用 state updater 或 reducer。

示例请参见 [基于当前状态更新 state](#updating-state-based-on-current-state)。

</Note>

---

### 同时更新多个值 {/*updating-multiple-values-together*/}

当一次乐观更新影响多个相关值时，请使用 reducer 将它们一起更新。这样可以确保 UI 保持一致。

这里有一个关注按钮，同时更新关注状态和粉丝数量：

<Sandpack>

```js src/App.js
import { useState, startTransition } from 'react';
import { followUser, unfollowUser } from './actions.js';
import FollowButton from './FollowButton';

export default function App() {
  const [user, setUser] = useState({
    name: 'React',
    isFollowing: false,
    followerCount: 10500
  });

  async function followAction(shouldFollow) {
    if (shouldFollow) {
      await followUser(user.name);
    } else {
      await unfollowUser(user.name);
    }
    startTransition(() => {
      setUser(current => ({
        ...current,
        isFollowing: shouldFollow,
        followerCount: current.followerCount + (shouldFollow ? 1 : -1)
      }));
    });
  }

  return <FollowButton user={user} followAction={followAction} />;
}
```

```js src/FollowButton.js active
import { useOptimistic, startTransition } from 'react';

export default function FollowButton({ user, followAction }) {
  const [optimisticState, updateOptimistic] = useOptimistic(
    { isFollowing: user.isFollowing, followerCount: user.followerCount },
    (current, isFollowing) => ({
      isFollowing,
      followerCount: current.followerCount + (isFollowing ? 1 : -1)
    })
  );

  function handleClick() {
    const newFollowState = !optimisticState.isFollowing;
    startTransition(async () => {
      updateOptimistic(newFollowState);
      await followAction(newFollowState);
    });
  }

  return (
    <div>
      <p><strong>{user.name}</strong></p>
      <p>{optimisticState.followerCount} followers</p>
      <button onClick={handleClick}>
        {optimisticState.isFollowing ? 'Unfollow' : 'Follow'}
      </button>
    </div>
  );
}
```

```js src/actions.js hidden
export async function followUser(name) {
  await new Promise((res) => setTimeout(res, 1000));
}

export async function unfollowUser(name) {
  await new Promise((res) => setTimeout(res, 1000));
}
```

</Sandpack>

reducer 接收新的 `isFollowing` 值，并在一次更新中同时计算新的关注状态和更新后的粉丝数量。这样可以确保按钮文字和数量始终保持同步。


<DeepDive>

#### 在 updaters 和 reducers 之间选择 {/*choosing-between-updaters-and-reducers*/}

`useOptimistic` 支持两种基于当前状态计算状态的模式：

**Updater 函数** 的工作方式类似于 [useState updaters](/reference/react/useState#updating-state-based-on-the-previous-state)。向 setter 传入一个函数：

```js
const [optimistic, setOptimistic] = useOptimistic(value);
setOptimistic(current => !current);
```

**Reducer** 将更新逻辑与 setter 调用分离：

```js
const [optimistic, dispatch] = useOptimistic(value, (current, action) => {
  // 根据 current 和 action 计算下一个状态
});
dispatch(action);
```

**使用 updater** 适合那些 setter 调用本身就能自然描述更新的计算。这和在 `useState` 中使用 `setState(prev => ...)` 类似。

**使用 reducer** 适合需要向更新传递数据（例如要添加哪个项目），或者需要用一个 hook 处理多种更新类型的情况。

**为什么要使用 reducer？**

当基础状态可能在 Transition 处于待处理期间发生变化时，reducer 至关重要。如果 `todos` 在你的添加操作等待期间发生变化（例如，另一个用户添加了一个 todo），React 会使用新的 `todos` 重新运行你的 reducer 来重新计算要显示的内容。这样可以确保新 todo 被添加到最新列表，而不是过时副本。

像 `setOptimistic(prev => [...prev, newItem])` 这样的 updater 函数，只会看到 Transition 开始时的状态，而看不到异步工作期间发生的任何更新。

</DeepDive>

---

### 乐观地向列表中添加内容 {/*optimistically-adding-to-a-list*/}

当你需要以乐观方式向列表中添加项目时，请使用 `reducer`：

<Sandpack>

```js src/App.js
import { useState, startTransition } from 'react';
import { addTodo } from './actions.js';
import TodoList from './TodoList';

export default function App() {
  const [todos, setTodos] = useState([
    { id: 1, text: 'Learn React' }
  ]);

  async function addTodoAction(newTodo) {
    const savedTodo = await addTodo(newTodo);
    startTransition(() => {
      setTodos(todos => [...todos, savedTodo]);
    });
  }

  return <TodoList todos={todos} addTodoAction={addTodoAction} />;
}
```

```js src/TodoList.js active
import { useOptimistic, startTransition } from 'react';

export default function TodoList({ todos, addTodoAction }) {
  const [optimisticTodos, addOptimisticTodo] = useOptimistic(
    todos,
    (currentTodos, newTodo) => [
      ...currentTodos,
      { id: newTodo.id, text: newTodo.text, pending: true }
    ]
  );

  function handleAddTodo(text) {
    const newTodo = { id: crypto.randomUUID(), text: text };
    startTransition(async () => {
      addOptimisticTodo(newTodo);
      await addTodoAction(newTodo);
    });
  }

  return (
    <div>
      <button onClick={() => handleAddTodo('New todo')}>Add Todo</button>
      <ul>
        {optimisticTodos.map(todo => (
          <li key={todo.id}>
            {todo.text} {todo.pending && "(Adding...)"}
          </li>
        ))}
      </ul>
    </div>
  );
}
```

```js src/actions.js hidden
export async function addTodo(todo) {
  await new Promise((res) => setTimeout(res, 1000));
  // 在真实应用中，这会保存到服务器
  return { ...todo, pending: false };
}
```

</Sandpack>

`reducer` 接收当前 todo 列表和要添加的新 todo。这一点很重要，因为如果在添加操作待处理期间 `todos` prop 发生变化（例如，另一个用户添加了一个 todo），React 会通过使用更新后的列表重新运行 reducer 来更新你的乐观状态。这样可以确保新 todo 被添加到最新列表，而不是过时副本。

<Note>

每个乐观项都包含一个 `pending: true` 标记，因此你可以为单个项目显示加载状态。当服务器响应并且父组件使用已保存的项目更新标准 `todos` 列表时，乐观状态会更新为已确认的项目，并移除 pending 标记。

</Note>

---

### 处理多种 `action` 类型 {/*handling-multiple-action-types*/}

当你需要处理多种乐观更新类型（例如添加和删除项目）时，请使用带有 `action` 对象的 reducer 模式。

这个购物车示例展示了如何使用单个 reducer 处理添加和删除：

<Sandpack>

```js src/App.js
import { useState, startTransition } from 'react';
import { addToCart, removeFromCart, updateQuantity } from './actions.js';
import ShoppingCart from './ShoppingCart';

export default function App() {
  const [cart, setCart] = useState([]);

  const cartActions = {
    async add(item) {
      await addToCart(item);
      startTransition(() => {
        setCart(current => {
          const exists = current.find(i => i.id === item.id);
          if (exists) {
            return current.map(i =>
              i.id === item.id ? { ...i, quantity: i.quantity + 1 } : i
            );
          }
          return [...current, { ...item, quantity: 1 }];
        });
      });
    },
    async remove(id) {
      await removeFromCart(id);
      startTransition(() => {
        setCart(current => current.filter(item => item.id !== id));
      });
    },
    async updateQuantity(id, quantity) {
      await updateQuantity(id, quantity);
      startTransition(() => {
        setCart(current =>
          current.map(item =>
            item.id === id ? { ...item, quantity } : item
          )
        );
      });
    }
  };

  return <ShoppingCart cart={cart} cartActions={cartActions} />;
}
```

```js src/ShoppingCart.js active
import { useOptimistic, startTransition } from 'react';

export default function ShoppingCart({ cart, cartActions }) {
  const [optimisticCart, dispatch] = useOptimistic(
    cart,
    (currentCart, action) => {
      switch (action.type) {
        case 'add':
          const exists = currentCart.find(item => item.id === action.item.id);
          if (exists) {
            return currentCart.map(item =>
              item.id === action.item.id
                ? { ...item, quantity: item.quantity + 1, pending: true }
                : item
            );
          }
          return [...currentCart, { ...action.item, quantity: 1, pending: true }];
        case 'remove':
          return currentCart.filter(item => item.id !== action.id);
        case 'update_quantity':
          return currentCart.map(item =>
            item.id === action.id
              ? { ...item, quantity: action.quantity, pending: true }
              : item
          );
        default:
          return currentCart;
      }
    }
  );

  function handleAdd(item) {
    startTransition(async () => {
      dispatch({ type: 'add', item });
      await cartActions.add(item);
    });
  }

  function handleRemove(id) {
    startTransition(async () => {
      dispatch({ type: 'remove', id });
      await cartActions.remove(id);
    });
  }

  function handleUpdateQuantity(id, quantity) {
    startTransition(async () => {
      dispatch({ type: 'update_quantity', id, quantity });
      await cartActions.updateQuantity(id, quantity);
    });
  }

  const total = optimisticCart.reduce(
    (sum, item) => sum + item.price * item.quantity,
    0
  );

  return (
    <div>
      <h2>Shopping Cart</h2>
      <div style={{ marginBottom: 16 }}>
        <button onClick={() => handleAdd({
          id: 1, name: 'T-Shirt', price: 25
        })}>
          Add T-Shirt ($25)
        </button>{' '}
        <button onClick={() => handleAdd({
          id: 2, name: 'Mug', price: 15
        })}>
          Add Mug ($15)
        </button>
      </div>
      {optimisticCart.length === 0 ? (
        <p>Your cart is empty</p>
      ) : (
        <ul>
          {optimisticCart.map(item => (
            <li key={item.id}>
              {item.name} - ${item.price} ×
              {item.quantity}
              {' '}= ${item.price * item.quantity}
              <button
                onClick={() => handleRemove(item.id)}
                style={{ marginLeft: 8 }}
              >
                Remove
              </button>
              {item.pending && ' ...'}
            </li>
          ))}
        </ul>
      )}
      <p><strong>Total: ${total}</strong></p>
    </div>
  );
}
```

```js src/actions.js hidden
export async function addToCart(item) {
  await new Promise((res) => setTimeout(res, 800));
}

export async function removeFromCart(id) {
  await new Promise((res) => setTimeout(res, 800));
}

export async function updateQuantity(id, quantity) {
  await new Promise((res) => setTimeout(res, 800));
}
```

</Sandpack>

reducer 处理三种 `action` 类型（`add`、`remove`、`update_quantity`），并为每种类型返回新的乐观状态。每个 `action` 都会设置 `pending: true` 标记，这样你就可以在 [Server Function](/reference/rsc/server-functions) 运行时显示视觉反馈。

---

### 带错误恢复的乐观删除 {/*optimistic-delete-with-error-recovery*/}

在以乐观方式删除项目时，你应该处理 Action 失败的情况。

这个示例展示了当删除失败时如何显示错误消息，并且 UI 会自动回滚以再次显示该项目。

<Sandpack>

```js src/App.js
import { useState, startTransition } from 'react';
import { deleteItem } from './actions.js';
import ItemList from './ItemList';

export default function App() {
  const [items, setItems] = useState([
    { id: 1, name: 'Learn React' },
    { id: 2, name: 'Build an app' },
    { id: 3, name: 'Deploy to production' },
  ]);

  async function deleteAction(id) {
    await deleteItem(id);
    startTransition(() => {
      setItems(current => current.filter(item => item.id !== id));
    });
  }

  return <ItemList items={items} deleteAction={deleteAction} />;
}
```

```js src/ItemList.js active
import { useState, useOptimistic, startTransition } from 'react';

export default function ItemList({ items, deleteAction }) {
  const [error, setError] = useState(null);
  const [optimisticItems, removeItem] = useOptimistic(
    items,
    (currentItems, idToRemove) =>
      currentItems.map(item =>
        item.id === idToRemove
          ? { ...item, deleting: true }
          : item
      )
  );

  function handleDelete(id) {
    setError(null);
    startTransition(async () => {
      removeItem(id);
      try {
        await deleteAction(id);
      } catch (e) {
        setError(e.message);
      }
    });
  }

  return (
    <div>
      <h2>Your Items</h2>
      <ul>
        {optimisticItems.map(item => (
          <li
            key={item.id}
            style={{
              opacity: item.deleting ? 0.5 : 1,
              textDecoration: item.deleting ? 'line-through' : 'none',
              transition: 'opacity 0.2s'
            }}
          >
            {item.name}
            <button
              onClick={() => handleDelete(item.id)}
              disabled={item.deleting}
              style={{ marginLeft: 8 }}
            >
              {item.deleting ? 'Deleting...' : 'Delete'}
            </button>
          </li>
        ))}
      </ul>
      {error && (
        <p style={{ color: 'red', padding: 8, background: '#fee' }}>
          {error}
        </p>
      )}
    </div>
  );
}
```

```js src/actions.js hidden
export async function deleteItem(id) {
  await new Promise((res) => setTimeout(res, 1000));
  // 项目 3 始终失败，用于演示错误恢复
  if (id === 3) {
    throw new Error('Cannot delete. Permission denied.');
  }
}
```

</Sandpack>

试着删除 “Deploy to production”。当删除失败时，该项目会自动重新出现在列表中。

---

## 故障排查 {/*troubleshooting*/}

### 我遇到了一个错误：“An optimistic state update occurred outside a Transition or Action” {/*an-optimistic-state-update-occurred-outside-a-transition-or-action*/}

你可能会看到这个错误：

<ConsoleBlockMulti>

<ConsoleLogLine level="error">

An optimistic state update occurred outside a Transition or Action. To fix, move the update to an Action, or wrap with `startTransition`.

</ConsoleLogLine>

</ConsoleBlockMulti>

乐观更新的 setter 函数必须在 `startTransition` 内调用：

```js
// 🚩 不正确：在 Transition 之外
function handleClick() {
  setOptimistic(newValue);  // 警告！
  // ...
}

// ✅ 正确：在 Transition 内
function handleClick() {
  startTransition(async () => {
    setOptimistic(newValue);
    // ...
  });
}

// ✅ 也正确：在 Action prop 内
function submitAction(formData) {
  setOptimistic(newValue);
  // ...
}
```

当你在 Action 之外调用 setter 时，乐观状态会短暂出现，然后立即回退到原始值。这是因为没有 Transition 在你的 Action 运行时“维持”这个乐观状态。

### 我遇到了一个错误：“Cannot update optimistic state while rendering” {/*cannot-update-optimistic-state-while-rendering*/}

你可能会看到这个错误：

<ConsoleBlockMulti>

<ConsoleLogLine level="error">

Cannot update optimistic state while rendering.

</ConsoleLogLine>

</ConsoleBlockMulti>

当你在组件的渲染阶段调用乐观更新的 setter 时，就会发生此错误。你只能在事件处理函数、effects 或其他回调中调用它：

```js
// 🚩 不正确：在渲染期间调用
function MyComponent({ items }) {
  const [isPending, setPending] = useOptimistic(false);

  // 这会在渲染期间运行 - 不允许！
  setPending(true);

  // ...
}

// ✅ 正确：在 startTransition 中调用
function MyComponent({ items }) {
  const [isPending, setPending] = useOptimistic(false);

  function handleClick() {
    startTransition(() => {
      setPending(true);
      // ...
    });
  }

  // ...
}

// ✅ 也正确：从 Action 中调用
function MyComponent({ items }) {
  const [isPending, setPending] = useOptimistic(false);

  function action() {
    setPending(true);
    // ...
  }

  // ...
}
```

### 我的乐观更新显示的是过期值 {/*my-optimistic-updates-show-stale-values*/}

如果你的乐观状态似乎基于旧数据，请考虑使用更新器函数或 reducer 来相对于当前状态计算乐观状态。

```js
// 如果状态在 Action 期间发生变化，可能会显示过期数据
const [optimistic, setOptimistic] = useOptimistic(count);
setOptimistic(5);  // 始终设置为 5，即使 count 已更改

// 更好：相对更新能正确处理状态变化
const [optimistic, adjust] = useOptimistic(count, (current, delta) => current + delta);
adjust(1);  // 始终在当前 count 的基础上加 1
```

详情请参见[基于当前状态更新状态](#updating-state-based-on-current-state)。

### 我不知道我的乐观更新是否处于 pending 状态 {/*i-dont-know-if-my-optimistic-update-is-pending*/}

要知道 `useOptimistic` 是否处于 pending 状态，你有三种选择：

1. **检查 `optimisticValue === value` 是否成立**

```js
const [optimistic, setOptimistic] = useOptimistic(value);
const isPending = optimistic !== value;
```

如果这些值不相等，就表示有一个 Transition 正在进行中。

2. **添加一个 `useTransition`**

```js
const [isPending, startTransition] = useTransition();
const [optimistic, setOptimistic] = useOptimistic(value);

//...
startTransition(() => {
  setOptimistic(state);
})
```

由于 `useTransition` 在底层使用 `useOptimistic` 来提供 `isPending`，这与选项 1 等价。

3. **在 reducer 中添加一个 `pending` 标志**

```js
const [optimistic, addOptimistic] = useOptimistic(
  items,
  (state, newItem) => [...state, { ...newItem, isPending: true }]
);
```

由于每个乐观项都有自己的标志，你可以为单个项目显示加载状态。
