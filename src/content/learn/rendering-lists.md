---
title: 渲染列表
---

<Intro>

你经常会想要从一组数据中显示多个相似的组件。你可以使用 [JavaScript 数组方法](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Array#) 来操作一组数据。在本页中，你将结合 React 使用 [`filter()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/filter) 和 [`map()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map) 来过滤并将你的数据数组转换为组件数组。

</Intro>

<YouWillLearn>

* 如何使用 JavaScript 的 `map()` 从数组中渲染组件
* 如何使用 JavaScript 的 `filter()` 只渲染特定组件
* 何时以及为何使用 React keys

</YouWillLearn>

## 从数组中渲染数据 {/*rendering-data-from-arrays*/}

假设你有一组内容。

```js
<ul>
  <li>Creola Katherine Johnson：数学家</li>
  <li>Mario José Molina-Pasquel Henríquez：化学家</li>
  <li>Mohammad Abdus Salam：物理学家</li>
  <li>Percy Lavon Julian：化学家</li>
  <li>Subrahmanyan Chandrasekhar：天体物理学家</li>
</ul>
```

这些列表项之间唯一的区别是它们的内容，也就是它们的数据。在构建界面时，你经常需要使用不同的数据来显示同一个组件的多个实例：从评论列表到个人资料图片画廊。在这些情况下，你可以将这些数据存储在 JavaScript 对象和数组中，并使用像 [`map()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map) 和 [`filter()`](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Array/filter) 这样的函数从中渲染组件列表。

下面是一个简短示例，说明如何从数组生成项目列表：

1. **把** 数据移动到数组中：

```js
const people = [
  'Creola Katherine Johnson：数学家',
  'Mario José Molina-Pasquel Henríquez：化学家',
  'Mohammad Abdus Salam：物理学家',
  'Percy Lavon Julian：化学家',
  'Subrahmanyan Chandrasekhar：天体物理学家'
];
```

2. 使用 `map()` 将 `people` 成员**映射**到一个新的 JSX 节点数组 `listItems`：

```js
const listItems = people.map(person => <li>{person}</li>);
```

3. **从** 你的组件中**返回**用 `<ul>` 包裹的 `listItems`：

```js
return <ul>{listItems}</ul>;
```

结果如下：

<Sandpack>

```js
const people = [
  'Creola Katherine Johnson：数学家',
  'Mario José Molina-Pasquel Henríquez：化学家',
  'Mohammad Abdus Salam：物理学家',
  'Percy Lavon Julian：化学家',
  'Subrahmanyan Chandrasekhar：天体物理学家'
];

export default function List() {
  const listItems = people.map(person =>
    <li>{person}</li>
  );
  return <ul>{listItems}</ul>;
}
```

```css
li { margin-bottom: 10px; }
```

</Sandpack>

请注意，上面的沙盒显示了一个控制台错误：

<ConsoleBlock level="error">

警告：列表中的每个子项都应该有一个唯一的 "key" 属性。

</ConsoleBlock>

你将在本页后面学习如何修复这个错误。在此之前，让我们先为你的数据添加一些结构。

## 过滤数组中的项目 {/*filtering-arrays-of-items*/}

这组数据可以组织得更加结构化。

```js
const people = [{
  id: 0,
  name: 'Creola Katherine Johnson',
  profession: 'mathematician',
}, {
  id: 1,
  name: 'Mario José Molina-Pasquel Henríquez',
  profession: 'chemist',
}, {
  id: 2,
  name: 'Mohammad Abdus Salam',
  profession: 'physicist',
}, {
  id: 3,
  name: 'Percy Lavon Julian',
  profession: 'chemist',
}, {
  id: 4,
  name: 'Subrahmanyan Chandrasekhar',
  profession: 'astrophysicist',
}];
```

假设你想只显示职业是 `'chemist'` 的人。你可以使用 JavaScript 的 `filter()` 方法只返回这些人。这个方法接收一个项目数组，让它们通过一个“测试”（一个返回 `true` 或 `false` 的函数），并返回一个只包含通过测试（返回 `true`）的项目的新数组。

你只想要 `profession` 是 `'chemist'` 的项目。这个“测试”函数看起来像 `(person) => person.profession === 'chemist'`。下面是如何把它组合起来：

1. 通过在 `people` 上调用 `filter()`，按 `person.profession === 'chemist'` 过滤，**创建** 一个只包含“化学家”的新数组 `chemists`：

```js
const chemists = people.filter(person =>
  person.profession === 'chemist'
);
```

2. 现在对 `chemists` 进行 **map**：

```js {1,13}
const listItems = chemists.map(person =>
  <li>
     <img
       src={getImageUrl(person)}
       alt={person.name}
     />
     <p>
       <b>{person.name}:</b>
       {' ' + person.profession + ' '}
       known for {person.accomplishment}
     </p>
  </li>
);
```

3. 最后，从你的组件中**返回** `listItems`：

```js
return <ul>{listItems}</ul>;
```

<Sandpack>

```js src/App.js
import { people } from './data.js';
import { getImageUrl } from './utils.js';

export default function List() {
  const chemists = people.filter(person =>
    person.profession === 'chemist'
  );
  const listItems = chemists.map(person =>
    <li>
      <img
        src={getImageUrl(person)}
        alt={person.name}
      />
      <p>
        <b>{person.name}:</b>
        {' ' + person.profession + ' '}
        known for {person.accomplishment}
      </p>
    </li>
  );
  return <ul>{listItems}</ul>;
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
  grid-template-columns: auto 1fr;
  gap: 20px;
  align-items: center;
}
img { width: 100px; height: 100px; border-radius: 50%; }
```

</Sandpack>

<Pitfall>

箭头函数会隐式返回 `=>` 后面的表达式，所以你不需要写 `return` 语句：

```js
const listItems = chemists.map(person =>
  <li>...</li> // 隐式返回！
);
```

然而，如果你的 `=>` 后面跟着一个 `{` 花括号，你**必须显式写出** `return`！

```js
const listItems = chemists.map(person => { // 花括号
  return <li>...</li>;
});
```

包含 `=> {` 的箭头函数被称为具有 ["块体"](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions#function_body)。它们允许你编写不止一行代码，但你*必须*自己写 `return` 语句。要是忘了，什么都不会返回！

</Pitfall>

## 使用 `key` 保持列表项顺序 {/*keeping-list-items-in-order-with-key*/}

请注意，上面的所有沙盒在控制台中都显示了一个错误：

<ConsoleBlock level="error">

警告：列表中的每个子项都应该有一个唯一的 "key" 属性。

</ConsoleBlock>

你需要给每个数组项一个 `key`——一个字符串或数字，用来在该数组的其他项中唯一标识它：

```js
<li key={person.id}>...</li>
```

<Note>

直接写在 `map()` 调用中的 JSX 元素总是需要 keys！

</Note>

keys 告诉 React 每个组件对应数组中的哪一项，这样它之后才能把它们匹配起来。如果你的数组项可能移动（例如由于排序）、被插入或被删除，这一点就很重要。一个选择良好的 `key` 能帮助 React 推断到底发生了什么，并对 DOM 树进行正确的更新。

与其动态生成 key，不如把它们包含在你的数据中：

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
        <b>{person.name}</b>
          {' ' + person.profession + ' '}
          known for {person.accomplishment}
      </p>
    </li>
  );
  return <ul>{listItems}</ul>;
}
```

```js src/data.js active
export const people = [{
  id: 0, // 在 JSX 中用作 key
  name: 'Creola Katherine Johnson',
  profession: 'mathematician',
  accomplishment: 'spaceflight calculations',
  imageId: 'MK3eW3A'
}, {
  id: 1, // 在 JSX 中用作 key
  name: 'Mario José Molina-Pasquel Henríquez',
  profession: 'chemist',
  accomplishment: 'discovery of Arctic ozone hole',
  imageId: 'mynHUSa'
}, {
  id: 2, // 在 JSX 中用作 key
  name: 'Mohammad Abdus Salam',
  profession: 'physicist',
  accomplishment: 'electromagnetism theory',
  imageId: 'bE7W1ji'
}, {
  id: 3, // 在 JSX 中用作 key
  name: 'Percy Lavon Julian',
  profession: 'chemist',
  accomplishment: 'pioneering cortisone drugs, steroids and birth control pills',
  imageId: 'IOjWm71'
}, {
  id: 4, // 在 JSX 中用作 key
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
  grid-template-columns: auto 1fr;
  gap: 20px;
  align-items: center;
}
img { width: 100px; height: 100px; border-radius: 50%; }
```

</Sandpack>

<DeepDive>

#### 为每个列表项显示多个 DOM 节点 {/*displaying-several-dom-nodes-for-each-list-item*/}

如果每个项目需要渲染的不只是一个，而是多个 DOM 节点，你该怎么办？

简短的 [`<>...</>` Fragment](/reference/react/Fragment) 语法不允许你传递 key，所以你需要把它们组合到一个 `<div>` 中，或者使用稍长一些且[更明确的 `<Fragment>` 语法：](/reference/react/Fragment#rendering-a-list-of-fragments)

```js
import { Fragment } from 'react';

// ...

const listItems = people.map(person =>
  <Fragment key={person.id}>
    <h1>{person.name}</h1>
    <p>{person.bio}</p>
  </Fragment>
);
```

Fragments 会从 DOM 中消失，因此这会生成一个由 `<h1>`、`<p>`、`<h1>`、`<p>` 等组成的扁平列表。

</DeepDive>

### key 从哪里来 {/*where-to-get-your-key*/}

不同的数据来源会提供不同来源的 keys：

* **来自数据库的数据：** 如果你的数据来自数据库，你可以使用数据库键/ID，它们本身就是唯一的。
* **本地生成的数据：** 如果你的数据是在本地生成并持久化的（例如记事应用中的笔记），在创建项目时使用递增计数器、[`crypto.randomUUID()`](https://developer.mozilla.org/en-US/docs/Web/API/Crypto/randomUUID) 或像 [`uuid`](https://www.npmjs.com/package/uuid) 这样的包。

### keys 的规则 {/*rules-of-keys*/}

* **Keys 在同级元素之间必须唯一。** 但是，在_不同_数组中的 JSX 节点使用相同的 keys 是可以的。
* **Keys 不能改变**，否则它们的作用就失效了！不要在渲染时生成它们。

### 为什么 React 需要 keys？ {/*why-does-react-need-keys*/}

想象一下，如果你桌面上的文件没有名称。取而代之的是，你只能按照它们的顺序来指代它们——第一个文件、第二个文件，依此类推。你可能会慢慢习惯，但一旦你删除了一个文件，事情就会变得混乱。第二个文件会变成第一个文件，第三个文件会变成第二个文件，依此类推。

文件夹中的文件名和数组中的 JSX keys 起着类似的作用。它们让我们能够在同级项之间唯一地识别某一项。一个选择良好的 key 提供的信息比数组中的位置更多。即使由于重新排序导致_位置_发生变化，`key` 也能让 React 在其整个生命周期中识别该项。

<Pitfall>

你可能会想把数组项的索引用作它的 key。事实上，如果你根本不指定 `key`，React 就会使用这个值。但是，如果某个项目被插入、删除，或者数组顺序被重新排列，你渲染项目的顺序就会随着时间而变化。把索引当作 key 往往会导致细微而令人困惑的 bug。

同样，不要动态生成 keys，例如 `key={Math.random()}`。这会导致每次渲染之间 keys 都无法匹配，从而使你的所有组件和 DOM 每次都被重新创建。这不仅很慢，还会丢失列表项中的任何用户输入。相反，请使用基于数据的稳定 ID。

注意，你的组件不会把 `key` 作为 prop 接收到。它只会被 React 自身当作提示使用。如果你的组件需要一个 ID，你必须将其作为单独的 prop 传递：`<Profile key={id} userId={id} />`。

</Pitfall>

<Recap>

在本页中你学到了：

* 如何将数据从组件中移出，并放入数组和对象之类的数据结构中。
* 如何使用 JavaScript 的 `map()` 生成一组相似的组件。
* 如何使用 JavaScript 的 `filter()` 创建过滤后的项目数组。
* 为什么以及如何在集合中的每个组件上设置 `key`，这样即使它们的位置或数据发生变化，React 也能跟踪每个组件。

</Recap>



<Challenges>

#### 将一个列表拆成两个 {/*splitting-a-list-in-two*/}

这个示例展示了一个包含所有人的列表。

把它改成依次显示两个单独的列表：**化学家** 和 **其他所有人。** 和之前一样，你可以通过检查 `person.profession === 'chemist'` 来判断某人是否是化学家。

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
        known for {person.accomplishment}
      </p>
    </li>
  );
  return (
    <article>
      <h1>Scientists</h1>
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
  grid-template-columns: auto 1fr;
  gap: 20px;
  align-items: center;
}
img { width: 100px; height: 100px; border-radius: 50%; }
```

</Sandpack>

<Solution>

你可以使用两次 `filter()`，创建两个独立数组，然后分别对它们进行 `map`：

<Sandpack>

```js src/App.js
import { people } from './data.js';
import { getImageUrl } from './utils.js';

export default function List() {
  const chemists = people.filter(person =>
    person.profession === 'chemist'
  );
  const everyoneElse = people.filter(person =>
    person.profession !== 'chemist'
  );
  return (
    <article>
      <h1>Scientists</h1>
      <h2>Chemists</h2>
      <ul>
        {chemists.map(person =>
          <li key={person.id}>
            <img
              src={getImageUrl(person)}
              alt={person.name}
            />
            <p>
              <b>{person.name}:</b>
              {' ' + person.profession + ' '}
              known for {person.accomplishment}
            </p>
          </li>
        )}
      </ul>
      <h2>Everyone Else</h2>
      <ul>
        {everyoneElse.map(person =>
          <li key={person.id}>
            <img
              src={getImageUrl(person)}
              alt={person.name}
            />
            <p>
              <b>{person.name}:</b>
              {' ' + person.profession + ' '}
              known for {person.accomplishment}
            </p>
          </li>
        )}
      </ul>
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
  grid-template-columns: auto 1fr;
  gap: 20px;
  align-items: center;
}
img { width: 100px; height: 100px; border-radius: 50%; }
```

</Sandpack>

在这个方案中，`map` 调用直接内联放在父 `<ul>` 元素中，不过如果你觉得更易读，也可以为它们引入变量。

渲染出的列表之间仍然有一些重复。你可以进一步把重复部分提取到一个 `<ListSection>` 组件中：

<Sandpack>

```js src/App.js
import { people } from './data.js';
import { getImageUrl } from './utils.js';

function ListSection({ title, people }) {
  return (
    <>
      <h2>{title}</h2>
      <ul>
        {people.map(person =>
          <li key={person.id}>
            <img
              src={getImageUrl(person)}
              alt={person.name}
            />
            <p>
              <b>{person.name}:</b>
              {' ' + person.profession + ' '}
              known for {person.accomplishment}
            </p>
          </li>
        )}
      </ul>
    </>
  );
}

export default function List() {
  const chemists = people.filter(person =>
    person.profession === 'chemist'
  );
  const everyoneElse = people.filter(person =>
    person.profession !== 'chemist'
  );
  return (
    <article>
      <h1>Scientists</h1>
      <ListSection
        title="Chemists"
        people={chemists}
      />
      <ListSection
        title="Everyone Else"
        people={everyoneElse}
      />
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
  grid-template-columns: auto 1fr;
  gap: 20px;
  align-items: center;
}
img { width: 100px; height: 100px; border-radius: 50%; }
```

</Sandpack>

细心的读者可能会注意到，使用两个 `filter` 调用时，我们会两次检查每个人的职业。检查一个属性非常快，所以在这个例子中没问题。如果你的逻辑比这更耗费性能，你可以用一个循环替换这些 `filter` 调用，手动构建数组并且只检查每个人一次。

事实上，如果 `people` 永远不会改变，你甚至可以把这段代码移出组件。从 React 的角度看，重要的只是你最终给它一个 JSX 节点数组。它并不关心你是如何生成这个数组的：

<Sandpack>

```js src/App.js
import { people } from './data.js';
import { getImageUrl } from './utils.js';

let chemists = [];
let everyoneElse = [];
people.forEach(person => {
  if (person.profession === 'chemist') {
    chemists.push(person);
  } else {
    everyoneElse.push(person);
  }
});

function ListSection({ title, people }) {
  return (
    <>
      <h2>{title}</h2>
      <ul>
        {people.map(person =>
          <li key={person.id}>
            <img
              src={getImageUrl(person)}
              alt={person.name}
            />
            <p>
              <b>{person.name}:</b>
              {' ' + person.profession + ' '}
              known for {person.accomplishment}
            </p>
          </li>
        )}
      </ul>
    </>
  );
}

export default function List() {
  return (
    <article>
      <h1>Scientists</h1>
      <ListSection
        title="Chemists"
        people={chemists}
      />
      <ListSection
        title="Everyone Else"
        people={everyoneElse}
      />
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
  grid-template-columns: auto 1fr;
  gap: 20px;
  align-items: center;
}
img { width: 100px; height: 100px; border-radius: 50%; }
```

</Sandpack>

</Solution>

#### 在一个组件中嵌套列表 {/*nested-lists-in-one-component*/}

从这个数组中制作一个食谱列表！对于数组中的每个食谱，把它的名称显示为 `<h2>`，并把它的配料列表放在一个 `<ul>` 中。

<Hint>

这需要嵌套两个不同的 `map` 调用。

</Hint>

<Sandpack>

```js src/App.js
import { recipes } from './data.js';

export default function RecipeList() {
  return (
    <div>
      <h1>Recipes</h1>
    </div>
  );
}
```

```js src/data.js
export const recipes = [{
  id: 'greek-salad',
  name: 'Greek Salad',
  ingredients: ['tomatoes', 'cucumber', 'onion', 'olives', 'feta']
}, {
  id: 'hawaiian-pizza',
  name: 'Hawaiian Pizza',
  ingredients: ['pizza crust', 'pizza sauce', 'mozzarella', 'ham', 'pineapple']
}, {
  id: 'hummus',
  name: 'Hummus',
  ingredients: ['chickpeas', 'olive oil', 'garlic cloves', 'lemon', 'tahini']
}];
```

</Sandpack>

<Solution>

下面是一种可行的做法：

<Sandpack>

```js src/App.js
import { recipes } from './data.js';

export default function RecipeList() {
  return (
    <div>
      <h1>Recipes</h1>
      {recipes.map(recipe =>
        <div key={recipe.id}>
          <h2>{recipe.name}</h2>
          <ul>
            {recipe.ingredients.map(ingredient =>
              <li key={ingredient}>
                {ingredient}
              </li>
            )}
          </ul>
        </div>
      )}
    </div>
  );
}
```

```js src/data.js
export const recipes = [{
  id: 'greek-salad',
  name: 'Greek Salad',
  ingredients: ['tomatoes', 'cucumber', 'onion', 'olives', 'feta']
}, {
  id: 'hawaiian-pizza',
  name: 'Hawaiian Pizza',
  ingredients: ['pizza crust', 'pizza sauce', 'mozzarella', 'ham', 'pineapple']
}, {
  id: 'hummus',
  name: 'Hummus',
  ingredients: ['chickpeas', 'olive oil', 'garlic cloves', 'lemon', 'tahini']
}];
```

</Sandpack>

每个 `recipes` 都已经包含一个 `id` 字段，所以外层循环用它作为 `key`。而配料没有可用于循环的 ID。不过，可以合理地假设同一种配料不会在同一个食谱中重复出现，所以它的名称可以作为 `key`。另外，你也可以修改数据结构添加 ID，或者使用索引作为 `key`（但要注意，不能安全地重新排序配料）。

</Solution>

#### 提取一个列表项组件 {/*extracting-a-list-item-component*/}

这个 `RecipeList` 组件包含两个嵌套的 `map` 调用。为了简化它，可以从中提取一个 `Recipe` 组件，它将接收 `id`、`name` 和 `ingredients` props。外层的 `key` 应该放在哪里，为什么？

<Sandpack>

```js src/App.js
import { recipes } from './data.js';

export default function RecipeList() {
  return (
    <div>
      <h1>Recipes</h1>
      {recipes.map(recipe =>
        <div key={recipe.id}>
          <h2>{recipe.name}</h2>
          <ul>
            {recipe.ingredients.map(ingredient =>
              <li key={ingredient}>
                {ingredient}
              </li>
            )}
          </ul>
        </div>
      )}
    </div>
  );
}
```

```js src/data.js
export const recipes = [{
  id: 'greek-salad',
  name: 'Greek Salad',
  ingredients: ['tomatoes', 'cucumber', 'onion', 'olives', 'feta']
}, {
  id: 'hawaiian-pizza',
  name: 'Hawaiian Pizza',
  ingredients: ['pizza crust', 'pizza sauce', 'mozzarella', 'ham', 'pineapple']
}, {
  id: 'hummus',
  name: 'Hummus',
  ingredients: ['chickpeas', 'olive oil', 'garlic cloves', 'lemon', 'tahini']
}];
```

</Sandpack>

<Solution>

你可以把外层 `map` 的 JSX 复制粘贴到一个新的 `Recipe` 组件里，并返回那段 JSX。然后你可以把 `recipe.name` 改成 `name`，把 `recipe.id` 改成 `id`，等等，并将它们作为 props 传给 `Recipe`：

<Sandpack>

```js
import { recipes } from './data.js';

function Recipe({ id, name, ingredients }) {
  return (
    <div>
      <h2>{name}</h2>
      <ul>
        {ingredients.map(ingredient =>
          <li key={ingredient}>
            {ingredient}
          </li>
        )}
      </ul>
    </div>
  );
}

export default function RecipeList() {
  return (
    <div>
      <h1>Recipes</h1>
      {recipes.map(recipe =>
        <Recipe {...recipe} key={recipe.id} />
      )}
    </div>
  );
}
```

```js src/data.js
export const recipes = [{
  id: 'greek-salad',
  name: 'Greek Salad',
  ingredients: ['tomatoes', 'cucumber', 'onion', 'olives', 'feta']
}, {
  id: 'hawaiian-pizza',
  name: 'Hawaiian Pizza',
  ingredients: ['pizza crust', 'pizza sauce', 'mozzarella', 'ham', 'pineapple']
}, {
  id: 'hummus',
  name: 'Hummus',
  ingredients: ['chickpeas', 'olive oil', 'garlic cloves', 'lemon', 'tahini']
}];
```

</Sandpack>

这里，`<Recipe {...recipe} key={recipe.id} />` 是一种语法快捷写法，意思是“把 `recipe` 对象的所有属性都作为 props 传给 `Recipe` 组件”。你也可以显式写出每个 prop：`<Recipe id={recipe.id} name={recipe.name} ingredients={recipe.ingredients} key={recipe.id} />`。

**请注意，`key` 是指定在 `<Recipe>` 本身上，而不是指定在从 `Recipe` 返回的根 `<div>` 上。** 这是因为这个 `key` 需要直接位于外层数组的上下文中。之前，你有一个 `<div>` 的数组，所以每个 `<div>` 都需要一个 `key`，但现在你有的是一个 `<Recipe>` 的数组。换句话说，当你提取组件时，不要忘记把 `key` 放在你复制和粘贴的 JSX 外面。

</Solution>

#### 带分隔符的列表 {/*list-with-a-separator*/}

这个示例渲染了松尾芭蕉的一首著名俳句，每一行都包在一个 `<p>` 标签中。你的任务是在每个段落之间插入一个 `<hr />` 分隔符。最终结构应该像这样：

```js
<article>
  <p>我写下、擦掉、重写</p>
  <hr />
  <p>再次擦掉，然后</p>
  <hr />
  <p>一朵罂粟花绽放。</p>
</article>
```

一首俳句只有三行，但你的解决方案应该适用于任意行数。注意，`<hr />` 元素只出现在 `<p>` 元素之间，不出现在开头或结尾！

<Sandpack>

```js
const poem = {
  lines: [
    '我写下、擦掉、重写',
    '再次擦掉，然后',
    '一朵罂粟花绽放。'
  ]
};

export default function Poem() {
  return (
    <article>
      {poem.lines.map((line, index) =>
        <p key={index}>
          {line}
        </p>
      )}
    </article>
  );
}
```

```css
body {
  text-align: center;
}
p {
  font-family: Georgia, serif;
  font-size: 20px;
  font-style: italic;
}
hr {
  margin: 0 120px 0 120px;
  border: 1px dashed #45c3d8;
}
```

</Sandpack>

（这是少数可以接受使用索引作为 key 的情况，因为诗句的行永远不会重新排序。）

<Hint>

你需要把 `map` 转换为手动循环，或者使用 Fragment。

</Hint>

<Solution>

你可以写一个手动循环，边生成边把 `<hr />` 和 `<p>...</p>` 插入到输出数组中：

<Sandpack>

```js
const poem = {
  lines: [
    '我写下、擦掉、重写',
    '再次擦掉，然后',
    '一朵罂粟花绽放。'
  ]
};

export default function Poem() {
  let output = [];

  // 填充输出数组
  poem.lines.forEach((line, i) => {
    output.push(
      <hr key={i + '-separator'} />
    );
    output.push(
      <p key={i + '-text'}>
        {line}
      </p>
    );
  });
  // 移除第一个 <hr />
  output.shift();

  return (
    <article>
      {output}
    </article>
  );
}
```

```css
body {
  text-align: center;
}
p {
  font-family: Georgia, serif;
  font-size: 20px;
  font-style: italic;
}
hr {
  margin: 0 120px 0 120px;
  border: 1px dashed #45c3d8;
}
```

</Sandpack>

使用原始行索引作为 `key` 已经不再适用了，因为现在每个分隔符和段落都在同一个数组里。不过，你可以给它们分别加上不同的后缀来提供不同的 key，例如 `key={i + '-text'}`。

另外，你也可以渲染一组包含 `<hr />` 和 `<p>...</p>` 的 Fragments。不过，`<>...</>` 缩写语法不支持传递 keys，所以你必须显式写 `<Fragment>`：

<Sandpack>

```js
import { Fragment } from 'react';

const poem = {
  lines: [
    '我写下、擦掉、重写',
    '再次擦掉，然后',
    '一朵罂粟花绽放。'
  ]
};

export default function Poem() {
  return (
    <article>
      {poem.lines.map((line, i) =>
        <Fragment key={i}>
          {i > 0 && <hr />}
          <p>{line}</p>
        </Fragment>
      )}
    </article>
  );
}
```

```css
body {
  text-align: center;
}
p {
  font-family: Georgia, serif;
  font-size: 20px;
  font-style: italic;
}
hr {
  margin: 0 120px 0 120px;
  border: 1px dashed #45c3d8;
}
```

</Sandpack>

记住，Fragments（通常写作 `<> </>`）可以让你在不添加额外 `<div>` 的情况下组合 JSX 节点！

</Solution>

</Challenges>
