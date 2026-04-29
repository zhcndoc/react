---
title: 保留和重置状态
---

<Intro>

状态在组件之间是隔离的。React 会根据它们在 UI 树中的位置来跟踪哪个状态属于哪个组件。你可以控制何时在重新渲染之间保留状态，以及何时重置状态。

</Intro>

<YouWillLearn>

* 什么时候 React 会选择保留或重置状态
* 如何强制 React 重置组件的状态
* 键和类型如何影响状态是否被保留

</YouWillLearn>

## 状态与渲染树中的位置相关 {/*state-is-tied-to-a-position-in-the-tree*/}

React 会为你 UI 中的组件结构构建 [渲染树](learn/understanding-your-ui-as-a-tree#the-render-tree)。

当你给组件添加状态时，你可能会认为状态“存在”于组件内部。但实际上，状态是保存在 React 里面的。React 会根据组件在渲染树中的位置，将它持有的每一份状态与正确的组件关联起来。

这里，只有一个 `<Counter />` JSX 标签，但它在两个不同的位置被渲染：

<Sandpack>

```js
import { useState } from 'react';

export default function App() {
  const counter = <Counter />;
  return (
    <div>
      {counter}
      {counter}
    </div>
  );
}

function Counter() {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = 'counter';
  if (hover) {
    className += ' hover';
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>{score}</h1>
      <button onClick={() => setScore(score + 1)}>
        加一
      </button>
    </div>
  );
}
```

```css
label {
  display: block;
  clear: both;
}

.counter {
  width: 100px;
  text-align: center;
  border: 1px solid gray;
  border-radius: 4px;
  padding: 20px;
  margin: 0 20px 20px 0;
  float: left;
}

.hover {
  background: #ffffd8;
}
```

</Sandpack>

它们在树中的样子如下：

<DiagramGroup>

<Diagram name="preserving_state_tree" height={248} width={395} alt="React 组件树的示意图。根节点标记为 'div'，并有两个子节点。每个子节点都标记为 'Counter'，并且都包含一个标记为 'count'、值为 0 的状态气泡。">

React 树

</Diagram>

</DiagramGroup>

**这实际上是两个独立的计数器，因为它们各自渲染在树中的不同位置。** 你通常不需要为了使用 React 而考虑这些位置，但理解它的工作方式会很有帮助。

在 React 中，屏幕上的每个组件都拥有完全隔离的状态。例如，如果你并排渲染两个 `Counter` 组件，它们各自都会拥有独立的 `score` 和 `hover` 状态。

试着点击这两个计数器，注意它们不会互相影响：

<Sandpack>

```js
import { useState } from 'react';

export default function App() {
  return (
    <div>
      <Counter />
      <Counter />
    </div>
  );
}

function Counter() {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = 'counter';
  if (hover) {
    className += ' hover';
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>{score}</h1>
      <button onClick={() => setScore(score + 1)}>
        加一
      </button>
    </div>
  );
}
```

```css
.counter {
  width: 100px;
  text-align: center;
  border: 1px solid gray;
  border-radius: 4px;
  padding: 20px;
  margin: 0 20px 20px 0;
  float: left;
}

.hover {
  background: #ffffd8;
}
```

</Sandpack>

如你所见，当一个计数器更新时，只有那个组件的状态会被更新：


<DiagramGroup>

<Diagram name="preserving_state_increment" height={248} width={441} alt="React 组件树的示意图。根节点标记为 'div'，并有两个子节点。左侧子节点标记为 'Counter'，并包含一个标记为 'count'、值为 0 的状态气泡。右侧子节点标记为 'Counter'，并包含一个标记为 'count'、值为 1 的状态气泡。右侧子节点的状态气泡以黄色高亮，表示它的值已更新。">

更新状态

</Diagram>

</DiagramGroup>


只要你持续在树中的同一位置渲染同一个组件，React 就会一直保留该状态。要看到这一点，请把两个计数器都加到某个数值，然后通过取消勾选“渲染第二个计数器”复选框来移除第二个组件，再重新勾选它把它加回来：

<Sandpack>

```js
import { useState } from 'react';

export default function App() {
  const [showB, setShowB] = useState(true);
  return (
    <div>
      <Counter />
      {showB && <Counter />}
      <label>
        <input
          type="checkbox"
          checked={showB}
          onChange={e => {
            setShowB(e.target.checked)
          }}
        />
        渲染第二个计数器
      </label>
    </div>
  );
}

function Counter() {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = 'counter';
  if (hover) {
    className += ' hover';
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>{score}</h1>
      <button onClick={() => setScore(score + 1)}>
        加一
      </button>
    </div>
  );
}
```

```css
label {
  display: block;
  clear: both;
}

.counter {
  width: 100px;
  text-align: center;
  border: 1px solid gray;
  border-radius: 4px;
  padding: 20px;
  margin: 0 20px 20px 0;
  float: left;
}

.hover {
  background: #ffffd8;
}
```

</Sandpack>

注意，当你停止渲染第二个计数器的那一刻，它的状态会完全消失。这是因为当 React 移除一个组件时，它会销毁其状态。

<DiagramGroup>

<Diagram name="preserving_state_remove_component" height={253} width={422} alt="React 组件树的示意图。根节点标记为 'div'，并有两个子节点。左侧子节点标记为 'Counter'，并包含一个标记为 'count'、值为 0 的状态气泡。右侧子节点缺失，原位置是一个黄色的 'poof' 图像，强调该组件正从树中被删除。">

删除组件

</Diagram>

</DiagramGroup>

当你勾选“渲染第二个计数器”时，第二个 `Counter` 及其状态会从头初始化（`score = 0`），并添加到 DOM 中。

<DiagramGroup>

<Diagram name="preserving_state_add_component" height={258} width={500} alt="React 组件树的示意图。根节点标记为 'div'，并有两个子节点。左侧子节点标记为 'Counter'，并包含一个标记为 'count'、值为 0 的状态气泡。右侧子节点标记为 'Counter'，并包含一个标记为 'count'、值为 0 的状态气泡。整个右侧子节点以黄色高亮，表示它刚刚被添加到树中。">

添加组件

</Diagram>

</DiagramGroup>

**只要组件持续在 UI 树中的同一位置被渲染，React 就会保留它的状态。** 如果它被移除，或者同一位置渲染了另一个不同的组件，React 就会丢弃它的状态。

## 同一个位置上的同一个组件会保留状态 {/*same-component-at-the-same-position-preserves-state*/}

在这个示例中，有两个不同的 `<Counter />` 标签：

<Sandpack>

```js
import { useState } from 'react';

export default function App() {
  const [isFancy, setIsFancy] = useState(false);
  return (
    <div>
      {isFancy ? (
        <Counter isFancy={true} />
      ) : (
        <Counter isFancy={false} />
      )}
      <label>
        <input
          type="checkbox"
          checked={isFancy}
          onChange={e => {
            setIsFancy(e.target.checked)
          }}
        />
        使用精美样式
      </label>
    </div>
  );
}

function Counter({ isFancy }) {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = 'counter';
  if (hover) {
    className += ' hover';
  }
  if (isFancy) {
    className += ' fancy';
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>{score}</h1>
      <button onClick={() => setScore(score + 1)}>
        加一
      </button>
    </div>
  );
}
```

```css
label {
  display: block;
  clear: both;
}

.counter {
  width: 100px;
  text-align: center;
  border: 1px solid gray;
  border-radius: 4px;
  padding: 20px;
  margin: 0 20px 20px 0;
  float: left;
}

.fancy {
  border: 5px solid gold;
  color: #ff6767;
}

.hover {
  background: #ffffd8;
}
```

</Sandpack>

当你勾选或取消勾选复选框时，计数器状态不会被重置。无论 `isFancy` 是 `true` 还是 `false`，你始终都会在根 `App` 组件返回的 `div` 的第一个子元素位置上拥有一个 `<Counter />`：

<DiagramGroup>

<Diagram name="preserving_state_same_component" height={461} width={600} alt="一个由箭头连接、分成两部分的示意图。每一部分都包含一个组件布局：标记为 'App' 的父组件包含一个标记为 isFancy 的状态气泡。这个组件有一个标记为 'div' 的子组件，它指向一个包含 isFancy（以紫色高亮）的属性气泡，传递给唯一的子组件。最后一个子组件标记为 'Counter'，并且在两个图中都包含一个标记为 'count'、值为 3 的状态气泡。在示意图左侧，未高亮，isFancy 的父状态值为 false。在右侧，isFancy 的父状态值变为 true 并以黄色高亮，其下方的属性气泡也同样被高亮，并且其 isFancy 值也变为 true。">

更新 `App` 的状态不会重置 `Counter`，因为 `Counter` 仍然处于同一位置

</Diagram>

</DiagramGroup>


它是同一个位置上的同一个组件，所以从 React 的角度来看，它就是同一个计数器。

<Pitfall>

记住，对 React 来说，**重要的是在 UI 树中的位置，而不是 JSX 标记中的位置！** 这个组件在 `if` 内外有两个不同的 `return` 子句，其中包含不同的 `<Counter />` JSX 标签：

<Sandpack>

```js
import { useState } from 'react';

export default function App() {
  const [isFancy, setIsFancy] = useState(false);
  if (isFancy) {
    return (
      <div>
        <Counter isFancy={true} />
        <label>
          <input
            type="checkbox"
            checked={isFancy}
            onChange={e => {
              setIsFancy(e.target.checked)
            }}
          />
          使用精美样式
        </label>
      </div>
    );
  }
  return (
    <div>
      <Counter isFancy={false} />
      <label>
        <input
          type="checkbox"
          checked={isFancy}
          onChange={e => {
            setIsFancy(e.target.checked)
          }}
        />
        使用精美样式
      </label>
    </div>
  );
}

function Counter({ isFancy }) {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = 'counter';
  if (hover) {
    className += ' hover';
  }
  if (isFancy) {
    className += ' fancy';
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>{score}</h1>
      <button onClick={() => setScore(score + 1)}>
        加一
      </button>
    </div>
  );
}
```

```css
label {
  display: block;
  clear: both;
}

.counter {
  width: 100px;
  text-align: center;
  border: 1px solid gray;
  border-radius: 4px;
  padding: 20px;
  margin: 0 20px 20px 0;
  float: left;
}

.fancy {
  border: 5px solid gold;
  color: #ff6767;
}

.hover {
  background: #ffffd8;
}
```

</Sandpack>

你可能会以为勾选复选框时状态会重置，但实际上不会！这是因为**这两个 `<Counter />` 标签都渲染在同一个位置。** React 并不知道你在函数中把条件写在了哪里。它“看到”的只有你返回的那棵树。

在这两种情况下，`App` 组件都返回了一个以 `<Counter />` 作为第一个子元素的 `<div>`。对 React 来说，这两个计数器拥有相同的“地址”：根节点的第一个子元素中的第一个子元素。这就是 React 在前后两次渲染之间对它们进行匹配的方式，不管你的逻辑结构是怎样组织的。

</Pitfall>

## 位于同一位置的不同组件会重置状态 {/*different-components-at-the-same-position-reset-state*/}

在这个示例中，勾选复选框会把 `<Counter>` 替换成一个 `<p>`：

<Sandpack>

```js
import { useState } from 'react';

export default function App() {
  const [isPaused, setIsPaused] = useState(false);
  return (
    <div>
      {isPaused ? (
        <p>待会儿见！</p>
      ) : (
        <Counter />
      )}
      <label>
        <input
          type="checkbox"
          checked={isPaused}
          onChange={e => {
            setIsPaused(e.target.checked)
          }}
        />
        休息一下
      </label>
    </div>
  );
}

function Counter() {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = 'counter';
  if (hover) {
    className += ' hover';
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>{score}</h1>
      <button onClick={() => setScore(score + 1)}>
        加 1
      </button>
    </div>
  );
}
```

```css
label {
  display: block;
  clear: both;
}

.counter {
  width: 100px;
  text-align: center;
  border: 1px solid gray;
  border-radius: 4px;
  padding: 20px;
  margin: 0 20px 20px 0;
  float: left;
}

.hover {
  background: #ffffd8;
}
```

</Sandpack>

在这里，你在同一个位置切换了_不同_的组件类型。最初，`<div>` 的第一个子元素是一个 `Counter`。但当你换成 `p` 时，React 会从 UI 树中移除 `Counter` 并销毁它的状态。

<DiagramGroup>

<Diagram name="preserving_state_diff_pt1" height={290} width={753} alt="Diagram with three sections, with an arrow transitioning each section in between. The first section contains a React component labeled 'div' with a single child labeled 'Counter' containing a state bubble labeled 'count' with value 3. The middle section has the same 'div' parent, but the child component has now been deleted, indicated by a yellow 'proof' image. The third section has the same 'div' parent again, now with a new child labeled 'p', highlighted in yellow.">

当 `Counter` 变成 `p` 时，`Counter` 被删除，`p` 被添加

</Diagram>

</DiagramGroup>

<DiagramGroup>

<Diagram name="preserving_state_diff_pt2" height={290} width={753} alt="Diagram with three sections, with an arrow transitioning each section in between. The first section contains a React component labeled 'p'. The middle section has the same 'div' parent, but the child component has now been deleted, indicated by a yellow 'proof' image. The third section has the same 'div' parent again, now with a new child labeled 'Counter' containing a state bubble labeled 'count' with value 0, highlighted in yellow.">

切换回来时，`p` 被删除，`Counter` 被添加

</Diagram>

</DiagramGroup>

同样，**当你在同一位置渲染不同的组件时，它会重置其整个子树的状态。** 想看看这是怎么工作的，可以先增加计数器，然后勾选复选框：

<Sandpack>

```js
import { useState } from 'react';

export default function App() {
  const [isFancy, setIsFancy] = useState(false);
  return (
    <div>
      {isFancy ? (
        <div>
          <Counter isFancy={true} />
        </div>
      ) : (
        <section>
          <Counter isFancy={false} />
        </section>
      )}
      <label>
        <input
          type="checkbox"
          checked={isFancy}
          onChange={e => {
            setIsFancy(e.target.checked)
          }}
        />
        使用花哨样式
      </label>
    </div>
  );
}

function Counter({ isFancy }) {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = 'counter';
  if (hover) {
    className += ' hover';
  }
  if (isFancy) {
    className += ' fancy';
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>{score}</h1>
      <button onClick={() => setScore(score + 1)}>
        加 1
      </button>
    </div>
  );
}
```

```css
label {
  display: block;
  clear: both;
}

.counter {
  width: 100px;
  text-align: center;
  border: 1px solid gray;
  border-radius: 4px;
  padding: 20px;
  margin: 0 20px 20px 0;
  float: left;
}

.fancy {
  border: 5px solid gold;
  color: #ff6767;
}

.hover {
  background: #ffffd8;
}
```

</Sandpack>

当你点击复选框时，计数器状态会被重置。虽然你渲染了一个 `Counter`，但 `div` 的第一个子元素从 `section` 变成了 `div`。当子元素 `section` 从 DOM 中移除时，其下方整棵树（包括 `Counter` 及其状态）也被销毁了。

<DiagramGroup>

<Diagram name="preserving_state_diff_same_pt1" height={350} width={794} alt="Diagram with three sections, with an arrow transitioning each section in between. The first section contains a React component labeled 'div' with a single child labeled 'section', which has a single child labeled 'Counter' containing a state bubble labeled 'count' with value 3. The middle section has the same 'div' parent, but the child components have now been deleted, indicated by a yellow 'proof' image. The third section has the same 'div' parent again, now with a new child labeled 'div', highlighted in yellow, also with a new child labeled 'Counter' containing a state bubble labeled 'count' with value 0, all highlighted in yellow.">

当 `section` 变成 `div` 时，`section` 被删除，新的 `div` 被添加

</Diagram>

</DiagramGroup>

<DiagramGroup>

<Diagram name="preserving_state_diff_same_pt2" height={350} width={794} alt="Diagram with three sections, with an arrow transitioning each section in between. The first section contains a React component labeled 'div' with a single child labeled 'div', which has a single child labeled 'Counter' containing a state bubble labeled 'count' with value 0. The middle section has the same 'div' parent, but the child components have now been deleted, indicated by a yellow 'proof' image. The third section has the same 'div' parent again, now with a new child labeled 'section', highlighted in yellow, also with a new child labeled 'Counter' containing a state bubble labeled 'count' with value 0, all highlighted in yellow.">

切换回来时，`div` 被删除，新的 `section` 被添加

</Diagram>

</DiagramGroup>

一般来说，**如果你想在重新渲染之间保留状态，树的结构就需要在前后两次渲染中“对上”**。如果结构不同，状态就会被销毁，因为 React 会在从树中移除组件时销毁状态。

<Pitfall>

这就是为什么你不应该嵌套定义组件函数。

在这里，`MyTextField` 组件函数是定义在 `MyComponent` *内部* 的：

<Sandpack>

```js {expectedErrors: {'react-compiler': [7]}}
import { useState } from 'react';

export default function MyComponent() {
  const [counter, setCounter] = useState(0);

  function MyTextField() {
    const [text, setText] = useState('');

    return (
      <input
        value={text}
        onChange={e => setText(e.target.value)}
      />
    );
  }

  return (
    <>
      <MyTextField />
      <button onClick={() => {
        setCounter(counter + 1)
      }}>已点击 {counter} 次</button>
    </>
  );
}
```

</Sandpack>

每次你点击按钮，输入框状态都会消失！这是因为 `MyComponent` 每次渲染都会创建一个*不同*的 `MyTextField` 函数。你在同一个位置渲染了一个*不同*的组件，所以 React 会重置其下方的所有状态。这会导致 bug 和性能问题。为避免这个问题，**始终在顶层声明组件函数，不要嵌套定义它们。**

</Pitfall>

## 在同一位置重置状态 {/*resetting-state-at-the-same-position*/}

默认情况下，只要 React 保持某个组件处于同一位置，就会保留它的状态。通常这正是你想要的，所以它作为默认行为是合理的。但有时，你可能想要重置组件的状态。考虑这个应用，它允许两个玩家在每一回合中记录自己的分数：

<Sandpack>

```js
import { useState } from 'react';

export default function Scoreboard() {
  const [isPlayerA, setIsPlayerA] = useState(true);
  return (
    <div>
      {isPlayerA ? (
        <Counter person="Taylor" />
      ) : (
        <Counter person="Sarah" />
      )}
      <button onClick={() => {
        setIsPlayerA(!isPlayerA);
      }}>
        下一位玩家！
      </button>
    </div>
  );
}

function Counter({ person }) {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = 'counter';
  if (hover) {
    className += ' hover';
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>{person} 的分数：{score}</h1>
      <button onClick={() => setScore(score + 1)}>
        加 1
      </button>
    </div>
  );
}
```

```css
h1 {
  font-size: 18px;
}

.counter {
  width: 100px;
  text-align: center;
  border: 1px solid gray;
  border-radius: 4px;
  padding: 20px;
  margin: 0 20px 20px 0;
}

.hover {
  background: #ffffd8;
}
```

</Sandpack>

目前，当你切换玩家时，分数会被保留。两个 `Counter` 出现在同一位置，所以 React 把它们视为*同一个* `Counter`，只是它的 `person` prop 改变了。

但从概念上说，在这个应用里它们应该是两个独立的计数器。它们可能在 UI 中出现在同一个位置，但一个是 Taylor 的计数器，另一个是 Sarah 的计数器。

切换它们时重置状态有两种方法：

1. 将组件渲染在不同的位置
2. 通过 `key` 为每个组件赋予明确的身份


### 选项 1：将组件渲染在不同的位置 {/*option-1-rendering-a-component-in-different-positions*/}

如果你希望这两个 `Counter` 相互独立，可以把它们渲染在两个不同的位置：

<Sandpack>

```js
import { useState } from 'react';

export default function Scoreboard() {
  const [isPlayerA, setIsPlayerA] = useState(true);
  return (
    <div>
      {isPlayerA &&
        <Counter person="Taylor" />
      }
      {!isPlayerA &&
        <Counter person="Sarah" />
      }
      <button onClick={() => {
        setIsPlayerA(!isPlayerA);
      }}>
        下一位玩家！
      </button>
    </div>
  );
}

function Counter({ person }) {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = 'counter';
  if (hover) {
    className += ' hover';
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>{person} 的分数：{score}</h1>
      <button onClick={() => setScore(score + 1)}>
        加 1
      </button>
    </div>
  );
}
```

```css
h1 {
  font-size: 18px;
}

.counter {
  width: 100px;
  text-align: center;
  border: 1px solid gray;
  border-radius: 4px;
  padding: 20px;
  margin: 0 20px 20px 0;
}

.hover {
  background: #ffffd8;
}
```

</Sandpack>

* 最初，`isPlayerA` 为 `true`。所以第一个位置包含 `Counter` 状态，第二个位置为空。
* 当你点击“下一位玩家”按钮时，第一个位置被清空，但第二个位置现在包含一个 `Counter`。

<DiagramGroup>

<Diagram name="preserving_state_diff_position_p1" height={375} width={504} alt="Diagram with a tree of React components. The parent is labeled 'Scoreboard' with a state bubble labeled isPlayerA with value 'true'. The only child, arranged to the left, is labeled Counter with a state bubble labeled 'count' and value 0. All of the left child is highlighted in yellow, indicating it was added.">

初始状态

</Diagram>

<Diagram name="preserving_state_diff_position_p2" height={375} width={504} alt="Diagram with a tree of React components. The parent is labeled 'Scoreboard' with a state bubble labeled isPlayerA with value 'false'. The state bubble is highlighted in yellow, indicating that it has changed. The left child is replaced with a yellow 'poof' image indicating that it has been deleted and there is a new child on the right, highlighted in yellow indicating that it was added. The new child is labeled 'Counter' and contains a state bubble labeled 'count' with value 0.">

点击“next”

</Diagram>

<Diagram name="preserving_state_diff_position_p3" height={375} width={504} alt="Diagram with a tree of React components. The parent is labeled 'Scoreboard' with a state bubble labeled isPlayerA with value 'true'. The state bubble is highlighted in yellow, indicating that it has changed. There is a new child on the left, highlighted in yellow indicating that it was added. The new child is labeled 'Counter' and contains a state bubble labeled 'count' with value 0. The right child is replaced with a yellow 'poof' image indicating that it has been deleted.">

再次点击“next”

</Diagram>

</DiagramGroup>

每个 `Counter` 在从 DOM 中移除时，其状态都会被销毁。这就是为什么每次点击按钮时它们都会重置。

当你只有少量在同一位置渲染的独立组件时，这个方案很方便。在这个例子里你只有两个，所以在 JSX 中分别渲染它们并不麻烦。

### 选项 2：使用 key 重置状态 {/*option-2-resetting-state-with-a-key*/}

还有另一种更通用的方法，可以用来重置组件状态。

你可能在[渲染列表](/learn/rendering-lists#keeping-list-items-in-order-with-key)时见过 `key`。`key` 不只是用于列表！你可以用 `key` 让 React 区分任意组件。默认情况下，React 使用父组件中的顺序（“第一个计数器”“第二个计数器”）来区分组件。但 `key` 可以让你告诉 React，这不只是一个*第一个*计数器或*第二个*计数器，而是一个特定的计数器——例如，*Taylor 的*计数器。这样，无论 Taylor 的计数器出现在树中的哪个位置，React 都能识别出来！

在这个例子中，两个 `<Counter />` 即使在 JSX 中出现在同一个位置，也不会共享状态：

<Sandpack>

```js
import { useState } from 'react';

export default function Scoreboard() {
  const [isPlayerA, setIsPlayerA] = useState(true);
  return (
    <div>
      {isPlayerA ? (
        <Counter key="Taylor" person="Taylor" />
      ) : (
        <Counter key="Sarah" person="Sarah" />
      )}
      <button onClick={() => {
        setIsPlayerA(!isPlayerA);
      }}>
        下一位玩家！
      </button>
    </div>
  );
}

function Counter({ person }) {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = 'counter';
  if (hover) {
    className += ' hover';
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>{person} 的分数：{score}</h1>
      <button onClick={() => setScore(score + 1)}>
        加 1
      </button>
    </div>
  );
}
```

```css
h1 {
  font-size: 18px;
}

.counter {
  width: 100px;
  text-align: center;
  border: 1px solid gray;
  border-radius: 4px;
  padding: 20px;
  margin: 0 20px 20px 0;
}

.hover {
  background: #ffffd8;
}
```

</Sandpack>

在 Taylor 和 Sarah 之间切换不会保留状态。这是因为**你给了它们不同的 `key`：**

```js
{isPlayerA ? (
  <Counter key="Taylor" person="Taylor" />
) : (
  <Counter key="Sarah" person="Sarah" />
)}
```

指定 `key` 会告诉 React 将 `key` 本身作为位置的一部分，而不是父组件中的顺序。这就是为什么，即使你在 JSX 中把它们渲染在同一个位置，React 也会把它们视为两个不同的计数器，因此它们永远不会共享状态。每次计数器出现在屏幕上时，都会创建它的状态。每次它被移除时，都会销毁它的状态。在它们之间切换会反复重置状态。

<Note>

请记住，`key` 不是全局唯一的。它们只指定*父组件内部*的位置。

</Note>

### 使用 key 重置表单 {/*resetting-a-form-with-a-key*/}

使用 key 重置状态在处理表单时特别有用。

在这个聊天应用中，`<Chat>` 组件包含文本输入状态：

<Sandpack>

```js src/App.js
import { useState } from 'react';
import Chat from './Chat.js';
import ContactList from './ContactList.js';

export default function Messenger() {
  const [to, setTo] = useState(contacts[0]);
  return (
    <div>
      <ContactList
        contacts={contacts}
        selectedContact={to}
        onSelect={contact => setTo(contact)}
      />
      <Chat contact={to} />
    </div>
  )
}

const contacts = [
  { id: 0, name: 'Taylor', email: 'taylor@mail.com' },
  { id: 1, name: 'Alice', email: 'alice@mail.com' },
  { id: 2, name: 'Bob', email: 'bob@mail.com' }
];
```

```js src/ContactList.js
export default function ContactList({
  selectedContact,
  contacts,
  onSelect
}) {
  return (
    <section className="contact-list">
      <ul>
        {contacts.map(contact =>
          <li key={contact.id}>
            <button onClick={() => {
              onSelect(contact);
            }}>
              {contact.name}
            </button>
          </li>
        )}
      </ul>
    </section>
  );
}
```

```js src/Chat.js
import { useState } from 'react';

export default function Chat({ contact }) {
  const [text, setText] = useState('');
  return (
    <section className="chat">
      <textarea
        value={text}
        placeholder={'与 ' + contact.name 聊天'}
        onChange={e => setText(e.target.value)}
      />
      <br />
      <button>发送给 {contact.email}</button>
    </section>
  );
}
```

```css
.chat, .contact-list {
  float: left;
  margin-bottom: 20px;
}
ul, li {
  list-style: none;
  margin: 0;
  padding: 0;
}
li button {
  width: 100px;
  padding: 10px;
  margin-right: 10px;
}
textarea {
  height: 150px;
}
```

</Sandpack>

试着在输入框里输入一些内容，然后点击 “Alice” 或 “Bob” 来选择不同的收件人。你会注意到输入框状态被保留了，因为 `<Chat>` 被渲染在树中的同一位置。

**在很多应用中，这可能是期望的行为，但在聊天应用里就不是！** 你不希望用户因为误点而把已经输入的消息发给错误的人。要修复这个问题，添加一个 `key`：

```js
<Chat key={to.id} contact={to} />
```

这样可以确保当你选择不同的收件人时，`Chat` 组件会从头重新创建，包括其下方树中的任何状态。React 也会重新创建 DOM 元素，而不是复用它们。

现在切换收件人时总会清空文本框：

<Sandpack>

```js src/App.js
import { useState } from 'react';
import Chat from './Chat.js';
import ContactList from './ContactList.js';

export default function Messenger() {
  const [to, setTo] = useState(contacts[0]);
  return (
    <div>
      <ContactList
        contacts={contacts}
        selectedContact={to}
        onSelect={contact => setTo(contact)}
      />
      <Chat key={to.id} contact={to} />
    </div>
  )
}

const contacts = [
  { id: 0, name: 'Taylor', email: 'taylor@mail.com' },
  { id: 1, name: 'Alice', email: 'alice@mail.com' },
  { id: 2, name: 'Bob', email: 'bob@mail.com' }
];
```

```js src/ContactList.js
export default function ContactList({
  selectedContact,
  contacts,
  onSelect
}) {
  return (
    <section className="contact-list">
      <ul>
        {contacts.map(contact =>
          <li key={contact.id}>
            <button onClick={() => {
              onSelect(contact);
            }}>
              {contact.name}
            </button>
          </li>
        )}
      </ul>
    </section>
  );
}
```

```js src/Chat.js
import { useState } from 'react';

export default function Chat({ contact }) {
  const [text, setText] = useState('');
  return (
    <section className="chat">
      <textarea
        value={text}
        placeholder={'与 ' + contact.name 聊天'}
        onChange={e => setText(e.target.value)}
      />
      <br />
      <button>发送给 {contact.email}</button>
    </section>
  );
}
```

```css
.chat, .contact-list {
  float: left;
  margin-bottom: 20px;
}
ul, li {
  list-style: none;
  margin: 0;
  padding: 0;
}
li button {
  width: 100px;
  padding: 10px;
  margin-right: 10px;
}
textarea {
  height: 150px;
}
```

</Sandpack>

<DeepDive>

#### 为被移除的组件保留状态 {/*preserving-state-for-removed-components*/}

在真实的聊天应用中，当用户再次选择之前的收件人时，你大概希望恢复输入框状态。要让一个不再可见的组件的状态继续“存活”，有几种方法：

- 你可以渲染 _所有_ 聊天内容，而不是只渲染当前聊天，同时用 CSS 把其他的都隐藏起来。这样聊天项不会从树中移除，因此它们的本地状态会被保留。这个方案对简单界面很有效。但如果隐藏的树很大并且包含很多 DOM 节点，它会变得很慢。
- 你可以[将状态提升](/learn/sharing-state-between-components)到父组件中，在父组件里保存每个收件人的待发消息。这样，当子组件被移除时就没关系了，因为真正保存关键信息的是父组件。这是最常见的解决方案。
- 你也可以在 React state 之外再使用别的数据源。例如，即使用户不小心关闭了页面，你可能仍然希望消息草稿被保留。为实现这一点，你可以让 `Chat` 组件通过读取 [`localStorage`](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage) 来初始化状态，并把草稿也保存到那里。

不管你选择哪种策略，给定 _Alice_ 的聊天与给定 _Bob_ 的聊天在概念上都是不同的，因此根据当前收件人给 `<Chat>` 树设置一个 `key` 是合理的。

</DeepDive>

<Recap>

- 只要同一个组件在同一位置被渲染，React 就会保留它的状态。
- 状态并不保存在 JSX 标签中。它与放入该 JSX 的树位置相关联。
- 你可以通过给子树不同的 `key` 来强制重置其状态。
- 不要嵌套定义组件，否则你会不小心重置状态。

</Recap>



<Challenges>

#### 修复消失的输入文本 {/*fix-disappearing-input-text*/}

这个示例在你点击按钮时会显示一条消息。然而，点击按钮也会意外地重置输入框。为什么会这样？修复它，使点击按钮不会重置输入文本。

<Sandpack>

```js src/App.js
import { useState } from 'react';

export default function App() {
  const [showHint, setShowHint] = useState(false);
  if (showHint) {
    return (
      <div>
        <p><i>提示：你最喜欢的城市？</i></p>
        <Form />
        <button onClick={() => {
          setShowHint(false);
        }}>隐藏提示</button>
      </div>
    );
  }
  return (
    <div>
      <Form />
      <button onClick={() => {
        setShowHint(true);
      }}>显示提示</button>
    </div>
  );
}

function Form() {
  const [text, setText] = useState('');
  return (
    <textarea
      value={text}
      onChange={e => setText(e.target.value)}
    />
  );
}
```

```css
textarea { display: block; margin: 10px 0; }
```

</Sandpack>

<Solution>

问题在于 `Form` 被渲染在不同的位置。在 `if` 分支中，它是 `<div>` 的第二个子元素；但在 `else` 分支中，它是第一个子元素。因此，每个位置上的组件类型都发生了变化。第一个位置在 `p` 和 `Form` 之间切换，而第二个位置在 `Form` 和 `button` 之间切换。每次组件类型变化时，React 都会重置状态。

最简单的解决方案是统一分支，让 `Form` 始终渲染在同一位置：

<Sandpack>

```js src/App.js
import { useState } from 'react';

export default function App() {
  const [showHint, setShowHint] = useState(false);
  return (
    <div>
      {showHint &&
        <p><i>提示：你最喜欢的城市？</i></p>
      }
      <Form />
      {showHint ? (
        <button onClick={() => {
          setShowHint(false);
        }}>隐藏提示</button>
      ) : (
        <button onClick={() => {
          setShowHint(true);
        }}>显示提示</button>
      )}
    </div>
  );
}

function Form() {
  const [text, setText] = useState('');
  return (
    <textarea
      value={text}
      onChange={e => setText(e.target.value)}
    />
  );
}
```

```css
textarea { display: block; margin: 10px 0; }
```

</Sandpack>


从技术上讲，你也可以在 `else` 分支中在 `<Form />` 前面添加 `null`，以匹配 `if` 分支的结构：

<Sandpack>

```js src/App.js
import { useState } from 'react';

export default function App() {
  const [showHint, setShowHint] = useState(false);
  if (showHint) {
    return (
      <div>
        <p><i>提示：你最喜欢的城市？</i></p>
        <Form />
        <button onClick={() => {
          setShowHint(false);
        }}>隐藏提示</button>
      </div>
    );
  }
  return (
    <div>
      {null}
      <Form />
      <button onClick={() => {
        setShowHint(true);
      }}>显示提示</button>
    </div>
  );
}

function Form() {
  const [text, setText] = useState('');
  return (
    <textarea
      value={text}
      onChange={e => setText(e.target.value)}
    />
  );
}
```

```css
textarea { display: block; margin: 10px 0; }
```

</Sandpack>

这样一来，`Form` 始终是第二个子元素，因此它会保持在同一位置并保留状态。但这种方法不太直观，而且有可能被别人把那个 `null` 删掉。

</Solution>

#### 交换两个表单字段 {/*swap-two-form-fields*/}

这个表单让你输入姓和名。它还有一个复选框，用来控制哪个字段在前面。当你勾选复选框时，“姓”字段会出现在“名”字段前面。

它几乎能工作，但有个 bug。如果你在“名”输入框中填入内容，然后勾选复选框，文本会留在第一个输入框里（现在这个输入框是“姓”）。修复它，使得当你反转顺序时，输入文本也会一起移动。

<Hint>

看起来对这些字段来说，它们在父组件中的位置还不够。有没有办法告诉 React 如何在重新渲染之间匹配状态？

</Hint>

<Sandpack>

```js src/App.js
import { useState } from 'react';

export default function App() {
  const [reverse, setReverse] = useState(false);
  let checkbox = (
    <label>
      <input
        type="checkbox"
        checked={reverse}
        onChange={e => setReverse(e.target.checked)}
      />
      反转顺序
    </label>
  );
  if (reverse) {
    return (
      <>
        <Field label="姓" />
        <Field label="名" />
        {checkbox}
      </>
    );
  } else {
    return (
      <>
        <Field label="名" />
        <Field label="姓" />
        {checkbox}
      </>
    );
  }
}

function Field({ label }) {
  const [text, setText] = useState('');
  return (
    <label>
      {label}:{' '}
      <input
        type="text"
        value={text}
        placeholder={label}
        onChange={e => setText(e.target.value)}
      />
    </label>
  );
}
```

```css
label { display: block; margin: 10px 0; }
```

</Sandpack>

<Solution>

在 `if` 和 `else` 分支中都给两个 `<Field>` 组件添加 `key`。这会告诉 React 即使它们在父组件中的顺序改变了，也该如何为任意一个 `<Field>`“匹配”正确的状态：

<Sandpack>

```js src/App.js
import { useState } from 'react';

export default function App() {
  const [reverse, setReverse] = useState(false);
  let checkbox = (
    <label>
      <input
        type="checkbox"
        checked={reverse}
        onChange={e => setReverse(e.target.checked)}
      />
      反转顺序
    </label>
  );
  if (reverse) {
    return (
      <>
        <Field key="lastName" label="姓" />
        <Field key="firstName" label="名" />
        {checkbox}
      </>
    );
  } else {
    return (
      <>
        <Field key="firstName" label="名" />
        <Field key="lastName" label="姓" />
        {checkbox}
      </>
    );
  }
}

function Field({ label }) {
  const [text, setText] = useState('');
  return (
    <label>
      {label}:{' '}
      <input
        type="text"
        value={text}
        placeholder={label}
        onChange={e => setText(e.target.value)}
      />
    </label>
  );
}
```

```css
label { display: block; margin: 10px 0; }
```

</Sandpack>

</Solution>

#### 重置详情表单 {/*reset-a-detail-form*/}

这是一个可编辑的联系人列表。你可以编辑所选联系人的详细信息，然后点击“Save”来更新它，或者点击“Reset”来撤销你的更改。

当你选择另一个联系人（例如 Alice）时，状态会更新，但表单仍然显示上一个联系人的详细信息。修复它，使得当所选联系人改变时，表单被重置。

<Sandpack>

```js src/App.js
import { useState } from 'react';
import ContactList from './ContactList.js';
import EditContact from './EditContact.js';

export default function ContactManager() {
  const [
    contacts,
    setContacts
  ] = useState(initialContacts);
  const [
    selectedId,
    setSelectedId
  ] = useState(0);
  const selectedContact = contacts.find(c =>
    c.id === selectedId
  );

  function handleSave(updatedData) {
    const nextContacts = contacts.map(c => {
      if (c.id === updatedData.id) {
        return updatedData;
      } else {
        return c;
      }
    });
    setContacts(nextContacts);
  }

  return (
    <div>
      <ContactList
        contacts={contacts}
        selectedId={selectedId}
        onSelect={id => setSelectedId(id)}
      />
      <hr />
      <EditContact
        initialData={selectedContact}
        onSave={handleSave}
      />
    </div>
  )
}

const initialContacts = [
  { id: 0, name: 'Taylor', email: 'taylor@mail.com' },
  { id: 1, name: 'Alice', email: 'alice@mail.com' },
  { id: 2, name: 'Bob', email: 'bob@mail.com' }
];
```

```js src/ContactList.js
export default function ContactList({
  contacts,
  selectedId,
  onSelect
}) {
  return (
    <section>
      <ul>
        {contacts.map(contact =>
          <li key={contact.id}>
            <button onClick={() => {
              onSelect(contact.id);
            }}>
              {contact.id === selectedId ?
                <b>{contact.name}</b> :
                contact.name
              }
            </button>
          </li>
        )}
      </ul>
    </section>
  );
}
```

```js src/EditContact.js
import { useState } from 'react';

export default function EditContact({ initialData, onSave }) {
  const [name, setName] = useState(initialData.name);
  const [email, setEmail] = useState(initialData.email);
  return (
    <section>
      <label>
        姓名：{' '}
        <input
          type="text"
          value={name}
          onChange={e => setName(e.target.value)}
        />
      </label>
      <label>
        邮箱：{' '}
        <input
          type="email"
          value={email}
          onChange={e => setEmail(e.target.value)}
        />
      </label>
      <button onClick={() => {
        const updatedData = {
          id: initialData.id,
          name: name,
          email: email
        };
        onSave(updatedData);
      }}>
        保存
      </button>
      <button onClick={() => {
        setName(initialData.name);
        setEmail(initialData.email);
      }}>
        重置
      </button>
    </section>
  );
}
```

```css
ul, li {
  list-style: none;
  margin: 0;
  padding: 0;
}
li { display: inline-block; }
li button {
  padding: 10px;
}
label {
  display: block;
  margin: 10px 0;
}
button {
  margin-right: 10px;
  margin-bottom: 10px;
}
```

</Sandpack>

<Solution>

给 `EditContact` 组件添加 `key={selectedId}`。这样，在不同联系人之间切换时就会重置表单：

<Sandpack>

```js src/App.js
import { useState } from 'react';
import ContactList from './ContactList.js';
import EditContact from './EditContact.js';

export default function ContactManager() {
  const [
    contacts,
    setContacts
  ] = useState(initialContacts);
  const [
    selectedId,
    setSelectedId
  ] = useState(0);
  const selectedContact = contacts.find(c =>
    c.id === selectedId
  );

  function handleSave(updatedData) {
    const nextContacts = contacts.map(c => {
      if (c.id === updatedData.id) {
        return updatedData;
      } else {
        return c;
      }
    });
    setContacts(nextContacts);
  }

  return (
    <div>
      <ContactList
        contacts={contacts}
        selectedId={selectedId}
        onSelect={id => setSelectedId(id)}
      />
      <hr />
      <EditContact
        key={selectedId}
        initialData={selectedContact}
        onSave={handleSave}
      />
    </div>
  )
}

const initialContacts = [
  { id: 0, name: 'Taylor', email: 'taylor@mail.com' },
  { id: 1, name: 'Alice', email: 'alice@mail.com' },
  { id: 2, name: 'Bob', email: 'bob@mail.com' }
];
```

```js src/ContactList.js
export default function ContactList({
  contacts,
  selectedId,
  onSelect
}) {
  return (
    <section>
      <ul>
        {contacts.map(contact =>
          <li key={contact.id}>
            <button onClick={() => {
              onSelect(contact.id);
            }}>
              {contact.id === selectedId ?
                <b>{contact.name}</b> :
                contact.name
              }
            </button>
          </li>
        )}
      </ul>
    </section>
  );
}
```

```js src/EditContact.js
import { useState } from 'react';

export default function EditContact({ initialData, onSave }) {
  const [name, setName] = useState(initialData.name);
  const [email, setEmail] = useState(initialData.email);
  return (
    <section>
      <label>
        姓名：{' '}
        <input
          type="text"
          value={name}
          onChange={e => setName(e.target.value)}
        />
      </label>
      <label>
        邮箱：{' '}
        <input
          type="email"
          value={email}
          onChange={e => setEmail(e.target.value)}
        />
      </label>
      <button onClick={() => {
        const updatedData = {
          id: initialData.id,
          name: name,
          email: email
        };
        onSave(updatedData);
      }}>
        保存
      </button>
      <button onClick={() => {
        setName(initialData.name);
        setEmail(initialData.email);
      }}>
        重置
      </button>
    </section>
  );
}
```

```css
ul, li {
  list-style: none;
  margin: 0;
  padding: 0;
}
li { display: inline-block; }
li button {
  padding: 10px;
}
label {
  display: block;
  margin: 10px 0;
}
button {
  margin-right: 10px;
  margin-bottom: 10px;
}
```

</Sandpack>

</Solution>

#### 图片加载时清空它 {/*clear-an-image-while-its-loading*/}

当你点击“Next”时，浏览器会开始加载下一张图片。然而，因为它显示在同一个 `<img>` 标签中，默认情况下你仍然会看到上一张图片，直到下一张加载完成。如果文本必须始终与图片匹配，这可能并不理想。请修改它，让你点击“Next”的那一刻，上一张图片立刻清空。

<Hint>

有没有办法告诉 React 重新创建 DOM，而不是复用它？

</Hint>

<Sandpack>

```js
import { useState } from 'react';

export default function Gallery() {
  const [index, setIndex] = useState(0);
  const hasNext = index < images.length - 1;

  function handleClick() {
    if (hasNext) {
      setIndex(index + 1);
    } else {
      setIndex(0);
    }
  }

  let image = images[index];
  return (
    <>
      <button onClick={handleClick}>
        下一张
      </button>
      <h3>
        第 {index + 1} 张，共 {images.length} 张
      </h3>
      <img src={image.src} />
      <p>
        {image.place}
      </p>
    </>
  );
}

let images = [{
  place: '马来西亚，槟城',
  src: 'https://react.dev/images/docs/scientists/FJeJR8M.jpg'
}, {
  place: '葡萄牙，里斯本',
  src: 'https://react.dev/images/docs/scientists/dB2LRbj.jpg'
}, {
  place: '西班牙，毕尔巴鄂',
  src: 'https://react.dev/images/docs/scientists/z08o2TS.jpg'
}, {
  place: '智利，瓦尔帕莱索',
  src: 'https://react.dev/images/docs/scientists/Y3utgTi.jpg'
}, {
  place: '瑞士，施维茨',
  src: 'https://react.dev/images/docs/scientists/JBbMpWY.jpg'
}, {
  place: '捷克，布拉格',
  src: 'https://react.dev/images/docs/scientists/QwUKKmF.jpg'
}, {
  place: '斯洛文尼亚，卢布尔雅那',
  src: 'https://react.dev/images/docs/scientists/3aIiwfm.jpg'
}];
```

```css
img { width: 150px; height: 150px; }
```

</Sandpack>

<Solution>

你可以给 `<img>` 标签提供一个 `key`。当这个 `key` 改变时，React 会从头重新创建 `<img>` DOM 节点。这会在每张图片加载时导致短暂闪烁，所以你不会想在应用中的每张图片都这样做。但如果你想确保图片始终与文本匹配，这么做是合理的。

<Sandpack>

```js
import { useState } from 'react';

export default function Gallery() {
  const [index, setIndex] = useState(0);
  const hasNext = index < images.length - 1;

  function handleClick() {
    if (hasNext) {
      setIndex(index + 1);
    } else {
      setIndex(0);
    }
  }

  let image = images[index];
  return (
    <>
      <button onClick={handleClick}>
        下一张
      </button>
      <h3>
        第 {index + 1} 张，共 {images.length} 张
      </h3>
      <img key={image.src} src={image.src} />
      <p>
        {image.place}
      </p>
    </>
  );
}

let images = [{
  place: '马来西亚，槟城',
  src: 'https://react.dev/images/docs/scientists/FJeJR8M.jpg'
}, {
  place: '葡萄牙，里斯本',
  src: 'https://react.dev/images/docs/scientists/dB2LRbj.jpg'
}, {
  place: '西班牙，毕尔巴鄂',
  src: 'https://react.dev/images/docs/scientists/z08o2TS.jpg'
}, {
  place: '智利，瓦尔帕莱索',
  src: 'https://react.dev/images/docs/scientists/Y3utgTi.jpg'
}, {
  place: '瑞士，施维茨',
  src: 'https://react.dev/images/docs/scientists/JBbMpWY.jpg'
}, {
  place: '捷克，布拉格',
  src: 'https://react.dev/images/docs/scientists/QwUKKmF.jpg'
}, {
  place: '斯洛文尼亚，卢布尔雅那',
  src: 'https://react.dev/images/docs/scientists/3aIiwfm.jpg'
}];
```

```css
img { width: 150px; height: 150px; }
```

</Sandpack>

</Solution>

#### 修复列表中错位的状态 {/*fix-misplaced-state-in-the-list*/}

在这个列表中，每个 `Contact` 都有一个状态，用来决定是否已经按下过“Show email”。先为 Alice 按下“Show email”，然后勾选“Show in reverse order”复选框。你会注意到现在展开的是 _Taylor_ 的邮箱，但移到下面的 Alice 却仍然是折叠状态。

请修复它，使展开状态与每个联系人绑定，而不受所选排序影响。

<Sandpack>

```js src/App.js
import { useState } from 'react';
import Contact from './Contact.js';

export default function ContactList() {
  const [reverse, setReverse] = useState(false);

  const displayedContacts = [...contacts];
  if (reverse) {
    displayedContacts.reverse();
  }

  return (
    <>
      <label>
        <input
          type="checkbox"
          checked={reverse}
          onChange={e => {
            setReverse(e.target.checked)
          }}
        />{' '}
        反向显示顺序
      </label>
      <ul>
        {displayedContacts.map((contact, i) =>
          <li key={i}>
            <Contact contact={contact} />
          </li>
        )}
      </ul>
    </>
  );
}

const contacts = [
  { id: 0, name: 'Alice', email: 'alice@mail.com' },
  { id: 1, name: 'Bob', email: 'bob@mail.com' },
  { id: 2, name: 'Taylor', email: 'taylor@mail.com' }
];
```

```js src/Contact.js
import { useState } from 'react';

export default function Contact({ contact }) {
  const [expanded, setExpanded] = useState(false);
  return (
    <>
      <p><b>{contact.name}</b></p>
      {expanded &&
        <p><i>{contact.email}</i></p>
      }
      <button onClick={() => {
        setExpanded(!expanded);
      }}>
        {expanded ? '隐藏' : '显示'}邮箱
      </button>
    </>
  );
}
```

```css
ul, li {
  list-style: none;
  margin: 0;
  padding: 0;
}
li {
  margin-bottom: 20px;
}
label {
  display: block;
  margin: 10px 0;
}
button {
  margin-right: 10px;
  margin-bottom: 10px;
}
```

</Sandpack>

<Solution>

问题在于这个示例把索引用作了 `key`：

```js
{displayedContacts.map((contact, i) =>
  <li key={i}>
```

然而，你希望状态与_每一个具体联系人_关联。

改用联系人 ID 作为 `key` 就能修复这个问题：

<Sandpack>

```js src/App.js
import { useState } from 'react';
import Contact from './Contact.js';

export default function ContactList() {
  const [reverse, setReverse] = useState(false);

  const displayedContacts = [...contacts];
  if (reverse) {
    displayedContacts.reverse();
  }

  return (
    <>
      <label>
        <input
          type="checkbox"
          checked={reverse}
          onChange={e => {
            setReverse(e.target.checked)
          }}
        />{' '}
        反向显示顺序
      </label>
      <ul>
        {displayedContacts.map(contact =>
          <li key={contact.id}>
            <Contact contact={contact} />
          </li>
        )}
      </ul>
    </>
  );
}

const contacts = [
  { id: 0, name: 'Alice', email: 'alice@mail.com' },
  { id: 1, name: 'Bob', email: 'bob@mail.com' },
  { id: 2, name: 'Taylor', email: 'taylor@mail.com' }
];
```

```js src/Contact.js
import { useState } from 'react';

export default function Contact({ contact }) {
  const [expanded, setExpanded] = useState(false);
  return (
    <>
      <p><b>{contact.name}</b></p>
      {expanded &&
        <p><i>{contact.email}</i></p>
      }
      <button onClick={() => {
        setExpanded(!expanded);
      }}>
        {expanded ? '隐藏' : '显示'}邮箱
      </button>
    </>
  );
}
```

```css
ul, li {
  list-style: none;
  margin: 0;
  padding: 0;
}
li {
  margin-bottom: 20px;
}
label {
  display: block;
  margin: 10px 0;
}
button {
  margin-right: 10px;
  margin-bottom: 10px;
}
```

</Sandpack>

状态与树中的位置相关联。`key` 让你可以指定一个命名位置，而不是依赖顺序。

</Solution>

</Challenges>
