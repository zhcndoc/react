---
title: 响应事件
---

<Intro>

React 让你可以向 JSX 中添加 *事件处理函数*。事件处理函数是你自己编写的函数，它们会在点击、悬停、聚焦表单输入框等交互发生时被触发。

</Intro>

<YouWillLearn>

* 编写事件处理函数的不同方式
* 如何从父组件传递事件处理逻辑
* 事件如何传播以及如何阻止传播

</YouWillLearn>

## 添加事件处理函数 {/*adding-event-handlers*/}

要添加一个事件处理函数，你首先要定义一个函数，然后将它[作为 prop 传递](/learn/passing-props-to-a-component)给对应的 JSX 标签。例如，这里有一个目前还不会做任何事情的按钮：

<Sandpack>

```js
export default function Button() {
  return (
    <button>
      我什么都不会做
    </button>
  );
}
```

</Sandpack>

你可以通过以下三个步骤，让它在用户点击时显示一条消息：

1. 在你的 `Button` 组件*内部*声明一个名为 `handleClick` 的函数。
2. 在该函数内部实现逻辑（使用 `alert` 显示消息）。
3. 将 `onClick={handleClick}` 添加到 `<button>` JSX 中。

<Sandpack>

```js
export default function Button() {
  function handleClick() {
    alert('你点击了我！');
  }

  return (
    <button onClick={handleClick}>
      点击我
    </button>
  );
}
```

```css
button { margin-right: 10px; }
```

</Sandpack>

你定义了 `handleClick` 函数，然后将它[作为 prop 传递](/learn/passing-props-to-a-component)给了 `<button>`。`handleClick` 是一个**事件处理函数。** 事件处理函数通常：

* 定义在*组件内部*。
* 名称以 `handle` 开头，后跟事件名称。

按照惯例，通常会将事件处理函数命名为 `handle` 加上事件名。你经常会看到 `onClick={handleClick}`、`onMouseEnter={handleMouseEnter}` 等写法。

另外，你也可以在 JSX 中直接内联定义事件处理函数：

```jsx
<button onClick={function handleClick() {
  alert('你点击了我！');
}}>
```

或者更简洁地，使用箭头函数：

```jsx
<button onClick={() => {
  alert('你点击了我！');
}}>
```

这些写法都是等价的。对于较短的函数来说，内联事件处理函数很方便。

<Pitfall>

传递给事件处理函数的应该是函数本身，而不是调用它。例如：

| 传递一个函数（正确）     | 调用一个函数（错误）     |
| -------------------------------- | ---------------------------------- |
| `<button onClick={handleClick}>` | `<button onClick={handleClick()}>` |

区别很微妙。在第一个示例中，`handleClick` 函数作为 `onClick` 事件处理函数被传递。这样 React 就会记住它，并且只会在用户点击按钮时调用你的函数。

在第二个示例中，`handleClick()` 末尾的 `()` 会在[渲染](/learn/render-and-commit)过程中*立即*执行函数，而不是等到点击发生。这是因为 [JSX 中的 `{` 和 `}`](/learn/javascript-in-jsx-with-curly-braces) 里的 JavaScript 会立刻执行。

当你内联编写代码时，同样的陷阱会以另一种方式出现：

| 传递一个函数（正确）            | 调用一个函数（错误）    |
| --------------------------------------- | --------------------------------- |
| `<button onClick={() => alert('...')}>` | `<button onClick={alert('...')}>` |


像这样传入内联代码并不会在点击时触发——它会在组件每次渲染时执行：

```jsx
// 这个 alert 会在组件渲染时触发，而不是在点击时触发！
<button onClick={alert('你点击了我！')}>
```

如果你想内联定义事件处理函数，请像这样把它包在匿名函数里：

```jsx
<button onClick={() => alert('你点击了我！')}>
```

这样不会在每次渲染时执行里面的代码，而是创建一个稍后会被调用的函数。

无论哪种情况，你要传递的都应该是一个函数：

* `<button onClick={handleClick}>` 传递的是 `handleClick` 函数。
* `<button onClick={() => alert('...')}>` 传递的是 `() => alert('...')` 函数。

