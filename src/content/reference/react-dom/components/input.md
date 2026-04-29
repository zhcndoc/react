---
title: "<input>"
---

<Intro>

[内置浏览器 `<input>` 组件](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input) 让你可以渲染不同类型的表单输入。

```js
<input />
```

</Intro>

<InlineToc />

---

## 参考 {/*reference*/}

### `<input>` {/*input*/}

要显示一个输入框，渲染 [内置浏览器 `<input>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input) 组件。

```js
<input name="myInput" />
```

[在下方查看更多示例。](#usage)

#### Props {/*props*/}

`<input>` 支持所有 [常见元素 props。](/reference/react-dom/components/common#common-props)

- [`formAction`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input#formaction)：字符串或函数。会覆盖 `type="submit"` 和 `type="image"` 的父级 `<form action>`。当向 `action` 传入 URL 时，表单行为与标准 HTML 表单相同。当向 `formAction` 传入函数时，该函数将处理表单提交。参见 [`<form action>`](/reference/react-dom/components/form#props)。

你可以通过传入以下 props 之一来 [将输入框变为受控](#controlling-an-input-with-a-state-variable)：

* [`checked`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLInputElement#checked)：布尔值。对于复选框输入或单选按钮，用于控制其是否被选中。
* [`value`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLInputElement#value)：字符串。对于文本输入，控制其文本。（对于单选按钮，指定其表单数据。）

当你传入其中任意一个时，还必须传入一个会更新所传值的 `onChange` 处理函数。

这些 `<input>` props 仅与非受控输入相关：

* [`defaultChecked`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLInputElement#defaultChecked)：布尔值。为 `type="checkbox"` 和 `type="radio"` 输入指定 [初始值](#providing-an-initial-value-for-an-input)。
* [`defaultValue`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLInputElement#defaultValue)：字符串。为文本输入指定 [初始值](#providing-an-initial-value-for-an-input)。

这些 `<input>` props 同时与非受控和受控输入相关：

* [`accept`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input#accept)：字符串。指定 `type="file"` 输入接受哪些文件类型。
* [`alt`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input#alt)：字符串。为 `type="image"` 输入指定替代图像文本。
* [`capture`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input#capture)：字符串。指定 `type="file"` 输入捕获的媒体（麦克风、视频或相机）。
* [`autoComplete`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input#autocomplete)：字符串。指定可能的 [自动完成行为。](https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes/autocomplete#values)
* [`autoFocus`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input#autofocus)：布尔值。如果为 `true`，React 会在挂载时聚焦该元素。
* [`dirname`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input#dirname)：字符串。指定该元素方向性的表单字段名。
* [`disabled`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input#disabled)：布尔值。如果为 `true`，输入框将不可交互，并且会显示为灰暗状态。
* `children`：`<input>` 不接受子元素。
* [`form`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input#form)：字符串。指定此输入框所属 `<form>` 的 `id`。如果省略，则为最近的父级表单。
* [`formAction`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input#formaction)：字符串。会覆盖 `type="submit"` 和 `type="image"` 的父级 `<form action>`。
* [`formEnctype`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input#formenctype)：字符串。会覆盖 `type="submit"` 和 `type="image"` 的父级 `<form enctype>`。
* [`formMethod`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input#formmethod)：字符串。会覆盖 `type="submit"` 和 `type="image"` 的父级 `<form method>`。
* [`formNoValidate`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input#formnovalidate)：字符串。会覆盖 `type="submit"` 和 `type="image"` 的父级 `<form noValidate>`。
* [`formTarget`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input#formtarget)：字符串。会覆盖 `type="submit"` 和 `type="image"` 的父级 `<form target>`。
* [`height`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input#height)：字符串。指定 `type="image"` 输入的图像高度。
* [`list`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input#list)：字符串。指定包含自动完成选项的 `<datalist>` 的 `id`。
* [`max`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input#max)：数值。指定数值和日期时间输入的最大值。
* [`maxLength`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input#maxlength)：数值。指定文本和其他输入的最大长度。
* [`min`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input#min)：数值。指定数值和日期时间输入的最小值。
* [`minLength`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input#minlength)：数值。指定文本和其他输入的最小长度。
* [`multiple`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input#multiple)：布尔值。指定 `<type="file"` 和 `type="email"` 是否允许多个值。
* [`name`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input#name)：字符串。指定该输入框的名称，并会 [随表单一起提交。](#reading-the-input-values-when-submitting-a-form)
* `onChange`：一个 [`Event` 处理函数](/reference/react-dom/components/common#event-handler)。受控输入 [必需。](#controlling-an-input-with-a-state-variable) 当用户更改输入值时立即触发（例如，每次按键都会触发）。行为类似浏览器的 [`input` 事件。](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/input_event)
* `onChangeCapture`：`onChange` 的一个版本，会在 [捕获阶段。](/learn/responding-to-events#capture-phase-events) 触发
* [`onInput`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/input_event)：一个 [`Event` 处理函数](/reference/react-dom/components/common#event-handler)。当值被用户更改时立即触发。由于历史原因，在 React 中通常使用同样类似的 `onChange` 来代替。
* `onInputCapture`：`onInput` 的一个版本，会在 [捕获阶段。](/learn/responding-to-events#capture-phase-events) 触发
* [`onInvalid`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLInputElement/invalid_event)：一个 [`Event` 处理函数](/reference/react-dom/components/common#event-handler)。当输入框在表单提交时未通过验证时触发。与内置的 `invalid` 事件不同，React 的 `onInvalid` 事件会冒泡。
* `onInvalidCapture`：`onInvalid` 的一个版本，会在 [捕获阶段。](/learn/responding-to-events#capture-phase-events) 触发
* [`onSelect`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLInputElement/select_event)：一个 [`Event` 处理函数](/reference/react-dom/components/common#event-handler)。当 `<input>` 内的选区发生变化后触发。React 扩展了 `onSelect` 事件，使其也会在空选区以及编辑时触发（这可能会影响选区）。
* `onSelectCapture`：`onSelect` 的一个版本，会在 [捕获阶段。](/learn/responding-to-events#capture-phase-events) 触发
* [`pattern`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input#pattern)：字符串。指定 `value` 必须匹配的模式。
* [`placeholder`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input#placeholder)：字符串。当输入值为空时，以灰暗颜色显示。
* [`readOnly`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input#readonly)：布尔值。如果为 `true`，用户无法编辑该输入框。
* [`required`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input#required)：布尔值。如果为 `true`，则必须提供该值表单才能提交。
* [`size`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input#size)：数值。类似于设置宽度，但单位取决于控件。
* [`src`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input#src)：字符串。为 `type="image"` 输入指定图像来源。
* [`step`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input#step)：正数或 `'any'` 字符串。指定有效值之间的间隔。
* [`type`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input#type)：字符串。属于 [输入类型。](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input#input_types) 之一
* [`width`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input#width)：字符串。指定 `type="image"` 输入的图像宽度。

#### 注意事项 {/*caveats*/}

- 复选框需要 `checked`（或 `defaultChecked`），而不是 `value`（或 `defaultValue`）。
- 如果文本输入接收一个字符串 `value` prop，它会被 [视为受控。](#controlling-an-input-with-a-state-variable)
- 如果复选框或单选按钮接收一个布尔值 `checked` prop，它会被 [视为受控。](#controlling-an-input-with-a-state-variable)
- 一个输入框不能同时既是受控的又是非受控的。
- 输入框在其生命周期内不能在受控和非受控之间切换。
- 每个受控输入都需要一个 `onChange` 事件处理函数，用于同步更新其底层值。

---

## 用法 {/*usage*/}

### 显示不同类型的输入 {/*displaying-inputs-of-different-types*/}

要显示一个输入框，渲染一个 `<input>` 组件。默认情况下，它会是一个文本输入框。你可以传入 `type="checkbox"` 来创建复选框，传入 `type="radio"` 来创建单选按钮，[或其他输入类型之一。](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input#input_types)

<Sandpack>

```js
export default function MyForm() {
  return (
    <>
      <label>
        文本输入：<input name="myInput" />
      </label>
      <hr />
      <label>
        复选框：<input type="checkbox" name="myCheckbox" />
      </label>
      <hr />
      <p>
        单选按钮：
        <label>
          <input type="radio" name="myRadio" value="option1" />
          选项 1
        </label>
        <label>
          <input type="radio" name="myRadio" value="option2" />
          选项 2
        </label>
        <label>
          <input type="radio" name="myRadio" value="option3" />
          选项 3
        </label>
      </p>
    </>
  );
}
```

```css
label { display: block; }
input { margin: 5px; }
```

</Sandpack>

---

### 为输入提供标签 {/*providing-a-label-for-an-input*/}

通常，你会把每个 `<input>` 放在一个 [`<label>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/label) 标签内。这会告诉浏览器这个标签与该输入相关联。当用户点击标签时，浏览器会自动聚焦输入框。对于可访问性来说，这也很重要：当用户聚焦关联的输入时，屏幕阅读器会朗读该标签文本。

