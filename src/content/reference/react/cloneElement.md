---
title: cloneElement
---

<Pitfall>

使用 `cloneElement` 并不常见，而且可能导致代码脆弱。[查看常见替代方案。](#alternatives)

</Pitfall>

<Intro>

`cloneElement` 允许你基于另一个元素作为起点来创建一个新的 React 元素。

```js
const clonedElement = cloneElement(element, props, ...children)
```

</Intro>

<InlineToc />

---

## 参考 {/*reference*/}

### `cloneElement(element, props, ...children)` {/*cloneelement*/}

调用 `cloneElement` 可基于 `element` 创建一个 React 元素，但使用不同的 `props` 和 `children`：

```js
import { cloneElement } from 'react';

// ...
const clonedElement = cloneElement(
  <Row title="Cabbage">
    Hello
  </Row>,
  { isHighlighted: true },
  'Goodbye'
);

console.log(clonedElement); // <Row title="Cabbage" isHighlighted={true}>Goodbye</Row>
```

[查看更多示例。](#usage)

#### 参数 {/*parameters*/}

* `element`：`element` 参数必须是一个有效的 React 元素。例如，它可以是像 `<Something />` 这样的 JSX 节点、调用 [`createElement`](/reference/react/createElement) 的结果，或者另一次 `cloneElement` 调用的结果。

* `props`：`props` 参数必须是一个对象或者 `null`。如果你传入 `null`，克隆后的元素将保留原始 `element.props` 的全部内容。否则，对于 `props` 对象中的每个 prop，返回的元素都会“优先”采用 `props` 中的值，而不是 `element.props` 中的值。其余的 props 将从原始的 `element.props` 中补齐。如果你传入 `props.key` 或 `props.ref`，它们将替换原有的值。

* **可选** `...children`：零个或多个子节点。它们可以是任何 React 节点，包括 React 元素、字符串、数字、[portal](/reference/react-dom/createPortal)、空节点（`null`、`undefined`、`true` 和 `false`）以及 React 节点数组。如果你没有传入任何 `...children` 参数，原始的 `element.props.children` 将被保留。

#### 返回值 {/*returns*/}

`cloneElement` 返回一个带有若干属性的 React 元素对象：

* `type`：与 `element.type` 相同。
* `props`：将 `element.props` 与你传入的覆盖 `props` 进行浅合并后的结果。
* `ref`：原始的 `element.ref`，除非它被 `props.ref` 覆盖。
* `key`：原始的 `element.key`，除非它被 `props.key` 覆盖。

通常，你会从组件中返回这个元素，或者把它作为另一个元素的子元素。虽然你可以读取元素的属性，但最好在元素创建后把它当作不透明对象，只负责渲染它。

#### 注意事项 {/*caveats*/}

* 克隆一个元素**不会修改原始元素。**

* 只有在 children **都是静态已知的情况下，才应将它们作为多个参数传入 `cloneElement`，**例如 `cloneElement(element, null, child1, child2, child3)`。如果 children 是动态的，请将整个数组作为第三个参数传入：`cloneElement(element, null, listItems)`。这样可以确保 React 会对任何动态列表中缺失的 `key` 发出[警告](/learn/rendering-lists#keeping-list-items-in-order-with-key)。对于静态列表来说，这不是必须的，因为它们不会重新排序。

* `cloneElement` 会让追踪数据流变得更困难，因此**请尝试改用 [替代方案](#alternatives)。**

---

## 用法 {/*usage*/}

### 覆盖元素的 props {/*overriding-props-of-an-element*/}

要覆盖某个 <CodeStep step={1}>React 元素</CodeStep> 的 props，请将它传给 `cloneElement`，并带上你想要覆盖的 <CodeStep step={2}>props</CodeStep>：

```js {1,7}
import { cloneElement } from 'react';

// ...
const clonedElement = cloneElement(
  <Row title="Cabbage" />,
  { isHighlighted: true }
);
```

这里得到的 <CodeStep step={3}>克隆元素</CodeStep> 将是 `<Row title="Cabbage" isHighlighted={true} />`。

**让我们通过一个例子来看看它什么时候有用。**

假设有一个 `List` 组件，它将自己的 [`children`](/learn/passing-props-to-a-component#passing-jsx-as-children) 渲染为可选择的行列表，并带有一个“Next”按钮，用于改变当前选中的行。`List` 组件需要以不同方式渲染被选中的 `Row`，因此它会克隆收到的每个 `<Row>` 子元素，并额外添加一个 `isHighlighted: true` 或 `isHighlighted: false` prop：

```js {6-8}
export default function List({ children }) {
  const [selectedIndex, setSelectedIndex] = useState(0);
  return (
    <div className="List">
      {Children.map(children, (child, index) =>
        cloneElement(child, {
          isHighlighted: index === selectedIndex
        })
      )}
```

假设 `List` 接收到的原始 JSX 如下所示：

```js {2-4}
<List>
  <Row title="Cabbage" />
  <Row title="Garlic" />
  <Row title="Apple" />
</List>
```

通过克隆它的 children，`List` 可以向内部的每个 `Row` 传递额外信息。结果看起来像这样：

```js {4,8,12}
<List>
  <Row
    title="Cabbage"
    isHighlighted={true}
  />
  <Row
    title="Garlic"
    isHighlighted={false}
  />
  <Row
    title="Apple"
    isHighlighted={false}
  />
</List>
```

注意点击“Next”后会更新 `List` 的状态，并高亮另一行：

<Sandpack>

```js
import List from './List.js';
import Row from './Row.js';
import { products } from './data.js';

export default function App() {
  return (
    <List>
      {products.map(product =>
        <Row
          key={product.id}
          title={product.title}
        />
      )}
    </List>
  );
}
```

```js src/List.js active
import { Children, cloneElement, useState } from 'react';

export default function List({ children }) {
  const [selectedIndex, setSelectedIndex] = useState(0);
  return (
    <div className="List">
      {Children.map(children, (child, index) =>
        cloneElement(child, {
          isHighlighted: index === selectedIndex
        })
      )}
      <hr />
      <button onClick={() => {
        setSelectedIndex(i =>
          (i + 1) % Children.count(children)
        );
      }}>
        Next
      </button>
    </div>
  );
}
```

```js src/Row.js
export default function Row({ title, isHighlighted }) {
  return (
    <div className={[
      'Row',
      isHighlighted ? 'RowHighlighted' : ''
    ].join(' ')}>
      {title}
    </div>
  );
}
```

```js src/data.js
export const products = [
  { title: 'Cabbage', id: 1 },
  { title: 'Garlic', id: 2 },
  { title: 'Apple', id: 3 },
];
```

```css
.List {
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

.RowHighlighted {
  background: #ffa;
}

button {
  height: 40px;
  font-size: 20px;
}
```

</Sandpack>

总之，`List` 克隆了它接收到的 `<Row />` 元素，并向它们添加了一个额外的 prop。

<Pitfall>

克隆 children 会让你难以看清数据在应用中的流动方式。请尝试使用[替代方案。](#alternatives)

</Pitfall>

---

## 替代方案 {/*alternatives*/}

### 使用 render prop 传递数据 {/*passing-data-with-a-render-prop*/}

与其使用 `cloneElement`，不如考虑接收一个类似 `renderItem` 的 *render prop*。在这里，`List` 将 `renderItem` 作为 prop 接收。`List` 会为每个条目调用 `renderItem`，并将 `isHighlighted` 作为参数传入：

```js {1,7}
export default function List({ items, renderItem }) {
  const [selectedIndex, setSelectedIndex] = useState(0);
  return (
    <div className="List">
      {items.map((item, index) => {
        const isHighlighted = index === selectedIndex;
        return renderItem(item, isHighlighted);
      })}
```

`renderItem` 这个 prop 被称为“render prop”，因为它是一个用来指定如何渲染某物的 prop。例如，你可以传入一个 `renderItem` 的实现，它会使用给定的 `isHighlighted` 值来渲染一个 `<Row>`：

```js {3,7}
<List
  items={products}
  renderItem={(product, isHighlighted) =>
    <Row
      key={product.id}
      title={product.title}
      isHighlighted={isHighlighted}
    />
  }
/>
```

最终结果与使用 `cloneElement` 时相同：

```js {4,8,12}
<List>
  <Row
    title="Cabbage"
    isHighlighted={true}
  />
  <Row
    title="Garlic"
    isHighlighted={false}
  />
  <Row
    title="Apple"
    isHighlighted={false}
  />
</List>
```

不过，你可以清楚地追踪 `isHighlighted` 的值来自哪里。

<Sandpack>

```js
import List from './List.js';
import Row from './Row.js';
import { products } from './data.js';

export default function App() {
  return (
    <List
      items={products}
      renderItem={(product, isHighlighted) =>
        <Row
          key={product.id}
          title={product.title}
          isHighlighted={isHighlighted}
        />
      }
    />
  );
}
```

```js src/List.js active
import { useState } from 'react';

export default function List({ items, renderItem }) {
  const [selectedIndex, setSelectedIndex] = useState(0);
  return (
    <div className="List">
      {items.map((item, index) => {
        const isHighlighted = index === selectedIndex;
        return renderItem(item, isHighlighted);
      })}
      <hr />
      <button onClick={() => {
        setSelectedIndex(i =>
          (i + 1) % items.length
        );
      }}>
        Next
      </button>
    </div>
  );
}
```

```js src/Row.js
export default function Row({ title, isHighlighted }) {
  return (
    <div className={[
      'Row',
      isHighlighted ? 'RowHighlighted' : ''
    ].join(' ')}>
      {title}
    </div>
  );
}
```

```js src/data.js
export const products = [
  { title: 'Cabbage', id: 1 },
  { title: 'Garlic', id: 2 },
  { title: 'Apple', id: 3 },
];
```

```css
.List {
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

.RowHighlighted {
  background: #ffa;
}

button {
  height: 40px;
  font-size: 20px;
}
```

</Sandpack>

这种模式比 `cloneElement` 更推荐，因为它更明确。

---

### 通过 context 传递数据 {/*passing-data-through-context*/}

`cloneElement` 的另一种替代方案是[通过 context 传递数据。](/learn/passing-data-deeply-with-context)


例如，你可以调用 [`createContext`](/reference/react/createContext) 来定义一个 `HighlightContext`：

```js
export const HighlightContext = createContext(false);
```

你的 `List` 组件可以把它渲染的每个条目包裹在一个 `HighlightContext` provider 中：

```js {8,10}
export default function List({ items, renderItem }) {
  const [selectedIndex, setSelectedIndex] = useState(0);
  return (
    <div className="List">
      {items.map((item, index) => {
        const isHighlighted = index === selectedIndex;
        return (
          <HighlightContext key={item.id} value={isHighlighted}>
            {renderItem(item)}
          </HighlightContext>
        );
      })}
```

采用这种方式后，`Row` 根本不需要接收 `isHighlighted` prop。相反，它直接读取 context：

```js src/Row.js {2}
export default function Row({ title }) {
  const isHighlighted = useContext(HighlightContext);
  // ...
```

这样调用方组件就不需要知道，也不用担心把 `isHighlighted` 传给 `<Row>`：

```js {4}
<List
  items={products}
  renderItem={product =>
    <Row title={product.title} />
  }
/>
```

取而代之的是，`List` 和 `Row` 通过 context 协同实现高亮逻辑。

<Sandpack>

```js
import List from './List.js';
import Row from './Row.js';
import { products } from './data.js';

export default function App() {
  return (
    <List
      items={products}
      renderItem={(product) =>
        <Row title={product.title} />
      }
    />
  );
}
```

```js src/List.js active
import { useState } from 'react';
import { HighlightContext } from './HighlightContext.js';

export default function List({ items, renderItem }) {
  const [selectedIndex, setSelectedIndex] = useState(0);
  return (
    <div className="List">
      {items.map((item, index) => {
        const isHighlighted = index === selectedIndex;
        return (
          <HighlightContext
            key={item.id}
            value={isHighlighted}
          >
            {renderItem(item)}
          </HighlightContext>
        );
      })}
      <hr />
      <button onClick={() => {
        setSelectedIndex(i =>
          (i + 1) % items.length
        );
      }}>
        Next
      </button>
    </div>
  );
}
```

```js src/Row.js
import { useContext } from 'react';
import { HighlightContext } from './HighlightContext.js';

export default function Row({ title }) {
  const isHighlighted = useContext(HighlightContext);
  return (
    <div className={[
      'Row',
      isHighlighted ? 'RowHighlighted' : ''
    ].join(' ')}>
      {title}
    </div>
  );
}
```

```js src/HighlightContext.js
import { createContext } from 'react';

export const HighlightContext = createContext(false);
```

```js src/data.js
export const products = [
  { title: 'Cabbage', id: 1 },
  { title: 'Garlic', id: 2 },
  { title: 'Apple', id: 3 },
];
```

```css
.List {
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

.RowHighlighted {
  background: #ffa;
}

button {
  height: 40px;
  font-size: 20px;
}
```

</Sandpack>

[了解更多关于通过 context 传递数据的内容。](/reference/react/useContext#passing-data-deeply-into-the-tree)

---

### 将逻辑提取到自定义 Hook 中 {/*extracting-logic-into-a-custom-hook*/}

你还可以尝试的另一种方法是将“非可视化”逻辑提取到你自己的 Hook 中，并使用 Hook 返回的信息来决定渲染什么。例如，你可以编写一个 `useList` 自定义 Hook，如下所示：

```js
import { useState } from 'react';

export default function useList(items) {
  const [selectedIndex, setSelectedIndex] = useState(0);

  function onNext() {
    setSelectedIndex(i =>
      (i + 1) % items.length
    );
  }

  const selected = items[selectedIndex];
  return [selected, onNext];
}
```

然后你可以像这样使用它：

```js {2,9,13}
export default function App() {
  const [selected, onNext] = useList(products);
  return (
    <div className="List">
      {products.map(product =>
        <Row
          key={product.id}
          title={product.title}
          isHighlighted={selected === product}
        />
      )}
      <hr />
      <button onClick={onNext}>
        Next
      </button>
    </div>
  );
}
```

数据流是显式的，但状态位于 `useList` 自定义 Hook 内，你可以在任何组件中使用它：

<Sandpack>

```js
import Row from './Row.js';
import useList from './useList.js';
import { products } from './data.js';

export default function App() {
  const [selected, onNext] = useList(products);
  return (
    <div className="List">
      {products.map(product =>
        <Row
          key={product.id}
          title={product.title}
          isHighlighted={selected === product}
        />
      )}
      <hr />
      <button onClick={onNext}>
        Next
      </button>
    </div>
  );
}
```

```js src/useList.js
import { useState } from 'react';

export default function useList(items) {
  const [selectedIndex, setSelectedIndex] = useState(0);

  function onNext() {
    setSelectedIndex(i =>
      (i + 1) % items.length
    );
  }

  const selected = items[selectedIndex];
  return [selected, onNext];
}
```

```js src/Row.js
export default function Row({ title, isHighlighted }) {
  return (
    <div className={[
      'Row',
      isHighlighted ? 'RowHighlighted' : ''
    ].join(' ')}>
      {title}
    </div>
  );
}
```

```js src/data.js
export const products = [
  { title: 'Cabbage', id: 1 },
  { title: 'Garlic', id: 2 },
  { title: 'Apple', id: 3 },
];
```

```css
.List {
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

.RowHighlighted {
  background: #ffa;
}

button {
  height: 40px;
  font-size: 20px;
}
```

</Sandpack>

如果你想在不同组件之间复用这段逻辑，这种方法尤其有用。
