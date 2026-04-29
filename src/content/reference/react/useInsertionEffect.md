---
title: useInsertionEffect
---

<Pitfall>

`useInsertionEffect` 适用于 CSS-in-JS 库作者。除非你正在开发一个 CSS-in-JS 库并且需要一个插入样式的位置，否则你大概应该改用 [`useEffect`](/reference/react/useEffect) 或 [`useLayoutEffect`](/reference/react/useLayoutEffect)。

</Pitfall>

<Intro>

`useInsertionEffect` 允许在任何布局 Effects 触发之前将元素插入到 DOM 中。

```js
useInsertionEffect(setup, dependencies?)
```

</Intro>

<InlineToc />

---

## Reference {/*reference*/}

### `useInsertionEffect(setup, dependencies?)` {/*useinsertioneffect*/}

调用 `useInsertionEffect`，以便在任何可能需要读取布局的 Effects 触发之前插入样式：

```js
import { useInsertionEffect } from 'react';

// 在你的 CSS-in-JS 库内部
function useCSS(rule) {
  useInsertionEffect(() => {
    // ... 在这里注入 <style> 标签 ...
  });
  return rule;
}
```

[查看更多示例。](#usage)

#### Parameters {/*parameters*/}

* `setup`：包含你的 Effect 逻辑的函数。你的 setup 函数也可以选择返回一个 *cleanup* 函数。当你的组件被添加到 DOM 中时，但在任何布局 Effects 触发之前，React 会运行你的 setup 函数。在每次依赖项变化后重新渲染时，React 会先使用旧值运行 cleanup 函数（如果你提供了它），然后使用新值运行 setup 函数。当你的组件从 DOM 中移除时，React 会运行你的 cleanup 函数。

* **optional** `dependencies`：`setup` 代码中引用的所有响应式值列表。响应式值包括 props、state，以及在组件主体中直接声明的所有变量和函数。如果你的 linter 已为 [React 配置](/learn/editor-setup#linting)，它会验证每个响应式值都被正确指定为依赖项。依赖项列表必须具有固定数量的项，并且要以内联方式编写，例如 `[dep1, dep2, dep3]`。React 会使用 [`Object.is`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is) 比较算法将每个依赖项与其之前的值进行比较。如果你完全不指定依赖项，你的 Effect 将在组件每次重新渲染后重新运行。

#### Returns {/*returns*/}

`useInsertionEffect` 返回 `undefined`。

#### Caveats {/*caveats*/}

* Effects 只在客户端运行。它们不会在服务器渲染期间运行。
* 你不能在 `useInsertionEffect` 内部更新 state。
* 到 `useInsertionEffect` 运行时，refs 还没有附加。
* `useInsertionEffect` 可能会在 DOM 更新之前或之后运行。你不应该依赖 DOM 在某个特定时间点已经更新。
* 不同于其他类型的 Effects，它们会先为每个 Effect 触发 cleanup，再为每个 Effect 触发 setup，`useInsertionEffect` 会一次处理一个组件，先触发 cleanup 再触发 setup。这会导致 cleanup 和 setup 函数以“交错”的方式执行。
---

## Usage {/*usage*/}

### 从 CSS-in-JS 库中注入动态样式 {/*injecting-dynamic-styles-from-css-in-js-libraries*/}

传统上，你会使用普通 CSS 来为 React 组件设置样式。

```js
// 在你的 JS 文件中：
<button className="success" />

// 在你的 CSS 文件中：
.success { color: green; }
```

有些团队更喜欢直接在 JavaScript 代码中编写样式，而不是编写 CSS 文件。这通常需要使用 CSS-in-JS 库或工具。CSS-in-JS 常见有三种方式：

1. 使用编译器静态提取到 CSS 文件
2. 内联样式，例如 `<div style={{ opacity: 1 }}>`
3. 在运行时注入 `<style>` 标签

如果你使用 CSS-in-JS，我们建议结合前两种方式（静态样式使用 CSS 文件，动态样式使用内联样式）。**我们不推荐运行时注入 `<style>` 标签，原因有两个：**

1. 运行时注入会迫使浏览器更频繁地重新计算样式。
2. 如果发生在 React 生命周期中的错误时机，运行时注入可能会非常慢。

第一个问题无法解决，但 `useInsertionEffect` 可以帮助你解决第二个问题。

调用 `useInsertionEffect`，以便在任何布局 Effects 触发之前插入样式：

```js {4-11}
// 在你的 CSS-in-JS 库内部
let isInserted = new Set();
function useCSS(rule) {
  useInsertionEffect(() => {
    // 如前所述，我们不推荐运行时注入 <style> 标签。
    // 但如果你必须这样做，那么在 useInsertionEffect 中执行它很重要。
    if (!isInserted.has(rule)) {
      isInserted.add(rule);
      document.head.appendChild(getStyleForRule(rule));
    }
  });
  return rule;
}

function Button() {
  const className = useCSS('...');
  return <div className={className} />;
}
```

与 `useEffect` 类似，`useInsertionEffect` 不会在服务器上运行。如果你需要在服务器上收集已使用了哪些 CSS 规则，可以在渲染期间进行：

```js {1,4-6}
let collectedRulesSet = new Set();

function useCSS(rule) {
  if (typeof window === 'undefined') {
    collectedRulesSet.add(rule);
  }
  useInsertionEffect(() => {
    // ...
  });
  return rule;
}
```

[了解更多关于使用 `useInsertionEffect` 升级带有运行时注入的 CSS-in-JS 库的信息。](https://github.com/reactwg/react-18/discussions/110)

<DeepDive>

#### 这为什么比在渲染期间或 useLayoutEffect 中注入样式更好？ {/*how-is-this-better-than-injecting-styles-during-rendering-or-uselayouteffect*/}

如果你在渲染期间插入样式，并且 React 正在处理一个[非阻塞更新](/reference/react/useTransition#perform-non-blocking-updates-with-actions)，浏览器会在渲染组件树的每一帧都重新计算样式，这可能会**极其缓慢。**

`useInsertionEffect` 比在 [`useLayoutEffect`](/reference/react/useLayoutEffect) 或 [`useEffect`](/reference/react/useEffect) 中插入样式更好，因为它能确保当其他 Effects 在你的组件中运行时，`<style>` 标签已经被插入。否则，常规 Effects 中的布局计算会因为过时的样式而出错。

</DeepDive>
