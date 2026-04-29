---
title: <Activity>
---

<Intro>

`<Activity>` 允许你隐藏和恢复其子组件的 UI 和内部状态。

```js
<Activity mode={visibility}>
  <Sidebar />
</Activity>
```

</Intro>

<InlineToc />

---

## 参考 {/*reference*/}

### `<Activity>` {/*activity*/}

你可以使用 Activity 来隐藏应用程序的一部分：

```js [[1, 1, "\\"hidden\\""], [2, 2, "<Sidebar />"], [3, 1, "\\"visible\\""]]
<Activity mode={isShowingSidebar ? "visible" : "hidden"}>
  <Sidebar />
</Activity>
```

当 Activity 边界处于 <CodeStep step={1}>hidden</CodeStep> 状态时，React 会使用 `display: "none"` CSS 属性在视觉上隐藏 <CodeStep step={2}>其子组件</CodeStep>。它还会销毁它们的 Effects，清理任何正在进行的订阅。

在隐藏期间，子组件仍然会因新的 props 而重新渲染，只是优先级低于其余内容。

当边界再次变为 <CodeStep step={3}>visible</CodeStep> 时，React 会恢复显示子组件，并还原它们之前的状态，同时重新创建它们的 Effects。

通过这种方式，可以将 Activity 理解为一种渲染“后台活动”的机制。与其完全丢弃那些很可能再次变为可见的内容，不如使用 Activity 来维护和恢复这些内容的 UI 和内部状态，同时确保隐藏内容不会产生不希望的副作用。

