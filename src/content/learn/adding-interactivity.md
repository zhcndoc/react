---
title: 添加交互性
---

<Intro>

屏幕上的某些内容会根据用户输入而更新。例如，点击图片图库会切换当前图片。在 React 中，随时间变化的数据称为 *state*。你可以向任何组件添加 state，并在需要时更新它。在本章中，你将学习如何编写能够处理交互、更新其 state 并随着时间显示不同输出的组件。

</Intro>

<YouWillLearn isChapter={true}>

* [如何处理用户发起的事件](/learn/responding-to-events)
* [如何使用 state 让组件“记住”信息](/learn/state-a-components-memory)
* [React 如何分两个阶段更新 UI](/learn/render-and-commit)
* [为什么 state 在你更改后不会立刻更新](/learn/state-as-a-snapshot)
* [如何排队多个 state 更新](/learn/queueing-a-series-of-state-updates)
* [如何更新 state 中的对象](/learn/updating-objects-in-state)
* [如何更新 state 中的数组](/learn/updating-arrays-in-state)

</YouWillLearn>

## 响应事件 {/*responding-to-events*/}

React 允许你向 JSX 添加 *事件处理器*。事件处理器是你自己编写的函数，会在用户交互时被触发，比如点击、悬停、聚焦表单输入框等等。

像 `<button>` 这样的内置组件只支持浏览器内置事件，比如 `onClick`。不过，你也可以创建自己的组件，并为它们的事件处理器 props 赋予任何你喜欢的、特定于应用的名称。

<Sandpack>

```js
export default function App() {
  return (
    <Toolbar
      onPlayMovie={() => alert('播放中！')}
      onUploadImage={() => alert('正在上传！')}
    />
  );
}

function Toolbar({ onPlayMovie, onUploadImage }) {
  return (
    <div>
      <Button onClick={onPlayMovie}>
        播放电影
      </Button>
      <Button onClick={onUploadImage}>
        上传图片
      </Button>
    </div>
  );
}

function Button({ onClick, children }) {
  return (
    <button onClick={onClick}>
      {children}
    </button>
  );
}
```

```css
button { margin-right: 10px; }
```

</Sandpack>

<LearnMore path="/learn/responding-to-events">

阅读 **[响应事件](/learn/responding-to-events)** 以了解如何添加事件处理器。

</LearnMore>

## State：组件的记忆 {/*state-a-components-memory*/}

组件通常需要根据交互来改变屏幕上的内容。在表单中输入应该更新输入框字段，点击图片轮播中的“next”应该更改显示的图片，点击“buy”会把商品放入购物车。组件需要“记住”一些东西：当前输入值、当前图片、购物车。在 React 中，这种组件特有的记忆称为 *state*。

你可以使用 [`useState`](/reference/react/useState) Hook 为组件添加 state。*Hooks* 是特殊的函数，它们让组件能够使用 React 功能（state 就是其中之一）。`useState` Hook 允许你声明一个 state 变量。它接收初始 state，并返回一对值：当前 state，以及一个用于更新它的 state 设置函数。

```js
const [index, setIndex] = useState(0);
const [showMore, setShowMore] = useState(false);
```

下面是一个图片图库如何在点击时使用并更新 state：

<Sandpack>

```js
import { useState } from 'react';
import { sculptureList } from './data.js';

export default function Gallery() {
  const [index, setIndex] = useState(0);
  const [showMore, setShowMore] = useState(false);
  const hasNext = index < sculptureList.length - 1;

  function handleNextClick() {
    if (hasNext) {
      setIndex(index + 1);
    } else {
      setIndex(0);
    }
  }

  function handleMoreClick() {
    setShowMore(!showMore);
  }

  let sculpture = sculptureList[index];
  return (
    <>
      <button onClick={handleNextClick}>
        下一个
      </button>
      <h2>
        <i>{sculpture.name} </i>
        作者 {sculpture.artist}
      </h2>
      <h3>
        （{index + 1} / {sculptureList.length}）
      </h3>
      <button onClick={handleMoreClick}>
        {showMore ? '隐藏' : '显示'}详情
      </button>
      {showMore && <p>{sculpture.description}</p>}
      <img
        src={sculpture.url}
        alt={sculpture.alt}
      />
    </>
  );
}
```

