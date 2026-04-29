---
title: PureComponent
---

<Pitfall>

我们建议将组件定义为函数而不是类。[查看如何迁移。](#alternatives)

</Pitfall>

<Intro>

`PureComponent` 类似于 [`Component`](/reference/react/Component)，但它会跳过 props 和 state 相同情况下的重新渲染。React 仍然支持类组件，但我们不建议在新代码中使用它们。

```js
class Greeting extends PureComponent {
  render() {
    return <h1>Hello, {this.props.name}!</h1>;
  }
}
```

</Intro>

<InlineToc />

---

## Reference {/*reference*/}

### `PureComponent` {/*purecomponent*/}

要在 props 和 state 相同的情况下跳过类组件的重新渲染，请扩展 `PureComponent` 而不是 [`Component`:](/reference/react/Component)

```js
import { PureComponent } from 'react';

class Greeting extends PureComponent {
  render() {
    return <h1>Hello, {this.props.name}!</h1>;
  }
}
```

`PureComponent` 是 `Component` 的子类，并支持 [`Component` 的所有 API。](/reference/react/Component#reference) 扩展 `PureComponent` 等同于定义一个自定义的 [`shouldComponentUpdate`](/reference/react/Component#shouldcomponentupdate) 方法，该方法会对 props 和 state 进行浅比较。


[查看更多示例。](#usage)

---

## Usage {/*usage*/}

### 跳过类组件不必要的重新渲染 {/*skipping-unnecessary-re-renders-for-class-components*/}

React 通常会在父组件重新渲染时重新渲染组件。作为一种优化，你可以创建一个组件，只要它的新 props 和 state 与旧的 props 和 state 相同，React 就不会在其父组件重新渲染时重新渲染它。[类组件](/reference/react/Component) 可以通过扩展 `PureComponent` 来选择启用这种行为：

```js {1}
class Greeting extends PureComponent {
  render() {
    return <h1>Hello, {this.props.name}!</h1>;
  }
}
```

React 组件应该始终具有[纯渲染逻辑。](/learn/keeping-components-pure) 这意味着如果它的 props、state 和 context 没有改变，它必须返回相同的输出。通过使用 `PureComponent`，你是在告诉 React 你的组件符合这一要求，因此只要它的 props 和 state 没有改变，React 就不需要重新渲染它。不过，如果它使用的某个 context 发生变化，你的组件仍然会重新渲染。

在这个示例中，注意 `Greeting` 组件会在 `name` 变化时重新渲染（因为它是其 props 之一），但在 `address` 变化时不会重新渲染（因为它没有作为 prop 传给 `Greeting`）：

<Sandpack>

```js
import { PureComponent, useState } from 'react';

class Greeting extends PureComponent {
  render() {
    console.log("Greeting was rendered at", new Date().toLocaleTimeString());
    return <h3>Hello{this.props.name && ', '}{this.props.name}!</h3>;
  }
}

export default function MyApp() {
  const [name, setName] = useState('');
  const [address, setAddress] = useState('');
  return (
    <>
      <label>
        姓名{': '}
        <input value={name} onChange={e => setName(e.target.value)} />
      </label>
      <label>
        地址{': '}
        <input value={address} onChange={e => setAddress(e.target.value)} />
      </label>
      <Greeting name={name} />
    </>
  );
}
```

```css
label {
  display: block;
  margin-bottom: 16px;
}
```

</Sandpack>

<Pitfall>

我们建议将组件定义为函数而不是类。[查看如何迁移。](#alternatives)

</Pitfall>

---

## Alternatives {/*alternatives*/}

### 将 `PureComponent` 类组件迁移为函数组件 {/*migrating-from-a-purecomponent-class-component-to-a-function*/}

我们建议在新代码中使用函数组件，而不是[类组件](/reference/react/Component)。如果你已有一些使用 `PureComponent` 的类组件，下面是将它们转换的方法。这是原始代码：

<Sandpack>

```js
import { PureComponent, useState } from 'react';

class Greeting extends PureComponent {
  render() {
    console.log("Greeting was rendered at", new Date().toLocaleTimeString());
    return <h3>Hello{this.props.name && ', '}{this.props.name}!</h3>;
  }
}

export default function MyApp() {
  const [name, setName] = useState('');
  const [address, setAddress] = useState('');
  return (
    <>
      <label>
        姓名{': '}
        <input value={name} onChange={e => setName(e.target.value)} />
      </label>
      <label>
        地址{': '}
        <input value={address} onChange={e => setAddress(e.target.value)} />
      </label>
      <Greeting name={name} />
    </>
  );
}
```

```css
label {
  display: block;
  margin-bottom: 16px;
}
```

</Sandpack>

当你[将这个组件从类转换为函数时，](/reference/react/Component#alternatives)请用 [`memo`:](/reference/react/memo) 包裹它

<Sandpack>

```js
import { memo, useState } from 'react';

const Greeting = memo(function Greeting({ name }) {
  console.log("Greeting was rendered at", new Date().toLocaleTimeString());
  return <h3>Hello{name && ', '}{name}!</h3>;
});

export default function MyApp() {
  const [name, setName] = useState('');
  const [address, setAddress] = useState('');
  return (
    <>
      <label>
        姓名{': '}
        <input value={name} onChange={e => setName(e.target.value)} />
      </label>
      <label>
        地址{': '}
        <input value={address} onChange={e => setAddress(e.target.value)} />
      </label>
      <Greeting name={name} />
    </>
  );
}
```

```css
label {
  display: block;
  margin-bottom: 16px;
}
```

</Sandpack>

<Note>

与 `PureComponent` 不同，[`memo`](/reference/react/memo) 不会比较新的和旧的 state。在函数组件中，使用相同 state 调用 [`set` 函数](/reference/react/useState#setstate) [默认就已经会阻止重新渲染，](/reference/react/memo#updating-a-memoized-component-using-state) 即使不使用 `memo` 也是如此。

</Note>
