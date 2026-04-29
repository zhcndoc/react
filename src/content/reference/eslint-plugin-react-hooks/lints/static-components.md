---
title: static-components
---

<Intro>

验证组件是静态的，而不是在每次渲染时都重新创建。动态重新创建的组件可能会重置状态并触发过多的重新渲染。

</Intro>

## 规则详情 {/*rule-details*/}

在其他组件内部定义的组件会在每次渲染时重新创建。React 会将它们视为全新的组件类型，卸载旧组件并挂载新组件，在此过程中销毁所有状态和 DOM 节点。

### 无效 {/*invalid*/}

此规则的错误代码示例：

```js
// ❌ 在组件内部定义组件
function Parent() {
  const ChildComponent = () => { // 每次渲染都是新组件！
    const [count, setCount] = useState(0);
    return <button onClick={() => setCount(count + 1)}>{count}</button>;
  };

  return <ChildComponent />; // 状态会在每次渲染时重置
}

// ❌ 动态创建组件
function Parent({type}) {
  const Component = type === 'button'
    ? () => <button>Click</button>
    : () => <div>Text</div>;

  return <Component />;
}
```

### 有效 {/*valid*/}

此规则的正确代码示例：

```js
// ✅ 组件位于模块级别
const ButtonComponent = () => <button>Click</button>;
const TextComponent = () => <div>Text</div>;

function Parent({type}) {
  const Component = type === 'button'
    ? ButtonComponent  // 引用现有组件
    : TextComponent;

  return <Component />;
}
```

## 故障排查 {/*troubleshooting*/}

### 我需要有条件地渲染不同的组件 {/*conditional-components*/}

你可能会在内部定义组件以访问本地状态：

```js {expectedErrors: {'react-compiler': [13]}}
// ❌ 错误：使用内部组件访问父组件状态
function Parent() {
  const [theme, setTheme] = useState('light');

  function ThemedButton() { // 每次渲染都会重新创建！
    return (
      <button className={theme}>
        点击我
      </button>
    );
  }

  return <ThemedButton />;
}
```

改为将数据作为 props 传递：

```js
// ✅ 更好：将 props 传递给静态组件
function ThemedButton({theme}) {
  return (
    <button className={theme}>
      点击我
    </button>
  );
}

function Parent() {
  const [theme, setTheme] = useState('light');
  return <ThemedButton theme={theme} />;
}
```

<Note>

如果你发现自己想在其他组件内部定义组件来访问本地变量，这表明你应该改为传递 props。这样可以让组件更易复用和测试。

</Note>