---
title: addTransitionType
version: canary
---

<Canary>

**`addTransitionType` API 目前仅在 React 的 Canary 和 Experimental 渠道中可用。**

[在此了解更多关于 React 发布渠道的信息。](/community/versioning-policy#all-release-channels)

</Canary>

<Intro>

`addTransitionType` 让你可以指定一个过渡的原因。


```js
startTransition(() => {
  addTransitionType('my-transition-type');
  setState(newState);
});
```

</Intro>

<InlineToc />

---

## 参考 {/*reference*/}

### `addTransitionType` {/*addtransitiontype*/}

#### 参数 {/*parameters*/}

- `type`：要添加的过渡类型。可以是任意字符串。

#### 返回值 {/*returns*/}

`addTransitionType` 不返回任何内容。

#### 注意事项 {/*caveats*/}

- 如果组合了多个过渡，所有 Transition Types 都会被收集。你也可以向一个 Transition 添加多个类型。
- Transition Types 会在每次提交后重置。这意味着 `<Suspense>` fallback 会关联到 `startTransition` 之后的类型，但显示内容本身不会。

---

## 用法 {/*usage*/}

### 添加过渡的原因 {/*adding-the-cause-of-a-transition*/}

在 `startTransition` 内部调用 `addTransitionType`，以表明过渡的原因：

``` [[1, 6, "addTransitionType"], [2, 5, "startTransition", [3, 6, "'submit-click'"]]
import { startTransition, addTransitionType } from 'react';

function Submit({action) {
  function handleClick() {
    startTransition(() => {
      addTransitionType('submit-click');
      action();
    });
  }

  return <button onClick={handleClick}>Click me</button>;
}

```

当你在 <CodeStep step={2}>startTransition</CodeStep> 的作用域内调用 <CodeStep step={1}>addTransitionType</CodeStep> 时，React 会将 <CodeStep step={3}>submit-click</CodeStep> 关联为该 Transition 的原因之一。

目前，Transition Types 可用于根据导致 Transition 的原因来自定义不同的动画。你可以通过以下三种方式来使用它们：

- [使用浏览器 view transition types 自定义动画](#customize-animations-using-browser-view-transition-types)
- [使用 `View Transition` Class 自定义动画](#customize-animations-using-view-transition-class)
- [使用 `ViewTransition` 事件自定义动画](#customize-animations-using-viewtransition-events)

未来，我们计划支持更多使用 Transition 原因的场景。

---
### 使用浏览器 view transition types 自定义动画 {/*customize-animations-using-browser-view-transition-types*/}

当一个 [`ViewTransition`](/reference/react/ViewTransition) 由某个 transition 激活时，React 会把所有 Transition Types 作为浏览器的 [view transition types](https://www.w3.org/TR/css-view-transitions-2/#active-view-transition-pseudo-examples) 添加到该元素上。

这使你可以基于 CSS 作用域来自定义不同的动画：

```js [11]
function Component() {
  return (
    <ViewTransition>
      <div>Hello</div>
    </ViewTransition>
  );
}

startTransition(() => {
  addTransitionType('my-transition-type');
  setShow(true);
});
```

```css
:root:active-view-transition-type(my-transition-type) {
  &::view-transition-...(...) {
    ...
  }
}
```

---

### 使用 `View Transition` Class 自定义动画 {/*customize-animations-using-view-transition-class*/}

你可以通过向 View Transition Class 传入一个对象，基于类型来自定义已激活的 `ViewTransition` 动画：

```js
function Component() {
  return (
    <ViewTransition enter={{
      'my-transition-type': 'my-transition-class',
    }}>
      <div>Hello</div>
    </ViewTransition>
  );
}

// ...
startTransition(() => {
  addTransitionType('my-transition-type');
  setState(newState);
});
```

如果有多个类型匹配，那么它们会组合起来。如果没有任何类型匹配，则会改用特殊的 "default" 条目。如果某个类型的值为 "none"，那么它会优先生效，并且 `ViewTransition` 将被禁用（不会分配名称）。

这些可以与 enter/exit/update/layout/share props 组合使用，以便根据触发类型和 Transition Type 进行匹配。

```js
<ViewTransition enter={{
  'navigation-back': 'enter-right',
  'navigation-forward': 'enter-left',
}}
exit={{
  'navigation-back': 'exit-right',
  'navigation-forward': 'exit-left',
}}>
```

---

### 使用 `ViewTransition` 事件自定义动画 {/*customize-animations-using-viewtransition-events*/}

你可以使用 View Transition 事件，基于类型以命令式方式自定义已激活的 `ViewTransition` 动画：

```
<ViewTransition onUpdate={(inst, types) => {
  if (types.includes('navigation-back')) {
    ...
  } else if (types.includes('navigation-forward')) {
    ...
  } else {
    ...
  }
}}>
```

这使你可以根据原因选择不同的命令式动画。
