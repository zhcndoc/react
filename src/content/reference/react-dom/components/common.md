---
title: "常见组件（例如 <div>）"
---

<Intro>

所有内置浏览器组件，例如 [`<div>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/div)，都支持一些通用的属性和事件。

</Intro>

<InlineToc />

---

## 参考 {/*reference*/}

* `style`：一个包含 CSS 样式的对象，例如 `{ fontWeight: 'bold', margin: 20 }`。与 DOM 的 [`style`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/style) 属性类似，CSS 属性名需要使用 `camelCase` 形式，例如使用 `fontWeight` 而不是 `font-weight`。你可以将字符串或数字作为值传入。如果传入数字，例如 `width: 100`，React 会自动在值后追加 `px`（“像素”），除非它是[无单位属性。](https://github.com/react/react/blob/81d4ee9ca5c405dce62f64e61506b8e155f38d8d/packages/react-dom-bindings/src/shared/CSSProperty.js#L8-L57)我们建议仅在无法提前确定样式值的动态样式中使用 `style`。在其他情况下，使用 `className` 应用普通 CSS 类会更高效。[阅读更多关于 `className` 和 `style` 的内容。](#applying-css-styles)

* `suppressHydrationWarning`：一个布尔值。如果你使用 [服务端渲染，](/reference/react-dom/server)通常当服务端和客户端渲染出不同内容时会出现警告。在某些罕见情况下（如时间戳），要保证完全一致非常困难甚至不可能。如果将 `suppressHydrationWarning` 设为 `true`，React 将不会就该元素的属性和内容不匹配发出警告。它只作用于一层深度，并且 предназначляется 作为一种逃生出口。不要过度使用它。[阅读关于抑制 hydration 错误的内容。](/reference/react-dom/client/hydrateRoot#suppressing-unavoidable-hydration-mismatch-errors)

### `ref` 回调函数 {/*ref-callback*/}

你可以向 `ref` 属性传入一个函数，而不是 ref 对象（例如 [`useRef`](/reference/react/useRef#manipulating-the-dom-with-a-ref) 返回的那个）。

```js
<div ref={(node) => {
  console.log('Attached', node);

  return () => {
    console.log('Clean up', node)
  }
}}>
```

[查看一个使用 `ref` 回调的示例。](/learn/manipulating-the-dom-with-refs#how-to-manage-a-list-of-refs-using-a-ref-callback)

当 `<div>` DOM 节点被添加到屏幕上时，React 会用 DOM `node` 作为参数调用你的 `ref` 回调。当该 `<div>` DOM 节点被移除时，React 会调用回调返回的清理函数。

只要你传入的是一个*不同的* `ref` 回调，React 也会调用你的 `ref` 回调。在上面的示例中，`(node) => { ... }` 在每次渲染时都是不同的函数。当你的组件重新渲染时，*之前的*函数会以 `null` 作为参数被调用，而*下一个*函数会以 DOM 节点作为参数被调用。

#### 参数 {/*ref-callback-parameters*/}

* `node`：一个 DOM 节点。当 ref 绑定时，React 会将 DOM 节点传给你。除非你在每次渲染时都为 `ref` 回调传入相同的函数引用，否则该回调会在组件的每次重新渲染期间临时执行清理并重新创建。

<Note>

#### React 19 为 `ref` 回调添加了清理函数。 {/*react-19-added-cleanup-functions-for-ref-callbacks*/}

为兼容旧版本，如果 `ref` 回调没有返回清理函数，那么当 `ref` 被解绑时，`node` 会以 `null` 被调用。此行为将在未来版本中移除。

</Note>

#### 返回值 {/*returns*/}

* **可选** `清理函数`：当 `ref` 被解绑时，React 会调用清理函数。如果 `ref` 回调没有返回函数，那么当 `ref` 被解绑时，React 会再次以 `null` 作为参数调用该回调。此行为将在未来版本中移除。

#### 注意事项 {/*caveats*/}

* 当启用 Strict Mode 时，React 会在第一次真实的设置之前，**额外执行一次仅开发环境可见的设置+清理循环**。这是一个压力测试，用于确保你的清理逻辑能“镜像”你的设置逻辑，并停止或撤销设置所做的一切。如果这造成问题，请实现清理函数。
* 当你传入一个*不同的* `ref` 回调时，如果提供了清理函数，React 会调用*之前*回调的清理函数。如果没有定义清理函数，`ref` 回调将以 `null` 作为参数被调用。*下一个*函数将以 DOM 节点作为参数被调用。

---

### React 事件对象 {/*react-event-object*/}

你的事件处理函数将接收一个 *React 事件对象*。它有时也被称为“合成事件”。

```js
<button onClick={e => {
  console.log(e); // React 事件对象
}} />
```

它遵循与底层 DOM 事件相同的标准，但修复了一些浏览器不一致问题。

某些 React 事件不会直接映射到浏览器的原生事件。例如在 `onMouseLeave` 中，`e.nativeEvent` 会指向一个 `mouseout` 事件。具体映射并不是公共 API 的一部分，未来可能会改变。如果你出于某些原因需要底层浏览器事件，请从 `e.nativeEvent` 读取。

#### 属性 {/*react-event-object-properties*/}

React 事件对象实现了部分标准 [`Event`](https://developer.mozilla.org/en-US/docs/Web/API/Event) 属性：

* [`bubbles`](https://developer.mozilla.org/en-US/docs/Web/API/Event/bubbles)：一个布尔值。返回事件是否会通过 DOM 冒泡。
* [`cancelable`](https://developer.mozilla.org/en-US/docs/Web/API/Event/cancelable)：一个布尔值。返回事件是否可被取消。
* [`currentTarget`](https://developer.mozilla.org/en-US/docs/Web/API/Event/currentTarget)：一个 DOM 节点。返回当前处理函数在 React 树中所附着的节点。
* [`defaultPrevented`](https://developer.mozilla.org/en-US/docs/Web/API/Event/defaultPrevented)：一个布尔值。返回是否调用了 `preventDefault`。
* [`eventPhase`](https://developer.mozilla.org/en-US/docs/Web/API/Event/eventPhase)：一个数字。返回事件当前所处的阶段。
* [`isTrusted`](https://developer.mozilla.org/en-US/docs/Web/API/Event/isTrusted)：一个布尔值。返回事件是否由用户发起。
* [`target`](https://developer.mozilla.org/en-US/docs/Web/API/Event/target)：一个 DOM 节点。返回事件发生的节点（也可能是较远的子节点）。
* [`timeStamp`](https://developer.mozilla.org/en-US/docs/Web/API/Event/timeStamp)：一个数字。返回事件发生的时间。

此外，React 事件对象还提供这些属性：

* `nativeEvent`：一个 DOM [`Event`](https://developer.mozilla.org/en-US/docs/Web/API/Event)。原始浏览器事件对象。

#### 方法 {/*react-event-object-methods*/}

React 事件对象实现了部分标准 [`Event`](https://developer.mozilla.org/en-US/docs/Web/API/Event) 方法：

* [`preventDefault()`](https://developer.mozilla.org/en-US/docs/Web/API/Event/preventDefault)：阻止该事件的默认浏览器行为。
* [`stopPropagation()`](https://developer.mozilla.org/en-US/docs/Web/API/Event/stopPropagation)：停止事件在 React 树中的传播。

此外，React 事件对象还提供这些方法：

* `isDefaultPrevented()`：返回一个布尔值，表示是否调用了 `preventDefault`。
* `isPropagationStopped()`：返回一个布尔值，表示是否调用了 `stopPropagation`。
* `persist()`：不用于 React DOM。对于 React Native，可调用它以在事件之后读取事件属性。
* `isPersistent()`：不用于 React DOM。对于 React Native，返回是否已调用 `persist`。

#### 注意事项 {/*react-event-object-caveats*/}

* `currentTarget`、`eventPhase`、`target` 和 `type` 的值反映的是你的 React 代码所期望的值。在底层，React 会在根节点附加事件处理函数，但这不会反映在 React 事件对象中。例如，`e.currentTarget` 可能与底层的 `e.nativeEvent.currentTarget` 不同。对于被 polyfill 的事件，`e.type`（React 事件类型）可能与 `e.nativeEvent.type`（底层类型）不同。

---

### `AnimationEvent` 处理函数 {/*animationevent-handler*/}

用于 [CSS 动画](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Animations/Using_CSS_animations) 事件的事件处理函数类型。

```js
<div
  onAnimationStart={e => console.log('onAnimationStart')}
  onAnimationIteration={e => console.log('onAnimationIteration')}
  onAnimationEnd={e => console.log('onAnimationEnd')}
/>
```

#### 参数 {/*animationevent-handler-parameters*/}

* `e`：一个带有以下额外 [`AnimationEvent`](https://developer.mozilla.org/en-US/docs/Web/API/AnimationEvent) 属性的 [React 事件对象](#react-event-object)：
  * [`animationName`](https://developer.mozilla.org/en-US/docs/Web/API/AnimationEvent/animationName)
  * [`elapsedTime`](https://developer.mozilla.org/en-US/docs/Web/API/AnimationEvent/elapsedTime)
  * [`pseudoElement`](https://developer.mozilla.org/en-US/docs/Web/API/AnimationEvent/pseudoElement)

---

### `ClipboardEvent` 处理函数 {/*clipboadevent-handler*/}

用于 [Clipboard API](https://developer.mozilla.org/en-US/docs/Web/API/Clipboard_API) 事件的事件处理函数类型。

```js
<input
  onCopy={e => console.log('onCopy')}
  onCut={e => console.log('onCut')}
  onPaste={e => console.log('onPaste')}
/>
```

#### 参数 {/*clipboadevent-handler-parameters*/}

* `e`：一个带有以下额外 [`ClipboardEvent`](https://developer.mozilla.org/en-US/docs/Web/API/ClipboardEvent) 属性的 [React 事件对象](#react-event-object)：

  * [`clipboardData`](https://developer.mozilla.org/en-US/docs/Web/API/ClipboardEvent/clipboardData)

---

### `CompositionEvent` 处理函数 {/*compositionevent-handler*/}

用于 [输入法编辑器（IME）](https://developer.mozilla.org/en-US/docs/Glossary/Input_method_editor) 事件的事件处理函数类型。

```js
<input
  onCompositionStart={e => console.log('onCompositionStart')}
  onCompositionUpdate={e => console.log('onCompositionUpdate')}
  onCompositionEnd={e => console.log('onCompositionEnd')}
/>
```

#### 参数 {/*compositionevent-handler-parameters*/}

* `e`：一个带有以下额外 [`CompositionEvent`](https://developer.mozilla.org/en-US/docs/Web/API/CompositionEvent) 属性的 [React 事件对象](#react-event-object)：
  * [`data`](https://developer.mozilla.org/en-US/docs/Web/API/CompositionEvent/data)

---

### `DragEvent` 处理函数 {/*dragevent-handler*/}

用于 [HTML 拖放 API](https://developer.mozilla.org/en-US/docs/Web/API/HTML_Drag_and_Drop_API) 事件的事件处理函数类型。

```js
<>
  <div
    draggable={true}
    onDragStart={e => console.log('onDragStart')}
    onDragEnd={e => console.log('onDragEnd')}
  >
    拖拽源
  </div>

  <div
    onDragEnter={e => console.log('onDragEnter')}
    onDragLeave={e => console.log('onDragLeave')}
    onDragOver={e => { e.preventDefault(); console.log('onDragOver'); }}
    onDrop={e => console.log('onDrop')}
  >
    放置目标
  </div>
</>
```

#### 参数 {/*dragevent-handler-parameters*/}

* `e`：一个带有以下额外 [`DragEvent`](https://developer.mozilla.org/en-US/docs/Web/API/DragEvent) 属性的 [React 事件对象](#react-event-object)：
  * [`dataTransfer`](https://developer.mozilla.org/en-US/docs/Web/API/DragEvent/dataTransfer)

  它还包括继承自 [`MouseEvent`](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent) 的属性：

  * [`altKey`](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent/altKey)
  * [`button`](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent/button)
  * [`buttons`](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent/buttons)
  * [`ctrlKey`](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent/ctrlKey)
  * [`clientX`](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent/clientX)
  * [`clientY`](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent/clientY)
  * [`getModifierState(key)`](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent/getModifierState)
  * [`metaKey`](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent/metaKey)
  * [`movementX`](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent/movementX)
  * [`movementY`](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent/movementY)
  * [`pageX`](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent/pageX)
  * [`pageY`](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent/pageY)
  * [`relatedTarget`](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent/relatedTarget)
  * [`screenX`](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent/screenX)
  * [`screenY`](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent/screenY)
  * [`shiftKey`](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent/shiftKey)

  它还包括继承自 [`UIEvent`](https://developer.mozilla.org/en-US/docs/Web/API/UIEvent) 的属性：

  * [`detail`](https://developer.mozilla.org/en-US/docs/Web/API/UIEvent/detail)
  * [`view`](https://developer.mozilla.org/en-US/docs/Web/API/UIEvent/view)

---

### `FocusEvent` 处理函数 {/*focusevent-handler*/}

用于焦点事件的事件处理函数类型。

```js
<input
  onFocus={e => console.log('onFocus')}
  onBlur={e => console.log('onBlur')}
/>
```

[查看一个示例。](#handling-focus-events)

#### 参数 {/*focusevent-handler-parameters*/}

* `e`：一个 [React 事件对象](#react-event-object)，带有以下额外的 [`FocusEvent`](https://developer.mozilla.org/en-US/docs/Web/API/FocusEvent) 属性：
  * [`relatedTarget`](https://developer.mozilla.org/en-US/docs/Web/API/FocusEvent/relatedTarget)

  它还包含继承自 [`UIEvent`](https://developer.mozilla.org/en-US/docs/Web/API/UIEvent) 的属性：

  * [`detail`](https://developer.mozilla.org/en-US/docs/Web/API/UIEvent/detail)
  * [`view`](https://developer.mozilla.org/en-US/docs/Web/API/UIEvent/view)

---

### `Event` 处理函数 {/*event-handler*/}

一种用于通用事件的事件处理函数类型。

#### 参数 {/*event-handler-parameters*/}

* `e`：一个 [React 事件对象](#react-event-object)，没有额外属性。

---

### `InputEvent` 处理函数 {/*inputevent-handler*/}

一种用于 `onBeforeInput` 事件的事件处理函数类型。

```js
<input onBeforeInput={e => console.log('onBeforeInput')} />
```

#### 参数 {/*inputevent-handler-parameters*/}

* `e`：一个 [React 事件对象](#react-event-object)，带有以下额外的 [`InputEvent`](https://developer.mozilla.org/en-US/docs/Web/API/InputEvent) 属性：
  * [`data`](https://developer.mozilla.org/en-US/docs/Web/API/InputEvent/data)

---

### `KeyboardEvent` 处理函数 {/*keyboardevent-handler*/}

一种用于键盘事件的事件处理函数类型。

```js
<input
  onKeyDown={e => console.log('onKeyDown')}
  onKeyUp={e => console.log('onKeyUp')}
/>
```

[查看示例。](#handling-keyboard-events)

#### 参数 {/*keyboardevent-handler-parameters*/}

* `e`：一个 [React 事件对象](#react-event-object)，带有以下额外的 [`KeyboardEvent`](https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent) 属性：
  * [`altKey`](https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent/altKey)
  * [`charCode`](https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent/charCode)
  * [`code`](https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent/code)
  * [`ctrlKey`](https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent/ctrlKey)
  * [`getModifierState(key)`](https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent/getModifierState)
  * [`key`](https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent/key)
  * [`keyCode`](https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent/keyCode)
  * [`locale`](https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent/locale)
  * [`metaKey`](https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent/metaKey)
  * [`location`](https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent/location)
  * [`repeat`](https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent/repeat)
  * [`shiftKey`](https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent/shiftKey)
  * [`which`](https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent/which)

  它还包含继承自 [`UIEvent`](https://developer.mozilla.org/en-US/docs/Web/API/UIEvent) 的属性：

  * [`detail`](https://developer.mozilla.org/en-US/docs/Web/API/UIEvent/detail)
  * [`view`](https://developer.mozilla.org/en-US/docs/Web/API/UIEvent/view)

---

### `MouseEvent` 处理函数 {/*mouseevent-handler*/}

一种用于鼠标事件的事件处理函数类型。

```js
<div
  onClick={e => console.log('onClick')}
  onMouseEnter={e => console.log('onMouseEnter')}
  onMouseOver={e => console.log('onMouseOver')}
  onMouseDown={e => console.log('onMouseDown')}
  onMouseUp={e => console.log('onMouseUp')}
  onMouseLeave={e => console.log('onMouseLeave')}
/>
```

[查看示例。](#handling-mouse-events)

#### 参数 {/*mouseevent-handler-parameters*/}

* `e`：一个 [React 事件对象](#react-event-object)，带有以下额外的 [`MouseEvent`](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent) 属性：
  * [`altKey`](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent/altKey)
  * [`button`](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent/button)
  * [`buttons`](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent/buttons)
  * [`ctrlKey`](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent/ctrlKey)
  * [`clientX`](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent/clientX)
  * [`clientY`](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent/clientY)
  * [`getModifierState(key)`](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent/getModifierState)
  * [`metaKey`](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent/metaKey)
  * [`movementX`](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent/movementX)
  * [`movementY`](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent/movementY)
  * [`pageX`](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent/pageX)
  * [`pageY`](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent/pageY)
  * [`relatedTarget`](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent/relatedTarget)
  * [`screenX`](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent/screenX)
  * [`screenY`](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent/screenY)
  * [`shiftKey`](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent/shiftKey)

  它还包含继承自 [`UIEvent`](https://developer.mozilla.org/en-US/docs/Web/API/UIEvent) 的属性：

  * [`detail`](https://developer.mozilla.org/en-US/docs/Web/API/UIEvent/detail)
  * [`view`](https://developer.mozilla.org/en-US/docs/Web/API/UIEvent/view)

---

### `PointerEvent` 处理函数 {/*pointerevent-handler*/}

一种用于 [指针事件](https://developer.mozilla.org/en-US/docs/Web/API/Pointer_events) 的事件处理函数类型。

```js
<div
  onPointerEnter={e => console.log('onPointerEnter')}
  onPointerMove={e => console.log('onPointerMove')}
  onPointerDown={e => console.log('onPointerDown')}
  onPointerUp={e => console.log('onPointerUp')}
  onPointerLeave={e => console.log('onPointerLeave')}
/>
```

[查看示例。](#handling-pointer-events)

#### 参数 {/*pointerevent-handler-parameters*/}

* `e`：一个 [React 事件对象](#react-event-object)，带有以下额外的 [`PointerEvent`](https://developer.mozilla.org/en-US/docs/Web/API/PointerEvent) 属性：
  * [`height`](https://developer.mozilla.org/en-US/docs/Web/API/PointerEvent/height)
  * [`isPrimary`](https://developer.mozilla.org/en-US/docs/Web/API/PointerEvent/isPrimary)
  * [`pointerId`](https://developer.mozilla.org/en-US/docs/Web/API/PointerEvent/pointerId)
  * [`pointerType`](https://developer.mozilla.org/en-US/docs/Web/API/PointerEvent/pointerType)
  * [`pressure`](https://developer.mozilla.org/en-US/docs/Web/API/PointerEvent/pressure)
  * [`tangentialPressure`](https://developer.mozilla.org/en-US/docs/Web/API/PointerEvent/tangentialPressure)
  * [`tiltX`](https://developer.mozilla.org/en-US/docs/Web/API/PointerEvent/tiltX)
  * [`tiltY`](https://developer.mozilla.org/en-US/docs/Web/API/PointerEvent/tiltY)
  * [`twist`](https://developer.mozilla.org/en-US/docs/Web/API/PointerEvent/twist)
  * [`width`](https://developer.mozilla.org/en-US/docs/Web/API/PointerEvent/width)

  它还包含继承自 [`MouseEvent`](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent) 的属性：

  * [`altKey`](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent/altKey)
  * [`button`](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent/button)
  * [`buttons`](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent/buttons)
  * [`ctrlKey`](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent/ctrlKey)
  * [`clientX`](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent/clientX)
  * [`clientY`](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent/clientY)
  * [`getModifierState(key)`](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent/getModifierState)
  * [`metaKey`](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent/metaKey)
  * [`movementX`](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent/movementX)
  * [`movementY`](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent/movementY)
  * [`pageX`](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent/pageX)
  * [`pageY`](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent/pageY)
  * [`relatedTarget`](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent/relatedTarget)
  * [`screenX`](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent/screenX)
  * [`screenY`](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent/screenY)
  * [`shiftKey`](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent/shiftKey)

  它还包含继承自 [`UIEvent`](https://developer.mozilla.org/en-US/docs/Web/API/UIEvent) 的属性：

  * [`detail`](https://developer.mozilla.org/en-US/docs/Web/API/UIEvent/detail)
  * [`view`](https://developer.mozilla.org/en-US/docs/Web/API/UIEvent/view)

---

### `TouchEvent` 处理函数 {/*touchevent-handler*/}

一种用于 [触摸事件](https://developer.mozilla.org/en-US/docs/Web/API/Touch_events) 的事件处理函数类型。

```js
<div
  onTouchStart={e => console.log('onTouchStart')}
  onTouchMove={e => console.log('onTouchMove')}
  onTouchEnd={e => console.log('onTouchEnd')}
  onTouchCancel={e => console.log('onTouchCancel')}
/>
```

#### 参数 {/*touchevent-handler-parameters*/}

* `e`：一个 [React 事件对象](#react-event-object)，带有以下额外的 [`TouchEvent`](https://developer.mozilla.org/en-US/docs/Web/API/TouchEvent) 属性：
  * [`altKey`](https://developer.mozilla.org/en-US/docs/Web/API/TouchEvent/altKey)
  * [`ctrlKey`](https://developer.mozilla.org/en-US/docs/Web/API/TouchEvent/ctrlKey)
  * [`changedTouches`](https://developer.mozilla.org/en-US/docs/Web/API/TouchEvent/changedTouches)
  * [`getModifierState(key)`](https://developer.mozilla.org/en-US/docs/Web/API/TouchEvent/getModifierState)
  * [`metaKey`](https://developer.mozilla.org/en-US/docs/Web/API/TouchEvent/metaKey)
  * [`shiftKey`](https://developer.mozilla.org/en-US/docs/Web/API/TouchEvent/shiftKey)
  * [`touches`](https://developer.mozilla.org/en-US/docs/Web/API/TouchEvent/touches)
  * [`targetTouches`](https://developer.mozilla.org/en-US/docs/Web/API/TouchEvent/targetTouches)

  它还包含继承自 [`UIEvent`](https://developer.mozilla.org/en-US/docs/Web/API/UIEvent) 的属性：

  * [`detail`](https://developer.mozilla.org/en-US/docs/Web/API/UIEvent/detail)
  * [`view`](https://developer.mozilla.org/en-US/docs/Web/API/UIEvent/view)

---

### `TransitionEvent` 处理函数 {/*transitionevent-handler*/}

一种用于 CSS 过渡事件的事件处理函数类型。

```js
<div
  onTransitionEnd={e => console.log('onTransitionEnd')}
/>
```

#### 参数 {/*transitionevent-handler-parameters*/}

* `e`：一个 [React 事件对象](#react-event-object)，带有以下额外的 [`TransitionEvent`](https://developer.mozilla.org/en-US/docs/Web/API/TransitionEvent) 属性：
  * [`elapsedTime`](https://developer.mozilla.org/en-US/docs/Web/API/TransitionEvent/elapsedTime)
  * [`propertyName`](https://developer.mozilla.org/en-US/docs/Web/API/TransitionEvent/propertyName)
  * [`pseudoElement`](https://developer.mozilla.org/en-US/docs/Web/API/TransitionEvent/pseudoElement)

---

### `UIEvent` 处理函数 {/*uievent-handler*/}

一种用于通用 UI 事件的事件处理函数类型。

```js
<div
  onScroll={e => console.log('onScroll')}
/>
```

#### 参数 {/*uievent-handler-parameters*/}

* `e`：一个 [React 事件对象](#react-event-object)，带有以下额外的 [`UIEvent`](https://developer.mozilla.org/en-US/docs/Web/API/UIEvent) 属性：
  * [`detail`](https://developer.mozilla.org/en-US/docs/Web/API/UIEvent/detail)
  * [`view`](https://developer.mozilla.org/en-US/docs/Web/API/UIEvent/view)

---

### `WheelEvent` 处理函数 {/*wheelevent-handler*/}

一种用于 `onWheel` 事件的事件处理函数类型。

```js
<div
  onWheel={e => console.log('onWheel')}
/>
```

#### 参数 {/*wheelevent-handler-parameters*/}

* `e`：一个 [React 事件对象](#react-event-object)，带有以下额外的 [`WheelEvent`](https://developer.mozilla.org/en-US/docs/Web/API/WheelEvent) 属性：
  * [`deltaMode`](https://developer.mozilla.org/en-US/docs/Web/API/WheelEvent/deltaMode)
  * [`deltaX`](https://developer.mozilla.org/en-US/docs/Web/API/WheelEvent/deltaX)
  * [`deltaY`](https://developer.mozilla.org/en-US/docs/Web/API/WheelEvent/deltaY)
  * [`deltaZ`](https://developer.mozilla.org/en-US/docs/Web/API/WheelEvent/deltaZ)


  它还包含继承自 [`MouseEvent`](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent) 的属性：

  * [`altKey`](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent/altKey)
  * [`button`](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent/button)
  * [`buttons`](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent/buttons)
  * [`ctrlKey`](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent/ctrlKey)
  * [`clientX`](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent/clientX)
  * [`clientY`](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent/clientY)
  * [`getModifierState(key)`](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent/getModifierState)
  * [`metaKey`](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent/metaKey)
  * [`movementX`](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent/movementX)
  * [`movementY`](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent/movementY)
  * [`pageX`](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent/pageX)
  * [`pageY`](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent/pageY)
  * [`relatedTarget`](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent/relatedTarget)
  * [`screenX`](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent/screenX)
  * [`screenY`](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent/screenY)
  * [`shiftKey`](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent/shiftKey)

  它还包含继承自 [`UIEvent`](https://developer.mozilla.org/en-US/docs/Web/API/UIEvent) 的属性：

  * [`detail`](https://developer.mozilla.org/en-US/docs/Web/API/UIEvent/detail)
  * [`view`](https://developer.mozilla.org/en-US/docs/Web/API/UIEvent/view)

---

## 用法 {/*usage*/}

### 应用 CSS 样式 {/*applying-css-styles*/}

在 React 中，你可以使用 [`className`.](https://developer.mozilla.org/en-US/docs/Web/API/Element/className) 指定一个 CSS 类。它的工作方式与 HTML 中的 `class` 属性类似：

```js
<img className="avatar" />
```

然后你在单独的 CSS 文件中为它编写 CSS 规则：

```css
/* 在你的 CSS 中 */
.avatar {
  border-radius: 50%;
}
```

React 并不规定你如何添加 CSS 文件。在最简单的情况下，你需要向 HTML 添加一个 [`<link>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/link) 标签。如果你使用构建工具或框架，请查阅其文档，了解如何将 CSS 文件添加到你的项目中。

