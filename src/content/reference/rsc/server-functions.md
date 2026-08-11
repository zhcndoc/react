---
title: 服务器函数
---

<RSC>

服务器函数用于 [React 服务器组件](/reference/rsc/server-components)。

**注意：** 直到 2024 年 9 月，我们一直将所有服务器函数称为“服务器操作”。如果一个服务器函数作为 `action` 属性传递，或者在操作内部被调用，那么它就是一个服务器操作，但并非所有服务器函数都是服务器操作。本文档中的命名已更新，以反映服务器函数可以用于多种用途。

</RSC>

<Intro>

服务器函数允许客户端组件调用在服务器上执行的异步函数。

</Intro>

<InlineToc />

<Note>

#### 如何为服务器函数构建支持？ {/*how-do-i-build-support-for-server-functions*/}

虽然 React 19 中的服务器函数是稳定的，并且在次版本之间不会破坏，但用于在 React 服务器组件打包器或框架中实现服务器函数的底层 API 不遵循 semver，并且在 React 19.x 的次版本之间可能会发生破坏性变化。

要以打包器或框架的形式支持服务器函数，我们建议固定到某个特定的 React 版本，或者使用 Canary 版本。我们将继续与打包器和框架合作，使未来用于实现服务器函数的 API 稳定下来。

</Note>

当一个服务器函数使用 [`"use server"`](/reference/rsc/use-server) 指令定义时，你的框架会自动创建该服务器函数的引用，并将该引用传递给客户端组件。当该函数在客户端被调用时，React 会向服务器发送请求以执行该函数，并返回结果。

服务器函数可以在服务器组件中创建并作为属性传递给客户端组件，也可以被导入并在客户端组件中使用。

## 用法 {/*usage*/}

### 从服务器组件创建服务器函数 {/*creating-a-server-function-from-a-server-component*/}

服务器组件可以使用 `"use server"` 指令定义服务器函数：

```js [[2, 7, "'use server'"], [1, 5, "createNoteAction"], [1, 12, "createNoteAction"]]
// 服务器组件
import Button from './Button';

function EmptyNote () {
  async function createNoteAction() {
    // 服务器函数
    'use server';

    await db.notes.create();
  }

  return <Button onClick={createNoteAction}/>;
}
```

当 React 渲染 `EmptyNote` 服务器组件时，它会创建 `createNoteAction` 函数的引用，并将该引用传递给 `Button` 客户端组件。点击按钮时，React 会向服务器发送请求，使用提供的引用执行 `createNoteAction` 函数：

```js {5}
"use client";

export default function Button({onClick}) {
  console.log(onClick);
  // {$$typeof: Symbol.for("react.server.reference"), $$id: 'createNoteAction'}
  return <button onClick={() => onClick()}>创建空笔记</button>
}
```

更多内容请参阅 [`"use server"`](/reference/rsc/use-server) 文档。


### 从客户端组件导入服务器函数 {/*importing-server-functions-from-client-components*/}

客户端组件可以从使用 `"use server"` 指令的文件中导入服务器函数：

```js [[1, 3, "createNote"]]
"use server";

export async function createNote() {
  await db.notes.create();
}

```

当打包工具构建 `EmptyNote` 客户端组件时，它会在包中为 `createNote` 函数创建一个引用。点击 `button` 时，React 会使用提供的引用向服务器发送请求，以执行 `createNote` 函数：

```js [[1, 2, "createNote"], [1, 5, "createNote"], [1, 7, "createNote"]]
"use client";
import {createNote} from './actions';

function EmptyNote() {
  console.log(createNote);
  // {$$typeof: Symbol.for("react.server.reference"), $$id: 'createNote'}
  <button onClick={() => createNote()} />
}
```

更多内容请参阅 [`"use server"`](/reference/rsc/use-server) 文档。

### 带有 Actions 的服务器函数 {/*server-functions-with-actions*/}

服务器函数可以在客户端的 Actions 中被调用：

```js [[1, 3, "updateName"]]
"use server";

export async function updateName(name) {
  if (!name) {
    return {error: '需要姓名'};
  }
  await db.users.updateName(name);
}
```

```js [[1, 3, "updateName"], [1, 13, "updateName"], [2, 11, "submitAction"], [2, 25, "submitAction"]]
"use client";

import {updateName} from './actions';

function UpdateName() {
  const [name, setName] = useState('');
  const [error, setError] = useState(null);

  const [isPending, startTransition] = useTransition();

  const submitAction = async () => {
    startTransition(async () => {
      const {error} = await updateName(name);
      startTransition(() => {
        if (error) {
          setError(error);
        } else {
          setName('');
        }
      });
    })
  }

  return (
    <form action={submitAction}>
      <input type="text" name="name" disabled={isPending}/>
      {error && <span>失败：{error}</span>}
    </form>
  )
}
```

这使你可以通过在客户端将其包装在一个 Action 中，来访问服务器函数的 `isPending` 状态。

更多内容请参阅[在 `<form>` 外调用服务器函数](/reference/rsc/use-server#calling-a-server-function-outside-of-form) 文档

### 带有表单 Actions 的服务器函数 {/*using-server-functions-with-form-actions*/}

服务器函数可与 React 19 中新的表单特性配合使用。

你可以将服务器函数传递给表单，以自动将表单提交到服务器：


```js [[1, 3, "updateName"], [1, 7, "updateName"]]
"use client";

import {updateName} from './actions';

function UpdateName() {
  return (
    <form action={updateName}>
      <input type="text" name="name" />
    </form>
  )
}
```

当表单提交成功时，React 会自动重置表单。你可以添加 `useActionState` 来访问待处理状态、最后一次响应，或支持渐进增强。

更多内容请参阅[表单中的服务器函数](/reference/rsc/use-server#server-functions-in-forms) 文档。

### 与 `useActionState` 配合使用的服务器函数 {/*server-functions-with-use-action-state*/}

对于只需要访问 action 的待处理状态和最后返回响应的常见场景，你可以使用 `useActionState` 调用服务器函数：

```js [[1, 3, "updateName"], [1, 6, "updateName"], [2, 6, "submitAction"], [2, 9, "submitAction"]]
"use client";

import {updateName} from './actions';

function UpdateName() {
  const [state, submitAction, isPending] = useActionState(updateName, {error: null});

  return (
    <form action={submitAction}>
      <input type="text" name="name" disabled={isPending}/>
      {state.error && <span>失败：{state.error}</span>}
    </form>
  );
}
```

在将 `useActionState` 与服务器函数一起使用时，React 还会自动回放在水合完成前提交的表单。这意味着用户甚至可以在应用完成水合之前与你的应用交互。

更多内容请参阅 [`useActionState`](/reference/react/useActionState) 文档。

### 使用 `useActionState` 的渐进增强 {/*progressive-enhancement-with-useactionstate*/}

服务器函数也支持通过 `useActionState` 的第三个参数实现渐进增强。

```js [[1, 3, "updateName"], [1, 6, "updateName"], [2, 6, "/name/update"], [3, 6, "submitAction"], [3, 9, "submitAction"]]
"use client";

import {updateName} from './actions';

function UpdateName() {
  const [, submitAction] = useActionState(updateName, null, `/name/update`);

  return (
    <form action={submitAction}>
      ...
    </form>
  );
}
```

当将 <CodeStep step={2}>永久链接</CodeStep> 提供给 `useActionState` 时，如果表单在 JavaScript 包加载前就被提交，React 会重定向到提供的 URL。

更多内容请参阅 [`useActionState`](/reference/react/useActionState) 文档。
