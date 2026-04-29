---
title: "<textarea>"
---

<Intro>

内置浏览器 `<textarea>` 组件的 [内置浏览器 `<textarea>` 组件](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/textarea) 可让你渲染一个多行文本输入框。

```js
<textarea />
```

</Intro>

<InlineToc />

---

## 参考 {/*reference*/}

### `<textarea>` {/*textarea*/}

要显示一个文本区域，请渲染 [内置浏览器 `<textarea>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/textarea) 组件。

```js
<textarea name="postContent" />
```

[在下方查看更多示例。](#usage)

#### 属性 {/*props*/}

`<textarea>` 支持所有 [通用元素属性。](/reference/react-dom/components/common#common-props)

你可以通过传递 `value` 属性来 [将文本区域设为受控](#controlling-a-text-area-with-a-state-variable)：

* `value`: 一个字符串。控制文本区域中的文本。

当你传入 `value` 时，你还必须传入一个会更新所传值的 `onChange` 处理函数。

如果你的 `<textarea>` 是非受控的，你可以改为传入 `defaultValue` 属性：

* `defaultValue`: 一个字符串。为文本区域指定 [初始值](#providing-an-initial-value-for-a-text-area)。

以下 `<textarea>` 属性对非受控和受控文本区域都适用：

* [`autoComplete`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/textarea#autocomplete): `'on'` 或 `'off'`。指定自动完成行为。
* [`autoFocus`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/textarea#autofocus): 一个布尔值。如果为 `true`，React 会在挂载时聚焦该元素。
* `children`: `<textarea>` 不接受子元素。要设置初始值，请使用 `defaultValue`。
* [`cols`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/textarea#cols): 一个数字。指定按平均字符宽度计算的默认宽度。默认值为 `20`。
* [`disabled`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/textarea#disabled): 一个布尔值。如果为 `true`，输入框将不可交互，并且会显示为灰暗状态。
* [`form`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/textarea#form): 一个字符串。指定此输入所属 `<form>` 的 `id`。如果省略，则为最近的父级 form。
* [`maxLength`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/textarea#maxlength): 一个数字。指定文本的最大长度。
* [`minLength`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/textarea#minlength): 一个数字。指定文本的最小长度。
* [`name`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input#name): 一个字符串。指定此输入的名称，该名称会随表单一起 [提交。](#reading-the-textarea-value-when-submitting-a-form)
* `onChange`: 一个 [`Event` 处理函数](/reference/react-dom/components/common#event-handler) 函数。对于 [受控文本区域](#controlling-a-text-area-with-a-state-variable) 是必需的。当输入值被用户更改时会立即触发（例如，每次按键都会触发）。其行为类似于浏览器的 [`input` 事件。](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/input_event)
* `onChangeCapture`: `onChange` 的一个版本，会在 [捕获阶段。](/learn/responding-to-events#capture-phase-events) 触发
* [`onInput`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/input_event): 一个 [`Event` 处理函数](/reference/react-dom/components/common#event-handler) 函数。当值被用户更改时会立即触发。出于历史原因，在 React 中通常使用 `onChange` 代替，它的工作方式类似。
* `onInputCapture`: `onInput` 的一个版本，会在 [捕获阶段。](/learn/responding-to-events#capture-phase-events) 触发
* [`onInvalid`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLInputElement/invalid_event): 一个 [`Event` 处理函数](/reference/react-dom/components/common#event-handler) 函数。当输入在表单提交时未通过验证就会触发。不同于内置的 `invalid` 事件，React 的 `onInvalid` 事件会冒泡。
* `onInvalidCapture`: `onInvalid` 的一个版本，会在 [捕获阶段。](/learn/responding-to-events#capture-phase-events) 触发
* [`onSelect`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLTextAreaElement/select_event): 一个 [`Event` 处理函数](/reference/react-dom/components/common#event-handler) 函数。当 `<textarea>` 内的选区发生变化后触发。React 扩展了 `onSelect` 事件，使其在空选择和编辑时也会触发（这可能会影响选区）。
* `onSelectCapture`: `onSelect` 的一个版本，会在 [捕获阶段。](/learn/responding-to-events#capture-phase-events) 触发
* [`placeholder`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/textarea#placeholder): 一个字符串。当文本区域的值为空时，以灰暗颜色显示。
* [`readOnly`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/textarea#readonly): 一个布尔值。如果为 `true`，用户无法编辑文本区域。
* [`required`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/textarea#required): 一个布尔值。如果为 `true`，则必须提供该值表单才能提交。
* [`rows`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/textarea#rows): 一个数字。指定按平均字符高度计算的默认高度。默认值为 `2`。
* [`wrap`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/textarea#wrap): `'hard'`、`'soft'` 或 `'off'`。指定提交表单时文本应如何换行。

#### 注意事项 {/*caveats*/}

- 不允许传入类似 `<textarea>something</textarea>` 的子元素。[请使用 `defaultValue` 来设置初始内容。](#providing-an-initial-value-for-a-text-area)
- 如果文本区域接收到一个字符串类型的 `value` 属性，它将被 [视为受控组件。](#controlling-a-text-area-with-a-state-variable)
- 文本区域不能同时既受控又非受控。
- 文本区域在其生命周期内不能在受控与非受控之间切换。
- 每个受控文本区域都需要一个 `onChange` 事件处理函数，以同步更新其背后的值。

---

## 用法 {/*usage*/}

### 显示一个文本区域 {/*displaying-a-text-area*/}

渲染 `<textarea>` 来显示一个文本区域。你可以使用 [`rows`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/textarea#rows) 和 [`cols`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/textarea#cols) 属性来指定其默认大小，但默认情况下用户可以调整它的大小。要禁用调整大小，你可以在 CSS 中指定 `resize: none`。

<Sandpack>

```js
export default function NewPost() {
  return (
    <label>
      写下你的帖子：
      <textarea name="postContent" rows={4} cols={40} />
    </label>
  );
}
```

```css
input { margin-left: 5px; }
textarea { margin-top: 10px; }
label { margin: 10px; }
label, textarea { display: block; }
```

</Sandpack>

---

### 为文本区域提供标签 {/*providing-a-label-for-a-text-area*/}

通常，你会将每个 `<textarea>` 放在 [`<label>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/label) 标签内。这会告诉浏览器该标签与这个文本区域相关联。当用户点击标签时，浏览器会聚焦文本区域。这对可访问性也很重要：当用户聚焦文本区域时，屏幕阅读器会朗读该标签文本。

