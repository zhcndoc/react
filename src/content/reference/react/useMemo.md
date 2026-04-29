---
title: useMemo
---

<Intro>

`useMemo` 是一个 React Hook，它让你可以在重新渲染之间缓存计算结果。

```js
const cachedValue = useMemo(calculateValue, dependencies)
```

</Intro>

<Note>

[React Compiler](/learn/react-compiler) 会自动对值和函数进行 memoize，从而减少手动调用 `useMemo` 的需要。你可以使用编译器自动处理 memoization。

</Note>

<InlineToc />

---

## 参考 {/*reference*/}

### `useMemo(calculateValue, dependencies)` {/*usememo*/}

在组件顶层调用 `useMemo`，以在重新渲染之间缓存一次计算：

```js
import { useMemo } from 'react';

function TodoList({ todos, tab }) {
  const visibleTodos = useMemo(
    () => filterTodos(todos, tab),
    [todos, tab]
  );
  // ...
}
```

[查看更多下面的示例。](#usage)

#### 参数 {/*parameters*/}

* `calculateValue`：用于计算你想缓存的值的函数。它应该是纯函数，不接受任何参数，并且可以返回任意类型的值。React 会在初始渲染时调用你的函数。在后续渲染中，如果自上次渲染以来 `dependencies` 没有变化，React 会再次返回相同的值。否则，它会调用 `calculateValue`，返回其结果，并将其存储起来以便之后复用。

* `dependencies`：`calculateValue` 代码中引用的所有响应式值的列表。响应式值包括 props、state，以及在组件函数体内直接声明的所有变量和函数。如果你的 lint 规则已为 [React 配置](/learn/editor-setup#linting)，它会验证每个响应式值都被正确指定为依赖项。依赖项列表必须具有固定数量的项目，并且像 `[dep1, dep2, dep3]` 这样内联编写。React 会使用 [`Object.is`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is) 比较来逐个对比每个依赖项与其上一次的值。

#### 返回值 {/*returns*/}

在初始渲染时，`useMemo` 返回调用不带参数的 `calculateValue` 的结果。

在后续渲染中，它要么返回上一次渲染时已经存储的值（如果依赖项没有变化），要么再次调用 `calculateValue`，并返回 `calculateValue` 返回的结果。

#### 注意事项 {/*caveats*/}

* `useMemo` 是一个 Hook，所以你只能在**组件顶层**或你自己的 Hooks 中调用它。你不能在循环或条件语句中调用它。如果你需要那样做，请提取一个新组件并将 state 移入其中。
* 在严格模式下，React 会**调用你的计算函数两次**，以便 [帮助你发现意外的非纯性。](#my-calculation-runs-twice-on-every-re-render) 这是仅开发环境的行为，不会影响生产环境。如果你的计算函数是纯的（它本来就应该是），这不会影响你的逻辑。其中一次调用的结果会被忽略。
* React **不会丢弃缓存值，除非有明确原因需要这样做。** 例如，在开发环境中，当你编辑组件文件时，React 会丢弃缓存。在开发和生产环境中，如果你的组件在初始挂载时 suspend，React 也会丢弃缓存。未来，React 可能会添加更多利用丢弃缓存的特性——例如，如果未来 React 内置支持虚拟列表，那么对于滚出虚拟化表格视口的项目，丢弃缓存就是合理的。如果你只是将 `useMemo` 作为性能优化来使用，这没问题。否则，`state` 变量或 [`ref`](/reference/react/useRef#avoiding-recreating-the-ref-contents) 可能更合适。

<Note>

像这样缓存返回值也被称为 [*memoization*，](https://en.wikipedia.org/wiki/Memoization) 这就是这个 Hook 被命名为 `useMemo` 的原因。

</Note>

---

## 使用方法 {/*usage*/}

### 跳过昂贵的重新计算 {/*skipping-expensive-recalculations*/}

要在重新渲染之间缓存一次计算，请在组件顶层用 `useMemo` 包裹它：

```js [[3, 4, "visibleTodos"], [1, 4, "() => filterTodos(todos, tab)"], [2, 4, "[todos, tab]"]]
import { useMemo } from 'react';

function TodoList({ todos, tab, theme }) {
  const visibleTodos = useMemo(() => filterTodos(todos, tab), [todos, tab]);
  // ...
}
```

你需要向 `useMemo` 传入两样东西：

1. 一个不接受参数的<CodeStep step={1}>计算函数</CodeStep>，例如 `() =>`，并返回你想计算的内容。
2. 一个<CodeStep step={2}>依赖项列表</CodeStep>，其中包括组件中在计算里用到的每个值。

在初始渲染时，你从 `useMemo` 得到的<CodeStep step={3}>值</CodeStep>将是调用<CodeStep step={1}>计算函数</CodeStep>的结果。

在每次后续渲染中，React 会将<CodeStep step={2}>依赖项</CodeStep>与你在上一次渲染中传入的依赖项进行比较。如果所有依赖项都没有变化（与 [`Object.is`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is) 比较），`useMemo` 会返回你之前已经计算好的值。否则，React 会重新运行你的计算并返回新值。

换句话说，`useMemo` 会在重新渲染之间缓存一次计算结果，直到它的依赖项发生变化。

**让我们通过一个例子来看看这什么时候有用。**

默认情况下，每次组件重新渲染时，React 都会重新执行整个组件函数体。例如，如果这个 `TodoList` 更新了 state，或者从父组件收到了新的 props，`filterTodos` 函数就会重新运行：

```js {2}
function TodoList({ todos, tab, theme }) {
  const visibleTodos = filterTodos(todos, tab);
  // ...
}
```

通常这不会成为问题，因为大多数计算都非常快。但是，如果你正在过滤或转换一个很大的数组，或者在做一些昂贵的计算，那么如果数据没有变化，你可能希望跳过这一步。如果 `todos` 和 `tab` 与上次渲染时相同，像前面那样用 `useMemo` 包裹计算，就可以复用你之前已经算出的 `visibleTodos`。

这种类型的缓存称为 *[memoization.](https://en.wikipedia.org/wiki/Memoization)*

<Note>

**你应该只把 `useMemo` 作为性能优化手段来依赖。** 如果没有它代码就不能工作，那就先找出底层问题并修复它。然后你再可以添加 `useMemo` 来提升性能。

</Note>

<DeepDive>

#### 如何判断一个计算是否昂贵？ {/*how-to-tell-if-a-calculation-is-expensive*/}

一般来说，除非你正在创建或遍历成千上万个对象，否则它大概率不算昂贵。如果你想更有把握一些，可以加一条 console log 来测量一段代码花费的时间：

```js {1,3}
console.time('filter array');
const visibleTodos = filterTodos(todos, tab);
console.timeEnd('filter array');
```

执行你要测量的交互（例如，在输入框中输入）。然后你会在控制台看到类似 `filter array: 0.15ms` 的日志。如果总耗时累计到相当可观的程度（比如 `1ms` 或更高），那就可能值得把这个计算 memoize。作为实验，你可以把计算包裹进 `useMemo`，看看对于那次交互，总日志时间是否减少了：

```js
console.time('filter array');
const visibleTodos = useMemo(() => {
  return filterTodos(todos, tab); // 如果 todos 和 tab 没变，这里会被跳过
}, [todos, tab]);
console.timeEnd('filter array');
```

`useMemo` 不会让*首次*渲染更快。它只会帮助你在更新时跳过不必要的工作。

请记住，你的机器很可能比用户的机器更快，所以最好用人为降速来测试性能。例如，Chrome 提供了一个 [CPU Throttling](https://developer.chrome.com/blog/new-in-devtools-61/#throttling) 选项。

另外要注意，在开发环境中测量性能不会给出最准确的结果。（例如，当开启 [Strict Mode](/reference/react/StrictMode) 时，你会看到每个组件渲染两次，而不是一次。）要获得最准确的耗时数据，请将应用构建为生产版本，并在类似用户所使用的设备上测试。

</DeepDive>

<DeepDive>

#### 是否应该到处都加 useMemo？ {/*should-you-add-usememo-everywhere*/}

如果你的应用像这个网站一样，大多数交互都比较粗粒度（比如替换一个页面或一个完整区块），那么 memoization 通常是不必要的。另一方面，如果你的应用更像绘图编辑器，而且大多数交互都很细粒度（比如移动图形），那么你可能会发现 memoization 非常有帮助。

使用 `useMemo` 做优化只有在少数情况下才有价值：

- 你放进 `useMemo` 的计算明显很慢，而且它的依赖项很少变化。
- 你将它作为 prop 传给了一个用 [`memo`](/reference/react/memo) 包裹的组件。你希望在值没有变化时跳过重新渲染。memoization 让你的组件只在依赖项不同时重新渲染。
- 你传递的值后来被用作某个 Hook 的依赖项。例如，也许另一个 `useMemo` 的计算值依赖它。或者你在 [`useEffect`](/reference/react/useEffect) 中依赖这个值。

在其他情况下，把一个计算包裹进 `useMemo` 并没有收益。这样做也没有明显坏处，所以有些团队会选择不去考虑单个案例，而是尽可能多地做 memoize。这个方法的缺点是代码可读性会变差。而且，并不是所有 memoization 都有效：只要有一个“总是新的”值，就足以破坏整个组件的 memoization。

**在实践中，你可以通过遵循一些原则来让很多 memoization 变得不必要：**

1. 当一个组件在视觉上包裹其他组件时，让它[接受 JSX 作为 children。](/learn/passing-props-to-a-component#passing-jsx-as-children) 这样，当包装组件更新自身 state 时，React 知道它的 children 不需要重新渲染。
1. 优先使用局部 state，不要把 [state 上提](/learn/sharing-state-between-components) 得超过必要范围。例如，不要把像表单这类临时状态，或者某项是否被悬停这类状态放在组件树的顶部或全局状态库里。
1. 保持你的[渲染逻辑纯粹。](/learn/keeping-components-pure) 如果重新渲染一个组件会导致问题或产生某种明显的视觉瑕疵，那就是你组件里的 bug！修复这个 bug，而不是添加 memoization。
1. 避免[不必要的会更新 state 的 Effect。](/learn/you-might-not-need-an-effect) React 应用中大多数性能问题都来自 Effect 触发的一连串更新，这些更新会导致组件一遍又一遍地渲染。
1. 尝试[从 Effect 中移除不必要的依赖项。](/learn/removing-effect-dependencies) 例如，与其做 memoization，不如把某个对象或函数移到 Effect 里面，或者移到组件外面。

如果某个具体交互仍然感觉卡顿，可以[使用 React Developer Tools 的 profiler](https://legacy.reactjs.org/blog/2018/09/10/introducing-the-react-profiler.html) 看看哪些组件最值得做 memoization，然后在需要的地方添加 memoization。这些原则会让你的组件更容易调试和理解，所以无论如何都值得遵循。从长远来看，我们正在研究[自动进行细粒度 memoization](https://www.youtube.com/watch?v=lGEMwh32soc)，以一劳永逸地解决这个问题。

</DeepDive>

<Recipes titleText="useMemo 与直接计算值之间的区别" titleId="examples-recalculation">

#### 使用 `useMemo` 跳过重新计算 {/*skipping-recalculation-with-usememo*/}

在这个例子中，`filterTodos` 的实现被**人为降速**了，这样你就能看到：当你在渲染过程中调用的某个 JavaScript 函数确实很慢时，会发生什么。试着切换标签页并切换主题。

切换标签页会感觉很慢，因为它会强制重新执行被降速的 `filterTodos`。这是预期内的，因为 `tab` 变了，所以整个计算*需要*重新运行。（如果你想知道为什么它会运行两次，这里有解释。[这里。](#my-calculation-runs-twice-on-every-re-render)）

切换主题。**多亏了 `useMemo`，即使有人工降速，它也依然很快！** 慢速的 `filterTodos` 调用被跳过了，因为自上次渲染以来，`todos` 和 `tab`（你作为 `useMemo` 的依赖项传入的值）都没有变化。

<Sandpack>

```js src/App.js
import { useState } from 'react';
import { createTodos } from './utils.js';
import TodoList from './TodoList.js';

const todos = createTodos();

export default function App() {
  const [tab, setTab] = useState('all');
  const [isDark, setIsDark] = useState(false);
  return (
    <>
      <button onClick={() => setTab('all')}>
        All
      </button>
      <button onClick={() => setTab('active')}>
        Active
      </button>
      <button onClick={() => setTab('completed')}>
        Completed
      </button>
      <br />
      <label>
        <input
          type="checkbox"
          checked={isDark}
          onChange={e => setIsDark(e.target.checked)}
        />
        Dark mode
      </label>
      <hr />
      <TodoList
        todos={todos}
        tab={tab}
        theme={isDark ? 'dark' : 'light'}
      />
    </>
  );
}

```

```js src/TodoList.js active
import { useMemo } from 'react';
import { filterTodos } from './utils.js'

export default function TodoList({ todos, theme, tab }) {
  const visibleTodos = useMemo(
    () => filterTodos(todos, tab),
    [todos, tab]
  );
  return (
    <div className={theme}>
      <p><b>注意：<code>filterTodos</code> 被人为降速了！</b></p>
      <ul>
        {visibleTodos.map(todo => (
          <li key={todo.id}>
            {todo.completed ?
              <s>{todo.text}</s> :
              todo.text
            }
          </li>
        ))}
      </ul>
    </div>
  );
}
```

```js src/utils.js
export function createTodos() {
  const todos = [];
  for (let i = 0; i < 50; i++) {
    todos.push({
      id: i,
      text: "Todo " + (i + 1),
      completed: Math.random() > 0.5
    });
  }
  return todos;
}

export function filterTodos(todos, tab) {
  console.log('[ARTIFICIALLY SLOW] Filtering ' + todos.length + ' todos for "' + tab + '" tab.');
  let startTime = performance.now();
  while (performance.now() - startTime < 500) {
    // 什么都不做 500 毫秒，以模拟极慢的代码
  }

  return todos.filter(todo => {
    if (tab === 'all') {
      return true;
    } else if (tab === 'active') {
      return !todo.completed;
    } else if (tab === 'completed') {
      return todo.completed;
    }
  });
}
```

```css
label {
  display: block;
  margin-top: 10px;
}

.dark {
  background-color: black;
  color: white;
}

.light {
  background-color: white;
  color: black;
}
```

</Sandpack>

<Solution />

#### 总是重新计算值 {/*always-recalculating-a-value*/}

在这个例子中，`filterTodos` 的实现也被**人为降速**了，这样你就能看到：当你在渲染过程中调用的某个 JavaScript 函数确实很慢时，会发生什么。试着切换标签页并切换主题。

和前一个例子不同，现在切换主题也会变慢！这是因为**这个版本里没有 `useMemo` 调用，** 所以被人为降速的 `filterTodos` 会在每次重新渲染时都被调用。即使只是 `theme` 改变了，它也会被调用。

<Sandpack>

```js src/App.js
import { useState } from 'react';
import { createTodos } from './utils.js';
import TodoList from './TodoList.js';

const todos = createTodos();

export default function App() {
  const [tab, setTab] = useState('all');
  const [isDark, setIsDark] = useState(false);
  return (
    <>
      <button onClick={() => setTab('all')}>
        All
      </button>
      <button onClick={() => setTab('active')}>
        Active
      </button>
      <button onClick={() => setTab('completed')}>
        Completed
      </button>
      <br />
      <label>
        <input
          type="checkbox"
          checked={isDark}
          onChange={e => setIsDark(e.target.checked)}
        />
        Dark mode
      </label>
      <hr />
      <TodoList
        todos={todos}
        tab={tab}
        theme={isDark ? 'dark' : 'light'}
      />
    </>
  );
}

```

```js src/TodoList.js active
import { filterTodos } from './utils.js'

export default function TodoList({ todos, theme, tab }) {
  const visibleTodos = filterTodos(todos, tab);
  return (
    <div className={theme}>
      <ul>
        <p><b>注意：<code>filterTodos</code> 被人为降速了！</b></p>
        {visibleTodos.map(todo => (
          <li key={todo.id}>
            {todo.completed ?
              <s>{todo.text}</s> :
              todo.text
            }
          </li>
        ))}
      </ul>
    </div>
  );
}
```

```js src/utils.js
export function createTodos() {
  const todos = [];
  for (let i = 0; i < 50; i++) {
    todos.push({
      id: i,
      text: "Todo " + (i + 1),
      completed: Math.random() > 0.5
    });
  }
  return todos;
}

export function filterTodos(todos, tab) {
  console.log('[ARTIFICIALLY SLOW] Filtering ' + todos.length + ' todos for "' + tab + '" tab.');
  let startTime = performance.now();
  while (performance.now() - startTime < 500) {
    // 什么都不做 500 毫秒，以模拟极慢的代码
  }

  return todos.filter(todo => {
    if (tab === 'all') {
      return true;
    } else if (tab === 'active') {
      return !todo.completed;
    } else if (tab === 'completed') {
      return todo.completed;
    }
  });
}
```

```css
label {
  display: block;
  margin-top: 10px;
}

.dark {
  background-color: black;
  color: white;
}

.light {
  background-color: white;
  color: black;
}
```

</Sandpack>

不过，这里是移除了**人为降速**后的同一份代码。没有 `useMemo` 会有明显感觉吗？

<Sandpack>

```js src/App.js
import { useState } from 'react';
import { createTodos } from './utils.js';
import TodoList from './TodoList.js';

const todos = createTodos();

export default function App() {
  const [tab, setTab] = useState('all');
  const [isDark, setIsDark] = useState(false);
  return (
    <>
      <button onClick={() => setTab('all')}>
        All
      </button>
      <button onClick={() => setTab('active')}>
        Active
      </button>
      <button onClick={() => setTab('completed')}>
        Completed
      </button>
      <br />
      <label>
        <input
          type="checkbox"
          checked={isDark}
          onChange={e => setIsDark(e.target.checked)}
        />
        Dark mode
      </label>
      <hr />
      <TodoList
        todos={todos}
        tab={tab}
        theme={isDark ? 'dark' : 'light'}
      />
    </>
  );
}

```

```js src/TodoList.js active
import { filterTodos } from './utils.js'

export default function TodoList({ todos, theme, tab }) {
  const visibleTodos = filterTodos(todos, tab);
  return (
    <div className={theme}>
      <ul>
        {visibleTodos.map(todo => (
          <li key={todo.id}>
            {todo.completed ?
              <s>{todo.text}</s> :
              todo.text
            }
          </li>
        ))}
      </ul>
    </div>
  );
}
```

```js src/utils.js
export function createTodos() {
  const todos = [];
  for (let i = 0; i < 50; i++) {
    todos.push({
      id: i,
      text: "Todo " + (i + 1),
      completed: Math.random() > 0.5
    });
  }
  return todos;
}

export function filterTodos(todos, tab) {
  console.log('Filtering ' + todos.length + ' todos for "' + tab + '" tab.');

  return todos.filter(todo => {
    if (tab === 'all') {
      return true;
    } else if (tab === 'active') {
      return !todo.completed;
    } else if (tab === 'completed') {
      return todo.completed;
    }
  });
}
```

```css
label {
  display: block;
  margin-top: 10px;
}

.dark {
  background-color: black;
  color: white;
}

.light {
  background-color: white;
  color: black;
}
```

</Sandpack>

很多时候，不做 memoization 的代码也能正常工作。如果你的交互已经足够快，也许根本不需要 memoization。

你可以试着增加 `utils.js` 中 todo 项的数量，看看行为会如何变化。这个特定计算一开始并不算很昂贵，但如果 todo 数量显著增长，大多数开销会来自重新渲染，而不是过滤本身。继续往下看，了解如何使用 `useMemo` 优化重新渲染。

<Solution />

</Recipes>

---

### 跳过组件重新渲染 {/*skipping-re-rendering-of-components*/}

在某些情况下，`useMemo` 也可以帮助你优化子组件重新渲染的性能。为了说明这一点，假设这个 `TodoList` 组件把 `visibleTodos` 作为 prop 传给子组件 `List`：

```js {5}
export default function TodoList({ todos, tab, theme }) {
  // ...
  return (
    <div className={theme}>
      <List items={visibleTodos} />
    </div>
  );
}
```

你注意到切换 `theme` prop 时，应用会冻结一下；但如果你把 JSX 中的 `<List />` 删除，就会感觉很快。这说明值得尝试优化 `List` 组件。

**默认情况下，当组件重新渲染时，React 会递归地重新渲染它的所有子组件。** 这就是为什么当 `TodoList` 用不同的 `theme` 重新渲染时，`List` 组件也会重新渲染。对于那些重新渲染时不需要太多计算的组件，这没问题。但如果你已经确认某次重新渲染很慢，你可以通过用 [`memo`](/reference/react/memo) 包裹 `List`，让它在 props 与上次渲染相同时跳过重新渲染：

```js {3,5}
import { memo } from 'react';

const List = memo(function List({ items }) {
  // ...
});
```

**做了这个改动后，如果 `List` 的所有 props 与上次渲染时*相同*，它就会跳过重新渲染。** 这就是缓存计算变得重要的地方！想象一下，如果你没有使用 `useMemo` 来计算 `visibleTodos`：

```js {2-3,6-7}
export default function TodoList({ todos, tab, theme }) {
  // 每次 theme 变化时，这里都会得到一个不同的数组……
  const visibleTodos = filterTodos(todos, tab);
  return (
    <div className={theme}>
      {/* ……因此 List 的 props 永远不会相同，它会在每次都重新渲染 */}
      <List items={visibleTodos} />
    </div>
  );
}
```

**在上面的例子中，`filterTodos` 函数总是会创建一个*不同*的数组，** 类似于 `{}` 对象字面量总是创建一个新对象。通常这不会成为问题，但这意味着 `List` 的 props 永远不会相同，而你的 [`memo`](/reference/react/memo) 优化也就不会生效。这时 `useMemo` 就派上用场了：

```js {2-3,5,9-10}
export default function TodoList({ todos, tab, theme }) {
  // 告诉 React 在重新渲染之间缓存你的计算……
  const visibleTodos = useMemo(
    () => filterTodos(todos, tab),
    [todos, tab] // ……只要这些依赖项不变……
  );
  return (
    <div className={theme}>
      {/* ……List 会收到相同的 props，并且可以跳过重新渲染 */}
      <List items={visibleTodos} />
    </div>
  );
}
```


**通过用 `useMemo` 包裹 `visibleTodos` 的计算，你可以确保它在重新渲染之间具有*相同*的值**（直到依赖项变化）。你并*不一定*要把计算包裹进 `useMemo`，除非你有某个特定原因。在这个例子中，原因是你把它传给了一个用 [`memo`](/reference/react/memo) 包裹的组件，这样它就能跳过重新渲染。还有一些其他添加 `useMemo` 的原因，后文会继续说明。

<DeepDive>

#### 对单个 JSX 节点做 memoize {/*memoizing-individual-jsx-nodes*/}

你可以不把 `List` 用 [`memo`](/reference/react/memo) 包裹，而是直接把 `<List />` 这个 JSX 节点本身放进 `useMemo`：

```js {3,6}
export default function TodoList({ todos, tab, theme }) {
  const visibleTodos = useMemo(() => filterTodos(todos, tab), [todos, tab]);
  const children = useMemo(() => <List items={visibleTodos} />, [visibleTodos]);
  return (
    <div className={theme}>
      {children}
    </div>
  );
}
```

行为会是一样的。如果 `visibleTodos` 没变，`List` 就不会重新渲染。

像 `<List items={visibleTodos} />` 这样的 JSX 节点本质上是一个对象，例如 `{ type: List, props: { items: visibleTodos } }`。创建这个对象非常便宜，但 React 并不知道它的内容是否和上次一样。这就是为什么默认情况下，React 会重新渲染 `List` 组件。

不过，如果 React 看到的 JSX 和上一次渲染时完全相同，它就不会尝试重新渲染你的组件。这是因为 JSX 节点是[不可变的。](https://en.wikipedia.org/wiki/Immutable_object) 一个 JSX 节点对象不可能随着时间改变，所以 React 知道跳过重新渲染是安全的。不过，要让这生效，节点必须*真的是同一个对象*，而不只是代码看起来相同。这就是这个例子中 `useMemo` 的作用。

手动把 JSX 节点包进 `useMemo` 并不方便。例如，你不能有条件地这么做。通常这就是为什么你会选择用 [`memo`](/reference/react/memo) 包裹组件，而不是包裹 JSX 节点。

</DeepDive>

<Recipes titleText="跳过重新渲染与总是重新渲染之间的区别" titleId="examples-rerendering">

#### 使用 `useMemo` 和 `memo` 跳过重新渲染 {/*skipping-re-rendering-with-usememo-and-memo*/}

在这个例子中，`List` 组件被**人为降速**了，这样你就能看到：当你渲染的某个 React 组件确实很慢时，会发生什么。试着切换标签页并切换主题。

切换标签页会感觉很慢，因为它会强制被降速的 `List` 重新渲染。这是预期内的，因为 `tab` 已经变了，所以你需要把用户的新选择反映到屏幕上。

接下来，试着切换主题。**多亏了 `useMemo` 和 [`memo`](/reference/react/memo) 的组合，即使有人工降速，它也依然很快！** `List` 跳过了重新渲染，因为自上次渲染以来 `visibleTodos` 数组没有变化。`visibleTodos` 数组没有变化，是因为自上次渲染以来，`todos` 和 `tab`（你作为 `useMemo` 的依赖项传入的值）都没有变化。

<Sandpack>

```js src/App.js
import { useState } from 'react';
import { createTodos } from './utils.js';
import TodoList from './TodoList.js';

const todos = createTodos();

export default function App() {
  const [tab, setTab] = useState('all');
  const [isDark, setIsDark] = useState(false);
  return (
    <>
      <button onClick={() => setTab('all')}>
        All
      </button>
      <button onClick={() => setTab('active')}>
        Active
      </button>
      <button onClick={() => setTab('completed')}>
        Completed
      </button>
      <br />
      <label>
        <input
          type="checkbox"
          checked={isDark}
          onChange={e => setIsDark(e.target.checked)}
        />
        Dark mode
      </label>
      <hr />
      <TodoList
        todos={todos}
        tab={tab}
        theme={isDark ? 'dark' : 'light'}
      />
    </>
  );
}
```

```js src/TodoList.js active
import { useMemo } from 'react';
import List from './List.js';
import { filterTodos } from './utils.js'

export default function TodoList({ todos, theme, tab }) {
  const visibleTodos = useMemo(
    () => filterTodos(todos, tab),
    [todos, tab]
  );
  return (
    <div className={theme}>
      <p><b>注意：<code>List</code> 被人为降速了！</b></p>
      <List items={visibleTodos} />
    </div>
  );
}
```

```js {expectedErrors: {'react-compiler': [5, 6]}} src/List.js
import { memo } from 'react';

const List = memo(function List({ items }) {
  console.log('[ARTIFICIALLY SLOW] Rendering <List /> with ' + items.length + ' items');
  let startTime = performance.now();
  while (performance.now() - startTime < 500) {
    // 什么都不做 500 毫秒，以模拟极慢的代码
  }

  return (
    <ul>
      {items.map(item => (
        <li key={item.id}>
          {item.completed ?
            <s>{item.text}</s> :
            item.text
          }
        </li>
      ))}
    </ul>
  );
});

export default List;
```

```js src/utils.js
export function createTodos() {
  const todos = [];
  for (let i = 0; i < 50; i++) {
    todos.push({
      id: i,
      text: "Todo " + (i + 1),
      completed: Math.random() > 0.5
    });
  }
  return todos;
}

export function filterTodos(todos, tab) {
  return todos.filter(todo => {
    if (tab === 'all') {
      return true;
    } else if (tab === 'active') {
      return !todo.completed;
    } else if (tab === 'completed') {
      return todo.completed;
    }
  });
}
```

```css
label {
  display: block;
  margin-top: 10px;
}

.dark {
  background-color: black;
  color: white;
}

.light {
  background-color: white;
  color: black;
}
```

</Sandpack>

<Solution />

#### 总是重新渲染组件 {/*always-re-rendering-a-component*/}

在这个例子中，`List` 的实现也被**人为降速**了，这样你就能看到：当你渲染的某个 React 组件确实很慢时，会发生什么。试着切换标签页并切换主题。

和前一个例子不同，现在切换主题也会变慢！这是因为**这个版本里没有 `useMemo` 调用，** 所以 `visibleTodos` 总是不同的数组，而被降速的 `List` 组件就无法跳过重新渲染。

<Sandpack>

```js src/App.js
import { useState } from 'react';
import { createTodos } from './utils.js';
import TodoList from './TodoList.js';

const todos = createTodos();

export default function App() {
  const [tab, setTab] = useState('all');
  const [isDark, setIsDark] = useState(false);
  return (
    <>
      <button onClick={() => setTab('all')}>
        All
      </button>
      <button onClick={() => setTab('active')}>
        Active
      </button>
      <button onClick={() => setTab('completed')}>
        Completed
      </button>
      <br />
      <label>
        <input
          type="checkbox"
          checked={isDark}
          onChange={e => setIsDark(e.target.checked)}
        />
        Dark mode
      </label>
      <hr />
      <TodoList
        todos={todos}
        tab={tab}
        theme={isDark ? 'dark' : 'light'}
      />
    </>
  );
}
```

```js src/TodoList.js active
import List from './List.js';
import { filterTodos } from './utils.js'

export default function TodoList({ todos, theme, tab }) {
  const visibleTodos = filterTodos(todos, tab);
  return (
    <div className={theme}>
      <p><b>注意：<code>List</code> 被人为降速了！</b></p>
      <List items={visibleTodos} />
    </div>
  );
}
```

```js {expectedErrors: {'react-compiler': [5, 6]}} src/List.js
import { memo } from 'react';

const List = memo(function List({ items }) {
  console.log('[ARTIFICIALLY SLOW] Rendering <List /> with ' + items.length + ' items');
  let startTime = performance.now();
  while (performance.now() - startTime < 500) {
    // 什么都不做 500 毫秒，以模拟极慢的代码
  }

  return (
    <ul>
      {items.map(item => (
        <li key={item.id}>
          {item.completed ?
            <s>{item.text}</s> :
            item.text
          }
        </li>
      ))}
    </ul>
  );
});

export default List;
```

```js src/utils.js
export function createTodos() {
  const todos = [];
  for (let i = 0; i < 50; i++) {
    todos.push({
      id: i,
      text: "Todo " + (i + 1),
      completed: Math.random() > 0.5
    });
  }
  return todos;
}

export function filterTodos(todos, tab) {
  return todos.filter(todo => {
    if (tab === 'all') {
      return true;
    } else if (tab === 'active') {
      return !todo.completed;
    } else if (tab === 'completed') {
      return todo.completed;
    }
  });
}
```

```css
label {
  display: block;
  margin-top: 10px;
}

.dark {
  background-color: black;
  color: white;
}

.light {
  background-color: white;
  color: black;
}
```

</Sandpack>

不过，这里是移除了**人为降速**后的同一份代码。没有 `useMemo` 会有明显感觉吗？

<Sandpack>

```js src/App.js
import { useState } from 'react';
import { createTodos } from './utils.js';
import TodoList from './TodoList.js';

const todos = createTodos();

export default function App() {
  const [tab, setTab] = useState('all');
  const [isDark, setIsDark] = useState(false);
  return (
    <>
      <button onClick={() => setTab('all')}>
        All
      </button>
      <button onClick={() => setTab('active')}>
        Active
      </button>
      <button onClick={() => setTab('completed')}>
        Completed
      </button>
      <br />
      <label>
        <input
          type="checkbox"
          checked={isDark}
          onChange={e => setIsDark(e.target.checked)}
        />
        Dark mode
      </label>
      <hr />
      <TodoList
        todos={todos}
        tab={tab}
        theme={isDark ? 'dark' : 'light'}
      />
    </>
  );
}
```

```js src/TodoList.js active
import List from './List.js';
import { filterTodos } from './utils.js'

export default function TodoList({ todos, theme, tab }) {
  const visibleTodos = filterTodos(todos, tab);
  return (
    <div className={theme}>
      <List items={visibleTodos} />
    </div>
  );
}
```

```js src/List.js
import { memo } from 'react';

function List({ items }) {
  return (
    <ul>
      {items.map(item => (
        <li key={item.id}>
          {item.completed ?
            <s>{item.text}</s> :
            item.text
          }
        </li>
      ))}
    </ul>
  );
}

export default memo(List);
```

```js src/utils.js
export function createTodos() {
  const todos = [];
  for (let i = 0; i < 50; i++) {
    todos.push({
      id: i,
      text: "Todo " + (i + 1),
      completed: Math.random() > 0.5
    });
  }
  return todos;
}

export function filterTodos(todos, tab) {
  return todos.filter(todo => {
    if (tab === 'all') {
      return true;
    } else if (tab === 'active') {
      return !todo.completed;
    } else if (tab === 'completed') {
      return todo.completed;
    }
  });
}
```

```css
label {
  display: block;
  margin-top: 10px;
}

.dark {
  background-color: black;
  color: white;
}

.light {
  background-color: white;
  color: black;
}
```

</Sandpack>

很多时候，不做 memoization 的代码也能正常工作。如果你的交互已经足够快，就不需要 memoization。

请记住，你需要以生产模式运行 React，禁用 [React Developer Tools](/learn/react-developer-tools)，并使用与你的应用用户类似的设备，才能真实感受到到底是什么在拖慢你的应用。

<Solution />

</Recipes>

---

### 防止 Effect 过于频繁地触发 {/*preventing-an-effect-from-firing-too-often*/}

有时候，你可能想在一个 [Effect:](/learn/synchronizing-with-effects) 中使用某个值：

```js {4-7,10}
function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  const options = {
    serverUrl: 'https://localhost:1234',
    roomId: roomId
  }

  useEffect(() => {
    const connection = createConnection(options);
    connection.connect();
    // ...
```

这会产生一个问题。[每个响应式值都必须声明为你的 Effect 的依赖项。](/learn/lifecycle-of-reactive-effects#react-verifies-that-you-specified-every-reactive-value-as-a-dependency) 然而，如果你把 `options` 声明为依赖项，它会导致你的 Effect 不断重新连接到聊天室：


```js {5}
  useEffect(() => {
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [options]); // 🔴 问题：这个依赖项在每次渲染时都会变化
  // ...
```

要解决这个问题，你可以把你需要在 Effect 中调用的对象用 `useMemo` 包裹起来：

```js {4-9,16}
function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  const options = useMemo(() => {
    return {
      serverUrl: 'https://localhost:1234',
      roomId: roomId
    };
  }, [roomId]); // ✅ 只有在 roomId 变化时才会变化

  useEffect(() => {
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [options]); // ✅ 只有在 options 变化时才会变化
  // ...
```

这能确保如果 `useMemo` 返回缓存对象，那么在重新渲染之间 `options` 对象是相同的。

不过，由于 `useMemo` 是性能优化，而不是语义保证，React 可能会在[有明确原因需要这样做时](#caveats)丢弃缓存值。这也会导致 Effect 重新触发，**所以更好的做法是通过把对象*移到* Effect 内部，来消除对函数依赖的需要：**

```js {5-8,13}
function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  useEffect(() => {
    const options = { // ✅ 不需要 useMemo，也不需要对象依赖项！
      serverUrl: 'https://localhost:1234',
      roomId: roomId
    }

    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]); // ✅ 只有在 roomId 变化时才会变化
  // ...
```

现在你的代码更简单了，也不需要 `useMemo` 了。[了解更多关于移除 Effect 依赖项。](/learn/removing-effect-dependencies#move-dynamic-objects-and-functions-inside-your-effect)


### 为另一个 Hook 的依赖项做 memoize {/*memoizing-a-dependency-of-another-hook*/}

假设你有一个依赖于组件函数体内直接创建对象的计算：

```js {2}
function Dropdown({ allItems, text }) {
  const searchOptions = { matchMode: 'whole-word', text };

  const visibleItems = useMemo(() => {
    return searchItems(allItems, searchOptions);
  }, [allItems, searchOptions]); // 🚩 注意：依赖于组件函数体内创建的对象
  // ...
```

依赖一个这样的对象会抵消 memoization 的意义。当组件重新渲染时，组件函数体内直接写的所有代码都会再次运行。**创建 `searchOptions` 对象的那几行代码也会在每次重新渲染时运行。** 由于 `searchOptions` 是你 `useMemo` 调用的依赖项，而且它每次都不同，React 知道依赖项不同，因此会每次都重新计算 `searchItems`。

要修复这个问题，你可以在把 `searchOptions` 对象作为依赖项传入之前，先把它本身 memoize：

```js {2-4}
function Dropdown({ allItems, text }) {
  const searchOptions = useMemo(() => {
    return { matchMode: 'whole-word', text };
  }, [text]); // ✅ 只有在 text 变化时才会变化

  const visibleItems = useMemo(() => {
    return searchItems(allItems, searchOptions);
  }, [allItems, searchOptions]); // ✅ 只有在 allItems 或 searchOptions 变化时才会变化
  // ...
```

在上面的例子中，如果 `text` 没变，`searchOptions` 对象也不会变。不过，更好的修复方式是把 `searchOptions` 对象的声明*移到* `useMemo` 计算函数内部：

```js {3}
function Dropdown({ allItems, text }) {
  const visibleItems = useMemo(() => {
    const searchOptions = { matchMode: 'whole-word', text };
    return searchItems(allItems, searchOptions);
  }, [allItems, text]); // ✅ 只有在 allItems 或 text 变化时才会变化
  // ...
```

现在你的计算直接依赖于 `text`（这是一个字符串，不会“意外”变成别的值）。

---

### 为函数做 memoize {/*memoizing-a-function*/}

假设 `Form` 组件被 [`memo`](/reference/react/memo) 包裹。你想把一个函数作为 prop 传给它：

```js {2-7}
export default function ProductPage({ productId, referrer }) {
  function handleSubmit(orderDetails) {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails
    });
  }

  return <Form onSubmit={handleSubmit} />;
}
```

就像 `{}` 会创建一个不同的对象一样，函数声明如 `function() {}` 和表达式如 `() => {}` 在每次重新渲染时都会产生一个*不同*的函数。单独创建一个新函数并不是问题。这不是需要避免的事情！不过，如果 `Form` 组件已经被 memoize，那么你大概希望在 props 没变化时跳过它的重新渲染。一个*总是*不同的 prop 会抵消 memoization 的意义。

要用 `useMemo` 对函数进行 memoize，你的计算函数就必须返回另一个函数：

```js {2-3,8-9}
export default function Page({ productId, referrer }) {
  const handleSubmit = useMemo(() => {
    return (orderDetails) => {
      post('/product/' + productId + '/buy', {
        referrer,
        orderDetails
      });
    };
  }, [productId, referrer]);

  return <Form onSubmit={handleSubmit} />;
}
```

这看起来有点笨拙！**对函数进行 memoize 非常常见，因此 React 有一个专门内置的 Hook 来做这件事。请把你的函数包裹进 [`useCallback`](/reference/react/useCallback)，而不是 `useMemo`**，这样就不必额外写一个嵌套函数了：

```js {2,7}
export default function Page({ productId, referrer }) {
  const handleSubmit = useCallback((orderDetails) => {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails
    });
  }, [productId, referrer]);

  return <Form onSubmit={handleSubmit} />;
}
```

上面两个例子是完全等价的。`useCallback` 唯一的好处是它让你无需在内部再写一个额外的嵌套函数。它没有其他作用。[阅读更多关于 `useCallback` 的内容。](/reference/react/useCallback)

---

## 故障排查 {/*troubleshooting*/}

### 我的计算在每次重新渲染时都会运行两次 {/*my-calculation-runs-twice-on-every-re-render*/}

在 [严格模式](/reference/react/StrictMode) 下，React 会将你的某些函数调用两次，而不是一次：

```js {2,5,6}
function TodoList({ todos, tab }) {
  // 这个组件函数在每次渲染时都会运行两次。

  const visibleTodos = useMemo(() => {
    // 如果任何依赖项发生变化，这个计算会运行两次。
    return filterTodos(todos, tab);
  }, [todos, tab]);

  // ...
```

这是预期行为，不会破坏你的代码。

这种**仅限开发环境**的行为有助于你[保持组件纯粹。](/learn/keeping-components-pure) React 会使用其中一次调用的结果，并忽略另一次调用的结果。只要你的组件和计算函数是纯函数，这就不会影响你的逻辑。然而，如果它们不小心变成了不纯函数，这有助于你发现并修复错误。

例如，下面这个不纯的计算函数会修改你作为 prop 接收到的数组：

```js {2-3}
  const visibleTodos = useMemo(() => {
    // 🚩 错误：正在修改 prop
    todos.push({ id: 'last', text: '去散散步！' });
    const filtered = filterTodos(todos, tab);
    return filtered;
  }, [todos, tab]);
```

React 会调用你的函数两次，所以你会注意到 todo 被添加了两次。你的计算不应该改变任何现有对象，但可以修改你在计算过程中创建的任何*新*对象。例如，如果 `filterTodos` 函数总是返回一个*不同*的数组，你可以改为修改那个数组：

```js {3,4}
  const visibleTodos = useMemo(() => {
    const filtered = filterTodos(todos, tab);
    // ✅ 正确：修改你在计算过程中创建的对象
    filtered.push({ id: 'last', text: '去散散步！' });
    return filtered;
  }, [todos, tab]);
```

阅读[保持组件纯粹](/learn/keeping-components-pure)以了解更多关于纯度的内容。

另外，也可以查看关于在不使用 mutation 的情况下[更新对象](/learn/updating-objects-in-state)和[更新数组](/learn/updating-arrays-in-state)的指南。

---

### 我的 `useMemo` 调用本应返回一个对象，但却返回了 undefined {/*my-usememo-call-is-supposed-to-return-an-object-but-returns-undefined*/}

这段代码不起作用：

```js {1-2,5}
  // 🔴 你不能在 `() => {` 的箭头函数中直接返回对象
  const searchOptions = useMemo(() => {
    matchMode: 'whole-word',
    text: text
  }, [text]);
```

在 JavaScript 中，`() => {` 会开始箭头函数体，所以 `{` 大括号不是你的对象的一部分。这就是它为什么不会返回对象，并导致错误。你可以通过添加括号来修复，比如 `({` 和 `})`：

```js {1-2,5}
  // 这样可以工作，但很容易被别人再次破坏
  const searchOptions = useMemo(() => ({
    matchMode: 'whole-word',
    text: text
  }), [text]);
```

不过，这样仍然会让人困惑，而且很容易因为去掉括号而再次出错。

为了避免这个错误，请显式写出 `return` 语句：

```js {1-3,6-7}
  // ✅ 这样可以工作，而且很明确
  const searchOptions = useMemo(() => {
    return {
      matchMode: 'whole-word',
      text: text
    };
  }, [text]);
```

---

### 每次我的组件渲染时，`useMemo` 中的计算都会重新运行 {/*every-time-my-component-renders-the-calculation-in-usememo-re-runs*/}

确保你已经将依赖数组作为第二个参数传入！

如果你忘了传依赖数组，`useMemo` 会在每次渲染时重新运行计算：

```js {2-3}
function TodoList({ todos, tab }) {
  // 🔴 每次都重新计算：没有依赖数组
  const visibleTodos = useMemo(() => filterTodos(todos, tab));
  // ...
```

下面是作为第二个参数传入依赖数组后的修正版：

```js {2-3}
function TodoList({ todos, tab }) {
  // ✅ 不会进行不必要的重新计算
  const visibleTodos = useMemo(() => filterTodos(todos, tab), [todos, tab]);
  // ...
```

如果这仍然没有帮助，那么问题在于你的某个依赖项至少有一个与上一次渲染时不同。你可以通过手动把依赖项打印到控制台来调试这个问题：

```js
  const visibleTodos = useMemo(() => filterTodos(todos, tab), [todos, tab]);
  console.log([todos, tab]);
```

然后你可以在控制台中右键点击不同重新渲染时的数组，并选择“Store as a global variable”将它们都保存为全局变量。假设第一个保存为 `temp1`，第二个保存为 `temp2`，你就可以使用浏览器控制台检查两个数组中的每个依赖项是否相同：

```js
Object.is(temp1[0], temp2[0]); // 第一个依赖项在两个数组中是否相同？
Object.is(temp1[1], temp2[1]); // 第二个依赖项在两个数组中是否相同？
Object.is(temp1[2], temp2[2]); // ……依此类推，检查每一个依赖项……
```

当你找到破坏 memoization 的那个依赖项后，要么想办法移除它，要么[也把它进行 memoize。](#memoizing-a-dependency-of-another-hook)

---

### 我需要在循环中为每个列表项调用 `useMemo`，但这是不允许的 {/*i-need-to-call-usememo-for-each-list-item-in-a-loop-but-its-not-allowed*/}

假设 `Chart` 组件被 [`memo`](/reference/react/memo) 包裹了。你希望在 `ReportList` 组件重新渲染时跳过列表中每个 `Chart` 的重新渲染。然而，你不能在循环中调用 `useMemo`：

```js {expectedErrors: {'react-compiler': [6]}} {5-11}
function ReportList({ items }) {
  return (
    <article>
      {items.map(item => {
        // 🔴 你不能像这样在循环中调用 useMemo：
        const data = useMemo(() => calculateReport(item), [item]);
        return (
          <figure key={item.id}>
            <Chart data={data} />
          </figure>
        );
      })}
    </article>
  );
}
```

相反，为每个条目提取一个组件，并为单个条目的数据进行 memoize：

```js {5,12-18}
function ReportList({ items }) {
  return (
    <article>
      {items.map(item =>
        <Report key={item.id} item={item} />
      )}
    </article>
  );
}

function Report({ item }) {
  // ✅ 在顶层调用 useMemo：
  const data = useMemo(() => calculateReport(item), [item]);
  return (
    <figure>
      <Chart data={data} />
    </figure>
  );
}
```

或者，你也可以移除 `useMemo`，改为将 `Report` 本身包裹在 [`memo`.](/reference/react/memo) 中。如果 `item` prop 没有变化，`Report` 就会跳过重新渲染，因此 `Chart` 也会跳过重新渲染：

```js {5,6,12}
function ReportList({ items }) {
  // ...
}

const Report = memo(function Report({ item }) {
  const data = calculateReport(item);
  return (
    <figure>
      <Chart data={data} />
    </figure>
  );
});
```
