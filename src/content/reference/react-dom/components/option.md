---
title: "<option>"
---

<Intro>

[内置浏览器 `<option>` 组件](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/option) 让你可以在 [`<select>`](/reference/react-dom/components/select) 框内渲染一个选项。

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

### `<option>` {/*option*/}

[内置浏览器 `<option>` 组件](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/option) 让你可以在 [`<select>`](/reference/react-dom/components/select) 框内渲染一个选项。

```js
<select>
  <option value="someOption">某个选项</option>
  <option value="otherOption">另一个选项</option>
</select>
```

[在下面查看更多示例。](#usage)

#### 属性 {/*props*/}

`<option>` 支持所有[通用元素属性。](/reference/react-dom/components/common#common-props)

此外，`<option>` 还支持以下属性：

* [`disabled`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/option#disabled)：布尔值。如果为 `true`，该选项将不可选择，并且会显示为灰暗状态。
* [`label`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/option#label)：字符串。指定该选项的含义。如果未指定，则使用选项内部的文本。
* [`value`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/option#value)：当选中此选项时，[在表单中提交父级 `<select>` 时](/reference/react-dom/components/select#reading-the-select-box-value-when-submitting-a-form)要使用的值。

#### 注意事项 {/*caveats*/}

* React 不支持 `<option>` 上的 `selected` 属性。请改为将此选项的 `value` 传递给父级 [`<select defaultValue>`](/reference/react-dom/components/select#providing-an-initially-selected-option)，用于非受控选择框；或者传递给 [`<select value>`](/reference/react-dom/components/select#controlling-a-select-box-with-a-state-variable)，用于受控选择框。

---

## 用法 {/*usage*/}

### 使用选项显示一个选择框 {/*displaying-a-select-box-with-options*/}

渲染一个 `<select>`，其中包含一组 `<option>` 组件，以显示选择框。为每个 `<option>` 提供一个 `value`，表示将随表单提交的数据。

[阅读更多关于使用一组 `<option>` 组件显示 `<select>` 的内容。](/reference/react-dom/components/select)

<Sandpack>

```js
export default function FruitPicker() {
  return (
    <label>
      Pick a fruit:
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

