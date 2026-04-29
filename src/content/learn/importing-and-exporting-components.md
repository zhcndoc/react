---
title: 导入和导出组件
---

<Intro>

组件的魔力在于其可复用性：你可以创建由其他组件组成的组件。但随着你嵌套越来越多的组件，通常把它们拆分到不同文件中会更合理。这可以让你的文件更易于浏览，并在更多地方复用组件。

</Intro>

<YouWillLearn>

* 什么是根组件文件
* 如何导入和导出组件
* 何时使用默认导入/导出和命名导入/导出
* 如何从一个文件中导入和导出多个组件
* 如何将组件拆分到多个文件中

</YouWillLearn>

## 根组件文件 {/*the-root-component-file*/}

在 [你的第一个组件](/learn/your-first-component) 中，你创建了一个 `Profile` 组件和一个渲染它的 `Gallery` 组件：

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

这些内容目前位于一个**根组件文件**中，在这个示例里它的名字是 `App.js`。不过，具体取决于你的设置，根组件也可能位于其他文件中。如果你使用的是基于文件路由的框架，例如 Next.js，那么每个页面的根组件都不同。

## 导出和导入组件 {/*exporting-and-importing-a-component*/}

如果你将来想把着陆页改成展示一份科学书籍列表呢？或者把所有个人资料放到别的地方呢？把 `Gallery` 和 `Profile` 移出根组件文件是很合理的。这样可以让它们更模块化，也更容易在其他文件中复用。你可以分三步移动一个组件：

