---
title: "状态：组件的记忆"
---

<Intro>

组件通常需要根据交互来改变屏幕上显示的内容。向表单中输入内容应该更新输入字段，在图片轮播中点击“下一张”应该切换显示的图片，点击“购买”应该把商品加入购物车。组件需要“记住”一些东西：当前的输入值、当前的图片、购物车。在 React 中，这种组件特有的记忆被称为 *state*。

</Intro>

<YouWillLearn>

* 如何使用 [`useState`](/reference/react/useState) Hook 添加状态变量
* `useState` Hook 返回的是哪一对值
* 如何添加一个以上的状态变量
* 为什么 state 被称为局部状态

</YouWillLearn>

## 当普通变量不够用时 {/*when-a-regular-variable-isnt-enough*/}

这里有一个渲染雕塑图片的组件。点击“Next”按钮应该通过将 `index` 改为 `1`、然后 `2`，依此类推，来显示下一个雕塑。不过，这**不会起作用**（你可以试试看！）：

<Sandpack>

```js {expectedErrors: {'react-compiler': [7]}}
import { sculptureList } from './data.js';

export default function Gallery() {
  let index = 0;

  function handleClick() {
    index = index + 1;
  }

  let sculpture = sculptureList[index];
  return (
    <>
      <button onClick={handleClick}>
        Next
      </button>
      <h2>
        <i>{sculpture.name} </i>
        by {sculpture.artist}
      </h2>
      <h3>
        ({index + 1} of {sculptureList.length})
      </h3>
      <img
        src={sculpture.url}
        alt={sculpture.alt}
      />
      <p>
        {sculpture.description}
      </p>
    </>
  );
}
```

```js src/data.js
export const sculptureList = [{
  name: 'Homenaje a la Neurocirugía',
  artist: 'Marta Colvin Andrade',
  description: '虽然 Colvin 主要因那些暗示前西班牙时期符号的抽象主题而闻名，但这座巨大的雕塑——向神经外科致敬——是她最具辨识度的公共艺术作品之一。',
  url: 'https://react.dev/images/docs/scientists/Mx7dA2Y.jpg',
  alt: '一尊青铜雕像，两个交叉的手轻柔地用指尖托着一颗人脑。'
}, {
  name: 'Floralis Genérica',
  artist: 'Eduardo Catalano',
  description: '这朵巨大的（75 英尺或 23 米）银色花朵位于布宜诺斯艾利斯。它被设计成可以移动，在傍晚或强风吹拂时合拢花瓣，并在早晨打开。',
  url: 'https://react.dev/images/docs/scientists/ZF6s192m.jpg',
  alt: '一座巨大的金属花朵雕塑，带有反射镜面般的花瓣和粗壮的花蕊。'
}, {
  name: 'Eternal Presence',
  artist: 'John Woodrow Wilson',
  description: 'Wilson 以他对平等、社会正义以及人类本质与精神品质的关注而闻名。这座巨大的（7 英尺或 2.13 米）青铜作品，代表了他所说的“注入了普世人类意识的象征性黑人存在”。',
  url: 'https://react.dev/images/docs/scientists/aTtVpES.jpg',
  alt: '这座描绘人类头部的雕塑似乎始终存在，庄严肃穆。它散发着平静与安宁。'
}, {
  name: 'Moai',
  artist: 'Unknown Artist',
  description: '位于复活节岛上，有 1,000 尊摩艾石像，或现存的纪念性雕像，由早期拉帕努伊人创造，有些人认为它们代表了被神化的祖先。',
  url: 'https://react.dev/images/docs/scientists/RCwLEoQm.jpg',
  alt: '三尊纪念性石质半身像，头部比例异常大，面容严肃。'
}, {
  name: 'Blue Nana',
  artist: 'Niki de Saint Phalle',
  description: 'Nanas 是象征女性气质与母性的胜利生灵。最初，Saint Phalle 为 Nanas 使用布料和现成物品，后来又引入聚酯材料以获得更鲜艳的效果。',
  url: 'https://react.dev/images/docs/scientists/Sd1AgUOm.jpg',
  alt: '一座大型马赛克雕塑，展现一个充满奇想、翩翩起舞的女性形象，身着色彩鲜艳的服装，洋溢着喜悦。'
}, {
  name: 'Ultimate Form',
  artist: 'Barbara Hepworth',
  description: '这座抽象青铜雕塑是位于约克郡雕塑公园的《人类家族》系列的一部分。Hepworth 选择不去创作对世界的写实再现，而是发展出受人和风景启发的抽象形式。',
  url: 'https://react.dev/images/docs/scientists/2heNQDcm.jpg',
  alt: '一座由三个元素堆叠而成的高大雕塑，让人联想到人形。'
}, {
  name: 'Cavaliere',
  artist: 'Lamidi Olonade Fakeye',
  description: 'Fakeye 出身于四代木雕世家，他的作品融合了传统与当代的约鲁巴主题。',
  url: 'https://react.dev/images/docs/scientists/wIdGuZwm.png',
  alt: '一座精致的木雕武士骑在马上，神情专注，马身装饰着纹样。'
}, {
  name: 'Big Bellies',
  artist: 'Alina Szapocznikow',
  description: 'Szapocznikow 以其碎片化身体雕塑而闻名，这些作品将其作为青春与美丽脆弱和短暂的隐喻。这座雕塑描绘了两个非常逼真的大肚子彼此叠放，每个高约五英尺（1.5 米）。',
  url: 'https://react.dev/images/docs/scientists/AlHTAdDm.jpg',
  alt: '这座雕塑让人联想到一层层褶皱，与古典雕塑中的肚子截然不同。'
}, {
  name: 'Terracotta Army',
  artist: 'Unknown Artist',
  description: '兵马俑是一个由陶俑组成的收藏，描绘了秦始皇的军队。这支军队由 8,000 多名士兵、130 辆战车和 520 匹战马、以及 150 匹骑兵马组成。',
  url: 'https://react.dev/images/docs/scientists/HMFmH6m.jpg',
  alt: '12 尊陶俑形态的庄严武士雕塑，每尊都有独特的面部表情和盔甲。'
}, {
  name: 'Lunar Landscape',
  artist: 'Louise Nevelson',
  description: 'Nevelson 以从纽约市废弃物中搜集物件而闻名，随后将其组装成纪念性构筑物。在这件作品中，她使用了床柱、杂耍球棒和座椅碎片等不同部分，将它们钉合并粘接到盒子里，体现了立体主义对空间与形式的几何抽象所带来的影响。',
  url: 'https://react.dev/images/docs/scientists/rN7hY6om.jpg',
  alt: '一座哑光黑色雕塑，各个元素最初难以区分。'
}, {
  name: 'Aureole',
  artist: 'Ranjani Shettar',
  description: 'Shettar 将传统与现代、自然与工业相融合。她的艺术专注于人与自然之间的关系。她的作品被形容为抽象与具象皆引人注目、违背重力，以及“一种对不太可能材料的精妙综合”。',
  url: 'https://react.dev/images/docs/scientists/okTpbHhm.jpg',
  alt: '一座浅色、如金属丝般的雕塑安装在混凝土墙上并垂落到地面，看起来很轻盈。'
}, {
  name: 'Hippos',
  artist: 'Taipei Zoo',
  description: '台北动物园委托创作了一个河马广场，展示了一群在水中嬉戏的半 submerged 河马。',
  url: 'https://react.dev/images/docs/scientists/6o5Vuyu.jpg',
  alt: '一组青铜河马雕塑从铺石人行道中冒出，仿佛它们正在游泳。'
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

`handleClick` 事件处理函数正在更新一个局部变量 `index`。但有两个原因导致这个变化不会显示出来：

1. **局部变量不会在渲染之间保留。** 当 React 第二次渲染这个组件时，它会从头开始渲染——不会考虑任何对局部变量所做的更改。
2. **对局部变量的更改不会触发渲染。** React 不会意识到它需要使用新数据再次渲染该组件。

要用新数据更新组件，需要发生两件事：

1. **在渲染之间保留** 数据。
2. **触发** React 使用新数据渲染组件（重新渲染）。

[`useState`](/reference/react/useState) Hook 提供了这两件事：

1. 一个用于在渲染之间保留数据的**状态变量**。
2. 一个用于更新该变量并触发 React 再次渲染组件的**状态设置函数**。

## 添加状态变量 {/*adding-a-state-variable*/}

要添加一个状态变量，请先在文件顶部从 React 导入 `useState`：

```js
import { useState } from 'react';
```

然后，将这一行：

```js
let index = 0;
```

替换为

```js
const [index, setIndex] = useState(0);
```

`index` 是状态变量，`setIndex` 是设置函数。

> 这里的 `[` 和 `]` 语法称为 [数组解构](https://javascript.info/destructuring-assignment)，它让你可以从数组中读取值。`useState` 返回的数组始终正好有两个项目。

它们在 `handleClick` 中是这样协同工作的：

```js
function handleClick() {
  setIndex(index + 1);
}
```

现在点击“Next”按钮会切换当前的雕塑：

<Sandpack>

```js
import { useState } from 'react';
import { sculptureList } from './data.js';

