---
title: React 思维方式
---

<Intro>

React 可以改变你思考所见设计和所构建应用的方式。当你使用 React 构建用户界面时，你会先把它拆分成称为 *组件* 的不同部分。然后，你会描述每个组件的不同视觉状态。最后，你会把这些组件连接起来，让数据在它们之间流动。在本教程中，我们将引导你完成使用 React 构建一个可搜索的产品数据表的思考过程。

</Intro>

## 从模拟稿开始 {/*start-with-the-mockup*/}

假设你已经有了一个 JSON API 和设计师提供的模拟稿。

JSON API 返回的数据如下所示：

```json
[
  { category: "Fruits", price: "$1", stocked: true, name: "Apple" },
  { category: "Fruits", price: "$1", stocked: true, name: "Dragonfruit" },
  { category: "Fruits", price: "$2", stocked: false, name: "Passionfruit" },
  { category: "Vegetables", price: "$2", stocked: true, name: "Spinach" },
  { category: "Vegetables", price: "$4", stocked: false, name: "Pumpkin" },
  { category: "Vegetables", price: "$1", stocked: true, name: "Peas" }
]
```

模拟稿看起来像这样：

<img src="/images/docs/s_thinking-in-react_ui.png" width="300" style={{margin: '0 auto'}} />

要在 React 中实现一个 UI，通常会遵循同样的五个步骤。

## 第 1 步：将 UI 拆分成组件层次结构 {/*step-1-break-the-ui-into-a-component-hierarchy*/}

先在模拟稿中的每个组件和子组件周围画出方框，并给它们命名。如果你和设计师协作，他们可能已经在设计工具中为这些组件命名了。问问他们！

根据你的背景不同，你可以用不同的方式来思考如何把一个设计拆分成组件：

