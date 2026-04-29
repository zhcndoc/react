---
title: createRef
---

<Pitfall>

`createRef` 主要用于 [类组件。](/reference/react/Component) 函数组件通常改用 [`useRef`](/reference/react/useRef)。

</Pitfall>

<Intro>

`createRef` 会创建一个可包含任意值的 [ref](/learn/referencing-values-with-refs) 对象。

```js
class MyInput extends Component {
  inputRef = createRef();
  // ...
}
```

</Intro>

<InlineToc />

---

## 参考 {/*reference*/}

### `createRef()` {/*createref*/}

调用 `createRef`，在 [类组件。](/reference/react/Component) 中声明一个 [ref](/learn/referencing-values-with-refs)

```js
import { createRef, Component } from 'react';

class MyComponent extends Component {
  intervalRef = createRef();
  inputRef = createRef();
  // ...
```

[查看更多示例。](#usage)

#### 参数 {/*parameters*/}

`createRef` 不接受任何参数。

#### 返回值 {/*returns*/}

`createRef` 返回一个对象，该对象只有一个属性：

* `current`：初始值为 `null`。你之后可以把它设置为其他值。如果你将 ref 对象作为 `ref` 属性传递给 JSX 节点，React 会设置它的 `current` 属性。

#### 注意事项 {/*caveats*/}

* `createRef` 每次都会返回一个*不同的*对象。它等价于你自己写 `{ current: null }`。
* 在函数组件中，你大概率应该改用 [`useRef`](/reference/react/useRef)，因为它总是返回同一个对象。
* `const ref = useRef()` 等价于 `const [ref, _] = useState(() => createRef(null))`。

---

## 用法 {/*usage*/}

### 在类组件中声明 ref {/*declaring-a-ref-in-a-class-component*/}

要在 [类组件](/reference/react/Component) 内声明一个 ref，调用 `createRef` 并将其结果赋值给类字段：

```js {4}
import { Component, createRef } from 'react';

class Form extends Component {
  inputRef = createRef();

  // ...
}
```

如果你现在在 JSX 中把 `ref={this.inputRef}` 传给一个 `<input>`，React 会将输入框的 DOM 节点填充到 `this.inputRef.current` 中。例如，下面演示了如何创建一个聚焦输入框的按钮：

<Sandpack>

```js
import { Component, createRef } from 'react';

export default class Form extends Component {
  inputRef = createRef();

  handleClick = () => {
    this.inputRef.current.focus();
  }

  render() {
    return (
      <>
        <input ref={this.inputRef} />
        <button onClick={this.handleClick}>
          聚焦输入框
        </button>
      </>
    );
  }
}
```

</Sandpack>

<Pitfall>

`createRef` 主要用于 [类组件。](/reference/react/Component) 函数组件通常改用 [`useRef`](/reference/react/useRef)。

</Pitfall>

---

## 替代方案 {/*alternatives*/}

### 将使用 `createRef` 的类迁移为使用 `useRef` 的函数组件 {/*migrating-from-a-class-with-createref-to-a-function-with-useref*/}

我们建议在新代码中使用函数组件，而不是 [类组件](/reference/react/Component)。如果你已有一些使用 `createRef` 的类组件，下面介绍如何转换它们。这是原始代码：

<Sandpack>

```js
import { Component, createRef } from 'react';

export default class Form extends Component {
  inputRef = createRef();

  handleClick = () => {
    this.inputRef.current.focus();
  }

  render() {
    return (
      <>
        <input ref={this.inputRef} />
        <button onClick={this.handleClick}>
          聚焦输入框
        </button>
      </>
    );
  }
}
```

</Sandpack>

当你[将这个组件从类转换为函数时，](/reference/react/Component#alternatives)把对 `createRef` 的调用替换为对 [`useRef` 的调用：](/reference/react/useRef)

<Sandpack>

```js
import { useRef } from 'react';

export default function Form() {
  const inputRef = useRef(null);

  function handleClick() {
    inputRef.current.focus();
  }

  return (
    <>
      <input ref={inputRef} />
      <button onClick={handleClick}>
        聚焦输入框
      </button>
    </>
  );
}
```

</Sandpack>
