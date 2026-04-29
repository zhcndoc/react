---
title: 在 JSX 中使用花括号的 JavaScript
---

<Intro>

JSX 让你可以在 JavaScript 文件中编写类似 HTML 的标记，把渲染逻辑和内容放在同一个地方。有时你会想在这些标记中加入一点 JavaScript 逻辑，或者引用一个动态属性。在这种情况下，你可以在 JSX 中使用花括号，为 JavaScript 打开一扇窗。

</Intro>

<YouWillLearn>

* 如何使用引号传递字符串
* 如何在 JSX 中使用花括号引用 JavaScript 变量
* 如何在 JSX 中使用花括号调用 JavaScript 函数
* 如何在 JSX 中使用花括号使用 JavaScript 对象

</YouWillLearn>

## 使用引号传递字符串 {/*passing-strings-with-quotes*/}

当你想向 JSX 传递字符串属性时，可以把它放在单引号或双引号中：

<Sandpack>

```js
export default function Avatar() {
  return (
    <img
      className="avatar"
      src="https://react.dev/images/docs/scientists/7vQD0fPs.jpg"
      alt="Gregorio Y. Zara"
    />
  );
}
```

```css
.avatar { border-radius: 50%; height: 90px; }
```

</Sandpack>

这里，`"https://react.dev/images/docs/scientists/7vQD0fPs.jpg"` 和 `"Gregorio Y. Zara"` 都是以字符串形式传递的。

但如果你想动态指定 `src` 或 `alt` 文本呢？你可以**通过将 `"` 和 `"` 替换为 `{` 和 `}`，来使用 JavaScript 中的值**：

<Sandpack>

```js
export default function Avatar() {
  const avatar = 'https://react.dev/images/docs/scientists/7vQD0fPs.jpg';
  const description = 'Gregorio Y. Zara';
  return (
    <img
      className="avatar"
      src={avatar}
      alt={description}
    />
  );
}
```

```css
.avatar { border-radius: 50%; height: 90px; }
```

</Sandpack>

注意 `className="avatar"` 和 `src={avatar}` 之间的区别：前者指定了一个 `"avatar"` 的 CSS 类名，让图片变成圆形；后者则读取了名为 `avatar` 的 JavaScript 变量的值。这是因为花括号让你可以直接在标记中使用 JavaScript！

## 使用花括号：通往 JavaScript 世界的一扇窗 {/*using-curly-braces-a-window-into-the-javascript-world*/}

JSX 是一种特殊的 JavaScript 编写方式。这意味着你可以在其中使用 JavaScript——通过花括号 `{ }`。下面的示例先声明科学家的名字 `name`，然后将它通过花括号嵌入 `<h1>` 中：

<Sandpack>

```js
export default function TodoList() {
  const name = 'Gregorio Y. Zara';
  return (
    <h1>{name}'s To Do List</h1>
  );
}
```

</Sandpack>

试着把 `name` 的值从 `'Gregorio Y. Zara'` 改为 `'Hedy Lamarr'`。看看列表标题是如何变化的？

任何 JavaScript 表达式都可以放在花括号之间，包括像 `formatDate()` 这样的函数调用：

<Sandpack>

```js
const today = new Date();

function formatDate(date) {
  return new Intl.DateTimeFormat(
    'en-US',
    { weekday: 'long' }
  ).format(date);
}

export default function TodoList() {
  return (
    <h1>To Do List for {formatDate(today)}</h1>
  );
}
```

</Sandpack>

### 在哪里使用花括号 {/*where-to-use-curly-braces*/}

在 JSX 中，花括号只能以两种方式使用：

1. **作为文本** 直接写在 JSX 标签内部：`<h1>{name}'s To Do List</h1>` 可以，但 `<{tag}>Gregorio Y. Zara's To Do List</{tag}>` 不行。
2. **作为属性** 紧跟在 `=` 符号之后：`src={avatar}` 会读取 `avatar` 变量，但 `src="{avatar}"` 传递的只是字符串 `"{avatar}"`。

