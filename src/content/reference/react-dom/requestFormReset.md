---
title: requestFormReset
---

<Intro>

`requestFormReset` 让你在 Action 或 Transition 完成后重置由 React 管理的表单。

```js
requestFormReset(form)
```

</Intro>

<InlineToc />

---

## 参考 {/*reference*/}

### `requestFormReset(form)` {/*requestformreset*/}

调用 `requestFormReset`，请求在当前 Action 或 Transition 完成后重置表单。

```js
import { startTransition } from 'react';
import { requestFormReset } from 'react-dom';

function handleSubmit(form) {
  startTransition(async () => {
    const formData = new FormData(form);
    requestFormReset(form);
    await save(formData);
  });
}
```

[查看更多示例。](#usage)

#### 参数 {/*parameters*/}

* `form`：由 React 渲染的 [HTML 表单元素](https://developer.mozilla.org/en-US/docs/Web/API/HTMLFormElement)。

#### 返回值 {/*returns*/}

`requestFormReset` 不返回任何内容。

#### 注意事项 {/*caveats*/}

* 传入由 React 渲染的表单元素。传入非表单元素，或传入不由 React 管理的表单，都会抛出错误。
* `requestFormReset` 用于 Actions 和 Transitions。如果在 Action 或 Transition 外调用，React 会在开发环境中发出警告，并同步重置表单。
* 如果你自行实现表单提交逻辑（例如使用 `onSubmit` 和 `startTransition`），但仍想采用 React 的表单重置行为，那么 `requestFormReset` 就很有用。
* 此 API 请求重置非受控表单字段。对于受控字段，请通过更新 state 来处理重置，例如在 `onReset` 中更新。

---

## 用法 {/*usage*/}

### 在自定义表单 Action 中请求重置 {/*requesting-a-reset-in-a-custom-form-action*/}

React 会在 `<form action={...}>` 成功提交后[自动重置非受控字段](/reference/react-dom/components/form#handle-form-submission-with-an-action-prop)。如果你手动实现提交逻辑，可以调用 `requestFormReset` 来选择采用相同的行为。

```js
import { startTransition } from 'react';
import { requestFormReset } from 'react-dom';

function SearchForm() {
  async function onSubmit(event) {
    event.preventDefault();

    const form = event.currentTarget;
    const formData = new FormData(form);

    startTransition(async () => {
      requestFormReset(form);
      await submitSearch(formData);
    });
  }

  return (
    <form onSubmit={onSubmit}>
      <input name="query" defaultValue="react" />
      <button type="submit">Search</button>
    </form>
  );
}
```

如果提交使用内置的 `<form action={...}>` 行为，则无需手动调用 `requestFormReset`。
