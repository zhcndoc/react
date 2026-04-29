---
title: pure
---

<Intro>

通过检查它们是否调用已知的不纯函数，来验证 [组件/Hook 是纯的](/reference/rules/components-and-hooks-must-be-pure)。

</Intro>

## 规则详情 {/*rule-details*/}

React 组件必须是纯函数——在给定相同 props 的情况下，它们应始终返回相同的 JSX。当组件在渲染期间使用 `Math.random()` 或 `Date.now()` 之类的函数时，每次都会产生不同的输出，从而破坏 React 的假设，并导致诸如 hydration 不匹配、错误的 memoization 以及不可预测的行为等 bug。

## 常见违规 {/*common-violations*/}

一般来说，任何在相同输入下返回不同值的 API 都违反此规则。常见示例包括：

- `Math.random()`
- `Date.now()` / `new Date()`
- `crypto.randomUUID()`
- `performance.now()`

### 无效 {/*invalid*/}

以下是此规则的错误代码示例：

```js
// ❌ 在渲染中使用 Math.random()
function Component() {
  const id = Math.random(); // 每次渲染都不同
  return <div key={id}>内容</div>;
}

// ❌ 将 Date.now() 用于值
function Component() {
  const timestamp = Date.now(); // 每次渲染都会变化
  return <div>创建于：{timestamp}</div>;
}
```

### 有效 {/*valid*/}

以下是此规则的正确代码示例：

```js
// ✅ 来自初始 state 的稳定 ID
function Component() {
  const [id] = useState(() => crypto.randomUUID());
  return <div key={id}>内容</div>;
}
```

## 故障排查 {/*troubleshooting*/}

### 我需要显示当前时间 {/*current-time*/}

在渲染期间调用 `Date.now()` 会使你的组件变得不纯：

```js {expectedErrors: {'react-compiler': [3]}}
// ❌ 错误：时间每次渲染都会变化
function Clock() {
  return <div>当前时间：{Date.now()}</div>;
}
```

相反，[将不纯函数移到渲染之外](/reference/rules/components-and-hooks-must-be-pure#components-and-hooks-must-be-idempotent)：

```js
function Clock() {
  const [time, setTime] = useState(() => Date.now());

  useEffect(() => {
    const interval = setInterval(() => {
      setTime(Date.now());
    }, 1000);

    return () => clearInterval(interval);
  }, []);

  return <div>当前时间：{time}</div>;
}
```