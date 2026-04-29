---
title: 使用 Context 深层传递数据
---

<Intro>

通常，你会通过 props 将信息从父组件传递给子组件。但如果你必须把这些 props 一层层传给中间的许多组件，或者应用中的许多组件都需要相同的信息，那么传递 props 就会变得冗长且不方便。*Context* 可以让父组件把某些信息提供给它下面树中的任意组件——无论多深——而无需通过 props 显式传递。

</Intro>

<YouWillLearn>

- 什么是“prop drilling”
- 如何用 context 替代重复的 props 传递
- context 的常见使用场景
- context 的常见替代方案

</YouWillLearn>

## 传递 props 的问题 {/*the-problem-with-passing-props*/}

[传递 props](/learn/passing-props-to-a-component) 是一种很好的方式，可以显式地将数据通过 UI 树传递给使用它的组件。

但是，当你需要把某个 prop 很深地传递到树中，或者很多组件都需要同一个 prop 时，传递 props 就会变得冗长且不方便。最近的公共祖先可能离需要数据的组件很远，而把 state [提升到上层](/learn/sharing-state-between-components) 这么高的位置，可能会导致一种叫做“prop drilling”的情况。

<DiagramGroup>

<Diagram name="passing_data_lifting_state" height={160} width={608} captionPosition="top" alt="一棵包含三个组件的树的示意图。父组件包含一个表示值的气泡，气泡以紫色高亮。该值向下流向两个子组件，两个子组件也都以紫色高亮。" >

提升 state

</Diagram>
<Diagram name="passing_data_prop_drilling" height={430} width={608} captionPosition="top" alt="一棵包含十个节点的树的示意图，每个节点最多有两个子节点。根节点包含一个表示值的气泡，气泡以紫色高亮。该值沿着两个子节点向下传递，这两个子节点都传递了该值但并不拥有它。左侧子节点将该值传递给它的两个子节点，这两个子节点都以紫色高亮。根节点的右侧子节点将该值传递给它的两个子节点中的一个——右边那个，它以紫色高亮。那个子节点又把该值传给它唯一的子节点，而它的子节点又把该值传递给它的两个子节点，这两个子节点都以紫色高亮。">

Prop drilling

</Diagram>

</DiagramGroup>

如果有一种方法可以把数据“传送”给树中需要它的组件，而不用传递 props，那不是很好吗？React 的 context 功能就可以做到！

## Context：替代 props 传递的一种方式 {/*context-an-alternative-to-passing-props*/}

Context 允许父组件向其下方整个树提供数据。context 有很多用途。下面是一个例子。假设有一个 `Heading` 组件，它接收一个 `level` 作为大小：

<Sandpack>

```js
import Heading from './Heading.js';
import Section from './Section.js';

export default function Page() {
  return (
    <Section>
      <Heading level={1}>标题</Heading>
      <Heading level={2}>标题</Heading>
      <Heading level={3}>副标题</Heading>
      <Heading level={4}>副副标题</Heading>
      <Heading level={5}>副副副标题</Heading>
      <Heading level={6}>副副副副标题</Heading>
    </Section>
  );
}
```

```js src/Section.js
export default function Section({ children }) {
  return (
    <section className="section">
      {children}
    </section>
  );
}
```

```js src/Heading.js
export default function Heading({ level, children }) {
  switch (level) {
    case 1:
      return <h1>{children}</h1>;
    case 2:
      return <h2>{children}</h2>;
    case 3:
      return <h3>{children}</h3>;
    case 4:
      return <h4>{children}</h4>;
    case 5:
      return <h5>{children}</h5>;
    case 6:
      return <h6>{children}</h6>;
    default:
      throw Error('未知级别: ' + level);
  }
}
```

```css
.section {
  padding: 10px;
  margin: 5px;
  border-radius: 5px;
  border: 1px solid #aaa;
}
```

</Sandpack>

假设你希望同一个 `Section` 中的多个标题始终具有相同的大小：

<Sandpack>

