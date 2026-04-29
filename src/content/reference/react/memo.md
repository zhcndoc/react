---
title: memo
---

<Intro>

`memo` 让你在组件的 props 未变化时跳过重新渲染该组件。

```
const MemoizedComponent = memo(SomeComponent, arePropsEqual?)
```

</Intro>

<Note>

[React Compiler](/learn/react-compiler) 会自动将 `memo` 的等效优化应用到所有组件，从而减少手动记忆化的需要。你可以使用编译器自动处理组件记忆化。

</Note>

<InlineToc />

---

## 参考 {/*reference*/}

### `memo(Component, arePropsEqual?)` {/*memo*/}

将组件包裹在 `memo` 中，获得该组件的一个 *记忆化* 版本。只要 props 没有变化，这个记忆化版本的组件在其父组件重新渲染时通常不会重新渲染。但 React 仍然可能会重新渲染它：记忆化是一种性能优化，而不是保证。

```js
import { memo } from 'react';

const SomeComponent = memo(function SomeComponent(props) {
  // ...
});
```

[查看更多示例。](#usage)

#### 参数 {/*parameters*/}

* `Component`：你想要记忆化的组件。`memo` 不会修改这个组件，而是返回一个新的、记忆化的组件。任何有效的 React 组件都可以，包括函数组件和 [`forwardRef`](/reference/react/forwardRef) 组件。

* **可选** `arePropsEqual`：一个接受两个参数的函数：组件之前的 props 和新的 props。如果旧 props 和新 props 相等，也就是组件在新 props 下会渲染出相同的输出并表现出相同的行为时，它应该返回 `true`。否则它应该返回 `false`。通常你不需要指定这个函数。默认情况下，React 会使用 [`Object.is`.](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is) 比较每个 prop。

#### 返回值 {/*returns*/}

`memo` 返回一个新的 React 组件。它的行为与传给 `memo` 的组件相同，只是当其父组件重新渲染时，除非 props 发生变化，否则 React 不会总是重新渲染它。

---

## 用法 {/*usage*/}

### 在 props 未变化时跳过重新渲染 {/*skipping-re-rendering-when-props-are-unchanged*/}

React 通常会在组件的父组件重新渲染时重新渲染该组件。使用 `memo`，你可以创建一个组件：只要它的新 props 和旧 props 相同，React 就不会在父组件重新渲染时重新渲染它。这样的组件称为 *记忆化*。

要记忆化一个组件，把它包裹在 `memo` 中，并使用它返回的值代替原始组件：

```js
const Greeting = memo(function Greeting({ name }) {
  return <h1>Hello, {name}!</h1>;
});

export default Greeting;
```

React 组件应该始终具有[纯渲染逻辑。](/learn/keeping-components-pure) 这意味着如果它的 props、state 和 context 没有变化，它必须返回相同的输出。通过使用 `memo`，你是在告诉 React 你的组件符合这个要求，因此只要 props 没有变化，React 就不需要重新渲染。即使使用了 `memo`，如果组件自身的 state 变化，或者它使用的某个 context 变化，它仍然会重新渲染。

在这个示例中，注意 `Greeting` 组件会在 `name` 变化时重新渲染（因为它是 props 之一），但在 `address` 变化时不会重新渲染（因为它没有作为 prop 传给 `Greeting`）：

<Sandpack>

```js
import { memo, useState } from 'react';

export default function MyApp() {
  const [name, setName] = useState('');
  const [address, setAddress] = useState('');
  return (
    <>
      <label>
        名字{': '}
        <input value={name} onChange={e => setName(e.target.value)} />
      </label>
      <label>
        地址{': '}
        <input value={address} onChange={e => setAddress(e.target.value)} />
      </label>
      <Greeting name={name} />
    </>
  );
}

const Greeting = memo(function Greeting({ name }) {
  console.log("Greeting was rendered at", new Date().toLocaleTimeString());
  return <h3>你好{name && ', '}{name}!</h3>;
});
```

```css
label {
  display: block;
  margin-bottom: 16px;
}
```

</Sandpack>

<Note>

**你应该只把 `memo` 作为一种性能优化来依赖。** 如果没有它你的代码就不能工作，先找出根本问题并修复它。然后你可以添加 `memo` 来提升性能。

</Note>

<DeepDive>

#### 是否应该到处都加 memo？ {/*should-you-add-memo-everywhere*/}

如果你的应用像这个网站一样，而且大多数交互都比较粗粒度（比如替换整页或整个区域），那么通常不需要记忆化。另一方面，如果你的应用更像绘图编辑器，而且大多数交互都很细粒度（比如移动图形），那么你可能会发现记忆化非常有帮助。

只有当你的组件经常以完全相同的 props 重新渲染，并且其重新渲染逻辑很昂贵时，使用 `memo` 才有价值。如果组件重新渲染时没有明显的卡顿，那么 `memo` 是不必要的。请记住，如果传给组件的 props *总是不同*，那么 `memo` 完全没有用，例如你在渲染过程中传入了一个对象或一个普通函数。这就是为什么你经常需要把 [`useMemo`](/reference/react/useMemo#skipping-re-rendering-of-components) 和 [`useCallback`](/reference/react/useCallback#skipping-re-rendering-of-components) 与 `memo` 一起使用。

在其他情况下，把组件包裹在 `memo` 中没有任何好处。这样做也没有明显坏处，所以有些团队会选择不去考虑单个案例，而是尽可能多地进行记忆化。这样做的缺点是代码可读性会变差。另外，并非所有记忆化都有效：只要有一个“总是新建”的值，就足以破坏整个组件的记忆化。

**在实践中，遵循以下几个原则，可以让很多记忆化变得不再必要：**

1. 当一个组件在视觉上包裹了其他组件时，让它[接受作为 children 的 JSX。](/learn/passing-props-to-a-component#passing-jsx-as-children) 这样，当包装组件更新自身 state 时，React 就知道它的子组件不需要重新渲染。
1. 优先使用局部 state，不要比必要更进一步地[提升 state 的位置。](/learn/sharing-state-between-components) 例如，不要把表单这类临时状态，或者某个条目是否被悬停，保存在组件树的顶层或全局状态库中。
1. 保持你的[渲染逻辑纯净。](/learn/keeping-components-pure) 如果重新渲染某个组件会引发问题或产生明显的视觉瑕疵，那就是你的组件有 bug！应当修复 bug，而不是添加记忆化。
1. 避免[更新 state 的不必要 Effect。](/learn/you-might-not-need-an-effect) React 应用中大多数性能问题都源于 Effect 引发的一连串更新，导致组件反复渲染。
1. 尝试[移除 Effect 中不必要的依赖。](/learn/removing-effect-dependencies) 例如，与其做记忆化，不如把某些对象或函数移动到 Effect 内部，或移到组件外部。

如果某个特定交互仍然感觉卡顿，可以使用 [React Developer Tools profiler](https://legacy.reactjs.org/blog/2018/09/10/introducing-the-react-profiler.html) 查看哪些组件最值得进行记忆化，并在需要的地方添加记忆化。这些原则会让你的组件更容易调试和理解，所以无论如何都值得遵循。从长远来看，我们正在研究[自动进行细粒度记忆化](https://www.youtube.com/watch?v=lGEMwh32soc)，以一劳永逸地解决这个问题。

</DeepDive>

---

### 使用 state 更新记忆化组件 {/*updating-a-memoized-component-using-state*/}

即使一个组件被记忆化了，当它自身的 state 变化时，它仍然会重新渲染。记忆化只与从父组件传入该组件的 props 有关。

<Sandpack>

```js
import { memo, useState } from 'react';

export default function MyApp() {
  const [name, setName] = useState('');
  const [address, setAddress] = useState('');
  return (
    <>
      <label>
        名字{': '}
        <input value={name} onChange={e => setName(e.target.value)} />
      </label>
      <label>
        地址{': '}
        <input value={address} onChange={e => setAddress(e.target.value)} />
      </label>
      <Greeting name={name} />
    </>
  );
}

const Greeting = memo(function Greeting({ name }) {
  console.log('Greeting was rendered at', new Date().toLocaleTimeString());
  const [greeting, setGreeting] = useState('你好');
  return (
    <>
      <h3>{greeting}{name && ', '}{name}!</h3>
      <GreetingSelector value={greeting} onChange={setGreeting} />
    </>
  );
});

function GreetingSelector({ value, onChange }) {
  return (
    <>
      <label>
        <input
          type="radio"
          checked={value === '你好'}
          onChange={e => onChange('你好')}
        />
        常规问候语
      </label>
      <label>
        <input
          type="radio"
          checked={value === '你好，欢迎'}
          onChange={e => onChange('你好，欢迎')}
        />
        热情的问候语
      </label>
    </>
  );
}
```

```css
label {
  display: block;
  margin-bottom: 16px;
}
```

</Sandpack>

如果你把某个 state 变量设置为它当前的值，那么即使没有 `memo`，React 也会跳过重新渲染你的组件。你可能仍然会看到组件函数被额外调用一次，但结果会被丢弃。

---

### 使用 context 更新记忆化组件 {/*updating-a-memoized-component-using-a-context*/}

即使一个组件被记忆化了，当它使用的某个 context 变化时，它仍然会重新渲染。记忆化只与从父组件传入该组件的 props 有关。

<Sandpack>

```js
import { createContext, memo, useContext, useState } from 'react';

const ThemeContext = createContext(null);

export default function MyApp() {
  const [theme, setTheme] = useState('dark');

  function handleClick() {
    setTheme(theme === 'dark' ? 'light' : 'dark');
  }

  return (
    <ThemeContext value={theme}>
      <button onClick={handleClick}>
        切换主题
      </button>
      <Greeting name="Taylor" />
    </ThemeContext>
  );
}

const Greeting = memo(function Greeting({ name }) {
  console.log("Greeting was rendered at", new Date().toLocaleTimeString());
  const theme = useContext(ThemeContext);
  return (
    <h3 className={theme}>你好，{name}!</h3>
  );
});
```

```css
label {
  display: block;
  margin-bottom: 16px;
}

.light {
  color: black;
  background-color: white;
}

.dark {
  color: white;
  background-color: black;
}
```

</Sandpack>

如果你只想让组件在某个 context 的一部分变化时重新渲染，可以把组件拆成两部分。在外层组件中读取你需要的 context 内容，然后把它作为 prop 传给一个记忆化的子组件。

---

### 最小化 props 变化 {/*minimizing-props-changes*/}

当你使用 `memo` 时，只要某个 prop 与之前相比不是 *浅相等*，组件就会重新渲染。这意味着 React 会使用 [`Object.is`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is) 比较，把组件中的每个 prop 和它之前的值进行比较。注意，`Object.is(3, 3)` 是 `true`，但 `Object.is({}, {})` 是 `false`。


要充分发挥 `memo` 的作用，请尽量减少 props 变化的次数。例如，如果 prop 是一个对象，可以使用 [`useMemo`:](/reference/react/useMemo) 防止父组件每次都重新创建该对象：

```js {5-8}
function Page() {
  const [name, setName] = useState('Taylor');
  const [age, setAge] = useState(42);

  const person = useMemo(
    () => ({ name, age }),
    [name, age]
  );

  return <Profile person={person} />;
}

const Profile = memo(function Profile({ person }) {
  // ...
});
```

更好的减少 props 变化的方法，是确保组件在 props 中只接收最少必要的信息。例如，可以传递单独的值，而不是整个对象：

```js {4,7}
function Page() {
  const [name, setName] = useState('Taylor');
  const [age, setAge] = useState(42);
  return <Profile name={name} age={age} />;
}

const Profile = memo(function Profile({ name, age }) {
  // ...
});
```

即使是单独的值，有时也可以进一步投影为变化频率更低的值。例如，这里组件接收的是一个表示是否存在某个值的布尔值，而不是该值本身：

```js {3}
function GroupsLanding({ person }) {
  const hasGroups = person.groups !== null;
  return <CallToAction hasGroups={hasGroups} />;
}

const CallToAction = memo(function CallToAction({ hasGroups }) {
  // ...
});
```

当你需要向记忆化组件传递一个函数时，要么把它声明在组件外部，这样它就永远不会变化；要么使用 [`useCallback`](/reference/react/useCallback#skipping-re-rendering-of-components) 在多次重新渲染之间缓存它的定义。

---

### 指定自定义比较函数 {/*specifying-a-custom-comparison-function*/}

在少数情况下，减少记忆化组件的 props 变化可能不可行。在这种情况下，你可以提供一个自定义比较函数，React 会用它来比较旧 props 和新 props，而不是使用浅比较。这个函数作为第二个参数传给 `memo`。只有当新 props 会产生与旧 props 相同的输出时，它才应该返回 `true`；否则返回 `false`。

```js {3}
const Chart = memo(function Chart({ dataPoints }) {
  // ...
}, arePropsEqual);

function arePropsEqual(oldProps, newProps) {
  return (
    oldProps.dataPoints.length === newProps.dataPoints.length &&
    oldProps.dataPoints.every((oldPoint, index) => {
      const newPoint = newProps.dataPoints[index];
      return oldPoint.x === newPoint.x && oldPoint.y === newPoint.y;
    })
  );
}
```

如果你这样做，请使用浏览器开发者工具中的 Performance 面板，确保你的比较函数确实比重新渲染组件更快。你可能会感到惊讶。

在进行性能测量时，请确保 React 运行在生产模式下。

<Pitfall>

如果你提供了自定义的 `arePropsEqual` 实现，**你必须比较每个 prop，包括函数。** 函数经常会[闭包捕获](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures)父组件的 props 和 state。如果当 `oldProps.onClick !== newProps.onClick` 时你返回 `true`，那么你的组件会在其 `onClick` 处理函数中持续“看到”上一次渲染时的 props 和 state，从而导致非常令人困惑的 bug。

除非你百分之百确定正在处理的数据结构具有已知且有限的深度，否则不要在 `arePropsEqual` 中进行深度相等比较。**深度相等比较可能会变得极其缓慢**，如果之后有人修改了数据结构，它甚至会让你的应用卡死好几秒。

</Pitfall>

---

### 使用 React Compiler 后我还需要 React.memo 吗？ {/*react-compiler-memo*/}

当你启用 [React Compiler](/learn/react-compiler) 时，通常不再需要 `React.memo` 了。编译器会自动为你优化组件重新渲染。

下面是它的工作方式：

**没有 React Compiler 时**，你需要 `React.memo` 来防止不必要的重新渲染：

```js
// 父组件每秒重新渲染一次
function Parent() {
  const [seconds, setSeconds] = useState(0);

  useEffect(() => {
    const interval = setInterval(() => {
      setSeconds(s => s + 1);
    }, 1000);
    return () => clearInterval(interval);
  }, []);

  return (
    <>
      <h1>秒数：{seconds}</h1>
      <ExpensiveChild name="John" />
    </>
  );
}

// 没有 memo，即使 props 没变，这里也会每秒重新渲染一次
const ExpensiveChild = memo(function ExpensiveChild({ name }) {
  console.log('ExpensiveChild rendered');
  return <div>你好，{name}！</div>;
});
```

**启用 React Compiler 后**，同样的优化会自动发生：

```js
// 不需要 memo - 编译器会自动防止重新渲染
function ExpensiveChild({ name }) {
  console.log('ExpensiveChild rendered');
  return <div>你好，{name}！</div>;
}
```

下面是 React Compiler 生成内容的关键部分：

```js {6-12}
function Parent() {
  const $ = _c(7);
  const [seconds, setSeconds] = useState(0);
  // ... 其他代码 ...

  let t3;
  if ($[4] === Symbol.for("react.memo_cache_sentinel")) {
    t3 = <ExpensiveChild name="John" />;
    $[4] = t3;
  } else {
    t3 = $[4];
  }
  // ... return statement ...
}
```

注意高亮的这些行：编译器把 `<ExpensiveChild name="John" />` 包裹在一个缓存检查中。由于 `name` prop 始终是 `"John"`，这个 JSX 只会创建一次，并在父组件每次重新渲染时复用。这正是 `React.memo` 所做的事情——它会在 props 没有变化时阻止子组件重新渲染。

React Compiler 会自动：
1. 跟踪传给 `ExpensiveChild` 的 `name` prop 没有变化
2. 复用之前创建的 `<ExpensiveChild name="John" />` JSX
3. 完全跳过 `ExpensiveChild` 的重新渲染

这意味着**当你使用 React Compiler 时，可以安全地从组件中移除 `React.memo`**。编译器会自动提供相同的优化，让你的代码更简洁、更易维护。

<Note>

编译器的优化实际上比 `React.memo` 更全面。它还会对组件内部的中间值和昂贵计算进行记忆化，这类似于在整个组件树中结合使用 `React.memo` 和 `useMemo`。

</Note>

---

## 故障排查 {/*troubleshooting*/}
### 当我的组件的某个 prop 是对象、数组或函数时会重新渲染 {/*my-component-rerenders-when-a-prop-is-an-object-or-array*/}

React 通过浅比较来比较旧的和新的 props：也就是说，它会判断每个新的 prop 是否与旧的 prop 引用相等。如果你在父组件每次重新渲染时都创建一个新的对象或数组，即使其中的每个单独元素都相同，React 仍然会认为它发生了变化。同样，如果你在渲染父组件时创建了一个新的函数，即使该函数具有相同的定义，React 也会认为它已经发生了变化。为了避免这种情况，[请简化 props，或在父组件中对 props 进行 memoize](#minimizing-props-changes)。
