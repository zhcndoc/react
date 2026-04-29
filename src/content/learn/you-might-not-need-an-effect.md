---
title: '你可能不需要 Effect'
---

<Intro>

Effect 是 React 范式中的一个“逃生舱口”。它们让你可以“走出” React，并将你的组件与某些外部系统同步，比如非 React 组件、网络，或者浏览器 DOM。如果没有涉及外部系统（例如，你只是想在某些 props 或 state 改变时更新组件的 state），你通常不需要 Effect。移除不必要的 Effect 会让你的代码更容易理解、运行更快，并且更不容易出错。

</Intro>

<YouWillLearn>

* 为什么以及如何从组件中移除不必要的 Effect
* 如何在不使用 Effect 的情况下缓存昂贵的计算
* 如何在不使用 Effect 的情况下重置和调整组件 state
* 如何在事件处理函数之间共享逻辑
* 哪些逻辑应该移动到事件处理函数中
* 如何通知父组件发生了变化

</YouWillLearn>

## 如何移除不必要的 Effect {/*how-to-remove-unnecessary-effects*/}

有两种常见情况，你不需要 Effect：

* **你不需要用 Effect 来为渲染转换数据。** 例如，假设你想在显示列表之前先对它进行过滤。你可能会倾向于写一个 Effect，在列表变化时更新某个 state 变量。然而，这样做效率很低。当你更新 state 时，React 会先调用你的组件函数，计算屏幕上应该显示什么。然后 React 会把这些变化 ["提交"](/learn/render-and-commit) 到 DOM，更新屏幕。接着 React 才会运行你的 Effect。如果你的 Effect *也* 立刻更新了 state，那整个过程就会从头再来一遍！为了避免不必要的渲染过程，应在组件顶层直接转换所有数据。只要你的 props 或 state 改变，这段代码就会自动重新运行。
* **你不需要用 Effect 来处理用户事件。** 例如，假设你想在用户购买商品时发送一个 `/api/buy` POST 请求并显示通知。在 Buy 按钮的点击事件处理函数里，你确切知道发生了什么。而等到 Effect 运行时，你并不知道用户 *做了什么*（例如，点击了哪个按钮）。这就是为什么你通常应该在相应的事件处理函数中处理用户事件。

