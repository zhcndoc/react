---
title: <Fragment> (<>...</>)
---

<Intro>

`<Fragment>`，通常通过 `<>...</>` 语法使用，可让你将元素分组而无需额外的包装节点。

<Canary> Fragment 还可以接受 refs，这使得你可以在不添加包装元素的情况下与底层 DOM 节点交互。请参阅下方的参考与用法。</Canary>

```js
<>
  <OneChild />
  <AnotherChild />
</>
```

</Intro>

<InlineToc />

---

## 参考 {/*reference*/}

### `<Fragment>` {/*fragment*/}

使用 `<Fragment>` 将元素包裹起来，以便在需要单个元素的场景中将它们分组。用 `Fragment` 对元素进行分组不会对最终的 DOM 产生任何影响；它与这些元素未被分组时是一样的。在大多数情况下，空的 JSX 标签 `<></>` 是 `<Fragment></Fragment>` 的简写。

#### 属性 {/*props*/}

- **可选** `key`：使用显式 `<Fragment>` 语法声明的 Fragment 可以具有 [keys.](/learn/rendering-lists#keeping-list-items-in-order-with-key)
- <CanaryBadge />  **可选** `ref`：一个 ref 对象（例如来自 [`useRef`](/reference/react/useRef)）或 [回调函数](/reference/react-dom/components/common#ref-callback)。React 会提供一个 `FragmentInstance` 作为 ref 值，其中实现了用于与被 Fragment 包裹的 DOM 节点交互的方法。

### <CanaryBadge /> FragmentInstance {/*fragmentinstance*/}

当你向 fragment 传入 ref 时，React 会提供一个 `FragmentInstance` 对象，其中包含用于与被 fragment 包裹的 DOM 节点交互的方法：

**事件处理方法：**
- `addEventListener(type, listener, options?)`：向 Fragment 的所有一级 DOM 子节点添加事件监听器。
- `removeEventListener(type, listener, options?)`：从 Fragment 的所有一级 DOM 子节点移除事件监听器。
- `dispatchEvent(event)`：向 Fragment 的一个虚拟子节点分发事件，以调用任何已添加的监听器，并且可以冒泡到 DOM 父节点。

**布局方法：**
- `compareDocumentPosition(otherNode)`：比较 Fragment 与另一个节点在文档中的位置。
  - 如果 Fragment 有子节点，则返回原生的 `compareDocumentPosition` 值。
  - 空 Fragment 会尝试比较 React 树中的位置，并包含 `Node.DOCUMENT_POSITION_IMPLEMENTATION_SPECIFIC`。
  - 由于 portal 或其他插入方式导致 React 树与 DOM 树中的关系不同的元素，其值为 `Node.DOCUMENT_POSITION_IMPLEMENTATION_SPECIFIC`。
- `getClientRects()`：返回一个由 `DOMRect` 对象组成的扁平数组，表示所有子节点的边界矩形。
- `getRootNode()`：返回包含 Fragment 父 DOM 节点的根节点。

**焦点管理方法：**
- `focus(options?)`：使 Fragment 中第一个可聚焦的 DOM 节点获得焦点。会优先对嵌套子节点进行深度优先的聚焦尝试。
- `focusLast(options?)`：使 Fragment 中最后一个可聚焦的 DOM 节点获得焦点。会优先对嵌套子节点进行深度优先的聚焦尝试。
- `blur()`：如果 `document.activeElement` 位于 Fragment 内，则移除焦点。

**观察者方法：**
- `observeUsing(observer)`：使用 IntersectionObserver 或 ResizeObserver 开始观察 Fragment 的 DOM 子节点。
- `unobserveUsing(observer)`：使用指定的观察器停止观察 Fragment 的 DOM 子节点。

#### 注意事项 {/*caveats*/}

- 如果你想向 Fragment 传递 `key`，就不能使用 `<>...</>` 语法。你必须显式地从 `'react'` 导入 `Fragment`，并渲染 `<Fragment key={yourKey}>...</Fragment>`。

- 当你从渲染 `<><Child /></>` 切换到 `[<Child />]` 或反过来，或者从渲染 `<><Child /></>` 切换到 `<Child />` 再切回来时，React 不会 [重置状态](/learn/preserving-and-resetting-state)。这只在单层深度下有效：例如，从 `<><><Child /></></>` 切换到 `<Child />` 会重置状态。精确语义请参见 [这里。](https://gist.github.com/clemmy/b3ef00f9507909429d8aa0d3ee4f986b)

- <CanaryBadge /> 如果你想向 Fragment 传递 `ref`，就不能使用 `<>...</>` 语法。你必须显式地从 `'react'` 导入 `Fragment`，并渲染 `<Fragment ref={yourRef}>...</Fragment>`。

---

## 用法 {/*usage*/}

### 返回多个元素 {/*returning-multiple-elements*/}

使用 `Fragment`，或等价的 `<>...</>` 语法，将多个元素分组在一起。你可以在任何只能放单个元素的位置使用它来放置多个元素。例如，组件只能返回一个元素，但借助 Fragment，你可以将多个元素组合在一起，然后作为一个整体返回：

```js {3,6}
function Post() {
  return (
    <>
      <PostTitle />
      <PostBody />
    </>
  );
}
```

Fragment 很有用，因为使用 Fragment 对元素分组不会影响布局或样式，而如果你把这些元素包裹在另一个容器中（例如 DOM 元素）就会有影响。如果你使用浏览器工具检查这个示例，你会看到所有 `<h1>` 和 `<article>` DOM 节点都作为兄弟节点出现，而周围没有包装器：

<Sandpack>

```js
export default function Blog() {
  return (
    <>
      <Post title="An update" body="It's been a while since I posted..." />
      <Post title="My new blog" body="I am starting a new blog!" />
    </>
  )
}

function Post({ title, body }) {
  return (
    <>
      <PostTitle title={title} />
      <PostBody body={body} />
    </>
  );
}

function PostTitle({ title }) {
  return <h1>{title}</h1>
}

function PostBody({ body }) {
  return (
    <article>
      <p>{body}</p>
    </article>
  );
}
```

</Sandpack>

<DeepDive>

#### 如何在不使用特殊语法的情况下编写 Fragment？ {/*how-to-write-a-fragment-without-the-special-syntax*/}

上面的示例等价于从 React 导入 `Fragment`：

```js {1,5,8}
import { Fragment } from 'react';

function Post() {
  return (
    <Fragment>
      <PostTitle />
      <PostBody />
    </Fragment>
  );
}
```

通常你不需要这样做，除非你需要 [向你的 `Fragment` 传递 `key`。](#rendering-a-list-of-fragments)

</DeepDive>

---

### 将多个元素赋值给变量 {/*assigning-multiple-elements-to-a-variable*/}

和其他元素一样，你可以将 Fragment 元素赋值给变量、将它们作为 props 传递，等等：

```js
function CloseDialog() {
  const buttons = (
    <>
      <OKButton />
      <CancelButton />
    </>
  );
  return (
    <AlertDialog buttons={buttons}>
      你确定要离开这个页面吗？
    </AlertDialog>
  );
}
```

---

### 将元素与文本分组 {/*grouping-elements-with-text*/}

你可以使用 `Fragment` 将文本与组件组合在一起：

```js
function DateRangePicker({ start, end }) {
  return (
    <>
      从
      <DatePicker date={start} />
      到
      <DatePicker date={end} />
    </>
  );
}
```

---

### 渲染 Fragment 列表 {/*rendering-a-list-of-fragments*/}

这里有一种情况，你需要显式编写 `Fragment`，而不是使用 `<></>` 语法。当你在循环中 [渲染多个元素](/learn/rendering-lists) 时，需要为每个元素分配一个 `key`。如果循环中的元素是 Fragments，则需要使用普通的 JSX 元素语法，以便提供 `key` 属性：

```js {3,6}
function Blog() {
  return posts.map(post =>
    <Fragment key={post.id}>
      <PostTitle title={post.title} />
      <PostBody body={post.body} />
    </Fragment>
  );
}
```

你可以检查 DOM，以验证 Fragment 子节点周围没有包装元素：

<Sandpack>

```js
import { Fragment } from 'react';

const posts = [
  { id: 1, title: 'An update', body: "It's been a while since I posted..." },
  { id: 2, title: 'My new blog', body: 'I am starting a new blog!' }
];

export default function Blog() {
  return posts.map(post =>
    <Fragment key={post.id}>
      <PostTitle title={post.title} />
      <PostBody body={post.body} />
    </Fragment>
  );
}

function PostTitle({ title }) {
  return <h1>{title}</h1>
}

function PostBody({ body }) {
  return (
    <article>
      <p>{body}</p>
    </article>
  );
}
```

</Sandpack>

---

### <CanaryBadge /> 使用 Fragment refs 进行 DOM 交互 {/*using-fragment-refs-for-dom-interaction*/}

Fragment refs 允许你在不添加额外包装元素的情况下与被 Fragment 包裹的 DOM 节点交互。这对于事件处理、可见性跟踪、焦点管理，以及替代已弃用的模式（如 `ReactDOM.findDOMNode()`）都很有用。

```js
import { Fragment } from 'react';

function ClickableFragment({ children, onClick }) {
  return (
    <Fragment ref={fragmentInstance => {
      fragmentInstance.addEventListener('click', handleClick);
      return () => fragmentInstance.removeEventListener('click', handleClick);
    }}>
      {children}
    </Fragment>
  );
}
```
---

### <CanaryBadge /> 使用 Fragment refs 跟踪可见性 {/*tracking-visibility-with-fragment-refs*/}

Fragment refs 对可见性跟踪和交叉观察很有用。这使你能够在不要求子组件暴露 refs 的情况下监测内容何时变得可见：

```js {19,21,31-34}
import { Fragment, useRef, useLayoutEffect } from 'react';

function VisibilityObserverFragment({ threshold = 0.5, onVisibilityChange, children }) {
  const fragmentRef = useRef(null);

  useLayoutEffect(() => {
    const observer = new IntersectionObserver(
      (entries) => {
        onVisibilityChange(entries.some(entry => entry.isIntersecting))
      },
      { threshold }
    );

    fragmentRef.current.observeUsing(observer);
    return () => fragmentRef.current.unobserveUsing(observer);
  }, [threshold, onVisibilityChange]);

  return (
    <Fragment ref={fragmentRef}>
      {children}
    </Fragment>
  );
}

function MyComponent() {
  const handleVisibilityChange = (isVisible) => {
    console.log('组件是', isVisible ? '可见' : '隐藏');
  };

  return (
    <VisibilityObserverFragment onVisibilityChange={handleVisibilityChange}>
      <SomeThirdPartyComponent />
      <AnotherComponent />
    </VisibilityObserverFragment>
  );
}
```

这种模式是基于 Effect 的可见性日志记录的一种替代方案，而在大多数情况下，这属于反模式。仅仅依赖 Effects 并不能保证渲染出的组件对用户是可观察的。

---

### <CanaryBadge /> 使用 Fragment refs 进行焦点管理 {/*focus-management-with-fragment-refs*/}

Fragment refs 提供可跨 Fragment 内所有 DOM 节点工作的焦点管理方法：

```js
import { Fragment, useRef } from 'react';

function FocusFragment({ children }) {
  return (
    <Fragment ref={(fragmentInstance) => fragmentInstance?.focus()}>
      {children}
    </Fragment>
  );
}
```

`focus()` 方法会使 Fragment 内第一个可聚焦元素获得焦点，而 `focusLast()` 会使最后一个可聚焦元素获得焦点。
