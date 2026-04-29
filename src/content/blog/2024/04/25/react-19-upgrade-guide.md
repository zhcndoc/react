---
title: "React 19 升级指南"
author: Ricky Hanlon
date: 2024/04/25
description: React 19 新增的改进需要一些破坏性变更，但我们已经尽力让升级过程尽可能平滑，并且我们不认为这些变更会影响大多数应用。在这篇文章中，我们将指导你完成将应用和库升级到 React 19 的步骤。
---

2024 年 4 月 25 日，作者 [Ricky Hanlon](https://twitter.com/rickhanlonii)

---


<Intro>

React 19 新增的改进需要一些破坏性变更，但我们已经尽力让升级过程尽可能平滑，并且我们不认为这些变更会影响大多数应用。

</Intro>

<Note>

#### React 18.3 也已经发布 {/*react-18-3*/}

为了帮助更轻松地升级到 React 19，我们发布了 `react@18.3` 版本，它与 18.2 完全相同，但会为已弃用的 API 和 React 19 所需的其他变更添加警告。

我们建议先升级到 React 18.3，以便在升级到 React 19 之前帮助识别任何问题。

有关 18.3 中变更的列表，请参见 [发布说明](https://github.com/facebook/react/blob/main/CHANGELOG.md#1830-april-25-2024)。

</Note>

在这篇文章中，我们将指导你完成升级到 React 19 的步骤：

- [安装](#installing)
- [Codemod](#codemods)
- [破坏性变更](#breaking-changes)
- [新的弃用项](#new-deprecations)
- [值得注意的变更](#notable-changes)
- [TypeScript 变更](#typescript-changes)
- [更新日志](#changelog)

如果你想帮助我们测试 React 19，请按照本升级指南中的步骤操作，并[报告你遇到的任何问题](https://github.com/facebook/react/issues/new?assignees=&labels=React+19&projects=&template=19.md&title=%5BReact+19%5D)。有关 React 19 新增功能的列表，请参见 [React 19 发布文章](/blog/2024/12/05/react-19)。

---
## 安装 {/*installing*/}

<Note>

#### 现在需要新的 JSX 转换 {/*new-jsx-transform-is-now-required*/}

我们在 2020 年引入了 [新的 JSX 转换](https://legacy.reactjs.org/blog/2020/09/22/introducing-the-new-jsx-transform.html)，以改善包体积并在不导入 React 的情况下使用 JSX。在 React 19 中，我们还添加了额外的改进，例如将 ref 作为 prop 使用，以及需要新转换的 JSX 性能提升。

如果未启用新的转换，你会看到以下警告：

<ConsoleBlockMulti>

<ConsoleLogLine level="error">

你的应用（或其某个依赖）正在使用过时的 JSX 转换。请更新到现代 JSX 转换以获得更快的性能：https://react.dev/link/new-jsx-transform

</ConsoleLogLine>

</ConsoleBlockMulti>


我们预计大多数应用不会受到影响，因为在大多数环境中该转换已经启用。有关如何手动升级的说明，请参阅 [公告文章](https://legacy.reactjs.org/blog/2020/09/22/introducing-the-new-jsx-transform.html)。

</Note>


要安装最新版本的 React 和 React DOM：

```bash
npm install --save-exact react@^19.0.0 react-dom@^19.0.0
```

或者，如果你使用的是 Yarn：

```bash
yarn add --exact react@^19.0.0 react-dom@^19.0.0
```

如果你使用 TypeScript，你还需要更新类型。
```bash
npm install --save-exact @types/react@^19.0.0 @types/react-dom@^19.0.0
```

或者，如果你使用的是 Yarn：
```bash
yarn add --exact @types/react@^19.0.0 @types/react-dom@^19.0.0
```

我们还为最常见的替换提供了一个 codemod。请参见下方的 [TypeScript 变更](#typescript-changes)。

## Codemods {/*codemods*/}

为了帮助升级，我们与 [codemod.com](https://codemod.com) 团队合作发布了 codemod，它们会自动将你的代码更新为 React 19 中许多新的 API 和模式。

所有 codemod 都可在 [`react-codemod` 仓库](https://github.com/reactjs/react-codemod) 中找到，Codemod 团队也加入了维护这些 codemod 的工作。要运行这些 codemod，我们建议使用 `codemod` 命令而不是 `react-codemod`，因为它运行更快，能处理更复杂的代码迁移，并且对 TypeScript 提供更好的支持。


<Note>

#### 运行所有 React 19 codemod {/*run-all-react-19-codemods*/}

使用 React 19 的 `codemod` 配方运行本指南中列出的所有 codemod：

```bash
npx codemod@latest react/19/migration-recipe
```

这将运行来自 `react-codemod` 的以下 codemod：
- [`replace-reactdom-render`](https://github.com/reactjs/react-codemod?tab=readme-ov-file#replace-reactdom-render)
- [`replace-string-ref`](https://github.com/reactjs/react-codemod?tab=readme-ov-file#replace-string-ref)
- [`replace-act-import`](https://github.com/reactjs/react-codemod?tab=readme-ov-file#replace-act-import)
- [`replace-use-form-state`](https://github.com/reactjs/react-codemod?tab=readme-ov-file#replace-use-form-state)
- [`prop-types-typescript`](https://github.com/reactjs/react-codemod#react-proptypes-to-prop-types)

这不包括 TypeScript 变更。请参见下方的 [TypeScript 变更](#typescript-changes)。

</Note>

包含 codemod 的变更会在下面给出命令。

有关所有可用 codemod 的列表，请参见 [`react-codemod` 仓库](https://github.com/reactjs/react-codemod)。

## 破坏性变更 {/*breaking-changes*/}

### 渲染中的错误不会被重新抛出 {/*errors-in-render-are-not-re-thrown*/}

在以前版本的 React 中，在渲染期间抛出的错误会被捕获并重新抛出。在开发环境中，我们还会记录到 `console.error`，从而导致重复的错误日志。

在 React 19 中，我们通过[改进错误处理方式](/blog/2024/04/25/react-19#error-handling)来减少重复，不再重新抛出：

- **未捕获错误**：未被 Error Boundary 捕获的错误会报告给 `window.reportError`。
- **已捕获错误**：被 Error Boundary 捕获的错误会报告给 `console.error`。

这个变更不应该影响大多数应用，但如果你的生产环境错误报告依赖于错误被重新抛出，你可能需要更新错误处理。为此，我们为 `createRoot` 和 `hydrateRoot` 添加了新的方法，用于自定义错误处理：

```js [[1, 2, "onUncaughtError"], [2, 5, "onCaughtError"]]
const root = createRoot(container, {
  onUncaughtError: (error, errorInfo) => {
    // ... 记录错误报告
  },
  onCaughtError: (error, errorInfo) => {
    // ... 记录错误报告
  }
});
```

有关更多信息，请参阅 [`createRoot`](https://react.dev/reference/react-dom/client/createRoot) 和 [`hydrateRoot`](https://react.dev/reference/react-dom/client/hydrateRoot) 的文档。


### 移除了已弃用的 React API {/*removed-deprecated-react-apis*/}

#### 移除：函数的 `propTypes` 和 `defaultProps` {/*removed-proptypes-and-defaultprops*/}
`PropTypes` 已在 [2017 年 4 月（v15.5.0）](https://legacy.reactjs.org/blog/2017/04/07/react-v15.5.0.html#new-deprecation-warnings) 中弃用。

在 React 19 中，我们将从 React 包中移除 `propType` 检查，使用它们将被静默忽略。如果你正在使用 `propTypes`，我们建议迁移到 TypeScript 或其他类型检查方案。

我们还将从函数组件中移除 `defaultProps`，改用 ES6 默认参数。类组件仍将继续支持 `defaultProps`，因为没有 ES6 的替代方案。

```js
// 之前
import PropTypes from 'prop-types';

function Heading({text}) {
  return <h1>{text}</h1>;
}
Heading.propTypes = {
  text: PropTypes.string,
};
Heading.defaultProps = {
  text: 'Hello, world!',
};
```
```ts
// 之后
interface Props {
  text?: string;
}
function Heading({text = 'Hello, world!'}: Props) {
  return <h1>{text}</h1>;
}
```

<Note>

使用以下命令将 `propTypes` 转换为 TypeScript：

```bash
npx codemod@latest react/prop-types-typescript
```

</Note>

#### 移除：使用 `contextTypes` 和 `getChildContext` 的旧版 Context {/*removed-removing-legacy-context*/}

旧版 Context 已在 [2018 年 10 月（v16.6.0）](https://legacy.reactjs.org/blog/2018/10/23/react-v-16-6.html) 中弃用。

旧版 Context 只在使用 `contextTypes` 和 `getChildContext` 这些 API 的类组件中可用，并由于一些容易被忽视的细微 bug 而被 `contextType` 取代。在 React 19 中，我们移除了旧版 Context，以使 React 稍微更小、更快。

如果你仍在类组件中使用旧版 Context，你需要迁移到新的 `contextType` API：

```js {5-11,19-21}
// 之前
import PropTypes from 'prop-types';

class Parent extends React.Component {
  static childContextTypes = {
    foo: PropTypes.string.isRequired,
  };

  getChildContext() {
    return { foo: 'bar' };
  }

  render() {
    return <Child />;
  }
}

class Child extends React.Component {
  static contextTypes = {
    foo: PropTypes.string.isRequired,
  };

  render() {
    return <div>{this.context.foo}</div>;
  }
}
```

```js {2,7,9,15}
// 之后
const FooContext = React.createContext();

class Parent extends React.Component {
  render() {
    return (
      <FooContext value='bar'>
        <Child />
      </FooContext>
    );
  }
}

class Child extends React.Component {
  static contextType = FooContext;

  render() {
    return <div>{this.context}</div>;
  }
}
```

#### 移除：字符串 refs {/*removed-string-refs*/}
字符串 refs 已在 [2018 年 3 月（v16.3.0）](https://legacy.reactjs.org/blog/2018/03/27/update-on-async-rendering.html) 中弃用。

类组件在被 ref 回调替代之前支持字符串 refs，原因是其[有多个缺点](https://github.com/facebook/react/issues/1373)。在 React 19 中，我们将移除字符串 refs，以使 React 更简单、更易理解。

如果你仍在类组件中使用字符串 refs，你需要迁移到 ref 回调：

```js {4,8}
// 之前
class MyComponent extends React.Component {
  componentDidMount() {
    this.refs.input.focus();
  }

  render() {
    return <input ref='input' />;
  }
}
```

```js {4,8}
// 之后
class MyComponent extends React.Component {
  componentDidMount() {
    this.input.focus();
  }

  render() {
    return <input ref={input => this.input = input} />;
  }
}
```

<Note>

使用 `ref` 回调的方式将字符串 refs 转换：

```bash
npx codemod@latest react/19/replace-string-ref
```

</Note>

#### 移除：模块模式工厂 {/*removed-module-pattern-factories*/}
模块模式工厂已在 [2019 年 8 月（v16.9.0）](https://legacy.reactjs.org/blog/2019/08/08/react-v16.9.0.html#deprecating-module-pattern-factories) 中弃用。

这种模式很少被使用，支持它会让 React 变得稍大且稍慢。 在 React 19 中，我们将移除对模块模式工厂的支持，你需要迁移到普通函数：

```js
// 之前
function FactoryComponent() {
  return { render() { return <div />; } }
}
```

```js
// 之后
function FactoryComponent() {
  return <div />;
}
```

#### 移除：`React.createFactory` {/*removed-createfactory*/}
`createFactory` 已在 [2020 年 2 月（v16.13.0）](https://legacy.reactjs.org/blog/2020/02/26/react-v16.13.0.html#deprecating-createfactory) 中弃用。

在 JSX 获得广泛支持之前，使用 `createFactory` 很常见，但如今它很少使用，可以被 JSX 取代。在 React 19 中，我们将移除 `createFactory`，你需要迁移到 JSX：

```js
// 之前
import { createFactory } from 'react';

const button = createFactory('button');
```

```js
// 之后
const button = <button />;
```

#### 移除：`react-test-renderer/shallow` {/*removed-react-test-renderer-shallow*/}

在 React 18 中，我们更新了 `react-test-renderer/shallow`，使其重新导出 [react-shallow-renderer](https://github.com/enzymejs/react-shallow-renderer)。在 React 19 中，我们将移除 `react-test-render/shallow`，建议直接安装该包：

```bash
npm install react-shallow-renderer --save-dev
```
```diff
- import ShallowRenderer from 'react-test-renderer/shallow';
+ import ShallowRenderer from 'react-shallow-renderer';
```

<Note>

##### 请重新考虑浅渲染 {/*please-reconsider-shallow-rendering*/}

浅渲染依赖于 React 内部实现，可能会阻碍你未来的升级。我们建议将测试迁移到 [@testing-library/react](https://testing-library.com/docs/react-testing-library/intro/) 或 [@testing-library/react-native](https://testing-library.com/docs/react-native-testing-library/intro)。

</Note>

### 移除了已弃用的 React DOM API {/*removed-deprecated-react-dom-apis*/}

#### 移除：`react-dom/test-utils` {/*removed-react-dom-test-utils*/}

我们已将 `act` 从 `react-dom/test-utils` 移至 `react` 包：

<ConsoleBlockMulti>

<ConsoleLogLine level="error">

`ReactDOMTestUtils.act` 已弃用，请改用 `React.act`。请从 `react` 而不是 `react-dom/test-utils` 导入 `act`。更多信息请参见 https://react.dev/warnings/react-dom-test-utils。

</ConsoleLogLine>

</ConsoleBlockMulti>

要修复此警告，你可以从 `react` 导入 `act`：

```diff
- import {act} from 'react-dom/test-utils'
+ import {act} from 'react';
```

所有其他 `test-utils` 函数都已移除。这些工具使用并不常见，而且使得过于容易依赖组件和 React 的底层实现细节。在 React 19 中，这些函数在调用时会报错，并且它们的导出将在未来版本中移除。

有关替代方案，请参见[警告页面](https://react.dev/warnings/react-dom-test-utils)。

<Note>

将 `ReactDOMTestUtils.act` 转换为 `React.act`：

```bash
npx codemod@latest react/19/replace-act-import
```

</Note>

#### 移除：`ReactDOM.render` {/*removed-reactdom-render*/}

`ReactDOM.render` 已在 [2022 年 3 月（v18.0.0）](https://react.dev/blog/2022/03/08/react-18-upgrade-guide) 中弃用。在 React 19 中，我们将移除 `ReactDOM.render`，你需要迁移到使用 [`ReactDOM.createRoot`](https://react.dev/reference/react-dom/client/createRoot)：

```js
// 之前
import {render} from 'react-dom';
render(<App />, document.getElementById('root'));

// 之后
import {createRoot} from 'react-dom/client';
const root = createRoot(document.getElementById('root'));
root.render(<App />);
```

<Note>

将 `ReactDOM.render` 转换为 `ReactDOMClient.createRoot`：

```bash
npx codemod@latest react/19/replace-reactdom-render
```

</Note>

#### 移除：`ReactDOM.hydrate` {/*removed-reactdom-hydrate*/}

`ReactDOM.hydrate` 已在 [2022 年 3 月（v18.0.0）](https://react.dev/blog/2022/03/08/react-18-upgrade-guide) 中弃用。在 React 19 中，我们将移除 `ReactDOM.hydrate`，你需要迁移到使用 [`ReactDOM.hydrateRoot`](https://react.dev/reference/react-dom/client/hydrateRoot)，

```js
// 之前
import {hydrate} from 'react-dom';
hydrate(<App />, document.getElementById('root'));

// 之后
import {hydrateRoot} from 'react-dom/client';
hydrateRoot(document.getElementById('root'), <App />);
```

<Note>

将 `ReactDOM.hydrate` 转换为 `ReactDOMClient.hydrateRoot`：

```bash
npx codemod@latest react/19/replace-reactdom-render
```

</Note>

#### 移除：`unmountComponentAtNode` {/*removed-unmountcomponentatnode*/}

`ReactDOM.unmountComponentAtNode` 已在 [2022 年 3 月（v18.0.0）](https://react.dev/blog/2022/03/08/react-18-upgrade-guide) 中弃用。在 React 19 中，你需要迁移到使用 `root.unmount()`。


```js
// 之前
unmountComponentAtNode(document.getElementById('root'));

// 之后
root.unmount();
```

更多信息请参见 [`createRoot`](https://react.dev/reference/react-dom/client/createRoot#root-unmount) 和 [`hydrateRoot`](https://react.dev/reference/react-dom/client/hydrateRoot#root-unmount) 的 `root.unmount()`。

<Note>

将 `unmountComponentAtNode` 转换为 `root.unmount`：

```bash
npx codemod@latest react/19/replace-reactdom-render
```

</Note>

#### 移除：`ReactDOM.findDOMNode` {/*removed-reactdom-finddomnode*/}

`ReactDOM.findDOMNode` 已在 [2018 年 10 月（v16.6.0）](https://legacy.reactjs.org/blog/2018/10/23/react-v-16-6.html#deprecations-in-strictmode) 中弃用。

我们移除 `findDOMNode` 是因为它是一个旧的逃生通道，执行速度慢、重构脆弱、只返回第一个子节点，并且破坏了抽象层级（更多信息见[这里](https://legacy.reactjs.org/docs/strict-mode.html#warning-about-deprecated-finddomnode-usage)）。你可以将 `ReactDOM.findDOMNode` 替换为 [DOM refs](/learn/manipulating-the-dom-with-refs)：

```js
// 之前
import {findDOMNode} from 'react-dom';

function AutoselectingInput() {
  useEffect(() => {
    const input = findDOMNode(this);
    input.select()
  }, []);

  return <input defaultValue="Hello" />;
}
```

```js
// 之后
function AutoselectingInput() {
  const ref = useRef(null);
  useEffect(() => {
    ref.current.select();
  }, []);

  return <input ref={ref} defaultValue="Hello" />
}
```

## 新增弃用项 {/*new-deprecations*/}

### 已弃用：`element.ref` {/*deprecated-element-ref*/}

React 19 支持将 [`ref` 作为属性](/blog/2024/04/25/react-19#ref-as-a-prop)，因此我们将弃用 `element.ref`，改为使用 `element.props.ref`。

访问 `element.ref` 会显示警告：

<ConsoleBlockMulti>

<ConsoleLogLine level="error">

访问 element.ref 已不再受支持。ref 现在是一个普通属性。它将在未来版本中从 JSX Element 类型中移除。

</ConsoleLogLine>

</ConsoleBlockMulti>

### 已弃用：`react-test-renderer` {/*deprecated-react-test-renderer*/}

我们正在弃用 `react-test-renderer`，因为它实现了自己的一套渲染器环境，与用户实际使用的环境不一致，会促进对实现细节的测试，并且依赖对 React 内部实现的 introspection。

测试渲染器是在还没有像 [React Testing Library](https://testing-library.com) 这样更可行的测试策略之前创建的，而我们现在建议改用现代测试库。

在 React 19 中，`react-test-renderer` 会记录一条弃用警告，并已切换到并发渲染。我们建议将测试迁移到 [@testing-library/react](https://testing-library.com/docs/react-testing-library/intro/) 或 [@testing-library/react-native](https://testing-library.com/docs/react-native-testing-library/intro)，以获得现代且支持完善的测试体验。

## 重要变更 {/*notable-changes*/}

### StrictMode 变更 {/*strict-mode-improvements*/}

React 19 包含了对 Strict Mode 的若干修复和改进。

在开发环境下，Strict Mode 进行双重渲染时，`useMemo` 和 `useCallback` 会在第二次渲染中复用第一次渲染的 memoized 结果。已经兼容 Strict Mode 的组件不应注意到行为差异。

与所有 Strict Mode 行为一样，这些特性旨在在开发过程中主动暴露组件中的 bug，以便你能在它们发布到生产环境之前修复。例如，在开发环境中，Strict Mode 会在初始挂载时对 ref 回调函数进行两次调用，以模拟挂载的组件被 Suspense fallback 替换时会发生什么。

### 对 Suspense 的改进 {/*improvements-to-suspense*/}

在 React 19 中，当组件挂起时，React 会立即提交最近的 Suspense 边界的 fallback，而不会等待整个兄弟树渲染完成。在 fallback 提交后，React 会为挂起的兄弟节点安排另一次渲染，以便“预热”树中其余部分的懒加载请求：

<Diagram name="prerender" height={162} width={1270} alt="Diagram showing a tree of three components, one parent labeled Accordion and two children labeled Panel. Both Panel components contain isActive with value false.">

此前，当组件挂起时，会先渲染挂起的兄弟节点，然后再提交 fallback。

</Diagram>

<Diagram name="prewarm" height={162} width={1270} alt="The same diagram as the previous, with the isActive of the first child Panel component highlighted indicating a click with the isActive value set to true. The second Panel component still contains value false." >

在 React 19 中，当组件挂起时，会先提交 fallback，然后再渲染挂起的兄弟节点。

</Diagram>

这一变化意味着 Suspense fallback 显示得更快，同时仍会为挂起树中的懒加载请求预热。

### 已移除 UMD 构建 {/*umd-builds-removed*/}

过去，UMD 作为一种无需构建步骤即可加载 React 的便捷方式被广泛使用。现在，HTML 文档中作为脚本加载模块已有现代替代方案。从 React 19 开始，React 将不再生成 UMD 构建，以降低测试和发布流程的复杂度。

要通过 script 标签加载 React 19，我们建议使用基于 ESM 的 CDN，例如 [esm.sh](https://esm.sh/)。

```html
<script type="module">
  import React from "https://esm.sh/react@19/?dev"
  import ReactDOMClient from "https://esm.sh/react-dom@19/client?dev"
  ...
</script>
```

### 依赖 React 内部实现的库可能会阻止升级 {/*libraries-depending-on-react-internals-may-block-upgrades*/}

此版本包含对 React 内部实现的更改，这可能会影响那些忽视我们不要使用内部实现（例如 `SECRET_INTERNALS_DO_NOT_USE_OR_YOU_WILL_BE_FIRED`）的请求的库。这些更改对于在 React 19 中落地改进是必要的，并不会破坏遵循我们指南的库。

根据我们的 [版本策略](https://react.dev/community/versioning-policy#what-counts-as-a-breaking-change)，这些更新不被列为破坏性变更，我们也不会提供如何升级它们的文档。建议移除任何依赖内部实现的代码。

为了体现使用内部实现的影响，我们已将 `SECRET_INTERNALS` 后缀重命名为：

`_DO_NOT_USE_OR_WARN_USERS_THEY_CANNOT_UPGRADE`

未来，我们会更积极地阻止从 React 访问内部实现，以劝阻这种用法，并确保用户不会因升级受阻。

## TypeScript 变更 {/*typescript-changes*/}

### 移除了已弃用的 TypeScript 类型 {/*removed-deprecated-typescript-types*/}

我们已根据 React 19 中移除的 API 清理了 TypeScript 类型。其中一些已移除内容的类型已迁移到更相关的包中，其他则不再需要来描述 React 的行为。

<Note>
我们已发布 [`types-react-codemod`](https://github.com/eps1lon/types-react-codemod/)，用于迁移大多数与类型相关的破坏性变更：

```bash
npx types-react-codemod@latest preset-19 ./path-to-app
```

如果你对 `element.props` 有大量不安全访问，可以运行这个额外的 codemod：

```bash
npx types-react-codemod@latest react-element-default-any-props ./path-to-your-react-ts-files
```

</Note>

查看 [`types-react-codemod`](https://github.com/eps1lon/types-react-codemod/) 以获取受支持替换项的列表。如果你觉得缺少某个 codemod，可以在 [缺失的 React 19 codemod 列表](https://github.com/eps1lon/types-react-codemod/issues?q=is%3Aissue+is%3Aopen+sort%3Aupdated-desc+label%3A%22React+19%22+label%3Aenhancement) 中跟踪。

### 需要清理 `ref` {/*ref-cleanup-required*/}

_此更改已包含在 `react-19` codemod 预设中，名称为 [`no-implicit-ref-callback-return`](https://github.com/eps1lon/types-react-codemod/#no-implicit-ref-callback-return)。_

由于引入了 ref 清理函数，现在 ref 回调返回任何其他内容都会被 TypeScript 拒绝。修复方法通常是停止使用隐式返回：

```diff [[1, 1, "("], [1, 1, ")"], [2, 2, "{", 15], [2, 2, "}", 1]]
- <div ref={current => (instance = current)} />
+ <div ref={current => {instance = current}} />
```

原始代码返回了 `HTMLDivElement` 的实例，而 TypeScript 无法判断这是否本应是一个清理函数。

### `useRef` 需要参数 {/*useref-requires-argument*/}

_此更改已包含在 `react-19` codemod 预设中，名称为 [`refobject-defaults`](https://github.com/eps1lon/types-react-codemod/#refobject-defaults)。_

长期以来，TypeScript 和 React 的一个争议点一直是 `useRef`。我们已更改类型，使得 `useRef` 现在需要一个参数。这大大简化了它的类型签名。它现在会更像 `createContext`。

```ts
// @ts-expect-error: 需要 1 个参数，但未提供
useRef();
// 通过
useRef(undefined);
// @ts-expect-error: 需要 1 个参数，但未提供
createContext();
// 通过
createContext(undefined);
```

这也意味着现在所有 ref 都是可变的。你将不再遇到因为用 `null` 初始化而无法修改 ref 的问题：

```ts
const ref = useRef<number>(null);

// 无法为 'current' 赋值，因为它是只读属性
ref.current = 1;
```

`MutableRef` 现在已弃用，改为使用单一的 `RefObject` 类型，`useRef` 将始终返回该类型：

```ts
interface RefObject<T> {
  current: T
}

declare function useRef<T>: RefObject<T>
```

`useRef` 仍然保留了一个便捷重载 `useRef<T>(null)`，它会自动返回 `RefObject<T | null>`。为了缓解因 `useRef` 需要参数而带来的迁移难度，我们增加了一个便捷重载 `useRef(undefined)`，它会自动返回 `RefObject<T | undefined>`。

关于此变更的先前讨论，请查看 [[RFC] Make all refs mutable](https://github.com/DefinitelyTyped/DefinitelyTyped/pull/64772)。

### `ReactElement` TypeScript 类型的变更 {/*changes-to-the-reactelement-typescript-type*/}

_此更改已包含在 [`react-element-default-any-props`](https://github.com/eps1lon/types-react-codemod#react-element-default-any-props) codemod 中。_

如果元素被类型化为 `ReactElement`，React 元素的 `props` 现在默认是 `unknown`，而不是 `any`。如果你为 `ReactElement` 提供了类型参数，这不会影响你：

```ts
type Example2 = ReactElement<{ id: string }>["props"];
//   ^? { id: string }
```

但如果你依赖默认值，现在就必须处理 `unknown`：

```ts
type Example = ReactElement["props"];
//   ^? 以前是 'any'，现在是 'unknown'
```

只有在你有大量依赖于不安全访问元素 props 的旧代码时，才需要这个改动。元素 introspection 只是一种逃生舱口，你应通过显式的 `any` 明确表示你的 props 访问是不安全的。

### TypeScript 中的 JSX 命名空间 {/*the-jsx-namespace-in-typescript*/}
此更改已包含在 `react-19` codemod 预设中，名称为 [`scoped-jsx`](https://github.com/eps1lon/types-react-codemod#scoped-jsx)

一个长期以来的请求是从我们的类型中移除全局 `JSX` 命名空间，改为使用 `React.JSX`。这有助于防止全局类型污染，并避免不同利用 JSX 的 UI 库之间发生冲突。

现在你需要将 JSX 命名空间的模块增强包装在 `declare module "...."` 中：

```diff
// global.d.ts
+ declare module "react" {
    namespace JSX {
      interface IntrinsicElements {
        "my-element": {
          myElementProps: string;
        };
      }
    }
+ }
```

具体的模块标识符取决于你在 `tsconfig.json` 的 `compilerOptions` 中指定的 JSX 运行时：

- 对于 `"jsx": "react-jsx"`，应为 `react/jsx-runtime`。
- 对于 `"jsx": "react-jsxdev"`，应为 `react/jsx-dev-runtime`。
- 对于 `"jsx": "react"` 和 `"jsx": "preserve"`，应为 `react`。

### 更好的 `useReducer` 类型推断 {/*better-usereducer-typings*/}

由于 [@mfp22](https://github.com/mfp22) 的贡献，`useReducer` 现在拥有了改进的类型推断。

然而，这也带来了一个破坏性变更：`useReducer` 不再接受完整的 reducer 类型作为类型参数，而是要么不传类型参数（依赖上下文类型推断），要么同时提供 state 和 action 类型。

新的最佳实践是 _不要_ 向 `useReducer` 传递类型参数。
```diff
- useReducer<React.Reducer<State, Action>>(reducer)
+ useReducer(reducer)
```
在某些边缘情况下，这可能不起作用；你可以通过将 `Action` 放入元组中来显式指定 state 和 action：
```diff
- useReducer<React.Reducer<State, Action>>(reducer)
+ useReducer<State, [Action]>(reducer)
```
如果你在内联定义 reducer，我们建议改为标注函数参数：
```diff
- useReducer<React.Reducer<State, Action>>((state, action) => state)
+ useReducer((state: State, action: Action) => state)
```
如果你把 reducer 移到 `useReducer` 调用之外，也同样需要这样做：

```ts
const reducer = (state: State, action: Action) => state;
```

## 更新日志 {/*changelog*/}

### 其他破坏性变更 {/*other-breaking-changes*/}

- **react-dom**: `src` 和 `href` 中的 javascript URL 会报错 [#26507](https://github.com/facebook/react/pull/26507)
- **react-dom**: 从 `onRecoverableError` 中移除 `errorInfo.digest` [#28222](https://github.com/facebook/react/pull/28222)
- **react-dom**: 移除 `unstable_flushControlled` [#26397](https://github.com/facebook/react/pull/26397)
- **react-dom**: 移除 `unstable_createEventHandle` [#28271](https://github.com/facebook/react/pull/28271)
- **react-dom**: 移除 `unstable_renderSubtreeIntoContainer` [#28271](https://github.com/facebook/react/pull/28271)
- **react-dom**: 移除 `unstable_runWithPriority` [#28271](https://github.com/facebook/react/pull/28271)
- **react-is**: 从 `react-is` 中移除已弃用的方法 [28224](https://github.com/facebook/react/pull/28224)

### 其他值得注意的变更 {/*other-notable-changes*/}

- **react**: 批量同步、默认和连续的 lanes [#25700](https://github.com/facebook/react/pull/25700)
- **react**: 不再预渲染已挂起组件的兄弟组件 [#26380](https://github.com/facebook/react/pull/26380)
- **react**: 检测由渲染阶段更新导致的无限更新循环 [#26625](https://github.com/facebook/react/pull/26625)
- **react-dom**: popstate 中的过渡现在是同步的 [#26025](https://github.com/facebook/react/pull/26025)
- **react-dom**: 移除 SSR 期间的布局效果警告 [#26395](https://github.com/facebook/react/pull/26395)
- **react-dom**: 对 src/href 为空字符串时发出警告且不再设置为空字符串（锚点标签除外） [#28124](https://github.com/facebook/react/pull/28124)

完整的变更列表请参阅 [更新日志](https://github.com/facebook/react/blob/main/CHANGELOG.md#1900-december-5-2024)。

---

感谢 [Andrew Clark](https://twitter.com/acdlite)、[Eli White](https://twitter.com/Eli_White)、[Jack Pope](https://github.com/jackpope)、[Jan Kassens](https://github.com/kassens)、[Josh Story](https://twitter.com/joshcstory)、[Matt Carroll](https://twitter.com/mattcarrollcode)、[Noah Lemen](https://twitter.com/noahlemen)、[Sophie Alpert](https://twitter.com/sophiebits) 和 [Sebastian Silbermann](https://twitter.com/sebsilbermann) 对本文进行审阅和编辑。
