---
title: component-hook-factories
---

<Intro>

会对定义嵌套组件或 Hook 的高阶函数进行校验。组件和 Hook 应该定义在模块级别。

</Intro>

## Rule Details {/*rule-details*/}

在其他函数内部定义组件或 Hook 会在每次调用时创建新的实例。React 会将每个实例视为完全不同的组件，销毁并重新创建整个组件树，丢失所有状态，并导致性能问题。

### Invalid {/*invalid*/}

此规则的错误代码示例：

```js {expectedErrors: {'react-compiler': [14]}}
// ❌ 创建组件的工厂函数
function createComponent(defaultValue) {
  return function Component() {
    // ...
  };
}

// ❌ 定义在组件内部的组件
function Parent() {
  function Child() {
    // ...
  }

  return <Child />;
}

// ❌ Hook 工厂函数
function createCustomHook(endpoint) {
  return function useData() {
    // ...
  };
}
```

### Valid {/*valid*/}

此规则的正确代码示例：

```js
// ✅ 定义在模块级别的组件
function Component({ defaultValue }) {
  // ...
}

// ✅ 定义在模块级别的自定义 Hook
function useData(endpoint) {
  // ...
}
```

## Troubleshooting {/*troubleshooting*/}

### I need dynamic component behavior {/*dynamic-behavior*/}

你可能会认为需要使用工厂来创建定制化组件：

```js
// ❌ 错误：工厂模式
function makeButton(color) {
  return function Button({children}) {
    return (
      <button style={{backgroundColor: color}}>
        {children}
      </button>
    );
  };
}

const RedButton = makeButton('red');
const BlueButton = makeButton('blue');
```

改为传递 [JSX 作为 children](/learn/passing-props-to-a-component#passing-jsx-as-children)：

```js
// ✅ 更好的方式：将 JSX 作为 children 传递
function Button({color, children}) {
  return (
    <button style={{backgroundColor: color}}>
      {children}
    </button>
  );
}

function App() {
  return (
    <>
      <Button color="red">Red</Button>
      <Button color="blue">Blue</Button>
    </>
  );
}
```
