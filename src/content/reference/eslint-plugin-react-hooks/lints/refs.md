---
title: refs
---

<Intro>

验证 ref 的正确用法，不要在渲染期间读取/写入。另请参见 [`useRef()` usage](/reference/react/useRef#usage) 中的“pitfalls”部分。

</Intro>

## 规则详情 {/*rule-details*/}

Refs 保存的是不会用于渲染的值。与 state 不同，修改 ref 不会触发重新渲染。在渲染期间读取或写入 `ref.current` 会破坏 React 的预期。当你尝试读取 ref 时，它们可能尚未初始化，而且其值可能是过时或不一致的。

## 它如何检测 Refs {/*how-it-detects-refs*/}

该 lint 只会将这些规则应用到它已知是 ref 的值。编译器在看到以下任一模式时，会推断某个值是 ref：

- 从 `useRef()` 或 `React.createRef()` 返回。

  ```js
  const scrollRef = useRef(null);
  ```

- 名称为 `ref` 或以 `Ref` 结尾，并且读取或写入 `.current` 的标识符。

  ```js
  buttonRef.current = node;
  ```

- 通过 JSX 的 `ref` 属性传递（例如 `<div ref={someRef} />`）。

  ```jsx
  <input ref={inputRef} />
  ```

一旦某个东西被标记为 ref，这种推断就会沿着赋值、解构或辅助调用继续跟随这个值。这使得即使 `ref.current` 是在另一个接收该 ref 作为参数的函数内部被访问，lint 也能指出违规。

## 常见违规 {/*common-violations*/}

- 在渲染期间读取 `ref.current`
- 在渲染期间更新 `refs`
- 将 `refs` 用于本应使用 state 的值

### 无效 {/*invalid*/}

此规则的错误代码示例：

```js
// ❌ 在渲染期间读取 ref
function Component() {
  const ref = useRef(0);
  const value = ref.current; // 不要在渲染期间读取
  return <div>{value}</div>;
}

// ❌ 在渲染期间修改 ref
function Component({value}) {
  const ref = useRef(null);
  ref.current = value; // 不要在渲染期间修改
  return <div />;
}
```

### 有效 {/*valid*/}

此规则的正确代码示例：

```js
// ✅ 在 effects/handlers 中读取 ref
function Component() {
  const ref = useRef(null);

  useEffect(() => {
    if (ref.current) {
      console.log(ref.current.offsetWidth); // 在 effect 中是可以的
    }
  });

  return <div ref={ref} />;
}

// ✅ 为 UI 值使用 state
function Component() {
  const [count, setCount] = useState(0);

  return (
    <button onClick={() => setCount(count + 1)}>
      {count}
    </button>
  );
}

// ✅ ref 值的惰性初始化
function Component() {
  const ref = useRef(null);

  // 仅在第一次使用时初始化一次
  if (ref.current === null) {
    ref.current = expensiveComputation(); // 可以 - 惰性初始化
  }

  const handleClick = () => {
    console.log(ref.current); // 使用已初始化的值
  };

  return <button onClick={handleClick}>Click</button>;
}
```

## 故障排查 {/*troubleshooting*/}

### 该 lint 将我的普通对象与 `.current` 标记为违规 {/*plain-object-current*/}

名称启发式会有意将 `ref.current` 和 `fooRef.current` 视为真实的 refs。如果你在建模一个自定义容器对象，请换一个不同的名称（例如 `box`），或者把可变值移入 state。重命名可以避开 lint，因为编译器会停止将其推断为 ref。
