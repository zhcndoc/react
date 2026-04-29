---
title: 条件渲染
---

<Intro>

你的组件通常需要根据不同的条件显示不同的内容。在 React 中，你可以使用 JavaScript 语法（如 `if` 语句、`&&` 和 `? :` 运算符）有条件地渲染 JSX。

</Intro>

<YouWillLearn>

* 如何根据条件返回不同的 JSX
* 如何有条件地包含或排除一段 JSX
* 在 React 代码库中常见的条件语法简写

</YouWillLearn>

## 有条件地返回 JSX {/*conditionally-returning-jsx*/}

假设你有一个 `PackingList` 组件，它渲染多个 `Item`，这些项可以被标记为已打包或未打包：

<Sandpack>

```js
function Item({ name, isPacked }) {
  return <li className="item">{name}</li>;
}

export default function PackingList() {
  return (
    <section>
      <h1>Sally Ride 的打包清单</h1>
      <ul>
        <Item
          isPacked={true}
          name="太空服"
        />
        <Item
          isPacked={true}
          name="带金色叶饰的头盔"
        />
        <Item
          isPacked={false}
          name="Tam 的照片"
        />
      </ul>
    </section>
  );
}
```

</Sandpack>

注意到一些 `Item` 组件的 `isPacked` 属性设置为 `true` 而不是 `false`。如果 `isPacked={true}`，你希望给已打包的项目添加一个对勾（✅）。

你可以像这样把它写成一个 [`if`/`else` 语句](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/if...else)：

```js
if (isPacked) {
  return <li className="item">{name} ✅</li>;
}
return <li className="item">{name}</li>;
```

如果 `isPacked` 属性为 `true`，这段代码**会返回不同的 JSX 树。**通过这个改动，一些项目的末尾会出现一个对勾：

<Sandpack>

```js
function Item({ name, isPacked }) {
  if (isPacked) {
    return <li className="item">{name} ✅</li>;
  }
  return <li className="item">{name}</li>;
}

export default function PackingList() {
  return (
    <section>
      <h1>Sally Ride 的打包清单</h1>
      <ul>
        <Item
          isPacked={true}
          name="太空服"
        />
        <Item
          isPacked={true}
          name="带金色叶饰的头盔"
        />
        <Item
          isPacked={false}
          name="Tam 的照片"
        />
      </ul>
    </section>
  );
}
```

</Sandpack>

试着分别编辑两种情况下返回的内容，看看结果如何变化！

注意你是如何使用 JavaScript 的 `if` 和 `return` 语句创建分支逻辑的。在 React 中，控制流（如条件）由 JavaScript 处理。

### 使用 `null` 有条件地返回空内容 {/*conditionally-returning-nothing-with-null*/}

在某些情况下，你完全不想渲染任何内容。例如，假设你不想显示已打包的项目。组件必须返回某些内容。在这种情况下，你可以返回 `null`：

```js
if (isPacked) {
  return null;
}
return <li className="item">{name}</li>;
```

如果 `isPacked` 为 true，组件将什么都不返回，返回 `null`。否则，它会返回用于渲染的 JSX。

<Sandpack>

```js
function Item({ name, isPacked }) {
  if (isPacked) {
    return null;
  }
  return <li className="item">{name}</li>;
}

export default function PackingList() {
  return (
    <section>
      <h1>Sally Ride 的打包清单</h1>
      <ul>
        <Item
          isPacked={true}
          name="太空服"
        />
        <Item
          isPacked={true}
          name="带金色叶饰的头盔"
        />
        <Item
          isPacked={false}
          name="Tam 的照片"
        />
      </ul>
    </section>
  );
}
```

</Sandpack>

在实践中，从组件中返回 `null` 并不常见，因为这可能会让尝试渲染它的开发者感到意外。更多时候，你会在父组件的 JSX 中有条件地包含或排除该组件。下面就是这样做的方法！

