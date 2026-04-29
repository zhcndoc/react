---
title: React 规则
---

<Intro>
就像不同的编程语言有各自表达概念的方式一样，React 也有自己的惯用法——或者说规则——来以易于理解并能产出高质量应用的方式表达各种模式。
</Intro>

<InlineToc />

---

<Note>
要进一步了解如何使用 React 表达 UI，我们建议阅读 [Thinking in React](/learn/thinking-in-react)。
</Note>

本节描述了编写符合 React 惯用法代码所需遵循的规则。编写符合惯用法的 React 代码可以帮助你编写组织良好、安全且可组合的应用。这些特性会让你的应用更能适应变化，也更容易与其他开发者、库和工具协作。

这些规则被称为 **React 规则**。它们之所以称为规则——而不仅仅是建议——是因为如果被违反，你的应用很可能会出现 bug。你的代码也会变得不符合惯用法，更难理解和推理。

我们强烈建议将 [Strict Mode](/reference/react/StrictMode) 与 React 的 [ESLint plugin](https://www.npmjs.com/package/eslint-plugin-react-hooks) 一起使用，以帮助你的代码库遵循 React 规则。通过遵循 React 规则，你将能够发现并修复这些 bug，并保持应用的可维护性。

---

## 组件和 Hook 必须是纯的 {/*components-and-hooks-must-be-pure*/}

[组件和 Hook 中的纯度](/reference/rules/components-and-hooks-must-be-pure) 是 React 的一条关键规则，它能让你的应用可预测、易于调试，并允许 React 自动优化你的代码。

* [组件必须是幂等的](/reference/rules/components-and-hooks-must-be-pure#components-and-hooks-must-be-idempotent) – React 假定组件始终会针对其输入——props、state 和 context——返回相同的输出。
* [副作用必须在渲染之外运行](/reference/rules/components-and-hooks-must-be-pure#side-effects-must-run-outside-of-render) – 副作用不应在渲染过程中运行，因为 React 可能会多次渲染组件，以创造最佳的用户体验。
* [Props 和 state 是不可变的](/reference/rules/components-and-hooks-must-be-pure#props-and-state-are-immutable) – 组件的 props 和 state 是相对于单次渲染的不可变快照。永远不要直接修改它们。
* [Hooks 的返回值和参数是不可变的](/reference/rules/components-and-hooks-must-be-pure#return-values-and-arguments-to-hooks-are-immutable) – 一旦值被传递给 Hook，你就不应再修改它们。像 JSX 中的 props 一样，值在传给 Hook 后就变成不可变的。
* [值在传递给 JSX 后是不可变的](/reference/rules/components-and-hooks-must-be-pure#values-are-immutable-after-being-passed-to-jsx) – 不要在值已用于 JSX 之后再修改它们。应将修改移动到 JSX 创建之前。

---

## React 会调用组件和 Hooks {/*react-calls-components-and-hooks*/}

[React 负责在必要时渲染组件和 hook，以优化用户体验。](/reference/rules/react-calls-components-and-hooks) 它是声明式的：你在组件逻辑中告诉 React 要渲染什么，而 React 会自行决定如何以最佳方式向用户展示它。

* [永远不要直接调用组件函数](/reference/rules/react-calls-components-and-hooks#never-call-component-functions-directly) – 组件只能在 JSX 中使用。不要把它们当作普通函数调用。
* [永远不要把 hook 作为普通值传来传去](/reference/rules/react-calls-components-and-hooks#never-pass-around-hooks-as-regular-values) – Hook 只能在组件内部调用。不要把它当作普通值传递。

---

## Hooks 规则 {/*rules-of-hooks*/}

Hooks 使用 JavaScript 函数定义，但它们代表一种特殊的、可复用的 UI 逻辑类型，并且对其可被调用的位置有限制。使用它们时，你需要遵循 [Hooks 规则](/reference/rules/rules-of-hooks)。

* [只能在顶层调用 Hooks](/reference/rules/rules-of-hooks#only-call-hooks-at-the-top-level) – 不要在循环、条件语句或嵌套函数中调用 Hooks。相反，应始终在 React 函数的顶层使用 Hooks，并且要在任何提前返回之前。
* [只能从 React 函数中调用 Hooks](/reference/rules/rules-of-hooks#only-call-hooks-from-react-functions) – 不要从普通 JavaScript 函数中调用 Hooks。