```js src/data.js
export const sculptureList = [{
  name: '向神经外科致敬',
  artist: 'Marta Colvin Andrade',
  description: '尽管 Colvin 主要因暗示前西班牙时期符号的抽象主题而闻名，但这座巨大的雕塑——对神经外科的致敬——是她最具辨识度的公共艺术作品之一。',
  url: 'https://react.dev/images/docs/scientists/Mx7dA2Y.jpg',
  alt: '一尊青铜雕像，两个交叉的手指尖轻柔地托着一颗人脑。'
}, {
  name: '花之奇迹',
  artist: 'Eduardo Catalano',
  description: '这朵巨大的银色花朵（75 英尺，约 23 米）位于布宜诺斯艾利斯。它被设计为可移动，在傍晚或强风吹拂时闭合花瓣，在清晨重新绽放。',
  url: 'https://react.dev/images/docs/scientists/ZF6s192m.jpg',
  alt: '一座巨大的金属花朵雕塑，具有反光镜面般的花瓣和粗壮的花蕊。'
}, {
  name: '永恒的存在',
  artist: 'John Woodrow Wilson',
  description: 'Wilson 以对平等、社会正义以及人类本质和精神品质的关注而闻名。这件巨大的（7 英尺，约 2.13 米）青铜作品代表了他所描述的“融入普遍人性的象征性黑人存在”。',
  url: 'https://react.dev/images/docs/scientists/aTtVpES.jpg',
  alt: '这座描绘人类头部的雕塑似乎永远存在，庄严肃穆，散发出平静与安宁。'
}, {
  name: '摩艾石像',
  artist: 'Unknown Artist',
  description: '在复活节岛上，有 1000 尊摩艾石像，或称现存的纪念性雕像，由早期拉帕努伊人创作，一些人认为它们代表了神化的祖先。',
  url: 'https://react.dev/images/docs/scientists/RCwLEoQm.jpg',
  alt: '三尊纪念性石质半身像，头部比例异常大，面容严肃。'
}, {
  name: '蓝娜娜',
  artist: 'Niki de Saint Phalle',
  description: '娜娜们是胜利的生物，是女性气质和母性的象征。起初，Saint Phalle 为娜娜系列使用布料和现成物件，后来又引入聚酯材料以实现更有活力的效果。',
  url: 'https://react.dev/images/docs/scientists/Sd1AgUOm.jpg',
  alt: '一座大型马赛克雕塑，描绘一位穿着彩色服饰、充满奇趣的舞蹈女性形象，洋溢着喜悦。'
}, {
  name: '终极形式',
  artist: 'Barbara Hepworth',
  description: '这座抽象青铜雕塑是位于约克郡雕塑公园的《人类大家庭》系列的一部分。Hepworth 选择不去创作世界的具象表现，而是发展出受人物与景观启发的抽象形式。',
  url: 'https://react.dev/images/docs/scientists/2heNQDcm.jpg',
  alt: '一座高大的雕塑，由三个元素堆叠而成，令人联想到人形。'
}, {
  name: '骑士',
  artist: 'Lamidi Olonade Fakeye',
  description: 'Fakeye 出身于四代木雕世家，他的作品融合了传统与当代约鲁巴主题。',
  url: 'https://react.dev/images/docs/scientists/wIdGuZwm.png',
  alt: '一座精细的木雕，描绘了一位神情专注的骑士骑在一匹装饰有图案的马上。'
}, {
  name: '大肚子',
  artist: 'Alina Szapocznikow',
  description: 'Szapocznikow 以其对残缺身体的雕塑而闻名，将其作为青春与美丽脆弱和短暂的隐喻。这座雕塑描绘了两个非常逼真的大肚子，一个叠在另一个上面，每个大约五英尺（1.5 米）高。',
  url: 'https://react.dev/images/docs/scientists/AlHTAdDm.jpg',
  alt: '这座雕塑让人联想到层层褶皱，与古典雕塑中的腹部截然不同。'
}, {
  name: '兵马俑',
  artist: 'Unknown Artist',
  description: '兵马俑是一组陶制雕塑，描绘了秦始皇——中国第一位皇帝——的军队。这支军队由 8000 多名士兵、130 辆战车和 520 匹战马、150 匹骑兵马组成。',
  url: 'https://react.dev/images/docs/scientists/HMFmH6m.jpg',
  alt: '12 座陶俑雕塑，描绘庄严的战士，每一尊都有独特的面部表情和盔甲。'
}, {
  name: '月球景观',
  artist: 'Louise Nevelson',
  description: 'Nevelson 以从纽约市废墟中搜集物件而闻名，之后她会将这些物件组装成纪念性构筑物。在这件作品中，她使用了床柱、杂耍棒和座椅碎片等不同部件，将它们钉接和粘贴到盒子里，反映出立体主义对空间与形式的几何抽象影响。',
  url: 'https://react.dev/images/docs/scientists/rN7hY6om.jpg',
  alt: '一座黑色哑光雕塑，起初很难分辨各个单独元素。'
}, {
  name: '光环',
  artist: 'Ranjani Shettar',
  description: 'Shettar 将传统与现代、自然与工业融合在一起。她的艺术聚焦于人与自然之间的关系。她的作品被描述为既抽象又具象、令人印象深刻、具有反重力感，并且是“对不太可能材料的精妙综合”。',
  url: 'https://react.dev/images/docs/scientists/okTpbHhm.jpg',
  alt: '一座浅色、像金属线一样的雕塑，安装在混凝土墙上并延伸到地面，看起来很轻盈。'
}, {
  name: '河马',
  artist: 'Taipei Zoo',
  description: '台北动物园委托制作了一个包含水下嬉戏河马的“河马广场”。',
  url: 'https://react.dev/images/docs/scientists/6o5Vuyu.jpg',
  alt: '一组青铜河马雕塑从人行道铺装中浮现出来，仿佛正在游泳。'
}];
```

