---
title: 将 Props 传递给组件
---

<Intro>

React 组件使用 *props* 相互通信。每个父组件都可以通过给子组件传递 props 来传递一些信息。Props 可能会让你联想到 HTML 属性，但你可以通过它们传递任何 JavaScript 值，包括对象、数组和函数。

</Intro>

<YouWillLearn>

* 如何向组件传递 props
* 如何从组件中读取 props
* 如何为 props 指定默认值
* 如何向组件传递一些 JSX
* props 如何随时间变化

</YouWillLearn>

## 常见的 props {/*familiar-props*/}

Props 是你传递给 JSX 标签的信息。例如，`className`、`src`、`alt`、`width` 和 `height` 都是你可以传递给 `<img>` 的一些 props：

<Sandpack>

```js
function Avatar() {
  return (
    <img
      className="avatar"
      src="https://react.dev/images/docs/scientists/1bX5QH6.jpg"
      alt="Lin Lanying"
      width={100}
      height={100}
    />
  );
}

export default function Profile() {
  return (
    <Avatar />
  );
}
```

```css
body { min-height: 120px; }
.avatar { margin: 20px; border-radius: 50%; }
```

</Sandpack>

你可以传递给 `<img>` 标签的 props 是预先定义好的（ReactDOM 遵循 [HTML 标准](https://www.w3.org/TR/html52/semantics-embedded-content.html#the-img-element)）。但你可以向 *你自己的* 组件传递任何 props，比如 `<Avatar>`，来定制它们。下面就是方法！

## 向组件传递 props {/*passing-props-to-a-component*/}

在这段代码中，`Profile` 组件并没有向它的子组件 `Avatar` 传递任何 props：

```js
export default function Profile() {
  return (
    <Avatar />
  );
}
```

你可以分两步给 `Avatar` 传递一些 props。

### 第 1 步：向子组件传递 props {/*step-1-pass-props-to-the-child-component*/}

首先，向 `Avatar` 传递一些 props。例如，让我们传递两个 props：`person`（一个对象）和 `size`（一个数字）：

```js
export default function Profile() {
  return (
    <Avatar
      person={{ name: 'Lin Lanying', imageId: '1bX5QH6' }}
      size={100}
    />
  );
}
```

<Note>

如果 `person=` 后面的双层花括号让你困惑，可以回想一下，[它们本质上只是一个对象](/learn/javascript-in-jsx-with-curly-braces#using-double-curlies-css-and-other-objects-in-jsx) 放在 JSX 花括号中。

</Note>

现在你就可以在 `Avatar` 组件内部读取这些 props 了。

### 第 2 步：在子组件内部读取 props {/*step-2-read-props-inside-the-child-component*/}

你可以通过在 `function Avatar` 后面的 `({` 和 `})` 内部，直接用逗号分隔列出它们的名字 `person, size` 来读取这些 props。这样你就可以像使用变量一样在 `Avatar` 代码中使用它们。

```js
function Avatar({ person, size }) {
  // person 和 size 在这里可用
}
```

给 `Avatar` 添加一些使用 `person` 和 `size` props 来进行渲染的逻辑，就完成了。

现在你可以用不同的 props 将 `Avatar` 配置成多种不同的渲染方式。试着调整这些值！

<Sandpack>

```js src/App.js
import { getImageUrl } from './utils.js';

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

export default function Profile() {
  return (
    <div>
      <Avatar
        size={100}
        person={{
          name: 'Katsuko Saruhashi',
          imageId: 'YfeOqp2'
        }}
      />
      <Avatar
        size={80}
        person={{
          name: 'Aklilu Lemma',
          imageId: 'OKS67lh'
        }}
      />
      <Avatar
        size={50}
        person={{
          name: 'Lin Lanying',
          imageId: '1bX5QH6'
        }}
      />
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
body { min-height: 120px; }
.avatar { margin: 10px; border-radius: 50%; }
```

</Sandpack>

Props 让你可以独立思考父组件和子组件。例如，你可以在 `Profile` 中更改 `person` 或 `size` props，而不必考虑 `Avatar` 如何使用它们。类似地，你也可以更改 `Avatar` 使用这些 props 的方式，而无需查看 `Profile`。

你可以把 props 想成可以调整的“旋钮”。它们所起的作用和函数参数一样——实际上，props _就是_ 传递给组件的唯一参数！React 组件函数只接受一个参数，也就是一个 `props` 对象：

```js
function Avatar(props) {
  let person = props.person;
  let size = props.size;
  // ...
}
```

通常你不需要整个 `props` 对象本身，所以你会把它解构成单独的 props。

<Pitfall>

**不要遗漏** 在 `(` 和 `)` 里面的 `{` 和 `}` 这对花括号，在声明 props 时：

```js
function Avatar({ person, size }) {
  // ...
}
```

这种语法叫做 ["解构"](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment#Unpacking_fields_from_objects_passed_as_a_function_parameter)，它等同于从函数参数中读取属性：

```js
function Avatar(props) {
  let person = props.person;
  let size = props.size;
  // ...
}
```

</Pitfall>

## 为 prop 指定默认值 {/*specifying-a-default-value-for-a-prop*/}

如果你想在没有指定值时给某个 prop 一个回退的默认值，可以通过解构在参数后面直接写上 `=` 和默认值来实现：

```js
function Avatar({ person, size = 100 }) {
  // ...
}
```

现在，如果渲染 `<Avatar person={...} />` 时没有提供 `size` prop，那么 `size` 将被设为 `100`。

默认值只会在 `size` prop 缺失或你传递 `size={undefined}` 时使用。但如果你传递 `size={null}` 或 `size={0}`，则**不会**使用默认值。

## 使用 JSX 扩展语法转发 props {/*forwarding-props-with-the-jsx-spread-syntax*/}

有时候，传递 props 会变得非常重复：

```js
function Profile({ person, size, isSepia, thickBorder }) {
  return (
    <div className="card">
      <Avatar
        person={person}
        size={size}
        isSepia={isSepia}
        thickBorder={thickBorder}
      />
    </div>
  );
}
```

重复的代码并没有错——它可能更易读。但有时你可能更看重简洁性。有些组件会把它们所有的 props 都转发给子组件，就像这个 `Profile` 对 `Avatar` 所做的那样。因为它们并没有直接使用任何 props，所以使用更简洁的“扩展”语法是合理的：

```js
function Profile(props) {
  return (
    <div className="card">
      <Avatar {...props} />
    </div>
  );
}
```

这会把 `Profile` 的所有 props 转发给 `Avatar`，而无需逐一列出它们的名字。

**请谨慎使用扩展语法。** 如果你在几乎每个组件里都在用它，那就说明有问题了。通常这表示你应该拆分组件，并将 children 作为 JSX 传递。下面就会讲到！

## 将 JSX 作为 children 传递 {/*passing-jsx-as-children*/}

在内置的浏览器标签中嵌套内容是很常见的：

```js
<div>
  <img />
</div>
```

有时你也会希望以同样的方式嵌套自己的组件：

```js
<Card>
  <Avatar />
</Card>
```

当你把内容嵌套在 JSX 标签内部时，父组件会在一个名为 `children` 的 prop 中接收这些内容。例如，下面的 `Card` 组件会接收一个值为 `<Avatar />` 的 `children` prop，并将其渲染到一个包装 `div` 中：

<Sandpack>

```js src/App.js
import Avatar from './Avatar.js';

function Card({ children }) {
  return (
    <div className="card">
      {children}
    </div>
  );
}

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
```

```js src/Avatar.js
import { getImageUrl } from './utils.js';

export default function Avatar({ person, size }) {
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

试着把 `<Card>` 里的 `<Avatar>` 替换成一些文本，看看 `Card` 组件如何包装任何嵌套内容。它不需要“知道”内部渲染的是什么。你会在很多地方看到这种灵活的模式。

你可以把带有 `children` prop 的组件想象成有一个“洞”，这个“洞”可以被父组件用任意 JSX “填充”。你会经常将 `children` prop 用于视觉上的包装组件：面板、网格等等。

<Illustration src="/images/docs/illustrations/i_children-prop.png" alt='一个类似拼图的 Card 方块，带有一个可插入“children”碎片的槽位，例如文本和 Avatar' />

## props 如何随时间变化 {/*how-props-change-over-time*/}

下面的 `Clock` 组件从父组件接收两个 props：`color` 和 `time`。（父组件的代码被省略了，因为它使用了 [state](/learn/state-a-components-memory)，我们暂时还不会深入讨论。）

试着在下面的选择框中更改颜色：

<Sandpack>

```js src/Clock.js active
export default function Clock({ color, time }) {
  return (
    <h1 style={{ color: color }}>
      {time}
    </h1>
  );
}
```

```js src/App.js hidden
import { useState, useEffect } from 'react';
import Clock from './Clock.js';

function useTime() {
  const [time, setTime] = useState(() => new Date());
  useEffect(() => {
    const id = setInterval(() => {
      setTime(new Date());
    }, 1000);
    return () => clearInterval(id);
  }, []);
  return time;
}

export default function App() {
  const time = useTime();
  const [color, setColor] = useState('lightcoral');
  return (
    <div>
      <p>
        Pick a color:{' '}
        <select value={color} onChange={e => setColor(e.target.value)}>
          <option value="lightcoral">lightcoral</option>
          <option value="midnightblue">midnightblue</option>
          <option value="rebeccapurple">rebeccapurple</option>
        </select>
      </p>
      <Clock color={color} time={time.toLocaleTimeString()} />
    </div>
  );
}
```

</Sandpack>

这个例子说明，**组件可能会随着时间接收到不同的 props。** Props 并不总是静态的！这里，`time` prop 每秒都会变化，而 `color` prop 会在你选择另一个颜色时变化。Props 反映的是组件在任意时刻的数据，而不只是开始时的数据。

不过，props 是 [不可变的](https://en.wikipedia.org/wiki/Immutable_object)——这是计算机科学中的一个术语，意思是“不可改变的”。当组件需要改变它的 props 时（例如，为了响应用户交互或新数据），它必须“请求”父组件传递给它 _不同的 props_——一个新的对象！然后旧的 props 会被丢弃，最终由 JavaScript 引擎回收它们占用的内存。

**不要试图“改变 props”。** 当你需要响应用户输入（比如更改选中的颜色）时，你需要“设置 state”，你可以在 [State：组件的记忆。](/learn/state-a-components-memory) 中了解它。

<Recap>

* 要传递 props，只需像使用 HTML 属性那样把它们添加到 JSX 中。
* 要读取 props，使用 `function Avatar({ person, size })` 这种解构语法。
* 你可以指定默认值，比如 `size = 100`，它会用于缺失和 `undefined` 的 props。
* 你可以使用 `<Avatar {...props} />` 这种 JSX 扩展语法转发所有 props，但不要过度使用！
* 像 `<Card><Avatar /></Card>` 这样的嵌套 JSX 会作为 `Card` 组件的 `children` prop 出现。
* Props 是时间上的只读快照：每次渲染都会接收到一版新的 props。
* 你不能改变 props。当你需要交互性时，你需要设置 state。

</Recap>



<Challenges>

#### 提取组件 {/*extract-a-component*/}

这个 `Gallery` 组件中包含两个人物简介的非常相似的标记。把其中的 `Profile` 组件提取出来，以减少重复。你需要决定向它传递哪些 props。

<Sandpack>

```js src/App.js
import { getImageUrl } from './utils.js';

export default function Gallery() {
  return (
    <div>
      <h1>Notable Scientists</h1>
      <section className="profile">
        <h2>Maria Skłodowska-Curie</h2>
        <img
          className="avatar"
          src={getImageUrl('szV5sdG')}
          alt="Maria Skłodowska-Curie"
          width={70}
          height={70}
        />
        <ul>
          <li>
            <b>Profession: </b>
            physicist and chemist
          </li>
          <li>
            <b>Awards: 4 </b>
            (Nobel Prize in Physics, Nobel Prize in Chemistry, Davy Medal, Matteucci Medal)
          </li>
          <li>
            <b>Discovered: </b>
            polonium (chemical element)
          </li>
        </ul>
      </section>
      <section className="profile">
        <h2>Katsuko Saruhashi</h2>
        <img
          className="avatar"
          src={getImageUrl('YfeOqp2')}
          alt="Katsuko Saruhashi"
          width={70}
          height={70}
        />
        <ul>
          <li>
            <b>Profession: </b>
            geochemist
          </li>
          <li>
            <b>Awards: 2 </b>
            (Miyake Prize for geochemistry, Tanaka Prize)
          </li>
          <li>
            <b>Discovered: </b>
            a method for measuring carbon dioxide in seawater
          </li>
        </ul>
      </section>
    </div>
  );
}
```

```js src/utils.js
export function getImageUrl(imageId, size = 's') {
  return (
    'https://react.dev/images/docs/scientists/' +
    imageId +
    size +
    '.jpg'
  );
}
```

```css
.avatar { margin: 5px; border-radius: 50%; min-height: 70px; }
.profile {
  border: 1px solid #aaa;
  border-radius: 6px;
  margin-top: 20px;
  padding: 10px;
}
h1, h2 { margin: 5px; }
h1 { margin-bottom: 10px; }
ul { padding: 0px 10px 0px 20px; }
li { margin: 5px; }
```

</Sandpack>

<Hint>

先提取其中一位科学家的标记。然后找出第二个示例中与之不匹配的部分，并将它们变成可通过 props 配置的内容。

</Hint>

<Solution>

在这个解决方案中，`Profile` 组件接受多个 props：`imageId`（字符串）、`name`（字符串）、`profession`（字符串）、`awards`（字符串数组）、`discovery`（字符串）以及 `imageSize`（数字）。

注意，`imageSize` prop 有一个默认值，所以我们没有向组件传递它。

<Sandpack>

```js src/App.js
import { getImageUrl } from './utils.js';

function Profile({
  imageId,
  name,
  profession,
  awards,
  discovery,
  imageSize = 70
}) {
  return (
    <section className="profile">
      <h2>{name}</h2>
      <img
        className="avatar"
        src={getImageUrl(imageId)}
        alt={name}
        width={imageSize}
        height={imageSize}
      />
      <ul>
        <li><b>Profession:</b> {profession}</li>
        <li>
          <b>Awards: {awards.length} </b>
          ({awards.join(', ')})
        </li>
        <li>
          <b>Discovered: </b>
          {discovery}
        </li>
      </ul>
    </section>
  );
}

export default function Gallery() {
  return (
    <div>
      <h1>Notable Scientists</h1>
      <Profile
        imageId="szV5sdG"
        name="Maria Skłodowska-Curie"
        profession="physicist and chemist"
        discovery="polonium (chemical element)"
        awards={[
          'Nobel Prize in Physics',
          'Nobel Prize in Chemistry',
          'Davy Medal',
          'Matteucci Medal'
        ]}
      />
      <Profile
        imageId='YfeOqp2'
        name='Katsuko Saruhashi'
        profession='geochemist'
        discovery="a method for measuring carbon dioxide in seawater"
        awards={[
          'Miyake Prize for geochemistry',
          'Tanaka Prize'
        ]}
      />
    </div>
  );
}
```

```js src/utils.js
export function getImageUrl(imageId, size = 's') {
  return (
    'https://react.dev/images/docs/scientists/' +
    imageId +
    size +
    '.jpg'
  );
}
```

```css
.avatar { margin: 5px; border-radius: 50%; min-height: 70px; }
.profile {
  border: 1px solid #aaa;
  border-radius: 6px;
  margin-top: 20px;
  padding: 10px;
}
h1, h2 { margin: 5px; }
h1 { margin-bottom: 10px; }
ul { padding: 0px 10px 0px 20px; }
li { margin: 5px; }
```

</Sandpack>

注意，如果 `awards` 是一个数组，你就不需要单独的 `awardCount` prop。然后你可以用 `awards.length` 来统计获奖数量。记住 props 可以接受任何值，其中也包括数组！

另一种解法，更接近本页前面的示例，是把一个人的所有信息都放在一个对象里，然后把这个对象作为一个 prop 传递：

<Sandpack>

```js src/App.js
import { getImageUrl } from './utils.js';

function Profile({ person, imageSize = 70 }) {
  const imageSrc = getImageUrl(person)

  return (
    <section className="profile">
      <h2>{person.name}</h2>
      <img
        className="avatar"
        src={imageSrc}
        alt={person.name}
        width={imageSize}
        height={imageSize}
      />
      <ul>
        <li>
          <b>Profession:</b> {person.profession}
        </li>
        <li>
          <b>Awards: {person.awards.length} </b>
          ({person.awards.join(', ')})
        </li>
        <li>
          <b>Discovered: </b>
          {person.discovery}
        </li>
      </ul>
    </section>
  )
}

export default function Gallery() {
  return (
    <div>
      <h1>Notable Scientists</h1>
      <Profile person={{
        imageId: 'szV5sdG',
        name: 'Maria Skłodowska-Curie',
        profession: 'physicist and chemist',
        discovery: 'polonium (chemical element)',
        awards: [
          'Nobel Prize in Physics',
          'Nobel Prize in Chemistry',
          'Davy Medal',
          'Matteucci Medal'
        ],
      }} />
      <Profile person={{
        imageId: 'YfeOqp2',
        name: 'Katsuko Saruhashi',
        profession: 'geochemist',
        discovery: 'a method for measuring carbon dioxide in seawater',
        awards: [
          'Miyake Prize for geochemistry',
          'Tanaka Prize'
        ],
      }} />
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
.avatar { margin: 5px; border-radius: 50%; min-height: 70px; }
.profile {
  border: 1px solid #aaa;
  border-radius: 6px;
  margin-top: 20px;
  padding: 10px;
}
h1, h2 { margin: 5px; }
h1 { margin-bottom: 10px; }
ul { padding: 0px 10px 0px 20px; }
li { margin: 5px; }
```

</Sandpack>

虽然语法看起来略有不同，因为你描述的是 JavaScript 对象的属性，而不是一组 JSX 属性，但这些示例在大多数情况下是等价的，你可以任选一种方式。

</Solution>

#### 根据 prop 调整图片大小 {/*adjust-the-image-size-based-on-a-prop*/}

在这个示例中，`Avatar` 接收一个数字类型的 `size` prop，它决定 `<img>` 的宽度和高度。本示例中的 `size` prop 被设为 `40`。不过，如果你在新标签页中打开这张图片，你会发现图片本身更大（`160` 像素）。真实的图片尺寸由你请求的缩略图大小决定。

修改 `Avatar` 组件，使其根据 `size` prop 请求最接近的图片大小。具体来说，如果 `size` 小于 `90`，就传递 `'s'`（“small”）而不是 `'b'`（“big”）给 `getImageUrl` 函数。通过用不同的 `size` prop 值渲染头像，并在新标签页中打开图片，来验证你的修改是否有效。

<Sandpack>

```js src/App.js
import { getImageUrl } from './utils.js';

function Avatar({ person, size }) {
  return (
    <img
      className="avatar"
      src={getImageUrl(person, 'b')}
      alt={person.name}
      width={size}
      height={size}
    />
  );
}

export default function Profile() {
  return (
    <Avatar
      size={40}
      person={{
        name: 'Gregorio Y. Zara',
        imageId: '7vQD0fP'
      }}
    />
  );
}
```

```js src/utils.js
export function getImageUrl(person, size) {
  return (
    'https://react.dev/images/docs/scientists/' +
    person.imageId +
    size +
    '.jpg'
  );
}
```

```css
.avatar { margin: 20px; border-radius: 50%; }
```

</Sandpack>

<Solution>

你可以这样做：

<Sandpack>

```js src/App.js
import { getImageUrl } from './utils.js';

function Avatar({ person, size }) {
  let thumbnailSize = 's';
  if (size > 90) {
    thumbnailSize = 'b';
  }
  return (
    <img
      className="avatar"
      src={getImageUrl(person, thumbnailSize)}
      alt={person.name}
      width={size}
      height={size}
    />
  );
}

export default function Profile() {
  return (
    <>
      <Avatar
        size={40}
        person={{
          name: 'Gregorio Y. Zara',
          imageId: '7vQD0fP'
        }}
      />
      <Avatar
        size={120}
        person={{
          name: 'Gregorio Y. Zara',
          imageId: '7vQD0fP'
        }}
      />
    </>
  );
}
```

```js src/utils.js
export function getImageUrl(person, size) {
  return (
    'https://react.dev/images/docs/scientists/' +
    person.imageId +
    size +
    '.jpg'
  );
}
```

```css
.avatar { margin: 20px; border-radius: 50%; }
```

</Sandpack>

你也可以通过考虑 [`window.devicePixelRatio`](https://developer.mozilla.org/en-US/docs/Web/API/Window/devicePixelRatio) 来在高 DPI 屏幕上显示更清晰的图片：

<Sandpack>

```js src/App.js
import { getImageUrl } from './utils.js';

const ratio = window.devicePixelRatio;

function Avatar({ person, size }) {
  let thumbnailSize = 's';
  if (size * ratio > 90) {
    thumbnailSize = 'b';
  }
  return (
    <img
      className="avatar"
      src={getImageUrl(person, thumbnailSize)}
      alt={person.name}
      width={size}
      height={size}
    />
  );
}

export default function Profile() {
  return (
    <>
      <Avatar
        size={40}
        person={{
          name: 'Gregorio Y. Zara',
          imageId: '7vQD0fP'
        }}
      />
      <Avatar
        size={70}
        person={{
          name: 'Gregorio Y. Zara',
          imageId: '7vQD0fP'
        }}
      />
      <Avatar
        size={120}
        person={{
          name: 'Gregorio Y. Zara',
          imageId: '7vQD0fP'
        }}
      />
    </>
  );
}
```

```js src/utils.js
export function getImageUrl(person, size) {
  return (
    'https://react.dev/images/docs/scientists/' +
    person.imageId +
    size +
    '.jpg'
  );
}
```

```css
.avatar { margin: 20px; border-radius: 50%; }
```

</Sandpack>

Props 让你可以把这样的逻辑封装在 `Avatar` 组件内部（并在以后需要时更改它），这样每个人都可以使用 `<Avatar>` 组件，而不必考虑图片是如何请求和调整大小的。

</Solution>

#### 在 `children` prop 中传递 JSX {/*passing-jsx-in-a-children-prop*/}

从下面的标记中提取一个 `Card` 组件，并使用 `children` prop 向它传递不同的 JSX：

<Sandpack>

```js
export default function Profile() {
  return (
    <div>
      <div className="card">
        <div className="card-content">
          <h1>Photo</h1>
          <img
            className="avatar"
            src="https://react.dev/images/docs/scientists/OKS67lhm.jpg"
            alt="Aklilu Lemma"
            width={70}
            height={70}
          />
        </div>
      </div>
      <div className="card">
        <div className="card-content">
          <h1>About</h1>
          <p>Aklilu Lemma was a distinguished Ethiopian scientist who discovered a natural treatment to schistosomiasis.</p>
        </div>
      </div>
    </div>
  );
}
```

```css
.card {
  width: fit-content;
  margin: 20px;
  padding: 20px;
  border: 1px solid #aaa;
  border-radius: 20px;
  background: #fff;
}
.card-content {
  text-align: center;
}
.avatar {
  margin: 10px;
  border-radius: 50%;
}
h1 {
  margin: 5px;
  padding: 0;
  font-size: 24px;
}
```

</Sandpack>

<Hint>

你放在组件标签内部的任何 JSX 都会作为该组件的 `children` prop 传递给它。

</Hint>

<Solution>

你可以这样在两个地方使用 `Card` 组件：

<Sandpack>

```js
function Card({ children }) {
  return (
    <div className="card">
      <div className="card-content">
        {children}
      </div>
    </div>
  );
}

export default function Profile() {
  return (
    <div>
      <Card>
        <h1>Photo</h1>
        <img
          className="avatar"
          src="https://react.dev/images/docs/scientists/OKS67lhm.jpg"
          alt="Aklilu Lemma"
          width={100}
          height={100}
        />
      </Card>
      <Card>
        <h1>About</h1>
        <p>Aklilu Lemma was a distinguished Ethiopian scientist who discovered a natural treatment to schistosomiasis.</p>
      </Card>
    </div>
  );
}
```

```css
.card {
  width: fit-content;
  margin: 20px;
  padding: 20px;
  border: 1px solid #aaa;
  border-radius: 20px;
  background: #fff;
}
.card-content {
  text-align: center;
}
.avatar {
  margin: 10px;
  border-radius: 50%;
}
h1 {
  margin: 5px;
  padding: 0;
  font-size: 24px;
}
```

</Sandpack>

如果你希望每个 `Card` 都总是带有标题，也可以把 `title` 做成一个单独的 prop：

<Sandpack>

```js
function Card({ children, title }) {
  return (
    <div className="card">
      <div className="card-content">
        <h1>{title}</h1>
        {children}
      </div>
    </div>
  );
}

export default function Profile() {
  return (
    <div>
      <Card title="Photo">
        <img
          className="avatar"
          src="https://react.dev/images/docs/scientists/OKS67lhm.jpg"
          alt="Aklilu Lemma"
          width={100}
          height={100}
        />
      </Card>
      <Card title="About">
        <p>Aklilu Lemma was a distinguished Ethiopian scientist who discovered a natural treatment to schistosomiasis.</p>
      </Card>
    </div>
  );
}
```

```css
.card {
  width: fit-content;
  margin: 20px;
  padding: 20px;
  border: 1px solid #aaa;
  border-radius: 20px;
  background: #fff;
}
.card-content {
  text-align: center;
}
.avatar {
  margin: 10px;
  border-radius: 50%;
}
h1 {
  margin: 5px;
  padding: 0;
  font-size: 24px;
}
```

</Sandpack>

</Solution>

</Challenges>
