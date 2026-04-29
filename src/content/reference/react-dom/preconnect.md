---
title: preconnect
---

<Intro>

`preconnect` 允许你主动连接到你预期会从中加载资源的服务器。

```js
preconnect("https://example.com");
```

</Intro>

<InlineToc />

---

## 参考 {/*reference*/}

### `preconnect(href)` {/*preconnect*/}

要预连接到某个主机，请从 `react-dom` 调用 `preconnect` 函数。

```js
import { preconnect } from 'react-dom';

function AppRoot() {
  preconnect("https://example.com");
  // ...
}

```

[在下方查看更多示例。](#usage)

`preconnect` 函数会向浏览器提供一个提示，告诉它应当与给定的服务器建立连接。如果浏览器选择这样做，这可以加快从该服务器加载资源的速度。

#### 参数 {/*parameters*/}

* `href`：一个字符串。你想要连接的服务器 URL。


#### 返回值 {/*returns*/}

`preconnect` 不返回任何内容。

#### 注意事项 {/*caveats*/}

* 对同一服务器多次调用 `preconnect` 的效果与调用一次相同。
* 在浏览器中，你可以在任何场景下调用 `preconnect`：在渲染组件时、在 Effect 中、在事件处理函数中，等等。
* 在服务端渲染或渲染 Server Components 时，只有当你在渲染组件时或在源自组件渲染的异步上下文中调用 `preconnect`，它才会生效。其他任何调用都会被忽略。
* 如果你知道确切需要哪些资源，可以改为调用[其他函数](/reference/react-dom/#resource-preloading-apis)，它们会立即开始加载资源。
* 预连接到网页本身所托管的同一服务器没有任何好处，因为在提示发出时，连接其实已经建立好了。

---

## 用法 {/*usage*/}

### 在渲染时预连接 {/*preconnecting-when-rendering*/}

如果你知道其子组件将从该主机加载外部资源，那么在渲染组件时调用 `preconnect`。

```js
import { preconnect } from 'react-dom';

function AppRoot() {
  preconnect("https://example.com");
  return ...;
}
```

### 在事件处理函数中预连接 {/*preconnecting-in-an-event-handler*/}

在切换到需要外部资源的页面或状态之前，在事件处理函数中调用 `preconnect`。这样比在新页面或新状态渲染期间调用它更早开始该过程。

```js
import { preconnect } from 'react-dom';

function CallToAction() {
  const onClick = () => {
    preconnect('http://example.com');
    startWizard();
  }
  return (
    <button onClick={onClick}>开始向导</button>
  );
}
```
