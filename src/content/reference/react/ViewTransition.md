---
title: <ViewTransition>
version: canary
---



<Intro>

<Canary>

**`<ViewTransition />` API 目前仅在 React 的 Canary 和 Experimental 渠道中可用。**

[在此了解有关 React 发布渠道的更多信息。](/community/versioning-policy#all-release-channels)

</Canary>

`<ViewTransition>` 让你可以使用 Transitions 和 Suspense 来为组件树添加动画。

```js
import {ViewTransition} from 'react';

<ViewTransition>
  <div>...</div>
</ViewTransition>
```

</Intro>

<InlineToc />

---

## Reference {/*reference*/}

### `<ViewTransition>` {/*viewtransition*/}

将组件树包裹在 `<ViewTransition>` 中即可为其添加动画：

```js
<ViewTransition>
  <Page />
</ViewTransition>
```

[下面查看更多示例。](#usage)

<DeepDive>

#### `<ViewTransition>` 是如何工作的？ {/*how-does-viewtransition-work*/}

在底层，React 会将 `view-transition-name` 应用到嵌套在 `<ViewTransition>` 组件内部的最近 DOM 节点的内联样式上。如果有多个同级 DOM 节点，例如 `<ViewTransition><div /><div /></ViewTransition>`，那么 React 会在名称后添加后缀以使每个名称唯一，但从概念上说它们属于同一个。React 不会急切地应用这些，而只会在该边界应该参与动画时才应用。

React 会自动在内部调用 `startViewTransition`，所以你不应该自己这么做。实际上，如果页面上还有其他东西正在运行 ViewTransition，React 会中断它。因此建议你使用 React 自身来协调这些操作。如果你过去通过其他方式触发 ViewTransitions，我们建议你迁移到内置方式。

如果已经有其他 React ViewTransitions 在运行，React 会等待它们完成后再开始下一个。不过重要的是，如果第一个动画运行期间又发生了多个更新，这些更新都会被批量合并为一个。如果你先启动 A->B，然后在此期间又有更新变成 C，再变成 D。当第一个 A->B 动画结束后，下一个动画将从 B->D 进行。

`getSnapshotBeforeUpdate` 生命周期会在 `startViewTransition` 之前被调用，并且某些 `view-transition-name` 会同时更新。

然后 React 调用 `startViewTransition`。在 `updateCallback` 内，React 将会：

- 将其变更应用到 DOM 并调用 `useInsertionEffect`。
- 等待字体加载完成。
- 调用 `componentDidMount`、`componentDidUpdate`、`useLayoutEffect` 和 refs。
- 等待任何未完成的 Navigation 完成。
- 然后 React 会测量布局中的任何变化，以查看哪些边界需要执行动画。

在 `startViewTransition` 的 ready Promise 解析之后，React 会恢复 `view-transition-name`。然后 React 将调用 `onEnter`、`onExit`、`onUpdate` 和 `onShare` 回调，以便对动画进行手动编程控制。这会发生在内置默认动画已经被计算之后。

如果在这个序列中间发生了 `flushSync`，那么 React 会跳过该 Transition，因为它依赖于能够同步完成。

在 `startViewTransition` 的 finished Promise 解析之后，React 会调用 `useEffect`。这可以防止它们干扰动画性能。不过，这并不能保证，因为如果动画运行期间又发生了另一个 `setState`，为了保持顺序保证，它仍然必须更早地调用 `useEffect`。

</DeepDive>

#### Props {/*props*/}

- **可选** `name`：一个字符串或对象。用于 shared element transitions 的 View Transition 名称。如果未提供，React 会为每个 View Transition 使用唯一名称，以防止意外动画。
- [View Transition Class](#view-transition-class) 属性。
- [View Transition Event](#view-transition-event) 属性。

#### Caveats {/*caveats*/}

- 仅将 `name` 用于 [shared element transitions](#animating-a-shared-element)。对于所有其他动画，React 会自动生成唯一名称，以防止意外动画。
- 默认情况下，`setState` 会立即更新，并不会激活 `<ViewTransition>`；只有包裹在 [Transition](/reference/react/useTransition)、[`<Suspense>`](/reference/react/Suspense) 或 `useDeferredValue` 中的更新才会激活 ViewTransition。
- `<ViewTransition>` 会创建一个可以移动、缩放并交叉淡化的图像。与你可能在 React Native 或 Motion 中见过的布局动画不同，这意味着其中并不是每个单独的 Element 都会对其位置进行动画。这可以带来更好的性能，以及与逐个动画化每个部分相比更连续、更平滑的动画体验。不过，这也可能会让一些本应自行移动的内容失去连续性。因此，你可能需要手动添加更多 `<ViewTransition>` 边界。
- 目前，`<ViewTransition>` 仅在 DOM 中有效。我们正在努力为 React Native 和其他平台添加支持。

#### Animation triggers {/*animation-triggers*/}

React 会自动决定触发哪种 View Transition 动画：

- `enter`：如果 `ViewTransition` 是本次 Transition 中第一个被插入的组件，那么它会被激活。
- `exit`：如果 `ViewTransition` 是本次 Transition 中第一个被删除的组件，那么它会被激活。
- `update`：如果 `ViewTransition` 内部有 React 正在执行的任何 DOM 变更（例如 prop 发生变化），或者由于紧邻的兄弟节点导致 `ViewTransition` 边界本身发生大小或位置变化。如果存在嵌套的 `ViewTransition`，那么变更会应用到它们而不是父级。
- `share`：如果一个带名称的 `ViewTransition` 位于被删除的子树中，而另一个同名的 `ViewTransition` 是同一次 Transition 中插入的子树的一部分，那么它们会形成 Shared Element Transition，并且会从被删除的那个动画到被插入的那个。

默认情况下，`<ViewTransition>` 会使用平滑的交叉淡化进行动画（浏览器默认的视图过渡）。

你可以通过为 `<ViewTransition>` 组件在每种触发类型上提供一个 [View Transition Class](#view-transition-class) 来自定义动画（参见 [Styling View Transitions](#styling-view-transitions)），或者使用 [ViewTransition Events](#view-transition-events) 并借助 [Web Animations API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Animations_API) 用 JavaScript 控制动画。

<Note>

#### 始终检查 `prefers-reduced-motion` {/*always-check-prefers-reduced-motion*/}

许多用户可能不希望页面上有动画。React 不会针对这种情况自动禁用动画。

我们建议始终使用 `@media (prefers-reduced-motion)` 媒体查询，根据用户偏好来禁用动画或减弱动画效果。

未来，CSS 库可能会在其预设中内置这一点。

</Note>

### View Transition Class {/*view-transition-class*/}

`<ViewTransition>` 提供了用于定义触发哪些动画的属性：

```js
<ViewTransition
  default="none"
  enter="slide-up"
  exit="slide-down"
/>
```

#### Props {/*view-transition-class-props*/}

- **可选** `enter`：`"auto"`、`"none"`、一个字符串，或一个对象。
- **可选** `exit`：`"auto"`、`"none"`、一个字符串，或一个对象。
- **可选** `update`：`"auto"`、`"none"`、一个字符串，或一个对象。
- **可选** `share`：`"auto"`、`"none"`、一个字符串，或一个对象。
- **可选** `default`：`"auto"`、`"none"`、一个字符串，或一个对象。

#### Caveats {/*view-transition-class-caveats*/}

- 如果 `default` 是 `"none"`，那么除非显式列出，否则所有其他触发类型都会关闭。

#### Values {/*view-transition-values*/}

View Transition class 的值可以是：
- `auto`：默认值。使用浏览器默认动画。
- `none`：禁用此类型的动画。
- `<classname>`：用于 [自定义 View Transitions](#styling-view-transitions) 的自定义 CSS 类名。

对象值可以是一个带字符串键的对象，其值为 `auto`、`none` 或自定义 className：
- `{[type]: value}`：如果动画匹配 [Transition Type](/reference/react/addTransitionType)，则应用 `value`。
- `{default: value}`：如果没有匹配到任何 [Transition Type](/reference/react/addTransitionType)，则应用默认值。

例如，你可以将 ViewTransition 定义为：

```js
<ViewTransition
  /* 关闭下面未定义的任何动画 */
  default="none"
  enter={{
    /* 为 Transition Type `forward` 应用 slide-in */
    "forward": 'slide-in',
    /* 否则使用浏览器默认动画 */
    "default": 'auto'
  }}
  /* 退出动画使用浏览器默认值*/
  exit="auto"
  /* 为更新应用自定义的 `cross-fade` 类 */
  update="cross-fade"
>
```

有关如何为自定义动画定义 CSS 类，请参见 [Styling View Transitions](#styling-view-transitions)。

---

### View Transition Event {/*view-transition-event*/}

View Transition Events 允许你使用 [Web Animations API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Animations_API) 通过 JavaScript 控制动画：

```js
<ViewTransition
  onEnter={instance => {/* ... */}}
  onExit={instance => {/* ... */}}
/>
```

#### Props {/*view-transition-event-props*/}

- **可选** `onEnter`：在触发 `"enter"` 动画时调用。
- **可选** `onExit`：在触发 `"exit"` 动画时调用。
- **可选** `onShare`：在触发 `"share"` 动画时调用。
- **可选** `onUpdate`：在触发 `"update"` 动画时调用。


#### Caveats {/*view-transition-event-caveats*/}
- 每次 Transition 中，每个 `<ViewTransition>` 只会触发一个事件。`onShare` 的优先级高于 `onEnter` 和 `onExit`。
- 每个事件都应该返回一个**清理函数**。当 View Transition 完成时会调用清理函数，从而允许你取消或清理任何动画。

#### Arguments {/*view-transition-event-arguments*/}

每个事件都会接收两个参数：

- `instance`：一个 View Transition 实例，提供对视图过渡 [伪元素](https://developer.mozilla.org/en-US/docs/Web/API/View_Transition_API/Using#the_view_transition_process) 的访问
  - `old`：`::view-transition-old` 伪元素。
  - `new`：`::view-transition-new` 伪元素。
  - `name`：此边界的 `view-transition-name` 字符串。
  - `group`：`::view-transition-group` 伪元素。
  - `imagePair`：`::view-transition-image-pair` 伪元素。
- `types`：动画中包含的 [Transition Types](/reference/react/addTransitionType) 的 `Array<string>`。如果未指定类型，则为空数组。

例如，你可以定义一个通过 JavaScript 驱动动画的 `onEnter` 事件：

```js
<ViewTransition
  onEnter={(instance, types) => {
    const anim = instance.new.animate([{opacity: 0}, {opacity: 1}], {
      duration: 500,
    });
    return () => anim.cancel();
  }}>
  <div>...</div>
</ViewTransition>
```

更多示例请参见 [使用 JavaScript 进行动画](#animating-with-javascript)。

---

## 样式化视图过渡 {/*styling-view-transitions*/}

<Note>

在 Web 上许多关于 View Transitions 的早期示例中，你可能见过使用 [`view-transition-name`](https://developer.mozilla.org/en-US/docs/Web/CSS/view-transition-name)，然后再用 `::view-transition-...(my-name)` 选择器来进行样式设置。我们不建议这样做。相反，我们通常建议改用 View Transition Class。

</Note>

要为 `<ViewTransition>` 自定义动画，你可以把 View Transition Class 提供给某个激活属性。View Transition Class 是一个 CSS 类名，React 会在 ViewTransition 激活时将其应用到子元素上。

例如，要自定义 “enter” 动画，可以把一个类名提供给 `enter` 属性：

```js
<ViewTransition enter="slide-in">
```

当 `<ViewTransition>` 激活 “enter” 动画时，React 会添加类名 `slide-in`。然后你可以使用 [view transition 伪选择器](https://developer.mozilla.org/en-US/docs/Web/API/View_Transition_API#pseudo-elements) 来引用这个类，从而构建可复用的动画：

```css
::view-transition-group(.slide-in) {
}
::view-transition-old(.slide-in) {
}
::view-transition-new(.slide-in) {
}
```

未来，CSS 库可能会使用 View Transition Classes 内置这些动画，让它更容易使用。

---

## 用法 {/*usage*/}

### 为进入/退出时的元素添加动画 {/*animating-an-element-on-enter*/}

当组件在一个 transition 中添加或移除 `<ViewTransition>` 时，会触发 Enter/Exit 转场：

```js {3}
function Child() {
  return (
    <ViewTransition enter="auto" exit="auto" default="none">
      <div>Hi</div>
    </ViewTransition>
  );
}

function Parent() {
  const [show, setShow] = useState();
  if (show) {
    return <Child />;
  }
  return null;
}
```

当调用 `setShow` 时，`show` 会切换为 `true`，并渲染 `Child` 组件。当在 `startTransition` 内部调用 `setShow`，并且 `Child` 在任何其他 DOM 节点之前先渲染了一个 `ViewTransition` 时，就会触发 `enter` 动画。

当 `show` 再次切换为 `false` 时，会触发 `exit` 动画。

<Sandpack>

```js src/Video.js hidden
function Thumbnail({video, children}) {
  return (
    <div
      aria-hidden="true"
      tabIndex={-1}
      className={`thumbnail ${video.image}`}
    />
  );
}

export function Video({video}) {
  return (
    <div className="video">
      <div className="link">
        <Thumbnail video={video}></Thumbnail>
        <div className="info">
          <div className="video-title">{video.title}</div>
          <div className="video-description">{video.description}</div>
        </div>
      </div>
    </div>
  );
}
```

```js
import {ViewTransition, useState, startTransition} from 'react';
import {Video} from './Video';
import videos from './data';

function Item() {
  return (
    <ViewTransition enter="auto" exit="auto" default="none">
      <Video video={videos[0]} />
    </ViewTransition>
  );
}

export default function Component() {
  const [showItem, setShowItem] = useState(false);
  return (
    <>
      <button
        onClick={() => {
          startTransition(() => {
            setShowItem((prev) => !prev);
          });
        }}>
        {showItem ? '➖' : '➕'}
      </button>

      {showItem ? <Item /> : null}
    </>
  );
}
```

```js src/data.js hidden
export default [
  {
    id: '1',
    title: 'First video',
    description: 'Video description',
    image: 'blue',
  },
];
```

```css
#root {
  display: flex;
  flex-direction: column;
  align-items: center;
  min-height: 200px;
}
button {
  border: none;
  border-radius: 50%;
  width: 50px;
  height: 50px;
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #f0f8ff;
  color: white;
  font-size: 20px;
  cursor: pointer;
  transition: background-color 0.3s, border 0.3s;
}
button:hover {
  border: 2px solid #ccc;
  background-color: #e0e8ff;
}
.thumbnail {
  position: relative;
  aspect-ratio: 16 / 9;
  display: flex;
  overflow: hidden;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  border-radius: 0.5rem;
  outline-offset: 2px;
  width: 8rem;
  vertical-align: middle;
  background-color: #ffffff;
  background-size: cover;
  user-select: none;
}
.thumbnail.blue {
  background-image: conic-gradient(at top right, #c76a15, #087ea4, #2b3491);
}
.video {
  display: flex;
  flex-direction: row;
  gap: 0.75rem;
  align-items: center;
  margin-top: 1em;
}
.video .link {
  display: flex;
  flex-direction: row;
  flex: 1 1 0;
  gap: 0.125rem;
  outline-offset: 4px;
  cursor: pointer;
}
.video .info {
  display: flex;
  flex-direction: column;
  justify-content: center;
  margin-left: 8px;
  gap: 0.125rem;
}
.video .info:hover {
  text-decoration: underline;
}
.video-title {
  font-size: 15px;
  line-height: 1.25;
  font-weight: 700;
  color: #23272f;
}
.video-description {
  color: #5e687e;
  font-size: 13px;
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

<Pitfall>

#### 只有顶层的 ViewTransition 会在退出/进入时执行动画 {/*only-top-level-viewtransition-animates-on-exit-enter*/}

只有当 `<ViewTransition>` 放在任何 DOM 节点之前时，它才会激活 exit/enter。

如果 `<ViewTransition>` 上方有一个 `<div>`，则不会触发 exit/enter 动画：

```js [3, 5]
function Item() {
  return (
    <div> {/* 🚩<ViewTransition> 上方的 <div> 会破坏 exit/enter */}
      <ViewTransition enter="auto" exit="auto" default="none">
        <Video video={videos[0]} />
      </ViewTransition>
    </div>
  );
}
```

这个限制可以防止出现微妙的 bug，即动画过多或过少。

</Pitfall>

---

### 使用 Activity 为进入/退出添加动画 {/*animating-enter-exit-with-activity*/}

如果你想在保留组件状态的同时让组件进出动画化，或者为了动画而预渲染内容，可以使用 [`<Activity>`](/reference/react/Activity)。当 `<Activity>` 内部的 `<ViewTransition>` 变为可见时，会激活 `enter` 动画；当它变为隐藏时，会激活 `exit` 动画：

```js
<Activity mode={isVisible ? 'visible' : 'hidden'}>
  <ViewTransition enter="auto" exit="auto">
    <Counter />
  </ViewTransition>
</Activity>

```

在这个示例中，`Counter` 有一个带内部状态的计数器。你可以尝试增加计数器，然后隐藏它，再重新显示它。侧边栏进出时，计数器的值会被保留：

<Sandpack>

```js
import { Activity, ViewTransition, useState, startTransition } from 'react';

export default function App() {
  const [show, setShow] = useState(true);
  return (
    <div className="layout">
      <Toggle show={show} setShow={setShow} />
      <Activity mode={show ? 'visible' : 'hidden'}>
        <ViewTransition enter="auto" exit="auto" default="none">
          <Counter />
        </ViewTransition>
      </Activity>
    </div>
  );
}
function Toggle({show, setShow}) {
  return (
    <button
      className="toggle"
      onClick={() => {
        startTransition(() => {
          setShow(s => !s);
        });
      }}>
      {show ? '隐藏' : '显示'}
    </button>
  )
}
function Counter() {
  const [count, setCount] = useState(0);
  return (
    <div className="counter">
      <h2>计数器</h2>
      <p>计数：{count}</p>
      <button onClick={() => setCount(count + 1)}>
        增加
      </button>
    </div>
  );
}

```

```css
.layout {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  gap: 10px;
  min-height: 200px;
}
.counter {
  padding: 15px;
  background: #f0f4f8;
  border-radius: 8px;
  width: 200px;
}
.counter h2 {
  margin: 0 0 10px 0;
  font-size: 16px;
}
.counter p {
  margin: 0 0 10px 0;
}
.toggle {
  padding: 8px 16px;
  border: 1px solid #ccc;
  border-radius: 6px;
  background: #f0f8ff;
  cursor: pointer;
  font-size: 14px;
}
.toggle:hover {
  background: #e0e8ff;
}
.counter button {
  padding: 4px 12px;
  border: 1px solid #ccc;
  border-radius: 4px;
  background: white;
  cursor: pointer;
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

如果没有 `<Activity>`，每次侧边栏重新出现时，计数器都会重置为 `0`。

---

### 为共享元素添加动画 {/*animating-a-shared-element*/}

通常，我们不建议给 `<ViewTransition>` 指定名称，而是让 React 自动分配名称。你可能想指定名称的原因，是为了在一个树卸载、另一个树同时挂载时，在完全不同的组件之间做动画，以保持连续性。

```js
<ViewTransition name={UNIQUE_NAME}>
  <Child />
</ViewTransition>
```

当一个树卸载、另一个树挂载时，如果在卸载树和挂载树中存在一对相同名称的元素，它们都会触发 “share” 动画。动画会从卸载侧过渡到挂载侧。

与 exit/enter 动画不同，这种动画可以位于已删除/已挂载树的很深层内部。如果某个 `<ViewTransition>` 本来也符合 exit/enter 条件，那么 “share” 动画会优先。

如果 Transition 先卸载了一侧，然后导致在最终新名称挂载之前显示了 `<Suspense>` 回退内容，那么不会发生共享元素转场。

<Sandpack>

```js
import {ViewTransition, useState, startTransition} from 'react';
import {Video, Thumbnail, FullscreenVideo} from './Video';
import videos from './data';

export default function Component() {
  const [fullscreen, setFullscreen] = useState(false);
  if (fullscreen) {
    return (
      <FullscreenVideo
        video={videos[0]}
        onExit={() => startTransition(() => setFullscreen(false))}
      />
    );
  }
  return (
    <Video
      video={videos[0]}
      onClick={() => startTransition(() => setFullscreen(true))}
    />
  );
}
```

```js src/Video.js
import {ViewTransition} from 'react';

const THUMBNAIL_NAME = 'video-thumbnail';

export function Thumbnail({video, children}) {
  return (
    <ViewTransition name={THUMBNAIL_NAME}>
      <div
        aria-hidden="true"
        tabIndex={-1}
        className={`thumbnail ${video.image}`}
      />
    </ViewTransition>
  );
}

export function Video({video, onClick}) {
  return (
    <div className="video">
      <div className="link" onClick={onClick}>
        <Thumbnail video={video} />
        <div className="info">
          <div className="video-title">{video.title}</div>
          <div className="video-description">{video.description}</div>
        </div>
      </div>
    </div>
  );
}

export function FullscreenVideo({video, onExit}) {
  return (
    <div className="fullscreenLayout">
      <ViewTransition name={THUMBNAIL_NAME}>
        <div
          aria-hidden="true"
          tabIndex={-1}
          className={`thumbnail ${video.image} fullscreen`}
        />
        <button className="close-button" onClick={onExit}>
          ✖
        </button>
      </ViewTransition>
    </div>
  );
}
```

```js src/data.js hidden
export default [
  {
    id: '1',
    title: 'First video',
    description: 'Video description',
    image: 'blue',
  },
];
```

```css
#root {
  display: flex;
  flex-direction: column;
  align-items: center;
  height: 300px;
}
button {
  border: none;
  border-radius: 50%;
  width: 50px;
  height: 50px;
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #f0f8ff;
  color: white;
  font-size: 20px;
  cursor: pointer;
  transition: background-color 0.3s, border 0.3s;
}
button:hover {
  border: 2px solid #ccc;
  background-color: #e0e8ff;
}
.thumbnail {
  position: relative;
  aspect-ratio: 16 / 9;
  display: flex;
  overflow: hidden;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  border-radius: 0.5rem;
  outline-offset: 2px;
  width: 8rem;
  vertical-align: middle;
  background-color: #ffffff;
  background-size: cover;
  user-select: none;
}
.thumbnail.blue {
  background-image: conic-gradient(at top right, #c76a15, #087ea4, #2b3491);
}
.thumbnail.red {
  background-image: conic-gradient(at top right, #c76a15, #a6423a, #2b3491);
}
.thumbnail.fullscreen {
  width: 100%;
}
.video {
  display: flex;
  flex-direction: row;
  gap: 0.75rem;
  align-items: center;
  margin-top: 1em;
}
.video .link {
  display: flex;
  flex-direction: row;
  flex: 1 1 0;
  gap: 0.125rem;
  outline-offset: 4px;
  cursor: pointer;
}
.video .info {
  display: flex;
  flex-direction: column;
  justify-content: center;
  margin-left: 8px;
  gap: 0.125rem;
}
.video .info:hover {
  text-decoration: underline;
}
.video-title {
  font-size: 15px;
  line-height: 1.25;
  font-weight: 700;
  color: #23272f;
}
.video-description {
  color: #5e687e;
  font-size: 13px;
}
.fullscreenLayout {
  position: relative;
  height: 100%;
  width: 100%;
}
.close-button {
  position: absolute;
  top: 10px;
  right: 10px;
  color: black;
}
@keyframes progress-animation {
  from {
    width: 0;
  }
  to {
    width: 100%;
  }
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

<Note>

如果一对中的挂载侧或卸载侧有一侧在视口之外，那么就不会形成这对元素。这可以确保当某个元素被滚动到视口外时，不会让它飞入或飞出视口。相反，它会被当作普通的独立 enter/exit 处理。

如果是同一个组件实例改变了位置，则不会发生这种情况，此时会触发 “update”。这些动画不受某一侧是否在视口外的影响。

有一个已知情况：如果一个深层嵌套、已卸载的 `<ViewTransition>` 在视口内，而挂载侧不在视口内，那么即使它很深层，卸载侧仍会作为自身的 “exit” 动画执行，而不是作为父级动画的一部分。

</Note>

<Pitfall>

整个应用中，任何时候都必须只有一个具有相同名称的内容处于挂载状态。因此，使用唯一的命名空间来命名非常重要，以避免冲突。为了确保这一点，你可能希望在单独的模块中定义一个常量并导入它。

```js
export const MY_NAME = "my-globally-unique-name";
import {MY_NAME} from './shared-name';
...
<ViewTransition name={MY_NAME}>
```

</Pitfall>

---

### 为列表中的项重排添加动画 {/*animating-reorder-of-items-in-a-list*/}

```js
items.map((item) => <Component key={item.id} item={item} />);
```

在重排列表且不更新内容时，如果列表中的 `<ViewTransition>` 位于某个 DOM 节点外部，则每个 `<ViewTransition>` 都会触发 “update” 动画。类似于 enter/exit 动画。

这意味着下面的代码会触发这个 `<ViewTransition>` 的动画：

```js
function Component() {
  return (
    <ViewTransition>
      <div>...</div>
    </ViewTransition>
  );
}
```

<Sandpack>

```js src/Video.js hidden
function Thumbnail({video}) {
  return (
    <div
      aria-hidden="true"
      tabIndex={-1}
      className={`thumbnail ${video.image}`}
    />
  );
}

export function Video({video}) {
  return (
    <div className="video">
      <div className="link">
        <Thumbnail video={video}></Thumbnail>
        <div className="info">
          <div className="video-title">{video.title}</div>
          <div className="video-description">{video.description}</div>
        </div>
      </div>
    </div>
  );
}
```

```js
import {ViewTransition, useState, startTransition} from 'react';
import {Video} from './Video';
import videos from './data';

export default function Component() {
  const [orderedVideos, setOrderedVideos] = useState(videos);
  const reorder = () => {
    startTransition(() => {
      setOrderedVideos((prev) => {
        return [...prev.sort(() => Math.random() - 0.5)];
      });
    });
  };
  return (
    <>
      <button onClick={reorder}>🎲</button>
      <div className="listContainer">
        {orderedVideos.map((video, i) => {
          return (
            <ViewTransition key={video.title}>
              <Video video={video} />
            </ViewTransition>
          );
        })}
      </div>
    </>
  );
}
```

```js src/data.js hidden
export default [
  {
    id: '1',
    title: 'First video',
    description: 'Video description',
    image: 'blue',
  },
  {
    id: '2',
    title: 'Second video',
    description: 'Video description',
    image: 'red',
  },
  {
    id: '3',
    title: 'Third video',
    description: 'Video description',
    image: 'green',
  },
  {
    id: '4',
    title: 'Fourth video',
    description: 'Video description',
    image: 'purple',
  },
];
```

```css
#root {
  display: flex;
  flex-direction: column;
  align-items: center;
  min-height: 150px;
}
button {
  border: none;
  border-radius: 50%;
  width: 50px;
  height: 50px;
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #f0f8ff;
  color: white;
  font-size: 20px;
  cursor: pointer;
  transition: background-color 0.3s, border 0.3s;
}
button:hover {
  border: 2px solid #ccc;
  background-color: #e0e8ff;
}
.thumbnail {
  position: relative;
  aspect-ratio: 16 / 9;
  display: flex;
  overflow: hidden;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  border-radius: 0.5rem;
  outline-offset: 2px;
  width: 8rem;
  vertical-align: middle;
  background-color: #ffffff;
  background-size: cover;
  user-select: none;
}
.thumbnail.blue {
  background-image: conic-gradient(at top right, #c76a15, #087ea4, #2b3491);
}
.thumbnail.red {
  background-image: conic-gradient(at top right, #c76a15, #a6423a, #2b3491);
}
.thumbnail.green {
  background-image: conic-gradient(at top right, #c76a15, #388f7f, #2b3491);
}
.thumbnail.purple {
  background-image: conic-gradient(at top right, #c76a15, #575fb7, #2b3491);
}
.video {
  display: flex;
  flex-direction: row;
  gap: 0.75rem;
  align-items: center;
  margin-top: 1em;
}
.video .link {
  display: flex;
  flex-direction: row;
  flex: 1 1 0;
  gap: 0.125rem;
  outline-offset: 4px;
}
.video .info {
  display: flex;
  flex-direction: column;
  justify-content: center;
  margin-left: 8px;
  gap: 0.125rem;
}
.video .info:hover {
  text-decoration: underline;
}
.video-title {
  font-size: 15px;
  line-height: 1.25;
  font-weight: 700;
  color: #23272f;
}
.video-description {
  color: #5e687e;
  font-size: 13px;
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

不过，这不会为每个单独项分别添加动画：

```js
function Component() {
  return (
    <div>
      <ViewTransition>...</ViewTransition>
    </div>
  );
}
```

相反，任何父级 `<ViewTransition>` 都会进行交叉淡入淡出。如果没有父级 `<ViewTransition>`，那么这种情况下就不会有动画。

<Sandpack>

```js src/Video.js hidden
function Thumbnail({video}) {
  return (
    <div
      aria-hidden="true"
      tabIndex={-1}
      className={`thumbnail ${video.image}`}
    />
  );
}

export function Video({video}) {
  return (
    <div className="video">
      <div className="link">
        <Thumbnail video={video}></Thumbnail>
        <div className="info">
          <div className="video-title">{video.title}</div>
          <div className="video-description">{video.description}</div>
        </div>
      </div>
    </div>
  );
}
```

```js
import {ViewTransition, useState, startTransition} from 'react';
import {Video} from './Video';
import videos from './data';

export default function Component() {
  const [orderedVideos, setOrderedVideos] = useState(videos);
  const reorder = () => {
    startTransition(() => {
      setOrderedVideos((prev) => {
        return [...prev.sort(() => Math.random() - 0.5)];
      });
    });
  };
  return (
    <>
      <button onClick={reorder}>🎲</button>
      <ViewTransition>
        <div className="listContainer">
          {orderedVideos.map((video, i) => {
            return <Video video={video} key={video.title} />;
          })}
        </div>
      </ViewTransition>
    </>
  );
}
```

```js src/data.js hidden
export default [
  {
    id: '1',
    title: 'First video',
    description: 'Video description',
    image: 'blue',
  },
  {
    id: '2',
    title: 'Second video',
    description: 'Video description',
    image: 'red',
  },
  {
    id: '3',
    title: 'Third video',
    description: 'Video description',
    image: 'green',
  },
  {
    id: '4',
    title: 'Fourth video',
    description: 'Video description',
    image: 'purple',
  },
];
```

```css
#root {
  display: flex;
  flex-direction: column;
  align-items: center;
  min-height: 150px;
}
button {
  border: none;
  border-radius: 50%;
  width: 50px;
  height: 50px;
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #f0f8ff;
  color: white;
  font-size: 20px;
  cursor: pointer;
  transition: background-color 0.3s, border 0.3s;
}
button:hover {
  border: 2px solid #ccc;
  background-color: #e0e8ff;
}
.thumbnail {
  position: relative;
  aspect-ratio: 16 / 9;
  display: flex;
  overflow: hidden;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  border-radius: 0.5rem;
  outline-offset: 2px;
  width: 8rem;
  vertical-align: middle;
  background-color: #ffffff;
  background-size: cover;
  user-select: none;
}
.thumbnail.blue {
  background-image: conic-gradient(at top right, #c76a15, #087ea4, #2b3491);
}
.thumbnail.red {
  background-image: conic-gradient(at top right, #c76a15, #a6423a, #2b3491);
}
.thumbnail.green {
  background-image: conic-gradient(at top right, #c76a15, #388f7f, #2b3491);
}
.thumbnail.purple {
  background-image: conic-gradient(at top right, #c76a15, #575fb7, #2b3491);
}
.video {
  display: flex;
  flex-direction: row;
  gap: 0.75rem;
  align-items: center;
  margin-top: 1em;
}
.video .link {
  display: flex;
  flex-direction: row;
  flex: 1 1 0;
  gap: 0.125rem;
  outline-offset: 4px;
}
.video .info {
  display: flex;
  flex-direction: column;
  justify-content: center;
  margin-left: 8px;
  gap: 0.125rem;
}
.video .info:hover {
  text-decoration: underline;
}
.video-title {
  font-size: 15px;
  line-height: 1.25;
  font-weight: 700;
  color: #23272f;
}
.video-description {
  color: #5e687e;
  font-size: 13px;
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

这意味着，如果你希望让 Component 自己控制其重排动画，那么在列表中最好避免使用包裹元素：

```
items.map(item => <div><Component key={item.id} item={item} /></div>)
```

上面的规则也适用于：如果某个项更新导致尺寸变化，从而引起兄弟项也发生尺寸变化，那么它的兄弟 `<ViewTransition>` 也会被动画化，但前提是它们必须是直接兄弟。

这意味着在一次会导致大量重新布局的更新期间，它不会单独为页面上的每个 `<ViewTransition>` 添加动画。那样会产生大量嘈杂的动画，分散对实际变化的注意力。因此，React 对单个动画何时触发采取了更保守的策略。

<Pitfall>

在重排列表时，正确使用 key 来保留身份非常重要。看起来你也许可以使用 “name” 这种共享元素转场来实现重排动画，但如果一侧在视口之外，这种方式就不会触发。要为重排添加动画，你通常希望表现出它移动到了视口之外的位置。

</Pitfall>

---

### 为 Suspense 内容添加动画 {/*animating-from-suspense-content*/}

和任何 Transition 一样，React 会等待数据和新的 CSS（`<link rel="stylesheet" precedence="...">`）就绪后才运行动画。除此之外，ViewTransitions 还会在动画开始前最多等待 500ms 让新字体加载完成，以避免字体稍后出现闪烁。出于同样原因，被 ViewTransition 包裹的图片也会等待图片加载完成。

如果它位于一个新的 Suspense 边界实例内，那么会先显示 fallback。等 Suspense 边界完全加载后，它会触发 `<ViewTransition>`，将内容的显示过程做动画化。

根据你放置 `<ViewTransition>` 的位置，有两种为 Suspense 边界添加动画的方式：

**更新：**

```
<ViewTransition>
  <Suspense fallback={<A />}>
    <B />
  </Suspense>
</ViewTransition>
```

在这个场景中，当内容从 A 变为 B 时，它会被视为 “update”，并在适当时应用该类。A 和 B 会获得相同的 view-transition-name，因此默认表现为交叉淡入淡出。

<Sandpack>

```js src/Video.js hidden
function Thumbnail({video, children}) {
  return (
    <div
      aria-hidden="true"
      tabIndex={-1}
      className={`thumbnail ${video.image}`}
    />
  );
}

export function Video({video}) {
  return (
    <div className="video">
      <div className="link">
        <Thumbnail video={video}></Thumbnail>
        <div className="info">
          <div className="video-title">{video.title}</div>
          <div className="video-description">{video.description}</div>
        </div>
      </div>
    </div>
  );
}

export function VideoPlaceholder() {
  const video = {image: 'loading'};
  return (
    <div className="video">
      <div className="link">
        <Thumbnail video={video}></Thumbnail>
        <div className="info">
          <div className="video-title loading" />
          <div className="video-description loading" />
        </div>
      </div>
    </div>
  );
}
```

```js
import {ViewTransition, useState, startTransition, Suspense} from 'react';
import {Video, VideoPlaceholder} from './Video';
import {useLazyVideoData} from './data';

function LazyVideo() {
  const video = useLazyVideoData();
  return <Video video={video} />;
}

export default function Component() {
  const [showItem, setShowItem] = useState(false);
  return (
    <>
      <button
        onClick={() => {
          startTransition(() => {
            setShowItem((prev) => !prev);
          });
        }}>
        {showItem ? '➖' : '➕'}
      </button>
      {showItem ? (
        <ViewTransition>
          <Suspense fallback={<VideoPlaceholder />}>
            <LazyVideo />
          </Suspense>
        </ViewTransition>
      ) : null}
    </>
  );
}
```

```js src/data.js hidden
import {use} from 'react';

let cache = null;

function fetchVideo() {
  if (!cache) {
    cache = new Promise((resolve) => {
      setTimeout(() => {
        resolve({
          id: '1',
          title: 'First video',
          description: 'Video description',
          image: 'blue',
        });
      }, 1000);
    });
  }
  return cache;
}

export function useLazyVideoData() {
  return use(fetchVideo());
}
```

```css
#root {
  display: flex;
  flex-direction: column;
  align-items: center;
  min-height: 200px;
}
button {
  border: none;
  border-radius: 50%;
  width: 50px;
  height: 50px;
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #f0f8ff;
  color: white;
  font-size: 20px;
  cursor: pointer;
  transition: background-color 0.3s, border 0.3s;
}
button:hover {
  border: 2px solid #ccc;
  background-color: #e0e8ff;
}
.thumbnail {
  position: relative;
  aspect-ratio: 16 / 9;
  display: flex;
  overflow: hidden;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  border-radius: 0.5rem;
  outline-offset: 2px;
  width: 8rem;
  vertical-align: middle;
  background-color: #ffffff;
  background-size: cover;
  user-select: none;
}
.thumbnail.blue {
  background-image: conic-gradient(at top right, #c76a15, #087ea4, #2b3491);
}
.loading {
  background-image: linear-gradient(
    90deg,
    rgba(173, 216, 230, 0.3) 25%,
    rgba(135, 206, 250, 0.5) 50%,
    rgba(173, 216, 230, 0.3) 75%
  );
  background-size: 200% 100%;
  animation: shimmer 1.5s infinite;
}
@keyframes shimmer {
  0% {
    background-position: -200% 0;
  }
  100% {
    background-position: 200% 0;
  }
}
.video {
  display: flex;
  flex-direction: row;
  gap: 0.75rem;
  align-items: center;
  margin-top: 1em;
}
.video .link {
  display: flex;
  flex-direction: row;
  flex: 1 1 0;
  gap: 0.125rem;
  outline-offset: 4px;
  cursor: pointer;
}
.video .info {
  display: flex;
  flex-direction: column;
  justify-content: center;
  margin-left: 8px;
  gap: 0.125rem;
}
.video .info:hover {
  text-decoration: underline;
}
.video-title {
  font-size: 15px;
  line-height: 1.25;
  font-weight: 700;
  color: #23272f;
}
.video-title.loading {
  height: 20px;
  width: 80px;
  border-radius: 0.5rem;
}
.video-description {
  color: #5e687e;
  font-size: 13px;
  border-radius: 0.5rem;
}
.video-description.loading {
  height: 15px;
  width: 100px;
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

**Enter/Exit：**

```
<Suspense fallback={<ViewTransition><A /></ViewTransition>}>
  <ViewTransition><B /></ViewTransition>
</Suspense>
```

在这个场景中，这两个是彼此独立的 ViewTransition 实例，各自拥有自己的 `view-transition-name`。这会被视为 `<A>` 的 “exit” 和 `<B>` 的 “enter”。

你可以根据自己选择放置 `<ViewTransition>` 边界的位置，获得不同效果。

---

### 取消某个动画 {/*opting-out-of-an-animation*/}

有时你会包裹一个较大的现有组件，比如整个页面，并希望为某些更新添加动画，例如切换主题。然而，你并不希望整个页面中所有更新都在发生变化时自动交叉淡入淡出，尤其是在你逐步添加更多动画时。

你可以使用 “none” 类来取消某个动画。通过将子元素包裹在 “none” 中，你可以禁用它们的更新动画，而父级仍然会触发动画。

```js
<ViewTransition>
  <div className={theme}>
    <ViewTransition update="none">{children}</ViewTransition>
  </div>
</ViewTransition>
```

这只会在主题发生变化时添加动画，而不会在仅子元素更新时添加动画。子元素仍然可以通过它们自己的 `<ViewTransition>` 再次启用动画，但这至少又变回了手动控制。

---

### 自定义动画 {/*customizing-animations*/}

默认情况下，`<ViewTransition>` 包含浏览器提供的默认交叉淡入淡出。

要自定义动画，你可以为 `<ViewTransition>` 组件提供属性，以便根据 `<ViewTransition>` 的激活方式指定要使用的动画。

例如，我们可以放慢默认的交叉淡出动画：

```js
<ViewTransition default="slow-fade">
  <Video />
</ViewTransition>
```

并使用 view transition classes 在 CSS 中定义 slow-fade：

```css
::view-transition-old(.slow-fade) {
  animation-duration: 500ms;
}

::view-transition-new(.slow-fade) {
  animation-duration: 500ms;
}
```

<Sandpack>

```js src/Video.js hidden
function Thumbnail({video, children}) {
  return (
    <div
      aria-hidden="true"
      tabIndex={-1}
      className={`thumbnail ${video.image}`}
    />
  );
}

export function Video({video}) {
  return (
    <div className="video">
      <div className="link">
        <Thumbnail video={video}></Thumbnail>

        <div className="info">
          <div className="video-title">{video.title}</div>
          <div className="video-description">{video.description}</div>
        </div>
      </div>
    </div>
  );
}
```

```js
import {ViewTransition, useState, startTransition} from 'react';
import {Video} from './Video';
import videos from './data';

function Item() {
  return (
    <ViewTransition default="slow-fade">
      <Video video={videos[0]} />
    </ViewTransition>
  );
}

export default function Component() {
  const [showItem, setShowItem] = useState(false);
  return (
    <>
      <button
        onClick={() => {
          startTransition(() => {
            setShowItem((prev) => !prev);
          });
        }}>
        {showItem ? '➖' : '➕'}
      </button>

      {showItem ? <Item /> : null}
    </>
  );
}
```

```js src/data.js hidden
export default [
  {
    id: '1',
    title: 'First video',
    description: 'Video description',
    image: 'blue',
  },
];
```

```css
::view-transition-old(.slow-fade) {
  animation-duration: 500ms;
}

::view-transition-new(.slow-fade) {
  animation-duration: 500ms;
}

#root {
  display: flex;
  flex-direction: column;
  align-items: center;
  min-height: 200px;
}
button {
  border: none;
  border-radius: 50%;
  width: 50px;
  height: 50px;
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #f0f8ff;
  color: white;
  font-size: 20px;
  cursor: pointer;
  transition: background-color 0.3s, border 0.3s;
}
button:hover {
  border: 2px solid #ccc;
  background-color: #e0e8ff;
}
.thumbnail {
  position: relative;
  aspect-ratio: 16 / 9;
  display: flex;
  overflow: hidden;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  border-radius: 0.5rem;
  outline-offset: 2px;
  width: 8rem;
  vertical-align: middle;
  background-color: #ffffff;
  background-size: cover;
  user-select: none;
}
.thumbnail.blue {
  background-image: conic-gradient(at top right, #c76a15, #087ea4, #2b3491);
}
.video {
  display: flex;
  flex-direction: row;
  gap: 0.75rem;
  align-items: center;
  margin-top: 1em;
}
.video .link {
  display: flex;
  flex-direction: row;
  flex: 1 1 0;
  gap: 0.125rem;
  outline-offset: 4px;
  cursor: pointer;
}
.video .info {
  display: flex;
  flex-direction: column;
  justify-content: center;
  margin-left: 8px;
  gap: 0.125rem;
}
.video .info:hover {
  text-decoration: underline;
}
.video-title {
  font-size: 15px;
  line-height: 1.25;
  font-weight: 700;
  color: #23272f;
}
.video-description {
  color: #5e687e;
  font-size: 13px;
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

除了设置 `default` 之外，你还可以为 `enter`、`exit`、`update` 和 `share` 动画提供配置。

<Sandpack>

```js src/Video.js hidden
function Thumbnail({video, children}) {
  return (
    <div
      aria-hidden="true"
      tabIndex={-1}
      className={`thumbnail ${video.image}`}
    />
  );
}

export function Video({video}) {
  return (
    <div className="video">
      <div className="link">
        <Thumbnail video={video}></Thumbnail>

        <div className="info">
          <div className="video-title">{video.title}</div>
          <div className="video-description">{video.description}</div>
        </div>
      </div>
    </div>
  );
}
```

```js
import {ViewTransition, useState, startTransition} from 'react';
import {Video} from './Video';
import videos from './data';

function Item() {
  return (
    <ViewTransition enter="slide-in" exit="slide-out">
      <Video video={videos[0]} />
    </ViewTransition>
  );
}

export default function Component() {
  const [showItem, setShowItem] = useState(false);
  return (
    <>
      <button
        onClick={() => {
          startTransition(() => {
            setShowItem((prev) => !prev);
          });
        }}>
        {showItem ? '➖' : '➕'}
      </button>

      {showItem ? <Item /> : null}
    </>
  );
}
```

```js src/data.js hidden
export default [
  {
    id: '1',
    title: 'First video',
    description: 'Video description',
    image: 'blue',
  },
];
```

```css
::view-transition-old(.slide-in) {
  animation-name: slideOutRight;
  animation-duration: 500ms;
  animation-timing-function: ease-in-out;
}

::view-transition-new(.slide-in) {
  animation-name: slideInRight;
  animation-duration: 500ms;
  animation-timing-function: ease-in-out;
}

::view-transition-old(.slide-out) {
  animation-name: slideOutLeft;
  animation-duration: 500ms;
  animation-timing-function: ease-in-out;
}

::view-transition-new(.slide-out) {
  animation-name: slideInLeft;
  animation-duration: 500ms;
  animation-timing-function: ease-in-out;
}

@keyframes slideOutLeft {
  from {
    transform: translateX(0);
    opacity: 1;
  }
  to {
    transform: translateX(-100%);
    opacity: 0;
  }
}

@keyframes slideInLeft {
  from {
    transform: translateX(-100%);
    opacity: 0;
  }
  to {
    transform: translateX(0);
    opacity: 1;
  }
}

@keyframes slideOutRight {
  from {
    transform: translateX(0);
    opacity: 1;
  }
  to {
    transform: translateX(100%);
    opacity: 0;
  }
}

@keyframes slideInRight {
  from {
    transform: translateX(100%);
    opacity: 0;
  }
  to {
    transform: translateX(0);
    opacity: 1;
  }
}

@keyframes slideInRight {
  from {
    transform: translateX(100%);
    opacity: 0;
  }
  to {
    transform: translateX(0);
    opacity: 1;
  }
}

#root {
  display: flex;
  flex-direction: column;
  align-items: center;
  min-height: 200px;
}
button {
  border: none;
  border-radius: 50%;
  width: 50px;
  height: 50px;
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #f0f8ff;
  color: white;
  font-size: 20px;
  cursor: pointer;
  transition: background-color 0.3s, border 0.3s;
}
button:hover {
  border: 2px solid #ccc;
  background-color: #e0e8ff;
}
.thumbnail {
  position: relative;
  aspect-ratio: 16 / 9;
  display: flex;
  overflow: hidden;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  border-radius: 0.5rem;
  outline-offset: 2px;
  width: 8rem;
  vertical-align: middle;
  background-color: #ffffff;
  background-size: cover;
  user-select: none;
}
.thumbnail.blue {
  background-image: conic-gradient(at top right, #c76a15, #087ea4, #2b3491);
}
.video {
  display: flex;
  flex-direction: row;
  gap: 0.75rem;
  align-items: center;
  margin-top: 1em;
}
.video .link {
  display: flex;
  flex-direction: row;
  flex: 1 1 0;
  gap: 0.125rem;
  outline-offset: 4px;
  cursor: pointer;
}
.video .info {
  display: flex;
  flex-direction: column;
  justify-content: center;
  margin-left: 8px;
  gap: 0.125rem;
}
.video .info:hover {
  text-decoration: underline;
}
.video-title {
  font-size: 15px;
  line-height: 1.25;
  font-weight: 700;
  color: #23272f;
}
.video-description {
  color: #5e687e;
  font-size: 13px;
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

### 使用 JavaScript 添加动画 {/*animating-with-javascript*/}

虽然 [View Transition Classes](#view-transition-class) 让你可以用 CSS 定义动画，但有时你需要对动画进行命令式控制。`onEnter`、`onExit`、`onUpdate` 和 `onShare` 回调会让你直接访问 view transition 伪元素，这样你就可以使用 [Web Animations API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Animations_API) 为它们添加动画。

每个回调都会接收一个带有 `.old` 和 `.new` 属性的 `instance`，分别表示 view transition 伪元素。你可以像操作 DOM 元素一样对它们调用 `.animate()`：

```js
<ViewTransition
  onEnter={(instance) => {
    const anim = instance.new.animate(
      [
        {transform: 'scale(0.8)'},
        {transform: 'scale(1)'},
      ],
      {duration: 300, easing: 'ease-out'}
    );
    return () => anim.cancel();
  }}>
  <div>...</div>
</ViewTransition>
```

这样你就可以将 CSS 驱动的动画与 JavaScript 驱动的动画结合起来。

在下面的示例中，默认的交叉淡入淡出由 CSS 处理，而滑动动画则由 `onEnter` 和 `onExit` 中的 JavaScript 驱动：

<Sandpack>

```js src/Video.js hidden
function Thumbnail({video, children}) {
  return (
    <div
      aria-hidden="true"
      tabIndex={-1}
      className={`thumbnail ${video.image}`}
    />
  );
}

export function Video({video}) {
  return (
    <div className="video">
      <div className="link">
        <Thumbnail video={video}></Thumbnail>

        <div className="info">
          <div className="video-title">{video.title}</div>
          <div className="video-description">{video.description}</div>
        </div>
      </div>
    </div>
  );
}
```

```js
import {ViewTransition, useState, startTransition} from 'react';
import {Video} from './Video';
import videos from './data';
import {SLIDE_IN, SLIDE_OUT} from './animations';

function Item() {
  return (
    <ViewTransition
      default="none"
      /* CSS 驱动的交叉淡入淡出默认值 */
      enter="auto"
      exit="auto"
      /* JS 驱动的滑动动画 */
      onEnter={(instance) => {
        const anim = instance.new.animate(
          SLIDE_IN,
          {duration: 500, easing: 'ease-out'}
        );
        return () => anim.cancel();
      }}
      onExit={(instance) => {
        const anim = instance.old.animate(
          SLIDE_OUT,
          {duration: 300, easing: 'ease-in'}
        );
        return () => anim.cancel();
      }}>
      <Video video={videos[0]} />
    </ViewTransition>
  );
}

export default function Component() {
  const [showItem, setShowItem] = useState(false);
  return (
    <>
      <button
        onClick={() => {
          startTransition(() => {
            setShowItem((prev) => !prev);
          });
        }}>
        {showItem ? '➖' : '➕'}
      </button>

      {showItem ? <Item /> : null}
    </>
  );
}
```

```js src/animations.js
export const SLIDE_IN = [
  {transform: 'translateY(20px)'},
  {transform: 'translateY(0)'},
];

export const SLIDE_OUT = [
  {transform: 'translateY(0)'},
  {transform: 'translateY(-20px)'},
];
```

```js src/data.js hidden
export default [
  {
    id: '1',
    title: 'First video',
    description: 'Video description',
    image: 'blue',
  },
];
```

```css
#root {
  display: flex;
  flex-direction: column;
  align-items: center;
  min-height: 200px;
}
button {
  border: none;
  border-radius: 50%;
  width: 50px;
  height: 50px;
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #f0f8ff;
  color: white;
  font-size: 20px;
  cursor: pointer;
  transition: background-color 0.3s, border 0.3s;
}
button:hover {
  border: 2px solid #ccc;
  background-color: #e0e8ff;
}
.thumbnail {
  position: relative;
  aspect-ratio: 16 / 9;
  display: flex;
  overflow: hidden;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  border-radius: 0.5rem;
  outline-offset: 2px;
  width: 8rem;
  vertical-align: middle;
  background-color: #ffffff;
  background-size: cover;
  user-select: none;
}
.thumbnail.blue {
  background-image: conic-gradient(at top right, #c76a15, #087ea4, #2b3491);
}
.video {
  display: flex;
  flex-direction: row;
  gap: 0.75rem;
  align-items: center;
  margin-top: 1em;
}
.video .link {
  display: flex;
  flex-direction: row;
  flex: 1 1 0;
  gap: 0.125rem;
  outline-offset: 4px;
  cursor: pointer;
}
.video .info {
  display: flex;
  flex-direction: column;
  justify-content: center;
  margin-left: 8px;
  gap: 0.125rem;
}
.video .info:hover {
  text-decoration: underline;
}
.video-title {
  font-size: 15px;
  line-height: 1.25;
  font-weight: 700;
  color: #23272f;
}
.video-description {
  color: #5e687e;
  font-size: 13px;
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

<Note>

#### 始终清理 View Transition 事件 {/*always-clean-up-view-transition-events*/}

View Transition 事件应始终返回一个清理函数：

```js {7}
<ViewTransition
  onEnter={(instance) => {
    const anim = instance.new.animate(
      SLIDE_IN,
      {duration: 500, easing: 'ease-out'}
    );
    return () => anim.cancel();
  }}
>
```

这使浏览器能够在 View Transition 被中断时取消动画。

</Note>

---

### 使用 JavaScript 为过渡类型添加动画 {/*animating-transition-types-with-javascript*/}

你可以在传递给 `ViewTransition` 事件的 `types` 中，根据过渡是如何触发的来有条件地应用不同的动画。

```js {3}
 <ViewTransition
  onEnter={(instance, types) => {
    const duration = types.includes('fast') ? 150 : 2000;
    const anim = instance.new.animate(
      SLIDE_IN,
      {duration: duration, easing: 'ease-out'}
    );
    return () => anim.cancel();
  }}
>
```

这个示例调用 [`addTransitionType`](/reference/react/addTransitionType) 将一个过渡标记为 "fast"，然后调整动画时长：

<Sandpack>

```js src/Video.js hidden
function Thumbnail({video, children}) {
  return (
    <div
      aria-hidden="true"
      tabIndex={-1}
      className={`thumbnail ${video.image}`}
    />
  );
}

export function Video({video}) {
  return (
    <div className="video">
      <div className="link">
        <Thumbnail video={video}></Thumbnail>

        <div className="info">
          <div className="video-title">{video.title}</div>
          <div className="video-description">{video.description}</div>
        </div>
      </div>
    </div>
  );
}
```

```js
import {ViewTransition, useState, startTransition, addTransitionType} from 'react';
import {Video} from './Video';
import videos from './data';
import {SLIDE_IN, SLIDE_OUT} from './animations';

function Item() {
  return (
    <ViewTransition
      onEnter={(instance, types) => {
        const duration = types.includes('fast') ? 150 : 2000;
        const anim = instance.new.animate(
          SLIDE_IN,
          {duration: duration, easing: 'ease-out'}
        );
        return () => anim.cancel();
      }}
      onExit={(instance, types) => {
        const duration = types.includes('fast') ? 150 : 500;
        const anim = instance.old.animate(
          SLIDE_OUT,
          {duration: duration, easing: 'ease-in'}
        );
        return () => anim.cancel();
      }}>
      <Video video={videos[0]} />
    </ViewTransition>
  );
}

export default function Component() {
  const [showItem, setShowItem] = useState(false);
  const [isFast, setIsFast] = useState(false);
  return (
    <>
      <div>
        快速： <input type="checkbox" onChange={() => {setIsFast(f => !f)}} value={isFast}></input>
      </div><br />
      <button
        onClick={() => {
          startTransition(() => {
            if (isFast) {
              addTransitionType('fast');
            }
            setShowItem((prev) => !prev);
          });
        }}>
        {showItem ? '➖' : '➕'}
      </button>

      {showItem ? <Item /> : null}
    </>
  );
}
```

```js src/animations.js
export const SLIDE_IN = [
  {opacity: 0, transform: 'translateY(20px)'},
  {opacity: 1, transform: 'translateY(0)'},
];

export const SLIDE_OUT = [
  {opacity: 1, transform: 'translateY(0)'},
  {opacity: 0, transform: 'translateY(-20px)'},
];
```

```js src/data.js hidden
export default [
  {
    id: '1',
    title: '第一个视频',
    description: '视频描述',
    image: 'blue',
  },
];
```

```css
#root {
  display: flex;
  flex-direction: column;
  align-items: center;
  min-height: 200px;
}
button {
  border: none;
  border-radius: 50%;
  width: 50px;
  height: 50px;
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #f0f8ff;
  color: white;
  font-size: 20px;
  cursor: pointer;
  transition: background-color 0.3s, border 0.3s;
}
button:hover {
  border: 2px solid #ccc;
  background-color: #e0e8ff;
}
.thumbnail {
  position: relative;
  aspect-ratio: 16 / 9;
  display: flex;
  overflow: hidden;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  border-radius: 0.5rem;
  outline-offset: 2px;
  width: 8rem;
  vertical-align: middle;
  background-color: #ffffff;
  background-size: cover;
  user-select: none;
}
.thumbnail.blue {
  background-image: conic-gradient(at top right, #c76a15, #087ea4, #2b3491);
}
.video {
  display: flex;
  flex-direction: row;
  gap: 0.75rem;
  align-items: center;
  margin-top: 1em;
}
.video .link {
  display: flex;
  flex-direction: row;
  flex: 1 1 0;
  gap: 0.125rem;
  outline-offset: 4px;
  cursor: pointer;
}
.video .info {
  display: flex;
  flex-direction: column;
  justify-content: center;
  margin-left: 8px;
  gap: 0.125rem;
}
.video .info:hover {
  text-decoration: underline;
}
.video-title {
  font-size: 15px;
  line-height: 1.25;
  font-weight: 700;
  color: #23272f;
}
.video-description {
  color: #5e687e;
  font-size: 13px;
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

### 构建支持 View Transition 的路由器 {/*building-view-transition-enabled-routers*/}

React 会等待任何挂起的导航完成，以确保滚动恢复发生在动画内部。如果导航被 React 阻塞，你的路由器必须在 `useLayoutEffect` 中解除阻塞，因为 `useEffect` 会导致死锁。

如果 `startTransition` 是从旧版 popstate 事件中启动的，例如在“后退”导航期间，那么它必须同步完成，以确保滚动和表单恢复正常工作。这与运行 View Transition 动画相冲突。因此，React 会跳过来自 popstate 的动画，后退按钮不会运行动画。你可以通过升级路由器以使用 Navigation API 来修复这个问题。

---

## 故障排除 {/*troubleshooting*/}

### 我的 `<ViewTransition>` 没有激活 {/*my-viewtransition-is-not-activating*/}

`<ViewTransition>` 只有在它放在任何 DOM 节点之前时才会激活：

```js [3, 5]
function Component() {
  return (
    <div>
      <ViewTransition>Hi</ViewTransition>
    </div>
  );
}
```

要修复，请确保 `<ViewTransition>` 出现在任何其他 DOM 节点之前：

```js [3, 5]
function Component() {
  return (
    <ViewTransition>
      <div>Hi</div>
    </ViewTransition>
  );
}
```

### 我遇到了错误“有两个 `<ViewTransition name=%s>` 组件同时以相同的名称挂载。” {/*two-viewtransition-with-same-name*/}

当两个具有相同 `name` 的 `<ViewTransition>` 组件同时挂载时，就会发生此错误：

```js [3]
function Item() {
  // 🚩 所有项目都会获得相同的 "name"。
  return <ViewTransition name="item">...</ViewTransition>;
}

function ItemList({items}) {
  return (
    <>
      {items.map((item) => (
        <Item key={item.id} />
      ))}
    </>
  );
}
```

这会导致 View Transition 报错。在开发环境中，React 会检测到这个问题并将其提示出来，同时记录两条错误：

<ConsoleBlockMulti>
<ConsoleLogLine level="error">

有两个 `<ViewTransition name=%s>` 组件同时以相同的名称挂载。这不受支持，并且会导致 View Transitions 出错。请尝试使用更唯一的名称，例如使用命名空间前缀，并将某个项目的 id 添加到名称中。
{' '}at Item
{' '}at ItemList

</ConsoleLogLine>

<ConsoleLogLine level="error">

现有重复的 `<ViewTransition name=%s>` 有以下堆栈跟踪。
{' '}at Item
{' '}at ItemList

</ConsoleLogLine>
</ConsoleBlockMulti>

要修复此问题，请确保整个应用中同一时间只挂载一个同名的 `<ViewTransition>`，方法是确保 `name` 是唯一的，或者在名称中添加一个 `id`：

```js [3]
function Item({id}) {
  // ✅ 所有项目都会获得唯一的名称。
  return <ViewTransition name={`item-${id}`}>...</ViewTransition>;
}

function ItemList({items}) {
  return (
    <>
      {items.map((item) => (
        <Item key={item.id} item={item} />
      ))}
    </>
  );
}
```
