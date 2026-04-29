---
title: 渲染与提交
---

<Intro>

在你的组件显示到屏幕之前，它们必须先由 React 渲染。理解这个过程中的各个步骤，将帮助你思考代码是如何执行的，并解释它的行为。

</Intro>

<YouWillLearn>

* React 中“渲染”是什么意思
* React 何时以及为什么会渲染组件
* 在屏幕上显示组件所涉及的步骤
* 为什么渲染不一定会产生 DOM 更新

</YouWillLearn>

想象你的组件是厨房里的厨师，用各种食材准备美味菜肴。在这个场景中，React 就是服务员，负责接收顾客的请求并把订单送达。这个请求和提供 UI 的过程分为三步：

1. **触发** 渲染（把客人的订单送到厨房）
2. **渲染** 组件（在厨房里准备订单）
3. **提交** 到 DOM（把订单放到桌上）

<IllustrationBlock sequential>
  <Illustration caption="触发" alt="React 作为餐厅里的服务员，从用户那里取走订单并送到组件厨房。" src="/images/docs/illustrations/i_render-and-commit1.png" />
  <Illustration caption="渲染" alt="卡片厨师把一个新的 Card 组件交给 React。" src="/images/docs/illustrations/i_render-and-commit2.png" />
  <Illustration caption="提交" alt="React 将 Card 送到用户桌前。" src="/images/docs/illustrations/i_render-and-commit3.png" />
</IllustrationBlock>

## 第 1 步：触发渲染 {/*step-1-trigger-a-render*/}

组件需要渲染有两个原因：

1. 它的 **初始渲染**。
2. 该组件（或其某个祖先组件）的 **状态已更新**。

### 初始渲染 {/*initial-render*/}

当你的应用启动时，你需要触发初始渲染。框架和沙盒有时会隐藏这段代码，但本质上是通过对目标 DOM 节点调用 [`createRoot`](/reference/react-dom/client/createRoot)，然后使用你的组件调用它的 `render` 方法来完成的：

<Sandpack>

```js src/index.js active
import Image from './Image.js';
import { createRoot } from 'react-dom/client';

const root = createRoot(document.getElementById('root'))
root.render(<Image />);
```

```js src/Image.js
export default function Image() {
  return (
    <img
      src="https://react.dev/images/docs/scientists/ZF6s192.jpg"
      alt="'Floralis Genérica' by Eduardo Catalano: a gigantic metallic flower sculpture with reflective petals"
    />
  );
}
```

</Sandpack>

试着把 `root.render()` 调用注释掉，看看组件是否会消失！

### 状态更新时的重新渲染 {/*re-renders-when-state-updates*/}

