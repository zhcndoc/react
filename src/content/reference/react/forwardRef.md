---
title: forwardRef
---

<Deprecated>

在 React 19 中，`forwardRef` 已不再需要。请改为将 `ref` 作为 prop 传递。

`forwardRef` 将在未来的版本中被弃用。了解更多信息 [这里](/blog/2024/04/25/react-19#ref-as-a-prop)。

</Deprecated>

<Intro>

`forwardRef` 让你的组件可以通过 [ref.](/learn/manipulating-the-dom-with-refs) 向父组件暴露一个 DOM 节点

```js
const SomeComponent = forwardRef(render)
```

</Intro>

<InlineToc />

---

## 参考 {/*reference*/}

### `forwardRef(render)` {/*forwardref*/}

调用 `forwardRef()`，让你的组件接收一个 ref，并将它转发给子组件：

```js
import { forwardRef } from 'react';

const MyInput = forwardRef(function MyInput(props, ref) {
  // ...
});
```

[查看更多示例。](#usage)

#### 参数 {/*parameters*/}

* `render`：你组件的渲染函数。React 会使用你的组件从父组件接收到的 props 和 `ref` 调用这个函数。你返回的 JSX 将成为你组件的输出。

#### 返回值 {/*returns*/}

`forwardRef` 返回一个可以在 JSX 中渲染的 React 组件。与普通函数定义的 React 组件不同，`forwardRef` 返回的组件也能够接收 `ref` prop。

#### 注意事项 {/*caveats*/}

* 在严格模式下，React 会**调用你的渲染函数两次**，以[帮助你发现意外的副作用。](/reference/react/useState#my-initializer-or-updater-function-runs-twice)这是仅在开发环境中的行为，不会影响生产环境。如果你的渲染函数是纯函数（它应该如此），这不会影响你组件的逻辑。两次调用中的一次结果会被忽略。


---

### `render` 函数 {/*render-function*/}

`forwardRef` 接受一个渲染函数作为参数。React 会使用 `props` 和 `ref` 调用这个函数：

```js
const MyInput = forwardRef(function MyInput(props, ref) {
  return (
    <label>
      {props.label}
      <input ref={ref} />
    </label>
  );
});
```

#### 参数 {/*render-parameters*/}

* `props`：父组件传递的 props。

* `ref`：父组件传递的 `ref` 属性。`ref` 可以是一个对象或一个函数。如果父组件没有传递 ref，它将为 `null`。你应该将接收到的 `ref` 传递给另一个组件，或者传给 [`useImperativeHandle`.](/reference/react/useImperativeHandle)

#### 返回值 {/*render-returns*/}

`forwardRef` 返回一个可以在 JSX 中渲染的 React 组件。与普通函数定义的 React 组件不同，`forwardRef` 返回的组件可以接收 `ref` prop。

---

## 用法 {/*usage*/}

### 向父组件暴露一个 DOM 节点 {/*exposing-a-dom-node-to-the-parent-component*/}

默认情况下，每个组件的 DOM 节点都是私有的。不过，有时向父组件暴露一个 DOM 节点会很有用——例如，允许它获得焦点。要启用这一点，请把你的组件定义包裹在 `forwardRef()` 中：

```js {3,11}
import { forwardRef } from 'react';

const MyInput = forwardRef(function MyInput(props, ref) {
  const { label, ...otherProps } = props;
  return (
    <label>
      {label}
      <input {...otherProps} />
    </label>
  );
});
```

你会在 props 之后收到第二个参数 <CodeStep step={1}>ref</CodeStep>。把它传给你想要暴露的 DOM 节点：

```js {8} [[1, 3, "ref"], [1, 8, "ref", 30]]
import { forwardRef } from 'react';

const MyInput = forwardRef(function MyInput(props, ref) {
  const { label, ...otherProps } = props;
  return (
    <label>
      {label}
      <input {...otherProps} ref={ref} />
    </label>
  );
});
```

这让父组件 `Form` 能够访问 `MyInput` 暴露的 <CodeStep step={2}>`<input>` DOM 节点</CodeStep>：

```js [[1, 2, "ref"], [1, 10, "ref", 41], [2, 5, "ref.current"]]
function Form() {
  const ref = useRef(null);

  function handleClick() {
    ref.current.focus();
  }

  return (
    <form>
      <MyInput label="Enter your name:" ref={ref} />
      <button type="button" onClick={handleClick}>
        Edit
      </button>
    </form>
  );
}
```

这个 `Form` 组件[传递了一个 ref](/reference/react/useRef#manipulating-the-dom-with-a-ref) 给 `MyInput`。`MyInput` 组件将这个 ref *转发* 到 `<input>` 浏览器标签。结果，`Form` 组件可以访问那个 `<input>` DOM 节点，并对它调用 [`focus()`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/focus)。

请记住，将 ref 暴露给组件内部的 DOM 节点，会让你以后更难修改组件内部实现。通常你会从可复用的底层组件中暴露 DOM 节点，比如按钮或文本输入框，但不会对头像或评论这类应用层组件这样做。

<Recipes titleText="转发 ref 的示例">

#### 聚焦文本输入框 {/*focusing-a-text-input*/}

点击按钮会聚焦输入框。`Form` 组件定义了一个 ref，并将它传递给 `MyInput` 组件。`MyInput` 组件将这个 ref 转发给浏览器 `<input>`。这让 `Form` 组件可以聚焦 `<input>`。

<Sandpack>

```js
import { useRef } from 'react';
import MyInput from './MyInput.js';

export default function Form() {
  const ref = useRef(null);

  function handleClick() {
    ref.current.focus();
  }

  return (
    <form>
      <MyInput label="Enter your name:" ref={ref} />
      <button type="button" onClick={handleClick}>
        Edit
      </button>
    </form>
  );
}
```

```js src/MyInput.js
import { forwardRef } from 'react';

const MyInput = forwardRef(function MyInput(props, ref) {
  const { label, ...otherProps } = props;
  return (
    <label>
      {label}
      <input {...otherProps} ref={ref} />
    </label>
  );
});

export default MyInput;
```

```css
input {
  margin: 5px;
}
```

</Sandpack>

<Solution />

#### 播放和暂停视频 {/*playing-and-pausing-a-video*/}

点击按钮会在一个 `<video>` DOM 节点上调用 [`play()`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLMediaElement/play) 和 [`pause()`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLMediaElement/pause)。`App` 组件定义了一个 ref，并将它传递给 `MyVideoPlayer` 组件。`MyVideoPlayer` 组件将这个 ref 转发给浏览器 `<video>` 节点。这让 `App` 组件可以播放和暂停 `<video>`。

<Sandpack>

```js
import { useRef } from 'react';
import MyVideoPlayer from './MyVideoPlayer.js';

export default function App() {
  const ref = useRef(null);
  return (
    <>
      <button onClick={() => ref.current.play()}>
        播放
      </button>
      <button onClick={() => ref.current.pause()}>
        暂停
      </button>
      <br />
      <MyVideoPlayer
        ref={ref}
        src="https://interactive-examples.mdn.mozilla.net/media/cc0-videos/flower.mp4"
        type="video/mp4"
        width="250"
      />
    </>
  );
}
```

```js src/MyVideoPlayer.js
import { forwardRef } from 'react';

const VideoPlayer = forwardRef(function VideoPlayer({ src, type, width }, ref) {
  return (
    <video width={width} ref={ref}>
      <source
        src={src}
        type={type}
      />
    </video>
  );
});

export default VideoPlayer;
```

```css
button { margin-bottom: 10px; margin-right: 10px; }
```

</Sandpack>

<Solution />

</Recipes>

---

### 通过多个组件转发 ref {/*forwarding-a-ref-through-multiple-components*/}

你可以不将 `ref` 转发给某个 DOM 节点，而是把它转发给你自己的组件，比如 `MyInput`：

```js {1,5}
const FormField = forwardRef(function FormField(props, ref) {
  // ...
  return (
    <>
      <MyInput ref={ref} />
      ...
    </>
  );
});
```

如果那个 `MyInput` 组件将 ref 转发到它的 `<input>` 上，那么指向 `FormField` 的 ref 就会给你那个 `<input>`：

```js {2,5,10}
function Form() {
  const ref = useRef(null);

  function handleClick() {
    ref.current.focus();
  }

  return (
    <form>
      <FormField label="Enter your name:" ref={ref} isRequired={true} />
      <button type="button" onClick={handleClick}>
        Edit
      </button>
    </form>
  );
}
```

`Form` 组件定义了一个 ref，并将它传递给 `FormField`。`FormField` 组件将这个 ref 转发给 `MyInput`，而 `MyInput` 又将它转发给浏览器的 `<input>` DOM 节点。这就是 `Form` 访问那个 DOM 节点的方式。


<Sandpack>

```js
import { useRef } from 'react';
import FormField from './FormField.js';

export default function Form() {
  const ref = useRef(null);

  function handleClick() {
    ref.current.focus();
  }

  return (
    <form>
      <FormField label="Enter your name:" ref={ref} isRequired={true} />
      <button type="button" onClick={handleClick}>
        Edit
      </button>
    </form>
  );
}
```

```js src/FormField.js
import { forwardRef, useState } from 'react';
import MyInput from './MyInput.js';

const FormField = forwardRef(function FormField({ label, isRequired }, ref) {
  const [value, setValue] = useState('');
  return (
    <>
      <MyInput
        ref={ref}
        label={label}
        value={value}
        onChange={e => setValue(e.target.value)}
      />
      {(isRequired && value === '') &&
        <i>必填</i>
      }
    </>
  );
});

export default FormField;
```


```js src/MyInput.js
import { forwardRef } from 'react';

const MyInput = forwardRef((props, ref) => {
  const { label, ...otherProps } = props;
  return (
    <label>
      {label}
      <input {...otherProps} ref={ref} />
    </label>
  );
});

export default MyInput;
```

```css
input, button {
  margin: 5px;
}
```

</Sandpack>

---

### 暴露一个命令式句柄，而不是 DOM 节点 {/*exposing-an-imperative-handle-instead-of-a-dom-node*/}

你可以不暴露整个 DOM 节点，而是暴露一个自定义对象，称为 *命令式句柄*，并提供一组更受限制的方法。为此，你需要定义一个单独的 ref 来保存 DOM 节点：

```js {2,6}
const MyInput = forwardRef(function MyInput(props, ref) {
  const inputRef = useRef(null);

  // ...

  return <input {...props} ref={inputRef} />;
});
```

将你收到的 `ref` 传给 [`useImperativeHandle`](/reference/react/useImperativeHandle)，并指定你想要暴露给 `ref` 的值：

```js {6-15}
import { forwardRef, useRef, useImperativeHandle } from 'react';

const MyInput = forwardRef(function MyInput(props, ref) {
  const inputRef = useRef(null);

  useImperativeHandle(ref, () => {
    return {
      focus() {
        inputRef.current.focus();
      },
      scrollIntoView() {
        inputRef.current.scrollIntoView();
      },
    };
  }, []);

  return <input {...props} ref={inputRef} />;
});
```

如果某个组件拿到了 `MyInput` 的 ref，它接收到的将只是你的 `{ focus, scrollIntoView }` 对象，而不是 DOM 节点。这样你就可以把暴露给外界的 DOM 节点信息限制到最少。

<Sandpack>

```js
import { useRef } from 'react';
import MyInput from './MyInput.js';

export default function Form() {
  const ref = useRef(null);

  function handleClick() {
    ref.current.focus();
    // 这将不起作用，因为 DOM 节点没有被暴露：
    // ref.current.style.opacity = 0.5;
  }

  return (
    <form>
      <MyInput placeholder="Enter your name" ref={ref} />
      <button type="button" onClick={handleClick}>
        Edit
      </button>
    </form>
  );
}
```

```js src/MyInput.js
import { forwardRef, useRef, useImperativeHandle } from 'react';

const MyInput = forwardRef(function MyInput(props, ref) {
  const inputRef = useRef(null);

  useImperativeHandle(ref, () => {
    return {
      focus() {
        inputRef.current.focus();
      },
      scrollIntoView() {
        inputRef.current.scrollIntoView();
      },
    };
  }, []);

  return <input {...props} ref={inputRef} />;
});

export default MyInput;
```

```css
input {
  margin: 5px;
}
```

</Sandpack>

[阅读更多关于使用命令式句柄的内容。](/reference/react/useImperativeHandle)

<Pitfall>

**不要过度使用 refs。** 你应该只将 refs 用于那些你无法用 props 表达的 *命令式* 行为：例如，滚动到某个节点、聚焦某个节点、触发动画、选中文本，等等。

**如果某件事可以用 prop 表达，就不应该使用 ref。** 例如，不要从 `Modal` 组件中暴露像 `{ open, close }` 这样的命令式句柄，而应该把 `isOpen` 作为 prop 传入，例如 `<Modal isOpen={isOpen} />`。[Effects](/learn/synchronizing-with-effects) 可以帮助你通过 props 暴露命令式行为。

</Pitfall>

---

## 故障排除 {/*troubleshooting*/}

### 我的组件被 `forwardRef` 包裹了，但它的 `ref` 总是 `null` {/*my-component-is-wrapped-in-forwardref-but-the-ref-to-it-is-always-null*/}

这通常意味着你忘了实际使用你收到的 `ref`。

例如，这个组件没有对它的 `ref` 做任何事情：

```js {1}
const MyInput = forwardRef(function MyInput({ label }, ref) {
  return (
    <label>
      {label}
      <input />
    </label>
  );
});
```

要修复它，请把 `ref` 传递给一个 DOM 节点，或者另一个可以接受 ref 的组件：

```js {1,5}
const MyInput = forwardRef(function MyInput({ label }, ref) {
  return (
    <label>
      {label}
      <input ref={ref} />
    </label>
  );
});
```

如果某些逻辑是条件性的，`MyInput` 的 `ref` 也可能是 `null`：

```js {1,5}
const MyInput = forwardRef(function MyInput({ label, showInput }, ref) {
  return (
    <label>
      {label}
      {showInput && <input ref={ref} />}
    </label>
  );
});
```

如果 `showInput` 是 `false`，那么 ref 不会被转发到任何节点，`MyInput` 的 ref 将保持为空。如果条件隐藏在另一个组件中，这一点尤其容易被忽略，比如这个例子中的 `Panel`：

```js {5,7}
const MyInput = forwardRef(function MyInput({ label, showInput }, ref) {
  return (
    <label>
      {label}
      <Panel isExpanded={showInput}>
        <input ref={ref} />
      </Panel>
    </label>
  );
});
```