[在下方查看更多示例。](#usage)

#### 属性 {/*props*/}

* `children`：你打算显示和隐藏的 UI。
* `mode`：字符串值 `'visible'` 或 `'hidden'`。如果省略，默认为 `'visible'`。

#### 注意事项 {/*caveats*/}

- 如果 Activity 渲染在 [ViewTransition](/reference/react/ViewTransition) 内部，并且它因为 [startTransition](/reference/react/startTransition) 触发的更新而变为可见，它将激活 ViewTransition 的 `enter` 动画。如果它变为隐藏，则会激活其 `exit` 动画。
- 一个仅渲染文本的 *hidden* Activity 不会渲染任何内容，而不会渲染隐藏文本，因为没有对应的 DOM 元素可以应用可见性变更。例如，`<Activity mode="hidden"><ComponentThatJustReturnsText /></Activity>` 对于 `const ComponentThatJustReturnsText = () => "Hello, World!"` 不会在 DOM 中产生任何输出。`<Activity mode="visible"><ComponentThatJustReturnsText /></Activity>` 会渲染可见文本。

---

## 用法 {/*usage*/}

### 恢复隐藏组件的状态 {/*restoring-the-state-of-hidden-components*/}

在 React 中，当你想有条件地显示或隐藏一个组件时，通常会根据该条件挂载或卸载它：

```jsx
{isShowingSidebar && (
  <Sidebar />
)}
```

但卸载组件会销毁其内部状态，而这并不总是你想要的。

当你改为使用 Activity 边界隐藏组件时，React 会把它的状态“保存”起来，以便稍后恢复：

```jsx
<Activity mode={isShowingSidebar ? "visible" : "hidden"}>
  <Sidebar />
</Activity>
```

这使得你可以先隐藏组件，之后再以其先前的状态恢复它们。

下面这个例子有一个带可展开区域的侧边栏。你可以按下 “Overview” 来展开下面的三个子项。主应用区域还有一个按钮可以隐藏和显示侧边栏。

试着展开 Overview 区域，然后把侧边栏关闭再打开：

<Sandpack>

```js src/App.js active
import { useState } from 'react';
import Sidebar from './Sidebar.js';

export default function App() {
  const [isShowingSidebar, setIsShowingSidebar] = useState(true);

  return (
    <>
      {isShowingSidebar && (
        <Sidebar />
      )}

      <main>
        <button onClick={() => setIsShowingSidebar(!isShowingSidebar)}>
          Toggle sidebar
        </button>
        <h1>Main content</h1>
      </main>
    </>
  );
}
```

```js src/Sidebar.js
import { useState } from 'react';

export default function Sidebar() {
  const [isExpanded, setIsExpanded] = useState(false)

  return (
    <nav>
      <button onClick={() => setIsExpanded(!isExpanded)}>
        Overview
        <span className={`indicator ${isExpanded ? 'down' : 'right'}`}>
          &#9650;
        </span>
      </button>

      {isExpanded && (
        <ul>
          <li>Section 1</li>
          <li>Section 2</li>
          <li>Section 3</li>
        </ul>
      )}
    </nav>
  );
}
```

```css
body { height: 275px; margin: 0; }
#root {
  display: flex;
  gap: 10px;
  height: 100%;
}
nav {
  padding: 10px;
  background: #eee;
  font-size: 14px;
  height: 100%;
}
main {
  padding: 10px;
}
p {
  margin: 0;
}
h1 {
  margin-top: 10px;
}
.indicator {
  margin-left: 4px;
  display: inline-block;
  rotate: 90deg;
}
.indicator.down {
  rotate: 180deg;
}
```

</Sandpack>

Overview 区域始终从折叠状态开始。因为我们在 `isShowingSidebar` 变为 `false` 时卸载了侧边栏，所以它的所有内部状态都会丢失。

这正是 Activity 的绝佳使用场景。即使在视觉上隐藏侧边栏，我们也可以保留它的内部状态。

让我们把侧边栏的条件渲染替换为一个 Activity 边界：

```jsx {7,9}
// 之前
{isShowingSidebar && (
  <Sidebar />
)}

// 之后
<Activity mode={isShowingSidebar ? 'visible' : 'hidden'}>
  <Sidebar />
</Activity>
```

然后看看新的行为：

<Sandpack>

```js src/App.js active
import { Activity, useState } from 'react';

import Sidebar from './Sidebar.js';

export default function App() {
  const [isShowingSidebar, setIsShowingSidebar] = useState(true);

  return (
    <>
      <Activity mode={isShowingSidebar ? 'visible' : 'hidden'}>
        <Sidebar />
      </Activity>

      <main>
        <button onClick={() => setIsShowingSidebar(!isShowingSidebar)}>
          Toggle sidebar
        </button>
        <h1>Main content</h1>
      </main>
    </>
  );
}
```

```js src/Sidebar.js
import { useState } from 'react';

export default function Sidebar() {
  const [isExpanded, setIsExpanded] = useState(false)

  return (
    <nav>
      <button onClick={() => setIsExpanded(!isExpanded)}>
        Overview
        <span className={`indicator ${isExpanded ? 'down' : 'right'}`}>
          &#9650;
        </span>
      </button>

      {isExpanded && (
        <ul>
          <li>Section 1</li>
          <li>Section 2</li>
          <li>Section 3</li>
        </ul>
      )}
    </nav>
  );
}
```

```css
body { height: 275px; margin: 0; }
#root {
  display: flex;
  gap: 10px;
  height: 100%;
}
nav {
  padding: 10px;
  background: #eee;
  font-size: 14px;
  height: 100%;
}
main {
  padding: 10px;
}
p {
  margin: 0;
}
h1 {
  margin-top: 10px;
}
.indicator {
  margin-left: 4px;
  display: inline-block;
  rotate: 90deg;
}
.indicator.down {
  rotate: 180deg;
}
```

</Sandpack>

现在，侧边栏的内部状态已经恢复，而无需对其实现做任何更改。

---

### 恢复隐藏组件的 DOM {/*restoring-the-dom-of-hidden-components*/}

由于 Activity 边界通过 `display: none` 隐藏其子组件，因此在隐藏时，其子组件的 DOM 也会被保留。这使它们非常适合维护 UI 中用户很可能再次交互的部分里的短暂状态。

在这个示例中，Contact 选项卡中有一个 `<textarea>`，用户可以在其中输入消息。如果你输入一些文本，切换到 Home 选项卡，再切回 Contact 选项卡，草稿消息会丢失：

<Sandpack>

```js src/App.js
import { useState } from 'react';
import TabButton from './TabButton.js';
import Home from './Home.js';
import Contact from './Contact.js';

export default function App() {
  const [activeTab, setActiveTab] = useState('contact');

  return (
    <>
      <TabButton
        isActive={activeTab === 'home'}
        onClick={() => setActiveTab('home')}
      >
        Home
      </TabButton>
      <TabButton
        isActive={activeTab === 'contact'}
        onClick={() => setActiveTab('contact')}
      >
        Contact
      </TabButton>

      <hr />

      {activeTab === 'home' && <Home />}
      {activeTab === 'contact' && <Contact />}
    </>
  );
}
```

```js src/TabButton.js
export default function TabButton({ onClick, children, isActive }) {
  if (isActive) {
    return <b>{children}</b>
  }

  return (
    <button onClick={onClick}>
      {children}
    </button>
  );
}
```

```js src/Home.js
export default function Home() {
  return (
    <p>Welcome to my profile!</p>
  );
}
```

```js src/Contact.js active
export default function Contact() {
  return (
    <div>
      <p>Send me a message!</p>

      <textarea />

      <p>You can find me online here:</p>
      <ul>
        <li>admin@mysite.com</li>
        <li>+123456789</li>
      </ul>
    </div>
  );
}
```

```css
body { height: 275px; }
button { margin-right: 10px }
b { display: inline-block; margin-right: 10px; }
.pending { color: #777; }
```

</Sandpack>

这是因为我们在 `App` 中完全卸载了 `Contact`。当 Contact 选项卡卸载时，`<textarea>` 元素的内部 DOM 状态就会丢失。

如果我们改为使用 Activity 边界来显示和隐藏当前选项卡，就可以保留每个选项卡的 DOM 状态。试着输入文本并再次切换选项卡，你会看到草稿消息不再被重置：

<Sandpack>

```js src/App.js active
import { Activity, useState } from 'react';
import TabButton from './TabButton.js';
import Home from './Home.js';
import Contact from './Contact.js';

export default function App() {
  const [activeTab, setActiveTab] = useState('contact');

  return (
    <>
      <TabButton
        isActive={activeTab === 'home'}
        onClick={() => setActiveTab('home')}
      >
        Home
      </TabButton>
      <TabButton
        isActive={activeTab === 'contact'}
        onClick={() => setActiveTab('contact')}
      >
        Contact
      </TabButton>

      <hr />

      <Activity mode={activeTab === 'home' ? 'visible' : 'hidden'}>
        <Home />
      </Activity>
      <Activity mode={activeTab === 'contact' ? 'visible' : 'hidden'}>
        <Contact />
      </Activity>
    </>
  );
}
```

```js src/TabButton.js
export default function TabButton({ onClick, children, isActive }) {
  if (isActive) {
    return <b>{children}</b>
  }

  return (
    <button onClick={onClick}>
      {children}
    </button>
  );
}
```

```js src/Home.js
export default function Home() {
  return (
    <p>Welcome to my profile!</p>
  );
}
```

```js src/Contact.js
export default function Contact() {
  return (
    <div>
      <p>Send me a message!</p>

      <textarea />

      <p>You can find me online here:</p>
      <ul>
        <li>admin@mysite.com</li>
        <li>+123456789</li>
      </ul>
    </div>
  );
}
```

```css
body { height: 275px; }
button { margin-right: 10px }
b { display: inline-block; margin-right: 10px; }
.pending { color: #777; }
```

</Sandpack>

同样地，Activity 边界让我们在不改变 Contact 选项卡实现的情况下保留了它的内部状态。

---

### 预渲染可能即将变为可见的内容 {/*pre-rendering-content-thats-likely-to-become-visible*/}

到目前为止，我们已经看到 Activity 如何在不丢弃内容短暂状态的情况下，隐藏用户已经交互过的某些内容。

但 Activity 边界也可用于为用户尚未首次看到的内容做“准备”：

```jsx [[1, 1, "\\"hidden\\""]]
<Activity mode="hidden">
  <SlowComponent />
</Activity>
```

当 Activity 边界在初始渲染期间处于 <CodeStep step={1}>hidden</CodeStep> 状态时，它的子组件不会在页面上可见——但它们仍然会被渲染，只是优先级低于可见内容，并且不会挂载它们的 Effects。

这种“预渲染”允许子组件提前加载所需的任何代码或数据，从而在稍后 Activity 边界变为可见时，子组件能够以更快的速度显示，减少加载时间。

让我们看一个例子。

在这个演示中，Posts 选项卡会加载一些数据。如果你点击它，你会看到在数据获取期间显示一个 Suspense 回退内容：

<Sandpack>

```js src/App.js
import { useState, Suspense } from 'react';
import TabButton from './TabButton.js';
import Home from './Home.js';
import Posts from './Posts.js';

export default function App() {
  const [activeTab, setActiveTab] = useState('home');

  return (
    <>
      <TabButton
        isActive={activeTab === 'home'}
        onClick={() => setActiveTab('home')}
      >
        Home
      </TabButton>
      <TabButton
        isActive={activeTab === 'posts'}
        onClick={() => setActiveTab('posts')}
      >
        Posts
      </TabButton>

      <hr />

      <Suspense fallback={<h1>🌀 Loading...</h1>}>
        {activeTab === 'home' && <Home />}
        {activeTab === 'posts' && <Posts />}
      </Suspense>
    </>
  );
}
```

```js src/TabButton.js hidden
export default function TabButton({ onClick, children, isActive }) {
  if (isActive) {
    return <b>{children}</b>
  }

  return (
    <button onClick={onClick}>
      {children}
    </button>
  );
}
```

```js src/Home.js
export default function Home() {
  return (
    <p>Welcome to my profile!</p>
  );
}
```

```js src/Posts.js
import { use } from 'react';
import { fetchData } from './data.js';

export default function Posts() {
  const posts = use(fetchData('/posts'));

  return (
    <ul className="items">
      {posts.map(post =>
        <li className="item" key={post.id}>
          {post.title}
        </li>
      )}
    </ul>
  );
}
```

```js src/data.js hidden
// 注意：你如何进行数据获取取决于
// 你与 Suspense 一起使用的框架。
// 通常，缓存逻辑会在框架内部。

let cache = new Map();

export function fetchData(url) {
  if (!cache.has(url)) {
    cache.set(url, getData(url));
  }
  return cache.get(url);
}

async function getData(url) {
  if (url.startsWith('/posts')) {
    return await getPosts();
  } else {
    throw Error('Not implemented');
  }
}

async function getPosts() {
  // 添加一个假的延迟，使等待更明显。
  await new Promise(resolve => {
    setTimeout(resolve, 1000);
  });
  let posts = [];
  for (let i = 0; i < 10; i++) {
    posts.push({
      id: i,
      title: 'Post #' + (i + 1)
    });
  }
  return posts;
}
```

```css
body { height: 275px; }
button { margin-right: 10px }
b { display: inline-block; margin-right: 10px; }
.pending { color: #777; }
video { width: 300px; margin-top: 10px; aspect-ratio: 16/9; }
```

</Sandpack>

这是因为 `App` 直到 `Posts` 选项卡处于激活状态才会挂载它。

如果我们更新 `App`，使用 Activity 边界来显示和隐藏当前选项卡，那么在应用首次加载时，`Posts` 就会被预渲染，这样在它变为可见之前就能先获取数据。

现在试着点击 Posts 选项卡：

<Sandpack>

```js src/App.js
import { Activity, useState, Suspense } from 'react';
import TabButton from './TabButton.js';
import Home from './Home.js';
import Posts from './Posts.js';

export default function App() {
  const [activeTab, setActiveTab] = useState('home');

  return (
    <>
      <TabButton
        isActive={activeTab === 'home'}
        onClick={() => setActiveTab('home')}
      >
        Home
      </TabButton>
      <TabButton
        isActive={activeTab === 'posts'}
        onClick={() => setActiveTab('posts')}
      >
        Posts
      </TabButton>

      <hr />

      <Suspense fallback={<h1>🌀 Loading...</h1>}>
        <Activity mode={activeTab === 'home' ? 'visible' : 'hidden'}>
          <Home />
        </Activity>
        <Activity mode={activeTab === 'posts' ? 'visible' : 'hidden'}>
          <Posts />
        </Activity>
      </Suspense>
    </>
  );
}
```

```js src/TabButton.js hidden
export default function TabButton({ onClick, children, isActive }) {
  if (isActive) {
    return <b>{children}</b>
  }

  return (
    <button onClick={onClick}>
      {children}
    </button>
  );
}
```

```js src/Home.js
export default function Home() {
  return (
    <p>Welcome to my profile!</p>
  );
}
```

```js src/Posts.js
import { use } from 'react';
import { fetchData } from './data.js';

export default function Posts() {
  const posts = use(fetchData('/posts'));

  return (
    <ul className="items">
      {posts.map(post =>
        <li className="item" key={post.id}>
          {post.title}
        </li>
      )}
    </ul>
  );
}
```

```js src/data.js hidden
// 注意：你如何进行数据获取取决于
// 你与 Suspense 一起使用的框架。
// 通常，缓存逻辑会在框架内部。

let cache = new Map();

export function fetchData(url) {
  if (!cache.has(url)) {
    cache.set(url, getData(url));
  }
  return cache.get(url);
}

async function getData(url) {
  if (url.startsWith('/posts')) {
    return await getPosts();
  } else {
    throw Error('Not implemented');
  }
}

async function getPosts() {
  // 添加一个假的延迟，使等待更明显。
  await new Promise(resolve => {
    setTimeout(resolve, 1000);
  });
  let posts = [];
  for (let i = 0; i < 10; i++) {
    posts.push({
      id: i,
      title: 'Post #' + (i + 1)
    });
  }
  return posts;
}
```

```css
body { height: 275px; }
button { margin-right: 10px }
b { display: inline-block; margin-right: 10px; }
.pending { color: #777; }
video { width: 300px; margin-top: 10px; aspect-ratio: 16/9; }
```

</Sandpack>

`Posts` 已经能够为更快的渲染做好准备，这要归功于隐藏的 Activity 边界。

---

使用隐藏的 Activity 边界预渲染组件，是减少用户很可能下一步会交互的 UI 部分加载时间的一种强大方式。

<Note>

**只有支持 Suspense 的数据源才会在预渲染期间被获取。** 它们包括：

- 使用支持 Suspense 的框架进行数据获取，例如 [Relay](https://relay.dev/docs/guided-tour/rendering/loading-states/) 和 [Next.js](https://nextjs.org/docs/app/building-your-application/routing/loading-ui-and-streaming#streaming-with-suspense)
- 使用 [`lazy`](/reference/react/lazy) 懒加载组件代码
- 使用 [`use`](/reference/react/use) 读取缓存 Promise 的值

Activity **不会**检测在 Effect 中获取的数据。

上面 `Posts` 组件中如何加载数据，具体取决于你的框架。如果你使用支持 Suspense 的框架，你可以在其数据获取文档中找到详细信息。

目前尚不支持在不使用特定框架的情况下进行支持 Suspense 的数据获取。实现支持 Suspense 的数据源所需的条件是不稳定且未文档化的。React 的未来版本将提供一个用于将数据源与 Suspense 集成的官方 API。

</Note>

---


### 加快页面加载期间的交互速度 {/*speeding-up-interactions-during-page-load*/}

React 内部包含一种称为 Selective Hydration 的性能优化。它通过“分块”对应用的初始 HTML 进行 hydration，使某些组件即使在页面上的其他组件尚未加载其代码或数据时，也能变得可交互。

Suspense 边界参与了 Selective Hydration，因为它们天然地将组件树划分为彼此独立的单元：

```jsx
function Page() {
  return (
    <>
      <MessageComposer />

      <Suspense fallback="Loading chats...">
        <Chats />
      </Suspense>
    </>
  )
}
```

在这里，`MessageComposer` 可以在页面初始渲染期间完全完成 hydration，甚至可以早于 `Chats` 挂载并开始获取数据。

因此，通过将组件树拆分为离散单元，Suspense 允许 React 将服务端渲染的 HTML 分块进行 hydration，从而使应用的各个部分尽可能快地变得可交互。

但如果页面没有使用 Suspense 呢？

来看这个选项卡示例：

```jsx
function Page() {
  const [activeTab, setActiveTab] = useState('home');

  return (
    <>
      <TabButton onClick={() => setActiveTab('home')}>
        Home
      </TabButton>
      <TabButton onClick={() => setActiveTab('video')}>
        Video
      </TabButton>

      {activeTab === 'home' && (
        <Home />
      )}
      {activeTab === 'video' && (
        <Video />
      )}
    </>
  )
}
```

在这里，React 必须一次性对整个页面进行 hydration。如果 `Home` 或 `Video` 的渲染更慢，它们可能会让选项卡按钮在 hydration 期间显得没有响应。

在活动选项卡周围添加 Suspense 可以解决这个问题：

```jsx {13,20}
function Page() {
  const [activeTab, setActiveTab] = useState('home');

  return (
    <>
      <TabButton onClick={() => setActiveTab('home')}>
        Home
      </TabButton>
      <TabButton onClick={() => setActiveTab('video')}>
        Video
      </TabButton>

      <Suspense fallback={<Placeholder />}>
        {activeTab === 'home' && (
          <Home />
        )}
        {activeTab === 'video' && (
          <Video />
        )}
      </Suspense>
    </>
  )
}
```

……但这也会改变 UI，因为 `Placeholder` 回退内容会在初始渲染时显示出来。

相反，我们可以使用 Activity。由于 Activity 边界会显示和隐藏其子组件，它们天然地将组件树划分为独立单元。而且和 Suspense 一样，这个特性使它们能够参与 Selective Hydration。

让我们更新示例，在活动选项卡周围使用 Activity 边界：

```jsx {13-18}
function Page() {
  const [activeTab, setActiveTab] = useState('home');

  return (
    <>
      <TabButton onClick={() => setActiveTab('home')}>
        Home
      </TabButton>
      <TabButton onClick={() => setActiveTab('video')}>
        Video
      </TabButton>

      <Activity mode={activeTab === "home" ? "visible" : "hidden"}>
        <Home />
      </Activity>
      <Activity mode={activeTab === "video" ? "visible" : "hidden"}>
        <Video />
      </Activity>
    </>
  )
}
```

现在，我们最初服务端渲染的 HTML 看起来与原始版本相同，但借助 Activity，React 可以先对选项卡按钮进行 hydration，甚至在挂载 `Home` 或 `Video` 之前就开始。

---

因此，除了隐藏和显示内容之外，Activity 边界还通过让 React 知道页面中哪些部分可以独立变得可交互，帮助提升应用在 hydration 过程中的性能。

即使你的页面从不隐藏其内容的一部分，你仍然可以添加始终可见的 Activity 边界来提升 hydration 性能：

```jsx
function Page() {
  return (
    <>
      <Post />

      <Activity>
        <Comments />
      </Activity>
    </>
  );
}
```

---

## 故障排查 {/*troubleshooting*/}

### 我的隐藏组件有不希望出现的副作用 {/*my-hidden-components-have-unwanted-side-effects*/}

Activity 边界会通过将其子组件设为 `display: none` 并清理它们的任何 Effects 来隐藏其内容。因此，大多数行为良好的 React 组件，只要正确清理了副作用，就已经能够很好地应对被 Activity 隐藏的情况。

但在某些情况下，隐藏组件的行为会与卸载组件不同。最明显的是，由于隐藏组件的 DOM 并未被销毁，来自该 DOM 的任何副作用都会持续存在，即使组件已经被隐藏也是如此。

例如，考虑一个 `<video>` 标签。通常它不需要任何清理，因为即使你正在播放视频，卸载该标签也会停止浏览器中的视频和音频播放。试着播放下面这个示例中的视频，然后按 Home：

<Sandpack>

```js src/App.js active
import { useState } from 'react';
import TabButton from './TabButton.js';
import Home from './Home.js';
import Video from './Video.js';

export default function App() {
  const [activeTab, setActiveTab] = useState('video');

  return (
    <>
      <TabButton
        isActive={activeTab === 'home'}
        onClick={() => setActiveTab('home')}
      >
        Home
      </TabButton>
      <TabButton
        isActive={activeTab === 'video'}
        onClick={() => setActiveTab('video')}
      >
        Video
      </TabButton>

      <hr />

      {activeTab === 'home' && <Home />}
      {activeTab === 'video' && <Video />}
    </>
  );
}
```

```js src/TabButton.js hidden
export default function TabButton({ onClick, children, isActive }) {
  if (isActive) {
    return <b>{children}</b>
  }

  return (
    <button onClick={onClick}>
      {children}
    </button>
  );
}
```

```js src/Home.js
export default function Home() {
  return (
    <p>欢迎来到我的个人资料！</p>
  );
}
```

```js src/Video.js
export default function Video() {
  return (
    <video
      // “Big Buck Bunny” 由 Blender 基金会根据 CC 3.0 许可发布。由 archive.org 托管
      src="https://archive.org/download/BigBuckBunny_124/Content/big_buck_bunny_720p_surround.mp4"
      controls
      playsInline
    />

  );
}
```

```css
body { height: 275px; }
button { margin-right: 10px }
b { display: inline-block; margin-right: 10px; }
.pending { color: #777; }
video { width: 300px; margin-top: 10px; aspect-ratio: 16/9; }
```

</Sandpack>

视频会按预期停止播放。

现在，假设我们想保留用户上次观看到的时间点，这样当他们切回视频时，就不会从头开始播放。

这正是 Activity 的一个很好的使用场景！

让我们更新 `App`，用隐藏的 Activity 边界来隐藏未激活的标签页，而不是卸载它，并看看这次示例的表现：

<Sandpack>

```js src/App.js active
import { Activity, useState } from 'react';
import TabButton from './TabButton.js';
import Home from './Home.js';
import Video from './Video.js';

export default function App() {
  const [activeTab, setActiveTab] = useState('video');

  return (
    <>
      <TabButton
        isActive={activeTab === 'home'}
        onClick={() => setActiveTab('home')}
      >
        Home
      </TabButton>
      <TabButton
        isActive={activeTab === 'video'}
        onClick={() => setActiveTab('video')}
      >
        Video
      </TabButton>

      <hr />

      <Activity mode={activeTab === 'home' ? 'visible' : 'hidden'}>
        <Home />
      </Activity>
      <Activity mode={activeTab === 'video' ? 'visible' : 'hidden'}>
        <Video />
      </Activity>
    </>
  );
}
```

```js src/TabButton.js hidden
export default function TabButton({ onClick, children, isActive }) {
  if (isActive) {
    return <b>{children}</b>
  }

  return (
    <button onClick={onClick}>
      {children}
    </button>
  );
}
```

```js src/Home.js
export default function Home() {
  return (
    <p>欢迎来到我的个人资料！</p>
  );
}
```

```js src/Video.js
export default function Video() {
  return (
    <video
      controls
      playsInline
      // “Big Buck Bunny” 由 Blender 基金会根据 CC 3.0 许可发布。由 archive.org 托管
      src="https://archive.org/download/BigBuckBunny_124/Content/big_buck_bunny_720p_surround.mp4"
    />

  );
}
```

```css
body { height: 275px; }
button { margin-right: 10px }
b { display: inline-block; margin-right: 10px; }
.pending { color: #777; }
video { width: 300px; margin-top: 10px; aspect-ratio: 16/9; }
```

</Sandpack>

糟了！即使标签页已经被隐藏，视频和音频仍然会继续播放，因为该标签页的 `<video>` 元素仍然保留在 DOM 中。

为了解决这个问题，我们可以添加一个带有清理函数的 Effect，在清理时暂停视频：

```jsx {2,4-10,14}
export default function VideoTab() {
  const ref = useRef();

  useLayoutEffect(() => {
    const videoRef = ref.current;

    return () => {
      videoRef.pause()
    }
  }, []);

  return (
    <video
      ref={ref}
      controls
      playsInline
      src="..."
    />

  );
}
```

我们使用 `useLayoutEffect` 而不是 `useEffect`，因为从概念上讲，清理代码与组件 UI 在视觉上被隐藏这件事是绑定的。如果使用普通的 effect，这段代码可能会因为（比如说）重新挂起的 Suspense 边界或视图过渡而被延迟。

让我们看看新的行为。试着播放视频，切换到 Home 标签页，然后再切回 Video 标签页：

<Sandpack>

```js src/App.js active
import { Activity, useState } from 'react';
import TabButton from './TabButton.js';
import Home from './Home.js';
import Video from './Video.js';

export default function App() {
  const [activeTab, setActiveTab] = useState('video');

  return (
    <>
      <TabButton
        isActive={activeTab === 'home'}
        onClick={() => setActiveTab('home')}
      >
        Home
      </TabButton>
      <TabButton
        isActive={activeTab === 'video'}
        onClick={() => setActiveTab('video')}
      >
        Video
      </TabButton>

      <hr />

      <Activity mode={activeTab === 'home' ? 'visible' : 'hidden'}>
        <Home />
      </Activity>
      <Activity mode={activeTab === 'video' ? 'visible' : 'hidden'}>
        <Video />
      </Activity>
    </>
  );
}
```

```js src/TabButton.js hidden
export default function TabButton({ onClick, children, isActive }) {
  if (isActive) {
    return <b>{children}</b>
  }

  return (
    <button onClick={onClick}>
      {children}
    </button>
  );
}
```

```js src/Home.js
export default function Home() {
  return (
    <p>欢迎来到我的个人资料！</p>
  );
}
```

```js src/Video.js
import { useRef, useLayoutEffect } from 'react';

export default function Video() {
  const ref = useRef();

  useLayoutEffect(() => {
    const videoRef = ref.current

    return () => {
      videoRef.pause()
    };
  }, [])

  return (
    <video
      ref={ref}
      controls
      playsInline
      // “Big Buck Bunny” 由 Blender 基金会根据 CC 3.0 许可发布。由 archive.org 托管
      src="https://archive.org/download/BigBuckBunny_124/Content/big_buck_bunny_720p_surround.mp4"
    />

  );
}
```

```css
body { height: 275px; }
button { margin-right: 10px }
b { display: inline-block; margin-right: 10px; }
.pending { color: #777; }
video { width: 300px; margin-top: 10px; aspect-ratio: 16/9; }
```

</Sandpack>

效果很好！我们的清理函数确保视频在被 Activity 边界隐藏时会停止播放；更棒的是，由于 `<video>` 标签从未被销毁，时间点会被保留，而且当用户切回来继续观看时，视频本身也无需重新初始化或再次下载。

这是一个很好的例子，说明如何使用 Activity 来保留那些会被隐藏、但用户很可能很快再次交互的 UI 部分的短暂 DOM 状态。

---

我们的示例说明了，对于某些像 `<video>` 这样的标签，卸载和隐藏会产生不同的行为。如果某个组件渲染了带有副作用的 DOM，并且你希望在 Activity 边界隐藏它时阻止该副作用，就添加一个带有返回函数的 Effect 来清理它。

最常见的情况来自以下标签：

  - `<video>`
  - `<audio>`
  - `<iframe>`

不过通常来说，你的大多数 React 组件本来就应该能够很好地应对被 Activity 边界隐藏。而从概念上讲，你应该把“隐藏”的 Activities 视为已卸载。

为了主动发现其他没有正确清理的 Effects——这不仅对 Activity 边界很重要，对 React 中许多其他行为也很重要——我们建议使用 [`<StrictMode>`](/reference/react/StrictMode)。

---


### 我的隐藏组件有没有运行的 Effects {/*my-hidden-components-have-effects-that-arent-running*/}

当 `<Activity>` 处于“hidden”时，它所有子组件的 Effects 都会被清理。从概念上讲，这些子组件会被卸载，但 React 会将它们的状态保存起来，以便之后使用。这是 Activity 的一个特性，因为这意味着订阅不会对 UI 中隐藏的部分保持激活，从而减少处理隐藏内容所需的工作量。

如果你依赖某个 Effect 在挂载时去清理组件的副作用，请重构该 Effect，把工作放到返回的清理函数中去完成。

为了尽早发现有问题的 Effects，我们建议添加 [`<StrictMode>`](/reference/react/StrictMode)，它会主动执行 Activity 的卸载和挂载，以捕获任何意料之外的副作用。
