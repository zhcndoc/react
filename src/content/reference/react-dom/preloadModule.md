---
title: preloadModule
---

<Note>

[基于 React 的框架](/learn/creating-a-react-app)通常会为你处理资源加载，因此你可能不需要自己调用此 API。详情请参阅你所使用框架的文档。

</Note>

<Intro>

`preloadModule` 让你能够预先获取你预计会使用的 ESM 模块。

```js
preloadModule("https://example.com/module.js", {as: "script"});
```

</Intro>

<InlineToc />

---

## 参考 {/*reference*/}

### `preloadModule(href, options)` {/*preloadmodule*/}

要预加载一个 ESM 模块，请从 `react-dom` 调用 `preloadModule` 函数。

```js
import { preloadModule } from 'react-dom';

function AppRoot() {
  preloadModule("https://example.com/module.js", {as: "script"});
  // ...
}

```

[查看更多示例。](#usage)

`preloadModule` 函数会向浏览器提供一个提示，告诉它应该开始下载给定的模块，这样可以节省时间。

#### 参数 {/*parameters*/}

* `href`：一个字符串。你要下载的模块的 URL。
* `options`：一个对象。它包含以下属性：
  *  `as`：必需的字符串。必须是 `'script'`。
  *  `crossOrigin`：一个字符串。要使用的 [CORS 策略](https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes/crossorigin)。可选值为 `anonymous` 和 `use-credentials`。
  *  `integrity`：一个字符串。模块的加密哈希，用于[验证其真实性](https://developer.mozilla.org/en-US/docs/Web/Security/Subresource_Integrity)。
  *  `nonce`：一个字符串。在使用严格的内容安全策略时，用于允许该模块的加密[nonce](https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/nonce)。


#### 返回值 {/*returns*/}

`preloadModule` 不返回任何内容。

#### 注意事项 {/*caveats*/}

* 对同一个 `href` 多次调用 `preloadModule` 的效果与调用一次相同。
* 在浏览器中，你可以在任何情况下调用 `preloadModule`：在渲染组件时、在 Effect 中、在事件处理函数中，等等。
* 在服务端渲染或渲染 Server Components 时，`preloadModule` 只有在你渲染组件时调用它，或者在源自渲染组件的异步上下文中调用它时才会生效。任何其他调用都会被忽略。

---

## 用法 {/*usage*/}

### 在渲染时预加载 {/*preloading-when-rendering*/}

如果你知道某个组件或其子组件将使用特定模块，请在渲染组件时调用 `preloadModule`。

```js
import { preloadModule } from 'react-dom';

function AppRoot() {
  preloadModule("https://example.com/module.js", {as: "script"});
  return ...;
}
```

如果你希望浏览器立即开始执行该模块（而不仅仅是下载它），请改用 [`preinitModule`](/reference/react-dom/preinitModule)。如果你想加载的不是 ESM 模块的脚本，请使用 [`preload`](/reference/react-dom/preload)。

### 在事件处理函数中预加载 {/*preloading-in-an-event-handler*/}

在事件处理函数中，在切换到需要该模块的页面或状态之前调用 `preloadModule`。这样可以比在新页面或新状态渲染期间调用更早开始该过程。

```js
import { preloadModule } from 'react-dom';

function CallToAction() {
  const onClick = () => {
    preloadModule("https://example.com/module.js", {as: "script"});
    startWizard();
  }
  return (
    <button onClick={onClick}>Start Wizard</button>
  );
}
```
