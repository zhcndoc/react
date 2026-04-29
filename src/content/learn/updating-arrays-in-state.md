---
title: 在状态中更新数组
---

<Intro>

数组在 JavaScript 中是可变的，但当你把它们存储在状态中时，应该把它们当作不可变的。和对象一样，当你想更新存储在状态中的数组时，你需要创建一个新的数组（或者复制现有数组），然后将状态设置为使用这个新数组。

</Intro>

<YouWillLearn>

- 如何在 React 状态中的数组里添加、删除或更改项目
- 如何更新数组中的对象
- 如何使用 Immer 让数组复制变得不那么重复

</YouWillLearn>

## 更新数组而不发生突变 {/*updating-arrays-without-mutation*/}

在 JavaScript 中，数组只是另一种对象。[和对象一样](/learn/updating-objects-in-state)，**你应该把 React 状态中的数组视为只读。** 这意味着你不应该像 `arr[0] = 'bird'` 这样重新赋值数组中的项，也不应该使用会修改数组的方法，比如 `push()` 和 `pop()`。

相反，每当你想更新一个数组时，都应该把一个*新*数组传递给状态设置函数。为此，你可以通过调用不改变原数组的方法（例如 `filter()` 和 `map()`），从状态中的原始数组创建一个新数组。然后你就可以将状态设置为得到的新数组。

下面是常见数组操作的参考表。当处理 React 状态中的数组时，你需要避免左侧的这些方法，而更倾向于右侧的方法：