* **编程**——使用决定是否应该创建新函数或对象时相同的技巧。其中一个技巧是 [关注点分离](https://en.wikipedia.org/wiki/Separation_of_concerns)，也就是说，一个组件理想情况下只应关注一件事。如果它最终变得很大，就应该被分解成更小的子组件。
* **CSS**——考虑你会为哪些内容创建类选择器。（不过，组件的粒度会稍微粗一些。）
* **设计**——考虑你会如何组织设计的图层。

如果你的 JSON 结构良好，你通常会发现它会自然地映射到 UI 的组件结构。这是因为 UI 和数据模型通常具有相同的信息架构——也就是相同的形状。把你的 UI 拆分成组件，每个组件对应数据模型中的一部分。

这个屏幕上有五个组件：

<FullWidth>

<CodeDiagram flip>

<img src="/images/docs/s_thinking-in-react_ui_outline.png" width="500" style={{margin: '0 auto'}} />

1. `FilterableProductTable`（灰色）包含整个应用。
2. `SearchBar`（蓝色）接收用户输入。
3. `ProductTable`（淡紫色）根据用户输入显示并过滤列表。
4. `ProductCategoryRow`（绿色）为每个类别显示一个标题。
5. `ProductRow`（黄色）为每个产品显示一行。

</CodeDiagram>

</FullWidth>

如果你看 `ProductTable`（淡紫色），你会发现表头（包含 "Name" 和 "Price" 标签）并不是它自己的组件。这取决于个人偏好，两种方式都可以。在这个例子中，它是 `ProductTable` 的一部分，因为它出现在 `ProductTable` 的列表内部。不过，如果这个表头变得更复杂（例如，你添加排序功能），你可以把它移动到自己的 `ProductTableHeader` 组件中。

现在你已经识别出模拟稿中的组件，把它们安排成一个层次结构。模拟稿中出现在另一个组件内部的组件，在层次结构中也应该作为子级出现：

* `FilterableProductTable`
    * `SearchBar`
    * `ProductTable`
        * `ProductCategoryRow`
        * `ProductRow`

## 第 2 步：用 React 构建静态版本 {/*step-2-build-a-static-version-in-react*/}

现在你已经有了组件层次结构，该开始实现你的应用了。最直接的方法是先构建一个能根据数据模型渲染 UI、但还没有任何交互性的版本……先别急！通常先构建静态版本，再添加交互性会更容易。构建静态版本需要大量输入但不需要思考，而添加交互性则需要大量思考但不需要太多输入。

要构建一个能够渲染数据模型的应用静态版本，你需要创建一些 [组件](/learn/your-first-component)，它们复用其他组件，并通过 [props](/learn/passing-props-to-a-component) 传递数据。Props 是一种从父组件向子组件传递数据的方式。（如果你熟悉 [state](/learn/state-a-components-memory) 的概念，那么在构建这个静态版本时完全不要使用 state。State 只用于交互，也就是随时间变化的数据。由于这是应用的静态版本，所以你不需要它。）

你可以选择“自上而下”，从层次结构中更上层的组件开始构建（比如 `FilterableProductTable`）；也可以选择“自下而上”，从更下层的组件开始（比如 `ProductRow`）。在更简单的例子里，通常自上而下更容易；而在更大的项目中，自下而上更容易。

<Sandpack>

```jsx src/App.js
function ProductCategoryRow({ category }) {
  return (
    <tr>
      <th colSpan="2">
        {category}
      </th>
    </tr>
  );
}

function ProductRow({ product }) {
  const name = product.stocked ? product.name :
    <span style={{ color: 'red' }}>
      {product.name}
    </span>;

  return (
    <tr>
      <td>{name}</td>
      <td>{product.price}</td>
    </tr>
  );
}

function ProductTable({ products }) {
  const rows = [];
  let lastCategory = null;

  products.forEach((product) => {
    if (product.category !== lastCategory) {
      rows.push(
        <ProductCategoryRow
          category={product.category}
          key={product.category} />
      );
    }
    rows.push(
      <ProductRow
        product={product}
        key={product.name} />
    );
    lastCategory = product.category;
  });

  return (
    <table>
      <thead>
        <tr>
          <th>Name</th>
          <th>Price</th>
        </tr>
      </thead>
      <tbody>{rows}</tbody>
    </table>
  );
}

function SearchBar() {
  return (
    <form>
      <input type="text" placeholder="搜索..." />
      <label>
        <input type="checkbox" />
        {' '}
        只显示有库存的产品
      </label>
    </form>
  );
}

function FilterableProductTable({ products }) {
  return (
    <div>
      <SearchBar />
      <ProductTable products={products} />
    </div>
  );
}

const PRODUCTS = [
  {category: "Fruits", price: "$1", stocked: true, name: "Apple"},
  {category: "Fruits", price: "$1", stocked: true, name: "Dragonfruit"},
  {category: "Fruits", price: "$2", stocked: false, name: "Passionfruit"},
  {category: "Vegetables", price: "$2", stocked: true, name: "Spinach"},
  {category: "Vegetables", price: "$4", stocked: false, name: "Pumpkin"},
  {category: "Vegetables", price: "$1", stocked: true, name: "Peas"}
];

export default function App() {
  return <FilterableProductTable products={PRODUCTS} />;
}
```

```css
body {
  padding: 5px
}
label {
  display: block;
  margin-top: 5px;
  margin-bottom: 5px;
}
th {
  padding-top: 10px;
}
td {
  padding: 2px;
  padding-right: 40px;
}
```

</Sandpack>

（如果这段代码看起来很吓人，先去看一下 [快速开始](/learn/)！）

在构建完组件之后，你会拥有一个可复用组件库，用于渲染你的数据模型。由于这是一个静态应用，组件只会返回 JSX。层次结构顶部的组件（`FilterableProductTable`）会把你的数据模型作为 prop 接收。这被称为 _单向数据流_，因为数据会从顶层组件向下流动到树底部的组件。

<Pitfall>

此时，你不应该使用任何 state 值。那是下一步的内容！

</Pitfall>

## 第 3 步：找出 UI state 的最小但完整表示 {/*step-3-find-the-minimal-but-complete-representation-of-ui-state*/}

要让 UI 具备交互性，你需要让用户能够修改底层数据模型。为此你将使用 *state*。

把 state 想成应用需要记住的、会变化的数据的最小集合。组织 state 最重要的原则是保持 [DRY（不要重复自己）](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself)。找出应用真正需要的 state 的绝对最小表示，然后按需计算其他所有内容。例如，如果你在构建一个购物清单，你可以把条目作为 state 中的数组存储。如果你还想显示清单中的条目数量，不要再把数量存成另一个 state 值——而是读取数组的长度。

现在来思考这个示例应用中的所有数据：

1. 原始的产品列表
2. 用户输入的搜索文本
3. 复选框的值
4. 过滤后的产品列表

这些里面哪些是 state？先找出那些不是 state 的：

* 它是否会随着时间**保持不变**？如果是，那它就不是 state。
* 它是否通过 props 从父组件**传入**？如果是，那它就不是 state。
* 你是否可以基于组件中已有的 state 或 props **计算出它**？如果可以，那它**肯定**不是 state！

剩下的，可能就是 state。

让我们再逐个看一遍：

1. 原始的产品列表是**通过 props 传入的，所以它不是 state。**
2. 搜索文本看起来是 state，因为它会随时间变化，而且无法由其他内容计算得出。
3. 复选框的值看起来是 state，因为它会随时间变化，而且无法由其他内容计算得出。
4. 过滤后的产品列表**不是 state，因为它可以通过**拿原始产品列表并根据搜索文本和复选框的值进行过滤来计算得到。

这意味着只有搜索文本和复选框的值是 state！做得不错！

<DeepDive>

#### Props 与 State {/*props-vs-state*/}

React 中有两种“模型”数据：props 和 state。它们非常不同：

* [**Props** 就像你传递给函数的参数。](/learn/passing-props-to-a-component) 它们让父组件可以向子组件传递数据并定制其外观。例如，`Form` 可以把 `color` prop 传给 `Button`。
* [**State** 就像组件的记忆。](/learn/state-a-components-memory) 它让组件能够跟踪一些信息，并在交互发生时更新它。例如，`Button` 可能会跟踪 `isHovered` state。

Props 和 state 是不同的，但它们会协同工作。父组件通常会把一些信息保存在 state 中（以便能够修改它），然后把它们作为 props *向下传递* 给子组件。第一次阅读时，如果你仍然觉得这个区别有些模糊，也没关系。要真正掌握它，需要一些练习！

</DeepDive>

## 第 4 步：确定你的状态应该放在哪里 {/*step-4-identify-where-your-state-should-live*/}

在确定了应用中的最小状态数据之后，你需要找出由哪个组件负责改变这个状态，或者说，哪个组件“拥有”这个状态。记住：React 使用单向数据流，数据会沿着组件层级从父组件传递到子组件。起初并不一定能立刻看出某个状态应该归哪个组件所有。如果你是第一次接触这个概念，这可能会有点难，但你可以按照以下步骤来找出来！

对于应用中的每一份状态：

1. 找出*每一个*基于这份状态进行渲染的组件。
2. 找到它们最近的共同父组件——也就是层级中位于它们之上的那个组件。
3. 决定状态应该放在哪里：
    1. 通常，你可以把状态直接放到它们的共同父组件中。
    2. 你也可以把状态放到某个位于共同父组件之上的组件中。
    3. 如果你找不到一个合适的组件来拥有这份状态，那就新建一个只用于保存状态的组件，并把它添加到层级中位于共同父组件之上的某个位置。

在上一步中，你找到了这个应用中的两份状态：搜索输入文本，以及复选框的值。在这个示例中，它们总是一起出现，所以把它们放在同一个地方是合理的。

现在让我们按这个策略来处理它们：

1. **找出使用状态的组件：**
    * `ProductTable` 需要根据这份状态（搜索文本和复选框值）过滤产品列表。
    * `SearchBar` 需要显示这份状态（搜索文本和复选框值）。
2. **找出它们的共同父组件：** 两个组件共享的第一个父组件是 `FilterableProductTable`。
3. **决定状态放在哪里**：我们将把过滤文本和已勾选状态的值保存在 `FilterableProductTable` 中。

所以，这些状态值将位于 `FilterableProductTable` 中。

使用 [`useState()` Hook.](/reference/react/useState) 为组件添加状态。Hook 是一种特殊函数，可以让你“接入” React。把两个状态变量添加到 `FilterableProductTable` 顶部，并指定它们的初始状态：

```js
function FilterableProductTable({ products }) {
  const [filterText, setFilterText] = useState('');
  const [inStockOnly, setInStockOnly] = useState(false);
```

然后，将 `filterText` 和 `inStockOnly` 作为 props 传递给 `ProductTable` 和 `SearchBar`：

```js
<div>
  <SearchBar
    filterText={filterText}
    inStockOnly={inStockOnly} />
  <ProductTable
    products={products}
    filterText={filterText}
    inStockOnly={inStockOnly} />
</div>
```

你现在可以开始看到应用将如何运行了。把下面沙盒代码中的 `filterText` 初始值从 `useState('')` 改为 `useState('fruit')`。你会看到搜索输入文本和表格都会更新：

<Sandpack>

```jsx src/App.js
import { useState } from 'react';

function FilterableProductTable({ products }) {
  const [filterText, setFilterText] = useState('');
  const [inStockOnly, setInStockOnly] = useState(false);

  return (
    <div>
      <SearchBar
        filterText={filterText}
        inStockOnly={inStockOnly} />
      <ProductTable
        products={products}
        filterText={filterText}
        inStockOnly={inStockOnly} />
    </div>
  );
}

function ProductCategoryRow({ category }) {
  return (
    <tr>
      <th colSpan="2">
        {category}
      </th>
    </tr>
  );
}

function ProductRow({ product }) {
  const name = product.stocked ? product.name :
    <span style={{ color: 'red' }}>
      {product.name}
    </span>;

  return (
    <tr>
      <td>{name}</td>
      <td>{product.price}</td>
    </tr>
  );
}

function ProductTable({ products, filterText, inStockOnly }) {
  const rows = [];
  let lastCategory = null;

  products.forEach((product) => {
    if (
      product.name.toLowerCase().indexOf(
        filterText.toLowerCase()
      ) === -1
    ) {
      return;
    }
    if (inStockOnly && !product.stocked) {
      return;
    }
    if (product.category !== lastCategory) {
      rows.push(
        <ProductCategoryRow
          category={product.category}
          key={product.category} />
      );
    }
    rows.push(
      <ProductRow
        product={product}
        key={product.name} />
    );
    lastCategory = product.category;
  });

  return (
    <table>
      <thead>
        <tr>
          <th>Name</th>
          <th>Price</th>
        </tr>
      </thead>
      <tbody>{rows}</tbody>
    </table>
  );
}

function SearchBar({ filterText, inStockOnly }) {
  return (
    <form>
      <input
        type="text"
        value={filterText}
        placeholder="Search..."/>
      <label>
        <input
          type="checkbox"
          checked={inStockOnly} />
        {' '}
        仅显示有库存的产品
      </label>
    </form>
  );
}

const PRODUCTS = [
  {category: "Fruits", price: "$1", stocked: true, name: "Apple"},
  {category: "Fruits", price: "$1", stocked: true, name: "Dragonfruit"},
  {category: "Fruits", price: "$2", stocked: false, name: "Passionfruit"},
  {category: "Vegetables", price: "$2", stocked: true, name: "Spinach"},
  {category: "Vegetables", price: "$4", stocked: false, name: "Pumpkin"},
  {category: "Vegetables", price: "$1", stocked: true, name: "Peas"}
];

export default function App() {
  return <FilterableProductTable products={PRODUCTS} />;
}
```

```css
body {
  padding: 5px
}
label {
  display: block;
  margin-top: 5px;
  margin-bottom: 5px;
}
th {
  padding-top: 5px;
}
td {
  padding: 2px;
}
```

</Sandpack>

注意，编辑表单目前还不能工作。上面的沙盒中有一个控制台错误，解释了原因：

<ConsoleBlock level="error">

你向表单字段提供了一个 `value` 属性，但没有提供 `onChange` 处理函数。这会将其渲染为只读字段。

</ConsoleBlock>

在上面的沙盒中，`ProductTable` 和 `SearchBar` 读取 `filterText` 和 `inStockOnly` props 来渲染表格、输入框和复选框。例如，下面是 `SearchBar` 如何填充输入框值的：

```js {1,6}
function SearchBar({ filterText, inStockOnly }) {
  return (
    <form>
      <input
        type="text"
        value={filterText}
        placeholder="Search..."/>
```

不过，你还没有添加任何代码来响应用户操作，比如输入内容。下一步我们就要完成这个。

## 第 5 步：添加反向数据流 {/*step-5-add-inverse-data-flow*/}

目前，你的应用已经可以通过 props 和 state 沿着层级向下流动来正确渲染。但要根据用户输入来改变 state，你还需要支持数据向相反方向流动：层级深处的表单组件需要更新 `FilterableProductTable` 中的 state。

React 让这种数据流显式化，但这比双向数据绑定多写一点代码。如果你尝试在上面的示例中输入内容或勾选复选框，你会发现 React 忽略了你的输入。这是有意为之。通过编写 `<input value={filterText} />`，你把 `input` 的 `value` prop 设置为始终等于从 `FilterableProductTable` 传入的 `filterText` state。由于 `filterText` state 从未被设置，输入框也就永远不会改变。

你希望做到：只要用户更改表单输入，state 就会更新以反映这些变化。这个 state 由 `FilterableProductTable` 拥有，所以只有它才能调用 `setFilterText` 和 `setInStockOnly`。为了让 `SearchBar` 能更新 `FilterableProductTable` 的 state，你需要把这些函数向下传递给 `SearchBar`：

```js {2,3,10,11}
function FilterableProductTable({ products }) {
  const [filterText, setFilterText] = useState('');
  const [inStockOnly, setInStockOnly] = useState(false);

  return (
    <div>
      <SearchBar
        filterText={filterText}
        inStockOnly={inStockOnly}
        onFilterTextChange={setFilterText}
        onInStockOnlyChange={setInStockOnly} />
```

在 `SearchBar` 内部，你要添加 `onChange` 事件处理函数，并据此设置父组件的 state：

```js {4,5,13,19}
function SearchBar({
  filterText,
  inStockOnly,
  onFilterTextChange,
  onInStockOnlyChange
}) {
  return (
    <form>
      <input
        type="text"
        value={filterText}
        placeholder="Search..."
        onChange={(e) => onFilterTextChange(e.target.value)}
      />
      <label>
        <input
          type="checkbox"
          checked={inStockOnly}
          onChange={(e) => onInStockOnlyChange(e.target.checked)}
```

现在这个应用完全可用了！

<Sandpack>

```jsx src/App.js
import { useState } from 'react';

function FilterableProductTable({ products }) {
  const [filterText, setFilterText] = useState('');
  const [inStockOnly, setInStockOnly] = useState(false);

  return (
    <div>
      <SearchBar
        filterText={filterText}
        inStockOnly={inStockOnly}
        onFilterTextChange={setFilterText}
        onInStockOnlyChange={setInStockOnly} />
      <ProductTable
        products={products}
        filterText={filterText}
        inStockOnly={inStockOnly} />
    </div>
  );
}

function ProductCategoryRow({ category }) {
  return (
    <tr>
      <th colSpan="2">
        {category}
      </th>
    </tr>
  );
}

function ProductRow({ product }) {
  const name = product.stocked ? product.name :
    <span style={{ color: 'red' }}>
      {product.name}
    </span>;

  return (
    <tr>
      <td>{name}</td>
      <td>{product.price}</td>
    </tr>
  );
}

function ProductTable({ products, filterText, inStockOnly }) {
  const rows = [];
  let lastCategory = null;

  products.forEach((product) => {
    if (
      product.name.toLowerCase().indexOf(
        filterText.toLowerCase()
      ) === -1
    ) {
      return;
    }
    if (inStockOnly && !product.stocked) {
      return;
    }
    if (product.category !== lastCategory) {
      rows.push(
        <ProductCategoryRow
          category={product.category}
          key={product.category} />
      );
    }
    rows.push(
      <ProductRow
        product={product}
        key={product.name} />
    );
    lastCategory = product.category;
  });

  return (
    <table>
      <thead>
        <tr>
          <th>Name</th>
          <th>Price</th>
        </tr>
      </thead>
      <tbody>{rows}</tbody>
    </table>
  );
}

function SearchBar({
  filterText,
  inStockOnly,
  onFilterTextChange,
  onInStockOnlyChange
}) {
  return (
    <form>
      <input
        type="text"
        value={filterText} placeholder="Search..."
        onChange={(e) => onFilterTextChange(e.target.value)} />
      <label>
        <input
          type="checkbox"
          checked={inStockOnly}
          onChange={(e) => onInStockOnlyChange(e.target.checked)} />
        {' '}
        仅显示有库存的产品
      </label>
    </form>
  );
}

const PRODUCTS = [
  {category: "Fruits", price: "$1", stocked: true, name: "Apple"},
  {category: "Fruits", price: "$1", stocked: true, name: "Dragonfruit"},
  {category: "Fruits", price: "$2", stocked: false, name: "Passionfruit"},
  {category: "Vegetables", price: "$2", stocked: true, name: "Spinach"},
  {category: "Vegetables", price: "$4", stocked: false, name: "Pumpkin"},
  {category: "Vegetables", price: "$1", stocked: true, name: "Peas"}
];

export default function App() {
  return <FilterableProductTable products={PRODUCTS} />;
}
```

```css
body {
  padding: 5px
}
label {
  display: block;
  margin-top: 5px;
  margin-bottom: 5px;
}
th {
  padding: 4px;
}
td {
  padding: 2px;
}
```

</Sandpack>

你可以在[添加交互性](/learn/adding-interactivity)部分了解更多关于处理事件和更新 state 的内容。

## 接下来该去哪里 {/*where-to-go-from-here*/}

这是对如何使用 React 构建组件和应用程序的一次非常简短的介绍。你现在就可以[开始一个 React 项目](/learn/installation)，或者[更深入地了解本教程中使用的所有语法](/learn/describing-the-ui)。
