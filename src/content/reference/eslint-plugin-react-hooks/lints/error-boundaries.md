---
title: error-boundaries
---

<Intro>

验证在子组件错误处理中使用 Error Boundaries，而不是 try/catch。

</Intro>

## 规则详情 {/*rule-details*/}

try/catch 块无法捕获 React 渲染过程中发生的错误。在渲染方法或 hooks 中抛出的错误会沿着组件树向上冒泡。只有 [Error Boundaries](/reference/react/Component#catching-rendering-errors-with-an-error-boundary) 能捕获这些错误。

### 无效 {/*invalid*/}

此规则的错误代码示例：

```js {expectedErrors: {'react-compiler': [4]}}
// ❌ try/catch 无法捕获渲染错误
function Parent() {
  try {
    return <ChildComponent />; // 如果这里抛出错误，catch 也无能为力
  } catch (error) {
    return <div>发生错误</div>;
  }
}
```

### 有效 {/*valid*/}

此规则的正确代码示例：

```js
// ✅ 使用错误边界
function Parent() {
  return (
    <ErrorBoundary>
      <ChildComponent />
    </ErrorBoundary>
  );
}
```

## 故障排查 {/*troubleshooting*/}

### 为什么 linter 提示我不要把 `use` 包在 `try`/`catch` 里？ {/*why-is-the-linter-telling-me-not-to-wrap-use-in-trycatch*/}

`use` Hook 并不会以传统意义上的方式抛出错误，它会挂起组件执行。当 `use` 遇到一个待处理的 promise 时，它会挂起该组件并让 React 显示回退内容。只有 Suspense 和 Error Boundaries 能处理这些情况。linter 警告不要在 `use` 外层使用 `try`/`catch`，是为了避免误解，因为 `catch` 块永远不会执行。

```js {expectedErrors: {'react-compiler': [5]}}
// ❌ 在 `use` hook 外层使用 try/catch
function Component({promise}) {
  try {
    const data = use(promise); // 不会被 catch 捕获 - `use` 是挂起，不是抛出
    return <div>{data}</div>;
  } catch (error) {
    return <div>加载失败</div>; // 不可达
  }
}

// ✅ Error boundary 捕获 `use` 错误
function App() {
  return (
    <ErrorBoundary fallback={<div>加载失败</div>}>
      <Suspense fallback={<div>加载中...</div>}>
        <DataComponent promise={fetchData()} />
      </Suspense>
    </ErrorBoundary>
  );
}
```