---
title: "React 19.3"
author: The React Team
date: 2026/09/09
description: React 19.3 带来了 View Transitions、Fragment Refs、browser()、Trusted Types 等新功能。
---

2026 年 9 月 9 日，由 [React 团队](/community/team)发布

---

<Intro>

React 19.3 现已在 npm 上发布！

</Intro>


[去年](/blog/2025/04/23/react-labs-view-transitions-activity-and-more)，我们介绍了即将加入 React 的两个实验性 API：View Transitions 和 Fragment Refs。现在很高兴宣布，它们都已在 React 19.3 中成为稳定 API！

本文将介绍它们的工作方式，以及本次版本中的其他重要变化。

<InlineToc />

---

## React 新功能 {/*new-react-features*/}

### View Transitions {/*view-transition*/}

新的 `<ViewTransition>` 组件使用浏览器的 [View Transition API](https://developer.mozilla.org/en-US/docs/Web/API/View_Transition_API)，为元素的进入、退出、移动或尺寸变化添加动画。我们[去年](/blog/2025/04/23/react-labs-view-transitions-activity-and-more#view-transitions)将它作为实验性 API 介绍，而在 19.3 中，它已稳定并可投入使用。

要为部分 UI 添加动画，请用 `<ViewTransition>` 包裹它：

```js
import { ViewTransition } from 'react';

{isShowing && (
  <ViewTransition>
    <Component />
  </ViewTransition>
)}
```

现在，每当标记为 [Transition](/reference/react/useTransition) 的更新改变了子组件的样式，或导致 `ViewTransition` 挂载或卸载时，React 都会为该更新添加动画。

{/*
Updates outside of a Transition don't trigger animations, as those are meant to be urgent and reflected immediately in the UI. State updates inside of [startTransition](/reference/react/startTransition), a [`<Suspense>`](/reference/react/Suspense) reveal, or an update from [`useDeferredValue`](/reference/react/useDeferredValue) all cause a View Transition to animate.
*/}

React 会根据组件树的变化方式选择要运行的动画：

- **enter**：添加了 `<ViewTransition>`。
- **exit**：移除了 `<ViewTransition>`。
- **update**：`<ViewTransition>` 的子元素样式或内容发生变化。
- **share**：在一个位置移除带名称的 `<ViewTransition>`，并在另一个位置添加同名组件。

请注意，未标记为 Transition 的更新不会触发动画，因为这类更新属于紧急更新，应立即反映在 UI 中。在 [startTransition](/reference/react/startTransition) 中执行的状态更新、[`<Suspense>`](/reference/react/Suspense) 内容显示，或 [`useDeferredValue`](/reference/react/useDeferredValue) 引起的更新，都会触发 View Transition 动画。

下面是一个简单的进入/退出动画示例：

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
import { ViewTransition, useState, startTransition } from 'react';
import { Video } from './Video';
import videos from './data';

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

      {showItem && (
        <ViewTransition>
          <Video video={videos[0]} />
        </ViewTransition>
      )}
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
    "react": "19.3.0-canary-f1f7ed2a-20260904",
    "react-dom": "19.3.0-canary-f1f7ed2a-20260904",
    "react-scripts": "latest"
  }
}
```

</Sandpack>

默认情况下，`<ViewTransition>` 会使用平滑的交叉淡化动画。你可以传入 [View Transition 类](/reference/react/ViewTransition#view-transition-class)并在 CSS 中定义动画，以自定义不同类型的动画；也可以使用 [Web Animations API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Animations_API)，通过[事件属性](/reference/react/ViewTransition#view-transition-event)（`onEnter`、`onExit`、`onShare`、`onUpdate`）命令式地触发动画。

{/*
需要注意几条规则：只有当 `<ViewTransition>` 是其子树中最先渲染的内容、位于任何 DOM 节点之前时，它才会在 enter 和 exit 时播放动画；并且在同一时刻，每个共享的 `name` 在整个应用中都必须唯一。详情请参阅[故障排查](/reference/react/ViewTransition#troubleshooting)。
 */}

目前，`<ViewTransition>` 仅适用于 DOM。我们正在开发对 React Native 和其他平台的支持。

更多信息请参阅 [`<ViewTransition>` 文档](/reference/react/ViewTransition)。

---

#### `addTransitionType` {/*add-transition-type*/}

有时，同一种状态更新需要使用不同的动画。例如，轮播图_向前_导航到第三张幻灯片时，应让幻灯片从右向左移动；_向后_导航时，则应从左向右移动，尽管这两种操作都将 `currentSlide` 设为 3。

你可以在更新状态的同时调用 `addTransitionType`，自定义某个 View Transition 的动画。这样就能补充特定 Transition 的_原因_信息：

```js {3,10}
function nextSlide() {
  startTransition(() => {
    addTransitionType('next');
    setCurrentSlide(c => c + 1);
  });
}

function previousSlide() {
  startTransition(() => {
    addTransitionType('previous');
    setCurrentSlide(c => c - 1);
  });
}
```

然后，你可以根据该 Transition 类型指定不同的动画：

```js
<ViewTransition
  enter={{
    'next': 'from-right',
    'previous': 'from-left',
  }}
  exit={{
    'next': 'to-left',
    'previous': 'to-right',
  }}
