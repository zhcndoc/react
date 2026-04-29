---
title: 安装
---

<Intro>

React 从一开始就被设计为可逐步采用。你可以按需使用任意少量或大量的 React。无论你是想先体验一下 React，给 HTML 页面添加一些交互性，还是开始构建一个复杂的 React 驱动应用，本节都将帮助你入门。

</Intro>

## 试用 React {/*try-react*/}

你不需要安装任何东西就可以体验 React。试试编辑这个沙盒！

<Sandpack>

```js
function Greeting({ name }) {
  return <h1>Hello, {name}</h1>;
}

export default function App() {
  return <Greeting name="world" />
}
```

</Sandpack>

你可以直接编辑它，或者点击右上角的“Fork”按钮，在新标签页中打开它。

React 文档中的大多数页面都包含这样的沙盒。在 React 文档之外，也有许多支持 React 的在线沙盒：例如 [CodeSandbox](https://codesandbox.io/s/new)、[StackBlitz](https://stackblitz.com/fork/react) 或 [CodePen.](https://codepen.io/pen?template=QWYVwWN)

要在你的电脑上本地体验 React，请[下载这个 HTML 页面。](https://gist.githubusercontent.com/gaearon/0275b1e1518599bbeafcde4722e79ed1/raw/db72dcbf3384ee1708c4a07d3be79860db04bff0/example.html) 在你的编辑器和浏览器中打开它！

## 创建一个 React 应用 {/*creating-a-react-app*/}

如果你想开始一个新的 React 应用，你可以使用推荐的框架[创建一个 React 应用](/learn/creating-a-react-app)。

## 从零开始构建一个 React 应用 {/*build-a-react-app-from-scratch*/}

如果框架不适合你的项目，或者你更倾向于构建自己的框架，或者你只是想学习 React 应用的基础知识，你可以[从零开始构建一个 React 应用](/learn/build-a-react-app-from-scratch)。

## 将 React 添加到现有项目中 {/*add-react-to-an-existing-project*/}

如果你想在现有应用或网站中尝试使用 React，你可以[将 React 添加到现有项目中。](/learn/add-react-to-an-existing-project)


<Note>

#### 我应该使用 Create React App 吗？ {/*should-i-use-create-react-app*/}

不。Create React App 已经被弃用。有关更多信息，请参阅 [Sunsetting Create React App](/blog/2025/02/14/sunsetting-create-react-app)。

</Note>

## 下一步 {/*next-steps*/}

前往 [快速开始](/learn) 指南，了解你每天都会遇到的最重要的 React 概念。

