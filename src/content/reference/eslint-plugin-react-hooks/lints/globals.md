---
title: globals
---

<Intro>

验证在渲染期间对全局变量的赋值/修改，这是确保[副作用必须在渲染之外运行](/reference/rules/components-and-hooks-must-be-pure#side-effects-must-run-outside-of-render)的一部分。

</Intro>

## Rule Details {/*rule-details*/}

全局变量存在于 React 的控制之外。当你在渲染期间修改它们时，你就打破了 React 关于渲染是纯函数的假设。这可能导致组件在开发环境和生产环境中的行为不同，破坏 Fast Refresh，并使你的应用无法通过 React Compiler 等功能进行优化。

### Invalid {/*invalid*/}

以下是此规则的错误代码示例：

```js
// ❌ 全局计数器
let renderCount = 0;
function Component() {
  renderCount++; // 修改全局变量
  return <div>Count: {renderCount}</div>;
}

// ❌ 修改 window 属性
function Component({userId}) {
  window.currentUser = userId; // 全局修改
  return <div>User: {userId}</div>;
}

// ❌ 向全局数组追加
const events = [];
function Component({event}) {
  events.push(event); // 修改全局数组
  return <div>Events: {events.length}</div>;
}

// ❌ 缓存操作
const cache = {};
function Component({id}) {
  if (!cache[id]) {
    cache[id] = fetchData(id); // 在渲染期间修改缓存
  }
  return <div>{cache[id]}</div>;
}
```

### Valid {/*valid*/}

以下是此规则的正确代码示例：

```js
// ✅ 使用状态来统计计数器
function Component() {
  const [clickCount, setClickCount] = useState(0);

  const handleClick = () => {
    setClickCount(c => c + 1);
  };

  return (
    <button onClick={handleClick}>
      已点击：{clickCount} 次
    </button>
  );
}

// ✅ 使用上下文存储全局值
function Component() {
  const user = useContext(UserContext);
  return <div>用户：{user.id}</div>;
}

// ✅ 将外部状态与 React 同步
function Component({title}) {
  useEffect(() => {
    document.title = title; // 在 effect 中是可以的
  }, [title]);

  return <div>页面：{title}</div>;
}
```
