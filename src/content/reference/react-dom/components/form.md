---
title: "<form>"
---

<Intro>

内置的浏览器 `<form>` 组件（[built-in browser `<form>` component](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/form)）可让你创建交互式控件，用于提交信息。

```js
<form action={search}>
    <input name="query" />
    <button type="submit">搜索</button>
</form>
```

</Intro>

<InlineToc />

---

## 参考 {/*reference*/}

### `<form>` {/*form*/}

要创建用于提交信息的交互式控件，请渲染内置的浏览器 `<form>` 组件（[built-in browser `<form>` component](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/form)）。

```js
<form action={search}>
    <input name="query" />
    <button type="submit">搜索</button>
</form>
```

[查看更多示例。](#usage)

#### 属性 {/*props*/}

`<form>` 支持所有[通用元素属性。](/reference/react-dom/components/common#common-props)

* [`action`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/form#action)：包含 URL 的字符串或函数。传入 URL 时，表单的行为与标准 HTML 表单相同；传入函数时，该函数会处理提交。参见[使用 action 属性处理表单提交](#handle-form-submission-with-an-action-prop)。
  * 向 `action` 传入函数时，React 会按照[Action 属性模式](/reference/react/useTransition#exposing-action-props-from-components)，在一个 [Transition](/reference/react/useTransition) 中运行它。
  * 该函数可以是异步的。React 会以一个参数调用它，该参数包含已提交表单的 [`FormData`](https://developer.mozilla.org/en-US/docs/Web/API/FormData)。
  * `<button>`、`<input type="submit">` 或 `<input type="image">` 上的 `formAction` prop 会覆盖此 `action`。
* [`method`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/form#method)：指定 `action` 为 URL 时使用的 HTTP 方法的字符串。默认为 `get`。
* `onSubmit`：[事件处理函数](/reference/react-dom/components/common#react-event-object)，在表单提交时触发。参见[使用事件处理函数处理表单提交](#handle-form-submission-with-an-event-handler)。

#### 注意事项 {/*caveats*/}

* 向 `action` 或 `formAction` 传入函数时，无论 `method` prop 的值是什么，HTTP 方法都将是 `POST`。
* 向 `action` 或 `formAction` 传入函数时，Action 成功后 React 会重置所有[非受控](/reference/react-dom/components/input#reading-the-input-values-when-submitting-a-form)字段。参见[在 Action 后保留表单值](#preserve-form-values-after-an-action)。

---

## 用法 {/*usage*/}

### 使用事件处理函数处理表单提交 {/*handle-form-submission-with-an-event-handler*/}

将一个函数传递给 `onSubmit` 事件处理函数，以便在表单提交时运行代码。默认情况下，浏览器会将表单数据发送到当前 URL 并刷新页面。在事件处理函数中调用 [`e.preventDefault()`](https://developer.mozilla.org/en-US/docs/Web/API/Event/preventDefault) 可以覆盖这种行为。

如果你同时传入一个 `action` 函数，除非 `onSubmit` 调用了 `e.preventDefault()`，否则 React 会在 `onSubmit` 之后运行该函数。

<Sandpack>

```js src/App.js
export default function Search() {
  function handleSubmit(e) {
    // 阻止浏览器重新加载页面
    e.preventDefault();

    // 读取表单数据
    const form = e.target;
    const formData = new FormData(form);
    const query = formData.get('query');
    alert(`You searched for '${query}'`);
  }

  return (
    <form onSubmit={handleSubmit}>
      <input name="query" />
      <button type="submit">Search</button>
    </form>
  );
}
```

</Sandpack>

<Note>

`onSubmit` 在 React 的每个版本中都有效，并且可以直接访问[提交事件](https://developer.mozilla.org/en-US/docs/Web/API/HTMLFormElement/submit_event)。从 React 19 开始，[`action` prop](#handle-form-submission-with-an-action-prop) 是处理表单提交的首选 API。使用 `action` 时，React 会在 [Transition](/reference/react/useTransition) 中运行提交，并管理待处理状态及多次提交之间的并发。你还可以使用乐观更新、Error Boundary 和 [Server Functions](/reference/rsc/server-functions)。

</Note>

### 使用 `action` 属性处理表单提交 {/*handle-form-submission-with-an-action-prop*/}

将一个函数传递给表单的 `action` 属性，以便在表单提交时运行该函数。React 会将包含所有带 `name` 属性的输入值的 [`FormData`](https://developer.mozilla.org/en-US/docs/Web/API/FormData) 对象传给该函数。输入框可以是[非受控](/reference/react-dom/components/input#reading-the-input-values-when-submitting-a-form)的，无需设置成对的 `value`/`onChange`、`onSubmit` 处理函数或调用 `e.preventDefault()`。

向 `action` 传入函数时，React 会：

* 在 [Transition](/reference/react/useTransition) 中运行该函数。
* 通过 [`useFormStatus`](/reference/react-dom/hooks/useFormStatus) 向子组件提供待处理状态。
* 将错误传递给最近的 Error Boundary。
* 在函数成功后重置表单中的非受控字段。若要保留字段值，请参阅[在 Action 后保留表单值](#preserve-form-values-after-an-action)。

如果你手动处理提交，可以通过 [`requestFormReset`](/reference/react-dom/requestFormReset#requesting-a-reset-in-a-custom-form-action) 请求相同的重置行为。

由于 Action 在 Transition 中运行，你还可以使用 [`useActionState`](/reference/react/useActionState) 管理表单 state，使用 [`useOptimistic`](/reference/react/useOptimistic) 实现乐观 UI。有关 [Server Functions](/reference/rsc/server-functions) 和渐进增强，请参阅[使用 Server Function 处理表单提交](#handle-form-submission-with-a-server-function)。

<Sandpack>

```js src/App.js
export default function Search() {
  function search(formData) {
    const query = formData.get('query');
    alert(`你搜索了 '${query}'`);
  }
  return (
    <form action={search}>
      <input name="query" />
      <button type="submit">搜索</button>
    </form>
  );
}
```

</Sandpack>

---

### 使用 Server Function 处理表单提交 {/*handle-form-submission-with-a-server-function*/}

`<form>` 的 `action` prop 可以运行一个 Server Function（即标记了 [`'use server'`](/reference/rsc/use-server) 的函数）。用户可以在 JavaScript 加载前或禁用 JavaScript 时提交表单，这与将 URL 传给 `action` 时的行为相同。

React 会使用表单的 [`FormData`](https://developer.mozilla.org/en-US/docs/Web/API/FormData) 调用 Server Function，其中包含所有带 `name` 属性的输入。若要额外传入数据（例如来自 props 的 ID），请参阅[向 Action 传递额外数据](#passing-extra-data-to-an-action)。

```jsx
import { addComment } from './actions.js';

function CommentForm() {
  return (
    <form action={addComment}>
      <textarea name="comment" />
      <button type="submit">Comment</button>
    </form>
  );
}
```

```js
// actions.js
'use server';

export async function addComment(formData) {
  const comment = formData.get('comment');
  // ...
}
```

<DeepDive>

#### 向 Action 传递额外数据 {/*passing-extra-data-to-an-action*/}

Action 会收到已提交的 [`FormData`](https://developer.mozilla.org/en-US/docs/Web/API/FormData)。如果要包含用户未输入的数据（例如来自 props 的 ID），可以采用以下方式：

* **隐藏字段**会把值加入 `FormData`。该值会出现在 HTML 中，且 `formData.get` 返回字符串。JavaScript 加载前也可以提交表单。
* **使用 `bind` 绑定**会将值作为参数传给 Server Function，位于 `formData` 之前。参数会保留原本类型，例如数字仍是数字。它必须是[可序列化的](/reference/rsc/use-server#serializable-parameters-and-return-values)。JavaScript 加载前也可以提交表单。
* **在组件中定义函数**可以直接读取 props 和 state。该函数会在 JavaScript 加载后运行，然后将这些值传给 Server Function。

客户端会发送隐藏字段值和绑定后的参数。请在服务端检查当前登录用户是否有权执行此 Action。参见[安全注意事项](/reference/rsc/use-server#security)。

#### 隐藏字段 {/*passing-extra-data-hidden-fields*/}

在 Server Component 中，渲染一个带有 `name` 和 `value` 的 `<input type="hidden">`。React 会将其值包含在传给 Action 的 `FormData` 中。

```jsx
// Server Component
import { updateCart } from './lib.js';

function AddToCart({ productId }) {
  async function addToCart(formData) {
    'use server';
    await updateCart(formData.get('productId'));
  }
  return (
    <form action={addToCart}>
      <input type="hidden" name="productId" value={productId} />
      <button type="submit">添加到购物车</button>
    </form>
  );
}
```

React 在提交按钮发起表单提交时，也会将该按钮的 `name` 和 `value` 包含在 `FormData` 中。

#### 使用 `bind` 绑定 {/*passing-extra-data-bind*/}

在 Server Component 中，对 Server Function 调用 [`bind`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/bind)，然后将绑定后的函数传给 `action`。将 `null` 作为 `bind` 的第一个参数。`null` 之后的每个参数都会排在 React 提交表单时传入的 `formData` 参数之前。

```jsx
// Server Component
import { updateCart } from './lib.js';

function AddToCart({ productId }) {
  async function addToCart(productId, formData) {
    'use server';
    await updateCart(productId);
  }
  const addProductToCart = addToCart.bind(null, productId);
  return (
    <form action={addProductToCart}>
      <button type="submit">添加到购物车</button>
    </form>
  );
}
```

表单提交时，React 会调用 `addProductToCart(formData)`，后者再调用 `addToCart(productId, formData)`。

如果同时使用 [`useActionState`](/reference/react/useActionState)，Server Function 会依次收到 `productId`、`previousState` 和 `formData`。请将该函数定义在单独的文件中，并从 Client Component 绑定它。

#### 在组件中定义函数 {/*passing-extra-data-function*/}

在 Client Component 中定义传给 `action` 的函数，这样它就能直接读取组件的 props 或 state。该函数随后会将这些值传给 Server Function：

```jsx
// AddToCart.js
'use client';

import { updateCart } from './actions.js';

function AddToCart({ productId }) {
  async function addToCart(formData) {
    await updateCart(productId, formData);
  }
  return (
    <form action={addToCart}>
      <button type="submit">添加到购物车</button>
    </form>
  );
}
```

由于 `addToCart` 在浏览器中运行，表单必须等到 JavaScript 加载后才能提交。

</DeepDive>

### 在表单提交期间显示待处理状态 {/*display-a-pending-state-during-form-submission*/}
要在表单提交时显示待处理状态，你可以在 `<form>` 中渲染的组件里调用 `useFormStatus` Hook，并读取返回的 `pending` 属性。

这里，我们使用 `pending` 属性来表示表单正在提交。

<Sandpack>

```js src/App.js
import { useFormStatus } from 'react-dom';
import { submitForm } from './actions.js';

function Submit() {
  const { pending } = useFormStatus();
  return (
    <button type="submit" disabled={pending}>
      {pending ? '提交中...' : '提交'}
    </button>
  );
}

function Form({ action }) {
  return (
    <form action={action}>
      <Submit />
    </form>
  );
}

export default function App() {
  return <Form action={submitForm} />;
}
```

```js src/actions.js hidden
export async function submitForm(formData) {
  await new Promise((res) => setTimeout(res, 1000));
}
```

</Sandpack>

要了解更多关于 `useFormStatus` Hook 的内容，请参见[参考文档](/reference/react-dom/hooks/useFormStatus)。

### 乐观更新表单数据 {/*optimistically-updating-form-data*/}
`useOptimistic` Hook 提供了一种在后台操作（如网络请求）完成之前，乐观更新用户界面的方式。在表单场景中，这种技术有助于让应用感觉更响应迅速。当用户提交表单时，界面会立即根据预期结果进行更新，而不是等待服务器响应后再反映变化。

例如，当用户在表单中输入消息并点击“发送”按钮时，`useOptimistic` Hook 允许该消息立即出现在列表中，并带有“发送中...”标签，甚至在消息真正发送到服务器之前也是如此。这种“乐观”做法会给人一种速度快、响应快的印象。随后表单会在后台尝试真正发送该消息。一旦服务器确认消息已接收，“发送中...”标签就会被移除。

<Sandpack>


```js src/App.js
import { startTransition, useOptimistic, useState, useRef } from 'react';
import { deliverMessage } from './actions.js';

function Thread({ messages, sendMessage }) {
  const formRef = useRef(null);
  async function formAction(formData) {
    addOptimisticMessage(formData.get('message'));
    formRef.current.reset();
    await sendMessage(formData);
  }
  const [optimisticMessages, addOptimisticMessage] = useOptimistic(
    messages,
    (state, newMessage) => [
      ...state,
      {
        text: newMessage,
        sending: true
      }
    ]
  );

  return (
    <>
      {optimisticMessages.map((message, index) => (
        <div key={index}>
          {message.text}
          {!!message.sending && <small>（发送中...）</small>}
        </div>
      ))}
      <form action={formAction} ref={formRef}>
        <input type="text" name="message" placeholder="你好！" />
        <button type="submit">发送</button>
      </form>
    </>
  );
}

export default function App() {
  const [messages, setMessages] = useState([
    { text: '你好呀！', sending: false, key: 1 }
  ]);
  async function sendMessage(formData) {
    const sentMessage = await deliverMessage(formData.get('message'));
    startTransition(() => {
      setMessages((messages) => [...messages, { text: sentMessage }]);
    });
  }
  return <Thread messages={messages} sendMessage={sendMessage} />;
}
```

```js src/actions.js
export async function deliverMessage(message) {
  // Simulate a message delivery delay
  await new Promise((res) => setTimeout(res, 1000));
  return message;
}
```

</Sandpack>

有关 `useOptimistic` Hook 的更多信息，请参阅[参考文档](/reference/react/useOptimistic)。

### 处理表单提交错误 {/*handling-form-submission-errors*/}

要处理传给 `<form>` `action` 属性的函数抛出的错误，请将表单包裹在 [Error Boundary](/reference/react/Component#catching-rendering-errors-with-an-error-boundary) 中。函数抛出错误时，React 会显示该边界的回退内容。

<Sandpack>

```js src/App.js
import { ErrorBoundary } from "react-error-boundary";

export default function Search() {
  function search() {
    throw new Error("搜索错误");
  }
  return (
    <ErrorBoundary
      fallback={<p>提交表单时出错了</p>}
    >
      <form action={search}>
        <input name="query" />
        <button type="submit">搜索</button>
      </form>
    </ErrorBoundary>
  );
}

```

```json package.json hidden
{
  "dependencies": {
    "react": "latest",
    "react-dom": "latest",
    "react-scripts": "latest",
    "react-error-boundary": "4.0.3"
  },
  "main": "/index.js"
}
```

</Sandpack>

#### 处理 JavaScript 加载前的表单提交错误 {/*display-a-form-submission-error-without-javascript*/}

在渐进增强中，要在 JavaScript 包加载之前显示表单提交错误消息，需要满足以下条件：

1. `<form>` 由 [Client Component](/reference/rsc/use-client) 渲染
1. 传递给 `<form>` 的 `action` 属性的函数是一个 [Server Function](/reference/rsc/server-functions)
1. 使用 [`useActionState`](/reference/react/useActionState) Hook 来显示错误消息

在单独的文件中使用 [`'use server'`](/reference/rsc/use-server) 指令定义 Server Function。它会先收到上一次的 state，再收到已提交的 `FormData`：

<Sandpack>

```js src/App.js
// actions.js
'use server';

import { signUpNewUser } from './api.js';

export async function signup(previousState, formData) {
  const email = formData.get('email');
  try {
    await signUpNewUser(email);
    return null;
  } catch (error) {
    return error.message;
  }
}
```

在 Client Component 中，将 Server Function 传给 `useActionState`。把返回的 Action 传给表单的 `action` prop，并渲染返回的 state：

```js src/Signup.js
// Signup.js
'use client';

import { useActionState } from 'react';
import { signup } from './actions.js';

export default function Signup() {
  const [message, signupAction] = useActionState(signup, null);
  return (
    <form action={signupAction} id="signup-form">
      <label htmlFor="email">邮箱： </label>
      <input name="email" id="email" placeholder="react@example.com" />
      <button>注册</button>
      {!!message && <p>{message}</p>}
    </form>
  );
}
```

```js src/api.js hidden
let emails = [];

export async function signUpNewUser(newEmail) {
  if (emails.includes(newEmail)) {
    throw new Error("此电子邮件地址已被添加");
  }
  emails.push(newEmail);
}
```

</Sandpack>

如果表单在 JavaScript 加载前提交，React 会在服务端渲染的响应中包含 Server Function 返回的错误消息。

---

### 在 Action 后保留表单值 {/*preserve-form-values-after-an-action*/}

传给 `action` 或 `formAction` 的函数成功完成后，React 会重置表单中的[非受控字段](/reference/react-dom/components/input#reading-the-input-values-when-submitting-a-form)。保存编辑内容或显示[验证错误](/reference/react/useActionState#displaying-validation-errors-and-preserving-form-values)时，你可能希望保留已提交的值。

你可以使用 [`useState`](/reference/react/useState) 保存这些值，并将它们传给各字段的 [`defaultValue`](/reference/react-dom/components/input#providing-an-initial-value-for-an-input)。Action 完成后更新 state。React 会先应用更新后的默认值，再重置表单，因此字段会保留这些值。

试着更改标题并点击 **保存**。此示例使用 [`useFormStatus`](/reference/react-dom/hooks/useFormStatus) 在模拟的一秒请求期间显示“保存中...”，随后 `setTitle` 会更新保存的标题并保留输入框中的值。

<Sandpack>

```js src/App.js
import { startTransition, useState } from 'react';
import SaveButton from './SaveButton.js';

export default function EditForm() {
  const [title, setTitle] = useState('My draft');

  async function saveAction(formData) {
    await new Promise((res) => setTimeout(res, 1000));
    startTransition(() => {
      setTitle(formData.get('title'));
    });
  }

  return (
    <form action={saveAction}>
      <input name="title" defaultValue={title} />
      <SaveButton />
      <p>Saved title: {title}</p>
    </form>
  );
}
```

```js src/SaveButton.js hidden
import { useFormStatus } from 'react-dom';

export default function SaveButton() {
  const { pending } = useFormStatus();
  return (
    <button type="submit" disabled={pending}>
      {pending ? 'Saving...' : 'Save'}
    </button>
  );
}
```

</Sandpack>

`await` 之后的 `setTitle` 需要再包裹一次 [`startTransition`](/reference/react/startTransition)，才能将其标记为 Transition 更新。

`saveAction` 也可以调用 [Server Function](/reference/rsc/server-functions)，并保存其返回值：

```js
// actions.js
'use server';

import { saveTitle } from './api.js';

export async function submitTitle(formData) {
  const title = formData.get('title');
  await saveTitle(title);
  return title;
}
```

在组件文件顶部添加 [`'use client'`](/reference/rsc/use-client) 指令，导入 `submitTitle` 并替换 `saveAction`，表单其余部分可以保持不变：

```js
import { submitTitle } from './actions.js';

async function saveAction(formData) {
  const savedTitle = await submitTitle(formData);
  startTransition(() => {
    setTitle(savedTitle);
  });
}
```

#### JavaScript 加载前保留表单值 {/*preserving-form-values-before-javascript-loads*/}

配合 [`useActionState`](/reference/react/useActionState) 使用时，React 可以在 hydration 完成之前显示 [Server Function](/reference/rsc/server-functions) 的返回值。表单可以将该值作为新的默认值。有关在 hydration 前显示服务器返回的验证错误，请参阅[处理 JavaScript 加载前的表单提交错误](#display-a-form-submission-error-without-javascript)。

<DeepDive>

#### 控制表单重置的其他方式 {/*other-ways-to-control-form-resets*/}

上面的示例通过在重置前保存每个已提交的值来保留它们。你也可以选择保留哪些值，或控制何时重置：

* **只在验证失败时保留值。** 每次提交都返回相同形状的 state。验证失败时，包含已提交的 `FormData`，并通过 `state.submitted?.get('title') ?? ''` 读取字段。验证成功时，将 `FormData` 设为 `null`，让重置清空这些字段。参见 [`useActionState`](/reference/react/useActionState#displaying-validation-errors-and-preserving-form-values) 中的示例。
* **使用 `onSubmit` 跳过自动重置。** 在 `onSubmit` 中处理提交，调用 `e.preventDefault()`，并在 [`startTransition`](/reference/react/useTransition) 中运行 Action。由于表单的 `action` 没有运行，React 不会重置表单。
* **在指定时机重置。** 调用表单元素的 [`reset()`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLFormElement/reset)，立即将非受控字段重置为默认值。若要将重置安排为 Action 或 Transition 的一部分，请从 `react-dom` 调用 [`requestFormReset`](/reference/react-dom/requestFormReset)。
* **同时重置表单和组件 state。** 更改渲染表单的组件的 [`key`](/learn/preserving-and-resetting-state#resetting-a-form-with-a-key)。React 会重新创建该组件及其 DOM，从而重置字段和本地 state。

</DeepDive>

---

### 处理多种提交类型 {/*handling-multiple-submission-types*/}

一个表单可以包含多个提交按钮，每个按钮运行不同的 Action。未设置 `formAction` 的按钮会运行表单的 `action`；设置了 `formAction` 的按钮则会运行自己的 Action。例如，下面的表单默认发布文章，但 **Save draft** 按钮会保存当前内容而不发布。它使用 [`useActionState`](/reference/react/useActionState) 保留已保存的草稿：

<Sandpack>

```js src/App.js
import { useActionState } from 'react';

export default function ArticleForm() {
  // Preserve the saved draft after submission
  const [formState, dispatchFormState] = useActionState((state, payload) => {
    const content = payload.data.get('content');
    switch (payload.type) {
      case 'save':
        alert(`Your draft of '${content}' was saved!`);
        return payload.data;
      case 'publish':
        alert(`'${content}' was published!`);
        return new FormData();
      default:
        return state;
    }
  }, new FormData());

  function publish(formData) {
    dispatchFormState({
      type: 'publish',
      data: formData,
    });
  }

  function save(formData) {
    dispatchFormState({
      type: 'save',
      data: formData,
    });
  }

  return (
    <form action={publish}>
      <textarea
        name="content"
        rows={4}
        cols={40}
        defaultValue={formState?.get('content') || ''}
      />
      <br />
      <button type="submit" name="button" value="submit">Publish</button>
      <button formAction={save}>Save draft</button>
    </form>
  );
}
```

</Sandpack>

---

## 故障排查 {/*troubleshooting*/}

### 提交后表单字段被清空 {/*my-form-fields-are-cleared-after-submission*/}

传给 `action` 或 `formAction` 的函数成功完成时，React 会将表单的非受控字段重置为默认值。没有 `defaultValue` 的文本字段会变为空。即使函数返回验证错误而没有抛出错误，表单仍会重置。

若要保留已提交的值，请将它存入 state 并传给字段的 `defaultValue`。示例请参阅[在 Action 后保留表单值](#preserve-form-values-after-an-action)，或参阅[验证失败时保留表单值](/reference/react/useActionState#displaying-validation-errors-and-preserving-form-values)。
