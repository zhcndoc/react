---
title: useCallback
---

<Intro>

`useCallback` 是一个 React Hook，它允许你在多次重新渲染之间缓存函数定义。

```js
const cachedFn = useCallback(fn, dependencies)
```

</Intro>

<Note>

[React Compiler](/learn/react-compiler) 会自动对值和函数进行记忆化，从而减少手动调用 `useCallback` 的需要。你可以使用编译器自动处理记忆化。

</Note>

<InlineToc />

---

## 参考 {/*reference*/}

### `useCallback(fn, dependencies)` {/*usecallback*/}

在组件顶层调用 `useCallback`，以便在多次重新渲染之间缓存函数定义：

```js {4,9}
import { useCallback } from 'react';

export default function ProductPage({ productId, referrer, theme }) {
  const handleSubmit = useCallback((orderDetails) => {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails,
    });
  }, [productId, referrer]);
```

[查看更多示例。](#usage)

#### 参数 {/*parameters*/}

* `fn`：你希望缓存的函数值。它可以接受任意参数并返回任意值。React 会在初始渲染时把你的函数返回给你（不是调用！）。在后续渲染中，如果 `dependencies` 自上次渲染以来没有变化，React 会再次给你同一个函数。否则，它会给你当前渲染中传入的函数，并将其存储起来，以便之后可以复用。React 不会调用你的函数。之所以把函数返回给你，是为了让你决定何时以及是否调用它。

* `dependencies`：`fn` 代码中引用的所有响应式值列表。响应式值包括 props、state，以及直接在组件函数体内声明的所有变量和函数。如果你的 linter 已[为 React 配置](/learn/editor-setup#linting)，它会验证每个响应式值都被正确指定为依赖项。依赖项列表必须具有固定数量的项目，并以内联形式编写，例如 `[dep1, dep2, dep3]`。React 会使用 [`Object.is`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is) 比较算法，将每个依赖项与其之前的值进行比较。

#### 返回值 {/*returns*/}

在初始渲染时，`useCallback` 返回你传入的 `fn` 函数。

在后续渲染中，它要么返回上一次渲染中已存储的 `fn` 函数（如果依赖项没有变化），要么返回你在本次渲染中传入的 `fn` 函数。

#### 注意事项 {/*caveats*/}

* `useCallback` 是一个 Hook，因此你只能在**组件顶层**或你自己的 Hooks 中调用它。你不能在循环或条件语句中调用它。如果你需要那样做，请提取一个新组件并把 state 移到其中。
* React **不会丢弃已缓存的函数，除非有特定原因这样做。** 例如，在开发环境中，当你编辑组件文件时，React 会丢弃缓存。在开发和生产环境中，如果你的组件在初始挂载时挂起，React 会丢弃缓存。将来，React 可能会增加更多会利用丢弃缓存的特性——例如，如果 React 未来为虚拟化列表提供内置支持，那么对于滚出虚拟化表格视口的项目，丢弃缓存将是合理的。如果你将 `useCallback` 作为性能优化手段，这应当符合你的预期。否则，使用 [state 变量](/reference/react/useState#im-trying-to-set-state-to-a-function-but-it-gets-called-instead) 或 [ref](/reference/react/useRef#avoiding-recreating-the-ref-contents) 可能更合适。

---

## 用法 {/*usage*/}

### 跳过组件重新渲染 {/*skipping-re-rendering-of-components*/}

当你优化渲染性能时，有时需要缓存传递给子组件的函数。我们先看一下如何实现的语法，然后再看看它在哪些情况下有用。

要在组件多次重新渲染之间缓存函数，请将其定义包裹到 `useCallback` Hook 中：

```js [[3, 4, "handleSubmit"], [2, 9, "[productId, referrer]"]]
import { useCallback } from 'react';

function ProductPage({ productId, referrer, theme }) {
  const handleSubmit = useCallback((orderDetails) => {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails,
    });
  }, [productId, referrer]);
  // ...
```

你需要向 `useCallback` 传入两样东西：

1. 你想要在多次重新渲染之间缓存的函数定义。
2. 一个 <CodeStep step={2}>依赖项列表</CodeStep>，其中包含你组件中在函数内使用到的每个值。

在初始渲染时，你从 `useCallback` 得到的 <CodeStep step={3}>返回函数</CodeStep> 会是你传入的那个函数。

在后续渲染中，React 会将 <CodeStep step={2}>依赖项</CodeStep> 与你在上一次渲染中传入的依赖项进行比较。如果没有任何依赖项发生变化（与 [`Object.is`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is) 比较），`useCallback` 会返回与之前相同的函数。否则，`useCallback` 会返回你在*本次*渲染中传入的函数。

换句话说，`useCallback` 会在多次重新渲染之间缓存一个函数，直到它的依赖项发生变化。

**让我们通过一个例子来看看它何时有用。**

假设你把 `handleSubmit` 函数从 `ProductPage` 传给了 `ShippingForm` 组件：

```js {5}
function ProductPage({ productId, referrer, theme }) {
  // ...
  return (
    <div className={theme}>
      <ShippingForm onSubmit={handleSubmit} />
    </div>
  );
```

你注意到切换 `theme` prop 时应用会卡顿一下，但如果你从 JSX 中移除 `<ShippingForm />`，它就会感觉很快。这说明值得尝试优化 `ShippingForm` 组件。

**默认情况下，当某个组件重新渲染时，React 会递归地重新渲染它的所有子组件。** 这就是为什么当 `ProductPage` 因为不同的 `theme` 而重新渲染时，`ShippingForm` 组件也会重新渲染。对于不需要太多计算就能重新渲染的组件来说，这没问题。但如果你确认某次重新渲染很慢，就可以通过将 `ShippingForm` 包裹在 [`memo`](/reference/react/memo) 中，让它在 props 与上次渲染相同时跳过重新渲染：

```js {3,5}
import { memo } from 'react';

const ShippingForm = memo(function ShippingForm({ onSubmit }) {
  // ...
});
```

**有了这个改动，如果 `ShippingForm` 的所有 props 与上一次渲染时*相同*，它就会跳过重新渲染。** 这时缓存函数就变得重要了！假设你没有使用 `useCallback` 来定义 `handleSubmit`：

```js {2,3,8,12-13}
function ProductPage({ productId, referrer, theme }) {
  // 每次 theme 变化时，这都会是一个不同的函数...
  function handleSubmit(orderDetails) {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails,
    });
  }

  return (
    <div className={theme}>
      {/* ...所以 ShippingForm 的 props 将永远不会相同，它会在每次都重新渲染 */}
      <ShippingForm onSubmit={handleSubmit} />
    </div>
  );
}
```

**在 JavaScript 中，`function () {}` 或 `() => {}` 总是会创建一个_不同的_函数，** 这与 `{}` 对象字面量总是会创建一个新对象类似。通常这不会有问题，但这意味着 `ShippingForm` 的 props 永远不会相同，而你的 [`memo`](/reference/react/memo) 优化也就无法生效。这就是 `useCallback` 派上用场的地方：

```js {2,3,8,12-13}
function ProductPage({ productId, referrer, theme }) {
  // 告诉 React 在多次重新渲染之间缓存你的函数...
  const handleSubmit = useCallback((orderDetails) => {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails,
    });
  }, [productId, referrer]); // ...只要这些依赖项没有变化...

  return (
    <div className={theme}>
      {/* ...ShippingForm 将收到相同的 props，并且可以跳过重新渲染 */}
      <ShippingForm onSubmit={handleSubmit} />
    </div>
  );
}
```

**通过将 `handleSubmit` 包裹在 `useCallback` 中，你可以确保它在多次重新渲染之间保持*相同*的函数**（直到依赖项变化）。你并不*必须*把函数包裹在 `useCallback` 中，除非你有某些特定原因。在这个例子中，原因是你把它传给了一个包裹在 [`memo`](/reference/react/memo) 中的组件，这样它就可以跳过重新渲染。你可能还会因为其他原因需要 `useCallback`，本页后面会继续介绍。

<Note>

**你应该只把 `useCallback` 作为一种性能优化手段来依赖。** 如果你的代码在没有它的情况下不能正常工作，请先找出根本问题并修复它。然后你可以再把 `useCallback` 加回来。

</Note>

<DeepDive>

#### useCallback 和 useMemo 有什么关系？ {/*how-is-usecallback-related-to-usememo*/}

你经常会看到 [`useMemo`](/reference/react/useMemo) 和 `useCallback` 一起使用。当你试图优化子组件时，它们都很有用。它们可以让你对传下去的内容进行[记忆化](https://en.wikipedia.org/wiki/Memoization)（换句话说，就是缓存）：

```js {6-8,10-15,19}
import { useMemo, useCallback } from 'react';

function ProductPage({ productId, referrer }) {
  const product = useData('/product/' + productId);

  const requirements = useMemo(() => { // 调用你的函数并缓存其结果
    return computeRequirements(product);
  }, [product]);

  const handleSubmit = useCallback((orderDetails) => { // 缓存函数本身
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails,
    });
  }, [productId, referrer]);

  return (
    <div className={theme}>
      <ShippingForm requirements={requirements} onSubmit={handleSubmit} />
    </div>
  );
}
```

区别在于它们允许你缓存的*内容*不同：

* **[`useMemo`](/reference/react/useMemo) 缓存的是调用你的函数的*结果*。** 在这个例子中，它缓存的是调用 `computeRequirements(product)` 的结果，这样除非 `product` 发生变化，否则结果就不会改变。这样你就可以把 `requirements` 对象传下去，而不会不必要地重新渲染 `ShippingForm`。在必要时，React 会在渲染过程中调用你传入的函数来计算结果。
* **`useCallback` 缓存的是*函数本身*。** 与 `useMemo` 不同，它不会调用你提供的函数。相反，它会缓存你提供的函数，这样 `handleSubmit` *本身*就不会变化，除非 `productId` 或 `referrer` 发生变化。这样你就可以把 `handleSubmit` 函数传下去，而不会不必要地重新渲染 `ShippingForm`。你的代码要等到用户提交表单时才会运行。

如果你已经熟悉 [`useMemo`](/reference/react/useMemo)，可以把 `useCallback` 理解为：

```js {expectedErrors: {'react-compiler': [3]}}
// 简化实现（在 React 内部）
function useCallback(fn, dependencies) {
  return useMemo(() => fn, dependencies);
}
```

[阅读更多关于 `useMemo` 和 `useCallback` 之间区别的内容。](/reference/react/useMemo#memoizing-a-function)

</DeepDive>

<DeepDive>

#### 要在所有地方都加上 useCallback 吗？ {/*should-you-add-usecallback-everywhere*/}

如果你的应用像这个网站一样，大多数交互都比较粗粒度（比如替换整页或整个区域），那么记忆化通常是不必要的。另一方面，如果你的应用更像绘图编辑器，而且大多数交互都很细粒度（比如移动形状），那么你可能会发现记忆化非常有帮助。

使用 `useCallback` 缓存函数只有在少数情况下才有价值：

- 你把它作为 prop 传给一个包裹在 [`memo`](/reference/react/memo) 中的组件。你希望在值未变化时跳过重新渲染。记忆化可以让你的组件只在依赖项变化时重新渲染。
- 你传递的函数之后会被用作某个 Hook 的依赖项。例如，另一个用 `useCallback` 包裹的函数依赖于它，或者你在 [`useEffect`](/reference/react/useEffect) 中依赖这个函数。

在其他情况下，把函数包裹在 `useCallback` 中没有任何收益。这样做也没有明显的坏处，所以有些团队选择不去考虑单个场景，而是尽可能地进行记忆化。缺点是代码可读性会变差。此外，并非所有记忆化都有效：只要有一个“总是新的”值，就足以破坏整个组件的记忆化。

请注意，`useCallback` 并不会阻止函数的*创建*。你始终都会创建一个函数（这没问题！），但如果没有任何变化，React 会忽略它并返回给你一个缓存的函数。

**在实践中，你可以通过遵循一些原则来让很多记忆化变得不必要：**

1. 当某个组件在视觉上包裹其他组件时，让它[接受 JSX 作为 children。](/learn/passing-props-to-a-component#passing-jsx-as-children) 这样，如果包裹组件更新了自己的 state，React 就知道它的子组件不需要重新渲染。
2. 优先使用局部 state，不要比必要的程度更进一步地[提升 state。](/learn/sharing-state-between-components) 不要把表单、某项是否被悬停等短暂状态放在组件树顶层或全局状态库中。
3. 保持你的[渲染逻辑纯净。](/learn/keeping-components-pure) 如果重新渲染某个组件会造成问题或产生明显的视觉异常，那说明你的组件有 bug！请修复 bug，而不是添加记忆化。
4. 避免[不必要的会更新 state 的 Effects。](/learn/you-might-not-need-an-effect) React 应用中大多数性能问题都来自 Effects 触发的更新链，它们会导致组件一遍又一遍地渲染。
5. 尝试[移除 Effects 中不必要的依赖项。](/learn/removing-effect-dependencies) 例如，与其做记忆化，不如把某些对象或函数移到 Effect 内部，或者移到组件外部，通常更简单。

如果某个具体交互仍然感觉卡顿，可以[使用 React Developer Tools 的 profiler](https://legacy.reactjs.org/blog/2018/09/10/introducing-the-react-profiler.html) 看看哪些组件最能从记忆化中受益，并在需要的地方添加记忆化。这些原则会让你的组件更容易调试和理解，所以无论如何都值得遵循。从长远来看，我们正在研究[自动进行记忆化](https://www.youtube.com/watch?v=lGEMwh32soc)来一劳永逸地解决这个问题。

</DeepDive>

<Recipes titleText="useCallback 与直接声明函数的区别" titleId="examples-rerendering">

#### 使用 `useCallback` 和 `memo` 跳过重新渲染 {/*skipping-re-rendering-with-usecallback-and-memo*/}

在这个例子中，`ShippingForm` 组件被**人为地放慢了速度**，这样你就可以看到当你渲染的 React 组件确实很慢时会发生什么。试着增加计数器并切换主题。

增加计数器会感觉很慢，因为它迫使被放慢的 `ShippingForm` 重新渲染。这是预期中的，因为计数器已经变化了，所以你需要把用户的新选择反映到屏幕上。

接着，试着切换主题。**多亏了 `useCallback` 和 [`memo`](/reference/react/memo) 一起使用，即使有人工减速，它仍然很快！** `ShippingForm` 跳过了重新渲染，因为 `handleSubmit` 函数没有变化。`handleSubmit` 函数没有变化，是因为 `productId` 和 `referrer`（你的 `useCallback` 依赖项）自上次渲染以来都没有变化。

<Sandpack>

```js src/App.js
import { useState } from 'react';
import ProductPage from './ProductPage.js';

export default function App() {
  const [isDark, setIsDark] = useState(false);
  return (
    <>
      <label>
        <input
          type="checkbox"
          checked={isDark}
          onChange={e => setIsDark(e.target.checked)}
        />
        深色模式
      </label>
      <hr />
      <ProductPage
        referrerId="wizard_of_oz"
        productId={123}
        theme={isDark ? 'dark' : 'light'}
      />
    </>
  );
}
```

```js src/ProductPage.js active
import { useCallback } from 'react';
import ShippingForm from './ShippingForm.js';

export default function ProductPage({ productId, referrer, theme }) {
  const handleSubmit = useCallback((orderDetails) => {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails,
    });
  }, [productId, referrer]);

  return (
    <div className={theme}>
      <ShippingForm onSubmit={handleSubmit} />
    </div>
  );
}

function post(url, data) {
  // 想象一下这会发送一个请求...
  console.log('POST /' + url);
  console.log(data);
}
```

```js {expectedErrors: {'react-compiler': [7, 8]}} src/ShippingForm.js
import { memo, useState } from 'react';

const ShippingForm = memo(function ShippingForm({ onSubmit }) {
  const [count, setCount] = useState(1);

  console.log('[人为减速] Rendering <ShippingForm />');
  let startTime = performance.now();
  while (performance.now() - startTime < 500) {
    // 什么也不做 500 毫秒，以模拟极慢的代码
  }

  function handleSubmit(e) {
    e.preventDefault();
    const formData = new FormData(e.target);
    const orderDetails = {
      ...Object.fromEntries(formData),
      count
    };
    onSubmit(orderDetails);
  }

  return (
    <form onSubmit={handleSubmit}>
      <p><b>注意：<code>ShippingForm</code> 被人为放慢了！</b></p>
      <label>
        商品数量：
        <button type="button" onClick={() => setCount(count - 1)}>–</button>
        {count}
        <button type="button" onClick={() => setCount(count + 1)}>+</button>
      </label>
      <label>
        街道：
        <input name="street" />
      </label>
      <label>
        城市：
        <input name="city" />
      </label>
      <label>
        邮政编码：
        <input name="zipCode" />
      </label>
      <button type="submit">提交</button>
    </form>
  );
});

export default ShippingForm;
```

```css
label {
  display: block; margin-top: 10px;
}

input {
  margin-left: 5px;
}

button[type="button"] {
  margin: 5px;
}

.dark {
  background-color: black;
  color: white;
}

.light {
  background-color: white;
  color: black;
}
```

</Sandpack>

<Solution />

#### 组件总是重新渲染 {/*always-re-rendering-a-component*/}

在这个例子中，`ShippingForm` 的实现同样被**人为地放慢了速度**，这样你就可以看到当某个你正在渲染的 React 组件确实很慢时会发生什么。试着增加计数器并切换主题。

与上一个例子不同，现在切换主题也很慢！这是因为**这个版本里没有 `useCallback` 调用，** 所以 `handleSubmit` 总是一个新函数，而被放慢的 `ShippingForm` 组件无法跳过重新渲染。

<Sandpack>

```js src/App.js
import { useState } from 'react';
import ProductPage from './ProductPage.js';

export default function App() {
  const [isDark, setIsDark] = useState(false);
  return (
    <>
      <label>
        <input
          type="checkbox"
          checked={isDark}
          onChange={e => setIsDark(e.target.checked)}
        />
        深色模式
      </label>
      <hr />
      <ProductPage
        referrerId="wizard_of_oz"
        productId={123}
        theme={isDark ? 'dark' : 'light'}
      />
    </>
  );
}
```

```js src/ProductPage.js active
import ShippingForm from './ShippingForm.js';

export default function ProductPage({ productId, referrer, theme }) {
  function handleSubmit(orderDetails) {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails,
    });
  }

  return (
    <div className={theme}>
      <ShippingForm onSubmit={handleSubmit} />
    </div>
  );
}

function post(url, data) {
  // 想象一下这会发送一个请求...
  console.log('POST /' + url);
  console.log(data);
}
```

```js {expectedErrors: {'react-compiler': [7, 8]}} src/ShippingForm.js
import { memo, useState } from 'react';

const ShippingForm = memo(function ShippingForm({ onSubmit }) {
  const [count, setCount] = useState(1);

  console.log('[人为减速] Rendering <ShippingForm />');
  let startTime = performance.now();
  while (performance.now() - startTime < 500) {
    // 什么也不做 500 毫秒，以模拟极慢的代码
  }

  function handleSubmit(e) {
    e.preventDefault();
    const formData = new FormData(e.target);
    const orderDetails = {
      ...Object.fromEntries(formData),
      count
    };
    onSubmit(orderDetails);
  }

  return (
    <form onSubmit={handleSubmit}>
      <p><b>注意：<code>ShippingForm</code> 被人为放慢了！</b></p>
      <label>
        商品数量：
        <button type="button" onClick={() => setCount(count - 1)}>–</button>
        {count}
        <button type="button" onClick={() => setCount(count + 1)}>+</button>
      </label>
      <label>
        街道：
        <input name="street" />
      </label>
      <label>
        城市：
        <input name="city" />
      </label>
      <label>
        邮政编码：
        <input name="zipCode" />
      </label>
      <button type="submit">提交</button>
    </form>
  );
});

export default ShippingForm;
```

```css
label {
  display: block; margin-top: 10px;
}

input {
  margin-left: 5px;
}

button[type="button"] {
  margin: 5px;
}

.dark {
  background-color: black;
  color: white;
}

.light {
  background-color: white;
  color: black;
}
```

</Sandpack>


不过，下面是同样的代码，**但去掉了人为减速。** 缺少 `useCallback` 会不会让你感觉明显不同？

<Sandpack>

```js src/App.js
import { useState } from 'react';
import ProductPage from './ProductPage.js';

export default function App() {
  const [isDark, setIsDark] = useState(false);
  return (
    <>
      <label>
        <input
          type="checkbox"
          checked={isDark}
          onChange={e => setIsDark(e.target.checked)}
        />
        深色模式
      </label>
      <hr />
      <ProductPage
        referrerId="wizard_of_oz"
        productId={123}
        theme={isDark ? 'dark' : 'light'}
      />
    </>
  );
}
```

```js src/ProductPage.js active
import ShippingForm from './ShippingForm.js';

export default function ProductPage({ productId, referrer, theme }) {
  function handleSubmit(orderDetails) {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails,
    });
  }

  return (
    <div className={theme}>
      <ShippingForm onSubmit={handleSubmit} />
    </div>
  );
}

function post(url, data) {
  // 想象一下这会发送一个请求...
  console.log('POST /' + url);
  console.log(data);
}
```

```js src/ShippingForm.js
import { memo, useState } from 'react';

const ShippingForm = memo(function ShippingForm({ onSubmit }) {
  const [count, setCount] = useState(1);

  console.log('Rendering <ShippingForm />');

  function handleSubmit(e) {
    e.preventDefault();
    const formData = new FormData(e.target);
    const orderDetails = {
      ...Object.fromEntries(formData),
      count
    };
    onSubmit(orderDetails);
  }

  return (
    <form onSubmit={handleSubmit}>
      <label>
        商品数量：
        <button type="button" onClick={() => setCount(count - 1)}>–</button>
        {count}
        <button type="button" onClick={() => setCount(count + 1)}>+</button>
      </label>
      <label>
        街道：
        <input name="street" />
      </label>
      <label>
        城市：
        <input name="city" />
      </label>
      <label>
        邮政编码：
        <input name="zipCode" />
      </label>
      <button type="submit">提交</button>
    </form>
  );
});

export default ShippingForm;
```

```css
label {
  display: block; margin-top: 10px;
}

input {
  margin-left: 5px;
}

button[type="button"] {
  margin: 5px;
}

.dark {
  background-color: black;
  color: white;
}

.light {
  background-color: white;
  color: black;
}
```

</Sandpack>


很多时候，不使用记忆化的代码也能正常工作。如果你的交互足够快，就不需要记忆化。

请记住，你需要在生产模式下运行 React，禁用 [React Developer Tools](/learn/react-developer-tools)，并使用与你的应用用户相似的设备，才能真实地感受到到底是什么在让应用变慢。

<Solution />

</Recipes>

---

### 使用 memoized callback 更新 state {/*updating-state-from-a-memoized-callback*/}

有时，你可能需要在一个记忆化的回调中基于之前的 state 来更新 state。

这个 `handleAddTodo` 函数把 `todos` 指定为依赖项，因为它会根据它计算下一个 todos：

```js {6,7}
function TodoList() {
  const [todos, setTodos] = useState([]);

  const handleAddTodo = useCallback((text) => {
    const newTodo = { id: nextId++, text };
    setTodos([...todos, newTodo]);
  }, [todos]);
  // ...
```

通常你会希望记忆化函数拥有尽可能少的依赖项。当你只是在读取某个 state 来计算下一个 state 时，可以通过改为传入一个[更新函数](/reference/react/useState#updating-state-based-on-the-previous-state)来移除该依赖项：

```js {6,7}
function TodoList() {
  const [todos, setTodos] = useState([]);

  const handleAddTodo = useCallback((text) => {
    const newTodo = { id: nextId++, text };
    setTodos(todos => [...todos, newTodo]);
  }, []); // ✅ 不再需要 todos 依赖项
  // ...
```

这里你不是把 `todos` 作为依赖项并在内部读取它，而是向 React 传递一个关于*如何*更新 state 的指令（`todos => [...todos, newTodo]`）。[阅读更多关于更新函数的内容。](/reference/react/useState#updating-state-based-on-the-previous-state)

---

### 防止 Effect 过于频繁地触发 {/*preventing-an-effect-from-firing-too-often*/}

有时，你可能想在 [Effect](/learn/synchronizing-with-effects) 中调用一个函数：

```js {4-9,12}
function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  function createOptions() {
    return {
      serverUrl: 'https://localhost:1234',
      roomId: roomId
    };
  }

  useEffect(() => {
    const options = createOptions();
    const connection = createConnection(options);
    connection.connect();
    // ...
```

这会带来一个问题。[每个响应式值都必须声明为你 Effect 的依赖项。](/learn/lifecycle-of-reactive-effects#react-verifies-that-you-specified-every-reactive-value-as-a-dependency) 但是，如果你把 `createOptions` 声明为依赖项，它会导致你的 Effect 不断重新连接到聊天室：


```js {6}
  useEffect(() => {
    const options = createOptions();
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [createOptions]); // 🔴 问题：这个依赖项会在每次渲染时变化
  // ...
```

要解决这个问题，你可以把需要在 Effect 中调用的函数包裹进 `useCallback`：

```js {4-9,16}
function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  const createOptions = useCallback(() => {
    return {
      serverUrl: 'https://localhost:1234',
      roomId: roomId
    };
  }, [roomId]); // ✅ 只有 roomId 变化时才会变化

  useEffect(() => {
    const options = createOptions();
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [createOptions]); // ✅ 只有 createOptions 变化时才会变化
  // ...
```

这可以确保如果 `roomId` 相同，`createOptions` 函数在多次重新渲染之间保持相同。**不过，更好的做法是直接移除对函数依赖项的需要。** 把你的函数移到 Effect *内部*：

```js {5-10,16}
function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  useEffect(() => {
    function createOptions() { // ✅ 不需要 useCallback 或函数依赖项！
      return {
        serverUrl: 'https://localhost:1234',
        roomId: roomId
      };
    }

    const options = createOptions();
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]); // ✅ 只有 roomId 变化时才会变化
  // ...
```

现在你的代码更简单了，也不需要 `useCallback`。[了解更多关于移除 Effect 依赖项的内容。](/learn/removing-effect-dependencies#move-dynamic-objects-and-functions-inside-your-effect)

---

### 优化自定义 Hook {/*optimizing-a-custom-hook*/}

如果你正在编写一个[自定义 Hook](/learn/reusing-logic-with-custom-hooks)，建议把它返回的任何函数都包裹在 `useCallback` 中：

```js {4-6,8-10}
function useRouter() {
  const { dispatch } = useContext(RouterStateContext);

  const navigate = useCallback((url) => {
    dispatch({ type: 'navigate', url });
  }, [dispatch]);

  const goBack = useCallback(() => {
    dispatch({ type: 'back' });
  }, [dispatch]);

  return {
    navigate,
    goBack,
  };
}
```

这可以确保你的 Hook 的使用者在需要时能够优化他们自己的代码。

---

## 故障排查 {/*troubleshooting*/}

### 每次我的组件渲染时，`useCallback` 都会返回不同的函数 {/*every-time-my-component-renders-usecallback-returns-a-different-function*/}

确保你已经将依赖数组作为第二个参数传入！

如果你忘记传入依赖数组，`useCallback` 每次都会返回一个新函数：

```js {7}
function ProductPage({ productId, referrer }) {
  const handleSubmit = useCallback((orderDetails) => {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails,
    });
  }); // 🔴 每次都会返回一个新函数：没有依赖数组
  // ...
```

这是修正后的版本，第二个参数传入了依赖数组：

```js {7}
function ProductPage({ productId, referrer }) {
  const handleSubmit = useCallback((orderDetails) => {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails,
    });
  }, [productId, referrer]); // ✅ 不会不必要地返回新函数
  // ...
```

如果这样还不行，那么问题在于你的至少一个依赖项与上一次渲染时不同。你可以通过手动将依赖项记录到控制台来调试这个问题：

```js {5}
  const handleSubmit = useCallback((orderDetails) => {
    // ..
  }, [productId, referrer]);

  console.log([productId, referrer]);
```

然后，你可以在控制台中右键点击不同重新渲染时输出的数组，并选择“Store as a global variable”，把它们都保存下来。假设第一个被保存为 `temp1`，第二个被保存为 `temp2`，那么你就可以使用浏览器控制台检查这两个数组中的每个依赖项是否相同：

```js
Object.is(temp1[0], temp2[0]); // 第一个依赖项在两个数组中是否相同？
Object.is(temp1[1], temp2[1]); // 第二个依赖项在两个数组中是否相同？
Object.is(temp1[2], temp2[2]); // ... 以此类推，检查每个依赖项 ...
```

当你找到是哪个依赖项破坏了记忆化时，要么想办法移除它，要么也[将它进行记忆化。](/reference/react/useMemo#memoizing-a-dependency-of-another-hook)

---

### 我需要在循环中为每个列表项调用 `useCallback`，但这是不允许的 {/*i-need-to-call-usememo-for-each-list-item-in-a-loop-but-its-not-allowed*/}

假设 `Chart` 组件被 [`memo`](/reference/react/memo) 包裹。你希望在 `ReportList` 组件重新渲染时，跳过列表中每个 `Chart` 的重新渲染。然而，你不能在循环中调用 `useCallback`：

```js {expectedErrors: {'react-compiler': [6]}} {5-14}
function ReportList({ items }) {
  return (
    <article>
      {items.map(item => {
        // 🔴 你不能像这样在循环中调用 useCallback：
        const handleClick = useCallback(() => {
          sendReport(item)
        }, [item]);

        return (
          <figure key={item.id}>
            <Chart onClick={handleClick} />
          </figure>
        );
      })}
    </article>
  );
}
```

相反，请为单个项提取一个组件，并在那里放置 `useCallback`：

```js {5,12-21}
function ReportList({ items }) {
  return (
    <article>
      {items.map(item =>
        <Report key={item.id} item={item} />
      )}
    </article>
  );
}

function Report({ item }) {
  // ✅ 在顶层调用 useCallback：
  const handleClick = useCallback(() => {
    sendReport(item)
  }, [item]);

  return (
    <figure>
      <Chart onClick={handleClick} />
    </figure>
  );
}
```

或者，你可以移除上一个示例中的 `useCallback`，改为将 `Report` 本身包裹在 [`memo`](/reference/react/memo) 中。如果 `item` 属性不变，`Report` 就会跳过重新渲染，因此 `Chart` 也会跳过重新渲染：

```js {5,6-8,15}
function ReportList({ items }) {
  // ...
}

const Report = memo(function Report({ item }) {
  function handleClick() {
    sendReport(item);
  }

  return (
    <figure>
      <Chart onClick={handleClick} />
    </figure>
  );
});
```