```js
import Heading from './Heading.js';
import Section from './Section.js';

export default function Page() {
  return (
    <Section>
      <Heading level={1}>标题</Heading>
      <Section>
        <Heading level={2}>标题</Heading>
        <Heading level={2}>标题</Heading>
        <Heading level={2}>标题</Heading>
        <Section>
          <Heading level={3}>副标题</Heading>
          <Heading level={3}>副标题</Heading>
          <Heading level={3}>副标题</Heading>
          <Section>
            <Heading level={4}>副副标题</Heading>
            <Heading level={4}>副副标题</Heading>
            <Heading level={4}>副副标题</Heading>
          </Section>
        </Section>
      </Section>
    </Section>
  );
}
```

```js src/Section.js
export default function Section({ children }) {
  return (
    <section className="section">
      {children}
    </section>
  );
}
```

```js src/Heading.js
export default function Heading({ level, children }) {
  switch (level) {
    case 1:
      return <h1>{children}</h1>;
    case 2:
      return <h2>{children}</h2>;
    case 3:
      return <h3>{children}</h3>;
    case 4:
      return <h4>{children}</h4>;
    case 5:
      return <h5>{children}</h5>;
    case 6:
      return <h6>{children}</h6>;
    default:
      throw Error('未知级别: ' + level);
  }
}
```

```css
.section {
  padding: 10px;
  margin: 5px;
  border-radius: 5px;
  border: 1px solid #aaa;
}
```

</Sandpack>

目前，你是分别把 `level` prop 传给每个 `<Heading>`：

```js
<Section>
  <Heading level={3}>关于</Heading>
  <Heading level={3}>照片</Heading>
  <Heading level={3}>视频</Heading>
</Section>
```

如果你可以把 `level` prop 传给 `<Section>` 组件，而不是传给 `<Heading>`，那就更好了。这样你就可以强制同一 section 中的所有标题具有相同的大小：

```js
<Section level={3}>
  <Heading>关于</Heading>
  <Heading>照片</Heading>
  <Heading>视频</Heading>
</Section>
```

但是 `<Heading>` 组件怎么知道它最近的 `<Section>` 的级别呢？**这就需要让子组件能够从树中更上层的某处“请求”数据。**

仅靠 props 是做不到的。这就是 context 登场的地方。你会分三步完成：

1. **创建**一个 context。（你可以把它叫做 `LevelContext`，因为它用于标题级别。）
2. 在需要数据的组件中**使用**这个 context。（`Heading` 将使用 `LevelContext`。）
3. 在指定数据的组件中**提供**这个 context。（`Section` 将提供 `LevelContext`。）

Context 允许父组件——即使是很远的父组件！——向其内部整个树提供某些数据。

<DiagramGroup>

<Diagram name="passing_data_context_close" height={160} width={608} captionPosition="top" alt="一棵包含三个组件的树的示意图。父组件包含一个表示值的气泡，气泡以橙色高亮，并向下投射到两个子组件，两个子组件也都以橙色高亮。" >

在近距离子组件中使用 context

</Diagram>

<Diagram name="passing_data_context_far" height={430} width={608} captionPosition="top" alt="一棵包含十个节点的树的示意图，每个节点最多有两个子节点。根父节点包含一个表示值的气泡，气泡以橙色高亮。该值直接投射到树中的四个叶子节点和一个中间组件上，它们都以橙色高亮。其他中间组件都没有高亮。">

在远距离子组件中使用 context

</Diagram>

</DiagramGroup>

### 第 1 步：创建 context {/*step-1-create-the-context*/}

首先，你需要创建这个 context。你需要将它**从一个文件中导出**，这样你的组件才能使用它：

<Sandpack>

```js
import Heading from './Heading.js';
import Section from './Section.js';

export default function Page() {
  return (
    <Section>
      <Heading level={1}>标题</Heading>
      <Section>
        <Heading level={2}>标题</Heading>
        <Heading level={2}>标题</Heading>
        <Heading level={2}>标题</Heading>
        <Section>
          <Heading level={3}>副标题</Heading>
          <Heading level={3}>副标题</Heading>
          <Heading level={3}>副标题</Heading>
          <Section>
            <Heading level={4}>副副标题</Heading>
            <Heading level={4}>副副标题</Heading>
            <Heading level={4}>副副标题</Heading>
          </Section>
        </Section>
      </Section>
    </Section>
  );
}
```

```js src/Section.js
export default function Section({ children }) {
  return (
    <section className="section">
      {children}
    </section>
  );
}
```

