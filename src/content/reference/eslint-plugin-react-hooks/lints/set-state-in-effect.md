---
title: set-state-in-effect
---

<Intro>

验证是否在 effect 中同步调用 setState，因为这会导致重新渲染，从而降低性能。

</Intro>

## Rule Details {/*rule-details*/}

在 effect 内立即设置状态会强制 React 重新开始整个渲染周期。当你在 effect 中更新状态时，React 必须重新渲染你的组件、将更改应用到 DOM，然后再次运行 effects。这会额外增加一次本可以通过在渲染期间直接转换数据或从 props 派生状态来避免的渲染过程。请改为在组件顶层转换数据。这样，当 props 或 state 变化时，这段代码会自然重新执行，而不会触发额外的渲染周期。

在 effects 中同步调用 `setState` 会在浏览器绘制之前触发立即重新渲染，从而引发性能问题和视觉卡顿。React 必须渲染两次：一次用于应用状态更新，另一次用于 effects 运行之后。当同样的结果可以通过一次渲染实现时，这种双重渲染就是浪费。

在许多情况下，你甚至可能根本不需要 effect。更多信息请参见 [You Might Not Need an Effect](/learn/you-might-not-need-an-effect)。

## Common Violations {/*common-violations*/}

此规则会捕获几种不必要地使用同步 setState 的模式：

- 同步设置 loading 状态
- 在 effects 中从 props 派生状态
- 在 effects 中转换数据，而不是在 render 中转换

### Invalid {/*invalid*/}

此规则的错误代码示例：

```js
// ❌ 在 effect 中同步 setState
function Component({data}) {
  const [items, setItems] = useState([]);

  useEffect(() => {
    setItems(data); // 额外的渲染，应使用初始状态代替
  }, [data]);
}

// ❌ 同步设置 loading 状态
function Component() {
  const [loading, setLoading] = useState(false);

  useEffect(() => {
    setLoading(true); // 同步执行，会导致额外渲染
    fetchData().then(() => setLoading(false));
  }, []);
}

// ❌ 在 effect 中转换数据
function Component({rawData}) {
  const [processed, setProcessed] = useState([]);

  useEffect(() => {
    setProcessed(rawData.map(transform)); // 应在 render 中派生
  }, [rawData]);
}

// ❌ 从 props 派生状态
function Component({selectedId, items}) {
  const [selected, setSelected] = useState(null);

  useEffect(() => {
    setSelected(items.find(i => i.id === selectedId));
  }, [selectedId, items]);
}
```

### Valid {/*valid*/}

此规则的正确代码示例：

```js
// ✅ 如果值来自 ref，那么在 effect 中 setState 是可以的
function Tooltip() {
  const ref = useRef(null);
  const [tooltipHeight, setTooltipHeight] = useState(0);

  useLayoutEffect(() => {
    const { height } = ref.current.getBoundingClientRect();
    setTooltipHeight(height);
  }, []);
}

// ✅ 在 render 期间计算
function Component({selectedId, items}) {
  const selected = items.find(i => i.id === selectedId);
  return <div>{selected?.name}</div>;
}
```

**当某个值可以由现有的 props 或 state 计算得出时，不要把它放进 state。** 相反，请在渲染期间计算它。这会让你的代码更快、更简单，也更不容易出错。更多信息请参见 [You Might Not Need an Effect](/learn/you-might-not-need-an-effect)。