1. **创建** 一个新的 JS 文件来放这些组件。
2. 从那个文件中**导出**你的函数组件（使用 [默认](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Statements/export#using_the_default_export) 导出或 [命名](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Statements/export#using_named_exports) 导出）。
3. 在你要使用该组件的文件中**导入**它（使用对应的 [默认](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Statements/import#importing_defaults) 导入或 [命名](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Statements/import#import_a_single_export_from_a_module) 导入方式）。

这里，`Profile` 和 `Gallery` 都已经从 `App.js` 移动到了一个名为 `Gallery.js` 的新文件中。现在你可以修改 `App.js`，从 `Gallery.js` 导入 `Gallery`：

<Sandpack>

```js src/App.js
import Gallery from './Gallery.js';

export default function App() {
  return (
    <Gallery />
  );
}
```

```js src/Gallery.js
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
img { margin: 0 10px 10px 0; height: 90px; }
```

</Sandpack>

注意这个示例现在被拆分成了两个组件文件：

1. `Gallery.js`：
     - 定义了 `Profile` 组件，它仅在同一个文件中使用，并且没有被导出。
     - 将 `Gallery` 组件作为**默认导出**导出。
2. `App.js`：
     - 从 `Gallery.js` 中将 `Gallery` 作为**默认导入**导入。
     - 将根 `App` 组件作为**默认导出**导出。


<Note>

你可能会遇到省略 `.js` 文件扩展名的写法，例如：

```js
import Gallery from './Gallery';
```

`'./Gallery.js'` 或 `'./Gallery'` 都可以在 React 中正常工作，不过前者更接近 [原生 ES Modules](https://developer.mozilla.org/docs/Web/JavaScript/Guide/Modules) 的工作方式。

</Note>

<DeepDive>

#### 默认导出与命名导出 {/*default-vs-named-exports*/}

用 JavaScript 导出值主要有两种方式：默认导出和命名导出。到目前为止，我们的示例只使用了默认导出。但你可以在同一个文件中使用其中一种或两种。**一个文件最多只能有一个 _默认_ 导出，但可以拥有任意多个 _命名_ 导出。**

![默认导出和命名导出](/images/docs/illustrations/i_import-export.svg)

你如何导出组件，决定了你必须如何导入它。如果你尝试以命名导出的方式导入默认导出，就会报错！这个表格可以帮助你区分：

| 语法           | 导出语句                           | 导入语句                          |
| -----------      | -----------                                | -----------                               |
| 默认  | `export default function Button() {}` | `import Button from './Button.js';`     |
| 命名    | `export function Button() {}`         | `import { Button } from './Button.js';` |

当你写一个 _默认_ 导入时，`import` 后面可以使用任何你想要的名字。例如，你也可以写成 `import Banana from './Button.js'`，它仍然会给你同样的默认导出。相反，对于命名导入，两边的名字必须匹配。这就是它们被称为 _命名_ 导入的原因！

**如果文件只导出一个组件，人们通常会使用默认导出；如果文件导出多个组件和其他值，则通常使用命名导出。** 无论你偏好哪种编码风格，都要始终给组件函数以及包含它们的文件起有意义的名字。像 `export default () => {}` 这样的无名组件不被推荐，因为它们会让调试更困难。

</DeepDive>

## 从同一个文件中导出和导入多个组件 {/*exporting-and-importing-multiple-components-from-the-same-file*/}

如果你只想显示一个 `Profile`，而不是一个画廊呢？你也可以导出 `Profile` 组件。不过 `Gallery.js` 已经有一个 *默认* 导出了，你不能有 _两个_ 默认导出。你可以创建一个带默认导出的新文件，或者给 `Profile` 添加一个 *命名* 导出。**一个文件只能有一个默认导出，但可以有很多命名导出！**

<Note>

为了减少默认导出和命名导出之间可能产生的混淆，有些团队选择只坚持一种风格（默认或命名），或者避免在同一个文件中混用它们。选择最适合你的方式即可！

</Note>

首先，使用命名导出从 `Gallery.js` **导出** `Profile`（不使用 `default` 关键字）：

```js
export function Profile() {
  // ...
}
```

然后，使用命名导入（带花括号）将 `Profile` 从 `Gallery.js` **导入**到 `App.js`：

```js
import { Profile } from './Gallery.js';
```

最后，从 `App` 组件中**渲染** `<Profile />`：

```js
export default function App() {
  return <Profile />;
}
```

现在 `Gallery.js` 包含两个导出：一个默认的 `Gallery` 导出，以及一个命名的 `Profile` 导出。`App.js` 同时导入了它们。尝试在这个示例中把 `<Profile />` 改成 `<Gallery />`，再改回来：

<Sandpack>

```js src/App.js
import Gallery from './Gallery.js';
import { Profile } from './Gallery.js';

export default function App() {
  return (
    <Profile />
  );
}
```

```js src/Gallery.js
export function Profile() {
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
img { margin: 0 10px 10px 0; height: 90px; }
```

</Sandpack>

现在你正在混合使用默认导出和命名导出：

* `Gallery.js`：
  - 将 `Profile` 组件作为一个**名为 `Profile` 的命名导出**导出。
  - 将 `Gallery` 组件作为**默认导出**导出。
* `App.js`：
  - 从 `Gallery.js` 中将 `Profile` 作为一个**名为 `Profile` 的命名导入**导入。
  - 从 `Gallery.js` 中将 `Gallery` 作为**默认导入**导入。
  - 将根 `App` 组件作为**默认导出**导出。

<Recap>

在本页中你学到了：

* 什么是根组件文件
* 如何导入和导出组件
* 何时以及如何使用默认导入/导出和命名导入/导出
* 如何从同一个文件中导出多个组件

</Recap>



<Challenges>

#### 进一步拆分组件 {/*split-the-components-further*/}

目前，`Gallery.js` 同时导出 `Profile` 和 `Gallery`，这有点让人困惑。

把 `Profile` 组件移动到它自己的 `Profile.js` 中，然后把 `App` 组件改为依次渲染 `<Profile />` 和 `<Gallery />`。

你可以为 `Profile` 使用默认导出或命名导出，但请确保在 `App.js` 和 `Gallery.js` 中使用对应的导入语法！你可以参考上面深度解析中的表格：

| 语法           | 导出语句                           | 导入语句                          |
| -----------      | -----------                                | -----------                               |
| 默认  | `export default function Button() {}` | `import Button from './Button.js';`     |
| 命名    | `export function Button() {}`         | `import { Button } from './Button.js';` |

<Hint>

不要忘记在调用组件的地方导入它们。难道 `Gallery` 也不会用到 `Profile` 吗？

</Hint>

<Sandpack>

```js src/App.js
import Gallery from './Gallery.js';
import { Profile } from './Gallery.js';

export default function App() {
  return (
    <div>
      <Profile />
    </div>
  );
}
```

```js src/Gallery.js active
// 把我移动到 Profile.js！
export function Profile() {
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

```js src/Profile.js
```

```css
img { margin: 0 10px 10px 0; height: 90px; }
```

</Sandpack>

在先用一种导出方式把它做通之后，再用另一种方式实现一次。

<Solution>

这是使用命名导出的解决方案：

<Sandpack>

```js src/App.js
import Gallery from './Gallery.js';
import { Profile } from './Profile.js';

export default function App() {
  return (
    <div>
      <Profile />
      <Gallery />
    </div>
  );
}
```

```js src/Gallery.js
import { Profile } from './Profile.js';

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

```js src/Profile.js
export function Profile() {
  return (
    <img
      src="https://react.dev/images/docs/scientists/QIrZWGIs.jpg"
      alt="Alan L. Hart"
    />
  );
}
```

```css
img { margin: 0 10px 10px 0; height: 90px; }
```

</Sandpack>

这是使用默认导出的解决方案：

<Sandpack>

```js src/App.js
import Gallery from './Gallery.js';
import Profile from './Profile.js';

export default function App() {
  return (
    <div>
      <Profile />
      <Gallery />
    </div>
  );
}
```

```js src/Gallery.js
import Profile from './Profile.js';

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
img { margin: 0 10px 10px 0; height: 90px; }
```

</Sandpack>

</Solution>

</Challenges>