## 使用“双花括号”：JSX 中的 CSS 和其他对象 {/*using-double-curlies-css-and-other-objects-in-jsx*/}

除了字符串、数字和其他 JavaScript 表达式之外，你甚至还可以在 JSX 中传递对象。对象也用花括号表示，例如 `{ name: "Hedy Lamarr", inventions: 5 }`。因此，要在 JSX 中传递一个 JS 对象，你必须再包上一层花括号：`person={{ name: "Hedy Lamarr", inventions: 5 }}`。

你可能会在 JSX 的行内 CSS 样式中见到这种写法。React 并不要求你使用行内样式（对大多数情况来说，CSS 类就很好用）。但当你需要行内样式时，可以向 `style` 属性传递一个对象：

<Sandpack>

```js
export default function TodoList() {
  return (
    <ul style={{
      backgroundColor: 'black',
      color: 'pink'
    }}>
      <li>Improve the videophone</li>
      <li>Prepare aeronautics lectures</li>
      <li>Work on the alcohol-fuelled engine</li>
    </ul>
  );
}
```

```css
body { padding: 0; margin: 0 }
ul { padding: 20px 20px 20px 40px; margin: 0; }
```

</Sandpack>

试着改变 `backgroundColor` 和 `color` 的值。

当你像这样写时，就能清楚地看到花括号中的 JavaScript 对象：

```js {2-5}
<ul style={
  {
    backgroundColor: 'black',
    color: 'pink'
  }
}>
```

下次在 JSX 中看到 `{{` 和 `}}` 时，要知道这不过是 JSX 花括号里包着的一个对象而已！

<Pitfall>

行内 `style` 属性使用驼峰命名法编写。例如，HTML 中的 `<ul style="background-color: black">` 在你的组件里应写成 `<ul style={{ backgroundColor: 'black' }}>`。

</Pitfall>

## 用 JavaScript 对象和花括号玩出更多花样 {/*more-fun-with-javascript-objects-and-curly-braces*/}

你可以把多个表达式放进一个对象中，然后在 JSX 的花括号里引用它们：

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
      <h1>{person.name}'s Todos</h1>
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

在这个示例中，`person` JavaScript 对象包含一个 `name` 字符串和一个 `theme` 对象：

```js
const person = {
  name: 'Gregorio Y. Zara',
  theme: {
    backgroundColor: 'black',
    color: 'pink'
  }
};
```

组件可以像这样使用 `person` 中的这些值：

```js
<div style={person.theme}>
  <h1>{person.name}'s Todos</h1>
```

作为模板语言，JSX 非常精简，因为它让你可以使用 JavaScript 来组织数据和逻辑。

<Recap>

现在你已经知道了几乎所有关于 JSX 的内容：

* 引号中的 JSX 属性会作为字符串传递。
* 花括号让你可以把 JavaScript 逻辑和变量带入标记中。
* 它们可以用于 JSX 标签内容中，或者属性中紧跟在 `=` 后面。
* `{{` 和 `}}` 并不是特殊语法：它只是一个放在 JSX 花括号里的 JavaScript 对象。

</Recap>

<Challenges>

#### 修复错误 {/*fix-the-mistake*/}

这段代码会报错，错误信息是 `Objects are not valid as a React child`：

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
      <h1>{person}'s Todos</h1>
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

你能找出问题吗？

<Hint>看看花括号里是什么。我们放进去的是正确的东西吗？</Hint>

<Solution>

之所以会这样，是因为这个示例把*对象本身*渲染到了标记中，而不是字符串：`<h1>{person}'s Todos</h1>` 试图渲染整个 `person` 对象！把原始对象作为文本内容会引发错误，因为 React 不知道你希望如何显示它们。

要修复它，把 `<h1>{person}'s Todos</h1>` 改成 `<h1>{person.name}'s Todos</h1>`：

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
      <h1>{person.name}'s Todos</h1>
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

</Solution>

#### 将信息提取到对象中 {/*extract-information-into-an-object*/}

