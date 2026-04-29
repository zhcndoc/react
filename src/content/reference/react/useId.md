---
title: useId
---

<Intro>

`useId` 是一个 React Hook，用于生成可以传递给无障碍属性的唯一 ID。

```js
const id = useId()
```

</Intro>

<InlineToc />

---

## 参考 {/*reference*/}

### `useId()` {/*useid*/}

在组件顶层调用 `useId` 来生成一个唯一 ID：

```js
import { useId } from 'react';

function PasswordField() {
  const passwordHintId = useId();
  // ...
```

[查看更多示例。](#usage)

#### 参数 {/*parameters*/}

`useId` 不接受任何参数。

#### 返回值 {/*returns*/}

`useId` 返回一个唯一的 ID 字符串，它与这个特定组件中的这次 `useId` 调用相关联。

#### 注意事项 {/*caveats*/}

* `useId` 是一个 Hook，因此你只能在**组件顶层**或你自己的 Hooks 中调用它。你不能在循环或条件中调用它。如果你需要这样做，请抽取一个新组件并将状态移入其中。

* `useId` **不应被用于为 [use()](/reference/react/use) 生成缓存键**。组件挂载时该 ID 是稳定的，但在渲染过程中可能会改变。缓存键应当由你的数据生成。

* `useId` **不应被用于在列表中生成键**。[键应该由你的数据生成。](/learn/rendering-lists#where-to-get-your-key)

* `useId` 目前不能用于 [异步 Server Components](/reference/rsc/server-components#async-components-with-server-components)。

---

## 用法 {/*usage*/}

<Pitfall>

**不要调用 `useId` 来为列表生成键。** [键应该由你的数据生成。](/learn/rendering-lists#where-to-get-your-key)

</Pitfall>

### 为无障碍属性生成唯一 ID {/*generating-unique-ids-for-accessibility-attributes*/}

在组件顶层调用 `useId` 来生成一个唯一 ID：

```js [[1, 4, "passwordHintId"]]
import { useId } from 'react';

function PasswordField() {
  const passwordHintId = useId();
  // ...
```

然后你可以将<CodeStep step={1}>生成的 ID</CodeStep>传递给不同的属性：

```js [[1, 2, "passwordHintId"], [1, 3, "passwordHintId"]]
<>
  <input type="password" aria-describedby={passwordHintId} />
  <p id={passwordHintId}>
</>
```

**让我们通过一个示例来看看这在什么时候有用。**

[HTML 无障碍属性](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA)（例如 [`aria-describedby`](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Attributes/aria-describedby)）可以让你指定两个标签彼此相关。例如，你可以指定一个元素（如输入框）由另一个元素（如段落）进行描述。

在普通 HTML 中，你会这样写：

```html {5,8}
<label>
  密码：
  <input
    type="password"
    aria-describedby="password-hint"
  />
</label>
<p id="password-hint">
  密码应至少包含 18 个字符
</p>
```

然而，在 React 中硬编码这样的 ID 并不是一个好做法。一个组件可能会在页面上渲染不止一次——但 ID 必须是唯一的！不要硬编码 ID，而是使用 `useId` 生成一个唯一 ID：

```js {4,11,14}
import { useId } from 'react';

function PasswordField() {
  const passwordHintId = useId();
  return (
    <>
      <label>
        密码：
        <input
          type="password"
          aria-describedby={passwordHintId}
        />
      </label>
      <p id={passwordHintId}>
        密码应至少包含 18 个字符
      </p>
    </>
  );
}
```

现在，即使 `PasswordField` 在屏幕上出现多次，生成的 ID 也不会冲突。

<Sandpack>

```js
import { useId } from 'react';

function PasswordField() {
  const passwordHintId = useId();
  return (
    <>
      <label>
        密码：
        <input
          type="password"
          aria-describedby={passwordHintId}
        />
      </label>
      <p id={passwordHintId}>
        密码应至少包含 18 个字符
      </p>
    </>
  );
}

export default function App() {
  return (
    <>
      <h2>选择密码</h2>
      <PasswordField />
      <h2>确认密码</h2>
      <PasswordField />
    </>
  );
}
```

```css
input { margin: 5px; }
```

</Sandpack>

[观看这个视频](https://www.youtube.com/watch?v=0dNzNcuEuOo) 来了解辅助技术下用户体验的差异。

<Pitfall>

在[服务端渲染](/reference/react-dom/server)中，**`useId` 要求服务端和客户端具有完全相同的组件树**。如果你在服务端和客户端渲染的树不完全匹配，生成的 ID 也不会匹配。

</Pitfall>

<DeepDive>

#### 为什么 useId 比递增计数器更好？ {/*why-is-useid-better-than-an-incrementing-counter*/}

你可能会想，为什么 `useId` 比像 `nextId++` 这样的全局变量递增更好。

`useId` 的主要优势在于，React 确保它可以与[服务端渲染](/reference/react-dom/server)配合使用。在服务端渲染期间，你的组件会生成 HTML 输出。随后在客户端，[hydration](/reference/react-dom/client/hydrateRoot) 会把事件处理器附加到生成的 HTML 上。要让 hydration 正常工作，客户端输出必须与服务端 HTML 匹配。

对于递增计数器来说，要保证这一点非常困难，因为客户端组件被 hydration 的顺序可能与服务端 HTML 的输出顺序不一致。通过调用 `useId`，你可以确保 hydration 正常工作，并且服务端与客户端之间的输出能够匹配。

在 React 内部，`useId` 是从调用该 Hook 的组件的“父路径”生成的。这就是为什么如果客户端和服务端的树相同，那么无论渲染顺序如何，“父路径”都会对应起来。

</DeepDive>

---

### 为多个相关元素生成 ID {/*generating-ids-for-several-related-elements*/}

如果你需要为多个相关元素提供 ID，可以调用 `useId` 为它们生成一个共享前缀：

<Sandpack>

```js
import { useId } from 'react';

export default function Form() {
  const id = useId();
  return (
    <form>
      <label htmlFor={id + '-firstName'}>名：</label>
      <input id={id + '-firstName'} type="text" />
      <hr />
      <label htmlFor={id + '-lastName'}>姓：</label>
      <input id={id + '-lastName'} type="text" />
    </form>
  );
}
```

```css
input { margin: 5px; }
```

</Sandpack>

这样你就不必为每一个需要唯一 ID 的元素都调用 `useId`。

---

### 为所有生成的 ID 指定共享前缀 {/*specifying-a-shared-prefix-for-all-generated-ids*/}

如果你在同一页面上渲染多个独立的 React 应用，请在调用 [`createRoot`](/reference/react-dom/client/createRoot#parameters) 或 [`hydrateRoot`](/reference/react-dom/client/hydrateRoot) 时，将 `identifierPrefix` 作为选项传入。这可以确保这两个不同应用生成的 ID 永远不会冲突，因为每个通过 `useId` 生成的标识符都会以你指定的不同前缀开头。

<Sandpack>

```html public/index.html
<!DOCTYPE html>
<html>
  <head><title>我的应用</title></head>
  <body>
    <div id="root1"></div>
    <div id="root2"></div>
  </body>
</html>
```

```js
import { useId } from 'react';

function PasswordField() {
  const passwordHintId = useId();
  console.log('生成的标识符:', passwordHintId)
  return (
    <>
      <label>
        密码：
        <input
          type="password"
          aria-describedby={passwordHintId}
        />
      </label>
      <p id={passwordHintId}>
        密码应至少包含 18 个字符
      </p>
    </>
  );
}

export default function App() {
  return (
    <>
      <h2>选择密码</h2>
      <PasswordField />
    </>
  );
}
```

```js src/index.js active
import { createRoot } from 'react-dom/client';
import App from './App.js';
import './styles.css';

const root1 = createRoot(document.getElementById('root1'), {
  identifierPrefix: 'my-first-app-'
});
root1.render(<App />);

const root2 = createRoot(document.getElementById('root2'), {
  identifierPrefix: 'my-second-app-'
});
root2.render(<App />);
```

```css
#root1 {
  border: 5px solid blue;
  padding: 10px;
  margin: 5px;
}

#root2 {
  border: 5px solid green;
  padding: 10px;
  margin: 5px;
}

input { margin: 5px; }
```

</Sandpack>

---

### 在客户端和服务端使用相同的 ID 前缀 {/*using-the-same-id-prefix-on-the-client-and-the-server*/}

如果你[在同一页面上渲染多个独立的 React 应用](#specifying-a-shared-prefix-for-all-generated-ids)，并且其中一些应用是服务端渲染的，请确保你在客户端传给 [`hydrateRoot`](/reference/react-dom/client/hydrateRoot) 调用的 `identifierPrefix` 与你传给诸如 [`renderToPipeableStream`](/reference/react-dom/server/renderToPipeableStream) 这类[服务端 API](/reference/react-dom/server) 的 `identifierPrefix` 相同。

```js
// 服务端
import { renderToPipeableStream } from 'react-dom/server';

const { pipe } = renderToPipeableStream(
  <App />,
  { identifierPrefix: 'react-app1' }
);
```

```js
// 客户端
import { hydrateRoot } from 'react-dom/client';

const domNode = document.getElementById('root');
const root = hydrateRoot(
  domNode,
  reactNode,
  { identifierPrefix: 'react-app1' }
);
```

如果页面上只有一个 React 应用，你不需要传递 `identifierPrefix`。
