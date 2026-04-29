---
title: 编辑器设置
---

<Intro>

正确配置的编辑器可以让代码更易读、书写更快。它甚至还能在你写代码时帮你发现 bug！如果这是你第一次设置编辑器，或者你想优化当前的编辑器，我们有一些推荐。

</Intro>

<YouWillLearn>

* 最流行的编辑器有哪些
* 如何自动格式化你的代码

</YouWillLearn>

## 你的编辑器 {/*your-editor*/}

[VS Code](https://code.visualstudio.com/) 是当今最流行的编辑器之一。它拥有庞大的扩展市场，并且能与 GitHub 之类的流行服务很好地集成。下面列出的许多功能也可以作为扩展添加到 VS Code 中，使它具有很强的可配置性！

React 社区中使用的其他流行文本编辑器包括：

* [WebStorm](https://www.jetbrains.com/webstorm/) 是一款专为 JavaScript 设计的集成开发环境。
* [Sublime Text](https://www.sublimetext.com/) 内置支持 JSX 和 TypeScript、[语法高亮](https://stackoverflow.com/a/70960574/458193) 以及自动补全。
* [Vim](https://www.vim.org/) 是一款高度可配置的文本编辑器，旨在高效地创建和修改各种文本。它在大多数 UNIX 系统以及 Apple OS X 中都以 "vi" 的形式包含。

## 推荐的文本编辑器功能 {/*recommended-text-editor-features*/}

有些编辑器内置了这些功能，但其他编辑器可能需要添加扩展。请查看你所选编辑器提供哪些支持，以确保满足需求！

### 代码检查 {/*linting*/}

代码检查工具会在你编写代码时发现问题，帮助你及早修复它们。[ESLint](https://eslint.org/) 是一个流行的、开源的 JavaScript 代码检查工具。

* [使用适用于 React 的推荐配置安装 ESLint](https://www.npmjs.com/package/eslint-config-react-app)（请确保你已经[安装了 Node！](https://nodejs.org/en/download/current/)）
* [通过官方扩展在 VSCode 中集成 ESLint](https://marketplace.visualstudio.com/items?itemName=dbaeumer.vscode-eslint)

**请确保你已经为项目启用了 [`eslint-plugin-react-hooks`](https://www.npmjs.com/package/eslint-plugin-react-hooks) 的所有规则。** 它们至关重要，并且能尽早发现最严重的 bug。推荐的 [`eslint-config-react-app`](https://www.npmjs.com/package/eslint-config-react-app) 预设已经包含了它们。

### 格式化 {/*formatting*/}

当你与其他贡献者共享代码时，最不想做的事就是讨论 [制表符和空格哪个更好](https://www.google.com/search?q=tabs+vs+spaces)！幸运的是，[Prettier](https://prettier.io/) 会通过重新格式化你的代码来清理它，使其符合预设的、可配置的规则。运行 Prettier 后，所有制表符都会被转换为空格——同时你的缩进、引号等也都会被修改为符合配置。在理想设置中，Prettier 会在你保存文件时运行，快速为你完成这些编辑。

你可以按照以下步骤安装 [VSCode 中的 Prettier 扩展](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode)：

1. 启动 VS Code
2. 使用快速打开（按 Ctrl/Cmd+P）
3. 粘贴 `ext install esbenp.prettier-vscode`
4. 按 Enter

#### 保存时格式化 {/*formatting-on-save*/}

理想情况下，你应该在每次保存时都格式化代码。VS Code 有相关设置！

1. 在 VS Code 中，按 `CTRL/CMD + SHIFT + P`。
2. 输入 "settings"
3. 按 Enter
4. 在搜索栏中输入 "format on save"
5. 确保 "format on save" 选项已勾选！

> 如果你的 ESLint 预设包含格式化规则，它们可能会与 Prettier 冲突。我们建议使用 [`eslint-config-prettier`](https://github.com/prettier/eslint-config-prettier) 禁用 ESLint 预设中的所有格式化规则，这样 ESLint 就*只*用于发现逻辑错误。如果你希望在 pull request 合并之前强制文件已完成格式化，请在持续集成中使用 [`prettier --check`](https://prettier.io/docs/en/cli.html#--check)。
