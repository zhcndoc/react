---
title: "use memo"
titleForTitleTag: "'use memo' 指令"
---

<Intro>

`"use memo"` 会将一个函数标记为可由 React Compiler 优化。

</Intro>

<Note>

在大多数情况下，你不需要 `"use memo"`。它主要用于 `annotation` 模式，在该模式下你必须显式标记要优化的函数。在 `infer` 模式下，编译器会根据命名模式自动检测组件和 Hook（组件使用 PascalCase，Hook 使用 `use` 前缀）。如果某个组件或 Hook 在 `infer` 模式下没有被编译，应该修正它的命名约定，而不是通过 `"use memo"` 强行编译。

</Note>

<InlineToc />

---

## 参考 {/*reference*/}

### `"use memo"` {/*use-memo*/}

在函数开头添加 `"use memo"`，即可将其标记为可由 React Compiler 优化。

```js {1}
function MyComponent() {
  "use memo";
  // ...
}
```

当函数包含 `"use memo"` 时，React Compiler 会在构建时分析并优化它。编译器会自动对值和组件进行记忆化，以避免不必要的重新计算和重新渲染。

#### 注意事项 {/*caveats*/}

* `"use memo"` 必须位于函数体的最开头，在任何导入或其他代码之前（注释可以）。
* 该指令必须使用双引号或单引号书写，不能使用反引号。
* 该指令必须与 `"use memo"` 完全一致。
* 只有函数中的第一个指令会被处理；后续指令会被忽略。
* 该指令的效果取决于你的 [`compilationMode`](/reference/react-compiler/compilationMode) 设置。

### `"use memo"` 如何标记函数以供优化 {/*how-use-memo-marks*/}

在使用 React Compiler 的 React 应用中，函数会在构建时被分析，以确定是否可以优化。默认情况下，编译器会自动推断哪些组件需要记忆化，但这也可能取决于你设置的 [`compilationMode`](/reference/react-compiler/compilationMode)。

`"use memo"` 会显式将一个函数标记为可优化，覆盖默认行为：

* 在 `annotation` 模式下：只有带有 `"use memo"` 的函数才会被优化
* 在 `infer` 模式下：编译器会使用启发式规则，但 `"use memo"` 会强制优化
* 在 `all` 模式下：默认会优化所有内容，因此 `"use memo"` 是多余的

该指令在代码库中为已优化和未优化的代码建立了清晰的边界，让你能够对编译过程进行细粒度控制。

### 何时使用 `"use memo"` {/*when-to-use*/}

你应该在以下情况下考虑使用 `"use memo"`：

#### 你正在使用 annotation 模式 {/*annotation-mode-use*/}
在 `compilationMode: 'annotation'` 中，任何你希望优化的函数都必须使用该指令：

```js
// ✅ 这个组件会被优化
function OptimizedList() {
  "use memo";
  // ...
}

// ❌ 这个组件不会被优化
function SimpleWrapper() {
  // ...
}
```

#### 你正在逐步采用 React Compiler {/*gradual-adoption*/}
从 `annotation` 模式开始，有选择地优化稳定的组件：

```js
// 先从优化叶子组件开始
function Button({ onClick, children }) {
  "use memo";
  // ...
}

// 随着你验证行为，再逐步向上推进组件树
function ButtonGroup({ buttons }) {
  "use memo";
  // ...
}
```

---

## 用法 {/*usage*/}

### 结合不同的编译模式使用 {/*compilation-modes*/}

`"use memo"` 的行为会根据你的编译器配置而变化：

```js
// babel.config.js
module.exports = {
  plugins: [
    ['babel-plugin-react-compiler', {
      compilationMode: 'annotation' // 或 'infer' 或 'all'
    }]
  ]
};
```

#### Annotation 模式 {/*annotation-mode-example*/}
```js
// ✅ 使用 "use memo" 进行优化
function ProductCard({ product }) {
  "use memo";
  // ...
}

// ❌ 未优化（没有指令）
function ProductList({ products }) {
  // ...
}
```

#### Infer 模式（默认） {/*infer-mode-example*/}
```js
// 由于名称像一个组件，因此会自动记忆化
function ComplexDashboard({ data }) {
  // ...
}

// 跳过：名称不像一个组件
function simpleDisplay({ text }) {
  // ...
}
```

在 `infer` 模式下，编译器会根据命名模式自动检测组件和 Hook（组件使用 PascalCase，Hook 使用 `use` 前缀）。如果某个组件或 Hook 在 `infer` 模式下没有被编译，应该修正它的命名约定，而不是通过 `"use memo"` 强行编译。

---

## 故障排查 {/*troubleshooting*/}

### 验证优化 {/*verifying-optimization*/}

要确认你的组件已被优化：

1. 检查构建产物中的编译结果
2. 使用 React DevTools 查看是否有 Memo ✨ 徽标

### 另见 {/*see-also*/}

* [`"use no memo"`](/reference/react-compiler/directives/use-no-memo) - 取消编译
* [`compilationMode`](/reference/react-compiler/compilationMode) - 配置编译行为
* [React Compiler](/learn/react-compiler) - 入门指南