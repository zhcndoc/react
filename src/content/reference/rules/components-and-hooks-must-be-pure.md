---
title: 组件和 Hooks 必须是纯的
---

<Intro>
纯函数只执行计算，不做其他任何事情。它让你的代码更容易理解和调试，并且让 React 能够自动正确地优化你的组件和 Hooks。
</Intro>

<Note>
本参考页面涵盖高级主题，并且需要熟悉 [保持组件纯净](/learn/keeping-components-pure) 页面中介绍的概念。
</Note>

<InlineToc />

### 为什么纯度很重要？ {/*why-does-purity-matter*/}

让 React 与众不同的一个关键概念是 _纯度_。纯组件或纯 Hook 具备以下特性：

* **幂等** – 只要输入相同——组件的输入是 props、state、context；Hook 的输入是参数——你[每次运行都会得到相同的结果](/learn/keeping-components-pure#purity-components-as-formulas)。
* **在渲染中没有副作用** – 带有副作用的代码应该[**与渲染分开**](#how-does-react-run-your-code)运行。例如作为 [事件处理函数](/learn/responding-to-events)——用户与 UI 交互并触发更新时执行；或者作为 [Effect](/reference/react/useEffect)——在渲染后执行。
* **不改变非局部值**：组件和 Hooks 在渲染时[绝不应该修改不是在本地创建的值](#mutation)。

当渲染保持纯净时，React 就能理解如何优先处理哪些更新对用户来说最重要。这之所以可行，是因为渲染纯度：由于组件在[渲染中](#how-does-react-run-your-code)没有副作用，React 可以暂停那些不那么重要的组件渲染，并在稍后需要时再回来继续处理它们。

具体来说，这意味着渲染逻辑可以多次执行，从而让 React 为用户提供更好的体验。不过，如果你的组件存在未被跟踪的副作用——例如在[渲染期间](#how-does-react-run-your-code)修改全局变量的值——那么当 React 再次运行你的渲染代码时，你的副作用会以与你期望不符的方式被触发。这通常会导致意外的 bug，并降低用户对应用的体验。你可以在 [保持组件纯净](/learn/keeping-components-pure#side-effects-unintended-consequences) 页面中看到一个[示例]。

#### React 如何运行你的代码？ {/*how-does-react-run-your-code*/}

React 是声明式的：你告诉 React _要渲染什么_，而 React 会决定 _如何最好地_ 将其展示给用户。为此，React 有若干阶段来运行你的代码。你不需要了解所有这些阶段就能很好地使用 React。但从高层来看，你应该知道哪些代码运行在 _render_ 中，以及哪些代码运行在它之外。

_Rendering_ 指的是计算 UI 的下一版应该是什么样子。渲染完成后，React 会拿这个新的计算结果与用于创建上一版 UI 的计算结果进行比较。然后 React 只将应用更改所需的最小改动提交到 [DOM](https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model)（也就是用户实际看到的内容）中。最后，[Effects](/learn/synchronizing-with-effects) 会被刷新（意思是它们会一直运行，直到没有更多为止）。更详细的信息请查看 [Render](/learn/render-and-commit) 和 [Commit and Effect Hooks](/reference/react/hooks#effect-hooks) 的文档。

<DeepDive>

#### 如何判断代码是否在 render 中运行 {/*how-to-tell-if-code-runs-in-render*/}

判断代码是否在渲染期间运行的一个简单经验法则是检查它放在哪里：如果它像下面的示例一样写在顶层，那么它很可能是在渲染期间运行的。

```js {2}
function Dropdown() {
  const selectedItems = new Set(); // 在渲染期间创建
  // ...
}
```

事件处理函数和 Effects 不会在渲染中运行：

```js {4}
function Dropdown() {
  const selectedItems = new Set();
  const onSelect = (item) => {
    // 这段代码位于事件处理函数中，所以它只会在用户触发该事件时运行
    selectedItems.add(item);
  }
}
```

```js {4}
function Dropdown() {
  const selectedItems = new Set();
  useEffect(() => {
    // 这段代码位于 Effect 内部，所以它只会在渲染之后运行
    logForAnalytics(selectedItems);
  }, [selectedItems]);
}
```
</DeepDive>

---

## 组件和 Hooks 必须是幂等的 {/*components-and-hooks-must-be-idempotent*/}

组件必须始终针对它们的输入——props、state 和 context——返回相同的输出。这被称为 _幂等性_。[幂等性](https://en.wikipedia.org/wiki/Idempotence) 是一个在函数式编程中被广泛使用的术语。它指的是：只要输入相同，你[每次运行都会得到相同的结果](learn/keeping-components-pure)。

这意味着，为了满足这条规则，[渲染期间](#how-does-react-run-your-code)运行的 _所有_ 代码也必须是幂等的。例如，下面这行代码就不是幂等的（因此这个组件也不是）：

```js {2}
function Clock() {
  const time = new Date(); // 🔴 不好：总是返回不同的结果！
  return <span>{time.toLocaleString()}</span>
}
```

`new Date()` 不是幂等的，因为它总是返回当前日期，并且每次调用都会改变结果。当你渲染上面的组件时，屏幕上显示的时间会一直停留在组件被渲染时的那个时间。同样，像 `Math.random()` 这样的函数也不是幂等的，因为即使输入相同，每次调用都会返回不同的结果。

这并不意味着你根本不应该使用像 `new Date()` 这样的非幂等函数——你只是应该避免在[渲染期间](#how-does-react-run-your-code)使用它们。在这种情况下，我们可以使用一个 [Effect](/reference/react/useEffect) 将最新日期同步到这个组件中：

<Sandpack>

```js
import { useState, useEffect } from 'react';

function useTime() {
  // 1. 追踪当前日期的状态。`useState` 会接收一个初始化函数作为
  //    初始状态。它只会在 Hook 被调用时执行一次，所以最初只会设置
  //    Hook 被调用时的当前日期。
  const [time, setTime] = useState(() => new Date());

  useEffect(() => {
    // 2. 使用 `setInterval` 每秒更新一次当前日期。
    const id = setInterval(() => {
      setTime(new Date()); // ✅ 好：非幂等代码不再在渲染中运行
    }, 1000);
    // 3. 返回一个清理函数，这样我们就不会泄漏 `setInterval` 定时器。
    return () => clearInterval(id);
  }, []);

  return time;
}

export default function Clock() {
  const time = useTime();
  return <span>{time.toLocaleString()}</span>;
}
```

</Sandpack>

通过把非幂等的 `new Date()` 调用包裹在 Effect 中，它就被移到了[渲染之外](#how-does-react-run-your-code)。

如果你不需要将某些外部状态与 React 同步，也可以考虑使用 [事件处理函数](/learn/responding-to-events)，前提是它只需要在响应用户交互时更新。

---

## 副作用必须在渲染之外运行 {/*side-effects-must-run-outside-of-render*/}

[副作用](/learn/keeping-components-pure#side-effects-unintended-consequences)不应该在[渲染中](#how-does-react-run-your-code)运行，因为 React 可能会多次渲染组件，以创造最佳的用户体验。

<Note>
副作用是一个比 Effects 更宽泛的术语。Effects 具体指的是被 `useEffect` 包裹的代码，而副作用则是一个通用术语，指除了向调用者返回值这一主要结果之外，任何可观察到效果的代码。

副作用通常写在 [事件处理函数](/learn/responding-to-events) 或 Effects 中。但绝不能在渲染期间写入。
</Note>

虽然渲染必须保持纯净，但为了让你的应用做出任何有趣的事情——比如在屏幕上显示内容——在某个时刻副作用又是必不可少的！这条规则的关键点是副作用不应该在[渲染中](#how-does-react-run-your-code)运行，因为 React 可以多次渲染组件。在大多数情况下，你会使用 [事件处理函数](learn/responding-to-events) 来处理副作用。使用事件处理函数会明确告诉 React，这段代码不需要在渲染期间运行，从而保持渲染的纯净。如果你已经尝试了所有选项——并且这是最后的手段——你也可以使用 `useEffect` 来处理副作用。

### 什么时候可以有 mutation？ {/*mutation*/}

#### 局部 mutation {/*local-mutation*/}
副作用的一个常见例子是 mutation，在 JavaScript 中这指的是改变一个非 [原始类型](https://developer.mozilla.org/en-US/docs/Glossary/Primitive) 值的内容。一般来说，虽然 mutation 不是 React 的惯用写法，但_局部_ mutation 完全没问题：

```js {2,7}
function FriendList({ friends }) {
  const items = []; // ✅ 好：在本地创建
  for (let i = 0; i < friends.length; i++) {
    const friend = friends[i];
    items.push(
      <Friend key={friend.id} friend={friend} />
    ); // ✅ 好：局部 mutation 没问题
  }
  return <section>{items}</section>;
}
```

没有必要为了避免局部 mutation 而把代码写得很拧巴。这里也可以为了简洁使用 [`Array.map`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map)，但在[渲染期间](#how-does-react-run-your-code)创建一个本地数组然后往里面 push 项目并没有问题。

尽管看起来我们在修改 `items`，但关键要注意的是，这段代码只是在_局部_修改它——当组件再次渲染时，这种 mutation 不会被“记住”。换句话说，`items` 只会在组件存在期间一直存在。由于每次渲染 `<FriendList />` 时都会重新_创建_ `items`，所以这个组件总是会返回相同的结果。

另一方面，如果 `items` 是在组件外部创建的，它就会保留之前的值并记住这些变化：

```js {1,7}
const items = []; // 🔴 不好：在组件外部创建
function FriendList({ friends }) {
  for (let i = 0; i < friends.length; i++) {
    const friend = friends[i];
    items.push(
      <Friend key={friend.id} friend={friend} />
    ); // 🔴 不好：修改了在渲染外创建的值
  }
  return <section>{items}</section>;
}
```

当 `<FriendList />` 再次运行时，我们会在每次组件执行时继续把 `friends` 追加到 `items` 中，从而导致出现多个重复结果。这个版本的 `<FriendList />` 在[渲染期间](#how-does-react-run-your-code)会产生可观察到的副作用，并且**违反了这条规则**。

#### 惰性初始化 {/*lazy-initialization*/}

尽管并不完全“纯”，惰性初始化也是可以的：

```js {2}
function ExpenseForm() {
  SuperCalculator.initializeIfNotReady(); // ✅ 好：如果它不会影响其他组件
  // 继续渲染...
}
```

#### 更改 DOM {/*changing-the-dom*/}

直接对用户可见的副作用不允许出现在 React 组件的渲染逻辑中。换句话说，单纯调用一个组件函数本身就不应该在屏幕上产生变化。

```js {2}
function ProductDetailPage({ product }) {
  document.title = product.title; // 🔴 不好：更改了 DOM
}
```

在渲染之外实现更新 `document.title` 的一个方法是[使用 `document` 同步组件](/learn/synchronizing-with-effects)。

只要多次调用组件是安全的，并且不会影响其他组件的渲染，React 并不在意它是否在严格的函数式编程意义上 100% 纯。更重要的是，[组件必须是幂等的](/reference/rules/components-and-hooks-must-be-pure)。

---

## Props 和 state 是不可变的 {/*props-and-state-are-immutable*/}

组件的 props 和 state 是不可变的 [快照](learn/state-as-a-snapshot)。不要直接修改它们。相反，传递新的 props，并使用 `useState` 中的 setter 函数。

你可以把 props 和 state 的值看作是在渲染后更新的快照。正因如此，你不应直接修改 props 或 state 变量：而是传递新的 props，或者使用提供给你的 setter 函数告诉 React 该状态需要在组件下次渲染时更新。

### 不要修改 Props {/*props*/}
Props 是不可变的，因为如果你修改它们，应用程序会产生不一致的输出，这会很难调试，因为它可能在不同情况下工作，也可能不工作。

```js {expectedErrors: {'react-compiler': [2]}} {2}
function Post({ item }) {
  item.url = new Url(item.url, base); // 🔴 不好：绝不要直接修改 props
  return <Link url={item.url}>{item.title}</Link>;
}
```

```js {2}
function Post({ item }) {
  const url = new Url(item.url, base); // ✅ 好：改为创建副本
  return <Link url={url}>{item.title}</Link>;
}
```

### 不要修改 State {/*state*/}
`useState` 会返回 state 变量以及用于更新该 state 的 setter。

```js
const [stateVariable, setter] = useState(0);
```

我们不应该就地更新 state 变量，而是需要使用 `useState` 返回的 setter 函数来更新它。更改 state 变量上的值不会导致组件更新，这会让用户看到过时的 UI。使用 setter 函数会通知 React 状态已经改变，并且我们需要排队进行重新渲染以更新 UI。

```js {expectedErrors: {'react-compiler': [2, 5]}} {5}
function Counter() {
  const [count, setCount] = useState(0);

  function handleClick() {
    count = count + 1; // 🔴 不好：绝不要直接修改 state
  }

  return (
    <button onClick={handleClick}>
      You pressed me {count} times
    </button>
  );
}
```

```js {5}
function Counter() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1); // ✅ 好：使用 useState 返回的 setter 函数
  }

  return (
    <button onClick={handleClick}>
      You pressed me {count} times
    </button>
  );
}
```

---

## 传给 Hooks 的返回值和参数是不可变的 {/*return-values-and-arguments-to-hooks-are-immutable*/}

一旦值被传递给某个 Hook，你就不应修改它们。和 JSX 中的 props 一样，值在传入 Hook 后就变成了不可变的。

```js {expectedErrors: {'react-compiler': [4]}} {4}
function useIconStyle(icon) {
  const theme = useContext(ThemeContext);
  if (icon.enabled) {
    icon.className = computeStyle(icon, theme); // 🔴 不好：绝不要直接修改 Hook 参数
  }
  return icon;
}
```

```js {3}
function useIconStyle(icon) {
  const theme = useContext(ThemeContext);
  const newIcon = { ...icon }; // ✅ 好：改为创建副本
  if (icon.enabled) {
    newIcon.className = computeStyle(icon, theme);
  }
  return newIcon;
}
```

React 中一个重要的原则是 _局部推理_：即通过单独查看组件或 Hook 的代码，就能理解它的作用。Hook 在被调用时应被视为“黑盒”。例如，一个自定义 Hook 可能把它的参数作为依赖项，用于在内部对值进行记忆化：

```js {4}
function useIconStyle(icon) {
  const theme = useContext(ThemeContext);

  return useMemo(() => {
    const newIcon = { ...icon };
    if (icon.enabled) {
      newIcon.className = computeStyle(icon, theme);
    }
    return newIcon;
  }, [icon, theme]);
}
```

如果你修改了 Hook 的参数，自定义 Hook 的记忆化就会变得不正确，因此避免这样做非常重要。

```js {4}
style = useIconStyle(icon);         // `style` based on `icon` 进行了记忆化
icon.enabled = false;               // 不好：🔴 绝不要直接修改 Hook 参数
style = useIconStyle(icon);         // 返回之前记忆化的结果
```

```js {4}
style = useIconStyle(icon);         // `style` based on `icon` 进行了记忆化
icon = { ...icon, enabled: false }; // 好：✅ 改为创建副本
style = useIconStyle(icon);         // 计算 `style` 的新值
```

同样，Hook 的返回值也不要修改，因为它们可能已经被记忆化了。

---

## 传递给 JSX 后的值是不可变的 {/*values-are-immutable-after-being-passed-to-jsx*/}

不要在值已经用于 JSX 之后再修改它们。把修改移动到创建 JSX 之前。

当你在表达式中使用 JSX 时，React 可能会在组件完成渲染之前急切地求值 JSX。这意味着，在值被传递给 JSX 之后再修改它们，可能会导致 UI 过时，因为 React 不会知道要更新组件的输出。

```js {expectedErrors: {'react-compiler': [4]}} {4}
function Page({ colour }) {
  const styles = { colour, size: "large" };
  const header = <Header styles={styles} />;
  styles.size = "small"; // 🔴 不好：styles 已经在上面的 JSX 中使用过了
  const footer = <Footer styles={styles} />;
  return (
    <>
      {header}
      <Content />
      {footer}
    </>
  );
}
```

```js {4}
function Page({ colour }) {
  const headerStyles = { colour, size: "large" };
  const header = <Header styles={headerStyles} />;
  const footerStyles = { colour, size: "small" }; // ✅ 好：我们创建了一个新值
  const footer = <Footer styles={footerStyles} />;
  return (
    <>
      {header}
      <Content />
      {footer}
    </>
  );
}
```
