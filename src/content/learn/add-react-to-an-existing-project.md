---
title: 将 React 添加到现有项目
---

<Intro>

如果你想为现有项目添加一些交互性，你不必用 React 重写它。将 React 添加到你现有的技术栈中，并在任何地方渲染交互式 React 组件。

</Intro>

<Note>

**你需要安装 [Node.js](https://nodejs.org/en/) 才能进行本地开发。** 虽然你可以在线 [试用 React](/learn/installation#try-react)，或者用一个简单的 HTML 页面来试用，但实际上你在开发中想使用的大多数 JavaScript 工具链都需要 Node.js。

</Note>

## 将 React 用于现有网站的整个子路由 {/*using-react-for-an-entire-subroute-of-your-existing-website*/}

假设你在 `example.com` 上有一个使用其他服务器技术（比如 Rails）构建的现有 Web 应用，并且你想用 React 完整实现所有以 `example.com/some-app/` 开头的路由。

我们建议这样设置：

1. **使用其中一个 [基于 React 的框架](/learn/creating-a-react-app) 构建应用的 React 部分**。
2. **在框架配置中将 `/some-app` 指定为 *基础路径***（方法如下：[Next.js](https://nextjs.org/docs/app/api-reference/config/next-config-js/basePath)、[Gatsby](https://www.gatsbyjs.com/docs/how-to/previews-deploys-hosting/path-prefix/)）。
3. **配置你的服务器或代理**，使 `/some-app/` 下的所有请求都由你的 React 应用处理。

这可以确保你应用中的 React 部分能够[受益于这些框架中内置的最佳实践](/learn/build-a-react-app-from-scratch#consider-using-a-framework)。

许多基于 React 的框架是全栈的，允许你的 React 应用利用服务器。不过，即使你不能或不想在服务器上运行 JavaScript，你也可以采用同样的方法。此时，将 HTML/CSS/JS 导出内容（Next.js 为 [`next export` 输出](https://nextjs.org/docs/advanced-features/static-html-export)，Gatsby 默认为此方式）改为托管在 `/some-app/` 即可。

## 将 React 用于现有页面的一部分 {/*using-react-for-a-part-of-your-existing-page*/}

假设你有一个用其他技术构建的现有页面（无论是像 Rails 这样的服务端技术，还是像 Backbone 这样的客户端技术），并且你想在该页面的某个位置渲染交互式 React 组件。这是集成 React 的常见方式——实际上，这也是 Meta 多年来大多数 React 使用方式的样子！

你可以分两步完成：

1. **搭建一个 JavaScript 环境**，让你可以使用 [JSX 语法](/learn/writing-markup-with-jsx)，使用 [`import`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import) / [`export`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/export) 语法将代码拆分为模块，并使用来自 [npm](https://www.npmjs.com/) 包注册表的包（例如 React）。
2. **渲染你的 React 组件** 到你希望它们出现在页面中的位置。

具体做法取决于你现有的页面设置，因此让我们逐步了解一些细节。

### 第 1 步：搭建模块化 JavaScript 环境 {/*step-1-set-up-a-modular-javascript-environment*/}

模块化 JavaScript 环境让你可以把 React 组件写在单独的文件中，而不是把所有代码都写在一个文件里。它还让你可以使用其他开发者在 [npm](https://www.npmjs.com/) 注册表上发布的各种优秀包——包括 React 本身！具体做法取决于你现有的设置：

* **如果你的应用已经拆分为使用 `import` 语句的多个文件，** 尝试直接使用你已有的设置。检查在你的 JS 代码中写 `<div />` 是否会导致语法错误。如果会导致语法错误，你可能需要使用 [Babel 转换你的 JavaScript 代码](https://babeljs.io/setup)，并启用 [Babel React 预设](https://babeljs.io/docs/babel-preset-react) 来使用 JSX。

* **如果你的应用还没有用于编译 JavaScript 模块的现有设置，** 请使用 [Vite](https://vite.dev/) 来搭建。Vite 社区维护着[许多与后端框架的集成](https://github.com/vitejs/awesome-vite#integrations-with-backends)，包括 Rails、Django 和 Laravel。如果你的后端框架不在列表中，请[按照此指南](https://vite.dev/guide/backend-integration.html)手动将 Vite 构建集成到你的后端中。

要检查你的设置是否可用，请在项目文件夹中运行此命令：

<TerminalBlock>
npm install react react-dom
</TerminalBlock>

然后将以下代码行添加到主 JavaScript 文件顶部（它可能名为 `index.js` 或 `main.js`）：

<Sandpack>

```html public/index.html hidden
<!DOCTYPE html>
<html>
  <head><title>我的应用</title></head>
  <body>
    <!-- 你的现有页面内容（在此示例中，它会被替换） -->
    <div id="root"></div>
  </body>
</html>
```

```js src/index.js active
import { createRoot } from 'react-dom/client';

// 清除现有的 HTML 内容
document.body.innerHTML = '<div id="app"></div>';

// 改为渲染你的 React 组件
const root = createRoot(document.getElementById('app'));
root.render(<h1>Hello, world</h1>);
```

</Sandpack>

如果你的整个页面内容被替换成了 “Hello, world!” ，那就说明一切正常！继续往下看。

<Note>

第一次将模块化 JavaScript 环境集成到现有项目中可能会让人觉得有些棘手，但这是值得的！如果你遇到困难，试试我们的[社区资源](/community)或 [Vite Chat](https://chat.vite.dev/)。

</Note>

### 第 2 步：在页面任意位置渲染 React 组件 {/*step-2-render-react-components-anywhere-on-the-page*/}

在上一步中，你把这段代码放在了主文件顶部：

```js
import { createRoot } from 'react-dom/client';

// 清除现有的 HTML 内容
document.body.innerHTML = '<div id="app"></div>';

// 改为渲染你的 React 组件
const root = createRoot(document.getElementById('app'));
root.render(<h1>Hello, world</h1>);
```

当然，你实际上并不想清除现有的 HTML 内容！

删除这段代码。

相反，你可能希望把 React 组件渲染到 HTML 中的特定位置。打开你的 HTML 页面（或生成它的服务器模板），并为任意标签添加一个唯一的 [`id`](https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/id) 属性，例如：

```html
<!-- ... 你的 html 中的某处 ... -->
<nav id="navigation"></nav>
<!-- ... 更多 html ... -->
```

这样你就可以使用 [`document.getElementById`](https://developer.mozilla.org/en-US/docs/Web/API/Document/getElementById) 找到该 HTML 元素，并将它传给 [`createRoot`](/reference/react-dom/client/createRoot)，从而在其中渲染你自己的 React 组件：

<Sandpack>

```html public/index.html
<!DOCTYPE html>
<html>
  <head><title>我的应用</title></head>
  <body>
    <p>这个段落是 HTML 的一部分。</p>
    <nav id="navigation"></nav>
    <p>这个段落也是 HTML 的一部分。</p>
  </body>
</html>
```

```js src/index.js active
import { createRoot } from 'react-dom/client';

function NavigationBar() {
  // TODO：实际实现一个导航栏
  return <h1>Hello from React!</h1>;
}

const domNode = document.getElementById('navigation');
const root = createRoot(domNode);
root.render(<NavigationBar />);
```

</Sandpack>

请注意，`index.html` 中原有的 HTML 内容得以保留，而你自己的 `NavigationBar` React 组件现在出现在 HTML 中的 `<nav id="navigation">` 内。阅读 [`createRoot` 使用文档](/reference/react-dom/client/createRoot#rendering-a-page-partially-built-with-react)，了解更多关于在现有 HTML 页面中渲染 React 组件的信息。

当你在现有项目中引入 React 时，通常会先从较小的交互式组件（比如按钮）开始，然后逐步不断“向上迁移”，直到最终整个页面都由 React 构建。如果你真的达到了那一步，我们建议你随后立即迁移到[一个 React 框架](/learn/creating-a-react-app)，以充分发挥 React 的优势。

## 在现有原生移动应用中使用 React Native {/*using-react-native-in-an-existing-native-mobile-app*/}

[React Native](https://reactnative.dev/) 也可以逐步集成到现有原生应用中。如果你已经有一个 Android（Java 或 Kotlin）或 iOS（Objective-C 或 Swift）的原生应用，请[按照此指南](https://reactnative.dev/docs/integration-with-existing-apps)为其添加一个 React Native 界面。