如果你不能把 `<textarea>` 嵌套进 `<label>` 中，可以通过向 `<textarea id>` 和 [`<label htmlFor>`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLLabelElement/htmlFor) 传入相同的 ID 来关联它们。为避免同一个组件的多个实例之间发生冲突，可以使用 [`useId`](/reference/react/useId) 生成这样的 ID。

<Sandpack>

```js
import { useId } from 'react';

export default function Form() {
  const postTextAreaId = useId();
  return (
    <>
      <label htmlFor={postTextAreaId}>
        写下你的帖子：
      </label>
      <textarea
        id={postTextAreaId}
        name="postContent"
        rows={4}
        cols={40}
      />
    </>
  );
}
```

```css
input { margin: 5px; }
```

</Sandpack>

---

### 提供文本区域的初始值 {/*providing-an-initial-value-for-a-text-area*/}

你可以选择性地指定文本区域的初始值。将其作为 `defaultValue` 字符串传入。

<Sandpack>

```js
export default function EditPost() {
  return (
    <label>
      编辑你的帖子：
      <textarea
        name="postContent"
        defaultValue="我昨天骑车真的很开心！"
        rows={4}
        cols={40}
      />
    </label>
  );
}
```

```css
input { margin-left: 5px; }
textarea { margin-top: 10px; }
label { margin: 10px; }
label, textarea { display: block; }
```

</Sandpack>

<Pitfall>

不同于 HTML，不支持像 `<textarea>Some content</textarea>` 这样传入初始文本。

</Pitfall>

---

### 在提交表单时读取文本区域的值 {/*reading-the-text-area-value-when-submitting-a-form*/}

