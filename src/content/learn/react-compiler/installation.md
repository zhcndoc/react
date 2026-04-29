---
title: 安装
---

<Intro>
本指南将帮助你在 React 应用中安装和配置 React Compiler。
</Intro>

<YouWillLearn>

* 如何安装 React Compiler
* 不同构建工具的基础配置
* 如何验证你的设置是否正常工作

</YouWillLearn>

## 前提条件 {/*prerequisites*/}

React Compiler 的设计目标是与 React 19 配合得最好，但它也支持 React 17 和 18。了解更多关于[React 版本兼容性](/reference/react-compiler/target)。

## 安装 {/*installation*/}

将 React Compiler 作为 `devDependency` 安装：

<TerminalBlock>
npm install -D babel-plugin-react-compiler@latest
</TerminalBlock>

或者使用 Yarn：

<TerminalBlock>
yarn add -D babel-plugin-react-compiler@latest
</TerminalBlock>

或者使用 pnpm：

<TerminalBlock>
pnpm install -D babel-plugin-react-compiler@latest
</TerminalBlock>

## 基本设置 {/*basic-setup*/}

React Compiler 的设计目标是默认即可工作，无需任何配置。不过，如果你需要在特殊情况下对其进行配置（例如，面向低于 19 的 React 版本），请参阅[编译器选项参考](/reference/react-compiler/configuration)。

设置过程取决于你的构建工具。React Compiler 包含一个可集成到构建流水线中的 Babel 插件。

<Pitfall>
React Compiler 必须在你的 Babel 插件流水线中**最先**运行。编译器需要原始源代码信息进行正确分析，因此它必须先于其他转换处理你的代码。
</Pitfall>

### Babel {/*babel*/}

创建或更新你的 `babel.config.js`：

```js {3}
module.exports = {
  plugins: [
    'babel-plugin-react-compiler', // 必须最先运行！
    // ... 其他插件
  ],
  // ... 其他配置
};
```

### Vite {/*vite*/}

如果你使用 Vite，可以将插件添加到 vite-plugin-react 中：

```js {3,9}
// vite.config.js
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [
    react({
      babel: {
        plugins: ['babel-plugin-react-compiler'],
      },
    }),
  ],
});
```

或者，如果你更喜欢为 Vite 单独使用 Babel 插件：

<TerminalBlock>
npm install -D vite-plugin-babel
</TerminalBlock>

```js {2,11}
// vite.config.js
import babel from 'vite-plugin-babel';
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [
    react(),
    babel({
      babelConfig: {
        plugins: ['babel-plugin-react-compiler'],
      },
    }),
  ],
});
```

### Next.js {/*usage-with-nextjs*/}

