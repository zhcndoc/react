---
title: isValidElement
---

<Intro>

`isValidElement` 检查一个值是否为 React 元素。

```js
const isElement = isValidElement(value)
```

</Intro>

<InlineToc />

---

## 参考 {/*reference*/}

### `isValidElement(value)` {/*isvalidelement*/}

调用 `isValidElement(value)` 来检查 `value` 是否为 React 元素。

```js
import { isValidElement, createElement } from 'react';

// ✅ React 元素
console.log(isValidElement(<p />)); // true
console.log(isValidElement(createElement('p'))); // true

// ❌ 不是 React 元素
console.log(isValidElement(25)); // false
console.log(isValidElement('Hello')); // false
console.log(isValidElement({ age: 42 })); // false
```

[查看更多示例。](#usage)

#### 参数 {/*parameters*/}

* `value`：你想要检查的 `value`。它可以是任何类型的值。

#### 返回值 {/*returns*/}

如果 `value` 是 React 元素，`isValidElement` 返回 `true`。否则，它返回 `false`。

#### 注意事项 {/*caveats*/}

* **只有 [JSX 标签](/learn/writing-markup-with-jsx) 以及 [`createElement`](/reference/react/createElement) 返回的对象才会被认为是 React 元素。** 例如，尽管像 `42` 这样的数字是一个合法的 React *节点*（并且可以从组件中返回），但它不是合法的 React 元素。使用 [`createPortal`](/reference/react-dom/createPortal) 创建的数组和 portal 也*不*会被认为是 React 元素。

---

## 用法 {/*usage*/}

### 检查某些内容是否为 React 元素 {/*checking-if-something-is-a-react-element*/}

调用 `isValidElement` 来检查某个值是否是一个 *React 元素。*

React 元素包括：

- 通过编写 [JSX 标签](/learn/writing-markup-with-jsx) 产生的值
- 通过调用 [`createElement`](/reference/react/createElement) 产生的值

对于 React 元素，`isValidElement` 返回 `true`：

```js
import { isValidElement, createElement } from 'react';

// ✅ JSX 标签是 React 元素
console.log(isValidElement(<p />)); // true
console.log(isValidElement(<MyComponent />)); // true

// ✅ createElement 返回的值是 React 元素
console.log(isValidElement(createElement('p'))); // true
console.log(isValidElement(createElement(MyComponent))); // true
```

任何其他值，例如字符串、数字，或者任意对象和数组，都不是 React 元素。

对于它们，`isValidElement` 返回 `false`：

```js
// ❌ 这些 *不是* React 元素
console.log(isValidElement(null)); // false
console.log(isValidElement(25)); // false
console.log(isValidElement('Hello')); // false
console.log(isValidElement({ age: 42 })); // false
console.log(isValidElement([<div />, <div />])); // false
console.log(isValidElement(MyComponent)); // false
```

通常并不需要 `isValidElement`。它主要在你调用另一个 *只接受元素* 的 API（比如 [`cloneElement`](/reference/react/cloneElement)）时很有用，这样可以避免在参数不是 React 元素时出错。

除非你有非常特殊的理由添加 `isValidElement` 检查，否则你大概率不需要它。

<DeepDive>

#### React 元素 vs React 节点 {/*react-elements-vs-react-nodes*/}

当你编写一个组件时，你可以从中返回任何类型的 *React 节点*：

```js
function MyComponent() {
  // ... 你可以返回任何 React 节点 ...
}
```

React 节点可以是：

- 像 `<div />` 或 `createElement('div')` 这样创建的 React 元素
- 使用 [`createPortal`](/reference/react-dom/createPortal) 创建的 portal
- 字符串
- 数字
- `true`、`false`、`null` 或 `undefined`（这些不会显示出来）
- 其他 React 节点组成的数组

**注意，`isValidElement` 检查的是参数是否为 *React 元素*，而不是它是否为 React 节点。** 例如，`42` 不是合法的 React 元素。不过，它是完全合法的 React 节点：

```js
function MyComponent() {
  return 42; // 从组件中返回数字是没问题的
}
```

这就是为什么你不应该把 `isValidElement` 用作检查某个内容是否可以被渲染的方法。

</DeepDive>
