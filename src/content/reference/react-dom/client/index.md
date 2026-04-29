---
title: 客户端 React DOM APIs
---

<Intro>

`react-dom/client` APIs 让你可以在客户端（浏览器中）渲染 React 组件。这些 API 通常用于应用的顶层，以初始化你的 React 树。[框架](/learn/creating-a-react-app#full-stack-frameworks) 可能会替你调用它们。你的大多数组件都不需要导入或使用它们。

</Intro>

---

## 客户端 API {/*client-apis*/}

* [`createRoot`](/reference/react-dom/client/createRoot) 允许你创建一个 root，以在浏览器 DOM 节点中显示 React 组件。
* [`hydrateRoot`](/reference/react-dom/client/hydrateRoot) 允许你在一个浏览器 DOM 节点中显示 React 组件，而该节点的 HTML 内容此前是由 [`react-dom/server`.](/reference/react-dom/server) 生成的。

---

## 浏览器支持 {/*browser-support*/}

React 支持所有主流浏览器，包括 Internet Explorer 9 及以上版本。对于较旧的浏览器，例如 IE 9 和 IE 10，需要一些 polyfill。