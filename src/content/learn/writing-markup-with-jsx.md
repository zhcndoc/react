---
title: 使用 JSX 编写标记
---

<Intro>

*JSX* 是 JavaScript 的一种语法扩展，它让你可以在 JavaScript 文件中编写类似 HTML 的标记。虽然编写组件还有其他方式，但大多数 React 开发者更喜欢 JSX 的简洁性，而且大多数代码库都在使用它。

</Intro>

<YouWillLearn>

* 为什么 React 会把标记和渲染逻辑混合在一起
* JSX 与 HTML 有什么不同
* 如何使用 JSX 显示信息

</YouWillLearn>

## JSX：将标记放入 JavaScript 中 {/*jsx-putting-markup-into-javascript*/}

Web 是建立在 HTML、CSS 和 JavaScript 之上的。多年来，网页开发者一直把内容放在 HTML 中，把设计放在 CSS 中，把逻辑放在 JavaScript 中——通常还分散在不同的文件里！内容在 HTML 中被标记出来，而页面逻辑则单独存在于 JavaScript 中：

<DiagramGroup>

<Diagram name="writing_jsx_html" height={237} width={325} alt="带有紫色背景的 HTML 标记，其中一个 div 下有两个子标签：p 和 form。 ">

HTML

</Diagram>

<Diagram name="writing_jsx_js" height={237} width={325} alt="带有黄色背景的三个 JavaScript 处理函数：onSubmit、onLogin 和 onClick。">

JavaScript

</Diagram>

</DiagramGroup>

但随着 Web 变得越来越交互化，逻辑越来越多地决定了内容。JavaScript 开始负责 HTML！这就是为什么 **在 React 中，渲染逻辑和标记会一起存在于同一个地方——组件中。**

<DiagramGroup>

<Diagram name="writing_jsx_sidebar" height={330} width={325} alt="将前面示例中的 HTML 和 JavaScript 混合在一起的 React 组件。函数名是 Sidebar，它调用了函数 isLoggedIn，并以黄色高亮。函数中以紫色高亮嵌套着之前的 p 标签，以及一个引用下一张图所示组件的 Form 标签。">

`Sidebar.js` React 组件

</Diagram>

<Diagram name="writing_jsx_form" height={330} width={325} alt="将前面示例中的 HTML 和 JavaScript 混合在一起的 React 组件。函数名是 Form，包含两个以黄色高亮的处理函数 onClick 和 onSubmit。处理函数后面是以紫色高亮的 HTML。HTML 包含一个 form 元素，其中嵌套了一个 input 元素，每个元素都有一个 onClick prop。">

`Form.js` React 组件

</Diagram>

</DiagramGroup>

将按钮的渲染逻辑和标记放在一起，可以确保它们在每次编辑时都保持同步。相反，彼此无关的细节，例如按钮的标记和侧边栏的标记，会彼此隔离，从而让单独修改它们中的任意一个都更安全。

每个 React 组件都是一个 JavaScript 函数，可能包含一些 React 会渲染到浏览器中的标记。React 组件使用一种名为 JSX 的语法扩展来表示这些标记。JSX 看起来很像 HTML，但它更严格一些，并且可以显示动态信息。理解这一点的最好方法，是把一些 HTML 标记转换成 JSX 标记。

<Note>

