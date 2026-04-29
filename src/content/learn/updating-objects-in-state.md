---
title: 更新状态中的对象
---

<Intro>

状态可以保存任何类型的 JavaScript 值，包括对象。但你不应该直接修改存放在 React 状态中的对象。相反，当你想更新一个对象时，你需要创建一个新的对象（或者创建现有对象的副本），然后将状态设置为使用那个副本。

</Intro>

<YouWillLearn>

- 如何在 React 状态中正确更新对象
- 如何在不修改嵌套对象的情况下更新它
- 什么是不可变性，以及如何不破坏它
- 如何借助 Immer 让对象拷贝更少重复

</YouWillLearn>

## 什么是 mutation？ {/*whats-a-mutation*/}

你可以在状态中存储任何类型的 JavaScript 值。

```js
const [x, setX] = useState(0);
```

到目前为止，你一直在处理数字、字符串和布尔值。这些 JavaScript 值是“不可变的”，意思是不可更改或“只读”。你可以触发重新渲染来_替换_一个值：

```js
setX(5);
```

`x` 状态从 `0` 变成了 `5`，但_数字 `0` 本身_并没有改变。在 JavaScript 中，不可能对数字、字符串和布尔值这类内置原始值做任何修改。

现在考虑状态中的一个对象：

```js
const [position, setPosition] = useState({ x: 0, y: 0 });
```

从技术上讲，修改_对象本身_的内容是可能的。**这叫做 mutation（突变）：**

```js
position.x = 5;
```

然而，虽然 React 状态中的对象从技术上讲是可变的，你应该把它们**当作**不可变的——就像数字、布尔值和字符串一样。不要修改它们，而应该始终替换它们。

## 将状态视为只读 {/*treat-state-as-read-only*/}

换句话说，你应该**把放进状态里的任何 JavaScript 对象都当作只读的。**

这个示例在状态中保存了一个对象，用来表示当前指针位置。红点应该在你触摸或在预览区域内移动光标时跟着移动。但红点却停留在初始位置：

<Sandpack>

```js {expectedErrors: {'react-compiler': [11]}}
import { useState } from 'react';

export default function MovingDot() {
  const [position, setPosition] = useState({
    x: 0,
    y: 0
  });
  return (
    <div
      onPointerMove={e => {
        position.x = e.clientX;
        position.y = e.clientY;
      }}
      style={{
        position: 'relative',
        width: '100vw',
        height: '100vh',
      }}>
      <div style={{
        position: 'absolute',
        backgroundColor: 'red',
        borderRadius: '50%',
        transform: `translate(${position.x}px, ${position.y}px)`,
        left: -10,
        top: -10,
        width: 20,
        height: 20,
      }} />
    </div>
  );
}
```

```css
body { margin: 0; padding: 0; height: 250px; }
```

</Sandpack>

问题出在这段代码上。

```js
onPointerMove={e => {
  position.x = e.clientX;
  position.y = e.clientY;
}}
```

