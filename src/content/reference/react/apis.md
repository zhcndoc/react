---
title: "内置 React API"
---

<Intro>

除了 [Hooks](/reference/react/hooks) 和 [组件](/reference/react/components) 之外，`react` 包还导出了一些对定义组件有用的其他 API。本页列出了其余所有现代 React API。

</Intro>

---

* [`createContext`](/reference/react/createContext) 让你定义并向子组件提供上下文。与 [`useContext`.](/reference/react/useContext) 一起使用
* [`lazy`](/reference/react/lazy) 让你将组件代码的加载推迟到它首次渲染时。
* [`memo`](/reference/react/memo) 让你的组件在相同 props 下跳过重新渲染。与 [`useMemo`](/reference/react/useMemo) 和 [`useCallback`.](/reference/react/useCallback) 一起使用
* [`startTransition`](/reference/react/startTransition) 让你将状态更新标记为非紧急更新。类似于 [`useTransition`.](/reference/react/useTransition)
* [`addTransitionType`](/reference/react/addTransitionType) 让你指定 Transition 的原因。与 [`startTransition`](/reference/react/startTransition) 和 [`<ViewTransition>`.](/reference/react/ViewTransition) 一起使用
* [`act`](/reference/react/act) 让你在测试中将渲染和交互包裹起来，以确保在进行断言之前更新已经处理完成。
* [`cache`](/reference/react/cache) 让你缓存数据请求或计算的结果。
* [`cacheSignal`](/reference/react/cacheSignal) 让你知道 `cache()` 的生命周期何时结束。
* [`captureOwnerStack`](/reference/react/captureOwnerStack) 在开发环境中读取当前的 Owner Stack，并在可用时以字符串形式返回。

---

## 资源 API {/*resource-apis*/}

*资源* 可以被组件访问，而不必将其作为状态的一部分。例如，组件可以从 Promise 中读取消息，或者从上下文中读取样式信息。

你可以将以下类型的资源传递给 [`use`](/reference/react/use)：

* [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)，以读取它解析后的值。
* [上下文](/learn/passing-data-deeply-with-context)，以读取它的值。
* [`browser`](/reference/react-dom/browser) 返回的值，用于在服务端渲染期间将组件标记为仅在浏览器中渲染。

```js
function MessageComponent({ messagePromise }) {
  const message = use(messagePromise);
  const theme = use(ThemeContext);
  use(browser());
  // ...
}
```