JSX 和 React 是两个不同的东西。它们经常一起使用，但你也**可以** [把它们独立使用](https://reactjs.org/blog/2020/09/22/introducing-the-new-jsx-transform.html#whats-a-jsx-transform)。JSX 是一种语法扩展，而 React 是一个 JavaScript 库。

</Note>

## 将 HTML 转换为 JSX {/*converting-html-to-jsx*/}

假设你有一些（完全有效的）HTML：

```html
<h1>Hedy Lamarr's Todos</h1>
<img
  src="https://react.dev/images/docs/scientists/yXOvdOSs.jpg"
  alt="Hedy Lamarr"
  class="photo"
>
<ul>
    <li>Invent new traffic lights
    <li>Rehearse a movie scene
    <li>Improve the spectrum technology
</ul>
```

而你想把它放进你的组件中：

```js
export default function TodoList() {
  return (
    // ???
  )
}
```

如果你原样复制并粘贴，它将无法工作：


<Sandpack>

```js
export default function TodoList() {
  return (
    // 这样不完全行！
    <h1>Hedy Lamarr's Todos</h1>
    <img
      src="https://react.dev/images/docs/scientists/yXOvdOSs.jpg"
      alt="Hedy Lamarr"
      class="photo"
    >
    <ul>
      <li>Invent new traffic lights
      <li>Rehearse a movie scene
      <li>Improve the spectrum technology
    </ul>
  );
}
```

```css
img { height: 90px }
```

</Sandpack>

这是因为 JSX 更严格，并且比 HTML 多一些规则！如果你阅读上面的错误信息，它们会引导你修复标记；或者你也可以按照下面的指南来做。

<Note>

大多数时候，React 屏幕上的错误信息会帮助你找到问题所在。如果你卡住了，不妨读一读它们！

</Note>

## JSX 的规则 {/*the-rules-of-jsx*/}

### 1. 返回单个根元素 {/*1-return-a-single-root-element*/}

要从组件中返回多个元素，**请用一个单独的父标签把它们包裹起来。**

例如，你可以使用 `<div>`：

```js {1,11}
<div>
  <h1>Hedy Lamarr's Todos</h1>
  <img
    src="https://react.dev/images/docs/scientists/yXOvdOSs.jpg"
    alt="Hedy Lamarr"
    class="photo"
  >
  <ul>
    ...
  </ul>
</div>
```


如果你不想在标记中额外添加一个 `<div>`，也可以改写为 `<>` 和 `</>`：

```js {1,11}
<>
  <h1>Hedy Lamarr's Todos</h1>
  <img
    src="https://react.dev/images/docs/scientists/yXOvdOSs.jpg"
    alt="Hedy Lamarr"
    class="photo"
  >
  <ul>
    ...
  </ul>
</>
```

这个空标签称为 *[Fragment.](/reference/react/Fragment)* Fragment 允许你将内容分组，而不会在浏览器的 HTML 树中留下任何痕迹。

<DeepDive>

#### 为什么多个 JSX 标签需要被包裹起来？ {/*why-do-multiple-jsx-tags-need-to-be-wrapped*/}

JSX 看起来像 HTML，但在底层它会被转换成普通的 JavaScript 对象。你不能在不把它们包装到数组中的情况下从函数中返回两个对象。这也解释了为什么你不能在不把两个 JSX 标签包裹到另一个标签或 Fragment 中的情况下返回它们。

</DeepDive>

### 2. 关闭所有标签 {/*2-close-all-the-tags*/}

JSX 要求标签必须显式闭合：像 `<img>` 这样的自闭合标签必须写成 `<img />`，而像 `<li>oranges` 这样的包裹标签必须写成 `<li>oranges</li>`。

下面是 Hedy Lamarr 的图片和列表项闭合后的样子：

```js {2-6,8-10}
<>
  <img
    src="https://react.dev/images/docs/scientists/yXOvdOSs.jpg"
    alt="Hedy Lamarr"
    class="photo"
   />
  <ul>
    <li>Invent new traffic lights</li>
    <li>Rehearse a movie scene</li>
    <li>Improve the spectrum technology</li>
  </ul>
</>
```

### 3. 大多数东西都要用 camelCase！ {/*3-camelcase-salls-most-of-the-things*/}

JSX 会变成 JavaScript，而在 JSX 中编写的属性会成为 JavaScript 对象的键。在你自己的组件中，你经常会想把这些属性读入变量中。但 JavaScript 对变量名有一些限制。例如，变量名不能包含连字符，也不能是像 `class` 这样的保留字。

这就是为什么在 React 中，许多 HTML 和 SVG 属性都使用 camelCase 编写。例如，不是使用 `stroke-width`，而是使用 `strokeWidth`。由于 `class` 是保留字，在 React 中你要改用 `className`，这个名称来源于 [对应的 DOM 属性](https://developer.mozilla.org/en-US/docs/Web/API/Element/className)：

```js {4}
<img
  src="https://react.dev/images/docs/scientists/yXOvdOSs.jpg"
  alt="Hedy Lamarr"
  className="photo"
/>
```

你可以在 [DOM 组件 props 列表中找到所有这些属性。](/reference/react-dom/components/common) 如果你写错了，也不用担心——React 会在 [浏览器控制台](https://developer.mozilla.org/docs/Tools/Browser_Console) 中打印一条可能的更正信息。

<Pitfall>

出于历史原因，[`aria-*`](https://developer.mozilla.org/docs/Web/Accessibility/ARIA) 和 [`data-*`](https://developer.mozilla.org/docs/Learn/HTML/Howto/Use_data_attributes) 属性仍然像 HTML 一样使用连字符编写。

</Pitfall>

### 专业提示：使用 JSX 转换器 {/*pro-tip-use-a-jsx-converter*/}

把现有标记中的所有这些属性都转换过来可能会很繁琐！我们建议使用一个 [转换器](https://transform.tools/html-to-jsx) 来把你现有的 HTML 和 SVG 翻译成 JSX。转换器在实践中非常有用，但理解其原理仍然很值得，这样你就能自如地自己编写 JSX。

以下是最终结果：

<Sandpack>

```js
export default function TodoList() {
  return (
    <>
      <h1>Hedy Lamarr's Todos</h1>
      <img
        src="https://react.dev/images/docs/scientists/yXOvdOSs.jpg"
        alt="Hedy Lamarr"
        className="photo"
      />
      <ul>
        <li>Invent new traffic lights</li>
        <li>Rehearse a movie scene</li>
        <li>Improve the spectrum technology</li>
      </ul>
    </>
  );
}
```

```css
img { height: 90px }
```

</Sandpack>

<Recap>

现在你知道为什么会有 JSX，以及如何在组件中使用它了：

* React 组件把渲染逻辑和标记组合在一起，因为它们彼此相关。
* JSX 与 HTML 类似，但有一些不同。如果需要，你可以使用 [转换器](https://transform.tools/html-to-jsx)。
* 错误信息通常会指引你朝正确的方向去修复标记。

</Recap>



<Challenges>

#### 将一些 HTML 转换为 JSX {/*convert-some-html-to-jsx*/}

这段 HTML 被粘贴进了一个组件中，但它不是有效的 JSX。请修复它：

<Sandpack>

```js
export default function Bio() {
  return (
    <div class="intro">
      <h1>欢迎来到我的网站！</h1>
    </div>
    <p class="summary">
      你可以在这里找到我的想法。
      <br><br>
      <b>还有科学家的 <i>图片</b></i>！
    </p>
  );
}
```

```css
.intro {
  background-image: linear-gradient(to left, violet, indigo, blue, green, yellow, orange, red);
  background-clip: text;
  color: transparent;
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}

.summary {
  padding: 20px;
  border: 10px solid gold;
}
```

</Sandpack>

是手动完成还是使用转换器，都由你决定！

<Solution>

<Sandpack>

```js
export default function Bio() {
  return (
    <div>
      <div className="intro">
        <h1>欢迎来到我的网站！</h1>
      </div>
      <p className="summary">
        你可以在这里找到我的想法。
        <br /><br />
        <b>还有科学家的 <i>图片</i></b>！
      </p>
    </div>
  );
}
```

```css
.intro {
  background-image: linear-gradient(to left, violet, indigo, blue, green, yellow, orange, red);
  background-clip: text;
  color: transparent;
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}

.summary {
  padding: 20px;
  border: 10px solid gold;
}
```

</Sandpack>

</Solution>

</Challenges>
