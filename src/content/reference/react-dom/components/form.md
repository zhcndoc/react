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

[`action`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/form#action)：一个 URL 或函数。当将 URL 传递给 `action` 时，表单将表现得像 HTML 表单组件一样。当将函数传递给 `action` 时，该函数将在一个 Transition 中处理表单提交，遵循[Action 属性模式](/reference/react/useTransition#exposing-action-props-from-components)。传递给 `action` 的函数可以是异步的，并且会以单个参数被调用，该参数包含已提交表单的[表单数据](https://developer.mozilla.org/en-US/docs/Web/API/FormData)。`action` 属性可以被 `<button>`、`<input type="submit">` 或 `<input type="image">` 组件上的 `formAction` 属性覆盖。

#### 注意事项 {/*caveats*/}

* 当函数被传递给 `action` 或 `formAction` 时，无论 `method` 属性的值是什么，HTTP 方法都将是 POST。

---

## 用法 {/*usage*/}

### 在客户端处理表单提交 {/*handle-form-submission-on-the-client*/}

向 form 的 `action` 属性传递一个函数，以便在表单提交时运行该函数。[`formData`](https://developer.mozilla.org/en-US/docs/Web/API/FormData) 会作为参数传递给该函数，因此你可以访问表单提交的数据。这与传统的 [HTML action](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/form#action) 不同，后者只接受 URL。在 `action` 函数成功执行后，表单中所有非受控字段元素都会被重置。

<Sandpack>

```js src/App.js
export default function Search() {
  function search(formData) {
    const query = formData.get("query");
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

### 使用 Server Function 处理表单提交 {/*handle-form-submission-with-a-server-function*/}

渲染一个带有输入框和提交按钮的 `<form>`。将一个 Server Function（一个标记了[`'use server'`](/reference/rsc/use-server) 的函数）传递给 form 的 `action` 属性，以便在表单提交时运行该函数。

将 Server Function 传递给 `<form action>`，可以让用户在未启用 JavaScript 或代码尚未加载之前提交表单。这对网络连接慢、设备性能较弱或禁用 JavaScript 的用户很有帮助，其工作方式类似于将 URL 传递给 `action` 属性时的表单行为。

你可以使用隐藏表单字段向 `<form>` 的 action 提供数据。Server Function 将以隐藏表单字段数据作为 [`FormData`](https://developer.mozilla.org/en-US/docs/Web/API/FormData) 的实例被调用。

```jsx
import { updateCart } from './lib.js';

function AddToCart({productId}) {
  async function addToCart(formData) {
    'use server'
    const productId = formData.get('productId')
    await updateCart(productId)
  }
  return (
    <form action={addToCart}>
        <input type="hidden" name="productId" value={productId} />
        <button type="submit">添加到购物车</button>
    </form>

  );
}
```

如果不使用隐藏表单字段向 `<form>` 的 action 提供数据，你也可以调用 <CodeStep step={1}>`bind`</CodeStep> 方法来为其提供额外参数。这会将一个新参数（<CodeStep step={2}>`productId`</CodeStep>）绑定到函数上，除此之外还有作为参数传递给函数的 <CodeStep step={3}>`formData`</CodeStep>。

```jsx [[1, 8, "bind"], [2,8, "productId"], [2,4, "productId"], [3,4, "formData"]]
import { updateCart } from './lib.js';

function AddToCart({productId}) {
  async function addToCart(productId, formData) {
    "use server";
    await updateCart(productId)
  }
  const addProductToCart = addToCart.bind(null, productId);
  return (
    <form action={addProductToCart}>
      <button type="submit">添加到购物车</button>
    </form>
  );
}
```

当 `<form>` 由[Server Component](/reference/rsc/use-client) 渲染，并且有一个[Server Function](/reference/rsc/server-functions) 被传递给 `<form>` 的 `action` 属性时，该表单会进行[渐进增强](https://developer.mozilla.org/en-US/docs/Glossary/Progressive_Enhancement)。

### 在表单提交期间显示待处理状态 {/*display-a-pending-state-during-form-submission*/}
要在表单提交时显示待处理状态，你可以在 `<form>` 中渲染的组件里调用 `useFormStatus` Hook，并读取返回的 `pending` 属性。

这里，我们使用 `pending` 属性来表示表单正在提交。

<Sandpack>

```js src/App.js
import { useFormStatus } from "react-dom";
import { submitForm } from "./actions.js";

function Submit() {
  const { pending } = useFormStatus();
  return (
    <button type="submit" disabled={pending}>
      {pending ? "提交中..." : "提交"}
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
export async function submitForm(query) {
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
import { useOptimistic, useState, useRef } from "react";
import { deliverMessage } from "./actions.js";

function Thread({ messages, sendMessage }) {
  const formRef = useRef();
  async function formAction(formData) {
    addOptimisticMessage(formData.get("message"));
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
    { text: "你好呀！", sending: false, key: 1 }
  ]);
  async function sendMessage(formData) {
    const sentMessage = await deliverMessage(formData.get("message"));
    setMessages((messages) => [...messages, { text: sentMessage }]);
  }
  return <Thread messages={messages} sendMessage={sendMessage} />;
}
```

```js src/actions.js
export async function deliverMessage(message) {
  await new Promise((res) => setTimeout(res, 1000));
  return message;
}
```

</Sandpack>

[//]: # 'Uncomment the next line, and delete this line after the `useOptimistic` reference documentation page is published'
[//]: # 'To learn more about the `useOptimistic` Hook see the [reference documentation](/reference/react/useOptimistic).'

### 处理表单提交错误 {/*handling-form-submission-errors*/}

在某些情况下，由 `<form>` 的 `action` 属性调用的函数会抛出错误。你可以通过将 `<form>` 包裹在 Error Boundary 中来处理这些错误。如果由 `<form>` 的 `action` 属性调用的函数抛出错误，则会显示错误边界的回退内容。

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
    "react": "19.0.0-rc-3edc000d-20240926",
    "react-dom": "19.0.0-rc-3edc000d-20240926",
    "react-scripts": "^5.0.0",
    "react-error-boundary": "4.0.3"
  },
  "main": "/index.js",
  "devDependencies": {}
}
```

</Sandpack>

### 在没有 JavaScript 的情况下显示表单提交错误 {/*display-a-form-submission-error-without-javascript*/}

在渐进增强中，要在 JavaScript 包加载之前显示表单提交错误消息，需要满足以下条件：

1. `<form>` 由 [Client Component](/reference/rsc/use-client) 渲染
1. 传递给 `<form>` 的 `action` 属性的函数是一个 [Server Function](/reference/rsc/server-functions)
1. 使用 `useActionState` Hook 来显示错误消息

`useActionState` 接受两个参数：一个 [Server Function](/reference/rsc/server-functions) 和一个初始状态。`useActionState` 返回两个值：一个状态变量和一个 action。`useActionState` 返回的 action 应传递给表单的 `action` 属性。`useActionState` 返回的状态变量可用于显示错误消息。传递给 `useActionState` 的 Server Function 的返回值将用于更新该状态变量。

<Sandpack>

```js src/App.js
import { useActionState } from "react";
import { signUpNewUser } from "./api";

export default function Page() {
  async function signup(prevState, formData) {
    "use server";
    const email = formData.get("email");
    try {
      await signUpNewUser(email);
      alert(`已添加 "${email}"`);
    } catch (err) {
      return err.toString();
    }
  }
  const [message, signupAction] = useActionState(signup, null);
  return (
    <>
      <h1>注册我的新闻邮件</h1>
      <p>使用同一个邮箱注册两次即可看到错误</p>
      <form action={signupAction} id="signup-form">
        <label htmlFor="email">邮箱： </label>
        <input name="email" id="email" placeholder="react@example.com" />
        <button>注册</button>
        {!!message && <p>{message}</p>}
      </form>
    </>
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

了解更多关于从表单 action 更新状态的信息，请参阅 [`useActionState`](/reference/react/useActionState) 文档

### 处理多种提交类型 {/*handling-multiple-submission-types*/}

可以将表单设计为根据用户按下的按钮来处理多种提交操作。表单中的每个按钮都可以通过设置 `formAction` 属性来关联不同的 action 或行为。

当用户点击某个特定按钮时，表单会被提交，并执行该按钮属性和 action 所定义的相应操作。例如，表单可能默认提交一篇文章供审核，但有一个单独的按钮将 `formAction` 设置为把文章保存为草稿。

<Sandpack>

```js src/App.js
export default function Search() {
  function publish(formData) {
    const content = formData.get("content");
    const button = formData.get("button");
    alert(`'${content}' 已使用 '${button}' 按钮发布`);
  }

  function save(formData) {
    const content = formData.get("content");
    alert(`你的 '${content}' 草稿已保存！`);
  }

  return (
    <form action={publish}>
      <textarea name="content" rows={4} cols={40} />
      <br />
      <button type="submit" name="button" value="submit">发布</button>
      <button formAction={save}>保存草稿</button>
    </form>
  );
}
```

</Sandpack>
