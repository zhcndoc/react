---
title: React 开发者工具
---

<Intro>

使用 React 开发者工具来检查 React [组件](/learn/your-first-component)，编辑 [props](/learn/passing-props-to-a-component) 和 [state](/learn/state-a-components-memory)，并识别性能问题。

</Intro>

<YouWillLearn>

* 如何安装 React 开发者工具

</YouWillLearn>

## 浏览器扩展 {/*browser-extension*/}

调试使用 React 构建的网站的最简单方法是安装 React 开发者工具浏览器扩展。它支持几个流行的浏览器：

* [在 **Chrome** 中安装](https://chrome.google.com/webstore/detail/react-developer-tools/fmkadmapgofadopljbjfkapdkoienihi?hl=en)
* [在 **Firefox** 中安装](https://addons.mozilla.org/en-US/firefox/addon/react-devtools/)
* [在 **Edge** 中安装](https://microsoftedge.microsoft.com/addons/detail/react-developer-tools/gpphkfbcpidddadnkolkpfckpihlkkil)

现在，如果你访问一个**使用 React 构建**的网站，你会看到 _Components_ 和 _Profiler_ 面板。

![React 开发者工具扩展](/images/docs/react-devtools-extension.png)

### Safari 和其他浏览器 {/*safari-and-other-browsers*/}
对于其他浏览器（例如 Safari），请安装 [`react-devtools`](https://www.npmjs.com/package/react-devtools) npm 包：
```bash
# Yarn
yarn global add react-devtools

# Npm
npm install -g react-devtools
```

接下来，从终端打开开发者工具：
```bash
react-devtools
```

然后通过在你网站的 `<head>` 开头添加以下 `<script>` 标签来连接你的网站：
```html {3}
<html>
  <head>
    <script src="http://localhost:8097"></script>
```

现在在浏览器中重新加载你的网站，以便在开发者工具中查看它。

![React 开发者工具独立版](/images/docs/react-devtools-standalone.png)

## 移动端（React Native） {/*mobile-react-native*/}

要检查使用 [React Native](https://reactnative.dev/) 构建的应用，你可以使用 [React Native DevTools](https://reactnative.dev/docs/react-native-devtools)，这是一个与 React 开发者工具深度集成的内置调试器。所有功能都与浏览器扩展完全相同，包括原生元素的高亮和选择。

[了解更多关于 React Native 中调试的信息。](https://reactnative.dev/docs/debugging)

> 对于 0.76 之前的 React Native 版本，请按照上面的 [Safari 和其他浏览器](#safari-and-other-browsers) 指南使用 React DevTools 的独立版本。