请参阅 [Next.js 文档](https://nextjs.org/docs/app/api-reference/next-config-js/reactCompiler) 了解更多信息。

### React Router {/*usage-with-react-router*/}
安装 `vite-plugin-babel`，并将编译器的 Babel 插件添加到其中：

<TerminalBlock>
npm install vite-plugin-babel
</TerminalBlock>

```js {3-4,16}
// vite.config.js
import { defineConfig } from "vite";
import babel from "vite-plugin-babel";
import { reactRouter } from "@react-router/dev/vite";

const ReactCompilerConfig = { /* ... */ };

export default defineConfig({
  plugins: [
    reactRouter(),
    babel({
      filter: /\.[jt]sx?$/,
      babelConfig: {
        presets: ["@babel/preset-typescript"], // 如果你使用 TypeScript
        plugins: [
          ["babel-plugin-react-compiler", ReactCompilerConfig],
        ],
      },
    }),
  ],
});
```

### Webpack {/*usage-with-webpack*/}

社区维护的 webpack loader [现已可用](https://github.com/SukkaW/react-compiler-webpack)。

### Expo {/*usage-with-expo*/}

请参阅 [Expo 文档](https://docs.expo.dev/guides/react-compiler/) 以在 Expo 应用中启用并使用 React Compiler。

### Metro (React Native) {/*usage-with-react-native-metro*/}

React Native 通过 Metro 使用 Babel，因此请参阅[使用 Babel](#babel) 部分了解安装说明。

### Rspack {/*usage-with-rspack*/}

请参阅 [Rspack 文档](https://rspack.dev/guide/tech/react#react-compiler) 以在 Rspack 应用中启用并使用 React Compiler。

### Rsbuild {/*usage-with-rsbuild*/}

请参阅 [Rsbuild 文档](https://rsbuild.dev/guide/framework/react#react-compiler) 以在 Rsbuild 应用中启用并使用 React Compiler。


## ESLint 集成 {/*eslint-integration*/}

React Compiler 包含一条 ESLint 规则，可帮助识别无法优化的代码。当 ESLint 规则报告错误时，这意味着编译器会跳过对该特定组件或 hook 的优化。这是安全的：编译器会继续优化代码库的其他部分。你不需要立即修复所有违规项。可以按自己的节奏逐步处理它们，以逐渐增加可优化组件的数量。

安装 ESLint 插件：

<TerminalBlock>
npm install -D eslint-plugin-react-hooks@latest
</TerminalBlock>

如果你还没有配置 eslint-plugin-react-hooks，请按照 [readme 中的安装说明](https://github.com/facebook/react/blob/main/packages/eslint-plugin-react-hooks/README.md#installation) 进行设置。编译器规则可在 `recommended-latest` 预设中使用。

ESLint 规则将会：
- 识别[React 规则](/reference/rules)的违规项
- 显示哪些组件无法被优化
- 提供有助于修复问题的错误信息

## 验证你的设置 {/*verify-your-setup*/}

安装完成后，请验证 React Compiler 是否正常工作。

### 检查 React DevTools {/*check-react-devtools*/}

被 React Compiler 优化的组件会在 React DevTools 中显示一个 “Memo ✨” 徽标：

1. 安装 [React Developer Tools](/learn/react-developer-tools) 浏览器扩展
2. 以开发模式打开你的应用
3. 打开 React DevTools
4. 查看组件名称旁边是否有 ✨ 表情符号

如果编译器正常工作：
- 组件会在 React DevTools 中显示 “Memo ✨” 徽标
- 昂贵的计算会自动进行 memoization
- 不需要手动使用 `useMemo`

### 检查构建输出 {/*check-build-output*/}

你也可以通过检查构建输出来验证编译器是否正在运行。编译后的代码将包含编译器自动添加的自动 memoization 逻辑。

```js
import { c as _c } from "react/compiler-runtime";
export default function MyApp() {
  const $ = _c(1);
  let t0;
  if ($[0] === Symbol.for("react.memo_cache_sentinel")) {
    t0 = <div>Hello World</div>;
    $[0] = t0;
  } else {
    t0 = $[0];
  }
  return t0;
}

```

## 故障排除 {/*troubleshooting*/}

### 对特定组件选择退出 {/*opting-out-specific-components*/}

如果某个组件在编译后出现问题，你可以使用 `"use no memo"` 指令临时让它退出优化：

```js
function ProblematicComponent() {
  "use no memo";
  // 组件代码写在这里
}
```

这会告诉编译器跳过对该特定组件的优化。你应该修复底层问题，并在解决后移除该指令。

如需更多故障排除帮助，请参阅[调试指南](/learn/react-compiler/debugging)。

## 后续步骤 {/*next-steps*/}

现在你已经安装了 React Compiler，了解更多关于：

- [React 版本兼容性](/reference/react-compiler/target) 适用于 React 17 和 18
- [配置选项](/reference/react-compiler/configuration) 用于自定义编译器
- [渐进式采用策略](/learn/react-compiler/incremental-adoption) 适用于现有代码库
- [调试技巧](/learn/react-compiler/debugging) 用于排查问题
- [编译库指南](/reference/react-compiler/compiling-libraries) 用于编译你的 React 库