把图片 URL 提取到 `person` 对象中。

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
      <h1>{person.name}'s Todos</h1>
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

<Solution>

把图片 URL 移到名为 `person.imageUrl` 的属性中，然后在 `<img>` 标签里使用花括号读取它：

<Sandpack>

```js
const person = {
  name: 'Gregorio Y. Zara',
  imageUrl: "https://react.dev/images/docs/scientists/7vQD0fPs.jpg",
  theme: {
    backgroundColor: 'black',
    color: 'pink'
  }
};

export default function TodoList() {
  return (
    <div style={person.theme}>
      <h1>{person.name}'s Todos</h1>
      <img
        className="avatar"
        src={person.imageUrl}
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

</Solution>

#### 在 JSX 花括号中写表达式 {/*write-an-expression-inside-jsx-curly-braces*/}

在下面的对象中，完整的图片 URL 被拆分成四部分：基础 URL、`imageId`、`imageSize` 和文件扩展名。

我们希望图片 URL 将这些属性组合起来：基础 URL（始终是 `'https://react.dev/images/docs/scientists/'`）、`imageId`（`'7vQD0fP'`）、`imageSize`（`'s'`）以及文件扩展名（始终是 `'.jpg'`）。不过，`<img>` 标签指定其 `src` 的方式有问题。

你能修复它吗？

<Sandpack>

```js

const baseUrl = 'https://react.dev/images/docs/scientists/';
const person = {
  name: 'Gregorio Y. Zara',
  imageId: '7vQD0fP',
  imageSize: 's',
  theme: {
    backgroundColor: 'black',
    color: 'pink'
  }
};

export default function TodoList() {
  return (
    <div style={person.theme}>
      <h1>{person.name}'s Todos</h1>
      <img
        className="avatar"
        src="{baseUrl}{person.imageId}{person.imageSize}.jpg"
        alt={person.name}
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
.avatar { border-radius: 50%; }
```

</Sandpack>

为了检查你的修复是否有效，试着把 `imageSize` 的值改成 `'b'`。修改后图片应该会重新调整大小。

<Solution>

你可以把它写成 `src={baseUrl + person.imageId + person.imageSize + '.jpg'}`。

1. `{` 打开 JavaScript 表达式
2. `baseUrl + person.imageId + person.imageSize + '.jpg'` 生成正确的 URL 字符串
3. `}` 关闭 JavaScript 表达式

<Sandpack>

```js
const baseUrl = 'https://react.dev/images/docs/scientists/';
const person = {
  name: 'Gregorio Y. Zara',
  imageId: '7vQD0fP',
  imageSize: 's',
  theme: {
    backgroundColor: 'black',
    color: 'pink'
  }
};

export default function TodoList() {
  return (
    <div style={person.theme}>
      <h1>{person.name}'s Todos</h1>
      <img
        className="avatar"
        src={baseUrl + person.imageId + person.imageSize + '.jpg'}
        alt={person.name}
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
.avatar { border-radius: 50%; }
```

</Sandpack>

你也可以把这个表达式移到一个单独的函数中，比如下面的 `getImageUrl`：

<Sandpack>

```js src/App.js
import { getImageUrl } from './utils.js'

const person = {
  name: 'Gregorio Y. Zara',
  imageId: '7vQD0fP',
  imageSize: 's',
  theme: {
    backgroundColor: 'black',
    color: 'pink'
  }
};

export default function TodoList() {
  return (
    <div style={person.theme}>
      <h1>{person.name}'s Todos</h1>
      <img
        className="avatar"
        src={getImageUrl(person)}
        alt={person.name}
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

```js src/utils.js
export function getImageUrl(person) {
  return (
    'https://react.dev/images/docs/scientists/' +
    person.imageId +
    person.imageSize +
    '.jpg'
  );
}
```

```css
body { padding: 0; margin: 0 }
body > div > div { padding: 20px; }
.avatar { border-radius: 50%; }
```

</Sandpack>

变量和函数可以帮助你保持标记简洁！

</Solution>

</Challenges>
