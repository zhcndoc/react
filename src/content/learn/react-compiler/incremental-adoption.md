---
title: 渐进式采用
---

<Intro>
React Compiler 可以渐进式采用，允许你先在代码库的特定部分尝试它。本指南将展示如何在现有项目中逐步推广该编译器。
</Intro>

<YouWillLearn>

* 为什么推荐渐进式采用
* 如何使用 Babel overrides 基于目录进行采用
* 如何使用 "use memo" 指令进行选择性编译
* 如何使用 "use no memo" 指令排除组件
* 带门控的运行时功能标志
* 如何监控采用进度

</YouWillLearn>

## 为什么要渐进式采用？ {/*why-incremental-adoption*/}

React Compiler 旨在自动优化你的整个代码库，但你不必一次性全部采用。渐进式采用让你可以掌控推广过程，在将其扩展到应用其余部分之前，先在较小的部分上测试编译器。

从小范围开始有助于你建立对编译器优化的信心。你可以验证应用在编译后代码下是否正常运行，衡量性能提升，并识别任何与你的代码库相关的特殊边缘情况。这种方法对稳定性至关重要的生产应用尤其有价值。

渐进式采用还使得处理编译器可能发现的任何 React 规则违规更容易。你不必一次性修复整个代码库中的违规，而是可以在扩展编译器覆盖范围时系统性地处理它们。这样可以让迁移更易管理，并降低引入 bug 的风险。

通过控制代码中哪些部分会被编译，你还可以运行 A/B 测试来衡量编译器优化在真实环境中的影响。这些数据有助于你对全面采用做出明智决策，并向团队展示其价值。

## 渐进式采用的方法 {/*approaches-to-incremental-adoption*/}

有三种主要方式可以逐步采用 React Compiler：

1. **Babel overrides** - 将编译器应用到特定目录
2. **通过 "use memo" 进行选择加入** - 只编译明确选择加入的组件
3. **运行时门控** - 使用功能标志控制编译

所有方法都允许你在全面推广之前，先在应用的特定部分测试编译器。

## 使用 Babel Overrides 的基于目录的采用 {/*directory-based-adoption*/}

Babel 的 `overrides` 选项允许你对代码库的不同部分应用不同的插件。这非常适合按目录逐步采用 React Compiler。

### 基本配置 {/*basic-configuration*/}

首先将编译器应用到一个特定目录：

```js
// babel.config.js
module.exports = {
  plugins: [
    // 适用于所有文件的全局插件
  ],
  overrides: [
    {
      test: './src/modern/**/*.{js,jsx,ts,tsx}',
      plugins: [
        'babel-plugin-react-compiler'
      ]
    }
  ]
};
```

### 扩大覆盖范围 {/*expanding-coverage*/}

随着你越来越有信心，可以添加更多目录：

```js
// babel.config.js
module.exports = {
  plugins: [
    // 全局插件
  ],
  overrides: [
    {
      test: ['./src/modern/**/*.{js,jsx,ts,tsx}', './src/features/**/*.{js,jsx,ts,tsx}'],
      plugins: [
        'babel-plugin-react-compiler'
      ]
    },
    {
      test: './src/legacy/**/*.{js,jsx,ts,tsx}',
      plugins: [
        // 适用于旧代码的不同插件
      ]
    }
  ]
};
```

### 使用编译器选项 {/*with-compiler-options*/}

你也可以针对每个 override 配置编译器选项：

```js
// babel.config.js
module.exports = {
  plugins: [],
  overrides: [
    {
      test: './src/experimental/**/*.{js,jsx,ts,tsx}',
      plugins: [
        ['babel-plugin-react-compiler', {
          // 选项 ...
        }]
      ]
    },
    {
      test: './src/production/**/*.{js,jsx,ts,tsx}',
      plugins: [
        ['babel-plugin-react-compiler', {
          // 选项 ...
        }]
      ]
    }
  ]
};
```


## 使用 "use memo" 的选择加入模式 {/*opt-in-mode-with-use-memo*/}

为了获得最大的控制权，你可以使用 `compilationMode: 'annotation'`，仅编译那些明确通过 `"use memo"` 指令选择加入的组件和 Hook。

<Note>
这种方法让你可以对单个组件和 Hook 进行细粒度控制。当你想在不影响整个目录的情况下测试编译器时，这很有用。
</Note>

### 注解模式配置 {/*annotation-mode-configuration*/}

```js
// babel.config.js
module.exports = {
  plugins: [
    ['babel-plugin-react-compiler', {
      compilationMode: 'annotation',
    }],
  ],
};
```

### 使用该指令 {/*using-the-directive*/}

在你想要编译的函数开头添加 `"use memo"`：

```js
function TodoList({ todos }) {
  "use memo"; // 将此组件选择加入编译

  const sortedTodos = todos.slice().sort();

  return (
    <ul>
      {sortedTodos.map(todo => (
        <TodoItem key={todo.id} todo={todo} />
      ))}
    </ul>
  );
}

function useSortedData(data) {
  "use memo"; // 将此 Hook 选择加入编译

  return data.slice().sort();
}
```

使用 `compilationMode: 'annotation'` 时，你必须：
- 为每个想要优化的组件添加 `"use memo"`
- 为每个自定义 Hook 添加 `"use memo"`
- 记得在新组件中也添加它

这使你能够在评估编译器影响时，精确控制哪些组件会被编译。

## 带门控的运行时功能标志 {/*runtime-feature-flags-with-gating*/}

`gating` 选项使你能够在运行时使用功能标志来控制编译。这对于运行 A/B 测试，或基于用户群体逐步推广编译器非常有用。

### 门控如何工作 {/*how-gating-works*/}

编译器会将优化后的代码包装在一个运行时检查中。如果 gate 返回 `true`，则运行优化版本。否则，运行原始代码。

### 门控配置 {/*gating-configuration*/}

```js
// babel.config.js
module.exports = {
  plugins: [
    ['babel-plugin-react-compiler', {
      gating: {
        source: 'ReactCompilerFeatureFlags',
        importSpecifierName: 'isCompilerEnabled',
      },
    }],
  ],
};
```

### 实现功能标志 {/*implementing-the-feature-flag*/}

创建一个导出门控函数的模块：

```js
// ReactCompilerFeatureFlags.js
export function isCompilerEnabled() {
  // 使用你的功能标志系统
  return getFeatureFlag('react-compiler-enabled');
}
```

## 采用过程中的故障排查 {/*troubleshooting-adoption*/}

如果你在采用过程中遇到问题：

1. 使用 `"use no memo"` 临时排除有问题的组件
2. 查看 [调试指南](/learn/react-compiler/debugging) 了解常见问题
3. 修复 ESLint 插件识别出的 React 规则违规
4. 考虑使用 `compilationMode: 'annotation'` 以实现更渐进的采用

## 下一步 {/*next-steps*/}

- 阅读 [配置指南](/reference/react-compiler/configuration) 以了解更多选项
- 学习 [调试技巧](/learn/react-compiler/debugging)
- 查看 [API 参考](/reference/react-compiler/configuration) 以获取所有编译器选项