---
title: 未知属性警告
---

当你尝试渲染一个带有 React 未识别为合法 DOM 属性/属性值的 prop 的 DOM 元素时，就会触发 unknown-prop 警告。你应确保你的 DOM 元素上没有多余的 props 混在其中。

出现此警告的可能原因有以下几种：

1. 你是否在使用 `{...props}` 或 `cloneElement(element, props)`？当把 props 复制给子组件时，你应确保没有意外地把只应传给父组件的 props 继续向下传递。下面可以看到这个问题的常见修复方式。

2. 你正在原生 DOM 节点上使用非标准的 DOM 属性，可能是为了表示自定义数据。如果你想把自定义数据附加到标准 DOM 元素上，可以考虑使用 MDN 上所述的自定义数据属性：[on MDN](https://developer.mozilla.org/en-US/docs/Web/Guide/HTML/Using_data_attributes)。

3. React 还不能识别你指定的属性。这很可能会在未来版本的 React 中得到修复。如果你将属性名写成小写，React 将允许你在没有警告的情况下传递它。

4. 你正在使用一个首字母不是大写的 React 组件，例如 `<myButton />`。React 会将其解释为 DOM 标签，因为 React JSX 转换使用大小写约定来区分用户自定义组件和 DOM 标签。对于你自己的 React 组件，请使用 PascalCase。例如，写 `<MyButton />`，而不是 `<myButton />`。

---

如果你因为传递了像 `{...props}` 这样的 props 而收到此警告，那么你的父组件需要“消耗掉”任何本应只用于父组件、而不应传给子组件的 prop。示例：

**错误：** 意外的 `layout` prop 被转发到了 `div` 标签。

```js
function MyDiv(props) {
  if (props.layout === 'horizontal') {
    // 错误！因为你很确定 "layout" 不是 <div> 能理解的 prop。
    return <div {...props} style={getHorizontalStyle()} />
  } else {
    // 错误！因为你很确定 "layout" 不是 <div> 能理解的 prop。
    return <div {...props} style={getVerticalStyle()} />
  }
}
```

**正确：** 可以使用展开语法从 props 中取出变量，并把剩余的 props 放入一个变量中。

```js
function MyDiv(props) {
  const { layout, ...rest } = props
  if (layout === 'horizontal') {
    return <div {...rest} style={getHorizontalStyle()} />
  } else {
    return <div {...rest} style={getVerticalStyle()} />
  }
}
```

**正确：** 你也可以将 props 赋给一个新对象，并从新对象中删除你正在使用的键。注意不要从原始的 `this.props` 对象中删除 props，因为该对象应被视为不可变的。

```js
function MyDiv(props) {
  const divProps = Object.assign({}, props);
  delete divProps.layout;

  if (props.layout === 'horizontal') {
    return <div {...divProps} style={getHorizontalStyle()} />
  } else {
    return <div {...divProps} style={getVerticalStyle()} />
  }
}
```
