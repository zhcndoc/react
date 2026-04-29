---
title: "<select>"
---

<Intro>

[内置浏览器的 `<select>` 组件](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/select) 让你渲染一个带选项的选择框。

```js
<select>
  <option value="someOption">某个选项</option>
  <option value="otherOption">另一个选项</option>
</select>
```

</Intro>

<InlineToc />

---

## 参考 {/*reference*/}

### `<select>` {/*select*/}

要显示一个选择框，请渲染 [内置浏览器的 `<select>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/select) 组件。

```js
<select>
  <option value="someOption">某个选项</option>
  <option value="otherOption">另一个选项</option>
</select>
```

[查看更多示例。](#usage)

#### Props {/*props*/}

`<select>` 支持所有 [通用元素 props。](/reference/react-dom/components/common#common-props)

你可以通过传递 `value` prop 来[将选择框设为受控](#controlling-a-select-box-with-a-state-variable)：

* `value`：一个字符串（对于 [`multiple={true}`](#enabling-multiple-selection) 则为字符串数组）。控制选中了哪个选项。每个值字符串都应匹配 `<select>` 内部嵌套的某个 `<option>` 的 `value`。

当你传递 `value` 时，也必须传递一个会更新所传值的 `onChange` 处理函数。

如果你的 `<select>` 是非受控的，你可以改为传递 `defaultValue` prop：

* `defaultValue`：一个字符串（对于 [`multiple={true}`](#enabling-multiple-selection) 则为字符串数组）。指定[初始选中的选项。](#providing-an-initially-selected-option)

这些 `<select>` props 同时适用于非受控和受控选择框：

* [`autoComplete`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/select#autocomplete)：一个字符串。指定某种可能的[自动完成行为。](https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes/autocomplete#values)
* [`autoFocus`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/select#autofocus)：一个布尔值。如果为 `true`，React 会在挂载时聚焦该元素。
* `children`：`<select>` 接受 [`<option>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/option)、[`<optgroup>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/optgroup) 和 [`<datalist>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/datalist) 组件作为子元素。只要你的自定义组件最终渲染出允许的组件，也可以传入它们。如果你传入的自定义组件最终渲染 `<option>` 标签，那么你渲染的每个 `<option>` 都必须有一个 `value`。
* [`disabled`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/select#disabled)：一个布尔值。如果为 `true`，选择框将不可交互并会显示为灰暗。
* [`form`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/select#form)：一个字符串。指定该选择框所属的 `<form>` 的 `id`。如果省略，则为最近的父级 form。
* [`multiple`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/select#multiple)：一个布尔值。如果为 `true`，浏览器允许[多选。](#enabling-multiple-selection)
* [`name`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/select#name)：一个字符串。指定这个选择框的名称，它会[随表单一起提交。](#reading-the-select-box-value-when-submitting-a-form)
* `onChange`：一个 [`Event` 处理函数](/reference/react-dom/components/common#event-handler)。对于[受控选择框](#controlling-a-select-box-with-a-state-variable)是必需的。用户选择不同选项时会立即触发。行为类似于浏览器的 [`input` 事件。](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/input_event)
* `onChangeCapture`：在[捕获阶段](/learn/responding-to-events#capture-phase-events)触发的 `onChange` 版本。
* [`onInput`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/input_event)：一个 [`Event` 处理函数](/reference/react-dom/components/common#event-handler)。当用户更改值时会立即触发。出于历史原因，在 React 中通常使用同样能起类似作用的 `onChange` 来代替。
* `onInputCapture`：在[捕获阶段](/learn/responding-to-events#capture-phase-events)触发的 `onInput` 版本。
* [`onInvalid`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLInputElement/invalid_event)：一个 [`Event` 处理函数](/reference/react-dom/components/common#event-handler)。如果输入在表单提交时未通过校验，则触发。不同于内置的 `invalid` 事件，React 的 `onInvalid` 事件会冒泡。
* `onInvalidCapture`：在[捕获阶段](/learn/responding-to-events#capture-phase-events)触发的 `onInvalid` 版本。
* [`required`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/select#required)：一个布尔值。如果为 `true`，则表单提交时必须提供该值。
* [`size`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/select#size)：一个数字。对于 `multiple={true}` 的选择框，指定初始可见项目的期望数量。

#### 注意事项 {/*caveats*/}

- 与 HTML 不同，向 `<option>` 传递 `selected` 属性是不支持的。请改用 [`<select defaultValue>`](#providing-an-initially-selected-option) 处理非受控选择框，以及用 [`<select value>`](#controlling-a-select-box-with-a-state-variable) 处理受控选择框。
- 如果选择框接收到 `value` prop，它将被[视为受控。](#controlling-a-select-box-with-a-state-variable)
- 选择框不能同时既是受控又是非受控的。
- 选择框在其生命周期内不能在受控和非受控之间切换。
- 每个受控选择框都需要一个 `onChange` 事件处理函数，用于同步更新其底层值。

---

## 用法 {/*usage*/}

### 使用选项显示一个选择框 {/*displaying-a-select-box-with-options*/}

渲染一个内部包含 `<option>` 组件列表的 `<select>`，即可显示一个选择框。给每个 `<option>` 一个 `value`，表示要随表单提交的数据。

<Sandpack>

```js
export default function FruitPicker() {
  return (
    <label>
      选择一种水果：
      <select name="selectedFruit">
        <option value="apple">苹果</option>
        <option value="banana">香蕉</option>
        <option value="orange">橙子</option>
      </select>
    </label>
  );
}
```

```css
select { margin: 5px; }
```

</Sandpack>

---

### 为选择框提供标签 {/*providing-a-label-for-a-select-box*/}

通常，你会把每个 `<select>` 都放在一个 [`<label>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/label) 标签内。这会告诉浏览器该标签与这个选择框相关联。当用户点击标签时，浏览器会自动聚焦到选择框上。这对于可访问性也很重要：当用户聚焦选择框时，屏幕阅读器会朗读标签说明。

如果你不能把 `<select>` 嵌套进 `<label>`，可以通过给 `<select id>` 和 [`<label htmlFor>`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLLabelElement/htmlFor) 传入相同的 ID 来关联它们。为了避免同一个组件的多个实例之间发生冲突，可以用 [`useId`](/reference/react/useId) 生成这样的 ID。

<Sandpack>

```js
import { useId } from 'react';

export default function Form() {
  const vegetableSelectId = useId();
  return (
    <>
      <label>
        选择一种水果：
        <select name="selectedFruit">
          <option value="apple">苹果</option>
          <option value="banana">香蕉</option>
          <option value="orange">橙子</option>
        </select>
      </label>
      <hr />
      <label htmlFor={vegetableSelectId}>
        选择一种蔬菜：
      </label>
      <select id={vegetableSelectId} name="selectedVegetable">
        <option value="cucumber">黄瓜</option>
        <option value="corn">玉米</option>
        <option value="tomato">番茄</option>
      </select>
    </>
  );
}
```

```css
select { margin: 5px; }
```

</Sandpack>


---

### 提供一个初始选中的选项 {/*providing-an-initially-selected-option*/}

默认情况下，浏览器会选中列表中的第一个 `<option>`。如果想默认选中其他选项，请把该 `<option>` 的 `value` 作为 `defaultValue` 传给 `<select>` 元素。

<Sandpack>

```js
export default function FruitPicker() {
  return (
    <label>
      选择一种水果：
      <select name="selectedFruit" defaultValue="orange">
        <option value="apple">苹果</option>
        <option value="banana">香蕉</option>
        <option value="orange">橙子</option>
      </select>
    </label>
  );
}
```

```css
select { margin: 5px; }
```

</Sandpack>

<Pitfall>

与 HTML 不同，向单独的 `<option>` 传递 `selected` 属性是不支持的。

</Pitfall>

---

### 启用多选 {/*enabling-multiple-selection*/}

向 `<select>` 传递 `multiple={true}` 以允许用户选择多个选项。在这种情况下，如果你还指定了 `defaultValue` 来选择初始选中的选项，它必须是一个数组。

<Sandpack>

```js
export default function FruitPicker() {
  return (
    <label>
      选择一些水果：
      <select
        name="selectedFruit"
        defaultValue={['orange', 'banana']}
        multiple={true}
      >
        <option value="apple">苹果</option>
        <option value="banana">香蕉</option>
        <option value="orange">橙子</option>
      </select>
    </label>
  );
}
```

```css
select { display: block; margin-top: 10px; width: 200px; }
```

</Sandpack>

---

### 提交表单时读取选择框的值 {/*reading-the-select-box-value-when-submitting-a-form*/}

在选择框外包裹一个 [`<form>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/form)，并在其中放一个 [`<button type="submit">`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/button)。它会调用你的 `<form onSubmit>` 事件处理函数。默认情况下，浏览器会把表单数据发送到当前 URL 并刷新页面。你可以通过调用 `e.preventDefault()` 来覆盖这种行为。使用 [`new FormData(e.target)`](https://developer.mozilla.org/en-US/docs/Web/API/FormData) 读取表单数据。
<Sandpack>

```js
export default function EditPost() {
  function handleSubmit(e) {
    // 阻止浏览器重新加载页面
    e.preventDefault();
    // 读取表单数据
    const form = e.target;
    const formData = new FormData(form);
    // 你可以直接将 formData 作为 fetch 的 body：
    fetch('/some-api', { method: form.method, body: formData });
    // 你也可以像浏览器默认行为那样，从中生成一个 URL：
    console.log(new URLSearchParams(formData).toString());
    // 你也可以把它当作普通对象来使用。
    const formJson = Object.fromEntries(formData.entries());
    console.log(formJson); // (!) 这里不包括多选值
    // 或者你可以获取一个 name-value 对数组。
    console.log([...formData.entries()]);
  }

  return (
    <form method="post" onSubmit={handleSubmit}>
      <label>
        选择你最喜欢的水果：
        <select name="selectedFruit" defaultValue="orange">
          <option value="apple">苹果</option>
          <option value="banana">香蕉</option>
          <option value="orange">橙子</option>
        </select>
      </label>
      <label>
        选择你最喜欢的所有蔬菜：
        <select
          name="selectedVegetables"
          multiple={true}
          defaultValue={['corn', 'tomato']}
        >
          <option value="cucumber">黄瓜</option>
          <option value="corn">玉米</option>
          <option value="tomato">番茄</option>
        </select>
      </label>
      <hr />
      <button type="reset">重置</button>
      <button type="submit">提交</button>
    </form>
  );
}
```

```css
label, select { display: block; }
label { margin-bottom: 20px; }
```

</Sandpack>

<Note>

给你的 `<select>` 一个 `name`，例如 `<select name="selectedFruit" />`。你指定的 `name` 会作为表单数据中的键，例如 `{ selectedFruit: "orange" }`。

如果你使用 `<select multiple={true}>`，那么你从表单中读取的 [`FormData`](https://developer.mozilla.org/en-US/docs/Web/API/FormData) 会把每个被选中的值作为独立的 name-value 对包含进去。请仔细查看上面示例中的控制台日志。

</Note>

<Pitfall>

默认情况下，表单内*任何* `<button>` 都会提交表单。这可能会让人意外！如果你有自己的自定义 `Button` React 组件，建议返回 [`<button type="button">`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input/button) 而不是 `<button>`。然后，为了明确起见，把*确实*应该提交表单的按钮写成 `<button type="submit">`。

</Pitfall>

---

### 使用状态变量控制选择框 {/*controlling-a-select-box-with-a-state-variable*/}

像 `<select />` 这样的选择框是*非受控的*。即使你[传入一个初始选中值](#providing-an-initially-selected-option)，比如 `<select defaultValue="orange" />`，你的 JSX 也只指定了初始值，而不是当前值。

**要渲染一个_受控_选择框，请向它传递 `value` prop。** React 会强制让选择框始终具有你传入的 `value`。通常，你会通过声明一个[状态变量：](/reference/react/useState)来控制选择框：

```js {2,6,7}
function FruitPicker() {
  const [selectedFruit, setSelectedFruit] = useState('orange'); // 声明一个状态变量...
  // ...
  return (
    <select
      value={selectedFruit} // ...强制让选择框的值与状态变量匹配...
      onChange={e => setSelectedFruit(e.target.value)} // ...并在任何变化时更新状态变量！
    >
      <option value="apple">苹果</option>
      <option value="banana">香蕉</option>
      <option value="orange">橙子</option>
    </select>
  );
}
```

如果你想针对每次选择都重新渲染 UI 的某一部分，这就很有用。

<Sandpack>

```js
import { useState } from 'react';

export default function FruitPicker() {
  const [selectedFruit, setSelectedFruit] = useState('orange');
  const [selectedVegs, setSelectedVegs] = useState(['corn', 'tomato']);
  return (
    <>
      <label>
        选择一种水果：
        <select
          value={selectedFruit}
          onChange={e => setSelectedFruit(e.target.value)}
        >
          <option value="apple">苹果</option>
          <option value="banana">香蕉</option>
          <option value="orange">橙子</option>
        </select>
      </label>
      <hr />
      <label>
        选择你最喜欢的所有蔬菜：
        <select
          multiple={true}
          value={selectedVegs}
          onChange={e => {
            const options = [...e.target.selectedOptions];
            const values = options.map(option => option.value);
            setSelectedVegs(values);
          }}
        >
          <option value="cucumber">黄瓜</option>
          <option value="corn">玉米</option>
          <option value="tomato">番茄</option>
        </select>
      </label>
      <hr />
      <p>你最喜欢的水果：{selectedFruit}</p>
      <p>你最喜欢的蔬菜：{selectedVegs.join(', ')}</p>
    </>
  );
}
```

```css
select { margin-bottom: 10px; display: block; }
```

</Sandpack>

<Pitfall>

**如果你传递 `value` 却没有传递 `onChange`，就无法选择任何选项。** 当你通过传入某个 `value` 来控制选择框时，你*强制*它始终保持你传入的值。所以如果你把一个状态变量作为 `value` 传入，但忘记在 `onChange` 事件处理函数中同步更新这个状态变量，React 会在每次按键后把选择框恢复成你指定的 `value`。

与 HTML 不同，向单独的 `<option>` 传递 `selected` 属性是不支持的。

</Pitfall>
