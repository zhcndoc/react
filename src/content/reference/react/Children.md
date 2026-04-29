---
title: 子元素
---

<Pitfall>

使用 `Children` 并不常见，而且可能导致代码脆弱。[查看常见替代方案。](#alternatives)

</Pitfall>

<Intro>

`Children` 让你可以操作并转换作为 [`children` 属性](/learn/passing-props-to-a-component#passing-jsx-as-children) 接收到的 JSX。

```js
const mappedChildren = Children.map(children, child =>
  <div className="Row">
    {child}
  </div>
);

```

</Intro>

<InlineToc />

---

## 参考 {/*reference*/}

### `Children.count(children)` {/*children-count*/}

调用 `Children.count(children)` 来统计 `children` 数据结构中的子元素数量。

```js src/RowList.js active
import { Children } from 'react';

function RowList({ children }) {
  return (
    <>
      <h1>总行数：{Children.count(children)}</h1>
      ...
    </>
  );
}
```

[查看更多示例。](#counting-children)

#### 参数 {/*children-count-parameters*/}

* `children`：你的组件接收到的 [`children` 属性](/learn/passing-props-to-a-component#passing-jsx-as-children) 的值。

#### 返回值 {/*children-count-returns*/}

这些 `children` 中的节点数量。

#### 注意事项 {/*children-count-caveats*/}

- 空节点（`null`、`undefined` 和布尔值）、字符串、数字以及 [React 元素](/reference/react/createElement) 都算作单独的节点。数组不算作单独的节点，但它们的子节点会计算在内。**遍历不会深入到 React 元素内部：** 它们不会被渲染，也不会继续遍历它们的子节点。[Fragment](/reference/react/Fragment) 不会被遍历。

---

### `Children.forEach(children, fn, thisArg?)` {/*children-foreach*/}

调用 `Children.forEach(children, fn, thisArg?)` 来对 `children` 数据结构中的每个子元素执行一些代码。

```js src/RowList.js active
import { Children } from 'react';

function SeparatorList({ children }) {
  const result = [];
  Children.forEach(children, (child, index) => {
    result.push(child);
    result.push(<hr key={index} />);
  });
  // ...
```

[查看更多示例。](#running-some-code-for-each-child)

#### 参数 {/*children-foreach-parameters*/}

* `children`：你的组件接收到的 [`children` 属性](/learn/passing-props-to-a-component#passing-jsx-as-children) 的值。
* `fn`：你希望对每个子元素执行的函数，类似于 [数组 `forEach` 方法](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/forEach) 的回调。它会以子元素作为第一个参数、索引作为第二个参数被调用。索引从 `0` 开始，并在每次调用时递增。你需要从这个函数返回一个 React 节点。它可以是空节点（`null`、`undefined` 或布尔值）、字符串、数字、React 元素，或者其他 React 节点组成的数组。
* **可选** `thisArg`：调用 `fn` 函数时所使用的 [`this` 值](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/this)。如果省略，则为 `undefined`。

#### 返回值 {/*children-foreach-returns*/}

`Children.forEach` 返回 `undefined`。

#### 注意事项 {/*children-foreach-caveats*/}

- 空节点（`null`、`undefined` 和布尔值）、字符串、数字以及 [React 元素](/reference/react/createElement) 都算作单独的节点。数组不算作单独的节点，但它们的子节点会计算在内。**遍历不会深入到 React 元素内部：** 它们不会被渲染，也不会继续遍历它们的子节点。[Fragment](/reference/react/Fragment) 不会被遍历。

---

### `Children.map(children, fn, thisArg?)` {/*children-map*/}

调用 `Children.map(children, fn, thisArg?)` 来映射或转换 `children` 数据结构中的每个子元素。

```js src/RowList.js active
import { Children } from 'react';

function RowList({ children }) {
  return (
    <div className="RowList">
      {Children.map(children, child =>
        <div className="Row">
          {child}
        </div>
      )}
    </div>
  );
}
```

[查看更多示例。](#transforming-children)

#### 参数 {/*children-map-parameters*/}

* `children`：你的组件接收到的 [`children` 属性](/learn/passing-props-to-a-component#passing-jsx-as-children) 的值。
* `fn`：映射函数，类似于 [数组 `map` 方法](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map) 的回调。它会以子元素作为第一个参数、索引作为第二个参数被调用。索引从 `0` 开始，并在每次调用时递增。你需要从这个函数返回一个 React 节点。它可以是空节点（`null`、`undefined` 或布尔值）、字符串、数字、React 元素，或者其他 React 节点组成的数组。
* **可选** `thisArg`：调用 `fn` 函数时所使用的 [`this` 值](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/this)。如果省略，则为 `undefined`。

#### 返回值 {/*children-map-returns*/}

如果 `children` 是 `null` 或 `undefined`，则返回相同的值。

否则，返回一个由你从 `fn` 函数中返回的节点组成的扁平数组。返回的数组将包含你返回的所有节点，但 `null` 和 `undefined` 除外。

#### 注意事项 {/*children-map-caveats*/}

- 空节点（`null`、`undefined` 和布尔值）、字符串、数字以及 [React 元素](/reference/react/createElement) 都算作单独的节点。数组不算作单独的节点，但它们的子节点会计算在内。**遍历不会深入到 React 元素内部：** 它们不会被渲染，也不会继续遍历它们的子节点。[Fragment](/reference/react/Fragment) 不会被遍历。

- 如果你从 `fn` 返回一个带有 key 的元素，或者返回一个带有 key 的元素数组，**返回元素的 key 会自动与 `children` 中对应原始项的 key 组合。** 当你在数组中从 `fn` 返回多个元素时，它们的 key 只需要在彼此之间局部唯一即可。

---

### `Children.only(children)` {/*children-only*/}


调用 `Children.only(children)` 来断言 `children` 表示一个单独的 React 元素。

```js
function Box({ children }) {
  const element = Children.only(children);
  // ...
```

#### 参数 {/*children-only-parameters*/}

* `children`：你的组件接收到的 [`children` 属性](/learn/passing-props-to-a-component#passing-jsx-as-children) 的值。

#### 返回值 {/*children-only-returns*/}

如果 `children` [是一个有效元素，](/reference/react/isValidElement) 则返回该元素。

否则，抛出错误。

#### 注意事项 {/*children-only-caveats*/}

- 如果你将数组（例如 `Children.map` 的返回值）作为 `children` 传入，该方法总会**抛出错误。** 换句话说，它要求 `children` 是单个 React 元素，而不是包含单个元素的数组。

---

### `Children.toArray(children)` {/*children-toarray*/}

调用 `Children.toArray(children)` 来从 `children` 数据结构创建一个数组。

```js src/ReversedList.js active
import { Children } from 'react';

export default function ReversedList({ children }) {
  const result = Children.toArray(children);
  result.reverse();
  // ...
```

#### 参数 {/*children-toarray-parameters*/}

* `children`：你的组件接收到的 [`children` 属性](/learn/passing-props-to-a-component#passing-jsx-as-children) 的值。

#### 返回值 {/*children-toarray-returns*/}

返回 `children` 中元素组成的扁平数组。

#### 注意事项 {/*children-toarray-caveats*/}

- 空节点（`null`、`undefined` 和布尔值）会在返回的数组中被省略。**返回元素的 key 将根据原始元素的 key、其嵌套层级和位置计算得出。** 这可确保扁平化数组不会引入行为变化。

---

## 用法 {/*usage*/}

### 转换子元素 {/*transforming-children*/}

要转换你的组件[作为 `children` 属性接收到的子元素 JSX，](/learn/passing-props-to-a-component#passing-jsx-as-children)请调用 `Children.map`：

```js {6,10}
import { Children } from 'react';

function RowList({ children }) {
  return (
    <div className="RowList">
      {Children.map(children, child =>
        <div className="Row">
          {child}
        </div>
      )}
    </div>
  );
}
```

在上面的示例中，`RowList` 会把它接收到的每个子元素都包裹到一个 `<div className="Row">` 容器中。例如，假设父组件将三个 `<p>` 标签作为 `children` 属性传给 `RowList`：

```js
<RowList>
  <p>这是第一项。</p>
  <p>这是第二项。</p>
  <p>这是第三项。</p>
</RowList>
```

那么，使用上面的 `RowList` 实现后，最终渲染结果将如下所示：

```js
<div className="RowList">
  <div className="Row">
    <p>这是第一项。</p>
  </div>
  <div className="Row">
    <p>这是第二项。</p>
  </div>
  <div className="Row">
    <p>这是第三项。</p>
  </div>
</div>
```

`Children.map` 类似于[使用 `map()` 转换数组。](/learn/rendering-lists) 区别在于 `children` 数据结构被视为*不透明的*。这意味着即使它有时是一个数组，你也不应假设它一定是数组或其他任何特定数据类型。因此，如果你需要转换它，就应该使用 `Children.map`。

<Sandpack>

```js
import RowList from './RowList.js';

export default function App() {
  return (
    <RowList>
      <p>这是第一项。</p>
      <p>这是第二项。</p>
      <p>这是第三项。</p>
    </RowList>
  );
}
```

```js src/RowList.js active
import { Children } from 'react';

export default function RowList({ children }) {
  return (
    <div className="RowList">
      {Children.map(children, child =>
        <div className="Row">
          {child}
        </div>
      )}
    </div>
  );
}
```

```css
.RowList {
  display: flex;
  flex-direction: column;
  border: 2px solid grey;
  padding: 5px;
}

.Row {
  border: 2px dashed black;
  padding: 5px;
  margin: 5px;
}
```

</Sandpack>

<DeepDive>

#### 为什么 `children` 属性不总是数组？ {/*why-is-the-children-prop-not-always-an-array*/}

在 React 中，`children` 属性被视为一种*不透明*的数据结构。这意味着你不应该依赖它的结构。要转换、筛选或统计子元素，你应该使用 `Children` 方法。

在实践中，`children` 数据结构在内部通常会表示为数组。然而，如果只有一个子元素，React 就不会创建额外的数组，因为这会带来不必要的内存开销。只要你使用 `Children` 方法，而不是直接检查 `children` 属性，即使 React 更改了该数据结构的实际实现方式，你的代码也不会出错。

即使 `children` 是数组，`Children.map` 也有有用的特殊行为。例如，`Children.map` 会将返回元素上的 [key](/learn/rendering-lists#keeping-list-items-in-order-with-key) 与你传给它的 `children` 上的 key 组合起来。这样可以确保原始 JSX 子元素即使像上面的示例那样被包裹起来，也不会“丢失” key。

</DeepDive>

<Pitfall>

`children` 数据结构**不包含**你作为 JSX 传入的组件的渲染输出。在下面的示例中，`RowList` 接收到的 `children` 只包含两个条目，而不是三个：

1. `<p>这是第一项。</p>`
2. `<MoreRows />`

这就是为什么这个示例只生成了两个行包装器：

<Sandpack>

```js
import RowList from './RowList.js';

export default function App() {
  return (
    <RowList>
      <p>这是第一项。</p>
      <MoreRows />
    </RowList>
  );
}

function MoreRows() {
  return (
    <>
      <p>这是第二项。</p>
      <p>这是第三项。</p>
    </>
  );
}
```

```js src/RowList.js
import { Children } from 'react';

export default function RowList({ children }) {
  return (
    <div className="RowList">
      {Children.map(children, child =>
        <div className="Row">
          {child}
        </div>
      )}
    </div>
  );
}
```

```css
.RowList {
  display: flex;
  flex-direction: column;
  border: 2px solid grey;
  padding: 5px;
}

.Row {
  border: 2px dashed black;
  padding: 5px;
  margin: 5px;
}
```

</Sandpack>

在操作 `children` 时，**无法获取像 `<MoreRows />` 这样的内部组件的渲染输出。** 这就是为什么[通常最好使用其他替代方案。](#alternatives)

</Pitfall>

---

### 为每个子元素运行一些代码 {/*running-some-code-for-each-child*/}

调用 `Children.forEach` 来遍历 `children` 数据结构中的每个子元素。它不返回任何值，类似于 [数组 `forEach` 方法。](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/forEach) 你可以用它来执行自定义逻辑，比如构建你自己的数组。

<Sandpack>

```js
import SeparatorList from './SeparatorList.js';

export default function App() {
  return (
    <SeparatorList>
      <p>这是第一项。</p>
      <p>这是第二项。</p>
      <p>这是第三项。</p>
    </SeparatorList>
  );
}
```

```js src/SeparatorList.js active
import { Children } from 'react';

export default function SeparatorList({ children }) {
  const result = [];
  Children.forEach(children, (child, index) => {
    result.push(child);
    result.push(<hr key={index} />);
  });
  result.pop(); // 移除最后一个分隔符
  return result;
}
```

</Sandpack>

<Pitfall>

如前所述，在操作 `children` 时，无法获取内部组件的渲染输出。这就是为什么[通常最好使用其他替代方案。](#alternatives)

</Pitfall>

---

### 统计子元素数量 {/*counting-children*/}

调用 `Children.count(children)` 来计算子元素数量。

<Sandpack>

```js
import RowList from './RowList.js';

export default function App() {
  return (
    <RowList>
      <p>这是第一项。</p>
      <p>这是第二项。</p>
      <p>这是第三项。</p>
    </RowList>
  );
}
```

```js src/RowList.js active
import { Children } from 'react';

export default function RowList({ children }) {
  return (
    <div className="RowList">
      <h1 className="RowListHeader">
        总行数：{Children.count(children)}
      </h1>
      {Children.map(children, child =>
        <div className="Row">
          {child}
        </div>
      )}
    </div>
  );
}
```

```css
.RowList {
  display: flex;
  flex-direction: column;
  border: 2px solid grey;
  padding: 5px;
}

.RowListHeader {
  padding-top: 5px;
  font-size: 25px;
  font-weight: bold;
  text-align: center;
}

.Row {
  border: 2px dashed black;
  padding: 5px;
  margin: 5px;
}
```

</Sandpack>

<Pitfall>

如前所述，在操作 `children` 时，无法获取内部组件的渲染输出。这就是为什么[通常最好使用其他替代方案。](#alternatives)

</Pitfall>

---

### 将子元素转换为数组 {/*converting-children-to-an-array*/}

调用 `Children.toArray(children)` 将 `children` 数据结构转换为普通的 JavaScript 数组。这样你就可以使用内置数组方法来操作该数组，例如 [`filter`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/filter)、[`sort`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/sort) 或 [`reverse`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/reverse)。

<Sandpack>

```js
import ReversedList from './ReversedList.js';

export default function App() {
  return (
    <ReversedList>
      <p>这是第一项。</p>
      <p>这是第二项。</p>
      <p>这是第三项。</p>
    </ReversedList>
  );
}
```

```js src/ReversedList.js active
import { Children } from 'react';

export default function ReversedList({ children }) {
  const result = Children.toArray(children);
  result.reverse();
  return result;
}
```

</Sandpack>

<Pitfall>

如前所述，在操作 `children` 时，无法获取内部组件的渲染输出。这就是为什么[通常最好使用其他替代方案。](#alternatives)

</Pitfall>

---

## 替代方案 {/*alternatives*/}

<Note>

本节介绍 `Children` API（大写 `C`）的替代方案，它是这样导入的：

```js
import { Children } from 'react';
```

不要把它和 [使用 `children` prop](/learn/passing-props-to-a-component#passing-jsx-as-children)（小写 `c`）混淆，后者是很好的做法，并且值得推荐。

</Note>

### 暴露多个组件 {/*exposing-multiple-components*/}

使用 `Children` 方法来操作 children 往往会导致代码脆弱。当你在 JSX 中把 children 传给某个组件时，你通常不会期望这个组件去操作或转换单个 child。

在可能的情况下，尽量避免使用 `Children` 方法。例如，如果你希望 `RowList` 的每个 child 都被包裹在 `<div className="Row">` 中，那么可以导出一个 `Row` 组件，并像这样手动把每一行包裹进去：

<Sandpack>

```js
import { RowList, Row } from './RowList.js';

export default function App() {
  return (
    <RowList>
      <Row>
        <p>这是第一项。</p>
      </Row>
      <Row>
        <p>这是第二项。</p>
      </Row>
      <Row>
        <p>这是第三项。</p>
      </Row>
    </RowList>
  );
}
```

```js src/RowList.js
export function RowList({ children }) {
  return (
    <div className="RowList">
      {children}
    </div>
  );
}

export function Row({ children }) {
  return (
    <div className="Row">
      {children}
    </div>
  );
}
```

```css
.RowList {
  display: flex;
  flex-direction: column;
  border: 2px solid grey;
  padding: 5px;
}

.Row {
  border: 2px dashed black;
  padding: 5px;
  margin: 5px;
}
```

</Sandpack>

与使用 `Children.map` 不同，这种方法不会自动包裹每个 child。**不过，与前面使用 `Children.map` 的示例相比，这种方法有一个显著优势：即使你继续提取更多组件，它也能正常工作。**例如，即使你提取出自己的 `MoreRows` 组件，它仍然可以工作：

<Sandpack>

```js
import { RowList, Row } from './RowList.js';

export default function App() {
  return (
    <RowList>
      <Row>
        <p>这是第一项。</p>
      </Row>
      <MoreRows />
    </RowList>
  );
}

function MoreRows() {
  return (
    <>
      <Row>
        <p>这是第二项。</p>
      </Row>
      <Row>
        <p>这是第三项。</p>
      </Row>
    </>
  );
}
```

```js src/RowList.js
export function RowList({ children }) {
  return (
    <div className="RowList">
      {children}
    </div>
  );
}

export function Row({ children }) {
  return (
    <div className="Row">
      {children}
    </div>
  );
}
```

```css
.RowList {
  display: flex;
  flex-direction: column;
  border: 2px solid grey;
  padding: 5px;
}

.Row {
  border: 2px dashed black;
  padding: 5px;
  margin: 5px;
}
```

</Sandpack>

这在 `Children.map` 下不会生效，因为它会把 `<MoreRows />` 看作一个单独的 child（以及一个单独的行）。

---

### 接收对象数组作为 prop {/*accepting-an-array-of-objects-as-a-prop*/}

你也可以显式地将数组作为 prop 传入。例如，这个 `RowList` 接收一个 `rows` 数组作为 prop：

<Sandpack>

```js
import { RowList, Row } from './RowList.js';

export default function App() {
  return (
    <RowList rows={[
      { id: 'first', content: <p>这是第一项。</p> },
      { id: 'second', content: <p>这是第二项。</p> },
      { id: 'third', content: <p>这是第三项。</p> }
    ]} />
  );
}
```

```js src/RowList.js
export function RowList({ rows }) {
  return (
    <div className="RowList">
      {rows.map(row => (
        <div className="Row" key={row.id}>
          {row.content}
        </div>
      ))}
    </div>
  );
}
```

```css
.RowList {
  display: flex;
  flex-direction: column;
  border: 2px solid grey;
  padding: 5px;
}

.Row {
  border: 2px dashed black;
  padding: 5px;
  margin: 5px;
}
```

</Sandpack>

由于 `rows` 是普通的 JavaScript 数组，`RowList` 组件可以在它上面使用内置的数组方法，比如 [`map`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map)。

当你希望能将更多信息作为结构化数据与 children 一起传递时，这种模式尤其有用。在下面的示例中，`TabSwitcher` 组件接收一个对象数组作为 `tabs` prop：

<Sandpack>

```js
import TabSwitcher from './TabSwitcher.js';

export default function App() {
  return (
    <TabSwitcher tabs={[
      {
        id: 'first',
        header: '第一项',
        content: <p>这是第一项。</p>
      },
      {
        id: 'second',
        header: '第二项',
        content: <p>这是第二项。</p>
      },
      {
        id: 'third',
        header: '第三项',
        content: <p>这是第三项。</p>
      }
    ]} />
  );
}
```

```js src/TabSwitcher.js
import { useState } from 'react';

export default function TabSwitcher({ tabs }) {
  const [selectedId, setSelectedId] = useState(tabs[0].id);
  const selectedTab = tabs.find(tab => tab.id === selectedId);
  return (
    <>
      {tabs.map(tab => (
        <button
          key={tab.id}
          onClick={() => setSelectedId(tab.id)}
        >
          {tab.header}
        </button>
      ))}
      <hr />
      <div key={selectedId}>
        <h3>{selectedTab.header}</h3>
        {selectedTab.content}
      </div>
    </>
  );
}
```

</Sandpack>

与把 children 作为 JSX 传递不同，这种方法可以让你把像 `header` 这样的额外数据与每一项关联起来。因为你直接在使用 `tabs`，而且它是一个数组，所以你不需要 `Children` 方法。

---

### 调用 render prop 来自定义渲染 {/*calling-a-render-prop-to-customize-rendering*/}

你也可以不为每个单独的 item 生成 JSX，而是传入一个返回 JSX 的函数，并在需要时调用该函数。在这个示例中，`App` 组件向 `TabSwitcher` 组件传递了一个 `renderContent` 函数。`TabSwitcher` 组件只会为被选中的 tab 调用 `renderContent`：

<Sandpack>

```js
import TabSwitcher from './TabSwitcher.js';

export default function App() {
  return (
    <TabSwitcher
      tabIds={['first', 'second', 'third']}
      getHeader={tabId => {
        return tabId[0].toUpperCase() + tabId.slice(1);
      }}
      renderContent={tabId => {
        return <p>这是 {tabId} 项。</p>;
      }}
    />
  );
}
```

```js src/TabSwitcher.js
import { useState } from 'react';

export default function TabSwitcher({ tabIds, getHeader, renderContent }) {
  const [selectedId, setSelectedId] = useState(tabIds[0]);
  return (
    <>
      {tabIds.map((tabId) => (
        <button
          key={tabId}
          onClick={() => setSelectedId(tabId)}
        >
          {getHeader(tabId)}
        </button>
      ))}
      <hr />
      <div key={selectedId}>
        <h3>{getHeader(selectedId)}</h3>
        {renderContent(selectedId)}
      </div>
    </>
  );
}
```

</Sandpack>

像 `renderContent` 这样的 prop 被称为 *render prop*，因为它是一个指定如何渲染某部分用户界面的 prop。不过，它并没有什么特殊之处：它只是一个碰巧是函数的普通 prop。

Render prop 是函数，所以你可以向它传递信息。例如，这个 `RowList` 组件会把每一行的 `id` 和 `index` 传给 `renderRow` render prop，而 `renderRow` 使用 `index` 来高亮偶数行：

<Sandpack>

```js
import { RowList, Row } from './RowList.js';

export default function App() {
  return (
    <RowList
      rowIds={['first', 'second', 'third']}
      renderRow={(id, index) => {
        return (
          <Row isHighlighted={index % 2 === 0}>
            <p>这是 {id} 项。</p>
          </Row>
        );
      }}
    />
  );
}
```

```js src/RowList.js
import { Fragment } from 'react';

export function RowList({ rowIds, renderRow }) {
  return (
    <div className="RowList">
      <h1 className="RowListHeader">
        总行数：{rowIds.length}
      </h1>
      {rowIds.map((rowId, index) =>
        <Fragment key={rowId}>
          {renderRow(rowId, index)}
        </Fragment>
      )}
    </div>
  );
}

export function Row({ children, isHighlighted }) {
  return (
    <div className={[
      'Row',
      isHighlighted ? 'RowHighlighted' : ''
    ].join(' ')}>
      {children}
    </div>
  );
}
```

```css
.RowList {
  display: flex;
  flex-direction: column;
  border: 2px solid grey;
  padding: 5px;
}

.RowListHeader {
  padding-top: 5px;
  font-size: 25px;
  font-weight: bold;
  text-align: center;
}

.Row {
  border: 2px dashed black;
  padding: 5px;
  margin: 5px;
}

.RowHighlighted {
  background: #ffa;
}
```

</Sandpack>

这又是一个父组件和子组件可以在不操作 children 的情况下协作的例子。

---

## 故障排除 {/*troubleshooting*/}

### 我传入了一个自定义组件，但 `Children` 方法没有显示它的渲染结果 {/*i-pass-a-custom-component-but-the-children-methods-dont-show-its-render-result*/}

假设你像这样向 `RowList` 传递了两个 children：

```js
<RowList>
  <p>第一项</p>
  <MoreRows />
</RowList>
```

如果你在 `RowList` 内部执行 `Children.count(children)`，你会得到 `2`。即使 `MoreRows` 渲染了 10 个不同的项，或者它返回 `null`，`Children.count(children)` 仍然会是 `2`。从 `RowList` 的角度来看，它只会“看到”自己接收到的 JSX。它不会“看到” `MoreRows` 组件内部的实现。

这个限制会让提取组件变得困难。这就是为什么相比使用 `Children`，更推荐采用这些[替代方案](#alternatives)。
