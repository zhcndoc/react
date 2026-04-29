---
title: 快速开始
---

<Intro>

欢迎来到 React 文档！本页将向你介绍你在日常工作中会用到的 80% 的 React 概念。

</Intro>

<YouWillLearn>

- 如何创建和嵌套组件
- 如何添加标记和样式
- 如何显示数据
- 如何渲染条件和列表
- 如何响应事件并更新屏幕
- 如何在组件之间共享数据

</YouWillLearn>

## 创建和嵌套组件 {/*components*/}

React 应用由 *组件* 构成。组件是 UI（用户界面）的一部分，拥有自己的逻辑和外观。组件可以小到一个按钮，也可以大到整个页面。

React 组件是返回标记的 JavaScript 函数：

```js
function MyButton() {
  return (
    <button>我是一个按钮</button>
  );
}
```

现在你已经声明了 `MyButton`，就可以把它嵌套到另一个组件中：

```js {5}
export default function MyApp() {
  return (
    <div>
      <h1>欢迎来到我的应用</h1>
      <MyButton />
    </div>
  );
}
```

注意 `<MyButton />` 以大写字母开头。这就是你知道它是 React 组件的方式。React 组件名称必须始终以大写字母开头，而 HTML 标签必须是小写字母。

看看结果：

<Sandpack>

```js
function MyButton() {
  return (
    <button>
      我是一个按钮
    </button>
  );
}

export default function MyApp() {
  return (
    <div>
      <h1>欢迎来到我的应用</h1>
      <MyButton />
    </div>
  );
}
```

</Sandpack>