你 *确实* 需要 Effect 来与外部系统进行 [同步](/learn/synchronizing-with-effects#what-are-effects-and-how-are-they-different-from-events)。例如，你可以编写一个 Effect，让 jQuery 组件与 React state 保持同步。你也可以使用 Effect 获取数据：例如，你可以让搜索结果与当前搜索词保持同步。请记住，现代 [框架](/learn/creating-a-react-app#full-stack-frameworks) 提供了比直接在组件中编写 Effect 更高效的内置数据获取机制。

为了帮助你形成正确的直觉，我们来看一些常见的具体示例！

### 根据 props 或 state 更新 state {/*updating-state-based-on-props-or-state*/}

假设你有一个包含两个 state 变量的组件：`firstName` 和 `lastName`。你想通过拼接它们来计算出 `fullName`。此外，你希望 `fullName` 在 `firstName` 或 `lastName` 变化时自动更新。你的第一反应可能是添加一个 `fullName` state 变量，并在 Effect 中更新它：

```js {expectedErrors: {'react-compiler': [8]}} {5-9}
function Form() {
  const [firstName, setFirstName] = useState('Taylor');
  const [lastName, setLastName] = useState('Swift');

  // 🔴 避免：冗余 state 和不必要的 Effect
  const [fullName, setFullName] = useState('');
  useEffect(() => {
    setFullName(firstName + ' ' + lastName);
  }, [firstName, lastName]);
  // ...
}
```

这比必要的要复杂得多。它也很低效：它会先用 `fullName` 的旧值完成一次完整渲染，然后马上用更新后的值重新渲染。移除这个 state 变量和 Effect：

```js {4-5}
function Form() {
  const [firstName, setFirstName] = useState('Taylor');
  const [lastName, setLastName] = useState('Swift');
  // ✅ 好：在渲染期间计算
  const fullName = firstName + ' ' + lastName;
  // ...
}
```

**当某个值可以从现有的 props 或 state 计算出来时，[不要把它放进 state。](/learn/choosing-the-state-structure#avoid-redundant-state) 相反，应在渲染期间计算它。** 这样会让你的代码更快（你避免了额外的“级联”更新）、更简单（你移除了一些代码），并且更不容易出错（你避免了多个 state 变量彼此不同步引发的 bug）。如果这种做法对你来说还很新，[Thinking in React](/learn/thinking-in-react#step-3-find-the-minimal-but-complete-representation-of-ui-state) 会解释什么内容应该放入 state。

### 缓存昂贵的计算 {/*caching-expensive-calculations*/}

这个组件通过接收 props 中的 `todos` 并根据 `filter` prop 对其进行过滤，来计算 `visibleTodos`。你可能会想把结果存到 state 里，并通过 Effect 来更新它：

```js {expectedErrors: {'react-compiler': [7]}} {4-8}
function TodoList({ todos, filter }) {
  const [newTodo, setNewTodo] = useState('');

  // 🔴 避免：冗余 state 和不必要的 Effect
  const [visibleTodos, setVisibleTodos] = useState([]);
  useEffect(() => {
    setVisibleTodos(getFilteredTodos(todos, filter));
  }, [todos, filter]);

  // ...
}
```

和前面的例子一样，这既不必要，也低效。首先，移除 state 和 Effect：

```js {3-4}
function TodoList({ todos, filter }) {
  const [newTodo, setNewTodo] = useState('');
  // ✅ 如果 getFilteredTodos() 不慢，这样就没问题。
  const visibleTodos = getFilteredTodos(todos, filter);
  // ...
}
```

通常情况下，这段代码是没问题的！但也许 `getFilteredTodos()` 很慢，或者你的 `todos` 很多。在这种情况下，如果像 `newTodo` 这样的无关 state 变量发生变化，你就不希望重新计算 `getFilteredTodos()`。

你可以通过把昂贵的计算包裹在 [`useMemo`](/reference/react/useMemo) Hook 中来缓存（或 ["memoize"](https://en.wikipedia.org/wiki/Memoization)）它：

<Note>

[React Compiler](/learn/react-compiler) 可以自动为你缓存昂贵的计算，在很多情况下消除了手动使用 `useMemo` 的需要。

</Note>

```js {5-8}
import { useMemo, useState } from 'react';

function TodoList({ todos, filter }) {
  const [newTodo, setNewTodo] = useState('');
  const visibleTodos = useMemo(() => {
    // ✅ 只有 todos 或 filter 改变时才会重新运行
    return getFilteredTodos(todos, filter);
  }, [todos, filter]);
  // ...
}
```

或者，写成一行：

```js {5-6}
import { useMemo, useState } from 'react';

function TodoList({ todos, filter }) {
  const [newTodo, setNewTodo] = useState('');
  // ✅ 只有 todos 或 filter 改变时才会重新运行 getFilteredTodos()
  const visibleTodos = useMemo(() => getFilteredTodos(todos, filter), [todos, filter]);
  // ...
}
```

**这告诉 React，除非 `todos` 或 `filter` 有变化，否则你不希望内部函数重新运行。** React 会在初始渲染时记住 `getFilteredTodos()` 的返回值。在下一次渲染时，它会检查 `todos` 或 `filter` 是否不同。如果和上次一样，`useMemo` 就会返回它缓存的上一个结果。但如果它们不同，React 就会再次调用内部函数（并缓存其结果）。

你包裹在 [`useMemo`](/reference/react/useMemo) 中的函数是在渲染期间运行的，所以它只适用于 [纯计算。](/learn/keeping-components-pure)

<DeepDive>

#### 如何判断一个计算是否昂贵？ {/*how-to-tell-if-a-calculation-is-expensive*/}

一般来说，除非你在创建或遍历成千上万个对象，否则它大概率不算昂贵。如果你想更有把握，可以添加一个 console log 来测量某段代码花费的时间：

```js {1,3}
console.time('filter array');
const visibleTodos = getFilteredTodos(todos, filter);
console.timeEnd('filter array');
```

执行你正在测量的交互（例如，在输入框中打字）。然后你会在控制台里看到类似 `filter array: 0.15ms` 的日志。如果总计日志时间加起来很可观（比如 `1ms` 或更多），那可能就值得对这个计算进行缓存。作为实验，你可以把这个计算包裹进 `useMemo`，看看这次交互的总日志时间是否下降：

```js
console.time('filter array');
const visibleTodos = useMemo(() => {
  return getFilteredTodos(todos, filter); // 如果 todos 和 filter 没变，这里会被跳过
}, [todos, filter]);
console.timeEnd('filter array');
```

`useMemo` 不会让 *第一次* 渲染更快。它只会帮助你在更新时跳过不必要的工作。

请记住，你的机器可能比用户的机器快，所以最好使用人为的降速来测试性能。例如，Chrome 提供了 [CPU Throttling](https://developer.chrome.com/blog/new-in-devtools-61/#throttling) 选项。

另外要注意，在开发环境中测量性能不会给出最准确的结果。（例如，当 [Strict Mode](/reference/react/StrictMode) 开启时，你会看到每个组件渲染两次，而不是一次。）为了得到最准确的计时，请为生产环境构建你的应用，并在与你的用户相似的设备上进行测试。

</DeepDive>

### 当 prop 改变时重置所有 state {/*resetting-all-state-when-a-prop-changes*/}

这个 `ProfilePage` 组件接收一个 `userId` prop。页面中包含一个评论输入框，你用 `comment` state 变量保存它的值。有一天，你注意到一个问题：当你从一个个人资料页导航到另一个时，`comment` state 没有被重置。结果就是，很容易不小心把评论发到错误用户的主页上。为了解决这个问题，你想在 `userId` 变化时清空 `comment` state 变量：

```js {expectedErrors: {'react-compiler': [6]}} {4-7}
export default function ProfilePage({ userId }) {
  const [comment, setComment] = useState('');

  // 🔴 避免：在 Effect 中响应 prop 变化重置 state
  useEffect(() => {
    setComment('');
  }, [userId]);
  // ...
}
```

这很低效，因为 `ProfilePage` 及其子组件会先用旧值渲染一次，然后再渲染一次。它也很复杂，因为你需要在 `ProfilePage` 中所有包含 state 的组件里都这样做。例如，如果评论 UI 是嵌套的，你也需要清空嵌套的评论 state。

相反，你可以通过给每个用户的个人资料一个显式的 key，告诉 React：每个用户的资料在概念上是 _不同的_ 资料。把你的组件拆成两层，并从外层组件向内层组件传递 `key` 属性：

```js {5,11-12}
export default function ProfilePage({ userId }) {
  return (
    <Profile
      userId={userId}
      key={userId}
    />
  );
}

function Profile({ userId }) {
  // ✅ 这里以及下面的任何其他 state 都会在 key 改变时自动重置
  const [comment, setComment] = useState('');
  // ...
}
```

通常情况下，当同一个组件在同一个位置渲染时，React 会保留它的 state。**通过把 `userId` 作为 `key` 传给 `Profile` 组件，你是在要求 React 把两个 `userId` 不同的 `Profile` 组件视为两个不同的组件，它们不应该共享任何 state。** 每当 key（你把它设成了 `userId`）发生变化时，React 就会重新创建 DOM，并 [重置 state](/learn/preserving-and-resetting-state#option-2-resetting-state-with-a-key) 的 `Profile` 组件及其所有子组件。现在，在不同个人资料之间切换时，`comment` 字段会自动清空。

请注意，在这个例子里，只有外层的 `ProfilePage` 组件被导出并对项目中的其他文件可见。渲染 `ProfilePage` 的组件不需要把 key 传给它：它们只需把 `userId` 作为普通 prop 传入即可。`ProfilePage` 把它作为 `key` 传给内层 `Profile` 组件这一事实，是一个实现细节。

### 当 prop 改变时调整某些 state {/*adjusting-some-state-when-a-prop-changes*/}

有时，你可能想在 prop 变化时重置或调整 state 的一部分，而不是全部。

这个 `List` 组件接收一个 `items` 列表作为 prop，并在 `selection` state 变量中维护当前选中的项。你希望在 `items` prop 接收到一个不同的数组时，将 `selection` 重置为 `null`：

```js {expectedErrors: {'react-compiler': [7]}} {5-8}
function List({ items }) {
  const [isReverse, setIsReverse] = useState(false);
  const [selection, setSelection] = useState(null);

  // 🔴 避免：在 Effect 中响应 prop 变化调整 state
  useEffect(() => {
    setSelection(null);
  }, [items]);
  // ...
}
```

这也不理想。每次 `items` 变化时，`List` 及其子组件一开始都会带着旧的 `selection` 值渲染。然后 React 会更新 DOM 并运行 Effect。最后，`setSelection(null)` 的调用又会让 `List` 及其子组件重新渲染一次，整个过程再次重来。

先删除这个 Effect。相反，在渲染期间直接调整 state：

```js {5-11}
function List({ items }) {
  const [isReverse, setIsReverse] = useState(false);
  const [selection, setSelection] = useState(null);

  // 更好的方式：在渲染时调整 state
  const [prevItems, setPrevItems] = useState(items);
  if (items !== prevItems) {
    setPrevItems(items);
    setSelection(null);
  }
  // ...
}
```

像这样 [存储上一次渲染的信息](/reference/react/useState#storing-information-from-previous-renders) 可能不太容易理解，但它比在 Effect 中更新同一个 state 更好。在上面的例子中，`setSelection` 是在渲染过程中直接调用的。React 会在它执行到 `return` 语句并退出后 *立即* 重新渲染 `List`。React 还没有渲染 `List` 的子组件，也还没有更新 DOM，所以这样就能让 `List` 的子组件跳过使用旧 `selection` 值的渲染。

当你在渲染期间更新某个组件时，React 会丢弃返回的 JSX 并立即重试渲染。为了避免非常慢的级联重试，React 只允许你在一次渲染过程中更新 *同一个* 组件的 state。如果你在渲染过程中更新另一个组件的 state，你就会看到错误。像 `items !== prevItems` 这样的条件是避免循环所必需的。你可以像这样调整 state，但其他任何副作用（例如更改 DOM 或设置定时器）都应该留在事件处理函数或 Effect 中，以便 [保持组件纯粹。](/learn/keeping-components-pure)

**虽然这种模式比 Effect 更高效，但大多数组件其实也不需要它。** 无论你怎么做，基于 props 或其他 state 调整 state，都会让你的数据流更难理解和调试。请始终检查你是否可以改为 [使用 key 重置所有 state](#resetting-all-state-when-a-prop-changes) 或 [在渲染期间计算所有内容](#updating-state-based-on-props-or-state)。例如，与其存储（并重置）被选中的 *项目*，不如存储被选中的 *项目 ID：*

```js {3-5}
function List({ items }) {
  const [isReverse, setIsReverse] = useState(false);
  const [selectedId, setSelectedId] = useState(null);
  // ✅ 最佳：在渲染期间计算所有内容
  const selection = items.find(item => item.id === selectedId) ?? null;
  // ...
}
```

现在就完全不需要“调整” state 了。如果具有该选中 ID 的项目在列表中，它就保持选中状态。如果不在，渲染期间计算出的 `selection` 就会是 `null`，因为没有找到匹配的项目。这个行为虽然不同，但可以说更好，因为对 `items` 的大多数更改都会保留选中状态。

### 在事件处理函数之间共享逻辑 {/*sharing-logic-between-event-handlers*/}

假设你有一个商品页面，上面有两个按钮（Buy 和 Checkout），它们都能让用户购买该商品。你希望在用户把商品加入购物车时显示通知。在两个按钮的点击处理函数中都调用 `showNotification()` 会显得重复，所以你可能会想把这段逻辑放到一个 Effect 里：

```js {2-7}
function ProductPage({ product, addToCart }) {
  // 🔴 避免：把事件特定的逻辑放在 Effect 中
  useEffect(() => {
    if (product.isInCart) {
      showNotification(`Added ${product.name} to the shopping cart!`);
    }
  }, [product]);

  function handleBuyClick() {
    addToCart(product);
  }

  function handleCheckoutClick() {
    addToCart(product);
    navigateTo('/checkout');
  }
  // ...
}
```

这个 Effect 是不必要的。它还很可能引发 bug。例如，假设你的应用会在页面刷新之间“记住”购物车。如果你把某个商品加入购物车一次，然后刷新页面，通知又会再次出现。每次你刷新该商品页面时，它都会继续出现。这是因为页面加载时 `product.isInCart` 已经是 `true`，所以上面的 Effect 会调用 `showNotification()`。

**当你不确定某段代码应该放在 Effect 里还是事件处理函数里时，先问自己这段代码为什么需要运行。只有当代码应该 *因为* 组件被展示给用户而运行时，才使用 Effect。** 在这个例子里，通知应该出现是因为用户 *按下了按钮*，而不是因为页面被展示了！删除这个 Effect，并把共享逻辑放进一个由两个事件处理函数共同调用的函数中：

```js {2-6,9,13}
function ProductPage({ product, addToCart }) {
  // ✅ 好：事件特定逻辑由事件处理函数调用
  function buyProduct() {
    addToCart(product);
    showNotification(`Added ${product.name} to the shopping cart!`);
  }

  function handleBuyClick() {
    buyProduct();
  }

  function handleCheckoutClick() {
    buyProduct();
    navigateTo('/checkout');
  }
  // ...
}
```

这样既移除了不必要的 Effect，也修复了 bug。

### 发送 POST 请求 {/*sending-a-post-request*/}

这个 `Form` 组件会发送两种 POST 请求。它在挂载时发送一个分析事件。当你填写表单并点击 Submit 按钮时，它会向 `/api/register` 端点发送一个 POST 请求：

```js {5-8,10-16}
function Form() {
  const [firstName, setFirstName] = useState('');
  const [lastName, setLastName] = useState('');

  // ✅ 好：这段逻辑应该在组件被展示后运行
  useEffect(() => {
    post('/analytics/event', { eventName: 'visit_form' });
  }, []);

  // 🔴 避免：把事件特定的逻辑放在 Effect 中
  const [jsonToSubmit, setJsonToSubmit] = useState(null);
  useEffect(() => {
    if (jsonToSubmit !== null) {
      post('/api/register', jsonToSubmit);
    }
  }, [jsonToSubmit]);

  function handleSubmit(e) {
    e.preventDefault();
    setJsonToSubmit({ firstName, lastName });
  }
  // ...
}
```

让我们用和前面例子相同的标准来分析。

分析用的 POST 请求应该保留在 Effect 中。这是因为发送分析事件的 _原因_ 是表单被展示了。（它在开发环境中会触发两次，但关于如何处理这一点，请 [看这里](/learn/synchronizing-with-effects#sending-analytics)。）

然而，`/api/register` 的 POST 请求并不是由表单被 _展示_ 引起的。你只想在某一个特定时刻发送这个请求：也就是用户按下按钮时。它只应该发生在 _那一次特定交互_ 中。删除第二个 Effect，并把这个 POST 请求移动到事件处理函数里：

```js {12-13}
function Form() {
  const [firstName, setFirstName] = useState('');
  const [lastName, setLastName] = useState('');

  // ✅ 好：这段逻辑因为组件被展示而运行
  useEffect(() => {
    post('/analytics/event', { eventName: 'visit_form' });
  }, []);

  function handleSubmit(e) {
    e.preventDefault();
    // ✅ 好：事件特定逻辑放在事件处理函数里
    post('/api/register', { firstName, lastName });
  }
  // ...
}
```

当你决定把某些逻辑放进事件处理函数还是 Effect 时，你需要回答的主要问题是：从用户的角度看，这是什么类型的逻辑。如果这段逻辑是由某个特定交互触发的，就把它保留在事件处理函数中。如果它是由用户在屏幕上 _看到_ 组件触发的，就把它保留在 Effect 中。

### 计算链条 {/*chains-of-computations*/}

有时你可能会想把多个 Effect 串联起来，每个 Effect 都根据其他 state 调整一部分 state：

```js {7-29}
function Game() {
  const [card, setCard] = useState(null);
  const [goldCardCount, setGoldCardCount] = useState(0);
  const [round, setRound] = useState(1);
  const [isGameOver, setIsGameOver] = useState(false);

  // 🔴 避免：为了触发彼此而串联的 Effects
  useEffect(() => {
    if (card !== null && card.gold) {
      setGoldCardCount(c => c + 1);
    }
  }, [card]);

  useEffect(() => {
    if (goldCardCount > 3) {
      setRound(r => r + 1)
      setGoldCardCount(0);
    }
  }, [goldCardCount]);

  useEffect(() => {
    if (round > 5) {
      setIsGameOver(true);
    }
  }, [round]);

  useEffect(() => {
    alert('Good game!');
  }, [isGameOver]);

  function handlePlaceCard(nextCard) {
    if (isGameOver) {
      throw Error('Game already ended.');
    } else {
      setCard(nextCard);
    }
  }

  // ...
```

这段代码有两个问题。

第一个问题是它非常低效：这个组件（以及它的子组件）必须在链条中的每一次 `set` 调用之间重新渲染。在上面的例子中，最坏情况下（`setCard` → render → `setGoldCardCount` → render → `setRound` → render → `setIsGameOver` → render），下面的树会发生三次不必要的重新渲染。

第二个问题是，即使它不慢，随着代码演进，你也会遇到你写的这条“链条”不再适应新需求的情况。想象一下，你要增加一种方式来查看游戏历史中的每一步移动。你会通过把每个 state 变量更新为过去的某个值来实现它。然而，把 `card` state 设置为过去的值会再次触发那条 Effect 链，并改变你正在显示的数据。这类代码通常很死板，也很脆弱。

在这种情况下，最好是在渲染期间计算你能计算的部分，并在事件处理函数中调整 state：

```js {6-7,14-26}
function Game() {
  const [card, setCard] = useState(null);
  const [goldCardCount, setGoldCardCount] = useState(0);
  const [round, setRound] = useState(1);

  // ✅ 在渲染期间计算你能计算的部分
  const isGameOver = round > 5;

  function handlePlaceCard(nextCard) {
    if (isGameOver) {
      throw Error('Game already ended.');
    }

    // ✅ 在事件处理函数中计算所有下一个 state
    setCard(nextCard);
    if (nextCard.gold) {
      if (goldCardCount < 3) {
        setGoldCardCount(goldCardCount + 1);
      } else {
        setGoldCardCount(0);
        setRound(round + 1);
        if (round === 5) {
          alert('Good game!');
        }
      }
    }
  }

  // ...
```

这样会高效得多。而且，如果你实现了查看游戏历史的功能，现在你就可以把每个 state 变量设置为过去的某一步，而不会触发那个调整其他所有值的 Effect 链。如果你需要在多个事件处理函数之间复用逻辑，可以 [提取一个函数](#sharing-logic-between-event-handlers) 并在这些处理函数中调用它。

请记住，在事件处理函数内部，[state 的行为像一个快照。](/learn/state-as-a-snapshot) 例如，即使你调用了 `setRound(round + 1)`，`round` 变量仍然反映的是用户点击按钮时的值。如果你需要在计算中使用下一个值，请手动定义它，例如 `const nextRound = round + 1`。

在某些情况下，你 *无法* 在事件处理函数中直接计算下一个 state。例如，想象一个包含多个下拉框的表单，后一个下拉框的选项取决于前一个下拉框所选的值。这时，Effect 链就是合适的，因为你是在与网络同步。

### 初始化应用 {/*initializing-the-application*/}

有些逻辑只应该在应用加载时运行一次。

你可能会想把它放到顶层组件中的 Effect 里：

```js {2-6}
function App() {
  // 🔴 避免：包含只应该运行一次的逻辑的 Effects
  useEffect(() => {
    loadDataFromLocalStorage();
    checkAuthToken();
  }, []);
  // ...
}
```

然而，你很快会发现它在开发环境中 [会运行两次。](/learn/synchronizing-with-effects#how-to-handle-the-effect-firing-twice-in-development) 这可能会导致问题——例如，也许它会使认证 token 失效，因为这个函数并不是为被调用两次而设计的。一般来说，你的组件应该能够抵御被重新挂载。这也包括你的顶层 `App` 组件。

虽然它在生产环境中通常不会真正被重新挂载，但在所有组件中遵循相同的约束，会让代码更容易迁移和复用。如果某些逻辑必须是 *每次应用加载时运行一次*，而不是 *每次组件挂载时运行一次*，那就添加一个顶层变量来追踪它是否已经执行过：

```js {1,5-6,10}
let didInit = false;

function App() {
  useEffect(() => {
    if (!didInit) {
      didInit = true;
      // ✅ 每次应用加载时只运行一次
      loadDataFromLocalStorage();
      checkAuthToken();
    }
  }, []);
  // ...
}
```

你也可以在模块初始化时、应用渲染之前运行它：

```js {1,5}
if (typeof window !== 'undefined') { // 检查我们是否在浏览器中运行。
   // ✅ 每次应用加载时只运行一次
  checkAuthToken();
  loadDataFromLocalStorage();
}

function App() {
  // ...
}
```

顶层代码会在组件被导入时运行一次——即使它最终并没有被渲染。为了避免在导入任意组件时造成性能下降或出现令人惊讶的行为，不要过度使用这种模式。把应用级初始化逻辑保留在像 `App.js` 这样的根组件模块中，或者保留在应用入口点里。

### 通知父组件状态变化 {/*notifying-parent-components-about-state-changes*/}

假设你正在编写一个 `Toggle` 组件，它内部有一个 `isOn` state，可以是 `true` 或 `false`。它有几种切换方式（点击或拖动）。你希望在 `Toggle` 内部 state 变化时通知父组件，因此你暴露一个 `onChange` 事件，并在 Effect 中调用它：

```js {4-7}
function Toggle({ onChange }) {
  const [isOn, setIsOn] = useState(false);

  // 🔴 避免：onChange 处理函数运行得太晚
  useEffect(() => {
    onChange(isOn);
  }, [isOn, onChange])

  function handleClick() {
    setIsOn(!isOn);
  }

  function handleDragEnd(e) {
    if (isCloserToRightEdge(e)) {
      setIsOn(true);
    } else {
      setIsOn(false);
    }
  }

  // ...
}
```

和前面一样，这并不理想。`Toggle` 先更新自己的 state，React 再更新屏幕。然后 React 才运行 Effect，调用从父组件传入的 `onChange` 函数。此时父组件会更新自己的 state，从而开启另一次渲染过程。把所有事情放在一次过程里完成会更好。

删除这个 Effect，改为在同一个事件处理函数中更新 *两个* 组件的 state：

```js {5-7,11,16,18}
function Toggle({ onChange }) {
  const [isOn, setIsOn] = useState(false);

  function updateToggle(nextIsOn) {
    // ✅ 好：在引发更新的事件中完成所有更新
    setIsOn(nextIsOn);
    onChange(nextIsOn);
  }

  function handleClick() {
    updateToggle(!isOn);
  }

  function handleDragEnd(e) {
    if (isCloserToRightEdge(e)) {
      updateToggle(true);
    } else {
      updateToggle(false);
    }
  }

  // ...
}
```

采用这种方式时，`Toggle` 组件及其父组件都会在事件中更新 state。React 会将不同组件的 [批量更新](/learn/queueing-a-series-of-state-updates) 合并在一起，因此只会有一次渲染。

你也可能可以直接移除 state，而是从父组件接收 `isOn`：

```js {1,2}
// ✅ 也很好：组件完全由其父组件控制
function Toggle({ isOn, onChange }) {
  function handleClick() {
    onChange(!isOn);
  }

  function handleDragEnd(e) {
    if (isCloserToRightEdge(e)) {
      onChange(true);
    } else {
      onChange(false);
    }
  }

  // ...
}
```

["提升 state"](/learn/sharing-state-between-components) 会让父组件通过切换自身的 state 来完全控制 `Toggle`。这意味着父组件需要包含更多逻辑，但整体上需要担心的 state 会更少。每当你尝试让两个不同的 state 变量保持同步时，不妨考虑改为提升 state！

### 向父组件传递数据 {/*passing-data-to-the-parent*/}

这个 `Child` 组件获取一些数据，然后在 Effect 中把它传给 `Parent` 组件：

```js {9-14}
function Parent() {
  const [data, setData] = useState(null);
  // ...
  return <Child onFetched={setData} />;
}

function Child({ onFetched }) {
  const data = useSomeAPI();
  // 🔴 避免：在 Effect 中向父组件传递数据
  useEffect(() => {
    if (data) {
      onFetched(data);
    }
  }, [onFetched, data]);
  // ...
}
```

在 React 中，数据从父组件流向子组件。当你在屏幕上看到某些地方不对劲时，你可以沿着组件链向上追踪信息来源，直到找到哪个组件传错了 prop，或者持有错误的 state。当子组件在 Effect 中更新父组件的 state 时，数据流就变得非常难以追踪。既然子组件和父组件都需要相同的数据，就让父组件去获取这份数据，然后把它 *向下传递* 给子组件：

```js {4-5}
function Parent() {
  const data = useSomeAPI();
  // ...
  // ✅ 好：把数据向下传给子组件
  return <Child data={data} />;
}

function Child({ data }) {
  // ...
}
```

这更简单，也让数据流更可预测：数据从父组件流向子组件。

### 订阅外部 store {/*subscribing-to-an-external-store*/}

有时，你的组件可能需要订阅 React state 之外的数据。这些数据可能来自第三方库，或者浏览器内置 API。由于这些数据可以在 React 不知道的情况下发生变化，你需要手动让组件订阅它。这通常是通过 Effect 完成的，例如：

```js {2-17}
function useOnlineStatus() {
  // 不理想：在 Effect 中手动订阅 store
  const [isOnline, setIsOnline] = useState(true);
  useEffect(() => {
    function updateState() {
      setIsOnline(navigator.onLine);
    }

    updateState();

    window.addEventListener('online', updateState);
    window.addEventListener('offline', updateState);
    return () => {
      window.removeEventListener('online', updateState);
      window.removeEventListener('offline', updateState);
    };
  }, []);
  return isOnline;
}

function ChatIndicator() {
  const isOnline = useOnlineStatus();
  // ...
}
```

这里，组件订阅了一个外部数据源（在这个例子里，是浏览器的 `navigator.onLine` API）。由于这个 API 在服务器上不存在（所以不能用于初始 HTML），最初 state 被设为 `true`。每当浏览器中该数据源的值发生变化时，组件就会更新自己的 state。

虽然用 Effect 来做这件事很常见，但 React 也提供了一个专门用于订阅外部 store 的 Hook，使用它更合适。删除这个 Effect，并将其替换为对 [`useSyncExternalStore`](/reference/react/useSyncExternalStore) 的调用：

```js {11-16}
function subscribe(callback) {
  window.addEventListener('online', callback);
  window.addEventListener('offline', callback);
  return () => {
    window.removeEventListener('online', callback);
    window.removeEventListener('offline', callback);
  };
}

function useOnlineStatus() {
  // ✅ 好：使用内置 Hook 订阅外部 store
  return useSyncExternalStore(
    subscribe, // 只要你传入的是同一个函数，React 就不会重新订阅
    () => navigator.onLine, // 在客户端如何获取值
    () => true // 在服务器上如何获取值
  );
}

function ChatIndicator() {
  const isOnline = useOnlineStatus();
  // ...
}
```

与使用 Effect 手动把可变数据同步到 React state 相比，这种方式更不容易出错。通常，你会像上面的 `useOnlineStatus()` 那样编写一个自定义 Hook，这样就不需要在单个组件中重复这段代码。[阅读更多关于从 React 组件订阅外部 store 的内容。](/reference/react/useSyncExternalStore)

### 获取数据 {/*fetching-data*/}

很多应用会使用 Effect 来发起数据获取。写一个像这样的数据获取 Effect 很常见：

```js {5-10}
function SearchResults({ query }) {
  const [results, setResults] = useState([]);
  const [page, setPage] = useState(1);

  useEffect(() => {
    // 🔴 避免：没有清理逻辑的获取
    fetchResults(query, page).then(json => {
      setResults(json);
    });
  }, [query, page]);

  function handleNextPageClick() {
    setPage(page + 1);
  }
  // ...
}
```

你 *不* 需要把这个获取操作移动到事件处理函数中。

这看起来可能和前面那些“你需要把逻辑放进事件处理函数”的例子矛盾！然而，请考虑一下，真正发起获取的主要原因并不是 *打字这个事件*。搜索输入框常常会从 URL 中预先填充，用户也可能在不触碰输入框的情况下通过后退和前进来导航。

`page` 和 `query` 是从哪里来的并不重要。只要这个组件可见，你就希望让 `results` 与当前 `page` 和 `query` 的网络数据保持 [同步](/learn/synchronizing-with-effects)。这就是为什么这里应该使用 Effect。

不过，上面的代码有一个 bug。想象一下你快速输入 `"hello"`。那么 `query` 就会从 `"h"` 变成 `"he"`、`"hel"`、`"hell"`，再到 `"hello"`。这会触发多个独立的请求，但这些响应到达的顺序并没有保证。例如，`"hell"` 的响应可能会在 `"hello"` 的响应之后才到达。由于它最后调用了 `setResults()`，你最终展示的会是错误的搜索结果。这被称为 ["竞态条件"](https://en.wikipedia.org/wiki/Race_condition)：两个不同的请求彼此“竞争”，并且按你没预料到的顺序返回。

**要修复竞态条件，你需要 [添加一个清理函数](/learn/synchronizing-with-effects#fetching-data) 来忽略过时的响应：**

```js {5,7,9,11-13}
function SearchResults({ query }) {
  const [results, setResults] = useState([]);
  const [page, setPage] = useState(1);
  useEffect(() => {
    let ignore = false;
    fetchResults(query, page).then(json => {
      if (!ignore) {
        setResults(json);
      }
    });
    return () => {
      ignore = true;
    };
  }, [query, page]);

  function handleNextPageClick() {
    setPage(page + 1);
  }
  // ...
}
```

这可以确保当你的 Effect 获取数据时，除了最后一次请求的响应之外，其他所有响应都会被忽略。

处理竞态条件并不是实现数据获取时唯一的难点。你还可能需要考虑如何缓存响应（这样用户点击 Back 时可以立即看到之前的界面）、如何在服务器上获取数据（这样初始的服务端渲染 HTML 中就包含获取到的内容，而不是一个加载中指示器），以及如何避免网络瀑布（这样子组件就能在不等待每个父组件的情况下获取数据）。

**这些问题适用于任何 UI 库，不仅仅是 React。解决它们并不简单，这也是为什么现代 [框架](/learn/creating-a-react-app#full-stack-frameworks) 提供了比在 Effect 中获取数据更高效的内置数据获取机制。**

如果你不使用框架（也不想自己构建一个），但又希望让通过 Effect 获取数据更易用，可以考虑把获取逻辑提取到一个自定义 Hook 中，就像下面这个例子：

```js {4}
function SearchResults({ query }) {
  const [page, setPage] = useState(1);
  const params = new URLSearchParams({ query, page });
  const results = useData(`/api/search?${params}`);

  function handleNextPageClick() {
    setPage(page + 1);
  }
  // ...
}

function useData(url) {
  const [data, setData] = useState(null);
  useEffect(() => {
    let ignore = false;
    fetch(url)
      .then(response => response.json())
      .then(json => {
        if (!ignore) {
          setData(json);
        }
      });
    return () => {
      ignore = true;
    };
  }, [url]);
  return data;
}
```

你可能还会想添加一些错误处理逻辑，以及跟踪内容是否正在加载。你可以自己构建这样一个 Hook，或者使用 React 生态系统中已经可用的众多方案之一。**虽然仅靠这个并不会像使用框架内置的数据获取机制那样高效，但把数据获取逻辑移动到自定义 Hook 中，会让你以后更容易采用高效的数据获取策略。**

一般来说，每当你不得不写 Effect 时，都要留意是否可以把某个功能提取成一个自定义 Hook，并提供像上面 `useData` 那样更具声明性、专用性更强的 API。你的组件中原始的 `useEffect` 调用越少，应用就越容易维护。

<Recap>

- 如果你可以在渲染期间计算出某些东西，就不需要 Effect。
- 要缓存昂贵的计算，用 `useMemo`，而不是 `useEffect`。
- 要重置整个组件树的 state，就给它传入一个不同的 `key`。
- 要在 prop 变化时重置某个特定的 state，就在渲染期间设置它。
- 因为组件被 *展示* 而运行的代码应该放在 Effect 中，其余的应放在事件里。
- 如果你需要更新多个组件的 state，最好在单次事件中完成。
- 每当你尝试同步不同组件中的 state 变量时，考虑改为提升 state。
- 你可以使用 Effect 获取数据，但你需要实现清理逻辑以避免竞态条件。

</Recap>

<Challenges>

#### 不使用 Effect 转换数据 {/*transform-data-without-effects*/}

下面的 `TodoList` 显示一个待办事项列表。当勾选“仅显示未完成的待办事项”复选框时，已完成的待办事项不会显示在列表中。无论哪些待办事项可见，页脚都会显示尚未完成的待办事项数量。

通过移除所有不必要的 state 和 Effect 来简化这个组件。

<Sandpack>

```js {expectedErrors: {'react-compiler': [12, 16, 20]}}
import { useState, useEffect } from 'react';
import { initialTodos, createTodo } from './todos.js';

export default function TodoList() {
  const [todos, setTodos] = useState(initialTodos);
  const [showActive, setShowActive] = useState(false);
  const [activeTodos, setActiveTodos] = useState([]);
  const [visibleTodos, setVisibleTodos] = useState([]);
  const [footer, setFooter] = useState(null);

  useEffect(() => {
    setActiveTodos(todos.filter(todo => !todo.completed));
  }, [todos]);

  useEffect(() => {
    setVisibleTodos(showActive ? activeTodos : todos);
  }, [showActive, todos, activeTodos]);

  useEffect(() => {
    setFooter(
      <footer>
        {activeTodos.length} todos left
      </footer>
    );
  }, [activeTodos]);

  return (
    <>
      <label>
        <input
          type="checkbox"
          checked={showActive}
          onChange={e => setShowActive(e.target.checked)}
        />
        仅显示未完成的待办事项
      </label>
      <NewTodo onAdd={newTodo => setTodos([...todos, newTodo])} />
      <ul>
        {visibleTodos.map(todo => (
          <li key={todo.id}>
            {todo.completed ? <s>{todo.text}</s> : todo.text}
          </li>
        ))}
      </ul>
      {footer}
    </>
  );
}

function NewTodo({ onAdd }) {
  const [text, setText] = useState('');

  function handleAddClick() {
    setText('');
    onAdd(createTodo(text));
  }

  return (
    <>
      <input value={text} onChange={e => setText(e.target.value)} />
      <button onClick={handleAddClick}>
        添加
      </button>
    </>
  );
}
```

```js src/todos.js
let nextId = 0;

export function createTodo(text, completed = false) {
  return {
    id: nextId++,
    text,
    completed
  };
}

export const initialTodos = [
  createTodo('买苹果', true),
  createTodo('买橙子', true),
  createTodo('买胡萝卜'),
];
```

```css
label { display: block; }
input { margin-top: 10px; }
```

</Sandpack>

<Hint>

如果你可以在渲染期间计算出某些东西，你就不需要 state 或者更新它的 Effect。

</Hint>

<Solution>

在这个例子里，只有两个必要的 state：`todos` 列表，以及表示复选框是否被勾选的 `showActive` state 变量。其他所有 state 变量都是 [冗余的](/learn/choosing-the-state-structure#avoid-redundant-state)，都可以改为在渲染期间计算。这包括 `footer`，你可以把它直接放到外围 JSX 中。

你的结果应该如下所示：

<Sandpack>

```js
import { useState } from 'react';
import { initialTodos, createTodo } from './todos.js';

export default function TodoList() {
  const [todos, setTodos] = useState(initialTodos);
  const [showActive, setShowActive] = useState(false);
  const activeTodos = todos.filter(todo => !todo.completed);
  const visibleTodos = showActive ? activeTodos : todos;

  return (
    <>
      <label>
        <input
          type="checkbox"
          checked={showActive}
          onChange={e => setShowActive(e.target.checked)}
        />
        仅显示未完成的待办事项
      </label>
      <NewTodo onAdd={newTodo => setTodos([...todos, newTodo])} />
      <ul>
        {visibleTodos.map(todo => (
          <li key={todo.id}>
            {todo.completed ? <s>{todo.text}</s> : todo.text}
          </li>
        ))}
      </ul>
      <footer>
        {activeTodos.length} todos left
      </footer>
    </>
  );
}

function NewTodo({ onAdd }) {
  const [text, setText] = useState('');

  function handleAddClick() {
    setText('');
    onAdd(createTodo(text));
  }

  return (
    <>
      <input value={text} onChange={e => setText(e.target.value)} />
      <button onClick={handleAddClick}>
        添加
      </button>
    </>
  );
}
```

```js src/todos.js
let nextId = 0;

export function createTodo(text, completed = false) {
  return {
    id: nextId++,
    text,
    completed
  };
}

export const initialTodos = [
  createTodo('买苹果', true),
  createTodo('买橙子', true),
  createTodo('买胡萝卜'),
];
```

```css
label { display: block; }
input { margin-top: 10px; }
```

</Sandpack>

</Solution>

#### 不使用 Effect 缓存计算 {/*cache-a-calculation-without-effects*/}

在这个例子中，过滤待办事项被提取到了一个名为 `getVisibleTodos()` 的单独函数中。这个函数内部有一个 `console.log()` 调用，可以帮助你注意到它何时被调用。切换“仅显示未完成的待办事项”，你会注意到这会导致 `getVisibleTodos()` 重新运行。这是预期之中的，因为当你切换显示哪些项目时，可见的待办事项也会变化。

你的任务是移除 `TodoList` 组件中重新计算 `visibleTodos` 列表的 Effect。不过，你需要确保在你向输入框中输入内容时，`getVisibleTodos()` *不会* 重新运行（也就不会打印任何日志）。

<Hint>

一种解决方案是添加一个 `useMemo` 调用来缓存可见待办事项。还有另一种不那么明显的解决方案。

</Hint>

<Sandpack>

```js {expectedErrors: {'react-compiler': [11]}}
import { useState, useEffect } from 'react';
import { initialTodos, createTodo, getVisibleTodos } from './todos.js';

export default function TodoList() {
  const [todos, setTodos] = useState(initialTodos);
  const [showActive, setShowActive] = useState(false);
  const [text, setText] = useState('');
  const [visibleTodos, setVisibleTodos] = useState([]);

  useEffect(() => {
    setVisibleTodos(getVisibleTodos(todos, showActive));
  }, [todos, showActive]);

  function handleAddClick() {
    setText('');
    setTodos([...todos, createTodo(text)]);
  }

  return (
    <>
      <label>
        <input
          type="checkbox"
          checked={showActive}
          onChange={e => setShowActive(e.target.checked)}
        />
        仅显示未完成的待办事项
      </label>
      <input value={text} onChange={e => setText(e.target.value)} />
      <button onClick={handleAddClick}>
        添加
      </button>
      <ul>
        {visibleTodos.map(todo => (
          <li key={todo.id}>
            {todo.completed ? <s>{todo.text}</s> : todo.text}
          </li>
        ))}
      </ul>
    </>
  );
}
```

```js src/todos.js
let nextId = 0;
let calls = 0;

export function getVisibleTodos(todos, showActive) {
  console.log(`getVisibleTodos() was called ${++calls} times`);
  const activeTodos = todos.filter(todo => !todo.completed);
  const visibleTodos = showActive ? activeTodos : todos;
  return visibleTodos;
}

export function createTodo(text, completed = false) {
  return {
    id: nextId++,
    text,
    completed
  };
}

export const initialTodos = [
  createTodo('买苹果', true),
  createTodo('买橙子', true),
  createTodo('买胡萝卜'),
];
```

```css
label { display: block; }
input { margin-top: 10px; }
```

</Sandpack>

<Solution>

移除 state 变量和 Effect，然后添加一个 `useMemo` 调用来缓存调用 `getVisibleTodos()` 的结果：

<Sandpack>

```js
import { useState, useMemo } from 'react';
import { initialTodos, createTodo, getVisibleTodos } from './todos.js';

export default function TodoList() {
  const [todos, setTodos] = useState(initialTodos);
  const [showActive, setShowActive] = useState(false);
  const [text, setText] = useState('');
  const visibleTodos = useMemo(
    () => getVisibleTodos(todos, showActive),
    [todos, showActive]
  );

  function handleAddClick() {
    setText('');
    setTodos([...todos, createTodo(text)]);
  }

  return (
    <>
      <label>
        <input
          type="checkbox"
          checked={showActive}
          onChange={e => setShowActive(e.target.checked)}
        />
        仅显示未完成的待办事项
      </label>
      <input value={text} onChange={e => setText(e.target.value)} />
      <button onClick={handleAddClick}>
        添加
      </button>
      <ul>
        {visibleTodos.map(todo => (
          <li key={todo.id}>
            {todo.completed ? <s>{todo.text}</s> : todo.text}
          </li>
        ))}
      </ul>
    </>
  );
}
```

```js src/todos.js
let nextId = 0;
let calls = 0;

export function getVisibleTodos(todos, showActive) {
  console.log(`getVisibleTodos() was called ${++calls} times`);
  const activeTodos = todos.filter(todo => !todo.completed);
  const visibleTodos = showActive ? activeTodos : todos;
  return visibleTodos;
}

export function createTodo(text, completed = false) {
  return {
    id: nextId++,
    text,
    completed
  };
}

export const initialTodos = [
  createTodo('买苹果', true),
  createTodo('买橙子', true),
  createTodo('买胡萝卜'),
];
```

```css
label { display: block; }
input { margin-top: 10px; }
```

</Sandpack>

有了这个改动，只有当 `todos` 或 `showActive` 变化时，`getVisibleTodos()` 才会被调用。输入框里输入内容只会改变 `text` state 变量，因此不会触发对 `getVisibleTodos()` 的调用。

还有另一种不需要 `useMemo` 的解决方案。由于 `text` state 变量不可能影响待办事项列表，你可以把 `NewTodo` 表单提取成单独的组件，并把 `text` state 变量移到那个组件内部：

<Sandpack>

```js
import { useState, useMemo } from 'react';
import { initialTodos, createTodo, getVisibleTodos } from './todos.js';

export default function TodoList() {
  const [todos, setTodos] = useState(initialTodos);
  const [showActive, setShowActive] = useState(false);
  const visibleTodos = getVisibleTodos(todos, showActive);

  return (
    <>
      <label>
        <input
          type="checkbox"
          checked={showActive}
          onChange={e => setShowActive(e.target.checked)}
        />
        仅显示未完成的待办事项
      </label>
      <NewTodo onAdd={newTodo => setTodos([...todos, newTodo])} />
      <ul>
        {visibleTodos.map(todo => (
          <li key={todo.id}>
            {todo.completed ? <s>{todo.text}</s> : todo.text}
          </li>
        ))}
      </ul>
    </>
  );
}

function NewTodo({ onAdd }) {
  const [text, setText] = useState('');

  function handleAddClick() {
    setText('');
    onAdd(createTodo(text));
  }

  return (
    <>
      <input value={text} onChange={e => setText(e.target.value)} />
      <button onClick={handleAddClick}>
        添加
      </button>
    </>
  );
}
```

```js src/todos.js
let nextId = 0;
let calls = 0;

export function getVisibleTodos(todos, showActive) {
  console.log(`getVisibleTodos() was called ${++calls} times`);
  const activeTodos = todos.filter(todo => !todo.completed);
  const visibleTodos = showActive ? activeTodos : todos;
  return visibleTodos;
}

export function createTodo(text, completed = false) {
  return {
    id: nextId++,
    text,
    completed
  };
}

export const initialTodos = [
  createTodo('买苹果', true),
  createTodo('买橙子', true),
  createTodo('买胡萝卜'),
];
```

```css
label { display: block; }
input { margin-top: 10px; }
```

</Sandpack>

这个方法也满足要求。当你在输入框中输入时，只有 `text` 状态变量会更新。由于 `text` 状态变量位于子级 `NewTodo` 组件中，父级 `TodoList` 组件不会重新渲染。这就是你输入时 `getVisibleTodos()` 不会被调用的原因。（如果 `TodoList` 因其他原因重新渲染，它仍然会被调用。）

</Solution>

#### 无需 Effects 即可重置状态 {/*reset-state-without-effects*/}

这个 `EditContact` 组件通过 `savedContact` prop 接收一个形如 `{ id, name, email }` 的联系人对象。试着编辑姓名和邮箱输入框。当你点击 Save 时，表单上方联系人按钮会更新为你编辑后的姓名。当你点击 Reset 时，表单中的任何待处理更改都会被丢弃。你可以自己操作一下这个界面来感受它的行为。

当你使用顶部的按钮选择某个联系人时，表单会重置以反映该联系人的详细信息。这是通过 `EditContact.js` 中的一个 Effect 实现的。移除这个 Effect。想出另一种方法，当 `savedContact.id` 变化时重置表单。

<Sandpack>

```js src/App.js hidden
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
        savedContact={selectedContact}
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

```js src/ContactList.js hidden
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

```js {expectedErrors: {'react-compiler': [8, 9]}} src/EditContact.js active
import { useState, useEffect } from 'react';

export default function EditContact({ savedContact, onSave }) {
  const [name, setName] = useState(savedContact.name);
  const [email, setEmail] = useState(savedContact.email);

  useEffect(() => {
    setName(savedContact.name);
    setEmail(savedContact.email);
  }, [savedContact]);

  return (
    <section>
      <label>
        Name:{' '}
        <input
          type="text"
          value={name}
          onChange={e => setName(e.target.value)}
        />
      </label>
      <label>
        Email:{' '}
        <input
          type="email"
          value={email}
          onChange={e => setEmail(e.target.value)}
        />
      </label>
      <button onClick={() => {
        const updatedData = {
          id: savedContact.id,
          name: name,
          email: email
        };
        onSave(updatedData);
      }}>
        Save
      </button>
      <button onClick={() => {
        setName(savedContact.name);
        setEmail(savedContact.email);
      }}>
        Reset
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

<Hint>

如果有一种方式可以告诉 React：当 `savedContact.id` 不同的时候，`EditContact` 表单在概念上是 _另一个联系人的表单_，不应该保留状态，那就太好了。你记得有这种方式吗？

</Hint>

<Solution>

把 `EditContact` 组件拆成两个。将所有表单状态移到内部的 `EditForm` 组件中。导出外层的 `EditContact` 组件，并让它把 `savedContact.id` 作为 `key` 传给内部的 `EditForm` 组件。这样，当你选择不同的联系人时，内部的 `EditForm` 组件会重置所有表单状态，并重新创建 DOM。

<Sandpack>

```js src/App.js hidden
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
        savedContact={selectedContact}
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

```js src/ContactList.js hidden
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

```js src/EditContact.js active
import { useState } from 'react';

export default function EditContact(props) {
  return (
    <EditForm
      {...props}
      key={props.savedContact.id}
    />
  );
}

function EditForm({ savedContact, onSave }) {
  const [name, setName] = useState(savedContact.name);
  const [email, setEmail] = useState(savedContact.email);

  return (
    <section>
      <label>
        Name:{' '}
        <input
          type="text"
          value={name}
          onChange={e => setName(e.target.value)}
        />
      </label>
      <label>
        Email:{' '}
        <input
          type="email"
          value={email}
          onChange={e => setEmail(e.target.value)}
        />
      </label>
      <button onClick={() => {
        const updatedData = {
          id: savedContact.id,
          name: name,
          email: email
        };
        onSave(updatedData);
      }}>
        Save
      </button>
      <button onClick={() => {
        setName(savedContact.name);
        setEmail(savedContact.email);
      }}>
        Reset
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

#### 无需 Effects 即可提交表单 {/*submit-a-form-without-effects*/}

这个 `Form` 组件让你可以给朋友发送消息。当你提交表单时，`showForm` 状态变量会被设为 `false`。这会触发一个调用 `sendMessage(message)` 的 Effect，从而发送消息（你可以在控制台中看到）。消息发送后，你会看到一个 “Thank you” 对话框以及一个 “Open chat” 按钮，点击它可以回到表单。

你的应用用户发送的消息太多了。为了让聊天稍微更难一点，你决定先显示 “Thank you” 对话框，而不是表单。把 `showForm` 状态变量的初始值从 `true` 改成 `false`。一旦你这样改，控制台就会显示发送了一条空消息。这个逻辑里有问题！

这个问题的根本原因是什么？你又该如何修复它？

<Hint>

消息应该是因为用户看到了 “Thank you” 对话框才发送的吗？还是反过来？

</Hint>

<Sandpack>

```js
import { useState, useEffect } from 'react';

export default function Form() {
  const [showForm, setShowForm] = useState(true);
  const [message, setMessage] = useState('');

  useEffect(() => {
    if (!showForm) {
      sendMessage(message);
    }
  }, [showForm, message]);

  function handleSubmit(e) {
    e.preventDefault();
    setShowForm(false);
  }

  if (!showForm) {
    return (
      <>
        <h1>Thanks for using our services!</h1>
        <button onClick={() => {
          setMessage('');
          setShowForm(true);
        }}>
          Open chat
        </button>
      </>
    );
  }

  return (
    <form onSubmit={handleSubmit}>
      <textarea
        placeholder="Message"
        value={message}
        onChange={e => setMessage(e.target.value)}
      />
      <button type="submit" disabled={message === ''}>
        Send
      </button>
    </form>
  );
}

function sendMessage(message) {
  console.log('Sending message: ' + message);
}
```

```css
label, textarea { margin-bottom: 10px; display: block; }
```

</Sandpack>

<Solution>

`showForm` 状态变量决定是显示表单还是 “Thank you” 对话框。但是，你发送消息并不是因为 “Thank you” 对话框被 _显示_ 了。你希望在用户 _提交表单_ 时发送消息。删除这个误导性的 Effect，并把 `sendMessage` 调用移到 `handleSubmit` 事件处理函数中：

<Sandpack>

```js
import { useState, useEffect } from 'react';

export default function Form() {
  const [showForm, setShowForm] = useState(true);
  const [message, setMessage] = useState('');

  function handleSubmit(e) {
    e.preventDefault();
    setShowForm(false);
    sendMessage(message);
  }

  if (!showForm) {
    return (
      <>
        <h1>Thanks for using our services!</h1>
        <button onClick={() => {
          setMessage('');
          setShowForm(true);
        }}>
          Open chat
        </button>
      </>
    );
  }

  return (
    <form onSubmit={handleSubmit}>
      <textarea
        placeholder="Message"
        value={message}
        onChange={e => setMessage(e.target.value)}
      />
      <button type="submit" disabled={message === ''}>
        Send
      </button>
    </form>
  );
}

function sendMessage(message) {
  console.log('Sending message: ' + message);
}
```

```css
label, textarea { margin-bottom: 10px; display: block; }
```

</Sandpack>

请注意，在这个版本中，只有 _提交表单_（这是一个事件）才会发送消息。无论 `showForm` 初始设置为 `true` 还是 `false`，它都能同样正常工作。（把它设为 `false`，并注意不会有额外的控制台消息。）

</Solution>

</Challenges>
