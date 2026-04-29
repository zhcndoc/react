---
title: Hooks 的规则
---

<Intro>
Hooks 使用 JavaScript 函数定义，但它们代表一种特殊类型的可复用 UI 逻辑，并且在调用位置上有一定限制。
</Intro>

<InlineToc />

---

##  只在顶层调用 Hooks {/*only-call-hooks-at-the-top-level*/}

以 `use` 开头的函数在 React 中被称为 [*Hooks*](/reference/react)。

**不要在循环、条件、嵌套函数或 `try`/`catch`/`finally` 代码块中调用 Hooks。** 相反，请始终在 React 函数的顶层使用 Hooks，并且要在任何提前返回之前。只有在 React 正在渲染函数组件时才能调用 Hooks：

* ✅ 在 [函数组件](/learn/your-first-component) 的主体顶层调用它们。
* ✅ 在 [自定义 Hook](/learn/reusing-logic-with-custom-hooks) 的主体顶层调用它们。

```js{2-3,8-9}
function Counter() {
  // ✅ 好：位于函数组件的顶层
  const [count, setCount] = useState(0);
  // ...
}

function useWindowWidth() {
  // ✅ 好：位于自定义 Hook 的顶层
  const [width, setWidth] = useState(window.innerWidth);
  // ...
}
```

在其他任何情况下调用 Hooks（以 `use` 开头的函数）都**不**受支持，例如：

* 🔴 不要在条件或循环中调用 Hooks。
* 🔴 不要在条件 `return` 语句之后调用 Hooks。
* 🔴 不要在事件处理函数中调用 Hooks。
* 🔴 不要在类组件中调用 Hooks。
* 🔴 不要在传递给 `useMemo`、`useReducer` 或 `useEffect` 的函数中调用 Hooks。
* 🔴 不要在 `try`/`catch`/`finally` 代码块中调用 Hooks。

如果你违反这些规则，你可能会看到这个错误。

```js{3-4,11-12,20-21}
function Bad({ cond }) {
  if (cond) {
    // 🔴 不好：位于条件中（要修复，请把它移到外面！）
    const theme = useContext(ThemeContext);
  }
  // ...
}

function Bad() {
  for (let i = 0; i < 10; i++) {
    // 🔴 不好：位于循环中（要修复，请把它移到外面！）
    const theme = useContext(ThemeContext);
  }
  // ...
}

function Bad({ cond }) {
  if (cond) {
    return;
  }
  // 🔴 不好：在条件返回之后（要修复，请把它移到 return 之前！）
  const theme = useContext(ThemeContext);
  // ...
}

function Bad() {
  function handleClick() {
    // 🔴 不好：位于事件处理函数中（要修复，请把它移到外面！）
    const theme = useContext(ThemeContext);
  }
  // ...
}

function Bad() {
  const style = useMemo(() => {
    // 🔴 不好：位于 useMemo 中（要修复，请把它移到外面！）
    const theme = useContext(ThemeContext);
    return createStyle(theme);
  });
  // ...
}

class Bad extends React.Component {
  render() {
    // 🔴 不好：位于类组件中（要修复，请改为编写函数组件，而不是类！）
    useEffect(() => {})
    // ...
  }
}

function Bad() {
  try {
    // 🔴 不好：位于 try/catch/finally 代码块中（要修复，请把它移到外面！）
    const [x, setX] = useState(0);
  } catch {
    const [x, setX] = useState(1);
  }
}
```

你可以使用 [`eslint-plugin-react-hooks` 插件](https://www.npmjs.com/package/eslint-plugin-react-hooks) 来捕获这些错误。

<Note>

[自定义 Hooks](/learn/reusing-logic-with-custom-hooks) *可以* 调用其他 Hooks（这正是它们的全部目的）。这是可行的，因为自定义 Hooks 也应该只在函数组件渲染时被调用。

</Note>

---

## 只从 React 函数中调用 Hooks {/*only-call-hooks-from-react-functions*/}

不要从普通 JavaScript 函数中调用 Hooks。相反，你可以：

✅ 从 React 函数组件中调用 Hooks。
✅ 从 [自定义 Hooks](/learn/reusing-logic-with-custom-hooks#extracting-your-own-custom-hook-from-a-component) 中调用 Hooks。

遵循这条规则可以确保组件中所有有状态逻辑都能从其源代码中清晰可见。

```js {2,5}
function FriendList() {
  const [onlineStatus, setOnlineStatus] = useOnlineStatus(); // ✅
}

function setOnlineStatus() { // ❌ 不是组件也不是自定义 Hook！
  const [onlineStatus, setOnlineStatus] = useOnlineStatus();
}
```
