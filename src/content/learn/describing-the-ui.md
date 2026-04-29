---
title: 描述 UI
---

<Intro>

React 是一个用于渲染用户界面（UI）的 JavaScript 库。UI 由按钮、文本和图片等小单元构成。React 让你可以把它们组合成可复用、可嵌套的 *组件*。从网站到手机应用，屏幕上的一切都可以拆分为组件。在本章中，你将学习如何创建、定制以及有条件地显示 React 组件。

</Intro>

<YouWillLearn isChapter={true}>

* [如何编写你的第一个 React 组件](/learn/your-first-component)
* [何时以及如何创建多组件文件](/learn/importing-and-exporting-components)
* [如何使用 JSX 向 JavaScript 添加标记](/learn/writing-markup-with-jsx)
* [如何在 JSX 中使用花括号，从组件中访问 JavaScript 功能](/learn/javascript-in-jsx-with-curly-braces)
* [如何使用 props 配置组件](/learn/passing-props-to-a-component)
* [如何有条件地渲染组件](/learn/conditional-rendering)
* [如何一次渲染多个组件](/learn/rendering-lists)
* [如何通过保持组件纯净来避免令人困惑的 bug](/learn/keeping-components-pure)
* [为什么将 UI 理解为树很有用](/learn/understanding-your-ui-as-a-tree)

</YouWillLearn>

## 你的第一个组件 {/*your-first-component*/}

React 应用由称为 *组件* 的独立 UI 片段构成。React 组件是一个可以穿插标记的 JavaScript 函数。组件可以小到一个按钮，也可以大到整个页面。下面是一个渲染三个 `Profile` 组件的 `Gallery` 组件：

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
      <h1>杰出的科学家</h1>
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

<LearnMore path="/learn/your-first-component">

阅读 **[你的第一个组件](/learn/your-first-component)**，了解如何声明和使用 React 组件。

</LearnMore>

## 导入和导出组件 {/*importing-and-exporting-components*/}

你可以在一个文件中声明许多组件，但大型文件可能会变得难以浏览。为了解决这个问题，你可以将组件 *导出* 到它自己的文件中，然后从另一个文件中 *导入* 这个组件：


<Sandpack>

```js src/App.js hidden
import Gallery from './Gallery.js';

export default function App() {
  return (
    <Gallery />
  );
}
```

```js src/Gallery.js active
import Profile from './Profile.js';

export default function Gallery() {
  return (
    <section>
      <h1>杰出的科学家</h1>
      <Profile />
      <Profile />
      <Profile />
    </section>
  );
}
```

```js src/Profile.js
export default function Profile() {
  return (
    <img
      src="https://react.dev/images/docs/scientists/QIrZWGIs.jpg"
      alt="Alan L. Hart"
    />
  );
}
```

```css
img { margin: 0 10px 10px 0; }
```

</Sandpack>

<LearnMore path="/learn/importing-and-exporting-components">

阅读 **[导入和导出组件](/learn/importing-and-exporting-components)**，了解如何将组件拆分到它们自己的文件中。

</LearnMore>

## 使用 JSX 编写标记 {/*writing-markup-with-jsx*/}

每个 React 组件都是一个 JavaScript 函数，其中可能包含一些由 React 渲染到浏览器中的标记。React 组件使用一种名为 JSX 的语法扩展来表示这些标记。JSX 看起来很像 HTML，但它更严格一些，并且可以显示动态信息。

如果我们把已有的 HTML 标记粘贴到 React 组件中，它并不总是能正常工作：

<Sandpack>

```js
export default function TodoList() {
  return (
    // 这不太行！
    <h1>Hedy Lamarr 的待办事项</h1>
    <img
      src="https://react.dev/images/docs/scientists/yXOvdOSs.jpg"
      alt="Hedy Lamarr"
      class="photo"
    >
    <ul>
      <li>Invent new traffic lights
      <li>Rehearse a movie scene
      <li>Improve spectrum technology
    </ul>
  );
}
```