这段代码修改了赋给 `position` 的对象，而这个对象来自[上一次渲染。](/learn/state-as-a-snapshot#rendering-takes-a-snapshot-in-time)但如果不使用状态设置函数，React 根本不知道这个对象已经改变了。所以 React 不会对此做出任何反应。这就像你已经吃完这顿饭之后才想去改变顺序一样。虽然在某些情况下修改状态可以工作，但我们不建议这样做。你应该把在一次渲染中可访问到的状态值视为只读。

要在这种情况下真正[触发重新渲染](/learn/state-as-a-snapshot#setting-state-triggers-renders)，**请创建一个*新*对象并把它传给状态设置函数：**

```js
onPointerMove={e => {
  setPosition({
    x: e.clientX,
    y: e.clientY
  });
}}
```

使用 `setPosition` 时，你是在告诉 React：

* 用这个新对象替换 `position`
* 然后再次渲染这个组件

注意当你触摸或在预览区域悬停时，红点现在会跟随你的指针移动：

<Sandpack>

```js
import { useState } from 'react';

export default function MovingDot() {
  const [position, setPosition] = useState({
    x: 0,
    y: 0
  });
  return (
    <div
      onPointerMove={e => {
        setPosition({
          x: e.clientX,
          y: e.clientY
        });
      }}
      style={{
        position: 'relative',
        width: '100vw',
        height: '100vh',
      }}>
      <div style={{
        position: 'absolute',
        backgroundColor: 'red',
        borderRadius: '50%',
        transform: `translate(${position.x}px, ${position.y}px)`,
        left: -10,
        top: -10,
        width: 20,
        height: 20,
      }} />
    </div>
  );
}
```

```css
body { margin: 0; padding: 0; height: 250px; }
```

</Sandpack>

<DeepDive>

#### 局部 mutation 是可以的 {/*local-mutation-is-fine*/}

像这样的代码会有问题，因为它修改了状态中*已有*的对象：

```js
position.x = e.clientX;
position.y = e.clientY;
```

但像这样的代码**完全没问题**，因为你修改的是一个你*刚刚创建*出来的新对象：

```js
const nextPosition = {};
nextPosition.x = e.clientX;
nextPosition.y = e.clientY;
setPosition(nextPosition);
```

实际上，它完全等价于这样写：

```js
setPosition({
  x: e.clientX,
  y: e.clientY
});
```

mutation 只有在你修改已经在状态中的*已有*对象时才会成为问题。修改你刚创建的对象是可以的，因为*还没有任何其他代码引用它。*修改它不会意外影响依赖它的东西。这叫做“局部 mutation”。你甚至可以在[渲染时](/learn/keeping-components-pure#local-mutation-your-components-little-secret)进行局部 mutation。非常方便，而且完全没问题！

</DeepDive>

## 使用展开语法复制对象 {/*copying-objects-with-the-spread-syntax*/}

在前面的示例中，`position` 对象总是根据当前光标位置重新创建。但很多时候，你会希望把*已有*数据作为你正在创建的新对象的一部分。例如，你可能只想更新表单中的*一个*字段，同时保留其他所有字段的先前值。

这些输入框无法工作，因为 `onChange` 处理函数修改了状态：

<Sandpack>

```js {expectedErrors: {'react-compiler': [11, 15, 19]}}
import { useState } from 'react';

export default function Form() {
  const [person, setPerson] = useState({
    firstName: 'Barbara',
    lastName: 'Hepworth',
    email: 'bhepworth@sculpture.com'
  });

  function handleFirstNameChange(e) {
    person.firstName = e.target.value;
  }

  function handleLastNameChange(e) {
    person.lastName = e.target.value;
  }

  function handleEmailChange(e) {
    person.email = e.target.value;
  }

  return (
    <>
      <label>
        名字：
        <input
          value={person.firstName}
          onChange={handleFirstNameChange}
        />
      </label>
      <label>
        姓氏：
        <input
          value={person.lastName}
          onChange={handleLastNameChange}
        />
      </label>
      <label>
        电子邮件：
        <input
          value={person.email}
          onChange={handleEmailChange}
        />
      </label>
      <p>
        {person.firstName}{' '}
        {person.lastName}{' '}
        ({person.email})
      </p>
    </>
  );
}
```

```css
label { display: block; }
input { margin-left: 5px; margin-bottom: 5px; }
```

</Sandpack>

例如，这一行修改了来自过去某次渲染的状态：

```js
person.firstName = e.target.value;
```

要获得你想要的行为，可靠的方法是创建一个新对象并把它传给 `setPerson`。但在这里，你还想**把现有数据复制进去**，因为只有一个字段发生了变化：

```js
setPerson({
  firstName: e.target.value, // 输入框中的新名字
  lastName: person.lastName,
  email: person.email
});
```

你可以使用 `...` [对象展开](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax#spread_in_object_literals)语法，这样就不必分别复制每个属性。

```js
setPerson({
  ...person, // 复制旧字段
  firstName: e.target.value // 但覆盖这一项
});
```

现在这个表单可以工作了！

注意你并没有为每个输入框都声明一个单独的状态变量。对于大型表单，把所有数据分组存放在一个对象里非常方便——只要你正确更新它就行！

<Sandpack>

```js
import { useState } from 'react';

export default function Form() {
  const [person, setPerson] = useState({
    firstName: 'Barbara',
    lastName: 'Hepworth',
    email: 'bhepworth@sculpture.com'
  });

  function handleFirstNameChange(e) {
    setPerson({
      ...person,
      firstName: e.target.value
    });
  }

  function handleLastNameChange(e) {
    setPerson({
      ...person,
      lastName: e.target.value
    });
  }

  function handleEmailChange(e) {
    setPerson({
      ...person,
      email: e.target.value
    });
  }

  return (
    <>
      <label>
        名字：
        <input
          value={person.firstName}
          onChange={handleFirstNameChange}
        />
      </label>
      <label>
        姓氏：
        <input
          value={person.lastName}
          onChange={handleLastNameChange}
        />
      </label>
      <label>
        电子邮件：
        <input
          value={person.email}
          onChange={handleEmailChange}
        />
      </label>
      <p>
        {person.firstName}{' '}
        {person.lastName}{' '}
        ({person.email})
      </p>
    </>
  );
}
```

```css
label { display: block; }
input { margin-left: 5px; margin-bottom: 5px; }
```

</Sandpack>

请注意，`...` 展开语法是“浅层”的——它只会复制一层内容。这使它很快，但也意味着如果你想更新嵌套属性，就必须多次使用它。

<DeepDive>

#### 对多个字段使用单个事件处理函数 {/*using-a-single-event-handler-for-multiple-fields*/}

你也可以在对象定义中使用 `[` 和 `]` 大括号来指定一个动态名称的属性。下面是同一个示例，但用一个事件处理函数代替了三个不同的处理函数：

<Sandpack>

```js
import { useState } from 'react';

export default function Form() {
  const [person, setPerson] = useState({
    firstName: 'Barbara',
    lastName: 'Hepworth',
    email: 'bhepworth@sculpture.com'
  });

  function handleChange(e) {
    setPerson({
      ...person,
      [e.target.name]: e.target.value
    });
  }

  return (
    <>
      <label>
        名字：
        <input
          name="firstName"
          value={person.firstName}
          onChange={handleChange}
        />
      </label>
      <label>
        姓氏：
        <input
          name="lastName"
          value={person.lastName}
          onChange={handleChange}
        />
      </label>
      <label>
        电子邮件：
        <input
          name="email"
          value={person.email}
          onChange={handleChange}
        />
      </label>
      <p>
        {person.firstName}{' '}
        {person.lastName}{' '}
        ({person.email})
      </p>
    </>
  );
}
```

```css
label { display: block; }
input { margin-left: 5px; margin-bottom: 5px; }
```

</Sandpack>

这里，`e.target.name` 指的是赋给 `<input>` DOM 元素的 `name` 属性。

</DeepDive>

## 更新嵌套对象 {/*updating-a-nested-object*/}

考虑一个这样的嵌套对象结构：

```js
const [person, setPerson] = useState({
  name: 'Niki de Saint Phalle',
  artwork: {
    title: 'Blue Nana',
    city: 'Hamburg',
    image: 'https://react.dev/images/docs/scientists/Sd1AgUOm.jpg',
  }
});
```

如果你想更新 `person.artwork.city`，用 mutation 的方式很容易做到：

```js
person.artwork.city = 'New Delhi';
```

但在 React 中，你应该把 state 当作不可变的！为了改变 `city`，你首先需要生成新的 `artwork` 对象（预先填入前一个对象中的数据），然后再生成指向新 `artwork` 的新的 `person` 对象：

```js
const nextArtwork = { ...person.artwork, city: 'New Delhi' };
const nextPerson = { ...person, artwork: nextArtwork };
setPerson(nextPerson);
```

或者，写成一次函数调用：

```js
setPerson({
  ...person, // 复制其他字段
  artwork: { // 但替换 artwork
    ...person.artwork, // 使用相同的那个
    city: 'New Delhi' // 但城市改为 New Delhi！
  }
});
```

这会有点啰嗦，但在很多情况下都很好用：

<Sandpack>

```js
import { useState } from 'react';

export default function Form() {
  const [person, setPerson] = useState({
    name: 'Niki de Saint Phalle',
    artwork: {
      title: 'Blue Nana',
      city: 'Hamburg',
      image: 'https://react.dev/images/docs/scientists/Sd1AgUOm.jpg',
    }
  });

  function handleNameChange(e) {
    setPerson({
      ...person,
      name: e.target.value
    });
  }

  function handleTitleChange(e) {
    setPerson({
      ...person,
      artwork: {
        ...person.artwork,
        title: e.target.value
      }
    });
  }

  function handleCityChange(e) {
    setPerson({
      ...person,
      artwork: {
        ...person.artwork,
        city: e.target.value
      }
    });
  }

  function handleImageChange(e) {
    setPerson({
      ...person,
      artwork: {
        ...person.artwork,
        image: e.target.value
      }
    });
  }

  return (
    <>
      <label>
        名字：
        <input
          value={person.name}
          onChange={handleNameChange}
        />
      </label>
      <label>
        标题：
        <input
          value={person.artwork.title}
          onChange={handleTitleChange}
        />
      </label>
      <label>
        城市：
        <input
          value={person.artwork.city}
          onChange={handleCityChange}
        />
      </label>
      <label>
        图片：
        <input
          value={person.artwork.image}
          onChange={handleImageChange}
        />
      </label>
      <p>
        <i>{person.artwork.title}</i>
        {' 作者 '}
        {person.name}
        <br />
        （位于 {person.artwork.city}）
      </p>
      <img
        src={person.artwork.image}
        alt={person.artwork.title}
      />
    </>
  );
}
```

```css
label { display: block; }
input { margin-left: 5px; margin-bottom: 5px; }
img { width: 200px; height: 200px; }
```

</Sandpack>

<DeepDive>

#### 对象其实并不是真的嵌套 {/*objects-are-not-really-nested*/}

像这样的对象在代码中看起来是“嵌套”的：

```js
let obj = {
  name: 'Niki de Saint Phalle',
  artwork: {
    title: 'Blue Nana',
    city: 'Hamburg',
    image: 'https://react.dev/images/docs/scientists/Sd1AgUOm.jpg',
  }
};
```

然而，“嵌套”并不是理解对象行为的准确方式。当代码执行时，并不存在所谓“嵌套”的对象。你实际上看到的是两个不同的对象：

```js
let obj1 = {
  title: 'Blue Nana',
  city: 'Hamburg',
  image: 'https://react.dev/images/docs/scientists/Sd1AgUOm.jpg',
};

let obj2 = {
  name: 'Niki de Saint Phalle',
  artwork: obj1
};
```

`obj1` 并不“在” `obj2` 里面。比如，`obj3` 也可以“指向” `obj1`：

```js
let obj1 = {
  title: 'Blue Nana',
  city: 'Hamburg',
  image: 'https://react.dev/images/docs/scientists/Sd1AgUOm.jpg',
};

let obj2 = {
  name: 'Niki de Saint Phalle',
  artwork: obj1
};

let obj3 = {
  name: 'Copycat',
  artwork: obj1
};
```

如果你去修改 `obj3.artwork.city`，它会同时影响 `obj2.artwork.city` 和 `obj1.city`。这是因为 `obj3.artwork`、`obj2.artwork` 和 `obj1` 是同一个对象。当你把对象想成“嵌套”的时候，这一点很难看出来。实际上，它们是彼此独立的对象，只是通过属性“指向”彼此。

</DeepDive>

### 使用 Immer 编写简洁的更新逻辑 {/*write-concise-update-logic-with-immer*/}

如果你的 state 嵌套很深，你可以考虑[把它扁平化。](/learn/choosing-the-state-structure#avoid-deeply-nested-state)但如果你不想改变 state 结构，你可能会更喜欢一个替代深层展开语法的快捷方式。[Immer](https://github.com/immerjs/use-immer) 是一个很受欢迎的库，它允许你使用方便但看起来像 mutation 的语法来编写代码，并负责为你生成副本。使用 Immer 时，你写出来的代码看起来像是在“违反规则”并修改对象：

```js
updatePerson(draft => {
  draft.artwork.city = 'Lagos';
});
```

但和普通的 mutation 不同，它不会覆盖过去的 state！

<DeepDive>

#### Immer 是如何工作的？ {/*how-does-immer-work*/}

Immer 提供的 `draft` 是一种特殊类型的对象，叫做 [Proxy](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy)，它会“记录”你对它所做的操作。这就是为什么你可以随心所欲地修改它！在内部，Immer 会找出 `draft` 中哪些部分发生了变化，并生成一个包含你修改内容的全新对象。

</DeepDive>

要试用 Immer：

1. 运行 `npm install use-immer` 将 Immer 添加为依赖
2. 然后将 `import { useState } from 'react'` 替换为 `import { useImmer } from 'use-immer'`

下面是将上面的示例转换为 Immer 的版本：

<Sandpack>

```js
import { useImmer } from 'use-immer';

export default function Form() {
  const [person, updatePerson] = useImmer({
    name: 'Niki de Saint Phalle',
    artwork: {
      title: 'Blue Nana',
      city: 'Hamburg',
      image: 'https://react.dev/images/docs/scientists/Sd1AgUOm.jpg',
    }
  });

  function handleNameChange(e) {
    updatePerson(draft => {
      draft.name = e.target.value;
    });
  }

  function handleTitleChange(e) {
    updatePerson(draft => {
      draft.artwork.title = e.target.value;
    });
  }

  function handleCityChange(e) {
    updatePerson(draft => {
      draft.artwork.city = e.target.value;
    });
  }

  function handleImageChange(e) {
    updatePerson(draft => {
      draft.artwork.image = e.target.value;
    });
  }

  return (
    <>
      <label>
        名字：
        <input
          value={person.name}
          onChange={handleNameChange}
        />
      </label>
      <label>
        标题：
        <input
          value={person.artwork.title}
          onChange={handleTitleChange}
        />
      </label>
      <label>
        城市：
        <input
          value={person.artwork.city}
          onChange={handleCityChange}
        />
      </label>
      <label>
        图片：
        <input
          value={person.artwork.image}
          onChange={handleImageChange}
        />
      </label>
      <p>
        <i>{person.artwork.title}</i>
        {' 作者 '}
        {person.name}
        <br />
        （位于 {person.artwork.city}）
      </p>
      <img
        src={person.artwork.image}
        alt={person.artwork.title}
      />
    </>
  );
}
```

```json package.json
{
  "dependencies": {
    "immer": "1.7.3",
    "react": "latest",
    "react-dom": "latest",
    "react-scripts": "latest",
    "use-immer": "0.5.1"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject"
  }
}
```

```css
label { display: block; }
input { margin-left: 5px; margin-bottom: 5px; }
img { width: 200px; height: 200px; }
```

</Sandpack>

注意这些事件处理函数变得多么简洁了。你可以在同一个组件中随意混用 `useState` 和 `useImmer`。Immer 是保持更新处理函数简洁的绝佳方式，尤其是在 state 有嵌套、并且对象复制会导致重复代码时。

<DeepDive>

#### 为什么不建议在 React 中直接修改 state？ {/*why-is-mutating-state-not-recommended-in-react*/}

原因有几个：

* **调试：** 如果你使用 `console.log` 且不修改 state，那么过去的日志不会被更新的 state 变化覆盖。这样你就可以清楚地看到两次渲染之间 state 是如何变化的。
* **优化：** 常见的 React [优化策略](/reference/react/memo) 依赖于在前后 props 或 state 相同时跳过工作。如果你从不修改 state，就很容易检查是否发生了变化。如果 `prevObj === obj`，你就能确定它内部没有任何东西发生变化。
* **新特性：** 我们正在构建的 React 新特性依赖于将 state [当作快照来处理。](/learn/state-as-a-snapshot)如果你在修改 state 的历史版本，这可能会阻止你使用这些新特性。
* **需求变化：** 一些应用功能，比如实现撤销/重做、显示变更历史，或者让用户把表单重置为更早的值，在没有 mutation 的情况下更容易实现。这是因为你可以将过去的 state 副本保存在内存中，并在合适的时候重用它们。如果你一开始采用的是修改式写法，那么之后再添加这类功能会很困难。
* **实现更简单：** 因为 React 不依赖 mutation，它不需要对你的对象做任何特殊处理。它不需要劫持它们的属性、总是把它们包装成 Proxy，或者像许多“响应式”方案那样在初始化时做其他工作。这也是为什么 React 允许你把任何对象放入 state——不管它有多大——而不会带来额外的性能或正确性陷阱。

在实践中，你常常可以在 React 中“侥幸”直接修改 state，但我们强烈建议你不要这样做，这样你才能使用按照这种思路开发的新 React 特性。未来的贡献者，甚至未来的你自己，都会感谢你！

</DeepDive>

<Recap>

* 把 React 中的所有 state 都视为不可变的。
* 当你把对象存入 state 时，修改它们不会触发重新渲染，而且会改变之前渲染“快照”中的 state。
* 不要直接修改对象，而是创建它的一个*新*版本，并通过把 state 设置为它来触发重新渲染。
* 你可以使用 `{...obj, something: 'newValue'}` 这种对象展开语法来创建对象副本。
* 展开语法是浅拷贝：它只复制一层深度。
* 要更新一个嵌套对象，你需要从你要更新的位置一路向上都创建副本。
* 为了减少重复的复制代码，可以使用 Immer。

</Recap>



<Challenges>

#### 修复错误的 state 更新 {/*fix-incorrect-state-updates*/}

这个表单有几个 bug。点击几次增加分数的按钮。注意分数并没有增加。然后编辑名字，注意分数突然“追上”了你的修改。最后，编辑姓氏，注意分数完全消失了。

你的任务是修复所有这些 bug。修复时，请解释每个 bug 发生的原因。

<Sandpack>

```js {expectedErrors: {'react-compiler': [11]}}
import { useState } from 'react';

export default function Scoreboard() {
  const [player, setPlayer] = useState({
    firstName: 'Ranjani',
    lastName: 'Shettar',
    score: 10,
  });

  function handlePlusClick() {
    player.score++;
  }

  function handleFirstNameChange(e) {
    setPlayer({
      ...player,
      firstName: e.target.value,
    });
  }

  function handleLastNameChange(e) {
    setPlayer({
      lastName: e.target.value
    });
  }

  return (
    <>
      <label>
        分数：<b>{player.score}</b>
        {' '}
        <button onClick={handlePlusClick}>
          +1
        </button>
      </label>
      <label>
        名字：
        <input
          value={player.firstName}
          onChange={handleFirstNameChange}
        />
      </label>
      <label>
        姓氏：
        <input
          value={player.lastName}
          onChange={handleLastNameChange}
        />
      </label>
    </>
  );
}
```

```css
label { display: block; margin-bottom: 10px; }
input { margin-left: 5px; margin-bottom: 5px; }
```

</Sandpack>

<Solution>

下面是修复了这两个 bug 的版本：

<Sandpack>

```js
import { useState } from 'react';

export default function Scoreboard() {
  const [player, setPlayer] = useState({
    firstName: 'Ranjani',
    lastName: 'Shettar',
    score: 10,
  });

  function handlePlusClick() {
    setPlayer({
      ...player,
      score: player.score + 1,
    });
  }

  function handleFirstNameChange(e) {
    setPlayer({
      ...player,
      firstName: e.target.value,
    });
  }

  function handleLastNameChange(e) {
    setPlayer({
      ...player,
      lastName: e.target.value
    });
  }

  return (
    <>
      <label>
        分数：<b>{player.score}</b>
        {' '}
        <button onClick={handlePlusClick}>
          +1
        </button>
      </label>
      <label>
        名字：
        <input
          value={player.firstName}
          onChange={handleFirstNameChange}
        />
      </label>
      <label>
        姓氏：
        <input
          value={player.lastName}
          onChange={handleLastNameChange}
        />
      </label>
    </>
  );
}
```

```css
label { display: block; }
input { margin-left: 5px; margin-bottom: 5px; }
```

</Sandpack>

`handlePlusClick` 的问题在于它修改了 `player` 对象。因此，React 不知道有理由重新渲染，也就没有更新屏幕上的分数。这就是为什么当你编辑名字时，state 被更新了，触发了一次重新渲染，而这次重新渲染也顺带更新了屏幕上的分数。

`handleLastNameChange` 的问题在于，它没有把现有的 `...player` 字段复制到新对象中。这就是为什么你编辑姓氏后分数会丢失。

</Solution>

#### 查找并修复 mutation {/*find-and-fix-the-mutation*/}

有一个可拖动的盒子，背景是静态的。你可以通过选择框来改变盒子的颜色。

但这里有一个 bug。如果你先移动盒子，然后再改变它的颜色，背景（本不应该移动！）会“跳”到盒子的位置。但这不应该发生：`Background` 的 `position` prop 被设置为 `initialPosition`，也就是 `{ x: 0, y: 0 }`。为什么更改颜色后背景会移动？

找出 bug 并修复它。

<Hint>

如果有东西发生了意外变化，那一定有 mutation。找到 `App.js` 中的 mutation 并修复它。

</Hint>

<Sandpack>

```js {expectedErrors: {'react-compiler': [17]}} src/App.js
import { useState } from 'react';
import Background from './Background.js';
import Box from './Box.js';

const initialPosition = {
  x: 0,
  y: 0
};

export default function Canvas() {
  const [shape, setShape] = useState({
    color: 'orange',
    position: initialPosition
  });

  function handleMove(dx, dy) {
    shape.position.x += dx;
    shape.position.y += dy;
  }

  function handleColorChange(e) {
    setShape({
      ...shape,
      color: e.target.value
    });
  }

  return (
    <>
      <select
        value={shape.color}
        onChange={handleColorChange}
      >
        <option value="orange">orange</option>
        <option value="lightpink">lightpink</option>
        <option value="aliceblue">aliceblue</option>
      </select>
      <Background
        position={initialPosition}
      />
      <Box
        color={shape.color}
        position={shape.position}
        onMove={handleMove}
      >
        拖动我！
      </Box>
    </>
  );
}
```

```js src/Box.js
import { useState } from 'react';

export default function Box({
  children,
  color,
  position,
  onMove
}) {
  const [
    lastCoordinates,
    setLastCoordinates
  ] = useState(null);

  function handlePointerDown(e) {
    e.target.setPointerCapture(e.pointerId);
    setLastCoordinates({
      x: e.clientX,
      y: e.clientY,
    });
  }

  function handlePointerMove(e) {
    if (lastCoordinates) {
      setLastCoordinates({
        x: e.clientX,
        y: e.clientY,
      });
      const dx = e.clientX - lastCoordinates.x;
      const dy = e.clientY - lastCoordinates.y;
      onMove(dx, dy);
    }
  }

  function handlePointerUp(e) {
    setLastCoordinates(null);
  }

  return (
    <div
      onPointerDown={handlePointerDown}
      onPointerMove={handlePointerMove}
      onPointerUp={handlePointerUp}
      style={{
        width: 100,
        height: 100,
        cursor: 'grab',
        backgroundColor: color,
        position: 'absolute',
        border: '1px solid black',
        display: 'flex',
        justifyContent: 'center',
        alignItems: 'center',
        transform: `translate(
          ${position.x}px,
          ${position.y}px
        )`,
      }}
    >{children}</div>
  );
}
```

```js src/Background.js
export default function Background({
  position
}) {
  return (
    <div style={{
      position: 'absolute',
      transform: `translate(
        ${position.x}px,
        ${position.y}px
      )`,
      width: 250,
      height: 250,
      backgroundColor: 'rgba(200, 200, 0, 0.2)',
    }} />
  );
};
```

```css
body { height: 280px; }
select { margin-bottom: 10px; }
```

</Sandpack>

<Solution>

问题出在 `handleMove` 里的 mutation。它修改了 `shape.position`，但那是 `initialPosition` 指向的同一个对象。这就是为什么形状和背景都会移动。（由于这是 mutation，变化不会立刻反映到屏幕上，直到某个无关的更新——颜色变化——触发重新渲染。）

修复方法是移除 `handleMove` 中的 mutation，并使用展开语法复制 shape。注意 `+=` 是 mutation，所以你需要把它改写成普通的 `+` 运算。

<Sandpack>

```js src/App.js
import { useState } from 'react';
import Background from './Background.js';
import Box from './Box.js';

const initialPosition = {
  x: 0,
  y: 0
};

export default function Canvas() {
  const [shape, setShape] = useState({
    color: 'orange',
    position: initialPosition
  });

  function handleMove(dx, dy) {
    setShape({
      ...shape,
      position: {
        x: shape.position.x + dx,
        y: shape.position.y + dy,
      }
    });
  }

  function handleColorChange(e) {
    setShape({
      ...shape,
      color: e.target.value
    });
  }

  return (
    <>
      <select
        value={shape.color}
        onChange={handleColorChange}
      >
        <option value="orange">orange</option>
        <option value="lightpink">lightpink</option>
        <option value="aliceblue">aliceblue</option>
      </select>
      <Background
        position={initialPosition}
      />
      <Box
        color={shape.color}
        position={shape.position}
        onMove={handleMove}
      >
        拖动我！
      </Box>
    </>
  );
}
```

```js src/Box.js
import { useState } from 'react';

export default function Box({
  children,
  color,
  position,
  onMove
}) {
  const [
    lastCoordinates,
    setLastCoordinates
  ] = useState(null);

  function handlePointerDown(e) {
    e.target.setPointerCapture(e.pointerId);
    setLastCoordinates({
      x: e.clientX,
      y: e.clientY,
    });
  }

  function handlePointerMove(e) {
    if (lastCoordinates) {
      setLastCoordinates({
        x: e.clientX,
        y: e.clientY,
      });
      const dx = e.clientX - lastCoordinates.x;
      const dy = e.clientY - lastCoordinates.y;
      onMove(dx, dy);
    }
  }

  function handlePointerUp(e) {
    setLastCoordinates(null);
  }

  return (
    <div
      onPointerDown={handlePointerDown}
      onPointerMove={handlePointerMove}
      onPointerUp={handlePointerUp}
      style={{
        width: 100,
        height: 100,
        cursor: 'grab',
        backgroundColor: color,
        position: 'absolute',
        border: '1px solid black',
        display: 'flex',
        justifyContent: 'center',
        alignItems: 'center',
        transform: `translate(
          ${position.x}px,
          ${position.y}px
        )`,
      }}
    >{children}</div>
  );
}
```

```js src/Background.js
export default function Background({
  position
}) {
  return (
    <div style={{
      position: 'absolute',
      transform: `translate(
        ${position.x}px,
        ${position.y}px
      )`,
      width: 250,
      height: 250,
      backgroundColor: 'rgba(200, 200, 0, 0.2)',
    }} />
  );
};
```

```css
body { height: 280px; }
select { margin-bottom: 10px; }
```

</Sandpack>

</Solution>

#### 使用 Immer 更新对象 {/*update-an-object-with-immer*/}

这是前一个挑战中相同的有 bug 示例。这次，请使用 Immer 来修复 mutation。为方便起见，`useImmer` 已经被导入，所以你只需要把 `shape` state 变量改成使用它即可。

<Sandpack>

```js {expectedErrors: {'react-compiler': [18]}} src/App.js
import { useState } from 'react';
import { useImmer } from 'use-immer';
import Background from './Background.js';
import Box from './Box.js';

const initialPosition = {
  x: 0,
  y: 0
};

export default function Canvas() {
  const [shape, setShape] = useState({
    color: 'orange',
    position: initialPosition
  });

  function handleMove(dx, dy) {
    shape.position.x += dx;
    shape.position.y += dy;
  }

  function handleColorChange(e) {
    setShape({
      ...shape,
      color: e.target.value
    });
  }

  return (
    <>
      <select
        value={shape.color}
        onChange={handleColorChange}
      >
        <option value="orange">orange</option>
        <option value="lightpink">lightpink</option>
        <option value="aliceblue">aliceblue</option>
      </select>
      <Background
        position={initialPosition}
      />
      <Box
        color={shape.color}
        position={shape.position}
        onMove={handleMove}
      >
        拖动我！
      </Box>
    </>
  );
}
```

```js src/Box.js
import { useState } from 'react';

export default function Box({
  children,
  color,
  position,
  onMove
}) {
  const [
    lastCoordinates,
    setLastCoordinates
  ] = useState(null);

  function handlePointerDown(e) {
    e.target.setPointerCapture(e.pointerId);
    setLastCoordinates({
      x: e.clientX,
      y: e.clientY,
    });
  }

  function handlePointerMove(e) {
    if (lastCoordinates) {
      setLastCoordinates({
        x: e.clientX,
        y: e.clientY,
      });
      const dx = e.clientX - lastCoordinates.x;
      const dy = e.clientY - lastCoordinates.y;
      onMove(dx, dy);
    }
  }

  function handlePointerUp(e) {
    setLastCoordinates(null);
  }

  return (
    <div
      onPointerDown={handlePointerDown}
      onPointerMove={handlePointerMove}
      onPointerUp={handlePointerUp}
      style={{
        width: 100,
        height: 100,
        cursor: 'grab',
        backgroundColor: color,
        position: 'absolute',
        border: '1px solid black',
        display: 'flex',
        justifyContent: 'center',
        alignItems: 'center',
        transform: `translate(
          ${position.x}px,
          ${position.y}px
        )`,
      }}
    >{children}</div>
  );
}
```

```js src/Background.js
export default function Background({
  position
}) {
  return (
    <div style={{
      position: 'absolute',
      transform: `translate(
        ${position.x}px,
        ${position.y}px
      )`,
      width: 250,
      height: 250,
      backgroundColor: 'rgba(200, 200, 0, 0.2)',
    }} />
  );
};
```

```css
body { height: 280px; }
select { margin-bottom: 10px; }
```

```json package.json
{
  "dependencies": {
    "immer": "1.7.3",
    "react": "latest",
    "react-dom": "latest",
    "react-scripts": "latest",
    "use-immer": "0.5.1"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject"
  }
}
```

</Sandpack>

<Solution>

这是使用 Immer 重写后的解法。注意事件处理函数是以修改式风格编写的，但 bug 没有出现。这是因为在底层，Immer 从不修改现有对象。

<Sandpack>

```js src/App.js
import { useImmer } from 'use-immer';
import Background from './Background.js';
import Box from './Box.js';

const initialPosition = {
  x: 0,
  y: 0
};

export default function Canvas() {
  const [shape, updateShape] = useImmer({
    color: 'orange',
    position: initialPosition
  });

  function handleMove(dx, dy) {
    updateShape(draft => {
      draft.position.x += dx;
      draft.position.y += dy;
    });
  }

  function handleColorChange(e) {
    updateShape(draft => {
      draft.color = e.target.value;
    });
  }

  return (
    <>
      <select
        value={shape.color}
        onChange={handleColorChange}
      >
        <option value="orange">orange</option>
        <option value="lightpink">lightpink</option>
        <option value="aliceblue">aliceblue</option>
      </select>
      <Background
        position={initialPosition}
      />
      <Box
        color={shape.color}
        position={shape.position}
        onMove={handleMove}
      >
        拖动我！
      </Box>
    </>
  );
}
```

```js src/Box.js
import { useState } from 'react';

export default function Box({
  children,
  color,
  position,
  onMove
}) {
  const [
    lastCoordinates,
    setLastCoordinates
  ] = useState(null);

  function handlePointerDown(e) {
    e.target.setPointerCapture(e.pointerId);
    setLastCoordinates({
      x: e.clientX,
      y: e.clientY,
    });
  }

  function handlePointerMove(e) {
    if (lastCoordinates) {
      setLastCoordinates({
        x: e.clientX,
        y: e.clientY,
      });
      const dx = e.clientX - lastCoordinates.x;
      const dy = e.clientY - lastCoordinates.y;
      onMove(dx, dy);
    }
  }

  function handlePointerUp(e) {
    setLastCoordinates(null);
  }

  return (
    <div
      onPointerDown={handlePointerDown}
      onPointerMove={handlePointerMove}
      onPointerUp={handlePointerUp}
      style={{
        width: 100,
        height: 100,
        cursor: 'grab',
        backgroundColor: color,
        position: 'absolute',
        border: '1px solid black',
        display: 'flex',
        justifyContent: 'center',
        alignItems: 'center',
        transform: `translate(
          ${position.x}px,
          ${position.y}px
        )`,
      }}
    >{children}</div>
  );
}
```

```js src/Background.js
export default function Background({
  position
}) {
  return (
    <div style={{
      position: 'absolute',
      transform: `translate(
        ${position.x}px,
        ${position.y}px
      )`,
      width: 250,
      height: 250,
      backgroundColor: 'rgba(200, 200, 0, 0.2)',
    }} />
  );
};
```

```css
body { height: 280px; }
select { margin-bottom: 10px; }
```

```json package.json
{
  "dependencies": {
    "immer": "1.7.3",
    "react": "latest",
    "react-dom": "latest",
    "react-scripts": "latest",
    "use-immer": "0.5.1"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject"
  }
}
```

</Sandpack>

</Solution>

</Challenges>
