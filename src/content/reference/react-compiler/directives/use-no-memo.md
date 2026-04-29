---
title: "use no memo"
titleForTitleTag: "'use no memo' 指令"
---

<Intro>

`"use no memo"` 可阻止 React Compiler 对函数进行优化。

</Intro>

<InlineToc />

---

## 参考 {/*reference*/}

### `"use no memo"` {/*use-no-memo*/}

在函数开头添加 `"use no memo"`，以阻止 React Compiler 优化。

```js {1}
function MyComponent() {
  "use no memo";
  // ...
}
```

当函数包含 `"use no memo"` 时，React Compiler 会在优化过程中完全跳过它。这在调试期间，或处理与编译器无法正确配合的代码时，作为临时退出机制非常有用。

#### 注意事项 {/*caveats*/}

* `"use no memo"` 必须位于函数体的最开头，在任何 import 或其他代码之前（注释可以）。
* 该指令必须使用双引号或单引号书写，不能使用反引号。
* 该指令必须与 `"use no memo"` 或其别名 `"use no forget"` 完全一致。
* 该指令优先于所有编译模式和其他指令。
* 它设计为临时调试工具，而不是永久解决方案。

### `"use no memo"` 如何退出优化 {/*how-use-no-memo-opts-out*/}

React Compiler 会在构建时分析你的代码以应用优化。`"use no memo"` 会创建一个明确的边界，告诉编译器完全跳过某个函数。

该指令优先于所有其他设置：
* 在 `all` 模式下：即使全局设置存在，函数也会被跳过
* 在 `infer` 模式下：即使启发式规则会优化它，函数也会被跳过

编译器会将这些函数视为 React Compiler 未启用，保持它们完全按照原样不变。

### 何时使用 `"use no memo"` {/*when-to-use*/}

`"use no memo"` 应谨慎且临时地使用。常见场景包括：

#### 调试编译器问题 {/*debugging-compiler*/}
当你怀疑编译器导致了问题时，临时禁用优化以隔离问题：

```js
function ProblematicComponent({ data }) {
  "use no memo"; // TODO: 修复 issue #123 后移除

  // 未被静态检测到的 React 规则违规
  // ...
}
```

#### 第三方库集成 {/*third-party*/}
当集成的库可能与编译器不兼容时：

```js
function ThirdPartyWrapper() {
  "use no memo";

  useThirdPartyHook(); // 存在编译器可能错误优化的副作用
  // ...
}
```

---

## 用法 {/*usage*/}

`"use no memo"` 指令放在函数体开头，用于阻止 React Compiler 优化该函数：

```js
function MyComponent() {
  "use no memo";
  // 函数体
}
```

该指令也可以放在文件顶部，以影响该模块中的所有函数：

```js
"use no memo";

// 此文件中的所有函数都将被编译器跳过
```

函数级别的 `"use no memo"` 会覆盖模块级别的指令。

---

## 故障排查 {/*troubleshooting*/}

### 指令未阻止编译 {/*not-preventing*/}

如果 `"use no memo"` 不起作用：

```js
// ❌ 错误 - 指令放在代码之后
function Component() {
  const data = getData();
  "use no memo"; // 太晚了！
}

// ✅ 正确 - 指令在最前面
function Component() {
  "use no memo";
  const data = getData();
}
```

另外请检查：
* 拼写 - 必须完全是 `"use no memo"`
* 引号 - 必须使用单引号或双引号，不能使用反引号

### 最佳实践 {/*best-practices*/}

**始终记录原因**，说明你为什么禁用优化：

```js
// ✅ 好 - 说明清晰且可追踪
function DataProcessor() {
  "use no memo"; // TODO: 修复 react 违规规则后移除
  // ...
}

// ❌ 差 - 没有说明
function Mystery() {
  "use no memo";
  // ...
}
```

### 另请参阅 {/*see-also*/}

* [`"use memo"`](/reference/react-compiler/directives/use-memo) - 启用编译
* [React Compiler](/learn/react-compiler) - 入门指南