如果你不能把 `<input>` 嵌套进 `<label>`，可以通过向 `<input id>` 和 [`<label htmlFor>`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLLabelElement/htmlFor) 传入相同的 ID 来关联它们。为避免同一个组件的多个实例之间发生冲突，可以使用 [`useId`](/reference/react/useId) 生成这样的 ID。

<Sandpack>

```js
import { useId } from 'react';

export default function Form() {
  const ageInputId = useId();
  return (
    <>
      <label>
        你的名字：
        <input name="firstName" />
      </label>
      <hr />
      <label htmlFor={ageInputId}>你的年龄：</label>
      <input id={ageInputId} name="age" type="number" />
    </>
  );
}
```

```css
input { margin: 5px; }
```

</Sandpack>

---

### 为输入提供初始值 {/*providing-an-initial-value-for-an-input*/}

你可以为任何输入可选地指定初始值。对于文本输入，将其作为 `defaultValue` 字符串传入。复选框和单选按钮则应使用 `defaultChecked` 布尔值来指定初始值。

<Sandpack>

```js
export default function MyForm() {
  return (
    <>
      <label>
        文本输入：<input name="myInput" defaultValue="Some initial value" />
      </label>
      <hr />
      <label>
        复选框：<input type="checkbox" name="myCheckbox" defaultChecked={true} />
      </label>
      <hr />
      <p>
        单选按钮：
        <label>
          <input type="radio" name="myRadio" value="option1" />
          选项 1
        </label>
        <label>
          <input
            type="radio"
            name="myRadio"
            value="option2"
            defaultChecked={true}
          />
          选项 2
        </label>
        <label>
          <input type="radio" name="myRadio" value="option3" />
          选项 3
        </label>
      </p>
    </>
  );
}
```

