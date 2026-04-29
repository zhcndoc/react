---
title: React 调用组件和 Hooks
---

<Intro>
React 负责在必要时渲染组件和 Hooks，以优化用户体验。它是声明式的：你在组件逻辑中告诉 React 需要渲染什么，React 会自行决定以最佳方式向用户展示它。
</Intro>

<InlineToc />

---

## 永远不要直接调用组件函数 {/*never-call-component-functions-directly*/}
组件只能在 JSX 中使用。不要把它们当作普通函数来调用。应由 React 来调用它们。

React 必须决定你的组件函数何时被调用 [在渲染期间](/reference/rules/components-and-hooks-must-be-pure#how-does-react-run-your-code)。在 React 中，你应该使用 JSX 来做到这一点。

```js {2}
function BlogPost() {
  return <Layout><Article /></Layout>; // ✅ 好：只在 JSX 中使用组件
}
```

```js {expectedErrors: {'react-compiler': [2]}} {2}
function BlogPost() {
  return <Layout>{Article()}</Layout>; // 🔴 坏：永远不要直接调用它们
}
```

如果组件包含 Hooks，当组件在循环中或有条件地被直接调用时，就很容易违反 [Hooks 规则](/reference/rules/rules-of-hooks)。

让 React 统一协调渲染还带来了许多好处：

* **组件不再只是函数。** React 可以通过与组件在树中的身份绑定的 Hooks，为它们增强诸如 _本地状态_ 之类的特性。
* **组件类型参与协调过程。** 通过让 React 调用你的组件，你也向它传达了更多关于树的概念结构的信息。例如，当你从渲染 `<Feed>` 切换到 `<Profile>` 页面时，React 不会尝试复用它们。
* **React 可以增强你的用户体验。** 例如，它可以让浏览器在组件调用之间处理一些工作，从而使重新渲染大型组件树不会阻塞主线程。
* **更好的调试体验。** 如果组件是一等公民，且库对它们有感知，我们就可以在开发环境中构建丰富的开发者工具用于审查。
* **更高效的协调。** React 可以精确决定树中哪些组件需要重新渲染，并跳过那些不需要的组件。这会让你的应用更快、更流畅。

---

## 永远不要把 Hooks 当作普通值传来传去 {/*never-pass-around-hooks-as-regular-values*/}

Hooks 只能在组件或 Hooks 内部调用。不要把它们当作普通值传来传去。

Hooks 允许你用 React 特性增强组件。它们应该始终作为函数被调用，而不应该作为普通值传递。这使得 _局部推理_ 成为可能，也就是开发者只需单独查看某个组件，就能理解它能做的一切。

违反这条规则会导致 React 无法自动优化你的组件。

### 不要动态地修改 Hook {/*dont-dynamically-mutate-a-hook*/}

Hooks 应尽可能保持“静态”。这意味着你不应该动态地修改它们。例如，这意味着你不应该编写高阶 Hooks：

```js {expectedErrors: {'react-compiler': [2, 3]}} {2}
function ChatInput() {
  const useDataWithLogging = withLogging(useData); // 🔴 坏：不要编写高阶 Hooks
  const data = useDataWithLogging();
}
```

Hooks 应该是不可变的，不应被修改。不要动态地修改 Hook，而是创建一个具有所需功能的静态版本 Hook。

```js {2,6}
function ChatInput() {
  const data = useDataWithLogging(); // ✅ 好：创建 Hook 的新版本
}

function useDataWithLogging() {
  // ... 创建 Hook 的新版本，并在这里内联逻辑
}
```

### 不要动态地使用 Hooks {/*dont-dynamically-use-hooks*/}

Hooks 也不应被动态使用：例如，不要在组件中通过传递 Hook 作为值来进行依赖注入：

```js {expectedErrors: {'react-compiler': [2]}} {2}
function ChatInput() {
  return <Button useData={useDataWithLogging} /> // 🔴 坏：不要把 Hooks 作为 props 传递
}
```

你应该始终将 Hook 的调用内联到那个组件中，并在其中处理所有逻辑。

```js {6}
function ChatInput() {
  return <Button />
}

function Button() {
  const data = useDataWithLogging(); // ✅ 好：直接使用 Hook
}

function useDataWithLogging() {
  // 如果有任何条件逻辑需要改变 Hook 的行为，应该将其内联到
  // Hook 中
}
```

这样一来，`<Button />` 就更容易理解和调试。当 Hooks 以动态方式使用时，会大大增加应用的复杂性，并阻碍局部推理，从长远来看会降低团队的生产力。这也会让你更容易意外破坏 [Hooks 规则](/reference/rules/rules-of-hooks)，即 Hooks 不应有条件地被调用。如果你发现自己需要在测试中 mock 组件，通常更好的做法是 mock 服务端来返回预设数据。如果可能的话，使用端到端测试来测试你的应用通常也更有效。

