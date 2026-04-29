---
title: 组件之间共享状态
---

<Intro>

有时，你希望两个组件的状态总是一起变化。要做到这一点，把这两个组件中的状态移除，将它提升到它们最近的共同父组件中，然后通过 props 传递给它们。这被称为 *状态提升*，而且这是你在编写 React 代码时最常做的事情之一。

</Intro>

<YouWillLearn>

- 如何通过提升状态来在组件之间共享状态
- 什么是受控组件和非受控组件

</YouWillLearn>

## 通过示例提升状态 {/*lifting-state-up-by-example*/}

在这个示例中，父级 `Accordion` 组件渲染两个独立的 `Panel`：

* `Accordion`
  - `Panel`
  - `Panel`

每个 `Panel` 组件都有一个布尔类型的 `isActive` 状态，用来决定其内容是否可见。

点击两个面板的 Show 按钮：

<Sandpack>

```js
import { useState } from 'react';

function Panel({ title, children }) {
  const [isActive, setIsActive] = useState(false);
  return (
    <section className="panel">
      <h3>{title}</h3>
      {isActive ? (
        <p>{children}</p>
      ) : (
        <button onClick={() => setIsActive(true)}>
          显示
        </button>
      )}
    </section>
  );
}

export default function Accordion() {
  return (
    <>
      <h2>阿拉木图，哈萨克斯坦</h2>
      <Panel title="关于">
        阿拉木图人口约 200 万，是哈萨克斯坦最大的城市。1929 年到 1997 年间，它曾是哈萨克斯坦的首都。
      </Panel>
      <Panel title="词源">
        这个名字来自 <span lang="kk-KZ">алма</span>，这是哈萨克语中“苹果”的意思，通常被翻译为“苹果之都”。实际上，阿拉木图周边地区被认为是苹果的祖先家园，而野生的 <i lang="la">Malus sieversii</i> 被认为很可能是现代栽培苹果的祖先。
      </Panel>
    </>
  );
}
```

```css
h3, p { margin: 5px 0px; }
.panel {
  padding: 10px;
  border: 1px solid #aaa;
}
```

</Sandpack>

注意，点击一个面板的按钮不会影响另一个面板——它们是相互独立的。

<DiagramGroup>

<Diagram name="sharing_state_child" height={367} width={477} alt="图示显示一棵包含三个组件的树，一个父组件标记为 Accordion，两个子组件标记为 Panel。两个 Panel 组件都包含 isActive，值为 false。">

最初，每个 `Panel` 的 `isActive` 状态都是 `false`，所以它们看起来都是折叠的

</Diagram>

<Diagram name="sharing_state_child_clicked" height={367} width={480} alt="与前一个相同的图示，第一子级 Panel 组件的 isActive 被高亮，表示点击并将 isActive 值设为 true。第二个 Panel 组件仍然包含值 false。">

点击任意一个 `Panel` 的按钮只会单独更新那个 `Panel` 的 `isActive` 状态

</Diagram>

</DiagramGroup>

**但现在假设你想把它改成任意时刻都只能展开一个面板。** 按照这种设计，展开第二个面板时应该折叠第一个面板。你会怎么做？

要协调这两个面板，你需要通过三个步骤把它们的“状态提升”到父组件中：

1. **移除**子组件中的状态。
2. 从共同父组件中**传递**硬编码数据。
3. 向共同父组件**添加**状态，并连同事件处理函数一起向下传递。

这样 `Accordion` 组件就可以协调这两个 `Panel`，并且一次只展开一个。

### 第 1 步：从子组件中移除状态 {/*step-1-remove-state-from-the-child-components*/}

你将把 `Panel` 的 `isActive` 控制权交给它的父组件。这意味着父组件会改为把 `isActive` 作为 props 传给 `Panel`。首先，从 `Panel` 组件中**移除这一行**：

```js
const [isActive, setIsActive] = useState(false);
```

然后，把 `isActive` 添加到 `Panel` 的 props 列表中：

