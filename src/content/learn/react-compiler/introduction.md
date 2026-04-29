---
title: 介绍
---

<Intro>
React Compiler 是一个新的构建时工具，它会自动优化你的 React 应用。它可与普通 JavaScript 配合使用，并且能够理解 [React 规则](/reference/rules)，因此你无需重写任何代码即可使用它。
</Intro>

<YouWillLearn>

* React Compiler 的作用
* 如何开始使用编译器
* 渐进式采用策略
* 当出现问题时如何调试和排查
* 如何在你的 React 库中使用编译器

</YouWillLearn>

## React Compiler 做什么？ {/*what-does-react-compiler-do*/}

React Compiler 会在构建时自动优化你的 React 应用。React 在不做优化的情况下通常已经足够快，但有时你需要手动对组件和值进行记忆化，才能保持应用的响应性。这种手动记忆化既繁琐，又容易出错，还会增加额外的维护代码。React Compiler 会自动为你完成这些优化，让你从这种心智负担中解放出来，从而专注于构建功能。

### 在 React Compiler 之前 {/*before-react-compiler*/}

如果没有编译器，你需要手动对组件和值进行记忆化，以优化重新渲染：

```js
import { useMemo, useCallback, memo } from 'react';

const ExpensiveComponent = memo(function ExpensiveComponent({ data, onClick }) {
  const processedData = useMemo(() => {
    return expensiveProcessing(data);
  }, [data]);

  const handleClick = useCallback((item) => {
    onClick(item.id);
  }, [onClick]);

  return (
    <div>
      {processedData.map(item => (
        <Item key={item.id} onClick={() => handleClick(item)} />
      ))}
    </div>
  );
});
```


<Note>

这种手动记忆化有一个微妙的 bug，会破坏记忆化：

```js [[2, 1, "() => handleClick(item)"]]
<Item key={item.id} onClick={() => handleClick(item)} />
```

尽管 `handleClick` 被 `useCallback` 包裹了，箭头函数 `() => handleClick(item)` 仍会在组件每次渲染时创建一个新函数。这意味着 `Item` 总是会接收到一个新的 `onClick` prop，从而破坏记忆化。

React Compiler 能够在有或没有箭头函数的情况下都正确优化这一点，确保只有当 `props.onClick` 改变时 `Item` 才会重新渲染。

</Note>

### 在 React Compiler 之后 {/*after-react-compiler*/}

有了 React Compiler，你可以编写相同的代码，而无需手动记忆化：

```js
function ExpensiveComponent({ data, onClick }) {
  const processedData = expensiveProcessing(data);

  const handleClick = (item) => {
    onClick(item.id);
  };

  return (
    <div>
      {processedData.map(item => (
        <Item key={item.id} onClick={() => handleClick(item)} />
      ))}
    </div>
  );
}
```

