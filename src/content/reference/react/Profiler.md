---
title: <Profiler>
---

<Intro>

`<Profiler>` 允许你以编程方式测量 React 树的渲染性能。

```js
<Profiler id="App" onRender={onRender}>
  <App />
</Profiler>
```

</Intro>

<InlineToc />

---

## 参考 {/*reference*/}

### `<Profiler>` {/*profiler*/}

将组件树包裹在 `<Profiler>` 中，以测量其渲染性能。

```js
<Profiler id="App" onRender={onRender}>
  <App />
</Profiler>
```

#### 属性 {/*props*/}

* `id`：一个用于标识你正在测量的 UI 部分的字符串。
* `onRender`：一个 [`onRender` 回调](#onrender-callback)，React 会在被分析的树中的组件每次更新时调用它。它会接收有关渲染了什么以及耗时多少的信息。

#### 注意事项 {/*caveats*/}

* 分析会带来一些额外开销，因此**默认在生产构建中是禁用的。** 若要在生产环境中启用分析，你需要启用一个[启用了分析的特殊生产构建。](/reference/dev-tools/react-performance-tracks#using-profiling-builds)

---

### `onRender` 回调 {/*onrender-callback*/}

React 会调用你的 `onRender` 回调，并传入有关渲染内容的信息。

```js
function onRender(id, phase, actualDuration, baseDuration, startTime, commitTime) {
  // 汇总或记录渲染耗时...
}
```

#### 参数 {/*onrender-parameters*/}

* `id`：刚刚提交的 `<Profiler>` 树的字符串 `id` 属性。如果你使用了多个 profiler，这可以帮助你识别是哪一部分树被提交了。
* `phase`：`"mount"`、`"update"` 或 `"nested-update"`。这可以让你知道该树是刚刚第一次挂载，还是由于 props、state 或 Hooks 的变化而重新渲染。
* `actualDuration`：当前更新中，渲染 `<Profiler>` 及其后代所花费的毫秒数。这表示子树对 memoization（例如 [`memo`](/reference/react/memo) 和 [`useMemo`](/reference/react/useMemo)）的利用程度。理想情况下，在初始挂载之后，这个值应显著下降，因为许多后代只需要在其特定 props 改变时才重新渲染。
* `baseDuration`：估算在没有任何优化的情况下，重新渲染整个 `<Profiler>` 子树所需的毫秒数。它通过将树中每个组件最近一次的渲染耗时相加计算得出。这个值估计的是渲染的最坏情况成本（例如初始挂载，或一个没有 memoization 的树）。将 `actualDuration` 与它比较，看看 memoization 是否生效。
* `startTime`：React 开始渲染当前更新时的数值时间戳。
* `commitTime`：React 提交当前更新时的数值时间戳。该值在一次提交中的所有 profiler 之间共享，因此在需要时可以将它们分组。

---

## 用法 {/*usage*/}

### 以编程方式测量渲染性能 {/*measuring-rendering-performance-programmatically*/}

将 `<Profiler>` 组件包裹在 React 树外层，以测量其渲染性能。

```js {2,4}
<App>
  <Profiler id="Sidebar" onRender={onRender}>
    <Sidebar />
  </Profiler>
  <PageContent />
</App>
```

它需要两个 props：一个 `id`（字符串）和一个 `onRender` 回调（函数），React 会在树中的某个组件“提交”更新时调用它。

<Pitfall>

分析会带来一些额外开销，因此**默认在生产构建中是禁用的。** 若要在生产环境中启用分析，你需要启用一个[启用了分析的特殊生产构建。](/reference/dev-tools/react-performance-tracks#using-profiling-builds)

</Pitfall>

<Note>

`<Profiler>` 允许你以编程方式收集测量数据。如果你想要交互式 profiler，可以试试 [React Developer Tools](/learn/react-developer-tools) 中的 Profiler 标签页。它作为浏览器扩展提供了类似的功能。

被 `<Profiler>` 包裹的组件也会在 React Performance tracks 的 [Component tracks](/reference/dev-tools/react-performance-tracks#components) 中被标记，即使在 profiling 构建中也是如此。
在开发构建中，无论组件是否被 `<Profiler>` 包裹，都会在 Components track 中被标记。

</Note>

---

### 测量应用的不同部分 {/*measuring-different-parts-of-the-application*/}

你可以使用多个 `<Profiler>` 组件来测量应用的不同部分：

```js {5,7}
<App>
  <Profiler id="Sidebar" onRender={onRender}>
    <Sidebar />
  </Profiler>
  <Profiler id="Content" onRender={onRender}>
    <Content />
  </Profiler>
</App>
```

你也可以嵌套 `<Profiler>` 组件：

```js {5,7,9,12}
<App>
  <Profiler id="Sidebar" onRender={onRender}>
    <Sidebar />
  </Profiler>
  <Profiler id="Content" onRender={onRender}>
    <Content>
      <Profiler id="Editor" onRender={onRender}>
        <Editor />
      </Profiler>
      <Preview />
    </Content>
  </Profiler>
</App>
```

尽管 `<Profiler>` 是一个轻量级组件，但它应仅在必要时使用。每次使用都会为应用增加一些 CPU 和内存开销。

---

