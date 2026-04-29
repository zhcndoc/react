---
title: 你的第一个组件
---

<Intro>

*组件* 是 React 的核心概念之一。它们是你构建用户界面（UI）的基础，因此是开始 React 之旅的最佳起点！

</Intro>

<YouWillLearn>

* 什么是组件
* 组件在 React 应用中扮演什么角色
* 如何编写你的第一个 React 组件

</YouWillLearn>

## 组件：UI 构建块 {/*components-ui-building-blocks*/}

在 Web 上，HTML 让我们可以使用内置的一组标签（如 `<h1>` 和 `<li>`）来创建丰富且结构化的文档：

```html
<article>
  <h1>我的第一个组件</h1>
  <ol>
    <li>组件：UI 构建块</li>
    <li>定义一个组件</li>
    <li>使用一个组件</li>
  </ol>
</article>
```

这段标记表示这篇文章 `<article>`、它的标题 `<h1>`，以及一个（简略的）目录，目录以有序列表 `<ol>` 的形式呈现。像这样的标记，结合用于样式的 CSS 和用于交互性的 JavaScript，构成了你在 Web 上看到的每个侧边栏、头像、模态框、下拉菜单——每一个 UI 组件的背后。

React 让你可以将标记、CSS 和 JavaScript 组合成自定义的“组件”，**即应用中可复用的 UI 元素。** 你上面看到的目录代码可以变成一个 `<TableOfContents />` 组件，你可以在每个页面上渲染它。在底层，它仍然使用相同的 HTML 标签，例如 `<article>`、`<h1>` 等。

就像 HTML 标签一样，你可以组合、排列和嵌套组件来设计整页内容。例如，你正在阅读的文档页面就是由 React 组件构成的：

```js
<PageLayout>
  <NavigationHeader>
    <SearchBar />
    <Link to="/docs">文档</Link>
  </NavigationHeader>
  <Sidebar />
  <PageContent>
    <TableOfContents />
    <DocumentationText />
  </PageContent>
</PageLayout>
```