```js
function Panel({ title, children, isActive }) {
```

现在，`Panel` 的父组件可以通过[将其作为 prop 传下去](/learn/passing-props-to-a-component)来*控制* `isActive`。反过来说，`Panel` 组件现在对 `isActive` 的值*没有控制权*——这完全取决于父组件！

### 第 2 步：从共同父组件传递硬编码数据 {/*step-2-pass-hardcoded-data-from-the-common-parent*/}

要提升状态，你必须找出你想协调的*两个*子组件的最近共同父组件：

* `Accordion` *(最近共同父组件)*
  - `Panel`
  - `Panel`

在这个例子里，它就是 `Accordion` 组件。由于它位于两个面板之上，并且可以控制它们的 props，所以它会成为当前哪个面板处于活动状态的“单一事实来源”。让 `Accordion` 组件向两个面板传递一个硬编码的 `isActive` 值（例如 `true`）：

<Sandpack>

```js
import { useState } from 'react';

export default function Accordion() {
  return (
    <>
      <h2>阿拉木图，哈萨克斯坦</h2>
      <Panel title="关于" isActive={true}>
        阿拉木图人口约 200 万，是哈萨克斯坦最大的城市。1929 年到 1997 年间，它曾是哈萨克斯坦的首都。
      </Panel>
      <Panel title="词源" isActive={true}>
        这个名字来自 <span lang="kk-KZ">алма</span>，这是哈萨克语中“苹果”的意思，通常被翻译为“苹果之都”。实际上，阿拉木图周边地区被认为是苹果的祖先家园，而野生的 <i lang="la">Malus sieversii</i> 被认为很可能是现代栽培苹果的祖先。
      </Panel>
    </>
  );
}

function Panel({ title, children, isActive }) {
  return (
    <section className="panel">
      <h3>{title}</h3>
      {isActive ? (
        <p>{children}</p>
      ) : (
        <button onClick={() => setIsActive(true)}>
          显示
        </button>
      )}
    </section>
  );
}
```

```css
h3, p { margin: 5px 0px; }
.panel {
  padding: 10px;
  border: 1px solid #aaa;
}
```

</Sandpack>

试着编辑 `Accordion` 组件中硬编码的 `isActive` 值，看看屏幕上的结果。

### 第 3 步：向共同父组件添加状态 {/*step-3-add-state-to-the-common-parent*/}

提升状态通常会改变你把什么内容存储为状态的方式。

在这个例子里，任意时刻都只能有一个面板处于活动状态。这意味着 `Accordion` 这个共同父组件需要跟踪*哪个*面板是活动的。与其使用布尔值，不如使用数字作为状态变量中活动 `Panel` 的索引：

```js
const [activeIndex, setActiveIndex] = useState(0);
```

当 `activeIndex` 为 `0` 时，第一个面板处于活动状态；当它为 `1` 时，第二个面板处于活动状态。