```css
label { display: block; }
input { margin: 5px; }
```

</Sandpack>

---

### 提交表单时读取输入值 {/*reading-the-input-values-when-submitting-a-form*/}

在输入框外包一层 [`<form>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/form)，并在内部放一个 [`<button type="submit">`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/button)。它会调用你的 `<form onSubmit>` 事件处理函数。默认情况下，浏览器会将表单数据发送到当前 URL 并刷新页面。你可以通过调用 `e.preventDefault()` 来覆盖这种行为。使用 [`new FormData(e.target)`](https://developer.mozilla.org/en-US/docs/Web/API/FormData) 读取表单数据。
<Sandpack>

```js
export default function MyForm() {
  function handleSubmit(e) {
    // 阻止浏览器重新加载页面
    e.preventDefault();

    // 读取表单数据
    const form = e.target;
    const formData = new FormData(form);

    // 你可以直接将 formData 作为 fetch 的 body 传入：
    fetch('/some-api', { method: form.method, body: formData });

    // 或者你也可以将其作为普通对象来处理：
    const formJson = Object.fromEntries(formData.entries());
    console.log(formJson);
  }

  return (
    <form method="post" onSubmit={handleSubmit}>
      <label>
        文本输入：<input name="myInput" defaultValue="Some initial value" />
      </label>
      <hr />
      <label>
        复选框：<input type="checkbox" name="myCheckbox" defaultChecked={true} />
      </label>
      <hr />
      <p>
        单选按钮：
        <label><input type="radio" name="myRadio" value="option1" /> 选项 1</label>
        <label><input type="radio" name="myRadio" value="option2" defaultChecked={true} /> 选项 2</label>
        <label><input type="radio" name="myRadio" value="option3" /> 选项 3</label>
      </p>
      <hr />
      <button type="reset">重置表单</button>
      <button type="submit">提交表单</button>
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

给每个 `<input>` 都加上一个 `name`，例如 `<input name="firstName" defaultValue="Taylor" />`。你指定的 `name` 会作为键用于表单数据，例如 `{ firstName: "Taylor" }`。

</Note>

<Pitfall>

默认情况下，`<form>` 中没有 `type` 属性的 `<button>` 会提交表单。这可能出人意料！如果你有自己的自定义 `Button` React 组件，考虑使用 [`<button type="button">`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/button) 代替 `<button>`（不带 type）。然后，为了明确起见，将 `<button type="submit">` 用于那些*确实*应该提交表单的按钮。

</Pitfall>

---

### 使用状态变量控制输入 {/*controlling-an-input-with-a-state-variable*/}

像 `<input />` 这样的输入是*非受控*的。即使你 [传入初始值](#providing-an-initial-value-for-an-input)，例如 `<input defaultValue="Initial text" />`，你的 JSX 也只是指定了初始值。它并不会控制当前值应该是什么。

**要渲染一个 _受控_ 输入，请向其传入 `value` prop（复选框和单选按钮则传入 `checked`）。** React 会强制该输入始终具有你传入的 `value`。通常，你会通过声明一个 [状态变量：](/reference/react/useState)

```js {2,6,7}
function Form() {
  const [firstName, setFirstName] = useState(''); // 声明一个状态变量...
  // ...
  return (
    <input
      value={firstName} // ...强制输入框的值与状态变量一致...
      onChange={e => setFirstName(e.target.value)} // ...并在任何编辑时更新状态变量！
    />
  );
}
```

如果你本来就需要状态，那么受控输入就很有意义——例如，希望在每次编辑时重新渲染你的 UI：

```js {2,9}
function Form() {
  const [firstName, setFirstName] = useState('');
  return (
    <>
      <label>
        名字：
        <input value={firstName} onChange={e => setFirstName(e.target.value)} />
      </label>
      {firstName !== '' && <p>你的名字是 {firstName}。</p>}
      ...
```

如果你想提供多种方式来调整输入状态，这也很有用（例如，点击按钮）：

```js {3-4,10-11,14}
function Form() {
  // ...
  const [age, setAge] = useState('');
  const ageAsNumber = Number(age);
  return (
    <>
      <label>
        年龄：
        <input
          value={age}
          onChange={e => setAge(e.target.value)}
          type="number"
        />
        <button onClick={() => setAge(ageAsNumber + 10)}>
          增加 10 岁
        </button>
```

你传给受控组件的 `value` 不应是 `undefined` 或 `null`。如果你需要初始值为空（例如下面的 `firstName` 字段），请将状态变量初始化为空字符串（`''`）。

<Sandpack>

```js
import { useState } from 'react';

export default function Form() {
  const [firstName, setFirstName] = useState('');
  const [age, setAge] = useState('20');
  const ageAsNumber = Number(age);
  return (
    <>
      <label>
        名字：
        <input
          value={firstName}
          onChange={e => setFirstName(e.target.value)}
        />
      </label>
      <label>
        年龄：
        <input
          value={age}
          onChange={e => setAge(e.target.value)}
          type="number"
        />
        <button onClick={() => setAge(ageAsNumber + 10)}>
          增加 10 岁
        </button>
      </label>
      {firstName !== '' &&
        <p>你的名字是 {firstName}。</p>
      }
      {ageAsNumber > 0 &&
        <p>你的年龄是 {ageAsNumber}。</p>
      }
    </>
  );
}
```

```css
label { display: block; }
input { margin: 5px; }
p { font-weight: bold; }
```

</Sandpack>

<Pitfall>

**如果你传入 `value` 但没有传入 `onChange`，就无法在输入框中输入内容。** 当你通过传入某个 `value` 来控制输入时，你*强制*它始终保持你传入的值。因此，如果你把一个状态变量作为 `value` 传入，但忘记在 `onChange` 事件处理函数中同步更新该状态变量，React 就会在每次按键后把输入框恢复为你指定的 `value`。

</Pitfall>

---

### 优化每次按键时的重新渲染 {/*optimizing-re-rendering-on-every-keystroke*/}

当你使用受控输入时，你会在每次按键时设置状态。如果包含该状态的组件重新渲染了一个很大的树，这可能会变慢。你可以通过几种方式优化重新渲染性能。

例如，假设你有一个表单，它会在每次按键时重新渲染整个页面内容：

```js {5-8}
function App() {
  const [firstName, setFirstName] = useState('');
  return (
    <>
      <form>
        <input value={firstName} onChange={e => setFirstName(e.target.value)} />
      </form>
      <PageContent />
    </>
  );
}
```

由于 `<PageContent />` 不依赖输入状态，你可以把输入状态移动到它自己的组件中：

```js {4,10-17}
function App() {
  return (
    <>
      <SignupForm />
      <PageContent />
    </>
  );
}

function SignupForm() {
  const [firstName, setFirstName] = useState('');
  return (
    <form>
      <input value={firstName} onChange={e => setFirstName(e.target.value)} />
    </form>
  );
}
```

这会显著提升性能，因为现在每次按键时只有 `SignupForm` 会重新渲染。

如果无法避免重新渲染（例如，`PageContent` 依赖搜索输入的值），[`useDeferredValue`](/reference/react/useDeferredValue#deferring-re-rendering-for-a-part-of-the-ui) 可以让受控输入即使在大规模重新渲染中也保持响应迅速。

---

## 故障排除 {/*troubleshooting*/}

### 当我在文本输入框中输入时，它没有更新 {/*my-text-input-doesnt-update-when-i-type-into-it*/}

如果你渲染了一个带有 `value` 但没有 `onChange` 的输入框，你会在控制台看到一个错误：

```js
// 🔴 Bug: 没有 onChange 处理函数的受控文本输入框
<input value={something} />
```

<ConsoleBlock level="error">

你为一个表单字段提供了 `value` 属性，但没有提供 `onChange` 处理函数。这会将其渲染为只读字段。如果该字段应该是可变的，请使用 `defaultValue`。否则，请设置 `onChange` 或 `readOnly`。

</ConsoleBlock>

正如错误信息所示，如果你只是想[指定 *初始* 值，](#providing-an-initial-value-for-an-input)请改为传入 `defaultValue`：

```js
// ✅ Good: 带有初始值的非受控输入框
<input defaultValue={something} />
```

如果你想[使用状态变量来控制这个输入框，](#controlling-an-input-with-a-state-variable)请指定一个 `onChange` 处理函数：

```js
// ✅ Good: 带有 onChange 的受控输入框
<input value={something} onChange={e => setSomething(e.target.value)} />
```

如果该值有意设为只读，请添加 `readOnly` 属性以抑制该错误：

```js
// ✅ Good: 没有 on change 的只读受控输入框
<input value={something} readOnly={true} />
```

---

### 当我点击复选框时，它没有更新 {/*my-checkbox-doesnt-update-when-i-click-on-it*/}

如果你渲染了一个带有 `checked` 但没有 `onChange` 的复选框，你会在控制台看到一个错误：

```js
// 🔴 Bug: 没有 onChange 处理函数的受控复选框
<input type="checkbox" checked={something} />
```

<ConsoleBlock level="error">

你为一个表单字段提供了 `checked` 属性，但没有提供 `onChange` 处理函数。这会将其渲染为只读字段。如果该字段应该是可变的，请使用 `defaultChecked`。否则，请设置 `onChange` 或 `readOnly`。

</ConsoleBlock>

正如错误信息所示，如果你只是想[指定 *初始* 值，](#providing-an-initial-value-for-an-input)请改为传入 `defaultChecked`：

```js
// ✅ Good: 带有初始值的非受控复选框
<input type="checkbox" defaultChecked={something} />
```

如果你想[使用状态变量来控制这个复选框，](#controlling-an-input-with-a-state-variable)请指定一个 `onChange` 处理函数：

```js
// ✅ Good: 带有 onChange 的受控复选框
<input type="checkbox" checked={something} onChange={e => setSomething(e.target.checked)} />
```

<Pitfall>

对于复选框，你需要读取 `e.target.checked`，而不是 `e.target.value`。

</Pitfall>

如果复选框有意设为只读，请添加 `readOnly` 属性以抑制该错误：

```js
// ✅ Good: 没有 on change 的只读受控输入框
<input type="checkbox" checked={something} readOnly={true} />
```

---

### 我的输入框光标在每次按键时都会跳到开头 {/*my-input-caret-jumps-to-the-beginning-on-every-keystroke*/}

如果你[控制一个输入框，](#controlling-an-input-with-a-state-variable)你必须在 `onChange` 期间将其状态变量更新为来自 DOM 的输入值。

你不能把它更新为 `e.target.value` 之外的其他内容（复选框则是 `e.target.checked`）：

```js
function handleChange(e) {
  // 🔴 Bug: 将输入更新为 e.target.value 以外的值
  setFirstName(e.target.value.toUpperCase());
}
```

你也不能异步更新它：

```js
function handleChange(e) {
  // 🔴 Bug: 异步更新输入
  setTimeout(() => {
    setFirstName(e.target.value);
  }, 100);
}
```

要修复你的代码，请同步地将其更新为 `e.target.value`：

```js
function handleChange(e) {
  // ✅ 以同步方式将受控输入更新为 e.target.value
  setFirstName(e.target.value);
}
```

如果这不能解决问题，可能是输入框在每次按键时都会从 DOM 中移除并重新添加。这可能发生在你无意中在每次重新渲染时[重置状态](/learn/preserving-and-resetting-state)的情况下，例如输入框或其某个父组件总是接收不同的 `key` 属性，或者你嵌套了组件函数定义（这不受支持，并且会导致“内部”组件始终被视为不同的树）。

---

### 我遇到一个错误：“A component is changing an uncontrolled input to be controlled” {/*im-getting-an-error-a-component-is-changing-an-uncontrolled-input-to-be-controlled*/}


如果你向组件提供了 `value`，它在整个生命周期中必须始终是字符串。

你不能先传入 `value={undefined}`，然后再传入 `value="some string"`，因为 React 不会知道你想让这个组件是非受控还是受控。受控组件应该始终接收字符串类型的 `value`，而不是 `null` 或 `undefined`。

如果你的 `value` 来自 API 或状态变量，它可能会被初始化为 `null` 或 `undefined`。在这种情况下，要么最初将其设为空字符串（`''`），要么传入 `value={someValue ?? ''}` 以确保 `value` 是字符串。

同样地，如果你给复选框传入 `checked`，请确保它始终是布尔值。
