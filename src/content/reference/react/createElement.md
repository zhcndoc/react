---
title: createElement
---

<Intro>

`createElement` 可让你创建一个 React 元素。它可以作为编写 [JSX.](/learn/writing-markup-with-jsx) 的替代方案

```js
const element = createElement(type, props, ...children)
```

</Intro>

<InlineToc />

---

## 参考 {/*reference*/}

### `createElement(type, props, ...children)` {/*createelement*/}

调用 `createElement`，使用给定的 `type`、`props` 和 `children` 创建一个 React 元素。

```js
import { createElement } from 'react';

function Greeting({ name }) {
  return createElement(
    'h1',
    { className: 'greeting' },
    'Hello'
  );
}
```

[在下面查看更多示例。](#usage)

#### 参数 {/*parameters*/}

* `type`：`type` 参数必须是一个有效的 React 组件类型。例如，它可以是一个标签名字符串（如 `'div'` 或 `'span'`），也可以是一个 React 组件（函数、类，或者像 [`Fragment`](/reference/react/Fragment) 这样的特殊组件）。

* `props`：`props` 参数必须是一个对象或 `null`。如果传入 `null`，它会被视为与空对象相同。React 会创建一个元素，其 props 与你传入的 `props` 相匹配。请注意，`props` 对象中的 `ref` 和 `key` 是特殊的，它们不会作为返回的 `element` 上的 `element.props.ref` 和 `element.props.key` 提供。它们将作为 `element.ref` 和 `element.key` 提供。

* **可选** `...children`：零个或多个子节点。它们可以是任何 React 节点，包括 React 元素、字符串、数字、[portals](/reference/react-dom/createPortal)、空节点（`null`、`undefined`、`true` 和 `false`）以及 React 节点数组。

#### 返回值 {/*returns*/}

`createElement` 会返回一个具有以下几个属性的 React 元素对象：

* `type`：你传入的 `type`。
* `props`：你传入的 `props`，但不包括 `ref` 和 `key`。
* `ref`：你传入的 `ref`。如果缺失，则为 `null`。
* `key`：你传入的 `key`，会被强制转换为字符串。如果缺失，则为 `null`。

通常，你会从组件中返回该元素，或者将它作为另一个元素的子元素。虽然你可以读取元素的属性，但最好在元素创建后将其视为不透明的，并且只对它进行渲染。

#### 注意事项 {/*caveats*/}

* 你必须将 **React 元素及其 props 视为 [不可变](https://en.wikipedia.org/wiki/Immutable_object)**，并且在创建后永远不要修改它们的内容。在开发环境中，React 会浅层 [冻结](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/freeze) 返回的元素及其 `props` 属性，以强制执行这一点。

* 当你使用 JSX 时，**必须以大写字母开头书写标签，才能渲染你自己的自定义组件。** 换句话说，`<Something />` 等价于 `createElement(Something)`，而 `<something />`（小写）等价于 `createElement('something')`（注意这是一个字符串，因此它会被当作内置 HTML 标签处理）。

* 只有在子元素 **都是静态已知的情况下，才应将多个 children 作为多个参数传递给 `createElement`**，例如 `createElement('h1', {}, child1, child2, child3)`。如果你的子元素是动态的，请将整个数组作为第三个参数传入：`createElement('ul', {}, listItems)`。这样可以确保 React 对任何动态列表中缺少 `key` 的情况发出 [警告](/learn/rendering-lists#keeping-list-items-in-order-with-key)。对于静态列表来说，这不是必需的，因为它们不会重新排序。

---

## 用法 {/*usage*/}

### 不使用 JSX 创建元素 {/*creating-an-element-without-jsx*/}

如果你不喜欢 [JSX](/learn/writing-markup-with-jsx)，或者在项目中不能使用它，你可以使用 `createElement` 作为替代方案。

要在不使用 JSX 的情况下创建元素，请使用某些 <CodeStep step={1}>类型</CodeStep>、<CodeStep step={2}>props</CodeStep> 和 <CodeStep step={3}>children</CodeStep> 调用 `createElement`：

```js [[1, 5, "'h1'"], [2, 6, "{ className: 'greeting' }"], [3, 7, "'Hello ',"], [3, 8, "createElement('i', null, name),"], [3, 9, "'. Welcome!'"]]
import { createElement } from 'react';

function Greeting({ name }) {
  return createElement(
    'h1',
    { className: 'greeting' },
    'Hello ',
    createElement('i', null, name),
    '. Welcome!'
  );
}
```

<CodeStep step={3}>children</CodeStep> 是可选的，你可以按需传入任意多个（上面的示例有三个 children）。这段代码会显示一个带有问候语的 `<h1>` 标题。作为对比，下面是使用 JSX 重写的同一个示例：

```js [[1, 3, "h1"], [2, 3, "className=\\"greeting\\""], [3, 4, "Hello <i>{name}</i>. Welcome!"], [1, 5, "h1"]]
function Greeting({ name }) {
  return (
    <h1 className="greeting">
      Hello <i>{name}</i>. Welcome!
    </h1>
  );
}
```

要渲染你自己的 React 组件，请将像 `Greeting` 这样的函数作为 <CodeStep step={1}>类型</CodeStep> 传入，而不是像 `'h1'` 这样的字符串：

```js [[1, 2, "Greeting"], [2, 2, "{ name: 'Taylor' }"]]
export default function App() {
  return createElement(Greeting, { name: 'Taylor' });
}
```

使用 JSX 时，它会像这样：

```js [[1, 2, "Greeting"], [2, 2, "name=\\"Taylor\\""]]
export default function App() {
  return <Greeting name="Taylor" />;
}
```

下面是一个使用 `createElement` 编写的完整示例：

<Sandpack>

```js
import { createElement } from 'react';

function Greeting({ name }) {
  return createElement(
    'h1',
    { className: 'greeting' },
    'Hello ',
    createElement('i', null, name),
    '. Welcome!'
  );
}

export default function App() {
  return createElement(
    Greeting,
    { name: 'Taylor' }
  );
}
```

```css
.greeting {
  color: darkgreen;
  font-family: Georgia;
}
```

</Sandpack>

下面是使用 JSX 编写的同一个示例：

<Sandpack>

```js
function Greeting({ name }) {
  return (
    <h1 className="greeting">
      Hello <i>{name}</i>. Welcome!
    </h1>
  );
}

export default function App() {
  return <Greeting name="Taylor" />;
}
```

```css
.greeting {
  color: darkgreen;
  font-family: Georgia;
}
```

</Sandpack>

这两种编码风格都可以，因此你可以在项目中使用自己更喜欢的那一种。与 `createElement` 相比，使用 JSX 的主要好处是很容易看出哪个结束标签对应哪个开始标签。

<DeepDive>

#### React 元素到底是什么？ {/*what-is-a-react-element-exactly*/}

元素是用户界面某一部分的轻量级描述。例如，`<Greeting name="Taylor" />` 和 `createElement(Greeting, { name: 'Taylor' })` 都会生成一个如下对象：

```js
// 略微简化
{
  type: Greeting,
  props: {
    name: 'Taylor'
  },
  key: null,
  ref: null,
}
```

**请注意，创建这个对象并不会渲染 `Greeting` 组件，也不会创建任何 DOM 元素。**

React 元素更像是一种描述——一种供 React 以后渲染 `Greeting` 组件的指令。通过从你的 `App` 组件返回这个对象，你告诉 React 接下来要做什么。

创建元素的开销极低，因此你无需尝试对其进行优化或避免创建它。

</DeepDive>
