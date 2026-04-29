---
title: 无效的 ARIA 属性警告
---

如果你尝试渲染一个带有 `aria-*` 属性的 DOM 元素，而该属性并不存在于 Web Accessibility Initiative（WAI）Accessible Rich Internet Application（ARIA）[规范](https://www.w3.org/TR/wai-aria-1.1/#states_and_properties)中，就会触发此警告。

1. 如果你认为自己使用的是一个有效的属性，请仔细检查拼写。`aria-labelledby` 和 `aria-activedescendant` 经常被拼错。

2. 如果你写的是 `aria-role`，你可能想写的是 `role`。

3. 否则，如果你使用的是最新版本的 React DOM，并且已经确认你使用的是 ARIA 规范中列出的有效属性名，请[报告一个 bug](https://github.com/facebook/react/issues/new/choose)。