>
  <Page />
</ViewTransition>
```

下面是一个示例：

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
import {
  ViewTransition,
  addTransitionType,
  useState,
  startTransition,
  Fragment
} from 'react';
import { Video } from './Video';
import videos from './data';
import './animations.css';

export default function Component() {
  const [selected, setSelected] = useState(0)
  const video = videos[selected];

  return (
    <>
      <div className="button-container">
        <button
          onClick={() => {
            startTransition(() => {
              addTransitionType('previous');
              setSelected(c => c > 0 ? c - 1 : videos.length - 1 )
            });
          }}>
          ⬅️
        </button>
        <button
          onClick={() => {
            startTransition(() => {
              addTransitionType('next');
              setSelected(c => c + 1 < videos.length ? c + 1 : 0)
            });
          }}>
          ➡️
        </button>
      </div>

      <ViewTransition
        key={video.id}
        enter={{
          'next': 'from-right',
          'previous': 'from-left'
        }}
        exit={{
          'next': 'to-left',
          'previous': 'to-right'
        }}
      >
        <Video video={video} />
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
  {
    id: '5',
    title: 'Fifth video',
    description: 'Video description',
    image: 'yellow',
  },
  {
    id: '6',
    title: 'Sixth video',
    description: 'Video description',
    image: 'gray',
  },
];
```

```css src/animations.css
::view-transition-old(*),
::view-transition-new(*) {
  animation-duration: 250ms;
  animation-timing-function: cubic-bezier(0.22, 1, 0.36, 1);
}

::view-transition-new(.from-right) {
  --offset: 100%;
  animation-name: slide-in;
}

::view-transition-new(.from-left) {
  --offset: -100%;
  animation-name: slide-in;
}

::view-transition-old(.to-right) {
  --offset: 100%;
  animation-name: slide-out;
}

::view-transition-old(.to-left) {
  --offset: -100%;
  animation-name: slide-out;
}

@keyframes slide-in {
  from {
    transform: translateX(var(--offset));
    opacity: 0;
  }
}

@keyframes slide-out {
  to {
    transform: translateX(var(--offset));
    opacity: 0;
  }
}
```

```css src/styles.css
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
.button-container {
  display: flex;
  gap: 8px;
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

.thumbnail.yellow {
  background-image: conic-gradient(at top right, #c76a15, #FABD62, #2b3491);
}

.thumbnail.gray {
  background-image: conic-gradient(at top right, #c76a15, #4E5769, #2b3491);
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
    "react": "19.3.0-canary-f1f7ed2a-20260904",
    "react-dom": "19.3.0-canary-f1f7ed2a-20260904",
    "react-scripts": "latest"
  }
}
```

</Sandpack>