[阅读更多关于箭头函数的内容。](https://javascript.info/arrow-functions-basics)

</Pitfall>

### 在事件处理函数中读取 props {/*reading-props-in-event-handlers*/}

因为事件处理函数是在组件内部声明的，所以它们可以访问组件的 props。下面这个按钮在被点击时，会弹出一个包含其 `message` prop 的提示框：

<Sandpack>

```js
function AlertButton({ message, children }) {
  return (
    <button onClick={() => alert(message)}>
      {children}
    </button>
  );
}

export default function Toolbar() {
  return (
    <div>
      <AlertButton message="Playing!">
        播放电影
      </AlertButton>
      <AlertButton message="Uploading!">
        上传图片
      </AlertButton>
    </div>
  );
}
```

```css
button { margin-right: 10px; }
```

</Sandpack>

这让这两个按钮可以显示不同的消息。试着修改传给它们的消息。

### 将事件处理函数作为 props 传递 {/*passing-event-handlers-as-props*/}

通常你会希望父组件指定子组件的事件处理函数。以按钮为例：根据你在什么地方使用 `Button` 组件，你可能希望执行不同的函数——一个播放电影，另一个上传图片。

要做到这一点，可以像这样把组件从父组件接收到的 prop 作为事件处理函数传递进去：

<Sandpack>

```js
function Button({ onClick, children }) {
  return (
    <button onClick={onClick}>
      {children}
    </button>
  );
}

function PlayButton({ movieName }) {
  function handlePlayClick() {
    alert(`正在播放 ${movieName}！`);
  }

  return (
    <Button onClick={handlePlayClick}>
      播放 "{movieName}"
    </Button>
  );
}

function UploadButton() {
  return (
    <Button onClick={() => alert('Uploading!')}>
      上传图片
    </Button>
  );
}

export default function Toolbar() {
  return (
    <div>
      <PlayButton movieName="Kiki's Delivery Service" />
      <UploadButton />
    </div>
  );
}
```

```css
button { margin-right: 10px; }
```

</Sandpack>

这里，`Toolbar` 组件渲染了一个 `PlayButton` 和一个 `UploadButton`：

- `PlayButton` 将 `handlePlayClick` 作为 `onClick` prop 传给内部的 `Button`。
- `UploadButton` 将 `() => alert('Uploading!')` 作为 `onClick` prop 传给内部的 `Button`。

最后，你的 `Button` 组件接收一个名为 `onClick` 的 prop。它把这个 prop 直接传给浏览器内置的 `<button>`，使用 `onClick={onClick}`。这会告诉 React 在点击时调用传入的函数。

如果你使用[设计系统](https://uxdesign.cc/everything-you-need-to-know-about-design-systems-54b109851969)，按钮之类的组件通常只负责样式，而不指定行为。相反，像 `PlayButton` 和 `UploadButton` 这样的组件会向下传递事件处理函数。

### 为事件处理函数 prop 命名 {/*naming-event-handler-props*/}

像 `<button>` 和 `<div>` 这样的内置组件只支持诸如 `onClick` 之类的[浏览器事件名称](/reference/react-dom/components/common#common-props)。不过，当你构建自己的组件时，你可以随意命名它们的事件处理函数 prop。

按照惯例，事件处理函数 prop 应该以 `on` 开头，后跟一个大写字母。

例如，`Button` 组件的 `onClick` prop 也可以叫做 `onSmash`：

<Sandpack>

```js
function Button({ onSmash, children }) {
  return (
    <button onClick={onSmash}>
      {children}
    </button>
  );
}

export default function App() {
  return (
    <div>
      <Button onSmash={() => alert('Playing!')}>
        播放电影
      </Button>
      <Button onSmash={() => alert('Uploading!')}>
        上传图片
      </Button>
    </div>
  );
}
```

```css
button { margin-right: 10px; }
```

</Sandpack>

在这个示例中，`<button onClick={onSmash}>` 表明浏览器的 `<button>`（小写）仍然需要一个名为 `onClick` 的 prop，但你的自定义 `Button` 组件接收的 prop 名称由你决定！

当你的组件支持多个交互时，你可以根据应用中的具体概念来命名事件处理函数 prop。例如，这个 `Toolbar` 组件接收 `onPlayMovie` 和 `onUploadImage` 这两个事件处理函数：

<Sandpack>

```js
export default function App() {
  return (
    <Toolbar
      onPlayMovie={() => alert('Playing!')}
      onUploadImage={() => alert('Uploading!')}
    />
  );
}

function Toolbar({ onPlayMovie, onUploadImage }) {
  return (
    <div>
      <Button onClick={onPlayMovie}>
        播放电影
      </Button>
      <Button onClick={onUploadImage}>
        上传图片
      </Button>
    </div>
  );
}

function Button({ onClick, children }) {
  return (
    <button onClick={onClick}>
      {children}
    </button>
  );
}
```

```css
button { margin-right: 10px; }
```

</Sandpack>

注意 `App` 组件并不需要知道 `Toolbar` 会如何处理 `onPlayMovie` 或 `onUploadImage`。这属于 `Toolbar` 的实现细节。这里，`Toolbar` 将它们作为 `onClick` 处理函数传给了它内部的 `Button`，但以后也可以在键盘快捷键触发时调用它们。根据应用中的具体交互来命名 prop，比如 `onPlayMovie`，能让你在以后灵活改变它们的使用方式。

<Note>

请确保为你的事件处理函数使用合适的 HTML 标签。例如，要处理点击事件，请使用 [`<button onClick={handleClick}>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/button) 而不是 `<div onClick={handleClick}>`。使用真正的浏览器 `<button>` 可以启用内置的浏览器行为，例如键盘导航。如果你不喜欢按钮默认的浏览器样式，并且希望它看起来更像链接或其他 UI 元素，可以用 CSS 来实现。 [了解更多关于编写可访问标记的信息。](https://developer.mozilla.org/en-US/docs/Learn/Accessibility/HTML)

</Note>

## 事件传播 {/*event-propagation*/}

事件处理函数也会捕获组件子元素触发的事件。我们说事件会在树中“冒泡”或“传播”：它从事件发生的位置开始，然后向上遍历组件树。

这个 `<div>` 包含两个按钮。`<div>` 和每个按钮都有自己的 `onClick` 处理函数。你觉得点击按钮时会触发哪些处理函数？

<Sandpack>

```js
export default function Toolbar() {
  return (
    <div className="Toolbar" onClick={() => {
      alert('你点击了工具栏！');
    }}>
      <button onClick={() => alert('Playing!')}>
        播放电影
      </button>
      <button onClick={() => alert('Uploading!')}>
        上传图片
      </button>
    </div>
  );
}
```

```css
.Toolbar {
  background: #aaa;
  padding: 5px;
}
button { margin: 5px; }
```

</Sandpack>

如果你点击任意一个按钮，它的 `onClick` 会先运行，然后父级 `<div>` 的 `onClick` 也会运行。所以会出现两条消息。如果你点击工具栏本身，则只会运行父级 `<div>` 的 `onClick`。

<Pitfall>

在 React 中，除了 `onScroll` 之外，所有事件都会传播，`onScroll` 只在你附加它的 JSX 标签上生效。

</Pitfall>

### 阻止传播 {/*stopping-propagation*/}

事件处理函数接收一个 **事件对象** 作为唯一参数。按照惯例，它通常被称为 `e`，代表 “event”。你可以使用这个对象来读取事件信息。

这个事件对象也能让你阻止传播。如果你想阻止事件到达父组件，就需要像下面这个 `Button` 组件一样调用 `e.stopPropagation()`：

<Sandpack>

```js
function Button({ onClick, children }) {
  return (
    <button onClick={e => {
      e.stopPropagation();
      onClick();
    }}>
      {children}
    </button>
  );
}

export default function Toolbar() {
  return (
    <div className="Toolbar" onClick={() => {
      alert('你点击了工具栏！');
    }}>
      <Button onClick={() => alert('Playing!')}>
        播放电影
      </Button>
      <Button onClick={() => alert('Uploading!')}>
        上传图片
      </Button>
    </div>
  );
}
```

```css
.Toolbar {
  background: #aaa;
  padding: 5px;
}
button { margin: 5px; }
```

</Sandpack>

当你点击按钮时：

1. React 调用传给 `<button>` 的 `onClick` 处理函数。
2. 这个在 `Button` 中定义的处理函数会执行以下操作：
   * 调用 `e.stopPropagation()`，阻止事件继续向上冒泡。
   * 调用 `onClick` 函数，这是从 `Toolbar` 组件传入的 prop。
3. 这个在 `Toolbar` 组件中定义的函数会显示按钮自己的提示信息。
4. 由于传播已被阻止，父级 `<div>` 的 `onClick` 处理函数*不会*运行。

由于调用了 `e.stopPropagation()`，现在点击按钮时只会显示一个提示框（来自 `<button>`），而不是两个（来自 `<button>` 和父级工具栏 `<div>`）。点击按钮和点击周围的工具栏不是一回事，所以在这个 UI 中阻止传播是合理的。

<DeepDive>

#### 捕获阶段事件 {/*capture-phase-events*/}

在少数情况下，即使子元素已经阻止传播，你也可能需要捕获所有子元素上的事件。例如，也许你想把每次点击都记录到分析系统中，而不管传播逻辑如何。你可以通过在事件名末尾添加 `Capture` 来实现：

```js
<div onClickCapture={() => { /* 这里先执行 */ }}>
  <button onClick={e => e.stopPropagation()} />
  <button onClick={e => e.stopPropagation()} />
</div>
```

每个事件都会经过三个阶段：

1. 它向下传播，调用所有 `onClickCapture` 处理函数。
2. 它运行被点击元素的 `onClick` 处理函数。
3. 它向上传播，调用所有 `onClick` 处理函数。

捕获事件适用于路由或分析等代码，但你大概不会在应用代码中使用它们。

</DeepDive>

### 将处理函数作为传播的替代方案 {/*passing-handlers-as-alternative-to-propagation*/}

注意这个点击处理函数是如何先执行一行代码，然后再调用父组件传入的 `onClick` prop 的：

```js {4,5}
function Button({ onClick, children }) {
  return (
    <button onClick={e => {
      e.stopPropagation();
      onClick();
    }}>
      {children}
    </button>
  );
}
```

你也可以在调用父级 `onClick` 事件处理函数之前，向这个处理函数添加更多代码。这个模式提供了传播的一种*替代方案*。它让子组件处理事件，同时也让父组件指定一些额外行为。与传播不同，它不是自动发生的。但这种模式的好处是，你可以清楚地跟踪某个事件触发后执行的整条代码链。

如果你依赖传播，而很难追踪哪些处理函数会执行以及原因，那么可以试试这种方法。

### 阻止默认行为 {/*preventing-default-behavior*/}

某些浏览器事件会有默认行为。例如，`<form>` 的提交事件在其中的按钮被点击时触发，默认会重新加载整个页面：

<Sandpack>

```js
export default function Signup() {
  return (
    <form onSubmit={() => alert('Submitting!')}>
      <input />
      <button>发送</button>
    </form>
  );
}
```

```css
button { margin-left: 5px; }
```

</Sandpack>

你可以在事件对象上调用 `e.preventDefault()` 来阻止这种行为发生：

<Sandpack>

```js
export default function Signup() {
  return (
    <form onSubmit={e => {
      e.preventDefault();
      alert('Submitting!');
    }}>
      <input />
      <button>发送</button>
    </form>
  );
}
```

```css
button { margin-left: 5px; }
```

</Sandpack>

不要混淆 `e.stopPropagation()` 和 `e.preventDefault()`。它们都很有用，但彼此无关：

* [`e.stopPropagation()`](https://developer.mozilla.org/docs/Web/API/Event/stopPropagation) 会阻止附加在上层标签上的事件处理函数被触发。
* [`e.preventDefault()` ](https://developer.mozilla.org/docs/Web/API/Event/preventDefault) 会阻止少数具有默认行为的事件执行浏览器默认行为。

## 事件处理函数可以有副作用吗？ {/*can-event-handlers-have-side-effects*/}

当然可以！事件处理函数正是执行副作用的最佳位置。

与渲染函数不同，事件处理函数不需要是[纯函数](/learn/keeping-components-pure)，所以这里非常适合去*改变*某些东西——例如，根据输入改变输入框的值，或者根据按钮点击改变列表。不过，要改变某些信息，你首先需要某种方式来存储它。在 React 中，这是通过使用[state，组件的记忆。](/learn/state-a-components-memory)来实现的。你会在下一页学到它的全部内容。

<Recap>

* 你可以通过将一个函数作为 prop 传递给像 `<button>` 这样的元素来处理事件。
* 事件处理函数必须被传递，**不能被调用！** `onClick={handleClick}`，而不是 `onClick={handleClick()}`。
* 你可以单独定义事件处理函数，也可以内联定义。
* 事件处理函数定义在组件内部，因此它们可以访问 props。
* 你可以在父组件中声明一个事件处理函数，并把它作为 prop 传给子组件。
* 你可以使用应用特定的名称来定义自己的事件处理函数 prop。
* 事件会向上传播。调用第一个参数上的 `e.stopPropagation()` 可以阻止这一点。
* 事件可能具有不希望的浏览器默认行为。调用 `e.preventDefault()` 可以阻止这一点。
* 直接从子组件处理函数中调用事件处理函数 prop，是传播之外的一个不错替代方案。

</Recap>



<Challenges>

#### 修复一个事件处理函数 {/*fix-an-event-handler*/}

点击这个按钮本应在白色和黑色之间切换页面背景色。然而，点击它时并没有任何反应。修复这个问题。（不用担心 `handleClick` 内部的逻辑——那部分是正确的。）

<Sandpack>

```js {expectedErrors: {'react-compiler': [5, 7]}}
export default function LightSwitch() {
  function handleClick() {
    let bodyStyle = document.body.style;
    if (bodyStyle.backgroundColor === 'black') {
      bodyStyle.backgroundColor = 'white';
    } else {
      bodyStyle.backgroundColor = 'black';
    }
  }

  return (
    <button onClick={handleClick()}>
      Toggle the lights
    </button>
  );
}
```

</Sandpack>

<Solution>

问题在于 `<button onClick={handleClick()}>` 在渲染时 _调用_ 了 `handleClick` 函数，而不是将其 _传递_ 进去。去掉 `()` 调用，使其变成 `<button onClick={handleClick}>` 就能修复这个问题：

<Sandpack>

```js
export default function LightSwitch() {
  function handleClick() {
    let bodyStyle = document.body.style;
    if (bodyStyle.backgroundColor === 'black') {
      bodyStyle.backgroundColor = 'white';
    } else {
      bodyStyle.backgroundColor = 'black';
    }
  }

  return (
    <button onClick={handleClick}>
      Toggle the lights
    </button>
  );
}
```

</Sandpack>

或者，你也可以把调用包装到另一个函数里，比如 `<button onClick={() => handleClick()}>`：

<Sandpack>

```js
export default function LightSwitch() {
  function handleClick() {
    let bodyStyle = document.body.style;
    if (bodyStyle.backgroundColor === 'black') {
      bodyStyle.backgroundColor = 'white';
    } else {
      bodyStyle.backgroundColor = 'black';
    }
  }

  return (
    <button onClick={() => handleClick()}>
      Toggle the lights
    </button>
  );
}
```

</Sandpack>

</Solution>

#### 连接这些事件 {/*wire-up-the-events*/}

这个 `ColorSwitch` 组件渲染了一个按钮。它本应改变页面颜色。请将它连接到它从父组件接收到的 `onChangeColor` 事件处理函数 prop，这样点击按钮就会改变颜色。

在你完成之后，会发现点击按钮也会增加页面点击计数器。写父组件的同事坚持认为 `onChangeColor` 并不会增加任何计数器。那还可能是什么原因呢？把它修复掉，使点击按钮*只*改变颜色，而不会增加计数器。

<Sandpack>

```js src/ColorSwitch.js active
export default function ColorSwitch({
  onChangeColor
}) {
  return (
    <button>
      Change color
    </button>
  );
}
```

```js src/App.js hidden
import { useState } from 'react';
import ColorSwitch from './ColorSwitch.js';

export default function App() {
  const [clicks, setClicks] = useState(0);

  function handleClickOutside() {
    setClicks(c => c + 1);
  }

  function getRandomLightColor() {
    let r = 150 + Math.round(100 * Math.random());
    let g = 150 + Math.round(100 * Math.random());
    let b = 150 + Math.round(100 * Math.random());
    return `rgb(${r}, ${g}, ${b})`;
  }

  function handleChangeColor() {
    let bodyStyle = document.body.style;
    bodyStyle.backgroundColor = getRandomLightColor();
  }

  return (
    <div style={{ width: '100%', height: '100%' }} onClick={handleClickOutside}>
      <ColorSwitch onChangeColor={handleChangeColor} />
      <br />
      <br />
      <h2>Clicks on the page: {clicks}</h2>
    </div>
  );
}
```

</Sandpack>

<Solution>

首先，你需要添加事件处理函数，例如 `<button onClick={onChangeColor}>`。

不过，这会引入计数器递增的问题。如果 `onChangeColor` 并不会执行这个操作，正如你的同事所说，那么问题就出在这个事件向上传播了，并且上层的某个处理函数做了这件事。要解决这个问题，你需要停止传播。但别忘了，你仍然应该调用 `onChangeColor`。

<Sandpack>

```js src/ColorSwitch.js active
export default function ColorSwitch({
  onChangeColor
}) {
  return (
    <button onClick={e => {
      e.stopPropagation();
      onChangeColor();
    }}>
      Change color
    </button>
  );
}
```

```js src/App.js hidden
import { useState } from 'react';
import ColorSwitch from './ColorSwitch.js';

export default function App() {
  const [clicks, setClicks] = useState(0);

  function handleClickOutside() {
    setClicks(c => c + 1);
  }

  function getRandomLightColor() {
    let r = 150 + Math.round(100 * Math.random());
    let g = 150 + Math.round(100 * Math.random());
    let b = 150 + Math.round(100 * Math.random());
    return `rgb(${r}, ${g}, ${b})`;
  }

  function handleChangeColor() {
    let bodyStyle = document.body.style;
    bodyStyle.backgroundColor = getRandomLightColor();
  }

  return (
    <div style={{ width: '100%', height: '100%' }} onClick={handleClickOutside}>
      <ColorSwitch onChangeColor={handleChangeColor} />
      <br />
      <br />
      <h2>Clicks on the page: {clicks}</h2>
    </div>
  );
}
```

</Sandpack>

</Solution>

</Challenges>