```js src/Heading.js
export default function Heading({ level, children }) {
  switch (level) {
    case 1:
      return <h1>{children}</h1>;
    case 2:
      return <h2>{children}</h2>;
    case 3:
      return <h3>{children}</h3>;
    case 4:
      return <h4>{children}</h4>;
    case 5:
      return <h5>{children}</h5>;
    case 6:
      return <h6>{children}</h6>;
    default:
      throw Error('未知级别: ' + level);
  }
}
```

```js src/LevelContext.js active
import { createContext } from 'react';

export const LevelContext = createContext(1);
```

```css
.section {
  padding: 10px;
  margin: 5px;
  border-radius: 5px;
  border: 1px solid #aaa;
}
```

</Sandpack>

`createContext` 的唯一参数是 _默认_ 值。这里，`1` 指的是最大的标题级别，但你可以传入任何类型的值（甚至是一个对象）。默认值的重要性会在下一步中体现出来。

### 第 2 步：使用 context {/*step-2-use-the-context*/}

从 React 中导入 `useContext` Hook，以及你的 context：

```js
import { useContext } from 'react';
import { LevelContext } from './LevelContext.js';
```

目前，`Heading` 组件是从 props 中读取 `level`：

```js
export default function Heading({ level, children }) {
  // ...
}
```

现在，把 `level` prop 删除，并从你刚导入的 context `LevelContext` 中读取该值：

```js {2}
export default function Heading({ children }) {
  const level = useContext(LevelContext);
  // ...
}
```

`useContext` 是一个 Hook。和 `useState`、`useReducer` 一样，你只能在 React 组件内部立即调用 Hook（不能在循环或条件语句中调用）。**`useContext` 会告诉 React，`Heading` 组件想要读取 `LevelContext`。**

现在 `Heading` 组件已经没有 `level` prop 了，所以你不再需要像这样在 JSX 中把 level prop 传给 `Heading`：

```js
<Section>
  <Heading level={4}>副副标题</Heading>
  <Heading level={4}>副副标题</Heading>
  <Heading level={4}>副副标题</Heading>
</Section>
```

把 JSX 更新为由 `Section` 来接收它：

```jsx
<Section level={4}>
  <Heading>副副标题</Heading>
  <Heading>副副标题</Heading>
  <Heading>副副标题</Heading>
</Section>
```

提醒一下，这就是你想要实现的标记结构：

<Sandpack>

```js
import Heading from './Heading.js';
import Section from './Section.js';

export default function Page() {
  return (
    <Section level={1}>
      <Heading>标题</Heading>
      <Section level={2}>
        <Heading>标题</Heading>
        <Heading>标题</Heading>
        <Heading>标题</Heading>
        <Section level={3}>
          <Heading>副标题</Heading>
          <Heading>副标题</Heading>
          <Heading>副标题</Heading>
          <Section level={4}>
            <Heading>副副标题</Heading>
            <Heading>副副标题</Heading>
            <Heading>副副标题</Heading>
          </Section>
        </Section>
      </Section>
    </Section>
  );
}
```

```js src/Section.js
export default function Section({ children }) {
  return (
    <section className="section">
      {children}
    </section>
  );
}
```

```js src/Heading.js
import { useContext } from 'react';
import { LevelContext } from './LevelContext.js';

export default function Heading({ children }) {
  const level = useContext(LevelContext);
  switch (level) {
    case 1:
      return <h1>{children}</h1>;
    case 2:
      return <h2>{children}</h2>;
    case 3:
      return <h3>{children}</h3>;
    case 4:
      return <h4>{children}</h4>;
    case 5:
      return <h5>{children}</h5>;
    case 6:
      return <h6>{children}</h6>;
    default:
      throw Error('未知级别: ' + level);
  }
}
```

```js src/LevelContext.js
import { createContext } from 'react';

export const LevelContext = createContext(1);
```

```css
.section {
  padding: 10px;
  margin: 5px;
  border-radius: 5px;
  border: 1px solid #aaa;
}
```

</Sandpack>

注意，这个示例现在还不能完全工作！所有标题的大小都相同，因为**尽管你正在 *使用* context，但你还没有 *提供* 它。** React 还不知道该从哪里获取它！

如果你不提供 context，React 就会使用你在上一步中指定的默认值。在这个例子中，你把 `1` 作为 `createContext` 的参数，所以 `useContext(LevelContext)` 返回 `1`，这会让所有这些标题都变成 `<h1>`。让我们通过让每个 `Section` 提供自己的 context 来修复这个问题。