## 有条件地包含 JSX {/*conditionally-including-jsx*/}

在前面的例子中，你控制了组件会返回哪一个（如果有的话！）JSX 树。你可能已经注意到渲染输出中有一些重复：

```js
<li className="item">{name} ✅</li>
```

与下面这一段非常相似：

```js
<li className="item">{name}</li>
```

这两个条件分支都返回 `<li className="item">...</li>`：

```js
if (isPacked) {
  return <li className="item">{name} ✅</li>;
}
return <li className="item">{name}</li>;
```

虽然这种重复并无害处，但它会让代码更难维护。如果你想修改 `className` 呢？你就得在代码里两个地方都改！在这种情况下，你可以有条件地插入一点 JSX，让代码更 [DRY。](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself)

### 条件（三元）运算符 (`? :`) {/*conditional-ternary-operator--*/}

JavaScript 有一种紧凑的语法用于编写条件表达式——[条件运算符](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Conditional_Operator)或“三元运算符”。

与其写：

```js
if (isPacked) {
  return <li className="item">{name} ✅</li>;
}
return <li className="item">{name}</li>;
```

你可以写成：

```js
return (
  <li className="item">
    {isPacked ? name + ' ✅' : name}
  </li>
);
```

你可以把它读作：*“如果 `isPacked` 为 true，那么（`?`）渲染 `name + ' ✅'`，否则（`:`）渲染 `name`”*。

<DeepDive>

#### 这两个例子完全等价吗？ {/*are-these-two-examples-fully-equivalent*/}

如果你来自面向对象编程背景，你可能会认为上面两个例子有细微差别，因为其中一个可能创建两个不同的 `<li>` “实例”。但 JSX 元素并不是“实例”，因为它们不保存任何内部状态，也不是真正的 DOM 节点。它们只是轻量级的描述，就像蓝图一样。所以这两个例子实际上*完全等价*。[保留和重置状态](/learn/preserving-and-resetting-state) 会详细解释它是如何工作的。

</DeepDive>

现在假设你想把已完成项目的文本包裹到另一个 HTML 标签中，比如用 `<del>` 来表示删除线。你可以增加更多换行和括号，以便更容易在每种情况下嵌套更多 JSX：

<Sandpack>

```js
function Item({ name, isPacked }) {
  return (
    <li className="item">
      {isPacked ? (
        <del>
          {name + ' ✅'}
        </del>
      ) : (
        name
      )}
    </li>
  );
}

export default function PackingList() {
  return (
    <section>
      <h1>Sally Ride 的打包清单</h1>
      <ul>
        <Item
          isPacked={true}
          name="太空服"
        />
        <Item
          isPacked={true}
          name="带金色叶饰的头盔"
        />
        <Item
          isPacked={false}
          name="Tam 的照片"
        />
      </ul>
    </section>
  );
}
```

</Sandpack>

这种风格适用于简单条件，但要适度使用。如果你的组件因为嵌套太多条件标记而变得混乱，可以考虑提取子组件来整理代码。在 React 中，标记是代码的一部分，所以你可以使用变量和函数等工具来整理复杂表达式。

### 逻辑与运算符 (`&&`) {/*logical-and-operator-*/}

