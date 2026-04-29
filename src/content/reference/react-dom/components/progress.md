---
title: "<progress>"
---

<Intro>

内置的浏览器 `<progress>` 组件[https://developer.mozilla.org/en-US/docs/Web/HTML/Element/progress](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/progress) 允许你渲染一个进度指示器。

```js
<progress value={0.5} />
```

</Intro>

<InlineToc />

---

## 参考 {/*reference*/}

### `<progress>` {/*progress*/}

要显示进度指示器，请渲染内置的浏览器 `<progress>` 组件[https://developer.mozilla.org/en-US/docs/Web/HTML/Element/progress](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/progress)。

```js
<progress value={0.5} />
```

[查看更多示例。](#usage)

#### 属性 {/*props*/}

`<progress>` 支持所有[通用元素属性。](/reference/react-dom/components/common#common-props)

此外，`<progress>` 还支持这些属性：

* [`max`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/progress#max)：一个数字。指定最大 `value`。默认为 `1`。
* [`value`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/progress#value)：介于 `0` 和 `max` 之间的数字，或在不确定进度时为 `null`。指定已完成的进度量。

---

## 用法 {/*usage*/}

### 控制进度指示器 {/*controlling-a-progress-indicator*/}

要显示进度指示器，请渲染一个 `<progress>` 组件。你可以传入一个介于 `0` 和你指定的 `max` 值之间的数字 `value`。如果你没有传入 `max` 值，默认会假定它为 `1`。

如果操作没有进行中，请传入 `value={null}`，将进度指示器置于不确定状态。

<Sandpack>

```js
export default function App() {
  return (
    <>
      <progress value={0} />
      <progress value={0.5} />
      <progress value={0.7} />
      <progress value={75} max={100} />
      <progress value={1} />
      <progress value={null} />
    </>
  );
}
```

```css
progress { display: block; }
```

</Sandpack>
