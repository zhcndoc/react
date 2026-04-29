---
title: 使用 TypeScript
re: https://github.com/reactjs/react.dev/issues/5960
---

<Intro>

TypeScript 是一种为 JavaScript 代码库添加类型定义的流行方式。开箱即用的情况下，TypeScript [支持 JSX](/learn/writing-markup-with-jsx)，只需将 [`@types/react`](https://www.npmjs.com/package/@types/react) 和 [`@types/react-dom`](https://www.npmjs.com/package/@types/react-dom) 添加到你的项目中，就可以获得完整的 React Web 支持。

</Intro>

<YouWillLearn>

* [在 React 组件中使用 TypeScript](/learn/typescript#typescript-with-react-components)
* [使用 Hooks 进行类型标注的示例](/learn/typescript#example-hooks)
* [来自 `@types/react` 的常用类型](/learn/typescript#useful-types)
* [进一步学习的地点](/learn/typescript#further-learning)

</YouWillLearn>

## 安装 {/*installation*/}

所有[生产级 React 框架](/learn/creating-a-react-app#full-stack-frameworks)都支持使用 TypeScript。请根据对应框架的指南进行安装：

- [Next.js](https://nextjs.org/docs/app/building-your-application/configuring/typescript)
- [Remix](https://remix.run/docs/en/1.19.2/guides/typescript)
- [Gatsby](https://www.gatsbyjs.com/docs/how-to/custom-configuration/typescript/)
- [Expo](https://docs.expo.dev/guides/typescript/)

### 将 TypeScript 添加到现有 React 项目中 {/*adding-typescript-to-an-existing-react-project*/}

要安装最新版本的 React 类型定义：

<TerminalBlock>
npm install --save-dev @types/react @types/react-dom
</TerminalBlock>

需要在你的 `tsconfig.json` 中设置以下编译器选项：

1. `dom` 必须包含在 [`lib`](https://www.typescriptlang.org/tsconfig/#lib) 中（注意：如果未指定 `lib` 选项，默认会包含 `dom`）。
2. [`jsx`](https://www.typescriptlang.org/tsconfig/#jsx) 必须设置为有效选项之一。对于大多数应用，`preserve` 就足够了。
  如果你正在发布一个库，请查阅 [`jsx` 文档](https://www.typescriptlang.org/tsconfig/#jsx) 来决定选择哪个值。

## 在 React 组件中使用 TypeScript {/*typescript-with-react-components*/}

<Note>

每个包含 JSX 的文件都必须使用 `.tsx` 文件扩展名。这是 TypeScript 特有的扩展名，用于告诉 TypeScript 该文件包含 JSX。

</Note>

用 TypeScript 编写 React 与用 JavaScript 编写 React 非常相似。处理组件时，关键区别在于你可以为组件的 props 提供类型。这些类型可用于正确性检查，也可以在编辑器中提供内联文档。

以 [快速开始](/learn) 指南中的 [`MyButton` 组件](/learn#components) 为例，我们可以为按钮的 `title` 添加一个类型：

<Sandpack>

```tsx src/App.tsx active
function MyButton({ title }: { title: string }) {
  return (
    <button>{title}</button>
  );
}

export default function MyApp() {
  return (
    <div>
      <h1>欢迎来到我的应用</h1>
      <MyButton title="我是一个按钮" />
    </div>
  );
}
```

```js src/App.js hidden
import AppTSX from "./App.tsx";
export default App = AppTSX;
```
</Sandpack>

 <Note>

这些沙盒可以处理 TypeScript 代码，但不会运行类型检查器。这意味着你可以修改 TypeScript 沙盒来学习，但不会得到任何类型错误或警告。要进行类型检查，你可以使用 [TypeScript Playground](https://www.typescriptlang.org/play)，或使用功能更完整的在线沙盒。

</Note>

这种内联语法是为组件提供类型的最简单方式，不过一旦你开始描述几个字段，它就会变得笨拙。替代方案是使用 `interface` 或 `type` 来描述组件的 props：

<Sandpack>

```tsx src/App.tsx active
interface MyButtonProps {
  /** 要显示在按钮内部的文本 */
  title: string;
  /** 按钮是否可以交互 */
  disabled: boolean;
}

function MyButton({ title, disabled }: MyButtonProps) {
  return (
    <button disabled={disabled}>{title}</button>
  );
}

export default function MyApp() {
  return (
    <div>
      <h1>欢迎来到我的应用</h1>
      <MyButton title="我是一个禁用的按钮" disabled={true}/>
    </div>
  );
}
```

```js src/App.js hidden
import AppTSX from "./App.tsx";
export default App = AppTSX;
```

</Sandpack>

用于描述组件 props 的类型可以根据需要简单或复杂，但它们应该是通过 `type` 或 `interface` 描述的对象类型。你可以在 [对象类型](https://www.typescriptlang.org/docs/handbook/2/objects.html) 中了解 TypeScript 如何描述对象，不过你也可能会对使用 [联合类型](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#union-types) 来描述某个 prop 可以是几种不同类型之一，以及参考 [从类型创建类型](https://www.typescriptlang.org/docs/handbook/2/types-from-types.html) 指南来处理更高级的用例感兴趣。


## 示例 Hooks {/*example-hooks*/}

来自 `@types/react` 的类型定义包含内置 Hooks 的类型，因此你可以在组件中直接使用它们，而无需任何额外设置。它们会考虑到你在组件中编写的代码，因此很多时候你会得到 [推断类型](https://www.typescriptlang.org/docs/handbook/type-inference.html)，理想情况下也不需要处理提供这些类型的细枝末节。

不过，我们可以看几个为 Hooks 提供类型的示例。

### `useState` {/*typing-usestate*/}

[`useState` Hook](/reference/react/useState) 会重用作为初始状态传入的值，来确定该值应是什么类型。例如：

```ts
// 将类型推断为 "boolean"
const [enabled, setEnabled] = useState(false);
```

这会将 `boolean` 类型赋给 `enabled`，而 `setEnabled` 将是一个函数，接受一个 `boolean` 参数，或者接受一个返回 `boolean` 的函数。如果你想显式提供状态类型，可以通过在 `useState` 调用中提供类型参数来实现：

```ts
// 显式将类型设置为 "boolean"
const [enabled, setEnabled] = useState<boolean>(false);
```

在这个例子中这并不是很有用，但一个常见的你可能想提供类型的场景是当你有联合类型时。例如，这里的 `status` 可以是几种不同字符串之一：

```ts
type Status = "idle" | "loading" | "success" | "error";

const [status, setStatus] = useState<Status>("idle");
```

或者，正如[构建 state 的原则](/learn/choosing-the-state-structure#principles-for-structuring-state)所建议的，你可以将相关 state 组合成一个对象，并通过对象类型来描述不同可能性：

```ts
type RequestState =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success', data: any }
  | { status: 'error', error: Error };

const [requestState, setRequestState] = useState<RequestState>({ status: 'idle' });
```

### `useReducer` {/*typing-usereducer*/}

[`useReducer` Hook](/reference/react/useReducer) 是一个更复杂的 Hook，它接收一个 reducer 函数和一个初始状态。reducer 函数的类型会根据初始状态推断出来。你可以选择在 `useReducer` 调用中提供一个类型参数来为 state 提供类型，但通常更好的做法是直接在初始状态上设置类型：

<Sandpack>

```tsx src/App.tsx active
import {useReducer} from 'react';

interface State {
   count: number
};

type CounterAction =
  | { type: "reset" }
  | { type: "setCount"; value: State["count"] }

const initialState: State = { count: 0 };

function stateReducer(state: State, action: CounterAction): State {
  switch (action.type) {
    case "reset":
      return initialState;
    case "setCount":
      return { ...state, count: action.value };
    default:
      throw new Error("Unknown action");
  }
}

export default function App() {
  const [state, dispatch] = useReducer(stateReducer, initialState);

  const addFive = () => dispatch({ type: "setCount", value: state.count + 5 });
  const reset = () => dispatch({ type: "reset" });

  return (
    <div>
      <h1>欢迎来到我的计数器</h1>

      <p>计数：{state.count}</p>
      <button onClick={addFive}>加 5</button>
      <button onClick={reset}>重置</button>
    </div>
  );
}

```

```js src/App.js hidden
import AppTSX from "./App.tsx";
export default App = AppTSX;
```

</Sandpack>


我们在几个关键位置使用了 TypeScript：

 - `interface State` 描述了 reducer state 的形状。
 - `type CounterAction` 描述了可以分发给 reducer 的不同动作。
 - `const initialState: State` 为初始状态提供了类型，也提供了 `useReducer` 默认使用的类型。
 - `stateReducer(state: State, action: CounterAction): State` 为 reducer 函数的参数和返回值设置了类型。

除了在 `initialState` 上设置类型之外，更明确的替代方案是为 `useReducer` 提供一个类型参数：

```ts
import { stateReducer, State } from './your-reducer-implementation';

const initialState = { count: 0 };

export default function App() {
  const [state, dispatch] = useReducer<State>(stateReducer, initialState);
}
```

### `useContext` {/*typing-usecontext*/}

[`useContext` Hook](/reference/react/useContext) 是一种在不需要通过组件逐层传递 props 的情况下，将数据向下传递到组件树中的技术。它通过创建一个 provider 组件来使用，并且通常还会创建一个 Hook 来在子组件中消费该值。

context 提供的值的类型，会根据传入 `createContext` 调用的值进行推断：

<Sandpack>

```tsx src/App.tsx active
import { createContext, useContext, useState } from 'react';

type Theme = "light" | "dark" | "system";
const ThemeContext = createContext<Theme>("system");

const useGetTheme = () => useContext(ThemeContext);

export default function MyApp() {
  const [theme, setTheme] = useState<Theme>('light');

  return (
    <ThemeContext value={theme}>
      <MyComponent />
    </ThemeContext>
  )
}

function MyComponent() {
  const theme = useGetTheme();

  return (
    <div>
      <p>当前主题：{theme}</p>
    </div>
  )
}
```

```js src/App.js hidden
import AppTSX from "./App.tsx";
export default App = AppTSX;
```

</Sandpack>

当你有一个有意义的默认值时，这种技术就适用——但有时你并没有这样的默认值，在这种情况下，将 `null` 作为默认值会显得合理。然而，为了让类型系统理解你的代码，你需要在 `createContext` 上显式设置 `ContextShape | null`。

这会带来一个问题：在 context 消费者的类型中，你需要去掉 `| null`。我们的建议是让 Hook 在运行时检查它是否存在，如果不存在就抛出错误：

```js {5, 16-20}
import { createContext, useContext, useState, useMemo } from 'react';

// 这是一个更简单的示例，但你可以想象这里是一个更复杂的对象
type ComplexObject = {
  kind: string
};

// context 在类型中使用 `| null` 创建，以准确反映默认值。
const Context = createContext<ComplexObject | null>(null);

// 通过 Hook 中的检查，`| null` 会被移除。
const useGetComplexObject = () => {
  const object = useContext(Context);
  if (!object) { throw new Error("useGetComplexObject must be used within a Provider") }
  return object;
}

export default function MyApp() {
  const object = useMemo(() => ({ kind: "complex" }), []);

  return (
    <Context value={object}>
      <MyComponent />
    </Context>
  )
}

function MyComponent() {
  const object = useGetComplexObject();

  return (
    <div>
      <p>当前对象：{object.kind}</p>
    </div>
  )
}
```

### `useMemo` {/*typing-usememo*/}

<Note>

[React Compiler](/learn/react-compiler) 会自动对值和函数进行记忆化，减少手动调用 `useMemo` 的需要。你可以使用编译器自动处理记忆化。

</Note>

[`useMemo`](/reference/react/useMemo) Hooks 会通过函数调用创建/重新获取一个已记忆的值，并且只会在作为第二个参数传入的依赖项发生变化时重新执行该函数。调用该 Hook 的结果会根据第一个参数中的函数返回值进行推断。你也可以通过向该 Hook 提供类型参数来更明确地指定类型。

```ts
// visibleTodos 的类型会根据 filterTodos 的返回值推断出来
const visibleTodos = useMemo(() => filterTodos(todos, tab), [todos, tab]);
```


### `useCallback` {/*typing-usecallback*/}

<Note>

[React Compiler](/learn/react-compiler) 会自动对值和函数进行记忆化，减少手动调用 `useCallback` 的需要。你可以使用编译器自动处理记忆化。

</Note>

[`useCallback`](/reference/react/useCallback) 会提供一个函数的稳定引用，只要传入第二个参数的依赖项保持不变即可。与 `useMemo` 类似，函数的类型会根据第一个参数中的函数返回值进行推断，你也可以通过向该 Hook 提供类型参数来更明确地指定类型。


```ts
const handleClick = useCallback(() => {
  // ...
}, [todos]);
```

在 TypeScript 严格模式下使用 `useCallback` 时，需要为回调参数添加类型。这是因为回调的类型是根据函数返回值推断出来的，如果没有参数，类型就无法被完全理解。

根据你的代码风格偏好，你可以使用 React 类型中的 `*EventHandler` 函数，在定义回调的同时为事件处理器提供类型：

```ts
import { useState, useCallback } from 'react';

export default function Form() {
  const [value, setValue] = useState("Change me");

  const handleChange = useCallback<React.ChangeEventHandler<HTMLInputElement>>((event) => {
    setValue(event.currentTarget.value);
  }, [setValue])

  return (
    <>
      <input value={value} onChange={handleChange} />
      <p>值：{value}</p>
    </>
  );
}
```

## 有用的类型 {/*useful-types*/}

`@types/react` 包中提供了一组相当广泛的类型，当你对 React 和 TypeScript 如何交互感到熟悉之后，值得一读。你可以在 [DefinitelyTyped 中 React 的文件夹](https://github.com/DefinitelyTyped/DefinitelyTyped/blob/master/types/react/index.d.ts) 里找到它们。这里我们将介绍一些更常见的类型。

### DOM 事件 {/*typing-dom-events*/}

在 React 中处理 DOM 事件时，事件的类型通常可以从事件处理函数中推断出来。然而，当你想把一个函数提取出来并传给事件处理函数时，你需要显式地设置事件的类型。

<Sandpack>

```tsx src/App.tsx active
import { useState } from 'react';

export default function Form() {
  const [value, setValue] = useState("Change me");

  function handleChange(event: React.ChangeEvent<HTMLInputElement>) {
    setValue(event.currentTarget.value);
  }

  return (
    <>
      <input value={value} onChange={handleChange} />
      <p>Value: {value}</p>
    </>
  );
}
```

```js src/App.js hidden
import AppTSX from "./App.tsx";
export default App = AppTSX;
```

</Sandpack>

React 类型中提供了许多事件类型 - 完整列表可以在 [这里](https://github.com/DefinitelyTyped/DefinitelyTyped/blob/b580df54c0819ec9df62b0835a315dd48b8594a9/types/react/index.d.ts#L1247C1-L1373) 找到，它基于 [DOM 中最常见的事件](https://developer.mozilla.org/en-US/docs/Web/Events)。

在确定你要查找的类型时，你可以先查看你正在使用的事件处理函数的悬停提示信息，其中会显示事件的类型。

如果你需要使用此列表中未包含的事件，你可以使用 `React.SyntheticEvent` 类型，它是所有事件的基础类型。

### 子元素 {/*typing-children*/}

描述组件的子元素有两种常见方式。第一种是使用 `React.ReactNode` 类型，它是所有可以作为 JSX 子元素传入的类型的联合类型：

```ts
interface ModalRendererProps {
  title: string;
  children: React.ReactNode;
}
```

这是对子元素非常宽泛的定义。第二种是使用 `React.ReactElement` 类型，它只表示 JSX 元素，而不包括字符串或数字等 JavaScript 基础类型：

```ts
interface ModalRendererProps {
  title: string;
  children: React.ReactElement;
}
```

请注意，你不能使用 TypeScript 来描述子元素必须是某种特定类型的 JSX 元素，因此你不能使用类型系统来描述一个只接受 `<li>` 子元素的组件。

你可以在 [这个 TypeScript playground](https://www.typescriptlang.org/play?#code/JYWwDg9gTgLgBAJQKYEMDG8BmUIjgIilQ3wChSB6CxYmAOmXRgDkIATJOdNJMGAZzgwAFpxAR+8YADswAVwGkZMJFEzpOjDKw4AFHGEEBvUnDhphwADZsi0gFw0mDWjqQBuUgF9yaCNMlENzgAXjgACjADfkctFnYkfQhDAEpQgD44AB42YAA3dKMo5P46C2tbJGkvLIpcgt9-QLi3AEEwMFCItJDMrPTTbIQ3dKywdIB5aU4kKyQQKpha8drhhIGzLLWODbNs3b3s8YAxKBQAcwXpAThMaGWDvbH0gFloGbmrgQfBzYpd1YjQZbEYARkB6zMwO2SHSAAlZlYIBCdtCRkZpHIrFYahQYQD8UYYFA5EhcfjyGYqHAXnJAsIUHlOOUbHYhMIIHJzsI0Qk4P9SLUBuRqXEXEwAKKfRZcNA8PiCfxWACecAAUgBlAAacFm80W-CU11U6h4TgwUv11yShjgJjMLMqDnN9Dilq+nh8pD8AXgCHdMrCkWisVoAet0R6fXqhWKhjKllZVVxMcavpd4Zg7U6Qaj+2hmdG4zeRF10uu-Aeq0LBfLMEe-V+T2L7zLVu+FBWLdLeq+lc7DYFf39deFVOotMCACNOCh1dq219a+30uC8YWoZsRyuEdjkevR8uvoVMdjyTWt4WiSSydXD4NqZP4AymeZE072ZzuUeZQKheQgA) 中使用类型检查器查看 `React.ReactNode` 和 `React.ReactElement` 的示例。

### 样式属性 {/*typing-style-props*/}

在 React 中使用内联样式时，你可以使用 `React.CSSProperties` 来描述传递给 `style` 属性的对象。这个类型是所有可能的 CSS 属性的联合类型，是确保你向 `style` 属性传递了有效 CSS 属性的好方法，也能在编辑器中获得自动补全。

```ts
interface MyComponentProps {
  style: React.CSSProperties;
}
```

## 进一步学习 {/*further-learning*/}

本指南已经介绍了在 React 中使用 TypeScript 的基础知识，但还有很多内容值得学习。
文档中的各个 API 页面可能包含更深入的说明，介绍如何将它们与 TypeScript 一起使用。

我们推荐以下资源：

 - [TypeScript 手册](https://www.typescriptlang.org/docs/handbook/) 是 TypeScript 的官方文档，涵盖了大多数关键语言特性。

 - [TypeScript 发布说明](https://devblogs.microsoft.com/typescript/) 会深入介绍新特性。

 - [React TypeScript Cheatsheet](https://react-typescript-cheatsheet.netlify.app/) 是由社区维护的 React + TypeScript 速查表，涵盖了许多有用的边界情况，内容也比本文档更广泛。

 - [TypeScript 社区 Discord](https://discord.com/invite/typescript) 是提问并获得 TypeScript 和 React 问题帮助的绝佳场所。
