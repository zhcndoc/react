---
title: "内置 React DOM Hooks"
---

<Intro>

`react-dom` 包包含仅受 Web 应用程序支持的 Hooks（这些应用程序运行在浏览器 DOM 环境中）。这些 Hooks 不受 iOS、Android 或 Windows 应用程序等非浏览器环境支持。如果你正在寻找在 Web 浏览器 *以及其他环境* 中都受支持的 Hooks，请参见 [React Hooks 页面](/reference/react/hooks)。此页面列出了 `react-dom` 包中的所有 Hooks。

</Intro>

---

## 表单 Hooks {/*form-hooks*/}

*表单* 让你可以创建用于提交信息的交互式控件。要在组件中管理表单，请使用以下 Hooks 之一：

* [`useFormStatus`](/reference/react-dom/hooks/useFormStatus) 允许你根据表单状态更新 UI。

```js
function Form({ action }) {
  async function increment(n) {
    return n + 1;
  }
  const [count, incrementFormAction] = useActionState(increment, 0);
  return (
    <form action={action}>
      <button formAction={incrementFormAction}>计数：{count}</button>
      <Button />
    </form>
  );
}

function Button() {
  const { pending } = useFormStatus();
  return (
    <button disabled={pending} type="submit">
      提交
    </button>
  );
}
```
