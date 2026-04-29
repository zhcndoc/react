---
title: use-memo
---

<Intro>

验证 `useMemo` Hook 是否带有返回值。更多信息请参见 [`useMemo` 文档](/reference/react/useMemo)。

</Intro>

## 规则详情 {/*rule-details*/}

`useMemo` 用于计算并缓存昂贵的值，而不是用于副作用。没有返回值时，`useMemo` 会返回 `undefined`，这就失去了它的用途，而且很可能说明你使用了错误的 Hook。

### 无效 {/*invalid*/}

此规则的错误代码示例如下：

```js {expectedErrors: {'react-compiler': [3]}}
// ❌ 没有返回值
function Component({ data }) {
  const processed = useMemo(() => {
    data.forEach(item => console.log(item));
    // 缺少 return！
  }, [data]);

  return <div>{processed}</div>; // 始终为 undefined
}
```

### 有效 {/*valid*/}

此规则的正确代码示例如下：

```js
// ✅ 返回计算出的值
function Component({ data }) {
  const processed = useMemo(() => {
    return data.map(item => item * 2);
  }, [data]);

  return <div>{processed}</div>;
}
```

## 故障排查 {/*troubleshooting*/}

### 当依赖项变化时，我需要运行副作用 {/*side-effects*/}

你可能会尝试使用 `useMemo` 来处理副作用：

{/* TODO(@poteto) fix compiler validation to check for unassigned useMemos */}
```js {expectedErrors: {'react-compiler': [4]}}
// ❌ 错误：useMemo 中的副作用
function Component({user}) {
  // 没有返回值，只是副作用
  useMemo(() => {
    analytics.track('UserViewed', {userId: user.id});
  }, [user.id]);

  // 没有赋值给变量
  useMemo(() => {
    return analytics.track('UserViewed', {userId: user.id});
  }, [user.id]);
}
```

如果副作用需要在用户交互时发生，最好把副作用与事件放在一起：

```js
// ✅ 好：在事件处理函数中处理副作用
function Component({user}) {
  const handleClick = () => {
    analytics.track('ButtonClicked', {userId: user.id});
    // 其他点击逻辑...
  };

  return <button onClick={handleClick}>点击我</button>;
}
```

如果副作用是在将 React 状态与某些外部状态同步（或反之），请使用 `useEffect`：

```js
// ✅ 好：在 useEffect 中进行同步
function Component({theme}) {
  useEffect(() => {
    localStorage.setItem('preferredTheme', theme);
    document.body.className = theme;
  }, [theme]);

  return <div>当前主题：{theme}</div>;
}
```
