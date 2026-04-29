---
title: prefetchDNS
---

<Intro>

`prefetchDNS` 可让你主动查找你预期会从中加载资源的服务器 IP。

```js
prefetchDNS("https://example.com");
```

</Intro>

<InlineToc />

---

## 参考 {/*reference*/}

### `prefetchDNS(href)` {/*prefetchdns*/}

要查找主机，请从 `react-dom` 调用 `prefetchDNS` 函数。

```js
import { prefetchDNS } from 'react-dom';

function AppRoot() {
  prefetchDNS("https://example.com");
  // ...
}

```

[查看更多示例。](#usage)

`prefetchDNS` 函数会向浏览器提供一个提示，提示它应该查找给定服务器的 IP 地址。如果浏览器选择这样做，这可以加快从该服务器加载资源的速度。

#### 参数 {/*parameters*/}

* `href`：字符串。你想要连接的服务器 URL。

#### 返回值 {/*returns*/}

`prefetchDNS` 不返回任何内容。

#### 注意事项 {/*caveats*/}

* 对同一服务器多次调用 `prefetchDNS` 与调用一次的效果相同。
* 在浏览器中，你可以在任何情况下调用 `prefetchDNS`：在渲染组件时、在 Effect 中、在事件处理函数中，等等。
* 在服务端渲染或渲染 Server Components 时，`prefetchDNS` 只有在你于渲染组件时或在源自渲染组件的异步上下文中调用它时才会生效。任何其他调用都会被忽略。
* 如果你知道自己需要哪些具体资源，可以改为调用[其他函数](/reference/react-dom/#resource-preloading-apis)，它们会立即开始加载资源。
* 预取网页自身所托管服务器的 DNS 没有收益，因为在发出该提示时，它的地址通常已经被查找过了。
* 与 [`preconnect`](/reference/react-dom/preconnect) 相比，如果你要推测性地连接大量域名，`prefetchDNS` 可能更合适，因为此时预连接的开销可能会超过收益。

---

## 用法 {/*usage*/}

### 在渲染时预取 DNS {/*prefetching-dns-when-rendering*/}

如果你知道子组件会从该主机加载外部资源，那么在渲染组件时调用 `prefetchDNS`。

```js
import { prefetchDNS } from 'react-dom';

function AppRoot() {
  prefetchDNS("https://example.com");
  return ...;
}
```

### 在事件处理函数中预取 DNS {/*prefetching-dns-in-an-event-handler*/}

在切换到需要外部资源的页面或状态之前，在事件处理函数中调用 `prefetchDNS`。这样会比在新页面或新状态渲染期间调用它更早开始该过程。

```js
import { prefetchDNS } from 'react-dom';

function CallToAction() {
  const onClick = () => {
    prefetchDNS('http://example.com');
    startWizard();
  }
  return (
    <button onClick={onClick}>开始向导</button>
  );
}
```