```css
img { height: 90px; }
```

</Sandpack>

如果你已有这样的 HTML，可以使用一个 [转换器](https://transform.tools/html-to-jsx) 来修复它：

<Sandpack>

```js
export default function TodoList() {
  return (
    <>
      <h1>Hedy Lamarr 的待办事项</h1>
      <img
        src="https://react.dev/images/docs/scientists/yXOvdOSs.jpg"
        alt="Hedy Lamarr"
        className="photo"
      />
      <ul>
        <li>Invent new traffic lights</li>
        <li>Rehearse a movie scene</li>
        <li>Improve spectrum technology</li>
      </ul>
    </>
  );
}
```

```css
img { height: 90px; }
```

</Sandpack>

<LearnMore path="/learn/writing-markup-with-jsx">

阅读 **[使用 JSX 编写标记](/learn/writing-markup-with-jsx)**，了解如何编写有效的 JSX。

</LearnMore>

## 在 JSX 中使用花括号编写 JavaScript {/*javascript-in-jsx-with-curly-braces*/}

JSX 让你可以在 JavaScript 文件中编写类似 HTML 的标记，从而将渲染逻辑和内容放在同一个地方。有时你会想在这些标记中添加一点 JavaScript 逻辑，或者引用某个动态属性。在这种情况下，你可以在 JSX 中使用花括号，向 JavaScript “打开一扇窗”：

<Sandpack>

```js
const person = {
  name: 'Gregorio Y. Zara',
  theme: {
    backgroundColor: 'black',
    color: 'pink'
  }
};

export default function TodoList() {
  return (
    <div style={person.theme}>
      <h1>{person.name} 的待办事项</h1>
      <img
        className="avatar"
        src="https://react.dev/images/docs/scientists/7vQD0fPs.jpg"
        alt="Gregorio Y. Zara"
      />
      <ul>
        <li>Improve the videophone</li>
        <li>Prepare aeronautics lectures</li>
        <li>Work on the alcohol-fuelled engine</li>
      </ul>
    </div>
  );
}
```

```css
body { padding: 0; margin: 0 }
body > div > div { padding: 20px; }
.avatar { border-radius: 50%; height: 90px; }
```

</Sandpack>

<LearnMore path="/learn/javascript-in-jsx-with-curly-braces">

阅读 **[在 JSX 中使用花括号编写 JavaScript](/learn/javascript-in-jsx-with-curly-braces)**，了解如何从 JSX 访问 JavaScript 数据。

</LearnMore>

## 向组件传递 props {/*passing-props-to-a-component*/}

React 组件使用 *props* 来彼此通信。每个父组件都可以通过给子组件传递 props，向它们提供一些信息。props 可能会让你联想到 HTML 属性，但你可以通过它们传递任何 JavaScript 值，包括对象、数组、函数，甚至 JSX！

<Sandpack>

```js
import { getImageUrl } from './utils.js'

export default function Profile() {
  return (
    <Card>
      <Avatar
        size={100}
        person={{
          name: 'Katsuko Saruhashi',
          imageId: 'YfeOqp2'
        }}
      />
    </Card>
  );
}

function Avatar({ person, size }) {
  return (
    <img
      className="avatar"
      src={getImageUrl(person)}
      alt={person.name}
      width={size}
      height={size}
    />
  );
}

function Card({ children }) {
  return (
    <div className="card">
      {children}
    </div>
  );
}

```

```js src/utils.js
export function getImageUrl(person, size = 's') {
  return (
    'https://react.dev/images/docs/scientists/' +
    person.imageId +
    size +
    '.jpg'
  );
}
```

```css
.card {
  width: fit-content;
  margin: 5px;
  padding: 5px;
  font-size: 20px;
  text-align: center;
  border: 1px solid #aaa;
  border-radius: 20px;
  background: #fff;
}
.avatar {
  margin: 20px;
  border-radius: 50%;
}
```

</Sandpack>

<LearnMore path="/learn/passing-props-to-a-component">

阅读 **[向组件传递 props](/learn/passing-props-to-a-component)**，了解如何传递和读取 props。

</LearnMore>

## 条件渲染 {/*conditional-rendering*/}

你的组件通常需要根据不同条件显示不同内容。在 React 中，你可以使用 JavaScript 语法如 `if` 语句、`&&` 和 `? :` 运算符来有条件地渲染 JSX。

在这个例子中，使用 JavaScript 的 `&&` 运算符来有条件地渲染一个对勾：

<Sandpack>

```js
function Item({ name, isPacked }) {
  return (
    <li className="item">
      {name} {isPacked && '✅'}
    </li>
  );
}

export default function PackingList() {
  return (
    <section>
      <h1>Sally Ride 的打包清单</h1>
      <ul>
        <Item
          isPacked={true}
          name="Space suit"
        />
        <Item
          isPacked={true}
          name="Helmet with a golden leaf"
        />
        <Item
          isPacked={false}
          name="Photo of Tam"
        />
      </ul>
    </section>
  );
}
```

</Sandpack>

<LearnMore path="/learn/conditional-rendering">

阅读 **[条件渲染](/learn/conditional-rendering)**，了解有条件地渲染内容的不同方式。

</LearnMore>

## 渲染列表 {/*rendering-lists*/}

你经常会希望从一组数据中显示多个相似的组件。你可以将 JavaScript 的 `filter()` 和 `map()` 与 React 一起使用，把数据数组筛选并转换为组件数组。

对于每个数组项，你都需要指定一个 `key`。通常，你会希望使用数据库中的 ID 作为 `key`。即使列表发生变化，key 也能让 React 追踪列表中每一项的位置。

<Sandpack>

```js src/App.js
import { people } from './data.js';
import { getImageUrl } from './utils.js';

export default function List() {
  const listItems = people.map(person =>
    <li key={person.id}>
      <img
        src={getImageUrl(person)}
        alt={person.name}
      />
      <p>
        <b>{person.name}:</b>
        {' ' + person.profession + ' '}
        擅长 {person.accomplishment}
      </p>
    </li>
  );
  return (
    <article>
      <h1>科学家</h1>
      <ul>{listItems}</ul>
    </article>
  );
}
```

```js src/data.js
export const people = [{
  id: 0,
  name: 'Creola Katherine Johnson',
  profession: 'mathematician',
  accomplishment: 'spaceflight calculations',
  imageId: 'MK3eW3A'
}, {
  id: 1,
  name: 'Mario José Molina-Pasquel Henríquez',
  profession: 'chemist',
  accomplishment: 'discovery of Arctic ozone hole',
  imageId: 'mynHUSa'
}, {
  id: 2,
  name: 'Mohammad Abdus Salam',
  profession: 'physicist',
  accomplishment: 'electromagnetism theory',
  imageId: 'bE7W1ji'
}, {
  id: 3,
  name: 'Percy Lavon Julian',
  profession: 'chemist',
  accomplishment: 'pioneering cortisone drugs, steroids and birth control pills',
  imageId: 'IOjWm71'
}, {
  id: 4,
  name: 'Subrahmanyan Chandrasekhar',
  profession: 'astrophysicist',
  accomplishment: 'white dwarf star mass calculations',
  imageId: 'lrWQx8l'
}];
```

```js src/utils.js
export function getImageUrl(person) {
  return (
    'https://react.dev/images/docs/scientists/' +
    person.imageId +
    's.jpg'
  );
}
```

```css
ul { list-style-type: none; padding: 0px 10px; }
li {
  margin-bottom: 10px;
  display: grid;
  grid-template-columns: 1fr 1fr;
  align-items: center;
}
img { width: 100px; height: 100px; border-radius: 50%; }
h1 { font-size: 22px; }
h2 { font-size: 20px; }
```

</Sandpack>

<LearnMore path="/learn/rendering-lists">

阅读 **[渲染列表](/learn/rendering-lists)**，了解如何渲染组件列表，以及如何选择 key。

</LearnMore>

## 将组件保持纯净 {/*keeping-components-pure*/}

一些 JavaScript 函数是*纯函数*。纯函数：

* **只处理自己的事情。** 它不会更改在调用之前已存在的任何对象或变量。
* **相同的输入，相同的输出。** 给定相同的输入，纯函数应始终返回相同的结果。

通过严格地只将组件编写为纯函数，你可以避免随着代码库增长而出现的一整类令人困惑的 bug 和不可预测的行为。下面是一个不纯组件的示例：

<Sandpack>

```js {expectedErrors: {'react-compiler': [5]}}
let guest = 0;

function Cup() {
  // 糟糕：正在修改一个已存在的变量！
  guest = guest + 1;
  return <h2>给第 #{guest} 位客人的茶杯</h2>;
}

export default function TeaSet() {
  return (
    <>
      <Cup />
      <Cup />
      <Cup />
    </>
  );
}
```

</Sandpack>

你可以通过传递 prop，而不是修改一个已存在的变量，来让这个组件变成纯函数：

<Sandpack>

```js
function Cup({ guest }) {
  return <h2>给第 #{guest} 位客人的茶杯</h2>;
}

export default function TeaSet() {
  return (
    <>
      <Cup guest={1} />
      <Cup guest={2} />
      <Cup guest={3} />
    </>
  );
}
```

</Sandpack>

<LearnMore path="/learn/keeping-components-pure">

阅读 **[Keeping Components Pure](/learn/keeping-components-pure)**，了解如何将组件编写为纯净、可预测的函数。

</LearnMore>

## 你的 UI 是一棵树 {/*your-ui-as-a-tree*/}

React 使用树来表示组件和模块之间的关系。

React 渲染树是组件之间父子关系的一种表示。

<Diagram name="generic_render_tree" height={250} width={500} alt="A tree graph with five nodes, with each node representing a component. The root node is located at the top the tree graph and is labelled 'Root Component'. It has two arrows extending down to two nodes labelled 'Component A' and 'Component C'. Each of the arrows is labelled with 'renders'. 'Component A' has a single 'renders' arrow to a node labelled 'Component B'. 'Component C' has a single 'renders' arrow to a node labelled 'Component D'.">

一个 React 渲染树示例。

</Diagram>

树顶部、靠近根组件的组件被视为顶层组件。没有子组件的组件是叶子组件。这种对组件的分类有助于理解数据流和渲染性能。

将 JavaScript 模块之间的关系建模，是理解你的应用的另一种有用方式。我们将其称为模块依赖树。

<Diagram name="generic_dependency_tree" height={250} width={500} alt="A tree graph with five nodes. Each node represents a JavaScript module. The top-most node is labelled 'RootModule.js'. It has three arrows extending to the nodes: 'ModuleA.js', 'ModuleB.js', and 'ModuleC.js'. Each arrow is labelled as 'imports'. 'ModuleC.js' node has a single 'imports' arrow that points to a node labelled 'ModuleD.js'.">

一个模块依赖树示例。

</Diagram>

构建工具通常会使用依赖树来把客户端下载和渲染所需的所有相关 JavaScript 代码打包在一起。过大的 bundle 会降低 React 应用的用户体验。理解模块依赖树有助于调试这类问题。

<LearnMore path="/learn/understanding-your-ui-as-a-tree">

阅读 **[Your UI as a Tree](/learn/understanding-your-ui-as-a-tree)**，了解如何为 React 应用创建渲染树和模块依赖树，以及它们如何作为改进用户体验和性能的有用心智模型。

</LearnMore>


## 接下来是什么？ {/*whats-next*/}

前往 [Your First Component](/learn/your-first-component)，开始逐页阅读本章节！

或者，如果你已经熟悉这些主题，为什么不继续阅读 [Adding Interactivity](/learn/adding-interactivity) 呢？
