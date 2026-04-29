---
title: useImperativeHandle
---

<Intro>

`useImperativeHandle` 是一个 React Hook，它可以让你自定义作为 [ref.](/learn/manipulating-the-dom-with-refs) 暴露出去的句柄。

```js
useImperativeHandle(ref, createHandle, dependencies?)
```

</Intro>

<InlineToc />

---

## 参考 {/*reference*/}

### `useImperativeHandle(ref, createHandle, dependencies?)` {/*useimperativehandle*/}

在组件顶层调用 `useImperativeHandle`，以自定义它所暴露的 ref 句柄：

```js
import { useImperativeHandle } from 'react';

function MyInput({ ref }) {
  useImperativeHandle(ref, () => {
    return {
      // ... 你的方法 ...
    };
  }, []);
  // ...
```

[查看更多示例。](#usage)

#### 参数 {/*parameters*/}

* `ref`：你作为 props 从 `MyInput` 组件中接收到的 `ref`。

* `createHandle`：一个不接受参数并返回你想要暴露的 ref 句柄的函数。该 ref 句柄可以是任何类型。通常，你会返回一个包含你想要暴露的方法的对象。

* **可选** `dependencies`：`createHandle` 代码内部引用的所有响应式值列表。响应式值包括 props、state，以及直接在组件函数体内声明的所有变量和函数。如果你的 linter 已为 [React 配置](/learn/editor-setup#linting)，它会验证每个响应式值都被正确地指定为依赖项。依赖项列表必须包含固定数量的项，并且要内联写成 `[dep1, dep2, dep3]` 这样的形式。React 会使用 [`Object.is`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is) 比较来对比每个依赖项与其之前的值。如果一次重新渲染导致某个依赖项发生变化，或者你省略了这个参数，你的 `createHandle` 函数将会重新执行，并且新创建的句柄会被赋值给 ref。

<Note>

从 React 19 开始，[`ref` 可以作为 prop 使用。](/blog/2024/12/05/react-19#ref-as-a-prop) 在 React 18 及更早版本中，需要通过 [`forwardRef`.](/reference/react/forwardRef) 获取 `ref`。

</Note>

#### 返回值 {/*returns*/}

`useImperativeHandle` 返回 `undefined`。

---

## 用法 {/*usage*/}

### 向父组件暴露自定义的 ref 句柄 {/*exposing-a-custom-ref-handle-to-the-parent-component*/}

要向父元素暴露一个 DOM 节点，请将 `ref` prop 传给该节点。

```js {2}
function MyInput({ ref }) {
  return <input ref={ref} />;
};
```

使用上面的代码时，[`MyInput` 的 ref 将接收到 `<input>` DOM 节点。](/learn/manipulating-the-dom-with-refs) 不过，你也可以改为暴露一个自定义值。要自定义暴露的句柄，请在组件顶层调用 `useImperativeHandle`：

```js {4-8}
import { useImperativeHandle } from 'react';

function MyInput({ ref }) {
  useImperativeHandle(ref, () => {
    return {
      // ... 你的方法 ...
    };
  }, []);

  return <input />;
};
```

注意，在上面的代码中，`ref` 不再传给 `<input>`。

例如，假设你不想暴露整个 `<input>` DOM 节点，而是想暴露它的两个方法：`focus` 和 `scrollIntoView`。为此，将真实的浏览器 DOM 保存在一个单独的 ref 中。然后使用 `useImperativeHandle` 来暴露一个只包含你希望父组件调用的方法的句柄：

```js {7-14}
import { useRef, useImperativeHandle } from 'react';

function MyInput({ ref }) {
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

  return <input ref={inputRef} />;
};
```

现在，如果父组件获得了 `MyInput` 的 ref，它就可以调用其上的 `focus` 和 `scrollIntoView` 方法。不过，它不会获得底层 `<input>` DOM 节点的完整访问权限。

<Sandpack>

```js
import { useRef } from 'react';
import MyInput from './MyInput.js';

export default function Form() {
  const ref = useRef(null);

  function handleClick() {
    ref.current.focus();
    // 这不会起作用，因为 DOM 节点没有被暴露出来：
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
import { useRef, useImperativeHandle } from 'react';

function MyInput({ ref, ...props }) {
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
};

export default MyInput;
```

```css
input {
  margin: 5px;
}
```

</Sandpack>

---

### 暴露你自己的命令式方法 {/*exposing-your-own-imperative-methods*/}

你通过命令式句柄暴露的方法不必与 DOM 方法完全一致。例如，这个 `Post` 组件通过命令式句柄暴露了一个 `scrollAndFocusAddComment` 方法。这让父组件 `Page` 能够在你点击按钮时滚动评论列表，并聚焦输入框：

<Sandpack>

```js
import { useRef } from 'react';
import Post from './Post.js';

export default function Page() {
  const postRef = useRef(null);

  function handleClick() {
    postRef.current.scrollAndFocusAddComment();
  }

  return (
    <>
      <button onClick={handleClick}>
        Write a comment
      </button>
      <Post ref={postRef} />
    </>
  );
}
```

```js src/Post.js
import { useRef, useImperativeHandle } from 'react';
import CommentList from './CommentList.js';
import AddComment from './AddComment.js';

function Post({ ref }) {
  const commentsRef = useRef(null);
  const addCommentRef = useRef(null);

  useImperativeHandle(ref, () => {
    return {
      scrollAndFocusAddComment() {
        commentsRef.current.scrollToBottom();
        addCommentRef.current.focus();
      }
    };
  }, []);

  return (
    <>
      <article>
        <p>欢迎来到我的博客！</p>
      </article>
      <CommentList ref={commentsRef} />
      <AddComment ref={addCommentRef} />
    </>
  );
};

export default Post;
```


```js src/CommentList.js
import { useRef, useImperativeHandle } from 'react';

function CommentList({ ref }) {
  const divRef = useRef(null);

  useImperativeHandle(ref, () => {
    return {
      scrollToBottom() {
        const node = divRef.current;
        node.scrollTop = node.scrollHeight;
      }
    };
  }, []);

  let comments = [];
  for (let i = 0; i < 50; i++) {
    comments.push(<p key={i}>评论 #{i}</p>);
  }

  return (
    <div className="CommentList" ref={divRef}>
      {comments}
    </div>
  );
}

export default CommentList;
```

```js src/AddComment.js
import { useRef, useImperativeHandle } from 'react';

function AddComment({ ref }) {
  return <input placeholder="添加评论..." ref={ref} />;
}

export default AddComment;
```

```css
.CommentList {
  height: 100px;
  overflow: scroll;
  border: 1px solid black;
  margin-top: 20px;
  margin-bottom: 20px;
}
```

</Sandpack>

<Pitfall>

**不要过度使用 refs。** 你只应该将 refs 用于那些你无法用 props 表达的 *命令式* 行为：例如，滚动到某个节点、聚焦某个节点、触发动画、选中文本，等等。

**如果某件事可以用 prop 表达，就不应该使用 ref。** 例如，与其从 `Modal` 组件中暴露像 `{ open, close }` 这样的命令式句柄，不如将 `isOpen` 作为 prop 传入，例如 `<Modal isOpen={isOpen} />`。[Effects](/learn/synchronizing-with-effects) 可以帮助你通过 props 暴露命令式行为。

</Pitfall>
