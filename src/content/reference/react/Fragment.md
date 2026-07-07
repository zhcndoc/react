---
title: <Fragment> (<>...</>)
---

<Intro>

`<Fragment>`，通常通过 `<>...</>` 语法使用，可让你将元素分组而无需额外的包装节点。

<Canary>Fragment 还可以接受 refs，这使你能够在不添加包装元素的情况下与底层 DOM 节点交互。</Canary>

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

- **可选** `key`: 以显式 `<Fragment>` 语法声明的 Fragments 可以有 [keys.](/learn/rendering-lists#keeping-list-items-in-order-with-key)
- <CanaryBadge /> **可选** `ref`: 一个 ref 对象（例如来自 [`useRef`](/reference/react/useRef)）或 [回调函数](/reference/react-dom/components/common#ref-callback)。React 会提供一个 `FragmentInstance` 作为 ref 值，它实现了用于与被 Fragment 包裹的 DOM 节点交互的方法。

#### 注意事项 {/*caveats*/}

* 如果你想向 Fragment 传递 `key`，就不能使用 `<>...</>` 语法。你必须显式地从 `'react'` 中导入 `Fragment`，并渲染 `<Fragment key={yourKey}>...</Fragment>`。

* 当你从渲染 `<><Child /></>` 切换到 `[<Child />]` 或切换回来，或者当你从渲染 `<><Child /></>` 切换到 `<Child />` 再切换回来时，React 不会[重置状态](/learn/preserving-and-resetting-state)。这只在单层深度时有效：例如，从 `<><><Child /></></>` 切换到 `<Child />` 会重置状态。请参阅这里的精确语义[。](https://gist.github.com/clemmy/b3ef00f9507909429d8aa0d3ee4f986b)

* <CanaryBadge /> 如果你想向 Fragment 传递 `ref`，就不能使用 `<>...</>` 语法。你必须显式地从 `'react'` 中导入 `Fragment`，并渲染 `<Fragment ref={yourRef}>...</Fragment>`。

---

### <CanaryBadge /> `FragmentInstance` {/*fragmentinstance*/}

当你将 `ref` 传给一个 Fragment 时，React 会提供一个 `FragmentInstance` 对象。它实现了一些用于与 Fragment 包裹的一级 DOM 子节点交互的方法。

* [`addEventListener`](#addeventlistener) 和 [`removeEventListener`](#removeeventlistener) 管理所有一级 DOM 子节点上的事件监听器。
* [`dispatchEvent`](#dispatchevent) 在 Fragment 上派发事件，该事件可以冒泡到 DOM 父节点。
* [`focus`](#focus)、[`focusLast`](#focuslast) 和 [`blur`](#blur) 按深度优先方式管理所有嵌套子节点的焦点。
* [`observeUsing`](#observeusing) 和 [`unobserveUsing`](#unobserveusing) 绑定和解除绑定 `IntersectionObserver` 或 `ResizeObserver` 实例。
* [`getClientRects`](#getclientrects) 返回所有一级 DOM 子节点的边界矩形。
* [`getRootNode`](#getrootnode) 返回 Fragment 父节点的根节点。
* [`compareDocumentPosition`](#comparedocumentposition) 比较 Fragment 与另一个节点的位置。
* [`scrollIntoView`](#scrollintoview) 将 Fragment 的子节点滚动到视图中。

---

#### `addEventListener(type, listener, options?)` {/*addeventlistener*/}

向 Fragment 的所有一级 DOM 子节点添加事件监听器。

```js
fragmentRef.current.addEventListener('click', handleClick);
```

##### 参数 {/*addeventlistener-parameters*/}

* `type`：表示要监听的事件类型的字符串（例如 `'click'`、`'focus'`）。
* `listener`：事件处理函数。
* **可选** `options`：用于捕获阶段的选项对象或布尔值，符合 [DOM `addEventListener` API.](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener)

##### 返回值 {/*addeventlistener-returns*/}

`addEventListener` 不返回任何内容（`undefined`）。

---

#### `removeEventListener(type, listener, options?)` {/*removeeventlistener*/}

从 Fragment 的所有一级 DOM 子节点中移除事件监听器。

```js
fragmentRef.current.removeEventListener('click', handleClick);
```

##### 参数 {/*removeeventlistener-parameters*/}

* `type`：事件类型字符串。
* `listener`：要移除的事件处理函数。
* **可选** `options`：选项对象或布尔值，符合 [DOM `removeEventListener` API.](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/removeEventListener)

##### 返回值 {/*removeeventlistener-returns*/}

`removeEventListener` 不返回任何内容（`undefined`）。

---

#### `dispatchEvent(event)` {/*dispatchevent*/}

在 Fragment 上派发一个事件。已添加的事件监听器会被调用，并且该事件可以冒泡到 Fragment 的 DOM 父节点。

```js
fragmentRef.current.dispatchEvent(new Event('custom', { bubbles: true }));
```

##### 参数 {/*dispatchevent-parameters*/}

* `event`：要派发的一个 [`Event`](https://developer.mozilla.org/en-US/docs/Web/API/Event) 对象。如果 `bubbles` 为 `true`，事件会冒泡到 Fragment 的父 DOM 节点。

##### 返回值 {/*dispatchevent-returns*/}

如果事件未被取消，则返回 `true`；如果调用了 `preventDefault()`，则返回 `false`。

---

#### `focus(options?)` {/*focus*/}

将焦点设置到 Fragment 中第一个可聚焦的 DOM 节点。与对 DOM 元素调用 `element.focus()` 不同，此方法会按深度优先方式搜索 *所有* 嵌套子节点，直到找到一个可聚焦元素——不仅仅是该元素本身或它的直接子节点。

```js
fragmentRef.current.focus();
```

##### 参数 {/*focus-parameters*/}

* **可选** `options`：一个 [`FocusOptions`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/focus#options) 对象（例如 `{ preventScroll: true }`）。

##### 返回值 {/*focus-returns*/}

`focus` 不返回任何内容（`undefined`）。

---

#### `focusLast(options?)` {/*focuslast*/}

将焦点设置到 Fragment 中最后一个可聚焦的 DOM 节点。按深度优先方式搜索嵌套子节点，然后反向遍历。

```js
fragmentRef.current.focusLast();
```

##### 参数 {/*focuslast-parameters*/}

* **可选** `options`：一个 [`FocusOptions`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/focus#options) 对象。

##### 返回值 {/*focuslast-returns*/}

`focusLast` 不返回任何内容（`undefined`）。

---

#### `blur()` {/*blur*/}

如果当前活动元素位于 Fragment 内，则移除其焦点。如果 `document.activeElement` 不在 Fragment 内，`blur` 不执行任何操作。

```js
fragmentRef.current.blur();
```

##### 返回值 {/*blur-returns*/}

`blur` 不返回任何内容（`undefined`）。

---

#### `observeUsing(observer)` {/*observeusing*/}

使用提供的观察器开始观察 Fragment 的所有一级 DOM 子节点。

```js
const observer = new IntersectionObserver(callback, options);
fragmentRef.current.observeUsing(observer);
```

##### 参数 {/*observeusing-parameters*/}

* `observer`：一个 [`IntersectionObserver`](https://developer.mozilla.org/en-US/docs/Web/API/IntersectionObserver) 或 [`ResizeObserver`](https://developer.mozilla.org/en-US/docs/Web/API/ResizeObserver) 实例。

##### 返回值 {/*observeusing-returns*/}

`observeUsing` 不返回任何内容（`undefined`）。

---

#### `unobserveUsing(observer)` {/*unobserveusing*/}

停止使用指定观察器观察 Fragment 的 DOM 子节点。

```js
fragmentRef.current.unobserveUsing(observer);
```

##### 参数 {/*unobserveusing-parameters*/}

* `observer`：之前传给 [`observeUsing`](#observeusing) 的同一个 `IntersectionObserver` 或 `ResizeObserver` 实例。

##### 返回值 {/*unobserveusing-returns*/}

`unobserveUsing` 不返回任何内容（`undefined`）。

---

#### `getClientRects()` {/*getclientrects*/}

返回一个由 [`DOMRect`](https://developer.mozilla.org/en-US/docs/Web/API/DOMRect) 对象组成的扁平数组，表示所有一级 DOM 子节点的边界矩形。

```js
const rects = fragmentRef.current.getClientRects();
```

##### 返回值 {/*getclientrects-returns*/}

一个包含所有子节点边界矩形的 `Array<DOMRect>`。

---

#### `getRootNode(options?)` {/*getrootnode*/}

返回包含 Fragment 父 DOM 节点的根节点，行为与 [`Node.getRootNode()`](https://developer.mozilla.org/en-US/docs/Web/API/Node/getRootNode) 一致。

```js
const root = fragmentRef.current.getRootNode();
```

##### 参数 {/*getrootnode-parameters*/}

* **可选** `options`：一个带有 `composed` 布尔属性的对象，符合 [DOM `getRootNode` API.](https://developer.mozilla.org/en-US/docs/Web/API/Node/getRootNode#options)

##### 返回值 {/*getrootnode-returns*/}

一个 `Document`、`ShadowRoot`，或者在没有父 DOM 节点时返回 `FragmentInstance` 本身。

---

#### `compareDocumentPosition(otherNode)` {/*comparedocumentposition*/}

比较 Fragment 与另一个节点在文档中的位置，返回一个与 [`Node.compareDocumentPosition()`](https://developer.mozilla.org/en-US/docs/Web/API/Node/compareDocumentPosition) 行为一致的位掩码。

```js
const position = fragmentRef.current.compareDocumentPosition(otherElement);
```

##### 参数 {/*comparedocumentposition-parameters*/}

* `otherNode`：要进行比较的 DOM 节点。

##### 返回值 {/*comparedocumentposition-returns*/}

一个 [位置标志](https://developer.mozilla.org/en-US/docs/Web/API/Node/compareDocumentPosition#return_value) 的位掩码。空 Fragment 和通过 [portal](/reference/react-dom/createPortal) 渲染子节点的 Fragment，结果中包含 `Node.DOCUMENT_POSITION_IMPLEMENTATION_SPECIFIC`。

---

#### `scrollIntoView(alignToTop?)` {/*scrollintoview*/}

将 Fragment 的子节点滚动到视图中。当 `alignToTop` 为 `true` 或省略时，会滚动以将第一个子节点与可滚动祖先元素的顶部对齐。当 `alignToTop` 为 `false` 时，会滚动以将最后一个子节点与底部对齐。

```js
fragmentRef.current.scrollIntoView();
```

##### 参数 {/*scrollintoview-parameters*/}

* **可选** `alignToTop`：一个布尔值。如果为 `true`（默认值），会将第一个子节点滚动到可滚动区域顶部。如果为 `false`，会将最后一个子节点滚动到底部。与 [`Element.scrollIntoView()`](https://developer.mozilla.org/en-US/docs/Web/API/Element/scrollIntoView) 不同，此方法不接受 `ScrollIntoViewOptions` 对象。

##### 返回值 {/*scrollintoview-returns*/}

`scrollIntoView` 不返回任何内容（`undefined`）。

##### 注意事项 {/*scrollintoview-caveats*/}

* `scrollIntoView` 不接受选项对象。传入该对象会抛出错误。请改用 `alignToTop` 布尔值。
* 当 Fragment 没有子节点时，`scrollIntoView` 会回退为将最近的兄弟节点或父节点滚动到视图中。

---

#### `FragmentInstance` 注意事项 {/*fragmentinstance-caveats*/}

* 作用于子节点的方法（例如 `addEventListener`、`observeUsing` 和 `getClientRects`）只会作用于 Fragment 的*一级宿主（DOM）子节点*。它们不会直接作用于嵌套在另一个 DOM 元素中的子节点。
* `focus` 和 `focusLast` 会按深度优先方式搜索嵌套子节点中的可聚焦元素，不同于事件和观察器方法，它们只作用于一级宿主子节点。
* `observeUsing` 不适用于文本节点。如果 Fragment 只包含文本子节点，React 会在开发环境中记录警告。
* React 不会将通过 `addEventListener` 添加的事件监听器应用到隐藏的 [`<Activity>`](/reference/react/Activity) 树。当 `Activity` 边界从隐藏切换为可见时，监听器会自动应用。
* 带有 `ref` 的 Fragment 的每个一级 DOM 子节点都会获得一个 `reactFragments` 属性——一个包含所有拥有该元素的 Fragment 实例的 `Set<FragmentInstance>`。这使得可以在多个 Fragment 之间缓存共享的观察器。[缓存全局 IntersectionObserver](#caching-global-intersection-observer)

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

### <CanaryBadge /> 在没有包装元素的情况下添加事件监听器 {/*adding-event-listeners-without-wrapper*/}

Fragment `ref` 允许你在不添加包装 DOM 节点的情况下，给一组元素添加事件监听器。使用 [ref 回调](/reference/react-dom/components/common#ref-callback) 来挂载并清理监听器：

<Sandpack>

```js
import { Fragment, useState, useRef, useEffect } from 'react';

function ClickableFragment({ children, onClick }) {
  const fragmentRef = useRef(null);
  useEffect(() => {
    const fragmentInstance = fragmentRef.current;
    if (fragmentInstance === null) {
      return;
    }
    fragmentInstance.addEventListener('click', onClick);
    return () => {
      fragmentInstance.removeEventListener(
        'click',
        onClick
      );
    };
  }, [onClick])
  return (
    <Fragment ref={fragmentRef}>
      {children}
    </Fragment>
  );
}

export default function App() {
  const [clicks, setClicks] = useState(0);

  return (
    <>
      <p>总点击次数：{clicks}</p>
      <ClickableFragment onClick={() => {
        setClicks(c => c + 1);
      }}>
        <button>按钮 A</button>
        <button>按钮 B</button>
        <button>按钮 C</button>
      </ClickableFragment>
    </>
  );
}
```

```json package.json hidden
{
  "dependencies": {
    "react": "canary",
    "react-dom": "canary",
    "react-scripts": "latest"
  }
}
```

</Sandpack>

`addEventListener` 调用会将监听器应用到 Fragment 的每个第一层 DOM 子节点。当前动态添加或移除子节点时，`FragmentInstance` 会自动添加或移除该监听器。

<DeepDive>

#### Fragment ref 会指向哪些子元素？ {/*which-children-does-a-fragment-ref-target*/}

`FragmentInstance` 会指向 Fragment 的**第一层宿主（DOM）子节点**。考虑这棵树：

```js
<Fragment ref={ref}>
  <div id="A" />
  <Wrapper>
    <div id="B">
      <div id="C" />
    </div>
  </Wrapper>
  <div id="D" />
</Fragment>
```

`Wrapper` 是一个 React 组件，所以 `FragmentInstance` 会穿过它去查找 DOM 节点。被指向的子节点是 `A`、`B` 和 `D`。`C` 不会被指向，因为它嵌套在 DOM 元素 `B` 内部。

像 `addEventListener`、`observeUsing` 和 `getClientRects` 这样的方法，作用于这些第一层 DOM 子节点。`focus` 和 `focusLast` 则不同——它们会以深度优先的方式搜索*所有*嵌套子节点，以找到可聚焦的元素。

</DeepDive>

---

### <CanaryBadge /> 在一组元素之间管理焦点 {/*managing-focus-across-elements*/}

Fragment `ref` 提供了 `focus`、`focusLast` 和 `blur` 方法，可作用于 Fragment 内的所有 DOM 节点：

<Sandpack>

```js
import { Fragment, useRef } from 'react';

function FormFields({ children }) {
  const fragmentRef = useRef(null);

  return (
    <>
      <div className="buttons">
        <button onClick={() => {
          fragmentRef.current.focus();
        }}>
          聚焦第一个
        </button>
        <button onClick={() => {
          fragmentRef.current.focusLast();
        }}>
          聚焦最后一个
        </button>
        <button onClick={() => {
          fragmentRef.current.blur();
        }}>
          取消聚焦
        </button>
      </div>
      <Fragment ref={fragmentRef}>
        {children}
      </Fragment>
    </>
  );
}

// 即使这些输入框是深层嵌套的，
// focus() 也会按深度优先搜索来找到它们。
export default function App() {
  return (
    <FormFields>
      <fieldset>
        <legend>Shipping</legend>
        <label>
          Street: <input name="street" />
        </label>
        <label>
          City: <input name="city" />
        </label>
      </fieldset>
    </FormFields>
  );
}
```

```css
.buttons {
  display: flex;
  gap: 8px;
  margin-bottom: 10px;
}

label {
  display: inline-block;
}
```

```json package.json hidden
{
  "dependencies": {
    "react": "canary",
    "react-dom": "canary",
    "react-scripts": "latest"
  }
}
```

</Sandpack>

调用 `focus()` 会聚焦 `street` 输入框——即使它嵌套在 `<fieldset>` 和 `<label>` 里面。`focus()` 会以深度优先的方式搜索所有嵌套子节点，而不只是 Fragment 的直接子节点。`focusLast()` 以相反的顺序执行相同的操作，而 `blur()` 会在当前聚焦的元素位于 Fragment 内时移除焦点。

---

### <CanaryBadge /> 将一组元素滚动到视图中 {/*scrolling-group-into-view*/}

使用 `scrollIntoView` 可以在不使用包装元素的情况下，将 Fragment 的子元素滚动到视图中。传入 `true`（或省略该参数）会将第一个子元素滚动到顶部。传入 `false` 会将最后一个子元素滚动到底部：

<Sandpack>

```js
import { Fragment, useRef } from 'react';

function ScrollableSection({ children }) {
  const fragmentRef = useRef(null);

  return (
    <>
      <div className="buttons">
        <button onClick={() => {
          fragmentRef.current.scrollIntoView();
        }}>
          滚动到顶部
        </button>
        <button onClick={() => {
          fragmentRef.current.scrollIntoView(false);
        }}>
          滚动到底部
        </button>
      </div>
      <div className="container">
        <Fragment ref={fragmentRef}>
          {children}
        </Fragment>
      </div>
    </>
  );
}

const items = [];
for (let i = 1; i <= 25; i++) {
  items.push('项目 ' + i);
}

export default function App() {
  return (
    <ScrollableSection>
      <h3>部分开头</h3>
      {items.map((item) => (
        <p key={item}>{item}</p>
      ))}
      <h3>部分结尾</h3>
    </ScrollableSection>
  );
}
```

```css
.buttons {
  display: flex;
  gap: 8px;
  margin-bottom: 10px;
}

.container {
  height: 200px;
  overflow-y: auto;
  border: 2px solid #c4c4c4;
  border-radius: 4px;
  padding: 10px;
}

h3 {
  margin: 4px 0;
  /* 内边距用于处理滚动时全局固定导航栏的偏移，例如 */
  padding-top: 4em;
  color: #1a73e8;
}

p {
  margin: 4px 0;
}
```

```json package.json hidden
{
  "dependencies": {
    "react": "canary",
    "react-dom": "canary",
    "react-scripts": "latest"
  }
}
```

</Sandpack>

---

### <CanaryBadge /> 在没有包装元素的情况下观察可见性 {/*observing-visibility-without-wrapper*/}

使用 `observeUsing` 将 `IntersectionObserver` 附加到 Fragment 的所有一级 DOM 子元素上。这让你可以在不要求子组件暴露 `ref` 或添加包装元素的情况下跟踪可见性：

<Sandpack>

```js
import {
  Fragment,
  useRef,
  useLayoutEffect,
  useState,
} from 'react';
import Card from './Card';

function VisibleGroup({ onVisibilityChange, children }) {
  const fragmentRef = useRef(null);

  useLayoutEffect(() => {
    const visibleElements = new Set();
    const observer = new IntersectionObserver(
      (entries) => {
        entries.forEach(e => {
          if (e.isIntersecting) {
            visibleElements.add(e.target);
          } else {
            visibleElements.delete(e.target);
          }
        });
        onVisibilityChange(visibleElements.size > 0);
      }
    );
    const fragmentInstance = fragmentRef.current;
    fragmentInstance.observeUsing(observer);
    return () => {
      fragmentInstance.unobserveUsing(observer);
    };
  }, [onVisibilityChange]);

  return (
    <Fragment ref={fragmentRef}>
      {children}
    </Fragment>
  );
}

export default function App() {
  const [isVisible, setIsVisible] = useState(true);

  return (
    <div className={isVisible ? 'page visible' : 'page'}>
      <div className="filler">向下滚动</div>
      <VisibleGroup onVisibilityChange={setIsVisible}>
        <Card title="第一部分" />
        <Card title="第二部分" />
      </VisibleGroup>
      <div className="filler">向上滚动</div>
    </div>
  );
}
```

```css
.page {
  transition: background 0.3s;
}

.page.visible {
  background: #d4edda;
}

.filler {
  height: 500px;
  display: flex;
  align-items: center;
  justify-content: center;
  color: #aaa;
  font-size: 14px;
}

.card {
  padding: 16px;
  background: white;
  border: 1px solid #ddd;
  border-radius: 8px;
  margin: 8px 16px;
  box-shadow: 0 1px 3px rgba(0,0,0,0.08);
  font-weight: 600;
  font-size: 14px;
}
```

```js src/Card.js hidden
export default function Card({ title }) {
  return <div className="card">{title}</div>;
}
```

```json package.json hidden
{
  "dependencies": {
    "react": "canary",
    "react-dom": "canary",
    "react-scripts": "latest"
  }
}
```

</Sandpack>

---

### <CanaryBadge /> 缓存全局 IntersectionObserver {/*caching-global-intersection-observer*/}

对于有许多 observer 的网站，一个常见的性能优化是：按配置共享一个 `IntersectionObserver`，并根据哪个元素发生交叉，将其条目路由到正确的回调。带有 `ref` 的 Fragment 通过 `reactFragments` 属性支持同样的模式。

带有 `ref` 的 Fragment 的每个一级 DOM 子元素都有一个 `reactFragments` 属性：一个包含该元素的 `FragmentInstance` 对象的 `Set`。当共享的 observer 触发时，你可以使用这个属性查找哪个 `FragmentInstance` 拥有发生交叉的元素，并执行正确的回调。

<Sandpack>

```js src/App.js active
import { useState, useCallback } from 'react';
import ObservedGroup from './ObservedGroup';
import Card from './Card';

export default function App() {
  const [bgColor, setBgColor] = useState(null);

  const onGreen = useCallback((entry) => {
    if (entry.isIntersecting) {
      setBgColor('#d4edda');
    }
  }, []);

  const onBlue = useCallback((entry) => {
    if (entry.isIntersecting) {
      setBgColor('#cce5ff');
    }
  }, []);

  return (
    <div className="page" style={{
      background: bgColor || 'white',
    }}>
      <div className="filler">向下滚动</div>
      <ObservedGroup onIntersection={onGreen}>
        <Card title="绿色区域" className="green" />
      </ObservedGroup>
      <div className="filler" />
      <ObservedGroup onIntersection={onBlue}>
        <Card title="蓝色区域" className="blue" />
      </ObservedGroup>
      <div className="filler">向上滚动</div>
    </div>
  );
}
```

```js src/ObservedGroup.js
import {
  Fragment,
  useRef,
  useLayoutEffect,
} from 'react';

const callbackMap = new WeakMap();
const observerCache = new Map();

function getOptionsKey(options) {
  const root = options?.root ?? null;
  const rootMargin = options?.rootMargin ?? '0px';
  const threshold = options?.threshold ?? 0;
  return `${rootMargin}|${threshold}`;
}

function getSharedObserver(
  fragmentInstance,
  onIntersection,
  options,
) {
  // 为
  // fragmentInstance 注册这个回调。
  const existing =
    callbackMap.get(fragmentInstance);
  callbackMap.set(
    fragmentInstance,
    existing
      ? [...existing, onIntersection]
      : [onIntersection],
  );

  const key = getOptionsKey(options);
  if (observerCache.has(key)) {
    return observerCache.get(key);
  }

  const observer = new IntersectionObserver(
    (entries) => {
      for (const entry of entries) {
        // 查找哪些 FragmentInstance 拥有
        // 这个元素。
        const fragmentInstances =
          entry.target.reactFragments;
        if (fragmentInstances) {
          for (const inst of fragmentInstances) {
            const callbacks =
              callbackMap.get(inst) || [];
            callbacks.forEach(cb => cb(entry));
          }
        }
      }
    },
    options,
  );

  observerCache.set(key, observer);
  return observer;
}

export default function ObservedGroup({
  onIntersection,
  options,
  children,
}) {
  const fragmentRef = useRef(null);

  useLayoutEffect(() => {
    const fragmentInstance = fragmentRef.current;
    const observer = getSharedObserver(
      fragmentInstance,
      onIntersection,
      options,
    );
    fragmentInstance.observeUsing(observer);
    return () => {
      fragmentInstance.unobserveUsing(observer);
      callbackMap.delete(fragmentInstance);
    };
  }, [onIntersection, options]);

  return (
    <Fragment ref={fragmentRef}>
      {children}
    </Fragment>
  );
}
```

```css
.page {
  transition: background 0.3s;
}

.filler {
  height: 500px;
  display: flex;
  align-items: center;
  justify-content: center;
  color: #aaa;
  font-size: 14px;
}

.card {
  padding: 16px;
  background: white;
  border: 1px solid #ddd;
  border-radius: 8px;
  margin: 0 16px;
  box-shadow: 0 1px 3px rgba(0,0,0,0.08);
  font-weight: 600;
  font-size: 14px;
}

.card.green {
  border-left: 3px solid #28a745;
}

.card.blue {
  border-left: 3px solid #007bff;
}
```

```js src/Card.js hidden
export default function Card({ title, className }) {
  return <div className={'card' + (className ? ' ' + className : '')}>{title}</div>;
}
```

```json package.json hidden
{
  "dependencies": {
    "react": "canary",
    "react-dom": "canary",
    "react-scripts": "latest"
  }
}
```

</Sandpack>

多个使用相同选项的 `ObservedGroup` 组件会复用一个 `IntersectionObserver`。当任一区域滚动进入视野时，共享的 observer 会触发，并使用 `reactFragments` 将条目路由到正确的回调。
