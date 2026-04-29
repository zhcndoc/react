---
title: "React v19"
author: The React Team
date: 2024/12/05
description: React 19 现已在 npm 上可用！在这篇文章中，我们将概述 React 19 的新特性，以及你如何采用它们。
---

2024 年 12 月 05 日，由 [The React Team](/community/team) 发布

---
<Note>

### React 19 现已稳定发布！ {/*react-19-is-now-stable*/}

自这篇文章最初与 4 月份的 React 19 RC 一起分享以来的新增内容：

- **对已挂起树的预热**：请参见 [Suspense 的改进](/blog/2024/04/25/react-19-upgrade-guide#improvements-to-suspense)。
- **React DOM 静态 API**：请参见 [新的 React DOM 静态 API](#new-react-dom-static-apis)。

_本文日期已更新，以反映稳定版发布日期。_

</Note>

<Intro>

React v19 现已在 npm 上可用！

</Intro>

在我们的 [React 19 升级指南](/blog/2024/04/25/react-19-upgrade-guide) 中，我们提供了将你的应用升级到 React 19 的逐步说明。在这篇文章中，我们将概述 React 19 的新特性，以及你如何采用它们。

- [React 19 中的新内容](#whats-new-in-react-19)
- [React 19 中的改进](#improvements-in-react-19)
- [如何升级](#how-to-upgrade)

有关破坏性变更列表，请参见 [升级指南](/blog/2024/04/25/react-19-upgrade-guide)。

---

## React 19 中的新内容 {/*whats-new-in-react-19*/}

### Actions {/*actions*/}

React 应用中的一个常见用例是执行数据变更，然后响应式地更新状态。例如，当用户提交表单以更改姓名时，你会发起一个 API 请求，然后处理响应。过去，你需要手动处理待处理状态、错误、乐观更新以及顺序请求。

例如，你可以在 `useState` 中处理待处理和错误状态：

```js
// 在 Actions 之前
function UpdateName({}) {
  const [name, setName] = useState("");
  const [error, setError] = useState(null);
  const [isPending, setIsPending] = useState(false);

  const handleSubmit = async () => {
    setIsPending(true);
    const error = await updateName(name);
    setIsPending(false);
    if (error) {
      setError(error);
      return;
    }
    redirect("/path");
  };

  return (
    <div>
      <input value={name} onChange={(event) => setName(event.target.value)} />
      <button onClick={handleSubmit} disabled={isPending}>
        更新
      </button>
      {error && <p>{error}</p>}
    </div>
  );
}
```

在 React 19 中，我们增加了在转场中使用 async 函数的支持，以自动处理待处理状态、错误、表单和乐观更新。

例如，你可以使用 `useTransition` 来为你处理待处理状态：

```js
// 使用来自 Actions 的待处理状态
function UpdateName({}) {
  const [name, setName] = useState("");
  const [error, setError] = useState(null);
  const [isPending, startTransition] = useTransition();

  const handleSubmit = () => {
    startTransition(async () => {
      const error = await updateName(name);
      if (error) {
        setError(error);
        return;
      }
      redirect("/path");
    })
  };

  return (
    <div>
      <input value={name} onChange={(event) => setName(event.target.value)} />
      <button onClick={handleSubmit} disabled={isPending}>
        更新
      </button>
      {error && <p>{error}</p>}
    </div>
  );
}
```

这个 async 转场会立即将 `isPending` 状态设为 true，发起 async 请求，并在所有转场完成后将 `isPending` 切换为 false。这使你能够在数据变化时保持当前 UI 的响应性和交互性。

<Note>

#### 按照约定，使用 async 转场的函数称为 "Actions"。 {/*by-convention-functions-that-use-async-transitions-are-called-actions*/}

Actions 会自动替你管理数据提交：

- **待处理状态**：Actions 提供一种待处理状态，它从请求开始时启动，并在最终状态更新提交后自动重置。
- **乐观更新**：Actions 支持新的 [`useOptimistic`](#new-hook-optimistic-updates) 钩子，因此你可以在请求提交期间向用户显示即时反馈。
- **错误处理**：Actions 提供错误处理，因此当请求失败时你可以显示错误边界，并自动将乐观更新还原为其原始值。
- **表单**：`<form>` 元素现在支持将函数传递给 `action` 和 `formAction` 属性。将函数传递给 `action` 属性默认会使用 Actions，并在提交后自动重置表单。

</Note>

在 Actions 的基础上，React 19 引入了 [`useOptimistic`](#new-hook-optimistic-updates) 来管理乐观更新，以及一个新的钩子 [`React.useActionState`](#new-hook-useactionstate) 来处理 Actions 的常见场景。在 `react-dom` 中，我们增加了 [`<form>` Actions](#form-actions) 来自动管理表单，以及 [`useFormStatus`](#new-hook-useformstatus) 来支持表单中 Actions 的常见场景。

在 React 19 中，上面的示例可以简化为：

```js
// 使用 <form> Actions 和 useActionState
function ChangeName({ name, setName }) {
  const [error, submitAction, isPending] = useActionState(
    async (previousState, formData) => {
      const error = await updateName(formData.get("name"));
      if (error) {
        return error;
      }
      redirect("/path");
      return null;
    },
    null,
  );

  return (
    <form action={submitAction}>
      <input type="text" name="name" />
      <button type="submit" disabled={isPending}>更新</button>
      {error && <p>{error}</p>}
    </form>
  );
}
```

在下一节中，我们将拆解 React 19 中每个新的 Action 特性。

### 新的钩子：`useActionState` {/*new-hook-useactionstate*/}

为了让 Actions 的常见场景更容易使用，我们新增了一个名为 `useActionState` 的钩子：

```js
const [error, submitAction, isPending] = useActionState(
  async (previousState, newName) => {
    const error = await updateName(newName);
    if (error) {
      // 你可以返回 action 的任意结果。
      // 这里我们只返回错误。
      return error;
    }

    // 处理成功
    return null;
  },
  null,
);
```

`useActionState` 接受一个函数（即 "Action"），并返回一个可调用的包装后的 Action。这之所以可行，是因为 Actions 是可组合的。当包装后的 Action 被调用时，`useActionState` 会将该 Action 的最后结果作为 `data` 返回，并将该 Action 的待处理状态作为 `pending` 返回。

<Note>

`React.useActionState` 在 Canary 版本中之前叫做 `ReactDOM.useFormState`，但我们已经将其重命名并弃用了 `useFormState`。

更多信息请参见 [#28491](https://github.com/facebook/react/pull/28491)。

</Note>

有关更多信息，请参见 [`useActionState`](/reference/react/useActionState) 文档。

### React DOM：`<form>` Actions {/*form-actions*/}

Actions 也已与 React 19 在 `react-dom` 中新增的 `<form>` 特性集成。我们增加了支持将函数作为 `<form>`、`<input>` 和 `<button>` 元素的 `action` 和 `formAction` 属性传递，从而使用 Actions 自动提交表单：

```js [[1,1,"actionFunction"]]
<form action={actionFunction}>
```

当 `<form>` Action 成功时，React 会自动重置非受控组件的表单。如果你需要手动重置 `<form>`，可以调用新的 `requestFormReset` React DOM API。

有关更多信息，请参见 `react-dom` 文档中的 [`<form>`](/reference/react-dom/components/form)、[`<input>`](/reference/react-dom/components/input) 和 `<button>`。

### React DOM：新的钩子：`useFormStatus` {/*new-hook-useformstatus*/}

在设计系统中，编写需要访问其所在 `<form>` 信息的设计组件很常见，而不必通过 props 一层层传递到组件。这可以通过 Context 实现，但为了让常见场景更容易，我们新增了一个 `useFormStatus` 钩子：

```js [[1, 4, "pending"], [1, 5, "pending"]]
import {useFormStatus} from 'react-dom';

function DesignButton() {
  const {pending} = useFormStatus();
  return <button type="submit" disabled={pending} />
}
```

`useFormStatus` 会读取父级 `<form>` 的状态，就像该表单是一个 Context 提供者一样。

有关更多信息，请参见 `react-dom` 文档中的 [`useFormStatus`](/reference/react-dom/hooks/useFormStatus)。

### 新的钩子：`useOptimistic` {/*new-hook-optimistic-updates*/}

在执行数据变更时，另一种常见的 UI 模式是在 async 请求进行中乐观地显示最终状态。在 React 19 中，我们新增了一个名为 `useOptimistic` 的钩子来简化这件事：

```js {2,6,13,19}
function ChangeName({currentName, onUpdateName}) {
  const [optimisticName, setOptimisticName] = useOptimistic(currentName);

  const submitAction = async formData => {
    const newName = formData.get("name");
    setOptimisticName(newName);
    const updatedName = await updateName(newName);
    onUpdateName(updatedName);
  };

  return (
    <form action={submitAction}>
      <p>你的名字是：{optimisticName}</p>
      <p>
        <label>更改姓名：</label>
        <input
          type="text"
          name="name"
          disabled={currentName !== optimisticName}
        />
      </p>
    </form>
  );
}
```

当 `updateName` 请求正在进行时，`useOptimistic` 钩子会立即渲染 `optimisticName`。当更新完成或出错时，React 会自动切换回 `currentName` 值。

有关更多信息，请参见 [`useOptimistic`](/reference/react/useOptimistic) 文档。

### 新的 API：`use` {/*new-feature-use*/}

在 React 19 中，我们引入了一个新的 API，用于在渲染中读取资源：`use`。

例如，你可以使用 `use` 读取一个 promise，而 React 会一直挂起直到该 promise 解析：

```js {1,5}
import {use} from 'react';

function Comments({commentsPromise}) {
  // `use` 会一直挂起直到 promise 解析。
  const comments = use(commentsPromise);
  return comments.map(comment => <p key={comment.id}>{comment}</p>);
}

function Page({commentsPromise}) {
  // 当 `use` 在 Comments 中挂起时，
  // 这个 Suspense 边界会被显示。
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <Comments commentsPromise={commentsPromise} />
    </Suspense>
  )
}
```

<Note>

#### `use` 不支持在渲染中创建的 promise。 {/*use-does-not-support-promises-created-in-render*/}

如果你尝试将一个在渲染中创建的 promise 传给 `use`，React 会警告：

<ConsoleBlockMulti>

<ConsoleLogLine level="error">

组件被一个未缓存的 promise 挂起。尚不支持在 Client Component 或 hook 内部创建 promise，除非通过兼容 Suspense 的库或框架。

</ConsoleLogLine>

</ConsoleBlockMulti>

要修复这个问题，你需要从支持 promise 缓存的、由 Suspense 驱动的库或框架中传入 promise。未来我们计划推出一些特性，让在渲染中缓存 promise 变得更容易。

</Note>

你也可以使用 `use` 读取 context，从而允许你有条件地读取 Context，例如在提前返回之后：

```js {1,11}
import {use} from 'react';
import ThemeContext from './ThemeContext'

function Heading({children}) {
  if (children == null) {
    return null;
  }

  // 这在 useContext 下无法工作
  // 因为这里有提前返回。
  const theme = use(ThemeContext);
  return (
    <h1 style={{color: theme.color}}>
      {children}
    </h1>
  );
}
```

`use` API 只能在渲染中调用，类似于钩子。不同于钩子，`use` 可以有条件地调用。未来我们计划支持更多使用 `use` 在渲染中消费资源的方式。

有关更多信息，请参见 [`use`](/reference/react/use) 文档。

## 新的 React DOM 静态 API {/*new-react-dom-static-apis*/}

我们为静态站点生成在 `react-dom/static` 中新增了两个 API：
- [`prerender`](/reference/react-dom/static/prerender)
- [`prerenderToNodeStream`](/reference/react-dom/static/prerenderToNodeStream)

这些新 API 通过在生成静态 HTML 之前等待数据加载，对 `renderToString` 进行了改进。它们旨在与 Node.js Streams 和 Web Streams 之类的流式环境配合使用。例如，在 Web Stream 环境中，你可以使用 `prerender` 将 React 树预渲染为静态 HTML：

```js
import { prerender } from 'react-dom/static';

async function handler(request) {
  const {prelude} = await prerender(<App />, {
    bootstrapScripts: ['/main.js']
  });
  return new Response(prelude, {
    headers: { 'content-type': 'text/html' },
  });
}
```

Prerender API 会在返回静态 HTML 流之前等待所有数据加载完成。流可以被转换为字符串，或者通过流式响应发送。它们不支持在内容加载时进行流式传输，而现有的 [React DOM 服务器渲染 API](/reference/react-dom/server) 支持这一点。

更多信息请参阅 [React DOM 静态 API](/reference/react-dom/static)。

## React Server Components {/*react-server-components*/}

### Server Components {/*server-components*/}

Server Components 是一种新选项，允许在打包之前、在与你的客户端应用或 SSR 服务器分离的环境中提前渲染组件。这个独立环境就是 React Server Components 中的“服务器”。Server Components 可以在 CI 服务器上的构建时运行一次，也可以通过 Web 服务器按每个请求运行。

React 19 包含了从 Canary 渠道引入的全部 React Server Components 功能。这意味着，带有 Server Components 的库现在可以将 React 19 作为 peer dependency，并使用 `react-server` [导出条件](https://github.com/reactjs/rfcs/blob/main/text/0227-server-module-conventions.md#react-server-conditional-exports)，以便在支持 [全栈 React 架构](/learn/creating-a-react-app#which-features-make-up-the-react-teams-full-stack-architecture-vision) 的框架中使用。


<Note>

#### 我该如何为 Server Components 构建支持？ {/*how-do-i-build-support-for-server-components*/}

尽管 React 19 中的 React Server Components 是稳定的，并且不会在小版本之间破坏，但用于实现 React Server Components 打包器或框架的底层 API 不遵循 semver，可能会在 React 19.x 的小版本之间发生破坏性变化。

为了让打包器或框架支持 React Server Components，我们建议锁定到特定的 React 版本，或者使用 Canary 版本。我们将继续与打包器和框架合作，未来稳定用于实现 React Server Components 的 API。

</Note>


更多内容请参阅 [React Server Components](/reference/rsc/server-components) 文档。

### Server Actions {/*server-actions*/}

Server Actions 允许 Client Components 调用在服务器上执行的异步函数。

当使用 `"use server"` 指令定义 Server Action 时，你的框架会自动创建对该服务器函数的引用，并将该引用传递给 Client Component。当客户端调用该函数时，React 会向服务器发送请求来执行该函数，并返回结果。

<Note>

#### Server Components 没有对应的指令。 {/*there-is-no-directive-for-server-components*/}

一个常见的误解是，Server Components 由 `"use server"` 标记，但实际上并没有用于 Server Components 的指令。`"use server"` 指令是用于 Server Actions 的。

更多信息请参阅 [Directives](/reference/rsc/directives) 文档。

</Note>

Server Actions 可以在 Server Components 中创建并作为 props 传递给 Client Components，也可以被导入并在 Client Components 中使用。

更多内容请参阅 [React Server Actions](/reference/rsc/server-actions) 文档。

## React 19 的改进 {/*improvements-in-react-19*/}

### 将 `ref` 作为 prop {/*ref-as-a-prop*/}

从 React 19 开始，你现在可以在函数组件中将 `ref` 作为 prop 访问：

```js [[1, 1, "ref"], [1, 2, "ref", 45], [1, 6, "ref", 14]]
function MyInput({placeholder, ref}) {
  return <input placeholder={placeholder} ref={ref} />
}

//...
<MyInput ref={ref} />
```

新的函数组件将不再需要 `forwardRef`，我们还会发布一个 codemod 来自动更新你的组件以使用新的 `ref` prop。未来版本中，我们将弃用并移除 `forwardRef`。

<Note>

传递给类组件的 `ref` 不会作为 props 传入，因为它们引用的是组件实例。

</Note>

### hydration 错误的差异对比 {/*diffs-for-hydration-errors*/}

我们还改进了 `react-dom` 中 hydration 错误的错误报告。例如，现在不会像以前那样在 DEV 中记录多个错误而没有任何不匹配信息：

<ConsoleBlockMulti>

<ConsoleLogLine level="error">

Warning: Text content did not match. Server: "Server" Client: "Client"
{'  '}at span
{'  '}at App

</ConsoleLogLine>

<ConsoleLogLine level="error">

Warning: An error occurred during hydration. The server HTML was replaced with client content in \<div\>.

</ConsoleLogLine>

<ConsoleLogLine level="error">

Warning: Text content did not match. Server: "Server" Client: "Client"
{'  '}at span
{'  '}at App

</ConsoleLogLine>

<ConsoleLogLine level="error">

Warning: An error occurred during hydration. The server HTML was replaced with client content in \<div\>.

</ConsoleLogLine>

<ConsoleLogLine level="error">

Uncaught Error: Text content does not match server-rendered HTML.
{'  '}at checkForUnmatchedText
{'  '}...

</ConsoleLogLine>

</ConsoleBlockMulti>

现在我们会记录一条带有不匹配差异的消息：


<ConsoleBlockMulti>

<ConsoleLogLine level="error">

Uncaught Error: Hydration failed because the server rendered HTML didn't match the client. As a result this tree will be regenerated on the client. This can happen if an SSR-ed Client Component used:{'\n'}
\- A server/client branch `if (typeof window !== 'undefined')`.
\- Variable input such as `Date.now()` or `Math.random()` which changes each time it's called.
\- Date formatting in a user's locale which doesn't match the server.
\- External changing data without sending a snapshot of it along with the HTML.
\- Invalid HTML tag nesting.{'\n'}
It can also happen if the client has a browser extension installed which messes with the HTML before React loaded.{'\n'}
https://react.dev/link/hydration-mismatch {'\n'}
{'  '}\<App\>
{'    '}\<span\>
{'+    '}Client
{'-    '}Server{'\n'}
{'  '}at throwOnHydrationMismatch
{'  '}...

</ConsoleLogLine>

</ConsoleBlockMulti>

### 将 `<Context>` 作为 provider {/*context-as-a-provider*/}

在 React 19 中，你可以渲染 `<Context>` 作为 provider，而不是 `<Context.Provider>`：


```js {5,7}
const ThemeContext = createContext('');

function App({children}) {
  return (
    <ThemeContext value="dark">
      {children}
    </ThemeContext>
  );
}
```

新的 Context provider 可以使用 `<Context>`，我们还会发布一个 codemod 来转换现有的 provider。未来版本中，我们将弃用 `<Context.Provider>`。

### ref 的清理函数 {/*cleanup-functions-for-refs*/}

现在我们支持从 `ref` 回调中返回清理函数：

```js {7-9}
<input
  ref={(ref) => {
    // 已创建 ref

    // 新增：返回一个清理函数，用于在
    // 元素从 DOM 中移除时重置 ref。
    return () => {
      // ref 清理
    };
  }}
/>
```

当组件卸载时，React 会调用 `ref` 回调返回的清理函数。这适用于 DOM refs、指向类组件的 refs，以及 `useImperativeHandle`。

<Note>

此前，React 在卸载组件时会用 `null` 调用 `ref` 函数。如果你的 `ref` 返回了一个清理函数，React 现在会跳过这一步。

在未来版本中，我们将弃用在卸载组件时使用 `null` 调用 refs 的方式。

</Note>

由于引入了 ref 清理函数，从 `ref` 回调中返回其他任何内容现在都会被 TypeScript 拒绝。通常的修复方式是停止使用隐式返回，例如：

```diff [[1, 1, "("], [1, 1, ")"], [2, 2, "{", 15], [2, 2, "}", 1]]
- <div ref={current => (instance = current)} />
+ <div ref={current => {instance = current}} />
```

原始代码返回了 `HTMLDivElement` 的实例，而 TypeScript 无法判断这是否 _本应_ 是一个清理函数，或者你只是并不想返回清理函数。

你可以使用 [`no-implicit-ref-callback-return`](https://github.com/eps1lon/types-react-codemod/#no-implicit-ref-callback-return) 这个 codemod 来处理这种模式。

### `useDeferredValue` 的初始值 {/*use-deferred-value-initial-value*/}

我们为 `useDeferredValue` 添加了一个 `initialValue` 选项：

```js [[1, 1, "deferredValue"], [1, 4, "deferredValue"], [2, 4, "''"]]
function Search({deferredValue}) {
  // 初始渲染时，该值为 ''。
  // 然后会使用 deferredValue 调度一次重新渲染。
  const value = useDeferredValue(deferredValue, '');

  return (
    <Results query={value} />
  );
}
````

当提供 <CodeStep step={2}>initialValue</CodeStep> 时，`useDeferredValue` 会在组件的初始渲染中将其作为 `value` 返回，并在后台使用返回的 <CodeStep step={1}>deferredValue</CodeStep> 调度一次重新渲染。

更多内容请参阅 [`useDeferredValue`](/reference/react/useDeferredValue)。

### 支持文档元数据 {/*support-for-metadata-tags*/}

在 HTML 中，像 `<title>`、`<link>` 和 `<meta>` 这样的文档元数据标签是保留用于放置在文档的 `<head>` 部分中的。在 React 中，决定应用该使用哪些元数据的组件可能离你渲染 `<head>` 的位置很远，或者 React 根本不渲染 `<head>`。过去，这些元素需要在 effect 中手动插入，或者借助 [`react-helmet`](https://github.com/nfl/react-helmet) 之类的库，并且在服务器渲染 React 应用时需要仔细处理。

在 React 19 中，我们增加了对在组件中原生渲染文档元数据标签的支持：

```js {5-8}
function BlogPost({post}) {
  return (
    <article>
      <h1>{post.title}</h1>
      <title>{post.title}</title>
      <meta name="author" content="Josh" />
      <link rel="author" href="https://twitter.com/joshcstory/" />
      <meta name="keywords" content={post.keywords} />
      <p>
        Eee equals em-see-squared...
      </p>
    </article>
  );
}
```

当 React 渲染这个组件时，它会识别 `<title>`、`<link>` 和 `<meta>` 标签，并自动将它们提升到文档的 `<head>` 部分。通过原生支持这些元数据标签，我们能够确保它们在仅客户端应用、流式 SSR 和 Server Components 中都能正常工作。

<Note>

#### 你可能仍然会想使用元数据库 {/*you-may-still-want-a-metadata-library*/}

对于简单用例，以标签形式渲染文档元数据可能已经足够，但库可以提供更强大的功能，比如基于当前路由用特定元数据覆盖通用元数据。这些功能使得像 [`react-helmet`](https://github.com/nfl/react-helmet) 这样的框架和库更容易支持元数据标签，而不是替代它们。

</Note>

更多信息请参阅 [`<title>`](/reference/react-dom/components/title)、[`<link>`](/reference/react-dom/components/link) 和 [`<meta>`](/reference/react-dom/components/meta) 的文档。

### 支持样式表 {/*support-for-stylesheets*/}

样式表，无论是外部链接的（`<link rel="stylesheet" href="...">`）还是内联的（`<style>...</style>`），由于样式优先级规则，都需要在 DOM 中仔细放置。构建一种允许组件内可组合使用的样式表能力是很难的，因此用户通常要么把所有样式加载得离可能依赖它们的组件很远，要么使用封装了这种复杂性的样式库。

在 React 19 中，我们正在解决这种复杂性，并通过内置的样式表支持，为客户端的并发渲染和服务器的流式渲染提供更深层次的集成。如果你告诉 React 你的样式表的 `precedence`，它就会管理样式表在 DOM 中的插入顺序，并确保样式表（如果是外部样式表）在展示依赖这些样式规则的内容之前已加载完成。

```js {4,5,17}
function ComponentOne() {
  return (
    <Suspense fallback="loading...">
      <link rel="stylesheet" href="foo" precedence="default" />
      <link rel="stylesheet" href="bar" precedence="high" />
      <article class="foo-class bar-class">
        {...}
      </article>
    </Suspense>
  )
}

function ComponentTwo() {
  return (
    <div>
      <p>{...}</p>
      <link rel="stylesheet" href="baz" precedence="default" />  <-- 将插入在 foo 和 bar 之间
    </div>
  )
}
```

在服务端渲染期间，React 会把样式表包含在 `<head>` 中，这可以确保浏览器在加载完成之前不会进行绘制。如果样式表在我们已经开始流式传输之后才被发现，React 会确保在显示依赖该样式表的 Suspense 边界内容之前，先将样式表插入到客户端的 `<head>` 中。

在客户端渲染期间，React 会等待新渲染的样式表加载完成后再提交渲染。如果你在应用的多个位置渲染这个组件，React 只会在文档中包含一次该样式表：

```js {5}
function App() {
  return <>
    <ComponentOne />
    ...
    <ComponentOne /> // 不会导致 DOM 中出现重复的样式表链接
  </>
}
```

对于习惯手动加载样式表的用户来说，这提供了一个机会，可以把这些样式表放在依赖它们的组件旁边，从而更便于本地推理，并且更容易确保只加载你实际依赖的样式表。

样式库以及与打包器集成的样式方案也可以采用这一新能力，因此即使你没有直接渲染自己的样式表，随着工具升级以使用这一功能，你仍然可以受益。

更多细节请阅读 [`<link>`](/reference/react-dom/components/link) 和 [`<style>`](/reference/react-dom/components/style) 的文档。

### 支持异步脚本 {/*support-for-async-scripts*/}

在 HTML 中，普通脚本（`<script src="...">`）和延迟脚本（`<script defer="" src="...">`）会按文档顺序加载，这使得在组件树深处渲染这类脚本具有挑战性。然而，异步脚本（`<script async="" src="...">`）则会以任意顺序加载。

在 React 19 中，我们通过允许你把它们渲染在组件树中的任意位置、放在实际依赖该脚本的组件内部，而无需管理脚本实例的迁移和去重，来提供更好的异步脚本支持。

```js {4,15}
function MyComponent() {
  return (
    <div>
      <script async={true} src="..." />
      Hello World
    </div>
  )
}

function App() {
  <html>
    <body>
      <MyComponent>
      ...
      <MyComponent> // 不会导致 DOM 中出现重复脚本
    </body>
  </html>
}
```

在所有渲染环境中，异步脚本都会被去重，因此即使由多个不同组件渲染，React 也只会加载并执行该脚本一次。

在服务端渲染中，异步脚本会包含在 `<head>` 中，并优先级低于阻止绘制的更关键资源，如样式表、字体和图片预加载。

更多细节请阅读 [`<script>`](/reference/react-dom/components/script) 的文档。

### 支持资源预加载 {/*support-for-preloading-resources*/}

在初始文档加载期间以及客户端更新时，尽早让浏览器了解它可能需要加载的资源，可以对页面性能产生显著影响。

React 19 包含多个用于加载和预加载浏览器资源的新 API，尽可能轻松地构建不会被低效资源加载拖累的出色体验。

```js
import { prefetchDNS, preconnect, preload, preinit } from 'react-dom'
function MyComponent() {
  preinit('https://.../path/to/some/script.js', {as: 'script' }) // 及早加载并执行此脚本
  preload('https://.../path/to/font.woff', { as: 'font' }) // 预加载此字体
  preload('https://.../path/to/stylesheet.css', { as: 'style' }) // 预加载此样式表
  prefetchDNS('https://...') // 当你可能实际上不会向此主机请求任何内容时
  preconnect('https://...') // 当你会请求某些内容但不确定具体是什么时
}
```
```html
<!-- 上面的内容会生成如下 DOM/HTML -->
<html>
  <head>
    <!-- 链接/脚本按其对早期加载的效用优先排序，而不是按调用顺序 -->
    <link rel="prefetch-dns" href="https://...">
    <link rel="preconnect" href="https://...">
    <link rel="preload" as="font" href="https://.../path/to/font.woff">
    <link rel="preload" as="style" href="https://.../path/to/stylesheet.css">
    <script async="" src="https://.../path/to/some/script.js"></script>
  </head>
  <body>
    ...
  </body>
</html>
```

这些 API 可用于通过将字体等额外资源的发现从样式表加载中移出，来优化初始页面加载。它们还可以通过预取预期导航会用到的一组资源，然后在点击时甚至在悬停时及早预加载这些资源，来加快客户端更新。

更多细节请参阅 [资源预加载 API](/reference/react-dom#resource-preloading-apis)。

### 与第三方脚本和扩展的兼容性 {/*compatibility-with-third-party-scripts-and-extensions*/}

我们改进了 hydration，以考虑第三方脚本和浏览器扩展的影响。

在进行 hydration 时，如果客户端渲染的某个元素与服务器 HTML 中找到的元素不匹配，React 会强制进行客户端重新渲染来修正内容。以前，如果某个元素是由第三方脚本或浏览器扩展插入的，就会触发不匹配错误并进行客户端渲染。

在 React 19 中，`<head>` 和 `<body>` 中意外出现的标签会被跳过，从而避免不匹配错误。如果由于与 hydration 无关的不匹配而需要重新渲染整个文档，React 会保留由第三方脚本和浏览器扩展插入的样式表。

### 更好的错误报告 {/*error-handling*/}

我们改进了 React 19 中的错误处理，以减少重复并为处理已捕获和未捕获错误提供选项。例如，当渲染中发生错误并被 Error Boundary 捕获时，之前 React 会抛出两次错误（一次是原始错误，另一次是在自动恢复失败之后），然后再调用 `console.error` 并附带错误发生位置的信息。

这会导致每个被捕获的错误出现三条错误信息：

<ConsoleBlockMulti>

<ConsoleLogLine level="error">

Uncaught Error: hit
{'  '}at Throws
{'  '}at renderWithHooks
{'  '}...

</ConsoleLogLine>

<ConsoleLogLine level="error">

Uncaught Error: hit<span className="ms-2 text-gray-30">{'    <--'} 重复</span>
{'  '}at Throws
{'  '}at renderWithHooks
{'  '}...

</ConsoleLogLine>

<ConsoleLogLine level="error">

The above error occurred in the Throws component:
{'  '}at Throws
{'  '}at ErrorBoundary
{'  '}at App{'\n'}
React will try to recreate this component tree from scratch using the error boundary you provided, ErrorBoundary.

</ConsoleLogLine>

</ConsoleBlockMulti>

在 React 19 中，我们只会记录一条包含全部错误信息的错误：

<ConsoleBlockMulti>

<ConsoleLogLine level="error">

Error: hit
{'  '}at Throws
{'  '}at renderWithHooks
{'  '}...{'\n'}
The above error occurred in the Throws component:
{'  '}at Throws
{'  '}at ErrorBoundary
{'  '}at App{'\n'}
React will try to recreate this component tree from scratch using the error boundary you provided, ErrorBoundary.
{'  '}at ErrorBoundary
{'  '}at App

</ConsoleLogLine>

</ConsoleBlockMulti>

此外，我们新增了两个 root 选项，以补充 `onRecoverableError`：

- `onCaughtError`：当 React 在 Error Boundary 中捕获到错误时调用。
- `onUncaughtError`：当错误被抛出且未被 Error Boundary 捕获时调用。
- `onRecoverableError`：当错误被抛出并自动恢复时调用。

更多信息和示例请参阅 [`createRoot`](/reference/react-dom/client/createRoot) 和 [`hydrateRoot`](/reference/react-dom/client/hydrateRoot) 的文档。

### 支持自定义元素 {/*support-for-custom-elements*/}

React 19 为自定义元素添加了完整支持，并且通过了 [Custom Elements Everywhere](https://custom-elements-everywhere.com/) 的所有测试。

在以前的版本中，在 React 中使用自定义元素很困难，因为 React 会将无法识别的 props 视为属性而不是 property。在 React 19 中，我们通过以下策略增加了对 property 的支持，可在客户端和 SSR 中生效：

- **服务端渲染**：传递给自定义元素的 props，如果类型是诸如 `string`、`number` 这样的原始值，或者值为 `true`，则会渲染为属性。类型为 `object`、`symbol`、`function` 等非原始类型，或者值为 `false` 的 props 将被省略。
- **客户端渲染**：如果 props 与 Custom Element 实例上的某个 property 匹配，则会将其作为 property 赋值，否则将作为属性赋值。

感谢 [Joey Arhar](https://github.com/josepharhar) 推动了 React 中自定义元素支持的设计和实现。


#### 如何升级 {/*how-to-upgrade*/}
请参阅 [React 19 升级指南](/blog/2024/04/25/react-19-upgrade-guide)，了解逐步说明和完整的破坏性变更与重要变更列表。

_注：本文最初发布于 2024/04/25，并已更新至 2024/12/05 的稳定版发布。_
