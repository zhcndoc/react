---
title: "旧版 React API"
---

<Intro>

这些 API 从 `react` 包中导出，但不建议在新编写的代码中使用。请查看链接到的各个 API 页面以了解建议的替代方案。

</Intro>

---

## 旧版 API {/*legacy-apis*/}

* [`Children`](/reference/react/Children) 允许你操作和转换作为 `children` 属性接收的 JSX。[查看替代方案。](/reference/react/Children#alternatives)
* [`cloneElement`](/reference/react/cloneElement) 允许你以另一个元素为起点创建 React 元素。[查看替代方案。](/reference/react/cloneElement#alternatives)
* [`Component`](/reference/react/Component) 允许你将 React 组件定义为 JavaScript 类。[查看替代方案。](/reference/react/Component#alternatives)
* [`createElement`](/reference/react/createElement) 允许你创建 React 元素。通常，你会改用 JSX。
* [`createRef`](/reference/react/createRef) 创建一个 ref 对象，该对象可以包含任意值。[查看替代方案。](/reference/react/createRef#alternatives)
* [`forwardRef`](/reference/react/forwardRef) 允许你的组件通过 [ref.](/learn/manipulating-the-dom-with-refs) 将 DOM 节点暴露给父组件
* [`isValidElement`](/reference/react/isValidElement) 检查某个值是否为 React 元素。通常与 [`cloneElement`.](/reference/react/cloneElement) 一起使用
* [`PureComponent`](/reference/react/PureComponent) 与 [`Component`,](/reference/react/Component) 类似，但它会跳过相同 props 的重新渲染。[查看替代方案。](/reference/react/PureComponent#alternatives)

---

## 已移除的 API {/*removed-apis*/}

这些 API 已在 React 19 中移除：

* [`createFactory`](https://18.react.dev/reference/react/createFactory)：改用 JSX。
* 类组件：[`static contextTypes`](https://18.react.dev//reference/react/Component#static-contexttypes)：改用 [`static contextType`](#static-contexttype)。
* 类组件：[`static childContextTypes`](https://18.react.dev//reference/react/Component#static-childcontexttypes)：改用 [`static contextType`](#static-contexttype)。
* 类组件：[`static getChildContext`](https://18.react.dev//reference/react/Component#getchildcontext)：改用 [`Context`](/reference/react/createContext#provider)。
* 类组件：[`static propTypes`](https://18.react.dev//reference/react/Component#static-proptypes)：改用如 [TypeScript](https://www.typescriptlang.org/) 之类的类型系统。
* 类组件：[`this.refs`](https://18.react.dev//reference/react/Component#refs)：改用 [`createRef`](/reference/react/createRef)。