你还会经常遇到另一种常见简写：[JavaScript 逻辑与（`&&`）运算符。](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Logical_AND#:~:text=The%20logical%20AND%20(%20%26%26%20)%20operator,it%20returns%20a%20Boolean%20value.) 在 React 组件中，当你想在条件为 true 时渲染一些 JSX，**否则什么都不渲染**，它就经常出现。使用 `&&`，你可以仅在 `isPacked` 为 `true` 时有条件地渲染对勾：

```js
return (
  <li className="item">
    {name} {isPacked && '✅'}
  </li>
);
```

你可以把它读作：*“如果 `isPacked` 为真，那么（`&&`）渲染对勾，否则什么也不渲染”*。

下面是实际效果：

<Sandpack>

```js
function Item({ name, isPacked }) {
  return (
    <li className="item">
      {name} {isPacked && '✅'}
    </li>
  );
}

export default function PackingList() {
  return (
    <section>
      <h1>Sally Ride 的打包清单</h1>
      <ul>
        <Item
          isPacked={true}
          name="太空服"
        />
        <Item
          isPacked={true}
          name="带金色叶饰的头盔"
        />
        <Item
          isPacked={false}
          name="Tam 的照片"
        />
      </ul>
    </section>
  );
}
```

</Sandpack>

如果左侧（我们的条件）为 `true`，[JavaScript 的 `&&` 表达式](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Logical_AND)会返回右侧的值（在我们的例子中是对勾）。但如果条件为 `false`，整个表达式就会变成 `false`。React 会把 `false` 视为 JSX 树中的一个“空洞”，就像 `null` 或 `undefined` 一样，并且不会在其位置渲染任何内容。


<Pitfall>

**不要把数字放在 `&&` 的左侧。**

为了测试条件，JavaScript 会自动把左侧转换为布尔值。然而，如果左侧是 `0`，那么整个表达式就会得到这个值（`0`），而 React 会愉快地渲染 `0`，而不是空内容。

例如，一个常见错误是写出 `messageCount && <p>New messages</p>` 这样的代码。很容易以为当 `messageCount` 是 `0` 时它什么也不会渲染，但实际上它会把 `0` 本身渲染出来！

要修复它，请让左侧变成布尔值：`messageCount > 0 && <p>New messages</p>`。

</Pitfall>

### 将 JSX 有条件地赋值给变量 {/*conditionally-assigning-jsx-to-a-variable*/}

当这些简写妨碍你编写普通代码时，试着使用 `if` 语句和变量。你可以重新给用 [`let`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/let) 定义的变量赋值，所以先提供你想显示的默认内容，也就是 name：

```js
let itemContent = name;
```

如果 `isPacked` 为 `true`，使用 `if` 语句将一个 JSX 表达式重新赋给 `itemContent`：

```js
if (isPacked) {
  itemContent = name + " ✅";
}
```

[花括号打开了“通往 JavaScript 的窗口”。](/learn/javascript-in-jsx-with-curly-braces#using-curly-braces-a-window-into-the-javascript-world) 使用花括号将变量嵌入返回的 JSX 树中，把之前计算好的表达式嵌套到 JSX 里面：

```js
<li className="item">
  {itemContent}
</li>
```

这种风格最为冗长，但也最灵活。下面是实际效果：

<Sandpack>

```js
function Item({ name, isPacked }) {
  let itemContent = name;
  if (isPacked) {
    itemContent = name + " ✅";
  }
  return (
    <li className="item">
      {itemContent}
    </li>
  );
}

export default function PackingList() {
  return (
    <section>
      <h1>Sally Ride 的打包清单</h1>
      <ul>
        <Item
          isPacked={true}
          name="太空服"
        />
        <Item
          isPacked={true}
          name="带金色叶饰的头盔"
        />
        <Item
          isPacked={false}
          name="Tam 的照片"
        />
      </ul>
    </section>
  );
}
```

</Sandpack>

和前面一样，这不仅适用于文本，也适用于任意 JSX：

<Sandpack>

```js
function Item({ name, isPacked }) {
  let itemContent = name;
  if (isPacked) {
    itemContent = (
      <del>
        {name + " ✅"}
      </del>
    );
  }
  return (
    <li className="item">
      {itemContent}
    </li>
  );
}

export default function PackingList() {
  return (
    <section>
      <h1>Sally Ride 的打包清单</h1>
      <ul>
        <Item
          isPacked={true}
          name="太空服"
        />
        <Item
          isPacked={true}
          name="带金色叶饰的头盔"
        />
        <Item
          isPacked={false}
          name="Tam 的照片"
        />
      </ul>
    </section>
  );
}
```

</Sandpack>

如果你不熟悉 JavaScript，这些风格一开始可能会让你觉得有些难以招架。不过，学习它们会帮助你阅读和编写任何 JavaScript 代码——不只是 React 组件！先选择一种你喜欢的方式入门，如果你忘了其他方式的工作原理，再回来看这份参考。

<Recap>

* 在 React 中，你使用 JavaScript 来控制分支逻辑。
* 你可以使用 `if` 语句有条件地返回一个 JSX 表达式。
* 你可以有条件地把一些 JSX 保存到变量中，然后通过花括号将其包含在其他 JSX 中。
* 在 JSX 中，`{cond ? <A /> : <B />}` 表示 *“如果 `cond`，渲染 `<A />`，否则渲染 `<B />`”*。
* 在 JSX 中，`{cond && <A />}` 表示 *“如果 `cond`，渲染 `<A />`，否则什么也不渲染”*。
* 这些简写很常见，但如果你更喜欢普通的 `if`，也完全可以不用它们。

</Recap>



<Challenges>

#### 对不完整项目显示图标，使用 `? :` {/*show-an-icon-for-incomplete-items-with--*/}

使用条件运算符（`cond ? a : b`）在 `isPacked` 不是 `true` 时渲染一个 ❌。

<Sandpack>

```js
function Item({ name, isPacked }) {
  return (
    <li className="item">
      {name} {isPacked && '✅'}
    </li>
  );
}

export default function PackingList() {
  return (
    <section>
      <h1>Sally Ride 的打包清单</h1>
      <ul>
        <Item
          isPacked={true}
          name="太空服"
        />
        <Item
          isPacked={true}
          name="带金色叶饰的头盔"
        />
        <Item
          isPacked={false}
          name="Tam 的照片"
        />
      </ul>
    </section>
  );
}
```

</Sandpack>

<Solution>

<Sandpack>

```js
function Item({ name, isPacked }) {
  return (
    <li className="item">
      {name} {isPacked ? '✅' : '❌'}
    </li>
  );
}

export default function PackingList() {
  return (
    <section>
      <h1>Sally Ride 的打包清单</h1>
      <ul>
        <Item
          isPacked={true}
          name="太空服"
        />
        <Item
          isPacked={true}
          name="带金色叶饰的头盔"
        />
        <Item
          isPacked={false}
          name="Tam 的照片"
        />
      </ul>
    </section>
  );
}
```

</Sandpack>

</Solution>

#### 使用 `&&` 显示项目的重要性 {/*show-the-item-importance-with-*/}

在这个例子中，每个 `Item` 都接收一个数值型的 `importance` 属性。使用 `&&` 运算符以斜体渲染 "_(重要性：X)_"，但只对重要性非零的项目显示。你的项目列表最终应如下所示：

* 太空服 _(重要性：9)_
* 带金色叶饰的头盔
* Tam 的照片 _(重要性：6)_

别忘了在两个标签之间加一个空格！

<Sandpack>

```js
function Item({ name, importance }) {
  return (
    <li className="item">
      {name}
    </li>
  );
}

export default function PackingList() {
  return (
    <section>
      <h1>Sally Ride 的打包清单</h1>
      <ul>
        <Item
          importance={9}
          name="太空服"
        />
        <Item
          importance={0}
          name="带金色叶饰的头盔"
        />
        <Item
          importance={6}
          name="Tam 的照片"
        />
      </ul>
    </section>
  );
}
```

</Sandpack>

<Solution>

这应该能解决问题：

<Sandpack>

```js
function Item({ name, importance }) {
  return (
    <li className="item">
      {name}
      {importance > 0 && ' '}
      {importance > 0 &&
        <i>(重要性: {importance})</i>
      }
    </li>
  );
}

export default function PackingList() {
  return (
    <section>
      <h1>Sally Ride 的打包清单</h1>
      <ul>
        <Item
          importance={9}
          name="太空服"
        />
        <Item
          importance={0}
          name="带金色叶饰的头盔"
        />
        <Item
          importance={6}
          name="Tam 的照片"
        />
      </ul>
    </section>
  );
}
```

</Sandpack>

注意你必须写 `importance > 0 && ...` 而不是 `importance && ...`，这样当 `importance` 为 `0` 时，就不会把 `0` 作为结果渲染出来！

在这个解决方案中，使用了两个独立的条件来在名称和重要性标签之间插入空格。或者，你也可以使用带前导空格的 Fragment：`importance > 0 && <> <i>...</i></>`，或者在 `<i>` 内部立即添加一个空格：`importance > 0 && <i> ...</i>`。

</Solution>

#### 将一系列 `? :` 重构为 `if` 和变量 {/*refactor-a-series-of---to-if-and-variables*/}

这个 `Drink` 组件使用一系列 `? :` 条件来根据 `name` 属性是 `"tea"` 还是 `"coffee"` 显示不同信息。问题在于，每种饮料的信息分散在多个条件里。将这段代码重构为使用一个 `if` 语句，而不是三个 `? :` 条件。

<Sandpack>

```js
function Drink({ name }) {
  return (
    <section>
      <h1>{name}</h1>
      <dl>
        <dt>植物部分</dt>
        <dd>{name === 'tea' ? '叶子' : '豆子'}</dd>
        <dt>咖啡因含量</dt>
        <dd>{name === 'tea' ? '15–70 毫克/杯' : '80–185 毫克/杯'}</dd>
        <dt>年龄</dt>
        <dd>{name === 'tea' ? '4,000 多年' : '1,000 多年'}</dd>
      </dl>
    </section>
  );
}

export default function DrinkList() {
  return (
    <div>
      <Drink name="tea" />
      <Drink name="coffee" />
    </div>
  );
}
```

</Sandpack>

当你把代码重构为使用 `if` 之后，你还有其他简化它的想法吗？

<Solution>

有多种方式可以实现，下面是一个起点：

<Sandpack>

```js
function Drink({ name }) {
  let part, caffeine, age;
  if (name === 'tea') {
    part = 'leaf';
    caffeine = '15–70 mg/cup';
    age = '4,000+ years';
  } else if (name === 'coffee') {
    part = 'bean';
    caffeine = '80–185 mg/cup';
    age = '1,000+ years';
  }
  return (
    <section>
      <h1>{name}</h1>
      <dl>
        <dt>植物部分</dt>
        <dd>{part}</dd>
        <dt>咖啡因含量</dt>
        <dd>{caffeine}</dd>
        <dt>年龄</dt>
        <dd>{age}</dd>
      </dl>
    </section>
  );
}

export default function DrinkList() {
  return (
    <div>
      <Drink name="tea" />
      <Drink name="coffee" />
    </div>
  );
}
```

</Sandpack>

这里每种饮料的信息被组合在一起，而不是分散在多个条件中。这让将来添加更多饮料变得更容易。

另一种解决方案是把信息移到对象中，从而完全去掉条件判断：

<Sandpack>

```js
const drinks = {
  tea: {
    part: 'leaf',
    caffeine: '15–70 mg/cup',
    age: '4,000+ years'
  },
  coffee: {
    part: 'bean',
    caffeine: '80–185 mg/cup',
    age: '1,000+ years'
  }
};

function Drink({ name }) {
  const info = drinks[name];
  return (
    <section>
      <h1>{name}</h1>
      <dl>
        <dt>植物部分</dt>
        <dd>{info.part}</dd>
        <dt>咖啡因含量</dt>
        <dd>{info.caffeine}</dd>
        <dt>年龄</dt>
        <dd>{info.age}</dd>
      </dl>
    </section>
  );
}

export default function DrinkList() {
  return (
    <div>
      <Drink name="tea" />
      <Drink name="coffee" />
    </div>
  );
}
```

</Sandpack>

</Solution>

</Challenges>
