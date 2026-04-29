---
title: 编译库
---

<Intro>
本指南帮助库作者了解如何使用 React Compiler 为用户发布经过优化的库代码。
</Intro>

<InlineToc />

## 为什么要发布编译后的代码？ {/*why-ship-compiled-code*/}

作为库作者，你可以在发布到 npm 之前先编译你的库代码。这带来几个好处：

- **为所有用户带来性能提升** - 即使你的库用户还没有使用 React Compiler，他们也能获得优化后的代码
- **无需用户进行配置** - 优化开箱即用
- **行为一致** - 无论构建设置如何，所有用户都会获得相同的优化版本

## 设置编译 {/*setting-up-compilation*/}

将 React Compiler 添加到你的库构建流程中：

<TerminalBlock>
npm install -D babel-plugin-react-compiler@latest
</TerminalBlock>

配置你的构建工具来编译你的库。例如，使用 Babel：

```js
// babel.config.js
module.exports = {
  plugins: [
    'babel-plugin-react-compiler',
  ],
  // ... 其他配置
};
```

## 向后兼容性 {/*backwards-compatibility*/}

如果你的库支持 React 19 以下的版本，你将需要额外的配置：

### 1. 安装运行时包 {/*install-runtime-package*/}

我们建议将 react-compiler-runtime 作为直接依赖安装：

<TerminalBlock>
npm install react-compiler-runtime@latest
</TerminalBlock>

```json
{
  "dependencies": {
    "react-compiler-runtime": "^1.0.0"
  },
  "peerDependencies": {
    "react": "^17.0.0 || ^18.0.0 || ^19.0.0"
  }
}
```

### 2. 配置目标版本 {/*configure-target-version*/}

设置你的库所支持的最低 React 版本：

```js
{
  target: '17', // 最低支持的 React 版本
}
```

## 测试策略 {/*testing-strategy*/}

分别在启用和未启用编译的情况下测试你的库，以确保兼容性。将你现有的测试套件运行在编译后的代码上，同时再创建一个绕过编译器的独立测试配置。这有助于发现编译过程中可能出现的任何问题，并确保你的库在所有场景下都能正常工作。

## 故障排查 {/*troubleshooting*/}

### 库在较旧的 React 版本中无法工作 {/*library-doesnt-work-with-older-react-versions*/}

如果你编译后的库在 React 17 或 18 中抛出错误：

1. 确认你已经将 `react-compiler-runtime` 作为依赖安装
2. 检查你的 `target` 配置是否与你支持的最低 React 版本一致
3. 确保运行时包已包含在你发布的 bundle 中

### 编译与其他 Babel 插件冲突 {/*compilation-conflicts-with-other-babel-plugins*/}

某些 Babel 插件可能与 React Compiler 冲突：

1. 将 `babel-plugin-react-compiler` 放在你的插件列表靠前的位置
2. 在其他插件中禁用有冲突的优化
3. 彻底测试你的构建输出

### 未找到运行时模块 {/*runtime-module-not-found*/}

如果用户看到 “Cannot find module 'react-compiler-runtime'”：

1. 确保运行时包列在 `dependencies` 中，而不是 `devDependencies`
2. 检查你的 bundler 是否将运行时包含在输出中
3. 验证该包是否已与你的库一起发布到 npm

## 后续步骤 {/*next-steps*/}

- 了解用于编译后代码的[调试技巧](/learn/react-compiler/debugging)
- 查看所有编译器选项的[配置选项](/reference/react-compiler/configuration)
- 探索用于选择性优化的[编译模式](/reference/react-compiler/compilationMode)