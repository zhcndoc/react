---
title: compilationMode
---

<Intro>

`compilationMode` 选项控制 React Compiler 选择要编译哪些函数的方式。

</Intro>

```js
{
  compilationMode: 'infer' // 或 'annotation'、'syntax'、'all'
}
```

<InlineToc />

---

## 参考 {/*reference*/}

### `compilationMode` {/*compilationmode*/}

控制用于确定 React Compiler 将优化哪些函数的策略。

#### 类型 {/*type*/}

```
'infer' | 'syntax' | 'annotation' | 'all'
```

#### 默认值 {/*default-value*/}

`'infer'`

#### 选项 {/*options*/}

- **`'infer'`**（默认）：编译器使用智能启发式规则来识别 React 组件和 hooks：
  - 明确使用 `"use memo"` 指令标注的函数
  - 名称像组件（PascalCase）或 hook（`use` 前缀）的函数，并且创建 JSX 和/或调用其他 hooks

- **`'annotation'`**：仅编译显式标记了 `"use memo"` 指令的函数。非常适合渐进式采用。

- **`'syntax'`**：仅编译使用 Flow 的 [component](https://flow.org/en/docs/react/component-syntax/) 和 [hook](https://flow.org/en/docs/react/hook-syntax/) 语法的组件和 hooks。

- **`'all'`**：编译所有顶层函数。不推荐，因为它可能会编译非 React 函数。

#### 注意事项 {/*caveats*/}

- `'infer'` 模式要求函数遵循 React 命名约定才能被检测到
- 使用 `'all'` 模式可能会通过编译工具函数而对性能产生负面影响
- `'syntax'` 模式需要 Flow，并且不能与 TypeScript 一起使用
- 无论采用哪种模式，带有 `"use no memo"` 指令的函数都会始终被跳过

---

## 用法 {/*usage*/}

### 默认推断模式 {/*default-inference-mode*/}

默认的 `'infer'` 模式适用于大多数遵循 React 约定的代码库：

```js
{
  compilationMode: 'infer'
}
```

在此模式下，这些函数将被编译：

```js
// ✅ 已编译：名称像组件 + 返回 JSX
function Button(props) {
  return <button>{props.label}</button>;
}

// ✅ 已编译：名称像 hook + 调用 hooks
function useCounter() {
  const [count, setCount] = useState(0);
  return [count, setCount];
}

// ✅ 已编译：显式指令
function expensiveCalculation(data) {
  "use memo";
  return data.reduce(/* ... */);
}

// ❌ 未编译：不是组件/hook 模式
function calculateTotal(items) {
  return items.reduce((a, b) => a + b, 0);
}
```

### 使用 annotation 模式进行渐进式采用 {/*incremental-adoption*/}

对于渐进迁移，使用 `'annotation'` 模式仅编译已标记的函数：

```js
{
  compilationMode: 'annotation'
}
```

然后显式标记要编译的函数：

```js
// 只有这个函数会被编译
function ExpensiveList(props) {
  "use memo";
  return (
    <ul>
      {props.items.map(item => (
        <li key={item.id}>{item.name}</li>
      ))}
    </ul>
  );
}

// 如果没有该指令，这个不会被编译
function NormalComponent(props) {
  return <div>{props.content}</div>;
}
```

### 使用 Flow 语法模式 {/*flow-syntax-mode*/}

如果你的代码库使用 Flow 而不是 TypeScript：

```js
{
  compilationMode: 'syntax'
}
```

然后使用 Flow 的组件语法：

```js
// 已编译：Flow 组件语法
component Button(label: string) {
  return <button>{label}</button>;
}

// 已编译：Flow hook 语法
hook useCounter(initial: number) {
  const [count, setCount] = useState(initial);
  return [count, setCount];
}

// 未编译：普通函数语法
function helper(data) {
  return process(data);
}
```

### 对特定函数选择不编译 {/*opting-out*/}

无论编译模式如何，都可以使用 `"use no memo"` 跳过编译：

```js
function ComponentWithSideEffects() {
  "use no memo"; // 阻止编译

  // 这个组件有不应被 memoize 的副作用
  logToAnalytics('component_rendered');

  return <div>Content</div>;
}
```

---

## 故障排除 {/*troubleshooting*/}

### 在 infer 模式下组件未被编译 {/*component-not-compiled-infer*/}

在 `'infer'` 模式下，请确保你的组件遵循 React 约定：

```js
// ❌ 不会被编译：名称小写
function button(props) {
  return <button>{props.label}</button>;
}

// ✅ 会被编译：PascalCase 名称
function Button(props) {
  return <button>{props.label}</button>;
}

// ❌ 不会被编译：没有创建 JSX 或调用 hooks
function useData() {
  return window.localStorage.getItem('data');
}

// ✅ 会被编译：调用了 hook
function useData() {
  const [data] = useState(() => window.localStorage.getItem('data'));
  return data;
}
```