### 第 3 步：提供 context {/*step-3-provide-the-context*/}

`Section` 组件目前会渲染它的子元素：

```js
export default function Section({ children }) {
  return (
    <section className="section">
      {children}
    </section>
  );
}
```

**用 context provider 包裹它们**，为它们提供 `LevelContext`：

```js {1,6,8}
import { LevelContext } from './LevelContext.js';

export default function Section({ level, children }) {
  return (
    <section className="section">
      <LevelContext value={level}>
        {children}
      </LevelContext>
    </section>
  );
}
```

这会告诉 React：“如果这个 `<Section>` 内部的任何组件请求 `LevelContext`，就把这个 `level` 给它们。” 该组件会使用其上方 UI 树中最近的 `<LevelContext>` 的值。

<Sandpack>

```js
import Heading from './Heading.js';
import Section from './Section.js';

export default function Page() {
  return (
    <Section level={1}>
      <Heading>标题</Heading>
      <Section level={2}>
        <Heading>标题</Heading>
        <Heading>标题</Heading>
        <Heading>标题</Heading>
        <Section level={3}>
          <Heading>副标题</Heading>
          <Heading>副标题</Heading>
          <Heading>副标题</Heading>
          <Section level={4}>
            <Heading>副副标题</Heading>
            <Heading>副副标题</Heading>
            <Heading>副副标题</Heading>
          </Section>
        </Section>
      </Section>
    </Section>
  );
}
```

```js src/Section.js
import { LevelContext } from './LevelContext.js';

export default function Section({ level, children }) {
  return (
    <section className="section">
      <LevelContext value={level}>
        {children}
      </LevelContext>
    </section>
  );
}
```

```js src/Heading.js
import { useContext } from 'react';
import { LevelContext } from './LevelContext.js';

export default function Heading({ children }) {
  const level = useContext(LevelContext);
  switch (level) {
    case 1:
      return <h1>{children}</h1>;
    case 2:
      return <h2>{children}</h2>;
    case 3:
      return <h3>{children}</h3>;
    case 4:
      return <h4>{children}</h4>;
    case 5:
      return <h5>{children}</h5>;
    case 6:
      return <h6>{children}</h6>;
    default:
      throw Error('未知级别: ' + level);
  }
}
```

```js src/LevelContext.js
import { createContext } from 'react';

export const LevelContext = createContext(1);
```

```css
.section {
  padding: 10px;
  margin: 5px;
  border-radius: 5px;
  border: 1px solid #aaa;
}
```

</Sandpack>

结果和原始代码一样，但你不需要再把 `level` prop 传给每个 `Heading` 组件了！相反，它会通过询问上方最近的 `Section` 来“弄清楚”自己的标题级别：

1. 你把 `level` prop 传给 `<Section>`。
2. `Section` 将它的子元素包裹在 `<LevelContext value={level}>` 中。
3. `Heading` 通过 `useContext(LevelContext)` 向上请求最近的 `LevelContext` 值。

## 使用并从同一组件提供上下文 {/*using-and-providing-context-from-the-same-component*/}

目前，你仍然必须手动指定每个部分的 `level`：

```js
export default function Page() {
  return (
    <Section level={1}>
      ...
      <Section level={2}>
        ...
        <Section level={3}>
          ...
```

由于上下文允许你读取上方组件中的信息，因此每个 `Section` 都可以从上方的 `Section` 读取 `level`，并自动向下传递 `level + 1`。下面是实现方式：

```js src/Section.js {5,8}
import { useContext } from 'react';
import { LevelContext } from './LevelContext.js';

export default function Section({ children }) {
  const level = useContext(LevelContext);
  return (
    <section className="section">
      <LevelContext value={level + 1}>
        {children}
      </LevelContext>
    </section>
  );
}
```

有了这个改动，你就不需要再向 `<Section>` 或 `<Heading>` 传递 `level` 属性了：

<Sandpack>