有时，样式值依赖于数据。使用 `style` 属性来动态传入一些样式：

```js {3-6}
<img
  className="avatar"
  style={{
    width: user.imageSize,
    height: user.imageSize
  }}
/>
```


在上面的示例中，`style={{}}` 不是一种特殊语法，而是 `style={ }` [JSX 花括号。](/learn/javascript-in-jsx-with-curly-braces) 中的一个普通 `{}` 对象。我们建议只有在样式依赖于 JavaScript 变量时才使用 `style` 属性。

<Sandpack>

```js src/App.js
import Avatar from './Avatar.js';

const user = {
  name: 'Hedy Lamarr',
  imageUrl: 'https://react.dev/images/docs/scientists/yXOvdOSs.jpg',
  imageSize: 90,
};

export default function App() {
  return <Avatar user={user} />;
}
```

```js src/Avatar.js active
export default function Avatar({ user }) {
  return (
    <img
      src={user.imageUrl}
      alt={'照片：' + user.name}
      className="avatar"
      style={{
        width: user.imageSize,
        height: user.imageSize
      }}
    />
  );
}
```

```css src/styles.css
.avatar {
  border-radius: 50%;
}
```

</Sandpack>

<DeepDive>

#### 如何有条件地应用多个 CSS 类？ {/*how-to-apply-multiple-css-classes-conditionally*/}

