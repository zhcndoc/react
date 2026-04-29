---
title: eslint-plugin-react-hooks
version: rc
---

<Intro>

`eslint-plugin-react-hooks` 提供 ESLint 规则来强制执行 [React 规则](/reference/rules)。

</Intro>

这个插件帮助你在构建时捕获对 React 规则的违反，确保你的组件和 hooks 遵循 React 的正确性和性能规则。这些 lint 检查涵盖了基础的 React 模式（exhaustive-deps 和 rules-of-hooks）以及由 React Compiler 标记的问题。React Compiler 的诊断会通过这个 ESLint 插件自动显示，即使你的应用尚未采用 compiler，也可以使用。

<Note>
当 compiler 报告一条诊断时，这意味着 compiler 能够静态检测到一种不受支持或违反 React 规则的模式。检测到这种情况时，它会**自动**跳过这些组件和 hooks，同时保持你应用的其余部分继续编译。这确保了可安全优化内容的最佳覆盖率，而不会破坏你的应用。

这对 lint 检查意味着，你不需要立即修复所有违反项。你可以按自己的节奏逐步处理它们，以渐进增加经过优化的组件数量。
</Note>

## 推荐规则 {/*recommended*/}

这些规则包含在 `eslint-plugin-react-hooks` 的 `recommended` 预设中：

* [`exhaustive-deps`](/reference/eslint-plugin-react-hooks/lints/exhaustive-deps) - 验证 React hooks 的依赖数组是否包含所有必要的依赖项
* [`rules-of-hooks`](/reference/eslint-plugin-react-hooks/lints/rules-of-hooks) - 验证组件和 hooks 是否遵循 Hooks 规则
* [`component-hook-factories`](/reference/eslint-plugin-react-hooks/lints/component-hook-factories) - 验证定义嵌套组件或 hooks 的高阶函数
* [`config`](/reference/eslint-plugin-react-hooks/lints/config) - 验证 compiler 配置选项
* [`error-boundaries`](/reference/eslint-plugin-react-hooks/lints/error-boundaries) - 验证在子错误处理中使用 Error Boundaries，而不是 try/catch
* [`gating`](/reference/eslint-plugin-react-hooks/lints/gating) - 验证 gating 模式的配置
* [`globals`](/reference/eslint-plugin-react-hooks/lints/globals) - 验证在渲染期间对全局变量的赋值/修改
* [`immutability`](/reference/eslint-plugin-react-hooks/lints/immutability) - 验证不要修改 props、state 和其他不可变值
* [`incompatible-library`](/reference/eslint-plugin-react-hooks/lints/incompatible-library) - 验证不要使用与 memoization 不兼容的库
* [`preserve-manual-memoization`](/reference/eslint-plugin-react-hooks/lints/preserve-manual-memoization) - 验证 compiler 保留现有的手动 memoization
* [`purity`](/reference/eslint-plugin-react-hooks/lints/purity) - 通过检查已知的不纯函数来验证组件/hooks 是否是纯的
* [`refs`](/reference/eslint-plugin-react-hooks/lints/refs) - 验证 refs 的正确使用，不要在渲染期间读取/写入
* [`set-state-in-effect`](/reference/eslint-plugin-react-hooks/lints/set-state-in-effect) - 验证不要在 effect 中同步调用 setState
* [`set-state-in-render`](/reference/eslint-plugin-react-hooks/lints/set-state-in-render) - 验证不要在渲染期间设置 state
* [`static-components`](/reference/eslint-plugin-react-hooks/lints/static-components) - 验证组件是静态的，不会在每次渲染时重新创建
* [`unsupported-syntax`](/reference/eslint-plugin-react-hooks/lints/unsupported-syntax) - 验证不要使用 React Compiler 不支持的语法
* [`use-memo`](/reference/eslint-plugin-react-hooks/lints/use-memo) - 验证 `useMemo` hook 的使用是否没有返回值