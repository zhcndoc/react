---
title: 调试与排查问题
---

<Intro>
本指南帮助你在使用 React 编译器时识别并修复问题。了解如何调试编译问题并解决常见问题。
</Intro>

<YouWillLearn>

* 编译器错误与运行时问题之间的区别
* 会破坏编译的常见模式
* 分步骤的调试工作流程

</YouWillLearn>

## 理解编译器行为 {/*understanding-compiler-behavior*/}

React 编译器旨在处理符合 [React 规则](/reference/rules) 的代码。当它遇到可能违反这些规则的代码时，它会安全地跳过优化，而不是冒着改变应用行为的风险。

### 编译器错误 vs 运行时问题 {/*compiler-errors-vs-runtime-issues*/}

**编译器错误** 发生在构建时，会阻止代码编译。这类错误很少见，因为编译器的设计是跳过有问题的代码，而不是直接失败。

**运行时问题** 发生在编译后的代码表现与预期不同的时候。大多数情况下，如果你遇到 React 编译器相关的问题，那通常是运行时问题。这通常发生在你的代码以编译器无法检测到的微妙方式违反了 React 规则，而编译器错误地编译了一个本应跳过的组件。

在调试运行时问题时，请重点查找受影响组件中未被 ESLint 规则检测到的 React 规则违反情况。编译器依赖你的代码遵循这些规则；当这些规则以它无法检测的方式被破坏时，就会出现运行时问题。


## 常见的破坏模式 {/*common-breaking-patterns*/}

React 编译器可能破坏你的应用的一个主要方式，是你的代码是以依赖记忆化来保证正确性的方式编写的。这意味着你的应用依赖某些特定值被记忆化后才能正常工作。由于编译器的记忆化方式可能与你手动的做法不同，这可能导致意外行为，例如 effect 过度触发、无限循环或更新缺失。

会发生这种情况的常见场景包括：

- **依赖引用相等性的 effect** - 当 effect 依赖对象或数组在多次渲染之间保持相同引用时
- **需要稳定引用的依赖数组** - 当不稳定的依赖导致 effect 触发过于频繁或产生无限循环时
- **基于引用检查的条件逻辑** - 当代码使用引用相等性检查进行缓存或优化时

## 调试工作流程 {/*debugging-workflow*/}

遇到问题时，请按以下步骤操作：

### 编译器构建错误 {/*compiler-build-errors*/}

如果你遇到一个意外破坏构建的编译器错误，这很可能是编译器中的 bug。请将它报告到 [facebook/react](https://github.com/facebook/react/issues) 仓库，并附上：
- 错误消息
- 导致错误的代码
- 你的 React 和编译器版本

### 运行时问题 {/*runtime-issues*/}

对于运行时行为问题：

### 1. 临时禁用编译 {/*temporarily-disable-compilation*/}

使用 `"use no memo"` 来隔离问题是否与编译器有关：

```js
function ProblematicComponent() {
  "use no memo"; // 跳过此组件的编译
  // ... 组件其余部分
}
```

如果问题消失了，那很可能与 React 规则违反有关。

你也可以尝试从有问题的组件中移除手动记忆化（useMemo、useCallback、memo），以验证在没有任何记忆化的情况下你的应用是否能正确工作。如果在移除所有记忆化后问题仍然存在，那么你就有一个需要修复的 React 规则违反问题。

### 2. 逐步修复问题 {/*fix-issues-step-by-step*/}

1. 找出根本原因（通常是为了正确性的记忆化）
2. 每修复一步后进行测试
3. 修复完成后移除 `"use no memo"`
4. 验证该组件在 React DevTools 中显示 ✨ 徽标

## 报告编译器 bug {/*reporting-compiler-bugs*/}

如果你认为自己发现了编译器 bug：

1. **确认这不是 React 规则的违反** - 使用 ESLint 检查
2. **创建最小可复现示例** - 在一个小例子中隔离问题
3. **在没有编译器的情况下测试** - 确认问题只在启用编译时发生
4. **提交一个 [issue](https://github.com/facebook/react/issues/new?template=compiler_bug_report.yml)**：
   - React 和编译器版本
   - 最小可复现代码
   - 预期行为与实际行为
   - 任何错误消息

## 后续步骤 {/*next-steps*/}

- 查看 [React 规则](/reference/rules) 以防止问题
- 查看 [渐进式采用指南](/learn/react-compiler/incremental-adoption) 以了解逐步推广策略