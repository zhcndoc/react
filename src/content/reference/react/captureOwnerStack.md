---
title: captureOwnerStack
---

<Intro>

`captureOwnerStack` 在开发环境中读取当前的 Owner Stack，并在可用时将其作为字符串返回。

```js
const stack = captureOwnerStack();
```

</Intro>

<InlineToc />

---

## 参考 {/*reference*/}

### `captureOwnerStack()` {/*captureownerstack*/}

调用 `captureOwnerStack` 以获取当前的 Owner Stack。

```js {5,5}
import * as React from 'react';

function Component() {
  if (process.env.NODE_ENV !== 'production') {
    const ownerStack = React.captureOwnerStack();
    console.log(ownerStack);
  }
}
```

#### 参数 {/*parameters*/}

`captureOwnerStack` 不接受任何参数。

#### 返回值 {/*returns*/}

`captureOwnerStack` 返回 `string | null`。

Owner Stacks 可在以下场景中使用：
- 组件渲染
- Effects（例如 `useEffect`）
- React 的事件处理器（例如 `<button onClick={...} />`）
- React 错误处理器（[React Root options](/reference/react-dom/client/createRoot#parameters) 中的 `onCaughtError`、`onRecoverableError` 和 `onUncaughtError`）

如果没有可用的 Owner Stack，则返回 `null`（参见 [故障排查：Owner Stack 为 `null`](#the-owner-stack-is-null)）。

#### 注意事项 {/*caveats*/}

- Owner Stacks 仅在开发环境中可用。`captureOwnerStack` 在开发环境之外始终返回 `null`。

<DeepDive>

#### Owner Stack 与 Component Stack 的区别 {/*owner-stack-vs-component-stack*/}

Owner Stack 与 React 错误处理器中可用的 Component Stack 不同，例如 [`onUncaughtError` 中的 `errorInfo.componentStack`](/reference/react-dom/client/hydrateRoot#error-logging-in-production)。

例如，考虑以下代码：

<Sandpack>

```js src/App.js
import {Suspense} from 'react';

function SubComponent({disabled}) {
  if (disabled) {
    throw new Error('disabled');
  }
}

export function Component({label}) {
  return (
    <fieldset>
      <legend>{label}</legend>
      <SubComponent key={label} disabled={label === 'disabled'} />
    </fieldset>
  );
}

function Navigation() {
  return null;
}

export default function App({children}) {
  return (
    <Suspense fallback="loading...">
      <main>
        <Navigation />
        {children}
      </main>
    </Suspense>
  );
}
```

```js src/index.js
import {captureOwnerStack} from 'react';
import {createRoot} from 'react-dom/client';
import App, {Component} from './App.js';
import './styles.css';

createRoot(document.createElement('div'), {
  onUncaughtError: (error, errorInfo) => {
    // 这里将堆栈信息记录到日志中，而不是直接在 UI 中显示，
    // 以强调浏览器会对已记录的堆栈应用 sourcemap。
    // 请注意，sourcemap 只会在真实的浏览器控制台中应用，
    // 而不会在本页显示的虚假控制台中应用。
    // 点击“fork”后，你就可以在真实控制台中查看经过 sourcemap 映射的堆栈。
    console.log(errorInfo.componentStack);
    console.log(captureOwnerStack());
  },
}).render(
  <App>
    <Component label="disabled" />
  </App>
);
```

```html public/index.html hidden
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>文档</title>
  </head>
  <body>
    <p>检查控制台输出。</p>
  </body>
</html>
```

</Sandpack>

`SubComponent` 会抛出错误。  
该错误的 Component Stack 将为：

```
at SubComponent
at fieldset
at Component
at main
at React.Suspense
at App
```

然而，Owner Stack 只会显示：

```
at Component
```

在这个堆栈中，`App` 和 DOM 组件（例如 `fieldset`）都不被视为 Owner，因为它们并未参与“创建”包含 `SubComponent` 的节点。`App` 和 DOM 组件只是传递了该节点。`App` 只是渲染了 `children` 节点，而 `Component` 则通过 `<SubComponent />` 创建了一个包含 `SubComponent` 的节点。

`Navigation` 和 `legend` 根本不会出现在堆栈中，因为它们只是包含 `<SubComponent />` 的节点的兄弟节点。

`SubComponent` 被省略了，因为它已经是调用栈的一部分。

</DeepDive>

## 用法 {/*usage*/}

### 增强自定义错误覆盖层 {/*enhance-a-custom-error-overlay*/}

```js [[1, 5, "console.error"], [4, 7, "captureOwnerStack"]]
import { captureOwnerStack } from "react";
import { instrumentedConsoleError } from "./errorOverlay";

const originalConsoleError = console.error;
console.error = function patchedConsoleError(...args) {
  originalConsoleError.apply(console, args);
  const ownerStack = captureOwnerStack();
  onConsoleError({
    // 请注意，在真实应用中，console.error 可能会传入多个参数，
    // 你需要对此进行处理。
    consoleMessage: args[0],
    ownerStack,
  });
};
```

如果你拦截 <CodeStep step={1}>`console.error`</CodeStep> 调用并在错误覆盖层中高亮它们，那么你可以调用 <CodeStep step={2}>`captureOwnerStack`</CodeStep> 来包含 Owner Stack。

<Sandpack>

```css src/styles.css
* {
  box-sizing: border-box;
}

body {
  font-family: sans-serif;
  margin: 20px;
  padding: 0;
}

h1 {
  margin-top: 0;
  font-size: 22px;
}

h2 {
  margin-top: 0;
  font-size: 20px;
}

code {
  font-size: 1.2em;
}

ul {
  padding-inline-start: 20px;
}

label, button { display: block; margin-bottom: 20px; }
html, body { min-height: 300px; }

#error-dialog {
  position: absolute;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
  background-color: white;
  padding: 15px;
  opacity: 0.9;
  text-wrap: wrap;
  overflow: scroll;
}

.text-red {
  color: red;
}

.-mb-20 {
  margin-bottom: -20px;
}

.mb-0 {
  margin-bottom: 0;
}

.mb-10 {
  margin-bottom: 10px;
}

pre {
  text-wrap: wrap;
}

pre.nowrap {
  text-wrap: nowrap;
}

.hidden {
 display: none;
}
```

```html public/index.html hidden
<!DOCTYPE html>
<html>
<head>
  <title>我的应用</title>
</head>
<body>
<!--
  原始 HTML 中的错误对话框
  因为 React 应用中的错误可能会导致崩溃。
-->
<div id="error-dialog" class="hidden">
  <h1 id="error-title" class="text-red">错误</h1>
  <p>
    <pre id="error-body"></pre>
  </p>
  <h2 class="-mb-20">Owner Stack：</h4>
  <pre id="error-owner-stack" class="nowrap"></pre>
  <button
    id="error-close"
    class="mb-10"
    onclick="document.getElementById('error-dialog').classList.add('hidden')"
  >
    关闭
  </button>
</div>
<!-- 这是 DOM 节点 -->
<div id="root"></div>
</body>
</html>

```

```js src/errorOverlay.js

export function onConsoleError({ consoleMessage, ownerStack }) {
  const errorDialog = document.getElementById("error-dialog");
  const errorBody = document.getElementById("error-body");
  const errorOwnerStack = document.getElementById("error-owner-stack");

  // 显示 console.error() 消息
  errorBody.innerText = consoleMessage;

  // 显示 owner stack
  errorOwnerStack.innerText = ownerStack;

  // 显示对话框
  errorDialog.classList.remove("hidden");
}
```

```js src/index.js active
import { captureOwnerStack } from "react";
import { createRoot } from "react-dom/client";
import App from './App';
import { onConsoleError } from "./errorOverlay";
import './styles.css';

const originalConsoleError = console.error;
console.error = function patchedConsoleError(...args) {
  originalConsoleError.apply(console, args);
  const ownerStack = captureOwnerStack();
  onConsoleError({
    // 请注意，在真实应用中，console.error 可能会传入多个参数，
    // 你需要对此进行处理。
    consoleMessage: args[0],
    ownerStack,
  });
};

const container = document.getElementById("root");
createRoot(container).render(<App />);
```

```js src/App.js
function Component() {
  return <button onClick={() => console.error('某个 console error')}>触发 console.error()</button>;
}

export default function App() {
  return <Component />;
}
```

</Sandpack>

## 故障排查 {/*troubleshooting*/}

### Owner Stack 为 `null` {/*the-owner-stack-is-null*/}

调用 `captureOwnerStack` 发生在 React 控制的函数之外，例如在 `setTimeout` 回调中、`fetch` 调用之后，或者在自定义 DOM 事件处理器中。在渲染、Effects、React 事件处理器以及 React 错误处理器（例如 `hydrateRoot#options.onCaughtError`）期间，Owner Stack 应该是可用的。

在下面的示例中，点击按钮会记录一个空的 Owner Stack，因为 `captureOwnerStack` 是在自定义 DOM 事件处理器中调用的。必须更早地捕获 Owner Stack，例如将 `captureOwnerStack` 的调用移动到 Effect 主体中。
<Sandpack>

```js
import {captureOwnerStack, useEffect} from 'react';

export default function App() {
  useEffect(() => {
    // 应该在这里调用 `captureOwnerStack`。
    function handleEvent() {
      // 在自定义 DOM 事件处理器中调用它已经太晚了。
      // 此时 Owner Stack 将为 `null`。
      console.log('Owner Stack: ', captureOwnerStack());
    }

    document.addEventListener('click', handleEvent);

    return () => {
      document.removeEventListener('click', handleEvent);
    }
  })

  return <button>点击我以查看在自定义 DOM 事件处理器中不可用的 Owner Stack</button>;
}
```

</Sandpack>

### `captureOwnerStack` 不可用 {/*captureownerstack-is-not-available*/}

`captureOwnerStack` 只会在开发构建中导出。在生产构建中它将是 `undefined`。如果 `captureOwnerStack` 被用于同时会打包到生产和开发环境的文件中，你应该通过命名空间导入有条件地访问它。

```js
// 不要在同时会被打包用于开发和生产的文件中使用 `captureOwnerStack` 的命名导入。
import {captureOwnerStack} from 'react';
// 而是改用命名空间导入，并有条件地访问 `captureOwnerStack`。
import * as React from 'react';

if (process.env.NODE_ENV !== 'production') {
  const ownerStack = React.captureOwnerStack();
  console.log('Owner Stack', ownerStack);
}
```
