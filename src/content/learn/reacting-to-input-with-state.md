---
title: 使用状态响应输入
---

<Intro>

React 提供了一种声明式的方式来操作 UI。你不是直接操作 UI 的某一部分，而是描述组件可能处于的不同状态，并根据用户输入在这些状态之间切换。这和设计师思考 UI 的方式很相似。

</Intro>

<YouWillLearn>

* 声明式 UI 编程与命令式 UI 编程有何不同
* 如何列举组件可能处于的不同视觉状态
* 如何从代码中触发不同视觉状态之间的变化

</YouWillLearn>

## 声明式 UI 与命令式 UI 的比较 {/*how-declarative-ui-compares-to-imperative*/}

当你设计 UI 交互时，你大概会想到 UI 如何随着用户操作而*变化*。考虑一个让用户提交答案的表单：

* 当你在表单中输入内容时，“提交”按钮**变为可用。**
* 当你按下“提交”时，表单和按钮都会**变为不可用，**并且会**出现**一个加载动画。
* 如果网络请求成功，表单**会被隐藏，**而“谢谢你”消息**会出现。**
* 如果网络请求失败，**会出现**一条错误消息，并且表单再次**变为可用。**

在**命令式编程**中，上面的描述会直接对应你如何实现交互。你必须根据刚刚发生的事情，编写精确的指令来操作 UI。换个方式来理解：想象你坐在车里，告诉旁边的人每一个转弯该往哪里走。

<Illustration src="/images/docs/illustrations/i_imperative-ui-programming.png"  alt="车里坐着一个看起来很焦虑的人代表 JavaScript，乘客指挥司机按复杂的转弯导航顺序行驶。" />

他们不知道你想去哪里，只是照着你的命令做。（如果你给的路线错了，最后就会到错的地方！）之所以叫*命令式*，是因为你必须“命令”每个元素，从加载动画到按钮，告诉计算机*如何*更新 UI。

在这个命令式 UI 编程示例中，表单是在*没有* React 的情况下构建的。它只使用浏览器的 [DOM](https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model)：

<Sandpack>

```js src/index.js active
async function handleFormSubmit(e) {
  e.preventDefault();
  disable(textarea);
  disable(button);
  show(loadingMessage);
  hide(errorMessage);
  try {
    await submitForm(textarea.value);
    show(successMessage);
    hide(form);
  } catch (err) {
    show(errorMessage);
    errorMessage.textContent = err.message;
  } finally {
    hide(loadingMessage);
    enable(textarea);
    enable(button);
  }
}

function handleTextareaChange() {
  if (textarea.value.length === 0) {
    disable(button);
  } else {
    enable(button);
  }
}

function hide(el) {
  el.style.display = 'none';
}

function show(el) {
  el.style.display = '';
}

function enable(el) {
  el.disabled = false;
}

function disable(el) {
  el.disabled = true;
}

function submitForm(answer) {
  // 假装它在请求网络。
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (answer.toLowerCase() === 'istanbul') {
        resolve();
      } else {
        reject(new Error('猜得不错，但答案错了。再试一次！'));
      }
    }, 1500);
  });
}

let form = document.getElementById('form');
let textarea = document.getElementById('textarea');
let button = document.getElementById('button');
let loadingMessage = document.getElementById('loading');
let errorMessage = document.getElementById('error');
let successMessage = document.getElementById('success');
form.onsubmit = handleFormSubmit;
textarea.oninput = handleTextareaChange;
```

```js sandbox.config.json hidden
{
  "hardReloadOnChange": true
}
```

```html public/index.html
<form id="form">
  <h2>城市测验</h2>
  <p>
    哪座城市位于两个大陆上？
  </p>
  <textarea id="textarea"></textarea>
  <br />
  <button id="button" disabled>提交</button>
  <p id="loading" style="display: none">加载中...</p>
  <p id="error" style="display: none; color: red;"></p>
</form>
<h1 id="success" style="display: none">答对了！</h1>

<style>
* { box-sizing: border-box; }
body { font-family: sans-serif; margin: 20px; padding: 0; }
</style>
```

</Sandpack>

对于独立的例子来说，命令式地操作 UI 还算足够好用，但在更复杂的系统中，管理起来会变得指数级更困难。想象一下要更新一个充满类似表单的页面。添加一个新的 UI 元素或新的交互，都需要仔细检查所有现有代码，以确保你没有引入 bug（例如，忘记显示或隐藏某些内容）。

