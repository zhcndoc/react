---
title: preinit
---

<Note>

[基于 React 的框架](/learn/creating-a-react-app)通常会为你处理资源加载，因此你可能不必自己调用此 API。请查阅你的框架文档以了解详情。

</Note>

<Intro>

`preinit` 可让你主动获取并执行样式表或外部脚本。

```js
preinit("https://example.com/script.js", {as: "script"});
```

</Intro>

<InlineToc />

---

## 参考 {/*reference*/}

### `preinit(href, options)` {/*preinit*/}

要预初始化脚本或样式表，请从 `react-dom` 调用 `preinit` 函数。

```js
import { preinit } from 'react-dom';

function AppRoot() {
  preinit("https://example.com/script.js", {as: "script"});
  // ...
}

```

[查看更多示例。](#usage)

`preinit` 函数会向浏览器提供一个提示，表示它应该开始下载并执行给定资源，这可以节省时间。你 `preinit` 的脚本会在下载完成后执行。你预初始化的样式表会被插入到文档中，从而立即生效。

#### 参数 {/*parameters*/}

* `href`：一个字符串。你想要下载并执行的资源 URL。
* `options`：一个对象。它包含以下属性：
  * `as`：必需的字符串。资源类型。可选值为 `script` 和 `style`。
  * `precedence`：一个字符串。样式表必需。表示将样式表插入到相对于其他样式表的哪个位置。优先级更高的样式表可以覆盖优先级更低的样式表。可选值为 `reset`、`low`、`medium`、`high`。
  * `crossOrigin`：一个字符串。要使用的 [CORS 策略](https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes/crossorigin)。可选值为 `anonymous` 和 `use-credentials`。
  * `integrity`：一个字符串。资源的加密哈希，用于[验证其真实性](https://developer.mozilla.org/en-US/docs/Web/Security/Subresource_Integrity)。
  * `nonce`：一个字符串。在使用严格内容安全策略时，用于[允许该资源的加密 nonce](https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/nonce)。
  * `fetchPriority`：一个字符串。建议资源获取的相对优先级。可选值为 `auto`（默认）、`high` 和 `low`。

#### 返回值 {/*returns*/}

`preinit` 不返回任何内容。

#### 注意事项 {/*caveats*/}

* 多次使用相同 `href` 调用 `preinit` 的效果与调用一次相同。
* 在浏览器中，你可以在任何情况下调用 `preinit`：在渲染组件时、在 Effect 中、在事件处理函数中，等等。
* 在服务端渲染或渲染 Server Components 时，只有当你在渲染组件时调用 `preinit`，或在源自渲染组件的异步上下文中调用时，它才会生效。任何其他调用都会被忽略。

---

## 用法 {/*usage*/}

### 在渲染时预初始化 {/*preiniting-when-rendering*/}

如果你知道某个组件或其子组件会使用特定资源，并且你可以接受该资源在下载完成后立即被执行并因此生效，那么就在渲染组件时调用 `preinit`。

<Recipes titleText="预初始化示例">

#### 预初始化外部脚本 {/*preiniting-an-external-script*/}

```js
import { preinit } from 'react-dom';

function AppRoot() {
  preinit("https://example.com/script.js", {as: "script"});
  return ...;
}
```

如果你希望浏览器下载脚本但不要立即执行它，请改用 [`preload`](/reference/react-dom/preload)。如果你想加载一个 ESM 模块，请使用 [`preinitModule`](/reference/react-dom/preinitModule)。

<Solution />

#### 预初始化样式表 {/*preiniting-a-stylesheet*/}

```js
import { preinit } from 'react-dom';

function AppRoot() {
  preinit("https://example.com/style.css", {as: "style", precedence: "medium"});
  return ...;
}
```

`precedence` 选项是必需的，它可以让你控制文档中样式表的顺序。优先级更高的样式表可以覆盖优先级更低的样式表。

如果你希望下载样式表但不要立即将其插入文档，请改用 [`preload`](/reference/react-dom/preload)。

<Solution />

</Recipes>

### 在事件处理函数中预初始化 {/*preiniting-in-an-event-handler*/}

在切换到需要外部资源的页面或状态之前，可以在事件处理函数中调用 `preinit`。这会比在新页面或新状态渲染期间调用它更早开始该过程。

```js
import { preinit } from 'react-dom';

function CallToAction() {
  const onClick = () => {
    preinit("https://example.com/wizardStyles.css", {as: "style"});
    startWizard();
  }
  return (
    <button onClick={onClick}>Start Wizard</button>
  );
}
```