```js
import Heading from './Heading.js';
import Section from './Section.js';

export default function Page() {
  return (
    <Section>
      <Heading>标题</Heading>
      <Section>
        <Heading>标题</Heading>
        <Heading>标题</Heading>
        <Heading>标题</Heading>
        <Section>
          <Heading>子标题</Heading>
          <Heading>子标题</Heading>
          <Heading>子标题</Heading>
          <Section>
            <Heading>子子标题</Heading>
            <Heading>子子标题</Heading>
            <Heading>子子标题</Heading>
          </Section>
        </Section>
      </Section>
    </Section>
  );
}
```

```js src/Section.js
import { useContext } from 'react';
import { LevelContext } from './LevelContext.js';

export default function Section({ children }) {
  const level = useContext(LevelContext);
  return (
    <section className="section">
      <LevelContext value={level + 1}>
        {children}
      </LevelContext>
    </section>
  );
}
```

```js src/Heading.js
import { useContext } from 'react';
import { LevelContext } from './LevelContext.js';

export default function Heading({ children }) {
  const level = useContext(LevelContext);
  switch (level) {
    case 0:
      throw Error('Heading must be inside a Section!');
    case 1:
      return <h1>{children}</h1>;
    case 2:
      return <h2>{children}</h2>;
    case 3:
      return <h3>{children}</h3>;
    case 4:
      return <h4>{children}</h4>;
    case 5:
      return <h5>{children}</h5>;
    case 6:
      return <h6>{children}</h6>;
    default:
      throw Error('Unknown level: ' + level);
  }
}
```

```js src/LevelContext.js
import { createContext } from 'react';

export const LevelContext = createContext(0);
```

```css
.section {
  padding: 10px;
  margin: 5px;
  border-radius: 5px;
  border: 1px solid #aaa;
}
```

</Sandpack>

现在 `Heading` 和 `Section` 都会读取 `LevelContext` 来判断它们处于多“深”的层级。并且 `Section` 会把它的子元素包裹在 `LevelContext` 中，用来指定其中的内容处于更“深”一层的级别。

<Note>

这个示例使用标题层级，是因为它们能直观地展示嵌套组件如何覆盖上下文。不过，上下文在许多其他场景中也很有用。你可以向整个子树传递所需的任何信息：当前配色主题、当前登录用户，等等。

</Note>

## 上下文会穿透中间组件 {/*context-passes-through-intermediate-components*/}

在提供上下文的组件和使用它的组件之间，你可以插入任意多个组件。这既包括像 `<div>` 这样的内置组件，也包括你自己编写的组件。

在这个示例中，同一个 `Post` 组件（带虚线边框）会在两个不同的嵌套层级中渲染。注意其中的 `<Heading>` 会自动从最近的 `<Section>` 获取它的层级：

<Sandpack>

```js
import Heading from './Heading.js';
import Section from './Section.js';

export default function ProfilePage() {
  return (
    <Section>
      <Heading>我的资料</Heading>
      <Post
        title="你好，旅人！"
        body="阅读我的冒险经历。"
      />
      <AllPosts />
    </Section>
  );
}

function AllPosts() {
  return (
    <Section>
      <Heading>帖子</Heading>
      <RecentPosts />
    </Section>
  );
}

function RecentPosts() {
  return (
    <Section>
      <Heading>最近的帖子</Heading>
      <Post
        title="里斯本的风味"
        body="……那些葡式蛋挞！"
      />
      <Post
        title="布宜诺斯艾利斯的探戈节奏"
        body="我很喜欢！"
      />
    </Section>
  );
}

function Post({ title, body }) {
  return (
    <Section isFancy={true}>
      <Heading>
        {title}
      </Heading>
      <p><i>{body}</i></p>
    </Section>
  );
}
```

```js src/Section.js
import { useContext } from 'react';
import { LevelContext } from './LevelContext.js';

export default function Section({ children, isFancy }) {
  const level = useContext(LevelContext);
  return (
    <section className={
      'section ' +
      (isFancy ? 'fancy' : '')
    }>
      <LevelContext value={level + 1}>
        {children}
      </LevelContext>
    </section>
  );
}
```

```js src/Heading.js
import { useContext } from 'react';
import { LevelContext } from './LevelContext.js';

export default function Heading({ children }) {
  const level = useContext(LevelContext);
  switch (level) {
    case 0:
      throw Error('Heading must be inside a Section!');
    case 1:
      return <h1>{children}</h1>;
    case 2:
      return <h2>{children}</h2>;
    case 3:
      return <h3>{children}</h3>;
    case 4:
      return <h4>{children}</h4>;
    case 5:
      return <h5>{children}</h5>;
    case 6:
      return <h6>{children}</h6>;
    default:
      throw Error('Unknown level: ' + level);
  }
}
```