点击任意一个 `Panel` 中的 "Show" 按钮都需要改变 `Accordion` 中的活动索引。`Panel` 不能直接设置 `activeIndex` 状态，因为它定义在 `Accordion` 内部。`Accordion` 组件需要通过[将事件处理函数作为 prop 传下去](/learn/responding-to-events#passing-event-handlers-as-props)来*明确允许* `Panel` 组件改变它的状态：

```js
<>
  <Panel
    isActive={activeIndex === 0}
    onShow={() => setActiveIndex(0)}
  >
    ...
  </Panel>
  <Panel
    isActive={activeIndex === 1}
    onShow={() => setActiveIndex(1)}
  >
    ...
  </Panel>
</>
```

`Panel` 内部的 `<button>` 现在会使用 `onShow` prop 作为它的点击事件处理函数：

<Sandpack>

```js
import { useState } from 'react';

export default function Accordion() {
  const [activeIndex, setActiveIndex] = useState(0);
  return (
    <>
      <h2>阿拉木图，哈萨克斯坦</h2>
      <Panel
        title="关于"
        isActive={activeIndex === 0}
        onShow={() => setActiveIndex(0)}
      >
        阿拉木图人口约 200 万，是哈萨克斯坦最大的城市。1929 年到 1997 年间，它曾是哈萨克斯坦的首都。
      </Panel>
      <Panel
        title="词源"
        isActive={activeIndex === 1}
        onShow={() => setActiveIndex(1)}
      >
        这个名字来自 <span lang="kk-KZ">алма</span>，这是哈萨克语中“苹果”的意思，通常被翻译为“苹果之都”。实际上，阿拉木图周边地区被认为是苹果的祖先家园，而野生的 <i lang="la">Malus sieversii</i> 被认为很可能是现代栽培苹果的祖先。
      </Panel>
    </>
  );
}

function Panel({
  title,
  children,
  isActive,
  onShow
}) {
  return (
    <section className="panel">
      <h3>{title}</h3>
      {isActive ? (
        <p>{children}</p>
      ) : (
        <button onClick={onShow}>
          显示
        </button>
      )}
    </section>
  );
}
```

```css
h3, p { margin: 5px 0px; }
.panel {
  padding: 10px;
  border: 1px solid #aaa;
}
```

</Sandpack>

这就完成了状态提升！将状态移到共同父组件中，使你能够协调这两个面板。使用活动索引而不是两个“是否显示”标志，确保任意时刻只有一个面板处于活动状态。把事件处理函数向下传递给子组件，则允许子组件改变父组件的状态。

<DiagramGroup>

<Diagram name="sharing_state_parent" height={385} width={487} alt="图示显示一棵包含三个组件的树，一个父组件标记为 Accordion，两个子组件标记为 Panel。Accordion 包含一个值为零的 activeIndex，它变为传递给第一个 Panel 的 isActive true 值，以及传递给第二个 Panel 的 isActive false 值。">

最初，`Accordion` 的 `activeIndex` 为 `0`，所以第一个 `Panel` 接收到 `isActive = true`

</Diagram>

<Diagram name="sharing_state_parent_clicked" height={385} width={521} alt="与前一个相同的图示，父组件 Accordion 的 activeIndex 值被高亮，表示点击并将值改为一。流向两个子级 Panel 组件的路径也被高亮，并且传递给每个子组件的 isActive 值设为相反：第一个 Panel 为 false，第二个为 true。">

当 `Accordion` 的 `activeIndex` 状态变为 `1` 时，第二个 `Panel` 改为接收到 `isActive = true`

</Diagram>

</DiagramGroup>

<DeepDive>

#### 受控和非受控组件 {/*controlled-and-uncontrolled-components*/}

通常会把带有某些局部状态的组件称为“非受控”组件。例如，最初那个带有 `isActive` 状态变量的 `Panel` 组件就是非受控的，因为它的父组件无法影响面板是否处于活动状态。

相反，当组件中的关键信息由 props 而不是它自己的局部状态驱动时，你可以说它是“受控”的。这样父组件就可以完全指定它的行为。最终版本的 `Panel` 组件带有 `isActive` prop，它由 `Accordion` 组件控制。

非受控组件在父组件中更容易使用，因为它们需要更少的配置。但当你想要把它们协调在一起时，它们的灵活性就较差。受控组件的灵活性最高，但它们要求父组件用 props 完整配置它们。

在实践中，“受控”和“非受控”并不是严格的技术术语——每个组件通常都会同时包含一些局部状态和 props。不过，这是一种很有用的方式，用来讨论组件是如何设计的，以及它们提供了哪些能力。

编写组件时，考虑其中哪些信息应该是受控的（通过 props），哪些信息应该是非受控的（通过 state）。但你总是可以之后改变主意并进行重构。

</DeepDive>

## 每个状态的单一事实来源 {/*a-single-source-of-truth-for-each-state*/}

在 React 应用中，许多组件都会有自己的状态。有些状态可能会“存在”于更接近叶子组件（树底部的组件）的地方，比如输入框。其他状态则可能“存在”于更接近应用顶部的地方。例如，即使是客户端路由库，通常也是通过把当前路由存储在 React 状态中，再通过 props 向下传递来实现的！

**对于每一份独特的状态，你都需要选择一个“拥有”它的组件。** 这一原则也被称为拥有一个[“单一事实来源”。](https://en.wikipedia.org/wiki/Single_source_of_truth) 这并不意味着所有状态都要放在同一个地方——而是指对于_每一份_状态，都有一个_特定的_组件持有这份信息。不要在组件之间重复共享状态，而是把它*提升*到它们共同的父组件中，再通过 props *向下传递*给需要它的子组件。

你的应用会随着开发而变化。在你还在弄清楚每一份状态“存在”在哪里时，把状态向下移动或向上移动是很常见的。这一切都是过程的一部分！

想要通过更多组件来看看这在实践中是什么样子，请阅读[在 React 中思考。](/learn/thinking-in-react)

<Recap>

* 当你想要协调两个组件时，把它们的状态移动到它们的共同父组件中。
* 然后通过 props 从共同父组件向下传递信息。
* 最后，把事件处理函数也向下传递，这样子组件就可以改变父组件的状态。
* 把组件视为“受控的”（由 props 驱动）或“非受控的”（由状态驱动）是很有用的。

</Recap>

<Challenges>

#### 同步输入框 {/*synced-inputs*/}

这两个输入框是彼此独立的。让它们保持同步：编辑其中一个输入框应当用相同的文本更新另一个输入框，反之亦然。

<Hint>

你需要把它们的状态提升到父组件中。

</Hint>

<Sandpack>

```js
import { useState } from 'react';

export default function SyncedInputs() {
  return (
    <>
      <Input label="First input" />
      <Input label="Second input" />
    </>
  );
}

function Input({ label }) {
  const [text, setText] = useState('');

  function handleChange(e) {
    setText(e.target.value);
  }

  return (
    <label>
      {label}
      {' '}
      <input
        value={text}
        onChange={handleChange}
      />
    </label>
  );
}
```

```css
input { margin: 5px; }
label { display: block; }
```

</Sandpack>

<Solution>

把 `text` 状态变量和 `handleChange` 处理函数一起移动到父组件中。然后把它们作为 props 传给两个 `Input` 组件。这样它们就会保持同步。

<Sandpack>

```js
import { useState } from 'react';

export default function SyncedInputs() {
  const [text, setText] = useState('');

  function handleChange(e) {
    setText(e.target.value);
  }

  return (
    <>
      <Input
        label="First input"
        value={text}
        onChange={handleChange}
      />
      <Input
        label="Second input"
        value={text}
        onChange={handleChange}
      />
    </>
  );
}

function Input({ label, value, onChange }) {
  return (
    <label>
      {label}
      {' '}
      <input
        value={value}
        onChange={onChange}
      />
    </label>
  );
}
```

```css
input { margin: 5px; }
label { display: block; }
```

</Sandpack>

</Solution>

#### 过滤列表 {/*filtering-a-list*/}

在这个示例中，`SearchBar` 有自己的 `query` 状态来控制文本输入框。它的父组件 `FilterableList` 显示一个项目 `List`，但并没有考虑搜索查询。

使用 `filterItems(foods, query)` 函数根据搜索查询来过滤列表。要测试你的修改，请确认在输入框中输入 "s" 会把列表筛选为 "Sushi"、"Shish kebab" 和 "Dim sum"。

注意 `filterItems` 已经被实现并导入了，所以你不需要自己编写它！

<Hint>

你需要从 `SearchBar` 中移除 `query` 状态和 `handleChange` 处理函数，并把它们移动到 `FilterableList` 中。然后将它们作为 `query` 和 `onChange` props 传给 `SearchBar`。

</Hint>

<Sandpack>

```js
import { useState } from 'react';
import { foods, filterItems } from './data.js';

export default function FilterableList() {
  return (
    <>
      <SearchBar />
      <hr />
      <List items={foods} />
    </>
  );
}

function SearchBar() {
  const [query, setQuery] = useState('');

  function handleChange(e) {
    setQuery(e.target.value);
  }

  return (
    <label>
      Search:{' '}
      <input
        value={query}
        onChange={handleChange}
      />
    </label>
  );
}

function List({ items }) {
  return (
    <table>
      <tbody>
        {items.map(food => (
          <tr key={food.id}>
            <td>{food.name}</td>
            <td>{food.description}</td>
          </tr>
        ))}
      </tbody>
    </table>
  );
}
```

```js src/data.js
export function filterItems(items, query) {
  query = query.toLowerCase();
  return items.filter(item =>
    item.name.split(' ').some(word =>
      word.toLowerCase().startsWith(query)
    )
  );
}

export const foods = [{
  id: 0,
  name: 'Sushi',
  description: 'Sushi is a traditional Japanese dish of prepared vinegared rice'
}, {
  id: 1,
  name: 'Dal',
  description: 'The most common way of preparing dal is in the form of a soup to which onions, tomatoes and various spices may be added'
}, {
  id: 2,
  name: 'Pierogi',
  description: 'Pierogi are filled dumplings made by wrapping unleavened dough around a savoury or sweet filling and cooking in boiling water'
}, {
  id: 3,
  name: 'Shish kebab',
  description: 'Shish kebab is a popular meal of skewered and grilled cubes of meat.'
}, {
  id: 4,
  name: 'Dim sum',
  description: 'Dim sum is a large range of small dishes that Cantonese people traditionally enjoy in restaurants for breakfast and lunch'
}];
```

</Sandpack>

<Solution>

把 `query` 状态提升到 `FilterableList` 组件中。调用 `filterItems(foods, query)` 获取过滤后的列表，并把它传给 `List`。现在修改查询输入框会反映到列表中：

<Sandpack>

```js
import { useState } from 'react';
import { foods, filterItems } from './data.js';

export default function FilterableList() {
  const [query, setQuery] = useState('');
  const results = filterItems(foods, query);

  function handleChange(e) {
    setQuery(e.target.value);
  }

  return (
    <>
      <SearchBar
        query={query}
        onChange={handleChange}
      />
      <hr />
      <List items={results} />
    </>
  );
}

function SearchBar({ query, onChange }) {
  return (
    <label>
      Search:{' '}
      <input
        value={query}
        onChange={onChange}
      />
    </label>
  );
}

function List({ items }) {
  return (
    <table>
      <tbody>
        {items.map(food => (
          <tr key={food.id}>
            <td>{food.name}</td>
            <td>{food.description}</td>
          </tr>
        ))}
      </tbody>
    </table>
  );
}
```

```js src/data.js
export function filterItems(items, query) {
  query = query.toLowerCase();
  return items.filter(item =>
    item.name.split(' ').some(word =>
      word.toLowerCase().startsWith(query)
    )
  );
}

export const foods = [{
  id: 0,
  name: 'Sushi',
  description: 'Sushi is a traditional Japanese dish of prepared vinegared rice'
}, {
  id: 1,
  name: 'Dal',
  description: 'The most common way of preparing dal is in the form of a soup to which onions, tomatoes and various spices may be added'
}, {
  id: 2,
  name: 'Pierogi',
  description: 'Pierogi are filled dumplings made by wrapping unleavened dough around a savoury or sweet filling and cooking in boiling water'
}, {
  id: 3,
  name: 'Shish kebab',
  description: 'Shish kebab is a popular meal of skewered and grilled cubes of meat.'
}, {
  id: 4,
  name: 'Dim sum',
  description: 'Dim sum is a large range of small dishes that Cantonese people traditionally enjoy in restaurants for breakfast and lunch'
}];
```

</Sandpack>

</Solution>

</Challenges>
