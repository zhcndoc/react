---
title: 指令
---

<Intro>
React Compiler 指令是特殊的字符串字面量，用于控制特定函数是否被编译。
</Intro>

```js
function MyComponent() {
  "use memo"; // 将此组件纳入编译
  return <div>{/* ... */}</div>;
}
```

<InlineToc />

---

## 概览 {/*overview*/}

React Compiler 指令提供了对哪些函数会被编译器优化的细粒度控制。它们是放在函数体开头或模块顶部的字符串字面量。

### 可用指令 {/*available-directives*/}

* **[`"use memo"`](/reference/react-compiler/directives/use-memo)** - 将函数纳入编译
* **[`"use no memo"`](/reference/react-compiler/directives/use-no-memo)** - 将函数排除出编译

### 快速对比 {/*quick-comparison*/}

| 指令 | 目的 | 何时使用 |
|-----------|---------|-------------|
| [`"use memo"`](/reference/react-compiler/directives/use-memo) | 强制编译 | 在使用 `annotation` 模式时，或覆盖 `infer` 模式的启发式决策时 |
| [`"use no memo"`](/reference/react-compiler/directives/use-no-memo) | 阻止编译 | 调试问题或处理不兼容代码时 |

---

## 用法 {/*usage*/}

### 函数级指令 {/*function-level*/}

将指令放在函数开头以控制其编译：

```js
// 将其纳入编译
function OptimizedComponent() {
  "use memo";
  return <div>This will be optimized</div>;
}

// 将其排除出编译
function UnoptimizedComponent() {
  "use no memo";
  return <div>This won't be optimized</div>;
}
```

### 模块级指令 {/*module-level*/}

将指令放在文件顶部，以影响该模块中的所有函数：

```js
// 位于文件最顶部
"use memo";

// 此文件中的所有函数都会被编译
function Component1() {
  return <div>Compiled</div>;
}

function Component2() {
  return <div>Also compiled</div>;
}

// 可在函数级别覆盖
function Component3() {
  "use no memo"; // 这会覆盖模块指令
  return <div>Not compiled</div>;
}
```

### 与编译模式的交互 {/*compilation-modes*/}

指令的行为会因你的 [`compilationMode`](/reference/react-compiler/compilationMode) 而异：

* **`annotation` 模式**：只有带有 `"use memo"` 的函数会被编译
* **`infer` 模式**：编译器决定编译什么，指令会覆盖这些决定
* **`all` 模式**：所有内容都会被编译，`"use no memo"` 可以排除特定函数

---

## 最佳实践 {/*best-practices*/}

### 谨慎使用指令 {/*use-sparingly*/}

指令是逃生阀。更推荐在项目级别配置编译器：

```js
// ✅ 好 - 项目范围配置
{
  plugins: [
    ['babel-plugin-react-compiler', {
      compilationMode: 'infer'
    }]
  ]
}

// ⚠️ 仅在需要时使用指令
function SpecialCase() {
  "use no memo"; // 记录这样做的原因
  // ...
}
```

### 记录指令的使用原因 {/*document-usage*/}

始终说明使用指令的原因：

```js
// ✅ 好 - 说明清晰
function DataGrid() {
  "use no memo"; // TODO: 在修复动态行高问题后移除（JIRA-123）
  // 复杂的网格实现
}

// ❌ 坏 - 没有说明
function Mystery() {
  "use no memo";
  // ...
}
```

### 为移除做计划 {/*plan-removal*/}

退出编译的指令应当是临时性的：

1. 添加带有 TODO 注释的指令
2. 创建一个跟踪问题
3. 修复底层问题
4. 移除该指令

```js
function TemporaryWorkaround() {
  "use no memo"; // TODO: 在升级 ThirdPartyLib 到 v2.0 后移除
  return <ThirdPartyComponent />;
}
```

---

## 常见模式 {/*common-patterns*/}

### 渐进式采用 {/*gradual-adoption*/}

在大型代码库中采用 React Compiler 时：

```js
// 从 annotation 模式开始
{
  compilationMode: 'annotation'
}

// 为稳定组件纳入编译
function StableComponent() {
  "use memo";
  // 测试充分的组件
}

// 之后切换到 infer 模式，并让有问题的组件退出编译
function ProblematicComponent() {
  "use no memo"; // 在移除前先修复问题
  // ...
}
```


---

## 故障排查 {/*troubleshooting*/}

有关指令的具体问题，请参阅以下文档中的故障排查部分：

* [`"use memo"` troubleshooting](/reference/react-compiler/directives/use-memo#troubleshooting)
* [`"use no memo"` troubleshooting](/reference/react-compiler/directives/use-no-memo#troubleshooting)

### 常见问题 {/*common-issues*/}

1. **指令被忽略**：检查位置（必须是第一条）和拼写
2. **仍然发生编译**：检查 `ignoreUseNoForget` 设置
3. **模块指令不生效**：确保它位于所有 import 之前

---

## 另请参阅 {/*see-also*/}

* [`compilationMode`](/reference/react-compiler/compilationMode) - 配置编译器如何选择要优化的内容
* [`Configuration`](/reference/react-compiler/configuration) - 完整的编译器配置选项
* [React Compiler 文档](https://react.dev/learn/react-compiler) - 入门指南