随着项目的发展，你会发现很多设计都可以通过复用你已经写好的组件来组合完成，从而加快开发速度。上面的目录可以通过 `<TableOfContents />` 添加到任何界面中！你甚至可以借助 React 开源社区共享的成千上万个组件来快速启动项目，例如 [Chakra UI](https://chakra-ui.com/) 和 [Material UI.](https://material-ui.com/)

## 定义一个组件 {/*defining-a-component*/}

传统上，在创建网页时，网页开发者先标记内容，然后通过添加一些 JavaScript 来实现交互。这在“交互只是锦上添花”的 Web 场景中效果很好。如今，许多网站和所有应用都需要交互。React 将交互放在首位，同时仍然使用相同的技术：**React 组件就是一个你可以用标记“点缀”的 JavaScript 函数。** 下面是它的样子（你可以编辑下面的示例）：

<Sandpack>

```js
export default function Profile() {
  return (
    <img
      src="https://react.dev/images/docs/scientists/MK3eW3Am.jpg"
      alt="Katherine Johnson"
    />
  )
}
```

```css
img { height: 200px; }
```

</Sandpack>

下面是构建组件的方法：

### 第 1 步：导出组件 {/*step-1-export-the-component*/}

`export default` 前缀是 [标准的 JavaScript 语法](https://developer.mozilla.org/docs/web/javascript/reference/statements/export)（不是 React 特有的）。它可以让你将文件中的主函数标记出来，以便之后从其他文件中导入它。（关于导入的更多内容，请参见 [导入和导出组件](/learn/importing-and-exporting-components)！）

### 第 2 步：定义函数 {/*step-2-define-the-function*/}

使用 `function Profile() { }` 你就定义了一个名为 `Profile` 的 JavaScript 函数。

<Pitfall>

React 组件是普通的 JavaScript 函数，但**它们的名字必须以大写字母开头**，否则它们将无法工作！

</Pitfall>

### 第 3 步：添加标记 {/*step-3-add-markup*/}

这个组件返回一个带有 `src` 和 `alt` 属性的 `<img />` 标签。`<img />` 写起来像 HTML，但它实际上在底层是 JavaScript！这种语法称为 [JSX](/learn/writing-markup-with-jsx)，它允许你在 JavaScript 中嵌入标记。

return 语句可以写在同一行上，就像这个组件一样：

```js
return <img src="https://react.dev/images/docs/scientists/MK3eW3As.jpg" alt="Katherine Johnson" />;
```

但如果你的标记与 `return` 关键字不在同一行，你必须用一对括号将其包裹起来：

```js
return (
  <div>
    <img src="https://react.dev/images/docs/scientists/MK3eW3As.jpg" alt="Katherine Johnson" />
  </div>
);
```

<Pitfall>

如果没有括号，`return` 之后各行上的任何代码 [都会被忽略](https://stackoverflow.com/questions/2846283/what-are-the-rules-for-javascripts-automatic-semicolon-insertion-asi)！

</Pitfall>

## 使用一个组件 {/*using-a-component*/}

现在你已经定义了 `Profile` 组件，就可以将它嵌套到其他组件中。例如，你可以导出一个使用多个 `Profile` 组件的 `Gallery` 组件：

<Sandpack>

```js
function Profile() {
  return (
    <img
      src="https://react.dev/images/docs/scientists/MK3eW3As.jpg"
      alt="Katherine Johnson"
    />
  );
}

export default function Gallery() {
  return (
    <section>
      <h1>令人惊叹的科学家</h1>
      <Profile />
      <Profile />
      <Profile />
    </section>
  );
}
```

```css
img { margin: 0 10px 10px 0; height: 90px; }
```

</Sandpack>

### 浏览器看到的内容 {/*what-the-browser-sees*/}

注意大小写的区别：

* `<section>` 是小写，所以 React 知道我们指的是一个 HTML 标签。
* `<Profile />` 以大写 `P` 开头，所以 React 知道我们要使用名为 `Profile` 的组件。

而 `Profile` 内部又包含了更多 HTML：`<img />`。最终，浏览器看到的是这样：

```html
<section>
  <h1>令人惊叹的科学家</h1>
  <img src="https://react.dev/images/docs/scientists/MK3eW3As.jpg" alt="Katherine Johnson" />
  <img src="https://react.dev/images/docs/scientists/MK3eW3As.jpg" alt="Katherine Johnson" />
  <img src="https://react.dev/images/docs/scientists/MK3eW3As.jpg" alt="Katherine Johnson" />
</section>
```

### 嵌套和组织组件 {/*nesting-and-organizing-components*/}

组件是普通的 JavaScript 函数，因此你可以将多个组件放在同一个文件中。当组件相对较小，或者彼此紧密相关时，这会很方便。如果这个文件变得拥挤，你随时可以把 `Profile` 移到单独的文件中。你很快就会在 [关于导入的页面](/learn/importing-and-exporting-components) 上学到如何这样做。

因为 `Profile` 组件是渲染在 `Gallery` 内部的——甚至多次渲染！——所以我们可以说 `Gallery` 是一个**父组件**，它把每个 `Profile` 渲染为一个“子组件”。这就是 React 的魅力之一：你可以只定义一次组件，然后在任意多的地方、任意多次使用它。

<Pitfall>

组件可以渲染其他组件，但**你绝不能嵌套它们的定义：**

```js {2-5}
export default function Gallery() {
  // 🔴 绝不要在另一个组件内部定义组件！
  function Profile() {
    // ...
  }
  // ...
}
```

上面的代码片段 [非常慢并且会导致 bug。](/learn/preserving-and-resetting-state#different-components-at-the-same-position-reset-state) 相反，应当把每个组件都定义在顶层：

```js {5-8}
export default function Gallery() {
  // ...
}

// ✅ 在顶层声明组件
function Profile() {
  // ...
}
```

当子组件需要来自父组件的一些数据时，应该[通过 props 传递](/learn/passing-props-to-a-component)，而不是嵌套定义。

</Pitfall>

<DeepDive>

#### 组件层层嵌套 {/*components-all-the-way-down*/}

你的 React 应用从一个“根”组件开始。通常，在你启动一个新项目时，它会自动创建。例如，如果你使用 [CodeSandbox](https://codesandbox.io/)，或者使用 [Next.js](https://nextjs.org/) 框架，那么根组件会定义在 `pages/index.js` 中。在这些示例中，你一直在导出根组件。

大多数 React 应用都使用“组件层层嵌套”的方式。这意味着你不仅会将组件用于按钮这类可复用的部分，还会用于更大的部分，比如侧边栏、列表，以及最终完整的页面！即使某些组件只使用一次，组件也是组织 UI 代码和标记的一种便捷方式。

[基于 React 的框架](/learn/creating-a-react-app) 更进一步。它们不会使用空白 HTML 文件并让 React 用 JavaScript 来“接管”页面管理，而是还会自动根据你的 React 组件生成 HTML。这样可以让你的应用在 JavaScript 代码加载之前就显示一些内容。

不过，许多网站仍然只使用 React 来[为现有 HTML 页面添加交互性。](/learn/add-react-to-an-existing-project#using-react-for-a-part-of-your-existing-page) 它们有许多根组件，而不是整个页面只有一个根组件。你可以按需使用尽可能多——或者尽可能少——的 React。

</DeepDive>

<Recap>

你刚刚初步了解了 React！让我们回顾一下几个关键点。

* React 让你创建组件，**即应用中可复用的 UI 元素。**
* 在 React 应用中，UI 的每一部分都是一个组件。
* React 组件是普通的 JavaScript 函数，但有两个例外：

  1. 它们的名称总是以大写字母开头。
  2. 它们返回 JSX 标记。

</Recap>



<Challenges>

#### 导出组件 {/*export-the-component*/}

这个沙盒无法工作，因为根组件没有被导出：

<Sandpack>

```js
function Profile() {
  return (
    <img
      src="https://react.dev/images/docs/scientists/lICfvbD.jpg"
      alt="Aklilu Lemma"
    />
  );
}
```

```css
img { height: 181px; }
```

</Sandpack>

先尝试自己修复它，再看答案！

<Solution>

像这样在函数定义前加上 `export default`：

<Sandpack>

```js
export default function Profile() {
  return (
    <img
      src="https://react.dev/images/docs/scientists/lICfvbD.jpg"
      alt="Aklilu Lemma"
    />
  );
}
```

```css
img { height: 181px; }
```

</Sandpack>

你可能会想，为什么只写 `export` 还不足以修复这个示例。你可以在 [导入和导出组件。](/learn/importing-and-exporting-components) 中学习 `export` 和 `export default` 的区别

</Solution>

#### 修复 return 语句 {/*fix-the-return-statement*/}

这个 `return` 语句有些不对。你能修复它吗？

<Hint>

在尝试修复时，你可能会得到一个“Unexpected token”错误。在这种情况下，请检查分号是否出现在*右括号之后*。如果把分号留在 `return ( )` 内部，会导致错误。

</Hint>


<Sandpack>

```js
export default function Profile() {
  return
    <img src="https://react.dev/images/docs/scientists/jA8hHMpm.jpg" alt="Katsuko Saruhashi" />;
}
```

```css
img { height: 180px; }
```

</Sandpack>

<Solution>

你可以通过把 return 语句移到同一行来修复这个组件，如下所示：

<Sandpack>

```js
export default function Profile() {
  return <img src="https://react.dev/images/docs/scientists/jA8hHMpm.jpg" alt="Katsuko Saruhashi" />;
}
```

```css
img { height: 180px; }
```

</Sandpack>

或者在 `return` 后立即打开括号，把返回的 JSX 标记包裹起来：

<Sandpack>

```js
export default function Profile() {
  return (
    <img
      src="https://react.dev/images/docs/scientists/jA8hHMpm.jpg"
      alt="Katsuko Saruhashi"
    />
  );
}
```

```css
img { height: 180px; }
```

</Sandpack>

</Solution>

#### 找出错误 {/*spot-the-mistake*/}

`Profile` 组件的声明和使用方式有问题。你能找出错误吗？（试着回忆 React 是如何将组件与普通 HTML 标签区分开的！）

<Sandpack>

```js
function profile() {
  return (
    <img
      src="https://react.dev/images/docs/scientists/QIrZWGIs.jpg"
      alt="Alan L. Hart"
    />
  );
}

export default function Gallery() {
  return (
    <section>
      <h1>令人惊叹的科学家</h1>
      <profile />
      <profile />
      <profile />
    </section>
  );
}
```

```css
img { margin: 0 10px 10px 0; height: 90px; }
```

</Sandpack>

<Solution>

React 组件名称必须以大写字母开头。

将 `function profile()` 改为 `function Profile()`，然后把所有 `<profile />` 改为 `<Profile />`：

<Sandpack>

```js
function Profile() {
  return (
    <img
      src="https://react.dev/images/docs/scientists/QIrZWGIs.jpg"
      alt="Alan L. Hart"
    />
  );
}

export default function Gallery() {
  return (
    <section>
      <h1>令人惊叹的科学家</h1>
      <Profile />
      <Profile />
      <Profile />
    </section>
  );
}
```

```css
img { margin: 0 10px 10px 0; }
```

</Sandpack>

</Solution>

#### 你自己的组件 {/*your-own-component*/}

从头编写一个组件。你可以给它任意有效的名字并返回任意标记。如果你没什么灵感，可以写一个显示 `<h1>干得好！</h1>` 的 `Congratulations` 组件。别忘了导出它！

<Sandpack>

```js
// 在下面编写你的组件！

```

</Sandpack>

<Solution>

<Sandpack>

```js
export default function Congratulations() {
  return (
    <h1>干得好！</h1>
  );
}
```

</Sandpack>

</Solution>

</Challenges>
