---
title: 保持组件纯净
---

<Intro>

一些 JavaScript 函数是*纯函数*。纯函数只进行计算，不做其他任何事情。只要你严格把组件写成纯函数，随着代码库不断增长，你就可以避免一整类令人困惑的 bug 和不可预测的行为。不过，要获得这些好处，你必须遵守一些规则。

</Intro>

<YouWillLearn>

* 什么是纯净性，以及它如何帮助你避免 bug
* 如何通过避免在渲染阶段进行更改来保持组件纯净
* 如何使用 Strict Mode 在组件中发现错误

</YouWillLearn>

## 纯净性：把组件当作公式来看待 {/*purity-components-as-formulas*/}

在计算机科学中（尤其是在函数式编程的世界里），[纯函数](https://wikipedia.org/wiki/Pure_function)是具有以下特征的函数：

* **各司其职。** 它不会改变在它被调用之前就已存在的任何对象或变量。
* **相同的输入，相同的输出。** 在相同输入下，纯函数应始终返回相同结果。

你可能已经熟悉纯函数的一个例子：数学公式。

考虑这个数学公式：<Math><MathI>y</MathI> = 2<MathI>x</MathI></Math>。

如果 <Math><MathI>x</MathI> = 2</Math>，那么 <Math><MathI>y</MathI> = 4</Math>。始终如此。

如果 <Math><MathI>x</MathI> = 3</Math>，那么 <Math><MathI>y</MathI> = 6</Math>。始终如此。

如果 <Math><MathI>x</MathI> = 3</Math>，<MathI>y</MathI> 不会有时是 <Math>9</Math>、有时是 <Math>–1</Math>，或者根据一天中的时间或股市状况变成 <Math>2.5</Math>。

如果 <Math><MathI>y</MathI> = 2<MathI>x</MathI></Math> 且 <Math><MathI>x</MathI> = 3</Math>，<MathI>y</MathI> 将_总是_是 <Math>6</Math>。

如果我们把它写成一个 JavaScript 函数，它会是这样：

```js
function double(number) {
  return 2 * number;
}
```

在上面的例子中，`double` 是一个**纯函数。**如果你传入 `3`，它就会返回 `6`。始终如此。

React 的设计就围绕这一概念。**React 假设你写的每个组件都是纯函数。**这意味着你编写的 React 组件在相同输入下必须始终返回相同的 JSX：

<Sandpack>

```js src/App.js
function Recipe({ drinkers }) {
  return (
    <ol>
      <li>煮 {drinkers} 杯水。</li>
      <li>加入 {drinkers} 勺茶和 {0.5 * drinkers} 勺香料。</li>
      <li>加入 {0.5 * drinkers} 杯牛奶煮沸，并按口味加糖。</li>
    </ol>
  );
}

export default function App() {
  return (
    <section>
      <h1>香料奶茶配方</h1>
      <h2>两人份</h2>
      <Recipe drinkers={2} />
      <h2>聚会份</h2>
      <Recipe drinkers={4} />
    </section>
  );
}
```

</Sandpack>

当你向 `Recipe` 传入 `drinkers={2}` 时，它会返回包含 `2 cups of water` 的 JSX。始终如此。

如果你传入 `drinkers={4}`，它会返回包含 `4 cups of water` 的 JSX。始终如此。

就像数学公式一样。

你可以把组件看作食谱：如果你按照它来做，并且在烹饪过程中不加入新食材，那么每次都会得到同样的菜肴。这个“菜肴”就是组件提供给 React 用于[渲染](/learn/render-and-commit)的 JSX。

<Illustration src="/images/docs/illustrations/i_puritea-recipe.png" alt="一个供 x 个人使用的茶配方：取 x 杯水，加入 x 勺茶和 0.5x 勺香料，再加入 0.5x 杯牛奶" />

## 副作用：有意或无意的后果 {/*side-effects-unintended-consequences*/}

React 的渲染过程必须始终保持纯净。组件应该只*返回*它们的 JSX，而不应该*改变*渲染之前已经存在的任何对象或变量——那样会让它们变成不纯的！

下面是一个违反这条规则的组件：

<Sandpack>

```js {expectedErrors: {'react-compiler': [5]}}
let guest = 0;

function Cup() {
  // 坏：修改了一个先前已经存在的变量！
  guest = guest + 1;
  return <h2>Tea cup for guest #{guest}</h2>;
}

export default function TeaSet() {
  return (
    <>
      <Cup />
      <Cup />
      <Cup />
    </>
  );
}
```

</Sandpack>

这个组件正在读写在它外部声明的 `guest` 变量。这意味着**多次调用这个组件会产生不同的 JSX！**更糟糕的是，如果_其他_组件读取 `guest`，它们也会根据渲染时机的不同而产生不同的 JSX！这并不可预测。

回到我们的公式 <Math><MathI>y</MathI> = 2<MathI>x</MathI></Math>，现在即使 <Math><MathI>x</MathI> = 2</Math>，我们也不能相信 <Math><MathI>y</MathI> = 4</Math>。我们的测试可能会失败，用户会感到困惑，飞机会从天上掉下来——你可以看出这会导致多么混乱的 bug！

你可以通过[改为将 `guest` 作为 prop 传入](/learn/passing-props-to-a-component)来修复这个组件：

<Sandpack>

```js
function Cup({ guest }) {
  return <h2>Tea cup for guest #{guest}</h2>;
}

export default function TeaSet() {
  return (
    <>
      <Cup guest={1} />
      <Cup guest={2} />
      <Cup guest={3} />
    </>
  );
}
```

</Sandpack>

现在你的组件是纯净的，因为它返回的 JSX 只依赖于 `guest` prop。

一般来说，你不应该预期组件会按照任何特定顺序渲染。无论你是先调用 <Math><MathI>y</MathI> = 2<MathI>x</MathI></Math> 还是先调用 <Math><MathI>y</MathI> = 5<MathI>x</MathI></Math> 都无所谓：这两个公式都会彼此独立地求值。同样地，每个组件都应该只“考虑自己”，而不要在渲染期间试图与其他组件协调或依赖它们。渲染就像考试：每个组件都应该独立计算 JSX！

<DeepDive>

#### 使用 StrictMode 检测不纯的计算 {/*detecting-impure-calculations-with-strict-mode*/}

虽然你可能还没有全部用过，但在 React 中，渲染时可以读取三种输入：[props](/learn/passing-props-to-a-component)、[state](/learn/state-a-components-memory) 和 [context](/learn/passing-data-deeply-with-context)。你应该始终把这些输入视为只读。

当你想要在响应用户输入时*更改*某些内容时，你应该[设置 state](/learn/state-a-components-memory)，而不是写入某个变量。你永远不应该在组件渲染时更改先前已经存在的变量或对象。

React 提供了一个 “Strict Mode”，在开发环境中它会把每个组件函数调用两次。**通过调用组件函数两次，Strict Mode 可以帮助发现违反这些规则的组件。**

注意原始示例显示的是 “Guest #2”、“Guest #4” 和 “Guest #6”，而不是 “Guest #1”、“Guest #2” 和 “Guest #3”。原始函数是不纯的，所以调用两次就把它搞坏了。但修复后的纯版本即使每次都调用两次也能正常工作。**纯函数只做计算，所以调用两次不会改变任何东西**——就像调用 `double(2)` 两次不会改变返回值，以及解 <Math><MathI>y</MathI> = 2<MathI>x</MathI></Math> 两次也不会改变 <MathI>y</MathI> 的值一样。相同输入，相同输出。始终如此。

Strict Mode 在生产环境中没有影响，因此不会拖慢用户的应用。要启用 Strict Mode，你可以把根组件包裹在 `<React.StrictMode>` 中。有些框架默认就这么做。

</DeepDive>

### 局部突变：你组件的小秘密 {/*local-mutation-your-components-little-secret*/}

在上面的例子中，问题在于组件在渲染期间修改了一个*先前已经存在*的变量。这通常被称为**“突变”**，这样听起来会更可怕一点。纯函数不会修改函数作用域之外的变量，或在调用之前创建的对象——那会让它们变得不纯！

然而，**在渲染时修改你*刚刚创建*的变量和对象是完全可以的。**在这个例子中，你创建了一个 `[]` 数组，把它赋给 `cups` 变量，然后向其中 `push` 了十几个杯子：

<Sandpack>

```js
function Cup({ guest }) {
  return <h2>Tea cup for guest #{guest}</h2>;
}

export default function TeaGathering() {
  const cups = [];
  for (let i = 1; i <= 12; i++) {
    cups.push(<Cup key={i} guest={i} />);
  }
  return cups;
}
```

</Sandpack>

如果 `cups` 变量或 `[]` 数组是在 `TeaGathering` 函数外创建的，那就会是大问题！你会通过向数组中添加元素来修改一个*先前已经存在*的对象。

不过这没问题，因为你是在 `TeaGathering` 内部、*同一次渲染过程中*创建它们的。`TeaGathering` 外部的任何代码都不会知道这件事。这就叫做**“局部突变”**——就像你组件的小秘密。

## 你可以在哪里产生副作用 {/*where-you-_can_-cause-side-effects*/}

虽然函数式编程非常依赖纯净性，但在某个时刻、某个地方，_某些东西_必须改变。这也算编程的意义所在！这些变化——更新屏幕、启动动画、修改数据——称为**副作用**。它们是发生在“旁边”的事情，而不是在渲染期间发生的。

在 React 中，**副作用通常属于[事件处理函数](/learn/responding-to-events)**。事件处理函数是在你执行某些操作时由 React 运行的函数，例如点击按钮。即使事件处理函数定义在组件内部，它们也不会在渲染期间运行！**所以事件处理函数不需要是纯函数。**

如果你已经尝试了所有其他办法，仍然找不到合适的事件处理函数来处理副作用，你仍然可以在组件中通过调用 [`useEffect`](/reference/react/useEffect) 将它附加到返回的 JSX 上。这会告诉 React 在渲染之后、允许副作用时再执行它。**不过，这种方式应该是你的最后手段。**

在可能的情况下，尽量只通过渲染来表达你的逻辑。你会惊讶于这能带你走多远！

<DeepDive>

#### React 为什么关心纯净性？ {/*why-does-react-care-about-purity*/}

编写纯函数需要一些习惯和纪律。但它也能解锁一些令人惊叹的可能性：

* 你的组件可以在不同环境中运行——例如在服务器上！由于它们在相同输入下会返回相同结果，一个组件就可以服务于多个用户请求。
* 你可以通过[跳过渲染](/reference/react/memo)未发生变化的组件来提升性能。这是安全的，因为纯函数总是返回相同结果，因此可以安全缓存。
* 如果某些数据在深层组件树渲染的中途发生变化，React 可以重新开始渲染，而不会浪费时间去完成已经过时的渲染。纯净性使得在任何时刻停止计算都变得安全。

我们正在构建的每一个新 React 特性都在利用纯净性。从数据获取到动画再到性能，保持组件纯净能释放 React 范式的力量。

</DeepDive>

<Recap>

* 组件必须是纯净的，这意味着：
  * **各司其职。** 它不应该改变渲染之前已经存在的任何对象或变量。
  * **相同的输入，相同的输出。** 在相同输入下，组件应始终返回相同的 JSX。
* 渲染可以在任何时候发生，所以组件不应该依赖彼此的渲染顺序。
* 你不应该修改组件用于渲染的任何输入。这包括 props、state 和 context。要更新屏幕，请[“设置” state](/learn/state-a-components-memory)，而不是修改先前存在的对象。
* 尽量把组件的逻辑表达在你返回的 JSX 中。当你需要“改变某些东西”时，通常应该在事件处理函数中完成。作为最后手段，你可以使用 `useEffect`。
* 编写纯函数需要一些练习，但它能释放 React 范式的力量。

</Recap>



<Challenges>

#### 修复一个损坏的时钟 {/*fix-a-broken-clock*/}

这个组件试图在午夜到早上六点之间把 `<h1>` 的 CSS 类设为 `"night"`，并在其他时间设为 `"day"`。然而它并不起作用。你能修复这个组件吗？

你可以通过临时更改计算机的时区来验证你的解决方案是否有效。当当前时间在午夜到早上六点之间时，时钟应该显示反转的颜色！

<Hint>

渲染是一个*计算*，它不应该试图“做”任何事情。你能用不同方式表达同样的想法吗？

</Hint>

<Sandpack>

```js src/Clock.js active
export default function Clock({ time }) {
  const hours = time.getHours();
  if (hours >= 0 && hours <= 6) {
    document.getElementById('time').className = 'night';
  } else {
    document.getElementById('time').className = 'day';
  }
  return (
    <h1 id="time">
      {time.toLocaleTimeString()}
    </h1>
  );
}
```

```js src/App.js hidden
import { useState, useEffect } from 'react';
import Clock from './Clock.js';

function useTime() {
  const [time, setTime] = useState(() => new Date());
  useEffect(() => {
    const id = setInterval(() => {
      setTime(new Date());
    }, 1000);
    return () => clearInterval(id);
  }, []);
  return time;
}

export default function App() {
  const time = useTime();
  return (
    <Clock time={time} />
  );
}
```

```css
body > * {
  width: 100%;
  height: 100%;
}
.day {
  background: #fff;
  color: #222;
}
.night {
  background: #222;
  color: #fff;
}
```

</Sandpack>

<Solution>

你可以通过计算 `className` 并将其包含在渲染输出中来修复这个组件：

<Sandpack>

```js src/Clock.js active
export default function Clock({ time }) {
  const hours = time.getHours();
  let className;
  if (hours >= 0 && hours <= 6) {
    className = 'night';
  } else {
    className = 'day';
  }
  return (
    <h1 className={className}>
      {time.toLocaleTimeString()}
    </h1>
  );
}
```

```js src/App.js hidden
import { useState, useEffect } from 'react';
import Clock from './Clock.js';

function useTime() {
  const [time, setTime] = useState(() => new Date());
  useEffect(() => {
    const id = setInterval(() => {
      setTime(new Date());
    }, 1000);
    return () => clearInterval(id);
  }, []);
  return time;
}

export default function App() {
  const time = useTime();
  return (
    <Clock time={time} />
  );
}
```

```css
body > * {
  width: 100%;
  height: 100%;
}
.day {
  background: #fff;
  color: #222;
}
.night {
  background: #222;
  color: #fff;
}
```

</Sandpack>

在这个例子中，副作用（修改 DOM）完全没有必要。你只需要返回 JSX。

</Solution>

#### 修复一个损坏的个人资料 {/*fix-a-broken-profile*/}

两个 `Profile` 组件并排渲染，数据不同。点击第一个资料的 “Collapse”，然后再点击 “Expand”。你会注意到两个资料现在显示的是同一个人。这是一个 bug。

找出 bug 的原因并修复它。

<Hint>

有 bug 的代码在 `Profile.js` 中。确保你从上到下完整阅读它！

</Hint>

<Sandpack>

```js {expectedErrors: {'react-compiler': [7]}} src/Profile.js
import Panel from './Panel.js';
import { getImageUrl } from './utils.js';

let currentPerson;

export default function Profile({ person }) {
  currentPerson = person;
  return (
    <Panel>
      <Header />
      <Avatar />
    </Panel>
  )
}

function Header() {
  return <h1>{currentPerson.name}</h1>;
}

function Avatar() {
  return (
    <img
      className="avatar"
      src={getImageUrl(currentPerson)}
      alt={currentPerson.name}
      width={50}
      height={50}
    />
  );
}
```

```js src/Panel.js hidden
import { useState } from 'react';

export default function Panel({ children }) {
  const [open, setOpen] = useState(true);
  return (
    <section className="panel">
      <button onClick={() => setOpen(!open)}>
        {open ? 'Collapse' : 'Expand'}
      </button>
      {open && children}
    </section>
  );
}
```

```js src/App.js
import Profile from './Profile.js';

export default function App() {
  return (
    <>
      <Profile person={{
        imageId: 'lrWQx8l',
        name: 'Subrahmanyan Chandrasekhar',
      }} />
      <Profile person={{
        imageId: 'MK3eW3A',
        name: 'Creola Katherine Johnson',
      }} />
    </>
  )
}
```

```js src/utils.js hidden
export function getImageUrl(person, size = 's') {
  return (
    'https://react.dev/images/docs/scientists/' +
    person.imageId +
    size +
    '.jpg'
  );
}
```

```css
.avatar { margin: 5px; border-radius: 50%; }
.panel {
  border: 1px solid #aaa;
  border-radius: 6px;
  margin-top: 20px;
  padding: 10px;
  width: 200px;
}
h1 { margin: 5px; font-size: 18px; }
```

</Sandpack>

<Solution>

问题在于 `Profile` 组件把值写入了一个名为 `currentPerson` 的先前存在的变量，而 `Header` 和 `Avatar` 组件从中读取。这让这三个组件都变得不纯净，而且难以预测。

要修复这个 bug，请移除 `currentPerson` 变量。然后通过 props 将 `Profile` 中的所有信息传递给 `Header` 和 `Avatar`。你需要给这两个组件都添加一个 `person` prop，并一路向下传递。

<Sandpack>

```js src/Profile.js active
import Panel from './Panel.js';
import { getImageUrl } from './utils.js';

export default function Profile({ person }) {
  return (
    <Panel>
      <Header person={person} />
      <Avatar person={person} />
    </Panel>
  )
}

function Header({ person }) {
  return <h1>{person.name}</h1>;
}

function Avatar({ person }) {
  return (
    <img
      className="avatar"
      src={getImageUrl(person)}
      alt={person.name}
      width={50}
      height={50}
    />
  );
}
```

```js src/Panel.js hidden
import { useState } from 'react';

export default function Panel({ children }) {
  const [open, setOpen] = useState(true);
  return (
    <section className="panel">
      <button onClick={() => setOpen(!open)}>
        {open ? 'Collapse' : 'Expand'}
      </button>
      {open && children}
    </section>
  );
}
```

```js src/App.js
import Profile from './Profile.js';

export default function App() {
  return (
    <>
      <Profile person={{
        imageId: 'lrWQx8l',
        name: 'Subrahmanyan Chandrasekhar',
      }} />
      <Profile person={{
        imageId: 'MK3eW3A',
        name: 'Creola Katherine Johnson',
      }} />
    </>
  );
}
```

```js src/utils.js hidden
export function getImageUrl(person, size = 's') {
  return (
    'https://react.dev/images/docs/scientists/' +
    person.imageId +
    size +
    '.jpg'
  );
}
```

```css
.avatar { margin: 5px; border-radius: 50%; }
.panel {
  border: 1px solid #aaa;
  border-radius: 6px;
  margin-top: 20px;
  padding: 10px;
  width: 200px;
}
h1 { margin: 5px; font-size: 18px; }
```

</Sandpack>

请记住，React 不保证组件函数会按任何特定顺序执行，因此你不能通过设置变量在它们之间通信。所有通信都必须通过 props 进行。

</Solution>

#### 修复一个损坏的故事托盘 {/*fix-a-broken-story-tray*/}

你所在公司的 CEO 要求你给在线时钟应用添加“故事”，而你不能拒绝。你已经编写了一个 `StoryTray` 组件，它接收一个 `stories` 列表，然后跟着一个 “Create Story” 占位项。

你通过在作为 prop 接收到的 `stories` 数组末尾再 push 一个假的故事来实现 “Create Story” 占位项。但不知为何，“Create Story” 出现了不止一次。请修复这个问题。

<Sandpack>

```js src/StoryTray.js active
export default function StoryTray({ stories }) {
  stories.push({
    id: 'create',
    label: 'Create Story'
  });

  return (
    <ul>
      {stories.map(story => (
        <li key={story.id}>
          {story.label}
        </li>
      ))}
    </ul>
  );
}
```

```js {expectedErrors: {'react-compiler': [16]}} src/App.js hidden
import { useState, useEffect } from 'react';
import StoryTray from './StoryTray.js';

const initialStories = [
  {id: 0, label: "Ankit's Story" },
  {id: 1, label: "Taylor's Story" },
];

export default function App() {
  const [stories, setStories] = useState([...initialStories])
  const time = useTime();

  // 技巧：在你阅读文档时，防止内存无限增长。
  // 我们这里违反了自己的规则。
  if (stories.length > 100) {
    stories.length = 100;
  }

  return (
    <div
      style={{
        width: '100%',
        height: '100%',
        textAlign: 'center',
      }}
    >
      <h2>现在是 {time.toLocaleTimeString()}。</h2>
      <StoryTray stories={stories} />
    </div>
  );
}

function useTime() {
  const [time, setTime] = useState(() => new Date());
  useEffect(() => {
    const id = setInterval(() => {
      setTime(new Date());
    }, 1000);
    return () => clearInterval(id);
  }, []);
  return time;
}
```

```css
ul {
  margin: 0;
  list-style-type: none;
}

li {
  border: 1px solid #aaa;
  border-radius: 6px;
  float: left;
  margin: 5px;
  margin-bottom: 20px;
  padding: 5px;
  width: 70px;
  height: 100px;
}
```

```js sandbox.config.json hidden
{
  "hardReloadOnChange": true
}
```

</Sandpack>

<Solution>

注意每当时钟更新时，“Create Story” 会被添加*两次*。这提示我们在渲染期间发生了突变——Strict Mode 通过调用组件两次让这些问题更明显。

`StoryTray` 函数不是纯函数。它对收到的 `stories` 数组（一个 prop！）调用了 `push`，这会修改一个在 `StoryTray` 开始渲染之前就已经创建好的对象。这使它变得有 bug，而且极难预测。

最简单的修复方法是根本不碰这个数组，而是单独渲染 “Create Story”：

<Sandpack>

```js src/StoryTray.js active
export default function StoryTray({ stories }) {
  return (
    <ul>
      {stories.map(story => (
        <li key={story.id}>
          {story.label}
        </li>
      ))}
      <li>Create Story</li>
    </ul>
  );
}
```

```js {expectedErrors: {'react-compiler': [16]}} src/App.js hidden
import { useState, useEffect } from 'react';
import StoryTray from './StoryTray.js';

const initialStories = [
  {id: 0, label: "Ankit's Story" },
  {id: 1, label: "Taylor's Story" },
];

export default function App() {
  const [stories, setStories] = useState([...initialStories])
  const time = useTime();

  // 技巧：在你阅读文档时，防止内存无限增长。
  // 我们这里违反了自己的规则。
  if (stories.length > 100) {
    stories.length = 100;
  }

  return (
    <div
      style={{
        width: '100%',
        height: '100%',
        textAlign: 'center',
      }}
    >
      <h2>现在是 {time.toLocaleTimeString()}。</h2>
      <StoryTray stories={stories} />
    </div>
  );
}

function useTime() {
  const [time, setTime] = useState(() => new Date());
  useEffect(() => {
    const id = setInterval(() => {
      setTime(new Date());
    }, 1000);
    return () => clearInterval(id);
  }, []);
  return time;
}
```

```css
ul {
  margin: 0;
  list-style-type: none;
}

li {
  border: 1px solid #aaa;
  border-radius: 6px;
  float: left;
  margin: 5px;
  margin-bottom: 20px;
  padding: 5px;
  width: 70px;
  height: 100px;
}
```

</Sandpack>

或者，你也可以先创建一个_新_数组（通过复制现有数组），然后再向其中添加项目：

<Sandpack>

```js src/StoryTray.js active
export default function StoryTray({ stories }) {
  // 复制数组！
  const storiesToDisplay = stories.slice();

  // 不会影响原始数组：
  storiesToDisplay.push({
    id: 'create',
    label: 'Create Story'
  });

  return (
    <ul>
      {storiesToDisplay.map(story => (
        <li key={story.id}>
          {story.label}
        </li>
      ))}
    </ul>
  );
}
```

```js {expectedErrors: {'react-compiler': [16]}} src/App.js hidden
import { useState, useEffect } from 'react';
import StoryTray from './StoryTray.js';

const initialStories = [
  {id: 0, label: "Ankit's Story" },
  {id: 1, label: "Taylor's Story" },
];

export default function App() {
  const [stories, setStories] = useState([...initialStories])
  const time = useTime();

  // 技巧：在你阅读文档时，防止内存无限增长。
  // 我们这里违反了自己的规则。
  if (stories.length > 100) {
    stories.length = 100;
  }

  return (
    <div
      style={{
        width: '100%',
        height: '100%',
        textAlign: 'center',
      }}
    >
      <h2>现在是 {time.toLocaleTimeString()}。</h2>
      <StoryTray stories={stories} />
    </div>
  );
}

function useTime() {
  const [time, setTime] = useState(() => new Date());
  useEffect(() => {
    const id = setInterval(() => {
      setTime(new Date());
    }, 1000);
    return () => clearInterval(id);
  }, []);
  return time;
}
```

```css
ul {
  margin: 0;
  list-style-type: none;
}

li {
  border: 1px solid #aaa;
  border-radius: 6px;
  float: left;
  margin: 5px;
  margin-bottom: 20px;
  padding: 5px;
  width: 70px;
  height: 100px;
}
```

</Sandpack>

这会将你的突变限制在局部，并使你的渲染函数保持纯净。不过，你仍然需要小心：例如，如果你试图更改数组中已有的项目，那么你也必须克隆这些项目。

记住数组上哪些操作会修改它们，哪些不会，这很有用。例如，`push`、`pop`、`reverse` 和 `sort` 会修改原始数组，但 `slice`、`filter` 和 `map` 会创建一个新数组。

</Solution>

</Challenges>