```css
h2 { margin-top: 10px; margin-bottom: 0; }
h3 {
 margin-top: 5px;
 font-weight: normal;
 font-size: 100%;
}
img { width: 120px; height: 120px; }
button {
  display: block;
  margin-top: 10px;
  margin-bottom: 10px;
}
```

</Sandpack>

<LearnMore path="/learn/state-a-components-memory">

阅读 **[State: A Component's Memory](/learn/state-a-components-memory)** 以了解如何记住一个值并在交互时更新它。

</LearnMore>

## 渲染与提交 {/*render-and-commit*/}

在你的组件显示到屏幕上之前，它们必须先由 React 渲染。理解这个过程的步骤将帮助你思考代码是如何执行的，以及解释它的行为。

想象你的组件是厨房里的厨师，用各种食材做出美味佳肴。在这个场景中，React 是服务员，它接收顾客的点单并把订单送到厨房。这个请求和提供 UI 的过程分为三步：

1. **触发**一次渲染（把食客的订单送到厨房）
2. **渲染**组件（在厨房里准备订单）
3. **提交**到 DOM（把订单端上桌）

<IllustrationBlock sequential>
  <Illustration caption="触发" alt="React 作为餐厅里的服务员，从用户那里取订单并将它们送到组件厨房。" src="/images/docs/illustrations/i_render-and-commit1.png" />
  <Illustration caption="渲染" alt="卡片厨师给 React 一个新的 Card 组件。" src="/images/docs/illustrations/i_render-and-commit2.png" />
  <Illustration caption="提交" alt="React 将 Card 送到用户的餐桌上。" src="/images/docs/illustrations/i_render-and-commit3.png" />
</IllustrationBlock>

<LearnMore path="/learn/render-and-commit">

阅读 **[渲染与提交](/learn/render-and-commit)**，了解一次 UI 更新的生命周期。

</LearnMore>

## 状态如同快照 {/*state-as-a-snapshot*/}

与普通的 JavaScript 变量不同，React 状态更像一个快照。设置它并不会改变你已经拥有的那个状态变量，而是会触发一次重新渲染。一开始这可能会让人感到惊讶！

```js
console.log(count);  // 0
setCount(count + 1); // 请求使用 1 进行重新渲染
console.log(count);  // 仍然是 0！
```

这种行为可以帮助你避免一些微妙的 bug。下面是一个小聊天应用。试着猜猜，如果你先按下“发送”，*然后*把收件人改成 Bob，会发生什么。五秒后 `alert` 中会显示谁的名字？

<Sandpack>

```js
import { useState } from 'react';

export default function Form() {
  const [to, setTo] = useState('Alice');
  const [message, setMessage] = useState('Hello');

  function handleSubmit(e) {
    e.preventDefault();
    setTimeout(() => {
      alert(`You said ${message} to ${to}`);
    }, 5000);
  }

  return (
    <form onSubmit={handleSubmit}>
      <label>
        To:{' '}
        <select
          value={to}
          onChange={e => setTo(e.target.value)}>
          <option value="Alice">Alice</option>
          <option value="Bob">Bob</option>
        </select>
      </label>
      <textarea
        placeholder="消息"
        value={message}
        onChange={e => setMessage(e.target.value)}
      />
      <button type="submit">发送</button>
    </form>
  );
}
```

```css
label, textarea { margin-bottom: 10px; display: block; }
```

</Sandpack>


<LearnMore path="/learn/state-as-a-snapshot">

阅读 **[状态如同快照](/learn/state-as-a-snapshot)**，了解为什么状态在事件处理器内部看起来是“固定的”且不会变化。

</LearnMore>

## 排队一系列状态更新 {/*queueing-a-series-of-state-updates*/}

这个组件有 bug：点击“+3”只会让分数增加一次。

<Sandpack>

```js
import { useState } from 'react';

export default function Counter() {
  const [score, setScore] = useState(0);

  function increment() {
    setScore(score + 1);
  }

  return (
    <>
      <button onClick={() => increment()}>+1</button>
      <button onClick={() => {
        increment();
        increment();
        increment();
      }}>+3</button>
      <h1>Score: {score}</h1>
    </>
  )
}
```

```css
button { display: inline-block; margin: 10px; font-size: 20px; }
```

</Sandpack>

[状态如同快照](/learn/state-as-a-snapshot) 解释了为什么会这样。设置状态会请求一次新的重新渲染，但不会改变已经在运行的代码中的状态。所以在你调用 `setScore(score + 1)` 之后，`score` 仍然会是 `0`。

```js
console.log(score);  // 0
setScore(score + 1); // setScore(0 + 1);
console.log(score);  // 0
setScore(score + 1); // setScore(0 + 1);
console.log(score);  // 0
setScore(score + 1); // setScore(0 + 1);
console.log(score);  // 0
```

你可以在设置状态时传入一个*更新函数*来修复它。注意把 `setScore(score + 1)` 改成 `setScore(s => s + 1)` 是如何修复“+3”按钮的。这让你可以排队多个状态更新。

<Sandpack>

```js
import { useState } from 'react';

export default function Counter() {
  const [score, setScore] = useState(0);

  function increment() {
    setScore(s => s + 1);
  }

  return (
    <>
      <button onClick={() => increment()}>+1</button>
      <button onClick={() => {
        increment();
        increment();
        increment();
      }}>+3</button>
      <h1>Score: {score}</h1>
    </>
  )
}
```

```css
button { display: inline-block; margin: 10px; font-size: 20px; }
```

</Sandpack>

<LearnMore path="/learn/queueing-a-series-of-state-updates">

阅读 **[排队一系列状态更新](/learn/queueing-a-series-of-state-updates)**，了解如何对一系列状态更新进行排队。

</LearnMore>

## 更新状态中的对象 {/*updating-objects-in-state*/}

状态可以保存任何类型的 JavaScript 值，包括对象。但是你不应该直接修改保存在 React 状态中的对象和数组。相反，当你想要更新对象和数组时，需要创建一个新的对象（或者复制一个已有的对象），然后更新状态以使用那个副本。

通常，你会使用 `...` 展开语法来复制你想要更改的对象和数组。例如，更新一个嵌套对象可以像这样：

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
        Name:
        <input
          value={person.name}
          onChange={handleNameChange}
        />
      </label>
      <label>
        Title:
        <input
          value={person.artwork.title}
          onChange={handleTitleChange}
        />
      </label>
      <label>
        City:
        <input
          value={person.artwork.city}
          onChange={handleCityChange}
        />
      </label>
      <label>
        Image:
        <input
          value={person.artwork.image}
          onChange={handleImageChange}
        />
      </label>
      <p>
        <i>{person.artwork.title}</i>
        {' by '}
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

如果在代码中复制对象让你觉得繁琐，你可以使用像 [Immer](https://github.com/immerjs/use-immer) 这样的库来减少重复代码：

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
        Name:
        <input
          value={person.name}
          onChange={handleNameChange}
        />
      </label>
      <label>
        Title:
        <input
          value={person.artwork.title}
          onChange={handleTitleChange}
        />
      </label>
      <label>
        City:
        <input
          value={person.artwork.city}
          onChange={handleCityChange}
        />
      </label>
      <label>
        Image:
        <input
          value={person.artwork.image}
          onChange={handleImageChange}
        />
      </label>
      <p>
        <i>{person.artwork.title}</i>
        {' by '}
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

<LearnMore path="/learn/updating-objects-in-state">

阅读 **[更新状态中的对象](/learn/updating-objects-in-state)**，了解如何正确地更新对象。

</LearnMore>

## 在 state 中更新数组 {/*updating-arrays-in-state*/}

数组也是另一种可以存储在 state 中的可变 JavaScript 对象，但应将其视为只读。和对象一样，当你想要更新存储在 state 中的数组时，需要创建一个新的数组（或者制作现有数组的副本），然后将 state 设置为使用这个新数组：

<Sandpack>

```js
import { useState } from 'react';

const initialList = [
  { id: 0, title: '大肚子', seen: false },
  { id: 1, title: '月球景观', seen: false },
  { id: 2, title: '兵马俑', seen: true },
];

export default function BucketList() {
  const [list, setList] = useState(
    initialList
  );

  function handleToggle(artworkId, nextSeen) {
    setList(list.map(artwork => {
      if (artwork.id === artworkId) {
        return { ...artwork, seen: nextSeen };
      } else {
        return artwork;
      }
    }));
  }

  return (
    <>
      <h1>艺术愿望清单</h1>
      <h2>我想看的艺术作品列表：</h2>
      <ItemList
        artworks={list}
        onToggle={handleToggle} />
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

如果在代码中复制数组变得繁琐，你可以使用像 [Immer](https://github.com/immerjs/use-immer) 这样的库来减少重复代码：

<Sandpack>

```js
import { useState } from 'react';
import { useImmer } from 'use-immer';

const initialList = [
  { id: 0, title: '大肚子', seen: false },
  { id: 1, title: '月球景观', seen: false },
  { id: 2, title: '兵马俑', seen: true },
];

export default function BucketList() {
  const [list, updateList] = useImmer(initialList);

  function handleToggle(artworkId, nextSeen) {
    updateList(draft => {
      const artwork = draft.find(a =>
        a.id === artworkId
      );
      artwork.seen = nextSeen;
    });
  }

  return (
    <>
      <h1>艺术愿望清单</h1>
      <h2>我想看的艺术作品列表：</h2>
      <ItemList
        artworks={list}
        onToggle={handleToggle} />
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

<LearnMore path="/learn/updating-arrays-in-state">

阅读 **[在 state 中更新数组](/learn/updating-arrays-in-state)** 以了解如何正确更新数组。

</LearnMore>

## 接下来是什么？ {/*whats-next*/}

前往 [响应事件](/learn/responding-to-events) 开始逐页阅读本章！

或者，如果你已经熟悉这些主题，为什么不读一读 [管理 State](/learn/managing-state) 呢？
