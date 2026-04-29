---
title: useActionState
---

<Intro>

`useActionState` 是一个 React Hook，允许你使用 [Actions](/reference/react/useTransition#functions-called-in-starttransition-are-called-actions) 以及副作用来更新状态。

```js
const [state, dispatchAction, isPending] = useActionState(reducerAction, initialState, permalink?);
```

</Intro>

<InlineToc />

---

## 参考 {/*reference*/}

### `useActionState(reducerAction, initialState, permalink?)` {/*useactionstate*/}

在组件顶层调用 `useActionState`，为一个 Action 的结果创建状态。

```js
import { useActionState } from 'react';

function reducerAction(previousState, actionPayload) {
  // ...
}

function MyCart({initialState}) {
  const [state, dispatchAction, isPending] = useActionState(reducerAction, initialState);
  // ...
}
```

[查看更多示例。](#usage)

#### 参数 {/*parameters*/}

* `reducerAction`：当 Action 被触发时调用的函数。被调用时，它会接收上一次状态（最初是你提供的 `initialState`，之后是其上一次返回值）作为第一个参数，随后是传递给 `dispatchAction` 的 `actionPayload`。
* `initialState`：你希望状态最初的值。`dispatchAction` 第一次被调用后，React 会忽略这个参数。
* **可选** `permalink`：包含此表单所修改的唯一页面 URL 的字符串。
  * 用于具有[React Server Components](/reference/rsc/server-components)并支持渐进增强的页面。
  * 如果 `reducerAction` 是一个 [Server Function](/reference/rsc/server-functions)，并且表单在 JavaScript bundle 加载前提交，浏览器将导航到指定的 permalink URL，而不是当前页面的 URL。

#### 返回值 {/*returns*/}

`useActionState` 返回一个恰好包含三个值的数组：

1. 当前状态。在第一次渲染期间，它将与您传入的 `initialState` 相匹配。`dispatchAction` 被调用后，它将与 `reducerAction` 返回的值相匹配。
2. 一个 `dispatchAction` 函数，你可以在 [Actions](/reference/react/useTransition#functions-called-in-starttransition-are-called-actions) 中调用它。
3. `isPending` 标志，告诉你这个 Hook 的任何已派发 Actions 是否处于待处理状态。

#### 注意事项 {/*caveats*/}

* `useActionState` 是一个 Hook，因此你只能在**组件顶层**或你自己的 Hooks 中调用它。不能在循环或条件中调用它。如果你需要这样做，请提取一个新组件并将状态移动到其中。
* React 会按顺序排队并执行对 `dispatchAction` 的多次调用。每次对 `reducerAction` 的调用都会接收上一次调用的结果。
* `dispatchAction` 函数具有稳定的身份，因此你通常会看到它被省略在 Effect 依赖项中，但将它包含进去不会导致 Effect 触发。如果 linter 允许你在没有错误的情况下省略某个依赖项，那就是安全的。[了解更多关于移除 Effect 依赖项。](/learn/removing-effect-dependencies#move-dynamic-objects-and-functions-inside-your-effect)
* 使用 `permalink` 选项时，请确保目标页面渲染的是相同的表单组件（包括相同的 `reducerAction` 和 `permalink`），这样 React 才知道如何传递状态。一旦页面变得可交互，这个参数就不再起作用。
* 使用 Server Functions 时，`initialState` 需要是[可序列化的](/reference/rsc/use-server#serializable-parameters-and-return-values)（例如普通对象、数组、字符串和数字）。
* 如果 `dispatchAction` 抛出错误，React 会取消所有排队中的 Actions，并显示最近的 [Error Boundary](/reference/react/Component#catching-rendering-errors-with-an-error-boundary)。
* 如果有多个进行中的 Actions，React 会将它们批量处理。这是一个限制，未来版本中可能会移除。

<Note>

`dispatchAction` 必须从一个 Action 中调用。

你可以把它包裹在 [`startTransition`](/reference/react/startTransition) 中，或者将它传递给一个 [Action prop](/reference/react/useTransition#exposing-action-props-from-components)。在该作用域之外的调用不会被视为 Transition 的一部分，并且会在开发模式下[记录错误](#async-function-outside-transition)。

</Note>

---

### `reducerAction` 函数 {/*reduceraction*/}

传递给 `useActionState` 的 `reducerAction` 函数会接收前一个状态并返回一个新状态。

不同于 `useReducer` 中的 reducer，`reducerAction` 可以是异步的，并且可以执行副作用：

```js
async function reducerAction(previousState, actionPayload) {
  const newState = await post(actionPayload);
  return newState;
}
```

每次调用 `dispatchAction` 时，React 都会使用 `actionPayload` 调用 `reducerAction`。reducer 会执行诸如向服务器发送数据之类的副作用，并返回新状态。如果 `dispatchAction` 被多次调用，React 会按顺序排队执行它们，因此上一次调用的结果会作为当前调用的 `previousState`。

#### 参数 {/*reduceraction-parameters*/}

* `previousState`：上一次状态。初始时等于 `initialState`。在第一次调用 `dispatchAction` 之后，它等于上一次返回的状态。

* **可选** `actionPayload`：传递给 `dispatchAction` 的参数。它可以是任何类型的值。类似于 `useReducer` 的约定，它通常是一个带有 `type` 属性用于标识类型的对象，并且可选地包含其他提供附加信息的属性。

#### 返回值 {/*reduceraction-returns*/}

`reducerAction` 返回新状态，并触发一次 Transition 以使用该状态重新渲染。

#### 注意事项 {/*reduceraction-caveats*/}

* `reducerAction` 可以是同步或异步的。它可以执行同步操作，例如显示通知，也可以执行异步操作，例如向服务器发送更新。
* `reducerAction` 不会在 `<StrictMode>` 中被调用两次，因为 `reducerAction` 的设计就是允许副作用存在。
* `reducerAction` 的返回类型必须与 `initialState` 的类型匹配。如果 TypeScript 推断出不匹配，你可能需要显式标注状态类型。
* 如果你在 `reducerAction` 中于 `await` 之后设置状态，你当前需要将该状态更新包裹在额外的 `startTransition` 中。更多信息请参阅 [startTransition](/reference/react/useTransition#react-doesnt-treat-my-state-update-after-await-as-a-transition) 文档。
* 使用 Server Functions 时，`actionPayload` 需要是[可序列化的](/reference/rsc/use-server#serializable-parameters-and-return-values)（例如普通对象、数组、字符串和数字）。

<DeepDive>

#### 为什么叫 `reducerAction`？ {/*why-is-it-called-reduceraction*/}

传递给 `useActionState` 的函数被称为 *reducer action*，因为：

- 它会像 `useReducer` 一样，把前一个状态“归约”成一个新状态。
- 它是一个 *Action*，因为它是在 Transition 中被调用的，并且可以执行副作用。

从概念上讲，`useActionState` 就像带有副作用的 `useReducer`。

</DeepDive>

---

## 使用场景 {/*usage*/}

### 为 Action 添加状态 {/*adding-state-to-an-action*/}

在组件顶层调用 `useActionState`，为一个 Action 的结果创建状态。

```js [[1, 7, "count"], [2, 7, "dispatchAction"], [3, 7, "isPending"]]
import { useActionState } from 'react';

async function addToCartAction(prevCount) {
  // ...
}
function Counter() {
  const [count, dispatchAction, isPending] = useActionState(addToCartAction, 0);

  // ...
}
```

`useActionState` 返回一个恰好包含三个项的数组：

1. <CodeStep step={1}>当前状态</CodeStep>，初始设置为你提供的初始状态。
2. <CodeStep step={2}>action 派发器</CodeStep>，用于触发 `reducerAction`。
3. <CodeStep step={3}>pending 状态</CodeStep>，告诉你 Action 是否正在进行中。

要调用 `addToCartAction`，请调用 <CodeStep step={2}>action 派发器</CodeStep>。React 会用上一次数量来排队调用 `addToCartAction`。

<Sandpack>

```js src/App.js
import { useActionState, startTransition } from 'react';
import { addToCart } from './api';
import Total from './Total';

export default function Checkout() {
  const [count, dispatchAction, isPending] = useActionState(async (prevCount) => {
    return await addToCart(prevCount)
  }, 0);

  function handleClick() {
    startTransition(() => {
      dispatchAction();
    });
  }

  return (
    <div className="checkout">
      <h2>结账</h2>
      <div className="row">
        <span>Eras 巡演门票</span>
        <span>数量：{count}</span>
      </div>
      <div className="row">
        <button onClick={handleClick}>添加门票{isPending ? ' 🌀' : '  '}</button>
      </div>
      <hr />
      <Total quantity={count} />
    </div>
  );
}
```

```js src/Total.js
const formatter = new Intl.NumberFormat('en-US', {
  style: 'currency',
  currency: 'USD',
  minimumFractionDigits: 0,
});

export default function Total({quantity}) {
  return (
    <div className="row total">
      <span>总计</span>
      <span>{formatter.format(quantity * 9999)}</span>
    </div>
  );
}
```

```js src/api.js
export async function addToCart(count) {
  await new Promise(resolve => setTimeout(resolve, 1000));
  return count + 1;
}

export async function removeFromCart(count) {
  await new Promise(resolve => setTimeout(resolve, 1000));
  return Math.max(0, count - 1);
}
```

```css
.checkout {
  display: flex;
  flex-direction: column;
  gap: 12px;
  padding: 16px;
  border: 1px solid #ccc;
  border-radius: 8px;
  font-family: system-ui;
}

.checkout h2 {
  margin: 0 0 8px 0;
}

.row {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.row button {
  margin-left: auto;
  min-width: 150px;
}

.total {
  font-weight: bold;
}

hr {
  width: 100%;
  border: none;
  border-top: 1px solid #ccc;
  margin: 4px 0;
}

button {
  padding: 8px 16px;
  cursor: pointer;
}
```

</Sandpack>

每次你点击“添加门票”，React 都会排队调用一次 `addToCartAction`。在所有门票都添加完之前，React 会显示 pending 状态，然后用最终状态重新渲染。

<DeepDive>

#### `useActionState` 的排队机制如何工作 {/*how-useactionstate-queuing-works*/}

试着多次点击“添加门票”。每次点击，都会排队一个新的 `addToCartAction`。由于有一个人为设置的 1 秒延迟，这意味着 4 次点击大约需要 4 秒完成。

**这正是 `useActionState` 设计中的预期行为。**

我们必须等待 `addToCartAction` 的上一个结果，以便将 `prevCount` 传递给下一次 `addToCartAction` 调用。这意味着 React 必须等前一个 Action 完成后才能调用下一个 Action。

通常你可以通过[与 useOptimistic 结合使用](/reference/react/useActionState#using-with-useoptimistic)来解决这个问题，但对于更复杂的情况，你可能需要考虑[取消排队中的 actions](#cancelling-queued-actions) 或者不使用 `useActionState`。

</DeepDive>

---

### 使用多个 Action 类型 {/*using-multiple-action-types*/}

要处理多种类型，你可以向 `dispatchAction` 传递一个参数。

按照惯例，通常会把它写成一个 switch 语句。对于 switch 中的每个 case，计算并返回某个下一状态。该参数可以有任意结构，但常见做法是传递带有 `type` 属性来标识 action 的对象。

<Sandpack>

```js src/App.js
import { useActionState, startTransition } from 'react';
import { addToCart, removeFromCart } from './api';
import Total from './Total';

export default function Checkout() {
  const [count, dispatchAction, isPending] = useActionState(updateCartAction, 0);

  function handleAdd() {
    startTransition(() => {
      dispatchAction({ type: 'ADD' });
    });
  }

  function handleRemove() {
    startTransition(() => {
      dispatchAction({ type: 'REMOVE' });
    });
  }

  return (
    <div className="checkout">
      <h2>结账</h2>
      <div className="row">
        <span>Eras 巡演门票</span>
        <span className="stepper">
          <span className="qty">{isPending ? '🌀' : count}</span>
          <span className="buttons">
            <button onClick={handleAdd}>▲</button>
            <button onClick={handleRemove}>▼</button>
          </span>
        </span>
      </div>
      <hr />
      <Total quantity={count} isPending={isPending}/>
    </div>
  );
}

async function updateCartAction(prevCount, actionPayload) {
  switch (actionPayload.type) {
    case 'ADD': {
      return await addToCart(prevCount);
    }
    case 'REMOVE': {
      return await removeFromCart(prevCount);
    }
  }
  return prevCount;
}
```

```js src/Total.js
const formatter = new Intl.NumberFormat('en-US', {
  style: 'currency',
  currency: 'USD',
  minimumFractionDigits: 0,
});

export default function Total({quantity, isPending}) {
  return (
    <div className="row total">
      <span>总计</span>
      {isPending ? '🌀 正在更新...' : formatter.format(quantity * 9999)}
    </div>
  );
}
```

```js src/api.js hidden
export async function addToCart(count) {
  await new Promise(resolve => setTimeout(resolve, 1000));
  return count + 1;
}

export async function removeFromCart(count) {
  await new Promise(resolve => setTimeout(resolve, 1000));
  return Math.max(0, count - 1);
}
```

```css
.checkout {
  display: flex;
  flex-direction: column;
  gap: 12px;
  padding: 16px;
  border: 1px solid #ccc;
  border-radius: 8px;
  font-family: system-ui;
}

.checkout h2 {
  margin: 0 0 8px 0;
}

.row {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.stepper {
  display: flex;
  align-items: center;
  gap: 8px;
}

.qty {
  min-width: 20px;
  text-align: center;
}

.buttons {
  display: flex;
  flex-direction: column;
  gap: 2px;
}

.buttons button {
  padding: 0 8px;
  font-size: 10px;
  line-height: 1.2;
  cursor: pointer;
}

.pending {
  width: 20px;
  text-align: center;
}

.total {
  font-weight: bold;
}

hr {
  width: 100%;
  border: none;
  border-top: 1px solid #ccc;
  margin: 4px 0;
}
```

</Sandpack>

当你点击增加或减少数量时，会派发 `"ADD"` 或 `"REMOVE"`。在 `reducerAction` 中，会调用不同的 API 来更新数量。

在这个示例中，我们使用 Actions 的 pending 状态同时替换数量和总计。如果你想提供即时反馈，比如立即更新数量，可以使用 `useOptimistic`。

<DeepDive>

#### `useActionState` 与 `useReducer` 有什么不同？ {/*useactionstate-vs-usereducer*/}

你可能会注意到这个示例看起来很像 `useReducer`，但它们的用途不同：

- **使用 `useReducer`** 来管理 UI 的状态。reducer 必须是纯函数。

- **使用 `useActionState`** 来管理 Actions 的状态。reducer 可以执行副作用。

你可以把 `useActionState` 理解为“用于用户 Actions 副作用的 `useReducer`”。由于它会根据上一次 Action 计算下一步要执行的 Action，因此它必须[按顺序排队调用](/reference/react/useActionState#how-useactionstate-queuing-works)。如果你想并行执行 Actions，请直接使用 `useState` 和 `useTransition`。

</DeepDive>

---

### 与 `useOptimistic` 结合使用 {/*using-with-useoptimistic*/}

你可以将 `useActionState` 与 [`useOptimistic`](/reference/react/useOptimistic) 结合起来，显示即时的 UI 反馈：


<Sandpack>

```js src/App.js
import { useActionState, startTransition, useOptimistic } from 'react';
import { addToCart, removeFromCart } from './api';
import Total from './Total';

export default function Checkout() {
  const [count, dispatchAction, isPending] = useActionState(updateCartAction, 0);
  const [optimisticCount, setOptimisticCount] = useOptimistic(count);

  function handleAdd() {
    startTransition(() => {
      setOptimisticCount(c => c + 1);
      dispatchAction({ type: 'ADD' });
    });
  }

  function handleRemove() {
    startTransition(() => {
      setOptimisticCount(c => Math.max(0, c - 1));
      dispatchAction({ type: 'REMOVE' });
    });
  }

  return (
    <div className="checkout">
      <h2>结账</h2>
      <div className="row">
        <span>Eras 巡演门票</span>
        <span className="stepper">
          <span className="pending">{isPending && '🌀'}</span>
          <span className="qty">{optimisticCount}</span>
          <span className="buttons">
            <button onClick={handleAdd}>▲</button>
            <button onClick={handleRemove}>▼</button>
          </span>
        </span>
      </div>
      <hr />
      <Total quantity={optimisticCount} isPending={isPending}/>
    </div>
  );
}

async function updateCartAction(prevCount, actionPayload) {
  switch (actionPayload.type) {
    case 'ADD': {
      return await addToCart(prevCount);
    }
    case 'REMOVE': {
      return await removeFromCart(prevCount);
    }
  }
  return prevCount;
}
```

```js src/Total.js
const formatter = new Intl.NumberFormat('en-US', {
  style: 'currency',
  currency: 'USD',
  minimumFractionDigits: 0,
});

export default function Total({quantity, isPending}) {
  return (
    <div className="row total">
      <span>总计</span>
      <span>{isPending ? '🌀 正在更新...' : formatter.format(quantity * 9999)}</span>
    </div>
  );
}
```

```js src/api.js hidden
export async function addToCart(count) {
  await new Promise(resolve => setTimeout(resolve, 1000));
  return count + 1;
}

export async function removeFromCart(count) {
  await new Promise(resolve => setTimeout(resolve, 1000));
  return Math.max(0, count - 1);
}
```

```css
.checkout {
  display: flex;
  flex-direction: column;
  gap: 12px;
  padding: 16px;
  border: 1px solid #ccc;
  border-radius: 8px;
  font-family: system-ui;
}

.checkout h2 {
  margin: 0 0 8px 0;
}

.row {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.stepper {
  display: flex;
  align-items: center;
  gap: 8px;
}

.qty {
  min-width: 20px;
  text-align: center;
}

.buttons {
  display: flex;
  flex-direction: column;
  gap: 2px;
}

.buttons button {
  padding: 0 8px;
  font-size: 10px;
  line-height: 1.2;
  cursor: pointer;
}

.pending {
  width: 20px;
  text-align: center;
}

.total {
  font-weight: bold;
}

hr {
  width: 100%;
  border: none;
  border-top: 1px solid #ccc;
  margin: 4px 0;
}
```

</Sandpack>


`setOptimisticCount` 会立即更新数量，而 `dispatchAction()` 会排队执行 `updateCartAction`。数量和总计上都会显示一个 pending 指示器，给用户反馈说明更新仍在应用中。

---


### 与 Action props 结合使用 {/*using-with-action-props*/}

当你把 `dispatchAction` 函数传给一个暴露 [Action prop](/reference/react/useTransition#exposing-action-props-from-components) 的组件时，你不需要自己调用 `startTransition` 或 `useOptimistic`。

这个示例展示了如何使用 QuantityStepper 组件的 `increaseAction` 和 `decreaseAction` props：

<Sandpack>

```js src/App.js
import { useActionState } from 'react';
import { addToCart, removeFromCart } from './api';
import QuantityStepper from './QuantityStepper';
import Total from './Total';

export default function Checkout() {
  const [count, dispatchAction, isPending] = useActionState(updateCartAction, 0);

  function addAction() {
    dispatchAction({type: 'ADD'});
  }

  function removeAction() {
    dispatchAction({type: 'REMOVE'});
  }

  return (
    <div className="checkout">
      <h2>结账</h2>
      <div className="row">
        <span>Eras 巡演门票</span>
        <QuantityStepper
          value={count}
          increaseAction={addAction}
          decreaseAction={removeAction}
        />
      </div>
      <hr />
      <Total quantity={count} isPending={isPending} />
    </div>
  );
}

async function updateCartAction(prevCount, actionPayload) {
  switch (actionPayload.type) {
    case 'ADD': {
      return await addToCart(prevCount);
    }
    case 'REMOVE': {
      return await removeFromCart(prevCount);
    }
  }
  return prevCount;
}
```

```js src/QuantityStepper.js
import { startTransition, useOptimistic } from 'react';

export default function QuantityStepper({value, increaseAction, decreaseAction}) {
  const [optimisticValue, setOptimisticValue] = useOptimistic(value);
  const isPending = value !== optimisticValue;
  function handleIncrease() {
    startTransition(async () => {
      setOptimisticValue(c => c + 1);
      await increaseAction();
    });
  }

  function handleDecrease() {
    startTransition(async () => {
      setOptimisticValue(c => Math.max(0, c - 1));
      await decreaseAction();
    });
  }

  return (
    <span className="stepper">
      <span className="pending">{isPending && '🌀'}</span>
      <span className="qty">{optimisticValue}</span>
      <span className="buttons">
        <button onClick={handleIncrease}>▲</button>
        <button onClick={handleDecrease}>▼</button>
      </span>
    </span>
  );
}
```

```js src/Total.js
const formatter = new Intl.NumberFormat('en-US', {
  style: 'currency',
  currency: 'USD',
  minimumFractionDigits: 0,
});

export default function Total({quantity, isPending}) {
  return (
    <div className="row total">
      <span>总计</span>
      {isPending ? '🌀 正在更新...' : formatter.format(quantity * 9999)}
    </div>
  );
}
```

```js src/api.js hidden
export async function addToCart(count) {
  await new Promise(resolve => setTimeout(resolve, 1000));
  return count + 1;
}

export async function removeFromCart(count) {
  await new Promise(resolve => setTimeout(resolve, 1000));
  return Math.max(0, count - 1);
}
```

```css
.checkout {
  display: flex;
  flex-direction: column;
  gap: 12px;
  padding: 16px;
  border: 1px solid #ccc;
  border-radius: 8px;
  font-family: system-ui;
}

.checkout h2 {
  margin: 0 0 8px 0;
}

.row {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.stepper {
  display: flex;
  align-items: center;
  gap: 8px;
}

.qty {
  min-width: 20px;
  text-align: center;
}

.buttons {
  display: flex;
  flex-direction: column;
  gap: 2px;
}

.buttons button {
  padding: 0 8px;
  font-size: 10px;
  line-height: 1.2;
  cursor: pointer;
}

.pending {
  width: 20px;
  text-align: center;
}

.total {
  font-weight: bold;
}

hr {
  width: 100%;
  border: none;
  border-top: 1px solid #ccc;
  margin: 4px 0;
}
```

</Sandpack>

由于 `<QuantityStepper>` 内置了对 transitions、pending 状态以及乐观更新数量的支持，你只需要告诉 Action _要改变什么_，而 _如何改变_ 会由它替你处理。

---

### 取消排队中的 Actions {/*cancelling-queued-actions*/}

你可以使用 `AbortController` 来取消待处理的 Actions：

<Sandpack>

```js src/App.js
import { useActionState, useRef } from 'react';
import { addToCart, removeFromCart } from './api';
import QuantityStepper from './QuantityStepper';
import Total from './Total';

export default function Checkout() {
  const abortRef = useRef(null);
  const [count, dispatchAction, isPending] = useActionState(updateCartAction, 0);

  async function addAction() {
    if (abortRef.current) {
      abortRef.current.abort();
    }
    abortRef.current = new AbortController();
    await dispatchAction({ type: 'ADD', signal: abortRef.current.signal });
  }

  async function removeAction() {
    if (abortRef.current) {
      abortRef.current.abort();
    }
    abortRef.current = new AbortController();
    await dispatchAction({ type: 'REMOVE', signal: abortRef.current.signal });
  }

  return (
    <div className="checkout">
      <h2>结账</h2>
      <div className="row">
        <span>Eras 巡演门票</span>
        <QuantityStepper
          value={count}
          increaseAction={addAction}
          decreaseAction={removeAction}
        />
      </div>
      <hr />
      <Total quantity={count} isPending={isPending} />
    </div>
  );
}

async function updateCartAction(prevCount, actionPayload) {
  switch (actionPayload.type) {
    case 'ADD': {
      try {
        return await addToCart(prevCount, { signal: actionPayload.signal });
      } catch (e) {
        return prevCount + 1;
      }
    }
    case 'REMOVE': {
      try {
        return await removeFromCart(prevCount, { signal: actionPayload.signal });
      } catch (e) {
        return Math.max(0, prevCount - 1);
      }
    }
  }
  return prevCount;
}
```

```js src/QuantityStepper.js
import { startTransition, useOptimistic } from 'react';

export default function QuantityStepper({value, increaseAction, decreaseAction}) {
  const [optimisticValue, setOptimisticValue] = useOptimistic(value);
  const isPending = value !== optimisticValue;
  function handleIncrease() {
    startTransition(async () => {
      setOptimisticValue(c => c + 1);
      await increaseAction();
    });
  }

  function handleDecrease() {
    startTransition(async () => {
      setOptimisticValue(c => Math.max(0, c - 1));
      await decreaseAction();
    });
  }

  return (
          <span className="stepper">
      <span className="pending">{isPending && '🌀'}</span>
      <span className="qty">{optimisticValue}</span>
      <span className="buttons">
        <button onClick={handleIncrease}>▲</button>
        <button onClick={handleDecrease}>▼</button>
      </span>
    </span>
  );
}
```

```js src/Total.js
const formatter = new Intl.NumberFormat('en-US', {
  style: 'currency',
  currency: 'USD',
  minimumFractionDigits: 0,
});

export default function Total({quantity, isPending}) {
  return (
    <div className="row total">
      <span>总计</span>
      {isPending ? '🌀 正在更新...' : formatter.format(quantity * 9999)}
    </div>
  );
}
```

```js src/api.js hidden
class AbortError extends Error {
  name = 'AbortError';
  constructor(message = '操作已中止') {
    super(message);
  }
}

function sleep(ms, signal) {
  if (!signal) return new Promise((resolve) => setTimeout(resolve, ms));
  if (signal.aborted) return Promise.reject(new AbortError());

  return new Promise((resolve, reject) => {
    const id = setTimeout(() => {
      signal.removeEventListener('abort', onAbort);
      resolve();
    }, ms);

    const onAbort = () => {
      clearTimeout(id);
      reject(new AbortError());
    };

    signal.addEventListener('abort', onAbort, { once: true });
  });
}
export async function addToCart(count, opts) {
  await sleep(1000, opts?.signal);
  return count + 1;
}

export async function removeFromCart(count, opts) {
  await sleep(1000, opts?.signal);
  return Math.max(0, count - 1);
}
```

```css
.checkout {
  display: flex;
  flex-direction: column;
  gap: 12px;
  padding: 16px;
  border: 1px solid #ccc;
  border-radius: 8px;
  font-family: system-ui;
}

.checkout h2 {
  margin: 0 0 8px 0;
}

.row {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.stepper {
  display: flex;
  align-items: center;
  gap: 8px;
}

.qty {
  min-width: 20px;
  text-align: center;
}

.buttons {
  display: flex;
  flex-direction: column;
  gap: 2px;
}

.buttons button {
  padding: 0 8px;
  font-size: 10px;
  line-height: 1.2;
  cursor: pointer;
}

.pending {
  width: 20px;
  text-align: center;
}

.total {
  font-weight: bold;
}

hr {
  width: 100%;
  border: none;
  border-top: 1px solid #ccc;
  margin: 4px 0;
}
```

</Sandpack>

试着多次点击增加或减少，注意无论你点击多少次，总计都会在 1 秒内更新。这是因为它使用了 `AbortController` 来“完成”上一个 Action，从而让下一个 Action 可以继续执行。

<Pitfall>

中止一个 Action 并不总是安全的。

例如，如果 Action 执行的是一次变更操作（比如向数据库写入），中止网络请求并不会撤销服务端已经发生的更改。这就是 `useActionState` 默认不自动中止的原因。只有当你明确知道副作用可以安全地被忽略或重试时，这样做才安全。

</Pitfall>

---

### 与 `<form>` Action props 结合使用 {/*use-with-a-form*/}

你可以将 `dispatchAction` 函数作为 `action` prop 传给 `<form>`。

这样使用时，React 会自动把提交包裹在一个 Transition 中，所以你不需要自己调用 `startTransition`。`reducerAction` 会接收前一个状态和提交的 `FormData`：

<Sandpack>

```js src/App.js
import { useActionState, useOptimistic } from 'react';
import { addToCart, removeFromCart } from './api';
import Total from './Total';

export default function Checkout() {
  const [count, dispatchAction, isPending] = useActionState(updateCartAction, 0);
  const [optimisticCount, setOptimisticCount] = useOptimistic(count);

  async function formAction(formData) {
    const type = formData.get('type');
    if (type === 'ADD') {
      setOptimisticCount(c => c + 1);
    } else {
      setOptimisticCount(c => Math.max(0, c - 1));
    }
    return dispatchAction(formData);
  }

  return (
    <form action={formAction} className="checkout">
      <h2>结账</h2>
      <div className="row">
        <span>Eras 巡演门票</span>
        <span className="stepper">
          <span className="pending">{isPending && '🌀'}</span>
          <span className="qty">{optimisticCount}</span>
          <span className="buttons">
            <button type="submit" name="type" value="ADD">▲</button>
            <button type="submit" name="type" value="REMOVE">▼</button>
          </span>
        </span>
      </div>
      <hr />
      <Total quantity={count} isPending={isPending} />
    </form>
  );
}

async function updateCartAction(prevCount, formData) {
  const type = formData.get('type');
  switch (type) {
    case 'ADD': {
      return await addToCart(prevCount);
    }
    case 'REMOVE': {
      return await removeFromCart(prevCount);
    }
  }
  return prevCount;
}
```

```js src/Total.js
const formatter = new Intl.NumberFormat('en-US', {
  style: 'currency',
  currency: 'USD',
  minimumFractionDigits: 0,
});

export default function Total({quantity, isPending}) {
  return (
    <div className="row total">
      <span>总计</span>
      {isPending ? '🌀 正在更新...' : formatter.format(quantity * 9999)}
    </div>
  );
}
```

```js src/api.js hidden
export async function addToCart(count) {
  await new Promise(resolve => setTimeout(resolve, 1000));
  return count + 1;
}

export async function removeFromCart(count) {
  await new Promise(resolve => setTimeout(resolve, 1000));
  return Math.max(0, count - 1);
}
```

```css
.checkout {
  display: flex;
  flex-direction: column;
  gap: 12px;
  padding: 16px;
  border: 1px solid #ccc;
  border-radius: 8px;
  font-family: system-ui;
}

.checkout h2 {
  margin: 0 0 8px 0;
}

.row {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.stepper {
  display: flex;
  align-items: center;
  gap: 8px;
}

.qty {
  min-width: 20px;
  text-align: center;
}

.buttons {
  display: flex;
  flex-direction: column;
  gap: 2px;
}

.buttons button {
  padding: 0 8px;
  font-size: 10px;
  line-height: 1.2;
  cursor: pointer;
}

.pending {
  width: 20px;
  text-align: center;
}

.total {
  font-weight: bold;
}

hr {
  width: 100%;
  border: none;
  border-top: 1px solid #ccc;
  margin: 4px 0;
}
```

</Sandpack>

在这个示例中，当用户点击步进器箭头时，按钮会提交表单，而 `useActionState` 会使用表单数据调用 `updateCartAction`。该示例使用 `useOptimistic` 在服务器确认更新之前立即显示新数量。

<RSC>

当与 [Server Function](/reference/rsc/server-functions) 一起使用时，`useActionState` 允许在 hydration（React 绑定到服务器渲染的 HTML）完成之前就显示服务器的响应。你也可以在具有动态内容的页面上使用可选的 `permalink` 参数进行渐进增强（允许表单在 JavaScript 加载前工作）。这通常由你的框架替你处理。

</RSC>

有关在表单中使用 Actions 的更多信息，请参阅 [`<form>`](/reference/react-dom/components/form#handle-form-submission-with-a-server-function) 文档。

---

### 处理错误 {/*handling-errors*/}

使用 `useActionState` 处理错误有两种方式。

对于已知错误，例如后端返回的“数量不可用”验证错误，你可以将其作为 `reducerAction` 状态的一部分返回并在 UI 中显示。

对于未知错误，例如 `undefined is not a function`，你可以抛出错误。React 会取消所有排队中的 Actions，并通过从 `useActionState` hook 中重新抛出该错误来显示最近的 [Error Boundary](/reference/react/Component#catching-rendering-errors-with-an-error-boundary)。

<Sandpack>

```js src/App.js
import {useActionState, startTransition} from 'react';
import {ErrorBoundary} from 'react-error-boundary';
import {addToCart} from './api';
import Total from './Total';

function Checkout() {
  const [state, dispatchAction, isPending] = useActionState(
    async (prevState, quantity) => {
      const result = await addToCart(prevState.count, quantity);
      if (result.error) {
        // 将 API 返回的错误作为状态返回
        return {...prevState, error: `无法添加数量 ${quantity}：${result.error}`};
      }

      if (!isPending) {
        // 为第一次派发清除错误状态。
        return {count: result.count, error: null};
      }

      // 返回新的计数，以及发生的任何错误。
      return {count: result.count, error: prevState.error};


    },
    {
      count: 0,
      error: null,
    }
  );

  function handleAdd(quantity) {
    startTransition(() => {
      dispatchAction(quantity);
    });
  }

  return (
    <div className="checkout">
      <h2>结账</h2>
      <div className="row">
        <span>Eras 巡演门票</span>
        <span>
          {isPending && '🌀 '}数量：{state.count}
        </span>
      </div>
      <div className="buttons">
        <button onClick={() => handleAdd(1)}>添加 1</button>
        <button onClick={() => handleAdd(10)}>添加 10</button>
        <button onClick={() => handleAdd(NaN)}>添加 NaN</button>
      </div>
      {state.error && <div className="error">{state.error}</div>}
      <hr />
      <Total quantity={state.count} isPending={isPending} />
    </div>
  );
}



export default function App() {
  return (
    <ErrorBoundary
      fallbackRender={({resetErrorBoundary}) => (
        <div className="checkout">
          <h2>出错了</h2>
          <p>该操作无法完成。</p>
          <button onClick={resetErrorBoundary}>重试</button>
        </div>
      )}>
      <Checkout />
    </ErrorBoundary>
  );
}
```

```js src/Total.js
const formatter = new Intl.NumberFormat('en-US', {
  style: 'currency',
  currency: 'USD',
  minimumFractionDigits: 0,
});

export default function Total({quantity, isPending}) {
  return (
    <div className="row total">
      <span>总计</span>
      <span>
        {isPending ? '🌀 正在更新...' : formatter.format(quantity * 9999)}
      </span>
    </div>
  );
}
```

```js src/api.js hidden
export async function addToCart(count, quantity) {
  await new Promise((resolve) => setTimeout(resolve, 1000));
  if (quantity > 5) {
    return {error: '数量不可用'};
  } else if (isNaN(quantity)) {
    throw new Error('数量必须是数字');
  }
  return {count: count + quantity};
}
```

```css
.checkout {
  display: flex;
  flex-direction: column;
  gap: 12px;
  padding: 16px;
  border: 1px solid #ccc;
  border-radius: 8px;
  font-family: system-ui;
}

.checkout h2 {
  margin: 0 0 8px 0;
}

.row {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.total {
  font-weight: bold;
}

hr {
  width: 100%;
  border: none;
  border-top: 1px solid #ccc;
  margin: 4px 0;
}

button {
  padding: 8px 16px;
  cursor: pointer;
}

.buttons {
  display: flex;
  gap: 8px;
}

.error {
  color: red;
  font-size: 14px;
}
```

```json package.json hidden
{
  "dependencies": {
    "react": "19.0.0",
    "react-dom": "19.0.0",
    "react-scripts": "^5.0.0",
    "react-error-boundary": "4.0.3"
  },
  "main": "/index.js"
}
```

</Sandpack>

在这个示例中，“添加 10”模拟了一个返回验证错误的 API，`updateCartAction` 会把它存储到状态中并在界面内联显示。“添加 NaN”会得到一个无效计数，因此 `updateCartAction` 会抛出错误，该错误会通过 `useActionState` 传播到 `ErrorBoundary`，并显示重置界面。


---

## 故障排查 {/*troubleshooting*/}

### 我的 `isPending` 标志没有更新 {/*ispending-not-updating*/}

如果你是手动调用 `dispatchAction`（不是通过 `Action` prop），请确保你将该调用包裹在 [`startTransition`](/reference/react/startTransition) 中：

```js
import { useActionState, startTransition } from 'react';

function MyComponent() {
  const [state, dispatchAction, isPending] = useActionState(myAction, null);

  function handleClick() {
    // ✅ 正确：用 startTransition 包裹
    startTransition(() => {
      dispatchAction();
    });
  }

  // ...
}
```

当 `dispatchAction` 作为 `Action` prop 传入时，React 会自动将其包裹在 Transition 中。

---

### 我的 Action 无法读取表单数据 {/*action-cannot-read-form-data*/}

当你使用 `useActionState` 时，`reducerAction` 会在第一个参数位置额外接收一个参数：前一个状态或初始状态。因此，提交的表单数据会作为第二个参数，而不是第一个参数。

```js {2,7}
// 不使用 useActionState
function action(formData) {
  const name = formData.get('name');
}

// 使用 useActionState
function action(prevState, formData) {
  const name = formData.get('name');
}
```

---

### 我的 actions 被跳过了 {/*actions-skipped*/}

如果你多次调用 `dispatchAction`，但其中有些没有执行，可能是因为更早的一次 `dispatchAction` 调用抛出了错误。

当 `reducerAction` 抛出错误时，React 会跳过所有后续排队的 `dispatchAction` 调用。

要处理这种情况，请在你的 `reducerAction` 内部捕获错误，并返回错误状态，而不是抛出异常：

```js
async function myReducerAction(prevState, data) {
  try {
    const result = await submitData(data);
    return { success: true, data: result };
  } catch (error) {
    // ✅ 返回错误状态，而不是抛出异常
    return { success: false, error: error.message };
  }
}
```

---

### 我的状态没有重置 {/*reset-state*/}

`useActionState` 不提供内置的重置函数。要重置状态，你可以设计你的 `reducerAction` 来处理重置信号：

```js
const initialState = { name: '', error: null };

async function formAction(prevState, payload) {
  // 处理重置
  if (payload === null) {
    return initialState;
  }
  // 常规 action 逻辑
  const result = await submitData(payload);
  return result;
}

function MyComponent() {
  const [state, dispatchAction, isPending] = useActionState(formAction, initialState);

  function handleReset() {
    startTransition(() => {
      dispatchAction(null); // 传入 null 以触发重置
    });
  }

  // ...
}
```

或者，你可以给使用 `useActionState` 的组件添加一个 `key` prop，以强制它用全新的状态重新挂载；也可以使用 `<form>` 的 `action` prop，它会在提交后自动重置。

---

### 我遇到了一个错误：“An async function with useActionState was called outside of a transition.” {/*async-function-outside-transition*/}

一个常见的错误是忘记从 Transition 内部调用 `dispatchAction`：

<ConsoleBlockMulti>
<ConsoleLogLine level="error">

使用 `useActionState` 的异步函数在 Transition 外部被调用了。这很可能不是你想要的结果（例如，`isPending` 将无法正确更新）。请在 `startTransition` 内部调用返回的函数，或者将其传递给 `action` 或 `formAction` prop。

</ConsoleLogLine>
</ConsoleBlockMulti>


发生此错误是因为 `dispatchAction` 必须在 Transition 内部运行：

```js
function MyComponent() {
  const [state, dispatchAction, isPending] = useActionState(myAsyncAction, null);

  function handleClick() {
    // ❌ 错误：在 Transition 外部调用 `dispatchAction`
    dispatchAction();
  }

  // ...
}
```

要修复此问题，请将调用包裹在 [`startTransition`](/reference/react/startTransition) 中：

```js
import { useActionState, startTransition } from 'react';

function MyComponent() {
  const [state, dispatchAction, isPending] = useActionState(myAsyncAction, null);

  function handleClick() {
    // ✅ 正确：用 startTransition 包裹
    startTransition(() => {
      dispatchAction();
    });
  }

  // ...
}
```

或者，将 `dispatchAction` 传递给 `Action` prop，它会在 Transition 中被调用：

```js
function MyComponent() {
  const [state, dispatchAction, isPending] = useActionState(myAsyncAction, null);

  // ✅ 正确：action prop 会自动为你包裹在 Transition 中
  return <Button action={dispatchAction}>...</Button>;
}
```

---

### 我遇到了一个错误：“Cannot update action state while rendering” {/*cannot-update-during-render*/}

你不能在渲染期间调用 `dispatchAction`：

<ConsoleBlock level="error">

渲染期间无法更新 action 状态。

</ConsoleBlock>

这会导致一个无限循环，因为调用 `dispatchAction` 会安排一次状态更新，从而触发重新渲染，然后又再次调用 `dispatchAction`。

```js
function MyComponent() {
  const [state, dispatchAction, isPending] = useActionState(myAction, null);

  // ❌ 错误：在渲染期间调用 `dispatchAction`
  dispatchAction();

  // ...
}
```

要修复此问题，只在响应用户事件时调用 `dispatchAction`（例如表单提交或按钮点击）。
