---
title: rules-of-hooks
---

<Intro>

验证组件和 hooks 是否遵循 [Hook 规则](/reference/rules/rules-of-hooks)。

</Intro>

## 规则详情 {/*rule-details*/}

React 依赖 hooks 的调用顺序，以便在多次渲染之间正确保留状态。每次组件渲染时，React 都期望以完全相同的顺序调用完全相同的 hooks。当 hooks 被条件调用或在循环中调用时，React 会失去对哪个状态对应哪个 hook 调用的跟踪，从而导致诸如状态不匹配以及“比预期渲染了更少/更多的 hooks”之类的错误。

## 常见违规 {/*common-violations*/}

以下模式违反 Hook 规则：

- **条件中的 Hooks**（`if`/`else`、三元运算符、`&&`/`||`）
- **循环中的 Hooks**（`for`、`while`、`do-while`）
- **提前返回后的 Hooks**
- **回调/事件处理器中的 Hooks**
- **异步函数中的 Hooks**
- **类方法中的 Hooks**
- **模块级别的 Hooks**

<Note>

### `use` hook {/*use-hook*/}

`use` hook 与其他 React hooks 不同。你可以在条件中和循环中调用它：

```js
// ✅ `use` 可以是条件调用
if (shouldFetch) {
  const data = use(fetchPromise);
}

// ✅ `use` 可以在循环中
for (const promise of promises) {
  results.push(use(promise));
}
```

不过，`use` 仍然有一些限制：
- 不能被 try/catch 包裹
- 必须在组件或 hook 内部调用

了解更多：[`use` API 参考](/reference/react/use)

</Note>

### 无效 {/*invalid*/}

此规则的错误代码示例：

```js
// ❌ 条件中的 Hook
if (isLoggedIn) {
  const [user, setUser] = useState(null);
}

// ❌ 提前返回后的 Hook
if (!data) return <Loading />;
const [processed, setProcessed] = useState(data);

// ❌ 回调中的 Hook
<button onClick={() => {
  const [clicked, setClicked] = useState(false);
}}/>

// ❌ try/catch 中的 `use`
try {
  const data = use(promise);
} catch (e) {
  // 错误处理
}

// ❌ 模块级别的 Hook
const globalState = useState(0); // 在组件外部
```

### 有效 {/*valid*/}

此规则的正确代码示例：

```js
function Component({ isSpecial, shouldFetch, fetchPromise }) {
  // ✅ Hooks 在顶层
  const [count, setCount] = useState(0);
  const [name, setName] = useState('');

  if (!isSpecial) {
    return null;
  }

  if (shouldFetch) {
    // ✅ `use` 可以是条件调用
    const data = use(fetchPromise);
    return <div>{data}</div>;
  }

  return <div>{name}: {count}</div>;
}
```

## 故障排查 {/*troubleshooting*/}

### 我想根据某个条件获取数据 {/*conditional-data-fetching*/}

你正在尝试有条件地调用 useEffect：

```js
// ❌ 条件 Hook
if (isLoggedIn) {
  useEffect(() => {
    fetchUserData();
  }, []);
}
```

无条件地调用 hook，并在内部检查条件：

```js
// ✅ hook 内部的条件
useEffect(() => {
  if (isLoggedIn) {
    fetchUserData();
  }
}, [isLoggedIn]);
```

<Note>

相比在 useEffect 中获取数据，有更好的方式。可以考虑使用 TanStack Query、useSWR 或 React Router 6.4+ 来获取数据。这些方案可以处理请求去重、响应缓存，以及避免网络瀑布。

了解更多：[获取数据](/learn/synchronizing-with-effects#fetching-data)

</Note>

### 我需要针对不同场景使用不同的状态 {/*conditional-state-initialization*/}

你正在尝试有条件地初始化状态：

```js
// ❌ 条件状态
if (userType === 'admin') {
  const [permissions, setPermissions] = useState(adminPerms);
} else {
  const [permissions, setPermissions] = useState(userPerms);
}
```

始终调用 useState，并有条件地设置初始值：

```js
// ✅ 条件初始值
const [permissions, setPermissions] = useState(
  userType === 'admin' ? adminPerms : userPerms
);
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

- `additionalEffectHooks`：与应被视为 effect 的自定义 hooks 匹配的正则表达式模式。这使得 `useEffectEvent` 和类似的事件函数可以从你的自定义 effect hooks 中调用。

此共享配置同时用于 `rules-of-hooks` 和 `exhaustive-deps` 规则，确保所有与 hook 相关的 lint 检查行为保持一致。