`export default` 关键字指定文件中的主组件。如果你对某些 JavaScript 语法不熟悉，[MDN](https://developer.mozilla.org/en-US/docs/web/javascript/reference/statements/export) 和 [javascript.info](https://javascript.info/import-export) 提供了很好的参考资料。

## 使用 JSX 编写标记 {/*writing-markup-with-jsx*/}

上面你看到的标记语法称为 *JSX*。它是可选的，但大多数 React 项目为了方便都会使用 JSX。我们推荐用于本地开发的所有[工具](/learn/installation)都原生支持 JSX。

JSX 比 HTML 更严格。你必须关闭像 `<br />` 这样的标签。你的组件也不能返回多个 JSX 标签。你必须把它们包裹在一个共享的父元素中，比如 `<div>...</div>` 或一个空的 `<>...</>` 包裹器：

```js {3,6}
function AboutPage() {
  return (
    <>
      <h1>关于</h1>
      <p>你好。<br />你好吗？</p>
    </>
  );
}
```

如果你有很多 HTML 需要转换成 JSX，可以使用[在线转换器。](https://transform.tools/html-to-jsx)

## 添加样式 {/*adding-styles*/}

在 React 中，你使用 `className` 指定 CSS 类。它的工作方式与 HTML 的 [`class`](https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/class) 属性相同：

```js
<img className="avatar" />
```

然后你在单独的 CSS 文件中为它编写 CSS 规则：

```css
/* 在你的 CSS 中 */
.avatar {
  border-radius: 50%;
}
```

React 并不规定你如何添加 CSS 文件。在最简单的情况下，你需要在 HTML 中添加一个 [`<link>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/link) 标签。如果你使用构建工具或框架，请查阅其文档，了解如何将 CSS 文件添加到你的项目中。

## 显示数据 {/*displaying-data*/}

JSX 允许你把标记写进 JavaScript 中。花括号让你“返回到” JavaScript，这样你就可以嵌入代码中的某个变量并将其显示给用户。例如，下面会显示 `user.name`：

```js {3}
return (
  <h1>
    {user.name}
  </h1>
);
```

你也可以从 JSX 属性中“进入 JavaScript”，但必须使用花括号*而不是*引号。例如，`className="avatar"` 会把 `"avatar"` 字符串作为 CSS 类传递，而 `src={user.imageUrl}` 会读取 JavaScript 的 `user.imageUrl` 变量值，然后把该值作为 `src` 属性传递：

```js {3,4}
return (
  <img
    className="avatar"
    src={user.imageUrl}
  />
);
```

你也可以在 JSX 花括号中放入更复杂的表达式，例如，[字符串拼接](https://javascript.info/operators#string-concatenation-with-binary)：

<Sandpack>

```js
const user = {
  name: 'Hedy Lamarr',
  imageUrl: 'https://react.dev/images/docs/scientists/yXOvdOSs.jpg',
  imageSize: 90,
};

export default function Profile() {
  return (
    <>
      <h1>{user.name}</h1>
      <img
        className="avatar"
        src={user.imageUrl}
        alt={'Hedy Lamarr 的照片'}
        style={{
          width: user.imageSize,
          height: user.imageSize
        }}
      />
    </>
  );
}
```

```css
.avatar {
  border-radius: 50%;
}

.large {
  border: 4px solid gold;
}
```

</Sandpack>

在上面的示例中，`style={{}}` 并不是特殊语法，而是在 `style={ }` JSX 花括号内部的普通 `{}` 对象。当你的样式依赖 JavaScript 变量时，你可以使用 `style` 属性。

## 条件渲染 {/*conditional-rendering*/}

在 React 中，编写条件语句没有特殊语法。相反，你会使用与编写普通 JavaScript 代码时相同的技巧。例如，你可以使用 [`if`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/if...else) 语句来有条件地包含 JSX：

```js
let content;
if (isLoggedIn) {
  content = <AdminPanel />;
} else {
  content = <LoginForm />;
}
return (
  <div>
    {content}
  </div>
);
```

如果你更喜欢更紧凑的代码，可以使用[条件 `?` 运算符。](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Conditional_Operator) 与 `if` 不同，它可以在 JSX 内部工作：

```js
<div>
  {isLoggedIn ? (
    <AdminPanel />
  ) : (
    <LoginForm />
  )}
</div>
```

当你不需要 `else` 分支时，也可以使用更短的[逻辑 `&&` 语法](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Logical_AND#short-circuit_evaluation)：

```js
<div>
  {isLoggedIn && <AdminPanel />}
</div>
```

以上所有方法也适用于有条件地指定属性。如果你对其中某些 JavaScript 语法不熟悉，可以先始终使用 `if...else`。

## 渲染列表 {/*rendering-lists*/}

你将依赖 JavaScript 的一些特性，例如 [`for` 循环](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for) 和 [数组的 `map()` 函数](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map)，来渲染组件列表。

例如，假设你有一个产品数组：

```js
const products = [
  { title: 'Cabbage', id: 1 },
  { title: 'Garlic', id: 2 },
  { title: 'Apple', id: 3 },
];
```

在组件内部，使用 `map()` 函数将产品数组转换为 `<li>` 项数组：

```js
const listItems = products.map(product =>
  <li key={product.id}>
    {product.title}
  </li>
);

return (
  <ul>{listItems}</ul>
);
```

注意 `<li>` 具有一个 `key` 属性。对于列表中的每个项目，你都应该传递一个字符串或数字，用于在其兄弟项中唯一标识该项目。通常，key 应该来自你的数据，例如数据库 ID。React 使用你的 key 来判断如果你之后插入、删除或重新排序项目时发生了什么。

<Sandpack>

```js
const products = [
  { title: 'Cabbage', isFruit: false, id: 1 },
  { title: 'Garlic', isFruit: false, id: 2 },
  { title: 'Apple', isFruit: true, id: 3 },
];

export default function ShoppingList() {
  const listItems = products.map(product =>
    <li
      key={product.id}
      style={{
        color: product.isFruit ? 'magenta' : 'darkgreen'
      }}
    >
      {product.title}
    </li>
  );

  return (
    <ul>{listItems}</ul>
  );
}
```

</Sandpack>

## 响应事件 {/*responding-to-events*/}

你可以通过在组件内部声明 *事件处理函数* 来响应事件：

```js {2-4,7}
function MyButton() {
  function handleClick() {
    alert('你点击了我！');
  }

  return (
    <button onClick={handleClick}>
      点击我
    </button>
  );
}
```

注意 `onClick={handleClick}` 末尾没有括号！不要 _调用_ 事件处理函数：你只需要*传递*它。React 会在用户点击按钮时调用你的事件处理函数。

## 更新屏幕 {/*updating-the-screen*/}

通常，你会希望组件“记住”一些信息并将其显示出来。例如，也许你想统计一个按钮被点击的次数。为此，请为你的组件添加 *state*。

首先，从 React 导入 [`useState`](/reference/react/useState)：

```js
import { useState } from 'react';
```

现在你可以在组件内部声明一个 *state 变量*：

```js
function MyButton() {
  const [count, setCount] = useState(0);
  // ...
```

你会从 `useState` 获得两样东西：当前状态（`count`），以及允许你更新它的函数（`setCount`）。你可以为它们取任何名字，但惯例是写成 `[something, setSomething]`。

按钮第一次显示时，`count` 将是 `0`，因为你向 `useState()` 传入了 `0`。当你想更改状态时，调用 `setCount()` 并将新值传给它。点击这个按钮会使计数器递增：

```js {5}
function MyButton() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1);
  }

  return (
    <button onClick={handleClick}>
      点击了 {count} 次
    </button>
  );
}
```

React 会再次调用你的组件函数。这一次，`count` 将是 `1`。然后它会变成 `2`。以此类推。

如果你多次渲染同一个组件，每个组件都会拥有自己的 state。分别点击每个按钮：

<Sandpack>

```js
import { useState } from 'react';

export default function MyApp() {
  return (
    <div>
      <h1>分别更新的计数器</h1>
      <MyButton />
      <MyButton />
    </div>
  );
}

function MyButton() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1);
  }

  return (
    <button onClick={handleClick}>
      点击了 {count} 次
    </button>
  );
}
```

```css
button {
  display: block;
  margin-bottom: 5px;
}
```

</Sandpack>

注意每个按钮如何“记住”自己的 `count` 状态，并且不会影响其他按钮。

## 使用 Hooks {/*using-hooks*/}

以 `use` 开头的函数被称为 *Hooks*。`useState` 是 React 提供的内置 Hook。你可以在 [API reference.](/reference/react) 中找到其他内置 Hooks。你也可以通过组合现有的 Hooks 来编写你自己的 Hooks。

Hooks 比其他函数有更多限制。你只能在组件（或其他 Hooks）的 *顶层* 调用 Hooks。如果你想在条件语句或循环中使用 `useState`，请提取一个新组件并把它放在那里。

## 组件之间共享数据 {/*sharing-data-between-components*/}

在前面的示例中，每个 `MyButton` 都有自己独立的 `count`，并且当每个按钮被点击时，只有被点击按钮的 `count` 会发生变化：

<DiagramGroup>

<Diagram name="sharing_data_child" height={367} width={407} alt="Diagram showing a tree of three components, one parent labeled MyApp and two children labeled MyButton. Both MyButton components contain a count with value zero.">

最初，每个 `MyButton` 的 `count` 状态都是 `0`

</Diagram>

<Diagram name="sharing_data_child_clicked" height={367} width={407} alt="The same diagram as the previous, with the count of the first child MyButton component highlighted indicating a click with the count value incremented to one. The second MyButton component still contains value zero." >

第一个 `MyButton` 将它的 `count` 更新为 `1`

</Diagram>

</DiagramGroup>

然而，通常你会需要组件 *共享数据并始终一起更新*。

要让这两个 `MyButton` 组件显示相同的 `count` 并一起更新，你需要将状态从各个按钮“提升”到包含它们的最近公共组件。

在这个例子中，它是 `MyApp`：

<DiagramGroup>

<Diagram name="sharing_data_parent" height={385} width={410} alt="Diagram showing a tree of three components, one parent labeled MyApp and two children labeled MyButton. MyApp contains a count value of zero which is passed down to both of the MyButton components, which also show value zero." >

最初，`MyApp` 的 `count` 状态是 `0`，并被传递给两个子组件

</Diagram>

<Diagram name="sharing_data_parent_clicked" height={385} width={410} alt="The same diagram as the previous, with the count of the parent MyApp component highlighted indicating a click with the value incremented to one. The flow to both of the children MyButton components is also highlighted, and the count value in each child is set to one indicating the value was passed down." >

点击时，`MyApp` 将它的 `count` 状态更新为 `1`，并将其传递给两个子组件

</Diagram>

</DiagramGroup>

现在，当你点击任意一个按钮时，`MyApp` 中的 `count` 会改变，这会使 `MyButton` 中两个计数都发生变化。下面是你如何在代码中表达这一点。

首先，将状态从 `MyButton` *提升* 到 `MyApp` 中：

```js {2-6,18}
export default function MyApp() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1);
  }

  return (
    <div>
      <h1>分别更新的计数器</h1>
      <MyButton />
      <MyButton />
    </div>
  );
}

