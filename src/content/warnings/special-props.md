---
title: 特殊属性警告
---

JSX 元素上的大多数 props 都会传递给组件，不过，有两个特殊的 props（`ref` 和 `key`）会被 React 使用，因此不会转发给组件。

例如，你不能在组件中读取 `props.key`。如果你需要在子组件中访问相同的值，你应该将其作为不同的 prop 传递（例如：`<ListItemWrapper key={result.id} id={result.id} />` 并读取 `props.id`）。虽然这看起来有些多余，但将应用逻辑与传递给 React 的提示分开是很重要的。
