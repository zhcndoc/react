---
title: "'use client'"
titleForTitleTag: "'use client' 指令"
---

<RSC>

`'use client'` 用于 [React Server Components](/reference/rsc/server-components)。

</RSC>


<Intro>

`'use client'` 让你标记哪些代码在客户端运行。

</Intro>

<InlineToc />

---

## 参考 {/*reference*/}

### `'use client'` {/*use-client*/}

在文件顶部添加 `'use client'`，以标记该模块及其传递依赖为客户端代码。

```js {1}
'use client';

import { useState } from 'react';
import { formatDate } from './formatters';
import Button from './button';

export default function RichTextEditor({ timestamp, text }) {
  const date = formatDate(timestamp);
  // ...
  const editButton = <Button />;
  // ...
}
```

当一个标记了 `'use client'` 的文件从 Server Component 中被导入时，[兼容的打包器](/learn/creating-a-react-app#full-stack-frameworks) 会将该模块导入视为服务端运行代码与客户端运行代码之间的边界。

作为 `RichTextEditor` 的依赖，`formatDate` 和 `Button` 也会在客户端求值，无论它们的模块是否包含 `'use client'` 指令。请注意，同一个模块在从服务端代码导入时可能会在服务端求值，而在从客户端代码导入时可能会在客户端求值。

#### 注意事项 {/*caveats*/}

* `'use client'` 必须位于文件最开头，放在任何导入或其他代码之上（注释可以）。它们必须使用单引号或双引号书写，但不能使用反引号。
* 当一个 `'use client'` 模块从另一个客户端渲染的模块中导入时，该指令不起作用。
* 当一个组件模块包含 `'use client'` 指令时，对该组件的任何使用都保证是 Client Component。不过，即使没有 `'use client'` 指令，组件仍然可能在客户端求值。
	* 如果组件定义在带有 `'use client'` 指令的模块中，或者该组件是在包含 `'use client'` 指令的模块中作为传递依赖被导入并调用的，则该组件的使用会被视为 Client Component。否则，它是 Server Component。
* 被标记为客户端求值的代码不限于组件。Client 模块子树中的所有代码都会被发送到客户端并由客户端运行。
* 当一个服务端求值的模块从 `'use client'` 模块中导入值时，这些值必须要么是 React 组件，要么是[受支持的可序列化属性值](#passing-props-from-server-to-client-components)，以便传递给 Client Component。任何其他用例都会抛出异常。

### `'use client'` 如何标记客户端代码 {/*how-use-client-marks-client-code*/}

在 React 应用中，组件通常会被拆分到不同的文件中，或 [模块](/learn/importing-and-exporting-components#exporting-and-importing-a-component) 中。

对于使用 React Server Components 的应用，默认情况下应用会在服务端渲染。`'use client'` 在 [模块依赖树](/learn/understanding-your-ui-as-a-tree#the-module-dependency-tree) 中引入了服务端-客户端边界，从而有效地创建出一个 Client 模块子树。

为了更好地说明这一点，请看下面这个 React Server Components 应用。

<Sandpack>

```js src/App.js
import FancyText from './FancyText';
import InspirationGenerator from './InspirationGenerator';
import Copyright from './Copyright';

export default function App() {
  return (
    <>
      <FancyText title text="Get Inspired App" />
      <InspirationGenerator>
        <Copyright year={2004} />
      </InspirationGenerator>
    </>
  );
}

```

```js src/FancyText.js
export default function FancyText({title, text}) {
  return title
    ? <h1 className='fancy title'>{text}</h1>
    : <h3 className='fancy cursive'>{text}</h3>
}
```

```js src/InspirationGenerator.js
'use client';

import { useState } from 'react';
import inspirations from './inspirations';
import FancyText from './FancyText';

export default function InspirationGenerator({children}) {
  const [index, setIndex] = useState(0);
  const quote = inspirations[index];
  const next = () => setIndex((index + 1) % inspirations.length);

  return (
    <>
      <p>你的励志语录是：</p>
      <FancyText text={quote} />
      <button onClick={next}>再次激励我</button>
      {children}
    </>
  );
}
```

```js src/Copyright.js
export default function Copyright({year}) {
  return <p className='small'>©️ {year}</p>;
}
```

```js src/inspirations.js
export default [
  "Don’t let yesterday take up too much of today.” — Will Rogers",
  "Ambition is putting a ladder against the sky.",
  "A joy that's shared is a joy made double.",
];
```

```css
.fancy {
  font-family: 'Georgia';
}
.title {
  color: #007AA3;
  text-decoration: underline;
}
.cursive {
  font-style: italic;
}
.small {
  font-size: 10px;
}
```

</Sandpack>

在这个示例应用的模块依赖树中，`InspirationGenerator.js` 中的 `'use client'` 指令会将该模块及其所有传递依赖标记为 Client 模块。以 `InspirationGenerator.js` 为起点的子树现在被标记为 Client 模块。

<Diagram name="use_client_module_dependency" height={250} width={545} alt="一张树状图，顶层节点表示模块 'App.js'。'App.js' 有三个子节点：'Copyright.js'、'FancyText.js' 和 'InspirationGenerator.js'。'InspirationGenerator.js' 有两个子节点：'FancyText.js' 和 'inspirations.js'。包含并位于 'InspirationGenerator.js' 下方的节点都有黄色背景，以表示由于 'InspirationGenerator.js' 中的 'use client' 指令，该子图是客户端渲染的。">
`'use client'` 将 React Server Components 应用的模块依赖树分割开来，把 `InspirationGenerator.js` 及其所有依赖标记为客户端渲染。
</Diagram>

在渲染期间，框架会先对根组件进行服务端渲染，并继续遍历 [渲染树](/learn/understanding-your-ui-as-a-tree#the-render-tree)，跳过对任何从客户端标记代码导入的代码的求值。

随后，服务端渲染的那部分渲染树会被发送到客户端。客户端在下载了其客户端代码后，再完成树中剩余部分的渲染。

<Diagram name="use_client_render_tree" height={250} width={500} alt="一张树状图，其中每个节点代表一个组件及其子组件。顶层节点标记为 'App'，它有两个子组件 'InspirationGenerator' 和 'FancyText'。'InspirationGenerator' 有两个子组件，'FancyText' 和 'Copyright'。'InspirationGenerator' 及其子组件 'FancyText' 都被标记为客户端渲染。">
React Server Components 应用的渲染树。`InspirationGenerator` 及其子组件 `FancyText` 是从客户端标记代码导出的组件，并被视为 Client Components。
</Diagram>

我们引入以下定义：

* **Client Components** 是渲染树中在客户端渲染的组件。
* **Server Components** 是渲染树中在服务端渲染的组件。

结合这个示例应用来看，`App`、`FancyText` 和 `Copyright` 都是服务端渲染并被视为 Server Components。由于 `InspirationGenerator.js` 及其传递依赖被标记为客户端代码，组件 `InspirationGenerator` 及其子组件 `FancyText` 是 Client Components。

<DeepDive>
#### `FancyText` 如何既是 Server Component 又是 Client Component？ {/*how-is-fancytext-both-a-server-and-a-client-component*/}

根据上面的定义，组件 `FancyText` 既是 Server Component 又是 Client Component，这是怎么回事？

首先，让我们澄清一下，“组件”这个词并不十分精确。下面只是“组件”可以被理解的两种方式：

1. “组件”可以指 **组件定义**。在大多数情况下，这会是一个函数。

```js
// 这是一个组件定义
function MyComponent() {
  return <p>My Component</p>
}
```

2. “组件”也可以指其定义的 **组件使用**。
```js
import MyComponent from './MyComponent';

function App() {
  // 这是一个组件使用
  return <MyComponent />;
}
```

通常，在解释概念时这种不精确并不重要，但在这里它很重要。

当我们谈论 Server 或 Client Components 时，我们指的是组件使用。

* 如果组件定义在带有 `'use client'` 指令的模块中，或者该组件被导入并在 Client Component 中调用，那么该组件使用就是 Client Component。
* 否则，该组件使用就是 Server Component。


<Diagram name="use_client_render_tree" height={150} width={450} alt="一张树状图，其中每个节点代表一个组件及其子组件。顶层节点标记为 'App'，它有两个子组件 'InspirationGenerator' 和 'FancyText'。'InspirationGenerator' 有两个子组件，'FancyText' 和 'Copyright'。'InspirationGenerator' 及其子组件 'FancyText' 都被标记为客户端渲染。">一棵渲染树说明了组件使用。</Diagram>

回到 `FancyText` 这个问题，我们可以看到该组件定义本身 _没有_ `'use client'` 指令，而且它有两种使用方式。

`FancyText` 作为 `App` 的子组件时，这种使用会被标记为 Server Component。当 `FancyText` 在 `InspirationGenerator` 中被导入并调用时，由于 `InspirationGenerator` 包含 `'use client'` 指令，这种 `FancyText` 的使用就是 Client Component。

这意味着，`FancyText` 的组件定义既会在服务端求值，也会被客户端下载，用于渲染其 Client Component 的使用场景。

</DeepDive>

<DeepDive>

#### 为什么 `Copyright` 是 Server Component？ {/*why-is-copyright-a-server-component*/}

因为 `Copyright` 作为 Client Component `InspirationGenerator` 的子组件被渲染，你可能会惊讶它竟然是 Server Component。

请记住，`'use client'` 定义的是服务端和客户端代码之间在 _模块依赖树_ 上的边界，而不是在渲染树上。

<Diagram name="use_client_module_dependency" height={200} width={500} alt="一张树状图，顶层节点表示模块 'App.js'。'App.js' 有三个子节点：'Copyright.js'、'FancyText.js' 和 'InspirationGenerator.js'。'InspirationGenerator.js' 有两个子节点：'FancyText.js' 和 'inspirations.js'。包含并位于 'InspirationGenerator.js' 下方的节点都有黄色背景，以表示由于 'InspirationGenerator.js' 中的 'use client' 指令，该子图是客户端渲染的。">
`'use client'` 在模块依赖树上定义了服务端和客户端代码之间的边界。
</Diagram>

在模块依赖树中，我们看到 `App.js` 从 `Copyright.js` 模块中导入并调用了 `Copyright`。由于 `Copyright.js` 不包含 `'use client'` 指令，因此该组件使用是在服务端渲染的。`App` 作为根组件，也是服务端渲染的。

Client Components 可以渲染 Server Components，因为你可以将 JSX 作为 props 传递。在这种情况下，`InspirationGenerator` 通过 [children](/learn/passing-props-to-a-component#passing-jsx-as-children) 接收 `Copyright`。但是，`InspirationGenerator` 模块从未直接导入 `Copyright` 模块，也从未调用该组件，这一切都是由 `App` 完成的。事实上，在 `InspirationGenerator` 开始渲染之前，`Copyright` 组件就已经完全执行完了。

关键在于，组件之间的父子渲染关系并不保证它们处于相同的渲染环境中。

</DeepDive>

### 何时使用 `'use client'` {/*when-to-use-use-client*/}

借助 `'use client'`，你可以决定哪些组件是 Client Components。由于 Server Components 是默认行为，下面简要概述一下 Server Components 的优势和限制，以帮助你判断何时需要将某些内容标记为客户端渲染。

为简单起见，我们讨论的是 Server Components，但同样的原则也适用于应用中所有在服务端运行的代码。

#### Server Components 的优势 {/*advantages*/}
* Server Components 可以减少发送给客户端并由客户端运行的代码量。只有 Client 模块会被客户端打包和求值。
* Server Components 受益于在服务端运行。它们可以访问本地文件系统，并且在数据获取和网络请求方面可能具有较低延迟。

#### Server Components 的限制 {/*limitations*/}
* Server Components 无法支持交互，因为事件处理器必须由客户端注册和触发。
	* 例如，像 `onClick` 这样的事件处理器只能在 Client Components 中定义。
* Server Components 不能使用大多数 Hooks。
	* 当 Server Components 被渲染时，它们的输出本质上是一个供客户端渲染的组件列表。Server Components 在渲染后不会持续保存在内存中，因此也不能拥有自己的状态。

### Server Components 返回的可序列化类型 {/*serializable-types*/}

和任何 React 应用一样，父组件会向子组件传递数据。由于它们运行在不同的环境中，从 Server Component 向 Client Component 传递数据需要额外注意。

从 Server Component 传递给 Client Component 的 prop 值必须是可序列化的。

可序列化的 props 包括：
* 原始类型
	* [string](https://developer.mozilla.org/en-US/docs/Glossary/String)
	* [number](https://developer.mozilla.org/en-US/docs/Glossary/Number)
	* [bigint](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/BigInt)
	* [boolean](https://developer.mozilla.org/en-US/docs/Glossary/Boolean)
	* [undefined](https://developer.mozilla.org/en-US/docs/Glossary/Undefined)
	* [null](https://developer.mozilla.org/en-US/docs/Glossary/Null)
	* [symbol](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Symbol)，仅限通过 [`Symbol.for`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Symbol/for) 在全局 Symbol 注册表中注册的 symbol
* 包含可序列化值的可迭代对象
	* [String](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String)
	* [Array](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array)
	* [Map](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map)
	* [Set](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set)
	* [TypedArray](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/TypedArray) 和 [ArrayBuffer](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/ArrayBuffer)
* [Date](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date)
* 普通 [对象](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object)：使用 [对象初始化器](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Object_initializer) 创建的对象，并且其属性是可序列化的
* 作为 [Server Functions](/reference/rsc/server-functions) 的函数
* Client 或 Server Component 元素（JSX）
* [Promises](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)

特别地，以下内容不受支持：
* 不是从客户端标记模块导出，或未标记为 [`'use server'`](/reference/rsc/use-server) 的 [函数](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function)
* [类](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Objects/Classes_in_JavaScript)
* 任何类的实例对象（不包括前面提到的内置类型），或具有 [null 原型](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object#null-prototype_objects) 的对象
* 未全局注册的 symbol，例如 `Symbol('my new symbol')`


## 用法 {/*usage*/}

### 构建交互性和状态 {/*building-with-interactivity-and-state*/}

<Sandpack>

```js src/App.js
'use client';

import { useState } from 'react';

export default function Counter({initialValue = 0}) {
  const [countValue, setCountValue] = useState(initialValue);
  const increment = () => setCountValue(countValue + 1);
  const decrement = () => setCountValue(countValue - 1);
  return (
    <>
      <h2>计数值：{countValue}</h2>
      <button onClick={increment}>+1</button>
      <button onClick={decrement}>-1</button>
    </>
  );
}
```

</Sandpack>

由于 `Counter` 需要 `useState` Hook 和事件处理程序来增加或减少值，因此该组件必须是客户端组件，并且需要在顶部添加 `'use client'` 指令。

相比之下，不包含交互的 UI 渲染组件不需要是客户端组件。

```js
import { readFile } from 'node:fs/promises';
import Counter from './Counter';

export default async function CounterContainer() {
  const initialValue = await readFile('/path/to/counter_value');
  return <Counter initialValue={initialValue} />
}
```

例如，`Counter` 的父组件 `CounterContainer` 不需要 `'use client'`，因为它没有交互性，也不使用状态。此外，`CounterContainer` 必须是服务端组件，因为它会在服务器上从本地文件系统读取内容，而这只有在服务端组件中才可行。

还有一些组件不使用任何服务端或仅客户端特性，因此可以不关心它们在哪里渲染。在我们前面的示例中，`FancyText` 就是这样的一个组件。

```js
export default function FancyText({title, text}) {
  return title
    ? <h1 className='fancy title'>{text}</h1>
    : <h3 className='fancy cursive'>{text}</h3>
}
```

在这种情况下，我们不添加 `'use client'` 指令，这会导致当从服务端组件引用时，发送到浏览器的是 `FancyText` 的 _输出_（而不是其源代码）。如前面的 Inspirations 应用示例所示，`FancyText` 可作为服务端组件或客户端组件使用，具体取决于它被导入和使用的位置。

但如果 `FancyText` 的 HTML 输出相对于其源代码（包括依赖项）来说很大，那么强制它始终作为客户端组件可能会更高效。返回很长 SVG 路径字符串的组件就是一种可能更适合强制为客户端组件的情况。

### 使用客户端 API {/*using-client-apis*/}

你的 React 应用可能会使用特定于客户端的 API，例如浏览器用于 Web 存储、音视频处理以及设备硬件等的 API，此外还有[其他](https://developer.mozilla.org/en-US/docs/Web/API)。

在这个示例中，组件使用 [DOM API](https://developer.mozilla.org/en-US/docs/Glossary/DOM) 来操作一个 [`canvas`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/canvas) 元素。由于这些 API 只能在浏览器中使用，因此必须将其标记为客户端组件。

```js
'use client';

import {useRef, useEffect} from 'react';

export default function Circle() {
  const ref = useRef(null);
  useLayoutEffect(() => {
    const canvas = ref.current;
    const context = canvas.getContext('2d');
    context.reset();
    context.beginPath();
    context.arc(100, 75, 50, 0, 2 * Math.PI);
    context.stroke();
  });
  return <canvas ref={ref} />;
}
```

### 使用第三方库 {/*using-third-party-libraries*/}

在 React 应用中，你通常会利用第三方库来处理常见的 UI 模式或逻辑。

这些库可能依赖于组件 Hooks 或客户端 API。使用以下任何 React API 的第三方组件都必须在客户端运行：
* [createContext](/reference/react/createContext)
* [`react`](/reference/react/hooks) 和 [`react-dom`](/reference/react-dom/hooks) Hooks，不包括 [`use`](/reference/react/use) 和 [`useId`](/reference/react/useId)
* [forwardRef](/reference/react/forwardRef)
* [memo](/reference/react/memo)
* [startTransition](/reference/react/startTransition)
* 如果它们使用了客户端 API，例如 DOM 插入或原生平台视图

如果这些库已经更新为与 React Server Components 兼容，那么它们本身就会包含 `'use client'` 标记，这样你就可以直接从服务端组件中使用它们。如果某个库尚未更新，或者某个组件需要诸如事件处理程序之类只能在客户端指定的 props，那么你可能需要在第三方客户端组件和你希望使用它的服务端组件之间添加你自己的客户端组件文件。

[TODO]: <> (故障排查 - 需要用例)
