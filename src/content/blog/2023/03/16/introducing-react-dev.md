---
title: "介绍 react.dev"
author: Dan Abramov and Rachel Nabors
date: 2023/03/16
description: 今天我们非常高兴地推出 react.dev，这是 React 及其文档的新家。在这篇文章中，我们将带你参观这个新网站。
---

2023 年 3 月 16 日，来自 [Dan Abramov](https://bsky.app/profile/danabra.mov) 和 [Rachel Nabors](https://twitter.com/rachelnabors)

---

<Intro>

今天我们非常高兴地推出 [react.dev](https://react.dev)，这是 React 及其文档的新家。在这篇文章中，我们将带你参观这个新网站。

</Intro>

---

## 简而言之 {/*tldr*/}

* 新的 React 网站（[react.dev](https://react.dev)）通过函数组件和 Hooks 来教授现代 React。
* 我们加入了图表、插图、挑战题，以及 600 多个新的交互式示例。
* 之前的 React 文档网站现在已迁移到 [legacy.reactjs.org](https://legacy.reactjs.org)。

## 新网站、新域名、新首页 {/*new-site-new-domain-new-homepage*/}

首先，先做一点整理工作。

为了庆祝新文档的发布，更重要的是为了清晰地区分旧内容和新内容，我们已迁移到更简短的 [react.dev](https://react.dev) 域名。旧的 [reactjs.org](https://reactjs.org) 域名现在会重定向到这里。

旧的 React 文档现在已归档在 [legacy.reactjs.org](https://legacy.reactjs.org)。所有指向旧内容的现有链接都会自动重定向到那里，以避免“破坏网络”，但旧站点将不会再获得太多更新。

信不信由你，React 很快就要十岁了。按 JavaScript 的“年龄”来算，那简直是一整个世纪！我们已[更新 React 首页](https://react.dev)，以反映我们为什么认为 React 是当今创建用户界面的绝佳方式，并更新了入门指南，更醒目地介绍现代 React 相关框架。

如果你还没看过新首页，去看看吧！

## 全面拥抱使用 Hooks 的现代 React {/*going-all-in-on-modern-react-with-hooks*/}

当我们在 2018 年发布 React Hooks 时，Hooks 文档默认读者已经熟悉类组件。这帮助社区非常迅速地采用了 Hooks，但过了一段时间后，旧文档就不再能很好地服务新读者了。新读者不得不把 React 学两遍：先学类组件，再学一次 Hooks。

**新文档从一开始就使用 Hooks 教授 React。** 文档分为两个主要部分：

* **[学习 React](/learn)** 是一门自定进度的课程，从零开始教授 React。
* **[API 参考](/reference)** 提供每个 React API 的细节和使用示例。

让我们更仔细地看看你可以在每个部分中找到什么。

<Note>

仍然有少数罕见的类组件使用场景，还没有基于 Hook 的等价实现。类组件仍然受支持，并在新站点的 [旧版 API](/reference/react/legacy) 部分中有文档说明。

</Note>

## 快速开始 {/*quick-start*/}

“学习”部分从 [快速开始](/learn) 页面开始。这是一段简短的 React 入门导览。它介绍了组件、props 和 state 等概念的语法，但不会深入讲解如何使用它们。

如果你喜欢边做边学，我们建议下一步查看 [井字棋教程](/learn/tutorial-tic-tac-toe)。它会带你用 React 构建一个小游戏，同时教授你每天都会用到的技能。下面是你将要构建的内容：

<Sandpack>

```js src/App.js
import { useState } from 'react';

function Square({ value, onSquareClick }) {
  return (
    <button className="square" onClick={onSquareClick}>
      {value}
    </button>
  );
}

function Board({ xIsNext, squares, onPlay }) {
  function handleClick(i) {
    if (calculateWinner(squares) || squares[i]) {
      return;
    }
    const nextSquares = squares.slice();
    if (xIsNext) {
      nextSquares[i] = 'X';
    } else {
      nextSquares[i] = 'O';
    }
    onPlay(nextSquares);
  }

  const winner = calculateWinner(squares);
  let status;
  if (winner) {
    status = '获胜者：' + winner;
  } else {
    status = '下一位玩家：' + (xIsNext ? 'X' : 'O');
  }

  return (
    <>
      <div className="status">{status}</div>
      <div className="board-row">
        <Square value={squares[0]} onSquareClick={() => handleClick(0)} />
        <Square value={squares[1]} onSquareClick={() => handleClick(1)} />
        <Square value={squares[2]} onSquareClick={() => handleClick(2)} />
      </div>
      <div className="board-row">
        <Square value={squares[3]} onSquareClick={() => handleClick(3)} />
        <Square value={squares[4]} onSquareClick={() => handleClick(4)} />
        <Square value={squares[5]} onSquareClick={() => handleClick(5)} />
      </div>
      <div className="board-row">
        <Square value={squares[6]} onSquareClick={() => handleClick(6)} />
        <Square value={squares[7]} onSquareClick={() => handleClick(7)} />
        <Square value={squares[8]} onSquareClick={() => handleClick(8)} />
      </div>
    </>
  );
}

export default function Game() {
  const [history, setHistory] = useState([Array(9).fill(null)]);
  const [currentMove, setCurrentMove] = useState(0);
  const xIsNext = currentMove % 2 === 0;
  const currentSquares = history[currentMove];

  function handlePlay(nextSquares) {
    const nextHistory = [...history.slice(0, currentMove + 1), nextSquares];
    setHistory(nextHistory);
    setCurrentMove(nextHistory.length - 1);
  }

  function jumpTo(nextMove) {
    setCurrentMove(nextMove);
  }

  const moves = history.map((squares, move) => {
    let description;
    if (move > 0) {
      description = '转到第 ' + move + ' 步';
    } else {
      description = '回到游戏开始';
    }
    return (
      <li key={move}>
        <button onClick={() => jumpTo(move)}>{description}</button>
      </li>
    );
  });

  return (
    <div className="game">
      <div className="game-board">
        <Board xIsNext={xIsNext} squares={currentSquares} onPlay={handlePlay} />
      </div>
      <div className="game-info">
        <ol>{moves}</ol>
      </div>
    </div>
  );
}

function calculateWinner(squares) {
  const lines = [
    [0, 1, 2],
    [3, 4, 5],
    [6, 7, 8],
    [0, 3, 6],
    [1, 4, 7],
    [2, 5, 8],
    [0, 4, 8],
    [2, 4, 6],
  ];
  for (let i = 0; i < lines.length; i++) {
    const [a, b, c] = lines[i];
    if (squares[a] && squares[a] === squares[b] && squares[a] === squares[c]) {
      return squares[a];
    }
  }
  return null;
}
```

```css src/styles.css
* {
  box-sizing: border-box;
}

body {
  font-family: sans-serif;
  margin: 20px;
  padding: 0;
}

.square {
  background: #fff;
  border: 1px solid #999;
  float: left;
  font-size: 24px;
  font-weight: bold;
  line-height: 34px;
  height: 34px;
  margin-right: -1px;
  margin-top: -1px;
  padding: 0;
  text-align: center;
  width: 34px;
}

.board-row:after {
  clear: both;
  content: '';
  display: table;
}

.status {
  margin-bottom: 10px;
}
.game {
  display: flex;
  flex-direction: row;
}

.game-info {
  margin-left: 20px;
}
```

</Sandpack>

我们也想重点介绍一下 [React 思维](/learn/thinking-in-react)——这就是让 React 对我们许多人来说“豁然开朗”的教程。**我们已经把这两个经典教程都更新为使用函数组件和 Hooks，**所以它们焕然一新。

<Note>

上面的示例是一个 *sandbox*。我们在整个网站中加入了很多 sandbox——超过 600 个！——你可以在任何地方看到它们。你可以编辑任意 sandbox，或点击右上角的“Fork”在新标签页中打开它。sandbox 能让你快速尝试 React API、探索你的想法，并检查自己是否真正理解。

</Note>

## 逐步学习 React {/*learn-react-step-by-step*/}

我们希望世界上每个人都有平等机会，可以免费、独立地学习 React。

这也是“学习”部分被组织成一个自定进度、按章节划分的课程的原因。前两个章节介绍 React 的基础知识。如果你是 React 新手，或者想回顾一下，就从这里开始：

- **[描述 UI](/learn/describing-the-ui)** 教你如何用组件展示信息。
- **[添加交互性](/learn/adding-interactivity)** 教你如何响应用户输入来更新屏幕。

接下来的两个章节更高级，会让你更深入了解那些更棘手的部分：

- **[管理状态](/learn/managing-state)** 教你如何在应用复杂度增长时组织逻辑。
- **[逃生舱口](/learn/escape-hatches)** 教你如何“走出” React，以及在什么情况下这样做最合适。

每个章节都由若干相关页面组成。这些页面大多教授某项具体技能或技术——例如，[使用 JSX 编写标记](/learn/writing-markup-with-jsx)、[更新 state 中的对象](/learn/updating-objects-in-state) 或 [在组件之间共享 state](/learn/sharing-state-between-components)。有些页面专注于解释一个概念——比如 [渲染与提交](/learn/render-and-commit) 或 [快照中的 state](/learn/state-as-a-snapshot)。还有一些页面，比如 [你可能不需要 Effect](/learn/you-might-not-need-an-effect)，会基于我们这些年来学到的经验分享我们的建议。

你不必按顺序阅读这些章节。谁有这个时间啊？！不过你也可以这么做。“学习”部分中的页面只依赖于前面页面介绍过的概念。如果你想把它当作一本书来读，那就尽管去读吧！

### 通过挑战检验你的理解 {/*check-your-understanding-with-challenges*/}

“学习”部分中的大多数页面都会在结尾提供一些挑战题，用来检验你的理解。例如，这里有几个来自 [条件渲染](/learn/conditional-rendering#challenges) 页面上的挑战题。

你现在不必马上解答它们！除非你*真的*想。

<Challenges noTitle={true}>

#### 为未完成项目显示一个 `? :` 图标 {/*show-an-icon-for-incomplete-items-with--*/}

使用条件运算符（`cond ? a : b`）在 `isPacked` 不为 `true` 时渲染一个 ❌。

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
          name="Space suit"
        />
        <Item
          isPacked={true}
          name="Helmet with a golden leaf"
        />
        <Item
          isPacked={false}
          name="Photo of Tam"
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
          name="Space suit"
        />
        <Item
          isPacked={true}
          name="Helmet with a golden leaf"
        />
        <Item
          isPacked={false}
          name="Photo of Tam"
        />
      </ul>
    </section>
  );
}
```

</Sandpack>

</Solution>

#### 使用 `&&` 显示项目重要性 {/*show-the-item-importance-with-*/}

在这个示例中，每个 `Item` 都会接收一个数字类型的 `importance` prop。使用 `&&` 运算符以斜体渲染 “_(重要性：X)_”，但只针对重要性非零的项目。你的项目列表最终应如下所示：

* Space suit _(重要性：9)_
* Helmet with a golden leaf
* Photo of Tam _(重要性：6)_

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
          name="Space suit"
        />
        <Item
          importance={0}
          name="Helmet with a golden leaf"
        />
        <Item
          importance={6}
          name="Photo of Tam"
        />
      </ul>
    </section>
  );
}
```

</Sandpack>

<Solution>

这样就能解决：

<Sandpack>

```js
function Item({ name, importance }) {
  return (
    <li className="item">
      {name}
      {importance > 0 && ' '}
      {importance > 0 &&
        <i>(重要性：{importance})</i>
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
          name="Space suit"
        />
        <Item
          importance={0}
          name="Helmet with a golden leaf"
        />
        <Item
          importance={6}
          name="Photo of Tam"
        />
      </ul>
    </section>
  );
}
```

</Sandpack>

注意，你必须写 `importance > 0 && ...`，而不是 `importance && ...`，这样当 `importance` 为 `0` 时，`0` 就不会作为结果被渲染出来！

在这个解法中，使用了两个独立条件来在名称和重要性标签之间插入空格。或者，你也可以使用带前导空格的 Fragment：`importance > 0 && <> <i>...</i></>`，或者直接在 `<i>` 内部添加一个空格：`importance > 0 && <i> ...</i>`。

</Solution>

</Challenges>

注意左下角的“显示解答”按钮。如果你想自查，它会很方便！

### 通过图表和插图建立直觉 {/*build-an-intuition-with-diagrams-and-illustrations*/}

当我们无法仅用代码和文字清楚解释某个内容时，就会加入一些图表来帮助建立直觉。例如，这里有一张来自 [保留和重置 state](/learn/preserving-and-resetting-state) 的图：

<Diagram name="preserving_state_diff_same_pt1" height={350} width={794} alt="Diagram with three sections, with an arrow transitioning each section in between. The first section contains a React component labeled 'div' with a single child labeled 'section', which has a single child labeled 'Counter' containing a state bubble labeled 'count' with value 3. The middle section has the same 'div' parent, but the child components have now been deleted, indicated by a yellow 'proof' image. The third section has the same 'div' parent again, now with a new child labeled 'div', highlighted in yellow, also with a new child labeled 'Counter' containing a state bubble labeled 'count' with value 0, all highlighted in yellow.">

当 `section` 变为 `div` 时，`section` 会被删除，而新的 `div` 会被添加

</Diagram>

你还会在文档中看到一些插图——比如这张 [浏览器绘制屏幕](/learn/render-and-commit#epilogue-browser-paint) 的插图：

<Illustration alt="A browser painting 'still life with card element'." src="/images/docs/illustrations/i_browser-paint.png" />

我们已经向浏览器厂商确认，这幅图在科学上 100% 准确。

## 一个全新的、详细的 API 参考 {/*a-new-detailed-api-reference*/}

在 [API 参考](/reference/react) 中，现在每个 React API 都有一个专属页面。这包括各种类型的 API：

- 像 [`useState`](/reference/react/useState) 这样的内置 Hooks。
- 像 [`<Suspense>`](/reference/react/Suspense) 这样的内置组件。
- 像 [`<input>`](/reference/react-dom/components/input) 这样的内置浏览器组件。
- 像 [`renderToPipeableStream`](/reference/react-dom/server/renderToReadableStream) 这样的面向框架的 API。
- 像 [`memo`](/reference/react/memo) 这样的其他 React API。

你会注意到，每个 API 页面都至少被分成两个部分：*参考* 和 *用法*。

[参考](/reference/react/useState#reference) 通过列出其参数和返回值来描述正式的 API 签名。它简洁明了，但如果你不熟悉该 API，可能会觉得有点抽象。它描述的是 API 做什么，但没有说明如何使用它。

[用法](/reference/react/useState#usage) 会像同事或朋友一样，展示你为什么以及如何在实践中使用这个 API。它展示了** React 团队认为每个 API 应当被使用的典型场景。** 我们添加了带颜色标记的代码片段、多个 API 组合使用的示例，以及你可以复制粘贴的配方：

<Recipes titleText="useState 基础示例" titleId="examples-basic">

#### 计数器（数字） {/*counter-number*/}

在这个示例中，`count` 状态变量保存一个数字。点击按钮会使它递增。

<Sandpack>

```js
import { useState } from 'react';

export default function Counter() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1);
  }

  return (
    <button onClick={handleClick}>
      你已经按了我 {count} 次
    </button>
  );
}
```

</Sandpack>

<Solution />

#### 文本框（字符串） {/*text-field-string*/}

在这个示例中，`text` 状态变量保存一个字符串。当你输入时，`handleChange` 会从浏览器 input DOM 元素中读取最新的输入值，并调用 `setText` 来更新状态。这样你就可以在下方显示当前的 `text`。

<Sandpack>

```js
import { useState } from 'react';

export default function MyInput() {
  const [text, setText] = useState('hello');

  function handleChange(e) {
    setText(e.target.value);
  }

  return (
    <>
      <input value={text} onChange={handleChange} />
      <p>你输入了：{text}</p>
      <button onClick={() => setText('hello')}>
        重置
      </button>
    </>
  );
}
```

</Sandpack>

<Solution />

#### 复选框（布尔值） {/*checkbox-boolean*/}

在这个示例中，`liked` 状态变量保存一个布尔值。当你点击输入框时，`setLiked` 会根据浏览器复选框 input 是否被勾选来更新 `liked` 状态变量。`liked` 变量用于渲染复选框下方的文本。

<Sandpack>

```js
import { useState } from 'react';

export default function MyCheckbox() {
  const [liked, setLiked] = useState(true);

  function handleChange(e) {
    setLiked(e.target.checked);
  }

  return (
    <>
      <label>
        <input
          type="checkbox"
          checked={liked}
          onChange={handleChange}
        />
        我喜欢这个
      </label>
      <p>你{liked ? '喜欢' : '不喜欢'}这个。</p>
    </>
  );
}
```

</Sandpack>

<Solution />

#### 表单（两个变量） {/*form-two-variables*/}

你可以在同一个组件中声明多个状态变量。每个状态变量都是完全独立的。

<Sandpack>

```js
import { useState } from 'react';

export default function Form() {
  const [name, setName] = useState('Taylor');
  const [age, setAge] = useState(42);

  return (
    <>
      <input
        value={name}
        onChange={e => setName(e.target.value)}
      />
      <button onClick={() => setAge(age + 1)}>
        年龄增加
      </button>
      <p>你好，{name}。你{age}岁。</p>
    </>
  );
}
```

```css
button { display: block; margin-top: 10px; }
```

</Sandpack>

<Solution />

</Recipes>

一些 API 页面还包含 [故障排查](/reference/react/useEffect#troubleshooting)（用于常见问题）和 [替代方案](/reference/react-dom/findDOMNode#alternatives)（用于已弃用的 API）。

我们希望这种方式不仅能让 API 参考用于查找参数，还能帮助你了解针对任意给定 API 可以做的各种事情，以及它如何与其他 API 关联。

## 接下来是什么？ {/*whats-next*/}

这次简短的导览就到这里！请四处看看这个新网站，看看你喜欢什么、不喜欢什么，并继续在我们的 [问题跟踪器](https://github.com/reactjs/react.dev/issues) 中反馈。

我们承认这个项目花了很长时间才发布。我们希望保持 React 社区应得的高质量标准。在编写这些文档和创建所有示例时，我们发现了自己一些说明中的错误、React 中的 bug，甚至是 React 设计中的一些空白，而我们现在正努力解决这些问题。我们希望新的文档将帮助我们在未来对 React 本身提出更高的要求。

我们听到了许多你们希望扩展网站内容和功能的请求，例如：

- 为所有示例提供 TypeScript 版本；
- 创建更新后的性能、测试和可访问性指南；
- 将 React Server Components 与支持它们的框架分开进行文档编写；
- 与我们的国际社区合作，将新文档翻译出来；
- 为新网站添加缺失的功能（例如，这篇博客的 RSS）。

现在 [react.dev](https://react.dev/) 已经上线，我们将能够把重点从追赶第三方 React 教学资源，转移到添加新信息并进一步改进我们新的网站上。

我们认为，现在是学习 React 的最佳时机。

## 谁参与了这个项目？ {/*who-worked-on-this*/}

在 React 团队中，[Rachel Nabors](https://twitter.com/rachelnabors/)  ხელმძღვანed 这个项目（并提供了插图），而 [Dan Abramov](https://bsky.app/profile/danabra.mov) 设计了课程。他们也共同撰写了大部分内容。

当然，这么大的项目不可能孤立完成。我们要感谢很多人！

[Sylwia Vargas](https://twitter.com/SylwiaVargas) 彻底改造了我们的示例，让它们不再局限于“foo/bar/baz”和小猫，而是加入了来自世界各地的科学家、艺术家和城市。[Maggie Appleton](https://twitter.com/Mappletons) 将我们的涂鸦变成了一套清晰的图表系统。

感谢 [David McCabe](https://twitter.com/mcc_abe)、[Sophie Alpert](https://twitter.com/sophiebits)、[Rick Hanlon](https://twitter.com/rickhanlonii)、[Andrew Clark](https://twitter.com/acdlite) 和 [Matt Carroll](https://twitter.com/mattcarrollcode) 提供额外的写作贡献。我们也要感谢 [Natalia Tepluhina](https://twitter.com/n_tepluhina) 和 [Sebastian Markbåge](https://twitter.com/sebmarkbage) 提出的想法和反馈。

感谢 [Dan Lebowitz](https://twitter.com/lebo) 提供网站设计，以及 [Razvan Gradinar](https://dribbble.com/GradinarRazvan) 提供 sandbox 设计。

在开发方面，感谢 [Jared Palmer](https://twitter.com/jaredpalmer) 进行原型开发。感谢来自 [ThisDotLabs](https://www.thisdot.co/) 的 [Dane Grant](https://twitter.com/danecando) 和 [Dustin Goodman](https://twitter.com/dustinsgoodman) 对 UI 开发的支持。感谢来自 [CodeSandbox](https://codesandbox.io/) 的 [Ives van Hoorne](https://twitter.com/CompuIves)、[Alex Moldovan](https://twitter.com/alexnmoldovan)、[Jasper De Moor](https://twitter.com/JasperDeMoor) 和 [Danilo Woznica](https://twitter.com/danilowoz) 在 sandbox 集成方面的工作。感谢 [Rick Hanlon](https://twitter.com/rickhanlonii) 提供零散的开发和设计工作，打磨我们的配色和细节。感谢 [Harish Kumar](https://www.strek.in/) 和 [Luna Ruan](https://twitter.com/lunaruan) 为网站添加新功能并帮助维护。

非常感谢那些自愿抽出时间参与 alpha 和 beta 测试计划的人们。你们的热情和宝贵反馈帮助我们塑造了这些文档。特别要感谢我们的 beta 测试者 [Debbie O'Brien](https://twitter.com/debs_obrien)，她在 React Conf 2021 上分享了她使用 React 文档的体验。

最后，感谢 React 社区成为这项工作的灵感来源。正是因为你们，我们才在做这件事，我们也希望新的文档能帮助你们使用 React 构建任何你想要的用户界面。
