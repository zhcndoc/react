---
title: createContext
---

<Intro>

`createContext` 允许你创建一个 [context](/learn/passing-data-deeply-with-context)，组件可以提供或读取它。

```js
const SomeContext = createContext(defaultValue)
```

</Intro>

<InlineToc />

---

## 参考 {/*reference*/}

### `createContext(defaultValue)` {/*createcontext*/}

在任何组件外部调用 `createContext` 来创建一个 context。

```js
import { createContext } from 'react';

const ThemeContext = createContext('light');
```

[查看更多示例。](#usage)

#### 参数 {/*parameters*/}

* `defaultValue`：当读取 context 的组件上方树中没有匹配的 context 提供者时，你希望该 context 具有的值。如果你没有任何有意义的默认值，请指定 `null`。默认值是作为“最后手段”的回退值。它是静态的，且不会随着时间变化。

#### 返回值 {/*returns*/}

`createContext` 返回一个 context 对象。

**context 对象本身不持有任何信息。** 它表示其他组件读取或提供的是 _哪一个_ context。通常，你会在上方组件中使用 [`SomeContext`](#provider) 来指定 context 值，并在下方组件中调用 [`useContext(SomeContext)`](/reference/react/useContext) 来读取它。context 对象有几个属性：

* `SomeContext` 允许你向组件提供 context 值。
* `SomeContext.Consumer` 是一种替代且很少使用的读取 context 值的方式。
* `SomeContext.Provider` 是在 React 19 之前提供 context 值的旧式方式。

---

### `SomeContext` Provider {/*provider*/}

将你的组件包裹在一个 context provider 中，为其中所有组件指定该 context 的值：

```js
function App() {
  const [theme, setTheme] = useState('light');
  // ...
  return (
    <ThemeContext value={theme}>
      <Page />
    </ThemeContext>
  );
}
```

<Note>

从 React 19 开始，你可以将 `<SomeContext>` 作为 provider 来渲染。

在较旧版本的 React 中，请使用 `<SomeContext.Provider>`。

</Note>

#### Props {/*provider-props*/}

* `value`：你希望传递给该 provider 内所有读取此 context 的组件的值，无论它们有多深。context 值可以是任意类型。在 provider 内部调用 [`useContext(SomeContext)`](/reference/react/useContext) 的组件会接收到其上方最近的对应 context provider 的 `value`。

---

### `SomeContext.Consumer` {/*consumer*/}

在 `useContext` 出现之前，有一种较旧的读取 context 的方式：

```js
function Button() {
  // 🟡 旧式方式（不推荐）
  return (
    <ThemeContext.Consumer>
      {theme => (
        <button className={theme} />
      )}
    </ThemeContext.Consumer>
  );
}
```

虽然这种旧方式仍然可用，**但新编写的代码应该改为使用 [`useContext()`](/reference/react/useContext) 来读取 context：**

```js
function Button() {
  // ✅ 推荐方式
  const theme = useContext(ThemeContext);
  return <button className={theme} />;
}
```

#### Props {/*consumer-props*/}

* `children`：一个函数。React 会使用与 [`useContext()`](/reference/react/useContext) 相同的算法确定当前 context 值，并以此调用你传入的函数，然后渲染该函数返回的结果。只要父组件中的 context 发生变化，React 还会重新运行此函数并更新 UI。

---

## 用法 {/*usage*/}

### 创建 context {/*creating-context*/}

Context 允许组件在不显式传递 props 的情况下[向深层传递信息](/learn/passing-data-deeply-with-context)。

在任何组件外部调用 `createContext` 来创建一个或多个 context。

```js [[1, 3, "ThemeContext"], [1, 4, "AuthContext"], [3, 3, "'light'"], [3, 4, "null"]]
import { createContext } from 'react';

const ThemeContext = createContext('light');
const AuthContext = createContext(null);
```

`createContext` 返回一个 <CodeStep step={1}>context 对象</CodeStep>。组件可以通过将其传递给 [`useContext()`](/reference/react/useContext) 来读取 context：

```js [[1, 2, "ThemeContext"], [1, 7, "AuthContext"]]
function Button() {
  const theme = useContext(ThemeContext);
  // ...
}

function Profile() {
  const currentUser = useContext(AuthContext);
  // ...
}
```

默认情况下，它们接收到的值将是你在创建这些 context 时指定的 <CodeStep step={3}>默认值</CodeStep>。不过，这单独并没有什么用，因为默认值永远不会改变。

Context 很有用，因为你可以**从组件中提供其他动态值：**

```js {8-9,11-12}
function App() {
  const [theme, setTheme] = useState('dark');
  const [currentUser, setCurrentUser] = useState({ name: 'Taylor' });

  // ...

  return (
    <ThemeContext value={theme}>
      <AuthContext value={currentUser}>
        <Page />
      </AuthContext>
    </ThemeContext>
  );
}
```

现在，`Page` 组件以及其中的任何组件，无论多深，都将“看到”传入的 context 值。如果传入的 context 值发生变化，React 也会重新渲染读取该 context 的组件。

[阅读更多关于读取和提供 context 的内容并查看示例。](/reference/react/useContext)

---

### 从文件中导入和导出 context {/*importing-and-exporting-context-from-a-file*/}

通常，不同文件中的组件需要访问同一个 context。这就是为什么通常会将 context 声明在单独的文件中。然后你可以使用 [`export` 语句](https://developer.mozilla.org/en-US/docs/web/javascript/reference/statements/export) 让其他文件可以使用该 context：

```js {4-5}
// Contexts.js
import { createContext } from 'react';

export const ThemeContext = createContext('light');
export const AuthContext = createContext(null);
```

在其他文件中声明的组件随后可以使用 [`import`](https://developer.mozilla.org/en-US/docs/web/javascript/reference/statements/import) 语句来读取或提供这个 context：

```js {2}
// Button.js
import { ThemeContext } from './Contexts.js';

function Button() {
  const theme = useContext(ThemeContext);
  // ...
}
```

```js {2}
// App.js
import { ThemeContext, AuthContext } from './Contexts.js';

function App() {
  // ...
  return (
    <ThemeContext value={theme}>
      <AuthContext value={currentUser}>
        <Page />
      </AuthContext>
    </ThemeContext>
  );
}
```

这与[导入和导出组件](/learn/importing-and-exporting-components)的工作方式类似。

---

## 故障排除 {/*troubleshooting*/}

### 我找不到更改 context 值的方法 {/*i-cant-find-a-way-to-change-the-context-value*/}


像这样的代码指定的是 context 的*默认*值：

```js
const ThemeContext = createContext('light');
```

这个值永远不会改变。React 只有在上方找不到匹配的 provider 时才会使用这个值作为回退。

要让 context 随时间变化，[添加 state 并将组件包裹在 context provider 中。](/reference/react/useContext#updating-data-passed-via-context)