```js src/LevelContext.js
import { createContext } from 'react';

export const LevelContext = createContext(0);
```

```css
.section {
  padding: 10px;
  margin: 5px;
  border-radius: 5px;
  border: 1px solid #aaa;
}

.fancy {
  border: 4px dashed pink;
}
```

</Sandpack>

你并没有为此做什么特殊处理。`Section` 会为其内部的树指定上下文，因此你可以在任何位置插入 `<Heading>`，它都会具有正确的大小。试试上面的沙盒吧！

**上下文让你可以编写“适应周围环境”的组件，并根据它们被渲染在_哪里_（换句话说，_处于什么上下文中_）来以不同方式显示自己。**

上下文的工作方式可能会让你联想到 [CSS 属性继承](https://developer.mozilla.org/en-US/docs/Web/CSS/inheritance)。在 CSS 中，你可以为一个 `<div>` 指定 `color: blue`，其中任何 DOM 节点，无论嵌套多深，都会继承该颜色，除非中间某个其他 DOM 节点用 `color: green` 覆盖了它。同样，在 React 中，覆盖来自上方的某个上下文的唯一方式，就是用不同的值将子元素包裹到一个上下文提供器中。

在 CSS 中，不同的属性，比如 `color` 和 `background-color`，不会互相覆盖。你可以把所有 `<div>` 的 `color` 设为红色，而不会影响 `background-color`。类似地，**不同的 React 上下文不会互相覆盖。** 你用 `createContext()` 创建的每个上下文都与其他上下文完全独立，并将使用和提供_那个特定_上下文的组件连接起来。一个组件可以同时使用或提供很多不同的上下文，而不会有问题。

## 在使用上下文之前 {/*before-you-use-context*/}

上下文非常诱人，容易使用！不过，这也意味着它很容易被过度使用。**仅仅因为你需要把一些 props 向下传递好几层，并不意味着你应该把这些信息放进上下文。**

在使用上下文之前，你应该先考虑以下几种替代方案：

1. **先从 [传递 props](/learn/passing-props-to-a-component) 开始。** 如果你的组件并不简单，那么把十几个 props 通过十几个组件传下去并不罕见。这样做可能会让人觉得有点繁琐，但它能非常清楚地表明哪些组件使用了哪些数据！维护你代码的人会很高兴，因为你已经通过 props 让数据流变得显式。
2. **提取组件，并把 JSX 作为 `children` 传给它们。** 如果你把某些数据穿过许多层并不使用这些数据的中间组件（而只是继续向下传），这通常意味着你在过程中忘了提取一些组件。例如，可能你把像 `posts` 这样的数据 props 传给了不直接使用它们的视觉组件，比如 `<Layout posts={posts} />`。相反，应让 `Layout` 接收 `children` 作为 prop，并渲染 `<Layout><Posts posts={posts} /></Layout>`。这样可以减少指定数据的组件与需要这些数据的组件之间的层数。

如果这两种方式都不太适合你，再考虑上下文。

## 上下文的使用场景 {/*use-cases-for-context*/}

* **主题：** 如果你的应用允许用户更改外观（例如深色模式），你可以在应用顶部放置一个上下文提供器，并在需要调整视觉样式的组件中使用该上下文。
* **当前账户：** 许多组件可能需要知道当前登录用户是谁。把它放入上下文后，在树中的任何位置读取它都很方便。有些应用还允许你同时操作多个账户（例如以另一个用户的身份发表评论）。在这种情况下，把 UI 的一部分包裹到一个嵌套提供器中，并为其设置不同的当前账户值，会很方便。
* **路由：** 大多数路由方案都会在内部使用上下文来保存当前路由。这就是每个链接“知道”自己是否处于激活状态的方式。如果你自己构建路由器，可能也会想这样做。
* **管理状态：** 随着应用增长，你最终可能会在应用顶部附近放置很多状态。下面许多较远的组件可能希望修改它。通常会 [将 reducer 与上下文结合使用](/learn/scaling-up-with-reducer-and-context) 来管理复杂状态，并把它传递给较远的组件，而无需太多麻烦。

上下文不仅限于静态值。如果你在下一次渲染时传递了不同的值，React 会更新所有读取它的下层组件！这就是为什么上下文经常与状态一起使用。

一般来说，如果某些信息需要被树中不同部分的较远组件使用，这通常说明上下文会很有帮助。

<Recap>

* 上下文让组件可以向其下方整个树提供一些信息。
* 传递上下文的步骤：
  1. 使用 `export const MyContext = createContext(defaultValue)` 创建并导出它。
  2. 将它传给 `useContext(MyContext)` Hook，以便在任意子组件中读取它，无论多深。
  3. 把子元素包裹进 `<MyContext value={...}>`，从父组件提供它。
* 上下文会穿透中间的任意组件。
* 上下文让你可以编写“适应周围环境”的组件。
* 在使用上下文之前，先试试传递 props 或将 JSX 作为 `children` 传递。

</Recap>

<Challenges>

#### 用上下文替代层层传递 props {/*replace-prop-drilling-with-context*/}

在这个示例中，切换复选框会改变传递给每个 `<PlaceImage>` 的 `imageSize` prop。复选框状态保存在顶层的 `App` 组件中，但每个 `<PlaceImage>` 都需要知道它。

目前，`App` 将 `imageSize` 传给 `List`，`List` 再传给每个 `Place`，然后 `Place` 再传给 `PlaceImage`。删除 `imageSize` prop，改为直接从 `App` 组件传给 `PlaceImage`。

你可以在 `Context.js` 中声明上下文。

<Sandpack>

```js src/App.js
import { useState } from 'react';
import { places } from './data.js';
import { getImageUrl } from './utils.js';

export default function App() {
  const [isLarge, setIsLarge] = useState(false);
  const imageSize = isLarge ? 150 : 100;
  return (
    <>
      <label>
        <input
          type="checkbox"
          checked={isLarge}
          onChange={e => {
            setIsLarge(e.target.checked);
          }}
        />
        使用大图
      </label>
      <hr />
      <List imageSize={imageSize} />
    </>
  )
}

function List({ imageSize }) {
  const listItems = places.map(place =>
    <li key={place.id}>
      <Place
        place={place}
        imageSize={imageSize}
      />
    </li>
  );
  return <ul>{listItems}</ul>;
}

function Place({ place, imageSize }) {
  return (
    <>
      <PlaceImage
        place={place}
        imageSize={imageSize}
      />
      <p>
        <b>{place.name}</b>
        {': ' + place.description}
      </p>
    </>
  );
}

function PlaceImage({ place, imageSize }) {
  return (
    <img
      src={getImageUrl(place)}
      alt={place.name}
      width={imageSize}
      height={imageSize}
    />
  );
}
```

```js src/Context.js

```

```js src/data.js
export const places = [{
  id: 0,
  name: 'Bo-Kaap in Cape Town, South Africa',
  description: 'The tradition of choosing bright colors for houses began in the late 20th century.',
  imageId: 'K9HVAGH'
}, {
  id: 1,
  name: 'Rainbow Village in Taichung, Taiwan',
  description: 'To save the houses from demolition, Huang Yung-Fu, a local resident, painted all 1,200 of them in 1924.',
  imageId: '9EAYZrt'
}, {
  id: 2,
  name: 'Macromural de Pachuca, Mexico',
  description: 'One of the largest murals in the world covering homes in a hillside neighborhood.',
  imageId: 'DgXHVwu'
}, {
  id: 3,
  name: 'Selarón Staircase in Rio de Janeiro, Brazil',
  description: 'This landmark was created by Jorge Selarón, a Chilean-born artist, as a "tribute to the Brazilian people."',
  imageId: 'aeO3rpI'
}, {
  id: 4,
  name: 'Burano, Italy',
  description: 'The houses are painted following a specific color system dating back to 16th century.',
  imageId: 'kxsph5C'
}, {
  id: 5,
  name: 'Chefchaouen, Marocco',
  description: 'There are a few theories on why the houses are painted blue, including that the color repels mosquitos or that it symbolizes sky and heaven.',
  imageId: 'rTqKo46'
}, {
  id: 6,
  name: 'Gamcheon Culture Village in Busan, South Korea',
  description: 'In 2009, the village was converted into a cultural hub by painting the houses and featuring exhibitions and art installations.',
  imageId: 'ZfQOOzf'
}];
```

```js src/utils.js
export function getImageUrl(place) {
  return (
    'https://react.dev/images/docs/scientists/' +
    place.imageId +
    'l.jpg'
  );
}
```

```css
ul { list-style-type: none; padding: 0px 10px; }
li {
  margin-bottom: 10px;
  display: grid;
  grid-template-columns: auto 1fr;
  gap: 20px;
  align-items: center;
}
```

</Sandpack>

<Solution>

从所有组件中移除 `imageSize` prop。

在 `Context.js` 中创建并导出 `ImageSizeContext`。然后把 `List` 包裹进 `<ImageSizeContext value={imageSize}>` 来向下传递这个值，并在 `PlaceImage` 中使用 `useContext(ImageSizeContext)` 来读取它：

<Sandpack>

```js src/App.js
import { useState, useContext } from 'react';
import { places } from './data.js';
import { getImageUrl } from './utils.js';
import { ImageSizeContext } from './Context.js';

export default function App() {
  const [isLarge, setIsLarge] = useState(false);
  const imageSize = isLarge ? 150 : 100;
  return (
    <ImageSizeContext
      value={imageSize}
    >
      <label>
        <input
          type="checkbox"
          checked={isLarge}
          onChange={e => {
            setIsLarge(e.target.checked);
          }}
        />
        使用大图
      </label>
      <hr />
      <List />
    </ImageSizeContext>
  )
}

function List() {
  const listItems = places.map(place =>
    <li key={place.id}>
      <Place place={place} />
    </li>
  );
  return <ul>{listItems}</ul>;
}

function Place({ place }) {
  return (
    <>
      <PlaceImage place={place} />
      <p>
        <b>{place.name}</b>
        {': ' + place.description}
      </p>
    </>
  );
}

function PlaceImage({ place }) {
  const imageSize = useContext(ImageSizeContext);
  return (
    <img
      src={getImageUrl(place)}
      alt={place.name}
      width={imageSize}
      height={imageSize}
    />
  );
}
```

```js src/Context.js
import { createContext } from 'react';

export const ImageSizeContext = createContext(500);
```

```js src/data.js
export const places = [{
  id: 0,
  name: 'Bo-Kaap in Cape Town, South Africa',
  description: 'The tradition of choosing bright colors for houses began in the late 20th century.',
  imageId: 'K9HVAGH'
}, {
  id: 1,
  name: 'Rainbow Village in Taichung, Taiwan',
  description: 'To save the houses from demolition, Huang Yung-Fu, a local resident, painted all 1,200 of them in 1924.',
  imageId: '9EAYZrt'
}, {
  id: 2,
  name: 'Macromural de Pachuca, Mexico',
  description: 'One of the largest murals in the world covering homes in a hillside neighborhood.',
  imageId: 'DgXHVwu'
}, {
  id: 3,
  name: 'Selarón Staircase in Rio de Janeiro, Brazil',
  description: 'This landmark was created by Jorge Selarón, a Chilean-born artist, as a "tribute to the Brazilian people."',
  imageId: 'aeO3rpI'
}, {
  id: 4,
  name: 'Burano, Italy',
  description: 'The houses are painted following a specific color system dating back to 16th century.',
  imageId: 'kxsph5C'
}, {
  id: 5,
  name: 'Chefchaouen, Marocco',
  description: 'There are a few theories on why the houses are painted blue, including that the color repels mosquitos or that it symbolizes sky and heaven.',
  imageId: 'rTqKo46'
}, {
  id: 6,
  name: 'Gamcheon Culture Village in Busan, South Korea',
  description: 'In 2009, the village was converted into a cultural hub by painting the houses and featuring exhibitions and art installations.',
  imageId: 'ZfQOOzf'
}];
```

```js src/utils.js
export function getImageUrl(place) {
  return (
    'https://react.dev/images/docs/scientists/' +
    place.imageId +
    'l.jpg'
  );
}
```

```css
ul { list-style-type: none; padding: 0px 10px; }
li {
  margin-bottom: 10px;
  display: grid;
  grid-template-columns: auto 1fr;
  gap: 20px;
  align-items: center;
}
```

</Sandpack>

注意中间的组件现在不需要再传递 `imageSize` 了。

</Solution>

</Challenges>