_[在 React Compiler Playground 中查看此示例](https://playground.react.dev/#N4Igzg9grgTgxgUxALhAMygOzgFwJYSYAEAogB4AOCmYeAbggMIQC2Fh1OAFMEQCYBDHAIA0RQowA2eOAGsiAXwCURYAB1iROITA4iFGBERgwCPgBEhAogF4iCStVoMACoeO1MAcy6DhSgG4NDSItHT0ACwFMPkkmaTlbIi48HAQWFRsAPlUQ0PFMKRlZFLSWADo8PkC8hSDMPJgEHFhiLjzQgB4+eiyO-OADIwQTM0thcpYBClL02xz2zXz8zoBJMqJZBABPG2BU9Mq+BQKiuT2uTJyomLizkoOMk4B6PqX8pSUFfs7nnro3qEapgFCAFEA)_

React Compiler 会自动应用最优的记忆化，确保你的应用只在必要时重新渲染。

<DeepDive>
#### React Compiler 会添加什么类型的记忆化？ {/*what-kind-of-memoization-does-react-compiler-add*/}

React Compiler 的自动记忆化主要聚焦于**提升更新性能**（重新渲染现有组件），因此它主要针对以下两种用例：

1. **跳过组件的级联重新渲染**
    * 重新渲染 `<Parent />` 会导致其组件树中的许多组件重新渲染，即使只有 `<Parent />` 发生了变化
1. **跳过来自 React 之外的昂贵计算**
    * 例如，在需要这些数据的组件或 hook 内调用 `expensivelyProcessAReallyLargeArrayOfObjects()`

#### 优化重新渲染 {/*optimizing-re-renders*/}

React 允许你将 UI 表达为其当前状态的函数（更具体地说：它们的 props、state 和 context）。在当前实现中，当组件的 state 改变时，React 会重新渲染该组件 _以及它的所有子组件_ —— 除非你使用 `useMemo()`、`useCallback()` 或 `React.memo()` 应用了某种手动记忆化。例如，在下面的示例中，只要 `<FriendList>` 的 state 发生变化，`<MessageButton>` 就会重新渲染：

```javascript
function FriendList({ friends }) {
  const onlineCount = useFriendOnlineCount();
  if (friends.length === 0) {
    return <NoFriends />;
  }
  return (
    <div>
      <span>{onlineCount} online</span>
      {friends.map((friend) => (
        <FriendListCard key={friend.id} friend={friend} />
      ))}
      <MessageButton />
    </div>
  );
}
```
[_在 React Compiler Playground 中查看此示例_](https://playground.react.dev/#N4Igzg9grgTgxgUxALhAMygOzgFwJYSYAEAYjHgpgCYAyeYOAFMEWuZVWEQL4CURwADrEicQgyKEANnkwIAwtEw4iAXiJQwCMhWoB5TDLmKsTXgG5hRInjRFGbXZwB0UygHMcACzWr1ABn4hEWsYBBxYYgAeADkIHQ4uAHoAPksRbisiMIiYYkYs6yiqPAA3FMLrIiiwAAcAQ0wU4GlZBSUcbklDNqikusaKkKrgR0TnAFt62sYHdmp+VRT7SqrqhOo6Bnl6mCoiAGsEAE9VUfmqZzwqLrHqM7ubolTVol5eTOGigFkEMDB6u4EAAhKA4HCEZ5DNZ9ErlLIWYTcEDcIA)

React Compiler 会自动应用与手动记忆化等效的优化，确保随着 state 变化，应用中只有相关部分会重新渲染，这有时也被称为“细粒度响应性”。在上面的示例中，React Compiler 会判断 `<FriendListCard />` 的返回值即使在 `friends` 变化时也可以复用，并且可以避免重新创建这段 JSX，同时还能在计数变化时避免重新渲染 `<MessageButton>`。

#### 昂贵计算也会被记忆化 {/*expensive-calculations-also-get-memoized*/}

React Compiler 还可以自动对渲染期间使用的昂贵计算进行记忆化：

```js
// **不会** 被 React Compiler 记忆化，因为这不是组件或 hook
function expensivelyProcessAReallyLargeArrayOfObjects() { /* ... */ }

// 会被 React Compiler 记忆化，因为这是一个组件
function TableContainer({ items }) {
  // 这个函数调用会被记忆化：
  const data = expensivelyProcessAReallyLargeArrayOfObjects(items);
  // ...
}
```
[_在 React Compiler Playground 中查看此示例_](https://playground.react.dev/#N4Igzg9grgTgxgUxALhAejQAgFTYHIQAuumAtgqRAJYBeCAJpgEYCemASggIZyGYDCEUgAcqAGwQwANJjBUAdokyEAFlTCZ1meUUxdMcIcIjyE8vhBiYVECAGsAOvIBmURYSonMCAB7CzcgBuCGIsAAowEIhgYACCnFxioQAyXDAA5gixMDBcLADyzvlMAFYIvGAAFACUmMCYaNiYAHStOFgAvk5OGJgAshTUdIysHNy8AkbikrIKSqpaWvqGIiZmhE6u7p7ymAAqXEwSguZcCpKV9VSEFBodtcBOmAYmYHz0XIT6ALzefgFUYKhCJRBAxeLcJIsVIZLI5PKFYplCqVa63aoAbm6u0wMAQhFguwAPPRAQA+YAfL4dIloUmBMlODogDpAA)

不过，如果 `expensivelyProcessAReallyLargeArrayOfObjects` 确实是一个昂贵函数，你可能需要考虑在 React 之外为它实现自己的记忆化，因为：

- React Compiler 只会记忆化 React 组件和 hooks，不会记忆化每一个函数
- React Compiler 的记忆化不会在多个组件或 hooks 之间共享

因此，如果多个不同组件都使用了 `expensivelyProcessAReallyLargeArrayOfObjects`，那么即使传入的都是完全相同的 items，这个昂贵计算也会被反复执行。我们建议先进行[性能分析](reference/react/useMemo#how-to-tell-if-a-calculation-is-expensive)，确认它是否真的那么昂贵，然后再让代码变得更复杂。
</DeepDive>

## 我应该试用这个编译器吗？ {/*should-i-try-out-the-compiler*/}

我们鼓励每个人开始使用 React Compiler。虽然目前编译器对 React 来说仍然是一个可选项，但未来某些特性可能会要求必须使用编译器才能完全工作。

### 使用它安全吗？ {/*is-it-safe-to-use*/}

React Compiler 现在已经稳定，并且已经在生产环境中经过了广泛测试。虽然它已经在 Meta 等公司的生产环境中使用过，但将编译器推广到你应用的生产环境中，仍然取决于你代码库的健康状况，以及你对 [React 规则](/reference/rules) 的遵循程度。

## 支持哪些构建工具？ {/*what-build-tools-are-supported*/}

React Compiler 可以安装在 [多种构建工具](/learn/react-compiler/installation) 中，例如 Babel、Vite、Metro 和 Rsbuild。

React Compiler 主要是核心编译器外面的一层轻量 Babel 插件封装，而核心编译器本身被设计为与 Babel 解耦。虽然编译器的初始稳定版本仍将主要以 Babel 插件形式存在，但我们正在与 swc 和 [oxc](https://github.com/oxc-project/oxc/issues/10048) 团队合作，为 React Compiler 构建一流支持，这样未来你就不必再把 Babel 加回到你的构建流水线中了。

Next.js 用户可以通过使用 [v15.3.1](https://github.com/vercel/next.js/releases/tag/v15.3.1) 及以上版本来启用由 swc 调用的 React Compiler。

## 我应该如何处理 useMemo、useCallback 和 React.memo？ {/*what-should-i-do-about-usememo-usecallback-and-reactmemo*/}

默认情况下，React Compiler 会基于其分析和启发式规则对你的代码进行记忆化。在大多数情况下，这种记忆化会与你可能手写的版本一样精确，甚至更精确。

不过，在某些情况下，开发者可能需要对记忆化有更多控制。`useMemo` 和 `useCallback` hooks 仍然可以与 React Compiler 一起使用，作为一种“后门”来控制哪些值会被记忆化。一个常见用例是将记忆化后的值作为 effect 的依赖项，以确保即使其依赖项没有实质性变化，effect 也不会反复触发。

对于新代码，我们建议依赖编译器来进行记忆化，并在需要精确控制时使用 `useMemo`/`useCallback`。

对于现有代码，我们建议保留已有的记忆化不变（移除它可能会改变编译输出），或者在移除记忆化之前仔细测试。

## 试用 React Compiler {/*try-react-compiler*/}

本节将帮助你开始使用 React Compiler，并理解如何在项目中有效地使用它。

* **[安装](/learn/react-compiler/installation)** - 安装 React Compiler 并为你的构建工具进行配置
* **[React 版本兼容性](/reference/react-compiler/target)** - 支持 React 17、18 和 19
* **[配置](/reference/react-compiler/configuration)** - 根据你的具体需求自定义编译器
* **[渐进式采用](/learn/react-compiler/incremental-adoption)** - 在现有代码库中逐步推广编译器的策略
* **[调试与排查](/learn/react-compiler/debugging)** - 在使用编译器时识别并修复问题
* **[编译库](/reference/react-compiler/compiling-libraries)** - 发布已编译代码的最佳实践
* **[API 参考](/reference/react-compiler/configuration)** - 所有配置选项的详细文档

## 其他资源 {/*additional-resources*/}

除了这些文档外，我们还建议查看 [React Compiler Working Group](https://github.com/reactwg/react-compiler)，以获取有关编译器的更多信息和讨论。

