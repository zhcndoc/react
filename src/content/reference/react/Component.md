---
title: Component
---

<Pitfall>

我们建议将组件定义为函数而不是类。[查看如何迁移。](#alternatives)

</Pitfall>

<Intro>

`Component` 是 React 组件的基类，这些组件是作为 [JavaScript 类。](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes) 定义的。React 仍然支持类组件，但我们不建议在新代码中使用它们。

```js
class Greeting extends Component {
  render() {
    return <h1>Hello, {this.props.name}!</h1>;
  }
}
```

</Intro>

<InlineToc />

---

## 参考 {/*reference*/}

### `Component` {/*component*/}

要将 React 组件定义为类，请扩展内置的 `Component` 类并定义一个 [`render` 方法：](#render)

```js
import { Component } from 'react';

class Greeting extends Component {
  render() {
    return <h1>Hello, {this.props.name}!</h1>;
  }
}
```

只需要 `render` 方法，其他方法都是可选的。

[查看更多示例。](#usage)

---

### `context` {/*context*/}

类组件的 [context](/learn/passing-data-deeply-with-context) 可通过 `this.context` 访问。只有在你使用 [`static contextType`](#static-contexttype) 指定了*要接收哪个* context 时，它才可用。

类组件一次只能读取一个 context。

```js {2,5}
class Button extends Component {
  static contextType = ThemeContext;

  render() {
    const theme = this.context;
    const className = 'button-' + theme;
    return (
      <button className={className}>
        {this.props.children}
      </button>
    );
  }
}

```

<Note>

在类组件中读取 `this.context` 等同于在函数组件中使用 [`useContext`](/reference/react/useContext)。

[查看如何迁移。](#migrating-a-component-with-context-from-a-class-to-a-function)

</Note>

---

### `props` {/*props*/}

传递给类组件的 props 可通过 `this.props` 访问。

```js {3}
class Greeting extends Component {
  render() {
    return <h1>Hello, {this.props.name}!</h1>;
  }
}

<Greeting name="Taylor" />
```

<Note>

在类组件中读取 `this.props` 等同于在函数组件中[声明 props](/learn/passing-props-to-a-component#step-2-read-props-inside-the-child-component)。

[查看如何迁移。](#migrating-a-simple-component-from-a-class-to-a-function)

</Note>

---

### `state` {/*state*/}

类组件的 state 可通过 `this.state` 访问。`state` 字段必须是一个对象。不要直接修改 state。如果你想更改 state，请使用新的 state 调用 `setState`。

```js {2-4,7-9,18}
class Counter extends Component {
  state = {
    age: 42,
  };

  handleAgeChange = () => {
    this.setState({
      age: this.state.age + 1
    });
  };

  render() {
    return (
      <>
        <button onClick={this.handleAgeChange}>
        增加年龄
        </button>
        <p>你已经 {this.state.age} 岁了。</p>
      </>
    );
  }
}
```

<Note>

在类组件中定义 `state` 等同于在函数组件中调用 [`useState`](/reference/react/useState)。

[查看如何迁移。](#migrating-a-component-with-state-from-a-class-to-a-function)

</Note>

---

### `constructor(props)` {/*constructor*/}

[构造函数](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/constructor) 会在类组件*挂载*（被添加到屏幕上）之前运行。通常，构造函数在 React 中只用于两个目的。它让你可以声明 state，并将类方法 [绑定](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_objects/Function/bind) 到类实例上：

```js {2-6}
class Counter extends Component {
  constructor(props) {
    super(props);
    this.state = { counter: 0 };
    this.handleClick = this.handleClick.bind(this);
  }

  handleClick() {
    // ...
  }
```

如果你使用现代 JavaScript 语法，通常不需要构造函数。相反，你可以使用 [公共类字段语法](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/Public_class_fields) 重写上面的代码，这种语法既被现代浏览器支持，也被像 [Babel:](https://babeljs.io/) 这样的工具支持

```js {2,4}
class Counter extends Component {
  state = { counter: 0 };

  handleClick = () => {
    // ...
  }
```

构造函数不应包含任何副作用或订阅。

#### 参数 {/*constructor-parameters*/}

* `props`: 组件的初始 props。

#### 返回值 {/*constructor-returns*/}

`constructor` 不应返回任何内容。

#### 注意事项 {/*constructor-caveats*/}

* 不要在构造函数中运行任何副作用或订阅。相反，请使用 [`componentDidMount`](#componentdidmount) 来处理这些事情。

* 在构造函数内部，你需要在任何其他语句之前调用 `super(props)`。如果你不这样做，在构造函数执行期间 `this.props` 将是 `undefined`，这可能会让人困惑并导致 bug。

* 构造函数是唯一可以直接给 [`this.state`](#state) 赋值的地方。在所有其他方法中，你需要改用 [`this.setState()`](#setstate)。不要在构造函数中调用 `setState`。

* 当你使用[服务端渲染,](/reference/react-dom/server)时，构造函数也会在服务器上运行，随后是 [`render`](#render) 方法。不过，像 `componentDidMount` 或 `componentWillUnmount` 这样的生命周期方法不会在服务器上运行。

* 当启用 [严格模式](/reference/react/StrictMode) 时，React 会在开发环境中调用两次 `constructor`，然后丢弃其中一个实例。这可以帮助你发现那些意外的副作用，这些副作用需要从 `constructor` 中移出。

<Note>

函数组件中没有与 `constructor` 完全对应的东西。要在函数组件中声明 state，请调用 [`useState`.](/reference/react/useState) 为了避免重复计算初始 state，[向 `useState` 传入一个函数。](/reference/react/useState#avoiding-recreating-the-initial-state)

</Note>

---

### `componentDidCatch(error, info)` {/*componentdidcatch*/}

如果你定义了 `componentDidCatch`，当某个子组件（包括更远层级的子组件）在渲染过程中抛出错误时，React 会调用它。这让你可以在生产环境中将该错误记录到错误报告服务。

通常，它会与 [`static getDerivedStateFromError`](#static-getderivedstatefromerror) 一起使用，后者让你可以在发生错误时更新 state，并向用户显示错误信息。具有这些方法的组件称为 *错误边界*。

[查看示例。](#catching-rendering-errors-with-an-error-boundary)

#### 参数 {/*componentdidcatch-parameters*/}

* `error`: 抛出的错误。实际上，它通常是 [`Error`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Error) 的一个实例，但这并不保证，因为 JavaScript 允许 [`throw`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/throw) 任意值，包括字符串甚至 `null`。

* `info`: 一个包含有关错误的附加信息的对象。其 `componentStack` 字段包含一个堆栈跟踪，其中包含抛出错误的组件，以及所有父组件的名称和源码位置。在生产环境中，组件名称会被压缩。若你设置了生产环境错误报告，你可以像处理普通 JavaScript 错误堆栈一样，使用 sourcemaps 解析组件堆栈。

#### 返回值 {/*componentdidcatch-returns*/}

`componentDidCatch` 不应返回任何内容。

#### 注意事项 {/*componentdidcatch-caveats*/}

* 过去，通常会在 `componentDidCatch` 内调用 `setState` 来更新 UI 并显示回退错误信息。现在这已被弃用，建议改为定义 [`static getDerivedStateFromError`.](#static-getderivedstatefromerror)

* React 的生产版和开发版在 `componentDidCatch` 处理错误的方式上略有不同。在开发环境中，错误会继续冒泡到 `window`，这意味着任何 `window.onerror` 或 `window.addEventListener('error', callback)` 都会拦截到被 `componentDidCatch` 捕获的错误。在生产环境中，错误则不会继续冒泡，这意味着任何祖先错误处理器只会收到未被 `componentDidCatch` 明确捕获的错误。

<Note>

目前，函数组件中还没有 `componentDidCatch` 的直接对应项。如果你想避免创建类组件，可以像上面那样编写一个 `ErrorBoundary` 组件，并在整个应用中使用它。或者，你也可以使用 [`react-error-boundary`](https://github.com/bvaughn/react-error-boundary) 包，它会帮你完成这些工作。

</Note>

---

### `componentDidMount()` {/*componentdidmount*/}

如果你定义了 `componentDidMount` 方法，当你的组件被添加到屏幕上（*挂载*）时，React 会调用它。这通常是开始数据获取、设置订阅或操作 DOM 节点的地方。

如果你实现了 `componentDidMount`，通常还需要实现其他生命周期方法以避免 bug。例如，如果 `componentDidMount` 读取某些 state 或 props，你还必须实现 [`componentDidUpdate`](#componentdidupdate) 来处理它们的变化，以及实现 [`componentWillUnmount`](#componentwillunmount) 来清理 `componentDidMount` 所做的任何事情。

```js {6-8}
class ChatRoom extends Component {
  state = {
    serverUrl: 'https://localhost:1234'
  };

  componentDidMount() {
    this.setupConnection();
  }

  componentDidUpdate(prevProps, prevState) {
    if (
      this.props.roomId !== prevProps.roomId ||
      this.state.serverUrl !== prevState.serverUrl
    ) {
      this.destroyConnection();
      this.setupConnection();
    }
  }

  componentWillUnmount() {
    this.destroyConnection();
  }

  // ...
}
```

[查看更多示例。](#adding-lifecycle-methods-to-a-class-component)

#### 参数 {/*componentdidmount-parameters*/}

`componentDidMount` 不接受任何参数。

#### 返回值 {/*componentdidmount-returns*/}

`componentDidMount` 不应返回任何内容。

#### 注意事项 {/*componentdidmount-caveats*/}

- 当启用 [严格模式](/reference/react/StrictMode) 时，在开发环境中 React 会先调用 `componentDidMount`，然后立即调用 [`componentWillUnmount`,](#componentwillunmount) 接着再调用一次 `componentDidMount`。这可以帮助你发现是否忘记实现 `componentWillUnmount`，或者其逻辑是否没有完全“镜像” `componentDidMount` 的行为。

- 尽管你可以立即在 `componentDidMount` 中调用 [`setState`](#setstate)，但能避免时最好避免。这样会触发一次额外渲染，但它会发生在浏览器更新屏幕之前。这保证了即使在这种情况下 [`render`](#render) 会被调用两次，用户也不会看到中间状态。请谨慎使用这种模式，因为它经常会导致性能问题。在大多数情况下，你应该能够直接在 [`constructor`](#constructor) 中设置初始 state。不过，在某些场景下这可能是必要的，比如模态框和工具提示：当你需要在渲染依赖其尺寸或位置的内容之前先测量一个 DOM 节点时。

<Note>

对于许多使用场景，在类组件中将 `componentDidMount`、`componentDidUpdate` 和 `componentWillUnmount` 一起定义，等同于在函数组件中调用 [`useEffect`](/reference/react/useEffect)。在少数需要代码在浏览器绘制前运行的情况下，[`useLayoutEffect`](/reference/react/useLayoutEffect) 更接近。

[查看如何迁移。](#migrating-a-component-with-lifecycle-methods-from-a-class-to-a-function)

</Note>

---

### `componentDidUpdate(prevProps, prevState, snapshot?)` {/*componentdidupdate*/}

如果你定义了 `componentDidUpdate` 方法，当你的组件使用更新后的 props 或 state 重新渲染后，React 会立即调用它。这个方法不会在初始渲染时调用。

你可以用它在更新后操作 DOM。这也是执行网络请求的常见位置，只要你比较当前 props 和之前的 props 即可（例如，如果 props 没有变化，可能就不需要网络请求）。通常，你会把它与 [`componentDidMount`](#componentdidmount) 和 [`componentWillUnmount`:](#componentwillunmount) 一起使用

```js {10-18}
class ChatRoom extends Component {
  state = {
    serverUrl: 'https://localhost:1234'
  };

  componentDidMount() {
    this.setupConnection();
  }

  componentDidUpdate(prevProps, prevState) {
    if (
      this.props.roomId !== prevProps.roomId ||
      this.state.serverUrl !== prevState.serverUrl
    ) {
      this.destroyConnection();
      this.setupConnection();
    }
  }

  componentWillUnmount() {
    this.destroyConnection();
  }

  // ...
}
```

[查看更多示例。](#adding-lifecycle-methods-to-a-class-component)


#### 参数 {/*componentdidupdate-parameters*/}

* `prevProps`: 更新前的 props。将 `prevProps` 与 [`this.props`](#props) 比较以确定哪些内容发生了变化。

* `prevState`: 更新前的 state。将 `prevState` 与 [`this.state`](#state) 比较以确定哪些内容发生了变化。

* `snapshot`: 如果你实现了 [`getSnapshotBeforeUpdate`](#getsnapshotbeforeupdate)，`snapshot` 将包含该方法返回的值。否则，它将是 `undefined`。

#### 返回值 {/*componentdidupdate-returns*/}

`componentDidUpdate` 不应返回任何内容。

#### 注意事项 {/*componentdidupdate-caveats*/}

- 如果定义了 [`shouldComponentUpdate`](#shouldcomponentupdate) 并返回 `false`，则不会调用 `componentDidUpdate`。

- `componentDidUpdate` 内部的逻辑通常应包裹在条件判断中，将 `this.props` 与 `prevProps`、`this.state` 与 `prevState` 进行比较。否则，可能会导致无限循环。

- 尽管你可以立即在 `componentDidUpdate` 中调用 [`setState`](#setstate)，但能避免时最好避免。这样会触发一次额外渲染，但它会发生在浏览器更新屏幕之前。这保证了即使在这种情况下 [`render`](#render) 会被调用两次，用户也不会看到中间状态。这种模式经常会引发性能问题，但在少数场景下可能是必要的，比如模态框和工具提示：当你需要在渲染依赖其尺寸或位置的内容之前先测量一个 DOM 节点时。

<Note>

对于许多使用场景，在类组件中将 `componentDidMount`、`componentDidUpdate` 和 `componentWillUnmount` 一起定义，等同于在函数组件中调用 [`useEffect`](/reference/react/useEffect)。在少数需要代码在浏览器绘制前运行的情况下，[`useLayoutEffect`](/reference/react/useLayoutEffect) 更接近。

[查看如何迁移。](#migrating-a-component-with-lifecycle-methods-from-a-class-to-a-function)

</Note>
---

### `componentWillMount()` {/*componentwillmount*/}

<Deprecated>

这个 API 已从 `componentWillMount` 重命名为 [`UNSAFE_componentWillMount`.](#unsafe_componentwillmount) 旧名称已被弃用。在未来的 React 主要版本中，只有新名称可用。

运行 [`rename-unsafe-lifecycles` codemod](https://github.com/reactjs/react-codemod#rename-unsafe-lifecycles) 可自动更新你的组件。

</Deprecated>

---

### `componentWillReceiveProps(nextProps)` {/*componentwillreceiveprops*/}

<Deprecated>

这个 API 已从 `componentWillReceiveProps` 重命名为 [`UNSAFE_componentWillReceiveProps`.](#unsafe_componentwillreceiveprops) 旧名称已被弃用。在未来的 React 主要版本中，只有新名称可用。

运行 [`rename-unsafe-lifecycles` codemod](https://github.com/reactjs/react-codemod#rename-unsafe-lifecycles) 可自动更新你的组件。

</Deprecated>

---

### `componentWillUpdate(nextProps, nextState)` {/*componentwillupdate*/}

<Deprecated>

这个 API 已从 `componentWillUpdate` 重命名为 [`UNSAFE_componentWillUpdate`.](#unsafe_componentwillupdate) 旧名称已被弃用。在未来的 React 主要版本中，只有新名称可用。

运行 [`rename-unsafe-lifecycles` codemod](https://github.com/reactjs/react-codemod#rename-unsafe-lifecycles) 可自动更新你的组件。

</Deprecated>

---

### `componentWillUnmount()` {/*componentwillunmount*/}

如果你定义了 `componentWillUnmount` 方法，当你的组件从屏幕上移除（*卸载*）之前，React 会调用它。这通常是取消数据获取或移除订阅的地方。

`componentWillUnmount` 内部的逻辑应当与 [`componentDidMount`.](#componentdidmount) 内部的逻辑“镜像”对应。例如，如果 `componentDidMount` 设置了一个订阅，那么 `componentWillUnmount` 应该清理该订阅。如果你的 `componentWillUnmount` 中的清理逻辑会读取某些 props 或 state，那么你通常还需要实现 [`componentDidUpdate`](#componentdidupdate)，以清理与旧 props 和 state 对应的资源（如订阅）。

```js {20-22}
class ChatRoom extends Component {
  state = {
    serverUrl: 'https://localhost:1234'
  };

  componentDidMount() {
    this.setupConnection();
  }

  componentDidUpdate(prevProps, prevState) {
    if (
      this.props.roomId !== prevProps.roomId ||
      this.state.serverUrl !== prevState.serverUrl
    ) {
      this.destroyConnection();
      this.setupConnection();
    }
  }

  componentWillUnmount() {
    this.destroyConnection();
  }

  // ...
}
```

[查看更多示例。](#adding-lifecycle-methods-to-a-class-component)

#### 参数 {/*componentwillunmount-parameters*/}

`componentWillUnmount` 不接受任何参数。

#### 返回值 {/*componentwillunmount-returns*/}

`componentWillUnmount` 不应返回任何内容。

#### 注意事项 {/*componentwillunmount-caveats*/}

- 当启用 [严格模式](/reference/react/StrictMode) 时，在开发环境中 React 会先调用 [`componentDidMount`,](#componentdidmount) 然后立即调用 `componentWillUnmount`，接着再调用一次 `componentDidMount`。这可以帮助你发现是否忘记实现 `componentWillUnmount`，或者其逻辑是否没有完全“镜像” `componentDidMount` 的行为。

<Note>

对于许多使用场景，在类组件中将 `componentDidMount`、`componentDidUpdate` 和 `componentWillUnmount` 一起定义，等同于在函数组件中调用 [`useEffect`](/reference/react/useEffect)。在少数需要代码在浏览器绘制前运行的情况下，[`useLayoutEffect`](/reference/react/useLayoutEffect) 更接近。

[查看如何迁移。](#migrating-a-component-with-lifecycle-methods-from-a-class-to-a-function)

</Note>

---

### `forceUpdate(callback?)` {/*forceupdate*/}

强制组件重新渲染。

通常这并不必要。如果你的组件的 [`render`](#render) 方法只读取 [`this.props`](#props)、[`this.state`](#state) 或 [`this.context`](#context)，那么当你在组件内部或其某个父组件中调用 [`setState`](#setstate) 时，它会自动重新渲染。然而，如果你的组件的 `render` 方法直接读取外部数据源，那么当该数据源变化时，你必须告诉 React 更新用户界面。这正是 `forceUpdate` 允许你做的事情。

尽量避免使用 `forceUpdate`，并且只在 `render` 中读取 `this.props` 和 `this.state`。

#### 参数 {/*forceupdate-parameters*/}

* **可选** `callback` 如果指定，React 会在更新提交后调用你提供的 `callback`。

#### 返回值 {/*forceupdate-returns*/}

`forceUpdate` 不返回任何内容。

#### 注意事项 {/*forceupdate-caveats*/}

- 如果你调用 `forceUpdate`，React 会重新渲染，但不会调用 [`shouldComponentUpdate`.](#shouldcomponentupdate)

<Note>

在函数组件中，使用 `forceUpdate` 读取外部数据源并在其变化时强制类组件重新渲染，已被 [`useSyncExternalStore`](/reference/react/useSyncExternalStore) 所取代。

</Note>

---

### `getSnapshotBeforeUpdate(prevProps, prevState)` {/*getsnapshotbeforeupdate*/}

如果你实现了 `getSnapshotBeforeUpdate`，React 会在更新 DOM 之前立即调用它。它使你的组件能够在 DOM 可能被更改之前，从中捕获一些信息（例如滚动位置）。此生命周期方法返回的任何值都会作为参数传递给 [`componentDidUpdate`.](#componentdidupdate)

例如，你可以在需要在更新期间保持滚动位置的聊天线程之类的 UI 中使用它：

```js {7-15,17}
class ScrollingList extends React.Component {
  constructor(props) {
    super(props);
    this.listRef = React.createRef();
  }

  getSnapshotBeforeUpdate(prevProps, prevState) {
    // 我们正在向列表中添加新项目吗？
    // 捕获滚动位置，以便稍后调整滚动。
    if (prevProps.list.length < this.props.list.length) {
      const list = this.listRef.current;
      return list.scrollHeight - list.scrollTop;
    }
    return null;
  }

  componentDidUpdate(prevProps, prevState, snapshot) {
    // 如果我们有快照值，说明我们刚刚添加了新项目。
    // 调整滚动位置，这样这些新项目就不会把旧项目顶出视图。
    // （这里的 snapshot 是 getSnapshotBeforeUpdate 返回的值）
    if (snapshot !== null) {
      const list = this.listRef.current;
      list.scrollTop = list.scrollHeight - snapshot;
    }
  }

  render() {
    return (
      <div ref={this.listRef}>{/* ...内容... */}</div>
    );
  }
}
```

在上面的示例中，必须直接在 `getSnapshotBeforeUpdate` 中读取 `scrollHeight` 属性。在 [`render`](#render)、[`UNSAFE_componentWillReceiveProps`](#unsafe_componentwillreceiveprops) 或 [`UNSAFE_componentWillUpdate`](#unsafe_componentwillupdate) 中读取它都不安全，因为这些方法被调用到 React 更新 DOM 之间可能存在时间间隔。

#### 参数 {/*getsnapshotbeforeupdate-parameters*/}

* `prevProps`: 更新前的 props。将 `prevProps` 与 [`this.props`](#props) 比较以确定哪些内容发生了变化。

* `prevState`: 更新前的 state。将 `prevState` 与 [`this.state`](#state) 比较以确定哪些内容发生了变化。

#### 返回值 {/*getsnapshotbeforeupdate-returns*/}

你应当返回任意类型的快照值，或者 `null`。你返回的值将作为第三个参数传递给 [`componentDidUpdate`.](#componentdidupdate)

#### 注意事项 {/*getsnapshotbeforeupdate-caveats*/}

- 如果定义了 [`shouldComponentUpdate`](#shouldcomponentupdate) 并返回 `false`，则不会调用 `getSnapshotBeforeUpdate`。

<Note>

目前，函数组件还没有与 `getSnapshotBeforeUpdate` 对应的等价方法。这个使用场景非常少见，但如果你确实需要它，目前只能编写类组件。

</Note>

---

### `render()` {/*render*/}

`render` 方法是类组件中唯一必需的方法。

`render` 方法应指定你希望在屏幕上显示的内容，例如：

```js {4-6}
import { Component } from 'react';

class Greeting extends Component {
  render() {
    return <h1>Hello, {this.props.name}!</h1>;
  }
}
```

React 可能会在任何时刻调用 `render`，因此你不应假设它会在某个特定时间运行。通常，`render` 方法应返回一段 [JSX](/learn/writing-markup-with-jsx)，但也支持少数[其他返回类型](#render-returns)（如字符串）。为了计算返回的 JSX，`render` 方法可以读取 [`this.props`](#props)、[`this.state`](#state) 和 [`this.context`](#context)。

你应该将 `render` 方法写成纯函数，这意味着如果 props、state 和 context 相同，它应返回相同结果。它也不应包含副作用（如设置订阅）或与浏览器 API 交互。副作用应当发生在事件处理函数中，或者像 [`componentDidMount`](#componentdidmount) 这样的生命周期方法中。

#### 参数 {/*render-parameters*/}

`render` 不接受任何参数。

#### 返回值 {/*render-returns*/}

`render` 可以返回任何有效的 React 节点。这包括 React 元素，例如 `<div />`、字符串、数字、[portal](/reference/react-dom/createPortal)、空节点（`null`、`undefined`、`true` 和 `false`），以及 React 节点数组。

#### 注意事项 {/*render-caveats*/}

- `render` 应写成 props、state 和 context 的纯函数。它不应有副作用。

- 如果定义了 [`shouldComponentUpdate`](#shouldcomponentupdate) 并返回 `false`，则不会调用 `render`。

- 当启用 [严格模式](/reference/react/StrictMode) 时，React 会在开发环境中调用两次 `render`，然后丢弃其中一个结果。这可以帮助你发现那些意外的副作用，这些副作用需要从 `render` 方法中移出。

- `render` 调用与随后 `componentDidMount` 或 `componentDidUpdate` 调用之间并不是一一对应的。React 可能会丢弃部分 `render` 调用的结果，如果这样更有利的话。

---

### `setState(nextState, callback?)` {/*setstate*/}

调用 `setState` 来更新你的 React 组件的 state。

```js {8-10}
class Form extends Component {
  state = {
    name: 'Taylor',
  };

  handleNameChange = (e) => {
    const newName = e.target.value;
    this.setState({
      name: newName
    });
  }

  render() {
    return (
      <>
        <input value={this.state.name} onChange={this.handleNameChange} />
        <p>Hello, {this.state.name}.</p>
      </>
    );
  }
}
```

`setState` 会将更改入队到组件 state 中。它告诉 React 该组件及其子组件需要使用新 state 重新渲染。这是你响应交互来更新用户界面的主要方式。

<Pitfall>

调用 `setState` **不会**改变已经在执行中的代码里的当前 state：

```js {6}
function handleClick() {
  console.log(this.state.name); // "Taylor"
  this.setState({
    name: 'Robin'
  });
  console.log(this.state.name); // 仍然是 "Taylor"！
}
```

它只会影响从*下一次*渲染开始 `this.state` 返回的内容。

</Pitfall>

你也可以向 `setState` 传入一个函数。它让你可以根据上一个 state 来更新 state：

```js {2-6}
  handleIncreaseAge = () => {
    this.setState(prevState => {
      return {
        age: prevState.age + 1
      };
    });
  }
```

你不一定非要这样做，但如果你想在同一个事件中多次更新 state，这会很有用。

#### 参数 {/*setstate-parameters*/}

* `nextState`: 对象或函数均可。
  * 如果你传入一个对象作为 `nextState`，它会被浅合并到 `this.state` 中。
  * 如果你传入一个函数作为 `nextState`，它会被视为一个 _更新函数_。它必须是纯函数，应接收待处理的 state 和 props 作为参数，并返回一个对象，该对象会被浅合并到 `this.state` 中。React 会将你的更新函数放入队列并重新渲染组件。在下一次渲染期间，React 会通过将队列中的所有更新函数应用到上一个 state 来计算下一个 state。

* **可选** `callback`: 如果指定，React 会在更新提交后调用你提供的 `callback`。

#### 返回值 {/*setstate-returns*/}

`setState` 不返回任何内容。

#### 注意事项 {/*setstate-caveats*/}

- 可以将 `setState` 看作是一个*请求*，而不是立即更新组件的命令。当多个组件响应某个事件更新它们的 state 时，React 会批处理这些更新，并在事件结束时一次性重新渲染它们。在极少数需要强制某个 state 更新同步生效的情况下，你可以将其包裹在 [`flushSync`,](/reference/react-dom/flushSync) 中，但这可能会损害性能。

- `setState` 不会立即更新 `this.state`。因此，在调用 `setState` 之后立刻读取 `this.state` 可能是个坑。相反，应使用 [`componentDidUpdate`](#componentdidupdate) 或 `setState` 的 `callback` 参数，它们都保证会在更新应用后触发。如果你需要基于上一个 state 设置 state，可以像上面描述的那样向 `nextState` 传入一个函数。

<Note>

在类组件中调用 `setState` 类似于在函数组件中调用 [`set` 函数](/reference/react/useState#setstate)。

[查看如何迁移。](#migrating-a-component-with-state-from-a-class-to-a-function)

</Note>

---

### `shouldComponentUpdate(nextProps, nextState, nextContext)` {/*shouldcomponentupdate*/}

如果你定义了 `shouldComponentUpdate`，React 会调用它来判断是否可以跳过一次重新渲染。

如果你确信要手动编写它，可以将 `this.props` 与 `nextProps`、`this.state` 与 `nextState` 进行比较，并返回 `false` 来告诉 React 可以跳过更新。

```js {6-18}
class Rectangle extends Component {
  state = {
    isHovered: false
  };

  shouldComponentUpdate(nextProps, nextState) {
    if (
      nextProps.position.x === this.props.position.x &&
      nextProps.position.y === this.props.position.y &&
      nextProps.size.width === this.props.size.width &&
      nextProps.size.height === this.props.size.height &&
      nextState.isHovered === this.state.isHovered
    ) {
      // 什么都没有变化，因此不需要重新渲染
      return false;
    }
    return true;
  }

  // ...
}

```

当接收到新的 props 或 state 时，React 会在渲染前调用 `shouldComponentUpdate`。默认值为 `true`。该方法不会在初始渲染时调用，也不会在使用 [`forceUpdate`](#forceupdate) 时调用。

#### 参数 {/*shouldcomponentupdate-parameters*/}

- `nextProps`: 组件即将使用来渲染的下一个 props。将 `nextProps` 与 [`this.props`](#props) 比较以确定哪些内容发生了变化。
- `nextState`: 组件即将使用来渲染的下一个 state。将 `nextState` 与 [`this.state`](#props) 比较以确定哪些内容发生了变化。
- `nextContext`: 组件即将使用来渲染的下一个 context。将 `nextContext` 与 [`this.context`](#context) 比较以确定哪些内容发生了变化。只有在你指定了 [`static contextType`](#static-contexttype) 时才可用。

#### 返回值 {/*shouldcomponentupdate-returns*/}

如果你希望组件重新渲染，就返回 `true`。这是默认行为。

返回 `false` 可告诉 React 可以跳过重新渲染。

#### 注意事项 {/*shouldcomponentupdate-caveats*/}

- 这个方法*只*存在于性能优化场景。如果没有它你的组件会出错，那首先应该修复那个问题。

- 考虑使用 [`PureComponent`](/reference/react/PureComponent) 来代替手写 `shouldComponentUpdate`。`PureComponent` 会对 props 和 state 进行浅比较，减少你跳过必要更新的风险。

- 我们不建议在 `shouldComponentUpdate` 中进行深度相等比较或使用 `JSON.stringify`。这会使性能变得不可预测，并依赖于每个 prop 和 state 的数据结构。最好的情况是，你有可能给应用引入持续数秒的卡顿；最坏的情况是，你可能让应用崩溃。

- 返回 `false` 并不会阻止子组件在*它们自己的* state 变化时重新渲染。

- 返回 `false` 并不*保证*组件不会重新渲染。React 会将返回值作为提示，但如果出于其他原因重新渲染更合理，它仍可能选择重新渲染你的组件。

<Note>

使用 `shouldComponentUpdate` 优化类组件，类似于使用 [`memo`.](/reference/react/memo) 优化函数组件。函数组件还提供了更细粒度的优化方式，比如 [`useMemo`.](/reference/react/useMemo)

</Note>

---

### `UNSAFE_componentWillMount()` {/*unsafe_componentwillmount*/}

如果你定义了 `UNSAFE_componentWillMount`，React 会在 [`constructor`.](#constructor) 之后立即调用它。它只因历史原因存在，不应用于任何新代码。相反，请使用以下替代方案之一：

- 要初始化 state，请将 [`state`](#state) 声明为类字段，或者在 [`constructor`](#constructor) 内设置 `this.state`。
- 如果你需要运行副作用或设置订阅，请改为将该逻辑移到 [`componentDidMount`](#componentdidmount)。

[查看迁移离开不安全生命周期的示例。](https://legacy.reactjs.org/blog/2018/03/27/update-on-async-rendering.html#examples)

#### 参数 {/*unsafe_componentwillmount-parameters*/}

`UNSAFE_componentWillMount` 不接受任何参数。

#### 返回值 {/*unsafe_componentwillmount-returns*/}

`UNSAFE_componentWillMount` 不应返回任何内容。

#### 注意事项 {/*unsafe_componentwillmount-caveats*/}

- 如果组件实现了 [`static getDerivedStateFromProps`](#static-getderivedstatefromprops) 或 [`getSnapshotBeforeUpdate`.](#getsnapshotbeforeupdate)，则不会调用 `UNSAFE_componentWillMount`。

- 尽管名称如此，如果你的应用使用了像 [`Suspense`.](/reference/react/Suspense) 这样的现代 React 特性，`UNSAFE_componentWillMount` 并不能保证组件*一定*会挂载。如果某次渲染尝试被挂起（例如，因为某个子组件的代码还没有加载完成），React 会丢弃正在进行中的树，并在下一次尝试时从头构造组件。这就是这个方法“不安全”的原因。依赖挂载的代码（如添加订阅）应放到 [`componentDidMount`.](#componentdidmount) 中。

- `UNSAFE_componentWillMount` 是唯一会在 [服务端渲染](/reference/react-dom/server) 期间运行的生命周期方法。就实际用途而言，它与 [`constructor`,](#constructor) 几乎相同，因此你应当改用 `constructor` 来处理这类逻辑。

<Note>

在类组件中于 `UNSAFE_componentWillMount` 内调用 [`setState`](#setstate) 来初始化 state，等同于在函数组件中将该 state 作为初始 state 传给 [`useState`](/reference/react/useState)。

</Note>

---

### `UNSAFE_componentWillReceiveProps(nextProps, nextContext)` {/*unsafe_componentwillreceiveprops*/}

如果你定义了 `UNSAFE_componentWillReceiveProps`，当组件接收到新的 props 时，React 会调用它。它只因历史原因存在，不应用于任何新代码。相反，请使用以下替代方案之一：

- 如果你需要在 props 变化时**运行副作用**（例如获取数据、播放动画或重新初始化订阅），请改为将该逻辑移到 [`componentDidUpdate`](#componentdidupdate)。
- 如果你需要**仅在 prop 变化时避免重新计算某些数据，**请改用 [记忆化辅助函数](https://legacy.reactjs.org/blog/2018/06/07/you-probably-dont-need-derived-state.html#what-about-memoization)。
- 如果你需要在 prop 变化时**“重置”某些 state，**请考虑将组件设为[完全受控](https://legacy.reactjs.org/blog/2018/06/07/you-probably-dont-need-derived-state.html#recommendation-fully-controlled-component)或[带 key 的完全非受控](https://legacy.reactjs.org/blog/2018/06/07/you-probably-dont-need-derived-state.html#recommendation-fully-uncontrolled-component-with-a-key)。
- 如果你需要在 prop 变化时**“调整”某些 state，**请检查是否可以在渲染时仅凭 props 计算出所有必要信息。如果不行，请改用 [`static getDerivedStateFromProps`](/reference/react/Component#static-getderivedstatefromprops)。

[查看迁移离开不安全生命周期的示例。](https://legacy.reactjs.org/blog/2018/03/27/update-on-async-rendering.html#updating-state-based-on-props)

#### 参数 {/*unsafe_componentwillreceiveprops-parameters*/}

- `nextProps`: 组件即将从其父组件接收的下一个 props。将 `nextProps` 与 [`this.props`](#props) 比较以确定哪些内容发生了变化。
- `nextContext`: 组件即将从最近的提供者接收的下一个 context。将 `nextContext` 与 [`this.context`](#context) 比较以确定哪些内容发生了变化。只有在你指定了 [`static contextType`](#static-contexttype) 时才可用。

#### 返回值 {/*unsafe_componentwillreceiveprops-returns*/}

`UNSAFE_componentWillReceiveProps` 不应返回任何内容。

#### 注意事项 {/*unsafe_componentwillreceiveprops-caveats*/}

- 如果组件实现了 [`static getDerivedStateFromProps`](#static-getderivedstatefromprops) 或 [`getSnapshotBeforeUpdate`.](#getsnapshotbeforeupdate)，则不会调用 `UNSAFE_componentWillReceiveProps`。

- 尽管名称如此，如果你的应用使用了像 [`Suspense`.](/reference/react/Suspense) 这样的现代 React 特性，`UNSAFE_componentWillReceiveProps` 并不能保证组件*一定*会收到这些 props。如果某次渲染尝试被挂起（例如，因为某个子组件的代码还没有加载完成），React 会丢弃正在进行中的树，并在下一次尝试时从头构造组件。到下一次渲染尝试时，props 可能已经不同。这就是这个方法“不安全”的原因。应该只在提交后的更新中运行的代码（例如重置订阅）应放到 [`componentDidUpdate`.](#componentdidupdate) 中。

- `UNSAFE_componentWillReceiveProps` 并不意味着组件收到的 props 与上一次不同。你需要自己比较 `nextProps` 和 `this.props` 来检查是否有变化。

- React 不会在挂载期间用初始 props 调用 `UNSAFE_componentWillReceiveProps`。只有当某些组件的 props 将要更新时才会调用这个方法。例如，在同一个组件中调用 [`setState`](#setstate) 通常不会触发 `UNSAFE_componentWillReceiveProps`。

<Note>

在类组件中于 `UNSAFE_componentWillReceiveProps` 内调用 [`setState`](#setstate) 来“调整” state，等同于在函数组件中[在渲染期间调用 `useState` 的 `set` 函数](/reference/react/useState#storing-information-from-previous-renders)。

</Note>

---

### `UNSAFE_componentWillUpdate(nextProps, nextState)` {/*unsafe_componentwillupdate*/}


如果你定义了 `UNSAFE_componentWillUpdate`，React 会在使用新的 props 或 state 渲染之前调用它。它只因历史原因存在，不应用于任何新代码。相反，请使用以下替代方案之一：

- 如果你需要在 props 或 state 变化时运行副作用（例如获取数据、播放动画或重新初始化订阅），请改为将该逻辑移到 [`componentDidUpdate`](#componentdidupdate)。
- 如果你需要从 DOM 中读取一些信息（例如保存当前滚动位置），以便稍后在 [`componentDidUpdate`](#componentdidupdate) 中使用，请改为在 [`getSnapshotBeforeUpdate`](#getsnapshotbeforeupdate) 内读取。

[查看迁移离开不安全生命周期的示例。](https://legacy.reactjs.org/blog/2018/03/27/update-on-async-rendering.html#examples)

#### 参数 {/*unsafe_componentwillupdate-parameters*/}

- `nextProps`: 组件即将使用来渲染的下一个 props。将 `nextProps` 与 [`this.props`](#props) 比较以确定哪些内容发生了变化。
- `nextState`: 组件即将使用来渲染的下一个 state。将 `nextState` 与 [`this.state`](#state) 比较以确定哪些内容发生了变化。

#### 返回值 {/*unsafe_componentwillupdate-returns*/}

`UNSAFE_componentWillUpdate` 不应返回任何内容。

#### 注意事项 {/*unsafe_componentwillupdate-caveats*/}

- 如果定义了 [`shouldComponentUpdate`](#shouldcomponentupdate) 并返回 `false`，则不会调用 `UNSAFE_componentWillUpdate`。

- 如果组件实现了 [`static getDerivedStateFromProps`](#static-getderivedstatefromprops) 或 [`getSnapshotBeforeUpdate`.](#getsnapshotbeforeupdate)，则不会调用 `UNSAFE_componentWillUpdate`。

- 在 `componentWillUpdate` 期间不支持调用 [`setState`](#setstate)（或任何会导致调用 `setState` 的方法，例如派发 Redux action）。

- 尽管名称如此，如果你的应用使用了像 [`Suspense`.](/reference/react/Suspense) 这样的现代 React 特性，`UNSAFE_componentWillUpdate` 并不能保证组件*一定*会更新。如果某次渲染尝试被挂起（例如，因为某个子组件的代码还没有加载完成），React 会丢弃正在进行中的树，并在下一次尝试时从头构造组件。到下一次渲染尝试时，props 和 state 可能已经不同。这就是这个方法“不安全”的原因。应该只在提交后的更新中运行的代码（例如重置订阅）应放到 [`componentDidUpdate`.](#componentdidupdate) 中。

- `UNSAFE_componentWillUpdate` 并不意味着组件收到的 props 或 state 与上一次不同。你需要自己比较 `nextProps` 与 `this.props`，以及 `nextState` 与 `this.state` 来检查是否有变化。

- React 不会在挂载期间用初始 props 和 state 调用 `UNSAFE_componentWillUpdate`。

<Note>

函数组件中没有与 `UNSAFE_componentWillUpdate` 直接对应的东西。

</Note>

---

### `static contextType` {/*static-contexttype*/}

如果你想从类组件中读取 [`this.context`](#context-instance-field)，必须指定它需要读取哪个 context。你作为 `static contextType` 指定的 context 必须是之前通过 [`createContext`.](/reference/react/createContext) 创建的值。

```js {2}
class Button extends Component {
  static contextType = ThemeContext;

  render() {
    const theme = this.context;
    const className = 'button-' + theme;
    return (
      <button className={className}>
        {this.props.children}
      </button>
    );
  }
}
```

<Note>

在类组件中读取 `this.context` 等同于在函数组件中使用 [`useContext`](/reference/react/useContext)。

[查看如何迁移。](#migrating-a-component-with-context-from-a-class-to-a-function)

</Note>

---

### `static defaultProps` {/*static-defaultprops*/}

你可以定义 `static defaultProps` 来为类设置默认 props。它们会用于 `undefined` 和缺失的 props，但不会用于 `null` props。

例如，下面是如何将 `color` prop 的默认值设置为 `'blue'`：

```js {2-4}
class Button extends Component {
  static defaultProps = {
    color: 'blue'
  };

  render() {
    return <button className={this.props.color}>click me</button>;
  }
}
```

如果未提供 `color` prop 或其值为 `undefined`，它将默认被设置为 `'blue'`：

```js
<>
  {/* this.props.color 是 "blue" */}
  <Button />

  {/* this.props.color 是 "blue" */}
  <Button color={undefined} />

  {/* this.props.color 是 null */}
  <Button color={null} />

  {/* this.props.color 是 "red" */}
  <Button color="red" />
</>
```

<Note>

在类组件中定义 `defaultProps` 类似于在函数组件中使用[默认值](/learn/passing-props-to-a-component#specifying-a-default-value-for-a-prop)。

</Note>

---

### `static getDerivedStateFromError(error)` {/*static-getderivedstatefromerror*/}

如果你定义了 `static getDerivedStateFromError`，当某个子组件（包括更远层级的子组件）在渲染过程中抛出错误时，React 会调用它。这让你可以显示错误信息，而不是清空 UI。

通常，它会与 [`componentDidCatch`](#componentdidcatch) 一起使用，后者让你可以将错误报告发送到某个分析服务。具有这些方法的组件称为 *错误边界*。

[查看示例。](#catching-rendering-errors-with-an-error-boundary)

#### 参数 {/*static-getderivedstatefromerror-parameters*/}

* `error`: 抛出的错误。实际上，它通常是 [`Error`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Error) 的一个实例，但这并不保证，因为 JavaScript 允许 [`throw`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/throw) 任意值，包括字符串甚至 `null`。

#### 返回值 {/*static-getderivedstatefromerror-returns*/}

`static getDerivedStateFromError` 应返回告诉组件显示错误信息的 state。

#### 注意事项 {/*static-getderivedstatefromerror-caveats*/}

* `static getDerivedStateFromError` 应该是纯函数。如果你想执行副作用（例如调用分析服务），你还需要实现 [`componentDidCatch`.](#componentdidcatch)

<Note>

目前，函数组件中还没有 `static getDerivedStateFromError` 的直接对应项。如果你想避免创建类组件，可以像上面那样编写一个 `ErrorBoundary` 组件，并在整个应用中使用它。或者，使用 [`react-error-boundary`](https://github.com/bvaughn/react-error-boundary) 包，它会完成这些工作。

</Note>

---

### `static getDerivedStateFromProps(props, state)` {/*static-getderivedstatefromprops*/}

如果你定义了 `static getDerivedStateFromProps`，React 会在调用 [`render`](#render) 之前立即调用它，无论是在初始挂载还是后续更新时。它应返回一个用于更新 state 的对象，或者返回 `null` 表示不更新任何内容。

这个方法适用于[state 随着 props 随时间变化而变化的少数用例](https://legacy.reactjs.org/blog/2018/06/07/you-probably-dont-need-derived-state.html#when-to-use-derived-state)。例如，当 `userID` prop 变化时，这个 `Form` 组件会重置 `email` state：

```js {7-18}
class Form extends Component {
  state = {
    email: this.props.defaultEmail,
    prevUserID: this.props.userID
  };

  static getDerivedStateFromProps(props, state) {
    // 每当当前用户变化时，
    // 重置任何与该用户绑定的 state 部分。
    // 在这个简单例子里，就是 email。
    if (props.userID !== state.prevUserID) {
      return {
        prevUserID: props.userID,
        email: props.defaultEmail
      };
    }
    return null;
  }

  // ...
}
```

请注意，这种模式要求你在 state 中保留 prop 的前一个值（比如 `userID`）（如 `prevUserID`）。

<Pitfall>

派生 state 会导致代码冗长，并使组件难以理解。[请确保你熟悉更简单的替代方案：](https://legacy.reactjs.org/blog/2018/06/07/you-probably-dont-need-derived-state.html)

- 如果你需要在 props 变化时**执行副作用**（例如数据获取或动画），请改用 [`componentDidUpdate`](#componentdidupdate) 方法。
- 如果你只想在 prop 变化时**重新计算某些数据，**[请改用记忆化辅助函数。](https://legacy.reactjs.org/blog/2018/06/07/you-probably-dont-need-derived-state.html#what-about-memoization)
- 如果你想在 prop 变化时**“重置”某些 state，**请考虑将组件设为[完全受控](https://legacy.reactjs.org/blog/2018/06/07/you-probably-dont-need-derived-state.html#recommendation-fully-controlled-component)或[带 key 的完全非受控](https://legacy.reactjs.org/blog/2018/06/07/you-probably-dont-need-derived-state.html#recommendation-fully-uncontrolled-component-with-a-key)。

</Pitfall>

#### 参数 {/*static-getderivedstatefromprops-parameters*/}

- `props`: 组件即将使用来渲染的下一个 props。
- `state`: 组件即将使用来渲染的下一个 state。

#### 返回值 {/*static-getderivedstatefromprops-returns*/}

`static getDerivedStateFromProps` 返回一个用于更新 state 的对象，或者返回 `null` 表示不更新任何内容。

#### 注意事项 {/*static-getderivedstatefromprops-caveats*/}

- 该方法会在*每次*渲染时触发，不管原因是什么。这不同于 [`UNSAFE_componentWillReceiveProps`](#unsafe_cmoponentwillreceiveprops)，后者只会在父组件导致重新渲染时触发，而不会因本地 `setState` 而触发。

- 该方法无法访问组件实例。如果你愿意，可以通过将组件 props 和 state 的纯函数提取到类定义外部，在 `static getDerivedStateFromProps` 与其他类方法之间复用一些代码。

<Note>

在类组件中实现 `static getDerivedStateFromProps` 等同于在函数组件中[在渲染期间调用 `useState` 的 `set` 函数](/reference/react/useState#storing-information-from-previous-renders)。

</Note>

---

## 用法 {/*usage*/}

### 定义类组件 {/*defining-a-class-component*/}

要将 React 组件定义为类，请扩展内置的 `Component` 类并定义一个 [`render` 方法：](#render)

```js
import { Component } from 'react';

class Greeting extends Component {
  render() {
    return <h1>Hello, {this.props.name}!</h1>;
  }
}
```

每当 React 需要弄清楚屏幕上应该显示什么时，都会调用你的 [`render`](#render) 方法。通常，你会从中返回一些 [JSX](/learn/writing-markup-with-jsx)。你的 `render` 方法应该是一个 [纯函数：](https://en.wikipedia.org/wiki/Pure_function) 它只应该计算 JSX。

与 [函数组件](/learn/your-first-component#defining-a-component) 类似，类组件也可以从其父组件 [接收通过 props 传递的信息](/learn/your-first-component#defining-a-component)。不过，读取 props 的语法不同。例如，如果父组件渲染 `<Greeting name="Taylor" />`，那么你可以从 [`this.props`](#props) 中读取 `name` prop，例如 `this.props.name`：

<Sandpack>

```js
import { Component } from 'react';

class Greeting extends Component {
  render() {
    return <h1>Hello, {this.props.name}!</h1>;
  }
}

export default function App() {
  return (
    <>
      <Greeting name="Sara" />
      <Greeting name="Cahal" />
      <Greeting name="Edite" />
    </>
  );
}
```

</Sandpack>

请注意，Hooks（以 `use` 开头的函数，例如 [`useState`](/reference/react/useState)）不支持在类组件内部使用。

<Pitfall>

我们建议将组件定义为函数而不是类。[查看如何迁移。](#migrating-a-simple-component-from-a-class-to-a-function)

</Pitfall>

---

### 向类组件添加状态 {/*adding-state-to-a-class-component*/}

要向类添加 [state](/learn/state-a-components-memory)，请将一个对象赋值给名为 [`state`](#state) 的属性。要更新 state，请调用 [`this.setState`](#setstate)。

<Sandpack>

```js
import { Component } from 'react';

export default class Counter extends Component {
  state = {
    name: 'Taylor',
    age: 42,
  };

  handleNameChange = (e) => {
    this.setState({
      name: e.target.value
    });
  }

  handleAgeChange = () => {
    this.setState({
      age: this.state.age + 1
    });
  };

  render() {
    return (
      <>
        <input
          value={this.state.name}
          onChange={this.handleNameChange}
        />
        <button onClick={this.handleAgeChange}>
          增加年龄
        </button>
        <p>你好，{this.state.name}。你今年 {this.state.age} 岁。</p>
      </>
    );
  }
}
```

```css
button { display: block; margin-top: 10px; }
```

</Sandpack>

<Pitfall>

我们建议将组件定义为函数而不是类。[查看如何迁移。](#migrating-a-component-with-state-from-a-class-to-a-function)

</Pitfall>

---

### 向类组件添加生命周期方法 {/*adding-lifecycle-methods-to-a-class-component*/}

你可以在类上定义几个特殊方法。

如果你定义了 [`componentDidMount`](#componentdidmount) 方法，React 会在你的组件被添加到屏幕上时 *(挂载时)* 调用它。React 会在你的组件因 props 或 state 变化而重新渲染后调用 [`componentDidUpdate`](#componentdidupdate)。React 会在你的组件从屏幕上被移除 *(卸载)* 后调用 [`componentWillUnmount`](#componentwillunmount)。

如果你实现了 `componentDidMount`，通常还需要实现全部三个生命周期方法以避免 bug。例如，如果 `componentDidMount` 读取了一些 state 或 props，你也必须实现 `componentDidUpdate` 来处理它们的变化，并实现 `componentWillUnmount` 来清理 `componentDidMount` 所做的一切。

例如，这个 `ChatRoom` 组件将聊天连接与 props 和 state 保持同步：

<Sandpack>

```js src/App.js
import { useState } from 'react';
import ChatRoom from './ChatRoom.js';

export default function App() {
  const [roomId, setRoomId] = useState('general');
  const [show, setShow] = useState(false);
  return (
    <>
      <label>
        选择聊天房间：{' '}
        <select
          value={roomId}
          onChange={e => setRoomId(e.target.value)}
        >
          <option value="general">general</option>
          <option value="travel">travel</option>
          <option value="music">music</option>
        </select>
      </label>
      <button onClick={() => setShow(!show)}>
        {show ? '关闭聊天' : '打开聊天'}
      </button>
      {show && <hr />}
      {show && <ChatRoom roomId={roomId} />}
    </>
  );
}
```

```js src/ChatRoom.js active
import { Component } from 'react';
import { createConnection } from './chat.js';

export default class ChatRoom extends Component {
  state = {
    serverUrl: 'https://localhost:1234'
  };

  componentDidMount() {
    this.setupConnection();
  }

  componentDidUpdate(prevProps, prevState) {
    if (
      this.props.roomId !== prevProps.roomId ||
      this.state.serverUrl !== prevState.serverUrl
    ) {
      this.destroyConnection();
      this.setupConnection();
    }
  }

  componentWillUnmount() {
    this.destroyConnection();
  }

  setupConnection() {
    this.connection = createConnection(
      this.state.serverUrl,
      this.props.roomId
    );
    this.connection.connect();
  }

  destroyConnection() {
    this.connection.disconnect();
    this.connection = null;
  }

  render() {
    return (
      <>
        <label>
          服务器 URL：{' '}
          <input
            value={this.state.serverUrl}
            onChange={e => {
              this.setState({
                serverUrl: e.target.value
              });
            }}
          />
        </label>
        <h1>欢迎来到 {this.props.roomId} 房间！</h1>
      </>
    );
  }
}
```

```js src/chat.js
export function createConnection(serverUrl, roomId) {
  // 一个真实的实现会实际连接到服务器
  return {
    connect() {
      console.log('✅ 正在连接到位于 ' + serverUrl + ' 的 "' + roomId + '" 房间...');
    },
    disconnect() {
      console.log('❌ 已从位于 ' + serverUrl + ' 的 "' + roomId + '" 房间断开连接');
    }
  };
}
```

```css
input { display: block; margin-bottom: 20px; }
button { margin-left: 10px; }
```

</Sandpack>

请注意，在开发环境中当 [Strict Mode](/reference/react/StrictMode) 开启时，React 会调用 `componentDidMount`，然后立即调用 `componentWillUnmount`，接着 আবার再次调用 `componentDidMount`。这有助于你发现是否忘记实现 `componentWillUnmount`，或者其逻辑是否没有完全“镜像” `componentDidMount` 所做的事情。

<Pitfall>

我们建议将组件定义为函数而不是类。[查看如何迁移。](#migrating-a-component-with-lifecycle-methods-from-a-class-to-a-function)

</Pitfall>

---

### 使用 Error Boundary 捕获渲染错误 {/*catching-rendering-errors-with-an-error-boundary*/}

默认情况下，如果你的应用在渲染过程中抛出错误，React 会把它的 UI 从屏幕上移除。为了防止这种情况，你可以将 UI 的一部分包裹在一个 *Error Boundary* 中。Error Boundary 是一种特殊组件，它允许你显示一些备用 UI 来替代崩溃的那部分——例如，一条错误消息。

<Note>
Error boundary 不会捕获以下错误：

- 事件处理函数 [(了解更多)](/learn/responding-to-events)
- [服务器端渲染](/reference/react-dom/server)
- 在 error boundary 自身中抛出的错误（而不是其子组件中）
- 异步代码（例如 `setTimeout` 或 `requestAnimationFrame` 回调）；一个例外是由 [`useTransition`](/reference/react/useTransition) Hook 返回的 [`startTransition`](/reference/react/useTransition#starttransition) 函数的使用。在 transition 函数内部抛出的错误会被 error boundary 捕获 [(了解更多)](/reference/react/useTransition#displaying-an-error-to-users-with-error-boundary)

</Note>

要实现一个 Error Boundary 组件，你需要提供 [`static getDerivedStateFromError`](#static-getderivedstatefromerror)，它允许你在响应错误时更新 state，并向用户显示错误消息。你也可以选择性地实现 [`componentDidCatch`](#componentdidcatch) 来添加一些额外逻辑，例如将错误记录到分析服务中。

使用 [`captureOwnerStack`](/reference/react/captureOwnerStack) 你可以在开发期间包含 Owner Stack。

```js {9-12,14-27}
import * as React from 'react';

class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error) {
    // 更新 state，以便下一次渲染显示备用 UI。
    return { hasError: true };
  }

  componentDidCatch(error, info) {
    logErrorToMyService(
      error,
      // 示例 "componentStack":
      //   in ComponentThatThrows (created by App)
      //   in ErrorBoundary (created by App)
      //   in div (created by App)
      //   in App
      info.componentStack,
      // 警告：`captureOwnerStack` 在生产环境中不可用。
      React.captureOwnerStack(),
    );
  }

  render() {
    if (this.state.hasError) {
      // 你可以渲染任何自定义的备用 UI
      return this.props.fallback;
    }

    return this.props.children;
  }
}
```

然后你可以用它包裹组件树的一部分：

```js {1,3}
<ErrorBoundary fallback={<p>出错了</p>}>
  <Profile />
</ErrorBoundary>
```

如果 `Profile` 或其子组件抛出错误，`ErrorBoundary` 将“捕获”该错误，显示你提供的带有错误消息的备用 UI，并向你的错误报告服务发送生产环境错误报告。

你不需要将每个组件都包裹进单独的 Error Boundary。当你考虑 [Error Boundary 的粒度](https://www.brandondail.com/posts/fault-tolerance-react) 时，请考虑在哪里显示错误消息才合适。例如，在消息应用中，将 Error Boundary 放在会话列表外层是合理的。将每条单独消息外层包裹一个也同样合理。然而，把每个头像都包裹一层就不合理了。

<Note>

目前没有办法将 Error Boundary 写成函数组件。不过，你不必自己编写 Error Boundary 类。例如，你可以改用 [`react-error-boundary`](https://github.com/bvaughn/react-error-boundary)。

</Note>

---

## 替代方案 {/*alternatives*/}

### 将一个简单的组件从类迁移到函数 {/*migrating-a-simple-component-from-a-class-to-a-function*/}

通常，您应该改为[将组件定义为函数](/learn/your-first-component#defining-a-component)。

例如，假设您正在将这个 `Greeting` 类组件转换为函数：

<Sandpack>

```js
import { Component } from 'react';

class Greeting extends Component {
  render() {
    return <h1>你好，{this.props.name}！</h1>;
  }
}

export default function App() {
  return (
    <>
      <Greeting name="Sara" />
      <Greeting name="Cahal" />
      <Greeting name="Edite" />
    </>
  );
}
```

</Sandpack>

定义一个名为 `Greeting` 的函数。这就是您要把 `render` 函数主体移动到的位置。

```js
function Greeting() {
  // ... 将 render 方法中的代码移动到这里 ...
}
```

不要使用 `this.props.name`，而是使用[解构语法](/learn/passing-props-to-a-component)定义 `name` prop，并直接读取它：

```js
function Greeting({ name }) {
  return <h1>你好，{name}！</h1>;
}
```

以下是一个完整示例：

<Sandpack>

```js
function Greeting({ name }) {
  return <h1>你好，{name}！</h1>;
}

export default function App() {
  return (
    <>
      <Greeting name="Sara" />
      <Greeting name="Cahal" />
      <Greeting name="Edite" />
    </>
  );
}
```

</Sandpack>

---

### 将带有状态的组件从类迁移到函数 {/*migrating-a-component-with-state-from-a-class-to-a-function*/}

假设您正在将这个 `Counter` 类组件转换为函数：

<Sandpack>

```js
import { Component } from 'react';

export default class Counter extends Component {
  state = {
    name: 'Taylor',
    age: 42,
  };

  handleNameChange = (e) => {
    this.setState({
      name: e.target.value
    });
  }

  handleAgeChange = (e) => {
    this.setState({
      age: this.state.age + 1
    });
  };

  render() {
    return (
      <>
        <input
          value={this.state.name}
          onChange={this.handleNameChange}
        />
        <button onClick={this.handleAgeChange}>
          年龄加一
        </button>
        <p>你好，{this.state.name}。你今年 {this.state.age} 岁。</p>
      </>
    );
  }
}
```

```css
button { display: block; margin-top: 10px; }
```

</Sandpack>

先使用必要的[state 变量：](/reference/react/useState#adding-state-to-a-component)来声明一个函数

```js {4-5}
import { useState } from 'react';

function Counter() {
  const [name, setName] = useState('Taylor');
  const [age, setAge] = useState(42);
  // ...
```

接下来，转换事件处理函数：

```js {5-7,9-11}
function Counter() {
  const [name, setName] = useState('Taylor');
  const [age, setAge] = useState(42);

  function handleNameChange(e) {
    setName(e.target.value);
  }

  function handleAgeChange() {
    setAge(age + 1);
  }
  // ...
```

最后，将所有以 `this` 开头的引用替换为您在组件中定义的变量和函数。例如，将 `this.state.age` 替换为 `age`，并将 `this.handleNameChange` 替换为 `handleNameChange`。

以下是一个完全转换后的组件：

<Sandpack>

```js
import { useState } from 'react';

export default function Counter() {
  const [name, setName] = useState('Taylor');
  const [age, setAge] = useState(42);

  function handleNameChange(e) {
    setName(e.target.value);
  }

  function handleAgeChange() {
    setAge(age + 1);
  }

  return (
    <>
      <input
        value={name}
        onChange={handleNameChange}
      />
      <button onClick={handleAgeChange}>
        年龄加一
      </button>
      <p>你好，{name}。你今年 {age} 岁。</p>
    </>
  )
}
```

```css
button { display: block; margin-top: 10px; }
```

</Sandpack>

---

### 将带有生命周期方法的组件从类迁移到函数 {/*migrating-a-component-with-lifecycle-methods-from-a-class-to-a-function*/}

假设您正在将这个带有生命周期方法的 `ChatRoom` 类组件转换为函数：

<Sandpack>

```js src/App.js
import { useState } from 'react';
import ChatRoom from './ChatRoom.js';

export default function App() {
  const [roomId, setRoomId] = useState('general');
  const [show, setShow] = useState(false);
  return (
    <>
      <label>
        选择聊天室：{' '}
        <select
          value={roomId}
          onChange={e => setRoomId(e.target.value)}
        >
          <option value="general">general</option>
          <option value="travel">travel</option>
          <option value="music">music</option>
        </select>
      </label>
      <button onClick={() => setShow(!show)}>
        {show ? '关闭聊天' : '打开聊天'}
      </button>
      {show && <hr />}
      {show && <ChatRoom roomId={roomId} />}
    </>
  );
}
```

```js src/ChatRoom.js active
import { Component } from 'react';
import { createConnection } from './chat.js';

export default class ChatRoom extends Component {
  state = {
    serverUrl: 'https://localhost:1234'
  };

  componentDidMount() {
    this.setupConnection();
  }

  componentDidUpdate(prevProps, prevState) {
    if (
      this.props.roomId !== prevProps.roomId ||
      this.state.serverUrl !== prevState.serverUrl
    ) {
      this.destroyConnection();
      this.setupConnection();
    }
  }

  componentWillUnmount() {
    this.destroyConnection();
  }

  setupConnection() {
    this.connection = createConnection(
      this.state.serverUrl,
      this.props.roomId
    );
    this.connection.connect();
  }

  destroyConnection() {
    this.connection.disconnect();
    this.connection = null;
  }

  render() {
    return (
      <>
        <label>
          服务器 URL：{' '}
          <input
            value={this.state.serverUrl}
            onChange={e => {
              this.setState({
                serverUrl: e.target.value
              });
            }}
          />
        </label>
        <h1>欢迎来到 {this.props.roomId} 房间！</h1>
      </>
    );
  }
}
```

```js src/chat.js
export function createConnection(serverUrl, roomId) {
  // 一个真实实现实际上会连接到服务器
  return {
    connect() {
      console.log('✅ 正在连接到 "' + roomId + '" 房间，地址为 ' + serverUrl + '...');
    },
    disconnect() {
      console.log('❌ 已从 "' + roomId + '" 房间断开，地址为 ' + serverUrl);
    }
  };
}
```

```css
input { display: block; margin-bottom: 20px; }
button { margin-left: 10px; }
```

</Sandpack>

首先，验证您的 [`componentWillUnmount`](#componentwillunmount) 是否与 [`componentDidMount`](#componentdidmount) 做相反的事情。在上面的示例中，情况确实如此：它断开了 `componentDidMount` 建立的连接。如果缺少这样的逻辑，请先补上。

接下来，验证您的 [`componentDidUpdate`](#componentdidupdate) 方法是否处理了您在 `componentDidMount` 中使用的任何 props 和 state 的变化。在上面的示例中，`componentDidMount` 调用了 `setupConnection`，而它会读取 `this.state.serverUrl` 和 `this.props.roomId`。这就是为什么 `componentDidUpdate` 会检查 `this.state.serverUrl` 和 `this.props.roomId` 是否发生了变化，并在它们变化时重置连接。如果您的 `componentDidUpdate` 逻辑缺失，或者没有处理所有相关 props 和 state 的变化，请先修复这一点。

在上面的示例中，生命周期方法内部的逻辑将组件连接到 React 外部的某个系统（聊天服务器）。要将组件连接到外部系统，请[将这段逻辑描述为一个单独的 Effect：](/reference/react/useEffect#connecting-to-an-external-system)

```js {6-12}
import { useState, useEffect } from 'react';

function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [serverUrl, roomId]);

  // ...
}
```

这个 [`useEffect`](/reference/react/useEffect) 调用等同于上面生命周期方法中的逻辑。如果您的生命周期方法做了多个彼此无关的事情，[请将它们拆分为多个独立的 Effect。](/learn/removing-effect-dependencies#is-your-effect-doing-several-unrelated-things) 这里有一个完整示例，您可以动手试试：

<Sandpack>

```js src/App.js
import { useState } from 'react';
import ChatRoom from './ChatRoom.js';

export default function App() {
  const [roomId, setRoomId] = useState('general');
  const [show, setShow] = useState(false);
  return (
    <>
      <label>
        选择聊天室：{' '}
        <select
          value={roomId}
          onChange={e => setRoomId(e.target.value)}
        >
          <option value="general">general</option>
          <option value="travel">travel</option>
          <option value="music">music</option>
        </select>
      </label>
      <button onClick={() => setShow(!show)}>
        {show ? '关闭聊天' : '打开聊天'}
      </button>
      {show && <hr />}
      {show && <ChatRoom roomId={roomId} />}
    </>
  );
}
```

```js src/ChatRoom.js active
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

export default function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [roomId, serverUrl]);

  return (
    <>
      <label>
        服务器 URL：{' '}
        <input
          value={serverUrl}
          onChange={e => setServerUrl(e.target.value)}
        />
      </label>
      <h1>欢迎来到 {roomId} 房间！</h1>
    </>
  );
}
```

```js src/chat.js
export function createConnection(serverUrl, roomId) {
  // 一个真实实现实际上会连接到服务器
  return {
    connect() {
      console.log('✅ 正在连接到 "' + roomId + '" 房间，地址为 ' + serverUrl + '...');
    },
    disconnect() {
      console.log('❌ 已从 "' + roomId + '" 房间断开，地址为 ' + serverUrl);
    }
  };
}
```

```css
input { display: block; margin-bottom: 20px; }
button { margin-left: 10px; }
```

</Sandpack>

<Note>

如果您的组件不与任何外部系统同步，[您可能不需要 Effect。](/learn/you-might-not-need-an-effect)

</Note>

---

### 将带有上下文的组件从类迁移到函数 {/*migrating-a-component-with-context-from-a-class-to-a-function*/}

在这个示例中，`Panel` 和 `Button` 类组件从 [`this.context`:](#context) 读取[上下文](/learn/passing-data-deeply-with-context)

<Sandpack>

```js
import { createContext, Component } from 'react';

const ThemeContext = createContext(null);

class Panel extends Component {
  static contextType = ThemeContext;

  render() {
    const theme = this.context;
    const className = 'panel-' + theme;
    return (
      <section className={className}>
        <h1>{this.props.title}</h1>
        {this.props.children}
      </section>
    );
  }
}

class Button extends Component {
  static contextType = ThemeContext;

  render() {
    const theme = this.context;
    const className = 'button-' + theme;
    return (
      <button className={className}>
        {this.props.children}
      </button>
    );
  }
}

function Form() {
  return (
    <Panel title="欢迎">
      <Button>注册</Button>
      <Button>登录</Button>
    </Panel>
  );
}

export default function MyApp() {
  return (
    <ThemeContext value="dark">
      <Form />
    </ThemeContext>
  )
}
```

```css
.panel-light,
.panel-dark {
  border: 1px solid black;
  border-radius: 4px;
  padding: 20px;
}
.panel-light {
  color: #222;
  background: #fff;
}

.panel-dark {
  color: #fff;
  background: rgb(23, 32, 42);
}

.button-light,
.button-dark {
  border: 1px solid #777;
  padding: 5px;
  margin-right: 10px;
  margin-top: 10px;
}

.button-dark {
  background: #222;
  color: #fff;
}

.button-light {
  background: #fff;
  color: #222;
}
```

</Sandpack>

当您将它们转换为函数组件时，请用 [`useContext`](/reference/react/useContext) 调用替换 `this.context`：

<Sandpack>

```js
import { createContext, useContext } from 'react';

const ThemeContext = createContext(null);

function Panel({ title, children }) {
  const theme = useContext(ThemeContext);
  const className = 'panel-' + theme;
  return (
    <section className={className}>
      <h1>{title}</h1>
      {children}
    </section>
  )
}

function Button({ children }) {
  const theme = useContext(ThemeContext);
  const className = 'button-' + theme;
  return (
    <button className={className}>
      {children}
    </button>
  );
}

function Form() {
  return (
    <Panel title="欢迎">
      <Button>注册</Button>
      <Button>登录</Button>
    </Panel>
  );
}

export default function MyApp() {
  return (
    <ThemeContext value="dark">
      <Form />
    </ThemeContext>
  )
}
```

```css
.panel-light,
.panel-dark {
  border: 1px solid black;
  border-radius: 4px;
  padding: 20px;
}
.panel-light {
  color: #222;
  background: #fff;
}

.panel-dark {
  color: #fff;
  background: rgb(23, 32, 42);
}

.button-light,
.button-dark {
  border: 1px solid #777;
  padding: 5px;
  margin-right: 10px;
  margin-top: 10px;
}

.button-dark {
  background: #222;
  color: #fff;
}

.button-light {
  background: #fff;
  color: #222;
}
```

</Sandpack>
