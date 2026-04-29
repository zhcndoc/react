---
title: Hooks 规则
---

你之所以来到这里，可能是因为你遇到了以下错误信息：

<ConsoleBlock level="error">

Hooks 只能在函数组件的主体内部被调用。

</ConsoleBlock>

你可能看到它有三个常见原因：

1. 你可能**违反了 Hooks 规则**。
2. 你可能使用了**不匹配版本**的 React 和 React DOM。
3. 你在同一个应用中可能有**不止一份 React 副本**。

让我们逐一看看这些情况。

## 违反 Hooks 规则 {/*breaking-rules-of-hooks*/}

名字以 `use` 开头的函数在 React 中被称为 [*Hooks*](/reference/react)。

**不要在循环、条件语句或嵌套函数中调用 Hooks。** 相反，请始终在你的 React 函数的顶层调用 Hooks，在任何提前返回之前。你只能在 React 正在渲染函数组件时调用 Hooks：

* ✅ 在 [函数组件](/learn/your-first-component) 的主体顶层调用它们。
* ✅ 在 [自定义 Hook](/learn/reusing-logic-with-custom-hooks) 的主体顶层调用它们。

```js{2-3,8-9}
function Counter() {
  // ✅ 好：在函数组件的顶层
  const [count, setCount] = useState(0);
  // ...
}

function useWindowWidth() {
  // ✅ 好：在自定义 Hook 的顶层
  const [width, setWidth] = useState(window.innerWidth);
  // ...
}
```

在其他任何情况下调用 Hooks（以 `use` 开头的函数）都是**不被支持的**，例如：

* 🔴 不要在条件语句或循环中调用 Hooks。
* 🔴 不要在条件 `return` 语句之后调用 Hooks。
* 🔴 不要在事件处理器中调用 Hooks。
* 🔴 不要在类组件中调用 Hooks。
* 🔴 不要在传递给 `useMemo`、`useReducer` 或 `useEffect` 的函数内部调用 Hooks。

如果你违反这些规则，可能会看到这个错误。

```js{3-4,11-12,20-21}
function Bad({ cond }) {
  if (cond) {
    // 🔴 坏：在条件语句内部（要修复，请把它移到外面！）
    const theme = useContext(ThemeContext);
  }
  // ...
}

function Bad() {
  for (let i = 0; i < 10; i++) {
    // 🔴 坏：在循环内部（要修复，请把它移到外面！）
    const theme = useContext(ThemeContext);
  }
  // ...
}

function Bad({ cond }) {
  if (cond) {
    return;
  }
  // 🔴 坏：在条件 return 之后（要修复，请把它移到 return 之前！）
  const theme = useContext(ThemeContext);
  // ...
}

function Bad() {
  function handleClick() {
    // 🔴 坏：在事件处理器内部（要修复，请把它移到外面！）
    const theme = useContext(ThemeContext);
  }
  // ...
}

function Bad() {
  const style = useMemo(() => {
    // 🔴 坏：在 useMemo 内部（要修复，请把它移到外面！）
    const theme = useContext(ThemeContext);
    return createStyle(theme);
  });
  // ...
}

class Bad extends React.Component {
  render() {
    // 🔴 坏：在类组件内部（要修复，请改写为函数组件，而不是类！）
    useEffect(() => {})
    // ...
  }
}
```

你可以使用 [`eslint-plugin-react-hooks` 插件](https://www.npmjs.com/package/eslint-plugin-react-hooks) 来捕获这些错误。

<Note>

[自定义 Hooks](/learn/reusing-logic-with-custom-hooks) *可以*调用其他 Hooks（这正是它们存在的目的）。这是有效的，因为自定义 Hooks 也应该只在函数组件渲染时被调用。

</Note>

## React 和 React DOM 版本不匹配 {/*mismatching-versions-of-react-and-react-dom*/}

你可能正在使用一个尚不支持 Hooks 的 `react-dom`（< 16.8.0）或 `react-native`（< 0.59）版本。你可以在你的应用文件夹中运行 `npm ls react-dom` 或 `npm ls react-native` 来检查你正在使用的版本。如果你发现它们中有多个，也可能会引发问题（下面会详细说明）。

## 重复的 React {/*duplicate-react*/}

为了让 Hooks 正常工作，你的应用代码中的 `react` 导入需要解析到与 `react-dom` 包内部的 `react` 导入相同的模块。

如果这些 `react` 导入解析到两个不同的导出对象，你就会看到这个警告。这可能发生在你**不小心得到了两份** `react` 包的时候。

如果你使用 Node 进行包管理，可以在项目文件夹中运行以下检查：

<TerminalBlock>

npm ls react

</TerminalBlock>

如果你看到不止一个 React，你就需要弄清楚为什么会这样，并修复你的依赖树。例如，你正在使用的某个库可能错误地将 `react` 指定为依赖项（而不是 peer dependency）。在那个库修复之前，[Yarn resolutions](https://yarnpkg.com/lang/en/docs/selective-version-resolutions/) 是一种可能的解决办法。

你也可以通过添加一些日志并重启开发服务器来尝试调试这个问题：

```js
// 将此添加到 node_modules/react-dom/index.js 中
window.React1 = require('react');

// 将此添加到你的组件文件中
require('react-dom');
window.React2 = require('react');
console.log(window.React1 === window.React2);
```

如果它打印出 `false`，那么你可能有两个 React，需要弄清楚这是为什么。[这个问题](https://github.com/facebook/react/issues/13991)包含了社区遇到的一些常见原因。

当你使用 `npm link` 或类似工具时，这个问题也可能出现。在这种情况下，你的打包器可能会“看到”两个 React——一个在应用文件夹中，一个在你的库文件夹中。假设 `myapp` 和 `mylib` 是相邻文件夹，一种可能的修复方法是在 `mylib` 中运行 `npm link ../myapp/node_modules/react`。这应该会让该库使用应用中的 React 副本。

<Note>

通常，React 支持在同一页面上使用多个彼此独立的副本（例如，如果一个应用和一个第三方小部件都在使用它）。只有当组件中的 `require('react')` 与其渲染所使用的 `react-dom` 副本之间解析结果不同时，才会出问题。

</Note>

## 其他原因 {/*other-causes*/}

如果以上方法都不起作用，请在[这个 issue](https://github.com/facebook/react/issues/13991) 中留言，我们会尽力帮助你。尽量创建一个能重现问题的小示例——你可能会在这样做的过程中发现问题所在。