一旦组件完成了初始渲染，你就可以通过使用 [`set` 函数。](/reference/react/useState#setstate) 更新它的状态来触发后续渲染。更新组件的状态会自动把一次渲染排入队列。（你可以把它想象成餐厅里的客人在点完第一单后，又根据自己是渴还是饿，继续点茶、甜点以及各种各样的东西。）

<IllustrationBlock sequential>
  <Illustration caption="状态更新..." alt="React 作为餐厅里的服务员，向用户提供一个 Card UI；用户被表示为一个头上是光标的顾客。顾客表示他们想要一个粉色卡片，而不是黑色卡片！" src="/images/docs/illustrations/i_rerender1.png" />
  <Illustration caption="...触发..." alt="React 回到组件厨房，并告诉卡片厨师他们需要一个粉色的 Card。" src="/images/docs/illustrations/i_rerender2.png" />
  <Illustration caption="...渲染！" alt="卡片厨师把粉色的 Card 交给 React。" src="/images/docs/illustrations/i_rerender3.png" />
</IllustrationBlock>

## 第 2 步：React 渲染你的组件 {/*step-2-react-renders-your-components*/}

当你触发渲染后，React 会调用你的组件来弄清楚屏幕上应该显示什么。**“渲染”就是 React 调用你的组件。**

* **在初始渲染时，** React 会调用根组件。
* **在后续渲染时，** React 会调用触发了这次渲染的那个函数组件。

这个过程是递归的：如果更新后的组件返回了另一个组件，React 接下来就会渲染那个组件；如果那个组件也返回了别的东西，React 接着就会渲染那个组件，依此类推。这个过程会一直持续，直到不再有嵌套组件，React 也就能准确知道屏幕上应该显示什么。

在下面的示例中，React 会多次调用 `Gallery()` 和 `Image()`：

<Sandpack>

```js src/Gallery.js active
export default function Gallery() {
  return (
    <section>
      <h1>Inspiring Sculptures</h1>
      <Image />
      <Image />
      <Image />
    </section>
  );
}

function Image() {
  return (
    <img
      src="https://react.dev/images/docs/scientists/ZF6s192.jpg"
      alt="'Floralis Genérica' by Eduardo Catalano: a gigantic metallic flower sculpture with reflective petals"
    />
  );
}
```

```js src/index.js
import Gallery from './Gallery.js';
import { createRoot } from 'react-dom/client';

const root = createRoot(document.getElementById('root'))
root.render(<Gallery />);
```

```css
img { margin: 0 10px 10px 0; }
```

</Sandpack>

* **在初始渲染期间，** React 会为 `<section>`、`<h1>` 和三个 `<img>` 标签 [创建 DOM 节点](https://developer.mozilla.org/docs/Web/API/Document/createElement)。
* **在重新渲染期间，** React 会计算它们的哪些属性（如果有）自上一次渲染以来发生了变化。直到下一步，即提交阶段，它才会使用这些信息。

<Pitfall>

渲染必须始终是一个[纯计算](/learn/keeping-components-pure)：

* **相同输入，相同输出。** 在给定相同输入的情况下，组件应始终返回相同的 JSX。（当有人点一份加番茄的沙拉时，不应该拿到一份加洋葱的沙拉！）
* **各司其职。** 它不应修改渲染之前就已存在的任何对象或变量。（一份订单不应改变别人的订单。）

否则，随着代码库复杂度增加，你可能会遇到令人困惑的 bug 和不可预测的行为。在开发模式下使用“严格模式”时，React 会把每个组件函数调用两次，这有助于暴露由不纯函数导致的错误。

</Pitfall>

<DeepDive>

#### 优化性能 {/*optimizing-performance*/}

默认情况下，渲染更新组件内部所有嵌套组件的行为，在更新组件处于树的较高位置时，对性能并不理想。如果你遇到性能问题，可以在 [性能](https://reactjs.org/docs/optimizing-performance.html) 章节中找到几种可选的解决方式。**不要过早优化！**

</DeepDive>

## 第 3 步：React 将变更提交到 DOM {/*step-3-react-commits-changes-to-the-dom*/}

在渲染（调用）完你的组件后，React 会修改 DOM。

* **对于初始渲染，** React 会使用 DOM API [`appendChild()`](https://developer.mozilla.org/docs/Web/API/Node/appendChild) 将它创建的所有 DOM 节点放到屏幕上。
* **对于重新渲染，** React 会应用必要的最小操作（在渲染时计算得出！），使 DOM 与最新的渲染输出保持一致。

**只有当两次渲染之间存在差异时，React 才会改变 DOM 节点。** 例如，下面这个组件每秒都会从其父组件接收不同的 props 并重新渲染。注意，你可以向 `<input>` 中输入一些文本，更新它的 `value`，但当组件重新渲染时，文本并不会消失：

<Sandpack>

```js src/Clock.js active
export default function Clock({ time }) {
  return (
    <>
      <h1>{time}</h1>
      <input />
    </>
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
  return (
    <Clock time={time.toLocaleTimeString()} />
  );
}
```

</Sandpack>

这之所以有效，是因为在最后这一步中，React 只会用新的 `time` 更新 `<h1>` 的内容。它发现 `<input>` 在 JSX 中出现的位置和上一次相同，所以 React 不会碰 `<input>`——也不会碰它的 `value`！
## 尾声：浏览器绘制 {/*epilogue-browser-paint*/}

在渲染完成且 React 更新了 DOM 之后，浏览器会重新绘制屏幕。虽然这个过程被称为“浏览器渲染”，但为了避免在本文档中产生混淆，我们会把它称为“绘制”。

<Illustration alt="浏览器正在绘制“卡片元素静物”。" src="/images/docs/illustrations/i_browser-paint.png" />

<Recap>

* React 应用中的任何屏幕更新都分三步进行：
  1. 触发
  2. 渲染
  3. 提交
* 你可以使用严格模式来发现组件中的错误
* 如果渲染结果和上一次相同，React 就不会碰 DOM

</Recap>