React 也会将每个 Transition 类型添加为元素的浏览器 [view transition type](https://www.w3.org/TR/css-view-transitions-2/#active-view-transition-pseudo-examples)，这样你就可以在 CSS 中使用 `:active-view-transition-type(...)` 限定动画的作用范围。

更多信息请参阅 [`addTransitionType` 文档](/reference/react/addTransitionType)。

---

#### 使用 Suspense 为回退内容、图片和字体添加动画 {/*animating-fallbacks-images-and-fonts-with-suspense*/}

React 中 View Transitions 最令人期待的特性之一，是它与 Suspense 的集成。

将 Suspense 边界包裹在 `<ViewTransition>` 中，就可以在它显示子元素时添加动画：

```js
<ViewTransition>
  <Suspense fallback={<Loading />}>
    <Component />
  </Suspense>
</ViewTransition>
```

子元素加载完成后，React 会触发从回退内容到最终内容的 **update** 动画。

下面是一个示例。试着点击 ➕ 来渲染 `LazyVideo`；它首次渲染时会暂停：

<Sandpack>

```js src/Video.js hidden
import { ViewTransition } from 'react';

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
import { Suspense, useState, startTransition, use, ViewTransition } from 'react';
import { Video, VideoPlaceholder } from './Video';
import { fetchVideo } from './data';

export default function Component() {
  const [showItem, setShowItem] = useState(false);

  return (
    <>
      <button
        onClick={() => {
          startTransition(() => {
            setShowItem((prev) => !prev);
          });
        }}
      >
        {showItem ? '➖' : '➕'}
      </button>

      {showItem && (
        <ViewTransition>
          <Suspense fallback={<VideoPlaceholder />}>
            <LazyVideo />
          </Suspense>
        </ViewTransition>
      )}
    </>
  );
}

function LazyVideo() {
  const video = use(fetchVideo());

  return <Video video={video} />;
}
```


```js src/data.js hidden
let cache = null;

export function fetchVideo() {
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
```

```css
::view-transition-old(*),
::view-transition-new(*) {
  /* animation-duration: 750ms; */
  /* animation-timing-function: cubic-bezier(0.22, 1, 0.36, 1); */
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
.loading {
  background-image: linear-gradient(
    90deg,
    rgba(173, 216, 230, 0.3) 25%,
    rgba(135, 206, 250, 0.5) 50%,
    rgba(173, 216, 230, 0.3) 75%
  );
  background-size: 200% 100%;
  animation: shimmer 1.5s infinite;
  z-index: 999;
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
}
.video .info {
  display: flex;
  flex-direction: column;
  justify-content: center;
  margin-left: 8px;
  gap: 0.125rem;
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
    "react": "19.3.0-canary-f1f7ed2a-20260904",
    "react-dom": "19.3.0-canary-f1f7ed2a-20260904",
    "react-scripts": "latest"
  }
}
```

</Sandpack>

虽然这样可以正常运行，但你会发现，即使视频已经加载完成，后续每次显示和隐藏时它仍会播放动画。（你可能还会注意到，回退内容首次显示时也会淡入。）

通常，Suspense 动画应谨慎使用；对于原本可以立即显示的缓存 UI，应避免使用动画。

使用 Suspense 动画时，可以遵循以下原则来获得良好的用户体验：

- 回退内容应立即出现，_不使用动画_。
- 回退内容应通过_动画_过渡到最终内容。
- 不会暂停的子元素应立即出现，_不使用动画_。

这样，内容已经加载完成时，应用仍会感觉灵敏；动画只用于让回退内容过渡到最终内容的过程更流畅。

要修复上面的示例，我们可以只启用 update 动画，并禁用其他动画：

```js {1}
<ViewTransition update="auto" default="none">
  <Suspense fallback={<Fallback />}>
    <Component />
  </Suspense>
</ViewTransition>
```

现在来看看它的效果：

<Sandpack>

```js src/Video.js hidden
import { ViewTransition } from 'react';

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
import { Suspense, useState, startTransition, use, ViewTransition } from 'react';
import { Video, VideoPlaceholder } from './Video';
import { fetchVideo } from './data';

export default function Component() {
  const [showItem, setShowItem] = useState(false);

  return (
    <>
      <button
        onClick={() => {
          startTransition(() => {
            setShowItem((prev) => !prev);
          });
        }}
      >
        {showItem ? '➖' : '➕'}
      </button>

      {showItem && (
        <ViewTransition update="auto" default="none">
          <Suspense fallback={<VideoPlaceholder />}>
            <LazyVideo />
          </Suspense>
        </ViewTransition>
      )}
    </>
  );
}

function LazyVideo() {
  const video = use(fetchVideo());

  return <Video video={video} />;
}
```


```js src/data.js hidden
let cache = null;

export function fetchVideo() {
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
```

```css
::view-transition-old(*),
::view-transition-new(*) {
  /* animation-duration: 5000ms; */
  /* animation-timing-function: cubic-bezier(0.22, 1, 0.36, 1); */
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
.loading {
  background-image: linear-gradient(
    90deg,
    rgba(173, 216, 230, 0.3) 25%,
    rgba(135, 206, 250, 0.5) 50%,
    rgba(173, 216, 230, 0.3) 75%
  );
  background-size: 200% 100%;
  animation: shimmer 1.5s infinite;
  z-index: 999;
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
}
.video .info {
  display: flex;
  flex-direction: column;
  justify-content: center;
  margin-left: 8px;
  gap: 0.125rem;
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
    "react": "19.3.0-canary-f1f7ed2a-20260904",
    "react-dom": "19.3.0-canary-f1f7ed2a-20260904",
    "react-scripts": "latest"
  }
}
```

</Sandpack>

注意，点击按钮后回退内容会立即出现，因此 UI 能及时响应用户操作。此外，视频加载完成后，再切换显示状态也是即时的。

根据你想要实现的效果，还可以采用其他模式。更多信息请参阅[使用 Suspense 添加动画](/reference/react/ViewTransition#animating-from-suspense-content)文档。

---

除了为回退内容添加动画，View Transitions 还可以让图片或字体在加载期间触发 Suspense。

这样可以避免图片或字体在加载完成时突然出现的浏览器默认行为，并让你构建协调一致的加载流程，统一考虑组件所需的所有资源。

将图片或字体包裹在 `<ViewTransition>` 中，即可在它们加载期间触发 Suspense：

```js
<ViewTransition>
  <Suspense fallback={<Fallback />}>
    <img src={imageSrc} />

    <style href={fontSrc} precedence="default">
      {`@font-face {
        font-family: 'Fancy';
        src: url(${fontSrc}) format('truetype');
        font-display: swap;
      }`}
    </style>
  </Suspense>
</ViewTransition>
```

下面是一个组件示例：只有在数据、图片和字体都加载完成后，它才会结束暂停：


<Sandpack>

```js
import { ViewTransition, Suspense, use, useState, startTransition } from 'react';
import { fetchQuote } from './data.js';
import { freshStylesheetUrl, freshImageUrl } from './resources.js';
import { ProfileCard, ProfileCardLoading } from './ProfileCard.js';
import { VanillaProfileCard } from './VanillaProfileCard.js';

export default function App() {
  const [resources, setResources] = useState(null);
  return (
    <>
      <button
        onClick={() => {
          startTransition(() => {
            setResources({
              quotePromise: fetchQuote(),
              stylesheet: freshStylesheetUrl(),
              image: freshImageUrl(),
            });
          });
        }}>
        Show profile
      </button>

      {resources && (
        <ViewTransition update='auto' default='none'>
          <Suspense fallback={<ProfileCardLoading />}>
            <ProfileCard resources={resources} />
          </Suspense>
        </ViewTransition>
      )}

      <hr />

      <VanillaProfileCard />
    </>
  );
}
```

```js src/ProfileCard.js
import { use } from 'react';

export function ProfileCard({ resources }) {
  const quote = use(resources.quotePromise);
  return (
    <>
      <link rel="stylesheet" href={resources.stylesheet} precedence="default" />
      <div className="profile-card">
        <img src={resources.image} alt="Jack Pope" width={80} height={80} />
        <div>
          <p className="name">Jack Pope</p>
          <p className="bio">{quote}</p>
        </div>
      </div>
    </>
  );
}

export function ProfileCardLoading() {
  return (
    <div className="profile-card">
      <div className="avatar-placeholder" />
      <div>
        <p className="name name-placeholder">&nbsp;</p>
        <p className="bio bio-placeholder">&nbsp;</p>
      </div>
    </div>
  );
}
```


```js src/VanillaProfileCard.js
import { useRef } from 'react';
import { fetchQuote } from './data.js';
import { freshStylesheetUrl, freshImageUrl } from './resources.js';

export function VanillaProfileCard() {
  const ref = useRef(null);
  async function show() {
    const quote = await fetchQuote();
    const doc = ref.current.contentWindow.document;
    doc.open();
    doc.write(`
      <style>
        body { margin: 0; font-family: sans-serif; }
        img { object-fit: cover; }
        .profile-card { display: flex; gap: 12px; align-items: center; }
        .profile-card img { border-radius: 50%; background: #dfe3e9; }
        .name { margin: 0 0 4px; font-family: 'Caveat', sans-serif; font-size: 22px; line-height: 28px; font-weight: bold; }
        .bio { margin: 0; font-family: 'Caveat', sans-serif; font-size: 20px; line-height: 26px; }
      </style>
      <div class="profile-card">
        <img src="${freshImageUrl()}" alt="Jack Pope" width="80" height="80" />
        <div>
          <p class="name">Jack Pope</p>
          <p class="bio">${quote}</p>
        </div>
      </div>
      <link rel="stylesheet" href="${freshStylesheetUrl()}">
    `);
    doc.close();
  }
  return (
    <>
      <button onClick={show}>Show profile (without React)</button>
      <iframe ref={ref} title="Vanilla profile card" className="vanilla-frame" />
    </>
  );
}
```

```js src/resources.js hidden
// Add a unique parameter so the resources aren't cached,
// and every run shows the loading state.
export function freshStylesheetUrl() {
  return (
    'https://fonts.googleapis.com/css2?family=Caveat&display=swap' +
    '&t=' +
    Date.now()
  );
}

export function freshImageUrl() {
  return 'https://react.dev/images/team/jack-pope.jpg?t=' + Date.now();
}
```

```js src/data.js hidden
// Note: the way you would do data fetching depends on
// the framework that you use together with Suspense.

export async function fetchQuote() {
  // Add a fake delay to make waiting noticeable.
  await new Promise((resolve) => {
    setTimeout(resolve, 250);
  });
  return 'The best way to predict the future is to invent it.';
}
```

```css
#root {
  min-height: 320px;
}
button {
  margin-right: 8px;
}
hr {
  margin: 16px 0;
}
img {
  object-fit: cover;
}
.profile-card {
  display: flex;
  gap: 12px;
  align-items: center;
  margin-top: 1em;
}
.profile-card img {
  border-radius: 50%;
  background: #dfe3e9;
}
.name {
  margin: 0 0 4px;
  font-family: 'Caveat', sans-serif;
  font-size: 22px;
  line-height: 28px;
  font-weight: bold;
}
.bio {
  margin: 0;
  font-family: 'Caveat', sans-serif;
  font-size: 20px;
  line-height: 26px;
}
.profile-card img {
  display: block;
}
.avatar-placeholder {
  width: 80px;
  height: 80px;
  border-radius: 50%;
  background: #dfe3e9;
}
.name-placeholder,
.bio-placeholder {
  border-radius: 4px;
  background: #dfe3e9;
  color: transparent;
}
.name-placeholder {
  width: 90px;
}
.bio-placeholder {
  width: 220px;
}
.vanilla-frame {
  display: block;
  margin-top: 1em;
  border: none;
  width: 100%;
  height: 110px;
}
```

```json package.json hidden
{
  "dependencies": {
    "react": "19.3.0-canary-f1f7ed2a-20260904",
    "react-dom": "19.3.0-canary-f1f7ed2a-20260904",
    "react-scripts": "latest"
  }
}
```

</Sandpack>

更多关于等待图片、字体或样式表加载的信息，请参阅 [Suspense 文档](/reference/react/Suspense#waiting-for-a-font-to-load)。

---

### Fragment Refs {/*fragment-refs*/}

当你需要更精细地控制组件的 DOM 节点时（例如添加事件监听器、观察可见性或移动焦点），通常可以使用 ref。但在某些情况下，这会比较困难：

- 渲染一组没有共同父元素的兄弟节点的组件。
- 不会将 `ref` prop 传递给其他元素的组件。

```js
function Component() {
  // How can we work with the list of DOM nodes rendered by this component?
  return (
    {posts.map(post => (
      <Heading key={post.id}>
        {post.title}
      </Heading>
    ))}
  )
}
```

有时可以额外添加一个包装用的 `<div>` 来承载 ref，但这可能会干扰组件的样式或布局。此外，如果组件没有公开 `ref` prop，你就需要修改该组件来添加它；如果组件来自你无法控制的库，这可能做不到。

Fragment Refs 通过提供一组常用的 DOM 方法解决了这些问题。无论 React 组件渲染什么内容，这些方法都可以使用。

在 19.3 中，你可以直接向 [`<Fragment>`](/reference/react/Fragment) 传入 ref 来使用它们。该 ref 会提供一个 `FragmentInstance`，可用于操作 Fragment 的 DOM 子元素：

```js {2,5-6,10}
function Component() {
  const fragmentRef = useRef(null);

  useEffect(() => {
    const fragmentInstance = fragmentRef.current;
    fragmentInstance.focus();
  }, []);

  return (
    <Fragment ref={fragmentRef}>
      {posts.map(post => (
        <Heading key={post.id}>
          {post.title}
        </Heading>
      ))}
    </Fragment>
  )
}
```

`FragmentInstance` 会将子元素的 DOM _作为一个整体_来操作，而不会改变其结构：

- `addEventListener`、`removeEventListener` 和 `dispatchEvent` 管理第一层子元素的事件。
- `focus`、`focusLast` 和 `blur` 会按深度优先顺序在嵌套子元素间移动焦点。
- `observeUsing` 和 `unobserveUsing` 用于连接 `IntersectionObserver` 或 `ResizeObserver`。
- `getClientRects`、`getRootNode`、`compareDocumentPosition` 和 `scrollIntoView` 让你可以测量 Fragment 的第一层子元素并滚动到这些元素。

因此，Fragment Refs 让你可以为其他组件附加行为，而无需修改组件内部实现，也无需改变它们原本生成的 DOM 结构。

此示例展示了一个 `InView` 组件：每当其子元素进入或离开视口时，都会触发 `onChange` prop：

<Sandpack>

```js src/App.js active
import { useState } from 'react';
import Card from './Card';
import InView from './InView';

export default function App() {
  const [isVisible, setIsVisible] = useState(true);

  return (
    <div className={isVisible ? 'page visible' : 'page'}>
      <div className="filler">Scroll down</div>

      <InView onChange={setIsVisible}>
        <Card title="First section" />
        <Card title="Second section" />
      </InView>

      <div className="filler">Scroll up</div>
    </div>
  );
}
```

```js src/Card.js
export default function Card({ title }) {
  return <div className="card">{title}</div>;
}
```

```js src/InView.js
import {
  Fragment,
  useRef,
  useLayoutEffect,
} from 'react';

export default function InView({ onChange, children }) {
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
        onChange(visibleElements.size > 0);
      }
    );
    const fragmentInstance = fragmentRef.current;
    fragmentInstance.observeUsing(observer);
    return () => {
      fragmentInstance.unobserveUsing(observer);
    };
  }, [onChange]);

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


```json package.json hidden
{
  "dependencies": {
    "react": "19.3.0-canary-f1f7ed2a-20260904",
    "react-dom": "19.3.0-canary-f1f7ed2a-20260904",
    "react-scripts": "latest"
  }
}
```

</Sandpack>

注意，尽管没有共同的父 DOM 元素，且 `Card` 没有公开 `ref` prop，`InView` 仍能为其子元素附加行为。

更多关于 Fragment Refs 的信息，请参阅 [`<Fragment>` 文档](/reference/react/Fragment)。

---

## React DOM 新功能 {/*new-react-dom-features*/}

### `browser` {/*browser*/}

如果应用使用服务端渲染，组件会在两个不同的环境中渲染：

- 在服务端，组件会渲染并生成初始 HTML。
- 在客户端，组件会再次渲染，为 HTML 添加事件处理函数。

大多数情况下，组件应该能够生成与客户端初始渲染结果一致的 HTML，从而确保 hydration 正常进行，同时让用户在首次加载时尽可能多地看到页面内容。

但在少数情况下，组件可能无法在服务端生成有意义的 UI。例如，它可能依赖 `localStorage` 之类仅在浏览器中可用的 API，或需要读取浏览器的本地时区。这时，你可能希望该组件完全跳过服务端渲染。

以前，你可能会通过在 Effect 中更新 state 来实现这一点，或者检查 `window` 之类的浏览器 API 是否存在：

```js
function Component() {
  const [mounted, setMounted] = useState(false);

  useEffect(() => {
    setMounted(true)
  }, [])

  // ...
}

function Component() {
  const isBrowser = typeof window !== 'undefined';

  // ...
}
```

在 19.3 中，React 为这种做法提供了专门的 API。

组件可以调用 `use(browser())` 来跳过服务端渲染：

```js {5}
import { use } from 'react';
import { browser } from 'react-dom';

function Component() {
  use(browser());

  // ...
}
```

这会在服务端触发 Suspense，_但不会_在客户端触发。服务端渲染期间，HTML 中会显示最近的 Suspense 边界的回退内容。组件在客户端完成 hydration 后，`use(browser())` 不会暂停，因此组件会继续正常渲染。

下面是一个渲染设备本地时区的组件示例。点击**重新加载**，即可先看到初始 HTML，再看到 React 在客户端的首次渲染：

<Sandpack>

```js
import { Suspense, use } from 'react';
import { browser } from 'react-dom';

function TimeZone() {
  use(browser());
  const timeZone = new Intl.DateTimeFormat().resolvedOptions().timeZone;

  return <p>{timeZone}</p>
}

export default function App() {
  return (
    <>
      <p>Your current time zone is:</p>
      <Suspense fallback="Loading...">
        <TimeZone />
      </Suspense>
    </>
  );
}
```


```js src/Document.js hidden
import App from './App.js';

export default function Document() {
  return (
    <html lang="en">
      <head>
        <title>Event details</title>
        <style>{`
          h1 { font-size: 24px; margin-top: 0; }
        `}</style>
      </head>
      <body>
        <App />
      </body>
    </html>
  );
}
```

```js src/index.js hidden
import { hydrateRoot } from 'react-dom/client';
import { renderToReadableStream } from 'react-dom/server';
import Document from './Document.js';
import { flushReadableStreamToFrame } from './demo-helpers.js';
import './styles.css';

async function main(frame) {
  const stream = await renderToReadableStream(<Document />);
  await flushReadableStreamToFrame(stream, frame);

  // Wait so both the fallback and hydrated content are visible.
  await new Promise(resolve => setTimeout(resolve, 1200));
  hydrateRoot(frame.contentDocument, <Document />);
}

main(document.getElementById('preview'));
```

```js src/demo-helpers.js hidden
export async function flushReadableStreamToFrame(readable, frame) {
  const doc = frame.contentWindow.document;
  const decoder = new TextDecoder();
  const reader = readable.getReader();

  while (true) {
    const {done, value} = await reader.read();
    if (done) {
      break;
    }
    doc.write(decoder.decode(value, {stream: true}));
  }

  doc.write(decoder.decode());
  doc.close();
}
```

```html public/index.html hidden
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Conditional browser rendering</title>
</head>
<body>
  <iframe id="preview" title="Rendered page"></iframe>
</body>
</html>
```

```css src/styles.css hidden
iframe {
  width: 100%;
  height: 240px;
  border: 0;
}
```

```json package.json hidden
{
  "dependencies": {
    "react": "19.3.0-canary-f1f7ed2a-20260904",
    "react-dom": "19.3.0-canary-f1f7ed2a-20260904",
    "react-scripts": "latest"
  }
}
```

</Sandpack>

由于 `TimeZone` 在服务端会暂停，初始 HTML 中会包含 Suspense 回退内容。经过短暂的人为延迟后，React 会对页面进行 hydration，随后组件便可在浏览器中正常渲染。

因此，对于无法在服务端渲染出有意义 UI 的组件，你可以使用 `browser` 和 Suspense 来处理加载状态，并使它们与其他暂停等待内容就绪的组件协同工作。

---

与其他 `use` 调用一样，`use(browser())` 可以放在条件语句中，或提早返回之后调用。这样，你就可以编写组件或自定义 Hook，根据某个条件（例如 prop 的值）选择跳过服务端渲染。

下面是上一个示例的变体：`TimeZone` 组件现在接受一个可选的默认值，并可将它作为初始 HTML 的一部分进行渲染：

<Sandpack>

```js
import { Suspense, use } from 'react';
import { browser } from 'react-dom';

function TimeZone({ defaultValue }) {
  if (defaultValue) {
    return <p>{defaultValue}</p>;
  }

  use(browser());
  const localTimeZone = new Intl.DateTimeFormat().resolvedOptions().timeZone;

  return <p>{localTimeZone}</p>
}

export default function App() {
  return (
    <>
      <div>
        <p>The event's time zone is:</p>
        <TimeZone defaultValue='America/New_York' />
      </div>

      <hr />

      <div>
        <p>Your current time zone is:</p>
        <Suspense fallback="Loading...">
          <TimeZone />
        </Suspense>
      </div>
    </>
  );
}
```


```js src/Document.js hidden
import App from './App.js';

export default function Document() {
  return (
    <html lang="en">
      <head>
        <title>Event details</title>
        <style>{`
          h1 { font-size: 24px; margin-top: 0; }
        `}</style>
      </head>
      <body>
        <App />
      </body>
    </html>
  );
}
```

```js src/index.js hidden
import { hydrateRoot } from 'react-dom/client';
import { renderToReadableStream } from 'react-dom/server';
import Document from './Document.js';
import { flushReadableStreamToFrame } from './demo-helpers.js';
import './styles.css';

async function main(frame) {
  const stream = await renderToReadableStream(<Document />);
  await flushReadableStreamToFrame(stream, frame);

  // Wait so both the fallback and hydrated content are visible.
  await new Promise(resolve => setTimeout(resolve, 1200));
  hydrateRoot(frame.contentDocument, <Document />);
}

main(document.getElementById('preview'));
```

```js src/demo-helpers.js hidden
export async function flushReadableStreamToFrame(readable, frame) {
  const doc = frame.contentWindow.document;
  const decoder = new TextDecoder();
  const reader = readable.getReader();

  while (true) {
    const {done, value} = await reader.read();
    if (done) {
      break;
    }
    doc.write(decoder.decode(value, {stream: true}));
  }

  doc.write(decoder.decode());
  doc.close();
}
```

```html public/index.html hidden
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Conditional browser rendering</title>
</head>
<body>
  <iframe id="preview" title="Rendered page"></iframe>
</body>
</html>
```

```css src/styles.css hidden
iframe {
  width: 100%;
  height: 240px;
  border: 0;
}
```

```json package.json hidden
{
  "dependencies": {
    "react": "19.3.0-canary-f1f7ed2a-20260904",
    "react-dom": "19.3.0-canary-f1f7ed2a-20260904",
    "react-scripts": "latest"
  }
}
```

</Sandpack>

注意，只有在第二种情况（未提供默认值）下，`TimeZone` 才会暂停。

这一模式的另一个用途是让 `useQuery` 之类的数据获取 Hook 跳过服务端渲染，除非已传入该查询的初始数据（例如来自 Server Component 或框架的 loader 函数）：

```js {3}
function useBrowserQuery(query, options) {
  if (options.initialData === undefined) {
    use(browser());
  }

  return useQuery(query, options);
}

function ProductDetails({ productId, initialData }) {
  const product = useBrowserQuery(`/api/products/${productId}`, {
    initialData,
  });

  return <h1>{product.name}</h1>;
}
```

现在，只要 `ProductDetails` 在服务端渲染期间收到 `initialData`，它就可以包含在 HTML 中。否则，它会暂停，直到在浏览器中渲染；此时 `useQuery` 会照常获取数据或从缓存中读取数据。

更多关于 `browser` 的信息，请[查看文档](/reference/react-dom/browser)。

---

### Trusted Types 支持 {/*trusted-types-support*/}

React 19.3 集成了浏览器的 [Trusted Types API](https://developer.mozilla.org/en-US/docs/Web/API/Trusted_Types_API)，这是一项用于防止基于 DOM 的 XSS 攻击的安全功能。当网站通过 `Content-Security-Policy: require-trusted-types-for 'script'` 强制使用 Trusted Types 时，浏览器要求传给 `innerHTML` 等注入接收点的值必须是由清理策略创建的类型化对象（`TrustedHTML`、`TrustedScript`、`TrustedScriptURL`），而不能是原始字符串。

以前，React 在将值传给 DOM API 之前总会先把它们强制转换为字符串（通过 `'' + value`），这会将 Trusted Types 对象变回浏览器会拒绝的普通字符串。现在，React 会原样传递这些值而不进行强制转换，因此浏览器可以验证它们，Trusted Types 策略也能按预期工作。

---

## React Server Components 新功能 {/*new-react-server-components-features*/}

### Server Component 可以直接渲染 `<Context>` {/*context-can-be-rendered-directly-in-server-components*/}

Server Component 虽然不能_创建_ Context，但可以从 `'use client'` 模块导入并_渲染_ Context。

以前，这需要客户端模块导出一个单独的包装组件，通常称为 Provider：

```js {7-9}
// user-context.js
'use client';
import { createContext } from 'react';

export const UserContext = createContext(null);

export function UserProvider({ currentUser, children }) {
  return <UserContext value={currentUser}>{children}</UserContext>;
}
```

```js {8}
// server-component.js
import { UserProvider } from './user-context';

export async function Layout({ children }) {
  const currentUser = await getCurrentUser();

  return (
    <UserProvider currentUser={currentUser}>
      {children}
    </UserProvider>
  )
}
```

注意，在此示例中，Provider 只负责将 Server Component 传来的 prop 直接传给 Context，没有做其他事情。

在 React 19.3 中，Server Component 可以直接从 `'use client'` 模块导入并渲染 Context，无需额外的包装组件：

```js {5}
// user-context.js
'use client';
import { createContext } from 'react';

export const UserContext = createContext(null);
```

```js {8}
// server-component.js
import { UserContext } from './user-context';

export async function Layout({ children }) {
  const currentUser = await getCurrentUser();

  return (
    <UserContext value={currentUser}>
      {children}
    </UserContext>
  )
}
```

对于那些仅用于让 Server Component 与客户端树的其余部分共享数据的 Context，这一特性尤其有用。


---

## 更新日志 {/*changelog*/}

其他重要变化
- `react`：将 Transitions 独立渲染，而不是合并到同一次渲染中，因此慢速 Transition 不再阻塞无关的 Transition [#37290](https://github.com/react/react/pull/37290)
- `react-dom`：hydration 期间在 Strict Mode 中对 Effects 进行双重调用，与客户端渲染的根保持一致 [#35961](https://github.com/react/react/pull/35961)
- `react`：在条件语句中错误使用 `use` 时添加警告 [#37104](https://github.com/react/react/pull/37104)
- `react`：在 `useActionState` 错误消息中将“form state”改为“action state” [#35790](https://github.com/react/react/pull/35790)
- `react-dom`：支持 `onFullscreenChange` 和 `onFullscreenError` 事件 [#34621](https://github.com/react/react/pull/34621)
- `react-dom`：支持 SVG 的 `maskType` 属性 [#35921](https://github.com/react/react/pull/35921)
- `react-dom`：为模块资源支持 `fetchPriority` [#36835](https://github.com/react/react/pull/36835)
- `react-dom`：React 在 Server Action 后自动重置表单时触发 `onReset` [#35176](https://github.com/react/react/pull/35176)
- `react-dom`：在 `submit` 事件中包含 `submitter` [#35590](https://github.com/react/react/pull/35590)
- `react-dom`：将 `credentialless` 识别为 iframe 的布尔属性 [#36148](https://github.com/react/react/pull/36148)
- `react-dom`：将 `resize` 事件触发的更新批处理到下一帧 [#35117](https://github.com/react/react/pull/35117)
- `react-server`：将 `Error.cause` [#35810](https://github.com/react/react/pull/35810) 和 `AggregateError.errors` [#36156](https://github.com/react/react/pull/36156) 传递到客户端
- `react-server`：在 Flight 中支持 `<Activity>` [#34697](https://github.com/react/react/pull/34697)

重要错误修复

- `react`：修复 `useDeferredValue` 卡在旧值的问题 [#36134](https://github.com/react/react/pull/36134)
- `react`：修复 Context 值未传递到 Suspense 回退内容中 [#36160](https://github.com/react/react/pull/36160)，以及未传递穿过暂停状态的 Suspense 边界的问题 [#35839](https://github.com/react/react/pull/35839)
- `react`：修复在隐藏树中更新已脱水的 Suspense 边界时发生卡死的问题 [#37135](https://github.com/react/react/pull/37135)
- `react`：修复 `<Activity>` 树隐藏期间发生的 store 变更被 `useSyncExternalStore` 遗漏的问题 [#36947](https://github.com/react/react/pull/36947)
- `react`：修复 `useEffectEvent` 在 `forwardRef` 和 `memo` 组件中读取不到最新值的问题 [#34831](https://github.com/react/react/pull/34831)
- `react`：修复组件 state 更新时表单状态被重置的问题 [#34075](https://github.com/react/react/pull/34075)
- `react`：修复若干 Fast Refresh 问题，涉及 `lazy`、`memo`，以及改变组件种类的编辑 [#36965](https://github.com/react/react/pull/36965)、[#36964](https://github.com/react/react/pull/36964)、[#36963](https://github.com/react/react/pull/36963)、[#36950](https://github.com/react/react/pull/36950)
- `react`：修复包含 `<title>` 的 `<Activity>` 从 `visible` 切换为 `hidden` 后，`<title>` 仍被提升到 `<head>` 的问题 [#34983](https://github.com/react/react/pull/34983)
- `react`：阻止错误从隐藏的 `<Activity>` 中逸出 [#35074](https://github.com/react/react/pull/35074)
- `react`：隐藏渲染在隐藏 `<Activity>` 内部的 portal 内容 [#35091](https://github.com/react/react/pull/35091)
- `react`：避免在错误消息中引用内部的 `<Offscreen>` 类型 [#35763](https://github.com/react/react/pull/35763)
- `react-dom`：修复委托处理和已聚焦元素的焦点问题 [#36010](https://github.com/react/react/pull/36010)
- `react-dom`：根据 DOM 规范规范化 capture 选项，修复 `FragmentInstance` 监听器泄漏 [#36047](https://github.com/react/react/pull/36047)
- `react-dom`：修复 `<ViewTransition>` 在 Mobile Safari 中崩溃的问题 [#35337](https://github.com/react/react/pull/35337)
- `react-dom`：修复 `<ViewTransition>` 与 `SuspenseList` 一起使用时崩溃的问题 [#35520](https://github.com/react/react/pull/35520)
- `react-dom`：更新 `type="number"` 输入框的 `defaultValue`，使其与其他输入类型一致 [#36980](https://github.com/react/react/pull/36980)
- `react-dom`：当 `innerHTML` 未发生变化时避免再次设置 [#36949](https://github.com/react/react/pull/36949)
- `react-dom`：修复 `nonce` 属性导致的误报 hydration 不匹配 [#37030](https://github.com/react/react/pull/37030)
- `react-dom`：修复 `react-dom/server` 在 Deno 中卡住的问题 [#35235](https://github.com/react/react/pull/35235)
- `react-server`：修复 `decodeReplyFromBusboy` 丢弃 `FormData` 条目的问题 [#36468](https://github.com/react/react/pull/36468)
- `react-server`：修复深层异步调用链导致的堆栈溢出 [#35612](https://github.com/react/react/pull/35612)，以及调试信息指数级增长导致的 `RangeError` [#37481](https://github.com/react/react/pull/37481)

完整变更列表请参阅 [Changelog](https://github.com/react/react/blob/main/CHANGELOG.md)。

---

_感谢 [Sam Selikoff](https://x.com/samselikoff) 撰写本文，并感谢 [Matt Carroll](https://mattcarrollcode.com/)、[Dan Abramov](https://bsky.app/profile/danabra.mov) 和 [Andrew Clark](https://x.com/acdlite) 审阅本文。_
