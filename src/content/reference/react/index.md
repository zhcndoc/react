---
title: React 参考概览
---

<Intro>

本节提供了有关使用 React 的详细参考文档。要了解 React 的介绍，请访问 [Learn](/learn) 部分。

</Intro>

React 参考文档被划分为以下功能子部分：

## React {/*react*/}

程序化的 React 特性：

* [Hooks](/reference/react/hooks) - 在你的组件中使用不同的 React 特性。
* [Components](/reference/react/components) - 可在你的 JSX 中使用的内置组件。
* [APIs](/reference/react/apis) - 用于定义组件的有用 API。
* [Directives](/reference/rsc/directives) - 向与 React Server Components 兼容的打包器提供指令。

## React DOM {/*react-dom*/}

React DOM 包含仅受 Web 应用支持的特性（这些应用运行在浏览器 DOM 环境中）。本节分为以下内容：

* [Hooks](/reference/react-dom/hooks) - 适用于运行在浏览器 DOM 环境中的 Web 应用的 Hooks。
* [Components](/reference/react-dom/components) - React 支持所有浏览器内置的 HTML 和 SVG 组件。
* [APIs](/reference/react-dom) - `react-dom` 包包含仅在 Web 应用中受支持的方法。
* [Client APIs](/reference/react-dom/client) - `react-dom/client` API 让你能够在客户端（浏览器中）渲染 React 组件。
* [Server APIs](/reference/react-dom/server) - `react-dom/server` API 让你能够在服务器上将 React 组件渲染为 HTML。
* [Static APIs](/reference/react-dom/static) - `react-dom/static` API 让你能够为 React 组件生成静态 HTML。

## React Compiler {/*react-compiler*/}

React Compiler 是一个构建时优化工具，它会自动对你的 React 组件和值进行记忆化：

* [Configuration](/reference/react-compiler/configuration) - React Compiler 的配置选项。
* [Directives](/reference/react-compiler/directives) - 用于控制编译的函数级指令。
* [Compiling Libraries](/reference/react-compiler/compiling-libraries) - 打包预编译库代码的指南。

## ESLint Plugin React Hooks {/*eslint-plugin-react-hooks*/}

[适用于 React Hooks 的 ESLint 插件](/reference/eslint-plugin-react-hooks) 有助于强制执行 React 规则：

* [Lints](/reference/eslint-plugin-react-hooks) - 每个 lint 的详细文档及示例。

## Rules of React {/*rules-of-react*/}

React 有一些习惯用法——或者说规则——来以易于理解的方式表达模式，并生成高质量的应用程序：

* [Components and Hooks must be pure](/reference/rules/components-and-hooks-must-be-pure) – 纯净性使你的代码更容易理解、调试，并允许 React 正确地自动优化你的组件和 hooks。
* [React calls Components and Hooks](/reference/rules/react-calls-components-and-hooks) – React 负责在必要时渲染组件和 hooks，以优化用户体验。
* [Rules of Hooks](/reference/rules/rules-of-hooks) – Hooks 使用 JavaScript 函数定义，但它们表示一种可复用的特殊 UI 逻辑，并对其调用位置有所限制。

## Legacy APIs {/*legacy-apis*/}

* [Legacy APIs](/reference/react/legacy) - 从 `react` 包导出，但不建议在新编写的代码中使用。
