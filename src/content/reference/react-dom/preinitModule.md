---
title: preinitModule
---

<Note>

[基于 React 的框架](/learn/creating-a-react-app)通常会为你处理资源加载，因此你可能不需要自己调用这个 API。详情请查阅你所使用框架的文档。

</Note>

<Intro>

`preinitModule` 允许你预先获取并执行一个 ESM 模块。

```js
preinitModule("https://example.com/module.js", {as: "script"});
```

</Intro>

<InlineToc />

---

## 参考 {/*reference*/}

### `preinitModule(href, options)` {/*preinitmodule*/}

要预初始化一个 ESM 模块，请从 `react-dom` 调用 `preinitModule` 函数。

```js
import { preinitModule } from 'react-dom';

function AppRoot() {
  preinitModule("https://example.com/module.js", {as: "script"});
  // ...
}

```

[在下方查看更多示例。](#usage)

`preinitModule` 函数会向浏览器提供一个提示，表示它应该开始下载并执行给定的模块，这可以节省时间。你 `preinit` 的模块会在下载完成后立即执行。

#### 参数 {/*parameters*/}

* `href`：字符串。你想要下载并执行的模块 URL。
* `options`：对象。它包含以下属性：
  *  `as`：必需的字符串。其值必须是 `'script'`。
  *  `crossOrigin`：字符串。要使用的 [CORS 策略](https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes/crossorigin)。可选值为 `anonymous` 和 `use-credentials`。
  *  `integrity`：字符串。模块的加密哈希值，用于[验证其真实性](https://developer.mozilla.org/en-US/docs/Web/Security/Subresource_Integrity)。
  *  `nonce`：字符串。在使用严格内容安全策略时，用于允许该模块的加密[nonce](https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/nonce)。

#### 返回值 {/*returns*/}

`preinitModule` 不返回任何内容。

#### 注意事项 {/*caveats*/}

* 多次使用相同的 `href` 调用 `preinitModule` 与调用一次的效果相同。
* 在浏览器中，你可以在任何情况下调用 `preinitModule`：在渲染组件时、在 Effect 中、在事件处理程序中，等等。
* 在服务端渲染或渲染 Server Components 时，只有当你在渲染组件时或在源自渲染组件的异步上下文中调用 `preinitModule`，它才会生效。任何其他调用都会被忽略。

---

## 用法 {/*usage*/}

### 在渲染时预加载 {/*preloading-when-rendering*/}

如果你知道某个组件或其子组件将使用特定模块，并且你接受该模块被执行后在下载完成时立即生效，那么请在渲染组件时调用 `preinitModule`。

```js
import { preinitModule } from 'react-dom';

function AppRoot() {
  preinitModule("https://example.com/module.js", {as: "script"});
  return ...;
}
```

如果你希望浏览器下载该模块但不要立即执行它，请改用 [`preloadModule`](/reference/react-dom/preloadModule)。如果你想预初始化的不是 ESM 模块的脚本，请使用 [`preinit`](/reference/react-dom/preinit)。

### 在事件处理程序中预加载 {/*preloading-in-an-event-handler*/}

在事件处理程序中、在转换到需要该模块的页面或状态之前调用 `preinitModule`。这样会比你在新页面或新状态渲染期间调用它更早地启动流程。

```js
import { preinitModule } from 'react-dom';

function CallToAction() {
  const onClick = () => {
    preinitModule("https://example.com/module.js", {as: "script"});
    startWizard();
  }
  return (
    <button onClick={onClick}>开始向导</button>
  );
}
```
