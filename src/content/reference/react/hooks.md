---
title: "内置 React Hooks"
---

<Intro>

*Hooks* 让你能够在组件中使用不同的 React 功能。你既可以使用内置 Hooks，也可以将它们组合起来构建自己的 Hooks。本页列出了 React 中所有内置 Hooks。

</Intro>

---

## 状态 Hooks {/*state-hooks*/}

*状态* 让组件能够[“记住”诸如用户输入之类的信息。](/learn/state-a-components-memory) 例如，表单组件可以使用状态来存储输入值，而图片画廊组件可以使用状态来存储所选图片的索引。

要向组件添加状态，请使用以下 Hooks 之一：

* [`useState`](/reference/react/useState) 声明一个你可以直接更新的状态变量。
* [`useReducer`](/reference/react/useReducer) 声明一个在 [reducer 函数](/learn/extracting-state-logic-into-a-reducer) 中包含更新逻辑的状态变量。

```js
function ImageGallery() {
  const [index, setIndex] = useState(0);
  // ...
```

---

## Context Hooks {/*context-hooks*/}

*Context* 让组件能够[从远处的父组件接收信息，而无需通过 props 逐层传递。](/learn/passing-props-to-a-component) 例如，你的应用顶层组件可以将当前 UI 主题传递给其下方的所有组件，无论它们有多深。

* [`useContext`](/reference/react/useContext) 读取并订阅一个 context。

```js
function Button() {
  const theme = useContext(ThemeContext);
  // ...
```

---

## Ref Hooks {/*ref-hooks*/}

*Refs* 让组件能够[保存一些不用于渲染的信息，](/learn/referencing-values-with-refs)比如一个 DOM 节点或一个超时 ID。与状态不同，更新 ref 不会重新渲染组件。Refs 是 React 范式中的一个“逃生口”。当你需要与非 React 系统交互时，它们很有用，例如内置的浏览器 API。

* [`useRef`](/reference/react/useRef) 声明一个 ref。你可以在其中保存任何值，但最常见的是保存一个 DOM 节点。
* [`useImperativeHandle`](/reference/react/useImperativeHandle) 让你自定义组件暴露的 ref。这很少使用。

```js
function Form() {
  const inputRef = useRef(null);
  // ...
```

---

## Effect Hooks {/*effect-hooks*/}

*Effects* 让组件能够[连接并同步外部系统。](/learn/synchronizing-with-effects) 这包括处理网络、浏览器 DOM、动画、使用其他 UI 库编写的小部件，以及其他非 React 代码。

* [`useEffect`](/reference/react/useEffect) 将组件连接到外部系统。

```js
function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(roomId);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]);
  // ...
```

Effects 是 React 范式中的一个“逃生口”。不要使用 Effects 来编排应用的数据流。如果你没有与外部系统交互， [你可能不需要 Effect。](/learn/you-might-not-need-an-effect)

`useEffect` 有两个很少使用的变体，它们在时机上有所不同：

* [`useLayoutEffect`](/reference/react/useLayoutEffect) 在浏览器重绘屏幕之前触发。你可以在这里测量布局。
* [`useInsertionEffect`](/reference/react/useInsertionEffect) 在 React 对 DOM 进行更改之前触发。库可以在这里插入动态 CSS。

你也可以将事件与 Effects 分离：

- [`useEffectEvent`](/reference/react/useEffectEvent) 创建一个非响应式事件，可从任何 Effect hook 中触发。
---

## 性能 Hooks {/*performance-hooks*/}

一种常见的优化重新渲染性能的方法是跳过不必要的工作。例如，你可以告诉 React 重用缓存的计算结果，或者在数据自上次渲染以来没有变化时跳过重新渲染。

要跳过计算和不必要的重新渲染，请使用以下 Hooks 之一：

- [`useMemo`](/reference/react/useMemo) 让你缓存昂贵计算的结果。
- [`useCallback`](/reference/react/useCallback) 让你在将函数定义传递给经过优化的组件之前对其进行缓存。

```js
function TodoList({ todos, tab, theme }) {
  const visibleTodos = useMemo(() => filterTodos(todos, tab), [todos, tab]);
  // ...
}
```

有时，你无法跳过重新渲染，因为屏幕实际上需要更新。在这种情况下，你可以通过将必须同步的阻塞更新（如在输入框中输入）与不需要阻塞用户界面的非阻塞更新（如更新图表）分离来提升性能。

要优先处理渲染，请使用以下 Hooks 之一：

- [`useTransition`](/reference/react/useTransition) 让你将状态转换标记为非阻塞，并允许其他更新中断它。
- [`useDeferredValue`](/reference/react/useDeferredValue) 让你延迟更新 UI 中非关键部分，并让其他部分先更新。

---

## 其他 Hooks {/*other-hooks*/}

这些 Hooks 主要对库作者有用，在应用代码中并不常用。

- [`useDebugValue`](/reference/react/useDebugValue) 让你自定义 React DevTools 为你的自定义 Hook 显示的标签。
- [`useId`](/reference/react/useId) 让组件为自身关联一个唯一 ID。通常与可访问性 API 一起使用。
- [`useSyncExternalStore`](/reference/react/useSyncExternalStore) 让组件订阅外部 store。
* [`useActionState`](/reference/react/useActionState) 允许你管理 action 的状态。

---

## 你自己的 Hooks {/*your-own-hooks*/}

你也可以将 [你自己的自定义 Hooks](/learn/reusing-logic-with-custom-hooks#extracting-your-own-custom-hook-from-a-component) 定义为 JavaScript 函数。
