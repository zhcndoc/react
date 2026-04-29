---
title: useFormStatus
---

<Intro>

`useFormStatus` 是一个 Hook，它会为你提供上一次表单提交的状态信息。

```js
const { pending, data, method, action } = useFormStatus();
```

</Intro>

<InlineToc />

---

## 参考 {/*reference*/}

### `useFormStatus()` {/*use-form-status*/}

`useFormStatus` Hook 提供上一次表单提交的状态信息。

```js {5},[[1, 6, "status.pending"]]
import { useFormStatus } from "react-dom";
import action from './actions';

function Submit() {
  const status = useFormStatus();
  return <button disabled={status.pending}>Submit</button>
}

export default function App() {
  return (
    <form action={action}>
      <Submit />
    </form>
  );
}
```

要获取状态信息，`Submit` 组件必须在 `<form>` 内部渲染。该 Hook 会返回诸如 <CodeStep step={1}>`pending`</CodeStep> 属性之类的信息，它会告诉你表单是否正在提交中。

在上面的示例中，`Submit` 使用这些信息在表单提交时禁用 `<button>` 点击。

[查看更多示例。](#usage)

#### 参数 {/*parameters*/}

`useFormStatus` 不接受任何参数。

#### 返回值 {/*returns*/}

一个包含以下属性的 `status` 对象：

* `pending`：布尔值。如果为 `true`，表示父级 `<form>` 正在等待提交。否则为 `false`。

* `data`：一个实现了 [`FormData interface`](https://developer.mozilla.org/en-US/docs/Web/API/FormData) 的对象，包含父级 `<form>` 正在提交的数据。如果没有正在进行的提交或没有父级 `<form>`，它将为 `null`。

* `method`：字符串值，取值为 `'get'` 或 `'post'`。表示父级 `<form>` 是通过 `GET` 还是 `POST` [HTTP method](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods) 提交。默认情况下，`<form>` 将使用 `GET` 方法，并且可以通过 [`method`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/form#method) 属性指定。

[//]: # (Link to `<form>` documentation. "Read more on the `action` prop on `<form>`.")
* `action`：传递给父级 `<form>` 的 `action` 属性的函数引用。如果没有父级 `<form>`，该属性为 `null`。如果 `action` 属性提供的是 URI 值，或者未指定 `action` 属性，则 `status.action` 将为 `null`。

#### 注意事项 {/*caveats*/}

* `useFormStatus` Hook 必须在一个渲染于 `<form>` 内部的组件中调用。
* `useFormStatus` 只会返回父级 `<form>` 的状态信息。它不会返回在同一组件或子组件中渲染的任何 `<form>` 的状态信息。

---

## 用法 {/*usage*/}

### 在表单提交期间显示待处理状态 {/*display-a-pending-state-during-form-submission*/}
要在表单提交时显示待处理状态，你可以在一个渲染于 `<form>` 内的组件中调用 `useFormStatus` Hook，并读取返回的 `pending` 属性。

这里，我们使用 `pending` 属性来指示表单正在提交。

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

<Pitfall>

##### `useFormStatus` 不会返回在同一组件中渲染的 `<form>` 的状态信息。 {/*useformstatus-will-not-return-status-information-for-a-form-rendered-in-the-same-component*/}

`useFormStatus` Hook 只会返回父级 `<form>` 的状态信息，而不会返回在调用该 Hook 的同一组件中渲染的任何 `<form>`，也不会返回子组件中的 `<form>`。

```js
function Form() {
  // 🚩 `pending` 永远不会为 true
  // useFormStatus 不会跟踪在此组件中渲染的表单
  const { pending } = useFormStatus();
  return <form action={submit}></form>;
}
```

相反，请从位于 `<form>` 内部的组件中调用 `useFormStatus`。

```js
function Submit() {
  // ✅ `pending` 将由包裹 Submit 组件的表单派生
  const { pending } = useFormStatus();
  return <button disabled={pending}>...</button>;
}

function Form() {
  // 这就是 `useFormStatus` 跟踪的 <form>
  return (
    <form action={submit}>
      <Submit />
    </form>
  );
}
```

</Pitfall>

### 读取正在提交的表单数据 {/*read-form-data-being-submitted*/}

你可以使用从 `useFormStatus` 返回的状态信息中的 `data` 属性来显示用户正在提交的数据。

这里，我们有一个表单，用户可以请求用户名。我们可以使用 `useFormStatus` 来显示一条临时状态消息，确认他们请求了哪个用户名。

<Sandpack>

```js src/UsernameForm.js active
import {useState, useMemo, useRef} from 'react';
import {useFormStatus} from 'react-dom';

export default function UsernameForm() {
  const {pending, data} = useFormStatus();

  return (
    <div>
      <h3>请求一个用户名：</h3>
      <input type="text" name="username" disabled={pending}/>
      <button type="submit" disabled={pending}>
        提交
      </button>
      <br />
      <p>{data ? `正在请求 ${data?.get("username")}...`: ''}</p>
    </div>
  );
}
```

```js src/App.js
import UsernameForm from './UsernameForm';
import { submitForm } from "./actions.js";
import {useRef} from 'react';

export default function App() {
  const ref = useRef(null);
  return (
    <form ref={ref} action={async (formData) => {
      await submitForm(formData);
      ref.current.reset();
    }}>
      <UsernameForm />
    </form>
  );
}
```

```js src/actions.js hidden
export async function submitForm(query) {
    await new Promise((res) => setTimeout(res, 2000));
}
```

```css
p {
    height: 14px;
    padding: 0;
    margin: 2px 0 0 0 ;
    font-size: 14px
}

button {
    margin-left: 2px;
}

```

</Sandpack>

---

## 故障排查 {/*troubleshooting*/}

### `status.pending` 永远不是 `true` {/*pending-is-never-true*/}

`useFormStatus` 只会返回父级 `<form>` 的状态信息。

如果调用 `useFormStatus` 的组件没有嵌套在 `<form>` 中，`status.pending` 将始终返回 `false`。请确认 `useFormStatus` 是在作为 `<form>` 元素子元素的组件中调用的。

`useFormStatus` 不会跟踪在同一组件中渲染的 `<form>` 的状态。更多细节请参见 [注意事项](#useformstatus-will-not-return-status-information-for-a-form-rendered-in-the-same-component)。
