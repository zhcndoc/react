---
title: set-state-in-render
---

<Intro>

验证在渲染期间无条件设置状态的情况，这可能会触发额外的渲染以及潜在的无限渲染循环。

</Intro>

## Rule Details {/*rule-details*/}

在渲染期间无条件调用 `setState` 会在当前渲染完成之前触发另一次渲染。这会创建一个无限循环，导致你的应用崩溃。

## Common Violations {/*common-violations*/}

### Invalid {/*invalid*/}

```js {expectedErrors: {'react-compiler': [4]}}
// ❌ 在 render 中直接无条件 setState
function Component({value}) {
  const [count, setCount] = useState(0);
  setCount(value); // 无限循环！
  return <div>{count}</div>;
}
```

### Valid {/*valid*/}

```js
// ✅ 在 render 期间派生
function Component({items}) {
  const sorted = [...items].sort(); // 只需在 render 中计算它
  return <ul>{sorted.map(/*...*/)}</ul>;
}

// ✅ 在事件处理器中设置状态
function Component() {
  const [count, setCount] = useState(0);
  return (
    <button onClick={() => setCount(count + 1)}>
      {count}
    </button>
  );
}

// ✅ 从 props 派生，而不是设置状态
function Component({user}) {
  const name = user?.name || '';
  const email = user?.email || '';
  return <div>{name}</div>;
}

// ✅ 根据 props 和来自先前渲染的 state 有条件地派生 state
function Component({ items }) {
  const [isReverse, setIsReverse] = useState(false);
  const [selection, setSelection] = useState(null);

  const [prevItems, setPrevItems] = useState(items);
  if (items !== prevItems) { // 这个条件使其有效
    setPrevItems(items);
    setSelection(null);
  }
  // ...
}
```

## Troubleshooting {/*troubleshooting*/}

### I want to sync state to a prop {/*clamp-state-to-prop*/}

一个常见问题是在渲染后尝试“修正”状态。假设你想防止计数器超过 `max` prop：

```js
// ❌ 错误：在 render 期间进行限制
function Counter({max}) {
  const [count, setCount] = useState(0);

  if (count > max) {
    setCount(max);
  }

  return (
    <button onClick={() => setCount(count + 1)}>
      {count}
    </button>
  );
}
```

一旦 `count` 超过 `max`，就会触发无限循环。

相反，通常更好的做法是把这段逻辑移到事件中（也就是首次设置状态的地方）。例如，你可以在更新状态时直接强制限制上限：

```js
// ✅ 在更新时限制
function Counter({max}) {
  const [count, setCount] = useState(0);

  const increment = () => {
    setCount(current => Math.min(current + 1, max));
  };

  return <button onClick={increment}>{count}</button>;
}
```

现在，setter 只会在点击时运行，React 会正常完成渲染，而且 `count` 永远不会超过 `max`。

在少数情况下，你可能需要根据前一次渲染的信息来调整状态。对于这些情况，请遵循[此模式](https://react.dev/reference/react/useState#storing-information-from-previous-renders)来有条件地设置状态。
