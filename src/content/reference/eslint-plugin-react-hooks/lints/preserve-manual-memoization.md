---
title: preserve-manual-memoization
---

<Intro>

验证编译器会保留现有的手动 memoization。只有当 React Compiler 的推断[与现有的手动 memoization 相匹配或更优时](/learn/react-compiler/introduction#what-should-i-do-about-usememo-usecallback-and-reactmemo)，它才会编译组件和 hooks。

</Intro>

## 规则详情 {/*rule-details*/}

React Compiler 会保留你现有的 `useMemo`、`useCallback` 和 `React.memo` 调用。如果你手动对某些内容进行了 memoization，编译器会认为你这样做是有充分理由的，不会将其移除。不过，不完整的依赖项会阻止编译器理解代码的数据流并应用进一步的优化。

### 无效 {/*invalid*/}

以下是此规则的错误代码示例：

```js
// ❌ useMemo 中缺少依赖项
function Component({ data, filter }) {
  const filtered = useMemo(
    () => data.filter(filter),
    [data] // 缺少 'filter' 依赖项
  );

  return <List items={filtered} />;
}

// ❌ useCallback 中缺少依赖项
function Component({ onUpdate, value }) {
  const handleClick = useCallback(() => {
    onUpdate(value);
  }, [onUpdate]); // 缺少 'value'

  return <button onClick={handleClick}>更新</button>;
}
```

### 有效 {/*valid*/}

以下是此规则的正确代码示例：

```js
// ✅ 完整的依赖项
function Component({ data, filter }) {
  const filtered = useMemo(
    () => data.filter(filter),
    [data, filter] // 包含所有依赖项
  );

  return <List items={filtered} />;
}

// ✅ 或者让编译器来处理
function Component({ data, filter }) {
  // 不需要手动 memoization
  const filtered = data.filter(filter);
  return <List items={filtered} />;
}
```

## 故障排查 {/*troubleshooting*/}

### 我应该移除手动 memoization 吗？ {/*remove-manual-memoization*/}

你可能会想，React Compiler 是否会让手动 memoization 变得不再必要：

```js
// 我还需要这个吗？
function Component({items, sortBy}) {
  const sorted = useMemo(() => {
    return [...items].sort((a, b) => {
      return a[sortBy] - b[sortBy];
    });
  }, [items, sortBy]);

  return <List items={sorted} />;
}
```

如果使用 React Compiler，你可以安全地移除它：

```js
// ✅ 更好：让编译器进行优化
function Component({items, sortBy}) {
  const sorted = [...items].sort((a, b) => {
    return a[sortBy] - b[sortBy];
  });

  return <List items={sorted} />;
}
```