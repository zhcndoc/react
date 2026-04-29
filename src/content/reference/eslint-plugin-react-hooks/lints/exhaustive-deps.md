---
title: exhaustive-deps
---

<Intro>

验证 React hooks 的依赖数组是否包含所有必要的依赖。

</Intro>

## 规则详情 {/*rule-details*/}

像 `useEffect`、`useMemo` 和 `useCallback` 这样的 React hooks 接受依赖数组。当这些 hooks 内部引用的值未包含在依赖数组中时，React 在该依赖变化时不会重新执行 effect 或重新计算该值。这会导致陈旧闭包，hooks 使用到的是过时的值。

## 常见违规 {/*common-violations*/}

当你试图通过“欺骗” React 关于依赖关系来控制 effect 何时运行时，通常会发生这个错误。Effect 应该让你的组件与外部系统保持同步。依赖数组会告诉 React 这个 effect 使用了哪些值，因此 React 才知道何时需要重新同步。

如果你发现自己在和 linter 对抗，你很可能需要重构代码。查看 [移除 Effect 依赖](/learn/removing-effect-dependencies) 了解方法。

### 无效 {/*invalid*/}

以下是此规则的错误代码示例：

```js
// ❌ 缺少依赖
useEffect(() => {
  console.log(count);
}, []); // 缺少 'count'

// ❌ 缺少 prop
useEffect(() => {
  fetchUser(userId);
}, []); // 缺少 'userId'

// ❌ 依赖不完整
useMemo(() => {
  return items.sort(sortOrder);
}, [items]); // 缺少 'sortOrder'
```

### 有效 {/*valid*/}

以下是此规则的正确代码示例：

```js
// ✅ 包含所有依赖
useEffect(() => {
  console.log(count);
}, [count]);

// ✅ 包含所有依赖
useEffect(() => {
  fetchUser(userId);
}, [userId]);
```

## 故障排查 {/*troubleshooting*/}

### 添加函数依赖会导致无限循环 {/*function-dependency-loops*/}

你有一个 effect，但你在每次渲染时都创建了一个新函数：

```js
// ❌ 会导致无限循环
const logItems = () => {
  console.log(items);
};

useEffect(() => {
  logItems();
}, [logItems]); // 无限循环！
```

在大多数情况下，你并不需要这个 effect。改为在动作发生的地方调用这个函数：

```js
// ✅ 在事件处理函数中调用它
const logItems = () => {
  console.log(items);
};

return <button onClick={logItems}>记录</button>;

// ✅ 或者如果没有副作用，就在渲染过程中直接派生
items.forEach(item => {
  console.log(item);
});
```

如果你确实需要这个 effect（例如，订阅某个外部内容），请让依赖保持稳定：

```js
// ✅ useCallback 会保持函数引用稳定
const logItems = useCallback(() => {
  console.log(items);
}, [items]);

useEffect(() => {
  logItems();
}, [logItems]);

// ✅ 或者直接把逻辑移到 effect 中
useEffect(() => {
  console.log(items);
}, [items]);
```

### 只运行一次 effect {/*effect-on-mount*/}

你希望 effect 在挂载时只运行一次，但 linter 提示缺少依赖：

```js
// ❌ 缺少依赖
useEffect(() => {
  sendAnalytics(userId);
}, []); // 缺少 'userId'
```

要么包含该依赖（推荐），要么如果你确实只需要运行一次，就使用 ref：

```js
// ✅ 包含依赖
useEffect(() => {
  sendAnalytics(userId);
}, [userId]);

// ✅ 或者在 effect 中使用 ref 保护
const sent = useRef(false);

useEffect(() => {
  if (sent.current) {
    return;
  }

  sent.current = true;
  sendAnalytics(userId);
}, [userId]);
```

## 选项 {/*options*/}

你可以使用共享的 ESLint 设置来配置自定义 effect hooks（适用于 `eslint-plugin-react-hooks` 6.1.1 及更高版本）：

```js
{
  "settings": {
    "react-hooks": {
      "additionalEffectHooks": "(useMyEffect|useCustomEffect)"
    }
  }
}
```

- `additionalEffectHooks`：用于匹配应检查完整依赖项的自定义 hooks 的正则表达式模式。此配置会在所有 `react-hooks` 规则之间共享。

为了向后兼容，此规则也接受规则级别的选项：

```js
{
  "rules": {
    "react-hooks/exhaustive-deps": ["warn", {
      "additionalHooks": "(useMyCustomHook|useAnotherHook)"
    }]
  }
}
```

- `additionalHooks`：用于检查完整依赖项的 hooks 的正则表达式。**注意：**如果指定了此规则级别的选项，它将优先于共享的 `settings` 配置。
