---
title: 配置
---

<Intro>

本页列出了 React Compiler 中所有可用的配置选项。

</Intro>

<Note>

对于大多数应用，默认选项开箱即用。如果你有特殊需求，可以使用这些高级选项。

</Note>

```js
// babel.config.js
module.exports = {
  plugins: [
    [
      'babel-plugin-react-compiler', {
        // 编译器选项
      }
    ]
  ]
};
```

---

## 编译控制 {/*compilation-control*/}

这些选项控制编译器优化的*内容*以及它如何选择要编译的组件和 Hook。

* [`compilationMode`](/reference/react-compiler/compilationMode) 控制选择要编译的函数的策略（例如，全部函数、仅带注释的函数，或智能检测）。

```js
{
  compilationMode: 'annotation' // 仅编译 "use memo" 函数
}
```

---

## 版本兼容性 {/*version-compatibility*/}

React 版本配置可确保编译器生成与你的 React 版本兼容的代码。

[`target`](/reference/react-compiler/target) 指定你正在使用的 React 版本（17、18 或 19）。

```js
// 针对 React 18 项目
{
  target: '18' // 同时还需要 react-compiler-runtime 包
}
```

---

## 错误处理 {/*error-handling*/}

这些选项控制编译器如何响应不遵循 [React 规则](/reference/rules) 的代码。

[`panicThreshold`](/reference/react-compiler/panicThreshold) 决定是让构建失败还是跳过有问题的组件。

```js
// 生产环境推荐
{
  panicThreshold: 'none' // 出现错误时跳过组件，而不是让构建失败
}
```

---

## 调试 {/*debugging*/}

日志记录和分析选项可帮助你了解编译器在做什么。

[`logger`](/reference/react-compiler/logger) 提供编译事件的自定义日志记录。

```js
{
  logger: {
    logEvent(filename, event) {
      if (event.kind === 'CompileSuccess') {
        console.log('已编译：', filename);
      }
    }
  }
}
```

---

## 功能标志 {/*feature-flags*/}

条件编译可让你控制何时使用优化后的代码。

[`gating`](/reference/react-compiler/gating) 启用运行时功能标志，用于 A/B 测试或逐步发布。

```js
{
  gating: {
    source: 'my-feature-flags',
    importSpecifierName: 'isCompilerEnabled'
  }
}
```

---

## 常见配置模式 {/*common-patterns*/}

### 默认配置 {/*default-configuration*/}

对于大多数 React 19 应用，编译器无需配置即可工作：

```js
// babel.config.js
module.exports = {
  plugins: [
    'babel-plugin-react-compiler'
  ]
};
```

### React 17/18 项目 {/*react-17-18*/}

较旧的 React 版本需要 runtime 包和 target 配置：

```bash
npm install react-compiler-runtime@latest
```

```js
{
  target: '18' // 或 '17'
}
```

### 渐进式采用 {/*incremental-adoption*/}

从特定目录开始，逐步扩展：

```js
{
  compilationMode: 'annotation' // 仅编译 "use memo" 函数
}
```