在你的 textarea 外面包一层 [`<form>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/form)，并在里面放一个 [`<button type="submit">`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/button)。它会调用你的 `<form onSubmit>` 事件处理函数。默认情况下，浏览器会将表单数据发送到当前 URL 并刷新页面。你可以通过调用 `e.preventDefault()` 来覆盖这一行为。使用 [`new FormData(e.target)`](https://developer.mozilla.org/en-US/docs/Web/API/FormData) 读取表单数据。
<Sandpack>

```js
export default function EditPost() {
  function handleSubmit(e) {
    // 阻止浏览器重新加载页面
    e.preventDefault();

    // 读取表单数据
    const form = e.target;
    const formData = new FormData(form);

    // 你可以直接将 formData 作为 fetch 请求体：
    fetch('/some-api', { method: form.method, body: formData });

    // 或者你可以把它当作普通对象来处理：
    const formJson = Object.fromEntries(formData.entries());
    console.log(formJson);
  }

  return (
    <form method="post" onSubmit={handleSubmit}>
      <label>
        帖子标题： <input name="postTitle" defaultValue="骑车" />
      </label>
      <label>
        编辑你的帖子：
        <textarea
          name="postContent"
          defaultValue="我昨天骑车真的很开心！"
          rows={4}
          cols={40}
        />
      </label>
      <hr />
      <button type="reset">重置编辑</button>
      <button type="submit">保存帖子</button>
    </form>
  );
}
```

```css
label { display: block; }
input { margin: 5px; }
```

</Sandpack>

<Note>

给你的 `<textarea>` 加上一个 `name`，例如 `<textarea name="postContent" />`。你指定的 `name` 会作为表单数据中的键使用，例如 `{ postContent: "Your post" }`。

</Note>

<Pitfall>

默认情况下，*任何* 位于 `<form>` 内的 `<button>` 都会提交表单。这可能会出人意料！如果你有自己的自定义 `Button` React 组件，考虑返回 [`<button type="button">`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input/button) 而不是 `<button>`。然后，为了明确起见，对那些*确实*应该提交表单的按钮使用 `<button type="submit">`。

</Pitfall>

---

### 使用状态变量控制文本区域 {/*controlling-a-text-area-with-a-state-variable*/}

像 `<textarea />` 这样的文本区域是*非受控*的。即使你 [传入初始值](#providing-an-initial-value-for-a-text-area)，比如 `<textarea defaultValue="Initial text" />`，你的 JSX 也只是指定了初始值，而不是当前值。

**要渲染一个 _受控_ 文本区域，请向它传入 `value` 属性。** React 会强制文本区域始终拥有你传入的 `value`。通常，你会通过声明一个 [状态变量：](/reference/react/useState) 来控制文本区域

```js {2,6,7}
function NewPost() {
  const [postContent, setPostContent] = useState(''); // 声明一个状态变量...
  // ...
  return (
    <textarea
      value={postContent} // ...强制输入框的值与状态变量保持一致...
      onChange={e => setPostContent(e.target.value)} // ...并在任何编辑时更新状态变量！
    />
  );
}
```

如果你想在每次按键时重新渲染 UI 的某一部分，这会很有用。

<Sandpack>

```js
import { useState } from 'react';
import MarkdownPreview from './MarkdownPreview.js';

export default function MarkdownEditor() {
  const [postContent, setPostContent] = useState('_Hello,_ **Markdown**!');
  return (
    <>
      <label>
        输入一些 markdown：
        <textarea
          value={postContent}
          onChange={e => setPostContent(e.target.value)}
        />
      </label>
      <hr />
      <MarkdownPreview markdown={postContent} />
    </>
  );
}
```

```js src/MarkdownPreview.js
import { Remarkable } from 'remarkable';

const md = new Remarkable();

export default function MarkdownPreview({ markdown }) {
  const renderedHTML = md.render(markdown);
  return <div dangerouslySetInnerHTML={{__html: renderedHTML}} />;
}
```

```json package.json
{
  "dependencies": {
    "react": "latest",
    "react-dom": "latest",
    "react-scripts": "latest",
    "remarkable": "2.0.1"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject"
  }
}
```

```css
textarea { display: block; margin-top: 5px; margin-bottom: 10px; }
```

</Sandpack>

<Pitfall>

**如果你传入 `value` 却不传 `onChange`，那么就无法在文本区域中输入内容。** 当你通过传递某个 `value` 来控制文本区域时，你会*强制*它始终拥有你传入的值。因此，如果你把一个状态变量作为 `value` 传入，却忘记在 `onChange` 事件处理函数期间同步更新该状态变量，React 就会在每次按键后把文本区域恢复回你指定的 `value`。

</Pitfall>

---

## 故障排查 {/*troubleshooting*/}

### 我输入时文本区域没有更新 {/*my-text-area-doesnt-update-when-i-type-into-it*/}

如果你渲染了一个带有 `value` 但没有 `onChange` 的文本区域，你会在控制台中看到一个错误：

```js
// 🔴 错误：没有 onChange 处理器的受控文本区域
<textarea value={something} />
```

<ConsoleBlock level="error">

你为一个表单字段提供了 `value` 属性，但没有提供 `onChange` 处理器。这将渲染一个只读字段。如果该字段应该是可修改的，请使用 `defaultValue`。否则，请设置 `onChange` 或 `readOnly`。

</ConsoleBlock>

正如错误信息所示，如果你只是想[指定*初始*值，](#providing-an-initial-value-for-a-text-area)请改为传递 `defaultValue`：

```js
// ✅ 好的：带有初始值的非受控文本区域
<textarea defaultValue={something} />
```

如果你想[使用状态变量来控制这个文本区域，](#controlling-a-text-area-with-a-state-variable)请指定一个 `onChange` 处理器：

```js
// ✅ 好的：带有 onChange 的受控文本区域
<textarea value={something} onChange={e => setSomething(e.target.value)} />
```

如果该值有意设置为只读，请添加 `readOnly` 属性以抑制该错误：

```js
// ✅ 好的：没有 onChange 的只读受控文本区域
<textarea value={something} readOnly={true} />
```

---

### 我文本区域里的光标在每次按键时都会跳到开头 {/*my-text-area-caret-jumps-to-the-beginning-on-every-keystroke*/}

如果你[控制一个文本区域，](#controlling-a-text-area-with-a-state-variable)你必须在 `onChange` 期间将其状态变量更新为来自 DOM 的文本区域值。

你不能把它更新为 `e.target.value` 之外的其他内容：

```js
function handleChange(e) {
  // 🔴 错误：将输入更新为 e.target.value 之外的内容
  setFirstName(e.target.value.toUpperCase());
}
```

你也不能异步更新它：

```js
function handleChange(e) {
  // 🔴 错误：异步更新输入
  setTimeout(() => {
    setFirstName(e.target.value);
  }, 100);
}
```

要修复你的代码，请同步将其更新为 `e.target.value`：

```js
function handleChange(e) {
  // ✅ 将受控输入同步更新为 e.target.value
  setFirstName(e.target.value);
}
```

如果这不能解决问题，可能是文本区域在每次按键时都会从 DOM 中被移除并重新添加。如果你在每次重新渲染时不小心[重置了状态](/learn/preserving-and-resetting-state)，就可能发生这种情况。例如，如果文本区域或它的某个父级总是接收到不同的 `key` 属性，就会发生这种情况；或者如果你嵌套了组件定义（这是 React 不允许的，并且会导致“内部”组件在每次渲染时重新挂载）。

---

### 我遇到了一个错误：“组件正在将非受控输入变为受控输入” {/*im-getting-an-error-a-component-is-changing-an-uncontrolled-input-to-be-controlled*/}


如果你为组件提供了 `value`，那么它在整个生命周期中都必须保持为字符串。

你不能先传入 `value={undefined}`，然后再传入 `value="some string"`，因为 React 不知道你是想让组件保持非受控还是受控。受控组件应该始终接收一个字符串类型的 `value`，而不是 `null` 或 `undefined`。

如果你的 `value` 来自 API 或状态变量，它可能一开始被初始化为 `null` 或 `undefined`。在这种情况下，请么最初将其设为空字符串（`''`），要么传入 `value={someValue ?? ''}` 以确保 `value` 是字符串。