export default function Gallery() {
  const [index, setIndex] = useState(0);

  function handleClick() {
    setIndex(index + 1);
  }

  let sculpture = sculptureList[index];
  return (
    <>
      <button onClick={handleClick}>
        Next
      </button>
      <h2>
        <i>{sculpture.name} </i>
        by {sculpture.artist}
      </h2>
      <h3>
        ({index + 1} of {sculptureList.length})
      </h3>
      <img
        src={sculpture.url}
        alt={sculpture.alt}
      />
      <p>
        {sculpture.description}
      </p>
    </>
  );
}
```

```js src/data.js
export const sculptureList = [{
  name: 'Homenaje a la Neurocirugía',
  artist: 'Marta Colvin Andrade',
  description: '虽然 Colvin 主要因那些暗示前西班牙时期符号的抽象主题而闻名，但这座巨大的雕塑——向神经外科致敬——是她最具辨识度的公共艺术作品之一。',
  url: 'https://react.dev/images/docs/scientists/Mx7dA2Y.jpg',
  alt: '一尊青铜雕像，两个交叉的手轻柔地用指尖托着一颗人脑。'
}, {
  name: 'Floralis Genérica',
  artist: 'Eduardo Catalano',
  description: '这朵巨大的（75 英尺或 23 米）银色花朵位于布宜诺斯艾利斯。它被设计成可以移动，在傍晚或强风吹拂时合拢花瓣，并在早晨打开。',
  url: 'https://react.dev/images/docs/scientists/ZF6s192m.jpg',
  alt: '一座巨大的金属花朵雕塑，带有反射镜面般的花瓣和粗壮的花蕊。'
}, {
  name: 'Eternal Presence',
  artist: 'John Woodrow Wilson',
  description: 'Wilson 以他对平等、社会正义以及人类本质与精神品质的关注而闻名。这座巨大的（7 英尺或 2.13 米）青铜作品，代表了他所说的“注入了普世人类意识的象征性黑人存在”。',
  url: 'https://react.dev/images/docs/scientists/aTtVpES.jpg',
  alt: '这座描绘人类头部的雕塑似乎始终存在，庄严肃穆。它散发着平静与安宁。'
}, {
  name: 'Moai',
  artist: 'Unknown Artist',
  description: '位于复活节岛上，有 1,000 尊摩艾石像，或现存的纪念性雕像，由早期拉帕努伊人创造，有些人认为它们代表了被神化的祖先。',
  url: 'https://react.dev/images/docs/scientists/RCwLEoQm.jpg',
  alt: '三尊纪念性石质半身像，头部比例异常大，面容严肃。'
}, {
  name: 'Blue Nana',
  artist: 'Niki de Saint Phalle',
  description: 'Nanas 是象征女性气质与母性的胜利生灵。最初，Saint Phalle 为 Nanas 使用布料和现成物品，后来又引入聚酯材料以获得更鲜艳的效果。',
  url: 'https://react.dev/images/docs/scientists/Sd1AgUOm.jpg',
  alt: '一座大型马赛克雕塑，展现一个充满奇想、翩翩起舞的女性形象，身着色彩鲜艳的服装，洋溢着喜悦。'
}, {
  name: 'Ultimate Form',
  artist: 'Barbara Hepworth',
  description: '这座抽象青铜雕塑是位于约克郡雕塑公园的《人类家族》系列的一部分。Hepworth 选择不去创作对世界的写实再现，而是发展出受人和风景启发的抽象形式。',
  url: 'https://react.dev/images/docs/scientists/2heNQDcm.jpg',
  alt: '一座由三个元素堆叠而成的高大雕塑，让人联想到人形。'
}, {
  name: 'Cavaliere',
  artist: 'Lamidi Olonade Fakeye',
  description: 'Fakeye 出身于四代木雕世家，他的作品融合了传统与当代的约鲁巴主题。',
  url: 'https://react.dev/images/docs/scientists/wIdGuZwm.png',
  alt: '一座精致的木雕武士骑在马上，神情专注，马身装饰着纹样。'
}, {
  name: 'Big Bellies',
  artist: 'Alina Szapocznikow',
  description: 'Szapocznikow 以其碎片化身体雕塑而闻名，这些作品将其作为青春与美丽脆弱和短暂的隐喻。这座雕塑描绘了两个非常逼真的大肚子彼此叠放，每个高约五英尺（1.5 米）。',
  url: 'https://react.dev/images/docs/scientists/AlHTAdDm.jpg',
  alt: '这座雕塑让人联想到一层层褶皱，与古典雕塑中的肚子截然不同。'
}, {
  name: 'Terracotta Army',
  artist: 'Unknown Artist',
  description: '兵马俑是一个由陶俑组成的收藏，描绘了秦始皇的军队。这支军队由 8,000 多名士兵、130 辆战车和 520 匹战马、以及 150 匹骑兵马组成。',
  url: 'https://react.dev/images/docs/scientists/HMFmH6m.jpg',
  alt: '12 尊陶俑形态的庄严武士雕塑，每尊都有独特的面部表情和盔甲。'
}, {
  name: 'Lunar Landscape',
  artist: 'Louise Nevelson',
  description: 'Nevelson 以从纽约市废弃物中搜集物件而闻名，随后将其组装成纪念性构筑物。在这件作品中，她使用了床柱、杂耍球棒和座椅碎片等不同部分，将它们钉合并粘接到盒子里，体现了立体主义对空间与形式的几何抽象所带来的影响。',
  url: 'https://react.dev/images/docs/scientists/rN7hY6om.jpg',
  alt: '一座哑光黑色雕塑，各个元素最初难以区分。'
}, {
  name: 'Aureole',
  artist: 'Ranjani Shettar',
  description: 'Shettar 将传统与现代、自然与工业相融合。她的艺术专注于人与自然之间的关系。她的作品被形容为抽象与具象皆引人注目、违背重力，以及“一种对不太可能材料的精妙综合”。',
  url: 'https://react.dev/images/docs/scientists/okTpbHhm.jpg',
  alt: '一座浅色、如金属丝般的雕塑安装在混凝土墙上并垂落到地面，看起来很轻盈。'
}, {
  name: 'Hippos',
  artist: 'Taipei Zoo',
  description: '台北动物园委托创作了一个河马广场，展示了一群在水中嬉戏的半 submerged 河马。',
  url: 'https://react.dev/images/docs/scientists/6o5Vuyu.jpg',
  alt: '一组青铜河马雕塑从铺石人行道中冒出，仿佛它们正在游泳。'
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

### 认识你的第一个 Hook {/*meet-your-first-hook*/}

在 React 中，`useState`，以及任何以 "`use`" 开头的其他函数，都被称为 Hook。

*Hook* 是特殊的函数，只能在 React [渲染](/learn/render-and-commit#step-1-trigger-a-render) 时使用（我们会在下一页更详细地讨论这一点）。它们让你“挂接”到不同的 React 特性上。

State 只是这些特性之一，不过你稍后还会认识其他 Hook。

<Pitfall>

**Hook——以 `use` 开头的函数——只能在组件顶层或[你自己的 Hook](/learn/reusing-logic-with-custom-hooks) 顶层调用。** 你不能在条件语句、循环或其他嵌套函数中调用 Hook。Hook 是函数，但把它们看作关于组件需求的无条件声明会更有帮助。你在组件顶部“使用” React 特性的方式，类似于你在文件顶部“导入”模块的方式。

</Pitfall>

### `useState` 的结构 {/*anatomy-of-usestate*/}

当你调用 [`useState`](/reference/react/useState) 时，你是在告诉 React，你希望这个组件记住一些东西：

```js
const [index, setIndex] = useState(0);
```

在这个例子中，你希望 React 记住 `index`。

<Note>

惯例是把这一对命名为 `const [something, setSomething]`。你当然可以随意命名，但遵循惯例会让不同项目之间更容易理解。

</Note>

`useState` 唯一的参数是你的状态变量的**初始值**。在这个例子中，`index` 的初始值通过 `useState(0)` 被设为 `0`。

每次组件渲染时，`useState` 都会给你一个包含两个值的数组：

1. **状态变量**（`index`），其值就是你存储的值。
2. **状态设置函数**（`setIndex`），它可以更新状态变量并触发 React 再次渲染组件。

下面是它在实际中的工作方式：

```js
const [index, setIndex] = useState(0);
```

1. **你的组件第一次渲染。** 因为你把 `0` 作为 `index` 的初始值传给了 `useState`，它会返回 `[0, setIndex]`。React 记住 `0` 是最新的状态值。
2. **你更新状态。** 当用户点击按钮时，会调用 `setIndex(index + 1)`。`index` 是 `0`，所以这就是 `setIndex(1)`。这告诉 React 现在要记住 `index` 是 `1`，并触发另一次渲染。
3. **组件的第二次渲染。** React 仍然看到的是 `useState(0)`，但因为 React *记住* 了你把 `index` 设为 `1`，所以它返回的是 `[1, setIndex]`。
4. 以此类推！

## 为组件设置多个状态变量 {/*giving-a-component-multiple-state-variables*/}

你可以在一个组件中拥有任意数量、任意类型的状态变量。这个组件有两个状态变量，一个数字 `index` 和一个布尔值 `showMore`，当你点击“显示详情”时它会切换：

<Sandpack>

```js
import { useState } from 'react';
import { sculptureList } from './data.js';

export default function Gallery() {
  const [index, setIndex] = useState(0);
  const [showMore, setShowMore] = useState(false);

  function handleNextClick() {
    setIndex(index + 1);
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
        作者：{sculpture.artist}
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
  name: 'Homenaje a la Neurocirugía',
  artist: 'Marta Colvin Andrade',
  description: '虽然 Colvin 主要以暗指前西班牙时期符号的抽象主题而闻名，但这座巨大的雕塑——对神经外科的致敬——是她最具辨识度的公共艺术作品之一。',
  url: 'https://react.dev/images/docs/scientists/Mx7dA2Y.jpg',
  alt: '一尊青铜雕像，两个交叉的手指尖轻柔地托着一颗人脑。'
}, {
  name: 'Floralis Genérica',
  artist: 'Eduardo Catalano',
  description: '这朵巨大的（75 英尺，约 23 米）银色花朵位于布宜诺斯艾利斯。它被设计成会移动，傍晚或强风吹拂时会闭合花瓣，清晨则会重新开放。',
  url: 'https://react.dev/images/docs/scientists/ZF6s192m.jpg',
  alt: '一座巨大的金属花朵雕塑，带有反光、镜面般的花瓣和粗壮的花蕊。'
}, {
  name: 'Eternal Presence',
  artist: 'John Woodrow Wilson',
  description: 'Wilson 以其对平等、社会正义以及人类本质和精神特质的关注而闻名。这件巨大的青铜雕塑（7 英尺，约 2.13 米）代表了他所描述的“被赋予普遍人性意识的象征性黑人存在”。',
  url: 'https://react.dev/images/docs/scientists/aTtVpES.jpg',
  alt: '这座描绘人头的雕塑似乎无处不在且庄严肃穆，散发着平静与安宁。'
}, {
  name: 'Moai',
  artist: 'Unknown Artist',
  description: '位于复活节岛的摩艾石像共有 1,000 尊，或者说是现存的纪念性雕像，由早期的拉帕努伊人创造，一些人认为它们代表着神化的祖先。',
  url: 'https://react.dev/images/docs/scientists/RCwLEoQm.jpg',
  alt: '三尊纪念性的石质半身像，头部异常巨大，表情忧郁。'
}, {
  name: 'Blue Nana',
  artist: 'Niki de Saint Phalle',
  description: 'Nana 是欢欣鼓舞的生灵，象征着女性气质与母性。起初，Saint Phalle 为 Nana 使用织物和现成物件，后来又引入聚酯材料，以获得更鲜艳的效果。',
  url: 'https://react.dev/images/docs/scientists/Sd1AgUOm.jpg',
  alt: '一座大型马赛克雕塑，描绘了一个穿着彩色服装、跳舞的奇幻女性形象，洋溢着欢乐。'
}, {
  name: 'Ultimate Form',
  artist: 'Barbara Hepworth',
  description: '这座抽象青铜雕塑是位于约克郡雕塑公园的《人类大家庭》系列的一部分。Hepworth 选择不去创作世界的写实再现，而是发展出受人物与风景启发的抽象形式。',
  url: 'https://react.dev/images/docs/scientists/2heNQDcm.jpg',
  alt: '一座由三个元素堆叠而成的高耸雕塑，让人联想到人体轮廓。'
}, {
  name: 'Cavaliere',
  artist: 'Lamidi Olonade Fakeye',
  description: 'Fakeye 的作品传承自四代木雕匠人，融合了传统与当代约鲁巴主题。',
  url: 'https://react.dev/images/docs/scientists/wIdGuZwm.png',
  alt: '一座精致的木雕，描绘了一位面容专注的骑马战士，身上装饰着纹样。'
}, {
  name: 'Big Bellies',
  artist: 'Alina Szapocznikow',
  description: 'Szapocznikow 以其碎片化身体雕塑而闻名，这些作品将身体作为青春与美丽脆弱和短暂的隐喻。这座雕塑描绘了两个非常逼真的大肚子，一个叠在另一个上面，每个大约五英尺（1.5 米）高。',
  url: 'https://react.dev/images/docs/scientists/AlHTAdDm.jpg',
  alt: '这座雕塑让人联想到一层层褶皱，与古典雕塑中的腹部形象大不相同。'
}, {
  name: 'Terracotta Army',
  artist: 'Unknown Artist',
  description: '兵马俑是描绘秦始皇军队的陶俑雕塑集合。这支军队由 8,000 多名士兵、130 辆战车和 520 匹马，以及 150 匹骑兵马组成。',
  url: 'https://react.dev/images/docs/scientists/HMFmH6m.jpg',
  alt: '12 尊陶俑战士雕塑，每一尊都有独特的面部表情和盔甲。'
}, {
  name: 'Lunar Landscape',
  artist: 'Louise Nevelson',
  description: 'Nevelson 以从纽约市废弃物中搜集物件而闻名，随后她会将它们组装成纪念性构筑物。在这件作品中，她使用了床柱、杂耍棒和座椅碎片等互不相同的部件，将它们钉上并粘进盒子里，体现了立体主义对空间与形式几何抽象的影响。',
  url: 'https://react.dev/images/docs/scientists/rN7hY6om.jpg',
  alt: '一座哑光黑色雕塑，单个元素起初难以分辨。'
}, {
  name: 'Aureole',
  artist: 'Ranjani Shettar',
  description: 'Shettar 融合了传统与现代、自然与工业。她的艺术聚焦于人与自然之间的关系。她的作品被形容为在抽象和具象层面都引人入胜、违背重力，以及“对不太可能材料的精妙综合”。',
  url: 'https://react.dev/images/docs/scientists/okTpbHhm.jpg',
  alt: '一座浅色、类似金属丝的雕塑安装在混凝土墙上并向地面垂下，看起来很轻盈。'
}, {
  name: 'Hippos',
  artist: 'Taipei Zoo',
  description: '台北动物园委托创作了一个河马广场，展示了半浸没在水中的河马嬉戏场景。',
  url: 'https://react.dev/images/docs/scientists/6o5Vuyu.jpg',
  alt: '一组青铜河马雕塑从人行道的缝隙中冒出，仿佛它们正在游泳。'
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

如果某些状态彼此无关，比如这个例子中的 `index` 和 `showMore`，那么拥有多个状态变量是个好主意。但如果你发现自己经常需要同时修改两个状态变量，那么把它们合并成一个可能会更容易。例如，如果你有一个包含许多字段的表单，那么拥有一个保存对象的单一状态变量，比为每个字段分别设置状态变量更方便。阅读 [选择状态结构](/learn/choosing-the-state-structure) 以获取更多提示。

<DeepDive>

#### React 如何知道要返回哪个状态？ {/*how-does-react-know-which-state-to-return*/}

你可能已经注意到，`useState` 调用并没有接收任何关于它所引用的是*哪个*状态变量的信息。传给 `useState` 的并没有“标识符”，那它是怎么知道要返回哪个状态变量的呢？它依赖于解析你的函数之类的魔法吗？答案是否定的。

相反，为了保持其简洁的语法，Hooks **依赖于同一个组件每次渲染时稳定的调用顺序。** 这在实践中非常有效，因为如果你遵循上面的规则（“只在顶层调用 Hooks”），Hooks 总会以相同的顺序被调用。此外，[linter 插件](https://www.npmjs.com/package/eslint-plugin-react-hooks) 可以捕获大多数错误。

在内部，React 为每个组件维护一个状态对数组。它还维护当前的配对索引，在渲染前该索引被设置为 `0`。每次你调用 `useState`，React 都会给你下一个状态对并递增索引。你可以在 [React Hooks：不是魔法，只是数组。](https://medium.com/@ryardley/react-hooks-not-magic-just-arrays-cd4f1857236e) 中了解更多关于这一机制的内容。

这个例子**没有使用 React**，但它能让你对 `useState` 的内部工作方式有一个概念：

<Sandpack>

```js src/index.js active
let componentHooks = [];
let currentHookIndex = 0;

// useState 在 React 内部是如何工作的（简化版）。
function useState(initialState) {
  let pair = componentHooks[currentHookIndex];
  if (pair) {
    // 这不是第一次渲染，
    // 所以状态对已经存在。
    // 返回它并为下一次 Hook 调用做准备。
    currentHookIndex++;
    return pair;
  }

  // 这是我们第一次渲染，
  // 所以创建一个状态对并存储它。
  pair = [initialState, setState];

  function setState(nextState) {
    // 当用户请求更改状态时，
    // 将新值放入该状态对中。
    pair[0] = nextState;
    updateDOM();
  }

  // 为将来的渲染保存这个状态对
  // 并为下一次 Hook 调用做准备。
  componentHooks[currentHookIndex] = pair;
  currentHookIndex++;
  return pair;
}

function Gallery() {
  // 每次 useState() 调用都会获得下一个状态对。
  const [index, setIndex] = useState(0);
  const [showMore, setShowMore] = useState(false);

  function handleNextClick() {
    setIndex(index + 1);
  }

  function handleMoreClick() {
    setShowMore(!showMore);
  }

  let sculpture = sculptureList[index];
  // 这个例子没有使用 React，所以
  // 返回一个输出对象而不是 JSX。
  return {
    onNextClick: handleNextClick,
    onMoreClick: handleMoreClick,
    header: `${sculpture.name} by ${sculpture.artist}`,
    counter: `${index + 1} of ${sculptureList.length}`,
    more: `${showMore ? 'Hide' : 'Show'} details`,
    description: showMore ? sculpture.description : null,
    imageSrc: sculpture.url,
    imageAlt: sculpture.alt
  };
}

function updateDOM() {
  // 在渲染组件之前，
  // 重置当前 Hook 索引。
  currentHookIndex = 0;
  let output = Gallery();

  // 更新 DOM 以匹配输出。
  // 这是 React 为你完成的部分。
  nextButton.onclick = output.onNextClick;
  header.textContent = output.header;
  moreButton.onclick = output.onMoreClick;
  moreButton.textContent = output.more;
  image.src = output.imageSrc;
  image.alt = output.imageAlt;
  if (output.description !== null) {
    description.textContent = output.description;
    description.style.display = '';
  } else {
    description.style.display = 'none';
  }
}

let nextButton = document.getElementById('nextButton');
let header = document.getElementById('header');
let moreButton = document.getElementById('moreButton');
let description = document.getElementById('description');
let image = document.getElementById('image');
let sculptureList = [{
  name: 'Homenaje a la Neurocirugía',
  artist: 'Marta Colvin Andrade',
  description: '虽然 Colvin 主要以暗指前西班牙时期符号的抽象主题而闻名，但这座巨大的雕塑——对神经外科的致敬——是她最具辨识度的公共艺术作品之一。',
  url: 'https://react.dev/images/docs/scientists/Mx7dA2Y.jpg',
  alt: '一尊青铜雕像，两个交叉的手指尖轻柔地托着一颗人脑。'
}, {
  name: 'Floralis Genérica',
  artist: 'Eduardo Catalano',
  description: '这朵巨大的（75 英尺，约 23 米）银色花朵位于布宜诺斯艾利斯。它被设计成会移动，傍晚或强风吹拂时会闭合花瓣，清晨则会重新开放。',
  url: 'https://react.dev/images/docs/scientists/ZF6s192m.jpg',
  alt: '一座巨大的金属花朵雕塑，带有反光、镜面般的花瓣和粗壮的花蕊。'
}, {
  name: 'Eternal Presence',
  artist: 'John Woodrow Wilson',
  description: 'Wilson 以其对平等、社会正义以及人类本质和精神特质的关注而闻名。这件巨大的青铜雕塑（7 英尺，约 2.13 米）代表了他所描述的“被赋予普遍人性意识的象征性黑人存在”。',
  url: 'https://react.dev/images/docs/scientists/aTtVpES.jpg',
  alt: '这座描绘人头的雕塑似乎无处不在且庄严肃穆，散发着平静与安宁。'
}, {
  name: 'Moai',
  artist: 'Unknown Artist',
  description: '位于复活节岛的摩艾石像共有 1,000 尊，或者说是现存的纪念性雕像，由早期的拉帕努伊人创造，一些人认为它们代表着神化的祖先。',
  url: 'https://react.dev/images/docs/scientists/RCwLEoQm.jpg',
  alt: '三尊纪念性的石质半身像，头部异常巨大，表情忧郁。'
}, {
  name: 'Blue Nana',
  artist: 'Niki de Saint Phalle',
  description: 'Nana 是欢欣鼓舞的生灵，象征着女性气质与母性。起初，Saint Phalle 为 Nana 使用织物和现成物件，后来又引入聚酯材料，以获得更鲜艳的效果。',
  url: 'https://react.dev/images/docs/scientists/Sd1AgUOm.jpg',
  alt: '一座大型马赛克雕塑，描绘了一个穿着彩色服装、跳舞的奇幻女性形象，洋溢着欢乐。'
}, {
  name: 'Ultimate Form',
  artist: 'Barbara Hepworth',
  description: '这座抽象青铜雕塑是位于约克郡雕塑公园的《人类大家庭》系列的一部分。Hepworth 选择不去创作世界的写实再现，而是发展出受人物与风景启发的抽象形式。',
  url: 'https://react.dev/images/docs/scientists/2heNQDcm.jpg',
  alt: '一座由三个元素堆叠而成的高耸雕塑，让人联想到人体轮廓。'
}, {
  name: 'Cavaliere',
  artist: 'Lamidi Olonade Fakeye',
  description: 'Fakeye 的作品传承自四代木雕匠人，融合了传统与当代约鲁巴主题。',
  url: 'https://react.dev/images/docs/scientists/wIdGuZwm.png',
  alt: '一座精致的木雕，描绘了一位面容专注的骑马战士，身上装饰着纹样。'
}, {
  name: 'Big Bellies',
  artist: 'Alina Szapocznikow',
  description: 'Szapocznikow 以其碎片化身体雕塑而闻名，这些作品将身体作为青春与美丽脆弱和短暂的隐喻。这座雕塑描绘了两个非常逼真的大肚子，一个叠在另一个上面，每个大约五英尺（1.5 米）高。',
  url: 'https://react.dev/images/docs/scientists/AlHTAdDm.jpg',
  alt: '这座雕塑让人联想到一层层褶皱，与古典雕塑中的腹部形象大不相同。'
}, {
  name: 'Terracotta Army',
  artist: 'Unknown Artist',
  description: '兵马俑是描绘秦始皇军队的陶俑雕塑集合。这支军队由 8,000 多名士兵、130 辆战车和 520 匹马，以及 150 匹骑兵马组成。',
  url: 'https://react.dev/images/docs/scientists/HMFmH6m.jpg',
  alt: '12 尊陶俑战士雕塑，每一尊都有独特的面部表情和盔甲。'
}, {
  name: 'Lunar Landscape',
  artist: 'Louise Nevelson',
  description: 'Nevelson 以从纽约市废弃物中搜集物件而闻名，随后她会将它们组装成纪念性构筑物。在这件作品中，她使用了床柱、杂耍棒和座椅碎片等互不相同的部件，将它们钉上并粘进盒子里，体现了立体主义对空间与形式几何抽象的影响。',
  url: 'https://react.dev/images/docs/scientists/rN7hY6om.jpg',
  alt: '一座哑光黑色雕塑，单个元素起初难以分辨。'
}, {
  name: 'Aureole',
  artist: 'Ranjani Shettar',
  description: 'Shettar 融合了传统与现代、自然与工业。她的艺术聚焦于人与自然之间的关系。她的作品被形容为在抽象和具象层面都引人入胜、违背重力，以及“对不太可能材料的精妙综合”。',
  url: 'https://react.dev/images/docs/scientists/okTpbHhm.jpg',
  alt: '一座浅色、类似金属丝的雕塑安装在混凝土墙上并向地面垂下，看起来很轻盈。'
}, {
  name: 'Hippos',
  artist: 'Taipei Zoo',
  description: '台北动物园委托创作了一个河马广场，展示了半浸没在水中的河马嬉戏场景。',
  url: 'https://react.dev/images/docs/scientists/6o5Vuyu.jpg',
  alt: '一组青铜河马雕塑从人行道的缝隙中冒出，仿佛它们正在游泳。'
}];

// 使 UI 与初始状态保持一致。
updateDOM();
```

```html public/index.html
<button id="nextButton">
  下一个
</button>
<h3 id="header"></h3>
<button id="moreButton"></button>
<p id="description"></p>
<img id="image">

<style>
* { box-sizing: border-box; }
body { font-family: sans-serif; margin: 20px; padding: 0; }
button { display: block; margin-bottom: 10px; }
</style>
```

```css
button { display: block; margin-bottom: 10px; }
```

</Sandpack>

你不必理解它也能使用 React，但你可能会发现这是一种有帮助的心智模型。

</DeepDive>

## 状态是隔离且私有的 {/*state-is-isolated-and-private*/}

状态只属于屏幕上某个组件实例。换句话说，**如果你把同一个组件渲染两次，那么每个副本都会拥有完全隔离的状态！** 修改其中一个不会影响另一个。

在这个示例中，前面提到的 `Gallery` 组件被渲染了两次，但其逻辑没有任何变化。试着点击每个画廊里的按钮。注意它们的状态是彼此独立的：

<Sandpack>

```js
import Gallery from './Gallery.js';

export default function Page() {
  return (
    <div className="Page">
      <Gallery />
      <Gallery />
    </div>
  );
}

```

```js src/Gallery.js
import { useState } from 'react';
import { sculptureList } from './data.js';

export default function Gallery() {
  const [index, setIndex] = useState(0);
  const [showMore, setShowMore] = useState(false);

  function handleNextClick() {
    setIndex(index + 1);
  }

  function handleMoreClick() {
    setShowMore(!showMore);
  }

  let sculpture = sculptureList[index];
  return (
    <section>
      <button onClick={handleNextClick}>
        下一张
      </button>
      <h2>
        <i>{sculpture.name} </i>
        作者：{sculpture.artist}
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
    </section>
  );
}
```

```js src/data.js
export const sculptureList = [{
  name: 'Homenaje a la Neurocirugía',
  artist: 'Marta Colvin Andrade',
  description: '虽然 Colvin 主要以暗示前西班牙时期符号的抽象主题而闻名，但这件巨大的雕塑是对神经外科的致敬，是她最具辨识度的公共艺术作品之一。',
  url: 'https://react.dev/images/docs/scientists/Mx7dA2Y.jpg',
  alt: '一尊青铜雕像，两个交叉的手指尖轻柔地托着一颗人脑。'
}, {
  name: 'Floralis Genérica',
  artist: 'Eduardo Catalano',
  description: '这朵巨大的银色花朵（75 英尺，约 23 米）位于布宜诺斯艾利斯。它会活动：在傍晚或强风时闭合花瓣，在早晨开放。',
  url: 'https://react.dev/images/docs/scientists/ZF6s192m.jpg',
  alt: '一座巨大的金属花朵雕塑，带有反光镜面般的花瓣和粗壮的花蕊。'
}, {
  name: 'Eternal Presence',
  artist: 'John Woodrow Wilson',
  description: 'Wilson 以关注平等、社会正义，以及人类本质与精神品质而闻名。这座巨大的青铜雕塑（7 英尺，约 2.13 米）体现了他所描述的“被赋予普遍人性意识的象征性黑人形象”。',
  url: 'https://react.dev/images/docs/scientists/aTtVpES.jpg',
  alt: '这座描绘人头的雕塑仿佛始终存在，庄严肃穆。它散发着平静与安宁。'
}, {
  name: 'Moai',
  artist: 'Unknown Artist',
  description: '位于复活节岛上，共有 1,000 尊摩艾石像，即现存的纪念性雕像，由早期拉帕努伊人创造，一些人认为它们代表着神化的祖先。',
  url: 'https://react.dev/images/docs/scientists/RCwLEoQm.jpg',
  alt: '三座纪念性石质半身像，头部比例极大，面容凝重。'
}, {
  name: 'Blue Nana',
  artist: 'Niki de Saint Phalle',
  description: 'Nana 是象征女性气质与母性的胜利生物。起初，Saint Phalle 为 Nana 使用布料和现成物品，后来又引入聚酯材料以获得更鲜活的效果。',
  url: 'https://react.dev/images/docs/scientists/Sd1AgUOm.jpg',
  alt: '一座大型马赛克雕塑，描绘一个穿着彩色服装、充满奇思妙想的舞动女性形象，洋溢着喜悦。'
}, {
  name: 'Ultimate Form',
  artist: 'Barbara Hepworth',
  description: '这件抽象青铜雕塑是位于约克郡雕塑公园的《人类大家庭》系列的一部分。Hepworth 并未选择创作世界的写实再现，而是发展出受人和风景启发的抽象形式。',
  url: 'https://react.dev/images/docs/scientists/2heNQDcm.jpg',
  alt: '一座由三个元素堆叠而成的高挑雕塑，让人联想到人体形象。'
}, {
  name: 'Cavaliere',
  artist: 'Lamidi Olonade Fakeye',
  description: 'Fakeye 来自四代木雕世家，他的作品融合了传统与当代约鲁巴主题。',
  url: 'https://react.dev/images/docs/scientists/wIdGuZwm.png',
  alt: '一座精致的木雕，表现一名神情专注的武士骑在装饰着纹样的马上。'
}, {
  name: 'Big Bellies',
  artist: 'Alina Szapocznikow',
  description: 'Szapocznikow 以人体碎片化雕塑著称，将其作为青春与美丽脆弱和短暂的隐喻。这件雕塑展示了两个非常逼真的大肚子上下堆叠在一起，每个大约五英尺（1.5 米）高。',
  url: 'https://react.dev/images/docs/scientists/AlHTAdDm.jpg',
  alt: '这件雕塑让人联想到一连串褶皱，与古典雕塑中的腹部形象截然不同。'
}, {
  name: 'Terracotta Army',
  artist: 'Unknown Artist',
  description: '兵马俑是一组描绘秦始皇军队的陶俑雕塑。军队由 8,000 多名士兵、130 辆战车和 520 匹战马以及 150 匹骑兵马组成。',
  url: 'https://react.dev/images/docs/scientists/HMFmH6m.jpg',
  alt: '12 尊陶俑武士雕塑，每一尊都有独特的面部表情和盔甲。'
}, {
  name: 'Lunar Landscape',
  artist: 'Louise Nevelson',
  description: 'Nevelson 以从纽约市废墟中搜集物件而闻名，她后来将这些物件组装成纪念性构筑物。在这件作品中，她使用了床柱、杂耍棒、座椅碎片等不同部件，并将它们钉接和粘合到盒子中，体现了立体主义对空间与形式的几何抽象影响。',
  url: 'https://react.dev/images/docs/scientists/rN7hY6om.jpg',
  alt: '一座黑色哑光雕塑，各个元素最初难以分辨。'
}, {
  name: 'Aureole',
  artist: 'Ranjani Shettar',
  description: 'Shettar 融合了传统与现代、自然与工业。她的艺术聚焦于人与自然的关系。她的作品被描述为在抽象和具象层面都引人注目、违背重力，以及“对不太可能的材料的精妙综合”。',
  url: 'https://react.dev/images/docs/scientists/okTpbHhm.jpg',
  alt: '一座浅色、像细线一样的雕塑安装在混凝土墙上并垂落到地面，看起来很轻盈。'
}, {
  name: 'Hippos',
  artist: 'Taipei Zoo',
  description: '台北动物园委托创作了一个河马广场，展示了半 submerged 的河马在玩耍。',
  url: 'https://react.dev/images/docs/scientists/6o5Vuyu.jpg',
  alt: '一组青铜河马雕塑从铺装人行道中冒出，仿佛正在游泳。'
}];
```

```css
button { display: block; margin-bottom: 10px; }
.Page > * {
  float: left;
  width: 50%;
  padding: 10px;
}
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

这就是为什么状态不同于你可能在模块顶部声明的普通变量。状态并不绑定到某个特定函数调用或代码中的某个位置，而是“局部”于屏幕上的特定位置。你渲染了两个 `<Gallery />` 组件，所以它们的状态是分别存储的。

还要注意，`Page` 组件并不知道任何关于 `Gallery` 状态的事情，甚至不知道它是否有状态。不同于 props，**状态对声明它的组件来说是完全私有的。** 父组件不能修改它。这让你可以给任意组件添加状态，或者移除状态，而不会影响其他组件。

如果你希望两个画廊保持同步，该怎么做？在 React 中正确的做法是从子组件中*移除*状态，并将其提升到最近的共享父组件中。接下来的几页将聚焦于如何组织单个组件的状态，但我们会在[组件间共享状态。](/learn/sharing-state-between-components)中回到这个话题

<Recap>

* 当组件需要在多次渲染之间“记住”某些信息时，使用状态变量。
* 状态变量通过调用 `useState` Hook 来声明。
* Hook 是以 `use` 开头的特殊函数。它们让你能够“接入” React 的特性，比如状态。
* Hook 可能会让你联想到导入：它们必须无条件地调用。调用 Hook，包括 `useState`，只在组件或另一个 Hook 的顶层有效。
* `useState` Hook 返回一对值：当前状态和更新它的函数。
* 你可以有多个状态变量。React 在内部根据它们的顺序来匹配它们。
* 状态对组件是私有的。如果你在两个地方渲染它，每个副本都会获得自己的状态。

</Recap>



<Challenges>

#### 完成画廊 {/*complete-the-gallery*/}

当你按下最后一件雕塑上的“下一张”时，代码会崩溃。修复逻辑以防止崩溃。你可以通过在事件处理器中添加额外逻辑来实现，或者在动作不可用时禁用按钮。

修复崩溃后，再添加一个“上一张”按钮来显示上一件雕塑。它不应该在第一件雕塑上崩溃。

<Sandpack>

```js
import { useState } from 'react';
import { sculptureList } from './data.js';

export default function Gallery() {
  const [index, setIndex] = useState(0);
  const [showMore, setShowMore] = useState(false);

  function handleNextClick() {
    setIndex(index + 1);
  }

  function handleMoreClick() {
    setShowMore(!showMore);
  }

  let sculpture = sculptureList[index];
  return (
    <>
      <button onClick={handleNextClick}>
        下一张
      </button>
      <h2>
        <i>{sculpture.name} </i>
        作者：{sculpture.artist}
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
  name: 'Homenaje a la Neurocirugía',
  artist: 'Marta Colvin Andrade',
  description: '虽然 Colvin 主要以暗示前西班牙时期符号的抽象主题而闻名，但这件巨大的雕塑是对神经外科的致敬，是她最具辨识度的公共艺术作品之一。',
  url: 'https://react.dev/images/docs/scientists/Mx7dA2Y.jpg',
  alt: '一尊青铜雕像，两个交叉的手指尖轻柔地托着一颗人脑。'
}, {
  name: 'Floralis Genérica',
  artist: 'Eduardo Catalano',
  description: '这朵巨大的银色花朵（75 英尺，约 23 米）位于布宜诺斯艾利斯。它会活动：在傍晚或强风时闭合花瓣，在早晨开放。',
  url: 'https://react.dev/images/docs/scientists/ZF6s192m.jpg',
  alt: '一座巨大的金属花朵雕塑，带有反光镜面般的花瓣和粗壮的花蕊。'
}, {
  name: 'Eternal Presence',
  artist: 'John Woodrow Wilson',
  description: 'Wilson 以关注平等、社会正义，以及人类本质与精神品质而闻名。这座巨大的青铜雕塑（7 英尺，约 2.13 米）体现了他所描述的“被赋予普遍人性意识的象征性黑人形象”。',
  url: 'https://react.dev/images/docs/scientists/aTtVpES.jpg',
  alt: '这座描绘人头的雕塑仿佛始终存在，庄严肃穆。它散发着平静与安宁。'
}, {
  name: 'Moai',
  artist: 'Unknown Artist',
  description: '位于复活节岛上，共有 1,000 尊摩艾石像，即现存的纪念性雕像，由早期拉帕努伊人创造，一些人认为它们代表着神化的祖先。',
  url: 'https://react.dev/images/docs/scientists/RCwLEoQm.jpg',
  alt: '三座纪念性石质半身像，头部比例极大，面容凝重。'
}, {
  name: 'Blue Nana',
  artist: 'Niki de Saint Phalle',
  description: 'Nana 是象征女性气质与母性的胜利生物。起初，Saint Phalle 为 Nana 使用布料和现成物品，后来又引入聚酯材料以获得更鲜活的效果。',
  url: 'https://react.dev/images/docs/scientists/Sd1AgUOm.jpg',
  alt: '一座大型马赛克雕塑，描绘一个穿着彩色服装、充满奇思妙想的舞动女性形象，洋溢着喜悦。'
}, {
  name: 'Ultimate Form',
  artist: 'Barbara Hepworth',
  description: '这件抽象青铜雕塑是位于约克郡雕塑公园的《人类大家庭》系列的一部分。Hepworth 并未选择创作世界的写实再现，而是发展出受人和风景启发的抽象形式。',
  url: 'https://react.dev/images/docs/scientists/2heNQDcm.jpg',
  alt: '一座由三个元素堆叠而成的高挑雕塑，让人联想到人体形象。'
}, {
  name: 'Cavaliere',
  artist: 'Lamidi Olonade Fakeye',
  description: 'Fakeye 来自四代木雕世家，他的作品融合了传统与当代约鲁巴主题。',
  url: 'https://react.dev/images/docs/scientists/wIdGuZwm.png',
  alt: '一座精致的木雕，表现一名神情专注的武士骑在装饰着纹样的马上。'
}, {
  name: 'Big Bellies',
  artist: 'Alina Szapocznikow',
  description: 'Szapocznikow 以人体碎片化雕塑著称，将其作为青春与美丽脆弱和短暂的隐喻。这件雕塑展示了两个非常逼真的大肚子上下堆叠在一起，每个大约五英尺（1.5 米）高。',
  url: 'https://react.dev/images/docs/scientists/AlHTAdDm.jpg',
  alt: '这件雕塑让人联想到一连串褶皱，与古典雕塑中的腹部形象截然不同。'
}, {
  name: 'Terracotta Army',
  artist: 'Unknown Artist',
  description: '兵马俑是一组描绘秦始皇军队的陶俑雕塑。军队由 8,000 多名士兵、130 辆战车和 520 匹战马以及 150 匹骑兵马组成。',
  url: 'https://react.dev/images/docs/scientists/HMFmH6m.jpg',
  alt: '12 尊陶俑武士雕塑，每一尊都有独特的面部表情和盔甲。'
}, {
  name: 'Lunar Landscape',
  artist: 'Louise Nevelson',
  description: 'Nevelson 以从纽约市废墟中搜集物件而闻名，她后来将这些物件组装成纪念性构筑物。在这件作品中，她使用了床柱、杂耍棒、座椅碎片等不同部件，并将它们钉接和粘合到盒子中，体现了立体主义对空间与形式的几何抽象影响。',
  url: 'https://react.dev/images/docs/scientists/rN7hY6om.jpg',
  alt: '一座黑色哑光雕塑，各个元素最初难以分辨。'
}, {
  name: 'Aureole',
  artist: 'Ranjani Shettar',
  description: 'Shettar 融合了传统与现代、自然与工业。她的艺术聚焦于人与自然的关系。她的作品被描述为在抽象和具象层面都引人注目、违背重力，以及“对不太可能的材料的精妙综合”。',
  url: 'https://react.dev/images/docs/scientists/okTpbHhm.jpg',
  alt: '一座浅色、像细线一样的雕塑安装在混凝土墙上并垂落到地面，看起来很轻盈。'
}, {
  name: 'Hippos',
  artist: 'Taipei Zoo',
  description: '台北动物园委托创作了一个河马广场，展示了半 submerged 的河马在玩耍。',
  url: 'https://react.dev/images/docs/scientists/6o5Vuyu.jpg',
  alt: '一组青铜河马雕塑从铺装人行道中冒出，仿佛正在游泳。'
}];
```

```css
button { display: block; margin-bottom: 10px; }
.Page > * {
  float: left;
  width: 50%;
  padding: 10px;
}
h2 { margin-top: 10px; margin-bottom: 0; }
h3 {
  margin-top: 5px;
  font-weight: normal;
  font-size: 100%;
}
img { width: 120px; height: 120px; }
```

</Sandpack>

<Solution>

这会在两个事件处理器中都添加保护条件，并在需要时禁用按钮：

<Sandpack>

```js
import { useState } from 'react';
import { sculptureList } from './data.js';

export default function Gallery() {
  const [index, setIndex] = useState(0);
  const [showMore, setShowMore] = useState(false);

  let hasPrev = index > 0;
  let hasNext = index < sculptureList.length - 1;

  function handlePrevClick() {
    if (hasPrev) {
      setIndex(index - 1);
    }
  }

  function handleNextClick() {
    if (hasNext) {
      setIndex(index + 1);
    }
  }

  function handleMoreClick() {
    setShowMore(!showMore);
  }

  let sculpture = sculptureList[index];
  return (
    <>
      <button
        onClick={handlePrevClick}
        disabled={!hasPrev}
      >
        上一张
      </button>
      <button
        onClick={handleNextClick}
        disabled={!hasNext}
      >
        下一张
      </button>
      <h2>
        <i>{sculpture.name} </i>
        作者：{sculpture.artist}
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

```js src/data.js hidden
export const sculptureList = [{
  name: 'Homenaje a la Neurocirugía',
  artist: 'Marta Colvin Andrade',
  description: '虽然 Colvin 主要以暗示前西班牙时期符号的抽象主题而闻名，但这件巨大的雕塑是对神经外科的致敬，是她最具辨识度的公共艺术作品之一。',
  url: 'https://react.dev/images/docs/scientists/Mx7dA2Y.jpg',
  alt: '一尊青铜雕像，两个交叉的手指尖轻柔地托着一颗人脑。'
}, {
  name: 'Floralis Genérica',
  artist: 'Eduardo Catalano',
  description: '这朵巨大的银色花朵（75 英尺，约 23 米）位于布宜诺斯艾利斯。它会活动：在傍晚或强风时闭合花瓣，在早晨开放。',
  url: 'https://react.dev/images/docs/scientists/ZF6s192m.jpg',
  alt: '一座巨大的金属花朵雕塑，带有反光镜面般的花瓣和粗壮的花蕊。'
}, {
  name: 'Eternal Presence',
  artist: 'John Woodrow Wilson',
  description: 'Wilson 以关注平等、社会正义，以及人类本质与精神品质而闻名。这座巨大的青铜雕塑（7 英尺，约 2.13 米）体现了他所描述的“被赋予普遍人性意识的象征性黑人形象”。',
  url: 'https://react.dev/images/docs/scientists/aTtVpES.jpg',
  alt: '这座描绘人头的雕塑仿佛始终存在，庄严肃穆。它散发着平静与安宁。'
}, {
  name: 'Moai',
  artist: 'Unknown Artist',
  description: '位于复活节岛上，共有 1,000 尊摩艾石像，即现存的纪念性雕像，由早期拉帕努伊人创造，一些人认为它们代表着神化的祖先。',
  url: 'https://react.dev/images/docs/scientists/RCwLEoQm.jpg',
  alt: '三座纪念性石质半身像，头部比例极大，面容凝重。'
}, {
  name: 'Blue Nana',
  artist: 'Niki de Saint Phalle',
  description: 'Nana 是象征女性气质与母性的胜利生物。起初，Saint Phalle 为 Nana 使用布料和现成物品，后来又引入聚酯材料以获得更鲜活的效果。',
  url: 'https://react.dev/images/docs/scientists/Sd1AgUOm.jpg',
  alt: '一座大型马赛克雕塑，描绘一个穿着彩色服装、充满奇思妙想的舞动女性形象，洋溢着喜悦。'
}, {
  name: 'Ultimate Form',
  artist: 'Barbara Hepworth',
  description: '这件抽象青铜雕塑是位于约克郡雕塑公园的《人类大家庭》系列的一部分。Hepworth 并未选择创作世界的写实再现，而是发展出受人和风景启发的抽象形式。',
  url: 'https://react.dev/images/docs/scientists/2heNQDcm.jpg',
  alt: '一座由三个元素堆叠而成的高挑雕塑，让人联想到人体形象。'
}, {
  name: 'Cavaliere',
  artist: 'Lamidi Olonade Fakeye',
  description: 'Fakeye 来自四代木雕世家，他的作品融合了传统与当代约鲁巴主题。',
  url: 'https://react.dev/images/docs/scientists/wIdGuZwm.png',
  alt: '一座精致的木雕，表现一名神情专注的武士骑在装饰着纹样的马上。'
}, {
  name: 'Big Bellies',
  artist: 'Alina Szapocznikow',
  description: 'Szapocznikow 以人体碎片化雕塑著称，将其作为青春与美丽脆弱和短暂的隐喻。这件雕塑展示了两个非常逼真的大肚子上下堆叠在一起，每个大约五英尺（1.5 米）高。',
  url: 'https://react.dev/images/docs/scientists/AlHTAdDm.jpg',
  alt: '这件雕塑让人联想到一连串褶皱，与古典雕塑中的腹部形象截然不同。'
}, {
  name: 'Terracotta Army',
  artist: 'Unknown Artist',
  description: '兵马俑是一组描绘秦始皇军队的陶俑雕塑。军队由 8,000 多名士兵、130 辆战车和 520 匹战马以及 150 匹骑兵马组成。',
  url: 'https://react.dev/images/docs/scientists/HMFmH6m.jpg',
  alt: '12 尊陶俑武士雕塑，每一尊都有独特的面部表情和盔甲。'
}, {
  name: 'Lunar Landscape',
  artist: 'Louise Nevelson',
  description: 'Nevelson 以从纽约市废墟中搜集物件而闻名，她后来将这些物件组装成纪念性构筑物。在这件作品中，她使用了床柱、杂耍棒、座椅碎片等不同部件，并将它们钉接和粘合到盒子中，体现了立体主义对空间与形式的几何抽象影响。',
  url: 'https://react.dev/images/docs/scientists/rN7hY6om.jpg',
  alt: '一座黑色哑光雕塑，各个元素最初难以分辨。'
}, {
  name: 'Aureole',
  artist: 'Ranjani Shettar',
  description: 'Shettar 融合了传统与现代、自然与工业。她的艺术聚焦于人与自然的关系。她的作品被描述为在抽象和具象层面都引人注目、违背重力，以及“对不太可能的材料的精妙综合”。',
  url: 'https://react.dev/images/docs/scientists/okTpbHhm.jpg',
  alt: '一座浅色、像细线一样的雕塑安装在混凝土墙上并垂落到地面，看起来很轻盈。'
}, {
  name: 'Hippos',
  artist: 'Taipei Zoo',
  description: '台北动物园委托创作了一个河马广场，展示了半 submerged 的河马在玩耍。',
  url: 'https://react.dev/images/docs/scientists/6o5Vuyu.jpg',
  alt: '一组青铜河马雕塑从铺装人行道中冒出，仿佛正在游泳。'
}];
```

```css
button { display: block; margin-bottom: 10px; }
.Page > * {
  float: left;
  width: 50%;
  padding: 10px;
}
h2 { margin-top: 10px; margin-bottom: 0; }
h3 {
  margin-top: 5px;
  font-weight: normal;
  font-size: 100%;
}
img { width: 120px; height: 120px; }
```

</Sandpack>

注意 `hasPrev` 和 `hasNext` 既用于返回的 JSX 中，也用于事件处理器内部！这种巧妙的模式之所以可行，是因为事件处理函数会对渲染期间声明的任何变量形成“闭包”。

</Solution>

#### 修复卡住的表单输入 {/*fix-stuck-form-inputs*/}

当你在输入框中键入时，什么也不会出现。就好像输入值被“卡”在空字符串里。第一个 `<input>` 的 `value` 被设置为始终匹配 `firstName` 变量，而第二个 `<input>` 的 `value` 被设置为始终匹配 `lastName` 变量。这是正确的。两个输入框都有 `onChange` 事件处理器，它们试图基于最新的用户输入（`e.target.value`）更新变量。然而，这些变量似乎不会在重新渲染之间“记住”自己的值。请改用状态变量来修复这个问题。

<Sandpack>

```js {expectedErrors: {'react-compiler': [6]}}
export default function Form() {
  let firstName = '';
  let lastName = '';

  function handleFirstNameChange(e) {
    firstName = e.target.value;
  }

  function handleLastNameChange(e) {
    lastName = e.target.value;
  }

  function handleReset() {
    firstName = '';
    lastName = '';
  }

  return (
    <form onSubmit={e => e.preventDefault()}>
      <input
        placeholder="名字"
        value={firstName}
        onChange={handleFirstNameChange}
      />
      <input
        placeholder="姓氏"
        value={lastName}
        onChange={handleLastNameChange}
      />
      <h1>你好，{firstName} {lastName}</h1>
      <button onClick={handleReset}>重置</button>
    </form>
  );
}
```

```css
h1 { margin-top: 10px; }
```

</Sandpack>

<Solution>

首先，从 React 中导入 `useState`。然后用通过调用 `useState` 声明的状态变量替换 `firstName` 和 `lastName`。最后，把每一处 `firstName = ...` 赋值改成 `setFirstName(...)`，`lastName` 也同样处理。别忘了更新 `handleReset`，这样重置按钮才能正常工作。

<Sandpack>

```js
import { useState } from 'react';

export default function Form() {
  const [firstName, setFirstName] = useState('');
  const [lastName, setLastName] = useState('');

  function handleFirstNameChange(e) {
    setFirstName(e.target.value);
  }

  function handleLastNameChange(e) {
    setLastName(e.target.value);
  }

  function handleReset() {
    setFirstName('');
    setLastName('');
  }

  return (
    <form onSubmit={e => e.preventDefault()}>
      <input
        placeholder="名字"
        value={firstName}
        onChange={handleFirstNameChange}
      />
      <input
        placeholder="姓氏"
        value={lastName}
        onChange={handleLastNameChange}
      />
      <h1>你好，{firstName} {lastName}</h1>
      <button onClick={handleReset}>重置</button>
    </form>
  );
}
```

```css
h1 { margin-top: 10px; }
```

</Sandpack>

</Solution>

#### 修复崩溃 {/*fix-a-crash*/}

下面是一个小表单，本来应该让用户留下反馈。当反馈提交后，它应该显示一条感谢消息。然而，它会崩溃，并显示一条错误信息：“Rendered fewer hooks than expected”。你能找出错误并修复它吗？

<Hint>

Hook 的调用位置有任何限制吗？这个组件是否违反了任何规则？检查一下是否有禁用 lint 检查的注释——这类 bug 往往就藏在这里！

</Hint>

<Sandpack>

```js {expectedErrors: {'react-compiler': [9]}}
import { useState } from 'react';

export default function FeedbackForm() {
  const [isSent, setIsSent] = useState(false);
  if (isSent) {
    return <h1>Thank you!</h1>;
  } else {
    // eslint-disable-next-line
    const [message, setMessage] = useState('');
    return (
      <form onSubmit={e => {
        e.preventDefault();
        alert(`Sending: "${message}"`);
        setIsSent(true);
      }}>
        <textarea
          placeholder="Message"
          value={message}
          onChange={e => setMessage(e.target.value)}
        />
        <br />
        <button type="submit">Send</button>
      </form>
    );
  }
}
```

</Sandpack>

<Solution>

Hook 只能在组件函数的顶层调用。这里，`isSent` 的定义符合这个规则，但 `message` 的定义被嵌套在一个条件语句里了。

把它移出条件语句就能修复问题：

<Sandpack>

```js
import { useState } from 'react';

export default function FeedbackForm() {
  const [isSent, setIsSent] = useState(false);
  const [message, setMessage] = useState('');

  if (isSent) {
    return <h1>Thank you!</h1>;
  } else {
    return (
      <form onSubmit={e => {
        e.preventDefault();
        alert(`Sending: "${message}"`);
        setIsSent(true);
      }}>
        <textarea
          placeholder="Message"
          value={message}
          onChange={e => setMessage(e.target.value)}
        />
        <br />
        <button type="submit">Send</button>
      </form>
    );
  }
}
```

</Sandpack>

请记住，Hook 必须无条件地调用，并且始终按相同顺序调用！

你也可以移除不必要的 `else` 分支来减少嵌套。不过，所有 Hook 调用仍然必须发生在第一个 `return` 之前，这一点依然很重要。

<Sandpack>

```js
import { useState } from 'react';

export default function FeedbackForm() {
  const [isSent, setIsSent] = useState(false);
  const [message, setMessage] = useState('');

  if (isSent) {
    return <h1>Thank you!</h1>;
  }

  return (
    <form onSubmit={e => {
      e.preventDefault();
      alert(`Sending: "${message}"`);
      setIsSent(true);
    }}>
      <textarea
        placeholder="Message"
        value={message}
        onChange={e => setMessage(e.target.value)}
      />
      <br />
      <button type="submit">Send</button>
    </form>
  );
}
```

</Sandpack>

试着把第二个 `useState` 调用移到 `if` 条件之后，看看它又是如何再次被破坏的。

如果你的 lint 工具已为 [React 配置](/learn/editor-setup#linting)，当你犯这样的错误时应该会看到 lint 错误。如果你在本地尝试有问题的代码时没有看到错误，那么你需要为你的项目设置 lint 检查。

</Solution>

#### 移除不必要的状态 {/*remove-unnecessary-state*/}

当按钮被点击时，这个示例应该请求用户输入姓名，然后显示一条问候提示。你尝试使用状态来保存名字，但不知为何，第一次显示的是“Hello, !”，之后每次都显示带有上一次输入的“Hello, [name]!”。

要修复这段代码，请移除不必要的状态变量。（我们稍后会讨论[为什么这不起作用](/learn/state-as-a-snapshot)。）

你能解释为什么这个状态变量是不必要的吗？

<Sandpack>

```js
import { useState } from 'react';

export default function FeedbackForm() {
  const [name, setName] = useState('');

  function handleClick() {
    setName(prompt('你叫什么名字？'));
    alert(`Hello, ${name}!`);
  }

  return (
    <button onClick={handleClick}>
      打招呼
    </button>
  );
}
```

</Sandpack>

<Solution>

下面是一个修复后的版本，它在需要的函数中声明一个普通的 `name` 变量：

<Sandpack>

```js
export default function FeedbackForm() {
  function handleClick() {
    const name = prompt('你叫什么名字？');
    alert(`Hello, ${name}!`);
  }

  return (
    <button onClick={handleClick}>
      打招呼
    </button>
  );
}
```

</Sandpack>

状态变量只在需要在组件多次重新渲染之间保留信息时才有必要。在单个事件处理器内部，普通变量就足够了。如果普通变量能很好地工作，就不要引入状态变量。

</Solution>

</Challenges>