要有条件地应用 CSS 类，你需要自己使用 JavaScript 生成 `className` 字符串。

例如，`className={'row ' + (isSelected ? 'selected': '')}` 会根据 `isSelected` 是否为 `true`，生成 `className="row"` 或 `className="row selected"`。

为了让它更易读，你可以使用一个轻量的辅助库，比如 [`classnames`:](https://github.com/JedWatson/classnames)

```js
import cn from 'classnames';

function Row({ isSelected }) {
  return (
    <div className={cn('row', isSelected && 'selected')}>
      ...
    </div>
  );
}
```

如果你有多个条件类，这会特别方便：

```js
import cn from 'classnames';

function Row({ isSelected, size }) {
  return (
    <div className={cn('row', {
      selected: isSelected,
      large: size === 'large',
      small: size === 'small',
    })}>
      ...
    </div>
  );
}
```

</DeepDive>

---

### 使用 ref 操作 DOM 节点 {/*manipulating-a-dom-node-with-a-ref*/}

有时，你需要获取与 JSX 中某个标签关联的浏览器 DOM 节点。例如，如果你想在按钮被点击时聚焦一个 `<input>`，你需要在浏览器的 `<input>` DOM 节点上调用 [`focus()`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/focus)。

要获取某个标签对应的浏览器 DOM 节点，[声明一个 ref](/reference/react/useRef) 并将其作为 `ref` 属性传给该标签：

```js {7}
import { useRef } from 'react';

export default function Form() {
  const inputRef = useRef(null);
  // ...
  return (
    <input ref={inputRef} />
    // ...
```

在渲染到屏幕后，React 会把 DOM 节点放入 `inputRef.current` 中。

<Sandpack>

```js
import { useRef } from 'react';

export default function Form() {
  const inputRef = useRef(null);

  function handleClick() {
    inputRef.current.focus();
  }

  return (
    <>
      <input ref={inputRef} />
      <button onClick={handleClick}>
        聚焦输入框
      </button>
    </>
  );
}
```

</Sandpack>

进一步了解 [使用 refs 操作 DOM](/learn/manipulating-the-dom-with-refs) 和 [查看更多示例。](/reference/react/useRef#usage)

对于更高级的用例，`ref` 属性也接受一个 [回调函数。](#ref-callback)

---

### 危险地设置 inner HTML {/*dangerously-setting-the-inner-html*/}

你可以像这样向元素传递一段原始 HTML 字符串：

```js
const markup = { __html: '<p>一些原始 html</p>' };
return <div dangerouslySetInnerHTML={markup} />;
```

**这是危险的。与底层 DOM 的 [`innerHTML`](https://developer.mozilla.org/en-US/docs/Web/API/Element/innerHTML) 属性一样，你必须格外小心！除非这段标记来自完全可信的来源，否则很容易通过这种方式引入 [XSS](https://en.wikipedia.org/wiki/Cross-site_scripting) 漏洞。**

例如，如果你使用一个将 Markdown 转换为 HTML 的 Markdown 库，你信任它的解析器没有 bug，并且用户只会看到自己的输入，那么你可以像这样显示生成的 HTML：

<Sandpack>

```js
import { useState } from 'react';
import MarkdownPreview from './MarkdownPreview.js';

export default function MarkdownEditor() {
  const [postContent, setPostContent] = useState('_Hello,_ **Markdown**!');
  return (
    <>
      <label>
        输入一些 markdown：
        <textarea
          value={postContent}
          onChange={e => setPostContent(e.target.value)}
        />
      </label>
      <hr />
      <MarkdownPreview markdown={postContent} />
    </>
  );
}
```

```js src/MarkdownPreview.js active
import { Remarkable } from 'remarkable';

const md = new Remarkable();

function renderMarkdownToHTML(markdown) {
  // 这只有在安全时才成立，因为输出的 HTML
  // 会展示给同一个用户，而且你
  // 信任这个 Markdown 解析器没有 bug。
  const renderedHTML = md.render(markdown);
  return {__html: renderedHTML};
}

export default function MarkdownPreview({ markdown }) {
  const markup = renderMarkdownToHTML(markdown);
  return <div dangerouslySetInnerHTML={markup} />;
}
```

```json package.json
{
  "dependencies": {
    "react": "latest",
    "react-dom": "latest",
    "react-scripts": "latest",
    "remarkable": "2.0.1"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject"
  }
}
```

```css
textarea { display: block; margin-top: 5px; margin-bottom: 10px; }
```

</Sandpack>

`{__html}` 对象应尽可能在生成 HTML 的位置附近创建，就像上面的示例在 `renderMarkdownToHTML` 函数中所做的那样。这样可以确保代码中使用的所有原始 HTML 都被明确标记，同时也确保只有你期望包含 HTML 的变量会传给 `dangerouslySetInnerHTML`。不建议像 `<div dangerouslySetInnerHTML={{__html: markup}} />` 这样内联创建该对象。

要了解为什么渲染任意 HTML 是危险的，请将上面的代码替换为以下内容：

```js {1-4,7,8}
const post = {
  // 想象一下，这段内容存储在数据库中。
  content: `<img src="" onerror='alert("you were hacked")'>`
};

export default function MarkdownPreview() {
  // 🔴 安全漏洞：将不受信任的输入传递给 dangerouslySetInnerHTML
  const markup = { __html: post.content };
  return <div dangerouslySetInnerHTML={markup} />;
}
```

嵌入在 HTML 中的代码会运行。黑客可以利用这个安全漏洞窃取用户信息，或者代表他们执行操作。**只应将 `dangerouslySetInnerHTML` 用于可信且经过清理的数据。**

---

### 处理鼠标事件 {/*handling-mouse-events*/}

此示例展示了一些常见的 [鼠标事件](#mouseevent-handler) 以及它们何时触发。

<Sandpack>

```js
export default function MouseExample() {
  return (
    <div
      onMouseEnter={e => console.log('onMouseEnter (parent)')}
      onMouseLeave={e => console.log('onMouseLeave (parent)')}
    >
      <button
        onClick={e => console.log('onClick (first button)')}
        onMouseDown={e => console.log('onMouseDown (first button)')}
        onMouseEnter={e => console.log('onMouseEnter (first button)')}
        onMouseLeave={e => console.log('onMouseLeave (first button)')}
        onMouseOver={e => console.log('onMouseOver (first button)')}
        onMouseUp={e => console.log('onMouseUp (first button)')}
      >
        第一个按钮
      </button>
      <button
        onClick={e => console.log('onClick (second button)')}
        onMouseDown={e => console.log('onMouseDown (second button)')}
        onMouseEnter={e => console.log('onMouseEnter (second button)')}
        onMouseLeave={e => console.log('onMouseLeave (second button)')}
        onMouseOver={e => console.log('onMouseOver (second button)')}
        onMouseUp={e => console.log('onMouseUp (second button)')}
      >
        第二个按钮
      </button>
    </div>
  );
}
```

```css
label { display: block; }
input { margin-left: 10px; }
```

</Sandpack>

---

### 处理指针事件 {/*handling-pointer-events*/}

此示例展示了一些常见的 [指针事件](#pointerevent-handler) 以及它们何时触发。

<Sandpack>

```js
export default function PointerExample() {
  return (
    <div
      onPointerEnter={e => console.log('onPointerEnter (parent)')}
      onPointerLeave={e => console.log('onPointerLeave (parent)')}
      style={{ padding: 20, backgroundColor: '#ddd' }}
    >
      <div
        onPointerDown={e => console.log('onPointerDown (first child)')}
        onPointerEnter={e => console.log('onPointerEnter (first child)')}
        onPointerLeave={e => console.log('onPointerLeave (first child)')}
        onPointerMove={e => console.log('onPointerMove (first child)')}
        onPointerUp={e => console.log('onPointerUp (first child)')}
        style={{ padding: 20, backgroundColor: 'lightyellow' }}
      >
        第一个子元素
      </div>
      <div
        onPointerDown={e => console.log('onPointerDown (second child)')}
        onPointerEnter={e => console.log('onPointerEnter (second child)')}
        onPointerLeave={e => console.log('onPointerLeave (second child)')}
        onPointerMove={e => console.log('onPointerMove (second child)')}
        onPointerUp={e => console.log('onPointerUp (second child)')}
        style={{ padding: 20, backgroundColor: 'lightblue' }}
      >
        第二个子元素
      </div>
    </div>
  );
}
```

```css
label { display: block; }
input { margin-left: 10px; }
```

</Sandpack>

---

### 处理焦点事件 {/*handling-focus-events*/}

在 React 中，[焦点事件](#focusevent-handler) 会冒泡。你可以使用 `currentTarget` 和 `relatedTarget` 来区分聚焦或失焦事件是否源自父元素外部。这个示例展示了如何检测子元素获得焦点、父元素获得焦点，以及如何检测焦点进入或离开整个子树。

<Sandpack>

```js
export default function FocusExample() {
  return (
    <div
      tabIndex={1}
      onFocus={(e) => {
        if (e.currentTarget === e.target) {
          console.log('focused parent');
        } else {
          console.log('focused child', e.target.name);
        }
        if (!e.currentTarget.contains(e.relatedTarget)) {
          // 在子元素之间切换焦点时不会触发
          console.log('focus entered parent');
        }
      }}
      onBlur={(e) => {
        if (e.currentTarget === e.target) {
          console.log('unfocused parent');
        } else {
          console.log('unfocused child', e.target.name);
        }
        if (!e.currentTarget.contains(e.relatedTarget)) {
          // 在子元素之间切换焦点时不会触发
          console.log('focus left parent');
        }
      }}
    >
      <label>
        名字：
        <input name="firstName" />
      </label>
      <label>
        姓氏：
        <input name="lastName" />
      </label>
    </div>
  );
}
```

```css
label { display: block; }
input { margin-left: 10px; }
```

</Sandpack>

---

### 处理键盘事件 {/*handling-keyboard-events*/}

此示例展示了一些常见的 [键盘事件](#keyboardevent-handler) 以及它们何时触发。

<Sandpack>

```js
export default function KeyboardExample() {
  return (
    <label>
      名字：
      <input
        name="firstName"
        onKeyDown={e => console.log('onKeyDown:', e.key, e.code)}
        onKeyUp={e => console.log('onKeyUp:', e.key, e.code)}
      />
    </label>
  );
}
```

```css
label { display: block; }
input { margin-left: 10px; }
```

</Sandpack>