function MyButton() {
  // ... 我们正在把代码从这里移走 ...
}

```

然后，将状态从 `MyApp` *向下传递* 给每个 `MyButton`，同时传入共享的点击处理函数。你可以使用 JSX 花括号将信息传递给 `MyButton`，就像你之前对 `<img>` 这类内置标签所做的那样：

```js {11-12}
export default function MyApp() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1);
  }

  return (
    <div>
      <h1>一起更新的计数器</h1>
      <MyButton count={count} onClick={handleClick} />
      <MyButton count={count} onClick={handleClick} />
    </div>
  );
}
```

你这样向下传递的信息被称为 _props_。现在 `MyApp` 组件包含了 `count` 状态和 `handleClick` 事件处理函数，并且把它们都作为 props *向下传递* 给每个按钮。

最后，修改 `MyButton`，让它*读取*从父组件传递过来的 props：

```js {1,3}
function MyButton({ count, onClick }) {
  return (
    <button onClick={onClick}>
      点击了 {count} 次
    </button>
  );
}
```

当你点击按钮时，`onClick` 处理函数会被触发。每个按钮的 `onClick` prop 都被设置为 `MyApp` 内部的 `handleClick` 函数，所以其中的代码会执行。那段代码调用 `setCount(count + 1)`，使 `count` 状态变量加一。新的 `count` 值会作为 prop 传递给每个按钮，因此它们都会显示新值。这被称为“状态提升”。通过提升状态，你已经在组件之间共享了它。

<Sandpack>

```js
import { useState } from 'react';

export default function MyApp() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1);
  }

  return (
    <div>
      <h1>一起更新的计数器</h1>
      <MyButton count={count} onClick={handleClick} />
      <MyButton count={count} onClick={handleClick} />
    </div>
  );
}

function MyButton({ count, onClick }) {
  return (
    <button onClick={onClick}>
      点击了 {count} 次
    </button>
  );
}
```

```css
button {
  display: block;
  margin-bottom: 5px;
}
```

</Sandpack>

## 下一步 {/*next-steps*/}

到现在为止，你已经了解了编写 React 代码的基础知识！

看看 [教程](/learn/tutorial-tic-tac-toe)，把这些知识付诸实践，并用 React 构建你的第一个迷你应用。