React 就是为了解决这个问题而构建的。

在 React 中，你不会直接操作 UI——也就是说，你不会直接启用、禁用、显示或隐藏组件。相反，你会**声明你想要显示什么，**然后由 React 来决定如何更新 UI。可以把它想象成坐进出租车，告诉司机你想去哪里，而不是告诉他每一个转弯该怎么走。把你送到目的地是司机的工作，而且他们甚至可能知道一些你没想到的捷径！

<Illustration src="/images/docs/illustrations/i_declarative-ui-programming.png" alt="由 React 驾驶的车里，乘客要求被带到地图上的某个具体地点。React 负责想办法实现这一点。" />

## 声明式地思考 UI {/*thinking-about-ui-declaratively*/}

你已经看过上面如何用命令式方式实现一个表单。为了更好地理解如何在 React 中思考，下面你将一步步在 React 中重新实现这个 UI：

1. **识别**组件的不同视觉状态
2. **确定**是什么触发了这些状态变化
3. 使用 `useState` **在内存中表示**状态
4. **移除**任何非必要的状态变量
5. **连接**事件处理函数来设置状态

### 第 1 步：识别组件的不同视觉状态 {/*step-1-identify-your-components-different-visual-states*/}

在计算机科学中，你可能会听到“[状态机](https://en.wikipedia.org/wiki/Finite-state_machine)”处于若干“状态”之一。如果你和设计师合作过，你可能见过不同“视觉状态”的设计稿。React 处于设计和计算机科学的交汇点，因此这两种想法都是灵感来源。

首先，你需要把用户可能看到的 UI 的所有不同“状态”都想象出来：

* **空白**：表单有一个不可用的“提交”按钮。
* **输入中**：表单有一个可用的“提交”按钮。
* **提交中**：表单完全不可用。显示加载动画。
* **成功**：显示“谢谢你”消息，而不是表单。
* **错误**：与输入中状态相同，但额外显示一条错误消息。

就像设计师一样，在添加逻辑之前，你最好先为不同状态“画出原型”或创建“模拟图”。例如，下面只是表单视觉部分的一个模拟图。这个模拟图由一个名为 `status` 的 prop 控制，默认值为 `'empty'`：

<Sandpack>

```js
export default function Form({
  status = 'empty'
}) {
  if (status === 'success') {
    return <h1>答对了！</h1>
  }
  return (
    <>
      <h2>城市测验</h2>
      <p>
        在哪个城市有一块把空气变成可饮用水的广告牌？
      </p>
      <form>
        <textarea />
        <br />
        <button>
          提交
        </button>
      </form>
    </>
  )
}
```

</Sandpack>

这个 prop 叫什么都可以，命名并不重要。试着把 `status = 'empty'` 改成 `status = 'success'`，看看成功消息出现。模拟图可以让你在编写任何逻辑之前快速迭代 UI。下面是同一个组件更完整的原型，仍然由 `status` prop“控制”：

<Sandpack>

```js
export default function Form({
  // 试试 'submitting'、'error'、'success'：
  status = 'empty'
}) {
  if (status === 'success') {
    return <h1>答对了！</h1>
  }
  return (
    <>
      <h2>城市测验</h2>
      <p>
        在哪个城市有一块把空气变成可饮用水的广告牌？
      </p>
      <form>
        <textarea disabled={
          status === 'submitting'
        } />
        <br />
        <button disabled={
          status === 'empty' ||
          status === 'submitting'
        }>
          提交
        </button>
        {status === 'error' &&
          <p className="Error">
            猜得不错，但答案错了。再试一次！
          </p>
        }
      </form>
      </>
  );
}
```

```css
.Error { color: red; }
```

</Sandpack>

<DeepDive>

#### 同时展示多个视觉状态 {/*displaying-many-visual-states-at-once*/}

如果一个组件有很多视觉状态，把它们都显示在同一页上会很方便：

<Sandpack>

```js src/App.js active
import Form from './Form.js';

let statuses = [
  'empty',
  'typing',
  'submitting',
  'success',
  'error',
];

export default function App() {
  return (
    <>
      {statuses.map(status => (
        <section key={status}>
          <h4>表单（{status}）：</h4>
          <Form status={status} />
        </section>
      ))}
    </>
  );
}
```

```js src/Form.js
export default function Form({ status }) {
  if (status === 'success') {
    return <h1>答对了！</h1>
  }
  return (
    <form>
      <textarea disabled={
        status === 'submitting'
      } />
      <br />
      <button disabled={
        status === 'empty' ||
        status === 'submitting'
      }>
        提交
      </button>
      {status === 'error' &&
        <p className="Error">
          猜得不错，但答案错了。再试一次！
        </p>
      }
    </form>
  );
}
```

```css
section { border-bottom: 1px solid #aaa; padding: 20px; }
h4 { color: #222; }
body { margin: 0; }
.Error { color: red; }
```

</Sandpack>

这样的页面常被称为“活的样式指南”或“Storybook”。

</DeepDive>

### 第 2 步：确定是什么触发这些状态变化 {/*step-2-determine-what-triggers-those-state-changes*/}

你可以响应两类输入来触发状态更新：

* **人为输入**，例如点击按钮、在字段中输入、点击链接导航。
* **计算机输入**，例如网络响应到达、超时完成、图片加载完成。

<IllustrationBlock>
  <Illustration caption="人为输入" alt="一根手指。" src="/images/docs/illustrations/i_inputs1.png" />
  <Illustration caption="计算机输入" alt="一串零和一。" src="/images/docs/illustrations/i_inputs2.png" />
</IllustrationBlock>

在这两种情况下，**你都必须设置[状态变量](/learn/state-a-components-memory#anatomy-of-usestate)来更新 UI。** 对于你正在开发的表单，你需要根据几种不同输入来改变状态：

* **更改文本输入**（人为）应该会把它从*空白*状态切换到*输入中*状态，或者相反，取决于文本框是否为空。
* **点击提交按钮**（人为）应该会把它切换到*提交中*状态。
* **成功的网络响应**（计算机）应该会切换到*成功*状态。
* **失败的网络响应**（计算机）应该会切换到带有匹配错误消息的*错误*状态。

<Note>

注意，人为输入通常需要[事件处理函数](/learn/responding-to-events)！

</Note>

为了帮助你可视化这个流程，试着把每个状态画成一个带标签的圆圈，再把两个状态之间的变化画成箭头。你可以这样草绘出很多流程，并在实现之前就找出 bug。

<DiagramGroup>

<Diagram name="responding_to_input_flow" height={350} width={688} alt="从左到右移动的流程图，有 5 个节点。第一个标注为 'empty' 的节点有一条标注 'start typing' 的边连接到标注为 'typing' 的节点。该节点有一条标注 'press submit' 的边连接到标注为 'submitting' 的节点，而后者有两条边。左边标注 'network error'，连接到标注为 'error' 的节点。右边标注 'network success'，连接到标注为 'success' 的节点。">

表单状态

</Diagram>

</DiagramGroup>

### 第 3 步：使用 `useState` 在内存中表示状态 {/*step-3-represent-the-state-in-memory-with-usestate*/}

接下来，你需要使用 [`useState`.](/reference/react/useState) 在内存中表示组件的视觉状态。简洁是关键：每一份状态都是一个“活动部件”，而**你希望这样的“活动部件”越少越好。** 复杂度越高，bug 就越多！

先从*绝对必须存在*的状态开始。例如，你需要存储输入框的 `answer`，以及（如果存在）存储上一次错误的 `error`：

```js
const [answer, setAnswer] = useState('');
const [error, setError] = useState(null);
```

然后，你还需要一个状态变量来表示你想显示哪一种视觉状态。通常，在内存中表示这件事的方法不止一种，所以你需要进行尝试。

如果你一时想不出最好的方式，可以先添加足够多的状态，确保你*完全确定*覆盖了所有可能的视觉状态：

```js
const [isEmpty, setIsEmpty] = useState(true);
const [isTyping, setIsTyping] = useState(false);
const [isSubmitting, setIsSubmitting] = useState(false);
const [isSuccess, setIsSuccess] = useState(false);
const [isError, setIsError] = useState(false);
```

你最初的想法很可能不是最好的，但没关系——重构状态本身就是这个过程的一部分！

### 第 4 步：移除任何非必要的状态变量 {/*step-4-remove-any-non-essential-state-variables*/}

你希望避免状态内容中的重复，这样你只追踪必要的信息。花一点时间重构状态结构，会让你的组件更容易理解、减少重复，并避免不想要的含义。你的目标是**防止出现这种情况：内存中的状态并不代表你希望用户看到的任何有效 UI。**（例如，你绝不希望同时显示错误消息和禁用输入框，否则用户就无法修正错误了！）

你可以针对状态变量问自己几个问题：

* **这个状态会造成悖论吗？** 例如，`isTyping` 和 `isSubmitting` 不可能同时为 `true`。悖论通常意味着状态约束得不够。两个布尔值有四种组合，但只有三种对应有效状态。要移除这个“不可能”状态，你可以把它们合并成一个必须取三个值之一的 `status`：`'typing'`、`'submitting'` 或 `'success'`。
* **同样的信息是否已经可以从另一个状态变量中得到？** 另一个悖论：`isEmpty` 和 `isTyping` 不可能同时为 `true`。把它们拆成独立状态变量，会有不同步并引发 bug 的风险。幸运的是，你可以移除 `isEmpty`，改为检查 `answer.length === 0`。
* **是否可以通过另一个状态变量的反面得到同样的信息？** 不需要 `isError`，因为你可以改为检查 `error !== null`。

经过这一步清理后，你只剩下 3 个（从 7 个减少到了 3 个！）*必要*状态变量：

```js
const [answer, setAnswer] = useState('');
const [error, setError] = useState(null);
const [status, setStatus] = useState('typing'); // 'typing'、'submitting' 或 'success'
```

你知道它们是必要的，因为如果移除其中任何一个，功能就会被破坏。

<DeepDive>

#### 用 reducer 消除“不可能”状态 {/*eliminating-impossible-states-with-a-reducer*/}

这三个变量已经足够表示这个表单的状态了。不过，仍然存在一些中间状态并不完全合理。例如，当 `status` 是 `'success'` 时，非空的 `error` 就没有意义。为了更精确地建模状态，你可以[把它提取到 reducer 中。](/learn/extracting-state-logic-into-a-reducer) Reducer 可以让你把多个状态变量统一到一个对象里，并将所有相关逻辑集中起来！

</DeepDive>

### 第 5 步：连接事件处理函数来设置状态 {/*step-5-connect-the-event-handlers-to-set-state*/}

最后，创建更新状态的事件处理函数。下面是最终版本，已经连接好了所有事件处理函数：

<Sandpack>

```js
import { useState } from 'react';

export default function Form() {
  const [answer, setAnswer] = useState('');
  const [error, setError] = useState(null);
  const [status, setStatus] = useState('typing');

  if (status === 'success') {
    return <h1>答对了！</h1>
  }

  async function handleSubmit(e) {
    e.preventDefault();
    setStatus('submitting');
    try {
      await submitForm(answer);
      setStatus('success');
    } catch (err) {
      setStatus('typing');
      setError(err);
    }
  }

  function handleTextareaChange(e) {
    setAnswer(e.target.value);
  }

  return (
    <>
      <h2>城市测验</h2>
      <p>
        在哪个城市有一块把空气变成可饮用水的广告牌？
      </p>
      <form onSubmit={handleSubmit}>
        <textarea
          value={answer}
          onChange={handleTextareaChange}
          disabled={status === 'submitting'}
        />
        <br />
        <button disabled={
          answer.length === 0 ||
          status === 'submitting'
        }>
          提交
        </button>
        {error !== null &&
          <p className="Error">
            {error.message}
          </p>
        }
      </form>
    </>
  );
}

function submitForm(answer) {
  // 假装它在请求网络。
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      let shouldError = answer.toLowerCase() !== 'lima'
      if (shouldError) {
        reject(new Error('猜得不错，但答案错了。再试一次！'));
      } else {
        resolve();
      }
    }, 1500);
  });
}
```

```css
.Error { color: red; }
```

</Sandpack>

虽然这段代码比原来的命令式示例更长，但它脆弱性要小得多。把所有交互都表达成状态变化，可以让你以后在不破坏现有视觉状态的情况下引入新的视觉状态。它还允许你在不改变交互逻辑本身的前提下，改变每个状态下应该显示什么。

<Recap>

* 声明式编程意味着描述每个视觉状态下的 UI，而不是微观管理 UI（命令式）。
* 开发组件时：
  1. 识别它所有的视觉状态。
  2. 确定触发状态变化的人为和计算机输入。
  3. 使用 `useState` 建模状态。
  4. 移除非必要状态以避免 bug 和悖论。
  5. 连接事件处理函数来设置状态。

</Recap>



<Challenges>

#### 添加和移除 CSS 类 {/*add-and-remove-a-css-class*/}

使得点击图片时，会从外层 `<div>` 移除 `background--active` 这个 CSS 类，但会给 `<img>` 添加 `picture--active` 类。再次点击背景时，应恢复原始的 CSS 类。

从视觉上看，你应该预期点击图片会移除紫色背景，并高亮图片边框。点击图片外部会高亮背景，但移除图片边框的高亮。

<Sandpack>

```js
export default function Picture() {
  return (
    <div className="background background--active">
      <img
        className="picture"
        alt="印度尼西亚 Kampung Pelangi 的彩虹房屋"
        src="https://react.dev/images/docs/scientists/5qwVYb1.jpeg"
      />
    </div>
  );
}
```

```css
body { margin: 0; padding: 0; height: 250px; }

.background {
  width: 100vw;
  height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  background: #eee;
}

.background--active {
  background: #a6b5ff;
}

.picture {
  width: 200px;
  height: 200px;
  border-radius: 10px;
  border: 5px solid transparent;
}

.picture--active {
  border: 5px solid #a6b5ff;
}
```

</Sandpack>

<Solution>

这个组件有两个视觉状态：图片处于激活状态，以及图片处于未激活状态：

* 当图片处于激活状态时，CSS 类是 `background` 和 `picture picture--active`。
* 当图片处于未激活状态时，CSS 类是 `background background--active` 和 `picture`。

一个布尔状态变量就足以记住图片是否处于激活状态。原始任务是移除或添加 CSS 类。不过在 React 中，你需要*描述*你想看到什么，而不是*操作* UI 元素。所以你需要根据当前状态计算两个 CSS 类。你还需要[停止冒泡](/learn/responding-to-events#stopping-propagation)，这样点击图片就不会被记录为点击背景。

通过先点击图片，再点击图片外部来验证这个版本是否工作：

<Sandpack>

```js
import { useState } from 'react';

export default function Picture() {
  const [isActive, setIsActive] = useState(false);

  let backgroundClassName = 'background';
  let pictureClassName = 'picture';
  if (isActive) {
    pictureClassName += ' picture--active';
  } else {
    backgroundClassName += ' background--active';
  }

  return (
    <div
      className={backgroundClassName}
      onClick={() => setIsActive(false)}
    >
      <img
        onClick={e => {
          e.stopPropagation();
          setIsActive(true);
        }}
        className={pictureClassName}
        alt="印度尼西亚 Kampung Pelangi 的彩虹房屋"
        src="https://react.dev/images/docs/scientists/5qwVYb1.jpeg"
      />
    </div>
  );
}
```

```css
body { margin: 0; padding: 0; height: 250px; }

.background {
  width: 100vw;
  height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  background: #eee;
}

.background--active {
  background: #a6b5ff;
}

.picture {
  width: 200px;
  height: 200px;
  border-radius: 10px;
  border: 5px solid transparent;
}

.picture--active {
  border: 5px solid #a6b5ff;
}
```

</Sandpack>

或者，你也可以返回两段不同的 JSX：

<Sandpack>

```js
import { useState } from 'react';

export default function Picture() {
  const [isActive, setIsActive] = useState(false);
  if (isActive) {
    return (
      <div
        className="background"
        onClick={() => setIsActive(false)}
      >
        <img
          className="picture picture--active"
          alt="印度尼西亚 Kampung Pelangi 的彩虹房屋"
          src="https://react.dev/images/docs/scientists/5qwVYb1.jpeg"
          onClick={e => e.stopPropagation()}
        />
      </div>
    );
  }
  return (
    <div className="background background--active">
      <img
        className="picture"
        alt="印度尼西亚 Kampung Pelangi 的彩虹房屋"
        src="https://react.dev/images/docs/scientists/5qwVYb1.jpeg"
        onClick={() => setIsActive(true)}
      />
    </div>
  );
}
```

```css
body { margin: 0; padding: 0; height: 250px; }

.background {
  width: 100vw;
  height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  background: #eee;
}

.background--active {
  background: #a6b5ff;
}

.picture {
  width: 200px;
  height: 200px;
  border-radius: 10px;
  border: 5px solid transparent;
}

.picture--active {
  border: 5px solid #a6b5ff;
}
```

</Sandpack>

请记住，如果两段不同的 JSX 描述的是同一棵树，它们的嵌套结构（第一个 `<div>` → 第一个 `<img>`）必须对得上。否则，切换 `isActive` 会重新创建下面的整个树，并[重置其状态。](/learn/preserving-and-resetting-state) 这就是为什么，如果两种情况下返回的 JSX 树类似，最好把它们写成同一段 JSX。

</Solution>

#### 个人资料编辑器 {/*profile-editor*/}

这里有一个使用原生 JavaScript 和 DOM 实现的小表单。先试着操作它，理解它的行为：

<Sandpack>

```js src/index.js active
function handleFormSubmit(e) {
  e.preventDefault();
  if (editButton.textContent === 'Edit Profile') {
    editButton.textContent = 'Save Profile';
    hide(firstNameText);
    hide(lastNameText);
    show(firstNameInput);
    show(lastNameInput);
  } else {
    editButton.textContent = 'Edit Profile';
    hide(firstNameInput);
    hide(lastNameInput);
    show(firstNameText);
    show(lastNameText);
  }
}

function handleFirstNameChange() {
  firstNameText.textContent = firstNameInput.value;
  helloText.textContent = (
    'Hello ' +
    firstNameInput.value + ' ' +
    lastNameInput.value + '!'
  );
}

function handleLastNameChange() {
  lastNameText.textContent = lastNameInput.value;
  helloText.textContent = (
    'Hello ' +
    firstNameInput.value + ' ' +
    lastNameInput.value + '!'
  );
}

function hide(el) {
  el.style.display = 'none';
}

function show(el) {
  el.style.display = '';
}

let form = document.getElementById('form');
let editButton = document.getElementById('editButton');
let firstNameInput = document.getElementById('firstNameInput');
let firstNameText = document.getElementById('firstNameText');
let lastNameInput = document.getElementById('lastNameInput');
let lastNameText = document.getElementById('lastNameText');
let helloText = document.getElementById('helloText');
form.onsubmit = handleFormSubmit;
firstNameInput.oninput = handleFirstNameChange;
lastNameInput.oninput = handleLastNameChange;
```

```js sandbox.config.json hidden
{
  "hardReloadOnChange": true
}
```

```html public/index.html
<form id="form">
  <label>
    名字：
    <b id="firstNameText">Jane</b>
    <input
      id="firstNameInput"
      value="Jane"
      style="display: none">
  </label>
  <label>
    姓氏：
    <b id="lastNameText">Jacobs</b>
    <input
      id="lastNameInput"
      value="Jacobs"
      style="display: none">
  </label>
  <button type="submit" id="editButton">编辑资料</button>
  <p><i id="helloText">你好，Jane Jacobs！</i></p>
</form>

<style>
* { box-sizing: border-box; }
body { font-family: sans-serif; margin: 20px; padding: 0; }
label { display: block; margin-bottom: 20px; }
</style>
```

</Sandpack>

这个表单在两种模式之间切换：在编辑模式下，你会看到输入框；在查看模式下，你只会看到结果。按钮标签会根据模式在“编辑”和“保存”之间变化。当你修改输入框时，底部的欢迎信息会实时更新。

你的任务是在下面的沙盒中用 React 重新实现它。为了方便起见，标记已经被转换为 JSX，但你需要让它像原版一样显示和隐藏输入框。

也要确保底部文本会更新！

<Sandpack>

```js
export default function EditProfile() {
  return (
    <form>
      <label>
        名字：{' '}
        <b>Jane</b>
        <input />
      </label>
      <label>
        姓氏：{' '}
        <b>Jacobs</b>
        <input />
      </label>
      <button type="submit">
        编辑资料
      </button>
      <p><i>你好，Jane Jacobs！</i></p>
    </form>
  );
}
```

```css
label { display: block; margin-bottom: 20px; }
```

</Sandpack>

<Solution>

你需要两个状态变量来保存输入值：`firstName` 和 `lastName`。你还需要一个 `isEditing` 状态变量，用来表示是否显示输入框。你_不_需要 `fullName` 变量，因为完整姓名总是可以由 `firstName` 和 `lastName` 计算出来。

最后，你应该根据 `isEditing` 使用[条件渲染](/learn/conditional-rendering)来显示或隐藏输入框。

<Sandpack>

```js
import { useState } from 'react';

export default function EditProfile() {
  const [isEditing, setIsEditing] = useState(false);
  const [firstName, setFirstName] = useState('Jane');
  const [lastName, setLastName] = useState('Jacobs');

  return (
    <form onSubmit={e => {
      e.preventDefault();
      setIsEditing(!isEditing);
    }}>
      <label>
        名字：{' '}
        {isEditing ? (
          <input
            value={firstName}
            onChange={e => {
              setFirstName(e.target.value)
            }}
          />
        ) : (
          <b>{firstName}</b>
        )}
      </label>
      <label>
        姓氏：{' '}
        {isEditing ? (
          <input
            value={lastName}
            onChange={e => {
              setLastName(e.target.value)
            }}
          />
        ) : (
          <b>{lastName}</b>
        )}
      </label>
      <button type="submit">
        {isEditing ? '保存' : '编辑'} 资料
      </button>
      <p><i>你好，{firstName} {lastName}！</i></p>
    </form>
  );
}
```

```css
label { display: block; margin-bottom: 20px; }
```

</Sandpack>

把这个解决方案和原来的命令式代码比较一下。它们有什么不同？

</Solution>

#### 在不使用 React 的情况下重构命令式方案 {/*refactor-the-imperative-solution-without-react*/}

下面是上一个挑战中的原始沙盒，它是用命令式方式写的，没有使用 React：

<Sandpack>

```js src/index.js active
function handleFormSubmit(e) {
  e.preventDefault();
  if (editButton.textContent === 'Edit Profile') {
    editButton.textContent = 'Save Profile';
    hide(firstNameText);
    hide(lastNameText);
    show(firstNameInput);
    show(lastNameInput);
  } else {
    editButton.textContent = 'Edit Profile';
    hide(firstNameInput);
    hide(lastNameInput);
    show(firstNameText);
    show(lastNameText);
  }
}

function handleFirstNameChange() {
  firstNameText.textContent = firstNameInput.value;
  helloText.textContent = (
    'Hello ' +
    firstNameInput.value + ' ' +
    lastNameInput.value + '!'
  );
}

function handleLastNameChange() {
  lastNameText.textContent = lastNameInput.value;
  helloText.textContent = (
    'Hello ' +
    firstNameInput.value + ' ' +
    lastNameInput.value + '!'
  );
}

function hide(el) {
  el.style.display = 'none';
}

function show(el) {
  el.style.display = '';
}

let form = document.getElementById('form');
let editButton = document.getElementById('editButton');
let firstNameInput = document.getElementById('firstNameInput');
let firstNameText = document.getElementById('firstNameText');
let lastNameInput = document.getElementById('lastNameInput');
let lastNameText = document.getElementById('lastNameText');
let helloText = document.getElementById('helloText');
form.onsubmit = handleFormSubmit;
firstNameInput.oninput = handleFirstNameChange;
lastNameInput.oninput = handleLastNameChange;
```

```js sandbox.config.json hidden
{
  "hardReloadOnChange": true
}
```

```html public/index.html
<form id="form">
  <label>
    名字：
    <b id="firstNameText">Jane</b>
    <input
      id="firstNameInput"
      value="Jane"
      style="display: none">
  </label>
  <label>
    姓氏：
    <b id="lastNameText">Jacobs</b>
    <input
      id="lastNameInput"
      value="Jacobs"
      style="display: none">
  </label>
  <button type="submit" id="editButton">编辑资料</button>
  <p><i id="helloText">你好，Jane Jacobs！</i></p>
</form>

<style>
* { box-sizing: border-box; }
body { font-family: sans-serif; margin: 20px; padding: 0; }
label { display: block; margin-bottom: 20px; }
</style>
```

</Sandpack>

假设 React 并不存在。你能以一种让逻辑更不脆弱、并且更接近 React 版本的方式重构这段代码吗？如果状态是显式的，就像在 React 里那样，它会是什么样子？

如果你在想从哪里开始有困难，下面的骨架已经有了大部分结构。如果你从这里开始，就把缺失的逻辑补到 `updateDOM` 函数里。（需要时参考原始代码。）

<Sandpack>

```js src/index.js active
let firstName = 'Jane';
let lastName = 'Jacobs';
let isEditing = false;

function handleFormSubmit(e) {
  e.preventDefault();
  setIsEditing(!isEditing);
}

function handleFirstNameChange(e) {
  setFirstName(e.target.value);
}

function handleLastNameChange(e) {
  setLastName(e.target.value);
}

function setFirstName(value) {
  firstName = value;
  updateDOM();
}

function setLastName(value) {
  lastName = value;
  updateDOM();
}

function setIsEditing(value) {
  isEditing = value;
  updateDOM();
}

function updateDOM() {
  if (isEditing) {
    editButton.textContent = '保存资料';
    // TODO: 显示输入框，隐藏内容
  } else {
    editButton.textContent = '编辑资料';
    // TODO: 隐藏输入框，显示内容
  }
  // TODO: 更新文本标签
}

function hide(el) {
  el.style.display = 'none';
}

function show(el) {
  el.style.display = '';
}

let form = document.getElementById('form');
let editButton = document.getElementById('editButton');
let firstNameInput = document.getElementById('firstNameInput');
let firstNameText = document.getElementById('firstNameText');
let lastNameInput = document.getElementById('lastNameInput');
let lastNameText = document.getElementById('lastNameText');
let helloText = document.getElementById('helloText');
form.onsubmit = handleFormSubmit;
firstNameInput.oninput = handleFirstNameChange;
lastNameInput.oninput = handleLastNameChange;
```

```js sandbox.config.json hidden
{
  "hardReloadOnChange": true
}
```

```html public/index.html
<form id="form">
  <label>
    名字：
    <b id="firstNameText">Jane</b>
    <input
      id="firstNameInput"
      value="Jane"
      style="display: none">
  </label>
  <label>
    姓氏：
    <b id="lastNameText">Jacobs</b>
    <input
      id="lastNameInput"
      value="Jacobs"
      style="display: none">
  </label>
  <button type="submit" id="editButton">编辑资料</button>
  <p><i id="helloText">你好，Jane Jacobs！</i></p>
</form>

<style>
* { box-sizing: border-box; }
body { font-family: sans-serif; margin: 20px; padding: 0; }
label { display: block; margin-bottom: 20px; }
</style>
```

</Sandpack>

<Solution>

缺失的逻辑包括切换输入框和内容的显示，以及更新标签：

<Sandpack>

```js src/index.js active
let firstName = 'Jane';
let lastName = 'Jacobs';
let isEditing = false;

function handleFormSubmit(e) {
  e.preventDefault();
  setIsEditing(!isEditing);
}

function handleFirstNameChange(e) {
  setFirstName(e.target.value);
}

function handleLastNameChange(e) {
  setLastName(e.target.value);
}

function setFirstName(value) {
  firstName = value;
  updateDOM();
}

function setLastName(value) {
  lastName = value;
  updateDOM();
}

function setIsEditing(value) {
  isEditing = value;
  updateDOM();
}

function updateDOM() {
  if (isEditing) {
    editButton.textContent = '保存资料';
    hide(firstNameText);
    hide(lastNameText);
    show(firstNameInput);
    show(lastNameInput);
  } else {
    editButton.textContent = '编辑资料';
    hide(firstNameInput);
    hide(lastNameInput);
    show(firstNameText);
    show(lastNameText);
  }
  firstNameText.textContent = firstName;
  lastNameText.textContent = lastName;
  helloText.textContent = (
    '你好，' +
    firstName + ' ' +
    lastName + '！'
  );
}

function hide(el) {
  el.style.display = 'none';
}

function show(el) {
  el.style.display = '';
}

let form = document.getElementById('form');
let editButton = document.getElementById('editButton');
let firstNameInput = document.getElementById('firstNameInput');
let firstNameText = document.getElementById('firstNameText');
let lastNameInput = document.getElementById('lastNameInput');
let lastNameText = document.getElementById('lastNameText');
let helloText = document.getElementById('helloText');
form.onsubmit = handleFormSubmit;
firstNameInput.oninput = handleFirstNameChange;
lastNameInput.oninput = handleLastNameChange;
```

```js sandbox.config.json hidden
{
  "hardReloadOnChange": true
}
```

```html public/index.html
<form id="form">
  <label>
    名字：
    <b id="firstNameText">Jane</b>
    <input
      id="firstNameInput"
      value="Jane"
      style="display: none">
  </label>
  <label>
    姓氏：
    <b id="lastNameText">Jacobs</b>
    <input
      id="lastNameInput"
      value="Jacobs"
      style="display: none">
  </label>
  <button type="submit" id="editButton">编辑资料</button>
  <p><i id="helloText">你好，Jane Jacobs！</i></p>
</form>

<style>
* { box-sizing: border-box; }
body { font-family: sans-serif; margin: 20px; padding: 0; }
label { display: block; margin-bottom: 20px; }
</style>
```

</Sandpack>

你写的 `updateDOM` 函数展示了当你设置状态时，React 在底层所做的事情。（不过，React 还会避免触碰那些自上次设置后没有变化的 DOM 属性。）

</Solution>

</Challenges>
