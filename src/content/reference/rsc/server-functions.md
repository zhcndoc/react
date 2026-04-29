---
title: Server Functions
---

<RSC>

Server Functions 用于 [React Server Components](/reference/rsc/server-components)。

**注意：** 直到 2024 年 9 月，我们一直将所有 Server Functions 称为 "Server Actions"。如果一个 Server Function 作为 action prop 传递，或者在 action 内部被调用，那么它就是一个 Server Action，但并非所有 Server Functions 都是 Server Actions。本文档中的命名已更新，以反映 Server Functions 可以用于多种用途。

</RSC>

<Intro>

Server Functions 允许 Client Components 调用在服务器上执行的 async 函数。

</Intro>

<InlineToc />

<Note>

#### 如何为 Server Functions 构建支持？ {/*how-do-i-build-support-for-server-functions*/}

虽然 React 19 中的 Server Functions 是稳定的，并且在 minor 版本之间不会破坏，但用于在 React Server Components bundler 或 framework 中实现 Server Functions 的底层 API 不遵循 semver，并且在 React 19.x 的 minor 版本之间可能会发生破坏性变化。

要以 bundler 或 framework 的形式支持 Server Functions，我们建议固定到某个特定的 React 版本，或者使用 Canary 版本。我们将继续与 bundlers 和 frameworks 合作，未来使用于实现 Server Functions 的 API 稳定下来。

</Note>

当一个 Server Function 使用 [`"use server"`](/reference/rsc/use-server) 指令定义时，你的 framework 会自动创建该 Server Function 的引用，并将该引用传递给 Client Component。当该函数在客户端被调用时，React 会向服务器发送请求以执行该函数，并返回结果。

Server Functions 可以在 Server Components 中创建并作为 props 传递给 Client Components，也可以被导入并在 Client Components 中使用。

## 用法 {/*usage*/}

### 从 Server Component 创建 Server Function {/*creating-a-server-function-from-a-server-component*/}

Server Components 可以使用 `"use server"` 指令定义 Server Functions：

```js [[2, 7, "'use server'"], [1, 5, "createNoteAction"], [1, 12, "createNoteAction"]]
// Server Component
import Button from './Button';

function EmptyNote () {
  async function createNoteAction() {
    // Server Function
    'use server';

    await db.notes.create();
  }

  return <Button onClick={createNoteAction}/>;
}
```

当 React 渲染 `EmptyNote` Server Component 时，它会创建 `createNoteAction` 函数的引用，并将该引用传递给 `Button` Client Component。点击按钮时，React 会向服务器发送请求，使用提供的引用执行 `createNoteAction` 函数：

```js {5}
"use client";

export default function Button({onClick}) {
  console.log(onClick);
  // {$$typeof: Symbol.for("react.server.reference"), $$id: 'createNoteAction'}
  return <button onClick={() => onClick()}>创建空笔记</button>
}
```

更多内容请参阅 [`"use server"`](/reference/rsc/use-server) 文档。


### 从 Client Component 导入 Server Functions {/*importing-server-functions-from-client-components*/}

Client Components 可以从使用 `"use server"` 指令的文件中导入 Server Functions：

```js [[1, 3, "createNote"]]
"use server";

export async function createNote() {
  await db.notes.create();
}

```

当 bundler 构建 `EmptyNote` Client Component 时，它会在 bundle 中为 `createNote` 函数创建一个引用。点击 `button` 时，React 会使用提供的引用向服务器发送请求，以执行 `createNote` 函数：

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

### 带有 Actions 的 Server Functions {/*server-functions-with-actions*/}

Server Functions 可以在客户端的 Actions 中被调用：

```js [[1, 3, "updateName"]]
"use server";

export async function updateName(name) {
  if (!name) {
    return {error: '需要姓名'};
  }
  await db.users.updateName(name);
}
```

```js [[1, 3, "updateName"], [1, 13, "updateName"], [2, 11, "submitAction"],  [2, 23, "submitAction"]]
"use client";

import {updateName} from './actions';

function UpdateName() {
  const [name, setName] = useState('');
  const [error, setError] = useState(null);

  const [isPending, startTransition] = useTransition();

  const submitAction = async () => {
    startTransition(async () => {
      const {error} = await updateName(name);
      if (error) {
        setError(error);
      } else {
        setName('');
      }
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

这使你可以通过在客户端将其包装在一个 Action 中，来访问 Server Function 的 `isPending` 状态。

更多内容请参阅 [在 `<form>` 外调用 Server Function](/reference/rsc/use-server#calling-a-server-function-outside-of-form) 文档

### 带有 Form Actions 的 Server Functions {/*using-server-functions-with-form-actions*/}

Server Functions 可与 React 19 中新的 Form 特性配合使用。

你可以将 Server Function 传递给 Form，以自动将表单提交到服务器：


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

当 Form 提交成功时，React 会自动重置表单。你可以添加 `useActionState` 来访问 pending 状态、最后一次响应，或支持渐进增强。

更多内容请参阅 [Forms 中的 Server Functions](/reference/rsc/use-server#server-functions-in-forms) 文档。

### 与 `useActionState` 配合使用的 Server Functions {/*server-functions-with-use-action-state*/}

对于只需要访问 action 的 pending 状态和最后返回响应的常见场景，你可以使用 `useActionState` 调用 Server Functions：

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

在将 `useActionState` 与 Server Functions 一起使用时，React 还会自动回放在 hydration 完成前提交的表单。这意味着用户甚至可以在应用完成 hydration 之前与你的应用交互。

更多内容请参阅 [`useActionState`](/reference/react/useActionState) 文档。

### 使用 `useActionState` 的渐进增强 {/*progressive-enhancement-with-useactionstate*/}

Server Functions 也支持通过 `useActionState` 的第三个参数实现渐进增强。

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

当将 <CodeStep step={2}>permalink</CodeStep> 提供给 `useActionState` 时，如果表单在 JavaScript bundle 加载前就被提交，React 会重定向到提供的 URL。

更多内容请参阅 [`useActionState`](/reference/react/useActionState) 文档。