|           | 避免（会修改数组）                  | 更推荐（返回新数组）                                                |
| --------- | ----------------------------------- | ------------------------------------------------------------------- |
| 添加      | `push`, `unshift`                   | `concat`, `[...arr]` 扩展语法 ([示例](#adding-to-an-array))         |
| 删除      | `pop`, `shift`, `splice`            | `filter`, `slice` ([示例](#removing-from-an-array))                |
| 替换      | `splice`, `arr[i] = ...` 赋值       | `map` ([示例](#replacing-items-in-an-array))                       |
| 排序      | `reverse`, `sort`                   | 先复制数组 ([示例](#making-other-changes-to-an-array))              |

或者，你也可以[使用 Immer](#write-concise-update-logic-with-immer)，这样你就能同时使用两列中的方法。

<Pitfall>

不幸的是，[`slice`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/slice) 和 [`splice`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/splice) 的名字很相似，但它们非常不同：

* `slice` 可以让你复制数组或其中的一部分。
* `splice` 会**修改**数组（用于插入或删除项目）。

在 React 中，你会更常用 `slice`（没有 `p`！），因为你不想修改状态中的对象或数组。[更新对象](/learn/updating-objects-in-state) 解释了什么是突变，以及为什么不建议在状态中使用它。

</Pitfall>

### 向数组中添加内容 {/*adding-to-an-array*/}

`push()` 会修改数组，这不是你想要的：

<Sandpack>

```js
import { useState } from 'react';

let nextId = 0;

export default function List() {
  const [name, setName] = useState('');
  const [artists, setArtists] = useState([]);

  return (
    <>
      <h1>鼓舞人心的雕塑家：</h1>
      <input
        value={name}
        onChange={e => setName(e.target.value)}
      />
      <button onClick={() => {
        artists.push({
          id: nextId++,
          name: name,
        });
      }}>添加</button>
      <ul>
        {artists.map(artist => (
          <li key={artist.id}>{artist.name}</li>
        ))}
      </ul>
    </>
  );
}
```

```css
button { margin-left: 5px; }
```

</Sandpack>

相反，创建一个包含现有项目*以及*末尾新项目的*新*数组。有多种方法可以做到这一点，但最简单的方法是使用 `...` [数组展开](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax#spread_in_array_literals)语法：

```js
setArtists( // 替换状态
  [ // 为一个新数组
    ...artists, // 它包含所有旧项目
    { id: nextId++, name: name } // 以及末尾的一个新项目
  ]
);
```

现在它可以正确工作了：

<Sandpack>

```js
import { useState } from 'react';

let nextId = 0;

export default function List() {
  const [name, setName] = useState('');
  const [artists, setArtists] = useState([]);

  return (
    <>
      <h1>鼓舞人心的雕塑家：</h1>
      <input
        value={name}
        onChange={e => setName(e.target.value)}
      />
      <button onClick={() => {
        setArtists([
          ...artists,
          { id: nextId++, name: name }
        ]);
      }}>添加</button>
      <ul>
        {artists.map(artist => (
          <li key={artist.id}>{artist.name}</li>
        ))}
      </ul>
    </>
  );
}
```

```css
button { margin-left: 5px; }
```

</Sandpack>

数组展开语法还允许你通过把项目放在原始 `...artists` 的*前面*来在开头插入一个项目：

```js
setArtists([
  { id: nextId++, name: name },
  ...artists // 把旧项目放到末尾
]);
```

这样，展开语法既可以通过在数组末尾添加内容来实现 `push()` 的效果，也可以通过在数组开头添加内容来实现 `unshift()` 的效果。在上面的沙盒里试试吧！

### 从数组中删除内容 {/*removing-from-an-array*/}

从数组中移除项目最简单的方法是*把它过滤掉*。换句话说，你将生成一个不包含该项目的新数组。为此，请使用 `filter` 方法，例如：

<Sandpack>

```js
import { useState } from 'react';

let initialArtists = [
  { id: 0, name: 'Marta Colvin Andrade' },
  { id: 1, name: 'Lamidi Olonade Fakeye'},
  { id: 2, name: 'Louise Nevelson'},
];

export default function List() {
  const [artists, setArtists] = useState(
    initialArtists
  );

  return (
    <>
      <h1>鼓舞人心的雕塑家：</h1>
      <ul>
        {artists.map(artist => (
          <li key={artist.id}>
            {artist.name}{' '}
            <button onClick={() => {
              setArtists(
                artists.filter(a =>
                  a.id !== artist.id
                )
              );
            }}>
              删除
            </button>
          </li>
        ))}
      </ul>
    </>
  );
}
```

</Sandpack>

点击几次“删除”按钮，看看它的点击处理函数。

```js
setArtists(
  artists.filter(a => a.id !== artist.id)
);
```

这里，`artists.filter(a => a.id !== artist.id)` 的意思是“创建一个数组，由那些 ID 与 `artist.id` 不同的 `artists` 组成”。换句话说，每个艺术家的“删除”按钮都会把该艺术家过滤出数组，然后请求用结果数组重新渲染。注意，`filter` 不会修改原始数组。

### 转换数组 {/*transforming-an-array*/}

如果你想更改数组中的部分或全部项目，可以使用 `map()` 创建一个**新**数组。你传给 `map` 的函数可以根据每个项目的数据或索引（或两者）决定如何处理该项目。

在这个例子中，一个数组保存了两个圆和一个正方形的坐标。当你按下按钮时，它只会把圆向下移动 50 像素。它通过使用 `map()` 生成一个新的数据数组来实现这一点：

<Sandpack>

```js
import { useState } from 'react';

let initialShapes = [
  { id: 0, type: 'circle', x: 50, y: 100 },
  { id: 1, type: 'square', x: 150, y: 100 },
  { id: 2, type: 'circle', x: 250, y: 100 },
];

export default function ShapeEditor() {
  const [shapes, setShapes] = useState(
    initialShapes
  );

  function handleClick() {
    const nextShapes = shapes.map(shape => {
      if (shape.type === 'square') {
        // 没有变化
        return shape;
      } else {
        // 返回一个向下 50px 的新圆
        return {
          ...shape,
          y: shape.y + 50,
        };
      }
    });
    // 使用新数组重新渲染
    setShapes(nextShapes);
  }

  return (
    <>
      <button onClick={handleClick}>
        向下移动圆形！
      </button>
      {shapes.map(shape => (
        <div
          key={shape.id}
          style={{
          background: 'purple',
          position: 'absolute',
          left: shape.x,
          top: shape.y,
          borderRadius:
            shape.type === 'circle'
              ? '50%' : '',
          width: 20,
          height: 20,
        }} />
      ))}
    </>
  );
}
```

```css
body { height: 300px; }
```

</Sandpack>

### 替换数组中的项目 {/*replacing-items-in-an-array*/}

在数组中替换一个或多个项目尤其常见。像 `arr[0] = 'bird'` 这样的赋值会修改原始数组，所以你同样应该为此使用 `map`。

要替换一个项目，可以使用 `map` 创建一个新数组。在你的 `map` 调用中，你会在第二个参数中得到项目索引。利用它来决定返回原始项目（第一个参数）还是其他内容：

<Sandpack>

```js
import { useState } from 'react';

let initialCounters = [
  0, 0, 0
];

export default function CounterList() {
  const [counters, setCounters] = useState(
    initialCounters
  );

  function handleIncrementClick(index) {
    const nextCounters = counters.map((c, i) => {
      if (i === index) {
        // 增加被点击的计数器
        return c + 1;
      } else {
        // 其余的都没有改变
        return c;
      }
    });
    setCounters(nextCounters);
  }

  return (
    <ul>
      {counters.map((counter, i) => (
        <li key={i}>
          {counter}
          <button onClick={() => {
            handleIncrementClick(i);
          }}>+1</button>
        </li>
      ))}
    </ul>
  );
}
```

```css
button { margin: 5px; }
```

</Sandpack>

### 插入到数组中 {/*inserting-into-an-array*/}

有时，你可能想在一个既不在开头也不在末尾的特定位置插入项目。为此，你可以将 `...` 数组展开语法与 `slice()` 方法一起使用。`slice()` 方法可以让你切出数组的一“片”。要插入一个项目，你需要创建一个数组：先展开插入点之前的切片，然后是新项目，最后是原始数组的其余部分。

在这个例子中，Insert 按钮总是在索引 `1` 处插入：

<Sandpack>

```js
import { useState } from 'react';

let nextId = 3;
const initialArtists = [
  { id: 0, name: 'Marta Colvin Andrade' },
  { id: 1, name: 'Lamidi Olonade Fakeye'},
  { id: 2, name: 'Louise Nevelson'},
];

export default function List() {
  const [name, setName] = useState('');
  const [artists, setArtists] = useState(
    initialArtists
  );

  function handleClick() {
    const insertAt = 1; // 可以是任意索引
    const nextArtists = [
      // 插入点之前的项目：
      ...artists.slice(0, insertAt),
      // 新项目：
      { id: nextId++, name: name },
      // 插入点之后的项目：
      ...artists.slice(insertAt)
    ];
    setArtists(nextArtists);
    setName('');
  }

  return (
    <>
      <h1>鼓舞人心的雕塑家：</h1>
      <input
        value={name}
        onChange={e => setName(e.target.value)}
      />
      <button onClick={handleClick}>
        插入
      </button>
      <ul>
        {artists.map(artist => (
          <li key={artist.id}>{artist.name}</li>
        ))}
      </ul>
    </>
  );
}
```

```css
button { margin-left: 5px; }
```

</Sandpack>

### 对数组进行其他更改 {/*making-other-changes-to-an-array*/}

有些事情你无法仅靠展开语法以及像 `map()` 和 `filter()` 这样的非变异方法来完成。例如，你可能想反转或排序一个数组。JavaScript 的 `reverse()` 和 `sort()` 方法会修改原始数组，所以你不能直接使用它们。

**不过，你可以先复制数组，然后再对它进行修改。**

例如：

<Sandpack>

```js
import { useState } from 'react';

const initialList = [
  { id: 0, title: 'Big Bellies' },
  { id: 1, title: 'Lunar Landscape' },
  { id: 2, title: 'Terracotta Army' },
];

export default function List() {
  const [list, setList] = useState(initialList);

  function handleClick() {
    const nextList = [...list];
    nextList.reverse();
    setList(nextList);
  }

  return (
    <>
      <button onClick={handleClick}>
        反转
      </button>
      <ul>
        {list.map(artwork => (
          <li key={artwork.id}>{artwork.title}</li>
        ))}
      </ul>
    </>
  );
}
```

</Sandpack>

这里，你先使用 `[...list]` 展开语法创建原始数组的副本。既然你已经有了副本，就可以使用像 `nextList.reverse()` 或 `nextList.sort()` 这样的会修改数组的方法，甚至可以通过 `nextList[0] = "something"` 逐项赋值。

不过，**即使你复制了一个数组，也不能直接修改其中已有的项目。** 这是因为复制是浅拷贝——新数组仍然会包含与原数组相同的项目。所以如果你修改了复制数组中的某个对象，就等于修改了现有状态。例如，下面这样的代码就是有问题的：

```js
const nextList = [...list];
nextList[0].seen = true; // 问题：修改了 list[0]
setList(nextList);
```

尽管 `nextList` 和 `list` 是两个不同的数组，**`nextList[0]` 和 `list[0]` 指向的是同一个对象。** 所以当你改变 `nextList[0].seen` 时，也同时改变了 `list[0].seen`。这是一种状态突变，你应该避免！你可以用类似于[更新嵌套的 JavaScript 对象](/learn/updating-objects-in-state#updating-a-nested-object)的方式来解决这个问题——复制你想要更改的单个项目，而不是直接修改它们。下面是做法。

## 更新数组中的对象 {/*updating-objects-inside-arrays*/}

对象其实并不 _真的_ 位于数组“内部”。它们在代码中看起来像是“在里面”，但数组中的每个对象都是一个独立的值，而数组只是“指向”它。正因如此，当你修改像 `list[0]` 这样的嵌套字段时要格外小心。别人的艺术品列表可能也指向数组中的同一个元素！

**当更新嵌套状态时，你需要从你想要更新的那个位置开始创建副本，一直创建到顶层。** 我们来看看这是如何工作的。

在这个例子中，两个独立的艺术品列表有着相同的初始状态。它们本应彼此隔离，但由于一次突变，它们的状态意外地被共享了，在一个列表里勾选复选框会影响另一个列表：

<Sandpack>

```js
import { useState } from 'react';

let nextId = 3;
const initialList = [
  { id: 0, title: 'Big Bellies', seen: false },
  { id: 1, title: 'Lunar Landscape', seen: false },
  { id: 2, title: 'Terracotta Army', seen: true },
];

export default function BucketList() {
  const [myList, setMyList] = useState(initialList);
  const [yourList, setYourList] = useState(
    initialList
  );

  function handleToggleMyList(artworkId, nextSeen) {
    const myNextList = [...myList];
    const artwork = myNextList.find(
      a => a.id === artworkId
    );
    artwork.seen = nextSeen;
    setMyList(myNextList);
  }

  function handleToggleYourList(artworkId, nextSeen) {
    const yourNextList = [...yourList];
    const artwork = yourNextList.find(
      a => a.id === artworkId
    );
    artwork.seen = nextSeen;
    setYourList(yourNextList);
  }

  return (
    <>
      <h1>艺术待办清单</h1>
      <h2>我想看的艺术作品列表：</h2>
      <ItemList
        artworks={myList}
        onToggle={handleToggleMyList} />
      <h2>你想看的艺术作品列表：</h2>
      <ItemList
        artworks={yourList}
        onToggle={handleToggleYourList} />
    </>
  );
}

function ItemList({ artworks, onToggle }) {
  return (
    <ul>
      {artworks.map(artwork => (
        <li key={artwork.id}>
          <label>
            <input
              type="checkbox"
              checked={artwork.seen}
              onChange={e => {
                onToggle(
                  artwork.id,
                  e.target.checked
                );
              }}
            />
            {artwork.title}
          </label>
        </li>
      ))}
    </ul>
  );
}
```

</Sandpack>

问题出在这样的代码里：

```js
const myNextList = [...myList];
const artwork = myNextList.find(a => a.id === artworkId);
artwork.seen = nextSeen; // 问题：修改了现有项
setMyList(myNextList);
```

虽然 `myNextList` 数组本身是新的，但其中的 *项目本身* 和原来的 `myList` 数组里的是同一个对象。所以修改 `artwork.seen` 实际上修改的是 *原始* 的 artwork 项。这个 artwork 项也在 `yourList` 中，于是就引发了这个 bug。这样的 bug 可能很难一下子想明白，但幸运的是，只要避免直接修改状态，它们就会消失。

**你可以使用 `map` 用更新后的版本替换旧项，而无需修改原对象。**

```js
setMyList(myList.map(artwork => {
  if (artwork.id === artworkId) {
    // 创建一个带有修改的新对象
    return { ...artwork, seen: nextSeen };
  } else {
    // 没有变化
    return artwork;
  }
}));
```

这里的 `...` 是对象展开语法，用于[创建对象副本。](/learn/updating-objects-in-state#copying-objects-with-the-spread-syntax)

采用这种方式后，现有状态项都没有被修改，bug 也就修复了：

<Sandpack>

```js
import { useState } from 'react';

let nextId = 3;
const initialList = [
  { id: 0, title: 'Big Bellies', seen: false },
  { id: 1, title: 'Lunar Landscape', seen: false },
  { id: 2, title: 'Terracotta Army', seen: true },
];

export default function BucketList() {
  const [myList, setMyList] = useState(initialList);
  const [yourList, setYourList] = useState(
    initialList
  );

  function handleToggleMyList(artworkId, nextSeen) {
    setMyList(myList.map(artwork => {
      if (artwork.id === artworkId) {
        // 创建一个带有修改的新对象
        return { ...artwork, seen: nextSeen };
      } else {
        // 没有变化
        return artwork;
      }
    }));
  }

  function handleToggleYourList(artworkId, nextSeen) {
    setYourList(yourList.map(artwork => {
      if (artwork.id === artworkId) {
        // 创建一个带有修改的新对象
        return { ...artwork, seen: nextSeen };
      } else {
        // 没有变化
        return artwork;
      }
    }));
  }

  return (
    <>
      <h1>艺术待办清单</h1>
      <h2>我想看的艺术作品列表：</h2>
      <ItemList
        artworks={myList}
        onToggle={handleToggleMyList} />
      <h2>你想看的艺术作品列表：</h2>
      <ItemList
        artworks={yourList}
        onToggle={handleToggleYourList} />
    </>
  );
}

function ItemList({ artworks, onToggle }) {
  return (
    <ul>
      {artworks.map(artwork => (
        <li key={artwork.id}>
          <label>
            <input
              type="checkbox"
              checked={artwork.seen}
              onChange={e => {
                onToggle(
                  artwork.id,
                  e.target.checked
                );
              }}
            />
            {artwork.title}
          </label>
        </li>
      ))}
    </ul>
  );
}
```

</Sandpack>

一般来说，**你只应该修改刚刚创建的对象。** 如果你是在插入一个*新*的艺术作品，你可以修改它，但如果你处理的是已经存在于状态中的内容，就需要先创建副本。

### 使用 Immer 编写简洁的更新逻辑 {/*write-concise-update-logic-with-immer*/}

在不修改原对象的情况下更新嵌套数组，可能会有点重复。和[对象一样](/learn/updating-objects-in-state#write-concise-update-logic-with-immer)：

- 一般来说，你不需要把状态更新到超过两三层深。如果你的状态对象非常深，你可能需要[以不同方式重构它们](/learn/choosing-the-state-structure#avoid-deeply-nested-state)，让它们变成扁平结构。
- 如果你不想改变状态结构，你可能更倾向于使用 [Immer](https://github.com/immerjs/use-immer)，它允许你使用方便但看似可变的语法来编写代码，并负责为你生成副本。

下面是使用 Immer 重写后的艺术待办清单示例：

<Sandpack>

```js
import { useState } from 'react';
import { useImmer } from 'use-immer';

let nextId = 3;
const initialList = [
  { id: 0, title: 'Big Bellies', seen: false },
  { id: 1, title: 'Lunar Landscape', seen: false },
  { id: 2, title: 'Terracotta Army', seen: true },
];

export default function BucketList() {
  const [myList, updateMyList] = useImmer(
    initialList
  );
  const [yourList, updateYourList] = useImmer(
    initialList
  );

  function handleToggleMyList(id, nextSeen) {
    updateMyList(draft => {
      const artwork = draft.find(a =>
        a.id === id
      );
      artwork.seen = nextSeen;
    });
  }

  function handleToggleYourList(artworkId, nextSeen) {
    updateYourList(draft => {
      const artwork = draft.find(a =>
        a.id === artworkId
      );
      artwork.seen = nextSeen;
    });
  }

  return (
    <>
      <h1>艺术待办清单</h1>
      <h2>我想看的艺术作品列表：</h2>
      <ItemList
        artworks={myList}
        onToggle={handleToggleMyList} />
      <h2>你想看的艺术作品列表：</h2>
      <ItemList
        artworks={yourList}
        onToggle={handleToggleYourList} />
    </>
  );
}

function ItemList({ artworks, onToggle }) {
  return (
    <ul>
      {artworks.map(artwork => (
        <li key={artwork.id}>
          <label>
            <input
              type="checkbox"
              checked={artwork.seen}
              onChange={e => {
                onToggle(
                  artwork.id,
                  e.target.checked
                );
              }}
            />
            {artwork.title}
          </label>
        </li>
      ))}
    </ul>
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

</Sandpack>

注意在 Immer 中，**像 `artwork.seen = nextSeen` 这样的修改现在是可以的：**

```js
updateMyTodos(draft => {
  const artwork = draft.find(a => a.id === artworkId);
  artwork.seen = nextSeen;
});
```

这是因为你修改的不是 _原始_ 状态，而是 Immer 提供的一个特殊 `draft` 对象。同样地，你也可以对 `draft` 的内容使用像 `push()` 和 `pop()` 这样的可变方法。

在内部，Immer 会根据你对 `draft` 所做的更改，从头开始构建下一个状态。这样一来，你的事件处理函数就能保持非常简洁，同时又不会修改状态。

<Recap>

- 你可以把数组放进状态里，但不能修改它们。
- 不要直接修改数组，而是创建它的一个 *新* 版本，并把状态更新为这个新版本。
- 你可以使用 `[...arr, newItem]` 这种数组展开语法来创建包含新项的数组。
- 你可以使用 `filter()` 和 `map()` 来创建经过过滤或转换的新数组。
- 你可以使用 Immer 来保持代码简洁。

</Recap>



<Challenges>

#### 更新购物车中的一项 {/*update-an-item-in-the-shopping-cart*/}

补全 `handleIncreaseClick` 的逻辑，以便按下 "+" 时增加对应的数量：

<Sandpack>

```js
import { useState } from 'react';

const initialProducts = [{
  id: 0,
  name: 'Baklava',
  count: 1,
}, {
  id: 1,
  name: 'Cheese',
  count: 5,
}, {
  id: 2,
  name: 'Spaghetti',
  count: 2,
}];

export default function ShoppingCart() {
  const [
    products,
    setProducts
  ] = useState(initialProducts)

  function handleIncreaseClick(productId) {

  }

  return (
    <ul>
      {products.map(product => (
        <li key={product.id}>
          {product.name}
          {' '}
          (<b>{product.count}</b>)
          <button onClick={() => {
            handleIncreaseClick(product.id);
          }}>
            +
          </button>
        </li>
      ))}
    </ul>
  );
}
```

```css
button { margin: 5px; }
```

</Sandpack>

<Solution>

你可以使用 `map` 函数创建一个新数组，然后使用 `...` 对象展开语法为新数组中发生变化的对象创建副本：

<Sandpack>

```js
import { useState } from 'react';

const initialProducts = [{
  id: 0,
  name: 'Baklava',
  count: 1,
}, {
  id: 1,
  name: 'Cheese',
  count: 5,
}, {
  id: 2,
  name: 'Spaghetti',
  count: 2,
}];

export default function ShoppingCart() {
  const [
    products,
    setProducts
  ] = useState(initialProducts)

  function handleIncreaseClick(productId) {
    setProducts(products.map(product => {
      if (product.id === productId) {
        return {
          ...product,
          count: product.count + 1
        };
      } else {
        return product;
      }
    }))
  }

  return (
    <ul>
      {products.map(product => (
        <li key={product.id}>
          {product.name}
          {' '}
          (<b>{product.count}</b>)
          <button onClick={() => {
            handleIncreaseClick(product.id);
          }}>
            +
          </button>
        </li>
      ))}
    </ul>
  );
}
```

```css
button { margin: 5px; }
```

</Sandpack>

</Solution>

#### 从购物车中移除一项 {/*remove-an-item-from-the-shopping-cart*/}

这个购物车有一个可用的 "+" 按钮，但 "–" 按钮没有任何作用。你需要给它添加一个事件处理函数，这样按下它时会减少对应商品的 `count`。如果在 `count` 为 1 时按下 "–"，则该商品应自动从购物车中移除。确保它绝不会显示 0。

<Sandpack>

```js
import { useState } from 'react';

const initialProducts = [{
  id: 0,
  name: 'Baklava',
  count: 1,
}, {
  id: 1,
  name: 'Cheese',
  count: 5,
}, {
  id: 2,
  name: 'Spaghetti',
  count: 2,
}];

export default function ShoppingCart() {
  const [
    products,
    setProducts
  ] = useState(initialProducts)

  function handleIncreaseClick(productId) {
    setProducts(products.map(product => {
      if (product.id === productId) {
        return {
          ...product,
          count: product.count + 1
        };
      } else {
        return product;
      }
    }))
  }

  return (
    <ul>
      {products.map(product => (
        <li key={product.id}>
          {product.name}
          {' '}
          (<b>{product.count}</b>)
          <button onClick={() => {
            handleIncreaseClick(product.id);
          }}>
            +
          </button>
          <button>
            –
          </button>
        </li>
      ))}
    </ul>
  );
}
```

```css
button { margin: 5px; }
```

</Sandpack>

<Solution>

你可以先使用 `map` 生成一个新数组，然后使用 `filter` 移除 `count` 设为 `0` 的商品：

<Sandpack>

```js
import { useState } from 'react';

const initialProducts = [{
  id: 0,
  name: 'Baklava',
  count: 1,
}, {
  id: 1,
  name: 'Cheese',
  count: 5,
}, {
  id: 2,
  name: 'Spaghetti',
  count: 2,
}];

export default function ShoppingCart() {
  const [
    products,
    setProducts
  ] = useState(initialProducts)

  function handleIncreaseClick(productId) {
    setProducts(products.map(product => {
      if (product.id === productId) {
        return {
          ...product,
          count: product.count + 1
        };
      } else {
        return product;
      }
    }))
  }

  function handleDecreaseClick(productId) {
    let nextProducts = products.map(product => {
      if (product.id === productId) {
        return {
          ...product,
          count: product.count - 1
        };
      } else {
        return product;
      }
    });
    nextProducts = nextProducts.filter(p =>
      p.count > 0
    );
    setProducts(nextProducts)
  }

  return (
    <ul>
      {products.map(product => (
        <li key={product.id}>
          {product.name}
          {' '}
          (<b>{product.count}</b>)
          <button onClick={() => {
            handleIncreaseClick(product.id);
          }}>
            +
          </button>
          <button onClick={() => {
            handleDecreaseClick(product.id);
          }}>
            –
          </button>
        </li>
      ))}
    </ul>
  );
}
```

```css
button { margin: 5px; }
```

</Sandpack>

</Solution>

#### 使用非突变方法修复突变 {/*fix-the-mutations-using-non-mutative-methods*/}

在这个示例中，`App.js` 中所有事件处理函数都使用了突变。因此，编辑和删除待办事项都无法工作。请将 `handleAddTodo`、`handleChangeTodo` 和 `handleDeleteTodo` 重写为使用非突变方法：

<Sandpack>

```js src/App.js
import { useState } from 'react';
import AddTodo from './AddTodo.js';
import TaskList from './TaskList.js';

let nextId = 3;
const initialTodos = [
  { id: 0, title: 'Buy milk', done: true },
  { id: 1, title: 'Eat tacos', done: false },
  { id: 2, title: 'Brew tea', done: false },
];

export default function TaskApp() {
  const [todos, setTodos] = useState(
    initialTodos
  );

  function handleAddTodo(title) {
    todos.push({
      id: nextId++,
      title: title,
      done: false
    });
  }

  function handleChangeTodo(nextTodo) {
    const todo = todos.find(t =>
      t.id === nextTodo.id
    );
    todo.title = nextTodo.title;
    todo.done = nextTodo.done;
  }

  function handleDeleteTodo(todoId) {
    const index = todos.findIndex(t =>
      t.id === todoId
    );
    todos.splice(index, 1);
  }

  return (
    <>
      <AddTodo
        onAddTodo={handleAddTodo}
      />
      <TaskList
        todos={todos}
        onChangeTodo={handleChangeTodo}
        onDeleteTodo={handleDeleteTodo}
      />
    </>
  );
}
```

```js src/AddTodo.js
import { useState } from 'react';

export default function AddTodo({ onAddTodo }) {
  const [title, setTitle] = useState('');
  return (
    <>
      <input
        placeholder="Add todo"
        value={title}
        onChange={e => setTitle(e.target.value)}
      />
      <button onClick={() => {
        setTitle('');
        onAddTodo(title);
      }}>Add</button>
    </>
  )
}
```

```js src/TaskList.js
import { useState } from 'react';

export default function TaskList({
  todos,
  onChangeTodo,
  onDeleteTodo
}) {
  return (
    <ul>
      {todos.map(todo => (
        <li key={todo.id}>
          <Task
            todo={todo}
            onChange={onChangeTodo}
            onDelete={onDeleteTodo}
          />
        </li>
      ))}
    </ul>
  );
}

function Task({ todo, onChange, onDelete }) {
  const [isEditing, setIsEditing] = useState(false);
  let todoContent;
  if (isEditing) {
    todoContent = (
      <>
        <input
          value={todo.title}
          onChange={e => {
            onChange({
              ...todo,
              title: e.target.value
            });
          }} />
        <button onClick={() => setIsEditing(false)}>
          Save
        </button>
      </>
    );
  } else {
    todoContent = (
      <>
        {todo.title}
        <button onClick={() => setIsEditing(true)}>
          Edit
        </button>
      </>
    );
  }
  return (
    <label>
      <input
        type="checkbox"
        checked={todo.done}
        onChange={e => {
          onChange({
            ...todo,
            done: e.target.checked
          });
        }}
      />
      {todoContent}
      <button onClick={() => onDelete(todo.id)}>
        Delete
      </button>
    </label>
  );
}
```

```css
button { margin: 5px; }
li { list-style-type: none; }
ul, li { margin: 0; padding: 0; }
```

</Sandpack>

<Solution>

在 `handleAddTodo` 中，你可以使用数组展开语法。在 `handleChangeTodo` 中，你可以使用 `map` 创建新数组。在 `handleDeleteTodo` 中，你可以使用 `filter` 创建新数组。现在列表可以正常工作了：

<Sandpack>

```js src/App.js
import { useState } from 'react';
import AddTodo from './AddTodo.js';
import TaskList from './TaskList.js';

let nextId = 3;
const initialTodos = [
  { id: 0, title: 'Buy milk', done: true },
  { id: 1, title: 'Eat tacos', done: false },
  { id: 2, title: 'Brew tea', done: false },
];

export default function TaskApp() {
  const [todos, setTodos] = useState(
    initialTodos
  );

  function handleAddTodo(title) {
    setTodos([
      ...todos,
      {
        id: nextId++,
        title: title,
        done: false
      }
    ]);
  }

  function handleChangeTodo(nextTodo) {
    setTodos(todos.map(t => {
      if (t.id === nextTodo.id) {
        return nextTodo;
      } else {
        return t;
      }
    }));
  }

  function handleDeleteTodo(todoId) {
    setTodos(
      todos.filter(t => t.id !== todoId)
    );
  }

  return (
    <>
      <AddTodo
        onAddTodo={handleAddTodo}
      />
      <TaskList
        todos={todos}
        onChangeTodo={handleChangeTodo}
        onDeleteTodo={handleDeleteTodo}
      />
    </>
  );
}
```

```js src/AddTodo.js
import { useState } from 'react';

export default function AddTodo({ onAddTodo }) {
  const [title, setTitle] = useState('');
  return (
    <>
      <input
        placeholder="Add todo"
        value={title}
        onChange={e => setTitle(e.target.value)}
      />
      <button onClick={() => {
        setTitle('');
        onAddTodo(title);
      }}>Add</button>
    </>
  )
}
```

```js src/TaskList.js
import { useState } from 'react';

export default function TaskList({
  todos,
  onChangeTodo,
  onDeleteTodo
}) {
  return (
    <ul>
      {todos.map(todo => (
        <li key={todo.id}>
          <Task
            todo={todo}
            onChange={onChangeTodo}
            onDelete={onDeleteTodo}
          />
        </li>
      ))}
    </ul>
  );
}

function Task({ todo, onChange, onDelete }) {
  const [isEditing, setIsEditing] = useState(false);
  let todoContent;
  if (isEditing) {
    todoContent = (
      <>
        <input
          value={todo.title}
          onChange={e => {
            onChange({
              ...todo,
              title: e.target.value
            });
          }} />
        <button onClick={() => setIsEditing(false)}>
          Save
        </button>
      </>
    );
  } else {
    todoContent = (
      <>
        {todo.title}
        <button onClick={() => setIsEditing(true)}>
          Edit
        </button>
      </>
    );
  }
  return (
    <label>
      <input
        type="checkbox"
        checked={todo.done}
        onChange={e => {
          onChange({
            ...todo,
            done: e.target.checked
          });
        }}
      />
      {todoContent}
      <button onClick={() => onDelete(todo.id)}>
        Delete
      </button>
    </label>
  );
}
```

```css
button { margin: 5px; }
li { list-style-type: none; }
ul, li { margin: 0; padding: 0; }
```

</Sandpack>

</Solution>


#### 使用 Immer 修复突变 {/*fix-the-mutations-using-immer*/}

这和上一个挑战中的示例是一样的。这次，请使用 Immer 来修复这些突变。为方便起见，`useImmer` 已经导入好了，所以你只需要把 `todos` 状态变量改为使用它。

<Sandpack>

```js src/App.js
import { useState } from 'react';
import { useImmer } from 'use-immer';
import AddTodo from './AddTodo.js';
import TaskList from './TaskList.js';

let nextId = 3;
const initialTodos = [
  { id: 0, title: 'Buy milk', done: true },
  { id: 1, title: 'Eat tacos', done: false },
  { id: 2, title: 'Brew tea', done: false },
];

export default function TaskApp() {
  const [todos, setTodos] = useState(
    initialTodos
  );

  function handleAddTodo(title) {
    todos.push({
      id: nextId++,
      title: title,
      done: false
    });
  }

  function handleChangeTodo(nextTodo) {
    const todo = todos.find(t =>
      t.id === nextTodo.id
    );
    todo.title = nextTodo.title;
    todo.done = nextTodo.done;
  }

  function handleDeleteTodo(todoId) {
    const index = todos.findIndex(t =>
      t.id === todoId
    );
    todos.splice(index, 1);
  }

  return (
    <>
      <AddTodo
        onAddTodo={handleAddTodo}
      />
      <TaskList
        todos={todos}
        onChangeTodo={handleChangeTodo}
        onDeleteTodo={handleDeleteTodo}
      />
    </>
  );
}
```

```js src/AddTodo.js
import { useState } from 'react';

export default function AddTodo({ onAddTodo }) {
  const [title, setTitle] = useState('');
  return (
    <>
      <input
        placeholder="Add todo"
        value={title}
        onChange={e => setTitle(e.target.value)}
      />
      <button onClick={() => {
        setTitle('');
        onAddTodo(title);
      }}>Add</button>
    </>
  )
}
```

```js src/TaskList.js
import { useState } from 'react';

export default function TaskList({
  todos,
  onChangeTodo,
  onDeleteTodo
}) {
  return (
    <ul>
      {todos.map(todo => (
        <li key={todo.id}>
          <Task
            todo={todo}
            onChange={onChangeTodo}
            onDelete={onDeleteTodo}
          />
        </li>
      ))}
    </ul>
  );
}

function Task({ todo, onChange, onDelete }) {
  const [isEditing, setIsEditing] = useState(false);
  let todoContent;
  if (isEditing) {
    todoContent = (
      <>
        <input
          value={todo.title}
          onChange={e => {
            onChange({
              ...todo,
              title: e.target.value
            });
          }} />
        <button onClick={() => setIsEditing(false)}>
          Save
        </button>
      </>
    );
  } else {
    todoContent = (
      <>
        {todo.title}
        <button onClick={() => setIsEditing(true)}>
          Edit
        </button>
      </>
    );
  }
  return (
    <label>
      <input
        type="checkbox"
        checked={todo.done}
        onChange={e => {
          onChange({
            ...todo,
            done: e.target.checked
          });
        }}
      />
      {todoContent}
      <button onClick={() => onDelete(todo.id)}>
        Delete
      </button>
    </label>
  );
}
```

```css
button { margin: 5px; }
li { list-style-type: none; }
ul, li { margin: 0; padding: 0; }
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

使用 Immer 时，你可以用可变的方式编写代码，只要你修改的是 Immer 提供给你的 `draft` 的一部分即可。这里，所有修改都发生在 `draft` 上，所以代码可以正常工作：

<Sandpack>

```js src/App.js
import { useState } from 'react';
import { useImmer } from 'use-immer';
import AddTodo from './AddTodo.js';
import TaskList from './TaskList.js';

let nextId = 3;
const initialTodos = [
  { id: 0, title: 'Buy milk', done: true },
  { id: 1, title: 'Eat tacos', done: false },
  { id: 2, title: 'Brew tea', done: false },
];

export default function TaskApp() {
  const [todos, updateTodos] = useImmer(
    initialTodos
  );

  function handleAddTodo(title) {
    updateTodos(draft => {
      draft.push({
        id: nextId++,
        title: title,
        done: false
      });
    });
  }

  function handleChangeTodo(nextTodo) {
    updateTodos(draft => {
      const todo = draft.find(t =>
        t.id === nextTodo.id
      );
      todo.title = nextTodo.title;
      todo.done = nextTodo.done;
    });
  }

  function handleDeleteTodo(todoId) {
    updateTodos(draft => {
      const index = draft.findIndex(t =>
        t.id === todoId
      );
      draft.splice(index, 1);
    });
  }

  return (
    <>
      <AddTodo
        onAddTodo={handleAddTodo}
      />
      <TaskList
        todos={todos}
        onChangeTodo={handleChangeTodo}
        onDeleteTodo={handleDeleteTodo}
      />
    </>
  );
}
```

```js src/AddTodo.js
import { useState } from 'react';

export default function AddTodo({ onAddTodo }) {
  const [title, setTitle] = useState('');
  return (
    <>
      <input
        placeholder="Add todo"
        value={title}
        onChange={e => setTitle(e.target.value)}
      />
      <button onClick={() => {
        setTitle('');
        onAddTodo(title);
      }}>Add</button>
    </>
  )
}
```

```js src/TaskList.js
import { useState } from 'react';

export default function TaskList({
  todos,
  onChangeTodo,
  onDeleteTodo
}) {
  return (
    <ul>
      {todos.map(todo => (
        <li key={todo.id}>
          <Task
            todo={todo}
            onChange={onChangeTodo}
            onDelete={onDeleteTodo}
          />
        </li>
      ))}
    </ul>
  );
}

function Task({ todo, onChange, onDelete }) {
  const [isEditing, setIsEditing] = useState(false);
  let todoContent;
  if (isEditing) {
    todoContent = (
      <>
        <input
          value={todo.title}
          onChange={e => {
            onChange({
              ...todo,
              title: e.target.value
            });
          }} />
        <button onClick={() => setIsEditing(false)}>
          Save
        </button>
      </>
    );
  } else {
    todoContent = (
      <>
        {todo.title}
        <button onClick={() => setIsEditing(true)}>
          Edit
        </button>
      </>
    );
  }
  return (
    <label>
      <input
        type="checkbox"
        checked={todo.done}
        onChange={e => {
          onChange({
            ...todo,
            done: e.target.checked
          });
        }}
      />
      {todoContent}
      <button onClick={() => onDelete(todo.id)}>
        Delete
      </button>
    </label>
  );
}
```

```css
button { margin: 5px; }
li { list-style-type: none; }
ul, li { margin: 0; padding: 0; }
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

你也可以在 Immer 中混合使用可变和不可变两种方式。

例如，在这个版本里，`handleAddTodo` 通过修改 Immer 的 `draft` 来实现，而 `handleChangeTodo` 和 `handleDeleteTodo` 则使用非突变的 `map` 和 `filter` 方法：

<Sandpack>

```js src/App.js
import { useState } from 'react';
import { useImmer } from 'use-immer';
import AddTodo from './AddTodo.js';
import TaskList from './TaskList.js';

let nextId = 3;
const initialTodos = [
  { id: 0, title: 'Buy milk', done: true },
  { id: 1, title: 'Eat tacos', done: false },
  { id: 2, title: 'Brew tea', done: false },
];

export default function TaskApp() {
  const [todos, updateTodos] = useImmer(
    initialTodos
  );

  function handleAddTodo(title) {
    updateTodos(draft => {
      draft.push({
        id: nextId++,
        title: title,
        done: false
      });
    });
  }

  function handleChangeTodo(nextTodo) {
    updateTodos(todos.map(todo => {
      if (todo.id === nextTodo.id) {
        return nextTodo;
      } else {
        return todo;
      }
    }));
  }

  function handleDeleteTodo(todoId) {
    updateTodos(
      todos.filter(t => t.id !== todoId)
    );
  }

  return (
    <>
      <AddTodo
        onAddTodo={handleAddTodo}
      />
      <TaskList
        todos={todos}
        onChangeTodo={handleChangeTodo}
        onDeleteTodo={handleDeleteTodo}
      />
    </>
  );
}
```

```js src/AddTodo.js
import { useState } from 'react';

export default function AddTodo({ onAddTodo }) {
  const [title, setTitle] = useState('');
  return (
    <>
      <input
        placeholder="Add todo"
        value={title}
        onChange={e => setTitle(e.target.value)}
      />
      <button onClick={() => {
        setTitle('');
        onAddTodo(title);
      }}>Add</button>
    </>
  )
}
```

```js src/TaskList.js
import { useState } from 'react';

export default function TaskList({
  todos,
  onChangeTodo,
  onDeleteTodo
}) {
  return (
    <ul>
      {todos.map(todo => (
        <li key={todo.id}>
          <Task
            todo={todo}
            onChange={onChangeTodo}
            onDelete={onDeleteTodo}
          />
        </li>
      ))}
    </ul>
  );
}

function Task({ todo, onChange, onDelete }) {
  const [isEditing, setIsEditing] = useState(false);
  let todoContent;
  if (isEditing) {
    todoContent = (
      <>
        <input
          value={todo.title}
          onChange={e => {
            onChange({
              ...todo,
              title: e.target.value
            });
          }} />
        <button onClick={() => setIsEditing(false)}>
          Save
        </button>
      </>
    );
  } else {
    todoContent = (
      <>
        {todo.title}
        <button onClick={() => setIsEditing(true)}>
          Edit
        </button>
      </>
    );
  }
  return (
    <label>
      <input
        type="checkbox"
        checked={todo.done}
        onChange={e => {
          onChange({
            ...todo,
            done: e.target.checked
          });
        }}
      />
      {todoContent}
      <button onClick={() => onDelete(todo.id)}>
        Delete
      </button>
    </label>
  );
}
```

```css
button { margin: 5px; }
li { list-style-type: none; }
ul, li { margin: 0; padding: 0; }
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

使用 Immer 时，你可以为每个具体情况选择最自然的写法。

</Solution>

</Challenges>
