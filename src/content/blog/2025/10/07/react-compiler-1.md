---
title: "React Compiler v1.0"
author: Lauren Tan, Joe Savona, and Mofei Zhang
date: 2025/10/07
description: 我们今天发布编译器的第一个稳定版。

---

2025年10月7日，作者：[Lauren Tan](https://x.com/potetotes)、[Joe Savona](https://x.com/en_JS) 和 [Mofei Zhang](https://x.com/zmofei)。

---

<Intro>

React 团队很高兴分享最新进展：

</Intro>

1. React Compiler 1.0 今日可用。
2. 编译器驱动的 lint 规则已随 `eslint-plugin-react-hooks` 的 `recommended` 和 `recommended-latest` 预设一同发布。
3. 我们发布了一份渐进式采用指南，并与 Expo、Vite 和 Next.js 合作，让新应用可以在启用编译器的情况下直接开始。

---

我们今天发布编译器的第一个稳定版。React Compiler 同时适用于 React 和 React Native，无需重写即可自动优化组件和 hooks。该编译器已在 Meta 的主要应用中经过实战检验，并已完全可用于生产环境。

[React Compiler](/learn/react-compiler) 是一种构建时工具，通过自动记忆化来优化你的 React 应用。去年，我们发布了 React Compiler 的[首个 beta 版](/blog/2024/10/21/react-compiler-beta-release)，并收到了大量很棒的反馈和贡献。我们对采用编译器的用户所取得的成果感到振奋（参见来自 [Sanity Studio](https://github.com/reactwg/react-compiler/discussions/33) 和 [Wakelet](https://github.com/reactwg/react-compiler/discussions/52) 的案例研究），也很期待将编译器带给 React 社区中的更多用户。

这个版本是近十年中一项巨大而复杂的工程努力的结晶。React 团队对编译器的首次探索始于 2017 年的 [Prepack](https://github.com/facebookarchive/prepack)。虽然这个项目最终被关闭了，但其中许多经验影响了团队对 Hooks 的设计；Hooks 在设计时就考虑了未来编译器的需要。2021 年，[Xuan Huang](https://x.com/Huxpro) 演示了 React Compiler 新构想的[第一版原型](https://www.youtube.com/watch?v=lGEMwh32soc)。

尽管这个新 React Compiler 的第一个版本最终被重写了，但最初的原型让我们更加确信这是一个可行的问题，而这些经验也让我们认识到，另一种编译器架构可以精确提供我们想要的记忆化特性。[Joe Savona](https://x.com/en_JS)、[Sathya Gunasekaran](https://x.com/_gsathya)、[Mofei Zhang](https://x.com/zmofei) 和 [Lauren Tan](https://x.com/potetotes) 完成了我们的首次重写，将编译器架构迁移到了基于控制流图（CFG）的高级中间表示（HIR）。这为 React Compiler 中更加精确的分析，甚至类型推断，铺平了道路。从那时起，编译器的许多重要部分都被重写过，每次重写都吸收了我们从前一次尝试中得到的经验。而且一路上，[React 团队](/community/team)的许多成员也给予了我们大量帮助和贡献。

这个稳定版只是我们众多版本中的第一个。编译器将继续演进和改进，我们预计它将成为 React 未来十年乃至更久的新基础和新阶段。

你可以直接跳到[快速开始](/learn/react-compiler)，或者继续阅读 React Conf 2025 的重点内容。

<DeepDive>

#### React Compiler 是如何工作的？ {/*how-does-react-compiler-work*/}

React Compiler 是一个优化型编译器，通过自动记忆化来优化组件和 hooks。虽然它目前是作为 Babel 插件实现的，但编译器在很大程度上与 Babel 解耦，会将 Babel 提供的抽象语法树（AST）转换为其自身新的 HIR，并通过多个编译器阶段，仔细理解你的 React 代码的数据流和可变性。这使得编译器能够细粒度地记忆化渲染中使用的值，包括条件性记忆化的能力，而这是手动记忆化无法实现的。

```js {8}
import { use } from 'react';

export default function ThemeProvider(props) {
  if (!props.children) {
    return null;
  }
  // 编译器在条件返回之后仍然可以对代码进行记忆化
  const theme = mergeTheme(props.theme, use(ThemeContext));
  return (
    <ThemeContext value={theme}>
      {props.children}
    </ThemeContext>
  );
}
```
_在 [React Compiler Playground](https://playground.react.dev/#N4Igzg9grgTgxgUxALhASwLYAcIwC4AEwBUYCBAvgQGYwQYEDkMCAhnHowNwA6AdvwQAPHPgIATBNVZQANoWpQ+HNBD4EAKgAsEGBAAU6ANzSSYACix0sYAJRF+BAmmoFzAQisQbAOjha0WXEWPntgRycCFjxYdT45WV51Sgi4NTBCPB09AgBeAj0YAHMEbV0ES2swHyzygBoSMnMyvQBhNTxhPFtbJKdo2LcIpwAeFoR2vk6hQiNWWSgEXOBavQoAPmHI4C9ff0DghD4KLZGAenHJ6bxN5N7+ChA6kDS+ajQilHRsXEyATyw5GI+gWRTQfAA8lg8Ko+GBKDQ6AxGAAjVgohCyAC0WFB4KxLHYeCxaWwgQQMDO4jQGW4-H45nCyTOZ1JWECrBhagAshBJMgCDwQPNZEKHgQwJyae8EPCQVAwZDobC7FwnuAtBAAO4ASSmFL48zAKGksjIFCAA) 中查看此示例_

除了自动记忆化之外，React Compiler 还对你的 React 代码运行验证阶段。这些阶段会编码 [React 规则](/reference/rules)，并利用编译器对数据流和可变性的理解，在违反 React 规则时提供诊断信息。这些诊断信息通常会暴露隐藏在 React 代码中的潜在 bug，主要通过 `eslint-plugin-react-hooks` 呈现。

要了解更多关于编译器如何优化你的代码，请访问 [Playground](https://playground.react.dev)。

</DeepDive>

## 今天就使用 React Compiler {/*use-react-compiler-today*/}
安装编译器：

npm
<TerminalBlock>
npm install --save-dev --save-exact babel-plugin-react-compiler@latest
</TerminalBlock>

pnpm
<TerminalBlock>
pnpm add --save-dev --save-exact babel-plugin-react-compiler@latest
</TerminalBlock>

yarn
<TerminalBlock>
yarn add --dev --exact babel-plugin-react-compiler@latest
</TerminalBlock>

作为稳定版发布的一部分，我们一直在让 React Compiler 更易于添加到你的项目中，并改进编译器生成记忆化的方式。React Compiler 现在支持将可选链和数组索引作为依赖项。这些改进最终会带来更少的重渲染和更灵敏的 UI，同时让你继续编写符合惯例的声明式代码。

你可以在[我们的文档](/learn/react-compiler)中找到更多关于使用 Compiler 的细节。

## 我们在生产环境中的观察 {/*react-compiler-at-meta*/}
[该编译器已经在 Meta Quest Store 等应用中上线](https://youtu.be/lyEKhv8-3n0?t=3002)。我们看到首次加载和跨页面导航性能提升了高达 12%，而某些交互速度则快了 2.5 倍以上。即使有这些收益，内存使用仍保持中性。虽然具体效果可能因场景而异，但我们建议你在自己的应用中尝试编译器，看看是否能获得类似的性能提升。

## 向后兼容性 {/*backwards-compatibility*/}
如 Beta 公告中所述，React Compiler 兼容 React 17 及以上版本。如果你还没有使用 React 19，只需在编译器配置中指定最低目标版本，并将 `react-compiler-runtime` 作为依赖项添加即可使用 React Compiler。你可以在[这里](/reference/react-compiler/target#targeting-react-17-or-18)找到相关文档。

## 使用编译器驱动的 linting 强化 React 规则 {/*migrating-from-eslint-plugin-react-compiler-to-eslint-plugin-react-hooks*/}
React Compiler 包含一条 ESLint 规则，可帮助识别违反 [React 规则](/reference/rules) 的代码。该 linter 不要求安装编译器，因此升级 `eslint-plugin-react-hooks` 没有风险。我们建议所有人今天就升级。

如果你已经安装了 `eslint-plugin-react-compiler`，现在可以将其移除并使用 `eslint-plugin-react-hooks@latest`。非常感谢 [@michaelfaith](https://bsky.app/profile/michael.faith) 为这一改进所做的贡献！

安装方式：

npm
<TerminalBlock>
npm install --save-dev eslint-plugin-react-hooks@latest
</TerminalBlock>

pnpm
<TerminalBlock>
pnpm add --save-dev eslint-plugin-react-hooks@latest
</TerminalBlock>

yarn
<TerminalBlock>
yarn add --dev eslint-plugin-react-hooks@latest
</TerminalBlock>

```js {6}
// eslint.config.js（扁平配置）
import reactHooks from 'eslint-plugin-react-hooks';
import { defineConfig } from 'eslint/config';

export default defineConfig([
  reactHooks.configs.flat.recommended,
]);
```

```js {3}
// eslintrc.json（传统配置）
{
  "extends": ["plugin:react-hooks/recommended"],
  // ...
}
```

要启用 React Compiler 规则，我们建议使用 `recommended` 预设。你也可以查看 [README](https://github.com/facebook/react/blob/main/packages/eslint-plugin-react-hooks/README.md) 获取更多说明。以下是我们在 React Conf 上展示的一些示例：

- 使用 [`set-state-in-render`](/reference/eslint-plugin-react-hooks/lints/set-state-in-render) 捕捉会导致渲染循环的 `setState` 模式。
- 通过 [`set-state-in-effect`](/reference/eslint-plugin-react-hooks/lints/set-state-in-effect) 标记 effect 中的昂贵工作。
- 使用 [`refs`](/reference/eslint-plugin-react-hooks/lints/refs) 防止在渲染期间进行不安全的 ref 访问。

## 我该如何看待 useMemo、useCallback 和 React.memo？ {/*what-should-i-do-about-usememo-usecallback-and-reactmemo*/}
默认情况下，React Compiler 会基于其分析和启发式方法来对你的代码进行记忆化。在大多数情况下，这种记忆化会与您可能手写的实现一样精确，甚至更精确——如上所述，编译器甚至可以在 `useMemo`/`useCallback` 无法使用的场景下进行记忆化，例如在早返回之后。

不过，在某些情况下，开发者可能需要对记忆化有更多控制。`useMemo` 和 `useCallback` hooks 仍可与 React Compiler 一起使用，作为一种“逃生舱口”，以便控制哪些值被记忆化。一个常见用例是：某个记忆化值被用作 effect 依赖项，以确保即使其依赖项没有发生有意义的变化，effect 也不会反复触发。

对于新代码，我们建议依赖编译器进行记忆化，并在需要时使用 `useMemo`/`useCallback` 来获得精确控制。

对于现有代码，我们建议保留已有的记忆化实现（移除它可能会改变编译结果），或者在移除记忆化之前进行仔细测试。

## 新应用应使用 React Compiler {/*new-apps-should-use-react-compiler*/}
我们已经与 Expo、Vite 和 Next.js 团队合作，将编译器添加到新应用体验中。

[Expo SDK 54](https://docs.expo.dev/guides/react-compiler/) 及以上版本默认启用编译器，因此新应用从一开始就能自动受益于编译器。

<TerminalBlock>
npx create-expo-app@latest
</TerminalBlock>

[Vite](https://vite.dev/guide/) 和 [Next.js](https://nextjs.org/docs/app/api-reference/cli/create-next-app) 用户可以在 `create-vite` 和 `create-next-app` 中选择启用编译器的模板。

<TerminalBlock>
npm create vite@latest
</TerminalBlock>

<br />

<TerminalBlock>
npx create-next-app@latest
</TerminalBlock>

## 逐步采用 React Compiler {/*adopt-react-compiler-incrementally*/}
如果你正在维护一个现有应用，可以按自己的节奏逐步引入编译器。我们发布了一份逐步的[增量采用指南](/learn/react-compiler/incremental-adoption)，其中涵盖了门控策略、兼容性检查和发布工具，因此你可以放心启用编译器。

## swc 支持（实验性） {/*swc-support-experimental*/}
React Compiler 可以安装在[多种构建工具](/learn/react-compiler#installation)中，例如 Babel、Vite 和 Rsbuild。

除了这些工具之外，我们还一直在与 [swc](https://swc.rs/) 团队的 Kang Dongyoon（[@kdy1dev](https://x.com/kdy1dev)）合作，为 React Compiler 作为 swc 插件添加额外支持。虽然这项工作还未完成，但当[在你的 Next.js 应用中启用 React Compiler](https://nextjs.org/docs/app/api-reference/config/next-config-js/reactCompiler)时，Next.js 的构建性能现在应该会明显更快。

我们建议使用 Next.js [15.3.1](https://github.com/vercel/next.js/releases/tag/v15.3.1) 或更高版本，以获得最佳构建性能。

Vite 用户可以继续使用 [vite-plugin-react](https://github.com/vitejs/vite-plugin-react) 来启用编译器，只需将其作为 [Babel 插件](/learn/react-compiler/installation#vite)添加即可。我们也在与 [oxc](https://oxc.rs/) 团队合作，[为编译器添加支持](https://github.com/oxc-project/oxc/issues/10048)。一旦 [rolldown](https://github.com/rolldown/rolldown) 正式发布并在 Vite 中得到支持，并且 oxc 为 React Compiler 添加了支持，我们会更新文档，说明如何迁移。

## 升级 React Compiler {/*upgrading-react-compiler*/}
React Compiler 在应用自动 memoization 时，最好严格将其用于性能优化。编译器的未来版本可能会改变 memoization 的应用方式，例如它可能会变得更细粒度、更精确。

然而，由于产品代码有时会以 JavaScript 中并不总能静态检测到的方式破坏 [React 规则](/reference/rules)，因此更改 memoization 偶尔可能会产生意外结果。例如，某个先前已 memoized 的值可能会在组件树中的某处被用作 `useEffect` 的依赖项。改变该值的 memoization 方式，或是否对其进行 memoization，可能会导致该 `useEffect` 过度触发或触发不足。虽然我们鼓励[仅将 useEffect 用于同步](/learn/synchronizing-with-effects)，但你的代码库中可能存在用于其他场景的 `useEffect`，例如只需要在特定值变化时运行的 effect。

换句话说，改变 memoization 在极少数情况下可能会导致意外行为。因此，我们建议遵循 React 规则，并对你的应用进行持续的端到端测试，这样你就可以放心升级编译器，并识别出任何可能导致问题的 React 规则违规。

如果你没有良好的测试覆盖，我们建议将编译器固定到确切版本（例如 `1.0.0`），而不是 SemVer 范围（例如 `^1.0.0`）。你可以在升级编译器时传入 `--save-exact`（npm/pnpm）或 `--exact`（yarn）标志来实现这一点。之后你应该手动进行编译器的任何升级，并注意检查你的应用是否仍按预期工作。

---

感谢 [Jason Bonta](https://x.com/someextent)、[Jimmy Lai](https://x.com/feedthejim)、[Kang Dongyoon](https://x.com/kdy1dev)（@kdy1dev）以及 [Dan Abramov](https://bsky.app/profile/danabra.mov) 对本文进行审阅和编辑。
