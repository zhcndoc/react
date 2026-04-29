---
title: "'use server'"
titleForTitleTag: "'use server' 指令"
---

<RSC>

`'use server'` 用于与 [使用 React Server Components](/reference/rsc/server-components) 一起使用。

</RSC>


<Intro>

`'use server'` 用于标记可从客户端代码调用的服务端函数。

</Intro>

<InlineToc />

---

## 参考 {/*reference*/}

### `'use server'` {/*use-server*/}

在异步函数体的顶部添加 `'use server'`，以将该函数标记为可由客户端调用。我们将这些函数称为 [_Server Functions_](/reference/rsc/server-functions)。

```js {2}
async function addToCart(data) {
  'use server';
  // ...
}
```

在客户端调用 Server Function 时，它会向服务器发起网络请求，其中包含传入的任何参数的序列化副本。如果 Server Function 返回一个值，该值也会被序列化并返回给客户端。

你也可以不单独用 `'use server'` 标记每个函数，而是在文件顶部添加该指令，将该文件中的所有导出都标记为 Server Functions，这些函数可在任何地方使用，包括在客户端代码中导入。

#### 注意事项 {/*caveats*/}
* `'use server'` 必须位于其函数或模块的最开头；在任何其他代码之上，包括导入语句（指令上方的注释是可以的）。它们必须使用单引号或双引号编写，不能使用反引号。
* `'use server'` 只能在服务端文件中使用。生成的 Server Functions 可以通过 props 传递给 Client Components。请参阅用于序列化的受支持 [类型](#serializable-parameters-and-return-values)。
* 要从 [客户端代码](/reference/rsc/use-client) 导入 Server Functions，必须在模块级别使用该指令。
* 由于底层网络调用始终是异步的，`'use server'` 只能用于 async 函数。
* 始终将传递给 Server Functions 的参数视为不可信输入，并对任何修改进行授权。请参阅 [安全考虑](#security)。
* Server Functions 应在 [过渡](/reference/react/useTransition) 中调用。通过 [`<form action>`](/reference/react-dom/components/form#props) 或 [`formAction`](/reference/react-dom/components/input#props) 传递的 Server Functions 会自动在过渡中调用。
* Server Functions 是为更新服务端状态的变更而设计的；不建议用于数据获取。因此，实现 Server Functions 的框架通常一次只处理一个操作，并且没有缓存返回值的方法。

### 安全考虑 {/*security*/}

传递给 Server Functions 的参数完全由客户端控制。出于安全考虑，请始终将其视为不可信输入，并确保根据需要验证和转义参数。

在任何 Server Function 中，都要确保验证当前登录用户是否有权执行该操作。

<Wip>

为防止从 Server Function 发送敏感数据，有一些实验性的 taint API，可防止唯一值和对象被传递给客户端代码。

请参阅 [experimental_taintUniqueValue](/reference/react/experimental_taintUniqueValue) 和 [experimental_taintObjectReference](/reference/react/experimental_taintObjectReference)。

</Wip>

### 可序列化的参数和返回值 {/*serializable-parameters-and-return-values*/}

由于客户端代码通过网络调用 Server Function，传入的任何参数都需要是可序列化的。

以下是 Server Function 参数支持的类型：

* 原始类型
	* [string](https://developer.mozilla.org/en-US/docs/Glossary/String)
	* [number](https://developer.mozilla.org/en-US/docs/Glossary/Number)
	* [bigint](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/BigInt)
	* [boolean](https://developer.mozilla.org/en-US/docs/Glossary/Boolean)
	* [undefined](https://developer.mozilla.org/en-US/docs/Glossary/Undefined)
	* [null](https://developer.mozilla.org/en-US/docs/Glossary/Null)
	* [symbol](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Symbol)，仅限通过 [`Symbol.for`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Symbol/for) 在全局 Symbol 注册表中注册的 symbol
* 包含可序列化值的可迭代对象
	* [String](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String)
	* [Array](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array)
	* [Map](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map)
	* [Set](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set)
	* [TypedArray](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/TypedArray) 和 [ArrayBuffer](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/ArrayBuffer)
* [Date](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date)
* [FormData](https://developer.mozilla.org/en-US/docs/Web/API/FormData) 实例
* 普通 [对象](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object)：使用 [对象初始化器](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Object_initializer) 创建、且其属性可序列化的对象
* Server Functions
* [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)

特别地，以下内容不受支持：
* React 元素，或 [JSX](/learn/writing-markup-with-jsx)
* 函数，包括组件函数或任何不是 Server Function 的其他函数
* [类](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Objects/Classes_in_JavaScript)
* 任何类的实例对象（不包括上述内建类型）或具有 [null 原型](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object#null-prototype_objects) 的对象
* 未全局注册的 symbol，例如 `Symbol('my new symbol')`
* 来自事件处理器的事件


支持的可序列化返回值与边界 Client Component 的 [可序列化 props](/reference/rsc/use-client#serializable-types) 相同。


## 用法 {/*usage*/}

### 表单中的 Server Functions {/*server-functions-in-forms*/}

Server Functions 最常见的使用场景是调用会修改数据的函数。在浏览器中，[HTML form 元素](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/form) 是用户提交变更的传统方式。使用 React Server Components 时，React 为在 [表单](/reference/react-dom/components/form) 中将 Server Functions 作为 Actions 使用引入了一级支持。

下面是一个允许用户请求用户名的表单。

```js [[1, 3, "formData"]]
// App.js

async function requestUsername(formData) {
  'use server';
  const username = formData.get('username');
  // ...
}

export default function App() {
  return (
    <form action={requestUsername}>
      <input type="text" name="username" />
      <button type="submit">请求</button>
    </form>
  );
}
```

在这个示例中，`requestUsername` 是作为 `<form>` 传递的 Server Function。当用户提交此表单时，会向 Server Function `requestUsername` 发起网络请求。在表单中调用 Server Function 时，React 会将表单的 <CodeStep step={1}>[FormData](https://developer.mozilla.org/en-US/docs/Web/API/FormData)</CodeStep> 作为 Server Function 的第一个参数提供。

通过将 Server Function 传递给表单的 `action`，React 可以对表单进行 [渐进增强](https://developer.mozilla.org/en-US/docs/Glossary/Progressive_Enhancement)。这意味着即使 JavaScript bundle 还未加载完成，表单也可以被提交。

#### 处理表单中的返回值 {/*handling-return-values*/}

在用户名请求表单中，可能会出现用户名不可用的情况。`requestUsername` 应该告诉我们请求是否失败。

要在支持渐进增强的同时根据 Server Function 的结果更新 UI，请使用 [`useActionState`](/reference/react/useActionState)。

```js
// requestUsername.js
'use server';

export default async function requestUsername(formData) {
  const username = formData.get('username');
  if (canRequest(username)) {
    // ...
    return 'successful';
  }
  return 'failed';
}
```

```js {4,8}, [[2, 2, "'use client'"]]
// UsernameForm.js
'use client';

import { useActionState } from 'react';
import requestUsername from './requestUsername';

function UsernameForm() {
  const [state, action] = useActionState(requestUsername, null, 'n/a');

  return (
    <>
      <form action={action}>
        <input type="text" name="username" />
        <button type="submit">请求</button>
      </form>
      <p>上次提交请求返回：{state}</p>
    </>
  );
}
```

请注意，与大多数 Hooks 一样，`useActionState` 只能在 <CodeStep step={1}>[客户端代码](/reference/rsc/use-client)</CodeStep> 中调用。

### 在 `<form>` 外调用 Server Function {/*calling-a-server-function-outside-of-form*/}

Server Functions 是暴露的服务端端点，可以在客户端代码中的任何位置调用。

在 [form](/reference/react-dom/components/form) 外使用 Server Function 时，请在 [过渡](/reference/react/useTransition) 中调用该 Server Function，这样你就可以显示加载指示器、展示 [乐观状态更新](/reference/react/useOptimistic)，以及处理意外错误。表单会自动将 Server Functions 包装在过渡中。

```js {9-12}
import incrementLike from './actions';
import { useState, useTransition } from 'react';

function LikeButton() {
  const [isPending, startTransition] = useTransition();
  const [likeCount, setLikeCount] = useState(0);

  const onClick = () => {
    startTransition(async () => {
      const currentCount = await incrementLike();
      setLikeCount(currentCount);
    });
  };

  return (
    <>
      <p>总点赞数：{likeCount}</p>
      <button onClick={onClick} disabled={isPending}>点赞</button>;
    </>
  );
}
```

```js
// actions.js
'use server';

let likeCount = 0;
export default async function incrementLike() {
  likeCount++;
  return likeCount;
}
```

要读取 Server Function 的返回值，你需要 `await` 它返回的 promise。
